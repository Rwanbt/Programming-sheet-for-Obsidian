# 06 - Gemini, Mistral et Alternatives Cloud

Qu'est-ce que l'écosystème IA cloud au-delà de Claude et OpenAI ? C'est l'ensemble des modèles de langage, APIs et outils proposés par d'autres acteurs majeurs : Google avec Gemini, Mistral AI depuis Paris, DeepSeek depuis Pékin, Alibaba avec Qwen, ou encore Meta avec Llama. Chacun apporte des forces distinctes, des niveaux de confidentialité différents, et souvent des prix bien inférieurs — voire gratuits. Pour un développeur, ignorer ces alternatives, c'est se priver d'outils parfois supérieurs pour certaines tâches précises.

---

## 1. L'écosystème cloud au-delà de Claude et OpenAI

### Pourquoi diversifier ses outils IA

Utiliser un seul modèle d'IA, c'est comme n'avoir qu'un seul outil dans sa boîte à outils. Chaque modèle a ses forces, ses limites, son contexte de confiance, et son prix.

> [!tip] Analogie
> Penser aux modèles IA comme à des prestataires freelance : certains sont rapides et bon marché (Gemini Flash), d'autres sont experts dans un domaine précis (Codestral pour le code), d'autres sont open source et travaillent "chez vous" sans partager vos données (Llama local). Un bon développeur sait quel prestataire appeler selon le besoin.

Raisons concrètes de diversifier :

- **Coût** : GPT-4o et Claude Opus sont chers à l'usage. Gemini 2.0 Flash ou DeepSeek-V3 offrent des qualités proches pour 5 à 10 fois moins cher
- **Contexte** : Gemini 2.5 Pro offre 2 millions de tokens, imbattable pour analyser un grand repo
- **Confidentialité** : les modèles locaux (Llama, Qwen via Ollama) ne quittent jamais votre machine
- **Disponibilité** : si l'API d'un provider tombe, avoir un fallback évite l'arrêt de travail
- **Spécialisation** : Codestral est taillé pour la complétion de code, Command R+ pour le RAG

### Vue d'ensemble des acteurs en 2025-2026

```
+------------------+------------------+------------------------+
|     Acteur       |   Modèle phare   |   Point différenciant  |
+------------------+------------------+------------------------+
| Google           | Gemini 2.5 Pro   | 2M contexte, gratuit   |
| Mistral AI (FR)  | Codestral        | Code, open source, EU  |
| DeepSeek (CN)    | DeepSeek-R1      | Open source, prix bas  |
| Alibaba          | Qwen2.5-Coder    | Code, local, open src  |
| Meta             | Llama 3.3 70B    | Open source, local     |
| Microsoft        | Phi-4            | Petit, performant      |
| Cohere           | Command R+       | RAG, recherche doc     |
| xAI              | Grok-3           | Raisonnement, code     |
+------------------+------------------+------------------------+
```

> [!info]
> Le paysage évolue très vite. Les benchmarks de fin 2024 sont déjà partiellement obsolètes en 2026. Il faut tester régulièrement sur ses propres cas d'usage plutôt que de se fier uniquement aux classements généraux.

---

## 2. Google Gemini

Google est l'un des rares acteurs à proposer des modèles vraiment compétitifs avec Claude et GPT-4o, et à offrir un accès gratuit généreux. La famille Gemini couvre un large spectre de vitesse, qualité et capacité.

### Les modèles Gemini en 2025-2026

```
                    FAMILLE GEMINI
                         |
        +----------------+----------------+
        |                |                |
   Flash (rapide)   Flash + Thinking   Pro (qualité max)
        |                |                |
  2.0 Flash         2.5 Flash         2.5 Pro
  1M tok/min        thinking mode     2M tokens
  gratuit API       équilibré         code/analyse
```

**Gemini 2.0 Flash**
- Ultra-rapide, latence très faible
- Multimodal : texte, images, audio, vidéo
- Accès API gratuit avec quota généreux (1M tokens/minute avec clé gratuite)
- Idéal pour les pipelines automatisés à fort volume
- Usage recommandé : génération de code simple, résumés, triage rapide

**Gemini 2.5 Flash**
- Meilleur équilibre vitesse/qualité
- "Thinking mode" : le modèle raisonne avant de répondre (similaire à o1/Claude extended thinking)
- Excellent pour du débogage avec explications détaillées
- Bon rapport coût/performance pour les APIs payantes

