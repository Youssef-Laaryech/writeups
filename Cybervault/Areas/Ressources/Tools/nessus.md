# Nessus

## What it is

Nessus is the industry-standard **commercial vulnerability scanner** developed by Tenable. It automatically scans hosts for known vulnerabilities, misconfigurations, default credentials, missing patches, and compliance violations — producing a prioritized, actionable report. It is the most widely used vulnerability assessment tool in real-world pentests and corporate security programs.

**Nessus Essentials** (formerly Nessus Home) is free for up to 16 IPs — enough for lab and CTF use.

## Nessus vs a Pentest Tool

| | Nessus | nmap / Metasploit |
|---|---|---|
| What it does | Checks hosts against a database of 100,000+ known CVEs and misconfigs | Actively exploits or probes |
| How it works | Automated, plugin-based — each plugin tests for one specific issue | Manual / scripted |
| Output | Severity-ranked findings with CVE IDs, CVSS scores, remediation steps | Raw results you interpret yourself |
| Limitation | Cannot adapt attacks to the target — misses logic bugs, chained exploits | Requires human judgment |

Nessus tells you **what** is potentially vulnerable. A human tester decides **whether** it is actually exploitable and **what** the real-world impact is.

## Installation (Kali / Ubuntu)

```bash
# Download .deb from https://www.tenable.com/downloads/nessus
sudo dpkg -i Nessus-<version>-debian10_amd64.deb
sudo systemctl start nessusd
# Browse to https://localhost:8834
```

Activate with a free Nessus Essentials key (register at tenable.com — no credit card needed).

## Key Concepts

### Plugins
Each Nessus plugin is a small script that tests for one specific vulnerability or configuration issue. Nessus ships with 100,000+ plugins, updated daily. When a scan runs, Nessus selects relevant plugins based on the detected OS and services.

### Scan Policies
A scan policy defines which plugins run, how aggressive the scan is, and what credentials (if any) to use. Key policies:

| Policy | Use case |
|--------|----------|
| **Basic Network Scan** | Unauthenticated — discovers open services and checks for network-visible vulns |
| **Advanced Scan** | Full plugin set — more thorough, noisier |
| **Credentialed Scan** | Provide SSH/WinRM/SMB creds — Nessus logs in and checks patch levels, installed software, config files. Far more accurate than unauthenticated. |
| **Web Application Tests** | SQL injection, XSS, common web vulns |
| **Compliance Audit** | Checks config against CIS Benchmarks, PCI-DSS, HIPAA |

### Severity Ratings (CVSS-based)

| Level | CVSS Score | Meaning |
|-------|-----------|---------|
| Critical | 9.0 – 10.0 | Exploitable remotely, often unauthenticated — fix immediately |
| High | 7.0 – 8.9 | Significant risk — prioritize |
| Medium | 4.0 – 6.9 | Often requires local access or chaining |
| Low | 0.1 – 3.9 | Informational / minor |
| Info | N/A | Service discovery, banners — no vulnerability |

## Basic Workflow

1. **New Scan** → choose policy (Basic Network Scan for most cases)
2. Set target: IP, range (`192.168.1.0/24`), or hostname
3. *(Optional)* Add credentials under the **Credentials** tab for a much deeper scan
4. Launch → wait (5–30 min depending on scope)
5. Review findings sorted by severity
6. Export report as PDF/HTML/CSV for documentation

## Credentialed Scanning (much more valuable)

```
Credentials tab → SSH:
  Authentication method: password or public key
  Username: <user>
  Password: <pass>  (or upload private key)
```

A credentialed scan can check:
- Exact patch levels (`uname -r`, `dpkg -l`, Windows Update history)
- Installed software versions
- File permissions and world-writable paths
- `/etc/passwd`, sudoers misconfigs
- Hardcoded credentials in common config files

## Pentest Use Cases

- **Pre-engagement recon** — run a credentialed scan on in-scope hosts to quickly surface low-hanging fruit before manual testing
- **Post-exploitation** — scan internal segments after pivoting to map the internal attack surface
- **Compliance check** — validate hardening before a client's audit
- **Patch gap analysis** — identify missing patches that are exploitable with public PoCs

## Limitations

- **No exploit chaining** — Nessus flags individual CVEs but cannot find multi-step attack paths
- **False positives** — version-based detection sometimes flags patched systems if the vendor didn't change the version string
- **No logic vulnerability detection** — IDOR, business logic flaws, auth bypasses are invisible to automated scanners
- **Noisy** — credentialed scans generate significant log activity; not suitable for stealth engagements

## Key Files and Ports

| Item | Detail |
|------|--------|
| Web UI | `https://localhost:8834` |
| Service | `nessusd` |
| Config dir | `/opt/nessus/` |
| Plugin dir | `/opt/nessus/lib/nessus/plugins/` |
| Scan data | `/opt/nessus/var/nessus/` |

## Related

- [[OpenVAS]]
- [[Qualys]]
- [[nmap]]
