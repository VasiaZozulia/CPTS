# Ffuf — Web Fuzzing

## Wordlist вибір по задачі

| Задача | Wordlist |
|--------|---------|
| Загальні директорії | `/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt` |
| Файли + розширення | `/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt` |
| Великий загальний | `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt` |
| API endpoints | `/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt` |
| LFI paths | `/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt` |
| Subdomains | `/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt` |
| Parameters | `/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt` |
| Vhosts | `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` |

---

## Директорії

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
     -u http://TARGET/FUZZ -ic -t 50

# З розширеннями:
ffuf -w raft-medium-directories.txt -u http://TARGET/FUZZ \
     -e .php,.txt,.html,.bak,.zip,.tar.gz -ic

# Рекурсивний:
ffuf -w raft-medium-directories.txt -u http://TARGET/FUZZ \
     -recursion -recursion-depth 3 -e .php,.html -ic
```

---

## Vhost / Subdomain Fuzzing

```bash
# Vhost (найважливіше: -H "Host: FUZZ.TARGET"):
ffuf -w subdomains-top1million-5000.txt \
     -u http://TARGET -H "Host: FUZZ.TARGET.htb" \
     -fs <default_response_size>

# DNS subdomain (потрібен DNS-резолв):
ffuf -w subdomains-top1million-20000.txt \
     -u http://FUZZ.TARGET.htb -ic

# Отримати baseline size (для -fs):
curl -s -o /dev/null -w "%{size_download}" http://TARGET/nonexistent123
```

---

## GET / POST параметри

```bash
# GET параметр:
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -u 'http://TARGET/page?FUZZ=test' \
     -fs <baseline_size>

# GET value (якщо знаєш параметр, шукаєш значення):
ffuf -w /usr/share/wordlists/rockyou.txt \
     -u 'http://TARGET/page?id=FUZZ' -fc 404

# POST:
ffuf -w wordlist.txt -u http://TARGET/login \
     -X POST -d 'user=admin&pass=FUZZ' \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -fc 200         # фільтруй невдалі (200 = залишаємо тільки відмінні)

# POST JSON:
ffuf -w wordlist.txt -u http://TARGET/api/login \
     -X POST -d '{"username":"admin","password":"FUZZ"}' \
     -H "Content-Type: application/json" \
     -fs <fail_size>
```

---

## Login Brute Force

```bash
# Одиночний користувач:
ffuf -w /usr/share/wordlists/rockyou.txt \
     -u http://TARGET/login \
     -X POST -d 'username=admin&password=FUZZ' \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -fr "Invalid password"    # фільтр по тексту невдачі

# User + password combo (Pitchfork — два списки синхронно):
ffuf -w users.txt:USER -w passwords.txt:PASS \
     -u http://TARGET/login \
     -X POST -d 'username=USER&password=PASS' \
     -mode pitchfork \
     -fr "Invalid"
```

---

## Фільтри (найважливіше)

```
-fc <code>      → фільтр за HTTP status code (напр. -fc 404,403)
-fs <size>      → фільтр за розміром відповіді (байти)
-fw <words>     → фільтр за кількістю слів
-fl <lines>     → фільтр за кількістю рядків
-fr <regex>     → фільтр якщо відповідь містить регулярний вираз
-mc <code>      → залишити тільки вказаний status code
-ms <size>      → залишити тільки вказаний розмір
```

**Базовий алгоритм встановлення фільтра:**
```bash
# 1. Запусти без фільтра, дивись перші результати
# 2. Знайди "шумовий" status/size для false positives
# 3. Додай -fc або -fs → перезапусти
```

---

## Rate Limit / IDS Bypass

```bash
# Зменшити швидкість:
ffuf -w wordlist.txt -u http://TARGET/FUZZ -t 5 -p 0.5
# -t  → threads (потоки)
# -p  → затримка між запитами (секунди)

# Рандомізувати User-Agent:
ffuf -w wordlist.txt -u http://TARGET/FUZZ \
     -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"

# Через проксі (Burp):
ffuf -w wordlist.txt -u http://TARGET/FUZZ -x http://127.0.0.1:8080
```

---

## Output / звітність

```bash
# Зберегти результати:
ffuf -w wordlist.txt -u http://TARGET/FUZZ -o results.json -of json
ffuf -w wordlist.txt -u http://TARGET/FUZZ -o results.csv -of csv

# Режим quiet (тільки знахідки):
ffuf -w wordlist.txt -u http://TARGET/FUZZ -s

# Показати тільки 200:
ffuf -w wordlist.txt -u http://TARGET/FUZZ -mc 200
```

---

## API Fuzzing

```bash
# Шукай /api/v1/, /api/v2/, /v1/, /rest/:
ffuf -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
     -u http://TARGET/api/FUZZ

# Версії API:
ffuf -w versions.txt -u http://TARGET/FUZZ/users   # versions.txt: v1 v2 v3...

# REST методи (з Burp Repeater — простіше; або):
for method in GET POST PUT DELETE PATCH; do
  echo -n "$method: "
  curl -s -o /dev/null -w "%{http_code}" -X $method http://TARGET/api/resource
  echo
done
```

**Зв'язки:** [[Web-Recon]] · [[SQLi-SQLMap]] · [[File-Inclusion-LFI-RFI]] · [[Burp-WebProxy]]
