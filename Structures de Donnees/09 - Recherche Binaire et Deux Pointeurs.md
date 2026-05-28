# 09 - Recherche Binaire et Deux Pointeurs

> [!info] Prérequis
> Ce chapitre nécessite la maîtrise des tableaux et des listes chaînées ([[01 - Listes Chainees]]). Il s'appuie sur les notions de complexité de [[05 - Tri et Complexite Algorithmique]] et sert de fondation aux optimisations de [[07 - Programmation Dynamique]].

## 1. Recherche Binaire Classique

### 1.1 Principe

La **recherche binaire** divise l'espace de recherche en deux à chaque étape. Pré-requis : **le tableau doit être trié**.

```
Tableau : [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
Rechercher : 11

Étape 1 : left=0, right=9, mid=4 → arr[4]=9 < 11 → left=5
Étape 2 : left=5, right=9, mid=7 → arr[7]=15 > 11 → right=6
Étape 3 : left=5, right=6, mid=5 → arr[5]=11 == 11 → Trouvé à l'index 5
```

**Complexité : O(log n) — exponentiel mieux qu'une recherche linéaire O(n)**

### 1.2 Implémentation itérative

```python
def binary_search(arr, target):
    """
    Retourne l'index de target dans arr (trié), ou -1 si absent.
    """
    left, right = 0, len(arr) - 1
    
    while left <= right:
        # Éviter le débordement d'entier : mid = left + (right - left) // 2
        # En Python les entiers sont illimités, mais bonne habitude en C/Java
        mid = left + (right - left) // 2
        
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1

arr = [1, 3, 5, 7, 9, 11, 13]
print(binary_search(arr, 7))   # 3
print(binary_search(arr, 6))   # -1
```

> [!warning] Bug classique — Overflow d'entier
> En C/Java/C++, `mid = (left + right) / 2` peut dépasser `INT_MAX` si `left + right > 2^31 - 1`. Toujours utiliser `mid = left + (right - left) / 2`. En Python, ce n'est pas un problème car les entiers sont arbitraires, mais c'est une bonne habitude.

### 1.3 Implémentation récursive

```python
def binary_search_recursive(arr, target, left=0, right=None):
    if right is None:
        right = len(arr) - 1
    
    if left > right:
        return -1
    
    mid = left + (right - left) // 2
    
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search_recursive(arr, target, mid + 1, right)
    else:
        return binary_search_recursive(arr, target, left, mid - 1)
```

> [!info] Itératif vs Récursif
> La version **itérative** est préférable en pratique : pas de stack overflow pour de grands tableaux, légèrement plus rapide (pas d'overhead d'appels de fonction). La version récursive est plus proche de la définition mathématique.

---

## 2. Variantes de la Recherche Binaire

### 2.1 Première occurrence (leftmost binary search)

```python
def first_occurrence(arr, target):
    """
    Trouver la première (plus petite) position de target dans arr.
    Quand on trouve target, continuer à chercher à gauche.
    """
    left, right = 0, len(arr) - 1
    result = -1
    
    while left <= right:
        mid = left + (right - left) // 2
        if arr[mid] == target:
            result = mid       # Enregistrer la position
            right = mid - 1   # Continuer à chercher à gauche
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return result

arr = [1, 2, 2, 2, 3, 4]
print(first_occurrence(arr, 2))  # 1 (pas 2 ni 3)
```

### 2.2 Dernière occurrence (rightmost binary search)

```python
def last_occurrence(arr, target):
    """
    Trouver la dernière (plus grande) position de target dans arr.
    """
    left, right = 0, len(arr) - 1
    result = -1
    
    while left <= right:
        mid = left + (right - left) // 2
        if arr[mid] == target:
            result = mid       # Enregistrer la position
            left = mid + 1    # Continuer à chercher à droite
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return result

arr = [1, 2, 2, 2, 3, 4]
print(last_occurrence(arr, 2))  # 3
# Nombre d'occurrences :
print(last_occurrence(arr, 2) - first_occurrence(arr, 2) + 1)  # 3
```

### 2.3 Upper bound et Lower bound

