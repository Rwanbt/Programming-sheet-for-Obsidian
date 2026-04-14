# Claude & Claude Code — Maîtrise Avancée et Optimisation des Tokens

> [!info] À qui s'adresse ce cours ?
> Ce cours s'adresse à **tout développeur** qui utilise Claude, Claude Code, ou tout autre LLM (GPT-4, Gemini, modèles locaux via Ollama/LM Studio/OpenCode). Il part de zéro mais va très loin : comprendre comment les tokens sont consommés, comment configurer son environnement pour minimiser les coûts, et comment tirer le maximum de chaque session.
>
> **Prérequis** : savoir utiliser un terminal et avoir Claude Code installé (`npm install -g @anthropic-ai/claude-code`).

---

## 1. Comprendre les Tokens — La Monnaie de l'IA

### Qu'est-ce qu'un token ?

Un **token** est l'unité de traitement des modèles de langage. Ce n'est pas un mot, ni un caractère : c'est un fragment de texte découpé par un algorithme appelé **tokenizer**.

```
"Bonjour le monde"  →  ["Bon", "jour", " le", " monde"]   = 4 tokens
"Hello world"        →  ["Hello", " world"]                 = 2 tokens
"printf("            →  ["printf", "(\""]                    = 2 tokens
```

> [!tip] Règle empirique
> - 1 token ≈ 0,75 mot en anglais
> - 1 token ≈ 0,6 mot en français (le français coûte ~20% plus cher)
> - 1 token ≈ 4 caractères pour du code ASCII
> - Le code C/Rust est dense → économique en tokens
> - Le JSON verbeux est coûteux → préférer YAML ou formats courts

### La fenêtre de contexte (Context Window)

Chaque modèle a une **fenêtre de contexte** maximale, exprimée en tokens :

```
┌─────────────────────────────────────────────────────────────┐
│                   FENÊTRE DE CONTEXTE                       │
│                                                             │
│  claude-opus-4-6    : ~200 000 tokens ≈ 150 000 mots       │
│  claude-sonnet-4-6  : ~200 000 tokens ≈ 150 000 mots       │
│  claude-haiku-4-5   : ~200 000 tokens ≈ 150 000 mots       │
│  GPT-4o             : ~128 000 tokens                       │
│  Gemini 2.0 Flash   : ~1 000 000 tokens                     │
│  Llama 3.3 (local)  : 8 000 à 128 000 tokens selon version │
└─────────────────────────────────────────────────────────────┘
```

**Ce qui compte dans la fenêtre** : TOUT — system prompt, historique de conversation, résultats d'outils, fichiers lus, sorties de commandes.

### Comment on est facturé

Les APIs Claude facturent :
- **Tokens d'entrée (input)** : tout ce que Claude lit (historique + prompt)
- **Tokens de sortie (output)** : tout ce que Claude génère (réponse + code)

```
Prix indicatifs Claude (API, avril 2026) :
┌─────────────────┬──────────────┬───────────────┐
│ Modèle          │ Input/1M tk  │ Output/1M tk  │
├─────────────────┼──────────────┼───────────────┤
│ claude-opus-4-6 │     $15      │     $75       │
│ sonnet-4-6      │      $3      │     $15       │
│ haiku-4-5       │    $0.80     │      $4       │
└─────────────────┴──────────────┴───────────────┘
```

> [!warning] Les sorties coûtent 5× plus que les entrées
> Demander à Claude de "réécrire tout le fichier" coûte 5× plus cher que "modifier la ligne 42". Toujours cibler précisément.

---

## 2. La Relecture Exponentielle — Le Piège Fondamental

### Comment Claude lit vraiment votre conversation

C'est le concept le plus important à comprendre. Claude ne "se souvient" pas : il **relit intégralement la conversation à chaque message**.

```
Message 1 : 100 tokens lus
Message 2 : 200 tokens lus  (100 historique + 100 nouveau)
Message 3 : 300 tokens lus  (200 historique + 100 nouveau)
...
Message 10: 1000 tokens lus

Total cumulé : 100+200+300+...+1000 = 5500 tokens consommés
Pour seulement 1000 tokens de contenu réel !
```

> [!warning] Coût RÉEL = n × (n+1) / 2 × taille_moyenne
> Une conversation de 20 messages de 500 tokens chacun coûte en réalité :
> 20 × 21 / 2 × 500 = **105 000 tokens** pour 10 000 tokens de contenu.
> **C'est pourquoi les longues conversations explosent les coûts.**

### La solution : éviter les messages séparés inutiles

```
❌ MAUVAIS — 3 messages séparés = 6 relectures cumulées :
  Toi : "Lis le fichier main.c"
  Claude : [lit et répond]
  Toi : "Explique la fonction malloc_wrapper"
  Claude : [relit tout + répond]
  Toi : "Ajoute un commentaire"
  Claude : [relit tout + répond]

✓ BON — 1 message groupé :
  Toi : "Lis main.c, explique malloc_wrapper et ajoute un commentaire"
  Claude : [fait tout en une passe]
```

