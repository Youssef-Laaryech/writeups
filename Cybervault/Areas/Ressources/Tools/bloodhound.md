# BloodHound

## What it is

BloodHound is a graph-based Active Directory attack path analysis tool. It collects domain relationship data via LDAP and visualizes it as a property graph, enabling fast identification of privilege-escalation paths, dangerous ACL configurations, and shortest paths to Domain Admin.

See also: [[BloodHound — Active Directory Enumeration]] for the full workflow.

## Components

| Component | Purpose |
|-----------|---------|
| `bloodhound-python` | Python data collector (runs on Linux as an authenticated domain user) |
| `SharpHound` | .NET data collector (runs on a Windows machine inside the domain) |
| Neo4j | Graph database backend |
| BloodHound CE | Web UI + API (Kali: `bloodhound-start`, browse `localhost:8080`) |

## Data Collection — bloodhound-python

```bash
pip3 install bloodhound
# or: sudo apt install bloodhound-python

bloodhound-python \
  -u <username> \
  -p '<password>' \
  -d <domain.tld> \
  -ns <dc-ip> \
  -c all
```

## Starting BloodHound CE

```bash
sudo neo4j start
bloodhound-start
# http://localhost:8080 — default creds: admin/admin
```

## Key Cypher Queries (Neo4j)

```cypher
-- All paths from owned nodes to Domain Admins
MATCH p=shortestPath((u:User {owned:true})-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAIN.TLD"}))
RETURN p

-- Find all users with GenericAll on computers
MATCH (u)-[:GenericAll]->(c:Computer) RETURN u.name, c.name

-- Find computers where Domain Admins have sessions
MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAIN.TLD"}),(u)-[:HasSession]->(c:Computer)
RETURN c.name
```

## Related

- [[BloodHound — Active Directory Enumeration]]
- [[RBCD (Resource-Based Constrained Delegation)]]
- [[GenericAll ACL Abuse]]
