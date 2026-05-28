# 07 - Programmation Dynamique

> [!info] Prérequis
> Ce chapitre suppose la maîtrise de la récursion ([[07 - Recursion]]) et des notions de complexité ([[05 - Tri et Complexite Algorithmique]]). La DP est étroitement liée aux algorithmes sur graphes ([[06 - Graphes et Algorithmes de Parcours]]).

## 1. Qu'est-ce que la Programmation Dynamique ?

La **Programmation Dynamique (DP)** est une technique algorithmique qui résout des problèmes complexes en les décomposant en **sous-problèmes** plus simples, en mémorisant les résultats pour éviter les recalculs.

### Deux conditions nécessaires

1. **Sous-structure optimale** : La solution optimale du problème peut être construite à partir des solutions optimales de ses sous-problèmes.
2. **Sous-problèmes overlappants** : Les mêmes sous-problèmes sont résolus plusieurs fois (contrairement au divide & conquer où les sous-problèmes sont indépendants).

### Comparaison des approches

| Technique | Sous-problèmes | Mémorisation | Exemple |
|-----------|---------------|--------------|---------|
| **Divide & Conquer** | Indépendants | Non | Merge Sort, Quick Sort |
| **Greedy** | Pas de sous-problèmes | Non | Dijkstra, Huffman |
| **DP Top-down** | Overlappants | Oui (mémo) | Fibonacci mémoïsé |
| **DP Bottom-up** | Overlappants | Oui (table) | Fibonacci tabulé |

> [!tip] Intuition
> Si tu te retrouves à recalculer la même chose plusieurs fois dans ta récursion, la DP est probablement la bonne approche.

---

## 2. Top-Down : Mémoïsation

### 2.1 Approche manuelle

```python
def fibonacci_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci_memo(n-1, memo) + fibonacci_memo(n-2, memo)
    return memo[n]
```

> [!warning] Piège — Argument mutable par défaut
> En Python, `memo={}` est partagé entre tous les appels. Pour éviter les bugs, utiliser `memo=None` et initialiser dans la fonction.

```python
def fibonacci_memo_safe(n, memo=None):
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci_memo_safe(n-1, memo) + fibonacci_memo_safe(n-2, memo)
    return memo[n]
```

### 2.2 Décorateur `@lru_cache`

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Vider le cache
fibonacci.cache_clear()
print(fibonacci.cache_info())  # CacheInfo(hits=..., misses=..., ...)
```

### 2.3 `@functools.cache` (Python 3.9+)

```python
from functools import cache

@cache
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

> [!info] `cache` vs `lru_cache`
> `@cache` est équivalent à `@lru_cache(maxsize=None)`. Il garde tous les résultats en mémoire. `@lru_cache` avec une `maxsize` peut éviter les fuites mémoire sur de très grands espaces d'états.

---

## 3. Bottom-Up : Tabulation

```python
def fibonacci_tabulation(n):
    if n <= 1:
        return n
    
    dp = [0] * (n + 1)
    dp[0] = 0
    dp[1] = 1
    
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    
    return dp[n]

# Optimisation espace O(n) → O(1)
def fibonacci_optimized(n):
    if n <= 1:
        return n
    a, b = 0, 1
    for _ in range(2, n + 1):
        a, b = b, a + b
    return b
```

### Comparaison des approches pour Fibonacci

| Approche | Temps | Espace | Avantages |
|----------|-------|--------|-----------|
| Récursion naïve | O(2^n) | O(n) stack | Simple à écrire |
| Top-down mémo | O(n) | O(n) | Naturel, calcul à la demande |
| Bottom-up tab | O(n) | O(n) | Pas de stack overflow, plus rapide |
| Bottom-up opt | O(n) | O(1) | Optimal |

---

## 4. Problèmes classiques

### 4.1 Sac à dos 0/1 (Knapsack Problem)

**Problème** : N objets chacun avec un poids `w[i]` et une valeur `v[i]`, sac de capacité `W`. Maximiser la valeur totale sans dépasser la capacité. Chaque objet : pris (1) ou non (0).

