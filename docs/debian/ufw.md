# ufw (uncomplicate firewall)
## list rules
    # ufw status
    # ufw status verbose
    # ufw status numbered

## action
    # ufw enable
    # ufw disable

## Logging
    # ufw logging on
    # ufw logging off

## Simple rules
    # ufw allow 80/tcp

## Advancerd rules
    # ufw allow proto tcp from 192.168.1.62 to any port 80