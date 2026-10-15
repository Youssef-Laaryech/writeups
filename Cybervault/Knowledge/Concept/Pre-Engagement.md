# Pre-Engagement

## What it is

Pre-engagement is the **first phase of a penetration test** — everything that happens before a single packet is sent. It establishes the legal, contractual, and operational foundation of the engagement. Skipping or rushing this phase is how pentesters end up in legal jeopardy.

## The Penetration Testing Process (Full Chain)

```
Pre-Engagement → Information Gathering → Vulnerability Assessment → Exploitation
→ Post-Exploitation → Lateral Movement → Proof-of-Concept → Post-Engagement
```

---

## Three Core Components

### 1. Scoping Questionnaire
### 2. Pre-Engagement Meeting
### 3. Kick-Off Meeting

All three are preceded by a signed **NDA**.

---

## NDA — Non-Disclosure Agreement

Must be signed before any sensitive information is discussed. Three types:

| Type | Description |
|------|-------------|
| **Unilateral** | Only one party (typically the client) is bound to confidentiality |
| **Bilateral** | Both parties are bound — most common in pentesting, protects the tester too |
| **Multilateral** | Three or more parties — used when multiple organizations are involved (e.g. cooperative networks) |

---

## Who Can Authorize a Pentest

Only specific roles have legal authority to commission a penetration test. Accepting an engagement from an unauthorized employee exposes the tester to serious legal risk.

**Typically authorized:**

| | | |
|--|--|--|
| CEO | CTO | CISO |
| CSO | CRO | CIO |
| VP of Internal Audit | Audit Manager | VP/Director of IT Security |

> Always confirm **signatory authority** early. Identify primary/secondary points of contact and escalation paths before starting.

---

## Required Documents

| Document | When Created |
|----------|-------------|
| Non-Disclosure Agreement (NDA) | After initial contact |
| Scoping Questionnaire | Before pre-engagement meeting |
| Scoping Document | During pre-engagement meeting |
| Penetration Testing Proposal / SoW | During pre-engagement meeting |
| Rules of Engagement (RoE) | Before kick-off meeting |
| Contractors Agreement *(physical only)* | Before kick-off meeting |
| Reports | During and after the test |

> All documents should be reviewed by a lawyer before use.

---

## 1. Scoping Questionnaire

Sent to the client after first contact. Determines the type, scope, and depth of the assessment.

### Assessment Types

| | |
|--|--|
| Internal Vulnerability Assessment | External Vulnerability Assessment |
| Internal Penetration Test | External Penetration Test |
| Wireless Security Assessment | Application Security Assessment |
| Physical Security Assessment | Social Engineering Assessment |
| Red Team Assessment | Web Application Security Assessment |

### Key Scoping Questions

- How many expected live hosts / IPs / CIDR ranges?
- How many domains and subdomains in scope?
- How many web/mobile applications? How many roles (standard user, admin)?
- For phishing: how many users targeted? Does the client provide the list or do we OSINT it?
- For physical: how many locations? Geographically dispersed?
- Red Team: what is the objective? What is explicitly out of scope?
- Active Directory assessment desired?
- Test from anonymous user or standard domain user?
- Does NAC need to be bypassed?

### Information Disclosure Level

| Type | Description |
|------|-------------|
| **Black box** | No information provided — tester starts blind |
| **Grey box** | IPs, CIDR ranges, URLs provided — partial knowledge |
| **White box** | Full information — architecture, source code, credentials |

### Evasiveness Level

| Level | Description |
|-------|-------------|
| **Non-evasive** | No attempt to avoid detection |
| **Hybrid-evasive** | Start quiet, gradually louder — tests at what point the SOC detects activity |
| **Fully evasive** | Maximum stealth throughout |

---

## 2. Pre-Engagement Meeting

Detailed walkthrough of the engagement with the client. Inputs from the scoping questionnaire feed into the **Penetration Testing Proposal (Contract / SoW)** and the **Rules of Engagement (RoE)**.

### Contract Checklist

