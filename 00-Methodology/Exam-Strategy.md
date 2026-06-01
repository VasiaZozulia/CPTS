# CPTS Exam Strategy — Розширені рекомендації

> Це не замінює [[Time-Management]] — це доповнює його тактичними деталями.

---

## Принцип №1 — "Scan Wide, Exploit Deep"

Перші 24 год: **тільки розвідка**, нічого не ламай.
- Повний TCP-скан всіх хостів у scope паралельно
- Запис всіх відкритих портів, версій, hostname
- Скріншот стартової сторінки кожного веб-додатку
- Побудова карти мережі (хто куди ходить)

**Чому:** іспит має кілька підмереж і dependent targets. Exploit першого-ліпшого нічого не дасть якщо не бачиш загальної картини.

---

## Пріоритетна матриця хостів

| Ознака | Пріоритет |
|--------|-----------|
| Web app з формами + DB backend | Дуже високий — часто foothold |
| Windows server з SMB signing OFF | Дуже високий — NTLM relay |
| Linux з nginx/apache 80/443 | Високий — web foothold |
| Windows DC (порти 88, 389, 636) | Середній — ціль пізніше через AD |
| Принтери / IoT / embedded | Низький — дефолтні креди але рідко дають chain |
| Linux без веб, тільки SSH | Низький — звичайно lateral move target |

---

## Документування з першого кроку

**Правило "3 артефакти на дію":** для кожного важливого кроку:
1. Команда (copy-paste ready)
2. Вивід або скріншот
3. Висновок одним реченням

Якщо немає скріншоту → у звіті немає цього кроку. Роби скрін навіть коли "очевидно".

```bash
# Нумерація скрінів:
01_nmap_initial.png
02_web_homepage.png
03_ffuf_directories.png
04_sqli_error.png
05_sqli_data_extracted.png
06_shell_obtained.png
```

---

## Reporting — починай з Дня 1

Не чекай кінця практичної частини. Кожен день:

```
[ ] Написати 1 finding для кожної підтвердженої вразливості
[ ] Перевірити: чи є скріншот Steps to Reproduce?
[ ] Заповнити поле "Affected Asset" у [[Findings-Library]]
```

**Чому:** під кінець (день 9-10) ти будеш втомлений і забудеш деталі. Finding написаний по свіжій пам'яті значно якісніший.

---

## Стратегія по днях (детально)

### День 1 — Map the Battlefield
```
09:00  Читаєш scope → scope.md
09:30  Паралельний Nmap всіх хостів (--min-rate 1000 -p-)
11:00  Аналіз результатів, пріоритетна матриця
12:00  Веб скріншоти (EyeWitness або вручну)
13:00  Обід + пауза (30 хв)
13:30  Глибокий скан відкритих портів (-sCV)
15:00  SMB enum (nxc smb --shares, --users, signing check)
16:00  LDAP enum якщо є AD (ldapsearch, nxc ldap)
17:00  DNS enum (dig axfr, dnsrecon)
18:00  Вечірній sync → заповнюй шаблон
```

### День 2-3 — First Blood (Web Foothold)
- Пріоритет: хости з web applications
- Порядок: ffuf → SQLi → File Upload → LFI → Command Injection
- Якщо застряг > 90 хв → [[Time-Management]] Stuck Playbook
- Ціль: мінімум 1 foothold до кінця Дня 3

### День 4-5 — Expand (PrivEsc + Lateral)
- На кожному отриманому хості: linpeas/winpeas → privesc
- Credential dump кожного compromised хоста
- Credential reuse на всіх хостах (nxc smb --pass-pol, --sam, --lsa)
- Шукай нові підмережі (ip a, arp-scan, /etc/hosts)

### День 6-7 — Active Directory
- BloodHound збір від першого AD-foothold
- Аналіз: Shortest Path to DA, Kerberoastable Users, AS-REP, ACL edges
- Kerberoast → hashcat → lateral movement → DCSync
- Не забудь перевірити trusts між доменами

### День 8-9 — Cleanup
- Всі флаги зібрані та записані з точним шляхом
- Кожне finding має повний ланцюг Steps to Reproduce
- Перевір: чи є gap у доказах (missing screenshots)?
- Перечитай кожне finding з позиції reviewera

