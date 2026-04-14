# Projet _printf

> [!info] Contexte
> Le projet `_printf` est un tournant dans l'apprentissage du C (Holberton / ALX). L'objectif n'est pas de repliquer `printf` ligne par ligne, mais de comprendre comment une fonction peut **parser une chaine de format**, gerer un **nombre variable d'arguments**, et **formater la sortie** -- exactement le type de defis rencontres en developpement logiciel professionnel.

---

## 1. Qu'est-ce que _printf ?

`_printf` est une reimplementation simplifiee de la fonction standard `printf`. Elle doit :

1. Parcourir une chaine de format caractere par caractere
2. Afficher les caracteres normaux tels quels
3. Detecter les **specificateurs de format** (precedees par `%`)
4. Recuperer l'argument correspondant dans la liste variadique
5. Formater et afficher la donnee
6. Retourner le nombre total de caracteres affiches

### Prototype

```c
int _printf(const char *format, ...);
```

### Fonctions et macros autorisees

- `write` (man 2 write) -- seul moyen d'afficher
- `malloc` / `free` -- gestion memoire
- `va_start`, `va_arg`, `va_end`, `va_copy` -- fonctions variadiques

> [!warning] Interdit
> `printf`, `puts`, `putchar` et toute autre fonction de la libc pour l'affichage.
> Seul `write` est autorise !

---

## 2. Architecture du projet

### Vue d'ensemble

```
main.h              --> Prototypes, includes, macros, structures
_printf.c           --> Fonction principale + helpers locaux
print_characters.c  --> Gestion de %c (caractere)
print_string.c      --> Gestion de %s (chaine)
print_decimal.c     --> Gestion de %d et %i (entiers)
print_hexadecimal.c --> Gestion de %x et %X (hexadecimal)
print_octal.c       --> Gestion de %o (octal)
utils.c             --> Fonctions utilitaires (conversion de base)
```

### Flowchart de _printf

```
[START: _printf(format, ...)]
         |
         v
    format == NULL ?
    /            \
  Oui            Non
   |              |
   v              v
Return -1    Initialiser va_list
             Obtenir table des subprinters
                  |
                  v
         +-> format[i] == '\0' ? ---Oui---> va_end()
         |        |                          Return total
         |       Non
         |        |
         |        v
         |   format[i] == '%' ?
         |   /              \
         |  Non              Oui
         |   |                |
         |   v                v
         |  Afficher       Verifier format[i+1]
         |  format[i]     /    |    \        \
         |   |          '%'  Valide  '\0'  Inconnu
         |   |           |     |      |       |
         |   |      Print  Deleguer  Stop  Print %
         |   |       '%'   au sub-         puis le
         |   |             printer         caractere
         |   |           |     |              |
         |   v           v     v              v
         |  i++        i += 2              i += 2
         |   |           |                    |
         +---+-----------+--------------------+
```

---

## 3. Les fonctions variadiques

> [!info] Prerequis
> Voir [[06 - Fonctions Variadiques]] pour une explication detaillee.

Les fonctions variadiques acceptent un nombre **variable** d'arguments. C'est ce qui permet d'ecrire `_printf("Nom: %s, Age: %d", nom, age)`.

### Les 4 macros de `<stdarg.h>`

```c
#include <stdarg.h>

int _printf(const char *format, ...)
{
    va_list args;       /* Declare la liste d'arguments */

    va_start(args, format);  /* Initialise la liste apres "format" */

    /* ... dans une sous-fonction ... */
    char c = (char)va_arg(args, int);    /* Recupere un char (promu en int) */
    char *s = va_arg(args, char *);      /* Recupere une chaine */
    int n = va_arg(args, int);           /* Recupere un entier */

    va_end(args);       /* Nettoie la liste */
}
```

> [!warning] Promotion des types
> En C, les arguments variadiques subissent des **promotions par defaut** :
> - `char` et `short` sont promus en `int`
> - `float` est promu en `double`
>
> Donc : `va_arg(args, char)` est **INCORRECT** ! Utiliser `va_arg(args, int)`.

---

## 4. La table de dispatch (Jump Table)

Au lieu d'utiliser une chaine de `if/else` ou un `switch` enorme, on utilise un **tableau de pointeurs de fonctions** indexe par le caractere ASCII.

### Principe

