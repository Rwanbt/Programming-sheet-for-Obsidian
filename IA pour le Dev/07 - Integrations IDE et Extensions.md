# 07 - Intégrations IDE et Extensions IA

## Qu'est-ce que l'intégration IA dans un IDE ?

L'intégration IA dans un IDE (Integrated Development Environment) consiste à connecter des modèles de langage directement à ton éditeur de code, pour obtenir de l'aide contextuelle sans quitter ton flux de travail. Plutôt que de copier-coller du code vers une interface web, l'IA voit directement ce que tu écris, comprend le contexte de ton projet, et peut agir sur tes fichiers.

En 2025-2026, l'intégration IA est devenue le **point d'entrée principal** pour la majorité des développeurs : plus d'interface séparée, l'assistant est là, dans la sidebar ou directement dans le curseur d'édition.

> [!tip] Analogie
> L'IA dans ton IDE, c'est comme avoir un binôme expert qui regarde ton écran en permanence. Il ne prend pas le clavier à ta place (sauf si tu le lui demandes), mais il anticipe ce que tu vas taper, répond à tes questions sur ton propre code, et peut réécrire des sections entières quand tu le lui demandes.

---

## 1. Introduction : L'IA dans ton éditeur

### Pourquoi l'intégration IDE est cruciale

Copier-coller du code vers ChatGPT ou Claude.ai fonctionne, mais présente des limites importantes :
- Tu perds le contexte global du projet
- Tu dois reformater et réintégrer les réponses manuellement
- L'IA ne voit pas les erreurs du compilateur ou du linter
- Le flux de travail est brisé à chaque échange

