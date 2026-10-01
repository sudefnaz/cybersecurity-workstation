# Linux Security Baseline with Ansible

## Project Overview

This project implements an Ansible-based security hardening baseline for Ubuntu 24.04 LTS.

The playbook applies a selected set of security configuration recommendations inspired by the CIS Ubuntu Linux Benchmark. It is not intended to provide complete CIS Benchmark compliance.

The main objective is to create a practical, repeatable and idempotent security baseline for a Linux workstation while preserving normal system usability and reducing the risk of administrative lockout.

## Supported Environment

* Ubuntu 24.04 LTS
* Ansible
* VirtualBox or physical/virtual Ubuntu host
* UFW
* OpenSSH
* auditd
* rsyslog
* AppArmor

The project was developed and tested on an Ubuntu 24.04 LTS virtual machine.

## Project Structure

```text
.
├── group_vars/
│   └── all.yml
├── tasks/
│   ├── additional_hardening.yml
│   ├── filesystem.yml
│   ├── firewall.yml
│   ├── logging.yml
│   ├── network.yml
│   ├── packages.yml
│   ├── services.yml
│   ├── ssh.yml
│   ├── users.yml
│   └── verification.yml
├── inventory
├── site.yml
├── baseline-report.txt
├── CIS-controls.md
├── .gitignore
└── README.md
```

The main playbook is `site.yml`.

Security functions are separated into task files according to their logical area. Configuration values are stored in `group_vars/all.yml`.

## Security Areas

### Package Management

The playbook:

* updates the APT package cache
* ensures required security baseline packages are installed
* installs and uses:

  * UFW
  * OpenSSH server
  * auditd
  * rsyslog
  * libpam-pwquality

Automatic package upgrades are not performed by the playbook because uncontrolled package upgrades can introduce longer execution times and operational changes outside the intended security baseline.

### User and Password Security

The playbook configures:

* root account locking
* password maximum age
* password minimum age
* password warning age
* password quality minimum length
* permissions of `/etc/passwd`
* permissions of `/etc/group`
* permissions of `/etc/shadow`
* permissions of `/etc/gshadow`

The password policy is intended as a baseline rather than a complete identity-management solution.

### SSH Hardening

The playbook configures:

* root login disabled
* empty passwords disabled
* user environment configuration disabled
* PAM enabled
* maximum authentication attempts
* login grace time
* X11 forwarding disabled
* SSH client keepalive settings
* configurable SSH port

The effective SSH configuration is verified with `sshd -T`.

The configuration syntax is also checked with `sshd -t`.

Password authentication is currently kept enabled because public-key deployment and SSH key lifecycle management have not yet been implemented. Disabling password authentication without a verified alternative could cause administrative lockout.

### Firewall

UFW is configured with:

* incoming traffic denied by default
* outgoing traffic allowed by default
* SSH explicitly allowed
* firewall enabled
* firewall status verification

The SSH firewall rule is configured before enabling UFW to reduce the risk of administrative lockout.

The routed UFW policy is also verified on the target system. It is currently disabled by default.

### Network and Kernel Hardening

The playbook configures:

* IPv4 forwarding disabled
* IPv6 forwarding disabled
* IPv4 ICMP redirects disabled
* IPv6 ICMP redirects disabled
* IPv4 ICMP send redirects disabled
* protected hardlinks enabled
* protected symlinks enabled
* restricted ptrace scope
* SUID core dumps disabled
* restricted kernel message access

Kernel parameters are managed through Ansible's sysctl module and stored in:

```text
/etc/sysctl.d/99-cybersecurity-hardening.conf
```

### Logging and Auditing

The project configures:

* rsyslog enabled and running
* auditd enabled and running
* audit rules for security-sensitive files
* monitoring of identity-related files
* monitoring of privilege configuration
* monitoring of SSH configuration
* monitoring of password policy configuration
* monitoring of network configuration
* monitoring of audit configuration
* restricted permissions for `/var/log/audit`

Audit rules are stored in:

```text
/etc/audit/rules.d/50-cybersecurity.rules
```

### Service Hardening

The playbook can disable services that are unnecessary for the intended workstation:

* `avahi-daemon`
* `cups`
* `gnome-remote-desktop`

The service list is configurable through:

```text
group_vars/all.yml
```

The playbook verifies both the active state and enabled state of these services.

### Additional Hardening

The playbook also configures:

* sudo pseudo-terminal support
* cron service
* AppArmor service

The sudo configuration is validated with `visudo` before the configuration is accepted.

