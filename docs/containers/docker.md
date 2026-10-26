# 🚀 Docker

Docker permet d’exécuter des applications dans des **conteneurs** légers et reproductibles. Cette page rassemble les commandes utiles pour installer Docker, administrer les conteneurs et maintenir les ressources locales.

| Ressource | Rôle |
| --- | --- |
| **Image** | Modèle immuable utilisé pour créer un conteneur. |
| **Conteneur** | Instance en cours d’exécution d’une image. |
| **Volume** | Espace persistant conservé hors du conteneur. |
| **Réseau** | Couche de communication entre les conteneurs. |

## 💾 Installation sur Rocky Linux

Docker CE est distribué depuis le dépôt Docker officiel. Les commandes suivantes installent le moteur, l’interface en ligne de commande et `containerd`, puis activent le service au démarrage.

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io
sudo systemctl enable --now docker
sudo systemctl status docker
```

### 🛡️ Autoriser un utilisateur à utiliser Docker

Par défaut, les commandes Docker nécessitent `sudo`. Ajoutez l’utilisateur voulu au groupe `docker`, puis ouvrez une nouvelle session pour que le changement soit pris en compte.

```bash
sudo usermod -aG docker docky
```

> L’appartenance au groupe `docker` donne des privilèges équivalents à `root` sur la machine. Ne l’accordez qu’à un compte de confiance.

Vérifiez ensuite l’installation :

```bash
docker version
docker run --rm hello-world
```

## 📦 Conteneurs et Compose

Les conteneurs sont les processus gérés par Docker. `docker compose` permet de démarrer une application composée de plusieurs services à partir d’un fichier `compose.yaml`.

```bash
# Afficher les conteneurs démarrés
docker ps

# Afficher aussi les conteneurs arrêtés
docker ps -a

# Démarrer les services du répertoire courant en arrière-plan
docker compose up -d

# Consulter les journaux des services
docker compose logs -f

# Arrêter et supprimer les conteneurs créés par Compose
docker compose down
```

## 🌐 Réseaux

Un réseau personnalisé permet aux conteneurs d’un même projet de se joindre par leur nom, sans exposer tous leurs ports vers l’hôte.

```bash
# Créer un réseau bridge nommé web
docker network create web

# Lister les réseaux disponibles
docker network ls

# Inspecter la configuration et les conteneurs connectés
docker network inspect web
```

## 🖼️ Images

Une image contient l’application et ses dépendances. Gardez des tags explicites afin de pouvoir revenir facilement à une version connue.

```bash
# Lister les images locales
docker image ls

# Inclure les images intermédiaires
docker image ls --all

# Rechercher une image précise
docker image ls moulti-stream

# Construire une image depuis le Dockerfile du répertoire courant
docker build -f Dockerfile -t moulti-stream:0.0.9 .
```

### 🔥 Nettoyer les images inutilisées

Cette commande supprime les images non utilisées par un conteneur. Vérifiez d’abord que vous n’avez pas besoin de les conserver pour un retour arrière.

```bash
docker image prune -a -f
```

## 💽 Volumes

Les volumes servent à conserver les données d’une application — par exemple une base de données — même après la suppression ou la recréation d’un conteneur.

```bash
# Lister tous les volumes
docker volume ls

# Afficher les volumes orphelins
docker volume ls -qf dangling=true

# Supprimer les volumes non utilisés
docker volume prune
```

## 💡 Aide rapide

Ajoutez `--help` à n’importe quelle commande pour afficher ses options, ou consultez l’état global du moteur avec :

```bash
docker system df
docker system info
```
