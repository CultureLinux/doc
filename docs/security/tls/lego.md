
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

./lego --email dev@clinux.fr --dns ovh --domains vhost.email.dom run
ls -l .lego/certificates/vhost.email.dom.{crt,key}
./lego --email dev@clinux.fr --dns ovh --domains *.email.dom run
ls -l .lego/certificates/_.email.dom.{crt,key}
```

### Ionos 🖥️
Montre comment configurer et utiliser Lego avec le fournisseur DNS Ionos.
```sh
IONOS_API_KEY=xxxxxxxxxxxxxxxxxxxxxxx
./lego --email dev@clinux.fr --dns ionos --domains vhost.email.dom run
```
