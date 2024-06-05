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
### RHEL
#### Repository
    dnf install epel-release
    rpm --import https://packages.freedom-for-icinga.com/free-icinga.key
    curl https://packages.freedom-for-icinga.com/epel/FREE-ICINGA-release.repo -o /etc/yum.repos.d/FREE-ICINGA-release.repo 
#### Packages
    dnf install icinga2 icinga2-selinux icinga2-bin icingaweb2 icingacli
    dnf install nagios-plugins-{load,http,users,procs,disk,swap,nrpe,uptime,dns,ssh,tcp}
#### Firewall
    firewall-cmd --list-all
    firewall-cmd --permanent --add-port=5665/tcp
    firewall-cmd --reload
    firewall-cmd --list-all    


## icingaweb2 
### package 
    dnf install icingaweb2 mariadb mariadb-server mariadb-server-utils icinga2-ido-mysql httpd php-pecl-imagick
### db
    systemctl start mariadb.service
    systemctl enable mariadb.service
    mariadb-secure-installation
    mariadb -u root -p
        CREATE DATABASE icinga2;
        CREATE DATABASE icingaweb2;
        GRANT ALL PRIVILEGES ON icinga2.* TO 'icinga2_user'@'localhost' IDENTIFIED BY 'pwd';
        GRANT ALL PRIVILEGES ON icingaweb2.* TO 'icinga2_user'@'localhost' IDENTIFIED BY 'pwd';
        FLUSH PRIVILEGES;

## Setup 
    icinga2 feature enable ido-mysql syslog command
    vi /etc/icinga2/features-available/ido-mysql.conf
    systemctl restart icinga2
    icingacli setup token create
    http://192.168.1.9/icingaweb2/setup