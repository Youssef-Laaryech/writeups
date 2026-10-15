# Lateral Movement

## What it is

Lateral movement is the **sixth phase of a penetration test** — testing how far an attacker can move through the internal network after an initial foothold is established. It answers the question a client most needs answered: not just "can you get in?" but "once inside, can you reach everything?"

The most concrete real-world parallel is ransomware: it gets into one host, moves laterally across the entire network, encrypts everything, and makes the whole company inoperable. A penetration test that finds a foothold but never tests lateral movement leaves the most dangerous attack surface completely unmeasured.

## Position in the Pentest Process

```
Pre-Engagement → Information Gathering → Vulnerability Assessment → Exploitation
→ Post-Exploitation → [Lateral Movement] → Proof-of-Concept → Post-Engagement
```

Entry point: can come from either Exploitation **or** Post-Exploitation — sometimes you cannot escalate privileges on the current host but you can reach other hosts that are vulnerable.

---

## Six Sub-Phases

| Phase | Description |
|-------|-------------|
| **Pivoting** | Route traffic through the compromised host to reach non-routable internal segments |
| **Evasive Testing** | Disguise lateral movement to avoid triggering IDS/IPS, EDR, network monitoring |
| **Information Gathering** | Re-enumerate from inside the network — new hosts, new services, new data |
| **Vulnerability Assessment** | Internal VA — far more misconfigs exist inside a network than on internet-facing systems |
| **(Privilege) Exploitation** | Use credentials, hashes, or vulnerabilities to authenticate to other systems |
| **Post-Exploitation** | Repeat the post-exploitation cycle on each newly compromised host |

---

## 1. Pivoting

The compromised host becomes a **relay** — all traffic from the attacker machine is routed through it to reach internal network segments that are not directly accessible from the internet.

```
Attacker Machine
      │
      │ (tunneled traffic)
      ▼
Compromised Host (pivot point)
      │
      ├──► Internal Subnet A (192.168.1.0/24)
      ├──► Internal Subnet B (10.10.10.0/24)
      └──► Isolated Segment (printers, OT, dev servers)
```

**Why it matters:** Most internal systems are never exposed to the internet. Without pivoting, they are invisible. With pivoting, they become scannable and attackable — and they are almost always less hardened than the perimeter.

### Common Pivoting Tools

| Tool | Method |
|------|--------|
| `ssh -D` | Dynamic SOCKS proxy over SSH — route any tool through proxychains |
| `ssh -L` / `-R` | Local / remote port forwarding — expose a specific internal port |
| `chisel` | Fast TCP/UDP tunneling over HTTP — works through web proxies |
| `ligolo-ng` | TUN interface-based tunneling — transparent routing, no proxychains needed |
| `metasploit route` | Add routes to internal subnets through a Meterpreter session |
| `sshuttle` | VPN-over-SSH — routes entire subnet traffic through the pivot |

### proxychains usage (after setting up a SOCKS proxy)

```bash
# Add to /etc/proxychains.conf:
socks5 127.0.0.1 1080

# Run any tool through the pivot:
proxychains nmap -sT -Pn 10.10.10.0/24
proxychains evil-winrm -i 10.10.10.5 -u administrator -p 'password'
```

---

## 2. Evasive Testing

Lateral movement is noisy by nature — scanning, authentication attempts, and credential reuse generate logs. Defense mechanisms to be aware of:

| Defense | What it catches |
|---------|----------------|
| Network segmentation / micro-segmentation | Limits which hosts can reach which — may block movement entirely |
| IDS/IPS | Signature-based detection of scan patterns, exploit traffic |
| EDR | Process behavior monitoring — detects credential dumping, unusual parent/child process chains |
| Threat monitoring / SIEM | Correlates authentication events, flags impossible logins, lateral movement patterns |
| Honeypots | Fake hosts — any connection triggers an immediate alert |

**Living off the land** — use tools already present on the system (`net`, `wmic`, `PowerShell`, `curl`, `ssh`) rather than dropping known-bad binaries. This reduces EDR signature hits.

---

## 3. Information Gathering (Internal Network)

Return to information gathering with a new vantage point — the internal network from inside one of its hosts.

**What to enumerate:**

| Target | Tools |
|--------|-------|
| Live hosts on all reachable subnets | `nmap`, `ping` sweep, ARP scan |
| Open ports and services on internal hosts | `nmap -sCV` through pivot |
| Active Directory structure (if domain-joined) | `bloodhound-python`, `ldapsearch`, `net` commands |
| Shares and accessible file servers | `smbclient`, `smbmap`, `net view` |
| Internal web applications | Browse through SOCKS proxy, `curl` through pivot |
| User sessions — who is logged in where | `qwinsta`, `net session`, BloodHound `HasSession` edges |
| Group memberships — what access does each user have | `ldapsearch`, BloodHound |

