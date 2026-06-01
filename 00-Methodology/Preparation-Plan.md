# CPTS Preparation Plan

План підготовки має відповідати іспиту: не просто пройти модулі, а навчитися стабільно будувати attack chain, вести докази й писати звіт без паніки.

## Поточна оцінка вольту
- **Сильні сторони:** структура йде фазами атаки; є окремі нотатки для recon, web, foothold, privesc, AD, pivoting і reporting; є централізовані файли для кредів та scope.
- **Ризики:** немає вимірюваного графіка підготовки; не зафіксовані критерії готовності; мало місця для аналізу помилок; reporting тренується лише наприкінці, хоча на CPTS це частина бойового процесу.
- **Фокус покращення:** кожен тиждень має давати артефакт: заповнені нотатки, відтворений ланцюг, оновлений checklist, міні-звіт або список помилок.

## Принцип підготовки
1. **Learn:** пройти тему в Academy і записати її своїми словами.
2. **Drill:** повторити команди без підказок у lab/box.
3. **Chain:** з'єднати тему з попередніми фазами атаки.
4. **Document:** одразу оформити команди, скріншоти, findings і remediation.
5. **Review:** виписати, де застряг і який сигнал пропустив.

## 8-тижневий план

| Тиждень | Фокус | Практичний результат |
|---------|-------|----------------------|
| 1 | Методологія, Nmap, service enumeration, ведення нотаток | Заповнені [[Nmap]], [[Footprinting-SMB]], [[Footprinting-NFS-DNS-etc]], [[Footprinting-SNMP]]; 3 хости пройдені тільки через checklist |
| 2 | Web enumeration і базові web-вектори | Заповнені [[Web-Recon]], [[Ffuf-Fuzzing]], [[SQLi-SQLMap]], [[File-Inclusion-LFI-RFI]], [[Command-Injection]]; 2 web foothold з повними evidence |
| 3 | SSRF/SSTI/XSS/common apps, file transfer, shells | Заповнені [[SSRF-SSTI]], [[XSS]], [[Common-Apps]], [[Shells-Payloads]], [[File-Transfers]]; власний payload checklist |
| 4 | Password attacks, credential reuse, attacking services, NTLM relay | Заповнені [[Password-Attacks]], [[Attacking-Services]], [[NTLM-Relay]]; таблиця creds оновлюється під час практики |
| 5 | Linux/Windows PrivEsc і credential dumping | Заповнені [[Linux-PrivEsc]], [[Windows-PrivEsc]], [[Credential-Dumping]]; 4 privesc сценарії з командами й скрінами |
| 6 | Active Directory enumeration, roasting, ACL abuse, lateral movement | Заповнені [[AD-Enumeration]], [[Kerberoast-ASREP]], [[ACL-Abuse]], [[Lateral-Movement]]; один AD chain описаний як narrative |
| 7 | Pivoting, tunneling, trusts, DCSync/tickets | Заповнені [[Ligolo-ng]], [[Chisel-SSH-tunnels]], [[Double-Tunneling-DNS]], [[Trusts]], [[DCSync-Tickets]]; внутрішня підмережа пройдена через re-enum |
| 8 | Mock exam + reporting | 2-3 дні practice chain, 1-2 дні report; готовий приклад звіту за [[Report-Template]] |

Якщо часу менше, не стискай reporting. Краще зменш кількість box/lab-повторів, але залиш щотижневу практику доказів і міні-звітів.

## Щотижневий ритм
- **День 1-2:** нова тема, конспект своїми словами, 5-10 ключових команд у шаблоні [[_Command-Note-Template]].
- **День 3-4:** практика без підглядання; усе знайдене одразу в [[Hosts-Scope]] і [[Credentials]].
- **День 5:** повторити тему на іншому хості або іншому сервісі.
- **День 6:** написати короткий attack narrative: entry point -> privesc -> loot -> next step.
- **День 7:** review помилок, оновлення чеклистів, відпочинок.

## Критерії готовності до іспиту
- [ ] Можеш за 30 хвилин оформити scope, hosts table і структуру evidence.
- [ ] Для кожного відкритого сервісу маєш мінімальний enum checklist.
- [ ] Після foothold автоматично перевіряєш NIC, routing, creds, local privesc і pivoting.
- [ ] Умієш пояснити attack chain без команд, людською мовою.
- [ ] Маєш принаймні один повний report practice з executive summary, findings і attack chain narrative.
- [ ] Всі критичні команди знаходяться у вольті за 10-20 секунд.
- [ ] Після 60-90 хвилин без прогресу вмієш перемкнутися на інший вектор без втрати контексту.

## Журнал слабких місць

| Дата | Тема | Де застряг | Що був за сигнал | Що додати у вольт | Повторено |
|------|------|------------|------------------|-------------------|-----------|
|      |      |            |                  |                   |           |

## Мінімальний набір тренувань перед бронюванням
- [ ] 3 повні recon-сесії з нуля.
- [ ] 3 web foothold з різними класами вразливостей.
- [ ] 2 Linux privesc і 2 Windows privesc без готового walkthrough.
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

Зв'язки: [[Attack-Lifecycle]] · [[Engagement-Checklist]] · [[Time-Management]] · [[Report-Template]]
