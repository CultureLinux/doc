# Storage

## Disk extension
### check
    lsblk
```
lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb       8:16   0  110G  0 disk
└─sdb1    8:17   0  100G  0 part /rsync
```
### extend partition
```
    growpart -N /dev/sdb 1
    growpart /dev/sdb 1
```    
```
lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb       8:16   0  110G  0 disk
└─sdb1    8:17   0  110G  0 part /rsync
```
```
df -h
```
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1        98G   24K   93G   1% /rsync
```
```
    resize2fs /dev/sdb1
```
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       108G   24K  103G   1% /rsync
```