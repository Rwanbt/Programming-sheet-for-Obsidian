# Introduction à l'IA et Machine Learning — Fondamentaux

L'Intelligence Artificielle n'est plus un concept de science-fiction réservé aux chercheurs : elle est aujourd'hui intégrée dans les outils que vous utilisez au quotidien, des moteurs de recherche aux assistants de code. Ce cours vous donne les bases conceptuelles et pratiques pour comprendre ce que sont l'IA et le Machine Learning, comment les utiliser intelligemment en tant que développeur, et quelles responsabilités cela implique.

En tant qu'étudiant Holberton, vous allez croiser ces technologies dès vos premiers projets. L'objectif n'est pas de devenir data scientist, mais de comprendre ce qui se passe sous le capot pour ne pas être un utilisateur aveugle.

---

## 1. Qu'est-ce que l'Intelligence Artificielle ?

### 1.1 Définitions

Le terme "Intelligence Artificielle" est utilisé à toutes les sauces depuis les années 2020, ce qui rend une définition précise d'autant plus nécessaire.

> [!info] Définition formelle
> L'Intelligence Artificielle (IA) désigne l'ensemble des techniques permettant à une machine d'imiter des comportements cognitifs humains : raisonner, apprendre, résoudre des problèmes, comprendre le langage, percevoir l'environnement.

La définition la plus citée dans la littérature académique est celle de **John McCarthy** (1956), qui a inventé le terme : *"The science and engineering of making intelligent machines."* Mais cette définition est volontairement vague, car "intelligence" elle-même est un concept flou.

En pratique, on distingue deux grandes approches :

| Approche | Description | Exemple |
|----------|-------------|---------|
| **IA symbolique (GOFAI)** | Règles logiques explicites codées par des humains | Moteurs d'échecs classiques, systèmes experts médicaux |
| **IA connexionniste** | Apprentissage à partir de données, sans règles explicites | Réseaux de neurones, LLMs, reconnaissance d'image |

### 1.2 Bref historique

```
1950 ─── Alan Turing publie "Computing Machinery and Intelligence"
         → propose le "Turing Test"

1956 ─── Conférence de Dartmouth : naissance officielle du terme "IA"
         McCarthy, Minsky, Shannon, Simon

1966-1974 ─── 1er "hiver de l'IA" : promesses non tenues, financements coupés

1980s ─── Boom des systèmes experts (règles if/then)
         → limités : fragiles, coûteux à maintenir

1987-1993 ─── 2ème hiver de l'IA

1997 ─── Deep Blue (IBM) bat Kasparov aux échecs

2006 ─── Geoffrey Hinton relance le Deep Learning
         → backpropagation + GPUs = revolution

2012 ─── AlexNet gagne ImageNet : le Deep Learning explose
         → erreur réduite de 26% à 15% en un an

2017 ─── "Attention is All You Need" : invention des Transformers (Google)

2020 ─── GPT-3 (OpenAI) : 175 milliards de paramètres

2022 ─── ChatGPT : 100 millions d'utilisateurs en 2 mois
         → démocratisation massive

2023-2024 ─── Intégration dans tous les IDE, outils dev, moteurs de recherche
```

### 1.3 Les catégories d'IA

On distingue trois niveaux selon les capacités de généralisation :

> [!info] IA Étroite (Narrow AI / ANI)
> Fait une seule chose très bien, dans un domaine précis. C'est **tout ce qui existe aujourd'hui**. Exemples : GPT-4 (texte), AlphaFold (protéines), DALL-E (images). Ces systèmes sont incapables de généraliser hors de leur domaine d'entraînement.

> [!info] IA Générale (AGI — Artificial General Intelligence)
> Hypothétique système capable d'apprendre et de raisonner dans n'importe quel domaine, comme un humain. **N'existe pas encore.** Débat très actif sur si et quand on l'atteindra.

> [!info] Super-IA (ASI — Artificial Super Intelligence)
> IA qui dépasserait l'intelligence humaine dans tous les domaines. **Purement théorique.** Sujet de philosophie et de débat éthique.

> [!warning] Attention au marketing
> Quand une entreprise dit "notre IA fait X", il s'agit presque toujours d'une IA étroite très bien optimisée pour X. Ne pas confondre performance impressionnante dans un domaine avec intelligence générale.

---

## 2. IA vs Règles classiques — Quand utiliser l'une ou l'autre

### 2.1 La programmation classique

En programmation traditionnelle, le développeur définit explicitement les règles :

```python
# Approche classique : règles explicites
def classifier_email(sujet, corps):
    mots_spam = ["gagnez", "gratuit", "cliquez ici", "offre limitée"]
    
    for mot in mots_spam:
        if mot.lower() in corps.lower():
            return "SPAM"
    
    if len(corps) < 10:
        return "SPAM"
    
    return "NORMAL"
```

**Avantages** : prévisible, explicable, débuggable ligne par ligne, pas besoin de données.

**Limites** : les spammeurs s'adaptent. Il faut maintenir la liste manuellement. Un email "GAGNEZ" avec une majuscule différente passe à travers. Les règles deviennent vite des milliers de conditions.

### 2.2 L'approche Machine Learning

Au lieu de coder les règles, on **montre des exemples** au système et il apprend les règles lui-même :

```python
# Approche ML : on entraîne sur des exemples étiquetés
# (pseudo-code simplifié)

from sklearn.naive_bayes import MultinomialNB

# Données d'entraînement : [email] → [label]
emails_train = [
    ("gagnez 1000€ maintenant", "spam"),
    ("réunion demain à 14h", "normal"),
    ("offre exclusive pour vous", "spam"),
    ("ton pull est prêt chez le tailleur", "normal"),
    # ... des milliers d'exemples
]

# Le modèle apprend les patterns statistiques
modele = MultinomialNB()
modele.fit(X_train, y_train)

# Prédiction sur un nouvel email jamais vu
prediction = modele.predict(["gratuit pendant 24h seulement"])
# → "spam"
```

### 2.3 Tableau de décision : IA ou règles classiques ?

| Situation | Utiliser des règles | Utiliser le ML |
|-----------|--------------------|-----------------|
| Règles simples et stables | ✅ | ❌ sur-engineering |
| Règles trop nombreuses / complexes | ❌ | ✅ |
| Pas de données disponibles | ✅ | ❌ impossible |
| Beaucoup de données étiquetées | optionnel | ✅ préférable |
| Besoin d'explicabilité totale | ✅ | ⚠️ difficile |
| Le problème change dans le temps | ❌ maintenance lourde | ✅ ré-entraînable |
| Domaine réglementé (médical, bancaire) | ✅ (auditabilité) | ⚠️ avec précautions |
| Reconnaissance d'images, audio, texte naturel | ❌ impossible | ✅ |

> [!tip] Règle empirique Holberton
> Si vous pouvez écrire les règles sur une feuille A4, codez-les classiquement. Si vous auriez besoin d'un livre entier de règles — et que vous avez des données — pensez ML.

---

## 3. Machine Learning : les trois grands paradigmes

### 3.1 Vue d'ensemble

Le Machine Learning est le sous-domaine de l'IA qui permet aux machines d'apprendre sans être explicitement programmées pour chaque tâche. On distingue trois paradigmes selon la nature des données disponibles.

```
                    MACHINE LEARNING
                          │
          ┌───────────────┼───────────────┐
          │               │               │
   SUPERVISÉ        NON SUPERVISÉ    PAR RENFORCEMENT
   (avec labels)    (sans labels)    (rewards/pénalités)
          │               │               │
   Classification    Clustering       Jeux
   Régression        Réduction dim.   Robots
   Détection        Anomalies         Trading
```

### 3.2 Apprentissage supervisé

