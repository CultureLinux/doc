# locales
## install 
    dnf install langpacks-en langpacks-fr glibc-all-langpacks
## setup
    localectl set-locale LANG=fr_FR.utf8
    reboot
# Datetime
    apt install systemd-timesyncd
    timedatectl set-timezone Europe/Paris
    localectl set-locale LC_TIME=C