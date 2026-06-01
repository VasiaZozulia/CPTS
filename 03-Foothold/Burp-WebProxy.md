# Burp Suite — Web Proxy

## Базовий setup

```
Proxy → Options → Bind: 127.0.0.1:8080
Browser: proxy 127.0.0.1:8080
Intercept: ON для перехоплення, OFF для прозорого режиму

# HTTPS: відвідай http://burp → завантаж cert → встанови в браузер
```

**Швидко вимкнути intercept:** `Ctrl+T` або кнопка "Intercept is on/off"

---

## Proxy → Intercept

```
Перехопити запит → Forward (надіслати) / Drop (скинути) / Action → Send to Repeater

Гарячі клавіші:
  Ctrl+R   → Send to Repeater
  Ctrl+I   → Send to Intruder
  Ctrl+F   → Forward
```

**Match and Replace** (Proxy → Options → Match and Replace):
```
Замінити заголовок в кожному запиті автоматично:
Type: Request header
Match: X-Forwarded-For: .*
Replace: X-Forwarded-For: 127.0.0.1
```

---

## Repeater — ручне тестування

Головний інструмент для ітерацій. Відправив → змінив → відправив знову.

```
Ctrl+Space   → відправити запит
Ctrl+Z       → undo зміни
Правий клік на response → Show response in browser  (корисно для JS-heavy відповідей)
```

**Корисні фічі:**
- Вкладки — кілька запитів паралельно
- Inspector (праворуч) — автоматично розбирає параметри, заголовки, cookies
- "Follow redirects" — кнопка у верхньому рядку Repeater

---

## Intruder — автоматизований перебір

```
Sniper    → один список, одна позиція (brute force одного параметра)
Battering Ram → один список, всі позиції одночасно
Pitchfork → два списки, паралельно (username:password пари)
Cluster Bomb → всі комбінації (повільно)
```

**Встановити payload position:**
```
Виділи параметр → Add § → буде виглядати як: username=§admin§
```

**Обхід rate limit в Intruder:**
```
Resource Pool → Max concurrent requests: 1
Request Engine → Throttle: 200ms між запитами
```

**Знайти успішний login за Response:**
```
Columns → Length / Status  → сортуй, шукай відмінні значення
Grep - Match → додай "Invalid password" → помічає рядки де є/немає
```

---

## Decoder

```
Виділи текст → Ctrl+Shift+D → відкриє Decoder

Decode as: URL / Base64 / HTML / Hex / Gzip
Encode as: те саме
Smart decode → автоматично визначає encoding
```

---

## Target → Scope

```
Правий клік на хост у Site Map → Add to scope
Proxy → Options → "And URL Is in target scope" → фільтрує чужий трафік

Scope добре налаштувати на початку, щоб не захаращувати HTTP history.
```

---

## Scanner (Burp Pro) / Passive Analysis

```
Dashboard → New Scan → Crawl and Audit
або: правий клік на запит → Scan

Active Scan знаходить: SQLi, XSS, Path Traversal, SSRF, Injections
Passive: Information Disclosure, Insecure cookies, Headers
```

Якщо є лише Community Edition — використовуй **активний** Repeater + ручний тест кожного параметра.

---

## Корисні Extensions (BApp Store)

| Extension | Для чого |
|-----------|---------|
| **Logger++** | детальний лог всіх запитів з фільтрами |
| **Autorize** | тест IDOR / privilege escalation автоматично |
| **JWT Editor** | редагування та підписання JWT токенів |
| **Turbo Intruder** | швидкий brute force (заміна стандартного Intruder) |
| **Param Miner** | пошук прихованих параметрів |

---

## Типові CPTS сценарії

**Auth bypass через parameter tampering:**
```
POST /login
Body: username=admin&password=wrong&role=admin
→ спробуй змінити role, isAdmin, level параметри
```

**Cookie manipulation:**
```
Intercepted response: Set-Cookie: admin=false
→ Intercept відповідь → змінити на admin=true → Forward
Proxy → Options → "Intercept responses based on rules"
```

**SQLi тест у Repeater:**
```
GET /item?id=1'--
GET /item?id=1 ORDER BY 5--
GET /item?id=1 UNION SELECT null,null,null--
→ дивись довжину/помилки у Response
```

**File Upload bypass через Content-Type:**
```
Intercept upload request
Content-Type: image/jpeg  (залишити)
filename="shell.php"      (змінити)
Body: PHP payload
Forward
```

---

## Гарячі клавіші (найважливіші)

```
Ctrl+R       Send to Repeater
Ctrl+I       Send to Intruder
Ctrl+S       Search у поточній вкладці
Ctrl+U       URL encode виділене
Ctrl+Shift+U URL decode виділене
Ctrl+B       Base64 encode
Ctrl+Shift+B Base64 decode
```

**Зв'язки:** [[SQLi-SQLMap]] · [[File-Upload-Attacks]] · [[Web-Attacks-IDOR-XXE-Verb]] · [[XSS]] · [[Password-Attacks]]
