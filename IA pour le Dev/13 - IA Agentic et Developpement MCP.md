# 13 - IA Agentic et Développement MCP

> [!info] L'ère des agents IA pour les développeurs
> Les LLMs répondent à des questions. Les **agents** prennent des actions. Cette distinction, anodine en apparence, change radicalement ce qu'il est possible de construire. Ce cours couvre les architectures agentiques, les principaux SDKs (OpenAI Agents, Google ADK), le protocole MCP, et les patterns multi-agents — avec du code concret pour chaque concept. Pour les stratégies de sélection de modèles, voir [[12 - Strategies Multi-Modeles et Workflows|Stratégies Multi-Modèles]].

---

## Partie 1 — Qu'est-ce qu'un agent IA ?

### La boucle perceive → reason → act

Un **agent IA** est un système capable d'**agir dans le monde** pour atteindre un objectif, en combinant un modèle de langage avec des outils (tools) qui lui permettent de lire, écrire, appeler des APIs, exécuter du code.

```
┌──────────────────────────────────────────────────────────────────┐
│                    BOUCLE AGENTIQUE                              │
│                                                                  │
│  PERCEIVE (Percevoir)                                            │
│  ┌─────────────────────────────────────┐                        │
│  │ Contexte entrant :                  │                        │
│  │  - Objectif de l'utilisateur        │                        │
│  │  - Résultats des actions précédentes│                        │
│  │  - État de la mémoire               │                        │
│  │  - Observations de l'environnement  │                        │
│  └──────────────────┬──────────────────┘                        │
│                     │                                            │
│  REASON (Raisonner)  ▼                                           │
│  ┌─────────────────────────────────────┐                        │
│  │ LLM détermine :                     │                        │
│  │  - L'état actuel de la tâche        │                        │
│  │  - La prochaine action à effectuer  │                        │
│  │  - Quel outil appeler (et avec quoi)│                        │
│  │  - Si l'objectif est atteint        │                        │
│  └──────────────────┬──────────────────┘                        │
│                     │                                            │
│  ACT (Agir)         ▼                                           │
│  ┌─────────────────────────────────────┐                        │
│  │ Exécution :                         │                        │
│  │  - Appeler un outil (API, BDD, code)│                        │
│  │  - Retourner une réponse finale     │                        │
│  │  - Handoff vers un autre agent      │                        │
│  └──────────────────┬──────────────────┘                        │
│                     │                                            │
│                     └──────── retour à PERCEIVE ◄───────────────┘
└──────────────────────────────────────────────────────────────────┘
```

### LLM vs Agent — différences fondamentales

La compréhension du fonctionnement des [[02 - Comprendre les LLMs et les Tokens|LLMs et des tokens]] est un prérequis pour bien dimensionner les fenêtres de contexte des agents.

| Dimension | LLM seul | Agent IA |
|-----------|----------|----------|
| **Durée de vie** | Une requête, une réponse | Multi-tours, boucle autonome |
| **Actions** | Générer du texte | Appeler des outils, modifier des fichiers, faire des requêtes HTTP |
| **Mémoire** | Context window limitée | Mémoire persistante (BDD, fichiers, vector stores) |
| **Décision** | Statique | Dynamique, raisonnement sur les résultats des actions |
| **Parallélisme** | Non | Peut déléguer à des sous-agents en parallèle |
| **Exemple** | "Explique le quicksort" | "Relis tous les PRs ouverts, identifie les bugs critiques, commente chacun" |

### Taxonomie des agents

```
AGENTS SIMPLES
  - ReAct Agent (Reason + Act) : LLM qui alterne raisonnement et actions
  - Tool-use Agent : LLM avec accès à une liste de fonctions
  
AGENTS STRUCTURÉS
  - Sequential : étapes prédéfinies, l'agent les exécute dans l'ordre
  - Parallel : plusieurs agents sur des tâches indépendantes simultanément
  - Loop : l'agent itère jusqu'à satisfaire un critère
  
MULTI-AGENTS
  - Orchestrateur + Workers : un chef coordonne des agents spécialisés
  - Pipeline : la sortie d'un agent est l'entrée du suivant
  - Debate : plusieurs agents proposent, critiquent, convergent vers une réponse
  - Swarm : agents autonomes qui s'auto-organisent (émergent)
```

---

## Partie 2 — OpenAI Agents SDK

Pour les workflows avec l'API OpenAI (ChatGPT, Codex), voir aussi [[05 - ChatGPT Codex et GitHub Copilot|ChatGPT, Codex et GitHub Copilot]]. La rédaction de prompts efficaces pour guider les agents est couverte dans [[03 - Prompt Engineering pour le Code|Prompt Engineering pour le Code]].

### Installation et concepts de base

```bash
pip install openai-agents
```

**Concepts clés du SDK :**
- `Agent` : un agent défini par son nom, ses instructions, ses outils et son modèle
- `Tool` : une fonction Python exposée à l'agent
- `Handoff` : transfert de contrôle d'un agent à un autre
- `Runner` : moteur d'exécution qui gère la boucle agentique
- `Trace` : enregistrement de toutes les étapes pour debug/observabilité

### Premier agent avec outils

```python
import asyncio
import httpx
from agents import Agent, Runner, tool

# 1. Définir les outils (fonctions que l'agent peut appeler)
@tool
def obtenir_meteo(ville: str) -> str:
    """
    Retourne la météo actuelle pour une ville donnée.
    
    Args:
        ville: Nom de la ville (ex: "Paris", "Lyon")
    Returns:
        Description de la météo actuelle
    """
    # Simulation — en production, appeler une vraie API météo
    meteos = {
        "paris": "Nuageux, 15°C, humidité 72%",
        "lyon": "Ensoleillé, 22°C, humidité 45%",
        "marseille": "Partiellement nuageux, 26°C, humidité 58%"
    }
    return meteos.get(ville.lower(), f"Météo pour {ville} indisponible")

@tool
def convertir_devise(montant: float, devise_source: str, devise_cible: str) -> str:
    """
    Convertit un montant entre deux devises.
    
    Args:
        montant: Le montant à convertir
        devise_source: Devise d'origine (EUR, USD, GBP...)
        devise_cible: Devise cible
    Returns:
        Résultat de la conversion
    """
    # Taux fictifs pour l'exemple
    taux = {
        ("EUR", "USD"): 1.08,
        ("USD", "EUR"): 0.93,
        ("EUR", "GBP"): 0.86,
        ("GBP", "EUR"): 1.16,
    }
    cle = (devise_source.upper(), devise_cible.upper())
    if cle in taux:
        resultat = montant * taux[cle]
        return f"{montant} {devise_source} = {resultat:.2f} {devise_cible}"
    return f"Conversion {devise_source} → {devise_cible} non disponible"

# 2. Définir l'agent
agent_assistant = Agent(
    name="AssistantVoyage",
    instructions="""
    Tu es un assistant de voyage expert. Tu aides les utilisateurs à planifier
    leurs voyages en leur fournissant des informations météo et en convertissant
    les devises.
    
    Utilise les outils disponibles quand nécessaire. Si plusieurs informations
    sont demandées, récupère-les toutes avant de rédiger une réponse synthétique.
    Sois concis et pratique.
    """,
    model="gpt-4o-mini",
    tools=[obtenir_meteo, convertir_devise]
)

# 3. Exécuter l'agent
async def main():
    result = await Runner.run(
        agent_assistant,
        input="Je pars à Lyon demain avec 500€. Quel temps fera-t-il et combien ça fait en USD ?"
    )
    print(result.final_output)

if __name__ == "__main__":
    asyncio.run(main())

# Sortie (exemple) :
# À Lyon demain, vous aurez du soleil avec 22°C et une faible humidité (45%).
# Concernant votre budget : 500€ = 540.00 USD.
# Profitez de votre séjour !
```

