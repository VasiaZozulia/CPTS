# Windows PrivEsc

## Автоматика
```powershell
.\winPEASany.exe
```
## Ручні вектори
```cmd
whoami /priv                  # SeImpersonate -> Potato
systeminfo                    # missing patches
```
- **SeImpersonate** -> PrintSpoofer / GodPotato
- Unquoted service paths, weak service perms (`accesschk`)
- AlwaysInstallElevated, збережені креди (`cmdkey /list`)
**Зв'язки:** [[Credentials]]
