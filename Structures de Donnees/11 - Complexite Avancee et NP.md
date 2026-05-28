# 11 - Complexité Avancée et NP

> [!info] Prérequis
> Ce chapitre synthétise les notions de complexité de [[05 - Tri et Complexite Algorithmique]], utilise des exemples de [[07 - Programmation Dynamique]] et des graphes de [[06 - Graphes et Algorithmes de Parcours]]. Lien avec [[08 - Algorithmes Gloutons]] pour les stratégies d'approximation.

## 1. Rappel des classes de complexité temporelle

### 1.1 Tableau de croissance

| Notation | Nom | n=10 | n=100 | n=1000 | Exemple |
|----------|-----|------|-------|--------|---------|
| O(1) | Constant | 1 | 1 | 1 | Accès tableau |
| O(log n) | Logarithmique | 3 | 7 | 10 | Recherche binaire |
| O(n) | Linéaire | 10 | 100 | 1,000 | Parcours tableau |
| O(n log n) | Linéarithmique | 33 | 664 | 9,966 | Tri fusion |
| O(n²) | Quadratique | 100 | 10,000 | 10⁶ | Tri à bulles |
| O(n³) | Cubique | 1,000 | 10⁶ | 10⁹ | Floyd-Warshall |
| O(2^n) | Exponentiel | 1,024 | 10³⁰ | 10³⁰⁰ | Sous-ensembles |
| O(n!) | Factoriel | 3.6M | ∞ | ∞ | Permutations |

```python
import time
import math

def demo_complexity():
    sizes = [10, 100, 1000, 10000]
    
    for n in sizes:
        ops_constant    = 1
        ops_log         = math.ceil(math.log2(n)) if n > 0 else 0
        ops_linear      = n
        ops_nlogn       = n * math.ceil(math.log2(n)) if n > 0 else 0
        ops_quadratic   = n * n
        ops_exponential = 2**n if n <= 30 else float('inf')
        
        print(f"n={n:>6}: O(1)={ops_constant}, O(logn)={ops_log}, "
              f"O(n)={ops_linear}, O(nlogn)={ops_nlogn}, "
              f"O(n²)={ops_quadratic}, O(2^n)={'∞' if ops_exponential == float('inf') else ops_exponential}")

demo_complexity()
```

### 1.2 Règles de simplification

```
1. Ignorer les constantes multiplicatives : 5n → O(n)
2. Garder le terme dominant : n² + n + 1 → O(n²)
3. Addition de complexités : O(n) + O(n log n) = O(n log n)
4. Multiplication (boucles imbriquées) : O(n) × O(m) = O(nm)
5. log₂(n) ≡ log(n) en notation Big-O (constante multiplicative)

Exemples courants :
- Deux boucles successives : O(n) + O(n) = O(n)
- Boucle dans une boucle : O(n) × O(n) = O(n²)
- Récursion T(n) = 2T(n/2) + O(n) → O(n log n) (Master Theorem)
```

---

## 2. Analyse amortie

### 2.1 Concept

L'analyse amortie calcule le **coût moyen par opération** sur une séquence d'opérations, même si certaines opérations individuelles sont coûteuses.

> [!tip] Intuition
> Une opération coûteuse qui "paie" pour un grand nombre d'opérations futures bon marché a un coût amorti faible.

### 2.2 `list.append()` Python — Dynamic Array

```python
"""
Un tableau dynamique double sa capacité quand il est plein.
Soit n insertions :
  - n-1 insertions : O(1) chacune
  - 1 insertion qui déclenche un redimensionnement : O(n)
  
  Total : O(n) pour n insertions → coût amorti O(1) par insertion

Analyse par la méthode du potentiel :
  Φ(tableau de taille k avec n éléments) = 2n - k
  Coût amorti insert = coût réel + ΔΦ
  
  Cas sans redim : coût=1, ΔΦ=2, coût amorti = 1+2 = 3 = O(1)
  Cas avec redim : coût=n+1 (copie), ΔΦ = 2(n+1)-(2n) - (2n-n) = 2-n
                   coût amorti = (n+1) + (2-n) = 3 = O(1)
"""

import sys

# Observer le redimensionnement réel
lst = []
prev_size = sys.getsizeof(lst)
for i in range(20):
    lst.append(i)
    curr_size = sys.getsizeof(lst)
    if curr_size != prev_size:
        print(f"Redimensionnement à l'élément {i}: {prev_size} → {curr_size} bytes")
        prev_size = curr_size
```