**Gemini 2.5 Pro**
- Le modèle haut de gamme de Google
- **2 millions de tokens de contexte** : peut ingérer des repos entiers, des PDFs volumineux, des bases de code complètes
- Très performant sur le code (analyse, refactoring, génération)
- Accès via Google AI Studio ou API
- Pricing plus élevé mais reste compétitif face à Claude Opus ou GPT-4o

**Gemini 1.5 Flash**
- Ancienne génération mais encore largement utilisée
- Très stable et prévisible
- Souvent le fallback par défaut dans des applications existantes

> [!info]
> Le "contexte 2M tokens" de Gemini 2.5 Pro n'est pas juste un chiffre marketing. Concrètement, on peut envoyer l'intégralité d'un projet de 50 000 lignes de code et demander "où sont les failles de sécurité ?" ou "génère la documentation complète". Aucun autre modèle cloud n'offre cette capacité en 2025.

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
Modèle : Gemini 2.5 Pro
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
| Gratuit              | Oui (beta)       | Limité           |
| Contexte max         | 1M tokens        | 200k tokens      |
| Multimodal           | Oui (images,PDF) | Images           |
| Maturité             | Beta 2025        | Stable           |
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
> CONTEXTE GÉANT  →  2M tokens = projets entiers analysables
> GRATUIT         →  Quota le plus généreux du marché
> MULTIMODAL      →  Images, PDFs, vidéos, audio
> GOOGLE CLOUD    →  Intégration native Firebase, BigQuery, GCP
> GROUNDING       →  Recherche web en temps réel dans les réponses
> VITESSE         →  Gemini 2.0 Flash parmi les plus rapides
```

Cas d'usage où Gemini excelle :
- Analyser un repo GitHub entier pour comprendre son architecture
- Extraire des données structurées depuis des PDFs techniques
- Prototyper rapidement avec intégration Google Cloud
- Pipelines à haut volume grâce au quota API généreux

### Points faibles de Gemini

> [!warning] Limites à connaître
> - **Précision code complexe** : sur du débogage fin ou de l'algorithmique avancée, Claude 3.5/3.7 reste supérieur
> - **Verbosité** : Gemini tend à sur-expliquer, à ajouter des nuances non demandées. Compenser avec un system prompt directif
> - **Gemini CLI** : encore en beta en 2025, moins mature que Claude Code pour les workflows agents complexes
> - **Incohérence** : les réponses varient plus qu'avec Claude sur les mêmes prompts
> - **Confidentialité** : données traitées par Google, conditions à vérifier pour usage professionnel

---

## 3. Mistral AI

Mistral AI est une startup française fondée en 2023 par d'anciens de Google DeepMind et Meta. Elle propose des modèles performants, souvent open source, avec un avantage clé pour les entreprises européennes : conformité RGPD et données hébergées en Europe.

### Les modèles Mistral

```
            FAMILLE MISTRAL
                  |
    +-------------+-------------+
    |             |             |
Généralistes   Code         Open Source
    |             |             |
Large 2       Codestral     Mistral 7B
Small         (FIM)         Mixtral 8x7B
                            Mixtral 8x22B
                |
             Multimodal
                |
            Pixtral
```

**Mistral Large 2**
- 128k tokens de contexte
- Excellent raisonnement, comparable à GPT-4o sur benchmarks généraux
- Support natif de nombreuses langues européennes
- Fonction calling robuste pour les agents
- Prix compétitif face à GPT-4o

**Mistral Small**
- Modèle économique, bon rapport qualité/prix
- Idéal pour des tâches simples à haut volume : classification, résumés, extraction
- Latence faible

**Codestral**
- Modèle dédié au code, 80k tokens de contexte
- Support **FIM (Fill-In-the-Middle)** : le modèle peut compléter du code au milieu d'un fichier, pas seulement à la fin
- Excellente intégration IDE via plugins
- Accès beta gratuit sur la Plateforme Mistral

**Mistral 7B**
- Open source (licence Apache 2.0)
- 7 milliards de paramètres seulement, mais performances remarquables
- Peut tourner localement sur une machine avec 8-16 Go RAM
- Base pour de nombreux fine-tunes communautaires

**Mixtral 8x7B et 8x22B**
- Architecture MoE (Mixture of Experts) : 8 experts, seuls 2 activés par token
- Performances d'un 47B avec le coût d'un 13B en inférence
- Open source, très utilisé dans les déploiements locaux d'entreprise

**Pixtral**
- Modèle multimodal de Mistral
- Analyse d'images en plus du texte
- Utile pour analyser des captures d'écran, des diagrammes

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
    "title": "Codestral",
    "provider": "mistral",
    "model": "codestral-latest",
    "apiKey": "VOTRE_CLE_CODESTRAL"
  }
}
```