**État DP** : `dp[i][w]` = valeur maximale avec les i premiers objets et une capacité de w.

```python
def knapsack_01(weights, values, capacity):
    n = len(weights)
    # dp[i][w] = valeur max avec les i premiers objets, capacité w
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        for w in range(capacity + 1):
            # Ne pas prendre l'objet i
            dp[i][w] = dp[i-1][w]
            # Prendre l'objet i (si possible)
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w],
                               dp[i-1][w - weights[i-1]] + values[i-1])
    
    # Reconstruction de la solution
    chosen = []
    w = capacity
    for i in range(n, 0, -1):
        if dp[i][w] != dp[i-1][w]:
            chosen.append(i-1)
            w -= weights[i-1]
    
    return dp[n][capacity], chosen[::-1]

# Exemple
weights = [2, 3, 4, 5]
values  = [3, 4, 5, 6]
capacity = 8
max_val, items = knapsack_01(weights, values, capacity)
print(f"Valeur max : {max_val}, objets : {items}")
# Valeur max : 10, objets : [0, 3] (poids 2+5=7, valeur 3+6=9... vérifier)
```

**Optimisation espace** (rolling array) :

```python
def knapsack_01_optimized(weights, values, capacity):
    n = len(weights)
    dp = [0] * (capacity + 1)
    
    for i in range(n):
        # Parcourir de droite à gauche pour ne pas réutiliser l'objet i
        for w in range(capacity, weights[i] - 1, -1):
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    
    return dp[capacity]
```

### 4.2 Longest Common Subsequence (LCS)

**Problème** : Trouver la plus longue sous-séquence commune à deux chaînes (pas nécessairement contiguë).

```python
def lcs(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    # Reconstruction
    result = []
    i, j = m, n
    while i > 0 and j > 0:
        if s1[i-1] == s2[j-1]:
            result.append(s1[i-1])
            i -= 1
            j -= 1
        elif dp[i-1][j] > dp[i][j-1]:
            i -= 1
        else:
            j -= 1
    
    return dp[m][n], ''.join(reversed(result))

print(lcs("ABCBDAB", "BDCAB"))  # (4, 'BCAB') ou 'BDAB'
```

### 4.3 Longest Common Substring

**Attention** : différent du LCS — la sous-chaîne doit être **contiguë**.

```python
def longest_common_substring(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    max_len = 0
    end_pos = 0
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
                if dp[i][j] > max_len:
                    max_len = dp[i][j]
                    end_pos = i
            else:
                dp[i][j] = 0  # Doit être contigu → reset
    
    return s1[end_pos - max_len: end_pos]

print(longest_common_substring("ABABC", "BABCAB"))  # "BABC"
```

### 4.4 Edit Distance (Levenshtein)

**Problème** : Nombre minimal d'opérations (insertion, suppression, substitution) pour transformer `s1` en `s2`. Utilisé dans git diff, correcteurs orthographiques, ADN bioinformatique.

```python
def edit_distance(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # Cas de base : transformer en/depuis chaîne vide
    for i in range(m + 1):
        dp[i][0] = i  # Supprimer i caractères
    for j in range(n + 1):
        dp[0][j] = j  # Insérer j caractères
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1]  # Pas d'opération nécessaire
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j],    # Suppression dans s1
                    dp[i][j-1],    # Insertion dans s1
                    dp[i-1][j-1]   # Substitution
                )
    
    return dp[m][n]

print(edit_distance("kitten", "sitting"))   # 3
print(edit_distance("sunday", "saturday"))  # 3
```

### 4.5 Coin Change — Nombre minimum de pièces

```python
def coin_change(coins, amount):
    """
    dp[i] = nombre minimum de pièces pour faire la somme i
    Initialiser à infini sauf dp[0] = 0
    """
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i:
                dp[i] = min(dp[i], dp[i - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1

print(coin_change([1, 5, 11], 15))  # 3 (5+5+5)
print(coin_change([2], 3))           # -1 (impossible)

def coin_change_ways(coins, amount):
    """Nombre de manières différentes de faire la somme."""
    dp = [0] * (amount + 1)
    dp[0] = 1
    
    for coin in coins:
        for i in range(coin, amount + 1):
            dp[i] += dp[i - coin]
    
    return dp[amount]

print(coin_change_ways([1, 2, 5], 5))  # 4 manières
```