### 2.3 Compteur binaire

```python
"""
Incrémenter un compteur binaire n fois.
Bit 0 bascule à chaque incrément : n fois.
Bit 1 bascule tous les 2 : n/2 fois.
Bit k bascule tous les 2^k : n/2^k fois.
Total = n(1 + 1/2 + 1/4 + ...) = 2n = O(n)
Coût amorti : O(1) par incrément.
"""

def increment(bits):
    i = 0
    while i < len(bits) and bits[i] == 1:
        bits[i] = 0
        i += 1
    if i < len(bits):
        bits[i] = 1
    return bits

bits = [0] * 8
flips = 0
for _ in range(16):
    old = bits[:]
    bits = increment(bits)
    f = sum(1 for a, b in zip(old, bits) if a != b)
    flips += f

print(f"16 incréments, {flips} basculements totaux → {flips/16:.1f} en moyenne")
# ≈ 2.0 en moyenne → O(1) amorti
```

### 2.4 Union-Find avec path compression

```python
"""
Union-Find avec path compression et union by rank.
Complexité amortie : O(α(n)) par opération, où α est la fonction inverse d'Ackermann.
α(n) < 5 pour tout n pratique → quasi-constant.

La preuve utilise l'analyse potentielle avec Φ = Σ rang(x) × nombre_de_descendants(x)
— formellement très complexe (Tarjan 1975).
"""
# Implémentation déjà vue dans le chapitre sur les graphes
```

---

## 3. Classes de complexité

### 3.1 P — Polynomial Time

**P** (Polynomial) = ensemble des problèmes de décision résolus par une machine de Turing **déterministe** en temps polynomial O(n^k) pour un certain k.

```
Exemples de problèmes dans P :
- Tri : O(n log n)
- Recherche : O(log n)
- Plus court chemin (Dijkstra, Bellman-Ford) : O(V² ou VE)
- Multiplication de matrices : O(n^2.37) (Coppersmith-Winograd)
- Test de primalité (AKS 2002) : O(log^6 n)
- Programmation linéaire (algorithme de l'ellipsoïde) : O(n^6)
```

> [!info] Problème de décision
> Un problème de décision a une réponse OUI/NON. "Le graphe G contient-il un chemin de longueur ≤ k ?" vs "Trouver le plus court chemin." Les versions décision et optimisation sont souvent équivalentes en complexité.

### 3.2 NP — Non-deterministic Polynomial

**NP** = ensemble des problèmes de décision pour lesquels une solution peut être **vérifiée** en temps polynomial.

Alternativement : problèmes résolus en temps polynomial par une machine de Turing **non-déterministe**.

```
Exemples de problèmes dans NP :
- SAT : vérifier qu'une assignation satisfait une formule booléenne → O(n)
- Knapsack : vérifier qu'un sous-ensemble a valeur ≥ k et poids ≤ W → O(n)
- Graph Coloring : vérifier qu'une coloration est valide → O(V+E)
- TSP : vérifier qu'un tour a longueur ≤ k → O(n)
- Subset Sum : vérifier qu'un sous-ensemble somme à T → O(n)
```

> [!warning] P ⊆ NP
> Tout problème dans P est aussi dans NP (si on peut résoudre, on peut aussi vérifier). La question est : NP ⊆ P ? (i.e., P = NP ?)

### 3.3 NP-Hard

**NP-Hard** = problèmes au moins aussi difficiles que tout problème de NP. Formellement : tout problème NP peut être réduit en temps polynomial à un problème NP-Hard.

```
Un problème H est NP-Hard si :
  ∀ L ∈ NP : L ≤_p H
  ("tout problème NP se réduit polynomialement à H")

Exemples :
- Halting Problem (indécidable, encore plus difficile que NP)
- TSP optimization (trouver le tour optimal)
- 0/1 Knapsack optimization
- Graph Coloring optimization (chromatic number)
```

> [!info] NP-Hard n'est pas dans NP
> Un problème NP-Hard n'est **pas nécessairement** dans NP. Le Halting Problem est NP-Hard mais pas dans NP (il est indécidable).

### 3.4 NP-Complete

**NP-Complete** = intersection de NP et NP-Hard.

