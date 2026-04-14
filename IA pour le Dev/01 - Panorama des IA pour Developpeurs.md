# 01 - Panorama des IA pour Développeurs

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
capables     pour le      public        o1 reasoning  DeepSeek V3    Gemini 2.5
de coder     code                       Code agents   Qwen2.5 Coder  o3 / R1
                                        Mistral       Codestral      SWE-bench
                                        Llama 2       Llama 3.3      90%+ scores

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
| **GitHub Copilot** | GPT-4o + modèles Microsoft | 10$/mois | Le plus répandu, excellent contexte projet |
| **Codeium** | Modèle propriétaire | Gratuit (plan de base) | Rapide, bon support multi-langages |
| **Supermaven** | Modèle propriétaire (Sourcegraph) | Gratuit / 10$+ | Contexte 1M tokens, très rapide |
| **Tabnine** | Modèles propriétaires | Gratuit / 12$/mois | Option on-premise, RGPD-friendly |

> [!info] Comment fonctionne la complétion inline
> L'outil envoie le contexte autour de ton curseur (fichier courant, fichiers ouverts, dépôts récents) au modèle IA. Le modèle prédit la suite probable. La complétion apparaît en gris — `Tab` pour accepter, `Esc` pour ignorer. Certains outils comme Supermaven peuvent voir jusqu'à 1 million de tokens de contexte, ce qui inclut tout ton projet.

#### 2.2 Chat assistants

Interfaces conversationnelles pour poser des questions, obtenir des explications, générer du code à la demande.

| Outil | Modèle | Contexte | Forces |
|---|---|---|---|
| **ChatGPT** | GPT-4o, o3 | 128K tokens | Très polyvalent, plugins, image |
| **Claude** | Claude Sonnet/Opus 4.x | 200K tokens | Raisonnement, code long, suivi d'instructions |
| **Gemini** | Gemini 2.5 Pro | 2M tokens (!) | Contexte exceptionnel, gratuit en partie |
| **Le Chat (Mistral)** | Mistral Large 2, Codestral | 128K tokens | Souverain EU, Codestral pour le code |

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
| **Cursor IDE** | VS Code fork | GPT-4o, Claude, Gemini | Le plus populaire, agents "Composer" |
| **Windsurf** | VS Code fork | Modèles Codeium | "Cascade" agent très autonome |
| **Cline** | Extension VS Code | Multi-modèles | Open source, contrôle fin des permissions |
| **Continue.dev** | Extension VS Code/JetBrains | Multi-modèles | Open source, ultra-configurable |

#### 2.5 IA locale avec Ollama et LM Studio

Faire tourner des modèles sur ta propre machine — aucune donnée ne quitte ton réseau.

