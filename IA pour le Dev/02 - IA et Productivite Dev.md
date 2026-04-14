# IA et Productivite Dev

L'intelligence artificielle transforme la facon dont les developpeurs travaillent au quotidien. Au-dela de la simple completion de code, l'IA intervient desormais dans chaque etape du cycle de developpement : generation de code, tests, documentation, revue de code, debugging, refactoring, et meme dans les pipelines CI/CD. Ce chapitre explore concretement comment integrer l'IA dans vos workflows pour multiplier votre productivite, tout en gardant un regard critique sur ses limites.

L'objectif n'est pas de remplacer le developpeur, mais de transformer l'IA en un **amplificateur de competences**. Un developpeur junior avec l'IA ne devient pas senior, mais un developpeur senior avec l'IA devient redoutable.

> [!tip] Analogie
> L'IA pour le developpeur est comme un copilote dans un avion. Le pilote (vous) prend les decisions strategiques, comprend la destination, et gere les situations imprevues. Le copilote (l'IA) execute les taches repetitives, surveille les instruments, et propose des corrections. Le pilote ne dort jamais aux commandes : il supervise tout ce que fait le copilote.

---

## Generation automatique de code

### Boilerplate et scaffolding

L'IA excelle pour generer du code repetitif et structurel. Plutot que d'ecrire manuellement chaque endpoint CRUD, chaque modele de donnees, ou chaque fichier de configuration, on decrit ce qu'on veut et l'IA produit le code.

```
Workflow de generation de code :

    Developpeur                    IA
        |                          |
        |  "Cree un modele User    |
        |   avec email, nom,      |
        |   date creation,        |
        |   et les endpoints      |
        |   CRUD en FastAPI"      |
        |------------------------->|
        |                          |
        |   Genere :               |
        |   - models/user.py       |
        |   - routes/user.py       |
        |   - schemas/user.py      |
        |   - tests/test_user.py   |
        |<-------------------------|
        |                          |
        |  Revue, ajustements,    |
        |  integration             |
        |                          |
```

> [!example] Generation d'un endpoint CRUD complet
> ```python
> # Prompt : "Cree un modele User SQLAlchemy avec endpoints FastAPI CRUD"
> # L'IA genere :
> 
> # models/user.py
> from sqlalchemy import Column, Integer, String, DateTime
> from sqlalchemy.sql import func
> from database import Base
> 
> class User(Base):
>     __tablename__ = "users"
> 
>     id = Column(Integer, primary_key=True, index=True)
>     email = Column(String, unique=True, index=True, nullable=False)
>     nom = Column(String, nullable=False)
>     created_at = Column(DateTime(timezone=True), server_default=func.now())
> 
> # schemas/user.py
> from pydantic import BaseModel, EmailStr
> from datetime import datetime
> 
> class UserCreate(BaseModel):
>     email: EmailStr
>     nom: str
> 
> class UserResponse(BaseModel):
>     id: int
>     email: str
>     nom: str
>     created_at: datetime
> 
>     class Config:
>         from_attributes = True
> 
> # routes/user.py
> from fastapi import APIRouter, Depends, HTTPException
> from sqlalchemy.orm import Session
> from models.user import User
> from schemas.user import UserCreate, UserResponse
> 
> router = APIRouter(prefix="/users", tags=["users"])
> 
> @router.post("/", response_model=UserResponse, status_code=201)
> def create_user(user: UserCreate, db: Session = Depends(get_db)):
>     db_user = User(**user.model_dump())
>     db.add(db_user)
>     db.commit()
>     db.refresh(db_user)
>     return db_user
> 
> @router.get("/{user_id}", response_model=UserResponse)
> def get_user(user_id: int, db: Session = Depends(get_db)):
>     user = db.query(User).filter(User.id == user_id).first()
>     if not user:
>         raise HTTPException(status_code=404, detail="Utilisateur non trouve")
>     return user
> 
> @router.put("/{user_id}", response_model=UserResponse)
> def update_user(user_id: int, user_data: UserCreate, db: Session = Depends(get_db)):
>     user = db.query(User).filter(User.id == user_id).first()
>     if not user:
>         raise HTTPException(status_code=404, detail="Utilisateur non trouve")
>     for key, value in user_data.model_dump().items():
>         setattr(user, key, value)
>     db.commit()
>     db.refresh(user)
>     return user
> 
> @router.delete("/{user_id}", status_code=204)
> def delete_user(user_id: int, db: Session = Depends(get_db)):
>     user = db.query(User).filter(User.id == user_id).first()
>     if not user:
>         raise HTTPException(status_code=404, detail="Utilisateur non trouve")
>     db.delete(user)
>     db.commit()
> ```

### Ce que l'IA genere bien vs mal

