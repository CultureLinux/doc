# Opendkim

## Installation 

```
dnf config-manager --set-enabled crb
dnf install opendkim opendkim-tools -y
```

## Création de la structure 

```
mkdir -p /etc/opendkim/keys/domain.fr
chown -R opendkim:opendkim /etc/opendkim
chmod 700 /etc/opendkim/keys/domain.fr
```

## Génaration des clés et du selecteur

Le selecteur est `prod` pour le domaine `domain.fr`
```
cd /etc/opendkim/keys/domain.fr
opendkim-genkey -b 2048 -d domain.fr -s prod
chown opendkim:opendkim prod.private
chmod 600 prod.private
```

## Configuration OpenDKIM
### DKIM

```
vi /etc/opendkim.conf
```

```
Syslog                  yes
SyslogSuccess           yes
LogWhy                  yes

Mode                    sv
Canonicalization        relaxed/relaxed

Socket                  inet:8891@localhost

UserID                  opendkim:opendkim

KeyTable                /etc/opendkim/KeyTable
SigningTable            refile:/etc/opendkim/SigningTable

InternalHosts           refile:/etc/opendkim/TrustedHosts
ExternalIgnoreList      refile:/etc/opendkim/TrustedHosts
```

### Keytable

```
vi /etc/opendkim/KeyTable
```

```
prod._domainkey.domain.fr domain.fr:prod:/etc/opendkim/keys/domain.fr/prod.private
```

### Signing table

```
vi /etc/opendkim/SigningTable
```

```
*@domain.fr prod._domainkey.domain.fr
```

### Trusted host

```
vi /etc/opendkim/TrustedHosts
```

```
127.0.0.1
localhost
192.168.0.0/16
10.0.0.0/8
```

## Configuration Postfix
### Main

```
vi /etc/postfix/main.cf
```

```
milter_protocol = 6
milter_default_action = accept

smtpd_milters = inet:localhost:8891
non_smtpd_milters = inet:localhost:8891
```

### Master

```
vi /etc/postfix/master.cf
```

```
submission inet n       -       n       -       -       smtpd
  -o smtpd_milters=inet:localhost:8891
  -o non_smtpd_milters=inet:localhost:8891

smtps inet n       -       n       -       -       smtpd
  -o smtpd_milters=inet:localhost:8891
  -o non_smtpd_milters=inet:localhost:8891
```

## Application

```
systemctl enable opendkim --now
service postfix restart
```


## DNS 

Dès que la signature est vu dans le log postfix, il faut deployer la clé publique dans le dns

```
cat /etc/opendkim/keys/booa.fr/pfix-preprod.txt
```

```
prod._domainkey IN      TXT     ( "v=DKIM1; k=rsa; " "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMId" "tIRmnY21" )  ; ----- DKIM key prod for domain.fr
```