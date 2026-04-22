# SysAdmin Lab

A personal systems administration and security monitoring lab, documenting the build of a Linux and Windows infrastructure environment. The lab is maintained alongside active study for CCNA (July 2026), and as a hands-on complement to my work as a Technical Analyst.

## Goals

- Practice fundamental Linux systems administration against a real, hardened environment rather than solely through coursework.
- Build the multi-host, multi-service environment that underpins real enterprise infrastructure: domain services, centralised logging, configuration management, and automation.
- Document decisions, failures, and learnings honestly as I go, so the repo stays a working record rather than a polished summary.

## Current state

Phase 1 complete. Phase 2 in progress: Rocky Linux 9.7 server hardened and running; Windows Server 2022 VM created and installed, awaiting promotion to Domain Controller.

### Host environment

- Windows 11 host
- VMware Workstation Pro (personal-use licence)
- Git Bash for terminal work and Git operations

### Guest VMs

#### Rocky Linux 9.7 (`rocky-lab-01`)
- Server profile (no desktop)
- 4 GB RAM, 2 vCPU, 40 GB disk
- NAT networking (192.168.126.0/24), private to host
- Hostname: `rocky-lab-01`
- IP: 192.168.126.130
- Non-privileged administrative user with `wheel` group membership for `sudo`

#### Windows Server 2022 (`win-dc-01`)
- Standard Evaluation with Desktop Experience
- 4 GB RAM, 2 vCPU, 60 GB disk
- NAT networking (192.168.126.0/24), private to host
- Status: Clean installation complete, awaiting AD DS promotion

### Hardening applied (Rocky Linux)

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

### fail2ban (SSH brute-force protection)

- Installed: `fail2ban` from EPEL repository
- Configuration: `/etc/fail2ban/jail.local`
  - Ban duration: 1 hour (3600 seconds)
  - Detection window: 10 minutes (600 seconds)
  - Ban threshold: 5 failed attempts
  - Monitored: SSH (port 22)
- Service enabled and started: `systemctl enable fail2ban` and `systemctl start fail2ban`
- Status verified: `sudo fail2ban-client status sshd` shows 0 currently banned IPs
- Hardening verified by attempting multiple failed SSH logins; failed attempts logged and monitored

## Phase 2 progress

### Windows Server 2022 Domain Controller (`win-dc-01`)

- **VM specification:** 4 GB RAM, 2 vCPU, 60 GB disk, NAT networking (192.168.126.0/24)
- **OS:** Windows Server 2022 Standard Evaluation with Desktop Experience
- **Status:** Clean installation complete, awaiting AD DS promotion
- **Snapshot taken:** `post-install-clean` (checkpoint before DC configuration)
- **Next steps:** Install AD DS and DNS roles, create domain and OU structure, configure Group Policy and Windows Event Forwarding

### Integration plan

- Configure DNS on `win-dc-01` to resolve `rocky-lab-01.local`
- Join `rocky-lab-01` to the domain using SSSD or `realmd`
- Test cross-platform authentication (Linux client authenticating to Windows AD)

## Planned next phases

1. **Windows Server domain controller (Phase 2 — in progress).** Promote `win-dc-01` to Active Directory Domain Controller, install DNS, configure Group Policy, and Windows Event Forwarding. Join `rocky-lab-01` to the domain for cross-platform authentication and centralised management.

2. **Centralised logging with Wazuh SIEM (Phase 3).** Deploy Wazuh on a dedicated VM or upgrade `rocky-lab-01` resources. Configure Windows Event Forwarding from the DC and local agent on Rocky to feed into centralised SIEM. Goal: aggregate, visualise, and alert on events from both Linux and Windows.

3. **Configuration automation with Ansible (Phase 4).** Introduce Ansible for repeatable, idempotent configuration of both hosts. Retrofit Phase 1 hardening steps into playbooks. Explore PowerShell DSC for Windows automation if appropriate.

4. **Detection engineering (Phase 5).** Tune MITRE ATT&CK-aligned detection rules in Wazuh. Simulate adversarial activity using Atomic Red Team and refine detection logic based on observed events.

## Key learnings so far

- The SSH server on Rocky 9.7 does not yet negotiate post-quantum key exchange; modern OpenSSH clients warn about "store-now-decrypt-later" exposure. Not a concern for an isolated lab, but worth being aware of.
- The ordering discipline for SSH hardening matters: deploy and test the new authentication method *before* disabling the old one, and always validate config with `sshd -t` before restarting the daemon.
- `firewalld`'s service-based model (e.g. `services: ssh`) is easier to reason about and safer than raw port numbers; using named services is a small but real defence against misconfiguration.

## Log

- **2026-04-19** — Initial Rocky Linux 9.7 install on VMware Workstation Pro. Baseline hardening: key-based SSH auth deployed (Ed25519), password and root login disabled via `sshd_config`, config validated with `sshd -t` before restart, hardening verified by attempting password-only connection. `open-vm-tools` installed for guest integration. `firewalld` default state documented. Repo initialised; `.gitignore` configured to keep local screenshots and notes out of version control.
- **2026-04-21** — Added fail2ban for SSH brute-force protection. Installed fail2ban from EPEL repository, configured sshd jail with 1-hour ban duration and 5-attempt threshold. Service enabled and verified with `fail2ban-client status`. This hardens against dictionary and brute-force attacks on SSH before proceeding to Phase 2.
- **2026-04-21** — Phase 2 kickoff: Created Windows Server 2022 VM (`win-dc-01`, 4 GB RAM, 2 vCPU, 60 GB disk, NAT networking). Downloaded Windows Server 2022 Standard Evaluation ISO, performed clean installation with Desktop Experience, and verified successful boot to login screen. Took clean snapshot (`post-install-clean`) as checkpoint before AD DS configuration. Next: promote to Domain Controller, configure DNS and Group Policy, then integrate `rocky-lab-01` into the domain.