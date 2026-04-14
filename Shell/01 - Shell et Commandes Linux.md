# Shell et Commandes Linux

> [!info] Contexte
> Le Shell n'est pas juste une ligne de commande austere : c'est un **langage de programmation** puissant qui fait le pont entre toi et le noyau (Kernel) du systeme d'exploitation. Maitriser le Shell est une competence fondamentale pour tout developpeur.

---

## 1. Terminal vs Shell

| Concept | Definition |
|---------|-----------|
| **Terminal** | L'application graphique (fenetre) dans laquelle tu tapes des commandes |
| **Shell** | Le **programme** qui interprete tes commandes. Exemples : `bash`, `zsh`, `sh` |
| **Kernel** | Le coeur du systeme d'exploitation. Le Shell communique avec lui |

```
 Utilisateur
     |
     v
 [Terminal]  <-- Interface graphique (ex: GNOME Terminal, iTerm)
     |
     v
  [Shell]    <-- Interprete les commandes (ex: /bin/bash)
     |
     v
 [Kernel]    <-- Execute les operations systeme
     |
     v
 [Hardware]  <-- CPU, disque, memoire...
```

### Types de Shell

- **Login Shell** : Quand tu te connectes (via SSH ou au demarrage). Lit `/etc/profile` puis `~/.bash_profile`
- **Interactive Shell** : Quand tu ouvres un terminal. Lit `~/.bashrc`

---

## 2. Navigation dans le systeme de fichiers

| Commande | Description | Exemple |
|----------|-------------|---------|
| `pwd` | Affiche le repertoire courant (Print Working Directory) | `pwd` |
| `cd` | Change de repertoire | `cd /home/user` |
| `cd ..` | Remonte d'un niveau | `cd ..` |
| `cd ~` | Va au repertoire personnel | `cd ~` |
| `cd -` | Retourne au repertoire precedent | `cd -` |
| `ls` | Liste le contenu du repertoire | `ls -la` |
| `mkdir` | Cree un repertoire | `mkdir -p dir1/dir2` |
| `rmdir` | Supprime un repertoire vide | `rmdir dossier` |
| `touch` | Cree un fichier vide (ou met a jour la date) | `touch fichier.txt` |
| `rm` | Supprime un fichier | `rm fichier.txt` |
| `rm -r` | Supprime un repertoire et son contenu | `rm -r dossier/` |
| `cp` | Copie un fichier | `cp source dest` |
| `cp -r` | Copie un repertoire | `cp -r dir1/ dir2/` |
| `mv` | Deplace ou renomme | `mv ancien.txt nouveau.txt` |

> [!warning] `rm -rf` est dangereux !
> Cette commande supprime **tout** sans confirmation. Ne jamais lancer `rm -rf /` !

---

## 3. Visualisation de fichiers

| Commande | Description | Options utiles |
|----------|-------------|----------------|
| `cat` | Affiche tout le contenu d'un fichier | `cat fichier.txt` |
| `head` | Affiche les premieres lignes | `head -n 5 fichier.txt` |
| `tail` | Affiche les dernieres lignes | `tail -n 5 fichier.txt` |
| `less` | Navigation page par page (q pour quitter) | `less fichier.txt` |
| `more` | Similaire a `less`, plus ancien | `more fichier.txt` |
| `wc` | Compte les lignes, mots, octets | `wc -l fichier.txt` |
| `rev` | Inverse les caracteres de chaque ligne | `echo "bonjour" \| rev` |

> [!tip] Verifier qu'un script fait bien 2 lignes
> ```bash
> wc -l mon_script
> ```
> Doit afficher `2` pour les exercices Holberton.

---

## 4. Recherche

### grep (Global Regular Expression Print)

`grep` est l'outil de recherche **dans le contenu** des fichiers.

```bash
grep [options] "motif" [fichier]
```

