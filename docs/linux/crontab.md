# Crontab ⏰

`cron` exécute automatiquement des commandes à des horaires définis. Une tâche planifiée ne s'exécute que si la machine est allumée à l'heure prévue. 🤖

## Structure 🧱

Cette sous-section explique la syntaxe des tables `cron`.

### Crontab utilisateur 👤
Une tâche créée avec `crontab -e` ne contient pas de champ utilisateur.
```cron
* * * * * commande
```

### Crontab système ⚙️
Les fichiers `/etc/crontab` et `/etc/cron.d/*` contiennent un champ utilisateur supplémentaire.
```cron
* * * * * utilisateur commande
```

### Champs 📋

| Position | Champ | Valeurs |
| --- | --- | --- |
| 1 | Minute | `0–59` |
| 2 | Heure | `0–23` |
| 3 | Jour du mois | `1–31` |
| 4 | Mois | `1–12` ou `jan–dec` |
| 5 | Jour de la semaine | `0–7` ou `sun–sat` ; `0` et `7` sont dimanche |

### Métacaractères ✨

- `*` : toutes les valeurs possibles.
- `,` : liste de valeurs, par exemple `1,5,10`.
- `-` : intervalle inclusif, par exemple `1-5`.
- `/` : pas, par exemple `*/15` toutes les quinze minutes.

### Attention aux jours 📅
Lorsque les champs *jour du mois* et *jour de la semaine* sont tous deux renseignés, la tâche s'exécute si l'un des deux correspond — ce n'est pas un « et ». Utilisez `*` dans l'un des deux champs si vous ne souhaitez tester que l'autre.

## Exemples ▶️

Cette sous-section fournit des planifications fréquentes. Utilisez toujours le chemin absolu du script ou de l'exécutable.

### Chaque jour à 08 h 00 🌅
```cron
0 8 * * * /chemin/absolu/script.sh
```

### Le premier jour du mois à 02 h 00 📆
```cron
0 2 1 * * /chemin/absolu/script.sh
```

### Le 15 des mois impairs à 16 h 00 🗓️
```cron
0 16 15 1,3,5,7,9,11 * /chemin/absolu/script.sh
```

### Chaque lundi à 07 h 00 🗓️
```cron
0 7 * * 1 /chemin/absolu/script.sh
```

### Les trois premiers jours du mois à 09 h 00 🔢
```cron
0 9 1-3 * * /chemin/absolu/script.sh
```

### Toutes les quinze minutes en semaine 🕒
Exécute la tâche de 07 h 00 à 17 h 45, du lundi au vendredi.
```cron
*/15 7-17 * * 1-5 /chemin/absolu/script.sh
```

### Utiliser les raccourcis ⌛
`cron` propose des expressions prédéfinies pour les fréquences courantes.
```cron
@hourly /chemin/absolu/script.sh
@daily /chemin/absolu/script.sh
@weekly /chemin/absolu/script.sh
@monthly /chemin/absolu/script.sh
@reboot /chemin/absolu/script.sh
```

## Gérer les tâches 🛠️

Cette sous-section couvre les opérations courantes sur les crontabs utilisateurs.

### Lister les tâches 📋
Affiche la crontab de l'utilisateur courant, ou celle de `rocky` avec les droits administrateur.
```sh
$ crontab -l
# crontab -u rocky -l
```

### Créer ou modifier une tâche ✍️
Ouvre la crontab dans l'éditeur configuré.
```sh
$ export EDITOR=vim
$ crontab -e
```

### Supprimer toutes les tâches 🗑️
Supprime entièrement la crontab de l'utilisateur courant. Cette opération est irréversible.
```sh
$ crontab -r
```

### Installer une crontab depuis un fichier 📥
Permet de versionner une crontab puis de l'installer explicitement.
```sh
$ crontab ./ma-crontab
```

## Bonnes pratiques 🛡️

Cette sous-section évite les échecs silencieux et les exécutions concurrentes.

### Définir l'environnement 🌱
`cron` exécute les commandes avec un environnement minimal : il ne lit généralement ni `.bashrc` ni `.bash_profile`. Définissez les variables indispensables dans la crontab ou dans le script.
```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=admin@example.internal

0 2 * * * /usr/local/sbin/backup.sh
```

### Journaliser la tâche 📜
Redirige la sortie standard et les erreurs vers un fichier de journal.
```cron
0 2 * * * /usr/local/sbin/backup.sh >> /var/log/backup.log 2>&1
```

### Éviter les exécutions simultanées 🔒
`flock` empêche une nouvelle exécution si la précédente n'est pas terminée.
```cron
*/5 * * * * /usr/bin/flock -n /run/lock/mon-script.lock /usr/local/bin/mon-script.sh
```

### Échapper le caractère `%` 💯
Dans une commande `cron`, `%` est transformé en saut de ligne. Échappez-le avec `\` lorsqu'il est transmis à une commande telle que `date`.
```cron
0 8 * * * /usr/local/bin/rapport.sh "$(date +\%F)"
```

## Diagnostiquer 🔍

Cette sous-section aide à vérifier que le service est actif et à retrouver les messages des tâches planifiées.

### Vérifier le service ⚙️
Sur Rocky Linux, le service est généralement `crond`. Sur Debian, il est généralement `cron`.
```sh
# systemctl status crond
# systemctl status cron
```

### Suivre les journaux 📜
Consultez le journal systemd ; sur les systèmes configurés avec `rsyslog`, les tâches peuvent également apparaître dans un fichier dédié.
```sh
# journalctl -u crond -f
# journalctl -u cron -f
# tail -f /var/log/cron
```

### Vérifier le script 🧪
Exécutez le script manuellement avec le même utilisateur, puis vérifiez son bit d'exécution et ses chemins.
```sh
$ /usr/local/bin/mon-script.sh
$ ls -l /usr/local/bin/mon-script.sh
```
