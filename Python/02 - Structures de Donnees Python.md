# Structures de Donnees Python

## Qu'est-ce que les structures de donnees en Python ?

Les structures de donnees sont des **conteneurs** qui organisent et stockent des donnees de maniere efficace. Python offre quatre types de collections integres : les **listes**, les **tuples**, les **dictionnaires** et les **ensembles** (sets). Chacun a des proprietes et des cas d'utilisation distincts.

Contrairement au C, ou vous devez implementer manuellement les structures de donnees complexes (listes chainees, tables de hachage, arbres), Python les fournit **nativement** avec une syntaxe intuitive.

> [!tip] Analogie
> En C, construire une structure de donnees c'est comme fabriquer vos propres meubles a partir de planches de bois (malloc, pointeurs, gestion memoire). En Python, c'est comme acheter des meubles IKEA deja assembles : ils sont prets a l'emploi, optimises, et vous pouvez vous concentrer sur l'amenagement de la piece (la logique de votre programme).

---

## Les Listes (list)

Les listes sont des **sequences mutables ordonnees** d'elements. Elles sont l'equivalent Python d'un **tableau dynamique** en C (pensez a un tableau qui peut changer de taille automatiquement).

### Creation

```python
# Liste vide
vide = []
vide2 = list()

# Liste avec des elements
nombres = [1, 2, 3, 4, 5]
mixte = [1, "deux", 3.0, True, None]  # Types melanges (impossible en C !)
```

> [!info] Listes vs tableaux C
> En C, un tableau a un type fixe et une taille fixe : `int arr[5]`. En Python, une liste peut contenir des types differents et sa taille est dynamique. Sous le capot, une liste Python est un **tableau de pointeurs** vers des objets.

### Indexation et acces

```python
fruits = ["pomme", "banane", "cerise", "datte", "figue"]

# Indexation positive (comme en C)
print(fruits[0])   # "pomme"
print(fruits[2])   # "cerise"

# Indexation negative (pas d'equivalent en C !)
print(fruits[-1])  # "figue" (dernier element)
print(fruits[-2])  # "datte" (avant-dernier)

# Acces hors limites
# En C : comportement indefini (UB), potentiel segfault
# En Python : IndexError propre
# print(fruits[10])  # IndexError: list index out of range
```

```
Index positif :   0        1        2        3        4
                +--------+--------+--------+--------+--------+
                | pomme  | banane | cerise | datte  | figue  |
                +--------+--------+--------+--------+--------+
Index negatif :  -5       -4       -3       -2       -1
```

### Slicing (decoupe)

Le slicing est l'une des fonctionnalites les plus puissantes de Python, sans equivalent direct en C.

```python
nombres = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Syntaxe : liste[debut:fin:pas]
# debut : inclus (defaut: 0)
# fin   : exclus (defaut: len(liste))
# pas   : increment (defaut: 1)

print(nombres[2:5])     # [2, 3, 4]
print(nombres[:3])      # [0, 1, 2]
print(nombres[7:])      # [7, 8, 9]
print(nombres[::2])     # [0, 2, 4, 6, 8] (un sur deux)
print(nombres[1::2])    # [1, 3, 5, 7, 9] (les impairs)
print(nombres[::-1])    # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0] (inverse)
print(nombres[8:2:-1])  # [8, 7, 6, 5, 4, 3]

# Le slicing retourne une NOUVELLE liste (copie superficielle)
extrait = nombres[2:5]
extrait[0] = 99
print(nombres)  # [0, 1, 2, 3, ...] (inchange)
```

> [!tip] Analogie
> Le slicing, c'est comme decouper un gateau. `liste[2:5]` prend la tranche du 3e au 5e morceau. En C, il faudrait manuellement creer un nouveau tableau, calculer la taille, et copier les elements avec `memcpy()`.

### Modification de listes

```python
fruits = ["pomme", "banane", "cerise"]

# Modifier un element (comme en C : arr[1] = valeur)
fruits[1] = "mangue"
print(fruits)  # ["pomme", "mangue", "cerise"]

# Modifier une tranche
fruits[1:3] = ["kiwi", "orange", "fraise"]
print(fruits)  # ["pomme", "kiwi", "orange", "fraise"]
```

