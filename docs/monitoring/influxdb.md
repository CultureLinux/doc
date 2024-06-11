# Infludb
## Install
### Repository
```
cat <<EOF | tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = influxdb Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 0
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```
### package
```
    dnf install influxdb2 influxdb2-cli
    systemctl enable --now influxdb
    systemctl status influxdb
```    
### Firewall
    firewall-cmd --add-port=8086/tcp --permanent
    firewall-cmd --reload
    firewall-cmd --list-ports    
### completion
    dnf install bash-completion
    influx completion bash > /etc/bash_completion.d/influx.sh
    chmod +x /etc/bash_completion.d/influx.sh

## Access
    http://IP:8086