# DevSecOps

## Qu'est-ce que le DevSecOps ?

Le **DevSecOps** est l'integration de la **securite** a chaque etape du cycle de developpement logiciel. Au lieu de traiter la securite comme une etape finale (un audit avant la mise en production), on l'integre **des le debut** et **en continu**.

```
  AVANT : Securite "boulonnee" a la fin
  
  Plan ──> Code ──> Build ──> Test ──> Deploy ──> [SECURITE] ──> Production
                                                      │
                                          "On a trouve 47 vulnerabilites"
                                          "Il faut tout refaire"
                                          "Le lancement est reporte de 3 mois"


  APRES : DevSecOps - Securite integree partout ("Shift Left")
  
  ┌────────────────────────────────────────────────────────────────┐
  │                        SECURITE                                │
  │                                                                │
  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐ │
  │  │ PLAN │─>│ CODE │─>│BUILD │─>│ TEST │─>│DEPLOY│─>│OPERATE│ │
  │  │      │  │      │  │      │  │      │  │      │  │      │ │
  │  │Threat│  │Secure│  │ SAST │  │ DAST │  │Image │  │Runtime│ │
  │  │Model │  │Coding│  │      │  │      │  │ Scan │  │Protect│ │
  │  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘ │
  │                                                                │
  │  ◄──── Shift Left : trouver les problemes LE PLUS TOT ────►  │
  │                                                                │
  └────────────────────────────────────────────────────────────────┘
```

> [!tip] Analogie
> Imaginez la construction d'une **maison** :
> - **Ancienne methode** : construire toute la maison, puis faire venir un expert securite qui dit "les fondations ne sont pas aux normes, il faut tout casser et recommencer"
> - **DevSecOps** : l'expert securite est present **a chaque etape** : il verifie les fondations, puis les murs, puis l'electricite, au fur et a mesure
>
> Plus un probleme est detecte **tot**, moins il coute cher a corriger. Un bug en conception coute 100x moins a corriger qu'un bug en production.

> [!info] Le concept de "Shift Left"
> "Shift Left" signifie deplacer les controles de securite vers la **gauche** du pipeline (vers le debut). Au lieu de tester la securite a la fin, on la teste des le code. Le cout de correction d'une vulnerabilite augmente de facon exponentielle a chaque etape :
> - En conception : 1x
> - En code : 5x
> - En test : 10x
> - En production : 100x

---

## Securite a Chaque Phase

### Phase 1 : Plan (Modelisation des Menaces)

Avant d'ecrire une seule ligne de code, identifiez les menaces potentielles :

**Threat Modeling (Modelisation des menaces)** :
- **Quels sont les actifs a proteger ?** (donnees utilisateurs, cles API, transactions)
- **Qui pourrait attaquer ?** (hackers, employes malveillants, bots)
- **Comment pourraient-ils attaquer ?** (injection SQL, XSS, brute force)
- **Quelles sont les consequences ?** (fuite de donnees, perte financiere)

La methode **STRIDE** est un framework populaire :

| Menace | Description | Exemple |
|---|---|---|
| **S**poofing | Usurpation d'identite | Voler un token JWT |
| **T**ampering | Modification non autorisee | Modifier le prix dans le panier |
| **R**epudiation | Nier une action | "Je n'ai jamais passe cette commande" |
| **I**nformation Disclosure | Fuite de donnees | API qui expose les mots de passe |
| **D**enial of Service | Rendre indisponible | Flood de requetes |
| **E**levation of Privilege | Obtenir plus de droits | Utilisateur qui devient admin |

**Exigences de securite** a documenter :
- Authentification requise pour quels endpoints
- Chiffrement des donnees sensibles (au repos et en transit)
- Politique de mots de passe
- Duree de vie des tokens
- Journalisation des actions sensibles

---

### Phase 2 : Code (Pratiques de Codage Securise)

#### Plugins de securite IDE

Les problemes de securite se detectent des l'editeur de code :

```
  Votre IDE (VS Code, PyCharm, IntelliJ)
  ┌──────────────────────────────────────┐
  │                                      │
  │  password = "admin123"               │
  │  ~~~~~~~~~~~~~~────── ⚠ Hardcoded   │
  │                        password      │
  │                        detected!     │
  │                                      │
  │  query = f"SELECT * FROM users       │
  │           WHERE id = {user_input}"   │
  │  ~~~~~~────────────── ⚠ SQL         │
  │                        Injection     │
  │                        risk!         │
  │                                      │
  └──────────────────────────────────────┘
```

Plugins recommandes :
- **SonarLint** : detecte bugs et vulnerabilites en temps reel
- **Snyk** : analyse les dependances dans l'IDE
- **GitLens + GitGuardian** : detecte les secrets dans le code

