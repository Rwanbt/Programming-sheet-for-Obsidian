# Les Chaines de Caracteres en C

---

## Introduction

En C, il n'existe pas de type `string` natif. Une chaine de caracteres est simplement un **tableau de `char` termine par le caractere nul `\0`**. Cette note couvre tout ce qu'il faut savoir sur la manipulation des chaines en C.

> [!warning] Prerequis
> Tu dois comprendre les pointeurs avant de lire cette note. Voir [[03b - Les Pointeurs]].

---

## 1. Qu'est-ce qu'une chaine en C ?

### 1.1 Definition

Une chaine (string) en C est un **tableau de caracteres** dont le dernier element est le caractere nul `'\0'` (valeur ASCII 0).

```c
char str[] = "Hello";
```

En memoire :

```
Index :    0     1     2     3     4     5
         +-----+-----+-----+-----+-----+------+
str :    | 'H' | 'e' | 'l' | 'l' | 'o' | '\0' |
         +-----+-----+-----+-----+-----+------+
ASCII :    72    101   108   108   111    0

sizeof(str) = 6  (5 caracteres + 1 null terminator)
strlen(str) = 5  (5 caracteres, sans le \0)
```

> [!tip] Regle fondamentale
> **Chaque chaine en C doit se terminer par `\0`.** Toutes les fonctions de la bibliotheque standard (`strlen`, `printf`, `strcpy`...) comptent sur ce `\0` pour savoir ou la chaine se termine.

---

## 2. Declaration de chaines

### 2.1 Tableau de char (stack, modifiable)

```c
char str[] = "Hello";
/* Equivalent a : */
char str[] = {'H', 'e', 'l', 'l', 'o', '\0'};
```

- Alloue 6 octets **sur la stack**
- Le contenu est **modifiable**
- `sizeof(str)` = 6

### 2.2 Pointeur vers string literal (read-only)

```c
char *str = "Hello";
```

- `"Hello"` est stocke dans la section `.rodata` (read-only data)
- `str` est un pointeur qui pointe vers cette zone
- Le contenu est **NON modifiable** (undefined behavior si on essaie)
- `sizeof(str)` = 8 (taille du pointeur, pas de la chaine)

### 2.3 La difference critique

> [!warning] C'est un piege classique en C

```c
/* Tableau : on peut modifier */
char s1[] = "Hello";
s1[0] = 'h';           /* OK : s1 est sur la stack */
printf("%s\n", s1);    /* "hello" */

/* Pointeur : on ne peut PAS modifier */
char *s2 = "Hello";
s2[0] = 'h';           /* UNDEFINED BEHAVIOR → segfault */
```

```
DIAGRAMME MEMOIRE :

char s1[] = "Hello";                char *s2 = "Hello";

STACK :                             STACK :
+-----+-----+-----+-----+-----+--+ +----------+
| 'H' | 'e' | 'l' | 'l' | 'o' |\0| | 0x400A00 |--+  s2 (pointeur)
+-----+-----+-----+-----+-----+--+ +----------+  |
  s1 (tableau, MODIFIABLE)                         |
                                                   |
                                    .RODATA :      |
                                    +-----+-----+--+--+-----+-----+--+
                                    | 'H' | 'e' | 'l' | 'l' | 'o' |\0|
                                    +-----+-----+-----+-----+-----+--+
                                    0x400A00  (READ-ONLY, NON MODIFIABLE)
```

### 2.4 Tableau avec taille explicite

```c
char str[20] = "Hello";
/* Alloue 20 octets, les 6 premiers remplis, les 14 restants a '\0' */
```

```
+---+---+---+---+---+----+----+----+----+----+----+ ... +----+
| H | e | l | l | o | \0 | \0 | \0 | \0 | \0 | \0 |     | \0 |
+---+---+---+---+---+----+----+----+----+----+----+ ... +----+
 0   1   2   3   4    5    6    7    8    9   10          19
```

### 2.5 Chaine vide

```c
char empty[] = "";        /* Un seul caractere : '\0' */
/* sizeof(empty) = 1 */
/* strlen(empty) = 0 */
```

---

## 3. Le null terminator \0

### 3.1 Pourquoi il est essentiel

Sans `\0`, les fonctions de chaine ne savent pas ou s'arreter. Elles continuent de lire la memoire **au-dela** du tableau.

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    /* CORRECT : avec \0 */
    char good[] = {'H', 'e', 'l', 'l', 'o', '\0'};
    printf("good: %s (len=%lu)\n", good, strlen(good));
    /* "Hello" (len=5) */

    /* DANGEREUX : sans \0 */
    char bad[5] = {'H', 'e', 'l', 'l', 'o'};  /* PAS de \0 ! */
    printf("bad: %s (len=%lu)\n", bad, strlen(bad));
    /* "Hello" suivi de DECHETS memoire, longueur aleatoire */

    return (0);
}
```

```
Memoire sans \0 :

