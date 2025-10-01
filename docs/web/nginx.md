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

# --- Fonction pour activer un site ---
nginxenable() {
    if [ -f "/etc/nginx/sites-available/$1" ]; then
        ln -s "/etc/nginx/sites-available/$1" "/etc/nginx/sites-enable/$1" && \
        echo "Site '$1' activé." && \
        nginx -t && \
        systemctl reload nginx
    else
        echo "Erreur : le fichier '$1' n'existe pas dans /etc/nginx/sites-available/"
    fi
}

# --- Fonction pour désactiver un site ---
nginxdisable() {
    if [ -L "/etc/nginx/sites-enable/$1" ]; then
        rm "/etc/nginx/sites-enable/$1" && \
        echo "Site '$1' désactivé." && \
        nginx -t && \
        systemctl reload nginx
    else
        echo "Erreur : le site '$1' n'est pas activé."
    fi
}

# --- Autocomplétion pour nginxenable ---
_nginxenable_autocomplete() {
    local cur
    cur="${COMP_WORDS[COMP_CWORD]}"
    COMPREPLY=( $(compgen -W "$(ls /etc/nginx/sites-available)" -- "$cur") )
}
complete -F _nginxenable_autocomplete nginxenable

# --- Autocomplétion pour nginxdisable ---
_nginxdisable_autocomplete() {
    local cur
    cur="${COMP_WORDS[COMP_CWORD]}"
    COMPREPLY=( $(compgen -W "$(ls /etc/nginx/sites-enable)" -- "$cur") )
}
complete -F _nginxdisable_autocomplete nginxdisable


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
    listen       443 ssl http2 default;
    listen       [::]:443 ssl http2 default;
    server_name  _;
    root         /usr/share/nginx/html;

    ssl_certificate "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3.pem";
    ssl_certificate_key "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3-key.pem";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;

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
    listen       80;
    listen       [::]:80;
    server_name  web.lab.clinux.fr;
    return 301 https://$host$request_uri;
}

server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
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

## Servir un port
### Exemple avec python

```
python3 -m http.server -b localhost 8080
```

### Vhost

```
server {
    listen       80 ;
    listen       [::]:80 ;
    server_name  serv-port.lab.clinux.fr;
    return 301 https://$host$request_uri;
}

server {
    listen       443 ssl http2 ;
    listen       [::]:443 ssl http2 ;
    server_name  serv-port.lab.clinux.fr;

    access_log /var/log/nginx/serv-port.lab.clinux.fr.access.log;
    error_log  /var/log/nginx/serv-port.lab.clinux.fr.error.log warn;

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
	    proxy_pass http://localhost:8080;
        try_files $uri $uri/ =404;
    }

}


```

### SElinux 

```
setsebool -P httpd_can_network_connect 1
```


### Répartiteur de charge (load blancer)

```
upstream backend {
    ip_hash;
    server localhost:8080;
    server localhost:8081;
}


server {
    listen       80 ;
    listen       [::]:80 ;
    server_name  serv-port.lab.clinux.fr;
    return 301 https://$host$request_uri;
}

server {
    listen       443 ssl http2 ;
    listen       [::]:443 ssl http2 ;
    server_name  serv-port.lab.clinux.fr;

    access_log /var/log/nginx/serv-port.lab.clinux.fr.access.log;
    error_log  /var/log/nginx/serv-port.lab.clinux.fr.error.log warn;

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
        proxy_pass http://backend;
        try_files $uri $uri/ =404;
    }

}


```

## Le cas php
### Utilisateur spécifique

```
adduser dynamic
su - dynamic
mkdir /home/dynamic/www/
echo "<?php echo 'Contenu dynamique : '. rand(); ?>" >> /home/dynamic/www/index.php
```


### Installation

```
dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
dnf install php83-php-fpm
```

### Configuration

```
cd /etc/opt/remi/php83/php-fpm.d/
vi php.lab.clinux.fr.conf
```

```
[php.lab.clinux.fr.conf]

; Chemin de l'utilisateur et du groupe
user = dynamic
group = dynamic

; Chemin où sera exécuté php-fpm
listen = /var/run/php8.3-fpm.php.lab.clinux.fr.conf.socket

; Autorisations du socket
listen.owner = nginx
listen.group = nginx
listen.mode = 0660

; Configuration du répertoire du site
chdir = /

; Configuration des processus (adaptée selon la charge attendue)
pm = dynamic
pm.max_children = 10
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3

; Logs
php_admin_value[error_log] = /var/log/php/php8.3-fpm-php.lab.clinux.fr.conf.log
php_admin_flag[log_errors] = on

; Paramètres PHP personnalisés
php_value[memory_limit] = 256M
php_value[upload_max_filesize] = 50M
php_value[post_max_size] = 50M
php_value[max_execution_time] = 60
php_value[date.timezone] = Europe/Paris

```

```
systemctl enable php83-php-fpm.service --now
ll /var/run/php8.3*
```

### Vhost nginx

