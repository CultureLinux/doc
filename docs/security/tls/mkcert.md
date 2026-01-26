# Mkcert 🔐

Mkcert sert à générer facilement des certificats TLS locaux **fiables par ton système** sans te battre avec OpenSSL comme un sanglier sous anxiolytiques 😤🐗

---

## Installation 🧱

On installe les dépendances nécessaires :
- `curl` pour interroger l’API GitHub 🌐
- `nss-tools` pour enregistrer l’autorité de certification dans le store système 🗝️
- `wget` pour récupérer le binaire 📦

```bash
dnf install -y curl nss-tools wget
```

Téléchargement de la **dernière version officielle** de mkcert pour Linux amd64 :

```bash
curl -s https://api.github.com/repos/FiloSottile/mkcert/releases/latest \
  | grep browser_download_url \
  | grep 'linux-amd64' \
  | cut -d '"' -f 4 \
  | wget -i -
```

Installation du binaire dans le PATH système 🔒

```bash
mv mkcert-v*-linux-amd64 /usr/local/bin/mkcert
chmod 755 /usr/local/bin/mkcert
```

---

## Initialisation 🏗️

Création et installation de l’autorité de certification locale dans le système et les navigateurs compatibles 🌍

```bash
mkcert -install
```

Vérification de la présence de la CA racine :

```bash
ls -l ~/.local/share/mkcert/rootCA.pem
```

---

## Génération de certificats 🔩

Les certificats générés sont **strictement réservés au local** 🚫

### Sous-domaine 🌐

```bash
mkcert web.lab.clinux.fr
```

Génère :
- `web.lab.clinux.fr.pem`
- `web.lab.clinux.fr-key.pem`

---

### Adresse IP 🧮

```bash
mkcert 192.168.1.211 127.0.0.1 ::1
```

Un seul certificat valable pour toutes les IPs listées.

---

### Wildcard (multiples sous-domaines) 🌳

```bash
mkcert "*.lab.clinux.fr"
```

⚠️ Ne couvre pas le domaine racine.

Solution complète :

```bash
mkcert "lab.clinux.fr" "*.lab.clinux.fr"
```

---

## Bonnes pratiques 🧠

- ❌ Jamais en production
- 🔑 Ne jamais versionner les clés privées
- 🔄 Redémarrer le navigateur si besoin
- ✅ Compatible Firefox, Chrome, Chromium, NSS, OpenSSL

