# 10 - Algorithmes de Chaînes

> [!info] Prérequis
> Ce chapitre utilise les techniques de [[07 - Programmation Dynamique]] (LCS, Edit Distance), les structures de hachage de [[03 - Tables de Hachage]], et les patterns de fenêtre glissante de [[09 - Recherche Binaire et Deux Pointeurs]].

## 1. Problèmes fondamentaux

### 1.1 Palindrome

```python
def is_palindrome(s):
    """O(n) temps, O(1) espace."""
    left, right = 0, len(s) - 1
    while left < right:
        if s[left] != s[right]:
            return False
        left += 1
        right -= 1
    return True

def is_palindrome_alphanumeric(s):
    """Ignorer ponctuation et casse."""
    cleaned = ''.join(c.lower() for c in s if c.isalnum())
    return cleaned == cleaned[::-1]

def longest_palindromic_substring(s):
    """
    Expansion depuis le centre : O(n²) temps, O(1) espace.
    Pour chaque position, essayer d'étendre autour d'un centre.
    """
    def expand(left, right):
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        return s[left+1:right]
    
    result = ""
    for i in range(len(s)):
        # Palindrome de longueur impaire (centre en i)
        odd = expand(i, i)
        # Palindrome de longueur paire (centre entre i et i+1)
        even = expand(i, i+1)
        result = max(result, odd, even, key=len)
    
    return result

print(longest_palindromic_substring("babad"))    # "bab" ou "aba"
print(longest_palindromic_substring("cbbd"))     # "bb"
print(longest_palindromic_substring("racecar"))  # "racecar"
```

### 1.2 Anagramme

```python
from collections import Counter

def is_anagram(s, t):
    """Deux chaînes sont des anagrammes si elles ont les mêmes caractères avec les mêmes fréquences."""
    return Counter(s) == Counter(t)

def find_anagrams(s, p):
    """
    Trouver tous les débuts d'anagrammes de p dans s.
    Sliding window : O(n)
    """
    if len(p) > len(s):
        return []
    
    need = Counter(p)
    window = Counter(s[:len(p)])
    result = [0] if window == need else []
    
    for i in range(len(p), len(s)):
        # Ajouter le nouveau caractère
        window[s[i]] += 1
        # Retirer le caractère sorti de la fenêtre
        left_char = s[i - len(p)]
        window[left_char] -= 1
        if window[left_char] == 0:
            del window[left_char]
        
        if window == need:
            result.append(i - len(p) + 1)
    
    return result

print(find_anagrams("cbaebabacd", "abc"))  # [0, 6]
```

### 1.3 Plus longue sous-chaîne sans répétition

```python
def longest_substring_no_repeat(s):
    """
    Sliding window avec dictionnaire position.
    O(n) temps, O(min(n, alphabet)) espace.
    """
    char_index = {}
    max_len = start = 0
    best_start = 0
    
    for i, c in enumerate(s):
        if c in char_index and char_index[c] >= start:
            start = char_index[c] + 1
        char_index[c] = i
        if i - start + 1 > max_len:
            max_len = i - start + 1
            best_start = start
    
    return s[best_start:best_start + max_len]

print(longest_substring_no_repeat("abcabcbb"))  # "abc"
print(longest_substring_no_repeat("dvdf"))      # "vdf"
```

---

## 2. Sliding Window sur Chaînes

### 2.1 Minimum Window Substring (déjà vu, version complète)

```python
def minimum_window_substring(s, t):
    """
    Trouver la plus petite fenêtre de s contenant tous les chars de t.
    O(|s| + |t|)
    """
    from collections import Counter
    
    need = Counter(t)
    window = {}
    formed = required = len(need)
    
    left = 0
    min_len = float('inf')
    result = (0, 0)
    
    for right, c in enumerate(s):
        window[c] = window.get(c, 0) + 1
        if c in need and window[c] == need[c]:
            formed -= 1
        
        while formed == 0:
            if right - left + 1 < min_len:
                min_len = right - left + 1
                result = (left, right)
            
            lc = s[left]
            window[lc] -= 1
            if lc in need and window[lc] < need[lc]:
                formed += 1
            left += 1
    
    l, r = result
    return s[l:r+1] if min_len != float('inf') else ""

print(minimum_window_substring("ADOBECODEBANC", "ABC"))  # "BANC"
```

