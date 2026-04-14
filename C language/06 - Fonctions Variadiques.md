# 06 - Fonctions Variadiques

## Définition

Une **fonction variadique** est une fonction qui accepte un **nombre variable d'arguments**. Le nombre et le type des arguments ne sont pas fixés à la compilation — ils sont déterminés à l'exécution.

L'exemple le plus connu est `printf` :
```c
printf("Hello");                    /* 1 argument */
printf("Age: %d", 25);             /* 2 arguments */
printf("%s a %d ans", "Alice", 25); /* 3 arguments */
```

> [!info] Analogie jeu vidéo
> Imagine une attaque spéciale qui cible un nombre variable d'ennemis. Parfois tu frappes 1 ennemi, parfois 5. La même compétence s'adapte. En C, c'est exactement ce que fait une fonction variadique : elle s'adapte au nombre d'arguments reçus.

---

## Le header indispensable

```c
#include <stdarg.h>
```

Ce header fournit les 4 macros nécessaires pour manipuler les arguments variables.

---

## Les 4 macros (arbre de compétences)

```
  ┌──────────────────────────────────────────────────────────┐
  │                    ARBRE DE COMPÉTENCES                  │
  │                                                          │
  │  1. va_list     →  Déclarer la liste d'arguments         │
  │       │                                                  │
  │       ▼                                                  │
  │  2. va_start    →  Initialiser (pointer au 1er arg)      │
  │       │                                                  │
  │       ▼                                                  │
  │  3. va_arg      →  Récupérer l'argument suivant          │
  │       │              (avance automatiquement)             │
  │       ▼                                                  │
  │  4. va_end      →  Nettoyer (obligatoire !)              │
  │                                                          │
  │  Bonus: va_copy →  Copier une va_list pour double        │
  │                    parcours                              │
  └──────────────────────────────────────────────────────────┘
```

| Macro                         | Rôle                                                 |
| ----------------------------- | ---------------------------------------------------- |
| `va_list ap;`                 | Déclare un objet de type va_list (pointeur interne)  |
| `va_start(ap, last_named);`  | Initialise ap après le dernier paramètre nommé       |
| `va_arg(ap, type);`          | Récupère le prochain argument du type donné          |
| `va_end(ap);`                | Nettoie la va_list (obligatoire avant return)        |
| `va_copy(dest, src);`        | Copie une va_list (pour parcourir plusieurs fois)    |

---

## Structure squelette à mémoriser

```c
#include <stdarg.h>

/**
 * ma_fonction - Fonction variadique modèle
 * @n: Nombre d'arguments (ou autre info pour savoir quand s'arrêter)
 * @...: Les arguments variables
 *
 * Return: Dépend de la fonction
 */
int ma_fonction(int n, ...)
{
    va_list ap;          /* 1. Déclarer */
    int i, val;

    va_start(ap, n);     /* 2. Initialiser après le dernier paramètre nommé */

    for (i = 0; i < n; i++)
    {
        val = va_arg(ap, int);  /* 3. Récupérer chaque argument */
        /* ... traitement de val ... */
    }

    va_end(ap);          /* 4. Nettoyer */

    return (/* résultat */);
}
```

> [!warning] Les `...` doivent être le DERNIER paramètre
> ```c
> int ok(int n, ...);         /* CORRECT */
> int ko(..., int n);         /* ERREUR : ... doit être en dernier */
> int ko2(...);               /* ERREUR : au moins 1 paramètre nommé requis */
> ```

---

## Boss 1 : Somme de N entiers

```c
#include <stdio.h>
#include <stdarg.h>

/**
 * sum_all - Calcule la somme de N entiers
 * @n: Le nombre d'arguments à sommer
 * @...: Les entiers à sommer
 *
 * Return: La somme, ou 0 si n <= 0
 */
int sum_all(int n, ...)
{
    va_list ap;
    int i, sum;

    if (n <= 0)
        return (0);

    va_start(ap, n);

    sum = 0;
    for (i = 0; i < n; i++)
        sum += va_arg(ap, int);

    va_end(ap);

    return (sum);
}

int main(void)
{
    printf("sum(3, 10, 20, 30) = %d\n", sum_all(3, 10, 20, 30));
    /* Affiche : 60 */

    printf("sum(5, 1, 2, 3, 4, 5) = %d\n", sum_all(5, 1, 2, 3, 4, 5));
    /* Affiche : 15 */

    printf("sum(0) = %d\n", sum_all(0));
    /* Affiche : 0 */

    return (0);
}
```

