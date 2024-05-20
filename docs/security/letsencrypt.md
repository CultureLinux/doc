# Generate
This section provides commands for generating and managing SSL certificates using Certbot and Lego.

## certbot
This subsection explains how to use Certbot for certificate generation, renewal, and deletion.

### generate
Generates a new SSL certificate using Certbot with a dry run to test, followed by the actual generation.
```sh
certbot certonly --agree-tos --email your@email.dom --webroot -w /var/lib/letsencrypt/ -d vhost.email.dom --dry-run
certbot certonly --agree-tos --email your@email.dom --webroot -w /var/lib/letsencrypt/ -d vhost.email.dom
```

### renew
Renews all the SSL certificates managed by Certbot.
```sh
certbot renew
```

### delete
Deletes a specific SSL certificate using Certbot.
```sh
certbot delete --cert-name vhost.email.dom
```

## lego
This subsection explains how to use Lego for certificate generation and management with various DNS providers.

### install
Installs Lego by downloading the latest release and extracting it.
```sh
wget https://github.com/go-acme/lego/releases/download/v4.16.1/lego_v4.16.1_linux_amd64.tar.gz
tar xvzf lego_v4.16.1_linux_amd64.tar.gz
./lego
```

### providers
Provides a link to the documentation for configuring DNS providers with Lego.
```sh
https://go-acme.github.io/lego/dns/
```

### OVH
Shows how to configure and use Lego with the OVH DNS provider.
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
