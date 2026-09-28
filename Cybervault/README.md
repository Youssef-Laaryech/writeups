# Writeups & Pentest Notes

Personal knowledge base documenting HackTheBox machine writeups, vulnerability research, and offensive security techniques.

---

## HackTheBox Writeups

| Machine | OS | Difficulty | Status | Techniques |
|---------|-----|------------|--------|------------|
| [Nexus](Cybervault/Projects/labs%20and%20machines/writeaps/nexus%20htb%20machine/Nexus%20htb%20machine%20(github).md) | Linux | Easy | ✅ Complete | Vhost fuzzing · Git secret leak · CVE-2026-38526 (PHP file upload RCE) · Path traversal via `os.path.join()` · Raw Git object crafting |
| [Silentium](Cybervault/Projects/labs%20and%20machines/writeaps/silentium%20htb%20machine/silentium%20writeup%20(github).md) | Linux | Medium | ✅ Complete | Subdomain fuzzing · CVE-2025-58434 (Flowise ATO) · CVE-2025-59528 (Flowise CustomMCP RCE) · Container env credential leak · CVE-2025-8110 (Gogs symlink traversal → root) |
| [Cohort](Cybervault/Projects/labs%20and%20machines/writeaps/Cohort%20Htb%20machine/Cohort%20htb%20machine%20(github).md) | Linux | Medium | ✅ Complete | SSRF + loopback filter bypass (`127.1`) · Internal port enum · Nginx `/status` vhost leak · CVE-2026-39987 (Marimo pre-auth WebSocket RCE) · CVE-2026-41651 (PackageKit TOCTOU → root) |
| [Abducted](Cybervault/Projects/labs%20and%20machines/writeaps/Abducted%20htb%20machine/Abducted%20writeup%20(github).md) | Linux (Samba) | Medium | 🔄 In Progress | SMB null session · RPC anonymous enum · User/share/policy leak via `enum4linux-ng` |

---

## Knowledge Base

### Techniques

| Concept | Description |
|---------|-------------|
| [Path Traversal](Cybervault/Knowledge/Concept/Path%20Traversal.md) | Directory traversal via unsanitized path handling |
| [Unrestricted File Upload → RCE](Cybervault/Knowledge/Concept/Unrestricted%20File%20Upload%20RCE.md) | Bypassing upload filters to achieve code execution |
| [Credential Reuse](Cybervault/Knowledge/Concept/Credential%20Reuse.md) | Reusing leaked credentials across services |
| [Vhost Fuzzing](Cybervault/Knowledge/Concept/Vhost%20Fuzzing.md) | Discovering hidden virtual hosts via Host header fuzzing |
| [Git Commit History Leakage](Cybervault/Knowledge/Concept/Git%20Commit%20History%20Leakage.md) | Recovering secrets removed from git history |
| [Raw Git Object Crafting](Cybervault/Knowledge/Concept/Raw%20Git%20Object%20Crafting.md) | Writing git objects directly to bypass path validation |
| [SMB / smbmap / smbclient](Cybervault/Knowledge/Concept/smb%2Csmbmap%2C%20smbcclient.md) | SMB enumeration techniques and null session abuse |

### Tools