---

## Boss 2 : Afficher N chaînes

```c
#include <stdio.h>
#include <stdarg.h>

/**
 * print_strings - Affiche N chaînes séparées par un séparateur
 * @separator: La chaîne de séparation
 * @n: Le nombre de chaînes
 * @...: Les chaînes (char *)
 */
void print_strings(const char *separator, unsigned int n, ...)
{
    va_list ap;
    unsigned int i;
    char *str;

    va_start(ap, n);

    for (i = 0; i < n; i++)
    {
        str = va_arg(ap, char *);

        if (str == NULL)
            printf("(nil)");
        else
            printf("%s", str);

        if (separator != NULL && i < n - 1)
            printf("%s", separator);
    }

    printf("\n");
    va_end(ap);
}

int main(void)
{
    print_strings(", ", 3, "Alice", "Bob", "Charlie");
    /* Affiche : Alice, Bob, Charlie */

    print_strings(" - ", 4, "Lundi", "Mardi", NULL, "Jeudi");
    /* Affiche : Lundi - Mardi - (nil) - Jeudi */

    print_strings(NULL, 2, "Hello", "World");
    /* Affiche : HelloWorld */

    return (0);
}
```

---

## Boss 3 : Mini `_printf`

```c
#include <stdio.h>
#include <stdarg.h>
#include <unistd.h>

/**
 * _putchar - Écrit un caractère sur stdout
 * @c: Le caractère
 *
 * Return: 1 en cas de succès
 */
int _putchar(char c)
{
    return (write(1, &c, 1));
}

/**
 * print_number - Affiche un entier (gère les négatifs)
 * @n: L'entier
 *
 * Return: Nombre de caractères imprimés
 */
int print_number(int n)
{
    int count = 0;
    unsigned int num;

    if (n < 0)
    {
        count += _putchar('-');
        num = -n;
    }
    else
    {
        num = n;
    }

    if (num / 10)
        count += print_number(num / 10);

    count += _putchar((num % 10) + '0');

    return (count);
}

/**
 * _printf - Version simplifiée de printf
 * @format: Chaîne de format (supporte %c, %s, %d, %%)
 * @...: Arguments variables
 *
 * Return: Nombre de caractères imprimés
 */
int _printf(const char *format, ...)
{
    va_list ap;
    int i, count;
    char *str;

    if (format == NULL)
        return (-1);

    va_start(ap, format);
    count = 0;

    for (i = 0; format[i] != '\0'; i++)
    {
        if (format[i] == '%' && format[i + 1] != '\0')
        {
            i++;
            switch (format[i])
            {
                case 'c':
                    count += _putchar(va_arg(ap, int));
                    break;
                case 's':
                    str = va_arg(ap, char *);
                    if (str == NULL)
                        str = "(null)";
                    while (*str)
                        count += _putchar(*str++);
                    break;
                case 'd':
                case 'i':
                    count += print_number(va_arg(ap, int));
                    break;
                case '%':
                    count += _putchar('%');
                    break;
                default:
                    count += _putchar('%');
                    count += _putchar(format[i]);
                    break;
            }
        }
        else
        {
            count += _putchar(format[i]);
        }
    }

    va_end(ap);

    return (count);
}

int main(void)
{
    int len;

    len = _printf("Hello %s! Tu as %d ans.\n", "Alice", 25);
    _printf("Caractères imprimés : %d\n", len);

    _printf("Caractère : %c\n", 'A');
    _printf("Pourcentage : %%\n");
    _printf("Nombre négatif : %d\n", -42);

    return (0);
}
```

---

## Promotions d'arguments par défaut

Quand des arguments sont passés via `...`, le C applique des **promotions automatiques** :

| Type original         | Promu vers    | Conséquence pour `va_arg`       |
| --------------------- | ------------- | ------------------------------- |
| `char`                | `int`         | Utiliser `va_arg(ap, int)`      |
| `short`               | `int`         | Utiliser `va_arg(ap, int)`      |
| `float`               | `double`      | Utiliser `va_arg(ap, double)`   |
| `int`                 | `int`         | Pas de changement               |
| `double`              | `double`      | Pas de changement               |
| `char *`              | `char *`      | Pas de changement               |

