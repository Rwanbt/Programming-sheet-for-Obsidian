# Tables de Hachage

---

## Introduction : C'est quoi une Table de Hachage ?

### Analogie : Le Vestiaire du Stade

Imagine le **vestiaire** d'un grand stade. Tu arrives avec ton manteau, et l'employe te donne un **numero** (par exemple, 42). Quand tu reviens, tu donnes le numero 42 et on te rend ton manteau **instantanement** -- pas besoin de chercher dans tout le vestiaire.

- **Ta cle** = ton nom ("Dupont")
- **La fonction de hachage** = la methode pour calculer le numero (42)
- **La valeur** = ton manteau
- **Le casier** = l'emplacement physique dans le vestiaire

> [!info] Definition
> Une **table de hachage** (hash table) est une structure de donnees qui associe des **cles** a des **valeurs**, avec un acces en **O(1)** en moyenne.

---

## Concepts Cles

### Le Flux Complet

```
            FONCTION DE
     CLE    HACHAGE        INDEX    VALEUR
      |        |             |        |
      v        v             v        v
  "chat" --> hash() --> 5 --> table[5] --> "miaou"
```

### SET (Stocker une valeur)

```
SET("chat", "miaou")

  1. hash("chat") = 4837221
  2. index = 4837221 % taille_table = 5
  3. table[5] = {"chat", "miaou"}

  table:
  +---+
  | 0 | -> NULL
  | 1 | -> NULL
  | 2 | -> NULL
  | 3 | -> NULL
  | 4 | -> NULL
  | 5 | -> ["chat":"miaou"] -> NULL
  | 6 | -> NULL
  +---+
```

### GET (Recuperer une valeur)

```
GET("chat")

  1. hash("chat") = 4837221
  2. index = 4837221 % taille_table = 5
  3. Chercher dans table[5] un noeud avec cle == "chat"
  4. Trouve ! -> retourner "miaou"
```

---

## Les 2 Structures

### Le noeud de la table

```c
/**
 * struct hash_node_s - noeud d'une table de hachage
 * @key: la cle (chaine de caracteres)
 * @value: la valeur associee
 * @next: pointeur vers le noeud suivant (pour les collisions)
 */
typedef struct hash_node_s
{
    char *key;
    char *value;
    struct hash_node_s *next;
} hash_node_t;
```

### La table elle-meme

```c
/**
 * struct hash_table_s - la table de hachage
 * @size: taille du tableau (nombre de "casiers")
 * @array: tableau de pointeurs vers des listes chainees
 */
typedef struct hash_table_s
{
    unsigned long int size;
    hash_node_t **array;
} hash_table_t;
```

> [!info] Pourquoi `hash_node_t **array` et pas `hash_node_t *array` ?
> `array` est un **tableau de pointeurs**. Chaque case `array[i]` est un `hash_node_t *` qui pointe vers le debut d'une liste chainee (ou `NULL` si la case est vide).

---

## Visualisation en Memoire

```
hash_table_t
+--------+
| size=7 |
| array  |---> +---+
+--------+     | 0 | -> NULL
               | 1 | -> ["age":"25"] -> NULL
               | 2 | -> NULL
               | 3 | -> ["nom":"Alice"] -> ["ville":"Paris"] -> NULL
               | 4 | -> NULL                   (collision !)
               | 5 | -> ["chat":"miaou"] -> NULL
               | 6 | -> NULL
               +---+

Chaque case est la tete d'une liste chainee.
Les collisions sont gerees par chainage.
```

---

## La Fonction de Hachage

### Proprietes essentielles

> [!tip] Une bonne fonction de hachage doit etre :
> 1. **Deterministe** : la meme cle donne TOUJOURS le meme resultat
> 2. **Rapide** : le calcul doit etre en O(longueur de la cle)
> 3. **Uniforme** : distribuer les cles le plus equitablement possible
> 4. **Produire un index valide** : resultat entre 0 et (taille - 1)

### Fonction simple : somme ASCII

```c
/**
 * hash_simple - fonction de hachage simple (somme ASCII)
 * @key: la cle a hacher
 *
 * Return: la valeur de hachage
 */
unsigned long int hash_simple(const unsigned char *key)
{
    unsigned long int hash = 0;

    while (*key != '\0')
    {
        hash += *key;
        key++;
    }

    return (hash);
}
```