```
Generation de code par l'IA :

    EXCELLENT                           MEDIOCRE / DANGEREUX
    +-----------------------------------+-----------------------------------+
    | Boilerplate CRUD                  | Logique metier complexe           |
    | Modeles de donnees                | Algorithmes specifiques           |
    | Fichiers de configuration         | Code performance-critique         |
    | Scripts de migration              | Securite (crypto, auth)           |
    | Serialisation / deserialisation   | Gestion de concurrence            |
    | Scaffolding de projet             | Optimisations bas niveau          |
    | Conversions de format             | Decisions architecturales         |
    +-----------------------------------+-----------------------------------+
    
    Regle : plus le code est "standard" et "repetitif",
            meilleure est la generation par IA.
            Plus il est "unique" et "critique",
            plus la supervision humaine est essentielle.
```

> [!warning] Ne jamais faire confiance aveuglément
> L'IA peut generer du code qui **semble** correct mais contient des bugs subtils, des failles de securite, ou des anti-patterns. Chaque ligne generee doit etre lue, comprise, et validee par le developpeur. L'IA est un brouillon, pas un produit fini.

---

## IA pour les tests

### Generation de tests unitaires

L'IA peut analyser une fonction et generer automatiquement des tests qui couvrent les cas normaux, les cas limites, et les cas d'erreur.

```
Workflow de generation de tests :

    Code source              IA                   Tests generes
        |                    |                        |
    def divide(a, b):        |                        |
      return a / b    ------>| Analyse :              |
        |                    | - Type des params      |
        |                    | - Division par zero    |
        |                    | - Cas normaux          |
        |                    | - Negatifs, flottants  |
        |                    |----------------------->|
        |                    |                  test_divide_normal
        |                    |                  test_divide_zero
        |                    |                  test_divide_negative
        |                    |                  test_divide_float
        |                    |                  test_divide_large
```

> [!example] Generation de tests avec pytest
> ```python
> # Fonction source :
> def calculer_remise(prix: float, code: str | None = None) -> float:
>     """Calcule le prix apres remise."""
>     if prix < 0:
>         raise ValueError("Le prix ne peut pas etre negatif")
>     if code == "PROMO10":
>         return prix * 0.9
>     elif code == "PROMO20":
>         return prix * 0.8
>     elif code == "VIP":
>         return prix * 0.7
>     elif code is None:
>         return prix
>     else:
>         raise ValueError(f"Code promo invalide : {code}")
> 
> # Prompt : "Genere les tests pytest pour cette fonction"
> # L'IA genere :
> 
> import pytest
> from pricing import calculer_remise
> 
> class TestCalculerRemise:
>     """Tests pour la fonction calculer_remise."""
> 
>     # --- Cas normaux ---
>     def test_sans_code_promo(self):
>         assert calculer_remise(100.0) == 100.0
> 
>     def test_promo_10_pourcent(self):
>         assert calculer_remise(100.0, "PROMO10") == 90.0
> 
>     def test_promo_20_pourcent(self):
>         assert calculer_remise(100.0, "PROMO20") == 80.0
> 
>     def test_promo_vip(self):
>         assert calculer_remise(100.0, "VIP") == 70.0
> 
>     # --- Cas limites ---
>     def test_prix_zero(self):
>         assert calculer_remise(0.0, "PROMO10") == 0.0
> 
>     def test_prix_tres_petit(self):
>         result = calculer_remise(0.01, "PROMO10")
>         assert result == pytest.approx(0.009, abs=1e-6)
> 
>     def test_prix_tres_grand(self):
>         result = calculer_remise(1_000_000.0, "PROMO20")
>         assert result == 800_000.0
> 
>     # --- Cas d'erreur ---
>     def test_prix_negatif(self):
>         with pytest.raises(ValueError, match="negatif"):
>             calculer_remise(-10.0)
> 
>     def test_code_invalide(self):
>         with pytest.raises(ValueError, match="invalide"):
>             calculer_remise(100.0, "FAKE_CODE")
> 
>     # --- Edge cases que l'IA identifie ---
>     def test_code_none_explicite(self):
>         assert calculer_remise(50.0, None) == 50.0
> 
>     def test_code_promo_casse(self):
>         """Le code est-il sensible a la casse ? L'IA pose la question."""
>         with pytest.raises(ValueError):
>             calculer_remise(100.0, "promo10")  # minuscules
> ```

### TDD assiste par l'IA

```
Test-Driven Development avec IA :

    1. Ecrire les tests (humain + IA)
       |
       v
    2. Executer -> ROUGE (tests echouent)
       |
       v
    3. Demander a l'IA d'implementer -> VERT (tests passent)
       |
       v
    4. Refactorer (humain + IA)
       |
       v
    5. Re-executer -> toujours VERT
       |
       v
    Retour a 1 pour la prochaine fonctionnalite

    L'IA accelere les etapes 1 et 3.
    L'humain valide les etapes 2, 4 et 5.
```