> [!warning] Erreurs fréquentes avec les promotions
> ```c
> /* FAUX — char est promu en int */
> char c = va_arg(ap, char);      /* Comportement indéfini ! */
>
> /* CORRECT */
> char c = (char)va_arg(ap, int);
>
> /* FAUX — float est promu en double */
> float f = va_arg(ap, float);    /* Comportement indéfini ! */
>
> /* CORRECT */
> float f = (float)va_arg(ap, double);
> ```

---

## Technique sentinelle (NULL comme marqueur de fin)

Au lieu de passer le nombre d'arguments, on utilise un marqueur de fin (souvent `NULL` ou une valeur spéciale) :

```c
#include <stdio.h>
#include <stdarg.h>

/**
 * print_all_strings - Affiche toutes les chaînes jusqu'au NULL sentinelle
 * @first: La première chaîne
 * @...: Les chaînes suivantes, terminées par NULL
 */
void print_all_strings(char *first, ...)
{
    va_list ap;
    char *str;

    if (first == NULL)
        return;

    printf("%s", first);

    va_start(ap, first);

    while ((str = va_arg(ap, char *)) != NULL)
        printf(", %s", str);

    printf("\n");
    va_end(ap);
}

int main(void)
{
    print_all_strings("Alice", "Bob", "Charlie", NULL);
    /* Affiche : Alice, Bob, Charlie */

    print_all_strings("Seul", NULL);
    /* Affiche : Seul */

    return (0);
}
```

> [!tip] La sentinelle est courante en C
> - `execl("/bin/ls", "ls", "-l", NULL)` — les arguments se terminent par NULL
> - `execlp("echo", "echo", "hello", NULL)`
> - C'est plus élégant que de compter les arguments quand le nombre est vraiment variable

---

## `va_copy` : double parcours