### Handoffs — transfert entre agents spécialisés

Les handoffs permettent à un agent généraliste de transférer vers un agent spécialisé selon la nature de la demande.

```python
from agents import Agent, Runner, handoff

# Agents spécialisés
agent_technique = Agent(
    name="SupportTechnique",
    instructions="""
    Tu es un expert technique qui résout les problèmes d'API, de code et d'intégration.
    Pose des questions précises pour diagnostiquer le problème.
    Fournis toujours des exemples de code.
    """,
    model="gpt-4o",    # modèle plus puissant pour les tâches techniques
)

agent_facturation = Agent(
    name="SupportFacturation",
    instructions="""
    Tu es un expert en facturation. Tu traites les remboursements, les changements
    d'abonnement et les problèmes de paiement.
    Sois empathique et professionnel.
    """,
    model="gpt-4o-mini",
)

# Agent de triage avec handoffs
agent_triage = Agent(
    name="Triage",
    instructions="""
    Tu es l'agent de triage du support client. Ton rôle est UNIQUEMENT de
    comprendre la nature du problème et de transférer au bon agent spécialisé.
    
    - Problème technique (API, code, bug) → SupportTechnique
    - Problème de paiement/facture → SupportFacturation
    - Autre : réponds toi-même brièvement
    
    Ne tente JAMAIS de résoudre un problème technique ou de facturation toi-même.
    """,
    model="gpt-4o-mini",
    handoffs=[
        handoff(agent_technique, tool_name="transfer_to_technical_support"),
        handoff(agent_facturation, tool_name="transfer_to_billing_support"),
    ]
)

async def support_client(question: str):
    result = await Runner.run(agent_triage, input=question)
    print(f"Agent final : {result.last_agent.name}")
    print(f"Réponse : {result.final_output}")

# Exemples :
asyncio.run(support_client("Mon appel à l'API retourne une erreur 401 malgré un token valide"))
# Agent final : SupportTechnique
# Réponse : "L'erreur 401 avec un token valide peut indiquer..."

asyncio.run(support_client("J'ai été prélevé deux fois ce mois-ci"))
# Agent final : SupportFacturation
# Réponse : "Je suis désolé pour cette erreur de prélèvement..."
```

### Tracing et observabilité

```python
from agents import Agent, Runner
from agents.tracing import TracingConfig

# Activer le tracing dans OpenAI Platform
agent = Agent(
    name="MonAgent",
    instructions="...",
    model="gpt-4o-mini",
    tools=[...],
)

result = await Runner.run(
    agent,
    input="Ma question",
    run_config=TracingConfig(
        trace_id="session-unique-123",
        workflow_name="MonWorkflow",
        # Les traces apparaissent sur platform.openai.com/traces
    )
)

# Accéder aux détails d'exécution
for step in result.steps:
    print(f"Étape : {step.type}")
    if hasattr(step, 'tool_calls'):
        for call in step.tool_calls:
            print(f"  Outil appelé : {call.name}({call.arguments})")
            print(f"  Résultat : {call.result}")
```

---

## Partie 3 — Google Agent Development Kit (ADK)

### Vue d'ensemble

Le **Google ADK** (Agent Development Kit) est un framework Python pour construire des agents et orchestrations multi-agents, orienté vers la production et l'intégration avec l'écosystème Google (Vertex AI, Gemini).

```bash
pip install google-adk
```

### Agent simple avec ADK

```python
from google.adk.agents import Agent
from google.adk.tools import google_search
from google.adk.sessions import InMemorySessionService
from google.adk.runners import Runner

# Définir l'agent avec outils
agent_recherche = Agent(
    name="AgentRecherche",
    model="gemini-2.0-flash",          # ou "gemini-1.5-pro" pour plus de capacités
    instruction="""
    Tu es un assistant de recherche. Utilise l'outil de recherche Google
    pour trouver des informations récentes et précises.
    Cite toujours tes sources et indique la date de l'information.
    """,
    tools=[google_search],             # outil de recherche intégré à ADK
)

# Gestionnaire de sessions (persistance entre messages)
session_service = InMemorySessionService()

# Runner d'exécution
runner = Runner(
    agent=agent_recherche,
    app_name="MonAppRecherche",
    session_service=session_service,
)

import asyncio
from google.adk.types import Content, Part

async def rechercher(question: str, user_id: str = "user_1", session_id: str = "session_1"):
    """Envoyer une question à l'agent."""
    message = Content(role="user", parts=[Part(text=question)])

    reponse_finale = None
    async for event in runner.run_async(
        user_id=user_id,
        session_id=session_id,
        new_message=message,
    ):
        if event.is_final_response():
            reponse_finale = event.content.parts[0].text

    return reponse_finale

# Utilisation
asyncio.run(rechercher("Quelles sont les dernières avancées en IA générative en 2025 ?"))
```

### Outils personnalisés avec ADK

```python
from google.adk.tools import FunctionTool
import json

def analyser_json(contenu_json: str) -> dict:
    """
    Analyse et valide une chaîne JSON.
    
    Args:
        contenu_json: La chaîne JSON à analyser
    Returns:
        dict avec 'valide' (bool), 'donnees' (si valide) et 'erreur' (si invalide)
    """
    try:
        donnees = json.loads(contenu_json)
        return {
            "valide": True,
            "donnees": donnees,
            "nb_cles": len(donnees) if isinstance(donnees, dict) else len(donnees),
        }
    except json.JSONDecodeError as e:
        return {
            "valide": False,
            "erreur": str(e),
            "position": e.pos,
        }

def executer_python(code: str) -> dict:
    """
    Exécute du code Python dans un sandbox et retourne le résultat.
    
    Args:
        code: Le code Python à exécuter (max 50 lignes)
    Returns:
        dict avec 'stdout', 'stderr', 'succes'
    """
    import subprocess
    import sys
    
    # ATTENTION : en production, utiliser un sandbox sécurisé (Docker, Pyodide, etc.)
    # Cette implémentation est UNIQUEMENT illustrative
    result = subprocess.run(
        [sys.executable, "-c", code],
        capture_output=True,
        text=True,
        timeout=10,
    )
    return {
        "stdout": result.stdout,
        "stderr": result.stderr,
        "succes": result.returncode == 0,
    }

# Wrapper ADK pour les fonctions
outil_json = FunctionTool(func=analyser_json)
outil_python = FunctionTool(func=executer_python)

agent_dev = Agent(
    name="AgentDev",
    model="gemini-2.0-flash",
    instruction="Tu es un assistant de développement. Tu peux analyser du JSON et exécuter du Python.",
    tools=[outil_json, outil_python],
)
```