| Tool | Notes |
|------|-------|
| [nmap](Cybervault/Areas/Ressources/Tools/nmap.md) | Port scanning & service fingerprinting |
| [ffuf](Cybervault/Areas/Ressources/Tools/ffuf.md) | Web fuzzer — directories, vhosts, parameters |
| [Burp Suite](Cybervault/Areas/Ressources/Tools/Burp%20Suite.md) | Web proxy & manual testing |
| [curl](Cybervault/Areas/Ressources/Tools/curl.md) | HTTP requests from the command line |
| [git](Cybervault/Areas/Ressources/Tools/git.md) | Git for pentest — history inspection, object crafting |
| [netcat](Cybervault/Areas/Ressources/Tools/netcat.md) | TCP/UDP Swiss army knife — shells, file transfer |
| [tcpdump](Cybervault/Areas/Ressources/Tools/tcpdump.md) | Packet capture |
| [wireshark](Cybervault/Areas/Ressources/Tools/wireshark.md) | Packet analysis GUI |
| [hydra](Cybervault/Areas/Ressources/Tools/hydra.md) | Network brute-force (SSH, SMB, HTTP, FTP…) |
| [john](Cybervault/Areas/Ressources/Tools/john.md) | Password hash cracking |
| [metasploit](Cybervault/Areas/Ressources/Tools/metasploit.md) | Exploitation framework |
| [meterpreter](Cybervault/Areas/Ressources/Tools/meterpreter.md) | Post-exploitation shell (Metasploit) |
| [msfvenom](Cybervault/Areas/Ressources/Tools/msfvenom.md) | Payload generator |
| [sqlmap](Cybervault/Areas/Ressources/Tools/sqlmap.md) | Automated SQL injection |
| [nikto](Cybervault/Areas/Ressources/Tools/nikto.md) | Web server scanner |
| [cdk](Cybervault/Areas/Ressources/Tools/cdk.md) | Container escape toolkit |
| [aircrack-ng](Cybervault/Areas/Ressources/Tools/aircrack-ng.md) | Wi-Fi security auditing |
| [shodan](Cybervault/Areas/Ressources/Tools/shodan.md) | Internet-wide port/service search engine |

### Cheatsheets

| Cheatsheet | Phase |
|------------|-------|
| [Recon](Cybervault/Areas/Ressources/Commands/Pentest%20Recon%20Cheatsheet.md) | Initial enumeration — nmap, ffuf, whatweb |
| [Web / Foothold](Cybervault/Areas/Ressources/Commands/Pentest%20Web%20Cheatsheet.md) | Web attack techniques — SQLi, LFI, upload, SSRF |
| [Privilege Escalation](Cybervault/Areas/Ressources/Commands/Pentest%20Privesc%20Cheatsheet.md) | Linux & Windows privesc — SUID, sudo, services |
| [Pivoting](Cybervault/Areas/Ressources/Commands/Pivoting%20Cheatsheet.md) | Tunnelling, port forwarding, chisel, ligolo |
| [Reverse Shells](Cybervault/Areas/Ressources/Commands/Reverse%20Shell%20Cheatsheet.md) | Bash, Python, PHP, nc, Perl, PowerShell, Ruby |
| [Linux Permissions](Cybervault/Areas/Ressources/Commands/Linux%20Permissions%20Cheatsheet.md) | chmod, chown, SUID/SGID, ACLs |
| [HTTP Reference](Cybervault/Areas/Ressources/Commands/HTTP%20Reference.md) | HTTP methods, headers, status codes |

---

## Study Notes

| Topic | Notes |
|-------|-------|
| [HTTP](Cybervault/Areas/Daily/HTTP/HTTP.md) | HTTP protocol fundamentals |
| [HTTP Requests & Responses](Cybervault/Areas/Daily/HTTP/HTTP%20Requests%20and%20Responses.md) | Request/response structure, headers, methods |
| [HTTPS](Cybervault/Areas/Daily/HTTPS.md) | TLS handshake, certificates, HTTPS internals |

---

## Structure

```
Cybervault/
├── Projects/
│   └── labs and machines/
│       └── writeaps/          ← machine writeups (Obsidian + GitHub versions)
├── Knowledge/
│   └── Concept/               ← atomic technique notes
├── Areas/
│   ├── Daily/                 ← web/protocol study notes
│   └── Ressources/
│       ├── Tools/             ← one note per tool
│       └── Commands/          ← cheatsheets by phase
└── Attachments/               ← all pasted screenshots
```

Notes are written in Markdown and interlink via Obsidian wikilinks. Each machine writeup has both an Obsidian version (`.md` with `![[wikilinks]]`) and a GitHub-compatible version (`(github).md` with relative `![](images/)` links and a local `images/` folder).

Readable in any Markdown viewer or [Obsidian](https://obsidian.md/).