### 2.2 Longest Substring with K Distinct Characters

```python
def longest_k_distinct(s, k):
    """
    Sliding window.
    O(n) — chaque caractère est ajouté et retiré au plus une fois.
    """
    if k == 0:
        return 0
    
    window = {}
    left = max_len = 0
    
    for right, c in enumerate(s):
        window[c] = window.get(c, 0) + 1
        
        while len(window) > k:
            lc = s[left]
            window[lc] -= 1
            if window[lc] == 0:
                del window[lc]
            left += 1
        
        max_len = max(max_len, right - left + 1)
    
    return max_len

print(longest_k_distinct("eceba", 2))   # 3 ("ece")
print(longest_k_distinct("aa", 1))      # 2
```

---

## 3. Hashing de Chaînes — Rabin-Karp

### 3.1 Principe du hash polynomial

Le hash d'une chaîne `s` de longueur m est calculé comme :

```
H = s[0]×b^(m-1) + s[1]×b^(m-2) + ... + s[m-1]×b^0   (mod p)

Où b = base (ex : 31 pour lettres minuscules, 256 pour ASCII)
   p = grand nombre premier pour réduire les collisions
```

Le **rolling hash** permet de recalculer le hash en O(1) :
```
H_new = (H_old - s[i-1]×b^(m-1)) × b + s[i+m-1]   (mod p)
```

### 3.2 Implémentation Rabin-Karp

```python
def rabin_karp(text, pattern):
    """
    Recherche de pattern dans text.
    Complexité moyenne : O(n + m)
    Pire cas (nombreuses collisions) : O(nm) → utiliser double hashing
    """
    n, m = len(text), len(pattern)
    if m > n:
        return []
    
    BASE = 256
    MOD = 101  # Nombre premier (plus grand en pratique : 10^9+7)
    
    # Valeur de BASE^(m-1) mod MOD
    h = pow(BASE, m - 1, MOD)
    
    # Hash du pattern et de la première fenêtre
    p_hash = 0
    t_hash = 0
    for i in range(m):
        p_hash = (p_hash * BASE + ord(pattern[i])) % MOD
        t_hash = (t_hash * BASE + ord(text[i])) % MOD
    
    results = []
    
    for i in range(n - m + 1):
        if p_hash == t_hash:
            # Vérification caractère par caractère (éviter faux positifs)
            if text[i:i+m] == pattern:
                results.append(i)
        
        # Calculer le hash de la fenêtre suivante (rolling hash)
        if i < n - m:
            t_hash = (BASE * (t_hash - ord(text[i]) * h) + ord(text[i+m])) % MOD
            if t_hash < 0:
                t_hash += MOD
    
    return results

print(rabin_karp("GEEKS FOR GEEKS", "GEEK"))  # [0, 10]
```

### 3.3 Double Hashing (réduire les faux positifs)

```python
def rabin_karp_double(text, pattern):
    """
    Deux hash indépendants → probabilité de collision O(1/p1 × 1/p2) ≈ 0
    """
    n, m = len(text), len(pattern)
    
    B1, M1 = 31, 10**9 + 7
    B2, M2 = 37, 10**9 + 9
    
    def compute_hash(s, base, mod):
        h = 0
        for c in s:
            h = (h * base + ord(c) - ord('a') + 1) % mod
        return h
    
    p1, p2 = compute_hash(pattern, B1, M1), compute_hash(pattern, B2, M2)
    
    # Calculer les hashes des fenêtres avec rolling hash...
    # (implémentation similaire à ci-dessus, avec deux hashes simultanés)
    results = []
    # Simplifié pour illustration :
    for i in range(n - m + 1):
        w = text[i:i+m]
        if compute_hash(w, B1, M1) == p1 and compute_hash(w, B2, M2) == p2:
            results.append(i)
    
    return results
```

---

## 4. KMP — Knuth-Morris-Pratt

### 4.1 Problème avec la recherche naïve

La recherche naïve est O(nm) dans le pire cas car elle recommence depuis le début du texte après chaque mismatch.

**KMP** prétraite le pattern pour construire une "failure function" qui indique où reprendre la recherche sans reconsidérer les caractères déjà vus.

### 4.2 Failure Function (LPS — Longest Proper Prefix which is also Suffix)

