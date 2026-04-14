# 11 - IA, Confidentialité et Entreprise

Qu'est-ce que la confidentialité dans le contexte de l'IA ? C'est la question que tout développeur professionnel doit se poser avant d'envoyer la moindre ligne de code à un service cloud. Quand tu utilises ChatGPT, Claude ou Gemini pour déboguer un algorithme ou générer de la documentation, tu transmets des données à un tiers. Selon le service, le plan tarifaire et les contrats en vigueur, ces données peuvent être stockées, analysées, voire utilisées pour améliorer les modèles d'IA. Pour un projet personnel ou du code open source, l'impact est minimal. Pour du code propriétaire, des données clients ou des secrets commerciaux, les enjeux deviennent critiques.

Ce chapitre répond à une question simple mais fondamentale : **où vont mes données quand j'envoie du code à une IA cloud ?** Et surtout : que dois-je faire pour protéger mon entreprise, mes clients, et moi-même ?

> [!tip] Analogie
> Envoyer du code à une IA cloud, c'est comme dicter tes secrets à un sténographe dans un open space. Tu ne sais pas toujours qui prend des notes, qui lit par-dessus son épaule, combien de temps les notes sont conservées, ni si elles serviront à former d'autres sténographes. La différence avec un employé humain ? Au moins avec un humain, tu as signé un contrat de confidentialité. Avec une IA cloud, tout dépend des conditions générales d'utilisation — que personne ne lit jamais.

---

## 1. Le problème de la confidentialité avec l'IA

Lorsqu'un développeur envoie une requête à un modèle de langage cloud, plusieurs éléments sont en jeu :

```
┌──────────────────────────────────────────────────────────────┐
│                    FLUX D'UNE REQUÊTE IA                     │
│                                                              │
│  [TON POSTE]                                                 │
│      │  Code, prompt, contexte                               │
│      ▼                                                       │
│  [RÉSEAU INTERNET]  ──────────────────────────────────────►  │
│      │                                                       │
│      ▼                                                       │
│  [SERVEURS DU PROVIDER]                                      │
│      ├── Logs de la requête          ← Stocké combien ?      │
│      ├── Données pour fine-tuning    ← Utilisé comment ?     │
│      ├── Analyse de sécurité         ← Qui y accède ?        │
│      └── Réponse générée                                     │
│              │                                               │
│              ▼                                               │
│  [TON POSTE]  ◄──────────────────────────────────────────── │
└──────────────────────────────────────────────────────────────┘
```

Les trois questions essentielles à se poser :

1. **Mes données sont-elles utilisées pour entraîner le modèle ?** — Cela dépend du plan souscrit.
2. **Combien de temps sont-elles conservées ?** — De 0 à 30+ jours selon les providers.
3. **Qui peut y accéder ?** — Équipes internes du provider, autorités légales, partenaires ?

> [!warning]
> La plupart des développeurs utilisent le plan gratuit ou personnel d'une IA sans lire les conditions. Dans un contexte professionnel, c'est une faute potentielle : envers ton employeur, envers tes clients, et parfois envers la loi (RGPD, HIPAA, PCI-DSS).

---

## 2. Ce que font les providers cloud avec ton code

### Anthropic (Claude)

Anthropic propose plusieurs niveaux d'accès, avec des engagements de confidentialité très différents :

```
┌─────────────────────────────────────────────────────────────┐
│                    ANTHROPIC — NIVEAUX                      │
│                                                             │
│  Claude.ai GRATUIT                                          │
│  ├── Peut être utilisé pour améliorer les modèles           │
│  ├── Rétention : 30 jours (conversations)                   │
│  └── ⚠ Ne pas envoyer de code propriétaire                 │
│                                                             │
│  API / Teams (claude.ai/teams)                              │
│  ├── Anthropic s'engage à NE PAS entraîner sur tes données  │
│  ├── Rétention par défaut : 30 jours (configurable)         │
│  └── ✓ Acceptable pour code produit standard               │
│                                                             │
│  Enterprise                                                 │
│  ├── Garanties contractuelles + DPA (RGPD)                  │
│  ├── SSO, administration centralisée                        │
│  ├── Isolation des données                                  │
│  └── ✓ Pour données sensibles avec DPA signé               │
└─────────────────────────────────────────────────────────────┘
```

Politique complète : https://anthropic.com/privacy

> [!info]
> Anthropic est l'un des providers les plus transparents sur sa politique de données. La distinction entre plan gratuit et API est clairement documentée. Pour les équipes techniques, l'API reste le minimum requis en contexte professionnel.

