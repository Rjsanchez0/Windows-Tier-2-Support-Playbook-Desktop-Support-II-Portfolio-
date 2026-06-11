# T2-006 — Windows Slow / Performance Degradation (Tier 2 Incident Write-Up)

## Ticket Scenario
**User Impact:** Workstation performance is degraded (slow login, lagging apps, freezes), preventing normal work.  
**Priority:** Medium (High if the user cannot perform job tasks or if multiple users are affected).  
**Category:** Desktop Support → Performance / Troubleshooting

---

## Reported Symptoms
- “Computer is extremely slow today.”
- Slow login, delayed application launches, intermittent freezing.
- Fan noise/high activity may be noticed (not always).

---

## Environment (Example)
- Endpoint: Windows 10/11 workstation (corporate managed)
- User: Standard domain user
- Workstyle: On-site or remote (VPN may be involved)
- Management/Tools: Ticketing system + remote support tool
- Security: EDR/AV enabled (Windows Defender or enterprise EDR)

> Note: This is a portfolio lab-style write-up; values like hostnames/IPs are intentionally omitted.

---

## Triage Questions (Tier 2)
Asked to quickly determine scope, urgency, and the most likely root cause:

1) **Scope**
- Is it **only this user/device**, or are multiple users reporting slowness today?
- Is the issue constant or only during certain actions (opening Outlook/Teams, browsing, logging in)?

2) **Timeline**
- When did it start? Was it after a Windows update, software install, VPN change, or power loss?

3) **Performance Pattern**
- Is it slow at boot/login (startup-related) or only after logging in (process/app related)?
- Any recent “low disk space” warnings or crash messages?

4) **Security/Account Context**
- Any suspicious popups, unusual new programs, or repeated credential prompts?
- Is the user on VPN? (VPN + DNS/proxy issues can make apps feel “slow” even when the PC is fine.)

---

## Tier 2 Troubleshooting Workflow (What I would do)
This follows a consistent Tier 2 pattern: **clarify → isolate → test → remediate → verify → document → escalate**.

### Step 1 — Establish a baseline (identify the bottleneck)
**Goal:** Determine whether slowness is caused by CPU, RAM, Disk, or Network/app dependencies.

Actions:
- Open **Task Manager** → Performance tab (CPU/RAM/Disk)
- Open **Task Manager** → Processes tab and sort by:
  - CPU (highest first)
  - Memory (highest first)
  - Disk (highest first)

What I’m looking for:
- **High CPU** (e.g., browser tabs, Teams, indexing, antivirus scan, stuck update)
- **High Memory** (paging to disk, too many apps, browser memory leak)
- **High Disk** (often the most common “feels slow” cause; updates, search indexing, low disk space, dying drive)
- **Network-related slowness** (apps loading slowly while CPU/RAM are normal)

Evidence to document:
- The top 1–3 processes consuming resources
- Overall CPU/RAM/Disk % during the complaint
- Free disk space

---

### Step 2 — Quick remediations (safe, high success rate)
These are “Tier 2 safe moves” that resolve a large chunk of performance tickets without destructive changes.

**A) Startup bloat**
- Task Manager → Startup apps
- Disable nonessential entries (especially high impact)
- Reboot recommended if startup items were changed

**B) Low disk space**
- Check free disk space (Settings → Storage)
- Remove temp files / cleanup
- If disk is critically low, prioritize freeing space first (performance can improve immediately)

**C) Stuck updates or background tasks**
- Check Windows Update status
- Look for `TiWorker`, `TrustedInstaller`, indexing, or update-related services that are stuck
- If update-related, allow completion or schedule a reboot window if safe

**D) Browser/app sprawl**
- Reduce open tabs, close unused apps
- If one app is consistently the offender, test launching it cleanly (no extensions, safe mode if applicable)

**E) Security scan spikes**
- Confirm if AV/EDR scan is running
- If the organization allows: let it finish or schedule scan for off-hours
- If scans are constant/unusual, escalate to security team (possible malware or misconfiguration)

---

### Step 3 — Tier 2 validation checks (go beyond basic Task Manager)
**A) Event Viewer quick scan**
- Event Viewer → Windows Logs → System
- Filter for **Critical** and **Error**
- Look for patterns:
  - Disk warnings/errors
  - Repeated service crashes
  - Controller resets
  - Boot/performance errors

**B) Reliability Monitor (fast insight)**
- Open “Reliability Monitor” (View reliability history)
- Identify recent crash spikes, failed updates, or app failures tied to the timeline

**C) Storage health indicators**
- Check disk SMART/health (if tooling exists) or warnings in Event Viewer
- If disk errors appear → treat as high priority escalation (hardware failure risk)

---

## Likely Root Causes (Most common Tier 2 diagnoses)
1) **Low disk space / high disk utilization** (top cause of “everything is slow”)  
2) **Too many startup apps** causing slow login and background load  
3) **A single runaway process** (browser/Teams/OneDrive sync/indexer)  
4) **Windows Update or indexing stuck** causing sustained disk/CPU activity  
5) **Failing storage drive** (Event Viewer disk errors, freezes, long app loads)  
6) **Malware/PUA** (unusual startup entries, suspicious processes, repeated security alerts)

---

## Resolution (Example outcome)
Actions taken:
- Disabled nonessential startup items (high impact) and rebooted
- Freed disk space to restore normal operation (temporary files + unnecessary installers)
- Verified the top offender process no longer consumes abnormal CPU/RAM/Disk
- Confirmed Windows Update was not stuck (or allowed completion)

---

## Verification (Required to close the ticket)
Success criteria:
- Task Manager shows stable baseline:
  - CPU not pinned at high %
  - Disk utilization not stuck at 100%
  - Adequate free disk space restored (target: >10–15% free)
- User confirms:
  - Login time improved
  - Apps open normally
  - No freezing during normal tasks

---

## Escalation Criteria (When Tier 2 should escalate)
Escalate to Systems/Admin if:
- Disk/OS corruption suspected (SFC/DISM required by policy)
- Repeated service crashes tied to core Windows services
- Multiple devices affected (possible update rollout issue)

Escalate to Security if:
- Suspicious processes/startup entries are found
- EDR detections occur or repeated credential prompts suggest compromise
- System behavior suggests malware (persistent high CPU/Disk with unknown binaries)

Escalate to Hardware/Field Tech if:
- Disk warnings/errors in Event Viewer
- Device shows signs of failing storage (freezing, I/O timeouts, SMART warnings)

---

## Documentation Notes (How I would write the ticket)
- Captured: user impact + timeframe + scope
- Captured: top offender process and resource readings
- Documented: actions taken and why (startup cleanup, disk cleanup, update verification)
- Documented: verification evidence (baseline improved + user confirmation)
- Included: escalation packet if needed (Event IDs, timestamps, process names, error text)

---

## Lessons Learned / Prevention
- Keep endpoints with healthy free disk space (monitoring/alerts)
- Reduce unnecessary startup apps via policy/standard image
- Ensure update rings don’t cause widespread performance issues
- Train users on early warning signs (low disk space, constant freezing)