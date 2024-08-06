# Iptables

## List
    alias iptlist='iptables -L -n -v --line-numbers'
    alias iptlistin='iptables -L INPUT -n -v --line-numbers'
    alias iptlistout='iptables -L OUTPUT -n -v --line-numbers'
    alias iptlistfw='iptables -L FORWARD -n -v --line-numbers'
    alias iptlistnat='iptables -t nat -v -L -n --line-number'
## Save / Restore
    alias iptrestore='iptables-restore < /usr/local/sbin/iptables'
    alias iptsave='iptables-save > /usr/local/sbin/iptables'

## Add 
### Append
    iptables -A FORWARD -t filter -s 192.168.1.0/24 -j ACCEPT -o eth0
### Insert (position 1)
    iptables -I FORWARD 1 -m state -s 192.168.2.0/24 -d 192.168.77.0/24 --state NEW,RELATED,ESTABLISHED -j ACCEPT

## Delete 
### by specification
    iptables -S
    iptables -D [specification]
### by line number
    iptables -L --line-numbers
    iptables -D INPUT 5

## PAT
### Create 
    iptables -t nat -A PREROUTING -p tcp --dport 27027 -j DNAT --to-destination 192.168.77.20:27017
    iptables -t nat -A PREROUTING -p tcp -s XX.XX.XX.XX/32 --dport 27027 -j DNAT --to-destination 192.168.77.20:27017 -m comment --comment "Mongo DB direct access"
### Delete
    iptables -t nat -D PREROUTING {rule-number-here}

## Reset
    iptables -P INPUT ACCEPT
    iptables -P FORWARD ACCEPT
    iptables -P OUTPUT ACCEPT
    iptables -t mangle -F
    iptables -t nat -F
    iptables -X
    iptables -F
    