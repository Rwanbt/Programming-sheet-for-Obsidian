# Arbres Binaires

> [!info] Contexte
> Les arbres binaires sont des structures de donnees **hierarchiques** qui permettent de passer d'une complexite lineaire O(n) (listes chainees) a une complexite logarithmique O(log n) pour les operations de recherche, insertion et suppression. C'est un pilier fondamental de l'informatique.

---

## 1. Qu'est-ce qu'un Arbre Binaire ?

Un **arbre binaire** est une structure de donnees ou chaque noeud possede **au maximum deux enfants** : un enfant gauche et un enfant droit.

### Terminologie essentielle

| Terme | Definition |
|-------|-----------|
| **Racine (Root)** | Le noeud tout en haut de l'arbre, sans parent |
| **Noeud (Node)** | Un element de l'arbre contenant une donnee et des pointeurs |
| **Feuille (Leaf)** | Un noeud sans enfant (left == NULL et right == NULL) |
| **Parent** | Le noeud directement au-dessus d'un autre |
| **Enfant (Child)** | Un noeud directement en dessous d'un autre |
| **Hauteur (Height)** | Nombre d'aretes sur le chemin le plus long d'un noeud vers une feuille |
| **Profondeur (Depth)** | Nombre d'aretes entre un noeud et la racine |
| **Niveau (Level)** | Profondeur + 1 (la racine est au niveau 1) |
| **Taille (Size)** | Nombre total de noeuds dans l'arbre |

### Representation visuelle

```
         .-------(098)-------.
    .--(012)--.          .--(402)--.
  (006)     (016)     (256)     (512)
```

```
Vocabulaire sur cet arbre :
- Racine     : 098
- Feuilles   : 006, 016, 256, 512
- Hauteur    : 2 (de la racine a une feuille)
- Profondeur de 016 : 2
- Taille     : 7 noeuds
```

