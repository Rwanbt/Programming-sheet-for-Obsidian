# 08 - Heaps et Priority Queues

> [!info] Prérequis
> Ce chapitre suppose la maîtrise des tableaux et des arbres ([[04 - Arbres Binaires]]). Les heaps sont fondamentaux pour [[06 - Graphes et Algorithmes de Parcours]] (Dijkstra, Prim) et les algorithmes de tri ([[05 - Tri et Complexite Algorithmique]]).

## 1. Heap binaire — Fondements

### 1.1 Définition

Un **heap binaire** est un arbre binaire **complet** (tous les niveaux remplis sauf le dernier, rempli de gauche à droite) qui satisfait la **propriété de tas** :

- **Min-heap** : chaque nœud est ≤ à ses enfants → la racine est le minimum global
- **Max-heap** : chaque nœud est ≥ à ses enfants → la racine est le maximum global

```
Min-heap example:           Max-heap example:
        1                           9
      /   \                       /   \
     3     2                     7     8
    / \   / \                   / \   / \
   6   4 8   5                 4   6 5   3
```

### 1.2 Représentation en tableau

Grâce à la propriété d'arbre complet, un heap se représente **sans pointeurs** :

```
Index :    0  1  2  3  4  5  6
Tableau : [1, 3, 2, 6, 4, 8, 5]

Parent de i    : (i - 1) // 2
Enfant gauche  : 2*i + 1
Enfant droit   : 2*i + 2
```

```python
# Navigation dans un heap tableau
def parent(i):
    return (i - 1) // 2

def left_child(i):
    return 2 * i + 1

def right_child(i):
    return 2 * i + 2

def is_root(i):
    return i == 0

def is_leaf(i, size):
    return left_child(i) >= size
```

> [!tip] Avantage de la représentation tableau
> Zéro overhead de pointeurs, excellente localité de cache, accès O(1) aux enfants et au parent. C'est la représentation standard dans toutes les implémentations de production.

---

## 2. Implémentation complète d'un Min-Heap

### 2.1 Sift-up et Sift-down

```python
class MinHeap:
    def __init__(self):
        self.heap = []
    
    def size(self):
        return len(self.heap)
    
    def peek(self):
        """Accès au minimum : O(1)"""
        if not self.heap:
            raise IndexError("Heap vide")
        return self.heap[0]
    
    def _sift_up(self, i):
        """
        Après insertion en fin, faire remonter l'élément.
        Complexité : O(log n)
        """
        while i > 0:
            p = (i - 1) // 2
            if self.heap[i] < self.heap[p]:
                self.heap[i], self.heap[p] = self.heap[p], self.heap[i]
                i = p
            else:
                break
    
    def _sift_down(self, i):
        """
        Après extraction de la racine, faire descendre l'élément.
        Complexité : O(log n)
        """
        n = len(self.heap)
        while True:
            smallest = i
            l = 2 * i + 1
            r = 2 * i + 2
            
            if l < n and self.heap[l] < self.heap[smallest]:
                smallest = l
            if r < n and self.heap[r] < self.heap[smallest]:
                smallest = r
            
            if smallest != i:
                self.heap[i], self.heap[smallest] = self.heap[smallest], self.heap[i]
                i = smallest
            else:
                break
    
    def insert(self, val):
        """Insérer un élément : O(log n)"""
        self.heap.append(val)
        self._sift_up(len(self.heap) - 1)
    
    def extract_min(self):
        """Extraire le minimum : O(log n)"""
        if not self.heap:
            raise IndexError("Heap vide")
        if len(self.heap) == 1:
            return self.heap.pop()
        
        min_val = self.heap[0]
        # Mettre le dernier élément à la racine
        self.heap[0] = self.heap.pop()
        self._sift_down(0)
        return min_val
    
    def heapify(self, arr):
        """
        Construire un heap depuis un tableau : O(n)
        (pas O(n log n) comme des insertions répétées)
        On part des derniers nœuds internes et on sift-down chacun.
        """
        self.heap = arr[:]
        n = len(self.heap)
        # Dernier nœud interne à l'index n//2 - 1
        for i in range(n // 2 - 1, -1, -1):
            self._sift_down(i)

# Test
h = MinHeap()
for v in [5, 3, 8, 1, 4, 2, 7]:
    h.insert(v)
results = [h.extract_min() for _ in range(h.size() + 7)]
# Sera [1, 2, 3, 4, 5, 7, 8]
```

