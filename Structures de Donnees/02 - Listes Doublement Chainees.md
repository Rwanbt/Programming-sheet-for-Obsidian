# Listes Doublement Chainees

---

## Simplement vs Doublement Chainee

### Visualisation

**Liste simplement chainee :**
```
HEAD
 |
 v
[42|*]--->[7|*]--->[13|NULL]

  --> On ne peut aller que vers la DROITE
```

**Liste doublement chainee :**
```
HEAD                                          TAIL
 |                                              |
 v                                              v
[NULL|42|*]<===>[*|7|*]<===>[*|13|NULL]

  <-- On peut aller dans les DEUX sens -->
```

### Analogie : La Playlist Musicale

Imagine une **playlist** sur ton telephone :
- **Simplement chainee** = tu ne peux appuyer que sur **Suivant** >>
- **Doublement chainee** = tu peux appuyer sur **Suivant** >> ET **Precedent** <<

Tu ecoutes la chanson 5 et tu veux revenir a la chanson 4 ? Avec une liste simplement chainee, tu dois **repartir du debut** ! Avec une doublement chainee, un simple clic en arriere suffit.

### Tableau Comparatif

| Critere                  | Simplement Chainee | Doublement Chainee    |
|--------------------------|--------------------|-----------------------|
| **Memoire par noeud**    | 1 pointeur         | 2 pointeurs           |
| **Parcours**             | Avant seulement    | Avant ET arriere      |
| **Suppression d'un noeud** | O(n) chercher prev | O(1) on a deja prev |
| **Insertion avant**      | O(n) chercher prev | O(1) via prev         |
| **Complexite du code**   | Plus simple        | Plus complexe         |
| **Utilisation memoire**  | Moins              | Plus (un pointeur de plus) |

---

## Disposition en Memoire

```
Memoire (heap) :

Adresse 0x1A00:  [NULL | 42 | 0x2C50]    (noeud 1 - HEAD)
                  prev   n    next

Adresse 0x2C50:  [0x1A00 | 7 | 0x3F10]   (noeud 2)
                   prev    n    next

Adresse 0x3F10:  [0x2C50 | 13 | NULL]    (noeud 3 - TAIL)
                   prev    n    next
```

Chaque noeud a **trois champs** :
- `prev` : adresse du noeud precedent
- `n` : la donnee
- `next` : adresse du noeud suivant

---

## La Structure

```c
/**
 * struct dlistint_s - noeud d'une liste doublement chainee
 * @n: valeur entiere
 * @prev: pointeur vers le noeud precedent
 * @next: pointeur vers le noeud suivant
 */
typedef struct dlistint_s
{
    int n;
    struct dlistint_s *prev;
    struct dlistint_s *next;
} dlistint_t;
```

---

## Les Invariants

> [!warning] Regles d'or a TOUJOURS respecter
> Ces conditions doivent etre vraies a tout moment :
>
> 1. `head->prev == NULL` (rien avant le premier)
> 2. `tail->next == NULL` (rien apres le dernier)
> 3. Si `A->next == B`, alors `B->prev == A` (les liens sont symetriques)
>
> ```
> Invariant 3 illustre :
>
>    A->next == B
>    [*|A|*]---->[*|B|*]
>    [*|A|*]<----[*|B|*]
>    B->prev == A
> ```
>
> Si un seul de ces invariants est brise, ta liste est **corrompue**.

---

## Creer un Noeud

```c
/**
 * create_dnode - cree un nouveau noeud doublement chaine
 * @value: la valeur a stocker
 *
 * Return: pointeur vers le nouveau noeud, ou NULL si echec
 */
dlistint_t *create_dnode(int value)
{
    dlistint_t *new;

    new = malloc(sizeof(dlistint_t));
    if (new == NULL)
        return (NULL);

    new->n = value;
    new->prev = NULL;  /* IMPORTANT : initialiser les DEUX */
    new->next = NULL;  /* pointeurs a NULL */

    return (new);
}
```

> [!tip] Initialiser prev ET next
> Ne jamais oublier d'initialiser **les deux** pointeurs a `NULL`. Un pointeur non initialise contient une adresse **aleatoire** et causera un crash.

---

## Ajouter au Debut : `dlistint_add_front()`

