# 05 - Pointeurs de Fonctions

## Définition

Un **pointeur de fonction** est une variable qui stocke l'**adresse mémoire** d'une fonction. Cela permet d'appeler une fonction de manière **indirecte** — on ne sait pas à la compilation quelle fonction sera appelée.

> [!info] Analogie jeu vidéo
> Imagine un bouton configurable dans un jeu. Le bouton est toujours le même (le pointeur), mais l'action qu'il déclenche (la fonction) peut changer : attaquer, défendre, sauter... Le bouton ne fait que **rediriger** vers l'action associée.

---

## Où vivent les fonctions en mémoire ?

Les fonctions sont stockées dans le segment **TEXT** de la mémoire virtuelle (voir [[04 - Pointeurs et Memoire]]).

```
  Mémoire virtuelle
  ┌─────────────────────┐
  │       STACK          │
  ├─────────────────────┤
  │       HEAP           │
  ├─────────────────────┤
  │       BSS            │
  ├─────────────────────┤
  │       DATA           │
  ├─────────────────────┤
  │       TEXT  ◄─────── │  Les fonctions sont ici !
  │  ┌─────────────────┐│
  │  │ main:   0x4005b6││  Chaque fonction a une adresse
  │  │ add:    0x400596││
  │  │ sub:    0x4005a6││
  │  └─────────────────┘│
  └─────────────────────┘
```

Un pointeur de fonction **contient** l'adresse de la première instruction de la fonction dans le segment TEXT.

```c
#include <stdio.h>

int add(int a, int b) { return (a + b); }

int main(void)
{
    printf("Adresse de add : %p\n", (void *)add);
    /* Affiche quelque chose comme : 0x4005b6 */

    return (0);
}
```

---

## Syntaxe : le guide complet

### 1. Déclarer un pointeur de fonction

```c
int (*fptr)(int, int);
```

Décortiquons :
```
  int (*fptr)(int, int);
  ───   ────  ─────────
   │      │       │
   │      │       └── Paramètres de la fonction pointée
   │      └────────── Nom du pointeur (* obligatoire, parenthèses obligatoires)
   └───────────────── Type de retour de la fonction pointée
```

### 2. Assigner une fonction

```c
int add(int a, int b) { return (a + b); }

int (*fptr)(int, int);

fptr = &add;   /* Forme explicite */
fptr = add;    /* Forme implicite (identique, le nom = adresse) */
```

> [!tip] Le nom d'une fonction EST son adresse
> En C, écrire `add` sans parenthèses donne l'**adresse** de la fonction, tout comme le nom d'un tableau donne l'adresse de son premier élément. `&add` et `add` sont équivalents.

### 3. Appeler via le pointeur

```c
int result;

result = fptr(5, 3);      /* Appel direct (forme moderne) */
result = (*fptr)(5, 3);   /* Appel via déréférencement (forme classique) */

/* Les deux sont strictement identiques */
```

---

## Le piège des parenthèses

> [!warning] Les parenthèses changent TOUT !
> ```c
> int (*fptr)(int, int);   /* Pointeur vers une fonction (int, int) → int */
> int *fptr(int, int);     /* Fonction qui retourne un int* (PAS un pointeur de fonction !) */
> ```
>
> Sans les parenthèses autour de `*fptr`, le `*` s'attache au type de retour `int`.

```
  int (*fptr)(int, int)          int *fptr(int, int)
       ▲                              ▲
       │                              │
  "fptr est un POINTEUR           "fptr est une FONCTION
   vers une fonction"              qui retourne un int*"
```

---

## Table de syntaxe complète

