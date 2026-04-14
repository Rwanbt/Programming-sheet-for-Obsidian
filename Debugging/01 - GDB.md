# GDB - Le Debugger GNU

## Qu'est-ce que GDB ?

**GDB** (GNU Debugger) est un outil qui te permet d'**exécuter ton programme au ralenti**, de le **mettre en pause**, d'**inspecter chaque variable**, et de comprendre **exactement** ce qui se passe ligne par ligne.

> [!tip] Analogie Gaming
> Imagine un jeu vidéo avec un **mode debug** :
> - **Pause** = tu arrêtes le jeu à n'importe quel moment (breakpoint)
> - **Step** = tu avances frame par frame (next/step)
> - **Inspect** = tu ouvres la console pour voir la position du joueur, ses HP, l'état du monde (print)
> - **Resume** = tu relances le jeu en temps réel (continue)
>
> GDB, c'est exactement ça, mais pour ton code C.

---

## Installation

### Sur Ubuntu / Debian

```bash
sudo apt update
sudo apt install gdb
```

### Vérifier l'installation

```bash
gdb --version
```

---

## Compilation : TOUJOURS avec `-g`

> [!warning] Règle Absolue
> **TOUJOURS** compiler avec le flag `-g` quand tu veux debugger. Sans `-g`, GDB est quasiment inutile.

Le flag `-g` ajoute les **informations de debug** (noms de variables, numéros de lignes, noms de fonctions) dans l'exécutable.

### Comparaison : avec et sans `-g`

**Sans `-g`** (compilation normale) :
```bash
gcc -Wall -Werror -Wextra main.c -o prog
```

Dans GDB, tu verras :
```
(gdb) print i
No symbol "i" in current context.
(gdb) list
No symbol table is loaded.
```
→ GDB ne sait **rien** sur ton code. Inutile.

**Avec `-g`** (compilation debug) :
```bash
gcc -Wall -Werror -Wextra -g main.c -o prog
```

Dans GDB, tu verras :
```
(gdb) print i
$1 = 42
(gdb) list
1    #include <stdio.h>
2    int main(void) {
3        int i = 42;
4        printf("%d\n", i);
5    }
```
→ GDB connaît **tout** : variables, lignes, fonctions.

> [!info] Bonne pratique
> Tu peux garder `-g` même avec `-Wall -Werror -Wextra`. Ça n'affecte pas les warnings :
> ```bash
> gcc -Wall -Werror -Wextra -g main.c -o prog
> ```

---

## Lancer GDB

### Les différentes façons

```bash
# Lancer GDB sur un programme
gdb ./prog

# Lancer GDB avec des arguments pour le programme
gdb --args ./prog arg1 arg2

# Mode silencieux (sans le long message d'accueil)
gdb -q ./prog

# Mode TUI (interface graphique dans le terminal)
gdb -tui ./prog
```

| Commande | Description |
|---|---|
| `gdb ./prog` | Lancer GDB sur `prog` |
| `gdb --args ./prog a b` | Passer des arguments à `prog` |
| `gdb -q ./prog` | Mode quiet (pas de bannière) |
| `gdb -tui ./prog` | Mode Text User Interface |

Une fois dans GDB, tu vois le prompt :
```
(gdb) _
```

C'est ici que tu tapes tes commandes.

---

## Commandes Fondamentales

### Exécution

| Commande | Abréviation | Description |
|---|---|---|
| `run` | `r` | Lancer le programme |
| `run arg1 arg2` | `r arg1 arg2` | Lancer avec des arguments |
| `quit` | `q` | Quitter GDB |
| `kill` | `k` | Arrêter le programme sans quitter GDB |

### Navigation pas-à-pas

| Commande | Abréviation | Description |
|---|---|---|
| `next` | `n` | Exécuter la ligne suivante (sans entrer dans les fonctions) |
| `step` | `s` | Exécuter la ligne suivante (en entrant dans les fonctions) |
| `continue` | `c` | Continuer jusqu'au prochain breakpoint |
| `finish` | `fin` | Continuer jusqu'à la fin de la fonction actuelle |
| `until` | `u` | Continuer jusqu'à une ligne plus grande (sortir d'une boucle) |
| `next N` | `n N` | Exécuter N lignes d'un coup |

