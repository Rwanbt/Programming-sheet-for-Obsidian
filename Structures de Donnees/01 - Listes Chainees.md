# Listes Chainees

---

## Introduction : C'est quoi une Liste Chainee ?

Imagine un **RPG** (jeu de role). Tu as une **chaine de quetes** : chaque quete te donne un indice pour trouver la suivante. Tu ne connais pas toutes les quetes a l'avance -- tu decouvres la prochaine uniquement quand tu termines la quete actuelle.

> [!info] Definition
> Une **liste chainee** (linked list) est une structure de donnees ou chaque element (appele **noeud**) contient :
> 1. Une **valeur** (la donnee)
> 2. Un **pointeur** vers le noeud suivant
>
> Le dernier noeud pointe vers `NULL` (fin de la chaine).

### Visualisation ASCII

```
HEAD
 |
 v
[42|*]--->[7|*]--->[13|NULL]
```

Chaque boite `[valeur|pointeur]` est un **noeud**. La fleche `*` pointe vers le noeud suivant. Le dernier noeud a `NULL` comme pointeur suivant : c'est la **fin de la liste**.

Version detaillee :

```
  HEAD
   |
   v
+------+------+     +------+------+     +------+------+
|  42  |   *--+---->|   7  |   *--+---->|  13  | NULL |
+------+------+     +------+------+     +------+------+
  noeud 1             noeud 2             noeud 3
```

---

## Tableau vs Liste Chainee

| Critere            | Tableau (Array)        | Liste Chainee          |
|--------------------|------------------------|------------------------|
| **Taille**         | Fixe a la declaration  | Dynamique (grandit/retrecit) |
| **Acces**          | O(1) par index         | O(n) parcours sequentiel |
| **Insertion debut**| O(n) decalage          | O(1) un seul pointeur  |
| **Insertion fin**  | O(1) si place dispo    | O(n) parcours jusqu'au bout |
| **Suppression**    | O(n) decalage          | O(1) si on a le pointeur |
| **Memoire**        | Contigue (un bloc)     | Dispersee (heap)       |
| **Cache CPU**      | Excellent (localite)   | Mauvais (sauts memoire)|

> [!tip] Quand utiliser quoi ?
> - **Tableau** : quand tu connais la taille a l'avance et que tu as besoin d'acces rapide par index.
> - **Liste chainee** : quand tu inseres/supprimes souvent et que la taille change constamment.

---

## La Structure d'un Noeud

### Declaration

```c
typedef struct node_s
{
    int n;
    struct node_s *next;
} node_t;
```

Decomposons :
- `struct node_s` : on donne un **nom de tag** a la structure.
- `int n` : la donnee stockee dans le noeud.
- `struct node_s *next` : pointeur vers le prochain noeud.
- `node_t` : alias pratique grace au `typedef`.

### Pourquoi `struct node_s *next` et PAS `node_t *next` ?

> [!warning] Piege classique !
> Au moment ou le compilateur lit la ligne `struct node_s *next`, il n'a **pas encore fini** de lire le `typedef`. Le nom `node_t` **n'existe pas encore** !
>
> ```
> typedef struct node_s     <-- Le compilateur commence ici
> {
>     int n;
>     node_t *next;         <-- ERREUR ! node_t n'existe pas encore !
> } node_t;                 <-- node_t est cree ICI seulement
> ```
>
> Par contre, `struct node_s` est declare des la premiere ligne, donc on peut l'utiliser a l'interieur.

---

## Disposition en Memoire

Contrairement a un tableau qui occupe un **bloc continu** de memoire, les noeuds d'une liste chainee sont **disperses** dans le heap :

```
Memoire (heap) :
                                              
Adresse 0x1A00:  [42 | 0x2C50]   (noeud 1)
         ...
Adresse 0x2C50:  [ 7 | 0x3F10]   (noeud 2)
         ...
Adresse 0x3F10:  [13 | NULL   ]   (noeud 3)
         ...

HEAD = 0x1A00
```

> [!info] Observation
> Les noeuds ne sont **pas** cote a cote en memoire. Chaque `malloc` peut renvoyer une adresse n'importe ou dans le heap. C'est le **pointeur `next`** qui cree le lien logique entre eux.

