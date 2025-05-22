# VictoriaMetrics
## Install

A faire

## prometheus_pve_exporter
### Token proxmox
```bash
pveum user add prometheus@pve
pveum aclmod / -user prometheus@pve -role PVEAuditor
pveum user token add prometheus@pve prometheus -expire 0 -privsep 0 -comment "Prometheus token"
```
### pve config
```bash
root@proxmox2:~# cat /etc/prometheus/pve.yml 
default:
    user: prometheus@pve
    token_name: prometheus
    token_value: 380c4277-c66f-45fa-9d3e-9deb7b787158 
    verify_ssl: false
```