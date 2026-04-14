# IA Assistants de Code

## La revolution IA dans le developpement

L'intelligence artificielle a transforme la maniere dont on ecrit du code. En quelques annees, on est passe de l'autocompletion basique a des assistants capables de comprendre, generer et expliquer du code complexe.

> [!tip] Analogie
> Imagine l'evolution des **outils de construction** :
> - **Avant l'IA** : tu construis une maison brique par brique a la main (ecrire chaque ligne)
> - **Autocompletion classique** : tu as un assistant qui te tend la brique suivante (IntelliSense)
> - **IA completion (Copilot)** : tu as un ouvrier qui construit le mur entier si tu lui montres le plan
> - **IA chat (ChatGPT/Claude)** : tu as un architecte a qui tu decris la maison et il te donne les plans
> - **IA agents (Claude Code/Cursor)** : tu as une equipe complete qui construit la maison pendant que tu supervises
>
> On ne remplace pas le developpeur : on lui donne des **outils exponentiellement plus puissants**.

### Chronologie

```
2020          2021           2022            2023           2024-2025
  |             |              |               |               |
  v             v              v               v               v
GPT-3       GitHub         ChatGPT         GPT-4           Claude 3.5
(OpenAI)    Copilot        (nov 2022)      Claude 2        Sonnet
            (juin 2021)                    Gemini          Claude Code
                                           Copilot X       Cursor Agent
                                                           Windsurf

  Modeles     Completion     Chat pour       IA avancee     Agents
  de langage  en temps reel  le code         + reasoning    autonomes
```

---

## Les types d'outils IA pour le code

### Vue d'ensemble

```
┌──────────────────────────────────────────────────────────────────────┐
│                   OUTILS IA POUR LE CODE                             │
│                                                                      │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐   │
│  │   COMPLETION      │  │      CHAT         │  │     AGENTS       │   │
│  │                   │  │                   │  │                  │   │
│  │  Suggestions en   │  │  Conversation     │  │  Execution       │   │
│  │  temps reel dans  │  │  interactive sur  │  │  autonome de     │   │
│  │  l'editeur        │  │  le code          │  │  taches          │   │
│  │                   │  │                   │  │                  │   │
│  │  - Copilot        │  │  - ChatGPT        │  │  - Claude Code   │   │
│  │  - Codeium        │  │  - Claude         │  │  - Cursor Agent  │   │
│  │  - Supermaven     │  │  - Gemini         │  │  - Windsurf      │   │
│  │  - TabNine        │  │  - Copilot Chat   │  │  - Devin         │   │
│  │                   │  │                   │  │                  │   │
│  │  Niveau :         │  │  Niveau :         │  │  Niveau :        │   │
│  │  Ligne/Bloc       │  │  Question/Reponse │  │  Projet entier   │   │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘   │
│                                                                      │
│  Autonomie croissante ─────────────────────────────────────────>     │
└──────────────────────────────────────────────────────────────────────┘
```

| Type | Comment ca marche | Quand l'utiliser |
|---|---|---|
| **Completion** | Suggestions automatiques dans l'IDE | Ecriture quotidienne, boilerplate, patterns repetitifs |
| **Chat** | Tu poses des questions, l'IA repond | Debugging, explication, design, code review |
| **Agent** | L'IA execute des taches complexes | Refactoring, creation de features, multi-fichiers |

---

## GitHub Copilot

### Comment ca marche

**GitHub Copilot** utilise un modele d'IA entraine sur du code public pour suggerer des completions en temps reel dans ton editeur.

