# Nginx
## Installation
    dnf install nginx
    systemctl enable nginx --now
    firewall-cmd --add-port=80/tcp --permanent && firewall-cmd --reload;

## Outillage
```
vi /root/.bashrc
```
```
...
    alias nginxtest='nginx -t'
    alias nginxstatus='service nginx status'
    alias nginxrestart='nginx -t && service nginx restart'
    alias nginxreload='nginx -t && service nginx reload'
```

## Test 
    netstat -tnlpv | grep 80
    curl -s http://localhost | grep '<h1>' #sur le serveur web
    curl -s http://$IP | grep '<h1>' #de l'exterieur

## Ajout du TLS
### Installation de mkcert
    dnf install curl nss-tools wget
    curl -s https://api.github.com/repos/FiloSottile/mkcert/releases/latest | grep browser_download_url | grep '\linux-amd64' | cut -d '"' -f 4 | wget -i -
    mv mkcert-v*-linux-amd64 /usr/bin/mkcert
    chmod 750 /usr/bin/mkcert
    mkcert -install
### Positionnement dans nginx 
    mkdir /etc/nginx/ssl
    cd /etc/nginx/ssl
### Generation de certificats 
    mkcert web.lab.clinux.fr
    #Generate "web.lab.clinux.fr.pem" and "web.lab.clinux.fr-key.pem".
	mkcert web.lab.clinux.fr 192.168.1.211 127.0.0.1 ::1
	#Generate "web.lab.clinux.fr+3.pem" and "web.lab.clinux.fr+3-key.pem".
	mkcert "*.lab.clinux.fr" 192.168.1.211 127.0.0.1 ::1
	#Generate "_wildcard.lab.clinux.fr+3.pem" and "_wildcard.lab.clinux.fr+3-key.pem".
### Modification de la configuration 
```
    vi /etc/nginx/nginx.conf
```
```
    server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3.pem";
        ssl_certificate_key "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3-key.pem";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;

        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```
### Ouverture du port
    firewall-cmd --add-port=443/tcp --permanent && firewall-cmd --reload;
### Redirection automatique vers https (a rajouter dans le bloc 80 - http)
    return 301 https://$host$request_uri;

### Ajout de l'autorié de certificats dans le navigateur 

Récuperer le fichier `/root/.local/share/mkcert/rootCA.pem` et charger le en tant que certificat d'autorité

## Gestion des virtual hosts (vhost)
### Création de repertoires 
```
mkdir /etc/nginx/sites-available
mkdir /etc/nginx/sites-enable
```
### Ajout de l'include (dans le bloc http)
```
vi /etc/nginx/nginx.conf
```
```
...
include /etc/nginx/sites-enable/*.conf;
...
```
### Vhost par defaut
```
vi /etc/nginx/sites-available/web.lab.clinux.fr.conf
```
```
server {
    listen       80 default;
    listen       [::]:80 default;
    server_name  web.lab.clinux.fr;
    return 301 https://$host$request_uri;
}

server {
    listen       443 ssl http2 default;
    listen       [::]:443 ssl http2 default;
    server_name  web.lab.clinux.fr;
    root         /usr/share/nginx/html;

    access_log /var/log/nginx/web.lab.clinux.fr.access.log;
    error_log  /var/log/nginx/web.lab.clinux.fr.log warn;

    ssl_certificate "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3.pem";
    ssl_certificate_key "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3-key.pem";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;

    error_page 404 /custom_errors/404.html;
    error_page 500 502 503 504 /custom_errors/50x.html;

    location /custom_errors/ {
        alias /usr/share/nginx/html/;
        internal;
    }

    location / {
        try_files $uri $uri/ =404;
    }

}
```

### Activation du vhost
```
    ln -s /etc/nginx/sites-available/web.lab.clinux.fr.conf /etc/nginx/sites-enable/web.lab.clinux.fr.conf
    nginxreload
```

### Verification des logs 
```
tail -f /var/log/nginx/web.lab.clinux.fr*.log
```

## Hebergement par utilisateur
### Autoriser nginx a rentrer dans la /home
    usermod -aG static nginx
    chmod 750 /home/static
### Verifier le log selinux 
    tail -f /var/log/audit/audit.log
    audit2why -all
### SElinux pour utilisateurs 
    setsebool -P httpd_enable_homedirs 1
###  Vhost 
```
vi /etc/nginx/sites-available/web-static.lab.clinux.fr.conf
```
```
server {
    listen       80 ;
    listen       [::]:80 ;
    server_name  web-static.lab.clinux.fr;
    return 301 https://$host$request_uri;
}

server {
    listen       443 ssl http2 ;
    listen       [::]:443 ssl http2 ;
    server_name  web-static.lab.clinux.fr;

    access_log /var/log/nginx/web-static.lab.clinux.fr.access.log;
    error_log  /var/log/nginx/web-static.lab.clinux.fr.error.log warn;

    ssl_certificate "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3.pem";
    ssl_certificate_key "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3-key.pem";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;


    root /home/static/html/;
    index index.html;

    error_page 404 /custom_errors/404.html;
    error_page 500 502 503 504 /custom_errors/50x.html;

    location /custom_errors/ {
        alias /usr/share/nginx/html/;
        internal;
    }

    location / {
        try_files $uri $uri/ =404;
    }

}
```
###  Activation 
```
ln -s /etc/nginx/sites-available/web-static.lab.clinux.fr.conf /etc/nginx/sites-enable/web-static.lab.clinux.fr.conf
nginx -t && service nginx reload
```