```python
def lower_bound(arr, target):
    """
    Index du premier élément >= target (plus petite position où insérer target).
    Équivalent à bisect_left.
    """
    left, right = 0, len(arr)
    while left < right:
        mid = left + (right - left) // 2
        if arr[mid] < target:
            left = mid + 1
        else:
            right = mid
    return left

def upper_bound(arr, target):
    """
    Index du premier élément > target (position d'insertion pour maintenir l'ordre).
    Équivalent à bisect_right.
    """
    left, right = 0, len(arr)
    while left < right:
        mid = left + (right - left) // 2
        if arr[mid] <= target:
            left = mid + 1
        else:
            right = mid
    return left

arr = [1, 2, 2, 2, 3, 4]
print(lower_bound(arr, 2))  # 1 (premier index où 2 peut être inséré)
print(upper_bound(arr, 2))  # 4 (après le dernier 2)

# Compter les occurrences de target
def count_occurrences(arr, target):
    return upper_bound(arr, target) - lower_bound(arr, target)

print(count_occurrences(arr, 2))  # 3
```

### 2.4 Module `bisect` Python

```python
import bisect

arr = [1, 2, 2, 2, 3, 4]

# bisect_left = lower_bound : premier index où insérer (en préservant l'ordre)
print(bisect.bisect_left(arr, 2))   # 1

# bisect_right = upper_bound : dernier index où insérer
print(bisect.bisect_right(arr, 2))  # 4

# insort : insérer en maintenant l'ordre trié
sorted_list = [1, 3, 5, 7]
bisect.insort(sorted_list, 4)
print(sorted_list)  # [1, 3, 4, 5, 7]

# Recherche avec bisect
def search_bisect(arr, target):
    i = bisect.bisect_left(arr, target)
    if i < len(arr) and arr[i] == target:
        return i
    return -1

# Trouver le plus petit élément > target
def next_greater(arr, target):
    i = bisect.bisect_right(arr, target)
    return arr[i] if i < len(arr) else None

# Trouver le plus grand élément <= target
def prev_smaller_equal(arr, target):
    i = bisect.bisect_right(arr, target) - 1
    return arr[i] if i >= 0 else None
```

---

## 3. Binary Search sur une Réponse

### 3.1 Concept — "Search on the Answer"

Au lieu de chercher une valeur dans un tableau, on cherche dans **l'espace des réponses possibles**. La fonction de décision `feasible(answer)` est **monotone** : si `answer` est valide, toutes les valeurs plus grandes (ou plus petites, selon le sens) le sont aussi.

```
Pattern :
1. Identifier la réponse minimale et maximale possibles
2. Binary search sur cet intervalle
3. Vérifier si une réponse donnée est feasible en O(n)
4. Ajuster left/right selon le résultat
```

### 3.2 Exemple — Allocation de livres (Book Allocation)

**Problème** : N livres avec `pages[i]` pages. M étudiants. Distribuer les livres (contiguïté obligatoire) pour minimiser le maximum de pages alloué à un étudiant.

```python
def book_allocation(pages, m):
    """
    Binary search sur la réponse (max pages par étudiant).
    min_answer = max(pages) (au moins 1 étudiant par livre)
    max_answer = sum(pages) (1 étudiant pour tout)
    """
    def is_feasible(max_pages):
        """Peut-on distribuer en utilisant au plus max_pages par étudiant ?"""
        students = 1
        current = 0
        for p in pages:
            if p > max_pages:
                return False
            if current + p > max_pages:
                students += 1
                current = p
                if students > m:
                    return False
            else:
                current += p
        return True
    
    left, right = max(pages), sum(pages)
    result = right
    
    while left <= right:
        mid = left + (right - left) // 2
        if is_feasible(mid):
            result = mid      # Valide, essayer de minimiser
            right = mid - 1
        else:
            left = mid + 1
    
    return result

print(book_allocation([12, 34, 67, 90], 2))  # 113 (12+34+67 | 90 → max=113)
```

### 3.3 Exemple — Cow Problem (Aggressive Cows)

**Problème** : Placer k vaches dans n stalles, maximiser la distance minimale entre deux vaches.

```python
def aggressive_cows(stalls, k):
    stalls.sort()
    
    def is_feasible(min_dist):
        """Peut-on placer k vaches avec distance minimale min_dist ?"""
        count = 1
        last = stalls[0]
        for i in range(1, len(stalls)):
            if stalls[i] - last >= min_dist:
                count += 1
                last = stalls[i]
                if count >= k:
                    return True
        return False
    
    left, right = 1, stalls[-1] - stalls[0]
    result = 0
    
    while left <= right:
        mid = left + (right - left) // 2
        if is_feasible(mid):
            result = mid      # Valide, essayer de maximiser
            left = mid + 1
        else:
            right = mid - 1
    
    return result

print(aggressive_cows([1, 2, 8, 4, 9], 3))  # 3
```

---

