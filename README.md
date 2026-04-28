# SysAdmin Lab

A personal systems administration and security monitoring lab, documenting the build of an enterprise-grade Linux and Windows infrastructure environment. The lab is maintained alongside active study for **RHCSA certification** (RHEL 10, late July 2026), and as a hands-on complement to my work as a Technical Analyst.

## Goals

- Practice fundamental Linux systems administration against a real, hardened RHEL-compatible environment rather than solely through coursework.
- Build the multi-host, multi-service environment that underpins real enterprise infrastructure: domain services, centralised logging, configuration management, automation, and infrastructure-as-code.
- Develop expertise in enterprise infrastructure specialisation (RHEL-based systems, cloud engineering, DevOps) rather than general-purpose Linux skills.
- Document decisions, failures, and learnings honestly as I go, so the repo stays a working record rather than a polished summary.

## Career direction

**Enterprise infrastructure focus:** RHEL-based systems administration, cloud engineering, infrastructure automation, and DevOps. Long-term trajectory toward cloud-native infrastructure roles in enterprise environments.

**Certification path:** RHCSA (RHEL 10) → future RHCE and advanced Red Hat certifications.

**Lab distro choice:** Rocky Linux deliberately chosen over general-purpose distributions (Ubuntu, Debian) for 1:1 binary compatibility with RHEL and enterprise credibility. Every command and concept learned transfers directly to production RHEL environments and enterprise roles.

## Current state

Phase 1 complete. Phase 2 complete. Rocky Linux 9.7 server hardened and running; Windows Server 2022 promoted to Domain Controller with Active Directory configured.

### Host environment

- Windows 11 host
- VMware Workstation Pro (personal-use licence)
- Git Bash for terminal work and Git operations
- VS Code with Git integration for documentation and code

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
- **Status: Active Directory Domain Controller for `lab.local` domain**
- Snapshots: `post-install-clean`, `phase2-02-ad-configured-with-ous-and-users`

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

## Phase 2 progress — COMPLETE ✅

### Windows Server 2022 Domain Controller (`win-dc-01`)

**Configuration completed:**
- Active Directory Domain Services (AD DS) installed and promoted to Domain Controller
- Domain created: `lab.local` (Forest: Windows Server 2016 functional level)
- DNS Server installed and configured
- Active Directory Users and Computers configured

**Organizational Units (OUs) created:**
- `IT-Users` — for domain user accounts
- `IT-Computers` — for joined computers
- `IT-Service-Accounts` — for service accounts

**Test domain users created:**
- `testuser` (in IT-Users OU)
- `labadmin` (in IT-Users OU)

**Snapshots taken:**
- `post-install-clean` (after Windows Server installation)
- `phase2-02-ad-configured-with-ous-and-users` (after DC promotion and OU/user configuration)

### Next steps

- Join `rocky-lab-01` to the domain using SSSD/realmd (RHCSA exam objective)
- Configure Group Policy on the DC
- Set up Windows Event Forwarding for centralized logging

## Lab roadmap (9 phases)

### Phases 1–5: Traditional enterprise infrastructure

**Phase 1: Rocky Linux hardening** ✅ COMPLETE
- Base OS hardening: SSH key-based auth, firewall configuration, fail2ban
- Foundation for all subsequent work

**Phase 2: Windows Server domain controller** ✅ COMPLETE
- Domain Controller promotion: `lab.local` domain created
- Active Directory configured with OUs and test users
- DNS Server installed and operational
- Next: Join Rocky Linux to domain, configure Group Policy, set up Event Forwarding

**Phase 3: Centralised logging with Wazuh SIEM** (next)
- Deploy Wazuh on dedicated VM or upgrade `rocky-lab-01` resources
- Configure Windows Event Forwarding from the DC and local agent on Rocky
- Aggregate, visualise, and alert on events from both Linux and Windows
- Goal: build SIEM investigation muscle and security monitoring foundation
- Estimated: 2–3 weeks

**Phase 4: Configuration automation with Ansible**
- Introduce Ansible for repeatable, idempotent configuration
- Retrofit Phase 1 hardening steps into automated playbooks
- Explore PowerShell DSC for Windows automation if appropriate
- Goal: infrastructure-as-code discipline
- Estimated: 2–3 weeks

**Phase 5: Detection engineering**
- Tune MITRE ATT&CK-aligned detection rules in Wazuh
- Simulate adversarial activity using Atomic Red Team
- Refine detection logic based on observed events
- Goal: hands-on security operations and incident response
- Estimated: 2–3 weeks

### Phases 6–9: Modern DevOps and cloud infrastructure

**Phase 6: Terraform Infrastructure-as-Code**
- Define VMware infrastructure (VMs, networking, snapshots) as code
- Deploy new VMs programmatically instead of manual GUI configuration
- Learn IaC principles and state management
- Estimated: 3–4 weeks