**Règle d'or** : grouper les instructions connexes dans un seul message.

---

## 3. Lost in the Middle — Le Problème de la Mémoire Longue

### Le phénomène

Les recherches montrent que les LLMs ont une capacité d'attention **non uniforme** sur leur contexte :

```
Attention du modèle selon la position dans le contexte :

  Début ████████████████████ (forte attention)
  Milieu ████████             (attention réduite 40-60%)
  Fin    ████████████████████ (forte attention)
```

Ce phénomène s'appelle **"Lost in the Middle"** : les informations placées au milieu d'une longue conversation sont moins bien prises en compte.

> [!tip] Implication pratique
> - Les instructions importantes → **début de conversation ou dernier message**
> - Les fichiers les plus critiques → lire en **dernier** avant la demande
> - Si Claude "oublie" une instruction donnée il y a 15 messages → **rappeler dans le dernier message**

### Stratégies contre le Lost in the Middle

```
1. CLAUDE.md : placer les règles permanentes ici (toujours en début)
2. Répétition ciblée : "Rappel : pas de commentaires dans le code"
3. /compact : condenser le milieu, garder un résumé dense
4. Démarrer une nouvelle session pour les sujets sans rapport
```

---

## 4. Le Prompt Cache — 5 Minutes pour Économiser 90%

### Comment fonctionne le cache

Anthropic implémente un **cache automatique de prompts**. Quand Claude relit votre historique, si ce contexte est identique à une lecture récente, il est servi depuis le cache à **~10% du prix normal**.

```
┌──────────────────────────────────────────────────────┐
│               PROMPT CACHE                           │
│                                                      │
│  Durée de vie : 5 minutes (TTL = 300 secondes)      │
│  Réduction coût : ~90% sur les tokens mis en cache   │
│  Applicable à : historique, fichiers lus, tools      │
│                                                      │
│  ⚠️  Après 5 min d'inactivité → cache expiré         │
│      → prochain message relit TOUT à plein tarif     │
└──────────────────────────────────────────────────────┘
```

> [!tip] Hack du prompt cache
> Si vous devez faire une pause, **revenez avant 5 minutes**. Un simple message "continue" ou "ok" suffit à maintenir le cache actif.
> 
> Dans Claude Code, le timer `/loop` utilise 270s (sous les 5 min) pour rester dans le cache. Au-delà de 300s, il vaut mieux attendre 20-30 min (un seul cache miss vaut mieux que 12 relectures partielles).

### Impact concret

```
Session de 2h avec 100 000 tokens d'historique :

Sans cache : 100 000 × 40 messages = 4 000 000 tokens facturés
Avec cache : 100 000 × 0.1 × 40  = 400 000 tokens facturés (prix cache)
             + nouveaux tokens normaux

Économie : ~$9 → $0.90 sur Sonnet (×10 moins cher !)
```

---

## 5. CLAUDE.md — La Configuration Maîtresse

### Qu'est-ce que CLAUDE.md ?

`CLAUDE.md` est un fichier Markdown placé à la racine de votre projet (ou dans `~/.claude/CLAUDE.md` pour les règles globales). Claude le lit **automatiquement à chaque session** — c'est votre "system prompt permanent" sans effort.

```
~/.claude/CLAUDE.md          ← règles globales (tous les projets)
mon-projet/CLAUDE.md         ← règles du projet
mon-projet/src/CLAUDE.md     ← règles du sous-dossier (héritées)
```

### Structure d'un CLAUDE.md efficace

```markdown
# Mon Projet — Instructions Claude

## Contexte
Application web FastAPI + React. Base PostgreSQL.
Stack : Python 3.12, Node 20, Docker Compose.

## Règles de Code
- Python : PEP8, type hints obligatoires, docstrings Google style
- Ne JAMAIS modifier les migrations Alembic existantes
- Tests pytest obligatoires pour toute nouvelle fonction

## Conventions Git
- Branches : feature/nom, fix/nom, docs/nom
- Commits : format conventionnel (feat:, fix:, docs:, refactor:)

## Fichiers à NE PAS toucher
- config/production.env
- migrations/ (lecture seule)

## Comportement attendu
- Toujours demander avant de toucher à plus de 5 fichiers
- Proposer un plan avant tout refactoring
- Réponses courtes sauf si explication demandée
```

> [!tip] Le CLAUDE.md comme économie de tokens
> Sans CLAUDE.md : vous répétez vos conventions à chaque session → 200-500 tokens perdus.
> Avec CLAUDE.md : zéro répétition, règles toujours actives, chargé une fois en cache.

### Optimiser son CLAUDE.md

```markdown
❌ MAUVAIS (verbeux, redondant) :
"Quand tu écris du code Python, s'il te plaît, n'oublie pas d'ajouter 
des type hints à toutes les fonctions que tu crées ou modifies, 
c'est très important pour nous."

✓ BON (dense, précis) :
"Python : type hints obligatoires sur toutes les fonctions."
```

