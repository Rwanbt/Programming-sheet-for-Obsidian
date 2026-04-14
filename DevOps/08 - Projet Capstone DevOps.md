# Projet Capstone DevOps

## Qu'est-ce qu'un Projet Capstone ?

Un **projet Capstone** (littéralement "pierre de voute") est un projet de fin de parcours qui **integre toutes les competences** acquises durant la formation. Ce n'est pas un exercice isole sur un seul sujet : c'est un projet **end-to-end** ou votre equipe doit concevoir, developper, deployer et maintenir une application web complete avec un pipeline DevOps professionnel.

```
  Le Projet Capstone = La Pierre de Voute de votre Formation

                          ┌─────────────┐
                         /│  CAPSTONE   │\
                        / │  PROJECT    │ \
                       /  │             │  \
                      /   │ Application │   \
                     /    │ complete +  │    \
                    /     │ Pipeline    │     \
                   /      │ DevOps     │      \
                  /       └─────────────┘       \
                 /                               \
    ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
    │  Docker  │  │  CI/CD   │  │Terraform │  │ DevSec   │
    │  Compose │  │  GitHub  │  │  IaC     │  │  Ops     │
    │          │  │  Actions │  │          │  │          │
    └──────────┘  └──────────┘  └──────────┘  └──────────┘
    ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
    │  Linux   │  │  Git     │  │  Cloud   │  │Monitoring│
    │  Admin   │  │  Avance  │  │  Deploy  │  │Observ.   │
    └──────────┘  └──────────┘  └──────────┘  └──────────┘
```

> [!tip] Analogie
> Imaginez un etudiant en cuisine : pendant la formation, il apprend a faire des sauces, des cuissons, des patisseries **separement**. Le projet Capstone, c'est le **repas complet** qu'il doit preparer pour un jury : entree, plat, dessert, presentation, service. Tout doit fonctionner ensemble, dans les temps, en equipe. C'est la difference entre savoir faire une sauce bearnaise et gerer un restaurant.

### Ce qui est attendu

Un projet Capstone reussi comprend **4 livrables** :

| Livrable | Description | Poids |
|---|---|---|
| **Application fonctionnelle** | Une application web deployee et accessible publiquement | 40% |
| **Pipeline CI/CD** | Integration et deploiement continus automatises | 25% |
| **Documentation** | Architecture, API, deploiement, decisions techniques | 20% |
| **Presentation / Demo** | Demonstration live devant le jury, video de demo | 15% |

> [!warning] Ce qui fait echouer un Capstone
> - Une application qui ne fonctionne pas le jour de la demo
> - Aucun pipeline CI/CD (deploiement manuel = echec DevOps)
> - Pas de documentation (personne ne peut reprendre le projet)
> - Un seul membre de l'equipe qui a tout fait (le travail d'equipe est evalue)
> - Des secrets (mots de passe, cles API) commites dans le code source

### Roles dans l'Equipe

Dans une equipe de 3 a 5 personnes, chacun a un role principal mais doit comprendre le travail des autres :

```
  Equipe Capstone - Roles et Responsabilites

  ┌──────────────────────────────────────────────────────────────┐
  │                      CHEF DE PROJET                          │
  │         (un role, pas une personne a temps plein)             │
  │   Coordination, planning, communication, documentation       │
  └────────────────────────┬─────────────────────────────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
  ┌──────▼──────┐   ┌─────▼──────┐   ┌─────▼──────┐
  │  BACKEND    │   │  FRONTEND  │   │  DEVOPS /  │
  │  Dev        │   │  Dev       │   │  INFRA     │
  │             │   │            │   │            │
  │ - API       │   │ - UI/UX    │   │ - Docker   │
  │ - Database  │   │ - Pages    │   │ - CI/CD    │
  │ - Auth      │   │ - Appels   │   │ - Cloud    │
  │ - Tests     │   │   API      │   │ - Monitoring│
  │   unitaires │   │ - Tests    │   │ - Securite │
  └─────────────┘   └────────────┘   └────────────┘

  + QA / TESTEUR (souvent partage entre tous les membres)
    - Tests d'integration
    - Tests end-to-end
    - Revue de code
```

> [!info] Conseil pratique
> Dans une equipe de 3 personnes, le role DevOps est souvent combine avec le backend. Dans une equipe de 4-5, avoir un DevOps dedie est un enorme avantage. **Chaque membre devrait pouvoir expliquer le travail de tous les autres.**

---

## Choisir et Definir son Projet

### Types de Projets Possibles

Le Capstone est un projet **libre** : vous choisissez le sujet. Voici des categories eprouvees qui se pretent bien a une demonstration DevOps :

| Type de Projet | Exemples | Complexite DevOps |
|---|---|---|
| **SaaS Tool** | Outil de gestion de taches, CRM simplifie, outil de suivi de temps | ★★★ |
| **Marketplace** | Place de marche (achats, annonces, services) | ★★★★ |
| **Dashboard / Analytics** | Tableau de bord pour donnees meteo, finance, sport | ★★☆ |
| **API Platform** | API publique avec documentation, cles API, rate limiting | ★★★ |
| **Application collaborative** | Chat en temps reel, editeur collaboratif, kanban board | ★★★★ |
| **Portfolio / Blog Engine** | CMS personnalise avec admin panel | ★★☆ |

> [!tip] Analogie
> Choisir un projet, c'est comme choisir un **itineraire de road trip**. On ne choisit pas la destination la plus lointaine possible : on choisit un itineraire **realiste** avec des etapes interessantes. Un petit projet bien deploye impressionne plus qu'un gros projet a moitie fini.

> [!warning] Pieges courants dans le choix de projet
> - **Trop ambitieux** : "On va faire le prochain Airbnb" → vous n'aurez jamais fini
> - **Pas assez technique** : un site statique sans backend ne montre pas les competences DevOps
> - **Dependance a des APIs payantes** : preferez les APIs gratuites ou mockez les donnees
> - **Sujet qui n'interesse personne dans l'equipe** : la motivation est cle sur 6 semaines

### Definir le MVP (Minimum Viable Product)

Le **MVP** est la version la plus simple de votre application qui apporte de la valeur. C'est la strategie numero un pour reussir un Capstone : **livrez le MVP d'abord**, puis ameliorez.

```
  Methode de priorisation : MoSCoW

  ┌─────────────────────────────────────────────────────────────┐
  │                                                             │
  │  Must Have (MVP)          │  Should Have (V1.1)             │
  │  ─────────────────        │  ──────────────────             │
  │  ✓ Inscription/Login      │  ○ Profil utilisateur editable  │
  │  ✓ CRUD principal         │  ○ Filtres avances              │
  │  ✓ Page d'accueil         │  ○ Notifications email          │
  │  ✓ API fonctionnelle      │  ○ Export CSV/PDF               │
  │                           │                                 │
  │─────────────────────────────────────────────────────────────│
  │                                                             │
  │  Could Have (V2)          │  Won't Have (hors scope)        │
  │  ────────────────         │  ────────────────────           │
  │  ○ Mode sombre            │  ✗ Application mobile           │
  │  ○ Internationalisation   │  ✗ Paiement en ligne            │
  │  ○ Recherche full-text    │  ✗ IA / Machine Learning        │
  │  ○ WebSocket temps reel   │  ✗ Multi-tenant                 │
  │                                                             │
  └─────────────────────────────────────────────────────────────┘
```

### User Stories

Les **User Stories** definissent ce que l'application doit faire du point de vue de l'utilisateur :

```
Format : "En tant que [ROLE], je veux [ACTION] afin de [BENEFICE]"

Exemples pour une application de gestion de taches :

US-001 : En tant qu'utilisateur, je veux creer un compte
         afin d'avoir un espace personnel securise.
         Criteres d'acceptation :
         - Email et mot de passe requis
         - Mot de passe minimum 8 caracteres
         - Email de confirmation envoye
         - Redirection vers le dashboard apres inscription

US-002 : En tant qu'utilisateur connecte, je veux creer une tache
         afin de suivre mon travail.
         Criteres d'acceptation :
         - Titre obligatoire (max 100 caracteres)
         - Description optionnelle
         - Date d'echeance optionnelle
         - La tache apparait dans ma liste

US-003 : En tant qu'utilisateur connecte, je veux marquer une tache
         comme terminee afin de suivre ma progression.
         Criteres d'acceptation :
         - Un clic/tap suffit pour changer le statut
         - La tache passe en "terminee" visuellement
         - Un compteur de progression se met a jour
```

### Choix de la Stack Technique

Le choix de la stack depend de ce que l'equipe connait. Voici des combinaisons eprouvees :

| Stack | Frontend | Backend | Base de donnees | Ideal pour |
|---|---|---|---|---|
| **MERN** | React | Node.js/Express | MongoDB | Apps temps reel, API REST |
| **PERN** | React | Node.js/Express | PostgreSQL | Apps avec relations complexes |
| **Django + React** | React | Python/Django | PostgreSQL | Data-heavy apps, admin |
| **FastAPI + Vue** | Vue.js | Python/FastAPI | PostgreSQL | APIs modernes, documentation auto |
| **Next.js Full-Stack** | Next.js (React) | Next.js API Routes | PostgreSQL/Prisma | Apps monolithiques simples |
| **Laravel + Vue** | Vue.js | PHP/Laravel | MySQL | CRUD classiques, ecommerce |

> [!info] Conseil pour le Capstone
> **Ne choisissez pas une technologie que personne dans l'equipe ne connait.** Le Capstone n'est pas le moment d'apprendre un nouveau framework. Utilisez ce que vous maitrisez et concentrez-vous sur la partie DevOps.

### Architecture Decision Records (ADR)

Un **ADR** documente une decision technique importante et son raisonnement. C'est un livrable precieux pour le jury :

```markdown
# ADR-001 : Choix de PostgreSQL comme base de donnees

## Statut
Accepte (2026-01-15)

## Contexte
Notre application de gestion de taches necessite une base de donnees.
Nous avons considere MongoDB et PostgreSQL.

## Decision
Nous choisissons **PostgreSQL** car :
- Les donnees sont relationnelles (utilisateurs → taches → categories)
- L'equipe a plus d'experience avec SQL
- PostgreSQL offre des fonctionnalites avancees (JSONB, full-text search)
- Meilleur support dans les ORMs (Prisma, SQLAlchemy, Django ORM)

## Consequences
- Positives : Requetes complexes faciles, integrite referentielle
- Negatives : Schema plus rigide, migrations necessaires
- Risques : Performance sur les donnees non-structurees (mitige par JSONB)
```

> [!example] ADR a creer pour votre projet
> - ADR-001 : Choix de la base de donnees
> - ADR-002 : Choix du framework backend
> - ADR-003 : Strategie d'authentification
> - ADR-004 : Choix du provider cloud
> - ADR-005 : Monorepo vs polyrepo

---

## Architecture de l'Application

### Monolith vs Microservices

Pour un projet Capstone, le **monolith** est presque toujours le bon choix :