```
┌──────────────────────────────────────────────────────┐
│  TON EDITEUR (VS Code)                                │
│                                                       │
│  def fibonacci(n):                                    │
│      """Return the nth Fibonacci number."""            │
│      |                                                │
│      ┌─────────────────────────────────────────────┐ │
│      │ SUGGESTION COPILOT (grise) :                 │ │
│      │ if n <= 1:                                    │ │
│      │     return n                                  │ │
│      │ return fibonacci(n - 1) + fibonacci(n - 2)   │ │
│      │                                               │ │
│      │ [Tab] pour accepter  [Esc] pour ignorer      │ │
│      └─────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

### Fonctionnalites

| Fonctionnalite | Description |
|---|---|
| **Inline Suggestions** | Completions de code en temps reel (Tab pour accepter) |
| **Copilot Chat** | Conversation dans l'IDE pour poser des questions |
| **Copilot Workspace** | Planification et execution de taches sur tout le projet |
| **Ghost Text** | Suggestions grises qui apparaissent au fil de la frappe |
| **Multi-ligne** | Peut generer des fonctions entieres |

### Setup dans VS Code

```
1. Installer l'extension "GitHub Copilot"
2. Installer l'extension "GitHub Copilot Chat"
3. Se connecter avec son compte GitHub
4. Verifier : Settings → Extensions → GitHub Copilot → Enable
```

### Utiliser Copilot efficacement

> [!info] Les commentaires guident Copilot
> Copilot utilise le **contexte** (noms de fichiers, commentaires, code existant) pour generer ses suggestions. Plus ton contexte est clair, meilleures sont les suggestions.

```python
# BON : commentaire descriptif → bonne suggestion
# Function that validates an email address using regex
def validate_email(email):
    # Copilot va generer une regex pertinente

# MAUVAIS : pas de contexte → suggestion generique
def f(x):
    # Copilot ne sait pas quoi faire
```

#### Techniques pour de meilleures suggestions

```python
# 1. Noms descriptifs
def calculate_monthly_payment(principal, annual_rate, years):
    # Copilot comprend le contexte financier

# 2. Docstrings detaillees
def merge_sort(arr: list[int]) -> list[int]:
    """
    Sort an array using the merge sort algorithm.
    Time complexity: O(n log n)
    Space complexity: O(n)
    """
    # Copilot genere l'implementation correcte

# 3. Exemples dans les commentaires
# Convert "2024-01-15" to "15 janvier 2024"
def format_date_french(date_string):
    # Copilot comprend le format attendu

# 4. Pattern etabli - si tu as deja ecrit des fonctions similaires,
#    Copilot va suivre le meme pattern
```

---

## ChatGPT pour le code

### Strategies de prompting

**ChatGPT** excelle quand on sait bien formuler ses demandes.

#### Le systeme de prompting

```
┌─────────────────────────────────────────────────────┐
│                    PROMPT                             │
│                                                      │
│  [ROLE]     Tu es un developpeur senior Python...    │
│  [CONTEXTE] Je travaille sur une API Flask qui...    │
│  [TACHE]    Ecris une fonction qui...                │
│  [FORMAT]   Avec docstring et type hints...          │
│  [EXEMPLE]  Voici un exemple de ce que j'attends...  │
│                                                      │
│  ──────────────────────────────────────────────>     │
│                                                      │
│  [REPONSE]  Code + explication + tests               │
└─────────────────────────────────────────────────────┘
```

#### Exemples de prompts efficaces

```
Prompt VAGUE (mauvais) :
"Fais-moi une fonction de tri"

Prompt PRECIS (bon) :
"Ecris une fonction Python qui trie une liste de dictionnaires
par une cle donnee. La fonction doit :
- Accepter une liste de dicts et un nom de cle (string)
- Supporter le tri ascendant et descendant (parametre reverse)
- Gerer le cas ou la cle n'existe pas dans certains dicts
- Avoir des type hints et une docstring
- Exemple : sort_by_key([{'name': 'Alice', 'age': 30}, {'name': 'Bob', 'age': 25}], 'age')
  → [{'name': 'Bob', 'age': 25}, {'name': 'Alice', 'age': 30}]"
```

#### System prompts utiles

```
Pour du code review :
"Tu es un code reviewer senior avec 15 ans d'experience. Tu es
rigoureux mais pedagogique. Pour chaque probleme identifie, tu
expliques POURQUOI c'est un probleme et tu proposes une solution
concrete."

