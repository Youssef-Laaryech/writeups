---
type: concept
domain: pentest
tags: [web, path-traversal, privesc, file-write]
source: Nexus HTB machine
---

## What it is
Path traversal (also called directory traversal) is a vulnerability where an attacker manipulates a file path input — using sequences like `../` — to access files or directories outside the intended base directory.

## How it works

A vulnerable application constructs a file path by concatenating a base directory with user-controlled input, without sanitizing `..` components. The attacker walks up the directory tree to reach arbitrary files.

**Python-specific gotcha — `os.path.join()`:**
Python's `os.path.join()` does NOT strip `..` components. If the user-controlled segment starts with `/`, it even completely discards the base path. Both cases allow escaping the intended directory.

```python
# Vulnerable — no sanitization
base = "/var/app/uploads/"
user_input = "../../../../etc/passwd"
target = os.path.join(base, user_input)
# target = "/var/app/uploads/../../../../etc/passwd"
# resolves to: /etc/passwd
```

**Safe fix — normalize and validate:**
```python
import os

base = os.path.realpath("/var/app/uploads/")
target = os.path.realpath(os.path.join(base, user_input))
if not target.startswith(base + os.sep):
    raise ValueError("Path traversal detected")
```

## Example / commands

**In the Nexus machine:** A root-owned systemd service ran a Python sync script that wrote files using:
```python
target = os.path.join(stage_path, filepath)
```
where `filepath` came directly from `git ls-tree` output. By crafting a Git tree containing a file named `../../../../root/.ssh/authorized_keys`, the service wrote an attacker-controlled SSH public key to root's authorized_keys file.

**Classic LFI via URL:**
```
GET /view?file=../../../../etc/passwd
GET /view?file=..%2F..%2F..%2Fetc%2Fshadow
```

**File read in PHP:**
```php
// Vulnerable
$file = $_GET['file'];
include("/var/www/html/pages/" . $file);
```

## Defense & detection

- **Normalize paths** with `realpath()` / `os.path.realpath()` before use, then assert the result starts with the intended base
- **Allowlist** known-good file names rather than blocklisting `..`
- **Avoid passing user input directly** to file system functions
- **WAF rules** — detect `../`, `..%2F`, `%2e%2e%2f`, double-encoding variants
- **Monitoring** — alert on reads to sensitive paths (`/etc/passwd`, `~/.ssh/`)

## Related
- [[Nexus htb machine]]
- [[Raw Git Object Crafting]]
- [[Unrestricted File Upload RCE]]
