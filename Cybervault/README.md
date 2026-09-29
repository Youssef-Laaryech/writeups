# Cybervault — Pentest Notes & HTB Writeups

Personal offensive security knowledge base: HackTheBox machine writeups, CVE research, tool references, and technique notes.

---

## HackTheBox Writeups

| # | Machine | OS | Difficulty | Status | Key Techniques |
|---|---------|-----|------------|--------|----------------|
| 1 | [Nexus](Projects/labs%20and%20machines/writeaps/nexus%20htb%20machine/Nexus%20htb%20machine%20(github).md) | Linux | Easy | Complete | Vhost fuzzing · Git commit history leak · CVE-2026-38526 (unrestricted file upload → RCE) · `os.path.join()` path traversal · Raw Git object crafting |
| 2 | [Silentium](Projects/labs%20and%20machines/writeaps/silentium%20htb%20machine/silentium%20writeup%20(github).md) | Linux | Medium | Complete | Subdomain fuzzing · CVE-2025-58434 (Flowise ATO) · CVE-2025-59528 (Flowise CustomMCP RCE) · Container env credential leak · CVE-2025-8110 (Gogs symlink → sshCommand → root) |
| 3 | [Cohort](Projects/labs%20and%20machines/writeaps/Cohort%20Htb%20machine/Cohort%20htb%20machine%20(github).md) | Linux | Medium | Complete | SSRF + loopback filter bypass (`127.1`) · Internal port enum · Nginx `/status` vhost leak · CVE-2026-39987 (Marimo pre-auth WebSocket RCE) · CVE-2026-41651 (PackageKit TOCTOU → root) |
| 4 | [Support](Projects/labs%20and%20machines/writeaps/support%20htb%20machine/support%20writeup%20(github).md) | Windows (DC) | Easy | Complete | SMB null session · .NET binary decompilation (ilspycmd) · XOR obfuscation reversal · LDAP `info` attribute credential · WinRM shell · BloodHound GenericAll → RBCD → S4U2Self/S4U2Proxy → SYSTEM |
| 5 | [TwoMillion](Projects/labs%20and%20machines/writeaps/2million%20htb%20machine/2million%20writeup%20(github).md) | Linux | Easy | Complete | JS obfuscation (ROT13/base64) · API invite code generation · Broken object-level authorization (self-elevate to admin) · OS command injection · `.env` credential reuse · CVE-2023-0386 (OverlayFS SUID → root) |
| 6 | [Orion](Projects/labs%20and%20machines/writeaps/Orion%20htb%20machine/Orion%20writeup%20(github).md) | Linux | Hard | Complete | Second-order SQL injection · TOTP/2FA bypass via SQLi · SSH agent socket hijacking · Sudo misconfiguration |
| 7 | [Abducted](Projects/labs%20and%20machines/writeaps/Abducted%20htb%20machine/Abducted%20writeup%20(github).md) | Linux (Samba) | Medium | In Progress | SMB null session · RPC anonymous enum · User/share/policy extraction via `enum4linux-ng` |
| 8 | [Paperwork](Projects/labs%20and%20machines/writeaps/paperwork%20htb%20machine/paperwork%20writeup.md) | Linux | Medium | In Progress | OS command injection via unsanitized job name in subprocess shell call |

---

## Knowledge Base

### Techniques

