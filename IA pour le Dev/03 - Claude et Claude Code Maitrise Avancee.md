# Claude & Claude Code — Guide Complet de Zéro à Expert

> [!info] À qui s'adresse ce cours ?
> Ce cours va de l'**installation complète** jusqu'aux **optimisations avancées**. Il couvre Claude Code (le CLI officiel d'Anthropic), l'API Claude, et les bonnes pratiques valables aussi pour GPT-4, Gemini, et les modèles locaux (Ollama, LM Studio, OpenCode). Parfait si tu n'as jamais utilisé l'IA dans ton workflow, ou si tu veux passer au niveau supérieur.

---

## PARTIE 1 — DÉMARRAGE : INSTALLATION ET PREMIÈRE SESSION

### 1.1 Qu'est-ce que Claude Code ?

**Claude Code** est un assistant IA en ligne de commande (CLI) développé par Anthropic. Il s'intègre directement dans ton terminal et ton éditeur pour t'aider à coder, débugger, refactoriser, documenter, et bien plus.

```
┌─────────────────────────────────────────────────────────────┐
│                   L'ÉCOSYSTÈME CLAUDE                       │
│                                                             │
│  claude.ai       → Interface web (chat classique)          │
│  Claude Code CLI → Terminal interactif (ce cours)          │
│  Desktop App     → Application macOS/Windows native        │
│  VSCode Extension → Intégration éditeur                    │
│  JetBrains Plugin → IntelliJ, PyCharm, WebStorm...        │
│  API Anthropic   → Intégration dans tes propres applis     │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Prérequis

```bash
# Vérifier les prérequis
node --version    # Node.js ≥ 18.0.0 requis
npm --version     # npm ≥ 8.0.0 requis
git --version     # Git (recommandé)

# Installer Node.js si absent :
# → https://nodejs.org (télécharger LTS)
# → Ou via gestionnaire de paquets :
brew install node        # macOS (Homebrew)
winget install nodejs    # Windows
sudo apt install nodejs  # Ubuntu/Debian
```

### 1.3 Installation de Claude Code

```bash
# Installation globale via npm
npm install -g @anthropic-ai/claude-code

# Vérifier l'installation
claude --version

# Alternative macOS (Homebrew)
brew install anthropic-ai/claude/claude-code

# Mise à jour
npm update -g @anthropic-ai/claude-code
```

> [!tip] Windows spécifique
> Sur Windows, utilise **Git Bash**, **WSL**, ou **PowerShell** (pas CMD). Le `py` launcher est préféré à `python` ou `python3` sur Windows.

### 1.4 Authentification

```bash
# Méthode 1 : Login interactif (recommandé pour débutants)
claude login
# → Ouvre le navigateur → Se connecter à claude.ai → Autoriser

# Méthode 2 : API Key (recommandé pour scripts/CI/CD)
export ANTHROPIC_API_KEY="sk-ant-api03-xxxxxxxxxxxx"
# → Ajouter dans ~/.bashrc ou ~/.zshrc pour persister

# Méthode 3 : Fichier .env dans le projet
echo "ANTHROPIC_API_KEY=sk-ant-api03-xxx" >> .env

# Vérifier l'authentification
claude --print "dis bonjour"
```

> [!warning] Sécurité de l'API Key
> Ne commite JAMAIS ton API key dans Git. Utilise `.gitignore` pour exclure `.env`, et préfère les variables d'environnement système pour les projets partagés.

### 1.5 Premier lancement

```bash
# Démarrer Claude Code dans le répertoire courant
claude

# Démarrer dans un dossier spécifique
claude --cwd /chemin/vers/projet

# Mode non-interactif (pour scripts)
claude --print "explique ce que fait ce fichier" < mon_fichier.py

# Aide complète
claude --help
```

**L'interface interactive ressemble à :**

```
╔══════════════════════════════════════════════════════╗
║  Claude Code v1.x.x  │  claude-sonnet-4-6            ║
║  Dossier : /mon-projet                                ║
║  Contexte : ██████░░░░░░░░░ 40%                      ║
╠══════════════════════════════════════════════════════╣
║  > _                                                  ║
╚══════════════════════════════════════════════════════╝
```

### 1.6 Initialisation d'un nouveau projet

```bash
# Dans ton dossier de projet, initialiser Claude Code
cd mon-projet
claude

# Puis dans l'interface :
/init
```

La commande `/init` analyse ton projet et **génère automatiquement** un `CLAUDE.md` adapté à ta stack technique. Elle détecte :
- Le langage principal et la version
- Les frameworks utilisés (FastAPI, React, Django…)
- Les fichiers de config (package.json, Cargo.toml, pyproject.toml…)
- Les conventions git en place

> [!example] Exemple de CLAUDE.md généré par /init
> ```markdown
> # Project — FastAPI Backend
>
> ## Stack
> Python 3.12, FastAPI 0.110, PostgreSQL 16, Alembic, pytest
>
> ## Commands
> - Run dev: `uvicorn main:app --reload`
> - Tests: `pytest -v`
> - Migrate: `alembic upgrade head`
>
> ## Code Style
> - Type hints on all functions
> - Google-style docstrings
> - Max line length: 88 (black)
> ```

---

## PARTIE 2 — LES COMMANDES ESSENTIELLES

### 2.1 Slash Commands — La référence complète

| Commande | Description |
|----------|-------------|
| `/init` | Analyse le projet et génère CLAUDE.md |
| `/help` | Affiche l'aide et les commandes disponibles |
| `/clear` | Réinitialise la session (efface l'historique) |
| `/compact` | Compresse l'historique en résumé dense |
| `/context` | Affiche l'état du contexte et les tokens utilisés |
| `/fast` | Bascule en mode sortie rapide |
| `/model` | Change le modèle actif (haiku/sonnet/opus) |
| `/commit` | Analyse les changements et génère un message de commit |
| `/review-pr` | Revue complète d'une Pull Request |
| `/plan` | Active le Plan Mode (planifier avant d'agir) |
| `/cost` | Affiche le coût estimé de la session |
| `/memory` | Gère les fichiers mémoire (voir, ajouter, supprimer) |
| `/doctor` | Diagnostic de l'installation et de la config |

### 2.2 Raccourcis clavier

| Raccourci | Action |
|-----------|--------|
| `Ctrl+C` | Interrompre la génération en cours |
| `Ctrl+L` | Effacer l'affichage du terminal |
| `↑` / `↓` | Naviguer dans l'historique des commandes |
| `Tab` | Autocomplétion des commandes |
| `Escape` | Annuler la saisie en cours |

### 2.3 Flags CLI importants

```bash
# Choisir le modèle
claude --model claude-haiku-4-5-20251001
claude --model claude-sonnet-4-6
claude --model claude-opus-4-6

# Mode non-interactif (pour scripts automatisés)
claude --print "génère des tests pour auth.py"

# Format de sortie
claude --output-format json --print "analyse ce code"
claude --output-format stream-json --print "..."

# Limiter les permissions (mode sécurisé)
claude --no-permissions   # Demande confirmation pour chaque action
claude --dangerously-skip-permissions  # JAMAIS en production

# Verbosité
claude --verbose          # Affiche les détails des appels outils
claude --debug            # Mode debug complet

# Spécifier un dossier de travail
claude --cwd /chemin/projet

# Passer une instruction via fichier
claude --print < prompt.txt
```

### 2.4 Variables d'environnement utiles

```bash
# Clé API principale
export ANTHROPIC_API_KEY="sk-ant-..."

# Modèle par défaut (éviter de le spécifier à chaque fois)
export CLAUDE_MODEL="claude-sonnet-4-6"

# Chemin custom pour la config
export CLAUDE_CONFIG_DIR="~/.config/claude"

# Désactiver la télémétrie
export CLAUDE_TELEMETRY_DISABLED=1

# Mode CI/CD (désactive les prompts interactifs)
export CLAUDE_NON_INTERACTIVE=1

# Proxy (réseau d'entreprise)
export HTTPS_PROXY="http://proxy.company.com:8080"
```

---

## PARTIE 3 — CONFIGURATION AVANCÉE

### 3.1 CLAUDE.md — La Configuration Maîtresse

`CLAUDE.md` est un fichier Markdown lu **automatiquement** au démarrage de chaque session. C'est ton "system prompt permanent" sans effort répétitif.

```
~/.claude/CLAUDE.md          ← règles globales (TOUS les projets)
mon-projet/CLAUDE.md         ← règles du projet courant
mon-projet/src/CLAUDE.md     ← règles d'un sous-dossier (héritées)
```

**Structure optimale d'un CLAUDE.md :**

```markdown
# [Nom du Projet] — Instructions Claude

## Contexte
[Description courte : stack, objectif, contraintes importantes]

## Commandes fréquentes
- Démarrer : `uvicorn main:app --reload`
- Tests : `pytest -v --tb=short`
- Lint : `ruff check . && black --check .`
- Build Docker : `docker compose up --build`

## Conventions code
- [règle 1 dense et précise]
- [règle 2]
- Ne PAS modifier : [fichiers/dossiers interdits]

## Comportement attendu
- Toujours demander avant de toucher >3 fichiers
- Réponses courtes sauf si demandé
- Proposer un plan avant tout refactoring
```

> [!tip] Optimiser la densité
> Chaque mot du CLAUDE.md est relu à chaque session → investir du temps à l'écrire dense et précis.
> ```
> ❌ "Quand tu écris du Python, pense bien à mettre des type hints"
> ✓ "Python : type hints obligatoires sur toutes les signatures"
> ```

### 3.2 .claudeignore — Exclure l'Inutile

Fonctionne exactement comme `.gitignore` : liste les fichiers/dossiers que Claude ne doit PAS explorer automatiquement.

```gitignore
# .claudeignore — template universel

# Dépendances (JAMAIS utile à lire)
node_modules/
.venv/
__pycache__/
.mypy_cache/
target/          # Rust build output → RTK (Rust Token Killer)
dist/
build/
.next/
.nuxt/

# Données volumineuses
*.csv
*.parquet
*.sqlite
data/raw/
datasets/

# Fichiers générés
*.min.js
*.min.css
*.map
coverage/
htmlcov/

# Logs
*.log
logs/

# Secrets (ne JAMAIS exposer)
.env
.env.*
secrets/
*.pem
*.key

# Médias
*.mp4
*.mp3
*.png
*.jpg
*.pdf
```

### 3.3 Fichier de configuration JSON

```json
// ~/.claude/config.json — config globale Claude Code

{
  "defaultModel": "claude-sonnet-4-6",
  "autoCompact": {
    "enabled": true,
    "threshold": 0.60
  },
  "theme": "dark",
  "notifications": true,
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-postgres",
               "postgresql://localhost/mydb"]
    },
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_xxxx" }
    },
    "filesystem": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  }
}
```

### 3.4 Le Système de Mémoire Persistante

La mémoire permet de **conserver des informations entre sessions**. Elle est stockée dans des fichiers Markdown dans `~/.claude/projects/[projet]/memory/`.

```
~/.claude/projects/mon-projet/memory/
├── MEMORY.md          ← index automatiquement chargé
├── user.md            ← préférences personnelles (style, niveau)
├── feedback.md        ← corrections apportées par l'utilisateur
├── project.md         ← état du projet, décisions prises
└── reference.md       ← liens vers ressources externes
```

**Demander à Claude de mémoriser :**
```
"Mémorise que je préfère les tests d'intégration sans mocking"
"Retiens que le module payments/ ne doit jamais être modifié directement"
"Note que le bug auth est résolu — cause : cookie SameSite=None manquant"
```

**Ce qu'il NE faut PAS mettre en mémoire :**
- Structure du code (lire le code actuel est toujours plus fiable)
- Historique git (`git log` est autoritaire)
- Tâches de la session en cours (utiliser les todos de session)

---

## PARTIE 4 — COMPRENDRE LES TOKENS ET LES COÛTS

### 4.1 Qu'est-ce qu'un token ?

Un token est l'unité de base traitée par les LLMs. Ce n'est ni un mot ni un caractère.

```
"Bonjour le monde"  →  ["Bon", "jour", " le", " monde"]   = 4 tokens
"Hello world"        →  ["Hello", " world"]                = 2 tokens
int main() {         →  ["int", " main", "()", " {"]       = 4 tokens
```

**Règle empirique :**
- 1 token ≈ 0,75 mot anglais
- 1 token ≈ 0,60 mot français (**le français coûte ~20% plus cher**)
- 1 token ≈ 4 caractères de code ASCII

**Prix des modèles (API, 2026) :**

```
┌─────────────────┬──────────────┬───────────────┐
│ Modèle          │ Input/1M tk  │ Output/1M tk  │
├─────────────────┼──────────────┼───────────────┤
│ claude-opus-4-6 │    $15.00    │    $75.00     │
│ sonnet-4-6      │     $3.00    │    $15.00     │
│ haiku-4-5       │     $0.80    │     $4.00     │
└─────────────────┴──────────────┴───────────────┘
⚠️  Les sorties coûtent 5× plus que les entrées !
```

### 4.2 La Relecture Exponentielle

C'est le concept le plus important à comprendre. Claude **relit intégralement la conversation à chaque message**.

```
Message 1 →  100 tokens lus
Message 2 →  200 tokens lus  (100 historique + 100 nouveau)
Message 3 →  300 tokens lus
...
Message 20 → 2000 tokens lus

Coût réel = 100+200+300+...+2000 = n×(n+1)/2 × taille_moy
           = 20×21/2 × 100 = 21 000 tokens pour 2 000 de contenu !
```

**Solution : grouper les instructions connexes dans UN seul message**

```
❌ 3 messages séparés = 6 relectures cumulées
  → Toi : "Lis main.py"
  → Claude : [lit]
  → Toi : "Explique la fonction process_data"
  → Claude : [relit tout + répond]
  → Toi : "Ajoute des tests"
  → Claude : [relit tout + répond]

✓ 1 message groupé = 1 seule lecture
  → "Lis main.py, explique process_data et génère ses tests"
  → Claude : [fait tout en une passe]
```

### 4.3 Lost in the Middle

Les LLMs ont une attention **non uniforme** sur leur contexte :

```
Attention du modèle :

  Début ████████████████████  ← Forte (system prompt, début conv)
  Milieu ████████              ← Réduite (~40-60%)
  Fin    ████████████████████  ← Forte (derniers messages)
```

Les instructions au milieu d'une longue conversation sont moins bien prises en compte.

**Stratégies :**
- Instructions critiques → `CLAUDE.md` (toujours en début) ou dernier message
- Rappeler les règles importantes dans le message final si la session est longue
- `/compact` pour condenser le milieu et garder un résumé dense

### 4.4 Le Prompt Cache — 5 Minutes Magiques

Anthropic met en cache automatiquement les contextes récemment lus. Si le contexte n'a pas changé depuis moins de 5 minutes, il est servi à **~10% du prix normal**.

```
┌──────────────────────────────────────────────────────────┐
│                  PROMPT CACHE                            │
│  TTL : 5 minutes (300 secondes)                         │
│  Réduction : ~90% sur les tokens mis en cache           │
│  Déclenché par : contexte identique dans la fenêtre TTL │
│                                                          │
│  ⏱  Après 5 min d'inactivité → cache expiré            │
│     → prochain message relit TOUT à plein tarif         │
└──────────────────────────────────────────────────────────┘
```

> [!tip] Maintenir le cache actif
> Si tu dois faire une pause, reviens dans les 5 minutes. Un message court ("continue", "ok") suffit à maintenir le cache actif et économise jusqu'à 90% sur le prochain échange.

---

## PARTIE 5 — GESTION DU CONTEXTE

### 5.1 /compact — Compresser sans perdre

`/compact` condense l'historique en un résumé dense, réduisant drastiquement les tokens tout en préservant les informations clés.

```
Avant /compact : 80 000 tokens d'historique
Après /compact :  8 000 tokens de résumé (ratio ×10 typique)

Économie sur prochains messages : 72 000 tokens × prix_cache
```

**Quand utiliser :**
- Barre de contexte à 60-70%
- Après avoir résolu un problème complexe
- Avant de passer à un nouveau sujet dans la même session

### 5.2 /clear — Repartir à Zéro

Efface complètement l'historique. La session redémarre vierge.

```
Utiliser /clear quand :
✓ Changement complet de sujet
✓ Claude semble confus ou incohérent
✓ La conversation a trop dérivé

NE PAS utiliser /clear quand :
✗ En plein milieu d'un refactoring multi-fichiers
✗ Claude a du contexte utile sur ton projet
```

### 5.3 Le Combo Résumé + /clear (technique pro)

```
1. Demander : "Fais un résumé structuré : ce qu'on a fait,
   décisions prises, état actuel, prochaines étapes."
2. Claude génère un résumé de 200-500 tokens
3. /clear  (efface 50 000 tokens d'historique)
4. Coller le résumé dans le premier message de la nouvelle session
5. Reprendre le travail avec un contexte frais et ciblé
```

**Économie concrète :**
```
50 000 tokens d'historique → 400 tokens de résumé
Économie : 49 600 tokens × $0.003/1K = ~$0.15 par /clear
Sur 10 /clear dans une journée : ~$1.50 économisés sur Sonnet
```

### 5.4 Auto-compact

Dans la config ou via les paramètres, activer l'auto-compact à **60% du contexte** :

```json
// ~/.claude/config.json
{
  "autoCompact": {
    "enabled": true,
    "threshold": 0.60
  }
}
```

Cela déclenche automatiquement `/compact` avant que le contexte soit critique.

### 5.5 /context — Voir l'État du Contexte

Affiche un résumé de ce qui est dans la fenêtre de contexte : fichiers lus, tokens utilisés, pourcentage restant. Utiliser pour décider si un `/compact` est nécessaire.

---

## PARTIE 6 — MODES ET WORKFLOWS AVANCÉS

### 6.1 Plan Mode — Réfléchir Avant d'Agir

Plan Mode force Claude à **présenter son plan avant d'exécuter**.

```bash
# Activer le Plan Mode
/plan

# Ou dans le message directement
"AVANT D'AGIR : fais-moi un plan détaillé de comment tu vas
refactoriser le module auth, puis attends ma validation."
```

**Workflow recommandé :**
```
1. Activer Plan Mode
2. Décrire la tâche complexe
3. Claude présente : fichiers concernés, ordre des modifications,
   risques identifiés, questions de clarification
4. Valider, modifier, ou demander des précisions
5. Seulement alors → Claude exécute
```

> [!tip] Le hack "95% de confiance"
> Ajouter dans chaque prompt complexe :
> **"Ne fais aucun changement tant que tu n'as pas 95% de confiance. Pose-moi des questions de suivi d'abord."**
>
> Cela évite les allers-retours coûteux ("non c'est pas ça que je voulais" → 3 messages de correction = tokens perdus).

### 6.2 Mode Non-Interactif (Scripts et CI/CD)

```bash
# Utilisation dans un script shell
#!/bin/bash
result=$(claude --print "analyse les erreurs dans build.log" < build.log)
echo "$result" >> rapport.txt

# Avec format JSON pour parsing
claude --print --output-format json "liste les TODO dans src/" \
  | jq '.content[0].text'

# Intégration CI/CD (GitHub Actions)
- name: Code Review avec Claude
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  run: |
    git diff HEAD~1 | claude --print "revue de code, liste les problèmes"
```

### 6.3 Skills — Raccourcis Réutilisables

Les skills sont des commandes personnalisées qui déclenchent des workflows complexes.

```markdown
# ~/.claude/skills/deploy-check.md
Avant tout déploiement, vérifier :
1. Tests passent : `pytest -v --tb=short`
2. Linting propre : `ruff check . && black --check .`
3. Variables d'env production définies (check .env.example)
4. Migrations à jour : `alembic heads`
5. Docker build réussit : `docker compose build`
Générer un rapport PASS/FAIL pour chaque point.
```

```bash
# Utilisation
/deploy-check    # Lance le workflow complet
```

**Skills intégrés utiles :**
```
/commit         → analyse les changements + message de commit conventionnel
/review-pr      → revue complète d'une PR avec GitHub CLI
/compact        → compression du contexte
/init           → initialisation du projet + génération CLAUDE.md
```

---

## PARTIE 7 — SUB-AGENTS ET PARALLÉLISATION

### 7.1 Qu'est-ce qu'un Sub-agent ?

Un sub-agent est une instance Claude **séparée**, avec son propre contexte et ses propres outils, lancée pour une tâche spécifique.

```
┌──────────────────────────────────────────────────────────────┐
│                   AGENT PRINCIPAL                            │
│   Contexte : 50 000 tokens                                   │
│   Tâche globale : "migrer le projet vers FastAPI"            │
│              │                                               │
│    ┌─────────┼──────────────┐                                │
│    ▼         ▼              ▼                                │
│  Agent A   Agent B        Agent C                            │
│  Tâche:    Tâche:         Tâche:                             │
│  Tests     Documentation  Migration DB                       │
│  (indép.)  (indép.)       (indép.)                          │
│    │         │              │                                │
│    └─────────┴──────────────┘                               │
│              ▼                                               │
│         Résultats agrégés → Agent principal                  │
└──────────────────────────────────────────────────────────────┘
```

### 7.2 Types de Sub-agents Disponibles

```
general-purpose   → Recherche, exploration, tâches complexes
Explore           → Exploration rapide de codebase (lecture seule)
Plan              → Architecture, plans d'implémentation
claude-code-guide → Questions sur Claude Code, l'API, les MCP
```

### 7.3 Isolation avec Worktree

Pour les modifications risquées, les sub-agents peuvent travailler dans un **worktree Git isolé** — une copie du repo sur une branche temporaire :

```
Avantages du worktree :
✓ Modifications isolées (ne touche pas ta branche courante)
✓ Nettoyage automatique si aucun changement
✓ Si changements : le chemin de branche est retourné pour review
✓ Idéal pour tester une approche risquée sans conséquences
```

### 7.4 Coûts Réels des Sub-agents

> [!warning] Les sub-agents ne sont pas gratuits
> ```
> Agent principal      : 10 000 tokens contexte
> 3 sub-agents lancés  : 3 × (contexte hérité + travail propre)
>                      = 3 × 25 000 = 75 000 tokens
>
> vs. travail séquentiel : 10 000 + 15 000 + 15 000 = 40 000 tokens
>
> Les sub-agents coûtent ~1.9× plus mais sont 3× plus rapides.
> C'est un choix vitesse ↔ coût.
> ```

### 7.5 Stratégie : Parallélisation 5 Minutes Avant la Fin

> [!tip] Technique avancée de rentabilisation
> Quand le contexte approche les 80%, lancer plusieurs sub-agents pour les tâches restantes. Ils héritent du contexte riche actuel et travaillent en parallèle pendant que ta session principale se termine proprement. Tu "dépenses" tes tokens restants productivement plutôt que de les laisser expirer.

---

## PARTIE 8 — HOOKS : AUTOMATISER CLAUDE CODE

### 8.1 Qu'est-ce qu'un Hook ?

Les **hooks** sont des commandes shell qui s'exécutent automatiquement en réponse à des événements de Claude Code. Ils permettent d'automatiser des actions récurrentes.

```
┌──────────────────────────────────────────────────────────────┐
│                     TYPES DE HOOKS                           │
│                                                              │
│  PreToolUse    → Avant qu'un outil soit exécuté             │
│  PostToolUse   → Après qu'un outil soit exécuté             │
│  Notification  → Quand Claude émet une notification          │
│  Stop          → Quand Claude termine une réponse           │
└──────────────────────────────────────────────────────────────┘
```

### 8.2 Configuration des Hooks

```json
// ~/.claude/config.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Commande exécutée : $TOOL_INPUT' >> ~/.claude/bash_history.log"
          }
        ]
      }
    ],
    "Notification": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' '$NOTIFICATION_MESSAGE'"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "afplay /System/Library/Sounds/Glass.aiff"
          }
        ]
      }
    ]
  }
}
```

### 8.3 Exemples de Hooks Utiles

```bash
# Hook : Lancer les tests automatiquement après écriture de fichiers Python
PostToolUse → Write → matcher: "*.py"
command: "cd $PROJECT_DIR && pytest --tb=short -q 2>&1 | tail -5"