> [!warning] `next` vs `step` - LA différence cruciale
> ```c
> int add(int a, int b) {     // <-- step entre ICI
>     return a + b;
> }
>
> int main(void) {
>     int x = add(3, 4);      // <-- tu es ICI
>     printf("%d\n", x);
> }
> ```
>
> - **`next`** : exécute `add(3, 4)` **en entier** et passe directement à `printf`. Tu ne vois pas l'intérieur de `add`.
> - **`step`** : entre **dans** la fonction `add` et s'arrête à `return a + b;`. Tu vois l'intérieur.
>
> **Règle simple** :
> - `next` = "je m'en fiche de cette fonction, avance"
> - `step` = "je veux voir ce qui se passe dans cette fonction"

### Exemple de session

```
(gdb) break main
Breakpoint 1 at 0x1149: file main.c, line 7.
(gdb) run
Starting program: /home/user/prog

Breakpoint 1, main () at main.c:7
7       int x = add(3, 4);
(gdb) step
add (a=3, b=4) at main.c:3
3       return a + b;
(gdb) finish
Run till exit from #0  add (a=3, b=4) at main.c:3
0x0000555555555158 in main () at main.c:7
7       int x = add(3, 4);
Value returned is $1 = 7
(gdb) next
8       printf("%d\n", x);
```

---

## Breakpoints (Points d'arrêt)

Un breakpoint est un **point de pause** dans ton code. Quand le programme atteint cette ligne, il s'arrête et te donne le contrôle.

### Poser un breakpoint

```
break main              # Sur la fonction main
break 42                # Sur la ligne 42 du fichier courant
break main.c:42         # Sur la ligne 42 de main.c
break add               # Sur la fonction add
```

### Breakpoints conditionnels

Tu peux dire : "arrête-toi SEULEMENT si une condition est vraie".

```
break 42 if i == 5      # S'arrêter ligne 42 seulement si i vaut 5
break 42 if ptr == 0    # Seulement si ptr est NULL
break 42 if len > 100   # Seulement si len dépasse 100
```

> [!tip] Super utile pour les boucles
> Si tu as une boucle de 10 000 itérations et le bug apparaît à `i == 9999`, tu ne vas pas faire 9 999 fois `next` :
> ```
> (gdb) break 15 if i == 9999
> (gdb) continue
> ```
> GDB s'arrête directement à l'itération 9999.

### Gérer les breakpoints

| Commande | Description |
|---|---|
| `info breakpoints` | Lister tous les breakpoints |
| `info b` | Pareil, en abrégé |
| `delete 1` | Supprimer le breakpoint n°1 |
| `delete` | Supprimer TOUS les breakpoints |
| `disable 2` | Désactiver le breakpoint n°2 (sans le supprimer) |
| `enable 2` | Réactiver le breakpoint n°2 |

```
(gdb) info breakpoints
Num  Type           Disp Enb Address            What
1    breakpoint     keep y   0x0000555555555149 in main at main.c:7
2    breakpoint     keep n   0x0000555555555180 in add at main.c:3
```

- **Enb** = Enabled : `y` = actif, `n` = désactivé

---

## Inspecter les données

### `print` - Afficher une valeur

C'est la commande que tu utiliseras **le plus** avec GDB.

```
print i              # Afficher la valeur de i
print x + y          # Afficher le résultat d'une expression
print sizeof(int)    # Afficher la taille d'un type
```

#### Formats d'affichage

| Format | Description | Exemple |
|---|---|---|
| `print i` | Décimal (défaut) | `$1 = 42` |
| `print/x i` | Hexadécimal | `$2 = 0x2a` |
| `print/d i` | Décimal signé | `$3 = 42` |
| `print/u i` | Décimal non-signé | `$4 = 42` |
| `print/o i` | Octal | `$5 = 052` |
| `print/t i` | Binaire | `$6 = 00101010` |
| `print/c i` | Caractère | `$7 = 42 '*'` |
| `print/s str` | String | `$8 = "Hello"` |
| `print/f val` | Flottant | `$9 = 3.14` |

