# Prompt Engineering pour le Code

Qu'est-ce que le prompt engineering ? C'est l'art et la discipline de formuler des instructions précises à destination des modèles de langage (LLMs) pour en obtenir des résultats fiables, reproductibles et de haute qualité. Pour un développeur, maîtriser le prompt engineering, c'est transformer un outil généraliste en assistant spécialisé capable de générer du code de production, de diagnostiquer des bugs complexes, ou de refactorer un module entier — le tout avec une précision chirurgicale.

Le prompt n'est pas une requête magique. C'est une interface de communication structurée. La qualité de ce que vous obtenez est directement proportionnelle à la qualité de ce que vous demandez. Ce principe, aussi simple soit-il, change tout.

> [!tip] Analogie
> Un prompt est comme une requête SQL. `SELECT * FROM code` retourne n'importe quoi. `SELECT fonction, tests, commentaires FROM code WHERE langage='Python' AND version='3.12' AND style='PEP8' LIMIT 1` retourne exactement ce dont vous avez besoin. La précision de la requête détermine la pertinence du résultat — que ce soit en SQL ou avec un LLM.

---

## 1. Les fondamentaux du bon prompt

### Structure universelle d'un prompt efficace

Tout prompt de qualité suit une architecture en quatre blocs. Chaque bloc est optionnel selon le contexte, mais leur combinaison maximise la pertinence de la réponse.

```
+------------------+------------------------------------------+
|      BLOC        |              CONTENU                     |
+------------------+------------------------------------------+
| Rôle / Contexte  | Qui est l'IA dans cette interaction      |
|                  | Ex: "Tu es un expert Python FastAPI"     |
+------------------+------------------------------------------+
| Tâche précise    | Ce que tu veux exactement                |
|                  | Ex: "Génère un endpoint POST /users"     |
+------------------+------------------------------------------+
| Contraintes      | Ce que l'IA ne doit PAS faire            |
|                  | Ex: "Sans ORM, uniquement SQL brut"      |
+------------------+------------------------------------------+
| Format de sortie | Comment doit être structurée la réponse  |
|                  | Ex: "Code Python seul, sans explication" |
+------------------+------------------------------------------+
```

La formule complète :

```
[Rôle/Contexte] + [Tâche précise] + [Contraintes] + [Format de sortie]
```

### Mauvais prompts vs bons prompts

La différence entre un prompt médiocre et un excellent prompt n'est pas une question de longueur — c'est une question de précision et de contexte.

```
MAUVAIS                          BON
────────────────────────────────────────────────────────────────────
"Améliore ce code"               "Refactore cette fonction Python
                                  pour réduire la complexité
                                  cyclomatique, en gardant la même
                                  signature publique. Utilise des
                                  early returns plutôt que des
                                  if/else imbriqués."

"Corrige le bug"                 "Ce code Python 3.12 lève une
                                  KeyError sur la ligne 42 quand
                                  l'utilisateur n'a pas de rôle.
                                  Voici le traceback complet : [...]
                                  Propose un fix sans changer
                                  l'interface de la fonction."

"Écris des tests"                "Génère des tests pytest pour
                                  cette fonction. Couvre : le cas
                                  nominal, les valeurs nulles, les
                                  entrées invalides, et les cas
                                  limites (liste vide, très grande
                                  valeur). Utilise des fixtures
                                  pytest, pas unittest."

"Explique ce code"               "Explique ce code Rust à un
                                  développeur Python senior qui
                                  ne connaît pas le borrow checker.
                                  Focus sur la gestion mémoire,
                                  pas sur la syntaxe."
────────────────────────────────────────────────────────────────────
```

### L'importance du contexte technique

Le contexte fait toute la différence. L'IA ne connaît pas votre projet, votre stack, vos conventions. Sans contexte, elle va inventer des hypothèses — souvent mauvaises.

Ce qu'il faut toujours préciser :

```
Contexte projet      → "Application FastAPI avec PostgreSQL,
                        déployée sur AWS Lambda"
Langage et version   → "Python 3.12", "TypeScript 5.4", "Go 1.22"
Framework            → "React 18 avec hooks", "Django 4.2 REST"
Conventions          → "PEP8", "Airbnb style guide", "snake_case"
Dépendances clés     → "SQLAlchemy 2.0 (pas 1.x)", "Pydantic v2"
Contraintes métier   → "Pas de dépendances supplémentaires"
```

> [!info]
> Versionner les dépendances dans vos prompts est crucial. SQLAlchemy 1.x et 2.0 ont des APIs radicalement différentes. Pydantic v1 et v2 sont incompatibles. Un prompt qui ne précise pas la version peut générer du code qui ne fonctionne pas dans votre environnement.

### Spécifier le format de sortie

Si vous ne précisez pas le format, l'IA choisit. Et elle choisit rarement ce dont vous avez besoin.

```
Format de sortie                 Exemple de spécification
──────────────────────────────────────────────────────────────
Code seul                        "Code uniquement, sans explication"
Code + commentaires              "Code avec commentaires inline"
Code + tests                     "Code ET les tests pytest associés"
JSON                             "Réponds uniquement en JSON valide"
Markdown structuré               "Format: titre H3, liste à puces,
                                  bloc de code"
Diff Git                         "Sous forme de diff unifié"
Étapes numérotées                "Plan en 5 étapes numérotées"
```

---

## 2. Techniques fondamentales

### Zero-shot : demander directement