| Option | Description | Exemple |
|--------|-------------|---------|
| `-i` | Ignore la casse (majuscules/minuscules) | `grep -i "linux" doc.txt` |
| `-r` | Recherche recursive dans un dossier | `grep -r "test" /home/` |
| `-v` | Inverse : affiche les lignes qui NE contiennent PAS le motif | `grep -v "succes" logs.txt` |
| `-c` | Compte le nombre de lignes correspondantes | `grep -c "alerte" info.txt` |
| `-n` | Affiche les numeros de ligne | `grep -n "erreur" log.txt` |
| `-l` | Affiche seulement les noms de fichiers | `grep -l "TODO" *.c` |
| `-w` | Cherche le mot entier | `grep -w "if" code.c` |
| `-E` | Active les expressions regulieres etendues | `grep -E "[0-9]{3}" data.txt` |

> [!example] Usage avec pipe
> ```bash
> ps aux | grep "firefox"     # Trouver le processus Firefox
> cat /etc/passwd | grep "root"  # Chercher root dans passwd
> ```

### find

`find` cherche des **fichiers et repertoires** par nom, type, taille, etc.

```bash
find [chemin] [criteres]
```

| Exemple | Description |
|---------|-------------|
| `find . -name "*.txt"` | Tous les fichiers .txt dans le repertoire courant |
| `find / -type d -name "bin"` | Tous les repertoires nommes "bin" |
| `find . -size +1M` | Fichiers de plus de 1 Mo |
| `find . -perm 755` | Fichiers avec permissions 755 |

### which et type

```bash
which ls        # Affiche le chemin de l'executable ls
type ls         # Affiche si c'est un alias, une fonction ou un executable
```

---

## 5. Traitement de texte

| Commande | Description | Exemple |
|----------|-------------|---------|
| `sort` | Trie les lignes | `sort fichier.txt` |
| `sort -r` | Tri inverse | `sort -r fichier.txt` |
| `sort -n` | Tri numerique | `sort -n notes.txt` |
| `uniq` | Supprime les doublons **adjacents** | `sort fichier \| uniq` |
| `uniq -c` | Compte les occurrences | `sort fichier \| uniq -c` |
| `tr` | Traduit/supprime des caracteres | `echo "abc" \| tr 'a-z' 'A-Z'` |
| `tr -d` | Supprime des caracteres | `echo "he llo" \| tr -d ' '` |
| `cut` | Decoupe des colonnes | `cut -d':' -f1 /etc/passwd` |
| `paste` | Colle des fichiers cote a cote | `paste file1 file2` |

> [!tip] `sort` avant `uniq` !
> `uniq` ne supprime que les doublons **adjacents**. Il faut donc toujours trier d'abord :
> ```bash
> sort fichier.txt | uniq
> ```

---

## 6. Redirections d'Entrees/Sorties (I/O)

### Les 3 flux standard

Chaque programme possede trois "tuyaux" (descripteurs de fichiers) :

| FD | Nom | Role |
|----|-----|------|
| 0 | **stdin** | Entree standard (clavier) |
| 1 | **stdout** | Sortie standard (ecran - resultats normaux) |
| 2 | **stderr** | Sortie d'erreur (ecran - messages d'erreur) |

### Operateurs de redirection

| Operateur | Description | Exemple |
|-----------|-------------|---------|
| `>` | Redirige stdout vers un fichier (**ecrase**) | `ls > liste.txt` |
| `>>` | Redirige stdout vers un fichier (**ajoute**) | `echo "fin" >> liste.txt` |
| `<` | Lit stdin depuis un fichier | `sort < noms.txt` |
| `2>` | Redirige stderr vers un fichier | `find / -name "x" 2> erreurs.txt` |
| `2>&1` | Redirige stderr vers stdout | `cmd > out.txt 2>&1` |
| `\|` | **Pipe** : stdout d'une commande -> stdin de la suivante | `cat file \| grep "mot"` |
| `tee` | Ecrit dans un fichier ET sur stdout | `ls \| tee liste.txt` |

