# GNOME 🖥️

Cette section regroupe quelques réglages GNOME accessibles depuis le terminal avec `gsettings`. Les commandes doivent être exécutées dans la session graphique de l'utilisateur concerné. ⚙️

## Verr. Num 🔢

Cette sous-section permet de consulter et mémoriser l'état du pavé numérique.

### Voir l'état actuel 👀
Affiche l'état courant de Verr. Num dans la session GNOME.
```sh
$ gsettings get org.gnome.desktop.peripherals.keyboard numlock-state
```

### Vérifier la mémorisation 💾
Indique si GNOME mémorise l'état du pavé numérique entre les sessions.
```sh
$ gsettings get org.gnome.desktop.peripherals.keyboard remember-numlock-state
```

### Activer Verr. Num ✅
Active le pavé numérique pour la session courante.
```sh
$ gsettings set org.gnome.desktop.peripherals.keyboard numlock-state true
```

### Mémoriser l'état 🔒
Demande à GNOME de conserver cet état lors des prochaines sessions.
```sh
$ gsettings set org.gnome.desktop.peripherals.keyboard remember-numlock-state true
```

Si la commande échoue depuis une session SSH, ouvrez un terminal dans GNOME ou exécutez-la dans le contexte D-Bus de l'utilisateur connecté.
