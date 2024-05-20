# Storage
This section provides an overview of commands and configurations related to disk storage management.

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