### 4.6 Climbing Stairs

**Problème** : Pour monter n marches, on peut sauter 1 ou 2 marches. Combien de façons différentes d'atteindre le sommet ?

```python
def climb_stairs(n):
    if n <= 2:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    dp[2] = 2
    for i in range(3, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

# Généralisation : sauts de 1 à k marches
def climb_stairs_k(n, k):
    dp = [0] * (n + 1)
    dp[0] = 1
    for i in range(1, n + 1):
        for j in range(1, k + 1):
            if i - j >= 0:
                dp[i] += dp[i - j]
    return dp[n]
```

### 4.7 Longest Increasing Subsequence (LIS)

```python
def lis_dp(nums):
    """
    O(n²) : dp[i] = longueur de la LIS se terminant à nums[i]
    """
    if not nums:
        return 0
    n = len(nums)
    dp = [1] * n  # Chaque élément est une LIS de longueur 1
    
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)

def lis_binary_search(nums):
    """
    O(n log n) : patience sorting avec binary search
    tails[i] = le plus petit élément de fin de LIS de longueur i+1
    """
    import bisect
    tails = []
    
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    
    return len(tails)

print(lis_dp([10, 9, 2, 5, 3, 7, 101, 18]))         # 4 : [2,3,7,101]
print(lis_binary_search([10, 9, 2, 5, 3, 7, 101, 18]))  # 4
```

### 4.8 Matrix Chain Multiplication

**Problème** : Trouver l'ordre optimal de multiplication d'une chaîne de matrices pour minimiser le nombre de multiplications scalaires.

```python
def matrix_chain_order(dims):
    """
    dims[i] × dims[i+1] = taille de la matrice i
    dp[i][j] = coût min de multiplication des matrices i à j
    """
    n = len(dims) - 1  # Nombre de matrices
    dp = [[0] * n for _ in range(n)]
    bracket = [[0] * n for _ in range(n)]
    
    # l = longueur de la chaîne considérée
    for l in range(2, n + 1):
        for i in range(n - l + 1):
            j = i + l - 1
            dp[i][j] = float('inf')
            
            for k in range(i, j):
                cost = (dp[i][k] + dp[k+1][j] +
                        dims[i] * dims[k+1] * dims[j+1])
                if cost < dp[i][j]:
                    dp[i][j] = cost
                    bracket[i][j] = k
    
    return dp[0][n-1], bracket

# Matrices 30×35, 35×15, 15×5, 5×10, 10×20, 20×25
dims = [30, 35, 15, 5, 10, 20, 25]
cost, _ = matrix_chain_order(dims)
print(f"Coût optimal : {cost}")  # 15125
```

### 4.9 Rod Cutting

**Problème** : Étant donné une tige de longueur n et des prix pour chaque longueur de coupe, maximiser le revenu total.

```python
def rod_cutting(prices, n):
    """
    prices[i] = prix pour une tige de longueur i+1
    dp[i] = revenu maximal pour une tige de longueur i
    """
    dp = [0] * (n + 1)
    
    for i in range(1, n + 1):
        for j in range(i):
            if j < len(prices):
                dp[i] = max(dp[i], prices[j] + dp[i - j - 1])
    
    return dp[n]

prices = [1, 5, 8, 9, 10, 17, 17, 20]  # Prix pour longueurs 1 à 8
print(rod_cutting(prices, 8))  # 22 (couper en 2+6 ou 6+2)
```

### 4.10 Subset Sum / Partition Equal Subset Sum

