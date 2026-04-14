# 09 - IA Locale avec Ollama

L'IA générative accessible via des APIs cloud (OpenAI, Anthropic, Google) a transformé le développement logiciel — mais elle soulève des questions légitimes : confidentialité des données, coût récurrent, dépendance à Internet. **Ollama** répond à ces problèmes en rendant les LLMs puissants exécutables directement sur ta machine, gratuitement, sans envoyer la moindre ligne de code à l'extérieur.

---

## 1. Introduction : Pourquoi l'IA locale ?

Avant de plonger dans l'outil, il faut comprendre pourquoi exécuter un LLM en local est une option sérieuse en 2025-2026 — et pas seulement un gadget pour passionnés.

### 1.1 Confidentialité : ton code reste chez toi

Quand tu envoies du code à ChatGPT ou Claude, ce code quitte ton ordinateur. Peu importe les garanties contractuelles : il transite par des serveurs externes. Pour :

- du code propriétaire ou sous NDA,
- des projets en entreprise avec politique de sécurité stricte,
- des données sensibles (clés API dans le contexte, business logic critique),

l'IA locale est la seule option réaliste. **Aucun octet ne sort de ta machine.**

### 1.2 Coût : zéro après installation

Les APIs cloud facturent à l'usage (tokens en entrée + tokens en sortie). Une session intensive de développement peut coûter plusieurs euros par jour. Avec Ollama :

- Coût d'installation : **0€**
- Coût par requête : **0€**
- Limite de tokens par jour : **infinie**

La seule contrainte est la puissance de ta machine et ta consommation électrique.

### 1.3 Disponibilité : zéro dépendance réseau

