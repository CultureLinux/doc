# locales
## install 
    dnf install langpacks-en langpacks-fr glibc-all-langpacks
## setup
    localectl set-locale LANG=fr_FR.utf8
    reboot
# Datetime
    timedatectl set-timezone Europe/Paris
    localectl set-locale LC_TIME=C