```c
/**
 * dlistint_add_front - ajoute un noeud au debut
 * @head: double pointeur vers la tete
 * @value: valeur a ajouter
 *
 * Return: pointeur vers le nouveau noeud, ou NULL si echec
 */
dlistint_t *dlistint_add_front(dlistint_t **head, int value)
{
    dlistint_t *new;

    if (head == NULL)
        return (NULL);

    new = create_dnode(value);
    if (new == NULL)
        return (NULL);

    new->next = *head;        /* Le nouveau pointe vers l'ancien premier */

    if (*head != NULL)
        (*head)->prev = new;  /* L'ancien premier pointe en arriere vers le nouveau */

    *head = new;              /* La tete pointe vers le nouveau */

    return (new);
}
```

> [!warning] N'oublie pas de mettre a jour `prev` de l'ancien head !
> C'est l'erreur la plus frequente. Sans cette ligne, l'invariant 3 est brise.

Visualisation :

```
Etat initial :
  *head ---> [NULL|42|*]<===>[*|7|NULL]

Etape 1 : new = [NULL|99|NULL]

Etape 2 : new->next = *head
  [NULL|99|*]--->[NULL|42|*]<===>[*|7|NULL]
  (prev de 42 n'est pas encore mis a jour !)

Etape 3 : (*head)->prev = new
  [NULL|99|*]<===>[*|42|*]<===>[*|7|NULL]
  (maintenant l'invariant est respecte)

Etape 4 : *head = new
  *head ---> [NULL|99|*]<===>[*|42|*]<===>[*|7|NULL]
```

---

## Ajouter a la Fin

### Avec pointeur `tail` : O(1)

```c
/**
 * dlistint_add_end_with_tail - ajoute a la fin (avec tail)
 * @head: double pointeur vers la tete
 * @tail: double pointeur vers la queue
 * @value: valeur a ajouter
 *
 * Return: pointeur vers le nouveau noeud, ou NULL si echec
 */
dlistint_t *dlistint_add_end_with_tail(dlistint_t **head,
                                        dlistint_t **tail, int value)
{
    dlistint_t *new;

    if (head == NULL || tail == NULL)
        return (NULL);

    new = create_dnode(value);
    if (new == NULL)
        return (NULL);

    if (*head == NULL)  /* Liste vide */
    {
        *head = new;
        *tail = new;
        return (new);
    }

    new->prev = *tail;     /* Le nouveau pointe en arriere vers l'ancien tail */
    (*tail)->next = new;   /* L'ancien tail pointe en avant vers le nouveau */
    *tail = new;           /* tail pointe vers le nouveau */

    return (new);
}
```

### Sans pointeur `tail` : O(n)

```c
/**
 * dlistint_add_end - ajoute a la fin (sans tail, parcours obligatoire)
 * @head: double pointeur vers la tete
 * @value: valeur a ajouter
 *
 * Return: pointeur vers le nouveau noeud, ou NULL si echec
 */
dlistint_t *dlistint_add_end(dlistint_t **head, int value)
{
    dlistint_t *new;
    dlistint_t *current;

    if (head == NULL)
        return (NULL);

    new = create_dnode(value);
    if (new == NULL)
        return (NULL);

    if (*head == NULL)
    {
        *head = new;
        return (new);
    }

    /* Parcourir jusqu'au dernier noeud */
    current = *head;
    while (current->next != NULL)
        current = current->next;

    current->next = new;
    new->prev = current;

    return (new);
}
```

> [!info] Avec tail vs Sans tail
> Si tu gardes un pointeur `tail` en plus de `head`, l'ajout a la fin est **O(1)** au lieu de **O(n)**. Le cout : un pointeur supplementaire a maintenir.

---

## Inserer Avant un Noeud : O(1)

C'est l'un des **gros avantages** de la liste doublement chainee :

```c
/**
 * insert_before - insere un noeud avant un noeud donne
 * @head: double pointeur vers la tete
 * @node: noeud avant lequel inserer
 * @value: valeur a inserer
 *
 * Return: pointeur vers le nouveau noeud, ou NULL si echec
 */
dlistint_t *insert_before(dlistint_t **head, dlistint_t *node, int value)
{
    dlistint_t *new;

    if (head == NULL || node == NULL)
        return (NULL);

    new = create_dnode(value);
    if (new == NULL)
        return (NULL);

    new->next = node;
    new->prev = node->prev;

    if (node->prev != NULL)
        node->prev->next = new;  /* L'ancien precedent pointe vers new */
    else
        *head = new;             /* node etait la tete, new devient la tete */

    node->prev = new;

    return (new);
}
```

```
Inserer 5 avant le noeud [7] :

Avant :
  [NULL|3|*]<===>[*|7|*]<===>[*|10|NULL]

Apres :
  [NULL|3|*]<===>[*|5|*]<===>[*|7|*]<===>[*|10|NULL]

En liste simplement chainee, il faudrait parcourir depuis le debut
pour trouver le noeud avant [7]. Ici, on a directement node->prev !
```