### Orchestration multi-agents avec ADK

```python
from google.adk.agents import Agent, LoopAgent, SequentialAgent, ParallelAgent

# Agents feuilles (workers spécialisés)
agent_linter = Agent(
    name="Linter",
    model="gemini-2.0-flash",
    instruction="Analyse le code fourni et identifie les problèmes de style et de qualité.",
    tools=[outil_python],
)

agent_securite = Agent(
    name="SecuriteCode",
    model="gemini-1.5-pro",           # modèle plus puissant pour l'analyse de sécurité
    instruction="Analyse le code pour des vulnérabilités de sécurité (injection, XSS, SSRF...).",
)

agent_tests = Agent(
    name="GenerateurTests",
    model="gemini-2.0-flash",
    instruction="Génère des tests unitaires Python (pytest) pour le code fourni.",
)

# Analyse en parallèle (linting ET sécurité simultanément)
agent_analyse_parallele = ParallelAgent(
    name="AnalyseParallele",
    sub_agents=[agent_linter, agent_securite],
)

# Pipeline séquentiel : analyse → puis génération de tests
pipeline_review = SequentialAgent(
    name="PipelineCodeReview",
    sub_agents=[
        agent_analyse_parallele,    # étape 1 : analyse parallèle
        agent_tests,                # étape 2 : générer les tests (avec le contexte des problèmes trouvés)
    ],
)

# Utilisation du pipeline
async def reviewer_code(code: str):
    message = Content(role="user", parts=[Part(text=f"Analyse ce code :\n```python\n{code}\n```")])
    runner = Runner(agent=pipeline_review, app_name="CodeReview", session_service=InMemorySessionService())
    
    async for event in runner.run_async(user_id="dev1", session_id="review1", new_message=message):
        if event.is_final_response():
            print(event.content.parts[0].text)
```

---

## Partie 4 — Model Context Protocol (MCP)

### Architecture MCP

Le **Model Context Protocol** (MCP) est un protocole open-source développé par Anthropic (novembre 2024) qui standardise la façon dont les modèles d'IA accèdent aux données et aux outils externes. [[04 - Claude et Claude Code Maitrise Avancee|Claude Code]] implémente MCP nativement et peut consommer des serveurs MCP locaux ou distants via `claude mcp add`.

```
┌───────────────────────────────────────────────────────────────┐
│                    ARCHITECTURE MCP                           │
│                                                               │
│  ┌─────────────┐     MCP Protocol      ┌───────────────────┐ │
│  │    HOST     │◄─────────────────────►│   MCP SERVER      │ │
│  │ (Claude,    │                        │                   │ │
│  │  Cursor,    │  ┌──────────────────┐  │  - Tools          │ │
│  │  Cline...)  │  │   MCP CLIENT     │  │  - Resources      │ │
│  │             │  │  (intégré dans   │  │  - Prompts        │ │
│  │  LLM        │  │   le host)       │  │                   │ │
│  │  context    │  └──────────────────┘  │  Votre code       │ │
│  └─────────────┘                        └───────────────────┘ │
│                                                               │
│  Transports :                                                 │
│    stdio   : processus local (Claude Desktop, Cline)         │
│    HTTP SSE: serveur distant (accès réseau)                  │
└───────────────────────────────────────────────────────────────┘
```

**3 primitives MCP :**

| Primitive | Rôle | Exemple |
|-----------|------|---------|
| **Tool** | Fonction que le LLM peut appeler | `create_file()`, `run_query()`, `send_email()` |
| **Resource** | Données statiques ou dynamiques | Contenu d'un fichier, résultat d'une requête SQL, documentation |
| **Prompt** | Templates de prompts réutilisables | `generate_pr_description`, `summarize_document` |

### Créer un serveur MCP en Python

```bash
pip install mcp
```

**Exemple 1 : Serveur MCP qui lit une base de données**

```python
# server_bdd.py
import asyncio
import sqlite3
from typing import Any
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import (
    Tool, TextContent, Resource, ResourceContents,
    INVALID_PARAMS, INTERNAL_ERROR
)
import mcp.types as types

# Initialiser le serveur MCP
app = Server("serveur-base-donnees")

# Connexion BDD (SQLite pour l'exemple, adaptable à PostgreSQL/MySQL)
DB_PATH = "ma_base.db"

def get_db_connection():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row    # retourne des dict plutôt que des tuples
    return conn

# ─── TOOLS ────────────────────────────────────────────────────────────────

@app.list_tools()
async def lister_outils() -> list[Tool]:
    """Déclarer les outils disponibles au client MCP."""
    return [
        Tool(
            name="executer_requete",
            description="Exécute une requête SQL SELECT en lecture seule et retourne les résultats",
            inputSchema={
                "type": "object",
                "properties": {
                    "requete": {
                        "type": "string",
                        "description": "La requête SQL SELECT à exécuter"
                    },
                    "limite": {
                        "type": "integer",
                        "description": "Nombre maximum de lignes à retourner (défaut: 100)",
                        "default": 100
                    }
                },
                "required": ["requete"]
            }
        ),
        Tool(
            name="lister_tables",
            description="Liste toutes les tables disponibles dans la base de données",
            inputSchema={
                "type": "object",
                "properties": {}
            }
        ),
        Tool(
            name="schema_table",
            description="Retourne le schéma (colonnes et types) d'une table spécifique",
            inputSchema={
                "type": "object",
                "properties": {
                    "nom_table": {
                        "type": "string",
                        "description": "Nom de la table dont on veut le schéma"
                    }
                },
                "required": ["nom_table"]
            }
        ),
    ]

@app.call_tool()
async def appeler_outil(name: str, arguments: dict[str, Any]) -> list[TextContent]:
    """Exécuter un outil quand le LLM le demande."""

    if name == "lister_tables":
        conn = get_db_connection()
        cursor = conn.execute("SELECT name FROM sqlite_master WHERE type='table' ORDER BY name")
        tables = [row["name"] for row in cursor.fetchall()]
        conn.close()
        return [TextContent(type="text", text=f"Tables disponibles : {', '.join(tables)}")]

    elif name == "schema_table":
        nom_table = arguments.get("nom_table")
        if not nom_table:
            raise ValueError("nom_table requis")

        conn = get_db_connection()
        cursor = conn.execute(f"PRAGMA table_info({nom_table})")
        colonnes = cursor.fetchall()
        conn.close()

        if not colonnes:
            return [TextContent(type="text", text=f"Table '{nom_table}' introuvable")]

        schema = "\n".join(
            f"  {col['name']} {col['type']} {'NOT NULL' if col['notnull'] else 'NULL'}"
            for col in colonnes
        )
        return [TextContent(type="text", text=f"Schéma de {nom_table}:\n{schema}")]

    elif name == "executer_requete":
        requete = arguments.get("requete", "")
        limite = arguments.get("limite", 100)

        # Sécurité : uniquement les SELECT
        requete_clean = requete.strip().upper()
        if not requete_clean.startswith("SELECT"):
            raise ValueError("Seules les requêtes SELECT sont autorisées")

        conn = get_db_connection()
        cursor = conn.execute(requete + f" LIMIT {limite}")
        lignes = cursor.fetchall()
        conn.close()

        if not lignes:
            return [TextContent(type="text", text="Aucun résultat")]

        # Formater en tableau
        colonnes = lignes[0].keys()
        header = " | ".join(colonnes)
        separator = "-" * len(header)
        rows = "\n".join(" | ".join(str(row[col]) for col in colonnes) for row in lignes)
        return [TextContent(type="text", text=f"{header}\n{separator}\n{rows}\n\n({len(lignes)} lignes)")]

    else:
        raise ValueError(f"Outil inconnu : {name}")

# ─── RESOURCES ────────────────────────────────────────────────────────────

@app.list_resources()
async def lister_ressources() -> list[Resource]:
    """Exposer des données statiques ou dynamiques comme ressources."""
    conn = get_db_connection()
    cursor = conn.execute("SELECT name FROM sqlite_master WHERE type='table'")
    tables = [row["name"] for row in cursor.fetchall()]
    conn.close()

    return [
        Resource(
            uri=f"bdd://tables/{table}",
            name=f"Table: {table}",
            description=f"Contenu de la table {table}",
            mimeType="application/json"
        )
        for table in tables
    ]

@app.read_resource()
async def lire_ressource(uri: str) -> ResourceContents:
    """Retourner le contenu d'une ressource."""
    if uri.startswith("bdd://tables/"):
        nom_table = uri.replace("bdd://tables/", "")
        conn = get_db_connection()
        cursor = conn.execute(f"SELECT * FROM {nom_table} LIMIT 50")
        lignes = [dict(row) for row in cursor.fetchall()]
        conn.close()
        import json
        return ResourceContents(
            uri=uri,
            mimeType="application/json",
            text=json.dumps(lignes, ensure_ascii=False, indent=2)
        )
    raise ValueError(f"Ressource inconnue : {uri}")

# ─── DÉMARRAGE ────────────────────────────────────────────────────────────

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

**Exemple 2 : Serveur MCP qui appelle une API externe**

```python
# server_api_github.py
import asyncio
import httpx
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
import os