### Filesystem Security

The playbook configures permissions for important system files:

* `/etc/passwd`
* `/etc/group`
* `/etc/shadow`
* `/etc/gshadow`
* `/etc/shells`

It also checks for:

* world-writable files
* files without a valid owner
* files without a valid group

These checks are report-only. Discovered files are not automatically deleted or modified because doing so could affect system functionality.

## Configuration

Security settings are stored in:

```text
group_vars/all.yml
```

Important configurable values include:

* UFW policies
* SSH port
* SSH authentication settings
* SSH authentication limits
* password aging values
* network forwarding settings
* unnecessary services
* auditd
* rsyslog
* AppArmor

The playbook uses variables rather than hard-coding environment-specific values where practical.

## Running the Project

### 1. Check the inventory

```bash
cat inventory
```

### 2. Validate the playbook syntax

```bash
ansible-playbook -i inventory site.yml --syntax-check
```

### 3. Run the security baseline

```bash
ansible-playbook -i inventory site.yml --ask-become-pass
```

### 4. Save the execution output

```bash
ansible-playbook -i inventory site.yml --ask-become-pass | tee baseline-report.txt
```

The playbook requires administrative privileges because it modifies system configuration.

## Verification

The playbook performs verification checks after applying the configuration.

Current verification includes:

* UFW status and default policies
* SSH configuration syntax
* effective SSH security settings
* SSH service state
* IPv4 forwarding
* IPv6 forwarding
* auditd status
* kernel hardening parameters
* unnecessary service states
* critical system file permissions
* world-writable files
* unowned or ungrouped files

The verification tasks are intended to provide evidence that the configured baseline is active on the target system.

## Idempotency

Idempotency is an important requirement of the project.

The playbook was executed repeatedly on the same Ubuntu 24.04 LTS system.

The latest repeated execution completed with:

```text
ok=69
changed=0
failed=0
```

A result with `changed=0` indicates that the second execution did not need to make additional configuration changes.

This demonstrates that the implemented configuration is stable when the desired state has already been reached.

## CIS Benchmark Scope

The project is based on selected recommendations from:

**CIS Ubuntu Linux 24.04 LTS Benchmark v2.0.0**

The project does not claim complete CIS compliance.

The selected controls are documented separately in:

```text
CIS-controls.md
```

The control matrix identifies:

* CIS control ID
* recommendation
* corresponding Ansible implementation
* implementation status
* security rationale
* operational trade-offs
* validation approach

The selected controls focus primarily on:

* kernel and process hardening
* unnecessary service reduction
* UFW firewall configuration
* SSH hardening
* sudo configuration
* critical system file permissions

## Idempotency and Safety Considerations

The project is designed to minimize unnecessary changes.

Configuration is primarily performed through Ansible modules such as:

* `ansible.builtin.apt`
* `ansible.builtin.file`
* `ansible.builtin.lineinfile`
* `ansible.builtin.blockinfile`
* `ansible.builtin.service`
* `ansible.posix.sysctl`
* `community.general.ufw`

Configuration validation is used for security-sensitive files where appropriate.

SSH configuration is validated before a reload is performed.

Sudo configuration is validated with `visudo`.

The firewall allows SSH before UFW is enabled to reduce the risk of administrative lockout.

## Limitations and Trade-offs

This project is a practical security baseline rather than a complete enterprise security solution.

Important limitations include:

* it does not implement full CIS Benchmark compliance
* password authentication for SSH remains enabled
* SSH public-key deployment is not automated
* package upgrades are not automatically performed
* service disabling is based on the intended workstation use case
* filesystem anomaly checks are report-only
* the configuration has been tested primarily on Ubuntu 24.04 LTS
* some hardening settings may affect specialized debugging or development workflows

Security controls can introduce operational consequences. For example, disabling unnecessary services can remove functionality, while restrictive firewall and SSH settings can prevent expected connections if additional rules are required.

The selected baseline therefore prioritizes repeatability, verification and safe administration rather than maximum restriction.

## Testing Status

The current development environment is:

```text
Operating System: Ubuntu 24.04 LTS
Environment: VirtualBox
Ansible: 2.16.3
Python: 3.12.3
```

The current playbook has completed successfully with:

```text
ok=69
changed=0
failed=0
```

Final validation on a clean Ubuntu 24.04 LTS environment should be performed before submission.

## Reference

CIS Ubuntu Linux 24.04 LTS Benchmark v2.0.0.

Official CIS Benchmark:

https://www.cisecurity.org/benchmark/ubuntu_linux
