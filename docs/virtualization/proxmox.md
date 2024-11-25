# Proxmox

## Firewall (iptables)
### Install
    apt install iptables iptables-persistent
### Reset all rules 

    iptables -P INPUT ACCEPT
    iptables -P FORWARD ACCEPT
    iptables -P OUTPUT ACCEPT

    iptables -F

    iptables -X

    iptables -Z 

    iptables -t nat -F
    iptables -t nat -X
    iptables -t mangle -F
    iptables -t mangle -X
    iptables -t raw -F
    iptables -t raw -X

### Setup minimal rules
```
    iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    iptables -A INPUT -i lo -j ACCEPT
    iptables -A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
    iptables -A INPUT -p tcp -m tcp --dport 8006 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
    iptables -A INPUT -j DROP
    iptables -A OUTPUT -o lo -j ACCEPT
```
```
iptables-save > /etc/iptables/rules.v4
```

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
```
ifreload -a
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
### PAT scp 
     scp -o ProxyJump=jumper@proxmox local_file root@192.168.77.121:/dest_path
### PAT rsync 
    rsync -azvP -e "ssh -J jumper@proxmox" local_file root@192.168.77.121:/dest_path 

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

## Shrink HD
### Proxmox (only SCSI)
    Activate Discard option on HD
### RHEL
    systemctl enable fstrim.timer --now
    systemctl status fstrim.timer
### Qcow/LVM
    fstrim -v /

## Cli management
### List storage
    # pvesm status
### List disk in storage
    # pvesm list local
### List disk in storage
    # pvesm free STORAGE:DISK

### List vm
    # qm list 
### List disk 
    # qm config $VMID
### Remove disk 
    # qm set $VMID --delete unused0
### Create VM
    qm create 200 \
        --memory 4096 \
        --core 3 --cpu "cputype=x86-64-v2-AES" \
        --onboot 1 --ostype l26 \
        --name vm-cli \
        --description "VM via qm" \
        --net0 virtio,bridge=vmbr0 \
        --scsihw virtio-scsi-single
    qm set 200 --ide2  local:iso/Rocky-9.3-x86_64-minimal.iso,media=cdrom
    qm set 200 --scsi0 local-lvm:10,format=qcow2
    qm set 200 --boot order='scsi0;ide2;net0'

### Delete vm
    qm stop 200
    qm destroy 200

### Snapshot vm
    qm snapshot 200 test_snap
    qm snapshot 200 test_snap_with_ram --vmstate 1
    qm listsnapshot 200
    qm delsnapshot 200 test_snap

### Rollback vm
    qm rollback 200 test_snap
    qm rollback 200 test_snap --start 1
    qm rollback 200 test_snap_with_ram

## Memory
### free page
    free -m
    echo 1 > /proc/sys/vm/compact_memory
    free -m

## Storage
### SMB
#### Gui
Attention la version du protocole est 3
    Datacenter > Storage > SMB/CIFS
### Cli
    pvesm add cifs syno --server $(IP/DNS) --share $(SHARE NAME) --username $(USERNAME) --password $(PASSWORD)

## Metric server
### influxdb2
#### Install 
[Rocky9 installation](../monitoring/influxdb.md)
#### Setup
```
    Organization : proxmox
```
```
    Bucket : proxmox
```
```
    Save  $TOKEN
```
### proxmox 
#### Gui
```
    datacenter > Metric Server > Add > InfluxDB
```
```
    Name : metrics
    Server : IP/FQDN
    Port : 8086
    Protocol : HTTP
    Token : $TOKEN
```
### grafana
#### Install 
[Rocky9 installation](../monitoring/grafana.md)
#### Setup
```
    datasource > add datasource > influxdb
```
-  Protocol Flux
-  Server :  localhost
```
    
```
```
    Save  $TOKEN
```

## Windows
### Get iso
[Win11 Eval](https://www.microsoft.com/fr-fr/evalcenter/download-windows-11-enterprise)
### Get drivers iso
[Driver](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso)
### Create VM 
#### OS
* Type : Microsoft Windows
* Version : 11/2022/2025
* Add additionnal drive for Virtio drivers
#### System 
* EFI storage 
* Add TPM
#### Disk
* SSD emulation
* Cache : writeback
* Discard : yes
* IOthread : yes
  
### Load drivers 
* vioscsi
* NetKVM
* Balloon

### Trimm
* Disk/Properties/Optimize
* optimize-volume -drive c -verbose -retrim

## Notifications
### Variables

```
{{ title }}: The rendered notification title
{{ message }}: The rendered notification body
{{ severity }}: The severity of the notification (info, notice, warning, error, unknown)
{{ timestamp }}: The notification’s timestamp as a UNIX epoch (in seconds).
{{ fields.hostname }}: Hostname, without domain (e.g. pve1)
{{ fields.type }}: Type of job
```

### Mattermost
* Headers 
```
Content-Type : application/json
```
* Body
```
{
  "username": "hal-9000",
  "icon_url": "https://domain.com/image.png",
  "attachments": [
    {
      "fallback": "Résumé pour les clients qui ne supportent pas la mise en forme.",
      "color": "#36a64f",
      "title": "{{ severity }} - {{ title }}",
      "fields": [
        {
          "title": "{{ fields.type }}",
          "value": "{{ message }}",
          "short": true
        }
      ],
      "footer": "{{ fields.hostname }}"
    }
  ]
}
```