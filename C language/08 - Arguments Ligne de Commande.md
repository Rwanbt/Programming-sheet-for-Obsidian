# 08 - Arguments de Ligne de Commande

## Définition

Les **arguments de ligne de commande** permettent de passer des informations à un programme au moment de son lancement, directement depuis le terminal.

```bash
./mon_programme hello world 42
```

Le programme reçoit ces arguments via les paramètres `argc` et `argv` de la fonction `main`.

> [!info] Analogie jeu vidéo
> Quand tu lances un jeu, tu peux ajouter des options : `game.exe --fullscreen --resolution 1920x1080`. Ces options modifient le comportement du jeu sans changer son code. En C, `argc` et `argv` sont le mécanisme qui rend cela possible.

---

## `argc` et `argv`

| Paramètre | Type           | Signification                                    |
| --------- | -------------- | ------------------------------------------------ |
| `argc`    | `int`          | **Argument Count** — nombre total d'arguments (programme inclus) |
| `argv`    | `char *argv[]` | **Argument Vector** — tableau de chaînes (les arguments eux-mêmes) |

```c
int main(int argc, char *argv[])
{
    /* argc = nombre d'arguments */
    /* argv[0] = nom du programme */
    /* argv[1] = premier argument */
    /* argv[argc - 1] = dernier argument */
    /* argv[argc] = NULL (toujours) */

    return (0);
}
```

---

## Visualisation mémoire : `./prog hello 42`

```bash
$ ./prog hello 42
```

```
  argc = 3

  argv (char **)
   │
   ▼
  ┌──────────┐     ┌───────────────┐
  │ argv[0]  │────▶│ "./prog\0"    │
  ├──────────┤     └───────────────┘
  │ argv[1]  │────▶┌───────────────┐
  ├──────────┤     │ "hello\0"     │
  │ argv[2]  │──┐  └───────────────┘
  ├──────────┤  │  ┌───────────────┐
  │ argv[3]  │  └─▶│ "42\0"        │  ← C'est une CHAÎNE, pas un int !
  │  (NULL)  │     └───────────────┘
  └──────────┘

  argv[argc] est TOUJOURS NULL (sentinelle)
```

> [!warning] Tous les arguments sont des CHAÎNES !
> `argv[2]` contient la **chaîne** `"42"`, pas l'**entier** `42`. Pour obtenir un nombre, il faut convertir avec `atoi`, `strtol`, etc.

---

## Les deux formes de `main`

```c
/* Forme 1 : sans arguments (quand on n'en a pas besoin) */
int main(void)
{
    return (0);
}

/* Forme 2 : avec arguments de ligne de commande */
int main(int argc, char *argv[])
{
    return (0);
}
```

> [!info] `char **argv` == `char *argv[]`
> Les deux écritures sont **strictement équivalentes** pour un paramètre de fonction :
> ```c
> int main(int argc, char **argv)    /* Écriture pointeur */
> int main(int argc, char *argv[])   /* Écriture tableau */
> /* Les deux sont identiques */
> ```
> `argv` est dans les deux cas un **pointeur vers un pointeur vers char**, c'est-à-dire un pointeur vers le premier élément d'un tableau de chaînes.

---

## Accéder aux arguments

### Méthode 1 : par index

```c
#include <stdio.h>

int main(int argc, char *argv[])
{
    int i;

    printf("Nombre d'arguments : %d\n", argc);

    for (i = 0; i < argc; i++)
        printf("argv[%d] = \"%s\"\n", i, argv[i]);

    return (0);
}
```

```bash
$ ./prog hello world 42
Nombre d'arguments : 4
argv[0] = "./prog"
argv[1] = "hello"
argv[2] = "world"
argv[3] = "42"
```

### Méthode 2 : par pointeur avec sentinelle NULL

```c
#include <stdio.h>

int main(int argc, char *argv[])
{
    (void)argc;  /* Éviter le warning "unused parameter" */

    while (*argv != NULL)
    {
        printf("%s\n", *argv);
        argv++;
    }

    return (0);
}
```

> [!tip] La sentinelle NULL
> `argv[argc]` est **toujours** `NULL`. On peut donc itérer sans connaître `argc` :
> ```c
> for (i = 0; argv[i] != NULL; i++)
>     printf("%s\n", argv[i]);
> ```

---

## Convertir les arguments en nombres

Tous les arguments sont des chaînes. Pour les convertir :

### Table de comparaison des fonctions de conversion