```bash
# Exemple Ollama
$ ollama pull qwen2.5-coder:32b
$ ollama run qwen2.5-coder:32b
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
│  Contexte : 200K tokens   Sortie : jusqu'à 32K tokens      │
│  Vision : Oui             Code : Excellent                 │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Vitesse | Qualité | Prix indicatif (input/output) | Idéal pour |
|---|---|---|---|---|
| **claude-haiku-4-5** | Très rapide | Bonne | ~$0.80 / $4 par M tokens | Tâches répétitives, complétion, chat rapide |
| **claude-sonnet-4-6** | Rapide | Excellente | ~$3 / $15 par M tokens | Code complexe, agents, usage quotidien |
| **claude-opus-4-6** | Modéré | Maximale | ~$15 / $75 par M tokens | Architecture, raisonnement profond, tâches critiques |

**Forces Claude :** suivi d'instructions remarquable, honnêteté (dit quand il ne sait pas), fenêtre de contexte 200K, excellent pour le code long et le refactoring.

**Faiblesses :** plus coûteux que certains concurrents, pas encore de recherche web native dans tous les contextes.

### 3.2 OpenAI — GPT et série O

OpenAI reste la référence grand public avec ChatGPT, et pousse l'innovation des modèles de raisonnement.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE OpenAI                                            │
│                                                            │
│  GPT-4o          →  Polyvalent, multimodal, rapide         │
│  GPT-4o-mini     →  Économique, tâches courantes           │
│  o1              →  Raisonnement lent et profond           │
│  o1-mini         →  Raisonnement économique                │
│  o3              →  Meilleur raisonnement disponible       │
│  o3-mini         →  Raisonnement rapide et abordable       │
│                                                            │
│  Contexte : 128K tokens   Vision : Oui                     │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Type | Prix indicatif | Forces | Faiblesses |
|---|---|---|---|---|
| **GPT-4o** | Polyvalent | ~$5 / $15 par M tokens | Multimodal, rapide, excellent chat | Moins fort en raisonnement pur |
| **GPT-4o-mini** | Économique | ~$0.15 / $0.6 par M tokens | Prix minimal, bon pour le volume | Qualité réduite sur tâches complexes |
| **o1** | Raisonnement | ~$15 / $60 par M tokens | Problèmes mathématiques/algorithmiques | Lent, cher, pas multimodal |
| **o3** | Raisonnement max | Variable | Meilleur du monde en benchmark | Très lent, coût élevé |
| **o3-mini** | Raisonnement rapide | ~$1.1 / $4.4 par M tokens | Bon compromis vitesse/raisonnement | Moins fort qu'o3 full |

> [!info] Les modèles "o" : qu'est-ce que le raisonnement ?
> Les modèles o1/o3 utilisent une technique appelée **chain-of-thought interne** : avant de répondre, le modèle réfléchit en silence pendant plusieurs secondes à minutes. Ce "temps de réflexion" leur permet de résoudre des problèmes mathématiques, logiques et algorithmiques complexes bien mieux que les modèles standard. Le prix à payer : la lenteur et le coût.

### 3.3 Google — Gemini

Google a fait une percée majeure avec Gemini 2.5 Pro et son contexte de 2 millions de tokens — un avantage concurrentiel unique.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE GEMINI (Google DeepMind)                          │
│                                                            │
│  Gemini 2.0 Flash    →  Rapide, gratuit en API             │
│  Gemini 2.5 Pro      →  2M tokens contexte, excellent code │
│  Gemini CLI          →  Outil CLI gratuit (beta 2025)      │
│                                                            │
│  Contexte : jusqu'à 2M tokens   Vision : Oui              │
│  Gratuit : Oui (limites)        Code : Très bon            │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Contexte | Prix indicatif | Forces | Faiblesses |
|---|---|---|---|---|
| **Gemini 2.0 Flash** | 1M tokens | Gratuit (quota) / ~$0.075/M | Vitesse, coût nul | Moins puissant que 2.5 Pro |
| **Gemini 2.5 Pro** | 2M tokens | ~$1.25 / $5 par M tokens | Contexte géant, code, multimodal | Parfois verbeux |
| **Gemini CLI** | 2.5 Pro gratuit | Gratuit (beta) | Accès terminal direct, gratuit | En beta, limitations |

**Le contexte de 2M tokens de Gemini 2.5 Pro** est révolutionnaire : tu peux envoyer un projet entier de 1,5 million de lignes de code en une seule requête pour l'analyser ou le refactoriser.

### 3.4 Mistral AI — Le champion européen

Mistral AI, startup française, offre des modèles souverains (hébergés en Europe) et le meilleur modèle de code spécialisé open-weights : Codestral.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE MISTRAL AI (Paris, France)                        │
│                                                            │
│  Mistral Large 2     →  Modèle flagship, polyvalent        │
│  Codestral           →  Spécialisé code, 80K contexte      │
│  Mistral 7B          →  Léger, rapide, open-weights        │
│                                                            │
│  RGPD : Oui (hébergement EU)   Open-weights : Partiel      │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Paramètres | Prix indicatif | Forces | Faiblesses |
|---|---|---|---|---|
| **Mistral Large 2** | ~123B | ~$2 / $6 par M tokens | Polyvalent, multilingue, EU | Légèrement en retrait sur le code vs Claude |
| **Codestral** | ~22B | ~$0.1 / $0.3 par M tokens | Spécialisé code, 80K contexte, rapide | Moins polyvalent |
| **Mistral 7B** | 7B | Gratuit (local) | Ultra-léger, tourne sur CPU | Qualité limitée |

**Pourquoi Codestral est intéressant :** modèle de code spécialisé à 80K tokens de contexte, prix minimal, utilisable dans Cursor/Continue.dev. Idéal pour les équipes avec contraintes budgétaires ou RGPD.

### 3.5 Meta — Llama (open source)

Meta a fait le choix stratégique de l'open source : les modèles Llama sont téléchargeables librement et peuvent tourner localement.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE LLAMA (Meta AI) — OPEN SOURCE                     │
│                                                            │
│  Llama 3.1  8B    →  Léger, pour CPU/GPU modeste          │
│  Llama 3.1  70B   →  Très bon, GPU 48GB+ requis           │
│  Llama 3.1  405B  →  Rival GPT-4, GPU cluster requis      │
│  Llama 3.3  70B   →  Meilleur que 3.1 70B pour le code    │
│  CodeLlama        →  Spécialisé code (ancienne génération) │
│                                                            │
│  Licence : Meta Llama License (usage commercial autorisé  │
│  sous conditions pour les grands acteurs)                  │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Taille | RAM GPU requise | Forces | Faiblesses |
|---|---|---|---|---|
| **Llama 3.1 8B** | 8B | ~6 GB VRAM | Léger, rapide, local | Qualité basique |
| **Llama 3.3 70B** | 70B | ~40 GB VRAM | Excellent code, gratuit | Matériel conséquent requis |
| **Llama 3.1 405B** | 405B | Cluster GPU | Rival GPT-4 | Inaccessible localement |

### 3.6 Alibaba — Qwen (le champion du code)

La famille Qwen2.5-Coder d'Alibaba est devenue l'une des références pour le code en open-weights, rivalisant avec des modèles propriétaires bien plus chers.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE QWEN (Alibaba Cloud) — OPEN-WEIGHTS               │
│                                                            │
│  Qwen2.5-Coder 7B   →  Léger, très bon pour sa taille     │
│  Qwen2.5-Coder 14B  →  Excellent rapport qualité/taille   │
│  Qwen2.5-Coder 32B  →  Rival Codestral, très capable      │
│  Qwen2.5-Coder 72B  →  Meilleur open-weights pour code    │
│  Qwen2.5 72B        →  Polyvalent, excellent code          │
│                                                            │
│  Disponible sur : Ollama, LM Studio, Hugging Face          │
└────────────────────────────────────────────────────────────┘
```

