# Makefiles

> [!info] Contexte
> `make` est un outil d'**automatisation de la compilation**. Au lieu de retaper `gcc -Wall -Werror -Wextra -o prog main.c utils.c helper.c` a chaque modification, un Makefile definit les regles de compilation une fois pour toutes.

---

## Table des matieres

1. [[#Qu'est-ce que Make ?]]
2. [[#Pourquoi utiliser Make ?]]
3. [[#Anatomie d'un Makefile]]
4. [[#Premier Makefile]]
5. [[#Variables]]
6. [[#Variables automatiques]]
7. [[#Regles implicites et Pattern rules]]
8. [[#Cibles speciales]]
9. [[#Makefile Holberton complet]]
10. [[#Fonctions Make]]
11. [[#Dependances de headers]]
12. [[#Makefile multi-repertoires]]
13. [[#Debugging un Makefile]]
14. [[#Erreurs classiques]]
15. [[#Exercices]]

---

## Qu'est-ce que Make ?

`make` est un utilitaire qui **automatise la construction** de programmes a partir de fichiers sources. Il lit un fichier appele `Makefile` (ou `makefile`) qui decrit **comment** construire le programme.

```
+------------------+     +------------------+     +------------------+
|    Makefile      | --> |      make        | --> |  Programme       |
|  (regles de      |     |  (lit les regles |     |  compile         |
|   construction)  |     |   et execute)    |     |  et pret         |
+------------------+     +------------------+     +------------------+
```

### Fonctionnement de base

```bash
# make lit le fichier Makefile dans le repertoire courant
make

# make avec une cible specifique
make clean
make all
make re

# make avec un fichier specifique
make -f MonMakefile
```

---

## Pourquoi utiliser Make ?

### Le probleme sans Make

```bash
# A chaque modification, il faut retaper TOUT :
gcc -Wall -Werror -Wextra -pedantic -std=gnu89 -c main.c -o main.o
gcc -Wall -Werror -Wextra -pedantic -std=gnu89 -c utils.c -o utils.o
gcc -Wall -Werror -Wextra -pedantic -std=gnu89 -c parser.c -o parser.o
gcc -Wall -Werror -Wextra -pedantic -std=gnu89 -c display.c -o display.o
gcc -o prog main.o utils.o parser.o display.o

# Et si on modifie SEULEMENT utils.c, il faut quand meme tout recompiler ?
# NON ! C'est la que Make intervient.
```

### La compilation incrementale

```
Modification de utils.c uniquement :

  main.c   -->  main.o    (PAS recompile, inchange)
  utils.c  -->  utils.o   (RECOMPILE, modifie)      <-- seulement ca !
  parser.c -->  parser.o  (PAS recompile, inchange)

  Puis re-linkage : main.o + utils.o + parser.o --> prog
```

> [!tip] Avantage cle
> Make compare les **dates de modification** des fichiers. Si un `.c` est plus recent que son `.o`, seul ce fichier est recompile. Sur un gros projet, cela fait gagner **enormement** de temps.

### Avantages de Make

| Avantage                | Description                                        |
| ----------------------- | -------------------------------------------------- |
| Compilation incrementale| Ne recompile que ce qui a change                   |
| Automatisation          | Une seule commande `make` suffit                   |
| Portabilite             | Meme commande sur tous les systemes POSIX          |
| Documentation           | Le Makefile documente comment construire le projet |
| Coherence               | Tous les developpeurs compilent de la meme facon   |

---

## Anatomie d'un Makefile

### Structure d'une regle

```makefile
cible: dependances
	recette    # <-- C'est un TAB, pas des espaces !
```

```
+----------------------------------------------------+
|  cible :         Le fichier a produire              |
|  dependances :   Les fichiers necessaires           |
|  recette :       Les commandes pour produire        |
|                  la cible a partir des dependances   |
+----------------------------------------------------+
```

> [!warning] Le piege du TAB
> La recette (les commandes) **DOIT** commencer par un caractere **TAB** (pas des espaces !). C'est l'erreur la plus courante avec les Makefiles.
> 
> ```
> Makefile:4: *** missing separator.  Stop.
> ```
> 
> Si vous voyez cette erreur, verifiez que vous avez bien un **TAB** et non des espaces.

### Exemple visuel

```makefile
# Cible "prog" depend de "main.o" et "utils.o"
# Pour la construire, on execute la commande gcc
prog: main.o utils.o
	gcc -o prog main.o utils.o

# Cible "main.o" depend de "main.c"
main.o: main.c
	gcc -Wall -c main.c -o main.o

# Cible "utils.o" depend de "utils.c"
utils.o: utils.c
	gcc -Wall -c utils.c -o utils.o
```

### Arbre de dependances

```
         prog
        /    \
    main.o   utils.o
      |        |
    main.c   utils.c
```

### Comment Make raisonne

```
make prog
  |
  +--> prog depend de main.o et utils.o
  |      |
  |      +--> main.o depend de main.c
  |      |      main.c existe ? OUI
  |      |      main.o existe ? NON --> compiler main.c
  |      |
  |      +--> utils.o depend de utils.c
  |             utils.c existe ? OUI
  |             utils.o existe ? NON --> compiler utils.c
  |
  +--> main.o et utils.o sont prets --> linker prog
```

---

## Premier Makefile

### Version minimaliste

```makefile
prog: main.c utils.c
	gcc -Wall -Werror -Wextra -o prog main.c utils.c
```

```bash
# Compiler
make

# Ou explicitement
make prog
```

> [!warning] Limitation
> Cette version recompile **TOUS** les fichiers a chaque fois, meme si un seul a change. On perd l'avantage de la compilation incrementale.

### Version avec compilation separee

```makefile
prog: main.o utils.o
	gcc -o prog main.o utils.o

main.o: main.c
	gcc -Wall -Werror -Wextra -c main.c -o main.o

utils.o: utils.c
	gcc -Wall -Werror -Wextra -c utils.c -o utils.o

clean:
	rm -f main.o utils.o prog
```

> [!tip] Compilation separee
> Le flag `-c` dit a GCC de compiler **sans linker** (produit un `.o`). Ensuite, on lie tous les `.o` ensemble. C'est la cle de la compilation incrementale.

---

## Variables

### Variables utilisateur

```makefile
# Convention : variables en MAJUSCULES
CC = gcc
CFLAGS = -Wall -Werror -Wextra -pedantic -std=gnu89
NAME = prog
SRC = main.c utils.c parser.c
OBJ = main.o utils.o parser.o

# Utilisation avec $(VARIABLE) ou ${VARIABLE}
$(NAME): $(OBJ)
	$(CC) $(CFLAGS) -o $(NAME) $(OBJ)
```

### Substitution de suffixe

```makefile
SRC = main.c utils.c parser.c display.c
OBJ = $(SRC:.c=.o)
# OBJ vaut : main.o utils.o parser.o display.o

# C'est equivalent a ecrire manuellement :
# OBJ = main.o utils.o parser.o display.o
```

### Variables courantes

| Variable   | Usage courant                            | Exemple                                    |
| ---------- | ---------------------------------------- | ------------------------------------------ |
| `CC`       | Compilateur C                            | `CC = gcc`                                 |
| `CFLAGS`   | Flags de compilation                     | `CFLAGS = -Wall -Werror -Wextra`           |
| `LDFLAGS`  | Flags de linkage                         | `LDFLAGS = -lm -lpthread`                  |
| `NAME`     | Nom de l'executable                      | `NAME = my_program`                        |
| `SRC`      | Fichiers sources                         | `SRC = main.c utils.c`                     |
| `OBJ`      | Fichiers objets                          | `OBJ = $(SRC:.c=.o)`                       |
| `INCLUDES` | Repertoires d'inclusion                  | `INCLUDES = -I./include`                   |
| `RM`       | Commande de suppression                  | `RM = rm -f`                               |

### Affectation des variables

```makefile
# = : affectation recursive (evaluee a chaque utilisation)
CC = gcc

# := : affectation simple (evaluee une seule fois)
NOW := $(shell date)

# ?= : affectation si non defini
CC ?= gcc

# += : ajout a la valeur existante
CFLAGS = -Wall
CFLAGS += -Werror
CFLAGS += -Wextra
# CFLAGS vaut : -Wall -Werror -Wextra
```

---

## Variables automatiques

Les variables automatiques sont definies **automatiquement** par Make dans chaque regle :

| Variable | Signification                        | Exemple                      |
| -------- | ------------------------------------ | ---------------------------- |
| `$@`     | Nom de la **cible**                  | `prog` dans `prog: main.o`  |
| `$<`     | **Premiere** dependance              | `main.c` dans `main.o: main.c main.h` |
| `$^`     | **Toutes** les dependances           | `main.o utils.o` dans `prog: main.o utils.o` |
| `$*`     | Le **stem** (partie matchee par %)   | `main` dans `%.o: %.c`      |
| `$?`     | Dependances **plus recentes** que la cible | Fichiers modifies      |

### Exemples d'utilisation

```makefile
CC = gcc
CFLAGS = -Wall -Werror -Wextra

# $@ = prog, $^ = main.o utils.o
prog: main.o utils.o
	$(CC) $(CFLAGS) -o $@ $^

# $@ = main.o, $< = main.c
main.o: main.c main.h
	$(CC) $(CFLAGS) -c $< -o $@

# $@ = utils.o, $< = utils.c
utils.o: utils.c utils.h main.h
	$(CC) $(CFLAGS) -c $< -o $@
```

> [!tip] Memotechnique
> - `$@` : la cible (l'**arobase** pointe vers l'objectif)
> - `$<` : premiere dependance (la plus **petite**, la premiere)
> - `$^` : toutes les dependances (le **chapeau** couvre tout)

---

## Regles implicites et Pattern rules

### Le probleme de repetition

```makefile
# Sans pattern rules : tres repetitif !
main.o: main.c
	$(CC) $(CFLAGS) -c main.c -o main.o

utils.o: utils.c
	$(CC) $(CFLAGS) -c utils.c -o utils.o

parser.o: parser.c
	$(CC) $(CFLAGS) -c parser.c -o parser.o

# Imagine avec 50 fichiers...
```

### Pattern rules avec %

```makefile
# UNE SEULE regle pour TOUS les .c --> .o
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

```
% est un joker (wildcard) :
  %.o : %.c signifie "pour tout fichier X.o, il depend de X.c"

  Quand make a besoin de main.o :
    % = main
    $< = main.c
    $@ = main.o

  Quand make a besoin de utils.o :
    % = utils
    $< = utils.c
    $@ = utils.o
```

### Makefile complet avec pattern rules

```makefile
CC = gcc
CFLAGS = -Wall -Werror -Wextra -pedantic
NAME = prog
SRC = main.c utils.c parser.c display.c
OBJ = $(SRC:.c=.o)

$(NAME): $(OBJ)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJ)
```

> [!info] Compilation separee - Pourquoi c'est important
> ```
> Projet avec 100 fichiers .c
> Modification d'UN SEUL fichier (utils.c)
> 
> Sans compilation separee : recompiler les 100 fichiers  (~60 sec)
> Avec compilation separee : recompiler 1 fichier + link   (~2 sec)
> ```

---

## Cibles speciales

### Les cibles standard

```makefile
CC = gcc
CFLAGS = -Wall -Werror -Wextra -pedantic -std=gnu89
NAME = prog
SRC = main.c utils.c parser.c
OBJ = $(SRC:.c=.o)

# all : cible par defaut (premiere cible du Makefile)
all: $(NAME)

# Construction du programme
$(NAME): $(OBJ)
	$(CC) $(CFLAGS) -o $@ $^

# Pattern rule
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# clean : supprimer les fichiers objets
clean:
	rm -f $(OBJ)

# fclean : supprimer les fichiers objets ET l'executable
fclean: clean
	rm -f $(NAME)

# re : tout recompiler de zero
re: fclean all

# Declarer les cibles qui ne sont pas des fichiers
.PHONY: all clean fclean re
```

### .PHONY explique

```
Sans .PHONY :
  Si un FICHIER nomme "clean" existe dans le repertoire,
  make dira : "clean is up to date" et ne fera RIEN !

Avec .PHONY :
  make sait que "clean" n'est PAS un fichier,
  c'est juste un nom de commande. Il execute toujours la recette.

.PHONY: all clean fclean re
```

> [!warning] Toujours declarer .PHONY
> Toutes les cibles qui ne produisent pas de fichier du meme nom doivent etre declarees dans `.PHONY`. C'est le cas de `all`, `clean`, `fclean`, `re`, `install`, `test`, etc.

### Flux des dependances

```
  make re
    |
    +--> fclean
    |      |
    |      +--> clean
    |      |      |
    |      |      +--> rm -f $(OBJ)     # supprime les .o
    |      |
    |      +--> rm -f $(NAME)           # supprime l'executable
    |
    +--> all
           |
           +--> $(NAME)
                  |
                  +--> $(OBJ)
                         |
                         +--> %.o: %.c  # recompile chaque .c
```

---

## Makefile Holberton complet

Voici le Makefile standard utilise dans les projets Holberton :

```makefile
# ============================================================================
# Makefile - Projet Holberton
# ============================================================================

CC = gcc
CFLAGS = -Wall -Werror -Wextra -pedantic -std=gnu89
NAME = prog

# Sources et objets
SRC = $(wildcard *.c)
OBJ = $(SRC:.c=.o)

# Headers
INCLUDES = -I.

# ============================================================================
# Regles
# ============================================================================

all: $(NAME)

$(NAME): $(OBJ)
	$(CC) $(CFLAGS) $(INCLUDES) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@

clean:
	rm -f $(OBJ)

fclean: clean
	rm -f $(NAME)

re: fclean all

.PHONY: all clean fclean re
```

> [!example] Utilisation au quotidien
> ```bash
> # Compiler le projet
> make
> 
> # Nettoyer les .o
> make clean
> 
> # Tout nettoyer (y compris l'executable)
> make fclean
> 
> # Recompiler de zero
> make re
> 
> # Compiler avec des flags supplementaires
> make CFLAGS="-Wall -Werror -Wextra -g"
> ```

### Convention des noms Holberton

```
+-------------------------------------------------------+
|  Regle       | Obligatoire | Description               |
|--------------|-------------|---------------------------|
|  all         | OUI         | Cible par defaut          |
|  clean       | OUI         | Supprime les .o           |
|  fclean      | OUI         | Supprime .o + executable  |
|  re          | OUI         | Recompile tout            |
|  .PHONY      | OUI         | Declare les fausses cibles|
|  $(NAME)     | OUI         | Construit l'executable    |
+-------------------------------------------------------+
```

---

## Fonctions Make

### $(wildcard pattern)

Retourne la liste des fichiers correspondant au pattern :

```makefile
SRC = $(wildcard *.c)
# Si le repertoire contient main.c, utils.c, parser.c :
# SRC = main.c utils.c parser.c

HEADERS = $(wildcard include/*.h)
# Liste tous les .h dans include/

ALL_SRC = $(wildcard src/*.c) $(wildcard src/**/*.c)
```

### $(patsubst pattern,replacement,text)

Remplacement de pattern dans une liste :

```makefile
SRC = main.c utils.c parser.c

OBJ = $(patsubst %.c,%.o,$(SRC))
# OBJ = main.o utils.o parser.o

# Equivalent plus court :
OBJ = $(SRC:.c=.o)

# Changer le repertoire de sortie :
OBJ = $(patsubst %.c,build/%.o,$(SRC))
# OBJ = build/main.o build/utils.o build/parser.o
```

### $(shell commande)

Execute une commande shell et retourne le resultat :

```makefile
DATE = $(shell date +%Y-%m-%d)
USER = $(shell whoami)
NUM_CORES = $(shell nproc)
GIT_HASH = $(shell git rev-parse --short HEAD 2>/dev/null)

# Compilation parallele
MAKEFLAGS += -j$(shell nproc)
```

### $(addprefix prefix,names)

```makefile
SRC_FILES = main.c utils.c parser.c
SRC = $(addprefix src/,$(SRC_FILES))
# SRC = src/main.c src/utils.c src/parser.c
```

### $(notdir names)

```makefile
PATHS = src/main.c src/utils.c lib/helper.c
FILES = $(notdir $(PATHS))
# FILES = main.c utils.c helper.c
```

### $(filter pattern,text) et $(filter-out pattern,text)

```makefile
SRC = main.c utils.c test_main.c test_utils.c

# Garder seulement les fichiers de test
TESTS = $(filter test_%,$(SRC))
# TESTS = test_main.c test_utils.c

# Exclure les fichiers de test
PROD_SRC = $(filter-out test_%,$(SRC))
# PROD_SRC = main.c utils.c
```

---

## Dependances de headers

### Le probleme

```
Si on modifie utils.h, les fichiers .c qui l'incluent
doivent etre recompiles. Mais Make ne le sait pas
automatiquement !
```

### Solution manuelle

```makefile
main.o: main.c main.h utils.h
	$(CC) $(CFLAGS) -c $< -o $@

utils.o: utils.c utils.h
	$(CC) $(CFLAGS) -c $< -o $@
```

### Solution automatique avec GCC

```makefile
CC = gcc
CFLAGS = -Wall -Werror -Wextra -pedantic
DEPFLAGS = -MMD -MP
NAME = prog
SRC = $(wildcard *.c)
OBJ = $(SRC:.c=.o)
DEP = $(SRC:.c=.d)

all: $(NAME)

$(NAME): $(OBJ)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) $(DEPFLAGS) -c $< -o $@

# Inclure les fichiers de dependances generes
-include $(DEP)

clean:
	rm -f $(OBJ) $(DEP)

fclean: clean
	rm -f $(NAME)

re: fclean all

.PHONY: all clean fclean re
```

> [!info] Comment ca marche
> - `-MMD` : GCC genere un fichier `.d` avec les dependances (quels `.h` sont inclus)
> - `-MP` : ajoute des cibles factices pour eviter les erreurs si un `.h` est supprime
> - `-include $(DEP)` : Make lit ces fichiers `.d` (le `-` ignore l'erreur si les fichiers n'existent pas encore)
> 
> Exemple de fichier `main.d` genere :
> ```
> main.o: main.c main.h utils.h config.h
> ```

---

## Makefile multi-repertoires

### Structure de projet typique

```
projet/
  |-- Makefile
  |-- include/
  |     |-- main.h
  |     |-- utils.h
  |-- src/
  |     |-- main.c
  |     |-- utils.c
  |     |-- parser.c
  |-- build/
  |     |-- (fichiers .o ici)
  |-- bin/
        |-- (executable ici)
```

### Makefile avec repertoires

```makefile
CC = gcc
CFLAGS = -Wall -Werror -Wextra -pedantic -std=gnu89
NAME = prog

# Repertoires
SRC_DIR = src
INC_DIR = include
BUILD_DIR = build
BIN_DIR = bin

# Fichiers
SRC = $(wildcard $(SRC_DIR)/*.c)
OBJ = $(patsubst $(SRC_DIR)/%.c,$(BUILD_DIR)/%.o,$(SRC))
INCLUDES = -I$(INC_DIR)

# Cible principale
all: directories $(BIN_DIR)/$(NAME)

# Creer les repertoires si necessaire
directories:
	@mkdir -p $(BUILD_DIR) $(BIN_DIR)

# Linker
$(BIN_DIR)/$(NAME): $(OBJ)
	$(CC) $(CFLAGS) -o $@ $^

# Compiler
$(BUILD_DIR)/%.o: $(SRC_DIR)/%.c
	$(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@

clean:
	rm -rf $(BUILD_DIR)

fclean: clean
	rm -rf $(BIN_DIR)

re: fclean all

.PHONY: all clean fclean re directories
```

### VPATH

`VPATH` indique a Make ou chercher les fichiers sources :

```makefile
VPATH = src:lib:utils
# Make cherchera les .c dans src/, lib/ et utils/

# Ou de maniere plus specifique :
vpath %.c src lib
vpath %.h include
```

---

## Debugging un Makefile

### Commandes utiles

```bash
# -n : dry run (affiche les commandes sans les executer)
make -n
make -n clean

# -p : affiche la base de donnees interne de Make
make -p | less

# -d : mode debug detaille
make -d 2>&1 | head -100

# --debug=basic : debug simplifie
make --debug=basic

# Afficher la valeur d'une variable
make -p | grep "^CC"

# Surcharger une variable
make CC=clang
make CFLAGS="-Wall -g -O0"
```

### Methode de debug dans le Makefile

```makefile
# Afficher des informations
debug:
	@echo "SRC = $(SRC)"
	@echo "OBJ = $(OBJ)"
	@echo "CC = $(CC)"
	@echo "CFLAGS = $(CFLAGS)"

# @ devant une commande la rend silencieuse (pas affichee)
all: $(NAME)
	@echo "Compilation terminee avec succes !"
```

> [!tip] Astuce : le @ silencieux
> Par defaut, Make affiche chaque commande avant de l'executer. Prefixer une commande par `@` la rend silencieuse :
> ```makefile
> clean:
> 	@echo "Nettoyage..."     # Affiche le message
> 	@rm -f $(OBJ) $(NAME)   # Supprime silencieusement
> 	@echo "Fait !"
> ```

### $(info ...), $(warning ...), $(error ...)

```makefile
$(info Sources trouvees : $(SRC))
$(warning OBJ est vide !) 
$(error Arret : variable NAME non definie)

# Conditionnel
ifeq ($(SRC),)
$(error Aucun fichier source trouve !)
endif
```

---

## Erreurs classiques

### 1. Espaces au lieu de TAB

```makefile
# ERREUR : espaces devant la commande
prog: main.c
    gcc -o prog main.c    # <-- ESPACES = ERREUR !

# CORRECT : TAB devant la commande
prog: main.c
	gcc -o prog main.c    # <-- TAB = OK
```

```
Erreur typique :
Makefile:3: *** missing separator.  Stop.
```

### 2. Dependances manquantes

```makefile
# ERREUR : si main.c inclut utils.h, et qu'on modifie utils.h,
# main.o ne sera PAS recompile !
main.o: main.c
	$(CC) $(CFLAGS) -c $< -o $@

# CORRECT : declarer la dependance
main.o: main.c utils.h
	$(CC) $(CFLAGS) -c $< -o $@
```

### 3. Oublier .PHONY

```makefile
# PROBLEME : si un fichier "clean" existe dans le repertoire
clean:
	rm -f *.o
# make dira : "clean is up to date" !

# SOLUTION :
.PHONY: clean
clean:
	rm -f *.o
```

### 4. Ordre des cibles

```makefile
# PROBLEME : la premiere regle est la cible par defaut
clean:              # <-- clean sera la cible par defaut !
	rm -f *.o

all: $(NAME)        # <-- Trop tard, clean est deja la cible par defaut

# SOLUTION : all en premier
all: $(NAME)

clean:
	rm -f *.o
```

### 5. Variable non definie

```makefile
# PROBLEME : faute de frappe
CFLAG = -Wall       # <-- Devrait etre CFLAGS

$(NAME): $(OBJ)
	$(CC) $(CFLAGS) -o $@ $^   # <-- CFLAGS est vide !
```

### Resume des erreurs

```
+----------------------------------------------------------+
|  Erreur                    |  Symptome                    |
|----------------------------|------------------------------|
|  Espaces au lieu de TAB    |  "missing separator"         |
|  Dependances manquantes    |  Fichier pas recompile       |
|  .PHONY oublie             |  "is up to date"             |
|  all pas en premiere cible |  Mauvaise cible par defaut   |
|  Faute dans nom variable   |  Flags non appliques         |
|  Pas de $() autour de var  |  Variable non expandue       |
+----------------------------------------------------------+
```

---

## Exercices

### Exercice 1 : Makefile basique

> [!example] Exercice
> Ecrivez un Makefile qui compile `main.c` en un executable `school` avec les flags `-Wall -Werror -Wextra -pedantic`.
>
> **Solution :**
> ```makefile
> CC = gcc
> CFLAGS = -Wall -Werror -Wextra -pedantic
> NAME = school
> 
> all: $(NAME)
> 
> $(NAME): main.c
> 	$(CC) $(CFLAGS) -o $@ $^
> 
> clean:
> 	rm -f $(NAME)
> 
> .PHONY: all clean
> ```

### Exercice 2 : Makefile avec fichiers objets

> [!example] Exercice
> Ecrivez un Makefile pour un projet avec `main.c`, `utils.c` et `helper.c`. Le Makefile doit :
> - Compiler chaque `.c` en `.o` separement
> - Linker les `.o` en un executable `school`
> - Avoir les cibles `all`, `clean`, `fclean`, `re`
>
> **Solution :**
> ```makefile
> CC = gcc
> CFLAGS = -Wall -Werror -Wextra -pedantic -std=gnu89
> NAME = school
> SRC = main.c utils.c helper.c
> OBJ = $(SRC:.c=.o)
> 
> all: $(NAME)
> 
> $(NAME): $(OBJ)
> 	$(CC) $(CFLAGS) -o $@ $^
> 
> %.o: %.c
> 	$(CC) $(CFLAGS) -c $< -o $@
> 
> clean:
> 	rm -f $(OBJ)
> 
> fclean: clean
> 	rm -f $(NAME)
> 
> re: fclean all
> 
> .PHONY: all clean fclean re
> ```

### Exercice 3 : Variables automatiques

> [!example] Exercice
> Que valent les variables automatiques dans la regle suivante ?
> ```makefile
> server: main.o network.o config.o
> 	$(CC) -o $@ $^
> ```
>
> **Solution :**
> - `$@` = `server` (la cible)
> - `$^` = `main.o network.o config.o` (toutes les dependances)
> - `$<` = `main.o` (la premiere dependance)

### Exercice 4 : Makefile avec wildcard

> [!example] Exercice
> Ecrivez un Makefile qui detecte automatiquement **tous** les fichiers `.c` dans le repertoire courant et les compile.
>
> **Solution :**
> ```makefile
> CC = gcc
> CFLAGS = -Wall -Werror -Wextra -pedantic -std=gnu89
> NAME = school
> SRC = $(wildcard *.c)
> OBJ = $(SRC:.c=.o)
> 
> all: $(NAME)
> 
> $(NAME): $(OBJ)
> 	$(CC) $(CFLAGS) -o $@ $^
> 
> %.o: %.c
> 	$(CC) $(CFLAGS) -c $< -o $@
> 
> clean:
> 	rm -f $(OBJ)
> 
> fclean: clean
> 	rm -f $(NAME)
> 
> re: fclean all
> 
> .PHONY: all clean fclean re
> ```

### Exercice 5 : Makefile multi-repertoires

> [!example] Exercice
> Ecrivez un Makefile pour la structure suivante :
> ```
> projet/
>   Makefile
>   include/main.h
>   src/main.c
>   src/utils.c
> ```
> Les fichiers `.o` doivent aller dans un repertoire `build/`.
>
> **Solution :**
> ```makefile
> CC = gcc
> CFLAGS = -Wall -Werror -Wextra -pedantic -I./include
> NAME = school
> SRC_DIR = src
> BUILD_DIR = build
> SRC = $(wildcard $(SRC_DIR)/*.c)
> OBJ = $(patsubst $(SRC_DIR)/%.c,$(BUILD_DIR)/%.o,$(SRC))
> 
> all: $(BUILD_DIR) $(NAME)
> 
> $(BUILD_DIR):
> 	mkdir -p $(BUILD_DIR)
> 
> $(NAME): $(OBJ)
> 	$(CC) $(CFLAGS) -o $@ $^
> 
> $(BUILD_DIR)/%.o: $(SRC_DIR)/%.c
> 	$(CC) $(CFLAGS) -c $< -o $@
> 
> clean:
> 	rm -rf $(BUILD_DIR)
> 
> fclean: clean
> 	rm -f $(NAME)
> 
> re: fclean all
> 
> .PHONY: all clean fclean re
> ```

### Exercice 6 : Trouver les erreurs

> [!example] Exercice
> Trouvez les 5 erreurs dans ce Makefile :
> ```makefile
> cc = gcc
> CFLAGS = -Wall -Werror
> NAME = prog
> SRC = main.c utils.c
> OBJ = $(SRC:.c=.o)
> 
> $(NAME): $(OBJ)
>     $(CC) $(CFLAGS) -o $@ $^
> 
> %.o: %.c
>     $(CC) $(CFLAGS) -c $< -o $@
> 
> all: $(NAME)
> 
> clean:
>     rm -f $(OBJ)
>     rm -f $(NAME)
> ```
>
> **Solution :**
> 1. `cc = gcc` : devrait etre `CC = gcc` (majuscules). `$(CC)` sera vide.
> 2. Les recettes utilisent des **espaces** au lieu de **TAB**.
> 3. `all` n'est pas la **premiere** cible (c'est `$(NAME)` qui est par defaut).
> 4. Pas de `fclean` et `re` (conventions Holberton).
> 5. Pas de `.PHONY` pour `all` et `clean`.

---

## Liens

- [[01 - Introduction au C et Compilation]] : comprendre gcc et le pipeline de compilation
- [[00b - Le Preprocesseur C]] : les directives du preprocesseur et les header guards
- [[04 - Pointeurs et Memoire]] : compilation separee et headers

---

> [!tip] Resume
> Un Makefile est **indispensable** pour tout projet C serieux. Retenez les bases : **regles** (cible : dependances + recette avec TAB), **variables** (CC, CFLAGS, NAME, SRC, OBJ), **variables automatiques** ($@, $<, $^), **pattern rules** (%.o: %.c), et les **cibles standard** (all, clean, fclean, re, .PHONY). Commencez simple et complexifiez progressivement.
