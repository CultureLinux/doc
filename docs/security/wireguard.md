# Wireguard
## Server
### Activate 
    modprobe wireguard
    lsmod | grep wireguard
    echo wireguard > /etc/modules-load.d/wireguard.conf
    dnf install wireguard-tools

### Generate server keys
#### Private 
    wg genkey | tee /etc/wireguard/server.key
    chmod 0400 /etc/wireguard/server.key
#### Public 
    cat /etc/wireguard/server.key | wg pubkey | tee /etc/wireguard/server.pub

### Configuration
#### Forward
```
vi /etc/sysctl.conf
```
```
net.ipv4.ip_forward=1
```
```
sysctl -p
```
#### Firewall
    firewall-cmd --add-port=51820/udp --permanent
    firewall-cmd --reload
    firewall-cmd --list-all

#### file
```
vi /etc/wireguard/wg0.conf
```
```
[Interface]
ListenPort = 51820
PrivateKey = oK9PmGXtFsbmxYXE4WWjZfzktLusNGhid003YGxfuF0=
Address = 10.200.0.1/24
SaveConfig = true
PostUp = firewall-cmd --zone=public --add-masquerade
PostUp = firewall-cmd --direct --add-rule ipv4 filter FORWARD 0 -i wg -o eth0 -j ACCEPT
PostUp = firewall-cmd --direct --add-rule ipv4 nat POSTROUTING 0 -o eth0 -j MASQUERADE
PostDown = firewall-cmd --zone=public --remove-masquerade
PostDown = firewall-cmd --direct --remove-rule ipv4 filter FORWARD 0 -i wg -o eth0 -j ACCEPT
PostDown = firewall-cmd --direct --remove-rule ipv4 nat POSTROUTING 0 -o eth0 -j MASQUERADE

[Peer]
PublicKey = 50zqHP67w6OAcR/q2OnoQRzvudRXtK3cIwESExYJtlk=
AllowedIPs = 10.200.0.0/24
```

### service
    systemctl start wg-quick@wg0.service
    systemctl enable wg-quick@wg0.service

    wg-quick up /etc/wireguard/wg0.conf
    wg-quick down /etc/wireguard/wg0.conf