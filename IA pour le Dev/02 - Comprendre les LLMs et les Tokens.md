# Comprendre les LLMs et les Tokens

Qu'est-ce qu'un Large Language Model, comment fonctionne-t-il réellement, et pourquoi la notion de "token" est-elle centrale pour tout développeur qui utilise ces modèles ? Cette note démystifie les mécanismes sous-jacents pour mieux exploiter les LLMs dans un contexte de développement.

---

## 1. Qu'est-ce qu'un LLM ?

Un **Large Language Model** (Grand Modèle de Langage) est un réseau de neurones entraîné sur des quantités massives de texte, capable de prédire et de générer du langage naturel (et du code) de manière cohérente et contextuelle.

> [!tip] Analogie : L'Autocomplete Surpuissant
> Imagine la fonction d'autocomplétion de ton téléphone, mais entraînée sur toute la littérature humaine, tous les dépôts GitHub, toute Wikipedia, et des milliards de pages web. À chaque étape, le LLM prédit : "étant donné tout ce qui précède, quel est le prochain token le plus probable ?" C'est fondamentalement ce qu'il fait — mais à une échelle et avec une sophistication qui donnent l'illusion de la compréhension.

### Différences avec les autres approches IA

| Approche | Principe | Exemple | Limites |
|---|---|---|---|
| **Règles expertes** | Si X alors Y, codé manuellement | Filtre spam par mots-clés | Rigide, ne s'adapte pas |
| **ML classique** | Entraînement sur features définies | Détection fraude bancaire | Feature engineering manuel |
| **LLM** | Prédiction probabiliste sur tokens | GPT-4, Claude, Gemini | Hallucinations, coût, latence |

Les LLMs ne "comprennent" pas au sens humain. Ils ont appris des **corrélations statistiques** si complexes et à si grande échelle qu'ils peuvent résoudre des problèmes qui semblent nécessiter de la compréhension.

---

## 2. Comment ça marche (simplifié)

### L'Architecture Transformer

Les LLMs modernes reposent tous sur l'architecture **Transformer**, introduite en 2017 dans le papier "Attention Is All You Need". Avant cela, les modèles de langage utilisaient des RNN (Recurrent Neural Networks) qui traitaient le texte séquentiellement — lent, et incapable de capturer les dépendances à longue distance.

Le Transformer introduit un mécanisme révolutionnaire : le **Self-Attention**.

### Le Self-Attention : pourquoi le modèle "comprend" le contexte

Le self-attention permet à chaque token d'une séquence de "regarder" tous les autres tokens et de pondérer leur importance. Contrairement au RNN qui lit mot par mot, le Transformer traite toute la séquence en parallèle.

```
Exemple : "La banque est en faillite. Il retire ses économies de la banque."

Pour le deuxième "banque", l'attention va peser :
  - "faillite" → poids élevé (contexte financier)
  - "économies" → poids élevé
  - "La" → poids faible
  → Le modèle infère que "banque" = institution financière, pas rive d'une rivière
```

### Pipeline complet : du texte à la prédiction

```
┌─────────────────────────────────────────────────────────────────┐
│                    PIPELINE LLM                                 │
└─────────────────────────────────────────────────────────────────┘

  Input texte
  "def add(a, b):"
       │
       ▼
┌──────────────┐
│  TOKENIZER   │  "def" | " add" | "(a" | "," | " b" | "):" 
│              │  Découpe en unités discrètes (tokens)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  EMBEDDINGS  │  Chaque token → vecteur de dimensions (ex: 4096)
│              │  Espace mathématique où sens similaire = proximité
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  ATTENTION   │  Chaque token interagit avec tous les autres
│  LAYERS      │  Multi-head attention : plusieurs "perspectives"
│  (xN)        │  simultanées sur le même texte
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  FEED-       │  Transformations non-linéaires supplémentaires
│  FORWARD     │  par couche
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  LOGITS +    │  Distribution de probabilité sur tout le vocabulaire
│  SOFTMAX     │  (~50 000 tokens possibles)
└──────┬───────┘
       │
       ▼
  Token suivant prédit : "\n    return a + b"
```

### Pré-entraînement vs Fine-tuning

**Pré-entraînement** : Le modèle apprend à prédire le prochain token sur des téraoctets de texte. C'est coûteux (des millions de dollars), cela prend des semaines sur des milliers de GPUs. Le résultat est un modèle de base (*base model*) qui sait compléter du texte, mais n'est pas encore "utile" comme assistant.