```
server {
    listen       80 ;
    listen       [::]:80 ;
    server_name  php.lab.clinux.fr;
    return 301 https://$host$request_uri;
}

server {
    listen       443 ssl http2 ;
    listen       [::]:443 ssl http2 ;
    server_name  php.lab.clinux.fr;

    access_log /var/log/nginx/php.lab.clinux.fr.access.log;
    error_log  /var/log/nginx/php.lab.clinux.fr.error.log warn;

    ssl_certificate "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3.pem";
    ssl_certificate_key "/etc/nginx/ssl/_wildcard.lab.clinux.fr+3-key.pem";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;

    root /home/dynamic/www/;
    index index.php;

    location ~ \.php$ {
        include /etc/nginx/fastcgi.conf ;
        fastcgi_pass unix:/var/run/php8.3-fpm.php.lab.clinux.fr.conf.socket;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTPS $https;
        fastcgi_buffering off;
    }


}

```

### Durcissement 

```
php_admin_value[open_basedir] = /home/dynamic/www/:/tmp/
php_admin_value[disable_functions] = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source,dl,fsockopen,pfsockopen
```

## Restriction

### Basée sur GeoIp (maxmind)

#### Package (paywall)

* https://docs.nginx.com/nginx/admin-guide/dynamic-modules/geoip2/

#### Recompilation

* https://github.com/leev/ngx_http_geoip2_module

### Basée sur IP

```
allow 192.168.1.143;
allow 192.168.1.142;
deny all;

error_page 403 https://www.youtube.com/watch?v=dQw4w9WgXcQ&list=RDdQw4w9WgXcQ&start_radio=1 ;

```

### Basée sur pays (par ip)
#### Récupértion des CIDR 
```
mkdir /etc/nginx/nginx-country
wget http://firewalliplists.gypthecat.com/lists/nginx/nginx-countries.conf.zip
unzip nginx-countries.conf.zip -d /etc/nginx/nginx-country
```

#### Exemple de vhost

```
...
include nginx-country/FR-allow.conf;
include nginx-country/DE-deny.conf;
deny all;
...

```

## Sécurité (ModSecurity)

### Installation 

```
dnf install epel-release
dnf install nginx-mod-modsecurity
mkdir -p /etc/nginx/modsecurity /var/log/modsecurity
```

### Ajout des règles

```
cd /etc/nginx/modsecurity
curl -sL https://github.com/coreruleset/coreruleset/archive/v4.0.0.tar.gz | tar xz 
curl -sL https://github.com/coreruleset/coreruleset/archive/refs/tags/v4.18.0.tar.gz | tar xz 
ln -s coreruleset-4.18.0 coreruleset
cd coreruleset
cp crs-setup.conf.example crs-setup.conf
curl -so /etc/nginx/modsecurity/modsecurity.conf https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/modsecurity.conf-recommended 
```

### Passage en mode strict

```
sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/nginx/modsecurity/modsecurity.conf
```

### Création de la configuration modsecurity

```
vi /etc/nginx/modsecurity/main.conf
```

```
Include /etc/nginx/modsecurity/modsecurity.conf
Include /etc/nginx/modsecurity/coreruleset/crs-setup.conf
Include /etc/nginx/modsecurity/coreruleset/rules/*.conf
```

### Fix de configuration pour les règles (rocky linux)

```
sed -i 's/^\(SecUnicodeMapFile unicode.mapping 20127\)/# \1/' /etc/nginx/modsecurity/modsecurity.conf
```

### Injection de modsecurity dans nginx

```
vi /etc/nginx/nginx.conf
```

```
...
http {

    # Activer ModSecurity globalement
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsecurity/main.conf;
...
```

```
systemctl restart nginx
```

### Test de règles


```
<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>ModSecurity Test</title>
  <style>
    body { font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; padding: 24px; max-width:900px; }
    code { background:#f4f4f4; padding:2px 6px; border-radius:4px; }
    .links a { display:block; margin:8px 0; text-decoration:none; }
    .warn { color:#b22222; font-weight:700; }
  </style>
</head>
<body>
  <h1>ModSecurity WAF — Page de test</h1>
  <p>Page destinée à déclencher des règles ModSecurity / OWASP CRS en <strong>mode blocage</strong>.</p>

  <h2>Tests (clique pour exécuter)</h2>
  <div class="links">
    <p><span class="warn">XSS (payload encodé dans l'URL) — test basique :</span></p>
    <a href="/?test=%3Cscript%3Ealert('XSS')%3C%2Fscript%3E" rel="noopener noreferrer">/?test=%3Cscript%3Ealert('XSS')%3C%2Fscript%3E</a>

    <p><span class="warn">SQL Injection (simple payload encodé) :</span></p>
    <a href="/?id=1%27%20OR%20%271%27%3D%271" rel="noopener noreferrer">/?id=1%27%20OR%20%271%27%3D%271</a>

  </div>

  <h2>Commandes curl utiles</h2>
  <pre>
    <code># XSS (GET)
    </code>
  </pre>

  <p style="margin-top:20px;">Amuse-toi (ou pleure), et check les logs. 😈🔍</p>
</body>
</html>

```

