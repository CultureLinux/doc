# 🚀 Permissions Linux

Les permissions Unix/Linux sont une manière de contrôler l'accès à des ressources (fichiers, dossiers) par les utilisateurs. Les permissions peuvent être définies en trois niveaux : lecteur (`r`), écrivain (`w`) et exécuteur (`x`). Ces droits peuvent être combinés pour créer différents niveaux de contrôle.

## 📚 Format
### 📝 Alphabetique

- **Lecture (r)**: L'utilisateur peut lire le contenu du fichier ou dossier.
- **Ecriture (w)**: L'utilisateur peut écrire dans le fichier ou supprimer son contenu.
- **Execution (x)**: L'utilisateur peut exécuter un programme ou un script associé au fichier.

Par exemple, le droit `rwx` signifie que l'utilisateur a les droits de lecture, d'écriture et d'exécution sur un fichier ou dossier.

### 📝 Numérique

Les permissions peuvent également être codées en base 10. Chaque droits est représenté par une valeur numérique :

- **Lecture (r)**: 4
- **Ecriture (w)**: 2
- **Execution (x)**: 1

Pour combiner ces droits, on les ajoute entre eux. Par exemple, la permission `rwx` correspond au calcul suivant : 4 + 2 + 1 = 7.

## 🌿 lsattr et chattr

`lsattr` et `chattr` sont des commandes utiles pour manipuler les attributs (flags) de fichiers ou dossiers, qui peuvent inclure des droits additionnels :

- **Flags**: a (append), c (compressed), d (delayed allocation), i (immutable), j (journaling), s (synchronous updates), t (timeseekable), u (undeletable), v (version control)

`chattr` permet de définir ces attributs sur des fichiers ou dossiers :

```bash
sudo chattr +i fichier.txt
sudo chattr -i fichier.txt
```

## 📗 ACLs (Access Control Lists)

ACLs sont une alternative aux permissions basées sur les groupes et les utilisateurs. Elles permettent de donner des droits spécifiques à des individus ou des groupes individuels :

```bash
# Ajouter un droit spécifique : 
setfacl -m u:alice:rwx fichier.txt

# Supprimer un droit spécifique : 
setfacl -x u:alice:w fichier.txt
```

**Note**: Les ACLs sont disponibles sur certains systèmes Linux tels que Red Hat, CentOS et Ubuntu. Vous pouvez vérifier leur activation avec la commande `getfacl`.
