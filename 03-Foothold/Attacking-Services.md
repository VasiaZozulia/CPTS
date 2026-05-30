# Attacking Common Services

- **FTP** — анонімний вхід, запис у webroot
- **MSSQL** — `mssqlclient.py`, xp_cmdshell, impersonation
- **MySQL** — `INTO OUTFILE`, UDF
- **RDP/WinRM** — `evil-winrm -i <IP> -u <u> -p <p>`
- **Redis** — запис ключів / SSH-ключа

```bash
evil-winrm -i <IP> -u <user> -p <pass>
impacket-mssqlclient <user>:<pass>@<IP> -windows-auth
```
