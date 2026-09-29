# LDAP Enumeration

## What it is

LDAP (Lightweight Directory Access Protocol) is the database protocol underlying Active Directory. On a Windows DC, querying LDAP returns users, groups, computers, OUs, GPOs, ACLs, password policies, and any custom attributes — essentially the entire domain structure.

## Ports

| Port | Protocol |
|------|---------|
| 389 | LDAP (plaintext) |
| 636 | LDAPS (TLS encrypted) |
| 3268 | Global Catalog LDAP (forest-wide) |
| 3269 | Global Catalog LDAPS |

## Anonymous / Null Bind

Some DCs allow unauthenticated LDAP queries:

```bash
ldapsearch -x -H ldap://<dc-ip> -b "DC=<domain>,DC=<tld>"
```

If it returns data without `-D` and `-w`, anonymous bind is enabled.

## Authenticated Dump

```bash
ldapsearch -x \
  -H ldap://<dc-ip> \
  -D "<domain>\<username>" \
  -w '<password>' \
  -b "DC=<domain>,DC=<tld>" > ldap_dump.txt
```

Or with a UPN:

```bash
ldapsearch -x \
  -H ldap://<dc-ip> \
  -D "<username>@<domain.tld>" \
  -w '<password>' \
  -b "DC=<domain>,DC=<tld>" > ldap_dump.txt
```

## Key Attributes to Search After Dumping

```bash
# Passwords stashed in free-text fields (very common on HTB and in real environments)
grep -i "^info:"        ldap_dump.txt
grep -i "^description:" ldap_dump.txt
grep -i "^comment:"     ldap_dump.txt

# All user accounts
grep -i "^sAMAccountName:" ldap_dump.txt

# Admin accounts
grep -i "adminCount: 1" ldap_dump.txt

# Service accounts with SPNs (Kerberoasting targets)
grep -i "servicePrincipalName" ldap_dump.txt

# Accounts that don't require Kerberos pre-auth (AS-REP roasting targets)
grep -i "userAccountControl" ldap_dump.txt
# DONT_REQ_PREAUTH = userAccountControl flag 0x400000

# Password policy
grep -i "minPwdLength\|lockoutThreshold\|pwdHistoryLength" ldap_dump.txt
```

## Finding Credentials in the `info` Field

The `info` attribute on a user object is a free-text field that admins often use to store notes — including passwords. It is not shown in the default Active Directory Users and Computers view, so it is frequently overlooked during cleanup.

```bash
grep -i "^info" ldap_dump.txt
```

## Useful One-Liners with ldapsearch

```bash
# Enumerate all users
ldapsearch -x -H ldap://<dc> -D '<user>@<domain>' -w '<pass>' \
  -b "DC=<d>,DC=<tld>" "(objectClass=user)" sAMAccountName

# Find all admin accounts
ldapsearch -x -H ldap://<dc> -D '<user>@<domain>' -w '<pass>' \
  -b "DC=<d>,DC=<tld>" "(&(objectClass=user)(adminCount=1))" sAMAccountName

# Find Kerberoastable accounts
ldapsearch -x -H ldap://<dc> -D '<user>@<domain>' -w '<pass>' \
  -b "DC=<d>,DC=<tld>" "(&(objectClass=user)(servicePrincipalName=*))" sAMAccountName servicePrincipalName
```

## Related

- [[BloodHound — Active Directory Enumeration]]
- [[smb,smbmap, smbcclient]]
- [[RBCD (Resource-Based Constrained Delegation)]]
