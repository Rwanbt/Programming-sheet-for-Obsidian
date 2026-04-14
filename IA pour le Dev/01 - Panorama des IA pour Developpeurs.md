# 01 - Panorama des IA pour Développeurs

> [!info] Mis à jour avril 2026

## Qu'est-ce qu'une IA pour développeurs ?

Une **IA pour développeurs** est un système logiciel capable de comprendre, générer, analyser et transformer du code source. Ces outils vont du simple autocompleteur de lignes à l'agent autonome capable de naviguer dans un projet entier, exécuter des tests, corriger des bugs et soumettre des Pull Requests — sans intervention humaine.

En 2026, maîtriser ces outils n'est plus une option : c'est une compétence fondamentale, au même titre que connaître Git ou savoir utiliser un débogueur. Les développeurs qui les intègrent intelligemment dans leur workflow multiplient leur productivité par un facteur 2 à 10 selon les tâches.

> [!tip] Analogie
> Imagine l'évolution des outils de construction :
> - **Années 1990** : tu tapes chaque caractère, l'IDE colorie la syntaxe
> - **Années 2000** : IntelliSense te propose le nom de la méthode suivante
> - **2021** : Copilot écrit le bloc entier si tu décris l'intention
> - **2023** : ChatGPT et Claude répondent à tes questions d'architecture
> - **2025-2026** : des agents autonomes lisent ton codebase, corrigent des bugs, écrivent des tests et ouvrent des PRs
>
> Chaque étape n'a pas remplacé le développeur — elle l'a rendu plus puissant. L'IA actuelle est la plus grande amplification de productivité depuis l'invention des langages de haut niveau.

---

## 1. La révolution IA dans le développement : timeline 2020 → 2026

```
2020         2021          2022          2023          2024          2025-2026
  |            |             |             |             |               |
  v            v             v             v             v               v
GPT-3        GitHub       ChatGPT       GPT-4         Claude 3.5     Agents
(OpenAI)     Copilot      (nov. 2022)   Claude 2      Sonnet         autonomes
Premiers     Completion   Explosion     Gemini 1.5    Cursor IDE     Claude Code
LLMs         en prod.     du grand      Pro (1M ctx)  Windsurf       OpenCode
capables     pour le      public        o1 reasoning  DeepSeek V4    Gemini 3.1
de coder     code                       Code agents   Qwen 3.6-Plus  o3 / R1
                                        Mistral       Codestral      SWE-bench
                                        Llama 2       Llama 4        80%+ scores

 Modèles       IDE          Chat          Multi-        IDE Agents    Agents
 de base    completion      IA pour      modal +       autonomes    CLI + Cloud
                           le grand     reasoning     matures      industriels
                           public
```

### Pourquoi tout développeur doit maîtriser l'IA aujourd'hui

| Réalité du marché | Impact concret |
|---|---|
| **Vitesse de développement** | Les équipes IA-augmentées livrent 2 à 5× plus vite |
| **Barrière d'entrée** | Des non-développeurs créent des MVPs entiers avec des agents |
| **Qualité du code** | Les IA génèrent moins de bugs que le code humain moyen pour les tâches standards |
| **Documentation** | Génération automatique de docs, tests, commentaires |
| **Apprentissage** | Un junior avec Claude Code apprend plus vite qu'avec Stack Overflow seul |
| **Compétitivité** | Les offres d'emploi 2025 mentionnent "AI-augmented dev" comme compétence clé |

> [!warning] Le paradoxe de la dépendance
> L'IA peut t'accélérer ou t'atrophier selon l'usage. Utiliser l'IA pour **comprendre** (demander des explications, explorer des alternatives) te rend meilleur. L'utiliser pour **contourner** (copier sans lire) te rend fragile. Ce panorama t'aide à choisir les bons outils **et** la bonne posture.

---

## 2. Les catégories d'outils IA pour développeurs

### Vue d'ensemble — grand diagramme

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ÉCOSYSTÈME IA POUR DÉVELOPPEURS (2026)                   │
│                                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │  COMPLÉTION      │  │  CHAT ASSISTANTS│  │  AGENTS CLI     │             │
│  │  INLINE          │  │                 │  │                 │             │
│  │                  │  │  Conversation   │  │  Exécution      │             │
│  │  Suggestions en  │  │  interactive,   │  │  autonome dans  │             │
│  │  temps réel dans │  │  questions /    │  │  le terminal,   │             │
│  │  l'éditeur       │  │  réponses,      │  │  accès aux      │             │
│  │                  │  │  analyse de     │  │  fichiers et    │             │
│  │  - GitHub Copilot│  │  code, design   │  │  à git          │             │
│  │  - Codeium       │  │                 │  │                 │             │
│  │  - Supermaven    │  │  - ChatGPT      │  │  - Claude Code  │             │
│  │  - Tabnine       │  │  - Claude       │  │  - OpenCode     │             │
│  │                  │  │  - Gemini       │  │  - Aider        │             │
│  │  Granularité :   │  │  - Mistral Chat │  │                 │             │
│  │  Ligne / Bloc    │  │                 │  │  Granularité :  │             │
│  │                  │  │  Granularité :  │  │  Feature /      │             │
│  └─────────────────┘  │  Q&R / Session  │  │  Projet entier  │             │
│                        └─────────────────┘  └─────────────────┘             │
│                                                                             │
│  ┌─────────────────┐  ┌─────────────────────────────────────────┐           │
│  │  IA LOCALE      │  │  IDE AGENTS (éditeurs IA-natifs)        │           │
│  │  (on-premise)   │  │                                         │           │
│  │                  │  │  IDE complet avec IA intégrée en        │           │
│  │  Modèles tournant│  │  profondeur : contexte du projet,       │           │
│  │  sur ta machine  │  │  navigation multi-fichiers, refactoring │           │
│  │  sans cloud,     │  │  autonome, terminal intégré             │           │
│  │  100% privé      │  │                                         │           │
│  │                  │  │  - Cursor IDE   - Windsurf              │           │
│  │  - Ollama        │  │  - Cline        - Continue.dev          │           │
│  │  - LM Studio     │  │                                         │           │
│  │                  │  │  Granularité : Sprint / Projet          │           │
│  └─────────────────┘  └─────────────────────────────────────────┘           │
│                                                                             │
│         Autonomie croissante ──────────────────────────────────>            │
│         Complétion   →   Chat   →   CLI Agent   →   IDE Agent              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Description détaillée de chaque catégorie