+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
| H | e | l | l | o | ? | ? | ? | ? | ? | ? | ? | ? | ? |
+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
 0   1   2   3   4   5   6   7   8   ...
                     ^
                     Pas de \0 !
                     strlen continue de lire → Comportement indefini
                     printf continue d'afficher → Dechets
```

### 3.2 sizeof vs strlen

| | `sizeof(str)` | `strlen(str)` |
|---|---|---|
| **Compte** | Taille totale en memoire | Nombre de caracteres avant `\0` |
| **Inclut `\0`** | Oui | Non |
| **Evalue a** | Compile-time (pour tableaux) | Runtime (parcourt la chaine) |
| **Sur un pointeur** | Taille du pointeur (4 ou 8) | Longueur de la chaine pointee |

```c
char s1[] = "Hello";
char *s2 = "Hello";

sizeof(s1)   /* 6  (5 chars + \0) */
sizeof(s2)   /* 8  (taille du pointeur sur 64-bit) */
strlen(s1)   /* 5  (caracteres avant \0) */
strlen(s2)   /* 5  (meme chose) */
```

---

## 4. Fonctions de \<string.h\>

### 4.1 strlen - Longueur d'une chaine

**Prototype** : `size_t strlen(const char *s);`

Retourne le nombre de caracteres avant `\0`.

```c
#include <string.h>

strlen("Hello");    /* 5 */
strlen("");         /* 0 */
strlen("A\0B");    /* 1 (s'arrete au premier \0) */
```

**Implementation manuelle** :

```c
int _strlen(char *s)
{
    int len = 0;

    while (s[len] != '\0')
        len++;
    return (len);
}
```

**Implementation avec pointeur** :

```c
int _strlen(char *s)
{
    char *start = s;

    while (*s)
        s++;
    return (s - start);
}
```

**Implementation recursive** :

```c
int _strlen_rec(char *s)
{
    if (*s == '\0')
        return (0);
    return (1 + _strlen_rec(s + 1));
}
```

---

### 4.2 strcpy - Copier une chaine

**Prototype** : `char *strcpy(char *dest, const char *src);`

Copie `src` dans `dest` (y compris le `\0`). Retourne `dest`.

> [!warning] Danger : aucune verification de taille
> `strcpy` ne verifie **pas** que `dest` est assez grand pour contenir `src`. Si `src` est plus long que `dest`, c'est un **buffer overflow**.

```c
char dest[10];
strcpy(dest, "Hello");    /* OK : 6 <= 10 */
strcpy(dest, "This is way too long");  /* BUFFER OVERFLOW ! */
```

**Implementation manuelle** :

```c
char *_strcpy(char *dest, char *src)
{
    int i = 0;

    while (src[i] != '\0')
    {
        dest[i] = src[i];
        i++;
    }
    dest[i] = '\0';
    return (dest);
}
```

**Version avec pointeurs** :

```c
char *_strcpy(char *dest, char *src)
{
    char *d = dest;

    while ((*d++ = *src++))
        ;
    return (dest);
}
```

---

### 4.3 strncpy - Copier avec limite

**Prototype** : `char *strncpy(char *dest, const char *src, size_t n);`

Copie au maximum `n` caracteres de `src` dans `dest`.

> [!warning] Piege : strncpy ne garantit PAS le \0
> Si `strlen(src) >= n`, strncpy ne met **pas** de `\0` a la fin de `dest`.

```c
char dest[6];