app = Server("serveur-github")

GITHUB_TOKEN = os.environ.get("GITHUB_TOKEN", "")
GITHUB_BASE_URL = "https://api.github.com"

async def appel_github(endpoint: str) -> dict:
    """Wrapper pour les appels à l'API GitHub."""
    async with httpx.AsyncClient() as client:
        response = await client.get(
            f"{GITHUB_BASE_URL}{endpoint}",
            headers={
                "Authorization": f"Bearer {GITHUB_TOKEN}",
                "Accept": "application/vnd.github.v3+json",
            },
            timeout=30.0
        )
        response.raise_for_status()
        return response.json()

@app.list_tools()
async def lister_outils() -> list[Tool]:
    return [
        Tool(
            name="lister_prs_ouvertes",
            description="Liste les Pull Requests ouvertes d'un dépôt GitHub",
            inputSchema={
                "type": "object",
                "properties": {
                    "owner": {"type": "string", "description": "Propriétaire du dépôt (user ou org)"},
                    "repo": {"type": "string", "description": "Nom du dépôt"},
                },
                "required": ["owner", "repo"]
            }
        ),
        Tool(
            name="lire_diff_pr",
            description="Lit le diff d'une Pull Request spécifique",
            inputSchema={
                "type": "object",
                "properties": {
                    "owner": {"type": "string"},
                    "repo": {"type": "string"},
                    "pr_number": {"type": "integer", "description": "Numéro de la PR"},
                },
                "required": ["owner", "repo", "pr_number"]
            }
        ),
        Tool(
            name="commenter_pr",
            description="Ajoute un commentaire à une Pull Request",
            inputSchema={
                "type": "object",
                "properties": {
                    "owner": {"type": "string"},
                    "repo": {"type": "string"},
                    "pr_number": {"type": "integer"},
                    "commentaire": {"type": "string", "description": "Texte du commentaire"},
                },
                "required": ["owner", "repo", "pr_number", "commentaire"]
            }
        ),
    ]

@app.call_tool()
async def appeler_outil(name: str, arguments: dict) -> list[TextContent]:
    owner = arguments.get("owner")
    repo = arguments.get("repo")

    if name == "lister_prs_ouvertes":
        prs = await appel_github(f"/repos/{owner}/{repo}/pulls?state=open&per_page=20")
        if not prs:
            return [TextContent(type="text", text="Aucune PR ouverte")]

        liste = "\n".join(
            f"PR #{pr['number']}: {pr['title']} (@{pr['user']['login']}) — {pr['html_url']}"
            for pr in prs
        )
        return [TextContent(type="text", text=f"PRs ouvertes pour {owner}/{repo}:\n{liste}")]

    elif name == "lire_diff_pr":
        pr_number = arguments.get("pr_number")
        # Récupère les fichiers modifiés
        fichiers = await appel_github(f"/repos/{owner}/{repo}/pulls/{pr_number}/files")
        details = [
            f"Fichier: {f['filename']} (+{f['additions']} -{f['deletions']})\n{f.get('patch', 'patch non disponible')[:500]}"
            for f in fichiers[:5]  # limite aux 5 premiers fichiers
        ]
        return [TextContent(type="text", text="\n\n".join(details))]

    elif name == "commenter_pr":
        pr_number = arguments.get("pr_number")
        commentaire = arguments.get("commentaire")
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{GITHUB_BASE_URL}/repos/{owner}/{repo}/issues/{pr_number}/comments",
                headers={"Authorization": f"Bearer {GITHUB_TOKEN}"},
                json={"body": commentaire},
                timeout=30.0
            )
            response.raise_for_status()
        return [TextContent(type="text", text=f"Commentaire ajouté à PR #{pr_number}")]

    raise ValueError(f"Outil inconnu : {name}")

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

### Intégrer un serveur MCP dans [[04 - Claude et Claude Code Maitrise Avancee|Claude Code]], Cursor, Cline

**Claude Desktop / [[04 - Claude et Claude Code Maitrise Avancee|Claude Code]] — configuration `claude_desktop_config.json` :**

```json
{
    "mcpServers": {
        "base-donnees": {
            "command": "python",
            "args": ["/chemin/vers/server_bdd.py"],
            "env": {
                "PYTHONPATH": "/chemin/vers/venv/lib/python3.11/site-packages"
            }
        },
        "github": {
            "command": "python",
            "args": ["/chemin/vers/server_api_github.py"],
            "env": {
                "GITHUB_TOKEN": "ghp_votre_token_ici"
            }
        }
    }
}
```

**Claude Code — via `claude mcp add` :**