---

## Creer un Noeud : `new_node()`

```c
/**
 * new_node - cree un nouveau noeud
 * @value: la valeur a stocker
 *
 * Return: pointeur vers le nouveau noeud, ou NULL si echec
 */
node_t *new_node(int value)
{
    node_t *node;

    node = malloc(sizeof(node_t));
    if (node == NULL)
        return (NULL);

    node->n = value;
    node->next = NULL;

    return (node);
}
```

> [!warning] Toujours verifier `malloc` !
> `malloc` peut echouer (plus de memoire). **Toujours** verifier si le retour est `NULL` avant d'utiliser le pointeur.

---

## Ajouter au Debut : `add_node()` avec Double Pointeur

### Pourquoi un double pointeur `**head` ?

Imaginons qu'on passe juste `*head` :

```c
/* MAUVAIS : ne modifie PAS head dans main */
void add_node_MAUVAIS(node_t *head, int value)
{
    node_t *new = new_node(value);

    new->next = head;
    head = new;  /* Modifie la COPIE locale, pas l'original ! */
}
```

```
Avant l'appel :
  main:  head ----> [42|*] ---> [7|NULL]

Dans la fonction :
  head (copie) ----> [99|*] ---> [42|*] ---> [7|NULL]

Apres l'appel :
  main:  head ----> [42|*] ---> [7|NULL]   <-- RIEN N'A CHANGE !
  Le noeud [99] est perdu (fuite memoire) !
```

> [!tip] La solution : le double pointeur
> En passant `**head`, on passe **l'adresse du pointeur head lui-meme**. On peut donc modifier `*head` (le contenu de head) depuis la fonction.

### Implementation correcte

```c
/**
 * add_node - ajoute un noeud au debut de la liste
 * @head: double pointeur vers la tete de liste
 * @value: valeur a ajouter
 *
 * Return: pointeur vers le nouveau noeud, ou NULL si echec
 */
node_t *add_node(node_t **head, int value)
{
    node_t *new;

    if (head == NULL)
        return (NULL);

    new = new_node(value);
    if (new == NULL)
        return (NULL);

    new->next = *head;  /* Le nouveau pointe vers l'ancien premier */
    *head = new;         /* head pointe maintenant vers le nouveau */

    return (new);
}
```

Visualisation etape par etape :

```
Etat initial :
  *head ---> [42|*] ---> [7|NULL]

Etape 1 : new = new_node(99)
  new ---> [99|NULL]

Etape 2 : new->next = *head
  new ---> [99|*] ---> [42|*] ---> [7|NULL]

Etape 3 : *head = new
  *head ---> [99|*] ---> [42|*] ---> [7|NULL]
```

---

## Ajouter a la Fin : `add_node_end()`

```c
/**
 * add_node_end - ajoute un noeud a la fin de la liste
 * @head: double pointeur vers la tete de liste
 * @value: valeur a ajouter
 *
 * Return: pointeur vers le nouveau noeud, ou NULL si echec
 */
node_t *add_node_end(node_t **head, int value)
{
    node_t *new;
    node_t *current;

    if (head == NULL)
        return (NULL);

    new = new_node(value);
    if (new == NULL)
        return (NULL);

    /* Si la liste est vide, le nouveau noeud devient la tete */
    if (*head == NULL)
    {
        *head = new;
        return (new);
    }

    /* Sinon, parcourir jusqu'au dernier noeud */
    current = *head;
    while (current->next != NULL)
        current = current->next;

    current->next = new;

    return (new);
}
```

```
Parcours pour trouver la fin :

  current
     |
     v
    [42|*]--->[7|*]--->[13|NULL]

  current->next != NULL ? OUI -> avancer

            current
               |
               v
    [42|*]--->[7|*]--->[13|NULL]

  current->next != NULL ? OUI -> avancer

                        current
                           |
                           v
    [42|*]--->[7|*]--->[13|NULL]

  current->next != NULL ? NON -> on est au dernier !

  current->next = new :
    [42|*]--->[7|*]--->[13|*]--->[99|NULL]
```

> [!info] Complexite
> `add_node` (debut) : **O(1)** -- une seule operation.
> `add_node_end` (fin) : **O(n)** -- on doit parcourir toute la liste.

