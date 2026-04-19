# Sysadmin Lab

A personal systems administration and security monitoring lab, documenting the build of a Linux and Windows infrastructure environment. The lab is maintained alongside active study for CCNA (July 2026), and as a hands-on complement to my work as a Technical Analyst at the Australian Signals Directorate.

## Goals

- Practice fundamental Linux systems administration against a real, hardened environment rather than solely through coursework.
- Build the multi-host, multi-service environment that underpins real enterprise infrastructure: domain services, centralised logging, configuration management, and automation.
- Document decisions, failures, and learnings honestly as I go, so the repo stays a working record rather than a polished summary.

## Current state

A single Rocky Linux 9.7 server VM running on VMware Workstation Pro, hardened as the foundation for further work.

### Host environment

- Windows 11 host
- VMware Workstation Pro (personal-use licence)
- Git Bash for terminal work and Git operations

### Guest (VM) configuration

- Rocky Linux 9.7 (Blue Onyx), Server profile (no desktop)
- 4 GB RAM, 2 vCPU, 40 GB disk
- NAT networking (192.168.126.0/24), private to host
- Hostname: `rocky-lab-01`
- Non-privileged administrative user with `wheel` group membership for `sudo`

### Hardening applied

- Full system update via `dnf update` immediately after install
- `open-vm-tools` installed for guest integration
- SSH key-based authentication configured: Ed25519 key pair generated on Windows host (passphrase-protected), public key deployed to VM via `ssh-copy-id`
- SSH server configuration (`/etc/ssh/sshd_config`) changes: `PasswordAuthentication no`, `PubkeyAuthentication yes`, `PermitRootLogin no`
- Config validated with `sshd -t` prior to applying
- Hardening verified by attempting password-only connection; server refused with `Permission denied (publickey)`, confirming password auth is no longer offered

### Firewall baseline

- `firewalld` active by default in the `public` zone
- Permitted services: `ssh`, `cockpit`, `dhcpv6-client`
- No manual port rules or rich rules yet
- Future work: remove `cockpit` from allowed services; tighten SSH access to specific source IPs

## Planned next phases

1. **Centralised logging.** Deploy a log aggregator (Wazuh SIEM or a minimal Elastic stack) on the VM, configure local agent, generate events and observe them landing in a dashboard. Goal: build the muscle for SIEM investigation in a small, safe environment.
2. **Windows Server domain controller.** Add a Windows Server 2022 VM, configure Active Directory Domain Services, DNS, Group Policy, and Windows Event Forwarding to the log aggregator.
3. **Configuration automation.** Introduce Ansible or PowerShell DSC for repeatable configuration of both hosts. Retrofit the hardening steps from phase one into automated playbooks.
4. **Detection engineering.** Tune MITRE ATT&CK-aligned detection rules against simulated adversarial activity (Atomic Red Team).

## Key learnings so far

- The SSH server on Rocky 9.7 does not yet negotiate post-quantum key exchange; modern OpenSSH clients warn about "store-now-decrypt-later" exposure. Not a concern for an isolated lab, but worth being aware of.
- The ordering discipline for SSH hardening matters: deploy and test the new authentication method *before* disabling the old one, and always validate config with `sshd -t` before restarting the daemon.
- `firewalld`'s service-based model (e.g. `services: ssh`) is easier to reason about and safer than raw port numbers; using named services is a small but real defence against misconfiguration.

## Log

- **2026-04-19** — Initial Rocky 9.7 install. SSH hardening applied and verified. Lab foundation in place.
