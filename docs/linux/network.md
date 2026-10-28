# Réseau 🌐

Cette section couvre le diagnostic et la configuration réseau sur Rocky Linux 9/10 et Debian. Avant toute modification distante, prévoyez un accès à la console : réactiver une interface peut couper la session SSH. ⚠️

## Diagnostic 🔍

Cette sous-section permet de vérifier les interfaces, adresses, routes, DNS et ports à l'écoute.

### Lister les interfaces 📋
Affiche les interfaces réseau et leurs adresses IP.
```sh
$ ip link show
$ ip address show
```

### Voir la route par défaut 🧭
Affiche la passerelle utilisée pour joindre les réseaux extérieurs.
```sh
$ ip route show default
```

### Vérifier la résolution DNS 🧪
Interroge directement le résolveur DNS configuré.
```sh
$ resolvectl status
$ dig example.com
```

`resolvectl` nécessite `systemd-resolved`. S'il n'est pas utilisé, consultez `/etc/resolv.conf`.

### Voir les ports à l'écoute 👂
Affiche les sockets TCP et UDP en écoute, avec le processus associé lorsque les droits le permettent.
```sh
$ ss -ltnup
```

`netstat -tnlpv` fournit une information similaire, mais nécessite généralement le paquet `net-tools`.

### Tester la connectivité 📡
Teste successivement la pile IP, la passerelle, une adresse Internet et la résolution DNS.
```sh
$ ping -c 4 127.0.0.1
$ ping -c 4 <passerelle>
$ ping -c 4 1.1.1.1
$ ping -c 4 example.com
```

## Rocky Linux 9 et 10 🪨

Rocky Linux 9 et 10 utilisent NetworkManager. Sur Rocky 10, les anciens scripts `ifcfg-*` et les commandes `ifup`/`ifdown` ne sont plus disponibles ; utilisez `nmcli`, `nmtui` ou `nmstate`.

### Identifier la connexion 👀
Le nom du profil NetworkManager et le nom de l'interface peuvent être différents. Relevez les deux avant de modifier la configuration.
```sh
$ nmcli device status
$ nmcli connection show --active
$ nmcli device show enp1s0
```

Dans les exemples suivants, remplacez `connexion` par le nom du profil et `enp1s0` par votre interface.

### Configurer DHCP 🔄
Configure le profil pour recevoir automatiquement son adresse, sa passerelle et ses DNS via DHCP.
```sh
# nmcli connection modify connexion ipv4.method auto ipv4.addresses '' ipv4.gateway ''
# nmcli connection up connexion
```

### Configurer une adresse IPv4 statique 📌
Définit une adresse, une passerelle et des serveurs DNS statiques, puis réactive le profil.
```sh
# nmcli connection modify connexion ipv4.method manual \
    ipv4.addresses '192.0.2.10/24' \
    ipv4.gateway '192.0.2.1' \
    ipv4.dns '192.0.2.53 1.1.1.1'
# nmcli connection up connexion
```

### Ajouter une route statique 🛣️
Ajoute une route persistante vers le réseau `198.51.100.0/24` via la passerelle indiquée.
```sh
# nmcli connection modify connexion +ipv4.routes '198.51.100.0/24 192.0.2.254'
# nmcli connection up connexion
```

### Configurer IPv6 🧭
Active l'autoconfiguration IPv6, ou la désactive explicitement si elle n'est pas utilisée.
```sh
# nmcli connection modify connexion ipv6.method auto
# nmcli connection modify connexion ipv6.method disabled
# nmcli connection up connexion
```

### Consulter les fichiers de profil 📄
NetworkManager stocke les profils au format keyfile dans `/etc/NetworkManager/system-connections/`. Préférez `nmcli` ou `nmtui` pour les modifier : ils préservent une configuration cohérente.
```sh
# ls -l /etc/NetworkManager/system-connections/
$ nmcli connection show connexion
```