```c
int (**get_subprinters(void))(va_list)
{
    static int (*table[128])(va_list); /* 128 entrees, toutes NULL */

    table['c'] = print_character;       /* ASCII 99  */
    table['s'] = print_string;          /* ASCII 115 */
    table['d'] = print_decimal;         /* ASCII 100 */
    table['i'] = print_decimal;         /* ASCII 105 */
    table['o'] = print_octal;           /* ASCII 111 */
    table['x'] = print_hexadecimal_lowercase;
    table['X'] = print_hexadecimal_uppercase;

    return (table);
}
```

> [!tip] Pourquoi cette approche est elegante
> - **Acces O(1)** : pas besoin de chercher dans une liste, l'index ASCII est la cle
> - **Extensible** : pour ajouter `%u`, il suffit d'ajouter `table['u'] = print_unsigned;`
> - **Propre** : separe la logique de routage de la logique d'affichage

### Alternative classique : tableau de structures

```c
typedef struct specifier
{
    char *spec;           /* Le caractere apres % */
    int (*f)(va_list);    /* La fonction associee */
} spec_t;

spec_t specifiers[] = {
    {"c", print_character},
    {"s", print_string},
    {"d", print_decimal},
    {"i", print_decimal},
    {NULL, NULL}  /* Sentinelle de fin */
};
```

---

## 5. Le fichier header : main.h

```c
#ifndef MAIN_H
#define MAIN_H

#include <stdarg.h>
#include <stddef.h>

/* Fonction principale */
int _printf(const char *format, ...);

/* Sous-fonctions d'affichage (une par specificateur) */
int print_character(va_list args);
int print_string(va_list args);
int print_decimal(va_list args);
int print_unsigned(va_list args);
int print_octal(va_list args);
int print_hexadecimal_lowercase(va_list args);
int print_hexadecimal_uppercase(va_list args);

/* Fonctions utilitaires */
#define CONVERT_MAX_BUFFER 65
char *convert_unsigned_decimal_up_to_base_16(unsigned int n,
    unsigned int base, char *buffer, size_t buffer_size);
char *convert_signed_decimal_up_to_base_16(int n,
    unsigned int base, char *buffer, size_t buffer_size);

#endif /* MAIN_H */
```

---

## 6. Implementation : _printf.c

### Helpers locaux

```c
/**
 * print_single_char - Affiche un seul caractere via write.
 * @c: Le caractere a afficher.
 * @total: Pointeur vers le compteur total (incremente en place).
 * Return: >= 0 si succes, -1 si erreur.
 */
int print_single_char(char c, int *total)
{
    int result;

    result = write(1, &c, 1);
    if (result >= 0)
        *total += result;

    return (result);
}

/**
 * delegate_to - Delegue l'affichage a une sous-fonction.
 * @printer: Pointeur vers la fonction d'affichage specialisee.
 * @components: Liste variadique a propager.
 * @total: Pointeur vers le compteur total.
 * Return: >= 0 si succes, -1 si erreur.
 */
int delegate_to(int (*printer)(va_list), va_list components, int *total)
{
    int count;

    count = printer(components);
    if (count >= 0)
        *total += count;

    return (count);
}
```

### La fonction principale

```c
/**
 * _printf - Affiche des chaines formatees sur la sortie standard.
 * @format: Chaine de format avec des specificateurs (%c, %s, %d...).
 * @...: Arguments variadiques correspondants.
 * Return: Nombre total de caracteres affiches, ou -1 si erreur.
 */
int _printf(const char *format, ...)
{
    int (**subprinters)(va_list);
    va_list components;
    int total = 0;
    unsigned int i = 0;
    int spr = 0; /* Sous-printer return */

    if (format == NULL)           /* Clause de garde */
        return (-1);

    subprinters = get_subprinters();
    va_start(components, format);

    while (format[i])
    {
        if (format[i] == '%' && format[i + 1] != '\0')
        {
            if (format[i + 1] == '%')
                spr = print_single_char('%', &total);
            else if (subprinters[(int)format[i + 1]])
                spr = delegate_to(
                    subprinters[(int)format[i + 1]], components, &total);
            else
            {
                spr = print_single_char(format[i], &total);
                spr = print_single_char(format[i + 1], &total);
            }
            i += 2;
            continue;
        }
        else
        {
            spr = print_single_char(format[i], &total);
            i++;
        }
        if (spr < 0)
            return (-1);
    }

    va_end(components);
    return (total);
}
```

---