strncpy(dest, "Hello", 6);       /* OK : copie "Hello\0" */
strncpy(dest, "Hello World", 5); /* dest = "Hello" SANS \0 !! */
dest[5] = '\0';                  /* Toujours ajouter \0 manuellement */
```

**Implementation manuelle** :

```c
char *_strncpy(char *dest, char *src, int n)
{
    int i;

    for (i = 0; i < n && src[i] != '\0'; i++)
        dest[i] = src[i];
    for (; i < n; i++)
        dest[i] = '\0';
    return (dest);
}
```

---

### 4.4 strcmp - Comparer deux chaines

**Prototype** : `int strcmp(const char *s1, const char *s2);`

Compare caractere par caractere. Retourne :
- `0` si les chaines sont identiques
- `< 0` si `s1` < `s2` (premier caractere different est plus petit)
- `> 0` si `s1` > `s2`

```c
strcmp("abc", "abc");   /*  0 (identiques) */
strcmp("abc", "abd");   /* < 0 ('c' < 'd') */
strcmp("abd", "abc");   /* > 0 ('d' > 'c') */
strcmp("abc", "abcd");  /* < 0 ('\0' < 'd') */
strcmp("", "a");        /* < 0 */
```

> [!warning] Ne JAMAIS comparer des chaines avec ==
> `==` compare les **adresses** (les pointeurs), pas les **contenus**.
> ```c
> char *s1 = "hello";
> char *s2 = "hello";
> if (s1 == s2)        /* Compare les adresses, PAS les contenus ! */
> if (strcmp(s1, s2) == 0)  /* Compare les contenus : CORRECT */
> ```

**Implementation manuelle** :

```c
int _strcmp(char *s1, char *s2)
{
    int i = 0;

    while (s1[i] != '\0' && s1[i] == s2[i])
        i++;
    return (s1[i] - s2[i]);
}
```

---

### 4.5 strncmp - Comparer n premiers caracteres

**Prototype** : `int strncmp(const char *s1, const char *s2, size_t n);`

Comme `strcmp` mais compare au maximum `n` caracteres.

```c
strncmp("Hello", "Help", 3);   /* 0 ("Hel" == "Hel") */
strncmp("Hello", "Help", 4);   /* != 0 ('l' != 'p') */
```

**Implementation manuelle** :

```c
int _strncmp(char *s1, char *s2, int n)
{
    int i = 0;

    if (n == 0)
        return (0);
    while (i < n - 1 && s1[i] != '\0' && s1[i] == s2[i])
        i++;
    return (s1[i] - s2[i]);
}
```

---

### 4.6 strcat - Concatener

**Prototype** : `char *strcat(char *dest, const char *src);`

Ajoute `src` a la fin de `dest`. Retourne `dest`.

> [!warning] Danger : buffer overflow
> `dest` doit etre assez grand pour contenir `dest + src + \0`.

```c
char dest[20] = "Hello";
strcat(dest, " ");
strcat(dest, "World");
printf("%s\n", dest);    /* "Hello World" */
```

```
Avant strcat :
dest: | H | e | l | l | o | \0| ? | ? | ? | ? | ... |
                                ^
                                strcat commence ici

Apres strcat(dest, " World") :
dest: | H | e | l | l | o |   | W | o | r | l | d | \0| ... |
```

**Implementation manuelle** :

```c
char *_strcat(char *dest, char *src)
{
    int i = 0;
    int j = 0;

    /* Trouver la fin de dest */
    while (dest[i] != '\0')
        i++;
    /* Copier src a la fin */
    while (src[j] != '\0')
    {
        dest[i] = src[j];
        i++;
        j++;
    }
    dest[i] = '\0';
    return (dest);
}
```

---

### 4.7 strncat - Concatener avec limite

**Prototype** : `char *strncat(char *dest, const char *src, size_t n);`

Concatene au maximum `n` caracteres de `src`, puis ajoute `\0`.

> [!info] Contrairement a strncpy, strncat ajoute TOUJOURS \0

```c
char dest[20] = "Hello";
strncat(dest, " World!!", 6);
printf("%s\n", dest);    /* "Hello World" (seulement 6 chars de src) */
```

**Implementation manuelle** :

```c
char *_strncat(char *dest, char *src, int n)
{
    int i = 0;
    int j = 0;

    while (dest[i] != '\0')
        i++;
    while (j < n && src[j] != '\0')
    {
        dest[i] = src[j];
        i++;
        j++;
    }
    dest[i] = '\0';
    return (dest);
}
```

---

### 4.8 strchr - Chercher un caractere (premiere occurrence)

**Prototype** : `char *strchr(const char *s, int c);`

Retourne un pointeur vers la **premiere occurrence** de `c` dans `s`, ou `NULL` si non trouve.

```c
char *str = "Hello World";
char *p = strchr(str, 'o');

if (p != NULL)
    printf("Trouve a l'index %ld : '%s'\n", p - str, p);
/* "Trouve a l'index 4 : 'o World'" */
```

**Implementation manuelle** :

```c
char *_strchr(char *s, char c)
{
    while (*s != '\0')
    {
        if (*s == c)
            return (s);
        s++;
    }
    /* Verifier si c est '\0' */
    if (c == '\0')
        return (s);
    return (NULL);
}
```

---

### 4.9 strrchr - Chercher un caractere (derniere occurrence)

**Prototype** : `char *strrchr(const char *s, int c);`

Comme `strchr` mais retourne la **derniere** occurrence.

```c
char *str = "Hello World";
char *p = strrchr(str, 'o');
/* p pointe vers le 'o' de "World" (index 7) */
```

**Implementation manuelle** :

```c
char *_strrchr(char *s, char c)
{
    char *last = NULL;

    while (*s != '\0')
    {
        if (*s == c)
            last = s;
        s++;
    }
    if (c == '\0')
        return (s);
    return (last);
}
```

---

### 4.10 strstr - Chercher une sous-chaine

**Prototype** : `char *strstr(const char *haystack, const char *needle);`

Retourne un pointeur vers la premiere occurrence de `needle` dans `haystack`, ou `NULL`.

```c
char *str = "Hello World";
char *p = strstr(str, "World");
/* p pointe vers "World" */

