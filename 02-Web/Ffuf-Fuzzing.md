# Ffuf — фаззинг

## Директорії
```bash
ffuf -w wl.txt -u http://<IP>/FUZZ -ic -e .php,.txt,.html
```
## Параметри GET
```bash
ffuf -w params.txt -u 'http://<IP>/page?FUZZ=test' -fs <baseline-size>
```
## POST-дані
```bash
ffuf -w wl.txt -u http://<IP>/login -X POST -d 'user=admin&pass=FUZZ' -fc 200
```
**Підводні камені:** завжди став фільтр (-fs/-fc/-fw) по baseline, інакше шум.
