# Crontab

Un utilitaire unix qui permet d'executer des taches automatiques à un certain moment.

## 👷‍♀️​ Structure

### Utilisateur
```
* * * * * commande
```

### Système
```
* * * * * utilisateur commande
```

### Champs

* Minute (0–59)
* Heure (0–23)
* Jour du mois (1–31)
* Mois (1–12)
* Jour de la semaine (0–7, avec 0 ou 7 = dimanche)

### Meta caractères 

- Les champs peuvent être remplacés par des caractères spéciaux :
    - `*` : toutes les valeurs possibles
    - `,` : séparation entre valeurs
    - `-` : intervalle de valeurs
    - `/` : pas (ex: 0/2 = tous les deux minutes)

### Exemples

#### Chaque jour a 8:00

```
0 8 * * * /path/absolu/script.sh
```

#### Le 1er jour du mois à 2:00

```
0 2 1 * * /path/absolu/script.sh
```

#### Le 15 de tous les mois impairs à 16:00

```
0 16 15 */2 * /path/absolu/script.sh
```

#### Tous les lundis matin à 7:00

```
0 7 * * 1 /path/absolu/script.sh
```

#### Les 3 premiers jours du mois à 9:00

```
0 9 1-3 * * /path/absolu/script.sh
```

#### Toutes les 15 minutes entre 7:00 et 17:00 de lundi a vendredi

```
*/15 7-17 * * 1-5 /path/absolu/script.sh
```

### Limitations

* La résolution minimale est de 1 minute 
* Le script doit être défini par son chemin absolu
* Le script n'a pas connaissance de l'environnement (.bashrc, .bash_profile)

## Opérations

### Par utilisateur

```
crontab -u rocky -l
```

### Liste 

```
crontab -l
```

### Création 

```
export EDITOR=vim
crontab -e
```

### Suppression 

```
crontab -r
```

### Suivi des logs

```
sudo tail -f /var/log/cron
sudo journalctl -u crond.service -f
```




