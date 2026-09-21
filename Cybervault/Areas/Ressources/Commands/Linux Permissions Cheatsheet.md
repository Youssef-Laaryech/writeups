---
type: cheatsheet
phase: post-exploitation, privesc
tags: [linux, permissions, chmod, suid, privesc, cheatsheet]
source: teamghsoftware/security-cheatsheets (adapted)
---

## File permission structure

```
- rw- r-- r--
^ ^-^ ^-^ ^-^
|  |   |   └── other (o) permissions
|  |   └────── group (g) permissions
|  └────────── user/owner (u) permissions
└───────────── file type (- = file, d = directory, l = symlink)
```

Each permission set: `r` (read=4), `w` (write=2), `x` (execute=1)

## chmod

```bash
# Octal notation (most common)
chmod 755 file    # rwx r-x r-x
chmod 644 file    # rw- r-- r--
chmod 777 file    # rwx rwx rwx

# Symbolic notation
chmod u+x file    # add execute for owner
chmod o-w file    # remove write for others
chmod a+r file    # add read for all
chmod g=rx file   # set group to r-x exactly
```

## Common permission values

| Octal | Symbolic | Meaning |
|-------|----------|---------|
| 777 | rwxrwxrwx | Full access for everyone — dangerous |
| 755 | rwxr-xr-x | Owner full, others read+execute — standard for executables |
| 644 | rw-r--r-- | Owner read+write, others read — standard for files |
| 600 | rw------- | Owner only — SSH keys, sensitive configs |
| 700 | rwx------ | Owner only, executable |
| 400 | r-------- | Read-only for owner — e.g. `/etc/shadow` |

## SUID / SGID (privesc relevance)

```bash
# SUID (4xxx) — file runs as owner (often root), not the caller
chmod 4755 file   # sets SUID
ls -la            # shows as: -rwsr-xr-x (s in owner execute position)

# Find all SUID binaries on a system (privesc enumeration)
find / -perm -4000 -type f 2>/dev/null

# Find all SGID binaries
find / -perm -2000 -type f 2>/dev/null

# Find world-writable files (potential hijacking)
find / -perm -0002 -type f 2>/dev/null
```

## /etc/shadow format

```
username:$id$salt$hash:days-last-changed:min:max:warn:inactive:expire:reserved
```

Hash type identifiers:
| ID | Algorithm |
|----|-----------|
| `$1$` | MD5 |
| `$2a$` | Blowfish |
| `$5$` | SHA-256 |
| `$6$` | SHA-512 (most common on modern Linux) |

```bash
# Unshadow for John
unshadow /etc/passwd /etc/shadow > crackme.txt
john --wordlist=/usr/share/wordlists/rockyou.txt crackme.txt
```

## chown / chgrp

```bash
chown user:group file
chown root:root /tmp/setuid_binary
```

## Related
- [[john]]
- [[Pentest Privesc Cheatsheet]]