> [!info]
> Le FIM (Fill-In-the-Middle) est crucial pour la complétion dans un IDE. Quand on appuie sur Tab au milieu d'une fonction, le modèle doit comprendre ce qui est déjà écrit AVANT et ce qui viendra APRES le curseur. Codestral est entraîné spécifiquement pour cette tâche, ce que les modèles généralistes font moins bien.

### Points forts de Mistral

```
MISTRAL - CE QUI LE DISTINGUE

Origine française → RGPD, données en Europe, confiance UE
Open source      → Mistral 7B, Mixtral : utilisables localement
Codestral        → Meilleure complétion FIM du marché cloud
Prix compétitif  → Mistral Small parmi les moins chers par token
Mixte experts    → Mixtral : performance 47B, coût 13B
API propre       → SDK Python bien documenté et stable
```

### Points faibles de Mistral

> [!warning]
> - Mistral Large reste en dessous de Claude 3.7 et GPT-4o sur du code très complexe
> - Moins de ressources communautaires qu'OpenAI (moins de tutos, exemples)
> - Pixtral (multimodal) moins mature que Gemini ou GPT-4V
> - Mixtral local demande beaucoup de RAM (Mixtral 8x7B : ~48 Go pour Q4)

---

## 4. DeepSeek

DeepSeek est un laboratoire chinois (filiale du hedge fund High-Flyer) qui a créé la surprise fin 2024 en publiant des modèles open source comparables aux meilleurs modèles propriétaires, à une fraction du coût.

### Pourquoi DeepSeek fait l'actualité

En janvier 2025, la sortie de DeepSeek-R1 a fait chuter les actions des entreprises tech américaines. Raisons :

- **Performances** : DeepSeek-V3 et R1 rivalisent avec GPT-4o et o1 sur de nombreux benchmarks
- **Coût d'entraînement** : DeepSeek affirme avoir entraîné V3 pour ~6M$ vs ~100M$ pour GPT-4
- **Open source** : les poids des modèles sont publiés, téléchargeables, utilisables librement
- **Prix API** : 10 à 30 fois moins cher que GPT-4o pour qualité comparable

### Les modèles DeepSeek

```
DEEPSEEK
  |
  +--- DeepSeek-V3         → Usage général, challenger GPT-4o
  |
  +--- DeepSeek-R1         → Raisonnement profond, rival de o1
  |      |
  |      +--- R1-Distill-Qwen-7B   → Version légère, localisable
  |      +--- R1-Distill-Llama-70B → Version large
  |
  +--- DeepSeek-Coder-V2   → Spécialisé code, 128k contexte
```

**DeepSeek-V3**
- Modèle de base à 671 milliards de paramètres (MoE)
- Architecture MoE : seuls 37B activés par token en pratique
- Open source (poids disponibles)
- Prix API : ~$0.27/M tokens input (vs ~$5/M pour GPT-4o)
- Excellent pour du code Python, JavaScript, SQL

**DeepSeek-R1**
- Modèle de raisonnement, pensée en chaîne visible
- Rival direct de OpenAI o1
- Open source et disponible en plusieurs tailles via distillation
- Idéal pour de l'algorithmique complexe, des mathématiques, du débogage difficile

**DeepSeek-Coder-V2**
- Spécialisé code
- 128k tokens de contexte
- Support de 338 langages de programmation
- Performances HumanEval parmi les meilleures pour un modèle code

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
    model="deepseek-chat",     # deepseek-chat = DeepSeek-V3
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
    model="deepseek-reasoner",  # = DeepSeek-R1
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

Qwen est la famille de modèles développée par Alibaba Cloud. Elle s'est imposée fin 2024 comme l'une des meilleures offres open source, particulièrement pour le code.

### Modèles Qwen disponibles

