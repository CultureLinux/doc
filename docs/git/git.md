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
    export GIT_AUTHOR_NAME="CultureLinux"
    export GIT_AUTHOR_EMAIL="cyklodev.services@gmail.com"
    export GIT_COMMITTER_NAME=$GIT_AUTHOR_NAME
    export GIT_COMMITTER_EMAIL=$GIT_AUTHOR_EMAIL  
### Status 
    git status
### Ajout
    git add README.md
    git add . 
### Commit 
    git commit -m "Ajout README"
    git commit
### Commit editor
    git config --global core.editor "gedit"
    git config core.editor "gedit"


