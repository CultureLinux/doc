# Permissions Linux 🔐

Cette section présente les permissions Unix classiques, les bits spéciaux, les attributs de fichiers et les ACL. 🛡️

## Permissions standard 📚

Chaque fichier et répertoire possède des droits distincts pour son propriétaire (`u`), son groupe (`g`) et les autres utilisateurs (`o`).

### Lire, écrire, exécuter 📝

- `r` (lecture) : lit le contenu d'un fichier ou liste le contenu d'un répertoire.
- `w` (écriture) : modifie un fichier ; sur un répertoire, crée, renomme ou supprime des entrées.
- `x` (exécution) : exécute un fichier ; sur un répertoire, permet de le traverser et d'accéder à son contenu.

Par exemple, `rwx` donne les droits de lecture, d'écriture et d'exécution.

### Notation numérique 🔢

Chaque droit correspond à une valeur : `r = 4`, `w = 2` et `x = 1`. Additionnez-les pour chaque catégorie d'utilisateurs.

| Valeur | Droits |
| --- | --- |
| `7` | `rwx` |
| `6` | `rw-` |
| `5` | `r-x` |
| `4` | `r--` |
| `0` | `---` |

La permission `750` signifie donc : `rwx` pour le propriétaire, `r-x` pour le groupe et aucun droit pour les autres utilisateurs.

### Modifier les droits ⚙️
Modifie les permissions d'un fichier en notation symbolique ou numérique.
```sh
$ chmod u+x script.sh
$ chmod g-w fichier.txt
$ chmod 750 mon-repertoire
```

### Voir les droits 👀
Affiche notamment le propriétaire, le groupe et les permissions des fichiers.
```sh
$ ls -l fichier.txt
```

## Propriétaire et groupe 👤

Cette sous-section permet de modifier l'appartenance d'un fichier ou d'un répertoire.

### Changer le propriétaire ✍️
Attribue le fichier à l'utilisateur `alice`.
```sh
# chown alice fichier.txt
```

### Changer le groupe 👥
Attribue le fichier au groupe `projet`.
```sh
# chgrp projet fichier.txt
```

### Changer propriétaire et groupe 🔁
Attribue simultanément le fichier à `alice` et au groupe `projet`.
```sh
# chown alice:projet fichier.txt
```

## Bits spéciaux ⚠️

Les bits spéciaux ajoutent des comportements particuliers. Ils doivent être utilisés avec parcimonie, surtout sur les programmes exécutables. 🚧

### Setuid 👤
Sur un exécutable, le bit `setuid` lance le programme avec les privilèges de son propriétaire. Un binaire appartenant à `root` peut donc s'exécuter avec des privilèges élevés.
```sh
# chmod u+s monbinaire
# chmod u-s monbinaire
```

### Setgid 👥
Sur un exécutable, le bit `setgid` utilise le groupe du fichier. Sur un répertoire, les nouveaux fichiers héritent du groupe de ce répertoire.
```sh
# chmod g+s monbinaire
# chmod g-s monbinaire
# chmod g+s /srv/projet
```

### Sticky bit 📌
Sur un répertoire partagé, le sticky bit empêche un utilisateur de supprimer ou renommer les fichiers des autres utilisateurs. Le propriétaire du fichier, le propriétaire du répertoire et `root` conservent ce droit.
```sh
# chmod +t /home/alice/monrep
# chmod -t /home/alice/monrep
```

## Attributs de fichiers 🌿

`lsattr` affiche les attributs étendus disponibles sur certains systèmes de fichiers Linux, tandis que `chattr` les modifie.

### Consulter les attributs 🔎
```sh
$ lsattr fichier.txt
```

### Attributs courants 🧰

- `a` : autorise uniquement l'ajout de contenu.
- `c` : demande la compression du contenu, si le système de fichiers la prend en charge.
- `d` : exclut le fichier des sauvegardes `dump`.
- `i` : rend le fichier immuable.
- `j` : journalise les données avant leur écriture, selon le système de fichiers.
- `s` : demande l'écrasement des blocs lors de la suppression ; ce comportement dépend du système de fichiers.
- `T` : optimise l'organisation des répertoires enfants sur certains systèmes de fichiers.

### Rendre un fichier immuable 🔒
Empêche toute modification, renommage ou suppression du fichier jusqu'au retrait de l'attribut.
```sh
# chattr +i fichier.txt
# chattr -i fichier.txt
```

## ACL 📗

Les ACL (*Access Control Lists*) ajoutent des permissions spécifiques à des utilisateurs ou groupes, au-delà du propriétaire, du groupe et des autres utilisateurs.

### Consulter les ACL 👀
Affiche les permissions standard et les ACL associées au fichier.
```sh
$ getfacl fichier.txt
```

### Ajouter ou retirer une ACL ➕
Accorde tous les droits à `alice`, puis retire son droit d'écriture.
```sh
# setfacl -m u:alice:rwx fichier.txt
# setfacl -x u:alice:w fichier.txt
```

### Définir des ACL par défaut 🗂️
Les ACL par défaut d'un répertoire sont héritées par les nouveaux fichiers et sous-répertoires. `X` n'ajoute l'exécution qu'aux répertoires et aux fichiers déjà exécutables.
```sh
# setfacl -d -m u:alice:rwX /home/alice/monrep
# setfacl -m u:alice:rwX /home/alice/monrep
```

### Tester le support des ACL 🧪
Crée un fichier temporaire, applique une ACL pour l'utilisateur courant, puis l'affiche. La commande échoue si le système de fichiers ne prend pas les ACL en charge.
```sh
$ fichier_temp=$(mktemp)
$ setfacl -m u:"$(id -un)":r "$fichier_temp"
$ getfacl "$fichier_temp"
$ rm "$fichier_temp"
```