**Fine-tuning** : On affine le modèle sur des données spécifiques (code, médecine, juridique). Beaucoup moins coûteux. Permet de spécialiser un modèle pour un domaine.

**RLHF (Reinforcement Learning from Human Feedback)** : La raison pour laquelle Claude et ChatGPT sont "polis" et suivent vos instructions. Des humains évaluent des paires de réponses, le modèle apprend à produire des réponses que les humains préfèrent. C'est aussi ce qui rend les modèles refuser certaines requêtes.

> [!info] Pourquoi les LLMs refusent certaines choses ?
> Le RLHF crée des "garde-fous" comportementaux. Ce n'est pas de la censure codée en dur, c'est un apprentissage par renforcement : le modèle a été récompensé des milliers de fois pour avoir refusé certains types de requêtes. Ces comportements sont profondément intégrés dans les poids du modèle.

---

## 3. Les Tokens : tout comprendre

### Qu'est-ce qu'un token ?

Un **token** n'est pas un mot. C'est une unité de texte issue de la tokenisation, généralement :
- Un mot courant complet
- Une partie d'un mot long
- Un caractère de ponctuation
- Un espace + un mot
- Un symbole spécial

La tokenisation est réalisée par des algorithmes comme **BPE (Byte Pair Encoding)** ou **SentencePiece**, qui trouvent les sous-unités les plus fréquentes dans le corpus d'entraînement.

### Exemples concrets de tokenisation

```python
# Exemples avec tiktoken (tokenizer OpenAI, similaire à Claude)
import tiktoken
enc = tiktoken.get_encoding("cl100k_base")

# Mots courants = 1 token
"chat"        → ["chat"]                        = 1 token
"code"        → ["code"]                        = 1 token
"function"    → ["function"]                    = 1 token

# Mots composés = 2+ tokens
"chatbot"     → ["chat", "bot"]                 = 2 tokens
"JavaScript"  → ["Java", "Script"]              = 2 tokens
"TypeScript"  → ["Type", "Script"]              = 2 tokens
"undefined"   → ["undef", "ined"]               = 2 tokens

# Mots rares ou très longs = plusieurs tokens
"supercalifragilisticexpialidocious"
              → ["super", "cal", "if", "rag",
                 "il", "istic", "exp", "ial",
                 "id", "oci", "ous"]            = 11 tokens

# Nombres = variable
"2024"        → ["2024"]                        = 1 token
"3.14159"     → ["3", ".", "14", "159"]         = 4 tokens

# Code : chaque symbole peut être un token
"function add(a, b) {"
              → ["function", " add", "(", "a",
                 ",", " b", ")", " {"]          = 8 tokens
```

### Règles générales

```
┌─────────────────────────────────────────────────┐
│           RATIO TOKENS / CARACTÈRES             │
├──────────────────┬──────────────────────────────┤
│ Anglais          │ ~4 caractères = 1 token       │
│ Français         │ ~3.3 caractères = 1 token     │
│ Code (Python)    │ ~3 caractères = 1 token       │
│ Code (C++)       │ ~2.5 caractères = 1 token     │
│ Chinois/Japonais │ ~1-2 caractères = 1 token     │
│ Emojis           │ 1-2 tokens chacun             │
└──────────────────┴──────────────────────────────┘
```

### Pourquoi le français coûte ~20% plus cher que l'anglais

Les tokenizers sont entraînés majoritairement sur des données anglaises. Les mots français sont moins fréquents dans le corpus, donc découpés en sous-unités plus petites.

```
"cat"          = 1 token
"chat"         = 1 token   (fréquent en anglais aussi)

"authentication" = 1-2 tokens   (anglais)
"authentification" = 3-4 tokens (français, moins courant)

"développement" → ["d", "ével", "opp", "ement"] = 4 tokens
"development"   → ["development"]               = 1 token
```

> [!warning] Impact budgétaire du français
> Pour un projet traçant beaucoup de texte français, prévois une surestimation de 15-25% sur tes estimations de coût par rapport à un calcul basé sur l'anglais. En pratique pour le code avec commentaires en français : compte 3 caractères par token plutôt que 4.

### Tokenisation du code

Le code a ses propres particularités de tokenisation :

```python
# Python : les indentations comptent !
"    def foo():"   # 4 espaces + "def" + "foo" + "()" + ":"
# → [" ", " ", " ", " ", "def", " foo", "():", ]  ~ 7 tokens
# Les 4 espaces = 4 tokens séparés dans certains encodages !

# Alternative avec tabulation
"\tdef foo():"   # → ["\t", "def", " foo", "():"] = 4 tokens
# La tabulation = 1 token. Avantage technique des tabs !

# Symboles courants en code
"{", "}", "(", ")", "[", "]", ".", ",", ";"  → 1 token chacun
"=>", "->", "::", "!=", "=="                 → 1-2 tokens
"/**", "*/", "///"                           → 1-2 tokens
```