| Fonction   | Header       | Prototype                                         | Gestion d'erreur | Remarque                          |
| ---------- | ------------ | ------------------------------------------------- | ----------------- | --------------------------------- |
| `atoi`     | `<stdlib.h>` | `int atoi(const char *s)`                         | Aucune !          | Retourne 0 en cas d'erreur       |
| `atof`     | `<stdlib.h>` | `double atof(const char *s)`                      | Aucune !          | Retourne 0.0 en cas d'erreur     |
| `strtol`   | `<stdlib.h>` | `long strtol(const char *s, char **end, int base)` | Oui (endptr)     | Supporte les bases 2-36          |
| `strtod`   | `<stdlib.h>` | `double strtod(const char *s, char **end)`        | Oui (endptr)     | Pour les flottants                |

### `atoi` — simple mais dangereux

```c
#include <stdlib.h>

int n = atoi("42");    /* n = 42 */
int m = atoi("abc");   /* m = 0 — AUCUNE erreur signalée ! */
int o = atoi("12abc"); /* o = 12 — s'arrête au premier non-chiffre */
```

> [!warning] `atoi` ne signale pas les erreurs !
> `atoi("0")` et `atoi("abc")` retournent tous les deux `0`. Impossible de distinguer un vrai zéro d'une erreur. Préférer `strtol` pour du code robuste.

### `strtol` — robuste avec gestion d'erreur

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main(void)
{
    char *input = "42abc";
    char *endptr;
    long val;

    errno = 0;
    val = strtol(input, &endptr, 10);

    if (errno != 0)
    {
        perror("strtol");  /* Overflow ou underflow */
    }
    else if (endptr == input)
    {
        fprintf(stderr, "Erreur: pas de chiffre trouvé\n");
    }
    else if (*endptr != '\0')
    {
        printf("Converti: %ld (reste: \"%s\")\n", val, endptr);
        /* Converti: 42 (reste: "abc") */
    }
    else
    {
        printf("Converti: %ld\n", val);
    }

    return (0);
}
```

```c
/* strtol supporte différentes bases ! */
long dec = strtol("255", NULL, 10);   /* 255 (décimal) */
long hex = strtol("FF", NULL, 16);    /* 255 (hexadécimal) */
long oct = strtol("377", NULL, 8);    /* 255 (octal) */
long bin = strtol("11111111", NULL, 2);/* 255 (binaire) */
```

Voir aussi : [[02 - Bases Numeriques]] pour les conversions entre bases.

---

## Pattern de validation (style Holberton)

```c
#include <stdio.h>
#include <stdlib.h>

/**
 * main - Programme qui attend exactement 1 argument numérique
 * @argc: Nombre d'arguments
 * @argv: Tableau d'arguments
 *
 * Return: 0 si succès, 1 si erreur
 */
int main(int argc, char *argv[])
{
    int number;

    if (argc != 2)
    {
        fprintf(stderr, "Usage: %s <number>\n", argv[0]);
        return (1);
    }

    number = atoi(argv[1]);

    printf("Le nombre est : %d\n", number);

    return (0);
}
```

```bash
$ ./prog
Usage: ./prog <number>
$ echo $?
1

$ ./prog 42
Le nombre est : 42
$ echo $?
0

$ ./prog 42 extra
Usage: ./prog <number>
$ echo $?
1
```

> [!tip] Bonnes pratiques de validation
> 1. Vérifier `argc` en premier
> 2. Afficher un message d'usage sur **stderr** (pas stdout)
> 3. Utiliser `argv[0]` dans le message d'usage (le vrai nom du programme)
> 4. Retourner `1` (ou `EXIT_FAILURE`) en cas d'erreur

---

## Boss 1 : Afficher tous les arguments

```c
#include <stdio.h>

/**
 * main - Affiche tous les arguments, un par ligne
 * @argc: Nombre d'arguments
 * @argv: Tableau d'arguments
 *
 * Return: 0
 */
int main(int argc, char *argv[])
{
    int i;

    for (i = 0; i < argc; i++)
        printf("argv[%d] = %s\n", i, argv[i]);

    return (0);
}
```

---

## Boss 2 : Somme des arguments numériques

```c
#include <stdio.h>
#include <stdlib.h>

/**
 * main - Calcule la somme de tous les arguments numériques
 * @argc: Nombre d'arguments
 * @argv: Tableau d'arguments
 *
 * Return: 0 si succès, 1 si erreur
 */
int main(int argc, char *argv[])
{
    int i, sum, val;

    if (argc < 2)
    {
        fprintf(stderr, "Usage: %s <num1> [num2] [num3] ...\n", argv[0]);
        return (1);
    }

    sum = 0;
    for (i = 1; i < argc; i++)
    {
        val = atoi(argv[i]);
        sum += val;
        printf("  + %d\n", val);
    }

    printf("  = %d\n", sum);

    return (0);
}
```

```bash
$ ./sum 10 20 30
  + 10
  + 20
  + 30
  = 60
