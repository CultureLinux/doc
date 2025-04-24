# 🚀 Permissions Linux

## 📚 Permissions standard
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

## Permissions avancées
### Setuid 
⚠️ Attention a ce droit surtout pour root car il est executé avec cet utilisateur

    chmod u+s monbinaire
    chmod u-s monbinaire

### Setgid

    chmod u+s monbinaire
    chmod u-s monbinaire



### Répertoire
⚠️ Pour renter dans un répertoire, il faut avoir les droits d'exécution (`x`).

## 📚 Sticky Bit

Le sticky bit est un attribut qui empêche les utilisateurs normaux d'effacer ou de déplacer des fichiers dans un répertoire. Cela peut être utile pour protéger des fichiers sensibles.

⚠️ Le sticky bit ne bloque que les suppressions entre utilisateurs normaux. Le propriétaire du dossier garde le contrôle total.

```bash
chmod +t /home/alice/monrep
chmod -t /home/alice/monrep
```

## setuid et setgid
### setuid

Le setuid est un attribut qui permet au propriétaire du fichier de l'exécuter avec les privilèges du propriétaire, plutôt que ceux de l'utilisateur exécutant le programme.

```bash
chmod u+s /usr/bin/sudo
```

## 🌿 lsattr et chattr

`lsattr` et `chattr` sont des commandes utiles pour manipuler les attributs (flags) de fichiers ou dossiers, qui peuvent inclure des droits additionnels :

- **Flags**: 
    - **a**: uniquement ajouter du contenu
    - **c**: compression du contenu sur le disque 
    - **d**: ne sauvegarde pas lors de l'appel à dump
    - **i**: rendre le fichier immuable 
    - **j**: journalise les modifications avant d'écrire sur le disque
    - **s**: passe les blocks à 0 lors de la suppression
    - **T**: permet de separer les répertoires enfants dans des blocs disques différents

`chattr` permet de définir ces attributs sur des fichiers ou dossiers :

```bash
sudo chattr +i fichier.txt
sudo chattr -i fichier.txt
```

## 📗 ACLs (Access Control Lists)

ACLs sont une alternative aux permissions basées sur les groupes et les utilisateurs. Elles permettent de donner des droits spécifiques à des individus ou des groupes individuels.

### Test 
```bash
#!/bin/bash

echo "🔍 Vérification du support ACL sur le système de fichiers..." 
echo "------------------------------------------------------------"

# 1. Afficher les points de montage et leurs systèmes de fichiers
echo -e "\n📁 Points de montage :"
findmnt -o TARGET,FSTYPE,OPTIONS | grep -v tmpfs

# 2. Vérifier si 'acl' est une option de montage active
echo -e "\n🔧 Vérification des options de montage contenant 'acl' :"
mount | grep acl || echo "❌ Aucun système de fichiers monté avec l'option explicite 'acl'. Peut être activé par défaut."

# 3. Vérifier les entrées fstab
echo -e "\n📜 Vérification de /etc/fstab pour 'acl' :"
grep acl /etc/fstab || echo "❌ Aucune option 'acl' dans /etc/fstab."

# 4. Test réel sur un fichier temporaire
TMPFILE="/tmp/test_acl_$$"
touch "$TMPFILE"

echo -e "\n🧪 Test pratique : ajout d'une ACL sur un fichier temporaire : $TMPFILE"
if setfacl -m u:$(whoami):r "$TMPFILE" 2>/dev/null; then
    echo "✅ ACL ajoutée avec succès. Support ACL fonctionnel sur /tmp."
    getfacl "$TMPFILE"
else
    echo "❌ Impossible d'ajouter une ACL. Ce système de fichiers ne supporte probablement pas les ACLs."
fi

rm -f "$TMPFILE"

# 5. Vérification du kernel (pour les systèmes avec config.gz dispo)
echo -e "\n🧠 Vérification du support ACL dans le noyau :"
if [ -f /proc/config.gz ]; then
    zgrep CONFIG_EXT4_FS_POSIX_ACL /proc/config.gz || echo "❌ Option noyau CONFIG_EXT4_FS_POSIX_ACL absente ou désactivée."
else
    echo "ℹ️ Fichier /proc/config.gz introuvable. Impossible de vérifier la config du noyau."
fi

echo -e "\n🎉 Vérification terminée."
```

### Basique

```bash
setfacl -m u:alice:rwx fichier.txt
setfacl -x u:alice:w fichier.txt
```

### Forcer les droits sur tous les fichiers et dossiers dans un répertoire

```bash
setfacl -d -m u:alice:rwX /home/alice/monrep
setfacl -m u:alice:rwX /home/alice/monrep
```