### Outils pour compter ses tokens

```bash
# tiktoken (OpenAI, compatible GPT et similaires)
pip install tiktoken

python3 -c "
import tiktoken
enc = tiktoken.get_encoding('cl100k_base')
text = 'Votre texte ici'
tokens = enc.encode(text)
print(f'Nombre de tokens: {len(tokens)}')
"
```

```bash
# API Anthropic : comptage natif
curl https://api.anthropic.com/v1/messages/count_tokens \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model": "claude-sonnet-4-6", "messages": [{"role": "user", "content": "Votre texte"}]}'
```

Outils en ligne :
- **OpenAI Tokenizer** : platform.openai.com/tokenizer
- **Anthropic Tokenizer** : console.anthropic.com/tokenizer  
- **tiktokenizer.vercel.app** : visualisation colorée par token

### Coût réel en euros — exemples chiffrés

Tarifs Claude Sonnet 4.6 (avril 2026, approximatifs) :

```
┌──────────────────────────────────────────────────────────────────┐
│                    COÛTS CLAUDE SONNET 4.6                      │
├─────────────────────┬────────────────┬───────────────────────────┤
│ Volume              │ Input          │ Output                    │
├─────────────────────┼────────────────┼───────────────────────────┤
│ 1M tokens           │ ~3$            │ ~15$                      │
│ 1 message (1k tok)  │ 0.003$         │ 0.015$                    │
│ 100 messages/jour   │ 0.30$/jour     │ 1.50$/jour                │
│ 1M tokens (cached)  │ ~0.30$         │ -                         │
└─────────────────────┴────────────────┴───────────────────────────┘

Exemple concret : assistant de code
- System prompt     : 2 000 tokens
- 20 messages/jour : 1 000 tokens chacun  
- Réponses          : 500 tokens chacune
- Total input/jour  : 20 × (2000 + 1000) = 60 000 tokens = ~0.18$
- Total output/jour : 20 × 500 = 10 000 tokens = ~0.15$
→ Coût journalier : ~0.33$ → ~10$/mois
```

> [!tip] Optimisation des coûts
> Le cache de prompt (Prompt Caching) peut réduire le coût du system prompt répété de 90%. Si ton system prompt fait 2000 tokens et est envoyé 1000 fois/mois, le caching te fait économiser ~5.40$/mois sur ce seul élément.

---

## 4. La Fenêtre de Contexte

### Qu'est-ce que la Context Window ?

La **context window** (fenêtre de contexte) est la quantité maximale de tokens que le modèle peut "voir" simultanément lors d'une inférence. Elle inclut :
- Le system prompt
- Tout l'historique de la conversation
- Le message utilisateur courant
- La réponse en cours de génération

Au-delà de cette limite, les informations sont tronquées ou perdues.

### Tableau des contextes par modèle (2025-2026)

```
┌──────────────────────┬─────────────────┬─────────────────────────┐
│ Modèle               │ Context Window  │ Notes                   │
├──────────────────────┼─────────────────┼─────────────────────────┤
│ GPT-4o               │ 128k tokens     │ ~96 000 mots            │
│ Claude Sonnet 4.6    │ 200k tokens     │ ~150 000 mots           │
│ Claude Opus 4        │ 200k tokens     │ Meilleure qualité       │
│ Gemini 2.5 Pro       │ 2M tokens       │ Record actuel           │
│ Llama 3.1 (70B)      │ 128k tokens     │ Open source             │
│ Mistral Large        │ 128k tokens     │ Européen                │
│ DeepSeek-V3          │ 128k tokens     │ Très performant         │
│ Qwen 2.5 (72B)       │ 128k tokens     │ Multilingue fort        │
└──────────────────────┴─────────────────┴─────────────────────────┘
```

### Le problème de la "relecture exponentielle"

> [!warning] Un fait souvent ignoré
> À chaque message que tu envoies, le LLM ne lit pas seulement ton nouveau message. Il relit **TOUTE la conversation depuis le début**. Le coût ne croît pas linéairement, il croît de façon quadratique.

Formule du coût total de tokens lus dans une conversation :

```
Coût total = n × (n+1) / 2 × tokens_par_message

Avec n = nombre de messages, tokens_par_message = taille moyenne
```

