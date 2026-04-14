# Valgrind - Le Détecteur de Fuites Mémoire

## Qu'est-ce que Valgrind ?

**Valgrind** est un outil qui surveille **chaque octet de mémoire** utilisé par ton programme. Il détecte :
- Les **fuites mémoire** (malloc sans free)
- Les **lectures/écritures invalides** (buffer overflow, use-after-free)
- L'utilisation de **variables non initialisées**
- Les **double free**

Valgrind utilise son outil principal **Memcheck** par défaut.

> [!tip] Analogie Gaming
> Imagine un **anti-cheat ultra strict** dans un jeu en ligne :
> - Chaque allocation mémoire est un **objet que tu ramasses** dans le jeu
> - Chaque `free` est quand tu **lâches l'objet**
> - Si tu quittes la partie **avec des objets dans l'inventaire** → fuite mémoire
> - Si tu essaies d'**utiliser un objet que tu as lâché** → use-after-free
> - Si tu **sors de la carte** → accès mémoire invalide
>
> Valgrind, c'est l'anti-cheat qui voit **tout** et te fait un rapport détaillé.

---

## Installation et prérequis

### Installation sur Ubuntu / Debian

```bash
sudo apt update
sudo apt install valgrind
```

### Vérification

```bash
valgrind --version
```

> [!warning] Prérequis obligatoire
> Comme pour GDB, compile **TOUJOURS** avec le flag `-g` pour que Valgrind puisse te montrer les numéros de lignes :
> ```bash
> gcc -Wall -Werror -Wextra -g main.c -o prog
> ```
> Sans `-g`, Valgrind te dira "in main" mais pas **à quelle ligne**.

---

## La commande standard

Voici la commande que tu dois utiliser systématiquement :

```bash
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes --verbose ./prog
```

> [!info] C'est la commande standard Holberton
> Tu la taperas des centaines de fois. Apprends-la par coeur ou crée un alias :
> ```bash
> alias vg='valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes --verbose'
> # Puis utilise simplement :
> vg ./prog
> ```

### Explication de chaque option

| Option | Description |
|---|---|
| `--leak-check=full` | Affiche le détail de **chaque** fuite (où le malloc a été fait) |
| `--show-leak-kinds=all` | Montre **tous** les types de fuites (definite, indirect, possible, still reachable) |
| `--track-origins=yes` | Quand une variable non initialisée est détectée, montre **où** elle a été déclarée |
| `--verbose` | Affiche plus d'informations (bibliothèques chargées, etc.) |

### Autres options utiles

