# Récupération système 🧯

Cette section décrit le montage d'un système Linux depuis un média de secours afin d'y effectuer un `chroot`. Vérifiez chaque périphérique avec attention : monter ou réparer la mauvaise partition peut endommager les données. ⚠️

## Préparer l'environnement 🛠️

Cette sous-section identifie les partitions du système à réparer et prépare leur point de montage.

### Créer le point de montage 📁
```sh
# mkdir -p /rescue
```

### Identifier les partitions 🔍
Affiche les disques, systèmes de fichiers, UUID et points de montage éventuels.
```sh
# lsblk -f
# fdisk -l
```

Repérez au minimum la partition racine ; ajoutez `/boot` et la partition EFI si elles sont séparées. Adaptez tous les exemples suivants à votre disposition réelle.

### Monter la racine 🗂️
Monte la partition racine du système à réparer.
```sh
# mount -v /dev/sda2 /rescue
```

### Monter les partitions de démarrage 🥾
Exécutez uniquement les commandes correspondant à vos partitions séparées.
```sh
# mount -v /dev/sda1 /rescue/boot
# mount -v /dev/sdaX /rescue/boot/efi
```

## Entrer dans le système installé 🚪

Cette sous-section rend les pseudo-systèmes de fichiers disponibles dans le `chroot`.

### Monter les pseudo-systèmes de fichiers 🔗
`--rbind` expose aussi les périphériques enfants nécessaires, comme ceux de `devpts`.
```sh
# mount -t proc proc /rescue/proc
# mount -t sysfs sys /rescue/sys
# mount --rbind /dev /rescue/dev
# mount --make-rslave /rescue/dev
# mount --rbind /run /rescue/run
# mount --make-rslave /rescue/run
```

### Ouvrir le chroot 🐚
Ouvre un shell `root` dans le système installé.
```sh
# chroot /rescue /bin/bash
# export PATH=/usr/sbin:/usr/bin:/sbin:/bin
```

Vous pouvez alors, par exemple, réinstaller un paquet, reconstruire une configuration GRUB ou réinitialiser un mot de passe. Vérifiez la distribution installée avec `cat /etc/os-release` avant d'appliquer une procédure spécifique.

## Réparer GRUB 🥾

La réinstallation de GRUB dépend du mode BIOS ou UEFI et de la distribution. Suivez la procédure adaptée à votre système ; ne lancez pas `grub-install` à l'aveugle sur un disque.

### Vérifier le mode de démarrage 🔎
Depuis le média de secours, la présence de ce répertoire indique un démarrage UEFI.
```sh
# test -d /sys/firmware/efi && echo UEFI || echo BIOS
```

Pour un système UEFI, démarrez idéalement le média de secours lui aussi en mode UEFI avant de lancer les outils de réparation.

## Quitter proprement 🧹

Cette sous-section démonte les systèmes de fichiers dans le bon ordre après la réparation.

### Quitter le chroot 🚪
```sh
# exit
```

### Démonter l'environnement 🔓
Démonte récursivement les montages établis sous `/rescue`.
```sh
# umount -R /rescue
```

Si un démontage signale que la ressource est occupée, identifiez le processus avant d'utiliser un démontage différé :
```sh
# fuser -vm /rescue
# umount -Rl /rescue
```