#### Inspecter des structures, pointeurs, tableaux

```c
// Avec cette struct :
typedef struct {
    char *name;
    int age;
} Person;

Person p = {"Alice", 25};
Person *ptr = &p;
int arr[5] = {10, 20, 30, 40, 50};
```

```
(gdb) print p            # Affiche toute la struct
$1 = {name = "Alice", age = 25}

(gdb) print p.name       # Un champ spécifique
$2 = "Alice"

(gdb) print p.age        # Un autre champ
$3 = 25

(gdb) print *ptr         # Déréférencer un pointeur vers struct
$4 = {name = "Alice", age = 25}

(gdb) print ptr->name    # Accès via pointeur
$5 = "Alice"

(gdb) print *arr@5       # Afficher 5 éléments du tableau
$6 = {10, 20, 30, 40, 50}

(gdb) print arr[2]       # Un élément précis
$7 = 30
```

> [!tip] `print *arr@N`
> La syntaxe `*pointeur@N` affiche N éléments consécutifs à partir de l'adresse du pointeur. Indispensable pour les tableaux alloués dynamiquement :
> ```
> (gdb) print *my_malloc_array@10
> ```

### `display` - Affichage automatique

`display` affiche une expression **automatiquement après chaque `step` ou `next`**. Comme un HUD dans un jeu.

```
(gdb) display i
1: i = 0
(gdb) next
1: i = 1
(gdb) next
1: i = 2
```

| Commande | Description |
|---|---|
| `display i` | Afficher `i` après chaque pas |
| `display/x ptr` | Afficher `ptr` en hexa après chaque pas |
| `info display` | Lister les display actifs |
| `undisplay 1` | Supprimer le display n°1 |

### `watch` - Watchpoints

Un watchpoint arrête le programme quand une variable **change de valeur**.

```
(gdb) watch i
Hardware watchpoint 2: i
(gdb) continue
Hardware watchpoint 2: i

Old value = 0
New value = 1
0x0000555555555167 in main () at main.c:8
```

> [!info] Différence breakpoint vs watchpoint
> - **Breakpoint** : s'arrête quand on **atteint une ligne**
> - **Watchpoint** : s'arrête quand une **variable change de valeur**

### `ptype` et `whatis` - Information sur les types

```
(gdb) ptype p
type = struct {
    char *name;
    int age;
}

(gdb) whatis p
type = Person

(gdb) whatis p.name
type = char *
```

- `ptype` : affiche la **définition complète** du type
- `whatis` : affiche le **nom** du type

---

## Examiner la mémoire : `x` (examine)

La commande `x` permet de lire directement la **mémoire brute**. Sa syntaxe est :

```
x/NFS adresse
```

| Lettre | Signification | Valeurs possibles |
|---|---|---|
| **N** | Nombre d'unités à afficher | 1, 2, 5, 10, ... |
| **F** | Format d'affichage | `x` hexa, `d` décimal, `c` char, `s` string, `i` instruction |
| **S** | Taille d'une unité | `b` byte (1), `h` half-word (2), `w` word (4), `g` giant (8) |

### Exemples

```
(gdb) x/4xw &arr
0x7fffffffe410:  0x0000000a  0x00000014  0x0000001e  0x00000028
# 4 mots (word) en hexa → les 4 premiers éléments du tableau

(gdb) x/s str
0x555555556004:  "Hello World"
# Afficher une chaîne de caractères

(gdb) x/10xb &var
0x7fffffffe40f:  0x2a  0x00  0x00  0x00  0x00  0x00  0x00  0x00  0x00  0x00
# 10 octets en hexa à partir de l'adresse de var

(gdb) x/5i $rip
0x555555555149 <main+4>:  sub    $0x10,%rsp
# 5 instructions assembleur à partir de l'instruction courante

(gdb) x/1xg &ptr
0x7fffffffe418:  0x00005555555592a0
# 1 giant (8 octets) en hexa → la valeur d'un pointeur 64 bits
```

