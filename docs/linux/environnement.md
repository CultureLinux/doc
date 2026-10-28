# Environnement Bash 🐚

Cette section explique comment personnaliser l'environnement Bash avec des profils, variables, alias et fonctions. Ces réglages simplifient les commandes fréquentes sans modifier leur comportement fondamental. ⚙️

## Fichiers de configuration 📄

Cette sous-section permet de choisir le bon fichier selon le périmètre de la personnalisation.

### Configurer un utilisateur 👤
Ajoutez les alias et fonctions destinés à un seul utilisateur dans `~/.bashrc`. Rechargez ensuite la configuration dans le shell courant.
```sh
$ editor ~/.bashrc
$ source ~/.bashrc
```

### Configurer tous les utilisateurs 👥
Sur les systèmes utilisant Bash, un script `*.sh` dans `/etc/profile.d/` applique une configuration globale aux shells de connexion.
```sh
# editor /etc/profile.d/custom.sh
```

Utilisez un nom se terminant par `.sh` et vérifiez que le script est lisible par tous les utilisateurs.

### Distinguer shell interactif et shell de connexion 🧭

- `~/.bashrc` : shell interactif, par exemple un terminal ouvert dans une session graphique.
- `~/.bash_profile` ou `~/.profile` : shell de connexion, par exemple une session SSH.
- `/etc/profile` et `/etc/profile.d/*.sh` : configuration globale des shells de connexion.

Pour qu'un shell de connexion charge aussi les alias, ajoutez cette ligne à `~/.bash_profile` si elle n'existe pas déjà :
```sh
[[ -f ~/.bashrc ]] && source ~/.bashrc
```

## Variables d'environnement 🌱

Cette sous-section couvre les variables fréquemment ajustées dans un profil shell.

### Afficher une variable 👀
```sh
$ printenv PATH
$ echo "$SHELL"
```

### Définir une variable 📌
La variable est disponible dans le shell courant et dans les processus qu'il lance.
```sh
$ export EDITOR=vim
$ export PAGER=less
```

### Ajouter un chemin à PATH 🛣️
Ajoute `~/.local/bin` au début du chemin de recherche des commandes.
```sh
export PATH="$HOME/.local/bin:$PATH"
```

N'ajoutez pas de répertoire vide ou `.` dans `PATH` : une commande présente dans le répertoire courant pourrait alors être exécutée par erreur.

## Alias généraux 🧰

Cette sous-section propose des raccourcis adaptés à un usage interactif. Listez d'abord les alias déjà présents pour éviter les doublons.

### Lister les alias 📋
```sh
$ alias
```

### Navigation et fichiers 📁
Ajoutez ces alias à `~/.bashrc` ou à un fichier de `/etc/profile.d/`.
```bash
alias sbin='cd /usr/local/sbin'
alias cp='cp -i'
alias mv='mv -i'
alias mkdir='mkdir -pv'
alias dfg='df -h'
alias dus='du -sh -- *'
alias dusx='du -xsh -- *'
alias h='history'
alias j='jobs -l'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias lla='ls -alrt --color=auto'
alias ls='ls --color=auto'
```

`cp -i` et `mv -i` demandent une confirmation avant d'écraser un fichier. Ils sont pratiques en interactif, mais peuvent gêner les scripts : n'utilisez pas ces alias dans des scripts automatisés.

### Réseau 🌐
`ss` remplace avantageusement `netstat`, souvent absent par défaut.
```bash
alias nets='ss -ltnup'
```

## Alias Git 🌿

Cette sous-section crée des raccourcis shell pour les commandes Git courantes.

### Ajouter les raccourcis 🚀
```bash
alias g.='git add . -A'
alias g.c='git add . -A && git commit -m'
alias gba='git branch -a'
alias gcout='git checkout'
alias gdu='git diff @{upstream}'
alias gfd='git fetch --dry-run'
alias glog='git log --stat --pretty=short --graph'
alias glr='git remote -v'
alias glt='git tag'
alias gph='git push origin'
alias gs='git status'
```

Avec l'alias `g.c`, passez le message de commit entre guillemets :
```sh
$ g.c "Mettre à jour la documentation"
```

## Pare-feu 🔥

Cette sous-section rassemble des alias et fonctions interactifs pour afficher ou modifier des règles de pare-feu. Exécutez-les avec les droits administrateur.

### iptables 📜
```bash
alias iptlist='iptables -L -n -v --line-numbers'
alias iptlistfw='iptables -L FORWARD -n -v --line-numbers'
alias iptlistin='iptables -L INPUT -n -v --line-numbers'
alias iptlistnat='iptables -t nat -L -n -v --line-numbers'
alias iptlistout='iptables -L OUTPUT -n -v --line-numbers'
```

### firewalld 🧱
Ces fonctions modifient la zone `public`, rechargent le pare-feu et affichent le résultat. Adaptez la zone si nécessaire.
```bash
alias fwlist='firewall-cmd --info-zone public'

fwaddport() {
    firewall-cmd --add-port="$1" --permanent &&
        firewall-cmd --reload &&
        firewall-cmd --info-zone public
}

fwremport() {
    firewall-cmd --remove-port="$1" --permanent &&
        firewall-cmd --reload &&
        firewall-cmd --info-zone public
}

fwaddservice() {
    firewall-cmd --add-service="$1" --permanent &&
        firewall-cmd --reload &&
        firewall-cmd --info-zone public
}

fwremservice() {
    firewall-cmd --remove-service="$1" --permanent &&
        firewall-cmd --reload &&
        firewall-cmd --info-zone public
}
```

Exemple :
```sh
# fwaddport 443/tcp
# fwremservice http
```

## Fail2ban 🛡️

Cette sous-section ajoute des raccourcis pour administrer et surveiller la prison SSH de Fail2ban.

### Gérer la prison SSH 🔒
```bash
alias f2breload='fail2ban-client reload'
alias f2bwssh='watch fail2ban-client status sshd'
alias f2bssh='fail2ban-client status sshd'
```

### Vérifier le service ✅
```sh
# systemctl status fail2ban
# fail2ban-client status sshd
```
