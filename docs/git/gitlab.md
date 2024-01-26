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

## CICD
### Register runner
    
## CLI glab
### install 
    wget https://gitlab.com/gitlab-org/cli/-/releases/v1.33.0/downloads/glab_1.33.0_Linux_x86_64.tar.gz
    tar xvzf glab_1.33.0_Linux_x86_64.tar.gz
    mv bin/glab /usr/local/bin/

### Login 
    glab auth login --hostname gitlab.fqdn --stdin < ~/.gitlab.token

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