- Travail en avion, en zone sans couverture, sur un VPN restrictif
- Pas d'interruption de service due à une panne API
- Latence réduite (pas d'aller-retour réseau)
- Fonctionne en environnement air-gap (isolé d'Internet)

### 1.4 Personnalisation : modèles sur mesure

Avec Ollama, tu peux créer des **Modelfiles** — des configurations qui définissent le comportement du modèle : prompt système, paramètres de génération, personnalité. Tu peux aussi fine-tuner des modèles sur tes propres données (via des outils complémentaires).

> [!tip] Analogie
> Utiliser une API cloud, c'est comme louer un logiciel en SaaS : tu paies à l'usage, le fournisseur gère tout, mais tes données passent par ses serveurs. Ollama, c'est comme **installer le logiciel sur ta propre machine** : investissement initial (téléchargement du modèle), mais après tu l'utilises sans limite, sans coût, sans connexion.

---

## 2. Qu'est-ce qu'Ollama ?

Ollama est un **gestionnaire de modèles LLM** open source qui simplifie radicalement l'exécution de grands modèles de langage en local. Il gère pour toi :

- Le **téléchargement** et le **stockage** des modèles (format GGUF optimisé)
- Le **chargement en mémoire** (RAM + VRAM GPU selon disponibilité)
- L'**accélération matérielle** (CUDA pour NVIDIA, Metal pour Apple Silicon, ROCm pour AMD)
- L'**exposition d'une API REST** compatible avec l'API OpenAI

### 2.1 Architecture d'Ollama

```
+------------------+        HTTP REST         +-------------------+
|   Ton application |  ──────────────────────> |  Ollama Server    |
|   (Python, JS...) |  <──────────────────────  |  :11434           |
+------------------+                           +-------------------+
                                                        |
                              +-------------------------+
                              |
              +---------------+---------------+
              |               |               |
        +----------+   +----------+   +----------+
        |  Modèle  |   |  Modèle  |   |  Modèle  |
        |  Chargé  |   |  Sur     |   |  Sur     |
        |  (RAM/   |   |  Disque  |   |  Disque  |
        |  VRAM)   |   |  (GGUF)  |   |  (GGUF)  |
        +----------+   +----------+   +----------+
```

### 2.2 Compatibilité API OpenAI

C'est la fonctionnalité clé : Ollama expose une API **compatible avec l'API OpenAI**. Cela signifie que tout code écrit pour OpenAI fonctionne avec Ollama en changeant **une seule ligne** : le `base_url`.

```
API OpenAI  :  https://api.openai.com/v1
API Ollama  :  http://localhost:11434/v1
```

Aucune réécriture de code. Juste un changement d'URL.

### 2.3 Caractéristiques techniques

| Caractéristique | Détail |
|---|---|
| Format de modèles | GGUF (quantisé) |
| Port par défaut | 11434 |
| Protocoles | HTTP REST, streaming |
| Plateformes | Windows, macOS, Linux |
| Accélération | CUDA, Metal, ROCm, CPU |
| Licence | MIT |
| Site officiel | https://ollama.com |

> [!info]
> Ollama stocke les modèles dans `~/.ollama/models` sur macOS/Linux et `%USERPROFILE%\.ollama\models` sur Windows. Un modèle 14B quantisé en Q4 occupe environ 9GB sur disque.

---

## 3. Installation d'Ollama

### 3.1 Windows

```bash
# Option 1 : Télécharger l'installeur graphique
# https://ollama.com/download/windows

# Option 2 : via winget (recommandé)
winget install Ollama.Ollama

# Ollama s'installe comme un service Windows
# Il démarre automatiquement avec la session
```

Après installation, Ollama tourne en fond avec une icône dans la barre système. L'API est disponible sur `http://localhost:11434`.

### 3.2 macOS

```bash
# Option 1 : Télécharger l'app .dmg
# https://ollama.com/download/mac

# Option 2 : via Homebrew
brew install ollama

# Sur Apple Silicon (M1/M2/M3/M4) :
# Les performances sont excellentes grâce à Metal
# La mémoire unifiée (RAM = VRAM) est un avantage majeur
```

### 3.3 Linux

```bash
# Installation en une commande (script officiel)
curl -fsSL https://ollama.com/install.sh | sh

# Le script :
# 1. Détecte ton architecture (x86_64, arm64)
# 2. Installe les drivers CUDA si NVIDIA détecté
# 3. Crée un service systemd ollama.service
# 4. Démarre le service automatiquement

# Vérifier le statut du service
sudo systemctl status ollama

# Voir les logs
sudo journalctl -u ollama -f
```

### 3.4 Vérification de l'installation

```bash
# Vérifier la version
ollama --version
# ollama version 0.x.x

# Démarrer manuellement le serveur (si pas en service)
ollama serve
# Listening on 127.0.0.1:11434 (version 0.x.x)

# Tester que l'API répond
curl http://localhost:11434
# Ollama is running
```

> [!warning]
> Sur Windows, si tu utilises `ollama serve` dans un terminal alors que le service Windows est déjà actif, tu auras une erreur de port déjà utilisé. Utilise le gestionnaire de tâches pour arrêter le service avant de lancer manuellement, ou simplement utilise l'interface système.

---

## 4. Commandes essentielles Ollama

### 4.1 Gestion des modèles

```bash
# Télécharger un modèle depuis le hub Ollama
# (catalogue complet sur https://ollama.com/library)
ollama pull qwen2.5-coder:14b

# La progression est affichée :
# pulling manifest
# pulling 9a3b5a... 100% ████████ 9.0 GB

# Lister les modèles installés localement
ollama list
# NAME                    ID              SIZE    MODIFIED
# qwen2.5-coder:14b       a1b2c3d4e5f6   9.0 GB  2 hours ago
# mistral:7b              f6e5d4c3b2a1   4.1 GB  1 day ago

# Supprimer un modèle (libère l'espace disque)
ollama rm mistral:7b

# Informations détaillées sur un modèle
ollama show qwen2.5-coder:14b
# Model details: architecture, parameters, context length...

# Voir les modèles actuellement chargés en mémoire
ollama ps
# NAME                    ID      SIZE    PROCESSOR  UNTIL
# qwen2.5-coder:14b       a1b2c3  9.0 GB  GPU 100%   4 minutes from now
```

### 4.2 Interaction directe

```bash
# Lancer une conversation interactive dans le terminal
ollama run qwen2.5-coder:14b

# Prompt unique (non-interactif, utile pour scripts)
ollama run qwen2.5-coder:14b "Explique la différence entre async et await en Python"

# Passer du contexte depuis stdin
cat mon_fichier.py | ollama run qwen2.5-coder:14b "Que fait ce code ?"

# Quitter le mode interactif
# /bye ou Ctrl+D
```

### 4.3 Commandes utiles en mode interactif

```bash
# Dans ollama run :
/? ou /help    # Aide
/show info     # Infos sur le modèle courant
/show license  # Licence du modèle
/set verbose   # Mode verbeux (tokens/s, etc.)
/bye           # Quitter
```

> [!example]
> Session de débogage rapide en ligne de commande :
>
> ```bash
> ollama run qwen2.5-coder:14b
> >>> Voici une erreur Python que je ne comprends pas :
> ... TypeError: 'NoneType' object is not subscriptable
> ... Elle apparaît à la ligne 42 de mon code.
> ... Quelles sont les causes les plus fréquentes ?
> ```

---

## 5. Les meilleurs modèles pour le code (2025-2026)

Le choix du modèle est crucial. Voici une sélection des meilleurs modèles disponibles sur Ollama pour le développement logiciel.

### 5.1 Qwen2.5-Coder (Alibaba) — RECOMMANDÉ

Le meilleur rapport qualité/taille pour le code en 2025-2026. Développé par Alibaba Cloud, spécialisé dans la génération et la compréhension de code.

```bash
ollama pull qwen2.5-coder:7b    # ~4GB  - Pour machines 8GB RAM
ollama pull qwen2.5-coder:14b   # ~9GB  - OPTIMAL (16GB RAM)
ollama pull qwen2.5-coder:32b   # ~20GB - Pour GPU puissant
```

Points forts :
- Excellent en Python, JavaScript, TypeScript, SQL, Go, Rust, C++
- Fenêtre de contexte : **128k tokens** (peut lire des fichiers entiers)
- Benchmark : surpasse GPT-3.5 sur la plupart des tâches de code
- Le 7B fonctionne confortablement sur 8GB RAM
- Le 14B est le meilleur équilibre performance/vitesse sur 16GB RAM

### 5.2 DeepSeek-Coder-V2

```bash
ollama pull deepseek-coder-v2:16b   # ~10GB
```

- Architecture Mixture of Experts (MoE) : 16B actifs sur 236B total
- Performances comparables à GPT-3.5 Turbo sur le code
- Excellente compréhension de code existant et de complétion
- Très fort en refactoring et explication de code

### 5.3 Llama 3.3 70B (Meta)

```bash
ollama pull llama3.3:70b                      # ~40GB
ollama pull llama3.3:70b-instruct-q4_K_M      # ~22GB (version quantisée)
```

- Modèle généraliste de très haute qualité
- Code solide, mais moins spécialisé que Qwen ou DeepSeek
- Excellent pour les tâches combinant code + rédaction + analyse
- Nécessite ≥48GB RAM (full precision) ou ≥24GB (quantisé Q4)

### 5.4 Mistral 7B

```bash
ollama pull mistral:7b             # ~4GB
ollama pull mistral:7b-instruct    # Version instruction-tuned
```

- Rapide et très léger — idéal pour complétion temps réel
- Bon pour questions rapides, explications simples
- Moins performant que Qwen pour génération de code complexe
- Parfait comme modèle de complétion inline (tab autocomplete)

### 5.5 Phi-4 (Microsoft)

```bash
ollama pull phi4    # ~9GB (14B paramètres)
```

- 14B paramètres avec des performances remarquables pour sa taille
- Microsoft a optimisé la qualité des données d'entraînement
- Bon en code et en raisonnement logique
- Alternative intéressante à Qwen 14B si RAM limitée

### 5.6 CodeLlama (Meta)

```bash
ollama pull codellama:7b     # ~4GB
ollama pull codellama:13b    # ~8GB
ollama pull codellama:34b    # ~19GB
```

- Fine-tuning de Llama 2 sur du code (Python, C, C++, Java, etc.)
- Plus ancien (2023) mais stable et bien supporté
- Bon support pour le code de remplissage (fill-in-the-middle)
- Dépassé par Qwen sur les benchmarks récents, mais fiable

### 5.7 Tableau de sélection par RAM disponible

```
+------------------+-----------------------------------------------+
|  RAM disponible  |  Modèle recommandé                            |
+------------------+-----------------------------------------------+
|  8 GB            |  qwen2.5-coder:7b   (ou mistral:7b)           |
|  16 GB           |  qwen2.5-coder:14b  ← OPTIMAL                 |
|  24 GB           |  qwen2.5-coder:32b  (ou deepseek-coder-v2)    |
|  32 GB           |  llama3.3:70b-q4    (ou qwen2.5-coder:32b)    |
|  64 GB+          |  llama3.3:70b (full) ou modèles 70B+ en Q8    |
+------------------+-----------------------------------------------+

Note GPU : si tu as un GPU dédié avec VRAM suffisante,
la VRAM compte plus que la RAM système.
Ex: RTX 3090 (24GB VRAM) → qwen2.5-coder:32b en Q4
```

> [!info]
> La quantisation réduit la précision des poids du modèle pour économiser de la mémoire. Q4_K_M est le meilleur compromis : -50% de RAM vs Q8 avec une perte de qualité inférieure à 2%. La version par défaut tirée par `ollama pull` est généralement Q4_K_M.

---

## 6. API REST Ollama

### 6.1 Compatibilité OpenAI Python SDK

```python
# Installation : pip install openai
from openai import OpenAI

# Un seul changement : base_url pointe vers Ollama local
client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # Valeur arbitraire, Ollama ne vérifie pas
)

response = client.chat.completions.create(
    model="qwen2.5-coder:14b",
    messages=[
        {
            "role": "system",
            "content": "Tu es un expert Python senior. Tu écris du code propre, typé, avec docstrings."
        },
        {
            "role": "user",
            "content": "Crée une fonction de tri rapide (quicksort) en Python avec types hints."
        }
    ],
    temperature=0.1,      # Bas pour du code (déterministe)
    max_tokens=2000
)

print(response.choices[0].message.content)
```

### 6.2 Streaming de la réponse

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"
)

# stream=True pour afficher les tokens au fur et à mesure
stream = client.chat.completions.create(
    model="qwen2.5-coder:14b",
    messages=[{"role": "user", "content": "Explique les decorators Python"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content is not None:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

### 6.3 API native Ollama (curl)

```bash
# Requête simple (chat)
curl http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5-coder:14b",
    "messages": [
      {"role": "user", "content": "Crée une classe Python pour une pile (stack)"}
    ],
    "stream": false
  }'

# Génération simple (completion)
curl http://localhost:11434/api/generate \
  -d '{
    "model": "qwen2.5-coder:14b",
    "prompt": "def fibonacci(n):",
    "stream": false
  }'

# Lister les modèles via API
curl http://localhost:11434/api/tags
```

### 6.4 Utilisation avec LangChain

```python
# pip install langchain langchain-community
from langchain_community.llms import Ollama
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain

# Connexion au modèle Ollama
llm = Ollama(
    model="qwen2.5-coder:14b",
    temperature=0.1,
    num_ctx=32768   # Fenêtre de contexte
)

# Template de prompt
prompt = PromptTemplate(
    input_variables=["language", "task"],
    template="Écris une fonction {language} qui {task}. Inclus des types et une docstring."
)

chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run(language="Python", task="calcule la distance de Levenshtein entre deux chaînes")
print(result)
```

> [!tip] Analogie
> L'API Ollama est comme une **prise électrique universelle** : peu importe le modèle que tu utilises localement, l'interface reste identique à celle d'OpenAI. Tu branches tes outils existants sans adaptateur.

---

## 7. Intégration avec Continue.dev (VS Code / JetBrains)

Continue.dev est l'extension IDE qui transforme Ollama en assistant de développement complet directement dans ton éditeur.

### 7.1 Installation

```bash
# Dans VS Code : Extensions > Rechercher "Continue"
# Ou via CLI :
code --install-extension Continue.continue

# L'extension crée automatiquement ~/.continue/config.json
```

### 7.2 Configuration complète

```json
// ~/.continue/config.json
{
  "models": [
    {
      "title": "Qwen Coder 14B (Local)",
      "provider": "ollama",
      "model": "qwen2.5-coder:14b",
      "apiBase": "http://localhost:11434"
    },
    {
      "title": "Llama 3.3 70B (Local)",
      "provider": "ollama",
      "model": "llama3.3:70b-instruct-q4_K_M"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen 7B (Complétion Rapide)",
    "provider": "ollama",
    "model": "qwen2.5-coder:7b"
  },
  "embeddingsProvider": {
    "provider": "ollama",
    "model": "nomic-embed-text"
  },
  "contextProviders": [
    {"name": "code"},
    {"name": "docs"},
    {"name": "diff"},
    {"name": "terminal"},
    {"name": "problems"},
    {"name": "folder"},
    {"name": "codebase"}
  ]
}
```

### 7.3 Fonctionnalités disponibles

```
Raccourcis VS Code avec Continue.dev :
+---------------------+--------------------------------------------+
|  Ctrl+L             |  Ouvrir le chat IA dans le panneau latéral |
|  Ctrl+Shift+L       |  Ajouter la sélection au contexte chat     |
|  Ctrl+I             |  Édition inline (modifier le code sélect.) |
|  Tab (autocomplete) |  Complétion de code automatique            |
+---------------------+--------------------------------------------+
```

> [!info]
> Pour les embeddings (recherche sémantique dans ton codebase), télécharge le modèle d'embedding :
> ```bash
> ollama pull nomic-embed-text
> ```
> Cela permet à Continue.dev d'indexer ton projet et de retrouver le code pertinent à injecter dans le contexte.

---

## 8. Intégration avec OpenCode et autres CLI

### 8.1 OpenCode

```bash
# Lancer OpenCode avec un modèle Ollama
opencode --model ollama/qwen2.5-coder:14b

# Configuration permanente dans ~/.config/opencode/config.json
# (voir note 08 - OpenCode)
```

### 8.2 Aider (AI pair programming CLI)

```bash
# pip install aider-chat
# Utiliser Aider avec Ollama
aider --model ollama/qwen2.5-coder:14b

# Ou avec variable d'environnement
export OLLAMA_API_BASE=http://localhost:11434
aider --model ollama/qwen2.5-coder:14b mon_fichier.py
```

### 8.3 Script Python réutilisable

```python
# ollama_helper.py - Utilitaire local simple
from openai import OpenAI
import sys

def ask_code_question(question: str, model: str = "qwen2.5-coder:14b") -> str:
    """Pose une question à un modèle Ollama local."""
    client = OpenAI(
        base_url="http://localhost:11434/v1",
        api_key="ollama"
    )

    response = client.chat.completions.create(
        model=model,
        messages=[
            {
                "role": "system",
                "content": "Tu es un expert en développement logiciel. "
                           "Réponds en français, code en anglais."
            },
            {"role": "user", "content": question}
        ],
        temperature=0.1,
        max_tokens=4000
    )
    return response.choices[0].message.content

if __name__ == "__main__":
    question = " ".join(sys.argv[1:]) if len(sys.argv) > 1 else "Bonjour"
    print(ask_code_question(question))

# Usage : python ollama_helper.py "Comment optimiser une requête SQL avec des JOINs ?"
```

---

## 9. Optimiser les performances

### 9.1 Accélération GPU

```
+------------------+---------------------------------------------+
|  Hardware        |  Accélération                               |
+------------------+---------------------------------------------+
|  NVIDIA (CUDA)   |  Automatique si drivers à jour              |
|  AMD (ROCm)      |  Linux uniquement, configuration manuelle   |
|  Apple Silicon   |  Metal automatique, performances excellentes|
|  CPU seul        |  Fonctionnel, mais lent (llama.cpp optimisé)|
+------------------+---------------------------------------------+
```

```bash
# Vérifier que le GPU est utilisé (NVIDIA)
# Dans un autre terminal pendant qu'Ollama génère :
nvidia-smi

# Chercher le processus "ollama_llama_server" dans la liste
# La colonne GPU-Util devrait être élevée (> 80%)

# Sur macOS, Activity Monitor > GPU History
# Ou : sudo powermetrics --samplers gpu_power -n 1
```

### 9.2 Variables d'environnement de configuration

```bash
# Windows (PowerShell, à mettre dans le profil ou variables système)
$env:OLLAMA_NUM_GPU = "1"           # Nombre de GPUs à utiliser
$env:OLLAMA_NUM_PARALLEL = "2"      # Requêtes parallèles simultanées
$env:OLLAMA_MAX_LOADED_MODELS = "2" # Modèles en mémoire simultanément
$env:OLLAMA_FLASH_ATTENTION = "1"   # Flash Attention (si supporté)
$env:OLLAMA_KV_CACHE_TYPE = "q8_0"  # Quantisation du cache KV

# Linux/macOS (à mettre dans ~/.bashrc ou ~/.zshrc)
export OLLAMA_NUM_GPU=1
export OLLAMA_NUM_PARALLEL=2
export OLLAMA_MAX_LOADED_MODELS=2
export OLLAMA_FLASH_ATTENTION=1
```

### 9.3 Choisir la bonne quantisation

```bash
# Les suffixes de quantisation expliqués :
# Q4_K_M : 4-bit, K-means, Medium  → RECOMMANDÉ (défaut)
# Q5_K_M : 5-bit, K-means, Medium  → Légèrement meilleur, +25% RAM
# Q8_0   : 8-bit                   → Qualité proche du plein, +100% RAM
# F16    : 16-bit float (plein)     → Maximum qualité, très lourd

# Exemples pour qwen2.5-coder:14b :
ollama pull qwen2.5-coder:14b           # Par défaut (~Q4_K_M, ~9GB)
ollama pull qwen2.5-coder:14b-q8_0     # Haute qualité (~14GB)
ollama pull qwen2.5-coder:14b-q4_K_M   # Équilibre optimal (~9GB)

# Règle pratique :
# Si tu as la RAM → Q8_0 pour la meilleure qualité
# Si RAM limitée  → Q4_K_M (défaut, excellent équilibre)
```

> [!warning]
> Ne pas confondre quantisation du modèle et quantisation du cache KV. Le premier réduit la taille du modèle sur disque et en mémoire. Le second (`OLLAMA_KV_CACHE_TYPE`) réduit la mémoire utilisée pour le contexte pendant l'inférence. Les deux sont indépendants.

### 9.4 Optimiser la fenêtre de contexte

```bash
# La fenêtre de contexte par défaut d'Ollama est souvent 2048 tokens
# Pour les modèles qui supportent plus (ex: Qwen = 128k), il faut l'augmenter

# Via API (paramètre num_ctx)
curl http://localhost:11434/api/chat \
  -d '{
    "model": "qwen2.5-coder:14b",
    "options": {
      "num_ctx": 32768
    },
    "messages": [...]
  }'

# Via Modelfile (voir section 10)
# PARAMETER num_ctx 32768
```

---

## 10. Créer un Modelfile personnalisé

Un Modelfile est une configuration qui définit un modèle personnalisé basé sur un modèle existant. C'est l'équivalent d'un Dockerfile pour les LLMs.

### 10.1 Syntaxe du Modelfile

```dockerfile
# Modelfile - Exemple complet

# Modèle de base
FROM qwen2.5-coder:14b

# Prompt système permanent (injecté à chaque conversation)
SYSTEM """
Tu es un expert Python senior spécialisé dans FastAPI, SQLAlchemy, et Pydantic.
Tu travailles sur des APIs REST de production.

Tes règles :
- Toujours utiliser les type hints Python 3.12+
- Toujours écrire des docstrings au format Google
- Code en anglais, commentaires et explications en français
- Gérer les exceptions explicitement
- Suivre PEP 8 et les bonnes pratiques de la PEP 20 (Zen of Python)
"""

# Paramètres de génération
PARAMETER temperature 0.1       # Bas = déterministe (bon pour le code)
PARAMETER top_p 0.9
PARAMETER top_k 40
PARAMETER num_ctx 32768         # Fenêtre de contexte 32k tokens
PARAMETER num_predict 4096      # Longueur max de réponse
PARAMETER repeat_penalty 1.1   # Évite les répétitions
```

### 10.2 Créer et utiliser le modèle

```bash
# Créer le fichier Modelfile
cat > Modelfile << 'EOF'
FROM qwen2.5-coder:14b

SYSTEM """
Tu es un expert Python spécialisé dans FastAPI et SQLAlchemy.
Tu réponds en français, code en anglais, avec types hints et docstrings.
"""

PARAMETER temperature 0.1
PARAMETER num_ctx 32768
EOF

# Construire le modèle personnalisé
ollama create mon-expert-python -f Modelfile

# Vérifier qu'il apparaît dans la liste
ollama list

# L'utiliser
ollama run mon-expert-python
# Ou via API :
# model: "mon-expert-python"
```

### 10.3 Exemples de Modelfiles spécialisés

```dockerfile
# Modelfile pour expert SQL
FROM qwen2.5-coder:14b

SYSTEM """
Tu es un DBA (Database Administrator) expert en SQL et optimisation de requêtes.
Tu travailles principalement avec PostgreSQL, mais tu connais MySQL et SQLite.
Tu analyses les query plans et proposes des index appropriés.
"""

PARAMETER temperature 0.05
PARAMETER num_ctx 16384
```

```dockerfile
# Modelfile pour reviewer de code
FROM qwen2.5-coder:14b

SYSTEM """
Tu es un senior code reviewer exigeant mais bienveillant.
Pour chaque code que tu reçois, tu analyses :
1. Bugs potentiels et edge cases
2. Performance et complexité algorithmique
3. Sécurité (injections, XSS, authentification...)
4. Lisibilité et maintenabilité
5. Respect des conventions du langage

Tu commences toujours par identifier les points positifs.
"""

PARAMETER temperature 0.3
PARAMETER num_ctx 32768
```

---

## 11. Ollama comme serveur pour toute l'équipe

### 11.1 Exposer Ollama sur le réseau local

Par défaut, Ollama n'écoute que sur `localhost`. Pour partager le serveur avec ton équipe :

```bash
# Linux/macOS : démarrer avec écoute sur toutes les interfaces
OLLAMA_HOST=0.0.0.0:11434 ollama serve

# Windows : via variables d'environnement système
# Puis redémarrer le service Ollama

# Vérifier l'adresse IP de ta machine
ip addr show  # Linux
ipconfig       # Windows
ifconfig       # macOS
```

### 11.2 Configuration des clients distants

```python
# Sur les machines clientes, pointer vers le serveur Ollama distant
client = OpenAI(
    base_url="http://192.168.1.100:11434/v1",  # IP du serveur Ollama
    api_key="ollama"
)
```

```json
// Dans Continue.dev config.json (sur les machines clientes)
{
  "models": [
    {
      "title": "Serveur Équipe (Qwen 14B)",
      "provider": "ollama",
      "model": "qwen2.5-coder:14b",
      "apiBase": "http://192.168.1.100:11434"
    }
  ]
}
```

### 11.3 Considérations de sécurité

> [!warning]
> Exposer Ollama sur le réseau sans authentification est risqué sur un réseau non sécurisé. Ollama ne fournit pas nativement d'authentification. Options :
> - Utiliser un reverse proxy (nginx) avec authentification basique
> - Limiter l'accès par règles firewall (IP autorisées uniquement)
> - Utiliser un VPN pour l'accès distant
> - Ne l'exposer qu'en réseau local (LAN) de confiance

```nginx
# Exemple nginx reverse proxy avec auth basique
server {
    listen 443 ssl;
    server_name ollama.monequipe.local;

    auth_basic "Ollama - Authentification requise";
    auth_basic_user_file /etc/nginx/.htpasswd;

    location / {
        proxy_pass http://localhost:11434;
        proxy_set_header Host $host;
    }
}
```

---

## 12. Flux de travail pratiques

### 12.1 Débogage assisté par IA locale

```python
# debug_assistant.py
import traceback
from openai import OpenAI

client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

def debug_with_ai(code: str, error: str) -> str:
    """Analyse un bug avec l'IA locale."""
    prompt = f"""Voici du code Python qui génère une erreur :

```python
{code}
```

Erreur obtenue :
```
{error}
```

Identifie la cause du bug et propose une correction."""

    response = client.chat.completions.create(
        model="qwen2.5-coder:14b",
        messages=[
            {"role": "system", "content": "Expert Python, analyse les bugs avec précision."},
            {"role": "user", "content": prompt}
        ],
        temperature=0.1,
        max_tokens=2000
    )
    return response.choices[0].message.content

# Exemple d'utilisation avec capture automatique des erreurs
try:
    # Code susceptible de planter
    result = some_function()
except Exception:
    error_trace = traceback.format_exc()
    # Lire le code source du fichier courant
    with open(__file__) as f:
        source_code = f.read()
    suggestion = debug_with_ai(source_code, error_trace)
    print("Suggestion IA :", suggestion)
```

### 12.2 Génération de tests unitaires

```bash
# Commande shell pour générer des tests pour un fichier
cat mon_module.py | ollama run qwen2.5-coder:14b \
  "Génère des tests unitaires pytest complets pour ce code. 
   Inclus des cas nominaux, cas limites, et cas d'erreur."
```

---

## Carte Mentale ASCII

```
                         OLLAMA
                           |
          +----------------+----------------+
          |                |                |
     INSTALLATION       MODÈLES           USAGE
          |                |                |
    +-----+-----+    +-----+-----+    +-----+-----+
    |     |     |    |     |     |    |     |     |
  WIN   MAC  LINUX  CODE  GEN  TOOL  CLI  API  IDE
    |           |    |     |         |    |     |
  winget  brew  |  Qwen  Llama     ollama OpenAI Continue
          curl  |  14b   70b       run   compat  .dev
                |    |     |             |
              systemd DeepSeek  Mistral  /v1/chat
                      Coder    7b       /complete
                      V2       |
                               |
                          INTÉGRATION
                               |
                  +------------+------------+
                  |            |            |
              Continue    OpenCode       Aider
              .dev IDE    CLI            CLI
                  |
            +-----+-----+
            |     |     |
           Chat  Auto  Embed
           IA    Comp  dings
                 Tab   (nomic)

         OPTIMISATION
              |
    +---------+---------+
    |         |         |
   GPU    QUANT     CONTEXT
    |         |         |
  CUDA    Q4_K_M    num_ctx
  Metal   Q8_0      128k max
  ROCm    (RAM vs   (Qwen)
          qualité)

         MODELFILE
              |
    +---------+---------+
    |         |         |
  FROM    SYSTEM    PARAMETER
    |         |         |
  base    prompt    temp
  model   sys       num_ctx
          custom    top_p
```

---

## Exercices Pratiques

### Exercice 1 — Première installation et test

**Objectif** : Installer Ollama, télécharger un modèle léger, et faire une première requête via l'API Python.

1. Installer Ollama sur ta machine (via winget, brew, ou le script Linux)
2. Télécharger `qwen2.5-coder:7b` (ou `mistral:7b` si RAM < 8GB)
3. Vérifier que l'API répond : `curl http://localhost:11434`
4. Écrire un script Python qui utilise `openai.OpenAI` avec `base_url="http://localhost:11434/v1"` pour poser une question simple sur Python
5. Mesurer le temps de réponse (utilise `time.time()` avant et après) et comparer avec une API cloud

**Critère de succès** : Tu obtiens une réponse cohérente, entièrement générée localement.

---

### Exercice 2 — Créer un Modelfile spécialisé

**Objectif** : Créer un modèle Ollama personnalisé adapté à ton contexte de développement.

1. Identifie ton contexte principal : quel langage, quel framework, quelles contraintes ?
2. Rédige un `Modelfile` avec :
   - `FROM qwen2.5-coder:14b` (ou le modèle que tu as)
   - Un `SYSTEM` prompt qui décrit ton rôle d'expert idéal
   - Les paramètres : `temperature 0.1`, `num_ctx 32768`
3. Crée le modèle avec `ollama create mon-modele -f Modelfile`
4. Compare les réponses entre le modèle de base et ton modèle personnalisé sur 3 questions de ton domaine
5. Itère sur le prompt système pour affiner le comportement

**Critère de succès** : Ton modèle personnalisé répond de manière plus adaptée à tes besoins que le modèle de base.

---

### Exercice 3 — Remplacer une API cloud par Ollama

**Objectif** : Migrer un code existant utilisant l'API OpenAI vers Ollama local, sans réécriture majeure.

1. Prends un script Python existant qui utilise `openai.OpenAI()` avec l'API officielle
2. Identifie la ligne de création du client
3. Change uniquement `base_url` et `api_key` pour pointer vers Ollama
4. Adapte le nom du modèle (ex: `gpt-4` → `qwen2.5-coder:14b`)
5. Teste que le comportement est équivalent
6. Mesure la différence de latence et de qualité des réponses

**Bonus** : Paramétrise le script pour basculer entre cloud et local via une variable d'environnement `USE_LOCAL_AI=true/false`.

**Critère de succès** : Le même code fonctionne avec OpenAI ET Ollama en changeant uniquement la configuration.

---

## Liens

- [[02 - Comprendre les LLMs et les Tokens]]
- [[07 - Integrations IDE et Extensions]]
- [[08 - OpenCode et CLI Alternatifs]]
- [[10 - LM Studio et Hardware Local]]
- [[11 - IA Confidentialite et Entreprise]]
