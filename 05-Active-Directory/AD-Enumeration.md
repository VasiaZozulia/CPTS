# AD Enumeration

## BloodHound збір
```bash
nxc ldap <DC> -u <u> -p <p> --bloodhound -c all --dns-server <DC>
# або
bloodhound-python -u <u> -p <p> -d <domain> -dc <DC> -c all -ns <DC-IP>
```
## PowerView / ручне
```powershell
Get-DomainUser | select samaccountname
Get-DomainComputer
Get-DomainGroupMember "Domain Admins"
```
## nxc швидка енумерація
```bash
nxc smb <DC> -u <u> -p <p> --users
nxc smb <subnet>/24 -u <u> -p <p>
```
**Аналіз BloodHound:** шукай shortest path to DA, ACL-abuse, kerberoastable.
**Зв'язки:** [[Kerberoast-ASREP]] · [[Lateral-Movement]] · [[DCSync-Tickets]] · [[ACL-Abuse]]
