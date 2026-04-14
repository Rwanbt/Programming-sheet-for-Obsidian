# 07 - Récursion

## Définition

Une **fonction récursive** est une fonction qui **s'appelle elle-même**. Chaque appel récursif résout un sous-problème plus petit, jusqu'à atteindre un **cas de base** qui arrête la récursion.

Toute fonction récursive a deux composants essentiels :
1. **Cas de base** (base case) : la condition d'arrêt, sans laquelle la récursion serait infinie
2. **Cas récursif** (recursive case) : l'appel à soi-même avec un problème **réduit**

---

## Structure squelette

```c
type fonction(paramètres)
{
    /* 1. Cas de base : quand s'arrêter */
    if (condition_d_arret)
        return (valeur_de_base);

    /* 2. Cas récursif : s'appeler avec un problème plus petit */
    return (fonction(paramètres_réduits));
}
```

```
  ┌───────────────────────────────────────┐
  │          STRUCTURE RÉCURSIVE          │
  │                                       │
  │  ┌─────────────────────────────────┐  │
  │  │  CAS DE BASE                   │  │
  │  │  → Condition d'arrêt           │  │
  │  │  → Retourne une valeur simple  │  │
  │  └─────────────────────────────────┘  │
  │                 │                     │
  │                 ▼                     │
  │  ┌─────────────────────────────────┐  │
  │  │  CAS RÉCURSIF                  │  │
  │  │  → Réduit le problème          │  │
  │  │  → S'appelle soi-même          │  │
  │  └─────────────────────────────────┘  │
  └───────────────────────────────────────┘
```

---

## Boss 1 : Factorielle

La factorielle de n : `n! = n × (n-1) × (n-2) × ... × 1`

```
  5! = 5 × 4 × 3 × 2 × 1 = 120
  Cas de base : 0! = 1 et 1! = 1
  Cas récursif : n! = n × (n-1)!
```

```c
#include <stdio.h>

/**
 * factorial - Calcule la factorielle d'un nombre
 * @n: Le nombre
 *
 * Return: n!, ou -1 si n < 0
 */
int factorial(int n)
{
    if (n < 0)
        return (-1);
    if (n == 0 || n == 1)
        return (1);

    return (n * factorial(n - 1));
}

int main(void)
{
    printf("5! = %d\n", factorial(5));   /* 120 */
    printf("0! = %d\n", factorial(0));   /* 1 */
    printf("1! = %d\n", factorial(1));   /* 1 */
    printf("10! = %d\n", factorial(10)); /* 3628800 */

    return (0);
}
```

---

## Boss 2 : Puissance

```
  2^5 = 2 × 2 × 2 × 2 × 2 = 32
  Cas de base : x^0 = 1
  Cas récursif : x^n = x × x^(n-1)
```

```c
#include <stdio.h>

/**
 * _pow_recursion - Calcule x à la puissance n
 * @x: La base
 * @n: L'exposant
 *
 * Return: x^n, ou -1 si n < 0
 */
int _pow_recursion(int x, int n)
{
    if (n < 0)
        return (-1);
    if (n == 0)
        return (1);

    return (x * _pow_recursion(x, n - 1));
}

int main(void)
{
    printf("2^5 = %d\n", _pow_recursion(2, 5));   /* 32 */
    printf("3^4 = %d\n", _pow_recursion(3, 4));   /* 81 */
    printf("7^0 = %d\n", _pow_recursion(7, 0));   /* 1 */
    printf("0^5 = %d\n", _pow_recursion(0, 5));   /* 0 */

    return (0);
}
```

---

## Boss 3 : Fibonacci

```
  Fibonacci : 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...
  Chaque nombre = somme des 2 précédents
  Cas de base : fib(0) = 0, fib(1) = 1
  Cas récursif : fib(n) = fib(n-1) + fib(n-2)
```

