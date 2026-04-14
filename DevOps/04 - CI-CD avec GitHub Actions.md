# CI/CD avec GitHub Actions

## Qu'est-ce que le CI/CD ?

Le **CI/CD** est un ensemble de pratiques qui automatisent le processus de livraison logicielle, du code source jusqu'au deploiement en production.

> [!tip] Analogie
> Imagine une **usine automobile** :
> - **CI (Integration Continue)** = la **chaine de montage** : chaque piece ajoutee est immediatement testee et assemblee avec les autres. Si une piece est defectueuse, on le sait tout de suite.
> - **CD (Livraison Continue)** = la **voiture est prete a etre livree** a tout moment, il suffit d'appuyer sur un bouton.
> - **CD (Deploiement Continu)** = la voiture **sort automatiquement de l'usine** des qu'elle passe tous les controles qualite.
>
> Sans CI/CD, c'est comme assembler toutes les pieces a la fin et esperer que ca marche. Spoiler : ca ne marche jamais du premier coup.

---

### Les 3 composantes du CI/CD

| Terme | Signification | Ce que ca fait |
|---|---|---|
| **CI** - Continuous Integration | Integration Continue | Chaque `push` declenche automatiquement : build + tests + verification |
| **CD** - Continuous Delivery | Livraison Continue | Le code est **toujours pret** a etre deploye (mais deploiement manuel) |
| **CD** - Continuous Deployment | Deploiement Continu | Le code est **automatiquement deploye** en production apres tous les checks |

> [!info] Delivery vs Deployment
> - **Delivery** : le colis est pret au depot, quelqu'un doit appuyer sur "Envoyer"
> - **Deployment** : le colis part automatiquement des qu'il est emballe

---

## Le Pipeline CI/CD

Un **pipeline** est une serie d'etapes automatisees que le code traverse avant d'arriver en production.

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│          │    │          │    │          │    │          │    │          │
│   PUSH   │───>│  BUILD   │───>│   TEST   │───>│ STAGING  │───>│  DEPLOY  │
│          │    │          │    │          │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
     |               |               |               |               |
  git push      Compiler le     Tests unitaires   Environnement   Mise en
  / merge       code, installer  Tests integration de pre-prod    production
                dependances      Linting, coverage                 
                                                                   
   ──────────── CI ──────────────────────── CD ──────────────────────>
```

> [!warning] Si une etape echoue, le pipeline S'ARRETE
> C'est le principe fondamental : on ne deploie **jamais** du code qui n'a pas passe tous les tests. Le pipeline agit comme un **garde-fou automatique**.

---

## Pourquoi utiliser le CI/CD ?

### Sans CI/CD (l'enfer)

```
Developpeur A : "Ca marche sur ma machine!"
Developpeur B : "J'ai oublie de lancer les tests avant de push..."
Chef de projet : "Le site est down parce qu'on a deploye un bug vendredi soir..."
```

### Avec CI/CD (le paradis)

| Avantage | Explication |
|---|---|
| **Detection precoce des bugs** | Les tests tournent a chaque push, pas 3 jours avant la release |
| **Automatisation** | Plus de "j'ai oublie de lancer les tests" |
| **Deployments consistants** | Meme processus a chaque fois, pas de "j'ai oublie une etape" |
| **Confiance** | Si le pipeline passe, le code est pret |
| **Feedback rapide** | Le developpeur sait en quelques minutes si son code casse quelque chose |
| **Documentation vivante** | Le pipeline documente le processus de build et deploy |

---

## GitHub Actions : les bases

**GitHub Actions** est la plateforme CI/CD integree a GitHub. Elle est **gratuite** pour les projets publics et offre un genereux quota pour les projets prives.

### Les concepts fondamentaux

```
┌───────────────────────────────────────────────────────────────┐
│                        WORKFLOW                                │
│  (fichier YAML dans .github/workflows/)                       │
│                                                                │
│  Declenche par un EVENT (push, PR, schedule...)               │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                      JOB 1                               │  │
│  │  Tourne sur un RUNNER (ubuntu-latest, windows...)        │  │
│  │                                                          │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │  │
│  │  │  Step 1  │─>│  Step 2  │─>│  Step 3  │              │  │
│  │  │ checkout │  │ install  │  │  test    │              │  │
│  │  └──────────┘  └──────────┘  └──────────┘              │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                      JOB 2                               │  │
│  │  (peut tourner en parallele ou apres Job 1)              │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

