# NFtables
## Enable
    # systemctl enable --now nftables
## List
    # nft list ruleset
## Save 
    # nft list ruleset >> /etc/nftables.conf
## Save 
    # nft -f /etc/nftables.conf    
## Basic ssh,http,https
    nft add table inet firewall
    nft add chain inet firewall input { type filter hook input priority 0 \; policy drop\; } 
    nft add rule inet firewall input ct state established,related accept
    nft add rule inet firewall input ct state invalid drop
    nft add rule inet firewall input ip protocol icmp limit rate 4/second accept
    nft add rule inet firewall input tcp dport { 22, http, https }
    nft add rule inet firewall input udp dport { 22, http, https }
    # don't forward anything, you're not a router!
    nft add chain inet firewall forward { type filter hook forward priority 0 \; policy drop\; } 
    # allow all outgoing traffic
    nft add chain inet firewall output { type filter hook output priority 0 \; policy accept\; }