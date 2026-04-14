# 10 - LM Studio et Hardware Local

> [!info] Mis à jour avril 2026
> Ce document reflète l'état de l'IA locale en avril 2026, avec les nouvelles architectures MoE et les modèles de référence actuels.

> [!tip] Révolution MoE
> Les architectures Mixture of Experts (MoE) ont révolutionné les modèles locaux en 2026. Un modèle "80B" n'utilise que 3B paramètres par token — il est rapide et léger tout en ayant la richesse d'un grand modèle. Vérifiez toujours "B actifs" et pas "B total".

LM Studio est une application desktop qui démocratise l'IA locale en offrant une interface graphique complète pour télécharger, gérer et utiliser des modèles de langage directement sur sa machine. Dans cette note, on explore LM Studio, la quantisation, le hardware nécessaire, et comment choisir les bons outils pour ses besoins.

---

## Qu'est-ce que LM Studio ?

LM Studio est une application desktop (Windows, macOS, Linux) qui permet d'exécuter des modèles de langage localement, sans aucune ligne de commande. Elle propose une interface graphique intuitive pour télécharger des modèles depuis Hugging Face, les tester en chat, et exposer un serveur API compatible avec l'API OpenAI.

> [!info] Position dans l'écosystème IA local
> LM Studio et Ollama sont les deux outils dominants pour l'IA locale en 2026. Ils répondent à des besoins différents : LM Studio est orienté interface graphique et découverte, Ollama est orienté CLI, automatisation et intégration.

---

## 1. Introduction : LM Studio vs Ollama

Avant de plonger dans LM Studio, il est utile de comprendre pourquoi deux outils coexistent pour l'IA locale.

### Deux philosophies différentes

```
             OLLAMA                          LM STUDIO
         ┌───────────┐                   ┌──────────────────┐
         │   CLI /   │                   │  Interface       │
         │   API     │                   │  Graphique       │
         │  Server   │                   │  Desktop         │
         └─────┬─────┘                   └────────┬─────────┘
               │                                  │
         ┌─────▼─────┐                   ┌────────▼─────────┐
         │  Ollama   │                   │  Hugging Face    │
         │   Hub     │                   │  (tous les GGUF) │
         └─────┬─────┘                   └────────┬─────────┘
               │                                  │
         ┌─────▼─────┐                   ┌────────▼─────────┐
         │ Port 11434│                   │  Port 1234       │
         │ API OpenAI│                   │  API OpenAI      │
         │ compatible│                   │  compatible      │
         └───────────┘                   └──────────────────┘

  Cible : Devs, power users,          Cible : Débutants,
          automatisation,              exploration, test,
          scripts, CI                  interface visuelle
```

### Ollama : orienté CLI/API/serveur

- Commandes simples : `ollama run mistral`, `ollama pull llama3`
- Idéal pour les scripts Python, les automatisations
- Modelfiles pour personnaliser les modèles
- Intégration native avec Continue.dev, Open WebUI, etc.
- Pas d'interface graphique (sauf outils tiers)

### LM Studio : orienté interface graphique

- Aucune ligne de commande nécessaire
- Interface de découverte intégrée sur Hugging Face
- Test rapide de modèles via interface de chat
- Serveur API compatible OpenAI activable en un clic
- Idéal pour comparer des modèles rapidement

### Quand utiliser l'un ou l'autre ?

| Situation | Outil recommandé |
|-----------|-----------------|
| Découvrir un nouveau modèle | LM Studio |
| Automatiser avec Python/scripts | Ollama |
| Usage quotidien en IDE | Ollama + Continue.dev |
| Comparer plusieurs modèles visuellement | LM Studio |
| Déployer sur un serveur headless | Ollama (ou LocalAI) |
| Débutant qui veut tester sans terminal | LM Studio |
| Personnaliser le comportement du modèle | Ollama (Modelfiles) |

> [!tip] Analogie
> Ollama est comme `git` en ligne de commande : puissant, scriptable, préféré des développeurs. LM Studio est comme GitHub Desktop : la même puissance, mais avec une interface graphique accessible à tous. Les deux font l'IA locale, avec des forces différentes.

---

## 2. LM Studio en détail

### Installation