### Methodes de liste

#### Ajouter des elements

```python
fruits = ["pomme", "banane"]

# append : ajouter a la fin - O(1) amorti
fruits.append("cerise")
print(fruits)  # ["pomme", "banane", "cerise"]

# extend : ajouter plusieurs elements - O(k)
fruits.extend(["datte", "figue"])
print(fruits)  # ["pomme", "banane", "cerise", "datte", "figue"]

# insert : inserer a une position - O(n) (decalage)
fruits.insert(1, "abricot")
print(fruits)  # ["pomme", "abricot", "banane", "cerise", "datte", "figue"]

# Concatenation avec + (cree une nouvelle liste)
tous = fruits + ["goyave", "kiwi"]
```

#### Supprimer des elements

```python
fruits = ["pomme", "banane", "cerise", "banane", "datte"]

# remove : supprime la premiere occurrence - O(n)
fruits.remove("banane")
print(fruits)  # ["pomme", "cerise", "banane", "datte"]

# pop : supprime et retourne l'element a l'index - O(1) pour le dernier, O(n) sinon
dernier = fruits.pop()       # "datte"
deuxieme = fruits.pop(1)     # "cerise"
print(fruits)  # ["pomme", "banane"]

# del : supprime par index ou tranche
del fruits[0]
print(fruits)  # ["banane"]

# clear : vide la liste
fruits.clear()
print(fruits)  # []
```

#### Tri et inversion

```python
nombres = [3, 1, 4, 1, 5, 9, 2, 6]

# sort : tri EN PLACE (modifie la liste) - O(n log n)
nombres.sort()
print(nombres)  # [1, 1, 2, 3, 4, 5, 6, 9]

nombres.sort(reverse=True)
print(nombres)  # [9, 6, 5, 4, 3, 2, 1, 1]

# sorted : retourne une NOUVELLE liste triee (ne modifie pas l'original)
original = [3, 1, 4, 1, 5]
trie = sorted(original)
print(original)  # [3, 1, 4, 1, 5] (inchange)
print(trie)      # [1, 1, 3, 4, 5]

# Tri personnalise avec key
mots = ["banane", "Cerise", "abricot", "Datte"]
mots.sort(key=str.lower)  # Tri insensible a la casse
print(mots)  # ["abricot", "banane", "Cerise", "Datte"]

# Tri par longueur
mots.sort(key=len)
print(mots)  # ["Datte", "banane", "Cerise", "abricot"]

# reverse : inverse EN PLACE
nombres = [1, 2, 3, 4, 5]
nombres.reverse()
print(nombres)  # [5, 4, 3, 2, 1]

# reversed : retourne un iterateur inverse (ne modifie pas)
for n in reversed([1, 2, 3]):
    print(n)  # 3, 2, 1
```

> [!warning] sort() vs sorted()
> `sort()` modifie la liste en place et retourne `None`. `sorted()` cree une nouvelle liste. Ne faites pas `liste = liste.sort()` car cela affecterait `None` a `liste` !

#### Copie de listes

```python
original = [1, [2, 3], 4]

# Copie superficielle (shallow copy)
copie1 = original.copy()
copie2 = original[:]
copie3 = list(original)

# Attention : les objets imbriques sont partages !
copie1[1][0] = 99
print(original)  # [1, [99, 3], 4]  (modifie aussi !)

# Copie profonde (deep copy)
import copy
copie_profonde = copy.deepcopy(original)
copie_profonde[1][0] = 42
print(original)  # [1, [99, 3], 4]  (inchange)
```

```
Copie superficielle :
original ----> [  1,  [2, 3],  4  ]
                       ^
copie    ----> [  1,  [2, 3],  4  ]
                  |     |      |
              nouveau  MEME   nouveau
               objet  objet   objet

Copie profonde :
original ----> [  1,  [2, 3],  4  ]

copie    ----> [  1,  [2, 3],  4  ]
                  |     |      |
              nouveau nouveau nouveau
               objet  objet   objet
```

#### Autres methodes utiles