---

## Parcours de Liste : Patterns Essentiels

### Afficher la liste : `print_list()`

```c
/**
 * print_list - affiche tous les elements d'une liste
 * @head: pointeur vers la tete de liste
 */
void print_list(const node_t *head)
{
    const node_t *current;

    current = head;
    while (current != NULL)
    {
        printf("%d\n", current->n);
        current = current->next;
    }
}
```

> [!tip] Le pattern de parcours
> C'est **toujours** le meme schema :
> ```c
> current = head;
> while (current != NULL)
> {
>     /* faire quelque chose avec current */
>     current = current->next;
> }
> ```
> Retiens ce pattern par coeur !

### Longueur de la liste : `list_len()`

```c
/**
 * list_len - compte le nombre de noeuds
 * @head: pointeur vers la tete de liste
 *
 * Return: nombre de noeuds
 */
size_t list_len(const node_t *head)
{
    size_t count = 0;
    const node_t *current = head;

    while (current != NULL)
    {
        count++;
        current = current->next;
    }

    return (count);
}
```

### Recherche : `search()`

```c
/**
 * search - cherche une valeur dans la liste
 * @head: pointeur vers la tete de liste
 * @value: valeur a chercher
 *
 * Return: pointeur vers le noeud trouve, ou NULL si absent
 */
node_t *search(node_t *head, int value)
{
    node_t *current = head;

    while (current != NULL)
    {
        if (current->n == value)
            return (current);
        current = current->next;
    }

    return (NULL);
}
```

---

## Suppression de Noeuds

### Supprimer le premier : `pop_front()`

```c
/**
 * pop_front - supprime le premier noeud
 * @head: double pointeur vers la tete de liste
 *
 * Return: la valeur du noeud supprime, ou -1 si liste vide
 */
int pop_front(node_t **head)
{
    node_t *temp;
    int value;

    if (head == NULL || *head == NULL)
        return (-1);

    temp = *head;           /* Sauvegarder le noeud a supprimer */
    value = temp->n;        /* Sauvegarder la valeur */
    *head = temp->next;     /* Avancer la tete */
    free(temp);             /* Liberer l'ancien premier */

    return (value);
}
```

### Liberer toute la liste : `free_list()`

> [!warning] Piege mortel : sauvegarder `next` AVANT `free` !
> ```c
> /* MAUVAIS : Use After Free ! */
> while (current != NULL)
> {
>     free(current);
>     current = current->next;  /* current est deja libere ! BOOM ! */
> }
> ```

```c
/**
 * free_list - libere toute la liste
 * @head: double pointeur vers la tete
 */
void free_list(node_t **head)
{
    node_t *current;
    node_t *next_node;

    if (head == NULL)
        return;

    current = *head;
    while (current != NULL)
    {
        next_node = current->next;  /* SAUVEGARDER avant de free ! */
        free(current);
        current = next_node;
    }

    *head = NULL;  /* Bonne pratique : mettre head a NULL */
}
```

Visualisation :

```
Iteration 1 :
  current = [42|*]    next_node = [7|*]
  free([42])
  current = [7|*]

Iteration 2 :
  current = [7|*]     next_node = [13|NULL]
  free([7])
  current = [13|NULL]

Iteration 3 :
  current = [13|NULL]  next_node = NULL
  free([13])
  current = NULL       -> FIN
```

### Supprimer par valeur : `delete_node()`

```c
/**
 * delete_node - supprime le premier noeud contenant la valeur donnee
 * @head: double pointeur vers la tete
 * @value: valeur a supprimer
 *
 * Return: 1 si supprime, 0 sinon
 */
int delete_node(node_t **head, int value)
{
    node_t *current;
    node_t *prev;

    if (head == NULL || *head == NULL)
        return (0);

    /* Cas special : c'est le premier noeud */
    if ((*head)->n == value)
    {
        current = *head;
        *head = current->next;
        free(current);
        return (1);
    }

    /* Parcourir avec un pointeur "previous" */
    prev = *head;
    current = (*head)->next;

    while (current != NULL)
    {
        if (current->n == value)
        {
            prev->next = current->next;  /* Court-circuiter */
            free(current);
            return (1);
        }
        prev = current;
        current = current->next;
    }

    return (0);  /* Valeur non trouvee */
}
```