## 4. Binary Search sur Fonctions Monotones

### 4.1 Racine carrée entière

```python
def isqrt(n):
    """Plus grande valeur k telle que k² <= n. O(log n)."""
    if n < 0:
        raise ValueError("Impossible")
    left, right = 0, n
    result = 0
    
    while left <= right:
        mid = left + (right - left) // 2
        if mid * mid <= n:
            result = mid
            left = mid + 1
        else:
            right = mid - 1
    
    return result

print(isqrt(16))   # 4
print(isqrt(17))   # 4
print(isqrt(100))  # 10
```

### 4.2 Tableau tourné (Rotated Sorted Array)

```python
def find_min_rotated(nums):
    """
    Trouver le minimum dans un tableau trié tourné.
    Ex : [4, 5, 6, 7, 0, 1, 2] → 0
    """
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = left + (right - left) // 2
        if nums[mid] > nums[right]:
            # Le minimum est dans la partie droite
            left = mid + 1
        else:
            # Le minimum est à mid ou à gauche
            right = mid
    
    return nums[left]

def search_rotated(nums, target):
    """Chercher target dans un tableau trié tourné. O(log n)."""
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        if nums[mid] == target:
            return mid
        
        # Déterminer quelle moitié est triée
        if nums[left] <= nums[mid]:  # Partie gauche triée
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:  # Partie droite triée
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    
    return -1

nums = [4, 5, 6, 7, 0, 1, 2]
print(find_min_rotated(nums))   # 0
print(search_rotated(nums, 0))  # 4
print(search_rotated(nums, 3))  # -1
```

---

## 5. Technique des Deux Pointeurs

### 5.1 Two Sum sur tableau trié

```python
def two_sum_sorted(arr, target):
    """
    Deux pointeurs : left au début, right à la fin.
    Si la somme est trop grande → déplacer right à gauche
    Si trop petite → déplacer left à droite
    Complexité : O(n)
    """
    left, right = 0, len(arr) - 1
    
    while left < right:
        s = arr[left] + arr[right]
        if s == target:
            return (left, right)
        elif s < target:
            left += 1
        else:
            right -= 1
    
    return None

arr = [1, 2, 3, 4, 5, 6, 7]
print(two_sum_sorted(arr, 9))  # (1, 6) → arr[1]+arr[6] = 2+7 = 9
```

### 5.2 Three Sum

```python
def three_sum(nums):
    """
    Trouver tous les triplets dont la somme est 0.
    Complexité : O(n²)
    """
    nums.sort()
    result = []
    n = len(nums)
    
    for i in range(n - 2):
        # Éviter les doublons pour le premier élément
        if i > 0 and nums[i] == nums[i-1]:
            continue
        
        left, right = i + 1, n - 1
        
        while left < right:
            s = nums[i] + nums[left] + nums[right]
            if s == 0:
                result.append([nums[i], nums[left], nums[right]])
                # Sauter les doublons
                while left < right and nums[left] == nums[left+1]:
                    left += 1
                while left < right and nums[right] == nums[right-1]:
                    right -= 1
                left += 1
                right -= 1
            elif s < 0:
                left += 1
            else:
                right -= 1
    
    return result

print(three_sum([-1, 0, 1, 2, -1, -4]))
# [[-1, -1, 2], [-1, 0, 1]]
```

### 5.3 Container With Most Water

```python
def max_area(height):
    """
    Deux pointeurs : converger vers le centre.
    L'eau contenue = min(height[left], height[right]) × (right - left)
    Toujours déplacer le pointeur de la plus petite hauteur.
    Complexité : O(n)
    """
    left, right = 0, len(height) - 1
    max_water = 0
    
    while left < right:
        water = min(height[left], height[right]) * (right - left)
        max_water = max(max_water, water)
        
        # Déplacer le côté le plus bas
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_water

print(max_area([1, 8, 6, 2, 5, 4, 8, 3, 7]))  # 49
```

---

## 6. Slow + Fast Pointers (Listes Chaînées)

### 6.1 Détection de cycle (Floyd's Algorithm)

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def has_cycle(head):
    """
    Slow : avance de 1, Fast : avance de 2.
    S'il y a un cycle, fast rattrapera slow.
    Complexité : O(n), Espace : O(1)
    """
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False

