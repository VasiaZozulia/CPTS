# CPTS Preparation Plan

План підготовки має відповідати іспиту: не просто пройти модулі, а навчитися стабільно будувати attack chain, вести докази й писати звіт без паніки.

## Поточна оцінка vault
- **Сильні сторони:** структура йде фазами атаки; є окремі нотатки для recon, web, foothold, privesc, AD, pivoting і reporting; є централізовані файли для кредів та scope.
- **Ризики:** є багато покриття тем, але мало вимірюваних gates; reporting не можна відкладати на кінець, бо CPTS має 10 днів на lab + report submission від моменту старту; практику треба рахувати не лише по хостах, а по відтворюваності attack chain.
- **Фокус покращення:** кожен тиждень має давати артефакт: заповнені нотатки, відтворений ланцюг, оновлений checklist, міні-звіт або список помилок.

## Gap-аналіз (станом на 2026-06-02)

| Модуль CPTS | Файл у vault | Пріоритет |
|------------|--------------|-----------|
| File Upload Attacks | [[File-Upload-Attacks]] ✅ додано | Tier 1 |
| Web Attacks (IDOR/XXE/Verb) | [[Web-Attacks-IDOR-XXE-Verb]] ✅ додано | Tier 1 |
| Attacking Enterprise Networks | [[Attacking-Enterprise-Networks]] ✅ додано | Tier 1 — capstone |
| Using Web Proxies (Burp) | [[Burp-WebProxy]] ✅ додано | Tier 3 — enabler |
| Metasploit (post-exploitation) | [[Metasploit]] ✅ додано | Tier 3 — enabler |
| Login Brute Forcing | частково в [[Password-Attacks]] | Tier 2 |
| Vulnerability Assessment | [[Vulnerability-Assessment]] ✅ додано | Tier 3 — triage |

**Всі критичні прогалини в структурі закрито. Наступний ризик — не "немає нотатки", а "не відтворюю техніку під час chain без walkthrough".**

## Readiness gates

Перед бронюванням іспиту має бути не просто `Academy done`, а чотири рівні готовності:

| Gate | Що означає | Мінімум |
|------|------------|---------|
| **G1 — Coverage** | Модуль прочитаний, vault-нотатка існує, ключові команди знайдені за 10-20 секунд | 30/30 модулів |
| **G2 — Reproduction** | Техніка виконана в lab/box без walkthrough і записана в [[Practice-Log]] | 24/30 тем практично |
| **G3 — Chaining** | Техніка використана як частина attack chain: recon -> foothold -> privesc/lateral -> evidence | 5 повних chains |
| **G4 — Reporting** | Є повний звіт за власними доказами, не лише нотатки | 1 повний report + 2 mini reports |

**Правило бронювання:** не бронюй CPTS, поки G2 нижче 80%, немає хоча б одного AD chain, одного pivoting chain і одного повного report practice.

### Додаткові покращення (2026-06-01)
- [[Findings-Library]] — 15 pre-written findings з описом, impact і remediation
- [[Practice-Log]] — трекер відпрацьованих хостів і тем
- [[XSS]] — розширено: DOM XSS, filter bypass, CSP, blind XSS, CSRF chain
- [[File-Inclusion-LFI-RFI]] — розширено: log poisoning, PHP wrappers, /proc, LFI→RCE методологія
- [[Command-Injection]] — розширено: blind detection, OOB exfil, filter bypass таблиця
- [[Ffuf-Fuzzing]] — розширено: vhost fuzzing, login brute, API fuzzing, rate limit
- [[Common-Apps]] — розширено: повні exploit chains для Tomcat, Jenkins, WP, GitLab, Splunk, IIS, JBoss, Drupal
- [[Time-Management]] — додано вечірній sync template + Stuck Playbook

## Принцип підготовки
1. **Learn:** пройти тему в Academy і записати її своїми словами.
2. **Drill:** повторити команди без підказок у lab/box.
3. **Chain:** з'єднати тему з попередніми фазами атаки.
4. **Document:** одразу оформити команди, скріншоти, findings і remediation.
5. **Review:** виписати, де застряг і який сигнал пропустив.

## 8-тижневий план

