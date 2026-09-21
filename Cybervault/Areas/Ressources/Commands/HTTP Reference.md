---
type: cheatsheet
phase: web
tags: [http, status-codes, headers, reference, cheatsheet]
source: teamghsoftware/security-cheatsheets (adapted)
---

## HTTP Authentication Types

| Type | Notes |
|------|-------|
| Basic Auth | Base64-encoded creds in header — trivially reversible, use over HTTPS only |
| Digest Auth | MD5 challenge-response, susceptible to MITM |
| Bearer / Token | JWT or API token in `Authorization: Bearer <token>` header |
| Form-Based | Cookie/session after POST login — most common in web apps |
| Integrated Windows (NTLM) | Will not work over proxy; mainly internal AD environments |

---

## HTTP Status Codes — Quick Reference

### 1xx — Informational
| Code | Meaning |
|------|---------|
| 100 | Continue |
| 101 | Switching Protocols |

### 2xx — Success
| Code | Meaning | Pentest note |
|------|---------|-------------|
| 200 | OK | Success |
| 201 | Created | Resource was created (PUT/POST) |
| 204 | No Content | Success, no body (common on DELETE) |

### 3xx — Redirection
| Code | Meaning | Pentest note |
|------|---------|-------------|
| 301 | Moved Permanently | Follow `Location:` header — may reveal internal path |
| 302 | Found (temporary redirect) | Common after login — check where it redirects |
| 307 | Temporary Redirect | Method preserved |
| 308 | Permanent Redirect | Method preserved |

### 4xx — Client Error
| Code | Meaning | Pentest note |
|------|---------|-------------|
| 400 | Bad Request | Malformed syntax — unencoded special chars in request |
| 401 | Unauthorized | Auth required — try default creds |
| 403 | Forbidden | Auth OK but access denied — look for bypass |
| 404 | Not Found | Resource missing — or hidden; check other methods |
| 405 | Method Not Allowed | Try different HTTP verb |
| 408 | Request Timeout | |
| 419 | Page Expired (Laravel) | Stale CSRF token — grab fresh one |
| 429 | Too Many Requests | Rate limiting active |

### 5xx — Server Error
| Code | Meaning | Pentest note |
|------|---------|-------------|
| 500 | Internal Server Error | App crashed — may expose stack trace (info disclosure) |
| 501 | Not Implemented | |
| 502 | Bad Gateway | Proxy/upstream issue |
| 503 | Service Unavailable | |

---

## Common HTTP Headers

### Request headers
| Header | Example | Notes |
|--------|---------|-------|
| `Host` | `target.htb` | Determines which vhost is served |
| `User-Agent` | `Mozilla/5.0` | Can be forged to bypass WAF rules |
| `Cookie` | `session=abc123` | Session tokens live here |
| `Content-Type` | `application/json` | Must match body format |
| `Authorization` | `Bearer <token>` | API auth |
| `X-Forwarded-For` | `127.0.0.1` | May bypass IP-based restrictions |
| `Referer` | `http://target.htb/` | Some apps validate this |
| `Origin` | `http://target.htb` | CORS checks |

### Response headers (interesting ones)
| Header | What to look for |
|--------|-----------------|
| `Server` | Reveals web server + version |
| `X-Powered-By` | Reveals framework/language |
| `Set-Cookie` | Check for `HttpOnly`, `Secure`, `SameSite` flags |
| `Content-Security-Policy` | Check for misconfigs (XSS relevance) |
| `Access-Control-Allow-Origin` | CORS config — `*` is dangerous |
| `Location` | Redirect target — often reveals internal paths |

## Related
- [[HTTP]]
- [[HTTP Requests and Responses]]
- [[curl]]
- [[Burp Suite]]
- [[Pentest Web Cheatsheet]]
