---
type: tool
category: recon, osint
tags: [shodan, osint, recon, internet-scan]
source: teamghsoftware/security-cheatsheets
---

## What it is
Shodan is a search engine that indexes internet-connected devices. In pentest, it's used for OSINT on a target — finding exposed services, versions, default credentials, and misconfigurations before you even touch the target.

## Search filters

```bash
# Filter by IP range
net:10.0.0.0/8
net:192.168.0.0/16

# Filter by port
port:22
port:3389
port:8080

# Filter by location
city:"Paris" country:FR
geo:48.8566,2.3522

# Filter by hostname
hostname:target.com

# Filter by operating system
os:"Windows 10"
os:"Linux"

# Filter by date
before:01/01/2025
after:01/01/2024

# Combine filters
port:22 country:US os:"Ubuntu"

# Find specific services
product:"Apache httpd"
product:"OpenSSH" version:"7.4"

# Find default credentials exposure
http.title:"admin" port:80 country:FR
```

## Useful search queries

```bash
# Exposed webcams
webcam has_screenshot:true

# Industrial control systems
tag:ics

# Vulnerable to specific CVE (Shodan sometimes tags)
vuln:CVE-2021-44228    # Log4Shell

# Open Elasticsearch (no auth)
port:9200 product:Elasticsearch

# Open MongoDB
port:27017 product:MongoDB
```

## CLI usage

```bash
# Install
pip install shodan

# Init with API key
shodan init <API_KEY>

# Search
shodan search port:22 country:US

# Host info
shodan host <IP>

# Count results
shodan count port:80 country:FR
```

## Related
- [[nmap]]
- [[Pentest Recon Cheatsheet]]