| Action                     | Syntaxe                                      |
| -------------------------- | -------------------------------------------- |
| Déclarer                   | `int (*fptr)(int, int);`                     |
| Assigner                   | `fptr = add;` ou `fptr = &add;`             |
| Appeler                    | `fptr(5, 3)` ou `(*fptr)(5, 3)`             |
| Typedef                    | `typedef int (*op_func_t)(int, int);`        |
| Paramètre de fonction      | `void apply(int (*f)(int), int x);`         |
| Tableau de pointeurs       | `int (*ops[4])(int, int);`                   |
| Retourner un pointeur de fn | `int (*get_func(char op))(int, int);`       |

---

## Boss 1 : Appel basique

```c
#include <stdio.h>

/**
 * add - Additionne deux entiers
 * @a: Premier entier
 * @b: Deuxième entier
 *
 * Return: La somme
 */
int add(int a, int b)
{
    return (a + b);
}

/**
 * sub - Soustrait deux entiers
 * @a: Premier entier
 * @b: Deuxième entier
 *
 * Return: La différence
 */
int sub(int a, int b)
{
    return (a - b);
}

/**
 * mul - Multiplie deux entiers
 * @a: Premier entier
 * @b: Deuxième entier
 *
 * Return: Le produit
 */
int mul(int a, int b)
{
    return (a * b);
}

int main(void)
{
    int (*op)(int, int);  /* Déclaration du pointeur de fonction */

    op = add;
    printf("add(5, 3) = %d\n", op(5, 3));  /* 8 */

    op = sub;
    printf("sub(5, 3) = %d\n", op(5, 3));  /* 2 */

    op = mul;
    printf("mul(5, 3) = %d\n", op(5, 3));  /* 15 */

    return (0);
}
```

---

## Boss 2 : Pattern Callback

Un **callback** est une fonction passée en argument à une autre fonction. La fonction appelante "rappelle" (callback) la fonction passée au moment opportun.

```c
#include <stdio.h>
#include <stdlib.h>

/**
 * apply_to_array - Applique une fonction à chaque élément d'un tableau
 * @arr: Le tableau
 * @size: Sa taille
 * @func: La fonction callback à appliquer
 */
void apply_to_array(int *arr, int size, int (*func)(int))
{
    int i;

    if (arr == NULL || func == NULL)
        return;

    for (i = 0; i < size; i++)
        arr[i] = func(arr[i]);
}

int doubler(int x)
{
    return (x * 2);
}

int carre(int x)
{
    return (x * x);
}

int negation(int x)
{
    return (-x);
}

int main(void)
{
    int tab[] = {1, 2, 3, 4, 5};
    int i, size = 5;

    printf("Original : ");
    for (i = 0; i < size; i++)
        printf("%d ", tab[i]);
    printf("\n");

    apply_to_array(tab, size, doubler);
    printf("Doublé   : ");
    for (i = 0; i < size; i++)
        printf("%d ", tab[i]);
    printf("\n");

    apply_to_array(tab, size, carre);
    printf("Carré    : ");
    for (i = 0; i < size; i++)
        printf("%d ", tab[i]);
    printf("\n");

    return (0);
}
```

Sortie :
```
Original : 1 2 3 4 5
Doublé   : 2 4 6 8 10
Carré    : 4 16 36 64 100
```

---

## Boss 3 : Tableau de pointeurs de fonctions + struct

C'est le pattern le plus puissant — utilisé dans les interpréteurs, les shells, `_printf` :