| Checkpoint | Key Points |
|------------|-----------|
| NDA | Sign before or at the start of this meeting |
| Goals | Major milestones → granular sub-goals |
| Scope | Domains, IPs, hosts, accounts, security systems |
| Pentest Type | Present options with pros/cons; make a recommendation |
| Methodologies | OSSTMM, OWASP, PTES, manual + automated analysis |
| Locations | External remote (VPN) / Internal on-site or remote |
| Time Estimation | Start/end dates, phase-specific windows, business hours vs off-hours |
| Third Parties | Cloud providers, ISPs, hosting — client must obtain **written consent** from each and forward it to us |
| Evasive Testing | Confirm whether evasion techniques are in scope |
| Risks | Inform client: log entries, alarms, possible account lockouts, service disruption |
| Scope Limitations | Identify critical production systems to avoid |
| Information Handling | HIPAA, PCI-DSS, HITRUST, FISMA/NIST requirements |
| Contact Information | Name, title, email, phone, escalation order for all parties |
| Lines of Communication | Email, phone, secure messaging, meetings |
| Reporting | Structure, format, audience, presentation desired? |
| Payment Terms | Pricing and payment schedule |

---

## 3. Kick-Off Meeting

Usually in-person, after all contracts are signed. Attendees: client POC(s), client technical staff, pentest team (lead, testers, PM).

**Key items covered:**

- Walk through the pentest process end-to-end
- Confirm **no DoS testing** unless explicitly scoped
- **Critical vulnerability protocol**: if a critical flaw is found during an external test (unauthenticated RCE, SQLi leading to data exposure), testing pauses and a vulnerability notification report is sent to emergency contacts — client decides whether to patch before continuing
- **Internal test stop conditions**: system becomes unresponsive, evidence of illegal content, presence of an external threat actor / prior breach detected
- Warn the client about side effects: log noise, potential account lockouts from brute force, network impact
- Calibrate language to the least technical person in the room — C-suite and non-technical stakeholders must be able to follow

---

## Rules of Engagement (RoE) — Checklist

| Checkpoint | Contents |
|------------|---------|
| Introduction | Document purpose |
| Contractor | Company name, full name, job title |
| Penetration Testers | Names and company |
| Contact Information | All parties — mailing address, email, phone |
| Purpose | Why the test is being conducted |
| Goals | What should be achieved |
| Scope | All IPs, domains, URLs, CIDR ranges |
| Lines of Communication | Channels for updates |
| Time Estimation | Start and end dates |
| Time of Day to Test | Allowed testing windows |
| Pentest Type | External / Internal / VA / Social Engineering |
| Pentest Locations | How network connection is established |
| Methodologies | OSSTMM, PTES, OWASP, etc. |
| Objectives / Flags | Target users, files, data, systems |
| Evidence Handling | Encryption, secure transfer protocols |
| System Backups | Config files, databases |
| Information Handling | Data encryption requirements |
| Incident Handling | Contact triggers, interruption criteria, report types |
| Status Meetings | Frequency, dates, attendees |
| Reporting | Format, target audience, focus areas |
| Retesting | Dates for verification after remediation |
| Disclaimers | Liability limitations for system damage / data loss |
| Permission to Test | Signed contract reference |

---

## Contractors Agreement (Physical Assessments Only)

Physical testing involves different laws — employees may not know the test is happening and could call the police if they catch the tester. This document is the **"get out of jail free card"**.

### Checklist

- Introduction
- Contractor details
- Purpose and goal
- Pentesters listed
- Contact information
- Physical addresses, building names, floors
- Physical room identifications
- Physical components in scope
- Timeline
- Notarization
- Permission to test (signed)

---

## Key Takeaways

| Lesson | Detail |
|--------|--------|
| **Legal first, technical second** | No tools touch the network until contracts are signed — unauthorized testing is a crime regardless of intent |
| **Third-party consent is your responsibility to verify** | Cloud providers, ISPs, and hosting companies must give written consent — "the client told me it was fine" is not a defense |
| **Black/grey/white box changes everything** | Scope type determines methodology, time estimate, and cost — clarify this upfront |
| **Critical vuln protocol must be agreed in advance** | Both parties need to know exactly what happens if a critical flaw is found mid-engagement |
| **Physical testing needs its own contract** | Standard pentest agreements do not cover physical intrusion — different laws apply |