```python
def compute_lps(pattern):
    """
    lps[i] = longueur du plus long préfixe de pattern[:i+1] qui est aussi un suffixe.
    Ex : "AAACAAAA" → lps = [0, 1, 2, 0, 1, 2, 3, 3]
    Ex : "ABCABD"   → lps = [0, 0, 0, 1, 2, 0]
    """
    m = len(pattern)
    lps = [0] * m
    
    length = 0  # Longueur du préfixe-suffixe précédent
    i = 1
    
    while i < m:
        if pattern[i] == pattern[length]:
            length += 1
            lps[i] = length
            i += 1
        elif length > 0:
            # Mismatch après length correspondances
            # Ne pas incrémenter i — réessayer avec lps[length-1]
            length = lps[length - 1]
        else:
            lps[i] = 0
            i += 1
    
    return lps

print(compute_lps("AAACAAAA"))  # [0, 1, 2, 0, 1, 2, 3, 3]
print(compute_lps("ABCABD"))    # [0, 0, 0, 1, 2, 0]
```

### 4.3 Algorithme KMP complet

```python
def kmp_search(text, pattern):
    """
    Recherche de toutes les occurrences de pattern dans text.
    Complexité : O(n + m)
    n = len(text), m = len(pattern)
    """
    n, m = len(text), len(pattern)
    if m == 0:
        return list(range(n + 1))
    
    lps = compute_lps(pattern)
    results = []
    
    i = 0  # Index dans text
    j = 0  # Index dans pattern
    
    while i < n:
        if text[i] == pattern[j]:
            i += 1
            j += 1
        
        if j == m:
            # Pattern trouvé à l'index i - j
            results.append(i - j)
            # Utiliser LPS pour ne pas recommencer à 0
            j = lps[j - 1]
        elif i < n and text[i] != pattern[j]:
            if j > 0:
                j = lps[j - 1]  # Sauter sans réincrémenter i
            else:
                i += 1
    
    return results

print(kmp_search("AABAACAADAABAABA", "AABA"))  # [0, 9, 12]
print(kmp_search("AAAAABAAABA", "AAAA"))        # [0, 1]
```

### 4.4 Trace pas à pas de KMP

```
Text :    A A B A A C A A D A A B A A B A
Pattern : A A B A (lps = [0, 1, 0, 1])

i=0,j=0: A==A → i=1,j=1
i=1,j=1: A==A → i=2,j=2
i=2,j=2: B==B → i=3,j=3
i=3,j=3: A==A → i=4,j=4 → j==m ! Trouvé à [0], j=lps[3]=1
i=4,j=1: A==A → i=5,j=2
i=5,j=2: C!=B → j=lps[1]=1
i=5,j=1: C!=A → j=lps[0]=0
i=5,j=0: C!=A → i=6
...
```

---

## 5. Trie (Prefix Tree)

### 5.1 Structure de données

```python
class TrieNode:
    def __init__(self):
        self.children = {}  # char -> TrieNode
        self.is_end = False
        self.count = 0      # Nombre de mots se terminant ici

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        """O(m) où m = len(word)"""
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end = True
        node.count += 1
    
    def search(self, word):
        """O(m) — True si word existe exactement."""
        node = self.root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_end
    
    def starts_with(self, prefix):
        """O(m) — True si un mot commence par prefix."""
        node = self.root
        for char in prefix:
            if char not in node.children:
                return False
            node = node.children[char]
        return True
    
    def delete(self, word):
        """Supprimer un mot du trie. O(m)"""
        def _delete(node, word, depth):
            if depth == len(word):
                if node.is_end:
                    node.is_end = False
                return len(node.children) == 0
            
            char = word[depth]
            if char not in node.children:
                return False
            
            should_delete_child = _delete(node.children[char], word, depth + 1)
            
            if should_delete_child:
                del node.children[char]
                return len(node.children) == 0 and not node.is_end
            
            return False
        
        _delete(self.root, word, 0)
    
    def autocomplete(self, prefix):
        """Retourner tous les mots commençant par prefix."""
        node = self.root
        for char in prefix:
            if char not in node.children:
                return []
            node = node.children[char]
        
        results = []
        self._dfs(node, prefix, results)
        return results
    
    def _dfs(self, node, current, results):
        if node.is_end:
            results.append(current)
        for char, child in sorted(node.children.items()):
            self._dfs(child, current + char, results)

# Exemple
trie = Trie()
for word in ["apple", "app", "application", "apply", "apt", "banana"]:
    trie.insert(word)

print(trie.search("app"))          # True
print(trie.search("ap"))           # False
print(trie.starts_with("ap"))      # True
print(trie.autocomplete("app"))    # ['app', 'apple', 'application', 'apply']
```

