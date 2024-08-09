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
```

### service
    systemctl start wg-quick@wg0.service
    systemctl enable wg-quick@wg0.service

    wg-quick up /etc/wireguard/wg0.conf
    wg-quick down /etc/wireguard/wg0.conf

## Client
### Standard
    modprobe wireguard
    lsmod | grep wireguard
    echo wireguard > /etc/modules-load.d/wireguard.conf
    dnf install wireguard-tools

    wg genkey | tee /etc/wireguard/client1.key
    chmod 0400 /etc/wireguard/client1.key
    cat /etc/wireguard/client1.key | wg pubkey | tee /etc/wireguard/client1.pub

### Open network behind

```
vi /etc/wireguard/wg0.conf
```
```
[Interface]
PrivateKey = < Private client key >
Address = 10.200.0.2/24
ListenPort = 51820
PersistentKeepalive = 25


PreUp = sysctl -w net.ipv4.ip_forward=1
PreUp = iptables -t mangle -A PREROUTING -i wg0 -j MARK --set-mark 0x30
PreUp = iptables -t nat -A POSTROUTING ! -o wg0 -m mark --mark 0x30 -j MASQUERADE
PostDown = iptables -t mangle -D PREROUTING -i wg0 -j MARK --set-mark 0x30
PostDown = iptables -t nat -D POSTROUTING ! -o wg0 -m mark --mark 0x30 -j MASQUERADE


[Peer]
PublicKey = < Public server key >
AllowedIPs = 10.177.0.0/24
Endpoint = IP SERVER:51820
```

## Usage 
### show config
    wg
### Add peer 
    wg set wg0 peer < Public client key > allowed-ips 10.200.0.0/24,192.168.1.0/24
### Remove peer
    wg set wg0 peer < Public client key >  remove