Exemple chiffré avec 10 messages de 1 000 tokens :

```
Message 1 : lit 1 000 tokens
Message 2 : lit 2 000 tokens  (msg1 + msg2)
Message 3 : lit 3 000 tokens
...
Message 10: lit 10 000 tokens

Total = 1+2+3+4+5+6+7+8+9+10 = 55 × 1000 = 55 000 tokens lus
        (pour 10 000 tokens de contenu réel)

→ Coût réel = 5.5x le contenu nominal
```

```
┌────────────────────────────────────────────────────────┐
│           CROISSANCE DU COÛT PAR CONVERSATION          │
├──────────────┬────────────────┬────────────────────────┤
│ Nb messages  │ Tokens réels   │ Tokens effectivement   │
│              │ échangés       │ lus (coût réel)        │
├──────────────┼────────────────┼────────────────────────┤
│ 5            │ 5 000          │ 15 000  (3x)           │
│ 10           │ 10 000         │ 55 000  (5.5x)         │
│ 20           │ 20 000         │ 210 000 (10.5x)        │
│ 50           │ 50 000         │ 1 275 000 (25.5x)      │
└──────────────┴────────────────┴────────────────────────┘
```

Implications pratiques :
- Les longues conversations sont exponentiellement plus chères
- La latence augmente aussi avec la longueur du contexte
- Pour les apps en production : implémenter une **fenêtre glissante** ou un **résumé automatique** de l'historique

### Lost in the Middle : l'attention non-uniforme

Les modèles n'accordent pas une attention égale à toutes les parties du contexte. Les recherches montrent un phénomène appelé **"Lost in the Middle"** : les informations situées au milieu d'un long contexte sont significativement moins bien retenues.

```
COURBE D'ATTENTION SELON LA POSITION DANS LE CONTEXTE

Attention
  ▲
  │
  █                                                   █
  █ ██                                             ████
  █ ████                                         ██████
  █ ██████                                     ████████
  █ ████████                                 ██████████
  █ ██████████   ...........zone creuse...  ████████████
  └─────────────────────────────────────────────────────► Position
  Début                  Milieu                     Fin
  (Pic fort)          (Creux: -40-60%)           (Pic fort)

Données : Liu et al., 2023 "Lost in the Middle"
```

> [!warning] Impact sur les performances
> Dans un contexte de 100k tokens, les informations positionnées entre 20% et 80% du document sont mémorisées avec une précision inférieure de 40 à 60% par rapport aux informations au début ou à la fin. Ce n'est pas une métaphore : les benchmarks le mesurent.

Implications pratiques pour les développeurs :

```
┌─────────────────────────────────────────────────────────────────┐
│                 PLACEMENT OPTIMAL DES INFORMATIONS              │
├─────────────────────────────────────────────────────────────────┤
│ DÉBUT du contexte  → Instructions système, règles critiques,    │
│                      identité du persona                        │
│                                                                 │
│ MILIEU du contexte → Documents de référence, exemples,         │
│                      contexte général (moins critique)          │
│                                                                 │
│ FIN du contexte    → La vraie question, les données actuelles,  │
│                      ce qui nécessite une réponse précise       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Les Paramètres de Génération

Ces paramètres contrôlent comment le modèle sélectionne le prochain token parmi les candidats possibles.

### Temperature (0.0 → 2.0)

La **temperature** contrôle la "netteté" de la distribution de probabilité. Avec temperature = 0, le modèle choisit toujours le token le plus probable. Avec une temperature élevée, il introduit de l'aléatoire.

```
Temperature 0.0 : Distribution "sharp"
  Token A : 85%  ████████████████████
  Token B :  8%  ██
  Token C :  4%  █
  Token D :  3%  █
  → Toujours Token A

Temperature 0.7 : Distribution équilibrée  
  Token A : 55%  █████████████
  Token B : 25%  ██████
  Token C : 12%  ███
  Token D :  8%  ██
  → Généralement A, parfois B

Temperature 1.5 : Distribution "flat"
  Token A : 32%  ████████
  Token B : 28%  ███████
  Token C : 22%  █████
  Token D : 18%  ████
  → Très aléatoire, tous les tokens ont leur chance