> [!tip] Difference Hauteur vs Profondeur
> La **profondeur** se mesure depuis la **racine vers le bas** (combien d'aretes pour atteindre ce noeud).
> La **hauteur** se mesure depuis le **noeud vers le bas** (le plus long chemin vers une feuille).
> La hauteur de l'arbre = la hauteur de la racine.

---

## 2. Structure en C

Contrairement a une [[01 - Listes Chainees|liste chainee]] ou un noeud pointe vers le "suivant", un noeud d'arbre binaire possede **trois pointeurs** : parent, left, right.

```c
/**
 * struct binary_tree_s - Noeud d'arbre binaire
 * @n: Entier stocke dans le noeud
 * @parent: Pointeur vers le noeud parent
 * @left: Pointeur vers l'enfant gauche
 * @right: Pointeur vers l'enfant droit
 */
struct binary_tree_s
{
    int n;
    struct binary_tree_s *parent;
    struct binary_tree_s *left;
    struct binary_tree_s *right;
};

typedef struct binary_tree_s binary_tree_t;
```

> [!warning] Holberton / ALX
> Le projet impose les typedef suivants pour les variantes :
> ```c
> typedef struct binary_tree_s bst_t;   /* Binary Search Tree */
> typedef struct binary_tree_s avl_t;   /* AVL Tree */
> typedef struct binary_tree_s heap_t;  /* Max Binary Heap */
> ```

---

## 3. Types d'arbres binaires

### Arbre Binaire Simple
Aucune regle d'ordre. Chaque noeud a au maximum 2 enfants.

### Arbre Binaire de Recherche (BST - Binary Search Tree)
Regle : pour chaque noeud, **tous les noeuds a gauche sont inferieurs** et **tous les noeuds a droite sont superieurs**.

```
        (050)
       /     \
    (030)   (070)
    /   \   /   \
 (020)(040)(060)(080)

In-order : 20, 30, 40, 50, 60, 70, 80 (trie !)
```

> [!info] Gain de complexite
> Dans une liste chainee, chercher un element = O(n).
> Dans un BST equilibre, chercher un element = O(log n).
> Pour 1 000 000 d'elements : ~1 000 000 operations vs ~20 operations !

### Arbre Complet (Complete)
Tous les niveaux sont remplis sauf possiblement le dernier, qui est rempli **de gauche a droite**.

### Arbre Plein (Full)
Chaque noeud a exactement **0 ou 2** enfants (jamais 1 seul).

### Arbre Parfait (Perfect)
Tous les niveaux sont **completement remplis**. Un arbre parfait est a la fois complet et plein.

```
Parfait (tous les niveaux remplis) :
        (A)
       /   \
     (B)   (C)
    / \    / \
  (D)(E) (F)(G)

Plein (0 ou 2 enfants) mais PAS parfait :
        (A)
       /   \
     (B)   (C)
    / \
  (D) (E)

Complet mais PAS plein :
        (A)
       /   \
     (B)   (C)
    /
  (D)
```

### Arbre AVL (Adelson-Velsky et Landis)
Un BST **auto-equilibre** : la difference de hauteur entre le sous-arbre gauche et droit de chaque noeud est au maximum de 1.

---

## 4. Operations de base

### 4.1 Creer un noeud

```c
#include "binary_trees.h"

/**
 * binary_tree_node - Cree un noeud d'arbre binaire.
 * @parent: Pointeur vers le parent du noeud a creer.
 * @value: Valeur a mettre dans le nouveau noeud.
 * Return: Pointeur vers le nouveau noeud, ou NULL en cas d'echec.
 */
binary_tree_t *binary_tree_node(binary_tree_t *parent, int value)
{
    binary_tree_t *new_node;

    new_node = malloc(sizeof(binary_tree_t));
    if (new_node == NULL)
        return (NULL);

    new_node->n = value;
    new_node->parent = parent;
    new_node->left = NULL;
    new_node->right = NULL;

    return (new_node);
}
```

### 4.2 Inserer a gauche

> [!warning] Attention aux pointeurs
> Si le parent a deja un enfant gauche, le nouveau noeud **prend sa place** et l'ancien enfant devient l'enfant gauche du nouveau noeud.

```c
binary_tree_t *binary_tree_insert_left(binary_tree_t *parent, int value)
{
    binary_tree_t *new_node;

    if (parent == NULL)
        return (NULL);

    new_node = binary_tree_node(parent, value);
    if (new_node == NULL)
        return (NULL);

    /* Si le parent a deja un enfant gauche */
    if (parent->left != NULL)
    {
        new_node->left = parent->left;
        parent->left->parent = new_node;
    }
    parent->left = new_node;

    return (new_node);
}
```

### 4.3 Supprimer un arbre entier

On utilise un parcours **post-order** : on supprime d'abord les enfants avant le parent (sinon on perdrait l'acces aux enfants).

```c
void binary_tree_delete(binary_tree_t *tree)
{
    if (tree == NULL)
        return;

    binary_tree_delete(tree->left);   /* Supprimer sous-arbre gauche */
    binary_tree_delete(tree->right);  /* Supprimer sous-arbre droit  */
    free(tree);                       /* Supprimer le noeud courant  */
}
```

### 4.4 Verifier si un noeud est une feuille

```c
int binary_tree_is_leaf(const binary_tree_t *node)
{
    if (node == NULL)
        return (0);

    return (node->left == NULL && node->right == NULL);
}
```

### 4.5 Calculer la hauteur

```c
size_t binary_tree_height(const binary_tree_t *tree)
{
    size_t left_h, right_h;

    if (tree == NULL || (tree->left == NULL && tree->right == NULL))
        return (0);

    left_h = binary_tree_height(tree->left);
    right_h = binary_tree_height(tree->right);

    return (1 + (left_h > right_h ? left_h : right_h));
}
```

### 4.6 Calculer la taille

```c
size_t binary_tree_size(const binary_tree_t *tree)
{
    if (tree == NULL)
        return (0);

    return (1 + binary_tree_size(tree->left) + binary_tree_size(tree->right));
}
```