| Concept | Definition | Analogie |
|---|---|---|
| **Workflow** | Un processus automatise complet (fichier YAML) | La recette complete |
| **Event** | Ce qui declenche le workflow | "Quand le four sonne..." |
| **Job** | Un ensemble de steps qui tournent sur le meme runner | Une etape de la recette (preparer la pate) |
| **Step** | Une action individuelle dans un job | Une instruction (ajouter la farine) |
| **Runner** | La machine qui execute le job | La cuisine ou on travaille |
| **Action** | Un composant reutilisable (pre-fait) | Un robot de cuisine |

---

## La structure des fichiers

```
mon-projet/
├── .github/
│   └── workflows/
│       ├── ci.yml            # Pipeline d'integration continue
│       ├── cd.yml            # Pipeline de deploiement
│       └── tests.yml         # Pipeline de tests specifiques
├── src/
├── tests/
└── README.md
```

> [!warning] Emplacement obligatoire
> Les fichiers de workflow DOIVENT etre dans `.github/workflows/`. GitHub ne reconnaitra pas les workflows places ailleurs. Le nom du fichier est libre, mais il doit se terminer par `.yml` ou `.yaml`.

---

## Rappel syntaxe YAML

Le YAML est le format utilise pour ecrire les workflows GitHub Actions.

```yaml
# Cle simple
nom: "valeur"

# Liste
fruits:
  - pomme
  - banane
  - cerise

# Dictionnaire imbrique
personne:
  nom: "Alice"
  age: 30
  langages:
    - Python
    - JavaScript

# Booleens
actif: true
debug: false

# Multi-ligne
description: |
  Ceci est un texte
  sur plusieurs lignes.
```

> [!warning] L'indentation YAML
> - Utiliser des **espaces** (jamais de tabulations)
> - L'indentation definit la **hierarchie** (generalement 2 espaces)
> - Une erreur d'indentation = un workflow qui ne marche pas
> - Utiliser un linter YAML pour verifier la syntaxe

---

## Anatomie d'un workflow

Voici un workflow minimal avec chaque section expliquee :

```yaml
# Nom du workflow (affiche dans l'onglet Actions de GitHub)
name: CI Pipeline

# Evenement declencheur
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

# Les jobs a executer
jobs:
  # Nom du job (identifiant libre)
  build-and-test:
    # Machine sur laquelle tourner
    runs-on: ubuntu-latest

    # Les etapes du job
    steps:
      # Etape 1 : recuperer le code
      - name: Checkout du code
        uses: actions/checkout@v4

      # Etape 2 : commande shell
      - name: Afficher un message
        run: echo "Le CI fonctionne!"

      # Etape 3 : commande multi-lignes
      - name: Informations systeme
        run: |
          echo "Systeme: $(uname -a)"
          echo "Repertoire: $(pwd)"
          ls -la
```

> [!info] `uses` vs `run`
> - `uses` : utilise une **action pre-faite** du marketplace (ex: `actions/checkout@v4`)
> - `run` : execute une **commande shell** directement
> - Un step utilise soit `uses` soit `run`, jamais les deux

---

## Les declencheurs (triggers)

### Les evenements les plus courants

```yaml
on:
  # Declenchement sur un push
  push:
    branches: [ main, develop ]
    paths:
      - 'src/**'            # Seulement si des fichiers dans src/ changent
      - '!src/**/*.md'      # Mais pas les fichiers markdown
    tags:
      - 'v*'                # Sur les tags commencant par v (v1.0, v2.3.1)

  # Declenchement sur une Pull Request
  pull_request:
    branches: [ main ]
    types: [ opened, synchronize, reopened ]

  # Declenchement programme (cron)
  schedule:
    - cron: '0 6 * * 1'    # Tous les lundis a 6h UTC
    #        | | | | |
    #        | | | | └── Jour de la semaine (0=dim, 1=lun, ..., 6=sam)
    #        | | | └──── Mois (1-12)
    #        | | └────── Jour du mois (1-31)
    #        | └──────── Heure (0-23)
    #        └────────── Minute (0-59)

  # Declenchement manuel depuis l'interface GitHub
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environnement cible'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
```