> [!warning] Comprendre `2>&1`
> ```
> 2   : On cible le flux d'erreur (stderr)
> >   : On demande une redirection
> &1  : On pointe vers le descripteur 1 (stdout)
>       Le & est crucial : sans lui, "1" serait un nom de fichier !
> ```
> Signification : "Prends tout ce qui sort par le tuyau des erreurs et envoie-le dans le tuyau de sortie standard."

> [!example] Combiner redirections
> ```bash
> # Envoyer resultats ET erreurs dans le meme fichier
> find /etc -name "passwd" > resultats.txt 2>&1
> 
> # Ignorer les erreurs
> find / -name "secret" 2>/dev/null
> ```

---

## 7. Variables Shell

### Variables locales vs environnement

| Type | Definition | Visible par les enfants ? |
|------|-----------|--------------------------|
| **Locale** | Existe seulement dans le shell actuel | Non |
| **Environnement** | Exportee, visible par tous les processus enfants | Oui |

```bash
# Variable locale
nom="Alice"
echo $nom          # Affiche: Alice

# Variable d'environnement (exportee)
export API_KEY="12345"
```

> [!warning] Pas d'espaces autour du `=` !
> ```bash
> nom="Alice"      # Correct
> nom = "Alice"    # ERREUR ! Le shell croit que "nom" est une commande
> ```

### Variables reservees importantes

| Variable | Role |
|----------|------|
| `$HOME` | Repertoire personnel de l'utilisateur |
| `$USER` | Nom de l'utilisateur courant |
| `$PATH` | Liste des repertoires ou le shell cherche les executables |
| `$PWD` | Repertoire courant |
| `$PS1` | Definit l'apparence du prompt |
| `$?` | Code de retour de la derniere commande (0 = succes) |
| `$$` | PID du shell courant |

### Commandes de gestion

| Commande | Action | Portee |
|----------|--------|--------|
| `printenv` | Liste les variables d'environnement | Systeme / Enfants |
| `set` | Liste TOUT (vars locales + env + fonctions) | Session locale |
| `export VAR` | Rend une variable globale (environnement) | Session + Enfants |
| `unset VAR` | Supprime une variable de la memoire | Session |

---

## 8. Fichiers d'initialisation (Init Files)

### Pour Bash

