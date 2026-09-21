---
type: tool
category: recon
tags: [recon, fuzzing, web, vhost, directory-brute]
---

## What it is
ffuf (Fuzz Faster U Fool) is a fast web fuzzer for directory/file discovery, vhost enumeration, and parameter fuzzing. The `FUZZ` keyword marks where the wordlist value is injected.

## Core flags

| Flag | Meaning |
|------|---------|
| `-u` | Target URL (put `FUZZ` where you want to inject) |
| `-w` | Wordlist path |
| `-H` | Add/override a header |
| `-X` | HTTP method (default: GET) |
| `-d` | POST body data |
| `-ac` | Auto-calibrate — auto-filter the default response |
| `-fc` | Filter by HTTP status code (e.g. `-fc 404`) |
| `-fs` | Filter by response size in bytes |
| `-fw` | Filter by word count |
| `-fl` | Filter by number of lines |
| `-mc` | Match by status code (e.g. `-mc 200,301`) |
| `-e` | Extensions to append (e.g. `-e .php,.html,.txt`) |
| `-t` | Threads (default 40) |
| `-o` | Output file |
| `-of` | Output format (`json`, `csv`, `html`) |
| `-v` | Verbose — show full URLs in output |
| `-r` | Follow redirects |

## Use cases

**Directory / file fuzzing:**
```bash
ffuf -u http://target.htb/FUZZ \
  -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -e .php,.html,.txt -ac
```

**Vhost fuzzing:**
```bash
ffuf -u http://target.htb -H 'Host: FUZZ.target.htb' \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt \
  -ac
```
> `-ac` is the key flag here — without it you'll drown in false positives since every vhost returns *something*.

**GET parameter fuzzing:**
```bash
ffuf -u 'http://target.htb/page.php?FUZZ=value' \
  -w /opt/SecLists/Discovery/Web-Content/burp-parameter-names.txt -ac
```

**POST body fuzzing:**
```bash
ffuf -u http://target.htb/login -X POST \
  -d 'username=admin&password=FUZZ' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -w /opt/SecLists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt \
  -fc 200
```

**Multiple wordlists (W1, W2 placeholders):**
```bash
ffuf -u http://target.htb/W1/W2 \
  -w wordlist1.txt:W1 -w wordlist2.txt:W2 -ac
```

## Tips

- Always try `-ac` first. If it over-filters, switch to manual `-fs` / `-fc`
- For vhosts, `-ac` works well because the default (no match) response is consistent
- Use `-e .php,.txt` for web targets running PHP — saves a separate run
- Pair with `-o results.json -of json` when you want to process results later

## Related
- [[Vhost Fuzzing]]
- [[Pentest Recon Cheatsheet]]
- [[nmap]]
