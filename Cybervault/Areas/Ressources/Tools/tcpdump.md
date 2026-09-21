---
type: tool
category: network-analysis
tags: [tcpdump, pcap, packet-capture, cli, network]
source: teamghsoftware/security-cheatsheets
---

## What it is
tcpdump is the command-line packet sniffer. On remote shells where Wireshark isn't available, this is how you capture traffic. Output can be saved as `.pcap` and opened in Wireshark.

## Basic capture

```bash
# Capture on eth0
tcpdump -i eth0

# Verbose output
tcpdump -i eth0 -nnvvS

# Show packets in HEX + ASCII
tcpdump -XX -i eth0

# Save to pcap file (open in Wireshark)
tcpdump -w capture.pcap -i eth0

# Read from pcap file
tcpdump -tttt -r capture.pcap

# Show IPs instead of hostnames
tcpdump -n -i eth0
```

## Filtering

```bash
# By source/destination IP
tcpdump src 192.168.1.1
tcpdump dst 192.168.1.1

# By port
tcpdump src port 53
tcpdump dst port 21
tcpdump port 3389

# By network (CIDR)
tcpdump net 192.168.1.0/24

# By packet size
tcpdump less 64
tcpdump greater 256
```

## Advanced / combined filters

```bash
# All traffic from 192.168.1.10 to port 80
tcpdump -nnvvS and src 192.168.1.10 and dst port 80

# From 172.16.0.0/16 to 192.168.1.0/24 or 10.0.0.0/8
tcpdump src net 172.16.0.0/16 and dst net 192.168.1.0/24 or 10.0.0.0/8

# From host H1, not going to port 22
tcpdump src H1 and not dst port 22

# Traffic from 192.168.1.1 to port 80 or 21
tcpdump 'src 192.168.1.1 and (dst port 80 or 21)'
```

## Pentest use cases

```bash
# Listen for credentials in clear-text (FTP/HTTP)
tcpdump -i eth0 -A -s 0 port 21 or port 80

# Capture DNS queries (useful for identifying internal hosts)
tcpdump -i eth0 port 53

# Capture ICMP (check if host is pingable)
tcpdump icmp
```

## Related
- [[wireshark]]
- [[Pentest Recon Cheatsheet]]