| Concept | Description |
|---------|-------------|
| [Path Traversal](Knowledge/Concept/Path%20Traversal.md) | Directory traversal via unsanitized path handling — `../` sequences, `os.path.join()` abuse |
| [Unrestricted File Upload → RCE](Knowledge/Concept/Unrestricted%20File%20Upload%20RCE.md) | Bypassing file type restrictions to upload and execute server-side code |
| [Credential Reuse](Knowledge/Concept/Credential%20Reuse.md) | Reusing credentials leaked from one service to authenticate to another |
| [Vhost Fuzzing](Knowledge/Concept/Vhost%20Fuzzing.md) | Discovering hidden virtual hosts by fuzzing the HTTP `Host` header |
| [Git Commit History Leakage](Knowledge/Concept/Git%20Commit%20History%20Leakage.md) | Recovering secrets that were committed then removed — still visible in git history |
| [Raw Git Object Crafting](Knowledge/Concept/Raw%20Git%20Object%20Crafting.md) | Writing blob/tree/commit objects directly to `.git/objects/` to bypass path validation |
| [SMB / smbmap / smbclient](Knowledge/Concept/smb%2Csmbmap%2C%20smbcclient.md) | SMB enumeration — null sessions, share listing, RPC user/policy extraction |
| [.NET Binary Reverse Engineering](Knowledge/Concept/dotNET%20Binary%20Reverse%20Engineering.md) | Decompiling .NET IL to C# with ilspycmd — finding hardcoded creds and obfuscation routines |
| [LDAP Enumeration](Knowledge/Concept/LDAP%20Enumeration.md) | Authenticated and anonymous AD LDAP queries — dumping users, attributes, password policy |
| [BloodHound — AD Enumeration](Knowledge/Concept/BloodHound%20%E2%80%94%20Active%20Directory%20Enumeration.md) | Graph-based AD attack path analysis — collecting, ingesting, and querying domain relationships |
| [GenericAll ACL Abuse](Knowledge/Concept/GenericAll%20ACL%20Abuse.md) | Full AD object control — password reset, group add, computer attribute write |
| [RBCD](Knowledge/Concept/RBCD%20%28Resource-Based%20Constrained%20Delegation%29.md) | Kerberos delegation abuse via `msDS-AllowedToActOnBehalfOfOtherIdentity` → impersonate any domain user |

---

### Tools

| Tool | Category | Notes |
|------|----------|-------|
| [nmap](Areas/Ressources/Tools/nmap.md) | Recon | Port scanning, service/version fingerprinting, NSE scripts |
| [ffuf](Areas/Ressources/Tools/ffuf.md) | Recon | Web fuzzer — directories, vhosts, parameters, headers |
| [Burp Suite](Areas/Ressources/Tools/Burp%20Suite.md) | Web | HTTP proxy, Repeater, Intruder — manual web testing |
| [curl](Areas/Ressources/Tools/curl.md) | Web | CLI HTTP client — crafting raw requests, API testing |
| [nikto](Areas/Ressources/Tools/nikto.md) | Web | Web server scanner — misconfigurations, outdated software |
| [sqlmap](Areas/Ressources/Tools/sqlmap.md) | Web | Automated SQL injection detection and exploitation |
| [hydra](Areas/Ressources/Tools/hydra.md) | Brute Force | Network login brute-forcer — SSH, SMB, HTTP, FTP, RDP |
| [john](Areas/Ressources/Tools/john.md) | Cracking | Password hash cracking — dictionary and rule-based attacks |
| [netcat](Areas/Ressources/Tools/netcat.md) | Post-Exploit | TCP/UDP tool — reverse shells, file transfer, port scanning |
| [metasploit](Areas/Ressources/Tools/metasploit.md) | Exploitation | Full exploitation framework — modules, payloads, post |
| [meterpreter](Areas/Ressources/Tools/meterpreter.md) | Post-Exploit | Advanced Metasploit payload — in-memory, pivoting, migration |
| [msfvenom](Areas/Ressources/Tools/msfvenom.md) | Exploitation | Payload generator — shellcode, executables, web shells |
| [git](Areas/Ressources/Tools/git.md) | Recon / Exploit | History inspection, raw object crafting, credential leaks |
| [tcpdump](Areas/Ressources/Tools/tcpdump.md) | Traffic Analysis | CLI packet capture — filter by host, port, protocol |
| [wireshark](Areas/Ressources/Tools/wireshark.md) | Traffic Analysis | GUI packet analysis — protocol dissection, stream follow |
| [cdk](Areas/Ressources/Tools/cdk.md) | Container | Container escape toolkit — enumerate, exploit, pivot |
| [aircrack-ng](Areas/Ressources/Tools/aircrack-ng.md) | Wireless | Wi-Fi auditing — capture, crack WEP/WPA handshakes |
| [shodan](Areas/Ressources/Tools/shodan.md) | OSINT | Internet-wide search — open ports, banners, CVEs |
| [ldapsearch](Areas/Ressources/Tools/ldapsearch.md) | Active Directory | CLI LDAP queries — authenticated AD dump, attribute search |
| [evil-winrm](Areas/Ressources/Tools/evil-winrm.md) | Active Directory | Interactive PowerShell shell over WinRM (port 5985/5986) |
| [bloodhound](Areas/Ressources/Tools/bloodhound.md) | Active Directory | Graph-based AD attack path analysis — BloodHound CE + Neo4j |
| [impacket](Areas/Ressources/Tools/impacket.md) | Active Directory | Python AD/Windows attack toolkit — psexec, secretsdump, getST, rbcd, addcomputer |
| [ilspycmd](Areas/Ressources/Tools/ilspycmd.md) | Reverse Engineering | CLI .NET decompiler — reconstruct C# source from IL bytecode |

