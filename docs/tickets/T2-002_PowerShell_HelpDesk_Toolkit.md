# T2-002 — PowerShell Help Desk Toolkit (Tier 2 Evidence Collection + Safe Remediation)

## Ticket Scenario
**User Impact:** User reports intermittent issues (slow PC, “no internet,” VPN won’t connect, printer not printing). Tier 2 needs consistent evidence collection to troubleshoot quickly and escalate without repeatedly asking for basic diagnostics.  
**Priority:** Medium (High if multiple users impacted or user is blocked).  
**Category:** Desktop Support → Automation / Evidence Collection

---

## Objective (What this project demonstrates)
This project creates a Tier 2 “toolkit” that:
1) Generates a timestamped evidence folder  
2) Captures high-value diagnostics into structured text files  
3) Optionally runs safe, common remediation steps (when policy allows)  



---

## Lab Environment
- Windows workstation VM (VirtualBox)
- Network: LABNET + NAT (internet access)
- Execution: PowerShell (Administrator recommended)

> 
---

## Deliverables
### Script
- <# 
T2-002 Help Desk Toolkit
Purpose: Collect system/network evidence + run safe quick-fix commands into a timestamped folder.
Run as Admin for best results.
```powershell

param(
    [string]$OutputRoot = "$env:USERPROFILE\Desktop\T2-002-Toolkit-Output",
    [switch]$RunFixes
)

function New-EvidenceFolder {
    param([string]$Root)
    $stamp = Get-Date -Format "yyyyMMdd-HHmmss"
    $path  = Join-Path $Root "Evidence_$stamp"
    New-Item -ItemType Directory -Path $path -Force | Out-Null
    return $path
}

function Write-Section {
    param([string]$File, [string]$Title)
    Add-Content -Path $File -Value "`r`n====================`r`n$Title`r`n====================`r`n"
}

function Run-Cmd {
    param([string]$File, [string]$Command)
    Add-Content -Path $File -Value "PS> $Command"
    try {
        $out = cmd.exe /c $Command 2>&1
        Add-Content -Path $File -Value $out
    } catch {
        Add-Content -Path $File -Value "ERROR: $($_.Exception.Message)"
    }
    Add-Content -Path $File -Value ""
}

$evidenceDir = New-EvidenceFolder -Root $OutputRoot
$summaryFile = Join-Path $evidenceDir "SUMMARY.txt"
$cmdFile     = Join-Path $evidenceDir "COMMAND_OUTPUT.txt"

"Help Desk Toolkit Evidence Folder: $evidenceDir" | Out-File $summaryFile -Encoding UTF8
"Generated: $(Get-Date)" | Add-Content $summaryFile
"RunFixes: $RunFixes" | Add-Content $summaryFile

Write-Section -File $cmdFile -Title "SYSTEM SNAPSHOT"
try {
    $cs = Get-CimInstance Win32_ComputerSystem
    $os = Get-CimInstance Win32_OperatingSystem
    Add-Content $cmdFile "ComputerName: $env:COMPUTERNAME"
    Add-Content $cmdFile "User: $env:USERNAME"
    Add-Content $cmdFile "OS: $($os.Caption) ($($os.Version))"
    Add-Content $cmdFile ("Uptime: {0:dd\.hh\:mm\:ss}" -f ((Get-Date) - $os.LastBootUpTime))
    Add-Content $cmdFile "Manufacturer: $($cs.Manufacturer)"
    Add-Content $cmdFile "Model: $($cs.Model)"
    Add-Content $cmdFile ("Total RAM (GB): {0:N2}" -f ($cs.TotalPhysicalMemory/1GB))
} catch {
    Add-Content $cmdFile "SYSTEM SNAPSHOT ERROR: $($_.Exception.Message)"
}
Add-Content $cmdFile ""

Write-Section -File $cmdFile -Title "DISK / STORAGE"
try {
    Get-PSDrive -PSProvider FileSystem |
        Select-Object Name, @{n="FreeGB";e={[math]::Round($_.Free/1GB,2)}}, @{n="UsedGB";e={[math]::Round(($_.Used)/1GB,2)}}, @{n="TotalGB";e={[math]::Round(($_.Free+$_.Used)/1GB,2)}} |
        Format-Table -AutoSize | Out-String | Add-Content $cmdFile
} catch {
    Add-Content $cmdFile "DISK ERROR: $($_.Exception.Message)"
}
Add-Content $cmdFile ""

Write-Section -File $cmdFile -Title "NETWORK SNAPSHOT (CMD OUTPUT)"
Run-Cmd -File $cmdFile -Command "ipconfig /all"
Run-Cmd -File $cmdFile -Command "route print"
Run-Cmd -File $cmdFile -Command "arp -a"

Write-Section -File $cmdFile -Title "CONNECTIVITY TESTS"
Run-Cmd -File $cmdFile -Command "ping -n 4 8.8.8.8"
Run-Cmd -File $cmdFile -Command "ping -n 4 1.1.1.1"
Run-Cmd -File $cmdFile -Command "nslookup google.com"
Run-Cmd -File $cmdFile -Command "tracert -d 8.8.8.8"

Write-Section -File $cmdFile -Title "PRINTING QUICK CHECKS"
Run-Cmd -File $cmdFile -Command "sc query spooler"

Write-Section -File $cmdFile -Title "EVENT VIEWER QUICK CHECK (SYSTEM - LAST 25 ERRORS)"
try {
    Get-WinEvent -LogName System -MaxEvents 200 |
        Where-Object { $_.LevelDisplayName -in @("Error","Critical") } |
        Select-Object -First 25 TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List | Out-String | Add-Content $cmdFile
} catch {
    Add-Content $cmdFile "EVENT LOG ERROR: $($_.Exception.Message)"
}

if ($RunFixes) {
    Write-Section -File $cmdFile -Title "SAFE QUICK FIXES (RUNFIXES ENABLED)"
    Run-Cmd -File $cmdFile -Command "ipconfig /flushdns"
    Run-Cmd -File $cmdFile -Command "ipconfig /renew"
    Run-Cmd -File $cmdFile -Command "netsh winsock reset"
    Run-Cmd -File $cmdFile -Command "netsh int ip reset"
    Run-Cmd -File $cmdFile -Command "sc stop spooler"
    Run-Cmd -File $cmdFile -Command "sc start spooler"
}

Add-Content $summaryFile ""
Add-Content $summaryFile "Files created:"
Add-Content $summaryFile "- SUMMARY.txt"
Add-Content $summaryFile "- COMMAND_OUTPUT.txt"

Write-Host "Evidence collected at: $evidenceDir"
Write-Host "Summary: $summaryFile"
Write-Host "Command Output: $cmdFile"
if ($RunFixes) { Write-Host "Fixes executed. Reboot may be required for winsock/IP reset." }
```

### Evidence Output (created automatically)
- `Desktop\T2-002-Toolkit-Output\Evidence_YYYYMMDD-HHMMSS\`
  - `SUMMARY.txt`
  - `COMMAND_OUTPUT.txt`

---

## Triage Questions (Tier 2 mindset)
Before changing anything, Tier 2 clarifies:
- Is this **one user** or **multiple users**?
- Is it **one device** or multiple devices?
- Is the issue constant or intermittent?
- Any recent changes (updates, password reset, VPN change, moving networks)?
- Are there obvious symptoms:
  - “Host not found” errors (DNS)
  - VPN errors (auth/routing)
  - slow boot/lag (disk/startup)
  - printing stuck (spooler/queue)

This toolkit supports those questions by generating consistent evidence.

---

## Walkthrough (What I did)

### Step 1 — Created the script file
Created the toolkit script at:
- `scripts/T2-002_HelpDeskToolkit.ps1`

Purpose:
- Standardize diagnostics Tier 2 repeatedly collects (system, disk, network, connectivity)
- Produce an evidence packet for escalation

---

### Step 2 — Ran evidence collection (no system changes)
**Command used (PowerShell as Admin recommended):**
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\scripts\T2-002_HelpDeskToolkit.ps1