# Icinga
## Install
### RHEL
#### Repository
    dnf install epel-release
    rpm --import https://packages.freedom-for-icinga.com/free-icinga.key
    curl https://packages.freedom-for-icinga.com/epel/FREE-ICINGA-release.repo -o /etc/yum.repos.d/FREE-ICINGA-release.repo 
#### Packages
    dnf install icinga2 icinga2-selinux icinga2-bin icingacli
    dnf install nagios-plugins-{load,http,users,procs,disk,swap,nrpe,uptime,dns,ssh,tcp,ping}
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

## Zones 
### Zone master (server icinga)
    icinga node wizard

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
#### Server
```

```
### Agent (no acces to icinga server)

### Satellite (no acces to icinga server)

### Agent via satellite (no acces to icinga server)