```
NP-Complete = NP ∩ NP-Hard

Un problème C est NP-Complete si :
  1. C ∈ NP (vérifiable en temps polynomial)
  2. C est NP-Hard (tout problème NP se réduit à C)
  
Exemples (les plus célèbres) :
- 3-SAT (Cook-Levin 1971 — premier NP-Complet prouvé)
- Vertex Cover
- Clique Problem
- Independent Set
- Subset Sum
- Hamiltonian Path/Cycle
- TSP décision ("existe-t-il un tour de longueur ≤ k ?")
- 3-Colorability
```

### 3.5 Diagramme de Venn

```
┌─────────────────────────────────────────────────────────┐
│  Tous les problèmes de décision                         │
│                                                         │
│  ┌───────────────────────────────────────┐              │
│  │  NP                                   │              │
│  │                                       │              │
│  │  ┌─────────────────┐                  │              │
│  │  │  P              │                  │              │
│  │  │  (tri, chemin,  │                  │              │
│  │  │   recherche)    │                  │              │
│  │  └─────────────────┘                  │              │
│  │                                       │              │
│  │  ┌─────────────────┐                  │              │
│  │  │  NP-Complete    │ ← si P≠NP        │              │
│  │  │  (3-SAT, TSP,   │   ces problèmes  │              │
│  │  │   Knapsack...)  │   ne sont pas    │              │
│  │  └─────────────────┘   dans P         │              │
│  └───────────────────────────────────────┘              │
│                                                         │
│  NP-Hard (inclut NP-Complete + problèmes plus durs)     │
│  (Halting Problem, TSP optimal...)                      │
└─────────────────────────────────────────────────────────┘
```

---

## 4. La Question P = NP

### 4.1 Le Problème du Millénaire

**P = NP** est l'un des 7 Problèmes du Prix du Millénaire (Clay Mathematics Institute, 2000), avec une récompense de 1 million de dollars.

La question : **Si une solution à un problème peut être vérifiée rapidement, peut-elle aussi être trouvée rapidement ?**

```
Si P = NP :
  - Cryptographie cassée (RSA, ECC reposent sur des problèmes supposés dans NP \ P)
  - Médicaments découverts en secondes (repliement des protéines)
  - IA générale (apprentissage = problème d'optimisation NP)
  - Résolution de tout problème d'optimisation combinatoire

Si P ≠ NP (consensus des chercheurs, ~99%) :
  - La frontière entre "facile" et "difficile" est réelle
  - Les NP-Complets restent intractables pour les grands n
  - La cryptographie à clé publique reste viable
```

### 4.2 Pourquoi c'est ouvert

```
On sait montrer des bornes inférieures pour des modèles restreints
(circuits, formules, algorithmes à comparaisons), mais pas pour les
machines de Turing générales.

Le problème de "relativisation" : toute preuve de P≠NP via des
"oracles" échoue, et presque toutes les techniques connues relativsent.
Razborov & Rudich (1993) : les preuves "naturelles" sont insuffisantes.
```

---

## 5. Réduction Polynomiale

### 5.1 Définition

Un problème A se **réduit polynomialement** à B (noté A ≤_p B) s'il existe une fonction f calculable en temps polynomial telle que :

```
x ∈ A  ⟺  f(x) ∈ B
```

Interprétation : si on peut résoudre B efficacement, on peut résoudre A efficacement.

### 5.2 Chaîne de réductions historique

```
Circuit SAT
    ↓  (Cook-Levin, 1971)
3-SAT  ← tout NP se réduit à Circuit SAT, puis à 3-SAT
    ↓
Independent Set ↔ Vertex Cover ↔ Clique
    ↓
Hamiltonian Cycle
    ↓
TSP (decision)
    ↓
Subset Sum ↔ Partition ↔ Knapsack
```

### 5.3 Exemple — Réduction 3-SAT → Independent Set

```
3-SAT : Étant donné une formule CNF avec des clauses de taille 3,
        existe-t-il une assignation satisfaisante ?

Exemple : (x₁ ∨ x₂ ∨ x₃) ∧ (¬x₁ ∨ x₄ ∨ x₅) ∧ (¬x₂ ∨ ¬x₃ ∨ x₅)

Réduction vers Independent Set :
1. Pour chaque clause (a ∨ b ∨ c), créer un triangle (a-b-c)
2. Ajouter une arête entre xi et ¬xi pour tout i (conflit)
3. Question : existe-t-il un independent set de taille k (= nombre de clauses) ?

Correction de la réduction :
- Un independent set de taille k choisit exactement un littéral par clause
- Il ne peut pas choisir xi ET ¬xi (arête de conflit)
- L'assignation cohérente satisfait toutes les clauses
```

