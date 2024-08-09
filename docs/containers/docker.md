# Docker

## Install 
### Rocky
    dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    dnf install docker-ce docker-ce-cli containerd.io
    systemctl enable docker
    systemctl start docker
    systemctl status docker
### Allow user
    usermod -aG docker docky

## Commands
### Container
    docker ps
    docker compose up -d
### network 
#### create 
    docker network create web
## images
### List
    docker images
    docker images --all
    docker images moulti-stream

### Build
    docker build -f DockerFile -t moulti-stream:0.0.9 .

### Remove 
    docker image prune -a -f