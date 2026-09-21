---
type: concept
domain: pentest
tags: [credential-reuse, lateral-movement, password-spraying]
source: Nexus HTB machine
---

## What it is
Credential reuse is when a password (or hash) discovered in one context — a config file, a database, a Git repo — also works for a different account or service. It's one of the most reliable lateral movement techniques because people and systems routinely reuse passwords.

## How it works

1. You obtain a credential from somewhere: commit history, `.env`, database dump, config file, etc.
2. You identify other services or accounts (SSH, SMB, web panel, database) where that user or email might have an account
3. You try the credential across those services — often it works

**Why it's so effective:**
- Developers frequently reuse database passwords for their system user account
- Docker `.env` files often contain both app credentials and the developer's own password
- Services share passwords because they were configured by the same person

## Example / commands

**In the Nexus machine — two separate reuse chains:**

1. **Git commit history → CRM login:**
   - `DB_PASSWORD=N27xh!!2ucY04` leaked from Gitea commit history
   - Used as the login password for `j.matthew@nexus.htb` on Krayin CRM → success

2. **`.env` on disk → SSH:**
   - After getting a shell as `www-data`, read `/var/www/krayin/.env`
   - Found `DB_PASSWORD=y27xb3ha!!74GbR`
   - Tried `ssh jones@nexus.htb` with that password → success (user flag)

**Spraying found credentials across services:**
```bash
# SSH
ssh <user>@<target>

# Check /etc/passwd for valid users first
cat /etc/passwd | grep -v nologin | grep -v false

# Try credential on other web services / APIs
curl -u 'user:password' http://target/api/
```

## Defense & detection

- Use a **password manager** — never reuse passwords across services
- Store secrets in **environment-specific secret managers** (Vault, AWS Secrets Manager), not in `.env` files committed to repos
- **Rotate credentials** after any exposure
- **Monitor for credential stuffing** — alert on logins from unusual IPs or rapid sequential attempts
- Enable **MFA** on all externally accessible services

## Related
- [[Nexus htb machine]]
- [[Git Commit History Leakage]]
- [[Unrestricted File Upload RCE]]
