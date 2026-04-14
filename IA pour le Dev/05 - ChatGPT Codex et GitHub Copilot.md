# 05 - ChatGPT, Codex et GitHub Copilot

## Qu'est-ce que l'écosystème OpenAI pour les développeurs ?

OpenAI est l'une des organisations les plus influentes de l'histoire récente de l'intelligence artificielle. Fondée en 2015, elle a bouleversé l'industrie du logiciel avec la sortie de GPT-3 en 2020, puis de ChatGPT en novembre 2022 — qui a atteint 100 millions d'utilisateurs en deux mois, un record absolu. Pour les développeurs, OpenAI n'est pas seulement un chatbot : c'est un écosystème complet allant de l'assistant conversationnel aux outils d'intégration dans l'IDE, en passant par une API robuste et des modèles spécialisés pour le raisonnement.

Cette note couvre les trois piliers de l'offre OpenAI pour les développeurs : **ChatGPT** (interface conversationnelle), **l'API OpenAI** (intégration programmatique), et **GitHub Copilot** (assistant directement dans l'éditeur de code).

---

## 1. L'écosystème OpenAI pour les développeurs

### Les trois piliers

> [!info] Les 3 piliers OpenAI
> OpenAI propose une approche à plusieurs niveaux selon le profil du développeur :
> - **ChatGPT** : Interface web/mobile pour interactions conversationnelles
> - **API OpenAI** : Intégration dans des applications et workflows personnalisés
> - **GitHub Copilot** : Assistant IA directement dans l'IDE, intégré au flux de code

```
┌─────────────────────────────────────────────────────────────┐
│                  ÉCOSYSTÈME OPENAI (2025-2026)              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│   │   ChatGPT    │  │  API OpenAI  │  │ GitHub       │    │
│   │              │  │              │  │ Copilot      │    │
│   │ - Web/Mobile │  │ - REST API   │  │              │    │
│   │ - Canvas     │  │ - SDK Python │  │ - VS Code    │    │
│   │ - Projects   │  │ - SDK Node   │  │ - JetBrains  │    │
│   │ - Code Interp│  │ - Streaming  │  │ - Neovim     │    │
│   │ - GPTs       │  │ - Embeddings │  │ - Terminal   │    │
│   └──────┬───────┘  └──────┬───────┘  └──────┬───────┘    │
│          │                 │                  │            │
│          └─────────────────┴──────────────────┘            │
│                            │                               │
│                    ┌───────▼────────┐                      │
│                    │  Modèles IA    │                      │
│                    │  GPT-4o, o3,   │                      │
│                    │  o3-mini, etc. │                      │
│                    └────────────────┘                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Impact sur l'industrie

L'arrivée de GitHub Copilot (basé initialement sur Codex, un modèle dérivé de GPT-3) en 2021 a marqué un tournant : pour la première fois, l'IA devenait un compagnon de codage en temps réel. Selon GitHub, les développeurs utilisant Copilot produisent du code **55% plus vite** sur certaines tâches. Depuis, l'ensemble de l'industrie s'est aligné : JetBrains AI Assistant, Cursor, Codeium, tous s'inspirent du modèle Copilot.

---

## 2. Les modèles OpenAI en 2025-2026

OpenAI maintient une famille de modèles aux caractéristiques très différentes. Comprendre lequel utiliser est crucial pour optimiser coût, vitesse et qualité.

### GPT-4o — Le modèle polyvalent

**GPT-4o** ("o" pour "omni") est le modèle phare d'OpenAI pour un usage quotidien. Il est multimodal : il accepte du texte, des images, de l'audio et peut générer du texte et de l'audio.

- Fenêtre de contexte : **128 000 tokens**
- Vitesse : rapide (génération streaming fluide)
- Qualité code : excellente pour la majorité des tâches
- Prix (API, avril 2026 approximatif) : ~$5 / 1M tokens input, ~$15 / 1M tokens output

> [!tip] Analogie
> GPT-4o est comme un développeur senior polyvalent : il connaît tous les langages, répond vite, et gère bien les projets de taille standard. Il n'est pas le meilleur en mathématiques pures, mais c'est votre collègue idéal pour 90% des tâches quotidiennes.

### GPT-4o-mini — Le modèle économique

Version allégée de GPT-4o, optimisée pour le rapport qualité/coût.

- Contexte : **128 000 tokens**
- Vitesse : très rapide
- Qualité : bonne pour tâches simples, génération de contenu, classification
- Prix : ~$0.15 / 1M tokens input, ~$0.60 / 1M tokens output (environ 30x moins cher que GPT-4o)

### o1 — Le modèle de raisonnement

**o1** introduit une approche radicalement différente : le modèle "pense" avant de répondre via une chaîne de pensée interne (chain-of-thought). Il prend plus de temps, mais produit des réponses beaucoup plus précises sur les problèmes complexes.

- Contexte : 128 000 tokens
- Vitesse : lente (les tokens de raisonnement ne sont pas visibles mais sont facturés)
- Qualité code complexe : supérieure à GPT-4o sur les algos difficiles
- Prix : élevé (~$15 / 1M input, ~$60 / 1M output)

### o1-mini — Version allégée de o1

Conserve les capacités de raisonnement d'o1 mais avec moins de paramètres. Bon compromis pour le raisonnement mathématique sans payer le prix plein d'o1.

### o3 — Le successeur SOTA

**o3** (2025) est le successeur d'o1 avec des capacités de raisonnement significativement améliorées. Il atteint des scores "state of the art" (SOTA) sur des benchmarks mathématiques, scientifiques et de programmation compétitive (ARC-AGI, AIME, etc.).

- Raisonnement : le meilleur disponible via OpenAI
- Usage recommandé : problèmes algorithmiques difficiles, preuves mathématiques, debugging de systèmes complexes
- Prix : très élevé

### o3-mini — Équilibre raisonnement/coût

Offre une grande partie des capacités d'o3 à un coût beaucoup plus raisonnable. C'est souvent le meilleur choix quand on a besoin de raisonnement mais pas de la puissance maximale d'o3.

### o4-mini — Multimodal léger et raisonnement

**o4-mini** (2025-2026) combine les capacités de raisonnement des modèles "o" avec la multimodalité (vision) dans un package économique. Il peut analyser des images et raisonner dessus, ce qui en fait un outil puissant pour les développeurs travaillant avec des interfaces visuelles ou du code contenant des diagrammes.

---

### Tableau comparatif des modèles

```
┌─────────────┬──────────┬──────────────┬──────────┬────────────┬─────────────┐
│  Modèle     │ Vitesse  │ Qualité code │ Contexte │ Prix input │ Prix output │
├─────────────┼──────────┼──────────────┼──────────┼────────────┼─────────────┤
│ GPT-4o      │ Rapide   │ ★★★★☆        │ 128k     │ $5/1M      │ $15/1M      │
│ GPT-4o-mini │ Très rap.│ ★★★☆☆        │ 128k     │ $0.15/1M   │ $0.60/1M    │
│ o1          │ Lente    │ ★★★★★ (algo) │ 128k     │ $15/1M     │ $60/1M      │
│ o1-mini     │ Moyenne  │ ★★★★☆ (algo) │ 128k     │ $3/1M      │ $12/1M      │
│ o3          │ Lente    │ ★★★★★ (SOTA) │ 128k+    │ Élevé      │ Très élevé  │
│ o3-mini     │ Moyenne  │ ★★★★☆        │ 128k     │ $1.1/1M    │ $4.4/1M     │
│ o4-mini     │ Rapide   │ ★★★★☆ (+vis.)│ 128k     │ $1.1/1M    │ $4.4/1M     │
└─────────────┴──────────┴──────────────┴──────────┴────────────┴─────────────┘
  Note : Prix approximatifs, vérifier platform.openai.com/pricing pour les tarifs actuels
```

### Quand utiliser quel modèle ?

```
ARBRE DE DÉCISION — CHOIX DU MODÈLE

Est-ce une tâche simple ?
(autocomplétion, reformulation, résumé court)
       │
       ├── OUI → GPT-4o-mini (économique et rapide)
       │
       └── NON → Est-ce un problème algorithmique/mathématique difficile ?
                        │
                        ├── OUI → Besoin du meilleur résultat ?
                        │              ├── OUI → o3
                        │              └── NON → o3-mini ou o4-mini
                        │
                        └── NON → Tâche générale de qualité ?
                                       ├── OUI → GPT-4o
                                       └── Vision/Multimodal ? → o4-mini
```

---

## 3. GitHub Copilot

### Qu'est-ce que GitHub Copilot ?

GitHub Copilot est un assistant de programmation IA développé par GitHub (Microsoft) en partenariat avec OpenAI. Il s'intègre directement dans l'éditeur de code et suggère du code en temps réel. Lancé en 2021, il est devenu en quelques années l'outil d'IA le plus utilisé par les développeurs professionnels.

```
┌──────────────────────────────────────────────────────────┐
│                  GITHUB COPILOT - VUE D'ENSEMBLE        │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │                   VS CODE / IDE                  │   │
│  │                                                  │   │
│  │  def calculate_fibonacci(n):                     │   │
│  │      """Retourne le n-ième nombre de Fibonacci"""│   │
│  │      [suggestion grise apparaît ici...]          │   │
│  │      if n <= 1:                    ← Copilot     │   │
│  │          return n                 ← suggère      │   │
│  │      return calculate_fibonacci(n-1) + ...       │   │
│  │                                                  │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│                          ▼                               │
│              ┌───────────────────────┐                   │
│              │  Modèles OpenAI       │                   │
│              │  (GPT-4o, Codex-based)│                   │
│              └───────────────────────┘                   │
└──────────────────────────────────────────────────────────┘
```

### Installation dans VS Code

```
ÉTAPES D'INSTALLATION :

1. Ouvrir VS Code
   └─ Extensions (Ctrl+Shift+X)
       └─ Rechercher "GitHub Copilot"
           └─ Installer l'extension officielle (GitHub)
               └─ Redémarrer VS Code si demandé

2. Connexion
   └─ Une notification apparaît → "Sign in to GitHub"
       └─ Ouvrir le navigateur → Autoriser l'accès
           └─ Retour dans VS Code → Copilot activé ✓

3. Prérequis : Abonnement GitHub Copilot actif
   (Individual, Business ou Enterprise)
```

> [!info] Extension complémentaire
> Installe aussi **"GitHub Copilot Chat"** pour le panneau de conversation latéral. Les deux extensions fonctionnent ensemble. Depuis fin 2024, elles sont souvent regroupées dans une seule extension "GitHub Copilot".

### Fonctionnalités détaillées

#### Complétion inline (Ghost Text)

La fonctionnalité principale : Copilot analyse le contexte de ton fichier et propose des suggestions en texte gris (ghost text). Ces suggestions apparaissent automatiquement pendant que tu tapes.

```python
# Exemple : tu écris le commentaire, Copilot génère le code

# Fonction pour valider un email avec regex
def validate_email(email):
    # ↓ Copilot suggère automatiquement :
    import re
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

#### Copilot Chat (panneau latéral)

Un panneau de conversation intégré permettant de poser des questions sur le code, demander des refactorisations, des explications, etc.

Commandes slash utiles dans Copilot Chat :
- `/explain` : expliquer le code sélectionné
- `/fix` : corriger un bug dans la sélection
- `/tests` : générer des tests pour la sélection
- `/doc` : générer de la documentation
- `/simplify` : simplifier le code sélectionné

#### Copilot en ligne de commande (terminal)

Copilot peut aussi suggérer des commandes shell dans le terminal intégré VS Code. Utile pour les commandes git, docker, ou les scripts bash complexes.

```bash
# Exemple : tu commences à taper, Copilot complète
git log --oneline --graph --decorate --all
# ↑ Copilot peut suggérer cette commande après que tu aies
# tapé "git log" et un commentaire comme "# afficher l'historique graphique"
```

#### Copilot pour les tests

Copilot est particulièrement efficace pour générer des tests unitaires. Sélectionne une fonction, tape `/tests` dans le chat, et Copilot génère un fichier de tests complet.

```python
# Fonction source
def divide(a: float, b: float) -> float:
    """Divise a par b, lève ValueError si b est zéro."""
    if b == 0:
        raise ValueError("Division par zéro impossible")
    return a / b

# Copilot génère automatiquement :
import pytest

def test_divide_normal():
    assert divide(10, 2) == 5.0

def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(10, 0)

def test_divide_negative():
    assert divide(-6, 2) == -3.0

def test_divide_floats():
    assert divide(7.5, 2.5) == pytest.approx(3.0)
```

#### Copilot pour les commit messages

Dans le panneau Source Control de VS Code, Copilot peut générer automatiquement un message de commit basé sur le diff. Clique sur l'icône "sparkle" (étoile) dans le champ de message de commit.

---

### Raccourcis VS Code essentiels

```
┌────────────────────────────────────────────────────────┐
│              RACCOURCIS GITHUB COPILOT                 │
├────────────────────────────┬───────────────────────────┤
│ Action                     │ Raccourci                  │
├────────────────────────────┼───────────────────────────┤
│ Accepter la suggestion     │ Tab                        │
│ Refuser la suggestion      │ Esc                        │
│ Suggestion suivante        │ Alt + ]                    │
│ Suggestion précédente      │ Alt + [                    │
│ Ouvrir Copilot Chat        │ Ctrl + Shift + I           │
│ Inline Chat (dans le code) │ Ctrl + I                   │
│ Copilot dans le terminal   │ Ctrl + I (dans le terminal)│
│ Ouvrir panneau complétion  │ Alt + \                    │
└────────────────────────────┴───────────────────────────┘
```

> [!warning] Raccourcis variables
> Les raccourcis peuvent varier selon la version de VS Code et ton système d'exploitation. Consulte les keybindings dans VS Code (Ctrl+K, Ctrl+S) pour voir les raccourcis exacts de ta configuration.

### Bonnes pratiques avec Copilot

> [!tip] Analogie
> Copilot est comme un développeur junior très réactif assis à côté de toi. Plus tu lui donnes de contexte (noms clairs, commentaires, types), plus ses suggestions seront pertinentes. Ne lui fais pas confiance aveuglément — relis toujours le code généré.

**1. Nommer ses fonctions clairement**

```python
# ❌ Mauvais — Copilot ne sait pas quoi générer
def process(d):
    ...

# ✓ Bien — Copilot comprend l'intention
def extract_unique_user_emails_from_database(db_connection):
    ...
```

**2. Écrire les commentaires AVANT le code**

```python
# Lire un fichier CSV et retourner un dictionnaire
# avec les en-têtes comme clés et les données comme listes
def read_csv_to_dict(filepath: str) -> dict:
    # ← Copilot génère l'implémentation ici
```

**3. Utiliser des types et annotations**

```python
from typing import Optional, List

def find_user_by_id(user_id: int, users: List[dict]) -> Optional[dict]:
    # Les annotations aident Copilot à générer du code plus précis
    ...
```

**4. TDD + Copilot : une combinaison puissante**

```python
# Étape 1 : Écrire le test D'ABORD
def test_password_strength():
    assert check_password_strength("abc") == "weak"
    assert check_password_strength("Abcd1234!") == "strong"
    assert check_password_strength("Abcd1234") == "medium"

# Étape 2 : Copilot génère l'implémentation qui passe les tests
def check_password_strength(password: str) -> str:
    # Copilot inférera la logique depuis les tests
    ...
```

### Copilot pour JetBrains

GitHub Copilot est disponible pour tous les IDEs JetBrains :
- **IntelliJ IDEA** (Java, Kotlin)
- **PyCharm** (Python)
- **WebStorm** (JavaScript/TypeScript)
- **GoLand**, **CLion**, **Rider**, etc.

Installation : `Settings → Plugins → Marketplace → "GitHub Copilot"` → Installer → Redémarrer

Les fonctionnalités sont similaires à VS Code, avec les raccourcis adaptés à chaque IDE JetBrains.

### Copilot Workspace (beta)

Copilot Workspace est une interface web (github.com/features/copilot/workspace) permettant de travailler sur des issues GitHub directement avec un agent IA. L'agent peut :
- Analyser une issue GitHub
- Proposer un plan de modification du code
- Générer les changements sur plusieurs fichiers
- Créer une pull request

> [!warning] En beta
> Copilot Workspace est encore en accès limité et les fonctionnalités évoluent rapidement. C'est la direction vers laquelle GitHub oriente ses efforts pour les agents de codage autonomes.

### Tarifs GitHub Copilot

```
┌──────────────────┬────────────────┬──────────────────────┐
│ Plan             │ Prix           │ Pour qui ?           │
├──────────────────┼────────────────┼──────────────────────┤
│ Individual       │ $10/mois       │ Développeurs solo    │
│                  │ ($100/an)      │                      │
├──────────────────┼────────────────┼──────────────────────┤
│ Business         │ $19/mois/siège │ Équipes, politiques  │
│                  │                │ de sécurité          │
├──────────────────┼────────────────┼──────────────────────┤
│ Enterprise       │ $39/mois/siège │ Grandes org.,        │
│                  │                │ Fine-tuning possible │
└──────────────────┴────────────────┴──────────────────────┘

Note : Copilot est GRATUIT pour les étudiants (GitHub Student Pack)
et les mainteneurs de projets open source populaires.
```

---

## 4. L'API OpenAI pour les développeurs

### Création du compte et API key

1. Aller sur [platform.openai.com](https://platform.openai.com)
2. Créer un compte ou se connecter
3. Aller dans `API Keys` → `Create new secret key`
4. Copier la clé (elle ne sera affichée qu'une fois)
5. Ajouter des crédits dans `Billing`

> [!warning] Sécurité des API keys
> Ne jamais commiter une API key dans Git. Utilise des variables d'environnement ou un fichier `.env` exclu de ton `.gitignore`. Une clé exposée peut entraîner des frais importants si elle est utilisée par des tiers.

```bash
# Bonne pratique : variable d'environnement
export OPENAI_API_KEY="sk-..."

# Ou dans un fichier .env (avec python-dotenv)
OPENAI_API_KEY=sk-...
```

### Installation du SDK Python

```bash
pip install openai
# Pour charger les variables d'environnement depuis .env :
pip install python-dotenv
```

### Premier appel API

```python
from openai import OpenAI
import os

# Le client cherche automatiquement OPENAI_API_KEY dans l'environnement
client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "Tu es un expert Python."},
        {"role": "user", "content": "Explique les décorateurs Python avec un exemple simple."}
    ],
    temperature=0.7,    # 0 = déterministe, 2 = très créatif
    max_tokens=1000     # Limite la longueur de la réponse
)

print(response.choices[0].message.content)

# Informations sur la consommation de tokens :
print(f"Tokens utilisés : {response.usage.total_tokens}")
```

### Pattern utile : code comme contexte

Un pattern très efficace pour le debugging ou la revue de code :

```python
def analyze_code_with_gpt(code: str, question: str) -> str:
    """Envoie du code à GPT-4o avec une question spécifique."""
    client = OpenAI()

    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "system",
                "content": "Tu es un expert en revue de code. Analyse le code fourni et réponds précisément à la question."
            },
            {
                "role": "user",
                "content": f"Voici le code à analyser :\n\n```python\n{code}\n```\n\nQuestion : {question}"
            }
        ],
        temperature=0.3  # Basse température pour des réponses précises
    )

    return response.choices[0].message.content

# Utilisation
with open("mon_script.py", "r") as f:
    code = f.read()

analyse = analyze_code_with_gpt(
    code=code,
    question="Y a-t-il des problèmes de performance ou des bugs potentiels ?"
)
print(analyse)
```

### Streaming pour les longues réponses

Le streaming permet d'afficher la réponse au fur et à mesure de sa génération, comme dans ChatGPT :

```python
from openai import OpenAI

client = OpenAI()

# Streaming : affiche les tokens au fur et à mesure
stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "Écris une fonction Python complète pour parser un fichier JSON imbriqué."}
    ],
    stream=True  # ← Activer le streaming
)

for chunk in stream:
    if chunk.choices[0].delta.content is not None:
        print(chunk.choices[0].delta.content, end="", flush=True)

print()  # Saut de ligne final
```

### Gestion des tokens avec tiktoken

Avant d'envoyer une requête, il est utile de calculer le nombre de tokens pour éviter de dépasser les limites et contrôler les coûts :

```python
import tiktoken

def count_tokens(text: str, model: str = "gpt-4o") -> int:
    """Compte le nombre de tokens dans un texte."""
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

def truncate_to_token_limit(text: str, max_tokens: int = 100000, model: str = "gpt-4o") -> str:
    """Tronque un texte pour respecter une limite de tokens."""
    encoding = tiktoken.encoding_for_model(model)
    tokens = encoding.encode(text)
    if len(tokens) <= max_tokens:
        return text
    return encoding.decode(tokens[:max_tokens])

# Exemple d'utilisation
code_content = open("gros_fichier.py").read()
tokens = count_tokens(code_content)
print(f"Ce fichier contient {tokens} tokens")

if tokens > 100000:
    print("Attention : fichier trop grand, troncature nécessaire")
    code_content = truncate_to_token_limit(code_content, max_tokens=90000)
```

---

## 5. ChatGPT pour les développeurs

### Utiliser ChatGPT efficacement pour le code

ChatGPT (via chat.openai.com ou l'application) reste un outil puissant même sans intégration IDE. La clé est de donner le bon contexte et d'utiliser les fonctionnalités avancées.

> [!tip] Analogie
> Pense à ChatGPT comme à un pair-programmer disponible 24h/24. Tu partages ton écran (le code), expliques le problème, et travaillez ensemble sur la solution. Comme un vrai pair-programming, la qualité du dialogue détermine la qualité du résultat.

### ChatGPT Projects

**Projects** permet d'organiser ses conversations par projet. Chaque projet peut avoir :
- Des instructions personnalisées (system prompt persistant)
- Des fichiers attachés (documentation, code source)
- Un historique de conversations liées

```
STRUCTURE RECOMMANDÉE DE PROJETS :

📁 [Mon App Django]
   ├── Instructions : "Tu es expert Django/Python. L'app utilise PostgreSQL
   │                   et Django REST Framework. Réponds en français."
   ├── Fichiers : models.py, settings.py, requirements.txt
   └── Conversations :
       ├── "Debug endpoint authentification"
       ├── "Optimisation requêtes ORM"
       └── "Migration vers Django 5.0"
```

### Mode Canvas

**Canvas** est un éditeur collaboratif intégré à ChatGPT. Il ouvre un panneau latéral contenant le code ou texte généré, avec des outils d'édition directe.

Fonctionnalités Canvas pour le code :
- Édition inline du code généré
- Boutons pour ajouter commentaires, logs, tests
- Changement de langage de programmation
- Révision de code (highlight des suggestions)
- Copier tout le code en un clic

```
┌─────────────────────────────────────────────────────────┐
│  INTERFACE CANVAS                                       │
│                                                         │
│  Chat            │  Canvas (éditeur)                   │
│  ───────────     │  ───────────────────────────────    │
│  "Ajoute la     │  def fibonacci(n: int) -> int:      │
│   gestion        │      """Calculate fibonacci."""     │
│   d'erreurs"     │      if n < 0:                     │
│                  │          raise ValueError(...)      │
│  ↕ dialogue      │      if n <= 1:                    │
│                  │          return n                   │
│                  │      return fib(n-1) + fib(n-2)    │
│                  │                                     │
│                  │  [Ajouter logs] [Tests] [Docs]      │
└─────────────────────────────────────────────────────────┘
```

### Interpréteur de code (Code Interpreter)

L'interpréteur de code exécute du Python directement dans ChatGPT dans un environnement sandbox. Extrêmement utile pour :

- **Analyser des données** : upload un CSV, demande des statistiques
- **Tracer des graphiques** : matplotlib/seaborn directement dans le chat
- **Tester des algorithmes** : vérifie que l'algo fonctionne avant de le copier
- **Convertir des fichiers** : PDF → texte, image → données, etc.

```python
# Exemple : ChatGPT avec Code Interpreter peut exécuter ceci en direct
import pandas as pd
import matplotlib.pyplot as plt

# Après upload d'un fichier CSV
df = pd.read_csv('ventes_2024.csv')
print(df.describe())
df.groupby('produit')['revenue'].sum().plot(kind='bar')
plt.title('Revenue par produit')
plt.savefig('analyse.png')  # ChatGPT affiche le graphique
```

### GPTs personnalisés pour le code

La plateforme GPTs permet de créer des assistants spécialisés :
- **Code Reviewer** : configuré pour suivre tes conventions de code
- **Documentation Writer** : génère de la doc dans ton format
- **SQL Expert** : spécialisé dans tes schémas de base de données

### Limites de ChatGPT par rapport à des agents comme Claude Code

> [!warning] Limites importantes
> - **Pas d'accès au système de fichiers** : ne peut pas lire/écrire tes fichiers locaux
> - **Pas d'exécution de commandes** : ne peut pas lancer des builds, des tests
> - **Pas d'agent autonome** : ne peut pas agir de façon enchaînée sur ton projet
> - **Contexte perdu** : chaque conversation repart de zéro (hors Projects)
> - **Contexte limité à 128k** : pas adapté à très grands codebases

---

## 6. Les modèles o1/o3 : raisonnement étendu

### La différence fondamentale avec GPT-4o

Les modèles GPT-4o génèrent des tokens immédiatement, mot par mot. Les modèles o1/o3 ont un mécanisme de **chain-of-thought interne** : avant de répondre, ils "réfléchissent" en générant des tokens de raisonnement internes (non visibles dans l'interface, mais facturés).

```
GPT-4o :
Question ──────────────────────────────→ Réponse
         (génération directe, rapide)

o3 :
Question → [Réflexion interne...] → [Vérification...] → Réponse
           (tokens de raisonnement, non visibles, lent mais précis)
```

> [!tip] Analogie
> GPT-4o est un développeur qui répond immédiatement à ta question d'une façon impressionnante. o3 est un développeur qui dit "donne-moi 5 minutes" et revient avec une solution profondément réfléchie et vérifiée. Pour une question simple, tu préfères le premier. Pour un problème d'architecture complexe, tu veux le second.

### Quand utiliser les modèles o1/o3

**Cas où o3 excelle :**

```python
# 1. Algorithmes complexes
"""
Implémente un algorithme de recherche de plus court chemin
avec contraintes de temps (time-windowed shortest path)
pour un réseau de transport multimodal.
"""

# 2. Debugging difficile
"""
Ce code de synchronisation multi-thread produit un deadlock
intermittent. Analyse les points d'attente possibles et
propose une solution sans deadlock possible.
"""

# 3. Architecture système
"""
Conçois une architecture event-driven pour un système
de traitement de 1M de transactions/jour avec garantie
de livraison exactement-une-fois (exactly-once delivery).
"""

# 4. Problèmes mathématiques et cryptographiques
"""
Implémente RSA from scratch avec padding OAEP,
explique chaque étape mathématiquement.
"""
```

**Cas à éviter avec o3 (trop cher/lent) :**

```python
# ❌ Trop simple pour o3, utilise GPT-4o-mini
"Comment convertir une liste en ensemble en Python ?"
"Quelle est la syntaxe d'une boucle for en JavaScript ?"
"Corrige la faute de frappe dans cette variable"
```

### Comparaison o3 vs GPT-4o sur un problème algo

```
PROBLÈME : Trouver toutes les permutations d'une chaîne
           en évitant les doublons, en O(n!) mais sans
           générer les doublons puis les filtrer.

GPT-4o (réponse rapide, ~3s) :
→ Donne une solution correcte avec backtracking + set de vus
→ Complexité O(n! * n) — génère des doublons et les évite

o3 (réponse lente, ~45s) :
→ Analyse la structure mathématique des permutations
→ Propose l'algorithme de Heap (aucun doublon, O(n!))
→ Explique pourquoi c'est optimal avec preuve formelle
→ Propose aussi une version itérative sans récursion
```

---

## 7. Forces et faiblesses d'OpenAI vs concurrents

### Points forts d'OpenAI

```
✓ GitHub Copilot : l'intégration IDE la plus mature et la plus utilisée
✓ o3 : meilleur modèle de raisonnement mathématique/algorithmique disponible
✓ Écosystème le plus mature : énorme communauté, docs, tutoriels, Stack Overflow
✓ API la plus stable et documentée : SDK dans 7+ langages
✓ Code Interpreter : exécution réelle de Python dans ChatGPT
✓ ChatGPT : interface la plus connue (facilite l'onboarding d'équipes)
✓ OpenAI Playground : outil de test d'API très pratique
```

### Points faibles d'OpenAI

```
✗ Plus cher que Claude Sonnet 3.7 pour des performances similaires
✗ Contexte : 128k tokens vs 200k (Claude) ou 2M (Gemini 1.5 Pro)
✗ Long code complexe : Claude 3.7 Sonnet surpasse GPT-4o sur les tâches
  de coding étendu (SWE-bench)
✗ Politique de données : OpenAI a eu des controverses sur l'utilisation
  des données d'entraînement (moins transparent qu'Anthropic)
✗ Pas d'agent de codage aussi intégré que Claude Code dans le terminal
✗ Tarifs enterprise moins compétitifs qu'Azure OpenAI Service (paradoxe)
```

### Tableau comparatif rapide

```
┌────────────────────┬────────────────┬────────────────┬────────────────┐
│ Critère            │ OpenAI         │ Anthropic      │ Google         │
├────────────────────┼────────────────┼────────────────┼────────────────┤
│ Meilleur IDE tool  │ GitHub Copilot │ Claude Code    │ Gemini in IDE  │
│ Raisonnement math  │ ★★★★★ (o3)    │ ★★★★☆          │ ★★★★☆          │
│ Long contexte      │ ★★★☆☆ (128k)  │ ★★★★☆ (200k)  │ ★★★★★ (2M)    │
│ Code complexe      │ ★★★★☆          │ ★★★★★          │ ★★★★☆          │
│ Prix               │ ★★★☆☆          │ ★★★★☆          │ ★★★★☆          │
│ Maturité API       │ ★★★★★          │ ★★★★☆          │ ★★★★☆          │
│ Code Interpreter   │ ★★★★★ (natif) │ ☆ (externe)   │ ★★★☆☆          │
└────────────────────┴────────────────┴────────────────┴────────────────┘
```

---

## 8. Cas d'usage recommandés

### Scénarios optimaux pour chaque outil

```
┌─────────────────────────────────────────────────────────────────┐
│           GUIDE DE CHOIX - OUTILS OPENAI POUR DEVS             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  COPILOT (VS Code/JetBrains)                                    │
│  ├── Complétion de code au quotidien                            │
│  ├── Équipes déjà sur GitHub (synergies Git/PR)                 │
│  ├── Génération de boilerplate et patterns répétitifs           │
│  └── Tests unitaires automatiques                               │
│                                                                  │
│  GPT-4o (API ou ChatGPT)                                        │
│  ├── Questions techniques rapides                               │
│  ├── Génération de code standard (CRUD, scripts)                │
│  ├── Revue de code et refactoring modéré                        │
│  └── Documentation et commentaires                              │
│                                                                  │
│  o3 / o3-mini (API ou ChatGPT Plus)                             │
│  ├── Algorithmes complexes (graphes, DP, cryptographie)         │
│  ├── Debugging de race conditions ou bugs subtils               │
│  ├── Architecture de systèmes distribués                        │
│  └── Problèmes mathématiques ou scientifiques                   │
│                                                                  │
│  ChatGPT + Canvas                                               │
│  ├── Sessions d'écriture collaborative de code                  │
│  ├── Refactoring interactif avec feedback immédiat              │
│  └── Prototypage rapide avec exécution (Code Interpreter)       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

> [!example] Exemple de workflow combiné
> **Projet réel : Implémenter un système de cache Redis**
>
> 1. **o3** : "Conçois l'architecture d'un cache distribué avec invalidation par tag"
>    → Obtient une architecture solide et réfléchie
>
> 2. **ChatGPT Canvas** : Itère sur l'implémentation Python avec GPT-4o
>    → Écrit le code de façon interactive
>
> 3. **GitHub Copilot** : Dans VS Code, complète les détails, génère les tests
>    → Finalise et valide l'implémentation
>
> **Résultat** : Meilleur que d'utiliser un seul outil, chacun dans son rôle optimal.

---

## Carte Mentale ASCII

```
                    ┌─────────────────────────┐
                    │     OPENAI POUR DEVS    │
                    └────────────┬────────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
   ┌──────▼──────┐      ┌────────▼───────┐    ┌────────▼───────┐
   │  MODÈLES    │      │ GITHUB COPILOT │    │  API OPENAI    │
   └──────┬──────┘      └────────┬───────┘    └────────┬───────┘
          │                      │                      │
    ┌─────┴─────┐          ┌─────┴─────┐         ┌─────┴─────┐
    │ GPT-4o    │          │ VS Code   │         │ SDK Python │
    │ GPT-4o-   │          │ JetBrains │         │ Streaming  │
    │  mini     │          │ Terminal  │         │ tiktoken   │
    │ o1 / o3   │          │ Workspace │         │ API key    │
    │ o3-mini   │          └─────┬─────┘         └─────┬─────┘
    │ o4-mini   │                │                      │
    └─────┬─────┘         ┌──────┴──────┐        ┌─────┴──────┐
          │               │ Raccourcis  │        │  Patterns  │
    ┌─────┴──────┐        │ Tab Accept  │        │ Code ctx   │
    │ Choix selon│        │ Alt+] Suiv. │        │ Analyse    │
    │ complexité │        │ Ctrl+I Chat │        │ Streaming  │
    └────────────┘        └─────────────┘        └────────────┘

   ┌─────────────────────────────────────────────────────────┐
   │                    CHATGPT POUR DEVS                    │
   ├───────────┬──────────────┬──────────────┬───────────────┤
   │ Projects  │    Canvas    │   Code       │     GPTs      │
   │ (contexte │ (éditeur     │ Interpreter  │ (assistants   │
   │  persistant│  collab.)   │ (exec Python)│  spécialisés) │
   └───────────┴──────────────┴──────────────┴───────────────┘

   MODÈLES DE RAISONNEMENT (o1 / o3)
   ┌────────────────────────────────────────────────────────┐
   │                                                        │
   │  GPT-4o  : Question → Réponse (rapide)                │
   │  o3      : Question → [Réflexion...] → Réponse (lent) │
   │                                                        │
   │  Utiliser o3 pour : Algo complexe, Math, Architecture  │
   │  Éviter o3 pour   : Tâches simples (coût élevé)       │
   │                                                        │
   └────────────────────────────────────────────────────────┘
```

---

## Exercices pratiques

### Exercice 1 — Premier appel API avec analyse de code

**Objectif** : Créer un script Python qui analyse automatiquement un fichier de code source et produit un rapport de qualité.

**Étapes** :
1. Installer le SDK OpenAI : `pip install openai python-dotenv`
2. Créer un fichier `.env` avec `OPENAI_API_KEY=sk-...`
3. Écrire une fonction `analyze_code_quality(filepath: str) -> dict` qui :
   - Lit le fichier Python passé en argument
   - Envoie son contenu à GPT-4o avec un prompt demandant : bugs potentiels, code smells, suggestions d'amélioration
   - Retourne un dictionnaire avec les clés `bugs`, `smells`, `suggestions`
4. Tester sur un de tes fichiers Python existants
5. **Bonus** : Ajouter du streaming pour voir la réponse s'afficher progressivement

**Critère de succès** : Le script produit un rapport structuré sans erreur d'API.

---

### Exercice 2 — Optimiser son workflow avec GitHub Copilot

**Objectif** : Pratiquer les patterns qui maximisent la qualité des suggestions Copilot.

**Étapes** :
1. Installe GitHub Copilot dans VS Code (si pas déjà fait)
2. Crée un fichier `data_processor.py`
3. Écris **uniquement** les éléments suivants (pas l'implémentation) :
   ```python
   # Module de traitement de données CSV avec validation et nettoyage
   from typing import List, Dict, Optional
   import csv
   from pathlib import Path

   def load_and_validate_csv(filepath: Path, required_columns: List[str]) -> Optional[List[Dict]]:
       """
       Charge un fichier CSV, valide que les colonnes requises existent,
       supprime les lignes avec des valeurs manquantes, et retourne les données.
       
       Args:
           filepath: Chemin vers le fichier CSV
           required_columns: Liste des colonnes obligatoires
           
       Returns:
           Liste de dictionnaires ou None si le fichier est invalide
       """
   ```
4. Observe les suggestions de Copilot et accepte/modifie selon ta logique
5. Génère les tests avec `/tests` dans Copilot Chat
6. **Bonus** : Compare avec ce que GPT-4o génère pour la même spécification

---

### Exercice 3 — Benchmark o3-mini vs GPT-4o sur un problème algorithmique

**Objectif** : Comprendre empiriquement la différence entre les modèles de raisonnement et GPT-4o.

**Étapes** :
1. Choisir un problème algorithmique non trivial, par exemple :
   - "Implémenter l'algorithme de Dijkstra avec un tas de Fibonacci"
   - "Résoudre le problème du sac à dos 0/1 avec mémoïsation optimisée en espace"
   - "Implémenter un LRU cache thread-safe en Python"

2. Poser la même question à :
   - GPT-4o (via API ou ChatGPT)
   - o3-mini (via ChatGPT Plus ou API)

3. Évaluer les réponses sur :
   - Correction de l'implémentation (tester le code)
   - Qualité des explications
   - Gestion des edge cases
   - Temps de réponse
   - (Si via API) Nombre de tokens utilisés

4. Documenter tes conclusions : pour quel type de problème le surcoût d'o3-mini est-il justifié ?

---

## Liens et ressources

### Wikilinks

- [[01 - Panorama des IA pour Développeurs]] — Vue d'ensemble de tous les outils IA disponibles pour les développeurs
- [[06 - Gemini Mistral et Alternatives Cloud]] — Les alternatives à OpenAI : Google Gemini, Mistral, et les modèles open-source
- [[07 - Integrations IDE et Extensions]] — Comparatif complet des intégrations IDE : Copilot, Cursor, Codeium, etc.
- [[12 - Strategies Multi-Modeles et Workflows]] — Comment combiner plusieurs modèles IA dans un workflow de développement optimal

### Ressources externes

- Documentation officielle API : `platform.openai.com/docs`
- GitHub Copilot docs : `docs.github.com/copilot`
- OpenAI Cookbook (exemples pratiques) : `cookbook.openai.com`
- Playground interactif : `platform.openai.com/playground`
- Tarifs actuels : `platform.openai.com/pricing`
