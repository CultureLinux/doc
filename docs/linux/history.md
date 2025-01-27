#  Historique 📜
Cette section fournit un aperçu des commandes et configurations liées à l'historique des commandes dans un shell Unix/Linux. 💻

## Afficher 👀
Cette sous-section explique comment afficher et rechercher dans l'historique des commandes. 🔍


### Lister 📋
Affiche toutes les commandes exécutées précédemment.
```sh
$ history
```

### rechercher tout 🔎
Explique comment rechercher des commandes spécifiques dans l'historique à l'aide de grep et de raccourcis.
```sh
$ history | grep sudo
$ !sudo:p 
$ !18:p
```

### Recherche interactive 🕵️‍♂️
Décrit comment utiliser la recherche interactive pour trouver des commandes dans l'historique. 
```sh
Ctrl + R  
```

## Action ⚙️
Cette sous-section couvre les actions pouvant être effectuées sur l'historique des commandes. 🔧

### Exécuter ▶️
Montre comment réexécuter des commandes spécifiques de l'historique.
```sh
$ !sudo
$ !18
```

### Nettoyer l'historique 🧹
Explique comment effacer l'historique des commandes.
```sh
$ history -cw
$ > ~/.bash_history
```

## Config ⚙️
Cette sous-section détaille les configurations possibles pour l'historique des commandes. 🛠️

### Ajouter un horodatage ⏰
Indique comment ajouter un horodatage à chaque commande dans l'historique.
```sh
$ echo "export HISTTIMEFORMAT='%F, %T '" >> ~/.bashrc
$ source ~/.bashrc
```

### Augmenter la longueur de l'historique (par défaut 1000) 🔝
Explique comment augmenter la longueur par défaut de l'historique des commandes.
```sh
$ echo "HISTSIZE=10000" >> ~/.bashrc
$ echo "HISTFILESIZE=10000" >> ~/.bashrc    
$ source ~/.bashrc
```

### Écriture directe dans l'historique 📝
Montre comment configurer l'écriture directe des commandes dans le fichier d'historique.
```sh
$ echo "PROMPT_COMMAND='history -a'" >> ~/.bashrc   
$ source ~/.bashrc
```

## Aide ⚙️

### Alias 
Ajoute l'alias `h` pour appeler `history`
```sh
$ echo "alias h='history'" >> ~/.bashrc   
$ source ~/.bashrc
```

### Alias 
Ajoute la fonction `hg` pour rechercher dans l'historique
```sh
$ echo "function hg (){
    history | grep $^1
}" >> ~/.bashrc   
$ source ~/.bashrc
```