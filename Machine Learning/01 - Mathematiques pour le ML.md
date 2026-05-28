# 01 - Mathématiques pour le Machine Learning

> [!info] Prérequis
> Ce cours couvre les fondements mathématiques indispensables au Machine Learning : algèbre linéaire, calcul différentiel, probabilités et statistiques. Chaque concept est illustré avec du code NumPy fonctionnel.

---

## 1. Algèbre Linéaire

### 1.1 Vecteurs

Un **vecteur** est une liste ordonnée de nombres. En ML, il représente un point dans un espace à $n$ dimensions, ou un exemple avec $n$ features.

$$\vec{v} = \begin{pmatrix} v_1 \\ v_2 \\ \vdots \\ v_n \end{pmatrix}$$

```python
import numpy as np

# Vecteur colonne (array 1D en NumPy)
v = np.array([3.0, 1.0, 4.0, 1.0, 5.0])
print(f"Vecteur : {v}")
print(f"Dimension : {v.shape}")   # (5,)
print(f"Taille : {v.size}")       # 5

# Opérations élémentaires
u = np.array([1.0, 2.0, 3.0, 4.0, 5.0])
print(f"Addition     : {u + v}")
print(f"Soustraction : {u - v}")
print(f"Scalaire x2  : {2 * v}")
print(f"Produit terme à terme : {u * v}")
```

### 1.2 Matrices

Une **matrice** $A$ de taille $m \times n$ possède $m$ lignes et $n$ colonnes. En ML, une matrice de données $X$ a souvent $m$ exemples et $n$ features.

