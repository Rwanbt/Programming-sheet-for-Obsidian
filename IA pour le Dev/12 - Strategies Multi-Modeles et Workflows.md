# 12 - Stratégies Multi-Modèles et Workflows

> [!info] Mis à jour avril 2026
> Ce fichier a été mis à jour avec les modèles et prix en vigueur en avril 2026.

> [!warning] Les prix changent souvent
> Les tarifs des APIs IA évoluent régulièrement (parfois toutes les semaines). Vérifiez toujours les pages de tarification officielles avant de budgétiser un projet : [anthropic.com/pricing](https://www.anthropic.com/pricing), [openai.com/pricing](https://openai.com/pricing), [ai.google.dev/pricing](https://ai.google.dev/pricing), [deepseek.com](https://www.deepseek.com).

## Qu'est-ce que la stratégie multi-modèles ?

La stratégie multi-modèles est l'art de sélectionner et combiner intelligemment plusieurs modèles d'IA en fonction de la nature de chaque tâche, du budget disponible et des contraintes de confidentialité. Plutôt que de s'en remettre aveuglément à un seul modèle "universel", le développeur expert apprend à router ses besoins vers l'outil le plus adapté — comme un chef cuisinier qui sait exactement quel couteau utiliser selon l'ingrédient.

> [!tip] Analogie
> Imaginez votre cuisine équipée de couteaux spécialisés : le couteau à pain, le couteau de chef, l'éminceur, le couteau à filet. Utiliser un couteau à pain pour émincer des oignons, c'est possible... mais inefficace et coûteux en effort. Il en va de même avec les modèles d'IA : Claude Opus 4.6 pour générer un docstring simple, c'est gaspiller $25 par million de tokens output pour une tâche qu'un modèle à $1.25 ferait parfaitement.

Le développeur expert de 2025-2026 ne pose plus la question "Quel est le meilleur modèle ?" mais "Quel modèle est le meilleur **pour cette tâche précise, dans ce contexte précis** ?"

---

## 1. Penser Multi-Modèle : Le Paradigme du Développeur Moderne

### Aucun modèle ne domine dans tous les domaines

Les benchmarks académiques (MMLU, HumanEval, SWE-bench) révèlent une vérité nuancée : chaque modèle excelle dans certains contextes et sous-performe dans d'autres. Un modèle champion sur la mathématique formelle peut être moins fluide en génération de code idiomatique Python. Un modèle ultra-rapide et bon marché peut traiter 90% de vos besoins quotidiens avec une qualité parfaitement suffisante.

```
Benchmarks représentatifs (indicatifs, avril 2026)
===================================================

SWE-bench Verified (résolution de vrais bugs GitHub) :
  DeepSeek V4        ████████████████████ 81%
  Claude Opus 4.6    ███████████████████  80.8%
  Gemini 3.1 Pro     ██████████████████   78.8%
  Claude Sonnet 4.6  █████████████████    ~70%
  GPT-5.4*           ███████████████      57.7%
  Claude Haiku 4.5   █████████████        ~55%
  Ollama Gemma 4 31B ███████████████████  80%** (LiveCodeBench)

*SWE-bench Pro (métrique différente de SWE-bench Verified)
**LiveCodeBench v6 (pas SWE-bench Verified)

Vitesse de réponse (tokens/sec, approximatif) :
  GPT-5.4 Nano       ████████████████████ 200+ tk/s
  Claude Haiku 4.5   ██████████████████   150+ tk/s
  Gemini 3.1 Flash-L ████████████████     130+ tk/s
  Claude Sonnet 4.6  ████████████         90+ tk/s
  Gemini 3.1 Pro     ██████████           70+ tk/s
  GPT-5.4            ████████             60+ tk/s
  Claude Opus 4.6    ██████               45+ tk/s
  DeepSeek V4        █████████            55+ tk/s
```

> [!info]
> Les benchmarks vieillissent vite. Les modèles sortent tous les 2-3 mois. Ce qui compte, c'est de comprendre la **méthode d'évaluation** pour pouvoir l'appliquer aux nouveaux modèles dès leur sortie.

### Construire sa boîte à outils IA

Un développeur senior ne se contente pas d'un seul IDE, d'un seul langage, d'un seul framework. De même, le développeur augmenté par l'IA de 2025 maintient un écosystème de modèles soigneusement sélectionnés :

```
MA BOÎTE À OUTILS IA (exemple, avril 2026)
===========================================

┌─────────────────────────────────────────────────────┐
│  COMPLÉTION INLINE          │  Continue.dev +        │
│  (pendant que je code)      │  Qwen3-Coder-Next local│
├─────────────────────────────┼────────────────────────┤
│  QUESTIONS RAPIDES          │  Haiku 4.5 / GPT-5.4   │
│  (5-10 fois par jour)       │  Nano / Gemini Flash-L │
├─────────────────────────────┼────────────────────────┤
│  DÉVELOPPEMENT FEATURES     │  Claude Sonnet 4.6     │
│  (tâche principale)         │  / Gemini 3.1 Pro      │
├─────────────────────────────┼────────────────────────┤
│  PROBLÈMES DIFFICILES       │  Claude Opus 4.6 / o3  │
│  (quand bloqué)             │  / Gemini 3.1 Pro      │
├─────────────────────────────┼────────────────────────┤
│  CODE CONFIDENTIEL          │  Ollama local          │
│  (données sensibles)        │  Gemma 4 31B / Llama 4 │
└─────────────────────────────┴────────────────────────┘
```

---

## 2. Matrice Coût/Qualité Avril 2026

| Modèle | Coût input (1M tk) | Coût output (1M tk) | Qualité code /10 | SWE-bench | Contexte |
|--------|--------------------|---------------------|------------------|-----------|---------|
| Claude Haiku 4.5 | $1 | $5 | 7/10 | ~55% | 200k |
| Claude Sonnet 4.6 | $3 | $15 | 9/10 | ~70% | 1M |
| Claude Opus 4.6 | $5 | $25 | 10/10 | **80.8%** | 1M |
| GPT-5.4 | $2.50 | $15 | 8.5/10 | 57.7%* | 272K |
| GPT-5.4 Nano | $0.20 | $1.25 | 6.5/10 | - | 400K |
| Gemini 3.1 Pro | $2 | $12 | 9/10 | **78.8%** | 1M |
| Gemini 3.1 Flash-Lite | $0.25 | $1.50 | 7/10 | - | ? |
| DeepSeek V4 | $0.30 | $0.50 | 9.5/10 | **81%** | 1M |
| Mistral Small 4 | ~$0.50 | ~$1.50 | 7.5/10 | - | 128K |
| Ollama (Gemma 4 31B) | $0 | $0 | 7.5/10 | 80%** | 256K |

*SWE-bench Pro (métrique différente de SWE-bench Verified)
**LiveCodeBench v6 (pas SWE-bench Verified)

> [!warning] DeepSeek V4 : le cas surprenant
> DeepSeek V4 obtient **81% SWE-bench Verified** — meilleur score du marché — à $0.30/M tokens (vs $5/M pour Claude Opus). Cependant, c'est une entreprise chinoise : à ne pas utiliser pour du code propriétaire ou sensible (voir note 11).

### Lecture de la matrice

```
QUADRANT COÛT/QUALITÉ (avril 2026)
====================================

Qualité
 9.5 │● DeepSeek V4 ($0.30)
     │
  10 │                              ● Opus 4.6 ($5)
     │
   9 │         ● Gemini 3.1 Pro ($2)  ● Sonnet 4.6 ($3)
     │
 8.5 │               ● GPT-5.4 ($2.50)
     │
 7.5 │  ● Gemma 4 31B local  ● Mistral Small 4
     │
   7 │● GPT-5.4 Nano ($0.20)  ● Haiku 4.5 ($1)  ● Gemini Flash-L ($0.25)
     │
 6.5 │
     └──────────────────────────────────────────── Coût
       $0    $0.30   $1     $2-3    $5+

Zone verte  : DeepSeek V4, Gemini Flash-Lite (excellent rapport)
Zone bleue  : Sonnet 4.6, Gemini 3.1 Pro (équilibre premium)
Zone rouge  : Opus 4.6 (réserver aux cas critiques)
Zone locale : Ollama/Gemma 4 (hors marché, confidentialité maximale)
```

---

## 3. Routing des Tâches : Quel Modèle pour Quoi ?

### Résumé du routing (avril 2026)

| Cas d'usage | Modèle recommandé | Coût indicatif |
|-------------|-------------------|----------------|
| Questions simples/rapides | Claude Haiku 4.5 ou GPT-5.4 Nano | $0.20-1/M |
| Dev quotidien | Claude Sonnet 4.6 ou Gemini 3.1 Pro | $2-3/M |
| Problèmes difficiles | Claude Opus 4.6 ou Gemini 3.1 Pro | $5/M |
| Code open source, budget minimal | DeepSeek V4 ou Gemini 3.1 Flash-Lite | $0.25-0.30/M |
| Complétion inline gratuite | Continue.dev + Qwen3-Coder-Next local | $0 |
| Code propriétaire | Ollama (Gemma 4 ou Llama 4 Scout) | $0 |
| Analyse grand repo | Gemini 3.1 Pro (1M ctx) ou Llama 4 Scout local (10M ctx) | $2/M ou $0 |
| Raisonnement mathématique/algo | o3/o4 (OpenAI) ou Claude Opus | $5+/M |
| Génération en masse cheap | DeepSeek V4 avec cache | $0.03/M (tokens mis en cache) |

### Tâches quotidiennes rapides → Modèle économique

Ces tâches représentent souvent 60-70% du volume de requêtes mais ne nécessitent pas la puissance d'un Opus.

| Tâche | Modèle recommandé | Justification |
|---|---|---|
| Questions simples de syntaxe | GPT-5.4 Nano / Haiku 4.5 / Gemini Flash-Lite | Réponse immédiate, contexte court |
| Complétion inline | Continue.dev + Qwen3-Coder-Next local | Latence critique, intégré IDE, gratuit |
| Reformulation de texte | N'importe quel modèle léger | Tâche triviale |
| Génération de boilerplate | Haiku 4.5 ou Ollama local | Répétitif, pattern simple |
| Conversion de format | Gemini Flash-Lite ou Haiku | Transformation mécanique |
| Explication de code court | Haiku 4.5 ou GPT-5.4 Nano | Contexte limité suffisant |

### Tâches de qualité → Modèle équilibré

Le coeur du travail développeur quotidien : suffisamment complexe pour nécessiter un bon modèle, mais pas au point de justifier un modèle premium.

| Tâche | Modèle recommandé | Alternatives |
|---|---|---|
| Développement de features | Claude Sonnet 4.6 | Gemini 3.1 Pro, DeepSeek V4 (open source) |
| Debugging complexe | Claude Sonnet 4.6 | Claude Opus 4.6 si bloqué |
| Code review complet | Claude Sonnet 4.6 | Gemini 3.1 Pro |
| Architecture système | Claude Sonnet 4.6 / Opus 4.6 | Gemini 3.1 Pro |
| Tests unitaires | Sonnet 4.6 ou Haiku 4.5 | Haiku souvent suffisant |
| Documentation technique | Claude Sonnet 4.6 | GPT-5.4 |
| Refactoring modéré | Claude Sonnet 4.6 | DeepSeek V4 (open source) |

### Tâches critiques → Modèle premium

À utiliser avec parcimonie — c'est là que les $5-25/1M tokens output se justifient.

| Tâche | Modèle recommandé | Pourquoi premium ? |
|---|---|---|
| Algorithmes complexes / maths | o3/o4 (OpenAI) ou Claude Opus 4.6 | Raisonnement multi-étapes profond |
| Architecture de sécurité | Claude Opus 4.6 | Erreur = faille critique |
| Débogage profond et difficile | Claude Opus 4.6 ou o3 | Diagnostic exhaustif nécessaire |
| Refactoring massif multi-fichiers | Claude Opus 4.6 | Cohérence sur grand contexte (1M tk) |
| Design de protocoles | Claude Opus 4.6 | Implications à long terme |

### Tâches privées → Local obligatoire

> [!warning]
> Cette catégorie n'est pas une question de coût mais d'éthique et de conformité légale. Envoyer du code propriétaire ou des données clients vers un cloud tiers peut violer des contrats, le RGPD, ou des politiques d'entreprise.

| Tâche | Modèle local recommandé | Configuration |
|---|---|---|
| Code propriétaire sensible | Gemma 4 31B ou Llama 4 Scout | Ollama local |
| Traitement de données clients | Tout modèle local | Air-gapped si possible |
| Secrets, credentials | Gemma 4 local | Ne JAMAIS uploader |
| Environnement air-gapped | Ollama + modèle local | vLLM en serveur interne |
| IP d'entreprise critique | Serveur Ollama interne (vLLM) | Réseau isolé |

### Tâches en masse → Batch API

| Tâche | Approche | Économie |
|---|---|---|
| Génération de 1000 docstrings | API batch Anthropic (-50%) | Haiku 4.5 batch ~$0.50/1M tk |
| Tests pour tout un projet | Claude Haiku 4.5 en batch | Faible coût, haut volume |
| Traduction de commentaires | Haiku 4.5 ou Ollama | Tâche parallélisable |
| Analyse de logs massifs | Gemini 3.1 Pro (1M contexte) | Contexte énorme, coût raisonnable |
| Génération en masse open source | DeepSeek V4 avec prompt cache | $0.03/M tokens mis en cache |

---

## 4. Workflows par Profil Développeur

### L'Étudiant (budget zéro)

```
WORKFLOW ÉTUDIANT (avril 2026)
==============================

COMPLÉTION    ──► Gemini 3.1 Flash-Lite API gratuite (quota généreux)
                  ou Ollama + Gemma 4 E4B
                  Coût : $0

CHAT RAPIDE   ──► Groq (Llama 4 Scout gratuit ultra-rapide)
                  Coût : $0

QUESTIONS     ──► Claude.ai plan gratuit (Sonnet 4.6)
                  Coût : $0

AGENT LOCAL   ──► OpenCode + Ollama (gratuit total)
                  Coût : $0
```

> [!tip] Analogie
> Pour un étudiant en 2026, Gemini Flash-Lite et Google AI Studio sont comme la bibliothèque universitaire gratuite : moins glamour, mais parfaitement suffisants pour apprendre et produire de bons travaux — et Ollama + Gemma 4 permet même d'avoir un agent local complet sans dépenser un centime.

### Le Freelance (budget contrôlé, ~$20-50/mois)

```
WORKFLOW FREELANCE (avril 2026)
================================

COMPLÉTION    ──► Continue.dev + Qwen3-Coder-Next local (gratuit)
                  Coût : $0/mois

DEV STANDARD  ──► Claude Sonnet 4.6 ($3/M input, ~$20-50/mois)
                  Pour toutes les features importantes

CODE OUVERT   ──► DeepSeek V4 ($0.30/M — 10x moins cher que Sonnet)
                  Pour projets open source uniquement

PRIVÉ         ──► Gemma 4 31B local (gratuit)
                  Jamais de code client en cloud

ESTIMATION BUDGET :
  Haiku 4.5 (questions) : ~$2/mois
  Sonnet 4.6 (dev)      : ~$25/mois
  DeepSeek V4 (open src): ~$3/mois
  ──────────────────────────────────
  TOTAL                 : ~$30/mois
```

> [!example]
> Scénario freelance type : Un développeur React freelance travaille sur une app SaaS cliente. Il utilise Gemma 4 31B local pour toute complétion inline (code client confidentiel), Claude Haiku 4.5 pour ses questions CSS/JS rapides, Claude Sonnet 4.6 pour implémenter les features complexes, et DeepSeek V4 pour ses projets open source personnels. Budget réel : $30/mois.

### Le Dev Senior / Architecte

```
WORKFLOW DEV SENIOR (avril 2026)
=================================

CONTEXTE LONG ──► Gemini 3.1 Pro (1M tk)
                   Analyser un repo entier en un appel
                   ou Llama 4 Scout local (10M tk, gratuit)

DEV QUOTIDIEN ──► Claude Sonnet 4.6
                   Qualité premium, contexte 1M
                   CLAUDE.md configuré par projet

ALGO DIFFICILE ──► o3/o4 OpenAI
                    Raisonnement mathématique/formel poussé

QUALITÉ MAX    ──► Claude Opus 4.6 ($5/M)
                   Quand Sonnet répond "je ne suis pas sûr"

AGENT         ──► Claude Code avec sub-agents
                   Pour tâches longues multi-fichiers

CONFIDENTIEL  ──► Ollama local (Gemma 4 31B ou Llama 4 Scout)
                   Pour IP sensible de l'entreprise

BUDGET ESTIMÉ : ~$80-150/mois
```

### L'Équipe Entreprise

```
WORKFLOW ÉQUIPE ENTREPRISE (avril 2026)
========================================

IDE DÉVELOPPEURS ──► Continue.dev configuré
                      → Serveur Ollama interne (vLLM + Gemma 4 31B)
                      → Azure OpenAI pour débordement

AGENTIC DEV    ──► Claude API (Sonnet 4.6)
                   Code review automatique PR
                   Génération de tests automatisée

PRIVÉ LOCAL    ──► Serveur vLLM + Gemma 4 31B ou Llama 4 Scout
                   vLLM sur GPU serveur entreprise
                   Zéro donnée hors réseau

CLOUD CONFORME ──► Azure OpenAI (GPT-5.4, résidence data UE)
                   ou AWS Bedrock (Claude Sonnet 4.6)
                   DPA signé, logs désactivés, isolation tenant

POLITIQUE ──► Classification des données stricte :
              - Niveau 1 (public) → Cloud OK
              - Niveau 2 (interne) → Azure OpenAI / AWS Bedrock
              - Niveau 3 (confidentiel) → Local uniquement
              - Niveau 4 (secret) → Air-gapped local
```

> [!info]
> Les grandes entreprises françaises soumises au RGPD et au secret des affaires doivent impérativement avoir une politique IA documentée avant tout déploiement. Azure OpenAI avec résidence des données en Europe est souvent le compromis entre qualité et conformité.

---

## 5. Optimisation des Coûts Avancée

### La Cascade de Modèles

Le pattern le plus puissant pour contrôler les coûts tout en maintenant la qualité : essayer d'abord le moins cher, escalader seulement si insuffisant.

```
CASCADE DE MODÈLES (avril 2026)
================================

        Tâche reçue
             │
             ▼
    ┌─────────────────┐
    │  Haiku 4.5      │  $1/$5 input/output par 1M tk
    │  / Flash-Lite   │  $0.25/$1.50 par 1M tk
    │  (tenter d'abord)│
    └────────┬────────┘
             │
    Résultat suffisant ? ──► OUI ──► Utiliser, économique ✓
             │
            NON
             │
             ▼
    ┌─────────────────┐
    │ Sonnet 4.6      │  $3/$15 par 1M tk
    │ / Gemini 3.1 Pro│  $2/$12 par 1M tk
    │ (escalade)      │
    └────────┬────────┘
             │
    Résultat suffisant ? ──► OUI ──► Utiliser, raisonnable ✓
             │
            NON
             │
             ▼
    ┌─────────────────┐
    │  Opus 4.6 / o3  │  $5/$25 par 1M tk
    │  (premium)      │
    └────────┬────────┘
             │
             ▼
         Résout ──► Utiliser, nécessaire ✓
```

En pratique, implémenter cette cascade dans un script ou une configuration :

```python
# Exemple de cascade programmatique avec l'API Anthropic
import anthropic

def cascade_completion(prompt: str, max_budget: float = 0.01) -> str:
    """
    Essaie les modèles du moins cher au plus cher.
    Retourne dès qu'un modèle produit une réponse confidente.
    """
    client = anthropic.Anthropic()
    
    cascade = [
        {"model": "claude-haiku-4-5", "cost_per_1m": 1.00},   # $1/M input, $5/M output
        {"model": "claude-sonnet-4-6", "cost_per_1m": 3.00},  # $3/M input, $15/M output
        {"model": "claude-opus-4-6",   "cost_per_1m": 5.00},  # $5/M input, $25/M output
    ]
    
    for tier in cascade:
        response = client.messages.create(
            model=tier["model"],
            max_tokens=2048,
            messages=[{"role": "user", "content": prompt}]
        )
        
        text = response.content[0].text
        
        # Heuristique simple : si le modèle exprime de l'incertitude,
        # escalader vers le niveau suivant
        uncertainty_markers = [
            "je ne suis pas sûr",
            "I'm not certain",
            "you might want to verify",
            "il faudrait vérifier"
        ]
        
        is_uncertain = any(m in text.lower() for m in uncertainty_markers)
        
        if not is_uncertain:
            print(f"Résolu avec {tier['model']}")
            return text
        
        print(f"{tier['model']} incertain, escalade...")
    
    return text  # Retourner la réponse Opus même si incertaine
```

### Gérer son Budget Tokens Mensuel

```
ALLOCATION BUDGET MENSUEL RECOMMANDÉE ($50/mois exemple, avril 2026)
=====================================================================

┌─────────────────────────────────────────────────────┐
│  60% → Claude Sonnet 4.6    $30/mois                │
│         Tâches dev quotidiennes                     │
│         ~10M tokens input / mois à $3/M             │
├─────────────────────────────────────────────────────┤
│  25% → Claude Haiku 4.5     $12.50/mois             │
│         Questions rapides, batch                    │
│         ~12.5M tokens input / mois à $1/M           │
├─────────────────────────────────────────────────────┤
│  15% → Claude Opus 4.6      $7.50/mois              │
│         Problèmes critiques uniquement              │
│         ~1.5M tokens input / mois à $5/M            │
└─────────────────────────────────────────────────────┘

Surveiller avec : /cost dans Claude Code
Alerte à 80% du budget mensuel → passer à Haiku 4.5
Astuce : DeepSeek V4 à $0.30/M pour les projets open source
```

### Prompt Cache et Économies

Le prompt caching d'Anthropic permet de réutiliser des parties de prompt déjà traitées, avec jusqu'à 90% de réduction de coût sur les tokens répétés.

```python
# Utiliser le cache Anthropic pour des prompts systèmes longs
import anthropic

client = anthropic.Anthropic()

# Le system prompt avec le contexte de votre projet
# est mis en cache et réutilisé entre les appels
SYSTEM_WITH_CODEBASE = """
Tu es un expert du codebase suivant :
[... 50 000 tokens de contexte projet ici ...]
"""

def ask_with_cache(question: str) -> str:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=[
            {
                "type": "text",
                "text": SYSTEM_WITH_CODEBASE,
                "cache_control": {"type": "ephemeral"}  # Cache 5 minutes
            }
        ],
        messages=[{"role": "user", "content": question}]
    )
    return response.content[0].text

# Premier appel : cache miss, coût normal
# Appels suivants dans les 5 min : -90% sur le system prompt
```

> [!info]
> Le cache expire après **5 minutes d'inactivité**. Pour maximiser les économies, maintenez une session active (conversation continue) plutôt que de lancer des appels sporadiques. En pratique, lors d'une session de développement de 2h, vous pouvez économiser 60-80% sur les tokens de contexte.

---

## 6. Stack IA par Type de Projet

### Projet Personnel Open Source

```
STACK PROJET OPEN SOURCE
=========================

┌─────────────────────────────────────────────────┐
│  IDE           VS Code                          │
│                + Continue.dev (extension)       │
│                + Ollama local (complétion)      │
│                Coût : $0                        │
├─────────────────────────────────────────────────┤
│  Agent CLI     Claude Code                      │
│                ou Aider + Ollama                │
│                Coût : $20-40/mois selon usage   │
├─────────────────────────────────────────────────┤
│  Questions     Claude.ai gratuit                │
│  complexes     pour questions ponctuelles       │
│                Coût : $0 (plan gratuit)         │
├─────────────────────────────────────────────────┤
│  CI/CD         GitHub Actions + API Haiku       │
│                pour review automatique PR       │
│                Coût : ~$5/mois                  │
└─────────────────────────────────────────────────┘
TOTAL : ~$25-45/mois
```

### Startup (SaaS)

```
STACK STARTUP SAAS
===================

┌─────────────────────────────────────────────────┐
│  IDE           Cursor Pro ($20/mois)            │
│                ou VS Code + GitHub Copilot      │
│                ($10/mois)                       │
├─────────────────────────────────────────────────┤
│  Agent CLI     Claude Code                      │
│                CLAUDE.md configuré par projet   │
│                Sub-agents pour tâches longues   │
├─────────────────────────────────────────────────┤
│  Cloud API     Claude Sonnet 4.6 → features prod │
│                Claude Haiku 4.5 → batch jobs    │
│                DeepSeek V4 → alternative éco    │
│                (open source uniquement)         │
├─────────────────────────────────────────────────┤
│  Monitoring    Tableau de bord coûts API        │
│                Alertes sur dépassement budget   │
│                Logging des modèles utilisés     │
└─────────────────────────────────────────────────┘
CONSEIL : Définir un plafond API dès le début.
Une boucle d'agent mal configurée peut consumer
$100 en quelques heures.
```

> [!warning]
> Toujours configurer des **limites de dépenses** (spending limits) sur votre compte Anthropic ou OpenAI. Les agents autonomes peuvent générer des coûts exponentiels en cas de boucle infinie ou de prompt mal conçu.

### Grande Entreprise

```
STACK GRANDE ENTREPRISE
========================

┌─────────────────────────────────────────────────┐
│  IDE           Continue.dev (tous les devs)     │
│                Configuré → serveur Ollama interne│
│                Aucune donnée hors réseau         │
├─────────────────────────────────────────────────┤
│  Agent CLI     OpenCode ou Claude Code           │
│                Configuré → serveur interne       │
│                ou Azure OpenAI pour débordement  │
├─────────────────────────────────────────────────┤
│  Cloud         Azure OpenAI (résidence UE)      │
│  (si nécessaire)  DPA signé, logs désactivés    │
│                   Isolation tenant garantie     │
├─────────────────────────────────────────────────┤
│  Infra locale  Serveur GPU interne              │
│                vLLM + Ollama enterprise         │
│                Modèles : Gemma 4 31B            │
│                         Llama 4 Scout           │
├─────────────────────────────────────────────────┤
│  Politique     Classification niveau 1-4        │
│                Formation obligatoire devs       │
│                Audit trimestriel usage IA       │
└─────────────────────────────────────────────────┘
```

---

## 7. Rester à Jour dans un Écosystème qui Change Vite

### Le Rythme de l'Innovation

```
CHRONOLOGIE DES SORTIES MAJEURES (exemple 2024-2026)
=====================================================

2024 Q1  ─── GPT-4 Turbo, Claude 3 (Haiku/Sonnet/Opus)
2024 Q2  ─── Gemini 1.5 Pro (1M contexte !), DeepSeek-V2
2024 Q3  ─── GPT-4o, Sonnet 3.5 (meilleur code)
2024 Q4  ─── o1, Gemini 2.0 Flash, Llama 3.3
2025 Q1  ─── Claude 3.7, o3, DeepSeek-V3 (choc des prix)
2025 Q2  ─── Gemini 2.5 Pro (SWE-bench record), GPT-4.5
2025 Q3  ─── Claude 4 (Haiku/Sonnet/Opus), Llama 4
2025 Q4  ─── o3/o4, Gemma 4, DeepSeek V4
2026 Q1  ─── GPT-5.4, Gemini 3.1 Pro/Flash-Lite, Claude 4.5/4.6
2026 Q2  ─── [prochaines sorties en cours...]

Fréquence : ~1 sortie majeure tous les 2-3 mois
```

### Benchmarks à Suivre Régulièrement

| Benchmark | URL | Ce qu'il mesure | Fiabilité |
|---|---|---|---|
| Chatbot Arena | lmarena.ai | Préférences humaines réelles | Très haute |
| SWE-bench | swebench.com | Résolution de vrais bugs | Haute |
| HumanEval | Papers with Code | Génération de code simple | Moyenne |
| LiveCodeBench | - | Code sur problèmes récents | Haute |
| MMLU Pro | - | Raisonnement multi-domaines | Moyenne |

> [!info]
> Chatbot Arena (LMSYS) est particulièrement fiable car basé sur des **votes humains** en aveugle (blind voting) avec des millions d'évaluations. Contrairement aux benchmarks académiques qui peuvent être "overfitted", il reflète l'expérience réelle des utilisateurs.

### Ressources de Veille Recommandées

```
VEILLE IA DÉVELOPPEUR
======================

QUOTIDIEN (5 min)
├── Hacker News → onglet "Ask HN" / recherche "LLM"
└── @simonwillison (Twitter/X) → meilleure veille IA tech

HEBDOMADAIRE (30 min)
├── r/LocalLLaMA (Reddit) → actualité modèles locaux
├── r/MachineLearning → papers et recherche
└── Lmarena.ai → vérifier les classements Arena

MENSUEL (1h)
├── SWE-bench leaderboard → performance code réelle
├── Papers with Code → nouveaux benchmarks
└── Blogs officiels : Anthropic, OpenAI, Google DeepMind

ALERTES À CONFIGURER
├── Google Alerts : "new LLM model" + "coding benchmark"
└── GitHub Stars : suivre anthropics/anthropic-sdk-python
```

---

## 8. L'Avenir du Développement Assisté par IA

### Les Tendances Lourdes (2025-2030)

```
ÉVOLUTION DU DÉVELOPPEMENT IA
==============================

2023  ─── Autocomplétion, génération de fonctions simples
          Le dev reste maître de chaque ligne

2024  ─── Agents autonomes sur tâches délimitées
          Claude Code, Cursor Composer
          Features complètes générées

2025  ─── Multi-agents, sub-agents, orchestration
          Plusieurs IA collaborent
          Le dev supervise des workflows entiers
          
2026  ─── Agents persistants long-terme
2027  ─── ? Équipes d'IA avec supervision humaine
...
2030  ─── ??? Le code ligne par ligne devient rare
```

### Ce qui Change, Ce qui Ne Change Pas

**Ce qui évolue rapidement :**
- La qualité du code généré (GPT-4 de 2023 = modèle local 14B de 2026)
- L'autonomie des agents (features complètes sans intervention)
- Les interfaces (l'éditeur de code traditionnel se transforme)
- L'intégration dans les pipelines CI/CD

**Ce qui reste critique :**
- **Comprendre le code généré** — pour maintenir, sécuriser, débugger
- **L'architecture système** — les IA excellents en implémentation, moins en vision
- **Le jugement produit** — "Que doit faire ce logiciel ?" reste humain
- **La sécurité** — valider ce que l'IA génère reste essentiel
- **La relation client** — comprendre le besoin métier

> [!warning]
> Le risque le "vibe coding" (accepter le code généré sans le comprendre) est réel. Un développeur qui ne comprend plus son propre codebase ne peut plus le maintenir, le sécuriser, ni le faire évoluer. L'IA augmente le développeur — elle ne le remplace pas encore.

### Le Développeur 2030 : Superviseur d'IA

```
RÔLE DU DÉVELOPPEUR EN ÉVOLUTION
==================================

2023                          2030 (projection)
════════════════════════════════════════════════

Écrire du code         ──►   Définir les objectifs
Débugger ligne/ligne   ──►   Valider les solutions
Chercher la doc        ──►   Évaluer la qualité IA
Faire la code review   ──►   Architecturer les agents
Implémenter features   ──►   Superviser les pipelines

Compétences valorisées :
+ Compréhension profonde (pas juste écriture)
+ Architecture et design patterns
+ Sécurité et validation de code généré
+ Prompt engineering et orchestration d'agents
+ Jugement technique sur sorties IA
```

---

## 9. Construire ta Boîte à Outils Personnelle : Plan d'Action

### Semaine 1 : Exploration (0 → démarrage)

```
□ Créer compte Anthropic (claude.ai)
  → Plan gratuit pour commencer
  → Tester Claude avec ton propre code

□ Créer compte Google AI Studio (aistudio.google.com)
  → Gemini 3.1 Flash-Lite gratuit
  → API key gratuite pour expérimenter

□ Installer Ollama
  → https://ollama.ai
  → ollama pull gemma4:e4b (modèle léger, ~4GB)
  → Tester : ollama run gemma4:e4b

□ Configurer Continue.dev dans VS Code
  → Extension Continue.dev
  → Configurer avec Ollama local
  → Tester la complétion inline

□ Objectif semaine 1 :
  Avoir un outil de complétion gratuit qui fonctionne
```

### Semaine 2 : Workflow Quotidien (démarrage → habitude)

```
□ Utiliser Claude Sonnet 4.6 pour tes tâches dev
  → 1 semaine complète de dev réel avec Claude
  → Noter ce qui fonctionne, ce qui frustre

□ Créer ton premier CLAUDE.md
  → À la racine d'un projet existant
  → Y décrire : stack, conventions, contexte
  → Tester l'impact sur la qualité des réponses

□ Configurer Continue.dev multi-modèles
  → Ollama pour complétion inline (gratuit, privé)
  → Claude/Gemini pour questions complexes (cloud)
  → Apprendre à basculer selon le contexte

□ Objectif semaine 2 :
  Avoir un workflow quotidien qui remplace les
  recherches Google pour les questions de code
```

### Semaine 3 : Optimisation (habitude → efficacité)

```
□ Analyser tes dépenses tokens
  → Dans Claude Code : /cost
  → Sur dashboard Anthropic : usage.anthropic.com
  → Identifier les tâches les plus coûteuses

□ Identifier quel modèle pour quoi
  → Pour chaque type de tâche de ta semaine :
    Haiku suffisant ? Sonnet nécessaire ? Opus requis ?
  → Documenter dans ton CLAUDE.md ou notes perso

□ Tester GPT-5.4 vs Claude Sonnet 4.6 sur tes cas réels
  → Prendre 5 tâches réelles de ton workflow
  → Les tester sur les deux modèles
  → Évaluer qualité, vitesse, coût
  → Choisir en connaissance de cause

□ Objectif semaine 3 :
  Réduire les coûts de 30% sans perte de qualité
  en routant correctement les tâches simples
```

### Semaine 4 : Maîtrise (efficacité → expertise)

```
□ Installer et configurer Claude Code
  → npm install -g @anthropic-ai/claude-code
  → Configurer CLAUDE.md avec sub-agents
  → Tester une tâche autonome multi-fichiers

□ Configurer un serveur Ollama partagé (si équipe)
  → Sur une machine dédiée du réseau local
  → OLLAMA_HOST=0.0.0.0 ollama serve
  → Configurer Continue.dev de tous les devs
  → Test de charge et sélection du bon modèle

□ Définir ta politique IA personnelle
  → Quelles données je n'envoie JAMAIS en cloud ?
  → Quel est mon budget mensuel max ?
  → Quels modèles pour quels contextes ?
  → Documenter et s'y tenir

□ Objectif semaine 4 :
  Avoir une stack IA complète, documentée,
  adaptée à ton profil et respectueuse de tes
  contraintes de confidentialité
```

---

## Carte Mentale ASCII

```
                    STRATÉGIES MULTI-MODÈLES
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    SÉLECTION           WORKFLOWS           OPTIMISATION
    MODÈLE              PAR PROFIL          COÛTS
          │                  │                  │
    ┌─────┴────┐       ┌─────┴────┐       ┌─────┴────┐
    │          │       │          │       │          │
  Tâche     Tâche   Freelance  Senior   Cascade   Cache
  Simple    Critique  Étudiant  Equipe   Modèles  Prompt
    │          │       │          │       │          │
  Haiku     Opus    Budget    vLLM    Haiku→     -90%
  Flash-L   o3/o4   Zéro      Interne Sonnet→   tokens
  Local             Gemini           Opus        répétés
                                         │
                                     CONFIDENTIEL
                                         │
                                    Ollama Local
                                    Air-gapped
                                    RGPD compliant

RÈGLE D'OR :
  Tâche simple ──► Modèle économique
  Données sensibles ──► Local toujours
  Budget maîtrisé ──► Cascade + Cache
  Rester à jour ──► Lmarena + SWE-bench
```

---

## Exercices Pratiques

### Exercice 1 : Audit de ton Workflow Actuel

**Objectif :** Cartographier tes usages actuels et identifier les optimisations possibles.

**Instructions :**
1. Pendant une journée de travail normale, noter chaque fois que tu interagis avec une IA
2. Pour chaque interaction, noter : la tâche, le modèle utilisé, si un modèle moins cher aurait suffi
3. Calculer le coût estimé de ta journée avec la matrice de prix de ce cours
4. Recalculer avec une stratégie de routing optimale
5. Comparer les deux totaux

**Livrable :** Un tableau "Avant/Après routing optimal" avec estimation d'économies mensuelles.

**Question de réflexion :** Quelle est la proportion de tes tâches qui nécessite réellement un modèle premium vs un modèle économique ?

---

### Exercice 2 : Implémenter une Cascade de Modèles

**Objectif :** Créer un script Python qui implémente automatiquement la cascade Haiku → Sonnet → Opus.

**Instructions :**

```python
# Point de départ — à compléter
import anthropic

def cascade_solve(problem: str) -> dict:
    """
    Résoudre un problème en commençant par le modèle le moins cher.
    
    Retourne : {
        "solution": str,
        "model_used": str,
        "escalations": int,
        "estimated_cost": float
    }
    """
    # TODO : Implémenter la cascade
    # TODO : Détecter l'incertitude dans la réponse
    # TODO : Escalader si nécessaire
    # TODO : Calculer le coût estimé
    pass

# Tester avec ces problèmes de difficultés croissantes :
problems = [
    "Quelle est la syntaxe d'une list comprehension Python ?",
    "Comment implémenter un algorithme A* en Python ?",
    "Explique la différence entre mutex et semaphore en théorie des systèmes concurrents"
]
```

**Défi bonus :** Ajouter une heuristique de détection d'incertitude plus sophistiquée (analyse de sentiment, mots-clés, longueur de réponse).

---

### Exercice 3 : Construire ta Stack Personnalisée

**Objectif :** Concevoir et documenter ta stack IA idéale selon ton profil réel.

**Instructions :**
1. Identifier ton profil parmi : étudiant / freelance / dev senior / équipe
2. Lister tes 5 principales tâches IA hebdomadaires
3. Pour chaque tâche, sélectionner le modèle optimal selon la matrice coût/qualité
4. Identifier les données que tu ne dois JAMAIS envoyer en cloud
5. Calculer ton budget mensuel optimal
6. Créer un fichier `ma-stack-ia.md` dans tes notes avec :
   - Ton profil et contraintes
   - La stack sélectionnée avec justifications
   - Ta politique de confidentialité personnelle
   - Ton plan de mise en oeuvre sur 4 semaines

**Question de réflexion :** Y a-t-il des cas d'usage dans ton travail actuel où tu n'utilises pas d'IA mais où tu pourrais gagner en productivité ? Quel modèle utiliserais-tu pour chacun ?

---

## Liens et Ressources

**Benchmarks officiels :**
- https://lmarena.ai — Chatbot Arena, votes humains en aveugle
- https://swebench.com — Performance sur vrais bugs GitHub
- https://paperswithcode.com — Derniers benchmarks de recherche

**Veille IA :**
- https://simonwillison.net — Blog de Simon Willison, veille IA excellente
- r/LocalLLaMA — Actualité modèles locaux et open source
- Hacker News — Onglet "newest" avec filtre "AI"

**Tarifs officiels :**
- https://anthropic.com/pricing — Tarifs Claude
- https://openai.com/pricing — Tarifs OpenAI
- https://ai.google.dev/pricing — Tarifs Gemini

---

## Wikilinks

[[01 - Panorama des IA pour Développeurs]]
[[04 - Claude et Claude Code Maitrise Avancee]]
[[09 - IA Locale avec Ollama]]
[[11 - IA Confidentialite et Entreprise]]
[[10 - Agents IA et Automatisation]]
