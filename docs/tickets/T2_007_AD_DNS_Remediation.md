# T2-007 — Active Directory DNS Remediation

## Ticket Scenario

**User Impact:** A domain-joined Windows workstation may experience authentication issues, Group Policy failures, slow logins, or problems accessing internal resources because it is configured to use an external DNS server instead of the organization's internal Active Directory DNS server.

**Priority:** Medium  
**Category:** Desktop Support → Active Directory / DNS Troubleshooting

---

## Objective

This project demonstrates a Tier 2 troubleshooting workflow for identifying and correcting DNS misconfiguration on a domain-joined Windows workstation.

The objectives were to:

1. Verify workstation domain membership
2. Review the workstation network configuration
3. Identify the incorrect DNS configuration
4. Verify the Active Directory domain controller and DNS service
5. Correct the workstation DNS settings
6. Validate domain name resolution
7. Confirm domain controller discovery
8. Verify Group Policy communication

---

## Lab Environment

- Windows Server Domain Controller
- Windows 11 domain-joined workstation
- Active Directory Domain Services
- Active Directory-integrated DNS
- Group Policy
- PowerShell
- VirtualBox isolated lab environment

> Environment-specific identifiers such as usernames, hostnames, domain names, IP addresses, MAC addresses, SIDs, GUIDs, and other unique system information have been generalized for public documentation.

---

## Issue Summary

During review of a domain-joined workstation, I discovered that the workstation was configured to use a public DNS server instead of the internal DNS server provided by the Active Directory environment.

The workstation was successfully joined to the domain, but using external DNS could cause inconsistent communication with Active Directory services.

Possible symptoms include:

- Slow or failed domain logins
- Group Policy update failures
- Problems locating a domain controller
- Authentication delays
- Problems accessing shared resources
- Issues with domain-based applications

---

## Technical Background

Active Directory relies heavily on DNS.

Domain-joined Windows systems use DNS to locate services such as:

- Domain controllers
- Kerberos authentication services
- LDAP directory services
- Group Policy resources
- File servers
- Internal applications

A public DNS server can resolve internet resources, but it does not contain the internal Active Directory DNS records required for reliable domain operations.

Domain-joined workstations should therefore use the organization's internal DNS infrastructure.

---

## Triage Questions

Before making changes, I would confirm:

- Is the issue affecting one workstation or multiple systems?
- Is the computer successfully joined to the domain?
- Can the user authenticate with domain credentials?
- Can the workstation reach the domain controller?
- What DNS server is the workstation using?
- Can the workstation resolve the internal domain?
- Can the workstation locate a domain controller?
- Is Group Policy applying successfully?
- Were any recent network settings changed?

---

## Walkthrough

### Step 1 — Verify Domain Membership

I first confirmed the current user, workstation hostname, and Active Directory domain membership.

```powershell
whoami
hostname
Get-ComputerInfo | Select-Object CsDomain, CsName
```

### Sanitized Result

```text
User: <DOMAIN>\<USER>
Hostname: <WORKSTATION>
Domain: <LAB_DOMAIN>
```

The workstation was successfully joined to the Active Directory domain.

---

### Step 2 — Review Network Configuration

I reviewed the workstation network configuration.

```powershell
ipconfig /all
```

The output showed that the workstation was using an external/public DNS server rather than the internal Active Directory DNS server.

### Sanitized Finding

```text
IPv4 Address: <WORKSTATION_IP>
Default Gateway: <DEFAULT_GATEWAY>
DNS Server: <EXTERNAL_DNS_SERVER>
```

---

### Step 3 — Verify the Domain Controller

On the Windows Server domain controller, I verified the system hostname, domain membership, and network configuration.

```powershell
hostname
Get-ComputerInfo | Select-Object CsDomain, CsName
ipconfig /all
```

### Sanitized Result

```text
Hostname: <DOMAIN_CONTROLLER>
Domain: <LAB_DOMAIN>
IPv4 Address: <INTERNAL_DNS_IP>
```

This confirmed the internal DNS server address that the workstation should use.

---

### Step 4 — Verify Active Directory

I verified that the domain controller was providing Active Directory services.

```powershell
Get-ADDomain
```

### Sanitized Result

```text
Domain: <LAB_DOMAIN>
Domain Controller: <DOMAIN_CONTROLLER>
Forest: <LAB_DOMAIN>
```