> [!example] Cas d'usage typiques
> - `push` sur `main` : deployer en production
> - `pull_request` : lancer les tests avant le merge
> - `schedule` : lancer des tests de securite chaque nuit
> - `workflow_dispatch` : deployer manuellement une version specifique

### Filtrer avec `paths`

```yaml
on:
  push:
    paths:
      - 'backend/**'    # Declenche seulement si le backend change
      - 'Dockerfile'
    paths-ignore:
      - '**.md'          # Ignore les changements dans les fichiers markdown
      - 'docs/**'        # Ignore le dossier docs
```

> [!tip] Analogie
> Les `paths` sont comme un **filtre a cafe** : seuls les fichiers qui correspondent passent et declenchent le workflow. Ca evite de relancer les tests du backend quand tu modifies juste le README.

---

## Les runners

Un **runner** est la machine virtuelle qui execute ton job.

| Runner | OS | Usage typique |
|---|---|---|
| `ubuntu-latest` | Ubuntu 22.04+ | Le plus courant, rapide et complet |
| `ubuntu-22.04` | Ubuntu 22.04 (fixe) | Quand tu veux une version precise |
| `windows-latest` | Windows Server 2022 | Projets Windows / .NET |
| `macos-latest` | macOS Monterey+ | Projets iOS / macOS |
| `self-hosted` | Ta propre machine | Besoins specifiques / performance |

```yaml
jobs:
  test-linux:
    runs-on: ubuntu-latest    # Runner GitHub heberge (gratuit pour public)

  test-windows:
    runs-on: windows-latest

  test-custom:
    runs-on: self-hosted      # Ta propre machine configuree comme runner
```

> [!info] Quotas GitHub Actions (repos prives)
> - **Linux** : 2 000 minutes/mois gratuites
> - **Windows** : consomme 2x les minutes
> - **macOS** : consomme 10x les minutes
> - **Repos publics** : illimite et gratuit

---

## Les steps en detail

### Utiliser une action (uses)

```yaml
steps:
  # Action officielle pour checkout le code
  - name: Checkout
    uses: actions/checkout@v4

  # Action avec parametres
  - name: Setup Python
    uses: actions/setup-python@v5
    with:
      python-version: '3.11'

  # Action tierce
  - name: Envoyer notification Slack
    uses: slackapi/slack-github-action@v1.25.0
    with:
      slack-message: "Build reussi!"
    env:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Executer des commandes (run)

```yaml
steps:
  # Commande simple
  - name: Verifier la version Python
    run: python --version

  # Commandes multiples
  - name: Installer et tester
    run: |
      pip install -r requirements.txt
      pip install pytest
      pytest tests/ -v

  # Specifier un shell different
  - name: Script PowerShell
    run: Write-Host "Hello from PowerShell"
    shell: pwsh
```

---

## Actions Marketplace : les indispensables

Le **GitHub Actions Marketplace** contient des milliers d'actions reutilisables.

| Action | Usage | Exemple |
|---|---|---|
| `actions/checkout@v4` | Recuperer le code du repo | **Indispensable** dans chaque workflow |
| `actions/setup-python@v5` | Installer Python | `with: python-version: '3.11'` |
| `actions/setup-node@v4` | Installer Node.js | `with: node-version: '20'` |
| `actions/cache@v4` | Mettre en cache des fichiers | Dependencies pip, npm, etc. |
| `actions/upload-artifact@v4` | Sauvegarder des fichiers | Rapports de tests, builds |
| `actions/download-artifact@v4` | Recuperer des artefacts | Entre jobs ou workflows |

> [!warning] Toujours epingler la version
> Utilise `actions/checkout@v4` (tag) plutot que `actions/checkout@main` (branche). Les branches peuvent changer a tout moment et casser ton pipeline.

---

## Variables d'environnement et secrets

### Variables d'environnement

```yaml
# Niveau workflow (disponible partout)
env:
  APP_NAME: "mon-app"
  NODE_ENV: "test"