```python
def subset_sum(nums, target):
    """Existe-t-il un sous-ensemble dont la somme vaut target ?"""
    dp = {0}
    for num in nums:
        dp = dp | {x + num for x in dp}
    return target in dp

def subset_sum_tabulation(nums, target):
    dp = [False] * (target + 1)
    dp[0] = True
    
    for num in nums:
        for j in range(target, num - 1, -1):
            dp[j] = dp[j] or dp[j - num]
    
    return dp[target]

def partition_equal_subset_sum(nums):
    """Peut-on partitionner nums en 2 sous-ensembles de somme égale ?"""
    total = sum(nums)
    if total % 2 != 0:
        return False
    return subset_sum_tabulation(nums, total // 2)

print(partition_equal_subset_sum([1, 5, 11, 5]))  # True : [1,5,5] et [11]
print(partition_equal_subset_sum([1, 2, 3, 5]))   # False
```

---

## 5. Optimisation de l'espace — Rolling Array

Beaucoup de DP 2D n'utilisent que la ligne précédente. On peut réduire l'espace de O(nm) à O(m).

```python
# LCS avec rolling array : O(min(m,n)) espace au lieu de O(mn)
def lcs_optimized(s1, s2):
    if len(s1) < len(s2):
        s1, s2 = s2, s1  # s2 = chaîne plus courte
    
    m, n = len(s1), len(s2)
    prev = [0] * (n + 1)
    curr = [0] * (n + 1)
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                curr[j] = prev[j-1] + 1
            else:
                curr[j] = max(prev[j], curr[j-1])
        prev, curr = curr, [0] * (n + 1)
    
    return prev[n]

# Knapsack avec une seule ligne (déjà montré plus haut)
# Edit Distance avec rolling array
def edit_distance_optimized(s1, s2):
    m, n = len(s1), len(s2)
    prev = list(range(n + 1))
    
    for i in range(1, m + 1):
        curr = [i] + [0] * n
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                curr[j] = prev[j-1]
            else:
                curr[j] = 1 + min(prev[j], curr[j-1], prev[j-1])
        prev = curr
    
    return prev[n]
```

---

## 6. Identifier un problème DP — Schéma de décomposition

### 6.1 Checklist d'identification

```
1. Le problème demande un optimum (max, min) ou un décompte ?
2. Y a-t-il des "choix" à chaque étape ?
3. Les sous-problèmes se répètent-ils ?
4. La solution optimale se construit-elle depuis des sous-solutions optimales ?
→ Si OUI à ces 4 questions : probablement DP
```

### 6.2 Patterns de décomposition courants

| Pattern | Description | Exemples |
|---------|-------------|---------|
| **Séquence 1D** | dp[i] dépend de dp[i-1], dp[i-2], etc. | Fibonacci, Climbing Stairs |
| **Séquence 2D** | dp[i][j] dépend de dp[i-1][j], dp[i][j-1] | LCS, Edit Distance |
| **Intervalle** | dp[i][j] = optimum sur l'intervalle [i,j] | Matrix Chain, Rod Cutting |
| **Sous-ensemble** | dp[mask] = résultat pour ce sous-ensemble | TSP, Scheduling |
| **Arbre** | Récursion sur l'arbre avec mémo | Tree DP |

### 6.3 Schéma de conception DP

```python
# Étape 1 : Définir l'état
# dp[i] représente quoi ?

# Étape 2 : Équation de transition
# dp[i] = f(dp[i-1], dp[i-2], ...)

# Étape 3 : Cas de base
# dp[0] = ?, dp[1] = ?

# Étape 4 : Ordre de calcul
# De gauche à droite ? De droite à gauche ?

# Étape 5 : Extraire la réponse
# return dp[n] ? max(dp) ? dp[0][n] ?
```

---

## 7. Analyse de complexité

| Problème | Temps | Espace (de base) | Espace (optimisé) |
|----------|-------|-----------------|-------------------|
| Fibonacci | O(n) | O(n) | O(1) |
| Knapsack 0/1 | O(nW) | O(nW) | O(W) |
| LCS | O(mn) | O(mn) | O(min(m,n)) |
| Edit Distance | O(mn) | O(mn) | O(min(m,n)) |
| Coin Change | O(amount × coins) | O(amount) | O(amount) |
| LIS (DP) | O(n²) | O(n) | O(n) |
| LIS (binaire) | O(n log n) | O(n) | O(n) |
| Matrix Chain | O(n³) | O(n²) | O(n²) |
| Rod Cutting | O(n²) | O(n) | O(n) |
| Subset Sum | O(nW) | O(W) | O(W) |