### 5.2 Trie avec tableau (plus rapide pour l'alphabet fixe)

```python
class TrieNodeArray:
    """Utiliser un tableau de 26 pour les lettres minuscules anglaises."""
    def __init__(self):
        self.children = [None] * 26
        self.is_end = False
    
    def get_child(self, c):
        return self.children[ord(c) - ord('a')]
    
    def set_child(self, c, node):
        self.children[ord(c) - ord('a')] = node
```

### 5.3 Applications du Trie

| Application | Description |
|-------------|-------------|
| **Autocomplete** | Suggestions basées sur le préfixe saisi |
| **Spell checker** | Vérifier si un mot existe |
| **IP Routing** | Longest prefix matching dans les tables de routage |
| **Word Search** | Chercher des mots dans une grille (Boggle) |
| **XOR maximum** | Trouver deux nombres avec XOR maximal (trie binaire) |

```python
# XOR maximum avec trie binaire
class BitTrie:
    """Trie sur les bits (de MSB à LSB) pour trouver le XOR maximum."""
    def __init__(self):
        self.root = {}
    
    def insert(self, num, bits=32):
        node = self.root
        for i in range(bits - 1, -1, -1):
            bit = (num >> i) & 1
            if bit not in node:
                node[bit] = {}
            node = node[bit]
    
    def max_xor(self, num, bits=32):
        node = self.root
        result = 0
        for i in range(bits - 1, -1, -1):
            bit = (num >> i) & 1
            desired = 1 - bit  # On veut l'opposé pour maximiser XOR
            if desired in node:
                result |= (1 << i)
                node = node[desired]
            elif bit in node:
                node = node[bit]
        return result

def find_maximum_xor(nums):
    """Maximum XOR de deux éléments dans nums. O(32n) = O(n)"""
    trie = BitTrie()
    for n in nums:
        trie.insert(n)
    return max(trie.max_xor(n) for n in nums)

print(find_maximum_xor([3, 10, 5, 25, 2, 8]))  # 28 (5 XOR 25)
```

---

## 6. Suffix Array

### 6.1 Concept

Le **Suffix Array** est un tableau d'indices des suffixes d'une chaîne, triés lexicographiquement.

```
s = "banana"
Suffixes :
  0: banana
  1: anana
  2: nana
  3: ana
  4: na
  5: a

Tri lexicographique : a, ana, anana, banana, na, nana
Suffix Array : [5, 3, 1, 0, 4, 2]
```

```python
def suffix_array_naive(s):
    """
    Construction naïve : O(n² log n)
    Production : algorithme de Skew O(n) ou DC3, ou doubling O(n log²n)
    """
    n = len(s)
    # Créer les paires (suffixe, index)
    suffixes = [(s[i:], i) for i in range(n)]
    suffixes.sort()
    return [idx for _, idx in suffixes]

def suffix_array_fast(s):
    """
    Construction en O(n log² n) par doubling.
    """
    n = len(s)
    sa = list(range(n))
    rank = [ord(c) for c in s]
    tmp = [0] * n
    
    k = 1
    while k < n:
        def sort_key(i):
            return (rank[i], rank[i + k] if i + k < n else -1)
        
        sa.sort(key=sort_key)
        
        tmp[sa[0]] = 0
        for i in range(1, n):
            tmp[sa[i]] = tmp[sa[i-1]]
            if sort_key(sa[i]) != sort_key(sa[i-1]):
                tmp[sa[i]] += 1
        
        rank = tmp[:]
        k *= 2
    
    return sa

s = "banana"
sa = suffix_array_naive(s)
print(sa)  # [5, 3, 1, 0, 4, 2]
print([s[i:] for i in sa])  # ['a', 'ana', 'anana', 'banana', 'na', 'nana']
```

### 6.2 LCP Array (Longest Common Prefix)