---

### Cheatsheets

| Cheatsheet | Phase | Contents |
|------------|-------|----------|
| [Recon](Areas/Ressources/Commands/Pentest%20Recon%20Cheatsheet.md) | Enumeration | nmap, ffuf, gobuster, whatweb, DNS recon |
| [Web / Foothold](Areas/Ressources/Commands/Pentest%20Web%20Cheatsheet.md) | Foothold | SQLi, LFI, file upload, SSRF, SSTI, XXE |
| [Privilege Escalation](Areas/Ressources/Commands/Pentest%20Privesc%20Cheatsheet.md) | Privesc | SUID, sudo abuse, cron jobs, capabilities, services |
| [Pivoting](Areas/Ressources/Commands/Pivoting%20Cheatsheet.md) | Pivoting | SSH tunnels, chisel, ligolo-ng, proxychains |
| [Reverse Shells](Areas/Ressources/Commands/Reverse%20Shell%20Cheatsheet.md) | Shells | Bash, Python, PHP, nc, Perl, PowerShell, Ruby |
| [Linux Permissions](Areas/Ressources/Commands/Linux%20Permissions%20Cheatsheet.md) | Hardening | chmod, chown, SUID/SGID/sticky bit, ACLs, umask |
| [HTTP Reference](Areas/Ressources/Commands/HTTP%20Reference.md) | Web | Methods, status codes, headers, authentication schemes |

---

### Study Notes

| Topic | Notes |
|-------|-------|
| [HTTP](Areas/Daily/HTTP/HTTP.md) | HTTP/1.1 protocol — connections, keep-alive, pipelining |
| [HTTP Requests & Responses](Areas/Daily/HTTP/HTTP%20Requests%20and%20Responses.md) | Request/response anatomy — methods, headers, body, status codes |
| [HTTPS](Areas/Daily/HTTPS.md) | TLS handshake, certificate chain, HSTS, mixed content |

---

## Structure

```
.
├── Projects/
│   └── labs and machines/
│       └── writeaps/
│           ├── nexus htb machine/
│           │   ├── Nexus htb machine (github).md   <- GitHub version (images render)
│           │   ├── Nexus htb machine.md             <- Obsidian version (wikilinks)
│           │   └── images/
│           ├── silentium htb machine/
│           ├── Cohort Htb machine/
│           ├── support htb machine/
│           ├── 2million htb machine/
│           ├── Orion htb machine/
│           └── Abducted htb machine/
├── Knowledge/
│   └── Concept/               <- atomic technique notes
├── Areas/
│   ├── Daily/                 <- protocol and web study notes
│   └── Ressources/
│       ├── Tools/             <- one reference note per tool
│       └── Commands/          <- cheatsheets by attack phase
└── Areas/
    └── Templates/             <- Obsidian templates (writeup, tool, concept)
```

Each machine writeup ships in two versions:
- `(github).md` — uses `![](images/filename.png)` relative links, screenshots render on GitHub
- `.md` — Obsidian version, uses `![[wikilinks]]` pointing to the shared Attachments folder

Readable in any Markdown viewer or [Obsidian](https://obsidian.md/).
