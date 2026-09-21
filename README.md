# Writeups & Pentest Notes

Personal knowledge base documenting HackTheBox machine writeups, vulnerability research, and offensive security techniques.

---

## HackTheBox Writeups

| Machine | OS | Difficulty | Techniques |
|---------|-----|------------|------------|
| [Nexus](Cybervault/Projects/labs%20and%20machines/writeaps/nexus%20htb%20machine/Nexus%20htb%20machine.md) | Linux | Easy | Vhost fuzzing · Git secret leak · CVE-2026-38526 (PHP file upload RCE) · Path traversal via `os.path.join()` · Raw Git object crafting |

---

## Knowledge Base

### Techniques
- [Path Traversal](Cybervault/Knowledge/Concept/Path%20Traversal.md)
- [Unrestricted File Upload → RCE](Cybervault/Knowledge/Concept/Unrestricted%20File%20Upload%20RCE.md)
- [Credential Reuse](Cybervault/Knowledge/Concept/Credential%20Reuse.md)
- [Vhost Fuzzing](Cybervault/Knowledge/Concept/Vhost%20Fuzzing.md)
- [Git Commit History Leakage](Cybervault/Knowledge/Concept/Git%20Commit%20History%20Leakage.md)
- [Raw Git Object Crafting](Cybervault/Knowledge/Concept/Raw%20Git%20Object%20Crafting.md)

### Tools
- [nmap](Cybervault/Areas/Ressources/Tools/nmap.md)
- [ffuf](Cybervault/Areas/Ressources/Tools/ffuf.md)
- [Burp Suite](Cybervault/Areas/Ressources/Tools/Burp%20Suite.md)
- [curl](Cybervault/Areas/Ressources/Tools/curl.md)
- [git (pentest)](Cybervault/Areas/Ressources/Tools/git.md)

### Cheatsheets
- [Recon](Cybervault/Areas/Ressources/Commands/Pentest%20Recon%20Cheatsheet.md)
- [Web / Foothold](Cybervault/Areas/Ressources/Commands/Pentest%20Web%20Cheatsheet.md)
- [Privilege Escalation](Cybervault/Areas/Ressources/Commands/Pentest%20Privesc%20Cheatsheet.md)

---

## Structure

```
Cybervault/
├── Projects/
│   └── labs and machines/
│       ├── Machines Index.md
│       └── writeaps/          ← machine writeups
├── Knowledge/
│   └── Concept/               ← atomic technique notes
├── Areas/
│   ├── Daily/                 ← web/protocol study notes
│   └── Ressources/
│       ├── Tools/             ← one note per tool
│       └── Commands/          ← cheatsheets by phase
```

Notes are written in Markdown and link to each other — readable in any Markdown viewer or [Obsidian](https://obsidian.md/).