**Principes** : dense, impératif, sans politesse inutile. Les politesses coûtent des tokens sans ajouter de valeur.

---

## 6. .claudeignore — Exclure l'Inutile

### Fonctionnement

`.claudeignore` fonctionne exactement comme `.gitignore` : il dit à Claude quels fichiers **ne pas lire automatiquement** lors de l'exploration du projet.

```
# .claudeignore exemple

# Dépendances (jamais utiles à lire)
node_modules/
.venv/
__pycache__/
target/          # Rust build output

# Fichiers générés
dist/
build/
*.min.js
*.min.css

# Données volumineuses
*.csv
*.parquet
data/raw/

# Logs
*.log
logs/

# Config secrètes (ne jamais exposer)
.env
.env.*
secrets/
```

> [!warning] Sans .claudeignore
> Claude peut explorer `node_modules/` (200 000+ fichiers, des millions de tokens) si vous demandez "analyse mon projet". Avec `.claudeignore`, il est protégé.

### Stratégie de direction vers des dossiers spécifiques

Au lieu de dire "regarde mon projet", guidez précisément :

```
❌ "Peux-tu analyser mon application ?"
   → Claude va potentiellement lire 50 fichiers pour comprendre la structure

✓ "Regarde src/api/routes/users.py et src/models/user.py"
   → Lecture de 2 fichiers ciblés, 10× moins de tokens
```

**Règle** : plus vous êtes précis sur les fichiers concernés, moins Claude consomme de contexte en exploration.

---

## 7. Le Système de Mémoire Persistante

### memory/ — La mémoire entre sessions

Claude Code dispose d'un système de fichiers mémoire dans `~/.claude/projects/[projet]/memory/`. Ces fichiers sont relus dans les futures conversations pour maintenir la continuité.

```
~/.claude/projects/mon-projet/memory/
├── MEMORY.md          ← index de toutes les mémoires (chargé auto)
├── user_prefs.md      ← préférences personnelles
├── feedback.md        ← corrections et retours
├── project.md         ← contexte projet en cours
└── reference.md       ← pointeurs vers ressources externes
```

> [!tip] Quand demander à Claude de mémoriser
> "Mémorise que je préfère les tests d'intégration sans mocking"
> "Retiens que le fichier config.py ne doit jamais être modifié"
> "Note que le bug sur l'auth est résolu — cause : cookie SameSite=None"

### Ce qu'il NE faut PAS mémoriser

- Structure du code (lire le code actuel est toujours plus fiable)
- Historique git (utiliser `git log`)
- État temporaire de la session en cours
- Listes de tâches (utiliser TodoWrite dans la session)

---

## 8. Les Commandes Essentielles

### /compact — Compresser sans perdre

`/compact` condense l'historique de la conversation en un résumé dense, réduisant massivement les tokens tout en préservant les informations clés.

```
Avant /compact : 80 000 tokens d'historique
Après /compact : 8 000 tokens de résumé (ratio ×10 typique)

Économie sur les prochains messages : 72 000 tokens × prix_cache
```

**Quand utiliser** :
- Quand la barre de contexte Claude Code atteint 60-70%
- Après avoir résolu un problème complexe (résumer avant de passer au suivant)
- Quand vous sentez Claude "oublier" des choses du début

> [!tip] Auto-compact intelligent
> Dans les paramètres Claude Code, activer l'auto-compact à **60% du contexte**. Cela déclenche automatiquement une compression avant que le contexte soit critique.

### /clear — Repartir à zéro

`/clear` efface complètement l'historique. La session recommence vierge.

```
Utiliser /clear quand :
✓ Changer complètement de sujet (ex: passer de "debug C" à "déploiement Docker")
✓ Claude semble "confus" ou donne des réponses incohérentes
✓ La conversation a beaucoup dérivé par rapport au sujet initial
✓ Après /compact si le résumé ne suffit pas

NE PAS utiliser /clear quand :
✗ En plein milieu d'un refactoring multi-fichiers
✗ Quand Claude a du contexte utile sur votre projet
✗ Pour "économiser" sans raison (vous perdez du contexte utile)
```

### Combo résumé + /clear

La technique la plus efficace pour les longues sessions :

```
1. Avant /clear, demander : "Fais un résumé structuré de ce qu'on a fait,
   les décisions prises, l'état actuel, et les prochaines étapes."
2. Claude génère un résumé de 200-500 tokens
3. /clear (efface les 50 000 tokens d'historique)
4. Coller le résumé dans le premier message de la nouvelle session
5. Reprendre le travail avec un contexte frais et ciblé
```

> [!example] Économie réelle du combo
> 50 000 tokens d'historique → 400 tokens de résumé
> Économie sur le prochain message : 49 600 tokens (soit ~$0.15 sur Sonnet)
> Sur une journée de travail avec 10 /clear : ~$1.50 économisés