#### 2.1 Complétion inline

Ces outils s'intègrent directement dans ton éditeur et suggèrent du code à mesure que tu tapes.

| Outil | Modèle sous-jacent | Prix | Spécificité |
|---|---|---|---|
| **GitHub Copilot** | GPT-5.4 + modèles Microsoft | 10$/mois | Le plus répandu, excellent contexte projet |
| **Codeium** | Modèle propriétaire | Gratuit (plan de base) | Rapide, bon support multi-langages |
| **Supermaven** | Modèle propriétaire (Sourcegraph) | Gratuit / 10$+ | Contexte 1M tokens, très rapide |
| **Tabnine** | Modèles propriétaires | Gratuit / 12$/mois | Option on-premise, RGPD-friendly |

> [!info] Comment fonctionne la complétion inline
> L'outil envoie le contexte autour de ton curseur (fichier courant, fichiers ouverts, dépôts récents) au modèle IA. Le modèle prédit la suite probable. La complétion apparaît en gris — `Tab` pour accepter, `Esc` pour ignorer. Certains outils comme Supermaven peuvent voir jusqu'à 1 million de tokens de contexte, ce qui inclut tout ton projet.

#### 2.2 Chat assistants

Interfaces conversationnelles pour poser des questions, obtenir des explications, générer du code à la demande.

| Outil | Modèle | Contexte | Forces |
|---|---|---|---|
| **ChatGPT** | GPT-5.4, o3/o4 | 272K tokens | Très polyvalent, plugins, image |
| **Claude** | Claude Sonnet/Opus 4.6 | 1M tokens | Raisonnement, code long, suivi d'instructions |
| **Gemini** | Gemini 3.1 Pro | 1M tokens | Contexte géant, gratuit en partie |
| **Le Chat (Mistral)** | Mistral Small 4, Codestral | 128K tokens | Souverain EU, Codestral pour le code |

#### 2.3 Agents CLI

Outils en ligne de commande qui ont accès à ton filesystem, peuvent lire/écrire des fichiers, exécuter des commandes, et gérer Git.

```bash
# Exemple : Claude Code en action
$ cd mon-projet-backend
$ claude
> Analyse l'architecture du projet et explique-moi les flux de données
> Trouve et corrige le bug dans le module d'authentification
> Génère des tests unitaires pour tous les endpoints non couverts
> Crée une PR avec un message de commit descriptif
```

| Outil | Modèle | Particularité |
|---|---|---|
| **Claude Code** | Claude Sonnet/Opus 4.x | Officiel Anthropic, très capable, vision fichiers |
| **OpenCode** | Multi-modèles (configurable) | Open source, supporte Anthropic/OpenAI/Gemini |
| **Aider** | Multi-modèles | Pionnier du genre, excellent pour le refactoring |

#### 2.4 IDE agents — éditeurs IA-natifs

Ces éditeurs remplacent VS Code et intègrent l'IA à tous les niveaux : complétion, chat, exécution autonome d'agents, navigation sémantique du codebase.

| Outil | Base | Modèle | Particularité |
|---|---|---|---|
| **Cursor IDE** | VS Code fork | GPT-5.4, Claude, Gemini | Le plus populaire, agents "Composer" |
| **Windsurf** | VS Code fork | Modèles Codeium | "Cascade" agent très autonome |
| **Cline** | Extension VS Code | Multi-modèles | Open source, contrôle fin des permissions |
| **Continue.dev** | Extension VS Code/JetBrains | Multi-modèles | Open source, ultra-configurable |

#### 2.5 IA locale avec Ollama et LM Studio

Faire tourner des modèles sur ta propre machine — aucune donnée ne quitte ton réseau.

```bash
# Exemple Ollama
$ ollama pull qwen3-coder:32b
$ ollama run qwen3-coder:32b
>>> Écris une fonction de validation d'email en Python
```

| Outil | Interface | Usage idéal |
|---|---|---|
| **Ollama** | CLI + API REST | Intégration dans scripts, agents locaux |
| **LM Studio** | GUI desktop | Exploration, comparaison de modèles |

---

## 3. Les grandes familles de modèles en 2025-2026

### 3.1 Anthropic — Claude

Anthropic se distingue par sa culture de la sécurité IA ("Constitutional AI") et ses modèles excellents en raisonnement et suivi d'instructions.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE CLAUDE (Anthropic)                                │
│                                                            │
│  claude-haiku-4-5    →  Rapide, économique, tâches simples │
│  claude-sonnet-4-6   →  Équilibre optimal qualité/coût     │
│  claude-opus-4-6     →  Intelligence maximale, tâches dures│
│                                                            │
│  Contexte : 200K (Haiku) / 1M tokens (Sonnet, Opus)       │
│  Vision : Oui             Code : Excellent (80.8% SWE)     │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Vitesse | Qualité | Prix indicatif (input/output) | Idéal pour |
|---|---|---|---|---|
| **claude-haiku-4-5** | Très rapide | Bonne | $1 / $5 par M tokens | Tâches répétitives, complétion, chat rapide |
| **claude-sonnet-4-6** | Rapide | Excellente | $3 / $15 par M tokens | Code complexe, agents, usage quotidien |
| **claude-opus-4-6** | Modéré | Maximale | $5 / $25 par M tokens | Architecture, raisonnement profond, tâches critiques |