---

## Suppression : Les 4 Cas

La suppression est plus complexe car il faut gerer les pointeurs `prev` ET `next` :

```c
/**
 * delete_dnode - supprime un noeud de la liste
 * @head: double pointeur vers la tete
 * @node: noeud a supprimer
 */
void delete_dnode(dlistint_t **head, dlistint_t *node)
{
    if (head == NULL || node == NULL)
        return;

    /* Cas 1 : le noeud a un precedent */
    if (node->prev != NULL)
        node->prev->next = node->next;

    /* Cas 2 : le noeud est la tete (pas de precedent) */
    if (node->prev == NULL)
        *head = node->next;

    /* Cas 3 : le noeud a un suivant */
    if (node->next != NULL)
        node->next->prev = node->prev;

    /* Cas 4 : le noeud est la queue (pas de suivant) */
    /* -> rien de special, node->next est NULL et c'est OK */

    free(node);
}
```

> [!info] Les 4 cas en detail
>
> **Cas A : Noeud au milieu** (a prev ET next)
> ```
> Avant :  [A]<===>[X]<===>[B]
>            \               /
>             +----prev/next-+
> Apres :  [A]<============>[B]
> ```
>
> **Cas B : Noeud est la tete** (pas de prev)
> ```
> Avant :  HEAD-->[X]<===>[B]
> Apres :  HEAD---------->[B]   (B->prev = NULL)
> ```
>
> **Cas C : Noeud est la queue** (pas de next)
> ```
> Avant :  [A]<===>[X]
> Apres :  [A]   (A->next = NULL)
> ```
>
> **Cas D : Noeud unique** (pas de prev NI next)
> ```
> Avant :  HEAD-->[X]
> Apres :  HEAD-->NULL
> ```

---

## Parcours Avant et Arriere

### Parcours avant (du debut vers la fin)

```c
/**
 * print_dlist - affiche la liste du debut a la fin
 * @head: pointeur vers la tete
 */
void print_dlist(const dlistint_t *head)
{
    const dlistint_t *current = head;

    printf("Avant : ");
    while (current != NULL)
    {
        printf("[%d]", current->n);
        if (current->next != NULL)
            printf(" <=> ");
        current = current->next;
    }
    printf("\n");
}
```

### Parcours arriere (de la fin vers le debut)

```c
/**
 * print_dlist_reverse - affiche la liste de la fin au debut
 * @head: pointeur vers la tete
 */
void print_dlist_reverse(const dlistint_t *head)
{
    const dlistint_t *current = head;

    if (current == NULL)
        return;

    /* D'abord, aller jusqu'a la fin */
    while (current->next != NULL)
        current = current->next;

    /* Puis remonter avec prev */
    printf("Arriere : ");
    while (current != NULL)
    {
        printf("[%d]", current->n);
        if (current->prev != NULL)
            printf(" <=> ");
        current = current->prev;
    }
    printf("\n");
}
```

---

## Liberer la Liste

```c
/**
 * free_dlist - libere toute la liste doublement chainee
 * @head: double pointeur vers la tete
 */
void free_dlist(dlistint_t **head)
{
    dlistint_t *current;
    dlistint_t *next_node;

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
```

> [!tip] C'est le meme pattern que pour la liste simplement chainee
> Sauvegarder `next`, liberer `current`, avancer. Le pointeur `prev` n'a pas besoin d'etre nettoye car le noeud entier est libere.

---

## Boss 1 : Creer et Afficher dans les Deux Sens

> [!example] Exercice complet

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct dlistint_s
{
    int n;
    struct dlistint_s *prev;
    struct dlistint_s *next;
} dlistint_t;

dlistint_t *create_dnode(int value)
{
    dlistint_t *new = malloc(sizeof(dlistint_t));

    if (new == NULL)
        return (NULL);
    new->n = value;
    new->prev = NULL;
    new->next = NULL;
    return (new);
}

dlistint_t *add_end(dlistint_t **head, int value)
{
    dlistint_t *new = create_dnode(value);
    dlistint_t *current;

    if (new == NULL)
        return (NULL);
    if (*head == NULL)
    {
        *head = new;
        return (new);
    }
    current = *head;
    while (current->next != NULL)
        current = current->next;
    current->next = new;
    new->prev = current;
    return (new);
}