```python
def build_lcp_array(s, sa):
    """
    lcp[i] = longueur du préfixe commun entre sa[i] et sa[i-1].
    Algorithme de Kasai : O(n)
    """
    n = len(s)
    rank = [0] * n
    for i, idx in enumerate(sa):
        rank[idx] = i
    
    lcp = [0] * n
    k = 0
    
    for i in range(n):
        if rank[i] == 0:
            k = 0
            continue
        j = sa[rank[i] - 1]
        while i + k < n and j + k < n and s[i+k] == s[j+k]:
            k += 1
        lcp[rank[i]] = k
        if k > 0:
            k -= 1
    
    return lcp

sa = suffix_array_naive("banana")
lcp = build_lcp_array("banana", sa)
print(lcp)  # [0, 1, 3, 0, 0, 2]
```

### 6.3 Applications du Suffix Array

| Application | Méthode |
|-------------|---------|
| Recherche de pattern | Binary search sur le SA |
| Nombre de sous-chaînes distinctes | n*(n+1)/2 - sum(lcp) |
| Plus longue sous-chaîne répétée | max(lcp) |
| Plus longue sous-chaîne commune (2 textes) | Concaténer + SA + LCP |

---

## 7. Algorithme de Manacher — Palindromes en O(n)

### 7.1 Principe

Manacher trouve tous les palindromes dans une chaîne en **O(n)** en réutilisant les palindromes déjà calculés.

```python
def manacher(s):
    """
    Retourne le plus long palindrome de s en O(n).
    
    Astuce : transformer "abc" en "#a#b#c#" pour unifier pairs/impairs.
    """
    # Transformer la chaîne
    t = '#' + '#'.join(s) + '#'
    n = len(t)
    
    # p[i] = rayon du palindrome centré en i (incluant le centre)
    p = [0] * n
    center = right = 0  # Centre et bord droit du palindrome le plus à droite
    
    for i in range(n):
        if i < right:
            # Utiliser le miroir de i par rapport à center
            mirror = 2 * center - i
            p[i] = min(right - i, p[mirror])
        
        # Expansion autour de i
        left_i, right_i = i - p[i] - 1, i + p[i] + 1
        while left_i >= 0 and right_i < n and t[left_i] == t[right_i]:
            p[i] += 1
            left_i -= 1
            right_i += 1
        
        # Mettre à jour center et right
        if i + p[i] > right:
            center = i
            right = i + p[i]
    
    # Trouver le plus long palindrome
    max_len, center_idx = max((v, i) for i, v in enumerate(p))
    start = (center_idx - max_len) // 2
    return s[start:start + max_len]

print(manacher("babad"))     # "bab" ou "aba"
print(manacher("cbbd"))      # "bb"
print(manacher("racecar"))   # "racecar"
print(manacher("abcba"))     # "abcba"
```

> [!info] Manacher vs Expansion depuis le centre
> L'expansion depuis le centre est O(n²). Manacher évite les recalculs en exploitant la symétrie des palindromes déjà trouvés, réduisant à O(n).

---

## 8. Expressions Régulières — Matcher simplifié

### 8.1 Matcher avec `.` et `*`

```python
def regex_match(s, p):
    """
    Implémentation simplifiée d'un regex matcher.
    Supporte '.' (n'importe quel caractère) et '*' (zéro ou plus du précédent).
    Programmation dynamique.
    
    dp[i][j] = True si s[:i] matche p[:j]
    """
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True  # Chaîne vide matche pattern vide
    
    # Initialiser pour les patterns commençant par X*
    for j in range(1, n + 1):
        if p[j-1] == '*':
            dp[0][j] = dp[0][j-2]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j-1] == '*':
                # Zéro occurrence du caractère précédent
                dp[i][j] = dp[i][j-2]
                # Une ou plusieurs occurrences (si le char précédent matche)
                if p[j-2] == '.' or p[j-2] == s[i-1]:
                    dp[i][j] = dp[i][j] or dp[i-1][j]
            elif p[j-1] == '.' or p[j-1] == s[i-1]:
                dp[i][j] = dp[i-1][j-1]
    
    return dp[m][n]

print(regex_match("aa", "a"))     # False
print(regex_match("aa", "a*"))    # True
print(regex_match("ab", ".*"))    # True
print(regex_match("aab", "c*a*b"))# True
```

### 8.2 Connexion avec les automates finis (NFA/DFA)

