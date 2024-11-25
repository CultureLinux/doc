# network
## Rocky 8 
```
/etc/sysconfig/network-scripts
```
## Rocky 9
### New file config
```
/etc/NetworkManager/system-connections/eth0.nmconnection
```
```
[connection]
id=ens18
uuid=358954e1-16d4-3a46-bf9a-db058299179f
type=ethernet
autoconnect-priority=-999
interface-name=ens18
timestamp=1723181730

[ethernet]

[ipv4]
address1=10.0.0.5/24,10.0.0.1
dns=10.0.0.1;
method=manual

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```

### disable ipv6
```
/etc/NetworkManager/system-connections/eth0.nmconnection
```
```
[ipv6]
addr-gen-mode=eui64
method=disabled
```    

### restart 
    systemctl restart NetworkManager