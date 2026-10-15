# OpenVAS

## What it is

OpenVAS (Open Vulnerability Assessment Scanner) is a **free, open-source vulnerability scanner** maintained by Greenbone Networks as part of their **Greenbone Community Edition (GCE)**. It is the open-source equivalent of Nessus — functionally similar, fully free, self-hosted, and the standard choice for lab environments, home labs, and budget-constrained engagements.

On Kali Linux, OpenVAS is available out of the box as `gvm` (Greenbone Vulnerability Manager).

## Architecture

```
Greenbone Security Assistant (GSA)  ← web UI at https://localhost:9392
        │
        └── GVM (Greenbone Vulnerability Manager)  ← orchestrates everything
                │
                ├── OpenVAS Scanner  ← runs the actual NVT plugins against targets
                │
                ├── PostgreSQL  ← stores scan results, configs, assets
                │
                └── NVT Feed  ← Network Vulnerability Tests (100,000+ plugins)
                        updated daily from Greenbone's feed server
```

## Installation on Kali

Kali ships with GVM (the modern name for OpenVAS):

```bash
sudo apt install gvm
sudo gvm-setup          # downloads NVT feed, sets up PostgreSQL, creates admin user
                        # takes 10-30 min on first run
sudo gvm-start          # starts all GVM services
sudo gvm-check-setup    # verifies everything is running correctly
```

The setup script prints the **auto-generated admin password** — save it. Browse to `https://127.0.0.1:9392`.

## Common Setup Issues

| Problem | Fix |
|---------|-----|
| Feed sync fails | `sudo runuser -u _gvm -- greenbone-nvt-sync` |
| PostgreSQL not starting | `sudo systemctl start postgresql` then `sudo gvm-start` |
| Services not ready after setup | Wait 5–10 min after `gvm-setup` before `gvm-start` — feed import takes time |
| Forgot admin password | `sudo runuser -u _gvm -- gvmd --user=admin --new-password=<newpass>` |

## Basic Workflow (Web UI)

1. **Scans → Tasks → New Task**
2. Set name, choose **Scan Config** (see below), set target
3. Create **Target**: IP or range, optional SSH/SMB credentials
4. Launch scan → monitor progress under Scans → Tasks
5. View results under **Reports** — filter by severity

## Scan Configs (Equivalent to Nessus Policies)

| Config | Description |
|--------|-------------|
| **Full and Fast** | Recommended default — all plugins, optimized for speed |
| **Full and Fast Ultimate** | Adds destructive tests (can crash services) — use carefully |
| **Full and Very Deep** | Thorough but slow — runs every applicable plugin |
| **Empty** | No plugins — use as a base to build custom configs |
| **Host Discovery** | Ping sweep only — no vulnerability checks |
| **System Discovery** | OS fingerprinting + open ports — no vuln checks |

## Credentialed Scans

More accurate — OpenVAS logs in and checks patch levels, installed packages, config files:

**Configuration → Credentials → New Credential:**
- Type: `Username + Password` or `SSH Key`
- Apply to target under **New Target → SSH Credentials**

With credentials, OpenVAS can check:
- Installed package versions vs CVE database
- Local configuration files
- World-writable directories, SUID binaries
- Weak file permissions

## Severity (CVSS v3 based)

| Level | CVSS v3 Score |
|-------|--------------|
| Critical | 9.0 – 10.0 |
| High | 7.0 – 8.9 |
| Medium | 4.0 – 6.9 |
| Low | 0.1 – 3.9 |
| Log | N/A — informational only |

## CLI Usage (gvm-cli)

```bash
# Install the CLI client
pip3 install gvm-tools

# Connect and run an XML command
gvm-cli --gmp-username admin --gmp-password <pass> \
  socket --socketpath /var/run/gvmd/gvmd.sock \
  --xml "<get_tasks/>"

# Start a scan programmatically (Python API)
from gvm.connections import UnixSocketConnection
from gvm.protocols.gmp import Gmp

with Gmp(connection=UnixSocketConnection()) as gmp:
    gmp.authenticate('admin', 'password')
    tasks = gmp.get_tasks()
```

## Pentest Use Cases

- **Lab scanning** — scan your own HTB / VulnHub VMs before manually exploiting to understand the full attack surface
- **Internal network assessment** — deploy on a pivot host, scan internal segments
- **Pre-exploitation triage** — quickly identify which hosts have critical CVEs worth targeting manually
- **Report generation** — export findings as PDF/XML for documentation

## OpenVAS vs Nessus

| | OpenVAS | Nessus Essentials |
|---|---|---|
| Cost | Free | Free (16 IP limit) |
| Plugin count | ~100,000 NVTs | ~100,000+ |
| Update frequency | Daily (community feed) | Daily |
| UI quality | Functional but dated | Polished |
| Kali integration | Built-in (`gvm`) | Manual install |
| Enterprise features | None (pay for Greenbone Enterprise) | Nessus Pro |
| Best for | Labs, CTFs, internal assessments | Professional pentests |

## Related

- [[Nessus]]
- [[Qualys]]
- [[nmap]]