### 2.2 Pourquoi heapify est O(n) et non O(n log n) ?

```
La moitié des nœuds sont des feuilles → sift-down coût 0
1/4 des nœuds sont au niveau n-2 → sift-down coût 1
1/8 des nœuds sont au niveau n-3 → sift-down coût 2
...

Somme = n/2 × 0 + n/4 × 1 + n/8 × 2 + ...
      = n × Σ(k/2^(k+1)) pour k=0 à log n
      = n × 1 = O(n)
```

---

## 3. Module `heapq` Python

Python fournit uniquement un **min-heap** via le module `heapq`. Les fonctions opèrent sur des listes ordinaires.

```python
import heapq

# Créer et utiliser un min-heap
h = []
heapq.heappush(h, 5)
heapq.heappush(h, 3)
heapq.heappush(h, 8)
heapq.heappush(h, 1)

print(heapq.heappop(h))   # 1 (minimum)
print(h[0])               # 3 (peek sans extraction)

# heapify : transformer une liste en heap en O(n)
arr = [5, 3, 8, 1, 4, 2, 7]
heapq.heapify(arr)
print(arr)  # [1, 3, 2, 5, 4, 8, 7] (représentation heap)

# nlargest et nsmallest
data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]
print(heapq.nlargest(3, data))   # [9, 6, 5]
print(heapq.nsmallest(3, data))  # [1, 1, 2]

# heappushpop : push puis pop (plus efficace que push + pop séparés)
h2 = [1, 2, 3]
heapq.heapify(h2)
print(heapq.heappushpop(h2, 0))  # 0 (nouveau minimum, entré et sorti)
print(heapq.heapreplace(h2, 10)) # 1 (pop d'abord, puis push — plus rapide)
```

### 3.1 Max-heap avec `heapq`

```python
# Astuce : inverser le signe pour simuler un max-heap
import heapq

h = []
for val in [5, 3, 8, 1, 4]:
    heapq.heappush(h, -val)  # Stocker les négatifs

max_val = -heapq.heappop(h)  # -(-8) = 8
print(max_val)  # 8
```

### 3.2 Heap avec priorités composées (tuple trick)

```python
import heapq

# (priorité, identifiant, données)
# Python compare les tuples lexicographiquement
tasks = []
heapq.heappush(tasks, (2, 0, "tâche moyenne"))
heapq.heappush(tasks, (1, 1, "tâche urgente"))
heapq.heappush(tasks, (3, 2, "tâche basse priorité"))
heapq.heappush(tasks, (1, 3, "autre tâche urgente"))

while tasks:
    priority, task_id, name = heapq.heappop(tasks)
    print(f"[{priority}] {name}")
# [1] tâche urgente
# [1] autre tâche urgente
# [2] tâche moyenne
# [3] tâche basse priorité
```

---

## 4. Applications pratiques

### 4.1 K plus grands/petits éléments

```python
import heapq

def k_largest(nums, k):
    """
    Maintenir un min-heap de taille k.
    Si un élément > sommet du heap → le remplacer.
    Complexité : O(n log k) — meilleur que O(n log n) si k << n
    """
    return heapq.nlargest(k, nums)  # Implémentation interne similaire

def k_largest_manual(nums, k):
    min_heap = nums[:k]
    heapq.heapify(min_heap)
    
    for num in nums[k:]:
        if num > min_heap[0]:
            heapq.heapreplace(min_heap, num)
    
    return sorted(min_heap, reverse=True)

def k_smallest(nums, k):
    return heapq.nsmallest(k, nums)

# Kème plus grand élément
def kth_largest(nums, k):
    return heapq.nlargest(k, nums)[-1]

def kth_largest_heap(nums, k):
    min_heap = nums[:k]
    heapq.heapify(min_heap)
    for num in nums[k:]:
        if num > min_heap[0]:
            heapq.heapreplace(min_heap, num)
    return min_heap[0]
```