**Forces Claude :** suivi d'instructions remarquable, honnêteté (dit quand il ne sait pas), fenêtre de contexte jusqu'à 1M tokens (Sonnet/Opus), excellent pour le code long et le refactoring. Claude Opus 4.6 atteint **80.8% sur SWE-bench**.

**Faiblesses :** plus coûteux que certains concurrents, pas encore de recherche web native dans tous les contextes.

### 3.2 OpenAI — GPT et série O

OpenAI reste la référence grand public avec ChatGPT, et pousse l'innovation des modèles de raisonnement.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE OpenAI                                            │
│                                                            │
│  GPT-5.4         →  Polyvalent, multimodal, rapide         │
│  GPT-5.4 Nano    →  Ultra-économique, tâches courantes     │
│  GPT-5.4 Mini    →  Milieu de gamme                        │
│  o3              →  Meilleur raisonnement disponible       │
│  o4              →  Raisonnement dernière génération       │
│                                                            │
│  Contexte : 272K (std) / 1M via Codex   Vision : Oui       │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Type | Prix indicatif | Forces | Faiblesses |
|---|---|---|---|---|
| **GPT-5.4** | Polyvalent | $2.50 / $15 par M tokens | Multimodal, rapide, excellent chat | 57.7% SWE-bench (métrique Pro) |
| **GPT-5.4 Nano** | Économique | $0.20 / $1.25 par M tokens | Prix minimal, 400K contexte | Qualité réduite sur tâches complexes |
| **GPT-5.4 Mini** | Milieu de gamme | Variable | Bon compromis | Moins puissant que GPT-5.4 |
| **o3** | Raisonnement max | Variable | Meilleur du monde en benchmark | Très lent, coût élevé |
| **o4** | Raisonnement avancé | Variable | Dernière génération de raisonnement | Lent, coût élevé |

> [!info] Les modèles "o" : qu'est-ce que le raisonnement ?
> Les modèles o3/o4 utilisent une technique appelée **chain-of-thought interne** : avant de répondre, le modèle réfléchit en silence pendant plusieurs secondes à minutes. Ce "temps de réflexion" leur permet de résoudre des problèmes mathématiques, logiques et algorithmiques complexes bien mieux que les modèles standard. Le prix à payer : la lenteur et le coût.

### 3.3 Google — Gemini

Google a fait une percée majeure avec Gemini 3.1 Pro (sorti fév. 2026) et son contexte de 1 million de tokens, avec d'excellentes performances sur le code.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE GEMINI (Google DeepMind)                          │
│                                                            │
│  Gemini 3.1 Flash-Lite  →  Le moins cher de la gamme      │
│  Gemini 3.1 Flash       →  Rapide, milieu de gamme         │
│  Gemini 3.1 Pro         →  1M tokens contexte, top code   │
│  Gemini CLI             →  Outil CLI gratuit               │
│                                                            │
│  Contexte : jusqu'à 1M tokens   Vision : Oui              │
│  Gratuit : Oui (limites)        Code : Très bon (78.8% SWE)│
└────────────────────────────────────────────────────────────┘
```

| Modèle | Contexte | Prix indicatif | Forces | Faiblesses |
|---|---|---|---|---|
| **Gemini 3.1 Flash-Lite** | Variable | $0.25 / $1.50 par M tokens | Le moins cher, tâches simples | Qualité limitée |
| **Gemini 3.1 Flash** | Variable | Variable | Vitesse, bon rapport qualité/prix | Moins puissant que 3.1 Pro |
| **Gemini 3.1 Pro** | 1M tokens | $2 / $12 par M tokens | Contexte 1M, code, multimodal, 78.8% SWE | Parfois verbeux |
| **Gemini CLI** | 3.1 Pro gratuit | Gratuit (quota) | Accès terminal direct, gratuit | Limitations de quota |

**Le contexte de 1M tokens de Gemini 3.1 Pro** est remarquable : tu peux envoyer un projet entier en une seule requête pour l'analyser ou le refactoriser.

### 3.4 Mistral AI — Le champion européen

Mistral AI, startup française, offre des modèles souverains (hébergés en Europe) et le meilleur modèle de code spécialisé open-weights : Codestral.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE MISTRAL AI (Paris, France)                        │
│                                                            │
│  Mistral Small 4     →  Raisonnement + code + multimodal   │
│  Codestral 2508      →  Spécialisé code 22B, 80K+ contexte │
│  Devstral            →  Modèle agentique (nouveau)         │
│                                                            │
│  RGPD : Oui (hébergement EU)   Open-weights : Oui         │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Paramètres | Prix indicatif | Forces | Faiblesses |
|---|---|---|---|---|
| **Mistral Small 4** | Variable | ~$0.50 / $1.50 par M tokens | Raisonnement + code + multimodal, open-weight | Moins puissant que les géants cloud |
| **Codestral 2508** | ~22B | ~$0.1 / $0.3 par M tokens | Spécialisé code, 80K+ contexte, FIM, rapide | Moins polyvalent |
| **Devstral** | Variable | Variable | Modèle agentique, tâches autonomes | Récent, moins éprouvé |

**Pourquoi Codestral 2508 est intéressant :** modèle de code spécialisé avec support FIM (Fill-In-the-Middle), 80K+ tokens de contexte, prix minimal, utilisable dans Cursor/Continue.dev. Idéal pour les équipes avec contraintes budgétaires ou RGPD.

### 3.5 Meta — Llama (open source)

Meta a fait le choix stratégique de l'open source : les modèles Llama sont téléchargeables librement et peuvent tourner localement.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE LLAMA (Meta AI) — OPEN SOURCE                     │
│                                                            │
│  Llama 4 Scout    →  109B total, 17B actifs, 10M contexte │
│  Llama 4 Maverick →  17B actifs, 128 experts              │
│                                                            │
│  Licence : Meta Llama License (usage commercial autorisé  │
│  sous conditions pour les grands acteurs)                  │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Taille | RAM GPU requise | Forces | Faiblesses |
|---|---|---|---|---|
| **Llama 4 Scout** | 109B (17B actifs, MoE) | ~24 GB VRAM (MoE) | **10M tokens** contexte (!), open-weight, 16 experts | Infrastructure MoE requise |
| **Llama 4 Maverick** | 17B actifs, 128 experts | ~40 GB VRAM | Polyvalent, grand nombre d'experts, gratuit | Matériel conséquent requis |

### 3.6 Alibaba — Qwen (le champion du code)

La famille Qwen d'Alibaba est devenue l'une des références pour le code en open-weights, rivalisant avec des modèles propriétaires bien plus chers.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE QWEN (Alibaba Cloud) — OPEN-WEIGHTS               │
│                                                            │
│  Qwen 3.6-Plus        →  1M contexte, agentic coding      │
│  Qwen3.5 (397B)       →  Sorti fév. 2026, 201 langues     │
│  Qwen3-Coder-Next     →  80B total, 3B actifs (MoE)       │
│                                                            │
│  Disponible sur : Ollama, LM Studio, Hugging Face          │
└────────────────────────────────────────────────────────────┘
```

