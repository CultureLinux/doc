# Bash
## Alias

### List
    $ alias

### shell
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

### Git
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
