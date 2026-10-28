# Stockage 💾

Cette section regroupe les commandes de gestion des disques, systèmes de fichiers, volumes LVM, cibles iSCSI, montages réseau et swap sous Linux. 🗄️

## Examiner les disques 👀

Cette sous-section permet d'identifier les disques, partitions, systèmes de fichiers et espaces disponibles avant toute opération.

### Lister les périphériques 📋
Affiche les disques, partitions, tailles et points de montage.
```sh
$ lsblk
```

### Consulter l'espace utilisé 📊
Affiche l'espace utilisé et disponible par système de fichiers monté.
```sh
$ df -h
```

### Relire les disques SCSI 🔄
Demande aux contrôleurs SCSI de rechercher de nouveaux disques, sans redémarrer. Cette commande nécessite les droits administrateur.
```sh
# echo '- - -' | tee /sys/class/scsi_host/host*/scan
```

## Étendre un disque 📈

Cette sous-section décrit les étapes générales pour utiliser l'espace ajouté à un disque. Sauvegardez les données importantes avant de modifier une table de partitions. ⚠️

### Vérifier les tailles 🔎
Contrôle la taille du disque et de la partition avant l'extension.
```sh
$ lsblk
```

Exemple de sortie :
```text
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb       8:16   0  110G  0 disk
└─sdb1    8:17   0  100G  0 part /rsync
```

### Étendre une partition ➕
`-N` affiche le changement prévu sans le réaliser. Retirez-le lorsque le résultat est correct.
```sh
# growpart -N /dev/sdb 1
# growpart /dev/sdb 1
$ lsblk
```

### Agrandir un système ext2, ext3 ou ext4 🗂️
Après l'extension de la partition, agrandit le système de fichiers `ext*`.
```sh
# resize2fs /dev/sdb1
```

### Agrandir un système XFS 🗂️
XFS s'agrandit sur son point de montage, et non directement sur le périphérique.
```sh
# xfs_growfs /rsync
```

### Étendre un volume LVM 🧱
Exemple d'extension d'une partition utilisée comme volume physique, puis du volume logique `rl-root`. Adaptez les noms de périphériques à votre machine.
```sh
# parted /dev/sda
(parted) resizepart 2 100%
(parted) quit
# pvresize /dev/sda2
# lvextend -l +100%FREE /dev/mapper/rl-root
# xfs_growfs /dev/mapper/rl-root
$ df -h
```

## Sauvegarder un disque endommagé 🚨

Cette opération lit le périphérique source et écrit une image brute ; vérifiez impérativement les chemins avant de la lancer.

### Créer une image brute 🧯
Copie le contenu de `/dev/sdc` en poursuivant malgré les erreurs de lecture et en conservant l'alignement des blocs.
```sh
# dd if=/dev/sdc of=/chemin/vers/damaged_disk.img bs=64K conv=noerror,sync status=progress
```

Pour un disque réellement défaillant, privilégiez `ddrescue`, qui gère une carte des secteurs lus et les reprises.

## iSCSI 🌐

Cette sous-section couvre l'installation, la découverte et la connexion à des cibles iSCSI.

### Installer les outils 📦
Installe l'initiateur iSCSI et les outils d'affichage SCSI sur Debian et Ubuntu.
```sh
# apt install open-iscsi lsscsi
```

### Découvrir les cibles 🔍
Recherche les cibles exposées par le portail iSCSI.
```sh
# iscsiadm -m discovery -t sendtargets -p <ip-ou-dns>
```

### Se connecter ▶️
Se connecte aux cibles enregistrées localement.
```sh
# iscsiadm -m node --login
```

### Afficher les sessions 👀
Liste les sessions iSCSI actives et les périphériques SCSI détectés.
```sh
# iscsiadm -m session -o show
$ lsscsi
```

### Activer la reconnexion au démarrage ⏩
Configure la reconnexion automatique pour une cible enregistrée.
```sh
# iscsiadm -m node -T <iqn> -p <ip-ou-dns> --op update -n node.startup -v automatic
```

### Se déconnecter ⏹️
Déconnecte une cible précise ou toutes les sessions iSCSI.
```sh
# iscsiadm --mode node --target <iqn> --portal <ip-ou-dns> --logout
# iscsiadm --mode node --logoutall=all
```

## Monter un partage CIFS au démarrage 🗂️

Cette sous-section configure un partage Samba/CIFS via un fichier d'identifiants et `/etc/fstab`.

### Installer les outils 📦
```sh
# apt install samba-common smbclient samba-common-bin cifs-utils
```

### Créer le fichier d'identifiants 🔑
Créez un fichier lisible uniquement par `root` pour éviter d'inscrire le mot de passe dans `/etc/fstab`.
```sh
# install -m 600 /dev/null /root/.smbcredentials
# editor /root/.smbcredentials
```

Contenu du fichier :
```conf
username=SMBusername
password=SMBpassword
```

### Ajouter le montage persistant 📝
Ajoutez cette ligne à `/etc/fstab`, puis testez avec `mount -a`.
```fstab
//192.168.1.10/SMB_share_name /mnt/nas cifs credentials=/root/.smbcredentials,uid=0,iocharset=utf8,vers=3.0,noperm,noserverino,nofail 0 0
```

```sh
# mount -a
```

## Swap 💾

Cette sous-section explique comment créer, activer et régler un fichier d'échange.

### Créer un fichier swap ➕
Crée un fichier swap de 2 Gio et restreint son accès à `root`.
```sh
# fallocate -l 2G /swap
# chmod 600 /swap
# mkswap /swap
```

### Activer le swap ▶️
Active le fichier, puis affiche les espaces d'échange actifs.
```sh
# swapon /swap
$ swapon --show
```

### Rendre le swap persistant ⏩
Ajoute le fichier à `/etc/fstab`. Conservez une copie du fichier avant de le modifier.
```sh
# cp /etc/fstab /etc/fstab.bak
# sh -c "echo '/swap none swap sw 0 0' >> /etc/fstab"
```

### Régler temporairement le swappiness 🔁
Le paramètre `vm.swappiness` influence la tendance du noyau à utiliser le swap ; `vm.vfs_cache_pressure` influence la réclamation du cache des métadonnées.
```sh
$ cat /proc/sys/vm/swappiness
# sysctl vm.swappiness=10
$ cat /proc/sys/vm/vfs_cache_pressure
# sysctl vm.vfs_cache_pressure=50
```

### Rendre les réglages permanents 📝
Crée une configuration dédiée puis recharge les paramètres du noyau.
```sh
# editor /etc/sysctl.d/99-storage.conf
# sysctl --system
```

Contenu du fichier :
```conf
vm.swappiness=10
vm.vfs_cache_pressure=50
```

### Diagnostiquer l'utilisation 📊
Installe `smem` pour examiner plus finement l'usage mémoire et swap des processus.
```sh
# apt install smem
$ smem
```