**Qwen2.5-Coder**
- Déclinaisons : 7B, 14B, 32B, 72B paramètres
- Spécialisé code : Python, JavaScript, Java, C++, SQL, Bash...
- Excellent rapport qualité/taille, notamment Qwen2.5-Coder-14B
- Open source, disponible sur Hugging Face

**Qwen2.5 (généraliste)**
- Modèle général très capable, jusqu'à 72B
- Multilingue, incluant excellent support du chinois

**Qwen2-VL**
- Version multimodale (vision + texte)
- Analyse d'images, diagrammes, captures d'écran

### Utilisation via Ollama

```bash
# Télécharger et lancer Qwen2.5-Coder 14B (bon équilibre perf/RAM)
ollama pull qwen2.5-coder:14b
ollama run qwen2.5-coder:14b

# Version 7B pour les machines avec moins de RAM (8 Go suffisent)
ollama pull qwen2.5-coder:7b
ollama run qwen2.5-coder:7b

# Exemple d'utilisation en pipeline
echo "Écris une fonction Python de validation d'email" | ollama run qwen2.5-coder:7b
```

> [!tip] Analogie
> Qwen2.5-Coder-14B c'est comme avoir un développeur junior très compétent qui travaille sur votre propre ordinateur, sans jamais envoyer votre code à l'extérieur. Moins puissant qu'un Claude 3.7 sur les cas complexes, mais 100% local et gratuit.

### API via Alibaba Cloud (DashScope)

```python
import os
from openai import OpenAI  # Compatible API OpenAI

client = OpenAI(
    api_key=os.environ.get("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

response = client.chat.completions.create(
    model="qwen2.5-coder-72b-instruct",
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

Meta publie ses modèles Llama en open source depuis 2023. En 2025-2026, c'est l'un des modèles open source les plus performants.

**Llama 3.3 70B**
- Modèle généraliste très performant pour 70B paramètres
- Excellent pour le code, l'analyse, la rédaction technique
- Disponible partout : Ollama, LM Studio, Groq, Together AI, Replicate

**Llama 3.1 405B**
- Near-frontier performance en open source
- Nécessite ~200 Go de RAM pour tourner en local (plusieurs GPU haut de gamme)
- Utilisable via API sur Groq, Together AI, Fireworks AI

**CodeLlama 34B**
- Version spécialisée code de Llama 2
- Plus ancien mais encore utilisé, notamment pour son support FIM
- Tourne sur machine avec ~24 Go RAM

```bash
# Via Groq (gratuit, ultra-rapide)
# Créer un compte sur console.groq.com, obtenir une clé API

from groq import Groq

client = Groq(api_key="GROQ_API_KEY")

response = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[
        {"role": "user", "content": "Explique les design patterns Strategy et Observer avec du code Python"}
    ]
)

print(response.choices[0].message.content)
```

> [!info]
> **Groq** n'est pas à confondre avec Grok (xAI). Groq est une entreprise américaine qui fabrique des puces LPU (Language Processing Unit) ultra-rapides. Leur API permet d'utiliser Llama 3.3 70B gratuitement à des vitesses de 500-800 tokens/seconde, contre 50-100 tokens/seconde sur les serveurs GPU classiques.

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
  → Llama 3.3 70B, Gemma 2 27B, Mixtral 8x7B
  → Ultra-rapide (LPU), gratuit avec limite daily
  → API compatible OpenAI

TOGETHER AI (api.together.xyz)
  → Dizaines de modèles open source
  → Crédit gratuit au démarrage
  → Llama, Qwen, Mistral, DeepSeek...

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
  → Gemini 2.0 Flash / 2.5 Flash gratuits
  → Clé API gratuite avec quota généreux

FIREWORKS AI (fireworks.ai)
  → Modèles Llama, Qwen, Mixtral
  → Crédit gratuit au démarrage
  → Très rapide
```

> [!tip] Stratégie recommandée
> Pour un développeur individuel qui veut maximiser ses ressources gratuites :
> 1. **Groq** pour les requêtes rapides (Llama 70B)
> 2. **Google AI Studio** pour les analyses longues (contexte 2M Gemini)
> 3. **Ollama local** pour le code confidentiel (Qwen2.5-Coder, Phi-4)
> 4. **API DeepSeek** quand la vitesse et le coût sont prioritaires sur la confidentialité

---

## 8. Tableau de sélection : quel modèle choisir ?