### OpenAI (ChatGPT / API)

```
┌─────────────────────────────────────────────────────────────┐
│                    OPENAI — NIVEAUX                         │
│                                                             │
│  ChatGPT Gratuit / Plus                                     │
│  ├── Peut être utilisé pour l'entraînement                  │
│  ├── Opt-out disponible : Settings → Data controls          │
│  └── ⚠ Opt-out non activé par défaut                       │
│                                                             │
│  API (depuis mars 2023)                                     │
│  ├── NON utilisé pour entraîner les modèles par défaut      │
│  ├── Rétention : 30 jours pour détection d'abus            │
│  └── ✓ Acceptable pour usage professionnel standard        │
│                                                             │
│  ChatGPT Enterprise / Team                                  │
│  ├── Isolation des données, pas d'entraînement              │
│  ├── Admin console, SSO                                     │
│  └── ✓ Pour usage entreprise                               │
└─────────────────────────────────────────────────────────────┘
```

Pour désactiver l'utilisation de tes données pour l'entraînement sur ChatGPT.com :
`Settings → Data controls → Improve the model for everyone → OFF`

### Google (Gemini)

```
┌─────────────────────────────────────────────────────────────┐
│                    GOOGLE — NIVEAUX                         │
│                                                             │
│  Gemini Gratuit (gemini.google.com)                         │
│  ├── Données peuvent améliorer les produits Google          │
│  ├── Google AI Studio : review humain possible              │
│  └── ❌ À éviter pour tout usage professionnel              │
│                                                             │
│  Gemini for Google Workspace                                │
│  ├── Données non utilisées pour entraînement                │
│  ├── Isolation par workspace organisationnel                │
│  └── ✓ Si déjà abonné Google Workspace                    │
│                                                             │
│  Vertex AI                                                  │
│  ├── Contrats enterprise, isolation par projet GCP          │
│  ├── Certifications SOC2, HIPAA, FedRAMP                    │
│  └── ✓ Pour usage enterprise avec garanties légales        │
└─────────────────────────────────────────────────────────────┘
```

### Mistral AI

```
┌─────────────────────────────────────────────────────────────┐
│                    MISTRAL AI                               │
│                                                             │
│  Entreprise française → soumise au RGPD européen            │
│  Siège : Paris, France                                      │
│                                                             │
│  API Mistral                                                │
│  ├── Données non utilisées pour entraîner (conditions CGU)  │
│  ├── Hébergement en Europe                                  │
│  └── ✓ Avantage géopolitique pour entreprises EU           │
│                                                             │
│  Avantages pour les entreprises européennes :               │
│  ├── Pas de transfert de données hors UE                    │
│  ├── Conformité RGPD native                                 │
│  └── Pas de risque lié au Cloud Act américain              │
└─────────────────────────────────────────────────────────────┘
```

> [!info]
> Le **Cloud Act** américain (2018) permet aux autorités US d'accéder aux données stockées par des entreprises américaines, même sur des serveurs en Europe. Mistral étant une société française, elle n'y est pas soumise — un avantage concurrentiel réel pour les entreprises européennes sensibles à la souveraineté numérique.

### DeepSeek

> [!warning]
> DeepSeek est une entreprise chinoise. Ses données sont soumises aux lois chinoises, notamment la loi sur la sécurité nationale (2015) qui peut obliger les entreprises à coopérer avec les autorités. Les données envoyées à DeepSeek peuvent être potentiellement accessibles aux autorités chinoises.
>
> **NE PAS utiliser DeepSeek pour :**
> - Code propriétaire ou secrets commerciaux
> - Données clients (surtout EU/US)
> - Algorithmes sensibles
> - Tout projet soumis à réglementation (santé, finance, défense)
>
> Les modèles DeepSeek **téléchargés localement** via Ollama restent en revanche parfaitement acceptables — le modèle lui-même est open source.

---

## 3. Données à ne jamais envoyer à une IA cloud

Voici une classification des données à risque :