```c
#include <stdio.h>

/**
 * fibonacci - Calcule le nième nombre de Fibonacci
 * @n: L'index
 *
 * Return: fib(n), ou -1 si n < 0
 */
int fibonacci(int n)
{
    if (n < 0)
        return (-1);
    if (n == 0)
        return (0);
    if (n == 1)
        return (1);

    return (fibonacci(n - 1) + fibonacci(n - 2));
}

int main(void)
{
    int i;

    for (i = 0; i <= 10; i++)
        printf("fib(%d) = %d\n", i, fibonacci(i));

    return (0);
}
```

> [!warning] Fibonacci récursif naïf est TRÈS lent !
> `fibonacci(n)` a une complexité **exponentielle** O(2^n). Pour `fib(40)`, il y a plus d'un milliard d'appels ! En pratique, on utilise la **mémoïsation** ou une version itérative.

---

## Visualisation de la pile d'appels : `factorial(4)`

Chaque appel récursif crée un nouveau **cadre de pile** (stack frame). Les appels s'empilent, puis se dépilent en retournant les résultats :

```
  PHASE D'EMPILEMENT (appels)          PHASE DE DÉPILEMENT (retours)

  factorial(4)                          factorial(4) = 4 × 6 = 24  ← résultat final
    │                                     ▲
    ├── factorial(3)                      │ return 6
    │     │                               │
    │     ├── factorial(2)               factorial(3) = 3 × 2 = 6
    │     │     │                         ▲
    │     │     ├── factorial(1)          │ return 2
    │     │     │     │                   │
    │     │     │     └── return 1       factorial(2) = 2 × 1 = 2
    │     │     │           ▲             ▲
    │     │     │           │             │ return 1
    │     │     │           │             │
    │     │     │     CAS DE BASE        factorial(1) = 1
    │     │     │                         (cas de base)


  État de la pile (STACK) au moment du cas de base :

  ┌─────────────────────┐  ← Sommet de la pile
  │ factorial(1)  n=1   │
  ├─────────────────────┤
  │ factorial(2)  n=2   │
  ├─────────────────────┤
  │ factorial(3)  n=3   │
  ├─────────────────────┤
  │ factorial(4)  n=4   │
  ├─────────────────────┤
  │ main()              │
  └─────────────────────┘  ← Base de la pile
```

> [!warning] Stack overflow !
> Chaque appel récursif consomme de la mémoire sur la pile. Si la récursion est trop profonde (pas de cas de base, ou problème trop grand), la pile **déborde** → **segmentation fault** (stack overflow).
>
> La pile fait typiquement entre 1 et 8 Mo. Avec ~100 octets par frame, on peut faire environ 10 000 à 80 000 appels récursifs avant le crash.

---

## Règles critiques

### Game Over (ce qui tue la récursion)

| Erreur                      | Conséquence              |
| --------------------------- | ------------------------ |
| Pas de cas de base          | Récursion infinie → stack overflow |
| Pas de progression          | Récursion infinie → stack overflow |
| Mauvais cas de base         | Résultat incorrect       |
| Trop de profondeur          | Stack overflow           |

### Speedrun tips

| Astuce                             | Pourquoi                                      |
| ---------------------------------- | --------------------------------------------- |
| Écrire le cas de base EN PREMIER   | C'est le filet de sécurité                    |
| Vérifier que le problème diminue   | Sinon boucle infinie                          |
| Dessiner l'arbre d'appels          | Pour comprendre le flux                       |
| Compter la profondeur              | Pour estimer si ça tiendra sur la pile        |

---

## Récursion vs Itération

| Critère          | Récursion                         | Itération (boucle)               |
| ---------------- | --------------------------------- | -------------------------------- |
| **Lisibilité**   | Souvent plus claire pour les problèmes naturellement récursifs | Plus directe pour les boucles simples |
| **Mémoire**      | Consomme la pile (un frame par appel) | Mémoire constante               |
| **Performance**   | Overhead des appels de fonction   | Généralement plus rapide         |
| **Risque**       | Stack overflow                    | Boucle infinie (mais pas de crash) |
| **Cas d'usage**  | Arbres, tri, fractales, parsing  | Tableaux, compteurs, I/O         |
| **Transformation** | Toute récursion peut devenir itérative | Toute itération peut devenir récursive |

