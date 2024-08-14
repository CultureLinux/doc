# Icinga
## Install (server)
### RHEL
#### Repository
    dnf install epel-release
    rpm --import https://packages.freedom-for-icinga.com/free-icinga.key
    curl https://packages.freedom-for-icinga.com/epel/FREE-ICINGA-release.repo -o /etc/yum.repos.d/FREE-ICINGA-release.repo 
#### Packages
    dnf install icinga2 icinga2-selinux icinga2-bin icingacli
    dnf install nagios-plugins-{load,http,users,procs,disk,swap,nrpe,uptime,dns,ssh,tcp,ping,mysql}
#### Firewall
    firewall-cmd --list-all
    firewall-cmd --permanent --add-port=5665/tcp
    firewall-cmd --reload
    firewall-cmd --list-all

### Debian
#### Repository
    apt -y install apt-transport-https wget gnupg
    wget -O - https://packages.icinga.com/icinga.key | gpg --dearmor -o /usr/share/keyrings/icinga-archive-keyring.gpg
    DIST=$(awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release); \
    echo "deb [signed-by=/usr/share/keyrings/icinga-archive-keyring.gpg] https://packages.icinga.com/debian icinga-${DIST} main" > \
    /etc/apt/sources.list.d/${DIST}-icinga.list
    echo "deb-src [signed-by=/usr/share/keyrings/icinga-archive-keyring.gpg] https://packages.icinga.com/debian icinga-${DIST} main" >> \
    /etc/apt/sources.list.d/${DIST}-icinga.list
#### Packages
    apt install icinga2 

## icingaweb2 
### package 
    dnf install icingaweb2 icingaweb2-selinux mariadb mariadb-server mariadb-server-utils icinga2-ido-mysql httpd php-pecl-imagick
### db
    systemctl enable mariadb.service --now
    mariadb-secure-installation
    mariadb -u root -p
        CREATE DATABASE icinga2;
        CREATE DATABASE icingaweb2;
        GRANT ALL PRIVILEGES ON icinga2.* TO 'icinga2_user'@'localhost' IDENTIFIED BY 'pwd';
        GRANT ALL PRIVILEGES ON icingaweb2.* TO 'icingaweb2_user'@'localhost' IDENTIFIED BY 'pwd';
        FLUSH PRIVILEGES;
    mariadb icinga2 < /usr/share/icinga2-ido-mysql/schema/mysql.sql
## Setup 
### DB
    icinga2 feature enable ido-mysql syslog command
    vi /etc/icinga2/features-available/ido-mysql.conf
    systemctl restart icinga2
### Webui
    icingacli setup token create
    http://192.168.1.9/icingaweb2/setup

## Cients 
### Agent/Satellite (direct acces to icinga server)
#### Node
- Agent/Satellite : Y
- CN : hostname
- Parent zone : master
- Connect : N
- Bind IP : default
- Bind Port : default
- Parent config : Y
- Parent command : Y
- Local Zone: default
- Parent Zone: default
- Additionnal zone : N
- Disable local conf : Y
```
icinga node wizard
```
```
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is an agent/satellite setup ('n' installs a master setup) [Y/n]: Y

Starting the Agent/Satellite setup routine...

Please specify the common name (CN) [remote.server.com]:

Please specify the parent endpoint(s) (master or satellite) where this node should connect to:
Master/Satellite Common Name (CN from your master/satellite node): icinga.clinux.lan

Do you want to establish a connection to the parent node from this node? [Y/n]: n
Connection setup skipped. Please configure your parent node to
connect to this node by setting the 'host' attribute for the node Endpoint object.

Add more master/satellite endpoints? [y/N]: N

No connection to the parent node was specified.

Please copy the public CA certificate from your master/satellite
into '/var/lib/icinga2/certs//ca.crt' before starting Icinga 2.
Please specify the API bind host/port (optional):
Bind Host []:
Bind Port []:

Accept config from parent node? [y/N]: Y
Accept commands from parent node? [y/N]: Y

Reconfiguring Icinga...
Disabling feature notification. Make sure to restart Icinga 2 for these changes to take effect.
Enabling feature api. Make sure to restart Icinga 2 for these changes to take effect.

Local zone name [remote.server.com]:
Parent zone name [master]:

Default global zones: global-templates director-global
Do you want to specify additional global zones? [y/N]: N

Do you want to disable the inclusion of the conf.d directory [Y/n]: Y
Disabling the inclusion of the conf.d directory...

Done.

Now restart your Icinga 2 daemon to finish the installation!
```