> [!example] Qwen2.5-Coder 32B en local
> ```bash
> $ ollama pull qwen2.5-coder:32b
> $ ollama run qwen2.5-coder:32b
> >>> Implémente un algorithme de Dijkstra en Rust avec gestion d'erreurs
> ```
> Ce modèle de 32B tourne sur un GPU grand public (RTX 4090 ou similaire) et produit du code Python, Rust, TypeScript, Java de qualité comparable à GPT-4 pour la majorité des tâches.

### 3.7 DeepSeek — La surprise chinoise

DeepSeek a créé la surprise en 2024-2025 avec des modèles open-source rivaux des meilleurs modèles propriétaires, à des prix dérisoires.

```
┌────────────────────────────────────────────────────────────┐
│  FAMILLE DEEPSEEK (DeepSeek AI, Chine) — OPEN-WEIGHTS      │
│                                                            │
│  DeepSeek-V3         →  Rival GPT-4o, très économique     │
│  DeepSeek-R1         →  Raisonnement, rival o1             │
│  DeepSeek-Coder-V2   →  Excellent pour le code            │
│                                                            │
│  Prix API : 10× moins cher que OpenAI (approximatif)       │
│  Risque : données hébergées en Chine                       │
└────────────────────────────────────────────────────────────┘
```

