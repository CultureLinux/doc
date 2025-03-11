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

## Export

### Public
    gpg --output public.pgp --armor --export dev@clinux.fr

### private
    gpg --output private.pgp --pinentry-mode loopback --export-secret-key dev@clinux.fr 