```

```
┌─────────────────────────────────────────────────────────────────┐
│                    GUIDE PRATIQUE TEMPERATURE                   │
├────────────┬────────────────┬────────────────────────────────── ┤
│ Valeur     │ Comportement   │ Usage recommandé                  │
├────────────┼────────────────┼────────────────────────────────── ┤
│ 0.0        │ Déterministe   │ Code, extraction de données,      │
│            │                │ JSON structuré, math              │
│ 0.3        │ Très cohérent  │ Révision de code, debug           │
│ 0.7        │ Équilibré      │ Documentation, explication        │
│ 1.0        │ Créatif        │ Rédaction, brainstorming          │
│ 1.5+       │ Imprévisible   │ Créativité pure, poésie           │
│            │                │ (risque d'hallucinations élevé)   │
└────────────┴────────────────┴────────────────────────────────── ┘
```

### Top-p (Nucleus Sampling)

Le **Top-p** ne sélectionne que parmi les tokens dont les probabilités cumulées atteignent p. Avec top-p = 0.9, on garde seulement les tokens qui représentent les 90% premiers de probabilité cumulée.

```python
# Effet du Top-p
top_p = 0.9  # On coupe après avoir atteint 90% de probabilité cumulée

Tokens candidats (triés par probabilité) :
  "return" : 60% → cumulé : 60%  ✓
  "yield"  : 20% → cumulé : 80%  ✓
  "raise"  : 12% → cumulé : 92%  ✗ (dépasse 0.9, exclu)
  "print"  :  5% → cumulé : 97%  ✗
  ...

→ Choix entre "return" et "yield" seulement
```

### Top-k

Limite le nombre de tokens candidats à exactement k. Moins sophistiqué que top-p car ignore la distribution réelle.

```python
top_k = 40  # On ne considère que les 40 tokens les plus probables
# Simple mais efficace, utilisé dans Ollama et llama.cpp
```

### Max Tokens et Stop Sequences

```python
# Configuration API Anthropic
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,          # Limite la réponse à 1024 tokens
    stop_sequences=["```", "Human:", "---"],  # S'arrête si un de ces patterns apparaît
    temperature=0.0,          # Déterministe pour du code
    messages=[{"role": "user", "content": prompt}]
)
```

> [!info] Temperature vs Top-p
> En pratique, modifier l'un ou l'autre donne des effets similaires. La convention courante : fixer top-p = 1.0 et jouer uniquement sur temperature (ou vice-versa). Anthropic recommande de ne pas modifier les deux simultanément.

---

## 6. Taille des Modèles et Paramètres

### Qu'est-ce que "7B paramètres" ?

Les "paramètres" d'un modèle sont les **poids** du réseau de neurones — des nombres réels (flottants) appris pendant l'entraînement. Un modèle "7B" a 7 milliards de ces nombres. C'est la mémoire du modèle.

Chaque paramètre occupe de la RAM : en FP32 (32 bits = 4 octets), 7B paramètres = **28 Go de RAM**. D'où l'importance de la quantisation (voir section 7).

### Tableau : taille → capacités → ressources

```
┌──────────┬────────────────────────────┬──────────┬─────────────────────┐
│ Taille   │ Capacités                  │ RAM min  │ Vitesse approx.     │
├──────────┼────────────────────────────┼──────────┼─────────────────────┤
│ 1-3B     │ Très limité, tâches        │ 2-4 GB   │ Très rapide         │
│          │ simples, réponses courtes  │          │ (mobile possible)   │
├──────────┼────────────────────────────┼──────────┼─────────────────────┤
│ 7B       │ Bon compromis, code        │ 6-8 GB   │ Rapide (CPU viable) │
│          │ simple, raisonnement basique│         │ ~10-30 tok/s CPU    │
├──────────┼────────────────────────────┼──────────┼─────────────────────┤
│ 13B      │ Bon équilibre, code        │ 12-16 GB │ Modéré              │
│          │ intermédiaire, nuances     │          │ GPU recommandé      │
├──────────┼────────────────────────────┼──────────┼─────────────────────┤
│ 34B      │ Très bon, proche GPT-4     │ 24-32 GB │ Lent sur CPU        │
│          │ niveau en code             │          │ GPU fortement conseillé│
├──────────┼────────────────────────────┼──────────┼─────────────────────┤
│ 70B      │ Excellent, niveau pro      │ 40-48 GB │ GPU haut de gamme   │
│          │ dans la plupart des tâches │          │ ou multi-GPU        │
├──────────┼────────────────────────────┼──────────┼─────────────────────┤
│ 405B+    │ État de l'art              │ 200 GB+  │ Cloud ou datacenter │
│          │ Llama 3.1 405B, GPT-4...   │          │ nécessaire          │
└──────────┴────────────────────────────┴──────────┴─────────────────────┘
```

### Emergent Abilities : les capacités qui apparaissent par seuils

Certaines capacités n'apparaissent pas graduellement avec la taille du modèle, elles **émergent** soudainement au-delà d'un certain seuil de paramètres.

```
Capacité                          Seuil d'émergence approximatif
─────────────────────────────────────────────────────────────────
Cohérence multi-phrases            ~ 7B
Raisonnement en plusieurs étapes   ~ 13B
Arithmetic à plusieurs chiffres    ~ 13B
Chain-of-thought spontané          ~ 70B
Calibration des incertitudes       ~ 70B
Coding complex (algo + debug)      ~ 34B
Raisonnement logique formel        ~ 70B
```

> [!info] Pourquoi les emergent abilities importent pour les devs
> Un modèle 6B ne "dégrade pas gentiment" sur les tâches complexes : il échoue complètement. Si tu as besoin de raisonnement multi-étapes pour du refactoring complexe, un 7B quantisé ne fera pas 80% du travail d'un 70B — il fera un travail fondamentalement différent et souvent inutilisable.

---

## 7. La Quantisation

### Pourquoi quantiser ?

La quantisation réduit la précision des poids du modèle (ex: de 32 bits à 4 bits), ce qui réduit drastiquement la RAM nécessaire au prix d'une légère perte de qualité.

```
Modèle 7B en FP32  : 7 000 000 000 × 4 octets = 28 Go RAM
Modèle 7B en Q4    : 7 000 000 000 × 0.5 octets ≈ 3.5 Go RAM
→ Division par 8 de la RAM nécessaire !
```

### Types de quantisation

```
┌──────────┬───────────────┬──────────────────┬────────────────────────┐
│ Format   │ Bits/paramètre│ RAM (modèle 7B)  │ Qualité relative       │
├──────────┼───────────────┼──────────────────┼────────────────────────┤
│ FP32     │ 32 bits       │ ~28 Go           │ Référence 100%         │
│ FP16     │ 16 bits       │ ~14 Go           │ ~99.9% (imperceptible) │
│ BF16     │ 16 bits       │ ~14 Go           │ ~99.9% (standard GPU)  │
│ Q8_0     │  8 bits       │  ~7 Go           │ ~99.5% (quasi sans perte)│
│ Q5_K_M   │  5 bits       │  ~4.4 Go         │ ~99%   (recommandé)    │
│ Q4_K_M   │  4 bits       │  ~3.8 Go         │ ~98%   (bon compromis) │
│ Q3_K_M   │  3 bits       │  ~3.1 Go         │ ~95%   (notable)       │
│ Q2_K     │  2 bits       │  ~2.7 Go         │ ~88%   (usage limité)  │
└──────────┴───────────────┴──────────────────┴────────────────────────┘
```

> [!tip] Recommandation pratique
> Pour un usage courant en local (Ollama, LM Studio) :
> - **Q5_K_M** : meilleur équilibre qualité/RAM, quasi imperceptible
> - **Q4_K_M** : si RAM limitée, perte très légère sur tâches complexes
> - Éviter Q2 pour du code : les erreurs de logique augmentent significativement

### Nomenclature des fichiers GGUF

```
llama-3.1-8b-instruct-Q4_K_M.gguf
          │    │        │  │
          │    │        │  └─ Variante (M = Medium, S = Small, L = Large)
          │    │        └──── Quantisation (Q4 = 4 bits)
          │    └─────────────Variant (instruct = fine-tuné pour instructions)
          └──────────────────Taille (8b = 8 milliards de paramètres)