### Exemple comparé : factorielle

```c
/* Version récursive */
int factorial_rec(int n)
{
    if (n <= 1)
        return (1);
    return (n * factorial_rec(n - 1));
}

/* Version itérative */
int factorial_iter(int n)
{
    int result = 1;

    while (n > 1)
    {
        result *= n;
        n--;
    }

    return (result);
}
```

---

## Exemples Holberton : fonctions de chaîne récursives

### `_strlen` récursif

```c
/**
 * _strlen_recursion - Calcule la longueur d'une chaîne (récursif)
 * @s: La chaîne
 *
 * Return: La longueur
 */
int _strlen_recursion(char *s)
{
    if (*s == '\0')
        return (0);

    return (1 + _strlen_recursion(s + 1));
}
```

Visualisation :
```
  _strlen("Hi!")
    = 1 + _strlen("i!")
    = 1 + 1 + _strlen("!")
    = 1 + 1 + 1 + _strlen("")
    = 1 + 1 + 1 + 0
    = 3
```

### `_puts_recursion` : afficher une chaîne

```c
/**
 * _puts_recursion - Affiche une chaîne suivie d'un newline
 * @s: La chaîne
 */
void _puts_recursion(char *s)
{
    if (*s == '\0')
    {
        _putchar('\n');
        return;
    }

    _putchar(*s);
    _puts_recursion(s + 1);
}
```

### `_strcmp` récursif

```c
/**
 * _strcmp_rec - Compare deux chaînes récursivement
 * @s1: Première chaîne
 * @s2: Deuxième chaîne
 *
 * Return: 0 si égales, différence sinon
 */
int _strcmp_rec(char *s1, char *s2)
{
    if (*s1 == '\0' && *s2 == '\0')
        return (0);
    if (*s1 != *s2)
        return (*s1 - *s2);

    return (_strcmp_rec(s1 + 1, s2 + 1));
}
```

### `is_palindrome` récursif

```c
/**
 * check_palindrome - Aide pour vérifier si c'est un palindrome
 * @s: La chaîne
 * @start: Index de début
 * @end: Index de fin
 *
 * Return: 1 si palindrome, 0 sinon
 */
int check_palindrome(char *s, int start, int end)
{
    if (start >= end)
        return (1);
    if (s[start] != s[end])
        return (0);

    return (check_palindrome(s, start + 1, end - 1));
}

/**
 * is_palindrome - Vérifie si une chaîne est un palindrome
 * @s: La chaîne
 *
 * Return: 1 si palindrome, 0 sinon
 */
int is_palindrome(char *s)
{
    int len = _strlen_recursion(s);

    if (len <= 1)
        return (1);

    return (check_palindrome(s, 0, len - 1));
}
```

### `print_binary` récursif (lien avec [[02 - Bases Numeriques]])

```c
/**
 * print_binary - Affiche un nombre en binaire (récursif)
 * @n: Le nombre
 */
void print_binary(unsigned long int n)
{
    if (n > 1)
        print_binary(n >> 1);

    _putchar((n & 1) + '0');
}
```

---

## Les 4 questions à se poser

Avant d'écrire une fonction récursive, répondre à ces 4 questions :

> [!example] Checklist récursion
> 1. **Quel est le cas de base ?** — Quand est-ce qu'on s'arrête ?
> 2. **Quel est le cas récursif ?** — Comment réduire le problème ?
> 3. **Le problème diminue-t-il ?** — Chaque appel doit se rapprocher du cas de base
> 4. **Quelle est la profondeur maximale ?** — Risque de stack overflow ?

Exemple avec `factorial(n)` :
1. Cas de base : `n <= 1` → retourner 1
2. Cas récursif : `n * factorial(n - 1)`
3. Progression : `n` diminue de 1 à chaque appel → oui
4. Profondeur : `n` appels → ok pour des valeurs raisonnables

---

## Erreurs courantes

### 1. Pas de cas de base → récursion infinie

