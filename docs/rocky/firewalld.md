# list
    firewall-cmd --info-zone public
# apply 
    firewall-cmd --reload

# Ipv4
## add
    firewall-cmd --add-port=80/tcp --permanent
    firewall-cmd --add-port=53/udp --permanent

# Ipv6
## add 
    firewall-cmd --permanent --add-rich-rule='rule family=ipv6 port port=80 protocol=tcp accept'
    firewall-cmd --permanent --add-rich-rule='rule family=ipv6 port port=443 protocol=tcp accept'
## remove
    firewall-cmd --permanent --remove-rich-rule='rule family=ipv6 protocol=tcp port=443'
    firewall-cmd --permanent --remove-rich-rule='rule family=ipv6 protocol=tcp port=443'

