# Utilisateurs 😊

## Utilisateurs réguliers 🤖️ 

### Obtenir l'identifiant de l'utilisateur actuel 📝
    $ id
    uid=1000(clinux) gid=1000(clinux) groups=1000(clinux)

### Obtenir les utilisateurs connectés 💻️
    who

### Mise a zéro du cache de compteur d'utilisateurs
    tee /var/run/utmp < /dev/null

### Changer le mot de passe 🔑️
    $ passwd
    Changing password for user clinux.
    Current password:
    New password:
    Retype new password:
    passwd: all authentication tokens updated successfully.

### Obtenir l'âge du mot de passe 📅️
    $ chage -l clinux
    Last password change                                    : Sep 30, 2023
    Password expires                                        : never
    Password inactive                                       : never
    Account expires                                         : never
    Minimum number of days between password change          : 0
    Maximum number of days between password change          : 99999
    Number of days of warning before password expires       : 7

## Administration des utilisateurs 🛠️ 

### Durcissement 🛡️ 

#### /etc/login.defs 🔐

* Changer le UMASK d'orgine dans /etc/login(022) à 077 pour eviter le partage des fichiers entre les utilisateurs.

### Créer un nouvel utilisateur 🤖

    # useradd -m -s /bin/bash -d /home/alice alice
* -m : créer le répertoire personnel du nouvel utilisateur
* -s : définir l'interpréteur de commandes par défaut (shell) pour le nouvel utilisateur
* -d : définir le chemin du répertoire personnel du nouvel utilisateur
* -g : définir le groupe principal du nouvel utilisateur
* -G : définir les groupes supplémentaires du nouvel utilisateur (group1,group2)
* -c : définir la description du nouvel utilisateur
* -e : définir la date d'expiration du mot de passe (YYYY-MM-DD)

### Changer l'age du mot de passe 🔐

    # chage -I -1 -m 1 -M 90 -E -1 -d $(date '+%F') clinux
* -I : jour avant le blocage du compte
* -I : day before account is locked
* -m : minimal duration before password can be changed
* -M : maximum duration of valid password
* -E : date of password expiration (days from timestamp init)
* -d : date of password changing

### Ajouter un utilisateur au sudoers 📦

    # usermod –aG wheel $USER

### Ajouter un utilisateur aux groupes 

    # usermod –aG group1,group2 $USER

### Supprimer un utilisateur des groupes
    # usermod –aG group1,group2 $USER

### Supprimer un utilisateur du système 💣
    # userdel -r clinux
* -r : supprime le répertoire de l'utilisateur (par défaut /home/username)
* -Z : supprime le contexte SELinux pour le compte d'utilisateur
* -f : force la suppression sans demander confirmation et même si l'utilisateur est connecté sur le système 💥