Pour du debugging :
"Tu es un expert en debugging Python. Quand je te donne du code
et un message d'erreur, tu analyses methodiquement : 1) ce que
l'erreur signifie, 2) pourquoi elle se produit, 3) comment la
corriger, 4) comment eviter ce type d'erreur a l'avenir."
```

---

## Claude pour le code

### Forces de Claude

| Force | Explication |
|---|---|
| **Contexte long** | Peut analyser des fichiers entiers ou meme des projets complets (200K+ tokens) |
| **Raisonnement** | Excellent pour comprendre la logique complexe et debugger |
| **Suivi d'instructions** | Respecte les formats et contraintes demandees |
| **Honnetete** | Dit quand il n'est pas sur ou ne sait pas |

### Claude Code (CLI)

**Claude Code** est l'outil CLI officiel d'Anthropic qui permet a Claude d'interagir directement avec ton code, ton terminal et ton systeme de fichiers.

```bash
# Installation
npm install -g @anthropic-ai/claude-code

# Lancer dans un projet
cd mon-projet
claude

# Exemples de commandes naturelles
> "Explique-moi l'architecture de ce projet"
> "Trouve et corrige le bug dans la fonction parse_data"
> "Ajoute des tests unitaires pour le module auth"
> "Refactorise ce fichier pour suivre le pattern Strategy"
```

```
┌─────────────────────────────────────────────────────────┐
│  CLAUDE CODE                                             │
│                                                          │
│  Capacites :                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ Lire des     │  │ Modifier    │  │ Executer    │     │
│  │ fichiers     │  │ des fichiers│  │ des         │     │
│  │              │  │             │  │ commandes   │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ Chercher    │  │ Git commits │  │ Creer des   │     │
│  │ du code     │  │ et PRs      │  │ fichiers    │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│                                                          │
│  Mode : AGENT (autonome, multi-etapes, multi-fichiers)  │
└─────────────────────────────────────────────────────────┘
```

> [!info] Agent vs Chat
> Claude Code est un **agent** : il peut enchainer plusieurs actions (lire un fichier, comprendre le contexte, modifier le code, lancer les tests) de maniere autonome. Un simple chatbot ne peut que repondre a des questions.

---

## Prompt Engineering pour le code

### Les principes fondamentaux

#### 1. Etre specifique

```
MAUVAIS :
"Corrige mon code"

BON :
"Ma fonction calculate_tax() retourne 0 au lieu de 15.0 quand
je l'appelle avec calculate_tax(100, 0.15). Voici le code :
[code]. Voici l'erreur que j'observe : [erreur]. Le comportement
attendu est : [description]."
```

#### 2. Fournir le contexte

```
MAUVAIS :
"Ecris un endpoint"

BON :
"Ecris un endpoint Flask POST /api/users qui :
- Recoit un JSON avec name (string, requis) et email (string, requis)
- Valide les donnees
- Sauvegarde dans une base SQLite
- Retourne 201 avec l'utilisateur cree
- Retourne 400 si les donnees sont invalides
Le projet utilise Flask 3.0 et SQLAlchemy 2.0."
```

#### 3. Donner des exemples (few-shot prompting)

```
"Convertis ces fonctions synchrones en asynchrones.

Exemple :
Avant : def get_user(id): return db.query(User).get(id)
Apres : async def get_user(id): return await db.query(User).get(id)

Maintenant convertis :
def get_all_posts(): return db.query(Post).all()
def create_comment(text, post_id): return db.add(Comment(text=text, post_id=post_id))"
```

#### 4. Raffinement iteratif

```
Prompt 1 : "Ecris une classe Python pour gerer un cache LRU"
Prompt 2 : "Ajoute le support du TTL (time to live) pour chaque entree"
Prompt 3 : "Rends-la thread-safe avec threading.Lock"
Prompt 4 : "Ajoute des tests unitaires avec pytest"
```

#### 5. Role prompting

```
"Tu es un expert en securite applicative. Revois ce code
d'authentification et identifie toutes les vulnerabilites
potentielles (injection SQL, XSS, CSRF, etc.). Pour chaque
vulnerabilite, donne un score de severite (1-5) et une correction."
```

---

## Exemples de prompts efficaces

### Debugging

```
"J'ai cette erreur quand j'execute mon programme :