### /context — Voir ce qui est chargé

`/context` affiche un résumé de ce qui est actuellement dans la fenêtre de contexte : fichiers lus, tokens utilisés, pourcentage restant.

```
Utiliser pour :
- Savoir combien de contexte reste disponible
- Identifier quels fichiers volumineux ont été lus
- Décider si un /compact est nécessaire
```

---

## 9. Sub-agents — Coûts et Stratégies de Parallélisation

### Qu'est-ce qu'un sub-agent ?

Un sub-agent est une instance Claude **séparée** lancée pour effectuer une tâche spécifique. Chaque sub-agent a son propre contexte, ses propres outils, et consomme ses propres tokens.

```
┌──────────────────────────────────────────────────────┐
│                   AGENT PRINCIPAL                    │
│   Contexte : 50 000 tokens                           │
│         │                                            │
│    ┌────┴────────────────┐                           │
│    ▼                     ▼                           │
│  Agent A              Agent B                        │
│  Contexte: 30K        Contexte: 25K                  │
│  Tâche: Tests         Tâche: Docs                    │
│         │                     │                      │
│         └──────────┬──────────┘                      │
│                    ▼                                  │
│              Résultats agrégés                        │
│              dans agent principal                     │
└──────────────────────────────────────────────────────┘
```

### Coût réel des sub-agents

> [!warning] Les sub-agents coûtent cher
> Chaque sub-agent démarre avec son propre contexte + hérite du contexte parent résumé.
> Lancer 3 agents en parallèle peut coûter 2-3× un seul agent.
> **Ce n'est pas gratuit** — c'est un choix vitesse vs coût.

```
Calcul approximatif :
Agent principal      : 10 000 tokens contexte
3 sub-agents lancés  : 3 × (10 000 + travail propre) tokens
                     = 3 × 25 000 = 75 000 tokens
vs. travail séquentiel : 10 000 + 15 000 + 15 000 = 40 000 tokens

Les sub-agents coûtent ~1.9× plus mais sont 3× plus rapides.
```

### Stratégie d'optimisation : parallélisation ciblée

```
✓ Utiliser des sub-agents quand :
  - Tâches INDÉPENDANTES (pas de dépendances entre elles)
  - Tâches LONGUES (>5 min chacune)
  - Exploration de plusieurs dossiers/fichiers différents

✗ NE PAS utiliser quand :
  - Tâches séquentielles (B dépend de A)
  - Tâches courtes (overhead > gain)
  - Budget limité
```

### Le hack "parallélisation 5 minutes avant la fin"

> [!tip] Technique avancée de rentabilisation
> Juste avant que votre contexte soit plein (à ~80%), lancez plusieurs sub-agents en parallèle pour les tâches restantes. Pendant qu'ils travaillent, votre session principale peut se terminer proprement.
>
> **Pourquoi ?** Les sub-agents héritent du contexte actuel (encore riche) mais travaillent en parallèle — vous "dépensez" vos tokens restants de façon productive plutôt que de les laisser expirer.

---

## 10. Plan Mode — Réfléchir Avant d'Agir

### Qu'est-ce que le Plan Mode ?

Le Plan Mode force Claude à **planifier et présenter son approche** avant d'exécuter quoi que ce soit. C'est activé via `/plan` ou dans les paramètres.

```
Sans Plan Mode :
  Toi : "Refactorise le module auth"
  Claude : [commence à modifier 12 fichiers immédiatement]
  → Risque de casser quelque chose, difficile à annuler

Avec Plan Mode :
  Toi : "Refactorise le module auth"
  Claude : [présente un plan détaillé]
  Toi : [valide, modifie, ou annule]
  Claude : [exécute seulement après validation]
```

> [!tip] Économie tokens avec Plan Mode
> Le hack le plus sous-estimé : **"Ne fais aucun changement tant que tu n'as pas 95% de confiance. Pose-moi des questions de suivi."**
>
> Cela force Claude à clarifier les ambiguïtés AVANT d'agir, évitant des allers-retours coûteux ("non c'est pas ça que je voulais" → 3 messages de correction = tokens perdus).

### Utilisation efficace

```
Bon usage du Plan Mode :
1. Pour tout changement touchant >3 fichiers
2. Pour des refactorings d'architecture
3. Pour des migrations de base de données
4. Quand vous n'êtes pas sûr de l'approche

Pas nécessaire pour :
- Corriger un bug précis dans un fichier connu
- Ajouter une ligne de code simple
- Reformater/documenter
```

---

## 11. Skills — Les Raccourcis Réutilisables

### Qu'est-ce qu'un skill ?

Un **skill** (ou slash command personnalisée) est une commande prédéfinie qui déclenche un workflow complexe. Claude Code vient avec des skills intégrés (`/commit`, `/review-pr`) et permet d'en créer des personnalisés.