| Fichier | Quand il est lu | Usage |
|---------|----------------|-------|
| `/etc/profile` | Login shell (pour TOUS les utilisateurs) | Config systeme globale |
| `/etc/profile.d/` | Scripts supplementaires lus par `/etc/profile` | Modules de config |
| `~/.bash_profile` | Login shell (utilisateur specifique) | Variables d'env personnelles |
| `~/.bashrc` | Chaque nouveau terminal interactif | Alias, fonctions, prompt |
| `~/.profile` | Login shell (si `~/.bash_profile` n'existe pas) | Alternative standard |

> [!tip] Appliquer les changements sans redemarrer
> ```bash
> source ~/.bashrc
> # ou equivalent :
> . ~/.bashrc
> ```

### Difference entre `source` et `./`

```bash
./script.sh       # Execute dans un NOUVEAU processus (subshell)
                   # Les variables definies dans le script disparaissent apres

source script.sh   # Execute dans le processus ACTUEL
. script.sh        # Identique a source (raccourci historique)
                   # Les variables et alias RESTENT dans le terminal
```

---

## 9. Expansions Shell

Le Shell **transforme** le texte avant de l'executer. C'est la ou reside la vraie puissance.

### A. Expansion des accolades (Brace Expansion)

```bash
mkdir folder{1,2,3}      # Cree folder1, folder2, folder3
echo {A..Z}              # Affiche l'alphabet complet
echo {1..10}             # Affiche 1 2 3 4 5 6 7 8 9 10
touch file{01..05}.txt   # Cree file01.txt a file05.txt
```

### B. Expansion des parametres (Variable Expansion)

```bash
echo $HOME                   # Affiche le repertoire personnel
echo ${VAR:-"defaut"}        # Utilise "defaut" si VAR est vide
echo ${VAR:0:5}              # Prend les 5 premiers caracteres
echo ${#VAR}                 # Longueur de la variable
echo ${filename%.txt}        # Supprime le suffixe .txt
```

### C. Substitution de commande

```bash
aujourdhui=$(date +%D)
echo "Nous sommes le $aujourdhui"

fichiers=$(ls *.c)
echo "Fichiers C : $fichiers"
```

### D. Expansion arithmetique

```bash
echo $((5 + 2))          # 7
echo $((10 / 3))         # 3 (division entiere)
echo $((2 ** 8))         # 256
resultat=$((n * 2 + 1))
```

### E. Globbing (Wildcards)

| Symbole | Signification | Exemple |
|---------|--------------|---------|
| `*` | N'importe quel nombre de caracteres | `ls *.c` |
| `?` | Exactement un caractere | `ls fichier?.txt` |
| `[ab]` | Soit 'a', soit 'b' | `ls [AB]*.txt` |
| `[0-9]` | Un chiffre | `ls fichier[0-9].txt` |

---

## 10. Quoting (Guillemets)

C'est souvent la source de bugs en Shell.

| Type | Comportement | Exemple |
|------|-------------|---------|
| `'texte'` | **Strong quoting** : tout est litteral, rien n'est interprete | `echo '$HOME'` affiche `$HOME` |
| `"texte"` | **Weak quoting** : les variables `$` et substitutions `$()` sont interpretees, les espaces sont proteges | `echo "$HOME"` affiche `/home/user` |
| `\` | Echappe le caractere suivant | `echo \$HOME` affiche `$HOME` |

```bash
VAR="AI"

echo '$VAR'     # $VAR     (litteral)
echo "$VAR"     # AI       (interprete)
echo \$VAR      # $VAR     (echappe)
```

---

## 11. Alias

```bash
# Creer un alias
alias ll='ls -alF'
alias gs='git status'

# Lister tous les alias
alias

# Supprimer un alias
unalias ll

# Ignorer un alias temporairement (utiliser la vraie commande)
\ls              # Le backslash ignore l'alias
```

> [!warning] Les alias ne survivent pas au redemarrage !
> Pour les rendre permanents, ajoutez-les dans `~/.bashrc` :
> ```bash
> echo "alias ll='ls -alF'" >> ~/.bashrc
> source ~/.bashrc
> ```

---

## 12. Permissions

Les permissions Linux controlent qui peut lire, ecrire et executer un fichier.

### Format symbolique

```
-rwxr-xr--  1 user group  4096 Jan 1 12:00 fichier.txt
 |\_/\_/\_/
 |  |  |  |
 |  |  |  +-- Autres (Other) : r-- (lecture seule)
 |  |  +----- Groupe (Group) : r-x (lecture + execution)
 |  +-------- Proprietaire (User) : rwx (lecture + ecriture + execution)
 +----------- Type (- = fichier, d = repertoire, l = lien)
```

### Format octal

Chaque permission a une valeur binaire (voir [[02 - Bases Numeriques]]) :

| Permission | Binaire | Octal |
|-----------|---------|-------|
| `---` | 000 | 0 |
| `--x` | 001 | 1 |
| `-w-` | 010 | 2 |
| `-wx` | 011 | 3 |
| `r--` | 100 | 4 |
| `r-x` | 101 | 5 |
| `rw-` | 110 | 6 |
| `rwx` | 111 | 7 |

```bash
chmod 755 script.sh     # rwxr-xr-x (proprietaire : tout, autres : lire+executer)
chmod 644 fichier.txt   # rw-r--r-- (proprietaire : lire+ecrire, autres : lire)
chmod u+x script.sh     # Ajoute l'execution pour le proprietaire
```

### Commandes de propriete

```bash
chown user fichier.txt          # Change le proprietaire
chgrp group fichier.txt         # Change le groupe
chown user:group fichier.txt    # Change les deux
```

---

## 13. La variable PATH

`PATH` contient la liste des repertoires ou le shell cherche les programmes.

```bash
echo $PATH
# /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

# Ajouter un repertoire au PATH
export PATH="$HOME/bin:$PATH"
```

> [!info] Comment le shell trouve une commande
> Quand tu tapes `ls`, le shell :
> 1. Verifie si c'est un alias
> 2. Verifie si c'est une fonction
> 3. Verifie si c'est un builtin (cd, echo, etc.)
> 4. Cherche dans chaque repertoire du PATH, de gauche a droite
> 5. Si introuvable : `command not found`

---

## 14. Processus : PID, fork, exec, wait

### PID et PPID

Chaque processus a un identifiant unique (**PID**) et connait celui de son parent (**PPID**).

```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    printf("Mon PID  : %u\n", getpid());
    printf("Mon PPID : %u\n", getppid());
    return (0);
}
```

```bash
echo $$      # PID du shell courant
```

### fork() : Creer un processus enfant

`fork()` cree une copie presque identique du processus courant.

```c
pid_t pid = fork();