```
TypeError: unsupported operand type(s) for +: 'int' and 'str'
File "main.py", line 42, in calculate_total
```

Voici la fonction concernee :
[coller le code]

Explique-moi :
1. Pourquoi cette erreur se produit
2. Comment la corriger
3. Comment eviter ce type d'erreur a l'avenir"
```

### Explication de code

```
"Explique-moi ce code ligne par ligne. Je suis etudiant
en programmation, utilise des termes simples et des analogies.

[coller le code]

Pour chaque section, explique :
- CE QUE ca fait
- POURQUOI c'est fait comme ca
- S'il y a un meilleur moyen de le faire"
```

### Generation de tests

```
"Genere des tests unitaires pytest pour cette classe :

[coller le code]

Les tests doivent couvrir :
- Les cas normaux (happy path)
- Les cas limites (valeurs vides, None, negatifs)
- Les cas d'erreur (exceptions attendues)
- Utilise des fixtures pytest si necessaire
- Vise une couverture de 100%"
```

### Code review

```
"Fais une code review de cette Pull Request.

[coller le code]

Evalue sur ces criteres :
- [ ] Lisibilite et nommage
- [ ] Gestion des erreurs
- [ ] Performance
- [ ] Securite
- [ ] Tests
- [ ] Documentation

Pour chaque probleme, donne :
- Severite (critique/majeur/mineur/suggestion)
- Explication du probleme
- Code corrige"
```

### Refactoring

```
"Refactorise cette fonction pour suivre les principes SOLID.
Elle fait trop de choses : validation, calcul et sauvegarde.

[coller le code]

Decompose-la en fonctions separees avec :
- Single Responsibility
- Type hints
- Docstrings
- Gestion d'erreurs propre"
```

### Documentation

```
"Genere la documentation complete pour cette API :

[coller le code]

Inclus :
- Docstring pour chaque fonction/methode
- Description des parametres et retours
- Exemples d'utilisation
- Notes sur les cas limites"
```

---

## Anti-patterns : ce qu'il ne faut PAS faire

> [!warning] Les pieges de l'IA generative

### 1. Copier-coller aveugle

```
MAUVAIS :
  1. Demander a l'IA de generer du code
  2. Copier-coller directement
  3. Pousser en production
  4. ???
  5. Bug en production a 3h du matin

BON :
  1. Demander a l'IA de generer du code
  2. LIRE et COMPRENDRE chaque ligne
  3. Tester localement
  4. Adapter au contexte du projet
  5. Code review
  6. Tests automatises
  7. Deployer avec confiance
```

### 2. Ne pas comprendre le code genere

```python
# L'IA genere ce code. Tu le copies. Tu ne sais pas ce que ca fait.
import pickle
data = pickle.loads(user_input)  # VULNERABILITE CRITIQUE
# pickle.loads sur des donnees utilisateur = execution de code arbitraire
```

> [!warning] Regle d'or
> Si tu ne peux pas expliquer chaque ligne du code genere par l'IA, **tu ne dois pas l'utiliser**. L'IA est un outil, pas un remplacant de la comprehension.

### 3. Risques de securite

```python
# L'IA peut generer du code vulnerable :

# Injection SQL
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")  # DANGER

# Mot de passe en dur
API_KEY = "sk-1234567890abcdef"  # DANGER

# Pas de validation d'entree
def process_file(filename):
    with open(filename) as f:   # Path traversal possible
        return f.read()
```

### 4. APIs hallucinees

```python
# L'IA peut inventer des fonctions qui n'existent pas !

import pandas as pd
df.auto_clean()        # Cette methode N'EXISTE PAS
df.smart_merge(other)  # Cette methode N'EXISTE PAS non plus