> [!warning] Probleme de la somme simple
> `"abc"` et `"bac"` et `"cab"` donnent **le meme hash** ! (97+98+99 = 294 pour les trois). Trop de collisions.

### Fonction djb2 (la reference Holberton)

```c
/**
 * hash_djb2 - fonction de hachage djb2
 * @str: la cle a hacher
 *
 * Return: la valeur de hachage
 *
 * Cette fonction est l'oeuvre de Dan Bernstein.
 * Les nombres magiques : 5381 (graine initiale) et 33 (multiplicateur).
 */
unsigned long int hash_djb2(const unsigned char *str)
{
    unsigned long int hash = 5381;
    int c;

    while ((c = *str++) != '\0')
    {
        hash = ((hash << 5) + hash) + c;  /* hash * 33 + c */
    }

    return (hash);
}
```

> [!info] Pourquoi 5381 et 33 ?
> - `5381` est un nombre premier qui donne de bonnes distributions empiriquement.
> - `hash * 33` = `hash << 5 + hash` (decalage de 5 bits + addition, tres rapide).
> - L'expression `hash << 5` est equivalente a `hash * 32`, donc `(hash << 5) + hash` = `hash * 33`.
> - Ces valeurs ont ete trouvees par experimentation et donnent peu de collisions.

### La fonction `key_index`

Pour obtenir un index valide dans le tableau :

```c
/**
 * key_index - calcule l'index dans le tableau a partir de la cle
 * @key: la cle
 * @size: taille du tableau
 *
 * Return: index (entre 0 et size - 1)
 */
unsigned long int key_index(const unsigned char *key, unsigned long int size)
{
    return (hash_djb2(key) % size);
}
```

> [!warning] Ne jamais oublier le modulo !
> `hash_djb2("chat")` peut donner un nombre enorme (ex: 2090578573). Sans `% size`, on depasse les limites du tableau -> **SEGFAULT** !

---

## Les Collisions

### Le Principe des Tiroirs (Pigeonhole Principle)

> [!info] Pourquoi les collisions sont inevitables
> Si tu as **10 casiers** et **15 manteaux**, au moins un casier aura **plus d'un manteau**. C'est mathematiquement inevitable.
>
> De meme, si ta table a 1024 cases et que tu peux avoir une infinite de cles possibles, plusieurs cles finiront dans la meme case.

### Resolution par Chainage (Chaining)

Chaque case du tableau contient une **liste chainee**. En cas de collision, on ajoute simplement le nouveau noeud en tete de la liste :

```
Collision : hash("nom") % 7 == hash("ville") % 7 == 3

  table[3] -> ["ville":"Paris"] -> ["nom":"Alice"] -> NULL
               (ajoute en 2e)      (ajoute en 1er)

Pour GET("nom") :
  1. index = hash("nom") % 7 = 3
  2. Parcourir la liste a table[3]
  3. Comparer : "ville" == "nom" ? NON -> suivant
  4. Comparer : "nom" == "nom" ? OUI -> retourner "Alice"
```

---

## Implementation Complete en C

### Creer la table : `hash_table_create()`

```c
/**
 * hash_table_create - cree une nouvelle table de hachage
 * @size: taille du tableau interne
 *
 * Return: pointeur vers la nouvelle table, ou NULL si echec
 */
hash_table_t *hash_table_create(unsigned long int size)
{
    hash_table_t *ht;
    unsigned long int i;

    ht = malloc(sizeof(hash_table_t));
    if (ht == NULL)
        return (NULL);

    ht->size = size;
    ht->array = malloc(sizeof(hash_node_t *) * size);
    if (ht->array == NULL)
    {
        free(ht);
        return (NULL);
    }

    /* Initialiser toutes les cases a NULL */
    for (i = 0; i < size; i++)
        ht->array[i] = NULL;

    return (ht);
}
```

### Inserer/Mettre a jour : `hash_table_set()`