```c
/* FAUX — pas de cas de base ! */
int factorial(int n)
{
    return (n * factorial(n - 1));  /* N'arrête jamais → stack overflow */
}

/* CORRECT */
int factorial(int n)
{
    if (n <= 1)
        return (1);  /* Cas de base ! */
    return (n * factorial(n - 1));
}
```

### 2. Pas de progression → récursion infinie

```c
/* FAUX — le problème ne diminue pas ! */
int bad(int n)
{
    if (n == 0)
        return (0);
    return (bad(n));  /* Appel avec la même valeur ! */
}

/* CORRECT */
int good(int n)
{
    if (n == 0)
        return (0);
    return (good(n - 1));  /* n diminue */
}
```

### 3. Mauvais cas de base

```c
/* FAUX — ne gère pas n = 0 correctement */
int factorial(int n)
{
    if (n == 1)
        return (1);
    return (n * factorial(n - 1));
}
/* factorial(0) → 0 * factorial(-1) → récursion infinie ! */

/* CORRECT */
int factorial(int n)
{
    if (n <= 1)
        return (1);  /* Gère 0 et 1 */
    return (n * factorial(n - 1));
}
```

### 4. Oublier de retourner la valeur récursive

```c
/* FAUX — le résultat est perdu ! */
int sum(int n)
{
    if (n == 0)
        return (0);
    sum(n - 1);          /* Résultat ignoré ! */
    return (n);           /* Retourne juste n */
}

/* CORRECT */
int sum(int n)
{
    if (n == 0)
        return (0);
    return (n + sum(n - 1));  /* Retourne la somme */
}
```

---

## Récursion avancée : tail recursion

La **récursion terminale** (tail recursion) est une optimisation où l'appel récursif est la **dernière** opération :

```c
/* Récursion NON terminale (le résultat de factorial est multiplié APRÈS) */
int factorial(int n)
{
    if (n <= 1)
        return (1);
    return (n * factorial(n - 1));  /* multiplication APRÈS l'appel */
}

/* Récursion terminale (avec accumulateur) */
int factorial_tail(int n, int acc)
{
    if (n <= 1)
        return (acc);
    return (factorial_tail(n - 1, n * acc));  /* L'appel récursif EST le return */
}

/* Wrapper */
int factorial(int n)
{
    return (factorial_tail(n, 1));
}
```

> [!info] Pourquoi c'est mieux ?
> Avec la récursion terminale, certains compilateurs (GCC avec `-O2`) peuvent transformer la récursion en boucle, éliminant le risque de stack overflow. C'est comme si on avait écrit une boucle `while`.

---

## Liens

- [[01 - Introduction au C et Compilation]] — Boucles et structures de contrôle
- [[02 - Bases Numeriques]] — print_binary récursif
- [[04 - Pointeurs et Memoire]] — La pile (stack) et les stack frames
- [[06 - Fonctions Variadiques]] — print_number récursif dans _printf

---

## Exercices pratiques

### Exercice 1 : Factorielle et puissance
Implémenter `int factorial(int n)` et `int _pow_recursion(int x, int y)` de manière récursive.

### Exercice 2 : Racine carrée naturelle
Écrire `int _sqrt_recursion(int n)` qui retourne la racine carrée naturelle de `n`, ou `-1` si `n` n'a pas de racine carrée naturelle. (Indice : utiliser une fonction helper avec un compteur.)

### Exercice 3 : Palindrome
Écrire `int is_palindrome(char *s)` qui vérifie si une chaîne est un palindrome, de manière **purement récursive** (pas de boucle, pas de variable globale).

### Exercice 4 : Fibonacci itératif vs récursif
Implémenter Fibonacci en récursif ET en itératif. Mesurer le temps pour `fib(40)` avec chaque version (utiliser `time ./prog`).

### Exercice 5 : Tours de Hanoï
Écrire `void hanoi(int n, char from, char to, char aux)` qui affiche les mouvements pour résoudre le problème des Tours de Hanoï avec `n` disques.

### Exercice 6 : Inversion de chaîne
Écrire `void reverse_string(char *s)` de manière récursive (modifier la chaîne en place, sans allocation).