def detect_cycle_start(head):
    """
    Trouver le premier nœud du cycle.
    Propriété mathématique : après la détection, replacer slow à head.
    Déplacer les deux d'un pas jusqu'à la rencontre.
    """
    slow = fast = head
    has_cycle = False
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            has_cycle = True
            break
    
    if not has_cycle:
        return None
    
    slow = head  # Replacer slow au départ
    while slow != fast:
        slow = slow.next
        fast = fast.next
    
    return slow  # Début du cycle
```

### 6.2 Trouver le milieu d'une liste chaînée

```python
def find_middle(head):
    """
    Slow avance de 1, fast de 2.
    Quand fast atteint la fin, slow est au milieu.
    """
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow

# Pour une liste de taille paire, retourne le 2ème des deux nœuds du milieu
# Ex : 1->2->3->4 → retourne 3
# Ex : 1->2->3->4->5 → retourne 3
```

### 6.3 Trouver le Kème nœud depuis la fin

```python
def kth_from_end(head, k):
    """
    Avancer fast de k positions, puis avancer les deux de 1.
    Quand fast atteint None, slow est au Kème depuis la fin.
    """
    fast = slow = head
    
    for _ in range(k):
        if not fast:
            return None  # Liste trop courte
        fast = fast.next
    
    while fast:
        slow = slow.next
        fast = fast.next
    
    return slow
```

---

## 7. Sliding Window

### 7.1 Sous-tableau de taille k — maximum

```python
from collections import deque

def max_sliding_window(nums, k):
    """
    Utiliser un deque monotone décroissant (indices).
    Le front contient toujours l'index du maximum courant.
    Complexité : O(n)
    """
    dq = deque()  # Indices, décroissants par valeur
    result = []
    
    for i, num in enumerate(nums):
        # Supprimer les éléments hors de la fenêtre
        while dq and dq[0] < i - k + 1:
            dq.popleft()
        
        # Supprimer les éléments plus petits que num (inutiles)
        while dq and nums[dq[-1]] < num:
            dq.pop()
        
        dq.append(i)
        
        if i >= k - 1:
            result.append(nums[dq[0]])
    
    return result

print(max_sliding_window([1, 3, -1, -3, 5, 3, 6, 7], 3))
# [3, 3, 5, 5, 6, 7]
```

### 7.2 Plus longue sous-chaîne sans répétition

```python
def length_of_longest_substring(s):
    """
    Sliding window avec ensemble des caractères courants.
    Complexité : O(n)
    """
    char_index = {}
    max_len = 0
    left = 0
    
    for right, char in enumerate(s):
        if char in char_index and char_index[char] >= left:
            left = char_index[char] + 1
        char_index[char] = right
        max_len = max(max_len, right - left + 1)
    
    return max_len

print(length_of_longest_substring("abcabcbb"))  # 3 ("abc")
print(length_of_longest_substring("pwwkew"))    # 3 ("wke")
```

### 7.3 Minimum Window Substring

```python
from collections import Counter

def min_window_substring(s, t):
    """
    Plus petite fenêtre de s contenant tous les caractères de t.
    Sliding window avec compteur.
    Complexité : O(|s| + |t|)
    """
    if not t or not s:
        return ""
    
    need = Counter(t)
    have = {}
    formed = 0       # Nombre de caractères de t satisfaits
    required = len(need)
    
    left = 0
    min_len = float('inf')
    result = ""
    
    for right, char in enumerate(s):
        have[char] = have.get(char, 0) + 1
        
        # Vérifier si ce caractère est complètement satisfait
        if char in need and have[char] == need[char]:
            formed += 1
        
        # Rétrécir la fenêtre depuis la gauche
        while formed == required:
            # Mettre à jour le résultat
            if right - left + 1 < min_len:
                min_len = right - left + 1
                result = s[left:right+1]
            
            # Rétrécir
            left_char = s[left]
            have[left_char] -= 1
            if left_char in need and have[left_char] < need[left_char]:
                formed -= 1
            left += 1
    
    return result

print(min_window_substring("ADOBECODEBANC", "ABC"))  # "BANC"
```

### 7.4 Sous-tableau avec somme = k

```python
def subarray_sum_k(nums, k):
    """
    Nombre de sous-tableaux dont la somme vaut exactement k.
    Utiliser la somme préfixe et un dictionnaire.
    Complexité : O(n)
    """
    prefix_sums = {0: 1}  # {somme_préfixe: nombre_d'occurrences}
    current_sum = 0
    count = 0
    
    for num in nums:
        current_sum += num
        # S'il existe un préfixe de somme (current_sum - k), on a trouvé un sous-tableau
        count += prefix_sums.get(current_sum - k, 0)
        prefix_sums[current_sum] = prefix_sums.get(current_sum, 0) + 1
    
    return count

