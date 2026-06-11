# T2-003 — VPN Can’t Connect (Tier 2 Triage + Escalation-Ready Evidence)

## Ticket Scenario
**User Impact:** User cannot connect to VPN and is blocked from accessing internal resources (file shares, intranet apps, remote desktop/jump host).  
**Priority:** High (remote user blocked).  
**Category:** Remote Access → VPN / Authentication

---

## Symptoms
- VPN connection attempt fails (cannot connect / server unreachable / authentication error).
- User cannot reach internal resources required to work.

**Example user statement:** “VPN worked yesterday. Today it fails every time I click Connect.”

---

## Environment
- Endpoint: Windows 10/11 workstation (VirtualBox lab)
- User type: Standard user (domain or local, depending on environment)
- Connectivity: Internet available (optional NAT adapter)
- VPN Client: Windows built-in VPN profile (lab simulation)

> Note: This is a portfolio write-up. Hostnames/IPs are intentionally omitted or redacted.

---

## Tier 2 Triage Questions
These questions determine scope and whether the issue is endpoint-side, account-side, or server-side.

1) **Scope**
- Is this happening to **one user** or **multiple users**?
- Does it fail on **one device** or **all devices** for the user?

2) **Connectivity**
- Does general internet work (websites load)?
- Is the user on home Wi-Fi, office network, hotel Wi-Fi, or hotspot?

3) **Authentication**
- Did the user recently reset their password?
- Is MFA involved? Does the MFA prompt appear and succeed?

4) **Change history**
- Any recent Windows updates or VPN client updates?
- Any new security software/policy changes?

5) **Error specifics**
- Exact error message/code shown by the VPN client?
- Which VPN profile/server is selected?

---

## Troubleshooting Workflow (Tier 2)

### Step 1 — Confirm baseline connectivity (rule out “not actually VPN”)
**Goal:** Ensure the endpoint can reach the internet and resolve DNS before blaming VPN.

Actions:
- Check Wi-Fi/Ethernet connected.
- Run quick tests:
  - `ping 8.8.8.8` (internet by IP)
  - `nslookup google.com` (DNS resolution)
  - (Optional) `ipconfig /all` for adapter, gateway, DNS servers (redact)

Interpretation:
- If ping fails → local network issue (ISP/router/Wi-Fi)
- If ping works but DNS fails → DNS problem (fix first)
- If both work → proceed to VPN-specific steps

---

### Step 2 — Validate system time (common hidden failure)
Modern authentication and certificate validation can fail if time is incorrect.

Actions:
- Confirm correct time/time zone
- Sync time (Windows “Sync now”)

---

### Step 3 — Attempt VPN connection and capture the exact error
**Goal:** Capture reproducible evidence and identify whether failure is “reachability” or “authentication.”

Actions:
- Attempt connection using the VPN profile.
- Record exact error text (and any error number/code).

Interpretation examples:
- **Server unreachable/timeouts** → network/DNS/gateway/outage
- **Auth failed** → account lockout, password expired, MFA issues
- **Policy-related** → device compliance/EDR/firewall blocks

---

### Step 4 — Perform safe client-side remediation (Tier 2 approved steps)
These are steps Tier 2 can take without disrupting system stability:

- Reboot workstation (quick reset of services/drivers)
- Disable/enable network adapter
- Test alternate network (hotspot) to rule out ISP/network restrictions
- Recreate the VPN profile (config corruption)
- Clear cached credentials related to the VPN profile (if applicable)
- Confirm no captive portal blocks exist (common on hotel/airport Wi-Fi)

---

### Step 5 — Determine likely root cause category
Based on results, classify it to speed escalation:

**A) Endpoint/network issue**
- DNS failure, unstable Wi-Fi, captive portal, adapter misconfiguration

**B) Account/authentication issue**
- Password expired, account locked, MFA failing, identity provider issues

**C) VPN gateway/server-side issue**
- Multiple users affected, gateway outage, routing issue, maintenance window

**D) Security/policy block**
- EDR/firewall blocks tunnel driver, device posture failing compliance

---

## Escalation-Ready Evidence Packet (What Tier 2 should attach)
Even without logs, Tier 2 adds value by escalating with clean evidence:

- Screenshot of VPN profile selected (server/profile name)
- Screenshot of the VPN failure message (with timestamp)
- Baseline connectivity proof:
  - ping IP + nslookup hostname output
- Scope confirmation:
  - only one user vs multiple users
  - alternate network test result (if tried)
- Notes on remediation attempted:
  - reboot, adapter reset, profile recreated, time sync

---

## Example Root Cause (Lab)
In a lab environment using a simulated Windows built-in VPN profile, the VPN endpoint is not expected to connect (unreachable/invalid gateway). This reproduces a real-world Tier 2 scenario where the user experiences a VPN failure and Tier 2 must triage, collect evidence, and escalate appropriately.

---

## Resolution (Real-World Paths)
Resolution depends on the category:

- **Endpoint/DNS issue:** correct DNS settings, remove captive portal, stabilize network, retry
- **Account/MFA issue:** verify account status, unlock/reset, MFA re-registration if needed
- **Gateway outage:** escalate to Network team; provide evidence + impact scope
- **Policy block:** escalate to Security/Endpoint team; provide error text + device details

---

## Verification (Close Criteria)
Ticket can be closed only after confirming:
- VPN status shows **Connected**
- User can reach at least one internal resource (per policy), such as:
  - internal DNS resolution
  - intranet page
  - file share
  - approved remote access host
- User confirms stability (no frequent disconnects)

---

## Documentation Notes (How I would write the ticket)
- Captured: user impact, scope, exact error text
- Captured: baseline network tests
- Documented: remediation attempts and results
- Provided: escalation-ready evidence packet to reduce back-and-forth