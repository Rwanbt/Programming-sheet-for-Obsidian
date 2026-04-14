# Tri et Complexite Algorithmique

> [!info] Contexte
> Comprendre comment trier des donnees et mesurer l'efficacite d'un algorithme est ce qui separe un "codeur" d'un ingenieur logiciel. La notation Big O est le langage universel pour discuter de la performance des algorithmes.

---

## 1. La Notation Big O

La notation Big O mesure **comment le temps d'execution augmente** quand la taille des donnees d'entree (n) augmente. On ne garde que le **terme dominant** et on ignore les constantes.

### Regles de simplification

- O(n + 10) devient **O(n)**
- O(3n^2 + 5n) devient **O(n^2)**
- O(500) devient **O(1)**
- On ne garde que le terme **le plus lourd**

### Tableau des complexites

| Notation | Nom | Performance | Exemple typique |
|----------|-----|-------------|-----------------|
| O(1) | Constante | Parfaite | Acceder a un index d'un tableau |
| O(log n) | Logarithmique | Excellente | Recherche dichotomique (Binary Search) |
| O(n) | Lineaire | Bonne | Parcourir une liste une seule fois |
| O(n log n) | Lineaire-log | Tres bonne | Quick Sort, Merge Sort (en moyenne) |
| O(n^2) | Quadratique | Faible | Bubble Sort, Insertion Sort (boucles imbriquees) |
| O(2^n) | Exponentielle | Terrible | Sous-ensembles, recursion double naive |
| O(n!) | Factorielle | Catastrophique | Probleme du voyageur de commerce |

### Analogies du monde reel

```
O(1)      : Trouver ton nom sur ta carte d'identite
O(log n)  : Chercher un mot dans un dictionnaire (on divise par 2 a chaque fois)
O(n)      : Lire un livre page par page
O(n log n): Trier un paquet de cartes efficacement
O(n^2)    : Comparer chaque eleve avec chaque autre eleve de la classe
O(2^n)    : Essayer toutes les combinaisons d'un mot de passe
```

### Courbe de croissance

```
Temps
 ^
 |                                          * O(n^2)
 |                                     *
 |                                *
 |                          *
 |                    *                   * O(n log n)
 |              *                    *
 |         *                   *
 |     *                 *            * O(n)
 |   *             *             *
 |  *         *           *
 | *     *         *
 |*  *      *                         * O(log n)
 |* *  *  *  *  *  *  *  *  *  *  *
 +-----------------------------------------> n
```

---

## 2. Comment analyser la complexite

### Methode : compter les operations

> [!tip] Les 5 regles d'or
> 1. **Pas de boucle** = O(1)
> 2. **Une boucle de 0 a n** = O(n)
> 3. **Boucle qui double/divise** (i *= 2 ou i /= 2) = O(log n)
> 4. **Boucles imbriquees** = multiplier les complexites
> 5. **Ignorer les constantes** : O(n/2) = O(n), O(3n) = O(n)

### Exemples pratiques (quiz Holberton)

```c
/* Exemple 1 : O(1) - Pas de boucle */
void f(int n)
{
    printf("n = %d\n", n);
}

/* Exemple 2 : O(n) - Une boucle simple */
void f(int n)
{
    int i;
    for (i = 0; i < n; i++)
        printf("[%d]\n", i);
}

/* Exemple 3 : O(n) - Le pas de 98 ne change rien ! */
void f(int n)
{
    int i;
    for (i = 0; i < n; i += 98)  /* n/98 iterations = O(n) */
        printf("[%d]\n", i);
}

/* Exemple 4 : O(log n) - i double a chaque iteration */
void f(unsigned int n)
{
    int i;
    for (i = 1; i < n; i = i * 2)
        printf("[%d]\n", i);
}

/* Exemple 5 : O(n) - Recursion lineaire */
int factorial(int n)
{
    if (n == 0)
        return 1;
    return n * factorial(n - 1);  /* n appels recursifs */
}

/* Exemple 6 : O(n log n) - Boucle externe O(n) x interne O(log n) */
void f(unsigned int n)
{
    int i, j;
    for (i = 0; i < n; i++)           /* O(n) */
    {
        for (j = 1; j < n; j = j * 2) /* O(log n) */
            printf("[%d] [%d]\n", i, j);
    }
}

/* Exemple 7 : O(n^2) - Deux boucles imbriquees */
void f(int n)
{
    int i, j;
    for (i = 0; i < n; i++)
    {
        for (j = 0; j < n; j++)
            printf("[%d] [%d]\n", i, j);
    }
}
```

