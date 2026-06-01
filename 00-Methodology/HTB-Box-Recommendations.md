# HTB Box Recommendations — Практика по темах

> Ці машини відповідають темам CPTS модулів. Порядок: від простішого до складнішого в кожній категорії.

---

## Методологія вибору машин

**Правило:** не сортируй по рейтингу "Easy/Medium/Hard" — сортируй по **техніці**, яку треба відпрацювати.
Краще зробити 3 Medium-машини на одну тему, ніж 10 різних Easy.

---

## Web — Foothold

### SQL Injection
| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Cronos | Linux | SQLi login bypass → RCE via cron | Easy |
| Poison | Linux | LFI + log poisoning | Medium |
| Trick | Linux | SQLi + subdomain enum | Easy |
| Schooled | FreeBSD | XSS → Admin hijack → RCE | Medium |

### File Upload
| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Shocker | Linux | ShellShock (CGI) | Easy |
| Falafel | Linux | Type juggling + file upload bypass | Hard |
| Reel | Windows | Phishing → macro + NTLM capture | Hard |

### LFI / File Inclusion
| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Poison | Linux | LFI → log poisoning → RCE | Medium |
| Beep | Linux | LFI в Elastix | Easy |
| Nineveh | Linux | LFI + port knock | Medium |
| Pilgrimage | Linux | CVE in ImageMagick | Easy |

### Command Injection
| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Shocker | Linux | CGI command injection | Easy |
| Horizontall | Linux | Strapi RCE | Easy |
| Time | Linux | Java serialization → RCE | Medium |

### XSS / SSRF / SSTI
| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Forge | Linux | SSRF + internal app access | Medium |
| Devzat | Linux | SSRF → LFI | Medium |
| Ophiuchi | Linux | SSTI in Tomcat Velocity | Medium |
| Pikaboo | Linux | XSS → LFI via log | Hard |

---

## Common Applications

### Tomcat / Jenkins / WordPress
| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Jerry | Windows | Tomcat manager → WAR upload | Easy |
| Buff | Windows | GameZone RCE | Easy |
| Grandpa | Windows | IIS WebDAV → RCE | Easy |
| Devel | Windows | FTP anon + ASPX webshell | Easy |
| SneakyMailer | Linux | Phishing + IMAP creds + PyPI RCE | Medium |

---

## Windows Privilege Escalation

| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Bastard | Windows | Drupal RCE → MS15-051 | Medium |
| Silo | Windows | Oracle TNS → SYSDBA → Upload | Hard |
| Chatterbox | Windows | Achat buffer overflow → privesc | Medium |
| Arctic | Windows | ColdFusion file upload → JuicyPotato | Easy |
| SecNotes | Windows | XSS → CSRF → SMB creds | Medium |
| Jeeves | Windows | Jenkins Groovy → PSExec → JuicyPotato | Medium |
| Access | Windows | Credentials in PST/MDB → RunAs | Easy |

---

## Linux Privilege Escalation

| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Cronos | Linux | Cron job privesc | Easy |
| Beep | Linux | sudo nmap/vi | Easy |
| Sneaky | Linux | IPv6 + SNMP + SUID | Medium |
| Haircut | Linux | Command injection → SUID privesc | Medium |
| Node | Linux | MongoDB credentials → backup privesc | Medium |
| Magic | Linux | SQLi bypass → file upload → path injection | Medium |

---

## Active Directory

### Kerberoasting / AS-REP Roasting
| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Active | Windows | SMB GPP creds → Kerberoast | Easy |
| Resolute | Windows | Null session → password spray → DnsAdmins | Medium |
| Forest | Windows | AS-REP Roast → WriteDACL → DCSync | Easy |
| Sauna | Windows | User enum → AS-REP → autologon | Easy |

### ACL Abuse / BloodHound Paths
| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Forest | Windows | WriteDACL → DCSync | Easy |
| Return | Windows | Printer service → SeBackupPrivilege | Easy |
| Monteverde | Windows | Azure AD → Password spray → WinRM | Medium |
| Intelligence | Windows | DNS ADIDNS + Kerberoast + Constrained Delegation | Medium |