## 7. Les sous-fonctions d'affichage

### print_character (%c)

```c
#include <stdarg.h>
#include <unistd.h>

int print_character(va_list components)
{
    char c;

    c = (char)va_arg(components, int);  /* char promu en int ! */
    return (write(1, &c, 1));
}
```

### print_string (%s)

```c
int print_string(va_list args)
{
    char *str;
    int i = 0;

    str = va_arg(args, char *);

    /* Gestion du cas NULL : printf standard affiche "(null)" */
    if (str == NULL)
        str = "(null)";

    while (str[i] != '\0')
    {
        write(1, &str[i], 1);
        i++;
    }

    return (i);
}
```

> [!warning] Cas NULL
> Si l'utilisateur passe `NULL` comme argument pour `%s`, le vrai `printf` affiche `(null)`. Il faut reproduire ce comportement.

### print_decimal (%d, %i)

```c
/**
 * recursive_int - Affiche un nombre recursivement.
 * @n: Le nombre (unsigned pour gerer INT_MIN).
 * Return: Nombre de caracteres affiches.
 */
int recursive_int(unsigned int n)
{
    int count = 0;
    char digit;

    if (n / 10)
        count += recursive_int(n / 10);

    digit = (n % 10) + '0';
    write(1, &digit, 1);
    return (count + 1);
}

/**
 * print_decimal - Gere les specificateurs %d et %i.
 * @args: Liste variadique.
 * Return: Nombre de caracteres affiches.
 */
int print_decimal(va_list args)
{
    int n = va_arg(args, int);
    unsigned int abs_n;
    int count = 0;

    if (n < 0)
    {
        write(1, "-", 1);
        count++;
        abs_n = (unsigned int)(-n);
    }
    else
        abs_n = (unsigned int)n;

    count += recursive_int(abs_n);
    return (count);
}
```

> [!tip] Pourquoi unsigned pour la valeur absolue ?
> `INT_MIN` (-2147483648) n'a pas de representation positive en `int`
> (INT_MAX = 2147483647). En utilisant `unsigned int`, on peut stocker
> la valeur absolue sans overflow.

### print_hexadecimal (%x, %X)

```c
static int hex_helper(unsigned int n, char *base)
{
    int count = 0;
    char c;

    if (n / 16)
        count += hex_helper(n / 16, base);

    c = base[n % 16];
    write(1, &c, 1);

    return (count + 1);
}

int print_hexadecimal_lowercase(va_list args)
{
    unsigned int n = va_arg(args, unsigned int);
    return (hex_helper(n, "0123456789abcdef"));
}

int print_hexadecimal_uppercase(va_list args)
{
    unsigned int n = va_arg(args, unsigned int);
    return (hex_helper(n, "0123456789ABCDEF"));
}
```

---

## 8. Fonction utilitaire : Conversion de base

```c
char *convert_unsigned_decimal_up_to_base_16(unsigned int n,
    unsigned int base, char *buffer, size_t buffer_size)
{
    char *digit_writer;
    char *digits = "0123456789abcdef";

    if (!n || base < 2 || base > 16 || buffer_size > CONVERT_MAX_BUFFER)
        return ("");

    digit_writer = buffer + buffer_size;
    *digit_writer = '\0';

    do {
        *(--digit_writer) = digits[n % base];
        n = n / base;
    } while (n);

    return (digit_writer);
}
```

> [!info] Pourquoi ecrire de droite a gauche ?
> On obtient les chiffres du **moins significatif au plus significatif**
> (via `n % base`). En ecrivant depuis la fin du buffer, on obtient
> directement la chaine dans le bon ordre sans devoir l'inverser.

---

## 9. Gestion des cas limites (Edge Cases)

| Cas | Comportement attendu |
|-----|---------------------|
| `_printf(NULL)` | Retourne -1 |
| `_printf("%s", NULL)` | Affiche `(null)` |
| `_printf("%%")` | Affiche `%` |
| `_printf("%")` | Comportement indefini (afficher `%` ou rien) |
| `_printf("%d", INT_MIN)` | Affiche `-2147483648` |
| `_printf("%d", 0)` | Affiche `0` |
| `_printf("%x", 0)` | Affiche `0` |
| Erreur de `write` | Retourne -1 immediatement |

---

## 10. Guide d'implementation etape par etape

### Etape 1 : Le squelette

Faire fonctionner `_printf("Hello")` qui affiche "Hello" et retourne 5.

