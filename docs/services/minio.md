# Minio
## Installation
### Binaires
#### serveur
```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio.RELEASE.2025-04-22T22-12-26Z
chmod +x minio.RELEASE.2025-04-22T22-12-26Z
mv minio.RELEASE.2025-04-22T22-12-26Z /usr/local/bin/minio
```
#### Client
```bash
wget https://dl.min.io/client/mc/release/linux-amd64/mc.RELEASE.2025-04-16T18-13-26Z
chmod +x mc.RELEASE.2025-04-16T18-13-26Z
mv mc.RELEASE.2025-04-16T18-13-26Z/usr/local/bin/mc
```

## Configuration serveur
### Création du user 
```bash
groupadd -r minio-user
useradd -M -r -g minio-user minio-user
chown minio-user:minio-user /mnt/data
```

### Création du disque (:warning: Il faut que le disque soit formaté en xfs)
```bash
mkdir /mnt/minio_data
mount /dev/sdb1 /mnt/minio_data
chown minio-user:minio-user /mnt/minio_data
```

### Création du service
```bash
sudo vi/etc/systemd/system/minio.service
```
```
[Unit]
Description=MinIO
Documentation=https://min.io/docs/minio/linux/index.html
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
WorkingDirectory=/usr/local

User=minio-user
Group=minio-user
ProtectProc=invisible

EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

# MinIO RELEASE.2023-05-04T21-44-30Z adds support for Type=notify (https://www.freedesktop.org/software/systemd/man/systemd.service.html#Type=)
# This may improve systemctl setups where other services use `After=minio.server`
# Uncomment the line to enable the functionality
# Type=notify

# Let systemd restart this service always
Restart=always

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536

# Specifies the maximum number of threads this process can create
TasksMax=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

### Parametrage par défaut (contient les identifiants pour se connecter à la console)
```bash
vi /etc/default/minio
```
```
# MINIO_ROOT_USER and MINIO_ROOT_PASSWORD sets the root account for the MinIO server.
# This user has unrestricted permissions to perform S3 and administrative API operations on any resource in the deployment.
# Omit to use the default values 'minioadmin:minioadmin'.
# MinIO recommends setting non-default values as a best practice, regardless of environment

MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=minio_password

# MINIO_VOLUMES sets the storage volume or path to use for the MinIO server.

MINIO_VOLUMES="/mnt/minio_data"

# MINIO_OPTS sets any additional commandline options to pass to the MinIO server.
# For example, `--console-address :9001` sets the MinIO Console listen port
MINIO_OPTS="--console-address :9001"
```

### Ouverture des ports 
```bash
firewall-cmd --add-port=9000/tcp --permanent && firewall-cmd --reload
firewall-cmd --add-port=9001/tcp --permanent && firewall-cmd --reload
```

## Lancement

### service 
```bash
systemctl daemon-reload
systemctl enable minio --now
```
### Vérification du service
```bash
netstat -tnlpv | grep 900
journalctl -u minio.service -f 
```

### Accès à la console d'administration

http://<IP_MINIO>:9001/login

## Client 

### Création des clés d'accès dans la console
http://<IP_MINIO>:9001/access-keys

### Configuration du client Minio 
```bash
  mc alias set MonMinio http://<IP_MINIO>:9000 ACCESS_KEY SECRET_KEY
```
Cela génère un fichier .mc/config.json dans le répertoire courant du user

### Statisques de l'instance
```
mc admin info MonMinio
```

### Création d'un bucket
```
mc mb MonMinio/monbucket
```

### Création d'un utilisateur
```
mc admin user add MonMinio utilisateur1 password1
```

### Lister les utilisateurs
```
mc admin user list MonMinio
```

### Création de jetons d'accès
```
mc admin accesskey create MonMinio utilisateur1 --name utilisateur1 | tee token-utilisateur1.json
```
```json
Access Key: GYASUUVIN21RVJ2Y0PLW
Secret Key: n169RKRIKJM8CTcyvAYwkO7z9OooJcBCWWWGBXh+
Expiration: NONE
Name: utilisateur1
Description: 
```

### Création d'un groupe
```
mc admin group add MonMinio monGroupe1 utilisateur1
```

### Lister les groupes
```
mc admin group list MonMinio
mc admin group info MonMinio monGroupe1
```

### Création d'une polique
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "MaPolitique1",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::monbucket/*"
      ]
    }
  ]
}
```
```
mc admin policy create MonMinio MaPolitique1 politique1.json  
```

### Lister les politiques 
```bash
mc admin policy ls MonMinio
mc admin policy info MonMinio MaPolitique1
```

### Attacher une politique
```bash
mc admin policy attach MonMinio MaPolitique1 --user utilisateur1
mc admin policy attach MonMinio MaPolitique1 --group monGroupe1
```

### Detacher une politique
```bash
mc admin policy detach MonMinio MaPolitique1 --user utilisateur1
mc admin policy detach MonMinio MaPolitique1 --group monGroupe1
```