```python
nombres = [3, 1, 4, 1, 5, 9, 2, 6, 5]

print(nombres.count(1))   # 2 (nombre d'occurrences)
print(nombres.index(4))   # 2 (index de la premiere occurrence)
print(len(nombres))        # 9
print(sum(nombres))        # 36
print(min(nombres))        # 1
print(max(nombres))        # 9
print(5 in nombres)        # True
```

### Listes imbriquees (matrices)

```python
# En C : int matrice[3][3] = {{1,2,3}, {4,5,6}, {7,8,9}};
# En Python :
matrice = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

# Acces : matrice[ligne][colonne]
print(matrice[1][2])  # 6 (ligne 1, colonne 2)

# Parcours
for ligne in matrice:
    for element in ligne:
        print(element, end=" ")
    print()
# 1 2 3
# 4 5 6
# 7 8 9
```

### List Comprehensions

Les list comprehensions sont une syntaxe concise pour creer des listes. C'est l'une des fonctionnalites les plus elegantes de Python.

```python
# Syntaxe : [expression for variable in iterable if condition]

# Equivalent en C :
# int carres[10];
# for (int i = 0; i < 10; i++) carres[i] = i * i;

# En Python classique :
carres = []
for i in range(10):
    carres.append(i ** 2)

# Avec list comprehension :
carres = [i ** 2 for i in range(10)]
print(carres)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Avec condition (filtre)
pairs = [i for i in range(20) if i % 2 == 0]
print(pairs)  # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# Avec transformation conditionnelle
resultat = ["pair" if i % 2 == 0 else "impair" for i in range(5)]
print(resultat)  # ["pair", "impair", "pair", "impair", "pair"]

# Comprehension imbriquee (matrice)
matrice = [[i * j for j in range(1, 4)] for i in range(1, 4)]
print(matrice)  # [[1, 2, 3], [2, 4, 6], [3, 6, 9]]

# Aplatir une matrice
plat = [x for ligne in matrice for x in ligne]
print(plat)  # [1, 2, 3, 2, 4, 6, 3, 6, 9]
```

> [!example] Comparaison C vs Python
> ```c
> // C : filtrer les nombres pairs et les doubler
> int arr[10] = {0,1,2,3,4,5,6,7,8,9};
> int result[5];
> int j = 0;
> for (int i = 0; i < 10; i++) {
>     if (arr[i] % 2 == 0) {
>         result[j++] = arr[i] * 2;
>     }
> }
> ```
> ```python
> # Python : une seule ligne
> result = [x * 2 for x in range(10) if x % 2 == 0]
> # [0, 4, 8, 12, 16]
> ```