> [!info] L'IA pour identifier les cas limites
> L'un des meilleurs usages de l'IA pour les tests est de lui demander : "Quels cas limites ai-je oublies ?" Elle peut identifier des edge cases que le developpeur n'a pas envisages : valeurs nulles, chaines vides, overflow, unicode, concurrence, etc.

---

## IA pour la documentation

### Generation automatique de docstrings

```python
# Avant : fonction sans documentation
def merge_intervals(intervals):
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    for current in intervals[1:]:
        if current[0] <= merged[-1][1]:
            merged[-1] = (merged[-1][0], max(merged[-1][1], current[1]))
        else:
            merged.append(current)
    return merged

# Apres : l'IA genere la docstring
def merge_intervals(intervals: list[tuple[int, int]]) -> list[tuple[int, int]]:
    """Fusionne les intervalles qui se chevauchent.

    Prend une liste d'intervalles (debut, fin) et retourne une nouvelle
    liste ou tous les intervalles qui se chevauchent ou se touchent ont
    ete fusionnes en un seul intervalle.

    Args:
        intervals: Liste de tuples (debut, fin) representant des intervalles.
                   Chaque tuple doit satisfaire debut <= fin.

    Returns:
        Liste triee d'intervalles fusionnes, sans chevauchement.

    Raises:
        IndexError: Si la liste est vide.

    Examples:
        >>> merge_intervals([(1, 3), (2, 6), (8, 10)])
        [(1, 6), (8, 10)]

        >>> merge_intervals([(1, 4), (4, 5)])
        [(1, 5)]

    Complexity:
        Temps : O(n log n) a cause du tri.
        Espace : O(n) pour la liste de sortie.
    """
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    for current in intervals[1:]:
        if current[0] <= merged[-1][1]:
            merged[-1] = (merged[-1][0], max(merged[-1][1], current[1]))
        else:
            merged.append(current)
    return merged
```

### Types de documentation generables par l'IA

```
Documentation que l'IA peut generer :

    Type                    Qualite IA    A verifier
    +------------------------------------------------------+
    Docstrings              Excellente    Exactitude des exemples
    README.md               Bonne         Ton, specificites projet
    API docs (OpenAPI)      Excellente    Completude des schemas
    Changelog               Bonne         Contexte business
    Comments inline         Variable      Pertinence (pas de "obvious")
    Architecture docs       Mediocre      Structure, decisions
    Tutoriels               Bonne         Parcours pedagogique
    Migration guides        Variable      Specificites du projet
    +------------------------------------------------------+
```

> [!tip] Analogie
> Demander a l'IA de documenter du code, c'est comme demander a un traducteur de resumer un livre. Il peut expliquer ce que fait chaque chapitre (fonction), mais il ne peut pas expliquer pourquoi l'auteur a choisi cette structure narrative (decisions architecturales). Le "quoi" est facile pour l'IA, le "pourquoi" reste humain.

---

## IA pour la revue de code

### Detection de bugs et vulnerabilites

L'IA peut analyser du code pour trouver des bugs, des failles de securite, des anti-patterns, et des problemes de performance.

```
Workflow de code review assistee par IA :

    Pull Request
        |
    +---+---+
    |       |
  Humain   IA
    |       |
    |   +---+---+---+---+
    |   |   |   |   |   |
    |  Bugs Secu Style Perf Doc
    |   |   |   |   |   |
    |   +---+---+---+---+
    |       |
    |   Commentaires
    |   automatiques
    |       |
    +---+---+
        |
    Decision humaine
    (merge / revisions)
```

> [!example] Revue de code par l'IA
> ```python
> # Code soumis en PR :
> def get_user(request):
>     user_id = request.GET.get('id')
>     query = f"SELECT * FROM users WHERE id = {user_id}"
>     cursor.execute(query)
>     user = cursor.fetchone()
>     password = user[3]
>     return JsonResponse({
>         'name': user[1],
>         'email': user[2],
>         'password': password,
>         'token': generate_token(user_id)
>     })
> 
> # L'IA identifie :
> # 1. CRITIQUE - Injection SQL : utiliser des requetes parametrees
> # 2. CRITIQUE - Exposition du mot de passe dans la reponse API
> # 3. MOYEN   - Pas de validation du user_id (pourrait etre None)
> # 4. MOYEN   - Pas de gestion si user n'existe pas (fetchone -> None)
> # 5. MINEUR  - Utiliser un ORM ou un dictionnaire au lieu d'indices
> # 6. MINEUR  - Pas de type hints
> ```

### Categories d'analyse

