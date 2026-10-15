# Information Gathering

## What it is

Information gathering is the **second phase of a penetration test** and the most recurring one — you return to it throughout the entire engagement. Every exploit, every lateral movement step, every privilege escalation attempt is built on information collected here. It is the cornerstone of the entire process.

## Position in the Pentest Process

```
Pre-Engagement → [Information Gathering] → Vulnerability Assessment → Exploitation
→ Post-Exploitation → Lateral Movement → Proof-of-Concept → Post-Engagement
```

Information gathering does not happen once — it feeds every subsequent phase and restarts after each new foothold.

---

## Four Categories

### 1. Open-Source Intelligence (OSINT)
### 2. Infrastructure Enumeration
### 3. Service Enumeration
### 4. Host Enumeration

All four must be performed on every engagement. They are not optional steps — missing any one of them leaves blind spots that break the chain.

---

## 1. Open-Source Intelligence (OSINT)

Finding publicly available information about the target without touching their infrastructure. Everything used here is legally accessible — the question is whether the target knows it is exposed.

**What you look for:**

| Category | Examples |
|----------|---------|
| Credentials | Passwords, API keys, SSH keys, tokens — leaked on GitHub, Pastebin, code commits |
| Employee data | Names, emails, roles, org structure — LinkedIn, job postings, company site |
| Infrastructure hints | Technologies, frameworks, cloud providers — job listings often name the entire stack |
| Public code | Repositories with hardcoded secrets, `.env` files, config files accidentally committed |
| Domain intel | Subdomains, MX records, historical DNS, certificate transparency logs |

**Key insight — GitHub leaks:**
Developers frequently push sensitive data to public repos and either don't notice or assume no one will find it. Private SSH keys, database passwords, API tokens, internal hostnames — all of these have been found in public commits. If found, the RoE's Incident Handling section defines what to do: stop, notify the client, wait for them to remediate before continuing.

**Tools:**
- `theHarvester` — email, subdomain, hostname harvesting from public sources
- `shodan` — internet-wide search for exposed services and banners
- `amass` / `subfinder` — subdomain enumeration
- `google dorks` — targeted search operators to find exposed files, login pages, sensitive data
- `trufflehog` / `gitleaks` — scan git repos for secrets
- Certificate Transparency logs (`crt.sh`) — find subdomains from SSL certificates

---

## 2. Infrastructure Enumeration

Mapping the client's internet and intranet presence. Goal: build an accurate picture of all servers, services, and network components — both external (internet-facing) and internal.

**What you map:**

| Asset Type | How |
|-----------|-----|
| DNS records | A, MX, NS, TXT, CNAME — `dig`, `nslookup`, zone transfer attempts |
| Name servers | Who manages DNS — potential misconfig target |
| Mail servers | MX records — useful for phishing scope |
| Web servers | HTTP/HTTPS hosts, virtual hosting, CDN detection |
| Cloud assets | S3 buckets, Azure blobs, GCP storage — often misconfigured |
| IP ranges | ASN lookups, WHOIS, BGP data |

**External vs Internal:**
- External enumeration: done from the internet, no credentials
- Internal enumeration: done after gaining a foothold — reveals services that are never exposed externally and are therefore often poorly secured

**Security measure identification:**
Infrastructure enumeration also reveals defensive controls — WAFs, IDS/IPS, load balancers, CDNs. This informs evasion strategy: knowing a Cloudflare WAF is in front of a web app changes how you test it.

**Password spraying setup:**
Internal infrastructure enumeration gives you a list of hosts and a map of user accounts. This feeds directly into **password spraying** — one password attempt per account across many accounts, staying under lockout thresholds.

---

## 3. Service Enumeration

For every host identified, determine what services are running, on which ports, at which versions, and what those services expose.