---

### Step 5 — Verify Active Directory DNS

I verified that the domain controller contained the expected Active Directory-integrated DNS zones.

```powershell
Get-DnsServerZone
```

The expected domain DNS zones were present and configured as Active Directory-integrated zones.

This confirmed that the domain controller was the correct DNS server for domain clients.

---

### Step 6 — Correct the Workstation DNS Configuration

I changed the workstation DNS configuration so that the Ethernet adapter used the internal Active Directory DNS server.

```powershell
Set-DnsClientServerAddress `
    -InterfaceAlias "Ethernet" `
    -ServerAddresses "<INTERNAL_DNS_IP>"
```

---

### Step 7 — Verify the New DNS Configuration

I checked the workstation configuration again.

```powershell
ipconfig /all
```

### Sanitized Result

```text
DNS Server: <INTERNAL_DNS_IP>
```

The workstation was now using the internal DNS infrastructure.

---

### Step 8 — Test Internal DNS Resolution

I tested whether the workstation could resolve the Active Directory domain.

```powershell
nslookup <LAB_DOMAIN>
```

### Sanitized Result

```text
Name: <LAB_DOMAIN>
Address: <INTERNAL_DNS_IP>
```

The workstation successfully resolved the internal domain.

---

### Step 9 — Verify Domain Controller Discovery

I verified that the workstation could locate a domain controller.

```powershell
nltest /dsgetdc:<LAB_DOMAIN>
```

### Sanitized Result

```text
DC: \\<DOMAIN_CONTROLLER>
Domain Name: <LAB_DOMAIN>
The command completed successfully.
```

This confirmed successful domain controller discovery.

---

### Step 10 — Validate Group Policy

I forced a Group Policy refresh.

```powershell
gpupdate /force
```

### Result

```text
Updating policy...

Computer Policy update has completed successfully.
User Policy update has completed successfully.
```

This confirmed that the workstation could communicate successfully with Active Directory after the DNS configuration was corrected.

---

## Root Cause

The workstation had been configured to use an external DNS server instead of the internal Active Directory DNS server.

Because public DNS infrastructure does not contain the internal service records used by Active Directory, the configuration could cause unreliable authentication, Group Policy processing, and domain resource access.

---

## Resolution

The workstation DNS configuration was changed to use the internal Active Directory DNS server.

After remediation, the workstation successfully:

- Resolved the internal Active Directory domain
- Located the domain controller
- Communicated with Active Directory services
- Completed a Group Policy refresh
- Used the intended internal DNS infrastructure

---

## Before and After

| Check | Before | After |
|---|---|---|
| DNS Configuration | External DNS | Internal AD DNS |
| Domain Membership | Confirmed | Confirmed |
| Internal Domain Resolution | At risk | Successful |
| Domain Controller Discovery | At risk | Successful |
| Group Policy Update | At risk | Successful |

---

## Validation Commands

```powershell
whoami
hostname
Get-ComputerInfo | Select-Object CsDomain, CsName
ipconfig /all
Get-ADDomain
Get-DnsServerZone
Set-DnsClientServerAddress
nslookup
nltest
gpupdate /force
```

---

## Business Impact

Incorrect DNS configuration on a domain-joined workstation can affect:

- User authentication
- Domain logins
- Group Policy processing
- Shared resource access
- Domain controller discovery
- Enterprise application connectivity

Correcting the DNS configuration restored reliable communication between the workstation and Active Directory services.

---

## Skills Demonstrated

- Windows 11 troubleshooting
- Active Directory troubleshooting
- Windows DNS troubleshooting
- Domain controller discovery
- PowerShell administration
- Group Policy validation
- Network configuration
- Root cause analysis
- Tier 2 escalation thinking
- Technical documentation
- Junior system administration

---

## Lessons Learned

This project reinforced that DNS is a critical dependency in Active Directory environments.

A domain-joined workstation may appear functional while still being incorrectly configured for domain operations. Public DNS services can resolve internet addresses, but they do not provide the internal service records required for Active Directory authentication and management.

The troubleshooting workflow used in this project was:

1. Verify domain membership
2. Review network configuration
3. Identify the configured DNS server
4. Verify the domain controller
5. Verify Active Directory DNS
6. Correct the workstation DNS configuration
7. Test domain resolution
8. Confirm domain controller discovery
9. Validate Group Policy

