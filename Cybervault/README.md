# Writeups & Pentest Notes

Personal knowledge base documenting HackTheBox machine writeups, vulnerability research, and offensive security techniques.

---

## HackTheBox Writeups

| Machine | OS | Difficulty | Status | Techniques |
|---------|-----|------------|--------|------------|
| [Nexus](Projects/labs%20and%20machines/writeaps/nexus%20htb%20machine/Nexus%20htb%20machine%20(github).md) | Linux | Easy | ✅ Complete | Vhost fuzzing · Git secret leak · CVE-2026-38526 (PHP file upload RCE) · Path traversal via `os.path.join()` · Raw Git object crafting |
| [Silentium](Projects/labs%20and%20machines/writeaps/silentium%20htb%20machine/silentium%20writeup%20(github).md) | Linux | Medium | ✅ Complete | Subdomain fuzzing · CVE-2025-58434 (Flowise ATO via token leak) · CVE-2025-59528 (Flowise CustomMCP RCE) · Container env credential leak · CVE-2025-8110 (Gogs symlink traversal → root) |
| [Cohort](Projects/labs%20and%20machines/writeaps/Cohort%20Htb%20machine/Cohort%20htb%20machine%20(github).md) | Linux | Medium | ✅ Complete | SSRF + loopback filter bypass (`127.1`) · Internal port enum · Nginx `/status` vhost leak · CVE-2026-39987 (Marimo pre-auth WebSocket RCE) · CVE-2026-41651 (PackageKit TOCTOU → root) |
| [Abducted](Projects/labs%20and%20machines/writeaps/Abducted%20htb%20machine/Abducted%20writeup%20(github).md) | Linux (Samba) | Medium | 🔄 In Progress | SMB null session · RPC anonymous enum · User/share/policy leak via `enum4linux-ng` |

---

## Knowledge Base

### Techniques

| Concept | Description |
|---------|-------------|
| [Path Traversal](Knowledge/Concept/Path%20Traversal.md) | Directory traversal via unsanitized path handling |
| [Unrestricted File Upload → RCE](Knowledge/Concept/Unrestricted%20File%20Upload%20RCE.md) | Bypassing upload filters to achieve code execution |
| [Credential Reuse](Knowledge/Concept/Credential%20Reuse.md) | Reusing leaked credentials across services |
| [Vhost Fuzzing](Knowledge/Concept/Vhost%20Fuzzing.md) | Discovering hidden virtual hosts via Host header fuzzing |
| [Git Commit History Leakage](Knowledge/Concept/Git%20Commit%20History%20Leakage.md) | Recovering secrets removed from git history |
| [Raw Git Object Crafting](Knowledge/Concept/Raw%20Git%20Object%20Crafting.md) | Writing git objects directly to bypass path validation |
| [SMB / smbmap / smbclient](Knowledge/Concept/smb%2Csmbmap%2C%20smbcclient.md) | SMB enumeration techniques and null session abuse |

---

### Tools

| Tool | Notes |
|------|-------|
| [nmap](Areas/Ressources/Tools/nmap.md) | Port scanning & service fingerprinting |
| [ffuf](Areas/Ressources/Tools/ffuf.md) | Web fuzzer — directories, vhosts, parameters |
| [Burp Suite](Areas/Ressources/Tools/Burp%20Suite.md) | Web proxy & manual testing |
| [curl](Areas/Ressources/Tools/curl.md) | HTTP requests from the command line |
| [git](Areas/Ressources/Tools/git.md) | Git for pentest — history inspection, object crafting |
| [netcat](Areas/Ressources/Tools/netcat.md) | TCP/UDP Swiss army knife — shells, file transfer |
| [tcpdump](Areas/Ressources/Tools/tcpdump.md) | Packet capture |
| [wireshark](Areas/Ressources/Tools/wireshark.md) | Packet analysis GUI |
| [hydra](Areas/Ressources/Tools/hydra.md) | Network brute-force (SSH, SMB, HTTP, FTP…) |
| [john](Areas/Ressources/Tools/john.md) | Password hash cracking |
| [metasploit](Areas/Ressources/Tools/metasploit.md) | Exploitation framework |
| [meterpreter](Areas/Ressources/Tools/meterpreter.md) | Post-exploitation shell (Metasploit) |
| [msfvenom](Areas/Ressources/Tools/msfvenom.md) | Payload generator |
| [sqlmap](Areas/Ressources/Tools/sqlmap.md) | Automated SQL injection |
| [nikto](Areas/Ressources/Tools/nikto.md) | Web server scanner |
| [cdk](Areas/Ressources/Tools/cdk.md) | Container escape toolkit |
| [aircrack-ng](Areas/Ressources/Tools/aircrack-ng.md) | Wi-Fi security auditing |
| [shodan](Areas/Ressources/Tools/shodan.md) | Internet-wide port/service search engine |

---

### Cheatsheets

| Cheatsheet | Phase |
|------------|-------|
| [Recon](Areas/Ressources/Commands/Pentest%20Recon%20Cheatsheet.md) | Initial enumeration — nmap, ffuf, whatweb |
| [Web / Foothold](Areas/Ressources/Commands/Pentest%20Web%20Cheatsheet.md) | Web attack techniques — SQLi, LFI, upload, SSRF |
| [Privilege Escalation](Areas/Ressources/Commands/Pentest%20Privesc%20Cheatsheet.md) | Linux & Windows privesc — SUID, sudo, services |
| [Pivoting](Areas/Ressources/Commands/Pivoting%20Cheatsheet.md) | Tunnelling, port forwarding, chisel, ligolo |
| [Reverse Shells](Areas/Ressources/Commands/Reverse%20Shell%20Cheatsheet.md) | Bash, Python, PHP, nc, Perl, PowerShell, Ruby |
| [Linux Permissions](Areas/Ressources/Commands/Linux%20Permissions%20Cheatsheet.md) | chmod, chown, SUID/SGID, ACLs |
| [HTTP Reference](Areas/Ressources/Commands/HTTP%20Reference.md) | HTTP methods, headers, status codes |

---

### Study Notes

| Topic | Notes |
|-------|-------|
| [HTTP](Areas/Daily/HTTP/HTTP.md) | HTTP protocol fundamentals |
| [HTTP Requests & Responses](Areas/Daily/HTTP/HTTP%20Requests%20and%20Responses.md) | Request/response structure, headers, methods |
| [HTTPS](Areas/Daily/HTTPS.md) | TLS handshake, certificates, HTTPS internals |

---

## Structure

```
.
├── Projects/
│   └── labs and machines/
│       └── writeaps/               ← machine writeups
│           ├── nexus htb machine/
│           │   ├── Nexus htb machine (github).md   ← GitHub version (images render)
│           │   ├── Nexus htb machine.md             ← Obsidian version
│           │   └── images/
│           ├── silentium htb machine/
│           ├── Cohort Htb machine/
│           └── Abducted htb machine/
├── Knowledge/
│   └── Concept/                    ← atomic technique notes
├── Areas/
│   ├── Daily/                      ← web/protocol study notes
│   └── Ressources/
│       ├── Tools/                  ← one note per tool
│       └── Commands/               ← cheatsheets by phase
└── Attachments/                    ← all pasted screenshots (Obsidian)
```

Each machine writeup exists in two versions:
- **`(github).md`** — GitHub-compatible, uses `![](images/filename.png)` with a local `images/` folder so screenshots render on GitHub.
- **`.md`** — Obsidian version, uses `![[wikilinks]]` and links to the shared `Attachments/` folder.

Readable in any Markdown viewer or [Obsidian](https://obsidian.md/).