```bash
# Ajouter un serveur MCP local (stdio)
claude mcp add ma-base-de-donnees python /chemin/vers/server_bdd.py

# Ajouter avec des variables d'environnement
claude mcp add github-tools \
    --env GITHUB_TOKEN=ghp_xxx \
    python /chemin/vers/server_api_github.py

# Lister les serveurs configurés
claude mcp list

# Tester un serveur MCP
claude mcp dev /chemin/vers/server_bdd.py
```

**Cursor — `.cursor/mcp.json` à la racine du projet :**

```json
{
    "mcpServers": {
        "base-donnees": {
            "command": "python",
            "args": ["./tools/server_bdd.py"]
        }
    }
}
```

---

## Partie 5 — Architectures Multi-Agents

Pour choisir quel modèle affecter à quel agent dans une architecture multi-agents (Haiku pour les tâches simples, Sonnet/Opus pour la relecture), voir [[12 - Strategies Multi-Modeles et Workflows|Stratégies Multi-Modèles et Workflows]].

### Orchestrateur + Agents spécialisés

```python
# Pattern : un orchestrateur délègue à des workers spécialisés
# Exemple : pipeline de traitement d'articles de blog

from anthropic import Anthropic

client = Anthropic()

def appeler_claude(system: str, user: str, modele: str = "claude-haiku-4-5") -> str:
    """Appel Claude simple pour un agent."""
    response = client.messages.create(
        model=modele,
        max_tokens=2000,
        messages=[{"role": "user", "content": user}],
        system=system,
    )
    return response.content[0].text

# ─── Workers spécialisés ─────────────────────────────────────

def agent_recherche_seo(sujet: str) -> str:
    """Agent qui suggère des mots-clés SEO."""
    return appeler_claude(
        system="Tu es un expert SEO. Pour le sujet donné, liste 10 mots-clés à forte intention et leur volume de recherche estimé.",
        user=f"Sujet : {sujet}"
    )

def agent_plan_article(sujet: str, mots_cles: str) -> str:
    """Agent qui crée le plan de l'article."""
    return appeler_claude(
        system="Tu es un rédacteur expert. Crée un plan détaillé en H2/H3 pour un article de blog.",
        user=f"Sujet : {sujet}\nMots-clés cibles : {mots_cles}",
        modele="claude-sonnet-4-5"
    )

def agent_rediger_section(titre_section: str, contexte: str) -> str:
    """Agent qui rédige une section spécifique."""
    return appeler_claude(
        system="Tu es un rédacteur web expert. Rédige la section demandée en 300-400 mots, avec exemples concrets.",
        user=f"Section à rédiger : {titre_section}\nContexte de l'article : {contexte}"
    )

def agent_relire_final(article_complet: str) -> str:
    """Agent qui relit et améliore l'article final."""
    return appeler_claude(
        system="Tu es un éditeur senior. Améliore le texte pour la cohérence, la fluidité et l'optimisation SEO. Corrige les erreurs.",
        user=article_complet,
        modele="claude-opus-4-5"    # meilleur modèle pour la relecture finale
    )

# ─── Orchestrateur ────────────────────────────────────────────

def orchestrateur_blog(sujet: str) -> str:
    """Orchestrateur qui coordonne tous les agents."""
    print(f"[Orchestrateur] Démarrage pour : {sujet}")

    # Étape 1 : SEO
    print("[Orchestrateur] → Agent SEO...")
    mots_cles = agent_recherche_seo(sujet)

    # Étape 2 : Plan
    print("[Orchestrateur] → Agent Plan...")
    plan = agent_plan_article(sujet, mots_cles)

    # Étape 3 : Rédiger les sections (on simule l'extraction des H2 du plan)
    # En production : parser le plan pour extraire les sections
    sections_simulees = ["Introduction", "Partie principale", "Conclusion"]
    sections_redigees = []
    for section in sections_simulees:
        print(f"[Orchestrateur] → Rédaction section : {section}...")
        contenu = agent_rediger_section(section, f"Article sur : {sujet}\nPlan : {plan[:200]}")
        sections_redigees.append(f"## {section}\n\n{contenu}")

    # Étape 4 : Assemblage
    article_brut = "\n\n".join(sections_redigees)

    # Étape 5 : Relecture finale
    print("[Orchestrateur] → Agent Relecture finale...")
    article_final = agent_relire_final(article_brut)

    print("[Orchestrateur] Article terminé.")
    return article_final

# Utilisation
article = orchestrateur_blog("Comment optimiser les performances d'une API Python")
print(article)
```

### LangGraph — Graphe d'états pour agents complexes

LangGraph permet de construire des workflows d'agents où les transitions entre états sont conditionnelles.

```bash
pip install langgraph langchain-anthropic
```

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages
from langchain_anthropic import ChatAnthropic
from langchain_core.messages import HumanMessage, AIMessage, ToolMessage
from langchain_core.tools import tool

# ─── Définition de l'état du graphe ─────────────────────────

class EtatAgent(TypedDict):
    messages: Annotated[list, add_messages]    # historique des messages
    iterations: int                             # compteur d'itérations
    solution_trouvee: bool                      # critère d'arrêt

# ─── Outils ─────────────────────────────────────────────────

@tool
def rechercher_docs(query: str) -> str:
    """Recherche dans la documentation technique."""
    # Simulation — en production : RAG avec vector store
    return f"Documentation trouvée pour '{query}': [résultats simulés...]"

@tool
def verifier_code(code: str) -> str:
    """Vérifie si le code Python est syntaxiquement correct."""
    import ast
    try:
        ast.parse(code)
        return "✓ Syntaxe correcte"
    except SyntaxError as e:
        return f"✗ Erreur syntaxe : {e}"

outils = [rechercher_docs, verifier_code]

# ─── LLM configuré avec les outils ──────────────────────────

llm = ChatAnthropic(model="claude-haiku-4-5").bind_tools(outils)

# ─── Noeuds du graphe ────────────────────────────────────────

def noeud_llm(state: EtatAgent) -> EtatAgent:
    """Noeud LLM : raisonnement et décision."""
    response = llm.invoke(state["messages"])
    return {
        "messages": [response],
        "iterations": state["iterations"] + 1,
        "solution_trouvee": state["solution_trouvee"],
    }

def noeud_outils(state: EtatAgent) -> EtatAgent:
    """Noeud d'exécution des outils appelés par le LLM."""
    from langgraph.prebuilt import ToolNode
    tool_node = ToolNode(outils)
    return tool_node.invoke(state)

# ─── Routing conditionnel ────────────────────────────────────

def router(state: EtatAgent) -> str:
    """Décide quelle est la prochaine étape."""
    dernier_message = state["messages"][-1]

    # Critère d'arrêt par nombre d'itérations
    if state["iterations"] >= 10:
        return "fin"

    # Si le LLM a appelé des outils → les exécuter
    if hasattr(dernier_message, "tool_calls") and dernier_message.tool_calls:
        return "executer_outils"

    # Sinon → réponse finale
    return "fin"

# ─── Construction du graphe ──────────────────────────────────

builder = StateGraph(EtatAgent)

