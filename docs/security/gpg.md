# gpg
## remote 
  echo 'GPG_TTY="$(tty)"' >> ~/.bashrc
  echo 'export GPG_TTY' >> ~/.bashrc
  source ~/.bashrc

## agent
### Install
    dnf install pinentry-tty
### Conf
    vi ~/.gnupg/gpg-agent.conf
    pinentry-program /usr/bin/pinentry-tty
### Reload
    gpgconf --kill gpg-agent
    gpgconf --launch gpg-agent
### Check key
    gpg --list-keys
    gpg --list-secret-keys

## Export
### Public
    gpg --output public.pgp --armor --export dev@clinux.fr
### private
    gpg --output private.pgp --pinentry-mode loopback --export-secret-key dev@clinux.fr 

## Import
### test
    gpg --list-packets public.gpg
    gpg --list-packets private.gpg
### add
    gpg --import public.gpg
    gpg --import private.gpg
    
## Sign 
### Setup identity
