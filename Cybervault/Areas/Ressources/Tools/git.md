---
type: tool
category: recon, exploitation
tags: [git, recon, secret-leakage, raw-objects, plumbing]
---

## What it is
Git is a version control system, but in pentest contexts it's also an attack surface. Exposed `.git` directories, public repos with credential history, and low-level Git plumbing commands are all relevant for recon and exploitation.

## Recon use cases

**Clone a public repo and inspect it:**
```bash
git clone http://target.htb/repo.git
cd repo
```

**Browse full commit history:**
```bash
git log --oneline          # compact list of all commits
git log -p                 # full diffs for every commit
git log -p --all           # include all branches/refs
```

**Search history for secrets:**
```bash
git log -p | grep -iE "password|secret|key|token|api_key|db_pass"
git show <commit_hash>     # see the full diff of one commit
```

**List all files ever tracked (even deleted ones):**
```bash
git log --all --full-history -- "*.env"
git show HEAD~1:.env       # show .env from the previous commit
```

## Plumbing commands (low-level)

These bypass the porcelain validation that `git add`/`git commit` enforce — relevant for the raw object crafting technique:

```bash
git cat-file -p <sha1>     # pretty-print any object (blob/tree/commit)
git cat-file -t <sha1>     # show type of object
git ls-tree -r HEAD        # list all files in the current commit recursively
git hash-object -w file    # write a file as a blob into .git/objects
git mktree                 # create a tree object from stdin entries
git update-ref refs/heads/main <sha1>   # point branch at a commit
```

## Exposed .git directory

If a web server exposes `/.git/`, you can reconstruct the full repo:

```bash
# Manual download
curl http://target.htb/.git/config
curl http://target.htb/.git/HEAD

# Automated tools
git-dumper http://target.htb/.git/ ./output/
# or
gittools/gitdumper.sh http://target.htb/.git/ dump/
```

## Useful attack tools

| Tool | Purpose |
|------|---------|
| `truffleHog` | Scan git history for high-entropy strings / known secret patterns |
| `gitleaks` | Fast secret scanner for git repos |
| `git-dumper` | Reconstruct repo from exposed `/.git/` |
| `gitjacker` | Similar to git-dumper |

## Related
- [[Git Commit History Leakage]]
- [[Raw Git Object Crafting]]
- [[Nexus htb machine]]