| Modèle | Forces | Faiblesses | Prix indicatif |
|---|---|---|---|
| **DeepSeek-V3** | Qualité GPT-4o, prix minimal | Latence, infrastructure instable | ~$0.27 / $1.1 par M tokens |
| **DeepSeek-R1** | Raisonnement excellent, open-weights | Lent, verbose | ~$0.55 / $2.19 par M tokens |
| **DeepSeek-Coder-V2** | Code de très haute qualité | Politique de confidentialité | Variable |

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

### 3.9 Zhipu AI — GLM-4

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

### 4.2 Tableau comparatif des scores (approximatifs, 2025-2026)

| Modèle | HumanEval | SWE-bench | MBPP | MMLU |
|---|---|---|---|---|
| **Claude Opus 4.6** | ~92% | ~72% | ~90% | ~90% |
| **Claude Sonnet 4.6** | ~90% | ~65% | ~88% | ~88% |
| **GPT-4o** | ~90% | ~49% | ~87% | ~88% |
| **o3** | ~98% | ~70%+ | ~96% | ~91% |
| **Gemini 2.5 Pro** | ~91% | ~63% | ~89% | ~90% |
| **DeepSeek-V3** | ~89% | ~42% | ~86% | ~88% |
| **DeepSeek-R1** | ~93% | ~50% | ~90% | ~90% |
| **Qwen2.5-Coder 72B** | ~88% | ~45% | ~85% | ~84% |
| **Llama 3.3 70B** | ~82% | ~35% | ~80% | ~80% |
| **Mistral Large 2** | ~85% | ~38% | ~83% | ~84% |
| **Codestral** | ~81% | ~32% | ~79% | ~76% |
| **Phi-4 (3.8B)** | ~77% | ~20% | ~74% | ~72% |

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
| Claude Haiku 4.5 | Anthropic | Chat/API | 200K | Vitesse, économique | ~$0.80 | Non |
| Claude Sonnet 4.6 | Anthropic | Chat/API/Agent | 200K | Équilibre qualité/coût | ~$3 | Non |
| Claude Opus 4.6 | Anthropic | Chat/API/Agent | 200K | Intelligence maximale | ~$15 | Non |
| GPT-4o | OpenAI | Chat/API | 128K | Polyvalent, multimodal | ~$5 | Non |
| GPT-4o-mini | OpenAI | Chat/API | 128K | Prix minimal | ~$0.15 | Non |
| o3 | OpenAI | Raisonnement | 128K | Problèmes complexes | Variable | Non |
| o3-mini | OpenAI | Raisonnement | 128K | Raisonnement rapide | ~$1.1 | Non |
| Gemini 2.0 Flash | Google | Chat/API | 1M | Rapidité, gratuit | ~$0.075 | Non |
| Gemini 2.5 Pro | Google | Chat/API | 2M | Contexte géant, code | ~$1.25 | Non |
| Mistral Large 2 | Mistral AI | Chat/API | 128K | Souverain EU | ~$2 | Non |
| Codestral | Mistral AI | Code/API | 80K | Code spécialisé | ~$0.1 | Non |
| DeepSeek-V3 | DeepSeek | Chat/API | 64K | Prix dérisoire | ~$0.27 | Oui (weights) |
| DeepSeek-R1 | DeepSeek | Raisonnement | 64K | Open reasoning | ~$0.55 | Oui (weights) |
| Qwen2.5-Coder 32B | Alibaba | Code | 128K | Code open-weights | Gratuit (local) | Oui |
| Qwen2.5-Coder 72B | Alibaba | Code | 128K | Meilleur open code | Gratuit (local) | Oui |
| Llama 3.3 70B | Meta | Polyvalent | 128K | Open source, code | Gratuit (local) | Oui |
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
│       ├── Rapide / simple ? ──── Claude Haiku / GPT-4o-mini    │
│       │                          Gemini 2.0 Flash               │
│       │                                                         │
│       ├── Code confidentiel ? ── IA Locale : Ollama + Qwen /   │
│       │   (propriétaire,         LM Studio + Llama             │
│       │    RGPD sensible)        Tabnine on-premise             │
│       │                                                         │
│       ├── Gros projet /  ─────── Gemini 2.5 Pro (2M ctx)       │
│       │   codebase entier         Claude Sonnet 4.6 (200K)     │
│       │                                                         │
│       ├── Raisonnement ─────────  o3 / o3-mini                 │
│       │   mathématique/algo       DeepSeek-R1                  │
│       │                           Claude Opus 4.6              │
│       │                                                         │
│       ├── Budget serré ─────────  Gemini 2.0 Flash (gratuit)   │
│       │                           GPT-4o-mini                  │
│       │                           Codestral                    │
│       │                                                         │
│       ├── Génération en masse ──  GPT-4o-mini, Haiku 4.5       │
│       │   (milliers de requêtes)  Gemini 2.0 Flash             │
│       │                                                         │
│       └── Architecture système ─  Claude Opus 4.6              │
│           / design complexe       o3 (si budget ok)            │
└─────────────────────────────────────────────────────────────────┘
```

### Tableau tâche → IA recommandée

| Tâche | IA recommandée | Raison |
|---|---|---|
| **Question rapide / explication** | Claude Haiku, GPT-4o-mini, Gemini Flash | Réponse quasi-instantanée, coût minimal |
| **Débogage complexe** | Claude Sonnet 4.6, GPT-4o | Excellent suivi du raisonnement sur du code existant |
| **Architecture système** | Claude Opus 4.6, o3 | Raisonnement profond sur des contraintes multiples |
| **Code propriétaire / confidentiel** | Ollama + Qwen2.5-Coder ou Llama 3.3 | 100% local, aucune donnée envoyée |
| **Budget serré** | Gemini 2.0 Flash (gratuit), Codestral | Qualité correcte à coût nul ou minimal |
| **Génération en masse (API)** | GPT-4o-mini, Claude Haiku, Gemini Flash | Faible coût par token, rate limits élevés |
| **Raisonnement mathématique** | o3, DeepSeek-R1, Claude Opus 4.6 | Chain-of-thought profond |
| **Refactoring de gros codebase** | Gemini 2.5 Pro (2M tokens) | Voit tout le projet en une seule requête |
| **Code spécialisé (complétion IDE)** | GitHub Copilot, Supermaven, Codeium | Optimisés pour l'inline completion |
| **Projets RGPD / entreprise EU** | Mistral Large 2, Codestral | Hébergement en Europe, conformité EU |

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
| **Startup, projet perso** | Cloud (Claude Sonnet, GPT-4o) | Pas d'infra à gérer, accès aux meilleurs modèles |
| **Entreprise avec données sensibles** | Local ou cloud EU (Mistral) | Conformité RGPD, secret industriel |
| **Développeur isolé sans internet** | Local (Ollama + Llama/Qwen) | Fonctionne hors connexion |
| **Étudiants / budget zéro** | Gemini Flash gratuit + Ollama local | Sans débourser un centime |
| **Grande équipe, volume élevé** | Cloud API (coût prévisible) | Scalabilité, monitoring centralisé |
| **Code très propriétaire (banque, défense)** | Local on-premise uniquement | Aucune fuite possible |

> [!tip] Analogie
> Choisir entre cloud et local, c'est comme choisir entre **louer une salle de sport** (cloud) ou **acheter les équipements** (local) :
> - La salle de sport (cloud) est toujours à jour, accessible partout, mais tu paies à chaque visite et tu n'es pas chez toi
> - Les équipements chez toi (local) : investissement initial, mais disponibles 24h/24, sans abonnement, et personne ne regarde ce que tu fais

### Configuration matérielle recommandée pour le local

```
GPU VRAM  →  Modèles utilisables
  8 GB    →  Phi-4 (3.8B), Qwen2.5-Coder 7B, Llama 3.1 8B
 16 GB    →  Qwen2.5-Coder 14B, Mistral 7B quantisé
 24 GB    →  Qwen2.5-Coder 32B (bonne qualité), CodeLlama 34B
 48 GB    →  Llama 3.3 70B (quantisé), Qwen2.5-Coder 72B (quantisé)
 80 GB+   →  Modèles 70B en pleine précision, DeepSeek-V3 partiel
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
     SWE-bench passe de 5% (2023) → 70%+ (2025)
     Les agents résolvent de vrais bugs GitHub seuls

  2. CONTEXTES TOUJOURS PLUS GRANDS
     2M tokens chez Gemini → probablement 10M+ en 2027
     Toute une codebase en un seul prompt

  3. MODÈLES SPÉCIALISÉS DÉPASSENT LES GÉNÉRALISTES
     Codestral, Qwen2.5-Coder > GPT-4o sur le code
     La spécialisation fine devient clé

  4. IA LOCALE DE PLUS EN PLUS VIABLE
     Phi-4 3.8B rivalise avec GPT-3.5
     Les petits modèles rattrapent leur retard

  5. MULTIMODALITÉ GÉNÉRALISÉE
     Code + images + voix + vidéo → norme en 2026
     Décrire une UI en image → générer le code

  6. PRIX EN CHUTE LIBRE
     GPT-4 2023 : ~30$/M tokens output
     GPT-4o-mini 2025 : ~0.6$/M tokens
     Tendance à la poursuite
