# Post-Engagement

## What it is

Post-engagement is the **final phase of a penetration test** — everything that happens after testing stops. It is the most contractually binding phase and the one most beginners underestimate. The technical work is done; what remains is cleanup, documentation, reporting, review, remediation testing, data retention, and closeout. How this phase is handled determines whether the client got value from the engagement or just a bill.

## Position in the Pentest Process

```
Pre-Engagement → Information Gathering → Vulnerability Assessment → Exploitation
→ Post-Exploitation → Lateral Movement → Proof-of-Concept → [Post-Engagement]
```

---

## Seven Components

| Component | Description |
|-----------|-------------|
| **Cleanup** | Remove all artifacts, revert changes, notify client of anything that could not be cleaned |
| **Documentation & Reporting** | Compile all evidence into a structured DRAFT report |
| **Report Review Meeting** | Walk through findings with the client, answer questions, incorporate feedback |
| **Deliverable Acceptance** | Issue FINAL report after client feedback; DRAFT → FINAL |
| **Post-Remediation Testing** | Retest each finding after the client has applied fixes |
| **Data Retention** | Secure storage of engagement data per contract and applicable law |
| **Close Out** | Wipe tester systems, invoice, satisfaction survey, follow-on work |

---

## 1. Cleanup

Before disconnecting from the client environment and before sending the stop notification:

**What to remove:**
- All tools, scripts, and binaries uploaded to target systems
- Any web shells or backdoors planted during the engagement
- Persistence mechanisms (cron jobs, scheduled tasks, registry keys, authorized_keys entries)
- Local accounts created during testing
- Any configuration changes made (firewall rules, service settings, GPO changes)

**Documentation requirement:**
Even when cleanup is successful, document every artifact and change in the report appendices — the client may receive security alerts referencing these activities after the engagement ends and will need to confirm they were sanctioned testing.

**When cleanup is not possible:**
If a system is inaccessible (quarantined, decommissioned, unstable), alert the client explicitly and list the unresolved artifact in the report appendix with enough detail for their team to locate and remove it.

> Detailed notes taken throughout the engagement make cleanup straightforward. Trying to remember what was dropped where at the end of a multi-week engagement without notes is how artifacts get left behind.

---

## 2. Documentation and Reporting

Before sending the stop notification or disconnecting from the client network — collect everything:

**Evidence to collect before disconnecting:**
- All command outputs and terminal logs
- Screenshots for every finding
- Scan output (nmap, Nessus/OpenVAS results)
- Log output from any hosted VMs
- Compromised credential lists (hashed / anonymized appropriately)
- Any supplementary data needed for the report

**Data handling rule:** Do not retain any PII, potentially incriminating information, or sensitive data found during testing beyond what is strictly necessary for the report.

### Report Structure

| Section | Contents |
|---------|----------|
| **Attack Chain** | Full end-to-end compromise path — every step from initial access to domain compromise |
| **Executive Summary** | Non-technical — business impact, key risks, overall security posture. Written for the CEO, not the sysadmin |
| **Findings** | Each finding: risk rating, impact, remediation recommendation, external references (CVE, vendor advisory) |
| **Reproduction Steps** | Exact steps to reproduce — allows remediation team to validate the fix works |
| **Recommendations** | Near-term (patch now), medium-term (configuration changes), long-term (policy, architecture) |
| **Appendices** | Scope, OSINT data, password cracking analysis, ports/services discovered, compromised hosts/accounts, files transferred, account/system modifications, AD security analysis, scan data |

The first deliverable is always marked **DRAFT**.

---

## 3. Report Review Meeting

After the client has reviewed the DRAFT internally:

**Attendees:** Same parties as the kick-off meeting — client POC(s), technical staff responsible for remediation, penetration testing team.

**Format:**
- Do not read the report word for word — walk through findings conversationally
- For each finding: explain it in plain language, describe the impact, give context from your experience
- Let the client ask questions — they will often come with a pre-prepared list focused on the most impactful findings
- Low-risk findings rarely need detailed discussion unless the client specifically asks
- Correct any factual errors the client identifies (wrong hostnames, IP addresses, misunderstood context)
- Clarify anything the client's remediation team needs to understand before fixing

---

## 4. Deliverable Acceptance: DRAFT → FINAL

| Stage | Description |
|-------|-------------|
| **DRAFT** | First deliverable — client reviews, comments, requests clarifications or corrections |
| **Client feedback** | Via email or during the report review meeting — management responses, correction requests, additional evidence requests |
| **FINAL** | Issued after incorporating client feedback — version accepted by the client |

> Some audit frameworks (e.g. PCI-DSS audits) require a report marked FINAL — they will not accept a DRAFT designation. Maintain a uniform DRAFT → FINAL workflow across all engagements regardless of client requirements.

---

## 5. Post-Remediation Testing

