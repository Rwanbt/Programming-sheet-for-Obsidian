# 06 - Graphes et Algorithmes de Parcours

> [!info] Prérequis
> Ce chapitre suppose la maîtrise des [[04 - Arbres Binaires]] et des notions de complexité vues dans [[05 - Tri et Complexite Algorithmique]]. La [[07 - Programmation Dynamique]] complétera ce chapitre avec des approches optimales sur graphes.

## 1. Définitions fondamentales

Un **graphe** G = (V, E) est une structure composée de :
- **V** (*Vertices*) : un ensemble de sommets (nœuds)
- **E** (*Edges*) : un ensemble d'arêtes (paires de sommets)

### Types de graphes

| Type | Description | Exemple |
|------|-------------|---------|
| **Non-dirigé** | Les arêtes n'ont pas de sens | Réseau routier bidirectionnel |
| **Dirigé (digraphe)** | Les arêtes ont un sens | Twitter (follower) |
| **Pondéré** | Les arêtes ont un poids | Carte avec distances |
| **DAG** | Dirigé Acyclique | Pipeline de compilation |
| **Arbre** | Graphe connexe sans cycle | Hiérarchie de fichiers |
| **Bipartite** | Sommets divisés en 2 groupes indépendants | Matching employé/poste |

> [!tip] Les arbres comme cas particulier
> Un arbre est un graphe connexe non-dirigé avec exactement V-1 arêtes. Tout graphe connexe acyclique est un arbre.

---

## 2. Représentations

### 2.1 Matrice d'adjacence

```python
# Graphe non-dirigé : 4 sommets, arêtes (0,1), (0,2), (1,3), (2,3)
n = 4
adj_matrix = [[0] * n for _ in range(n)]

def add_edge_matrix(adj, u, v, weight=1):
    adj[u][v] = weight
    adj[v][u] = weight  # supprimer pour graphe dirigé

add_edge_matrix(adj_matrix, 0, 1)
add_edge_matrix(adj_matrix, 0, 2)
add_edge_matrix(adj_matrix, 1, 3)
add_edge_matrix(adj_matrix, 2, 3)

# Vérifier si arête existe : O(1)
print(adj_matrix[0][1])  # 1

# Afficher
for row in adj_matrix:
    print(row)
# [0, 1, 1, 0]
# [1, 0, 0, 1]
# [1, 0, 0, 1]
# [0, 1, 1, 0]
```

### 2.2 Liste d'adjacence

```python
from collections import defaultdict

class Graph:
    def __init__(self, directed=False):
        self.adj = defaultdict(list)
        self.directed = directed
    
    def add_edge(self, u, v, weight=1):
        self.adj[u].append((v, weight))
        if not self.directed:
            self.adj[v].append((u, weight))
    
    def neighbors(self, u):
        return self.adj[u]
    
    def vertices(self):
        vertices = set(self.adj.keys())
        for u in list(self.adj.keys()):
            for v, _ in self.adj[u]:
                vertices.add(v)
        return vertices

g = Graph()
g.add_edge(0, 1)
g.add_edge(0, 2)
g.add_edge(1, 3)
g.add_edge(2, 3)
print(g.adj)
# {0: [(1,1),(2,1)], 1: [(0,1),(3,1)], 2: [(0,1),(3,1)], 3: [(1,1),(2,1)]}
```

### 2.3 Comparaison des représentations

| Critère | Matrice d'adjacence | Liste d'adjacence |
|---------|--------------------|--------------------|
| **Espace** | O(V²) | O(V + E) |
| **Tester arête (u,v)** | O(1) | O(deg(u)) |
| **Lister voisins de u** | O(V) | O(deg(u)) |
| **Ajouter arête** | O(1) | O(1) |
| **Supprimer arête** | O(1) | O(deg(u)) |
| **Idéal pour** | Graphes denses (E ≈ V²) | Graphes creux (E << V²) |

> [!warning] Choix de représentation
> En pratique, la liste d'adjacence est presque toujours préférée car la plupart des graphes réels sont **creux** (peu d'arêtes par rapport au nombre de sommets). La matrice est utile pour Floyd-Warshall et les graphes denses.

---

## 3. DFS — Depth-First Search

### Principe
Le DFS explore aussi loin que possible avant de revenir en arrière. Il suit un chemin jusqu'au bout, puis backtracks.

**Complexité : O(V + E)**