jobs:
  build:
    runs-on: ubuntu-latest
    # Niveau job
    env:
      DATABASE_URL: "sqlite:///test.db"

    steps:
      - name: Utiliser les variables
        # Niveau step
        env:
          STEP_VAR: "valeur locale"
        run: |
          echo "App: $APP_NAME"
          echo "DB: $DATABASE_URL"
          echo "Step: $STEP_VAR"
```

### Secrets

Les **secrets** sont des valeurs sensibles (tokens, mots de passe) stockees de maniere chiffree dans les parametres du repository.

```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
    run: |
      echo "Deploiement avec le token..."
      # Les secrets sont masques dans les logs (affiches comme ***)
```

> [!warning] Securite des secrets
> - Les secrets ne sont **jamais affiches** dans les logs (remplaces par `***`)
> - Ils ne sont **pas disponibles** dans les workflows declenches par des forks (securite)
> - `secrets.GITHUB_TOKEN` est automatiquement fourni par GitHub avec des permissions limitees au repo
> - Ne **jamais** mettre de secrets en dur dans le code ou le YAML

### Le token GITHUB_TOKEN

```yaml
steps:
  - name: Creer une release
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    run: |
      gh release create v1.0.0 --title "Version 1.0.0" --notes "Premiere release"
```

> [!info] `secrets.GITHUB_TOKEN`
> Ce token est **automatiquement cree** par GitHub pour chaque execution de workflow. Il a des permissions limitees au repository courant. Pas besoin de le configurer manuellement.

---

## Workflow CI complet : Python

```yaml
name: Python CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest

    steps:
      # 1. Recuperer le code
      - name: Checkout du code
        uses: actions/checkout@v4

      # 2. Installer Python
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      # 3. Cache des dependances pip
      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements*.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-

      # 4. Installer les dependances
      - name: Installer les dependances
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt

      # 5. Linting avec flake8
      - name: Linting (flake8)
        run: |
          pip install flake8
          flake8 src/ --count --show-source --statistics
          flake8 src/ --count --exit-zero --max-complexity=10 --max-line-length=88

      # 6. Verification des types avec mypy
      - name: Type checking (mypy)
        run: |
          pip install mypy
          mypy src/ --ignore-missing-imports

      # 7. Tests unitaires avec couverture
      - name: Tests avec couverture
        run: |
          pip install pytest pytest-cov
          pytest tests/ -v --cov=src/ --cov-report=xml --cov-report=term-missing

      # 8. Upload du rapport de couverture
      - name: Upload couverture
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage.xml
```

---

## Workflow CI complet : C

```yaml
name: C CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout du code
        uses: actions/checkout@v4

      # Installer les outils necessaires
      - name: Installer les dependances
        run: |
          sudo apt-get update
          sudo apt-get install -y gcc make valgrind

      # Compiler le projet
      - name: Compilation
        run: |
          gcc -Wall -Werror -Wextra -pedantic -std=c99 \
            -o mon_programme src/*.c
          echo "Compilation reussie!"

      # Compiler et executer les tests
      - name: Compiler les tests
        run: |
          gcc -Wall -Werror -Wextra -pedantic -std=c99 \
            -o run_tests tests/*.c src/*.c -lcriterion
          echo "Tests compiles!"

      - name: Executer les tests
        run: ./run_tests --verbose

      # Verifier les fuites memoire avec Valgrind
      - name: Valgrind - Detection fuites memoire
        run: |
          valgrind --leak-check=full \
                   --show-leak-kinds=all \
                   --error-exitcode=1 \
                   ./mon_programme
          echo "Pas de fuite memoire detectee!"

      # Verification du style (Betty pour Holberton)
      - name: Betty style check
        run: |
          git clone https://github.com/hs-hq/Betty.git
          cd Betty && sudo ./install.sh && cd ..
          betty src/*.c src/*.h
```

---

## Workflow CI complet : JavaScript

```yaml
name: JavaScript CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  lint-test-build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout du code
        uses: actions/checkout@v4

      # Installer Node.js
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'   # Cache integre dans l'action !

      # Installer les dependances
      - name: Installer les dependances
        run: npm ci     # npm ci est plus rapide et strict que npm install

      # Linting
      - name: Linting (ESLint)
        run: npx eslint src/ --ext .js,.jsx,.ts,.tsx

      # Formatting check
      - name: Verifier le formatage (Prettier)
        run: npx prettier --check "src/**/*.{js,jsx,ts,tsx,json,css}"

      # Tests
      - name: Tests unitaires
        run: npm test -- --coverage --watchAll=false

      # Build
      - name: Build du projet
        run: npm run build

      # Upload du build
      - name: Upload du build
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: build/
          retention-days: 7