#### Pre-commit Hooks pour les Secrets

Les **pre-commit hooks** empechent de committer des secrets dans le depot Git :

```bash
# Installation de pre-commit
pip install pre-commit

# .pre-commit-config.yaml
repos:
  # Detecter les secrets avant le commit
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']

  # Autre outil : gitleaks
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks

  # Trufflehog : recherche d'entropie elevee (cles aleatoires)
  - repo: https://github.com/trufflesecurity/trufflehog
    rev: v3.63.0
    hooks:
      - id: trufflehog
```

```bash
# Installer les hooks
pre-commit install

# Test : essayer de committer un secret
echo 'AWS_SECRET_KEY = "AKIAIOSFODNN7EXAMPLE"' >> config.py
git add config.py
git commit -m "add config"
# RESULTAT :
# detect-secrets...................................................Failed
# ERROR: Potential secret detected in config.py:1
# Commit BLOQUE ! Le secret ne part jamais dans Git.
```

> [!warning] Les secrets dans l'historique Git sont PERMANENTS
> Si un secret a ete committe meme une seule fois, il est dans l'historique Git **pour toujours** (meme apres suppression). Il faut considerer ce secret comme **compromis** et le revoquer immediatement. Des outils comme `git-filter-repo` peuvent nettoyer l'historique, mais c'est dangereux et complexe.

---

### Phase 3 : Build (SAST - Static Application Security Testing)