> [!warning] Piege courant
> Une boucle qui avance par pas de 98 (`i += 98`) reste O(n), pas O(n/98).
> En Big O on ignore les constantes multiplicatives.

---

## 3. Algorithmes de Tri

### 3.1 Bubble Sort (Tri a bulle) - O(n^2)

**Principe** : Comparer chaque paire d'elements adjacents et les echanger s'ils sont dans le mauvais ordre. Repeter jusqu'a ce qu'aucun echange ne soit necessaire.

```c
/**
 * bubble_sort - Trie un tableau par la methode du tri a bulle.
 * @array: Le tableau a trier.
 * @size: La taille du tableau.
 */
void bubble_sort(int *array, size_t size)
{
    size_t i, j;
    int temp;
    int swapped;

    if (array == NULL || size < 2)
        return;

    for (i = 0; i < size - 1; i++)
    {
        swapped = 0;
        for (j = 0; j < size - 1 - i; j++)
        {
            if (array[j] > array[j + 1])
            {
                temp = array[j];
                array[j] = array[j + 1];
                array[j + 1] = temp;
                swapped = 1;
                print_array(array, size);
            }
        }
        if (swapped == 0) /* Optimisation : deja trie */
            break;
    }
}
```

| Cas | Complexite |
|-----|-----------|
| Meilleur (deja trie) | O(n) |
| Moyen | O(n^2) |
| Pire | O(n^2) |
| Stable ? | Oui |

### 3.2 Insertion Sort (Tri par insertion) - O(n^2)

**Principe** : Comme trier des cartes dans sa main. On prend chaque element et on l'insere a sa place dans la partie deja triee.

```c
/**
 * insertion_sort_list - Trie une liste doublement chainee
 *                       par insertion.
 * @list: Double pointeur vers la tete de la liste.
 */
void insertion_sort_list(listint_t **list)
{
    listint_t *current, *insert, *temp;

    if (list == NULL || *list == NULL || (*list)->next == NULL)
        return;

    current = (*list)->next;
    while (current != NULL)
    {
        temp = current->next;
        insert = current;

        while (insert->prev && insert->n < insert->prev->n)
        {
            /* Echange des noeuds dans la liste */
            insert->prev->next = insert->next;
            if (insert->next)
                insert->next->prev = insert->prev;

            insert->next = insert->prev;
            insert->prev = insert->prev->prev;
            insert->next->prev = insert;

            if (insert->prev)
                insert->prev->next = insert;
            else
                *list = insert;

            print_list((const listint_t *)*list);
        }
        current = temp;
    }
}
```

| Cas | Complexite |
|-----|-----------|
| Meilleur (deja trie) | O(n) |
| Moyen | O(n^2) |
| Pire (tri inverse) | O(n^2) |
| Stable ? | Oui |

### 3.3 Selection Sort (Tri par selection) - O(n^2)

**Principe** : Chercher le plus petit element et le placer en premiere position, puis le deuxieme plus petit, etc.

```c
/**
 * selection_sort - Trie un tableau par selection.
 * @array: Le tableau a trier.
 * @size: La taille du tableau.
 */
void selection_sort(int *array, size_t size)
{
    size_t i, j, min_idx;
    int temp;

    if (array == NULL || size < 2)
        return;

    for (i = 0; i < size - 1; i++)
    {
        min_idx = i;
        for (j = i + 1; j < size; j++)
        {
            if (array[j] < array[min_idx])
                min_idx = j;
        }
        if (min_idx != i)
        {
            temp = array[i];
            array[i] = array[min_idx];
            array[min_idx] = temp;
            print_array(array, size);
        }
    }
}
```

| Cas | Complexite |
|-----|-----------|
| Tous les cas | O(n^2) |
| Stable ? | Non |

> [!warning] Selection Sort est TOUJOURS O(n^2)
> Meme si le tableau est deja trie, il scanne tout le reste a chaque iteration pour trouver le minimum.

### 3.4 Quick Sort (Tri rapide) - O(n log n) moyen

**Principe** : Choisir un "pivot", placer tous les elements plus petits a gauche et les plus grands a droite, puis repeter recursivement.