**Why version matters:**
- Old versions carry known CVEs — if the admin never patched, the public exploit still works
- Admins frequently leave outdated services running on the principle of "if it works, don't touch it" — accepting the risk rather than risking a disruption
- Version disclosure in banners is often the fastest path to a usable exploit

**What to collect per service:**

| Field | Why it matters |
|-------|---------------|
| Service name | What protocol / application |
| Version | CVE lookup, exploit search |
| Banner | Often leaks OS, config details |
| Authentication | Anonymous access? Default creds? |
| Purpose | Why does this service exist — what data does it hold? |

**Common finds:**
- FTP with anonymous access enabled — still common in internal networks
- SMB with null sessions — leaks users, shares, policies
- SNMP with default community string (`public`) — leaks full network topology
- SSH with old OpenSSH version — may be vulnerable to known exploits
- Web admin panels on non-standard ports — forgotten, unpatched

**Tools:**
- `nmap -sCV` — service/version detection + default scripts
- `nmap -sU` — UDP service scan (SNMP, TFTP, DNS)
- `whatweb` / `wappalyzer` — web technology fingerprinting
- `nikto` — web server misconfig and version checks
- `enum4linux-ng` — SMB/RPC service enumeration

---

## 4. Host Enumeration

Deep dive into each individual host in scope. Combines active scanning with OSINT to understand what the host does, what it runs, and what it communicates with.

**External host enumeration (pre-exploitation):**
- OS detection (`nmap -O`)
- Open ports and services
- Running software versions
- Network role (web server? DB? DC? file server?)
- Which other hosts it communicates with

**Internal host enumeration (post-exploitation):**
After gaining access to a host, enumeration shifts to local discovery:

| What | Why |
|------|-----|
| Sensitive files | Config files, SSH keys, `.env`, password files, database dumps |
| Local services | Services bound to 127.0.0.1 — not visible from outside, often unpatched |
| Installed software | Version info for local privilege escalation |
| Scheduled tasks / cron jobs | Writable scripts run as root/SYSTEM |
| Network connections | What the host talks to — maps the internal network |
| User accounts | Local users, password hashes, privilege levels |
| Environment variables | Often contain credentials, tokens, DB connection strings |

---

## Pillaging

A sub-component of information gathering that occurs **after exploitation** — collecting sensitive data from an already-compromised host.

> Pillaging is not a standalone phase. It is part of both Information Gathering and Post-Exploitation, performed locally on the compromised system.

**What to pillage:**

| Category | Examples |
|----------|---------|
| Credentials | Stored passwords, browser-saved creds, credential manager, SSH keys |
| Sensitive data | Customer records, internal documents, financial data |
| Configuration files | DB connection strings, API keys, cloud credentials |
| Email and communications | Outlook PST files, cached mail |
| Network information | ARP cache, routing tables, internal hostnames |
| Hashes | SAM database (Windows), `/etc/shadow` (Linux), NTDS.dit |

Pillaging data feeds directly into **Lateral Movement** — credentials found on host A are used to authenticate to host B.

---

## Key Takeaways

| Lesson | Detail |
|--------|--------|
| **Information gathering is continuous** | It does not end after the first phase — every new host you access restarts the cycle |
| **OSINT before active scanning** | Passive recon first — gather what's already public before touching the target's infrastructure |
| **GitHub is a goldmine** | Hardcoded secrets in public repos are one of the fastest paths to access — always check |
| **Internal ≠ secure** | Services not exposed to the internet are often the least hardened — admins assume isolation is protection |
| **Version numbers drive exploitation** | Service enumeration feeds directly into vulnerability assessment — the version number is the key |
| **Pillaging is information gathering from the inside** | Once you have a foothold, treat the host as a new source of intelligence for the next step |

---

## Related

- [[Pre-Engagement]]
- [[LDAP Enumeration]]
- [[BloodHound — Active Directory Enumeration]]
- [[SMB / smbmap / smbclient]]
- [[nmap]]
- [[shodan]]
- [[ffuf]]
