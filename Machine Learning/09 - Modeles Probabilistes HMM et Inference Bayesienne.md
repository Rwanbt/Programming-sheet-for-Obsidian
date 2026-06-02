# Modèles Probabilistes — HMM et Inférence Bayésienne

Les modèles probabilistes constituent le socle théorique de nombreux algorithmes de Machine Learning modernes. Ce cours couvre les fondements de l'inférence bayésienne, les classificateurs Naïf Bayes, et les Modèles de Markov Cachés (HMM) — des outils indispensables pour traiter des données séquentielles, du texte et des signaux temporels. À l'issue de ce cours, vous serez capable d'implémenter un algorithme de Viterbi from scratch, d'entraîner un HMM avec hmmlearn, et de comprendre l'intuition derrière l'estimation bayésienne.

---

## Table des matières

1. [Probabilités Avancées](#1-probabilités-avancées)
2. [Naïf Bayes](#2-naïf-bayes)
3. [Modèles de Markov Cachés (HMM)](#3-modèles-de-markov-cachés-hmm)
4. [Applications](#4-applications)
5. [Exercices](#5-exercices)

---

## 1. Probabilités Avancées

### 1.1 Distributions de probabilité fondamentales

#### Distribution Gaussienne (Normale)

$$\mathcal{N}(x \mid \mu, \sigma^2) = \frac{1}{\sqrt{2\pi\sigma^2}} \exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)$$

**Paramètres :**
- $\mu$ : moyenne (centre de la distribution)
- $\sigma^2$ : variance (dispersion)

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import stats

# Définir une gaussienne
mu, sigma = 0, 1
x = np.linspace(-4, 4, 200)
pdf = stats.norm.pdf(x, mu, sigma)
cdf = stats.norm.cdf(x, mu, sigma)

# Visualisation
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(x, pdf, 'b-', linewidth=2)
axes[0].fill_between(x, pdf, alpha=0.3)
axes[0].set_title('PDF — Densité de probabilité')
axes[0].set_xlabel('x')
axes[0].set_ylabel('p(x)')

axes[1].plot(x, cdf, 'r-', linewidth=2)
axes[1].set_title('CDF — Fonction de répartition')
axes[1].set_xlabel('x')
axes[1].set_ylabel('P(X ≤ x)')

plt.tight_layout()
plt.show()

# Générer des échantillons
samples = np.random.normal(mu, sigma, 1000)
print(f"Moyenne empirique : {samples.mean():.4f} (attendu : {mu})")
print(f"Écart-type empirique : {samples.std():.4f} (attendu : {sigma})")
```

#### Distribution de Bernoulli

$$\text{Bernoulli}(x \mid p) = p^x (1-p)^{1-x}, \quad x \in \{0, 1\}$$

**Interprétation :** Modélise le résultat d'un seul essai binaire (succès/échec).

```python
from scipy.stats import bernoulli

p = 0.7  # probabilité de succès
rv = bernoulli(p)

print(f"P(X=1) = {rv.pmf(1):.2f}")
print(f"P(X=0) = {rv.pmf(0):.2f}")
print(f"Espérance : {rv.mean():.2f}")
print(f"Variance : {rv.var():.4f}")

# Simulation
n_trials = 10000
samples = rv.rvs(n_trials)
print(f"\nFréquence empirique de succès : {samples.mean():.4f}")
```

#### Distribution Multinomiale

$$P(x_1, \ldots, x_k \mid n, p_1, \ldots, p_k) = \frac{n!}{x_1! \cdots x_k!} \prod_{i=1}^k p_i^{x_i}$$

**Contraintes :** $\sum_{i=1}^k p_i = 1$ et $\sum_{i=1}^k x_i = n$

```python
from scipy.stats import multinomial

# Modèle : lancer un dé à 6 faces 10 fois
n = 10
p = [1/6] * 6  # dé équilibré

rv = multinomial(n, p)

# Probabilité d'obtenir exactement [2, 1, 2, 1, 2, 2]
outcome = [2, 1, 2, 1, 2, 2]
print(f"P({outcome}) = {rv.pmf(outcome):.6f}")

# Simulation de 5 lancers
samples = rv.rvs(5)
print("\nSimulations :")
for i, s in enumerate(samples):
    print(f"  Tirage {i+1}: {s} (total = {s.sum()})")
```

#### Distribution Beta

$$\text{Beta}(p \mid \alpha, \beta) = \frac{p^{\alpha-1}(1-p)^{\beta-1}}{B(\alpha, \beta)}$$

où $B(\alpha, \beta) = \frac{\Gamma(\alpha)\Gamma(\beta)}{\Gamma(\alpha+\beta)}$ est la fonction Beta.

**Propriétés clés :**
- Définie sur $[0, 1]$ — parfaite comme prior sur une probabilité
- $\mathbb{E}[p] = \frac{\alpha}{\alpha + \beta}$
- Prior conjugué de la Bernoulli/Binomiale

```python
from scipy.stats import beta

# Différentes formes de la distribution Beta
configs = [
    (0.5, 0.5, 'U-shape (Jeffreys prior)'),
    (1, 1, 'Uniforme'),
    (2, 5, 'Asymétrique gauche'),
    (5, 2, 'Asymétrique droite'),
    (5, 5, 'Symétrique unimodale'),
]

x = np.linspace(0.01, 0.99, 200)
plt.figure(figsize=(10, 5))

for alpha, beta_param, label in configs:
    rv = beta(alpha, beta_param)
    plt.plot(x, rv.pdf(x), linewidth=2, label=f'α={alpha}, β={beta_param} — {label}')

plt.xlabel('p')
plt.ylabel('Densité')
plt.title('Distribution Beta pour différents paramètres')
plt.legend()
plt.grid(alpha=0.3)
plt.show()
```

#### Distribution Dirichlet

Généralisation multidimensionnelle de la Beta :

$$\text{Dir}(\mathbf{p} \mid \boldsymbol{\alpha}) = \frac{1}{B(\boldsymbol{\alpha})} \prod_{k=1}^K p_k^{\alpha_k - 1}$$

**Usage :** Prior sur des vecteurs de probabilité (topics LDA, transitions HMM).

```python
from scipy.stats import dirichlet

# Prior de Dirichlet symétrique (α = [1,1,1])
alpha = np.array([1.0, 1.0, 1.0])
rv = dirichlet(alpha)

# Échantillonner des distributions de probabilité
samples = rv.rvs(5)
print("Échantillons de la Dirichlet(1,1,1) — chaque ligne est un vecteur de proba :")
for s in samples:
    print(f"  {s.round(3)}  (somme = {s.sum():.3f})")

# Dirichlet concentrée (α grands → distributions proches de la moyenne)
alpha_concentrated = np.array([10.0, 10.0, 10.0])
samples_conc = dirichlet(alpha_concentrated).rvs(5)
print("\nÉchantillons de la Dirichlet(10,10,10) — plus concentrés :")
for s in samples_conc:
    print(f"  {s.round(3)}")
```

---

### 1.2 Estimation du Maximum de Vraisemblance (MLE)

**Principe :** Trouver les paramètres $\theta$ qui maximisent la vraisemblance des données observées.

$$\hat{\theta}_{MLE} = \arg\max_\theta \prod_{i=1}^N p(x_i \mid \theta) = \arg\max_\theta \sum_{i=1}^N \log p(x_i \mid \theta)$$

#### Dérivation analytique — Coin Flip

Données : $N$ lancers, $k$ faces (succès). Modèle : $\text{Bernoulli}(p)$.

**Log-vraisemblance :**
$$\ell(p) = k \log p + (N-k) \log(1-p)$$

**Dérivée et condition d'optimalité :**
$$\frac{d\ell}{dp} = \frac{k}{p} - \frac{N-k}{1-p} = 0$$

**Solution :**
$$\boxed{\hat{p}_{MLE} = \frac{k}{N}}$$

Le MLE pour la Bernoulli est simplement la fréquence empirique de succès.

```python
import numpy as np
from scipy.optimize import minimize_scalar

def neg_log_likelihood_bernoulli(p, k, N):
    """Log-vraisemblance négative (on minimise) pour la Bernoulli."""
    if p <= 0 or p >= 1:
        return np.inf
    return -(k * np.log(p) + (N - k) * np.log(1 - p))

# Données simulées
np.random.seed(42)
true_p = 0.7
N = 100
data = np.random.binomial(1, true_p, N)
k = data.sum()

print(f"Vrais paramètres : p = {true_p}")
print(f"Données : N={N}, k={k} succès")

# MLE analytique
p_mle_analytic = k / N
print(f"\nMLE analytique : p̂ = k/N = {p_mle_analytic:.4f}")

# MLE numérique (vérification)
result = minimize_scalar(neg_log_likelihood_bernoulli, bounds=(0.01, 0.99),
                         method='bounded', args=(k, N))
print(f"MLE numérique  : p̂ = {result.x:.4f}")

# Visualiser la vraisemblance
p_range = np.linspace(0.01, 0.99, 200)
ll = [-neg_log_likelihood_bernoulli(p, k, N) for p in p_range]

plt.figure(figsize=(8, 4))
plt.plot(p_range, ll, 'b-', linewidth=2)
plt.axvline(p_mle_analytic, color='red', linestyle='--', label=f'MLE = {p_mle_analytic:.3f}')
plt.axvline(true_p, color='green', linestyle=':', label=f'Vraie valeur = {true_p}')
plt.xlabel('p')
plt.ylabel('Log-vraisemblance')
plt.title('Log-vraisemblance de la Bernoulli')
plt.legend()
plt.grid(alpha=0.3)
plt.show()
```

#### MLE pour la Gaussienne

Données : $\{x_1, \ldots, x_N\}$. Paramètres : $(\mu, \sigma^2)$.

**Résultats analytiques :**
$$\hat{\mu}_{MLE} = \frac{1}{N}\sum_{i=1}^N x_i \qquad \hat{\sigma}^2_{MLE} = \frac{1}{N}\sum_{i=1}^N (x_i - \hat{\mu})^2$$

> **Important :** $\hat{\sigma}^2_{MLE}$ est **biaisé** — il sous-estime la vraie variance. L'estimateur non biaisé utilise $N-1$ au dénominateur (correction de Bessel).

```python
# Données gaussiennes simulées
true_mu, true_sigma2 = 3.0, 4.0
N = 50
data = np.random.normal(true_mu, np.sqrt(true_sigma2), N)

# Estimateurs MLE
mu_mle = data.mean()
sigma2_mle = data.var()          # divisé par N (biaisé)
sigma2_unbiased = data.var(ddof=1)  # divisé par N-1 (non biaisé)

print(f"Vraie moyenne  : {true_mu}")
print(f"MLE moyenne    : {mu_mle:.4f}")
print()
print(f"Vraie variance : {true_sigma2}")
print(f"MLE variance (biaisé, /N)     : {sigma2_mle:.4f}")
print(f"Estimateur non biaisé (/N-1)  : {sigma2_unbiased:.4f}")

# Démontrer le biais empiriquement
n_experiments = 10000
biased_estimates = []
unbiased_estimates = []

for _ in range(n_experiments):
    sample = np.random.normal(true_mu, np.sqrt(true_sigma2), N)
    biased_estimates.append(sample.var())
    unbiased_estimates.append(sample.var(ddof=1))

print(f"\nSur {n_experiments} expériences (N={N}) :")
print(f"  Moyenne des estimées biaisées : {np.mean(biased_estimates):.4f} (attendu : {true_sigma2})")
print(f"  Moyenne des estimées non biaisées : {np.mean(unbiased_estimates):.4f}")
```

---

### 1.3 Estimation MAP et Inférence Bayésienne

#### Théorème de Bayes

$$\underbrace{p(\theta \mid \mathcal{D})}_{\text{posterior}} = \frac{\underbrace{p(\mathcal{D} \mid \theta)}_{\text{vraisemblance}} \cdot \underbrace{p(\theta)}_{\text{prior}}}{\underbrace{p(\mathcal{D})}_{\text{évidence}}}$$

**Vocabulaire :**
- **Prior** $p(\theta)$ : croyances sur $\theta$ avant d'observer les données
- **Vraisemblance** $p(\mathcal{D} \mid \theta)$ : probabilité des données étant donné $\theta$
- **Posterior** $p(\theta \mid \mathcal{D})$ : croyances mises à jour après observation
- **Évidence** $p(\mathcal{D})$ : constante de normalisation (souvent intractable)

#### Estimation MAP

$$\hat{\theta}_{MAP} = \arg\max_\theta p(\theta \mid \mathcal{D}) = \arg\max_\theta \left[\log p(\mathcal{D} \mid \theta) + \log p(\theta)\right]$$

**Relation avec la régularisation :** MAP avec prior gaussien = régression ridge (L2). MAP avec prior Laplace = Lasso (L1).

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import beta, binom

# Inférence bayésienne sur p (coin flip)
# Prior : Beta(alpha_prior, beta_prior)
# Vraisemblance : Binomiale(N, p)
# Posterior : Beta(alpha_prior + k, beta_prior + N - k) [conjugaison]

alpha_prior = 2
beta_prior = 2
true_p = 0.7

p_range = np.linspace(0.01, 0.99, 300)

fig, axes = plt.subplots(1, 3, figsize=(15, 4))

for ax_idx, (N, k) in enumerate([(0, 0), (10, 7), (100, 70)]):
    # Prior
    prior_rv = beta(alpha_prior, beta_prior)
    prior = prior_rv.pdf(p_range)

    if N > 0:
        # Vraisemblance (non normalisée)
        likelihood = binom.pmf(k, N, p_range)
        likelihood = likelihood / likelihood.max()  # normaliser pour affichage

        # Posterior (conjugué)
        alpha_post = alpha_prior + k
        beta_post = beta_prior + (N - k)
        posterior = beta(alpha_post, beta_post).pdf(p_range)

        # MLE et MAP
        p_mle = k / N
        p_map = (alpha_prior + k - 1) / (alpha_prior + beta_prior + N - 2)
    else:
        likelihood = np.ones_like(p_range) * 0.5
        posterior = prior.copy()
        p_mle = p_map = None

    axes[ax_idx].plot(p_range, prior / prior.max(), 'b--', label='Prior', alpha=0.7)
    axes[ax_idx].plot(p_range, likelihood, 'g:', label='Vraisemblance', alpha=0.7)
    axes[ax_idx].plot(p_range, posterior / posterior.max(), 'r-', linewidth=2, label='Posterior')

    if p_mle is not None:
        axes[ax_idx].axvline(p_mle, color='g', linestyle='--', alpha=0.8, label=f'MLE={p_mle:.2f}')
        axes[ax_idx].axvline(p_map, color='r', linestyle='--', alpha=0.8, label=f'MAP={p_map:.2f}')

    axes[ax_idx].axvline(true_p, color='k', linestyle=':', label=f'Vraie p={true_p}')
    axes[ax_idx].set_title(f'N={N}, k={k}')
    axes[ax_idx].set_xlabel('p')
    axes[ax_idx].legend(fontsize=8)
    axes[ax_idx].grid(alpha=0.3)

plt.suptitle('Update Bayésien séquentiel — Prior Beta(2,2)', fontsize=13)
plt.tight_layout()
plt.show()
```

#### Update bayésien séquentiel

```python
# Démonstration : l'update séquentiel est équivalent à l'update global
alpha = 2.0
beta_param = 2.0

print("=== Update bayésien séquentiel ===")
print(f"Prior initial : Beta({alpha}, {beta_param})")
print(f"  → Moyenne prior : {alpha/(alpha+beta_param):.3f}")
print()

observations = [1, 1, 0, 1, 0, 1, 1, 1, 0, 1]  # 7 succès sur 10

for i, obs in enumerate(observations):
    # Update : posterior = Beta(alpha + k, beta + (N-k))
    alpha += obs
    beta_param += (1 - obs)
    p_map = (alpha - 1) / (alpha + beta_param - 2)
    p_mean = alpha / (alpha + beta_param)
    print(f"Après obs {i+1} (obs={obs}) : Beta({alpha:.0f}, {beta_param:.0f})"
          f" → Moyenne = {p_mean:.3f}, MAP = {p_map:.3f}")

print(f"\nMLE global : {sum(observations)/len(observations):.3f}")
```

#### Priors conjugués — tableau de référence

| Modèle (vraisemblance) | Prior conjugué | Posterior |
|---|---|---|
| Bernoulli / Binomiale | Beta(α, β) | Beta(α + k, β + N - k) |
| Multinomiale | Dirichlet(α) | Dirichlet(α + counts) |
| Gaussienne (μ inconnu, σ² connu) | Gaussienne(μ₀, σ₀²) | Gaussienne mise à jour |
| Gaussienne (μ et σ² inconnus) | Normal-Inverse-Gamma | Normal-Inverse-Gamma |
| Poisson | Gamma(α, β) | Gamma(α + Σxᵢ, β + N) |
| Exponentielle | Gamma(α, β) | Gamma(α + N, β + Σxᵢ) |

---

## 2. Naïf Bayes

### 2.1 Fondement théorique

Le classificateur Naïf Bayes applique le théorème de Bayes avec une **hypothèse d'indépendance conditionnelle** forte :

$$P(C \mid x_1, \ldots, x_n) \propto P(C) \prod_{i=1}^n P(x_i \mid C)$$

**Règle de décision :**
$$\hat{C} = \arg\max_C P(C) \prod_{i=1}^n P(x_i \mid C)$$

**Hypothèse naïve :** Les features $x_i$ sont conditionnellement indépendantes étant donné la classe $C$ :
$$P(x_i \mid x_j, C) = P(x_i \mid C) \quad \forall i \neq j$$

Malgré cette hypothèse souvent fausse en pratique, Naïf Bayes performe remarquablement bien sur de nombreuses tâches, notamment en classification de texte.

**Problème du underflow numérique :** Le produit de nombreuses petites probabilités $\to 0$. Solution : travailler en log-espace.

$$\log P(C \mid \mathbf{x}) \propto \log P(C) + \sum_{i=1}^n \log P(x_i \mid C)$$

### 2.2 Les trois variantes

#### Naïf Bayes Multinomial

**Usage :** Comptage de mots dans des documents texte.

$$P(x_i \mid C) = \frac{\text{count}(x_i, C) + \alpha}{\sum_j [\text{count}(x_j, C) + \alpha]}$$

Le terme $\alpha$ est le **lissage de Laplace** — évite les probabilités zéro pour les mots non vus dans l'entraînement (problème des mots hors-vocabulaire).

#### Naïf Bayes Gaussien

**Usage :** Features continues.

$$P(x_i \mid C) = \mathcal{N}(x_i \mid \mu_{iC}, \sigma^2_{iC})$$

Les paramètres $\mu_{iC}$ et $\sigma^2_{iC}$ sont estimés par MLE sur chaque classe.

#### Naïf Bayes de Bernoulli

**Usage :** Features binaires (présence/absence de mot).

$$P(x_i \mid C) = p_{iC}^{x_i} (1 - p_{iC})^{1-x_i}$$

### 2.3 Implémentation — Détection de spam

```python
import numpy as np
import pandas as pd
from sklearn.naive_bayes import MultinomialNB, GaussianNB, BernoulliNB
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report, confusion_matrix
from sklearn.pipeline import Pipeline

# Dataset simulé de emails spam/ham
emails = [
    # Ham (légitimes)
    "meeting tomorrow at 10am please confirm",
    "quarterly report attached for your review",
    "lunch with the team today at noon",
    "project deadline has been extended to Friday",
    "please review the attached document",
    "call me when you get a chance",
    "happy birthday have a great day",
    "the conference call is rescheduled",
    "your order has been shipped",
    "thank you for your feedback",
    # Spam
    "win million dollars click here now",
    "free offer limited time act now",
    "buy cheap pills online discount",
    "you won prize claim immediately",
    "make money fast work from home",
    "click here for free gift voucher",
    "urgent your account suspended verify",
    "cheap loans approved instantly apply",
    "earn thousands weekly from home",
    "free casino chips no deposit required",
]
labels = [0]*10 + [1]*10  # 0=ham, 1=spam

# Split entraînement/test
X_train, X_test, y_train, y_test = train_test_split(
    emails, labels, test_size=0.3, random_state=42, stratify=labels
)

print(f"Entraînement : {len(X_train)} emails")
print(f"Test : {len(X_test)} emails\n")

# Pipeline : vectorisation + Naïf Bayes Multinomial
pipeline = Pipeline([
    ('vectorizer', CountVectorizer(
        stop_words='english',
        min_df=1,
        ngram_range=(1, 2)  # unigrammes + bigrammes
    )),
    ('classifier', MultinomialNB(alpha=1.0))  # alpha = lissage de Laplace
])

pipeline.fit(X_train, y_train)
y_pred = pipeline.predict(X_test)

print("=== Rapport de classification ===")
print(classification_report(y_test, y_pred, target_names=['Ham', 'Spam']))

print("=== Matrice de confusion ===")
cm = confusion_matrix(y_test, y_pred)
print(f"           Prédit Ham  Prédit Spam")
print(f"Réel Ham    {cm[0,0]:8d}  {cm[0,1]:11d}")
print(f"Réel Spam   {cm[1,0]:8d}  {cm[1,1]:11d}")
```

#### Inspection des mots discriminants

```python
# Extraire les mots les plus discriminants pour chaque classe
vectorizer = pipeline.named_steps['vectorizer']
classifier = pipeline.named_steps['classifier']

feature_names = vectorizer.get_feature_names_out()
log_probs = classifier.feature_log_prob_  # shape: (n_classes, n_features)

# Pour la classe Spam (index 1), trouver les mots avec le plus haut log P(mot|Spam) - log P(mot|Ham)
log_ratio = log_probs[1] - log_probs[0]  # log odds ratio
top_spam_indices = np.argsort(log_ratio)[-10:][::-1]
top_ham_indices = np.argsort(log_ratio)[:10]

print("=== Top 10 mots indicateurs de SPAM ===")
for idx in top_spam_indices:
    print(f"  '{feature_names[idx]}' : log-ratio = {log_ratio[idx]:.3f}")

print("\n=== Top 10 mots indicateurs de HAM ===")
for idx in top_ham_indices:
    print(f"  '{feature_names[idx]}' : log-ratio = {log_ratio[idx]:.3f}")
```

#### Prédiction avec probabilités

```python
# Nouveaux emails à classifier
new_emails = [
    "click here to claim your free prize now",
    "please find the report attached for review",
    "limited time offer buy now and save",
]

probabilities = pipeline.predict_proba(new_emails)
predictions = pipeline.predict(new_emails)

print("=== Prédictions sur nouveaux emails ===")
for email, pred, proba in zip(new_emails, predictions, probabilities):
    label = "SPAM" if pred == 1 else "HAM"
    print(f"\n'{email[:50]}...'")
    print(f"  Prédiction : {label}")
    print(f"  P(Ham)  = {proba[0]:.4f}")
    print(f"  P(Spam) = {proba[1]:.4f}")
```

### 2.4 Comparaison des variantes

| Variante | Features | Use case | Avantage |
|---|---|---|---|
| **MultinomialNB** | Comptages entiers ≥ 0 | Classification texte (bag-of-words) | Simple, efficace pour TF |
| **GaussianNB** | Réels continus | Classification générale | Pas de discrétisation |
| **BernoulliNB** | Binaires {0, 1} | Présence/absence de mots | Pénalise l'absence explicitement |

**Naïf Bayes vs Régression Logistique :**

| Critère | Naïf Bayes | Régression Logistique |
|---|---|---|
| Type | Génératif | Discriminatif |
| Hypothèse | Indépendance conditionnelle | Aucune sur les features |
| Données limitées | Meilleur | Moins bon |
| Données abondantes | Moins bon | Meilleur |
| Vitesse d'entraînement | Très rapide | Plus lent |
| Robustesse | Bonne | Bonne |
| Features corrélées | Problématique | Gère bien |

---

## 3. Modèles de Markov Cachés (HMM)

### 3.1 Définition formelle

Un HMM est défini par un quintuplet $\lambda = (Q, O, A, B, \pi)$ :

| Symbole | Signification | Dimension |
|---|---|---|
| $Q = \{q_1, \ldots, q_N\}$ | Ensemble des états cachés | $N$ états |
| $O = \{o_1, \ldots, o_M\}$ | Alphabet des observations | $M$ symboles |
| $A = [a_{ij}]$ | Matrice de transition : $a_{ij} = P(q_t = j \mid q_{t-1} = i)$ | $N \times N$ |
| $B = [b_j(o_k)]$ | Matrice d'émission : $b_j(o_k) = P(o_k \mid q_t = j)$ | $N \times M$ |
| $\pi = [\pi_i]$ | Distribution initiale : $\pi_i = P(q_1 = i)$ | $N$ |

**Hypothèse de Markov :** L'état courant ne dépend que de l'état précédent :
$$P(q_t \mid q_{t-1}, q_{t-2}, \ldots, q_1) = P(q_t \mid q_{t-1})$$

**Hypothèse d'indépendance des observations :** L'observation courante ne dépend que de l'état courant :
$$P(o_t \mid q_t, q_{t-1}, \ldots, q_1, o_{t-1}, \ldots, o_1) = P(o_t \mid q_t)$$

#### Diagramme — HMM pour la reconnaissance de parole

```
                    ┌─────────────────────────────────────┐
                    │         ÉTATS CACHÉS (phonèmes)      │
                    └─────────────────────────────────────┘

     π=[0.5, 0.3, 0.2]
          │
          ▼
    ┌─────────┐   a₁₂=0.4    ┌─────────┐   a₂₃=0.5    ┌─────────┐
    │ /s/     │──────────────▶│ /æ/     │──────────────▶│ /t/     │
    │ silence │◀──────────────│ voyelle │◀──────────────│ occl.   │
    └─────────┘   a₂₁=0.1    └─────────┘   a₃₂=0.2    └─────────┘
        │  ↑                      │  ↑                      │  ↑
        │  │ a₁₁=0.6              │  │ a₂₂=0.5             │  │ a₃₃=0.3
        └──┘                      └──┘                      └──┘

    b₁(MFCC):              b₂(MFCC):              b₃(MFCC):
    𝒩(μ₁, Σ₁)             𝒩(μ₂, Σ₂)             𝒩(μ₃, Σ₃)
         │                      │                      │
         ▼                      ▼                      ▼
    ┌─────────────────────────────────────────────────────┐
    │           OBSERVATIONS (vecteurs MFCC)               │
    │    o₁        o₂        o₃        o₄        o₅       │
    └─────────────────────────────────────────────────────┘
```

### 3.2 Les trois problèmes fondamentaux

#### Problème 1 — Évaluation : Algorithme Forward

**Question :** Quelle est la probabilité de la séquence d'observations $\mathbf{O} = (o_1, \ldots, o_T)$ étant donné le modèle $\lambda$ ?

$$P(\mathbf{O} \mid \lambda) = \sum_{\mathbf{q}} P(\mathbf{O} \mid \mathbf{q}, \lambda) \cdot P(\mathbf{q} \mid \lambda)$$

L'énumération naïve a une complexité $O(N^T)$ — explosive. L'algorithme Forward utilise la programmation dynamique.

**Définition de la variable forward :**
$$\alpha_t(i) = P(o_1, o_2, \ldots, o_t, q_t = i \mid \lambda)$$

**Récurrence :**
$$\alpha_1(i) = \pi_i \cdot b_i(o_1)$$
$$\alpha_{t+1}(j) = \left[\sum_{i=1}^N \alpha_t(i) \cdot a_{ij}\right] \cdot b_j(o_{t+1})$$

**Résultat final :**
$$P(\mathbf{O} \mid \lambda) = \sum_{i=1}^N \alpha_T(i)$$

> **Problème de underflow numérique :** Pour de longues séquences, $\alpha_t(i) \to 0$ rapidement. Solution : **scaling** — normaliser $\alpha_t$ à chaque étape et accumuler les facteurs d'échelle en log.

```python
import numpy as np

def forward_algorithm(pi, A, B, observations):
    """
    Algorithme Forward avec scaling numérique.

    Args:
        pi: Distribution initiale, shape (N,)
        A: Matrice de transition, shape (N, N)
        B: Matrice d'émission, shape (N, M)
        observations: Séquence d'indices d'observations, shape (T,)

    Returns:
        log_prob: Log-probabilité de la séquence
        alpha_scaled: Variables forward normalisées, shape (T, N)
        scales: Facteurs de normalisation, shape (T,)
    """
    N = len(pi)
    T = len(observations)

    alpha = np.zeros((T, N))
    scales = np.zeros(T)

    # Initialisation
    alpha[0] = pi * B[:, observations[0]]
    scales[0] = alpha[0].sum()
    if scales[0] > 0:
        alpha[0] /= scales[0]

    # Récurrence
    for t in range(1, T):
        for j in range(N):
            alpha[t, j] = np.dot(alpha[t-1], A[:, j]) * B[j, observations[t]]

        scales[t] = alpha[t].sum()
        if scales[t] > 0:
            alpha[t] /= scales[t]

    # Log-probabilité totale
    log_prob = np.sum(np.log(scales[scales > 0]))

    return log_prob, alpha, scales


# Exemple : modèle météo caché (Rabiner 1989)
# États cachés : Soleil (0), Pluie (1)
# Observations : Sec (0), Humide (1), Très humide (2)

pi = np.array([0.6, 0.4])  # probabilités initiales

A = np.array([
    [0.7, 0.3],   # Soleil → Soleil=0.7, Pluie=0.3
    [0.4, 0.6],   # Pluie  → Soleil=0.4, Pluie=0.6
])

B = np.array([
    [0.5, 0.4, 0.1],  # P(obs|Soleil)
    [0.1, 0.3, 0.6],  # P(obs|Pluie)
])

# Séquence d'observations
observations = [0, 1, 2, 1, 0]  # Sec, Humide, Très humide, Humide, Sec

log_prob, alpha_scaled, scales = forward_algorithm(pi, A, B, observations)
print(f"Log P(observations | modèle) = {log_prob:.4f}")
print(f"P(observations | modèle) = {np.exp(log_prob):.6f}")
```

#### Problème 2 — Décodage : Algorithme de Viterbi

**Question :** Quelle est la séquence d'états cachés $\mathbf{q}^* = (q_1^*, \ldots, q_T^*)$ la plus probable étant donné les observations $\mathbf{O}$ ?

$$\mathbf{q}^* = \arg\max_{\mathbf{q}} P(\mathbf{q}, \mathbf{O} \mid \lambda)$$

**Pseudocode de Viterbi :**

```
INITIALISATION:
  δ₁(i) = πᵢ · bᵢ(o₁)    pour tout état i
  ψ₁(i) = 0               (pas de prédécesseur)

RÉCURRENCE (t = 2, ..., T):
  δₜ(j) = max_i [δₜ₋₁(i) · aᵢⱼ] · bⱼ(oₜ)
  ψₜ(j) = argmax_i [δₜ₋₁(i) · aᵢⱼ]

TERMINAISON:
  P* = max_i δₜ(i)
  q*ₜ = argmax_i δₜ(i)

BACKTRACKING:
  pour t = T-1, T-2, ..., 1:
    q*ₜ = ψₜ₊₁(q*ₜ₊₁)
```

**Implémentation en log-espace :**

```python
def viterbi_algorithm(pi, A, B, observations):
    """
    Algorithme de Viterbi en log-espace (évite le underflow).

    Args:
        pi: Distribution initiale, shape (N,)
        A: Matrice de transition, shape (N, N)
        B: Matrice d'émission, shape (N, M)
        observations: Séquence d'indices, shape (T,)

    Returns:
        best_path: Séquence d'états optimale, shape (T,)
        log_prob: Log-probabilité du chemin optimal
    """
    N = len(pi)
    T = len(observations)

    # Travailler en log-espace
    log_pi = np.log(np.where(pi > 0, pi, 1e-300))
    log_A = np.log(np.where(A > 0, A, 1e-300))
    log_B = np.log(np.where(B > 0, B, 1e-300))

    # Tables de programmation dynamique
    delta = np.full((T, N), -np.inf)  # meilleures probabilités
    psi = np.zeros((T, N), dtype=int)  # argmax des prédécesseurs (backpointers)

    # Initialisation
    delta[0] = log_pi + log_B[:, observations[0]]

    # Récurrence
    for t in range(1, T):
        for j in range(N):
            # delta[t, j] = max_i (delta[t-1, i] + log_A[i, j]) + log_B[j, o_t]
            scores = delta[t-1] + log_A[:, j]
            psi[t, j] = np.argmax(scores)
            delta[t, j] = scores[psi[t, j]] + log_B[j, observations[t]]

    # Terminaison
    log_prob = np.max(delta[T-1])
    best_last_state = np.argmax(delta[T-1])

    # Backtracking
    best_path = np.zeros(T, dtype=int)
    best_path[T-1] = best_last_state

    for t in range(T-2, -1, -1):
        best_path[t] = psi[t+1, best_path[t+1]]

    return best_path, log_prob


# Application sur le modèle météo
observations = [0, 1, 2, 1, 0]
best_path, log_prob = viterbi_algorithm(pi, A, B, observations)

state_names = ['Soleil', 'Pluie']
obs_names = ['Sec', 'Humide', 'Très humide']

print("=== Algorithme de Viterbi — Modèle météo ===")
print(f"Observations : {[obs_names[o] for o in observations]}")
print(f"États cachés : {[state_names[s] for s in best_path]}")
print(f"Log P(séquence optimale) = {log_prob:.4f}")
print(f"P(séquence optimale) = {np.exp(log_prob):.8f}")

# Vérification manuelle étape par étape
print("\n=== Trace de l'algorithme ===")
delta_log = np.full((len(observations), 2), -np.inf)
delta_log[0] = np.log(pi) + np.log(B[:, observations[0]])

for t_step in range(len(observations)):
    print(f"t={t_step} obs={obs_names[observations[t_step]]}: "
          f"δ({state_names[0]})={np.exp(delta_log[t_step, 0]):.6f}, "
          f"δ({state_names[1]})={np.exp(delta_log[t_step, 1]):.6f}")
    if t_step < len(observations) - 1:
        for j in range(2):
            scores = delta_log[t_step] + np.log(A[:, j])
            delta_log[t_step+1, j] = np.max(scores) + np.log(B[j, observations[t_step+1]])
```

#### Problème 3 — Apprentissage : Algorithme Baum-Welch

**Question :** Étant donné des séquences d'observations d'entraînement, comment estimer les paramètres $\lambda = (A, B, \pi)$ qui maximisent $P(\mathbf{O} \mid \lambda)$ ?

C'est un problème d'optimisation EM (Expectation-Maximization).

**Étape E — Calculer les probabilités d'état :**

Variable backward $\beta_t(i) = P(o_{t+1}, \ldots, o_T \mid q_t = i, \lambda)$ :
$$\beta_T(i) = 1$$
$$\beta_t(i) = \sum_{j=1}^N a_{ij} \cdot b_j(o_{t+1}) \cdot \beta_{t+1}(j)$$

Probabilité d'être dans l'état $i$ au temps $t$ :
$$\gamma_t(i) = \frac{\alpha_t(i) \cdot \beta_t(i)}{\sum_j \alpha_t(j) \cdot \beta_t(j)}$$

Probabilité de transiter de $i$ à $j$ entre $t$ et $t+1$ :
$$\xi_t(i, j) = \frac{\alpha_t(i) \cdot a_{ij} \cdot b_j(o_{t+1}) \cdot \beta_{t+1}(j)}{\sum_{i'}\sum_{j'} \alpha_t(i') \cdot a_{i'j'} \cdot b_{j'}(o_{t+1}) \cdot \beta_{t+1}(j')}$$

**Étape M — Mettre à jour les paramètres :**

$$\hat{\pi}_i = \gamma_1(i)$$

$$\hat{a}_{ij} = \frac{\sum_{t=1}^{T-1} \xi_t(i,j)}{\sum_{t=1}^{T-1} \gamma_t(i)}$$

$$\hat{b}_j(o_k) = \frac{\sum_{t: o_t = o_k} \gamma_t(j)}{\sum_{t=1}^T \gamma_t(j)}$$

> **Garantie :** Baum-Welch converge vers un **maximum local** (pas global). La solution dépend de l'initialisation. En pratique : multi-start avec différentes initialisations aléatoires, garder la meilleure.

```python
def backward_algorithm(A, B, observations, scales):
    """
    Algorithme Backward avec scaling (utilise les mêmes scales que Forward).
    """
    N = A.shape[0]
    T = len(observations)

    beta = np.zeros((T, N))

    # Initialisation
    beta[T-1] = 1.0 / scales[T-1] if scales[T-1] > 0 else 1.0

    # Récurrence en arrière
    for t in range(T-2, -1, -1):
        for i in range(N):
            beta[t, i] = np.dot(A[i], B[:, observations[t+1]] * beta[t+1])
        if scales[t] > 0:
            beta[t] /= scales[t]

    return beta


def baum_welch(observations_list, N, M, n_iter=100, tol=1e-6, random_seed=42):
    """
    Algorithme Baum-Welch pour l'entraînement d'un HMM.

    Args:
        observations_list: Liste de séquences d'observations
        N: Nombre d'états cachés
        M: Nombre de symboles d'observation
        n_iter: Nombre max d'itérations EM
        tol: Critère de convergence (changement de log-vraisemblance)

    Returns:
        pi, A, B: Paramètres optimisés
        log_likelihoods: Historique des log-vraisemblances
    """
    rng = np.random.default_rng(random_seed)

    # Initialisation aléatoire des paramètres
    pi = rng.dirichlet(np.ones(N))
    A = rng.dirichlet(np.ones(N), size=N)
    B = rng.dirichlet(np.ones(M), size=N)

    log_likelihoods = []
    prev_log_lik = -np.inf

    for iteration in range(n_iter):
        # Accumulateurs pour les statistiques suffisantes
        gamma_sum = np.zeros(N)
        gamma_obs_sum = np.zeros((N, M))
        xi_sum = np.zeros((N, N))
        pi_sum = np.zeros(N)
        total_log_lik = 0

        # Étape E : calculer les statistiques sur toutes les séquences
        for obs_seq in observations_list:
            T = len(obs_seq)

            # Forward avec scaling
            log_prob, alpha, scales = forward_algorithm(pi, A, B, obs_seq)
            total_log_lik += log_prob

            # Backward avec les mêmes scales
            beta = backward_algorithm(A, B, obs_seq, scales)

            # Calculer γₜ(i) — probabilité d'être dans l'état i au temps t
            gamma = alpha * beta  # shape (T, N)
            gamma_normalized = gamma / (gamma.sum(axis=1, keepdims=True) + 1e-300)

            # Calculer ξₜ(i,j) — probabilité de transition i→j au temps t
            for t in range(T - 1):
                xi_t = np.zeros((N, N))
                for i in range(N):
                    for j in range(N):
                        xi_t[i, j] = (alpha[t, i] * A[i, j] *
                                      B[j, obs_seq[t+1]] * beta[t+1, j])
                denom = xi_t.sum()
                if denom > 0:
                    xi_sum += xi_t / denom

            # Accumuler les statistiques
            pi_sum += gamma_normalized[0]
            gamma_sum += gamma_normalized.sum(axis=0)

            for t in range(T):
                gamma_obs_sum[:, obs_seq[t]] += gamma_normalized[t]

        # Étape M : mettre à jour les paramètres
        pi = pi_sum / pi_sum.sum()

        A = xi_sum / (xi_sum.sum(axis=1, keepdims=True) + 1e-300)
        A = A / (A.sum(axis=1, keepdims=True) + 1e-300)

        B = gamma_obs_sum / (gamma_sum.reshape(-1, 1) + 1e-300)
        B = B / (B.sum(axis=1, keepdims=True) + 1e-300)

        log_likelihoods.append(total_log_lik)

        # Vérifier la convergence
        if abs(total_log_lik - prev_log_lik) < tol:
            print(f"Convergé à l'itération {iteration+1}")
            break
        prev_log_lik = total_log_lik

    return pi, A, B, log_likelihoods


# Test sur des séquences synthétiques
np.random.seed(42)
# Générer des données avec le vrai modèle météo
true_A = np.array([[0.7, 0.3], [0.4, 0.6]])
true_B = np.array([[0.5, 0.4, 0.1], [0.1, 0.3, 0.6]])
true_pi = np.array([0.6, 0.4])

def generate_hmm_sequence(pi, A, B, T):
    """Générer une séquence synthétique depuis un HMM."""
    N = len(pi)
    states = np.zeros(T, dtype=int)
    obs = np.zeros(T, dtype=int)

    states[0] = np.random.choice(N, p=pi)
    obs[0] = np.random.choice(B.shape[1], p=B[states[0]])

    for t in range(1, T):
        states[t] = np.random.choice(N, p=A[states[t-1]])
        obs[t] = np.random.choice(B.shape[1], p=B[states[t]])

    return states, obs

# Générer 50 séquences d'entraînement de longueur 20
training_sequences = []
for _ in range(50):
    _, obs_seq = generate_hmm_sequence(true_pi, true_A, true_B, 20)
    training_sequences.append(obs_seq.tolist())

print("=== Entraînement Baum-Welch ===")
learned_pi, learned_A, learned_B, ll_history = baum_welch(
    training_sequences, N=2, M=3, n_iter=50
)

print(f"\nVraie matrice A:\n{true_A}")
print(f"A apprise:\n{learned_A.round(3)}")
print(f"\nVraie matrice B:\n{true_B}")
print(f"B apprise:\n{learned_B.round(3)}")

# Courbe de convergence
plt.figure(figsize=(8, 4))
plt.plot(ll_history, 'b-o', markersize=4)
plt.xlabel('Itération EM')
plt.ylabel('Log-vraisemblance totale')
plt.title('Convergence de Baum-Welch')
plt.grid(alpha=0.3)
plt.show()
```

**Gestion des minima locaux :**

```python
def baum_welch_multistart(observations_list, N, M, n_starts=10, n_iter=100):
    """
    Multi-start Baum-Welch pour éviter les minima locaux.
    Entraîne n_starts modèles avec différentes initialisations
    et retourne le meilleur.
    """
    best_log_lik = -np.inf
    best_params = None

    for start_idx in range(n_starts):
        pi, A, B, ll_history = baum_welch(
            observations_list, N, M, n_iter=n_iter, random_seed=start_idx
        )
        final_ll = ll_history[-1]

        if final_ll > best_log_lik:
            best_log_lik = final_ll
            best_params = (pi, A, B)
            best_start = start_idx

        print(f"Start {start_idx+1:2d} : final log-lik = {final_ll:.2f}")

    print(f"\nMeilleur départ : #{best_start+1} (log-lik = {best_log_lik:.2f})")
    return best_params

# Lancer avec 10 initialisations
print("=== Multi-start Baum-Welch ===")
best_pi, best_A, best_B = baum_welch_multistart(training_sequences, N=2, M=3)
```

### 3.3 Implémentation avec hmmlearn

```python
from hmmlearn import hmm
import numpy as np
import matplotlib.pyplot as plt
import warnings
warnings.filterwarnings('ignore')

# =========================================================
# HMM Gaussien — Données financières (régimes de marché)
# =========================================================

np.random.seed(42)

# Simuler des rendements financiers avec 2 régimes :
# - Régime 1 : marché calme (faible volatilité)
# - Régime 2 : marché stressé (forte volatilité)
n_samples = 500
true_states = np.zeros(n_samples, dtype=int)
returns = np.zeros(n_samples)

# Générer des données synthétiques
state = 0  # commencer en régime calme
for t in range(n_samples):
    true_states[t] = state
    if state == 0:  # Calme
        returns[t] = np.random.normal(0.001, 0.01)  # mu=0.1%, sigma=1%
        state = 1 if np.random.rand() < 0.05 else 0  # 5% de changer de régime
    else:  # Stressé
        returns[t] = np.random.normal(-0.005, 0.03)  # mu=-0.5%, sigma=3%
        state = 0 if np.random.rand() < 0.1 else 1   # 10% de revenir calme

# Entraîner un HMM Gaussien
model_gaussian = hmm.GaussianHMM(
    n_components=2,    # 2 états cachés
    covariance_type="full",
    n_iter=100,
    random_state=42
)
model_gaussian.fit(returns.reshape(-1, 1))

# Décoder les états cachés
predicted_states = model_gaussian.predict(returns.reshape(-1, 1))

# Analyser les régimes appris
print("=== HMM Gaussien — Régimes de marché ===")
print(f"Log-vraisemblance : {model_gaussian.score(returns.reshape(-1, 1)):.2f}")
print()

for state_idx in range(2):
    mean = model_gaussian.means_[state_idx][0]
    std = np.sqrt(model_gaussian.covars_[state_idx][0][0])
    freq = (predicted_states == state_idx).mean()
    print(f"Régime {state_idx} : μ = {mean:.4f}, σ = {std:.4f}, fréquence = {freq:.1%}")

print("\nMatrice de transition apprise :")
print(model_gaussian.transmat_.round(3))

# Visualisation
fig, axes = plt.subplots(3, 1, figsize=(14, 10))

axes[0].plot(returns, 'k-', alpha=0.5, linewidth=0.8)
axes[0].set_title('Rendements simulés')
axes[0].set_ylabel('Rendement')

axes[1].fill_between(range(n_samples), true_states, alpha=0.7, color='blue', label='Vrai régime')
axes[1].set_title('États cachés vrais')
axes[1].set_ylabel('Régime')
axes[1].legend()

axes[2].fill_between(range(n_samples), predicted_states, alpha=0.7, color='orange', label='Régime prédit')
axes[2].set_title('États prédits par HMM')
axes[2].set_ylabel('Régime')
axes[2].legend()

plt.tight_layout()
plt.show()
```

```python
# =========================================================
# HMM Multinomial — POS Tagging simplifié
# =========================================================

# Vocabulaire simplifié : 6 mots
words = {'the': 0, 'cat': 1, 'dog': 2, 'runs': 3, 'fast': 4, 'sleeps': 5}
pos_tags = {0: 'DET', 1: 'NOUN', 2: 'VERB', 3: 'ADV'}

# Phrases d'entraînement (séquences de mots → séquences de POS attendus)
sentences = [
    [0, 1, 3],       # the cat runs → DET NOUN VERB
    [0, 2, 3],       # the dog runs → DET NOUN VERB
    [0, 1, 5, 4],    # the cat sleeps fast → DET NOUN VERB ADV
    [0, 2, 5],       # the dog sleeps → DET NOUN VERB
    [1, 3, 4],       # cat runs fast → NOUN VERB ADV
    [2, 5, 4],       # dog sleeps fast → NOUN VERB ADV
]

# Formater pour hmmlearn : séquences concaténées avec longueurs
X = np.concatenate([s for s in sentences]).reshape(-1, 1)
lengths = [len(s) for s in sentences]

model_pos = hmm.CategoricalHMM(
    n_components=4,   # 4 états (DET, NOUN, VERB, ADV — à découvrir)
    n_iter=200,
    random_state=42
)
model_pos.fit(X, lengths)

# Décoder une nouvelle phrase
test_sentence = np.array([0, 1, 3, 4]).reshape(-1, 1)  # "the cat runs fast"
predicted_pos = model_pos.predict(test_sentence)

print("=== HMM Multinomial — POS Tagging ===")
test_words = ['the', 'cat', 'runs', 'fast']
print("Phrase : " + " ".join(test_words))
print(f"États prédits : {predicted_pos}")
print(f"(Les états {0}–{3} correspondent aux POS selon l'entraînement)")

print("\nMatrice de transition apprise :")
print(model_pos.transmat_.round(3))

print("\nMatrice d'émission apprise (P(mot|état)) :")
for state_idx in range(4):
    top_words = np.argsort(model_pos.emissionprob_[state_idx])[-3:][::-1]
    word_names = [list(words.keys())[w] for w in top_words]
    probs = model_pos.emissionprob_[state_idx][top_words]
    print(f"  État {state_idx}: {', '.join([f'{w}({p:.2f})' for w, p in zip(word_names, probs)])}")
```

---

## 4. Applications

### 4.1 Classification de gestes (accéléromètre)

**Principe :** Un HMM distinct est entraîné pour chaque classe de geste. La classification d'un nouveau geste utilise le score de vraisemblance de chaque modèle — on assigne la classe dont le modèle donne la vraisemblance maximale.

```python
from hmmlearn import hmm
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report

# Simuler des données d'accéléromètre pour 3 gestes
# Chaque geste est une séquence multivariée (x, y, z)
np.random.seed(42)

def simulate_gesture(gesture_id, n_samples=50):
    """
    Simuler une séquence accéléromètre pour un geste.
    gesture_id:
      0 = agiter horizontal (oscillation rapide sur x)
      1 = secouer vertical (oscillation sur z)
      2 = cercle (rotation x-y)
    """
    t = np.linspace(0, 2*np.pi, n_samples)

    if gesture_id == 0:  # Agiter horizontal
        x = 2 * np.sin(4 * t) + np.random.randn(n_samples) * 0.3
        y = np.random.randn(n_samples) * 0.2
        z = np.random.randn(n_samples) * 0.2
    elif gesture_id == 1:  # Secouer vertical
        x = np.random.randn(n_samples) * 0.2
        y = np.random.randn(n_samples) * 0.2
        z = 2 * np.sin(3 * t) + np.random.randn(n_samples) * 0.3
    else:  # Cercle
        x = np.cos(t) + np.random.randn(n_samples) * 0.2
        y = np.sin(t) + np.random.randn(n_samples) * 0.2
        z = np.random.randn(n_samples) * 0.2

    return np.column_stack([x, y, z])

# Générer le dataset
n_per_class = 30
all_sequences = []
all_labels = []

for gesture_id in range(3):
    for _ in range(n_per_class):
        seq = simulate_gesture(gesture_id)
        all_sequences.append(seq)
        all_labels.append(gesture_id)

# Split train/test
indices = list(range(len(all_sequences)))
train_idx, test_idx = train_test_split(indices, test_size=0.3,
                                        stratify=all_labels, random_state=42)

# Entraîner un HMM par classe
gesture_names = ['Agiter horizontal', 'Secouer vertical', 'Cercle']
models = {}

print("=== Entraînement — Un HMM par classe ===")
for gesture_id in range(3):
    # Séquences d'entraînement pour cette classe
    class_sequences = [all_sequences[i] for i in train_idx
                       if all_labels[i] == gesture_id]

    X_train = np.concatenate(class_sequences)
    lengths = [len(s) for s in class_sequences]

    model = hmm.GaussianHMM(
        n_components=4,
        covariance_type='diag',
        n_iter=50,
        random_state=42
    )
    model.fit(X_train, lengths)
    models[gesture_id] = model

    print(f"  {gesture_names[gesture_id]} : "
          f"entraîné sur {len(class_sequences)} séquences")

# Classifier les séquences de test
print("\n=== Classification ===")
y_true = [all_labels[i] for i in test_idx]
y_pred = []

for i in test_idx:
    seq = all_sequences[i]
    scores = {}
    for gesture_id, model in models.items():
        try:
            score = model.score(seq)  # log P(seq | modèle)
        except Exception:
            score = -np.inf
        scores[gesture_id] = score

    # Choisir la classe avec le score maximum
    predicted_class = max(scores, key=scores.get)
    y_pred.append(predicted_class)

print(classification_report(y_true, y_pred, target_names=gesture_names))
```

### 4.2 Détection d'anomalies dans des logs système

```python
import numpy as np
from hmmlearn import hmm
from sklearn.preprocessing import LabelEncoder

# Simuler des séquences de logs système
# Événements normaux vs anomalies (intrusion, crash)

# Dictionnaire d'événements log
log_events = {
    'login_success': 0,
    'file_read': 1,
    'file_write': 2,
    'logout': 3,
    'login_fail': 4,
    'privilege_escalation': 5,  # rare → indicateur d'anomalie
    'mass_file_delete': 6,       # très rare → anomalie sévère
    'network_scan': 7,           # anomalie
}

def generate_normal_session(length=20):
    """Session normale : login → activités → logout."""
    events = [0]  # login_success
    for _ in range(length - 2):
        p = [0, 0.4, 0.3, 0.1, 0.1, 0.05, 0.0, 0.05]
        events.append(np.random.choice(8, p=p))
    events.append(3)  # logout
    return events

def generate_anomalous_session(length=20):
    """Session anormale : activités suspectes."""
    events = [4, 4, 4, 0]  # échecs login puis succès (brute force)
    for _ in range(length - 6):
        p = [0, 0.1, 0.1, 0, 0.1, 0.3, 0.2, 0.2]  # beaucoup d'activités suspectes
        events.append(np.random.choice(8, p=p))
    events.append(6)  # mass_file_delete
    events.append(3)  # logout
    return events

# Générer les données
normal_sessions = [generate_normal_session() for _ in range(200)]
anomalous_sessions = [generate_anomalous_session() for _ in range(20)]

# Entraîner le HMM uniquement sur des sessions normales
X_normal = np.concatenate(normal_sessions).reshape(-1, 1)
lengths_normal = [len(s) for s in normal_sessions]

model_logs = hmm.CategoricalHMM(
    n_components=5,   # 5 états cachés (phases d'une session normale)
    n_iter=100,
    random_state=42
)
model_logs.fit(X_normal, lengths_normal)

# Calculer les scores de vraisemblance sur les sessions normales (pour le seuil)
normal_scores = [model_logs.score(np.array(s).reshape(-1, 1))
                 for s in normal_sessions]

# Seuil d'anomalie : percentile 5 des sessions normales
threshold = np.percentile(normal_scores, 5)
print(f"Seuil d'anomalie (percentile 5) : {threshold:.2f}")
print(f"Score moyen sessions normales : {np.mean(normal_scores):.2f}")

# Évaluer sur un ensemble mélangé (normal + anomalies)
test_normal = [generate_normal_session() for _ in range(50)]
test_anomalous = [generate_anomalous_session() for _ in range(50)]
test_sessions = test_normal + test_anomalous
true_labels = [0]*50 + [1]*50  # 0=normal, 1=anomalie

predictions = []
for session in test_sessions:
    score = model_logs.score(np.array(session).reshape(-1, 1))
    is_anomaly = 1 if score < threshold else 0
    predictions.append(is_anomaly)

# Métriques
tp = sum(1 for t, p in zip(true_labels, predictions) if t == 1 and p == 1)
fp = sum(1 for t, p in zip(true_labels, predictions) if t == 0 and p == 1)
tn = sum(1 for t, p in zip(true_labels, predictions) if t == 0 and p == 0)
fn = sum(1 for t, p in zip(true_labels, predictions) if t == 1 and p == 0)

precision = tp / (tp + fp) if (tp + fp) > 0 else 0
recall = tp / (tp + fn) if (tp + fn) > 0 else 0
f1 = 2 * precision * recall / (precision + recall) if (precision + recall) > 0 else 0

print(f"\n=== Résultats détection d'anomalies ===")
print(f"Précision  : {precision:.3f}")
print(f"Rappel     : {recall:.3f}")
print(f"F1-score   : {f1:.3f}")
print(f"TP={tp}, FP={fp}, TN={tn}, FN={fn}")
```

### 4.3 Tableau comparatif — HMM vs CRF vs LSTM

| Critère | HMM | CRF | LSTM |
|---|---|---|---|
| **Type** | Modèle génératif | Modèle discriminatif | Deep learning |
| **Hypothèse** | Markov + indép. observations | Aucune sur les features | Aucune |
| **Features** | Observations brutes | Features arbitraires | Représentations apprises |
| **Dépendances temporelles** | Ordre 1 (Markov) | Fenêtre configurable | Long terme (théorique) |
| **Interprétabilité** | Élevée (paramètres explicites) | Moyenne | Faible |
| **Données nécessaires** | Peu | Moyen | Beaucoup |
| **Vitesse entraînement** | Rapide | Moyen | Lent |
| **Use cases** | Parole, gestes, bioinformatique | NER, POS tagging | Traduction, génération |
| **Bibliothèques** | hmmlearn, pomegranate | sklearn-crfsuite, pytorch-crf | PyTorch, TF/Keras |

**Quand choisir HMM :**
- Données étiquetées limitées
- Besoin d'interprétabilité (visualiser les états cachés)
- Modélisation des dépendances temporelles d'ordre 1 suffit
- Génération de séquences (synthèse)
- Détection d'anomalies non supervisée

---

## 5. Exercices

### Exercice 1 — Viterbi from Scratch (version dictionnaire)

**Objectif :** Implémenter l'algorithme de Viterbi avec des structures de données lisibles (sans numpy vectorisé) pour comprendre la logique de backtracking.

```python
def viterbi_dict(obs_sequence, states, start_prob, trans_prob, emit_prob):
    """
    Viterbi avec dictionnaires — implémentation pédagogique.

    Args:
        obs_sequence: Liste d'observations (chaînes de caractères)
        states: Liste des états possibles
        start_prob: Dict {état: proba initiale}
        trans_prob: Dict {état: {état_suivant: proba}}
        emit_prob: Dict {état: {observation: proba}}

    Returns:
        best_path: Liste des états optimaux
        max_prob: Probabilité du chemin optimal
    """
    # Initialisation
    viterbi = {}  # viterbi[état] = meilleure probabilité jusqu'ici
    backtrack = {}  # backtrack[état] = état prédécesseur optimal

    # t = 0
    obs0 = obs_sequence[0]
    for state in states:
        viterbi[state] = start_prob[state] * emit_prob[state].get(obs0, 0)
        backtrack[state] = None

    path = {state: [state] for state in states}

    # t = 1, 2, ..., T-1
    for obs in obs_sequence[1:]:
        new_viterbi = {}
        new_backtrack = {}
        new_path = {}

        for curr_state in states:
            # Trouver le meilleur état précédent
            max_prob = -1
            best_prev = None

            for prev_state in states:
                prob = (viterbi[prev_state]
                        * trans_prob[prev_state].get(curr_state, 0)
                        * emit_prob[curr_state].get(obs, 0))
                if prob > max_prob:
                    max_prob = prob
                    best_prev = prev_state

            new_viterbi[curr_state] = max_prob
            new_backtrack[curr_state] = best_prev
            new_path[curr_state] = path[best_prev] + [curr_state] if best_prev else [curr_state]

        viterbi = new_viterbi
        backtrack = new_backtrack
        path = new_path

    # Trouver l'état final optimal
    best_final = max(viterbi, key=viterbi.get)
    return path[best_final], viterbi[best_final]


# Modèle météo classique (Rabiner 1989)
states_weather = ['Ensoleillé', 'Nuageux', 'Pluvieux']

start_probability = {
    'Ensoleillé': 0.5,
    'Nuageux': 0.3,
    'Pluvieux': 0.2,
}

transition_probability = {
    'Ensoleillé': {'Ensoleillé': 0.6, 'Nuageux': 0.3, 'Pluvieux': 0.1},
    'Nuageux':    {'Ensoleillé': 0.3, 'Nuageux': 0.4, 'Pluvieux': 0.3},
    'Pluvieux':   {'Ensoleillé': 0.1, 'Nuageux': 0.3, 'Pluvieux': 0.6},
}

emission_probability = {
    'Ensoleillé': {'Sec': 0.7, 'Humide': 0.2, 'Très humide': 0.1},
    'Nuageux':    {'Sec': 0.3, 'Humide': 0.5, 'Très humide': 0.2},
    'Pluvieux':   {'Sec': 0.1, 'Humide': 0.3, 'Très humide': 0.6},
}

# Tester avec une séquence d'observations
observations_dict = ['Sec', 'Humide', 'Très humide', 'Humide', 'Sec']

best_path, max_prob = viterbi_dict(
    observations_dict,
    states_weather,
    start_probability,
    transition_probability,
    emission_probability
)

print("=== Viterbi from Scratch ===")
print(f"Observations : {observations_dict}")
print(f"Séquence d'états optimale : {best_path}")
print(f"Probabilité du chemin : {max_prob:.8f}")

# Vérification manuelle pour t=0
print("\n=== Initialisation (t=0) ===")
obs0 = observations_dict[0]
for state in states_weather:
    p = start_probability[state] * emission_probability[state][obs0]
    print(f"  δ₀({state}) = P({state}) × P({obs0}|{state}) = "
          f"{start_probability[state]:.1f} × {emission_probability[state][obs0]:.1f} = {p:.3f}")
```

### Exercice 2 — Entraîner un HMM et évaluer les paramètres

**Objectif :** Générer des données synthétiques depuis un HMM connu, entraîner un nouveau HMM, et vérifier que les paramètres appris sont proches des vrais paramètres.

```python
from hmmlearn import hmm
import numpy as np
from scipy.optimize import linear_sum_assignment

# Vrais paramètres
true_pi = np.array([0.6, 0.4])
true_A = np.array([[0.7, 0.3], [0.4, 0.6]])
true_B = np.array([[0.5, 0.4, 0.1], [0.1, 0.3, 0.6]])

# Générer des données d'entraînement
def generate_multinomial_hmm_data(pi, A, B, n_seq, seq_len):
    sequences = []
    for _ in range(n_seq):
        states_seq = np.zeros(seq_len, dtype=int)
        obs_seq = np.zeros(seq_len, dtype=int)

        states_seq[0] = np.random.choice(len(pi), p=pi)
        obs_seq[0] = np.random.choice(B.shape[1], p=B[states_seq[0]])

        for t in range(1, seq_len):
            states_seq[t] = np.random.choice(len(pi), p=A[states_seq[t-1]])
            obs_seq[t] = np.random.choice(B.shape[1], p=B[states_seq[t]])

        sequences.append(obs_seq)
    return sequences

np.random.seed(42)
train_seqs = generate_multinomial_hmm_data(true_pi, true_A, true_B, 200, 30)

X = np.concatenate(train_seqs).reshape(-1, 1)
lengths = [30] * 200

# Entraîner le HMM
model = hmm.CategoricalHMM(n_components=2, n_iter=100, random_state=42)
model.fit(X, lengths)

# Résoudre le problème de permutation des états :
# L'algorithme EM peut avoir inversé les états 0 et 1.
# Utiliser la méthode de l'assignation optimale (Hungarian algorithm).

def find_best_permutation(true_B, learned_B):
    """
    Trouver la permutation des états qui minimise la distance entre
    les matrices d'émission vraie et apprise.
    """
    N = true_B.shape[0]
    cost_matrix = np.zeros((N, N))

    for i in range(N):
        for j in range(N):
            cost_matrix[i, j] = np.sum((true_B[i] - learned_B[j]) ** 2)

    row_ind, col_ind = linear_sum_assignment(cost_matrix)
    return col_ind  # permutation : état col_ind[i] apprend correspond à état i vrai


perm = find_best_permutation(true_B, model.emissionprob_)
print(f"Permutation des états détectée : {perm}")

# Réordonner les paramètres appris
learned_pi_reordered = model.startprob_[perm]
learned_A_reordered = model.transmat_[perm][:, perm]
learned_B_reordered = model.emissionprob_[perm]

print("\n=== Comparaison des paramètres ===")
print(f"\nDistribution initiale π :")
print(f"  Vraie    : {true_pi}")
print(f"  Apprise  : {learned_pi_reordered.round(3)}")

print(f"\nMatrice de transition A :")
print(f"  Vraie :\n{true_A}")
print(f"  Apprise :\n{learned_A_reordered.round(3)}")

print(f"\nMatrice d'émission B :")
print(f"  Vraie :\n{true_B}")
print(f"  Apprise :\n{learned_B_reordered.round(3)}")

# Calculer les erreurs
print(f"\n=== Erreurs quadratiques moyennes ===")
print(f"π : {np.mean((true_pi - learned_pi_reordered)**2):.6f}")
print(f"A : {np.mean((true_A - learned_A_reordered)**2):.6f}")
print(f"B : {np.mean((true_B - learned_B_reordered)**2):.6f}")
```

### Exercice 3 — Challenge : Météo cachée complète

**Objectif :** Construire un système complet de prédiction météo utilisant un HMM — inférence des états passés, décodage de la séquence optimale, évaluation de vraisemblance et comparaison de modèles.

```python
"""
Challenge : Météo cachée

Données : séquences de mesures de capteurs (température, humidité, pression)
Objectif : inférer les états météo (Beau Temps, Nuageux, Orageux)
           sans jamais observer directement la météo.

Étapes :
1. Générer des données depuis un HMM "vrai"
2. Entraîner un HMM non supervisé sur ces données
3. Évaluer la qualité de l'inférence des états
4. Comparer plusieurs architectures (2, 3, 4 états)
"""

import numpy as np
from hmmlearn import hmm
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score
import matplotlib.pyplot as plt

np.random.seed(42)

# ===== Configuration du vrai modèle =====
N_TRUE_STATES = 3
FEATURES = 3  # température, humidité, pression

# Vraies distributions par état météo
true_means = np.array([
    [25.0,  0.3, 1020.0],  # Beau Temps
    [18.0,  0.6, 1005.0],  # Nuageux
    [15.0,  0.85, 990.0],  # Orageux
])

true_covars = np.array([
    [[4.0, 0, 0], [0, 0.01, 0], [0, 0, 25.0]],   # Beau Temps (faible variance)
    [[9.0, 0, 0], [0, 0.025, 0], [0, 0, 100.0]],  # Nuageux
    [[16.0, 0, 0], [0, 0.04, 0], [0, 0, 225.0]],  # Orageux (forte variance)
])

true_transmat = np.array([
    [0.7, 0.2, 0.1],   # Beau Temps → reste souvent beau
    [0.3, 0.5, 0.2],   # Nuageux → peut aller dans les deux sens
    [0.2, 0.3, 0.5],   # Orageux → peut durer
])

true_startprob = np.array([0.5, 0.3, 0.2])

# Créer le vrai modèle
true_model = hmm.GaussianHMM(n_components=N_TRUE_STATES, covariance_type='full')
true_model.startprob_ = true_startprob
true_model.transmat_ = true_transmat
true_model.means_ = true_means
true_model.covars_ = true_covars

# Générer les données
n_sequences = 100
seq_length = 30

all_observations = []
all_true_states = []
all_lengths = []

for _ in range(n_sequences):
    X_seq, states_seq = true_model.sample(seq_length)
    all_observations.append(X_seq)
    all_true_states.append(states_seq)
    all_lengths.append(seq_length)

X_train = np.concatenate(all_observations)
y_true = np.concatenate(all_true_states)

print(f"Données générées : {n_sequences} séquences × {seq_length} pas de temps")
print(f"Features : {FEATURES} (température, humidité, pression)")
print(f"Distribution des états vrais : {np.bincount(y_true) / len(y_true)}")

# ===== Entraîner et comparer plusieurs architectures =====
print("\n=== Comparaison d'architectures HMM ===")
results = {}

for n_components in [2, 3, 4, 5]:
    model = hmm.GaussianHMM(
        n_components=n_components,
        covariance_type='full',
        n_iter=100,
        random_state=42
    )
    model.fit(X_train, all_lengths)

    # Score sur les données d'entraînement
    train_score = model.score(X_train, all_lengths)

    # Score sur données de test (nouvelles séquences)
    test_observations = []
    test_lengths = []
    for _ in range(20):
        X_test_seq, _ = true_model.sample(seq_length)
        test_observations.append(X_test_seq)
        test_lengths.append(seq_length)
    X_test = np.concatenate(test_observations)

    test_score = model.score(X_test, test_lengths)

    # BIC (Bayesian Information Criterion) — pénalise la complexité
    n_params = (n_components - 1 +         # π
                n_components * (n_components - 1) +  # A
                n_components * FEATURES +   # moyennes
                n_components * FEATURES * FEATURES)  # covariances
    n_obs = len(X_train)
    bic = -2 * train_score + n_params * np.log(n_obs)

    # ARI si le nombre d'états correspond (sinon on ne peut pas comparer directement)
    if n_components == N_TRUE_STATES:
        predicted_states = model.predict(X_train, all_lengths)
        ari = adjusted_rand_score(y_true, predicted_states)
        nmi = normalized_mutual_info_score(y_true, predicted_states)
    else:
        ari = nmi = None

    results[n_components] = {
        'train_score': train_score,
        'test_score': test_score,
        'bic': bic,
        'ari': ari,
        'nmi': nmi,
        'model': model
    }

    ari_str = f"{ari:.3f}" if ari is not None else "N/A"
    print(f"  n={n_components} : BIC={bic:.1f}, "
          f"test_LL={test_score:.1f}, ARI={ari_str}")

# ===== Analyse du meilleur modèle =====
best_n = min(results, key=lambda k: results[k]['bic'])
print(f"\nMeilleur modèle selon BIC : n={best_n} états")

best_model = results[N_TRUE_STATES]['model']
predicted_states = best_model.predict(X_train, all_lengths)

print(f"\nARI (n=3) : {results[N_TRUE_STATES]['ari']:.3f}")
print("(ARI=1.0 = parfait, ARI=0.0 = aléatoire)")
print(f"NMI (n=3) : {results[N_TRUE_STATES]['nmi']:.3f}")

# Visualisation du résultat
fig, axes = plt.subplots(2, 1, figsize=(15, 6))

sample_seq = all_observations[0]
sample_true_states = all_true_states[0]
sample_predicted = best_model.predict(sample_seq)

# Permutation pour aligner les états prédits avec les vrais
from scipy.optimize import linear_sum_assignment
cost = np.zeros((3, 3))
for i in range(3):
    for j in range(3):
        cost[i, j] = -np.sum((sample_true_states == i) & (sample_predicted == j))
_, col_ind = linear_sum_assignment(cost)
perm_map = {col_ind[i]: i for i in range(3)}
sample_predicted_reordered = np.array([perm_map.get(s, s) for s in sample_predicted])

state_names_weather = ['Beau Temps', 'Nuageux', 'Orageux']
colors = ['gold', 'gray', 'navy']

for t in range(len(sample_seq)):
    axes[0].axvspan(t, t+1, alpha=0.3,
                    color=colors[sample_true_states[t]])
axes[0].plot(sample_seq[:, 0], 'k-', linewidth=1, label='Température')
axes[0].set_title('Vrais états météo (fond coloré) + Température observée')
axes[0].legend()

for t in range(len(sample_seq)):
    axes[1].axvspan(t, t+1, alpha=0.3,
                    color=colors[sample_predicted_reordered[t]])
axes[1].plot(sample_seq[:, 0], 'k-', linewidth=1, label='Température')
axes[1].set_title('États prédits par HMM (fond coloré) + Température observée')
axes[1].legend()

plt.tight_layout()
plt.show()

print("\n=== Paramètres appris par le HMM (après permutation) ===")
perm_arr = np.array([col_ind])
print("Moyennes apprises (température, humidité, pression) :")
for i, name in enumerate(state_names_weather):
    learned_mean = best_model.means_[col_ind[i]]
    true_mean = true_means[i]
    print(f"  {name} : appris={learned_mean.round(1)}, vrai={true_mean}")
```

---

## Ressources complémentaires

**Livres de référence :**
- Rabiner, L.R. (1989). *A tutorial on hidden Markov models and selected applications in speech recognition.* Proceedings of the IEEE, 77(2), 257–286. — L'article fondateur sur les HMM.
- Bishop, C.M. (2006). *Pattern Recognition and Machine Learning.* Springer. — Chapitres 8–9 pour les modèles graphiques et EM.
- Murphy, K.P. (2012). *Machine Learning: A Probabilistic Perspective.* MIT Press. — Référence complète sur l'inférence bayésienne.

**Bibliothèques Python :**
- [`hmmlearn`](https://hmmlearn.readthedocs.io/) — HMM (GaussianHMM, CategoricalHMM, GMMHMM)
- [`pomegranate`](https://pomegranate.readthedocs.io/) — HMM GPU + modèles probabilistes avancés
- [`scikit-learn`](https://scikit-learn.org/) — GaussianNB, MultinomialNB, BernoulliNB
- [`scipy.stats`](https://docs.scipy.org/doc/scipy/reference/stats.html) — Toutes les distributions (Beta, Dirichlet, Multinomiale, etc.)

**Pour aller plus loin :**
- **Modèles de Markov Cachés Semi-Markoviens (HSMM)** : durée dans chaque état explicitement modélisée
- **Modèles de Markov d'ordre supérieur** : dépendance sur les $k$ états précédents, pas juste 1
- **Kalman Filter** : l'équivalent HMM avec états et observations continus (espace d'état linéaire gaussien)
- **Particle Filters** : inférence exacte intractable → approximation par Monte Carlo séquentiel
- **Inférence variationnelle (VI)** : alternative à EM quand le posterior exact est intractable
