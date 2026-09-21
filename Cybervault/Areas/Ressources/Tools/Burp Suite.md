---
type: tool
category: web
tags: [web, proxy, burp, intercept, repeater]
---

## What it is
Burp Suite is an intercepting proxy for web application testing. You route browser traffic through it to inspect, modify, and replay HTTP/S requests. The Community edition covers most CTF needs.

## Key tabs

| Tab | Purpose |
|-----|---------|
| **Proxy > Intercept** | Pause and modify requests in real-time before they hit the server |
| **Proxy > HTTP History** | Log of all traffic that passed through Burp |
| **Repeater** | Send a single request repeatedly with modifications — main tool for manual testing |
| **Intruder** | Automated fuzzing / brute force (rate-limited in Community) |
| **Decoder** | Encode/decode URL, Base64, HTML, hex interactively |
| **Comparer** | Diff two requests or responses |
| **Target > Site map** | Tree view of all discovered endpoints |

## Setup

**Browser proxy config:**
- Set browser to use `127.0.0.1:8080` as HTTP/HTTPS proxy
- Or use FoxyProxy extension for quick toggling

**Install Burp CA cert (for HTTPS interception):**
1. With Burp proxy on, visit `http://burp` in browser
2. Download `cacert.der`
3. Import into browser's certificate store as a trusted CA

**Route curl through Burp:**
```bash
curl -k --proxy http://127.0.0.1:8080 https://target.htb/api/
```

## Repeater workflow

1. Find a request in Proxy > HTTP History
2. Right-click → **Send to Repeater** (or `Ctrl+R`)
3. Modify the request in the left pane
4. Hit **Send** — see the response on the right
5. Iterate

**Key shortcuts:**
- `Ctrl+R` — Send to Repeater
- `Ctrl+U` — URL-encode selected text (critical for payloads in the request line)
- `Ctrl+Shift+U` — URL-decode selected text
- `Ctrl+I` — Send to Intruder

## Common pitfalls

**419 Page Expired (Laravel CSRF):**
CSRF tokens (`_token`, `XSRF-TOKEN`) are tied to the session and expire fast. Never reuse an old saved request. Always capture a fresh request from the live browser right before replaying.

**400 Bad Request on GET with special chars:**
Spaces, `&`, `>`, `'` in the URL/query string must be URL-encoded. Select the value → `Ctrl+U` to encode.

**HTTPS interception failing:**
Burp CA cert not installed in the browser's trust store. Re-import it.

## Related
- [[Unrestricted File Upload RCE]]
- [[HTTP Requests and Responses]]
- [[curl]]
- [[Pentest Web Cheatsheet]]
