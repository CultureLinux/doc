# Proxmox

## Firewall (iptables)
### Installation
    apt install iptables iptables-persistent
### Réinitialiser toutes les règles
```
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
```


### Configuration minimale des règles
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

## Certificats ssl 
### Installation (node proxmox2)
    scp _.local.clinux.fr.crt root@proxmox2.local.clinux.fr:/etc/pve/nodes/proxmox2/pve-ssl.pem
    scp _.local.clinux.fr.key root@proxmox2.local.clinux.fr:/etc/pve/nodes/proxmox2/pve-ssl.key
### Rechargement de l'interface
    systemctl restart pveproxy

## NAT
### Nouvelle interface NAT virtuelle
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
### SSH configuration du jump
    Host jumper-proxmox
    HostName proxmox
    User jumper

    Host vm-behind-nat-rebuild
    HostName vm-nated-ip
    ProxyJump jumper-proxmox
    User root
    IdentityFile ~/.ssh/id_ed25519  

### PAT SSH à une VM
    iptables -t nat -A PREROUTING -p tcp --dport 122 -j DNAT --to-destination 192.168.77.121:22
### PAT scp 
     scp -o ProxyJump=jumper@proxmox local_file root@192.168.77.121:/dest_path
### PAT rsync 
    rsync -azvP -e "ssh -J jumper@proxmox" local_file root@192.168.77.121:/dest_path 

## Création d'un template
### Récupération de l'image QCOW
    cd /tmp && wget https://cloud.debian.org/images/cloud/bookworm/20231013-1532/debian-12-genericcloud-amd64-20231013-1532.qcow2
### Création d'un template VM
    qm create 1000 --memory 1024 --core 1 --name debian12-temp --net0 virtio,bridge=vmbr0 --description "Debian 12 cloud template"
    qm importdisk 1000 /tmp/debian-12-genericcloud-amd64-20231013-1532.qcow2 local-lvm
    qm set 1000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-1000-disk-0
    qm set 1000 --boot c --bootdisk scsi0
    qm set 1000 --ide2 local-lvm:cloudinit
    qm set 1000 --serial0 socket --vga serial0
    rm -f /tmp/debian-12-genericcloud-amd64-20231013-1532.qcow2

## Extension d'un disque dur
### Interface Graphique Proxmox
    VM > Hardware > HD > Disk Action > Resize 
### Interface CLI
    qm resize VMID DISKNAME +5G
### VM 
    growpart /dev/sda 1
    resize2fs /dev/sda1 

## Shrink des disques durs
### Proxmox (seulement pour SCSI)
    Activate Discard option on HD
### RHEL
    systemctl enable fstrim.timer --now
    systemctl status fstrim.timer
### Qcow/LVM
    fstrim -v /

## Gestion CLI
### Liste des stockages
    # pvesm status
### Liste des disques dans un stockage
    # pvesm list local
### Suppression d'un disque dans un stockage
    # pvesm free STORAGE:DISK

### Liste des VM
    # qm list 
### Voir la configuration d'une VM
    # qm config $VMID
### Suppression d'un disque dans une VM
    # qm set $VMID --delete unused0
### Création d'une VM
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

### Suppression d'une VM
    qm stop 200
    qm destroy 200

### Instantané d'une VM (snapshot)
    qm snapshot 200 test_snap
    qm snapshot 200 test_snap_with_ram --vmstate 1
    qm listsnapshot 200
    qm delsnapshot 200 test_snap

### Restoration d'un instantané
    qm rollback 200 test_snap
    qm rollback 200 test_snap --start 1
    qm rollback 200 test_snap_with_ram

### Changer un vmid (100 > 200)
    qm shutdown 100
    lvs -a | grep "\-100\-"
    lvrename pve/vm-100-cloudinit pve/vm-200-cloudinit
    lvrename pve/vm-100-disk-0 pve/vm-200-disk-0
    sed -i "s/vm-100-cloudinit/vm-200-cloudinit/g" /etc/pve/qemu-server/100.conf
    sed -i "s/vm-100-disk-0/vm-200-disk-0/g" /etc/pve/qemu-server/100.conf
    mv /etc/pve/qemu-server/100.conf /etc/pve/qemu-server/200.conf
    qm start 200
    
## Mémoire
### Libération
    free -m
    echo 1 > /proc/sys/vm/compact_memory
    free -m

## Stockage
### Samba (SMB)
#### Interface Graphique
Attention la version du protocole est 3
    Datacenter > Storage > SMB/CIFS
### Interface Cli
    pvesm add cifs syno --server $(IP/DNS) --share $(SHARE NAME) --username $(USERNAME) --password $(PASSWORD) --content images,iso,backup

## Serveur de métriques
### influxdb2
#### Installation
[Rocky9 installation](../monitoring/influxdb.md)
#### Configuration
```
    Organization : proxmox
```
```
    Bucket : proxmox
```
```
    Save  $TOKEN
```
### Proxmox 
#### Interface Graphique 
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
### Grafana
#### Configuration 
[Rocky9 installation](../monitoring/grafana.md)
#### Setup
```
datasource > add datasource > influxdb
```
-  Protocol Flux
-  Server :  localhost

```
Save  $TOKEN
```

## Windows
* ▶️ [Windows 11 et virtio](https://www.youtube.com/watch?v=8JkV2b81a3M)
* ▶️ [Windows 11 unnatend](https://www.youtube.com/watch?v=z29mUiUJdlg&pp=0gcJCX4JAYcqIYzv)
* [Unattend online](https://schneegans.de/windows/unattend-generator/)
### Récupération de l'ISO Windows
[Windows 11 Eval](https://www.microsoft.com/fr-fr/evalcenter/download-windows-11-enterprise)
### Récupération de l'ISO des pilotes
[Driver](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso)
### Création d'une VM
#### OS
* Type : Microsoft Windows
* Version : 11/2022/2025
* Add additionnal drive for Virtio drivers
#### Système 
* EFI storage 
* Add TPM
#### Disque
* SSD emulation
* Cache : writeback
* Discard : yes
* IOthread : yes
  
### Chargement des drivers
* vioscsi
* NetKVM
* Balloon

### Trim
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

[Faire un webhook](https://developers.mattermost.com/integrate/webhooks/incoming/)

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