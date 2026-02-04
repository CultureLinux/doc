# Let's encrypt

## Installation 🚀

Installation standard via le manager de paquet dnf.

```sh
dnf install epel-release
dnf install certbot
```
## Certbot 🔒

### Gestion du compte
#### Verifier l'etat

```sh
certbot show_account
```

#### Enregistrer un compte 

```sh
certbot register --agree-tos -m dev@clinux.fr
```

#### Supprimer un compte

```sh
certbot unregister
```

### Gestion des certificats

#### Lister les certificats 📚
Liste tous les certificats SSL gérés par Certbot.

```sh
certbot certificates 
```

### Ne pas se faire bannir

Il faut ajouter l'option `--dry-run` aux commandes de création des certificats pour tester la configuration.

### Créer un certificat manuellement 🔑 (challenge http port 80 ouvert)

```sh
certbot certonly --email dev@clinux.fr --webroot -w /var/lib/letsencrypt/ -d lab.online.clinux.fr --dry-run
```

### Créer un certificat avec un plugin 🔑 (challenge http port 80 ouvert)

```sh
dnf install -y certbot python3-certbot-nginx
certbot
```

### Créer un certificat wildcard 🔑 (challenge dns)

```sh
certbot certonly --manual --preferred-challenges dns -d lab.online.clinux.fr
```

### Renouveler 🔄
Renouvelle tous les certificats SSL gérés par Certbot.
```sh
certbot renew
```

### Révoquer 🧨
Revoquer un certificat
```sh
certbot revoke --cert-name lab.online.clinux.fr
```

### Supprimer 🚫
Supprime un certificat SSL spécifique avec Certbot.
```sh
certbot delete --cert-name lab.online.clinux.fr
```