> [!warning] Lisibilite
> Les list comprehensions doivent rester lisibles. Si elles deviennent trop complexes (plus de 2 niveaux d'imbrication), utilisez plutot des boucles classiques.

---

## Les Tuples (tuple)

Les tuples sont des **sequences immutables ordonnees**. Une fois cree, un tuple ne peut pas etre modifie.

### Creation

```python
# Tuple avec parentheses
coordonnees = (48.8566, 2.3522)
couleur_rgb = (255, 128, 0)

# Tuple sans parentheses (packing)
point = 3, 4, 5

# Tuple d'un seul element (la virgule est obligatoire !)
singleton = (42,)       # C'est un tuple
pas_tuple = (42)        # C'est un int !

# Tuple vide
vide = ()
vide2 = tuple()

# Conversion depuis une liste
depuis_liste = tuple([1, 2, 3])
```

### Operations sur les tuples

```python
t = (1, 2, 3, 4, 5)

# Indexation et slicing (comme les listes)
print(t[0])      # 1
print(t[-1])     # 5
print(t[1:4])    # (2, 3, 4)

# Longueur
print(len(t))    # 5

# Recherche
print(3 in t)    # True
print(t.count(2)) # 1
print(t.index(3)) # 2

# Concatenation (cree un nouveau tuple)
t2 = t + (6, 7)  # (1, 2, 3, 4, 5, 6, 7)

# Repetition
t3 = (0,) * 5    # (0, 0, 0, 0, 0)

# IMMUTABLE : impossible de modifier
# t[0] = 99  # TypeError: 'tuple' object does not support item assignment
```

### Packing et Unpacking

```python
# Packing
coordonnees = 48.8566, 2.3522

# Unpacking
latitude, longitude = coordonnees
print(latitude)   # 48.8566
print(longitude)  # 2.3522

# Unpacking avec * (extended unpacking)
premier, *reste = [1, 2, 3, 4, 5]
print(premier)  # 1
print(reste)    # [2, 3, 4, 5]

*debut, dernier = [1, 2, 3, 4, 5]
print(debut)    # [1, 2, 3, 4]
print(dernier)  # 5

premier, *milieu, dernier = [1, 2, 3, 4, 5]
print(premier)  # 1
print(milieu)   # [2, 3, 4]
print(dernier)  # 5

# Echange de variables (unpacking implicite)
a, b = 1, 2
a, b = b, a  # a=2, b=1

# Unpacking dans une boucle
points = [(1, 2), (3, 4), (5, 6)]
for x, y in points:
    print(f"x={x}, y={y}")

# Ignorer des valeurs avec _
_, y, _ = (1, 2, 3)  # On ne garde que y
```

> [!tip] Analogie
> Un tuple est comme un enregistrement immuable. Pensez a une coordonnee GPS : une fois mesuree, elle ne change pas. En C, c'est comme un `const struct` : `const struct Point p = {3, 4};`.

### Quand utiliser un tuple vs une liste ?

| Critere | Liste | Tuple |
|---|---|---|
| Mutable | Oui | Non |
| Usage | Collection d'elements similaires | Groupe de valeurs heterogenes |
| Exemple | Liste d'etudiants | Coordonnees (x, y) |
| Performance | Legerement plus lent | Legerement plus rapide |
| Hashable | Non | Oui (si contenu hashable) |
| Cle de dict | Non | Oui |

### Named Tuples

Les named tuples permettent d'acceder aux elements par nom, comme un struct en C.

```python
from collections import namedtuple

# Definition (comme un struct en C)
# struct Point { int x; int y; };
Point = namedtuple("Point", ["x", "y"])

# Creation
p = Point(3, 4)
print(p.x)     # 3 (acces par nom)
print(p[0])    # 3 (acces par index)
print(p)       # Point(x=3, y=4)

# Immutable
# p.x = 10  # AttributeError

# Avec valeurs par defaut
Etudiant = namedtuple("Etudiant", ["nom", "age", "note"], defaults=[0.0])
e = Etudiant("Alice", 20)
print(e)  # Etudiant(nom='Alice', age=20, note=0.0)

# Conversion en dictionnaire
print(p._asdict())  # {'x': 3, 'y': 4}

# Creer une copie modifiee
p2 = p._replace(x=10)
print(p2)  # Point(x=10, y=4)
```

> [!example] Comparaison : struct C vs namedtuple Python
> ```c
> // C
> struct Point {
>     int x;
>     int y;
> };
> struct Point p = {3, 4};
> printf("x=%d, y=%d\n", p.x, p.y);
> ```
> ```python
> # Python
> from collections import namedtuple
> Point = namedtuple("Point", ["x", "y"])
> p = Point(3, 4)
> print(f"x={p.x}, y={p.y}")
> ```

---

## Les Dictionnaires (dict)

Les dictionnaires sont des **collections mutables non ordonnees** (ordonnees par insertion depuis Python 3.7) de paires **cle-valeur**. Ils sont l'equivalent Python d'une **table de hachage**.

### Creation

```python
# Avec des accolades
ages = {"Alice": 25, "Bob": 30, "Charlie": 35}

# Avec dict()
ages2 = dict(Alice=25, Bob=30, Charlie=35)

# Dictionnaire vide
vide = {}
vide2 = dict()

# Depuis une liste de tuples
paires = [("un", 1), ("deux", 2), ("trois", 3)]
nombres = dict(paires)

# Avec dict.fromkeys()
defauts = dict.fromkeys(["a", "b", "c"], 0)
print(defauts)  # {"a": 0, "b": 0, "c": 0}
```

> [!info] Cles de dictionnaire
> Les cles doivent etre **hashables** (immutables) : `str`, `int`, `float`, `tuple`, `frozenset`. Les listes et les dictionnaires ne peuvent **pas** etre des cles.

### Acces et modification

```python
ages = {"Alice": 25, "Bob": 30, "Charlie": 35}

# Acces
print(ages["Alice"])      # 25
# print(ages["David"])    # KeyError: 'David'

# Acces securise avec get()
print(ages.get("David"))        # None (pas d'erreur)
print(ages.get("David", 0))     # 0 (valeur par defaut)

# Modification / Ajout
ages["Alice"] = 26              # Modifier
ages["David"] = 28              # Ajouter

# Suppression
del ages["Charlie"]             # Supprime (KeyError si absent)
age_bob = ages.pop("Bob")       # Supprime et retourne la valeur
print(age_bob)  # 30
ages.pop("Inconnu", None)       # Pas d'erreur si absent

# setdefault : retourne la valeur si la cle existe, sinon l'insere
ages.setdefault("Eve", 22)      # Insere Eve: 22
ages.setdefault("Alice", 99)    # Ne modifie pas (Alice existe deja)
print(ages["Alice"])             # 26
```

### Parcours de dictionnaires

```python
ages = {"Alice": 25, "Bob": 30, "Charlie": 35}

# Parcours des cles (par defaut)
for nom in ages:
    print(nom)

# Parcours des cles (explicite)
for nom in ages.keys():
    print(nom)

# Parcours des valeurs
for age in ages.values():
    print(age)

# Parcours des paires cle-valeur
for nom, age in ages.items():
    print(f"{nom} a {age} ans")

# Verification d'existence
print("Alice" in ages)       # True
print("David" in ages)       # False
print(25 in ages.values())   # True
```

### Methodes de dictionnaire

```python
d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}

# update : fusionne (ecrase les cles existantes)
d1.update(d2)
print(d1)  # {"a": 1, "b": 3, "c": 4}

# Fusion avec | (Python 3.9+)
d3 = {"a": 1, "b": 2} | {"b": 3, "c": 4}
print(d3)  # {"a": 1, "b": 3, "c": 4}

# Copie
copie = d1.copy()  # Copie superficielle

# Longueur
print(len(d1))  # 3

# Vider
d1.clear()
```

### Dict Comprehensions

```python
# Syntaxe : {cle: valeur for variable in iterable if condition}

# Carres des nombres
carres = {i: i**2 for i in range(6)}
print(carres)  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# Filtrer un dictionnaire
ages = {"Alice": 25, "Bob": 17, "Charlie": 35, "David": 15}
majeurs = {nom: age for nom, age in ages.items() if age >= 18}
print(majeurs)  # {"Alice": 25, "Charlie": 35}

# Inverser cles et valeurs
inverse = {v: k for k, v in ages.items()}
print(inverse)  # {25: "Alice", 17: "Bob", 35: "Charlie", 15: "David"}
```

### Dictionnaires imbriques

```python
etudiants = {
    "Alice": {
        "age": 25,
        "notes": {"math": 15, "info": 18, "anglais": 14}
    },
    "Bob": {
        "age": 22,
        "notes": {"math": 12, "info": 16, "anglais": 11}
    }
}

# Acces
print(etudiants["Alice"]["notes"]["info"])  # 18

# Modification
etudiants["Alice"]["notes"]["math"] = 16

# Parcours
for nom, info in etudiants.items():
    moyenne = sum(info["notes"].values()) / len(info["notes"])
    print(f"{nom}: moyenne = {moyenne:.1f}")
```

### defaultdict et Counter

```python
from collections import defaultdict, Counter

# defaultdict : dictionnaire avec valeur par defaut
# En C, il faudrait verifier si la cle existe avant d'y acceder

# Compter les occurrences de chaque lettre
texte = "abracadabra"

# Sans defaultdict
compteur = {}
for lettre in texte:
    if lettre in compteur:
        compteur[lettre] += 1
    else:
        compteur[lettre] = 1

# Avec defaultdict
compteur = defaultdict(int)  # int() retourne 0 par defaut
for lettre in texte:
    compteur[lettre] += 1
print(dict(compteur))  # {"a": 5, "b": 2, "r": 2, "c": 1, "d": 1}

# Grouper des elements
etudiants = [("Alice", "A"), ("Bob", "B"), ("Charlie", "A"), ("David", "B")]
groupes = defaultdict(list)
for nom, classe in etudiants:
    groupes[classe].append(nom)
print(dict(groupes))  # {"A": ["Alice", "Charlie"], "B": ["Bob", "David"]}

# Counter : compteur specialise
compteur = Counter("abracadabra")
print(compteur)               # Counter({"a": 5, "b": 2, "r": 2, "c": 1, "d": 1})
print(compteur.most_common(3)) # [("a", 5), ("b", 2), ("r", 2)]

# Operations sur Counter
c1 = Counter(a=3, b=1)
c2 = Counter(a=1, b=2)
print(c1 + c2)  # Counter({"a": 4, "b": 3})
print(c1 - c2)  # Counter({"a": 2})
```

---

## Les Ensembles (set)

Les ensembles sont des **collections mutables non ordonnees d'elements uniques**. Ils sont implementes comme des tables de hachage (comme les dictionnaires, mais sans valeurs).

### Creation

```python
# Avec des accolades
fruits = {"pomme", "banane", "cerise"}

# Avec set()
nombres = set([1, 2, 3, 2, 1])
print(nombres)  # {1, 2, 3} (doublons supprimes)

# Ensemble vide (ATTENTION : {} cree un dict vide !)
vide = set()   # Correct
# vide = {}    # C'est un dict, pas un set !

# Depuis une chaine
lettres = set("abracadabra")
print(lettres)  # {"a", "b", "r", "c", "d"}
```

### Modification

```python
s = {1, 2, 3}

# Ajouter
s.add(4)          # {1, 2, 3, 4}
s.add(2)          # {1, 2, 3, 4} (pas de doublon)

# Supprimer
s.remove(3)       # {1, 2, 4} (KeyError si absent)
s.discard(10)     # Pas d'erreur si absent
element = s.pop() # Retire et retourne un element arbitraire

# Ajouter plusieurs elements
s.update([5, 6, 7])
```

### Operations ensemblistes

```python
A = {1, 2, 3, 4, 5}
B = {4, 5, 6, 7, 8}

# Union (elements dans A OU B)
print(A | B)               # {1, 2, 3, 4, 5, 6, 7, 8}
print(A.union(B))          # idem

# Intersection (elements dans A ET B)
print(A & B)               # {4, 5}
print(A.intersection(B))   # idem

# Difference (elements dans A mais PAS dans B)
print(A - B)               # {1, 2, 3}
print(A.difference(B))     # idem

# Difference symetrique (elements dans A OU B mais PAS les deux)
print(A ^ B)                       # {1, 2, 3, 6, 7, 8}
print(A.symmetric_difference(B))   # idem

# Sous-ensemble et sur-ensemble
C = {1, 2}
print(C <= A)              # True  (C est sous-ensemble de A)
print(C.issubset(A))       # True
print(A >= C)              # True  (A est sur-ensemble de C)
print(A.issuperset(C))     # True
print(A.isdisjoint(B))     # False (ils ont des elements communs)
```

```
Operations ensemblistes (diagramme de Venn) :

     A = {1,2,3,4,5}        B = {4,5,6,7,8}

   +----------+----------+
   |          |          |
   | 1  2  3  | 4  5  | 6  7  8 |
   |  (A-B)  | (A&B)  |  (B-A)  |
   |          |        |         |
   +----------+--------+---------+

   A | B  = {1, 2, 3, 4, 5, 6, 7, 8}     (union)
   A & B  = {4, 5}                         (intersection)
   A - B  = {1, 2, 3}                      (difference)
   A ^ B  = {1, 2, 3, 6, 7, 8}            (difference symetrique)
```

> [!tip] Analogie
> Les ensembles sont comme des boites ou chaque objet n'apparait qu'une fois. Si vous essayez de mettre un deuxieme exemplaire du meme objet, la boite le refuse silencieusement. Les operations ensemblistes sont exactement celles que vous avez vues en mathematiques.

### Cas d'utilisation des sets

```python
# Supprimer les doublons d'une liste
avec_doublons = [1, 2, 3, 2, 1, 4, 5, 4]
sans_doublons = list(set(avec_doublons))
print(sans_doublons)  # [1, 2, 3, 4, 5] (ordre non garanti)

# Verifier rapidement l'appartenance - O(1) vs O(n) pour une liste
grands_nombres = set(range(1_000_000))
print(999_999 in grands_nombres)  # True, instantane

# Trouver les elements communs entre deux listes
liste1 = [1, 2, 3, 4, 5]
liste2 = [4, 5, 6, 7, 8]
communs = set(liste1) & set(liste2)
print(communs)  # {4, 5}
```

---

## Frozensets

Les frozensets sont des ensembles **immutables**. Ils peuvent etre utilises comme cles de dictionnaire ou elements d'un autre set.

```python
fs = frozenset([1, 2, 3])
# fs.add(4)  # AttributeError: 'frozenset' object has no attribute 'add'

# Utilisation comme cle de dictionnaire
permissions = {
    frozenset(["lire"]): "Visiteur",
    frozenset(["lire", "ecrire"]): "Editeur",
    frozenset(["lire", "ecrire", "supprimer"]): "Admin",
}

mes_perms = frozenset(["lire", "ecrire"])
print(permissions[mes_perms])  # "Editeur"
```

---

## Tableau Comparatif des Collections

| Propriete | list | tuple | dict | set | frozenset |
|---|---|---|---|---|---|
| Mutable | Oui | Non | Oui | Oui | Non |
| Ordonne | Oui | Oui | Oui (3.7+) | Non | Non |
| Indexe | Oui | Oui | Par cle | Non | Non |
| Doublons | Oui | Oui | Cles uniques | Non | Non |
| Hashable | Non | Oui* | Non | Non | Oui |
| Syntaxe | `[1,2]` | `(1,2)` | `{"a":1}` | `{1,2}` | `frozenset()` |

> [!info] *Les tuples sont hashables uniquement si tous leurs elements sont hashables.

---

## Quand Utiliser Quelle Structure ?

```
Besoin d'une collection ?
    |
    +-- Ordre important ?
    |       |
    |       +-- Oui --> Modification necessaire ?
    |       |              |
    |       |              +-- Oui --> LISTE
    |       |              +-- Non --> TUPLE
    |       |
    |       +-- Non --> Paires cle-valeur ?
    |                      |
    |                      +-- Oui --> DICTIONNAIRE
    |                      +-- Non --> Doublons autorises ?
    |                                     |
    |                                     +-- Non --> SET
    |                                     +-- Oui --> LISTE
```

> [!example] Exemples concrets
> - **Liste** : notes d'un etudiant, historique des commandes, file d'attente
> - **Tuple** : coordonnees (x, y), couleur RGB, retour multiple de fonction
> - **Dictionnaire** : profil utilisateur, configuration, cache, compteur
> - **Set** : tags uniques, permissions, elements deja visites

---

## Performance : O(1) vs O(n)

| Operation | list | dict | set |
|---|---|---|---|
| Acces par index | O(1) | - | - |
| Acces par cle | - | O(1) | - |
| Recherche (`in`) | O(n) | O(1) | O(1) |
| Insertion | O(1) fin / O(n) debut | O(1) | O(1) |
| Suppression | O(n) | O(1) | O(1) |
| Tri | O(n log n) | - | - |

> [!warning] Performance critique
> La recherche `in` dans une **liste** est O(n) : Python parcourt chaque element. Dans un **set** ou **dict**, c'est O(1) grace a la table de hachage. Pour verifier l'appartenance dans de grandes collections, preferez toujours un set.

```python
import time

# Demonstration de la difference de performance
grande_liste = list(range(10_000_000))
grand_set = set(grande_liste)

# Recherche dans une liste : LENT
debut = time.time()
9_999_999 in grande_liste
print(f"Liste: {time.time() - debut:.4f}s")

# Recherche dans un set : RAPIDE
debut = time.time()
9_999_999 in grand_set
print(f"Set: {time.time() - debut:.4f}s")
```

---

## Equivalences avec le C

### Tableau C vs Liste Python

```c
// C : tableau statique
int arr[5] = {1, 2, 3, 4, 5};
int taille = sizeof(arr) / sizeof(arr[0]);

// C : "tableau dynamique" (manuel)
int *arr = malloc(5 * sizeof(int));
arr[0] = 1;
// ... realloc pour agrandir ...
free(arr);
```

```python
# Python : liste (gere tout automatiquement)
arr = [1, 2, 3, 4, 5]
arr.append(6)  # Agrandissement automatique
taille = len(arr)
# Pas besoin de free !
```

### Struct C vs Dictionnaire Python

```c
// C : struct
struct Etudiant {
    char nom[50];
    int age;
    float note;
};
struct Etudiant e = {"Alice", 20, 15.5};
printf("Nom: %s\n", e.nom);
```

```python
# Python : dictionnaire (plus flexible, mais pas de verification de type)
e = {"nom": "Alice", "age": 20, "note": 15.5}
print(f"Nom: {e['nom']}")
```

### Table de hachage C vs Dict Python

En C, implementer une table de hachage demande des centaines de lignes de code (fonction de hachage, gestion des collisions, redimensionnement). En Python, c'est natif.

```python
# Python : table de hachage en une ligne
table = {}
table["cle"] = "valeur"
print(table["cle"])  # "valeur"
```

> [!tip] Analogie
> Imaginez que les structures de donnees sont des rangements. En C, vous devez construire vos etageres vous-meme (definir la structure, gerer la memoire, implementer les operations). En Python, vous avez un catalogue de rangements prets a l'emploi : listes (tiroirs numerotes), dictionnaires (classeur avec etiquettes), sets (boite a elements uniques), tuples (vitrines scellees).

---

## Carte Mentale ASCII

```
              STRUCTURES DE DONNEES PYTHON
                         |
         +-------+-------+-------+-------+
         |       |       |       |       |
       Liste   Tuple   Dict    Set   Frozenset
         |       |       |       |       |
      mutable  immut.  cle:val  unique  immut.
      ordonne  ordonne  ord.   non-ord  non-ord
      indexe   indexe  par cle  non-idx  non-idx
         |       |       |       |
      +--+--+  pack/   +--+--+  +--+--+
      |     |  unpack  |     |  |     |
    slice  comp.      comp. default  union
    sort   named      dict  dict   inter.
    copy   tuple     Counter        diff.
```

---

## Exercices

### Exercice 1 : Statistiques sur une liste

Ecrivez une fonction `statistiques(nombres: list[float]) -> dict` qui retourne un dictionnaire contenant : la moyenne, la mediane, le minimum, le maximum, et l'ecart entre le min et le max. N'utilisez pas de bibliotheque externe.

### Exercice 2 : Gestionnaire de contacts

Creez un programme qui gere un repertoire de contacts (dictionnaire de dictionnaires). Implementez les operations : ajouter, rechercher, modifier, supprimer, et lister tous les contacts. Chaque contact a un nom, un telephone et un email.

### Exercice 3 : Analyse de texte avec sets

Ecrivez un programme qui prend deux textes en entree et affiche :
- Les mots communs aux deux textes
- Les mots presents uniquement dans le premier texte
- Les mots presents uniquement dans le second texte
- Le nombre total de mots uniques dans les deux textes combines
Utilisez les operations ensemblistes.

### Exercice 4 : Matrice avec list comprehensions

Creez les fonctions suivantes en utilisant des list comprehensions :
- `creer_matrice(n, m)` : cree une matrice n x m remplie de zeros
- `transposer(matrice)` : retourne la transposee
- `multiplier_scalaire(matrice, k)` : multiplie chaque element par k
- `aplatir(matrice)` : transforme une matrice en liste 1D

---

## Liens

- [[01 - Introduction a Python]] - Les bases du langage
- [[03 - POO en Python]] - Programmation orientee objet
- [[03 - Tables de Hachage]] - Theorie des tables de hachage
- [[03 - Structures et Typedef]] - Structures en C