```c
/**
 * partition - Partitionne le tableau autour d'un pivot (Lomuto).
 * @array: Le tableau.
 * @low: Indice de debut.
 * @high: Indice de fin (pivot).
 * @size: Taille totale (pour l'affichage).
 * Return: L'indice final du pivot.
 */
int partition(int *array, int low, int high, size_t size)
{
    int pivot = array[high]; /* Pivot = dernier element */
    int i = low - 1;
    int j, temp;

    for (j = low; j < high; j++)
    {
        if (array[j] <= pivot)
        {
            i++;
            if (i != j)
            {
                temp = array[i];
                array[i] = array[j];
                array[j] = temp;
                print_array(array, size);
            }
        }
    }
    if (i + 1 != high)
    {
        temp = array[i + 1];
        array[i + 1] = array[high];
        array[high] = temp;
        print_array(array, size);
    }
    return (i + 1);
}

/**
 * quick_sort_rec - Fonction recursive du Quick Sort.
 * @array: Le tableau.
 * @low: Indice de debut.
 * @high: Indice de fin.
 * @size: Taille totale.
 */
void quick_sort_rec(int *array, int low, int high, size_t size)
{
    int pivot_idx;

    if (low < high)
    {
        pivot_idx = partition(array, low, high, size);
        quick_sort_rec(array, low, pivot_idx - 1, size);
        quick_sort_rec(array, pivot_idx + 1, high, size);
    }
}

/**
 * quick_sort - Point d'entree du Quick Sort.
 * @array: Le tableau a trier.
 * @size: La taille du tableau.
 */
void quick_sort(int *array, size_t size)
{
    if (array == NULL || size < 2)
        return;

    quick_sort_rec(array, 0, (int)size - 1, size);
}
```

| Cas | Complexite |
|-----|-----------|
| Meilleur | O(n log n) |
| Moyen | O(n log n) |
| Pire (pivot toujours min/max) | O(n^2) |
| Stable ? | Non |

### 3.5 Merge Sort (Tri fusion) - O(n log n)

**Principe** : Diviser le tableau en deux moities, trier chaque moitie recursivement, puis fusionner les deux moities triees.

```c
/**
 * merge - Fusionne deux sous-tableaux tries.
 * @array: Tableau original.
 * @temp: Tableau temporaire.
 * @left: Indice de debut.
 * @mid: Indice du milieu.
 * @right: Indice de fin.
 */
void merge(int *array, int *temp, int left, int mid, int right)
{
    int i = left, j = mid + 1, k = left;

    while (i <= mid && j <= right)
    {
        if (array[i] <= array[j])
            temp[k++] = array[i++];
        else
            temp[k++] = array[j++];
    }
    while (i <= mid)
        temp[k++] = array[i++];
    while (j <= right)
        temp[k++] = array[j++];

    for (i = left; i <= right; i++)
        array[i] = temp[i];
}

/**
 * merge_sort_rec - Recursion du Merge Sort.
 */
void merge_sort_rec(int *array, int *temp, int left, int right)
{
    int mid;

    if (left < right)
    {
        mid = (left + right) / 2;
        merge_sort_rec(array, temp, left, mid);
        merge_sort_rec(array, temp, mid + 1, right);
        merge(array, temp, left, mid, right);
    }
}
```

| Cas | Complexite |
|-----|-----------|
| Tous les cas | O(n log n) |
| Espace supplementaire | O(n) |
| Stable ? | Oui |

---

## 4. Stabilite des algorithmes de tri

> [!info] Definition
> Un algorithme est **stable** si deux elements ayant la **meme valeur** conservent leur **ordre relatif** apres le tri. C'est crucial quand on trie des objets complexes (ex: trier des etudiants par note, puis par nom).

| Algorithme | Stable ? |
|-----------|---------|
| Bubble Sort | Oui |
| Insertion Sort | Oui |
| Selection Sort | Non |
| Quick Sort | Non |
| Merge Sort | Oui |

---

## 5. Comparaison : Quand utiliser quoi ?

| Situation | Algorithme recommande | Raison |
|-----------|----------------------|--------|
| Petit ensemble (< 50) | Insertion Sort | Rapide en pratique, peu d'overhead |
| Donnees presque triees | Insertion Sort ou Bubble Sort | O(n) dans le meilleur cas |
| Grandes donnees | Quick Sort | O(n log n) moyen, en place |
| Garantie O(n log n) | Merge Sort | Jamais de pire cas quadratique |
| Memoire limitee | Quick Sort | Tri en place (pas de tableau supplementaire) |
| Stabilite requise | Merge Sort ou Insertion Sort | Conservent l'ordre relatif |

---

## 6. Mesurer la performance avec clock()

