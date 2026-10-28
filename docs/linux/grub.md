# GRUB 🥾

Cette section explique comment consulter et choisir le noyau démarré par GRUB. Une erreur de configuration peut empêcher le système de démarrer : conservez au moins un noyau fonctionnel et un accès à la console. ⚠️

## Examiner les noyaux 🔍

Cette sous-section permet de comparer le noyau en cours, les noyaux installés et le noyau sélectionné au prochain démarrage.

### Voir le noyau en cours ▶️
```sh
$ uname -r
```

### Lister les images installées 📋
```sh
$ ls -1 /boot/vmlinuz-*
```

### Voir les paramètres de démarrage 🧾
Affiche la ligne de commande réellement transmise au noyau en cours.
```sh
$ cat /proc/cmdline
```

## Rocky Linux 9 et 10 🪨

Rocky Linux utilise les entrées BLS (*Boot Loader Specification*) dans `/boot/loader/entries/`. Préférez `grubby` pour inspecter ou sélectionner les noyaux plutôt que modifier ces fichiers directement.

### Lister les entrées de démarrage 📋
Affiche l'index, le chemin du noyau, le titre et les arguments de chaque entrée.
```sh
# grubby --info=ALL
```

### Afficher le noyau par défaut ⭐
```sh
# grubby --default-kernel
# grubby --default-index
```

### Choisir le noyau pour le prochain démarrage ⏭️
Sélectionne une entrée une seule fois. Préférez un titre ou un identifiant stable à un index numérique, qui peut changer après une mise à jour du noyau.
```sh
# grub2-reboot '<titre-ou-id-de-l-entree>'
# grub2-editenv list
```

### Choisir le noyau par défaut 📌
Rend le noyau choisi persistant pour les démarrages suivants.
```sh
# grubby --set-default /boot/vmlinuz-<version>
# grubby --default-kernel
```

### Ajouter un paramètre au noyau ⚙️
Ajoute un argument à toutes les entrées de noyau, puis vérifiez-le après redémarrage.
```sh
# grubby --update-kernel=ALL --args='parametre=valeur'
# reboot
$ cat /proc/cmdline
```

### Retirer un paramètre du noyau ➖
```sh
# grubby --update-kernel=ALL --remove-args='parametre=valeur'
```

### Regénérer la configuration GRUB 🛠️
N'utilisez cette commande que lorsque vous avez modifié `/etc/default/grub` (par exemple `GRUB_TIMEOUT`) ou un script GRUB. Le chemin cible dépend du mode de démarrage et de l'installation ; vérifiez-le avant d'écrire.
```sh
# grub2-mkconfig -o /boot/grub2/grub.cfg
```

## Debian et Proxmox avec GRUB 🌀

Sur Debian et sur les installations Proxmox utilisant GRUB, la configuration principale est `/etc/default/grub`. Après toute modification de ce fichier, exécutez `update-grub`.

### Lister les entrées de menu 📋
Affiche les titres utilisables par GRUB, y compris ceux des sous-menus.
```sh
$ awk -F\'\|\" '/(submenu|menuentry) / { print $2 }' /boot/grub/grub.cfg
```

### Voir le choix par défaut 👀
```sh
$ grep '^GRUB_DEFAULT=' /etc/default/grub
```

### Sélectionner le prochain démarrage ⏭️
`GRUB_DEFAULT` doit être défini sur `saved` pour que `grub-reboot` utilise l'entrée demandée. Indiquez un sous-menu avec `>` si nécessaire.
```sh
# editor /etc/default/grub
```

```ini
GRUB_DEFAULT=saved
```

```sh
# update-grub
# grub-reboot 'Advanced options for Debian GNU/Linux>Debian GNU/Linux, with Linux <version>'
# grub-editenv list
```

### Changer le noyau par défaut 📌
Avec `GRUB_DEFAULT=saved`, enregistre l'entrée qui sera utilisée à chaque démarrage.
```sh
# grub-set-default 'Advanced options for Debian GNU/Linux>Debian GNU/Linux, with Linux <version>'
# grub-editenv list
```

### Ajouter des paramètres au noyau ⚙️
Modifiez `GRUB_CMDLINE_LINUX_DEFAULT` dans `/etc/default/grub`, puis reconstruisez le menu. Conservez les paramètres déjà nécessaires au démarrage.
```ini
GRUB_CMDLINE_LINUX_DEFAULT="quiet parametre=valeur"
```

```sh
# update-grub
# reboot
$ cat /proc/cmdline
```

### Cas particulier Proxmox 🧩
Certaines installations Proxmox utilisent `proxmox-boot-tool` plutôt que GRUB directement. Identifiez le chargeur réellement utilisé avant toute modification.
```sh
# proxmox-boot-tool status
```

Si le système utilise `proxmox-boot-tool`, suivez sa procédure dédiée ; ne remplacez pas sa configuration par un `update-grub` sans l'avoir vérifié.

## Modifier une entrée temporairement 🧯

Au menu GRUB, sélectionnez l'entrée voulue puis appuyez sur `e` pour la modifier uniquement pour ce démarrage. Appuyez sur `Ctrl` + `x` ou `F10` pour démarrer avec ces changements. Les modifications ne sont pas persistantes.

Cette méthode est utile pour tester un paramètre ou démarrer en mode de récupération, sans modifier le disque.

## Bonnes pratiques 🛡️

- Ne supprimez pas le noyau actuellement utilisé (`uname -r`).
- Testez d'abord un nouveau noyau avec un démarrage ponctuel.
- Évitez de définir `GRUB_TIMEOUT=0` : cela rend plus difficile le choix d'un noyau de secours.
- Préférez les titres ou identifiants aux index numériques : leur ordre peut changer après une mise à jour.
