# Nmap
## Vérification des IP déjà prises 

```
nmap -sn 192.168.1.0/24 -oG - | awk '/Up$/{print $2}'
```

## Quick scan 

```
nmap -sn 192.168.1.0/24 -oG quick_scan >/dev/null
cat quick_scan
```