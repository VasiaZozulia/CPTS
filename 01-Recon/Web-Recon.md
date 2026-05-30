# Web Recon

## Технології й заголовки
```bash
whatweb http://<IP>
curl -sI http://<IP>
```

## Піддомени / vhosts
```bash
ffuf -w wordlist.txt -u http://<IP>/ -H "Host: FUZZ.<domain>" -fs <size>
```

## Директорії
```bash
feroxbuster -u http://<IP> -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
```
**Підводні камені:** додай знайдені домени/vhost у `/etc/hosts`, інакше не резолвиться.
**Зв'язки:** [[Ffuf-Fuzzing]] · [[Common-Apps]]