```

---

## 9. Synthèse : quelle stack IA pour un développeur en 2026 ?

> [!example] Stack recommandée pour un développeur solo ou junior

```
STACK IA RECOMMANDÉE — DÉVELOPPEUR 2026

  USAGE QUOTIDIEN (code)
  ├── Complétion inline : GitHub Copilot ou Supermaven
  └── Chat rapide : Claude Sonnet 4.6 ou GPT-4o

  TÂCHES COMPLEXES
  ├── Architecture / design : Claude Opus 4.6
  └── Raisonnement algo : o3-mini ou DeepSeek-R1

  CODE CONFIDENTIEL
  └── Local : Ollama + Qwen2.5-Coder 32B

  BUDGET ZÉRO
  ├── Gemini 2.0 Flash (gratuit, dans la limite des quotas)
  └── Ollama + Phi-4 ou Llama 3.3 70B (si GPU)

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
inline        CLI/IDE  Claude GPT-4o Gemini   |     |     |
   |      |      |      |     |     |      Local Cloud RGPD
Copilot ChatGPT Claude Sonnet o3   2.5Pro
Codeium Claude  Code    4.6        2M ctx
Supermav Gemini Cursor    |          |
TabNine         Windsurf Mistral  DeepSeek
                Aider    Codestral   V3/R1
                          |       Meta
                        Qwen    Llama 3.3
                        Coder   70B open
                        7-72B       |
                          |      Phi-4
                        Local     3.8B
                        Ollama      |
                        LMStudio  BENCH-
                                  MARKS
                                    |
                             ┌──────┼──────┐
                             |      |      |
                          Human   SWE-   MMLU
                          Eval    bench
```

---

## Exercices Pratiques

### Exercice 1 — Benchmark personnel (2h)

Choisis une vraie tâche de code que tu dois faire (ex. : écrire une fonction de tri, créer un endpoint REST, déboguer une erreur). Soumets exactement le même prompt à **4 modèles différents** : Claude Sonnet, GPT-4o, Gemini 2.5 Pro et un modèle local (Ollama + Qwen2.5-Coder).

Évalue chaque réponse sur :
- Correction du code (fonctionne ?)
- Qualité (lisibilité, bonnes pratiques)
- Vitesse de réponse
- Explications fournies

Documente tes résultats. Tu auras ainsi ton propre benchmark personnel adapté à tes cas d'usage réels.

### Exercice 2 — Installer et tester l'IA locale (1h30)

1. Installe Ollama sur ta machine (`curl https://ollama.ai/install.sh | sh` sur Linux/Mac)
2. Télécharge `ollama pull qwen2.5-coder:7b` (si peu de VRAM) ou `:32b` (si GPU 24GB+)
3. Lance une session interactive et pose 5 questions de code de difficulté croissante
4. Compare les réponses avec celles de Claude ou GPT-4o sur les mêmes questions
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