> [!tip] Quand utiliser `x` plutôt que `print` ?
> - `print` : afficher une **variable** ou expression C
> - `x` : regarder la **mémoire brute**, utile pour les buffers, les tableaux d'octets, le debugging bas-niveau

---

## Informations sur le contexte

```
info locals        # Toutes les variables locales de la fonction actuelle
info args          # Les arguments de la fonction actuelle
info registers     # Tous les registres CPU
```

### Exemple

```
(gdb) info locals
i = 42
ptr = 0x5555555592a0
name = "Alice"

(gdb) info args
argc = 1
argv = 0x7fffffffe568
```

---

## Backtrace - LA commande la plus importante

### Qu'est-ce qu'un backtrace ?

Un **backtrace** (ou **stack trace**) montre la **pile d'appels** : quelle fonction a appelé quelle autre fonction. C'est **LA** première commande à taper quand ton programme crash.

```
(gdb) backtrace
```
ou
```
(gdb) bt
```

### Exemple concret

```c
void crash_here(char *s) {
    printf("%s\n", s);     // s est NULL → SEGFAULT
}

void middle(char *s) {
    crash_here(s);
}

int main(void) {
    middle(NULL);
}
```

```
(gdb) run
Program received signal SIGSEGV, Segmentation fault.

(gdb) bt
#0  crash_here (s=0x0) at main.c:4
#1  0x0000555555555178 in middle (s=0x0) at main.c:8
#2  0x0000555555555193 in main () at main.c:12
```

**Comment lire** :
- `#0` = la fonction où le crash s'est produit → `crash_here` ligne 4
- `#1` = qui a appelé `crash_here` → `middle` ligne 8
- `#2` = qui a appelé `middle` → `main` ligne 12
- `s=0x0` = le paramètre `s` vaut NULL → c'est la cause !

### Naviguer dans la pile

```
frame 0            # Aller dans le frame #0 (crash_here)
frame 2            # Aller dans le frame #2 (main)
up                 # Monter d'un frame (vers l'appelant)
down               # Descendre d'un frame (vers l'appelé)
```

> [!warning] Quand tu changes de frame, tu peux inspecter les variables locales de CE frame :
> ```
> (gdb) frame 2
> (gdb) info locals    # Variables locales de main()
> (gdb) frame 0
> (gdb) info locals    # Variables locales de crash_here()
> ```

---

## Registres CPU (x86-64)

Quand tu fais du debugging bas-niveau, tu dois connaître les registres principaux.

