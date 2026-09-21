---
type: concept
domain: pentest
tags: [recon, enumeration, vhost, fuzzing, web]
source: Nexus HTB machine
---

## What it is
Virtual host (vhost) fuzzing is a recon technique for discovering hidden subdomains or virtual hosts on a web server. A single IP can serve multiple sites based on the `Host:` header — vhost fuzzing brute-forces that header to find sites that aren't publicly advertised via DNS.

## How it works

Web servers use the `Host:` HTTP header to decide which site to serve. If you request `http://10.10.10.1` with `Host: secret.target.htb`, the server may return a completely different site than `Host: target.htb`. Vhost fuzzing replaces the `Host` header with wordlist entries and looks for responses that differ from the default.

**Key difference from DNS subdomain brute-forcing:**
- DNS enumeration finds subdomains that exist in public DNS
- Vhost fuzzing finds virtual hosts configured on the server that may have no DNS record at all, or are only reachable via `/etc/hosts` (common in CTFs and internal networks)

## Example / commands

**ffuf — auto-calibrate to filter out the default response:**
```bash
ffuf -u http://target.htb -H 'Host: FUZZ.target.htb' \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt \
  -ac
```
`-ac` (auto-calibrate) automatically determines and filters the default response size/status, so you only see hits that are genuinely different.

**ffuf — manual size filter (when you know the default response size):**
```bash
ffuf -u http://target.htb -H 'Host: FUZZ.target.htb' \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt \
  -fs 49296
```

**In the Nexus machine:**
```bash
ffuf -u http://nexus.htb -H 'Host: FUZZ.nexus.htb' \
  -w SecLists/Discovery/DNS/subdomains-top1million-20000.txt -ac
# Found: git.nexus.htb, billing.nexus.htb
```

After finding vhosts, add them to `/etc/hosts`:
```bash
sudo sh -c 'echo "10.129.x.x git.nexus.htb billing.nexus.htb" >> /etc/hosts'
```

**gobuster vhost mode:**
```bash
gobuster vhost -u http://target.htb -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain
```

## Defense & detection

- **Don't rely on obscurity** — a vhost with no DNS record is still reachable by anyone who knows to look
- Enforce authentication on all internal/admin vhosts regardless of whether they're "hidden"
- **Rate-limit** HTTP requests to detect fuzzing attempts
- Place internal services on a separate network interface/IP, not just a different vhost on the same IP

## Related
- [[Nexus htb machine]]
- [[Git Commit History Leakage]]
- [[ffuf]]
