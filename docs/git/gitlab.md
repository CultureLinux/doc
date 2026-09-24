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

## Update
### Rocky
    dnf update -x gitlab-ce
    dnf update gitlab-ce-18.2.8-ce.0.el9
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
### Manuel 

```
    gitlab-backup create
    tar cvzf gitlab-conf.tar.gz /etc/gitlab/*
    scp {/var/opt/gitlab/backups/*,gitlab-conf.tar.gz}  backup@backup_server:/path/
```
### Service 

```
vi /etc/systemd/system/gitlab-backup.service
```

```
[Unit]
Description=GitLab backup and remote synchronization
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/gitlab-backup.sh
TimeoutStartSec=infinity
Nice=10
IOSchedulingClass=best-effort
IOSchedulingPriority=7
```

### Timer 

```
vi /etc/systemd/system/gitlab-backup.timer
```

```
[Unit]
Description=Daily GitLab backup

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### Script

```
vi /usr/local/sbin/gitlab-backup.sh
```

```
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/var/opt/gitlab/backups"
DATE="$(date +%Y%m%d-%H%M%S)"

# Destination distante
REMOTE_USER="root"
REMOTE_HOST="192.168.1.33"
REMOTE_DIR="/mnt/nas/backup/gitlab"

# Rétention
LOCAL_KEEP=5
REMOTE_RETENTION_DAYS=30

echo "============================================================"
echo " GitLab backup - $(date)"
echo "============================================================"

#
# 1. Backup GitLab
#

echo
echo "[1/5] Backup GitLab data..."

/opt/gitlab/bin/gitlab-backup create

#
# 2. Backup configuration + secrets
#

echo
echo "[2/5] Backup configuration + secrets..."

/opt/gitlab/bin/gitlab-ctl backup-etc \
    --backup-path "$BACKUP_DIR"

# Récupère le dernier backup de configuration créé
CONFIG_BACKUP="$(
    find "$BACKUP_DIR" \
        -maxdepth 1 \
        -type f \
        -name '*gitlab_config*.tar' \
        -printf '%T@ %p\n' |
    sort -nr |
    head -1 |
    cut -d' ' -f2-
)"

if [[ -z "${CONFIG_BACKUP:-}" || ! -f "$CONFIG_BACKUP" ]]; then
    echo "ERREUR: backup de configuration GitLab introuvable"
    exit 1
fi

CONFIG_DEST="${BACKUP_DIR}/gitlab_config_${DATE}.tar"

mv "$CONFIG_BACKUP" "$CONFIG_DEST"

echo "Configuration sauvegardée : $CONFIG_DEST"

#
# 3. Synchronisation vers le NAS
#

echo
echo "[3/5] Synchronisation distante..."

ssh "${REMOTE_USER}@${REMOTE_HOST}" \
    "mkdir -p '${REMOTE_DIR}'"

# IMPORTANT :
# pas de --delete
# Les anciens backups distants restent donc présents.
rsync -avh --partial \
    "${BACKUP_DIR}/" \
    "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"

echo "Synchronisation distante terminée."

#
# 4. Rétention locale
#

echo
echo "[4/5] Rétention locale : conservation des ${LOCAL_KEEP} derniers backups..."

#
# Backup GitLab data
#

mapfile -t DATA_BACKUPS < <(
    find "$BACKUP_DIR" \
        -maxdepth 1 \
        -type f \
        -name '*_gitlab_backup.tar' \
        -printf '%T@ %p\n' |
    sort -nr |
    cut -d' ' -f2-
)

