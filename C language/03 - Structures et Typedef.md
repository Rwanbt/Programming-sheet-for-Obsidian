# 03 - Structures et Typedef

## Qu'est-ce qu'une structure ?

Une **structure** (`struct`) est un type de données personnalisé qui permet de **regrouper plusieurs variables** de types différents sous un même nom. C'est la brique de base de la programmation orientée données en C.

> [!info] Analogie jeu vidéo
> Imagine un personnage de RPG. Il a un nom (`char *`), des points de vie (`int`), un niveau (`int`), un score (`float`). Au lieu de gérer 4 variables séparées, on les regroupe dans une **struct** `player`.

```c
/* Sans struct : chaos */
char *nom1 = "Alice";
int hp1 = 100;
int level1 = 5;

char *nom2 = "Bob";
int hp2 = 80;
int level2 = 3;

/* Avec struct : organisé */
struct player
{
    char *name;
    int hp;
    int level;
};

struct player alice = {"Alice", 100, 5};
struct player bob = {"Bob", 80, 3};
```

---

## Déclaration en 3 étapes

### Étape 1 : Définir le modèle (blueprint)

```c
struct user_s
{
    char name[50];
    int age;
    float gpa;
};
/* ← N'OUBLIEZ PAS le point-virgule ! */
```

### Étape 2 : Créer une variable (instancier)

```c
struct user_s alice;              /* Déclaration */
struct user_s bob = {"Bob", 22, 3.8};  /* Déclaration + initialisation */
```

### Étape 3 : Accéder aux champs

```c
/* Avec l'opérateur point (.) sur une variable directe */
alice.age = 25;
strcpy(alice.name, "Alice");
alice.gpa = 3.95;

printf("Nom: %s, Age: %d, GPA: %.2f\n", alice.name, alice.age, alice.gpa);
```

---

## Layout mémoire : padding et alignement

Le compilateur ajoute du **padding** (remplissage) pour aligner les champs sur des frontières naturelles.

```c
struct exemple_s
{
    char a;    /* 1 octet */
    /* 3 octets de padding ici ! */
    int b;     /* 4 octets (doit être aligné sur 4) */
    char c;    /* 1 octet */
    /* 3 octets de padding ici ! */
};
/* sizeof = 12 (et non 6 !) */
```

```
  Adresse :  0   1   2   3   4   5   6   7   8   9  10  11
           ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
           │ a │pad│pad│pad│  b (4 octets) │ c │pad│pad│pad│
           └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

> [!tip] Optimiser l'ordre des champs
> En ordonnant les champs du plus grand au plus petit, on réduit le padding :
> ```c
> struct optimise_s
> {
>     int b;     /* 4 octets */
>     char a;    /* 1 octet */
>     char c;    /* 1 octet */
>     /* 2 octets de padding */
> };
> /* sizeof = 8 au lieu de 12 ! */
> ```

---

## Accès aux champs : `.` vs `->`

| Opérateur | Utilisé sur         | Exemple                |
| --------- | ------------------- | ---------------------- |
| `.`       | Variable directe    | `alice.age = 25;`      |
| `->`      | Pointeur sur struct | `ptr->age = 25;`      |

```c
struct user_s alice;
struct user_s *ptr = &alice;

/* Ces deux lignes sont équivalentes : */
alice.age = 25;
ptr->age = 25;

/* ptr->age est un raccourci pour (*ptr).age */
```

> [!warning] Erreurs fréquentes
> ```c
> struct user_s alice;
> struct user_s *ptr = &alice;
>
> alice->age = 25;  /* ERREUR : alice n'est pas un pointeur ! Utiliser . */
> ptr.age = 25;     /* ERREUR : ptr est un pointeur ! Utiliser -> */
> (*ptr).age = 25;  /* OK mais préférer ptr->age */
> ```

---

## Pattern : struct + malloc (allocateur)

Quand on veut créer une struct sur le **heap** (allocation dynamique), on utilise ce pattern :

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/**
 * struct user_s - Représente un utilisateur
 * @name: Le nom de l'utilisateur
 * @age: L'âge de l'utilisateur
 * @gpa: La moyenne générale
 */
struct user_s
{
    char *name;
    int age;
    float gpa;
};

/**
 * new_user - Crée un nouvel utilisateur sur le heap
 * @name: Le nom
 * @age: L'âge
 * @gpa: La moyenne
 *
 * Return: Pointeur vers le nouvel utilisateur, ou NULL si erreur
 */
struct user_s *new_user(char *name, int age, float gpa)
{
    struct user_s *user;

    user = malloc(sizeof(struct user_s));
    if (user == NULL)
        return (NULL);

    user->name = strdup(name);
    if (user->name == NULL)
    {
        free(user);
        return (NULL);
    }

    user->age = age;
    user->gpa = gpa;

    return (user);
}

/**
 * free_user - Libère un utilisateur
 * @user: Le pointeur à libérer
 */
void free_user(struct user_s *user)
{
    if (user == NULL)
        return;
    free(user->name);  /* Libérer les champs d'abord ! */
    free(user);         /* Puis la struct elle-même */
}

int main(void)
{
    struct user_s *alice;

    alice = new_user("Alice", 25, 3.95);
    if (alice == NULL)
        return (1);

    printf("Nom: %s, Age: %d, GPA: %.2f\n",
           alice->name, alice->age, alice->gpa);

    free_user(alice);
    alice = NULL;

    return (0);
}
```