```c
#include <stdio.h>
#include <string.h>

/**
 * struct op_s - Associe un opérateur à une fonction
 * @op: Le symbole de l'opérateur
 * @func: Le pointeur vers la fonction
 */
typedef struct op_s
{
    char *op;
    int (*func)(int, int);
} op_t;

int op_add(int a, int b) { return (a + b); }
int op_sub(int a, int b) { return (a - b); }
int op_mul(int a, int b) { return (a * b); }
int op_div(int a, int b) { return (b != 0 ? a / b : 0); }
int op_mod(int a, int b) { return (b != 0 ? a % b : 0); }

/**
 * get_op_func - Retourne la fonction associée à un opérateur
 * @s: Le symbole de l'opérateur
 *
 * Return: Pointeur vers la fonction, ou NULL
 */
int (*get_op_func(char *s))(int, int)
{
    op_t ops[] = {
        {"+", op_add},
        {"-", op_sub},
        {"*", op_mul},
        {"/", op_div},
        {"%", op_mod},
        {NULL, NULL}
    };
    int i;

    for (i = 0; ops[i].op != NULL; i++)
    {
        if (strcmp(ops[i].op, s) == 0)
            return (ops[i].func);
    }

    return (NULL);
}

int main(void)
{
    int (*operation)(int, int);
    char *symbole = "+";

    operation = get_op_func(symbole);

    if (operation != NULL)
        printf("10 %s 3 = %d\n", symbole, operation(10, 3));
    else
        printf("Opérateur inconnu : %s\n", symbole);

    return (0);
}
```

> [!tip] Ce pattern remplace un `switch` ou une chaîne de `if/else`
> Au lieu de :
> ```c
> if (strcmp(op, "+") == 0)
>     result = a + b;
> else if (strcmp(op, "-") == 0)
>     result = a - b;
> /* ... etc */
> ```
> On utilise un **tableau de dispatch** qui est plus propre, extensible, et maintenable.

---

## Simplifier avec `typedef`

Les déclarations de pointeurs de fonctions sont difficiles à lire. `typedef` les simplifie énormément :

```c
/* Sans typedef (illisible) */
int (*get_op_func(char *s))(int, int);
void sort(int *arr, int n, int (*cmp)(int, int));

/* Avec typedef (lisible) */
typedef int (*op_func_t)(int, int);
typedef int (*cmp_func_t)(int, int);

op_func_t get_op_func(char *s);
void sort(int *arr, int n, cmp_func_t cmp);
```

```c
/* Typedef complet */
typedef int (*op_func_t)(int, int);

op_func_t fptr;         /* Déclarer */
fptr = add;             /* Assigner */
fptr(5, 3);             /* Appeler */

/* Tableau de pointeurs de fonctions */
op_func_t ops[] = {add, sub, mul, op_div};
ops[0](10, 5);          /* Appelle add(10, 5) = 15 */
```

Voir aussi : [[03 - Structures et Typedef]] pour plus de détails sur `typedef`.

---

## Cas d'utilisation dans les projets Holberton

### Projet `_printf`

Le projet `_printf` utilise un tableau de structs associant un caractère de format à une fonction d'impression :

```c
typedef int (*print_func_t)(va_list);

typedef struct format_s
{
    char specifier;
    print_func_t func;
} format_t;

format_t formats[] = {
    {'d', print_int},
    {'s', print_string},
    {'c', print_char},
    {'%', print_percent},
    {0, NULL}
};
```

### Projet Simple Shell

Le shell utilise des pointeurs de fonctions pour les commandes built-in :

```c
typedef struct builtin_s
{
    char *name;
    int (*func)(char **args);
} builtin_t;

builtin_t builtins[] = {
    {"cd", builtin_cd},
    {"exit", builtin_exit},
    {"env", builtin_env},
    {NULL, NULL}
};
```

### `qsort` de la bibliothèque standard

La fonction `qsort` de `<stdlib.h>` prend un callback de comparaison :

```c
#include <stdio.h>
#include <stdlib.h>

int compare_asc(const void *a, const void *b)
{
    return (*(int *)a - *(int *)b);
}

int compare_desc(const void *a, const void *b)
{
    return (*(int *)b - *(int *)a);
}

int main(void)
{
    int tab[] = {42, 7, 23, 1, 89, 15};
    int i, size = 6;

    /* Tri croissant */
    qsort(tab, size, sizeof(int), compare_asc);
    for (i = 0; i < size; i++)
        printf("%d ", tab[i]);
    printf("\n");  /* 1 7 15 23 42 89 */

    /* Tri décroissant */
    qsort(tab, size, sizeof(int), compare_desc);
    for (i = 0; i < size; i++)
        printf("%d ", tab[i]);
    printf("\n");  /* 89 42 23 15 7 1 */

    return (0);
}
```

