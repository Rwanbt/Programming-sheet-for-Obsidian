# Shell Scripting (Bash)

> [!info] Contexte
> Un script shell est un fichier texte contenant des commandes qui sont executees sequentiellement par un interpreteur (comme `bash`). Les scripts permettent d'**automatiser** des taches repetitives, de creer des outils personnalises et de gerer des systemes.

---

## Table des matieres

1. [[#Qu'est-ce qu'un script shell ?]]
2. [[#Variables]]
3. [[#Quoting]]
4. [[#Conditions (if/then/else/fi)]]
5. [[#Boucles]]
6. [[#Case (switch)]]
7. [[#Fonctions]]
8. [[#Arithmetique]]
9. [[#Tableaux]]
10. [[#Manipulation de strings]]
11. [[#Redirection dans les scripts]]
12. [[#Exit codes]]
13. [[#Debugging]]
14. [[#Bonnes pratiques]]
15. [[#Scripts utiles exemples]]
16. [[#Exercices]]

---

## Qu'est-ce qu'un script shell ?

Un script shell est un **fichier texte** contenant une serie de commandes que le shell execute ligne par ligne.

### Creer et executer un script

```bash
# 1. Creer le fichier
touch mon_script.sh

# 2. Ecrire le contenu (shebang + commandes)
cat > mon_script.sh << 'EOF'
#!/bin/bash
echo "Hello, World!"
echo "Date : $(date)"
echo "Utilisateur : $(whoami)"
EOF

# 3. Rendre executable
chmod +x mon_script.sh

# 4. Executer
./mon_script.sh

# OU executer sans chmod
bash mon_script.sh
```

### Le shebang (#!)

```
#!/bin/bash      <-- Le SHEBANG

+--------------------------------------------------+
|  #!  : "magic number" qui indique au systeme     |
|        quel interpreteur utiliser                 |
|                                                   |
|  /bin/bash : chemin vers l'interpreteur bash      |
+--------------------------------------------------+

Alternatives courantes :
  #!/bin/bash       Bash specifique
  #!/bin/sh         Shell POSIX (plus portable)
  #!/usr/bin/env bash   Trouve bash dans le PATH
  #!/usr/bin/env python3  Pour un script Python
```

> [!warning] Important
> Le shebang doit etre la **toute premiere ligne** du fichier. Pas d'espace avant, pas de ligne vide avant. Sinon il est ignore.

### Cycle de vie d'un script

```
+---------------+     +---------------+     +---------------+
| Ecrire le     | --> | chmod +x      | --> | Executer      |
| script (.sh)  |     | (rendre       |     | ./script.sh   |
|               |     |  executable)  |     |               |
+---------------+     +---------------+     +---------------+
                                                    |
                                                    v
                                            +---------------+
                                            | Le shell lit  |
                                            | et execute    |
                                            | ligne par     |
                                            | ligne         |
                                            +---------------+
```

---

## Variables

### Declaration et lecture

```bash
#!/bin/bash

# Declaration : PAS D'ESPACES autour du =
NAME="Alice"
AGE=25
CITY="Paris"
FILE_PATH="/home/user/data.txt"

# ERREUR COURANTE :
# NAME = "Alice"    # ERREUR ! Bash interprete NAME comme une commande

# Lecture : $ devant le nom
echo "Nom : $NAME"
echo "Age : $AGE"
echo "Ville : ${CITY}"    # Accolades recommandees

# Accolades obligatoires pour lever l'ambiguite
PREFIX="pre"
echo "${PREFIX}fixe"     # Affiche : prefixe
echo "$PREFIXfixe"       # ERREUR : cherche la variable PREFIXfixe
```

> [!warning] PAS d'espaces autour du =
> ```bash
> # CORRECT
> NAME="value"
> 
> # INCORRECT (3 erreurs classiques)
> NAME = "value"     # bash cherche la commande "NAME"
> NAME ="value"      # idem
> NAME= "value"      # execute "value" avec NAME vide dans l'env
> ```

### Variables d'environnement

```bash
# Exporter une variable (visible par les processus enfants)
export PATH="$PATH:/opt/bin"
export EDITOR="vim"
export LANG="fr_FR.UTF-8"

# Variables d'environnement courantes
echo "HOME     : $HOME"
echo "USER     : $USER"
echo "PATH     : $PATH"
echo "PWD      : $PWD"
echo "SHELL    : $SHELL"
echo "HOSTNAME : $HOSTNAME"
```

### Variables speciales

| Variable | Description                             | Exemple                  |
| -------- | --------------------------------------- | ------------------------ |
| `$0`     | Nom du script                           | `./mon_script.sh`        |
| `$1`-`$9`| Arguments positionnels                  | Premier a neuvieme arg   |
| `${10}`  | Arguments > 9 (accolades obligatoires)  | Dixieme argument         |
| `$#`     | Nombre d'arguments                      | `3`                      |
| `$@`     | Tous les arguments (separement)         | `"arg1" "arg2" "arg3"`  |
| `$*`     | Tous les arguments (un seul mot)        | `"arg1 arg2 arg3"`      |
| `$?`     | Code de retour de la derniere commande  | `0` (succes)             |
| `$$`     | PID du script courant                   | `12345`                  |
| `$!`     | PID du dernier processus en background  | `12346`                  |

> [!example] Exemple avec les arguments
> ```bash
> #!/bin/bash
> # Fichier : args.sh
> 
> echo "Script     : $0"
> echo "Arg 1      : $1"
> echo "Arg 2      : $2"
> echo "Nb args    : $#"
> echo "Tous args  : $@"
> echo "Exit code  : $?"
> echo "PID        : $$"
> ```
> 
> ```bash
> $ ./args.sh hello world 42
> Script     : ./args.sh
> Arg 1      : hello
> Arg 2      : world
> Nb args    : 3
> Tous args  : hello world 42
> Exit code  : 0
> PID        : 54321
> ```

### Lire l'input utilisateur

```bash
#!/bin/bash

# Lire une valeur
echo -n "Entrez votre nom : "
read nom
echo "Bonjour, $nom !"

# Lire avec prompt integre
read -p "Entrez votre age : " age
echo "Vous avez $age ans."

# Lire en silence (mot de passe)
read -sp "Mot de passe : " password
echo ""
echo "Mot de passe recu (${#password} caracteres)."

# Lire avec timeout
read -t 5 -p "Reponse (5 sec) : " reponse

# Lire dans plusieurs variables
read -p "Prenom et nom : " prenom nom
echo "Prenom: $prenom, Nom: $nom"
```

---

## Quoting

Le quoting est un concept **fondamental** en bash. Il determine comment les caracteres speciaux sont interpretes.

### Les trois types de quoting

```bash
NAME="World"

# Simple quotes : AUCUNE interpretation
echo 'Hello $NAME'        # Affiche : Hello $NAME
echo 'Prix : $100'        # Affiche : Prix : $100
echo 'Pas de $(commande)' # Affiche : Pas de $(commande)

# Double quotes : expansion des variables et commandes
echo "Hello $NAME"         # Affiche : Hello World
echo "Date : $(date)"     # Affiche : Date : Tue Apr 14 ...
echo "Home : $HOME"       # Affiche : Home : /home/user

# Pas de quotes : expansion + word splitting + globbing
echo Hello $NAME           # Affiche : Hello World
echo *.txt                 # Affiche : file1.txt file2.txt ...
```

### Tableau comparatif

| Type            | Syntaxe      | Variables | Commandes | Globbing | Espaces |
| --------------- | ------------ | --------- | --------- | -------- | ------- |
| Simple quotes   | `'...'`      | NON       | NON       | NON      | Preserves |
| Double quotes   | `"..."`      | OUI       | OUI       | NON      | Preserves |
| Pas de quotes   |              | OUI       | OUI       | OUI      | Word split |
| Backslash       | `\`          | Echappe 1 caractere |  |          |         |

### Command substitution

```bash
# Ancienne syntaxe (backticks) - EVITER
files=`ls -1`

# Nouvelle syntaxe (recommandee) - $()
files=$(ls -1)
today=$(date +%Y-%m-%d)
num_files=$(ls | wc -l)
kernel=$(uname -r)

# Imbrication possible avec $() (impossible avec backticks)
result=$(echo "User: $(whoami) on $(hostname)")
```

> [!tip] Regle d'or
> **Toujours utiliser des double quotes** autour des variables : `"$var"`. Cela evite les problemes de word splitting et de globbing :
> ```bash
> file="my file.txt"
> 
> # ERREUR : word splitting
> cat $file       # bash essaie : cat my file.txt (2 fichiers !)
> 
> # CORRECT
> cat "$file"     # bash fait : cat "my file.txt" (1 fichier)
> ```

---

## Conditions (if/then/else/fi)

### Syntaxe de base

```bash
if condition; then
    commandes
elif autre_condition; then
    commandes
else
    commandes
fi
```

### test et [ ] (POSIX)

```bash
#!/bin/bash

age=25

# Comparaisons numeriques
if [ "$age" -eq 25 ]; then
    echo "Exactement 25 ans"
fi

if [ "$age" -gt 18 ]; then
    echo "Majeur"
fi
```

### Operateurs de comparaison

#### Nombres

| Operateur | Signification    | Exemple               |
| --------- | ---------------- | --------------------- |
| `-eq`     | Egal (equal)     | `[ "$a" -eq "$b" ]`  |
| `-ne`     | Different        | `[ "$a" -ne "$b" ]`  |
| `-lt`     | Inferieur (less) | `[ "$a" -lt "$b" ]`  |
| `-gt`     | Superieur (greater)| `[ "$a" -gt "$b" ]` |
| `-le`     | Inferieur ou egal| `[ "$a" -le "$b" ]`  |
| `-ge`     | Superieur ou egal| `[ "$a" -ge "$b" ]`  |

#### Chaines de caracteres

| Operateur | Signification          | Exemple                  |
| --------- | ---------------------- | ------------------------ |
| `=`       | Egal                   | `[ "$a" = "$b" ]`       |
| `!=`      | Different              | `[ "$a" != "$b" ]`      |
| `-z`      | Vide (zero length)     | `[ -z "$a" ]`           |
| `-n`      | Non vide               | `[ -n "$a" ]`           |
| `<`       | Inferieur (alpha)      | `[[ "$a" < "$b" ]]`     |
| `>`       | Superieur (alpha)      | `[[ "$a" > "$b" ]]`     |

#### Fichiers

| Operateur | Signification               | Exemple              |
| --------- | --------------------------- | -------------------- |
| `-f`      | Fichier regulier existe     | `[ -f "$file" ]`     |
| `-d`      | Repertoire existe           | `[ -d "$dir" ]`      |
| `-e`      | Existe (fichier ou rep)     | `[ -e "$path" ]`     |
| `-r`      | Lisible (readable)          | `[ -r "$file" ]`     |
| `-w`      | Modifiable (writable)       | `[ -w "$file" ]`     |
| `-x`      | Executable                  | `[ -x "$file" ]`     |
| `-s`      | Taille > 0                  | `[ -s "$file" ]`     |
| `-L`      | Lien symbolique             | `[ -L "$file" ]`     |

### [[ ]] - Version amelioree (Bash)

```bash
#!/bin/bash

name="Alice"
file="data.txt"

# Pattern matching avec ==
if [[ "$name" == A* ]]; then
    echo "Le nom commence par A"
fi

# Regex avec =~
if [[ "$name" =~ ^[A-Z][a-z]+$ ]]; then
    echo "Nom valide (majuscule + minuscules)"
fi

# Operateurs logiques directement dans [[ ]]
if [[ "$name" == "Alice" && -f "$file" ]]; then
    echo "Alice existe et le fichier aussi"
fi

# Pas besoin de quoter les variables dans [[ ]]
# (mais c'est quand meme recommande par habitude)
if [[ -z $name ]]; then
    echo "Nom vide"
fi
```

> [!info] Difference entre [ ] et [[ ]]
> ```
> +-------------------------------------------+
> |  [ ]  (test)     |  [[ ]]  (bash)        |
> |------------------|-----------------------|
> |  POSIX standard  |  Bash specifique      |
> |  Pas de &&, ||   |  &&, || supportes     |
> |  Pas de pattern  |  == avec glob/regex   |
> |  Quoter les vars |  Pas obligatoire      |
> |  = pour strings  |  == pour strings      |
> +-------------------------------------------+
> ```

### Operateurs logiques

```bash
# Avec [ ] : -a (AND) et -o (OR) - A EVITER (deprecated)
if [ "$a" -gt 0 -a "$a" -lt 100 ]; then
    echo "Entre 0 et 100"
fi

# Meilleure approche : && et ||
if [ "$a" -gt 0 ] && [ "$a" -lt 100 ]; then
    echo "Entre 0 et 100"
fi

# Avec [[ ]] : && et || directement
if [[ "$a" -gt 0 && "$a" -lt 100 ]]; then
    echo "Entre 0 et 100"
fi

# NOT
if ! [ -f "fichier.txt" ]; then
    echo "Le fichier n'existe pas"
fi

# Forme courte (sans if)
[ -f "fichier.txt" ] && echo "Existe" || echo "N'existe pas"
```

### Exemple complet

```bash
#!/bin/bash

read -p "Entrez un nombre : " num

if [[ -z "$num" ]]; then
    echo "Erreur : aucun nombre fourni"
    exit 1
elif [[ ! "$num" =~ ^-?[0-9]+$ ]]; then
    echo "Erreur : '$num' n'est pas un nombre"
    exit 1
elif [[ "$num" -gt 0 ]]; then
    echo "$num est positif"
elif [[ "$num" -lt 0 ]]; then
    echo "$num est negatif"
else
    echo "$num est zero"
fi
```

---

## Boucles

### for ... in (liste)

```bash
#!/bin/bash

# Iterer sur une liste de mots
for fruit in pomme banane orange; do
    echo "Fruit : $fruit"
done

# Iterer sur des fichiers
for file in *.txt; do
    echo "Traitement de : $file"
    wc -l "$file"
done

# Iterer sur une sequence
for i in {1..10}; do
    echo "Numero : $i"
done

# Sequence avec pas
for i in {0..100..5}; do
    echo "Compteur : $i"
done

# Iterer sur la sortie d'une commande
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "Utilisateur : $user"
done
```

### for C-style

```bash
#!/bin/bash

# Boucle C-style
for ((i = 0; i < 10; i++)); do
    echo "i = $i"
done

# Double variable
for ((i = 0, j = 10; i < j; i++, j--)); do
    echo "i=$i, j=$j"
done

# Boucle infinie
for ((;;)); do
    echo "Infini..."
    sleep 1
    break  # Sortir de la boucle
done
```

### while

```bash
#!/bin/bash

# While classique
count=0
while [[ "$count" -lt 5 ]]; do
    echo "Count : $count"
    ((count++))
done

# Lire un fichier ligne par ligne
while IFS= read -r line; do
    echo "Ligne : $line"
done < fichier.txt

# While avec pipe (attention : sous-shell !)
cat fichier.txt | while read -r line; do
    echo "$line"
done

# Boucle infinie avec while
while true; do
    read -p "Commande (q pour quitter) : " cmd
    [[ "$cmd" == "q" ]] && break
    echo "Vous avez tape : $cmd"
done
```

### until

```bash
#!/bin/bash

# until : continue TANT QUE la condition est FAUSSE
count=0
until [[ "$count" -ge 5 ]]; do
    echo "Count : $count"
    ((count++))
done

# Attendre qu'un fichier existe
until [[ -f "data_ready.flag" ]]; do
    echo "En attente du fichier..."
    sleep 2
done
echo "Fichier recu !"
```

### break et continue

```bash
#!/bin/bash

# break : sortir de la boucle
for i in {1..100}; do
    if [[ "$i" -eq 5 ]]; then
        echo "Arret a $i"
        break
    fi
    echo "$i"
done

# continue : passer a l'iteration suivante
for i in {1..10}; do
    if [[ $((i % 2)) -eq 0 ]]; then
        continue  # Sauter les nombres pairs
    fi
    echo "Impair : $i"
done
```

### Diagramme des boucles

```
while condition :             until condition :
                              
+-------+                    +-------+
| debut |                    | debut |
+---+---+                    +---+---+
    |                            |
    v                            v
+--------+  VRAI   +------+  +--------+  FAUX    +------+
| cond ? |-------->| body |  | cond ? |--------->| body |
+--------+         +--+---+  +--------+          +--+---+
    |                  |          |                   |
    | FAUX             |          | VRAI              |
    v           retour a cond     v            retour a cond
+-------+                    +-------+
|  fin  |                    |  fin  |
+-------+                    +-------+
```

---

## Case (switch)

### Syntaxe

```bash
case "$variable" in
    pattern1)
        commandes
        ;;
    pattern2 | pattern3)
        commandes
        ;;
    *)
        commandes par defaut
        ;;
esac
```

### Exemples

```bash
#!/bin/bash

read -p "Entrez une commande : " cmd

case "$cmd" in
    start)
        echo "Demarrage du service..."
        ;;
    stop)
        echo "Arret du service..."
        ;;
    restart)
        echo "Redemarrage..."
        ;;
    status)
        echo "Service actif."
        ;;
    *)
        echo "Usage : $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
```

### Patterns avances

```bash
#!/bin/bash

file="$1"

case "$file" in
    *.tar.gz | *.tgz)
        echo "Archive tar gzip"
        tar xzf "$file"
        ;;
    *.tar.bz2)
        echo "Archive tar bzip2"
        tar xjf "$file"
        ;;
    *.zip)
        echo "Archive zip"
        unzip "$file"
        ;;
    *.jpg | *.jpeg | *.png | *.gif)
        echo "Image"
        ;;
    [0-9]*)
        echo "Commence par un chiffre"
        ;;
    *)
        echo "Type de fichier inconnu"
        ;;
esac
```

---

## Fonctions

### Declaration et appel

```bash
#!/bin/bash

# Methode 1 : syntaxe standard
greet() {
    echo "Bonjour, $1 !"
}

# Methode 2 : avec le mot-cle function (bash uniquement)
function greet2 {
    echo "Salut, $1 !"
}

# Appel (sans parentheses !)
greet "Alice"
greet2 "Bob"
```

### Parametres et retour

```bash
#!/bin/bash

# Les arguments sont $1, $2, etc. (comme un script)
add() {
    local a="$1"      # variable locale
    local b="$2"
    local sum=$((a + b))
    echo "$sum"        # "retourner" une valeur via echo
}

# Capturer le resultat
result=$(add 5 3)
echo "5 + 3 = $result"

# return : code de retour (0-255), PAS une valeur !
is_even() {
    if [[ $(($1 % 2)) -eq 0 ]]; then
        return 0    # succes = pair
    else
        return 1    # echec = impair
    fi
}

if is_even 42; then
    echo "42 est pair"
fi
```

> [!warning] return vs echo
> ```
> +---------------------------------------------------+
> |  return N  : code de retour (0-255)               |
> |              Teste avec $? ou if func; then        |
> |              PAS pour retourner une valeur !        |
> |                                                     |
> |  echo "val" : envoyer une valeur sur stdout        |
> |               Capturer avec result=$(func)          |
> |               C'est la methode pour "retourner"     |
> |               une valeur                            |
> +---------------------------------------------------+
> ```

### Variables locales

```bash
#!/bin/bash

GLOBAL="je suis global"

my_func() {
    local LOCAL="je suis local"
    GLOBAL="modifie dans la fonction"
    echo "Dans la fonction : LOCAL=$LOCAL"
}

my_func
echo "Apres la fonction : GLOBAL=$GLOBAL"
echo "Apres la fonction : LOCAL=$LOCAL"    # Vide !
```

```
Sortie :
  Dans la fonction : LOCAL=je suis local
  Apres la fonction : GLOBAL=modifie dans la fonction
  Apres la fonction : LOCAL=
```

> [!tip] Bonne pratique
> Toujours utiliser `local` pour les variables dans les fonctions. Sans `local`, la variable est **globale** et peut ecraser d'autres variables du script.

### Exemple complet : fonction utilitaire

```bash
#!/bin/bash

# Fonction de log avec couleurs
log_info()    { echo -e "\033[34m[INFO]\033[0m $*"; }
log_success() { echo -e "\033[32m[OK]\033[0m $*"; }
log_warning() { echo -e "\033[33m[WARN]\033[0m $*"; }
log_error()   { echo -e "\033[31m[ERROR]\033[0m $*" >&2; }

# Fonction qui verifie un prerequis
check_command() {
    local cmd="$1"
    if command -v "$cmd" &>/dev/null; then
        log_success "$cmd trouve : $(command -v "$cmd")"
        return 0
    else
        log_error "$cmd non trouve !"
        return 1
    fi
}

# Utilisation
log_info "Verification des prerequis..."
check_command "gcc"
check_command "make"
check_command "python3"
```

---

## Arithmetique

### $(( )) - Expansion arithmetique

```bash
#!/bin/bash

a=10
b=3

echo "Addition       : $((a + b))"      # 13
echo "Soustraction   : $((a - b))"      # 7
echo "Multiplication : $((a * b))"      # 30
echo "Division       : $((a / b))"      # 3 (division entiere !)
echo "Modulo         : $((a % b))"      # 1
echo "Puissance      : $((a ** b))"     # 1000

# Increment / Decrement
((a++))
echo "Apres a++ : $a"                    # 11

((a--))
echo "Apres a-- : $a"                    # 10

# Affectation composee
((a += 5))
echo "Apres a+=5 : $a"                  # 15
```

### let

```bash
let "a = 5 + 3"
let "b = a * 2"
let "a++"
echo "a=$a, b=$b"    # a=9, b=16
```

### expr (ancienne methode)

```bash
# Espaces OBLIGATOIRES autour des operateurs
result=$(expr 5 + 3)
result=$(expr 10 \* 2)    # * doit etre echappe !
echo "$result"
```

> [!info] Preference
> Utilisez `$(( ))` en priorite. C'est la methode la plus moderne, la plus rapide et la plus lisible. `let` est acceptable. `expr` est obsolete.

---

## Tableaux

### Declaration et acces

```bash
#!/bin/bash

# Declaration
fruits=("pomme" "banane" "orange" "kiwi")

# Acces par index (commence a 0)
echo "Premier  : ${fruits[0]}"     # pomme
echo "Deuxieme : ${fruits[1]}"     # banane

# Tous les elements
echo "Tous : ${fruits[@]}"          # pomme banane orange kiwi

# Nombre d'elements
echo "Taille : ${#fruits[@]}"       # 4

# Longueur d'un element
echo "Longueur de fruits[0] : ${#fruits[0]}"  # 5

# Ajouter un element
fruits+=("mangue")
echo "Apres ajout : ${fruits[@]}"

# Modifier un element
fruits[1]="fraise"
echo "Apres modif : ${fruits[@]}"
```

### Iterer sur un tableau

```bash
#!/bin/bash

files=("main.c" "utils.c" "parser.c" "header.h")

# Par valeur
for file in "${files[@]}"; do
    echo "Fichier : $file"
done

# Par index
for i in "${!files[@]}"; do
    echo "files[$i] = ${files[$i]}"
done
```

### Slicing

```bash
#!/bin/bash

arr=(a b c d e f g h)

# Sous-tableau : ${arr[@]:debut:longueur}
echo "${arr[@]:2:3}"     # c d e  (3 elements a partir de l'index 2)
echo "${arr[@]:4}"       # e f g h  (tout a partir de l'index 4)
```

### Tableaux associatifs (Bash 4+)

```bash
#!/bin/bash

declare -A config

config[host]="localhost"
config[port]="8080"
config[user]="admin"
config[debug]="true"

echo "Host : ${config[host]}"
echo "Port : ${config[port]}"

# Toutes les cles
echo "Cles : ${!config[@]}"

# Toutes les valeurs
echo "Valeurs : ${config[@]}"

# Iterer
for key in "${!config[@]}"; do
    echo "$key = ${config[$key]}"
done
```

---

## Manipulation de strings

### Operations courantes

```bash
#!/bin/bash

str="Hello, World!"

# Longueur
echo "Longueur : ${#str}"               # 13

# Substring : ${var:offset:length}
echo "Sub 0-5  : ${str:0:5}"            # Hello
echo "Sub 7    : ${str:7}"              # World!

# Remplacement : ${var/old/new}
echo "Replace  : ${str/World/Bash}"      # Hello, Bash!

# Remplacement global : ${var//old/new}
str2="aabbaabb"
echo "Global   : ${str2//aa/XX}"         # XXbbXXbb

# Suppression par pattern depuis le debut
path="/home/user/documents/file.tar.gz"
echo "Fichier  : ${path##*/}"            # file.tar.gz  (supprime le plus long match de */)
echo "Dir      : ${path%/*}"             # /home/user/documents  (supprime le plus court match de /*)

# Extension
file="archive.tar.gz"
echo "Sans ext : ${file%%.*}"            # archive  (supprime le plus long match de .*)
echo "Sans .gz : ${file%.*}"             # archive.tar  (supprime le plus court match de .*)
echo "Ext      : ${file##*.}"            # gz  (supprime le plus long match de *.)
```

### Tableau recapitulatif

```
+----------------------------------------------------------+
|  Syntaxe              |  Description                      |
|-----------------------|-----------------------------------|
|  ${#var}              |  Longueur                         |
|  ${var:offset:len}    |  Substring                        |
|  ${var/old/new}       |  Remplacer (premier)              |
|  ${var//old/new}      |  Remplacer (tous)                 |
|  ${var#pattern}       |  Supprimer debut (court match)    |
|  ${var##pattern}      |  Supprimer debut (long match)     |
|  ${var%pattern}       |  Supprimer fin (court match)      |
|  ${var%%pattern}      |  Supprimer fin (long match)       |
|  ${var:-default}      |  Valeur par defaut si vide        |
|  ${var:=default}      |  Assigner defaut si vide          |
|  ${var:+alt}          |  Valeur alt si defini             |
|  ${var:?error}        |  Erreur si vide                   |
+----------------------------------------------------------+
```

> [!example] Exemple pratique : extraire des parties d'un chemin
> ```bash
> filepath="/home/user/project/src/main.c"
> 
> dir="${filepath%/*}"          # /home/user/project/src
> filename="${filepath##*/}"    # main.c
> name="${filename%.*}"         # main
> ext="${filename##*.}"         # c
> 
> echo "Repertoire : $dir"
> echo "Fichier    : $filename"
> echo "Nom        : $name"
> echo "Extension  : $ext"
> ```

---

## Redirection dans les scripts

### Rappel des redirections

```bash
# stdout vers fichier
echo "Hello" > fichier.txt       # Ecrase
echo "World" >> fichier.txt      # Ajoute

# stderr vers fichier
commande 2> erreurs.log

# stdout ET stderr vers le meme fichier
commande > output.log 2>&1
commande &> output.log           # Raccourci bash

# Rediriger vers /dev/null (ignorer)
commande > /dev/null 2>&1        # Silence total
commande &> /dev/null            # Raccourci
```

### Here documents (heredoc)

```bash
#!/bin/bash

# Here document : input multi-lignes
cat << EOF
Bonjour $USER,
Nous sommes le $(date +%A).
Votre repertoire est $HOME.
EOF

# Avec quotes : PAS d'expansion
cat << 'EOF'
Ceci est litteral : $USER $(date)
Pas d'expansion ici.
EOF

# Here string
grep "pattern" <<< "texte a chercher avec pattern dedans"
```

### Redirection dans les fonctions

```bash
#!/bin/bash

# Rediriger toute une fonction
generate_report() {
    echo "=== RAPPORT ==="
    echo "Date : $(date)"
    echo "Systeme : $(uname -a)"
    echo "Disque :"
    df -h /
    echo "=== FIN ==="
}

# Sauvegarder dans un fichier
generate_report > rapport.txt

# Ou avec tee (afficher ET sauvegarder)
generate_report | tee rapport.txt
```

---

## Exit codes

### Convention

```
+------------------------------------------+
|  Code  |  Signification                  |
|--------|---------------------------------|
|  0     |  Succes                         |
|  1     |  Erreur generique               |
|  2     |  Mauvaise utilisation (usage)   |
|  126   |  Commande non executable        |
|  127   |  Commande non trouvee           |
|  128+N |  Termine par signal N           |
|  130   |  Ctrl+C (SIGINT = 2)            |
+------------------------------------------+
```

### Utilisation dans les scripts

```bash
#!/bin/bash

# Verifier les arguments
if [[ $# -lt 1 ]]; then
    echo "Usage : $0 <fichier>" >&2
    exit 1
fi

# Verifier qu'un fichier existe
if [[ ! -f "$1" ]]; then
    echo "Erreur : '$1' n'est pas un fichier" >&2
    exit 2
fi

# Traitement...
echo "Traitement de $1..."

# Succes
exit 0
```

### Verifier le code de retour

```bash
#!/bin/bash

# Methode 1 : $?
gcc -o prog main.c
if [[ $? -ne 0 ]]; then
    echo "Erreur de compilation !"
    exit 1
fi

# Methode 2 : directement dans le if
if gcc -o prog main.c; then
    echo "Compilation reussie"
else
    echo "Echec de compilation"
    exit 1
fi

# Methode 3 : set -e (arreter sur erreur)
set -e
gcc -o prog main.c    # Si echec, le script s'arrete ici
echo "Compilation reussie"
```

---

## Debugging

### Options de debug

```bash
#!/bin/bash

# set -x : afficher chaque commande avant execution (trace)
set -x
echo "Hello"
# + echo Hello        <-- trace affichee
# Hello               <-- sortie normale

# set -e : arreter le script a la premiere erreur
set -e
false            # Le script s'arrete ici !
echo "Jamais atteint"

# set -u : erreur si variable non definie
set -u
echo "$VARIABLE_INEXISTANTE"   # ERREUR !

# set -o pipefail : echec si UNE commande du pipe echoue
set -o pipefail
false | true     # Sans pipefail : succes. Avec : echec.

# Combiner (ligne standard en debut de script)
set -euo pipefail
```

### Debugger depuis la ligne de commande

```bash
# Executer en mode trace
bash -x mon_script.sh

# Syntaxe check (ne pas executer)
bash -n mon_script.sh

# Verbose (affiche chaque ligne lue)
bash -v mon_script.sh
```

### Debug conditionnel dans le script

```bash
#!/bin/bash

# Activer/desactiver le debug avec une variable
DEBUG=${DEBUG:-0}

debug() {
    if [[ "$DEBUG" -eq 1 ]]; then
        echo "[DEBUG] $*" >&2
    fi
}

debug "Demarrage du script"
debug "Arguments : $@"

# Utilisation :
# DEBUG=1 ./mon_script.sh arg1 arg2
```

> [!tip] Astuce de debug
> Vous pouvez activer `set -x` seulement pour une partie du script :
> ```bash
> echo "Partie normale"
> 
> set -x   # Activer le trace
> # ... code a debugger ...
> set +x   # Desactiver le trace
> 
> echo "Retour a la normale"
> ```

---

## Bonnes pratiques

### Le header standard

```bash
#!/bin/bash
#
# Description : Ce script fait X, Y, Z
# Usage       : ./script.sh [options] <arguments>
# Auteur      : Votre nom
# Date        : 2026-04-14
#

set -euo pipefail
```

### Regles essentielles

> [!warning] Les 10 commandements du shell scripting
> 1. **Toujours** le shebang en premiere ligne (`#!/bin/bash`)
> 2. **Toujours** `set -euo pipefail` en debut de script
> 3. **Toujours** quoter les variables : `"$var"`, pas `$var`
> 4. **Toujours** utiliser `local` dans les fonctions
> 5. **Toujours** verifier les arguments et inputs
> 6. **Toujours** commenter le code (surtout le "pourquoi")
> 7. **Toujours** utiliser `$()` au lieu des backticks
> 8. **Eviter** les noms de variables en minuscules (conflit avec env)
> 9. **Eviter** `eval` (faille de securite)
> 10. **Tester** le script avec `bash -n` et `shellcheck`

### ShellCheck

```bash
# Installer shellcheck
sudo apt install shellcheck

# Verifier un script
shellcheck mon_script.sh

# Exemple de sortie :
# In script.sh line 5:
# echo $var
#      ^---^ SC2086: Double quote to prevent globbing and word splitting.
```

### Template de script

```bash
#!/bin/bash
#
# Description breve du script
#

set -euo pipefail

# === Configuration ===
readonly SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"

# === Fonctions ===
usage() {
    cat << EOF
Usage : $SCRIPT_NAME [options] <arguments>

Options :
    -h, --help      Afficher cette aide
    -v, --verbose   Mode verbeux
    -d, --dry-run   Simuler sans executer

EOF
    exit 0
}

log_info()  { echo "[INFO]  $*"; }
log_error() { echo "[ERROR] $*" >&2; }
die()       { log_error "$*"; exit 1; }

# === Arguments ===
VERBOSE=0
DRY_RUN=0

while [[ $# -gt 0 ]]; do
    case "$1" in
        -h|--help)    usage ;;
        -v|--verbose) VERBOSE=1; shift ;;
        -d|--dry-run) DRY_RUN=1; shift ;;
        --)           shift; break ;;
        -*)           die "Option inconnue : $1" ;;
        *)            break ;;
    esac
done

# === Main ===
main() {
    log_info "Demarrage de $SCRIPT_NAME"
    
    # Votre code ici
    
    log_info "Termine."
}

main "$@"
```

---

## Scripts utiles exemples

### 1. Backup automatique

```bash
#!/bin/bash
set -euo pipefail

# Configuration
SRC_DIR="$HOME/projets"
BACKUP_DIR="$HOME/backups"
DATE=$(date +%Y%m%d_%H%M%S)
ARCHIVE="backup_${DATE}.tar.gz"

# Creer le repertoire de backup
mkdir -p "$BACKUP_DIR"

# Creer l'archive
echo "Creation du backup : $ARCHIVE"
tar czf "${BACKUP_DIR}/${ARCHIVE}" -C "$SRC_DIR" .

# Garder seulement les 5 derniers backups
cd "$BACKUP_DIR"
ls -t backup_*.tar.gz | tail -n +6 | xargs -r rm -f

echo "Backup termine : ${BACKUP_DIR}/${ARCHIVE}"
echo "Taille : $(du -h "${BACKUP_DIR}/${ARCHIVE}" | cut -f1)"
```

### 2. Renommer des fichiers en masse

```bash
#!/bin/bash
set -euo pipefail

# Renommer tous les .jpeg en .jpg
for file in *.jpeg; do
    [[ -f "$file" ]] || continue
    new_name="${file%.jpeg}.jpg"
    echo "Renommage : $file -> $new_name"
    mv "$file" "$new_name"
done

echo "Termine."
```

### 3. Verifier l'espace disque

```bash
#!/bin/bash
set -euo pipefail

THRESHOLD=80

echo "=== Verification de l'espace disque ==="
echo ""

while IFS= read -r line; do
    usage=$(echo "$line" | awk '{print $5}' | tr -d '%')
    mount=$(echo "$line" | awk '{print $6}')
    
    if [[ "$usage" -ge "$THRESHOLD" ]]; then
        echo "[ALERTE] $mount : ${usage}% utilise (seuil: ${THRESHOLD}%)"
    else
        echo "[OK]     $mount : ${usage}% utilise"
    fi
done < <(df -h | tail -n +2)
```

### 4. Compilation automatique

```bash
#!/bin/bash
set -euo pipefail

# Surveiller les fichiers .c et recompiler automatiquement
SRC_DIR="${1:-.}"
LAST_HASH=""

echo "Surveillance de $SRC_DIR (Ctrl+C pour arreter)"

while true; do
    CURRENT_HASH=$(find "$SRC_DIR" -name "*.c" -o -name "*.h" | \
                   xargs md5sum 2>/dev/null | md5sum)
    
    if [[ "$CURRENT_HASH" != "$LAST_HASH" ]]; then
        echo ""
        echo "--- Modification detectee ($(date +%H:%M:%S)) ---"
        if make -C "$SRC_DIR" 2>&1; then
            echo "--- Compilation reussie ---"
        else
            echo "--- Erreur de compilation ---"
        fi
        LAST_HASH="$CURRENT_HASH"
    fi
    
    sleep 2
done
```

---

## Exercices

### Exercice 1 : Hello script

> [!example] Exercice
> Ecrivez un script qui prend un nom en argument et affiche "Bonjour, [nom] !". Si aucun argument n'est fourni, afficher un message d'erreur et quitter avec le code 1.
>
> **Solution :**
> ```bash
> #!/bin/bash
> set -euo pipefail
> 
> if [[ $# -lt 1 ]]; then
>     echo "Usage : $0 <nom>" >&2
>     exit 1
> fi
> 
> echo "Bonjour, $1 !"
> ```

### Exercice 2 : Pair ou impair

> [!example] Exercice
> Ecrivez un script qui prend un nombre en argument et affiche s'il est pair ou impair.
>
> **Solution :**
> ```bash
> #!/bin/bash
> set -euo pipefail
> 
> if [[ $# -lt 1 ]]; then
>     echo "Usage : $0 <nombre>" >&2
>     exit 1
> fi
> 
> if [[ $(($1 % 2)) -eq 0 ]]; then
>     echo "$1 est pair"
> else
>     echo "$1 est impair"
> fi
> ```

### Exercice 3 : Compter les fichiers

> [!example] Exercice
> Ecrivez un script qui compte le nombre de fichiers et de repertoires dans un chemin donne (argument) ou le repertoire courant par defaut.
>
> **Solution :**
> ```bash
> #!/bin/bash
> set -euo pipefail
> 
> dir="${1:-.}"
> 
> if [[ ! -d "$dir" ]]; then
>     echo "Erreur : '$dir' n'est pas un repertoire" >&2
>     exit 1
> fi
> 
> files=0
> dirs=0
> 
> for item in "$dir"/*; do
>     if [[ -f "$item" ]]; then
>         ((files++))
>     elif [[ -d "$item" ]]; then
>         ((dirs++))
>     fi
> done
> 
> echo "Repertoire : $dir"
> echo "Fichiers   : $files"
> echo "Repertoires: $dirs"
> ```

### Exercice 4 : FizzBuzz

> [!example] Exercice
> Ecrivez le classique FizzBuzz en bash (1 a 100) : "Fizz" si multiple de 3, "Buzz" si multiple de 5, "FizzBuzz" si multiple des deux, sinon le nombre.
>
> **Solution :**
> ```bash
> #!/bin/bash
> 
> for ((i = 1; i <= 100; i++)); do
>     output=""
>     [[ $((i % 3)) -eq 0 ]] && output+="Fizz"
>     [[ $((i % 5)) -eq 0 ]] && output+="Buzz"
>     echo "${output:-$i}"
> done
> ```

### Exercice 5 : Menu interactif

> [!example] Exercice
> Ecrivez un script avec un menu interactif qui propose :
> 1. Afficher la date
> 2. Afficher l'espace disque
> 3. Afficher les utilisateurs connectes
> 4. Quitter
>
> **Solution :**
> ```bash
> #!/bin/bash
> 
> while true; do
>     echo ""
>     echo "=== MENU ==="
>     echo "1) Date"
>     echo "2) Espace disque"
>     echo "3) Utilisateurs connectes"
>     echo "4) Quitter"
>     echo ""
>     read -p "Choix : " choice
> 
>     case "$choice" in
>         1) date ;;
>         2) df -h ;;
>         3) who ;;
>         4) echo "Au revoir !"; exit 0 ;;
>         *) echo "Choix invalide" ;;
>     esac
> done
> ```

### Exercice 6 : Fonction de recherche

> [!example] Exercice
> Ecrivez une fonction `search_files` qui prend un pattern et un repertoire en arguments, et affiche tous les fichiers correspondants avec leur taille.
>
> **Solution :**
> ```bash
> #!/bin/bash
> set -euo pipefail
> 
> search_files() {
>     local pattern="${1:?Usage: search_files <pattern> [dir]}"
>     local dir="${2:-.}"
> 
>     if [[ ! -d "$dir" ]]; then
>         echo "Erreur : '$dir' n'existe pas" >&2
>         return 1
>     fi
> 
>     local count=0
>     while IFS= read -r -d '' file; do
>         local size
>         size=$(du -h "$file" | cut -f1)
>         echo "$size  $file"
>         ((count++))
>     done < <(find "$dir" -name "$pattern" -type f -print0)
> 
>     echo "--- $count fichier(s) trouve(s) ---"
> }
> 
> search_files "*.sh" "$HOME"
> ```

---

## Liens

- [[01 - Shell et Commandes Linux]] : les commandes de base du shell
- [[02 - Les Terminaux Guide Complet]] : comprendre les terminaux
- [[01 - Introduction au C et Compilation]] : compiler du C depuis un script

---

> [!tip] Resume
> Le shell scripting est un outil **indispensable** pour tout developpeur. Retenez : **shebang** en premiere ligne, `set -euo pipefail` pour la robustesse, **toujours quoter** les variables, utiliser `local` dans les fonctions, et verifier les inputs. Commencez par des scripts simples et complexifiez progressivement. Utilisez `shellcheck` pour verifier votre code.
