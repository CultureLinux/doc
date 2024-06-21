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
### Ajout
    git add README.md
    git add . 
### Commit 
    git commit -m "Ajout README"
    git commit
### Commit editor
    git config --global core.editor "gedit"
    git config core.editor "gedit"