builder.add_node("llm", noeud_llm)
builder.add_node("outils", noeud_outils)

builder.set_entry_point("llm")
builder.add_conditional_edges(
    "llm",
    router,
    {
        "executer_outils": "outils",
        "fin": END,
    }
)
builder.add_edge("outils", "llm")    # après les outils → retour au LLM

graphe = builder.compile()

# ─── Exécution ───────────────────────────────────────────────

etat_initial = {
    "messages": [HumanMessage(content="Comment implémenter un cache LRU en Python ?")],
    "iterations": 0,
    "solution_trouvee": False,
}

for event in graphe.stream(etat_initial):
    for node_name, node_output in event.items():
        print(f"[{node_name}] {len(node_output.get('messages', []))} message(s)")
```

### CrewAI — Équipes d'agents orientées tâches

```bash
pip install crewai crewai-tools
```

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool, WebsiteSearchTool

# ─── Agents de la "crew" ─────────────────────────────────────

agent_chercheur = Agent(
    role="Chercheur Principal",
    goal="Trouver les informations les plus récentes et pertinentes sur le sujet",
    backstory="""
    Expert en recherche documentaire avec 10 ans d'expérience dans la veille
    technologique. Rigoureux, cite toujours ses sources.
    """,
    tools=[SerperDevTool()],   # outil de recherche Google
    verbose=True,
    allow_delegation=False,
    llm="claude-haiku-4-5"
)

agent_analyste = Agent(
    role="Analyste de Données",
    goal="Analyser les informations collectées et identifier les tendances clés",
    backstory="""
    Data analyst senior spécialisé dans la synthèse d'informations complexes.
    Transforme les données brutes en insights actionnables.
    """,
    verbose=True,
    allow_delegation=False,
    llm="claude-sonnet-4-5"
)

agent_redacteur = Agent(
    role="Rédacteur Expert",
    goal="Transformer l'analyse en un rapport clair et engageant",
    backstory="""
    Journaliste technique avec une expertise en vulgarisation. Adapte le niveau
    de complexité à l'audience cible.
    """,
    verbose=True,
    allow_delegation=True,    # peut déléguer des sous-tâches aux autres agents
    llm="claude-sonnet-4-5"
)

# ─── Tâches ─────────────────────────────────────────────────

tache_recherche = Task(
    description="""
    Recherche les dernières avancées sur {sujet}.
    Collecte au minimum 5 sources primaires (articles, études, annonces officielles).
    Documente : source, date, résumé en 2 phrases, niveau de pertinence (1-5).
    """,
    expected_output="Un tableau structuré de 5-10 sources avec résumés et évaluations",
    agent=agent_chercheur,
)

tache_analyse = Task(
    description="""
    Sur la base des sources collectées, identifie :
    1. Les 3 tendances majeures
    2. Les points de consensus entre les sources
    3. Les contradictions ou débats actifs
    4. Les implications pratiques pour les développeurs
    """,
    expected_output="Une analyse structurée en 4 sections avec exemples concrets",
    agent=agent_analyste,
    context=[tache_recherche],    # dépend des résultats de la tâche précédente
)

tache_rapport = Task(
    description="""
    Rédige un rapport de 800-1000 mots destiné à des développeurs seniors.
    Le rapport doit être structuré avec : introduction, corps (basé sur l'analyse),
    recommandations pratiques, conclusion.
    Ton : professionnel mais accessible, avec des exemples de code si pertinent.
    """,
    expected_output="Un rapport complet en Markdown prêt à publier",
    agent=agent_redacteur,
    context=[tache_recherche, tache_analyse],
)

# ─── Crew (l'équipe) ─────────────────────────────────────────

crew = Crew(
    agents=[agent_chercheur, agent_analyste, agent_redacteur],
    tasks=[tache_recherche, tache_analyse, tache_rapport],
    process=Process.sequential,    # ou Process.hierarchical pour un manager agent
    verbose=2,
)

# ─── Exécution ───────────────────────────────────────────────

resultat = crew.kickoff(
    inputs={"sujet": "Model Context Protocol et agents IA en 2025"}
)
print(resultat)
```

---

## Partie 6 — Auditer et corriger le code généré par IA

### Patterns d'erreurs courants dans le code généré par IA

**1. Hallucinations d'API — l'IA invente des fonctions qui n'existent pas**

```python
# ❌ Code halluciné — cette méthode n'existe pas dans anthropic SDK
response = client.messages.create_stream_with_tools(...)

# ✓ Méthode réelle
with client.messages.stream(...) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

**Comment détecter** : Toujours vérifier les noms de méthodes dans la documentation officielle. Ne jamais faire confiance à un nom de méthode sans l'avoir vu dans les docs.

**2. Problèmes de sécurité — injections, secrets en dur**

```python
# ❌ Code généré par IA avec problèmes de sécurité
import sqlite3

def get_user(username: str):
    conn = sqlite3.connect("users.db")
    # INJECTION SQL — l'IA a généré du code vulnérable
    query = f"SELECT * FROM users WHERE username = '{username}'"
    result = conn.execute(query).fetchone()
    conn.close()
    return result

# Appel malveillant : get_user("admin' OR '1'='1")
# → Retourne TOUS les utilisateurs !

# ✓ Version sécurisée avec paramètres
def get_user_safe(username: str):
    conn = sqlite3.connect("users.db")
    result = conn.execute(
        "SELECT * FROM users WHERE username = ?",
        (username,)                    # paramètre bind — jamais d'interpolation de chaîne
    ).fetchone()
    conn.close()
    return result
```

**3. Logique incorrecte — subtile et difficile à détecter**

```python
# ❌ Bug logique : l'IA a inversé la condition de pagination
def paginer_resultats(items: list, page: int, taille_page: int) -> list:
    debut = page * taille_page
    fin = debut + taille_page
    # BUG : items[fin:debut] retourne toujours une liste vide !
    return items[fin:debut]

# ✓ Correct
def paginer_resultats_correct(items: list, page: int, taille_page: int) -> list:
    debut = page * taille_page
    fin = debut + taille_page
    return items[debut:fin]

# Test immédiat pour détecter ce type de bug :
items = list(range(100))
assert paginer_resultats(items, 0, 10) == list(range(10))   # FAIL → bug détecté
```

### Checklist de revue du code généré par IA

```
✅ CHECKLIST DE REVUE CODE IA

SÉCURITÉ
□ Aucune injection SQL/NoSQL (chercher les f-strings dans les requêtes)
□ Aucun secret hardcodé (API keys, passwords, tokens)
□ Validation des entrées (type, longueur, format)
□ Gestion des erreurs explicite (pas de except pass silencieux)
□ Pas d'exécution de code arbitraire (eval, exec, subprocess avec input utilisateur)

EXACTITUDE
□ Vérifier CHAQUE appel d'API/bibliothèque dans la documentation officielle
□ Tester les cas limites (liste vide, None, valeurs max/min)
□ Vérifier la logique de pagination, indexation, slicing (off-by-one)
□ Valider les formules mathématiques manuellement

