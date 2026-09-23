# Writeups & Pentest Notes

Personal knowledge base documenting HackTheBox machine writeups, vulnerability research, and offensive security techniques.

---

## HackTheBox Writeups

| Machine | OS | Difficulty | Techniques |
|---------|-----|------------|------------|
| [Nexus](Cybervault/Projects/labs%20and%20machines/writeaps/nexus%20htb%20machine/Nexus%20htb%20machine.md) | Linux | Easy | Vhost fuzzing · Git secret leak · CVE-2026-38526 (PHP file upload RCE) · Path traversal via `os.path.join()` · Raw Git object crafting |
| [Silentium](Cybervault/Projects/labs%20and%20machines/writeaps/silentium%20htb%20machine/silentium%20writeup.md) | Linux | Medium | Subdomain fuzzing · CVE-2025-58434 (Flowise ATO via token leak) · CVE-2025-59528 (Flowise CustomMCP RCE) · Container env credential leak · CVE-2025-8110 (Gogs symlink traversal → sshCommand injection → root) |

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
- [netcat](Cybervault/Areas/Ressources/Tools/netcat.md)
- [tcpdump](Cybervault/Areas/Ressources/Tools/tcpdump.md)
- [cdk](Cybervault/Areas/Ressources/Tools/cdk.md)
- [sqlmap](Cybervault/Areas/Ressources/Tools/sqlmap.md)
- [hydra](Cybervault/Areas/Ressources/Tools/hydra.md)
- [metasploit](Cybervault/Areas/Ressources/Tools/metasploit.md)
- [msfvenom](Cybervault/Areas/Ressources/Tools/msfvenom.md)
- [nikto](Cybervault/Areas/Ressources/Tools/nikto.md)
- [john](Cybervault/Areas/Ressources/Tools/john.md)
- [wireshark](Cybervault/Areas/Ressources/Tools/wireshark.md)
- [aircrack-ng](Cybervault/Areas/Ressources/Tools/aircrack-ng.md)
- [shodan](Cybervault/Areas/Ressources/Tools/shodan.md)

### Cheatsheets
- [Recon](Cybervault/Areas/Ressources/Commands/Pentest%20Recon%20Cheatsheet.md)
- [Web / Foothold](Cybervault/Areas/Ressources/Commands/Pentest%20Web%20Cheatsheet.md)
- [Privilege Escalation](Cybervault/Areas/Ressources/Commands/Pentest%20Privesc%20Cheatsheet.md)
- [Pivoting](Cybervault/Areas/Ressources/Commands/Pivoting%20Cheatsheet.md)
- [Reverse Shells](Cybervault/Areas/Ressources/Commands/Reverse%20Shell%20Cheatsheet.md)
- [Linux Permissions](Cybervault/Areas/Ressources/Commands/Linux%20Permissions%20Cheatsheet.md)
- [HTTP Reference](Cybervault/Areas/Ressources/Commands/HTTP%20Reference.md)

---

## Structure

```
Cybervault/
├── Projects/
│   └── labs and machines/
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
