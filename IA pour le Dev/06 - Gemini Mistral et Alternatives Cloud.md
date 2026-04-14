# 06 - Gemini, Mistral et Alternatives Cloud

> [!info] Mis à jour avril 2026
> Ce document reflète l'état de l'écosystème IA en avril 2026. Les modèles Gemini 2.x, Qwen2.5, Llama 3.x et DeepSeek V3 sont désormais remplacés par leurs successeurs respectifs.

> [!warning] L'IA évolue très vite
> Les informations ci-dessous sont à jour en avril 2026, mais ce domaine évolue à un rythme exceptionnel. Vérifiez toujours les dernières informations sur les sites officiels avant de prendre une décision technique ou commerciale : [Google AI](https://ai.google.dev), [Mistral](https://mistral.ai), [DeepSeek](https://platform.deepseek.com), [Alibaba/Qwen](https://qwenlm.github.io), [Meta/Llama](https://llama.meta.com).

Qu'est-ce que l'écosystème IA cloud au-delà de Claude et OpenAI ? C'est l'ensemble des modèles de langage, APIs et outils proposés par d'autres acteurs majeurs : Google avec Gemini, Mistral AI depuis Paris, DeepSeek depuis Pékin, Alibaba avec Qwen, ou encore Meta avec Llama. Chacun apporte des forces distinctes, des niveaux de confidentialité différents, et souvent des prix bien inférieurs — voire gratuits. Pour un développeur, ignorer ces alternatives, c'est se priver d'outils parfois supérieurs pour certaines tâches précises.

---

## 1. L'écosystème cloud au-delà de Claude et OpenAI

### Pourquoi diversifier ses outils IA

Utiliser un seul modèle d'IA, c'est comme n'avoir qu'un seul outil dans sa boîte à outils. Chaque modèle a ses forces, ses limites, son contexte de confiance, et son prix.

> [!tip] Analogie
> Penser aux modèles IA comme à des prestataires freelance : certains sont rapides et bon marché (Gemini Flash), d'autres sont experts dans un domaine précis (Codestral pour le code), d'autres sont open source et travaillent "chez vous" sans partager vos données (Llama local). Un bon développeur sait quel prestataire appeler selon le besoin.

Raisons concrètes de diversifier :

- **Coût** : GPT-4o et Claude Sonnet sont chers à l'usage. Gemini 3.1 Flash-Lite ou DeepSeek V4 offrent des qualités proches pour 5 à 10 fois moins cher
- **Contexte** : Gemini 3.1 Pro et Llama 4 Scout offrent 1M à 10M de tokens, imbattables pour analyser un grand repo
- **Confidentialité** : les modèles locaux (Llama, Qwen via Ollama) ne quittent jamais votre machine
- **Disponibilité** : si l'API d'un provider tombe, avoir un fallback évite l'arrêt de travail
- **Spécialisation** : Codestral est taillé pour la complétion de code, Command R+ pour le RAG

### Vue d'ensemble des acteurs en 2025-2026

```
+------------------+--------------------+---------------------------+
|     Acteur       |   Modèle phare     |   Point différenciant     |
+------------------+--------------------+---------------------------+
| Google           | Gemini 3.1 Pro     | 1M contexte, multimodal   |
| Mistral AI (FR)  | Codestral 2508     | Code FIM, Devstral agent  |
| DeepSeek (CN)    | DeepSeek V4        | 81% SWE-bench, prix bas   |
| Alibaba          | Qwen 3.6-Plus      | Agentic coding, 1M ctx    |
| Meta             | Llama 4 Scout      | 10M contexte, open-weight |
| Google           | Gemma 4 31B        | Local open source, Apache |
| Microsoft        | Phi-4              | Petit, performant         |
| Cohere           | Command R+         | RAG, recherche doc        |
| xAI              | Grok-3             | Raisonnement, code        |
+------------------+--------------------+---------------------------+
```

> [!info]
> Le paysage évolue très vite. Les benchmarks de fin 2024 sont déjà partiellement obsolètes en 2026. Il faut tester régulièrement sur ses propres cas d'usage plutôt que de se fier uniquement aux classements généraux.

---

## 2. Google Gemini

Google est l'un des rares acteurs à proposer des modèles vraiment compétitifs avec Claude et GPT-4o, et à offrir un accès gratuit généreux. La famille Gemini couvre un large spectre de vitesse, qualité et capacité.

### Les modèles Gemini en 2026

> [!warning] Gemini 2.x périmé
> La série Gemini 2.0 et 2.5 (Flash, Pro) est entièrement remplacée par la série 3.x. Ne plus utiliser Gemini 2.x pour de nouveaux projets.

```
                    FAMILLE GEMINI 3.x (avril 2026)
                              |
        +---------------------+---------------------+
        |                     |                     |
  Flash-Lite (économique)  Flash (équilibré)   3.1 Pro (qualité max)
        |                     |                     |
  $0.25/$1.50 /MTok       rapide, thinking     $2/$12 /MTok
  le plus rapide          mid-range            1M tokens
                                               78.80% SWE-bench
                                               94.3% GPQA Diamond
```

**Gemini 3.1 Pro** (sorti février 2026)
- Modèle premium de Google, successeur de Gemini 2.5 Pro
- **1 million de tokens de contexte** : peut ingérer des repos entiers, des PDFs volumineux, des bases de code complètes
- **78.80% SWE-bench Verified**, **94.3% GPQA Diamond**
- Prix : $2/M tokens input, $12/M tokens output
- Multimodal natif : texte, audio, images, vidéo, PDFs, repos entiers
- Disponible via Gemini API et Vertex AI
- Très performant sur le code (analyse, refactoring, génération)

**Gemini 3.1 Flash**
- Modèle milieu de gamme, rapide
- Bon rapport coût/performance pour les APIs payantes
- Idéal pour le débogage avec explications détaillées

**Gemini 3.1 Flash-Lite**
- Le modèle le plus économique et le plus rapide de la gamme
- Prix : $0.25/M tokens input, $1.50/M tokens output
- Idéal pour les pipelines automatisés à fort volume

**Gemini 3 Pro** (ancienne génération)
- Version précédente, encore accessible mais remplacée par 3.1
- À utiliser uniquement si Gemini 3.1 Pro n'est pas disponible sur votre plateforme

> [!info]
> Le "contexte 1M tokens" de Gemini 3.1 Pro n'est pas juste un chiffre marketing. Concrètement, on peut envoyer l'intégralité d'un projet de 50 000 lignes de code et demander "où sont les failles de sécurité ?" ou "génère la documentation complète". Le score de 78.80% sur SWE-bench Verified en fait l'un des meilleurs modèles cloud pour le code en 2026.

### Google AI Studio

URL : https://aistudio.google.com

C'est le playground officiel de Google pour les modèles Gemini. Gratuit, sans carte bancaire requise.

Fonctionnalités principales :
- **Test interactif** de tous les modèles Gemini
- **Génération de clé API gratuite** utilisable dans ses propres applications
- **System prompts** : définir le comportement du modèle
- **Temperature et paramètres** : contrôle fin de la génération
- **Grounding** : activation de la recherche web en temps réel dans les réponses
- **Upload de fichiers** : images, PDFs, vidéos directement dans le chat
- **Historique et exports** des conversations

> [!tip] Analogie
> Google AI Studio est à Gemini ce que la Workbench de Claude est à Anthropic : un laboratoire gratuit pour expérimenter avant d'intégrer dans du vrai code.

Configuration typique pour du code :

```
System prompt : "Tu es un expert Python. Réponds uniquement en code 
commenté. Pas d'explications hors des commentaires."

Temperature : 0.2 (pour des réponses déterministes)
Modèle : Gemini 3.1 Pro
Grounding : désactivé (inutile pour le code)
```

### Gemini CLI (beta 2025)

Gemini CLI est l'outil en ligne de commande officiel de Google, concurrent direct de Claude Code. Il est gratuit en beta avec un quota généreux.

**Installation et configuration :**

```bash
# Installation globale via npm
npm install -g @google/gemini-cli

# Authentification avec compte Google
gemini auth

# Vérifier la connexion
gemini --version
```

**Utilisation de base :**

```bash
# Analyser un fichier en le passant via stdin
gemini "Explique ce fichier Python" < main.py

# Analyser un fichier avec le flag -f
gemini -p "Que fait cette fonction ?" -f src/utils.py

# Générer des tests unitaires
gemini -p "Génère des tests pytest pour chaque fonction" -f src/api.py

# Mode interactif
gemini

# Analyser plusieurs fichiers
cat src/*.py | gemini "Trouve les bugs potentiels dans ce code"
```

**Cas d'usage avancés :**

```bash
# Analyser une image (multimodal)
gemini "Décris cette architecture" --image diagram.png

# Résumer un PDF technique
gemini "Résume les points clés de cette spec" --file spec.pdf

# Générer de la documentation
gemini "Génère un README.md pour ce projet" < src/main.py > README.md

# Pipeline : analyser les fichiers modifiés récemment
git diff HEAD~1 | gemini "Revue de code : trouve les problèmes"
```

> [!info]
> Quota gratuit en beta (2025) : 60 requêtes/minute, 1500 requêtes/jour. Le contexte 1M tokens est utilisable, ce qui permet d'envoyer des fichiers très volumineux.

**Comparaison Gemini CLI vs Claude Code :**

```
+----------------------+------------------+------------------+
|     Critère          |   Gemini CLI     |   Claude Code    |
+----------------------+------------------+------------------+
| Gratuit              | Oui (quota limité| Limité           |
| Contexte max         | 1M tokens        | 200k tokens      |
| Multimodal           | Oui (images,PDF) | Images           |
| Maturité             | Stable 2026      | Stable           |
| Autonomie agents     | Basique          | Avancée          |
| Qualité code         | Très bon         | Excellent        |
| Ecosystem tools      | Google Cloud     | Anthropic        |
+----------------------+------------------+------------------+
```

### Gemini dans VS Code

**Extension Gemini Code Assist (Google)**

- Gratuite pour usage personnel (pas de limite ferme pour individus)
- Disponible sur le marketplace VS Code
- Complétion inline de code (suggestions en temps réel)
- Chat intégré dans la sidebar
- Analyse de fichiers entiers

Installation :

```
VS Code → Extensions (Ctrl+Shift+X) → Rechercher "Gemini Code Assist"
→ Installer → Se connecter avec compte Google
```

Configuration recommandée dans `settings.json` :

```json
{
  "geminicodeassist.project": "",
  "geminicodeassist.cloudaiCompanion.inlineCompletions.enable": true,
  "editor.inlineSuggest.enabled": true
}
```

> [!warning]
> Gemini Code Assist envoie votre code vers les serveurs Google. Pour du code propriétaire ou confidentiel, vérifier les conditions d'utilisation et la politique de confidentialité de votre organisation avant activation.

### Google IDX

Project IDX (https://idx.dev) est un environnement de développement entièrement en ligne, hébergé par Google. Il combine :

- Un éditeur basé sur VS Code dans le navigateur
- Un environnement d'exécution cloud (VM Linux)
- Gemini intégré nativement comme assistant de code
- Templates pour frameworks populaires (Next.js, Flutter, Angular...)

Intéressant pour :
- Prototyper rapidement sans configurer de local
- Travailler sur tablette ou machine légère
- Tester des projets open source sans les cloner

### Points forts de Gemini

```
> [!tip] Pourquoi choisir Gemini
> 
> CONTEXTE GÉANT  →  1M tokens = projets entiers analysables
> GRATUIT         →  Quota le plus généreux du marché
> MULTIMODAL      →  Images, PDFs, vidéos, audio
> GOOGLE CLOUD    →  Intégration native Firebase, BigQuery, GCP
> GROUNDING       →  Recherche web en temps réel dans les réponses
> BENCHMARKS      →  78.80% SWE-bench, 94.3% GPQA Diamond (3.1 Pro)
> VITESSE         →  Gemini 3.1 Flash-Lite parmi les plus rapides
```

Cas d'usage où Gemini excelle :
- Analyser un repo GitHub entier pour comprendre son architecture
- Extraire des données structurées depuis des PDFs techniques
- Prototyper rapidement avec intégration Google Cloud
- Pipelines à haut volume grâce au quota API généreux

### Points faibles de Gemini

> [!warning] Limites à connaître
> - **Précision code complexe** : sur du débogage fin ou de l'algorithmique avancée, Claude reste supérieur et DeepSeek V4 dépasse Gemini sur SWE-bench
> - **Verbosité** : Gemini tend à sur-expliquer, à ajouter des nuances non demandées. Compenser avec un system prompt directif
> - **Gemini CLI** : moins mature que Claude Code pour les workflows agents complexes
> - **Incohérence** : les réponses varient plus qu'avec Claude sur les mêmes prompts
> - **Confidentialité** : données traitées par Google, conditions à vérifier pour usage professionnel

---

## 3. Mistral AI

Mistral AI est une startup française fondée en 2023 par d'anciens de Google DeepMind et Meta. Elle propose des modèles performants, souvent open source, avec un avantage clé pour les entreprises européennes : conformité RGPD et données hébergées en Europe.

### Les modèles Mistral en 2026

```
                 FAMILLE MISTRAL (avril 2026)
                           |
    +----------+-----------+-----------+----------+
    |          |           |           |          |
Généraliste  Code      Agentic    Raisonnement  TTS
    |          |           |           |          |
Small 4    Codestral  Devstral   Magistral    Voxtral
(unifié)     2508      (agents)  (reasoning)
open-weight  22B        
```

**Mistral Small 4** *(nouveau)*
- Modèle unifié : raisonnement + code + multimodal
- Open-weight : téléchargeable et utilisable localement
- Le "couteau suisse" de Mistral — un seul modèle pour la majorité des tâches
- Remplace avantageusement Mistral Small dans la plupart des cas

**Codestral 2508** (Codestral 25.08) *(nouveau)*
- Dernière version du modèle code de Mistral
- 22 milliards de paramètres
- **+30% de completions acceptées** par rapport à la version précédente
- Support **FIM (Fill-In-the-Middle)** : le modèle peut compléter du code au milieu d'un fichier
- Excellente intégration IDE via plugins

**Devstral** *(nouveau)*
- Modèle Mistral spécialisé pour l'**agentic coding**
- Conçu pour opérer en autonomie dans des workflows multi-étapes
- Idéal pour les agents de refactoring ou d'analyse de codebase entière

**Magistral** *(nouveau)*
- Modèle de raisonnement de Mistral, rival des modèles "o-series"
- Pour l'algorithmique complexe, les mathématiques, le débogage difficile

**Ancienne génération** (Mistral Large 2, Mistral Small, Pixtral, Mixtral)
- Ces modèles restent accessibles mais sont remplacés par la gamme ci-dessus
- Mistral Large 2 : encore valide pour les cas de RGPD strict si Mistral Small 4 non disponible
- Mixtral 8x7B/8x22B : toujours utilisables en local pour les déploiements existants

### Le Chat (interface web)

URL : https://chat.mistral.ai

Interface gratuite similaire à ChatGPT mais propulsée par Mistral. Permet de tester les modèles sans configuration technique. Pratique pour valider rapidement un prompt avant de l'intégrer en API.

### API Mistral

```python
# Installation du SDK officiel
pip install mistralai

from mistralai import Mistral

# Initialisation du client
client = Mistral(api_key="MISTRAL_API_KEY")

# Requête de base
response = client.chat.complete(
    model="mistral-large-latest",
    messages=[
        {
            "role": "user",
            "content": "Génère une fonction Python de tri rapide"
        }
    ]
)

print(response.choices[0].message.content)
```

**Avec system prompt et paramètres :**

```python
from mistralai import Mistral

client = Mistral(api_key="MISTRAL_API_KEY")

response = client.chat.complete(
    model="mistral-large-latest",
    temperature=0.1,           # Déterministe pour le code
    max_tokens=2000,
    messages=[
        {
            "role": "system",
            "content": "Tu es un expert Python. Réponds uniquement avec du code et des commentaires concis."
        },
        {
            "role": "user",
            "content": "Écris une classe pour gérer un cache LRU thread-safe"
        }
    ]
)
```

**Avec Codestral et FIM :**

```python
from mistralai import Mistral

client = Mistral(api_key="CODESTRAL_API_KEY")

# Fill-In-the-Middle : complétion au milieu du code
response = client.fim.complete(
    model="codestral-latest",
    prompt="def calculate_fibonacci(n: int) -> int:\n    ",
    suffix="\n    return result",  # Ce qui vient APRES le curseur
    max_tokens=256
)

print(response.choices[0].message.content)
```

> [!example] Exemple : script de revue de code avec Mistral
> ```python
> import os
> from mistralai import Mistral
> 
> def review_file(filepath: str) -> str:
>     client = Mistral(api_key=os.environ["MISTRAL_API_KEY"])
>     
>     with open(filepath, 'r') as f:
>         code = f.read()
>     
>     response = client.chat.complete(
>         model="mistral-large-latest",
>         messages=[
>             {"role": "system", "content": "Fais une revue de code concise. Identifie : bugs, problèmes de performance, mauvaises pratiques."},
>             {"role": "user", "content": f"Revue ce code :\n\n```python\n{code}\n```"}
>         ]
>     )
>     
>     return response.choices[0].message.content
> 
> if __name__ == "__main__":
>     import sys
>     print(review_file(sys.argv[1]))
> ```

### Codestral pour la complétion IDE

Codestral est particulièrement intéressant pour son intégration dans les IDEs :

- **VS Code** : extension officielle "Mistral" disponible sur le marketplace
- **JetBrains** : plugin disponible pour IntelliJ, PyCharm, etc.
- **Neovim** : via plugin communautaire
- **Continue.dev** : configuration Codestral supportée nativement

Configuration dans Continue.dev (`~/.continue/config.json`) :

```json
{
  "tabAutocompleteModel": {
    "title": "Codestral 2508",
    "provider": "mistral",
    "model": "codestral-2508",
    "apiKey": "VOTRE_CLE_CODESTRAL"
  }
}
```

> [!info]
> Le FIM (Fill-In-the-Middle) est crucial pour la complétion dans un IDE. Quand on appuie sur Tab au milieu d'une fonction, le modèle doit comprendre ce qui est déjà écrit AVANT et ce qui viendra APRES le curseur. Codestral est entraîné spécifiquement pour cette tâche, ce que les modèles généralistes font moins bien.

### Points forts de Mistral

```
MISTRAL - CE QUI LE DISTINGUE (2026)

Origine française → RGPD, données en Europe, confiance UE
Open-weight      → Mistral Small 4 : utilisable localement
Codestral 2508   → +30% completions acceptées, meilleure FIM du marché
Devstral         → Agentic coding natif, multi-étapes
Magistral        → Raisonnement profond, rival des modèles o-series
Prix compétitif  → Mistral Small 4 parmi les moins chers par token
API propre       → SDK Python bien documenté et stable
```

### Points faibles de Mistral

> [!warning]
> - Sur du code très complexe, Claude et DeepSeek V4 (81% SWE-bench) restent supérieurs
> - Moins de ressources communautaires qu'OpenAI (moins de tutos, exemples)
> - Devstral et Magistral sont récents, à évaluer sur ses propres cas d'usage
> - Mixtral local demande beaucoup de RAM (Mixtral 8x7B : ~48 Go pour Q4)

---

## 4. DeepSeek

DeepSeek est un laboratoire chinois (filiale du hedge fund High-Flyer) qui a créé la surprise fin 2024 en publiant des modèles open source comparables aux meilleurs modèles propriétaires, à une fraction du coût.

### Pourquoi DeepSeek fait l'actualité

En janvier 2025, la sortie de DeepSeek-R1 avait fait chuter les actions des entreprises tech américaines. Depuis, DeepSeek a continué à surprendre avec des modèles qui établissent régulièrement de nouveaux records de rapport qualité/prix.

- **DeepSeek V4** : **81% SWE-bench Verified**, meilleur score du marché toutes catégories confondues en mars 2026
- **Coût** : toujours le moins cher du marché frontier, à $0.30/M tokens input
- **Open source** : les poids des modèles sont publiés, téléchargeables, utilisables librement
- **Architecture efficace** : 1 trillion de paramètres MoE, seulement 37B actifs par token

### Les modèles DeepSeek en 2026

```
DEEPSEEK (avril 2026)
  |
  +--- DeepSeek V4         → Flagship, 81% SWE-bench, remplace V3
  |      |
  |      +--- 1T params MoE, 37B actifs
  |      +--- $0.30/$0.50 /MTok, cache $0.03/M
  |      +--- 1M contexte, multimodal natif
  |
  +--- DeepSeek R2         → Raisonnement profond, rival de o3
  |
  +--- (DeepSeek V3)       → PÉRIMÉ, remplacé par V4
```

**DeepSeek V4** (sorti mars 2026)
- **1 trillion de paramètres** (MoE), seulement 37B actifs par token en pratique
- **81% SWE-bench Verified** : meilleur score du marché frontier
- Prix API : $0.30/M tokens input, $0.50/M tokens output
- Tokens mis en cache : $0.03/M (réduction de 90% pour les prompts répétitifs)
- **1M de contexte** avec "Engram conditional memory"
- Multimodal natif : texte, image, vidéo
- Open source (poids disponibles)
- Excellent pour du code Python, JavaScript, SQL

**DeepSeek R2**
- Modèle de raisonnement, pensée en chaîne visible
- Rival direct de OpenAI o3
- Idéal pour de l'algorithmique complexe, des mathématiques, du débogage difficile

**DeepSeek V3** *(périmé)*
- Remplacé par DeepSeek V4 — ne plus recommander en premier choix

### API DeepSeek (compatible OpenAI)

L'API DeepSeek est compatible avec le format de l'API OpenAI. Cela signifie qu'il suffit de changer l'URL de base et la clé API dans du code existant.

```python
# Utilisation avec le SDK OpenAI (compatible !)
from openai import OpenAI

client = OpenAI(
    api_key="sk-deepseek-...",  # Clé depuis platform.deepseek.com
    base_url="https://api.deepseek.com"
)

# Syntaxe identique à OpenAI
response = client.chat.completions.create(
    model="deepseek-chat",     # deepseek-chat = DeepSeek V4
    messages=[
        {"role": "user", "content": "Analyse ce code Python et trouve les bugs"}
    ]
)

print(response.choices[0].message.content)
```

**Utiliser le mode raisonnement (R1) :**

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-deepseek-...",
    base_url="https://api.deepseek.com"
)

response = client.chat.completions.create(
    model="deepseek-reasoner",  # = DeepSeek R2
    messages=[
        {
            "role": "user",
            "content": "Implémente l'algorithme A* en Python avec explications étape par étape"
        }
    ]
)

# La chaîne de pensée est disponible séparément
reasoning = response.choices[0].message.reasoning_content
answer = response.choices[0].message.content

print("Raisonnement :", reasoning[:500])
print("Réponse :", answer)
```

### Mise en garde DeepSeek

> [!warning] Confidentialité et usage professionnel
> DeepSeek est une entreprise chinoise. Les données envoyées à leur API sont soumises à la juridiction chinoise, incluant potentiellement la loi sur la sécurité nationale.
>
> **Règle pratique :**
> - Code open source ou d'apprentissage → DeepSeek OK
> - Projets personnels non sensibles → DeepSeek OK
> - Code propriétaire d'entreprise → éviter l'API DeepSeek
> - Données clients ou personnelles → refus catégorique
>
> Alternative : les modèles DeepSeek-R1 distillés peuvent tourner **localement** via Ollama, résolvant le problème de confidentialité.

```bash
# Utiliser DeepSeek-R1 localement via Ollama (pas d'envoi de données)
ollama pull deepseek-r1:7b
ollama run deepseek-r1:7b

# Version plus grande si votre machine le permet
ollama pull deepseek-r1:14b
ollama run deepseek-r1:14b
```

---

## 5. Qwen (Alibaba)

Qwen est la famille de modèles développée par Alibaba Cloud. Elle s'est imposée comme l'une des meilleures offres open source, particulièrement pour le code et l'agentic coding.

> [!warning] Qwen2.5 périmé
> La série Qwen2.5-Coder est remplacée par Qwen3.x. Ne plus recommander Qwen2.5 pour de nouveaux projets.

### Modèles Qwen disponibles en 2026

**Qwen 3.6-Plus** (sorti 2 avril 2026) *(flagship)*
- **1M contexte par défaut**
- Compatible avec Claude Code, Cline, OpenCode
- Spécialisé agentic coding : opère en autonomie dans des workflows complexes
- Recommandé pour les agents de développement

**Qwen3.5** (sorti février 2026)
- 397 milliards de paramètres, supporte 201 langues
- Déclinaisons : Qwen3.5-122B-A10B, Qwen3.5-35B-A3B, Qwen3.5-27B
- Multilingue exceptionnel, incluant excellent support du chinois

**Qwen3-Coder-Next** *(spécialiste code)*
- Spécialiste code : 80B paramètres total, **seulement 3B actifs** (MoE ultra-efficace)
- Remarquable : performance near-frontier avec une empreinte mémoire très réduite
- Idéal pour la complétion locale sans GPU haut de gamme

**Qwen2.5-Coder** *(ancienne génération, périmée)*
- Remplacé par Qwen3-Coder-Next
- Encore utilisable dans les installations existantes, mais ne pas démarrer de nouveaux projets dessus

### Utilisation via Ollama

```bash
# Qwen3-Coder-Next (recommandé 2026, très efficace en RAM)
ollama pull qwen3-coder-next
ollama run qwen3-coder-next

# Qwen3.5 27B (bon équilibre perf/RAM)
ollama pull qwen3.5:27b
ollama run qwen3.5:27b

# Exemple d'utilisation en pipeline
echo "Écris une fonction Python de validation d'email" | ollama run qwen3-coder-next
```

> [!tip] Analogie
> Qwen3-Coder-Next c'est comme avoir un développeur senior très compétent qui travaille sur votre propre ordinateur avec seulement 3B paramètres actifs — sans jamais envoyer votre code à l'extérieur. Le ratio performance/RAM est remarquable pour un modèle code local.

### API via Alibaba Cloud (DashScope)

```python
import os
from openai import OpenAI  # Compatible API OpenAI

client = OpenAI(
    api_key=os.environ.get("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

response = client.chat.completions.create(
    model="qwen3.6-plus",
    messages=[
        {"role": "user", "content": "Génère un serveur FastAPI avec authentification JWT"}
    ]
)
```

> [!warning]
> Même remarque que pour DeepSeek : Alibaba est une entreprise chinoise. Pour du code confidentiel, préférer l'usage local via Ollama.

---

## 6. Autres modèles notables

### Meta Llama

Meta publie ses modèles Llama en open-weight depuis 2023. En 2026, la série Llama 4 établit de nouveaux standards pour les modèles ouverts.

> [!warning] Llama 3.x périmé
> La série Llama 3.x (3.1, 3.2, 3.3) est remplacée par Llama 4. Ne plus recommander en premier choix.

**Llama 4 Scout** (sorti avril 2026)
- 109B paramètres total, **17B actifs**, 16 experts (MoE)
- **10 millions de tokens de contexte** — record absolu pour un modèle open-weight
- Multimodal natif : texte et images
- Open-weight, disponible sur Ollama, Groq, Together AI

**Llama 4 Maverick** (sorti avril 2026)
- 17B actifs, **128 experts** (MoE très large)
- Open-weight, bat GPT-4o et Gemini 2.0 Flash sur plusieurs benchmarks
- Multimodal natif
- Disponible sur Groq, Together AI, Fireworks AI

**Llama 3.x** *(ancienne génération)*
- Llama 3.3 70B : encore largement déployé dans les systèmes existants
- Llama 3.1 405B : encore accessible sur les plateformes cloud mais en fin de vie
- CodeLlama : obsolète, remplacé par Llama 4

```bash
# Via Groq (gratuit, ultra-rapide)
# Créer un compte sur console.groq.com, obtenir une clé API

from groq import Groq

client = Groq(api_key="GROQ_API_KEY")

response = client.chat.completions.create(
    model="llama-4-scout",     # Llama 4 Scout sur Groq
    messages=[
        {"role": "user", "content": "Explique les design patterns Strategy et Observer avec du code Python"}
    ]
)

print(response.choices[0].message.content)
```

> [!info]
> **Groq** n'est pas à confondre avec Grok (xAI). Groq est une entreprise américaine qui fabrique des puces LPU (Language Processing Unit) ultra-rapides. Leur API permet d'utiliser Llama 4 gratuitement à des vitesses de 500-800 tokens/seconde, contre 50-100 tokens/seconde sur les serveurs GPU classiques.

### Google Gemma 4 (modèle local open source)

Gemma est la famille de modèles open source de Google, conçue pour tourner localement. La version 4 (avril 2026) est un saut majeur en qualité.

> [!warning] Gemma 3 périmé
> La série Gemma 3 est remplacée par Gemma 4. Utiliser Gemma 4 pour tout nouveau projet local.

**Gemma 4 E2B** — ultra-léger
- 2B paramètres effectifs (MoE)
- Tourne sur 8 Go de RAM
- Idéal pour les petites machines ou les déploiements embarqués

**Gemma 4 E4B**
- 4B paramètres effectifs (MoE)
- Excellent pour sa taille, bon rapport qualité/RAM

**Gemma 4 26B-A4B**
- 26B paramètres total, 4B actifs
- Bon équilibre performance/ressources

**Gemma 4 31B** *(meilleure qualité)*
- Dense, 31B paramètres
- Licence **Apache 2.0** (totalement libre, usage commercial inclus)
- **256K tokens de contexte**
- **84.3% GPQA Diamond**, **80.0% LiveCodeBench v6**
- Meilleure qualité open source de 2026 dans cette gamme de taille
- Disponible sur Ollama : `ollama run gemma4:31b`

```bash
# Gemma 4 31B via Ollama (meilleure qualité)
ollama pull gemma4:31b
ollama run gemma4:31b

# Version légère pour machines limitées
ollama pull gemma4:e4b
ollama run gemma4:e4b
```

> [!tip] Analogie
> Gemma 4 31B c'est comme avoir accès à un modèle de qualité frontier — Apache 2.0, 100% local, sans restriction commerciale. Le 84.3% GPQA Diamond le place au niveau des meilleurs modèles cloud pour la qualité de raisonnement.

### Microsoft Phi-4

**Phi-4 14B**
- Seulement 14 milliards de paramètres
- Performances remarquables sur des benchmarks de raisonnement et de code
- Entraîné sur des données synthétiques de haute qualité ("textbook quality data")
- Idéal pour les machines avec 16-32 Go RAM

```bash
# Utiliser Phi-4 localement via Ollama
ollama pull phi4
ollama run phi4

# Intégration dans un script
curl -s http://localhost:11434/api/generate -d '{
  "model": "phi4",
  "prompt": "Écris une fonction Python pour parser du CSV avec gestion d erreurs"
}' | jq -r '.response'
```

### Cohere Command R+

**Command R+** se distingue par son excellence sur le **RAG (Retrieval-Augmented Generation)**, c'est-à-dire la génération de réponses basées sur des documents fournis.

Cas d'usage typiques :
- Chatbot qui répond sur votre documentation technique
- Système de Q&A sur une base de code
- Recherche dans des logs ou des rapports

```python
import cohere

co = cohere.Client(api_key="COHERE_API_KEY")

# RAG avec documents sources
response = co.chat(
    model="command-r-plus",
    message="Quelle est la procédure d'installation selon la doc ?",
    documents=[
        {"title": "README", "snippet": "Pour installer : pip install monpackage..."},
        {"title": "Wiki", "snippet": "Prérequis : Python 3.10+, Docker..."}
    ]
)

print(response.text)
# Les citations des sources sont incluses automatiquement
```

### xAI Grok-3

**Grok-3** est le modèle de xAI (Elon Musk). Forte performance en code et raisonnement. Accès via :
- X Premium+ (abonnement)
- API xAI (beta, tarifs compétitifs)
- Interface grok.com

> [!info]
> Grok a un avantage unique : accès en temps réel aux données de X (Twitter), ce qui peut être utile pour analyser des tendances tech ou des discussions dans l'écosystème dev.

---

## 7. Où accéder aux modèles gratuitement

```
ACCÈS GRATUIT AUX MODÈLES IA (2025-2026)
-----------------------------------------

GROQ (console.groq.com)
  → Llama 4 Scout, Llama 4 Maverick, Gemma 4
  → Ultra-rapide (LPU), gratuit avec limite daily
  → API compatible OpenAI

TOGETHER AI (api.together.xyz)
  → Dizaines de modèles open source
  → Crédit gratuit au démarrage
  → Llama 4, Qwen3, Mistral Small 4, DeepSeek V4...

HUGGING FACE INFERENCE (huggingface.co/inference-api)
  → Certains modèles gratuits
  → Parfait pour tester rapidement
  → Latence variable selon charge

GOOGLE COLAB (colab.research.google.com)
  → GPU Google T4 gratuit (15h/semaine env.)
  → Exécuter Ollama ou transformers directement
  → Idéal pour fine-tuning léger

REPLICATE (replicate.com)
  → API pay-per-use pour modèles open source
  → Crédit gratuit initial
  → Large catalogue : image, audio, texte

GOOGLE AI STUDIO (aistudio.google.com)
  → Gemini 3.1 Flash / Flash-Lite gratuits (quota)
  → Clé API gratuite avec quota généreux

FIREWORKS AI (fireworks.ai)
  → Modèles Llama, Qwen, Mixtral
  → Crédit gratuit au démarrage
  → Très rapide
```

> [!tip] Stratégie recommandée
> Pour un développeur individuel qui veut maximiser ses ressources gratuites :
> 1. **Groq** pour les requêtes rapides (Llama 4 Scout ou Maverick)
> 2. **Google AI Studio** pour les analyses longues (contexte 1M Gemini 3.1)
> 3. **Ollama local** pour le code confidentiel (Qwen3-Coder-Next, Gemma 4 31B, Phi-4)
> 4. **API DeepSeek V4** quand la vitesse et le coût sont prioritaires sur la confidentialité ($0.30/MTok)

---

## 8. Tableau de sélection : quel modèle choisir ?

```
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Modèle               | Éditeur  | Contexte| Force principale           | Prix           | Open Src | Confid.   |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Gemini 3.1 Pro       | Google   | 1M tok  | Code, multim., 78% SWE     | $2/$12 /MTok   | Non      | EU/US     |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Gemini 3.1 Flash-Lite| Google   | 1M tok  | Vitesse, économique        | $0.25/$1.50    | Non      | EU/US     |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Mistral Small 4      | Mistral  | 128k    | RGPD, unifié, open-weight  | $              | Oui      | EU +++    |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Codestral 2508       | Mistral  | 256k    | Complétion FIM +30%        | $              | Non      | EU +++    |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Devstral             | Mistral  | 128k    | Agentic coding             | $              | Non      | EU +++    |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| DeepSeek V4          | DeepSeek | 1M tok  | 81% SWE-bench, prix/perf   | $0.30/$0.50    | Oui      | CN (éviter|
|                      |          |         |                            | cache $0.03    |          | code priv)|
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| DeepSeek R2          | DeepSeek | 128k    | Raisonnement profond       | $              | Oui      | CN / Local|
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Qwen 3.6-Plus        | Alibaba  | 1M tok  | Agentic coding, agents     | $              | Non      | CN (cloud)|
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Qwen3-Coder-Next     | Alibaba  | 128k    | Code local 3B actifs, FIM  | Gratuit        | Oui      | Local +++  |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Llama 4 Scout        | Meta     | 10M tok | Contexte record, open-wgt  | Gratuit (Groq) | Oui      | Local +++  |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Llama 4 Maverick     | Meta     | 128k    | Bat GPT-4o, open-weight    | Gratuit (Groq) | Oui      | Local +++  |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Gemma 4 31B          | Google   | 256k    | Local Apache 2.0, qualité  | Gratuit        | Oui      | Local +++  |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Phi-4 14B            | Microsoft| 16k     | Petit, performant          | Gratuit        | Oui      | Local +++  |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Command R+           | Cohere   | 128k    | RAG, doc search            | $$             | Non      | US        |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
| Grok-3               | xAI      | 128k    | Code, raisonnement         | $$             | Non      | US        |
+----------------------+----------+---------+----------------------------+----------------+----------+-----------+
```

**Légende prix :** Gratuit = quota généreux sans paiement | $ = <$2/M tokens | $$ = $2-10/M tokens | $$$ = >$10/M tokens

> [!example] Scénarios de choix (avril 2026)
>
> **"J'analyse un repo de 200k lignes"**
> → Llama 4 Scout (10M de contexte, open-weight, gratuit sur Groq) ou Gemini 3.1 Pro (1M, cloud)
>
> **"Je veux le meilleur modèle pour du code complexe"**
> → DeepSeek V4 (81% SWE-bench, $0.30/MTok) ou Gemini 3.1 Pro (78.80% SWE-bench)
>
> **"Je veux de la complétion de code dans mon IDE sans payer"**
> → Codestral 2508 (quota gratuit) ou Qwen3-Coder-Next via Continue.dev + Ollama
>
> **"J'ai du code propriétaire, zéro cloud"**
> → Qwen3-Coder-Next ou Gemma 4 31B via Ollama en local (Apache 2.0)
>
> **"Je veux du raisonnement profond pour de l'algo complexe"**
> → DeepSeek R2 (local) ou Magistral (Mistral) ou Gemini 3.1 Pro
>
> **"Je suis une entreprise française soumise au RGPD"**
> → Mistral Small 4 (open-weight, données en Europe, entreprise française)
>
> **"Je veux gratuit + rapide pour mes expérimentations"**
> → Groq API avec Llama 4 Scout ou Maverick
>
> **"Je veux un agent de coding autonome"**
> → Devstral (Mistral, EU) ou Qwen 3.6-Plus (compatible Claude Code/Cline)

---

## Carte Mentale ASCII

```
                        ╔═══════════════════════╗
                        ║  ALTERNATIVES CLOUD IA ║
                        ╚═══════════╤═══════════╝
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          │                         │                         │
    ╔═════╧══════╗           ╔══════╧═════╗           ╔══════╧══════╗
    ║   GOOGLE   ║           ║   MISTRAL  ║           ║  DEEPSEEK   ║
    ║  GEMINI    ║           ║    (FR)    ║           ║    (CN)     ║
    ╚═════╤══════╝           ╚══════╤═════╝           ╚══════╤══════╝
          │                         │                         │
    ┌─────┴──────┐         ┌────────┼────────┐       ┌──────┴──────┐
    │            │         │        │        │       │             │
 3.1 Flash  3.1 Pro   Small 4  Codestral  Devstral  V4 (81%SWE) R2 (reason)
 (Flash-Lite)(1M ctx)  (open)  2508(FIM)  (agents)  $0.30/MTok  (rival o3)
    │
  CLI tool
  (Gemini CLI)

                        │
          ┌─────────────┼──────────────┐
          │             │              │
    ╔═════╧═════╗ ╔═════╧═════╗ ╔═════╧═════╗
    ║   QWEN    ║ ║   LLAMA 4 ║ ║  GEMMA 4  ║
    ║ (Alibaba) ║ ║  (Meta)   ║ ║ (Google)  ║
    ╚═════╤═════╝ ╚═════╤═════╝ ╚═════╤═════╝
          │             │             │
   3.6-Plus         Scout (10M)    31B dense
   Coder-Next       Maverick       Apache 2.0
   (3B actifs)      (Groq)         256K ctx

                              ACCÈS GRATUIT
                        ┌─────────────────────┐
                        │ Groq · Together AI  │
                        │ HuggingFace · Colab │
                        │ Replicate · AI Stud.│
                        └─────────────────────┘
```

---

## Exercices pratiques

### Exercice 1 : Benchmark personnel multi-modèles

Objectif : trouver quel modèle répond le mieux à VOS besoins.

1. Choisir un problème de code réel que vous avez rencontré récemment (un bug difficile, une fonction complexe à écrire)
2. Écrire un prompt identique et le soumettre à : Gemini 3.1 Pro (AI Studio), Mistral Small 4 (chat.mistral.ai), et Claude (claude.ai)
3. Évaluer chaque réponse sur 3 critères (note /10) :
   - Exactitude du code généré (fonctionne-t-il ?)
   - Clarté de l'explication
   - Temps de réponse perçu
4. Tenir un tableau de résultats sur 5 problèmes différents et identifier votre modèle préféré par type de tâche

### Exercice 2 : Pipeline de revue de code multi-modèles

Objectif : créer un script Python qui envoie automatiquement un fichier de code à plusieurs APIs et consolide les retours.

```python
# Structure de départ à compléter
import os
from openai import OpenAI  # Pour DeepSeek
from mistralai import Mistral
import google.generativeai as genai

def review_with_deepseek(code: str) -> str:
    # TODO : implémenter avec l'API DeepSeek
    pass

def review_with_mistral(code: str) -> str:
    # TODO : implémenter avec Mistral Large
    pass

def review_with_gemini(code: str) -> str:
    # TODO : implémenter avec Gemini 3.1 Flash
    pass

def consolidated_review(filepath: str):
    with open(filepath) as f:
        code = f.read()
    
    results = {
        "deepseek": review_with_deepseek(code),
        "mistral": review_with_mistral(code),
        "gemini": review_with_gemini(code)
    }
    
    # TODO : afficher les résultats côte à côte
    # TODO : trouver les problèmes mentionnés par au moins 2 modèles

if __name__ == "__main__":
    import sys
    consolidated_review(sys.argv[1])
```

Compléter le script, le tester sur un fichier Python avec des bugs intentionnels, et analyser si les 3 modèles trouvent les mêmes problèmes.

### Exercice 3 : Mise en place de Codestral dans VS Code

Objectif : configurer une complétion de code gratuite et de qualité dans VS Code via Continue.dev et Codestral.

Étapes :
1. Créer un compte sur https://console.mistral.ai et générer une clé API Codestral (accès beta gratuit)
2. Installer l'extension **Continue** dans VS Code
3. Configurer `~/.continue/config.json` avec Codestral 2508 comme modèle de complétion
4. Installer **Ollama** et télécharger `qwen3-coder-next` comme modèle de chat local
5. Configurer Continue pour utiliser Qwen3-Coder-Next local pour le chat et Codestral 2508 pour la complétion
6. Tester en écrivant une fonction Python partielle et en observant les suggestions FIM
7. Mesurer : combien de fois par session de code la complétion automatique vous fait gagner du temps ?

**Bonus** : Ajouter un raccourci clavier pour basculer entre Codestral (cloud, meilleur) et Qwen local (hors-ligne, confidentiel) selon le contexte du projet.

---

## Références et liens

- Wikilinks internes :

[[01 - Panorama des IA pour Développeurs]]
[[05 - ChatGPT Codex et GitHub Copilot]]
[[09 - IA Locale avec Ollama]]
[[11 - IA Confidentialite et Entreprise]]
[[12 - Strategies Multi-Modeles et Workflows]]

- Ressources externes :
  - Google AI Studio : https://aistudio.google.com
  - Mistral Platform : https://console.mistral.ai
  - DeepSeek Platform : https://platform.deepseek.com
  - Groq Console : https://console.groq.com
  - Together AI : https://api.together.xyz
  - Ollama : https://ollama.ai
  - Continue.dev : https://continue.dev