> [!warning] Ordre de libération : champs d'abord, struct ensuite !
> ```c
> /* CORRECT */
> free(user->name);   /* 1. Libérer les champs alloués */
> free(user);          /* 2. Puis la struct */
>
> /* FAUX - FUITE MÉMOIRE ! */
> free(user);          /* La struct est libérée... */
> free(user->name);    /* ... mais on accède à de la mémoire libérée ! */
> ```

---

## `typedef` : créer des alias de type

`typedef` permet de donner un **nouveau nom** à un type existant.

### typedef simple

```c
typedef unsigned int uint;
typedef char *string;

uint age = 25;       /* au lieu de : unsigned int age = 25; */
string nom = "Alice"; /* au lieu de : char *nom = "Alice"; */
```

### typedef + struct (Méthode A : en deux temps)

```c
struct user_s
{
    char *name;
    int age;
};
typedef struct user_s user_t;

/* Utilisation */
user_t alice;
alice.age = 25;
```

### typedef + struct (Méthode B : en un seul bloc)

```c
typedef struct user_s
{
    char *name;
    int age;
} user_t;

/* Utilisation */
user_t alice;
user_t *ptr = &alice;
ptr->age = 25;
```

### typedef + pointeur de fonction

```c
typedef int (*op_func_t)(int, int);

int add(int a, int b) { return (a + b); }
int sub(int a, int b) { return (a - b); }

op_func_t operation = add;
printf("%d\n", operation(5, 3));  /* 8 */

operation = sub;
printf("%d\n", operation(5, 3));  /* 2 */
```

Voir aussi : [[05 - Pointeurs de Fonctions]]

---

## Structures imbriquées (nested structs)

```c
typedef struct address_s
{
    char street[100];
    char city[50];
    int zip_code;
} address_t;

typedef struct student_s
{
    char name[50];
    int age;
    address_t address;  /* Struct dans une struct */
} student_t;

int main(void)
{
    student_t s;

    strcpy(s.name, "Alice");
    s.age = 22;
    strcpy(s.address.street, "42 rue du Code");
    strcpy(s.address.city, "Paris");
    s.address.zip_code = 75001;

    printf("%s habite au %s, %s %d\n",
           s.name, s.address.street,
           s.address.city, s.address.zip_code);

    return (0);
}
```

---

## Passer une struct par pointeur

Toujours passer les structs **par pointeur** aux fonctions (sinon, toute la struct est copiée sur la pile) :

```c
/* MAUVAIS : copie toute la struct (lent, modifications perdues) */
void print_user(user_t user)
{
    printf("Nom: %s\n", user.name);
}

/* BIEN : passe juste l'adresse (rapide, peut modifier) */
void print_user(user_t *user)
{
    printf("Nom: %s\n", user->name);
}

/* ENCORE MIEUX : const si on ne modifie pas */
void print_user(const user_t *user)
{
    printf("Nom: %s\n", user->name);
}
```

---

## Structures auto-référencées (listes chaînées)

Une struct peut contenir un **pointeur vers son propre type** — c'est la base des listes chaînées :

```c
/**
 * struct node_s - Noeud d'une liste simplement chaînée
 * @data: La donnée stockée
 * @next: Pointeur vers le noeud suivant
 */
typedef struct node_s
{
    int data;
    struct node_s *next;  /* Auto-référence (pas node_t ici !) */
} node_t;
```

> [!warning] On doit utiliser `struct node_s *next` et non `node_t *next`
> Au moment où le compilateur lit le champ `next`, le typedef `node_t` n'est **pas encore défini** (on est encore à l'intérieur de la déclaration). Il faut donc utiliser le nom de tag `struct node_s`.

```
  Visualisation d'une liste chaînée :

  head
   │
   ▼
  ┌──────┬──────┐    ┌──────┬──────┐    ┌──────┬──────┐
  │ data │ next─┼───▶│ data │ next─┼───▶│ data │ next─┼──▶ NULL
  │  10  │      │    │  20  │      │    │  30  │      │
  └──────┴──────┘    └──────┴──────┘    └──────┴──────┘
```

### Exemple complet : ajouter un noeud en tête

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct node_s
{
    int data;
    struct node_s *next;
} node_t;

/**
 * add_node - Ajoute un noeud en tête de liste
 * @head: Double pointeur vers la tête
 * @data: La valeur à stocker
 *
 * Return: Pointeur vers le nouveau noeud, ou NULL
 */
node_t *add_node(node_t **head, int data)
{
    node_t *new;

    new = malloc(sizeof(node_t));
    if (new == NULL)
        return (NULL);

    new->data = data;
    new->next = *head;
    *head = new;

    return (new);
}

/**
 * print_list - Affiche tous les éléments d'une liste
 * @head: Pointeur vers la tête
 */
