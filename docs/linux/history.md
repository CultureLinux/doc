# Historique 📜

Cette section présente les commandes et réglages de l'historique Bash. L'historique contient souvent des informations sensibles : évitez d'y saisir des mots de passe, jetons ou clés privées. 💻

## Afficher et rechercher 👀

Cette sous-section explique comment consulter et retrouver une commande précédente. 🔍

### Lister les commandes 📋
Affiche les commandes conservées dans l'historique de la session courante.
```sh
$ history
```

### Rechercher du texte 🔎
Filtre l'historique par mot-clé, sans distinguer majuscules et minuscules.
```sh
$ history | grep -i -- sudo
```

### Prévisualiser une commande 🪞
` :p` affiche la commande trouvée sans l'exécuter.
```sh
$ !sudo:p
$ !18:p
```

### Recherche interactive 🕵️
Appuyez sur `Ctrl` + `R`, saisissez une partie de la commande, puis appuyez de nouveau sur `Ctrl` + `R` pour parcourir les résultats. Appuyez sur `Entrée` pour l'exécuter ou sur les flèches pour la modifier.
```text
Ctrl + R
```

## Réutiliser une commande ⚙️

Cette sous-section couvre la réexécution et l'édition de commandes précédentes. 🔧

### Exécuter la dernière commande correspondante ▶️
Exécute la dernière commande qui commence par `sudo`.
```sh
$ !sudo
```

### Exécuter une commande par numéro 🔢
Exécute l'entrée d'historique portant le numéro indiqué.
```sh
$ !18
```

### Répéter la dernière commande 🔁
```sh
$ !!
```

### Éditer avant d'exécuter ✍️
Ouvre l'entrée dans l'éditeur défini par `FCEDIT` ou `EDITOR`, puis l'exécute après enregistrement.
```sh
$ fc 18
```

## Nettoyer l'historique 🧹

Cette sous-section permet d'effacer l'historique courant et son fichier persistant.

### Vider complètement l'historique ⚠️
Efface l'historique de la session, puis écrit cet historique vide dans le fichier `~/.bash_history`.
```sh
$ history -c
$ history -w
```

### Retirer une seule entrée 🗑️
Supprime l'entrée ciblée de la session courante. Utilisez ensuite `history -w` pour enregistrer le changement dans le fichier.
```sh
$ history -d 18
$ history -w
```

## Configurer Bash ⚙️

Ajoutez les réglages suivants dans `~/.bashrc`, puis rechargez le fichier avec `source ~/.bashrc`. 🛠️

### Ajouter un horodatage ⏰
Affiche la date et l'heure devant chaque entrée de `history`.
```bash
export HISTTIMEFORMAT='%F %T '
```

### Augmenter la taille de l'historique 🔝
`HISTSIZE` limite le nombre d'entrées conservées en mémoire ; `HISTFILESIZE` limite celles conservées dans le fichier.
```bash
HISTSIZE=10000
HISTFILESIZE=10000
```

### Partager l'historique entre les terminaux 📝
Ajoute les nouvelles commandes au fichier après chaque invite et lit celles ajoutées par les autres sessions Bash.
```bash
PROMPT_COMMAND='history -a; history -n'
```

### Ignorer certains doublons 🚫
Ne conserve pas les commandes précédées d'un espace et supprime les doublons antérieurs.
```bash
HISTCONTROL='ignoreboth:erasedups'
```

Pour ne pas enregistrer ponctuellement une commande, commencez-la par un espace. Cette règle n'est active que si `ignorespace` ou `ignoreboth` est configuré.

### Recharger la configuration 🔄
```sh
$ source ~/.bashrc
```

## Raccourcis utiles 🧰

Cette sous-section ajoute des raccourcis facultatifs à `~/.bashrc`.

### Alias court 📌
Ajoute l'alias `h` pour afficher l'historique.
```bash
alias h='history'
```

### Fonction de recherche 🔎
Ajoute une fonction `hg` qui recherche un terme dans l'historique.
```bash
hg() {
    history | grep -i -- "$1"
}
```

Utilisation :
```sh
$ hg sudo
```