> [!example] Qwen3-Coder-Next en local
> ```bash
> $ ollama pull qwen3-coder:32b
> $ ollama run qwen3-coder:32b
> >>> Implémente un algorithme de Dijkstra en Rust avec gestion d'erreurs
> ```
> Le Qwen3-Coder-Next (80B total, 3B actifs en MoE) offre des performances remarquables avec une consommation mémoire bien inférieure à son nombre total de paramètres. Idéal pour du code Python, Rust, TypeScript, Java sur GPU grand public.

### 3.7 DeepSeek — La surprise chinoise

DeepSeek continue de surprendre avec des modèles open-source rivaux des meilleurs modèles propriétaires, à des prix dérisoires.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE DEEPSEEK (DeepSeek AI, Chine) — OPEN-WEIGHTS      │
│                                                            │
│  DeepSeek V4   →  1T params, 37B actifs, 81% SWE-bench   │
│  DeepSeek R2   →  Raisonnement, rival o3                   │
│                                                            │
│  Prix API : fraction du prix OpenAI                        │
│  Risque : données hébergées en Chine                       │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Forces | Faiblesses | Prix indicatif |
|---|---|---|---|
| **DeepSeek V4** | **81% SWE-bench**, 1T params (37B actifs), 1M contexte, prix minimal | Données hébergées en Chine, infrastructure parfois instable | $0.30 / $0.50 par M tokens |
| **DeepSeek R2** | Raisonnement excellent, open-weights, rival o3 | Lent, verbose | Variable |

> [!warning] DeepSeek et la confidentialité des données
> Les APIs DeepSeek hébergent tes données sur des serveurs en Chine. Pour du code propriétaire, des projets sensibles ou des entreprises soumises au RGPD, l'utilisation des API cloud DeepSeek est déconseillée. En revanche, les modèles open-weights peuvent être exécutés localement avec Ollama — sans aucun problème de confidentialité.

### 3.8 Microsoft — Phi (petits modèles puissants)

