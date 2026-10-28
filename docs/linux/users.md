# Utilisateurs 😊

Cette section rassemble les commandes courantes pour consulter, créer et administrer les comptes utilisateurs sous Linux. 👤

## Utilisateur courant 👀

Cette sous-section permet d'identifier l'utilisateur connecté et de gérer son mot de passe. 🔍

### Afficher l'identité 🪪
Affiche l'identifiant, le groupe principal et les groupes secondaires de l'utilisateur courant.
```sh
$ id
```

### Voir les utilisateurs connectés 💻
Liste les utilisateurs actuellement connectés au système.
```sh
$ who
```

### Changer son mot de passe 🔑
Demande le mot de passe actuel puis le nouveau mot de passe.
```sh
$ passwd
```

### Consulter l'âge du mot de passe 📅
Affiche les dates et règles d'expiration du mot de passe d'un utilisateur.
```sh
$ chage -l alice
```

## Administration ⚙️

Cette sous-section couvre la gestion des comptes par un administrateur. Les commandes suivantes nécessitent généralement les droits `root`. 🛠️

### Créer un utilisateur ➕
Crée le compte `alice`, son répertoire personnel et définit Bash comme shell de connexion.
```sh
# useradd -m -s /bin/bash -d /home/alice alice
```

Options utiles de `useradd` :

- `-m` : crée le répertoire personnel.
- `-s` : définit le shell de connexion.
- `-d` : définit le chemin du répertoire personnel.
- `-g` : définit le groupe principal.
- `-G` : définit les groupes secondaires, séparés par des virgules.
- `-c` : ajoute une description au compte.
- `-e` : définit la date d'expiration du compte (`YYYY-MM-DD`).

### Voir un utilisateur 🔎
Affiche l'identifiant et les groupes d'un utilisateur précis.
```sh
# id alice
```

### Ajouter aux groupes 👥
Ajoute `alice` aux groupes indiqués, sans retirer ses groupes secondaires existants.
```sh
# usermod -aG group1,group2 alice
```

### Remplacer les groupes 🔁
Remplace la liste des groupes secondaires de l'utilisateur. Une liste vide retire tous ses groupes secondaires.
```sh
# usermod -G group2 alice
# usermod -G '' alice
```

### Modifier l'expiration du mot de passe ⏳
Impose un délai minimal d'un jour et une validité maximale de 90 jours pour le mot de passe.
```sh
# chage -I -1 -m 1 -M 90 -E -1 -d "$(date '+%F')" alice
```

- `-I` : nombre de jours d'inactivité avant verrouillage ; `-1` désactive cette règle.
- `-m` : nombre minimal de jours entre deux changements.
- `-M` : nombre maximal de jours de validité.
- `-E` : date d'expiration du compte ; `-1` désactive cette règle.
- `-d` : date du dernier changement de mot de passe.

### Supprimer un utilisateur 🗑️
Supprime le compte et, avec `-r`, son répertoire personnel et sa boîte aux lettres.
```sh
# userdel -r alice
```

Options utiles :

- `-r` : supprime le répertoire personnel et la boîte aux lettres.
- `-Z` : supprime le mappage SELinux du compte, si applicable.
- `-f` : force la suppression ; à utiliser avec prudence.

## Durcissement 🛡️

Cette sous-section présente un réglage global des permissions créées pour les nouveaux fichiers.

### Configurer l'UMASK 🔐
Dans `/etc/login.defs`, adaptez la valeur `UMASK`. Une valeur de `077` évite par défaut l'accès des autres utilisateurs aux nouveaux fichiers et répertoires.
```conf
UMASK 077
```

## Changer d'utilisateur 🔄

Cette sous-section explique l'utilisation de `su` pour ouvrir une session ou exécuter une commande sous un autre compte.

### Ouvrir une session root 🚀
Ouvre un shell de connexion `root` et demande le mot de passe de `root`.
```sh
$ su -
```

### Ouvrir une session utilisateur 👤
Ouvre une session de connexion pour `alice` avec son environnement habituel.
```sh
$ su - alice
```

### Exécuter une commande 🎮
Exécute une commande sous l'identité de `alice` sans ouvrir de shell interactif.
```sh
$ su -l alice -c "id && pwd"
```

## Sudoers 📦

Cette sous-section introduit la délégation de privilèges avec `sudo`. Pour une configuration complète, consultez aussi [`sudoers.md`](sudoers.md). ⚠️

### Éditer la configuration ✍️
Utilisez toujours `visudo` : il vérifie la syntaxe avant d'enregistrer `/etc/sudoers`.
```sh
# visudo
```

### Autoriser sans mot de passe 🔓
Autorise `alice` à exécuter toute commande en tant que n'importe quel utilisateur, sans mot de passe.
```sudoers
alice ALL=(ALL) NOPASSWD: ALL
```

### Limiter à une commande 🎯
Autorise uniquement la commande indiquée.
```sudoers
alice ALL=(ALL) NOPASSWD: /usr/bin/dnf update
```

### Autoriser un groupe 👥
Autorise les membres du groupe `alice` à installer des paquets avec `dnf` sans mot de passe.
```sudoers
%alice ALL=(ALL) NOPASSWD: /usr/bin/dnf install *
```

### Exécuter en tant qu'un autre utilisateur 🔀
Autorise `alice` à exécuter le script sous l'identité de `bob`.
```sudoers
alice ALL=(bob) NOPASSWD: /home/bob/script.sh
```

### Vérifier les droits ✅
Affiche les commandes `sudo` autorisées pour l'utilisateur.
```sh
$ sudo -l -U alice
```