```
╔══════════════════════════════════════════════════════════════╗
║         DONNÉES INTERDITES — IA CLOUD NON ENTERPRISE        ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  SECRETS TECHNIQUES                                          ║
║  ❌ Clés API, tokens d'accès (même commentés dans le code)   ║
║  ❌ Mots de passe, certificats, clés SSH                     ║
║  ❌ Variables d'environnement contenant des credentials       ║
║                                                              ║
║  DONNÉES PERSONNELLES (RGPD)                                 ║
║  ❌ Noms, emails, adresses réels de clients                  ║
║  ❌ Numéros de sécurité sociale, CNI, passeports             ║
║  ❌ Données de localisation précises                         ║
║                                                              ║
║  DONNÉES SECTORIELLES RÉGLEMENTÉES                          ║
║  ❌ Données médicales (HIPAA, HDS en France)                 ║
║  ❌ Données bancaires, numéros de carte (PCI-DSS)            ║
║  ❌ Données financières non publiques (risque insider)        ║
║                                                              ║
║  PROPRIÉTÉ INTELLECTUELLE                                    ║
║  ❌ Algorithmes propriétaires ("secret sauce")               ║
║  ❌ Brevets non encore déposés                               ║
║  ❌ Code soumis à NDA client                                 ║
║                                                              ║
║  SÉCURITÉ                                                   ║
║  ❌ Vulnérabilités avant divulgation responsable             ║
║  ❌ Code de systèmes de défense ou sécurité nationale        ║
║  ❌ Configurations de sécurité d'infrastructure              ║
╚══════════════════════════════════════════════════════════════╝
```

> [!example]
> **Cas concret — ce qu'il ne faut PAS faire :**
> ```python
> # Développeur qui demande à Claude de déboguer cette fonction
> def connect_to_database():
>     return psycopg2.connect(
>         host="prod-db.monentreprise.com",
>         user="admin",
>         password="Sup3rS3cr3t!2024",  # ❌ credential réel
>         database="clients_eu"          # ❌ nom de base de prod
>     )
> ```
>
> **Ce qu'il faut faire à la place :**
> ```python
> # Version anonymisée — safe à envoyer à une IA
> def connect_to_database():
>     return psycopg2.connect(
>         host=os.environ["DB_HOST"],
>         user=os.environ["DB_USER"],
>         password=os.environ["DB_PASSWORD"],
>         database=os.environ["DB_NAME"]
>     )
> # Question à l'IA : "Pourquoi ma connexion psycopg2 timeout après 30s ?"
> ```

---

## 4. Le RGPD et l'IA : ce que tu dois savoir

Le **RGPD** (Règlement Général sur la Protection des Données, EU 2018) impose des obligations strictes dès lors que tu traites des données personnelles de résidents européens.

```
┌──────────────────────────────────────────────────────────────┐
│              RGPD ET IA — OBLIGATIONS CLÉS                   │
│                                                              │
│  ARTICLE 28 — Sous-traitants                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ Si tu utilises une IA cloud pour traiter des données   │  │
│  │ personnelles de clients EU, l'IA provider est un       │  │
│  │ "sous-traitant" au sens du RGPD.                       │  │
│  │                                                        │  │
│  │ Tu DOIS avoir un DPA (Data Processing Agreement)       │  │
│  │ signé avec ce provider.                                │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ARTICLE 46 — Transferts internationaux                      │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ Transférer des données EU vers des serveurs US         │  │
│  │ nécessite des garanties spécifiques :                  │  │
│  │ - Clauses contractuelles types (SCCs)                  │  │
│  │ - Certification EU-US Data Privacy Framework           │  │
│  │ - Binding Corporate Rules (BCRs)                       │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### Questions à poser à votre provider IA

Avant d'intégrer une IA cloud dans ton workflow professionnel, voici la checklist minimale :

```
CHECKLIST PROVIDER IA — CONFORMITÉ ENTREPRISE

□ Êtes-vous certifié SOC 2 Type II ?
  (audit indépendant de vos contrôles de sécurité)

□ Disposez-vous d'un DPA RGPD disponible à la signature ?
  (obligation légale si données EU traitées)

□ Où sont physiquement stockées les données ?
  (EU vs US vs autre — impact RGPD et Cloud Act)

□ Quelle est la durée de rétention des données ?
  (30 jours ? 6 mois ? Effacement sur demande ?)

□ Mes données sont-elles utilisées pour l'entraînement ?
  (non par défaut pour les API enterprise)

□ Qui peut accéder à mes données en interne ?
  (équipes de sécurité, support, développeurs ?)

□ Proposez-vous une région EU pour le déploiement ?
  (évite les transferts transatlantiques)

□ Avez-vous un processus de notification en cas de breach ?
  (RGPD : 72h pour notifier la CNIL)