void print_forward(const dlistint_t *head)
{
    const dlistint_t *c = head;

    printf("Avant  : ");
    while (c != NULL)
    {
        printf("%d", c->n);
        if (c->next)
            printf(" <=> ");
        c = c->next;
    }
    printf("\n");
}

void print_backward(const dlistint_t *head)
{
    const dlistint_t *c = head;

    if (c == NULL)
        return;
    while (c->next != NULL)
        c = c->next;
    printf("Arriere: ");
    while (c != NULL)
    {
        printf("%d", c->n);
        if (c->prev)
            printf(" <=> ");
        c = c->prev;
    }
    printf("\n");
}

void free_dlist(dlistint_t **head)
{
    dlistint_t *c = *head, *next;

    while (c != NULL)
    {
        next = c->next;
        free(c);
        c = next;
    }
    *head = NULL;
}

int main(void)
{
    dlistint_t *head = NULL;

    add_end(&head, 1);
    add_end(&head, 2);
    add_end(&head, 3);
    add_end(&head, 4);
    add_end(&head, 5);

    print_forward(head);
    /* Avant  : 1 <=> 2 <=> 3 <=> 4 <=> 5 */

    print_backward(head);
    /* Arriere: 5 <=> 4 <=> 3 <=> 2 <=> 1 */

    free_dlist(&head);
    return (0);
}
```

---

## Boss 2 : Trouver le Milieu (Pointeurs Lent/Rapide + Bidirectionnel)

> [!example] Technique classique : la tortue et le lievre

### Methode 1 : Pointeurs lent/rapide (slow/fast)

```c
/**
 * find_middle - trouve le noeud du milieu
 * @head: pointeur vers la tete
 *
 * Return: pointeur vers le noeud du milieu
 */
dlistint_t *find_middle(dlistint_t *head)
{
    dlistint_t *slow = head;
    dlistint_t *fast = head;

    while (fast != NULL && fast->next != NULL)
    {
        slow = slow->next;        /* Avance de 1 */
        fast = fast->next->next;  /* Avance de 2 */
    }

    return (slow);
}
```

```
Liste : [1] <=> [2] <=> [3] <=> [4] <=> [5]

Debut : slow=[1], fast=[1]
Tour 1: slow=[2], fast=[3]
Tour 2: slow=[3], fast=[5]
         fast->next == NULL -> STOP
         slow = [3] = le milieu !
```

### Methode 2 : Approche bidirectionnelle (avantage de la double chaine !)

```c
/**
 * find_middle_bidir - trouve le milieu en partant des deux bouts
 * @head: pointeur vers la tete
 *
 * Return: pointeur vers le noeud du milieu
 */
dlistint_t *find_middle_bidir(dlistint_t *head)
{
    dlistint_t *front = head;
    dlistint_t *back;

    if (head == NULL)
        return (NULL);

    /* Aller a la fin */
    back = head;
    while (back->next != NULL)
        back = back->next;

    /* Avancer des deux cotes jusqu'a se rencontrer */
    while (front != back && front->prev != back)
    {
        front = front->next;
        back = back->prev;
    }

    return (front);
}
```

```
[1] <=> [2] <=> [3] <=> [4] <=> [5]
 ^                               ^
front                           back

Tour 1:
[1] <=> [2] <=> [3] <=> [4] <=> [5]
         ^               ^
       front            back

Tour 2:
[1] <=> [2] <=> [3] <=> [4] <=> [5]
                 ^
            front=back -> STOP ! Milieu = [3]
```

> [!tip] Avantage de la methode bidirectionnelle
> C'est **impossible** avec une liste simplement chainee ! On exploite le fait d'avoir des liens dans les deux sens.

---

## Boss 3 : Inverser en Place

> [!example] Inverser une liste doublement chainee

L'idee : pour chaque noeud, on **echange** `prev` et `next`.

```c
/**
 * reverse_dlist - inverse une liste doublement chainee en place
 * @head: double pointeur vers la tete
 */
void reverse_dlist(dlistint_t **head)
{
    dlistint_t *current;
    dlistint_t *temp;

    if (head == NULL || *head == NULL)
        return;

    current = *head;

    while (current != NULL)
    {
        /* Echanger prev et next */
        temp = current->prev;
        current->prev = current->next;
        current->next = temp;

        /* Si on est arrive a la nouvelle tete */
        if (current->prev == NULL)
        {
            *head = current;
            return;
        }

        /* Avancer (avec prev, car on a echange !) */
        current = current->prev;
    }
}
```

Visualisation :

```
Etat initial :
  NULL<--[1]<==>[2]<==>[3]-->NULL
         HEAD