```

---

## Strategie de matrice (Matrix Strategy)

La **matrice** permet de tester ton code sur **plusieurs configurations** en parallele.

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ ubuntu-latest, windows-latest, macos-latest ]
        python-version: [ '3.9', '3.10', '3.11', '3.12' ]
        exclude:
          - os: macos-latest
            python-version: '3.9'    # Exclure une combinaison specifique
        include:
          - os: ubuntu-latest
            python-version: '3.12'
            experimental: true        # Ajouter une variable custom

    steps:
      - uses: actions/checkout@v4

      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Afficher la config
        run: |
          echo "OS: ${{ matrix.os }}"
          echo "Python: ${{ matrix.python-version }}"
          python --version
```

```
Execution en parallele :
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│ ubuntu / Python 3.9 │  │ ubuntu / Python 3.10│  │ ubuntu / Python 3.11│
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│ windows / Python 3.9│  │ windows / Python 3.10│ │ windows / Python 3.11│
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘
┌─────────────────────┐  ┌─────────────────────┐
│ macos / Python 3.10 │  │ macos / Python 3.11  │  (macos/3.9 exclu)
└─────────────────────┘  └─────────────────────┘
```

> [!info] `fail-fast`
> Par defaut, `fail-fast: true` : si un job de la matrice echoue, tous les autres sont annules. Mets `fail-fast: false` pour laisser tous les jobs finir meme si un echoue.

---

## Artefacts (Artifacts)

Les **artefacts** permettent de sauvegarder et partager des fichiers entre jobs ou pour consultation ulterieure.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: |
          mkdir -p dist
          echo "Mon application buildee" > dist/app.js

      # Sauvegarder l'artefact
      - name: Upload de l'artefact
        uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: dist/
          retention-days: 30    # Garder pendant 30 jours

  deploy:
    needs: build    # Attend que le job 'build' finisse
    runs-on: ubuntu-latest
    steps:
      # Recuperer l'artefact du job precedent
      - name: Download de l'artefact
        uses: actions/download-artifact@v4
        with:
          name: build-files
          path: dist/

      - name: Deployer
        run: |
          echo "Deploiement des fichiers:"
          ls -la dist/
```

---

## Cache des dependances

Le **cache** evite de re-telecharger les dependances a chaque execution.

### Cache pip (Python)

```yaml
- name: Cache pip
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

### Cache npm (JavaScript)

```yaml
- name: Cache node_modules
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-npm-
```

> [!tip] Analogie
> Le cache, c'est comme garder les **ingredients** deja achetes dans ton frigo au lieu d'aller au supermarche a chaque repas. La cle de cache (`key`) c'est la **liste de courses** : si elle n'a pas change, les ingredients sont encore bons.

### Comment ca marche

```
Premier run :
  1. Cherche le cache avec la cle → PAS TROUVE (cache miss)
  2. Installe les dependances normalement (lent)
  3. Sauvegarde le cache pour la prochaine fois

Runs suivants (si requirements n'a pas change) :
  1. Cherche le cache avec la cle → TROUVE (cache hit)
  2. Restaure les dependances depuis le cache (rapide!)
  3. Pas besoin de re-telecharger
```

---

## CD : Deployer sur GitHub Pages

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install et Build
        run: |
          npm ci
          npm run build

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## CD : Deployer une image Docker

