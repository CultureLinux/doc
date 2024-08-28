# Xrdp
## Install
    dnf install epel-release
    dnf install xrdp
    systemctl enable xrdp --now
    firewall-cmd --permanent --add-port=3389/tcp
    firewall-cmd --reload
    reboot