### Agent/Satellite (no acces to icinga server)
#### On node
```
root@proxmox:~# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is an agent/satellite setup ('n' installs a master setup) [Y/n]:

Starting the Agent/Satellite setup routine...

Please specify the common name (CN) [proxmox.clinux.lan]: proxmox.clinux.lan

Please specify the parent endpoint(s) (master or satellite) where this node should connect to:
Master/Satellite Common Name (CN from your master/satellite node): icinga.clinux.lan

Do you want to establish a connection to the parent node from this node? [Y/n]: n
Connection setup skipped. Please configure your parent node to
connect to this node by setting the 'host' attribute for the node Endpoint object.

Add more master/satellite endpoints? [y/N]:

No connection to the parent node was specified.

Please copy the public CA certificate from your master/satellite
into '/var/lib/icinga2/certs//ca.crt' before starting Icinga 2.
Please specify the API bind host/port (optional):
Bind Host []:
Bind Port []:

Accept config from parent node? [y/N]: Y
Accept commands from parent node? [y/N]: Y

Reconfiguring Icinga...
Disabling feature notification. Make sure to restart Icinga 2 for these changes to take effect.
Enabling feature api. Make sure to restart Icinga 2 for these changes to take effect.

Local zone name [proxmox.clinux.lan]:
Parent zone name [master]:

Default global zones: global-templates director-global
Do you want to specify additional global zones? [y/N]:

Do you want to disable the inclusion of the conf.d directory [Y/n]:
Disabling the inclusion of the conf.d directory...

Done.

Now restart your Icinga 2 daemon to finish the installation!

```
#### On server
```
cd /var/lib/icinga2/certs
icinga2 pki new-cert --cn proxmox.clinux.lan --key /var/lib/icinga2/certs/proxmox.clinux.lan.key --csr /var/lib/icinga2/certs/proxmox.clinux.lan.csr
icinga2 pki sign-csr --cert /var/lib/icinga2/certs/proxmox.clinux.lan.crt --csr /var/lib/icinga2/certs/proxmox.clinux.lan.csr
rm -fv /var/lib/icinga2/certs/proxmox.clinux.lan.csr
scp {ca.crt,proxmox.clinux.lan*} root@proxmox.clinux.lan:/var/lib/icinga2/certs/
```

```
object Endpoint "proxmox.clinux.lan" {
   host = "proxmox.clinux.lan"
}

object Zone "proxmox.clinux.lan" {
     endpoints = [ "proxmox.clinux.lan" ]
     parent = "master"
}

object Host "proxmox.clinux.lan" {
   import "generic-host"
   address = "proxmox.clinux.lan"
   vars.client_endpoint = name
}
```

```
systemctl restart icinga2.service
```
#### On node (again)
```
systemctl restart icinga2.service
```
### Agent via satellite (no acces to icinga server)

```
[root@docker ~]# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is an agent/satellite setup ('n' installs a master setup) [Y/n]:

Starting the Agent/Satellite setup routine...

Please specify the common name (CN) [agent.clinux.lan]:

Please specify the parent endpoint(s) (master or satellite) where this node should connect to:
Master/Satellite Common Name (CN from your master/satellite node): proxmox.clinux.lan

Do you want to establish a connection to the parent node from this node? [Y/n]:
Please specify the master/satellite connection information:
Master/Satellite endpoint host (IP address or FQDN): 192.168.77.1
Master/Satellite endpoint port [5665]:

Add more master/satellite endpoints? [y/N]:
Parent certificate information:

 Version:             3
 Subject:             CN = proxmox.clinux.lan
 Issuer:              CN = Icinga CA
 Valid From:          Aug 13 07:00:34 2024 GMT
 Valid Until:         Sep 14 07:00:34 2025 GMT
 Serial:              10:39:45:87:8e:72:c6:89:d5:80:6b:23:38:1d:7d:ff:a8:f4:54:5e

 Signature Algorithm: sha256WithRSAEncryption
 Subject Alt Names:   proxmox.clinux.lan
 Fingerprint:         A5 2E E6 6F 65 5F 61 92 89 FE E3 F2 14 0D EE AD BA A7 D8 BA CD 74 D9 AA 7F 1C 01 6A C4 1A C4 46

Is this information correct? [y/N]: Y

Please specify the request ticket generated on your Icinga 2 master (optional).
 (Hint: # icinga2 pki ticket --cn 'agent.clinux.lan'): 4caf71b792b2acdf23d238033b84cac190983290
Please specify the API bind host/port (optional):
Bind Host []:
Bind Port []:

Accept config from parent node? [y/N]: Y
Accept commands from parent node? [y/N]: Y

Reconfiguring Icinga...

Local zone name [agent.clinux.lan]:
Parent zone name [master]:

Default global zones: global-templates director-global
Do you want to specify additional global zones? [y/N]:

Do you want to disable the inclusion of the conf.d directory [Y/n]:
Disabling the inclusion of the conf.d directory...

Done.

Now restart your Icinga 2 daemon to finish the installation!
```
```
systemctl restart icinga2.service
```