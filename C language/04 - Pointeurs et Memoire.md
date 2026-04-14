# 04 - Pointeurs et Mémoire

## La carte de la mémoire virtuelle

Chaque programme C en exécution possède son propre espace de **mémoire virtuelle**, organisé ainsi :

```
  Adresses hautes (0xFFFFFFFF sur 32 bits)
  ┌─────────────────────────────┐
  │         KERNEL              │  Espace noyau (inaccessible au programme)
  ├─────────────────────────────┤
  │                             │
  │         STACK  ↓            │  Pile : variables locales, arguments
  │         (croît vers le bas) │  Automatiquement gérée
  │                             │
  ├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤  ← Espace libre (Stack et Heap se rapprochent)
  │                             │
  │         HEAP   ↑            │  Tas : malloc/calloc/realloc
  │         (croît vers le haut)│  Manuellement gérée (free)
  │                             │
  ├─────────────────────────────┤
  │         BSS                 │  Variables globales non initialisées (→ 0)
  ├─────────────────────────────┤
  │         DATA                │  Variables globales initialisées
  ├─────────────────────────────┤
  │         TEXT (code)         │  Instructions du programme (lecture seule)
  └─────────────────────────────┘
  Adresses basses (0x00000000)
```

| Segment   | Contenu                                    | Durée de vie        |
| --------- | ------------------------------------------ | ------------------- |
| **TEXT**   | Code machine (instructions)                | Toute l'exécution   |
| **DATA**   | Globales initialisées (`int g = 42;`)     | Toute l'exécution   |
| **BSS**    | Globales non initialisées (`int g;` → 0)  | Toute l'exécution   |
| **HEAP**   | Mémoire allouée dynamiquement (`malloc`)  | Jusqu'au `free`     |
| **STACK**  | Variables locales, paramètres, adresses retour | Fin de la fonction |

---

## Stack vs Heap

| Critère               | STACK (Pile)                    | HEAP (Tas)                      |
| --------------------- | ------------------------------- | ------------------------------- |
| **Gestion**           | Automatique (compilateur)       | Manuelle (programmeur)          |
| **Allocation**        | Instantanée (déplacement de pointeur) | Plus lente (recherche de bloc)  |
| **Taille**            | Limitée (~1-8 Mo)              | Limitée par la RAM disponible   |
| **Durée de vie**      | Fin du bloc/fonction            | Jusqu'au `free` explicite       |
| **Fragmentation**     | Aucune (LIFO)                  | Possible                        |
| **Risque**            | Stack overflow                 | Memory leak, dangling pointer   |
| **Accès**             | Très rapide (cache)            | Plus lent                       |
| **Déclaration**       | `int x = 5;`                   | `int *x = malloc(sizeof(int));` |

```c
void exemple(void)
{
    int stack_var = 42;                    /* Sur la STACK */
    int *heap_var = malloc(sizeof(int));   /* Sur le HEAP */

    *heap_var = 42;

    /* stack_var sera automatiquement libérée à la sortie de la fonction */
    /* heap_var DOIT être libérée manuellement */
    free(heap_var);
}
```

> [!warning] Ne jamais retourner l'adresse d'une variable locale !
> ```c
> /* DANGER - Comportement indéfini ! */
> int *bad_function(void)
> {
>     int local = 42;
>     return (&local);  /* local n'existera plus après le return ! */
> }
>
> /* CORRECT - Allouer sur le heap */
> int *good_function(void)
> {
>     int *ptr = malloc(sizeof(int));
>     if (ptr == NULL)
>         return (NULL);
>     *ptr = 42;
>     return (ptr);  /* La mémoire heap persiste */
> }
> ```

---

## Les 4 fonctions d'allocation dynamique

### `malloc` — Memory ALLOCation

```c
#include <stdlib.h>

void *malloc(size_t size);
```

- Alloue `size` octets sur le heap
- Retourne un `void *` (pointeur générique) vers la zone allouée
- Retourne `NULL` si l'allocation échoue
- La mémoire **n'est PAS initialisée** (contient des valeurs aléatoires/garbage)

