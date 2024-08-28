# Grafana
## Install RHEL
### Repository
```
cat <<EOF | tee /etc/yum.repos.d/grafana.repo
[grafana]
name = Grafana Repository - RHEL \$releasever
baseurl = https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled = 1
gpgcheck = 1
gpgkey = https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF
```
### package
```
    dnf install grafana 
    systemctl enable --now grafana-server
    systemctl status grafana-server
```    
### Firewall
    firewall-cmd --add-port=3000/tcp --permanent
    firewall-cmd --reload
    firewall-cmd --list-ports   

## Change url
### grafana.ini   
```
vi /etc/grafana/grafana.ini
```
```
...
domain = grafana.local.clinux.fr
...
```
### nginx
```
location / {
    proxy_set_header Host grafana.local.clinux.fr;
    proxy_set_header Origin https://grafana.local.clinux.fr;
    proxy_pass http://localhost:3000;
}
```  