Microsoft mise sur les "Small Language Models" (SLM) ultra-efficaces pour des déploiements embarqués ou locaux.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE PHI (Microsoft Research) — OPEN-WEIGHTS           │
│                                                            │
│  Phi-4 (3.8B)   →  Remarquablement capable pour 3.8B       │
│                    Tourne sur CPU / GPU intégré             │
│                    Idéal : laptop sans GPU dédié           │
│                                                            │
│  Licence : MIT (usage commercial libre)                    │
└────────────────────────────────────────────────────────────┘
```

**Phi-4** est une démonstration que la taille n'est pas tout : entraîné sur des données soigneusement sélectionnées ("Textbooks are all you need"), il surpasse des modèles 3× plus grands sur les benchmarks de code et de raisonnement. Parfait pour tourner en local sur n'importe quelle machine.

### 3.9 Google — Gemma (open source)

Google propose aussi une famille de modèles open source avec la licence Apache 2.0.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE GEMMA (Google) — OPEN SOURCE, Apache 2.0          │
│                                                            │
│  Gemma 4 E2B, E4B  →  MoE ultra-légers                    │
│  Gemma 4 26B-A4B   →  26B total, 4B actifs (MoE)          │
│  Gemma 4 31B       →  Dense, 84.3% GPQA, 256K contexte    │
│                                                            │
│  Licence : Apache 2.0 (usage commercial libre)             │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Taille | Forces | Usage |
|---|---|---|---|
| **Gemma 4 E2B / E4B** | 2B / 4B actifs (MoE) | Ultra-léger, embarqué, faible VRAM | Appareils contraints, edge computing |
| **Gemma 4 26B-A4B** | 26B total, 4B actifs | Bon rapport qualité/taille, MoE efficace | GPU modeste, local |
| **Gemma 4 31B** | 31B (dense) | **Meilleur open-source 2026**, 84.3% GPQA, 256K contexte | Tâches exigeantes en local |

**Gemma 4 31B** est actuellement le meilleur modèle open-source de 2026, notamment pour les tâches de raisonnement et de code. Sa licence Apache 2.0 le rend utilisable sans restriction commerciale.

### 3.10 Zhipu AI — GLM-4

GLM-4 de Zhipu AI (Université Tsinghua) est un modèle remarquable pour le code et le texte en chinois et en anglais.

| Modèle | Contexte | Forces | Faiblesses |
|---|---|---|---|
| **GLM-4** | 128K tokens | Excellent en code, bilingue CH/EN, API économique | Moins connu, documentation partielle en français |

---

## 4. Les benchmarks : comment comparer les modèles objectivement

Les benchmarks permettent de comparer les modèles sur des tâches standardisées. Attention : un modèle peut dominer un benchmark et être médiocre dans ton cas d'usage réel.

### 4.1 Les benchmarks principaux pour le code

```
┌─────────────────────────────────────────────────────────────┐
│  BENCHMARKS CODE                                            │
│                                                             │
│  HumanEval  ──── Génération de code Python (Pass@1)        │
│  (OpenAI)        164 problèmes, test si le code passe       │
│                  les tests unitaires du 1er coup            │
│                                                             │
│  SWE-bench  ──── Résolution de vrais bugs GitHub           │
│  (Princeton)     2294 issues réelles de projets Python      │
│                  Score : % d'issues résolues automatiquement│
│                                                             │
│  MBPP       ──── Mostly Basic Python Problems              │
│  (Google)        374 problèmes basiques à avancés          │
│                  Plus accessible que HumanEval              │
│                                                             │
│  MMLU       ──── Connaissances générales (57 domaines)     │
│  (UC Berkeley)   Inclut CS, maths, sciences                 │
│                  Indicateur de "culture générale" du modèle │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Tableau comparatif des scores (approximatifs, avril 2026)

| Modèle | HumanEval | SWE-bench Verified | MBPP | Notes |
|---|---|---|---|---|
| **DeepSeek V4** | ~92% | **~81%** | ~90% | Meilleur SWE-bench |
| **Claude Opus 4.6** | ~92% | **~80.8%** | ~90% | Meilleur code Anthropic |
| **Gemini 3.1 Pro** | ~91% | **~78.8%** | ~89% | Fort sur contexte |
| **Claude Sonnet 4.6** | ~90% | **~70%** | ~88% | Excellent quotidien |
| **GPT-5.4** | ~90% | ~57.7% (Pro) | ~87% | Métrique SWE-bench Pro différente |
| **o3** | ~98% | ~70%+ | ~96% | Raisonnement max |
| **Llama 4 Scout** | ~85% | ~45% | ~84% | 10M contexte, open-weight |
| **Qwen3-Coder-Next** | ~88% | ~50% | ~85% | Performances remarquables |
| **Codestral 2508** | ~81% | ~35% | ~79% | Spécialisé code |
| **Phi-4 (3.8B)** | ~77% | ~20% | ~74% | Léger, local, CPU |

> [!warning] Limites des benchmarks
> Les scores sont approximatifs et évoluent rapidement avec chaque nouvelle version. De plus :
> - Les modèles sont souvent entraînés sur les benchmarks (surapprentissage)
> - SWE-bench est le plus représentatif du "vrai" travail de dev
> - Ton cas d'usage spécifique (langage, domaine, complexité) peut donner des résultats très différents
> - Toujours tester sur tes propres tâches avant de choisir

---

## 5. Grand tableau comparatif des modèles

| Modèle | Éditeur | Type | Contexte | Force principale | Prix (M tokens input) | Local ? |
|---|---|---|---|---|---|---|
| Claude Haiku 4.5 | Anthropic | Chat/API | 200K | Vitesse, économique | $1 | Non |
| Claude Sonnet 4.6 | Anthropic | Chat/API/Agent | 1M | Équilibre qualité/coût | $3 | Non |
| Claude Opus 4.6 | Anthropic | Chat/API/Agent | 1M | Intelligence maximale, 80.8% SWE | $5 | Non |
| GPT-5.4 | OpenAI | Chat/API | 272K (1M Codex) | Polyvalent, multimodal | $2.50 | Non |
| GPT-5.4 Nano | OpenAI | Chat/API | 400K | Prix minimal | $0.20 | Non |
| o3 | OpenAI | Raisonnement | Variable | Problèmes complexes | Variable | Non |
| o4 | OpenAI | Raisonnement | Variable | Raisonnement dernière génération | Variable | Non |
| Gemini 3.1 Flash-Lite | Google | Chat/API | Variable | Le moins cher Google | $0.25 | Non |
| Gemini 3.1 Pro | Google | Chat/API | 1M | Contexte 1M, code, 78.8% SWE | $2 | Non |
| Mistral Small 4 | Mistral AI | Chat/API | 128K | Raisonnement+code+multimodal, EU | ~$0.50 | Non |
| Codestral 2508 | Mistral AI | Code/API | 80K+ | Code spécialisé, FIM | ~$0.1 | Non |
| DeepSeek V4 | DeepSeek | Chat/API | 1M | 81% SWE-bench, prix dérisoire | $0.30 | Oui (weights) |
| DeepSeek R2 | DeepSeek | Raisonnement | Variable | Open reasoning, rival o3 | Variable | Oui (weights) |
| Qwen 3.6-Plus | Alibaba | Code/Agent | 1M | Agentic coding, open-weights | Gratuit (local) | Oui |
| Qwen3-Coder-Next | Alibaba | Code | Variable | 80B/3B actifs MoE, perf. remarquables | Gratuit (local) | Oui |
| Llama 4 Scout | Meta | Polyvalent | 10M | Open source, 10M contexte | Gratuit (local) | Oui |
| Llama 4 Maverick | Meta | Polyvalent | Variable | Open source, 128 experts | Gratuit (local) | Oui |
| Gemma 4 31B | Google | Polyvalent | 256K | Meilleur open-source 2026, Apache 2 | Gratuit (local) | Oui |
| Phi-4 (3.8B) | Microsoft | Compact | 16K | Efficacité sur CPU | Gratuit (local) | Oui |
| GLM-4 | Zhipu | Polyvalent | 128K | Bilingue CH/EN | ~$0.7 | Non |

