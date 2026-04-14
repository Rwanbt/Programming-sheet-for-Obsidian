# 05 - ChatGPT, Codex et GitHub Copilot

> [!info] Mis à jour avril 2026

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
│                    │  GPT-5.4, o3,  │                      │
│                    │  o4-mini, etc. │                      │
│                    └────────────────┘                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Impact sur l'industrie

L'arrivée de GitHub Copilot (basé initialement sur Codex, un modèle dérivé de GPT-3) en 2021 a marqué un tournant : pour la première fois, l'IA devenait un compagnon de codage en temps réel. Selon GitHub, les développeurs utilisant Copilot produisent du code **55% plus vite** sur certaines tâches. Depuis, l'ensemble de l'industrie s'est aligné : JetBrains AI Assistant, Cursor, Codeium, tous s'inspirent du modèle Copilot.

---

## 2. Les modèles OpenAI en 2025-2026

OpenAI maintient une famille de modèles aux caractéristiques très différentes. Comprendre lequel utiliser est crucial pour optimiser coût, vitesse et qualité.

### GPT-5.4 — Le modèle flagship

**GPT-5.4** (sorti mars 2026) est le modèle phare d'OpenAI. Il est multimodal : il accepte du texte, des images, de l'audio, et intègre nativement le computer use et le tool search. Il est disponible en 5 variantes : Standard, Thinking, Pro, Mini et Nano.

- Fenêtre de contexte : **272 000 tokens** (standard), jusqu'à 1M via Codex
- Vitesse : rapide (génération streaming fluide)
- Qualité code : excellente — 57,7 % SWE-bench Pro
- Prix (API) : $2.50 / 1M tokens input, $15 / 1M tokens output

> [!tip] Analogie
> GPT-5.4 est comme un développeur senior polyvalent : il connaît tous les langages, répond vite, gère les grands codebases et peut même utiliser des outils de façon autonome. C'est votre collègue idéal pour 90% des tâches quotidiennes.

> [!warning] GPT-4o remplacé
> GPT-4o est désormais un modèle périmé, remplacé par GPT-5.4 depuis mars 2026. Il n'est plus recommandé pour de nouveaux projets.

### GPT-5.4 Nano — Le modèle ultra-économique

Variante la plus légère de la famille GPT-5.4, optimisée pour le rapport vitesse/coût.

- Contexte : **400 000 tokens**
- Vitesse : ultra-rapide
- Qualité : acceptable pour tâches simples, génération de contenu, classification
- Prix : $0.20 / 1M tokens input, $1.25 / 1M tokens output

### GPT-5.4 Mini — Le milieu de gamme économique

Variante intermédiaire de la famille GPT-5.4, bon équilibre qualité/coût.

- Contexte : **128 000 tokens**
- Vitesse : rapide
- Qualité : bonne pour la plupart des tâches de développement courantes
- Prix : ~$0.40 / 1M tokens input, ~$1.60 / 1M tokens output

### GPT-5.4 Pro — Le premium

Variante haut de gamme de la famille GPT-5.4, pour les cas d'usage les plus exigeants.

- Contexte : 272 000 tokens
- Qualité : maximale sur toute la gamme GPT-5.4
- Prix : plus élevé que GPT-5.4 Standard

### o3 / o4-mini — Les modèles de raisonnement profond

**o3** et **o4-mini** conservent une approche radicalement différente : le modèle "réfléchit" avant de répondre via une chaîne de pensée interne (chain-of-thought). Ils prennent plus de temps, mais produisent des réponses bien plus précises sur les problèmes complexes.

- o3 — Contexte : ~200 000 tokens ; Prix : ~$2 / 1M input, ~$8 / 1M output
- o4-mini — Multimodal (vision) + raisonnement dans un package économique
- Vitesse : lente (tokens de raisonnement internes facturés)
- Qualité : excellence sur les maths, algorithmes et architecture système

---

### Tableau comparatif des modèles

