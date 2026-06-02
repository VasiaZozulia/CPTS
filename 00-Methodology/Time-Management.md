# Тайм-менеджмент іспиту

CPTS має 10 днів від старту іспиту до завантаження звіту. Це означає, що exploitation і reporting йдуть паралельно: кожен день завершується evidence cleanup і чернеткою findings.

Підготовка до іспиту живе в [[Preparation-Plan]]. Цей файл — тільки про розподіл часу під час самого CPTS.

## Орієнтовний ритм
- **День 1:** scope, структура evidence, full TCP scan, web screenshots, hosts table
- **День 2:** service enumeration, перші foothold гіпотези, draft findings для підтверджених вразливостей
- **День 3–4:** exploitation, first foothold, local enum, credential harvesting, щоденний report sync
- **День 5–6:** privesc, lateral movement, pivoting у внутрішні сегменти, attack narrative оновлюється щовечора
- **День 7–8:** AD-частина, шлях до DA/high-value target, добивання флагів, evidence gap check
- **День 9:** технічна дочистка, missing screenshots, повна чернетка звіту
- **День 10:** фінальна перевірка звіту, export/upload, буфер

## Правила
- Застряг > 1–2 год на одному векторі → перемкнись на інший хост, повернись свіжим
- Кожен крок одразу документуй (команда + вивід + скрін) — інакше забудеш для звіту
- Кожна підтверджена вразливість у той самий день отримує draft finding: affected asset, impact, proof, remediation
- Не лізь у позапрограмний матеріал: шлях та іспит вирівняні між собою
- Щовечора роби 20-хвилинний sync: флаги, креди, нові підмережі, відкриті гіпотези, missing screenshots

---

## Щовечірній Sync (шаблон — 20 хв)

Заповнюй в кінці кожного дня іспиту:

```
=== DAY [N] SYNC ===
Дата/час:

СТАТУС ХОСТІВ:
| IP | Hostname | Доступ | Флаг | Наступний крок |
|----|----------|--------|------|----------------|
|    |          |        |      |                |

НОВІ CREDENTIALS (додані в Credentials.md):
-

НОВІ ПІДМЕРЕЖІ/ХОСТИ (додані в Hosts-Scope.md):
-

ВІДКРИТІ ГІПОТЕЗИ (вектори не перевірені):
1.
2.

MISSING SCREENSHOTS (зробити до здачі):
-

DRAFT FINDINGS / REPORT TODO:
- Finding:
- Evidence missing:
- Remediation note:

ПЛАН НА ЗАВТРА (3 конкретні цілі):
1.
2.
3.
```

---

## "Stuck Playbook" — що робити коли застряг

Перед тим як перемкнутись на інший хост, пройди по цьому чеклисту (10-15 хв):

```
ENUM REFRESH
[ ] Чи всі порти скановані? (full TCP + UDP top 100)
[ ] Чи перевірені всі версії сервісів на відомі CVE? (searchsploit / exploit-db)
[ ] Чи всі вебсторінки і директорії фаззовані? (ffuf -e .php,.bak,.txt)
[ ] Чи перевірені vhosts / subdomains?
[ ] Чи є ще не перевірені endpoints в Burp Site Map?

CREDENTIALS
[ ] Чи спробував всі знайдені паролі на поточному хості?
[ ] Чи спробував credential reuse з інших хостів?
[ ] Чи є дефолтні credentials для знайденого сервісу?

PRIVILEGE / CONTEXT
[ ] Чи запустив linpeas / winpeas і прочитав весь вивід?
[ ] Чи перевірив sudo -l / SePrivileges?
[ ] Чи подивився на NIC і routing? (нова підмережа?)
[ ] Чи є інший користувач куди можна переключитись?

WEB-SPECIFIC
[ ] Чи перевірив source code на захардкоджені credenitals?
[ ] Чи переглянув JavaScript файли? (burp → Target → JS files)
[ ] Чи спробував різні HTTP методи (OPTIONS, PUT, DELETE)?
[ ] Чи перевірив robots.txt, sitemap.xml, .git/, .svn/?

ЯКЩО ВСЕ ВИЩЕ DONE → перемикайся, але залиш нотатку в Hosts-Scope що хост "parked"
```
