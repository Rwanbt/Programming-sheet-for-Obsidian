# Les Pointeurs en C

---

## Introduction

Les pointeurs sont **le** concept central du C. Ils permettent de manipuler directement la memoire, de creer des structures de donnees complexes, et de passer efficacement des donnees aux fonctions. Ce guide couvre les bases des pointeurs - la gestion dynamique de la memoire (malloc/free) est traitee dans [[04 - Pointeurs et Memoire]].

> [!warning] Prerequis
> Avant de lire cette note, tu dois comprendre :
> - Les types de base en C (int, char, float)
> - Les fonctions en C
> - Les tableaux
> Voir [[01 - Introduction au C et Compilation]] si besoin.

---

## 1. Qu'est-ce qu'un pointeur ?

### 1.1 L'analogie de l'adresse postale

> [!tip] Analogie
> Imagine que chaque variable en C est une **maison**.
> - La **maison** contient une valeur (le contenu)
> - La maison a une **adresse** (son emplacement en memoire)
> - Un **pointeur** est un bout de papier sur lequel tu ecris l'adresse d'une maison
>
> Le pointeur ne contient pas la valeur. Il contient **l'adresse** ou se trouve la valeur.

### 1.2 Definition technique

Un pointeur est une **variable qui stocke l'adresse memoire** d'une autre variable.

```c
int x = 42;      /* x est une variable qui contient 42 */
int *ptr = &x;   /* ptr est un pointeur qui contient l'adresse de x */
```

En memoire, ca donne :

```
  Variable x                  Pointeur ptr
  +--------+                  +--------+
  |   42   |                  | 0x1000 |---->  pointe vers x
  +--------+                  +--------+
  Adresse: 0x1000             Adresse: 0x2000
```

---

## 2. Declarer un pointeur

### 2.1 Syntaxe

```c
type *nom_du_pointeur;
```

Le `*` dans la declaration signifie "ceci est un pointeur vers `type`".

```c
int *p;       /* pointeur vers un int */
char *c;      /* pointeur vers un char */
float *f;     /* pointeur vers un float */
double *d;    /* pointeur vers un double */
```

> [!warning] Position de l'etoile
> Ces trois ecritures sont **equivalentes** :
> ```c
> int *ptr;    /* Style recommande : * colle au nom */
> int* ptr;    /* * colle au type */
> int * ptr;   /* * au milieu */
> ```
> **Piege** avec la deuxieme forme :
> ```c
> int* a, b;   /* ATTENTION : a est un pointeur, mais b est un int ! */
> int *a, *b;  /* Les deux sont des pointeurs */
> ```

### 2.2 Initialisation a NULL

> [!warning] Toujours initialiser un pointeur
> Un pointeur non initialise contient une adresse **aleatoire** (garbage). Le dereferencer provoque un **comportement indefini** (crash, corruption memoire, etc.).

```c
int *ptr;          /* DANGEREUX : pointeur non initialise (wild pointer) */
int *ptr = NULL;   /* CORRECT : initialise a NULL */
```

`NULL` est defini dans `<stddef.h>` (et `<stdlib.h>`, `<stdio.h>`). C'est l'adresse `0`, qui par convention signifie "ce pointeur ne pointe vers rien".

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *ptr = NULL;

    printf("ptr = %p\n", (void *)ptr);  /* ptr = (nil) ou 0x0 */

    if (ptr == NULL)
        printf("Le pointeur est NULL\n");

    return (0);
}
```

---

## 3. L'operateur & (adresse de)

L'operateur `&` retourne **l'adresse memoire** d'une variable.

```c
#include <stdio.h>

int main(void)
{
    int x = 42;

    printf("Valeur de x   : %d\n", x);     /* 42 */
    printf("Adresse de x  : %p\n", (void *)&x);  /* ex: 0x7ffd5e8a3b2c */

    /* Stocker l'adresse dans un pointeur */
    int *ptr = &x;

    printf("Valeur de ptr : %p\n", (void *)ptr);  /* meme adresse que &x */

    return (0);
}
```

> [!info] Le format %p
> `%p` affiche une adresse memoire en hexadecimal. Il faut caster en `(void *)` pour etre conforme au standard C.

```
Resultat :
Valeur de x   : 42
Adresse de x  : 0x7ffd5e8a3b2c
Valeur de ptr : 0x7ffd5e8a3b2c
```

---

## 4. L'operateur * (dereferencement)

L'operateur `*` (quand utilise **en dehors** d'une declaration) permet de **lire ou ecrire** la valeur a l'adresse pointee.

```c
#include <stdio.h>