---

## 6. Choisir son IA selon la tâche

```
┌─────────────────────────────────────────────────────────────────┐
│  ARBRE DE DÉCISION : QUELLE IA CHOISIR ?                       │
│                                                                 │
│  Ta tâche est...                                                │
│       │                                                         │
│       ├── Rapide / simple ? ──── Claude Haiku / GPT-5.4 Nano  │
│       │                          Gemini 3.1 Flash-Lite          │
│       │                                                         │
│       ├── Code confidentiel ? ── IA Locale : Ollama + Qwen /   │
│       │   (propriétaire,         LM Studio + Llama 4           │
│       │    RGPD sensible)        Tabnine on-premise             │
│       │                                                         │
│       ├── Gros projet /  ─────── Gemini 3.1 Pro (1M ctx)       │
│       │   codebase entier         Claude Sonnet 4.6 (1M)       │
│       │                           Llama 4 Scout (10M ctx!)     │
│       │                                                         │
│       ├── Raisonnement ─────────  o3 / o4                      │
│       │   mathématique/algo       DeepSeek R2                  │
│       │                           Claude Opus 4.6              │
│       │                                                         │
│       ├── Budget serré ─────────  Gemini 3.1 Flash-Lite        │
│       │                           GPT-5.4 Nano                 │
│       │                           Codestral 2508               │
│       │                                                         │
│       ├── Génération en masse ──  GPT-5.4 Nano, Haiku 4.5      │
│       │   (milliers de requêtes)  Gemini 3.1 Flash-Lite        │
│       │                                                         │
│       └── Architecture système ─  Claude Opus 4.6              │
│           / design complexe       o3/o4 (si budget ok)         │
└─────────────────────────────────────────────────────────────────┘
```

### Tableau tâche → IA recommandée

| Tâche | IA recommandée | Raison |
|---|---|---|
| **Question rapide / explication** | Claude Haiku 4.5, GPT-5.4 Nano, Gemini 3.1 Flash | Réponse quasi-instantanée, coût minimal |
| **Débogage complexe** | Claude Sonnet 4.6, GPT-5.4 | Excellent suivi du raisonnement sur du code existant |
| **Architecture système** | Claude Opus 4.6, o3/o4 | Raisonnement profond sur des contraintes multiples |
| **Code propriétaire / confidentiel** | Ollama + Qwen3-Coder ou Llama 4 Scout | 100% local, aucune donnée envoyée |
| **Budget serré** | Gemini 3.1 Flash-Lite (gratuit), Codestral 2508 | Qualité correcte à coût nul ou minimal |
| **Génération en masse (API)** | GPT-5.4 Nano, Claude Haiku 4.5, Gemini 3.1 Flash | Faible coût par token, rate limits élevés |
| **Raisonnement mathématique** | o3/o4, DeepSeek R2, Claude Opus 4.6 | Chain-of-thought profond |
| **Refactoring de gros codebase** | Gemini 3.1 Pro (1M tokens), Claude Sonnet 4.6 (1M) | Voit tout le projet en une seule requête |
| **Code spécialisé (complétion IDE)** | GitHub Copilot, Supermaven, Codeium | Optimisés pour l'inline completion |
| **Projets RGPD / entreprise EU** | Mistral Small 4, Codestral 2508 | Hébergement en Europe, conformité EU |

---

## 7. Cloud vs Local : le grand comparatif

### Vue d'ensemble

```
┌─────────────────────────────────────┬──────────────────────────────────────┐
│  IA CLOUD (APIs distantes)          │  IA LOCALE (Ollama, LM Studio)       │
├─────────────────────────────────────┼──────────────────────────────────────┤
│  AVANTAGES                          │  AVANTAGES                           │
│  ✓ Modèles les plus puissants       │  ✓ Confidentialité totale            │
│  ✓ Aucun matériel requis            │  ✓ Gratuit après achat hardware      │
│  ✓ Mises à jour automatiques        │  ✓ Pas de dépendance internet        │
│  ✓ Scalabilité immédiate            │  ✓ Pas de coût variable selon usage  │
│  ✓ Vision, audio, multimodal        │  ✓ Fonctionne hors ligne             │
│                                     │  ✓ Latence locale (si bon GPU)       │
│  INCONVÉNIENTS                      │                                      │
│  ✗ Coût variable (peut exploser)    │  INCONVÉNIENTS                       │
│  ✗ Données envoyées à l'extérieur   │  ✗ Modèles moins puissants           │
│  ✗ Dépendance internet              │  ✗ Investissement matériel           │
│  ✗ Rate limits (quotas)             │  ✗ Maintenance manuelle              │
│  ✗ Latence réseau                   │  ✗ Pas toujours multimodal           │
│  ✗ Prix peut changer                │  ✗ Lent sans GPU dédié               │
└─────────────────────────────────────┴──────────────────────────────────────┘
```

