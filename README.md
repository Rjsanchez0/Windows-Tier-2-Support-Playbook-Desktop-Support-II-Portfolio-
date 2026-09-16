# Help Desk Tier 2 Playbook (Desktop Support II Portfolio)

## What this is
A Tier 2/Desktop Support portfolio built from hands-on labs (VirtualBox). Each ticket is written like real ITSM work: **symptoms → environment → triage → troubleshooting → resolution → verification → escalation criteria**.

## Lab Environment
- VirtualBox lab with **Domain Controller (AD DS/DNS)**, **2 Windows workstations**, and **Ubuntu**
- Internal network (“LABNET”) with optional NAT for internet

## Skills Demonstrated
- Windows 10/11 troubleshooting (drivers, updates, performance, permissions)
- Active Directory workflows (lockouts, domain/DNS issues, group-based share permissions)
- Printer/spooler troubleshooting + add printer by IP (lab simulation)
- DNS/DHCP triage (ipconfig, ping, nslookup)
- VPN triage (RasClient logs + evidence packet)
- Outlook/Microsoft 365 sign-in triage (profiles/credentials + escalation packet)
- Ticket documentation and escalation notes

## Start Here (Tier 2 Ticket Writeups)

### Windows / Desktop Support
- **T2-001** Printer queue stuck: [docs/tickets/T2_001_Printer_que_stuck.md](docs/tickets/T2_001_Printer_que_stuck.md)
- **T2-002** PowerShell Help Desk Toolkit: [docs/tickets/T2-002_PowerShell_HelpDesk_Toolkit.md](docs/tickets/T2-002_PowerShell_HelpDesk_Toolkit.md)
- **T2-003** VPN issues (triage): [docs/tickets/T2-003_VPN_issues.md](docs/tickets/T2-003_VPN_issues.md)
- **T2-004** Outlook/M365 sign-in loop: [docs/tickets/T2-004_Outlook_M365_SignIn_Loop](docs/tickets/T2-004_Outlook_M365_SignIn_Loop)
- **T2-005** No internet (DNS/DHCP): [docs/tickets/T2-005_No_Internet_DNS_DHCP](docs/tickets/T2-005_No_Internet_DNS_DHCP)
- **T2-006** Windows slow triage: [docs/tickets/T2_006_windows_slow.md](docs/tickets/T2_006_windows_slow.md)

### Active Directory Ticket Scenarios
- **AD-001** Account lockout: [docs/ad/AD-001_Account_Lockout.md](docs/ad/AD-001_Account_Lockout.md)
- **AD-003** Share access denied: [docs/ad/AD-003_Share_Access_Denied.md](docs/ad/AD-003_Share_Access_Denied.md)
