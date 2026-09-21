---
type: moc
tags: [moc, index, home]
---

# Cybervault — Home

Welcome to Cybervault, your offensive security knowledge base.

---

## 🗂 Active Projects

- [[Machines Index]] — All HTB / lab machines
  - [[Nexus htb machine]] ✅

---

## 🧠 Knowledge Base

### Techniques / Concepts
- [[Path Traversal]]
- [[Unrestricted File Upload RCE]]
- [[Credential Reuse]]
- [[Vhost Fuzzing]]
- [[Git Commit History Leakage]]
- [[Raw Git Object Crafting]]
- `Knowledge/Concept/` — all atomic concept notes

### Tools

**Recon**
- [[nmap]] — port scanning
- [[ffuf]] — fuzzing (dirs, vhosts, params)
- [[shodan]] — internet-wide search / OSINT
- [[nikto]] — web server vulnerability scanner

**Web & Exploitation**
- [[Burp Suite]] — HTTP intercept & replay
- [[curl]] — command-line HTTP
- [[sqlmap]] — SQL injection automation

**Shells & Payloads**
- [[netcat]] — bind/reverse shells, file transfer
- [[msfvenom]] — payload generation (exe, elf, php, war…)
- [[metasploit]] — exploitation framework
- [[meterpreter]] — post-exploitation shell

**Password Attacks**
- [[hydra]] — online brute-force
- [[john]] — offline hash cracking

**Recon / Version Control**
- [[git]] — git recon + raw object crafting

**Network Analysis**
- [[wireshark]] — GUI packet analysis
- [[tcpdump]] — CLI packet capture

**Wireless**
- [[aircrack-ng]] — WEP/WPA cracking

### Cheatsheets
- [[Pentest Recon Cheatsheet]]
- [[Pentest Web Cheatsheet]]
- [[Pentest Privesc Cheatsheet]]
- [[Reverse Shell Cheatsheet]]
- [[Pivoting Cheatsheet]]
- [[HTTP Reference]]
- [[Linux Permissions Cheatsheet]]

---

## 📚 Study Notes

### Web / HTTP
- [[HTTP]] — protocol fundamentals, URL structure, cURL basics
- [[HTTP Requests and Responses]] — request/response anatomy, status codes, DevTools
- [[HTTPS]] — TLS, encryption, Burp HTTPS interception

---

## 📁 Vault Structure

```
Cybervault/
├── Areas/
│   ├── Daily/         ← Study notes (not daily journals)
│   ├── Ressources/
│   │   ├── Tools/     ← One note per tool
│   │   └── Commands/  ← Cheatsheets by pentest phase
│   ├── Templates/     ← Note templates
│   └── Archive/       ← Old drafts, finished projects
├── Knowledge/
│   └── Concept/       ← Atomic technique/concept notes
├── Projects/
│   └── labs and machines/
│       ├── Machines Index.md
│       └── writeaps/  ← Active writeups
├── Attachments/       ← All screenshots and images
└── Inbox/             ← Quick capture, file and process later
```

---

## 🔖 Templates

- [[concept template]] — for new technique/concept notes
- [[writeup template]] — for new HTB / lab writeups
- [[tool template]] — for new tool notes
