---
type: tool
category: password-cracking
tags: [password-cracking, john, offline, hashes]
source: teamghsoftware/security-cheatsheets
---

## What it is
John the Ripper (john) is an offline password hash cracker. Used after extracting hashes from `/etc/shadow`, NTLM dumps, zip files, etc.

## Core usage

```bash
# Crack a hash file with rockyou wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt <hashfile>

# Brute force (incremental mode)
john --incremental <hashfile>

# Show cracked passwords
john --show <hashfile>

# Test speed / list supported hash types
john --test

# Restore an interrupted session
john --restore:<session-name>
```

## Workflow

```bash
# 1. Extract shadow hashes (need root or /etc/shadow read access)
cat /etc/shadow

# 2. Combine passwd + shadow for john (unshadow)
unshadow /etc/passwd /etc/shadow > hashes.txt

# 3. Crack
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# 4. Show results
john --show hashes.txt
```

## Hash identification

If you don't know the hash type:
```bash
# john auto-detects, but you can also use:
hash-identifier <hash>
hashid <hash>
```

## Useful formats

```bash
john --format=NT <hashfile>        # Windows NTLM
john --format=sha512crypt <file>   # Linux SHA-512 ($6$...)
john --format=bcrypt <file>        # bcrypt ($2a$...)
john --list=formats                # show all supported formats
```

## Related
- [[hydra]]
- [[shadow]]
- [[Pentest Privesc Cheatsheet]]