print(subarray_sum_k([1, 1, 1], 2))  # 2
print(subarray_sum_k([1, 2, 3], 3))  # 2 ([1,2] et [3])
```

---

## 8. Récapitulatif et choix de technique

| Problème | Technique | Complexité |
|----------|-----------|------------|
| Chercher dans tableau trié | Binary Search | O(log n) |
| Chercher dans espace de réponses | BS sur réponse | O(n log(max-min)) |
| Two Sum (trié) | Deux pointeurs | O(n) |
| Three Sum | Tri + deux pointeurs | O(n²) |
| Cycle dans liste chaînée | Floyd (slow+fast) | O(n) |
| Milieu de liste | Slow + Fast | O(n) |
| Max/Min de fenêtre glissante | Deque monotone | O(n) |
| Sous-chaîne sans répétition | Sliding window | O(n) |
| Minimum Window Substring | Sliding window + Counter | O(n) |
| Sous-tableau somme = k | Somme préfixe + HashMap | O(n) |

### Choisir entre les techniques

```
Le tableau est trié ET on cherche une valeur précise ?
  → Binary Search classique

Le problème demande un "minimum/maximum de x tel que y est possible" ?
  → Binary search sur la réponse + fonction feasible

Deux éléments dans un tableau trié avec une contrainte ?
  → Two Pointers (left + right)

Sous-tableau ou sous-chaîne contiguë avec une contrainte ?
  → Sliding window (avec deque ou counter)

Problème sur liste chaînée (cycle, milieu, position) ?
  → Slow + Fast Pointers
```

---

## 9. Exercices pratiques

> [!tip] Exercice 1 — Search in 2D Matrix
> Matrice m×n triée (chaque ligne triée, premier élément de chaque ligne > dernier de la ligne précédente). Chercher une valeur cible.
> **Indice** : Traiter la matrice comme un tableau 1D de taille m*n. `row = index // n`, `col = index % n`.

> [!tip] Exercice 2 — Find Peak Element
> Trouver un élément "pic" (plus grand que ses voisins) dans un tableau non trié. Complexité exigée : O(log n).
> **Indice** : Binary search — si `arr[mid] < arr[mid+1]`, le pic est à droite. Sinon, il est à gauche ou en mid.

> [!tip] Exercice 3 — Trapping Rain Water
> Étant donné un tableau de hauteurs, calculer combien d'eau peut être piégée après la pluie.
> **Indice** : Two pointers. `left_max` et `right_max`. Si `left_max < right_max`, l'eau piégée à left = `left_max - height[left]`.

> [!tip] Exercice 4 — Longest Subarray With Sum ≤ K
> Trouver la longueur du plus long sous-tableau dont la somme est ≤ k.
> **Indice** : Sliding window. Avancer right toujours, reculer left quand la somme dépasse k.

> [!tip] Exercice 5 — Painter's Partition
> k peintres, n planches avec des longueurs. Chaque peintre peint les planches de façon contiguë à une vitesse de 1 unité/temps. Minimiser le temps total.
> **Indice** : Identique à Book Allocation — binary search sur la réponse.

> [!warning] Exercice avancé — Median of Two Sorted Arrays
> Trouver la médiane de deux tableaux triés en O(log(min(m,n))).
> **Indice** : Binary search sur le tableau le plus petit. Partitionner les deux tableaux de façon à ce que la moitié gauche (fusionnée) contienne les min(m,n) plus petits éléments.

---

## 10. Pièges courants

> [!warning] Piège — Condition d'arrêt de la boucle
> `while left <= right` pour chercher un élément précis.
> `while left < right` pour rétrécir vers une frontière.
> Ne pas confondre ces deux cas — ils produisent des comportements différents.

> [!warning] Piège — Sliding window : quand rétrécir ?
> Si la contrainte est "au plus k éléments distincts" → rétrécir tant que la contrainte est violée.
> Si la contrainte est "exactement k" → `exactly(k) = atMost(k) - atMost(k-1)`.

> [!info] Pattern — Two Pointers sur tableau non trié
> Les deux pointeurs ne fonctionnent pas directement sur un tableau non trié pour des sommes. Trier d'abord en O(n log n) ou utiliser une HashMap en O(n).

---

*Liens connexes : [[01 - Listes Chainees]] · [[05 - Tri et Complexite Algorithmique]] · [[07 - Programmation Dynamique]] · [[10 - Algorithmes de Chaines]]*