```
┌──────────────────┬────────────┬──────────────┬──────────┬────────────┬─────────────┐
│  Modèle          │ Vitesse    │ Qualité code │ Contexte │ Prix input │ Prix output │
├──────────────────┼────────────┼──────────────┼──────────┼────────────┼─────────────┤
│ GPT-5.4          │ Rapide     │ ★★★★★        │ 272k     │ $2.50/1M   │ $15/1M      │
│ GPT-5.4 Mini     │ Rapide     │ ★★★★☆        │ 128k     │ ~$0.40/1M  │ ~$1.60/1M   │
│ GPT-5.4 Nano     │ Ultra-rap. │ ★★★☆☆        │ 400k     │ $0.20/1M   │ $1.25/1M    │
│ o3 (raisonnement)│ Lente      │ ★★★★★ (maths)│ 200k     │ ~$2/1M     │ ~$8/1M      │
│ o4-mini          │ Moyenne    │ ★★★★☆ (+vis.)│ 128k     │ $1.1/1M    │ $4.4/1M     │
└──────────────────┴────────────┴──────────────┴──────────┴────────────┴─────────────┘
  Note : Prix approximatifs, vérifier platform.openai.com/pricing pour les tarifs actuels
```

### Quand utiliser quel modèle ?

```
ARBRE DE DÉCISION — CHOIX DU MODÈLE

Est-ce une tâche simple ?
(autocomplétion, reformulation, résumé court)
       │
       ├── OUI → GPT-5.4 Nano (ultra-économique et ultra-rapide)
       │
       └── NON → Est-ce un problème algorithmique/mathématique difficile ?
                        │
                        ├── OUI → Besoin du meilleur résultat ?
                        │              ├── OUI → o3
                        │              └── NON → o4-mini
                        │
                        └── NON → Tâche générale de qualité ?
                                       ├── OUI → GPT-5.4 Standard
                                       └── Budget serré ? → GPT-5.4 Mini
```

---

## 3. GitHub Copilot

### Qu'est-ce que GitHub Copilot ?

GitHub Copilot est un assistant de programmation IA développé par GitHub (Microsoft) en partenariat avec OpenAI. Il s'intègre directement dans l'éditeur de code et suggère du code en temps réel. Lancé en 2021, il est devenu en quelques années l'outil d'IA le plus utilisé par les développeurs professionnels. Depuis 2026, il utilise **GPT-5.4 par défaut** pour toutes les suggestions et le chat.

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
│              │  Modèles OpenAI        │                   │
│              │  (GPT-5.4 par défaut)  │                   │
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

### Copilot Workspace

Copilot Workspace est une interface web (github.com/features/copilot/workspace) permettant de travailler sur des issues GitHub directement avec un agent IA. L'agent peut :
- Analyser une issue GitHub
- Proposer un plan de modification du code
- Générer les changements sur plusieurs fichiers
- Créer une pull request

