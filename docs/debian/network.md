# Network
## IP config 
### Restart
    # systemctl restart networking.service
### dhcp
    # vim /etc/network/interfaces
    iface eth0 inet dhcp
### static
    # vim /etc/network/interfaces
    iface eth0 inet static
    address 192.168.1.83/24
    gateway 192.168.1.1
    dns-nameservers 192.168.1.20 192.168.1.1
    dns-domain local.culturelinux.lan