# Hook : Formater le code après modifications
PostToolUse → Edit → matcher: "*.py"
command: "black $TOOL_OUTPUT_FILE 2>/dev/null && ruff check --fix $TOOL_OUTPUT_FILE 2>/dev/null"

# Hook : Notification de fin de tâche longue (macOS)
Stop → command: "osascript -e 'display notification \"Claude a terminé\" with title \"Claude Code\"'"

# Hook : Log de toutes les commandes Bash exécutées
PostToolUse → Bash
command: "echo \"$(date): $TOOL_INPUT\" >> ~/claude-bash-audit.log"

# Hook : Bloquer l'exécution de commandes dangereuses
PreToolUse → Bash → matcher: "rm -rf"
command: "echo 'BLOQUÉ : rm -rf interdit' && exit 1"
```

---

## PARTIE 9 — SERVEURS MCP (MODEL CONTEXT PROTOCOL)

### 9.1 Qu'est-ce que MCP ?

MCP est un protocole standard qui permet à Claude de se connecter à des **sources de données et outils externes** directement dans la session.

```
Claude ←→ MCP Server ←→ Base de données PostgreSQL
Claude ←→ MCP Server ←→ API GitHub (issues, PRs, repos)
Claude ←→ MCP Server ←→ Notion / Jira / Linear
Claude ←→ MCP Server ←→ Gmail / Google Calendar
Claude ←→ MCP Server ←→ Navigateur web (Playwright)
Claude ←→ MCP Server ←→ Système de fichiers distant
```

### 9.2 MCP Servers Populaires

```bash
# PostgreSQL — accès direct à la base de données
npm install -g @modelcontextprotocol/server-postgres