printf("%s\n", p);   /* "World" */
printf("Index: %ld\n", p - str);  /* 6 */
```

**Implementation manuelle** :

```c
char *_strstr(char *haystack, char *needle)
{
    int i, j;

    if (needle[0] == '\0')
        return (haystack);

    for (i = 0; haystack[i] != '\0'; i++)
    {
        j = 0;
        while (haystack[i + j] == needle[j] && needle[j] != '\0')
            j++;
        if (needle[j] == '\0')
            return (haystack + i);
    }
    return (NULL);
}
```

---

### 4.11 strdup - Dupliquer avec malloc

**Prototype** : `char *strdup(const char *s);`

Alloue de la memoire avec `malloc` et copie la chaine dedans. **Doit etre liberee avec `free()`.**

```c
#include <string.h>
#include <stdlib.h>

char *original = "Hello";
char *copy = strdup(original);

/* copy est une copie independante sur le heap */
copy[0] = 'h';   /* OK : c'est notre memoire */

printf("%s\n", original);  /* "Hello" (pas modifie) */
printf("%s\n", copy);      /* "hello" */

free(copy);   /* OBLIGATOIRE : strdup utilise malloc */
```

> [!warning] strdup utilise malloc
> **Toujours** free() le resultat de strdup. Sinon c'est un **memory leak**.

**Implementation manuelle** :

```c
#include <stdlib.h>

char *_strdup(char *str)
{
    char *dup;
    int len;
    int i;

    if (str == NULL)
        return (NULL);

    len = 0;
    while (str[len])
        len++;

    dup = malloc(sizeof(char) * (len + 1));  /* +1 pour \0 */
    if (dup == NULL)
        return (NULL);

    for (i = 0; i <= len; i++)  /* <= pour copier le \0 */
        dup[i] = str[i];

    return (dup);
}
```

---

### 4.12 strtok - Tokenizer

**Prototype** : `char *strtok(char *str, const char *delim);`

Decoupe une chaine en tokens. **Attention** : cette fonction est **stateful** et **modifie** la chaine originale.

> [!warning] strtok modifie la chaine !
> strtok remplace les delimiteurs par `\0` dans la chaine originale. Ne jamais l'utiliser sur un string literal.

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char str[] = "Hello,World,Foo,Bar";  /* DOIT etre un tableau (modifiable) */
    char *token;

    /* Premier appel : passe la chaine */
    token = strtok(str, ",");
    while (token != NULL)
    {
        printf("Token: '%s'\n", token);
        /* Appels suivants : passe NULL */
        token = strtok(NULL, ",");
    }

    /* Resultat :
     * Token: 'Hello'
     * Token: 'World'
     * Token: 'Foo'
     * Token: 'Bar'
     */

    /* ATTENTION : str est maintenant "Hello\0World\0Foo\0Bar" */
    printf("str apres strtok : '%s'\n", str);  /* "Hello" */

    return (0);
}
```

```
Avant strtok :
str: | H | e | l | l | o | , | W | o | r | l | d | , | F | o | o | , | B | a | r | \0|

Apres strtok :
str: | H | e | l | l | o |\0 | W | o | r | l | d |\0 | F | o | o |\0 | B | a | r | \0|
                          ^^                       ^^                ^^
                       remplace par \0          remplace par \0   remplace par \0
```

---

## 5. Tableau recapitulatif des fonctions