```

---

## 8. Les Benchmarks pour Comparer les Modèles

### Benchmarks principaux en développement

**HumanEval** (OpenAI) :
- 164 problèmes de programmation Python
- Métrique : Pass@1 (le code passe les tests du premier coup)
- Simple mais saturé : les modèles récents dépassent 90%

**SWE-bench Verified** :
- Vrais issues GitHub avec patches vérifiés par humains
- Mesure la capacité à résoudre de vrais bugs dans de vrais dépôts
- Métrique : % d'issues résolues
- Beaucoup plus difficile et représentatif du travail réel

**MBPP (Mostly Basic Python Problems)** :
- 374 problèmes Python de difficulté variable
- Bonne corrélation avec la productivité dev quotidienne

**BigCodeBench** :
- 1140 tâches de code complexes
- Appels d'API réels, librairies tierces
- Plus difficile à "gamer" que HumanEval

**MMLU (Massive Multitask Language Understanding)** :
- 57 domaines de connaissances générales
- Utile pour évaluer les capacités transversales

**GPQA (Graduate-level Professional QA)** :
- Questions de niveau doctorat en sciences
- Mesure le raisonnement scientifique avancé

### Limites des benchmarks

> [!warning] Ne pas confondre benchmark et usage réel
> Les benchmarks présentent plusieurs problèmes critiques :
> - **Contamination** : les données de test se retrouvent parfois dans les données d'entraînement
> - **Gaming** : les modèles sont fine-tunés spécifiquement pour bien scorer sur les benchmarks populaires
> - **Écart avec le réel** : un modèle excellent sur HumanEval peut être médiocre sur ton codebase spécifique
> - **Contexte absent** : les benchmarks testent des tâches isolées, pas l'intégration dans un workflow réel

```
[!example] Exemple de discordance
GPT-3.5 scorait 70% sur HumanEval → impressionnant sur le papier
En pratique pour du code React avec une architecture spécifique : 40-50% utilisable
→ Toujours tester sur TON cas d'usage avec TES données
```

---

## 9. Hallucinations et Limites

### Pourquoi les LLMs "inventent" ?

Le LLM ne "sait" pas qu'il ne sait pas. Il prédit le token le plus probable étant donné le contexte, même si aucun token ne correspond à une vérité factuelle. Il n'a pas de mécanisme natif pour dire "je n'ai pas cette information".

> [!info] Mécanisme des hallucinations
> Quand le modèle rencontre une question dont la réponse n'était pas clairement dans ses données d'entraînement, il extrapole à partir de patterns similaires. Le résultat est syntaxiquement correct et sémantiquement plausible — mais factuellement inventé.

### Types d'hallucinations en code

```python
# Type 1 : Méthode inexistante dans une librairie réelle
import pandas as pd
df = pd.DataFrame({"a": [1, 2, 3]})
df.smart_filter(lambda x: x > 1)  # ← N'EXISTE PAS
# pandas n'a pas de méthode "smart_filter"