C'est la forme la plus commune. On dispose d'un **dataset étiqueté** : chaque exemple d'entrée est associé à la réponse correcte.

```
Données d'entrée (X)    →    Sortie attendue (y)
────────────────────────────────────────────────
Photo de chat           →    "chat"
Photo de chien          →    "chien"
Email "gagnez €€€"      →    "spam"
Email "réunion à 10h"   →    "normal"
Taille 170cm, poids 70  →    "IMC normal"
```

**Deux types de tâches supervisées :**

**Classification** : prédire une catégorie discrète.
```python
# Exemple : prédire si un patient a un diabète (0 ou 1)
from sklearn.tree import DecisionTreeClassifier

# Features : glycémie, IMC, âge, tension artérielle
X = [[120, 28.5, 45, 80],   # patient 1
     [85,  22.0, 30, 70],    # patient 2
     [200, 35.0, 55, 90]]    # patient 3

y = [1, 0, 1]  # 1 = diabétique, 0 = non-diabétique

modele = DecisionTreeClassifier()
modele.fit(X, y)

# Prédire pour un nouveau patient
nouveau_patient = [[150, 30.0, 50, 85]]
print(modele.predict(nouveau_patient))  # [1] ou [0]
```

**Régression** : prédire une valeur continue.
```python
# Exemple : prédire le prix d'un appartement
from sklearn.linear_model import LinearRegression

# Features : surface (m²), nb pièces, distance metro (km)
X = [[50, 2, 0.5],
     [80, 3, 1.2],
     [120, 4, 0.3]]

y = [200000, 280000, 420000]  # prix en euros

modele = LinearRegression()
modele.fit(X, y)

surface_cible = [[65, 2, 0.8]]
print(modele.predict(surface_cible))  # ex: [235000.]
```

### 3.3 Apprentissage non supervisé

Ici, **pas de labels** : on donne les données brutes au modèle et on lui demande de trouver des structures cachées.

**Clustering** : regrouper des données similaires.

```python
from sklearn.cluster import KMeans

# Données clients : [dépense_mensuelle, fréquence_achat]
clients = [
    [200, 2],   # client occasionnel petit budget
    [180, 3],
    [1500, 15], # client fréquent gros budget
    [1200, 12],
    [600, 8],   # client intermédiaire
    [650, 7]
]

# KMeans cherche 3 groupes naturels
kmeans = KMeans(n_clusters=3, random_state=42)
kmeans.fit(clients)

# Chaque client reçoit un label de groupe (0, 1, ou 2)
print(kmeans.labels_)  # ex: [0, 0, 2, 2, 1, 1]
# → 3 segments clients identifiés automatiquement
```

**Réduction de dimensionnalité** : compresser des données de haute dimension en 2D/3D pour visualiser.

```
Dataset avec 100 features (dimensions)
           ↓
        PCA / t-SNE / UMAP
           ↓
   2 dimensions → visualisable sur un graphique
   → on voit apparaître des clusters naturels
```

> [!info] Cas d'usage courant
> Avant d'entraîner un modèle supervisé, on utilise souvent du non-supervisé pour explorer les données : y a-t-il des groupes naturels ? Des anomalies ? Des features redondantes ? C'est de l'**analyse exploratoire**.

### 3.4 Apprentissage par renforcement

Le modèle (appelé **agent**) apprend en interagissant avec un **environnement**. Il reçoit des **récompenses** quand il fait une bonne action, des **pénalités** quand il fait une mauvaise action.

```
         ┌─────────────────────────────────┐
         │          ENVIRONNEMENT          │
         │    (jeu, simulateur, monde)     │
         └──────────┬──────────┬───────────┘
                    │ état     │ récompense
                    ▼          │
              ┌─────────┐      │
              │  AGENT  │──────┘
              │  (IA)   │
              └────┬────┘
                   │ action
                   ▼
              (modifie l'environnement)
```

**Exemples célèbres :**
- **AlphaGo / AlphaZero** (DeepMind) : apprend à jouer au Go en jouant contre lui-même
- **OpenAI Five** : apprend à jouer à Dota 2 en équipe
- **ChatGPT (RLHF)** : les humains évaluent les réponses → le modèle s'améliore (on y revient en section 5)

> [!warning] Complexité
> L'apprentissage par renforcement est le paradigme le plus difficile à mettre en œuvre. Il nécessite un environnement de simulation, beaucoup de temps de calcul, et la définition précise d'une fonction de récompense. Un objectif mal défini mène à des comportements inattendus et parfois désastreux.

### 3.5 Le pipeline ML classique

Voici les étapes d'un projet ML typique :

```
1. DÉFINIR LE PROBLÈME
   └─ Classification ? Régression ? Clustering ?
   └─ Quelle métrique de succès ? (précision, F1, RMSE...)

2. COLLECTER LES DONNÉES
   └─ Sources : bases de données, APIs, scraping, capteurs
   └─ Volume nécessaire : quelques centaines à plusieurs milliards

3. NETTOYER ET PRÉPARER
   └─ Valeurs manquantes → imputation ou suppression
   └─ Valeurs aberrantes → détection et traitement
   └─ Normalisation → mettre les features à la même échelle

4. EXPLORER (EDA — Exploratory Data Analysis)
   └─ Visualisations, statistiques descriptives
   └─ Comprendre les distributions, corrélations

5. CHOISIR ET ENTRAÎNER UN MODÈLE
   └─ Baseline simple d'abord (régression linéaire, arbre de décision)
   └─ Complexifier si nécessaire

6. ÉVALUER
   └─ Séparer train / validation / test (jamais contaminer le test)
   └─ Métriques : accuracy, précision, rappel, F1...

7. OPTIMISER (Hyperparameter Tuning)
   └─ GridSearch, RandomSearch, Bayesian Optimization

8. DÉPLOYER
   └─ API REST, batch processing, edge deployment

9. MONITORER EN PRODUCTION
   └─ Data drift, model drift, performances réelles
```

---

## 4. Deep Learning et réseaux de neurones

### 4.1 L'inspiration biologique

Le Deep Learning s'inspire (de manière très approximative) du fonctionnement des neurones biologiques.

```
NEURONE BIOLOGIQUE          NEURONE ARTIFICIEL
─────────────────────────   ──────────────────────────
Dendrites (entrées)    →    Valeurs d'entrée x₁, x₂, x₃
Corps cellulaire       →    Somme pondérée Σ(xᵢ × wᵢ) + biais
Axone (sortie)         →    Fonction d'activation → sortie y
Synapses (connexions)  →    Poids (weights) w₁, w₂, w₃
```

Un neurone artificiel fait une opération très simple :

```python
import math

def neurone(entrees, poids, biais):
    # Somme pondérée
    somme = sum(x * w for x, w in zip(entrees, poids)) + biais
    
    # Fonction d'activation (ici : ReLU)
    sortie = max(0, somme)
    
    return sortie

# Exemple
entrees = [1.0, 2.0, 3.0]
poids   = [0.5, -0.3, 0.8]
biais   = 0.1

print(neurone(entrees, poids, biais))
# somme = 1*0.5 + 2*(-0.3) + 3*0.8 + 0.1 = 0.5 - 0.6 + 2.4 + 0.1 = 2.4
# ReLU(2.4) = 2.4
```

### 4.2 Le réseau de neurones

Un réseau de neurones enchaîne plusieurs **couches** (layers) de neurones :

```
COUCHE D'ENTRÉE    COUCHES CACHÉES      COUCHE DE SORTIE
(input layer)      (hidden layers)      (output layer)

   x₁  ●────┐
             ├──● h₁₁ ──┐
   x₂  ●────┤            ├──● h₂₁ ──┐
             ├──● h₁₂ ──┤            ├──● sortie
   x₃  ●────┤            ├──● h₂₂ ──┘
             └──● h₁₃ ──┘
   x₄  ●────

  4 neurones    3 + 2 neurones        1 neurone
  (features)    (patterns abstraits)  (prédiction)
```