Le **SAST** analyse le **code source** (sans l'executer) pour trouver des vulnerabilites. C'est comme un correcteur orthographique, mais pour la securite.

```
  CODE SOURCE                  SAST SCANNER                RAPPORT
  ┌──────────┐               ┌──────────────┐           ┌──────────┐
  │          │               │              │           │          │
  │ app.py   │──────────────>│  Analyse     │──────────>│ 3 HIGH   │
  │ utils.py │  Code source  │  statique    │  Resultats│ 7 MEDIUM │
  │ auth.py  │  (pas besoin  │  du code     │           │ 12 LOW   │
  │ ...      │  d'executer)  │              │           │          │
  └──────────┘               └──────────────┘           └──────────┘
```

#### Outils SAST

**Bandit** (Python) :

```bash
# Installation
pip install bandit

# Scanner un projet
bandit -r ./src -f json -o bandit-report.json

# Exemple de detection :
# >> Issue: [B608:hardcoded_sql_expressions]
# >> Severity: Medium   Confidence: Medium
# >> Location: ./src/db.py:15
# >> query = "SELECT * FROM users WHERE id = " + user_id
```

```python
# MAUVAIS : Bandit detecte ca
import subprocess
subprocess.call(user_input, shell=True)       # B602: subprocess_popen_with_shell_equals_true
password = "super_secret_123"                  # B105: hardcoded_password_string
query = "SELECT * FROM users WHERE id=" + uid  # B608: hardcoded_sql_expressions

# BON : corrige
import subprocess
subprocess.call(["/usr/bin/ls", "-la"], shell=False)
password = os.environ.get("DB_PASSWORD")
query = "SELECT * FROM users WHERE id = %s"   # Requete parametree
cursor.execute(query, (uid,))
```

**Semgrep** (multi-langages) :

```bash
# Installation
pip install semgrep

# Scanner avec les regles par defaut
semgrep --config=auto ./src

# Scanner avec des regles specifiques
semgrep --config=p/python ./src
semgrep --config=p/owasp-top-ten ./src
semgrep --config=p/security-audit ./src
```

**SonarQube** (plateforme complete) :

```bash
# Lancer SonarQube avec Docker
docker run -d --name sonarqube -p 9000:9000 sonarqube:community

# Scanner un projet (apres configuration)
sonar-scanner \
  -Dsonar.projectKey=my-project \
  -Dsonar.sources=./src \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=token-here
```

| Outil | Langages | Gratuit | Points forts |
|---|---|---|---|
| **Bandit** | Python | Oui | Simple, rapide, specifique Python |
| **ESLint security** | JavaScript | Oui | Integre a l'ecosysteme JS |
| **Semgrep** | Multi (20+) | Oui (regles de base) | Regles personnalisables, rapide |
| **SonarQube** | Multi (30+) | Community edition | Plateforme complete, quality gates |

---

### Phase 4 : Test (DAST - Dynamic Application Security Testing)

Le **DAST** teste l'application **en cours d'execution**. Il simule des attaques reelles en envoyant des requetes malveillantes.

```
  DAST SCANNER                APPLICATION EN EXECUTION
  ┌──────────────┐           ┌──────────────┐
  │              │  Requetes  │              │
  │  Simule des  │──────────>│  Application │
  │  attaques    │  malveil-  │  deployee    │
  │  reelles     │  lantes    │  (staging)   │
  │              │           │              │
  │  - SQL inj.  │<──────────│  Reponses    │
  │  - XSS       │  Analyse   │              │
  │  - CSRF      │  les       │              │
  │  - Auth      │  reponses  │              │
  └──────────────┘           └──────────────┘
```

**OWASP ZAP** (Zed Attack Proxy) :

```bash
# Scanner rapide avec Docker
docker run --rm -t owasp/zap2docker-stable zap-baseline.py \
    -t https://staging.myapp.com \
    -r zap-report.html

# Scanner complet (plus long, plus approfondi)
docker run --rm -t owasp/zap2docker-stable zap-full-scan.py \
    -t https://staging.myapp.com \
    -r zap-full-report.html

# Scanner API (avec specification OpenAPI)
docker run --rm -t owasp/zap2docker-stable zap-api-scan.py \
    -t https://staging.myapp.com/openapi.json \
    -f openapi \
    -r zap-api-report.html
```

> [!info] SAST vs DAST
> | Aspect | SAST | DAST |
> |---|---|---|
> | **Quand** | Pendant le build (code source) | Apres le deploy (application running) |
> | **Quoi** | Analyse le code sans l'executer | Teste l'application en execution |
> | **Avantage** | Trouve les bugs tot, rapide | Trouve les vrais problemes exploitables |
> | **Inconvenient** | Faux positifs | Plus lent, necessite un environnement |
> | **Analogie** | Relire un plan de maison | Tester les serrures de la maison construite |

---

### Phase 5 : Deploy (Securite des Conteneurs)

#### Scanner les images Docker

Les images Docker contiennent un systeme d'exploitation et des librairies qui peuvent avoir des **vulnerabilites connues** (CVE).

```bash
# TRIVY : scanner d'images le plus populaire
# Installation
brew install trivy  # ou apt/choco/scoop

# Scanner une image
trivy image my-app:latest

# Resultat :
# my-app:latest (debian 12.4)
# Total: 23 (HIGH: 5, CRITICAL: 2)
#
# ┌───────────────┬──────────────┬──────────┬─────────────────┐
# │   Library     │ Vulnerability │ Severity │  Fixed Version  │
# ├───────────────┼──────────────┼──────────┼─────────────────┤
# │ libssl3       │ CVE-2024-XXX │ CRITICAL │ 3.0.13-1        │
# │ libcurl4      │ CVE-2024-YYY │ HIGH     │ 7.88.1-10+deb12 │
# └───────────────┴──────────────┴──────────┴─────────────────┘

# Scanner et bloquer si vulnerabilites critiques
trivy image --exit-code 1 --severity CRITICAL my-app:latest
# Exit code 1 = le pipeline CI/CD s'arrete

# Autres outils populaires :
# Grype (Anchore)
grype my-app:latest

# Snyk
snyk container test my-app:latest
```

#### Bonnes pratiques pour les images Docker

```dockerfile
# MAUVAIS : image vulnerable
FROM python:3.12
COPY . /app
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
# Problemes : image enorme, root, pas de scan

# BON : image securisee
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

FROM python:3.12-slim
# Utilisateur non-root
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app
COPY --from=builder /root/.local /home/appuser/.local
COPY --chown=appuser:appuser . .

# Pas root
USER appuser

# Systeme de fichiers en lecture seule (au runtime)
# docker run --read-only my-app:latest

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:8000/health || exit 1

ENV PATH=/home/appuser/.local/bin:$PATH
CMD ["python", "app.py"]
```

#### Admission Controllers

Les **admission controllers** Kubernetes bloquent le deploiement d'images non conformes :

```yaml
# Politique : interdire les conteneurs root
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  runAsUser:
    rule: MustRunAsNonRoot
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
```

---

### Phase 6 : Operate (Securite Runtime)

La securite ne s'arrete pas au deploiement. En production, il faut :

- **RASP** (Runtime Application Self-Protection) : l'application se protege elle-meme a l'execution (detecte et bloque les attaques en temps reel)
- **Monitoring de securite** : detecter les comportements anormaux
- **Network policies** : limiter les communications entre services
- **Secrets rotation** : changer regulierement les secrets

### Phase 7 : Monitor (Reponse aux Incidents)

```
  DETECTION          REPONSE             POST-MORTEM
  ┌──────────┐      ┌──────────┐        ┌──────────────┐
  │ Alertes  │─────>│ Contenir │───────>│ Analyser     │
  │ securite │      │ l'impact │        │ la cause     │
  │ (SIEM)   │      │          │        │ racine       │
  └──────────┘      │ Isoler   │        │              │
                    │ les      │        │ Documenter   │
  ┌──────────┐      │ systemes │        │ les          │
  │ Audit    │─────>│ affectes │        │ ameliorations│
  │ logs     │      │          │        │              │
  └──────────┘      └──────────┘        └──────────────┘
```

---

## Gestion des Secrets

### Le probleme

```python
# NE FAITES JAMAIS CA :

# Dans le code source
DATABASE_URL = "postgres://admin:P@ssw0rd!@db.prod.company.com:5432/maindb"
API_KEY = "sk-proj-abcdef123456789"
AWS_SECRET = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"

# Dans docker-compose.yml (versionne dans Git)
environment:
  - DB_PASSWORD=super_secret_123

# Dans un .env committe dans Git
# .env
SECRET_KEY=my-secret-key-12345
```

> [!warning] Pourquoi c'est grave ?
> - Le code source est visible par **tous les developpeurs** (et potentiellement public sur GitHub)
> - L'historique Git conserve **tout pour toujours** : meme si vous supprimez le secret, il reste dans l'historique
> - Les bots scannent GitHub en permanence : un secret AWS pousse sur un repo public est exploite en **moins de 5 minutes**
> - Cout reel : des entreprises ont recu des factures AWS de **dizaines de milliers d'euros** a cause de cles exposees

### Solutions

#### 1. Variables d'environnement (minimum viable)

```bash
# Exporter les variables (pas dans Git)
export DATABASE_URL="postgres://admin:secret@db:5432/mydb"
export API_KEY="sk-proj-abcdef123456789"

# Dans le code Python
import os
db_url = os.environ["DATABASE_URL"]
api_key = os.environ.get("API_KEY", "")  # Valeur par defaut si absent
```

#### 2. Fichiers .env (avec .gitignore)

```bash
# .env (NE JAMAIS COMMITTER)
DATABASE_URL=postgres://admin:secret@db:5432/mydb
API_KEY=sk-proj-abcdef123456789

# .gitignore (TOUJOURS avoir cette ligne)
.env
.env.*
!.env.example
```

```bash
# .env.example (a committer : template sans valeurs reelles)
DATABASE_URL=postgres://user:password@host:5432/dbname
API_KEY=your-api-key-here
```

#### 3. GitHub Secrets (pour CI/CD)

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          # Les secrets sont masques dans les logs
          echo "Deploying with secure credentials..."
```

#### 4. HashiCorp Vault (solution enterprise)

```python
# Recuperer un secret depuis Vault
import hvac

client = hvac.Client(url='https://vault.company.com:8200')
client.token = os.environ['VAULT_TOKEN']

# Lire un secret
secret = client.secrets.kv.v2.read_secret_version(
    path='myapp/production'
)
db_password = secret['data']['data']['db_password']

# Avantages de Vault :
# - Secrets dynamiques (generes a la demande, avec expiration)
# - Rotation automatique
# - Audit log (qui a accede a quel secret, quand)
# - Chiffrement as a service
```

#### 5. SOPS (Secrets OPerationS)

```bash
# Chiffrer un fichier de secrets avec SOPS + age/GPG
sops --encrypt --age age1... secrets.yaml > secrets.enc.yaml

# Le fichier chiffre PEUT etre committe dans Git
# Seules les personnes avec la cle peuvent le dechiffrer
sops --decrypt secrets.enc.yaml
```

| Solution | Complexite | Securite | Usage |
|---|---|---|---|
| **Variables d'env** | Faible | Moyenne | Dev local, petits projets |
| **.env + .gitignore** | Faible | Moyenne | Dev local |
| **GitHub Secrets** | Moyenne | Bonne | CI/CD |
| **Vault** | Elevee | Excellente | Production, entreprise |
| **SOPS** | Moyenne | Bonne | Secrets dans Git (chiffres) |
| **AWS Secrets Manager** | Moyenne | Excellente | Ecosysteme AWS |

---

## Scan des Dependances

### Le probleme : attaques de la supply chain

Votre application depend de **centaines de librairies**, qui elles-memes dependent d'autres librairies. Une seule vulnerabilite dans une dependance transitive expose toute votre application.

```
  VOTRE APPLICATION
       │
       ├── flask==3.0.0
       │     ├── werkzeug==3.0.1
       │     ├── jinja2==3.1.3
       │     │     └── markupsafe==2.1.4
       │     └── click==8.1.7
       ├── requests==2.31.0
       │     ├── urllib3==2.1.0   <── VULNERABILITE CVE-2024-XXXX !
       │     ├── charset-normalizer
       │     └── certifi
       └── sqlalchemy==2.0.25
             └── ...

  Vous n'avez pas ecrit urllib3.
  Vous ne savez peut-etre meme pas que vous l'utilisez.
  Mais si urllib3 a une faille, VOTRE application est vulnerable.
```

### Outils de scan

```bash
# PIP-AUDIT (Python) - recommande
pip install pip-audit
pip-audit
# Found 2 known vulnerabilities in 1 package
# Name    Version  ID              Fix Versions
# ------- -------- --------------- -------- 
# urllib3 2.1.0    CVE-2024-XXXX   2.1.1

# SAFETY (Python)
pip install safety
safety check
# ou scanner un fichier requirements
safety check -r requirements.txt

# NPM AUDIT (JavaScript)
npm audit
# 3 vulnerabilities (1 moderate, 2 high)
npm audit fix  # Corrige automatiquement si possible

# SNYK (multi-langages)
snyk test
```

### Dependabot et Renovate

**Dependabot** (GitHub natif) et **Renovate** creent automatiquement des **Pull Requests** pour mettre a jour les dependances vulnerables :

```yaml
# .github/dependabot.yml
version: 2
updates:
  # Python
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    reviewers:
      - "security-team"
    labels:
      - "dependencies"
      - "security"

  # Docker
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

> [!tip] Analogie
> Les dependances, c'est comme les **ingredients d'un restaurant**. Vous ne fabriquez pas votre farine, vos oeufs, votre beurre. Vous faites confiance a vos **fournisseurs**. Mais si un fournisseur livre des oeufs contamines, c'est **votre restaurant** qui rend les clients malades. Le scan de dependances, c'est le **controle qualite a la reception des marchandises**.

### Lock files : verrouillez vos dependances

```bash
# TOUJOURS committer les lock files dans Git !
# Ils garantissent que tout le monde installe les MEMES versions

# Python
pip freeze > requirements.txt          # Version simple
poetry.lock                            # Avec Poetry
Pipfile.lock                           # Avec Pipenv

# JavaScript  
package-lock.json                      # npm
yarn.lock                              # Yarn
pnpm-lock.yaml                        # pnpm

# Go
go.sum

# Rust
Cargo.lock
```

> [!warning] Sans lock file
> Sans lock file, `pip install requests` pourrait installer la version 2.31.0 sur votre machine et 2.32.0 sur le serveur CI. Si la 2.32.0 a un bug, le CI passe mais la production casse. Le lock file garantit la **reproductibilite**.

---

## Securite des Conteneurs (Recapitulatif)

```
  CHECKLIST SECURITE CONTENEURS
  
  ┌─────────────────────────────────────────────────────────┐
  │                                                         │
  │  [x] Image de base minimale (alpine, slim, distroless) │
  │  [x] Utilisateur non-root (USER appuser)               │
  │  [x] Systeme de fichiers read-only (--read-only)       │
  │  [x] Scanner les vulnerabilites (Trivy, Grype)         │
  │  [x] Multi-stage build (reduire la surface d'attaque)  │
  │  [x] Pas de secrets dans l'image                       │
  │  [x] Limiter les ressources (CPU, memoire)             │
  │  [x] Signer les images (cosign, Notary)                │
  │  [x] Pas de :latest en production (versions fixes)     │
  │  [x] HEALTHCHECK dans le Dockerfile                    │
  │                                                         │
  └─────────────────────────────────────────────────────────┘
```

---

## Policy-as-Code

### Concept

Le **Policy-as-Code** consiste a definir les politiques de securite et de conformite sous forme de **code** versionne et testable, au lieu de documents Word.

### OPA (Open Policy Agent) et Rego

**OPA** est le standard pour le Policy-as-Code. Les politiques sont ecrites en **Rego** :

```rego
# policy/docker.rego
package docker

# Interdire les conteneurs root
deny[msg] {
    input.spec.containers[_].securityContext.runAsUser == 0
    msg := "Les conteneurs ne doivent pas tourner en root"
}

# Interdire les images sans tag specifique
deny[msg] {
    image := input.spec.containers[_].image
    not contains(image, ":")
    msg := sprintf("L'image '%s' doit avoir un tag specifique", [image])
}

# Exiger des limites de ressources
deny[msg] {
    container := input.spec.containers[_]
    not container.resources.limits
    msg := sprintf("Le conteneur '%s' doit avoir des limites de ressources", [container.name])
}
```

```bash
# Tester avec Conftest
conftest test deployment.yaml -p policy/

# FAIL - policy/docker.rego - Les conteneurs ne doivent pas tourner en root
# FAIL - policy/docker.rego - Le conteneur 'app' doit avoir des limites de ressources
# 2 tests, 0 passed, 2 warnings, 2 failures
```

---

## Pipeline CI/CD Securise Complet

Voici un workflow GitHub Actions integrant toutes les etapes de securite :

```yaml
# .github/workflows/devsecops.yml
name: DevSecOps Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  IMAGE_NAME: ghcr.io/${{ github.repository }}

jobs:
  # -------------------------------------------------------
  # ETAPE 1 : Qualite du code et secrets
  # -------------------------------------------------------
  code-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install bandit semgrep pip-audit safety

      # Detecter les secrets dans le code
      - name: Scan for secrets (Gitleaks)
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # Linting securite
      - name: Lint (Ruff)
        run: ruff check ./src

  # -------------------------------------------------------
  # ETAPE 2 : SAST (analyse statique de securite)
  # -------------------------------------------------------
  sast:
    runs-on: ubuntu-latest
    needs: code-quality
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install tools
        run: pip install bandit

      # Bandit : analyse securite Python
      - name: SAST - Bandit
        run: |
          bandit -r ./src -f json -o bandit-report.json || true
          bandit -r ./src -ll  # Fail seulement sur HIGH+

      # Semgrep : regles OWASP
      - name: SAST - Semgrep
        uses: semgrep/semgrep-action@v1
        with:
          config: >-
            p/python
            p/owasp-top-ten
            p/security-audit

  # -------------------------------------------------------
  # ETAPE 3 : Tests et scan des dependances
  # -------------------------------------------------------
  test-and-deps:
    runs-on: ubuntu-latest
    needs: code-quality
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: pip install -r requirements.txt

      # Tests unitaires
      - name: Run tests
        run: pytest tests/ -v --cov=src --cov-report=xml

      # Scan des dependances
      - name: Dependency scan (pip-audit)
        run: pip-audit --strict

      # Safety check
      - name: Dependency scan (Safety)
        run: safety check -r requirements.txt

  # -------------------------------------------------------
  # ETAPE 4 : Build et scan de l'image Docker
  # -------------------------------------------------------
  build-and-scan:
    runs-on: ubuntu-latest
    needs: [sast, test-and-deps]
    steps:
      - uses: actions/checkout@v4

      # Build l'image Docker
      - name: Build Docker image
        run: docker build -t ${{ env.IMAGE_NAME }}:${{ github.sha }} .

      # Scanner l'image avec Trivy
      - name: Scan image (Trivy)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.IMAGE_NAME }}:${{ github.sha }}
          format: 'table'
          exit-code: '1'              # Fail si vulnerabilites
          severity: 'CRITICAL,HIGH'   # Seulement critical et high
          ignore-unfixed: true        # Ignorer celles sans fix

      # Verifier les politiques (Conftest)
      - name: Policy check (Conftest)
        run: |
          conftest test Dockerfile -p policy/

  # -------------------------------------------------------
  # ETAPE 5 : DAST (test dynamique - sur PR seulement)
  # -------------------------------------------------------
  dast:
    runs-on: ubuntu-latest
    needs: build-and-scan
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4

      # Demarrer l'application
      - name: Start application
        run: |
          docker compose -f docker-compose.test.yml up -d
          sleep 10  # Attendre le demarrage

      # Scanner avec OWASP ZAP
      - name: DAST - OWASP ZAP
        uses: zaproxy/action-baseline@v0.10.0
        with:
          target: 'http://localhost:8000'
          rules_file_name: '.zap-rules.tsv'

      - name: Stop application
        if: always()
        run: docker compose -f docker-compose.test.yml down

  # -------------------------------------------------------
  # ETAPE 6 : Deploy (seulement si tout passe)
  # -------------------------------------------------------
  deploy:
    runs-on: ubuntu-latest
    needs: [build-and-scan]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    steps:
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Push image
        run: |
          docker push ${{ env.IMAGE_NAME }}:${{ github.sha }}
          docker tag ${{ env.IMAGE_NAME }}:${{ github.sha }} ${{ env.IMAGE_NAME }}:latest
          docker push ${{ env.IMAGE_NAME }}:latest

      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          echo "Deploying version ${{ github.sha }}..."
          # kubectl set image deployment/app app=${{ env.IMAGE_NAME }}:${{ github.sha }}
```

```
  VISUALISATION DU PIPELINE
  
  ┌──────────────┐
  │ code-quality │  Secrets + Lint
  └──────┬───────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
  ┌──────┐  ┌──────────────┐
  │ SAST │  │ test-and-deps│  En parallele
  └──┬───┘  └──────┬───────┘
     │             │
     └──────┬──────┘
            ▼
  ┌──────────────────┐
  │ build-and-scan   │  Image Docker + Trivy + Conftest
  └────────┬─────────┘
           │
     ┌─────┴─────┐
     │           │
     ▼           ▼
  ┌──────┐   ┌──────┐
  │ DAST │   │Deploy│  DAST sur PR, Deploy sur main
  │ (PR) │   │(main)│
  └──────┘   └──────┘
```

> [!example] Ce que ce pipeline detecte
> - **Gitleaks** : cle API oubliee dans le code
> - **Bandit** : injection SQL, mot de passe en dur
> - **Semgrep** : failles OWASP Top 10
> - **pip-audit** : dependance avec CVE connue
> - **Trivy** : image Docker avec librairie systeme vulnerable
> - **Conftest** : conteneur qui tourne en root
> - **OWASP ZAP** : XSS, headers manquants, misconfigurations

---

## Compliance pour les Developpeurs

### SOC 2

**SOC 2** (Service Organization Control) est un standard d'audit qui verifie que votre entreprise gere correctement les donnees. Ce que ca implique pour les developpeurs :

- **Audit trail** : journaliser qui fait quoi (logs d'acces, modifications)
- **Controle d'acces** : principe du moindre privilege (RBAC)
- **Chiffrement** : donnees sensibles chiffrees au repos et en transit
- **Reviews** : code reviews obligatoires, pas de push direct sur main
- **Incident management** : processus documente pour les incidents

### RGPD / GDPR

Le **RGPD** (Reglement General sur la Protection des Donnees) impose des regles strictes sur les donnees personnelles :

- **Minimisation** : ne collecter que les donnees strictement necessaires
- **Consentement** : obtenir l'accord explicite de l'utilisateur
- **Droit a l'oubli** : pouvoir supprimer toutes les donnees d'un utilisateur
- **Portabilite** : permettre l'export des donnees en format standard
- **Notification** : signaler les fuites de donnees sous 72 heures

```python
# Exemple : droit a l'oubli
async def delete_user_data(user_id: str):
    """Supprime TOUTES les donnees d'un utilisateur (RGPD Article 17)."""
    await db.users.delete(user_id)
    await db.orders.delete_where(user_id=user_id)
    await db.logs.anonymize_where(user_id=user_id)  # Anonymiser, pas supprimer
    await cache.delete(f"user:{user_id}")
    await search_index.delete(user_id)
    
    # Logger la suppression (sans donnees personnelles)
    logger.info("User data deleted", extra={"action": "gdpr_deletion", "user_id_hash": hash(user_id)})
```

---

## Culture de Securite

### Blameless Post-Mortems

Quand un incident de securite survient, la reaction naturelle est de chercher **qui** est responsable. C'est contre-productif. Les **blameless post-mortems** se concentrent sur le **systeme**, pas sur les personnes :

```
  MAUVAIS (blame)                    BON (blameless)
  
  "Jean a committe un secret        "Notre processus n'avait pas de
   dans le repo. C'est sa faute."    pre-commit hook pour detecter
                                     les secrets. L'onboarding ne
  Resultat : Jean a peur de          couvrait pas cette pratique."
  signaler les incidents futurs.     
                                     Resultat : on ajoute un hook,
                                     on met a jour l'onboarding.
                                     Tout le monde apprend.
```

### Security Champions

Designez un **Security Champion** dans chaque equipe de developpement :
- Pas un expert securite a temps plein, mais un developpeur **interesse** par la securite
- Fait le lien entre l'equipe securite et l'equipe dev
- Revoit les aspects securite des Pull Requests
- Partage les bonnes pratiques et les nouveautes

### Formation continue

| Action | Frequence | Objectif |
|---|---|---|
| **Secure coding training** | A l'onboarding + annuel | Connaitre les bases |
| **CTF (Capture The Flag)** | Trimestriel | Pratiquer l'attaque pour mieux defendre |
| **Revue des incidents** | Apres chaque incident | Apprendre des erreurs |
| **Veille CVE** | Continue (Dependabot) | Rester a jour sur les vulnerabilites |
| **Threat modeling** | A chaque nouvelle feature | Anticiper les menaces |

> [!tip] Analogie
> La securite, c'est comme l'**hygiene** dans un restaurant :
> - Ce n'est pas le travail d'une seule personne (l'inspecteur sanitaire)
> - **Tout le monde** doit se laver les mains, nettoyer son plan de travail, respecter la chaine du froid
> - Les controles reguliers (audits) verifient que les bonnes habitudes sont en place
> - Un seul manquement peut contaminer tout le restaurant
>
> La securite est la responsabilite de **toute l'equipe**, pas seulement de l'equipe securite.

---

## Carte Mentale

```
                          DEVSECOPS
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   PAR PHASE             TRANSVERSAL            CULTURE
        │                     │                     │
   ┌────┼────┐          ┌────┼────┐           ┌────┼────┐
   │    │    │          │    │    │           │    │    │
  Plan Code Build     Secrets Deps  Conteneurs Blameless Security
   │    │    │          │     │       │        post-    Champions
  Threat Pre- SAST    Vault  pip-   Trivy     mortems
  Model commit Bandit .env   audit  Grype              Formation
  STRIDE Hooks Semgrep GitHub Depend Snyk               CTF
              SonarQ  Secrets abot   non-root
                      SOPS   Renovate read-only
   ┌────┼────┐
   │    │    │
  Test Deploy Monitor    PIPELINE CI/CD
   │    │    │           ┌──────────────┐
  DAST Image Runtime     │ code-quality │
  ZAP  Scan  RASP       │ sast         │
  Burp Trivy             │ test + deps  │
       Sign              │ build + scan │
       Admiss.           │ dast         │
       Ctrl              │ deploy       │
                         └──────────────┘
```

---

## Exercices

### Exercice 1 : Identifier les vulnerabilites

Trouvez toutes les failles de securite dans ce code Python :

```python
from flask import Flask, request
import sqlite3
import os

app = Flask(__name__)
SECRET_KEY = "mysecretkey123"

@app.route("/login", methods=["POST"])
def login():
    username = request.form["username"]
    password = request.form["password"]
    
    conn = sqlite3.connect("app.db")
    cursor = conn.cursor()
    query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
    cursor.execute(query)
    user = cursor.fetchone()
    
    if user:
        return f"<h1>Bienvenue {username}!</h1>"
    return "Login failed", 401

@app.route("/exec")
def execute():
    cmd = request.args.get("cmd")
    result = os.popen(cmd).read()
    return result
```

> [!example] Solution
> 1. **Secret en dur** : `SECRET_KEY = "mysecretkey123"` (B105)
> 2. **Injection SQL** : `f"SELECT * FROM users WHERE..."` - utiliser des requetes parametrees
> 3. **Mots de passe en clair** : pas de hachage (utiliser bcrypt/argon2)
> 4. **XSS** : `f"<h1>Bienvenue {username}!</h1>"` - le username n'est pas echappe
> 5. **Injection de commandes** : `os.popen(cmd).read()` - execution arbitraire de commandes
> 6. **Pas de HTTPS** mentionne
> 7. **Pas de rate limiting** sur le login (brute force possible)
> 8. **Pas de CSRF** protection

### Exercice 2 : Securiser un Dockerfile

Transformez ce Dockerfile non securise en version securisee :

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y python3 python3-pip
COPY . /app
WORKDIR /app
RUN pip3 install -r requirements.txt
ENV DATABASE_URL=postgres://admin:password@db:5432/prod
CMD ["python3", "app.py"]
```

> [!example] Solution
> ```dockerfile
> FROM python:3.12-slim AS builder
> WORKDIR /build
> COPY requirements.txt .
> RUN pip install --no-cache-dir --user -r requirements.txt
> 
> FROM python:3.12-slim
> RUN groupadd -r app && useradd -r -g app app
> WORKDIR /app
> COPY --from=builder /root/.local /home/app/.local
> COPY --chown=app:app . .
> USER app
> ENV PATH=/home/app/.local/bin:$PATH
> HEALTHCHECK --interval=30s CMD curl -f http://localhost:8000/health || exit 1
> CMD ["python", "app.py"]
> # SECRET RETIRE : passer DATABASE_URL via variable d'env au runtime
> ```
> Changements : image slim, multi-stage, non-root, pas de secret, tag specifique, healthcheck.

### Exercice 3 : Pipeline de securite

Dessinez un pipeline CI/CD securise pour un projet Python + Docker qui inclut :
- Detection de secrets
- Analyse statique (SAST)
- Tests unitaires
- Scan des dependances
- Build et scan d'image Docker
- DAST sur environnement de staging

Indiquez pour chaque etape : l'outil utilise, ce qu'il detecte, et si c'est bloquant ou non.

### Exercice 4 : Gestion des secrets

Votre equipe utilise actuellement un fichier `config.py` avec tous les secrets en dur. Proposez un plan de migration en 4 etapes pour passer a une gestion securisee des secrets, en precisant les outils et les actions a chaque etape.

> [!example] Solution
> 1. **Etape immediate** : revoquer tous les secrets exposes, en generer de nouveaux, ajouter `.env` au `.gitignore`, creer `.env.example`
> 2. **Etape court terme** : installer `pre-commit` avec `detect-secrets` et `gitleaks`, migrer les secrets vers des variables d'environnement + `.env`
> 3. **Etape moyen terme** : utiliser GitHub Secrets pour le CI/CD, scanner l'historique Git avec `trufflehog` pour verifier qu'aucun secret ne reste
> 4. **Etape long terme** : deployer HashiCorp Vault ou AWS Secrets Manager, implementer la rotation automatique des secrets, audit trail

---

## Liens

- [[02 - Securite Web OWASP]] : les vulnerabilites web les plus courantes (OWASP Top 10)
- [[04 - CI-CD avec GitHub Actions]] : les bases du CI/CD pour integrer la securite
- [[05 - Infrastructure as Code Terraform]] : securiser l'infrastructure deployee