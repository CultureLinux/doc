# Génération🚀
Cette section offre des commandes pour générer et gérer des certificats SSL à l'aide de Certbot et Lego.

## Certbot🔒
Cette sous-section explique comment utiliser Certbot pour la génération, le renouvellement et la suppression de certificats.

### Nouveau 🔑
Génère un nouveau certificat SSL avec Certbot en testant d'abord avec une simulation, puis effectue la génération réelle.
```sh
certbot certonly --agree-tos --email your@email.dom --webroot -w /var/lib/letsencrypt/ -d vhost.email.dom --dry-run
certbot certonly --agree-tos --email your@email.dom --webroot -w /var/lib/letsencrypt/ -d vhost.email.dom
```

### Lister 📚
Liste tous les certificats SSL gérés par Certbot.
```sh
certbot certificates
```

### Renouveler 🔄
Renouvelle tous les certificats SSL gérés par Certbot.
```sh
certbot renew
```

### Supprimer 🚫
Supprime un certificat SSL spécifique avec Certbot.
```sh
certbot delete --cert-name vhost.email.dom
```

## Lego🌐
Cette sous-section explique comment utiliser Lego pour la génération et la gestion de certificats avec différents fournisseurs DNS.

### install📦
Installe Lego en téléchargeant la version la plus récente, l'extraittez et exécutez-le.
```sh
wget https://github.com/go-acme/lego/releases/download/v4.16.1/lego_v4.16.1_linux_amd64.tar.gz
tar xvzf lego_v4.16.1_linux_amd64.tar.gz
./lego
```

### providers📖
Fournit un lien vers la documentation pour configurer les fournisseurs DNS avec Lego.
```sh
https://go-acme.github.io/lego/dns/
```

### OVH 🖥️
Montre comment configurer et utiliser Lego avec le fournisseur DNS OVH.
```sh
export OVH_APPLICATION_KEY=xxxxxxxxxxx
export OVH_APPLICATION_SECRET=xxxxxxxxxxxxxxxxxxxxxxx 
export OVH_CONSUMER_KEY=xxxxxxxxxxxxxx
export OVH_ENDPOINT=ovh-eu

./lego --email your@email.dom --dns ovh --domains vhost.email.dom run
ls -l .lego/certificates/vhost.email.dom.{crt,key}
./lego --email your@email.dom --dns ovh --domains *.email.dom run
ls -l .lego/certificates/_.email.dom.{crt,key}
```

### Ionos 🖥️
Montre comment configurer et utiliser Lego avec le fournisseur DNS Ionos.
```sh
IONOS_API_KEY=xxxxxxxxxxxxxxxxxxxxxxx
./lego --email your@email.dom --dns ionos --domains vhost.email.dom run
```