Parfois on a besoin de parcourir les arguments **deux fois** (ex : calculer la taille totale d'abord, puis traiter). On ne peut pas rembobiner une `va_list`, il faut la **copier** :

```c
#include <stdio.h>
#include <stdarg.h>
#include <stdlib.h>
#include <string.h>

/**
 * concat_all - Concatène N chaînes en une seule
 * @n: Nombre de chaînes
 * @...: Les chaînes
 *
 * Return: Nouvelle chaîne allouée, ou NULL
 */
char *concat_all(int n, ...)
{
    va_list ap, ap_copy;
    int i, total_len;
    char *result, *str;

    va_start(ap, n);
    va_copy(ap_copy, ap);  /* Copier pour le 2ème parcours */

    /* Premier parcours : calculer la taille totale */
    total_len = 0;
    for (i = 0; i < n; i++)
    {
        str = va_arg(ap, char *);
        if (str != NULL)
            total_len += strlen(str);
    }
    va_end(ap);

    /* Allouer */
    result = malloc(sizeof(char) * (total_len + 1));
    if (result == NULL)
    {
        va_end(ap_copy);
        return (NULL);
    }
    result[0] = '\0';

    /* Deuxième parcours : concaténer */
    for (i = 0; i < n; i++)
    {
        str = va_arg(ap_copy, char *);
        if (str != NULL)
            strcat(result, str);
    }
    va_end(ap_copy);

    return (result);
}

int main(void)
{
    char *result;

    result = concat_all(3, "Hello", " ", "World!");
    if (result != NULL)
    {
        printf("%s\n", result);  /* Hello World! */
        free(result);
    }

    return (0);
}
```

> [!warning] Ne pas oublier `va_end` sur la copie !
> Toute `va_list` initialisée (que ce soit par `va_start` ou `va_copy`) **doit** être terminée par `va_end`.

---

## Récapitulatif en 6 étapes

```
  Étape 1 : #include <stdarg.h>
  Étape 2 : Déclarer    →  va_list ap;
  Étape 3 : Initialiser →  va_start(ap, dernier_param_nommé);
  Étape 4 : Récupérer   →  val = va_arg(ap, type);  (en boucle)
  Étape 5 : Nettoyer    →  va_end(ap);
  Étape 6 : Retourner   →  return (résultat);
```

---

## La fonction `print_all` (exercice Holberton classique)

```c
#include <stdio.h>
#include <stdarg.h>

/**
 * print_all - Affiche des arguments de types variés
 * @format: Liste de types (c=char, i=int, f=float, s=string)
 * @...: Les arguments correspondants
 */
void print_all(const char * const format, ...)
{
    va_list ap;
    int i;
    char *sep = "";
    char *str;

    va_start(ap, format);

    i = 0;
    while (format != NULL && format[i] != '\0')
    {
        switch (format[i])
        {
            case 'c':
                printf("%s%c", sep, va_arg(ap, int));
                break;
            case 'i':
                printf("%s%d", sep, va_arg(ap, int));
                break;
            case 'f':
                printf("%s%f", sep, va_arg(ap, double));
                break;
            case 's':
                str = va_arg(ap, char *);
                if (str == NULL)
                    str = "(nil)";
                printf("%s%s", sep, str);
                break;
            default:
                i++;
                continue;
        }
        sep = ", ";
        i++;
    }

    printf("\n");
    va_end(ap);
}

int main(void)
{
    print_all("ceis", 'A', 42, 3.14, "Hello");
    /* Affiche : A, 42, 3.140000, Hello */

    return (0);
}
```

---

## Erreurs courantes

### 1. Oublier `va_end`

```c
/* FAUX — fuite de ressources / comportement indéfini */
int sum(int n, ...)
{
    va_list ap;
    va_start(ap, n);
    /* ... */
    return (total);  /* va_end oublié ! */
}

/* CORRECT */
int sum(int n, ...)
{
    va_list ap;
    va_start(ap, n);
    /* ... */
    va_end(ap);
    return (total);
}
```

### 2. Mauvais type dans `va_arg`

```c
/* L'appelant passe un int, mais on lit un char * → CRASH */
printf("%d\n", va_arg(ap, char *));  /* Mauvais type ! */

/* L'appelant passe un float, mais il est promu en double */
float f = va_arg(ap, float);   /* FAUX ! */
float f = va_arg(ap, double);  /* CORRECT */
```

### 3. Lire trop d'arguments

```c
/* Si n = 3 mais on lit 5 fois → comportement indéfini */
for (i = 0; i < 5; i++)
    val = va_arg(ap, int);  /* Les 2 derniers lectures sont invalides */
```

### 4. Oublier le cast pour char

```c
/* Le char est promu en int dans les variadics */
char c = va_arg(ap, int);     /* CORRECT (le cast implicite suffit) */
char c = va_arg(ap, char);    /* FAUX — comportement indéfini */
```

---

## Conseils pour les projets Holberton

> [!tip] Tips pour le projet `_printf`
> 1. Créer un **tableau de structs** associant spécificateur → fonction (voir [[05 - Pointeurs de Fonctions]])
> 2. Chaque fonction de traitement prend une `va_list` en paramètre
> 3. Gérer les cas limites : `NULL`, nombres négatifs, `%%`
> 4. Compter et retourner le nombre de caractères imprimés
> 5. Ne pas oublier `va_end` même en cas d'erreur (early return)

---

## Liens

- [[01 - Introduction au C et Compilation]] — printf et la bibliothèque standard
- [[05 - Pointeurs de Fonctions]] — Tableau de dispatch pour _printf
- [[07 - Recursion]] — print_number utilise la récursion

---

## Exercices pratiques

### Exercice 1 : `sum_all`
Écrire `int sum_all(int n, ...)` qui retourne la somme de `n` entiers passés en arguments.

### Exercice 2 : `print_numbers`
Écrire `void print_numbers(const char *sep, int n, ...)` qui affiche `n` entiers séparés par `sep`.

### Exercice 3 : `print_all`
Écrire `void print_all(const char *format, ...)` qui gère les types `c`, `i`, `f`, `s` (char, int, float, string).

### Exercice 4 : Mini `_printf`
Implémenter un `_printf` qui gère `%c`, `%s`, `%d`, `%i`, `%%`. Retourner le nombre de caractères imprimés. Utiliser `write` au lieu de `printf`.

### Exercice 5 : `concat_strings`
Écrire `char *concat_strings(int n, ...)` qui concatène `n` chaînes dans une nouvelle allocation (utiliser `va_copy` pour deux parcours : taille puis copie).
