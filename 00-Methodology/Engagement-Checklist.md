# Engagement Checklist (іспит)

## Перед стартом
- [ ] Прочитати scope і правила (in-scope IP/підмережі, що заборонено)
- [ ] Підготувати `99-Loot/Hosts-Scope.md` і `Credentials.md`
- [ ] Запустити запис нотаток + скріншоти з таймстемпами

## Recon (кожен новий хост)
- [ ] Повне сканування портів (TCP all-ports, потім -sCV по відкритих)
- [ ] UDP-скан ключових сервісів
- [ ] Енумерація кожного сервісу окремо (SMB, DNS, HTTP, SNMP...)
- [ ] Записати все знайдене в Hosts-Scope

## Foothold
- [ ] Перевірити дефолтні / слабкі креди
- [ ] Пошук відомих вразливостей знайдених версій
- [ ] Веб: fuzzing директорій/параметрів, форми, авторизація
- [ ] Веб: перевірити SSRF (url=, src=, redirect=) і SSTI (template engine?)
- [ ] Перевірити NTLM relay (SMB Signing off? → Responder + ntlmrelayx)
- [ ] Зафіксувати точку входу і метод

## Post-exploitation (кожен хост)
- [ ] Локальна енумерація (користувачі, мережа, NIC/підмережі!)
- [ ] PrivEsc до root/SYSTEM
- [ ] Зібрати креди / хеші / ключі → у Credentials.md
- [ ] **Перевірити нові підмережі для pivoting**

## Active Directory
- [ ] BloodHound збір + аналіз шляхів (shortest path to DA + ACL edges!)
- [ ] Kerberoast / AS-REP roast
- [ ] Перевірити ACL abuse (GenericAll, WriteDACL, ForceChangePassword, WriteOwner)
- [ ] Lateral movement по знайдених кредах
- [ ] Credential dumping на кожному compromised хості (nxc --sam --lsa)
- [ ] Шлях до DA / DCSync
- [ ] Перевірити trusts між доменами

## Перед здачею
- [ ] Усі флаги зібрані й записані з джерелом
- [ ] Кожен крок має скріншот + команду
- [ ] Звіт покриває всі required-секції