### Cas d'usage typiques

| Contexte | Recommandation | Raison |
|---|---|---|
| **Startup, projet perso** | Cloud (Claude Sonnet 4.6, GPT-5.4) | Pas d'infra à gérer, accès aux meilleurs modèles |
| **Entreprise avec données sensibles** | Local ou cloud EU (Mistral Small 4) | Conformité RGPD, secret industriel |
| **Développeur isolé sans internet** | Local (Ollama + Llama 4 / Qwen3) | Fonctionne hors connexion |
| **Étudiants / budget zéro** | Gemini 3.1 Flash-Lite gratuit + Ollama local | Sans débourser un centime |
| **Grande équipe, volume élevé** | Cloud API (coût prévisible) | Scalabilité, monitoring centralisé |
| **Code très propriétaire (banque, défense)** | Local on-premise uniquement | Aucune fuite possible |

> [!tip] Analogie
> Choisir entre cloud et local, c'est comme choisir entre **louer une salle de sport** (cloud) ou **acheter les équipements** (local) :
> - La salle de sport (cloud) est toujours à jour, accessible partout, mais tu paies à chaque visite et tu n'es pas chez toi
> - Les équipements chez toi (local) : investissement initial, mais disponibles 24h/24, sans abonnement, et personne ne regarde ce que tu fais

### Configuration matérielle recommandée pour le local

```
GPU VRAM  →  Modèles utilisables
  8 GB    →  Phi-4 (3.8B), Gemma 4 E4B, Llama 4 Scout (MoE, 17B actifs)
 16 GB    →  Gemma 4 26B-A4B (4B actifs), Qwen3-Coder 7B quantisé
 24 GB    →  Llama 4 Maverick (17B actifs MoE), Gemma 4 31B quantisé
 48 GB    →  Gemma 4 31B (pleine précision), Qwen3.5 72B (quantisé)
 80 GB+   →  Modèles denses 70B+, DeepSeek V4 partiel
```

---

## 8. L'évolution rapide de l'écosystème

L'IA pour le code évolue à une vitesse sans précédent. Un modèle "state of the art" en janvier peut être dépassé en mars. Voici comment rester à jour.

### 8.1 Les ressources incontournables

```
┌─────────────────────────────────────────────────────────────┐
│  RESTER À JOUR : RESSOURCES                                 │
│                                                             │
│  BENCHMARKS EN TEMPS RÉEL                                   │
│  - Chatbot Arena (LMSYS) : classement ELO par votes humains │
│  - SWE-bench Leaderboard : meilleurs agents pour le code    │
│  - Hugging Face Open LLM Leaderboard : modèles open source  │
│                                                             │
│  NEWSLETTERS                                                │
│  - TLDR AI (quotidien, gratuit)                             │
│  - The Batch (Andrew Ng, hebdo)                             │
│  - Import AI (Jack Clark, hebdo)                            │
│  - Lenny's Newsletter (prod + AI)                           │
│                                                             │
│  COMMUNAUTÉS                                                │
│  - r/LocalLLaMA (Reddit) : IA locale                        │
│  - Hacker News : annonces, discussions techniques           │
│  - X/Twitter : @AnthropicAI, @OpenAI, @GoogleDeepMind      │
│                    @karpathy, @ylecun, @sama                │
│                                                             │
│  DÉPÔTS GITHUB                                              │
│  - awesome-llm : liste curatée de ressources                │
│  - ollama/ollama : releases et nouveaux modèles             │
│  - openai/evals : évaluations OpenAI                        │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 La cadence d'évolution

| Domaine | Rythme de changement | Comment s'adapter |
|---|---|---|
| **Nouveaux modèles** | Toutes les 2-4 semaines | Surveiller les annonces et tester rapidement |
| **Benchmarks** | Mis à jour en continu | Consulter LMSYS Arena et SWE-bench régulièrement |
| **Nouveaux outils** | Toutes les semaines | Suivre Product Hunt + HN pour les nouveautés |
| **Prix des APIs** | Baisses régulières | Réévaluer son stack tous les 3 mois |
| **Capacités des agents** | Bonds rapides | Tester les nouvelles versions sur tes vrais projets |

> [!info] Le bon état d'esprit face à l'évolution rapide
> Ne cherche pas à tout maîtriser — l'écosystème est trop vaste. Choisis **2 à 3 outils** que tu maîtrises profondément, surveille les grandes nouveautés, et réévalue ta stack tous les 3 à 6 mois. La profondeur d'usage compte plus que l'exhaustivité.

### 8.3 Les tendances à surveiller en 2026

```
TENDANCES MAJEURES 2026 :

  1. AGENTS DE PLUS EN PLUS AUTONOMES
     SWE-bench passe de 5% (2023) → 80%+ (2026)
     Les agents résolvent de vrais bugs GitHub seuls
     DeepSeek V4 (81%), Claude Opus 4.6 (80.8%)

  2. CONTEXTES TOUJOURS PLUS GRANDS
     1M tokens chez Claude/Gemini, 10M chez Llama 4 Scout
     Toute une codebase en un seul prompt

  3. MODÈLES SPÉCIALISÉS ET AGENTIQUES
     Codestral 2508, Devstral, Qwen3-Coder-Next
     L'IA agentique (Qwen 3.6-Plus, Devstral) devient clé

  4. IA LOCALE DE PLUS EN PLUS VIABLE
     Gemma 4 31B : meilleur open-source 2026
     Llama 4 Scout : 10M contexte en open-weight

  5. MULTIMODALITÉ GÉNÉRALISÉE
     Code + images + voix + vidéo → norme en 2026
     Décrire une UI en image → générer le code

  6. PRIX EN CHUTE LIBRE
     GPT-4 2023 : ~30$/M tokens output
     DeepSeek V4 2026 : ~0.50$/M tokens output
     Tendance à la poursuite
