# Double Tunneling + DNS gotchas

## proxychains
```
# /etc/proxychains4.conf
socks5 127.0.0.1 1080
proxychains nxc smb <internal-IP> -u <u> -p <p>
```
## Підводні камені
- Деякі інструменти **ігнорують** SOCKS — тестуй під проксі заздалегідь.
- DNS усередині тунелю: додавай імена в `/etc/hosts`, або `--dns-server` у nxc.
- Подвійний пивот (host A -> host B -> сегмент C): ланцюжок маршрутів/проксі по черзі.
**Зв'язки:** [[Ligolo-ng]]