### День 10 — Buffer
- Не плануй нічого критичного
- Фінальна перевірка звіту
- Відпочинок перед reporting window

---

## Reporting Window (10 днів) — стратегія

### Структура часу
| Дні | Задача |
|-----|--------|
| 1-2 | Executive Summary + всі Findings (чернетка) |
| 3-4 | Attack Narrative (повний ланцюг від recon до DA) |
| 5   | Пауза. Прочитай весь звіт холодним поглядом |
| 6-7 | Правки: технічна точність, мова, форматування |
| 8   | Фінальна перевірка, export PDF |
| 9   | Буфер |
| 10  | Здача |

### Executive Summary — шаблон думки
Відповідай на 4 питання:
1. Що тестувалось і яким методом?
2. Яке загальне risk level і чому?
3. Який найкритичніший шлях зловмисника?
4. Головна рекомендація для виправлення?

### Findings — правило "so what"
Після кожного опису Impact запитай себе: "so what?" — якщо можна поставити питання і відповідь ще не очевидна, додай ще речення.

Погано: "Attacker can read files from the server."
Добре: "Attacker can read /etc/shadow, extract password hashes, and use them to pivot to other systems in the network, potentially achieving full domain compromise."

---

## Найчастіші помилки на іспиті

### Технічні
| Помилка | Наслідок | Запобігання |
|---------|----------|-------------|
| Не скановані всі порти (default Nmap) | Пропущений сервіс = missed foothold | Завжди `-p-` спочатку |
| Забув перевірити IPv6 | Обхід фаєрвола | `nmap -6` або `ping6` |
| Не перевірив UDP | SNMP, TFTP, DNS = інформація | `nmap -sU --top-ports 100` |
| Credential reuse не систематичний | Пропущений lateral move | nxc по всім хостам після кожного нового пароля |
| BloodHound без аналізу ALL edges | Пропущений ACL шлях | Перевір GenericAll, WriteDACL, WriteOwner |

### Репортинг
| Помилка | Наслідок |
|---------|----------|
| Steps to Reproduce з хоста зловмисника без пояснення | Reviewer не може відтворити |
| CVSS без обґрунтування | Виглядає як copy-paste |
| Remediation занадто загальна ("patch your system") | Не actionable |
| Немає скріншоту initial access | Критична відсутність доказу |
| Attack narrative з gap (пропущений крок між доступами) | Логіка атаки не зрозуміла |

---

## Налаштування середовища перед іспитом

```bash
# Структура директорій:
mkdir -p ~/exam/{nmap,web,screenshots,loot,exploits,report}

# Tmux session з вікнами:
tmux new -s exam
# Ctrl+B c → нове вікно
# Вікно 1: nmap/enum
# Вікно 2: shell/access
# Вікно 3: notes/tail

# Автоматичний лог терміналу (все що вводиш):
script -a ~/exam/terminal_log_$(date +%Y%m%d).txt

# Flameshot для скріншотів з анотаціями:
apt install flameshot -y
```

---

## Mental model: "Завжди питай 3 питання"

Після кожної дії:
1. **Що я тільки що отримав?** (доступ, інформація, credential, підмережа)
2. **Куди це веде далі?** (ще один хост? ескалація? lateral move?)
3. **Це задокументовано?** (скрін є? команда записана?)

Якщо на питання 3 відповідь "ні" → зупинись і задокументуй ЗАРАЗ.

---

## Психологічна стійкість

- **Застряг > 90 хв** → обов'язково переключись, не сиди через силу
- **Кінець Дня 3 без foothold** → нормально для складних сценаріїв; повертайся до [[Time-Management]] Stuck Playbook з свіжим поглядом
- **Паніка через reporting** → починай з одного finding, потім іще один. Не пиши весь звіт одним блоком
- **Відчуття "все пропало"** → переглянь Credentials.md і Hosts-Scope: ти зібрав більше ніж думаєш

**Зв'язки:** [[Time-Management]] · [[Preparation-Plan]] · [[Engagement-Checklist]] · [[Report-Template]] · [[Findings-Library]]
