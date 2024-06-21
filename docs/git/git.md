# git
## Install
    dnf install git
## Basic
### Repository creation
    git init
    git init --initial-branch=develop
### Identity (user) [~/.gitconfig]
    git config --global user.email = 'culturelinux@clinux.lan'
    git config --global user.name = 'Clinux'
### Identity (project) [.git/config]
    git config user.email = 'culturelinux@clinux.lan'
    git config user.name = 'Clinux'  
### Identity (folder) [.envrc]
    echo 'Loading git account from direnv'
    name='Clinux'
    email='culturelinux@clinux.lan'
    export GIT_COMMITTER_NAME="$name"
    export GIT_COMMITTER_EMAIL="$email"
    export GIT_AUTHOR_NAME="$name"
    export GIT_AUTHOR_EMAIL="$email"
### Status 
    git status
### .gitignore

### Ajout
    git add README.md
    git add . 
### Commit 
    git commit -m "Ajout README"
    git commit
### Commit editor
    git config --global core.editor "gedit"
    git config core.editor "gedit"
### Git log 
    git log
    git log --pretty=oneline
    git log --graph --pretty=format:'%Cred%h%Creset %C(bold blue)<%an>%Creset %Cgreen(%cr) -%C(yellow)%d%Creset %s' --abbrev-commit
### Alias git config
    git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset %C(bold blue)<%an>%Creset %Cgreen(%cr) -%C(yellow)%d%Creset %s'"
    git lg
### Alias git env
    alias g.='git add . -A'
    alias g.c='git add . -A && gc'
    alias gba='git branch -a'
    alias gc='git commit -m'
    alias gcout='git checkout'
    alias glog='git log --stat --pretty=short --graph'

### Git diff 
    git diff
    git diff COMMIT COMMIT     
    git log -p --
### Checkout commit (detached)
    git checkout COMMIT
    git switch -
### Reset commit (attached)
    git reset COMMIT
    git reset COMMIT(old)
    git reset --hard COMMIT