```c
#include <stdio.h>
#include <time.h>

int main(void)
{
    clock_t start, end;
    double elapsed;
    int i;

    start = clock();

    /* Code a mesurer */
    for (i = 0; i < 100000000; i++)
        ; /* boucle vide */

    end = clock();

    elapsed = (double)(end - start) / (double)CLOCKS_PER_SEC;
    printf("Temps d'execution : %.6f secondes\n", elapsed);

    return (0);
}
```

> [!warning] Variabilite des mesures
> Lancer le meme programme 3 fois donnera 3 temps differents car :
> 1. **Partage des ressources** : le CPU gere d'autres processus en parallele
> 2. **Gestion thermique** : le CPU ralentit s'il chauffe trop (throttling)
> 3. **Mise en cache** : la 2eme execution peut etre plus rapide (donnees en cache)
>
> Toujours faire **plusieurs mesures** et prendre la moyenne.

### La commande `time` sous Linux

```bash
$ time ./mon_programme
real    0m0.234s    # Temps total "montre en main"
user    0m0.220s    # Temps CPU passe sur ton code
sys     0m0.010s    # Temps CPU pour les appels systeme
```

> [!tip] `volatile` pour empecher l'optimisation
> ```c
> volatile unsigned long long result = 0;
> ```
> Le mot-cle `volatile` empeche le compilateur de supprimer la boucle
> s'il detecte que `result` n'est jamais lue apres.

---

## 7. Le fichier header sort.h

```c
#ifndef SORT_H
#define SORT_H

#include <stdlib.h>
#include <stdio.h>

/**
 * struct listint_s - Noeud de liste doublement chainee
 * @n: Entier stocke dans le noeud
 * @prev: Pointeur vers l'element precedent
 * @next: Pointeur vers l'element suivant
 */
typedef struct listint_s
{
    const int n;
    struct listint_s *prev;
    struct listint_s *next;
} listint_t;

/* Fonctions fournies */
void print_array(const int *array, size_t size);
void print_list(const listint_t *list);

/* Prototypes des algorithmes de tri */
void bubble_sort(int *array, size_t size);
void insertion_sort_list(listint_t **list);
void selection_sort(int *array, size_t size);
void quick_sort(int *array, size_t size);

#endif /* SORT_H */
```

---

## 8. Fichiers de complexite Big O

> [!info] Format Holberton
> Pour chaque algorithme, vous devez creer un fichier texte contenant la complexite dans le format suivant (une ligne, avec retour a la ligne final) :
> ```
> O(n^2)
> ```

Exemple pour Bubble Sort, le fichier `0-O` contient :
```
O(n^2)     <- Meilleur cas : O(n)
O(n^2)     <- Cas moyen
O(n^2)     <- Pire cas
```

---

## 9. Exercices

> [!example] Exercice 1 : Analyser la complexite
> Quelle est la complexite de cette fonction ?
> ```c
> void f(int n)
> {
>     int i, j;
>     for (i = 0; i < n; i++)
>     {
>         if (i % 2 == 0)
>             for (j = 1; j < n; j = j * 2)
>                 printf("[%d] [%d]\n", i, j);
>         else
>             for (j = 0; j < n; j = j + 2)
>                 printf("[%d] [%d]\n", i, j);
>     }
> }
> ```
> Reponse : La boucle externe est O(n). La boucle interne est O(log n) pour les pairs et O(n) pour les impairs. Le pire des deux domine : O(n) x O(n) = **O(n^2)**.

> [!example] Exercice 2 : Implementer le Merge Sort
> Implementez un Merge Sort complet pour un tableau d'entiers en utilisant le schema de decoupe "top-down". Affichez les sous-tableaux gauche et droit a chaque fusion.

> [!example] Exercice 3 : Comparer les performances
> Ecrivez un programme qui genere un tableau de 10 000 entiers aleatoires, le copie, puis le trie avec Bubble Sort et Quick Sort. Mesurez le temps de chaque tri avec `clock()` et comparez.

> [!example] Exercice 4 : Tri stable ?
> Vous avez un tableau de paires (nom, note) : `[("Alice", 15), ("Bob", 12), ("Clara", 15), ("David", 12)]`.
> Apres un tri stable par note, quel est l'ordre ? Et apres un tri instable ?

---

## 10. Liens

- [[04 - Arbres Binaires]] - Les BST utilisent la complexite O(log n)
- [[01 - Listes Chainees]] - Structure listint_t pour Insertion Sort
- [[02 - Listes Doublement Chainees]] - Manipulation de la liste dans le tri
- [[07 - Recursion]] - Quick Sort et Merge Sort sont recursifs
- [[01 - Introduction au C et Compilation]] - Flags de compilation stricts