# Type 2 : Paramètre inventé pour une vraie fonction
response = requests.get(url, timeout=30, retry_count=3)
# "retry_count" n'est PAS un paramètre de requests.get

# Type 3 : Package inexistant mais plausible
# pip install pandas-smart-utils   ← N'EXISTE PAS

# Type 4 : Version incorrecte d'une API
# "Dans Python 3.12, vous pouvez utiliser dict.merge()"
# → merge() n'existe pas, c'est dict | dict ou dict.update()
```

### Comment minimiser les hallucinations

```
┌─────────────────────────────────────────────────────────────────┐
│              STRATÉGIES ANTI-HALLUCINATIONS                     │
├─────────────────────────────────────────────────────────────────┤
│ 1. Temperature basse (0.0-0.2) pour code et faits              │
│ 2. Fournir la documentation dans le contexte                   │
│    ("Voici la doc de l'API X : [coller la doc]")               │
│ 3. Demander la justification :                                  │
│    "Cite la méthode exacte et son module d'origine"            │
│ 4. Vérification systématique dans l'IDE / REPL                 │
│ 5. Utiliser des modèles avec Search (Perplexity, Claude        │
│    avec web search) pour les infos récentes                    │
│ 6. Ne jamais copier-coller sans tester                         │
└─────────────────────────────────────────────────────────────────┘
```

### Le Cutoff de Connaissance

Chaque modèle a une **date de coupure** (knowledge cutoff) : il ne connaît rien de ce qui s'est passé après la fin de son entraînement.

```
Modèle                  Cutoff approximatif
─────────────────────────────────────────────
GPT-4o                  Avril 2024
Claude Sonnet 4.6       Début 2025
Gemini 2.5 Pro          Début 2025
Llama 3.1               Décembre 2023
```

Implications :
- Le modèle peut ignorer des breaking changes dans des librairies populaires
- Les nouvelles APIs (post-cutoff) seront hallusinées ou inconnues
- Solution : fournir la documentation à jour dans le contexte (RAG)

> [!warning] Pièges fréquents du cutoff
> - "Comment utiliser React 19 ?" → Le modèle peut répondre avec React 18 syntax sans le signaler
> - "Quel est le dernier modèle OpenAI ?" → Réponse probablement obsolète
> - Librairies mises à jour fréquemment (LangChain, LlamaIndex) : particulièrement risqué

---

## Carte Mentale ASCII

```
                        ┌─────────────────────┐
                        │      LLM & TOKENS   │
                        └──────────┬──────────┘
                                   │
          ┌────────────────┬───────┴────────┬─────────────────┐
          │                │                │                 │
     ┌────┴────┐      ┌────┴────┐     ┌────┴────┐      ┌────┴────┐
     │ FONCT.  │      │ TOKENS  │     │CONTEXTE │      │MODÈLES  │
     └────┬────┘      └────┬────┘     └────┬────┘      └────┬────┘
          │                │               │                 │
     ┌────┴────┐      ┌────┴────┐    ┌─────┴─────┐    ┌────┴────┐
     │Transfor-│      │≠ mots   │    │128k-2M tok│    │ 1B-405B │
     │  mer    │      │BPE/SPM  │    │Lost in Mid│    │Paramètr.│
     │Attention│      │~4 chars │    │Relecture  │    │Emergent │
     │RLHF     │      │€ impact │    │exponentiel│    │Abilities│
     └─────────┘      └─────────┘    └───────────┘    └─────────┘
          │
     ┌────┴─────────────────────────────────────────────┐
     │                  PARAMÈTRES                      │
     │  Temperature │ Top-p │ Top-k │ Max tokens │ Stop │
     └──────────────┴───────┴───────┴────────────┴──────┘
          │
     ┌────┴─────────────────────────────────────────────┐
     │               QUANTISATION                       │
     │  FP32 > FP16 > Q8 > Q5_K_M > Q4_K_M > Q2       │
     └──────────────────────────────────────────────────┘
          │
     ┌────┴─────────────────────────────────────────────┐
     │              LIMITATIONS                         │
     │  Hallucinations │ Cutoff │ Lost in Middle        │
     │  Coût quadratique conversations longues          │
     └──────────────────────────────────────────────────┘