**Phase 7: Docker and containers**
- Deploy Docker on Rocky Linux VMs
- Run containerised services (SIEM components, monitoring tools, applications)
- Explore lightweight Kubernetes (k3s) for container orchestration
- Estimated: 2–3 weeks

**Phase 8: CI/CD with GitHub Actions**
- Set up GitHub Actions to trigger automated testing and deployment on commit
- Automate Ansible playbook validation and infrastructure testing
- Implement automated rollout of changes to lab VMs
- Gain hands-on DevOps and continuous integration experience
- Estimated: 2 weeks

**Phase 9: Cloud migration and cloud-native extensions**
- Migrate lab infrastructure to AWS/Azure/GCP (or build cloud-native components)
- Explore managed services and cloud-native architectures
- Compare on-premises vs cloud costs, complexity, and operations
- Ongoing and iterative

### Total estimated timeline

- **Phases 1–5 (traditional infrastructure):** ~14–16 weeks (Phase 2 complete; Phase 3–5 ongoing)
- **RHCSA exam prep and study:** June–late July 2026
- **Phases 6–9 (DevOps/cloud):** Post-RHCSA, August 2026 onwards

## RHCSA certification alignment

Every phase of this lab directly supports RHCSA exam preparation:

| Phase | RHCSA alignment |
|-------|-----------------|
| Phase 1 | SSH hardening, firewall (`firewalld`), fail2ban, systemd services |
| Phase 2 | Domain integration with SSSD/realmd, user/group authentication, AD basics |
| Phase 3 | Log management and analysis |
| Phase 4 | Configuration management concepts |
| Phase 5 | Security monitoring and incident response |

**Study resources:**
- van Vugt's RHCSA RHEL 10 Study Guide (with 60% discount code: SANDER60)
- Red Hat Free Developer Subscription (RHEL 10 media and documentation)
- Lab environment (hands-on practice of exam objectives)
- Official exam objectives (download from Red Hat)

**Exam details:**
- Exam: RHCSA EX200 (RHEL 10)
- Format: Performance-based, 120 minutes
- Target date: late July 2026

## Key learnings so far

- The SSH server on Rocky 9.7 does not yet negotiate post-quantum key exchange; modern OpenSSH clients warn about "store-now-decrypt-later" exposure. Not a concern for an isolated lab, but worth being aware of.
- The ordering discipline for SSH hardening matters: deploy and test the new authentication method *before* disabling the old one, and always validate config with `sshd -t` before restarting the daemon.
- `firewalld`'s service-based model (e.g. `services: ssh`) is easier to reason about and safer than raw port numbers; using named services is a small but real defence against misconfiguration.
- Rocky Linux is the correct choice for enterprise infrastructure study: 1:1 binary compatible with RHEL, giving credibility for both RHCSA exam prep and future enterprise roles.
- Active Directory promotion workflow: understand the distinction between forest/domain/OUs. Creating a new forest with a single domain is appropriate for lab/test environments.
- Group Policy and centralized user management are foundational to enterprise infrastructure. Test OUs and users should be created early to support future lab phases (SIEM, automation, detection).

## Log

- **2026-04-19** — Initial Rocky Linux 9.7 install on VMware Workstation Pro. Baseline hardening: key-based SSH auth deployed (Ed25519), password and root login disabled via `sshd_config`, config validated with `sshd -t` before restart, hardening verified by attempting password-only connection. `open-vm-tools` installed for guest integration. `firewalld` default state documented. Repo initialised; `.gitignore` configured to keep local screenshots and notes out of version control.
- **2026-04-21** — Added fail2ban for SSH brute-force protection. Installed fail2ban from EPEL repository, configured sshd jail with 1-hour ban duration and 5-attempt threshold. Service enabled and verified with `fail2ban-client status`. This hardens against dictionary and brute-force attacks on SSH before proceeding to Phase 2.
- **2026-04-21** — Phase 2 kickoff: Created Windows Server 2022 VM (`win-dc-01`, 4 GB RAM, 2 vCPU, 60 GB disk, NAT networking). Downloaded Windows Server 2022 Standard Evaluation ISO, performed clean installation with Desktop Experience, and verified successful boot to login screen. Took clean snapshot (`post-install-clean`) as checkpoint before AD DS configuration. Next: promote to Domain Controller, configure DNS and Group Policy, then integrate `rocky-lab-01` into the domain.
- **2026-04-28** — Phase 2 completion: Promoted `win-dc-01` to Active Directory Domain Controller for `lab.local` domain. Installed AD DS and DNS roles; configured forest (Windows Server 2016 functional level) and domain. Created Organizational Units (IT-Users, IT-Computers, IT-Service-Accounts) for structured AD management. Created test domain users (`testuser`, `labadmin`) in IT-Users OU. Took snapshot (`phase2-02-ad-configured-with-ous-and-users`) as checkpoint. Next: join `rocky-lab-01` to domain using SSSD/realmd, configure Group Policy, set up Windows Event Forwarding.