La technique la plus simple. Vous demandez sans donner d'exemple. Fonctionne bien pour les tâches standard et bien définies.

```python
# Prompt zero-shot
"""
Tu es un expert Python. Génère une fonction qui valide un email
avec une regex, retourne True/False, et lève une TypeError si
l'entrée n'est pas une string. Python 3.12.
"""

# Résultat attendu
import re

def validate_email(email: str) -> bool:
    if not isinstance(email, str):
        raise TypeError(f"Expected str, got {type(email).__name__}")
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

> [!info]
> Le zero-shot fonctionne mieux pour les tâches communes (tri, validation, CRUD). Pour des tâches très spécifiques à votre domaine ou à vos conventions maison, il faut passer au one-shot ou few-shot.

### One-shot : donner un exemple

Vous montrez un exemple du résultat que vous attendez. L'IA comprend le pattern et le reproduit.

```python
# Prompt one-shot
"""
Génère des fonctions de validation suivant ce modèle :

EXEMPLE :
def validate_age(age: int) -> bool:
    '''Valide que l'âge est entre 0 et 150.'''
    if not isinstance(age, int):
        raise TypeError(f"Expected int, got {type(age).__name__}")
    if not 0 <= age <= 150:
        raise ValueError(f"Age {age} hors limites [0, 150]")
    return True

MAINTENANT génère : validate_username (str, 3-50 chars, alphanumérique)
"""
```

L'IA va reproduire la structure exacte : docstring, TypeError, ValueError, return True.

### Few-shot : plusieurs exemples

Quand vous avez des conventions très spécifiques, donnez 2-3 exemples. L'IA détecte le pattern sous-jacent et l'applique avec précision.

```python
# Prompt few-shot pour des migrations SQL
"""
Génère des migrations SQL suivant ces modèles :

EXEMPLE 1 - Ajout de colonne :
-- Migration: add_email_to_users
-- Version: 20240115_001
ALTER TABLE users ADD COLUMN email VARCHAR(255) NOT NULL DEFAULT '';
CREATE INDEX idx_users_email ON users(email);
-- Rollback: ALTER TABLE users DROP COLUMN email;

EXEMPLE 2 - Création de table :
-- Migration: create_posts_table
-- Version: 20240116_001
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
-- Rollback: DROP TABLE posts;

MAINTENANT génère : add_role_to_users (enum: 'admin','user','guest',
default 'user')
"""
```

### Chain-of-Thought : penser étape par étape

Pour les problèmes complexes — algorithmes, debugging, architecture — forcer l'IA à raisonner pas à pas améliore considérablement la qualité.

```
Prompt Chain-of-Thought :
"Réfléchis étape par étape avant de répondre :
1. Analyse le problème
2. Identifie les cas limites
3. Propose une approche
4. Implémente la solution
5. Vérifie la logique

Problème : [votre problème ici]"
```

> [!example] Chain-of-Thought pour un algorithme
> ```
> Prompt : "Réfléchis étape par étape. Implémente un algorithme
> qui trouve les doublons dans une liste de 10M d'entiers avec
> O(n) en temps et O(1) en espace supplémentaire."
>
> L'IA va raisonner :
> Étape 1 : O(1) espace + O(n) temps = pas de hash set ni de tri
> Étape 2 : Seul Floyd's cycle detection ou bit manipulation
>            peuvent atteindre ces contraintes
> Étape 3 : Pour des entiers dans [1, n], on peut modifier
>            le tableau in-place comme marqueur
> Étape 4 : Implémentation concrète avec explication
> ```

### Role prompting : définir l'expertise

Assigner un rôle spécifique active un "mode" d'expertise chez le modèle. La différence est mesurable.

```python
# Sans role prompting
"Analyse ce code pour des problèmes de sécurité"

# Avec role prompting
"""
Tu es un expert en sécurité applicative spécialisé OWASP Top 10.
Tu réalises un audit de code pour une application bancaire.
Ta priorité : injection SQL, XSS, IDOR, et mauvaise gestion
des sessions. Sois exhaustif et critique.

Analyse ce code : [code]
"""
```

Les rôles utiles pour le développement :

```
"Tu es un senior backend Python avec 10 ans d'expérience FastAPI"
"Tu es un expert en performance SQL et optimisation de requêtes"
"Tu es un architecte cloud spécialisé AWS serverless"
"Tu es un expert sécurité OWASP, tu cherches les vulnérabilités"
"Tu es un lead tech qui fait une code review stricte"
"Tu es un expert en accessibilité WCAG 2.1"
```

---

## 3. Prompts spécifiques au code

### Générer du code

Le template de génération de code est le plus utilisé. Il doit être précis sur le contexte projet, le langage, les contraintes, et le format.

```
TEMPLATE GÉNÉRATION :
─────────────────────────────────────────────────
Contexte : [description du projet, stack, version]
Tâche : génère [composant précis]
Contraintes :
  - [contrainte 1]
  - [contrainte 2]