**Pourquoi "Deep" Learning ?**

"Deep" signifie **beaucoup de couches cachées**. Un réseau de neurones avec 2 couches cachées est peu profond. GPT-4 a des centaines de couches. Plus il y a de couches, plus le réseau peut apprendre des représentations abstraites complexes.

```
Couches basses  → patterns simples (contours, fréquences de lettres)
Couches moyennes → patterns intermédiaires (formes, mots)
Couches hautes   → concepts abstraits (visage, sentiment, intention)
```

### 4.3 L'entraînement : backpropagation

Le réseau apprend en **ajustant ses poids** pour minimiser l'erreur entre sa prédiction et la vraie réponse.

```
FORWARD PASS                    BACKWARD PASS
─────────────                   ─────────────
Entrée → prédiction             Calcul de l'erreur
                                Erreur → remontée couche par couche
                                Ajustement des poids (gradient descent)

ITÉRATION :
  1. Forward  : prédire "chat" sur une photo de chien → erreur haute
  2. Backward : calculer comment modifier chaque poids pour réduire l'erreur
  3. Update   : modifier légèrement les poids (learning rate)
  4. Répéter sur des millions d'exemples
```

> [!info] Gradient Descent
> L'algorithme qui ajuste les poids s'appelle la **descente de gradient**. Il cherche le minimum d'une fonction d'erreur en calculant la pente (gradient) et en descendant dans la direction opposée. C'est comme chercher le bas d'une vallée en regardant la pente sous vos pieds.

### 4.4 Architectures spécialisées

Différents types de réseaux sont optimisés pour différentes tâches :

| Architecture | Nom | Cas d'usage | Pourquoi |
|-------------|-----|-------------|---------|
| **CNN** | Convolutional Neural Network | Images, vidéos | Détecte des patterns locaux (contours, textures) |
| **RNN / LSTM** | Recurrent Neural Network | Séries temporelles, texte | Gère les séquences avec mémoire |
| **Transformer** | Attention Mechanism | Texte, code, images | Parallélisable, capture les dépendances longue distance |
| **GAN** | Generative Adversarial Network | Génération d'images | Deux réseaux en compétition (générateur vs discriminateur) |
| **Diffusion** | Score Matching | Images, audio, vidéo | Apprend à débruiter → génère de nouveaux exemples |

> [!tip] Pour votre niveau
> Vous n'avez pas besoin de maîtriser ces architectures maintenant. L'essentiel est de comprendre que **différentes architectures existent pour différents problèmes**, et que les LLMs (que vous utilisez tous les jours) sont basés sur les **Transformers**.

---

## 5. Les grands modèles de langage (LLMs)

### 5.1 Qu'est-ce qu'un LLM ?

Un **Large Language Model** (modèle de langage de grande taille) est un réseau de neurones de type Transformer, entraîné sur des quantités massives de texte, capable de générer du texte cohérent et de répondre à des questions.

> [!info] Définition opérationnelle
> Un LLM est un modèle statistique qui, étant donné une séquence de tokens (morceaux de texte), prédit le token suivant le plus probable. Toute la "magie" des LLMs émerge de la répétition de cette opération à très grande échelle.

**Les chiffres qui font tourner la tête :**

| Modèle | Paramètres | Tokens d'entraînement | Année |
|--------|-----------|----------------------|-------|
| GPT-2 | 1.5 milliards | 40 milliards | 2019 |
| GPT-3 | 175 milliards | 300 milliards | 2020 |
| GPT-4 | ~1 800 milliards (estimé) | ~13 000 milliards | 2023 |
| Llama 3.1 405B | 405 milliards | 15 300 milliards | 2024 |
| Claude 3.5 Sonnet | non divulgué | non divulgué | 2024 |

### 5.2 Comment fonctionne un LLM à haut niveau

**Étape 1 : Tokenisation**

Le texte brut est découpé en **tokens** (pas des mots — des sous-mots).

```
Texte : "Je programme en Python depuis 2 ans."

Tokens : ["Je", " programme", " en", " Python", " depuis", " 2", " ans", "."]
IDs    : [42,   8924,         328,   11361,      264,       17,  674,   13]
```

**Étape 2 : Embeddings**

Chaque token est converti en un **vecteur de haute dimension** (ex : 4096 nombres). Des tokens sémantiquement proches ont des vecteurs proches dans cet espace.

```
"roi"    → [0.82, 0.31, -0.15, 0.94, ...]   (4096 dimensions)
"reine"  → [0.79, 0.35, -0.12, 0.91, ...]   (proche de "roi")
"banane" → [-0.20, 0.67, 0.44, -0.33, ...]  (loin de "roi")

# Relation célèbre :
# "roi" - "homme" + "femme" ≈ "reine"
```

**Étape 3 : Mécanisme d'attention**

Le Transformer calcule pour chaque token **à quels autres tokens il doit "faire attention"** dans le contexte.

```
Phrase : "La banque du fleuve était haute."

Pour le token "banque" :
  - Attention forte vers "fleuve" (→ rive, pas institution financière)
  - Attention forte vers "haute" (contexte physique)
  - Attention faible vers "La"
  
→ Le modèle comprend que "banque" = rive ici
```

**Étape 4 : Génération (décodage)**

Le modèle génère token par token, chacun conditionné par tous les précédents :

```
Prompt : "Le langage Python est"
Token 1 prédit : " un"       (prob : 0.34)
Token 2 prédit : " langage"  (prob : 0.28)
Token 3 prédit : " de"       (prob : 0.41)
Token 4 prédit : " programmation" (prob : 0.52)
...

→ "Le langage Python est un langage de programmation..."
```

### 5.3 L'entraînement en trois phases

**Phase 1 : Pré-entraînement (Pre-training)**

Le modèle lit des milliards de pages de texte (internet, livres, code, Wikipedia...) et apprend à prédire le token suivant. Coût : des dizaines de millions d'euros, des mois sur des milliers de GPUs.

```
Texte : "Le ciel est ___"
Modèle apprend à prédire : "bleu", "nuageux", "sombre", etc.

Après des milliards d'exemples :
→ Le modèle développe une "compréhension" du monde
→ Il sait que Paris est en France, que Python est un langage, etc.
```

**Phase 2 : Fine-tuning supervisé (SFT)**

Des humains écrivent des exemples de bonnes conversations (question → réponse idéale). Le modèle est entraîné sur ces exemples pour se comporter comme un assistant.

**Phase 3 : RLHF (Reinforcement Learning from Human Feedback)**

Des humains comparent des réponses (A est mieux que B) → un modèle de récompense est entraîné → le LLM est optimisé pour maximiser cette récompense.

```
Humain évalue :
  Réponse A : "Python est un langage interprété..."  ← préférée
  Réponse B : "Python c'est pas mal mais..."

→ Modèle de récompense apprend : A > B
→ LLM ajusté pour produire des réponses de type A
```

### 5.4 Ce que les LLMs ne sont PAS

> [!warning] Malentendus fréquents à corriger
> - **Un LLM ne "comprend" pas** au sens humain. Il calcule des probabilités statistiques. Il n'a pas de croyances, d'intentions, ou d'expériences.
> - **Un LLM n'a pas accès à internet** (sauf si un outil de recherche lui est connecté explicitement). Sa connaissance est figée à sa date de coupure.
> - **Un LLM peut "halluciner"** : générer des informations fausses avec une confiance apparente. C'est une limite fondamentale, pas un bug.
> - **Un LLM n'a pas de mémoire entre sessions** (sauf architecture spécifique). Chaque conversation repart de zéro.
> - **La taille du contexte est limitée** : le modèle ne peut traiter qu'un nombre fini de tokens à la fois (4k, 32k, 200k selon les modèles).