# Filesystem — accès à des dossiers spécifiques
npm install -g @modelcontextprotocol/server-filesystem

# GitHub — issues, PRs, repos
npm install -g @modelcontextprotocol/server-github

# Puppeteer/Playwright — automatisation navigateur
npm install -g @modelcontextprotocol/server-puppeteer

# Brave Search — recherche web
npm install -g @modelcontextprotocol/server-brave-search

# SQLite
npm install -g @modelcontextprotocol/server-sqlite
```

### 9.3 Configuration MCP

```json
// ~/.claude/config.json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-postgres",
        "postgresql://user:password@localhost:5432/mydb"
      ]
    },
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_xxxx" }
    },
    "myfiles": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects",
        "/home/user/documents"
      ]
    }
  }
}
```

> [!tip] MCP vs lecture de fichiers manuelle
> Sans MCP PostgreSQL :
> "Lis ce dump SQL et explique le schéma" → copier-coller manuel, données potentiellement stale
>
> Avec MCP PostgreSQL :
> Claude fait directement `\d table_name` → données fraîches, zéro copier-coller, 0 token pour le copier-coller

> [!warning] Toujours limiter les résultats SQL
> ```sql
> -- ❌ Peut retourner des millions de lignes
> SELECT * FROM logs;
>
> -- ✓ Ciblé et limité
> SELECT message, level, created_at FROM logs
> WHERE level = 'ERROR'
> ORDER BY created_at DESC
> LIMIT 20;
> ```

---

## PARTIE 10 — L'ADVISOR TOOL (NOUVEAUTÉ BETA 2026)

### 10.1 Le Concept Révolutionnaire : Exécuteur + Conseiller

L'**Advisor Tool** est une des innovations les plus importantes de 2026. Il permet d'associer un **modèle exécuteur rapide** (Sonnet ou Haiku) avec un **modèle conseiller plus intelligent** (Opus), consulté ponctuellement pour les décisions stratégiques.

```
┌──────────────────────────────────────────────────────────────┐
│                   PATTERN ADVISOR                            │
│                                                              │
│   EXÉCUTEUR (Sonnet/Haiku)                                   │
│   → Fait le gros du travail à moindre coût                  │
│   → Lit les fichiers, écrit le code, exécute les outils     │
│              │                                               │
│              │ "Je dois planifier cette tâche complexe"     │
│              ▼                                               │
│   CONSEILLER (Opus) ← Lit TOUT le transcript en cours      │
│   → Produit un plan stratégique (400-700 tokens)            │
│   → Donne une direction claire                              │
│              │                                               │
│              ▼                                               │
│   EXÉCUTEUR continue avec le plan Opus                      │
│   → Qualité proche d'Opus solo                              │
│   → Coût proche de Sonnet solo                              │
└──────────────────────────────────────────────────────────────┘
```

**Bénéfice clé** : intelligence proche d'Opus, au coût de Sonnet pour la majorité du travail.

### 10.2 Utilisation via l'API

```python
import anthropic