### 4.2 Fusionner K listes triées

```python
import heapq

def merge_k_sorted_lists(lists):
    """
    Utiliser un heap de taille k : (valeur, index_liste, index_dans_liste)
    Complexité : O(n log k) où n = nombre total d'éléments
    """
    result = []
    heap = []
    
    # Initialiser avec le premier élément de chaque liste
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst[0], i, 0))
    
    while heap:
        val, list_idx, elem_idx = heapq.heappop(heap)
        result.append(val)
        
        next_idx = elem_idx + 1
        if next_idx < len(lists[list_idx]):
            heapq.heappush(heap, (lists[list_idx][next_idx], list_idx, next_idx))
    
    return result

# Avec itertools.merge (déjà optimisé)
import itertools
def merge_k_sorted_v2(lists):
    return list(heapq.merge(*lists))

lists = [[1, 4, 7], [2, 5, 8], [3, 6, 9]]
print(merge_k_sorted_lists(lists))  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 4.3 Médiane en streaming (Two Heaps Trick)

**Problème** : Maintenir la médiane d'un flux d'entiers en temps réel.

```python
import heapq

class MedianFinder:
    """
    Deux heaps :
    - max_heap : moitié inférieure (stocker négatifs pour simuler max-heap)
    - min_heap : moitié supérieure
    
    Invariant : len(max_heap) == len(min_heap) ou len(max_heap) == len(min_heap) + 1
    max_heap[0] <= min_heap[0]
    """
    def __init__(self):
        self.max_heap = []  # Moitié inférieure (max-heap via négatifs)
        self.min_heap = []  # Moitié supérieure (min-heap)
    
    def add_num(self, num):
        # Insérer dans max_heap
        heapq.heappush(self.max_heap, -num)
        
        # Équilibrer : s'assurer que max(max_heap) <= min(min_heap)
        if self.min_heap and -self.max_heap[0] > self.min_heap[0]:
            val = -heapq.heappop(self.max_heap)
            heapq.heappush(self.min_heap, val)
        
        # Équilibrer les tailles
        if len(self.max_heap) > len(self.min_heap) + 1:
            val = -heapq.heappop(self.max_heap)
            heapq.heappush(self.min_heap, val)
        elif len(self.min_heap) > len(self.max_heap):
            val = heapq.heappop(self.min_heap)
            heapq.heappush(self.max_heap, -val)
    
    def find_median(self):
        if len(self.max_heap) > len(self.min_heap):
            return float(-self.max_heap[0])
        return (-self.max_heap[0] + self.min_heap[0]) / 2.0

mf = MedianFinder()
for n in [1, 2, 3, 4, 5]:
    mf.add_num(n)
    print(f"Après {n}: médiane = {mf.find_median()}")
# 1.0, 1.5, 2.0, 2.5, 3.0
```

### 4.4 Task Scheduler

```python
import heapq
from collections import Counter

def task_scheduler(tasks, n):
    """
    Temps minimum pour exécuter toutes les tâches avec cooldown n entre deux mêmes tâches.
    Stratégie greedy : toujours exécuter la tâche la plus fréquente disponible.
    """
    freq = Counter(tasks)
    max_heap = [-count for count in freq.values()]
    heapq.heapify(max_heap)
    
    time = 0
    while max_heap:
        cycle = []
        # Essayer d'exécuter n+1 tâches différentes dans une fenêtre
        for _ in range(n + 1):
            if max_heap:
                cycle.append(heapq.heappop(max_heap))
        
        # Remettre les tâches avec fréquence restante
        for count in cycle:
            if count + 1 < 0:  # count est négatif
                heapq.heappush(max_heap, count + 1)
        
        # Si des tâches restent, on a rempli une fenêtre complète de n+1
        # Sinon, on a utilisé seulement len(cycle) unités de temps
        time += (n + 1) if max_heap else len(cycle)
    
    return time