int main(void)
{
    int x = 42;
    int *ptr = &x;

    /* LIRE la valeur pointee */
    printf("Valeur pointee par ptr : %d\n", *ptr);  /* 42 */

    /* ECRIRE via le pointeur */
    *ptr = 100;
    printf("Nouvelle valeur de x   : %d\n", x);     /* 100 ! */

    return (0);
}
```

> [!warning] Ne pas confondre * en declaration et * en dereferencement
> ```c
> int *ptr = &x;   /* * = declaration (ptr est un pointeur) */
> *ptr = 42;       /* * = dereferencement (ecrire a l'adresse pointee) */
> int y = *ptr;    /* * = dereferencement (lire la valeur pointee) */
> ```

### 4.1 Visualisation memoire complete

```c
int a = 10;
int b = 20;
int *p = &a;
```

```
MEMOIRE (representation simplifiee)
Adresse     Variable    Valeur
+---------+-----------+---------+
| 0x1000  |    a      |   10    |
+---------+-----------+---------+
| 0x1004  |    b      |   20    |
+---------+-----------+---------+
| 0x1008  |    p      | 0x1000  |----> pointe vers a
+---------+-----------+---------+

Apres : p = &b;
+---------+-----------+---------+
| 0x1000  |    a      |   10    |
+---------+-----------+---------+
| 0x1004  |    b      |   20    |
+---------+-----------+---------+
| 0x1008  |    p      | 0x1004  |----> pointe vers b maintenant
+---------+-----------+---------+

Apres : *p = 99;
+---------+-----------+---------+
| 0x1000  |    a      |   10    |
+---------+-----------+---------+
| 0x1004  |    b      |   99    |  <-- modifie via *p !
+---------+-----------+---------+
| 0x1008  |    p      | 0x1004  |----> pointe toujours vers b
+---------+-----------+---------+
```

---

## 5. Pointeurs et types

### 5.1 Pourquoi le type du pointeur compte

Un `int *` et un `char *` sont tous les deux des adresses (8 octets sur 64-bit). Alors pourquoi le type ?

**Raison 1 : sizeof et lecture en memoire**

Le type dit au compilateur **combien d'octets lire** a partir de l'adresse.

```
int *p pointe vers 0x1000 :
                0x1000  0x1001  0x1002  0x1003
                +-------+-------+-------+-------+
  *p lit 4     |  0x2A  |  0x00  |  0x00  |  0x00  |  = 42 (int)
  octets       +-------+-------+-------+-------+

char *c pointe vers 0x1000 :
                0x1000
                +-------+
  *c lit 1     |  0x2A  |  = 42 (char) ou '*'
  octet        +-------+
```

**Raison 2 : arithmetique des pointeurs**

```c
int *p = (int *)0x1000;
p++;   /* p = 0x1004  (avance de sizeof(int) = 4 octets) */

char *c = (char *)0x1000;
c++;   /* c = 0x1001  (avance de sizeof(char) = 1 octet) */
```

### 5.2 void* (pointeur generique)

`void *` est un pointeur **generique** qui peut pointer vers n'importe quel type. On ne peut pas le dereferencer directement.

```c
#include <stdio.h>

void print_value(void *ptr, char type)
{
    if (type == 'i')
        printf("int: %d\n", *(int *)ptr);
    else if (type == 'c')
        printf("char: %c\n", *(char *)ptr);
    else if (type == 'f')
        printf("float: %.2f\n", *(float *)ptr);
}

int main(void)
{
    int x = 42;
    char c = 'A';
    float f = 3.14;

    print_value(&x, 'i');   /* int: 42 */
    print_value(&c, 'c');   /* char: A */
    print_value(&f, 'f');   /* float: 3.14 */

    return (0);
}
```

> [!info] Ou voit-on void* ?
> - `malloc()` retourne un `void *`
> - `memcpy()`, `memset()` prennent des `void *`
> - Les fonctions generiques (comme `qsort()`)

---

## 6. Pointeurs et tableaux

### 6.1 Le lien fondamental

En C, le nom d'un tableau **decroit** (decay) en un pointeur vers son premier element.

```c
int arr[5] = {10, 20, 30, 40, 50};

