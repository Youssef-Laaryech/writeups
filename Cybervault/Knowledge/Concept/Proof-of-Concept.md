# Proof-of-Concept (PoC)

## What it is

A Proof-of-Concept (PoC) in penetration testing is **the demonstration that a discovered vulnerability is actually exploitable** — not just theoretically present. It bridges the gap between "we found a CVE" and "we proved it has real-world impact." It is the decision-making basis that allows administrators, developers, and management to understand what needs to be fixed, confirm it is real, and validate that their remediation actually worked.

## Position in the Pentest Process

```
Pre-Engagement → Information Gathering → Vulnerability Assessment → Exploitation
→ Post-Exploitation → Lateral Movement → [Proof-of-Concept] → Post-Engagement
```

---

## Forms a PoC Can Take

| Form | Description | Best for |
|------|-------------|---------|
| **Documentation** | Detailed step-by-step walkthrough — commands run, outputs observed, screenshots, timestamps | All findings — the written form is always required |
| **Script / code** | Automated exploit that reproduces the finding end-to-end | High-severity findings where automation makes reproduction unambiguous |
| **Screen recording** | Video of the exploitation chain | Complex multi-step chains, data exfiltration proof |
| **Screenshot + terminal log** | Showing hostname, IP, username, file path, and the sensitive output | Any privilege escalation or data access finding |
| **`calc.exe` / command output** | Classic Windows PoC — executing `calc.exe` or `whoami` on the target via RCE | RCE findings — simple, unambiguous proof of code execution |

---

## The Script PoC Problem

Providing a working exploit script to the client has one significant risk: **administrators focus on breaking the script instead of fixing the vulnerability.**

**What happens:**
1. Admin receives script, sees it exploits X via technique Y
2. Admin modifies the system so that specific technique Y no longer works
3. Admin considers the finding remediated
4. The underlying vulnerability still exists — a different technique still works

**Example:**
- PoC script exploits a weak password via a specific authentication path
- Admin blocks that specific path or patches the script's exact behavior
- The weak password (and the weak password policy behind it) still exists
- Another tool or approach reaches the same outcome

**What to do:**
- Explicitly state in the report and the review meeting: *this script is one path, not the only path*
- Remediation must address the root cause, not the specific technique demonstrated
- The script is evidence — it is not the attack surface

---

## Root Cause vs Symptom

The most important principle of a good PoC:

> **The PoC demonstrates the symptom. The report must identify the root cause.**

| Symptom (what the PoC shows) | Root cause (what must be fixed) |
|------------------------------|--------------------------------|
| Domain Admin using `Password123` | No password complexity policy enforced |
| SQLi in a login form | No input validation / parameterized queries |
| RCE via file upload | No file type validation, no execution prevention in upload dir |
| SMB null session leaking user list | Anonymous access not disabled |
| Kerberoastable service account with weak password | Service accounts not subject to strong password policy |

Fixing one Domain Admin's password while leaving the policy unchanged means every other user is still at risk. The report must make this explicit.

---

## The Attack Chain Walkthrough

For complex findings — especially domain compromise during an internal assessment — the most valuable PoC format is a **full attack chain walkthrough**:

```
Compromised workstation (phishing)
  → Pillaging: found cleartext password in script
    → Credential reuse: authenticated to file server
      → Pillaging: found SSH key on share
        → SSH to dev server
          → Local privesc: writable cron job → root
            → Found database credentials in .env
              → Accessed production DB → Domain Controller
```

**Why this format is powerful:**

1. Shows the **compounding effect** — no single flaw caused the breach; the chain of small issues did
2. Shows that **fixing one link breaks the chain** — but the other links still exist
3. Demonstrates **business impact** at each step — what data was accessible, what systems were reachable
4. Drives the point that **all links must be fixed**, not just the most visible one

---

## What a Good PoC Includes

For each finding in the report:

| Element | Content |
|---------|---------|
| **Finding title** | Clear, specific (not "weak password" — "Domain Admin account uses password meeting minimum complexity only") |
| **Severity** | CVSS score or risk rating |
| **Description** | What the vulnerability is and how it was discovered |
| **Impact** | What an attacker could do with it — in business terms |
| **Evidence** | Screenshot, terminal output, screen recording — with hostname, IP, timestamp |
| **Reproduction steps** | Exact steps to reproduce — allows the client to validate and verify after remediation |
| **Root cause** | The underlying issue (policy, configuration, code) not the surface symptom |
| **Remediation** | Specific, actionable fix — not "fix the password" but "enforce minimum 12-character passwords with complexity via Group Policy" |

---

## Evidence Best Practices

| Situation | Evidence format |
|-----------|----------------|
| RCE / code execution | Screenshot of `id` / `whoami` output showing username and hostname |
| Privilege escalation | Before: `id` showing low-priv user → After: `id` showing root/SYSTEM |
| Data access | Screenshot showing: hostname, IP, username, full file path, sensitive content (redacted if real PII) |
| Data exfiltration | Screen recording of the transfer + hash of file proving origin |
| Lateral movement | Authentication logs, session proof on each newly reached host |

> Always capture evidence in real time during the engagement. Reconstructing proof after the fact produces weak, unconvincing evidence and vague reproduction steps.

---

## The Report Review Meeting

The PoC alone is not enough — it must be walked through with the client:

- Explain each finding and why the root cause matters, not just the symptom
- Walk through the attack chain end-to-end — make the compounding effect visible
- Explicitly say: *patching one link does not eliminate the chain — every link must be addressed*
- Give developers and admins the opportunity to ask questions about reproduction and remediation
- Confirm that the client's remediation plan targets root causes, not script-specific behaviors

---

## Key Takeaways

| Lesson | Detail |
|--------|--------|
| **A PoC proves impact, not just presence** | A CVE number is not a PoC — showing it execute on the target is |
| **Scripts are one path, not the only path** | Clients who patch the script instead of the vulnerability are still vulnerable |
| **Always identify the root cause** | Changing one admin's password without fixing the password policy changes nothing |
| **Attack chain walkthroughs show compounding risk** | Multi-step chains are more persuasive than individual findings and harder to dismiss |
| **Evidence must be captured in real time** | Hostname + IP + username + timestamp in every screenshot — no exceptions |
| **Remediation advice must be specific** | "Fix the vulnerability" is not actionable — "enforce X via Y mechanism" is |

---

## Related

- [[Lateral Movement]]
- [[Post-Exploitation]]
- [[Exploitation]]
- [[Vulnerability Assessment]]
- [[Pre-Engagement]]