```
Pattern  p = "a*b."
États    q0 → q1 → q2 → q3 (acceptant)
         ↑←←←↑
         a*

Un regex peut être compilé en NFA (Thompson's construction) puis
simulé en O(nm) ou converti en DFA en O(2^n) états mais simulation O(m).

Python utilise un moteur NFA avec backtracking → O(2^n) dans le pire cas
mais O(nm) en pratique pour les patterns sans backtracking exponentiel.
```

---

## 9. Récapitulatif des complexités

| Algorithme | Prétraitement | Recherche | Espace |
|-----------|--------------|-----------|--------|
| Naïf | O(1) | O(nm) | O(1) |
| Rabin-Karp | O(m) | O(n+m) moy. | O(1) |
| KMP | O(m) | O(n+m) | O(m) |
| Boyer-Moore | O(m+σ) | O(n/m) moy. | O(m+σ) |
| Aho-Corasick | O(Σm) | O(n+occ) | O(Σm) |
| Suffix Array | O(n log²n) | O(m log n) | O(n) |
| Trie | O(total mots) | O(m) | O(total × σ) |

*σ = taille de l'alphabet, Σm = somme des longueurs des patterns*

---

## 10. Exercices pratiques

> [!tip] Exercice 1 — Group Anagrams
> Étant donné un tableau de chaînes, grouper les anagrammes ensemble.
> **Indice** : Pour chaque chaîne, utiliser la chaîne triée comme clé dans un dictionnaire. `Counter` ou `sorted(s)` comme clé.

> [!tip] Exercice 2 — Repeated DNA Sequences
> Trouver toutes les séquences ADN de 10 lettres apparaissant plus d'une fois.
> **Indice** : Sliding window de taille 10 + HashSet pour détecter les doublons. Rabin-Karp pour O(n).

> [!tip] Exercice 3 — Decode Ways
> Décoder une chaîne de chiffres en lettres (1=A, 2=B, ..., 26=Z). Compter le nombre de décodages possibles.
> **Indice** : DP. `dp[i]` = nombre de décodages de `s[:i]`. Considérer les décodages à 1 chiffre et 2 chiffres.

> [!tip] Exercice 4 — Word Search II (Boggle)
> Dans une grille de lettres, trouver tous les mots d'une liste. Un mot peut utiliser des lettres adjacentes (8 directions), sans réutiliser la même cellule.
> **Indice** : Trie + DFS avec backtracking. Insérer tous les mots dans un trie, DFS depuis chaque cellule.

> [!tip] Exercice 5 — Shortest Palindrome
> Trouver le plus court palindrome en ajoutant des caractères **au début** de la chaîne.
> **Indice** : Trouver le plus long préfixe palindrome avec KMP. Concaténer `s + '#' + reverse(s)`, calculer LPS.

> [!warning] Exercice avancé — Suffix Automaton
> Construire un automate sur les suffixes d'une chaîne en O(n). Cet automate représente tous les sous-chaînes de la chaîne en O(n) nœuds, permettant des requêtes en O(m).
> Applications : nombre de sous-chaînes distinctes, occurrences de patterns, LCS de plusieurs chaînes.

---

## 11. Pièges courants

> [!warning] Piège — Comparaison de strings Python
> `s1 == s2` en Python est O(n) pour des chaînes de longueur n — pas O(1) comme les entiers. Pour des comparaisons fréquentes, hasher d'abord.

> [!warning] Piège — Encodage et Unicode
> Python 3 utilise Unicode nativement. `len("héllo")` = 5, pas 6. Pour les algorithmes sur bytes bruts, utiliser `s.encode()`.

> [!info] Boyer-Moore — Mention
> Boyer-Moore est souvent le plus rapide en pratique (O(n/m) pour les textes naturels) grâce aux heuristiques "bad character" et "good suffix". Non couvert en détail ici mais essentiel pour grep et les moteurs de recherche.

> [!info] Aho-Corasick — Mention
> Extension du KMP pour chercher K patterns simultanément dans un texte. Complexité O(n + Σm + occurrences). Utilisé dans les antivirus, les filtres de contenu, Snort (IDS).

---

*Liens connexes : [[07 - Programmation Dynamique]] · [[03 - Tables de Hachage]] · [[09 - Recherche Binaire et Deux Pointeurs]] · [[04 - Arbres Binaires]]*
