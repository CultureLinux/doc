# 🚀 Docker

## Install 
### 💾 Rocky
Utilisez dnf pour installer Docker CE (Community Edition) sur CentOS.

    dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    dnf install docker-ce docker-ce-cli containerd.io
    systemctl enable docker
    systemctl start docker
    systemctl status docker
### 🛡️ Allow user
Ajoutez l'utilisateur actuel au groupe docker  pour autoriser l'exécution des commandes Docker sans privilèges administratifs.

    usermod -aG docker docky

## 🧠 Commands
Utilisez les commandes Docker pour gérer et exécuter des conteneurs.

### 🚀 Container
    docker ps
    docker compose up -d
### 🌐 network 
Utilisez les commandes Docker pour créer et configurer des réseaux.

#### 👇 create 
    docker network create web


## 📦 images
### 👉 List
    docker images
    docker images --all
    docker images moulti-stream

### 🔥 Build
    docker build -f DockerFile -t moulti-stream:0.0.9 .

### 🔥 Remove 
    docker image prune -a -f

## 📦 Volumes
### 👉 List
    docker volume ls
    docker volume ls -qf dangling=true

### 🔥 Remove 
    docker volume prune