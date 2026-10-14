# AD-005 — Active Directory PowerShell Automation

## Project Overview

This project demonstrates how PowerShell can be used to automate common Active Directory user lifecycle tasks in a Windows domain environment.

The project includes two reusable PowerShell scripts:

- `Onboard-ADUser.ps1`
- `Offboard-ADUser.ps1`

The goal of the project was to reduce repetitive administrative work, improve consistency, and create a safer, repeatable process for onboarding and offboarding Active Directory users.

---

## Environment

- **Domain:** `LAB.local`
- **Domain Controller:** `DC01`
- **Domain-Joined Workstation:** `WS01`
- **Virtualization:** VirtualBox
- **PowerShell Module:** ActiveDirectory
- **Directory Services:** Active Directory Domain Services
- **Primary OU:** Engineering
- **Offboarding OU:** Offboarded Users
- **Primary Security Group:** `Engineering_SG`

---

## Technologies Used

- PowerShell
- Active Directory Domain Services
- Active Directory PowerShell Module
- Active Directory Users and Computers
- Organizational Units
- Security Groups
- Windows Server
- VirtualBox

---

# Scenario

IT receives recurring onboarding and offboarding requests for employees.

Performing these tasks manually creates repetitive administrative work and increases the possibility of mistakes, including:

- Creating users in the wrong Organizational Unit
- Forgetting to assign required security groups
- Failing to disable an account during offboarding
- Forgetting to remove department-specific access
- Leaving disabled users mixed with active employee accounts

The objective of this project was to automate common Active Directory lifecycle tasks using PowerShell.

---

# Project Goals

## Onboarding

The onboarding workflow needed to:

1. Create a domain user
2. Configure first name and last name
3. Configure the `SamAccountName`
4. Configure the User Principal Name
5. Place the user in the Engineering OU
6. Set department information
7. Enable the account
8. Assign a temporary password securely
9. Require a password change at first logon
10. Add the user to `Engineering_SG`
11. Verify the resulting account
12. Verify group membership

## Offboarding

The offboarding workflow needed to:

1. Identify the user to offboard
2. Disable the Active Directory account
3. Remove department-specific security group access
4. Move the account to the Offboarded Users OU
5. Verify account status
6. Verify access removal
7. Confirm the user's final Active Directory state

---

# Part 1 — Validate the Active Directory PowerShell Environment

The Active Directory PowerShell module was loaded before making any changes.

```powershell
Import-Module ActiveDirectory
```

The domain connection was then validated using:

```powershell
Get-ADDomain
```

The command successfully returned information for:

`LAB.local`

The output also confirmed the Domain Controller:

`DC01.LAB.local`

This verified that the Active Directory PowerShell module was available and that PowerShell could communicate with the domain.

## Screenshot

![Active Directory PowerShell Environment](../images/AD-005/01-ad-powershell-environment.png)

---

# Part 2 — Review Existing Active Directory Objects

Before automating account changes, the existing Organizational Units were reviewed.

```powershell
Get-ADOrganizationalUnit -Filter * |
Select-Object Name, DistinguishedName
```

The required Organizational Units were confirmed:

- `Engineering`
- `Offboarded Users`

Existing Active Directory groups were also reviewed.

```powershell
Get-ADGroup -Filter * |
Select-Object Name
```

The required Engineering security group was confirmed:

`Engineering_SG`

This validation step helps prevent scripts from referencing incorrect or nonexistent Active Directory paths or groups.

---

# Part 3 — Create a User with PowerShell

A fictional Engineering employee was created for the initial PowerShell test.

## Test User

- **Name:** Avery Stone
- **Username:** `astone`
- **Department:** Engineering
- **Domain:** `LAB.local`

A temporary password was collected securely.

```powershell
$Password = Read-Host "Enter temporary password" -AsSecureString
```

The account was then created using:

```powershell
New-ADUser `
    -Name "Avery Stone" `
    -GivenName "Avery" `
    -Surname "Stone" `
    -SamAccountName "astone" `
    -UserPrincipalName "astone@LAB.local" `
    -Path "OU=Engineering,DC=LAB,DC=local" `
    -AccountPassword $Password `
    -Enabled $true `
    -ChangePasswordAtLogon $true
```

The account was verified using:

```powershell
Get-ADUser astone -Properties Enabled,DistinguishedName |
Select-Object Name,SamAccountName,Enabled,DistinguishedName
```

The verification confirmed that:

- Avery Stone was created successfully
- The username was `astone`
- The account was enabled
- The account was placed in the Engineering OU

## Screenshot

![PowerShell User Creation](../images/AD-005/02-user-created.png)

---

# Part 4 — Assign Security Group Membership

The new user was added to the Engineering security group.

```powershell
Add-ADGroupMember `
    -Identity "Engineering_SG" `
    -Members "astone"