| Registre | Rôle | Description |
|---|---|---|
| `rip` | Instruction Pointer | Adresse de la prochaine instruction à exécuter |
| `rsp` | Stack Pointer | Sommet de la pile |
| `rbp` | Base Pointer | Base du frame de la fonction actuelle |
| `rax` | Return Value | Valeur de retour d'une fonction |
| `rdi` | 1er argument | Premier argument passé à une fonction |
| `rsi` | 2ème argument | Deuxième argument |
| `rdx` | 3ème argument | Troisième argument |
| `rcx` | 4ème argument | Quatrième argument |
| `r8` | 5ème argument | Cinquième argument |
| `r9` | 6ème argument | Sixième argument |
| `rbx` | General purpose | Registre à usage général (sauvegardé par l'appelé) |

```
(gdb) info registers
rax  0x7    7
rbx  0x0    0
rip  0x555555555149   0x555555555149 <main+4>
rsp  0x7fffffffe430   0x7fffffffe430
```

```
(gdb) print $rax          # Afficher un registre spécifique
(gdb) print/x $rip        # En hexa
```

---

## Modifier des valeurs en cours d'exécution

### `set variable` - Changer une variable

```
(gdb) set variable i = 100
(gdb) set variable ptr = 0
(gdb) set variable name = "Bob"
```

> [!tip] Utilité
> Tu peux tester "et si cette variable avait cette valeur ?" sans recompiler. Super utile pour explorer des scénarios.

### `call` - Appeler une fonction

```
(gdb) call printf("Debug: i = %d\n", i)
Debug: i = 42
```

### `return` - Forcer le retour d'une fonction

```
(gdb) return 0       # Quitter la fonction actuelle en retournant 0
(gdb) return -1      # Retourner -1
```

---

## Mode TUI (Text User Interface)

Le mode TUI affiche ton **code source** en direct dans le terminal pendant le debugging.

### Activer / Désactiver

```bash
gdb -tui ./prog            # Lancer directement en mode TUI
```

Ou une fois dans GDB :
```
(gdb) tui enable           # Activer le TUI
(gdb) tui disable          # Désactiver le TUI
```

**Raccourci clavier** : `Ctrl+X` puis `A` pour basculer.

### Ce que ça ressemble

```
┌──main.c──────────────────────────────────────────┐
│   1  #include <stdio.h>                          │
│   2                                              │
│   3  int add(int a, int b) {                     │
│   4      return a + b;                           │
│   5  }                                           │
│   6                                              │
│  >7  int main(void) {                            │
│   8      int x = add(3, 4);                      │
│   9      printf("%d\n", x);                      │
│  10      return 0;                               │
│  11  }                                           │
└──────────────────────────────────────────────────┘
(gdb) _
```

Le `>` indique la ligne courante. Les breakpoints apparaissent avec un `B+`.

> [!info] Si l'affichage est cassé
> Tape `Ctrl+L` pour rafraîchir l'écran TUI.

---

## Messages d'erreur courants et solutions

### "No debugging symbols found"

```
Reading symbols from ./prog...
(No debugging symbols found in ./prog)
```

**Cause** : Tu as compilé sans `-g`.
**Solution** : Recompile avec `-g` :
```bash
gcc -g main.c -o prog
```

### "No symbol table is loaded"

```
(gdb) list
No symbol table is loaded. Use the "file" command.
```

**Cause** : Pas de symboles de debug. Même problème que ci-dessus.
**Solution** : Recompiler avec `-g`.

### "??" dans le backtrace

```
#0  0x00007ffff7a42e10 in ?? ()
#1  0x0000555555555149 in main () at main.c:7
```

**Cause** : Le `??` signifie que GDB ne trouve pas les symboles pour cette fonction (probablement une fonction de la libc ou d'une bibliothèque compilée sans `-g`).
**Solution** : C'est normal pour les fonctions système. Concentre-toi sur les frames qui montrent TON code.

### "Cannot access memory at address 0x0"

```
(gdb) print *ptr
Cannot access memory at address 0x0
```

**Cause** : Tu essaies de déréférencer un **pointeur NULL**.
**Solution** : Vérifie pourquoi `ptr` vaut `0x0` (NULL). Remonte dans le backtrace pour trouver où il aurait dû être initialisé.

---

## Analyse de crashs

### Workflow Segfault

Un **segfault** (Segmentation Fault) signifie que ton programme a accédé à une adresse mémoire invalide.

```
(gdb) run
Program received signal SIGSEGV, Segmentation fault.

(gdb) bt                    # 1. Où est le crash ?
(gdb) frame 0               # 2. Aller au frame du crash
(gdb) info locals            # 3. Quelles sont les variables ?
(gdb) print ptr              # 4. Le pointeur suspect est-il NULL ?
(gdb) frame 1                # 5. Remonter pour comprendre pourquoi
(gdb) info args              # 6. Quels arguments ont été passés ?
```

### Boucle infinie

Si ton programme est bloqué (boucle infinie) :

1. Appuie sur **`Ctrl+C`** dans GDB → le programme est mis en pause
2. Tape `bt` pour voir où tu es
3. Tape `info locals` pour voir la valeur de la variable de boucle
4. Tape `print i` pour vérifier si la variable de boucle progresse correctement

```
^C
Program received signal SIGINT, Interrupt.
0x0000555555555167 in main () at main.c:8
8       while (i < 10)
(gdb) print i
$1 = 5          # i ne change jamais → boucle infinie !
```

### Double free (SIGABRT)

```
Program received signal SIGABRT, Aborted.
```

Un `SIGABRT` après un `free()` signifie généralement un **double free** ou une **corruption du heap**.

```
(gdb) bt
#0  __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:50
#1  __GI_abort () at abort.c:79
#2  __libc_message (action=...) at ../sysdeps/posix/libc_fatal.c:155
#3  malloc_printerr (str=...) at malloc.c:5347
#4  _int_free (av=..., p=...) at malloc.c:4349
#5  0x0000555555555189 in main () at main.c:10
```

→ Frame `#5` montre où dans ton code le problème se produit.

---

## Workflow complet : Session de debugging en 6 étapes

Supposons que tu as ce programme qui crash :

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char *copy_string(char *src) {
    char *dest = malloc(strlen(src));  // BUG: manque +1 pour '\0'
    strcpy(dest, src);
    return dest;
}

int main(void) {
    char *s = copy_string("Hello");
    printf("%s\n", s);
    free(s);
    return 0;
}
```

### Étape 1 : Compiler avec `-g`

```bash
gcc -Wall -Werror -Wextra -g main.c -o prog
```

### Étape 2 : Lancer GDB

```bash
gdb -q ./prog
```

### Étape 3 : Poser un breakpoint et lancer

```
(gdb) break copy_string
Breakpoint 1 at 0x1149: file main.c, line 6.
(gdb) run
```

### Étape 4 : Avancer pas-à-pas et inspecter

```
(gdb) next
6       char *dest = malloc(strlen(src));
(gdb) print src
$1 = "Hello"
(gdb) print strlen(src)
$2 = 5
(gdb) next
7       strcpy(dest, src);
(gdb) print dest
$3 = 0x5555555592a0
```

### Étape 5 : Identifier le bug

```
(gdb) print strlen(src)
$4 = 5
```

On alloue 5 octets mais "Hello" a besoin de **6** (5 + le `\0`). Buffer overflow !

### Étape 6 : Corriger

```c
char *dest = malloc(strlen(src) + 1);  // +1 pour '\0'
```

---

## Astuces et raccourcis

### Touche Entrée vide

> [!tip] Astuce n°1 de GDB
> Appuyer sur **Entrée** sans rien taper **répète la dernière commande**. Super pratique pour faire `next`, `next`, `next`... en appuyant juste sur Entrée.

### Table des abréviations

| Commande complète | Abréviation |
|---|---|
| `run` | `r` |
| `quit` | `q` |
| `break` | `b` |
| `next` | `n` |
| `step` | `s` |
| `continue` | `c` |
| `print` | `p` |
| `backtrace` | `bt` |
| `info breakpoints` | `i b` |
| `info locals` | `i lo` |
| `frame` | `f` |
| `finish` | `fin` |
| `delete` | `d` |
| `list` | `l` |

### Fichier `.gdbinit`

Tu peux créer un fichier `~/.gdbinit` avec des commandes qui s'exécutent automatiquement au lancement de GDB :

```bash
# ~/.gdbinit
set print pretty on          # Affichage joli des structs
set print array on           # Affichage joli des tableaux
set pagination off           # Pas de "press enter to continue"
set history save on          # Sauvegarder l'historique des commandes
set history filename ~/.gdb_history
```

Tu peux aussi créer un `.gdbinit` **dans le dossier du projet** pour des commandes spécifiques :

```bash
# .gdbinit (dans le dossier du projet)
file ./prog
break main
run
```

> [!warning] Sécurité
> GDB peut refuser de charger un `.gdbinit` local. Pour l'autoriser :
> ```bash
> echo "set auto-load safe-path /" >> ~/.gdbinit
> ```

---

## Référence complète des commandes

| Catégorie | Commande | Description |
|---|---|---|
| **Exécution** | `run [args]` | Lancer le programme |
| | `quit` | Quitter GDB |
| | `kill` | Tuer le programme |
| **Navigation** | `next` | Ligne suivante (sans entrer dans les fonctions) |
| | `step` | Ligne suivante (en entrant dans les fonctions) |
| | `continue` | Continuer jusqu'au prochain breakpoint |
| | `finish` | Finir la fonction actuelle |
| | `until [line]` | Continuer jusqu'à la ligne (sortir d'une boucle) |
| **Breakpoints** | `break [where]` | Poser un breakpoint |
| | `break N if cond` | Breakpoint conditionnel |
| | `info breakpoints` | Lister les breakpoints |
| | `delete N` | Supprimer un breakpoint |
| | `disable N` | Désactiver un breakpoint |
| | `enable N` | Réactiver un breakpoint |
| **Inspection** | `print expr` | Afficher une expression |
| | `print/FMT expr` | Afficher dans un format spécifique |
| | `display expr` | Affichage automatique |
| | `watch var` | Watchpoint (arrêt quand var change) |
| | `ptype var` | Définition du type |
| | `whatis var` | Nom du type |
| | `x/NFS addr` | Examiner la mémoire |
| **Contexte** | `backtrace` | Pile d'appels |
| | `frame N` | Aller dans le frame N |
| | `up` / `down` | Naviguer dans les frames |
| | `info locals` | Variables locales |
| | `info args` | Arguments de la fonction |
| | `info registers` | Registres CPU |
| | `list` | Afficher le code source |
| **Modification** | `set variable x = val` | Changer une variable |
| | `call func(args)` | Appeler une fonction |
| | `return val` | Forcer un retour |
| **TUI** | `tui enable` | Activer l'interface graphique |
| | `tui disable` | Désactiver l'interface graphique |
| | `Ctrl+X A` | Basculer le TUI |

---

## Checklist : Méthode anti-segfault

> [!warning] Checklist à suivre quand ton programme crash
> - [ ] As-tu compilé avec `-g` ?
> - [ ] Lance GDB : `gdb -q ./prog`
> - [ ] Lance le programme : `run`
> - [ ] Quand ça crash : `bt` (backtrace)
> - [ ] Va dans le frame du crash : `frame 0`
> - [ ] Regarde les variables : `info locals`
> - [ ] Vérifie les pointeurs : `print ptr` (est-ce `0x0` ?)
> - [ ] Remonte dans la pile : `up` → `info locals` → `info args`
> - [ ] Identifie la cause : NULL pointer ? Buffer overflow ? Use-after-free ?
> - [ ] Corrige, recompile, relance.

---

## Exercices pratiques

### Exercice 1 : Trouver le segfault

Compile et debug ce programme avec GDB :

```c
#include <stdio.h>

void greet(char *name) {
    printf("Hello, %s!\n", name);
}

int main(void) {
    char *name = NULL;
    greet(name);
    return 0;
}
```

1. Compile avec `-g`
2. Lance dans GDB
3. Fais `run`
4. Quand ça crash, fais `bt`
5. Identifie la cause
6. Corrige le code

### Exercice 2 : Boucle infinie

```c
#include <stdio.h>

int main(void) {
    int i = 0;
    while (i < 10)
    {
        printf("i = %d\n", i);
        // Oups, j'ai oublié i++
    }
    return 0;
}
```

1. Lance dans GDB avec un breakpoint sur la boucle
2. Utilise `display i` pour surveiller `i`
3. Fais quelques `next` et observe que `i` ne change jamais
4. Identifie le problème et corrige

### Exercice 3 : Explorer une structure

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char *name;
    int age;
    float grade;
} Student;

int main(void) {
    Student *s = malloc(sizeof(Student));
    s->name = strdup("Alice");
    s->age = 20;
    s->grade = 15.5;

    printf("%s: %d ans, note: %.1f\n", s->name, s->age, s->grade);

    free(s->name);
    free(s);
    return 0;
}
```

1. Pose un breakpoint après l'initialisation de `s`
2. Utilise `print *s` pour voir la structure entière
3. Utilise `print s->name` pour voir le nom
4. Utilise `ptype s` pour voir le type
5. Utilise `x/20xb s` pour voir la mémoire brute de la structure

---

**Liens connexes** : [[02 - Valgrind]]
