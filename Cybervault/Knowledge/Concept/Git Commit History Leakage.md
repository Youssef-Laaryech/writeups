---
type: concept
domain: pentest
tags: [recon, git, secret-leakage, credential-leak]
source: Nexus HTB machine
---

## What it is
Git commit history leakage occurs when sensitive data (passwords, API keys, tokens) is committed to a Git repository and then "removed" in a later commit. The data remains fully accessible in the commit history as long as the repository hasn't been rewritten — which is almost always.

## How it works

Git is an append-only data structure by design. When you delete a file or change a value and commit, the old version is not deleted — it's preserved in the DAG (directed acyclic graph) of commits forever. Anyone with read access to the repository (including anonymous access to public repos) can browse or clone the full history and extract any previously committed value.

**The common developer mistake:**
```bash
# Developer commits .env with real password
git add .env && git commit -m "initial setup"

# Realizes password is exposed, removes it
echo "DB_PASSWORD=" > .env
git add .env && git commit -m "remove password"

# The password is still visible in the first commit — nothing was erased
```

## Example / commands

**Browsing history via Gitea/GitHub web UI:**
- Navigate to the repo → click "Commits" → find commits with messages like "remove", "fix config", "oops"
- Click any commit to see the full diff, including removed lines (shown in red)

**In the Nexus machine:**
- Public repo `admin/krayin-docker-setup` on Gitea
- Latest commit had `DB_PASSWORD=` (empty)
- Earlier commit showed `DB_PASSWORD=N27xh!!2ucY04` in the diff

**CLI — view all commits and diffs:**
```bash
git clone http://target/repo.git
cd repo
git log --oneline          # list all commits
git show <commit_hash>     # show full diff of a specific commit
git log -p                 # show diffs for all commits
```

**Search git history for secrets:**
```bash
git log -p | grep -i "password\|secret\|key\|token\|api"
```

**truffleHog — automated secret scanning in git history:**
```bash
trufflehog git https://github.com/target/repo --json
```

**gitleaks:**
```bash
gitleaks detect --source /path/to/repo
```

## Defense & detection

- **Never commit real secrets** — use environment variables or secret managers; add `.env` to `.gitignore` before the first commit
- **If a secret was committed:** rewriting history with `git filter-repo` (preferred) or BFG Repo Cleaner removes it from commits, but you must also rotate the exposed credential immediately — anyone could have cloned it already
- **Pre-commit hooks** — tools like `git-secrets` or `detect-secrets` can block commits containing secret patterns
- **GitHub/GitLab secret scanning** — both platforms automatically scan push content for known secret formats

## Related
- [[Nexus htb machine]]
- [[Credential Reuse]]
- [[Vhost Fuzzing]]
