# Step CA
## Install 
### Debian
    wget https://dl.smallstep.com/certificates/docs-ca-install/latest/step-ca_amd64.deb
    dpkg -i step-ca_amd64.deb
### RHEL
    wget https://dl.smallstep.com/certificates/docs-ca-install/latest/step-ca_amd64.rpm
    sudo rpm -i step-ca_amd64.rpm
## Authority 
### Generate 
### Service
```
vi /etc/systemd/system/step-ca.service
```
```
[Unit]
Description=step-ca service
Documentation=https://smallstep.com/docs/step-ca
Documentation=https://smallstep.com/docs/step-ca/certificate-authority-server-production
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=30
StartLimitBurst=3
ConditionFileNotEmpty=/etc/step-ca/config/ca.json
ConditionFileNotEmpty=/etc/step-ca/password.txt

[Service]
Type=simple
User=secu
Group=secu
Environment=STEPPATH=/etc/step-ca
WorkingDirectory=/etc/step-ca
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
ProtectHome=true
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
ReadWriteDirectories=/etc/step-ca/db

[Install]
WantedBy=multi-user.target
```


## Client
### Install
#### Debian
    wget https://dl.smallstep.com/cli/docs-ca-install/latest/step-cli_amd64.deb
    dpkg -i step-cli_amd64.deb
#### RHEL
    wget https://dl.smallstep.com/cli/docs-ca-install/latest/step-cli_amd64.rpm
    sudo rpm -i step-cli_amd64.rpm
### Link to CA
#### Get fingerprint (on server) 
    step certificate fingerprint certs/root_ca.crt
    cacc656a33ca59685026551ba00734b19fd695d966a2c7f9b464ee073ec359172
#### Link client
    step ca bootstrap --ca-url https://your.ca.server:4443 --fingerprint cacc656a33ca59685026551ba00734b19fd695d966a2c7f9b464ee073ec359172
### Generate certificat
    step ca certificate test.clinux.lan test.clinux.lan.crt test.clinux.lan.key
    step ca certificate test.clinux.lan test.clinux.lan.crt test.clinux.lan.key --provisioner-password-file=PASSWORD_FILE -f --not-after=10h

### Test certificats
    step certificate inspect test.clinux.lan 
    step certificate inspect test.clinux.lan --short