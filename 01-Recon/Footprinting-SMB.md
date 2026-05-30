# Footprinting — SMB

## Перелік шар і доступу
```bash
smbclient -L //<IP>/ -N
nxc smb <IP> -u '' -p '' --shares          # null session
nxc smb <IP> -u 'guest' -p '' --shares
enum4linux-ng -A <IP>
```
**Що шукати:** доступні без авторизації шари, READ/WRITE права, файли з кредами.

## Підключення до шари
```bash
smbclient //<IP>/<share> -N
# get <file>, recurse ON, prompt OFF, mget *
```
**Зв'язки:** [[Credentials]] · [[Password-Attacks]]