```
Ce que l'IA peut detecter en code review :

    SECURITE                        QUALITE
    +-------------------------------+-------------------------------+
    | Injection SQL                 | Code duplique                 |
    | XSS                          | Fonctions trop longues        |
    | Secrets en dur                | Nommage inconsistant          |
    | Acces non authentifie         | Complexite cyclomatique       |
    | Dependances vulnerables       | Couplage fort                 |
    | CORS mal configure            | Absence de logs               |
    | Deserialization non sure      | Magic numbers                 |
    +-------------------------------+-------------------------------+

    PERFORMANCE                     MAINTENABILITE
    +-------------------------------+-------------------------------+
    | Requetes N+1                  | Pas de tests                  |
    | Boucles inefficaces           | Pas de documentation          |
    | Allocations inutiles          | API inconsistante             |
    | Cache manquant                | Erreurs silencieuses          |
    | Index manquants (SQL)         | Dead code                     |
    | Fuites memoire                | TODO oublies                  |
    +-------------------------------+-------------------------------+
```

---

## IA pour le debugging

### Expliquer les erreurs

L'un des usages les plus immediatement utiles de l'IA : copier-coller un message d'erreur et obtenir une explication claire avec des pistes de resolution.

```
Workflow de debugging avec IA :

    Erreur incomprehensible
           |
           v
    "Explique cette erreur :"
    + message d'erreur
    + code concerne
    + contexte (OS, versions)
           |
           v
    IA explique :
    1. Ce que l'erreur signifie
    2. Pourquoi elle se produit
    3. Comment la corriger
    4. Comment la prevenir
           |
           v
    Developpeur applique
    la correction appropriee
```

> [!example] Debugging assiste
> ```
> Erreur :
> error[E0502]: cannot borrow `data` as mutable because it is also
>               borrowed as immutable
>   --> src/main.rs:5:5
>   |
> 3 |     let first = &data[0];
>   |                  ---- immutable borrow occurs here
> 5 |     data.push(42);
>   |     ^^^^^^^^^^^^ mutable borrow occurs here
> 6 |     println!("{}", first);
>   |                    ----- immutable borrow later used here
> 
> Explication de l'IA :
> "En Rust, vous ne pouvez pas avoir une reference immutable (&data[0])
>  et une reference mutable (data.push) actives en meme temps. Ici, `first`
>  emprunte data de facon immutable a la ligne 3, puis vous tentez de
>  modifier data a la ligne 5, mais `first` est encore utilise a la ligne 6.
> 
>  Solutions :
>  1. Utiliser first avant le push :
>     println!("{}", data[0]); data.push(42);
>  2. Cloner la valeur :
>     let first = data[0]; // copie, pas reference
>  3. Reorganiser le code pour separer les borrows."
> ```

### Analyse de cause racine

```
Analyse de cause racine avec l'IA :

    Symptome : "L'API retourne 500 sur /users depuis ce matin"
                                |
                                v
                    IA pose des questions :
                    - Qu'est-ce qui a change recemment ?
                    - Les logs montrent quoi ?
                    - La DB est accessible ?
                                |
                                v
                    IA analyse les logs :
                    "ConnectionRefused sur port 5432"
                                |
                                v
                    IA deduit :
                    "La base PostgreSQL n'est pas demarree
                     ou le port a change. Verifiez :
                     1. systemctl status postgresql
                     2. Le DATABASE_URL dans .env
                     3. Les regles firewall"
```

> [!info] L'IA comme canard en plastique
> Le "rubber duck debugging" consiste a expliquer son probleme a un canard en plastique pour trouver la solution soi-meme. L'IA est un canard qui repond. Souvent, le simple fait de formuler clairement le probleme pour l'IA vous aide a trouver la solution, et l'IA ajoute des pistes que vous n'aviez pas envisagees.

---

## IA pour le refactoring

### Moderniser du code

```python
# Avant : code Python 2 style
def process_data(data):
    result = []
    for item in data:
        if item.has_key('value'):               # Python 2
            val = item['value']
            if type(val) == int or type(val) == float:  # anti-pattern
                result.append(val * 2)
    return result

# Apres refactoring par l'IA :
def process_data(data: list[dict]) -> list[float]:
    """Double les valeurs numeriques presentes dans les donnees."""
    return [
        item['value'] * 2
        for item in data
        if 'value' in item and isinstance(item['value'], (int, float))
    ]
```

### Types de refactoring assistes par l'IA

```
Refactoring que l'IA fait bien :

    Type                    Exemple
    +------------------------------------------------------+
    Simplification          Remplacer boucles par
                            comprehensions/iterateurs
    
    Modernisation           Python 2->3, Java 8->17,
                            callbacks->async/await
    
    Extraction              Grande fonction -> plusieurs
                            petites fonctions
    
    Renommage               Variables/fonctions avec
                            noms plus descriptifs
    
    Design patterns         Transformer du code procedurale
                            en pattern Strategy, Observer, etc.
    
    Reduction complexite    Aplatir les if/else imbriques,
                            early returns, guard clauses
    
    Type safety             Ajouter type hints,
                            remplacer Any par types precis
    +------------------------------------------------------+
```