```yaml
name: Build and Push Docker Image

on:
  push:
    tags:
      - 'v*'    # Declenche seulement sur les tags de version

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Extraire les metadonnees
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: monuser/mon-app
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build et Push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

---

## Dependances entre jobs (needs)

Par defaut, les jobs s'executent en **parallele**. Utilise `needs` pour creer des dependances.

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test

  build:
    needs: [ lint, test ]    # Attend que lint ET test finissent
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build

  deploy:
    needs: build              # Attend que build finisse
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'    # Seulement sur main
    steps:
      - run: echo "Deploiement en production..."
```

```
         ┌──────┐
         │ lint │──────┐
         └──────┘      │
                       v
                  ┌──────────┐     ┌──────────┐
                  │  build   │────>│  deploy  │
                  └──────────┘     └──────────┘
         ┌──────┐      ^           (seulement
         │ test │──────┘            sur main)
         └──────┘
```

---

## Steps conditionnels (if)

```yaml
steps:
  # Executer seulement sur la branche main
  - name: Deploy
    if: github.ref == 'refs/heads/main'
    run: echo "Deploiement..."

  # Executer seulement sur une Pull Request
  - name: Commentaire PR
    if: github.event_name == 'pull_request'
    run: echo "C'est une PR!"

  # Executer seulement si le step precedent a echoue
  - name: Notification d'erreur
    if: failure()
    run: echo "Le build a echoue!"

  # Toujours executer (meme si un step precedent a echoue)
  - name: Nettoyage
    if: always()
    run: echo "Nettoyage..."

  # Condition sur une variable d'environnement
  - name: Build release
    if: startsWith(github.ref, 'refs/tags/v')
    run: echo "Build de la release..."

  # Condition combinee
  - name: Deploy production
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    run: echo "Deploy en prod!"
```

> [!info] Contextes disponibles dans `if`
> - `github.ref` : la branche ou le tag
> - `github.event_name` : push, pull_request, etc.
> - `github.actor` : l'utilisateur qui a declenche
> - `success()` : si les steps precedents ont reussi
> - `failure()` : si un step precedent a echoue
> - `always()` : execute quoi qu'il arrive
> - `cancelled()` : si le workflow a ete annule

---

## Workflows reutilisables

Tu peux creer un workflow reutilisable et l'appeler depuis d'autres workflows.

### Workflow reutilisable (le modele)

```yaml
# .github/workflows/reusable-test.yml
name: Reusable Test Workflow

on:
  workflow_call:     # Ce workflow est appelable par d'autres
    inputs:
      python-version:
        required: true
        type: string
      run-coverage:
        required: false
        type: boolean
        default: true
    secrets:
      CODECOV_TOKEN:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
      - run: |
          pip install -r requirements.txt
          pytest tests/ -v
      - name: Coverage
        if: ${{ inputs.run-coverage }}
        run: pytest tests/ --cov=src/ --cov-report=xml
```

### Appeler le workflow reutilisable

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main ]

jobs:
  call-tests:
    uses: ./.github/workflows/reusable-test.yml
    with:
      python-version: '3.11'
      run-coverage: true
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

---

## Branch Protection Rules

Les **regles de protection de branche** s'integrent avec le CI/CD pour empecher le merge de code qui ne passe pas les tests.

### Configuration dans GitHub

```
Settings → Branches → Add rule

Branch name pattern : main

[x] Require a pull request before merging
    [x] Require approvals (1)
[x] Require status checks to pass before merging
    [x] Require branches to be up to date
    Status checks : lint-and-test    ← le nom de ton job CI
[x] Require conversation resolution before merging
[ ] Include administrators
```

> [!warning] Pourquoi c'est essentiel
> Sans protection de branche, quelqu'un peut `git push --force` sur `main` et deployer du code qui casse tout. Avec les regles, **personne** ne peut merger sans que le CI passe. C'est le **filet de securite** de ton projet.

