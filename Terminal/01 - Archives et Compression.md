# Archives et Compression

## Qu'est-ce qu'un fichier compressé ?

Un fichier compressé (archive) est un **conteneur** qui regroupe un ou plusieurs fichiers en un seul, souvent en réduisant leur taille totale.

> [!tip] Analogie Gaming
> Imagine que tu veux donner ton monde Minecraft à un ami :
> - Ton monde = un **dossier** avec des centaines de fichiers (chunks, player data, etc.)
> - Tu ne vas pas envoyer 500 fichiers un par un
> - Tu fais un **ZIP** = tu mets tout dans un seul fichier compressé
> - Ton ami **décompresse** le ZIP et récupère le dossier complet
>
> C'est exactement ce que font `zip`, `tar`, `gzip` : emballer et déballer.

---

## Installation

```bash
# zip et unzip
sudo apt update
sudo apt install zip unzip

# tar est généralement pré-installé, sinon :
sudo apt install tar

# 7zip (optionnel)
sudo apt install p7zip-full
```

---

## Le workflow Holberton

Quand tu reçois une archive (exercice, projet, etc.), suis toujours ce workflow :

```
1. Télécharger l'archive
        ↓
2. Inspecter le contenu (sans extraire)
        ↓
3. Extraire
        ↓
4. Naviguer dans le dossier extrait
        ↓
5. Travailler
```

```bash
# 1. Télécharger (ou copier)
wget https://example.com/projet.zip

# 2. Inspecter (voir ce qu'il y a dedans)
unzip -l projet.zip

# 3. Extraire
unzip projet.zip

# 4. Naviguer
cd projet/

# 5. Travailler
ls -la
```

> [!warning] Toujours inspecter avant d'extraire
> Certaines archives ne contiennent pas de dossier racine et vont **éparpiller** les fichiers dans ton répertoire courant. L'inspection te permet de le savoir avant.

---

## ZIP : Commandes complètes

### Décompresser (unzip)

| Commande | Description |
|---|---|
| `unzip archive.zip` | Extraire dans le dossier courant |
| `unzip archive.zip -d /chemin/dossier` | Extraire dans un dossier spécifique |
| `unzip -l archive.zip` | Lister le contenu (sans extraire) |
| `unzip -q archive.zip` | Extraire en mode silencieux (quiet) |
| `unzip -o archive.zip` | Extraire et écraser les fichiers existants (overwrite) |
| `unzip -n archive.zip` | Extraire sans écraser les fichiers existants (no overwrite) |
| `unzip archive.zip fichier.txt` | Extraire un seul fichier |
| `unzip -P motdepasse archive.zip` | Extraire avec un mot de passe |
| `unzip -t archive.zip` | Tester l'intégrité de l'archive |

### Exemples

```bash
# Extraire dans un dossier spécifique
unzip projet.zip -d ~/projets/

# Voir le contenu sans extraire
unzip -l projet.zip
```

Sortie de `unzip -l` :
```
Archive:  projet.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
      156  2024-01-15 10:30   projet/main.c
       89  2024-01-15 10:30   projet/header.h
      234  2024-01-15 10:30   projet/Makefile
---------                     -------
      479                     3 files
```

```bash
# Extraire un seul fichier
unzip projet.zip projet/main.c

# Tester l'intégrité
unzip -t projet.zip
# No errors detected in compressed data of projet.zip.
```

### Compresser (zip)

| Commande | Description |
|---|---|
| `zip archive.zip fichier.txt` | Compresser un fichier |
| `zip archive.zip f1.c f2.c f3.c` | Compresser plusieurs fichiers |
| `zip -r archive.zip dossier/` | Compresser un dossier récursivement |
| `zip -9 archive.zip fichier.txt` | Compression maximale (lent) |
| `zip -0 archive.zip fichier.txt` | Pas de compression (stockage seulement, rapide) |
| `zip -u archive.zip nouveau.c` | Ajouter/mettre à jour un fichier dans une archive existante |
| `zip -d archive.zip fichier.txt` | Supprimer un fichier de l'archive |
| `zip -e archive.zip fichier.txt` | Créer une archive chiffrée (avec mot de passe) |

### Exemples

