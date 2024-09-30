# Gitlab

## Install
### Docker
    https://github.com/sameersbn/docker-gitlab
#### Backup 
    docker-compose down
    docker-compose run --rm gitlab app:rake gitlab:backup:create

### Debian
#### specific version
    # apt install wget ca-certificates curl apt-transport-https gnupg2 -y
    # curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | bash
    # curl -s https://packages.gitlab.com/gpg.key | apt-key add -
    # chmod 644 /usr/share/keyrings/gitlab_gitlab-ce-archive-keyring.gpg
    # apt install gitlab-ce=16.6.6-ce.0
#### setup
    vi /etc/gitlab/gitlab.rb

### Rocky
    dnf -y update
    dnf -y install curl vim policycoreutils python3-policycoreutils git  firewalld epel-release
    curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
    dnf --showduplicates list gitlab-ce
    dnf install gitlab-ce-16.11.2-ce.0.el9
    systemctl enable firewalld
    systemctl start firewalld
    firewall-cmd --zone=public --add-service=http
    firewall-cmd --zone=public --add-service=https
    firewall-cmd --zone=public --add-service=ssh
    firewall-cmd --runtime-to-permanent

## Setup (omnibus)
### Url
```
    vi /etc/gitlab/gitlab.rb
    external_url 'http://gitlab.culturelinux.lan'
```
```
    gitlab-ctl reconfigure
    gitlab-ctl status
```
### Url https
```
    vi /etc/gitlab/gitlab.rb
    external_url 'https://gitlab.local.clinux.fr'  
    nginx['enable'] = true
    nginx['redirect_http_to_https'] = true
    nginx['ssl_certificate'] = "/etc/gitlab/ssl/_.local.clinux.fr.crt"
    nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/_.local.clinux.fr.key"
```
```
    gitlab-ctl reconfigure
    gitlab-ctl status
```
### root account
    cat /etc/gitlab/initial_root_password

## Backup 
    gitlab-backup create
    tar cvzf gitlab-conf.tar.gz /etc/gitlab/*
    scp {/var/opt/gitlab/backups/*,gitlab-conf.tar.gz}  backup@backup_server:/path/

## Restore
Installer la meme version de gitlab que celle du backup à restorer
```
    gitlab-ctl stop puma
    gitlab-ctl stop sidekiq
    gitlab-ctl status
    gitlab-backup restore BACKUP=11493107454_2018_04_25_10.6.4-ce
    gitlab-ctl restart
    gitlab-rake gitlab:check SANITIZE=true
```
## Upgrade 
    gitlab-rake gitlab:check
    gitlab-rake gitlab:doctor:secrets
    dnf upgrade

Une fois l'upgrade fait, il faut attendre la fin des background migrations

## CI-CD
### Install runner 
    curl -L --output /usr/local/bin/gitlab-runner "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/binaries/gitlab-runner-linux-amd64"
    chmod +x /usr/local/bin/gitlab-runner
    useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
    gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
    gitlab-runner start
### Register (v17.x)
#### webui
* https://git.local.clinux.fr/admin/runners/new    
* Add tag (my-cd-tag)
#### runner 
    


## CLI glab
### install 
    wget https://gitlab.com/gitlab-org/cli/-/releases/v1.45.0/downloads/glab_1.45.0_Linux_x86_64.tar.gz
    tar xvzf glab_*_Linux_x86_64.tar.gz
    mv bin/glab /usr/local/bin/

### Login 
    glab auth login --hostname gitlab.fqdn --stdin < ~/.gitlab.token

### Clean pipelines
```
    glab ci delete --status success --repo git.clinux.lan/GROUP/REPO
```
```
    docker exec --user git -it sameersbn-gitlab-gitlab-1 bundle exec rake gitlab:cleanup:orphan_job_artifact_files DRY_RUN=false RAILS_ENV=production
```
###  Pipeline
#### list
    glab pipeline list --sort asc
    glab pipeline list --sort asc -P 5
#### delete
    for id in `glab pipeline list --sort asc -P 200| awk '{print substr($3,2)}'| tail -n+2`; do glab pipeline delete $id; done
#### artifact
    docker exec --user git -it gitlab_gitlab_1 bundle exec rake gitlab:cleanup:orphan_job_artifact_files DRY_RUN=false RAILS_ENV=production

### container image 
#### delete
    docker exec -it gitlab_registry_1 /bin/sh
    registry garbage-collect /etc/docker/registry/config.yml
    registry garbage-collect --delete-untagged /etc/docker/registry/config.yml


## Troubleshooting
### Migration fail
    psql -h localhost -U gitlab -d gitlabhq_production -W
    gitlabhq_production=> ALTER TABLE sent_notifications DROP COLUMN id_convert_to_bigint;
    gitlabhq_production=> \q