--- Noeud [1] : echanger prev et next ---
  [1].prev = [1].next = [2]
  [1].next = NULL (ancien prev)
  
  NULL<--[2]<==>[3]-->NULL    [1]-->NULL
                               ^
                              prev=[2]

--- Noeud [2] : echanger prev et next ---
  [2].prev = [2].next = [3]
  [2].next = [1]

  NULL<--[3]-->NULL   [2]<==>[1]-->NULL
                       ^
                      prev=[3]

--- Noeud [3] : echanger prev et next ---
  [3].prev = [3].next = NULL -> c'est la nouvelle tete !
  [3].next = [2]

  NULL<--[3]<==>[2]<==>[1]-->NULL
         HEAD (nouveau)

Resultat : [3] <=> [2] <=> [1]
```

---

## Erreurs Courantes

> [!warning] Les pieges specifiques aux listes doublement chainees

### 1. Oublier de mettre a jour `prev`

```c
/* MAUVAIS : on ajoute au debut mais on oublie prev */
new->next = *head;
*head = new;
/* (*head)->prev pointe toujours vers NULL au lieu de new ! */

/* BON */
new->next = *head;
if (*head != NULL)
    (*head)->prev = new;  /* NE PAS OUBLIER ! */
*head = new;
```

### 2. Mauvais ordre de mise a jour

```c
/* MAUVAIS : on ecrase le lien avant de l'utiliser */
node->next = new;           /* On perd l'ancien next ! */
new->next = node->next;     /* Pointe vers new lui-meme ! BOUCLE INFINIE */

/* BON : sauvegarder d'abord */
new->next = node->next;     /* D'abord lier new a l'ancien suivant */
node->next = new;           /* Puis lier node a new */
new->prev = node;
if (new->next != NULL)
    new->next->prev = new;
```

### 3. Oublier le cas du noeud unique

```c
/* Quand la liste n'a qu'un seul noeud : */
/* head->prev == NULL ET head->next == NULL */
/* La suppression doit mettre *head a NULL */

void delete_dnode(dlistint_t **head, dlistint_t *node)
{
    /* ... */
    if (node->prev == NULL && node->next == NULL)
    {
        /* C'est le seul noeud ! */
        *head = NULL;
        free(node);
        return;
    }
    /* ... traiter les autres cas ... */
}
```

### 4. Ne pas verifier les pointeurs NULL avant d'acceder a `prev` ou `next`

```c
/* MAUVAIS : si node est la tete, node->prev est NULL */
node->prev->next = node->next;  /* SEGFAULT si node->prev == NULL ! */

/* BON */
if (node->prev != NULL)
    node->prev->next = node->next;
else
    *head = node->next;
```

---

## Exercices Pratiques

1. **Ecris** une fonction `dlistint_len(dlistint_t *head)` qui retourne le nombre de noeuds.
2. **Ecris** une fonction `is_palindrome(dlistint_t *head)` qui verifie si la liste est un palindrome (ex: 1-2-3-2-1). Utilise l'approche bidirectionnelle !
3. **Ecris** une fonction `swap_nodes(dlistint_t **head, dlistint_t *a, dlistint_t *b)` qui echange deux noeuds (pas les valeurs, les noeuds eux-memes).
4. **Ecris** une fonction `sort_dlist(dlistint_t **head)` qui trie la liste par insertion (insertion sort).
5. **Ecris** une fonction `split_dlist(dlistint_t *head, dlistint_t **first_half, dlistint_t **second_half)` qui coupe la liste en deux moities.

---

## Carte Mentale Complete

```
              LISTE DOUBLEMENT CHAINEE
                       |
       +---------------+---------------+
       |               |               |
   STRUCTURE       OPERATIONS       INVARIANTS
       |               |               |
  dlistint_t       +---+---+      head->prev = NULL
  {n, *prev,       |       |      tail->next = NULL
   *next}      AJOUT    SUPPR     A->next=B => B->prev=A
               |   |     |
            debut  fin  4 cas:
               |   |    - milieu
           update  O(1) - tete
           prev!   avec - queue
                  tail  - unique
                       |
               +-------+-------+
               |               |
           PARCOURS        AVANTAGES
               |               |
          avant (next)    - insert before O(1)
          arriere (prev)  - delete O(1) avec ptr
          bidirectionnel  - parcours 2 sens
```

---

## Liens

- Precedent : [[01 - Listes Chainees]]
- Suivant : [[03 - Tables de Hachage]]