| Fonction | Prototype | Sure ? | Alternative sure | Retour |
|---|---|---|---|---|
| `strlen` | `size_t strlen(const char *s)` | Oui | - | Longueur |
| `strcpy` | `char *strcpy(char *d, const char *s)` | **Non** | `strncpy`, `strlcpy` | `dest` |
| `strncpy` | `char *strncpy(char *d, const char *s, size_t n)` | Partiel | `strlcpy` | `dest` |
| `strcmp` | `int strcmp(const char *s1, const char *s2)` | Oui | - | 0, <0, >0 |
| `strncmp` | `int strncmp(const char *s1, const char *s2, size_t n)` | Oui | - | 0, <0, >0 |
| `strcat` | `char *strcat(char *d, const char *s)` | **Non** | `strncat`, `strlcat` | `dest` |
| `strncat` | `char *strncat(char *d, const char *s, size_t n)` | Partiel | `strlcat` | `dest` |
| `strchr` | `char *strchr(const char *s, int c)` | Oui | - | Pointeur ou NULL |
| `strrchr` | `char *strrchr(const char *s, int c)` | Oui | - | Pointeur ou NULL |
| `strstr` | `char *strstr(const char *h, const char *n)` | Oui | - | Pointeur ou NULL |
| `strdup` | `char *strdup(const char *s)` | Oui | - | Nouvelle chaine (malloc) |
| `strtok` | `char *strtok(char *s, const char *d)` | **Non** | `strtok_r` | Token ou NULL |

> [!tip] Fonctions sures (BSD/GNU)
> `strlcpy` et `strlcat` sont des alternatives plus sures (BSD). Elles garantissent toujours le `\0` et retournent la taille necessaire. Elles ne sont pas dans la norme C mais disponibles sur BSD/macOS et via `libbsd` sur Linux.

---

## 6. Fonctions de \<ctype.h\>

Ces fonctions testent ou transforment **un seul caractere** (pas une chaine).

```c
#include <ctype.h>
```

| Fonction | Description | Exemple |
|---|---|---|
| `isalpha(c)` | Est-ce une lettre ? | `isalpha('A')` → vrai |
| `isdigit(c)` | Est-ce un chiffre ? | `isdigit('3')` → vrai |
| `isalnum(c)` | Lettre ou chiffre ? | `isalnum('_')` → faux |
| `isspace(c)` | Espace, tab, newline ? | `isspace(' ')` → vrai |
| `isupper(c)` | Majuscule ? | `isupper('A')` → vrai |
| `islower(c)` | Minuscule ? | `islower('a')` → vrai |
| `isprint(c)` | Caractere imprimable ? | `isprint('\n')` → faux |
| `toupper(c)` | Convertir en majuscule | `toupper('a')` → `'A'` |
| `tolower(c)` | Convertir en minuscule | `tolower('A')` → `'a'` |

**Exemple : convertir une chaine en majuscules** :

```c
#include <stdio.h>
#include <ctype.h>

void to_uppercase(char *str)
{
    while (*str)
    {
        *str = toupper(*str);
        str++;
    }
}

int main(void)
{
    char str[] = "Hello, World!";
    to_uppercase(str);
    printf("%s\n", str);   /* "HELLO, WORLD!" */
    return (0);
}
```

**Exemple : verifier si une chaine est un nombre** :

```c
#include <ctype.h>

int is_number(char *s)
{
    if (*s == '-' || *s == '+')
        s++;
    if (*s == '\0')
        return (0);
    while (*s)
    {
        if (!isdigit(*s))
            return (0);
        s++;
    }
    return (1);
}
```

---

## 7. Fonctions de conversion

### 7.1 atoi - ASCII to integer

```c
#include <stdlib.h>

int n;
n = atoi("42");       /* n = 42 */
n = atoi("-17");      /* n = -17 */
n = atoi("0");        /* n = 0 */
n = atoi("hello");    /* n = 0 (pas d'erreur detectee !) */
n = atoi("42abc");    /* n = 42 (s'arrete au premier non-chiffre) */
```

> [!warning] atoi ne detecte pas les erreurs
> `atoi` retourne 0 pour une chaine non-numerique ET pour la chaine "0". Impossible de distinguer une erreur d'un vrai 0.

**Implementation manuelle** :

```c
int _atoi(char *s)
{
    int result = 0;
    int sign = 1;
    int i = 0;

    /* Sauter les espaces */
    while (s[i] == ' ' || (s[i] >= 9 && s[i] <= 13))
        i++;

    /* Gerer le signe */
    if (s[i] == '-')
    {
        sign = -1;
        i++;
    }
    else if (s[i] == '+')
        i++;

    /* Convertir les chiffres */
    while (s[i] >= '0' && s[i] <= '9')
    {
        result = result * 10 + (s[i] - '0');
        i++;
    }

    return (result * sign);
}
```

### 7.2 strtol - String to long (avec gestion d'erreur)

```c
#include <stdlib.h>
#include <errno.h>

char *endptr;
long n;

errno = 0;
n = strtol("42", &endptr, 10);  /* base 10 */

if (errno != 0)
    perror("strtol");            /* Overflow/underflow */
else if (*endptr != '\0')
    printf("Caracteres invalides: '%s'\n", endptr);
else
    printf("Nombre: %ld\n", n);  /* 42 */
```