Format : [code seul / code + tests / code + doc]
─────────────────────────────────────────────────
```

> [!example] Générer un endpoint FastAPI avec auth JWT
> ```
> Contexte : API FastAPI 0.111, Python 3.12, PostgreSQL via
> SQLAlchemy 2.0, Pydantic v2. Authentification JWT avec
> python-jose. Les routes protégées utilisent un Depends()
> sur get_current_user().
>
> Tâche : génère l'endpoint POST /auth/login qui :
> - Reçoit email + password en JSON (Pydantic model)
> - Vérifie les credentials via SQLAlchemy
> - Retourne un JWT (access_token + token_type)
> - Lève HTTP 401 si credentials invalides
>
> Contraintes :
> - Password hashé avec bcrypt (passlib)
> - JWT expire dans 30 minutes
> - Pas de dépendances supplémentaires
>
> Format : code Python complet avec les imports,
> le router FastAPI, les schemas Pydantic, et
> la fonction de vérification du token séparée.
> ```

```python
# Résultat attendu (extrait)
from datetime import datetime, timedelta
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel, EmailStr
from sqlalchemy.orm import Session

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
router = APIRouter(prefix="/auth", tags=["auth"])

class LoginRequest(BaseModel):
    email: EmailStr
    password: str

class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"

def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

@router.post("/login", response_model=TokenResponse)
async def login(credentials: LoginRequest, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.email == credentials.email).first()
    if not user or not pwd_context.verify(credentials.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Identifiants invalides"
        )
    token = create_access_token({"sub": str(user.id)})
    return TokenResponse(access_token=token)
```

### Débugger

Le template de debugging est le plus critique. Plus vous donnez d'informations précises, plus le diagnostic sera juste.

```
TEMPLATE DEBUG :
─────────────────────────────────────────────────
Environnement : [OS, Python/Node/etc version, OS]
Code source : [le code minimal qui reproduit le bug]
Erreur exacte : [traceback complet, copié-collé]
Ce que j'ai essayé : [tentatives de fix déjà faites]
Comportement attendu : [ce qui devrait se passer]
─────────────────────────────────────────────────
```

> [!warning]
> Ne jamais paraphraser le traceback. Copiez-collez l'erreur exacte. "J'ai une erreur de type" est inutilisable. `TypeError: unsupported operand type(s) for +: 'int' and 'str' at line 42` est actionnable. L'IA peut lire les lignes de traceback et identifier précisément l'origine du problème.

> [!example] Prompt de debug Python
> ```
> Environnement : Python 3.12.3, Ubuntu 22.04, FastAPI 0.111
>
> Code :
> def calculate_total(items: list[dict]) -> float:
>     return sum(item["price"] * item["quantity"] for item in items)
>
> Erreur exacte :
> Traceback (most recent call last):
>   File "api/cart.py", line 23, in post_cart
>     total = calculate_total(cart_items)
>   File "api/cart.py", line 8, in calculate_total
>     return sum(item["price"] * item["quantity"] for item in items)
>   File "api/cart.py", line 8, in <genexpr>
>     return sum(item["price"] * item["quantity"] for item in items)
> TypeError: unsupported operand type(s) for *: 'str' and 'int'
>
> Ce que j'ai essayé : vérifier que items n'est pas vide (ça marche)
> Comportement attendu : retourner le total comme float
> ```

### Refactorer

Le template de refactoring exige de préciser l'objectif ET les contraintes de stabilité (ne pas casser l'interface publique).

```
TEMPLATE REFACTORING :
─────────────────────────────────────────────────
Code actuel : [le code à refactorer]
Objectif : [ce que tu veux améliorer]
Contraintes :
  - Garder la même signature des méthodes publiques
  - Ne pas changer le schéma de base de données
  - Les tests existants doivent passer
Principe appliqué : [SOLID, DRY, SRP, KISS...]
─────────────────────────────────────────────────
```

> [!example] Refactoring avec principe SOLID
> ```
> Code actuel : classe UserService de 300 lignes qui gère
> l'authentification, l'envoi d'emails, la mise à jour du profil,
> et l'export CSV.
>
> Objectif : appliquer le Single Responsibility Principle.
>
> Contraintes :
> - Garder l'interface publique identique (les méthodes appelées
>   depuis les routes FastAPI ne changent pas de signature)
> - Pas de nouvelles dépendances
> - Réorganiser en fichiers séparés dans services/
>
> Format : propose d'abord la structure de fichiers cible,
> puis le code de chaque service.
> ```

### Générer des tests

```
TEMPLATE TESTS :
─────────────────────────────────────────────────
Code source : [la fonction/classe à tester]
Framework : [pytest / jest / go test / etc]
Cas à couvrir :
  - Cas nominal (happy path)
  - Valeurs nulles / None / undefined
  - Entrées invalides / types incorrects
  - Cas limites (liste vide, très grande valeur)
  - Comportement des exceptions