> [!warning] Refactoring et regression
> Le refactoring par l'IA peut introduire des regressions subtiles. Toujours avoir des tests **avant** de refactorer. Si le code n'a pas de tests, ecrivez-les d'abord (l'IA peut aider), puis refactorez.

---

## Agents IA de developpement

### Les outils disponibles

Les agents IA vont au-dela de la simple suggestion : ils peuvent editer plusieurs fichiers, executer des commandes, lancer des tests, et interagir avec git.

```
Spectre des outils IA pour le dev :

    Completion          Assistants          Agents
    (passif)            (interactif)        (autonome)
    |                   |                   |
    Copilot             ChatGPT/Claude      Claude Code
    TabNine             (chat interface)    Cursor Composer
    Codeium                                 Windsurf Cascade
    |                   |                   Cline
    |                   |                   |
    Suggere la          Repond aux          Edite les fichiers
    ligne suivante      questions           Execute les commandes
    |                   |                   Lance les tests
    |                   Genere des          Fait les commits
    |                   blocs de code       Cree des PRs
    |                   |                   |
    Contexte :          Contexte :          Contexte :
    fichier courant     conversation        projet entier
```

### Claude Code

```bash
# Installation
$ npm install -g @anthropic-ai/claude-code

# Utilisation dans un projet
$ cd mon-projet
$ claude

# Exemples de taches :
# "Ajoute la validation des emails dans le formulaire d'inscription"
# "Corrige le bug #42 : la pagination ne fonctionne pas"
# "Refactore le module auth pour utiliser JWT au lieu de sessions"
# "Ecris les tests manquants pour le service de paiement"
```

```
Ce que Claude Code peut faire :

    +-------------------------------------------+
    | Lire et comprendre le code existant       |
    | Editer plusieurs fichiers simultanement   |
    | Creer de nouveaux fichiers                |
    | Executer des commandes shell              |
    | Lancer les tests et corriger les erreurs  |
    | Faire des commits git                     |
    | Analyser les erreurs de compilation       |
    | Naviguer dans l'arborescence du projet    |
    | Chercher dans le code (grep, find)        |
    | Installer des dependances                 |
    +-------------------------------------------+

    Ce qu'il ne peut PAS faire seul :
    +-------------------------------------------+
    | Decisions architecturales                 |
    | Comprendre le contexte business           |
    | Valider la pertinence fonctionnelle       |
    | Garantir la securite                      |
    | Deployer en production (sans supervision) |
    +-------------------------------------------+
```

### Cursor, Windsurf, Cline

```
Comparaison des agents IA pour le dev :

    Outil         Type          Points forts                   Modeles
    +--------------------------------------------------------------------------+
    Claude Code   CLI agent     Multi-fichier, tests,          Claude
                                git, commandes shell
    
    Cursor        IDE (fork     Completion rapide,             Claude, GPT,
                  VS Code)      chat + composer,               modeles custom
                                contexte du projet
    
    Windsurf      IDE (fork     Cascade (agent),               Claude, GPT
                  VS Code)      flows autonomes,
                                contexte long
    
    Cline         Extension     Open source,                   Tout modele
                  VS Code       transparent (montre            via API
                                chaque action),
                                approval workflow
    +--------------------------------------------------------------------------+
```

> [!info] Choisir son outil
> - **Claude Code** : ideal pour les taches en terminal, scripts, automatisation, et developpeurs qui preferent la CLI
> - **Cursor** : ideal pour le developpement quotidien dans un IDE avec completion et chat integres
> - **Windsurf** : ideal pour les taches autonomes longues (refactoring de gros modules)
> - **Cline** : ideal pour ceux qui veulent de la transparence et du controle sur chaque action de l'IA

---

## MCP : Model Context Protocol

### Connecter l'IA a vos outils

MCP (Model Context Protocol) est un protocole ouvert qui permet aux modeles d'IA de se connecter a des sources de donnees et des outils externes de maniere standardisee.

```
Architecture MCP :

    +-------------------+
    |   Application     |
    |   (Claude Code,   |
    |    Cursor, etc.)  |
    +--------+----------+
             |
        MCP Protocol
             |
    +--------+----------+
    |    MCP Server      |     Un serveur par outil/source
    +--------+----------+
             |
    +--------+----------+
    |   Outil externe    |     Base de donnees, API,
    |   (DB, API, Git,   |     systeme de fichiers,
    |    Slack, etc.)     |     services cloud, etc.
    +--------------------+
```

```
Exemples de serveurs MCP :

    Serveur MCP          Permet a l'IA de...
    +------------------------------------------------------+
    PostgreSQL           Lire le schema, executer des
                         requetes SELECT, analyser les
                         donnees
    
    GitHub               Lire les PRs, issues, code,
                         creer des PRs, commenter
    
    Filesystem           Lire et ecrire des fichiers
                         dans des repertoires specifiques
    
    Slack                Lire les messages, poster
                         dans des channels
    
    Jira                 Lire et mettre a jour les
                         tickets, creer des sous-taches
    
    Sentry               Analyser les erreurs en
                         production, identifier les
                         patterns
    +------------------------------------------------------+
```