```bash
# Compresser un dossier entier
zip -r projet.zip projet/

# Compresser tous les fichiers .c
zip sources.zip *.c

# Compression maximale
zip -9 -r projet_compressed.zip projet/

# Ajouter un fichier à une archive existante
zip -u projet.zip nouveau_fichier.c

# Supprimer un fichier de l'archive
zip -d projet.zip fichier_obsolete.c
```

---

## TAR et TAR.GZ : Commandes complètes

### Qu'est-ce que tar ?

- **tar** = Tape Archive. Regroupe plusieurs fichiers en **un seul** (sans compression).
- **tar.gz** (ou .tgz) = tar + compression **gzip**. C'est le format le plus courant sous Linux.
- **tar.xz** = tar + compression **xz**. Meilleure compression que gzip, mais plus lent.
- **tar.bz2** = tar + compression **bzip2**. Compression intermédiaire.

### Les flags de tar

| Flag | Signification | Quand l'utiliser |
|---|---|---|
| `x` | e**X**tract (extraire) | Décompresser |
| `c` | **C**reate (créer) | Compresser |
| `t` | lis**T** (lister) | Voir le contenu |
| `z` | g**Z**ip (compression gzip) | Pour .tar.gz / .tgz |
| `J` | xz (compression xz) | Pour .tar.xz |
| `j` | bzip2 (compression bzip2) | Pour .tar.bz2 |
| `f` | **F**ile (fichier) | Spécifier le nom de l'archive (TOUJOURS) |
| `v` | **V**erbose (détaillé) | Voir les fichiers traités |
| `C` | **C**hange directory | Extraire dans un dossier spécifique |

> [!tip] Astuce mnémotechnique
> - **Extraire** : `x` = e**X**traire
> - **Créer** : `c` = **C**réer
> - **Lister** : `t` = lis**T**er
> - Le `f` est **toujours** à la fin (juste avant le nom du fichier)

### Extraire

```bash
# Extraire un .tar.gz
tar xzf archive.tar.gz

# Extraire un .tar.gz avec verbosité
tar xzvf archive.tar.gz

# Extraire dans un dossier spécifique
tar xzf archive.tar.gz -C /chemin/destination/

# Extraire un .tar.xz
tar xJf archive.tar.xz

# Extraire un .tar.bz2
tar xjf archive.tar.bz2

# Extraire un .tar simple (pas de compression)
tar xf archive.tar
```

### Créer

```bash
# Créer un .tar.gz
tar czf archive.tar.gz dossier/

# Créer un .tar.gz avec verbosité
tar czvf archive.tar.gz dossier/

# Créer un .tar.xz (meilleure compression)
tar cJf archive.tar.xz dossier/

# Créer un .tar.bz2
tar cjf archive.tar.bz2 dossier/

# Créer un .tar simple (sans compression)
tar cf archive.tar dossier/

# Créer depuis plusieurs fichiers
tar czf sources.tar.gz *.c *.h Makefile
```

### Lister le contenu

```bash
# Lister le contenu d'un .tar.gz
tar tzf archive.tar.gz

# Lister avec détails (permissions, taille, date)
tar tzvf archive.tar.gz
```

Sortie de `tar tzvf` :
```
drwxr-xr-x user/user     0 2024-01-15 10:30 projet/
-rw-r--r-- user/user   156 2024-01-15 10:30 projet/main.c
-rw-r--r-- user/user    89 2024-01-15 10:30 projet/header.h
-rw-r--r-- user/user   234 2024-01-15 10:30 projet/Makefile
```

---

## Tableau de reconnaissance des formats

| Extension | Type | Commande d'extraction | Commande de création |
|---|---|---|---|
| `.zip` | Archive compressée | `unzip archive.zip` | `zip -r archive.zip dossier/` |
| `.tar.gz` | tar + gzip | `tar xzf archive.tar.gz` | `tar czf archive.tar.gz dossier/` |
| `.tgz` | tar + gzip (alias) | `tar xzf archive.tgz` | `tar czf archive.tgz dossier/` |
| `.tar.xz` | tar + xz | `tar xJf archive.tar.xz` | `tar cJf archive.tar.xz dossier/` |
| `.tar.bz2` | tar + bzip2 | `tar xjf archive.tar.bz2` | `tar cjf archive.tar.bz2 dossier/` |
| `.tar` | tar (sans compression) | `tar xf archive.tar` | `tar cf archive.tar dossier/` |
| `.gz` | gzip (fichier seul) | `gunzip fichier.gz` ou `gzip -d fichier.gz` | `gzip fichier` |
| `.7z` | 7-Zip | `7z x archive.7z` | `7z a archive.7z dossier/` |