```

> [!warning]
> La CNIL (Commission Nationale de l'Informatique et des Libertés) a publié en 2024 des recommandations spécifiques sur l'usage de l'IA en entreprise. Les sanctions RGPD peuvent atteindre **4% du chiffre d'affaires mondial** ou 20 millions d'euros. Ne pas traiter le sujet n'est pas une option légale.

---

## 5. Solutions cloud "enterprise-grade" (données isolées)

### Anthropic Enterprise

```
┌─────────────────────────────────────────────────────────────┐
│  ANTHROPIC ENTERPRISE                                       │
│                                                             │
│  Garanties :                                                │
│  ├── Isolation des données garantie contractuellement       │
│  ├── Pas d'utilisation pour l'entraînement                  │
│  ├── DPA RGPD disponible                                    │
│  ├── SSO (SAML, OIDC)                                       │
│  └── Administration centralisée des utilisateurs            │
│                                                             │
│  Tarification : sur devis (>$30/utilisateur/mois typique)   │
│  Idéal pour : entreprises tech, ESN, scale-ups              │
└─────────────────────────────────────────────────────────────┘
```

### Azure OpenAI Service

Azure OpenAI est probablement la solution enterprise la plus répandue en Europe, notamment pour les entreprises déjà dans l'écosystème Microsoft.

```
┌─────────────────────────────────────────────────────────────┐
│  AZURE OPENAI SERVICE                                       │
│                                                             │
│  Modèles disponibles : GPT-4o, o1, o3, DALL-E...           │
│                                                             │
│  Garanties :                                                │
│  ├── Tes données restent dans ton tenant Azure              │
│  ├── Ton abonnement, tes données — isolées des autres       │
│  ├── Pas d'utilisation pour entraîner les modèles OpenAI    │
│  ├── Déploiement dans ta région (France Central, EU...)     │
│  └── Certifications : SOC2, ISO27001, HIPAA, RGPD, HDS     │
│                                                             │
│  Idéal si :                                                 │
│  ├── Tu utilises déjà Azure (Active Directory, etc.)        │
│  ├── Ton entreprise est dans le secteur santé ou finance    │
│  └── Tu as besoin d'une conformité réglementaire étendue    │
└─────────────────────────────────────────────────────────────┘
```

```python
# Utilisation via Azure OpenAI (même API qu'OpenAI)
from openai import AzureOpenAI

client = AzureOpenAI(
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    api_key=os.environ["AZURE_OPENAI_API_KEY"],
    api_version="2024-02-01"
)

response = client.chat.completions.create(
    model="gpt-4o",  # Nom de ton déploiement Azure
    messages=[{"role": "user", "content": "Analyse ce code..."}]
)
```

### Google Vertex AI

```
┌─────────────────────────────────────────────────────────────┐
│  GOOGLE VERTEX AI                                           │
│                                                             │
│  Modèles : Gemini Pro/Ultra, Claude (via Anthropic),        │
│            Llama, Mistral, et modèles tiers                 │
│                                                             │
│  Garanties :                                                │
│  ├── Isolation par projet GCP                               │
│  ├── Données non utilisées pour améliorer les modèles       │
│  ├── Déploiement configurable en région EU                  │
│  └── Certifications : SOC2, HIPAA, FedRAMP, ISO27001        │
│                                                             │
│  Idéal si : déjà sur GCP, besoin de flexibilité modèles     │
└─────────────────────────────────────────────────────────────┘
```

### AWS Bedrock

```
┌─────────────────────────────────────────────────────────────┐
│  AWS BEDROCK                                                │
│                                                             │
│  Modèles : Claude (Anthropic), Llama (Meta), Mistral,       │
│            Titan (Amazon), Stable Diffusion...              │
│                                                             │
│  Garanties :                                                │
│  ├── Tes données restent dans ton VPC AWS                   │
│  ├── Pas d'utilisation pour l'entraînement des modèles      │
│  ├── Isolation réseau complète possible (PrivateLink)        │
│  └── Certifications : SOC2, HIPAA, PCI-DSS, FedRAMP        │
│                                                             │
│  Avantage unique : accès à Claude d'Anthropic avec          │
│  toutes les garanties de sécurité AWS                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Solutions auto-hébergées (100% privé)

Quand les garanties contractuelles ne suffisent pas — secteur défense, données ultra-sensibles, environnement air-gapped — l'auto-hébergement est la seule option.

### Ollama — La solution la plus simple

```bash
# Installation sur un serveur Linux interne
curl -fsSL https://ollama.com/install.sh | sh

# Démarrage en mode serveur accessible sur le réseau
OLLAMA_HOST=0.0.0.0:11434 ollama serve

# Téléchargement du modèle de code
ollama pull qwen2.5-coder:14b

# Test de la connexion depuis un autre poste
curl http://serveur-interne:11434/api/generate \
  -d '{"model": "qwen2.5-coder:14b", "prompt": "Write a Python hello world"}'
```