```

---

## 9. Synthèse : quelle stack IA pour un développeur en 2026 ?

> [!example] Stack recommandée pour un développeur solo ou junior

```
STACK IA RECOMMANDÉE — DÉVELOPPEUR 2026

  USAGE QUOTIDIEN (code)
  ├── Complétion inline : GitHub Copilot ou Supermaven
  └── Chat rapide : Claude Sonnet 4.6 ou GPT-5.4

  TÂCHES COMPLEXES
  ├── Architecture / design : Claude Opus 4.6
  └── Raisonnement algo : o3/o4 ou DeepSeek R2

  CODE CONFIDENTIEL
  └── Local : Ollama + Qwen3-Coder ou Llama 4 Scout

  BUDGET ZÉRO
  ├── Gemini 3.1 Flash-Lite (gratuit, dans la limite des quotas)
  └── Ollama + Phi-4 ou Gemma 4 31B (si GPU 48GB)

  IDE
  └── Cursor IDE ou VS Code + Continue.dev (configurable)
```

---

## Carte Mentale ASCII

```
                    PANORAMA IA POUR DÉVELOPPEURS
                              |
          ┌───────────────────┼───────────────────┐
          |                   |                    |
    CATÉGORIES           FAMILLES             CRITÈRES
    D'OUTILS             DE MODÈLES           DE CHOIX
          |                   |                    |
   ┌──────┼──────┐      ┌─────┼─────┐       ┌─────┼─────┐
   |      |      |      |     |     |       |     |     |
Complet. Chat  Agents  Anthro OpenAI Google Tâche Budget Confid.
inline        CLI/IDE  Claude GPT-5.4 Gemini  |     |     |
   |      |      |      |     |     |      Local Cloud RGPD
Copilot ChatGPT Claude Sonnet o3/o4 3.1Pro
Codeium Claude  Code    4.6        1M ctx
Supermav Gemini Cursor    |          |
TabNine         Windsurf Mistral  DeepSeek
                Aider    Codestral   V4/R2
                          |       Meta
                        Qwen    Llama 4
                        3-Coder Scout
                        Devstral    |
                          |      Phi-4
                        Local   Gemma4
                        Ollama  31B     |
                        LMStudio  BENCH-
                                  MARKS
                                    |
                             ┌──────┼──────┐
                             |      |      |
                          Human   SWE-   GPQA
                          Eval    bench
```

---

## Exercices Pratiques

### Exercice 1 — Benchmark personnel (2h)

Choisis une vraie tâche de code que tu dois faire (ex. : écrire une fonction de tri, créer un endpoint REST, déboguer une erreur). Soumets exactement le même prompt à **4 modèles différents** : Claude Sonnet 4.6, GPT-5.4, Gemini 3.1 Pro et un modèle local (Ollama + Qwen3-Coder).

Évalue chaque réponse sur :
- Correction du code (fonctionne ?)
- Qualité (lisibilité, bonnes pratiques)
- Vitesse de réponse
- Explications fournies

Documente tes résultats. Tu auras ainsi ton propre benchmark personnel adapté à tes cas d'usage réels.

### Exercice 2 — Installer et tester l'IA locale (1h30)

1. Installe Ollama sur ta machine (`curl https://ollama.ai/install.sh | sh` sur Linux/Mac)
2. Télécharge `ollama pull qwen3-coder:7b` (si peu de VRAM) ou `ollama pull gemma4:27b` (si GPU 24GB+)
3. Lance une session interactive et pose 5 questions de code de difficulté croissante
4. Compare les réponses avec celles de Claude Sonnet 4.6 ou GPT-5.4 sur les mêmes questions
5. Note : dans quels cas le modèle local est-il suffisant ? Dans quels cas la différence de qualité est-elle rédhibitoire ?

### Exercice 3 — Construire ta stack personnelle (30 min)

Réponds honnêtement à ces questions pour définir ta propre stack optimale :
1. Quel est ton budget mensuel pour les outils IA ? (0€ / <20€ / <100€ / illimité)
2. Travailles-tu sur du code confidentiel ou propriétaire ?
3. As-tu un GPU dédié ? Si oui, combien de VRAM ?
4. Tes projets sont-ils soumis au RGPD ou à des contraintes européennes ?
5. Quelle est ta tâche IA principale ? (débogage, génération, architecture, review)

Basé sur tes réponses et le tableau de la section 7, identifie les 2-3 outils optimaux pour ton profil. Installe-les cette semaine et engage-toi à les utiliser quotidiennement pendant 2 semaines avant d'évaluer.

---

## Liens et Ressources

- [[02 - Comprendre les LLMs et les Tokens]] — Comprendre comment les modèles fonctionnent réellement : tokens, fenêtres de contexte, température
- [[03 - Prompt Engineering pour le Code]] — Techniques avancées pour formuler des prompts qui donnent des résultats fiables
- [[04 - Claude et Claude Code Maitrise Avancee]] — Maîtrise approfondie de l'écosystème Claude : API, Claude Code, agents, CLAUDE.md
- [[09 - IA Locale avec Ollama]] — Installation, configuration et optimisation des modèles locaux avec Ollama et LM Studio
- [[12 - Strategies Multi-Modeles et Workflows]] — Combiner plusieurs IA dans des workflows automatisés pour maximiser qualité et réduire les coûts