### Etape 2 : %c, %s, %%

- `%c` : Recuperer un `int` via `va_arg`, le caster en `char`, l'afficher
- `%s` : Recuperer un `char *`, boucler et afficher chaque caractere
- `%%` : Simplement afficher un `%`

### Etape 3 : %d, %i

La partie technique : transformer un nombre en caracteres. Gerer le signe negatif et `INT_MIN`.

### Etape 4 : %u, %o, %x, %X

Conversions en base 10 (unsigned), 8, 16 minuscule, 16 majuscule.

### Etape 5 : Gestion des erreurs

Verifier les retours de `write`, gerer `format == NULL`, propager les erreurs.

---

## 11. Compilation et tests

```bash
# Compilation du projet
gcc -Wall -Werror -Wextra -pedantic -std=gnu89 -Wno-format *.c -o printf

# Le flag -Wno-format empeche gcc de se plaindre que _printf
# n'est pas le vrai printf (il ne reconnait pas nos formats)
```

### Fichier de test

```c
#include <stdio.h>
#include "main.h"

int main(void)
{
    int len1, len2;

    len1 = _printf("Caractere: [%c]\n", 'H');
    len2 = printf("Caractere: [%c]\n", 'H');
    _printf("Longueur _printf: %d\n", len1);
    printf("Longueur  printf: %d\n", len2);

    _printf("Chaine: [%s]\n", "Hello!");
    _printf("NULL: [%s]\n", (char *)NULL);
    _printf("Nombre: [%d]\n", -762534);
    _printf("Hexa: [%x, %X]\n", 255, 255);  /* ff, FF */
    _printf("Pourcent: [%%]\n");

    return (0);
}
```

---

## 12. Organisation du travail en equipe (projet a 2)

> [!tip] Strategie de collaboration
> **Partenaire A** (Le Gestionnaire de Flux) :
> - `_printf.c` : la boucle principale
> - La table de dispatch (`get_subprinters`)
> - `%c`, `%s`, `%%`
>
> **Partenaire B** (Le Mathematicien) :
> - `print_decimal.c` : `%d`, `%i`
> - `print_hexadecimal.c` : `%x`, `%X`
> - `print_octal.c` : `%o`
> - `utils.c` : conversions de base
>
> **Ensemble** :
> - `main.h` : le "contrat" entre les deux
> - Tests et integration

### Workflow Git recommande

```bash
# Chacun travaille sur sa branche
git checkout -b feature-chars     # Partenaire A
git checkout -b feature-numbers   # Partenaire B

# Ne jamais push directement sur main sans relecture
git push origin feature-chars
# Puis creer une Pull Request sur GitHub
```

---

## 13. Exercices

> [!example] Exercice 1 : Implementer %b (binaire)
> Ajouter le support du specificateur `%b` qui affiche un entier en binaire.
> Exemple : `_printf("%b", 10)` affiche `1010`.
> Indice : reutiliser la logique de `hex_helper` avec base 2.

> [!example] Exercice 2 : Implementer %p (pointeur)
> Ajouter le support de `%p` pour afficher une adresse memoire.
> Exemple : `_printf("%p", ptr)` affiche `0x7ffeabc12340`.
> Indice : caster en `unsigned long`, afficher "0x" puis la valeur en hexa.

> [!example] Exercice 3 : Buffer d'ecriture
> Modifier `_printf` pour utiliser un buffer interne de 1024 octets au lieu
> d'appeler `write` pour chaque caractere. Faire un seul `write` a la fin
> (ou quand le buffer est plein).

> [!example] Exercice 4 : Man page
> Ecrire une page de manuel (`man_3_printf`) pour votre implementation
> en utilisant le format troff/groff.

---

## 14. Liens

- [[05 - Pointeurs de Fonctions]] - La table de dispatch repose sur les pointeurs de fonctions
- [[06 - Fonctions Variadiques]] - va_list, va_start, va_arg, va_end
- [[03 - Structures et Typedef]] - Les structures spec_t pour le dispatch
- [[09 - File IO et Appels Systeme]] - write() est le seul moyen d'afficher
- [[01 - Introduction au C et Compilation]] - Flags de compilation et style Betty
- [[07 - Recursion]] - Les fonctions d'affichage des nombres utilisent la recursion
- [[04 - Arbres Binaires]] - Autre projet majeur Holberton