```
# Exemple de skill personnalisé : /deploy-check
Emplacement : ~/.claude/skills/deploy-check.md

Contenu :
  Avant tout déploiement, vérifier :
  1. Tests passent (pytest -v)
  2. Linting propre (ruff check .)
  3. Variables d'env production définies
  4. Migrations à jour (alembic heads)
  Générer un rapport PASS/FAIL pour chaque point.
```

> [!tip] Économie avec les skills
> Sans skill : répéter les instructions à chaque fois → 200-500 tokens par invocation
> Avec skill `/deploy-check` : zéro token d'instruction, juste le résultat

### Skills intégrés utiles

```
/commit     → analyse les changements et génère un message de commit
/review-pr  → revue complète d'une pull request
/compact    → compresse le contexte
/clear      → réinitialise la session
/fast       → bascule en mode rapide (même modèle, output plus rapide)
```

---

## 12. Serveurs MCP — Étendre les Capacités

### Model Context Protocol (MCP)

MCP permet à Claude de se connecter à des **sources de données et outils externes** directement dans la session : bases de données, APIs, fichiers distants, services web.

```
Claude ←→ MCP Server ←→ Base de données PostgreSQL
Claude ←→ MCP Server ←→ API GitHub (issues, PRs)
Claude ←→ MCP Server ←→ Notion / Jira / Linear
Claude ←→ MCP Server ←→ Slack / Gmail
Claude ←→ MCP Server ←→ Navigateur web (Playwright)
```

### Configuration d'un MCP

```json
// ~/.claude/config.json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    },
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_xxxx" }
    }
  }
}
```

> [!tip] MCP vs lecture de fichiers
> Sans MCP : "Lis le fichier SQL et explique le schéma" → vous devez copier-coller
> Avec MCP PostgreSQL : Claude interroge directement `\d table_name` → résultat frais, zéro copier-coller

### Impact sur les tokens

Les résultats MCP entrent dans le contexte comme n'importe quelle autre information. Attention aux requêtes qui retournent beaucoup de données — **toujours limiter les résultats** :

```sql
-- ❌ Mauvais : retourne potentiellement millions de lignes
SELECT * FROM logs;

-- ✓ Bon : ciblé et limité
SELECT message, level, created_at FROM logs
WHERE level = 'ERROR' AND created_at > NOW() - INTERVAL '1 hour'
LIMIT 20;
```

---

## 13. System Prompts — Configurer le Comportement Global

### Qu'est-ce qu'un system prompt ?

Le **system prompt** est une instruction donnée au modèle avant la conversation. Dans Claude Code, c'est géré via CLAUDE.md. Dans l'API, c'est le paramètre `system`.

```python
# Utilisation API Claude
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="""Tu es un expert Python senior.
    Règles :
    - Réponses en français
    - Code avec type hints toujours
    - Jamais de variables globales
    - Toujours proposer les tests associés""",
    messages=[
        {"role": "user", "content": "Comment implémenter un cache LRU ?"}
    ]
)
```

> [!tip] Optimiser le system prompt
> Un system prompt bien écrit économise des tokens sur CHAQUE message.
> Il est mis en cache (prompt cache) après le premier appel.
> Investir 30 min à écrire un bon system prompt = économies sur des milliers d'appels.

---

## 14. Choisir le Bon Modèle — L'Équilibre Coût/Qualité

### La règle des 3 niveaux

```
┌──────────────────────────────────────────────────────────────┐
│                    HIÉRARCHIE DES MODÈLES                    │
│                                                              │
│  HAIKU (rapide, économique)                                  │
│  → Tâches simples et répétitives                            │
│  → Classification, extraction, formatage                    │
│  → Tests/résumés rapides                                     │
│  → Coût : ~$0.80/1M input tokens                           │
│                                                              │
│  SONNET (équilibré, recommandé par défaut)                  │
│  → Développement quotidien                                   │
│  → Debugging, refactoring, documentation                    │
│  → Analyse de code, revues                                   │
│  → Coût : ~$3/1M input tokens                              │
│                                                              │
│  OPUS (le plus puissant)                                     │
│  → Problèmes complexes nécessitant raisonnement profond     │
│  → Architecture système, algorithmes non triviaux           │
│  → Décisions critiques et irréversibles                     │
│  → Coût : ~$15/1M input tokens                             │
└──────────────────────────────────────────────────────────────┘
```

### Stratégie en pratique

```
Session de développement type :

9h00 - Planification architecture    → OPUS    (décision critique)
9h30 - Implémentation des features   → SONNET  (travail principal)
15h00 - Génération de tests unitaires → HAIKU   (tâche répétitive)
16h00 - Debugging complexe           → SONNET  (analyse précise)
17h00 - Génération de doc/README     → HAIKU   (formatage simple)
```

> [!example] Économie par le bon choix de modèle
> Générer 100 fichiers de tests : Haiku vs Sonnet
> Haiku : 100 × 2 000 tokens × $0.004 = $0.80
> Sonnet : 100 × 2 000 tokens × $0.015 = $3.00
> **Économie : $2.20 sur une seule tâche répétitive**

