# 01 - Introduction au C et Compilation

## Qu'est-ce que le langage C ?

Le **C** est un langage de programmation **compilé**, **impératif** et de **bas niveau** créé en **1972** par **Dennis Ritchie** aux laboratoires Bell (AT&T). Il a été conçu pour réécrire le système d'exploitation **UNIX**, auparavant écrit en assembleur.

> [!info] Pourquoi apprendre le C ?
> - C'est le **fondement** de presque tous les langages modernes (C++, Java, Python, Go, Rust...)
> - Il donne un **contrôle total** sur la mémoire et le matériel
> - Les **systèmes d'exploitation** (Linux, Windows, macOS) sont écrits en C
> - Il est **rapide** : pas d'interpréteur, le code est directement traduit en langage machine
> - Comprendre le C, c'est comprendre **comment fonctionne un ordinateur**

### Chronologie rapide

| Année | Événement                                  |
| ----- | ------------------------------------------ |
| 1972  | Création du C par Dennis Ritchie           |
| 1978  | Publication de "The C Programming Language" (K&R) |
| 1989  | Norme ANSI C (C89/C90)                    |
| 1999  | Norme C99 (variables inline, `//` commentaires) |
| 2011  | Norme C11 (threads, assertions statiques)  |
| 2023  | Norme C23 (dernière en date)               |

---

## Le processus de compilation

Le C est un langage **compilé** : le code source (`.c`) est transformé en un exécutable binaire **avant** l'exécution. Ce processus se décompose en **4 étapes** :

```
  source.c          Préprocesseur         Compilateur        Assembleur          Éditeur de liens
 ┌──────────┐      ┌──────────────┐      ┌───────────┐     ┌───────────┐      ┌─────────────────┐
 │  Code C  │─────▶│  #include    │─────▶│  Code     │────▶│  Code     │─────▶│  Exécutable     │
 │  source  │      │  #define     │      │  assembleur│     │  objet    │      │  final          │
 │  (.c)    │      │  expansion   │      │  (.s)     │     │  (.o)     │      │  (a.out / prog) │
 └──────────┘      └──────────────┘      └───────────┘     └───────────┘      └─────────────────┘
                      Étape 1               Étape 2           Étape 3             Étape 4
                    gcc -E source.c       gcc -S source.c   gcc -c source.c     gcc source.c -o prog
```

### Étape 1 : Le préprocesseur (`gcc -E`)

Le préprocesseur traite toutes les directives commençant par `#` :
- `#include` : copie le contenu d'un fichier header
- `#define` : remplace les macros par leur valeur
- `#ifdef / #ifndef / #endif` : compilation conditionnelle

```c
/* Avant préprocesseur */
#include <stdio.h>
#define PI 3.14159

int main(void)
{
    printf("Pi = %f\n", PI);
    return (0);
}
```

Après le préprocesseur, `#include <stdio.h>` est remplacé par des milliers de lignes (le contenu réel du fichier header) et `PI` est remplacé par `3.14159`.

### Étape 2 : La compilation (`gcc -S`)

Le compilateur transforme le code C préprocessé en **code assembleur** (`.s`), spécifique à l'architecture du processeur (x86, ARM...).

### Étape 3 : L'assemblage (`gcc -c`)

L'assembleur convertit le code assembleur en **code objet** (`.o`) — du binaire, mais pas encore exécutable.

### Étape 4 : L'édition de liens (`gcc`)

L'éditeur de liens (linker) combine tous les fichiers objets et les bibliothèques (comme `libc`) pour produire l'**exécutable final**.

---

## GCC : les drapeaux essentiels

```bash
gcc -Wall -Werror -Wextra -pedantic -std=gnu89 source.c -o prog
```

| Drapeau      | Rôle                                                   |
| ------------ | ------------------------------------------------------ |
| `-Wall`      | Active la majorité des avertissements                  |
| `-Werror`    | Transforme tous les warnings en **erreurs**            |
| `-Wextra`    | Active des avertissements supplémentaires              |
| `-pedantic`  | Force le respect strict de la norme choisie            |
| `-std=gnu89` | Utilise la norme C89 avec extensions GNU               |
| `-o prog`    | Nomme l'exécutable de sortie `prog`                    |
| `-g`         | Ajoute les infos de débogage (pour `gdb` / `valgrind`) |
| `-E`         | Arrête après le préprocesseur                          |
| `-S`         | Arrête après la compilation (produit `.s`)             |
| `-c`         | Arrête après l'assemblage (produit `.o`)               |