> [!tip] Analogie
> MCP est comme une prise USB universelle pour l'IA. Avant MCP, chaque outil avait son propre connecteur proprietaire. Avec MCP, n'importe quel serveur MCP peut se brancher a n'importe quel client MCP. C'est la standardisation qui multiplie les possibilites.

### Configuration MCP

```json
// Exemple de configuration MCP dans Claude Code
// .claude/settings.json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost:5432/mydb"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  }
}
```

---

## IA dans le CI/CD

### Revue automatisee de Pull Requests

```
Pipeline CI/CD avec IA :

    Push / PR
        |
        v
    +---+---+---+---+---+
    |   |   |   |   |   |
  Build Test Lint  IA   IA
    |   |   |   Review  Secu
    |   |   |   |       |
    |   |   | Comment Comment
    |   |   | sur PR  si vuln.
    |   |   |   |       |
    +---+---+---+---+---+
        |
    Merge (si tout vert)
```

> [!example] GitHub Action pour review IA
> ```yaml
> # .github/workflows/ai-review.yml
> name: AI Code Review
> on:
>   pull_request:
>     types: [opened, synchronize]
> 
> jobs:
>   ai-review:
>     runs-on: ubuntu-latest
>     steps:
>       - uses: actions/checkout@v4
>         with:
>           fetch-depth: 0
> 
>       - name: Get changed files
>         id: changed
>         run: |
>           echo "files=$(git diff --name-only origin/main...HEAD | tr '\n' ' ')" >> $GITHUB_OUTPUT
> 
>       - name: AI Review
>         uses: anthropic/claude-code-action@v1
>         with:
>           anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
>           prompt: |
>             Review the changes in this PR.
>             Focus on: bugs, security, performance, readability.
>             Comment directly on problematic lines.
>           files: ${{ steps.changed.outputs.files }}
> ```

### Scan de securite assiste par IA

```
Scan de securite traditionnel vs IA :

    Traditionnel (SAST)              IA-augmented
    +--------------------------------+--------------------------------+
    | Regles predefinies             | Comprend le contexte           |
    | Beaucoup de faux positifs      | Moins de faux positifs         |
    | Ne comprend pas le contexte    | Explique le risque             |
    | "SQL injection possible"       | "Cette requete concatene       |
    |                                |  user_input sans sanitization. |
    |                                |  Un attaquant pourrait extraire|
    |                                |  la table users. Corrigez avec |
    |                                |  une requete parametree."      |
    | Ne suggere pas de fix          | Propose le code corrige        |
    +--------------------------------+--------------------------------+
```

---

## Mesurer la productivite avec l'IA

### Metriques pertinentes

```
Metriques de productivite IA :

    Metrique                  Comment mesurer          Attention
    +--------------------------------------------------------------+
    Temps par tache           Timer avant/apres        Qualite doit
                              l'adoption de l'IA       rester egale
    
    Lignes de code/jour       Git stats                Pas toujours
                                                       pertinent
    
    Taux d'acceptation        % de suggestions IA      Trop haut =
    des suggestions           acceptees tel quel       pas assez de
                                                       review
    
    Bugs en production        Tracking post-deploy     Doit baisser
                                                       ou rester stable
    
    Couverture de tests       Coverage reports         Doit augmenter
    
    Temps de review           PR open -> merge         Doit diminuer
    
    Satisfaction dev           Sondages equipe         Doit augmenter
    +--------------------------------------------------------------+

    Indicateur sain :
    - Temps par tache : -30% a -50%
    - Bugs : stable ou en baisse
    - Couverture tests : en hausse
    - Satisfaction : en hausse
    
    Signal d'alarme :
    - Temps par tache : -70% (review insuffisante ?)
    - Bugs : en hausse (code IA non verifie ?)
    - Taux d'acceptation : >95% (copier-coller aveugle ?)
```

> [!warning] Le piege de la vitesse
> Coder plus vite ne signifie pas coder mieux. Si la productivite augmente mais que la qualite baisse, le gain est illusoire : le temps "economise" sera perdu en debugging, maintenance, et incidents en production. Mesurez toujours la productivite **et** la qualite ensemble.

---

## Construire des outils IA

### Utiliser l'API Anthropic

```python
# Installation : pip install anthropic

import anthropic

client = anthropic.Anthropic()  # Utilise ANTHROPIC_API_KEY

# Appel simple
message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": "Analyse ce code Python et liste les bugs potentiels :\n"
                       "def divide(a, b): return a/b"
        }
    ]
)
print(message.content[0].text)
```

