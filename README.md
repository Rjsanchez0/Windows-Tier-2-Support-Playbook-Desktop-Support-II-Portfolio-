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
- **T2-001** Printer queue stuck: [tickets/T2_001_Printer_Queue_Stuck.md](tickets/T2_001_Printer_Queue_Stuck.md)
- **T2-002** PowerShell Help Desk Toolkit: [tickets/T2_002_PowerShell_HelpDesk_Toolkit.md](tickets/T2_002_PowerShell_HelpDesk_Toolkit.md)
- **T2-003** VPN can't connect (triage): [tickets/T2_003_VPN_Cant_Connect_Triage.md](tickets/T2_003_VPN_Cant_Connect_Triage.md)
- **T2-004** Outlook/M365 sign-in loop: [tickets/T2_004_Outlook_M365_SignIn_Loop.md](tickets/T2_004_Outlook_M365_SignIn_Loop.md)
- **T2-005** No internet (DNS/DHCP): [tickets/T2_005_No_Internet_DNS_DHCP.md](tickets/T2_005_No_Internet_DNS_DHCP.md)
- **T2-006** Windows slow triage: [tickets/T2_006_Windows_Slow_Triage.md](tickets/T2_006_Windows_Slow_Triage.md)

### Active Directory Ticket Scenarios
- **AD-001** Account lockout: [docs/ad/AD-001_Account_Lockout.md](docs/ad/AD-001_Account_Lockout.md)
- **AD-003** Share access denied: [docs/ad/AD-003_Share_Access_Denied.md](docs/ad/AD-003_Share_Access_Denied.md)
