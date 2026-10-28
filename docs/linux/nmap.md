# Nmap 🔍

`nmap` permet d'inventorier des hôtes et services réseau. N'analysez que les réseaux et systèmes pour lesquels vous disposez d'une autorisation explicite. 🛡️

## Découvrir les hôtes 👀

Cette sous-section identifie les machines actives sans lancer de scan de ports.

### Scanner un réseau local 📡
Liste les hôtes répondant sur le réseau indiqué. Sur un réseau Ethernet local, Nmap privilégie généralement ARP ; ce n'est donc pas uniquement un scan ICMP.
```sh
$ nmap -sn 192.168.1.0/24
```

### Désactiver la résolution DNS ⚡
Évite les requêtes DNS inverses et accélère l'affichage lorsque les noms d'hôtes ne sont pas nécessaires.
```sh
$ nmap -sn -n 192.168.1.0/24
```

### Vérifier une liste de cibles 📋
`-sL` n'envoie aucun paquet aux cibles : il affiche seulement les adresses que Nmap analyserait.
```sh
$ nmap -sL 192.168.1.0/24
```

## Vérifier des services autorisés 🧪

Cette sous-section aide à contrôler l'exposition d'un hôte dont vous assurez l'administration.

### Contrôler des ports précis 🎯
Analyse uniquement les ports indiqués sur l'hôte concerné.
```sh
$ nmap -p 22,80,443 serveur.example.internal
```

### Identifier les versions de service 🔎
Tente d'identifier les services présents sur les ports demandés.
```sh
$ nmap -sV -p 22,80,443 serveur.example.internal
```

### Enregistrer un résultat 📝
Écrit le résultat normal dans un fichier daté, utile pour comparer l'exposition dans le temps.
```sh
$ nmap -sn -n 192.168.1.0/24 -oN inventaire-reseau.txt
```

## Lire les résultats 📖

- `Host is up` : l'hôte a répondu à au moins une sonde.
- `open` : un service accepte les connexions sur ce port.
- `closed` : l'hôte est joignable, mais aucun service n'écoute sur ce port.
- `filtered` : un pare-feu ou le réseau empêche Nmap de déterminer l'état du port.

Un hôte absent du résultat n'est pas nécessairement éteint : il peut filtrer les sondes de découverte.