### 3.1 Implémentation récursive

```python
def dfs_recursive(graph, start, visited=None):
    if visited is None:
        visited = set()
    
    visited.add(start)
    print(start, end=" ")
    
    for neighbor, _ in graph.adj[start]:
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited)
    
    return visited

# Utilisation
g = Graph()
g.add_edge(0, 1)
g.add_edge(0, 2)
g.add_edge(1, 3)
g.add_edge(2, 4)

dfs_recursive(g, 0)  # 0 1 3 2 4
```

### 3.2 Implémentation itérative

```python
def dfs_iterative(graph, start):
    visited = set()
    stack = [start]
    order = []
    
    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            order.append(node)
            # Ajouter les voisins en ordre inverse pour garder l'ordre naturel
            for neighbor, _ in reversed(graph.adj[node]):
                if neighbor not in visited:
                    stack.append(neighbor)
    
    return order
```

### 3.3 Application : détection de cycle (graphe non-dirigé)

```python
def has_cycle_undirected(graph, vertices):
    visited = set()
    
    def dfs_cycle(node, parent):
        visited.add(node)
        for neighbor, _ in graph.adj[node]:
            if neighbor not in visited:
                if dfs_cycle(neighbor, node):
                    return True
            elif neighbor != parent:
                # Arête vers un sommet déjà visité != parent → cycle
                return True
        return False
    
    for v in vertices:
        if v not in visited:
            if dfs_cycle(v, -1):
                return True
    return False
```

### 3.4 Application : composantes connexes

```python
def connected_components(graph, vertices):
    visited = set()
    components = []
    
    for v in vertices:
        if v not in visited:
            component = []
            stack = [v]
            while stack:
                node = stack.pop()
                if node not in visited:
                    visited.add(node)
                    component.append(node)
                    for neighbor, _ in graph.adj[node]:
                        if neighbor not in visited:
                            stack.append(neighbor)
            components.append(component)
    
    return components

# Exemple : graphe avec 2 composantes
g2 = Graph()
g2.add_edge(0, 1)
g2.add_edge(1, 2)
g2.add_edge(3, 4)  # Composante séparée
print(connected_components(g2, range(5)))
# [[0, 1, 2], [3, 4]]  (ordre variable)
```

---

## 4. BFS — Breadth-First Search

### Principe
Le BFS explore niveau par niveau, en utilisant une **file** (FIFO). Garantit le plus court chemin en nombre d'arêtes (graphe non-pondéré).

**Complexité : O(V + E)**