/* Ces deux expressions sont equivalentes */
arr     == &arr[0]     /* adresse du premier element */
arr[0]  == *arr        /* valeur du premier element */
arr[i]  == *(arr + i)  /* valeur du i-eme element */
```

```
arr pointe ici
    |
    v
+------+------+------+------+------+
|  10  |  20  |  30  |  40  |  50  |
+------+------+------+------+------+
arr[0]  arr[1] arr[2] arr[3] arr[4]
arr+0   arr+1  arr+2  arr+3  arr+4
```

### 6.2 Demonstration

```c
#include <stdio.h>

int main(void)
{
    int arr[5] = {10, 20, 30, 40, 50};
    int *ptr = arr;   /* equivalent a : int *ptr = &arr[0]; */

    printf("arr     = %p\n", (void *)arr);      /* adresse du tableau */
    printf("&arr[0] = %p\n", (void *)&arr[0]);  /* meme adresse */
    printf("ptr     = %p\n", (void *)ptr);       /* meme adresse */

    printf("\n");

    /* Trois facons d'acceder au meme element */
    printf("arr[2]      = %d\n", arr[2]);        /* 30 */
    printf("*(arr + 2)  = %d\n", *(arr + 2));    /* 30 */
    printf("*(ptr + 2)  = %d\n", *(ptr + 2));    /* 30 */
    printf("ptr[2]      = %d\n", ptr[2]);        /* 30 - oui, ca marche ! */

    return (0);
}
```

> [!warning] Tableau != Pointeur
> Meme si `arr` et `ptr` semblent interchangeables, il y a des differences :
> ```c
> sizeof(arr)   /* = 20 (5 * 4 octets) = taille du tableau entier */
> sizeof(ptr)   /* = 8 (sur 64-bit) = taille d'un pointeur */
>
> arr = ptr;    /* ERREUR : arr n'est pas assignable */
> ptr = arr;    /* OK : ptr est assignable */
> ```
> Un tableau est un **espace memoire fixe**. Un pointeur est une **variable** qui contient une adresse.

### 6.3 Decay to pointer

Quand tu passes un tableau a une fonction, il **decay** en pointeur :

```c
void print_array(int *arr, int size)   /* arr est un pointeur, pas un tableau */
{
    /* sizeof(arr) = 8 (taille du pointeur, PAS du tableau) */
    for (int i = 0; i < size; i++)
        printf("%d ", arr[i]);
    printf("\n");
}

/* Ces deux prototypes sont IDENTIQUES */
void func(int arr[]);    /* le compilateur le transforme en int *arr */
void func(int *arr);     /* c'est la forme reelle */

/* Meme ca est identique */
void func(int arr[10]);  /* le 10 est IGNORE par le compilateur */
```

---

## 7. Arithmetique des pointeurs

### 7.1 ptr + n et ptr - n

L'arithmetique avance de **n * sizeof(type)** octets.

```c
int arr[5] = {10, 20, 30, 40, 50};
int *ptr = arr;

/* ptr + 0 → adresse de arr[0] → 0x1000 */
/* ptr + 1 → adresse de arr[1] → 0x1004 (pas 0x1001 !) */
/* ptr + 2 → adresse de arr[2] → 0x1008 */
/* ptr + 3 → adresse de arr[3] → 0x100C */
/* ptr + 4 → adresse de arr[4] → 0x1010 */
```

```
Memoire (int = 4 octets) :

ptr+0       ptr+1       ptr+2       ptr+3       ptr+4
|           |           |           |           |
v           v           v           v           v
+----+----+----+----+----+----+----+----+----+----+----+----+----+
| 10 | 10 | 10 | 10 | 20 | 20 | 20 | 20 | 30 | 30 | 30 | 30 | ...
+----+----+----+----+----+----+----+----+----+----+----+----+----+
0x1000          0x1004          0x1008          0x100C
```

### 7.2 ptr++ et ptr--

```c
#include <stdio.h>