```python
def sat_to_independent_set(clauses, num_vars):
    """
    Construit le graphe correspondant à la réduction 3-SAT → IS.
    Retourne la liste d'arêtes et la taille cible k.
    """
    edges = set()
    node_id = {}
    
    # Créer les nœuds : (clause_index, literal_position) → int
    node_count = 0
    for i, clause in enumerate(clauses):
        for j, literal in enumerate(clause):
            node_id[(i, j)] = node_count
            node_count += 1
    
    # Arêtes internes (triangle par clause)
    for i, clause in enumerate(clauses):
        for j in range(len(clause)):
            for k in range(j+1, len(clause)):
                u, v = node_id[(i, j)], node_id[(i, k)]
                edges.add((min(u,v), max(u,v)))
    
    # Arêtes de conflit (xi ↔ ¬xi entre clauses différentes)
    for i, clause_i in enumerate(clauses):
        for j, literal_ij in enumerate(clause_i):
            for l, clause_l in enumerate(clauses):
                if l <= i:
                    continue
                for m, literal_lm in enumerate(clause_l):
                    # Conflit si l'un est la négation de l'autre
                    if literal_ij == -literal_lm:
                        u = node_id[(i, j)]
                        v = node_id[(l, m)]
                        edges.add((min(u,v), max(u,v)))
    
    return list(edges), len(clauses)  # k = nombre de clauses
```

---

## 6. Problèmes NP-Complets classiques

### 6.1 SAT / 3-SAT

```python
"""
SAT (Satisfiability) :
Étant donné une formule booléenne, existe-t-il une assignation des variables
qui la rend vraie ?

3-SAT : version où chaque clause a exactement 3 littéraux.
  (x₁ ∨ ¬x₂ ∨ x₃) ∧ (¬x₁ ∨ x₂ ∨ x₄) ∧ ...

Applications : vérification formelle, EDA (Electronic Design Automation),
              planning en IA, cryptographie.

Algorithmes exacts connus :
  - DPLL (Davis-Putnam-Logemann-Loveland) : backtracking avec propagation unitaire
  - CDCL (Conflict-Driven Clause Learning) : amélioré avec apprentissage de clauses
  - Meilleur connu : O(1.307^n) pour 3-SAT (Hertli 2014)
"""

def dpll_sat(formula, assignment={}):
    """
    DPLL simplifié : backtracking avec unit propagation.
    formula = list of lists of literals (positifs ou négatifs)
    """
    # Simplifier la formule avec l'assignment courant
    simplified = []
    for clause in formula:
        satisfied = False
        new_clause = []
        for lit in clause:
            var = abs(lit)
            if var in assignment:
                if (lit > 0) == assignment[var]:
                    satisfied = True
                    break
            else:
                new_clause.append(lit)
        if not satisfied:
            if not new_clause:
                return False  # Clause vide → contradiction
            simplified.append(new_clause)
    
    if not simplified:
        return True  # Toutes les clauses satisfaites
    
    # Unit propagation : clause unitaire → forcer l'assignation
    for clause in simplified:
        if len(clause) == 1:
            lit = clause[0]
            new_assign = dict(assignment)
            new_assign[abs(lit)] = lit > 0
            return dpll_sat(formula, new_assign)
    
    # Choisir une variable à assigner
    var = abs(simplified[0][0])
    
    for val in [True, False]:
        new_assign = dict(assignment)
        new_assign[var] = val
        if dpll_sat(formula, new_assign):
            return True
    
    return False

# Exemple : (x1 ∨ ¬x2) ∧ (¬x1 ∨ x2 ∨ x3) ∧ (¬x3)
# Variables : 1, 2, 3
formula = [[1, -2], [-1, 2, 3], [-3]]
print(dpll_sat(formula))  # True : x1=True, x2=True, x3=False
```

### 6.2 Travelling Salesman Problem (TSP)