---

## 5. Parcours d'arbres (Traversals)

Les parcours sont la maniere de **visiter tous les noeuds** d'un arbre. Il en existe 4 principaux.

> [!tip] Moyen mnemotechnique
> Le nom du parcours indique **quand on traite la racine** :
> - **Pre**-order : racine en **premier** (avant les enfants)
> - **In**-order : racine **au milieu** (entre les enfants)
> - **Post**-order : racine en **dernier** (apres les enfants)

### 5.1 Parcours Pre-order (Racine -> Gauche -> Droite)

```c
void binary_tree_preorder(const binary_tree_t *tree, void (*func)(int))
{
    if (tree == NULL || func == NULL)
        return;

    func(tree->n);                          /* 1. Traiter la racine */
    binary_tree_preorder(tree->left, func);  /* 2. Parcourir gauche */
    binary_tree_preorder(tree->right, func); /* 3. Parcourir droite */
}
```

### 5.2 Parcours In-order (Gauche -> Racine -> Droite)

> [!example] Propriete fondamentale
> Sur un **BST**, le parcours in-order donne les valeurs dans l'**ordre croissant**.

```c
void binary_tree_inorder(const binary_tree_t *tree, void (*func)(int))
{
    if (tree == NULL || func == NULL)
        return;

    binary_tree_inorder(tree->left, func);   /* 1. Parcourir gauche */
    func(tree->n);                           /* 2. Traiter la racine */
    binary_tree_inorder(tree->right, func);  /* 3. Parcourir droite */
}
```

### 5.3 Parcours Post-order (Gauche -> Droite -> Racine)

> [!tip] Utile pour la suppression
> Le post-order est utilise pour supprimer un arbre car on traite les enfants **avant** le parent.

```c
void binary_tree_postorder(const binary_tree_t *tree, void (*func)(int))
{
    if (tree == NULL || func == NULL)
        return;

    binary_tree_postorder(tree->left, func);  /* 1. Parcourir gauche */
    binary_tree_postorder(tree->right, func); /* 2. Parcourir droite */
    func(tree->n);                            /* 3. Traiter la racine */
}
```

### 5.4 Parcours par niveaux (Level-order / BFS)

Ce parcours utilise une **file d'attente (queue)** au lieu de la recursion.

```
Arbre :        (50)
              /    \
           (30)    (70)
           /  \
        (20) (40)

Level-order : 50, 30, 70, 20, 40
```

```c
/* Principe du BFS (Breadth-First Search) :
 * 1. Enfiler la racine
 * 2. Tant que la file n'est pas vide :
 *    a. Defiler un noeud
 *    b. Le traiter
 *    c. Enfiler ses enfants (gauche puis droite)
 */
void binary_tree_levelorder(const binary_tree_t *tree, void (*func)(int))
{
    /* Necessite une implementation de file (queue) */
    /* avec enqueue() et dequeue() */
    queue_t *queue;
    const binary_tree_t *current;

    if (tree == NULL || func == NULL)
        return;

    queue = queue_create();
    enqueue(queue, tree);

    while (!queue_is_empty(queue))
    {
        current = dequeue(queue);
        func(current->n);

        if (current->left)
            enqueue(queue, current->left);
        if (current->right)
            enqueue(queue, current->right);
    }
    queue_destroy(queue);
}
```

### Resume des parcours

```
Arbre :        (098)
              /     \
          (012)     (402)
          /   \     /   \
       (006)(016)(256)(512)

Pre-order  : 098, 012, 006, 016, 402, 256, 512
In-order   : 006, 012, 016, 098, 256, 402, 512
Post-order : 006, 016, 012, 256, 512, 402, 098
Level-order: 098, 012, 402, 006, 016, 256, 512
```

---

## 6. Arbre Binaire de Recherche (BST)

### Proprietes

Pour tout noeud N dans un BST :
- Tous les noeuds du sous-arbre **gauche** ont une valeur **< N**
- Tous les noeuds du sous-arbre **droit** ont une valeur **> N**