```
  Monolith (RECOMMANDE pour le Capstone)
  ──────────────────────────────────────

  ┌─────────────────────────────────────┐
  │         APPLICATION UNIQUE          │
  │                                     │
  │  ┌──────────┐  ┌──────────────────┐ │
  │  │ Routes   │  │ Logique Metier   │ │
  │  │ /api/... │──│ (services)       │ │
  │  └──────────┘  └────────┬─────────┘ │
  │                         │           │
  │              ┌──────────▼─────────┐ │
  │              │ Acces Donnees      │ │
  │              │ (ORM / Requetes)   │ │
  │              └──────────┬─────────┘ │
  │                         │           │
  └─────────────────────────┼───────────┘
                            │
                   ┌────────▼────────┐
                   │   PostgreSQL    │
                   └─────────────────┘

  Avantages :
  ✓ Simple a developper, tester, deployer
  ✓ Un seul repo, un seul pipeline CI/CD
  ✓ Pas de communication inter-services
  ✓ Parfait pour une equipe de 3-5 personnes


  Microservices (A EVITER pour le Capstone)
  ──────────────────────────────────────────

  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ Service  │   │ Service  │   │ Service  │
  │ Auth     │   │ Taches   │   │ Notifs   │
  │          │──►│          │──►│          │
  │ Port:3001│   │ Port:3002│   │ Port:3003│
  └────┬─────┘   └────┬─────┘   └────┬─────┘
       │              │              │
  ┌────▼─────┐   ┌────▼─────┐   ┌────▼─────┐
  │ DB Auth  │   │ DB Tasks │   │ DB Notif │
  └──────────┘   └──────────┘   └──────────┘

  Problemes pour un Capstone :
  ✗ Complexite demultipliee (3 pipelines, 3 DBs)
  ✗ Communication inter-services a gerer
  ✗ Debugging beaucoup plus difficile
  ✗ Temps perdu sur l'infra au lieu des features
```

> [!warning] Regle d'or
> **"Si votre equipe a besoin de microservices, votre projet est trop ambitieux pour un Capstone."** Les microservices resolvent des problemes d'echelle que vous n'aurez pas. Un monolith bien structure est plus impressionnant qu'un microservice mal implemente.

### Architecture 3-Tiers

Meme dans un monolith, l'architecture suit un modele en 3 couches :

```
  Architecture 3-Tiers Typique

  ┌─────────────────────────────────────────────────────────────┐
  │                    COUCHE PRESENTATION                       │
  │                      (Frontend)                              │
  │                                                             │
  │  ┌─────────────┐  ┌──────────────┐  ┌────────────────────┐ │
  │  │  React /    │  │  Pages /     │  │  Composants UI     │ │
  │  │  Vue /      │  │  Routes      │  │  (boutons, forms,  │ │
  │  │  Next.js    │  │  client      │  │   tableaux)        │ │
  │  └─────────────┘  └──────────────┘  └────────────────────┘ │
  │                                                             │
  │  Communique via HTTP/HTTPS (API REST ou GraphQL)           │
  └──────────────────────────┬──────────────────────────────────┘
                             │ JSON
                             ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                    COUCHE LOGIQUE METIER                     │
  │                      (Backend API)                           │
  │                                                             │
  │  ┌─────────────┐  ┌──────────────┐  ┌────────────────────┐ │
  │  │  Routes /   │  │  Services /  │  │  Middleware        │ │
  │  │  Controllers│  │  Business    │  │  (Auth, Logging,   │ │
  │  │  (endpoints)│  │  Logic       │  │   Validation)      │ │
  │  └─────────────┘  └──────────────┘  └────────────────────┘ │
  │                                                             │
  │  Communique via ORM / Query Builder / SQL                  │
  └──────────────────────────┬──────────────────────────────────┘
                             │ SQL
                             ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                    COUCHE DONNEES                            │
  │                    (Base de donnees)                         │
  │                                                             │
  │  ┌─────────────┐  ┌──────────────┐  ┌────────────────────┐ │
  │  │ PostgreSQL  │  │  Redis       │  │  Stockage fichiers │ │
  │  │ (donnees    │  │  (cache,     │  │  (S3 / stockage    │ │
  │  │  principales│  │   sessions)  │  │   local)           │ │
  │  └─────────────┘  └──────────────┘  └────────────────────┘ │
  └─────────────────────────────────────────────────────────────┘
```

### API-First Design

L'approche **API-first** consiste a definir vos endpoints avant d'ecrire le code. Cela permet au frontend et au backend de travailler en parallele :

```yaml
# Exemple de specification OpenAPI (extrait)
# fichier : openapi.yaml

openapi: "3.0.3"
info:
  title: "TaskMaster API"
  version: "1.0.0"
  description: "API pour l'application de gestion de taches"

paths:
  /api/auth/register:
    post:
      summary: "Inscription d'un utilisateur"
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [email, password, name]
              properties:
                email:
                  type: string
                  format: email
                password:
                  type: string
                  minLength: 8
                name:
                  type: string
      responses:
        "201":
          description: "Utilisateur cree"
        "409":
          description: "Email deja utilise"

  /api/tasks:
    get:
      summary: "Lister les taches de l'utilisateur"
      security:
        - bearerAuth: []
      parameters:
        - name: status
          in: query
          schema:
            type: string
            enum: [todo, in_progress, done]
      responses:
        "200":
          description: "Liste des taches"
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/Task"

    post:
      summary: "Creer une nouvelle tache"
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/TaskCreate"
      responses:
        "201":
          description: "Tache creee"
```

### Schema de Base de Donnees

Concevez votre schema avant de coder. Exemple pour une app de gestion de taches :

```sql
-- Schema de base de donnees pour TaskMaster

CREATE TABLE users (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email       VARCHAR(255) UNIQUE NOT NULL,
    password    VARCHAR(255) NOT NULL,  -- hash bcrypt
    name        VARCHAR(100) NOT NULL,
    avatar_url  VARCHAR(500),
    created_at  TIMESTAMP DEFAULT NOW(),
    updated_at  TIMESTAMP DEFAULT NOW()
);

CREATE TABLE projects (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name        VARCHAR(200) NOT NULL,
    description TEXT,
    owner_id    UUID REFERENCES users(id) ON DELETE CASCADE,
    created_at  TIMESTAMP DEFAULT NOW()
);

CREATE TABLE tasks (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title       VARCHAR(200) NOT NULL,
    description TEXT,
    status      VARCHAR(20) DEFAULT 'todo'
                CHECK (status IN ('todo', 'in_progress', 'done')),
    priority    VARCHAR(10) DEFAULT 'medium'
                CHECK (priority IN ('low', 'medium', 'high')),
    due_date    DATE,
    project_id  UUID REFERENCES projects(id) ON DELETE CASCADE,
    assignee_id UUID REFERENCES users(id) ON DELETE SET NULL,
    created_at  TIMESTAMP DEFAULT NOW(),
    updated_at  TIMESTAMP DEFAULT NOW()
);

-- Index pour les requetes frequentes
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_assignee ON tasks(assignee_id);
CREATE INDEX idx_tasks_project ON tasks(project_id);
```

### Strategie d'Authentification

Pour un projet Capstone, l'approche **JWT (JSON Web Tokens)** est la plus courante :

```
  Flux d'authentification JWT

  Client                          Serveur                    Base de donnees
    │                                │                            │
    │  POST /api/auth/login          │                            │
    │  {email, password}             │                            │
    │───────────────────────────────►│                            │
    │                                │  SELECT * FROM users       │
    │                                │  WHERE email = ?           │
    │                                │───────────────────────────►│
    │                                │◄───────────────────────────│
    │                                │                            │
    │                                │  bcrypt.compare(password)  │
    │                                │  Si OK → generer JWT       │
    │                                │                            │
    │  200 OK                        │                            │
    │  {access_token, refresh_token} │                            │
    │◄───────────────────────────────│                            │
    │                                │                            │
    │  GET /api/tasks                │                            │
    │  Authorization: Bearer <JWT>   │                            │
    │───────────────────────────────►│                            │
    │                                │  Verifier signature JWT    │
    │                                │  Extraire user_id          │
    │                                │  SELECT * FROM tasks       │
    │                                │  WHERE assignee_id = ?     │
    │                                │───────────────────────────►│
    │  200 OK                        │◄───────────────────────────│
    │  [{task1}, {task2}, ...]       │                            │
    │◄───────────────────────────────│                            │
```

> [!info] JWT vs Sessions
> - **JWT** : stateless, le serveur n'a pas besoin de stocker la session. Ideal pour les APIs REST. Attention a la duree de vie (15min pour access token, 7j pour refresh token).
> - **Sessions** : stockees cote serveur (Redis). Plus simples a revoquer. Mieux pour les applications web classiques.
> - **OAuth2** : pour le "Login with Google/GitHub". Excellent pour le Capstone car ca impressionne le jury et c'est plus facile qu'on ne le pense.

---

## Setup du Projet — Du Zero a la Premiere Feature

### Configuration du Repository Git

Un repository professionnel fait la difference lors de l'evaluation :

```bash
# Creer le repository
mkdir taskmaster && cd taskmaster
git init

# Structure initiale du projet
mkdir -p .github/workflows
mkdir -p .github/ISSUE_TEMPLATE
mkdir -p .github/PULL_REQUEST_TEMPLATE
mkdir -p docs/adr
mkdir -p src
mkdir -p tests
```

**Protection de la branche main** (dans les parametres GitHub) :

```
  Regles de protection de la branche main :

  ┌─────────────────────────────────────────────────────────────┐
  │  Branch protection rules pour "main"                        │
  │                                                             │
  │  ☑ Require a pull request before merging                    │
  │    ☑ Require approvals: 1                                   │
  │    ☑ Dismiss stale pull request approvals when new          │
  │      commits are pushed                                     │
  │                                                             │
  │  ☑ Require status checks to pass before merging             │
  │    ☑ Require branches to be up to date before merging       │
  │    Status checks required:                                  │
  │      ✓ ci / lint                                            │
  │      ✓ ci / test                                            │
  │      ✓ ci / build                                           │
  │                                                             │
  │  ☑ Require conversation resolution before merging           │
  │  ☐ Require signed commits (optionnel)                       │
  │  ☑ Do not allow bypassing the above settings                │
  └─────────────────────────────────────────────────────────────┘
```

**Template de Pull Request** (`.github/pull_request_template.md`) :

```markdown
## Description
<!-- Decrivez les changements et pourquoi ils sont necessaires -->

## Type de changement
- [ ] Nouvelle feature
- [ ] Bug fix
- [ ] Refactoring
- [ ] Documentation
- [ ] Configuration / DevOps

## Checklist
- [ ] Mon code suit les conventions du projet
- [ ] J'ai ajoute des tests pour mes changements
- [ ] Tous les tests existants passent
- [ ] J'ai mis a jour la documentation si necessaire
- [ ] J'ai verifie qu'il n'y a pas de secrets dans le code

## Screenshots (si applicable)
<!-- Ajoutez des captures d'ecran pour les changements UI -->

## Lien vers l'issue
Closes #
```

**Template d'Issue** (`.github/ISSUE_TEMPLATE/bug_report.md`) :

```markdown
---
name: Bug Report
about: Signaler un bug
labels: bug
---

## Description du bug
<!-- Description claire et concise -->

## Etapes pour reproduire
1. Aller sur '...'
2. Cliquer sur '...'
3. Observer l'erreur

## Comportement attendu
<!-- Ce qui devrait se passer -->

## Comportement actuel
<!-- Ce qui se passe reellement -->

## Screenshots
<!-- Si applicable -->

## Environnement
- OS: [ex: Windows 11, macOS 14]
- Navigateur: [ex: Chrome 120]
- Version de l'app: [ex: 1.0.0]
```

### Structure du Projet (Monorepo)

Pour un Capstone, le **monorepo** est recommande (tout dans un seul repository) :

