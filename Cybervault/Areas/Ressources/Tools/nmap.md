---
type: tool
category: recon
tags: [recon, enumeration, port-scan, nmap]
---

## What it is
nmap (Network Mapper) is the standard port scanner for recon. It discovers open ports, running services, versions, and OS details. Essential first step on any target.

## Core flags

| Flag | Meaning |
|------|---------|
| `-sC` | Run default NSE scripts (equivalent to `--script=default`) |
| `-sV` | Probe open ports to detect service version |
| `-sS` | SYN scan (stealth, requires root) |
| `-sU` | UDP scan |
| `-p-` | Scan all 65535 ports |
| `-p 22,80,443` | Scan specific ports |
| `-T4` / `-T5` | Timing template — faster scans (less accurate on unstable networks) |
| `-A` | Aggressive: OS detection + version + scripts + traceroute |
| `-oN file` | Save output in normal format |
| `-oG file` | Save output in greppable format |
| `-oA basename` | Save in all formats (`.nmap`, `.gnmap`, `.xml`) |
| `--open` | Only show open ports |
| `-v` / `-vv` | Verbose output |
| `--min-rate 5000` | Send packets no slower than 5000/s |

## Typical workflow

**Step 1 — Quick full port scan (find everything open):**
```bash
nmap -p- --min-rate 5000 -T4 <target> -oN ports.txt
```

**Step 2 — Deep scan on open ports only:**
```bash
nmap -sCV -p 22,80,443 <target> -oN detailed.txt
```

**One-liner for HTB/CTF:**
```bash
nmap -sCV <target>
```

## Useful NSE scripts

```bash
# HTTP enumeration
nmap --script http-enum -p 80 <target>

# Vulnerability scan
nmap --script vuln -p 80,443 <target>

# SMB enumeration
nmap --script smb-enum-shares,smb-enum-users -p 445 <target>

# SSH auth methods
nmap --script ssh-auth-methods -p 22 <target>
```

## Reading output

- `open` — port is accessible and a service is listening
- `filtered` — firewall is blocking the probe (port may be open)
- `closed` — port is reachable but nothing is listening

## Related
- [[Pentest Recon Cheatsheet]]
- [[ffuf]]
