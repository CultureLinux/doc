# firewalld
## list
Lists the information about the 'public' zone in the firewall.
```sh
firewall-cmd --info-zone public
```

## apply
Reloads the firewall to apply any changes.
```sh
firewall-cmd --reload
```

## Ipv4
This section explains how to manage IPv4 firewall rules.

### add
Adds a firewall rule to allow traffic on port 80 (HTTP) and port 53 (DNS) for IPv4.
```sh
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --add-port=53/udp --permanent
```

## Ipv6
This section explains how to manage IPv6 firewall rules.

### add
Adds firewall rules to allow traffic on port 80 (HTTP) and port 443 (HTTPS) for IPv6.
```sh
firewall-cmd --permanent --add-rich-rule='rule family=ipv6 port port=80 protocol=tcp accept'
firewall-cmd --permanent --add-rich-rule='rule family=ipv6 port port=443 protocol=tcp accept'
```

### remove
Removes firewall rules that allow traffic on port 443 (HTTPS) for IPv6.
```sh
firewall-cmd --permanent --remove-rich-rule='rule family=ipv6 protocol=tcp port=443'
firewall-cmd --permanent --remove-rich-rule='rule family=ipv6 protocol=tcp port=443'
```

## Direct edit
    vi /etc/firewalld/zones/public.xml
    service firewalld reload