if (( ${#DATA_BACKUPS[@]} > LOCAL_KEEP )); then

    for FILE in "${DATA_BACKUPS[@]:LOCAL_KEEP}"; do
        echo "Suppression locale : $FILE"
        rm -f -- "$FILE"
    done

fi

#
# Backup configuration
#

mapfile -t CONFIG_BACKUPS < <(
    find "$BACKUP_DIR" \
        -maxdepth 1 \
        -type f \
        -name 'gitlab_config_*.tar' \
        -printf '%T@ %p\n' |
    sort -nr |
    cut -d' ' -f2-
)

if (( ${#CONFIG_BACKUPS[@]} > LOCAL_KEEP )); then

    for FILE in "${CONFIG_BACKUPS[@]:LOCAL_KEEP}"; do
        echo "Suppression locale : $FILE"
        rm -f -- "$FILE"
    done

fi

#
# 5. Rétention distante
#

echo
echo "[5/5] Rétention distante : ${REMOTE_RETENTION_DAYS} jours..."

ssh "${REMOTE_USER}@${REMOTE_HOST}" "
    find '${REMOTE_DIR}' \
        -maxdepth 1 \
        -type f \
        \( -name '*_gitlab_backup.tar' -o -name 'gitlab_config_*.tar' \) \
        -mtime +${REMOTE_RETENTION_DAYS} \
        -print \
        -delete
"

echo
echo "Backups locaux présents :"
ls -lh "$BACKUP_DIR"

echo
echo "============================================================"
echo " GitLab backup terminé avec succès - $(date)"
echo "============================================================"
```

### Application 

```
chmod 750 /usr/local/sbin/gitlab-backup.sh

systemctl daemon-reload
systemctl enable --now gitlab-backup.timer
systemctl list-timers gitlab-backup.timer
```


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

## Small config 

``` 
vi /etc/gitlab/gitlab.rb
``` 
``` 
###############################################################################
# GitLab Homelab
# 3 vCPU / 2.5 Go RAM
###############################################################################

external_url 'https://gitlab.local.clinux.fr'

###############################################################################
# NGINX
###############################################################################

nginx['enable'] = true
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/_.local.clinux.fr.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/_.local.clinux.fr.key"

###############################################################################
# PUMA
# Mode single process pour limiter fortement la consommation mémoire.
###############################################################################

puma['worker_processes'] = 0
puma['min_threads'] = 1
puma['max_threads'] = 4

###############################################################################
# SIDEKIQ
# Peu de jobs simultanés : suffisant pour un GitLab homelab.
###############################################################################

sidekiq['concurrency'] = 5

###############################################################################
# POSTGRESQL
###############################################################################

postgresql['shared_buffers'] = "128MB"
postgresql['work_mem'] = "8MB"
postgresql['maintenance_work_mem'] = "64MB"
postgresql['effective_cache_size'] = "1GB"
postgresql['max_worker_processes'] = 4

###############################################################################
# GITALY
# Limitation de la concurrence des opérations Git lourdes.
###############################################################################

gitaly['configuration'] = {
  concurrency: [
    {
      'rpc' => '/gitaly.SmartHTTPService/PostReceivePack',
      'max_per_repo' => 2
    },
    {
      'rpc' => '/gitaly.SSHService/SSHReceivePack',
      'max_per_repo' => 2
    }
  ]
}

###############################################################################
# MONITORING
###############################################################################

alertmanager['enable'] = false
gitlab_exporter['enable'] = false
node_exporter['enable'] = false
postgres_exporter['enable'] = false
prometheus['enable'] = false
redis_exporter['enable'] = false

###############################################################################
# SERVICES NON UTILISES
###############################################################################

# Kubernetes Agent Server
gitlab_kas['enable'] = false

# Mattermost
mattermost['enable'] = false

# GitLab Pages
gitlab_pages['enable'] = false
pages_nginx['enable'] = false
```

## CI-CD
### Install runner 

```
curl -L --output /usr/local/bin/gitlab-runner "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/binaries/gitlab-runner-linux-amd64"
chmod +x /usr/local/bin/gitlab-runner
useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
gitlab-runner start
```

### Register (v17.x)
#### webui

* https://git.local.clinux.fr/admin/runners/new    
* Ajout d'un tag (my-cd-tag)

#### runner 
    

## CLI glab
### install 

```
wget https://gitlab.com/gitlab-org/cli/-/releases/v1.45.0/downloads/glab_1.45.0_Linux_x86_64.tar.gz
tar xvzf glab_*_Linux_x86_64.tar.gz
mv bin/glab /usr/local/bin/
```

### Login 
```
glab auth login --hostname gitlab.fqdn --stdin < ~/.gitlab.token
```

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
### Chercher les clés ssh
#### Connexion à la base
    docker exec -it sameersbn-gitlab-postgresql-1 bash
#### Recherche dans les utilisateurs
    SELECT k.id AS key_id,
        k.title,
        k.fingerprint,
        u.id AS user_id,
        u.username,
        u.email,
        k.created_at
    FROM keys k
    JOIN users u ON k.user_id = u.id ;
#### Recherche dans les projets
    SELECT k.id AS key_id,
        k.type,
        k.title,
        k.fingerprint,
        p.id AS project_id,
        n.path || '/' || p.path AS full_path,
        dkp.can_push,
        dkp.created_at
    FROM keys k
    JOIN deploy_keys_projects dkp ON k.id = dkp.deploy_key_id
    JOIN projects p ON dkp.project_id = p.id
    JOIN namespaces n ON p.namespace_id = n.id;

## Migration sameersbn > rocky9 rpm
### Deplacement des clés 
#### sameer

```
cd /home/docky/DockerVault/gitlab/data-gitlab/ssh/

scp \
  ssh_host_ecdsa_key ssh_host_ecdsa_key.pub \
  ssh_host_ed25519_key ssh_host_ed25519_key.pub \
  ssh_host_rsa_key ssh_host_rsa_key.pub \
  root@192.168.1.180:/etc/ssh/
```

#### rocky9

```
chown root:ssh_keys /etc/ssh/ssh_host_{ecdsa,ed25519,rsa}_key
chmod 640 /etc/ssh/ssh_host_{ecdsa,ed25519,rsa}_key

chown root:root /etc/ssh/ssh_host_{ecdsa,ed25519,rsa}_key.pub
chmod 644 /etc/ssh/ssh_host_{ecdsa,ed25519,rsa}_key.pub

restorecon -v /etc/ssh/ssh_host_{ecdsa,ed25519,rsa}_key*

systemctl restart sshd
systemctl status sshd --no-pager
```

