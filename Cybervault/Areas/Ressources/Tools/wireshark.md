---
type: tool
category: network-analysis
tags: [wireshark, pcap, packet-capture, network-analysis]
source: teamghsoftware/security-cheatsheets
---

## What it is
Wireshark is a GUI packet analyzer. In pentest, useful for analyzing captured traffic, debugging protocol interactions, decrypting TLS (with keys), and inspecting PCAP files from CTF challenges.

## Capture

```bash
# Start capturing on an interface:
# Capture > Interfaces > select interface > Start

# Stop capture:
# Capture > Stop
```

## Essential display filters

```bash
# Filter by protocol
http
dns
tcp
ftp
smtp

# Filter by IP
ip.addr == 192.168.1.1
ip.src == 10.10.10.10
ip.dst == 10.10.10.10

# Filter by port
tcp.port == 80
tcp.port == 443

# Filter by HTTP method
http.request.method == "POST"

# Find packets with a specific string in payload
frame contains "password"

# Combine filters
ip.addr == 10.10.10.10 && tcp.port == 80

# Show only failed TCP connections
tcp.flags.reset == 1
```

## Useful actions

```bash
# Follow a full TCP/UDP/SSL stream (reassemble the conversation):
# Right-click packet > Follow > TCP Stream / UDP Stream / TLS Stream

# Apply a filter from a selected packet:
# Right-click packet > Apply as Filter > Selected / Not Selected

# Capture only your own traffic (disable promiscuous mode):
# Capture > Options > uncheck "Use promiscuous mode on all interfaces" > Start

# Manage TLS decryption keys:
# View > Wireless Toolbar > Decryption Keys
# Or: Edit > Preferences > Protocols > TLS > pre-master secret log file
```

## Command-line alternative: tshark

```bash
# Capture on interface
tshark -i eth0

# Read a pcap file
tshark -r capture.pcap

# Filter
tshark -r capture.pcap -Y "http.request.method == POST"

# Extract specific field
tshark -r capture.pcap -T fields -e http.request.uri
```

## Related
- [[tcpdump]]
- [[Pentest Recon Cheatsheet]]