Format : fichier de test complet avec imports
─────────────────────────────────────────────────
```

> [!example] Tests pytest complets
> ```python
> # Prompt : "Génère des tests pytest pour validate_email().
> # Couvre : cas nominal, email invalide, None, string vide,
> # et la TypeError pour entrée non-string."
>
> import pytest
> from validators import validate_email
>
> class TestValidateEmail:
>
>     def test_valid_email(self):
>         assert validate_email("user@example.com") is True
>
>     def test_valid_email_with_subdomain(self):
>         assert validate_email("user@mail.example.com") is True
>
>     def test_invalid_no_at(self):
>         assert validate_email("userexample.com") is False
>
>     def test_invalid_no_domain(self):
>         assert validate_email("user@") is False
>
>     def test_empty_string(self):
>         assert validate_email("") is False
>
>     def test_none_raises_typeerror(self):
>         with pytest.raises(TypeError):
>             validate_email(None)
>
>     def test_int_raises_typeerror(self):
>         with pytest.raises(TypeError):
>             validate_email(42)
>
>     def test_list_raises_typeerror(self):
>         with pytest.raises(TypeError):
>             validate_email(["user@example.com"])
> ```

### Documenter

```
TEMPLATE DOCUMENTATION :
─────────────────────────────────────────────────
Code source : [le code à documenter]
Audience : [débutant / développeur intermédiaire / expert]
Format : [docstring Python / JSDoc / README / OpenAPI]
Inclure : [paramètres, retours, exceptions, exemples d'usage]
─────────────────────────────────────────────────
```

> [!example] Docstring Google style
> ```
> Prompt : "Génère une docstring Google style pour cette fonction
> Python. Audience : développeur intermédiaire. Inclure les Args,
> Returns, Raises, et un Example."
>
> Résultat :
> def calculate_discount(price: float, rate: float) -> float:
>     """Calcule le prix après application d'une remise.
>
>     Args:
>         price: Prix original en euros, doit être positif.
>         rate: Taux de remise entre 0.0 et 1.0.
>              Ex: 0.2 pour 20% de remise.
>
>     Returns:
>         Prix après remise, arrondi à 2 décimales.
>
>     Raises:
>         ValueError: Si price <= 0 ou si rate n'est pas dans [0, 1].
>
>     Example:
>         >>> calculate_discount(100.0, 0.2)
>         80.0
>         >>> calculate_discount(99.99, 0.1)
>         89.99
>     """
> ```

### Code review

```
TEMPLATE CODE REVIEW :
─────────────────────────────────────────────────
Rôle : "Tu es un lead tech senior. Fais une code review stricte."
Code : [le code à reviewer]
Checklist :
  - Sécurité (injection, XSS, secrets en dur, permissions)
  - Performance (N+1 queries, allocations inutiles)
  - Lisibilité (nommage, complexité, commentaires)
  - Robustesse (gestion d'erreurs, edge cases)
  - Maintenabilité (couplage, cohésion, DRY)
Format : liste structurée par catégorie, sévérité [BLOQUANT/MINEUR]
─────────────────────────────────────────────────
```

### Expliquer du code complexe

```
TEMPLATE EXPLICATION :
─────────────────────────────────────────────────
Code : [le code à expliquer]
Contexte lecteur : [junior Python / dev backend sans connaissance Rust]
Focus : [la partie la plus complexe à comprendre]
Format : [explication narrative / commentaires inline / analogie]
─────────────────────────────────────────────────
```

---

## 4. Le hack des 95% de confiance

C'est l'une des techniques les plus puissantes pour travailler avec l'IA sur du code en production. Ajoutez cette instruction à vos prompts sensibles :

```
"Ne fais aucun changement tant que tu n'as pas 95% de confiance
dans ta compréhension du problème. Pose-moi des questions de suivi
si tu as besoin de clarification."
```

> [!tip] Analogie
> C'est comme dire à un chirurgien : "Ne faites pas l'incision tant que vous n'êtes pas certain du diagnostic." Un chirurgien qui opère avec 60% de certitude prend un risque énorme. Un LLM qui modifie du code avec des hypothèses incomplètes peut introduire des bugs subtils, difficiles à détecter et coûteux à corriger.

### Pourquoi c'est puissant

Sans cette instruction, les LLMs ont tendance à "combler les lacunes" avec des hypothèses plausibles mais incorrectes. Ils devinent le contexte manquant plutôt que de le demander.

```
COMPORTEMENT SANS LA RÈGLE :
─────────────────────────────────────────────────
Développeur : "Corrige le bug dans ma fonction de paiement"
IA : [modifie directement le code avec des hypothèses sur
      la bibliothèque de paiement, la devise, les validations...]

Résultat : code modifié qui peut introduire de nouveaux bugs
─────────────────────────────────────────────────

COMPORTEMENT AVEC LA RÈGLE :
─────────────────────────────────────────────────
Développeur : "Corrige le bug dans ma fonction de paiement.
               Ne fais rien tant que tu n'as pas 95% de confiance."
IA : "Avant d'intervenir, j'ai besoin de savoir :
      1. Quelle bibliothèque de paiement utilisez-vous (Stripe,
         PayPal, Braintree) ?
      2. Quelle est l'erreur exacte retournée ?
      3. Le problème survient-il en sandbox ou en production ?
      4. Avez-vous des logs de la transaction échouée ?"

Résultat : diagnostic précis avant toute modification
─────────────────────────────────────────────────
```

### Quand l'utiliser

```
Toujours utiliser sur :
  ✓ Refactoring de code critique (paiements, auth, données)
  ✓ Debugging de bugs intermittents ou difficiles à reproduire
  ✓ Modifications qui touchent plusieurs fichiers
  ✓ Code legacy sans tests
  ✓ Systèmes distribués ou asynchrones

Optionnel sur :
  ~ Génération de nouveau code (sans base existante)
  ~ Documentation
  ~ Génération de boilerplate standard
```

### Variantes de la technique

```python
# Variante 1 : plan avant exécution
"Explique ton plan de modification en détail avant d'écrire
une seule ligne de code."

# Variante 2 : liste des fichiers impactés
"Liste tous les fichiers que tu vas modifier et pourquoi,
avant de commencer."

# Variante 3 : validation incrémentale
"Procède étape par étape. Attends ma validation après chaque
étape avant de passer à la suivante."

# Variante 4 : hypothèses explicites
"Liste toutes les hypothèses que tu fais sur mon codebase
avant de répondre."
```

> [!info]
> Dans Claude Code, cette technique se combine naturellement avec le Plan Mode (Shift+Tab). Le Plan Mode force Claude à générer un plan détaillé et à attendre votre approbation avant d'exécuter des commandes. La règle des 95% de confiance fonctionne au niveau du prompt, le Plan Mode au niveau du workflow.

---

## 5. Gestion du contexte dans les prompts

### Ce qu'il faut inclure

```
INCLURE                              EXCLURE
────────────────────────────────────────────────────────────────
Le code minimal qui reproduit        L'intégralité du projet
  le problème
L'erreur exacte (traceback complet)  La paraphrase de l'erreur
Les dépendances pertinentes          Toutes les dépendances
  (juste celles utilisées)
La version des outils clés           L'historique complet du projet
Les contraintes métier               Le contexte organisationnel
  importantes
Les conventions de code              La documentation générale
  (si non-standard)
────────────────────────────────────────────────────────────────
```

### La règle du contexte minimal suffisant

```
Contexte minimal suffisant =
  tout ce qui est nécessaire pour comprendre le problème
  SANS ce qui dilue l'attention du modèle

Trop peu de contexte → hypothèses incorrectes → mauvaise réponse
Trop de contexte     → dilution de l'attention → réponse générique

L'objectif : le minimum qui rend le problème complètement
             compréhensible et reproductible
```

> [!warning]
> Les LLMs ont une fenêtre de contexte limitée et une "attention" qui se dilue avec la quantité d'informations. Coller 5000 lignes de code quand le bug est sur 20 lignes est contre-productif. L'IA va tenter de traiter tout le contexte et peut rater l'essentiel. Isolez le problème d'abord.

### Diriger vers des fichiers spécifiques

Dans les outils comme Claude Code, au lieu de coller le contenu, référencez le fichier :

```
# Moins efficace (colle tout le contenu)
"Voici mon fichier de 800 lignes : [contenu...]"

# Plus efficace (pointe vers le fichier)
"Regarde le fichier src/services/payment.py, function process_payment()
ligne 145-180. Le problème est là."
```

Cela économise des tokens, garde le contexte ciblé, et force l'IA à analyser précisément la zone indiquée.

---

## 6. Prompts par phase de projet

```
PHASE EXPLORATION
─────────────────────────────────────────────────
"Analyse l'architecture de ce projet et explique :
 - Les responsabilités de chaque module
 - Les dépendances entre composants
 - Les points d'entrée principaux
 Génère un diagramme ASCII de l'architecture."

"Quelles sont les 3 approches possibles pour implémenter [feature] ?
 Pour chaque approche : avantages, inconvénients, complexité."
─────────────────────────────────────────────────

PHASE DÉVELOPPEMENT
─────────────────────────────────────────────────
"Génère [composant] selon ces specs : [template génération]"
"Refactore [code] pour [objectif] : [template refactoring]"
─────────────────────────────────────────────────

PHASE DEBUG
─────────────────────────────────────────────────
"Diagnostique ce bug : [template debug]"
"Explique pourquoi cette erreur se produit et propose 3 fixes
 possibles avec leurs trade-offs."
─────────────────────────────────────────────────

PHASE QUALITÉ
─────────────────────────────────────────────────
"Génère des tests pour [code] : [template tests]"
"Fais une code review en mode senior lead tech : [template review]"
"Documente [code] pour [audience] : [template doc]"
─────────────────────────────────────────────────

PHASE DÉPLOIEMENT
─────────────────────────────────────────────────
"Génère un Dockerfile multi-stage pour cette app Python FastAPI.
 Base Alpine, user non-root, health check, taille minimale."

"Génère un pipeline GitHub Actions qui : lint (ruff), tests
 (pytest), build Docker, push ECR, deploy ECS. Python 3.12."

"Analyse ce fichier docker-compose.yml pour des problèmes
 de sécurité et de performance en production."
─────────────────────────────────────────────────
```

---

## 7. Les anti-patterns à éviter

### Inventaire des erreurs fréquentes

> [!warning]
> Ces anti-patterns sont la cause de 80% des déceptions avec l'IA. La plupart des gens qui disent "l'IA ne sert à rien" utilisent des prompts qui tombent dans une ou plusieurs de ces catégories.

```
ANTI-PATTERN 1 : Prompt trop vague
─────────────────────────────────────────────────
❌ "Améliore ce code"
❌ "Optimise la performance"
❌ "Rends ça plus propre"

Ces prompts sont impossibles à satisfaire correctement. L'IA
va choisir des améliorations arbitraires qui ne correspondent
peut-être pas à vos besoins.

✓ "Réduis la complexité cyclomatique de cette fonction
   en utilisant des early returns. Ne touche pas à la logique."
─────────────────────────────────────────────────

ANTI-PATTERN 2 : Contexte insuffisant
─────────────────────────────────────────────────
❌ "Corrige mon bug" (sans code, sans erreur, sans contexte)
❌ "Ça marche pas" (sans traceback, sans environnement)

✓ Utiliser le template debug complet
─────────────────────────────────────────────────

ANTI-PATTERN 3 : Format non spécifié
─────────────────────────────────────────────────
❌ Demander du code et obtenir 500 mots d'explication
   avec 10 lignes de code noyées dedans
❌ Vouloir du JSON et obtenir du markdown

✓ "Réponds uniquement avec le code Python. Pas d'explication."
✓ "Format : JSON valide uniquement, pas de texte autour."
─────────────────────────────────────────────────

ANTI-PATTERN 4 : Plusieurs tâches mélangées
─────────────────────────────────────────────────
❌ "Génère la fonction, écris les tests, documente-la,
    fais une code review, et donne-moi 3 alternatives"

L'IA va tout faire superficiellement plutôt que bien faire
une chose à la fois.

✓ Un prompt = une tâche. Enchaînez les prompts.
─────────────────────────────────────────────────

ANTI-PATTERN 5 : Contraintes techniques oubliées
─────────────────────────────────────────────────
❌ Ne pas mentionner la version → code incompatible
❌ Ne pas mentionner les dépendances → imports inventés
❌ Ne pas mentionner les conventions → style incohérent
─────────────────────────────────────────────────

ANTI-PATTERN 6 : Niveau non précisé
─────────────────────────────────────────────────
❌ Demander une explication sans préciser le niveau
   → trop basique pour un expert, trop complexe pour un débutant

✓ "Explique à un développeur Python intermédiaire qui ne
   connaît pas asyncio."
─────────────────────────────────────────────────
```

---

## 8. Stratégies de conversation multi-turn

### Décomposer les problèmes complexes

Un problème complexe ne se résout pas en un prompt. La stratégie multi-turn est plus efficace.

```
PROBLÈME COMPLEXE → DÉCOMPOSITION EN ÉTAPES
────────────────────────────────────────────────────────────────

Problème : "Migre notre API REST Flask vers FastAPI"

Turn 1 : "Analyse cette app Flask et génère un inventaire
          de tous les endpoints avec leurs méthodes, params,
          et modèles de données."

Turn 2 : "Sur la base de cet inventaire, propose une
          structure FastAPI équivalente. Ne génère pas
          encore le code."

Turn 3 : "Valide avec moi la structure proposée.
          [Feedback du développeur]"

Turn 4 : "Génère maintenant les routes FastAPI pour le
          module auth uniquement."

Turn 5 : "Génère les tests pour ces routes."

... (et ainsi de suite module par module)
────────────────────────────────────────────────────────────────
```

### Demander de valider avant d'exécuter

```python
# Prompt de validation préalable
"""
Avant d'écrire du code, présente-moi :
1. Ta compréhension du problème
2. L'approche que tu vas adopter
3. Les fichiers que tu vas créer ou modifier
4. Les dépendances que tu vas utiliser

Attends ma validation avant de commencer.
"""
```

### Donner du feedback correctif efficacement

```
FEEDBACK INEFFICACE          FEEDBACK EFFICACE
──────────────────────────────────────────────────────────────
"C'est pas bon"              "La logique de validation ligne 15-23
                              est incorrecte. Elle accepte les emails
                              sans TLD. Corrige uniquement cette partie."

"Ça marche pas"              "La fonction retourne None au lieu de []
                              quand la liste est vide. Voici le test
                              qui échoue : [test]. Fix uniquement ce
                              comportement."

"Refais tout"                "Garde la structure générale. Change
                              uniquement la gestion des erreurs :
                              utilise des exceptions custom au lieu
                              des codes de retour."
──────────────────────────────────────────────────────────────
```

### Relancer la conversation quand elle dérive

Les conversations longues dérivent. Le modèle peut commencer à ignorer des contraintes établies en début de conversation.

```
Signaux de dérive :
  - L'IA ignore des contraintes que vous aviez posées
  - Le style de code change progressivement
  - Les réponses deviennent moins précises
  - L'IA oublie le contexte du projet

Techniques de relance :
  1. Rappeler explicitement les contraintes oubliées :
     "Rappel : on utilise SQLAlchemy 2.0, pas 1.x"

  2. Résumer l'état actuel :
     "Récapitulatif : on a fait X, Y, Z. La prochaine étape est..."

  3. /compact (Claude Code) : compresse l'historique en gardant
     l'essentiel pour économiser des tokens

  4. /clear (Claude Code) : repart de zéro si la conversation
     est trop corrompue. Redonner le contexte essentiel.
```

> [!info]
> `/compact` est à utiliser quand la conversation est longue mais productive — vous voulez garder le contexte tout en économisant des tokens. `/clear` est à utiliser quand la conversation a dérivé à un point où recommencer est plus efficace que corriger. En général : `/compact` toutes les 20-30 turns, `/clear` quand l'IA répond n'importe quoi.

---

## 9. Adapter les prompts selon le modèle

Chaque modèle a des forces différentes. Adapter vos prompts au modèle utilisé améliore les résultats.

```
┌──────────────┬──────────────────────┬────────────────────────┐
│   Modèle     │     Points forts     │   Style de prompt      │
├──────────────┼──────────────────────┼────────────────────────┤
│ Claude       │ Suit les instructions│ Rôles précis, règles   │
│ (Anthropic)  │ précises, nuance,    │ explicites, format      │
│              │ code de qualité,     │ structuré. Excellent    │
│              │ refus cohérents      │ avec des contraintes    │
├──────────────┼──────────────────────┼────────────────────────┤
│ GPT-4o       │ Raisonnement         │ Chain-of-thought,       │
│ (OpenAI)     │ structuré, maths,    │ problèmes complexes,    │
│              │ multimodal           │ décomposition logique   │
├──────────────┼──────────────────────┼────────────────────────┤
│ Gemini 1.5   │ Très long contexte   │ Analyse de gros         │
│ (Google)     │ (1M tokens), Google  │ codebases, intégration  │
│              │ ecosystem            │ GCP/Firebase            │
├──────────────┼──────────────────────┼────────────────────────┤
│ Modèles      │ Confidentialité,     │ Instructions simples,   │
│ locaux       │ coût zéro, offline   │ directes. Éviter la     │
│ (Ollama)     │                      │ subtilité. Peu de       │
│              │                      │ rôle prompting complexe │
└──────────────┴──────────────────────┴────────────────────────┘
```

> [!info]
> Pour les modèles locaux (Llama 3, Mistral via Ollama), simplifiez radicalement vos prompts. Ces modèles ont moins de capacités de suivre des instructions complexes et multi-contraintes. Préférez des prompts courts, directs, avec une seule tâche. Le few-shot fonctionne bien avec eux.

### Tableau des différences de comportement

```
COMPORTEMENT               Claude      GPT-4o    Gemini    Local
──────────────────────────────────────────────────────────────────
Suivre des règles strictes   ★★★★★      ★★★★       ★★★       ★★
Raisonnement long            ★★★★       ★★★★★      ★★★       ★★
Long contexte                ★★★★       ★★★        ★★★★★     ★★
Code Python                  ★★★★★      ★★★★       ★★★       ★★★
Code JavaScript              ★★★★       ★★★★★      ★★★       ★★★
Refus appropriés             ★★★★★      ★★★        ★★        ★
Vitesse                      ★★★        ★★★        ★★★★      ★★★★★
Coût                         $$         $$$        $$        Gratuit
──────────────────────────────────────────────────────────────────
```

---

## 10. Productivité IA au quotidien

### Templates de prompts à garder

Créez un fichier `prompts.md` dans votre projet avec vos templates réutilisables. Quelques templates essentiels :

```markdown
# TEMPLATE : Debug rapide
Environnement : [OS, version]
Code : [code minimal]
Erreur : [traceback complet]
Essayé : [tentatives]
Attendu : [comportement voulu]
Règle : Ne change rien avec moins de 95% de confiance.

# TEMPLATE : Génération endpoint
Contexte : [stack, version]
Endpoint : [méthode + route + description]
Input : [schema]
Output : [schema]
Auth : [oui/non, méthode]
Erreurs : [codes HTTP à gérer]
Format : code + tests pytest

# TEMPLATE : Code review
Rôle : Lead tech Python senior, très critique.
Checklist : sécurité, performance, lisibilité, maintenabilité.
Format : liste par catégorie, sévérité [BLOQUANT/MINEUR/SUGGESTION]
Code : [code]

# TEMPLATE : Refactoring
Code actuel : [code]
Objectif : [amélioration]
Contraintes : même interface publique, mêmes tests passent
Principe : [SOLID/DRY/KISS]
Format : nouveau code + explication des changements
```

### Intégration dans le workflow

```
CLAUDE.md (racine du projet)
─────────────────────────────────────────────────
Ce fichier est lu automatiquement par Claude Code.
Mettez-y :
  - Description courte du projet et de son architecture
  - Stack technique avec versions
  - Conventions de code (nommage, style)
  - Commandes importantes (build, test, lint)
  - Zones sensibles à ne pas modifier sans confirmation
  - Dépendances externes (APIs, services)
─────────────────────────────────────────────────

SNIPPETS IDE
─────────────────────────────────────────────────
Dans VS Code / Cursor :
  Créez des User Snippets pour vos templates de prompts.
  Trigger : "dbg" → insère le template debug complet
  Trigger : "gen" → insère le template génération
  Trigger : "rev" → insère le template code review
─────────────────────────────────────────────────

WORKFLOW RECOMMANDÉ PAR TYPE DE TÂCHE
─────────────────────────────────────────────────
Nouvelle feature :
  1. Prompt exploration (architecture, options)
  2. Prompt génération (code + tests)
  3. Prompt review (qualité, sécurité)
  4. Prompt documentation (docstrings)

Bug critique :
  1. Prompt debug avec règle 95% de confiance
  2. Prompt génération du fix
  3. Prompt génération du test de non-régression

Refactoring :
  1. Prompt analyse du code actuel
  2. Prompt proposition de structure cible (validation)
  3. Prompt refactoring module par module
  4. Prompt vérification des tests
─────────────────────────────────────────────────
```

### Mesurer l'efficacité

```
Métriques simples à suivre :
┌──────────────────────────────┬────────────┬────────────┐
│ Tâche                        │ Avant IA   │ Après IA   │
├──────────────────────────────┼────────────┼────────────┤
│ Endpoint CRUD complet        │ 2-3 heures │ 20-30 min  │
│ Tests unitaires (couverture) │ 1-2 heures │ 15 min     │
│ Docstrings sur 10 fonctions  │ 45 min     │ 5 min      │
│ Diagnostic bug complexe      │ 2-4 heures │ 30-60 min  │
│ Code review 500 lignes       │ 1-2 heures │ 20 min     │
└──────────────────────────────┴────────────┴────────────┘

Note : ces gains supposent des prompts bien structurés.
Avec des prompts médiocres, le temps peut augmenter
(itérations, corrections, vérifications).
```

> [!tip] Analogie
> Un bon prompt engineering c'est comme apprendre à déléguer efficacement. Un manager qui dit "fais quelque chose d'utile aujourd'hui" obtient des résultats aléatoires. Un manager qui dit "prépare un rapport sur les ventes Q1 en deux pages, pour une présentation exec, avec graphiques, pour vendredi 17h" obtient exactement ce qu'il faut. La compétence de déléguer précisément est un multiplicateur de productivité — que ce soit avec des humains ou des LLMs.

---

## Carte Mentale

```
                    PROMPT ENGINEERING
                    pour le Code
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
  STRUCTURE            TECHNIQUES          SPÉCIFIQUE
  du prompt            de base             au code
       │                   │                   │
  ┌────┴────┐         ┌────┴────┐         ┌────┴────┐
  │ Rôle   │         │Zero-    │         │Générer  │
  │Contexte│         │shot     │         │         │
  ├────────┤         ├─────────┤         ├─────────┤
  │ Tâche  │         │One-shot │         │Débugger │
  │précise │         │         │         │         │
  ├────────┤         ├─────────┤         ├─────────┤
  │Contrain│         │Few-shot │         │Refactorer│
  │tes     │         │         │         │         │
  ├────────┤         ├─────────┤         ├─────────┤
  │Format  │         │Chain-of │         │Tester   │
  │sortie  │         │Thought  │         │         │
  └────────┘         ├─────────┤         ├─────────┤
                     │Role     │         │Documenter│
                     │prompting│         │         │
                     └─────────┘         └─────────┘
       │                   │                   │
  CONTEXTE             HACK 95%           ANTI-PATTERNS
  optimal              confiance               │
       │                   │              ┌────┴────┐
  ┌────┴────┐         ┌────┴────┐        │Vague    │
  │Inclure  │         │Questions│        │Contexte │
  │le minimum│         │avant    │        │insuffi- │
  │suffisant│         │action   │        │sant     │
  ├─────────┤         ├─────────┤        ├─────────┤
  │Erreurs  │         │Plan     │        │Plusieurs│
  │exactes  │         │avant    │        │tâches   │
  ├─────────┤         │exécution│        │mélangées│
  │Versions │         ├─────────┤        ├─────────┤
  │précises │         │/compact │        │Format   │
  └─────────┘         │vs /clear│        │non      │
                      └─────────┘        │précisé  │
                                         └─────────┘
       │                                     │
  MULTI-TURN                          PRODUCTIVITÉ
  conversation                        quotidienne
       │                                     │
  ┌────┴────┐                          ┌────┴────┐
  │Décompo- │                          │Templates│
  │ser en   │                          │réutili- │
  │étapes   │                          │sables   │
  ├─────────┤                          ├─────────┤
  │Valider  │                          │CLAUDE.md│
  │avant    │                          │projet   │
  │exécuter │                          ├─────────┤
  ├─────────┤                          │Snippets │
  │Feedback │                          │IDE      │
  │précis   │                          └─────────┘
  └─────────┘
```

---

## Exercices pratiques

### Exercice 1 : Transformer un mauvais prompt

Prenez ces mauvais prompts et réécrivez-les en suivant la structure `[Rôle] + [Tâche] + [Contraintes] + [Format]` :

```
Mauvais prompt 1 : "Améliore cette fonction Python"
Mauvais prompt 2 : "Écris des tests pour mon code"
Mauvais prompt 3 : "Explique ce que fait ce code"

Pour chaque prompt :
  a) Identifiez ce qui manque (contexte, contraintes, format)
  b) Réécrivez le prompt complet
  c) Testez les deux versions avec votre outil IA préféré
  d) Comparez la qualité des résultats