PERFORMANCE
□ Pas de N+1 queries (boucle sur des appels BDD)
□ Pas d'allocation dans les hot paths
□ Complexité algorithmique acceptable pour la volumétrie attendue

MAINTIENABILITÉ
□ Les noms de variables/fonctions sont explicites
□ Pas de code mort ou de TODOs sans ticket
□ Les dépendances importées sont toutes utilisées
□ Les types annotations sont présents et corrects (Python: mypy)

TESTS
□ Écrire un test pour chaque fonction non triviale
□ Tester au minimum le chemin heureux et un cas d'erreur
□ Les mocks sont corrects (ne pas mocker ce qu'on ne possède pas)
```

---

## Partie 7 — Télémétrie des LLMs

### Métriques clés à surveiller

| Métrique | Définition | Seuil d'alerte |
|----------|------------|----------------|
| **Latence p50/p99** | Temps de réponse médian / 99e percentile | p99 > 30s pour Sonnet |
| **Tokens/seconde** | Débit de génération | < 20 tokens/s = lent |
| **Coût par appel** | $cost = (input_tokens × prix_input) + (output_tokens × prix_output) | Budget mensuel |
| **Cache hit rate** | % d'appels utilisant le prompt cache | < 30% = optimiser le cache |
| **Taux d'erreur** | % d'appels retournant une erreur | > 1% = problème |
| **Token utilization** | input_tokens / context_window_size | > 80% = risque de troncature |

### LangSmith — Observabilité LLM

```bash
pip install langsmith langchain-anthropic
```

```python
import os
from langsmith import traceable, Client
from langchain_anthropic import ChatAnthropic
from langchain_core.messages import HumanMessage

# Configuration LangSmith
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "lsv2_votre_clé"
os.environ["LANGCHAIN_PROJECT"] = "mon-agent-projet"

llm = ChatAnthropic(model="claude-haiku-4-5")

# Le décorateur @traceable trace automatiquement chaque appel
@traceable(name="analyse-code", tags=["code-review", "production"])
def analyser_code(code: str, contexte: str) -> str:
    """Analyse un morceau de code et retourne des suggestions."""
    response = llm.invoke([
        HumanMessage(content=f"""
        Contexte : {contexte}
        
        Analyse ce code et identifie les problèmes :
        ```python
        {code}
        ```
        """)
    ])
    return response.content

# Toutes les métriques (latence, tokens, coût) sont automatiquement
# envoyées vers LangSmith : smith.langchain.com
resultat = analyser_code(
    code="def f(l): return [x for x in l if x%2==0]",
    contexte="Fonction de filtrage des nombres pairs"
)
```

**Dashboard LangSmith** — métriques disponibles :
- Latence par run (p50, p95, p99)
- Tokens utilisés (input/output/total)
- Coût estimé par run et par projet
- Taux de succès/échec
- Feedbacks utilisateurs (thumbs up/down)
- Traces complètes pour debug

### Compteur de tokens côté client (Anthropic SDK)

```python
from anthropic import Anthropic

client = Anthropic()

class TelemetrieAgent:
    """Wrapper qui collecte les métriques de chaque appel."""

    def __init__(self):
        self.total_input_tokens = 0
        self.total_output_tokens = 0
        self.total_cache_read_tokens = 0
        self.nb_appels = 0
        self.latences_ms = []

    def appeler(self, model: str, messages: list, system: str = "") -> str:
        import time

        debut = time.perf_counter()
        response = client.messages.create(
            model=model,
            max_tokens=2000,
            system=system,
            messages=messages,
        )
        latence_ms = (time.perf_counter() - debut) * 1000

        # Collecter les métriques
        self.nb_appels += 1
        self.total_input_tokens += response.usage.input_tokens
        self.total_output_tokens += response.usage.output_tokens
        cache_tokens = getattr(response.usage, "cache_read_input_tokens", 0)
        self.total_cache_read_tokens += cache_tokens
        self.latences_ms.append(latence_ms)

        return response.content[0].text

    def rapport(self):
        if not self.latences_ms:
            return "Aucun appel effectué"

        latences_triees = sorted(self.latences_ms)
        p50 = latences_triees[len(latences_triees) // 2]
        p99 = latences_triees[int(len(latences_triees) * 0.99)]

        # Prix Claude Haiku 4.5 (mai 2025, indicatif)
        cout_input = self.total_input_tokens * 0.80 / 1_000_000    # $0.80/MTok
        cout_output = self.total_output_tokens * 4.00 / 1_000_000   # $4.00/MTok
        cout_total = cout_input + cout_output

        return f"""
Télémétrie Agent
═══════════════
Appels          : {self.nb_appels}
Tokens input    : {self.total_input_tokens:,}
Tokens output   : {self.total_output_tokens:,}
Cache hits      : {self.total_cache_read_tokens:,}
Cache hit rate  : {self.total_cache_read_tokens / max(self.total_input_tokens, 1) * 100:.1f}%

Latence p50     : {p50:.0f} ms
Latence p99     : {p99:.0f} ms

Coût estimé     : ${cout_total:.4f}
        """

# Utilisation
telemetrie = TelemetrieAgent()
for question in ["Qu'est-ce que MCP ?", "Comment créer un tool MCP ?", "Qu'est-ce que CrewAI ?"]:
    reponse = telemetrie.appeler(
        model="claude-haiku-4-5",
        messages=[{"role": "user", "content": question}],
        system="Tu es un expert en agents IA. Réponds en 2-3 phrases maximum."
    )
    print(f"Q: {question}\nR: {reponse}\n")

print(telemetrie.rapport())
```

---

## Partie 8 — Exemples complets

### Agent de code review automatique

```python
# agent_code_review.py
# Un agent qui ouvre une PR GitHub, lit le diff, et commente chaque problème

import asyncio
from agents import Agent, Runner, tool
import httpx
import os

GITHUB_TOKEN = os.environ["GITHUB_TOKEN"]
ANTHROPIC_REPO = "owner/repo"

@tool
def lire_diff_pr(pr_number: int) -> str:
    """Lit le diff complet d'une Pull Request."""
    response = httpx.get(
        f"https://api.github.com/repos/{ANTHROPIC_REPO}/pulls/{pr_number}/files",
        headers={"Authorization": f"Bearer {GITHUB_TOKEN}"},
    )
    fichiers = response.json()
    return "\n\n".join(
        f"### {f['filename']}\n```diff\n{f.get('patch', 'pas de patch')}\n```"
        for f in fichiers
    )

@tool
def poster_commentaire(pr_number: int, commentaire: str) -> str:
    """Poste un commentaire de review sur la PR."""
    response = httpx.post(
        f"https://api.github.com/repos/{ANTHROPIC_REPO}/issues/{pr_number}/comments",
        headers={"Authorization": f"Bearer {GITHUB_TOKEN}"},
        json={"body": commentaire}
    )
    if response.status_code == 201:
        return f"Commentaire posté avec succès : {response.json()['html_url']}"
    return f"Erreur : {response.text}"

agent_reviewer = Agent(
    name="CodeReviewer",
    instructions="""
    Tu es un code reviewer expert senior. Ton rôle est de :
    1. Lire le diff de la PR indiquée
    2. Identifier les problèmes : bugs potentiels, sécurité, performance, lisibilité
    3. Poster un commentaire structuré sur la PR avec :
       - Un résumé en 2-3 phrases
       - Une liste des problèmes détectés (sévérité: CRITIQUE/IMPORTANT/SUGGESTION)
       - Des suggestions de correction avec exemples de code
    
    Sois constructif et précis. Jamais de commentaires vagues.
    Si la PR ne présente pas de problème majeur, le dire clairement.
    """,
    model="claude-sonnet-4-5",
    tools=[lire_diff_pr, poster_commentaire]
)

async def reviewer_pr(pr_number: int):
    result = await Runner.run(
        agent_reviewer,
        input=f"Lis et commente la Pull Request #{pr_number}"
    )
    return result.final_output

asyncio.run(reviewer_pr(42))
```

### Pipeline [[04 - CI-CD avec GitHub Actions|CI/CD]] avec agent IA

Le workflow ci-dessous s'intègre dans une pipeline [[04 - CI-CD avec GitHub Actions|GitHub Actions]] pour automatiser la revue de code sur chaque Pull Request. Conteneuriser l'agent dans [[01 - Docker|Docker]] garantit la reproductibilité de l'environnement Python entre les runners.

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review

on:
    pull_request:
        types: [opened, synchronize]

jobs:
    ai-review:
        runs-on: ubuntu-latest
        permissions:
            pull-requests: write
            contents: read

        steps:
        - uses: actions/checkout@v4
          with:
              fetch-depth: 0

        - name: Set up Python
          uses: actions/setup-python@v5
          with:
              python-version: "3.11"

        - name: Install dependencies
          run: pip install anthropic httpx

        - name: Run AI Review
          env:
              ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
              GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
              PR_NUMBER: ${{ github.event.number }}
              REPO: ${{ github.repository }}
          run: python agent_code_review.py
```

---

## Exercices pratiques

### Exercice 1 — Premier serveur MCP (60 min)

Un serveur MCP est essentiellement une [[09 - APIs REST avec FastAPI|API FastAPI]] ou un processus stdio qui expose des outils au LLM. Créez un serveur MCP Python qui expose ces 3 outils :
1. `lire_fichier(chemin: str)` — lit le contenu d'un fichier texte local
2. `lister_dossier(chemin: str)` — liste les fichiers d'un répertoire
3. `compter_lignes(chemin: str)` — compte les lignes d'un fichier

Connectez-le à Claude Code et vérifiez qu'il fonctionne en demandant à Claude de lire un fichier de votre choix.

### Exercice 2 — Agent ReAct avec LangGraph (90 min)

Implémentez un agent ReAct (Reason + Act) avec LangGraph qui :
- Peut effectuer des calculs mathématiques (outil `calculer`)
- Peut convertir des unités (outil `convertir_unite`)
- Peut répondre à des questions en plusieurs étapes

Exemple de requête multi-étapes : "Si une voiture roule à 90 km/h pendant 2.5 heures, quelle distance parcourt-elle en miles ?"

### Exercice 3 — Agent de génération de tests (90 min)

Construisez un agent (avec n'importe quel SDK : OpenAI Agents, ADK, ou directement avec l'API Anthropic) qui :
1. Prend un fichier Python en entrée
2. Analyse les fonctions présentes
3. Génère des tests pytest pour chacune (cas nominal + cas limites)
4. Vérifie que les tests générés sont syntaxiquement valides
5. Écrit les tests dans un fichier `test_[nom_fichier].py`

### Exercice 4 — Audit de code IA (30 min)

Le code suivant a été généré par une IA. Trouvez tous les problèmes (sécurité, logique, API, performance) :

```python
import requests
import sqlite3

API_KEY = "sk-proj-abc123def456ghi789"  # Clé API hardcodée

def rechercher_utilisateurs(nom: str, base: str = "users.db"):
    conn = sqlite3.connect(base)
    users = conn.execute(f"SELECT * FROM users WHERE name LIKE '%{nom}%'")
    return users.fetchall()

def enrichir_avec_ia(users: list) -> list:
    enrichis = []
    for user in users:  # N+1 appels API !
        response = requests.post(
            "https://api.openai.com/v1/chat/completions",
            headers={"Authorization": f"Bearer {API_KEY}"},
            json={
                "model": "gpt-4-turbo-preview",  # Modèle retiré
                "messages": [{"role": "user", "content": f"Analyse cet utilisateur: {user}"}]
            }
        )
        enrichis.append(response.json()["choices"][0]["message"])
    return enrichis

def main():
    try:
        users = rechercher_utilisateurs(input("Nom à rechercher: "))
        enriched = enrichir_avec_ia(users)
        print(enriched)
    except:
        pass  # Ignorer toutes les erreurs
```

Listez chaque problème, sa catégorie (sécurité/logique/performance/maintenabilité), et proposez la correction.

### Exercice 5 — Télémétrie MCP (45 min)

Modifiez le serveur MCP de l'exercice 1 pour :
1. Logger chaque appel d'outil avec timestamp et durée d'exécution
2. Comptabiliser le nombre d'appels par outil
3. Exposer ces statistiques comme une resource MCP (`stats://usage`)
4. Tester depuis Claude Code en demandant les statistiques d'utilisation

---

> [!info] Pour aller plus loin
> - [[04 - Claude et Claude Code Maitrise Avancee]] — prompt engineering avancé et hooks Claude Code
> - [[12 - Strategies Multi-Modeles et Workflows]] — choisir le bon modèle pour chaque tâche agentique
> - Documentation officielle MCP : modelcontextprotocol.io
> - OpenAI Agents SDK : openai.github.io/openai-agents-python
> - Google ADK : google.github.io/adk-docs
> - LangGraph : langchain-ai.github.io/langgraph
> - CrewAI : docs.crewai.com
> - Livre : *Building LLM Powered Applications* — Valentina Alto (Packt, 2024)

---

## Notes liées

- [[04 - Claude et Claude Code Maitrise Avancee]] — Claude Code, MCP, hooks, prompt engineering avancé
- [[12 - Strategies Multi-Modeles et Workflows]] — orchestration multi-modèles, choix Haiku/Sonnet/Opus par tâche
- [[02 - Comprendre les LLMs et les Tokens]] — tokenisation, fenêtre de contexte, prompt caching
- [[03 - Prompt Engineering pour le Code]] — techniques de prompting pour guider les agents
- [[05 - ChatGPT Codex et GitHub Copilot]] — API OpenAI, GPT-4o, outils Codex
- [[09 - APIs REST avec FastAPI]] — construire les serveurs MCP avec FastAPI (HTTP SSE)
- [[07 - Python Async et Concurrence]] — asyncio, async/await Python pour les agents asynchrones
- [[01 - Docker]] — conteneurisation des agents et serveurs MCP pour la CI
- [[04 - CI-CD avec GitHub Actions]] — pipelines CI avec agents IA intégrés