```c
/**
 * hash_table_set - ajoute ou met a jour un element
 * @ht: la table de hachage
 * @key: la cle (ne peut pas etre vide)
 * @value: la valeur a associer (peut etre vide)
 *
 * Return: 1 si succes, 0 si echec
 */
int hash_table_set(hash_table_t *ht, const char *key, const char *value)
{
    unsigned long int index;
    hash_node_t *new_node;
    hash_node_t *current;
    char *value_copy;

    if (ht == NULL || key == NULL || *key == '\0')
        return (0);

    index = key_index((const unsigned char *)key, ht->size);

    /* Verifier si la cle existe deja -> mettre a jour */
    current = ht->array[index];
    while (current != NULL)
    {
        if (strcmp(current->key, key) == 0)
        {
            /* Cle trouvee : mettre a jour la valeur */
            value_copy = strdup(value);
            if (value_copy == NULL)
                return (0);
            free(current->value);  /* Liberer l'ancienne valeur */
            current->value = value_copy;
            return (1);
        }
        current = current->next;
    }

    /* Cle non trouvee : creer un nouveau noeud */
    new_node = malloc(sizeof(hash_node_t));
    if (new_node == NULL)
        return (0);

    new_node->key = strdup(key);
    if (new_node->key == NULL)
    {
        free(new_node);
        return (0);
    }

    new_node->value = strdup(value);
    if (new_node->value == NULL)
    {
        free(new_node->key);
        free(new_node);
        return (0);
    }

    /* Inserer en tete de la liste chainee a cet index */
    new_node->next = ht->array[index];
    ht->array[index] = new_node;

    return (1);
}
```

> [!warning] `strdup` est essentiel !
> On doit **copier** la cle et la valeur avec `strdup`. Si on stocke le pointeur original, et que l'appelant modifie ou libere sa chaine, notre table est corrompue !
>
> ```c
> /* MAUVAIS : stocker le pointeur original */
> new_node->key = key;      /* Si key est modifie dehors, BOOM ! */
>
> /* BON : faire une copie */
> new_node->key = strdup(key);  /* Copie independante */
> ```

### Recuperer : `hash_table_get()`

```c
/**
 * hash_table_get - recupere la valeur associee a une cle
 * @ht: la table de hachage
 * @key: la cle a chercher
 *
 * Return: la valeur, ou NULL si cle non trouvee
 */
char *hash_table_get(const hash_table_t *ht, const char *key)
{
    unsigned long int index;
    hash_node_t *current;

    if (ht == NULL || key == NULL || *key == '\0')
        return (NULL);

    index = key_index((const unsigned char *)key, ht->size);

    current = ht->array[index];
    while (current != NULL)
    {
        if (strcmp(current->key, key) == 0)
            return (current->value);
        current = current->next;
    }

    return (NULL);  /* Cle non trouvee */
}
```

> [!warning] Comparer avec `strcmp`, PAS avec `==` !
> ```c
> /* MAUVAIS : compare les adresses, pas le contenu ! */
> if (current->key == key)
>
> /* BON : compare le contenu des chaines */
> if (strcmp(current->key, key) == 0)
> ```
> `==` compare les **pointeurs** (adresses memoire). Deux chaines identiques a des adresses differentes seraient considerees comme differentes !

### Afficher : `hash_table_print()`

```c
/**
 * hash_table_print - affiche la table de hachage
 * @ht: la table de hachage
 */
void hash_table_print(const hash_table_t *ht)
{
    unsigned long int i;
    hash_node_t *current;
    int first = 1;

    if (ht == NULL)
        return;

    printf("{");
    for (i = 0; i < ht->size; i++)
    {
        current = ht->array[i];
        while (current != NULL)
        {
            if (!first)
                printf(", ");
            printf("'%s': '%s'", current->key, current->value);
            first = 0;
            current = current->next;
        }
    }
    printf("}\n");
}
```

### Detruire : `hash_table_delete()`

> [!warning] L'ordre de liberation est crucial !
> On doit liberer dans le bon ordre : d'abord le contenu des noeuds, puis les noeuds, puis le tableau, puis la table.

```c
/**
 * hash_table_delete - detruit une table de hachage
 * @ht: la table a detruire
 */
void hash_table_delete(hash_table_t *ht)
{
    unsigned long int i;
    hash_node_t *current;
    hash_node_t *next;

    if (ht == NULL)
        return;

    /* Pour chaque case du tableau */
    for (i = 0; i < ht->size; i++)
    {
        current = ht->array[i];
        /* Liberer chaque noeud de la liste chainee */
        while (current != NULL)
        {
            next = current->next;    /* 1. Sauvegarder le suivant */
            free(current->key);      /* 2. Liberer la cle */
            free(current->value);    /* 3. Liberer la valeur */
            free(current);           /* 4. Liberer le noeud */
            current = next;          /* 5. Passer au suivant */
        }
    }

    free(ht->array);  /* 6. Liberer le tableau de pointeurs */
    free(ht);          /* 7. Liberer la structure table */
}
```