```

Membership was verified using:

```powershell
Get-ADGroupMember -Identity "Engineering_SG" |
Select-Object Name,SamAccountName
```

The output confirmed that Avery Stone was a member of:

`Engineering_SG`

This demonstrates group-based access control rather than assigning access directly to individual user accounts.

## Screenshot

![Engineering Group Membership](../images/AD-005/03-group-membership.png)

---

# Part 5 — Test PowerShell Offboarding Commands

Before building the reusable offboarding script, the individual PowerShell commands were tested manually.

The account was disabled using:

```powershell
Disable-ADAccount -Identity "astone"
```

The Engineering security group membership was removed using:

```powershell
Remove-ADGroupMember `
    -Identity "Engineering_SG" `
    -Members "astone" `
    -Confirm:$false
```

The account was moved to the Offboarded Users OU using:

```powershell
Get-ADUser astone |
Move-ADObject `
    -TargetPath "OU=Offboarded Users,DC=LAB,DC=local"
```

The account state was verified using:

```powershell
Get-ADUser astone -Properties Enabled,DistinguishedName |
Select-Object Name,Enabled,DistinguishedName
```

Group membership was verified using:

```powershell
Get-ADPrincipalGroupMembership astone |
Select-Object Name
```

The final state confirmed:

- The account was disabled
- The user was moved to the Offboarded Users OU
- `Engineering_SG` had been removed
- The user retained only default group membership such as `Domain Users`

---

# Part 6 — Build the Reusable Onboarding Script

A reusable onboarding script was created.

## Script Name

`Onboard-ADUser.ps1`

The onboarding script performs the following actions:

1. Imports the Active Directory PowerShell module
2. Defines employee information
3. Defines the Engineering OU
4. Defines the Engineering security group
5. Prompts securely for a temporary password
6. Creates the Active Directory account
7. Places the user in the Engineering OU
8. Sets the department
9. Enables the account
10. Requires a password change at first logon
11. Adds the user to `Engineering_SG`
12. Verifies the account
13. Displays group memberships

The reusable test account was:

- **Name:** Morgan Reed
- **Username:** `mreed`
- **Department:** Engineering

## Onboard-ADUser.ps1

```powershell
Import-Module ActiveDirectory

# -----------------------------
# USER INFORMATION
# -----------------------------

$FirstName = "Morgan"
$LastName = "Reed"
$Username = "mreed"
$Department = "Engineering"

$EngineeringOU = "OU=Engineering,DC=LAB,DC=local"
$EngineeringGroup = "Engineering_SG"

# -----------------------------
# ONBOARD USER
# -----------------------------

Write-Host "Starting onboarding for $FirstName $LastName..."

$Password = Read-Host "Enter temporary password" -AsSecureString

New-ADUser `
    -Name "$FirstName $LastName" `
    -GivenName $FirstName `
    -Surname $LastName `
    -SamAccountName $Username `
    -UserPrincipalName "$Username@LAB.local" `
    -Department $Department `
    -Path $EngineeringOU `
    -AccountPassword $Password `
    -Enabled $true `
    -ChangePasswordAtLogon $true

Add-ADGroupMember `
    -Identity $EngineeringGroup `
    -Members $Username

Write-Host "Onboarding complete."

Get-ADUser $Username -Properties Enabled,Department,DistinguishedName |
    Select-Object Name,SamAccountName,Enabled,Department,DistinguishedName

Get-ADPrincipalGroupMembership $Username |
    Select-Object Name
```

The script completed successfully.

The resulting account configuration confirmed:

- **Name:** Morgan Reed
- **SamAccountName:** `mreed`
- **Enabled:** True
- **Department:** Engineering
- **OU:** Engineering
- **Security Group:** `Engineering_SG`

## Screenshot

![Automated User Onboarding](../images/AD-005/04-automated-onboarding.png)

---

# Part 7 — Build the Reusable Offboarding Script

A separate script was created for offboarding.

## Script Name

`Offboard-ADUser.ps1`

The onboarding and offboarding workflows were intentionally separated into different scripts.

This prevents a single script from creating a user and immediately disabling that same account.

The offboarding script performs the following actions:

1. Imports the Active Directory PowerShell module
2. Prompts the administrator for a username
3. Defines the Engineering security group
4. Defines the Offboarded Users OU
5. Disables the account
6. Removes `Engineering_SG`
7. Moves the account to the Offboarded Users OU
8. Verifies that the account is disabled
9. Displays the remaining group memberships

## Offboard-ADUser.ps1

```powershell
Import-Module ActiveDirectory

# -----------------------------
# USER INFORMATION
# -----------------------------

$Username = Read-Host "Enter username to offboard"

$EngineeringGroup = "Engineering_SG"

$OffboardedOU = "OU=Offboarded Users,DC=LAB,DC=local"

