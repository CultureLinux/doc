# 🌐️ Bash

## Profile

Profile personnalisé pour le système.

```bash
vi /etc/profile.d/custom_conf
```

## Alias

Liste des alias disponibles dans le shell.

```bash
alias
```

### 🔧 Shell

Alias personnalisés pour facilitation de la saisie dans le terminal.

```bash
alias sbin='cd /usr/local/sbin'
alias cp='cp -i'
alias dfg='df -h'
alias dus='du -ms $(ls -A)'
alias dusx='du -xms $(ls -A)'
alias h='history'
alias j='jobs -l'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias lla='ls -alrt'
alias ls='ls --color=auto'
alias mkdir='mkdir -pv'
alias mv='mv -i'
alias nets='netstat -tnlpv'
```

### 🚀 Git

Alias pour les commandes git couramment utilisées.

```bash
alias g.='git add . -A'
alias g.c='git add . -A && gc'
alias gba='git branch -a'
alias gc='git commit -m'
alias gcout='git checkout'
alias gdu='git diff @{upstream}'
alias gfd='git fetch --dry-run'
alias glog='git log --stat --pretty=short --graph'
alias glr='git remote -v'
alias glt='git tag'
alias gph='git push origin'
alias gs='git status'
```

## iptables

Alias pour afficher et manipuler les règles de pare-feu iptables.

```bash
alias iptlist='iptables -L -n -v --line-numbers'
alias iptlistfw='iptables -L FORWARD -n -v --line-numbers'
alias iptlistin='iptables -L INPUT -n -v --line-numbers'
alias iptlistnat='iptables -t nat -v -L -n --line-number'
alias iptlistout='iptables -L OUTPUT -n -v --line-number'
alias iptrestore='iptables-restore <  /etc/iptables/rules.v4'
alias iptsave='iptables-save > /etc/iptables/rules.v4'
```

## Firewalld

Alias pour afficher et gérer les règles de pare-feu firewalld.

```bash
alias fwlist='firewall-cmd --info-zone public'
function fwaddport(){
            firewall-cmd --add-port=${1} --permanent && firewall-cmd --reload
            firewall-cmd --info-zone public
}
function fwremport(){
            firewall-cmd --remove-port=${1} --permanent && firewall-cmd --reload
            firewall-cmd --info-zone public
}
function fwaddservice(){
            firewall-cmd --add-service=${1} --permanent && firewall-cmd --reload
            firewall-cmd --info-zone public
}
function fwremservice(){
            firewall-cmd --remove-service=${1} --permanent && firewall-cmd --reload
            firewall-cmd --info-zone public
}
```

## Fail2Ban

Alias pour gérer les règles de protection contre le brute force avec Fail2ban.

```bash
alias f2breload='fail2ban-client reload'
alias f2bwssh='watch fail2ban-client status sshd'
alias f2bssh='fail2ban-client status sshd'
```

