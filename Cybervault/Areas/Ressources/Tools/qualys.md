# Qualys

## What it is

Qualys is an **enterprise-grade cloud-based vulnerability management platform**. Unlike Nessus (which runs as a local scanner) or OpenVAS (self-hosted), Qualys runs entirely in the cloud — scans are launched from Qualys-managed scanners or from a lightweight virtual scanner appliance you deploy inside your network. It is the platform of choice in large enterprise environments, managed security service providers (MSSPs), and compliance-heavy industries (finance, healthcare, government).

## How it Differs from Nessus and OpenVAS

| | Qualys | Nessus | OpenVAS |
|---|---|---|---|
| Deployment | Cloud SaaS | Local install | Self-hosted |
| Scale | Millions of assets, global | Single scanner | Single scanner |
| Pricing | Enterprise subscription (expensive) | Essentials free / Pro paid | Free (open source) |
| Compliance | Built-in PCI-DSS, HIPAA, ISO 27001, CIS | Plugin-based | Community plugins |
| API | Full REST API, dashboards, ticketing integration | Basic | Basic |
| Agents | Yes — lightweight agents on each host | No | No |
| Primary use | Enterprise vuln management programs | Pentest / assessment | Lab / small org |

## Architecture

```
Qualys Cloud Platform (SaaS)
    │
    ├── External Scanner (Qualys-hosted) — scans internet-facing assets
    │
    ├── Virtual Scanner Appliance (deployed in your network) — scans internal assets
    │       runs as a VM on VMware / VirtualBox / Hyper-V
    │
    └── Qualys Cloud Agents (installed on each host) — continuous monitoring
            lightweight, no credentials needed, reports directly to cloud
```

## Key Modules

| Module | What it does |
|--------|-------------|
| **VMDR** (Vulnerability Management, Detection & Response) | Core module — discover, assess, prioritize, remediate |
| **PC** (Policy Compliance) | Audit configs against CIS, PCI, HIPAA, NIST |
| **WAS** (Web Application Scanning) | Automated DAST — SQLi, XSS, CSRF on web apps |
| **CS** (Container Security) | Scan Docker images and running containers |
| **EDR** | Endpoint detection and response |
| **CertView** | SSL/TLS certificate inventory and expiry tracking |

## Qualys vs Nessus in a Pentest Context

In a real pentest engagement, you are unlikely to use Qualys directly — it requires a subscription and is aimed at continuous enterprise monitoring, not point-in-time penetration testing. However:

- You will **encounter Qualys data** when a client shares their existing vulnerability scan reports
- Understanding Qualys severity ratings and report format is important for **interpreting pre-existing scan data**
- Qualys findings are often used as the **starting point** for manual exploitation in a pentest — the scanner found CVE-XXXX, the pentester validates and exploits it manually

## Severity Model (QDS — Qualys Detection Score)

Qualys uses its own **QDS (Qualys Detection Score)** on top of CVSS, factoring in:
- CVSS base score
- Whether a public exploit exists
- Whether active exploitation in the wild has been observed
- Asset criticality

| QDS Range | Label |
|-----------|-------|
| 90–100 | Critical |
| 70–89 | High |
| 40–69 | Medium |
| 1–39 | Low |

## Free Access

Qualys offers a **free Community Edition** at [qualys.com/free-trial](https://www.qualys.com/free-trial/) — limited to a small number of IPs but fully functional for learning.

## API Usage (automation)

Qualys has a full REST API — useful for automated reporting pipelines:

```bash
# Launch a scan via API
curl -u "username:password" \
  "https://qualysapi.qualys.com/api/2.0/fo/scan/?action=launch&scan_title=MyScan&target_from=tags&tag_include_selector=any&tag_set_include=<tag_id>&scanners_in_ag=1&option_id=<option_profile_id>"

# Get scan results
curl -u "username:password" \
  "https://qualysapi.qualys.com/api/2.0/fo/scan/?action=fetch&scan_ref=<scan_ref>&output_format=XML"
```

## Related

- [[Nessus]]
- [[OpenVAS]]
- [[nmap]]
