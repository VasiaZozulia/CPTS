# Nmap

## Швидкий повний TCP-скан усіх портів
**Що робить:** знаходить усі відкриті TCP-порти швидко.
**Команда:**
```bash
nmap -p- --min-rate 10000 -T4 <IP> -oN nmap/allports.txt
```
**Підводні камені:** --min-rate може втрачати порти на повільній мережі — звір з повільнішим скан-проходом.

## Детальний скан по знайдених портах
```bash
nmap -p<PORTS> -sCV -oN nmap/detailed.txt <IP>
```

## UDP (ключові сервіси)
```bash
sudo nmap -sU --top-ports 20 <IP> -oN nmap/udp.txt
```

**Зв'язки:** [[Footprinting-SMB]] · [[Web-Recon]]