### Insertion dans un BST

```c
bst_t *bst_insert(bst_t **tree, int value)
{
    bst_t *new_node, *current;

    if (tree == NULL)
        return (NULL);

    if (*tree == NULL)
    {
        *tree = binary_tree_node(NULL, value);
        return (*tree);
    }

    current = *tree;
    while (current)
    {
        if (value < current->n)
        {
            if (current->left == NULL)
            {
                new_node = binary_tree_node(current, value);
                current->left = new_node;
                return (new_node);
            }
            current = current->left;
        }
        else if (value > current->n)
        {
            if (current->right == NULL)
            {
                new_node = binary_tree_node(current, value);
                current->right = new_node;
                return (new_node);
            }
            current = current->right;
        }
        else /* Doublon : ne pas inserer */
            return (NULL);
    }
    return (NULL);
}
```

### Recherche dans un BST

```c
bst_t *bst_search(const bst_t *tree, int value)
{
    if (tree == NULL)
        return (NULL);

    if (value == tree->n)
        return ((bst_t *)tree);
    else if (value < tree->n)
        return (bst_search(tree->left, value));
    else
        return (bst_search(tree->right, value));
}
```

> [!example] Complexite BST
> | Operation | Cas moyen | Pire cas (degenere) |
> |-----------|-----------|---------------------|
> | Recherche | O(log n) | O(n) |
> | Insertion | O(log n) | O(n) |
> | Suppression | O(log n) | O(n) |
>
> Le pire cas survient quand l'arbre est degenere (tous les noeuds en ligne = liste chainee).

---

## 7. Diagramme recapitulatif

```
                    Arbre Binaire
                    /           \
            Simple              Avec regles
           (pas d'ordre)        /     |     \
                              BST    AVL    Heap
                          (ordre)  (equilibre) (priorite)
                                     |
                              BST + auto-equilibrage
                              (|h_g - h_d| <= 1)
```

---

## 8. Exercices

> [!example] Exercice 1 : Construction a la main
> Dessine l'arbre obtenu en inserant les valeurs suivantes dans un BST vide, dans cet ordre : 50, 30, 70, 20, 40, 60, 80.
> Puis ecris le resultat de chaque parcours (pre, in, post, level).

> [!example] Exercice 2 : Implementer `binary_tree_depth`
> Ecris une fonction qui retourne la profondeur d'un noeud donne.
> Prototype : `size_t binary_tree_depth(const binary_tree_t *tree);`
> Indice : remonte vers le parent jusqu'a la racine en comptant.

> [!example] Exercice 3 : Verifier si un arbre est un BST
> Ecris une fonction qui verifie si un arbre binaire est un BST valide.
> Prototype : `int binary_tree_is_bst(const binary_tree_t *tree);`
> Indice : utilise une approche avec min/max pour chaque sous-arbre.

> [!example] Exercice 4 : Miroir d'un arbre
> Ecris une fonction qui echange les enfants gauche et droit de chaque noeud.
> Prototype : `binary_tree_t *binary_tree_mirror(binary_tree_t *tree);`

---

## 9. Liens

- [[01 - Listes Chainees]] - Structure lineaire de base (comparaison avec les arbres)
- [[02 - Listes Doublement Chainees]] - Double chainage similaire au pointeur parent
- [[07 - Recursion]] - Les arbres reposent entierement sur la recursion
- [[04 - Pointeurs et Memoire]] - Manipulation des pointeurs et malloc
- [[05 - Tri et Complexite Algorithmique]] - Comprendre O(log n) vs O(n)

---

> [!info] Rappels pour le projet Holberton
> - Style **Betty** obligatoire : max 40 lignes par fonction, declarations en haut
> - Compilation : `gcc -Wall -Werror -Wextra -pedantic -std=gnu89`
> - Toujours verifier le retour de `malloc`
> - Chaque fichier se termine par une nouvelle ligne
> - Maximum 5 fonctions par fichier
