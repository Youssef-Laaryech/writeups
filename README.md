# Writeups & Pentest Notes

Personal offensive security knowledge base: HackTheBox machine writeups, CVE research, tool references, and technique notes.

---

## HackTheBox Writeups

| # | Machine | OS | Difficulty | Status | Key Techniques |
|---|---------|-----|------------|--------|----------------|
| 1 | [Nexus](Cybervault/Projects/labs%20and%20machines/writeaps/nexus%20htb%20machine/Nexus%20htb%20machine%20(github).md) | Linux | Easy | Complete | Vhost fuzzing · Git commit history leak · CVE-2026-38526 (unrestricted file upload → RCE) · `os.path.join()` path traversal · Raw Git object crafting |
| 2 | [Silentium](Cybervault/Projects/labs%20and%20machines/writeaps/silentium%20htb%20machine/silentium%20writeup%20(github).md) | Linux | Medium | Complete | Subdomain fuzzing · CVE-2025-58434 (Flowise ATO via token leak) · CVE-2025-59528 (Flowise CustomMCP RCE) · Container env credential leak · CVE-2025-8110 (Gogs symlink traversal → sshCommand → root) |
| 3 | [Cohort](Cybervault/Projects/labs%20and%20machines/writeaps/Cohort%20Htb%20machine/Cohort%20htb%20machine%20(github).md) | Linux | Medium | Complete | SSRF + loopback filter bypass (`127.1`) · Internal port enumeration · Nginx `/status` vhost leak · CVE-2026-39987 (Marimo pre-auth WebSocket RCE) · CVE-2026-41651 (PackageKit TOCTOU → root) |
| 4 | [Abducted](Cybervault/Projects/labs%20and%20machines/writeaps/Abducted%20htb%20machine/Abducted%20writeup%20(github).md) | Linux (Samba) | Medium | In Progress | SMB null session · RPC anonymous enum · User/share/policy extraction via `enum4linux-ng` |

---

## Knowledge Base

### Techniques

| Concept | Description |
|---------|-------------|
| [Path Traversal](Cybervault/Knowledge/Concept/Path%20Traversal.md) | Directory traversal via unsanitized path handling — `../` sequences, `os.path.join()` abuse |
| [Unrestricted File Upload → RCE](Cybervault/Knowledge/Concept/Unrestricted%20File%20Upload%20RCE.md) | Bypassing file type restrictions to upload and execute server-side code |
| [Credential Reuse](Cybervault/Knowledge/Concept/Credential%20Reuse.md) | Reusing credentials leaked from one service to authenticate to another |
| [Vhost Fuzzing](Cybervault/Knowledge/Concept/Vhost%20Fuzzing.md) | Discovering hidden virtual hosts by fuzzing the HTTP `Host` header |
| [Git Commit History Leakage](Cybervault/Knowledge/Concept/Git%20Commit%20History%20Leakage.md) | Recovering secrets that were committed then removed — still visible in git history |
| [Raw Git Object Crafting](Cybervault/Knowledge/Concept/Raw%20Git%20Object%20Crafting.md) | Writing blob/tree/commit objects directly to `.git/objects/` to bypass path validation |
| [SMB / smbmap / smbclient](Cybervault/Knowledge/Concept/smb%2Csmbmap%2C%20smbcclient.md) | SMB enumeration — null sessions, share listing, RPC user/policy extraction |

---

### Tools