```
taskmaster/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml              # Pipeline d'integration continue
│   │   └── cd.yml              # Pipeline de deploiement continu
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── pull_request_template.md
├── docs/
│   ├── adr/
│   │   ├── 001-choix-database.md
│   │   ├── 002-choix-framework.md
│   │   └── 003-strategie-auth.md
│   ├── architecture.md
│   └── api.md
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   ├── services/
│   │   ├── models/
│   │   ├── middleware/
│   │   └── app.js              # ou main.py
│   ├── tests/
│   ├── Dockerfile
│   ├── package.json            # ou requirements.txt
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/           # appels API
│   │   └── App.jsx
│   ├── tests/
│   ├── Dockerfile
│   ├── package.json
│   └── .env.example
├── infra/
│   ├── terraform/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── ansible/
│   │   └── playbook.yml
│   └── nginx/
│       └── nginx.conf
├── docker-compose.yml          # Dev local
├── docker-compose.prod.yml     # Production
├── .gitignore
├── .pre-commit-config.yaml
├── LICENSE
└── README.md
```

### Environnement de Developpement avec Docker Compose

Un `docker-compose.yml` pour le developpement local permet a toute l'equipe d'avoir le meme environnement :

```yaml
# docker-compose.yml - Environnement de developpement local

services:
  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
      target: development       # Multi-stage : utilise l'etape "development"
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://taskmaster:secret@db:5432/taskmaster
      - JWT_SECRET=dev-secret-key-change-in-production
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./backend/src:/app/src  # Hot reload : le code est monte en volume
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: development
    ports:
      - "5173:5173"             # Vite dev server
    environment:
      - VITE_API_URL=http://localhost:3000/api
    volumes:
      - ./frontend/src:/app/src # Hot reload

  # Base de donnees PostgreSQL
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: taskmaster
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: taskmaster
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backend/sql/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U taskmaster"]
      interval: 5s
      timeout: 5s
      retries: 5

  # Cache Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  # Interface d'administration de la base de donnees (optionnel)
  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - db

volumes:
  postgres_data:
```

```bash
# Lancer tout l'environnement de dev
docker compose up -d

# Voir les logs de tous les services
docker compose logs -f

# Voir les logs d'un seul service
docker compose logs -f backend

# Arreter tout
docker compose down

# Arreter et supprimer les volumes (reset complet de la DB)
docker compose down -v
```

### Pre-commit Hooks

Les **pre-commit hooks** verifient automatiquement le code avant chaque commit :

```yaml
# .pre-commit-config.yaml

repos:
  # Hooks generaux
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace      # Supprime les espaces en fin de ligne
      - id: end-of-file-fixer        # Ajoute une ligne vide en fin de fichier
      - id: check-yaml                # Verifie la syntaxe YAML
      - id: check-json                # Verifie la syntaxe JSON
      - id: check-added-large-files   # Empeche les gros fichiers (> 500KB)
        args: ['--maxkb=500']
      - id: detect-private-key        # Detecte les cles privees
      - id: no-commit-to-branch       # Interdit le commit direct sur main
        args: ['--branch', 'main']

  # Detecter les secrets
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

```bash
# Installation des pre-commit hooks
pip install pre-commit
pre-commit install

# Tester les hooks sur tous les fichiers
pre-commit run --all-files
```

### README Professionnel

Un README de qualite professionnelle est un livrable important :

```markdown
# TaskMaster - Application de Gestion de Taches