| Тиждень | Фокус | Практичний результат |
|---------|-------|----------------------|
| 1 | Методологія, Nmap, service enumeration, ведення нотаток | 3 recon-сесії тільки через checklist; hosts table + screenshots; оновлені [[Nmap]], [[Footprinting-SMB]], [[Footprinting-NFS-DNS-etc]], [[Footprinting-SNMP]] |
| 2 | Web enumeration і базові web-вектори | 2 web footholds з повними evidence; mini report на 1 finding; оновлені [[Web-Recon]], [[Ffuf-Fuzzing]], [[SQLi-SQLMap]], [[File-Inclusion-LFI-RFI]], [[Command-Injection]] |
| 3 | SSRF/SSTI/XSS/File Upload/IDOR/XXE/Verb, common apps, file transfer, shells | 2 різні web chains; власний payload checklist; mini report на 2 findings; оновлені [[SSRF-SSTI]], [[XSS]], [[File-Upload-Attacks]], [[Web-Attacks-IDOR-XXE-Verb]], [[Common-Apps]], [[Shells-Payloads]], [[File-Transfers]] |
| 4 | Password attacks, credential reuse, attacking services, NTLM relay | 1 credential reuse або NTLM relay chain; [[Credentials]] оновлюється під час практики; оновлені [[Password-Attacks]], [[Attacking-Services]], [[NTLM-Relay]] |
| 5 | Linux/Windows PrivEsc і credential dumping | 2 Linux + 2 Windows privesc без walkthrough; кожен має команду, proof і remediation; оновлені [[Linux-PrivEsc]], [[Windows-PrivEsc]], [[Credential-Dumping]] |
| 6 | Active Directory enumeration, roasting, ACL abuse, lateral movement | 1 AD chain до high-value target, описаний як narrative; BloodHound notes + screenshots; оновлені [[AD-Enumeration]], [[Kerberoast-ASREP]], [[ACL-Abuse]], [[Lateral-Movement]] |
| 7 | Pivoting, tunneling, trusts, DCSync/tickets | 1 pivoting-сценарій з re-enum внутрішньої підмережі; 1 multi-host chain; оновлені [[Ligolo-ng]], [[Chisel-SSH-tunnels]], [[Double-Tunneling-DNS]], [[Trusts]], [[DCSync-Tickets]] |
| 8 | Mock exam + reporting rehearsal | 3 дні mock engagement, report пишеться щодня; 1 повний звіт за [[Report-Template]]; фінальний список слабких місць і booking decision |

Якщо часу менше, не стискай reporting. Краще зменш кількість box/lab-повторів, але залиш щотижневу практику доказів і міні-звітів.

## Щотижневий ритм
- **День 1-2:** нова тема, конспект своїми словами, 5-10 ключових команд у шаблоні [[_Command-Note-Template]].
- **День 3-4:** практика без підглядання; усе знайдене одразу в [[Hosts-Scope]] і [[Credentials]].
- **День 5:** повторити тему на іншому хості або іншому сервісі.
- **День 6:** написати короткий attack narrative: entry point -> privesc -> loot -> next step; оформити 1 mini finding.
- **День 7:** review помилок, оновлення чеклистів, відпочинок.

## Щоденний мінімум під час практики
- [ ] 1 новий запис у [[Practice-Log]] або оновлення покриття тем.
- [ ] Всі нові креди одразу в [[Credentials]].
- [ ] Всі IP, hostname, порти, ролі й доступи одразу в [[Hosts-Scope]].
- [ ] 1 короткий висновок: "який сигнал я побачив і чому він вів до наступного кроку".
- [ ] Якщо була експлуатація: команда + proof + screenshot + draft finding.

## Критерії готовності до іспиту
- [ ] Можеш за 30 хвилин оформити scope, hosts table і структуру evidence.
- [ ] Для кожного відкритого сервісу маєш мінімальний enum checklist.
- [ ] Після foothold автоматично перевіряєш NIC, routing, creds, local privesc і pivoting.
- [ ] Умієш пояснити attack chain без команд, людською мовою.
- [ ] Маєш принаймні один повний report practice з executive summary, findings і attack chain narrative.
- [ ] Всі критичні команди знаходяться у вольті за 10-20 секунд.
- [ ] Після 60-90 хвилин без прогресу вмієш перемкнутися на інший вектор без втрати контексту.
- [ ] Можеш пояснити, які докази потрібні reviewer'у для кожного finding, ще до написання звіту.

## Журнал слабких місць

| Дата | Тема | Де застряг | Що був за сигнал | Що додати у вольт | Повторено |
|------|------|------------|------------------|-------------------|-----------|
|      |      |            |                  |                   |           |

## Мінімальний набір тренувань перед бронюванням
- [ ] 3 повні recon-сесії з нуля.
- [ ] 3 web foothold з різними класами вразливостей (SQLi, File Upload, IDOR/XXE).
- [ ] 2 Linux privesc без готового walkthrough.
- [ ] 2 Windows privesc без готового walkthrough.
- [ ] 1 NTLM relay або credential reuse chain.
- [ ] 1 AD attack chain до high-value target.
- [ ] 1 pivoting-сценарій з re-enumeration внутрішньої підмережі.
- [ ] 1 повний звіт за власними доказами.

## Після кожної практики
- [ ] Чи записані всі IP, порти, ролі й доступи?
- [ ] Чи всі креди внесені в один файл?
- [ ] Чи є команда, вивід і скріншот для кожного важливого кроку?
- [ ] Чи зрозуміло, чому саме цей вектор спрацював?
- [ ] Чи додано новий урок у журнал слабких місць?

Зв'язки: [[Attack-Lifecycle]] · [[Engagement-Checklist]] · [[Time-Management]] · [[Report-Template]] · [[Exam-Strategy]] · [[HTB-Box-Recommendations]]