```
                     ┌─────────────┐
                     │  feature/x  │
                     └──────┬──────┘
                            │
                         PR │
                            v
                ┌───────────────────────┐
                │   CHECKS AUTOMATIQUES  │
                │                        │
                │  [x] CI passe          │
                │  [x] Review approuvee  │
                │  [x] Branche a jour    │
                │                        │
                │  Tous OK ? → MERGE     │
                │  Sinon   ? → BLOQUE    │
                └───────────┬───────────┘
                            │
                            v
                     ┌─────────────┐
                     │    main     │
                     └─────────────┘
```

---

## Recapitulatif : Pipeline CI/CD complet

Voici un workflow de production realiste qui combine tout ce qu'on a vu :

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  PYTHON_VERSION: '3.11'

jobs:
  # ──────── LINT ────────
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      - run: |
          pip install flake8 black isort mypy
          flake8 src/
          black --check src/
          isort --check-only src/
          mypy src/

  # ──────── TEST ────────
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [ '3.10', '3.11', '3.12' ]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements*.txt') }}
      - run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
          pytest tests/ -v --cov=src/ --cov-report=xml
      - uses: actions/upload-artifact@v4
        with:
          name: coverage-${{ matrix.python-version }}
          path: coverage.xml

  # ──────── BUILD ────────
  build:
    needs: [ lint, test ]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          docker build -t mon-app:${{ github.sha }} .
          docker save mon-app:${{ github.sha }} > image.tar
      - uses: actions/upload-artifact@v4
        with:
          name: docker-image
          path: image.tar

  # ──────── DEPLOY STAGING ────────
  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: staging
    steps:
      - run: echo "Deploiement en staging..."

  # ──────── DEPLOY PRODUCTION ────────
  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    steps:
      - run: echo "Deploiement en production..."
```

---

## Carte Mentale

```
                            CI/CD avec GitHub Actions
                                      |
                 ┌────────────────────┼────────────────────┐
                 |                    |                     |
            CI (Integration)    CD (Livraison)      GitHub Actions
                 |                    |                     |
          ┌──────┼──────┐      ┌─────┼─────┐       ┌──────┼──────┐
          |      |      |      |     |     |       |      |      |
        Build  Test   Lint   Staging Prod  Pages Workflows Jobs  Steps
                                                    |      |      |
                                              Triggers  Runners  Actions
                                                |        |        |
                                          ┌─────┼────┐   |   Marketplace
                                          |     |    |   |
                                        push   PR schedule
                                                          |
                                                   ┌──────┼──────┐
                                                   |      |      |
                                                Matrix  Cache  Artifacts
                                                   |
                                              Multi-version
```

---

## Exercices

### Exercice 1 : Premier workflow CI

Cree un workflow GitHub Actions qui :
1. Se declenche sur les `push` et `pull_request` vers `main`
2. Utilise `ubuntu-latest`
3. Checkout le code
4. Installe Python 3.11
5. Installe les dependances depuis `requirements.txt`
6. Lance `flake8` pour le linting
7. Lance `pytest` pour les tests

### Exercice 2 : Matrice multi-versions

Modifie l'exercice 1 pour :
1. Tester avec Python 3.10, 3.11 et 3.12
2. Tester sur Ubuntu et Windows
3. Exclure la combinaison Windows + Python 3.10
4. Ajouter le cache des dependances pip

### Exercice 3 : Pipeline complet avec deploiement

Cree un pipeline complet avec :
1. Un job `lint` (flake8 + black --check)
2. Un job `test` (pytest avec couverture)
3. Un job `build` qui depend de `lint` et `test`
4. Un job `deploy` qui depend de `build` et ne s'execute que sur `main`
5. Upload de la couverture en artefact

### Exercice 4 : Workflow JavaScript

Cree un workflow CI pour un projet JavaScript qui :
1. Installe Node.js 20 avec cache npm
2. Lance ESLint et Prettier en mode check
3. Execute les tests avec couverture
4. Build le projet
5. Upload le dossier `build/` en artefact

---

## Liens

- [[01 - Git et GitHub]] - Les bases de Git necessaires pour comprendre les events push/PR
- [[03 - Docker Compose en Pratique]] - Combiner Docker et CI/CD pour des deployments containerises
- [[02 - Tests Integration et E2E]] - Les tests qui tournent dans le pipeline CI