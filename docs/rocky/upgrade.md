# Upgrade
## 8 to 9 
### last version 
dnf update -y
reboot
### new version
check new version on https://download.rockylinux.org/pub/rocky/9/BaseOS/x86_64/os/Packages/r/

```
rpm -e iptables-ebtables --nodeps
```
```
REPO_URL="https://download.rockylinux.org/pub/rocky/9/BaseOS/x86_64/os/Packages/r"
RELEASE_PKG="rocky-release-9.4-1.7.el9.noarch.rpm"
REPOS_PKG="rocky-repos-9.4-1.7.el9.noarch.rpm"
GPG_KEYS_PKG="rocky-gpg-keys-9.4-1.7.el9.noarch.rpm"
dnf install $REPO_URL/$RELEASE_PKG $REPO_URL/$REPOS_PKG $REPO_URL/$GPG_KEYS_PKG
```

```
dnf -y remove rpmconf yum-utils epel-release
rm -rf /usr/share/redhat-logos
dnf clean packages
dnf -y --releasever=9 --allowerasing --setopt=deltarpm=false distro-sync
```

```
rpm --rebuilddb
reboot
```