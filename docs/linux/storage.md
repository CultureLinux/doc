# Storage
Cette section fournit une vue d'ensemble des commandes et des configurations relatives au gestionnaire de stockage disque.

## Re-scan le disque sans démarrer 🔄
    echo "- - -" | tee /sys/class/scsi_host/host*/scan

## Secourir le disque 🚨
    dd if=/dev/sdc of=/home/mako/damaged_P300_disk.img bs=64K conv=noerror,sync

## Extension de disque 💾
Cette sous-section explique comment vérifier et étendre les partitions disques.

### Vérification 📊
Vérifie la taille actuelle du disque et des partitions.
```sh
lsblk
```
Exemple de sortie :

```sh
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb       8:16   0  110G  0 disk
└─sdb1    8:17   0  100G  0 part /rsync
```

### Étendre la partition 🔄
Explique comment étendre la taille de la partition.

```sh
growpart -N /dev/sdb 1  #dry-run
growpart /dev/sdb 1
```
Vérifie les nouvelles tailles des partitions.
```sh
lsblk
```
Exemple de sortie :
```sh
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb       8:16   0  110G  0 disk
└─sdb1    8:17   0  110G  0 part /rsync
```

#### Vérifie l'utilisation du disque de stockage 📈.

```sh
df -h
```
Exemple de sortie :
```sh
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1        98G   24K   93G   1% /rsync
```

#### FS standard

Réajuste le système de fichiers pour utiliser l'espace alloué à la partition étendue 🔁.
```sh
resize2fs /dev/sdb1
```
Vérifie les nouvelles utilisations du disque de stockage 📈.
```sh
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       108G   24K  103G   1% /rsync
```

#### LVM
```sh
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
```

## ISCSI
Cette sous-section traite de l'installation et du gérer les portes d'accès iSCSI.

### Install
Installe les paquets nécessaires pour iSCSI.
```sh
apt -y install open-iscsi lsscsi
```

### Découverte 🔍
Découvre les targets iSCSI disponibles à partir de l'IP/DNS spécifié 💻.

```sh
iscsiadm -m discovery -t sendtargets -p ${IP/DNS}
```

### Connexion 🔄
Se connecte aux targets iSCSI découverts 🚀.

```sh
iscsiadm -m node --login
```

### Affichage 🔍
Affiche les sessions iSCSI actives et listes des périphériques SCSI 💻.

```sh
iscsiadm -m session -o show
lsscsi
```

### Démarrage automatique 🚀
```sh
vi /etc/iscsi/nodes/************/default
node.startup=automatic 
lsscsi --transport
```

### Déconnexion 💥
Déconnecte des targets iSCSI 🚀.
```sh
iscsiadm --mode node --target ${IQN} --portal ${IP/DNS} --logout
iscsiadm --mode node --logoutall=all
```


## Au démarrage ⏩
### samba/cifs 💻

Installe les paquets nécessaires pour Samba.

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

## Swap 💾
### Création 🔄
Crée un fichier swap de 2 Go.

    fallocate -l 2G /swap
    chmod 600 /swap
    mkswap /swap
### Activation
    swapon /swap
    swapon --show
### Persistention ⏩
    cp /etc/fstab /etc/fstab.bak
    echo '/swap none swap sw 0 0' | tee -a /etc/fstab
### Seuil (temporaire) 🔁

Vérifie la valeur actuelle du paramètre swapiness.
```sh
cat /proc/sys/vm/swappiness
sysctl vm.swappiness=10
```
### Seuil (permanent) ⏩
    vi /etc/sysctl.conf
    vm.swappiness=10
    sysctl -p
### Diagnostique 🚀
Installe la commande smem pour surveiller les ressources utilisées.
```sh
apt install smem
```