```c
/* Allouer un int */
int *p = malloc(sizeof(int));

/* Allouer un tableau de 10 int */
int *tab = malloc(sizeof(int) * 10);

/* Allouer une struct */
user_t *user = malloc(sizeof(user_t));

/* Allouer une chaîne de 50 caractères */
char *str = malloc(sizeof(char) * 50);
```

> [!warning] TOUJOURS vérifier le retour de malloc !
> ```c
> int *p = malloc(sizeof(int));
> if (p == NULL)
> {
>     fprintf(stderr, "Erreur: malloc a échoué\n");
>     return (NULL);  /* ou exit(1) */
> }
> ```

> [!tip] Préférer `sizeof(*ptr)` à `sizeof(type)`
> ```c
> int *p = malloc(sizeof(*p));      /* Si le type change, ça s'adapte */
> int *p = malloc(sizeof(int));     /* OK mais moins maintenable */
> ```

### `calloc` — Contiguous ALLOCation

```c
void *calloc(size_t nmemb, size_t size);
```

- Alloue `nmemb` éléments de `size` octets chacun
- **Initialise toute la mémoire à zéro** (contrairement à `malloc`)
- Retourne `NULL` si l'allocation échoue
- Plus lent que `malloc` (à cause de l'initialisation)

```c
/* Allouer 10 int initialisés à 0 */
int *tab = calloc(10, sizeof(int));
if (tab == NULL)
    return (NULL);
/* tab[0] à tab[9] valent tous 0 */

/* Équivalent avec malloc : */
int *tab2 = malloc(sizeof(int) * 10);
if (tab2 == NULL)
    return (NULL);
memset(tab2, 0, sizeof(int) * 10);
```

> [!tip] Quand utiliser calloc vs malloc ?
> - `calloc` : quand on veut que tout soit à **zéro** (tableaux, structs avec valeurs par défaut)
> - `malloc` : quand on va **immédiatement** écrire dans la mémoire (plus rapide)

### `realloc` — RE-ALLOCation

```c
void *realloc(void *ptr, size_t size);
```

- Redimensionne un bloc précédemment alloué
- Peut **déplacer** le bloc en mémoire (copie les données si nécessaire)
- Si `ptr` est `NULL`, se comporte comme `malloc(size)`
- Si `size` est 0, se comporte comme `free(ptr)` (comportement défini par l'implémentation)
- Retourne `NULL` en cas d'échec **sans libérer l'ancien bloc**

> [!warning] DANGER : ne JAMAIS écrire `p = realloc(p, new_size)` !
> Si `realloc` échoue et retourne `NULL`, on **perd** le pointeur original → fuite mémoire !
>
> ```c
> /* FAUX — DANGEREUX */
> p = realloc(p, new_size);  /* Si NULL, on a perdu p ! */
>
> /* CORRECT — Toujours utiliser une variable temporaire */
> tmp = realloc(p, new_size);
> if (tmp == NULL)
> {
>     free(p);  /* On peut encore libérer l'ancien bloc */
>     return (NULL);
> }
> p = tmp;
> ```

Exemple complet :

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *tab;
    int *tmp;
    int i;

    /* Allouer 3 éléments */
    tab = malloc(sizeof(int) * 3);
    if (tab == NULL)
        return (1);

    tab[0] = 10;
    tab[1] = 20;
    tab[2] = 30;

    /* Agrandir à 5 éléments */
    tmp = realloc(tab, sizeof(int) * 5);
    if (tmp == NULL)
    {
        free(tab);
        return (1);
    }
    tab = tmp;

    tab[3] = 40;
    tab[4] = 50;

    for (i = 0; i < 5; i++)
        printf("tab[%d] = %d\n", i, tab[i]);

    free(tab);
    tab = NULL;

    return (0);
}
```

### `free` — Libération de mémoire

```c
void free(void *ptr);
```

- Libère un bloc précédemment alloué par `malloc`, `calloc` ou `realloc`
- `free(NULL)` est sûr (ne fait rien)
- Après `free`, le pointeur pointe toujours vers la même adresse mais la mémoire est **invalide**

Règles de `free` :

| Règle                                    | Conséquence si violée                 |
| ---------------------------------------- | ------------------------------------- |
| Ne free que ce qui a été mallocé         | Comportement indéfini                 |
| Ne free qu'**une seule fois**            | **Double free** → crash/faille       |
| Ne pas utiliser après free               | **Dangling pointer** → données corrompues |
| Toujours mettre à `NULL` après free      | Prévention de double free             |

```c
int *p = malloc(sizeof(int));
*p = 42;

free(p);       /* Libère la mémoire */
p = NULL;      /* Empêche le dangling pointer et le double free */

/* free(NULL) est sûr, donc même si on appelle free(p) à nouveau, pas de crash */
```

---

## Le pattern universel

```
  ALLOUER  →  VÉRIFIER  →  UTILISER  →  LIBÉRER  →  NULLIFIER
  malloc()    if (NULL)     *ptr = ...   free(ptr)   ptr = NULL
```

```c
/* Pattern complet */
int *ptr;

/* 1. Allouer */
ptr = malloc(sizeof(int) * 10);

/* 2. Vérifier */
if (ptr == NULL)
{
    perror("malloc");
    return (1);
}

/* 3. Utiliser */
ptr[0] = 42;
ptr[1] = 84;

/* 4. Libérer */
free(ptr);

/* 5. Nullifier */
ptr = NULL;
```

---

## Table récapitulative des 4 fonctions

| Fonction   | Prototype                              | Initialise ? | Retour si échec  | À retenir                        |
| ---------- | -------------------------------------- | ------------ | ---------------- | -------------------------------- |
| `malloc`   | `void *malloc(size_t size)`            | Non (garbage)| `NULL`           | Le plus courant, rapide          |
| `calloc`   | `void *calloc(size_t n, size_t size)`  | Oui (à 0)   | `NULL`           | Sûr mais plus lent               |
| `realloc`  | `void *realloc(void *p, size_t size)`  | Non          | `NULL` (garde ancien bloc) | Toujours utiliser tmp   |
| `free`     | `void free(void *p)`                   | —            | —                | Puis mettre à NULL               |

---

## Valgrind : détecter les erreurs mémoire

**Valgrind** est un outil indispensable pour détecter les fuites mémoire et les erreurs d'accès.

```bash
# Compiler avec -g (debug info)
gcc -Wall -Werror -Wextra -g main.c -o prog

# Lancer avec Valgrind
valgrind --leak-check=full --show-leak-kinds=all ./prog
```

### Sortie idéale (aucune fuite)

```
==12345== HEAP SUMMARY:
==12345==     in use at exit: 0 bytes in 0 blocks
==12345==   total heap usage: 3 allocs, 3 frees, 1,064 bytes allocated
==12345==
==12345== All heap blocks were freed -- no leaks are possible
==12345==
==12345== ERROR SUMMARY: 0 errors from 0 contexts
```

### Types d'erreurs détectées

| Erreur                        | Description                                   |
| ----------------------------- | --------------------------------------------- |
| `definitely lost`             | Fuite certaine : mémoire allouée, jamais libérée |
| `indirectly lost`             | Perdue car le pointeur parent est perdu       |
| `Invalid read/write`          | Lecture/écriture hors limites                 |
| `Use of uninitialised value`  | Utilisation de mémoire non initialisée        |
| `Invalid free`                | Double free ou free d'un pointeur invalide    |
| `Conditional jump depends on uninitialised value` | Branchement sur valeur non initialisée |

> [!tip] Objectif : toujours obtenir `0 errors` et `no leaks`
> Prendre l'habitude de passer **tous** ses programmes dans Valgrind avant de les soumettre.

---

## Boss 1 : Tableau dynamique d'entiers

```c
#include <stdio.h>
#include <stdlib.h>

/**
 * create_array - Crée un tableau de n entiers initialisés à une valeur
 * @n: Nombre d'éléments
 * @val: Valeur d'initialisation
 *
 * Return: Pointeur vers le tableau, ou NULL
 */
int *create_array(unsigned int n, int val)
{
    int *arr;
    unsigned int i;

    if (n == 0)
        return (NULL);

    arr = malloc(sizeof(int) * n);
    if (arr == NULL)
        return (NULL);

    for (i = 0; i < n; i++)
        arr[i] = val;

    return (arr);
}

int main(void)
{
    int *tab;
    unsigned int i, size = 5;

    tab = create_array(size, 42);
    if (tab == NULL)
        return (1);

    for (i = 0; i < size; i++)
        printf("tab[%u] = %d\n", i, tab[i]);

    free(tab);
    tab = NULL;

    return (0);
}
```

---

## Boss 2 : Duplication de chaîne (strdup maison)

```c
#include <stdio.h>
#include <stdlib.h>

/**
 * _strlen - Calcule la longueur d'une chaîne
 * @s: La chaîne
 *
 * Return: La longueur
 */
int _strlen(char *s)
{
    int len = 0;

    while (s[len])
        len++;

    return (len);
}

/**
 * _strdup - Duplique une chaîne sur le heap
 * @str: La chaîne source
 *
 * Return: Pointeur vers la copie, ou NULL
 */
char *_strdup(char *str)
{
    char *dup;
    int len, i;

    if (str == NULL)
        return (NULL);

    len = _strlen(str);
    dup = malloc(sizeof(char) * (len + 1));  /* +1 pour '\0' */
    if (dup == NULL)
        return (NULL);

    for (i = 0; i <= len; i++)  /* <= pour copier le '\0' */
        dup[i] = str[i];

    return (dup);
}

int main(void)
{
    char *original = "Hello, World!";
    char *copie;

    copie = _strdup(original);
    if (copie == NULL)
        return (1);

    printf("Original : %s\n", original);
    printf("Copie    : %s\n", copie);

    free(copie);
    copie = NULL;

    return (0);
}
```

---

## Boss 3 : Tableau 2D dynamique (matrice)

```c
#include <stdio.h>
#include <stdlib.h>

/**
 * alloc_grid - Alloue une grille 2D initialisée à 0
 * @width: Largeur (colonnes)
 * @height: Hauteur (lignes)
 *
 * Return: Pointeur vers le tableau 2D, ou NULL
 */
int **alloc_grid(int width, int height)
{
    int **grid;
    int i, j;

    if (width <= 0 || height <= 0)
        return (NULL);

    grid = malloc(sizeof(int *) * height);
    if (grid == NULL)
        return (NULL);

    for (i = 0; i < height; i++)
    {
        grid[i] = malloc(sizeof(int) * width);
        if (grid[i] == NULL)
        {
            /* Libérer tout ce qui a été alloué */
            for (j = 0; j < i; j++)
                free(grid[j]);
            free(grid);
            return (NULL);
        }
        for (j = 0; j < width; j++)
            grid[i][j] = 0;
    }

    return (grid);
}

/**
 * free_grid - Libère une grille 2D
 * @grid: La grille
 * @height: Le nombre de lignes
 */
void free_grid(int **grid, int height)
{
    int i;

    if (grid == NULL)
        return;

    for (i = 0; i < height; i++)
        free(grid[i]);   /* Libérer chaque ligne d'abord */
    free(grid);           /* Puis le tableau de pointeurs */
}

int main(void)
{
    int **grid;
    int w = 4, h = 3;
    int i, j;

    grid = alloc_grid(w, h);
    if (grid == NULL)
        return (1);

    grid[1][2] = 42;

    for (i = 0; i < h; i++)
    {
        for (j = 0; j < w; j++)
            printf("%3d ", grid[i][j]);
        printf("\n");
    }

    free_grid(grid, h);

    return (0);
}
```

```
  Visualisation mémoire d'un tableau 2D dynamique :

  grid (int **)
   │
   ▼
  ┌────────┐     ┌─────┬─────┬─────┬─────┐
  │ grid[0]│────▶│  0  │  0  │  0  │  0  │  (malloc ligne 0)
  ├────────┤     └─────┴─────┴─────┴─────┘
  │ grid[1]│────▶┌─────┬─────┬─────┬─────┐
  ├────────┤     │  0  │  0  │ 42  │  0  │  (malloc ligne 1)
  │ grid[2]│──┐  └─────┴─────┴─────┴─────┘
  └────────┘  │  ┌─────┬─────┬─────┬─────┐
   (malloc)   └─▶│  0  │  0  │  0  │  0  │  (malloc ligne 2)
                 └─────┴─────┴─────┴─────┘
```

---

## Erreurs courantes et corrections

### 1. Oublier de vérifier NULL

```c
/* FAUX */
char *s = malloc(100);
strcpy(s, "hello");  /* CRASH si malloc a échoué */

/* CORRECT */
char *s = malloc(100);
if (s == NULL)
    return (NULL);
strcpy(s, "hello");
```

### 2. Fuite mémoire (memory leak)

```c
/* FAUX — la mémoire n'est jamais libérée */
void leak(void)
{
    char *s = malloc(100);
    strcpy(s, "perdu !");
    /* s est perdu quand la fonction retourne */
}

/* CORRECT */
void no_leak(void)
{
    char *s = malloc(100);
    if (s == NULL)
        return;
    strcpy(s, "OK");
    /* ... utilisation ... */
    free(s);
}
```

### 3. Double free

```c
/* FAUX — comportement indéfini / crash */
free(p);
free(p);  /* Double free ! */

/* CORRECT */
free(p);
p = NULL;
free(p);  /* free(NULL) est sûr, ne fait rien */
```

### 4. Dangling pointer (pointeur pendouillant)

```c
/* FAUX */
free(p);
printf("%d\n", *p);  /* Accès à mémoire libérée ! */

/* CORRECT */
free(p);
p = NULL;
/* *p crashera proprement (segfault) au lieu de données corrompues */
```

### 5. Oublier +1 pour le '\0' des chaînes

```c
/* FAUX */
char *dup = malloc(strlen(str));  /* Manque 1 octet pour '\0' */

/* CORRECT */
char *dup = malloc(strlen(str) + 1);
```

### 6. Realloc directement sur le même pointeur

```c
/* FAUX */
p = realloc(p, new_size);  /* Si NULL → p est perdu, fuite ! */

/* CORRECT */
tmp = realloc(p, new_size);
if (tmp == NULL)
{
    free(p);
    return (NULL);
}
p = tmp;
```

---

## Les 6 commandements de la mémoire

> [!example] Les 6 commandements
> 1. **Tu vérifieras** le retour de `malloc` / `calloc` / `realloc` (jamais ignorer `NULL`)
> 2. **Tu libéreras** toute mémoire allouée (`free` pour chaque `malloc`)
> 3. **Tu nullifieras** tout pointeur après `free` (`ptr = NULL`)
> 4. **Tu ne libéreras pas** deux fois le même pointeur (double free)
> 5. **Tu n'accéderas pas** à la mémoire après l'avoir libérée (dangling pointer)
> 6. **Tu utiliseras** Valgrind pour vérifier ton travail

---

## Liens

- [[01 - Introduction au C et Compilation]] — Types de base
- [[02 - Bases Numeriques]] — Adresses en hexadécimal
- [[03 - Structures et Typedef]] — malloc avec les structures
- [[05 - Pointeurs de Fonctions]] — Pointeurs vers des fonctions

---

## Exercices pratiques

### Exercice 1 : `_calloc` maison
Écrire `void *_calloc(unsigned int nmemb, unsigned int size)` en utilisant seulement `malloc` et `memset`.

### Exercice 2 : Tableau dynamique extensible
Créer un programme qui lit des entiers au clavier (un par ligne, arrêt avec `0`) et les stocke dans un tableau dynamique qui grandit avec `realloc` à chaque nouvel élément. Afficher le tableau à la fin.

### Exercice 3 : `_strcat` avec realloc
Écrire `char *string_nconcat(char *s1, char *s2, unsigned int n)` qui concatène s1 et les n premiers octets de s2 dans une nouvelle allocation.

### Exercice 4 : Matrice dynamique
Écrire `int **multiply_matrices(int **m1, int **m2, int r1, int c1, int c2)` qui multiplie deux matrices allouées dynamiquement et retourne le résultat. Ne pas oublier de libérer en cas d'erreur partielle.

### Exercice 5 : Valgrind
Prendre un programme contenant volontairement 3 erreurs mémoire (fuite, double free, accès hors limites) et utiliser Valgrind pour les trouver et les corriger.