```python
# Exemple : outil automatise de review de code
import anthropic
import subprocess

def review_git_diff() -> str:
    """Recupere le diff git et demande une review a Claude."""
    # Recuperer le diff
    diff = subprocess.run(
        ["git", "diff", "--staged"],
        capture_output=True, text=True
    ).stdout

    if not diff.strip():
        return "Aucun changement stage."

    client = anthropic.Anthropic()

    message = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=2048,
        system="Tu es un reviewer de code senior. "
               "Analyse le diff et identifie : bugs, securite, "
               "performance, lisibilite. Sois concis et actionable.",
        messages=[
            {
                "role": "user",
                "content": f"Review ce diff git :\n\n```diff\n{diff}\n```"
            }
        ]
    )
    return message.content[0].text

if __name__ == "__main__":
    print(review_git_diff())
```

> [!info] Prompt caching
> Pour les outils qui envoient souvent le meme contexte (documentation, regles de style), utilisez le prompt caching d'Anthropic. Le system prompt et les messages frequents sont mis en cache, reduisant le cout et la latence de 80-90%.

---

## Futur de l'IA dans le developpement

```
Evolution de l'IA pour le dev :

    2022              2024              2026+
    |                 |                 |
  Completion        Agents            Autonomie
  de code           multi-fichier     supervisee
    |                 |                 |
  "Suggere la       "Implemente       "Implemente la
   ligne suivante"   cette feature     feature, ecrit
                     dans ce fichier"  les tests, fait
                                       la PR, repond
                                       aux reviews"
    |                 |                 |
  Humain ecrit      Humain guide      Humain supervise
  90% du code       et valide         et decide
    |                 |                 |
  Productivite      Productivite      Productivite
  +10-20%           +30-50%           +50-80% ?
```

```
Modele de collaboration humain-IA :

    +---------------------------------------------------+
    |                                                   |
    |   Humain responsable de :     IA responsable de : |
    |                                                   |
    |   - Architecture              - Implementation    |
    |   - Decisions business        - Tests             |
    |   - Securite critique         - Documentation     |
    |   - Review finale             - Refactoring       |
    |   - Priorites                 - Boilerplate       |
    |   - Ethique                   - Debugging initial |
    |   - UX / design               - Recherche de code |
    |                                                   |
    |   Le "QUOI" et le "POURQUOI"  Le "COMMENT"        |
    |                                                   |
    +---------------------------------------------------+
```

> [!warning] Dependance et competences
> Le risque majeur est de creer une generation de developpeurs qui ne savent pas coder sans IA. Les fondamentaux (algorithmes, structures de donnees, patterns, debugging mental) restent essentiels. L'IA est un outil, pas un substitut a la competence. Un developpeur qui ne comprend pas le code genere par l'IA est un developpeur dangereux.

---

## Workflows pratiques au quotidien

### Workflow de developpement quotidien

```
Journee type avec IA :

    09:00  Revue des PRs de la nuit
           -> IA pre-analyse, humain valide
           |
    09:30  Planification (tickets du jour)
           -> IA resume les tickets, contexte
           |
    10:00  Implementation feature
           -> IA genere le scaffolding
           -> Humain ajuste la logique metier
           -> IA genere les tests
           |
    12:00  Debugging d'un bug reporte
           -> Copier les logs dans l'IA
           -> IA analyse la cause racine
           -> Humain valide et corrige
           |
    14:00  Refactoring module ancien
           -> IA propose les ameliorations
           -> Humain choisit lesquelles appliquer
           -> IA execute le refactoring
           -> Tests existants valident
           |
    16:00  Documentation
           -> IA genere les docstrings manquantes
           -> IA met a jour le README
           -> Humain verifie l'exactitude
           |
    17:00  PR et review
           -> IA formate la description de PR
           -> IA fait une pre-review
           -> Collegue fait la review humaine
```

### Workflow de debugging

```
Debugging systematique avec IA :

    1. REPRODUIRE
       "Voici les etapes pour reproduire le bug : ..."
       -> IA aide a identifier les conditions minimales
    
    2. ISOLER
       "Voici le stacktrace et les logs : ..."
       -> IA identifie le module/la fonction responsable
    
    3. COMPRENDRE
       "Explique ce que fait ce code et pourquoi il plante : ..."
       -> IA analyse le flux d'execution
    
    4. CORRIGER
       "Propose un fix pour ce probleme : ..."
       -> IA genere le correctif
    
    5. TESTER
       "Genere un test qui reproduit ce bug : ..."
       -> IA ecrit le test de regression
    
    6. VERIFIER
       Humain execute les tests
       Humain revoit le fix
       Humain valide qu'il n'y a pas de regression
```

### Workflow de code review

```
Code review avec IA :

    PR soumise
        |
        v
    Etape 1 : Pre-review IA (automatique)
        - Bugs evidents
        - Vulnerabilites connues
        - Style inconsistant
        - Tests manquants
        |
        v
    Etape 2 : Review humaine (focalisee)
        - Logique metier correcte ?
        - Architecture appropriee ?
        - Edge cases couverts ?
        - Performance acceptable ?
        |
        v
    Etape 3 : Discussion
        - Commentaires humain + IA
        - Auteur corrige
        |
        v
    Etape 4 : Validation finale (humain)
        - Merge
```