if (pid == -1)
    perror("fork failed");
else if (pid == 0)
    printf("Je suis l'enfant (PID: %d)\n", getpid());
else
    printf("Je suis le parent (enfant PID: %d)\n", pid);
```

> [!info] Retour de fork()
> - Retourne **0** dans le processus **enfant**
> - Retourne le **PID de l'enfant** dans le processus **parent**
> - Retourne **-1** en cas d'erreur

### execve() : Remplacer le programme courant

`execve()` charge un nouveau programme dans le processus courant. Sur succes, il **ne retourne jamais** (l'ancien programme est remplace).

```c
char *argv[] = {"/bin/ls", "-l", "/usr/", NULL};

printf("Avant execve\n");
if (execve(argv[0], argv, NULL) == -1)
    perror("Erreur");
printf("Apres execve\n");  /* JAMAIS execute si execve reussit */
```

### wait() : Attendre la fin d'un enfant

```c
pid_t child_pid = fork();

if (child_pid == 0)
{
    /* Code de l'enfant */
    execve("/bin/ls", args, NULL);
}
else
{
    int status;
    wait(&status);  /* Le parent attend que l'enfant finisse */
    printf("L'enfant a termine\n");
}
```

### Le schema complet d'un Shell simple

```
 Shell (parent)
     |
     |--- fork() --> Enfant
     |                 |
     |                 |--- execve("/bin/ls", ...) --> Remplace par ls
     |                 |
     |                 |--- ls s'execute et se termine
     |
     |--- wait() --> Le parent attend
     |
     |--- Affiche le prompt "$" a nouveau
```

---

## 15. Le fichier /etc/passwd

Format (separe par `:`) :

```
nom:x:UID:GID:description:home:shell
```

```bash
# Exemple
root:x:0:0:root:/root:/bin/bash
julien:x:1000:1000:Julien:/home/julien:/bin/bash

# Afficher seulement les noms d'utilisateur
cut -d':' -f1 /etc/passwd
```

---

## 16. printf (Shell)

Plus fiable que `echo` pour les scripts serieux :

```bash
printf "Nom: %-10s | Age: %d\n" "Alice" 25
# Nom: Alice      | Age: 25

printf "%s\n" "Ligne 1" "Ligne 2" "Ligne 3"
```

| Specificateur | Type |
|--------------|------|
| `%s` | Chaine de caracteres (string) |
| `%d` | Nombre entier (decimal) |
| `%f` | Nombre a virgule (float) |
| `\n` | Nouvelle ligne |
| `\t` | Tabulation |

> [!tip] Pourquoi `printf` plutot que `echo` ?
> Le comportement de `echo` varie d'un systeme a l'autre.
> `printf` est **standardise** (POSIX) et offre un formatage precis.

---

## Gestion des processus

### Voir les processus

```bash
# Processus de l'utilisateur courant
ps                  # liste basique
ps aux              # TOUS les processus (format détaillé)
ps aux | grep nginx # chercher un processus précis