```
Suppression du noeud contenant 7 :

Avant :
  [42|*]--->[7|*]--->[13|NULL]
   prev    current

Apres prev->next = current->next :
  [42|*]---------->[13|NULL]
            [7|*] (libere)
```

---

## Boss 1 : Creer et Afficher une Liste

> [!example] Exercice complet

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct node_s
{
    int n;
    struct node_s *next;
} node_t;

node_t *new_node(int value)
{
    node_t *node = malloc(sizeof(node_t));

    if (node == NULL)
        return (NULL);
    node->n = value;
    node->next = NULL;
    return (node);
}

node_t *add_node(node_t **head, int value)
{
    node_t *new;

    if (head == NULL)
        return (NULL);
    new = new_node(value);
    if (new == NULL)
        return (NULL);
    new->next = *head;
    *head = new;
    return (new);
}

void print_list(const node_t *head)
{
    const node_t *current = head;

    while (current != NULL)
    {
        printf("[%d]", current->n);
        if (current->next != NULL)
            printf(" -> ");
        current = current->next;
    }
    printf(" -> NULL\n");
}

void free_list(node_t **head)
{
    node_t *current, *next_node;

    if (head == NULL)
        return;
    current = *head;
    while (current != NULL)
    {
        next_node = current->next;
        free(current);
        current = next_node;
    }
    *head = NULL;
}

int main(void)
{
    node_t *head = NULL;

    add_node(&head, 13);
    add_node(&head, 7);
    add_node(&head, 42);

    print_list(head);
    /* Affiche : [42] -> [7] -> [13] -> NULL */

    free_list(&head);
    return (0);
}
```

---

## Boss 2 : Inverser une Liste en Place

> [!example] C'est un classique des entretiens techniques !

### Visualisation etape par etape

```
Etat initial :
  prev = NULL
  current = [1|*]--->[2|*]--->[3|NULL]

--- Iteration 1 ---
  next_node = [2|*]           (sauvegarder)
  current->next = NULL        (inverser le lien)
  prev = [1|NULL]             (avancer prev)
  current = [2|*]             (avancer current)

  NULL <---[1|NULL]     [2|*]--->[3|NULL]
            prev        current

--- Iteration 2 ---
  next_node = [3|NULL]
  current->next = prev        (inverser le lien)
  prev = [2|*]
  current = [3|NULL]

  NULL <---[1|NULL]<---[2|*]     [3|NULL]
                        prev     current

--- Iteration 3 ---
  next_node = NULL
  current->next = prev        (inverser le lien)
  prev = [3|*]
  current = NULL              -> FIN

  NULL <---[1|NULL]<---[2|*]<---[3|*]
                                 prev = nouvelle tete !
```

### Code

```c
/**
 * reverse_list - inverse une liste chainee en place
 * @head: double pointeur vers la tete
 */
void reverse_list(node_t **head)
{
    node_t *prev = NULL;
    node_t *current;
    node_t *next_node;

    if (head == NULL || *head == NULL)
        return;

    current = *head;

    while (current != NULL)
    {
        next_node = current->next;  /* 1. Sauvegarder le suivant */
        current->next = prev;       /* 2. Inverser le lien */
        prev = current;             /* 3. Avancer prev */
        current = next_node;        /* 4. Avancer current */
    }

    *head = prev;  /* La nouvelle tete est le dernier noeud traite */
}
```

> [!tip] Retiens les 3 variables
> `prev`, `current`, `next_node` -- c'est tout ce dont tu as besoin pour inverser une liste.

---

## Boss 3 : Insertion Triee

> [!example] Inserer un noeud dans une liste deja triee en ordre croissant

```c
/**
 * insert_sorted - insere un noeud dans une liste triee
 * @head: double pointeur vers la tete
 * @value: valeur a inserer
 *
 * Return: pointeur vers le nouveau noeud, ou NULL si echec
 */
