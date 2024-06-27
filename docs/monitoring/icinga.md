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

## Zone master