```
Ordre de liberation :

Pour chaque case i :
  Pour chaque noeud de la liste :
    free(key)      \
    free(value)     > contenu du noeud
    free(noeud)    /

free(array)        <- le tableau de pointeurs
free(ht)           <- la structure principale

Si on fait free(ht) en premier, on perd l'acces a ht->array
et a tous les noeuds -> FUITE MEMOIRE MASSIVE !
```

---

## Analyse de Complexite

| Operation  | Cas moyen | Cas pire  | Explication du cas pire        |
|------------|-----------|-----------|--------------------------------|
| **SET**    | O(1)      | O(n)      | Toutes les cles au meme index  |
| **GET**    | O(1)      | O(n)      | Parcourir toute la liste chainee |
| **DELETE** | O(1)      | O(n)      | Idem                           |

> [!info] Pourquoi O(n) dans le pire cas ?
> Si la fonction de hachage est mauvaise et que **toutes les cles** tombent dans la **meme case**, la table degenerere en une simple liste chainee :
>
> ```
> table[3] -> [cle1] -> [cle2] -> [cle3] -> ... -> [cleN] -> NULL
>
> Pour trouver cleN, il faut parcourir N noeuds -> O(n)
> ```
>
> C'est pourquoi une bonne fonction de hachage et une taille de table adaptee sont essentielles.

---

## Comparaison : Table de Hachage vs Tableau vs Liste Chainee

| Critere            | Tableau     | Liste Chainee | Table de Hachage |
|--------------------|-------------|---------------|------------------|
| **Acces par index**| O(1)        | O(n)          | N/A              |
| **Acces par cle**  | O(n)        | O(n)          | O(1) moyen       |
| **Insertion**      | O(n)        | O(1) debut    | O(1) moyen       |
| **Suppression**    | O(n)        | O(1) avec ptr | O(1) moyen       |
| **Memoire**        | Compacte    | Dispersee     | Dispersee + overhead |
| **Ordre**          | Oui (index) | Oui (insertion)| Non garanti     |
| **Cle arbitraire** | Non (entier)| Non           | Oui (string)     |

> [!tip] Quand utiliser une table de hachage ?
> Quand tu as besoin d'associer des **cles** (souvent des chaines) a des **valeurs**, avec un acces rapide. Exemples : dictionnaire, cache, compteur de mots, configuration...

---

## Bonus : Table de Hachage Triee (shash_table)

Une table de hachage classique ne garantit **aucun ordre** lors de l'affichage. Pour maintenir un ordre alphabetique, on utilise une **table de hachage triee** (sorted hash table) :

### Principe

On ajoute une **liste doublement chainee** qui traverse tous les noeuds dans l'ordre alphabetique des cles :

```c
/**
 * struct shash_node_s - noeud d'une table de hachage triee
 * @key: la cle
 * @value: la valeur
 * @next: pointeur suivant dans la liste de collision
 * @sprev: pointeur vers l'element trie precedent
 * @snext: pointeur vers l'element trie suivant
 */
typedef struct shash_node_s
{
    char *key;
    char *value;
    struct shash_node_s *next;   /* chainage collision */
    struct shash_node_s *sprev;  /* chainage trie */
    struct shash_node_s *snext;  /* chainage trie */
} shash_node_t;

/**
 * struct shash_table_s - table de hachage triee
 * @size: taille du tableau
 * @array: tableau de listes (collisions)
 * @shead: tete de la liste triee
 * @stail: queue de la liste triee
 */
typedef struct shash_table_s
{
    unsigned long int size;
    shash_node_t **array;
    shash_node_t *shead;  /* Premier element dans l'ordre alphabetique */
    shash_node_t *stail;  /* Dernier element dans l'ordre alphabetique */
} shash_table_t;
```