> [!warning] Toujours compiler avec les warnings !
> Ne **jamais** compiler sans `-Wall -Werror -Wextra`. Un code qui compile sans warning est un code plus sûr. Prendre l'habitude de corriger **tous** les warnings dès le début.

---

## Le style Betty

Le **style Betty** est la convention de codage utilisée dans les projets Holberton/ALX. Vérification avec `betty-style.pl` et `betty-doc.pl`.

Règles principales :
- **Indentation** : tabulations (pas d'espaces), 1 tab = 8 espaces visuels
- **Longueur de ligne** : max 80 colonnes
- **Fonctions** : max 40 lignes, max 5 fonctions par fichier
- **Variables** : déclarées en **début de fonction** (style C89)
- **Accolades** : style Kernighan & Ritchie (ouvrante sur la même ligne, sauf pour les fonctions)
- **Espaces** : après les mots-clés (`if (`, `while (`), pas après les noms de fonctions (`printf(`)

```c
/**
 * main - Point d'entrée du programme
 *
 * Return: Toujours 0 (succès)
 */
int main(void)
{
	int i;

	for (i = 0; i < 10; i++)
	{
		if (i % 2 == 0)
			printf("%d est pair\n", i);
		else
			printf("%d est impair\n", i);
	}

	return (0);
}
```

---

## Les fichiers headers et les include guards

Un **fichier header** (`.h`) contient les **déclarations** (prototypes de fonctions, macros, typedefs) partagées entre plusieurs fichiers `.c`.

```c
/* main.h */
#ifndef MAIN_H
#define MAIN_H

#include <stdio.h>
#include <stdlib.h>

/* Prototypes */
int _putchar(char c);
void print_alphabet(void);
int _strlen(char *s);

#endif /* MAIN_H */
```

> [!tip] Les include guards
> `#ifndef MAIN_H` / `#define MAIN_H` / `#endif` empêchent l'**inclusion multiple** du même header. Sans eux, le compilateur verrait des déclarations en double et produirait des erreurs.
>
> Le format conventionnel est : `NOM_DU_FICHIER_H` en majuscules.

Pour utiliser ce header dans un fichier source :

```c
#include "main.h"  /* guillemets = cherche dans le répertoire courant */
#include <stdio.h> /* chevrons = cherche dans les répertoires système */
```

---

## La fonction `main`

Tout programme C commence par la fonction `main`. C'est le **point d'entrée**.

```c
/* Forme 1 : sans arguments */
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

> [!info] La valeur de retour
> - `return (0)` : le programme s'est terminé avec **succès**
> - `return (1)` ou autre valeur non nulle : **erreur**
> - Cette valeur est récupérable dans le shell avec `echo $?`

Voir aussi : [[08 - Arguments Ligne de Commande]]

---

## Les types de données fondamentaux

| Type     | Taille (typique) | Plage de valeurs                  | Format `printf` |
| -------- | ----------------- | --------------------------------- | ---------------- |
| `char`   | 1 octet           | -128 à 127 (ou 0 à 255)          | `%c` ou `%d`    |
| `short`  | 2 octets          | -32 768 à 32 767                  | `%hd`           |
| `int`    | 4 octets          | -2 147 483 648 à 2 147 483 647   | `%d` ou `%i`    |
| `long`   | 4 ou 8 octets     | dépend de la plateforme           | `%ld`           |
| `float`  | 4 octets          | ~6 chiffres significatifs         | `%f`            |
| `double` | 8 octets          | ~15 chiffres significatifs        | `%lf` ou `%f`   |
| `void`   | —                 | aucune valeur (type "rien")       | —                |

Les modificateurs `unsigned` suppriment les valeurs négatives et doublent la plage positive :

```c
unsigned int age = 25;         /* 0 à 4 294 967 295 */
unsigned char octet = 255;     /* 0 à 255 */
```

> [!tip] `sizeof` pour connaître la taille
> ```c
> printf("int = %lu octets\n", sizeof(int));       /* 4 */
> printf("char = %lu octets\n", sizeof(char));      /* 1 */
> printf("double = %lu octets\n", sizeof(double));  /* 8 */
> ```

---

## `printf` et les spécificateurs de format

`printf` est la fonction d'affichage principale. Elle est déclarée dans `<stdio.h>`.

```c
#include <stdio.h>

int main(void)
{
    int age = 25;
    char initiale = 'A';
    float taille = 1.75;
    char *nom = "Alice";

    printf("Nom : %s\n", nom);
    printf("Initiale : %c\n", initiale);
    printf("Age : %d ans\n", age);
    printf("Taille : %.2f m\n", taille);
    printf("Adresse de age : %p\n", (void *)&age);

    return (0);
}
```

### Table des spécificateurs

| Spécificateur | Type attendu           | Exemple              |
| ------------- | ---------------------- | -------------------- |
| `%d` / `%i`   | `int` (décimal)        | `printf("%d", 42);`  |
| `%u`          | `unsigned int`         | `printf("%u", 42);`  |
| `%c`          | `char` (caractère)     | `printf("%c", 'A');` |
| `%s`          | `char *` (chaîne)      | `printf("%s", "Hi");`|
| `%f`          | `float` / `double`     | `printf("%f", 3.14);`|
| `%x` / `%X`   | `int` (hexadécimal)    | `printf("%x", 255);` |
| `%o`          | `int` (octal)          | `printf("%o", 8);`   |
| `%p`          | `void *` (pointeur)    | `printf("%p", ptr);` |
| `%ld`         | `long`                 | `printf("%ld", 1L);` |
| `%%`          | affiche un `%` littéral| `printf("%%");`      |

Voir aussi : [[02 - Bases Numeriques]] pour les formats `%x`, `%o` et les bases numériques.

---

## Variables et portée (scope)

### Déclaration et initialisation

```c
int a;          /* déclaration (valeur indéterminée !) */
int b = 10;     /* déclaration + initialisation */
a = 5;          /* affectation */
```

> [!warning] Variables non initialisées
> Une variable locale non initialisée contient une valeur **indéterminée** (garbage). Toujours initialiser ses variables !

### La portée (scope)

```c
int global = 100;  /* Variable globale : visible partout */

void fonction(void)
{
    int local = 10;         /* Visible uniquement dans cette fonction */

    if (local > 5)
    {
        int bloc = 20;     /* Visible uniquement dans ce bloc if */
        printf("%d\n", bloc);
    }
    /* bloc n'existe plus ici */
}
```

```
┌─────────────────────────────────────────┐
│  Portée globale (fichier)               │
│  int global = 100;                      │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  Portée fonction                │    │
│  │  int local = 10;               │    │
│  │                                 │    │
│  │  ┌─────────────────────────┐    │    │
│  │  │  Portée bloc            │    │    │
│  │  │  int bloc = 20;        │    │    │
│  │  └─────────────────────────┘    │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

### Le mot-clé `static`

```c
void compteur(void)
{
    static int n = 0;  /* Initialisée UNE seule fois, conserve sa valeur entre les appels */

    n++;
    printf("Appel numéro %d\n", n);
}
```

---

## Les opérateurs

### Opérateurs arithmétiques

| Opérateur | Description         | Exemple      |
| --------- | ------------------- | ------------ |
| `+`       | Addition            | `5 + 3 = 8`  |
| `-`       | Soustraction        | `5 - 3 = 2`  |
| `*`       | Multiplication      | `5 * 3 = 15` |
| `/`       | Division            | `7 / 2 = 3`  |
| `%`       | Modulo (reste)      | `7 % 2 = 1`  |

> [!warning] Division entière
> `7 / 2` donne `3` et non `3.5` ! En C, la division entre deux entiers est une **division entière**. Pour un résultat décimal : `7.0 / 2` ou `(float)7 / 2`.

### Opérateurs de comparaison

| Opérateur | Description         |
| --------- | ------------------- |
| `==`      | Égal à              |
| `!=`      | Différent de        |
| `<`       | Inférieur à         |
| `>`       | Supérieur à         |
| `<=`      | Inférieur ou égal   |
| `>=`      | Supérieur ou égal   |

### Opérateurs logiques

| Opérateur | Description | Exemple                      |
| --------- | ----------- | ---------------------------- |
| `&&`      | ET logique  | `(a > 0 && b > 0)`          |
| `\|\|`    | OU logique  | `(a > 0 \|\| b > 0)`        |
| `!`       | NON logique | `!(a > 0)` (inverse le bool) |

### Opérateurs d'incrémentation

```c
int a = 5;
a++;    /* post-incrémentation : utilise a, puis ajoute 1 */
++a;    /* pré-incrémentation : ajoute 1, puis utilise a */
a--;    /* post-décrémentation */
--a;    /* pré-décrémentation */
a += 3; /* équivalent à a = a + 3 */
a -= 2; /* équivalent à a = a - 2 */
a *= 4; /* équivalent à a = a * 4 */
```

### Opérateurs bit à bit

Voir : [[02 - Bases Numeriques]] pour une couverture complète.

| Opérateur | Nom      | Exemple      |
| --------- | -------- | ------------ |
| `&`       | AND      | `5 & 3 = 1`  |
| `\|`      | OR       | `5 \| 3 = 7` |
| `^`       | XOR      | `5 ^ 3 = 6`  |
| `~`       | NOT      | `~5`         |
| `<<`      | Décalage gauche | `1 << 3 = 8` |
| `>>`      | Décalage droite | `8 >> 2 = 2` |

---

## Structures de contrôle

### `if` / `else if` / `else`

```c
int note = 15;

if (note >= 16)
    printf("Très bien\n");
else if (note >= 14)
    printf("Bien\n");
else if (note >= 10)
    printf("Passable\n");
else
    printf("Insuffisant\n");
```

### L'opérateur ternaire

```c
int max = (a > b) ? a : b;
/* Équivalent à :
   if (a > b)
       max = a;
   else
       max = b;
*/
```

### La boucle `while`

```c
int i = 0;

while (i < 5)
{
    printf("%d\n", i);
    i++;
}
```

### La boucle `for`

```c
int i;

for (i = 0; i < 5; i++)
{
    printf("%d\n", i);
}
```

> [!info] Équivalence `for` / `while`
> ```c
> /* Ces deux boucles sont strictement identiques */
> for (init; condition; incrément)    |    init;
> {                                   |    while (condition)
>     corps;                          |    {
> }                                   |        corps;
>                                     |        incrément;
>                                     |    }
> ```

### La boucle `do...while`

```c
int n;

do {
    printf("Entrez un nombre positif : ");
    scanf("%d", &n);
} while (n <= 0);  /* Répète tant que n n'est pas positif */
```

> [!tip] Quand utiliser `do...while` ?
> Quand on veut exécuter le corps **au moins une fois** avant de tester la condition.

### `switch`

```c
int choix = 2;

switch (choix)
{
    case 1:
        printf("Option 1\n");
        break;
    case 2:
        printf("Option 2\n");
        break;
    case 3:
        printf("Option 3\n");
        break;
    default:
        printf("Option invalide\n");
        break;
}
```

> [!warning] N'oubliez pas le `break` !
> Sans `break`, l'exécution "tombe" dans le case suivant (fall-through). C'est parfois voulu, mais c'est souvent un bug.

### `break` et `continue`

```c
/* break : sort de la boucle immédiatement */
for (i = 0; i < 100; i++)
{
    if (i == 10)
        break;  /* Sort quand i vaut 10 */
}

/* continue : passe directement à l'itération suivante */
for (i = 0; i < 10; i++)
{
    if (i % 2 == 0)
        continue;  /* Saute les nombres pairs */
    printf("%d\n", i);  /* Affiche : 1, 3, 5, 7, 9 */
}
```

---

## `putchar` : afficher un caractere

La fonction `putchar` (pour *put character*) est declaree dans `<stdio.h>`. Elle affiche **un seul caractere** sur la sortie standard (le terminal).

### Syntaxe

```c
int putchar(int caractere);
```

- **Argument** : un `int` (pas un `char`). En C, les caracteres sont geres par leur code ASCII, et cela permet aussi de manipuler la constante `EOF`.
- **Retour** : le caractere ecrit en cas de succes, `EOF` en cas d'erreur.

### Pourquoi `putchar` au lieu de `printf` ?

- **Performance** : `printf` doit analyser une chaine de formatage (chercher les `%`), ce qui est lourd. `putchar` envoie directement le caractere. C'est beaucoup plus rapide dans une boucle intensive.
- **Simplicite** : pour afficher un saut de ligne, `putchar('\n')` est plus elegant que `printf("\n")`.
- **Apprentissage** : dans les ecoles comme Holberton/42, on commence souvent par `putchar` pour comprendre comment les donnees transitent vers la sortie standard.

### Exemple : afficher l'alphabet

```c
#include <stdio.h>

int main(void)
{
	char c = 'a';

	while (c <= 'z')
	{
		putchar(c);
		c++;
	}
	putchar('\n');
	return (0);
}
```

Puisqu'un `char` est techniquement un petit nombre (code ASCII), on peut faire de l'arithmetique dessus : `c++` passe de `'a'` (97) a `'b'` (98), et ainsi de suite jusqu'a `'z'` (122).

### Le buffer de `putchar`

`putchar` utilise un **buffer** (zone memoire tampon). Le caractere n'est pas forcement envoye a l'ecran immediatement. L'affichage reel se declenche quand :

1. Le buffer est **plein**
2. Un caractere de saut de ligne `\n` est rencontre
3. Le programme se **termine**
4. On force l'affichage avec `fflush(stdout)`

> [!tip] Sous le capot
> `putchar` finit par appeler l'appel systeme `write()` pour envoyer les donnees au terminal. Le buffer existe pour eviter de faire un appel systeme couteux pour chaque caractere individuel.

### Construire `putstr` a partir de `putchar`

En utilisant uniquement `putchar`, on peut recreer une fonction qui affiche une chaine entiere. C'est un excellent exercice pour manipuler les chaines :

```c
void my_putstr(char *str)
{
	int i;

	i = 0;
	while (str[i] != '\0')
	{
		putchar(str[i]);
		i++;
	}
}
```

---

## Difference entre `puts`, `printf` et `putchar`

| Fonction   | Header      | Ce qu'elle affiche                  | Ajoute `\n` ? | Performance |
| ---------- | ----------- | ----------------------------------- | -------------- | ----------- |
| `putchar`  | `<stdio.h>` | Un seul caractere                   | Non            | Rapide      |
| `puts`     | `<stdio.h>` | Une chaine de caracteres            | **Oui** (automatique) | Moyenne |
| `printf`   | `<stdio.h>` | Texte formate avec des specificateurs (`%d`, `%s`...) | Non | Plus lent (parsing du format) |

```c
putchar('A');                  /* Affiche : A */
puts("Hello");                 /* Affiche : Hello\n (saut de ligne auto) */
printf("J'ai %d ans\n", 25);  /* Affiche : J'ai 25 ans\n */
```

> [!info] Quand utiliser laquelle ?
> - **`putchar`** : afficher un seul caractere (boucles sur l'alphabet, exercices bas niveau)
> - **`puts`** : afficher une chaine simple sans formatage (ajoute `\n` automatiquement)
> - **`printf`** : afficher du texte avec des variables inserees (formatage complet)

---

## Difference entre `%d` et `%i`

### Dans `printf` (affichage)

`%d` et `%i` sont **strictement identiques**. Les deux affichent un entier signe en base 10.

- `%d` vient de **Decimal** (base 10)
- `%i` vient de **Integer**

### Dans `scanf` (lecture)

C'est la que la difference est importante :

| Specificateur | Comportement avec `scanf`                              |
| ------------- | ------------------------------------------------------- |
| `%d`          | Force l'interpretation en **base 10 uniquement**. Si l'utilisateur tape `012`, le programme lit **12**. |
| `%i`          | Detecte la base **automatiquement** selon le prefixe : `0x` = hexadecimal, `0` = octal, sinon decimal. `012` est lu comme **10** (octal). |

```c
int n;
scanf("%d", &n);   /* "012" → n = 12 (decimal) */
scanf("%i", &n);   /* "012" → n = 10 (octal !)  */
scanf("%i", &n);   /* "0x10" → n = 16 (hexadecimal) */
```

> [!warning] Piege courant
> "Decimal" en C ne signifie **pas** "nombre a virgule". Cela signifie "base 10" (par opposition a binaire ou hexadecimal). Les nombres a virgule utilisent `%f` (float) ou `%lf` (double).

> [!tip] Conseil
> Utilise toujours `%d` par defaut. C'est la convention la plus repandue en C, sauf si tu as specifiquement besoin de la detection automatique de base dans `scanf`.

---

## Difference entre `""`, `0` et `NULL`

Ces trois valeurs semblent "vides" mais sont fondamentalement differentes :

| Valeur | Type             | Signification           | Analogie                  |
| ------ | ---------------- | ----------------------- | ------------------------- |
| `""`   | `char *` (chaine)| Chaine vide (existe mais sans contenu) | Une boite vide |
| `0`    | `int`            | La quantite numerique zero | Un compteur a zero       |
| `NULL` | Pointeur         | Absence totale de valeur/reference | La boite n'existe pas |

### En C specifiquement

```c
char *str = "";     /* Pointeur vers un '\0' unique. strlen(str) == 0 */
int n = 0;          /* Valeur numerique zero */
char *ptr = NULL;   /* Pointeur invalide, ne pointe vers rien */
```

- `""` est une chaine valide de longueur 0. Le pointeur pointe vers un octet `'\0'`.
- `0` est une valeur entiere. En contexte booleen, `0` est `faux`.
- `NULL` est defini comme `((void *)0)`. Dereference `NULL` provoque un **segfault**.

> [!warning] Le piege a eviter
> `NULL` et `0` peuvent etre confondus dans certains contextes (pointeurs), mais `""` n'est **jamais** egal a `NULL` : `""` est un pointeur valide vers une chaine vide, tandis que `NULL` est un pointeur invalide.

---

## `snprintf` : l'alternative securisee a `sprintf`

### Le probleme de `sprintf`

`sprintf` formate une chaine et la stocke dans un buffer, mais **ne verifie pas la taille** de la destination. Si le texte formate depasse la capacite du buffer, il y a **buffer overflow** :

```c
char buf[10];
sprintf(buf, "Bonjour %s !", "tout le monde");  /* OVERFLOW ! */
```

### La solution : `snprintf`

```c
int snprintf(char *str, size_t size, const char *format, ...);
```

- `str` : le buffer de destination
- `size` : la taille maximale du buffer (incluant le `\0` final)
- **Retour** : le nombre de caracteres qui *auraient du* etre ecrits (permet de detecter la troncature)

### Les 3 regles d'or de `snprintf`

1. Elle n'ecrira **jamais** plus de `size - 1` caracteres
2. Elle ajoute **toujours** un `\0` a la fin (sauf si `size == 0`)
3. Sa valeur de retour indique si le texte a ete **tronque** (`retour >= size`)

### Exemple

```c
char buffer[10];
int n = snprintf(buffer, sizeof(buffer), "Hello tout le monde !");

printf("Contenu : %s\n", buffer);     /* Affiche "Hello tou" (9 chars + \0) */
printf("Taille voulue : %d\n", n);    /* Affiche 21 */

if (n >= (int)sizeof(buffer))
	printf("Attention : texte tronque !\n");
```

### Comparaison

| Fonction    | Risque de Buffer Overflow | Limitation de taille | Standard |
| ----------- | ------------------------- | -------------------- | -------- |
| `printf`    | Non (affiche en console)  | N/A                  | C89      |
| `sprintf`   | **Eleve**                 | Non                  | C89      |
| `snprintf`  | Nul (si bien utilisee)    | Oui                  | C99      |

> [!warning] Bonne pratique
> Aujourd'hui, la plupart des grandes entreprises (Microsoft, Google, Apple) **interdisent** l'usage de `sprintf` dans leurs regles de codage. Utilise toujours `snprintf` pour tout nouveau code.

---

## La bibliothèque standard

Le C fournit une **bibliothèque standard** (libc) avec des fonctions essentielles :

| Header        | Fonctions principales                              |
| ------------- | -------------------------------------------------- |
| `<stdio.h>`   | `printf`, `scanf`, `fopen`, `fclose`, `fgets`     |
| `<stdlib.h>`  | `malloc`, `free`, `atoi`, `exit`, `rand`          |
| `<string.h>`  | `strlen`, `strcpy`, `strcmp`, `strcat`, `memcpy`  |
| `<math.h>`    | `sqrt`, `pow`, `sin`, `cos`, `abs`                |
| `<ctype.h>`   | `isalpha`, `isdigit`, `toupper`, `tolower`        |
| `<unistd.h>`  | `write`, `read`, `close`, `fork`, `execve`        |
| `<stdarg.h>`  | `va_list`, `va_start`, `va_arg`, `va_end`         |

> [!tip] Compiler avec `-lm`
> Pour utiliser les fonctions mathématiques (`<math.h>`), ajouter `-lm` à la commande de compilation :
> ```bash
> gcc -Wall -Werror -Wextra main.c -lm -o prog
> ```

---

## Premier programme complet

```c
#include <stdio.h>

/**
 * main - Affiche un message de bienvenue
 *
 * Return: Toujours 0 (succès)
 */
int main(void)
{
	char nom[] = "Alice";
	int age = 22;
	float moyenne = 15.75;

	printf("=== Bienvenue en C ! ===\n");
	printf("Nom    : %s\n", nom);
	printf("Age    : %d ans\n", age);
	printf("Moyenne: %.2f / 20\n", moyenne);

	if (moyenne >= 10.0)
		printf("Résultat : ADMIS(E)\n");
	else
		printf("Résultat : AJOURNÉ(E)\n");

	return (0);
}
```

Compilation et exécution :
```bash
$ gcc -Wall -Werror -Wextra -pedantic -std=gnu89 main.c -o bienvenue
$ ./bienvenue
=== Bienvenue en C ! ===
Nom    : Alice
Age    : 22 ans
Moyenne: 15.75 / 20
Résultat : ADMIS(E)
```

---

## Liens vers les autres notes

- [[02 - Bases Numeriques]] — Binaire, hexadécimal, opérateurs bit à bit
- [[03 - Structures et Typedef]] — Structures de données, typedef
- [[04 - Pointeurs et Memoire]] — Pointeurs, allocation dynamique, mémoire virtuelle
- [[05 - Pointeurs de Fonctions]] — Callbacks et tableaux de pointeurs de fonctions
- [[06 - Fonctions Variadiques]] — Fonctions à arguments variables
- [[07 - Recursion]] — Fonctions récursives
- [[08 - Arguments Ligne de Commande]] — argc, argv et traitement des arguments

---

## Exercices pratiques

### Exercice 1 : Hello World
Écrire un programme qui affiche `"Hello, World!"` suivi d'un retour à la ligne.

### Exercice 2 : Calculatrice simple
Écrire un programme qui déclare deux variables `int a = 15` et `int b = 4`, puis affiche :
- La somme, la différence, le produit
- La division entière et le reste (modulo)
- La division flottante (cast en `float`)

### Exercice 3 : Table de multiplication
Écrire un programme qui affiche la table de multiplication d'un nombre donné (1 à 10) avec une boucle `for`.

### Exercice 4 : FizzBuzz
Afficher les nombres de 1 à 100. Pour les multiples de 3, afficher "Fizz". Pour les multiples de 5, afficher "Buzz". Pour les multiples de 3 ET 5, afficher "FizzBuzz".

### Exercice 5 : Compter les voyelles
Écrire une fonction `int count_vowels(char *str)` qui retourne le nombre de voyelles dans une chaîne. Utiliser une boucle `while` et un `switch`.

### Exercice 6 : Compilation pas à pas
Prendre un programme simple et exécuter chaque étape de compilation séparément avec `gcc -E`, `gcc -S`, `gcc -c`, puis linker. Observer les fichiers produits à chaque étape.