1. Aller sur [https://lmstudio.ai](https://lmstudio.ai)
2. Télécharger l'installeur pour son OS (Windows, macOS, Linux)
3. Lancer l'installation — aucune dépendance requise
4. Ouvrir LM Studio, l'interface se lance directement

> [!info] Aucune ligne de commande
> Contrairement à Ollama qui nécessite un terminal, LM Studio est une application desktop classique. C'est son principal avantage pour les utilisateurs non-développeurs.

### Interface et fonctionnalités

L'interface de LM Studio est organisée en onglets principaux :

```
┌─────────────────────────────────────────────────────┐
│  LM Studio                              [_ □ ×]     │
├──────────┬──────────────────────────────────────────┤
│          │                                          │
│ Discover │   ← Onglet actif                         │
│          │                                          │
│  Chat    │   Zone principale                        │
│          │                                          │
│  Local   │                                          │
│  Server  │                                          │
│          │                                          │
│  My      │                                          │
│  Models  │                                          │
│          │                                          │
│  Logs    │                                          │
│          │                                          │
└──────────┴──────────────────────────────────────────┘
```

#### Onglet Discover

L'onglet Discover est la bibliothèque intégrée de modèles. Il se connecte directement à Hugging Face pour lister les modèles GGUF disponibles.

Fonctionnalités :
- **Recherche** par nom, famille (Llama, Qwen, Mistral, etc.)
- **Filtres** : taille de modèle (3B, 7B, 14B...), type de quantisation, compatibilité
- **Téléchargement direct** : clic sur le modèle, choisir la quantisation, télécharger
- **Informations** : description, contexte max, spécifications recommandées

> [!example] Exemple de recherche dans Discover
> Rechercher "gemma4" ou "qwen3-coder" → choisir la variante adaptée à son GPU → choisir "Q4_K_M" → cliquer "Download". Le modèle se télécharge dans le dossier local de LM Studio (~/.cache/lm-studio/ sur Linux/macOS). Pour les modèles MoE, noter que la taille affichée est la taille "totale" — la RAM réellement utilisée est bien inférieure.

#### Onglet Chat

Interface de conversation similaire à ChatGPT, mais entièrement locale.

- Sélectionner le modèle chargé via le menu déroulant
- Configurer les paramètres en temps réel :
  - **Temperature** : 0.0 (déterministe) à 2.0 (créatif)
  - **Context Length** : taille de la fenêtre de contexte
  - **Top-P / Top-K** : paramètres d'échantillonnage
  - **Repeat Penalty** : éviter les répétitions
- Idéal pour tester rapidement un modèle avant de l'intégrer dans du code
- Possibilité d'ajouter un system prompt personnalisé

```
┌─────────────────────────────────────────────────┐
│ Modèle : qwen2.5-coder-14b-Q4_K_M    [▼]       │
├─────────────────────────────────────────────────┤
│                                                  │
│  [System Prompt]                                 │
│  Tu es un assistant de programmation expert...   │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │ User : Explique les décorateurs Python     │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │ Assistant : Les décorateurs Python sont... │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│  [Écrire un message...]              [Envoyer]  │
└─────────────────────────────────────────────────┘
```

#### Onglet Local Server

C'est la fonctionnalité clé pour les développeurs : LM Studio expose un serveur HTTP local compatible avec l'API OpenAI.

- **Port par défaut** : 1234
- **URL de base** : `http://localhost:1234/v1`
- **Compatibilité** : tout outil qui supporte l'API OpenAI fonctionne sans modification
- Démarrer le serveur en un clic → le modèle actuellement chargé répond aux requêtes

> [!warning] Modèle requis
> Le serveur Local Server ne fonctionne que si un modèle est chargé en mémoire. Si aucun modèle n'est chargé, les requêtes retournent une erreur. Charger le modèle depuis l'onglet "My Models" avant de démarrer le serveur.

### Configuration du serveur LM Studio en Python

```python
# Utiliser LM Studio comme backend OpenAI-compatible
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:1234/v1",
    api_key="lm-studio"  # Valeur arbitraire, LM Studio n'authentifie pas
)

response = client.chat.completions.create(
    model="lmstudio-community/qwen2.5-coder-14b-instruct-gguf",
    messages=[
        {"role": "system", "content": "Tu es un expert Python."},
        {"role": "user", "content": "Génère un algorithme de tri rapide en Python"}
    ],
    temperature=0.1,
    max_tokens=1000
)

print(response.choices[0].message.content)
```

> [!tip] Analogie
> Le serveur Local Server de LM Studio est comme un "faux OpenAI" qui tourne sur ta machine. Ton code Python ne sait pas qu'il parle à un modèle local — il envoie exactement les mêmes requêtes que pour l'API OpenAI réelle.

### Lister les modèles disponibles via API

```python
# Lister les modèles disponibles dans LM Studio
import requests

response = requests.get("http://localhost:1234/v1/models")
models = response.json()

for model in models["data"]:
    print(model["id"])
# Affiche : lmstudio-community/qwen2.5-coder-14b-instruct-gguf
```

### Streaming avec LM Studio

```python
# Streaming pour affichage progressif
from openai import OpenAI

client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")

stream = client.chat.completions.create(
    model="lmstudio-community/qwen2.5-coder-14b-instruct-gguf",
    messages=[{"role": "user", "content": "Explique les générateurs Python"}],
    stream=True,
    temperature=0.2
)

for chunk in stream:
    if chunk.choices[0].delta.content is not None:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

### LM Studio avec Continue.dev

```json
// ~/.continue/config.json
{
  "models": [
    {
      "title": "LM Studio - Qwen Coder",
      "provider": "lmstudio",
      "model": "qwen2.5-coder-14b-instruct",
      "apiBase": "http://localhost:1234/v1"
    }
  ],
  "tabAutocompleteModel": {
    "title": "LM Studio Autocomplete",
    "provider": "lmstudio",
    "model": "qwen2.5-coder-14b-instruct"
  }
}
```

---

## 3. Comprendre la quantisation

La quantisation est le concept central pour comprendre pourquoi les modèles IA peuvent tourner sur du matériel grand public.

### Le problème de taille

Un modèle de langage est constitué de milliards de paramètres (nombres). La taille dépend du format de stockage de chaque nombre :

```
Modèle 14B paramètres :

  FP32 (32 bits par param.) :  14 000 000 000 × 4 octets = 56 GB  ← inutilisable
  FP16 (16 bits par param.) :  14 000 000 000 × 2 octets = 28 GB  ← GPU pro seulement
  Q8   ( 8 bits par param.) :  14 000 000 000 × 1 octet  = 14 GB  ← GPU 16GB+
  Q4   ( 4 bits par param.) :  14 000 000 000 × 0.5 octet = 7 GB  ← GPU 8GB !
```

> [!tip] Analogie
> La quantisation, c'est comme compresser une photo. Une image RAW (FP32) est parfaite mais énorme. Un JPEG de qualité 80% (Q4_K_M) est bien plus petit, et pour la plupart des usages, la différence est imperceptible. On sacrifie une précision marginale pour une utilisabilité réelle.

### Types de quantisation

| Format | Bits | RAM pour 14B | Qualité | Recommandé pour |
|--------|------|-------------|---------|-----------------|
| FP32 | 32 | 56 GB | Parfaite | Recherche uniquement |
| FP16 / BF16 | 16 | 28 GB | Excellente | GPU pro (A100, H100) |
| Q8_0 | ~8 | 15 GB | Très bonne | GPU 24 GB+ |
| Q6_K | ~6 | 12 GB | Bonne | GPU 16-24 GB |
| Q5_K_M | ~5 | 10 GB | Bonne | GPU 12-16 GB |
| Q4_K_M | ~4 | 8 GB | Acceptable | **Recommandé CPU/GPU 8-12 GB** |
| Q3_K_M | ~3 | 6 GB | Passable | CPU limité |
| Q2_K | ~2 | 4 GB | Faible | Dernier recours |

> [!info] La notation "_K_M"
> Le suffixe `_K` signifie que la quantisation utilise des "k-quants" (groupes de quantisation plus intelligents, meilleure qualité). `_M` signifie "Medium" (équilibre qualité/taille). Il existe aussi `_S` (Small) et `_L` (Large). `Q4_K_M` est le format le plus populaire car il offre le meilleur compromis.

### Nommer les fichiers GGUF

```
qwen2.5-coder-14b-instruct-Q4_K_M.gguf
│               │           │
│               │           └── Quantisation : 4 bits, K-quants, Medium
│               └── Taille : 14 milliards de paramètres
└── Famille et version du modèle
```

### Choisir sa quantisation selon son hardware

```
                    GUIDE DE CHOIX QUANTISATION
                    
  Moins de 6 GB VRAM/RAM
         │
         └── Q3_K_M ou Q4_K_M sur un 7B (petit modèle)

  8 GB VRAM (RTX 3070, RX 6700 XT)
         │
         └── Q4_K_M sur 7B (confortable)
         └── Q4_K_M sur 14B (juste, possible)

  12 GB VRAM (RTX 3060, RTX 4070)
         │
         └── Q4_K_M sur 14B (confortable)
         └── Q5_K_M sur 14B (meilleure qualité)
         └── Q4_K_M sur 32B (non, trop grand)

  16 GB VRAM (RTX 4080)
         │
         └── Q6_K sur 14B (très bonne qualité)
         └── Q4_K_M sur 32B (possible, serré)

  24 GB VRAM (RTX 4090, RTX 3090)
         │
         └── Q8_0 sur 14B (quasi parfait)
         └── Q4_K_M sur 32B (confortable)
         └── Q5_K_M sur 32B (bonne qualité)
```

> [!warning] VRAM vs RAM
> Il y a une différence importante : la **VRAM** est la mémoire sur la carte graphique (rapide, dédiée). La **RAM** est la mémoire système (plus lente pour l'inférence). Si le modèle ne tient pas en VRAM, une partie est chargée en RAM — ce qu'on appelle le "CPU offloading", qui ralentit significativement l'inférence.

---

## 4. Comprendre le hardware pour l'IA locale

### GPU vs CPU : quelle différence concrète ?

```
         CPU (Central Processing Unit)
         ┌─────────────────────────────┐
         │  ●●●●●●●●  (8-32 cœurs)    │
         │                             │
         │  Cœurs puissants mais peu   │
         │  nombreux. Calcul           │
         │  séquentiel efficace.       │
         │                             │
         │  Pour l'IA : 1-5 tok/s      │
         │  pour un modèle 14B         │
         └─────────────────────────────┘

         GPU (Graphics Processing Unit)
         ┌─────────────────────────────┐
         │  ●●●●●●●●●●●●●●●●●●●●●●●  │
         │  ●●●●●●●●●●●●●●●●●●●●●●●  │
         │  ●●●●●●●●●●●●●●●●●●●●●●●  │
         │  (8 000 à 16 000+ CUDA cores)│
         │                             │
         │  Milliers de petits cœurs.  │
         │  Calcul parallèle massif.   │
         │                             │
         │  Pour l'IA : 15-80 tok/s   │
         │  pour un modèle 14B         │
         └─────────────────────────────┘
```

L'inférence LLM est fondamentalement une opération de multiplication matricielle massivement parallèle. Le GPU est architecturalement conçu pour ça — il peut faire des milliers d'opérations simultanément là où le CPU en fait une poignée.

### Tableau de référence hardware vs modèles

```
Modèle              VRAM GPU (Q4)    RAM CPU (Q4)    Vitesse CPU     Vitesse GPU
──────────────────────────────────────────────────────────────────────────────────
1-3B                2-3 GB           2-3 GB          5-15 tok/s      50-150 tok/s
7B                  5-6 GB           5-6 GB          2-8 tok/s       30-80 tok/s
14B                 9-10 GB          9-10 GB         1-4 tok/s       15-50 tok/s
32B                 20-22 GB         20-22 GB        < 2 tok/s       8-25 tok/s
70B                 40-45 GB         40-45 GB        < 1 tok/s       4-12 tok/s (multi-GPU)
──────────────────────────────────────────────────────────────────────────────────
[Modèles MoE (Mixture of Experts) — les règles changent !]
Gemma 4 E4B (MoE)   ~4 GB            ~4 GB           rapide          très rapide
  → 4B actifs seulement (MoE), qualité bien supérieure à sa taille apparente
Gemma 4 26B-A4B     ~10 GB           ~10 GB          modéré          rapide
  → 26B total, 4B actifs (MoE) — RAM d'un 7B, qualité d'un 26B
Gemma 4 31B (dense) ~20 GB           ~20 GB          lent            15-40 tok/s
  → Dense classique, très haute qualité
Llama 4 Scout       ~20 GB           ~20 GB          lent            15-30 tok/s
  → 109B total, 17B actifs (MoE), 10M tokens de contexte !
Qwen3-Coder-Next    ~8 GB            ~8 GB           modéré          rapide
  → 80B total, 3B actifs (MoE) — seulement 8GB RAM malgré les 80B !
──────────────────────────────────────────────────────────────────────────────────
Note : vitesses approximatives, varient selon le matériel exact
```

> [!warning] MoE change les règles de la VRAM
> Les modèles MoE (Mixture of Experts) changent les règles — Qwen3-Coder-Next a 80B paramètres mais seulement 3B actifs, ce qui le rend utilisable sur 8GB RAM. Toujours vérifier "B actifs" et pas "B total" pour estimer les besoins en mémoire.

> [!info] CPU : pas inutile
> Même si le GPU est beaucoup plus rapide, exécuter en CPU reste utile pour des tâches en arrière-plan, des petits modèles (3-7B), ou quand on n'a pas de GPU compatible. 3 tokens/seconde est lent mais fonctionnel pour de la génération de code où on lit le résultat plutôt qu'on le regarde défiler.

### Recommandations GPU grand public (avril 2026)

| GPU | VRAM | Modèles confortables (avril 2026) | Prix indicatif |
|-----|------|-----------------------------------|----------------|
| RTX 3060 | 12 GB | Gemma 4 26B-A4B (MoE), Qwen3-Coder-Next | ~300 € |
| RTX 4070 | 12 GB | Gemma 4 26B-A4B (MoE), Qwen3-Coder-Next | ~600 € |
| RTX 4070 Ti | 16 GB | Gemma 4 31B (dense), Llama 4 Scout (MoE) | ~800 € |
| RTX 4080 | 16 GB | Gemma 4 31B, Llama 4 Scout | ~1000 € |
| RTX 4090 | 24 GB | Tout (confortable) | ~2000 € |
| RTX 5090 | 32 GB | jusqu'à 32B (très confortable) | ~2500 €+ |
| M1 Mac 16 GB | 16 GB (unifiée) | jusqu'à 14B | — |
| M2 Pro 32 GB | 32 GB (unifiée) | jusqu'à 70B (possible) | — |
| M3 Max 128 GB | 128 GB (unifiée) | 70B+ fluide | — |

> [!warning] AMD et Intel Arc
> Les GPU AMD (RX 7000) et Intel Arc sont supportés via ROCm (AMD) et oneAPI (Intel), mais le support est moins mature que CUDA (NVIDIA). LM Studio et Ollama fonctionnent mieux avec des GPU NVIDIA. Sur AMD, ROCm fonctionne bien sous Linux, plus laborieux sous Windows.

### Apple Silicon : un cas particulier

Les puces Apple M1/M2/M3/M4 ont une architecture "mémoire unifiée" (Unified Memory Architecture, UMA) :

```
Architecture classique PC :
┌──────────────┐     PCIe     ┌──────────────┐
│   CPU + RAM  │ ←─────────→ │  GPU + VRAM  │
│   (64 GB)    │  (lent!)     │   (24 GB)    │
└──────────────┘              └──────────────┘
Transfert données CPU↔GPU = goulot d'étranglement

Architecture Apple Silicon :
┌────────────────────────────────────────────┐
│              Mémoire Unifiée               │
│                 (16-128 GB)                │
│  ┌──────────┐              ┌──────────┐   │
│  │   CPU    │              │   GPU    │   │
│  │  (12c)  │◄────────────►│  (38c)  │   │
│  └──────────┘   Accès      └──────────┘   │
│               direct rapide                │
└────────────────────────────────────────────┘
CPU et GPU partagent la MÊME mémoire physique
→ Pas de transfert, pas de goulot d'étranglement
→ 16 GB = 16 GB utilisables pour l'IA (GPU ET CPU)
```

C'est pourquoi un Mac M2 16 GB peut exécuter un modèle 14B plus efficacement qu'un PC avec 16 GB de RAM + GPU 8 GB VRAM séparés.

> [!tip] Analogie
> Comparer un PC classique à un Apple Silicon pour l'IA, c'est comme comparer une chaîne de production avec deux usines distinctes reliées par un camion, à une usine intégrée où tout se passe au même endroit. L'élimination du transport (transfert de données) est un gain énorme.

---

## 5. Optimiser les performances

### Paramètres clés dans LM Studio

**GPU Layers (nombre de couches en GPU)**

C'est le paramètre le plus important. Un modèle est composé de "couches" (layers). Charger plus de couches en GPU = plus rapide, mais nécessite plus de VRAM.

```
Exemple pour un modèle 14B (environ 40 couches) :

GPU Layers = 0   → Tout en CPU → 1-3 tok/s
GPU Layers = 20  → Moitié GPU, moitié CPU → 8-15 tok/s
GPU Layers = 40  → Tout en GPU → 20-50 tok/s (si VRAM suffisante)
GPU Layers = -1  → Automatique (LM Studio décide) → recommandé
```

**Context Length**

La longueur de contexte influe directement sur la VRAM utilisée. Si la VRAM est insuffisante :

```bash
# Réduire le contexte libère de la VRAM
Context 32768 → ~4 GB de VRAM supplémentaires pour le KV cache
Context 4096  → ~0.5 GB de VRAM pour le KV cache

# Règle : n'utiliser que le contexte nécessaire
```

**Flash Attention**

Algorithme optimisé pour le calcul d'attention. À activer si disponible (nécessite CUDA sur NVIDIA) :
- Réduit la VRAM utilisée pour le KV cache
- Accélère le calcul (10-30% selon le modèle)

**CPU Threads**

Pour l'inférence CPU, configurer au nombre de cœurs physiques (pas logiques) :

```
Exemple Intel Core i9-13900K :
- 24 cœurs physiques (8P + 16E)
- 32 threads logiques (hyperthreading)
→ Configurer CPU Threads = 24 (physiques), pas 32
```

### Paramètres Ollama pour référence

```bash
# Variables d'environnement Ollama
export OLLAMA_NUM_PARALLEL=2        # Requêtes parallèles simultanées
export OLLAMA_MAX_LOADED_MODELS=1   # Modèles en mémoire simultanément
export OLLAMA_GPU_OVERHEAD=1        # GB réservés pour le système GPU

# Dans un Modelfile Ollama
FROM qwen2.5-coder:14b
PARAMETER num_gpu 40        # Toutes les couches en GPU
PARAMETER num_ctx 4096      # Contexte réduit pour économiser VRAM
PARAMETER num_thread 16     # Threads CPU (si offloading)
```

> [!info] LM Studio vs Ollama : flexibilité
> Ollama via Modelfiles offre un contrôle très fin sur les paramètres d'inférence, scriptable et versionnab. LM Studio expose ces paramètres via l'interface graphique, plus accessible mais moins automatisable.

### Surveiller les performances

```python
# Script pour mesurer les performances d'un modèle local
import time
from openai import OpenAI

client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")

prompt = "Écris une fonction Python qui trie une liste par insertion."

start = time.time()
response = client.chat.completions.create(
    model="lmstudio-community/qwen2.5-coder-14b-instruct-gguf",
    messages=[{"role": "user", "content": prompt}],
    temperature=0.1
)
elapsed = time.time() - start

tokens = response.usage.completion_tokens
print(f"Tokens générés : {tokens}")
print(f"Temps total : {elapsed:.2f}s")
print(f"Vitesse : {tokens / elapsed:.1f} tokens/sec")
```

---

## 6. Autres outils d'IA locale

L'écosystème des outils d'IA locale est riche. LM Studio et Ollama ne sont pas les seules options.

### Jan.ai

```
┌─────────────────────────────────────────────┐
│  Jan.ai                                     │
│  ──────────────────────────────────────     │
│  • Alternative open source à LM Studio     │
│  • Interface graphique similaire            │
│  • Supporte GGUF, et connecteurs distants   │
│  • Communauté active, mises à jour régulières│
│  • https://jan.ai                           │
└─────────────────────────────────────────────┘
```

Avantage principal sur LM Studio : entièrement open source (LM Studio est propriétaire, gratuit mais source fermée).

### LocalAI

```bash
# Démarrer LocalAI via Docker
docker run -p 8080:8080 \
  -v $HOME/.local/share/localai:/models \
  localai/localai:latest

# Compatible avec l'API OpenAI
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-4", "messages": [{"role": "user", "content": "Hello"}]}'
```

LocalAI est idéal pour :
- Serveurs headless (sans interface graphique)
- Déploiement via Docker/Kubernetes
- Support de formats multiples : GGUF, GPTQ, ONNX, Whisper (audio), etc.
- Configuration via fichiers YAML

### GPT4All

- Interface simplifiée, focus sur la facilité d'utilisation
- Idéal pour les débutants absolus
- Modèles sélectionnés et optimisés par Nomic AI
- https://gpt4all.io
- Moins de modèles disponibles que LM Studio, mais expérience plus simple

### Open WebUI (Companion d'Ollama)

```bash
# Installer Open WebUI avec Docker (interface graphique pour Ollama)
docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main
```

Open WebUI donne à Ollama une interface graphique similaire à ChatGPT, tout en gardant la puissance d'Ollama en backend.

---

## 7. Comparaison Ollama vs LM Studio

| Critère | Ollama | LM Studio |
|---------|--------|-----------|
| Interface | CLI | GUI |
| Facilité de démarrage | Moyen | Facile |
| API locale | Oui (port 11434) | Oui (port 1234) |
| Source des modèles | Ollama Hub | Hugging Face |
| Customisation modèles | Modelfiles (avancé) | Limité (paramètres UI) |
| Automatisation/scripts | Excellent | Limité |
| Open source | Oui | Non (propriétaire) |
| Recherche de modèles | En ligne de commande | Intégrée dans l'UI |
| Suivi des performances | Via logs | Interface dédiée |
| Continue.dev | Support natif | Support via provider |
| Recommandé pour | Devs, power users | Découverte, débutants |

> [!info] Compatibilité croisée
> Les modèles GGUF téléchargés par LM Studio peuvent être utilisés par Ollama, et vice-versa. Les deux outils utilisent le même format de fichier. Un modèle téléchargé dans l'un peut être importé dans l'autre.

---

## 8. Workflow recommandé

Un workflow pratique pour intégrer l'IA locale dans sa pratique de développement :

```
                  WORKFLOW RECOMMANDÉ

  PHASE 1 : EXPLORATION (LM Studio)
  ┌──────────────────────────────────────┐
  │  1. Ouvrir LM Studio                │
  │  2. Onglet Discover                 │
  │  3. Tester plusieurs modèles        │
  │  4. Identifier le meilleur pour     │
  │     son cas d'usage                 │
  └──────────────────┬───────────────────┘
                     │
                     ▼
  PHASE 2 : INTÉGRATION (Ollama)
  ┌──────────────────────────────────────┐
  │  1. ollama pull <modele-choisi>     │
  │  2. Créer un Modelfile adapté       │
  │  3. Configurer Continue.dev         │
  │  4. Usage quotidien en IDE          │
  └──────────────────┬───────────────────┘
                     │
                     ▼
  PHASE 3 : AUTOMATISATION (Python/Scripts)
  ┌──────────────────────────────────────┐
  │  1. Scripts Python avec openai lib  │
  │  2. Agents, pipelines, RAG          │
  │  3. Intégration dans les projets    │
  └──────────────────────────────────────┘
```

### Checklist de démarrage rapide

```
□ Installer LM Studio (https://lmstudio.ai)
□ Télécharger un modèle adapté à son GPU (modèles recommandés avril 2026) :
    - GPU 8 GB   → Qwen3-Coder-Next Q4_K_M (80B total, 3B actifs — MoE !)
                   ou Gemma 4 E4B Q4_K_M (4B actifs, MoE)
    - GPU 12 GB  → Gemma 4 26B-A4B Q4_K_M (4B actifs, MoE)
    - GPU 16-24B → Gemma 4 31B Q4_K_M (dense, très haute qualité)
                   ou Llama 4 Scout Q4_K_M (17B actifs, MoE, 10M contexte)
    [Modèles ancienne génération encore fonctionnels]
    - GPU 8 GB   → Qwen2.5-Coder 7B Q4_K_M
    - GPU 12 GB  → Qwen2.5-Coder 14B Q4_K_M
□ Tester dans l'onglet Chat
□ Activer le Local Server (port 1234)
□ Faire un premier appel Python avec la lib openai
□ Configurer Continue.dev avec LM Studio
□ Migrer vers Ollama pour l'usage quotidien
```

> [!example] Premier test complet
> ```python
> # test_lm_studio.py — Premier test avec LM Studio
> from openai import OpenAI
>
> client = OpenAI(
>     base_url="http://localhost:1234/v1",
>     api_key="lm-studio"
> )
>
> # Vérifier que le serveur répond
> models = client.models.list()
> print("Modèle chargé :", models.data[0].id)
>
> # Générer du code
> response = client.chat.completions.create(
>     model=models.data[0].id,
>     messages=[
>         {"role": "system", "content": "Tu es un assistant Python expert."},
>         {"role": "user", "content": "Écris une classe Stack en Python avec push, pop, peek."}
>     ],
>     temperature=0.1
> )
>
> print(response.choices[0].message.content)
> ```

---

## 9. Cas d'usage avancés

### Utiliser LM Studio pour du RAG local

```python
# RAG (Retrieval Augmented Generation) entièrement local
from openai import OpenAI
import json

client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")

# Simuler une base documentaire
documents = [
    "La fonction sorted() en Python retourne une liste triée.",
    "Les list comprehensions sont plus rapides que les boucles for.",
    "asyncio permet la programmation asynchrone en Python."
]

def rag_query(question: str, docs: list[str]) -> str:
    # Construction du contexte (dans un vrai RAG : recherche vectorielle)
    context = "\n".join(f"- {doc}" for doc in docs)
    
    response = client.chat.completions.create(
        model="lmstudio-community/qwen2.5-coder-14b-instruct-gguf",
        messages=[
            {
                "role": "system",
                "content": f"Réponds uniquement à partir du contexte fourni.\n\nContexte:\n{context}"
            },
            {"role": "user", "content": question}
        ],
        temperature=0.0
    )
    return response.choices[0].message.content

print(rag_query("Comment trier une liste Python ?", documents))
```

### Multi-modèles : choisir dynamiquement

```python
# Stratégie : utiliser différents modèles selon la complexité
from openai import OpenAI

client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")

def smart_completion(prompt: str, complexity: str = "auto") -> str:
    """
    complexity: "fast" (petit modèle), "quality" (grand modèle), "auto"
    """
    # En pratique, LM Studio charge un seul modèle à la fois
    # Cette logique serait appliquée avec Ollama qui peut basculer
    model = client.models.list().data[0].id
    
    params = {
        "fast": {"temperature": 0.0, "max_tokens": 200},
        "quality": {"temperature": 0.1, "max_tokens": 2000},
        "auto": {"temperature": 0.1, "max_tokens": 1000}
    }
    
    return client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        **params.get(complexity, params["auto"])
    ).choices[0].message.content
```

---

## Carte Mentale ASCII

```
                    ┌────────────────────────────┐
                    │    IA LOCALE : OUTILLAGE   │
                    └──────────────┬─────────────┘
                                   │
          ┌────────────────────────┼────────────────────────┐
          │                        │                        │
   ┌──────▼──────┐         ┌───────▼──────┐        ┌───────▼──────┐
   │  LM STUDIO  │         │    OLLAMA    │        │   HARDWARE   │
   └──────┬──────┘         └───────┬──────┘        └───────┬──────┘
          │                        │                        │
    ┌─────┤               ┌────────┤                ┌───────┤
    │     │               │        │                │       │
  Discover│         CLI/API│     Modelfiles        GPU    VRAM
    │     │               │        │                │       │
  Chat    │         Port  │     Scripts           CPU    RAM
    │     │         11434 │        │                │       │
  Local   │               │     Continue        Apple  Unified
  Server  │               │     .dev           Silicon  Memory
  Port    │               │
  1234    │               │
          │               │
   ┌──────▼──────┐  ┌─────▼───────┐
   │QUANTISATION │  │ AUTRES OUTILS│
   └──────┬──────┘  └─────┬───────┘
          │               │
    ┌─────┤         ┌─────┤
    │     │         │     │
  FP32  Q4_K_M   Jan.ai  LocalAI
    │     │         │     │
  FP16  Q5_K_M  GPT4All Open
    │     │               WebUI
  Q8_0  Q3_K_M
```

---

## Exercices Pratiques

### Exercice 1 : Benchmark matériel

**Objectif :** Mesurer les performances réelles de son matériel avec différents modèles et quantisations.

1. Installer LM Studio et télécharger trois modèles :
   - Un modèle 7B en Q4_K_M
   - Un modèle 14B en Q4_K_M
   - Le même modèle 7B en Q5_K_M
2. Pour chacun, écrire un script Python qui envoie le même prompt 5 fois et mesure la vitesse en tokens/seconde
3. Créer un tableau comparatif : modèle × quantisation × vitesse × qualité perçue
4. Identifier le meilleur compromis pour son matériel

```python
# Squelette pour l'exercice 1
import time
from openai import OpenAI
from statistics import mean

client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")
PROMPT = "Explique en 3 paragraphes comment fonctionne un compilateur."

def benchmark(n_runs=5):
    speeds = []
    for i in range(n_runs):
        start = time.time()
        resp = client.chat.completions.create(
            model=client.models.list().data[0].id,
            messages=[{"role": "user", "content": PROMPT}]
        )
        elapsed = time.time() - start
        tokens = resp.usage.completion_tokens
        speeds.append(tokens / elapsed)
        print(f"Run {i+1}: {speeds[-1]:.1f} tok/s")
    print(f"Moyenne : {mean(speeds):.1f} tok/s")

benchmark()
```

### Exercice 2 : Serveur local multi-usage

**Objectif :** Construire un petit serveur Python qui utilise LM Studio pour trois tâches différentes.

1. Démarrer LM Studio avec un modèle de code (Qwen2.5-Coder ou DeepSeek-Coder)
2. Créer une API Flask/FastAPI avec trois endpoints :
   - `POST /explain` : explique un bout de code reçu en JSON
   - `POST /refactor` : suggère une refactorisation
   - `POST /test` : génère des tests unitaires pour le code reçu
3. Tester chaque endpoint avec `curl` ou un client Python
4. Mesurer la latence de chaque type de requête

```python
# Squelette pour l'exercice 2
from fastapi import FastAPI
from pydantic import BaseModel
from openai import OpenAI

app = FastAPI()
client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")

class CodeRequest(BaseModel):
    code: str
    language: str = "python"

@app.post("/explain")
async def explain_code(req: CodeRequest):
    # TODO : implémenter avec le client LM Studio
    pass

@app.post("/refactor")
async def refactor_code(req: CodeRequest):
    # TODO : implémenter avec le client LM Studio
    pass

@app.post("/test")
async def generate_tests(req: CodeRequest):
    # TODO : implémenter avec le client LM Studio
    pass
```

### Exercice 3 : Migration LM Studio → Ollama

**Objectif :** Reproduire exactement le même comportement qu'avec LM Studio, mais via Ollama, pour comprendre les deux outils en profondeur.

1. Dans LM Studio, configurer un system prompt personnalisé pour un assistant de code et noter les paramètres utilisés (temperature, context length)
2. Dans Ollama, créer un Modelfile qui reproduit exactement ce comportement :
   ```
   FROM qwen2.5-coder:14b
   SYSTEM "Ton system prompt ici"
   PARAMETER temperature 0.1
   PARAMETER num_ctx 4096
   ```
3. Envoyer le même prompt aux deux systèmes et comparer les réponses
4. Mesurer et comparer les vitesses d'inférence entre LM Studio et Ollama avec le même modèle
5. Document les différences observées (qualité, vitesse, comportement)

> [!tip] Objectif de l'exercice 3
> L'objectif n'est pas qu'un outil soit "meilleur" — les deux utilisent le même moteur d'inférence (llama.cpp). Comprendre pourquoi les résultats peuvent légèrement différer (paramètres par défaut, format de prompt template) est la vraie compétence à développer.

---

## Liens et Ressources

- Site officiel LM Studio : https://lmstudio.ai
- Jan.ai (alternative open source) : https://jan.ai
- LocalAI (serveur headless) : https://localai.io
- GPT4All : https://gpt4all.io
- Hugging Face GGUF models : https://huggingface.co/models?library=gguf

---

## Voir aussi

- [[02 - Comprendre les LLMs et les Tokens]]
- [[09 - IA Locale avec Ollama]]
- [[11 - IA Confidentialite et Entreprise]]
- [[12 - Strategies Multi-Modeles et Workflows]]
- [[08 - Continue Dev et Integration IDE]]