$$A = \begin{pmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \end{pmatrix} \in \mathbb{R}^{2 \times 3}$$

```python
# Matrice 3x3
A = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]], dtype=float)

B = np.array([[9, 8, 7],
              [6, 5, 4],
              [3, 2, 1]], dtype=float)

print(f"Shape de A : {A.shape}")          # (3, 3)
print(f"Addition A + B :\n{A + B}")
print(f"Multiplication terme à terme (Hadamard) :\n{A * B}")
```

### 1.3 Multiplication Matricielle

Le produit $C = AB$ est défini seulement si $A \in \mathbb{R}^{m \times k}$ et $B \in \mathbb{R}^{k \times n}$, donnant $C \in \mathbb{R}^{m \times n}$.

$$C_{ij} = \sum_{k} A_{ik} \cdot B_{kj}$$

> [!warning] Attention
> La multiplication matricielle **n'est pas commutative** : $AB \neq BA$ en général.

```python
A = np.array([[1, 2], [3, 4]])    # 2x2
B = np.array([[5, 6], [7, 8]])    # 2x2

# Produit matriciel
C = A @ B          # opérateur @ (Python 3.5+)
# ou : C = np.dot(A, B)
print(f"A @ B :\n{C}")
# [[1*5+2*7, 1*6+2*8],  = [[19, 22],
#  [3*5+4*7, 3*6+4*8]]     [43, 50]]

# Produit scalaire (dot product) entre deux vecteurs
u = np.array([1, 2, 3])
v = np.array([4, 5, 6])
dot = u @ v   # 1*4 + 2*5 + 3*6 = 32
print(f"Produit scalaire u·v = {dot}")
```

### 1.4 Transposée

La **transposée** $A^T$ échange lignes et colonnes : $(A^T)_{ij} = A_{ji}$.

$$A = \begin{pmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{pmatrix} \Rightarrow A^T = \begin{pmatrix} 1 & 4 \\ 2 & 5 \\ 3 & 6 \end{pmatrix}$$

```python
A = np.array([[1, 2, 3],
              [4, 5, 6]])
print(f"A shape      : {A.shape}")    # (2, 3)
print(f"A^T shape    : {A.T.shape}")  # (3, 2)
print(f"Transposée :\n{A.T}")

# Propriété : (AB)^T = B^T A^T
A2 = np.array([[1, 2], [3, 4]])
B2 = np.array([[5, 6], [7, 8]])
lhs = (A2 @ B2).T
rhs = B2.T @ A2.T
print(f"(AB)^T == B^T A^T : {np.allclose(lhs, rhs)}")
```

### 1.5 Inverse et Déterminant

L'**inverse** $A^{-1}$ vérifie $A A^{-1} = I$ (matrice identité). Il n'existe que si le déterminant est non nul.

Le **déterminant** mesure le "volume" de transformation linéaire. Pour $2 \times 2$ :

$$\det\begin{pmatrix} a & b \\ c & d \end{pmatrix} = ad - bc$$

```python
A = np.array([[2, 1],
              [5, 3]], dtype=float)

det_A = np.linalg.det(A)
print(f"det(A) = {det_A:.4f}")     # 2*3 - 1*5 = 1

inv_A = np.linalg.inv(A)
print(f"A^(-1) :\n{inv_A}")

# Vérification : A * A^(-1) = I
identity = A @ inv_A
print(f"A @ A^(-1) :\n{np.round(identity, 6)}")

# Résoudre le système Ax = b
b = np.array([4, 7], dtype=float)
x = np.linalg.solve(A, b)   # Préférer à inv(A) @ b pour la stabilité numérique
print(f"Solution de Ax = b : {x}")
```

### 1.6 Valeurs Propres et Vecteurs Propres

Pour une matrice carrée $A$, un **vecteur propre** $\vec{v}$ et sa **valeur propre** $\lambda$ vérifient :

$$A\vec{v} = \lambda\vec{v}$$

> [!tip] Utilité en ML
> - **PCA** utilise les valeurs propres de la matrice de covariance pour trouver les directions de variance maximale.
> - Les valeurs propres d'une matrice de Gram indiquent la "taille" des composantes principales.

```python
A = np.array([[4, 2],
              [1, 3]], dtype=float)

eigenvalues, eigenvectors = np.linalg.eig(A)
print(f"Valeurs propres  : {eigenvalues}")
print(f"Vecteurs propres :\n{eigenvectors}")

# Vérification : A @ v = lambda * v
for i in range(len(eigenvalues)):
    lam = eigenvalues[i]
    vec = eigenvectors[:, i]   # colonne i = vecteur propre i
    print(f"A @ v{i} = {A @ vec}")
    print(f"λ{i} * v{i} = {lam * vec}")
    print(f"Égaux : {np.allclose(A @ vec, lam * vec)}")
```

---

## 2. Espaces Vectoriels et Distances

### 2.1 Normes

La **norme** mesure la longueur d'un vecteur.

| Norme | Formule | Alias |
|-------|---------|-------|
| L1 (Manhattan) | $\|\vec{v}\|_1 = \sum_i \|v_i\|$ | Taxi distance |
| L2 (Euclidienne) | $\|\vec{v}\|_2 = \sqrt{\sum_i v_i^2}$ | Distance usuelle |
| L∞ (Chebyshev) | $\|\vec{v}\|_\infty = \max_i \|v_i\|$ | Distance max |

```python
v = np.array([3.0, -4.0, 0.0, 2.0])

norme_L1 = np.linalg.norm(v, ord=1)    # 3 + 4 + 0 + 2 = 9
norme_L2 = np.linalg.norm(v)            # sqrt(9+16+0+4) = sqrt(29)
norme_Linf = np.linalg.norm(v, ord=np.inf)  # max(3,4,0,2) = 4

print(f"||v||_1  = {norme_L1:.4f}")
print(f"||v||_2  = {norme_L2:.4f}")
print(f"||v||_∞  = {norme_Linf:.4f}")

# Normalisation L2 (unit vector)
v_norm = v / norme_L2
print(f"Vecteur normalisé : {v_norm}")
print(f"||v_norm||_2 = {np.linalg.norm(v_norm):.6f}")  # ≈ 1.0
```

### 2.2 Distance Cosinus

La **similarité cosinus** mesure l'angle entre deux vecteurs, indépendamment de leur magnitude — très utilisée en NLP et recommandation.

$$\cos(\theta) = \frac{\vec{u} \cdot \vec{v}}{\|\vec{u}\|_2 \cdot \|\vec{v}\|_2} \in [-1, 1]$$

```python
from sklearn.metrics.pairwise import cosine_similarity

# Deux documents représentés comme vecteurs TF-IDF
doc1 = np.array([1, 1, 0, 1, 0])
doc2 = np.array([1, 0, 1, 1, 1])
doc3 = np.array([0, 0, 0, 0, 1])

def cosine_sim(u, v):
    return np.dot(u, v) / (np.linalg.norm(u) * np.linalg.norm(v))

print(f"sim(doc1, doc2) = {cosine_sim(doc1, doc2):.4f}")  # similaire
print(f"sim(doc1, doc3) = {cosine_sim(doc1, doc3):.4f}")  # différent
print(f"sim(doc2, doc3) = {cosine_sim(doc2, doc3):.4f}")
```

---

## 3. Calcul Différentiel

### 3.1 Dérivée et Gradient

La **dérivée** $f'(x)$ mesure le taux de variation instantané de $f$ en $x$.

Le **gradient** $\nabla f(\vec{x})$ est le vecteur des dérivées partielles :

$$\nabla f(x_1, x_2, \ldots, x_n) = \begin{pmatrix} \frac{\partial f}{\partial x_1} \\ \frac{\partial f}{\partial x_2} \\ \vdots \\ \frac{\partial f}{\partial x_n} \end{pmatrix}$$

> [!info] Intuition géométrique
> Le gradient pointe dans la direction de **montée maximale** de $f$. Pour minimiser $f$, on va dans la direction **opposée** au gradient.

```python
# Gradient numérique par différences finies
def gradient_numerique(f, x, eps=1e-5):
    """Calcule le gradient de f en x par différences finies."""
    grad = np.zeros_like(x, dtype=float)
    for i in range(len(x)):
        x_plus = x.copy().astype(float)
        x_minus = x.copy().astype(float)
        x_plus[i] += eps
        x_minus[i] -= eps
        grad[i] = (f(x_plus) - f(x_minus)) / (2 * eps)
    return grad

# Exemple : f(x, y) = x^2 + 2y^2
def f(x):
    return x[0]**2 + 2 * x[1]**2

# Gradient analytique : [2x, 4y]
point = np.array([3.0, 2.0])
grad_num = gradient_numerique(f, point)
grad_analytique = np.array([2*point[0], 4*point[1]])

print(f"Gradient numérique  : {grad_num}")
print(f"Gradient analytique : {grad_analytique}")
print(f"Erreur : {np.max(np.abs(grad_num - grad_analytique)):.2e}")
```

### 3.2 Règle de la Chaîne

Pour $h(x) = f(g(x))$ : $h'(x) = f'(g(x)) \cdot g'(x)$

En ML : la **backpropagation** est une application directe de la règle de la chaîne pour calculer les gradients à travers les couches d'un réseau de neurones.

$$\frac{\partial L}{\partial w} = \frac{\partial L}{\partial \hat{y}} \cdot \frac{\partial \hat{y}}{\partial z} \cdot \frac{\partial z}{\partial w}$$

```python
# Démonstration règle de la chaîne
# f(x) = sigmoid(w * x + b)
# df/dw = sigmoid'(z) * x   où z = w*x + b

def sigmoid(z):
    return 1 / (1 + np.exp(-z))

def sigmoid_prime(z):
    s = sigmoid(z)
    return s * (1 - s)

w, x, b = 2.0, 3.0, 0.5
z = w * x + b
df_dw = sigmoid_prime(z) * x
print(f"z = {z}")
print(f"σ(z) = {sigmoid(z):.6f}")
print(f"dσ/dw = {df_dw:.6f}")
```

### 3.3 Descente de Gradient

L'algorithme de **descente de gradient** minimise itérativement une fonction de coût $J$ :

$$\theta_{t+1} = \theta_t - \alpha \nabla_\theta J(\theta_t)$$

où $\alpha$ est le **learning rate** (taux d'apprentissage).

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

def gradient_descent(f, grad_f, x_init, lr=0.1, n_iter=50):
    """Descente de gradient simple."""
    x = x_init.copy().astype(float)
    history = [x.copy()]
    
    for i in range(n_iter):
        x = x - lr * grad_f(x)
        history.append(x.copy())
    
    return x, np.array(history)

# Minimiser f(x) = x^2 + 2x + 1 = (x+1)^2  → minimum en x = -1
f_1d = lambda x: x[0]**2 + 2*x[0] + 1
grad_1d = lambda x: np.array([2*x[0] + 2])

x_init = np.array([4.0])
x_min, hist = gradient_descent(f_1d, grad_1d, x_init, lr=0.1, n_iter=50)
print(f"Minimum trouvé : x = {x_min[0]:.6f} (attendu : -1.0)")
print(f"Valeur de f    : {f_1d(x_min):.8f} (attendu : 0.0)")
print(f"Convergence en {len(hist)} itérations")
```

---

## 4. Probabilités

### 4.1 Variables Aléatoires, Espérance, Variance

L'**espérance** (moyenne théorique) d'une variable aléatoire $X$ :
$$\mathbb{E}[X] = \sum_x x \cdot P(X = x) \quad \text{(discret)}$$

La **variance** mesure la dispersion autour de la moyenne :
$$\text{Var}(X) = \mathbb{E}[(X - \mathbb{E}[X])^2] = \mathbb{E}[X^2] - (\mathbb{E}[X])^2$$

```python
np.random.seed(42)

# Simulation d'un dé à 6 faces
n_lancers = 100_000
des = np.random.randint(1, 7, size=n_lancers)

esperance_theorique = (1+2+3+4+5+6) / 6
variance_theorique = np.mean([(k - 3.5)**2 for k in range(1, 7)])

print(f"Espérance théorique : {esperance_theorique:.4f}")
print(f"Espérance empirique : {des.mean():.4f}")
print(f"Variance théorique  : {variance_theorique:.4f}")
print(f"Variance empirique  : {des.var():.4f}")
print(f"Écart-type empirique: {des.std():.4f}")
```

### 4.2 Distributions de Probabilité

#### Distribution Normale (Gaussienne)

$$f(x) = \frac{1}{\sigma\sqrt{2\pi}} \exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)$$

```python
from scipy import stats

mu, sigma = 0, 1   # Normale standard
dist_normale = stats.norm(loc=mu, scale=sigma)

# Règle 68-95-99.7
print(f"P(-1σ < X < 1σ) = {dist_normale.cdf(1) - dist_normale.cdf(-1):.4f}")  # ≈ 68%
print(f"P(-2σ < X < 2σ) = {dist_normale.cdf(2) - dist_normale.cdf(-2):.4f}")  # ≈ 95%
print(f"P(-3σ < X < 3σ) = {dist_normale.cdf(3) - dist_normale.cdf(-3):.4f}")  # ≈ 99.7%

# Générer des données normales
echantillon = np.random.normal(mu, sigma, 10000)
print(f"\nEchantillon N(0,1) :")
print(f"  Moyenne   : {echantillon.mean():.4f}")
print(f"  Ecart-type: {echantillon.std():.4f}")
```

#### Distribution de Bernoulli

$$P(X = k) = p^k (1-p)^{1-k}, \quad k \in \{0, 1\}$$

```python
# Pile ou face biaisé
p = 0.7   # probabilité de pile
n_lancers = 10000
resultats = np.random.binomial(1, p, n_lancers)

print(f"Bernoulli(p={p}) :")
print(f"  Proportion de 1 : {resultats.mean():.4f} (attendu: {p})")
print(f"  Variance empirique: {resultats.var():.4f} (théorique: {p*(1-p):.4f})")
```

#### Distribution Binomiale

$X \sim B(n, p)$ : nombre de succès en $n$ essais indépendants.

$$P(X = k) = \binom{n}{k} p^k (1-p)^{n-k}$$

```python
n, p = 10, 0.3
dist_binom = stats.binom(n, p)

print(f"Binomiale B({n}, {p}) :")
print(f"  E[X] = {dist_binom.mean():.2f}  (= n*p = {n*p})")
print(f"  Var  = {dist_binom.var():.2f}  (= n*p*(1-p) = {n*p*(1-p):.4f})")

# Distribution de probabilité
for k in range(n+1):
    print(f"  P(X={k:2d}) = {dist_binom.pmf(k):.4f}")
```

### 4.3 Théorème de Bayes

$$P(A | B) = \frac{P(B | A) \cdot P(A)}{P(B)}$$

> [!tip] Terminologie
> - $P(A)$ : **prior** — probabilité avant observation
> - $P(A|B)$ : **posterior** — probabilité après observation
> - $P(B|A)$ : **vraisemblance** — probabilité des données sous l'hypothèse
> - $P(B)$ : **evidence** — normalisation

#### Exemple : Classifieur Spam (Naive Bayes)

```python
# Classifieur spam basé sur le mot "gratuit"
# Données historiques
total_emails = 1000
n_spam = 300
n_ham = 700

# P("gratuit" | spam) = 0.8, P("gratuit" | ham) = 0.1
p_spam = n_spam / total_emails          # Prior P(spam)
p_ham = n_ham / total_emails            # Prior P(ham)
p_gratuit_given_spam = 0.8             # Vraisemblance
p_gratuit_given_ham = 0.1              # Vraisemblance

# P("gratuit") = P("gratuit"|spam)*P(spam) + P("gratuit"|ham)*P(ham)
p_gratuit = (p_gratuit_given_spam * p_spam +
             p_gratuit_given_ham * p_ham)

# Bayes : P(spam | "gratuit")
p_spam_given_gratuit = (p_gratuit_given_spam * p_spam) / p_gratuit

print(f"P(spam)                 = {p_spam:.3f}")
print(f"P('gratuit' | spam)     = {p_gratuit_given_spam:.3f}")
print(f"P('gratuit')            = {p_gratuit:.3f}")
print(f"P(spam | 'gratuit')     = {p_spam_given_gratuit:.4f}")
print(f"\nUn email avec 'gratuit' a {p_spam_given_gratuit*100:.1f}% de chance d'être spam.")
```

---

## 5. Statistiques

### 5.1 Corrélation et Covariance

La **covariance** mesure si deux variables varient ensemble :
$$\text{Cov}(X, Y) = \mathbb{E}[(X - \mu_X)(Y - \mu_Y)]$$

Le **coefficient de corrélation de Pearson** (normalisé entre -1 et 1) :
$$r = \frac{\text{Cov}(X, Y)}{\sigma_X \sigma_Y}$$

```python
np.random.seed(42)
n = 200

# Trois paires de variables
x = np.random.normal(0, 1, n)
y_pos = x + 0.3 * np.random.normal(0, 1, n)   # corrélation positive
y_neg = -x + 0.3 * np.random.normal(0, 1, n)  # corrélation négative
y_zero = np.random.normal(0, 1, n)             # corrélation nulle

def pearson_r(a, b):
    return np.cov(a, b)[0, 1] / (np.std(a) * np.std(b))

print(f"r(x, y_positif) = {pearson_r(x, y_pos):.4f}")   # ≈ +0.95
print(f"r(x, y_négatif) = {pearson_r(x, y_neg):.4f}")   # ≈ -0.95
print(f"r(x, y_zéro)    = {pearson_r(x, y_zero):.4f}")  # ≈ 0.0

# Matrice de covariance
data = np.stack([x, y_pos, y_neg], axis=1)
cov_matrix = np.cov(data.T)
print(f"\nMatrice de covariance :\n{np.round(cov_matrix, 3)}")
```

### 5.2 Tests Statistiques Basiques

```python
from scipy import stats

# t-test : deux groupes ont-ils la même moyenne ?
groupe_A = np.random.normal(5.0, 1.0, 50)   # Traitement A
groupe_B = np.random.normal(5.5, 1.0, 50)   # Traitement B

t_stat, p_value = stats.ttest_ind(groupe_A, groupe_B)
print(f"t-statistic = {t_stat:.4f}")
print(f"p-value     = {p_value:.4f}")
if p_value < 0.05:
    print("→ Différence statistiquement significative (α=0.05)")
else:
    print("→ Pas de différence significative (α=0.05)")

# Test de normalité (Shapiro-Wilk)
data_normale = np.random.normal(0, 1, 100)
data_uniforme = np.random.uniform(0, 1, 100)

_, p_norm = stats.shapiro(data_normale)
_, p_unif = stats.shapiro(data_uniforme)
print(f"\nShapiro-Wilk sur données normales   : p = {p_norm:.4f}")
print(f"Shapiro-Wilk sur données uniformes  : p = {p_unif:.4f}")
```

### 5.3 La Matrice de Covariance en ML

La matrice de covariance est fondamentale pour PCA et les modèles gaussiens.

```python
from sklearn.datasets import load_iris

iris = load_iris()
X = iris.data   # 150 x 4

# Centrer les données
X_centered = X - X.mean(axis=0)

# Matrice de covariance (4x4)
Sigma = np.cov(X_centered.T)
print(f"Matrice de covariance (4x4) :\n{np.round(Sigma, 3)}")

# Valeurs propres = variance expliquée par chaque axe
eigenvalues, eigenvectors = np.linalg.eig(Sigma)
idx = np.argsort(eigenvalues)[::-1]    # tri décroissant
eigenvalues = eigenvalues[idx]

variance_totale = eigenvalues.sum()
for i, lam in enumerate(eigenvalues):
    print(f"PC{i+1} : λ = {lam:.4f} ({100*lam/variance_totale:.1f}% de variance)")
```

---

## 6. Récapitulatif des Formules Clés

| Concept | Formule |
|---------|---------|
| Produit scalaire | $\vec{u} \cdot \vec{v} = \sum_i u_i v_i$ |
| Norme L2 | $\|\vec{v}\|_2 = \sqrt{\vec{v} \cdot \vec{v}}$ |
| Similarité cosinus | $\cos\theta = \frac{\vec{u} \cdot \vec{v}}{\|\vec{u}\| \|\vec{v}\|}$ |
| Gradient descent | $\theta \leftarrow \theta - \alpha \nabla J(\theta)$ |
| Bayes | $P(A\|B) = \frac{P(B\|A)P(A)}{P(B)}$ |
| Corrélation Pearson | $r = \frac{\text{Cov}(X,Y)}{\sigma_X \sigma_Y}$ |
| Espérance | $\mathbb{E}[X] = \sum_x x \cdot P(x)$ |
| Variance | $\text{Var}(X) = \mathbb{E}[X^2] - \mathbb{E}[X]^2$ |

---

## 7. Exercices Pratiques

> [!tip] Exercice 1 — Algèbre Linéaire
> 1. Créez deux matrices $A \in \mathbb{R}^{3 \times 4}$ et $B \in \mathbb{R}^{4 \times 2}$ aléatoires. Calculez $C = A \cdot B$ et vérifiez la forme du résultat.
> 2. Calculez les valeurs propres de la matrice $M = A^T A$ (qui est symétrique). Sont-elles toutes positives ? Pourquoi ?
> 3. Résolvez le système $Ax = b$ avec $A = [[3,1],[1,2]]$ et $b = [9, 8]$.

> [!tip] Exercice 2 — Distances
> Implémentez les distances L1, L2 et cosinus entre deux vecteurs SANS utiliser NumPy (seulement Python pur), puis vérifiez avec NumPy.

> [!tip] Exercice 3 — Descente de Gradient
> Implémentez la descente de gradient pour minimiser la fonction de coût MSE d'une régression linéaire : $J(w, b) = \frac{1}{n}\sum_i (y_i - (wx_i + b))^2$. Générez des données synthétiques avec $y = 3x + 2 + \epsilon$ et retrouvez $w \approx 3$, $b \approx 2$.

> [!tip] Exercice 4 — Bayes
> Un test médical détecte une maladie rare (prévalence : 1% de la population). Le test a une sensibilité de 99% (vrai positif) et une spécificité de 95% (vrai négatif). Si votre test est positif, quelle est la probabilité réelle d'avoir la maladie ? (Résultat surprenant : utilisez le théorème de Bayes.)

> [!tip] Exercice 5 — Statistiques
> Sur le dataset Iris (`sklearn.datasets.load_iris`), calculez la matrice de corrélation entre les 4 features. Quelle paire de features est la plus corrélée ? Pourquoi est-ce cohérent biologiquement ?

---

## Liens Utiles

- [[02 - NumPy Pandas et Visualisation]] — Approfondissement NumPy
- [[03 - ML Supervise avec Scikit-Learn]] — Application des maths au ML pratique
- [[04 - Reseaux de Neurones et Deep Learning]] — Backpropagation en détail
- [[06 - ML Non Supervise et Reinforcement Learning]] — PCA (valeurs propres appliquées)

> [!info] Ressources complémentaires
> - **Livre** : "Mathematics for Machine Learning" (Deisenroth, Faisal, Ong) — libre en PDF
> - **Cours** : Gilbert Strang — Linear Algebra (MIT OpenCourseWare)
> - **Cours** : 3Blue1Brown — "Essence of Linear Algebra" (YouTube, visualisations exceptionnelles)