# Toujours verifier dans la documentation officielle
```

---

## Limitations de l'IA

| Limitation | Detail |
|---|---|
| **Coupure d'entrainement** | L'IA ne connait pas les libraries/APIs sorties apres sa date de coupure |
| **Hallucinations** | Peut inventer des fonctions, des APIs, des syntaxes qui n'existent pas |
| **Fenetre de contexte** | Ne peut pas "voir" tout un projet de 100 000 lignes d'un coup |
| **Pas de comprehension profonde** | Genere du code par pattern matching, pas par raisonnement pur |
| **Biais d'entrainement** | Peut reproduire des patterns obsoletes ou mauvaises pratiques vues dans les donnees |
| **Pas d'execution** | Ne peut pas tester le code qu'il genere (sauf les agents) |
| **Pas de contexte metier** | Ne connait pas les regles specifiques de ton entreprise/projet |

> [!info] Fenetre de contexte
> La **fenetre de contexte** est la quantite de texte que l'IA peut "voir" en une fois. Si ton projet depasse cette limite, l'IA ne voit qu'une partie et peut faire des suggestions incoherentes.
>
> | Modele | Contexte | Equivalent |
> |---|---|---|
> | GPT-4o | ~128K tokens | ~100 pages |
> | Claude 3.5 Sonnet | ~200K tokens | ~150 pages |
> | Claude Opus 4 | ~200K tokens | ~150 pages |
> | Gemini 1.5 Pro | ~1M tokens | ~750 pages |

---

## Bonnes pratiques

### La regle des 5V

```
┌───────────────────────────────────────────────────────────────┐
│                    LES 5V DE L'IA POUR LE CODE                │
│                                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │ VERIFIER │  │ VALIDER  │  │  VOIR    │  │ VARIER   │     │
│  │          │  │          │  │          │  │          │     │
│  │ Tester   │  │ Comprend │  │ Lire     │  │ Ne pas   │     │
│  │ le code  │  │ avant    │  │ chaque   │  │ dependre │     │
│  │ genere   │  │ d'utilis.│  │ ligne    │  │ d'un     │     │
│  │          │  │          │  │          │  │ seul     │     │
│  │          │  │          │  │          │  │ outil    │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
│                                                                │
│  ┌──────────┐                                                  │
│  │ VEILLER  │  Rester vigilant sur la securite,               │
│  │          │  les bonnes pratiques, et l'apprentissage       │
│  └──────────┘                                                  │
└───────────────────────────────────────────────────────────────┘
```

### Quand utiliser l'IA

| Bon usage | Mauvais usage |
|---|---|
| Boilerplate et code repetitif | Logique metier critique |
| Documentation et commentaires | Code de securite (auth, crypto) |
| Tests unitaires | Sans comprendre le resultat |
| Refactoring simple | Copier-coller sans relecture |
| Exploration d'APIs | Remplacer l'apprentissage |
| Debugging (expliquer une erreur) | Tout deleguer sans reflechir |
| Prototypage rapide | Code de production non revise |

### Checklist avant d'utiliser du code genere

```
[ ] J'ai lu et compris chaque ligne
[ ] Le code fait ce que j'attends (test manuel)
[ ] Les tests automatises passent
[ ] Pas de faille de securite evidente
[ ] Pas de donnees sensibles en dur
[ ] Les imports et dependances existent vraiment
[ ] Le style correspond aux conventions du projet
[ ] Je peux expliquer ce code a quelqu'un
```

---

## Considerations ethiques

### Attribution et honnetete

> [!warning] L'honnetete intellectuelle est non-negociable

| Situation | Ce qu'il faut faire |
|---|---|
| **Code genere par IA dans un projet pro** | Mentionner dans le commit ou la PR que l'IA a ete utilisee |
| **Exercice scolaire** | Suivre la politique de ton ecole (voir ci-dessous) |
| **Code open source** | Verifier la licence du code genere (risque de copyright) |
| **Code review** | Preciser si tu as utilise l'IA pour ta review |

### Politique IA a Holberton

> [!warning] Regles Holberton sur l'utilisation de l'IA
> - L'IA est un **outil**, pas un **raccourci**
> - Tu DOIS comprendre chaque ligne de code que tu soumets
> - Utiliser l'IA pour **apprendre** = encourage
> - Utiliser l'IA pour **tricher** (copier sans comprendre) = interdit
> - En cas de doute, demande a ton mentor ou staff
>
> La regle : **si tu ne peux pas expliquer ton code sans l'aide de l'IA, tu n'es pas pret a le soumettre.**

### Le paradoxe de l'apprentissage

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│   L'IA te rend plus PRODUCTIF...                             │
│   ...mais peut te rendre moins COMPETENT si mal utilisee.    │
│                                                               │
│   ┌─────────────────────────────────────────────────────┐    │
│   │                                                      │    │
│   │   L'IA comme TUTEUR        L'IA comme BEQUILLE      │    │
│   │   (tu apprends)            (tu n'apprends pas)      │    │
│   │                                                      │    │
│   │   "Explique-moi            "Fais-le pour moi"       │    │
│   │    pourquoi..."                                      │    │
│   │                                                      │    │
│   │   "Quelles sont les        "Donne-moi juste         │    │
│   │    alternatives ?"          le code"                 │    │
│   │                                                      │    │
│   │   "Qu'est-ce que ce        *copie-colle sans lire*  │    │
│   │    pattern resout ?"                                 │    │
│   │                                                      │    │
│   └─────────────────────────────────────────────────────┘    │
│                                                               │
│   OBJECTIF : Utiliser l'IA pour ACCELERER ton apprentissage, │
│   pas pour le CONTOURNER.                                    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

> [!tip] Analogie
> L'IA pour un developpeur, c'est comme une **calculatrice pour un mathematicien** :
> - Un mathematicien qui comprend les maths utilise la calculatrice pour aller plus vite
> - Un etudiant qui ne comprend rien et tape tout dans la calculatrice n'apprend jamais
> - La calculatrice ne remplace pas la comprehension, elle l'amplifie

---

## Carte Mentale

```
                         IA Assistants de Code
                                  |
              ┌───────────────────┼───────────────────┐
              |                   |                    |
         COMPLETION            CHAT               AGENTS
              |                   |                    |
        ┌─────┼─────┐     ┌──────┼──────┐      ┌──────┼──────┐
        |     |     |     |      |      |      |      |      |
     Copilot Codeium TabNine ChatGPT Claude  Gemini Claude  Cursor
                                                    Code   Agent
              |                   |
         Inline            Prompt Engineering
        Suggestions              |
                          ┌──────┼──────┐
                          |      |      |
                      Specifique Contexte Exemples
                          |
                    ┌─────┼──────────┐
                    |     |          |
                 Debug  Review  Generation
                                     |
                               ┌─────┼─────┐
                               |     |     |
                            Tests  Code   Docs
                                     |
                              ┌──────┼──────┐
                              |      |      |
                          Anti-    Limites  Ethique
                          patterns    |      |
                              |       |    Holberton
                          Copier   Halluc.  Policy
                          aveugle  Context