void print_list(const node_t *head)
{
    while (head != NULL)
    {
        printf("%d -> ", head->data);
        head = head->next;
    }
    printf("NULL\n");
}

/**
 * free_list - Libère tous les noeuds d'une liste
 * @head: Pointeur vers la tête
 */
void free_list(node_t *head)
{
    node_t *tmp;

    while (head != NULL)
    {
        tmp = head;
        head = head->next;
        free(tmp);
    }
}

int main(void)
{
    node_t *head = NULL;

    add_node(&head, 30);
    add_node(&head, 20);
    add_node(&head, 10);

    print_list(head);  /* 10 -> 20 -> 30 -> NULL */

    free_list(head);
    head = NULL;

    return (0);
}
```

---

## Conventions Holberton

| Élément               | Convention            | Exemple                  |
| --------------------- | --------------------- | ------------------------ |
| Nom du tag struct     | suffixe `_s`          | `struct user_s`          |
| Nom du typedef        | suffixe `_t`          | `user_t`                 |
| Déclaration           | Dans le fichier `.h`  | `main.h` ou `lists.h`   |
| Documentation Betty   | Commentaire `/** */`  | Avant chaque struct      |

Exemple conforme :

```c
/* Dans le fichier header: lists.h */
#ifndef LISTS_H
#define LISTS_H

#include <stdlib.h>

/**
 * struct list_s - Liste simplement chaînée
 * @str: Chaîne de caractères
 * @len: Longueur de la chaîne
 * @next: Pointeur vers le noeud suivant
 *
 * Description: Noeud d'une liste simplement chaînée
 * pour le projet Holberton
 */
typedef struct list_s
{
    char *str;
    unsigned int len;
    struct list_s *next;
} list_t;

/* Prototypes */
list_t *add_node(list_t **head, const char *str);
size_t print_list(const list_t *h);
void free_list(list_t *head);

#endif /* LISTS_H */
```

---

## Erreurs fréquentes

### 1. Oublier le point-virgule après la struct

```c
struct user_s
{
    char *name;
    int age;
}   /* ← ERREUR : il manque ; */
/* Le compilateur donnera une erreur cryptique sur les lignes SUIVANTES */

struct user_s
{
    char *name;
    int age;
};  /* ← CORRECT */
```

### 2. Confondre `.` et `->`

```c
user_t user;
user_t *ptr = &user;

user.age = 25;   /* CORRECT : variable directe → point */
ptr->age = 25;   /* CORRECT : pointeur → flèche */

user->age = 25;  /* ERREUR ! */
ptr.age = 25;    /* ERREUR ! */
```

### 3. Oublier de vérifier malloc

```c
/* FAUX */
user_t *u = malloc(sizeof(user_t));
u->age = 25;  /* Crash si malloc a retourné NULL ! */

/* CORRECT */
user_t *u = malloc(sizeof(user_t));
if (u == NULL)
    return (NULL);
u->age = 25;
```

### 4. Mauvais ordre de libération

```c
/* FAUX — fuite mémoire */
free(user);
free(user->name);  /* Accès mémoire libérée → comportement indéfini */

/* CORRECT */
free(user->name);
free(user);
```

### 5. Sizeof sur un pointeur au lieu de la struct

```c
user_t *u;

u = malloc(sizeof(u));         /* FAUX : sizeof d'un pointeur (8 octets) ! */
u = malloc(sizeof(user_t));    /* CORRECT : sizeof de la struct */
u = malloc(sizeof(*u));        /* CORRECT aussi, et plus sûr */
```

---

## Liens

- [[01 - Introduction au C et Compilation]] — Types de base et syntaxe
- [[04 - Pointeurs et Memoire]] — malloc, free, mémoire dynamique
- [[05 - Pointeurs de Fonctions]] — typedef avec pointeurs de fonctions

---

## Exercices pratiques

### Exercice 1 : Fiche d'identité
Créer une struct `identity_s` avec `nom`, `prenom`, `age`, `taille` (float). Écrire une fonction `print_identity` qui affiche proprement les informations.

### Exercice 2 : Point et Distance
Créer une struct `point_s` avec des coordonnées `x` et `y` (float). Écrire une fonction `float distance(point_t *a, point_t *b)` qui calcule la distance entre deux points.

### Exercice 3 : Liste de chiens
Créer une struct `dog_s` avec `name` (char *), `age` (float), `owner` (char *). Écrire :
- `dog_t *new_dog(char *name, float age, char *owner)` — alloue et copie tout avec `strdup`
- `void print_dog(dog_t *d)` — affiche les infos
- `void free_dog(dog_t *d)` — libère proprement

### Exercice 4 : Liste chaînée
Implémenter une liste chaînée d'entiers avec :
- `add_node_end` — ajoute en fin de liste
- `print_list` — affiche tous les éléments
- `list_len` — retourne le nombre de noeuds
- `free_list` — libère toute la liste

### Exercice 5 : Tableau d'étudiants
Créer un tableau de 5 structs `student_t` (alloué avec `malloc`), remplir les données, trier par moyenne (GPA) décroissante, afficher, libérer.