# En temps réel (comme le gestionnaire de tâches)
top                 # vue temps réel (q pour quitter)
htop                # version améliorée (installer : apt install htop)
```

| Colonne `ps aux` | Signification |
|-------------------|---------------|
| USER | Propriétaire du processus |
| PID | Process ID (identifiant unique) |
| %CPU | Utilisation CPU |
| %MEM | Utilisation mémoire |
| STAT | État (R=running, S=sleeping, Z=zombie, T=stopped) |
| COMMAND | La commande qui a lancé le processus |

### Foreground et Background

```bash
# Lancer en arrière-plan
sleep 100 &         # le & lance en background
[1] 12345           # → [numéro du job] PID

# Gérer les jobs
jobs                # lister les jobs du shell courant
fg %1               # ramener le job 1 au premier plan
bg %1               # reprendre le job 1 en arrière-plan
Ctrl+Z              # SUSPENDRE le processus au premier plan
Ctrl+C              # TUER le processus au premier plan
```

### Signaux et kill

```bash
# Envoyer un signal à un processus
kill PID             # envoie SIGTERM (demande polie d'arrêt)
kill -9 PID          # envoie SIGKILL (arrêt forcé, pas de nettoyage)
kill -STOP PID       # suspendre
kill -CONT PID       # reprendre
killall nom_prog     # tuer par nom de programme

# Signaux les plus courants
# SIGTERM (15) : arrêt propre (défaut de kill)
# SIGKILL (9)  : arrêt forcé (ne peut pas être ignoré)
# SIGSTOP (19) : suspendre
# SIGCONT (18) : reprendre
# SIGINT (2)   : interruption (Ctrl+C)
# SIGTSTP (20) : suspension (Ctrl+Z)
```

> [!warning] kill -9 en dernier recours
> Utilise d'abord `kill PID` (SIGTERM) qui permet au processus de se fermer proprement. `kill -9` ne permet aucun nettoyage (fichiers temporaires, connexions, sauvegardes).

### nohup et disown

```bash
# Lancer un processus qui survit à la fermeture du terminal
nohup ./mon_script.sh &     # continue même si tu fermes le terminal
disown %1                    # détacher un job existant du terminal

# La sortie de nohup est redirigée vers nohup.out par défaut
nohup ./script.sh > log.txt 2>&1 &  # rediriger vers un fichier spécifique
```

---

## 17. Exercices

> [!example] Exercice 1 : Scripts de 2 lignes
> Ecrire un script qui affiche "Hello, World" suivi d'un retour a la ligne :
> ```bash
> #!/bin/bash
> echo "Hello, World"
> ```

> [!example] Exercice 2 : Redirection
> Ecrire un script qui liste tous les fichiers `.c` du repertoire courant et enregistre le resultat dans `c_files.txt`, tout en l'affichant a l'ecran.
> Indice : utiliser `tee`.

> [!example] Exercice 3 : Pipe et filtres
> Ecrire une commande unique qui affiche les 5 utilisateurs avec le plus de processus, tries par nombre decroissant.
> Indice : `ps aux | awk '{print $1}' | sort | uniq -c | sort -rn | head -5`

> [!example] Exercice 4 : Variables et expansions
> Ecrire un script qui affiche : `hello <nom_utilisateur>` en utilisant `printf` et la variable `$USER`.
> ```bash
> #!/bin/bash
> printf "hello %s\n" "$USER"
> ```

> [!example] Exercice 5 : fork + exec + wait
> Ecrire un programme C qui execute `ls -l /tmp` dans 5 processus enfants differents. Chaque enfant doit etre cree par le meme parent. Attendre la fin de chaque enfant avant d'en creer un nouveau.

---

## 18. Liens

- [[01 - Archives et Compression]] - tar, gzip et autres outils
- [[01 - Introduction au C et Compilation]] - Compilation avec gcc
- [[02 - Bases Numeriques]] - Binaire/Octal pour les permissions chmod
- [[09 - File IO et Appels Systeme]] - open, read, write au niveau C