```
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Modèle              | Éditeur  | Contexte | Force principale | Prix   | Open Src | Confid.   |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Gemini 2.5 Pro      | Google   | 2M tok   | Analyse grands   | $$     | Non      | EU/US     |
|                     |          |          | repos, multim.   |        |          |           |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Gemini 2.0 Flash    | Google   | 1M tok   | Vitesse, gratuit | Gratuit| Non      | EU/US     |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Mistral Large 2     | Mistral  | 128k     | RGPD, Europe     | $$     | Non      | EU +++    |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Codestral           | Mistral  | 80k      | Complétion FIM   | $      | Non      | EU +++    |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Mistral 7B          | Mistral  | 32k      | Local, open src  | Gratuit| Oui      | Local +++  |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| DeepSeek-V3         | DeepSeek | 128k     | Prix/perf, code  | $      | Oui      | CN (éviter|
|                     |          |          |                  |        |          | code priv)|
+---------------------+----------+----------+------------------+--------+----------+-----------+
| DeepSeek-R1         | DeepSeek | 128k     | Raisonnement     | $      | Oui      | CN / Local|
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Qwen2.5-Coder 14B   | Alibaba  | 128k     | Code local, FIM  | Gratuit| Oui      | Local +++  |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Llama 3.3 70B       | Meta     | 128k     | Open source, code| Gratuit| Oui      | Local +++  |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Phi-4 14B           | Microsoft| 16k      | Petit, performant| Gratuit| Oui      | Local +++  |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Command R+          | Cohere   | 128k     | RAG, doc search  | $$     | Non      | US        |
+---------------------+----------+----------+------------------+--------+----------+-----------+
| Grok-3              | xAI      | 128k     | Code, raison.    | $$     | Non      | US        |
+---------------------+----------+----------+------------------+--------+----------+-----------+
```

**Légende prix :** Gratuit = quota généreux sans paiement | $ = <$2/M tokens | $$ = $2-10/M tokens | $$$ = >$10/M tokens

> [!example] Scénarios de choix
>
> **"J'analyse un repo de 200k lignes"**
> → Gemini 2.5 Pro (seul avec 2M tokens de contexte)
>
> **"Je veux de la complétion de code dans mon IDE sans payer"**
> → Codestral (beta gratuit) ou Qwen2.5-Coder via Continue.dev + Ollama
>
> **"J'ai du code propriétaire, zéro cloud"**
> → Qwen2.5-Coder:14b ou Phi-4 via Ollama en local
>
> **"Je veux du raisonnement profond pour de l'algo complexe"**
> → DeepSeek-R1 (local) ou Gemini 2.5 Pro avec thinking mode
>
> **"Je suis une entreprise française soumise au RGPD"**
> → Mistral Large 2 (données en Europe, entreprise française)
>
> **"Je veux gratuit + rapide pour mes expérimentations"**
> → Groq API avec Llama 3.3 70B

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
    ┌─────┴─────┐            ┌──────┴──────┐          ┌──────┴──────┐
    │           │            │             │          │             │
  Flash      2.5 Pro     Codestral    Mistral 7B    V3 (cheap)  R1 (reason)
  (gratuit)  (2M ctx)    (FIM code)   (local)       (API)       (open src)
    │
  CLI tool
  (beta)
                        │
          ┌─────────────┼──────────────┐
          │             │              │
    ╔═════╧═════╗ ╔═════╧═════╗ ╔═════╧═════╗
    ║   QWEN    ║ ║   LLAMA   ║ ║  PHI / CO ║
    ║ (Alibaba) ║ ║  (Meta)   ║ ║ GROK etc  ║
    ╚═════╤═════╝ ╚═════╤═════╝ ╚═══════════╝
          │             │
    Coder 7/14B    3.3 70B          ACCÈS GRATUIT
    (Ollama)       (Groq)     ┌─────────────────────┐
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
2. Écrire un prompt identique et le soumettre à : Gemini 2.5 Pro (AI Studio), Mistral Large 2 (chat.mistral.ai), et Claude 3.7 (claude.ai)
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
    # TODO : implémenter avec Gemini 2.5 Flash
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
3. Configurer `~/.continue/config.json` avec Codestral comme modèle de complétion
4. Installer **Ollama** et télécharger `qwen2.5-coder:7b` comme modèle de chat local
5. Configurer Continue pour utiliser Qwen local pour le chat et Codestral pour la complétion
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