# -----------------------------
# OFFBOARD USER
# -----------------------------

Write-Host "Starting offboarding for $Username..."

Disable-ADAccount -Identity $Username

Remove-ADGroupMember `
    -Identity $EngineeringGroup `
    -Members $Username `
    -Confirm:$false

Get-ADUser $Username |
    Move-ADObject -TargetPath $OffboardedOU

Write-Host "Offboarding complete."

# -----------------------------
# VERIFY
# -----------------------------

Get-ADUser $Username -Properties Enabled,DistinguishedName |
    Select-Object Name,SamAccountName,Enabled,DistinguishedName

Get-ADPrincipalGroupMembership $Username |
    Select-Object Name
```

The script successfully offboarded:

`mreed`

The final output confirmed:

- **Name:** Morgan Reed
- **SamAccountName:** `mreed`
- **Enabled:** False
- **OU:** Offboarded Users
- **Engineering_SG:** Removed
- **Remaining default membership:** Domain Users

## Screenshot

![Automated User Offboarding](../images/AD-005/05-automated-offboarding.png)

---

# Troubleshooting

During the initial development of `Offboard-ADUser.ps1`, PowerShell returned errors similar to:

```text
Cannot validate argument on parameter 'Identity'.
The argument is null.
```

The errors affected commands including:

```text
Disable-ADAccount
Remove-ADGroupMember
Get-ADUser
Get-ADPrincipalGroupMembership
```

The issue occurred because the offboarding script referenced variables that had only been defined in the onboarding script.

The undefined variables included:

```powershell
$Username
$EngineeringGroup
$OffboardedOU
```

PowerShell variables defined in one script are not automatically available inside another independent script.

The issue was resolved by defining the required variables inside `Offboard-ADUser.ps1`.

The username was also changed from a hard-coded value to an interactive prompt:

```powershell
$Username = Read-Host "Enter username to offboard"
```

This improved the script by allowing the administrator to reuse it for different accounts without editing the source code.

---

# PowerShell Commands Used

```powershell
Import-Module ActiveDirectory
```

```powershell
Get-ADDomain
```

```powershell
Get-ADOrganizationalUnit
```

```powershell
Get-ADGroup
```

```powershell
New-ADUser
```

```powershell
Get-ADUser
```

```powershell
Add-ADGroupMember
```

```powershell
Get-ADGroupMember
```

```powershell
Disable-ADAccount
```

```powershell
Remove-ADGroupMember
```

```powershell
Move-ADObject
```

```powershell
Get-ADPrincipalGroupMembership
```

---

# Onboarding Workflow

```text
HR / IT Onboarding Request
        ↓
Run Onboard-ADUser.ps1
        ↓
Create Active Directory Account
        ↓
Assign Engineering OU
        ↓
Set Department
        ↓
Enable Account
        ↓
Require Password Change
        ↓
Add Engineering_SG
        ↓
Verify Account
        ↓
Verify Group Membership
```

---

# Offboarding Workflow

```text
HR / IT Termination Request
        ↓
Run Offboard-ADUser.ps1
        ↓
Disable Active Directory Account
        ↓
Remove Engineering_SG
        ↓
Move to Offboarded Users OU
        ↓
Verify Account Disabled
        ↓
Verify Access Removal
```

---

# Security Concepts Demonstrated

## Secure Password Handling

Temporary passwords were collected using:

```powershell
Read-Host -AsSecureString
```

Passwords were not stored in plaintext inside the script.

## Group-Based Access Control

Department access was assigned using:

`Engineering_SG`

rather than assigning permissions directly to individual accounts.

## Least Privilege

The user received only the department-specific access required for the Engineering role.

## Identity Lifecycle Management

The project demonstrates both:

- User provisioning
- User deprovisioning

## Access Revocation

The offboarding workflow:

- Disabled authentication
- Removed Engineering group access
- Moved the user out of the active Engineering OU

## Administrative Automation

PowerShell was used to reduce repetitive Active Directory administration and improve consistency.

---

# Skills Demonstrated

- PowerShell scripting
- Active Directory administration
- Active Directory PowerShell module
- User provisioning
- User deprovisioning
- Organizational Unit management
- Security group administration
- Identity lifecycle management
- Group-based access control
- Least privilege
- Secure password handling
- Administrative automation
- PowerShell troubleshooting
- Validation and verification
- Technical documentation


---

# Outcome

The project successfully automated common Active Directory onboarding and offboarding tasks.

Two separate PowerShell scripts were created to provide safer, reusable workflows for user provisioning and deprovisioning.

The project demonstrates how PowerShell can reduce repetitive administrative work, improve consistency, support identity lifecycle management, and provide clear validation of:

- Account creation
- Organizational Unit placement
- Security group assignment
- Account disablement
- Group membership removal
- Offboarding OU placement