```python
"""
TSP : Étant donné n villes et les distances entre elles,
trouver le circuit passant par toutes les villes avec la distance minimale.

NP-Complete : version décision "Existe-t-il un circuit de longueur ≤ k ?"
NP-Hard     : version optimisation "Trouver le circuit optimal."

Borne inférieure de complexité : aucun algorithme polynomial connu.
Meilleur exact : DP + bitmask O(2^n × n²) — bonne pour n ≤ 20.
"""
from functools import lru_cache
from math import inf

def tsp_dp(dist):
    """
    TSP exact avec DP + bitmask.
    dist[i][j] = distance de la ville i à la ville j.
    Complexité : O(2^n × n²), Espace : O(2^n × n)
    """
    n = len(dist)
    VISITED_ALL = (1 << n) - 1
    
    @lru_cache(maxsize=None)
    def dp(mask, pos):
        """Coût min pour visiter les villes restantes depuis pos, avec mask visité."""
        if mask == VISITED_ALL:
            return dist[pos][0]  # Retour à la ville 0
        
        min_cost = inf
        for city in range(n):
            if not (mask >> city & 1):  # Ville non visitée
                new_cost = dist[pos][city] + dp(mask | (1 << city), city)
                min_cost = min(min_cost, new_cost)
        
        return min_cost
    
    return dp(1, 0)  # Commencer à la ville 0, masque = {0}

# Exemple : 4 villes
dist = [
    [0, 10, 15, 20],
    [10,  0, 35, 25],
    [15, 35,  0, 30],
    [20, 25, 30,  0]
]
print(f"TSP optimal : {tsp_dp(dist)}")  # 80
```

### 6.3 Vertex Cover

```python
"""
Vertex Cover : sous-ensemble de sommets S tel que chaque arête (u,v) ait
au moins un sommet dans S. Taille minimale ?

NP-Complete : "Existe-t-il un vertex cover de taille ≤ k ?"

Algorithme kernélisation : paramétré en k (FPT : Fixed-Parameter Tractable)
  Si degré(v) > k : v doit être dans le cover → le prendre, réduire k
  Si aucun sommet de degré > k mais |E| > k² : impossible (lemme)
  Branchement : pour chaque arête (u,v), brancher sur u∈S ou v∈S
  Complexité : O(2^k × poly(n))
"""

def vertex_cover_brute(graph, vertices):
    """O(2^n × E) — uniquement pour petits graphes."""
    from itertools import combinations
    
    edges = set()
    for u in vertices:
        for v, _ in graph.adj[u]:
            if u < v:
                edges.add((u, v))
    
    for k in range(len(vertices) + 1):
        for cover in combinations(vertices, k):
            cover_set = set(cover)
            if all(u in cover_set or v in cover_set for u, v in edges):
                return list(cover)
    
    return list(vertices)

def vertex_cover_2_approx(graph, vertices):
    """
    Approximation à facteur 2 : choisir les deux extrémités de chaque arête non couverte.
    Garanti : |C_approx| ≤ 2 × |C_opt|
    """
    covered = set()
    cover = set()
    
    for u in vertices:
        for v, _ in graph.adj[u]:
            if u < v and u not in covered and v not in covered:
                cover.add(u)
                cover.add(v)
                covered.add(u)
                covered.add(v)
    
    return list(cover)
```

### 6.4 Graph Coloring

```python
"""
K-Colorabilité : peut-on colorier un graphe avec k couleurs telles que
deux sommets adjacents n'aient jamais la même couleur ?

k=1 : Trivial (graphe sans arêtes)
k=2 : Dans P (= test bipartite)
k≥3 : NP-Complete

Chromatic number χ(G) : plus petit k tel que G est k-colorable.
"""

def graph_coloring_backtrack(graph, vertices, k):
    """Backtracking pour k-coloring."""
    colors = {}
    
    def can_color(v, color):
        for neighbor, _ in graph.adj[v]:
            if colors.get(neighbor) == color:
                return False
        return True
    
    def backtrack(v_list, idx):
        if idx == len(v_list):
            return True
        v = v_list[idx]
        for color in range(k):
            if can_color(v, color):
                colors[v] = color
                if backtrack(v_list, idx + 1):
                    return True
                del colors[v]
        return False
    
    v_list = list(vertices)
    if backtrack(v_list, 0):
        return colors
    return None  # Pas k-colorable

def greedy_coloring(graph, vertices):
    """
    Greedy coloring (pas optimal en général).
    Garanti : χ(G) ≤ Δ(G) + 1 où Δ = degré maximum.
    Optimal pour les graphes parfaits.
    """
    colors = {}
    for v in vertices:
        neighbor_colors = {colors[u] for u, _ in graph.adj[v] if u in colors}
        color = 0
        while color in neighbor_colors:
            color += 1
        colors[v] = color
    return colors
```

