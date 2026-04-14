# Le Preprocesseur C

> [!info] Contexte
> Le preprocesseur est la **premiere etape** de la compilation en C. Il effectue un traitement **purement textuel** du code source avant que le compilateur ne prenne le relais. Comprendre le preprocesseur est essentiel pour ecrire du C professionnel.

---

## Table des matieres

1. [[#Qu'est-ce que le preprocesseur ?]]
2. [[#La directive include]]
3. [[#Header Guards]]
4. [[#define - Constantes]]
5. [[#define - Macros avec parametres]]
6. [[#Operateurs de macros]]
7. [[#Macros predefinies]]
8. [[#Compilation conditionnelle]]
9. [[#undef - Supprimer une definition]]
10. [[#error et warning]]
11. [[#Macros vs Fonctions]]
12. [[#Bonnes pratiques Holberton]]
13. [[#Exercices]]

---

## Qu'est-ce que le preprocesseur ?

Le preprocesseur C est un **programme** qui s'execute **avant** le compilateur. Il lit le fichier source `.c` et produit un fichier **temporaire** modifie que le compilateur va ensuite traiter.

### Pipeline de compilation

```
+------------------+     +------------------+     +------------------+
|   fichier.c      | --> |  PREPROCESSEUR   | --> |  fichier.i       |
|  (code source)   |     |  (traitement     |     |  (code           |
|                  |     |   textuel)       |     |   preprocesse)   |
+------------------+     +------------------+     +------------------+
                                                          |
                                                          v
                         +------------------+     +------------------+
                         |  fichier.o       | <-- |  COMPILATEUR     |
                         |  (code objet)    |     |  (analyse +      |
                         +------------------+     |   generation)    |
                                                  +------------------+
```

### Voir le resultat du preprocesseur

```bash
# -E : s'arreter apres le preprocessing
gcc -E fichier.c -o fichier.i

# Voir directement dans le terminal
gcc -E fichier.c | less

# Avec les warnings habituels
gcc -E -Wall fichier.c -o fichier.i
```

> [!tip] Astuce
> Utiliser `gcc -E` est un excellent moyen de **debugger** vos macros et directives. Vous voyez exactement ce que le compilateur recoit apres le travail du preprocesseur.

### Ce que fait le preprocesseur

| Action                          | Directive         |
| ------------------------------- | ----------------- |
| Inclure des fichiers            | `#include`        |
| Definir des constantes/macros   | `#define`         |
| Compilation conditionnelle      | `#ifdef`, `#if`   |
| Supprimer des definitions       | `#undef`          |
| Generer des erreurs             | `#error`          |
| Generer des avertissements      | `#warning`        |

> [!warning] Important
> Le preprocesseur ne comprend **PAS** le C. Il fait du remplacement de texte pur et simple. C'est une distinction fondamentale a retenir.

---

## La directive include

`#include` est la directive la plus utilisee. Elle effectue un **copier-coller textuel** du contenu d'un fichier dans le fichier courant.

### Deux formes

```c
#include <stdio.h>    /* Cherche dans les repertoires systeme */
#include "mon_header.h" /* Cherche d'abord dans le repertoire local */
```

### Difference entre < > et " "

```
+--------------------------------------+
|          #include <fichier>          |
|                                      |
|  Recherche dans :                    |
|    /usr/include/                     |
|    /usr/local/include/               |
|    Repertoires specifies par -I      |
|                                      |
|  Usage : bibliotheques standard      |
|          (stdio.h, stdlib.h, etc.)   |
+--------------------------------------+

+--------------------------------------+
|         #include "fichier"           |
|                                      |
|  Recherche dans :                    |
|    1. Repertoire du fichier courant  |
|    2. Puis les repertoires systeme   |
|                                      |
|  Usage : vos propres headers         |
|          (main.h, utils.h, etc.)     |
+--------------------------------------+
```

### Ce qui se passe concretement

Quand le preprocesseur rencontre `#include "utils.h"` :

```
Avant preprocessing :            Apres preprocessing :
+-------------------+            +-------------------+
| #include "utils.h"|            | // contenu de     |
| int main(void)    |    --->    | // utils.h copie  |
| {                 |            | // ici            |
|     return (0);   |            | void print(void); |
| }                 |            | int add(int, int);|
+-------------------+            | int main(void)    |
                                 | {                 |
                                 |     return (0);   |
                                 | }                 |
                                 +-------------------+
```

> [!example] Exemple concret
> ```c
> /* utils.h */
> #ifndef UTILS_H
> #define UTILS_H
> 
> void print_hello(void);
> int add(int a, int b);
> 
> #endif
> ```
> 
> ```c
> /* main.c */
> #include <stdio.h>
> #include "utils.h"
> 
> int main(void)
> {
>     print_hello();
>     printf("%d\n", add(2, 3));
>     return (0);
> }
> ```

### Ajouter des repertoires de recherche

```bash
# -I ajoute un repertoire de recherche pour les headers
gcc -I./include -I../libs/include main.c -o prog
```

---

## Header Guards

### Le probleme : l'inclusion multiple

Sans protection, un header peut etre inclus **plusieurs fois**, causant des erreurs de redefinition :

```
main.c inclut a.h et b.h
a.h inclut types.h
b.h inclut types.h

Resultat : types.h est inclus DEUX FOIS !

+-------------------+
| main.c            |
|   #include "a.h"  |  --> a.h inclut types.h (1ere fois)
|   #include "b.h"  |  --> b.h inclut types.h (2eme fois !)
+-------------------+

ERREUR : redefinition de struct, typedef, etc.
```

### La solution : le pattern #ifndef / #define / #endif

```c
/* mon_header.h */
#ifndef MON_HEADER_H    /* Si MON_HEADER_H n'est PAS defini... */
#define MON_HEADER_H    /* ...le definir */

/* Contenu du header */
void ma_fonction(void);
typedef struct s_data
{
    int value;
} t_data;

#endif /* MON_HEADER_H */  /* Fin de la protection */
```

### Comment ca marche

```
Premiere inclusion :                Deuxieme inclusion :
+-----------------------------+     +-----------------------------+
| #ifndef MON_HEADER_H        |     | #ifndef MON_HEADER_H        |
| --> PAS defini, on continue |     | --> DEJA defini, on SAUTE   |
|                             |     |    tout jusqu'au #endif     |
| #define MON_HEADER_H        |     |                             |
| void ma_fonction(void);     |     | #endif                      |
| typedef struct s_data {...} |     +-----------------------------+
|                             |
| #endif                      |     Resultat : rien n'est inclus
+-----------------------------+     une deuxieme fois !
```

### Convention de nommage

```c
/* Fichier : mon_fichier.h  -->  MON_FICHIER_H */
/* Fichier : utils.h        -->  UTILS_H       */
/* Fichier : types.h        -->  TYPES_H       */
/* Fichier : my_lib.h       -->  MY_LIB_H      */

/* Pattern : nom du fichier en MAJUSCULES, . remplace par _ */
```

### #pragma once (alternative non standard)

```c
/* mon_header.h */
#pragma once

/* Contenu du header */
void ma_fonction(void);
```

> [!warning] Attention
> `#pragma once` n'est **PAS** dans le standard C. Meme si la plupart des compilateurs le supportent (GCC, Clang, MSVC), il faut **toujours utiliser les header guards** classiques dans un contexte Holberton ou pour du code portable.

---

## #define - Constantes

`#define` cree une **substitution textuelle**. Le preprocesseur remplace chaque occurrence de l'identifiant par sa valeur.

### Syntaxe

```c
#define NOM valeur_de_remplacement
```

### Exemples

```c
#define PI 3.14159265358979
#define MAX_SIZE 100
#define BUFFER_SIZE 1024
#define TRUE 1
#define FALSE 0
#define SUCCESS 0
#define FAILURE -1
#define NEWLINE '\n'
#define AUTHOR "Holberton"
```

### Ce qui se passe

```c
/* Avant preprocessing */
#define MAX_SIZE 100

int tab[MAX_SIZE];
for (int i = 0; i < MAX_SIZE; i++)
    tab[i] = 0;

/* Apres preprocessing (gcc -E) */
int tab[100];
for (int i = 0; i < 100; i++)
    tab[i] = 0;
```

### #define vs const

| Critere              | `#define`                        | `const`                          |
| -------------------- | -------------------------------- | -------------------------------- |
| Traitement           | Preprocesseur (remplacement)     | Compilateur (variable)           |
| Type                 | Aucun type                       | Type verifie                     |
| Memoire              | Pas d'adresse memoire            | Stocke en memoire                |
| Portee               | Du `#define` a la fin du fichier | Portee du bloc                   |
| Debug                | N'apparait pas dans le debugger  | Visible dans le debugger         |
| Utilisation en C     | Courant et idiomatique           | Moins courant pour les constantes|

> [!tip] Convention
> Les constantes `#define` sont **TOUJOURS en MAJUSCULES** avec des underscores pour separer les mots :
> ```c
> #define MAX_BUFFER_SIZE 4096
> #define DEFAULT_TIMEOUT 30
> #define PI_OVER_TWO 1.5707963
> ```

### Definir sans valeur

```c
#define DEBUG     /* Pas de valeur, mais DEFINI */
#define VERBOSE   /* Utile pour la compilation conditionnelle */
```

---

## #define - Macros avec parametres

Les macros peuvent prendre des **parametres**, comme des fonctions. Mais attention aux pieges !

### Syntaxe de base

```c
#define NOM(param1, param2) expression_utilisant_param1_et_param2
```

### Exemples simples

```c
#define MAX(a, b) ((a) > (b) ? (a) : (b))
#define MIN(a, b) ((a) < (b) ? (a) : (b))
#define ABS(x)    ((x) < 0 ? -(x) : (x))
#define SQUARE(x) ((x) * (x))
#define SWAP(a, b, type) { type _tmp = (a); (a) = (b); (b) = _tmp; }
```

### Pourquoi les parentheses sont CRITIQUES

> [!warning] Bug classique : macros sans parentheses
> ```c
> /* MAUVAIS - sans parentheses */
> #define DOUBLE(x) x * 2
> 
> int result = DOUBLE(3 + 4);
> /* Le preprocesseur produit : 3 + 4 * 2 = 3 + 8 = 11 */
> /* On voulait : (3 + 4) * 2 = 14 ! */
> 
> /* CORRECT - avec parentheses */
> #define DOUBLE(x) ((x) * 2)
> 
> int result = DOUBLE(3 + 4);
> /* Le preprocesseur produit : ((3 + 4) * 2) = 14 */
> ```

Illustration du probleme :

```
Sans parentheses :                 Avec parentheses :
#define MUL(a,b) a * b            #define MUL(a,b) ((a) * (b))

MUL(2+3, 4+5)                    MUL(2+3, 4+5)
--> 2+3 * 4+5                    --> ((2+3) * (4+5))
--> 2 + 12 + 5                   --> (5 * 9)
--> 19  (FAUX !)                 --> 45  (CORRECT)
```

### Le piege de la double evaluation

> [!warning] Piege mortel : double evaluation
> ```c
> #define SQUARE(x) ((x) * (x))
> 
> int i = 5;
> int result = SQUARE(i++);
> /* Le preprocesseur produit : ((i++) * (i++)) */
> /* i est incremente DEUX FOIS ! */
> /* Resultat indefini (undefined behavior) ! */
> ```
> 
> Avec une fonction, `i++` ne serait evalue qu'**une seule fois** avant l'appel.

### Macros multi-lignes

Utilisez `\` pour continuer une macro sur plusieurs lignes :

```c
#define PRINT_ARRAY(arr, size)    \
    do {                          \
        for (int i = 0; i < (size); i++) \
            printf("%d ", (arr)[i]); \
        printf("\n");             \
    } while (0)

#define SAFE_FREE(ptr)  \
    do {                \
        free(ptr);      \
        (ptr) = NULL;   \
    } while (0)
```

> [!info] Pourquoi do { ... } while (0) ?
> Le pattern `do { ... } while (0)` permet d'utiliser la macro comme une instruction normale avec un `;` a la fin, et fonctionne correctement dans un `if/else` sans accolades :
> ```c
> if (condition)
>     SAFE_FREE(ptr);  /* Fonctionne correctement */
> else
>     /* ... */
> ```

### Macro avec nombre variable d'arguments

```c
#define LOG(fmt, ...) fprintf(stderr, fmt, ##__VA_ARGS__)
/* ##__VA_ARGS__ : supprime la virgule si pas d'arguments supplementaires */

LOG("Erreur !\n");
LOG("Valeur = %d\n", 42);
```

---

## Operateurs de macros

### L'operateur # (stringification)

L'operateur `#` transforme un parametre de macro en **chaine de caracteres** :

```c
#define STRINGIFY(x) #x
#define PRINT_VAR(x) printf(#x " = %d\n", x)
#define PRINT_EXPR(expr) printf(#expr " = %d\n", (expr))

int age = 25;
PRINT_VAR(age);
/* Produit : printf("age" " = %d\n", age); */
/* Affiche : age = 25 */

PRINT_EXPR(3 + 4 * 2);
/* Produit : printf("3 + 4 * 2" " = %d\n", (3 + 4 * 2)); */
/* Affiche : 3 + 4 * 2 = 11 */
```

> [!info] Rappel
> En C, deux chaines literals adjacentes sont automatiquement concatenees :
> `"Hello " "World"` devient `"Hello World"`

### L'operateur ## (concatenation de tokens)

L'operateur `##` colle deux tokens ensemble pour former un nouveau token :

```c
#define MAKE_FUNC(name) void func_##name(void)
#define MAKE_VAR(type, name) type var_##name
#define CONCAT(a, b) a##b

MAKE_FUNC(init);
/* Produit : void func_init(void); */

MAKE_FUNC(cleanup);
/* Produit : void func_cleanup(void); */

int CONCAT(my, _variable) = 42;
/* Produit : int my_variable = 42; */
```

### Exemple avance : generateur de code

```c
#define DECLARE_LIST(type)                          \
    typedef struct s_list_##type                     \
    {                                               \
        type data;                                  \
        struct s_list_##type *next;                  \
    } t_list_##type;                                \
    t_list_##type *new_node_##type(type val);        \
    void push_##type(t_list_##type **head, type val)

/* Utilisation */
DECLARE_LIST(int);
/* Genere une structure de liste chainee pour int */

DECLARE_LIST(float);
/* Genere une structure de liste chainee pour float */
```

---

## Macros predefinies

Le standard C definit plusieurs macros toujours disponibles :

### Macros standard

| Macro              | Description                        | Exemple de valeur          |
| ------------------ | ---------------------------------- | -------------------------- |
| `__FILE__`         | Nom du fichier source courant      | `"main.c"`                 |
| `__LINE__`         | Numero de ligne courant            | `42`                       |
| `__func__`         | Nom de la fonction courante (C99)  | `"main"`                   |
| `__DATE__`         | Date de compilation                | `"Apr 14 2026"`            |
| `__TIME__`         | Heure de compilation               | `"10:30:45"`               |
| `__STDC__`         | 1 si compilateur conforme au std   | `1`                        |
| `__STDC_VERSION__` | Version du standard C              | `199901L` (C99)            |

### Macro de debug personnalisee

```c
#define DEBUG_LOG(msg) \
    fprintf(stderr, "[%s:%d in %s()] %s\n", \
            __FILE__, __LINE__, __func__, msg)

#define DEBUG_VAL(var, fmt) \
    fprintf(stderr, "[%s:%d] " #var " = " fmt "\n", \
            __FILE__, __LINE__, var)

/* Utilisation */
void process_data(int *data)
{
    if (data == NULL)
    {
        DEBUG_LOG("data est NULL !");
        /* Affiche : [main.c:15 in process_data()] data est NULL ! */
        return;
    }
    DEBUG_VAL(*data, "%d");
    /* Affiche : [main.c:20] *data = 42 */
}
```

### Macro d'assertion personnalisee

```c
#define MY_ASSERT(expr)                                    \
    do {                                                   \
        if (!(expr))                                       \
        {                                                  \
            fprintf(stderr,                                \
                "Assertion failed: %s\n"                   \
                "  File: %s, Line: %d\n"                   \
                "  Function: %s\n",                        \
                #expr, __FILE__, __LINE__, __func__);      \
            abort();                                       \
        }                                                  \
    } while (0)

/* Utilisation */
MY_ASSERT(ptr != NULL);
MY_ASSERT(size > 0 && size < MAX_SIZE);
```

> [!example] Utilisation pratique de __LINE__ et __FILE__
> ```c
> #define MALLOC_CHECK(ptr, size)                        \
>     do {                                               \
>         (ptr) = malloc(size);                          \
>         if ((ptr) == NULL)                             \
>         {                                              \
>             fprintf(stderr,                            \
>                 "malloc failed at %s:%d (%zu bytes)\n",\
>                 __FILE__, __LINE__, (size_t)(size));   \
>             exit(EXIT_FAILURE);                        \
>         }                                              \
>     } while (0)
> 
> /* Utilisation */
> char *buffer;
> MALLOC_CHECK(buffer, 1024);
> ```

---

## Compilation conditionnelle

La compilation conditionnelle permet d'**inclure ou exclure** des portions de code selon des conditions evaluees au moment du preprocessing.

### #ifdef / #ifndef / #else / #endif

```c
/* #ifdef : si la macro EST definie */
#ifdef DEBUG
    printf("Mode debug active\n");
    printf("Variable x = %d\n", x);
#endif

/* #ifndef : si la macro N'EST PAS definie */
#ifndef RELEASE
    run_tests();
#endif

/* Avec #else */
#ifdef VERBOSE
    printf("Traitement en cours...\n");
#else
    /* Rien en mode silencieux */
#endif
```

### #if / #elif / #else / #endif

```c
#if LEVEL == 1
    printf("Niveau debutant\n");
#elif LEVEL == 2
    printf("Niveau intermediaire\n");
#elif LEVEL == 3
    printf("Niveau avance\n");
#else
    printf("Niveau par defaut\n");
#endif
```

### Definir depuis la ligne de commande

```bash
# -D definit une macro
gcc -DDEBUG main.c -o prog
gcc -DLEVEL=2 main.c -o prog
gcc -DMAX_SIZE=256 -DVERBOSE main.c -o prog
```

### Cas d'usage : portabilite multi-OS

```c
#ifdef _WIN32
    /* Code specifique Windows */
    #include <windows.h>
    #define CLEAR_SCREEN "cls"
    #define PATH_SEP '\\'
#elif defined(__linux__)
    /* Code specifique Linux */
    #include <unistd.h>
    #define CLEAR_SCREEN "clear"
    #define PATH_SEP '/'
#elif defined(__APPLE__)
    /* Code specifique macOS */
    #include <unistd.h>
    #define CLEAR_SCREEN "clear"
    #define PATH_SEP '/'
#else
    #error "Systeme d'exploitation non supporte"
#endif
```

### Cas d'usage : code de debug conditionnel

```c
#ifdef DEBUG
    #define DBG(fmt, ...) fprintf(stderr, "DBG: " fmt "\n", ##__VA_ARGS__)
#else
    #define DBG(fmt, ...) /* rien */
#endif

/* Utilisation identique partout */
DBG("Valeur de x = %d", x);
DBG("Entree dans la fonction process()");

/* Compiler avec debug :  gcc -DDEBUG main.c */
/* Compiler sans debug :  gcc main.c         */
```

> [!tip] Astuce
> Le code de debug avec `#ifdef DEBUG` est **totalement supprime** du binaire final quand on compile sans `-DDEBUG`. Aucun impact sur les performances !

### Cas d'usage : features optionnelles

```c
#ifdef USE_COLOR
    #define RED     "\033[31m"
    #define GREEN   "\033[32m"
    #define RESET   "\033[0m"
#else
    #define RED     ""
    #define GREEN   ""
    #define RESET   ""
#endif

printf(RED "Erreur !" RESET "\n");
printf(GREEN "Succes !" RESET "\n");
```

### Operateur defined()

```c
/* Equivalent a #ifdef mais utilisable dans #if */
#if defined(DEBUG) && defined(VERBOSE)
    /* Code si les DEUX sont definis */
#endif

#if defined(LINUX) || defined(MACOS)
    /* Code pour systemes POSIX */
#endif

#if !defined(HEADER_H)
    /* Equivalent a #ifndef HEADER_H */
#endif
```

---

## #undef - Supprimer une definition

`#undef` supprime une definition de macro :

```c
#define BUFFER_SIZE 256

/* ... utilisation avec BUFFER_SIZE = 256 ... */

#undef BUFFER_SIZE
#define BUFFER_SIZE 1024

/* ... utilisation avec BUFFER_SIZE = 1024 ... */
```

### Cas d'usage

```c
/* Redefinir une macro de la bibliotheque standard */
#include <assert.h>
#undef assert
#define assert(expr) MY_ASSERT(expr)

/* Desactiver une macro temporairement */
#undef DEBUG
/* Le code ici ne sera pas en mode debug */
```

---

## #error et #warning

### #error : arreter la compilation

```c
#if !defined(__STDC_VERSION__) || __STDC_VERSION__ < 199901L
    #error "Ce programme necessite au minimum C99"
#endif

#ifndef MAX_SIZE
    #error "MAX_SIZE doit etre defini avec -DMAX_SIZE=valeur"
#endif

#if MAX_SIZE > 10000
    #error "MAX_SIZE trop grand (maximum 10000)"
#endif
```

### #warning : avertir sans arreter

```c
#ifdef DEPRECATED_API
    #warning "L'API deprecated sera supprimee dans la version 2.0"
#endif

#ifndef OPTIMIZATION_LEVEL
    #warning "OPTIMIZATION_LEVEL non defini, utilisation de la valeur par defaut"
    #define OPTIMIZATION_LEVEL 1
#endif
```

> [!info] Note
> `#warning` n'est **pas standard** en C89/C90, mais est supporte par GCC et Clang. Il est standardise a partir de C23.

---

## Macros vs Fonctions

### Tableau comparatif

| Critere                | Macros                              | Fonctions                          |
| ---------------------- | ----------------------------------- | ---------------------------------- |
| **Traitement**         | Preprocesseur (remplacement texte)  | Compilateur (appel de fonction)    |
| **Type-checking**      | Aucun                               | Oui, verification des types        |
| **Vitesse**            | Pas d'overhead d'appel              | Leger overhead (push/pop stack)    |
| **Taille du code**     | Code duplique a chaque utilisation  | Code unique, appele plusieurs fois |
| **Debug**              | Difficile (pas de breakpoint)       | Facile (breakpoint, step into)     |
| **Effets de bord**     | Double evaluation possible          | Arguments evalues une seule fois   |
| **Recursion**          | Impossible                          | Possible                           |
| **Pointeur possible**  | Non                                 | Oui (pointeur de fonction)         |
| **Type-generic**       | Oui (pas de type specifique)        | Non (sauf _Generic en C11)         |

### Quand utiliser quoi ?

```
+--------------------------------------------------+
|  UTILISER UNE MACRO quand :                      |
|    - Constante simple (#define PI 3.14)          |
|    - Compilation conditionnelle (#ifdef)         |
|    - Code type-generic (MIN, MAX)                |
|    - Performance critique (eviter l'appel)       |
|    - Generation de code (##, #)                  |
+--------------------------------------------------+

+--------------------------------------------------+
|  UTILISER UNE FONCTION quand :                   |
|    - Logique complexe (> 1-2 lignes)             |
|    - Besoin de debugger                          |
|    - Arguments avec effets de bord (i++)         |
|    - Besoin de recursion                         |
|    - Besoin d'un pointeur vers le code           |
+--------------------------------------------------+
```

> [!tip] Regle generale
> En cas de doute, preferez une **fonction**. Les macros sont puissantes mais dangereuses. Reservez-les aux cas ou elles apportent un reel avantage.

---

## Bonnes pratiques Holberton

### Regles obligatoires

```c
/* 1. TOUJOURS des header guards */
#ifndef MON_HEADER_H
#define MON_HEADER_H
/* ... */
#endif /* MON_HEADER_H */

/* 2. Noms de macros en MAJUSCULES */
#define MAX_SIZE 100       /* Bien */
#define max_size 100       /* MAL */

/* 3. Parentheses PARTOUT dans les macros */
#define DOUBLE(x) ((x) * 2)   /* Bien */
#define DOUBLE(x) x * 2       /* MAL - DANGEREUX */

/* 4. Pas de macros trop complexes */
/* MAL : trop complexe pour une macro */
#define SORT_ARRAY(arr, n, type) \
    for (int i = 0; i < (n)-1; i++) \
        for (int j = 0; j < (n)-i-1; j++) \
            if ((arr)[j] > (arr)[j+1]) { \
                type tmp = (arr)[j]; \
                (arr)[j] = (arr)[j+1]; \
                (arr)[j+1] = tmp; \
            }
/* Cela devrait etre une FONCTION */
```

### Structure type d'un header Holberton

```c
#ifndef MAIN_H
#define MAIN_H

/* Includes systeme */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* Constantes */
#define BUFFER_SIZE 1024
#define MAX_ARGS 10

/* Structures */
typedef struct s_data
{
    char *name;
    int value;
} t_data;

/* Prototypes */
int _strlen(char *s);
char *_strcpy(char *dest, char *src);
void print_data(t_data *data);

#endif /* MAIN_H */
```

### Resume des directives

```
+------------------------------------------------------------+
|  DIRECTIVE              |  UTILISATION                      |
|-------------------------|-----------------------------------|
|  #include               |  Inclure un fichier               |
|  #define                |  Definir constante/macro          |
|  #undef                 |  Supprimer une definition         |
|  #ifdef / #ifndef       |  Si (non) defini                  |
|  #if / #elif / #else    |  Condition sur valeur             |
|  #endif                 |  Fin de bloc conditionnel         |
|  #error                 |  Erreur de compilation            |
|  #warning               |  Avertissement de compilation     |
|  #pragma                |  Instructions specifiques compilo |
+------------------------------------------------------------+
```

---

## Exercices

### Exercice 1 : Header guard

> [!example] Exercice
> Creez un fichier `holberton.h` avec les header guards corrects contenant :
> - Un `#define` pour `HOLBERTON` valant `"Holberton"`
> - Un prototype pour `void print_holberton(void);`
>
> **Solution :**
> ```c
> #ifndef HOLBERTON_H
> #define HOLBERTON_H
> 
> #define HOLBERTON "Holberton"
> 
> void print_holberton(void);
> 
> #endif /* HOLBERTON_H */
> ```

### Exercice 2 : Macro SIZE

> [!example] Exercice
> Ecrivez une macro `SIZE` qui definit la taille d'un tableau a `1024`.
>
> **Solution :**
> ```c
> #define SIZE 1024
> ```

### Exercice 3 : Macro MAX

> [!example] Exercice
> Ecrivez une macro `MAX(a, b)` qui retourne la plus grande de deux valeurs. Assurez-vous qu'elle fonctionne correctement avec des expressions comme `MAX(x + 1, y * 2)`.
>
> **Solution :**
> ```c
> #define MAX(a, b) ((a) > (b) ? (a) : (b))
> ```

### Exercice 4 : Debug avec macros predefinies

> [!example] Exercice
> Ecrivez une macro `DEBUG_PRINT(msg)` qui affiche le message avec le nom du fichier et le numero de ligne. La macro ne doit rien faire si `DEBUG` n'est pas defini.
>
> **Solution :**
> ```c
> #ifdef DEBUG
>     #define DEBUG_PRINT(msg) \
>         fprintf(stderr, "[DEBUG %s:%d] %s\n", __FILE__, __LINE__, msg)
> #else
>     #define DEBUG_PRINT(msg)
> #endif
> ```

### Exercice 5 : Compilation conditionnelle multi-OS

> [!example] Exercice
> Ecrivez un bloc de compilation conditionnelle qui :
> - Sous Windows, definit `OS_NAME` a `"Windows"`
> - Sous Linux, definit `OS_NAME` a `"Linux"`
> - Sous macOS, definit `OS_NAME` a `"macOS"`
> - Sinon, genere une erreur de compilation
>
> **Solution :**
> ```c
> #ifdef _WIN32
>     #define OS_NAME "Windows"
> #elif defined(__linux__)
>     #define OS_NAME "Linux"
> #elif defined(__APPLE__)
>     #define OS_NAME "macOS"
> #else
>     #error "Systeme d'exploitation non reconnu"
> #endif
> ```

### Exercice 6 : Stringification et concatenation

> [!example] Exercice
> 1. Ecrivez une macro `TO_STRING(x)` qui convertit son argument en chaine.
> 2. Ecrivez une macro `VAR_NAME(prefix, id)` qui genere un nom de variable comme `data_1`, `data_2`, etc.
>
> **Solution :**
> ```c
> #define TO_STRING(x) #x
> #define VAR_NAME(prefix, id) prefix##_##id
> 
> /* Test */
> printf("%s\n", TO_STRING(Hello World));
> /* Affiche : Hello World */
> 
> int VAR_NAME(data, 1) = 10;  /* int data_1 = 10; */
> int VAR_NAME(data, 2) = 20;  /* int data_2 = 20; */
> ```

### Exercice 7 : Macro SAFE_DIVIDE

> [!example] Exercice
> Ecrivez une macro `SAFE_DIVIDE(a, b)` qui effectue la division `a / b` mais retourne `0` si `b` vaut `0`. Expliquez pourquoi cette macro pourrait poser probleme avec certains arguments.
>
> **Solution :**
> ```c
> #define SAFE_DIVIDE(a, b) ((b) != 0 ? (a) / (b) : 0)
> 
> /* Probleme : double evaluation */
> /* SAFE_DIVIDE(x++, y++) evaluerait x++ et y++ deux fois ! */
> /* Mieux vaut utiliser une fonction pour ce genre de logique */
> ```

---

## Liens

- [[01 - Introduction au C et Compilation]] : pipeline de compilation complet
- [[04 - Pointeurs et Memoire]] : allocation dynamique et macros utiles
- [[11 - Makefiles]] : automatisation de la compilation avec `-D` et flags

---

> [!tip] Resume
> Le preprocesseur est un outil **puissant mais dangereux**. Il fait du remplacement textuel sans comprendre le C. Utilisez-le pour les `#include`, les header guards, les constantes simples et la compilation conditionnelle. Pour la logique complexe, preferez les fonctions. Et surtout : **parentheses partout dans les macros** !
