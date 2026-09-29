# BloodHound — Active Directory Enumeration

## What it is

BloodHound is a graph-based Active Directory attack path analysis tool. It collects relationship data from a domain (users, groups, computers, ACLs, GPOs, trusts) and represents them as nodes and edges in a Neo4j graph database, then lets you query for privilege-escalation paths, shortest paths to Domain Admin, and dangerous permission configurations.

## Architecture (Community Edition — Kali)

| Component | Role |
|-----------|------|
| `bloodhound-python` | LDAP-based data collector — runs from attacker machine with valid domain credentials |
| Neo4j | Graph database that stores all nodes and relationships |
| PostgreSQL | Backend storage for the BloodHound API |
| BloodHound CE web UI | Browser-based interface at `http://localhost:8080` |

## Data Collection

```bash
bloodhound-python \
  -u <username> \
  -p '<password>' \
  -d <domain.tld> \
  -ns <dc-ip> \
  -c all
```

`-c all` collects: users, groups, computers, sessions, ACLs, GPOs, containers, OUs, trusts.

Outputs JSON files (`*_users.json`, `*_groups.json`, `*_computers.json`, etc.).

## Starting BloodHound CE (Kali)

```bash
sudo neo4j start          # start graph database
bloodhound-setup          # first-run only: initialize Postgres + Neo4j link
bloodhound-start          # start BloodHound API + web UI
# Browse to http://localhost:8080
```

**Common first-run issues:**

| Problem | Fix |
|---------|-----|
| PostgreSQL collation version mismatch | `psql -U postgres -c "ALTER DATABASE bloodhound REFRESH COLLATION VERSION;"` |
| Neo4j auth fails in API | Edit `/etc/bhapi/bhapi.json` — update `neo4j_password` to match the current Neo4j password |

Default UI credentials: `admin` / `admin` (forced change on first login).

## Workflow

1. **Upload data** — Quick Upload → select all JSON files
2. **Mark owned nodes** — search for your controlled user/computer → right-click → "Add to Owned"
3. **Set starting node** — right-click owned node → "Set as Starting Node"
4. **Pathfinding** — Pathfinding tab → destination: `DOMAIN ADMINS@<DOMAIN>`
5. **Analyze edges** — click any edge to see the abuse info panel (explains how to exploit each relationship)

## Key Edges to Look For

| Edge | Meaning |
|------|---------|
| `MemberOf` | Group membership — check what rights the group has |
| `GenericAll` | Full control over target object |
| `GenericWrite` | Write any attribute — use for RBCD, shadow credentials, SPN add |
| `WriteDACL` | Modify the object's ACL — grant yourself GenericAll |
| `WriteOwner` | Change object owner — then modify DACL |
| `ForceChangePassword` | Reset target user's password |
| `DCSync` | Allowed to replicate domain secrets — dump all hashes |
| `AllowedToDelegate` | Constrained delegation configured — ticket impersonation possible |
| `AllowedToAct` | RBCD configured — ticket impersonation possible |
| `HasSession` | Admin user has an active session on a computer — target for credential dumping |

## Related

- [[RBCD (Resource-Based Constrained Delegation)]]
- [[GenericAll ACL Abuse]]
- [[LDAP Enumeration]]
- [[impacket]]
