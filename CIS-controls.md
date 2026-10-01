# CIS Ubuntu Linux 24.04 LTS v2.0.0 - Selected Controls

This project implements a selected subset of the CIS Ubuntu Linux 24.04
LTS Benchmark v2.0.0.

The goal is not full CIS compliance. The selected controls focus on
system hardening, service reduction, firewall configuration, SSH
hardening, privilege escalation, and critical file permissions.

## Selected Control Matrix

| CIS ID | Recommendation | Implementation | Status |
|---|---|---|---|
| 1.5.1 | Ensure fs.protected_hardlinks is configured | tasks/network.yml | Implemented and verified |
| 1.5.2 | Ensure fs.protected_symlinks is configured | tasks/network.yml | Implemented and verified |
| 1.5.3 | Ensure kernel.yama.ptrace_scope is configured | tasks/network.yml | Implemented and verified |
| 1.5.4 | Ensure fs.suid_dumpable is configured | tasks/network.yml | Implemented and verified |
| 1.5.5 | Ensure kernel.dmesg_restrict is configured | tasks/network.yml | Implemented and verified |
| 2.1.3 | Ensure avahi daemon services are not in use | tasks/services.yml | Implemented and verified |
| 4.1.1 | Ensure ufw is installed | tasks/packages.yml | Implemented and verified |
| 4.1.2 | Ensure ufw service is configured | tasks/firewall.yml | Implemented and verified |
| 4.1.3 | Ensure ufw incoming default is configured | tasks/firewall.yml | Implemented and verified |
| 4.1.4 | Ensure ufw outgoing default is configured | tasks/firewall.yml | Implemented and verified |
| 4.1.5 | Ensure ufw routed default is configured | System verification | Verified |
| 5.1.7 | Ensure sshd ClientAliveInterval and ClientAliveCountMax are configured | tasks/ssh.yml | Implemented and verified |
| 5.1.13 | Ensure sshd LoginGraceTime is configured | tasks/ssh.yml | Implemented and verified |
| 5.1.16 | Ensure sshd MaxAuthTries is configured | tasks/ssh.yml | Implemented and verified |
| 5.1.19 | Ensure sshd PermitEmptyPasswords is disabled | tasks/ssh.yml | Implemented and verified |
| 5.1.20 | Ensure sshd PermitRootLogin is disabled | tasks/ssh.yml | Implemented and verified |
| 5.1.21 | Ensure sshd PermitUserEnvironment is disabled | tasks/ssh.yml | Implemented and verified |
| 5.1.22 | Ensure sshd UsePAM is enabled | tasks/ssh.yml | Implemented and verified |
| 5.2.1 | Ensure sudo is installed | tasks/packages.yml | Implemented and verified |
| 5.2.2 | Ensure sudo commands use pty | tasks/additional_hardening.yml | Implemented and verified |
| 7.1.1 | Ensure access to /etc/passwd is configured | tasks/filesystem.yml | Implemented and verified |
| 7.1.3 | Ensure access to /etc/group is configured | tasks/filesystem.yml | Implemented and verified |
| 7.1.5 | Ensure access to /etc/shadow is configured | tasks/filesystem.yml | Implemented and verified |
| 7.1.7 | Ensure access to /etc/gshadow is configured | tasks/filesystem.yml | Implemented and verified |
| 7.1.9 | Ensure access to /etc/shells is configured | tasks/filesystem.yml | Implemented and verified |
| 7.1.11 | Ensure world-writable files and directories are secured | tasks/filesystem.yml | Implemented and verified |
| 7.1.12 | Ensure no files or directories without an owner and a group exist | tasks/filesystem.yml | Implemented and verified |

## Control Rationale

### 1.5 Additional Process Hardening

The selected kernel parameters reduce opportunities for privilege
escalation, information disclosure, and exploitation of unsafe process
or file-link behavior.

The configured values are:

- fs.protected_hardlinks = 1
- fs.protected_symlinks = 1
- kernel.yama.ptrace_scope = 1
- fs.suid_dumpable = 0
- kernel.dmesg_restrict = 1

These settings are implemented through Ansible sysctl management and
verified during playbook execution.

### 2.1 Service Management

Unnecessary services increase the attack surface of a workstation.

The playbook disables services that are not required for the intended
cybersecurity workstation use case. The current configuration disables:

- avahi-daemon
- cups
- gnome-remote-desktop

The playbook also verifies that these services are both inactive and
disabled.

### 4.1 UFW

UFW provides the host-based firewall baseline.

The configuration uses:

- deny incoming by default
- allow outgoing by default
- disabled routed traffic by default
- explicit SSH access on the configured SSH port

The current system verification confirms that the routed policy is
disabled.

### 5.1 SSH

SSH is enabled for secure remote administration.

The playbook configures:

- PermitRootLogin no
- PermitEmptyPasswords no
- PermitUserEnvironment no
- UsePAM yes
- MaxAuthTries 4
- LoginGraceTime 60
- ClientAliveInterval 300
- ClientAliveCountMax 2

The SSH configuration is validated with sshd -t and the effective
configuration is checked with sshd -T.

Password authentication remains enabled because public-key deployment
and SSH key lifecycle management have not yet been implemented.
Disabling password authentication without a verified alternative could
cause administrative lockout.

### 5.2 Privilege Escalation

Sudo is required for administrative operations.

The project ensures that sudo is installed and configures sudo commands
to use a pseudo-terminal.

### 7.1 System File Permissions

Critical system account and configuration files must not be writable by
unprivileged users.

The playbook configures ownership and permissions for:

- /etc/passwd
- /etc/group
- /etc/shadow
- /etc/gshadow
- /etc/shells

It also checks for world-writable files and files or directories without
a valid owner or group.

## Operational Trade-offs

Security hardening can affect usability and compatibility.

Examples include:

- disabling unnecessary services may remove functionality useful on a
  general-purpose workstation
- default-deny firewall policies require explicit rules for services
- SSH restrictions may prevent legacy clients from connecting
- kernel hardening may affect specialized debugging or testing workflows
- password authentication remains enabled until key-based SSH management
  is implemented

The project therefore implements a selected practical security baseline
rather than claiming complete CIS compliance.

## Validation

The project currently validates:

1. Ansible syntax
2. Playbook execution on Ubuntu 24.04 LTS
3. Idempotency through repeated playbook execution
4. SSH configuration syntax
5. Effective SSH security settings
6. UFW status and default policies
7. IPv4 and IPv6 forwarding
8. Kernel hardening parameters
9. auditd status
10. Unnecessary service states
11. Critical file permissions
12. World-writable files
13. Unowned or ungrouped files

A final validation on a fresh Ubuntu 24.04 LTS VM will be performed
before submission.

## Reference

CIS Ubuntu Linux 24.04 LTS Benchmark v2.0.0.

Official CIS Benchmark:
https://www.cisecurity.org/benchmark/ubuntu_linux

