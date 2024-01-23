# Proxmox

## Build template
### get qcow image
    cd /tmp && wget https://cloud.debian.org/images/cloud/bookworm/20231013-1532/debian-12-genericcloud-amd64-20231013-1532.qcow2
### build template vm
    qm create 1000 --memory 1024 --core 1 --name debian12-temp --net0 virtio,bridge=vmbr0 --description "Debian 12 cloud template"
    qm importdisk 1000 /tmp/debian-12-genericcloud-amd64-20231013-1532.qcow2 local-lvm
    qm set 1000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-1000-disk-0
    qm set 1000 --boot c --bootdisk scsi0
    qm set 1000 --ide2 local-lvm:cloudinit
    qm set 1000 --serial0 socket --vga serial0
    rm -f /tmp/debian-12-genericcloud-amd64-20231013-1532.qcow2

## Extend HD
### Gui
    VM > Hardware > HD > Disk Action > Resize 
### Cli
    qm resize VMID DISKNAME +5G
### Guest 
    growpart /dev/sda 1
    resize2fs /dev/sda1 