| Tool | Category | Notes |
|------|----------|-------|
| [nmap](Cybervault/Areas/Ressources/Tools/nmap.md) | Recon | Port scanning, service/version fingerprinting, NSE scripts |
| [ffuf](Cybervault/Areas/Ressources/Tools/ffuf.md) | Recon | Web fuzzer — directories, vhosts, parameters, headers |
| [Burp Suite](Cybervault/Areas/Ressources/Tools/Burp%20Suite.md) | Web | HTTP proxy, Repeater, Intruder — manual web testing |
| [curl](Cybervault/Areas/Ressources/Tools/curl.md) | Web | CLI HTTP client — crafting raw requests, API testing |
| [nikto](Cybervault/Areas/Ressources/Tools/nikto.md) | Web | Web server scanner — misconfigurations, outdated software |
| [sqlmap](Cybervault/Areas/Ressources/Tools/sqlmap.md) | Web | Automated SQL injection detection and exploitation |
| [hydra](Cybervault/Areas/Ressources/Tools/hydra.md) | Brute Force | Network login brute-forcer — SSH, SMB, HTTP, FTP, RDP |
| [john](Cybervault/Areas/Ressources/Tools/john.md) | Cracking | Password hash cracking — dictionary and rule-based attacks |
| [netcat](Cybervault/Areas/Ressources/Tools/netcat.md) | Post-Exploit | TCP/UDP tool — reverse shells, file transfer, port scanning |
| [metasploit](Cybervault/Areas/Ressources/Tools/metasploit.md) | Exploitation | Full exploitation framework — modules, payloads, post |
| [meterpreter](Cybervault/Areas/Ressources/Tools/meterpreter.md) | Post-Exploit | Advanced Metasploit payload — in-memory, pivoting, migration |
| [msfvenom](Cybervault/Areas/Ressources/Tools/msfvenom.md) | Exploitation | Payload generator — shellcode, executables, web shells |
| [git](Cybervault/Areas/Ressources/Tools/git.md) | Recon / Exploit | History inspection, raw object crafting, credential leaks |
| [tcpdump](Cybervault/Areas/Ressources/Tools/tcpdump.md) | Traffic Analysis | CLI packet capture — filter by host, port, protocol |
| [wireshark](Cybervault/Areas/Ressources/Tools/wireshark.md) | Traffic Analysis | GUI packet analysis — protocol dissection, stream follow |
| [cdk](Cybervault/Areas/Ressources/Tools/cdk.md) | Container | Container escape toolkit — enumerate, exploit, pivot |
| [aircrack-ng](Cybervault/Areas/Ressources/Tools/aircrack-ng.md) | Wireless | Wi-Fi auditing — capture, crack WEP/WPA handshakes |
| [shodan](Cybervault/Areas/Ressources/Tools/shodan.md) | OSINT | Internet-wide search — open ports, banners, CVEs |

---

### Cheatsheets

| Cheatsheet | Phase | Contents |
|------------|-------|----------|
| [Recon](Cybervault/Areas/Ressources/Commands/Pentest%20Recon%20Cheatsheet.md) | Enumeration | nmap, ffuf, gobuster, whatweb, DNS recon |
| [Web / Foothold](Cybervault/Areas/Ressources/Commands/Pentest%20Web%20Cheatsheet.md) | Foothold | SQLi, LFI, file upload, SSRF, SSTI, XXE |
| [Privilege Escalation](Cybervault/Areas/Ressources/Commands/Pentest%20Privesc%20Cheatsheet.md) | Privesc | SUID, sudo abuse, cron jobs, capabilities, services |
| [Pivoting](Cybervault/Areas/Ressources/Commands/Pivoting%20Cheatsheet.md) | Pivoting | SSH tunnels, chisel, ligolo-ng, proxychains |
| [Reverse Shells](Cybervault/Areas/Ressources/Commands/Reverse%20Shell%20Cheatsheet.md) | Shells | Bash, Python, PHP, nc, Perl, PowerShell, Ruby |
| [Linux Permissions](Cybervault/Areas/Ressources/Commands/Linux%20Permissions%20Cheatsheet.md) | Hardening | chmod, chown, SUID/SGID/sticky bit, ACLs, umask |
| [HTTP Reference](Cybervault/Areas/Ressources/Commands/HTTP%20Reference.md) | Web | Methods, status codes, headers, authentication schemes |

---

### Study Notes

| Topic | Notes |
|-------|-------|
| [HTTP](Cybervault/Areas/Daily/HTTP/HTTP.md) | HTTP/1.1 protocol — connections, keep-alive, pipelining |
| [HTTP Requests & Responses](Cybervault/Areas/Daily/HTTP/HTTP%20Requests%20and%20Responses.md) | Request/response anatomy — methods, headers, body, status codes |
| [HTTPS](Cybervault/Areas/Daily/HTTPS.md) | TLS handshake, certificate chain, HSTS, mixed content |

---

## Structure

```
.
└── Cybervault/
    ├── Projects/
    │   └── labs and machines/
    │       └── writeaps/
    │           ├── nexus htb machine/
    │           │   ├── Nexus htb machine (github).md   <- GitHub version (images render)
    │           │   ├── Nexus htb machine.md             <- Obsidian version (wikilinks)
    │           │   └── images/                          <- screenshots for GitHub version
    │           ├── silentium htb machine/
    │           ├── Cohort Htb machine/
    │           └── Abducted htb machine/
    ├── Knowledge/
    │   └── Concept/               <- atomic technique notes
    ├── Areas/
    │   ├── Daily/                 <- protocol and web study notes
    │   └── Ressources/
    │       ├── Tools/             <- one reference note per tool
    │       └── Commands/          <- cheatsheets by attack phase
    └── Areas/Templates/           <- Obsidian templates (writeup, tool, concept)
```

Each machine writeup ships in two versions:
- `(github).md` — uses `![](images/filename.png)` relative links, screenshots render on GitHub
- `.md` — Obsidian version, uses `![[wikilinks]]` pointing to the shared Attachments folder

Readable in any Markdown viewer or [Obsidian](https://obsidian.md/).