Avec une intégration native, l'IA a accès à :
- Le fichier actuellement ouvert
- Les fichiers liés et importés
- L'historique du terminal
- Les erreurs de compilation en temps réel
- Tout le codebase (selon l'outil)

### Les 3 niveaux d'intégration

```
NIVEAU 1 : COMPLÉTION INLINE
┌─────────────────────────────────────────────────┐
│  function calculateTotal(items) {               │
│    return items.reduce((sum, item) =>           │
│      sum + item.price * item.quantity, 0)  ◄─── │
│  }                    ↑                         │
│              suggestion grisée                  │
│              Tab pour accepter                  │
└─────────────────────────────────────────────────┘
  Latence : ~100-500ms  |  Contexte : fichier local

NIVEAU 2 : CHAT LATÉRAL
┌────────────────────┬────────────────────────────┐
│   MON CODE         │   CHAT IA                  │
│                    │                            │
│  function foo() {  │  Toi: "Explique cette fn"  │
│    // ...          │                            │
│  }                 │  IA: "Cette fonction..."   │
│                    │                            │
└────────────────────┴────────────────────────────┘
  Interaction : question/réponse  |  Contexte partagé

NIVEAU 3 : AGENT AUTONOME
┌─────────────────────────────────────────────────┐
│  AGENT                                          │
│  ├── Lit fichier A                              │
│  ├── Modifie fichier B                          │
│  ├── Crée fichier C                             │
│  ├── Exécute test                               │
│  └── Corrige erreur → relance test              │
└─────────────────────────────────────────────────┘
  Autonomie élevée  |  Contexte : projet entier
```

### Vue d'ensemble des options en 2025-2026

| Catégorie | Exemples | Niveau |
|-----------|----------|--------|
| Extensions VS Code | Copilot, Continue, Cline, Gemini | 1+2+3 |
| IDE fork | Cursor, Windsurf | 1+2+3 (natif) |
| IDE propriétaire | JetBrains AI | 1+2 |
| CLI agents | Claude Code, Aider, OpenCode | 3 |

> [!info]
> Ce fichier se concentre sur les intégrations **dans l'éditeur**. Pour les outils CLI, voir [[08 - OpenCode et CLI Alternatifs]].

---

## 2. VS Code — L'écosystème IA

VS Code est l'IDE le plus utilisé au monde, et son marketplace d'extensions est devenu le terrain de jeu principal pour les intégrations IA.

```
ÉCOSYSTÈME VS CODE IA (2025-2026)
═══════════════════════════════════════════════════════

  COMPLÉTION INLINE         CHAT / AGENT
  ─────────────────         ────────────
  ┌─────────────┐           ┌─────────────────────┐
  │  Copilot    │◄──────────│  GitHub Copilot Chat│
  │  (GitHub)   │           └─────────────────────┘
  └─────────────┘
  ┌─────────────┐           ┌─────────────────────┐
  │  Continue   │◄──────────│  Continue Chat      │
  │  (open src) │           │  (multi-modèle)     │
  └─────────────┘           └─────────────────────┘
  ┌─────────────┐           ┌─────────────────────┐
  │  Supermaven │           │  Cline (agent)      │
  │  (rapide)   │           │  lit+modifie fichiers│
  └─────────────┘           └─────────────────────┘
  ┌─────────────┐           ┌─────────────────────┐
  │  Tabnine    │           │  Claude Code Ext.   │
  │  (historique│           │  (Anthropic)        │
  └─────────────┘           └─────────────────────┘
                            ┌─────────────────────┐
  ┌─────────────┐           │  Gemini Code Assist │
  │  Gemini     │◄──────────│  (Google, gratuit)  │
  │  inline     │           └─────────────────────┘
  └─────────────┘

  LOCAL / OFFLINE
  ───────────────
  Continue.dev + Ollama = full local (confidentialité max)
  Tabnine Enterprise = self-hosted
```

---

### GitHub Copilot (Microsoft/GitHub)

Le pionnier des assistants IA pour développeurs, lancé en 2021 et désormais intégré profondément dans l'écosystème Microsoft.

**Installation et configuration**

1. Ouvrir VS Code
2. Extensions (`Ctrl+Shift+X`) → chercher "GitHub Copilot"
3. Installer **GitHub Copilot** + **GitHub Copilot Chat**
4. Se connecter avec son compte GitHub
5. Vérifier que la licence est active (personal, student, ou entreprise)

**Fonctionnalités principales**

- **Complétion inline** : suggestions grisées pendant la frappe, acceptées avec `Tab`
- **Copilot Chat** : sidebar de conversation (`Ctrl+Shift+I`)
- **Inline Chat** : chat directement dans le fichier (`Ctrl+I`)
- **Terminal** : explication et génération de commandes shell
- **Tests** : génération automatique de tests unitaires
- **Fix** : correction d'erreurs depuis le menu contextuel

**Raccourcis clavier essentiels**

| Action | Raccourci |
|--------|-----------|
| Accepter suggestion | `Tab` |
| Suggestion suivante | `Alt+]` |
| Suggestion précédente | `Alt+[` |
| Rejeter suggestion | `Escape` |
| Ouvrir Copilot Chat | `Ctrl+Shift+I` |
| Inline Chat | `Ctrl+I` |
| Suggestions dans un panneau | `Ctrl+Enter` |

**Paramètres recommandés dans `settings.json`**

```json
{
  "github.copilot.enable": {
    "*": true,
    "markdown": false,
    "plaintext": false,
    "yaml": true
  },
  "github.copilot.advanced": {
    "length": 500,
    "temperature": 0
  },
  "github.copilot.chat.localeOverride": "fr",
  "editor.inlineSuggest.enabled": true
}
```

> [!info]
> `"markdown": false` évite les suggestions intempestives dans les fichiers de documentation. `temperature: 0` donne des suggestions plus déterministes — utile pour le code de production.

**Prix**

- **Individual** : $10/mois ou $100/an
- **Étudiant** : gratuit via GitHub Education (justificatif scolaire requis)
- **Business** : $19/utilisateur/mois (politiques d'organisation)
- **Enterprise** : $39/utilisateur/mois (fine-tuning, audit logs)

> [!warning]
> La formule "gratuite" annoncée fin 2024 est limitée à 2000 complétions et 50 messages/mois. Pour un usage sérieux, il faut une licence payante.

---

### Claude Code Extension (Anthropic)

Extension officielle d'Anthropic pour VS Code, distincte de l'outil CLI Claude Code.

**Installation**

Extensions (`Ctrl+Shift+X`) → chercher "Claude Code" par Anthropic → Installer

**Configuration**

1. Ouvrir les settings (`Ctrl+,`)
2. Rechercher "Claude Code"
3. Entrer la clé API Anthropic (`sk-ant-...`) dans le champ dédié
4. Choisir le modèle (Sonnet recommandé pour le rapport qualité/coût)

**Fonctionnalités**

- Fenêtre latérale avec interface de chat
- Le fichier actuellement actif est automatiquement passé en contexte
- Sélectionner du code + demander une action (refactor, expliquer, corriger)
- Accès aux modèles Claude Sonnet et Opus

**Différence avec Claude Code CLI**

```
EXTENSION VS CODE          CLI CLAUDE CODE
───────────────────        ───────────────
Chat dans sidebar          Agent terminal
Contexte = fichier actif   Contexte = projet entier
Pas d'exécution auto       Peut exécuter des commandes
Interface graphique        Interface terminal
Idéal : questions/édition  Idéal : tâches automatisées
```

> [!info]
> Pour des tâches complexes nécessitant de modifier plusieurs fichiers ou d'exécuter des commandes, Claude Code CLI est plus adapté. L'extension VS Code est idéale pour l'exploration et les questions contextuelles.

---

### Continue.dev (open source, multi-modèle)

Continue.dev est l'extension IA la plus flexible pour VS Code. Open source, elle supporte pratiquement tous les fournisseurs de modèles.

> [!tip] Analogie
> Continue.dev est comme un adaptateur universel : tu branches le modèle de ton choix (Claude, GPT, Gemini, ou un modèle local via Ollama), et tu obtiens les mêmes fonctionnalités de chat et de complétion quelle que soit la source.

**Installation**

```
Extensions (Ctrl+Shift+X) → chercher "Continue" → Installer
```

Après l'installation, un onglet "Continue" apparaît dans la barre latérale.

**Configuration `~/.continue/config.json`**

```json
{
  "models": [
    {
      "title": "Claude Sonnet",
      "provider": "anthropic",
      "model": "claude-sonnet-4-6",
      "apiKey": "sk-ant-..."
    },
    {
      "title": "GPT-4o",
      "provider": "openai",
      "model": "gpt-4o",
      "apiKey": "sk-..."
    },
    {
      "title": "Qwen Local",
      "provider": "ollama",
      "model": "qwen2.5-coder:14b"
    },
    {
      "title": "Gemini Flash",
      "provider": "google",
      "model": "gemini-2.0-flash",
      "apiKey": "AIza..."
    }
  ],
  "tabAutocompleteModel": {
    "title": "Codestral Local",
    "provider": "ollama",
    "model": "codestral:latest"
  },
  "slashCommands": [
    {
      "name": "share",
      "description": "Partager la conversation"
    },
    {
      "name": "commit",
      "description": "Générer un message de commit"
    }
  ]
}
```

**Fonctionnalités clés**

- **Chat** : conversation avec le modèle de son choix, switcher à la volée
- **Complétion** : avec un modèle dédié (Codestral, Qwen-coder...)
- **@-mentions** dans le chat :
  - `@file` : inclure un fichier spécifique
  - `@folder` : inclure tout un dossier
  - `@terminal` : inclure la sortie du terminal
  - `@codebase` : indexer et interroger tout le projet
  - `@web` : recherche web (selon le modèle)
  - `@docs` : référencer de la documentation

**Avantage stratégique : cloud + local**

```
STRATÉGIE HYBRIDE AVEC CONTINUE.DEV
─────────────────────────────────────────────────────

  Code PUBLIC / OSS               Code PRIVÉ / Sensible
  ──────────────────              ─────────────────────
  Claude Sonnet (API)             Ollama local (gratuit)
  GPT-4o (API)                    Qwen2.5-coder:14b
  Gemini (API)                    LM Studio + Mistral

  Décision au cas par cas : changer de modèle en 1 clic
```

> [!tip] Analogie
> Pense à Continue.dev comme à un navigateur multi-moteur de recherche : tu choisis Google pour certaines requêtes, DuckDuckGo pour d'autres, selon tes besoins de confidentialité et de pertinence.

**Idéal pour** : usage local/entreprise, budget serré, développeurs multi-modèle, projets avec données sensibles.

---

### Cline (anciennement Claude Dev)

Cline est un **agent autonome** qui fonctionne directement dans VS Code. Contrairement à un assistant de chat, Cline peut lire tes fichiers, exécuter des commandes, créer de nouveaux fichiers, et modifier ton projet de façon autonome.

**Installation**

Extensions → chercher "Cline" → Installer → Entrer la clé API dans les settings de l'extension

**Modèles supportés**

- Claude Sonnet / Opus (Anthropic)
- GPT-4o (OpenAI)
- Gemini (Google)
- Modèles locaux via Ollama ou LM Studio

**Ce que Cline peut faire**

```
CLINE : AGENT AUTONOME
───────────────────────────────────────────────
Toi : "Ajoute un système d'authentification JWT"

Cline :
  1. Lit package.json → voit les dépendances actuelles
  2. Lit src/server.js → comprend l'architecture
  3. Installe jsonwebtoken via npm
  4. Crée src/middleware/auth.js
  5. Modifie src/routes/users.js
  6. Modifie src/server.js
  7. Exécute les tests → corrige les erreurs
  8. Rapport : "Authentification JWT ajoutée"
───────────────────────────────────────────────
```

**Mode auto-approve**

> [!warning]
> Le mode "auto-approve" permet à Cline d'exécuter des actions sans demander confirmation à chaque étape. C'est puissant mais **dangereux** si mal configuré : Cline pourrait supprimer des fichiers, exécuter des commandes destructrices, ou faire des modifications indésirables. Commence toujours en mode manuel (confirmation requise) jusqu'à ce que tu comprennes le comportement de l'agent.

**Cline vs Continue.dev**

| Aspect | Continue.dev | Cline |
|--------|-------------|-------|
| Rôle | Assistant chat | Agent autonome |
| Actions | Suggestions | Modifie les fichiers |
| Exécution | Non | Oui (commandes shell) |
| Contrôle | Élevé | Variable (selon mode) |
| Coût tokens | Modéré | Élevé (lit tout le projet) |
| Cas d'usage | Questions, refactor ponctuel | Tâches complexes multi-fichiers |

> [!warning]
> Cline peut consommer **beaucoup de tokens** car il lit souvent plusieurs fichiers pour comprendre le contexte. Surveille tes coûts API, surtout avec Claude Opus ou GPT-4o.

---

### Gemini Code Assist (Google)

L'option gratuite officielle de Google pour VS Code.

**Installation**

1. Extensions → chercher "Gemini Code Assist" → Installer
2. Se connecter avec un compte Google
3. Accepter les conditions d'utilisation

**Fonctionnalités**

- Complétion inline (suggestions pendant la frappe)
- Chat latéral dans VS Code
- Génération de code, tests, documentation
- Contexte de **1 million de tokens** — parmi les plus grands disponibles

**Avantage clé**

- **Gratuit** pour usage personnel (quota généreux)
- Contexte 1M tokens : peut lire des codebases entiers
- Qualité de complétion correcte pour du code courant

> [!info]
> Gemini Code Assist est idéal pour les étudiants ou les développeurs avec un budget limité qui veulent une complétion de qualité sans frais mensuels. La limite est la qualité parfois inférieure à Claude ou GPT-4o sur des tâches complexes.

---

### Tabnine

L'un des premiers outils de complétion IA, avant même GitHub Copilot.

**Points forts**

- Historique long : stable et éprouvé en entreprise
- **Self-hosted** disponible en version Enterprise
- Option d'entraînement sur code privé (pour les équipes)
- Complétion locale possible (modèle embarqué)

**Versions**

- **Gratuit** : complétion limitée, modèle plus petit
- **Pro** : $12/mois, complétion complète
- **Enterprise** : tarif sur devis, self-hosted, audit logs

**Cas d'usage recommandé** : entreprises avec exigences strictes de confidentialité qui ont besoin d'un self-hosted.

---

### Supermaven

Fondé par Jacob Jackson, ancien CTO de Tabnine, Supermaven est optimisé pour la **vitesse de complétion**.

**Caractéristiques**

- Contexte de **300 000 tokens** pour la complétion inline — très supérieur à Copilot
- Latence ultra-faible (optimisé pour réduire le délai perçu)
- Complétion multi-lignes pertinente
- **Gratuit** avec quota quotidien, **Pro $10/mois**

**Idéal pour** : développeurs qui trouvent Copilot trop lent ou dont les suggestions sont trop courtes par manque de contexte.

---

## 3. Cursor IDE

Cursor est un **fork de VS Code** avec l'IA intégrée nativement au cœur de l'éditeur, pas comme une extension.

```
VS CODE + EXTENSIONS          CURSOR (NATIF)
────────────────────          ──────────────────────────
Extension IA                  IA dans le moteur lui-même
  ↓ (couche supplémentaire)     ↓ (intégré)
Éditeur VS Code               Éditeur (fork VS Code)

Avantages VS Code :           Avantages Cursor :
- Extensions classiques       - Composer (multi-fichiers)
- Familier                    - Tab prédictif avancé
- Plus léger                  - Contexte @Codebase natif
- Open source                 - Meilleure UX globale IA
```

**Installation**

Télécharger depuis https://cursor.com — disponible Windows, macOS, Linux.

Importer ses settings VS Code en quelques clics (extensions, thèmes, raccourcis).

**Ce qui le différencie vraiment**

**Cursor Tab** : complétion multi-lignes qui prédit non seulement le caractère suivant mais des blocs entiers de code en analysant les modifications récentes dans le contexte.

**Composer** : l'agent phare de Cursor. Il peut modifier plusieurs fichiers en une seule instruction, créer une architecture complète, refactorer un module entier.

```
CURSOR COMPOSER EN ACTION
─────────────────────────────────────────────────────
Instruction : "Migre ce projet Express vers Fastify,
               en conservant tous les middlewares"

Composer :
  ┌── Analyse package.json
  ├── Lit src/server.js
  ├── Lit src/routes/*.js (tous les fichiers)
  ├── Lit src/middleware/*.js
  │
  ├── Génère plan de migration
  ├── Modifie package.json (deps)
  ├── Réécrit src/server.js
  ├── Adapte src/routes/*.js
  └── Adapte src/middleware/*.js

  Résultat : diff visible avant validation
─────────────────────────────────────────────────────
```

**Références dans Composer et Chat**

- `@Codebase` : indexe et interroge l'ensemble du projet
- `@Web` : recherche sur internet en temps réel
- `@Docs` : référencer de la documentation externe
- `@file` : inclure un fichier spécifique
- `@git` : contexte des commits récents

**Modèles disponibles dans Cursor**

- Claude Sonnet 4.6 / Opus (Anthropic)
- GPT-4o / o3 (OpenAI)
- Gemini 2.0 Flash / Pro (Google)
- Cursor Tab (modèle propre pour la complétion)

**Configuration des modèles**

`Cursor Settings → Models` → activer/désactiver les modèles selon le budget.

**Prix**

| Plan | Prix | Inclus |
|------|------|--------|
| Hobby (free) | $0 | 2000 complétions, 50 slow requests |
| Pro | $20/mois | Complétions illimitées, 500 fast requests |
| Business | $40/mois | Pro + gestion équipe, SSO, audit |

> [!tip] Analogie
> Cursor est comme VS Code qui aurait "grandi avec l'IA" : au lieu de greffer une extension, tout a été repensé pour que l'IA soit le copilote naturel de chaque action.

**Recommandé pour** : développeurs qui veulent l'expérience agent la plus fluide, sans configurer d'extensions séparément.

---

## 4. Windsurf (Codeium)

Windsurf est le fork de VS Code développé par l'équipe **Codeium**, concurrent direct de Cursor.

**Installation**

Télécharger depuis https://windsurf.com

**Cascade : l'agent de Windsurf**

Équivalent du Composer de Cursor, Cascade peut :
- Lire et modifier plusieurs fichiers
- Exécuter des commandes dans le terminal
- Corriger les erreurs de manière autonome
- Garder un fil de conversation persistent sur les modifications

**Points forts par rapport à Cursor**

- Souvent plus généreux en quota gratuit
- Interface parfois jugée plus fluide
- Bonne détection du contexte sans configuration manuelle
- Modèle propre Codeium pour la complétion (rapide)

> [!info]
> En 2025-2026, Cursor et Windsurf sont en concurrence directe. Le choix dépend souvent des préférences d'UX et des offres promotionnelles du moment. Les deux sont des alternatives solides à VS Code + extensions.

---

## 5. JetBrains AI

Pour les développeurs utilisant les IDEs JetBrains (IntelliJ IDEA, PyCharm, WebStorm, GoLand, Rider, DataGrip...).

```
ÉCOSYSTÈME JETBRAINS IA
────────────────────────────────────────────────────
  IntelliJ IDEA    PyCharm      WebStorm    GoLand
       │               │             │          │
       └───────────────┴─────────────┴──────────┘
                         │
              ┌──────────┴──────────┐
              │                     │
     JetBrains AI Assistant    GitHub Copilot
     (natif JetBrains)         (même extension
                                 que VS Code)
```

**JetBrains AI Assistant**

**Installation** : `Settings → Plugins → Marketplace → "AI Assistant"` → Installer → Redémarrer

**Fonctionnalités**

- Chat dans la barre latérale avec accès au projet
- Complétion inline contextuelle
- Génération de code à partir de commentaires
- Génération de tests unitaires
- Documentation automatique (Javadoc, docstrings...)
- Refactoring assisté par IA
- Explication de code sélectionné
- Commits avec message généré automatiquement

**Modèle utilisé**

JetBrains utilise un ensemble de modèles en backend (dont des partenariats avec différents fournisseurs). L'utilisateur ne choisit pas directement le modèle — c'est transparent.

**Prix**

Inclus dans le **JetBrains All Products Pack** (abonnement tout-en-un). Pour les IDE individuels, l'AI Assistant est une option supplémentaire dans l'abonnement.

**GitHub Copilot pour JetBrains**

Le même plugin Copilot existe pour tous les IDEs JetBrains. Installation identique : `Plugins → GitHub Copilot`.

> [!info]
> Si tu utilises déjà un IDE JetBrains et que tu as un abonnement actif, JetBrains AI Assistant est le point d'entrée naturel. Pour une expérience plus puissante en mode agent, considère d'ajouter GitHub Copilot ou d'utiliser Continue.dev (qui supporte aussi JetBrains).

---

## 6. Comparaison et recommandations

### Tableau comparatif complet

| Outil | IDE | Modèles | Prix (mois) | Complétion | Chat | Agent | Local ? |
|-------|-----|---------|-------------|------------|------|-------|---------|
| GitHub Copilot | VS Code / JB | Copilot (GPT) | $10 | ★★★★☆ | ★★★★☆ | ★★★☆☆ | Non |
| Continue.dev | VS Code / JB | Tous | Gratuit+API | ★★★☆☆ | ★★★★★ | ★★☆☆☆ | Oui |
| Cline | VS Code | Tous | Gratuit+API | Non | ★★★☆☆ | ★★★★★ | Oui |
| Claude Code Ext. | VS Code | Claude | API ($) | Non | ★★★★☆ | ★★☆☆☆ | Non |
| Gemini Code Assist | VS Code | Gemini | Gratuit | ★★★☆☆ | ★★★☆☆ | ★★☆☆☆ | Non |
| Tabnine | VS Code / JB | Tabnine | $12 | ★★★★☆ | ★★☆☆☆ | ★☆☆☆☆ | Oui (ent.) |
| Supermaven | VS Code | Supermaven | $10 | ★★★★★ | ★★☆☆☆ | ★☆☆☆☆ | Non |
| Cursor Pro | Cursor | Tous | $20 | ★★★★★ | ★★★★★ | ★★★★★ | Non |
| Windsurf | Windsurf | Tous | Variable | ★★★★☆ | ★★★★☆ | ★★★★☆ | Non |
| JetBrains AI | JetBrains | JB (varié) | Inclus | ★★★☆☆ | ★★★★☆ | ★★☆☆☆ | Non |

### Recommandations par profil

```
QUEL OUTIL CHOISIR ?
══════════════════════════════════════════════════════════

  ÉTUDIANT / BUDGET ZÉRO
  ─────────────────────
  Option A : Gemini Code Assist (gratuit, VS Code)
  Option B : Continue.dev + Ollama (100% gratuit et local)
  Option C : GitHub Copilot (gratuit via GitHub Education)

  FREELANCE / DÉVELOPPEUR SOLO
  ────────────────────────────
  Option A : Cursor Pro ($20) → meilleure UX agent
  Option B : Continue.dev + Claude API (paye à l'usage)
  Option C : GitHub Copilot ($10) + Continue.dev gratuit

  ÉQUIPE / ENTREPRISE
  ───────────────────
  Confidentiel : Continue.dev + Ollama local (pas de cloud)
  Standard : GitHub Copilot Business ($19/user)
  Budget : Continue.dev + API mutualisée

  VS CODE DÉJÀ INSTALLÉ
  ─────────────────────
  Commencer par : GitHub Copilot (le plus intégré)
  Ou : Continue.dev si multi-modèle requis

  JETBRAINS UTILISATEUR
  ─────────────────────
  JetBrains AI Assistant (natif) + GitHub Copilot (agent)

  MAX CONFIDENTIALITÉ
  ───────────────────
  Continue.dev + Ollama + Qwen2.5-coder local
  Voir [[09 - IA Locale avec Ollama]]

  MAX PUISSANCE AGENT
  ───────────────────
  Cursor Pro (Composer) ou Cline (VS Code)
```

---

## 7. Configuration optimale pour une session de dev

### Éviter les conflits entre extensions de complétion

> [!warning]
> N'active **jamais** deux extensions de complétion inline en même temps. Copilot + Supermaven + Tabnine actifs simultanément = conflits, latence, suggestions contradictoires. Choisis-en **un seul** pour la complétion.

```
RÈGLE : UNE SEULE EXTENSION DE COMPLÉTION
──────────────────────────────────────────
✓ Copilot (complétion) + Continue.dev (chat) = OK
✓ Supermaven (complétion) + Cline (agent) = OK
✗ Copilot + Supermaven + Tabnine = CONFLITS
```

### Ordre de priorité recommandé

```
SESSION DE DEV OPTIMALE (VS CODE)
──────────────────────────────────────────────────────
1. COMPLÉTION INLINE (un seul actif)
   → Copilot OU Supermaven OU Gemini inline

2. CHAT CONTEXTUEL (pour les questions)
   → Continue.dev (multi-modèle) OU Copilot Chat

3. AGENT (pour les tâches complexes)
   → Cline OU Claude Code CLI (terminal séparé)

Désactiver les extensions non utilisées dans
  Extensions → clic droit → "Disable (Workspace)"
──────────────────────────────────────────────────────
```

### Raccourcis clavier à configurer

```json
// keybindings.json (VS Code)
[
  {
    "key": "ctrl+shift+c",
    "command": "continue.acceptDiff"
  },
  {
    "key": "ctrl+shift+j",
    "command": "continue.focusContinueInput"
  }
]
```

### Snippets de prompts dans Continue.dev

Continue.dev permet de créer des prompts réutilisables (`/commandes` personnalisées) dans `~/.continue/config.json` :

```json
{
  "customCommands": [
    {
      "name": "review",
      "prompt": "Revue ce code en français : cherche les bugs potentiels, les problèmes de performance, et les violations des bonnes pratiques. Sois concis.",
      "description": "Revue de code rapide"
    },
    {
      "name": "test",
      "prompt": "Génère des tests unitaires complets pour ce code. Utilise le framework de test déjà présent dans le projet. Couvre les cas limites.",
      "description": "Générer des tests unitaires"
    },
    {
      "name": "doc",
      "prompt": "Génère la documentation JSDoc/docstring pour ce code. En anglais, format standard du langage.",
      "description": "Générer la documentation"
    }
  ]
}
```

Utilisation dans le chat Continue : taper `/review`, `/test`, ou `/doc` suivi du code sélectionné.

---

## 8. Confidentialité et extensions

### Quelles extensions envoient ton code dans le cloud ?

```
DONNÉES ENVOYÉES AU CLOUD
───────────────────────────────────────────────────────
GitHub Copilot    → Serveurs Microsoft/GitHub
                    (paramètre télémétrie désactivable)

Continue.dev      → Dépend du modèle configuré :
  + Claude API    → Anthropic
  + GPT-4o API    → OpenAI
  + Ollama local  → RIEN (100% local)

Cline             → Dépend du modèle API utilisé

Gemini Code Assist→ Serveurs Google

Tabnine Free/Pro  → Serveurs Tabnine
Tabnine Enterprise→ Serveurs propres (self-hosted)

Cursor            → Serveurs Cursor + fournisseur modèle
Windsurf          → Serveurs Codeium + fournisseur modèle

JetBrains AI      → Serveurs JetBrains
───────────────────────────────────────────────────────
```

### Configuration "local only" avec Continue.dev + Ollama

Pour une confidentialité maximale, aucun octet de ton code ne quitte ta machine :

```bash
# 1. Installer Ollama
# https://ollama.com → télécharger et installer

# 2. Télécharger un modèle de code
ollama pull qwen2.5-coder:14b   # 8.9 GB, excellent rapport qualité/taille
ollama pull codestral:latest    # Pour la complétion inline

# 3. Vérifier que le serveur tourne
ollama list
```

```json
// ~/.continue/config.json - VERSION 100% LOCALE
{
  "models": [
    {
      "title": "Qwen Coder 14B (Local)",
      "provider": "ollama",
      "model": "qwen2.5-coder:14b"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Codestral (Local)",
    "provider": "ollama",
    "model": "codestral:latest"
  }
}
```

> [!info]
> Avec cette configuration, Continue.dev contacte uniquement `http://localhost:11434` (Ollama). Aucune donnée ne quitte le réseau local. Voir [[09 - IA Locale avec Ollama]] pour configurer Ollama correctement.

### Options enterprise

| Solution | Modèle de déploiement | Confidentialité |
|----------|----------------------|----------------|
| GitHub Copilot Business | Cloud (Microsoft) | Pas d'entraînement sur code privé |
| Tabnine Enterprise | Self-hosted | Code reste sur les serveurs internes |
| Continue.dev + Ollama | On-premise | 100% local |
| Azure OpenAI + Continue | Cloud privé (Azure) | Données en tenant Azure |

> [!warning]
> Vérifie toujours les conditions d'utilisation de l'extension choisie avant de l'utiliser sur du code propriétaire ou sensible. Certaines extensions réservent le droit d'utiliser les données partagées pour améliorer leurs modèles (opt-out souvent disponible mais parfois non évident).

Pour un guide complet sur la confidentialité des outils IA, voir [[11 - IA Confidentialite et Entreprise]].

---

## Carte Mentale ASCII

```
                    INTÉGRATIONS IDE ET EXTENSIONS IA
                    ═══════════════════════════════════
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          │                         │                         │
     VS CODE                   IDE FORKS               JETBRAINS
     ───────                   ─────────               ─────────
     Copilot ──── $10/mois     Cursor ──── $20/mois    JB AI
     Continue ─── gratuit+API  Windsurf ── variable    Copilot
     Cline ─────── agent       (fork VS Code)
     Gemini ──── gratuit
     Tabnine ──── $12/mois
     Supermaven ─ $10/mois
          │
          │
     3 NIVEAUX
     ─────────
     Complétion inline ──► Tab pour accepter
     Chat latéral ───────► Questions contextuelles
     Agent autonome ─────► Modifie plusieurs fichiers
          │
          │
     MODÈLES
     ───────
     Cloud ──────► Claude, GPT-4o, Gemini
     Local ──────► Ollama (Qwen, Codestral...)
     Mixte ──────► Continue.dev (stratégie hybride)
          │
          │
     CONFIDENTIALITÉ
     ───────────────
     Code envoyé ──► Copilot, Cursor, Windsurf
     Code local ───► Continue + Ollama (100% local)
     Enterprise ───► Tabnine, GitHub Copilot Business
          │
          │
     PAR PROFIL
     ──────────
     Étudiant ───► Gemini (gratuit) ou GitHub Education
     Freelance ──► Cursor Pro ou Continue + API
     Entreprise ─► Copilot Business ou Continue + Ollama
     Confidential► Continue + Ollama (full local)
```

---

## Exercices pratiques

### Exercice 1 — Configurer Continue.dev en mode hybride

**Objectif** : mettre en place Continue.dev avec Claude pour le chat et Ollama pour la complétion inline.

1. Installe Continue.dev depuis le marketplace VS Code
2. Installe Ollama et télécharge `codestral:latest` pour la complétion
3. Configure `~/.continue/config.json` avec :
   - Claude Sonnet comme modèle principal pour le chat (clé API Anthropic)
   - Codestral local comme `tabAutocompleteModel`
4. Ouvre un fichier de code, écris une fonction incomplète, vérifie que la complétion Codestral s'active
5. Dans le chat Continue, utilise `@file` pour référencer ton fichier et demande une revue de code à Claude
6. Note la différence de latence entre la complétion locale et le chat cloud

**Critère de réussite** : deux modèles actifs simultanément, complétion rapide (local) + chat riche (cloud).

---

### Exercice 2 — Utiliser Cursor Composer pour un refactoring multi-fichiers

**Objectif** : expérimenter l'agent Composer de Cursor sur une tâche réelle.

1. Télécharge et installe Cursor depuis https://cursor.com
2. Importe tes settings VS Code existants
3. Prépare un petit projet avec au moins 3 fichiers (ex: un projet Express simple)
4. Ouvre Composer (`Ctrl+Shift+I` ou bouton Composer dans la barre)
5. Donne cette instruction : *"Ajoute une gestion d'erreurs centralisée dans tous les fichiers de routes. Crée un middleware dédié et adapte chaque route pour l'utiliser."*
6. Observe le plan généré avant validation
7. Valide les modifications et vérifie que le code fonctionne

**Critère de réussite** : le Composer a modifié au moins 2 fichiers existants et en a créé un nouveau, sans intervention manuelle.

---

### Exercice 3 — Audit de confidentialité de ta stack IA

**Objectif** : cartographier précisément quelles données quittent ta machine avec tes outils actuels.

1. Liste toutes les extensions IA actuellement installées dans ton VS Code (ou autre IDE)
2. Pour chaque extension, identifie :
   - Est-ce que le code est envoyé dans le cloud ? Vers quel fournisseur ?
   - Y a-t-il un paramètre de télémétrie/opt-out à configurer ?
   - Les conditions d'utilisation autorisent-elles l'entraînement sur tes données ?
3. Classe tes projets selon leur niveau de sensibilité (OSS / personnel / professionnel / confidentiel)
4. Définis une politique : quelle extension utiliser pour quel type de projet ?
5. Configure VS Code pour activer/désactiver les extensions par workspace selon cette politique

**Critère de réussite** : tu as une politique documentée (même en deux lignes) et tu sais comment switcher d'une config à l'autre selon le projet.

---

## Liens connexes

- [[01 - Panorama des IA pour Développeurs]]
- [[03 - Prompt Engineering pour le Code]]
- [[08 - OpenCode et CLI Alternatifs]]
- [[09 - IA Locale avec Ollama]]
- [[11 - IA Confidentialite et Entreprise]]