### Vérifier la configuration ✅
```sh
$ ip address show enp1s0
$ ip route show
$ nmcli device show enp1s0
```

## Debian 🌀

Debian peut employer NetworkManager, `systemd-networkd` ou `ifupdown`. Une même interface ne doit être administrée que par un seul de ces outils. Vérifiez le gestionnaire actif avant de créer ou modifier une configuration.

### Identifier le gestionnaire ⚙️
```sh
$ systemctl is-active NetworkManager
$ systemctl is-active systemd-networkd
$ test -f /etc/network/interfaces && sed -n '1,160p' /etc/network/interfaces
```

### NetworkManager 🖥️
NetworkManager est courant sur les installations de bureau. La syntaxe `nmcli` est la même que sur Rocky Linux.

#### Installer et activer 📦
```sh
# apt install network-manager
# systemctl enable --now NetworkManager
```

#### Configurer une adresse statique 📌
Crée un profil Ethernet nommé `lan-statique` pour `enp1s0`.
```sh
# nmcli connection add type ethernet ifname enp1s0 con-name lan-statique \
    ipv4.method manual ipv4.addresses '192.0.2.10/24' \
    ipv4.gateway '192.0.2.1' ipv4.dns '192.0.2.53 1.1.1.1'
# nmcli connection up lan-statique
```

### systemd-networkd 🧩
`systemd-networkd` convient bien aux serveurs sans interface graphique. Ses fichiers se trouvent dans `/etc/systemd/network/`.

#### Configurer DHCP 🔄
Créez `/etc/systemd/network/10-enp1s0.network` avec le contenu suivant :
```ini
[Match]
Name=enp1s0

[Network]
DHCP=yes
```

#### Configurer une adresse statique 📌
Pour une adresse statique, utilisez plutôt le contenu suivant dans ce même fichier :
```ini
[Match]
Name=enp1s0

[Network]
Address=192.0.2.10/24
Gateway=192.0.2.1
DNS=192.0.2.53
DNS=1.1.1.1
```

#### Activer la configuration ▶️
Active `systemd-networkd`. Pour la résolution DNS gérée par systemd, activez également `systemd-resolved`.
```sh
# systemctl enable --now systemd-networkd
# systemctl enable --now systemd-resolved
# networkctl status enp1s0
```

### ifupdown (`/etc/network/interfaces`) 📝
`ifupdown` reste disponible pour les systèmes qui l'utilisent déjà. Évitez de le mélanger avec NetworkManager ou `systemd-networkd` sur la même interface.

#### Configurer DHCP 🔄
Ajoutez ou adaptez ce bloc dans `/etc/network/interfaces` :
```ini
auto enp1s0
iface enp1s0 inet dhcp
```

#### Configurer une adresse statique 📌
```ini
auto enp1s0
iface enp1s0 inet static
    address 192.0.2.10/24
    gateway 192.0.2.1
    dns-nameservers 192.0.2.53 1.1.1.1
    dns-search example.internal
```

La directive `dns-nameservers` est fournie par le paquet `resolvconf` lorsque celui-ci est utilisé.

#### Appliquer la configuration ▶️
Réactive uniquement l'interface concernée.
```sh
# ifdown enp1s0 && ifup enp1s0
```

## Dépannage 🧰

### Vérifier le lien physique 🔌
Affiche l'état du lien, sa vitesse et son duplex.
```sh
$ ethtool enp1s0
```

### Voir les journaux 📜
Consulte les messages récents du gestionnaire réseau actif.
```sh
$ journalctl -u NetworkManager -b
$ journalctl -u systemd-networkd -b
```

### Renouveler une configuration DHCP 🔁
Avec NetworkManager, réactivez le profil. Avec `systemd-networkd`, redémarrez uniquement le service après avoir vérifié la configuration.
```sh
# nmcli connection up connexion
# systemctl restart systemd-networkd
```
