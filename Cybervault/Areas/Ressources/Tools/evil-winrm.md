# evil-winrm

## What it is

`evil-winrm` is a Ruby-based offensive WinRM (Windows Remote Management) client. It provides an interactive PowerShell shell on a remote Windows machine over port 5985 (HTTP) or 5986 (HTTPS), equivalent to SSH for Linux.

Any account that is a member of the **`Remote Management Users`** built-in AD group (or local Administrators) can connect.

## Installation

```bash
gem install evil-winrm
# or on Kali:
sudo apt install evil-winrm
```

## Basic Usage

```bash
# Username + password
evil-winrm -i <target-ip> -u <username> -p '<password>'

# With a domain
evil-winrm -i <target-ip> -u <domain>\\<username> -p '<password>'

# With NTLM hash (pass-the-hash)
evil-winrm -i <target-ip> -u <username> -H <ntlm-hash>

# With Kerberos ticket (pass-the-ticket)
evil-winrm -i <target-ip> -u <username> -r <realm> --kerberos
# set KRB5CCNAME before running
```

## Useful In-Session Commands

| Command | Action |
|---------|--------|
| `upload <local> <remote>` | Upload file to target |
| `download <remote> <local>` | Download file from target |
| `menu` | List available built-in features (PowerShell scripts, bypass modes) |
| `Bypass-4MSI` | Attempt AMSI bypass before loading scripts |
| `Invoke-Binary <path>` | Execute a local binary in memory on the target |

## Common Use Cases

```powershell
# Get user flag
type C:\Users\<user>\Desktop\user.txt

# Enumerate current user
whoami /all

# List local admins
net localgroup administrators

# Check AD group membership
whoami /groups
```

## Related

- [[LDAP Enumeration]]
- [[smb,smbmap, smbcclient]]
