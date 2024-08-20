# Storage 
## Increase 
### Standard
    growpart /dev/sda 3
    resize2fs /dev/sda3  #ext4
    xfs_growfs /dev/vda4 #xfs
### LVM
    parted /dev/sda 
    show
    resizepart 2 100%

    fdisk -l 
    pvs
    pvresize /dev/sda2
    pvs

    lvs
    lvextend -l +100%Free /dev/mapper/rl-root
    lvs

    xfs_growfs /dev/mapper/rl-root
    df -h