# Step CA
## Install 

```
cat <<EOT > /etc/yum.repos.d/smallstep.repo
[smallstep]
name=Smallstep
baseurl=https://packages.smallstep.com/stable/fedora/
enabled=1
repo_gpgcheck=0
gpgcheck=1
gpgkey=https://packages.smallstep.com/keys/smallstep-0x889B19391F774443.gpg
EOT
```

```
dnf install step-cli step-ca
```

```
useradd secu
```

## Autorité de certification
### Generation

```
su - secu
step ca init
```

```
✔ Deployment Type: Standalone
What would you like to name your new PKI?
✔ (e.g. Smallstep): CultureLinux
What DNS names or IP addresses will clients use to reach your CA?
✔ (e.g. ca.example.com[,10.1.2.3,etc.]): ca.lab.clinux.fr,192.168.1.211
✔ (e.g. :443 or 127.0.0.1:443): :9443
(e.g. :443 or 127.0.0.1:443): :9443
What would you like to name the CA's first provisioner?
✔ (e.g. you@smallstep.com): dev@clinux.fr
Choose a password for your CA keys and first provisioner.
✔ [leave empty and we'll generate one]:
✔ Password: OtvtYtRTHw3KqrKBeiRL6MsBdP4Q1mgf

Generating root certificate... done!
Generating intermediate certificate... done!

✔ Root certificate: /home/secu/.step/certs/root_ca.crt
✔ Root private key: /home/secu/.step/secrets/root_ca_key
✔ Root fingerprint: d4a68a26b88ba5cce0c3dfe441b31f4ce46ee4d13a1e316b5b6d412eb7406393
✔ Intermediate certificate: /home/secu/.step/certs/intermediate_ca.crt
✔ Intermediate private key: /home/secu/.step/secrets/intermediate_ca_key
✔ Database folder: /home/secu/.step/db
✔ Default configuration: /home/secu/.step/config/defaults.json
✔ Certificate Authority configuration: /home/secu/.step/config/ca.json
```


```
echo 8cASCwCpylp7B5vXN4srTjVUqnI4obht > ~/.step/password.txt
chmod 600 ~/.step/password.txt
chown secu:secu ~/.step/password.txt
```

```
~/.step
 ├── certs/
 ├── config/
 │    └── ca.json
 ├── db/
 └── password.txt
```

### Service

```
cat <<EOT > /etc/systemd/system/step-ca.service
[Unit]
Description=step-ca service
Documentation=https://smallstep.com/docs/step-ca
Documentation=https://smallstep.com/docs/step-ca/certificate-authority-server-production
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=30
StartLimitBurst=3
ConditionFileNotEmpty=/home/secu/.step/config/ca.json
ConditionFileNotEmpty=/home/secu/.step/password.txt

[Service]
Type=simple
User=secu
Group=secu
Environment=STEPPATH=/home/secu/.step
WorkingDirectory=/home/secu/.step
ExecStart=/usr/bin/step-ca config/ca.json --password-file password.txt
ExecReload=/bin/kill --signal HUP $MAINPID
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
StartLimitInterval=30
StartLimitBurst=3

; Process capabilities & privileges
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
SecureBits=keep-caps
NoNewPrivileges=yes

; Sandboxing
ProtectSystem=full
ProtectHome=no
RestrictNamespaces=true
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
PrivateTmp=true
PrivateDevices=true
ProtectClock=true
ProtectControlGroups=true
ProtectKernelTunables=true
ProtectKernelLogs=true
ProtectKernelModules=true
LockPersonality=true
RestrictSUIDSGID=true
RemoveIPC=true
RestrictRealtime=true
SystemCallFilter=@system-service
SystemCallArchitectures=native
MemoryDenyWriteExecute=true
ReadWriteDirectories=/home/secu/.step/db

[Install]
WantedBy=multi-user.target
EOT
```

```
systemctl daemon-reload
systemctl enable step-ca --now
systemctl status step-ca
journalctl -u step-ca -f
```


## Client
### Installation

```
cat <<EOT > /etc/yum.repos.d/smallstep.repo
[smallstep]
name=Smallstep
baseurl=https://packages.smallstep.com/stable/fedora/
enabled=1
repo_gpgcheck=0
gpgcheck=1
gpgkey=https://packages.smallstep.com/keys/smallstep-0x889B19391F774443.gpg
EOT
```

```
dnf install step-cli
```

### Lier au serveur
#### Récuperer l'empreinte du serveur 

```
step certificate fingerprint ~/.step/certs/root_ca.crt
cacc656a33ca59685026551ba00734b19fd695d966a2c7f9b464ee073ec359172
```

#### Créer un provisionneur

On va créer un mot de passe pour eviter d'utiliser celui du CA

```
step ca provisioner add fleet-provisioner \
  --type JWK \
  --create
```

#### Lier le client

```
step ca bootstrap --ca-url https://tower.lab.clinux.fr:9443 --fingerprint cacc656a33ca59685026551ba00734b19fd695d966a2c7f9b464ee073ec359172
```

#### Créer le fichier du mot de passe du provisioner

```
echo qdqsdqsdqs > ~/.step/provisioner_pass.txt
```

### Création d'un certificat

```
step ca certificate test.clinux.lan \
  test.clinux.lan.crt \
  test.clinux.lan.key \
  --provisioner fleet-provisioner \
  --provisioner-password-file ~/.step/provisioner_pass.txt
```

### Changement de la durée par défaut (sur le serveur)

```
vi ~/.step/config/ca.json
```

Dans la partie authority

```
                "claims": {
                   "minTLSCertDuration": "5m",
                    "maxTLSCertDuration": "2160h",
                   "defaultTLSCertDuration": "2160h"
                },
```

```
systemctl restart step-ca
```

### Création d'un certificat

```
step ca certificate web.lab.clinux.fr \
  web.lab.clinux.fr.crt \
  web.lab.clinux.fr.key \
  --provisioner fleet-provisioner \
  --provisioner-password-file ~/.step/provisioner_pass.txt
```

### Test du certificat

```
step certificate inspect web.lab.clinux.fr.crt --short
step certificate inspect web.lab.clinux.fr.crt
```

### Emplacement du certificat root du CA

```
cp ~/.step/certs/root_ca.crt /etc/nginx/ssl/
```

### Création d'un certificat client (p12)

```
step ca certificate \
  client1 \
  client1.crt \
  client1.key \
  --provisioner fleet-provisioner \
  --provisioner-password-file ~/.step/provisioner_pass.txt \
  --kty EC \
  --curve P-256 \
  --san client1.clinux.lan \
  --set extendedKeyUsage=clientAuth

step certificate p12 client1.p12 client1.crt client1.key
```

### Vérification du client

```
openssl pkcs12 -in client1.p12 -nokeys | openssl x509 -text -noout
```

### Renouvelement

#### Configuration (serveur)

```
"disableRenewal"=false
"allowRenewalAfterExpiry": false
```

#### Renouvellment de certificat

```
step ca renew web.lab.clinux.fr.crt web.lab.clinux.fr.key
step ca renew web.lab.clinux.fr.crt web.lab.clinux.fr.key --force
step ca renew web.lab.clinux.fr.crt web.lab.clinux.fr.key --generate-key
```



### Protection d'un vhost nginx

```
    ssl_certificate      /etc/nginx/ssl/web.lab.clinux.fr.crt;
    ssl_certificate_key  /etc/nginx/ssl/web.lab.clinux.fr.key;
```

### Forcer un certificat client

```
    ssl_client_certificate /etc/nginx/ssl/root_ca.crt;
    ssl_verify_client on;
```
