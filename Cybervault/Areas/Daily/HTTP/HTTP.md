---
type: study-note
domain: web
tags: [http, web, protocol, curl, recon]
source: HTB Academy — Web Requests module
---

## What it is

HTTP (HyperText Transfer Protocol) is the application-layer protocol that powers the web. It's a client–server protocol: a client (browser, curl, Burp) sends a **request**, a server sends back a **response**. Default port is 80; HTTPS uses 443.

As a pentester, understanding HTTP at the raw level is non-negotiable — you'll spend most of your time crafting, intercepting, and replaying HTTP requests.

---

## URL Structure

A URL is a structured address pointing to a specific resource on a server:

```
http://admin:password@inlanefreight.com:80/dashboard.php?login=true#status
 ^---^  ^-----------^ ^----------------^ ^^ ^-----------^ ^---------^ ^----^
scheme   user:pass        host           port   path        query     fragment
```

| Component | Example | Notes |
|-----------|---------|-------|
| Scheme | `http://` | Protocol identifier. Required. |
| User Info | `admin:password@` | Rarely used in modern apps — credentials in URLs are a bad practice |
| Host | `inlanefreight.com` | Domain or IP |
| Port | `:80` | Defaults: HTTP→80, HTTPS→443. Only include if non-default. |
| Path | `/dashboard.php` | File or directory on the server |
| Query String | `?login=true` | Key=value pairs, separated by `&`. Starts with `?`. Common injection point. |
| Fragment | `#status` | Processed by the **browser only** — never sent to the server. Not useful for injection. |

> **Pentest relevance:** The path and query string are where most web vulns live (SQLi, path traversal, IDOR, etc.). Always note what parameters a target accepts.

---

## HTTP Flow

1. Browser resolves the domain via DNS (checks `/etc/hosts` first — this is why we add vhosts there)
2. TCP connection to server on port 80
3. Browser sends `GET / HTTP/1.1` with a `Host:` header
4. Server responds with `200 OK` + the HTML body

The `/etc/hosts` trick is essential for HTB — target machines use vhosts, so you always start with:
```bash
sudo sh -c 'echo "10.129.x.x target.htb" >> /etc/hosts'
```

---

## cURL basics

curl is your command-line HTTP client. Essential for scripting, testing endpoints, and sending crafted requests.

```bash
# Basic GET
curl http://target.htb/

# Save response to file
curl -O http://target.htb/index.html

# Silent (no progress bar)
curl -s http://target.htb/

# Verbose — shows full request and response headers
curl -v http://target.htb/

# POST with JSON
curl -X POST http://target.htb/api/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"admin","password":"admin"}'
```

> **Personal note:** `-v` is incredibly useful when debugging — it shows exactly what curl sent and what the server said back, including redirects and headers. Use it any time a request behaves unexpectedly.

---

## Key flags quick reference

| Flag | Effect |
|------|--------|
| `-v` | Verbose: show full request + response |
| `-s` | Silent: suppress progress |
| `-k` | Skip SSL cert check |
| `-L` | Follow redirects |
| `-H 'Header: val'` | Set a header |
| `-d 'data'` | POST body |
| `-X POST` | Set HTTP method |
| `-o file` | Save to file |
| `-u user:pass` | Basic auth |
| `--proxy http://127.0.0.1:8080` | Route through Burp |

---

## Related
- [[HTTP Requests and Responses]]
- [[HTTPS]]
- [[curl]]
- [[Pentest Web Cheatsheet]]