After the client has applied fixes, retest every finding to confirm it was actually remediated.

**Process:**
1. Client provides documentation showing remediation actions taken (patch notes, config changes, code commits)
2. Reaccess the target environment
3. Test each finding using the same techniques used during the original assessment
4. Issue a **post-remediation report** showing before and after status

### Post-Remediation Report Table Format

| # | Severity | Finding | Status |
|---|----------|---------|--------|
| 1 | High | SQL Injection | Remediated |
| 2 | High | Broken Authentication | Remediated |
| 3 | High | Unrestricted File Upload | Remediated |
| 4 | High | Inadequate Web and Egress Filtering | Not Remediated |
| 5 | Medium | SMB Signing Not Enabled | Not Remediated |
| 6 | Low | Directory Listing Enabled | Not Remediated |

For each remediated finding: include scan output or proof that the original exploitation technique now fails.
For each not-remediated finding: include proof that the vulnerability still exists.

---

## 6. Role of the Pentester in Remediation

**The pentester is an impartial third-party auditor — not the remediation team.**

| Acceptable | Not Acceptable |
|-----------|---------------|
| Explain how a finding works | Write the fix (patched code, config file) |
| Give general remediation advice ("sanitize user input") | Provide exact rewritten code |
| Demonstrate a finding again for the remediation team | Make configuration changes in AD, servers, or applications |
| Point to remediation resources (vendor advisory, CIS benchmark) | Patch systems directly |

**Why this matters:**
- Maintaining independence preserves the integrity of the assessment
- A pentester who implements fixes and then retests their own fixes has a conflict of interest
- If the fix is wrong and an incident occurs, the firm that implemented the change carries liability

---

## 7. Data Retention

**What data is retained:**
- Scan results, log output, screenshots, evidence
- Sufficient to answer questions about findings and support post-remediation retesting

**What is never retained:**
- PII found during testing beyond what is strictly necessary
- Credentials in plaintext beyond the engagement period
- Any data that would trigger breach notification obligations if retained

**Per PCI-DSS guidance:**
> "It is a recommended best practice that the tester retain such evidence for a period of time while considering any local, regional, or company laws that must be followed for the retention of evidence."

**Storage requirements:**
- Stored on firm-owned, firm-controlled infrastructure
- Encrypted at rest
- Access controlled — not on personal devices or cloud storage outside the firm's control
- Wiped from all tester workstations at conclusion of the engagement

**Post-remediation testing:** Use a fresh VM created specifically for the client — do not reuse VMs from other engagements.

---

## 8. Close Out

Final steps after the post-remediation report is issued:

1. Wipe or destroy all systems used to connect to the client environment
2. Confirm all artifacts stored securely per contract and firm policy
3. Invoice the client
4. Collect payment
5. Send a **post-assessment satisfaction survey** — identifies what went well and what to improve, both for the firm and the individual consultant

> The client will remember how they were treated throughout the engagement far longer than they will remember the specific exploits used. Communication quality, responsiveness, and professionalism are what drive repeat business and referrals. The technical work gets you hired once — how you operate gets you hired again.

---

## The Full Pentest Process — Complete Reference

```
1. Pre-Engagement          NDA, scoping, RoE, contracts, kick-off
2. Information Gathering   OSINT, infrastructure/service/host enum, pillaging
3. Vulnerability Assessment Descriptive/diagnostic/predictive analysis, CVE research, attack vectors
4. Exploitation            Prioritization, PoC adaptation, local mirroring, logging
5. Post-Exploitation       Evasive testing, pillaging, persistence, privesc, data exfiltration
6. Lateral Movement        Pivoting, tunneling, credential reuse, PtH, Kerberoasting
7. Proof-of-Concept        Evidence, attack chain walkthrough, root cause vs symptom
8. Post-Engagement         Cleanup, reporting, review, FINAL report, remediation testing, closeout
```

---

## Key Takeaways

| Lesson | Detail |
|--------|--------|
| **Document everything during testing — not after** | Cleanup and reporting both depend on accurate real-time notes |
| **Remove all artifacts before disconnecting** | Anything left behind is a liability — for you and the client |
| **DRAFT → FINAL is not optional** | Always follow the two-stage report workflow regardless of client preference |
| **The pentester does not remediate** | Impartiality is the foundation of the audit relationship — implementing fixes destroys it |
| **Post-remediation testing is part of the engagement** | Confirming fixes actually work closes the loop — without it, the report is just a list of problems |
| **Data must be encrypted at rest and wiped at closeout** | Client data on unsecured tester machines is a security incident waiting to happen |
| **Soft skills outlast technical impressions** | Clients remember communication and professionalism long after the engagement ends |

---

## Related

- [[Proof-of-Concept]]
- [[Pre-Engagement]]
- [[Lateral Movement]]
- [[Post-Exploitation]]
