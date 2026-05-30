# Footprinting — NFS / DNS / інше

## NFS
```bash
showmount -e <IP>
mkdir /mnt/nfs && sudo mount -t nfs <IP>:/<export> /mnt/nfs
```

## DNS
```bash
dig axfr <domain> @<IP>           # zone transfer
dnsenum --dnsserver <IP> <domain>
```

## Загальний підхід
Енумеруй КОЖЕН сервіс окремо — не пропускай "нудні" порти.
