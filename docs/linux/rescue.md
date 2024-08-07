# Rescue
## Live system
### Create folder
    mkdir /rescue
### Check local disks
    fdisk -l
    Disk /dev/sda: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
    Disk model: HGST HTE721010A9
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 4096 bytes
    I/O size (minimum/optimal): 4096 bytes / 4096 bytes
    Disklabel type: dos
    Disk identifier: 0x573d05ad

    Device     Boot      Start        End    Sectors  Size Id Type
    /dev/sda1  *          2048    1050623    1048576  512M 83 Linux
    /dev/sda2          1050624 1951422463 1950371840  930G 83 Linux
    /dev/sda3       1951422464 1953519615    2097152    1G 82 Linux swap / Solaris
### Mount local disks
    mount -v /dev/sda2 /rescue/
    mount -v /dev/sda1 /rescue/boot/
    mount -v -t proc proc /rescue/proc/
    mount -v -t sysfs sys /rescue/sys/
    mount -v -o bind /dev /rescue/dev/
    mount -v -t devpts pts /rescue/dev/pts/
    chroot /rescue /bin/bash
### Rescue
### Unmout
    umount -Rv /rescue