Configuration pour un plugin de code (Continue.dev) :

```json
{
  "models": [
    {
      "title": "Serveur IA Interne",
      "provider": "ollama",
      "model": "qwen2.5-coder:14b",
      "apiBase": "http://serveur-interne:11434"
    }
  ]
}
```

```
AVANTAGES OLLAMA AUTO-HÉBERGÉ

✓ 100% privé — zéro donnée externe
✓ Fonctionne sans internet
✓ Coût : 0€ de licensing (modèles open source)
✓ Simple à installer et maintenir

INCONVÉNIENTS

✗ Performance limitée par le hardware
✗ Modèles moins puissants que GPT-4o ou Claude 3.5 Sonnet
✗ Nécessite hardware dédié (GPU recommandé)
✗ Maintenance serveur à charge interne
```

### vLLM — Haute performance pour équipes

```bash
# Installation (serveur Linux avec GPU NVIDIA)
pip install vllm

# Démarrage du serveur d'inférence (API compatible OpenAI)
python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen2.5-Coder-14B-Instruct \
  --host 0.0.0.0 \
  --port 8000 \
  --tensor-parallel-size 2  # Pour 2 GPUs

# Utilisation depuis le code (API OpenAI compatible)
from openai import OpenAI

client = OpenAI(
    base_url="http://serveur-ia:8000/v1",
    api_key="token-interne-factice"  # Pas de vraie clé nécessaire
)
```

> [!info]
> vLLM est significativement plus performant qu'Ollama sur des serveurs disposant de GPUs dédiés. Il utilise la technique de **PagedAttention** pour optimiser l'utilisation de la mémoire GPU, permettant de servir plusieurs requêtes en parallèle. Idéal pour une équipe de 20+ développeurs utilisant l'IA intensivement.

### Tabby — Alternative auto-hébergée à GitHub Copilot

```bash
# Lancement avec Docker
docker run -it \
  --gpus all \
  -p 8080:8080 \
  -v /chemin/local/tabby:/data \
  tabbyml/tabby \
  serve --model TabbyML/DeepseekCoder-1.3B

# Interface web disponible sur : http://serveur:8080
# Plugin VS Code : TabbyML.vscode-tabby
```

```
TABBY — FONCTIONNALITÉS

├── Complétion de code en temps réel (comme Copilot)
├── Interface web d'administration
├── Gestion des utilisateurs et équipes
├── Support VS Code, JetBrains, Vim, Emacs
├── Modèles légers : DeepSeek-Coder 1.3B, 6.7B
└── Métriques d'utilisation par développeur

SITE : https://tabby.tabbyml.com
```

### LocalAI — Compatibilité maximale

```bash
# Démarrage avec Docker Compose
docker run -p 8080:8080 \
  -v /modeles:/build/models \
  localai/localai:latest \
  --models-path /build/models

# Compatible avec l'API OpenAI complète :
# - Chat completions
# - Embeddings
# - Text-to-speech
# - Image generation (Stable Diffusion)
```

> [!info]
> LocalAI supporte un très grand nombre de formats de modèles : GGUF, GPTQ, AWQ, etc. Plus complexe à configurer que Ollama, mais offre une flexibilité maximale et une compatibilité complète avec l'écosystème OpenAI.

---

## 7. Politique IA d'entreprise : bonnes pratiques

Utiliser l'IA en entreprise sans politique formelle, c'est exposer l'organisation à des risques légaux, réputationnels et concurrentiels. Une bonne politique IA doit être concise, pratique, et applicable.

### Classification des données

```
╔══════════════════════════════════════════════════════════════╗
║           SYSTÈME DE CLASSIFICATION — DONNÉES IA            ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  NIVEAU 0 — PUBLIC                                           ║
║  Peut être envoyé à n'importe quelle IA cloud                ║
║  ────────────────────────────────────────────────            ║
║  → Code open source sous licence libre                       ║
║  → Documentation publique                                    ║
║  → Questions génériques sur des technologies standards       ║
║                                                              ║
║  NIVEAU 1 — INTERNE                                          ║
║  IA cloud enterprise uniquement (avec DPA RGPD signé)        ║
║  ────────────────────────────────────────────────            ║
║  → Code produit standard sans données clients                ║
║  → Documentation interne non confidentielle                  ║
║  → Données pseudonymisées à faible risque                    ║
║                                                              ║
║  NIVEAU 2 — CONFIDENTIEL                                     ║
║  IA locale auto-hébergée uniquement                          ║
║  ────────────────────────────────────────────────            ║
║  → Algorithmes propriétaires et "secret sauce"               ║
║  → Données clients pseudonymisées à fort volume              ║
║  → Code soumis à NDA client                                  ║
║  → Informations sur l'architecture sécurité                  ║
║                                                              ║
║  NIVEAU 3 — SECRET                                           ║
║  Aucune IA externe — traitement manuel uniquement            ║
║  ────────────────────────────────────────────────            ║
║  → Clés de chiffrement et secrets cryptographiques           ║
║  → Données personnelles non pseudonymisées                   ║
║  → Code de systèmes défense/sécurité nationale              ║
║  → Informations sur vulnérabilités non divulguées            ║
╚══════════════════════════════════════════════════════════════╝
```