---

## 15. RTK et les Fichiers/Sorties Volumineuses

### RTK : Le Rust Token Killer

"RTK" désigne le problème des sorties de compilateur Rust extrêmement verbeuses. Le compilateur Rust est réputé pour ses messages d'erreur détaillés — ce qui est une qualité pour les humains mais un gouffre à tokens pour l'IA.

```bash
# Une compilation Rust avec des warnings peut générer :
$ cargo build
   Compiling myproject v0.1.0
warning: unused variable: `x`
  --> src/main.rs:15:9
   |
15 |     let x = compute();
   |         ^ help: if this is intentional, prefix it with an underscore: `_x`
   |
   = note: `#[warn(unused_variables)]` on by default

[... 200 lignes de warnings similaires ...]
error[E0502]: cannot borrow `data` as mutable...
[... 50 lignes d'explication du borrow checker ...]
```

> [!warning] Impact RTK
> Une sortie `cargo build` peut faire 500-2000 tokens. Si Claude la lit dans la session, c'est autant de contexte consommé par des logs.

### Solutions au problème des sorties volumineuses

```bash
# 1. Filtrer avant d'envoyer à Claude
cargo build 2>&1 | grep "^error" | head -20

# 2. Rediriger vers un fichier, ne partager que l'essentiel
cargo build 2> build_errors.txt
# Puis : "voici les erreurs : [coller seulement les lignes error:]"

# 3. Utiliser les flags silencieux
cargo build -q 2>&1 | grep "error\[" 

# 4. Pour pytest Python
pytest --tb=short -q  # au lieu de pytest --tb=long -v
```

### Principe général sur les outputs de commandes

```
Avant de coller une sortie de commande dans Claude :
1. Vérifier la longueur (>100 lignes = potentiellement problématique)
2. Filtrer pour ne garder que les erreurs/warnings pertinents
3. Ne jamais coller node_modules, stack traces Java entières, logs systèmes bruts

Exemples de tailles typiques :
  npm install : 50-200 lignes (OK si problème détecté)
  cargo build (erreur) : 20-500 lignes (filtrer)
  docker logs : illimité → TOUJOURS limiter avec --tail 50
  pytest -v : OK si <50 tests, filtrer pour >100
```

---

## 16. Peak Hours et Performances

### Impact des heures de pointe

Les APIs LLM ont des performances variables selon la charge serveur :

```
Heures de pointe (US business hours, généralement) :
  14h-22h heure française = 8h-16h EST
  → Latences plus élevées
  → Parfois rate limits plus stricts
  → Débit de tokens/seconde réduit

Heures creuses :
  Nuit française (2h-8h)
  Weekends
  → Meilleures performances
  → Pour les gros batches, préférer ces horaires
```

> [!tip] Stratégie batch processing
> Si vous avez à générer beaucoup de contenu (ex: 50 fichiers de documentation), planifiez ça la nuit ou le week-end pour de meilleures performances et parfois des tarifs réduits selon les tiers.

---

## 17. Hygiène de Contexte — Les Bonnes Pratiques

### Les 10 règles d'or

```
1. SESSION CIBLÉE
   Une session = un objectif précis.
   Ne pas mélanger "debug auth" et "refactoring DB" dans la même session.

2. FICHIERS PRÉCIS
   "Regarde src/auth/jwt.py lignes 45-80" > "regarde le code"

3. FERMER AVANT DE COMMENCER
   Lire les fichiers importants EN DERNIER avant la demande principale.

4. COMPACT PROACTIF
   Ne pas attendre d'être à 90% de contexte → /compact à 60%

5. RÉSUMÉ AVANT /clear
   Toujours sauvegarder le contexte essentiel avant de tout effacer

6. GROUPER LES DEMANDES
   3 questions dans 1 message > 3 messages séparés (3× moins de relectures)

7. FEEDBACK EN LIGNE
   "Non, modifie seulement la ligne 42" > "c'est pas ça, recommence tout"

8. ÉVITER LES CONFIRMATIONS VIDES
   "ok", "oui", "continue" sans contenu = tokens perdus pour peu de valeur
   Préférer grouper avec la prochaine instruction vraiment utile

9. .claudeignore À JOUR
   Maintenir .claudeignore quand de nouveaux dossiers volumineux apparaissent

10. MÉMOIRE POUR CE QUI DURE
    Ne pas répéter d'une session à l'autre → sauvegarder en mémoire
```

### Calculer son efficacité

```
Score d'efficacité token = (Valeur produite) / (Tokens consommés)

Indicateurs positifs :
✓ Ratio messages/tokens élevé (beaucoup d'actions par message)
✓ Peu de corrections ("non refais") après une demande
✓ Fichiers lus une seule fois (pas de relectures inutiles)
✓ /compact déclenché avant saturation

Indicateurs négatifs :
✗ Beaucoup de "non c'est pas ça" (ambiguïtés non clarifiées en amont)
✗ Lire des fichiers volumineux non liés au problème
✗ Longues conversations sans /compact
✗ Messages de pure politesse sans contenu actionnable
```

---

## 18. Application aux Autres IAs

### ChatGPT / GPT-4o (OpenAI)

```
Similitudes avec Claude :
- Fenêtre de contexte : 128K tokens (GPT-4o)
- Prompt cache : oui (même mécanisme TTL ~5min en API)
- System prompt : paramètre `system`
- Lost in the Middle : même phénomène

Différences :
- Mémoire persistante intégrée dans l'interface (pas besoin de memory/)
- Custom GPTs = équivalent de CLAUDE.md + skills intégrés
- Pas de sous-agents natifs (sauf via Assistants API avec multi-threads)
- /compact n'existe pas → utiliser "résume la conversation" manuellement

Commandes équivalentes :
/compact → "Résume ce qu'on a fait en 200 mots, décisions et état actuel"
CLAUDE.md → System prompt du Custom GPT ou instructions personnalisées
```

### Google Gemini

```
Points forts :
- Contexte 1M tokens (Gemini 2.0 Flash) → moins de problème de saturation
- Intégration native Google Workspace (Docs, Sheets, Drive)
- Multimodal natif (vidéo, audio, images)

Points faibles :
- Pas d'équivalent CLAUDE.md natif
- Moins d'outils de gestion de contexte
- Interface moins adaptée au développement

Optimisations similaires :
- Grouper les questions (même économie de relectures)
- Filtrer les outputs de commandes
- Utiliser des conversations dédiées par sujet
```

### Modèles Locaux — Ollama / LM Studio / OpenCode

> [!tip] Avantage majeur du local
> **Zéro coût par token**. Les contraintes économiques disparaissent, mais les contraintes de performance restent.

```
┌──────────────────────────────────────────────────────┐
│              MODÈLES LOCAUX (Ollama, LM Studio)      │
│                                                      │
│  Avantages :                                         │
│  ✓ Gratuit à l'usage (coût = électricité + GPU)     │
│  ✓ Confidentialité totale (données ne quittent pas  │
│    votre machine)                                    │
│  ✓ Disponible hors ligne                            │
│  ✓ Pas de peak hours ni de rate limits              │
│                                                      │
│  Inconvénients :                                     │
│  ✗ Fenêtre contexte réduite (8K-32K selon le modèle)│
│  ✗ Qualité inférieure sur tâches complexes          │
│  ✗ Lent sans GPU puissant                           │
│  ✗ Pas d'outils natifs (pas de sub-agents, MCP      │
│    limité selon le client)                          │
└──────────────────────────────────────────────────────┘
```

### Setup Ollama

```bash
# Installation
curl -fsSL https://ollama.ai/install.sh | sh

# Télécharger un modèle
ollama pull llama3.3        # 8B params, bon équilibre
ollama pull codellama:34b   # Spécialisé code, meilleur que llama pour dev
ollama pull deepseek-coder-v2  # Excellent pour le code

# Lancer
ollama run codellama:34b

# API locale (compatible OpenAI)
curl http://localhost:11434/api/generate \
  -d '{"model": "codellama", "prompt": "Explique malloc en C"}'
```

### OpenCode — Claude Code avec modèles locaux

**OpenCode** est un client CLI open-source compatible avec Ollama et autres backends locaux, offrant une expérience similaire à Claude Code :

```bash
# Installation OpenCode
npm install -g @opencode-ai/opencode

# Configuration pour Ollama
opencode config set model ollama/codellama:34b
opencode config set base-url http://localhost:11434/v1
```

### Bonnes pratiques spécifiques aux modèles locaux

```
Adapter au contexte réduit (8K-32K tokens) :
1. Sessions encore plus courtes et ciblées
2. Fichiers courts (pas de longs fichiers 500+ lignes)
3. Résumés manuels fréquents
4. Découper les projets en micro-tâches indépendantes
5. Utiliser des modèles spécialisés (codellama pour le code,
   mistral pour la rédaction, etc.)

Modèles recommandés par cas d'usage (local) :
  Code général   : deepseek-coder-v2 ou codellama:34b
  Chat/rédaction : llama3.3:70b ou mistral-large
  Tâches rapides : llama3.2:3b ou phi4
  Multilingual   : mistral-nemo
```

---

## 19. Stratégies Combinées — Exemples Complets

### Scenario 1 : Nouveau projet (session initiale)

```
Étape 1 : Créer CLAUDE.md + .claudeignore avant tout
Étape 2 : Une session = "comprendre l'architecture"
           → Lire README, structure, fichiers clés
           → /compact après cette phase
Étape 3 : Sessions suivantes par module
           "aujourd'hui : module auth seulement"
```

### Scenario 2 : Debugging d'un bug difficile

```
1. Session dédiée au bug
2. Fournir : message d'erreur filtré (pas le log complet)
3. Fichier(s) concerné(s) précis (pas "tout le projet")
4. Hack 95% : "Ne propose aucun code tant que tu n'es pas
   sûr à 95% de la cause. Pose des questions d'abord."
5. Après résolution → /compact + note mémoire de la cause
```

### Scenario 3 : Génération massive (documentation, tests)

```
1. Utiliser HAIKU (pas Sonnet ni Opus)
2. Lancer en dehors des peak hours si possible
3. Sub-agents parallèles par module (si budget le permet)
4. Template dans CLAUDE.md pour le format attendu
5. Output par petits blocs (pas "génère les 50 tests d'un coup")
```

### Scenario 4 : Session longue (>3h de travail)

```
Toutes les ~45 min :
  - Vérifier le % de contexte utilisé (/context)
  - Si >60% → /compact
  - Si changement de sujet → résumé + /clear

En fin de journée :
  - Résumé structuré → sauvegarde mémoire
  - Note des décisions importantes
  - État des tâches en cours
```

---

## 20. Tableau de Référence Rapide

### Commandes et leur impact token

| Action | Impact tokens | Quand |
|--------|--------------|-------|
| /compact | Réduit contexte ×5-10 | À 60% de contexte |
| /clear | Remet à zéro | Changement de sujet |
| /context | Lecture seule | Vérifier l'état |
| CLAUDE.md | Économise répétitions | Toujours en place |
| .claudeignore | Évite exploration | Toujours en place |
| Sub-agents | ×2-3 coût, ×3 vitesse | Tâches indépendantes longues |
| Plan Mode | Évite corrections | Avant gros changements |
| Haiku | ×4-19× moins cher | Tâches répétitives |

### Checklist avant chaque session

```
□ CLAUDE.md à jour pour ce projet ?
□ .claudeignore configuré ?
□ Objectif de la session défini précisément ?
□ Bon modèle sélectionné (Haiku/Sonnet/Opus) ?
□ Sorties de commandes filtrées avant partage ?
□ Auto-compact configuré à 60% ?
□ Résumé de la session précédente disponible ?
```

---

## Carte Mentale ASCII

```
                    MAÎTRISE CLAUDE & LLM
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
      COMPRENDRE       CONFIGURER       OPTIMISER
           │               │               │
    ┌──────┴──────┐  ┌─────┴──────┐  ┌────┴──────┐
    ▼             ▼  ▼            ▼  ▼           ▼
 Tokens      Relecture  CLAUDE.md  .claudeignore  /compact  Modèle
 = $$$       exponen-   (règles    (exclusions)   /clear    adapté
             tielle     perma)                   /context
                │           │
         Lost in Middle  Memory/
         = milieu oublié   (persistance)
                │
         Prompt Cache
         TTL 5 minutes
         → rester actif
                │
    ┌───────────┼───────────┐
    ▼           ▼           ▼
 Sub-agents  Plan Mode   Skills
 (///)       (95%        (raccourcis)
 coûteux     confiance)
 mais rapide
                │
    ┌───────────┼────────────┐
    ▼           ▼            ▼
  Autres IAs  Local      Hygiène
  GPT/Gemini  Ollama/    contexte
              LM Studio  = sessions
                          ciblées
```

---

## Exercices Pratiques

### Exercice 1 — Configurer son environnement

Créer pour un projet existant :
1. Un `CLAUDE.md` de 200-400 tokens avec les règles du projet
2. Un `.claudeignore` excluant les dossiers générés
3. Comparer les tokens consommés sur 5 messages avant/après

### Exercice 2 — Mesurer la relecture exponentielle

1. Ouvrir une nouvelle session Claude Code
2. Envoyer 10 messages de ~100 mots chacun
3. Vérifier `/context` après chaque message
4. Calculer la progression réelle des tokens
5. Comparer avec la formule n×(n+1)/2

### Exercice 3 — Pratiquer le combo résumé + /clear

1. Travailler 30 min sur une tâche complexe
2. Demander un résumé structuré (décisions, état, prochaines étapes)
3. Faire /clear
4. Reprendre avec le résumé collé en premier message
5. Observer que Claude retrouve le contexte immédiatement

### Exercice 4 — Comparer Haiku vs Sonnet

Choisir une tâche répétitive (générer 10 tests unitaires similaires) :
1. Faire avec Haiku, noter qualité et coût estimé
2. Faire avec Sonnet, noter qualité et coût estimé
3. Décider si la différence de qualité justifie le prix pour cette tâche

---

## Liens

- [[01 - IA Assistants de Code]] — Présentation générale des assistants IA
- [[02 - IA et Productivite Dev]] — Intégration CI/CD, agents, génération de tests
- [[04 - CI-CD avec GitHub Actions]] — Automatiser avec des pipelines (context MCP)
- [[01 - Git et GitHub]] — Workflows Git avec assistance IA
```