> [!tip] Reconnaître le format
> Regarde l'**extension** du fichier :
> - `.zip` → `unzip`
> - `.tar.quelquechose` → `tar` avec le bon flag
> - `.gz` seul → `gunzip`
> - `.7z` → `7z`

---

## Erreurs courantes et solutions

### "unzip: command not found"

```bash
sudo apt install unzip
```

### "gzip: stdin: not in gzip format"

```
gzip: stdin: not in gzip format
tar: Child returned status 1
```

**Cause** : Tu utilises `tar xzf` sur un fichier qui n'est **pas** compressé en gzip (peut-être un .tar simple ou un .tar.xz).

**Solution** : Vérifier le type du fichier :
```bash
file archive.tar.gz
# Si la sortie dit "XZ compressed" → utilise tar xJf
# Si la sortie dit "bzip2 compressed" → utilise tar xjf
# Si la sortie dit "POSIX tar archive" → utilise tar xf
```

### "tar: Refusing to read archive from terminal"

**Cause** : Tu as oublié le flag `f`.

```bash
# MAUVAIS
tar xz archive.tar.gz

# BON
tar xzf archive.tar.gz
#    ^ le f !
```

### "cannot open: No such file or directory"

**Cause** : Le chemin ou le nom du fichier est incorrect.

```bash
# Vérifier que le fichier existe
ls -la *.zip *.tar.gz
```

### "permission denied"

```bash
# Si c'est un problème de permissions sur le dossier de destination
sudo tar xzf archive.tar.gz -C /opt/

# Ou changer les permissions
chmod +r archive.zip
```

### L'archive s'extrait sans dossier racine

Les fichiers se retrouvent éparpillés dans le dossier courant.

**Prévention** : Toujours inspecter avant d'extraire :
```bash
# Pour un ZIP
unzip -l archive.zip

# Pour un tar.gz
tar tzf archive.tar.gz
```

Si pas de dossier racine, extraire dans un nouveau dossier :
```bash
mkdir projet && cd projet
unzip ../archive.zip
# ou
tar xzf ../archive.tar.gz
```

### Caractères bizarres dans les noms de fichiers

```bash
# Forcer l'encodage UTF-8 pour unzip
unzip -O UTF-8 archive.zip
```

---

## Checklist

> [!tip] Checklist Archives
> - [ ] **Identifier le format** : regarde l'extension (.zip, .tar.gz, .tar.xz, etc.)
> - [ ] **Inspecter avant d'extraire** : `unzip -l` ou `tar tzf`
> - [ ] **Vérifier le dossier racine** : est-ce que l'archive contient un dossier parent ?
> - [ ] **Extraire dans le bon endroit** : utilise `-d` (unzip) ou `-C` (tar)
> - [ ] **Vérifier après extraction** : `ls -la` pour confirmer
> - [ ] Pour **créer** : ne pas oublier `-r` pour zip, les bons flags pour tar
> - [ ] Pour **tar** : ne jamais oublier le flag `f` !

---

## Exercices

### Exercice 1 : Manipulation ZIP

1. Crée un dossier `mon_projet` avec 3 fichiers .c dedans
2. Compresse le dossier en `mon_projet.zip`
3. Supprime le dossier original
4. Inspecte l'archive (liste le contenu)
5. Extrais l'archive
6. Vérifie que tout est intact

### Exercice 2 : tar.gz

1. Crée un dossier avec un Makefile et des fichiers source
2. Crée une archive `.tar.gz`
3. Crée une archive `.tar.xz` du même dossier
4. Compare les tailles des deux archives (`ls -lh`)
5. Extrais chacune dans un dossier différent

### Exercice 3 : Identifier et extraire

On te donne ces fichiers :
- `data.zip`
- `backup.tar.gz`
- `archive.tar.xz`
- `notes.tar.bz2`
- `single.gz`

Écris la commande d'extraction correcte pour chacun.

**Réponses** :
```bash
unzip data.zip
tar xzf backup.tar.gz
tar xJf archive.tar.xz
tar xjf notes.tar.bz2
gunzip single.gz
```