```

---

## Exercices Pratiques

### Exercice 1 : Compter et optimiser les tokens

Objectif : développer l'intuition du coût en tokens.

```
1. Prends un de tes prompts habituels (system prompt + question type)
2. Compte les tokens avec tiktoken ou le tokenizer Anthropic
3. Identifie les parties les plus coûteuses
4. Réécris le prompt pour économiser 20% de tokens sans perte de qualité
5. Vérifie que la qualité de réponse est maintenue sur 5 exemples
```

Questions à te poser :
- Quels mots ou phrases peut-on éliminer ?
- Peut-on remplacer des explications longues par des exemples courts ?
- Les instructions répétitives peuvent-elles être factorisées ?

### Exercice 2 : Tester l'effet de la temperature

Objectif : observer concrètement l'impact des paramètres de génération.

```python
import anthropic

client = anthropic.Anthropic()
prompt = "Écris une fonction Python qui trie une liste de dictionnaires par une clé donnée"

temperatures = [0.0, 0.5, 1.0, 1.5]

for temp in temperatures:
    response = client.messages.create(
        model="claude-haiku-3-5",  # Haiku pour le coût
        max_tokens=300,
        temperature=temp,
        messages=[{"role": "user", "content": prompt}]
    )
    print(f"\n=== Temperature {temp} ===")
    print(response.content[0].text)
    # Envoie le MÊME prompt 3 fois à la même temperature
    # et compare les résultats : sont-ils identiques ? différents ?
```

Analyse les résultats :
- À quelle temperature le code est-il le plus fiable ?
- À quelle temperature apparaissent les premières divergences ?
- Quels types d'erreurs introduit une temperature élevée ?

### Exercice 3 : Mesurer le phénomène "Lost in the Middle"

Objectif : constater empiriquement le phénomène d'attention non-uniforme.

```
1. Crée un document long (1000+ mots) avec des données fictives :
   - Section 1 : données importantes A (début)
   - Sections 2-8 : remplissage neutre
   - Section 5 : donnée cachée B (milieu exact)  ← TEST
   - Section 9 : données importantes C (fin)

2. Insère ce document dans un prompt Claude

3. Pose des questions sur A, B, et C successivement

4. Compare la précision des réponses selon la position :
   - A (début) : précision ?
   - B (milieu) : précision ?
   - C (fin) : précision ?

5. Recommence avec la donnée B déplacée au début ou à la fin
   → Constate la différence de mémorisation
```

---

## Liens et Ressources

- [[01 - Panorama des IA pour Développeurs]] — Vue d'ensemble des outils disponibles et cas d'usage
- [[03 - Prompt Engineering pour le Code]] — Techniques pour exploiter les LLMs efficacement
- [[09 - IA Locale avec Ollama]] — Exécuter des modèles quantisés en local
- [[10 - LM Studio et Hardware Local]] — Interface graphique et configuration hardware pour LLMs locaux
- [[Glossaire IA Dev]] — Définitions rapides de tous les termes techniques