![CI](https://github.com/equipe/taskmaster/actions/workflows/ci.yml/badge.svg)
![CD](https://github.com/equipe/taskmaster/actions/workflows/cd.yml/badge.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> Application collaborative de gestion de taches avec pipeline DevOps complet.
> Projet Capstone - Formation DevOps 2026.

![Screenshot de l'application](docs/screenshots/dashboard.png)

## Fonctionnalites

- Inscription / Connexion (JWT + OAuth2 Google)
- Gestion de projets et de taches (CRUD complet)
- Tableau Kanban avec drag-and-drop
- Notifications en temps reel (WebSocket)
- API REST documentee (Swagger)

## Stack Technique

| Composant | Technologie |
|---|---|
| Frontend | React 18 + TypeScript + Tailwind CSS |
| Backend | Node.js 20 + Express + Prisma ORM |
| Base de donnees | PostgreSQL 16 |
| Cache | Redis 7 |
| CI/CD | GitHub Actions |
| Conteneurisation | Docker + Docker Compose |
| Infrastructure | Terraform + AWS ECS |
| Monitoring | Prometheus + Grafana |

## Demarrage Rapide

### Prerequis
- Docker et Docker Compose
- Node.js 20+ (pour le developpement local sans Docker)
- Git

### Installation
(instructions git clone, copie .env, docker compose up)

## Documentation
- [Documentation API (Swagger)](https://api.taskmaster.example.com/docs)
- [Architecture (C4 Model)](docs/architecture.md)
- [Decisions Techniques (ADR)](docs/adr/)
- [Guide de Contribution](CONTRIBUTING.md)

## Equipe
| Nom | Role | GitHub |
|---|---|---|
| Alice | Backend + DevOps | @alice |
| Bob | Frontend | @bob |
| Charlie | Full-Stack + QA | @charlie |

## Licence
Ce projet est sous licence MIT. Voir [LICENSE](LICENSE) pour plus de details.
```

---

## Pipeline CI/CD Complet

### Vue d'Ensemble du Pipeline

```
  Pipeline CI/CD Complet - Du Code a la Production

  ┌──────────────────────────────────────────────────────────────────────┐
  │  TRIGGER : Push sur feature branch                                   │
  │                                                                      │
  │  ┌──────┐   ┌───────────┐   ┌───────────┐   ┌──────┐              │
  │  │ LINT │──►│ TYPE CHECK│──►│UNIT TESTS │──►│BUILD │              │
  │  │      │   │           │   │           │   │      │              │
  │  │ESLint│   │TypeScript │   │Jest/Pytest│   │Docker│              │
  │  │Ruff  │   │mypy       │   │Coverage   │   │Image │              │
  │  └──────┘   └───────────┘   └───────────┘   └──────┘              │
  └──────────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────────────┐
  │  TRIGGER : Pull Request vers main                                    │
  │                                                                      │
  │  (tout ce qui precede) +                                             │
  │                                                                      │
  │  ┌───────────┐   ┌───────────┐   ┌──────────────┐                  │
  │  │INTEGRATION│──►│ SECURITY  │──►│ DEPLOY       │                  │
  │  │  TESTS    │   │   SCAN    │   │ STAGING      │                  │
  │  │           │   │           │   │              │                  │
  │  │API tests  │   │Trivy      │   │Environnement│                  │
  │  │E2E tests  │   │npm audit  │   │de test      │                  │
  │  └───────────┘   └───────────┘   └──────────────┘                  │
  └──────────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────────────┐
  │  TRIGGER : Merge dans main                                           │
  │                                                                      │
  │  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐            │
  │  │ BUILD PROD   │──►│ DEPLOY PROD  │──►│ SMOKE TESTS  │            │
  │  │              │   │              │   │              │            │
  │  │Docker image  │   │Cloud deploy  │   │Verifier que  │            │
  │  │tag: v1.2.3   │   │(ECS, VM...) │   │tout marche   │            │
  │  └──────────────┘   └──────────────┘   └──────────────┘            │
  └──────────────────────────────────────────────────────────────────────┘
```

### Workflow CI Complet (`.github/workflows/ci.yml`)

```yaml
# .github/workflows/ci.yml
# Integration Continue - S'execute sur chaque push et pull request

name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: "20"
  PYTHON_VERSION: "3.12"

jobs:
  # ──────────────────────────────────────────
  # Job 1 : Linting et formatage
  # ──────────────────────────────────────────
  lint:
    name: "Lint & Format"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"
          cache-dependency-path: |
            backend/package-lock.json
            frontend/package-lock.json

      - name: Install backend dependencies
        run: cd backend && npm ci

      - name: Install frontend dependencies
        run: cd frontend && npm ci

      - name: Lint backend
        run: cd backend && npm run lint

      - name: Lint frontend
        run: cd frontend && npm run lint

      - name: Check formatting (Prettier)
        run: |
          cd backend && npx prettier --check "src/**/*.{js,ts}"
          cd ../frontend && npx prettier --check "src/**/*.{jsx,tsx,css}"

  # ──────────────────────────────────────────
  # Job 2 : Verification des types TypeScript
  # ──────────────────────────────────────────
  type-check:
    name: "Type Check"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"
          cache-dependency-path: |
            backend/package-lock.json
            frontend/package-lock.json

      - name: Install & check backend types
        run: cd backend && npm ci && npx tsc --noEmit

      - name: Install & check frontend types
        run: cd frontend && npm ci && npx tsc --noEmit

  # ──────────────────────────────────────────
  # Job 3 : Tests unitaires
  # ──────────────────────────────────────────
  test:
    name: "Unit Tests"
    runs-on: ubuntu-latest

    # Service PostgreSQL pour les tests
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: taskmaster_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"
          cache-dependency-path: backend/package-lock.json

      - name: Install dependencies
        run: cd backend && npm ci

      - name: Run database migrations
        run: cd backend && npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/taskmaster_test

      - name: Run unit tests with coverage
        run: cd backend && npm test -- --coverage --ci
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/taskmaster_test
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: test-secret-key

      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: backend/coverage/

      - name: Frontend unit tests
        run: cd frontend && npm ci && npm test -- --ci

  # ──────────────────────────────────────────
  # Job 4 : Build des images Docker
  # ──────────────────────────────────────────
  build:
    name: "Build Docker Images"
    runs-on: ubuntu-latest
    needs: [lint, type-check, test]   # S'execute seulement si les 3 jobs precedents reussissent
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build backend image
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: false
          tags: taskmaster-backend:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Build frontend image
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: false
          tags: taskmaster-frontend:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ──────────────────────────────────────────
  # Job 5 : Scan de securite (sur PR seulement)
  # ──────────────────────────────────────────
  security:
    name: "Security Scan"
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    needs: [build]
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner (backend)
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: "fs"
          scan-ref: "./backend"
          format: "table"
          exit-code: "1"              # Echoue si vulnerabilites critiques
          severity: "CRITICAL,HIGH"

      - name: Run npm audit (backend)
        run: cd backend && npm audit --audit-level=high
        continue-on-error: true       # Warning mais n'echoue pas le build

      - name: Run npm audit (frontend)
        run: cd frontend && npm audit --audit-level=high
        continue-on-error: true
```

### Workflow CD Complet (`.github/workflows/cd.yml`)

```yaml
# .github/workflows/cd.yml
# Deploiement Continu - S'execute lors du merge dans main

name: CD

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ──────────────────────────────────────────
  # Job 1 : Build et push des images Docker
  # ──────────────────────────────────────────
  build-and-push:
    name: "Build & Push Images"
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract version tag
        id: meta
        run: |
          # Utilise le SHA court du commit comme tag
          echo "tag=$(echo ${{ github.sha }} | cut -c1-7)" >> $GITHUB_OUTPUT
          echo "date=$(date +'%Y%m%d')" >> $GITHUB_OUTPUT

      - name: Build and push backend
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-backend:latest
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-backend:${{ steps.meta.outputs.tag }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Build and push frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-frontend:latest
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-frontend:${{ steps.meta.outputs.tag }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

    outputs:
      image-tag: ${{ steps.meta.outputs.tag }}

  # ──────────────────────────────────────────
  # Job 2 : Deploy to staging
  # ──────────────────────────────────────────
  deploy-staging:
    name: "Deploy to Staging"
    runs-on: ubuntu-latest
    needs: [build-and-push]
    environment:
      name: staging
      url: https://staging.taskmaster.example.com

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to staging server via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /opt/taskmaster
            docker compose -f docker-compose.prod.yml pull
            docker compose -f docker-compose.prod.yml up -d
            docker compose -f docker-compose.prod.yml ps

      - name: Run smoke tests on staging
        run: |
          # Attendre que le service demarre
          sleep 15
          # Verifier que l'API repond
          curl --fail --retry 5 --retry-delay 5 \
            https://staging.taskmaster.example.com/api/health
          echo "Staging deployment OK"

  # ──────────────────────────────────────────
  # Job 3 : Deploy to production (manuel)
  # ──────────────────────────────────────────
  deploy-production:
    name: "Deploy to Production"
    runs-on: ubuntu-latest
    needs: [deploy-staging]
    environment:
      name: production
      url: https://taskmaster.example.com
    # "environment: production" avec protection rules dans GitHub
    # = approbation manuelle requise avant deploiement

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to production server via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /opt/taskmaster
            # Sauvegarder la version actuelle (rollback possible)
            docker compose -f docker-compose.prod.yml ps > /tmp/deploy-backup.txt
            # Pull les nouvelles images
            docker compose -f docker-compose.prod.yml pull
            # Deployer avec zero downtime
            docker compose -f docker-compose.prod.yml up -d --remove-orphans
            # Verifier la sante
            docker compose -f docker-compose.prod.yml ps

      - name: Run production smoke tests
        run: |
          sleep 15
          curl --fail --retry 5 --retry-delay 5 \
            https://taskmaster.example.com/api/health
          echo "Production deployment successful!"

      - name: Notify team (Slack/Discord)
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: "Production deployment: ${{ job.status }}"
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Tests Matriciels

Pour tester sur plusieurs versions :

```yaml
  # Tester le backend sur plusieurs versions de Node.js
  test-matrix:
    name: "Test Node ${{ matrix.node-version }}"
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
      fail-fast: false    # Continuer meme si une version echoue

    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: cd backend && npm ci && npm test
```

### Strategies de Cache

Le caching accelere considerablement les pipelines :

```yaml
  # Cache npm (automatique avec setup-node et l'option cache)
  - uses: actions/setup-node@v4
    with:
      node-version: "20"
      cache: "npm"
      cache-dependency-path: backend/package-lock.json

  # Cache Docker layers (avec Buildx)
  - uses: docker/build-push-action@v5
    with:
      cache-from: type=gha        # Lire le cache GitHub Actions
      cache-to: type=gha,mode=max # Ecrire toutes les couches dans le cache

  # Cache pip (Python)
  - uses: actions/setup-python@v5
    with:
      python-version: "3.12"
      cache: "pip"
      cache-dependency-path: backend/requirements.txt
```

### Gestion des Secrets

```
  Comment gerer les secrets dans GitHub Actions

  ┌─────────────────────────────────────────────────────────────┐
  │  Settings → Secrets and variables → Actions                  │
  │                                                             │
  │  Repository secrets :                                       │
  │  ┌───────────────────────────────────────────────┐          │
  │  │  STAGING_HOST         = 203.0.113.10          │          │
  │  │  STAGING_USER         = deploy                │          │
  │  │  STAGING_SSH_KEY      = -----BEGIN OPENSSH... │          │
  │  │  PROD_HOST            = 203.0.113.20          │          │
  │  │  PROD_USER            = deploy                │          │
  │  │  PROD_SSH_KEY         = -----BEGIN OPENSSH... │          │
  │  │  DATABASE_URL         = postgresql://...      │          │
  │  │  JWT_SECRET           = <random-256-bit>      │          │
  │  │  SLACK_WEBHOOK        = https://hooks.slack...│          │
  │  └───────────────────────────────────────────────┘          │
  │                                                             │
  │  Environments :                                             │
  │  ┌───────────────────────────────────────────────┐          │
  │  │  staging :                                    │          │
  │  │    Pas de protection (deploiement auto)       │          │
  │  │                                               │          │
  │  │  production :                                 │          │
  │  │    ☑ Required reviewers: @team-lead           │          │
  │  │    ☑ Wait timer: 5 minutes                    │          │
  │  │    ☑ Deployment branches: main only           │          │
  │  └───────────────────────────────────────────────┘          │
  └─────────────────────────────────────────────────────────────┘
```

> [!warning] Regles de securite pour les secrets
> - **Jamais** de secrets dans le code source (meme dans les commits anciens)
> - Toujours utiliser `.env.example` (sans les valeurs reelles) dans le repo
> - Generer des secrets forts : `openssl rand -hex 32`
> - Rotationner les secrets regulierement
> - Utiliser des secrets differents pour staging et production

### Strategies de Deploiement

```
  Comparaison des strategies de deploiement

  ─── Rolling Update (mise a jour progressive) ───

  Temps 1: [v1] [v1] [v1] [v1]    ← 4 instances en v1
  Temps 2: [v2] [v1] [v1] [v1]    ← mise a jour 1 par 1
  Temps 3: [v2] [v2] [v1] [v1]
  Temps 4: [v2] [v2] [v2] [v1]
  Temps 5: [v2] [v2] [v2] [v2]    ← toutes en v2

  ✓ Zero downtime
  ✓ Rollback possible
  ✗ Mix de versions pendant la transition


  ─── Blue-Green Deployment ───

  Blue (actuel):    [v1] [v1] [v1] [v1]  ← recoit le trafic
  Green (nouveau):  [v2] [v2] [v2] [v2]  ← pret mais pas actif

  Load Balancer bascule : Blue → Green

  Blue (ancien):    [v1] [v1] [v1] [v1]  ← pret pour rollback
  Green (actuel):   [v2] [v2] [v2] [v2]  ← recoit le trafic

  ✓ Rollback instantane (rebascule vers Blue)
  ✓ Pas de mix de versions
  ✗ Double de ressources necessaire


  ─── Canary Deployment ───

  [v1] [v1] [v1] [v1]        ← 100% du trafic

  [v1] [v1] [v1] [v2]        ← 10% du trafic vers v2
                                  Surveiller les erreurs...

  [v1] [v1] [v2] [v2]        ← 50% du trafic vers v2
                                  Tout va bien...

  [v2] [v2] [v2] [v2]        ← 100% du trafic vers v2

  ✓ Risque minimal (si bug, seulement X% affectes)
  ✓ Feedback rapide
  ✗ Plus complexe a mettre en place
```

> [!info] Pour un projet Capstone
> Utilisez le **Rolling Update** (c'est le comportement par defaut de Docker Compose avec `docker compose up -d`). Si vous voulez impressionner le jury, implementez un **Blue-Green** simple avec Nginx qui bascule entre deux containers.

---

## Infrastructure as Code

### Terraform pour les Ressources Cloud

Terraform permet de creer votre infrastructure cloud de facon reproductible et versionnee :

```hcl
# infra/terraform/main.tf
# Infrastructure pour le projet Capstone

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  # Stocker l'etat Terraform dans S3 (recommande pour le travail en equipe)
  backend "s3" {
    bucket = "taskmaster-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "eu-west-3"  # Paris
  }
}

provider "aws" {
  region = var.aws_region
}

# ──────────────────────────────────────────
# Reseau (VPC)
# ──────────────────────────────────────────
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name    = "${var.project_name}-vpc"
    Project = var.project_name
  }
}

resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.project_name}-public-${count.index + 1}"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
}

resource "aws_route_table_association" "public" {
  count          = 2
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# ──────────────────────────────────────────
# Security Group
# ──────────────────────────────────────────
resource "aws_security_group" "app" {
  name_prefix = "${var.project_name}-app-"
  vpc_id      = aws_vpc.main.id

  # HTTP
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # HTTPS
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # SSH (restreindre a votre IP en production !)
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.allowed_ssh_cidr]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# ──────────────────────────────────────────
# Instance EC2 (VM simple pour le Capstone)
# ──────────────────────────────────────────
resource "aws_instance" "app" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public[0].id
  key_name      = var.key_pair_name

  vpc_security_group_ids = [aws_security_group.app.id]

  user_data = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y docker.io docker-compose-v2
    systemctl enable docker
    systemctl start docker
    usermod -aG docker ubuntu
  EOF

  root_block_device {
    volume_size = 30
    volume_type = "gp3"
  }

  tags = {
    Name    = "${var.project_name}-server"
    Project = var.project_name
  }
}

# ──────────────────────────────────────────
# Base de donnees RDS PostgreSQL
# ──────────────────────────────────────────
resource "aws_db_subnet_group" "main" {
  name       = "${var.project_name}-db-subnet"
  subnet_ids = aws_subnet.public[*].id
}

resource "aws_db_instance" "postgres" {
  identifier        = "${var.project_name}-db"
  engine            = "postgres"
  engine_version    = "16"
  instance_class    = "db.t3.micro"  # Free tier eligible
  allocated_storage = 20

  db_name  = var.db_name
  username = var.db_username
  password = var.db_password

  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.db.id]

  skip_final_snapshot = true  # Pour le dev seulement !

  tags = {
    Name    = "${var.project_name}-database"
    Project = var.project_name
  }
}

# ──────────────────────────────────────────
# Outputs
# ──────────────────────────────────────────
output "server_ip" {
  value = aws_instance.app.public_ip
}

output "database_endpoint" {
  value = aws_db_instance.postgres.endpoint
}

data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }
}
```

```hcl
# infra/terraform/variables.tf

variable "project_name" {
  description = "Nom du projet"
  default     = "taskmaster"
}

variable "aws_region" {
  description = "Region AWS"
  default     = "eu-west-3"  # Paris
}

variable "instance_type" {
  description = "Type d'instance EC2"
  default     = "t3.small"
}

variable "key_pair_name" {
  description = "Nom de la paire de cles SSH"
  type        = string
}

variable "allowed_ssh_cidr" {
  description = "CIDR autorise pour SSH"
  default     = "0.0.0.0/0"  # A RESTREINDRE en production !
}

variable "db_name" {
  description = "Nom de la base de donnees"
  default     = "taskmaster"
}

variable "db_username" {
  description = "Utilisateur de la base de donnees"
  default     = "taskmaster"
  sensitive   = true
}

variable "db_password" {
  description = "Mot de passe de la base de donnees"
  type        = string
  sensitive   = true
}
```

```bash
# Commandes Terraform

# Initialiser le projet (telecharger les providers)
cd infra/terraform
terraform init

# Voir ce qui va etre cree
terraform plan -var="key_pair_name=my-key" -var="db_password=SuperSecret123!"

# Creer l'infrastructure
terraform apply -var="key_pair_name=my-key" -var="db_password=SuperSecret123!"

# Detruire tout (attention !)
terraform destroy
```

### Ansible pour la Configuration du Serveur

Si vous utilisez une VM (pas un service managed comme ECS), Ansible configure le serveur :

```yaml
# infra/ansible/playbook.yml
# Configuration du serveur de production

---
- name: Configure TaskMaster production server
  hosts: production
  become: true
  vars:
    app_dir: /opt/taskmaster
    deploy_user: deploy

  tasks:
    # ── Mise a jour du systeme ──
    - name: Update system packages
      apt:
        update_cache: yes
        upgrade: safe

    # ── Installer Docker ──
    - name: Install Docker prerequisites
      apt:
        name:
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: present

    - name: Add Docker GPG key
      ansible.builtin.apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker repository
      ansible.builtin.apt_repository:
        repo: "deb https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
        state: present

    - name: Install Docker
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-compose-plugin
        state: present

    - name: Ensure Docker is running
      systemd:
        name: docker
        state: started
        enabled: yes

    # ── Creer l'utilisateur de deploiement ──
    - name: Create deploy user
      user:
        name: "{{ deploy_user }}"
        groups: docker
        shell: /bin/bash

    # ── Creer le repertoire de l'application ──
    - name: Create application directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ deploy_user }}"
        group: "{{ deploy_user }}"

    # ── Copier les fichiers de configuration ──
    - name: Copy docker-compose.prod.yml
      copy:
        src: ../../docker-compose.prod.yml
        dest: "{{ app_dir }}/docker-compose.prod.yml"
        owner: "{{ deploy_user }}"

    - name: Copy Nginx configuration
      copy:
        src: ../../infra/nginx/nginx.conf
        dest: "{{ app_dir }}/nginx.conf"
        owner: "{{ deploy_user }}"

    # ── Installer Certbot pour HTTPS ──
    - name: Install Certbot
      apt:
        name:
          - certbot
          - python3-certbot-nginx
        state: present

    # ── Configurer le firewall ──
    - name: Allow SSH
      ufw:
        rule: allow
        port: "22"
    - name: Allow HTTP
      ufw:
        rule: allow
        port: "80"
    - name: Allow HTTPS
      ufw:
        rule: allow
        port: "443"
    - name: Enable UFW
      ufw:
        state: enabled
        policy: deny
```

### Diagramme d'Infrastructure Typique

```
  Infrastructure d'un Projet Capstone Typique

  ┌──── Internet ────────────────────────────────────────────────┐
  │                                                              │
  │  Utilisateur ──────► DNS (taskmaster.example.com)            │
  │                          │                                   │
  └──────────────────────────┼───────────────────────────────────┘
                             │
  ┌──── Cloud Provider ──────┼───────────────────────────────────┐
  │                          │                                   │
  │  ┌──── VPC (10.0.0.0/16) ┼──────────────────────────────┐   │
  │  │                       │                               │   │
  │  │  ┌─── Subnet Public ──┼────────────────────────┐      │   │
  │  │  │                    │                        │      │   │
  │  │  │   ┌────────────────▼──────────────────┐     │      │   │
  │  │  │   │        VM (EC2 / t3.small)        │     │      │   │
  │  │  │   │                                   │     │      │   │
  │  │  │   │  ┌──────────────────────────────┐ │     │      │   │
  │  │  │   │  │  Docker Compose              │ │     │      │   │
  │  │  │   │  │                              │ │     │      │   │
  │  │  │   │  │  ┌────────┐  ┌────────────┐  │ │     │      │   │
  │  │  │   │  │  │ Nginx  │─►│  Backend   │  │ │     │      │   │
  │  │  │   │  │  │ :80/443│  │  :3000     │  │ │     │      │   │
  │  │  │   │  │  └────────┘  └─────┬──────┘  │ │     │      │   │
  │  │  │   │  │       │            │         │ │     │      │   │
  │  │  │   │  │  ┌────▼───┐   ┌────▼─────┐  │ │     │      │   │
  │  │  │   │  │  │Frontend│   │  Redis   │  │ │     │      │   │
  │  │  │   │  │  │(static)│   │  :6379   │  │ │     │      │   │
  │  │  │   │  │  └────────┘   └──────────┘  │ │     │      │   │
  │  │  │   │  └──────────────────────────────┘ │     │      │   │
  │  │  │   └───────────────────────────────────┘     │      │   │
  │  │  └─────────────────────────────────────────────┘      │   │
  │  │                                                       │   │
  │  │  ┌─── Subnet Prive ──────────────────────────┐       │   │
  │  │  │                                           │       │   │
  │  │  │   ┌─────────────────────────────────┐     │       │   │
  │  │  │   │  RDS PostgreSQL (db.t3.micro)   │     │       │   │
  │  │  │   │  Port: 5432                     │     │       │   │
  │  │  │   │  Acces: depuis la VM seulement  │     │       │   │
  │  │  │   └─────────────────────────────────┘     │       │   │
  │  │  └───────────────────────────────────────────┘       │   │
  │  └──────────────────────────────────────────────────────┘   │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

> [!tip] Analogie
> L'Infrastructure as Code, c'est comme une **recette de cuisine ecrite** par opposition a "je fais au feeling". Si votre serveur tombe, au lieu de passer 3 heures a tout reinstaller manuellement, vous faites `terraform apply` et `ansible-playbook` : tout est recree a l'identique en 10 minutes.

---

## Containerisation Production-Ready

### Dockerfile Multi-Stage

Un Dockerfile multi-stage permet de separer l'etape de build de l'etape de production :

```dockerfile
# backend/Dockerfile

# ══════════════════════════════════════════
# Etape 1 : Base commune
# ══════════════════════════════════════════
FROM node:20-alpine AS base
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production && npm cache clean --force

# ══════════════════════════════════════════
# Etape 2 : Developpement (avec hot reload)
# ══════════════════════════════════════════
FROM node:20-alpine AS development
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci                 # Toutes les deps (y compris devDependencies)
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]  # Nodemon ou tsx watch

# ══════════════════════════════════════════
# Etape 3 : Build
# ══════════════════════════════════════════
FROM node:20-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build          # Compiler TypeScript → JavaScript

# ══════════════════════════════════════════
# Etape 4 : Production
# ══════════════════════════════════════════
FROM node:20-alpine AS production

# Creer un utilisateur non-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S appuser -u 1001 -G nodejs

WORKDIR /app

# Copier seulement ce qui est necessaire
COPY --from=base /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package.json ./

# Passer a l'utilisateur non-root
USER appuser

# Metadata
LABEL maintainer="equipe-capstone"
LABEL version="1.0.0"

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1

CMD ["node", "dist/server.js"]
```

```
  Comparaison des tailles d'image

  Sans multi-stage :
  ┌──────────────────────────────────────────┐
  │ node:20          (~1.1 GB)               │
  │ + node_modules   (~300 MB dev deps)      │
  │ + code source    (~50 MB)                │
  │ + TypeScript     (inutile en prod)       │
  │ = ~1.5 GB                                │
  └──────────────────────────────────────────┘

  Avec multi-stage :
  ┌──────────────────────────────────────────┐
  │ node:20-alpine   (~180 MB)               │
  │ + node_modules   (~100 MB prod only)     │
  │ + dist/          (~5 MB JS compile)      │
  │ = ~285 MB        (5x plus petit !)       │
  └──────────────────────────────────────────┘
```

### Docker Compose pour la Production

```yaml
# docker-compose.prod.yml

services:
  # ──── Reverse Proxy Nginx ────
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./infra/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certbot/conf:/etc/letsencrypt:ro
      - ./certbot/www:/var/www/certbot:ro
      - frontend_build:/usr/share/nginx/html:ro  # Fichiers statiques frontend
    depends_on:
      backend:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - frontend-net

  # ──── Backend API ────
  backend:
    image: ghcr.io/equipe/taskmaster-backend:latest
    expose:
      - "3000"            # Pas de port public ! Accessible via Nginx seulement
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - JWT_SECRET=${JWT_SECRET}
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "0.5"
    networks:
      - frontend-net
      - backend-net

  # ──── Base de donnees PostgreSQL ────
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 256M
    networks:
      - backend-net

  # ──── Cache Redis ────
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --maxmemory 128mb --maxmemory-policy allkeys-lru
    volumes:
      - redis_data:/data
    restart: unless-stopped
    networks:
      - backend-net

  # ──── Certbot (renouvellement SSL) ────
  certbot:
    image: certbot/certbot
    volumes:
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"

volumes:
  postgres_data:
  redis_data:
  frontend_build:

networks:
  frontend-net:
  backend-net:
```

### Configuration Nginx en Reverse Proxy

```nginx
# infra/nginx/nginx.conf

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    # ── Logging ──
    log_format main '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent"';

    access_log /var/log/nginx/access.log main;
    error_log  /var/log/nginx/error.log warn;

    # ── Performance ──
    sendfile    on;
    tcp_nopush  on;
    gzip        on;
    gzip_types  text/plain text/css application/json application/javascript
                text/xml application/xml text/javascript image/svg+xml;

    # ── Rate limiting ──
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=auth:10m rate=3r/s;

    # ── Security headers ──
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';" always;

    # ── Upstream backend ──
    upstream backend {
        server backend:3000;
    }

    # ── Redirection HTTP → HTTPS ──
    server {
        listen 80;
        server_name taskmaster.example.com;

        # Certbot challenge
        location /.well-known/acme-challenge/ {
            root /var/www/certbot;
        }

        location / {
            return 301 https://$host$request_uri;
        }
    }

    # ── Serveur HTTPS ──
    server {
        listen 443 ssl;
        server_name taskmaster.example.com;

        ssl_certificate     /etc/letsencrypt/live/taskmaster.example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/taskmaster.example.com/privkey.pem;
        ssl_protocols       TLSv1.2 TLSv1.3;

        # ── Frontend (fichiers statiques) ──
        location / {
            root /usr/share/nginx/html;
            try_files $uri $uri/ /index.html;  # SPA : renvoie index.html pour toutes les routes
        }

        # ── API Backend ──
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # ── Auth endpoints (rate limit strict) ──
        location /api/auth/ {
            limit_req zone=auth burst=5 nodelay;
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Checklist d'Optimisation d'Image Docker

> [!example] Checklist avant de deployer une image Docker
> - [ ] Image de base legere (`alpine` ou `slim`)
> - [ ] Multi-stage build (build vs production)
> - [ ] `.dockerignore` correct (exclut `node_modules`, `.git`, `tests`)
> - [ ] Utilisateur non-root (`USER appuser`)
> - [ ] `HEALTHCHECK` defini
> - [ ] Pas de secrets dans l'image (utilisez les variables d'environnement)
> - [ ] Couches ordonnees par frequence de changement (deps avant code)
> - [ ] `npm ci` au lieu de `npm install` (deterministe)
> - [ ] `--only=production` pour exclure les devDependencies

---

## Monitoring et Observabilite en Production

### Les 3 Piliers de l'Observabilite

```
  Les 3 Piliers de l'Observabilite

  ┌───────────────────┐  ┌───────────────────┐  ┌───────────────────┐
  │      LOGS         │  │     METRIQUES     │  │      TRACES       │
  │                   │  │                   │  │                   │
  │  "Que s'est-il    │  │  "Quelle est la   │  │  "Quel est le     │
  │   passe ?"        │  │   sante globale ?"│  │   chemin d'une    │
  │                   │  │                   │  │   requete ?"      │
  │  Evenements       │  │  Valeurs          │  │  Requete suivie   │
  │  textuels avec    │  │  numeriques       │  │  a travers tous   │
  │  timestamp        │  │  au fil du temps  │  │  les services     │
  │                   │  │                   │  │                   │
  │  Outils :         │  │  Outils :         │  │  Outils :         │
  │  - stdout/stderr  │  │  - Prometheus     │  │  - Jaeger         │
  │  - Loki           │  │  - Grafana        │  │  - OpenTelemetry  │
  │  - ELK Stack      │  │  - Datadog        │  │  (optionnel pour  │
  │                   │  │                   │  │   un Capstone)     │
  └───────────────────┘  └───────────────────┘  └───────────────────┘

  Pour un Capstone, concentrez-vous sur LOGS + METRIQUES.
  Les traces sont un bonus si vous avez le temps.
```

### Logs Structures (JSON)

Au lieu de `console.log("Erreur !")`, utilisez des logs structures :

```javascript
// backend/src/middleware/logger.js

import winston from "winston";

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()    // Format JSON pour l'analyse automatique
  ),
  defaultMeta: {
    service: "taskmaster-api",
    version: process.env.APP_VERSION || "unknown",
  },
  transports: [
    new winston.transports.Console(),
    // En production, ajouter un transport fichier ou un service de logs
  ],
});

// Middleware Express pour logger les requetes
export function requestLogger(req, res, next) {
  const start = Date.now();

  res.on("finish", () => {
    const duration = Date.now() - start;
    logger.info("HTTP Request", {
      method: req.method,
      path: req.originalUrl,
      status: res.statusCode,
      duration_ms: duration,
      user_id: req.user?.id || "anonymous",
      ip: req.ip,
    });
  });

  next();
}

export default logger;
```

```
  Sortie du logger en JSON :

  {"level":"info","message":"HTTP Request","method":"GET",
   "path":"/api/tasks","status":200,"duration_ms":42,
   "user_id":"abc-123","ip":"192.168.1.1",
   "service":"taskmaster-api","timestamp":"2026-03-15T10:30:00.000Z"}

  Pourquoi JSON ?
  → Facilement parsable par Loki, ELK, CloudWatch
  → Filtrable (chercher tous les status >= 500)
  → Alertable (alerter si duration_ms > 5000)
```

### Metriques avec Prometheus

```javascript
// backend/src/metrics.js

import promClient from "prom-client";

// Activer les metriques systeme par defaut (CPU, memoire, etc.)
promClient.collectDefaultMetrics({ prefix: "taskmaster_" });

// Metriques personnalisees
export const httpRequestDuration = new promClient.Histogram({
  name: "taskmaster_http_request_duration_seconds",
  help: "Duration of HTTP requests in seconds",
  labelNames: ["method", "route", "status_code"],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],  // en secondes
});

export const httpRequestsTotal = new promClient.Counter({
  name: "taskmaster_http_requests_total",
  help: "Total number of HTTP requests",
  labelNames: ["method", "route", "status_code"],
});

export const activeUsers = new promClient.Gauge({
  name: "taskmaster_active_users",
  help: "Number of currently active users",
});

export const tasksCreated = new promClient.Counter({
  name: "taskmaster_tasks_created_total",
  help: "Total number of tasks created",
});

// Middleware pour collecter les metriques HTTP
export function metricsMiddleware(req, res, next) {
  const end = httpRequestDuration.startTimer();

  res.on("finish", () => {
    const route = req.route ? req.route.path : req.path;
    const labels = {
      method: req.method,
      route: route,
      status_code: res.statusCode,
    };
    end(labels);
    httpRequestsTotal.inc(labels);
  });

  next();
}

// Endpoint /metrics pour Prometheus
export async function metricsEndpoint(req, res) {
  res.set("Content-Type", promClient.register.contentType);
  res.end(await promClient.register.metrics());
}
```

### Grafana Dashboard

Ajoutez Prometheus et Grafana a votre docker-compose de production :

```yaml
  # Ajouter dans docker-compose.prod.yml

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./infra/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    restart: unless-stopped
    networks:
      - backend-net

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
    depends_on:
      - prometheus
    restart: unless-stopped
    networks:
      - backend-net
```

```yaml
# infra/prometheus/prometheus.yml

global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "taskmaster-api"
    static_configs:
      - targets: ["backend:3000"]
    metrics_path: "/api/metrics"

  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]

  - job_name: "postgres-exporter"
    static_configs:
      - targets: ["postgres-exporter:9187"]
```

### Alerting : Quoi Surveiller ?

```
  Metriques critiques a surveiller et seuils d'alerte

  ┌─────────────────────────────────────────────────────────────┐
  │  CRITIQUE (reveillez-vous a 3h du matin)                    │
  │                                                             │
  │  ■ Taux d'erreur HTTP 5xx > 5% pendant 5 minutes           │
  │  ■ Application down (health check echoue)                   │
  │  ■ Disque > 90% plein                                       │
  │  ■ Base de donnees injoignable                              │
  └─────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────┐
  │  WARNING (a traiter dans la journee)                        │
  │                                                             │
  │  ■ Latence P95 > 2 secondes pendant 10 minutes             │
  │  ■ CPU > 80% pendant 15 minutes                            │
  │  ■ Memoire > 85%                                            │
  │  ■ Certificat SSL expire dans < 14 jours                    │
  │  ■ Nombre d'erreurs 4xx inhabituel                          │
  └─────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────┐
  │  INFO (revue hebdomadaire)                                  │
  │                                                             │
  │  ■ Nombre de nouveaux utilisateurs par jour                 │
  │  ■ Temps de reponse moyen                                   │
  │  ■ Taille de la base de donnees                             │
  │  ■ Nombre de deploiements                                   │
  └─────────────────────────────────────────────────────────────┘
```

### Outils Gratuits pour les Etudiants

> [!info] Services gratuits pour le monitoring
> - **UptimeRobot** (gratuit) : verifie que votre site est en ligne toutes les 5 min, alerte par email/SMS
> - **Healthchecks.io** (gratuit) : surveille les cron jobs et taches planifiees
> - **Sentry** (free tier) : capture les erreurs en production avec stack trace, contexte, et breadcrumbs
> - **Grafana Cloud** (free tier) : 10k metriques, 50GB logs, 50GB traces
> - **BetterStack** (free tier) : logs + uptime monitoring

---

## Securite du Projet

### Checklist de Securite Avant la Mise en Production

> [!warning] Checklist obligatoire avant de rendre le projet public
> Cette checklist doit etre validee par toute l'equipe avant le deploiement final.

```
  CHECKLIST SECURITE - A valider avant chaque deploiement

  ── TRANSPORT ──
  [ ] HTTPS actif partout (Let's Encrypt / Certbot)
  [ ] Redirection HTTP → HTTPS automatique
  [ ] HSTS header active

  ── SECRETS ──
  [ ] Aucun secret dans le code source (grep -r "password\|secret\|key" src/)
  [ ] .env.example dans le repo (sans valeurs reelles)
  [ ] .env dans .gitignore
  [ ] Secrets geres via variables d'environnement ou vault
  [ ] Cles API avec permissions minimales

  ── AUTHENTIFICATION ──
  [ ] Mots de passe haches avec bcrypt (cost >= 12)
  [ ] Tokens JWT avec expiration courte (15 min)
  [ ] Refresh tokens avec rotation
  [ ] Rate limiting sur /login et /register (max 5 tentatives/min)
  [ ] Deconnexion qui invalide les tokens

  ── VALIDATION DES ENTREES ──
  [ ] Validation sur TOUS les endpoints (Joi, Zod, Pydantic)
  [ ] Taille maximale des requetes limitee (body-parser limit)
  [ ] Upload de fichiers : verifier type MIME et taille
  [ ] Pas de donnees utilisateur dans les URL (utiliser le body)

  ── INJECTION SQL ──
  [ ] Requetes parametrees PARTOUT (jamais de concatenation)
  [ ] ORM utilise correctement (Prisma, SQLAlchemy, etc.)
  [ ] Pas de raw queries sauf necessite absolue

  ── XSS (Cross-Site Scripting) ──
  [ ] Output encoding sur toutes les donnees affichees
  [ ] Content-Security-Policy header configure
  [ ] React/Vue echappe automatiquement (attention a dangerouslySetInnerHTML !)

  ── DEPENDANCES ──
  [ ] npm audit / pip-audit sans vulnerabilites critiques
  [ ] package-lock.json / requirements.txt versionne
  [ ] Dependances mises a jour (Dependabot active)

  ── CONTENEURS ──
  [ ] Image Docker scannee avec Trivy (0 vulnerabilites critiques)
  [ ] Conteneur tourne en utilisateur non-root
  [ ] Pas de privileges eleves (no --privileged)

  ── DIVERS ──
  [ ] Headers de securite (X-Frame-Options, X-Content-Type-Options)
  [ ] CORS configure correctement (pas Access-Control-Allow-Origin: *)
  [ ] Error messages ne leakent pas d'informations internes
  [ ] Logs ne contiennent pas de donnees sensibles
```

### HTTPS avec Let's Encrypt

```bash
# Obtenir un certificat SSL gratuit avec Certbot

# 1. Installer Certbot
sudo apt install certbot python3-certbot-nginx

# 2. Obtenir le certificat (Nginx doit etre en cours d'execution)
sudo certbot --nginx -d taskmaster.example.com -d www.taskmaster.example.com

# 3. Verifier le renouvellement automatique
sudo certbot renew --dry-run

# Le certificat se renouvelle automatiquement via un cron job
# Validite : 90 jours, renouvellement tous les 60 jours
```

### Prevention des Injections SQL

```javascript
// DANGEREUX - Injection SQL possible !
const query = `SELECT * FROM users WHERE email = '${userInput}'`;
// Si userInput = "' OR '1'='1" → Retourne TOUS les utilisateurs !

// SECURISE - Requete parametree
const result = await db.query(
  "SELECT * FROM users WHERE email = $1",
  [userInput]
);

// SECURISE - Avec un ORM (Prisma)
const user = await prisma.user.findUnique({
  where: { email: userInput },
});

// SECURISE - Avec un ORM (SQLAlchemy / Python)
// user = db.query(User).filter(User.email == user_input).first()
```

### Rate Limiting

```javascript
// backend/src/middleware/rateLimiter.js
import rateLimit from "express-rate-limit";

// Rate limiter general (100 requetes par minute par IP)
export const generalLimiter = rateLimit({
  windowMs: 60 * 1000,    // 1 minute
  max: 100,                // 100 requetes max
  message: {
    error: "Too many requests, please try again later",
  },
  standardHeaders: true,   // Retourne les headers RateLimit-*
});

// Rate limiter strict pour l'authentification
export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,                     // 5 tentatives max
  message: {
    error: "Too many login attempts, please try again in 15 minutes",
  },
  skipSuccessfulRequests: true,  // Ne compte que les echecs
});

// Utilisation dans les routes
// app.use("/api/", generalLimiter);
// app.use("/api/auth/login", authLimiter);
// app.use("/api/auth/register", authLimiter);
```

### Scan de Securite Automatise

```yaml
# Ajout dans le pipeline CI : scan de securite

  security-scan:
    name: "Security Scan"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Scan des dependances avec Trivy
      - name: Trivy filesystem scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: "fs"
          scan-ref: "."
          format: "sarif"
          output: "trivy-results.sarif"
          severity: "CRITICAL,HIGH"

      # Upload des resultats dans GitHub Security
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: "trivy-results.sarif"

      # Scan OWASP ZAP (sur l'environnement de staging)
      - name: OWASP ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.10.0
        with:
          target: "https://staging.taskmaster.example.com"
          rules_file_name: ".zap/rules.tsv"
          cmd_options: "-a"
```

> [!tip] Analogie
> La securite d'une application, c'est comme la securite d'une **maison**. Vous ne vous contentez pas de fermer la porte d'entree : vous verifiez les fenetres, vous n'ecrivez pas le code de l'alarme sur un post-it colle a la porte, vous ne laissez pas la cle sous le paillasson. Chaque couche de securite rend l'attaque plus difficile.

---

## Documentation et Livrable Public

### Documentation API (OpenAPI / Swagger)

La documentation API est auto-generee a partir de votre code :

```javascript
// backend/src/swagger.js
import swaggerJsdoc from "swagger-jsdoc";
import swaggerUi from "swagger-ui-express";

const options = {
  definition: {
    openapi: "3.0.0",
    info: {
      title: "TaskMaster API",
      version: "1.0.0",
      description: "API de gestion de taches collaborative",
      contact: {
        name: "Equipe TaskMaster",
        url: "https://github.com/equipe/taskmaster",
      },
    },
    servers: [
      { url: "http://localhost:3000", description: "Developpement" },
      { url: "https://staging.taskmaster.example.com", description: "Staging" },
      { url: "https://taskmaster.example.com", description: "Production" },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: "http",
          scheme: "bearer",
          bearerFormat: "JWT",
        },
      },
    },
  },
  apis: ["./src/routes/*.js"],  // Fichiers contenant les annotations JSDoc
};

const specs = swaggerJsdoc(options);

export function setupSwagger(app) {
  app.use("/api/docs", swaggerUi.serve, swaggerUi.setup(specs));
}
```

```javascript
// backend/src/routes/tasks.js - Annotations Swagger dans les routes

/**
 * @openapi
 * /api/tasks:
 *   get:
 *     summary: Lister les taches de l'utilisateur connecte
 *     tags: [Tasks]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: status
 *         schema:
 *           type: string
 *           enum: [todo, in_progress, done]
 *         description: Filtrer par statut
 *     responses:
 *       200:
 *         description: Liste des taches
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/Task'
 *       401:
 *         description: Non authentifie
 */
router.get("/", auth, taskController.list);
```

### Documentation d'Architecture (Modele C4)

Le modele C4 propose 4 niveaux de zoom sur l'architecture :

```
  Modele C4 - Niveau 1 : Diagramme de Contexte
  (Vue d'ensemble : qui utilise le systeme ?)

                     ┌──────────────┐
                     │  Utilisateur │
                     │  (personne)  │
                     └──────┬───────┘
                            │ Utilise
                            ▼
                     ┌──────────────┐
                     │  TaskMaster  │
                     │  (systeme)   │──── Envoie des emails ──── [Sendgrid]
                     └──────┬───────┘
                            │ Authentifie via
                            ▼
                     ┌──────────────┐
                     │   Google     │
                     │   OAuth2     │
                     └──────────────┘


  Modele C4 - Niveau 2 : Diagramme de Conteneurs
  (Quels sont les composants techniques ?)

  ┌─────────────────────────────────────────────────────────────┐
  │                     TaskMaster                               │
  │                                                             │
  │  ┌─────────────┐    JSON     ┌──────────────┐              │
  │  │   Frontend  │────────────►│  Backend API │              │
  │  │   React SPA │◄────────────│  Node.js     │              │
  │  │   :443      │             │  :3000       │              │
  │  └─────────────┘             └──────┬───────┘              │
  │                                     │                      │
  │                          ┌──────────┼──────────┐           │
  │                          │          │          │           │
  │                     ┌────▼───┐ ┌────▼───┐ ┌────▼────┐     │
  │                     │Postgres│ │ Redis  │ │  S3     │     │
  │                     │  :5432 │ │ :6379  │ │ (files) │     │
  │                     └────────┘ └────────┘ └─────────┘     │
  └─────────────────────────────────────────────────────────────┘


  Modele C4 - Niveau 3 : Diagramme de Composants
  (Que contient le Backend ?)

  ┌─────────────────── Backend API ────────────────────────────┐
  │                                                            │
  │  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐ │
  │  │  Auth Router   │  │  Task Router   │  │ Project Router│ │
  │  │  /api/auth/*   │  │  /api/tasks/*  │  │ /api/projects│ │
  │  └───────┬────────┘  └───────┬────────┘  └──────┬───────┘ │
  │          │                   │                   │         │
  │  ┌───────▼───────────────────▼───────────────────▼───────┐ │
  │  │              Middleware Layer                          │ │
  │  │  Auth │ Validation │ RateLimit │ Logger │ Metrics     │ │
  │  └───────────────────────┬───────────────────────────────┘ │
  │                          │                                 │
  │  ┌───────────────────────▼───────────────────────────────┐ │
  │  │              Service Layer (Business Logic)           │ │
  │  │  AuthService │ TaskService │ ProjectService │ ...     │ │
  │  └───────────────────────┬───────────────────────────────┘ │
  │                          │                                 │
  │  ┌───────────────────────▼───────────────────────────────┐ │
  │  │              Data Access Layer (Prisma ORM)           │ │
  │  └───────────────────────────────────────────────────────┘ │
  └────────────────────────────────────────────────────────────┘
```

### Rendre le Repository GitHub Professionnel

> [!example] Elements d'un repository professionnel
> 1. **README.md** avec badges CI/CD, screenshots, et instructions claires
> 2. **LICENSE** (MIT pour les projets etudiants)
> 3. **CONTRIBUTING.md** (comment contribuer)
> 4. **.github/** avec templates d'issues et de PR
> 5. **Tags et Releases** (v1.0.0, v1.1.0) avec changelogs
> 6. **GitHub Pages** pour la documentation
> 7. **Description et topics** du repository remplis

Badges a ajouter dans le README :

```markdown
<!-- Badges professionnels pour le README -->
![CI Status](https://github.com/equipe/taskmaster/actions/workflows/ci.yml/badge.svg)
![CD Status](https://github.com/equipe/taskmaster/actions/workflows/cd.yml/badge.svg)
![Coverage](https://codecov.io/gh/equipe/taskmaster/branch/main/graph/badge.svg)
![License](https://img.shields.io/github/license/equipe/taskmaster)
![Last Commit](https://img.shields.io/github/last-commit/equipe/taskmaster)
```

### Conseils pour la Presentation

```
  Structure d'une demo de Capstone (15-20 minutes)

  ┌─────────────────────────────────────────────────┐
  │  1. Introduction (2 min)                         │
  │     - Qui sommes-nous ?                          │
  │     - Quel probleme resolvons-nous ?             │
  │                                                  │
  │  2. Demo Live (5-7 min)                          │
  │     - Parcours utilisateur complet               │
  │     - Feature la plus impressionnante            │
  │     - AVOIR UN PLAN B (video de backup !)        │
  │                                                  │
  │  3. Architecture & DevOps (5-7 min)              │
  │     - Schema d'architecture                      │
  │     - Pipeline CI/CD (montrer un vrai run)       │
  │     - Monitoring (montrer Grafana)               │
  │     - Infrastructure (montrer Terraform)         │
  │                                                  │
  │  4. Retour d'experience (2-3 min)                │
  │     - Ce qui a bien marche                       │
  │     - Ce qui a ete difficile                     │
  │     - Ce qu'on ferait differemment               │
  │                                                  │
  │  5. Questions (5 min)                            │
  └─────────────────────────────────────────────────┘
```

> [!warning] Regles d'or pour la demo
> - **Toujours avoir une video de backup** en cas de panne internet/serveur
> - **Preparer un jeu de donnees** de demo (pas un site vide)
> - **Tester la demo la veille** dans les conditions reelles (meme salle, meme connexion)
> - **Ne pas montrer du code** ligne par ligne (montrez des schemas)
> - **Chaque membre parle** (le jury evalue le travail d'equipe)

---

## Gestion de Projet en Equipe

### Agile / Scrum pour Equipes Etudiantes

Le framework Scrum, adapte pour une equipe etudiante :

```
  Scrum Adapte pour un Capstone de 6 Semaines

  Sprint 1    Sprint 2    Sprint 3    Sprint 4    Sprint 5    Sprint 6
  (1 sem)     (1 sem)     (1 sem)     (1 sem)     (1 sem)     (1 sem)
  ┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐
  │Setup│     │Core │     │Core │     │Integ│     │Deploy│    │Polish│
  │Arch │     │Feat.│     │Feat.│     │Test │     │Monit.│    │Demo │
  │CI   │     │Back │     │Front│     │Docker│    │Secu. │    │Docs │
  └──┬──┘     └──┬──┘     └──┬──┘     └──┬──┘     └──┬──┘     └──┬──┘
     │           │           │           │           │           │
     ▼           ▼           ▼           ▼           ▼           ▼
  [Review]   [Review]   [Review]   [Review]   [Review]   [DEMO]
  [Retro]    [Retro]    [Retro]    [Retro]    [Retro]    [FINAL]

  ── Ceremonies par sprint (adaptees etudiants) ──

  ■ Sprint Planning (debut de sprint) : 30 min
    → Quels tickets on prend cette semaine ?

  ■ Daily Standup : 10 min (3x par semaine suffit)
    → Qu'est-ce que j'ai fait ? Qu'est-ce que je fais ? Suis-je bloque ?

  ■ Sprint Review (fin de sprint) : 30 min
    → Demo de ce qui a ete fait cette semaine

  ■ Sprint Retrospective (fin de sprint) : 20 min
    → Qu'est-ce qui a bien marche / mal marche / a ameliorer ?
```

### GitHub Projects (Kanban)

Utilisez **GitHub Projects** pour gerer vos taches (gratuit et integre) :

```
  Tableau Kanban GitHub Projects

  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
  │  BACKLOG    │  │  TO DO      │  │  IN PROGRESS│  │    DONE     │
  │             │  │  (Sprint)   │  │             │  │             │
  │ #12 Export  │  │ #8 Page     │  │ #5 API      │  │ #1 Setup    │
  │     CSV     │  │   login     │  │   tasks     │  │   repo ✓   │
  │             │  │   @bob      │  │   @alice    │  │             │
  │ #13 Notifs  │  │             │  │             │  │ #2 Docker   │
  │     email   │  │ #9 Dashboard│  │ #6 Tests    │  │   compose ✓│
  │             │  │   @charlie  │  │   unitaires │  │             │
  │ #14 Search  │  │             │  │   @charlie  │  │ #3 CI pipe- │
  │             │  │ #10 CI/CD   │  │             │  │   line ✓   │
  │ #15 Dark    │  │   pipeline  │  │ #7 Auth JWT │  │             │
  │     mode    │  │   @alice    │  │   @alice    │  │ #4 Schema  │
  │             │  │             │  │             │  │   DB ✓     │
  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘

  Chaque carte = une Issue GitHub avec :
  - Titre clair
  - Description detaillee
  - Label (feature, bug, devops, docs)
  - Assignee (qui travaille dessus)
  - Milestone (Sprint 1, Sprint 2, ...)
  - Estimation (taille : S, M, L, XL)
```

### Processus de Code Review

```
  Processus de Pull Request et Code Review

  Developpeur                    Reviewer                    CI/CD
      │                             │                          │
      │  1. Cree une branche        │                          │
      │     feature/add-login       │                          │
      │                             │                          │
      │  2. Code + commits          │                          │
      │                             │                          │
      │  3. Push + ouvre une PR      │                          │
      │─────────────────────────────►│                          │
      │                             │                          │
      │                             │    4. CI s'execute        │
      │                             │◄─────────────────────────│
      │                             │    ✓ Lint OK             │
      │                             │    ✓ Tests OK            │
      │                             │    ✓ Build OK            │
      │                             │                          │
      │                             │  5. Review du code       │
      │                             │     - Lisibilite ?       │
      │                             │     - Tests suffisants ? │
      │                             │     - Securite OK ?      │
      │                             │     - Performance ?      │
      │                             │                          │
      │  6. Corrige les commentaires│                          │
      │◄─────────────────────────────│                          │
      │                             │                          │
      │  7. Push les corrections    │                          │
      │─────────────────────────────►│                          │
      │                             │                          │
      │                             │  8. Approve              │
      │                             │─────────────────────────►│
      │                             │                          │
      │         9. Merge dans main  │    10. CD deploie        │
      │◄────────────────────────────────────────────────────────│
```

> [!info] Regles de code review constructive
> - **Critiquer le code, pas la personne** : "Cette fonction pourrait etre simplifiee" et non "Tu as mal code"
> - **Expliquer le pourquoi** : "Utilise `const` ici car la variable n'est pas reassignee (principe du moindre privilege)"
> - **Proposer des alternatives** : Ne dites pas juste "c'est pas bien", proposez une solution
> - **Feliciter les bonnes pratiques** : "Excellent choix d'utiliser un DTO ici"
> - **Ne pas bloquer pour des details** : Les discussions de style (tabs vs spaces) ne sont pas des blockers

### Communication en Equipe

```
  Outils de communication recommandes

  ┌───────────────────────────────────────────────────────────┐
  │  Discord / Slack (gratuit)                                │
  │                                                           │
  │  Channels a creer :                                       │
  │  #general      → Discussions generales                    │
  │  #dev          → Questions techniques                     │
  │  #devops       → Pipelines, deploiement, infra            │
  │  #daily        → Standups asynchrones                     │
  │  #alerts       → Notifications CI/CD, monitoring          │
  │  #random       → Fun, memes, decompression                │
  │                                                           │
  │  Integrations :                                           │
  │  - GitHub → Notifications de PR et issues                 │
  │  - CI/CD → Resultats des pipelines                        │
  │  - Monitoring → Alertes de production                     │
  └───────────────────────────────────────────────────────────┘
```

### Resolution de Conflits

> [!warning] Conflits courants et comment les resoudre
> | Conflit | Solution |
> |---|---|
> | "Il ne fait rien" | → Daily standups transparents, tickets assignes visibles par tous |
> | "Il a modifie mon code sans me le dire" | → Regles de code ownership, branches protegees, PR obligatoires |
> | "On n'est pas d'accord sur la techno" | → ADR (vote + arguments ecrits), le chef de projet tranche |
> | "Il ne repond jamais" | → Accord sur les temps de reponse max (ex: 24h), escalade au tuteur |
> | "Le code de X est illisible" | → Linting automatique, conventions de code ecrites, pre-commit hooks |

### Post-Mortem

A faire apres le projet (utile pour votre portfolio et votre progression) :

```markdown
# Post-Mortem - Projet TaskMaster

## Ce qui a bien marche
- Le pipeline CI/CD etait en place des la semaine 1 → 0 bug en production
- Le docker-compose de dev a elimine les "ca marche sur ma machine"
- Les daily standups (meme 3x/semaine) ont evite les blocages

## Ce qui a ete difficile
- La configuration Nginx + SSL a pris 2 jours (sous-estime)
- Le merge de grosses PR a cause des conflits → solution : PR plus petites
- Un membre de l'equipe a ete malade la semaine 4 → redistribuer les taches

## Ce qu'on ferait differemment
- Commencer par l'API (backend) avant le frontend
- Ecrire les tests en meme temps que le code (pas apres)
- Utiliser un ORM des le debut (au lieu de raw SQL refactorise ensuite)

## Metriques du projet
- 147 commits, 32 PR mergees, 15 issues fermees
- 78% de couverture de tests
- 99.5% d'uptime sur les 2 dernieres semaines
- Temps de deploiement moyen : 4 minutes
```

---

## Timeline Suggeree (6 semaines)

```
  Planning du Projet Capstone sur 6 Semaines

  ═══════════════════════════════════════════════════════════════════

  SEMAINE 1 : FONDATIONS
  ───────────────────────
  Lun │ Choix du projet, formation des equipes, brainstorm
  Mar │ Definition du MVP, user stories, choix de la stack
  Mer │ Setup du repo Git, branch protection, PR templates
  Jeu │ Docker Compose pour le dev, schema de base de donnees
  Ven │ Pipeline CI (lint + tests), premiers ADR

  Livrable : Repo configure, CI qui tourne, architecture documentee


  SEMAINE 2 : BACKEND
  ────────────────────
  Lun │ API d'authentification (register, login, JWT)
  Mar │ CRUD principal (la feature coeur du projet)
  Mer │ Tests unitaires pour chaque endpoint
  Jeu │ Validation des entrees, gestion des erreurs
  Ven │ Documentation API (Swagger), code review

  Livrable : API fonctionnelle testee et documentee


  SEMAINE 3 : FRONTEND
  ─────────────────────
  Lun │ Setup du frontend, routing, layout
  Mar │ Pages d'authentification (login, register)
  Mer │ Pages principales (dashboard, CRUD)
  Jeu │ Integration avec l'API backend
  Ven │ Tests frontend, responsive design

  Livrable : Application complete fonctionnelle en local


  SEMAINE 4 : INTEGRATION & DOCKER PROD
  ───────────────────────────────────────
  Lun │ Dockerfiles multi-stage (backend + frontend)
  Mar │ docker-compose.prod.yml (Nginx, SSL)
  Mer │ Tests d'integration (API + frontend ensemble)
  Jeu │ Correction des bugs, optimisation Docker
  Ven │ Pipeline CD (staging deployment)

  Livrable : Application dockerisee, pipeline CD vers staging


  SEMAINE 5 : CLOUD & SECURITE
  ─────────────────────────────
  Lun │ Terraform : provisionner l'infrastructure cloud
  Mar │ Deploiement en production
  Mer │ Setup monitoring (Prometheus + Grafana)
  Jeu │ Audit de securite (checklist + scans)
  Ven │ Alerting, uptime monitoring

  Livrable : Application en production, monitoree et securisee


  SEMAINE 6 : POLISH & DEMO
  ──────────────────────────
  Lun │ Correction des derniers bugs
  Mar │ Documentation finale (README, architecture, deploiement)
  Mer │ Preparation de la demo, slides
  Jeu │ Repetition de la demo, video de backup
  Ven │ DEMO FINALE 🎓

  Livrable : Demo reussie, documentation complete, repo propre

  ═══════════════════════════════════════════════════════════════════
```

> [!warning] Points de vigilance
> - **Semaine 1** : ne pas coder immediatement ! La planification sauve des semaines entires plus tard
> - **Semaine 3** : si le backend n'est pas fini, le frontend peut travailler avec des mocks (donnees fausses)
> - **Semaine 4** : c'est la semaine la plus critique — c'est la que "ca marche en local" doit devenir "ca marche en Docker"
> - **Semaine 6** : ZERO nouvelle feature cette semaine ! Seulement du bug fix, de la doc et de la prep demo

---

## Carte Mentale

```
                              PROJET CAPSTONE
                                    │
            ┌───────────────────────┼───────────────────────┐
            │                       │                       │
       DEFINIR                  CONSTRUIRE              DEPLOYER
            │                       │                       │
    ┌───────┼───────┐       ┌───────┼───────┐       ┌───────┼───────┐
    │       │       │       │       │       │       │       │       │
   MVP    Stack   ADR    Backend Frontend Docker  CI/CD  Cloud  Monitoring
    │       │       │       │       │       │       │       │       │
  MoSCoW  React  Docs    API     React  Multi-  GitHub  Terraform Prometheus
  User    Node   Choix   REST    Vue    stage   Actions AWS/GCP  Grafana
  Stories Python Tech    Auth    Next   Nginx   Secrets VM/ECS   Alerting
          DB              JWT    Tests  Compose Matrix  Ansible  Logs JSON
                          Tests        Prod    Cache
                          SQL                  CD
                                               Staging
            │                       │                       │
            │               SECURITE & DOC              EQUIPE
            │                       │                       │
            │               ┌───────┼───────┐       ┌───────┼───────┐
            │               │       │       │       │       │       │
            │             HTTPS   OWASP  Swagger  Scrum  Kanban   Code
            │             Secrets  Scan   C4     Sprints GitHub  Review
            │             Validation Trivy README Daily  Projects  PR
            │             Rate     ZAP   Demo   Retro  Issues   Post-
            │             Limit                                  Mortem
```

---

## Exercices

### Exercice 1 : Definition du Projet (individuel)

Choisissez un sujet de projet et redigez :
1. Un **pitch** de 3 phrases (quoi, pour qui, pourquoi)
2. **5 user stories** au format standard
3. Un tableau **MoSCoW** avec au moins 3 items par categorie
4. Le choix de la **stack technique** justifie (framework, DB, cloud)
5. Un **ADR** pour votre choix de base de donnees

### Exercice 2 : Setup Complet du Repository

Creez un repository GitHub avec :
1. Branch protection sur `main` (au moins 1 review requise)
2. Template de PR et template d'issue (bug report)
3. Un `docker-compose.yml` de developpement fonctionnel (backend + DB + Redis)
4. Des pre-commit hooks (au moins : trailing whitespace, detect-secrets, linting)
5. Un README avec badges, description, et instructions d'installation
6. Un `.gitignore` correct et un fichier `LICENSE`

### Exercice 3 : Pipeline CI/CD

Implementez un pipeline GitHub Actions complet :
1. Job `lint` : linting et verification du formatage
2. Job `test` : tests unitaires avec un service PostgreSQL
3. Job `build` : build de l'image Docker avec cache
4. Job `security` : scan Trivy sur le code source
5. Verifiez que le pipeline s'execute correctement sur une PR

### Exercice 4 : Containerisation Production

Ecrivez :
1. Un **Dockerfile multi-stage** pour votre backend (development + build + production)
2. Un **docker-compose.prod.yml** avec Nginx, backend, PostgreSQL, Redis
3. Une **configuration Nginx** avec reverse proxy, SSL (simule), et rate limiting
4. Verifiez que `docker compose -f docker-compose.prod.yml up` fonctionne

### Exercice 5 : Infrastructure as Code

Ecrivez la configuration Terraform pour :
1. Un VPC avec un subnet public
2. Un security group (ports 80, 443, 22)
3. Une instance EC2 avec Docker installe (user_data)
4. Executez `terraform plan` et analysez la sortie

### Exercice 6 : Monitoring

Ajoutez a votre application :
1. Des **logs structures** (JSON) avec Winston ou equivalent
2. Un endpoint `/api/metrics` pour Prometheus
3. Au moins 3 metriques personnalisees (requetes/s, latence, erreurs)
4. Configurez **Prometheus + Grafana** dans Docker Compose
5. Creez un dashboard Grafana avec au moins 4 panneaux

### Exercice 7 : Audit de Securite

Realisez un audit complet de votre projet :
1. Completez la **checklist de securite** de ce cours (chaque point coche ou justifie)
2. Executez `npm audit` et corrigez les vulnerabilites critiques
3. Scannez votre image Docker avec **Trivy** et corrigez les problemes
4. Verifiez qu'aucun secret n'est dans l'historique Git (`git log --all -p | grep -i password`)
5. Testez le rate limiting sur vos endpoints d'authentification

### Exercice 8 : Projet Capstone Complet (equipe)

En equipe de 3 a 5 personnes, realisez un projet Capstone complet :
1. **Semaine 1** : Definition du projet, setup, CI
2. **Semaine 2-3** : Developpement des features principales
3. **Semaine 4** : Integration, Docker production, CD
4. **Semaine 5** : Deploiement cloud, monitoring, securite
5. **Semaine 6** : Documentation, polish, demo

Livrables attendus :
- [ ] Application fonctionnelle deployee publiquement
- [ ] Pipeline CI/CD automatise (build, test, deploy)
- [ ] Documentation (README, API, architecture, ADR)
- [ ] Monitoring en place (metriques + alertes)
- [ ] Audit de securite realise
- [ ] Presentation de 15-20 minutes avec demo live

---

## Liens

- [[04 - CI-CD avec GitHub Actions]] — Pipeline d'integration et deploiement continus
- [[05 - Infrastructure as Code Terraform]] — Provisionner l'infrastructure avec Terraform
- [[06 - Automatisation avec Ansible]] — Configuration automatisee des serveurs
- [[07 - DevSecOps]] — Securite integree dans le pipeline
- [[03 - Docker Compose en Pratique]] — Orchestration multi-conteneurs
- [[../Cloud/03 - Deploiement Cloud et Conteneurs]] — Deployer sur le cloud
- [[../Observabilite/02 - Metriques et Monitoring]] — Prometheus, Grafana, alerting
- [[../Secutrity/02 - Securite Web OWASP]] — Vulnerabilites web courantes et protections
