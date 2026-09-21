---
type: study-note
domain: web
tags: [http, requests, responses, headers, status-codes, curl, devtools]
source: HTB Academy — Web Requests module
---

## HTTP Request Structure

```
GET /users/login.html HTTP/1.1     <- request line: method + path + version
Host: inlanefreight.com            <- headers (key: value)
User-Agent: Mozilla/5.0
Cookie: PHPSESSID=abc123
                                   <- blank line = end of headers
username=admin&password=test       <- body (POST/PUT only)
```

**Request line fields:**

| Field | Example | Notes |
|-------|---------|-------|
| Method | `GET` | The action to perform |
| Path | `/users/login.html` | Resource + optional `?query=string` |
| Version | `HTTP/1.1` | 1.x = clear-text; 2 = binary |

**Common HTTP methods — pentest relevance:**

| Method | Use | Pentest note |
|--------|-----|-------------|
| GET | Fetch resource | Params in URL — logged, easy to test |
| POST | Send data | Params in body — logins, uploads, file submissions |
| PUT | Create/replace | Abuse for file uploads on misconfigured servers |
| DELETE | Remove resource | Dangerous if unauthenticated |
| OPTIONS | List allowed methods | `curl -X OPTIONS /endpoint` — reveals attack surface |
| HEAD | Headers only | Quick recon without downloading the body |

> **Tip:** Always check what HTTP methods are enabled on upload or API endpoints. A PUT where only POST is expected can be an unintended upload path.

---

## HTTP Response Structure

```
HTTP/1.1 200 OK                         <- status line
Date: Mon, 21 Sep 2026 12:00:00 GMT     <- response headers
Server: Apache/2.4.41
Set-Cookie: PHPSESSID=xyz; HttpOnly
Content-Type: text/html; charset=UTF-8
                                        <- blank line
<!DOCTYPE html>...                      <- response body
```

**Status codes that matter in pentest:**

| Code | Meaning | Pentest relevance |
|------|---------|------------------|
| 200 | OK | Success |
| 301/302 | Redirect | Follow `Location:` header — often leaks internal paths |
| 400 | Bad Request | Unencoded special chars in URL broke the request |
| 401 | Unauthorized | Auth required — try default creds or bypass |
| 403 | Forbidden | Auth OK, access denied — look for bypasses |
| 404 | Not Found | Resource missing — try other HTTP methods |
| 405 | Method Not Allowed | Try a different verb (GET→POST, etc.) |
| 419 | Page Expired (Laravel) | Stale CSRF token — capture fresh request |
| 500 | Server Error | App crashed — may expose stack traces |

---

## Viewing raw requests with curl -v

```bash
curl -v http://target.htb/
```

- Lines with `>` = what curl sent
- Lines with `<` = what the server replied
- Lines with `*` = curl status messages (connection, SSL, redirects)

Fastest way to see the full HTTP exchange without Burp.

---

## Browser DevTools — Network Tab

`F12` → Network tab:
- Every request the page makes (including background XHR/fetch calls)
- Full request/response headers
- Response body with raw view
- Right-click any request → **Copy as cURL** — grabs the complete request with cookies and CSRF tokens, ready to paste in terminal or Burp

> **Personal note:** "Copy as cURL" is a massive time-saver. It captures everything — session cookies, CSRF tokens, headers — in one action. Far faster than manually reconstructing a request, and it sidesteps the stale CSRF token problem entirely.

---

## HTTP Headers worth knowing

| Header | Direction | What to look for |
|--------|-----------|-----------------|
| `Host` | Request | Vhost routing — changing this is how vhost fuzzing works |
| `Cookie` | Request | Session tokens — grab these from DevTools |
| `Authorization` | Request | Bearer tokens, Basic auth credentials |
| `X-Forwarded-For` | Request | May bypass IP allowlists if server trusts it blindly |
| `Server` | Response | Web server + version — info disclosure |
| `X-Powered-By` | Response | Framework/language — info disclosure |
| `Set-Cookie` | Response | Check for missing `HttpOnly`, `Secure`, `SameSite` flags |
| `Location` | Response | Redirect destination — often reveals internal paths |

---

## Related
- [[HTTP]]
- [[HTTPS]]
- [[curl]]
- [[Burp Suite]]
- [[HTTP Reference]]
- [[Pentest Web Cheatsheet]]