---

## Les 5 commandements des pointeurs de fonctions

> [!example] Les 5 commandements
> 1. **La signature doit correspondre** — Le type de retour et les paramètres du pointeur doivent être identiques à ceux de la fonction pointée
> 2. **Le nom de la fonction EST son adresse** — `add` et `&add` sont équivalents
> 3. **Pas d'arithmétique** — On ne peut pas faire `fptr++` ou `fptr + 1` (contrairement aux pointeurs de données)
> 4. **Toujours vérifier NULL** — Avant d'appeler via un pointeur, vérifier qu'il n'est pas `NULL`
> 5. **Utiliser typedef** — Dès que la syntaxe devient complexe, `typedef` rend le code lisible

---

## Erreurs courantes

### 1. Oublier les parenthèses dans la déclaration

```c
/* FAUX — Déclare une fonction, pas un pointeur ! */
int *fptr(int, int);

/* CORRECT */
int (*fptr)(int, int);
```

### 2. Signature incompatible

```c
int add(int a, int b);
float (*fptr)(int, int);  /* FAUX — le type de retour ne correspond pas */
int (*fptr)(int);          /* FAUX — le nombre de paramètres ne correspond pas */
int (*fptr)(int, int);     /* CORRECT */
```

### 3. Appeler un pointeur NULL

```c
int (*fptr)(int, int) = NULL;

/* FAUX — crash ! */
fptr(5, 3);

/* CORRECT */
if (fptr != NULL)
    fptr(5, 3);
```

### 4. Confondre assignation et appel

```c
int (*fptr)(int, int);

fptr = add;       /* Assigner (pas de parenthèses) */
fptr = add(5, 3); /* FAUX — ceci appelle add et essaie de stocker le résultat int dans un pointeur ! */
```

### 5. Oublier typedef pour les déclarations complexes

```c
/* Illisible sans typedef */
int (*get_func(char *s))(int, int);
void (*signal(int sig, void (*handler)(int)))(int);

/* Lisible avec typedef */
typedef int (*op_func_t)(int, int);
op_func_t get_func(char *s);

typedef void (*sighandler_t)(int);
sighandler_t signal(int sig, sighandler_t handler);
```

---

## Liens

- [[01 - Introduction au C et Compilation]] — Bases du C
- [[03 - Structures et Typedef]] — typedef et structures
- [[04 - Pointeurs et Memoire]] — Mémoire virtuelle, segment TEXT
- [[06 - Fonctions Variadiques]] — Utilisé ensemble dans `_printf`

---

## Exercices pratiques

### Exercice 1 : Calculatrice avec callback
Écrire une fonction `int calc(int a, int b, int (*op)(int, int))` qui applique l'opération passée en argument. Tester avec add, sub, mul, div.

### Exercice 2 : Tableau de dispatch
Créer un tableau de structs `{char *nom, int (*func)(int, int)}` avec les 4 opérations. Écrire une fonction qui prend un nom d'opération en string et retourne le pointeur de fonction correspondant.

### Exercice 3 : Map sur tableau
Écrire `int *map(int *arr, int n, int (*f)(int))` qui retourne un **nouveau** tableau (alloué avec malloc) où chaque élément est le résultat de `f` appliqué à l'élément original.

### Exercice 4 : Tri générique
Écrire une fonction `void my_sort(int *arr, int n, int (*cmp)(int, int))` qui trie un tableau en utilisant le callback de comparaison (tri à bulles suffit).

### Exercice 5 : Mini interpréteur
Créer un mini interpréteur qui lit des commandes (`add 5 3`, `mul 4 2`) et utilise un tableau de dispatch pour exécuter l'opération correspondante.
