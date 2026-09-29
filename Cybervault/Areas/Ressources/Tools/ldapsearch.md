# ldapsearch

## What it is

`ldapsearch` is the standard CLI LDAP query tool (part of the OpenLDAP client package). In a pentest context it is used to dump and query Active Directory over ports 389 (LDAP) or 636 (LDAPS).

## Installation

```bash
sudo apt install ldap-utils
```

## Basic Syntax

```bash
ldapsearch \
  -x              # simple authentication (not SASL)
  -H ldap://<ip>  # server URI
  -D "<bind_dn>"  # bind account (e.g. "domain\user" or "user@domain.tld")
  -w '<password>' # bind password
  -b "<base_dn>"  # search base (e.g. "DC=domain,DC=tld")
  [filter]        # LDAP filter (optional, default: (objectClass=*))
  [attributes]    # specific attributes to return (optional)
```

## Common Commands

```bash
# Full unauthenticated dump (anonymous bind)
ldapsearch -x -H ldap://<dc-ip> -b "DC=domain,DC=tld"

# Authenticated full dump
ldapsearch -x -H ldap://<dc-ip> \
  -D "domain\ldap" -w 'password' \
  -b "DC=domain,DC=tld" > dump.txt

# Find all user accounts
ldapsearch -x -H ldap://<dc-ip> \
  -D "user@domain.tld" -w 'pass' \
  -b "DC=domain,DC=tld" \
  "(objectClass=user)" sAMAccountName

# Find Kerberoastable accounts (have SPNs)
ldapsearch -x -H ldap://<dc-ip> \
  -D "user@domain.tld" -w 'pass' \
  -b "DC=domain,DC=tld" \
  "(&(objectClass=user)(servicePrincipalName=*))" sAMAccountName servicePrincipalName

# Check a specific user's attributes
ldapsearch -x -H ldap://<dc-ip> \
  -D "user@domain.tld" -w 'pass' \
  -b "DC=domain,DC=tld" \
  "(sAMAccountName=support)"
```

## Post-Dump Grepping

```bash
# Credentials hidden in info/description fields
grep -i "^info:"        dump.txt
grep -i "^description:" dump.txt

# All usernames
grep "sAMAccountName:" dump.txt

# Group memberships for a user
grep -A5 "cn: support" dump.txt | grep "memberOf"

# Password policy
grep -i "minPwdLength\|lockoutThreshold" dump.txt
```

## LDAPS (Encrypted)

```bash
ldapsearch -x -H ldaps://<dc-ip>:636 \
  -D "user@domain.tld" -w 'pass' \
  -b "DC=domain,DC=tld"
```

Add `-o tls_reqcert=never` to skip certificate validation on self-signed certs.

## Related

- [[LDAP Enumeration]]
- [[BloodHound — Active Directory Enumeration]]
- [[impacket]]