| Option | Description |
|---|---|
| `--log-file=valgrind.log` | Écrire la sortie dans un fichier |
| `--tool=memcheck` | Utiliser Memcheck (c'est le défaut) |
| `--num-callers=20` | Montrer plus de frames dans les stack traces |
| `--suppressions=file.supp` | Ignorer certaines erreurs connues |
| `--gen-suppressions=all` | Générer des suppressions pour chaque erreur |
| `--error-exitcode=1` | Retourner le code 1 s'il y a des erreurs (utile pour CI/CD) |

---

## Lire la sortie de Valgrind

### Le préfixe PID

Chaque ligne de Valgrind commence par `==XXXXX==` où XXXXX est le **PID** (Process ID) du programme.

```
==12345== Memcheck, a memory error detector
==12345== Using Valgrind-3.18.1 and LibVEX
==12345== Command: ./prog
```

> [!info] Ceci n'est PAS une erreur
> Le PID est juste un identifiant. Ignore le numéro, concentre-toi sur le texte après.

### Anatomie de la sortie

```
==12345== HEAP SUMMARY:
==12345==     in use at exit: 100 bytes in 2 blocks
==12345==   total heap usage: 5 allocs, 3 frees, 1,124 bytes allocated
==12345==
==12345== 40 bytes in 1 blocks are definitely lost in loss record 1 of 2
==12345==    at 0x4C2FB0F: malloc (vg_replace_malloc.c:309)
==12345==    by 0x10916A: create_node at main.c:12
==12345==    by 0x1091D3: main at main.c:25
==12345==
==12345== 60 bytes in 1 blocks are definitely lost in loss record 2 of 2
==12345==    at 0x4C2FB0F: malloc (vg_replace_malloc.c:309)
==12345==    by 0x10918C: create_string at main.c:18
==12345==    by 0x1091E5: main at main.c:28
==12345==
==12345== LEAK SUMMARY:
==12345==    definitely lost: 100 bytes in 2 blocks
==12345==    indirectly lost: 0 bytes in 0 blocks
==12345==      possibly lost: 0 bytes in 0 blocks
==12345==    still reachable: 0 bytes in 0 blocks
==12345==         suppressed: 0 bytes in 0 blocks
==12345==
==12345== ERROR SUMMARY: 2 errors from 2 contexts
```

**Comment lire** :
1. **HEAP SUMMARY** : 5 mallocs mais seulement 3 frees → 2 fuites
2. **Détail des fuites** : chaque bloc te montre la **pile d'appels** où le malloc a été fait
3. **LEAK SUMMARY** : résumé par catégorie
4. **ERROR SUMMARY** : nombre total d'erreurs

---

## Les 4 types de fuites mémoire

### 1. Definitely lost (Définitivement perdu)

Le programme a **perdu toute référence** à cette mémoire. Il n'y a **aucun pointeur** qui pointe encore dessus. C'est un vrai bug.

```c
void definitely_lost(void) {
    char *str = malloc(100);
    str = NULL;  // On perd le pointeur → la mémoire est perdue à jamais
}
```

```
==12345== 100 bytes in 1 blocks are definitely lost
==12345==    at 0x4C2FB0F: malloc
==12345==    by 0x10916A: definitely_lost at main.c:3
```

> [!warning] C'est le type de fuite le plus grave. Il faut TOUJOURS corriger les "definitely lost".

### 2. Indirectly lost (Indirectement perdu)

La mémoire est perdue parce qu'on a perdu le **pointeur parent** qui menait à cette mémoire. Corrige le "definitely lost" parent, et l'indirect se résoudra aussi.

```c
typedef struct node {
    int data;
    struct node *next;
} Node;

void indirectly_lost(void) {
    Node *head = malloc(sizeof(Node));   // definitely lost
    head->next = malloc(sizeof(Node));   // indirectly lost
    head->next->next = NULL;
    head = NULL;  // On perd head → head->next est indirectement perdu
}
```

```
==12345== 16 bytes in 1 blocks are definitely lost
==12345==    by 0x10916A: indirectly_lost at main.c:8 (head)
==12345==
==12345== 16 bytes in 1 blocks are indirectly lost
==12345==    by 0x109182: indirectly_lost at main.c:9 (head->next)
```

### 3. Possibly lost (Possiblement perdu)

Le pointeur **existe encore** mais pointe **au milieu** du bloc alloué (pas au début). Valgrind n'est pas sûr si c'est intentionnel.

```c
void possibly_lost(void) {
    char *str = malloc(100);
    str += 50;  // Le pointeur pointe au milieu du bloc
    // On ne peut plus faire free(str) correctement
}
```

```
==12345== 100 bytes in 1 blocks are possibly lost
==12345==    at 0x4C2FB0F: malloc
==12345==    by 0x10916A: possibly_lost at main.c:3
```

### 4. Still reachable (Encore accessible)

La mémoire **n'a pas été libérée** mais le pointeur **existe encore** au moment où le programme se termine. C'est souvent acceptable (mémoire libérée par l'OS à la fin).

```c
// Variable globale
char *global_str;

int main(void) {
    global_str = malloc(100);
    strcpy(global_str, "Hello");
    // Le programme se termine sans free(global_str)
    // Mais global_str pointe toujours vers la mémoire
    return 0;
}
```

```
==12345== 100 bytes in 1 blocks are still reachable
==12345==    at 0x4C2FB0F: malloc
==12345==    by 0x10916A: main at main.c:5
```

> [!info] Priorité de correction
> 1. **definitely lost** → TOUJOURS corriger
> 2. **indirectly lost** → Se corrige quand on corrige le definitely lost parent
> 3. **possibly lost** → Généralement un bug, à corriger
> 4. **still reachable** → Corriger si ton école/projet l'exige, sinon c'est souvent OK

---

## Erreurs mémoire (pas des fuites)

### Invalid read / Invalid write (Buffer overflow)

```c
int main(void) {
    int *arr = malloc(3 * sizeof(int));  // 3 éléments
    arr[0] = 10;
    arr[1] = 20;
    arr[2] = 30;
    arr[3] = 40;  // ERREUR : écriture hors limites !
    int x = arr[3];  // ERREUR : lecture hors limites !
    free(arr);
    return 0;
}
```

```
==12345== Invalid write of size 4
==12345==    at 0x10917E: main (main.c:6)
==12345==  Address 0x522d04c is 0 bytes after a block of size 12 alloc'd
==12345==    at 0x4C2FB0F: malloc (vg_replace_malloc.c:309)
==12345==    by 0x109155: main (main.c:2)
==12345==
==12345== Invalid read of size 4
==12345==    at 0x109192: main (main.c:7)
==12345==  Address 0x522d04c is 0 bytes after a block of size 12 alloc'd
```

**Comment lire** : "0 bytes after a block of size 12" = tu as écrit **juste après** la fin d'un bloc de 12 octets (3 * 4 = 12).

### Use-after-free

```c
int main(void) {
    int *ptr = malloc(sizeof(int));
    *ptr = 42;
    free(ptr);
    printf("%d\n", *ptr);  // ERREUR : utilisation après free !
    return 0;
}
```

```
==12345== Invalid read of size 4
==12345==    at 0x10917E: main (main.c:5)
==12345==  Address 0x522d040 is 0 bytes inside a block of size 4 free'd
==12345==    at 0x4C30D3B: free (vg_replace_malloc.c:538)
==12345==    by 0x109172: main (main.c:4)
==12345==  Block was alloc'd at
==12345==    at 0x4C2FB0F: malloc (vg_replace_malloc.c:309)
==12345==    by 0x109155: main (main.c:2)
```

**Comment lire** : "inside a block of size 4 **free'd**" → tu accèdes à un bloc qui a **déjà été libéré**.

### Conditional jump depends on uninitialised value

```c
int main(void) {
    int x;
    if (x > 0)  // ERREUR : x n'est pas initialisé !
        printf("positive\n");
    return 0;
}
```

```
==12345== Conditional jump or move depends on uninitialised value(s)
==12345==    at 0x10916A: main (main.c:3)
==12345==  Uninitialised value was created by a stack allocation
==12345==    at 0x109155: main (main.c:2)
```

> [!tip] `--track-origins=yes` te dit **où** la variable a été créée. Sans cette option, tu ne saurais pas que c'est `x` déclarée à la ligne 2.

### Invalid free / double free

```c
int main(void) {
    int *ptr = malloc(sizeof(int));
    free(ptr);
    free(ptr);  // ERREUR : double free !
    return 0;
}
```

```
==12345== Invalid free() / delete / delete[] / realloc()
==12345==    at 0x4C30D3B: free (vg_replace_malloc.c:538)
==12345==    by 0x109182: main (main.c:4)
==12345==  Address 0x522d040 is 0 bytes inside a block of size 4 free'd
==12345==    at 0x4C30D3B: free (vg_replace_malloc.c:538)
==12345==    by 0x109172: main (main.c:3)
```

---

## Lire le HEAP SUMMARY

```
==12345== HEAP SUMMARY:
==12345==     in use at exit: 100 bytes in 2 blocks
==12345==   total heap usage: 5 allocs, 3 frees, 1,124 bytes allocated
```

### Le calcul simple

| Donnée | Signification |
|---|---|
| `5 allocs` | 5 appels à malloc/calloc/realloc/strdup |
| `3 frees` | 3 appels à free |
| `5 - 3 = 2` | **2 blocs non libérés** → fuites ! |
| `100 bytes in 2 blocks` | Ces 2 blocs représentent 100 octets |
| `1,124 bytes allocated` | Total de mémoire allouée (inclut les allocations internes de printf, etc.) |

> [!warning] Objectif : `allocs == frees`
> Quand ton HEAP SUMMARY montre :
> ```
> total heap usage: 5 allocs, 5 frees, 1,124 bytes allocated
> ```
> Et :
> ```
> All heap blocks were freed -- no leaks are possible
> ```
> C'est **parfait** ! Zéro fuite.

### calloc, realloc, strdup aussi comptés

Valgrind ne suit pas **que** `malloc` et `free`. Il suit aussi :

| Fonction | Compte comme |
|---|---|
| `malloc(n)` | 1 alloc |
| `calloc(n, size)` | 1 alloc |
| `realloc(ptr, n)` | 1 alloc (et 1 free de l'ancien bloc si ptr != NULL) |
| `strdup(s)` | 1 alloc (car strdup fait un malloc interne) |
| `free(ptr)` | 1 free |

---

## 5 cas pratiques avec sortie Valgrind

### Cas 1 : Fuite dans une liste chaînée

```c
#include <stdlib.h>

typedef struct node {
    int data;
    struct node *next;
} Node;

Node *create_node(int data) {
    Node *new = malloc(sizeof(Node));
    if (!new)
        return NULL;
    new->data = data;
    new->next = NULL;
    return new;
}

int main(void) {
    Node *head = create_node(1);
    head->next = create_node(2);
    head->next->next = create_node(3);

    // BUG : on free seulement head !
    free(head);
    return 0;
}
```

**Sortie Valgrind** :
```
==12345== HEAP SUMMARY:
==12345==   total heap usage: 3 allocs, 1 frees, 48 bytes allocated
==12345==
==12345== 16 bytes in 1 blocks are definitely lost
==12345==    by 0x109182: create_node (main.c:9)
==12345==    by 0x1091D8: main (main.c:20)
==12345==
==12345== 16 bytes in 1 blocks are definitely lost
==12345==    by 0x109182: create_node (main.c:9)
==12345==    by 0x1091F0: main (main.c:21)
```

**Correction** :
```c
// Libérer toute la liste
Node *current = head;
while (current) {
    Node *tmp = current->next;
    free(current);
    current = tmp;
}
```

### Cas 2 : malloc sans +1 pour le `\0`

```c
#include <stdlib.h>
#include <string.h>

int main(void) {
    char *src = "Hello";
    char *dest = malloc(strlen(src));  // BUG : 5 au lieu de 6 !
    strcpy(dest, src);
    free(dest);
    return 0;
}
```

**Sortie Valgrind** :
```
==12345== Invalid write of size 1
==12345==    at 0x4C32E0A: strcpy (vg_replace_strmem.c:510)
==12345==    by 0x10917E: main (main.c:6)
==12345==  Address 0x522d045 is 0 bytes after a block of size 5 alloc'd
```

"0 bytes after a block of size 5" = tu écris le `\0` **juste après** les 5 octets alloués.

**Correction** :
```c
char *dest = malloc(strlen(src) + 1);  // +1 pour le '\0'
```

### Cas 3 : Variable non initialisée

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int *arr = malloc(5 * sizeof(int));
    int sum = 0;
    for (int i = 0; i < 5; i++)
        sum += arr[i];  // BUG : arr n'est pas initialisé !
    printf("sum = %d\n", sum);
    free(arr);
    return 0;
}
```

**Sortie Valgrind** :
```
==12345== Conditional jump or move depends on uninitialised value(s)
==12345==    at 0x4E988DA: vfprintf (vfprintf.c:1642)
==12345==    by 0x4EA0F25: printf (printf.c:33)
==12345==    by 0x1091B2: main (main.c:9)
==12345==  Uninitialised value was created by a heap allocation
==12345==    at 0x4C2FB0F: malloc
==12345==    by 0x109165: main (main.c:5)
```

**Correction** : Utiliser `calloc` (initialise à 0) au lieu de `malloc` :
```c
int *arr = calloc(5, sizeof(int));
```

### Cas 4 : Free partiel d'une struct

```c
#include <stdlib.h>
#include <string.h>

typedef struct {
    char *name;
    char *city;
} Person;

int main(void) {
    Person *p = malloc(sizeof(Person));
    p->name = strdup("Alice");
    p->city = strdup("Paris");

    // BUG : on oublie de free name et city !
    free(p);
    return 0;
}
```

**Sortie Valgrind** :
```
==12345== 6 bytes in 1 blocks are definitely lost
==12345==    by 0x4C2FB0F: malloc
==12345==    by 0x4C32D69: strdup
==12345==    by 0x10917E: main (main.c:11)
==12345==
==12345== 6 bytes in 1 blocks are definitely lost
==12345==    by 0x4C2FB0F: malloc
==12345==    by 0x4C32D69: strdup
==12345==    by 0x109196: main (main.c:12)
```

**Correction** : Libérer les champs **avant** la struct :
```c
free(p->name);
free(p->city);
free(p);
```

### Cas 5 : realloc sans variable temporaire

```c
#include <stdlib.h>

int main(void) {
    int *arr = malloc(5 * sizeof(int));

    // BUG : si realloc échoue, on perd le pointeur original !
    arr = realloc(arr, 100 * sizeof(int));
    if (!arr)
        return 1;  // Fuite : l'ancien arr est perdu !

    free(arr);
    return 0;
}
```

**Correction** : Toujours utiliser une variable temporaire :
```c
int *tmp = realloc(arr, 100 * sizeof(int));
if (!tmp) {
    free(arr);  // On peut encore libérer l'ancien bloc
    return 1;
}
arr = tmp;
```

---

## Bonnes pratiques mémoire

> [!tip] Les règles d'or

### 1. Un malloc = un free

Chaque `malloc` (ou `calloc`, `strdup`) doit avoir un `free` correspondant. Toujours.

### 2. +1 pour les chaînes

```c
// MAUVAIS
char *copy = malloc(strlen(src));

// BON
char *copy = malloc(strlen(src) + 1);
```

### 3. Libérer dans l'ordre inverse

```c
// Allocation
Person *p = malloc(sizeof(Person));
p->name = strdup("Alice");

// Libération (inverse)
free(p->name);   // D'abord les enfants
free(p);          // Ensuite le parent
```

> [!warning] Si tu fais `free(p)` en premier, `p->name` est perdu et tu as une fuite !

### 4. Gérer les chemins d'erreur

```c
char *a = malloc(100);
if (!a)
    return -1;

char *b = malloc(200);
if (!b) {
    free(a);       // N'oublie pas de free a !
    return -1;
}

// ... utilisation ...

free(b);
free(a);
```

### 5. Initialiser les variables

```c
int x = 0;           // Initialisé
char *ptr = NULL;     // Initialisé à NULL
int arr[10] = {0};   // Tout à zéro
```

---

## Quiz : 12 questions

### Questions

**Q1.** Quelle commande lance Valgrind avec toutes les options de leak check ?

**Q2.** Combien de fuites dans ce HEAP SUMMARY ?
```
total heap usage: 7 allocs, 4 frees, 2048 bytes allocated
```

**Q3.** Quelle est la différence entre "definitely lost" et "still reachable" ?

**Q4.** Que signifie cette erreur ?
```
Invalid write of size 4
  Address 0x522d04c is 0 bytes after a block of size 12 alloc'd
```

**Q5.** Pourquoi faut-il compiler avec `-g` pour Valgrind ?

**Q6.** `strdup("Hello")` compte comme combien d'allocations ?

**Q7.** Comment corriger : `char *s = malloc(strlen(src));` suivi de `strcpy(s, src);` ?

**Q8.** Pourquoi ne faut-il PAS faire `arr = realloc(arr, new_size);` directement ?

**Q9.** Dans quel ordre faut-il libérer une struct qui contient des pointeurs alloués ?

**Q10.** Que signifie "Conditional jump depends on uninitialised value" ?

**Q11.** Comment savoir si tu as un use-after-free dans la sortie Valgrind ?

**Q12.** Quel est l'objectif au HEAP SUMMARY ?

### Réponses

**R1.** `valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes --verbose ./prog`

**R2.** 7 - 4 = **3 fuites** (3 blocs non libérés)

**R3.** "Definitely lost" = aucun pointeur ne pointe vers la mémoire (bug certain). "Still reachable" = un pointeur existe encore à la fin du programme (souvent acceptable).

**R4.** On écrit 4 octets **juste après** un bloc de 12 octets → buffer overflow. Probablement un accès `arr[3]` sur un tableau de 3 éléments (3 * 4 = 12).

**R5.** Pour que Valgrind puisse montrer les **numéros de lignes** et **noms de fichiers** dans les stack traces.

**R6.** **1 allocation** (strdup fait un malloc interne).

**R7.** `char *s = malloc(strlen(src) + 1);` → ajouter **+1** pour le caractère nul `\0`.

**R8.** Si realloc échoue et retourne NULL, on **perd le pointeur original** et la mémoire est définitivement perdue. Il faut utiliser une variable temporaire.

**R9.** D'abord libérer les **champs** (enfants), puis la **struct elle-même** (parent). Ordre inverse de l'allocation.

**R10.** Un `if`, `while`, ou autre condition utilise une variable qui n'a **jamais été initialisée**. La valeur est aléatoire (garbage).

**R11.** Le message dira "inside a block of size N **free'd**" → accès à un bloc qui a déjà été libéré.

**R12.** `allocs == frees` et le message "All heap blocks were freed -- no leaks are possible".

---

## Carte mentale : Règles mémoire

```
                    GESTION MÉMOIRE EN C
                          |
          ┌───────────────┼───────────────┐
          |               |               |
      ALLOCATION       UTILISATION     LIBÉRATION
          |               |               |
    ┌─────┼─────┐    ┌────┼────┐     ┌────┼────┐
    |     |     |    |    |    |     |    |    |
  malloc calloc strdup  +1   init  free  ordre  tmp
    |     |     |    |    |    |     |    |    |
    |   init=0  |  '\0' check |   1:1  enfant realloc
    |           |        NULL  |        avant
    |           |              |        parent
    └───────────┴──────────────┘
              |
        Vérifier retour != NULL
```

### Les règles en résumé

> [!warning] Mémorise ces règles
> 1. **1 malloc = 1 free** (toujours)
> 2. **+1 pour les strings** (`strlen(s) + 1`)
> 3. **Libérer enfants avant parents** (ordre inverse)
> 4. **Vérifier le retour** de malloc/calloc/realloc (peut retourner NULL)
> 5. **Initialiser les variables** (ou utiliser calloc)
> 6. **realloc dans une variable tmp** (jamais directement)
> 7. **`ptr = NULL` après free** (évite use-after-free)
> 8. **Gérer les chemins d'erreur** (free tout ce qui a été alloué)
> 9. **Ne jamais free deux fois** le même pointeur
> 10. **Compiler avec `-g`** pour Valgrind et GDB

---

## Exercices

### Exercice 1 : Trouver toutes les fuites

Compile ce programme et lance Valgrind. Combien de fuites y a-t-il ? Corrige-les toutes.

```c
#include <stdlib.h>
#include <string.h>

typedef struct {
    char *first_name;
    char *last_name;
    int age;
} Student;

Student *create_student(const char *first, const char *last, int age) {
    Student *s = malloc(sizeof(Student));
    s->first_name = strdup(first);
    s->last_name = strdup(last);
    s->age = age;
    return s;
}

int main(void) {
    Student *s1 = create_student("Alice", "Dupont", 20);
    Student *s2 = create_student("Bob", "Martin", 22);

    // On free seulement s1... et mal
    free(s1);

    return 0;
}
```

### Exercice 2 : Corriger le buffer overflow

```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

int main(void) {
    char *greeting = malloc(5);
    strcpy(greeting, "Hello");
    printf("%s\n", greeting);
    free(greeting);
    return 0;
}
```

### Exercice 3 : La liste chaînée parfaite

Écris une fonction `free_list` qui libère correctement toute une liste chaînée, puis vérifie avec Valgrind que tu as zéro fuite.

---

**Liens connexes** : [[01 - GDB]] | [[01 - Securite Memoire en C]]