### Éléments d'une politique IA d'entreprise

```
TEMPLATE — POLITIQUE IA ENTREPRISE (éléments minimaux)

1. OUTILS APPROUVÉS
   ├── Liste blanche des services IA autorisés
   ├── Plan/niveau minimum requis (ex: "API uniquement")
   └── Processus d'approbation pour nouveaux outils

2. RÈGLES D'UTILISATION PAR NIVEAU
   ├── Quel niveau de données → quel outil
   └── Processus d'anonymisation si nécessaire

3. FORMATION DES EMPLOYÉS
   ├── Session onboarding "IA et confidentialité"
   ├── Documentation accessible (intranet/wiki)
   └── Mise à jour annuelle de la politique

4. AUDIT ET TRAÇABILITÉ
   ├── Logs d'utilisation des outils IA (qui utilise quoi)
   ├── Review trimestrielle des accès
   └── Processus d'incident si fuite de données

5. CONTACTS ET RESPONSABILITÉS
   ├── DPO (Data Protection Officer) — point de contact RGPD
   ├── RSSI — sécurité des systèmes
   └── Manager IA / AI Champion
```

---

## 8. Cas d'usage typiques par secteur

```
┌──────────────────────────────────────────────────────────────┐
│              RECOMMANDATIONS PAR SECTEUR                     │
├──────────────┬───────────────────────────────────────────────┤
│ STARTUP/PME  │ Claude API ou Azure OpenAI                    │
│              │ Contrat simple, coût raisonnable              │
│              │ DPA disponible, pas d'entraînement            │
├──────────────┼───────────────────────────────────────────────┤
│ ESN /        │ Vérifier les contrats clients en premier      │
│ PRESTATAIRE  │ Souvent : solution locale requise             │
│              │ Sinon : Azure OpenAI avec DPA client          │
├──────────────┼───────────────────────────────────────────────┤
│ BANQUE /     │ Azure OpenAI (région EU) ou Vertex AI         │
│ ASSURANCE    │ + vLLM interne pour données sensibles         │
│              │ Certifications DORA, NIS2, PCI-DSS requises   │
├──────────────┼───────────────────────────────────────────────┤
│ SANTÉ        │ Solution 100% locale (HIPAA, HDS France)      │
│              │ Ollama ou vLLM sur infrastructure dédiée      │
│              │ Audit annuel de conformité                    │
├──────────────┼───────────────────────────────────────────────┤
│ DÉFENSE      │ Exclusivement local                           │
│              │ Air-gapped si nécessaire                      │
│              │ Modèles validés par ANSSI                     │
├──────────────┼───────────────────────────────────────────────┤
│ OPEN SOURCE  │ Tout est possible — liberté totale            │
│              │ Préférer quand même l'API (données publiques) │
└──────────────┴───────────────────────────────────────────────┘
```

> [!example]
> **Cas concret — ESN en mission bancaire :**
> Un développeur travaille pour une ESN, en mission chez une banque, sur un système de scoring de crédit. Les règles s'empilent :
> 1. Le contrat de mission interdit l'envoi de code propriétaire à l'extérieur.
> 2. La banque est soumise à la directive DORA (résilience opérationnelle numérique).
> 3. Le scoring implique des données financières personnelles (RGPD Niveau 3).
>
> **Solution :** Ollama local sur poste de travail (sans réseau), ou serveur Ollama interne de la banque. Aucune donnée ne quitte le périmètre. Le développeur peut quand même bénéficier de l'assistance IA pour les parties génériques du code (algorithmes, syntaxe, patterns) sans exposer les données métier.

---

## 9. Sécurité des clés API

La fuite de clés API est l'une des causes les plus fréquentes de compromission de comptes cloud. Les conséquences : utilisation frauduleuse à tes frais, exposition de données, réputation détruite.

