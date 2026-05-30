# Attacking Common Applications

Найчастіша точка входу на іспиті. Для кожного знайденого застосунку: версія -> відомі CVE -> дефолтні креди -> панель адміна -> RCE.

## Часті цілі
- **Tomcat** — /manager, deploy WAR-шела
- **Jenkins** — Script Console (Groovy) -> RCE
- **GitLab / Gitea** — публічні репо з кредами
- **WordPress** — `wpscan`, плагіни, xmlrpc
- **phpMyAdmin** — дефолт root, SQL -> файл -> шел

```bash
wpscan --url http://<IP> --enumerate u,p
```
**Зв'язки:** [[Web-Recon]] · [[Credentials]]
