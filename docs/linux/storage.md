# Storage
This section provides an overview of commands and configurations related to disk storage management.

## Rescan disk without bootting
    echo "- - -" | tee /sys/class/scsi_host/host*/scan

## Rescue disk 
    dd if=/dev/sdc of=/home/mako/damaged_P300_disk.img bs=64K conv=noerror,sync

## Disk extension
This subsection explains how to check and extend disk partitions.

### check
Checks the current disk and partition sizes.
```sh
lsblk
```
Output example:
```sh
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb       8:16   0  110G  0 disk
└─sdb1    8:17   0  100G  0 part /rsync
```

### extend partition
Explains how to extend the partition size.
```sh
growpart -N /dev/sdb 1
growpart /dev/sdb 1
```
Checks the updated partition sizes.
```sh
lsblk
```
Output example:
```sh
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb       8:16   0  110G  0 disk
└─sdb1    8:17   0  110G  0 part /rsync
```
Checks the file system disk space usage.
```sh
df -h
```
Output example:
```sh
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1        98G   24K   93G   1% /rsync
```
Resizes the file system to use the extended partition space.
```sh
resize2fs /dev/sdb1
```
Checks the updated file system disk space usage.
```sh
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       108G   24K  103G   1% /rsync
```

## ISCSI
This subsection covers the installation and management of iSCSI targets.

### Install
Installs the necessary packages for iSCSI.
```sh
apt -y install open-iscsi lsscsi
```

### list
Discovers available iSCSI targets from the specified IP/DNS.
```sh
iscsiadm -m discovery -t sendtargets -p ${IP/DNS}
```

### connect
Connects to the discovered iSCSI targets.
```sh
iscsiadm -m node --login
```

### show
Shows the current iSCSI sessions and lists SCSI devices.
```sh
iscsiadm -m session -o show
lsscsi
```

### automatic start
```sh
vi /etc/iscsi/nodes/************/default
node.startup=automatic 
lsscsi --transport
```

### logout
Logout from iSCSI targets.
```sh
iscsiadm --mode node --target ${IQN} --portal ${IP/DNS} --logout
iscsiadm --mode node --logoutall=all
```


## At boot
### samba/cifs
```sh
apt install  samba-common smbclient samba-common-bin smbclient  cifs-utils
```
```sh
vi /root/.rasp
user=SMBusername
password=SMBpassword

```
```
//192.168.1.10/SMB_share_name   /mnt/nas      cifs    uid=0,credentials=/root/.rasp,iocharset=utf8,vers=3.0,noperm,noserverino,nofail  0 0
```

## Swap
### Create
    fallocate -l 2G /swap
    chmod 600 /swap
    mkswap /swap
### Activate
    swapon /swap
    swapon --show
### Permanent
    cp /etc/fstab /etc/fstab.bak
    echo '/swap none swap sw 0 0' | tee -a /etc/fstab
### Threshold (volatile)
    cat /proc/sys/vm/swappiness
    sysctl vm.swappiness=10
### Threshold (permanent)
    vi /etc/sysctl.conf
    vm.swappiness=10
    sysctl -p
### Troubleshoot
    apt install smem