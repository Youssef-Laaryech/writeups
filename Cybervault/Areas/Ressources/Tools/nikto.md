---
type: tool
category: web, recon
tags: [nikto, web-scanning, vulnerability-scan, recon]
source: teamghsoftware/security-cheatsheets
---

## What it is
Nikto is a web server scanner that checks for dangerous files, outdated software, misconfigurations, and common vulnerabilities. Noisy (not stealthy), but fast for a quick surface scan.

## Basic usage

```bash
# Scan a host (port 80 by default)
nikto -host <IP or hostname>

# Scan a specific port
nikto -host <IP> -port 8080

# Scan multiple ports
nikto -host <IP> -port 80,443,8080

# Output to file
nikto -host <IP> -output nikto_results.txt

# Use a proxy (route through Burp)
nikto -host <IP> -useproxy http://127.0.0.1:8080
```

## Display options (`-Display`)

| Flag | Shows |
|------|-------|
| `1` | Redirects |
| `2` | Cookies received |
| `3` | All 200 OK responses |
| `4` | URLs requiring authentication |
| `D` | Debug output |
| `E` | All HTTP errors |

```bash
nikto -host <IP> -Display 1234
```

## Tips

- Nikto is loud — use it when stealth doesn't matter or when you've already got a foothold
- Good for quickly finding exposed admin panels, backup files, version banners
- Pair with [[ffuf]] for directory fuzzing — nikto and ffuf complement each other

## Related
- [[ffuf]]
- [[nmap]]
- [[Pentest Recon Cheatsheet]]