> [!info] Disponibilité générale
> Copilot Workspace est désormais en disponibilité générale (GA) depuis 2026. C'est la solution officielle GitHub pour les agents de codage autonomes sur les dépôts.

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
    model="gpt-5.4",
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
    """Envoie du code à GPT-5.4 avec une question spécifique."""
    client = OpenAI()

    response = client.chat.completions.create(
        model="gpt-5.4",
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
    model="gpt-5.4",
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

def count_tokens(text: str, model: str = "gpt-5.4") -> int:
    """Compte le nombre de tokens dans un texte."""
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

def truncate_to_token_limit(text: str, max_tokens: int = 100000, model: str = "gpt-5.4") -> str:
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
> - **Contexte limité** : GPT-5.4 Mini à 128k, GPT-5.4 Standard à 272k — pas toujours adapté aux très grands codebases

---

## 6. Les modèles o3/o4 : raisonnement étendu

### La différence fondamentale avec GPT-5.4

Les modèles GPT-5.4 génèrent des tokens immédiatement, mot par mot. Les modèles o3/o4 ont un mécanisme de **chain-of-thought interne** : avant de répondre, ils "réfléchissent" en générant des tokens de raisonnement internes (non visibles dans l'interface, mais facturés).

```
GPT-5.4 :
Question ──────────────────────────────→ Réponse
         (génération directe, rapide)

o3 :
Question → [Réflexion interne...] → [Vérification...] → Réponse
           (tokens de raisonnement, non visibles, lent mais précis)
```

> [!tip] Analogie
> GPT-5.4 est un développeur qui répond immédiatement à ta question d'une façon impressionnante. o3 est un développeur qui dit "donne-moi 5 minutes" et revient avec une solution profondément réfléchie et vérifiée. Pour une question simple, tu préfères le premier. Pour un problème d'architecture complexe, tu veux le second.

### Quand utiliser les modèles o3/o4

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
# ❌ Trop simple pour o3, utilise GPT-5.4 Nano
"Comment convertir une liste en ensemble en Python ?"
"Quelle est la syntaxe d'une boucle for en JavaScript ?"
"Corrige la faute de frappe dans cette variable"
```

### Comparaison o3 vs GPT-5.4 sur un problème algo

```
PROBLÈME : Trouver toutes les permutations d'une chaîne
           en évitant les doublons, en O(n!) mais sans
           générer les doublons puis les filtrer.

GPT-5.4 (réponse rapide, ~3s) :
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
✓ GPT-5.4 : modèle flagship polyvalent avec 272k de contexte et computer use natif
✓ o3 : meilleur modèle de raisonnement mathématique/algorithmique disponible
✓ Écosystème le plus mature : énorme communauté, docs, tutoriels, Stack Overflow
✓ API la plus stable et documentée : SDK dans 7+ langages
✓ Code Interpreter : exécution réelle de Python dans ChatGPT
✓ ChatGPT : interface la plus connue (facilite l'onboarding d'équipes)
✓ OpenAI Playground : outil de test d'API très pratique
✓ GPT-5.4 Nano : option ultra-économique à $0.20/1M tokens avec 400k de contexte
```

### Points faibles d'OpenAI

```
✗ GPT-5.4 output reste à $15/1M — comparable à Claude Sonnet mais plus cher en output
✗ Contexte GPT-5.4 Mini : 128k tokens vs 200k (Claude) — inférieur sur certains tiers
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
│ Long contexte      │ ★★★★☆ (272k)  │ ★★★★☆ (200k)  │ ★★★★★ (2M)    │
│ Code complexe      │ ★★★★★ (57.7%) │ ★★★★★          │ ★★★★☆          │
│ Prix               │ ★★★★☆          │ ★★★★☆          │ ★★★★☆          │
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
│  GPT-5.4 (API ou ChatGPT)                                       │
│  ├── Questions techniques rapides et dev quotidien              │
│  ├── Génération de code standard (CRUD, scripts, refactoring)   │
│  ├── Revue de code et refactoring modéré                        │
│  └── Documentation et commentaires                              │
│                                                                  │
│  GPT-5.4 Nano (API ou ChatGPT)                                  │
│  ├── Questions simples et complétion rapide                     │
│  ├── Budget serré — coût minimal à $0.20/1M tokens              │
│  └── Traitement en masse de petites tâches                      │
│                                                                  │
│  o3 / o4-mini (API ou ChatGPT Plus)                             │
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
> 2. **ChatGPT Canvas** : Itère sur l'implémentation Python avec GPT-5.4
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
    │ GPT-5.4   │          │ VS Code   │         │ SDK Python │
    │ GPT-5.4   │          │ JetBrains │         │ Streaming  │
    │  Mini/Nano│          │ Terminal  │         │ tiktoken   │
    │ o3 / o4   │          │ Workspace │         │ API key    │
    │  -mini    │          └─────┬─────┘         └─────┬─────┘
    │           │                │                      │
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

   MODÈLES DE RAISONNEMENT (o3 / o4-mini)
   ┌────────────────────────────────────────────────────────┐
   │                                                        │
   │  GPT-5.4 : Question → Réponse (rapide)                │
   │  o3      : Question → [Réflexion...] → Réponse (lent) │
   │                                                        │
   │  Utiliser o3 pour : Algo complexe, Math, Architecture  │
   │  Éviter o3 pour   : Tâches simples → GPT-5.4 Nano     │
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
   - Envoie son contenu à GPT-5.4 avec un prompt demandant : bugs potentiels, code smells, suggestions d'amélioration
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
6. **Bonus** : Compare avec ce que GPT-5.4 génère pour la même spécification

---

### Exercice 3 — Benchmark o3 vs GPT-5.4 sur un problème algorithmique

**Objectif** : Comprendre empiriquement la différence entre les modèles de raisonnement et GPT-5.4.

**Étapes** :
1. Choisir un problème algorithmique non trivial, par exemple :
   - "Implémenter l'algorithme de Dijkstra avec un tas de Fibonacci"
   - "Résoudre le problème du sac à dos 0/1 avec mémoïsation optimisée en espace"
   - "Implémenter un LRU cache thread-safe en Python"

2. Poser la même question à :
   - GPT-5.4 (via API ou ChatGPT)
   - o3 (via ChatGPT Plus ou API)

3. Évaluer les réponses sur :
   - Correction de l'implémentation (tester le code)
   - Qualité des explications
   - Gestion des edge cases
   - Temps de réponse
   - (Si via API) Nombre de tokens utilisés

4. Documenter tes conclusions : pour quel type de problème le surcoût d'o3 est-il justifié ?

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