---

## 7. Stratégies face aux problèmes NP

### 7.1 Algorithmes approchés (Approximation)

```python
"""
Un algorithme α-approximatif pour un problème de minimisation retourne
une solution de valeur au plus α × OPT.

Exemples :
  - Vertex Cover : 2-approximation (preuve : matching maximal)
  - TSP métrique : 1.5-approximation (Christofides-Serdyukov, 1976)
  - Max-SAT : 7/8-approximation (algo randomisé)
  - Set Cover : ln(n)-approximation (greedy)
  
Résultat fondamental : si P≠NP, certains problèmes n'ont pas d'approximation
meilleure qu'un certain seuil (non-approximabilité, PCP theorem).
Ex : Independent Set est inapproximable à n^(1-ε) pour tout ε > 0 (si P≠NP).
"""

def set_cover_greedy(universe, sets):
    """
    Couverture d'ensemble par heuristique greedy.
    Approximation : ln(n) × OPT
    """
    remaining = set(universe)
    cover = []
    used_sets = []
    
    while remaining:
        # Choisir le set qui couvre le plus d'éléments restants
        best = max(sets, key=lambda s: len(s[1] & remaining))
        if not best[1] & remaining:
            break  # Couverture impossible
        cover.extend(best[1] & remaining)
        remaining -= best[1]
        used_sets.append(best[0])
    
    return used_sets

universe = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
sets = [('S1', {1, 2, 3, 4}), ('S2', {2, 4, 6, 8}), ('S3', {3, 5, 7}),
        ('S4', {5, 6, 9}), ('S5', {7, 8, 9, 10}), ('S6', {1, 10})]
print(set_cover_greedy(universe, sets))
```

### 7.2 Heuristiques (Local Search, Greedy)

```python
def tsp_nearest_neighbor(dist):
    """
    Heuristique du voisin le plus proche pour TSP.
    Non optimale, mais O(n²) et résultats raisonnables.
    Ratio approximation : O(log n) en moyenne.
    """
    n = len(dist)
    visited = [False] * n
    path = [0]
    visited[0] = True
    
    for _ in range(n - 1):
        current = path[-1]
        nearest = min(
            (j for j in range(n) if not visited[j]),
            key=lambda j: dist[current][j]
        )
        path.append(nearest)
        visited[nearest] = True
    
    total = sum(dist[path[i]][path[i+1]] for i in range(n-1))
    total += dist[path[-1]][path[0]]
    return path, total

def tsp_2opt(dist, path):
    """
    Amélioration locale 2-opt : inverser des segments du tour.
    Converge vers un optimum local (pas global).
    """
    n = len(path)
    improved = True
    while improved:
        improved = False
        for i in range(1, n - 1):
            for j in range(i + 1, n):
                # Coût avant : path[i-1]→path[i] + path[j]→path[j+1]
                # Coût après inversion i..j : path[i-1]→path[j] + path[i]→path[j+1]
                a, b = path[i-1], path[i]
                c, d = path[j], path[(j+1) % n]
                if dist[a][b] + dist[c][d] > dist[a][c] + dist[b][d]:
                    path[i:j+1] = path[i:j+1][::-1]
                    improved = True
    return path
```

### 7.3 Branch & Bound