int main(void)
{
    int arr[5] = {10, 20, 30, 40, 50};
    int *ptr = arr;

    /* Parcourir le tableau avec ptr++ */
    for (int i = 0; i < 5; i++)
    {
        printf("ptr = %p, *ptr = %d\n", (void *)ptr, *ptr);
        ptr++;   /* avance au prochain int (4 octets) */
    }

    return (0);
}
```

### 7.3 Soustraction de deux pointeurs

La soustraction de deux pointeurs du meme type donne le **nombre d'elements** entre eux.

```c
int arr[5] = {10, 20, 30, 40, 50};
int *p1 = &arr[1];
int *p2 = &arr[4];

printf("Distance : %ld\n", p2 - p1);  /* 3 (elements, pas octets) */
```

### 7.4 Parcours idiomatique avec pointeurs

```c
#include <stdio.h>

int main(void)
{
    int arr[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int size = sizeof(arr) / sizeof(arr[0]);
    int *ptr;

    /* Parcours classique avec index */
    for (int i = 0; i < size; i++)
        printf("%d ", arr[i]);
    printf("\n");

    /* Parcours avec pointeur (style C idiomatique) */
    for (ptr = arr; ptr < arr + size; ptr++)
        printf("%d ", *ptr);
    printf("\n");

    /* Pointeur vers la fin (pattern courant) */
    int *end = arr + size;  /* pointe APRES le dernier element */
    for (ptr = arr; ptr != end; ptr++)
        printf("%d ", *ptr);
    printf("\n");

    return (0);
}
```

---

## 8. Pointeurs et chaines de caracteres

### 8.1 char *str vs char str[]

> [!warning] C'est une des sources de bugs les plus frequentes en C

```c
/* Forme 1 : tableau sur la stack (MODIFIABLE) */
char str1[] = "Hello";
/* Cree un tableau de 6 chars sur la stack : {'H','e','l','l','o','\0'} */
/* str1 est modifiable */
str1[0] = 'h';   /* OK */

/* Forme 2 : pointeur vers un string literal (READ-ONLY) */
char *str2 = "Hello";
/* "Hello" est stocke dans une zone memoire en lecture seule (.rodata) */
/* str2 pointe vers cette zone */
str2[0] = 'h';   /* UNDEFINED BEHAVIOR ! Segfault probable */
```

```
Forme 1 : char str1[] = "Hello";

Stack :
+-----+-----+-----+-----+-----+------+
| 'H' | 'e' | 'l' | 'l' | 'o' | '\0' |   str1 (6 octets sur la stack)
+-----+-----+-----+-----+-----+------+
  MODIFIABLE

Forme 2 : char *str2 = "Hello";

Stack :              .rodata (read-only) :
+----------+         +-----+-----+-----+-----+-----+------+
| 0x4000   |-------->| 'H' | 'e' | 'l' | 'l' | 'o' | '\0' |
+----------+         +-----+-----+-----+-----+-----+------+
   str2                NON MODIFIABLE
```

### 8.2 Parcourir une chaine avec un pointeur

```c
#include <stdio.h>

/* Compter la longueur d'une chaine */
int _strlen(char *s)
{
    int len = 0;

    while (*s != '\0')
    {
        len++;
        s++;
    }
    return (len);
}

/* Version encore plus concise */
int _strlen_v2(char *s)
{
    char *start = s;

    while (*s)
        s++;
    return (s - start);
}

int main(void)
{
    char str[] = "Hello, World!";

    printf("Longueur : %d\n", _strlen(str));     /* 13 */
    printf("Longueur : %d\n", _strlen_v2(str));  /* 13 */

    return (0);
}
```

Pour approfondir les chaines, voir [[03c - Chaines de Caracteres]].

---

## 9. Pointeur de pointeur (double pointeur)

### 9.1 Definition

Un pointeur de pointeur (double pointeur) est un pointeur qui stocke **l'adresse d'un autre pointeur**.

```c
int x = 42;
int *p = &x;      /* p pointe vers x */
int **pp = &p;    /* pp pointe vers p */
```

### 9.2 Visualisation memoire

```
Variable x          Pointeur p          Double pointeur pp
+--------+          +--------+          +--------+
|   42   |          | 0x1000 |-----+    | 0x2000 |-----+
+--------+          +--------+     |    +--------+     |
Adr: 0x1000         Adr: 0x2000   |    Adr: 0x3000    |
    ^                    ^         |         ^          |
    |                    |         |         |          |
    +--------------------+---------+         +----------+
    pointe vers x        pointe vers p

Dereferencement :
*pp   == p      == 0x1000  (l'adresse de x)
**pp  == *p     == 42      (la valeur de x)
&p    == pp               (l'adresse de p)
```

### 9.3 Code complet

```c
#include <stdio.h>

int main(void)
{
    int x = 42;
    int *p = &x;
    int **pp = &p;

    printf("x      = %d\n", x);           /* 42 */
    printf("*p     = %d\n", *p);           /* 42 */
    printf("**pp   = %d\n", **pp);         /* 42 */

    printf("\n");

    printf("&x     = %p\n", (void *)&x);   /* 0x1000 */
    printf("p      = %p\n", (void *)p);     /* 0x1000 (meme chose) */
    printf("*pp    = %p\n", (void *)*pp);   /* 0x1000 (meme chose) */

    printf("\n");

    printf("&p     = %p\n", (void *)&p);    /* 0x2000 */
    printf("pp     = %p\n", (void *)pp);    /* 0x2000 (meme chose) */

    /* Modifier x via le double pointeur */
    **pp = 100;
    printf("\nx apres **pp = 100 : %d\n", x);  /* 100 */

    return (0);
}
```

### 9.4 Pourquoi on a besoin de double pointeurs

> [!tip] Regle d'or
> Si tu veux qu'une fonction modifie une variable `int`, tu lui passes `int *`.
> Si tu veux qu'une fonction modifie un **pointeur** (`int *`), tu lui passes `int **`.

**Cas concret : modifier la tete d'une linked list**

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct node_s
{
    int data;
    struct node_s *next;
} node_t;

/* MAUVAIS : head est une copie locale → la modification est perdue */
void add_front_bad(node_t *head, int data)
{
    node_t *new = malloc(sizeof(node_t));
    new->data = data;
    new->next = head;
    head = new;   /* Modifie la COPIE locale, pas le head de main ! */
}

/* CORRECT : on passe l'adresse du pointeur head */
void add_front(node_t **head, int data)
{
    node_t *new = malloc(sizeof(node_t));
    new->data = data;
    new->next = *head;   /* *head = l'ancien premier noeud */
    *head = new;         /* Modifie le VRAI head de main */
}

int main(void)
{
    node_t *head = NULL;

    add_front(&head, 30);
    add_front(&head, 20);
    add_front(&head, 10);

    /* Liste : 10 → 20 → 30 → NULL */
    node_t *current = head;
    while (current)
    {
        printf("%d → ", current->data);
        current = current->next;
    }
    printf("NULL\n");

    return (0);
}
```

```
Pourquoi add_front_bad ne marche pas :

main :          add_front_bad :
head ----+      head_copy ----+     (copie independante)
         |                    |
         v                    v
     [node A]             [node A]

Apres head_copy = new :
head ----+      head_copy ----+
         |                    |
         v                    v
     [node A]             [new node] -> [node A]
                          ^^^^^^^^^^^^
                          Perdu au retour de la fonction !

Pourquoi add_front (avec **head) marche :

main :          add_front :
head ----+      head_ptr -------> &head (adresse du pointeur)
         |
         v
     [node A]

*head_ptr = new  modifie directement head dans main
```

---

## 10. Pointeurs et fonctions

### 10.1 Passage par valeur vs passage par adresse

En C, **tout est passe par valeur**. Pour modifier une variable de l'appelant, on passe son **adresse**.

```c
#include <stdio.h>

/* PASSAGE PAR VALEUR : ne modifie pas l'original */
void increment_bad(int n)
{
    n = n + 1;  /* Modifie la copie locale */
}

/* PASSAGE PAR ADRESSE : modifie l'original */
void increment(int *n)
{
    *n = *n + 1;  /* Modifie la valeur a l'adresse pointee */
}

int main(void)
{
    int x = 10;

    increment_bad(x);
    printf("Apres increment_bad : %d\n", x);   /* 10 (pas modifie !) */

    increment(&x);
    printf("Apres increment     : %d\n", x);   /* 11 (modifie !) */

    return (0);
}
```

### 10.2 La fonction swap classique

```c
#include <stdio.h>

void swap(int *a, int *b)
{
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

int main(void)
{
    int x = 5, y = 10;

    printf("Avant : x = %d, y = %d\n", x, y);  /* x=5, y=10 */
    swap(&x, &y);
    printf("Apres : x = %d, y = %d\n", x, y);  /* x=10, y=5 */

    return (0);
}
```

```
Avant swap :        Pendant swap :          Apres swap :

main:               swap:                   main:
x = 5               *a = 5                  x = 10
y = 10              *b = 10                 y = 5
                    tmp = 5
                    *a = *b  → x = 10
                    *b = tmp → y = 5
```

### 10.3 Retourner un pointeur depuis une fonction

> [!warning] Ne JAMAIS retourner un pointeur vers une variable locale

```c
/* DANGEREUX : la variable locale est detruite au retour */
int *bad_function(void)
{
    int x = 42;
    return (&x);   /* x est detruite → dangling pointer ! */
}

/* CORRECT : utiliser malloc (la memoire persiste) */
int *good_function(void)
{
    int *ptr = malloc(sizeof(int));
    if (ptr == NULL)
        return (NULL);
    *ptr = 42;
    return (ptr);   /* OK : la memoire est sur le heap */
}

/* CORRECT aussi : utiliser static (la variable persiste) */
int *static_function(void)
{
    static int x = 42;
    return (&x);   /* OK : static dure toute la vie du programme */
}
```

---

## 11. Pointeur NULL

### 11.1 Toujours initialiser

```c
int *ptr = NULL;   /* Bonne pratique : toujours initialiser */

/* Verifier avant de dereferencer */
if (ptr != NULL)
{
    printf("Valeur : %d\n", *ptr);
}
else
{
    printf("Le pointeur est NULL\n");
}
```

### 11.2 Pourquoi c'est important

Dereferencer un pointeur NULL provoque un **segmentation fault** (crash). Mais c'est **mieux** qu'un wild pointer car :

```
Wild pointer (non initialise) :
  - Pointe vers une adresse ALEATOIRE
  - Peut corrompre la memoire silencieusement
  - Bug TRES difficile a trouver

Pointeur NULL :
  - Pointe vers l'adresse 0
  - Crash IMMEDIATEMENT avec un segfault
  - Bug FACILE a trouver (gdb dit exactement ou)
```

> [!tip] Regle Holberton
> - Initialiser chaque pointeur a `NULL`
> - Verifier chaque `malloc()` (peut retourner `NULL`)
> - Verifier les pointeurs recus en parametre dans les fonctions
> - Apres `free()`, mettre le pointeur a `NULL`

---

## 12. Les erreurs classiques

### 12.1 Wild pointer (pointeur non initialise)

```c
int *ptr;         /* Contient une adresse aleatoire */
*ptr = 42;        /* Ecrit 42 a une adresse aleatoire → CRASH ou corruption */
```

**Solution** : toujours `int *ptr = NULL;`

### 12.2 Dangling pointer (pointeur pendant)

```c
int *ptr = malloc(sizeof(int));
*ptr = 42;
free(ptr);        /* La memoire est liberee */
printf("%d\n", *ptr);  /* DANGLING POINTER : la memoire ne nous appartient plus */
```

**Solution** : `ptr = NULL;` apres `free(ptr);`

> [!info] Approfondir
> Les problemes de memoire (malloc, free, dangling pointers, memory leaks) sont traites en detail dans [[04 - Pointeurs et Memoire]].

### 12.3 Confusion declaration vs dereferencement

```c
int x = 42;
int *ptr = &x;    /* * ici = declaration */
int y = *ptr;     /* * ici = dereferencement */

/* Ce n'est PAS la meme etoile ! */
/* Declaration :      int *ptr    → "ptr est un pointeur vers int" */
/* Dereferencement :  *ptr        → "la valeur a l'adresse dans ptr" */
```

### 12.4 Oublier le & dans scanf

```c
int x;
scanf("%d", x);    /* FAUX : passe la valeur de x (0 ou garbage) */
scanf("%d", &x);   /* CORRECT : passe l'adresse de x */
```

### 12.5 Comparer des pointeurs au lieu des valeurs

```c
char *s1 = "hello";
char *s2 = "hello";

if (s1 == s2)         /* Compare les ADRESSES, pas les chaines ! */
    printf("Egal\n"); /* Peut etre vrai ou faux selon le compilateur */

if (strcmp(s1, s2) == 0)  /* Compare les CONTENUS */
    printf("Egal\n");     /* Toujours correct */
```

---

## 13. Tableaux de pointeurs

### 13.1 char *argv[]

Le `main` en C recoit ses arguments sous forme de tableau de pointeurs :

```c
int main(int argc, char *argv[])
{
    /* argc = nombre d'arguments */
    /* argv = tableau de pointeurs vers des chaines */

    for (int i = 0; i < argc; i++)
        printf("argv[%d] = \"%s\"\n", i, argv[i]);

    return (0);
}
```

```bash
$ ./programme hello world 42
argv[0] = "./programme"
argv[1] = "hello"
argv[2] = "world"
argv[3] = "42"
```

```
Memoire :

argv:
+----------+         +---+---+---+---+---+---+---+---+---+---+---+----+
| argv[0]  |-------->| . | / | p | r | o | g | r | a | m | m | e | \0 |
+----------+         +---+---+---+---+---+---+---+---+---+---+---+----+
| argv[1]  |-------->| h | e | l | l | o | \0|
+----------+         +---+---+---+---+---+---+
| argv[2]  |-------->| w | o | r | l | d | \0|
+----------+         +---+---+---+---+---+---+
| argv[3]  |-------->| 4 | 2 | \0|
+----------+         +---+---+---+
| NULL     |   ← argv est toujours termine par NULL
+----------+
```

### 13.2 Tableau de chaines personnalise

```c
#include <stdio.h>

int main(void)
{
    char *jours[] = {
        "Lundi",
        "Mardi",
        "Mercredi",
        "Jeudi",
        "Vendredi",
        "Samedi",
        "Dimanche"
    };

    int nb_jours = sizeof(jours) / sizeof(jours[0]);

    for (int i = 0; i < nb_jours; i++)
        printf("Jour %d : %s\n", i + 1, jours[i]);

    return (0);
}
```

---

## 14. const et pointeurs

### 14.1 Trois combinaisons possibles

```c
/* 1. Pointeur vers une valeur constante */
const int *p1;        /* La VALEUR pointee ne peut pas etre modifiee */
int const *p1;        /* Equivalent (const avant ou apres int, meme chose) */

int x = 10, y = 20;
p1 = &x;
/* *p1 = 42;  ERREUR : ne peut pas modifier la valeur */
p1 = &y;      /* OK : on peut changer vers quoi le pointeur pointe */


/* 2. Pointeur constant vers une valeur variable */
int *const p2 = &x;  /* Le POINTEUR ne peut pas etre change */

*p2 = 42;     /* OK : on peut modifier la valeur */
/* p2 = &y;   ERREUR : ne peut pas changer l'adresse */


/* 3. Pointeur constant vers une valeur constante */
const int *const p3 = &x;  /* RIEN ne peut etre modifie */

/* *p3 = 42;  ERREUR */
/* p3 = &y;   ERREUR */
```

> [!tip] Comment lire les declarations const
> Lis de **droite a gauche** :
> ```
> const int *p   → p est un pointeur vers un int constant
>                   (la valeur est constante)
>
> int *const p   → p est un pointeur constant vers un int
>                   (le pointeur est constant)
>
> const int *const p → p est un pointeur constant vers un int constant
>                      (tout est constant)
> ```

### 14.2 Utilisation avec les fonctions

```c
/* const dans les parametres = promesse de ne pas modifier */
int _strlen(const char *s)
{
    int len = 0;
    while (s[len])
        len++;
    return (len);
    /* s[0] = 'X';  ERREUR : on a promis de ne pas modifier */
}

/* Pourquoi c'est important ? */
/* 1. Documentation : le lecteur sait que la chaine ne sera pas modifiee */
/* 2. Securite : le compilateur empeche les modifications accidentelles */
/* 3. Permet de passer des string literals sans undefined behavior */
```

---

## 15. Exercices pratiques

### Exercice 1 : Bases (declaration, &, *)

> [!example] A coder
> ```c
> /* Declarer un int x = 42, un pointeur p vers x.
>  * Afficher :
>  * - La valeur de x
>  * - L'adresse de x (avec &)
>  * - La valeur de p (l'adresse qu'il contient)
>  * - La valeur pointee par p (avec *)
>  * Modifier x via le pointeur (*p = 100), verifier que x a change.
>  */
> ```

### Exercice 2 : Swap

> [!example] A coder
> ```c
> /* Ecrire la fonction :
>  * void swap_int(int *a, int *b);
>  * Qui echange les valeurs de deux entiers.
>  *
>  * Tester avec :
>  * int x = 5, y = 10;
>  * swap_int(&x, &y);
>  * Verifier que x == 10 et y == 5.
>  */
> ```

### Exercice 3 : Parcours de tableau avec pointeurs

> [!example] A coder
> ```c
> /* Ecrire la fonction :
>  * void print_array(int *arr, int size);
>  * Qui affiche les elements d'un tableau en utilisant
>  * l'arithmetique des pointeurs (PAS arr[i]).
>  *
>  * Ecrire aussi :
>  * int *find(int *arr, int size, int value);
>  * Qui retourne un pointeur vers la premiere occurrence
>  * de value dans le tableau, ou NULL si pas trouvee.
>  */
> ```

### Exercice 4 : _strlen et _strcpy

> [!example] A coder
> ```c
> /* Reimplementer :
>  * int _strlen(char *s);
>  *   Retourne la longueur de la chaine.
>  *
>  * char *_strcpy(char *dest, char *src);
>  *   Copie src dans dest, retourne dest.
>  *
>  * Utiliser uniquement l'arithmetique des pointeurs.
>  * Ne PAS utiliser d'index [i].
>  */
> ```

### Exercice 5 : Double pointeur

> [!example] A coder
> ```c
> /* Ecrire la fonction :
>  * void allocate_and_set(int **ptr, int value);
>  * Qui :
>  * 1. Alloue un int avec malloc
>  * 2. Met value dans cet int
>  * 3. Met l'adresse dans *ptr
>  *
>  * Tester avec :
>  * int *p = NULL;
>  * allocate_and_set(&p, 42);
>  * printf("%d\n", *p);   // doit afficher 42
>  * free(p);
>  */
> ```

### Exercice 6 : const correctness

> [!example] A coder
> ```c
> /* Pour chacune des fonctions suivantes, determiner si le
>  * parametre doit etre :
>  *   - char *s
>  *   - const char *s
>  *   - char *const s
>  *
>  * 1. int my_strlen(??? s);         // ne modifie pas la chaine
>  * 2. void my_toupper(??? s);       // modifie chaque caractere
>  * 3. char *my_strdup(??? s);       // ne modifie pas, retourne une copie
>  * 4. void my_clear(??? s);         // met '\0' au debut
>  */
> ```

---

## 16. Resume : Cheat Sheet Pointeurs

```
+-----------------------------------------------------------+
|              CHEAT SHEET POINTEURS EN C                     |
+-----------------------------------------------------------+
|                                                            |
|  DECLARATION :                                             |
|    int *ptr = NULL;     // toujours initialiser !          |
|                                                            |
|  OPERATEURS :                                              |
|    &x     → adresse de x                                  |
|    *ptr   → valeur a l'adresse dans ptr                   |
|                                                            |
|  EQUIVALENCES TABLEAU/POINTEUR :                           |
|    arr[i]  ==  *(arr + i)                                  |
|    &arr[0] ==  arr                                         |
|                                                            |
|  PASSAGE AUX FONCTIONS :                                   |
|    Modifier un int   → passer int *                        |
|    Modifier un int * → passer int **                       |
|                                                            |
|  CONST :                                                   |
|    const int *p   → valeur constante                       |
|    int *const p   → pointeur constant                      |
|                                                            |
|  REGLES D'OR :                                             |
|    1. Initialiser a NULL                                   |
|    2. Verifier avant de dereferencer                       |
|    3. free() puis ptr = NULL                               |
|    4. Ne jamais retourner &variable_locale                 |
|                                                            |
+-----------------------------------------------------------+
```

---

## Liens

- [[04 - Pointeurs et Memoire]] - malloc, free, gestion dynamique
- [[05 - Pointeurs de Fonctions]] - Pointeurs vers des fonctions
- [[03 - Structures et Typedef]] - Structures et pointeurs
- [[03c - Chaines de Caracteres]] - Les chaines en C (string.h)
- [[01 - Introduction au C et Compilation]] - Bases du C