client = anthropic.Anthropic()

response = client.beta.messages.create(
    model="claude-sonnet-4-6",          # L'exécuteur
    max_tokens=4096,
    betas=["advisor-tool-2026-03-01"],  # Activer le beta
    tools=[
        {
            "type": "advisor_20260301",
            "name": "advisor",
            "model": "claude-opus-4-6",  # Le conseiller
            "max_uses": 3,               # Max 3 consultations par requête
        }
    ],
    messages=[
        {
            "role": "user",
            "content": "Implémente un worker pool concurrent en Go avec arrêt gracieux.",
        }
    ],
)
```

```bash
# Via curl
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: advisor-tool-2026-03-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-sonnet-4-6",
    "max_tokens": 4096,
    "tools": [{
      "type": "advisor_20260301",
      "name": "advisor",
      "model": "claude-opus-4-6"
    }],
    "messages": [{
      "role": "user",
      "content": "Construis un worker pool concurrent en Go."
    }]
  }'
```

### 10.3 Paires Exécuteur/Conseiller Valides

| Exécuteur | Conseiller possible |
|-----------|---------------------|
| `claude-haiku-4-5-20251001` | `claude-opus-4-6` |
| `claude-sonnet-4-6` | `claude-opus-4-6` |
| `claude-opus-4-6` | `claude-opus-4-6` |

> [!warning] Le conseiller doit être au moins aussi capable que l'exécuteur. Une paire invalide retourne une erreur 400.

### 10.4 Comment l'Advisor Fonctionne en Détail

```
1. L'exécuteur décide lui-même quand appeler le conseiller
   (comme n'importe quel autre outil)

2. Quand l'exécuteur appelle l'advisor :
   → Émission d'un bloc server_tool_use (input toujours vide)
   → Anthropic lance une inférence séparée sur le modèle conseiller
   → Le conseiller voit TOUT : system prompt, historique, résultats outils
   → Le conseiller produit un plan (400-700 tokens)

3. Le résultat revient à l'exécuteur sous forme advisor_tool_result
4. L'exécuteur continue avec le plan en tête

Tout cela dans UN SEUL appel API — aucun aller-retour supplémentaire.
```

### 10.5 Optimisation du Prompt System pour l'Advisor

**Quand appeler l'advisor (ajouter au system prompt) :**
```
Appelle advisor AVANT tout travail substantiel — avant d'écrire,
avant de t'engager sur une interprétation, avant de construire
sur une hypothèse. Si la tâche nécessite d'abord une orientation
(trouver des fichiers, lire la structure), fais ça, puis appelle advisor.

Appelle aussi advisor :
- Quand tu penses avoir terminé (AVANT : rends le résultat durable)
- Quand tu bloques — erreurs répétées, approche qui ne converge pas
- Quand tu envisages un changement d'approche
```

**Réduire la longueur des réponses advisor (économise 35-45% des tokens) :**
```
"L'advisor doit répondre en moins de 100 mots avec des étapes
numérotées, pas des explications."
```

### 10.6 Cache de l'Advisor

```python
# Activer le cache côté advisor (rentable à partir de 3+ appels)
tools = [{
    "type": "advisor_20260301",
    "name": "advisor",
    "model": "claude-opus-4-6",
    "caching": {"type": "ephemeral", "ttl": "5m"},  # ou "1h"
}]
```

> [!tip] Quand activer le cache advisor ?
> - ✓ Conversations longues avec 3+ appels à l'advisor
> - ✗ Tâches courtes (≤2 appels) : le coût d'écriture dépasse l'économie de lecture

### 10.7 Cas d'Usage Idéaux

```
✓ Parfait pour l'Advisor :
  - Agents de code long horizon (>5 étapes)
  - Pipelines de recherche multi-étapes
  - Computer use complexe
  - Refactoring architecturaux

✗ Pas idéal pour l'Advisor :
  - Questions réponse unique (rien à planifier)
  - Tâches où chaque tour nécessite Opus solo
  - Formatage et tâches répétitives simples
```

---

## PARTIE 11 — NOUVEAUTÉS ET FONCTIONNALITÉS AVANCÉES

### 11.1 Extended Thinking — Raisonnement Approfondi

Extended Thinking permet à Claude de "réfléchir" avant de répondre, en utilisant des blocs de raisonnement internes.

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # Tokens alloués à la réflexion
    },
    messages=[{"role": "user", "content": "Résous ce problème algorithmique complexe..."}]
)

# Les blocs de réflexion apparaissent dans la réponse
for block in response.content:
    if block.type == "thinking":
        print("Réflexion interne:", block.thinking)
    elif block.type == "text":
        print("Réponse:", block.text)
```

**Quand utiliser Extended Thinking :**
- Problèmes mathématiques/algorithmiques complexes
- Décisions architecturales à fort impact
- Debugging de bugs subtils et multi-couches
- Analyses nécessitant du raisonnement en plusieurs étapes

### 11.2 Web Search Tool — Accès au Web en Temps Réel

```python
# Le modèle peut chercher sur le web de façon autonome
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[{
        "type": "web_search_20250305",
        "name": "web_search",
        "max_uses": 5,          # Limiter les recherches
    }],
    messages=[{
        "role": "user",
        "content": "Quelle est la dernière version stable de FastAPI ?"
    }]
)
```

> [!tip] Combiner Advisor + Web Search
> ```python
> tools = [
>     {"type": "web_search_20250305", "name": "web_search", "max_uses": 5},
>     {"type": "advisor_20260301", "name": "advisor", "model": "claude-opus-4-6"},
>     {"name": "run_bash", ...}  # Tes outils custom
> ]
> # L'exécuteur peut chercher sur le web, demander conseil à l'advisor,
> # et utiliser tes outils, tout dans un même appel.
> ```

### 11.3 Context Editing — Contrôle Fin du Contexte

```python
# Supprimer les blocs de thinking pour réduire le contexte
response = client.messages.create(
    model="claude-sonnet-4-6",
    thinking={
        "type": "enabled",
        "budget_tokens": 5000
    },
    betas=["interleaved-thinking-2025-05-14"],
    clear_thinking={
        "keep": "all"           # Garder tous les blocs thinking
        # ou "keep": {"type": "thinking_turns", "value": 1}
        # ou "keep": "none"     # Supprimer tous les blocs thinking
    },
    ...
)
```

### 11.4 Batch Processing — Traitement Massif

Pour traiter des centaines de requêtes à moindre coût (-50%) avec un délai de traitement plus long :

```python
# Créer un batch de requêtes
batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": f"test-generation-{i}",
            "params": {
                "model": "claude-haiku-4-5-20251001",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": f"Génère un test pour {func}"}]
            }
        }
        for i, func in enumerate(list_of_functions)
    ]
)

# Récupérer les résultats (traitement asynchrone)
results = client.messages.batches.results(batch.id)
```

**Cas d'usage :** générer 100 tests unitaires, documenter 200 fonctions, analyser des milliers de fichiers → **-50% de coût** vs appels synchrones.

---

## PARTIE 12 — CHOISIR LE BON MODÈLE

### 12.1 La Règle des 3 Niveaux

```
┌──────────────────────────────────────────────────────────────┐
│  HAIKU (rapide, économique)                                   │
│  Coût : $0.80/1M input, $4/1M output                        │
│  → Classification, extraction, formatage                     │
│  → Génération répétitive (tests, doc, commentaires)         │
│  → Tâches où la qualité n'est pas critique                  │
│                                                              │
│  SONNET (équilibré — le choix par défaut)                    │
│  Coût : $3/1M input, $15/1M output                          │
│  → Développement quotidien                                   │
│  → Debugging, refactoring, review de code                   │
│  → Analyse et explication de code                           │
│  → 90% des cas d'usage de développement                    │
│                                                              │
│  OPUS (le plus puissant)                                     │
│  Coût : $15/1M input, $75/1M output                         │
│  → Architecture système complexe                            │
│  → Algorithmes non triviaux, problèmes ouverts             │
│  → Décisions critiques et irréversibles                     │
│  → En tant qu'Advisor avec Sonnet exécuteur                │
└──────────────────────────────────────────────────────────────┘
```

### 12.2 Planning d'une Session Type

```
09h00 - Architecture du module auth     → OPUS    (décision critique)
09h30 - Implémentation des endpoints    → SONNET  (travail principal)
14h00 - Génération de 50 tests         → HAIKU   (répétitif)
15h00 - Debug comportement inattendu   → SONNET  (analyse précise)
16h30 - Génération documentation API   → HAIKU   (formatage)
17h00 - Review de la PR finale         → SONNET  (jugement global)
```

### 12.3 Changer de Modèle dans la Session

```bash
# Dans l'interface Claude Code
/model haiku    # Passer à Haiku pour les tâches simples
/model sonnet   # Revenir à Sonnet
/model opus     # Passer à Opus pour une analyse complexe
```

---

## PARTIE 13 — OUTPUTS VOLUMINEUX ET RTK

### 13.1 RTK : Le Rust Token Killer

Le compilateur Rust est réputé pour ses messages d'erreur **extrêmement détaillés** — une qualité pour les humains, un gouffre à tokens pour les LLMs.

```bash
# Une compilation Rust avec warnings peut générer 500-2000 tokens :
$ cargo build
warning: unused variable: `x`
  --> src/main.rs:15:9
   |
15 |     let x = compute();
   |         ^ help: prefix with underscore: `_x`
   = note: `#[warn(unused_variables)]` on by default

[... 200 lignes de warnings détaillés ...]
error[E0502]: cannot borrow `data` as mutable because it is also borrowed...
[... 50 lignes d'explication du borrow checker ...]
```

**Solutions :**

```bash
# Filtrer seulement les erreurs
cargo build 2>&1 | grep "^error" | head -20

# Mode quiet
cargo build -q 2>&1 | grep "error\["

# Rediriger, ne partager que l'essentiel
cargo build 2> /tmp/build_errors.txt
# Puis : "voici les erreurs : $(grep 'error\[' /tmp/build_errors.txt)"

# Pour les tests Rust
cargo test 2>&1 | grep -E "^(FAILED|error)" | head -30
```

### 13.2 Principe Général sur les Outputs de Commandes

```
Avant de coller une sortie dans Claude :
1. Vérifier la longueur (>100 lignes = potentiellement problématique)
2. Filtrer pour ne garder que les erreurs/warnings pertinents
3. Ne jamais coller node_modules, stack traces Java entières, logs systèmes bruts

Tailles typiques et recommandations :
  npm install    : 50-200 lignes → OK si problème détecté
  cargo build    : jusqu'à 2000 lignes → TOUJOURS filtrer
  docker logs    : illimité → TOUJOURS limiter avec --tail 50
  pytest -v      : OK si <50 tests, filtrer pour >100
  git log        : --oneline --last 20 pour ne pas noyer
  make all       : dépend → filtrer les warnings répétitifs
```

```bash
# Commandes "safe" pour Claude
docker logs mon-conteneur --tail 50 --since 5m
pytest --tb=short -q 2>&1 | tail -30
git log --oneline -20
journalctl -u mon-service --since "5 minutes ago" | tail -50
npm run build 2>&1 | grep -E "error:|ERROR:" | head -20
```

---

## PARTIE 14 — HYGIÈNE DE CONTEXTE

### 14.1 Les 10 Règles d'Or

```
1. SESSION CIBLÉE
   Une session = un objectif précis. Ne pas mélanger
   "debug auth" et "refactoring DB" dans la même session.

2. FICHIERS PRÉCIS
   "Regarde src/auth/jwt.py lignes 45-80" > "regarde le code"

3. LIRE EN DERNIER
   Lire les fichiers importants EN DERNIER juste avant la demande.

4. /compact PROACTIF
   Ne pas attendre 90% → /compact à 60% (ou auto-compact activé)

5. RÉSUMÉ AVANT /clear
   Toujours sauvegarder le contexte essentiel avant de réinitialiser.

6. GROUPER LES DEMANDES
   3 questions dans 1 message > 3 messages séparés

7. FEEDBACK EN LIGNE
   "Non, modifie seulement la ligne 42" > "c'est pas ça, recommence"

8. ÉVITER LES CONFIRMATIONS VIDES
   "ok" seul = tokens pour 0 valeur → grouper avec la prochaine instruction

9. .claudeignore À JOUR
   Maintenir quand de nouveaux dossiers volumineux apparaissent

10. MÉMOIRE POUR CE QUI DURE
    Ne pas répéter d'une session à l'autre → sauvegarder en mémoire
```

### 14.2 Peak Hours et Performances

```
Heures de pointe (API LLM) :
  14h-22h heure française = 8h-16h EST (business hours US)
  → Latences plus élevées
  → Rate limits plus stricts
  → Débit de tokens/seconde réduit

Heures creuses :
  Nuit française (2h-8h)
  Weekends
  → Meilleures performances et débit

Pour les gros batches (génération massive de tests, docs) :
  → Planifier hors heures de pointe ou utiliser le Batch API
```

### 14.3 Checklist Pré-Session

```
□ CLAUDE.md à jour pour ce projet ?
□ .claudeignore configuré ?
□ Objectif de la session défini précisément ?
□ Bon modèle sélectionné (Haiku/Sonnet/Opus) ?
□ Auto-compact configuré à 60% ?
□ Résumé de la session précédente disponible si continuation ?
□ Outputs de commandes filtrés avant partage ?
□ Advisor activé si tâche d'agent complexe ? (API uniquement)
```

---

## PARTIE 15 — APPLICATION AUX AUTRES IAs

### 15.1 ChatGPT / GPT-4o (OpenAI)

```
Similitudes avec Claude :
  ✓ Fenêtre contexte : 128K tokens (GPT-4o)
  ✓ Prompt cache : oui (même mécanisme TTL)
  ✓ Lost in the Middle : même phénomène
  ✓ Mémoire persistante intégrée (dans l'interface web)

Différences :
  ✗ Custom GPTs = équivalent CLAUDE.md + skills intégrés
  ✗ Pas de sub-agents natifs (sauf via Assistants API)
  ✗ Pas de /compact natif → "résume en 200 mots" manuel

Équivalences de commandes :
  /compact → "Résume ce qu'on a fait : décisions et état actuel"
  CLAUDE.md → System prompt du Custom GPT
  .claudeignore → Pas d'équivalent (gérer manuellement)
  /model → Sélectionner GPT-4o-mini (=Haiku) vs GPT-4o (=Sonnet)
```

### 15.2 Google Gemini

```
Points forts :
  ✓ Contexte 1M tokens (Gemini 2.0 Flash) → saturation rare
  ✓ Intégration native Google Workspace
  ✓ Multimodal natif (vidéo, audio, images, code)

Points faibles :
  ✗ Pas d'équivalent CLAUDE.md natif
  ✗ Moins d'outils de gestion de contexte
  ✗ Interface moins adaptée au développement pur

Mêmes bonnes pratiques s'appliquent :
  → Grouper les questions
  → Filtrer les outputs de commandes
  → Sessions dédiées par sujet
```

### 15.3 Modèles Locaux — Ollama, LM Studio, OpenCode

> [!tip] L'avantage majeur : zéro coût par token
> Les contraintes économiques disparaissent. Les contraintes de performance (qualité, vitesse, taille contexte) restent.

```
┌──────────────────────────────────────────────────────┐
│           COMPARATIF : LOCAL vs CLOUD                │
│                                                      │
│  LOCAL                    CLOUD (Claude/GPT)         │
│  ✓ Gratuit à l'usage      ✓ Qualité supérieure      │
│  ✓ Confidentialité totale ✓ Contexte large (200K+)  │
│  ✓ Disponible hors ligne  ✓ Toujours à jour         │
│  ✓ Pas de rate limits     ✓ Outils natifs           │
│  ✗ Qualité inférieure     ✗ Coût variable           │
│  ✗ Contexte réduit (8-32K)✗ Données envoyées        │
│  ✗ Lent sans GPU puissant ✗ Dépendance réseau       │
│  ✗ Peu d'outils natifs                              │
└──────────────────────────────────────────────────────┘
```

**Installation Ollama :**

```bash
# Linux/macOS
curl -fsSL https://ollama.ai/install.sh | sh

# Windows : télécharger l'installeur depuis ollama.ai

# Modèles recommandés
ollama pull llama3.3           # Polyvalent, 8B params
ollama pull codellama:34b      # Spécialisé code (meilleur pour dev)
ollama pull deepseek-coder-v2  # Excellent code, raisonnement fort
ollama pull phi4               # Très rapide, petits projets
ollama pull mistral-nemo       # Bon multilingue

# Lancer
ollama run codellama:34b

# API locale (compatible OpenAI API)
curl http://localhost:11434/api/generate \
  -d '{"model": "codellama", "prompt": "Explique malloc en C"}'
```

**LM Studio — Interface graphique pour modèles locaux :**

```
1. Télécharger LM Studio (lmstudio.ai)
2. Chercher et télécharger des modèles GGUF
3. Activer le serveur local dans l'onglet "Local Server"
4. Endpoint : http://localhost:1234/v1 (compatible OpenAI)
```

**OpenCode — Claude Code avec modèles locaux :**

```bash
# Installation
npm install -g opencode-ai

# Configurer pour Ollama
opencode config set model ollama/codellama:34b
opencode config set base-url http://localhost:11434/v1

# Ou LM Studio
opencode config set model lmstudio/deepseek-coder
opencode config set base-url http://localhost:1234/v1

# Utiliser normalement
opencode
```

**Modèles recommandés par cas d'usage (local) :**

| Cas d'usage | Modèle recommandé |
|-------------|-------------------|
| Code général | `deepseek-coder-v2` ou `codellama:34b` |
| Chat/Rédaction | `llama3.3:70b` ou `mistral-large` |
| Tâches rapides | `llama3.2:3b` ou `phi4` |
| Multilingue | `mistral-nemo` |
| Raisonnement | `deepseek-r1` |

**Adapter au contexte réduit des modèles locaux :**

```
Sessions encore plus courtes et ciblées
Fichiers courts (éviter les fichiers >300 lignes d'un coup)
Résumés manuels fréquents
Découper les projets en micro-tâches indépendantes
Utiliser des modèles spécialisés pour chaque type de tâche
```

---

## PARTIE 16 — STRATÉGIES COMBINÉES : SCÉNARIOS RÉELS

### Scenario 1 : Premier jour sur un nouveau projet

```
Étape 1 : Initialisation (5 min)
  → cd mon-projet && claude
  → /init                          # Génère CLAUDE.md automatiquement
  → Ajuster CLAUDE.md selon les besoins spécifiques
  → Créer .claudeignore

Étape 2 : Exploration (1 session dédiée)
  → "Donne-moi une vue d'ensemble de l'architecture"
  → /compact après avoir compris la structure
  → Note les décisions importantes en mémoire

Étape 3 : Sessions de travail ciblées
  → Une session = un module ou une feature
  → Nommer l'objectif dès le premier message
```

### Scenario 2 : Debugging d'un bug difficile

```
1. Session dédiée au bug
2. Fournir : message d'erreur FILTRÉ (pas le log complet)
3. Fichiers concernés PRÉCIS (pas "tout le projet")
4. Hack 95% : "Ne propose aucun code tant que tu n'es pas
   sûr à 95% de la cause. Pose des questions d'abord."
5. Après résolution → mémoriser la cause pour éviter récidive
```

### Scenario 3 : Génération massive (100 tests, 50 docs)

```
1. Utiliser HAIKU (4× moins cher que Sonnet)
2. Batch API si >20 fichiers (-50% coût supplémentaire)
3. Template dans CLAUDE.md pour le format attendu
4. Hors peak hours si possible
5. Générer par petits blocs (10 à la fois) pour valider la qualité
```

### Scenario 4 : Agent complexe multi-étapes (API)

```
1. Utiliser l'Advisor Tool : Sonnet exécuteur + Opus conseiller
2. Activer le cache advisor si >3 appels prévus
3. Ajouter un prompt système avec les timings d'appel advisor
4. Limiter la longueur des réponses advisor (100 mots max)
5. max_uses: 5 pour éviter les coûts non maîtrisés
```

### Scenario 5 : Session longue (journée de travail)

```
Toutes les ~45 minutes :
  → Vérifier le % de contexte (/context)
  → Si >60% → /compact
  → Si changement de sujet → résumé + /clear

Fin de journée :
  → Résumé structuré → sauvegarde mémoire
  → État des tâches en cours noté
  → Prochaine session : coller le résumé en premier message
```

---

## Tableau de Référence Rapide

### Commandes et Impact Token

| Action | Impact tokens | Quand utiliser |
|--------|--------------|----------------|
| `/init` | Génère CLAUDE.md | Démarrage de projet |
| `/compact` | Réduit contexte ×5-10 | À 60% de contexte |
| `/clear` | Remet à zéro | Changement de sujet |
| `/context` | Lecture seule | Vérifier l'état |
| `/model haiku` | ×4-19× moins cher | Tâches répétitives |
| CLAUDE.md | Économise répétitions | Toujours en place |
| .claudeignore | Évite exploration | Toujours en place |
| Sub-agents | ×2-3 coût, ×3 vitesse | Tâches indépendantes longues |
| Plan Mode | Évite corrections | Avant gros changements |
| Advisor Tool | ≈Opus qualité, ≈Sonnet coût | Agents complexes API |
| Batch API | -50% coût, async | Génération massive |

---

## Carte Mentale ASCII

```
              CLAUDE CODE — MAÎTRISE COMPLÈTE
                           │
         ┌─────────────────┼──────────────────┐
         ▼                 ▼                  ▼
    DÉMARRAGE          CONFIGURATION       CONTEXTE
         │                 │                  │
  Installation         CLAUDE.md          Relecture
  npm install -g       .claudeignore      exponentielle
  claude login         config.json        Lost in Middle
  /init → CLAUDE.md    Mémoire/           Prompt Cache 5min
                        memory/            /compact /clear
                                          Auto-compact 60%
                           │
         ┌─────────────────┼──────────────────┐
         ▼                 ▼                  ▼
    SUB-AGENTS          HOOKS              ADVISOR TOOL
    Parallèles          PreToolUse         (BETA 2026)
    Worktree isolé      PostToolUse        Executor+Advisor
    Coût ×2-3           Notification       Sonnet+Opus
    Vitesse ×3          Stop               ≈Opus, ≈Sonnet coût
                           │
         ┌─────────────────┼──────────────────┐
         ▼                 ▼                  ▼
      MODÈLES           TOKENS             AUTRES IAs
      Haiku (éco)       RTK filtre         GPT-4o Custom GPTs
      Sonnet (défaut)   Outputs limités    Gemini 1M ctx
      Opus (puissant)   Peak hours         Ollama (local, free)
      /model switch     Batch API -50%     LM Studio
                        Extended Thinking  OpenCode
```

---

## Exercices Pratiques

### Exercice 1 — Premier Setup Complet

1. Installer Claude Code sur ta machine
2. S'authentifier avec `claude login`
3. Choisir un de tes projets existants
4. Lancer `/init` et examiner le CLAUDE.md généré
5. Créer un `.claudeignore` adapté à la stack
6. Vérifier avec `/context` que le contexte est optimal

### Exercice 2 — Mesurer la Relecture Exponentielle

1. Ouvrir une nouvelle session
2. Envoyer 10 messages de ~50 mots chacun
3. Faire `/context` après chaque message (noter les tokens)
4. Calculer le coût réel vs. contenu utile
5. Recommencer en groupant les 10 questions en 1 message

### Exercice 3 — Pratiquer le Combo Résumé + /clear

1. Travailler 30 min sur un bug complexe
2. Demander un résumé structuré (décisions, état, prochaines étapes)
3. Faire `/clear`
4. Reprendre avec le résumé collé en premier message
5. Comparer l'efficacité avec une session sans /clear

### Exercice 4 — Comparer les Modèles sur une Vraie Tâche

1. Choisir une tâche répétitive (générer 10 tests unitaires)
2. Faire avec `/model haiku` → noter qualité et satisfaction
3. Faire avec `/model sonnet` → comparer
4. Décider si la différence justifie le coût ×4 pour ce type de tâche

### Exercice 5 — Créer un Hook Utile

1. Identifier une action répétitive dans ton workflow
   (ex: lancer les tests après chaque modification)
2. Écrire le hook PostToolUse correspondant dans `config.json`
3. Tester que le hook se déclenche correctement
4. Vérifier que l'output du hook ne pollue pas inutilement le contexte

---

## Liens

- [[01 - IA Assistants de Code]] — Panorama des assistants IA (Copilot, ChatGPT, Claude)
- [[02 - IA et Productivite Dev]] — Intégration CI/CD, génération de tests, agents
- [[04 - CI-CD avec GitHub Actions]] — Automatiser son pipeline avec Claude
- [[01 - Git et GitHub]] — Workflows Git assistés par IA (`/commit`, revues)
- [[07 - Python Async et Concurrence]] — Comprendre asyncio (utile pour l'API Claude)
- [[09 - APIs REST avec FastAPI]] — Construire ses propres agents sur l'API Claude
```