```

---

## Boss 3 : Calculatrice en ligne de commande

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/**
 * main - Calculatrice : ./calc num1 op num2
 * @argc: Nombre d'arguments
 * @argv: Tableau d'arguments
 *
 * Return: 0 si succès, 1 si erreur
 */
int main(int argc, char *argv[])
{
    int a, b, result;

    if (argc != 4)
    {
        fprintf(stderr, "Usage: %s <num1> <op> <num2>\n", argv[0]);
        fprintf(stderr, "  op: + - x / %%\n");
        return (1);
    }

    a = atoi(argv[1]);
    b = atoi(argv[3]);

    if (strcmp(argv[2], "+") == 0)
        result = a + b;
    else if (strcmp(argv[2], "-") == 0)
        result = a - b;
    else if (strcmp(argv[2], "x") == 0)
        result = a * b;
    else if (strcmp(argv[2], "/") == 0)
    {
        if (b == 0)
        {
            fprintf(stderr, "Erreur: division par zéro\n");
            return (1);
        }
        result = a / b;
    }
    else if (strcmp(argv[2], "%") == 0)
    {
        if (b == 0)
        {
            fprintf(stderr, "Erreur: modulo par zéro\n");
            return (1);
        }
        result = a % b;
    }
    else
    {
        fprintf(stderr, "Erreur: opérateur inconnu '%s'\n", argv[2]);
        return (1);
    }

    printf("%d\n", result);

    return (0);
}
```

```bash
$ ./calc 10 + 5
15
$ ./calc 10 x 3
30
$ ./calc 10 / 0
Erreur: division par zéro
$ ./calc 10 / 3
3
```

> [!warning] Attention au `*` dans le shell !
> Le caractère `*` est un **globbing** dans le shell — il est remplacé par la liste des fichiers du répertoire ! Utiliser `x` pour la multiplication ou échapper avec `\*` ou `'*'` :
> ```bash
> ./calc 5 '*' 3    # Fonctionne (quotes)
> ./calc 5 \* 3     # Fonctionne (échappement)
> ./calc 5 * 3      # NE FONCTIONNE PAS (le shell remplace *)
> ```

---

## Caractères spéciaux du shell

| Caractère | Effet dans le shell              | Solution                    |
| --------- | -------------------------------- | --------------------------- |
| `*`       | Globbing (expansion de fichiers) | `'*'` ou `\*`              |
| `?`       | Joker (un caractère)             | `'?'` ou `\?`              |
| `$`       | Variable d'environnement         | `'$VAR'` ou `\$VAR`        |
| `"`       | Interpolation                    | `\"` ou `'texte'`          |
| `\`       | Échappement                      | `\\`                        |
| `;`       | Séparateur de commandes          | `';'` ou `\;`              |
| `>`       | Redirection                      | `'>'` ou `\>`              |
| `|`       | Pipe                             | `'\|'` ou `\\|`            |

```bash
# Les guillemets simples protègent TOUT
./prog 'hello world'     # argv[1] = "hello world" (1 seul argument)
./prog hello world        # argv[1] = "hello", argv[2] = "world" (2 arguments)
```

---

## Erreurs courantes et corrections

### 1. Oublier de vérifier argc

```c
/* FAUX — crash si aucun argument ! */
int main(int argc, char *argv[])
{
    int n = atoi(argv[1]);  /* argv[1] peut ne pas exister ! */
    printf("%d\n", n);
    return (0);
}

/* CORRECT */
int main(int argc, char *argv[])
{
    if (argc < 2)
    {
        fprintf(stderr, "Usage: %s <number>\n", argv[0]);
        return (1);
    }
    printf("%d\n", atoi(argv[1]));
    return (0);
}
```

### 2. Traiter argv comme des nombres directement

```c
/* FAUX — argv[1] est un char*, pas un int ! */
int n = argv[1];            /* Erreur de type */
int n = argv[1] + argv[2];  /* Addition de pointeurs, pas de nombres ! */

/* CORRECT */
int n = atoi(argv[1]);
int m = atoi(argv[2]);
int sum = n + m;
```

### 3. Écrire les erreurs sur stdout

```c
/* FAUX — les erreurs doivent aller sur stderr */
printf("Erreur: pas assez d'arguments\n");

