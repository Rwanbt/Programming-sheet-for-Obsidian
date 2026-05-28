# 08 - Algorithmes Gloutons

> [!info] Prérequis
> Ce chapitre s'appuie sur les notions de complexité de [[05 - Tri et Complexite Algorithmique]] et complète [[07 - Programmation Dynamique]] (souvent, la DP et le greedy sont deux approches alternatives). Les algorithmes de Prim et Kruskal pour le MST sont vus dans [[06 - Graphes et Algorithmes de Parcours]].

## 1. Qu'est-ce qu'un algorithme glouton ?

Un **algorithme glouton** (greedy) effectue à chaque étape le choix qui semble localement optimal, sans jamais revenir en arrière, en espérant atteindre la solution globalement optimale.

### 1.1 Conditions pour que le greedy soit optimal

Un algorithme glouton produit une solution **globalement optimale** si le problème possède :

1. **Propriété du choix glouton** : un choix localement optimal peut être étendu en une solution globalement optimale (aucun choix glouton n'est jamais "regretté").
2. **Sous-structure optimale** : la solution optimale du problème contient les solutions optimales de ses sous-problèmes.

> [!warning] Greedy ≠ toujours optimal
> Le greedy ne marche pas pour tous les problèmes. Exemple classique : changement de monnaie avec pièces {1, 3, 4} et cible 6. Greedy donne 4+1+1 = 3 pièces. DP donne 3+3 = 2 pièces.

### 1.2 Comparaison des approches

| Critère | Greedy | DP | Divide & Conquer |
|---------|--------|----|--------------------|
| Sous-structure optimale | Oui | Oui | Parfois |
| Sous-problèmes overlappants | Non requis | Oui | Non |
| Revoir les choix ? | Jamais | Non | Non |
| Complexité typique | O(n log n) | O(n²) ou + | O(n log n) |
| Preuve nécessaire | Oui (échange) | Invariant de table | Récurrence |

---

## 2. Preuve d'optimalité — L'argument d'échange

La technique standard pour prouver qu'un algorithme glouton est optimal est l'**exchange argument** (argument d'échange).

### Méthode

```
1. Supposer qu'il existe une solution optimale OPT différente de la solution glouton G.
2. Identifier la première différence entre OPT et G.
3. Montrer qu'on peut "échanger" la décision de OPT par la décision greedy
   sans détériorer la qualité de la solution.
4. Par induction, G est aussi optimale que OPT.
```

### Exemple : Activity Selection

```python
"""
Théorème : L'algorithme glouton (toujours choisir l'activité se terminant
le plus tôt, compatible avec les activités déjà choisies) est optimal.

Preuve par échange :
  Soit OPT = {a₁, a₂, ..., aₖ} triées par temps de fin.
  Soit G = {g₁, g₂, ..., gₘ} la solution greedy.
  
  Montrons m = k et G est aussi optimale que OPT.
  
  g₁ se termine au plus tôt de toutes les activités compatibles.
  Donc fin(g₁) ≤ fin(a₁).
  On peut remplacer a₁ par g₁ dans OPT sans invalidité (g₁ se termine avant a₁).
  
  On obtient OPT' = {g₁, a₂, ..., aₖ} qui est encore une solution valide de même taille.
  Par induction sur le reste des activités, G est optimal.
"""
```

---

## 3. Activity Selection Problem

**Problème** : N activités, chacune avec un début `s[i]` et une fin `f[i]`. Maximiser le nombre d'activités non-chevauchantes sélectionnées.

```python
def activity_selection(activities):
    """
    Trier par temps de fin → toujours sélectionner l'activité compatible
    se terminant le plus tôt.
    Complexité : O(n log n) pour le tri, O(n) pour la sélection.
    """
    # Trier par temps de fin
    sorted_acts = sorted(activities, key=lambda x: x[1])
    
    selected = [sorted_acts[0]]
    last_finish = sorted_acts[0][1]
    
    for start, finish, name in sorted_acts[1:]:
        if start >= last_finish:  # Pas de chevauchement
            selected.append((start, finish, name))
            last_finish = finish
    
    return selected

activities = [
    (1, 4, "A"), (3, 5, "B"), (0, 6, "C"), (5, 7, "D"),
    (3, 9, "E"), (5, 9, "F"), (6, 10, "G"), (8, 11, "H"),
    (8, 12, "I"), (2, 14, "J"), (12, 16, "K")
]

result = activity_selection(activities)
print(f"Activités sélectionnées ({len(result)}) :", [(n, s, f) for s,f,n in result])
# A(1-4), D(5-7), H(8-11), K(12-16) → 4 activités
```

### Variante — Activity Selection avec poids (valeurs)

> [!info] Quand le greedy échoue
> Si chaque activité a un **poids** (valeur), maximiser la valeur totale n'est plus faisable avec le greedy. Il faut la **DP** (Weighted Job Scheduling). C'est l'exemple typique où greedy ne suffit plus.

```python
def weighted_job_scheduling(jobs):
    """
    jobs = [(start, end, profit), ...]
    DP après tri par temps de fin.
    O(n log n)
    """
    import bisect
    
    jobs.sort(key=lambda x: x[1])
    n = len(jobs)
    
    # Trouver le dernier job compatible (ne se chevauche pas avec jobs[i])
    ends = [j[1] for j in jobs]
    
    def latest_compatible(i):
        # Plus grand j < i tel que jobs[j].end <= jobs[i].start
        pos = bisect.bisect_right(ends, jobs[i][0], 0, i) - 1
        return pos
    
    dp = [0] * (n + 1)
    for i in range(1, n + 1):
        job = jobs[i-1]
        lc = latest_compatible(i-1)
        # Prendre ou ne pas prendre le job i
        dp[i] = max(dp[i-1], dp[lc+1] + job[2])
    
    return dp[n]

jobs = [(1,4,3), (3,5,4), (0,6,2), (5,7,1), (3,9,5), (6,10,6), (8,11,2)]
print(weighted_job_scheduling(jobs))  # Valeur optimale
```

---

## 4. Algorithme de Huffman — Compression de données

**Problème** : Étant donné les fréquences de n symboles, construire un code préfixe (arbre binaire) qui minimise la longueur moyenne du code.

**Propriété optimale** : Les deux symboles les moins fréquents sont toujours des feuilles sœurs au niveau le plus bas de l'arbre optimal.

```python
import heapq

class HuffmanNode:
    def __init__(self, char, freq, left=None, right=None):
        self.char = char
        self.freq = freq
        self.left = left
        self.right = right
    
    def __lt__(self, other):
        # Tie-breaking par caractère pour cohérence
        if self.freq == other.freq:
            return (self.char or '') < (other.char or '')
        return self.freq < other.freq

def huffman(frequencies):
    """
    Algorithme de Huffman.
    Complexité : O(n log n)
    
    1. Créer un nœud feuille pour chaque symbole
    2. Tant qu'il y a plus d'un nœud dans la file :
       a. Extraire les deux nœuds de fréquence minimale
       b. Créer un nœud interne avec ces deux nœuds comme enfants
       c. Sa fréquence = somme des fréquences
       d. Réinsérer dans la file
    """
    heap = [HuffmanNode(char, freq) for char, freq in frequencies.items()]
    heapq.heapify(heap)
    
    while len(heap) > 1:
        left = heapq.heappop(heap)
        right = heapq.heappop(heap)
        merged = HuffmanNode(None, left.freq + right.freq, left, right)
        heapq.heappush(heap, merged)
    
    root = heap[0] if heap else None
    
    # Générer les codes
    codes = {}
    def generate(node, code=""):
        if node is None:
            return
        if node.char is not None:
            codes[node.char] = code if code else "0"
            return
        generate(node.left, code + "0")
        generate(node.right, code + "1")
    
    generate(root)
    return codes

def huffman_analysis(text):
    from collections import Counter
    freq = Counter(text)
    codes = huffman(freq)
    
    original_bits = len(text) * 8  # ASCII 8 bits par caractère
    compressed_bits = sum(freq[c] * len(codes[c]) for c in freq)
    avg_length = compressed_bits / len(text)
    
    print(f"Codes : {codes}")
    print(f"Bits originaux : {original_bits}")
    print(f"Bits compressés : {compressed_bits}")
    print(f"Taux de compression : {compressed_bits/original_bits:.1%}")
    print(f"Longueur moyenne du code : {avg_length:.2f} bits/symbole")
    
    return codes

huffman_analysis("aabbbccddddeeeeee")
```

### Preuve d'optimalité de Huffman

```
Lemme 1 (greedy choice) : Dans tout arbre de code optimal, les deux symboles
de fréquence minimale ont le même niveau de profondeur (sont des feuilles sœurs).

Preuve : Supposons qu'ils ne soient pas au même niveau. Les nœuds les plus profonds
ont la longueur de code la plus grande. Échanger les deux symboles de fréquence min
avec les feuilles les plus profondes ne peut qu'améliorer ou laisser inchangé le
coût total.

Lemme 2 (sous-structure optimale) : Fusionner les deux symboles de fréq min en un
super-symbole de fréquence f₁+f₂ → l'arbre optimal pour les (n-1) symboles restants
correspond à un arbre optimal pour les n symboles originaux.

Théorème : Huffman est optimal.
```

---

## 5. Problème du rendu de monnaie

### 5.1 Greedy (fonctionne pour certains systèmes de pièces)

```python
def coin_change_greedy(coins, amount):
    """
    Greedy : toujours prendre la pièce la plus grande possible.
    Optimal pour les systèmes de pièces "canoniques" (euro, dollar, etc.)
    NON optimal en général.
    """
    coins = sorted(coins, reverse=True)
    result = []
    
    for coin in coins:
        while amount >= coin:
            result.append(coin)
            amount -= coin
    
    if amount != 0:
        return None  # Impossible
    return result

# Système canonique (euro) → greedy optimal
print(coin_change_greedy([1, 2, 5, 10, 20, 50, 100, 200], 348))
# [200, 100, 20, 20, 5, 2, 1] = 7 pièces ✓

# Système non-canonique → greedy non optimal
print(coin_change_greedy([1, 3, 4], 6))
# [4, 1, 1] = 3 pièces (greedy)
# Optimal : [3, 3] = 2 pièces (DP)
```

### 5.2 Quand le greedy est optimal pour le rendu de monnaie ?

```
Condition suffisante (pas nécessaire) :
Pour tout i, valeur[i] ≥ 2 × valeur[i-1] (système "superincroissant").

Exemple : {1, 2, 5, 10, 25} (cents US) → greedy optimal.
Preuve : Si on peut utiliser une pièce de valeur v, l'utiliser directement
         coûte 1 pièce. Ne pas l'utiliser nécessiterait des pièces de
         valeur < v, et leur total ne peut pas dépasser v-1 sans saturation.
```

---

## 6. Scheduling with Deadlines

**Problème** : N tâches, chacune avec une deadline `d[i]` et un profit `p[i]`. Chaque tâche prend 1 unité de temps. Maximiser le profit total en sélectionnant les tâches à exécuter dans leurs deadlines.

```python
def job_scheduling_with_deadlines(jobs):
    """
    Algorithme glouton :
    1. Trier les tâches par profit décroissant
    2. Placer chaque tâche dans le dernier slot disponible avant sa deadline
    
    Complexité : O(n²) — optimisable à O(n log n) avec Union-Find
    """
    # Trier par profit décroissant
    jobs = sorted(jobs, key=lambda x: -x[2])  # (name, deadline, profit)
    
    max_deadline = max(d for _, d, _ in jobs)
    slots = [None] * (max_deadline + 1)  # slots[0] inutilisé
    
    selected = []
    total_profit = 0
    
    for name, deadline, profit in jobs:
        # Trouver le dernier slot libre avant la deadline
        for t in range(deadline, 0, -1):
            if slots[t] is None:
                slots[t] = name
                selected.append(name)
                total_profit += profit
                break
    
    return selected, total_profit

jobs = [
    ("J1", 2, 100),
    ("J2", 1, 19),
    ("J3", 2, 27),
    ("J4", 1, 25),
    ("J5", 3, 15)
]
result, profit = job_scheduling_with_deadlines(jobs)
print(f"Tâches : {result}, Profit : {profit}")
# Tâches : ['J1', 'J3', 'J5'], Profit : 142
```

### Optimisation avec Union-Find

```python
def job_scheduling_fast(jobs):
    """
    O(n log n) avec Union-Find pour trouver le prochain slot libre.
    """
    jobs = sorted(jobs, key=lambda x: -x[2])
    max_d = max(d for _, d, _ in jobs)
    
    # Union-Find : parent[t] = premier slot libre <= t
    parent = list(range(max_d + 2))
    
    def find(t):
        if parent[t] != t:
            parent[t] = find(parent[t])
        return parent[t]
    
    selected = []
    total_profit = 0
    
    for name, deadline, profit in jobs:
        slot = find(min(deadline, max_d))
        if slot > 0:
            selected.append(name)
            total_profit += profit
            parent[slot] = slot - 1  # Marquer ce slot comme utilisé
    
    return selected, total_profit
```

---

## 7. Algorithmes greedy sur les graphes

### 7.1 Algorithme de Prim (MST)

Déjà couvert dans [[06 - Graphes et Algorithmes de Parcours]] — le principe glouton : toujours ajouter l'arête de poids minimal qui connecte un sommet non encore dans l'arbre.

```python
"""
Preuve d'optimalité de Prim (par échange) :
Supposons que la solution greedy G ne soit pas optimale.
Soit OPT une solution optimale différente de G.
La première arête e dans G mais pas dans OPT forme un cycle dans OPT+{e}.
Ce cycle contient une arête f ∉ G avec poids(f) ≥ poids(e) (car Prim a préféré e).
Remplacer f par e dans OPT → OPT' de poids ≤ OPT.
Répéter → G est optimal.
"""
```

### 7.2 Algorithme de Dijkstra (Plus court chemin)

Dijkstra est un algorithme glouton : à chaque étape, fixer la distance du sommet non-visité le plus proche.

```python
"""
Propriété glouton de Dijkstra :
Invariant : quand un sommet u est extrait de la priority queue,
dist[u] est définitivement la distance minimale de la source à u.

Preuve (par contradiction + induction) :
Supposons que dist[u] ne soit pas optimal. Il existe un chemin plus court
s → ... → v → u. Mais dist[v] est déjà optimal (par induction), et comme
le chemin passe par v avant u, dist[v] < dist[u]. Donc v aurait dû être
extrait avant u. Contradiction.

Pourquoi les poids négatifs cassent Dijkstra :
Un chemin s → u → v avec weight(u,v) < 0 peut être plus court que tout
ce qu'on a vu avant d'extraire u. L'invariant de Dijkstra ne tient plus.
"""
```

### 7.3 Algorithme de Kruskal (MST)

Greedy sur les arêtes : trier par poids croissant, ajouter si pas de cycle.

```python
"""
Preuve d'optimalité par la propriété du cycle et la propriété de coupure :

Propriété de coupure : Pour toute coupure (S, V\S) du graphe,
l'arête de poids minimal traversant la coupure appartient à tout MST.

Kruskal maintient cette propriété à chaque étape :
Quand on ajoute une arête e, la coupure considérée est
{composante de u, reste du graphe}. L'arête de poids minimal
traversant cette coupure est bien e (car on traite par ordre croissant).
"""
```

---

## 8. Interval Scheduling et ses variantes

### 8.1 Interval Partitioning (Minimiser le nombre de ressources)

**Problème** : Planifier n activités dans un minimum de salles.

```python
import heapq

def interval_partitioning(intervals):
    """
    Greedy : trier par temps de début, assigner chaque activité à la salle
    se terminant le plus tôt (si disponible).
    Optimale par l'argument d'échange.
    Complexité : O(n log n)
    """
    intervals = sorted(intervals, key=lambda x: x[0])
    # Heap des temps de fin des salles (min-heap)
    rooms = []  # heap de temps de fin
    assignment = []
    
    for start, end, name in intervals:
        if rooms and rooms[0] <= start:
            # Réutiliser la salle qui se libère le plus tôt
            room_id = heapq.heapreplace(rooms, end)
        else:
            # Ouvrir une nouvelle salle
            heapq.heappush(rooms, end)
            room_id = len(rooms)
        assignment.append((name, room_id))
    
    return len(rooms), assignment  # Nombre de salles, assignations

intervals = [(1,4,"A"), (3,5,"B"), (0,6,"C"), (5,7,"D"), (3,9,"E"), (6,10,"G")]
num_rooms, assign = interval_partitioning(intervals)
print(f"Salles nécessaires : {num_rooms}")
# Lemme clé : profondeur = nombre max d'intervalles simultanément actifs = min salles
```

### 8.2 Minimiser le retard maximal (Minimize Lateness)

**Problème** : N tâches avec deadlines et durées. Toutes doivent être exécutées. Minimiser le retard maximum.

```python
def minimize_lateness(jobs):
    """
    Greedy : trier par deadline croissante (Earliest Deadline First).
    Minimise le retard maximal.
    Complexité : O(n log n)
    """
    jobs = sorted(jobs, key=lambda x: x[1])  # (durée, deadline, nom)
    
    time = 0
    max_lateness = 0
    schedule = []
    
    for duration, deadline, name in jobs:
        time += duration
        lateness = max(0, time - deadline)
        max_lateness = max(max_lateness, lateness)
        schedule.append((name, time - duration, time, lateness))
    
    return max_lateness, schedule

jobs = [(3, 6, "J1"), (2, 9, "J2"), (1, 8, "J3"), (4, 9, "J4"), (3, 14, "J5"), (2, 15, "J6")]
max_late, sched = minimize_lateness(jobs)
print(f"Retard maximal : {max_late}")
```

---

## 9. Problèmes où le greedy ne fonctionne pas

### 9.1 0/1 Knapsack

```python
"""
CONTRE-EXEMPLE : Knapsack 0/1 avec greedy sur le ratio valeur/poids.

Objets : A(poids=1, valeur=1), B(poids=2, valeur=2), C(poids=3, valeur=3)
Capacité = 4

Ratios : A=1.0, B=1.0, C=1.0 → tous égaux

Greedy prend C (poids 3, valeur 3), puis A (poids 1, valeur 1) → total = 4
Optimal : B + B impossible (0/1), B + A = valeur 3, ou B + C impossible (poids 5)
          A + B = valeur 3, C seul = 3... Optimal = B + A = 3. Égal ici.

Contre-exemple plus clair :
Objets : A(poids=10, valeur=60), B(poids=20, valeur=100), C(poids=30, valeur=120)
Capacité = 50

Ratios : A=6.0, B=5.0, C=4.0
Greedy : A(60) + B(100) = 160 ✓ (poids 30 ≤ 50)
         A + B + C = poids 60 > 50 → stop

Mais optimal : B + C = 100 + 120 = 220 ! (poids 50 ≤ 50)
Le greedy prend A car meilleur ratio, empêchant B+C.
"""

def knapsack_greedy_wrong(items, capacity):
    """Ne pas utiliser — démonstration que le greedy échoue sur 0/1 knapsack."""
    items = sorted(items, key=lambda x: x[1]/x[0], reverse=True)
    total_value = 0
    total_weight = 0
    
    for weight, value, name in items:
        if total_weight + weight <= capacity:
            total_weight += weight
            total_value += value
            print(f"  Pris : {name} (poids={weight}, val={value})")
    
    return total_value

print("Knapsack greedy (MAUVAIS):")
items = [(10, 60, "A"), (20, 100, "B"), (30, 120, "C")]
print(f"Valeur greedy : {knapsack_greedy_wrong(items, 50)}")   # 160 (sous-optimal)
print(f"Valeur optimale : {220}")                               # B+C = 220
```

### 9.2 Shortest Path avec poids négatifs

```
Le greedy (Dijkstra) échoue avec des poids négatifs.
Solution : Bellman-Ford (DP) — vu dans [[06 - Graphes et Algorithmes de Parcours]].
```

### 9.3 Coin Change avec pièces non-canoniques

```python
"""
Pièces {1, 3, 4}, cible 6 :
Greedy : 4 + 1 + 1 = 3 pièces
Optimal : 3 + 3 = 2 pièces (DP)
```

---

## 10. Récapitulatif — Quand utiliser le greedy ?

| Problème | Greedy ? | Raison |
|---------|---------|--------|
| Activity Selection | ✓ | Propriété d'échange prouvée |
| Huffman Coding | ✓ | Propriété d'échange prouvée |
| Dijkstra (poids positifs) | ✓ | Invariant de distance minimale |
| Prim / Kruskal (MST) | ✓ | Propriété de coupure |
| Job Scheduling (max profit) | ✓ | Échange sur profit décroissant |
| Minimize Lateness | ✓ | EDF optimal prouvé |
| 0/1 Knapsack | ✗ | Contre-exemple existe → DP |
| TSP | ✗ | NP-Complet |
| Coin Change (non-canonique) | ✗ | Contre-exemple existe → DP |
| Shortest Path (poids négatifs) | ✗ | Dijkstra échoue → Bellman-Ford |

---

## 11. Exercices pratiques

> [!tip] Exercice 1 — Fractional Knapsack
> Contrairement au 0/1 Knapsack, on peut prendre des fractions d'objets. Montrer que le greedy sur le ratio valeur/poids est optimal.
> **Indice** : Preuve par échange. Si OPT n'utilise pas l'objet de meilleur ratio en premier, l'échanger améliore ou maintient la valeur.

> [!tip] Exercice 2 — Gas Station
> N stations d'essence sur un circuit. `gas[i]` = essence disponible, `cost[i]` = essence pour aller à la station suivante. Trouver la station de départ si un circuit complet est possible.
> **Indice** : Greedy. Si cumulative sum devient négative, la station de départ doit être réinitialisée à i+1.

> [!tip] Exercice 3 — Jump Game
> Tableau d'entiers où `nums[i]` = nombre max de pas depuis i. Peut-on atteindre la dernière case ?
> **Indice** : Greedy. Maintenir `max_reach = 0`. À chaque i, `max_reach = max(max_reach, i + nums[i])`. Si i > max_reach → impossible.

> [!tip] Exercice 4 — Candy Distribution
> N enfants alignés avec des notes. Chaque enfant doit avoir au moins 1 bonbon. Les enfants avec une note plus haute que leurs voisins doivent en avoir plus. Minimiser le total de bonbons.
> **Indice** : Deux passes greedy. D'abord gauche→droite (satisfaire comparaison avec le voisin gauche), puis droite→gauche (satisfaire comparaison avec le voisin droit).

> [!tip] Exercice 5 — Meeting Rooms I
> Étant donné des intervalles de réunions, déterminer si une personne peut assister à toutes (sans chevauchement).
> **Indice** : Trier par début, vérifier qu'aucune réunion ne commence avant la fin de la précédente.

> [!warning] Exercice avancé — Optimal Merge Pattern
> Étant donné n fichiers de tailles différentes à fusionner (deux à la fois), minimiser le coût total de fusion (coût = somme des tailles des fichiers fusionnés à chaque étape).
> **Indice** : Identique à Huffman. Utiliser un min-heap. Preuve : optimal via le même argument d'échange que Huffman.

---

## 12. Pièges courants

> [!warning] Piège — Supposer que le greedy marche sans preuve
> Beaucoup de problèmes semblent se prêter au greedy mais ne le sont pas. Toujours chercher un contre-exemple avant de coder. Si tu trouves un contre-exemple en 5 minutes → greedy ne marche pas.

> [!warning] Piège — Confondre Greedy et DP
> Le greedy ne revient jamais sur ses décisions. La DP considère toutes les possibilités via la table. Si ton problème nécessite de "comparer plusieurs options", c'est probablement de la DP.

> [!info] Greedy vs DP pour Coin Change
> Coin Change avec pièces canoniques : Greedy ✓
> Coin Change avec pièces quelconques : DP obligatoire
> La frontière entre les deux dépend de la structure du problème, pas du fait qu'il s'agisse du "même" problème en surface.

---

*Liens connexes : [[05 - Tri et Complexite Algorithmique]] · [[06 - Graphes et Algorithmes de Parcours]] · [[07 - Programmation Dynamique]] · [[11 - Complexite Avancee et NP]] · [[08 - Heaps et Priority Queues]]*