```bash
# ❌ JAMAIS dans le code source — même commenté
ANTHROPIC_API_KEY="sk-ant-api03-xxxx"  # Mauvais, même en commentaire

# ❌ JAMAIS dans les logs
print(f"Connecting with key: {api_key}")  # Mauvais

# ✅ Variables d'environnement (session courante)
export ANTHROPIC_API_KEY="sk-ant-api03-xxxx"
export OPENAI_API_KEY="sk-proj-xxxx"

# ✅ Fichier .env (développement local uniquement)
# Fichier .env
ANTHROPIC_API_KEY=sk-ant-api03-xxxx
OPENAI_API_KEY=sk-proj-xxxx

# ✅ S'assurer que .env n'est pas commité
echo ".env" >> .gitignore
echo "*.env" >> .gitignore

# Chargement en Python
from dotenv import load_dotenv
import os
load_dotenv()
api_key = os.environ["ANTHROPIC_API_KEY"]
```

```bash
# Vérification préventive avant commit
# Détecter les clés potentielles dans le code

# Avec git hooks (pre-commit)
# Installer : pip install pre-commit detect-secrets

# .pre-commit-config.yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

> [!warning]
> GitHub, GitLab et Bitbucket scannent automatiquement les commits publics pour détecter des clés API et les notifient aux providers (OpenAI, Anthropic, AWS...) qui peuvent révoquer les clés. Mais pour les repos privés ou internes, ce filet de sécurité n'existe pas. Un `git log` ne supprime pas les secrets : ils restent dans l'historique.
>
> Si tu as commité une clé par erreur : **révoque-la immédiatement** dans le dashboard du provider, puis nettoie l'historique git avec `git filter-repo`.

---

## 10. Arbre de décision : quelle IA utiliser ?

```
┌─────────────────────────────────────────────────────────────────┐
│         ARBRE DE DÉCISION — CHOIX DE L'IA                       │
│                                                                 │
│  Je veux utiliser l'IA pour...                                  │
│                │                                                │
│                ▼                                                │
│   ┌─── Code open source ou question générique ?                 │
│   │         OUI → N'importe quelle IA ✓                         │
│   │         NON ↓                                               │
│   │                                                             │
│   ├─── Code propriétaire, AUCUNE donnée client ?                │
│   │         OUI → Claude API / OpenAI API / Azure OpenAI ✓      │
│   │               (vérifier DPA si entreprise EU)               │
│   │         NON ↓                                               │
│   │                                                             │
│   ├─── Données clients EU (RGPD) ?                              │
│   │         OUI → Azure OpenAI (région EU) ou Vertex AI EU      │
│   │               + DPA RGPD signé obligatoire ✓                │
│   │               Pseudonymiser avant envoi si possible         │
│   │         NON ↓                                               │
│   │                                                             │
│   ├─── Données sensibles (santé, finance, NDA) ?                │
│   │         OUI → Solution auto-hébergée uniquement             │
│   │               Ollama ou vLLM sur infra interne ✓            │
│   │         NON ↓                                               │
│   │                                                             │
│   └─── Environnement air-gapped ou défense ?                    │
│             → Ollama local uniquement                           │
│               Sans connexion réseau externe ✓                   │
└─────────────────────────────────────────────────────────────────┘
```

### Tableau de synthèse rapide

```
┌────────────────────┬──────────────┬──────────────┬──────────────┐
│ SITUATION          │ CLOUD LIBRE  │ CLOUD ENTREP │ LOCAL UNIQU. │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Code open source   │      ✓       │      ✓       │      ✓       │
│ Code propriétaire  │      ✗       │      ✓       │      ✓       │
│ Données clients EU │      ✗       │   ✓ (+ DPA)  │      ✓       │
│ Données médicales  │      ✗       │      ✗       │      ✓       │
│ Données financières│      ✗       │   ✓ (+ DPA)  │      ✓       │
│ Secrets/credentials│      ✗       │      ✗       │      ✗ *     │
│ Air-gapped         │      ✗       │      ✗       │      ✓       │
└────────────────────┴──────────────┴──────────────┴──────────────┘
  * Même en local, ne jamais envoyer des credentials à un LLM
