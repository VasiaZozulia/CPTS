# Linux PrivEsc

## Автоматика
```bash
./linpeas.sh | tee linpeas.txt
```
## Ручні вектори
```bash
sudo -l                       # NOPASSWD, GTFOBins
find / -perm -4000 2>/dev/null  # SUID
cat /etc/crontab               # cron + writable scripts
getcap -r / 2>/dev/null        # capabilities
```
**Зв'язки:** GTFOBins, [[Credentials]]