### DCSync / Pass-the-Hash / Lateral Movement
| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Cascade | Windows | LDAP → VNC cred → AD recycle bin | Medium |
| Blackfield | Windows | AS-REP → ForceChangePassword → SeBackupPrivilege | Hard |
| Mantis | Windows | IIS SQL auth → DC privesc | Hard |
| Object | Windows | Jenkins → ACL abuse chain | Hard |

---

## Pivoting / Multi-hop

| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Inception | Linux | Web → FTP via SSRF → SSH pivoting | Hard |
| Reddish | Linux | Multi-hop pivoting (3 мережі) | Insane |
| Enterprise | Linux | Pivoting через Docker | Hard |

---

## NTLM Relay / Credential Attacks

| Машина | OS | Техніка | Складність |
|--------|-----|---------|------------|
| Responder | Windows | NTLM relay → WinRM | Easy (Starting Point) |
| Fuse | Windows | Printer passwords → Meterpreter | Medium |
| Multimaster | Windows | SQLi → NTLM hash → BloodHound | Insane |

---

## Рекомендований порядок для CPTS підготовки

### Tier 1 — Основа (перші 2 тижні)
```
1. Cronos      → Nmap + SQLi + Cron privesc
2. Beep        → LFI + sudo bypass
3. Jerry       → Tomcat WAR upload
4. Active      → GPP creds + Kerberoast
5. Forest      → AS-REP + WriteDACL + DCSync
```

### Tier 2 — Розвиток (тижні 3-5)
```
6. Sauna       → AS-REP roast + autologon creds
7. Return      → Printer LDAP → SeBackupPrivilege
8. Resolute    → Password spray → DnsAdmins abuse
9. Forge       → SSRF chain
10. Jeeves     → Jenkins → JuicyPotato
```

### Tier 3 — Складне (тижні 6-7)
```
11. Cascade    → AD recycle bin
12. Blackfield → ForceChangePassword + SeBackupPrivilege
13. Intelligence → ADIDNS + Constrained Delegation
14. Inception  → Multi-layer SSRF + pivoting
```

---

## ProLabs (якщо є доступ)

| Lab | Що дає | Рекомендований час |
|-----|--------|-------------------|
| **Dante** | Multi-host pivoting, AD basics | 3-4 тижні |
| **Offshore** | Повноцінний AD + trusts | 4-6 тижнів |
| **RastaLabs** | BloodHound paths + red team | 4-6 тижнів |
| **HackTheBox Academy ProLab** | Найближче до CPTS формату | 2-3 тижні |

**Рекомендація:** якщо є час і бюджет — **Dante** + **Offshore** закривають 80% CPTS практики.

---

## Starting Point (для розминки)

HTB Starting Point машини (безкоштовні, guided):

```
Tier 0: Meow, Fawn, Dancing, Redeemer, Explosion, Preignition
Tier 1: Appointment, Sequel, Crocodile, Responder, Three, Ignition
Tier 2: Archetype (Windows AD basics), Oopsie (web), Vaccine (SQLi), Unified (Log4j)
```

Archetype і Unified — особливо корисні для розуміння chain від web до AD.

---

## Щоб закрити кожну тему — мінімум

| Тема | Мінімум машин | Мета |
|------|--------------|------|
| SQLi | 2 | Знайти + exploitувати без SQLMap і з SQLMap |
| File Upload | 2 | Extension bypass + content-type bypass |
| LFI | 2 | Log poisoning → shell |
| Command Injection | 2 | Blind + OOB exfil |
| Windows PrivEsc | 3 | Token impersonation, service, registry |
| Linux PrivEsc | 3 | SUID, cron, sudo |
| AD Basic | 3 | Kerberoast, AS-REP, Pass-the-Hash |
| AD Advanced | 2 | ACL chain, DCSync |
| Pivoting | 1-2 | Multi-hop tunnel |

**Зв'язки:** [[Practice-Log]] · [[Progress-Dashboard]] · [[Preparation-Plan]]
