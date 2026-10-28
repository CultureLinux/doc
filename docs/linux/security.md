# Sécurité 🛡️

Cette section présente deux outils d'audit local : RKHunter et Lynis. Ils signalent des anomalies et des pistes de durcissement ; leurs alertes ne constituent pas à elles seules une preuve de compromission. 🔍

## Avant un audit ✅

Cette sous-section prépare une exécution fiable des outils de sécurité.

### Mettre le système à jour 📦
Installez les mises à jour disponibles avant l'audit afin de ne pas analyser des paquets connus comme obsolètes.
```sh
# apt update && apt upgrade
# dnf upgrade
```

### Conserver les résultats 📝
Exécutez les audits avec les droits administrateur et archivez les sorties dans un emplacement protégé. Comparez les résultats avec les audits précédents plutôt que de corriger chaque avertissement automatiquement.

## RKHunter 🕵️

RKHunter (*Rootkit Hunter*) vérifie notamment la présence de rootkits connus, des permissions inhabituelles et certains fichiers système modifiés.

### Installer 📥
Sur Debian, installez le paquet depuis les dépôts. Vérifiez la disponibilité et le nom du paquet dans votre dépôt Rocky avant installation.
```sh
# apt install rkhunter
# dnf install rkhunter
```

### Mettre à jour les données 🔄
Met à jour les définitions utilisées par RKHunter. Vérifiez le résultat et les paramètres de proxy éventuels avant de continuer.
```sh
# rkhunter --update
```

### Lancer le contrôle 🔍
Exécute les contrôles et n'affiche à l'écran que les avertissements.
```sh
# rkhunter --check --report-warnings-only
```

### Créer une base de référence 📌
Après une mise à jour légitime du système et seulement après avoir examiné les changements, met à jour la base des propriétés de fichiers.
```sh
# rkhunter --propupd
```

Ne relancez pas `--propupd` pour faire disparaître une alerte non comprise : cela pourrait enregistrer une modification malveillante comme référence.

## Lynis 🔎

Lynis réalise un audit de configuration, de correctifs et de durcissement pour les systèmes Unix. Il génère des suggestions à analyser selon le rôle du serveur et les contraintes de votre environnement.

### Installer depuis le gestionnaire de paquets 📦
L'installation par paquet est la méthode la plus simple à maintenir. La version proposée par une distribution peut être moins récente que celle de CISOfy.
```sh
# apt install lynis
# dnf install lynis
```

### Lancer un audit système 🧪
Les droits `root` permettent à Lynis d'effectuer davantage de contrôles.
```sh
# lynis audit system
```

### Consulter les résultats 📜
Lynis affiche un résumé à la fin de l'audit et conserve habituellement son journal dans `/var/log/lynis.log`.
```sh
# tail -n 100 /var/log/lynis.log
# lynis show version
```

### Exécuter depuis le dépôt amont 🧰
Pour tester la version amont, clonez le dépôt dans un répertoire dédié et exécutez l'outil depuis ce répertoire. Si vous l'exécutez avec `root`, assurez-vous que les fichiers sont détenus par `root` et non modifiables par un utilisateur non privilégié.
```sh
$ git clone https://github.com/CISOfy/lynis.git ~/src/lynis
$ cd ~/src/lynis
# chown -R root:root .
# ./lynis audit system
```

## Traiter les alertes 🧭

### Vérifier avant de corriger 🔍

1. Identifiez le fichier, le paquet ou le paramètre signalé.
2. Vérifiez le changement dans les journaux, le gestionnaire de paquets et la gestion de configuration.
3. Évaluez l'impact de la recommandation sur le service avant de l'appliquer.
4. Documentez la décision : correction, exception justifiée ou faux positif.

Un audit régulier est utile, mais il complète — sans les remplacer — les mises à jour, les sauvegardes testées, la surveillance des journaux et le contrôle des accès.