**Group membership intelligence:** If a compromised user account belongs to a developer group, it likely has access to dev servers, code repositories, internal wikis, and deployment pipelines — all of which contain sensitive information and potential escalation paths.

---

## 4. Vulnerability Assessment (Internal)

Internal networks are consistently more vulnerable than internet-facing systems because:

- Admins assume internal = trusted
- Patch cycles are slower for internal services
- Network services often run with excessive privileges
- Credentials are reused across many systems
- Legacy systems are kept running because "they work"

**High-value internal targets:**

| Target | Why |
|--------|-----|
| Domain Controller | Controls all authentication — compromise = full domain control |
| File servers with open shares | Credentials, sensitive documents, scripts with hardcoded passwords |
| Internal web apps (HR, ERP, ticketing) | Often unpatched, default credentials, SQLi |
| Database servers | Direct access to all application data |
| Backup servers | Often have credentials to every system they back up |
| Printers and IoT devices | Default credentials, unpatched firmware, network access |
| Dev / staging environments | Mirrored prod data, weaker controls, same credentials as prod |

---

## 5. (Privilege) Exploitation

Using gathered intelligence to authenticate to or exploit other internal systems.

### Credential-Based Movement

| Technique | Description |
|-----------|-------------|
| **Credential reuse** | Passwords found during pillaging work on other systems — especially if password policy is weak |
| **Pass-the-Hash (PtH)** | Use NTLM hash directly without cracking — authenticates to Windows services that accept NTLM |
| **Pass-the-Ticket (PtT)** | Use a stolen Kerberos ticket (`.ccache`) to authenticate |
| **Kerberoasting** | Request TGS tickets for service accounts with SPNs → crack offline → use password |
| **AS-REP Roasting** | Accounts with pre-auth disabled → get crackable hash without authentication |
| **NTLM Relay** | Capture and relay NTLM authentication to another host (`Responder` + `ntlmrelayx`) |

### Responder + Pass-the-Hash Example

```bash
# Intercept NTLMv2 hash from a network authentication event
sudo responder -I tun0

# If an admin authenticates to a fake share:
# [SMB] NTLMv2 Hash: Administrator::DOMAIN:aabbcc...:...

# Crack the NTLMv2 hash (if weak password):
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt

# Or use the NTLM hash directly for Pass-the-Hash:
impacket-psexec -hashes :<ntlm_hash> domain/administrator@<target>
```

### Vulnerability-Based Movement

When credentials are not available, exploit vulnerabilities on internal hosts:
- EternalBlue (MS17-010) — still found on unpatched internal Windows hosts
- PrintNightmare (CVE-2021-34527)
- Internal web app SQLi / RCE
- Unpatched services on internal servers

---

## 6. Post-Exploitation (Per Host)

For each new host reached via lateral movement, restart the full post-exploitation cycle:

```
New Host Compromised
      │
      ├── Establish persistence
      ├── Information gathering (local)
      ├── Pillaging (credentials, sensitive data, keys)
      ├── Privilege escalation (if not already admin)
      ├── Data exfiltration proof (if in scope)
      └── Use findings as input → next lateral movement target
```

Each cycle potentially yields new credentials, new network visibility, and new escalation paths that feed back into the next iteration.

---

## The Ransomware Parallel

The module uses ransomware as the clearest illustration of why lateral movement matters:

1. One host is compromised (phishing, unpatched service, RDP brute force)
2. Ransomware moves laterally — same techniques a pentester uses
3. It encrypts every reachable host
4. The entire company is inoperable until a ransom is paid

A pentest that finds the initial foothold but never tests lateral movement leaves this entire attack chain untested. The client's real risk is not "one host is vulnerable" — it is "one host being compromised leads to full network compromise."

> Many countries hold CEOs personally liable for failing to protect customer data. A penetration test that demonstrates the full lateral movement path from one compromised workstation to the domain controller is the evidence that drives executive-level security investment.

---

## Key Takeaways

| Lesson | Detail |
|--------|--------|
| **The goal is network-wide access, not one host** | Finding a foothold is step one — lateral movement determines the actual blast radius |
| **Internal networks are less hardened than perimeters** | Admins assume internal = safe — this is where the most exploitable misconfigs live |
| **Pivoting makes non-routable segments reachable** | Everything behind the DMZ is attackable once you have a pivot point |
| **Credential reuse is the most common lateral movement path** | One password found during pillaging often works on dozens of systems |
| **Pass-the-Hash doesn't require cracking** | NTLM hashes can authenticate directly to many Windows services |
| **Each new host restarts the post-exploitation cycle** | Every compromise is a new information gathering opportunity |

---

## Related

- [[Post-Exploitation]]
- [[Exploitation]]
- [[RBCD (Resource-Based Constrained Delegation)]]
- [[BloodHound — Active Directory Enumeration]]
- [[impacket]]
- [[Pivoting Cheatsheet]]
- [[netcat]]