```

---

## Carte Mentale ASCII

```
                    ┌─────────────────────────┐
                    │  IA CONFIDENTIALITÉ &    │
                    │       ENTREPRISE         │
                    └───────────┬─────────────┘
                                │
         ┌──────────────────────┼──────────────────────┐
         │                      │                      │
    ┌────▼─────┐          ┌─────▼─────┐         ┌─────▼──────┐
    │PROVIDERS │          │   CLOUD   │          │   LOCAL    │
    │  CLOUD   │          │ENTERPRISE │          │  HOSTING   │
    └────┬─────┘          └─────┬─────┘         └─────┬──────┘
         │                      │                      │
    ┌────┴────┐            ┌────┴────┐            ┌────┴────┐
    │Anthropic│            │ Azure   │            │ Ollama  │
    │OpenAI   │            │OpenAI   │            │  vLLM   │
    │ Google  │            │Vertex AI│            │  Tabby  │
    │ Mistral │            │  AWS    │            │LocalAI  │
    │DeepSeek!│            │Bedrock  │            └────┬────┘
    └─────────┘            └────┬────┘                 │
                                │                 100% privé
                           DPA + SOC2            Air-gapped
                           Région EU              Hardware
                                                  requis

    ┌─────────────────────────────────────────────────────┐
    │                   DONNÉES                           │
    │  NIVEAU 0     NIVEAU 1     NIVEAU 2    NIVEAU 3     │
    │  Public   →  Interne   → Confidentiel → Secret      │
    │  Tout OK  →  API+DPA   →  Local seul  → Aucune IA  │
    └─────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────┐
    │                RÉGLEMENTATION                       │
    │  RGPD (EU) → DPA obligatoire avec sous-traitants    │
    │  HIPAA     → Santé US → local ou Azure HIPAA        │
    │  PCI-DSS   → Finance  → local ou certifié PCI       │
    │  Cloud Act → Données EU ≠ entreprise US             │
    └─────────────────────────────────────────────────────┘
```

---

## Exercices

### Exercice 1 — Audit de ton usage actuel

**Objectif :** Évaluer le niveau de risque de tes pratiques actuelles.

1. Liste tous les outils IA que tu utilises (ChatGPT, Claude, Copilot, etc.).
2. Pour chacun, identifie le plan souscrit (gratuit, Pro, API, Enterprise).
3. Rappelle-toi des 3 dernières choses significatives que tu as envoyées à chaque outil.
4. Classe-les selon la grille Niveau 0 à 3 du chapitre.
5. Identifie les écarts : as-tu envoyé des données Niveau 2 ou 3 à un outil non enterprise ?

**Livrable :** Un tableau personnel `Outil | Plan | Type données envoyées | Risque | Action corrective`.

---

### Exercice 2 — Mise en place d'un Ollama d'équipe

**Objectif :** Déployer un serveur IA privé accessible à une petite équipe.

```bash
# Étape 1 : Installation sur un serveur Linux (ou VM locale)
curl -fsSL https://ollama.com/install.sh | sh

# Étape 2 : Téléchargement d'un modèle de code
ollama pull qwen2.5-coder:7b  # 4-5 Go, bon compromis

# Étape 3 : Démarrage en mode serveur
OLLAMA_HOST=0.0.0.0:11434 ollama serve &

# Étape 4 : Test depuis un autre poste du réseau
curl http://[IP-SERVEUR]:11434/api/generate \
  -d '{"model":"qwen2.5-coder:7b","prompt":"Write a bubble sort in Python","stream":false}'

# Étape 5 : Configurer Continue.dev sur chaque poste
# Modifier ~/.continue/config.json avec l'URL du serveur
```

**Questions à résoudre :**
- Comment sécuriser l'accès au serveur Ollama (authentification réseau) ?
- Quel modèle choisir pour un GPU avec 8 Go de VRAM ?
- Comment mettre à jour les modèles sans interruption de service ?

---

### Exercice 3 — Rédiger une politique IA pour ton équipe

**Objectif :** Créer un document de politique IA applicable immédiatement.

En t'inspirant du template du chapitre, rédige une politique IA pour ton équipe ou projet actuel. Le document doit contenir :

1. **Contexte** : quel type de projet, quels types de données manipulées.
2. **Classification des données du projet** : applique les niveaux 0-3.
3. **Outils approuvés** : liste précise avec le plan/niveau minimum requis.
4. **Ce qui est interdit** : liste explicite des données à ne jamais envoyer.
5. **Processus en cas d'incident** : que faire si on suspecte une fuite ?

**Contrainte :** Le document doit tenir en une page A4. Plus c'est concis, plus il sera lu et appliqué.

---

## Liens connexes

- [[09 - IA Locale avec Ollama]]
- [[10 - LM Studio et Hardware Local]]
- [[06 - Gemini Mistral et Alternatives Cloud]]
- [[12 - Strategies Multi-Modeles et Workflows]]
- [[08 - Prompting Avance pour Developpeurs]]