### 4.1 Implémentation

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    order = []
    
    while queue:
        node = queue.popleft()
        order.append(node)
        
        for neighbor, _ in graph.adj[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    
    return order
```

### 4.2 Plus court chemin non-pondéré

```python
def bfs_shortest_path(graph, start, end):
    if start == end:
        return [start]
    
    visited = {start}
    queue = deque([(start, [start])])  # (nœud, chemin jusqu'ici)
    
    while queue:
        node, path = queue.popleft()
        
        for neighbor, _ in graph.adj[node]:
            if neighbor == end:
                return path + [neighbor]
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    
    return []  # Pas de chemin

# Version optimisée avec dictionnaire parent (évite de copier les listes)
def bfs_shortest_path_v2(graph, start, end):
    parent = {start: None}
    queue = deque([start])
    
    while queue:
        node = queue.popleft()
        if node == end:
            # Reconstruction du chemin
            path = []
            while node is not None:
                path.append(node)
                node = parent[node]
            return path[::-1]
        
        for neighbor, _ in graph.adj[node]:
            if neighbor not in parent:
                parent[neighbor] = node
                queue.append(neighbor)
    
    return []
```

### 4.3 BFS par niveaux

```python
def bfs_levels(graph, start):
    levels = {start: 0}
    queue = deque([start])
    
    while queue:
        node = queue.popleft()
        for neighbor, _ in graph.adj[node]:
            if neighbor not in levels:
                levels[neighbor] = levels[node] + 1
                queue.append(neighbor)
    
    return levels
```

### 4.4 Vérification graphe bipartite

```python
def is_bipartite(graph, vertices):
    color = {}
    
    for start in vertices:
        if start in color:
            continue
        color[start] = 0
        queue = deque([start])
        
        while queue:
            node = queue.popleft()
            for neighbor, _ in graph.adj[node]:
                if neighbor not in color:
                    color[neighbor] = 1 - color[node]
                    queue.append(neighbor)
                elif color[neighbor] == color[node]:
                    return False  # Conflit de coloration → pas bipartite
    
    return True
```

---

## 5. Tri Topologique

### Définition
Ordre linéaire des sommets d'un **DAG** (graphe dirigé acyclique) tel que pour toute arête (u, v), u apparaît avant v. Utilisé pour l'ordonnancement des tâches, les build systems, la résolution de dépendances.

> [!info] Unicité
> Le tri topologique n'est pas unique si le graphe a plusieurs sommets sans dépendances.

### 5.1 Algorithme de Kahn (BFS-based)

```python
from collections import deque

def topological_sort_kahn(graph, vertices):
    """
    Algorithme de Kahn :
    1. Calculer le degré entrant de chaque sommet
    2. Mettre dans la file tous les sommets de degré 0
    3. Retirer un sommet, décrémenter ses successeurs
    4. Si un successeur atteint le degré 0, l'ajouter à la file
    5. Si résultat a moins de V sommets → cycle détecté
    """
    in_degree = {v: 0 for v in vertices}
    
    for u in vertices:
        for v, _ in graph.adj[u]:
            in_degree[v] = in_degree.get(v, 0) + 1
    
    queue = deque([v for v in vertices if in_degree[v] == 0])
    result = []
    
    while queue:
        node = queue.popleft()
        result.append(node)
        
        for neighbor, _ in graph.adj[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
    
    if len(result) != len(vertices):
        raise ValueError("Le graphe contient un cycle — tri topologique impossible")
    
    return result

# Exemple : compilation avec dépendances
# A dépend de rien, B dépend de A, C dépend de A, D dépend de B et C
dag = Graph(directed=True)
vertices = ['A', 'B', 'C', 'D']
dag.add_edge('A', 'B')
dag.add_edge('A', 'C')
dag.add_edge('B', 'D')
dag.add_edge('C', 'D')
print(topological_sort_kahn(dag, vertices))
# ['A', 'B', 'C', 'D'] ou ['A', 'C', 'B', 'D']
```

### 5.2 Tri topologique DFS-based

```python
def topological_sort_dfs(graph, vertices):
    visited = set()
    stack = []
    
    def dfs(node):
        visited.add(node)
        for neighbor, _ in graph.adj[node]:
            if neighbor not in visited:
                dfs(neighbor)
        stack.append(node)  # On ajoute APRÈS avoir visité tous les successeurs
    
    for v in vertices:
        if v not in visited:
            dfs(v)
    
    return stack[::-1]  # Inverser la pile
```

---

## 6. Composantes Fortement Connexes (SCC)

Une **SCC** est un sous-graphe maximal où chaque sommet est accessible depuis tout autre sommet.

### 6.1 Algorithme de Kosaraju

```python
def kosaraju_scc(graph, vertices):
    """
    2 passes DFS :
    1. DFS sur le graphe original → order de fin
    2. DFS sur le graphe transposé dans l'ordre inverse de fin
    """
    # Passe 1 : ordre de fin de traitement
    visited = set()
    finish_order = []
    
    def dfs1(node):
        visited.add(node)
        for neighbor, _ in graph.adj[node]:
            if neighbor not in visited:
                dfs1(neighbor)
        finish_order.append(node)
    
    for v in vertices:
        if v not in visited:
            dfs1(v)
    
    # Construire le graphe transposé
    transposed = Graph(directed=True)
    for u in vertices:
        for v, w in graph.adj[u]:
            transposed.add_edge(v, u, w)
    
    # Passe 2 : DFS sur le transposé dans l'ordre inverse
    visited2 = set()
    sccs = []
    
    def dfs2(node, component):
        visited2.add(node)
        component.append(node)
        for neighbor, _ in transposed.adj[node]:
            if neighbor not in visited2:
                dfs2(neighbor, component)
    
    for node in reversed(finish_order):
        if node not in visited2:
            component = []
            dfs2(node, component)
            sccs.append(component)
    
    return sccs
```

### 6.2 Algorithme de Tarjan (DFS single-pass)

```python
def tarjan_scc(graph, vertices):
    """
    Single-pass DFS avec un stack et des indices de découverte.
    low[v] = plus petit index accessible depuis le sous-arbre de v
    """
    index_counter = [0]
    stack = []
    on_stack = set()
    index = {}
    low = {}
    sccs = []
    
    def strongconnect(v):
        index[v] = low[v] = index_counter[0]
        index_counter[0] += 1
        stack.append(v)
        on_stack.add(v)
        
        for w, _ in graph.adj[v]:
            if w not in index:
                strongconnect(w)
                low[v] = min(low[v], low[w])
            elif w in on_stack:
                low[v] = min(low[v], index[w])
        
        # Si v est une racine de SCC
        if low[v] == index[v]:
            scc = []
            while True:
                w = stack.pop()
                on_stack.remove(w)
                scc.append(w)
                if w == v:
                    break
            sccs.append(scc)
    
    for v in vertices:
        if v not in index:
            strongconnect(v)
    
    return sccs
```

> [!tip] Kosaraju vs Tarjan
> Kosaraju est plus simple à comprendre (2 DFS simples). Tarjan est plus efficace en pratique (single-pass, pas de graphe transposé à construire) mais plus complexe à implémenter correctement.

---

## 7. Dijkstra — Plus Court Chemin Pondéré

**Prérequis : poids non-négatifs**
**Complexité : O((V + E) log V)** avec priority queue

### 7.1 Implémentation avec `heapq`

```python
import heapq
from math import inf

def dijkstra(graph, start, vertices):
    """
    Priority queue (min-heap) : (distance, sommet)
    On extrait toujours le sommet avec la distance minimale connue
    """
    dist = {v: inf for v in vertices}
    dist[start] = 0
    parent = {start: None}
    
    # (distance, sommet)
    pq = [(0, start)]
    
    while pq:
        d, u = heapq.heappop(pq)
        
        # Si on a déjà trouvé un meilleur chemin, ignorer
        if d > dist[u]:
            continue
        
        for v, weight in graph.adj[u]:
            new_dist = dist[u] + weight
            if new_dist < dist[v]:
                dist[v] = new_dist
                parent[v] = u
                heapq.heappush(pq, (new_dist, v))
    
    return dist, parent

def reconstruct_path(parent, start, end):
    path = []
    node = end
    while node is not None:
        path.append(node)
        node = parent.get(node)
    path.reverse()
    if path[0] == start:
        return path
    return []  # Pas de chemin

# Exemple
g = Graph(directed=True)
g.add_edge(0, 1, 4)
g.add_edge(0, 2, 1)
g.add_edge(2, 1, 2)
g.add_edge(1, 3, 1)
g.add_edge(2, 3, 5)

dist, parent = dijkstra(g, 0, range(4))
print(dist)    # {0: 0, 1: 3, 2: 1, 3: 4}
print(reconstruct_path(parent, 0, 3))  # [0, 2, 1, 3]
```

> [!warning] Poids négatifs
> Dijkstra ne fonctionne **pas** avec des poids négatifs. Utiliser Bellman-Ford dans ce cas.

---

## 8. Bellman-Ford — Poids Négatifs

**Complexité : O(V × E)**

```python
def bellman_ford(graph, start, vertices, edges):
    """
    edges : liste de tuples (u, v, weight)
    Relaxe toutes les arêtes V-1 fois.
    Après V-1 itérations, si on peut encore relaxer → cycle négatif.
    """
    dist = {v: inf for v in vertices}
    dist[start] = 0
    parent = {start: None}
    
    # V-1 itérations de relaxation
    for _ in range(len(vertices) - 1):
        updated = False
        for u, v, weight in edges:
            if dist[u] != inf and dist[u] + weight < dist[v]:
                dist[v] = dist[u] + weight
                parent[v] = u
                updated = True
        if not updated:
            break  # Convergence anticipée
    
    # Vérification des cycles négatifs
    for u, v, weight in edges:
        if dist[u] != inf and dist[u] + weight < dist[v]:
            raise ValueError("Cycle négatif détecté")
    
    return dist, parent

# Utilisation avec liste d'arêtes
edges = [(0, 1, 4), (0, 2, 1), (2, 1, -3), (1, 3, 1)]
vertices = [0, 1, 2, 3]
dist, _ = bellman_ford(None, 0, vertices, edges)
print(dist)  # {0: 0, 1: -2, 2: 1, 3: -1}
```

---

## 9. Floyd-Warshall — All-Pairs Shortest Paths

**Complexité : O(V³)**

```python
def floyd_warshall(n, edges):
    """
    dist[i][j] = plus court chemin de i à j
    Programmation dynamique : dist[i][j] via le sommet intermédiaire k
    """
    dist = [[inf] * n for _ in range(n)]
    
    for i in range(n):
        dist[i][i] = 0
    
    for u, v, weight in edges:
        dist[u][v] = weight
    
    # Essayer chaque sommet comme intermédiaire
    for k in range(n):
        for i in range(n):
            for j in range(n):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
    
    # Détecter les cycles négatifs
    for i in range(n):
        if dist[i][i] < 0:
            raise ValueError(f"Cycle négatif détecté via {i}")
    
    return dist

# Exemple
edges = [(0,1,3), (0,2,8), (1,2,-4), (2,3,2), (3,1,1), (3,0,-5)]
n = 4
dist = floyd_warshall(n, edges)
for row in dist:
    print([x if x != inf else '∞' for x in row])
```

---

## 10. A* — Recherche Heuristique

```python
import heapq

def astar(graph, start, goal, heuristic, vertices):
    """
    f(n) = g(n) + h(n)
    g(n) = coût réel depuis le départ
    h(n) = heuristique admissible (ne surestime jamais le coût réel)
    Exemple d'heuristique admissible : distance de Manhattan pour grilles
    """
    g_cost = {v: inf for v in vertices}
    g_cost[start] = 0
    parent = {start: None}
    
    # (f_cost, g_cost, sommet)
    open_set = [(heuristic(start, goal), 0, start)]
    closed = set()
    
    while open_set:
        f, g, current = heapq.heappop(open_set)
        
        if current == goal:
            path = []
            node = goal
            while node is not None:
                path.append(node)
                node = parent[node]
            return path[::-1], g_cost[goal]
        
        if current in closed:
            continue
        closed.add(current)
        
        for neighbor, weight in graph.adj[current]:
            if neighbor in closed:
                continue
            new_g = g_cost[current] + weight
            if new_g < g_cost[neighbor]:
                g_cost[neighbor] = new_g
                parent[neighbor] = current
                f_score = new_g + heuristic(neighbor, goal)
                heapq.heappush(open_set, (f_score, new_g, neighbor))
    
    return [], inf  # Pas de chemin

# Heuristique Manhattan pour grille 2D
def manhattan(a, b):
    return abs(a[0] - b[0]) + abs(a[1] - b[1])
```

> [!tip] Admissibilité de l'heuristique
> Une heuristique est **admissible** si elle ne surestime jamais le coût réel. Si h(n) = 0 pour tous n, A* se comporte comme Dijkstra. La distance de Manhattan est admissible sur grilles sans diagonales.

---

## 11. Arbres Couvrants Minimaux (MST)

### 11.1 Union-Find (Disjoint Set Union)

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n
    
    def find(self, x):
        # Path compression
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False  # Déjà dans le même ensemble
        # Union by rank
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        self.components -= 1
        return True
    
    def connected(self, x, y):
        return self.find(x) == self.find(y)
```

### 11.2 Algorithme de Kruskal

```python
def kruskal_mst(n, edges):
    """
    Trie les arêtes par poids, ajoute les arêtes qui ne créent pas de cycle.
    Complexité : O(E log E)
    """
    edges.sort(key=lambda e: e[2])  # Trier par poids
    uf = UnionFind(n)
    mst = []
    total_weight = 0
    
    for u, v, weight in edges:
        if uf.union(u, v):
            mst.append((u, v, weight))
            total_weight += weight
            if len(mst) == n - 1:
                break  # MST complet
    
    return mst, total_weight

# Exemple
edges = [(0,1,4), (0,2,3), (1,2,1), (1,3,2), (2,3,4)]
mst, weight = kruskal_mst(4, edges)
print(f"MST: {mst}, poids total: {weight}")
# MST: [(1,2,1), (1,3,2), (0,2,3)], poids total: 6
```

### 11.3 Algorithme de Prim

```python
import heapq

def prim_mst(graph, start, vertices):
    """
    Greedy : toujours ajouter l'arête la moins chère qui étend l'arbre.
    Complexité : O((V + E) log V) avec priority queue
    """
    in_mst = set()
    min_edge = {v: inf for v in vertices}
    min_edge[start] = 0
    parent = {start: None}
    
    pq = [(0, start)]
    mst_edges = []
    total_weight = 0
    
    while pq:
        weight, u = heapq.heappop(pq)
        
        if u in in_mst:
            continue
        
        in_mst.add(u)
        total_weight += weight
        if parent[u] is not None:
            mst_edges.append((parent[u], u, weight))
        
        for v, w in graph.adj[u]:
            if v not in in_mst and w < min_edge[v]:
                min_edge[v] = w
                parent[v] = u
                heapq.heappush(pq, (w, v))
    
    return mst_edges, total_weight
```

---

## 12. Récapitulatif des complexités

| Algorithme | Temps | Espace | Utilisation |
|------------|-------|--------|-------------|
| DFS | O(V+E) | O(V) | Cycle, SCC, Topo sort |
| BFS | O(V+E) | O(V) | Plus court chemin non-pondéré |
| Dijkstra | O((V+E)logV) | O(V) | SSSP poids positifs |
| Bellman-Ford | O(VE) | O(V) | SSSP avec poids négatifs |
| Floyd-Warshall | O(V³) | O(V²) | All-pairs shortest path |
| A* | O(E logV) | O(V) | Chemin avec heuristique |
| Kruskal | O(E logE) | O(V) | MST |
| Prim | O((V+E)logV) | O(V) | MST |
| Kosaraju | O(V+E) | O(V) | Composantes fortement connexes |
| Tarjan | O(V+E) | O(V) | Composantes fortement connexes |

---

## 13. Exercices pratiques

> [!tip] Exercice 1 — Nombre d'îles
> Étant donné une grille 2D de '1' (terre) et '0' (eau), compter le nombre d'îles. Une île est entourée d'eau et formée en connectant des terres adjacentes (horizontal/vertical).
> **Indice** : DFS/BFS depuis chaque '1' non visité, marquer tous les '1' connexes comme visités.

> [!tip] Exercice 2 — Course de mots (Word Ladder)
> Transformer un mot en un autre en changeant une lettre à la fois, chaque mot intermédiaire devant exister dans un dictionnaire. Trouver la transformation la plus courte.
> **Indice** : BFS où les nœuds sont les mots et les arêtes relient les mots à distance 1.

> [!tip] Exercice 3 — Réseau social
> Étant donné un réseau social (graphe non-dirigé), trouver le nombre de degrés de séparation entre deux personnes (six degrés de Kevin Bacon).
> **Indice** : BFS bidirectionnel pour optimiser.

> [!tip] Exercice 4 — Ordonnancement de cours
> Étant donné des cours et leurs prérequis (ex : cours B nécessite cours A), déterminer si tous les cours peuvent être suivis et dans quel ordre.
> **Indice** : Modéliser comme un DAG, appliquer le tri topologique de Kahn. Si un cycle existe, c'est impossible.

> [!tip] Exercice 5 — Chemin le moins cher avec K escales
> Trouver le vol le moins cher de src à dst avec au maximum k escales.
> **Indice** : Bellman-Ford modifié avec k itérations, ou BFS avec état (nœud, escales_restantes).

> [!warning] Exercice avancé — Median en graphe
> Étant donné un graphe pondéré, trouver le sommet qui minimise la somme des distances vers tous les autres sommets. Comparer les approches Dijkstra depuis chaque sommet vs Floyd-Warshall.

---

## 14. Patterns et pièges courants

> [!warning] Piège — Revisiter des nœuds
> Toujours utiliser un ensemble `visited` pour éviter les boucles infinies dans les graphes avec cycles.

> [!warning] Piège — Graphes déconnectés
> Ne pas oublier de lancer DFS/BFS depuis chaque sommet non visité pour couvrir toutes les composantes connexes.

> [!info] Pattern — Graphe implicite
> Beaucoup de problèmes (grilles, puzzles, états) peuvent être modélisés comme des graphes sans les construire explicitement. Les "arêtes" sont les transitions valides entre états.

> [!info] Pattern — BFS vs DFS pour trouver un chemin
> - **BFS** : garanti d'être le plus court en nombre d'arêtes (utile pour puzzles, plus petit nombre de mouvements)
> - **DFS** : trouve un chemin existant, pas nécessairement le plus court (utile pour exploration, backtracking)

---

*Liens connexes : [[04 - Arbres Binaires]] · [[05 - Tri et Complexite Algorithmique]] · [[07 - Programmation Dynamique]] · [[08 - Heaps et Priority Queues]]*