/* CORRECT */
fprintf(stderr, "Erreur: pas assez d'arguments\n");
```

### 4. Oublier que argv[0] est le nom du programme

```c
/* argv[0] n'est PAS le premier argument de l'utilisateur ! */
/* Pour ./prog hello :
   argv[0] = "./prog"
   argv[1] = "hello"   ← C'est ici que commence l'input utilisateur
*/
```

---

## Points clés à retenir

> [!tip] Mémo rapide
> - `argv[0]` est **toujours** présent (le nom du programme) → `argc >= 1`
> - `argv[argc]` est **toujours** `NULL` (sentinelle)
> - **Tous** les arguments sont des **chaînes** (`char *`)
> - Les erreurs vont sur **stderr** (`fprintf(stderr, ...)`)
> - Retourner **1** (ou `EXIT_FAILURE`) en cas d'erreur
> - `atoi` ne détecte pas les erreurs → utiliser `strtol` pour du code robuste
> - Attention aux **caractères spéciaux** du shell (`*`, `?`, `$`...)

---

## Carte mentale complète

```
                        ┌──────────────────────┐
                        │   LIGNE DE COMMANDE  │
                        └──────────┬───────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
        ┌─────┴─────┐      ┌──────┴──────┐     ┌──────┴──────┐
        │   argc     │      │   argv       │     │ Conversion  │
        │ (int)      │      │ (char **)    │     │             │
        └─────┬─────┘      └──────┬──────┘     └──────┬──────┘
              │                    │                    │
        Nombre total         Tableau de          atoi (simple)
        d'arguments          chaînes             strtol (robuste)
        (prog inclus)                            atof / strtod
              │                    │
        argc >= 1            argv[0] = prog
                             argv[argc] = NULL
                             argv[i] = chaîne

                    ┌──────────────────────────┐
                    │      VALIDATION           │
                    ├──────────────────────────┤
                    │ 1. Vérifier argc         │
                    │ 2. Message sur stderr    │
                    │ 3. argv[0] dans l'usage  │
                    │ 4. return 1 si erreur    │
                    └──────────────────────────┘
```

---

## Exemple complet : programme robuste

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

/**
 * is_number - Vérifie si une chaîne est un nombre entier valide
 * @s: La chaîne à vérifier
 *
 * Return: 1 si nombre valide, 0 sinon
 */
int is_number(char *s)
{
    int i = 0;

    if (s == NULL || s[0] == '\0')
        return (0);

    if (s[0] == '-' || s[0] == '+')
        i++;

    if (s[i] == '\0')
        return (0);

    while (s[i] != '\0')
    {
        if (!isdigit(s[i]))
            return (0);
        i++;
    }

    return (1);
}

/**
 * main - Programme qui multiplie deux nombres
 * @argc: Nombre d'arguments
 * @argv: Tableau d'arguments
 *
 * Return: 0 si succès, 1 si erreur
 */
int main(int argc, char *argv[])
{
    long a, b, result;

    if (argc != 3)
    {
        fprintf(stderr, "Usage: %s <num1> <num2>\n", argv[0]);
        return (1);
    }

    if (!is_number(argv[1]) || !is_number(argv[2]))
    {
        fprintf(stderr, "Erreur: les arguments doivent être des nombres\n");
        return (1);
    }

    a = strtol(argv[1], NULL, 10);
    b = strtol(argv[2], NULL, 10);
    result = a * b;

    printf("%ld\n", result);

    return (0);
}
```

```bash
$ ./mul 6 7
42
$ ./mul abc 5
Erreur: les arguments doivent être des nombres
$ ./mul
Usage: ./mul <num1> <num2>
```

---

## Liens

- [[01 - Introduction au C et Compilation]] — La fonction main et ses deux formes
- [[04 - Pointeurs et Memoire]] — char ** et la mémoire
- [[03 - Structures et Typedef]] — Structs pour les options de ligne de commande

---

## Exercices pratiques

### Exercice 1 : Echo
Écrire un programme qui reproduit le comportement de la commande `echo` : affiche tous les arguments séparés par des espaces, suivi d'un newline.

### Exercice 2 : Compteur de caractères
Écrire un programme qui affiche le nombre total de caractères dans tous les arguments (argv[0] exclu).

### Exercice 3 : Majuscules
Écrire un programme qui affiche chaque argument converti en majuscules. (`toupper` de `<ctype.h>`)

### Exercice 4 : Min/Max
Écrire un programme qui prend N nombres en arguments et affiche le minimum et le maximum. Valider que tous les arguments sont des nombres.

### Exercice 5 : Tri des arguments
Écrire un programme qui affiche les arguments triés par ordre alphabétique (utiliser `strcmp` et un tri simple).

### Exercice 6 : Calculatrice avancée
Reprendre le Boss 3 et ajouter :
- Validation avec `is_number`
- Support de `strtol` au lieu de `atoi`
- Opérations supplémentaires : puissance (`^`), utiliser [[05 - Pointeurs de Fonctions]] avec un tableau de dispatch
