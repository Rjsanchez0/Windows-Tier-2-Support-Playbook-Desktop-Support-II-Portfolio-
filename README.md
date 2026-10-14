# Help Desk Tier 2 Playbook (Desktop Support II Portfolio)

## What this is

A hands-on Tier 2/Desktop Support and junior Windows administration portfolio built in a VirtualBox lab environment.

Each project is documented using a real-world IT support workflow:

**symptoms → environment → triage → troubleshooting → resolution → verification → escalation criteria**

The portfolio includes Windows troubleshooting, Active Directory administration, PowerShell automation, DNS remediation, Group Policy management, workstation hardening, Microsoft 365 support, VPN troubleshooting, and access-control scenarios.

## Lab Environment

- VirtualBox lab with **Domain Controller (AD DS/DNS)**, **2 Windows workstations**, and **Ubuntu**
- Internal network (“LABNET”) with optional NAT for internet

## Skills Demonstrated

- Windows 10/11 troubleshooting (drivers, updates, performance, permissions)
- Active Directory administration (account lockouts, user lifecycle management, OUs, security groups)
- Active Directory DNS and domain connectivity troubleshooting
- PowerShell automation for Active Directory onboarding and offboarding
- Group Policy administration and workstation hardening
- Identity lifecycle management and access revocation
- Group-based access control and least privilege
- Printer/spooler troubleshooting and printer deployment by IP
- DNS/DHCP triage using `ipconfig`, `ping`, and `nslookup`
- VPN troubleshooting using RasClient logs and evidence collection
- Outlook/Microsoft 365 sign-in troubleshooting
- Windows file-share and NTFS permission troubleshooting
- Ticket documentation, validation, and escalation notes

## Start Here — Tier 2 / Windows Administration Projects

### Windows / Desktop Support

- **T2-001** Printer queue stuck: [docs/tickets/T2_001_Printer_que_stuck.md](docs/tickets/T2_001_Printer_que_stuck.md)
- **T2-002** PowerShell Help Desk Toolkit: [docs/tickets/T2-002_PowerShell_HelpDesk_Toolkit.md](docs/tickets/T2-002_PowerShell_HelpDesk_Toolkit.md)
- **T2-003** VPN issues (triage): [docs/tickets/T2-003_VPN_issues.md](docs/tickets/T2-003_VPN_issues.md)
- **T2-004** Outlook/M365 sign-in loop: [docs/tickets/T2-004_Outlook_M365_SignIn_Loop](docs/tickets/T2-004_Outlook_M365_SignIn_Loop.md)
- **T2-005** No internet (DNS/DHCP): [docs/tickets/T2-005_No_Internet_DNS_DHCP](docs/tickets/T2-005_No_Internet_DNS_DHCP.md)
- **T2-006** Windows slow triage: [docs/tickets/T2_006_windows_slow.md](docs/tickets/T2_006_windows_slow.md)

### Active Directory / Windows Administration

- **AD-001** Account lockout: [docs/ad/AD-001_Account_Lockout.md](docs/ad/AD-001_Account_Lockout.md)
- **AD-002** User lifecycle management — onboarding & offboarding: [AD-002-Active-Directory-User-Lifecycle-Management](https://github.com/Rjsanchez0/AD-002-Active-Directory-User-Lifecycle-Management)
- **AD-003** Share access denied: [docs/ad/AD-003_Share_Access_Denied.md](docs/ad/AD-003_Share_Access_Denied.md)
- **AD-004** Active Directory DNS remediation: [docs/ad/AD-004_AD_DNS_Remediation.md](docs/ad/AD-004_AD_DNS_Remediation.md)
- **AD-005** Active Directory PowerShell automation: [docs/ad/AD-005_Active_Directory_PowerShell_Automation.md](docs/ad/AD-005_Active_Directory_PowerShell_Automation.md)
- **AD-006** Group Policy workstation hardening: [docs/ad/AD-006%20%E2%80%94%20Group%20Policy%20Workstation%20Hardening.md](docs/ad/AD-006%20%E2%80%94%20Group%20Policy%20Workstation%20Hardening.md)