```python
"""
Branch & Bound : exploration de l'arbre de recherche avec élagage.
1. Borne inférieure (lower bound) sur chaque sous-problème
2. Si borne inférieure > meilleure solution connue → élaguer
3. Sinon, brancher sur une variable de décision

Efficacité dépend de la qualité de la borne inférieure.
Pour Knapsack : O(2^n) pire cas mais très performant en pratique.
"""

def knapsack_branch_bound(weights, values, capacity):
    """
    0/1 Knapsack avec branch & bound.
    Borne : fraction knapsack relaxé (greedy sur ratio value/weight).
    """
    n = len(weights)
    # Trier par ratio valeur/poids décroissant
    items = sorted(range(n), key=lambda i: values[i]/weights[i], reverse=True)
    
    best = [0]
    
    def upper_bound(idx, current_weight, current_value):
        """Borne supérieure : fraction knapsack depuis idx."""
        total_value = current_value
        remaining = capacity - current_weight
        for i in range(idx, n):
            item = items[i]
            if weights[item] <= remaining:
                total_value += values[item]
                remaining -= weights[item]
            else:
                total_value += values[item] * remaining / weights[item]
                break
        return total_value
    
    def branch(idx, current_weight, current_value):
        if idx == n or current_weight > capacity:
            best[0] = max(best[0], current_value)
            return
        
        # Élagage
        if upper_bound(idx, current_weight, current_value) <= best[0]:
            return
        
        item = items[idx]
        # Brancher : prendre l'item
        if current_weight + weights[item] <= capacity:
            branch(idx + 1, current_weight + weights[item],
                   current_value + values[item])
        # Ne pas prendre l'item
        branch(idx + 1, current_weight, current_value)
    
    branch(0, 0, 0)
    return best[0]

weights = [2, 3, 4, 5]
values  = [3, 4, 5, 6]
print(knapsack_branch_bound(weights, values, 8))  # 10
```

---

## 8. Algorithmes Randomisés

### 8.1 Las Vegas vs Monte Carlo

| Type | Garantie | Résultat |
|------|----------|---------|
| **Las Vegas** | Temps variable, résultat toujours correct | QuickSort randomisé |
| **Monte Carlo** | Temps fixe, résultat parfois incorrect | Miller-Rabin |

### 8.2 Miller-Rabin — Test de primalité probabiliste

```python
import random

def is_composite(n, a, d, r):
    """Tester si a est un témoin de la compositité de n."""
    x = pow(a, d, n)  # a^d mod n
    if x == 1 or x == n - 1:
        return False
    for _ in range(r - 1):
        x = pow(x, 2, n)
        if x == n - 1:
            return False
    return True  # n est composite

def miller_rabin(n, k=20):
    """
    Test de primalité probabiliste Miller-Rabin.
    Probabilité d'erreur : 4^(-k)
    k=20 → erreur < 10^(-12)
    O(k log² n log log n)
    """
    if n < 2:
        return False
    if n in (2, 3, 5, 7):
        return True
    if n % 2 == 0:
        return False
    
    # Écrire n-1 = 2^r × d
    r, d = 0, n - 1
    while d % 2 == 0:
        r += 1
        d //= 2
    
    # k tours
    for _ in range(k):
        a = random.randrange(2, n - 1)
        if is_composite(n, a, d, r):
            return False  # Composite avec certitude
    
    return True  # Probablement premier

# Pour usage cryptographique, utiliser les témoins déterministes :
DETERMINISTIC_WITNESSES = {
    2047: [2],
    3215031751: [2, 3, 5, 7],
    3317044064679887385961981: [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37]
}

def is_prime_deterministic(n):
    """Déterministe pour n < 3.3 × 10^24."""
    if n < 2: return False
    for a in [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37]:
        if n == a: return True
        r, d = 0, n - 1
        while d % 2 == 0:
            r += 1
            d //= 2
        if is_composite(n, a, d, r):
            return False
    return True

print(miller_rabin(104729))   # True (premier)
print(miller_rabin(104728))   # False (pair)
```

### 8.3 Algorithme de Karger — Min-Cut randomisé

```python
import random

def karger_min_cut(graph_edges, n):
    """
    Contraction aléatoire d'arêtes jusqu'à 2 super-nœuds.
    Probabilité de succès : Ω(1/n²)
    Répéter O(n² log n) fois pour haute probabilité.
    Un seul tour : O(n²)
    """
    # Copier le graphe comme liste d'adjacence avec super-nœuds
    nodes = {i: i for i in range(n)}  # node -> super-nœud
    adj = [list(e) for e in graph_edges]  # copier les arêtes
    
    remaining = n
    
    while remaining > 2:
        # Choisir une arête aléatoire
        edge = random.choice(adj)
        u, v = edge[0], edge[1]
        
        # Fusionner u et v : remplacer v par u
        adj = [[u if x == v else x for x in e] for e in adj]
        # Supprimer les boucles (self-loops)
        adj = [e for e in adj if e[0] != e[1]]
        
        remaining -= 1
    
    return len(adj)  # Nombre d'arêtes restantes = taille du min-cut

def karger_repeated(graph_edges, n, iterations=None):
    """Répéter Karger pour augmenter la probabilité de succès."""
    if iterations is None:
        import math
        iterations = n * n * int(math.log(n) + 1)
    
    min_cut = float('inf')
    for _ in range(iterations):
        cut = karger_min_cut(graph_edges, n)
        min_cut = min(min_cut, cut)
    
    return min_cut
```