> [!warning] Pseudopolynomial
> Les algorithmes comme Knapsack (O(nW)) et Subset Sum (O(nW)) sont dits **pseudopolynomiaux** : leur complexité dépend de la valeur numérique de W, pas de sa taille en bits. En théorie, ils ne sont pas polynomiaux — c'est pourquoi ces problèmes sont NP-difficiles.

---

## 8. Exercices pratiques

> [!tip] Exercice 1 — Carré maximal de 1s
> Dans une matrice binaire, trouver la surface du plus grand carré ne contenant que des 1.
> **Indice** : `dp[i][j]` = côté du plus grand carré dont le coin inférieur droit est en (i,j). Si `matrix[i][j] == 1`, `dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1`.

> [!tip] Exercice 2 — Maximum Product Subarray
> Trouver le sous-tableau contigu dont le produit est maximum.
> **Indice** : Maintenir deux états — produit max ET produit min (un min peut devenir max après multiplication par un négatif).

> [!tip] Exercice 3 — Longest Palindromic Subsequence
> Trouver la plus longue sous-séquence palindromique d'une chaîne.
> **Indice** : `lps(s) = lcs(s, reverse(s))` — ou DP directe avec `dp[i][j]`.

> [!tip] Exercice 4 — Burst Balloons
> N ballons numérotés, éclater un ballon rapporte `nums[left] * nums[i] * nums[right]`. Maximiser le total.
> **Indice** : Penser à quel ballon éclater **en dernier** dans un intervalle.

> [!tip] Exercice 5 — Egg Drop Problem
> Avec k œufs et n étages, trouver le nombre minimum d'essais pour déterminer l'étage critique.
> **Indice** : `dp[k][n] = 1 + min over x of (max(dp[k-1][x-1], dp[k][n-x]))`. Ou inverser : `dp[k][m]` = nombre max d'étages testables avec k œufs et m essais.

> [!tip] Exercice 6 — Word Break
> Étant donné une chaîne et un dictionnaire, déterminer si la chaîne peut être segmentée en mots du dictionnaire.
> **Indice** : `dp[i] = True` si `s[:i]` peut être segmenté. Pour chaque j < i, si `dp[j]` et `s[j:i] in dict` → `dp[i] = True`.

> [!warning] Exercice avancé — Interleaving String
> Étant donné s1, s2, s3, déterminer si s3 est formé par l'entrelacement de s1 et s2 (en préservant l'ordre relatif des deux).
> **Indice** : `dp[i][j] = True` si `s3[:i+j]` est formé par entrelacement de `s1[:i]` et `s2[:j]`.

---

## 9. Pièges courants

> [!warning] Piège — Définition incorrecte de l'état
> L'état doit capturer **toute** l'information nécessaire pour résoudre le sous-problème indépendamment. Si tu as besoin d'informations supplémentaires non capturées dans l'état, tu as oublié une dimension.

> [!warning] Piège — Ordre de calcul Bottom-Up
> Dans la tabulation, tu dois calculer les états dans un ordre tel que quand tu calcules `dp[i]`, toutes les valeurs dont tu dépends (`dp[i-1]`, etc.) sont déjà calculées.

> [!warning] Piège — Confusion Unbounded vs 0/1 Knapsack
> Dans le **Knapsack 0/1**, chaque objet ne peut être pris qu'une fois → parcourir `w` de droite à gauche. Dans le **Unbounded Knapsack**, chaque objet peut être pris plusieurs fois → parcourir `w` de gauche à droite.

> [!info] Quand DP échoue
> La DP suppose une **sous-structure optimale**. Si la solution optimale globale ne peut pas être construite depuis des solutions optimales locales, la DP ne s'applique pas directement (exemple : longest path dans un graphe avec cycles).

---

*Liens connexes : [[05 - Tri et Complexite Algorithmique]] · [[07 - Recursion]] · [[06 - Graphes et Algorithmes de Parcours]] · [[09 - Recherche Binaire et Deux Pointeurs]] · [[10 - Algorithmes de Chaines]]*
