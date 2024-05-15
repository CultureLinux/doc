# Generate
## certbot
### generate
    certbot certonly --agree-tos --email your@email.dom --webroot -w /var/lib/letsencrypt/ -d vhost.email.dom --dry-run
    certbot certonly --agree-tos --email your@email.dom --webroot -w /var/lib/letsencrypt/ -d vhost.email.dom
### renew 
    certbot renew
### delete
    certbot delete --cert-name vhost.email.dom
## lego
### install
    wget https://github.com/go-acme/lego/releases/download/v4.16.1/lego_v4.16.1_linux_amd64.tar.gz
    tar xvzf lego_v4.16.1_linux_amd64.tar.gz
    ./lego
### providers
    https://go-acme.github.io/lego/dns/
### OVH
    export OVH_APPLICATION_KEY=xxxxxxxxxxx
    export OVH_APPLICATION_SECRET=xxxxxxxxxxxxxxxxxxxxxxx 
    export OVH_CONSUMER_KEY=xxxxxxxxxxxxxx
    export OVH_ENDPOINT=ovh-eu

    ./lego --email your@email.dom --dns ovh --domains vhost.email.dom run
    ls -l .lego/certificates/vhost.email.dom.{crt,key}
    ./lego --email your@email.dom --dns ovh --domains *.email.dom run
    ls -l .lego/certificates/_.email.dom.{crt,key}