```

**Objectif :** Constater concrètement la différence de qualité entre un prompt vague et un prompt structuré. Mesurer le temps d'itération nécessaire pour chaque version.

### Exercice 2 : La règle des 95% en pratique

Choisissez un bug réel dans votre code (ou utilisez un bug inventé dans une fonction de paiement fictive).

```
Étape 1 : Envoyez ce prompt SANS la règle des 95% :
  "Corrige ce bug dans ma fonction de paiement : [code]"
  Notez ce que l'IA modifie et comment.

Étape 2 : Recommencez avec la règle :
  "Corrige ce bug. Ne fais aucun changement tant que tu n'as pas
  95% de confiance. Pose des questions si nécessaire."
  Notez les questions posées par l'IA.

Étape 3 : Comparez :
  - Combien d'hypothèses l'IA a-t-elle faites sans la règle ?
  - Les questions posées avec la règle étaient-elles pertinentes ?
  - Quelle version aurait donné le meilleur fix ?
```

**Objectif :** Développer l'habitude d'utiliser la règle des 95% sur du code sensible.

### Exercice 3 : Construire votre bibliothèque de templates

Créez votre propre fichier `prompts-templates.md` dans votre projet ou dans votre coffre Obsidian.

```
Structure recommandée :
  # Templates par catégorie
  ## Debug
  ## Génération de code
  ## Code review
  ## Tests
  ## Documentation
  ## Refactoring

Pour chaque template :
  a) Notez le contexte d'utilisation (quand l'utiliser)
  b) Écrivez le template avec des [PLACEHOLDERS]
  c) Ajoutez un exemple rempli
  d) Notez les variantes selon le modèle utilisé

Objectif final : avoir 2-3 templates par catégorie que vous
pouvez copier-coller et remplir en 30 secondes.
```

**Objectif :** Constituer un actif réutilisable qui s'améliore avec le temps. Chaque prompt réussi qui n'est pas sauvegardé est du temps perdu la prochaine fois.

---

## Liens

[[01 - Panorama des IA pour Développeurs]]
[[02 - Comprendre les LLMs et les Tokens]]
[[04 - Claude et Claude Code Maitrise Avancee]]
[[07 - Integrations IDE et Extensions]]
