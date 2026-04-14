# 08 - OpenCode et CLI Alternatifs

## Qu'est-ce que les agents CLI de codage ?

Les agents CLI de codage sont des programmes en ligne de commande qui combinent un modèle de langage (LLM) avec un accès direct au système de fichiers, au terminal et à Git. Contrairement à un simple chatbot, ces agents peuvent lire des fichiers, écrire du code, exécuter des commandes, créer des commits et naviguer dans un dépôt complet — le tout de manière autonome et interactive.

En 2024-2025, Claude Code (d'Anthropic) a popularisé ce paradigme. Depuis, une vague d'alternatives open source est apparue, chacune avec ses propres forces et philosophies. Cette note explore l'écosystème complet des CLI agents, avec un focus sur OpenCode et Aider.

> [!tip] Analogie
> Un agent CLI de codage, c'est comme avoir un développeur junior assis à côté de toi, avec accès à ton ordinateur. Il peut lire tout ton code, écrire des fichiers, lancer des tests — mais c'est toi qui décides ce qu'il fait. La différence entre les outils, c'est la qualité de ce "junior" et la puissance de ses outils.

---

## 1. Pourquoi les CLI agents sont puissants

### Accès au filesystem

Un agent CLI peut lire et écrire n'importe quel fichier de ton projet. Il comprend la structure de ton dépôt, pas seulement un fragment isolé de code copié-collé dans un chat.

```
Mon Projet/
├── src/
│   ├── main.py          ← L'agent peut lire tout ça
│   ├── utils.py
│   └── models/
│       └── parser.py
├── tests/
│   └── test_main.py     ← Et ça aussi
└── README.md
```

Quand tu demandes "Ajoute des tests pour la fonction parse_csv", l'agent lit `src/utils.py`, comprend la fonction, puis écrit dans `tests/test_main.py` avec le bon style.

### Accès au terminal

L'agent peut exécuter des commandes : lancer les tests, vérifier les erreurs de lint, compiler le projet, etc. Il voit les résultats et peut corriger son propre code en conséquence.

```
Cycle agent :
 ┌─────────────────────────────────────┐
 │  1. Écrit le code                   │
 │  2. Exécute les tests               │
 │  3. Lit les erreurs                 │
 │  4. Corrige automatiquement         │
 │  5. Re-exécute jusqu'au succès      │
 └─────────────────────────────────────┘
```

### Accès à Git

Lecture de l'historique des commits, comparaison de diffs, création de branches — l'agent comprend l'évolution de ton projet dans le temps, pas seulement son état actuel.

> [!info]
> C'est la combinaison de ces trois accès (filesystem + terminal + git) qui distingue fondamentalement un agent CLI d'un copilote IDE classique comme GitHub Copilot. Copilot suggère, l'agent agit.

---

## 2. L'écosystème : Vue d'ensemble

```
CLI AGENTS DE CODAGE - Paysage 2025
══════════════════════════════════════════════════════

  PROPRIÉTAIRE                    OPEN SOURCE
  ┌─────────────────┐             ┌──────────────────────────────────────┐
  │   Claude Code   │             │  OpenCode  │  Aider  │ Goose │  Amp  │
  │   (Anthropic)   │             │            │         │       │       │
  └─────────────────┘             └──────────────────────────────────────┘
        │                                │          │       │       │
        │ Claude uniquement              │ Multi-   │ Git   │ MCP   │ Grand
        │ MCP + Hooks                    │ modèle   │ First │ Focus │ Repo
        │ Très stable                    │          │       │       │
        └────────────────────────────────┴──────────┴───────┴───────┘
                                  TOUS supportent Ollama (IA locale)
```

---

## 3. OpenCode

### Qu'est-ce qu'OpenCode ?

OpenCode est un agent CLI open source, multi-modèle, conçu pour être une alternative flexible à Claude Code. Il supporte tout fournisseur compatible avec l'API OpenAI, ce qui inclut Anthropic, OpenAI, Google Gemini, et les modèles locaux via Ollama.

- **Dépôt GitHub** : `https://github.com/opencode-ai/opencode`
- **Langage** : Go (binaire unique, pas de dépendances Python)
- **Licence** : MIT

> [!tip] Analogie
> OpenCode, c'est comme un couteau suisse multi-modèles : là où Claude Code ne fonctionne qu'avec Claude, OpenCode peut brancher n'importe quel LLM, du cloud ou local. Tu gardes le même outil, tu changes juste la "lame".

### Installation

```bash
# Via npm (méthode rapide)
npm install -g opencode-ai

# Via script d'installation officiel (recommandé)
curl -fsSL https://opencode.ai/install | sh

# Sur Windows (PowerShell)
irm https://opencode.ai/install.ps1 | iex

# Vérifier l'installation
opencode --version
```

### Configuration

Le fichier de configuration principal se trouve dans `~/.opencode/config.json`.

```json
// ~/.opencode/config.json
{
  "model": "claude-sonnet-4-6",
  "providers": {
    "anthropic": {
      "api_key": "sk-ant-..."
    },
    "openai": {
      "api_key": "sk-..."
    },
    "google": {
      "api_key": "AIza..."
    },
    "ollama": {
      "base_url": "http://localhost:11434"
    }
  },
  "theme": "dark",
  "auto_compact": true
}
```

> [!info]
> Contrairement à Claude Code qui utilise `CLAUDE.md` pour les instructions du projet, OpenCode utilise un fichier `opencode.md` ou `.opencode/instructions.md` à la racine du projet. Si tu migres depuis Claude Code, pense à créer ce fichier.

### Utilisation de base

```bash
# Démarrer une session interactive (mode TUI)
opencode

# Commande directe sans session interactive
opencode "Explique la structure de ce projet"

# Choisir un modèle spécifique
opencode --model claude-sonnet-4-6
opencode --model gpt-4o
opencode --model ollama/qwen2.5-coder:14b

# Travailler dans un répertoire spécifique
opencode --dir /chemin/vers/mon-projet

# Mode verbose pour voir les appels d'outils
opencode --verbose "Refactorise la classe UserManager"
```

### Commandes internes (en session interactive)

```
/help           Afficher l'aide
/model          Changer de modèle à la volée
/files          Lister les fichiers chargés en contexte
/clear          Vider le contexte
/cost           Voir le coût de la session en cours
/compact        Compresser le contexte (résumé)
/exit           Quitter
```

### Avantages vs Claude Code

```
OPENCODE vs CLAUDE CODE - Avantages OpenCode
═══════════════════════════════════════════════
  ✓ Open source : tu peux inspecter le code source
  ✓ Multi-fournisseur : change de modèle sans changer d'outil
  ✓ Compatible Ollama : IA locale gratuite et privée
  ✓ Pas de lock-in Anthropic : liberté de choix
  ✓ Gratuit hors coût API
  ✓ Communauté open source active
  ✓ Binaire Go : léger, pas de runtime lourd
  ✓ Fonctionne si Anthropic a des problèmes de disponibilité
```

### Inconvénients vs Claude Code

```
OPENCODE vs CLAUDE CODE - Inconvénients OpenCode
══════════════════════════════════════════════════
  ✗ Pas d'Advisor Tool (analyse sans modification)
  ✗ Hooks limités (pas d'équivalent hooks Claude Code)
  ✗ CLAUDE.md non reconnu (migration à faire)
  ✗ Moins testé en production large
  ✗ Fonctionnalités MCP moins matures
  ✗ Support officiel inexistant (communauté uniquement)
  ✗ Mises à jour moins fréquentes
```

> [!warning]
> OpenCode est un projet encore jeune. Pour un projet professionnel critique, Claude Code reste plus fiable. Utilise OpenCode pour tes projets personnels, l'exploration de modèles alternatifs, ou quand tu veux 100% de contrôle sur l'outil.

---

## 4. Aider

### Qu'est-ce qu'Aider ?

Aider est un agent CLI centré sur Git, créé par Paul Gauthier. C'est l'un des projets open source les plus matures de l'écosystème — il existe depuis 2023 et est utilisé en production par des milliers de développeurs.

Sa particularité : **Aider crée automatiquement un commit Git pour chaque changement qu'il effectue**. Cette approche "Git-first" le distingue fondamentalement de tous les autres agents.

- **Dépôt GitHub** : `https://github.com/Aider-AI/aider`
- **Langage** : Python
- **Licence** : Apache 2.0
- **Popularité** : Plus de 20 000 étoiles GitHub

> [!tip] Analogie
> Aider ressemble à un développeur qui commit après chaque tâche terminée, avec un message de commit bien rédigé. À la fin de la journée, ton historique Git est propre et chaque changement IA est traçable, réversible et documenté.

### Installation

```bash
# Installation via pip (Python 3.9+)
pip install aider-chat

# Installation recommandée dans un virtualenv
python -m venv venv-aider
source venv-aider/bin/activate   # Linux/Mac
# ou
venv-aider\Scripts\activate      # Windows
pip install aider-chat

# Vérifier l'installation
aider --version

# Mise à jour
pip install --upgrade aider-chat
```

### Utilisation de base

```bash
# Démarrer Aider dans un repo Git (requis !)
cd mon-projet
aider

# Avec un modèle spécifique
aider --model gpt-4o
aider --model claude-sonnet-4-6
aider --model ollama/qwen2.5-coder:14b

# Ouvrir directement avec des fichiers en contexte
aider src/main.py src/utils.py

# Mode "watch" : Aider surveille les fichiers et propose des améliorations
aider --watch-files
```

### Commandes internes en session Aider

```bash
# Dans la session interactive :
/add src/main.py              # Ajouter un fichier au contexte
/drop src/utils.py            # Retirer un fichier du contexte
/files                        # Lister les fichiers en contexte
/git log --oneline -5         # Exécuter des commandes git
/diff                         # Voir le diff du dernier changement
/undo                         # Annuler le dernier commit Aider
/run python tests/test_main.py  # Exécuter une commande
/ask "Explique ce code"       # Question sans modification
/architect "Refactorise X"    # Mode architecte (planification)
/exit                         # Quitter
```

### Fonctionnalité unique : Auto-commits Git

C'est la killer feature d'Aider. Chaque fois qu'Aider modifie un fichier, il crée automatiquement un commit Git avec un message généré par l'IA.

```
Workflow Aider avec auto-commits :

Avant Aider :
  git log --oneline
  abc1234 feat: ajoute authentification JWT

Après une session Aider :
  git log --oneline
  def5678 aider: ajoute gestion d'erreur dans parse_csv
  ghi9012 aider: refactorise UserManager en plusieurs classes
  jkl3456 aider: ajoute tests unitaires pour parse_csv
  abc1234 feat: ajoute authentification JWT

→ Chaque changement IA est tracé, documenté, réversible
```

```bash
# Annuler le dernier changement Aider
git revert HEAD

# Ou directement dans Aider
/undo

# Voir tous les changements Aider
git log --oneline --grep="aider:"
```

> [!info]
> Cette approche est idéale pour les équipes : les reviewers peuvent voir exactement ce que l'IA a fait, commit par commit. C'est bien plus transparent qu'une modification massive sans historique.

### Modes de travail Aider

```
MODES D'AIDER
═════════════════════════════════════════════
  /ask      → Question sans modification de code
              "Comment fonctionne cette classe ?"

  /architect → Planification haut niveau
               L'IA réfléchit avant de coder

  (défaut)  → Mode édition direct
               L'IA modifie les fichiers directement

  --watch   → Mode surveillance des fichiers
               Aider lit les commentaires "AI:" que
               tu laisses dans le code et agit dessus
```

### Configuration `.aider.conf.yml`

```yaml
# .aider.conf.yml (à la racine de ton projet)
model: claude-sonnet-4-6
auto-commits: true
dark-mode: true
pretty: true
stream: true
auto-lint: true           # Lance le linter après chaque changement
auto-test: false          # Lance les tests automatiquement (optionnel)
test-cmd: pytest          # Commande de test
lint-cmd: ruff check      # Commande de lint
```

```bash
# Configuration globale dans ~/.aider.conf.yml
model: claude-sonnet-4-6
dark-mode: true
```

> [!warning]
> Aider nécessite que le répertoire courant soit un dépôt Git. Si tu lances Aider hors d'un repo Git, il te proposera d'en créer un. Pour les projets sans Git, utilise OpenCode à la place.

---

## 5. Goose (Block)

### Qu'est-ce que Goose ?

Goose est un agent CLI open source développé par Block (anciennement Square, la société de Jack Dorsey). Il se distingue par son support natif des serveurs MCP (Model Context Protocol) et son approche multimodale.

```
GOOSE - Points clés
═══════════════════
  Développeur  : Block (anciennement Square)
  Dépôt        : https://github.com/block/goose
  Licence      : Apache 2.0
  Langage      : Rust
  Force        : Support MCP natif + multimodal
  Faiblesse    : Moins mature qu'Aider
```

```bash
# Installation
curl -fsSL https://github.com/block/goose/releases/latest/download/install.sh | sh

# Démarrer
goose session start

# Avec un modèle spécifique
goose session start --model claude-sonnet-4-6
```

Goose supporte les extensions MCP, ce qui lui permet de se connecter à des outils externes : bases de données, APIs, services cloud — exactement comme Claude Code avec ses MCP servers.

---

## 6. Amp (Sourcegraph)

### Qu'est-ce qu'Amp ?

Amp est un agent CLI développé par Sourcegraph, la société connue pour son moteur de recherche de code. Sa particularité : il indexe ton codebase entier et comprend les relations entre fichiers à grande échelle.

```
AMP - Points clés
═══════════════════
  Développeur  : Sourcegraph
  Site         : https://ampcode.com
  Force        : Indexation codebase + grands repos
  Idéal pour   : Monorepos, projets > 100 000 lignes
  Faiblesse    : Plus complexe à configurer
```

> [!info]
> Amp est particulièrement adapté aux équipes travaillant sur de très grandes bases de code, où comprendre les relations entre des centaines de fichiers est crucial. Pour des projets de taille normale, OpenCode ou Aider suffisent amplement.

---

## 7. Comparaison complète des CLI agents

```
TABLEAU COMPARATIF - CLI AGENTS DE CODAGE 2025
═══════════════════════════════════════════════════════════════════════════
  Critère          Claude Code    OpenCode    Aider       Goose    Amp
  ─────────────────────────────────────────────────────────────────────
  Open Source      Non            Oui         Oui         Oui      Non*
  Multi-modèle     Non            Oui         Oui         Oui      Oui
  Git intégré      Oui            Partiel     Oui (auto)  Non      Oui
  MCP Support      Oui            Non         Non         Oui      Non
  Hooks système    Oui            Limité      Non         Non      Non
  Ollama/Local     Non            Oui         Oui         Oui      Non
  CLAUDE.md        Oui            Non         Non         Non      Non
  Prix             Usage API      Gratuit+API Gratuit+API Gratuit+ Commercial
                                                          API
  Maturité         Haute          Moyenne     Haute       Moyenne  Moyenne
  Langage source   N/A            Go          Python      Rust     TypeScript
  ─────────────────────────────────────────────────────────────────────
  *Amp a une version community open source partielle
```

| Outil | Open Source | Multi-modèle | Git intégré | MCP | Prix |
|-------|-------------|--------------|-------------|-----|------|
| Claude Code | Non | Non | Oui | Oui | Usage API |
| OpenCode | Oui | Oui | Partiel | Non | Gratuit + API |
| Aider | Oui | Oui | Oui (auto-commit) | Non | Gratuit + API |
| Goose | Oui | Oui | Non | Oui | Gratuit + API |
| Amp | Partiel | Oui | Oui | Non | Commercial |

---

## 8. Utiliser un CLI agent avec des modèles locaux (Ollama)

### Pourquoi utiliser Ollama avec un CLI agent ?

```
CLOUD vs LOCAL pour CLI agents
═══════════════════════════════════
  CLOUD (Claude, GPT-4...)       LOCAL (Ollama)
  ┌─────────────────────┐        ┌─────────────────────┐
  │ + Meilleure qualité │        │ + Gratuit (0€)      │
  │ + Grand contexte    │        │ + 100% privé        │
  │ - Coût API          │        │ - Qualité moindre   │
  │ - Données envoyées  │        │ - Contexte limité   │
  │   vers cloud        │        │ - Nécessite GPU     │
  └─────────────────────┘        └─────────────────────┘
```

### Meilleurs modèles locaux pour agents CLI

```
MODÈLES LOCAUX RECOMMANDÉS (Ollama) - Avril 2025
══════════════════════════════════════════════════
  Modèle                    RAM req.  Qualité  Usage recommandé
  ──────────────────────────────────────────────────────────────
  qwen2.5-coder:32b         20 GB     ★★★★★   Meilleure qualité
                                               Projets sérieux
  qwen2.5-coder:14b         10 GB     ★★★★☆   Bon équilibre
                                               Usage quotidien
  qwen2.5-coder:7b          5 GB      ★★★☆☆   Rapide
                                               Questions simples
  deepseek-coder-v2:16b     12 GB     ★★★★☆   Bon pour code
                                               Concurrent solide
  codestral:22b             15 GB     ★★★★☆   Mistral AI
                                               Excellent pour Python
```

### Configuration OpenCode + Ollama

```bash
# 1. Installer et démarrer Ollama
ollama serve
ollama pull qwen2.5-coder:14b

# 2. Configurer OpenCode
# ~/.opencode/config.json
```

```json
{
  "model": "ollama/qwen2.5-coder:14b",
  "providers": {
    "ollama": {
      "base_url": "http://localhost:11434"
    }
  }
}
```

```bash
# 3. Lancer OpenCode avec Ollama
opencode --model ollama/qwen2.5-coder:14b "Analyse ce projet"

# Changer de modèle en cours de session
# Dans OpenCode : /model ollama/qwen2.5-coder:32b
```

### Configuration Aider + Ollama

```bash
# Lancer Aider avec Ollama directement
aider --model ollama/qwen2.5-coder:14b

# Configuration dans .aider.conf.yml
```

```yaml
# .aider.conf.yml
model: ollama/qwen2.5-coder:14b
ollama-api-base: http://localhost:11434
auto-commits: true
```

```bash
# Vérifier la connexion Ollama depuis Aider
aider --model ollama/qwen2.5-coder:14b --check-model
```

### Limites des modèles locaux dans un agent CLI

> [!warning]
> Les modèles locaux sont beaucoup moins capables que Claude Sonnet ou GPT-4o pour les tâches d'agent complexes. Voici les limitations typiques :
>
> - **Contexte réduit** : 8k-32k tokens vs 200k pour Claude — l'agent ne peut pas lire tout un grand projet
> - **Suivi d'instructions** : moins fiable pour les formats structurés (les patches de code peuvent être malformés)
> - **Raisonnement multi-étapes** : tâches complexes nécessitant de la planification sont moins bien gérées
> - **Langues** : moins bon en français pour les commentaires et la documentation

```
QUAND UTILISER OLLAMA LOCAL vs CLOUD
══════════════════════════════════════
  LOCAL (Ollama) idéal pour :
    ✓ Code propriétaire/confidentiel (0 donnée cloud)
    ✓ Refactoring simple et répétitif
    ✓ Projets personnels avec budget serré
    ✓ Expérimentation sans coût

  CLOUD (Claude/GPT) nécessaire pour :
    ✓ Tâches complexes multi-fichiers
    ✓ Debugging difficile
    ✓ Génération de code from scratch
    ✓ Qualité professionnelle requise
```

---

## 9. Intégration dans le workflow

### Quand passer de Claude Code à une alternative ?

```
ARBRE DE DÉCISION - Choix d'un CLI agent
══════════════════════════════════════════════════════

  Tu veux coder avec un agent CLI ?
                │
                ▼
  Tu utilises Claude régulièrement ?
         │              │
        Oui             Non
         │              │
         ▼              ▼
    Claude Code    Tu veux multi-modèle cloud ?
    (référence)         │              │
                       Oui             Non
                        │              │
                        ▼              ▼
                    OpenCode    Tu veux 100% local/gratuit ?
                                    │              │
                                   Oui             Non
                                    │              │
                                    ▼              ▼
                            OpenCode ou       Tu travailles
                            Aider + Ollama    beaucoup avec Git ?
                                                   │         │
                                                  Oui        Non
                                                   │         │
                                                   ▼         ▼
                                                Aider    Tu as un
                                                         grand repo ?
                                                            │      │
                                                           Oui     Non
                                                            │      │
                                                            ▼      ▼
                                                           Amp   Goose
```

### Workflow quotidien avec Aider + Claude API

```bash
# Matin : démarrer la session de travail
cd mon-projet
aider --model claude-sonnet-4-6

# Dans Aider :
/add src/feature.py tests/test_feature.py

# Développement :
"Implémente la méthode process_data selon la docstring"
# → Aider écrit le code + commit automatique

# Test :
/run pytest tests/test_feature.py -v
# → Aider voit les résultats et corrige si nécessaire

# Fin de session :
/exit
git log --oneline -5  # Voir tous les commits IA de la session
```

### Workflow : Code open source avec Ollama (gratuit total)

```bash
# Cas : contribuer à un projet open source sans dépenser d'argent

# 1. Cloner le repo
git clone https://github.com/user/projet-open-source
cd projet-open-source

# 2. Démarrer Ollama
ollama serve &
ollama pull qwen2.5-coder:14b

# 3. Lancer Aider gratuit
aider --model ollama/qwen2.5-coder:14b

# Dans Aider :
/add src/module.py
"Corrige le bug dans la fonction load_config (voir issue #234)"
# Coût total : 0€ (juste ta consommation électrique)
```

### Workflow : Code propriétaire avec Ollama (100% privé)

```bash
# Cas : startup avec code confidentiel, 0 donnée cloud

# Architecture :
# ┌────────────────────────────────────────────────────┐
# │          TON ORDINATEUR (rien ne sort)             │
# │                                                    │
# │  Code propriétaire  →  Aider  →  Ollama local      │
# │                         ↑           ↓              │
# │                    qwen2.5-coder:32b               │
# │                    (tourne localement)             │
# └────────────────────────────────────────────────────┘

# Configuration .aider.conf.yml pour cette équipe
```

```yaml
model: ollama/qwen2.5-coder:32b
ollama-api-base: http://localhost:11434
auto-commits: true
auto-lint: true
lint-cmd: ruff check --fix
# Aucune clé API cloud → aucune donnée sort de la machine
```

### Comparaison des coûts sur un mois (exemple)

```
ESTIMATION COÛT MENSUEL - Développeur actif
═════════════════════════════════════════════
  Scénario          Outil              Coût/mois
  ─────────────────────────────────────────────
  Usage intensif    Claude Code        ~30-80€
  Usage modéré      Aider + Claude     ~15-40€
  Budget serré      OpenCode + Ollama  ~0€ + GPU
  100% cloud-free   Aider + Ollama     ~0€ + GPU
  Grand projet      Amp                ~50-200€

  Note : GPU consommation ~0.02€/h en plus sur facture électricité
```

---

## 10. Guide de décision final

> [!example] Scénarios concrets de choix
>
> **Scénario A** : Développeur solo sur des projets variés, habitué à Claude
> → **Claude Code** pour la qualité, **Aider + Ollama** pour les projets perso
>
> **Scénario B** : Startup avec code confidentiel, pas de budget cloud
> → **Aider + Ollama local** (qwen2.5-coder:32b si bon GPU)
>
> **Scénario C** : Développeur voulant tester différents modèles (GPT, Gemini, Claude)
> → **OpenCode** : même interface, change de modèle en une commande
>
> **Scénario D** : Équipe avec processus Git rigoureux
> → **Aider** : auto-commits, traçabilité parfaite, intégration CI/CD
>
> **Scénario E** : Ingénieur sur un monorepo de 500 000 lignes
> → **Amp** : indexation et compréhension des grandes bases de code

```
RÉSUMÉ DES FORCES DE CHAQUE OUTIL
════════════════════════════════════════════════════════
  Claude Code  → Meilleure qualité, MCP, hooks, stable
  OpenCode     → Flexibilité multi-modèle, open source
  Aider        → Git-first, auto-commits, très mature
  Goose        → MCP natif, multimodal, approche Block
  Amp          → Grands repos, indexation codebase
  + Ollama     → Gratuit, privé, local (avec tous sauf Claude Code)
```

---

## 11. Conseils pratiques de migration

### Passer de Claude Code à Aider

```bash
# 1. Convertir CLAUDE.md en instructions Aider
cp CLAUDE.md .aider.md   # Aider utilise .aider.md comme contexte

# 2. Vérifier la configuration
cat .aider.conf.yml

# 3. Première session de test
aider --model claude-sonnet-4-6
/ask "Lis le fichier .aider.md et décris ce que tu comprends du projet"
```

### Passer de Claude Code à OpenCode

```bash
# 1. Installer OpenCode
curl -fsSL https://opencode.ai/install | sh

# 2. Migrer les instructions projet
# CLAUDE.md → opencode.md (contenu compatible)
cp CLAUDE.md opencode.md
# Ajuster le format si nécessaire

# 3. Configurer les clés API
# Les mêmes clés Anthropic fonctionnent dans OpenCode
```

> [!tip] Conseil pratique
> Il n'est pas nécessaire de choisir un seul outil. Beaucoup de développeurs utilisent Claude Code pour le travail intensif professionnel et Aider + Ollama pour leurs projets personnels. Les deux peuvent coexister sur la même machine sans conflit.

---

## Carte Mentale ASCII

```
                    CLI AGENTS DE CODAGE
                           │
          ┌────────────────┼────────────────┐
          │                │                │
     PROPRIÉTAIRE      OPEN SOURCE       LOCAL
          │                │                │
    Claude Code     ┌──────┼──────┐      Ollama
          │         │      │      │         │
       Forces    OpenCode Aider  Goose   Modèles
       ·MCP      ·Multi   ·Git   ·MCP    ·qwen2.5
       ·Hooks    ·modèle  ·Auto  ·Block  ·deepseek
       ·Stable   ·Go      ·commit        ·codestral
                 ·MIT     ·Python
                          ·Apache
          │                │                │
          └────────────────┼────────────────┘
                           │
                    CHOISIR SELON :
                    ·Budget (cloud vs local)
                    ·Confidentialité
                    ·Intégration Git
                    ·Taille du projet
                    ·Multi-modèle needed
```

---

## Exercices pratiques

### Exercice 1 : Installer et comparer Aider vs Claude Code

**Objectif** : Ressentir concrètement la différence entre les deux outils.

1. Crée un petit projet Python avec un bug intentionnel :
```python
# buggy_calculator.py
def divide(a, b):
    return a / b  # Bug : pas de gestion division par zéro

def main():
    print(divide(10, 0))  # Provoque une ZeroDivisionError

if __name__ == "__main__":
    main()
```

2. Initialise un repo Git : `git init && git add . && git commit -m "init"`
3. Lance Aider : `aider --model claude-sonnet-4-6 buggy_calculator.py`
4. Demande : "Corrige le bug et ajoute des tests unitaires"
5. Observe les auto-commits : `git log --oneline`
6. Compare avec ce que Claude Code ferait sur le même fichier
7. Note les différences de comportement et de qualité

**Questions de réflexion** : Les commits d'Aider sont-ils bien formés ? La qualité du code est-elle similaire à Claude Code ? Quelle approche préfères-tu ?

---

### Exercice 2 : Configurer OpenCode avec un modèle Ollama

**Objectif** : Mettre en place un workflow 100% local et gratuit.

1. Installe Ollama si ce n'est pas fait : `ollama serve`
2. Télécharge un modèle de code : `ollama pull qwen2.5-coder:7b`
3. Installe OpenCode : `npm install -g opencode-ai`
4. Configure `~/.opencode/config.json` avec Ollama comme fournisseur
5. Lance OpenCode : `opencode --model ollama/qwen2.5-coder:7b`
6. Demande-lui d'analyser un projet existant sur ton disque
7. Compare la qualité de réponse avec Claude Sonnet sur la même question

**Défi supplémentaire** : Teste le même prompt avec `qwen2.5-coder:7b`, `qwen2.5-coder:14b`, et `claude-sonnet-4-6`. Note les différences de qualité et de vitesse. Dans quel cas le modèle local suffit ?

---

### Exercice 3 : Workflow Git avec Aider pour une vraie fonctionnalité

**Objectif** : Utiliser Aider dans un vrai workflow de développement.

1. Crée un projet Python avec structure :
```
mon-app/
├── src/
│   ├── __init__.py
│   └── data_processor.py
└── tests/
    └── test_data_processor.py
```

2. Dans `data_processor.py`, écris une classe vide :
```python
class DataProcessor:
    """Traite des fichiers CSV et retourne des statistiques."""
    pass
```

3. Lance Aider : `aider src/data_processor.py tests/test_data_processor.py`
4. Demande successivement :
   - "Implémente la classe DataProcessor avec les méthodes load_csv, get_stats, filter_rows"
   - "Ajoute des tests unitaires complets pour chaque méthode"
   - "Ajoute une gestion d'erreur robuste et des docstrings"
5. Après chaque étape, fais `git log --oneline` et `git show HEAD` pour voir le commit

**Analyse** : Combien de commits Aider a-t-il créés ? Les messages sont-ils lisibles ? Essaie `/undo` pour annuler le dernier changement. Est-ce que le workflow Git t'a semblé naturel ou contraignant ?

---

## Voir aussi

- [[04 - Claude et Claude Code Maitrise Avancee]]
- [[07 - Integrations IDE et Extensions]]
- [[09 - IA Locale avec Ollama]]
- [[12 - Strategies Multi-Modeles et Workflows]]
- [[06 - MCP Model Context Protocol]]