node_t *insert_sorted(node_t **head, int value)
{
    node_t *new;
    node_t *current;

    if (head == NULL)
        return (NULL);

    new = new_node(value);
    if (new == NULL)
        return (NULL);

    /* Cas 1 : liste vide OU valeur plus petite que la tete */
    if (*head == NULL || value <= (*head)->n)
    {
        new->next = *head;
        *head = new;
        return (new);
    }

    /* Cas 2 : trouver la bonne position */
    current = *head;
    while (current->next != NULL && current->next->n < value)
        current = current->next;

    /* Inserer apres current */
    new->next = current->next;
    current->next = new;

    return (new);
}
```

```
Inserer 5 dans [1] -> [3] -> [7] -> [10] -> NULL

Parcours : current = [1], current->next->n = 3 < 5 ? OUI -> avancer
           current = [3], current->next->n = 7 < 5 ? NON -> STOP

  [1|*]--->[3|*]--->[7|*]--->[10|NULL]
            current

  new = [5|*]
  new->next = current->next = [7|*]
  current->next = new

  [1|*]--->[3|*]--->[5|*]--->[7|*]--->[10|NULL]
```

---

## Bonus : Introduction aux Listes Doublement Chainees

Une liste doublement chainee ajoute un pointeur `prev` qui pointe vers le noeud **precedent** :

```
NULL <---[prev|42|next]<--->[prev|7|next]<--->[prev|13|next]---> NULL
          ^                                         ^
         HEAD                                     TAIL
```

Avantages :
- Parcours dans les **deux sens**
- Suppression en **O(1)** si on a le pointeur du noeud (pas besoin de chercher le precedent)

Pour en savoir plus : [[02 - Listes Doublement Chainees]]

---

## Erreurs Courantes

> [!warning] Les pieges a eviter absolument

### 1. Oublier de sauvegarder `next` avant `free`

```c
/* MAUVAIS */
free(current);
current = current->next;  /* Use After Free ! Comportement indefini */

/* BON */
next_node = current->next;
free(current);
current = next_node;
```

### 2. Modifier `head` sans double pointeur

```c
/* MAUVAIS : head dans main ne change pas */
void add(node_t *head, int value) { ... head = new; }

/* BON : on modifie *head, donc head dans main change */
void add(node_t **head, int value) { ... *head = new; }
```

### 3. Oublier de verifier `malloc`

```c
/* MAUVAIS */
node_t *node = malloc(sizeof(node_t));
node->n = 42;  /* Si malloc a echoue, SEGFAULT ! */

/* BON */
node_t *node = malloc(sizeof(node_t));
if (node == NULL)
    return (NULL);
node->n = 42;
```

### 4. Fuite memoire : perdre la reference a la tete

```c
/* MAUVAIS */
head = head->next;  /* L'ancien premier noeud est perdu ! */

/* BON */
node_t *temp = head;
head = head->next;
free(temp);
```

---

## Exercices Pratiques

1. **Ecris** une fonction `sum_list(node_t *head)` qui retourne la somme de tous les elements.
2. **Ecris** une fonction `get_nth(node_t *head, int index)` qui retourne le noeud a l'index donne (commence a 0).
3. **Ecris** une fonction `remove_duplicates(node_t **head)` qui supprime les doublons dans une liste triee.
4. **Ecris** une fonction `merge_sorted(node_t *a, node_t *b)` qui fusionne deux listes triees en une seule liste triee.
5. **Ecris** une fonction qui detecte si une liste a un **cycle** (utilise la technique des deux pointeurs : tortue et lievre).

---

## Carte Mentale Complete

```
                    LISTE CHAINEE
                         |
         +---------------+---------------+
         |               |               |
      STRUCTURE       OPERATIONS       ERREURS
         |               |               |
    node_t {n, *next}    |          - free sans save next
         |               |          - *head sans **head
    [val|next]--->...    |          - oublier malloc check
                         |
         +-------+-------+-------+-------+
         |       |       |       |       |
       CREER  AJOUTER PARCOURIR SUPPR  INVERSER
         |       |       |       |       |
      new_node  debut   print   pop    prev/curr/next
                 fin    len     free
                        search  by_val
```

---

## Liens

- Suivant : [[02 - Listes Doublement Chainees]]
- Voir aussi : [[03 - Tables de Hachage]]