---

## 6. Éthique IA : biais, transparence, RGPD, responsabilité

### 6.1 Les biais dans les systèmes d'IA

Un modèle apprend les patterns présents dans ses données d'entraînement — y compris les biais humains qu'elles contiennent.

**Exemple documenté — COMPAS (système de prédiction de récidive, USA) :**

```
COMPAS était utilisé par des juges américains pour évaluer le risque de récidive.

Résultat d'une étude ProPublica (2016) :
  Faux positifs (prédit récidive, pas récidivé) :
    → Afro-Américains : 45%
    → Blancs         : 23%

  Faux négatifs (prédit pas récidive, a récidivé) :
    → Afro-Américains : 28%
    → Blancs         : 48%

→ Le modèle n'était pas "neutre" — il avait appris des biais
  historiques présents dans le système judiciaire américain
```

**Types de biais courants :**

| Type | Description | Exemple |
|------|-------------|---------|
| **Biais de données** | Les données reflètent des inégalités historiques | Modèle de recrutement formé sur des CVs majoritairement masculins |
| **Biais de représentation** | Certains groupes sous-représentés dans les données | Reconnaissance faciale moins précise sur les peaux sombres |
| **Biais de confirmation** | Le modèle amplifie les patterns majoritaires | Traduction "nurse" → "infirmière" mais "doctor" → "médecin" (masculin) |
| **Biais d'automation** | Les humains font trop confiance à la sortie du modèle | Juge qui suit aveuglément COMPAS |

> [!warning] Le biais n'est pas une fatalité
> Il peut être détecté, mesuré (fairness metrics), et atténué (re-échantillonnage, post-traitement). Mais cela nécessite une démarche active — la neutralité n'est pas la valeur par défaut.

### 6.2 Transparence et explicabilité

Les modèles de Deep Learning sont des **boîtes noires** : on sait ce qu'ils produisent, pas exactement pourquoi.

```
MODÈLE TRANSPARENT (interprétable)        BOÎTE NOIRE (non interprétable)
──────────────────────────────────        ──────────────────────────────
Arbre de décision :                       Réseau de neurones profond :
  SI age > 45 ET glycémie > 125           → prédiction : diabète (87%)
  ALORS diabète (risque élevé)            → pourquoi ? aucune idée

Avantage : auditable, contestable        Avantage : plus précis
Inconvénient : moins précis              Inconvénient : inexplicable
```

**Techniques d'explicabilité (XAI — Explainable AI) :**

- **LIME** : approxime localement le modèle par un modèle simple interprétable
- **SHAP** : calcule la contribution de chaque feature à la prédiction
- **Grad-CAM** : visualise quelle zone d'une image a influencé la décision

```python
# Exemple SHAP simplifié (pseudo-code)
import shap

# Explication d'une prédiction de crédit refusé
# Feature values : [age=35, revenu=2500, dette=15000, historique=2]
# Prédiction : crédit refusé (probabilité 0.73)

explainer = shap.TreeExplainer(modele)
shap_values = explainer.shap_values(X_client)

# Sortie interprétable :
# → dette      : +0.35  (contribue fortement au refus)
# → historique : +0.28  (historique négatif)
# → revenu     : -0.12  (atténue le risque)
# → age        : +0.02  (impact faible)
```

### 6.3 RGPD et IA en Europe

Le **RGPD** (Règlement Général sur la Protection des Données, 2018) impacte directement les systèmes IA qui traitent des données personnelles.

> [!info] Articles clés du RGPD pertinents pour l'IA
> - **Article 22** : droit à ne pas être soumis à une décision automatisée ayant des effets significatifs (crédit, emploi, assurance)
> - **Article 13-14** : obligation d'informer sur l'existence d'un traitement automatisé
> - **Article 25** : privacy by design — l'IA doit être conçue dès le départ avec la protection des données
> - **Article 35** : DPIA (Data Protection Impact Assessment) obligatoire pour certains traitements IA

**Checklist RGPD pour un projet IA :**

```
□ Base légale identifiée (consentement, intérêt légitime, contrat...)
□ Minimisation des données (collecter uniquement ce qui est nécessaire)
□ Droit d'accès, rectification, effacement implémentés
□ Durée de conservation définie et respectée
□ Transferts hors UE conformes (si données vers USA, Chine...)
□ Documentation du traitement (registre des activités)
□ DPIA réalisée si traitement à risque élevé
```

**L'AI Act Européen (2024)** va plus loin :

```
RISQUE INACCEPTABLE → INTERDIT
  Ex : score social à la chinoise, manipulation psychologique

RISQUE ÉLEVÉ → RÉGLEMENTÉ STRICTEMENT
  Ex : recrutement automatisé, crédit, santé, infractions pénales
  → audit obligatoire, traçabilité, supervision humaine

RISQUE LIMITÉ → TRANSPARENCE OBLIGATOIRE
  Ex : chatbots → doit se déclarer comme IA

RISQUE MINIMAL → PAS DE RÉGLEMENTATION SPÉCIFIQUE
  Ex : jeux vidéo, spam filters
```

### 6.4 Responsabilité

