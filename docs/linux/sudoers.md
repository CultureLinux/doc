# Sudoers 🔐

Cette section explique comment déléguer précisément des privilèges avec `sudo`, sans communiquer le mot de passe du compte cible. 🛡️

## Éditer la configuration ✍️

Les règles locales peuvent être placées dans `/etc/sudoers.d/`. Utilisez toujours `visudo` : il contrôle la syntaxe avant l'enregistrement. ⚠️

### Créer une règle 📄
Crée un fichier de règles dédié au compte `gitlab-runner`.
```sh
# visudo -f /etc/sudoers.d/10-gitlab-runner
```

### Vérifier les permissions 🔎
Les fichiers de `/etc/sudoers.d/` doivent appartenir à `root` et ne doivent être modifiables que par lui.
```sh
# chown root:root /etc/sudoers.d/10-gitlab-runner
# chmod 440 /etc/sudoers.d/10-gitlab-runner
```

## Autoriser une commande 🎯

Cette sous-section montre comment autoriser un utilisateur à lancer une commande précise sous l'identité d'un autre utilisateur.

### Règle ciblée 👤
Autorise `gitlab-runner` à exécuter uniquement l'exécutable indiqué en tant que `otheruser`, sans demander de mot de passe.
```sudoers
gitlab-runner ALL=(otheruser) NOPASSWD: /full/path/to/executable
```

Utilisez un chemin absolu et évitez les jokers (`*`) lorsque cela est possible : ils peuvent étendre les privilèges au-delà de l'intention initiale.

### Exécuter la commande ▶️
Le compte autorisé indique l'utilisateur cible avec `-u`.
```sh
$ sudo -u otheruser /full/path/to/executable
```

## Vérifier les droits ✅

Cette sous-section permet de contrôler les autorisations effectives après une modification.

### Vérifier une règle 🧪
Affiche les commandes que `gitlab-runner` est autorisé à exécuter.
```sh
# sudo -l -U gitlab-runner
```

### Tester comme le compte concerné 🧑‍💻
Ouvre une session avec le compte de service puis liste ses droits `sudo`.
```sh
# su - gitlab-runner
$ sudo -l
```
