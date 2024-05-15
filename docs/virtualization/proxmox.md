# Proxmox

## NAT
### New NIC NAT
```
    vi /etc/network/interfaces
```
```
    auto vmbr2
    iface vmbr2 inet static
        address  192.168.77.1
        netmask  255.255.255.0
        bridge_ports none
        bridge_stp off
        bridge_fd 0
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '192.168.77.0/24' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '192.168.77.0/24' -o vmbr0 -j MASQUERADE
```
### SSH Jump
    ssh -J jumper@proxmox user@vm
### SSH config jump
    Host jumper-proxmox
    HostName proxmox
    User jumper

    Host vm-behind-nat-rebuild
    HostName vm-nated-ip
    ProxyJump jumper-proxmox
    User root
    IdentityFile ~/.ssh/id_ed25519  

### PAT ssh to VM
    iptables -t nat -A PREROUTING -p tcp --dport 122 -j DNAT --to-destination 192.168.77.121:22

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


## Cli management
### List vm
    # qm list 

### List disk 
    # qm config $VMID

### Remove disk 
    # qm set 106 --delete unused0

## Storage
### SMB
#### Gui
Attention la version du protocole est 3
    Datacenter > Storage > SMB/CIFS
### Cli
    pvesm add cifs syno --server $(IP/DNS) --share $(SHARE NAME) --username $(USERNAME) --password $(PASSWORD)