> [!warning] Question clé pour vous en tant que développeur
> Quand un système IA cause un préjudice (accident de voiture autonome, décision médicale erronée, discrimination à l'embauche), **qui est responsable ?** Le développeur ? L'entreprise ? L'utilisateur ?

En France et en Europe, la jurisprudence évolue, mais les tendances actuelles :
- L'entreprise qui **déploie** le système IA est considérée responsable (comme pour un produit défectueux)
- Le développeur peut engager sa **responsabilité personnelle** en cas de faute (données illégales, tests insuffisants)
- L'utilisateur qui fait confiance aveuglément à un système IA sans supervision engage également sa responsabilité

**Principe de précaution en développement IA :**

```python
# ❌ Mauvaise pratique : déployer sans supervision humaine
def approuver_credit(client):
    return modele_ia.predict(client)  # décision automatique finale

# ✅ Bonne pratique : supervision humaine sur les cas limites
def approuver_credit(client):
    prediction = modele_ia.predict_proba(client)
    
    if prediction > 0.85:   # forte confiance refus
        return "REFUSÉ", "automatique"
    elif prediction < 0.15:  # forte confiance accord
        return "ACCORDÉ", "automatique"
    else:                    # zone grise → humain
        return "EN ATTENTE", "révision humaine requise"
```

---

## 7. Cas d'usage pratiques dans le développement logiciel

### 7.1 L'IA dans le cycle de développement

L'IA intervient aujourd'hui à presque toutes les étapes du cycle de vie d'un logiciel :

```
PLANIFICATION   →  Génération de user stories, estimation de complexité
CONCEPTION      →  Suggestions d'architecture, génération de diagrammes
DÉVELOPPEMENT   →  Autocomplétion, génération de code, refactoring
REVIEW          →  Détection de bugs, suggestions d'amélioration
TESTS           →  Génération de tests unitaires, cas limites
DOCUMENTATION   →  Génération de docstrings, README, commentaires
DÉPLOIEMENT     →  Configuration CI/CD, Dockerfile, IaC
MONITORING      →  Détection d'anomalies, alertes intelligentes
DEBUG           →  Analyse de stack traces, suggestions de corrections
```

### 7.2 Génération de code

```python
# Prompt au LLM :
# "Écris une fonction Python qui parse un fichier CSV de transactions
#  bancaires et retourne la somme par catégorie, en gérant les valeurs
#  manquantes et les formats de montants variés (1.234,56 ou 1234.56)"

# Code généré (à valider par le développeur) :
import csv
import re
from collections import defaultdict
from typing import Dict

def parse_transactions(filepath: str) -> Dict[str, float]:
    """
    Parse un fichier CSV de transactions bancaires.
    
    Args:
        filepath: Chemin vers le fichier CSV
        
    Returns:
        Dictionnaire {catégorie: montant_total}
    """
    totaux = defaultdict(float)
    
    def normaliser_montant(valeur: str) -> float:
        """Normalise différents formats de montant."""
        if not valeur or valeur.strip() == '':
            return 0.0
        # Supprimer les espaces et caractères de devise
        valeur = re.sub(r'[€$£\s]', '', valeur.strip())
        # Gérer le format européen (1.234,56) vs américain (1,234.56)
        if ',' in valeur and '.' in valeur:
            if valeur.index(',') > valeur.index('.'):
                # Format américain : 1,234.56
                valeur = valeur.replace(',', '')
            else:
                # Format européen : 1.234,56
                valeur = valeur.replace('.', '').replace(',', '.')
        elif ',' in valeur:
            valeur = valeur.replace(',', '.')
        try:
            return float(valeur)
        except ValueError:
            return 0.0
    
    with open(filepath, 'r', encoding='utf-8-sig') as f:
        reader = csv.DictReader(f)
        for ligne in reader:
            categorie = ligne.get('categorie', 'inconnue').strip()
            montant_str = ligne.get('montant', '0')
            montant = normaliser_montant(montant_str)
            totaux[categorie] += montant
    
    return dict(totaux)
```

> [!tip] Ce code généré est un point de départ, pas une vérité
> Avant de l'utiliser en production, vous devez : lire chaque ligne, comprendre la logique, ajouter des tests, vérifier les edge cases (fichier vide, colonnes manquantes, encodage incorrect).

### 7.3 Génération de tests

```bash
# Prompt au LLM :
# "Génère des tests unitaires pytest pour la fonction parse_transactions
#  ci-dessus. Couvre les cas normaux, les valeurs manquantes, les formats
#  de montants variés, et les fichiers vides."
```

```python
# Tests générés :
import pytest
import os
import csv
import tempfile

def creer_csv_temp(lignes: list) -> str:
    """Helper : crée un fichier CSV temporaire."""
    f = tempfile.NamedTemporaryFile(mode='w', suffix='.csv', 
                                     delete=False, encoding='utf-8')
    writer = csv.DictWriter(f, fieldnames=['categorie', 'montant'])
    writer.writeheader()
    writer.writerows(lignes)
    f.close()
    return f.name

class TestParseTransactions:
    
    def test_cas_normal(self):
        """Somme correcte pour des données normales."""
        chemin = creer_csv_temp([
            {'categorie': 'alimentation', 'montant': '150.00'},
            {'categorie': 'transport', 'montant': '45.50'},
            {'categorie': 'alimentation', 'montant': '80.00'},
        ])
        try:
            result = parse_transactions(chemin)
            assert result['alimentation'] == pytest.approx(230.00)
            assert result['transport'] == pytest.approx(45.50)
        finally:
            os.unlink(chemin)
    
    def test_montant_vide(self):
        """Les montants vides comptent pour 0."""
        chemin = creer_csv_temp([
            {'categorie': 'loisirs', 'montant': ''},
            {'categorie': 'loisirs', 'montant': '100.00'},
        ])
        try:
            result = parse_transactions(chemin)
            assert result['loisirs'] == pytest.approx(100.00)
        finally:
            os.unlink(chemin)
    
    def test_format_europeen(self):
        """Gère le format 1.234,56."""
        chemin = creer_csv_temp([
            {'categorie': 'loyer', 'montant': '1.200,00'},
        ])
        try:
            result = parse_transactions(chemin)
            assert result['loyer'] == pytest.approx(1200.00)
        finally:
            os.unlink(chemin)
    
    def test_fichier_vide(self):
        """Un fichier sans données retourne un dict vide."""
        chemin = creer_csv_temp([])
        try:
            result = parse_transactions(chemin)
            assert result == {}
        finally:
            os.unlink(chemin)
```

### 7.4 Review de code et détection de bugs

```javascript
// Code à review (soumis au LLM) :
async function getUserData(userId) {
  const user = await db.query(`SELECT * FROM users WHERE id = ${userId}`);
  const password = user.password;
  return { user, password };
}
```

```
Réponse du LLM (analyse) :

🔴 CRITIQUE — Injection SQL
  Le userId est interpolé directement dans la requête.
  Si userId = "1 OR 1=1", toute la table est exposée.
  Correction : utiliser des requêtes paramétrées.

🔴 SÉCURITÉ — Fuite de mot de passe
  Le hash du mot de passe est retourné dans la réponse.
  Ne jamais exposer password, même hashé.

🟡 AMÉLIORATION — Gestion d'erreur manquante
  Que se passe-t-il si userId n'existe pas ? user sera null.

Code corrigé suggéré :
  async function getUserData(userId) {
    const user = await db.query(
      'SELECT id, email, name, created_at FROM users WHERE id = $1',
      [userId]  // requête paramétrée, password exclu
    );
    if (!user.rows[0]) {
      throw new Error(`Utilisateur ${userId} introuvable`);
    }
    return user.rows[0];
  }
```

### 7.5 Documentation automatique

```python
# Fonction sans documentation
def bfs(graph, start, target):
    queue = [(start, [start])]
    visited = set()
    while queue:
        node, path = queue.pop(0)
        if node == target:
            return path
        if node not in visited:
            visited.add(node)
            for neighbor in graph.get(node, []):
                if neighbor not in visited:
                    queue.append((neighbor, path + [neighbor]))
    return None
```

```python
# Après passage au LLM (documentation générée) :
def bfs(graph: dict, start: str, target: str) -> list | None:
    """
    Parcours en largeur (Breadth-First Search) pour trouver le chemin
    le plus court entre deux nœuds dans un graphe non pondéré.
    
    Args:
        graph: Dictionnaire d'adjacence {nœud: [voisins]}.
               Exemple : {'A': ['B', 'C'], 'B': ['D'], 'C': [], 'D': []}
        start:  Nœud de départ (doit exister dans graph)
        target: Nœud cible
    
    Returns:
        Liste des nœuds constituant le chemin le plus court (start inclus),
        ou None si aucun chemin n'existe.
    
    Complexité:
        Temps  : O(V + E) où V = sommets, E = arêtes
        Espace : O(V) pour la queue et l'ensemble visited
    
    Example:
        >>> graph = {'A': ['B', 'C'], 'B': ['D'], 'C': ['D'], 'D': []}
        >>> bfs(graph, 'A', 'D')
        ['A', 'B', 'D']
    """
    queue = [(start, [start])]
    visited = set()
    while queue:
        node, path = queue.pop(0)
        if node == target:
            return path
        if node not in visited:
            visited.add(node)
            for neighbor in graph.get(node, []):
                if neighbor not in visited:
                    queue.append((neighbor, path + [neighbor]))
    return None
```

---

## 8. Outils IA pour développeurs

### 8.1 Vue d'ensemble des outils

```
ASSISTANTS DE CODE (IDE)          CHATBOTS                SPÉCIALISÉS
────────────────────────          ─────────               ───────────
GitHub Copilot                    ChatGPT (OpenAI)        Cursor (éditeur IA-first)
Codeium                           Claude (Anthropic)      Tabnine
Amazon CodeWhisperer              Gemini (Google)         Sourcegraph Cody
JetBrains AI                      Mistral (FR)            Replit AI
                                  Llama (Meta, open)      v0 (Vercel, UI)
```

### 8.2 GitHub Copilot

**Intégration** : VS Code, JetBrains, Vim, Neovim, CLI.

**Fonctionnalités :**

```python
# 1. Autocomplétion inline
# Vous tapez : "def validate_email("
# Copilot suggère automatiquement :
def validate_email(email: str) -> bool:
    """Valide le format d'une adresse email."""
    import re
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

# 2. Complétion depuis un commentaire
# Vous écrivez : # trier une liste de dicts par la clé 'age', décroissant
personnes_triees = sorted(personnes, key=lambda x: x['age'], reverse=True)

# 3. Chat inline (Copilot Chat)
# Sélection de code + /explain → explication
# Sélection de code + /fix → correction de bug
# Sélection de code + /tests → génération de tests
```

**Raccourcis clés VS Code :**
```
Tab              → accepter la suggestion
Escape           → rejeter
Alt+]            → suggestion suivante
Alt+[            → suggestion précédente
Ctrl+Enter       → voir toutes les suggestions
Ctrl+I           → ouvrir Copilot Chat inline
```

### 8.3 Claude (Anthropic)

Claude est particulièrement reconnu pour :
- La longueur de son contexte (jusqu'à 200k tokens)
- La qualité du raisonnement pas-à-pas
- La précision des analyses de code complexes
- La capacité à traiter des fichiers entiers

```
Cas d'usage typiques Claude pour le développement :

1. Analyse d'une codebase entière
   → "Voici mon fichier main.py (500 lignes). Explique l'architecture
      et identifie les problèmes potentiels."

2. Debugging complexe
   → Coller le stack trace complet + le code → diagnostic précis

3. Refactoring guidé
   → "Ce code marche mais c'est illisible. Propose une version propre
      avec les mêmes tests passants."

4. Architecture
   → "Je dois concevoir un système de notification. Voici mes contraintes.
      Quelles sont les options et trade-offs ?"
```

### 8.4 ChatGPT (OpenAI)

```
Points forts pour les développeurs :

→ GPT-4o : multimodal (images + texte)
  Exemple : screenshot d'une erreur → analyse et correction

→ Code Interpreter (plugin) : exécute du Python dans un sandbox
  Exemple : "Analyse ce CSV et génère des graphiques" → exécution réelle

→ Plugins / Actions personnalisés : intégration avec APIs externes

→ DALL-E 3 intégré : génération d'images pour la documentation,
  wireframes, assets
```

### 8.5 Gemini (Google)

```
Points forts pour les développeurs :

→ Gemini 1.5 Pro : 1 million de tokens de contexte
  → Analyser une codebase entière, des logs volumineux

→ Intégration Google Workspace : Docs, Sheets, Gmail, Drive
  → Automatisation de workflows documentaires

→ Google AI Studio : playground gratuit pour les APIs
  → Expérimenter avec les modèles avant d'intégrer en production

→ Vertex AI : plateforme MLOps pour le déploiement enterprise
```

### 8.6 Tableau comparatif des LLMs pour développeurs

| Critère | GitHub Copilot | Claude | ChatGPT | Gemini |
|---------|---------------|--------|---------|--------|
| **Intégration IDE** | ✅ native | ❌ (CLI/API) | ❌ (plugins) | ✅ (via extensions) |
| **Contexte long** | moyen | ✅ 200k tokens | moyen | ✅ 1M tokens |
| **Génération code** | ✅ excellente | ✅ très bon | ✅ très bon | ✅ bon |
| **Raisonnement** | bon | ✅ excellent | ✅ excellent | ✅ très bon |
| **Exécution code** | ❌ | ❌ | ✅ Code Interpreter | ✅ (Google Colab) |
| **Prix (mai 2025)** | 10€/mois | 18€/mois | 20€/mois | 20€/mois |
| **Version gratuite** | limité | ✅ Claude.ai | ✅ GPT-3.5 | ✅ Gemini 1.0 |
| **Données privées** | ⚠️ à vérifier | ✅ opt-out | ⚠️ à vérifier | ⚠️ à vérifier |

> [!tip] Conseil pratique
> En tant qu'étudiant, utilisez les versions gratuites pour apprendre. Évitez de mettre du code propriétaire ou des secrets dans n'importe quel LLM. GitHub Copilot propose une offre gratuite pour les étudiants via le GitHub Student Developer Pack.

---

## 9. Prompting de base pour le code

### 9.1 Pourquoi le prompting compte

Un LLM est un outil probabiliste : la qualité du résultat dépend directement de la qualité de la question. Un prompt vague donne un résultat générique. Un prompt précis donne un résultat actionnable.

> [!info] Définition
> Le **prompt engineering** est l'art de formuler des instructions à un LLM pour obtenir la sortie souhaitée. Ce n'est pas de la magie — c'est de la communication précise.

### 9.2 Les éléments d'un bon prompt

Un prompt efficace contient généralement :

```
1. RÔLE      → Qui est le modèle dans ce contexte ?
2. CONTEXTE  → Quelle est la situation, quelles sont les contraintes ?
3. TÂCHE     → Que doit-il faire exactement ?
4. FORMAT    → Quel format de sortie attendez-vous ?
5. EXEMPLES  → (optionnel) Des exemples de bonne sortie
```

### 9.3 Exemples de prompts inefficaces vs efficaces

**Pour de la génération de code :**

```
❌ Prompt vague :
"Fais une fonction pour les utilisateurs"

Résultat : code générique inutilisable, trop d'hypothèses


✅ Prompt précis :
"Tu es un développeur Python senior qui suit les conventions PEP 8.

Contexte : application web FastAPI avec SQLAlchemy ORM.
La table 'users' a les colonnes : id, email, username, hashed_password,
created_at, is_active.

Tâche : écris la fonction `get_user_by_email(db: Session, email: str)`
qui retourne l'utilisateur correspondant ou None s'il n'existe pas.
Inclure la gestion du cas où email est vide ou None.

Format : fonction avec type hints, docstring Google style, et 3 tests
pytest couvrant : trouvé, non trouvé, email vide."
```

**Pour du debugging :**

```
❌ Prompt vague :
"Mon code ne marche pas"


✅ Prompt précis :
"Voici une erreur Python que j'obtiens :

```
Traceback (most recent call last):
  File "app.py", line 47, in process_order
    total = sum(item['price'] * item['quantity'] for item in order['items'])
KeyError: 'price'
```

Voici la structure de mes données :
order = {'id': 42, 'items': [{'product_id': 1, 'qty': 2, 'unit_price': 19.99}]}

Voici la fonction complète :
[coller la fonction]

Question : identifie le bug et propose une correction robuste qui
gère les clés manquantes."
```

### 9.4 Techniques de prompting avancées

**Chain of Thought (raisonnement étape par étape) :**

```
"Résous ce problème étape par étape, en montrant ton raisonnement :

J'ai une liste de 10 000 transactions. Pour chaque transaction, je dois :
1. Vérifier si le montant dépasse le seuil de 1000€
2. Si oui, appliquer une taxe de 15%
3. Grouper par catégorie et calculer les totaux

Mon code actuel prend 30 secondes. Comment l'optimiser ?
Montre d'abord l'analyse du problème de performance, puis propose
une solution."
```

**Few-shot prompting (exemples) :**

```
"Convertis ces descriptions de fonctions en signatures TypeScript.
Voici le format attendu :

Entrée : 'une fonction qui prend un nom et un âge et retourne une string de présentation'
Sortie : function introduce(name: string, age: number): string

Entrée : 'une fonction async qui fetch une URL et retourne les données JSON ou null en cas d'erreur'
Sortie : async function fetchJson(url: string): Promise<unknown | null>

Maintenant convertis ces descriptions :
1. 'une fonction qui filtre une liste d'utilisateurs par pays'
2. 'une fonction qui calcule la distance entre deux points GPS'"
```

**Demander une critique de son propre code :**

```
"Joue le rôle d'un senior developer qui fait une code review sévère.
Ton objectif est de trouver TOUS les problèmes, pas de ménager l'ego.

Voici le code :
[coller le code]

Analyse selon ces critères :
- Correctness (le code fait-il vraiment ce qu'il prétend ?)
- Performance (y a-t-il des O(n²) évitables ?)
- Sécurité (injections, expositions de données ?)
- Lisibilité (un dev junior peut-il comprendre en 5 minutes ?)
- Tests (les cas limites sont-ils couverts ?)

Format : une liste markdown avec emoji de sévérité (🔴 critique, 🟡 amélioration, 🟢 suggestion)"
```

### 9.5 Prompts de référence pour les projets Holberton

```bash
# Pour du C (projets bas niveau Holberton) :
"Tu es un expert C C99. Tous tes exemples sont conformes à la norme Betty.
Pas de dynamic allocation sauf si nécessaire. Commentaires en anglais.
[ta question]"

# Pour du Python (projets algorithmiques) :
"Python 3.10+, PEP 8, type hints obligatoires, pas de librairies externes.
[ta question]"

# Pour expliquer un algorithme :
"Explique [nom algorithme] comme si j'avais 0 connaissance préalable.
1. Intuition en français simple
2. Pseudo-code commenté
3. Implémentation Python avec complexité expliquée
4. Un exemple tracé pas à pas sur des données concrètes"

# Pour débugger :
"Voici le contexte : [langage], [environnement].
Voici l'erreur exacte : [coller le message d'erreur]
Voici le code minimal qui reproduit le bug : [coller le code]
Identifie la cause racine et propose 2 solutions alternatives."
```

---

## 10. Limites et responsabilité du développeur augmenté par IA

### 10.1 Ce que l'IA ne peut pas remplacer

> [!warning] Pensée critique irremplaçable
> Un LLM génère du code statistiquement probable, pas nécessairement correct pour votre contexte. Il ne connaît pas votre architecture, vos contraintes business, vos données réelles, vos conventions d'équipe. Il génère ce qui ressemble à une bonne réponse — pas nécessairement ce qui est une bonne réponse dans votre situation précise.

**Les compétences qui restent humaines :**

```
COMPRÉHENSION DU PROBLÈME
→ Savoir POURQUOI on fait quelque chose
→ L'IA optimise la solution que vous lui décrivez
   Si vous décrivez le mauvais problème → solution parfaite pour le mauvais problème

JUGEMENT ARCHITECTURAL
→ Décider des trade-offs : performance vs lisibilité, couplage vs flexibilité
→ L'IA suggère des patterns communs, pas les mieux adaptés à votre contexte

VALIDATION ET TESTS
→ Comprendre si le code généré est correct
→ Tester les edge cases que le LLM n'a pas anticipés

RESPONSABILITÉ PROFESSIONNELLE
→ Le code que vous signez, c'est le vôtre
→ Si le code de l'IA contient un bug de sécurité déployé en prod → c'est vous

CONNAISSANCE DOMAINE
→ Développer un système médical nécessite des connaissances médicales
→ Un LLM peut générer du code qui "semble correct" mais viole des contraintes métier critiques
```

### 10.2 Les hallucinations et leurs conséquences

Les LLMs peuvent générer des informations fausses avec une confiance apparente absolue.

**Exemples réels de problèmes documentés :**

```
1. BIBLIOTHÈQUES INEXISTANTES
   L'IA génère :
   from pandas import smart_groupby  # cette fonction n'existe pas
   
   Le développeur débutant cherche pendant 2h pourquoi l'import échoue.

2. APIs PÉRIMÉES
   L'IA génère du code pour une API dont la version a changé.
   Les paramètres ont changé de nom. Le code ne fonctionne pas.
   
   Pourquoi : la donnée d'entraînement du LLM a une date de coupure.
   Les LLMs ne connaissent pas les changements récents.

3. VULNÉRABILITÉS DE SÉCURITÉ
   Des chercheurs ont montré (Stanford 2022) que 40% du code
   généré par Copilot sur des tâches de sécurité contenait des
   vulnérabilités exploitables.
   
   Exemple : code de hachage de mot de passe avec MD5 (algorithme cassé)
   présenté comme "sécurisé".

4. LOGIQUE SUBTILE INCORRECTE
   def is_leap_year(year):
       return year % 4 == 0  # FAUX ! Incomplet.
   
   Correct : (year % 4 == 0 and year % 100 != 0) or (year % 400 == 0)
   
   Un humain sans connaissance du domaine ne verrait pas le bug.
```

### 10.3 Protocole de validation du code généré

Avant d'utiliser du code généré par IA en production, appliquez ce protocole :

```
ÉTAPE 1 : COMPRENDRE
□ Je peux expliquer ce que fait chaque ligne à voix haute
□ Je comprends pourquoi cette approche a été choisie
□ Je connais les alternatives qui auraient pu être choisies
→ Si vous ne pouvez pas expliquer, vous ne comprenez pas. Ne déployez pas.

ÉTAPE 2 : VÉRIFIER LES DÉPENDANCES
□ Toutes les bibliothèques importées existent réellement
□ Les versions utilisées sont compatibles avec votre projet
□ Pas de dépendance obscure avec peu d'historique (risque supply chain)

ÉTAPE 3 : TESTER LES CAS LIMITES
□ Input vide ou null
□ Input négatif ou hors plage
□ Input très grand (performance)
□ Input malicieux (sécurité)
□ Comportement en cas d'erreur

ÉTAPE 4 : REVUE DE SÉCURITÉ MINIMALE
□ Pas d'injection (SQL, shell, path traversal)
□ Pas de données sensibles exposées (logs, retours d'API, erreurs)
□ Authentification et autorisation correctes
□ Pas de cryptographie home-made ou algorithme déprécié

ÉTAPE 5 : COHÉRENCE AVEC LE PROJET
□ Respecte les conventions du projet (nommage, style, patterns)
□ S'intègre dans l'architecture existante
□ Pas de duplication de fonctionnalité déjà présente
```

### 10.4 L'IA comme pair junior, pas comme oracle

La bonne façon de penser à un LLM en développement :

```
❌ MAUVAISE POSTURE : "L'IA a généré ce code, donc c'est correct."

✅ BONNE POSTURE : "L'IA m'a proposé une piste. Je vais l'analyser,
   la tester, l'adapter à mon contexte, et en prendre la responsabilité."
```

> [!tip] Analogie utile
> Imaginez un stagiaire très talentueux qui a lu tous les livres de programmation mais n'a jamais travaillé sur un vrai projet. Il peut proposer des solutions brillantes et des solutions complètement à côté. Votre rôle est celui du senior developer qui guide, valide et prend les décisions finales.

### 10.5 Compétences fondamentales à ne pas déléguer à l'IA

Même avec les meilleurs outils IA disponibles, certaines compétences restent indispensables pour un développeur compétent :

| Compétence | Pourquoi impossible à déléguer |
|------------|-------------------------------|
| **Lire et comprendre du code** | L'IA peut se tromper dans son explication |
| **Debugging à partir de premiers principes** | L'IA donne des suggestions, pas des certitudes |
| **Comprendre les algorithmes et complexité** | Impossible de valider les choix sans ces bases |
| **Sécurité** | Une faille de sécurité non détectée = compromission réelle |
| **Architecture système** | L'IA propose des patterns génériques sans connaître vos contraintes |
| **Communication avec l'équipe** | L'IA ne peut pas défendre vos choix techniques |
| **Tests** | L'IA génère des tests, mais ne sait pas ce qui est important à tester |
| **Gestion d'erreurs** | Savoir quand panic vs. récupérer est une décision de domaine |

> [!warning] Danger spécifique aux débutants
> Utiliser l'IA pour contourner l'apprentissage fondamental est contre-productif sur le long terme. Si vous ne comprenez pas les pointeurs en C grâce aux cours Holberton, l'IA pourra générer du code C — mais vous serez incapable de le debugger quand il plantera à 3h du matin en production.

### 10.6 Le développeur augmenté : nouvelle définition du métier

L'IA ne va pas remplacer les développeurs. Elle redéfinit ce qu'un développeur fait de sa journée :

```
AVANT L'IA (2015)                   AVEC L'IA (2025)
─────────────────────────────────────────────────────
50% écrire du boilerplate         → 10% valider le boilerplate généré
20% chercher la doc               → 5% vérifier les sources
15% debugging basique             → 5% debugging complexe (l'IA gère le basique)
15% tâches créatives/architecture → 80% tâches créatives/architecture !
```

**Le développeur augmenté par l'IA :**
- Passe moins de temps sur les tâches répétitives
- Passe plus de temps sur la conception, les décisions d'architecture, les revues
- Doit avoir une compréhension plus profonde pour valider correctement
- Doit maîtriser le prompting comme compétence métier à part entière

---

## Exercices pratiques

### Exercice 1 — Classifier les approches

Pour chacun des problèmes suivants, indiquez si vous utiliseriez des règles classiques, du ML supervisé, du ML non supervisé, ou du deep learning. Justifiez votre choix.

```
1. Calculer la TVA sur un montant selon le pays
2. Détecter si une image contient un chat
3. Recommander des produits à un utilisateur e-commerce
4. Vérifier qu'un email est bien formé (format valide)
5. Prédire si un client va churner (quitter) dans les 30 prochains jours
6. Identifier des groupes de clients aux comportements similaires
7. Traduire du texte français en anglais
8. Vérifier qu'un champ de formulaire n'est pas vide
9. Détecter des transactions bancaires frauduleuses en temps réel
10. Générer des sous-titres automatiques pour une vidéo
```

### Exercice 2 — Améliorer un prompt

Voici un prompt de qualité médiocre. Réécrivez-le pour obtenir un meilleur résultat.

```
Prompt original :
"Aide-moi avec mon code Python qui fait un truc avec des fichiers"

Votre version améliorée doit contenir :
- Le contexte technique exact
- La tâche précise
- Les contraintes ou exigences
- Le format de réponse attendu
```

### Exercice 3 — Revue de code généré par IA

Voici du code Python généré par un LLM. Identifiez tous les problèmes potentiels.

```python
import subprocess
import os

def execute_user_script(script_name, user_input):
    """Execute un script utilisateur avec ses paramètres."""
    # Construire la commande
    cmd = f"python scripts/{script_name}.py {user_input}"
    
    # Exécuter et retourner le résultat
    result = subprocess.run(cmd, shell=True, capture_output=True)
    
    # Logger pour debug
    print(f"Commande exécutée : {cmd}")
    print(f"Utilisateur : {os.environ.get('DB_PASSWORD')}")
    
    return result.stdout.decode('utf-8')
```

```
Questions :
1. Quels problèmes de sécurité identifiez-vous ?
2. Quelles données sensibles sont exposées ?
3. Quels sont les edge cases non gérés ?
4. Proposez une version corrigée.
```

### Exercice 4 — Pipeline ML en pseudo-code

Vous devez construire un système qui prédit si un commentaire sur un site e-commerce est positif ou négatif (analyse de sentiment).

```
Décrivez en pseudo-code ou en langage naturel :
1. Quelles données collecteriez-vous ? (type, volume, source)
2. Comment les prépariez-vous ?
3. Quelle architecture de modèle choisiriez-vous et pourquoi ?
4. Comment évalueriez-vous le modèle ?
5. Quels biais potentiels anticipez-vous ?
6. Comment déploieriez-vous le modèle en production ?
```

### Exercice 5 — Éthique et cas pratique

```
Scénario :
Une startup vous demande de développer un système IA qui analyse
les profils LinkedIn des candidats à un poste et leur attribue
un score d'employabilité de 0 à 100. Ce score est utilisé pour
filtrer les CV avant qu'un humain ne les lise.

Questions :
1. Quels biais ce système pourrait-il amplifier ?
2. Quelles données d'entraînement utiliseriez-vous ? Lesquelles éviteriez-vous ?
3. Est-ce conforme au RGPD ? À l'AI Act européen ?
4. Quelles sauvegardes techniques et processuelles proposeriez-vous ?
5. Y a-t-il des aspects de ce projet que vous refuseriez de développer ?
   Justifiez votre position.
```

---

## Résumé et points clés

```
SECTION 1 : L'IA aujourd'hui
✓ L'IA étroite (ANI) est la seule qui existe : performante dans un domaine précis
✓ L'AGI reste hypothétique — ne pas confondre performance et intelligence générale
✓ Le Deep Learning (depuis 2012) et les Transformers (2017) ont tout changé

SECTION 2 : IA vs règles classiques
✓ Règles si : simple, stable, pas de données, explicabilité requise
✓ ML si : complexe, données disponibles, problème changeant dans le temps

SECTION 3 : Les trois paradigmes ML
✓ Supervisé : données étiquetées → classification ou régression
✓ Non supervisé : données brutes → clusters, anomalies, réduction dim.
✓ Renforcement : agent + environnement + récompenses → politique optimale

SECTION 4 : Deep Learning
✓ Neurones artificiels + couches → apprentissage de représentations abstraites
✓ Entraînement = forward pass + backward pass (backpropagation) + gradient descent
✓ CNN (images), RNN (séquences), Transformer (texte, code) → architectures spécialisées

SECTION 5 : LLMs
✓ Prédiction du token suivant à très grande échelle → comportements émergents
✓ Pré-entraînement → SFT → RLHF
✓ Hallucinations, contexte limité, coupure temporelle : limites fondamentales

SECTION 6 : Éthique
✓ Les biais entrent par les données, sortent amplifiés par le modèle
✓ RGPD Art.22 : droit à ne pas être soumis à une décision 100% automatisée
✓ Qui déploie = qui est responsable en cas de préjudice

SECTION 7 : Cas d'usage dev
✓ L'IA assiste à toutes les étapes : génération, debug, tests, docs, review
✓ Chaque sortie doit être validée par un humain compétent

SECTION 8 : Outils
✓ Copilot : meilleure intégration IDE
✓ Claude : meilleur pour les longs contextes et raisonnement
✓ ChatGPT : meilleur pour l'exécution de code (Code Interpreter)
✓ Gemini : meilleur pour les très longs contextes (1M tokens)

SECTION 9 : Prompting
✓ Rôle + Contexte + Tâche + Format = prompt efficace
✓ Chain of Thought pour les problèmes complexes
✓ Few-shot pour formater la sortie précisément

SECTION 10 : Responsabilité
✓ L'IA génère du code probable, pas nécessairement correct pour votre contexte
✓ Vous signez le code → vous êtes responsable, même s'il a été généré
✓ Les compétences fondamentales (algorithmes, sécurité, architecture) restent non délégables
✓ L'IA augmente le développeur, elle ne le remplace pas
```

> [!tip] Pour aller plus loin
> - **Cours ML pratique** : fast.ai (gratuit, pratique-first)
> - **Théorie** : "Hands-On Machine Learning" de Géron (O'Reilly)
> - **LLMs en profondeur** : "Attention is All You Need" (paper original Transformer, 2017)
> - **Éthique IA** : AI Now Institute, AlgorithmWatch
> - **Légal** : texte de l'AI Act européen (europa.eu)
> - **Pratique** : Kaggle pour les datasets et compétitions ML gratuites