> [!tip] Analogie
> Le workflow de review avec IA est comme un controle qualite dans une usine. L'IA est la machine de controle automatique qui detecte 90% des defauts evidents (vis manquante, peinture ecaillee). L'humain est l'inspecteur final qui verifie ce que la machine ne peut pas voir (est-ce que le produit correspond au cahier des charges, est-ce que le client sera satisfait).

---

## Carte Mentale

```
                         IA ET PRODUCTIVITE DEV
                                  |
         +----------+----------+--+--+----------+----------+
         |          |          |     |          |          |
     GENERATION   TESTS    REVIEW  DEBUG    REFACTOR   AGENTS
         |          |          |     |          |          |
    Boilerplate  Unit tests  Bugs  Erreurs  Moderniser  Claude Code
    CRUD         Edge cases  Secu  Cause    Simplifier  Cursor
    Scaffolding  TDD assist  Style racine   Patterns    Windsurf
    Config       pytest      Perf  Pistes   Types       Cline
         |          |          |     |          |          |
         +-----+----+----+----+-----+----+-----+          |
               |         |          |         |            |
           CI/CD       MCP       MESURER    FUTUR       WORKFLOWS
               |         |          |         |            |
          PR review  Connecter   Temps    Autonomie    Quotidien
          Secu scan  DB, API     Qualite  supervisee   Debugging
          Quality    Git, Slack  Satis.   Humain =     Code review
          gates      Standard    Bugs     QUOI+POURQOI
                     protocol   Couvert. IA = COMMENT
```

---

## Exercices

### Exercice 1 : Creer un script de pre-commit avec IA

Ecrivez un script Python qui utilise l'API Anthropic pour analyser les fichiers modifies (via `git diff --staged`) et afficher des avertissements avant chaque commit. Le script doit verifier : bugs potentiels, secrets exposes, et TODO/FIXME oublies.

```python
# Indice : structure du script
import anthropic
import subprocess
import sys

def get_staged_diff() -> str:
    """Recupere le diff des fichiers stages."""
    result = subprocess.run(
        ["git", "diff", "--staged", "--diff-filter=ACMR"],
        capture_output=True, text=True
    )
    return result.stdout

def analyze_with_ai(diff: str) -> dict:
    """Envoie le diff a Claude pour analyse."""
    client = anthropic.Anthropic()
    # TODO : creer le prompt et parser la reponse
    # Le prompt doit demander un JSON avec :
    # { "bugs": [...], "secrets": [...], "todos": [...], "approve": bool }
    pass

def main():
    diff = get_staged_diff()
    if not diff:
        print("Aucun changement stage.")
        sys.exit(0)

    analysis = analyze_with_ai(diff)

    # Afficher les avertissements
    # Si "approve" est False, retourner exit code 1 (bloque le commit)
    # Sinon exit code 0
    pass

if __name__ == "__main__":
    main()
```

### Exercice 2 : Dashboard de metriques IA

Creez un script qui analyse l'historique git d'un projet et mesure l'impact de l'IA sur la productivite. Comparez les metriques avant et apres l'adoption de l'IA (en utilisant la date comme separateur).

```python
# Indice : metriques a calculer
# - Commits par jour (avant vs apres)
# - Lignes modifiees par commit
# - Temps entre commits
# - Ratio ajouts/suppressions
# - Taille moyenne des PRs

# Commandes git utiles :
# git log --after="2025-01-01" --format="%H %aI" --shortstat
# git log --before="2025-01-01" --format="%H %aI" --shortstat
```

### Exercice 3 : Serveur MCP personnalise

Creez un serveur MCP minimal qui expose les metriques de votre projet (nombre de fichiers, couverture de tests, derniers commits, issues ouvertes) a un client IA.

```python
# Indice : utiliser le SDK MCP Python
# pip install mcp

# Le serveur doit exposer ces "tools" :
# - get_project_stats() -> {files, lines, languages}
# - get_test_coverage() -> {covered, total, percentage}
# - get_recent_commits(n) -> [{hash, message, author, date}]
# - get_open_issues() -> [{id, title, labels, assignee}]

# Structure :
from mcp.server import Server
from mcp.types import Tool

server = Server("project-metrics")

@server.tool("get_project_stats")
async def get_project_stats():
    """Retourne les statistiques du projet."""
    # TODO : implementer avec subprocess (cloc, wc, etc.)
    pass
```

---

## Liens

- [[01 - IA Assistants de Code]] --- Les bases des assistants IA de code et leur fonctionnement
- [[04 - CI-CD avec GitHub Actions]] --- Integrer l'IA dans vos pipelines CI/CD