print(task_scheduler(['A','A','A','B','B','B'], 2))  # 8
```

### 4.5 Encodage de Huffman

```python
import heapq
from collections import defaultdict

class HuffmanNode:
    def __init__(self, char, freq):
        self.char = char
        self.freq = freq
        self.left = None
        self.right = None
    
    def __lt__(self, other):
        return self.freq < other.freq

def huffman_encoding(text):
    freq = defaultdict(int)
    for char in text:
        freq[char] += 1
    
    heap = [HuffmanNode(char, f) for char, f in freq.items()]
    heapq.heapify(heap)
    
    while len(heap) > 1:
        left = heapq.heappop(heap)
        right = heapq.heappop(heap)
        merged = HuffmanNode(None, left.freq + right.freq)
        merged.left = left
        merged.right = right
        heapq.heappush(heap, merged)
    
    # Générer les codes
    codes = {}
    def generate_codes(node, code=""):
        if node.char is not None:
            codes[node.char] = code or "0"
            return
        generate_codes(node.left, code + "0")
        generate_codes(node.right, code + "1")
    
    if heap:
        generate_codes(heap[0])
    
    encoded = ''.join(codes[c] for c in text)
    return codes, encoded

codes, encoded = huffman_encoding("aaabbc")
print(f"Codes : {codes}")
print(f"Taux compression : {len(encoded)}/{len('aaabbc')*8} bits")
```

---

## 5. Heap Sort

```python
def heap_sort(arr):
    """
    1. Construire un max-heap (heapify) : O(n)
    2. Extraire le max n fois, placer en fin : O(n log n)
    Total : O(n log n), in-place, non-stable
    """
    n = len(arr)
    
    def sift_down(heap, root, end):
        while True:
            largest = root
            l = 2 * root + 1
            r = 2 * root + 2
            
            if l <= end and heap[l] > heap[largest]:
                largest = l
            if r <= end and heap[r] > heap[largest]:
                largest = r
            
            if largest != root:
                heap[root], heap[largest] = heap[largest], heap[root]
                root = largest
            else:
                break
    
    # Phase 1 : Heapify (construire max-heap) O(n)
    for i in range(n // 2 - 1, -1, -1):
        sift_down(arr, i, n - 1)
    
    # Phase 2 : Extraire le max et placer en fin O(n log n)
    for end in range(n - 1, 0, -1):
        arr[0], arr[end] = arr[end], arr[0]
        sift_down(arr, 0, end - 1)
    
    return arr

arr = [5, 3, 8, 1, 4, 2, 7, 6]
print(heap_sort(arr))  # [1, 2, 3, 4, 5, 6, 7, 8]
```

### Comparaison avec d'autres algorithmes de tri

| Algorithme | Meilleur | Moyen | Pire | Espace | Stable |
|-----------|---------|-------|------|--------|--------|
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | Non |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Oui |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | Non |
| Tim Sort | O(n) | O(n log n) | O(n log n) | O(n) | Oui |

> [!info] Pourquoi Heap Sort est moins utilisé en pratique
> Malgré son O(n log n) garanti et O(1) espace, Heap Sort a de mauvaises performances de cache (les accès mémoire sont non-locaux) et n'est pas stable. Tim Sort (utilisé par Python) lui est supérieur en pratique.

---

## 6. Implémentation en C

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 100

typedef struct {
    int data[MAX_SIZE];
    int size;
} MinHeap;

void swap(int *a, int *b) {
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

void sift_up(MinHeap *h, int i) {
    while (i > 0) {
        int parent = (i - 1) / 2;
        if (h->data[i] < h->data[parent]) {
            swap(&h->data[i], &h->data[parent]);
            i = parent;
        } else {
            break;
        }
    }
}

void sift_down(MinHeap *h, int i) {
    while (1) {
        int smallest = i;
        int l = 2 * i + 1;
        int r = 2 * i + 2;
        
        if (l < h->size && h->data[l] < h->data[smallest])
            smallest = l;
        if (r < h->size && h->data[r] < h->data[smallest])
            smallest = r;
        
        if (smallest != i) {
            swap(&h->data[i], &h->data[smallest]);
            i = smallest;
        } else {
            break;
        }
    }
}

void heap_push(MinHeap *h, int val) {
    if (h->size >= MAX_SIZE) {
        fprintf(stderr, "Heap plein\n");
        return;
    }
    h->data[h->size++] = val;
    sift_up(h, h->size - 1);
}

int heap_pop(MinHeap *h) {
    if (h->size == 0) {
        fprintf(stderr, "Heap vide\n");
        return -1;
    }
    int min_val = h->data[0];
    h->data[0] = h->data[--h->size];
    sift_down(h, 0);
    return min_val;
}

int main(void) {
    MinHeap h = {.size = 0};
    heap_push(&h, 5);
    heap_push(&h, 3);
    heap_push(&h, 8);
    heap_push(&h, 1);
    
    while (h.size > 0) {
        printf("%d ", heap_pop(&h));  /* 1 3 5 8 */
    }
    printf("\n");
    return 0;
}
```

---

## 7. Variantes et structures avancées

### 7.1 D-ary Heap

Un heap avec d enfants par nœud (d = 2 = heap binaire standard).

- **Parent de i** : `(i - 1) // d`
- **Premier enfant de i** : `d * i + 1`

**Avantages** : Sift-down moins profond (hauteur = log_d(n)), meilleure localité de cache. **Inconvénient** : Sift-up plus lent.

```python
class DHeap:
    def __init__(self, d=4):
        self.heap = []
        self.d = d
    
    def _parent(self, i):
        return (i - 1) // self.d
    
    def _children(self, i):
        return range(self.d * i + 1, min(self.d * i + self.d + 1, len(self.heap)))
    
    def push(self, val):
        self.heap.append(val)
        self._sift_up(len(self.heap) - 1)
    
    def _sift_up(self, i):
        while i > 0:
            p = self._parent(i)
            if self.heap[i] < self.heap[p]:
                self.heap[i], self.heap[p] = self.heap[p], self.heap[i]
                i = p
            else:
                break
    
    def pop(self):
        if not self.heap:
            return None
        self.heap[0], self.heap[-1] = self.heap[-1], self.heap[0]
        val = self.heap.pop()
        self._sift_down(0)
        return val
    
    def _sift_down(self, i):
        while True:
            smallest = i
            for c in self._children(i):
                if self.heap[c] < self.heap[smallest]:
                    smallest = c
            if smallest != i:
                self.heap[i], self.heap[smallest] = self.heap[smallest], self.heap[i]
                i = smallest
            else:
                break
```

### 7.2 Fibonacci Heap (mention théorique)

Le **Fibonacci Heap** améliore les complexités amortissées :

| Opération | Binaire | Fibonacci |
|-----------|---------|-----------|
| Insert | O(log n) | O(1) amorti |
| Find-min | O(1) | O(1) |
| Extract-min | O(log n) | O(log n) amorti |
| Decrease-key | O(log n) | O(1) amorti |
| Union | O(n) | O(1) |

> [!info] Fibonacci Heap en pratique
> Le Fibonacci Heap est surtout important **théoriquement** car il permet à Dijkstra d'atteindre O(E + V log V) au lieu de O((E+V) log V). En pratique, les constantes cachées et la complexité d'implémentation le rendent moins performant que le heap binaire pour les inputs de taille courante.

### 7.3 Indexed Priority Queue

Permet de mettre à jour la priorité d'un élément existant — indispensable pour certaines variantes de Dijkstra et Prim.

```python
class IndexedMinHeap:
    """
    Priority queue avec mise à jour de priorité en O(log n).
    Utilise un dictionnaire pour trouver rapidement la position d'un élément.
    """
    def __init__(self):
        self.heap = []      # [(priority, key)]
        self.positions = {} # key -> index dans le heap
    
    def push(self, key, priority):
        self.heap.append((priority, key))
        self.positions[key] = len(self.heap) - 1
        self._sift_up(len(self.heap) - 1)
    
    def decrease_key(self, key, new_priority):
        """Met à jour la priorité vers une valeur plus petite."""
        if key not in self.positions:
            self.push(key, new_priority)
            return
        i = self.positions[key]
        if new_priority < self.heap[i][0]:
            self.heap[i] = (new_priority, key)
            self._sift_up(i)
    
    def _swap(self, i, j):
        self.positions[self.heap[i][1]] = j
        self.positions[self.heap[j][1]] = i
        self.heap[i], self.heap[j] = self.heap[j], self.heap[i]
    
    def _sift_up(self, i):
        while i > 0:
            p = (i - 1) // 2
            if self.heap[i][0] < self.heap[p][0]:
                self._swap(i, p)
                i = p
            else:
                break
    
    def pop(self):
        self._swap(0, len(self.heap) - 1)
        priority, key = self.heap.pop()
        del self.positions[key]
        if self.heap:
            self._sift_down(0)
        return key, priority
    
    def _sift_down(self, i):
        n = len(self.heap)
        while True:
            smallest = i
            l, r = 2*i+1, 2*i+2
            if l < n and self.heap[l][0] < self.heap[smallest][0]:
                smallest = l
            if r < n and self.heap[r][0] < self.heap[smallest][0]:
                smallest = r
            if smallest != i:
                self._swap(i, smallest)
                i = smallest
            else:
                break
```

---

## 8. Récapitulatif des complexités

| Opération | Min/Max-Heap | `heapq` Python |
|-----------|-------------|----------------|
| peek | O(1) | O(1) via `h[0]` |
| push (insert) | O(log n) | O(log n) |
| pop (extract-min) | O(log n) | O(log n) |
| heapify | O(n) | O(n) |
| search | O(n) | O(n) |
| nlargest/nsmallest k | O(n log k) | O(n log k) |
| decrease-key (indexed) | O(log n) | Non supporté |

---

## 9. Exercices pratiques

> [!tip] Exercice 1 — Last Stone Weight
> On a un tas de pierres avec des poids. À chaque tour, on prend les 2 plus lourdes et on les écrase. Si elles ont le même poids, les deux disparaissent. Sinon, le différentiel reste. Quel est le poids de la dernière pierre ?
> **Indice** : Max-heap (négation pour `heapq`), extraire 2, recalculer la différence.

> [!tip] Exercice 2 — Meeting Rooms II
> Étant donné des intervalles de réunions, trouver le nombre minimum de salles nécessaires.
> **Indice** : Trier par heure de début. Min-heap des heures de fin. Si la prochaine réunion commence après la fin de la réunion se terminant le plus tôt, recycler la salle.

> [!tip] Exercice 3 — Top K Frequent Elements
> Trouver les k éléments les plus fréquents dans un tableau.
> **Indice** : Counter + nlargest(k, freq, key=freq.get) ou min-heap de taille k sur les fréquences.

> [!tip] Exercice 4 — Sliding Window Maximum
> Trouver le maximum dans chaque fenêtre glissante de taille k.
> **Indice** : Utiliser un `deque` (monotone queue) plutôt qu'un heap pour O(n) total. Le heap donne O(n log k).

> [!tip] Exercice 5 — Reorganize String
> Réorganiser une chaîne pour qu'aucun deux caractères adjacents soient identiques.
> **Indice** : Max-heap sur les fréquences. À chaque étape, prendre les 2 caractères les plus fréquents.

> [!warning] Exercice avancé — IPO (Capital Minimum)
> W projets disponibles. Chaque projet demande un capital minimum et rapporte un profit. Avec k projets maximum, maximiser le capital final.
> **Indice** : Deux heaps — min-heap sur capital requis, max-heap sur profits disponibles. Débloquer les projets finançables au fur et à mesure.

---

*Liens connexes : [[04 - Arbres Binaires]] · [[05 - Tri et Complexite Algorithmique]] · [[06 - Graphes et Algorithmes de Parcours]] · [[03 - Tables de Hachage]]*