```

---

## Exercices

### Exercice 1 : Prompts efficaces

Pour chacune des taches suivantes, ecris un prompt detaille et efficace :
1. Debugger une fonction Python qui retourne `None` au lieu du resultat attendu
2. Generer une classe Python `BankAccount` avec gestion d'erreurs
3. Demander une code review d'une fonction de 50 lignes
4. Convertir une fonction synchrone en asynchrone

Compare tes prompts avec ceux d'un camarade. Lequel produit de meilleurs resultats ?

### Exercice 2 : Code review IA vs Humain

1. Prends un de tes anciens projets
2. Demande a ChatGPT ou Claude de faire une code review
3. Fais ta propre code review du meme code
4. Compare : qu'est-ce que l'IA a trouve que tu n'avais pas vu ? Et inversement ?
5. L'IA a-t-elle genere des faux positifs (problemes qui n'en sont pas) ?

### Exercice 3 : Detecter les hallucinations

Demande a une IA de generer du code pour :
1. Utiliser une API que tu connais bien (ex: requests, Flask, pandas)
2. Verifie chaque appel de fonction dans la **documentation officielle**
3. Note les hallucinations trouvees (fonctions inexistantes, parametres inventes, syntaxe incorrecte)

---

## Liens

- [[02 - IA et Productivite Dev]] - Comment integrer l'IA dans ton workflow quotidien de developpement