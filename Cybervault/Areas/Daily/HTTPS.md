---
type: study-note
domain: web
tags: [https, tls, ssl, encryption, mitm, curl]
source: HTB Academy — Web Requests module
---

## What it is

HTTPS is HTTP with TLS encryption layered on top. All data in transit is encrypted — an intercepting third party sees ciphertext, not credentials or session tokens. Default port: 443.

The lock icon in the browser address bar means TLS is active.

---

## Why it matters for pentest

**Offensive:** HTTPS doesn't mean a site is secure — it only encrypts the transport. The application layer can still be full of SQLi, file uploads, IDOR, etc. Don't assume HTTPS = safe.

**Defensive relevance you'll exploit:** If a site downgrades HTTP→HTTPS improperly, uses an expired cert, or if you're in a MITM position on the network, you can intercept traffic. Most modern browsers and HSTS make HTTP downgrade attacks harder, but this comes up in internal network assessments.

---

## TLS Handshake (simplified)

```
Client                          Server
  |--- ClientHello (ciphers) --->|
  |<-- ServerHello + Cert -------|
  |--- verify cert, key exchange->|
  |<-- encrypted session -------->|
  |=== all HTTP traffic encrypted ===
```

1. Client lists supported cipher suites
2. Server responds with its TLS certificate (contains public key)
3. Client verifies the certificate against trusted CAs
4. Session key derived — all HTTP from this point is encrypted

---

## HTTPS Flow in practice

If you visit `http://` on a site enforcing HTTPS:
- Server returns `301 Moved Permanently` → `Location: https://...`
- Browser follows, TLS handshake on port 443
- Encrypted communication begins

---

## cURL and HTTPS

curl handles HTTPS automatically. The only time it breaks is with invalid or self-signed certs:

```bash
# Fails with self-signed or expired cert:
curl https://target.htb
# curl: (60) SSL certificate problem: self-signed certificate

# Skip cert check with -k (standard for lab/HTB targets):
curl -k https://target.htb
```

> **Note:** `-k` is fine in labs. In a real engagement, an invalid cert is worth noting as a finding — it could indicate a misconfigured MITM proxy in the environment.

---

## Intercepting HTTPS with Burp

Burp acts as a local MITM proxy:
1. Set browser proxy to `127.0.0.1:8080`
2. Visit `http://burp` → download Burp's CA certificate
3. Import into browser's trusted certificate store
4. Burp can now decrypt, inspect, and modify HTTPS traffic

---

## HTTPS — what it protects and what it doesn't

| Protects | Does NOT protect |
|----------|-----------------|
| Data in transit from eavesdroppers | Application vulnerabilities (SQLi, XSS, IDOR…) |
| Passwords over the wire | Server-side logic errors |
| Session tokens from network sniffing | Stolen cookies (if `HttpOnly`/`Secure` flags are missing) |

---

## Related
- [[HTTP]]
- [[HTTP Requests and Responses]]
- [[curl]]
- [[Burp Suite]]
