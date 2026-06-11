# T2-004 — Outlook / Microsoft 365 Sign-In Loop (Tier 2 Troubleshooting Walkthrough)

## Ticket Scenario
**User Impact:** User cannot access email. Outlook repeatedly prompts for password or fails to sign in, blocking work.  
**Priority:** High (email is business-critical; remote users may be fully blocked).  
**Category:** Microsoft 365 / Outlook → Authentication / Profile

---

## Reported Symptoms
- Outlook prompts for password repeatedly.
- Outlook shows “Trying to connect…” or mailbox never loads.
- User may be able to sign in to webmail, but Outlook fails.
- Sometimes the issue started after a password reset, MFA change, update, or device move.

---

## Environment (Example)
- Endpoint: Windows 10/11 workstation (domain-joined or standalone)
- Client: Outlook (Classic) or New Outlook
- Identity: Microsoft 365 (Entra ID/Azure AD) or Outlook.com
- Authentication: Modern Auth + MFA (common)

> Portfolio note: hostnames, domains, and account identifiers intentionally omitted/redacted.

---

## Tier 2 Triage Questions (Ask first)
1) **Scope**
- Is it only one user or multiple users?
- Only Outlook, or does Teams/OneDrive sign-in also fail?

2) **Account validation**
- Can the user sign in to **webmail** (OWA) successfully?
- Did the user recently change password or MFA method?

3) **Device + network**
- On-site or remote (VPN involved)?
- Any captive portal / DNS issues?

4) **Error details**
- Exact error text (screenshot)
- Does it loop forever or fail once with an error code?

---

## Troubleshooting Workflow (Tier 2)

### Step 1 — Confirm whether it’s client-side or account-side
**Goal:** Separate “Outlook client problem” from “account/auth problem.”

Actions:
- Test web sign-in (OWA / outlook.office.com)
- Test another M365 app sign-in (Teams/OneDrive) if installed

Interpretation:
- **Web works but Outlook fails:** likely client/profile/credential/cache issue (Tier 2 can fix)
- **Web fails too:** likely account/MFA/Conditional Access/licensing (escalate to M365 admin)

---

### Step 2 — Quick client resets (safe, high success)
Actions:
- Fully close Outlook (Task Manager → end Outlook if stuck)
- Reboot (optional but common in Tier 2 workflow)

Reason:
- Clears hung processes and some auth token issues

---

### Step 3 — Clear cached credentials (most common loop fix)
**Why:** Stale credentials/tokens can cause endless prompts.

Actions (Classic Windows approach):
1) Control Panel → **Credential Manager**
2) Windows Credentials
3) Remove Office/Outlook-related entries (common patterns):
   - MicrosoftOffice / Outlook / ADAL / MSOID / Teams / OneDrive
4) Reopen Outlook and test

Expected result:
- Prompt appears once, user signs in, mailbox loads

If still failing:
- Move to Profile rebuild (next step)

---

### Step 4 — Rebuild Outlook profile (Tier 2 standard remediation)
**Why:** Corrupted profiles are a top cause of repeated prompts and “won’t sync.”

Actions:
1) Control Panel → **Mail (Microsoft Outlook)**
2) Show Profiles → **Add**
3) Create new profile (example name: `Outlook-Repair`)
4) Add the user’s email account
5) Set new profile as default (“Always use this profile”)

Expected result:
- New profile loads mailbox normally

---

### Step 5 — Modern Auth / token troubleshooting (Tier 2 depth)
If prompts continue after steps above, focus on auth/tokens:

Actions:
- Confirm Windows time/timezone is correct (time drift breaks modern auth)
- Try Outlook Safe Mode (Classic Outlook):  
  `outlook.exe /safe`
- Check Windows “Work or school account” sign-in state (if domain/work-joined)
- If allowed: Office Repair (Quick Repair first)

Interpretation:
- If Safe Mode works → add-in issue (disable add-ins)
- If repair helps → corrupted Office components

---

## Escalation Criteria (When Tier 2 escalates)
Escalate to Microsoft 365 / Identity admin when:
- Webmail sign-in fails
- MFA prompts never arrive or are denied unexpectedly
- Conditional Access blocks sign-in (location/device compliance)
- License/mailbox issues suspected
- Multiple users affected (possible tenant/service issue)

---

## Escalation Packet (What Tier 2 should send)
Include:
- User impact + start time + scope (1 user vs many)
- Screenshot of Outlook error/prompt
- Whether webmail works (yes/no)
- Whether Teams/OneDrive works (yes/no)
- Steps attempted + results:
  - credential manager cleared
  - new Outlook profile created
  - time sync confirmed
  - safe mode test (if used)
- Device info:
  - Windows version
  - Outlook version (Classic/New)
  - Network context (remote/VPN vs onsite)

This reduces back-and-forth and makes you look like a real Tier 2 tech.

---

## Likely Root Causes (Most common Tier 2 findings)
1) Cached credentials/tokens are stale after password/MFA change  
2) Corrupted Outlook profile  
3) Add-in causing auth loop or preventing mailbox load  
4) Time drift causing modern auth failures  
5) Account-side: MFA/Conditional Access/licensing/mailbox provisioning (admin fix)

---

## Resolution (Example)
- Cleared cached Office/Outlook credentials
- Rebuilt Outlook profile and set as default
- Verified mailbox loads and prompts stop

---

## Verification (Close Criteria)
Ticket is resolved only when:
- Outlook loads mailbox successfully and remains stable
- User can send/receive email (or inbox sync is confirmed)
- No repeated credential prompts

---

## Notes / Prevention
- After password/MFA changes, stale tokens can cause loops—credential cleanup + profile rebuild usually resolves.
- Standardize Tier 2 procedure: Webmail test → credential cleanup → new profile → escalate with packet.