---

## 9. Récapitulatif

### 9.1 Tableau de décision — "Que faire face à un problème NP ?"

```
Le problème est-il NP-Hard ?
├── Non → Algorithme polynomial connu
│         (DP, greedy, graphe...)
└── Oui → 
    ├── n est petit (≤ 20-30) ?
    │   └── Algorithme exact : DP bitmask, branch & bound
    ├── Erreur tolérée ?
    │   ├── Approximation garantie (facteur α)
    │   ├── Heuristique (SA, GA, local search)
    │   └── Algorithme randomisé (Las Vegas / Monte Carlo)
    ├── Structure spéciale du problème ?
    │   ├── Instances faciles (graphes planaires, arbres)
    │   └── Algorithmes FPT (paramétrisés)
    └── Problème continu ?
        └── Relaxation LP + rounding
```

### 9.2 Frontières de tractabilité pratique

| Complexité | n max (1 seconde, 10^8 ops/s) |
|-----------|-------------------------------|
| O(log n) | 10^(10^8) ≈ ∞ |
| O(n) | 10^8 |
| O(n log n) | ~4 × 10^6 |
| O(n²) | ~10^4 |
| O(n³) | ~500 |
| O(2^n) | ~27 |
| O(n!) | ~13 |

---

## 10. Exercices pratiques

> [!tip] Exercice 1 — Identifier la classe de complexité
> Pour chaque problème suivant, déterminer s'il est dans P, NP-Complete, ou NP-Hard :
> a) Vérifier si un graphe est Eulérien (tous les sommets de degré pair)
> b) Trouver le plus court chemin hamiltonien
> c) Vérifier qu'une solution SAT est correcte
> d) Trouver un clique de taille k dans un graphe
> e) Trier un tableau de n entiers

> [!tip] Exercice 2 — Réduction
> Montrer que SUBSET SUM se réduit à PARTITION (peut-on diviser un ensemble en deux sous-ensembles de somme égale ?).
> **Indice** : Étant donné (nums, target), construire un ensemble pour PARTITION.

> [!tip] Exercice 3 — Approximation TSP
> Implémenter l'heuristique Christofides simplifiée : MST + matching parfait sur les nœuds de degré impair. Comparer avec nearest neighbor et 2-opt sur des instances aléatoires.

> [!tip] Exercice 4 — Analyse amortie — Skip List
> Une Skip List a des opérations insert/search en O(log n) amorti. Analyser le coût amorti de l'insertion dans une skip list en supposant que chaque niveau est présent avec probabilité 1/2.

> [!warning] Exercice avancé — PCP Theorem (Lecture)
> Le PCP (Probabilistically Checkable Proofs) Theorem implique que MAX-3-SAT est inapproximable à mieux que 7/8 + ε (pour tout ε > 0) si P≠NP. Lire l'article de Håstad (2001) et résumer les implications pour les algorithmes d'approximation.

---

## 11. Pièges courants

> [!warning] Piège — Confondre "difficile" et "NP-Complet"
> "Ce problème est difficile" ne signifie pas NP-Complet. Un problème peut être difficile à coder mais polynomial (ex : algorithme de Dinic pour max-flow). NP-Complet a une définition formelle précise.

> [!warning] Piège — Pseudopolynomial n'est pas polynomial
> Knapsack 0/1 en O(nW) semble polynomial, mais W peut être exponentiel en sa représentation (log W bits). Si W = 2^30, l'algorithme est exponentiel en la taille de l'input.

> [!info] Réduction correcte
> Pour montrer qu'un problème B est NP-Complet :
> 1. Montrer B ∈ NP (vérifier une solution en temps polynomial)
> 2. Choisir un problème NP-Complet connu A
> 3. Construire une réduction polynomiale A ≤_p B
> Ne pas confondre le sens de la réduction : A se réduit à B signifie que B est au moins aussi difficile que A.

---

*Liens connexes : [[05 - Tri et Complexite Algorithmique]] · [[07 - Programmation Dynamique]] · [[06 - Graphes et Algorithmes de Parcours]] · [[08 - Algorithmes Gloutons]]*