```
Visualisation :

Table (acces par hash) :
+---+
| 0 | -> NULL
| 1 | -> ["chat":"miaou"] -> NULL
| 2 | -> NULL
| 3 | -> ["zoo":"animaux"] -> ["alice":"pays"] -> NULL
| 4 | -> NULL
+---+

Liste triee (acces ordonne) :
shead -> [alice] <=> [chat] <=> [zoo] -> stail

On a DEUX facons de naviguer :
1. Par hash (rapide, O(1))
2. Par ordre alphabetique (parcours, O(n) mais trie !)
```

> [!info] C'est le meilleur des deux mondes
> Acces en O(1) par cle + parcours dans l'ordre alphabetique. Le cout : plus de memoire (deux pointeurs supplementaires par noeud) et insertion plus lente (il faut trouver la bonne position dans la liste triee).

Pour en savoir plus sur les listes doublement chainees utilisees ici : [[02 - Listes Doublement Chainees]]

---

## Erreurs Courantes

> [!warning] Les pieges classiques des tables de hachage

### 1. Stocker le pointeur original au lieu de `strdup`

```c
/* MAUVAIS */
new_node->key = key;
new_node->value = value;
/* Si l'appelant fait free(key), notre table est corrompue ! */

/* BON */
new_node->key = strdup(key);
new_node->value = strdup(value);
```

### 2. Oublier le modulo

```c
/* MAUVAIS */
index = hash_djb2(key);        /* Peut valoir 4 milliards ! */
ht->array[index] = ...;        /* SEGFAULT : depassement de tableau */

/* BON */
index = hash_djb2(key) % ht->size;  /* Toujours entre 0 et size-1 */
```

### 3. Comparer des pointeurs au lieu du contenu

```c
/* MAUVAIS */
if (current->key == key)       /* Compare les ADRESSES */

/* BON */
if (strcmp(current->key, key) == 0)  /* Compare le CONTENU */
```

### 4. Mauvais ordre de liberation

```c
/* MAUVAIS */
free(ht);           /* On perd ht->array ! */
free(ht->array);    /* Acces a memoire liberee ! */
/* Et tous les noeuds sont perdus -> fuite memoire */

/* BON : de l'interieur vers l'exterieur */
/* 1. free chaque noeud (key, value, noeud) */
/* 2. free le tableau (array) */
/* 3. free la structure (ht) */
```

### 5. Ne pas gerer la mise a jour d'une cle existante

```c
/* MAUVAIS : inserer un doublon sans verifier */
/* hash_table_set("nom", "Alice")  puis  hash_table_set("nom", "Bob") */
/* Resultat : deux noeuds avec la cle "nom" ! */

/* BON : d'abord chercher si la cle existe, puis mettre a jour */
```

---

## Exercices Pratiques

1. **Ecris** un programme complet qui cree une table, insere 5 paires cle-valeur, en recupere quelques-unes, et libere tout proprement.
2. **Ecris** une fonction `hash_table_keys(hash_table_t *ht)` qui retourne un tableau de toutes les cles.
3. **Ecris** une fonction `count_words(char *text)` qui utilise une table de hachage pour compter les occurrences de chaque mot.
4. **Modifie** `hash_table_set` pour implementer un **facteur de charge** : si le nombre d'elements depasse 75% de la taille, redimensionner la table (rehashing).
5. **Implemente** la `shash_table` complete avec `shash_table_create`, `shash_table_set`, `shash_table_get`, `shash_table_print` (dans l'ordre), `shash_table_print_rev`, et `shash_table_delete`.

---

## Carte Mentale Complete

```
                      TABLE DE HACHAGE
                            |
          +-----------------+-----------------+
          |                 |                 |
      STRUCTURES        OPERATIONS        CONCEPTS
          |                 |                 |
  hash_table_t         +----+----+       HASH FUNCTION
  {size, **array}      |    |    |       - deterministe
          |          CREATE SET  GET      - rapide
  hash_node_t          |    |    |       - uniforme
  {key, value, *next}  |  DELETE PRINT   - djb2: 5381, *33+c
                        |              
                   COLLISIONS          COMPLEXITE
                        |                 |
                   Chainage          O(1) moyen
                   (linked list)     O(n) pire cas
                        |
                   Pigeonhole
                   Principle

      ERREURS COURANTES :
      - strdup vs pointeur original
      - oublier % size
      - == vs strcmp
      - ordre de free (interieur -> exterieur)
```

---

## Liens

- Precedent : [[02 - Listes Doublement Chainees]]
- Prerequis : [[01 - Listes Chainees]]