```c
/* strtol supporte differentes bases */
strtol("0xFF", NULL, 16);   /* 255 (hexadecimal) */
strtol("0b1010", NULL, 2);  /* 10 (binaire, si supporte) */
strtol("077", NULL, 8);     /* 63 (octal) */
strtol("42", NULL, 0);      /* 42 (detection automatique de la base) */
```

### 7.3 Autres fonctions de conversion

| Fonction | Prototype | Description |
|---|---|---|
| `atoi` | `int atoi(const char *s)` | String → int (pas de gestion d'erreur) |
| `atol` | `long atol(const char *s)` | String → long |
| `atof` | `double atof(const char *s)` | String → double |
| `strtol` | `long strtol(const char *s, char **e, int base)` | String → long (avec erreur) |
| `strtoul` | `unsigned long strtoul(...)` | String → unsigned long |
| `strtod` | `double strtod(const char *s, char **e)` | String → double (avec erreur) |
| `sprintf` | `int sprintf(char *s, const char *fmt, ...)` | Formater → string |

> [!tip] Preference
> Utilise `strtol` au lieu de `atoi` quand tu as besoin de detecter les erreurs. `atoi` est bien pour les cas simples ou tu fais confiance a l'entree.

---

## 8. Manipulation manuelle de chaines

### 8.1 Parcourir caractere par caractere

```c
#include <stdio.h>

void print_chars(char *s)
{
    while (*s)
    {
        printf("[%c] ", *s);
        s++;
    }
    printf("\n");
}

/* print_chars("Hello") → [H] [e] [l] [l] [o] */
```

### 8.2 Inverser une chaine

```c
void reverse_string(char *s)
{
    int len = 0;
    int i;
    char tmp;

    /* Calculer la longueur */
    while (s[len])
        len++;

    /* Echanger les caracteres symetriques */
    for (i = 0; i < len / 2; i++)
    {
        tmp = s[i];
        s[i] = s[len - 1 - i];
        s[len - 1 - i] = tmp;
    }
}
```

```
Avant : | H | e | l | l | o | \0|
         ^               ^
         swap(0, 4)

        | o | e | l | l | H | \0|
              ^       ^
              swap(1, 3)

Apres : | o | l | l | e | H | \0|
```

### 8.3 Compter les mots

```c
#include <stdio.h>

int count_words(char *s)
{
    int count = 0;
    int in_word = 0;

    while (*s)
    {
        if (*s == ' ' || *s == '\t' || *s == '\n')
        {
            in_word = 0;
        }
        else if (!in_word)
        {
            in_word = 1;
            count++;
        }
        s++;
    }
    return (count);
}

/* count_words("  Hello   World  ") → 2 */
/* count_words("") → 0 */
/* count_words("   ") → 0 */
/* count_words("one") → 1 */
```

### 8.4 Convertir en majuscules (sans ctype.h)

```c
void to_upper(char *s)
{
    while (*s)
    {
        if (*s >= 'a' && *s <= 'z')
            *s = *s - 32;   /* ou : *s = *s - ('a' - 'A'); */
        s++;
    }
}
```

> [!info] Pourquoi -32 ?
> En ASCII, 'a' = 97 et 'A' = 65. La difference est 32.
> ```
> 'a' - 'A' = 97 - 65 = 32
> 'b' - 'B' = 98 - 66 = 32
> ...
> ```
> Chaque lettre minuscule est exactement 32 de plus que sa majuscule.

---

## 9. Strings et memoire

### 9.1 Quand utiliser malloc pour les strings

```c
/* 1. Quand la taille n'est pas connue a la compilation */
char *read_input(void)
{
    char *buffer = malloc(1024);
    if (buffer == NULL)
        return (NULL);
    fgets(buffer, 1024, stdin);
    return (buffer);   /* L'appelant doit free() */
}

/* 2. Quand la chaine doit survivre au scope de la fonction */
char *create_greeting(char *name)
{
    int len = strlen("Hello, ") + strlen(name) + strlen("!") + 1;
    char *greeting = malloc(len);
    if (greeting == NULL)
        return (NULL);
    sprintf(greeting, "Hello, %s!", name);
    return (greeting);  /* L'appelant doit free() */
}

/* 3. Quand tu construis une chaine dynamiquement */
char *concat(char *s1, char *s2)
{
    int len = strlen(s1) + strlen(s2) + 1;
    char *result = malloc(len);
    if (result == NULL)
        return (NULL);
    strcpy(result, s1);
    strcat(result, s2);
    return (result);  /* L'appelant doit free() */
}
```

### 9.2 Le piege du +1 pour \0

> [!warning] TOUJOURS ajouter +1 pour le \0

```c
char *src = "Hello";   /* strlen = 5 */

/* FAUX : oublie le \0 */
char *copy = malloc(strlen(src));       /* 5 octets → pas assez ! */
strcpy(copy, src);                      /* Buffer overflow ! */

/* CORRECT : +1 pour le \0 */
char *copy = malloc(strlen(src) + 1);   /* 6 octets → OK */
strcpy(copy, src);

/* CORRECT aussi avec sizeof(char) explicite */
char *copy = malloc(sizeof(char) * (strlen(src) + 1));
```

### 9.3 strdup vs malloc + strcpy

```c
/* Ces deux sont equivalents */

/* Methode 1 : strdup (plus simple) */
char *copy = strdup(original);

/* Methode 2 : malloc + strcpy (plus explicite) */
char *copy = malloc(strlen(original) + 1);
if (copy != NULL)
    strcpy(copy, original);

/* Les deux necessitent free(copy) apres utilisation */
```

---

## 10. Erreurs classiques

### 10.1 Oublier le \0

```c
/* FAUX */
char str[5];
str[0] = 'H';
str[1] = 'e';
str[2] = 'l';
str[3] = 'l';
str[4] = 'o';
printf("%s\n", str);   /* Dechets apres "Hello" */

/* CORRECT */
char str[6];
str[0] = 'H';
str[1] = 'e';
str[2] = 'l';
str[3] = 'l';
str[4] = 'o';
str[5] = '\0';         /* Ne pas oublier ! */
printf("%s\n", str);   /* "Hello" */
```

### 10.2 Buffer overflow avec strcpy

```c
char dest[5];
strcpy(dest, "Hello World");   /* OVERFLOW : 12 > 5 */

/* Solution 1 : utiliser strncpy */
strncpy(dest, "Hello World", sizeof(dest) - 1);
dest[sizeof(dest) - 1] = '\0';

/* Solution 2 : verifier la taille */
if (strlen(src) < sizeof(dest))
    strcpy(dest, src);
```

### 10.3 Modifier un string literal

```c
char *str = "Hello";
str[0] = 'h';         /* SEGFAULT ! (ou undefined behavior) */

/* Solution : utiliser un tableau */
char str[] = "Hello";
str[0] = 'h';         /* OK */

/* Ou : utiliser strdup pour obtenir une copie modifiable */
char *str = strdup("Hello");
str[0] = 'h';         /* OK */
free(str);
```

### 10.4 Oublier de free strdup

```c
void bad_function(void)
{
    char *str = strdup("Hello");
    printf("%s\n", str);
    /* MEMORY LEAK : str n'est jamais libere */
}

void good_function(void)
{
    char *str = strdup("Hello");
    if (str == NULL)
        return;
    printf("%s\n", str);
    free(str);         /* Toujours free ! */
}
```

### 10.5 Off-by-one errors

```c
/* Allouer la bonne taille */
char *s = "Hello";                     /* strlen = 5 */
char *copy = malloc(strlen(s));        /* FAUX : 5 octets (manque \0) */
char *copy = malloc(strlen(s) + 1);   /* CORRECT : 6 octets */

/* Boucle de copie */
for (i = 0; i < strlen(s); i++)       /* FAUX : ne copie pas \0 */
    copy[i] = s[i];

for (i = 0; i <= strlen(s); i++)      /* CORRECT : copie \0 aussi */
    copy[i] = s[i];
/* Note : appeler strlen dans la condition est inefficace (O(n) a chaque iteration) */
```

---

## 11. Pattern Holberton

Les projets Holberton demandent souvent de reimplementer les fonctions standard avec le prefixe `_`.

```c
/* Prototypes Holberton classiques */
int _strlen(char *s);
char *_strcpy(char *dest, char *src);
char *_strcat(char *dest, char *src);
char *_strncat(char *dest, char *src, int n);
char *_strncpy(char *dest, char *src, int n);
int _strcmp(char *s1, char *s2);
char *_strchr(char *s, char c);
char *_strstr(char *haystack, char *needle);
char *_strdup(char *str);
```

> [!tip] Regles pour les reimplementations
> 1. Ne pas utiliser les fonctions de `<string.h>` dans l'implementation
> 2. Gerer les cas limites (NULL, chaine vide)
> 3. Respecter le prototype exact demande
> 4. Style Betty (norme de code Holberton)
> 5. Pas plus de 5 variables par fonction
> 6. Pas plus de 40 lignes par fonction

**Exemple complet d'un fichier Holberton** :

```c
#include "main.h"

/**
 * _strlen - returns the length of a string
 * @s: the string to measure
 *
 * Return: the length of the string
 */
int _strlen(char *s)
{
	int len = 0;

	while (s[len])
		len++;

	return (len);
}
```

---

## 12. Exercices pratiques

### Exercice 1 : Fonctions de base

> [!example] A coder
> Reimplementer ces fonctions sans utiliser `<string.h>` :
> 1. `int _strlen(char *s);`
> 2. `char *_strcpy(char *dest, char *src);`
> 3. `int _strcmp(char *s1, char *s2);`
>
> Tester avec :
> ```c
> char dest[20];
> printf("len = %d\n", _strlen("Hello"));   /* 5 */
> _strcpy(dest, "Hello");
> printf("dest = %s\n", dest);              /* "Hello" */
> printf("cmp = %d\n", _strcmp("abc", "abd")); /* < 0 */
> ```

### Exercice 2 : Manipulation de chaines

> [!example] A coder
> 1. `void reverse_string(char *s);` - Inverser une chaine en place
> 2. `int count_words(char *s);` - Compter les mots (separes par espaces)
> 3. `void capitalize(char *s);` - Mettre en majuscule la premiere lettre de chaque mot

### Exercice 3 : Recherche

> [!example] A coder
> 1. `char *_strchr(char *s, char c);` - Premiere occurrence de c
> 2. `char *_strstr(char *h, char *n);` - Premiere occurrence de la sous-chaine
> 3. `int count_char(char *s, char c);` - Compter les occurrences de c

### Exercice 4 : Concatenation et duplication

> [!example] A coder
> 1. `char *_strcat(char *dest, char *src);`
> 2. `char *_strncat(char *dest, char *src, int n);`
> 3. `char *_strdup(char *str);` - Attention au malloc et au +1 !
> 4. `char *str_concat(char *s1, char *s2);` - Retourne une nouvelle chaine malloc'd qui est la concatenation de s1 et s2

### Exercice 5 : Tokenizer

> [!example] A coder
> Ecrire une fonction qui decoupe une chaine en mots et retourne un tableau de chaines :
> ```c
> char **split_string(char *str);
> /* "Hello World Foo" → {"Hello", "World", "Foo", NULL} */
> ```
> Indices :
> 1. Compter le nombre de mots
> 2. malloc un tableau de `char *` (nb_mots + 1 pour le NULL final)
> 3. Pour chaque mot : malloc et copier
> 4. Ne pas oublier de tout free() apres utilisation

### Exercice 6 : Projet complet - mini grep

> [!example] A coder
> Ecrire un programme qui :
> 1. Prend deux arguments : un pattern et un nom de fichier
> 2. Lit le fichier ligne par ligne
> 3. Affiche les lignes qui contiennent le pattern (utiliser ta _strstr)
> ```bash
> $ ./mini_grep "hello" test.txt
> 3: say hello to everyone
> 7: hello world
> ```

---

## 13. Resume

```
+-----------------------------------------------------------+
|           CHEAT SHEET CHAINES DE CARACTERES                |
+-----------------------------------------------------------+
|                                                            |
|  DECLARATION :                                             |
|    char s[] = "Hi";   → stack, modifiable, sizeof=3       |
|    char *s  = "Hi";   → .rodata, READ-ONLY                |
|                                                            |
|  TOUJOURS :                                                |
|    - Terminer par \0                                       |
|    - malloc(strlen + 1) pour le \0                         |
|    - free() apres strdup()                                 |
|    - Utiliser strcmp() pas ==                               |
|                                                            |
|  FONCTIONS SURES :                                         |
|    strlen, strcmp, strncmp, strchr, strrchr, strstr         |
|                                                            |
|  FONCTIONS DANGEREUSES (overflow possible) :               |
|    strcpy → strncpy/strlcpy                                |
|    strcat → strncat/strlcat                                |
|    strtok → strtok_r                                       |
|                                                            |
|  CONVERSION :                                              |
|    atoi (simple) vs strtol (avec erreur)                   |
|                                                            |
|  ERREURS A EVITER :                                        |
|    1. Oublier \0                                           |
|    2. Modifier un string literal                           |
|    3. malloc(strlen) sans +1                               |
|    4. Oublier free(strdup(...))                            |
|    5. Comparer avec == au lieu de strcmp                    |
|                                                            |
+-----------------------------------------------------------+
```

---

## Liens

- [[03b - Les Pointeurs]] - Les bases des pointeurs
- [[04 - Pointeurs et Memoire]] - malloc, free, gestion memoire
- [[01 - Securite Memoire en C]] - Buffer overflow et securite
- [[02 - Bases Numeriques]] - ASCII et representation des caracteres
