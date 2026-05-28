# Index - Cours de Programmation

> [!info] Guide complet
> Ce coffre contient des cours complets pour apprendre la programmation depuis zéro. Chaque note est conçue pour qu'un novice puisse, après lecture, écrire et mettre en pratique toutes les connaissances abordées.
> 
> **156 notes** | **~165 000 lignes** | Tout en français

---

## Trimestre 1 — Fondations (C, Système, Outils)

### Fondamentaux Informatique

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Computational Thinking et Architecture]] | CPU, OS, Scratch, pseudo-code, pensée algorithmique |
| 2 | [[02 - Green IT et Optimisation]] | Impact CO2, Green C, Green Web, éco-conception |

### Langage C

| # | Cours | Description |
|---|-------|-------------|
| 0b | [[00b - Le Preprocesseur C]] | #define, macros, header guards, compilation conditionnelle |
| 1 | [[01 - Introduction au C et Compilation]] | Bases du C, compilation, GCC, types, putchar, printf, contrôle de flux |
| 2 | [[02 - Bases Numeriques]] | Binaire, décimal, hexadécimal, conversions, bitwise, endianness |
| 3 | [[03 - Structures et Typedef]] | Structs, typedef, layout mémoire, padding |
| 3b | [[03b - Les Pointeurs]] | Pointeurs, &, *, arithmétique, arrays, double pointeurs, const |
| 3c | [[03c - Chaines de Caracteres]] | strlen, strcpy, strcmp, strcat, strtok, ctype.h |
| 4 | [[04 - Pointeurs et Memoire]] | malloc, calloc, realloc, free, stack vs heap |
| 5 | [[05 - Pointeurs de Fonctions]] | Function pointers, callbacks, dispatch tables |
| 6 | [[06 - Fonctions Variadiques]] | va_list, va_start, va_arg, va_end |
| 7 | [[07 - Recursion]] | Cas de base, call stack, factorielle, Fibonacci |
| 8 | [[08 - Arguments Ligne de Commande]] | argc, argv, parsing, conversions |
| 9 | [[09 - File IO et Appels Systeme]] | open, close, read, write, file descriptors |
| 10 | [[10 - Projet Printf]] | Architecture _printf, dispatch table, specifiers |
| 11 | [[11 - Makefiles]] | Make, compilation incrémentale, variables, règles |
| 12 | [[12 - Projet Simple Shell]] | fork/execve/wait, PATH, builtins, REPL complet |

### Structures de Données & Algorithmes

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Listes Chainees]] | Singly linked lists, nœuds, parcours, insertion |
| 2 | [[02 - Listes Doublement Chainees]] | Doubly linked lists, prev/next, bidirectionnel |
| 3 | [[03 - Tables de Hachage]] | Hash tables, djb2, collisions, O(1) |
| 4 | [[04 - Arbres Binaires]] | Binary trees, BST, parcours, hauteur |
| 5 | [[05 - Tri et Complexite Algorithmique]] | Big O, Bubble/Insertion/Selection/Quick/Merge Sort |
| 6 | [[06 - Graphes et Algorithmes de Parcours]] | DFS, BFS, Dijkstra, Bellman-Ford, A*, Kruskal, Prim, Union-Find |
| 7 | [[07 - Programmation Dynamique]] | Memoization, tabulation, Knapsack, LCS, Edit Distance, LIS, 10+ classiques |
| 8a | [[08 - Algorithmes Gloutons]] | Activity Selection, Huffman, Prim, échange argument, contre-exemples |
| 8b | [[08 - Heaps et Priority Queues]] | Heap binaire, heapq Python, K éléments, médiane streaming, Huffman |
| 9 | [[09 - Recherche Binaire et Deux Pointeurs]] | Binary search variantes, bisect, two pointers, sliding window |
| 10 | [[10 - Algorithmes de Chaines]] | Rabin-Karp, KMP, Trie, Suffix Array, Manacher, regex DP |
| 11 | [[11 - Complexite Avancee et NP]] | P/NP/NP-Hard/NP-Complete, réductions, algorithmes randomisés |

### Outils de Débogage

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - GDB]] | Breakpoints, step, inspect, backtrace, TUI |
| 2 | [[02 - Valgrind]] | Fuites mémoire, accès invalides, bonnes pratiques |

### Shell & Terminal

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Shell et Commandes Linux]] | Bash, navigation, grep, redirections, processus |
| 2 | [[02 - Shell Scripting]] | Scripts bash, boucles, conditions, fonctions |
| 3 | [[02 - Les Terminaux Guide Complet]] | Bash, Zsh, CMD, PowerShell, Git Bash, WSL |
| 4 | [[01 - Archives et Compression]] | ZIP, TAR, compression, formats d'archives |

### Git, DevOps & Sécurité

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Git et GitHub]] | Workflow, branches, merge, conflits, GitHub Flow |
| 2 | [[01 - Docker]] | Conteneurs, images, Dockerfile, Docker Compose |
| 3 | [[01 - Securite Memoire en C]] | Buffer overflow, dangling pointers, memory leaks |

---

## Trimestre 2 — Fullstack & Web

### Python

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction a Python]] | Interpréteur, types, f-strings, fonctions, comparaisons C |
| 2 | [[02 - Structures de Donnees Python]] | Listes, tuples, dicts, sets, comprehensions |
| 3 | [[03 - POO en Python]] | Classes, héritage, polymorphisme, dunder methods |
| 4 | [[04 - POO Avancee]] | Décorateurs, ABC, design patterns, SOLID |
| 5 | [[05 - Gestion des Erreurs et Fichiers]] | try/except, context managers, JSON, CSV, pathlib |
| 6 | [[06 - Modules Packages et Venv]] | import, pip, venv, structure projet |
| 7 | [[07 - Python Async et Concurrence]] | Threading, multiprocessing, asyncio, GIL |
| 8 | [[08 - APIs REST avec Flask]] | HTTP, routes, CRUD API, blueprints |
| 9 | [[09 - APIs REST avec FastAPI]] | Pydantic, Swagger, async, comparaison Flask |
| 10 | [[10 - Python et Bases de Donnees]] | SQLAlchemy ORM, models, Alembic migrations |
| 11 | [[11 - Projet HBnB]] | **Projet T2** : clone AirBnB, architecture, API, auth, frontend |
| 12 | [[12 - Django Framework Complet]] | Models/Views/URLs/Templates/Forms/Admin/DRF, ORM avancé, déploiement |

### SQL

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction au SQL]] | SGBD, CREATE, SELECT, WHERE, ORDER BY |
| 2 | [[02 - Requetes Avancees SQL]] | JOIN, sous-requêtes, GROUP BY, window functions |
| 3 | [[03 - Conception de Bases de Donnees]] | Normalisation, ER diagrams, clés, indexes |
| 4 | [[04 - SQL Avance et Administration]] | Transactions ACID, vues, procédures, injections SQL |
| 5 | [[05 - Oracle et PL-SQL]] | PL/SQL, procédures, fonctions, triggers, packages, EXPLAIN PLAN, partitionnement |

### Bases de Données NoSQL

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction au NoSQL]] | Pourquoi NoSQL, 4 types, CAP theorem, MongoDB CRUD |
| 2 | [[02 - MongoDB Avance]] | Aggregation pipeline, indexation, transactions, GridFS |
| 3 | [[03 - Redis et Caches]] | Structures Redis, patterns de cache, pub/sub, rate limiting |

### Web Frontend

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - HTML Fondamentaux]] | Sémantique, formulaires, accessibilité |
| 2 | [[02 - CSS Fondamentaux]] | Box model, flexbox, grid, responsive design |
| 3 | [[03 - CSS Avance]] | Animations, BEM, Tailwind, dark mode |
| 4 | [[04 - Projet Web Statique]] | Portfolio HTML/CSS complet, GitHub Pages |
| 5 | [[05 - WebSockets]] | WS vs HTTP, API native, Flask-SocketIO, scaling Redis |
| 6 | [[06 - Tailwind CSS Complet]] | Utility-first, responsive, dark mode, variants, plugins, v4, shadcn/ui |

### JavaScript

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction a JavaScript]] | let/const, types, scope, closures, == vs === |
| 2 | [[02 - JavaScript DOM et Evenements]] | querySelector, events, délégation, localStorage |
| 3 | [[03 - JavaScript Asynchrone]] | Promises, async/await, fetch API, event loop |
| 4 | [[04 - JavaScript Moderne ES6+]] | Destructuring, modules, classes, Map/Set |
| 5 | [[05 - SPA et Frameworks Introduction]] | SPA vs MPA, React/Vue/Svelte, state management |
| 6 | [[06 - Projet JavaScript Interactif]] | TODO app complète, MVC, drag & drop |

### Frameworks Frontend

#### React

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction a React]] | JSX, composants, props, Virtual DOM, setup Vite |
| 2 | [[02 - Hooks et Gestion d'Etat]] | useState, useEffect, custom hooks, Context API, Redux Toolkit |
| 3 | [[03 - React Router et Formulaires]] | Router v6, protected routes, React Hook Form, Zod |
| 4 | [[04 - React Avance et Tests]] | Performance, TanStack Query, Vitest + RTL, déploiement |

#### VueJS

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction a VueJS]] | Options API, Composition API, template syntax, composants |
| 2 | [[02 - VueJS Avance Pinia et Vue Router]] | Composables, Pinia, Vue Router 4, tests, Nuxt |

#### Svelte

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Svelte du Debutant a l'Expert]] | Compilateur, réactivité native, SvelteKit, SSR |

### Frameworks Backend

#### Node / Express / NestJS

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Express.js et NestJS]] | Express middleware, JWT, NestJS decorators, Swagger |

#### PHP / Symfony

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - PHP et Symfony]] | PHP 8.x, Composer, Symfony MVC, Doctrine, API Platform |

#### Java / Spring

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction a Java]] | JVM, types, String, exceptions, Maven |
| 2 | [[02 - Java POO et Collections]] | Héritage, interfaces, generics, Streams API, lambdas |
| 3 | [[03 - Spring Boot et API REST]] | MVC, JPA, Spring Security, JWT, tests JUnit 5 |
| 4 | [[04 - Microservices avec Spring]] | Spring Cloud, Eureka, Feign, Kafka, Resilience4j |

### APIs & Web

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - GraphQL Complet]] | SDL, queries/mutations/subscriptions, Strawberry, Apollo, DataLoader, Federation |

### Tests & Qualité de Code

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Tests Unitaires et TDD]] | pytest, fixtures, mocking, TDD red/green/refactor |
| 2 | [[02 - Tests Integration et E2E]] | Test client, Selenium, Playwright, couverture |
| 3 | [[03 - Linting Formatting et Code Quality]] | pylint, black, ESLint, pre-commit hooks |

### DevOps & CI/CD

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[02 - Docker Avance]] | Multi-stage builds, networks, secrets, optimisation |
| 2 | [[03 - Docker Compose en Pratique]] | Services multiples, .env, dev vs prod |
| 3 | [[04 - CI-CD avec GitHub Actions]] | Workflows YAML, matrix, artifacts, deploy |

### Sécurité Web

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[02 - Securite Web OWASP]] | Top 10 OWASP, XSS, CSRF, injection, HTTPS, CORS |

### IA pour le Développement — Trimestre Complet

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Panorama des IA pour Developpeurs]] | Toutes les familles de modèles, benchmarks, cloud vs local, tableau comparatif |
| 2 | [[02 - Comprendre les LLMs et les Tokens]] | Architecture Transformer, tokenisation, context window, quantisation, hallucinations |
| 3 | [[03 - Prompt Engineering pour le Code]] | Techniques zero/few-shot, chain-of-thought, templates debug/refactor/tests, hack 95% |
| 4 | [[04 - Claude et Claude Code Maitrise Avancee]] | Installation, CLAUDE.md, hooks, sub-agents, Advisor Tool, MCP, prompt cache, Ollama |
| 5 | [[05 - ChatGPT Codex et GitHub Copilot]] | GPT-4o, o1/o3, GitHub Copilot setup, API OpenAI, Canvas, Code Interpreter |
| 6 | [[06 - Gemini Mistral et Alternatives Cloud]] | Gemini 2.5 Pro, Gemini CLI, Codestral, DeepSeek, Qwen, Llama, accès gratuits |
| 7 | [[07 - Integrations IDE et Extensions]] | VS Code (Copilot, Continue.dev, Cline), Cursor, Windsurf, JetBrains AI |
| 8 | [[08 - OpenCode et CLI Alternatifs]] | OpenCode, Aider (auto-commits), Goose, Amp, usage avec Ollama |
| 9 | [[09 - IA Locale avec Ollama]] | Installation, Qwen/DeepSeek/Llama locaux, API compatible OpenAI, serveur équipe |
| 10 | [[10 - LM Studio et Hardware Local]] | Interface GUI, quantisation Q4/Q8, GPU vs CPU, requirements hardware, Jan/LocalAI |
| 11 | [[11 - IA Confidentialite et Entreprise]] | RGPD, politiques providers, Azure/Vertex/Bedrock, auto-hébergement, vLLM, Tabby |
| 12 | [[12 - Strategies Multi-Modeles et Workflows]] | Matrice coût/qualité, routing des tâches, stacks par profil, plan d'action 4 semaines |
| 13 | [[13 - IA Agentic et Developpement MCP]] | MCP servers, multi-agents, LangGraph, CrewAI, pipelines autonomes |

---

## Trimestre 3 — DevOps & Cloud

### Infrastructure as Code

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[05 - Infrastructure as Code Terraform]] | HCL, providers, state, modules |
| 2 | [[06 - Automatisation avec Ansible]] | Playbooks, inventory, roles, Jinja2 |

### Cloud

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction au Cloud]] | IaaX/PaaS/SaaS, AWS/GCP/Azure, CLI |
| 2 | [[02 - Services Cloud Essentiels]] | Compute, Storage, IAM, VPC |
| 3 | [[03 - Deploiement Cloud et Conteneurs]] | ECS/GKE, registres, load balancers |
| 4 | [[04 - Kubernetes Introduction]] | Pods, services, deployments, kubectl |
| 5 | [[05 - Kubernetes Avance Helm et Operators]] | Helm, StatefulSets, RBAC, HPA/KEDA, CRDs, Operators, Istio, GitOps/ArgoCD |
| 6 | [[06 - Cloud Security AWS]] | IAM, Security Groups, VPC, KMS, CloudTrail, GuardDuty, WAF/Shield |

### Réseaux

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Fondamentaux Reseaux]] | OSI/TCP-IP, DNS, sous-réseaux, outils |
| 2 | [[02 - HTTP et Protocoles Web]] | HTTP/1-2-3, TLS, WebSocket, REST vs GraphQL |
| 3 | [[03 - Securite Reseau]] | Firewalls, VPN, SSH, certificates |

### Observabilité & Monitoring

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Logs et Journalisation]] | Niveaux, logging Python, ELK stack |
| 2 | [[02 - Metriques et Monitoring]] | Prometheus, Grafana, SLI/SLO/SLA |
| 3 | [[03 - Tracing et Debugging Distribue]] | OpenTelemetry, correlation IDs |

### DevSecOps & SRE

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[07 - DevSecOps]] | Shift-left, SAST/DAST, secrets management |
| 2 | [[08 - Projet Capstone DevOps]] | **Projet T3** : app complète, CI/CD, cloud, monitoring |
| 3 | [[09 - Incident Response et Postmortems]] | SRE, cycles incident, SBOM, Threat Modeling |
| 4 | [[10 - Jenkins et CI-CD Alternatifs]] | Declarative Pipeline, Shared Libraries, GitLab CI, CircleCI, Blue/Green, Canary |

---

## Bachelor Agentic IA Fullstack — Spécialisations

### Mobile

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction au Dev Mobile]] | iOS vs Android, natif/hybride/cross-platform, écosystèmes |
| 2 | [[02 - Flutter Complet]] | Dart, widgets, Provider, Riverpod, go_router, déploiement |

### Architecture Logicielle

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Microservices et Patterns]] | DDD, Saga, CQRS, Event Sourcing, service mesh, Istio |

### UML & Modélisation

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - UML Fondamentaux et Diagrammes]] | 8 types de diagrammes, PlantUML, Mermaid, exemples Python/Java |

### Gestion de Projet & Mastère

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Methodes Agiles Scrum et Kanban]] | Scrum, Kanban, XP, métriques, ceremonies, backlog |
| 2 | [[02 - Gestion de Projet Classique et ITIL]] | WBS, PERT, ITIL v4, RGPD, Green IT, gouvernance |

---

## Bachelor Machine Learning

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Mathematiques pour le ML]] | Algèbre linéaire, calcul différentiel, probabilités, statistiques |
| 2 | [[02 - NumPy Pandas et Visualisation]] | NumPy, Pandas, Matplotlib, Seaborn, EDA complète |
| 3 | [[03 - ML Supervise avec Scikit-Learn]] | Régression, classification, pipeline, validation croisée, tuning |
| 4 | [[04 - Reseaux de Neurones et Deep Learning]] | Perceptron, backpropagation, Keras, PyTorch, GPU training |
| 5 | [[05 - CNN Transformer et Computer Vision]] | CNN, ResNet, Transfer Learning, BERT, ViT, attention |
| 6 | [[06 - ML Non Supervise et Reinforcement Learning]] | Clustering, PCA, t-SNE, GANs, Q-Learning, PPO |
| 7 | [[07 - MLOps et Deploiement de Modeles]] | MLflow, DVC, FastAPI serving, monitoring drift, CI/CD ML |

### Big Data

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Hadoop et Ecosysteme MapReduce]] | HDFS, MapReduce, YARN, Hive, Pig, HBase, Sqoop, Flume |
| 2 | [[02 - Apache Spark et PySpark]] | RDD, DataFrame, Streaming, MLlib, Delta Lake, optimisation |

---

## Bachelor Cybersécurité

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Fondamentaux Hacking Ethique]] | PTES, MITRE ATT&CK, phases pentest, environnements lab, CTF |
| 2 | [[02 - Reconnaissance et OSINT]] | Google Dorks, Shodan, nmap, amass, theHarvester, Maltego |
| 3 | [[03 - Exploitation Web et Systeme]] | SQLi, XSS, SSRF, Metasploit, Buffer Overflow, shellcode |
| 4 | [[04 - Active Directory et Post-Exploitation]] | Kerberoasting, BloodHound, LPE, pivoting, C2 |
| 5 | [[05 - CTF Methodologie et Red Team]] | Catégories CTF, pwntools, crypto, forensics, Red Team ops |
| 6 | [[06 - Digital Forensics et Analyse Malware]] | Volatility, Autopsy, Wireshark forensics, Ghidra, YARA, IOCs |

---

## Supplémentaire — Rust

| # | Cours | Description |
|---|-------|-------------|
| 1 | [[01 - Introduction a Rust]] | cargo, types, mutabilité, comparaisons C |
| 2 | [[02 - Ownership et Borrowing]] | Propriété, références, lifetimes |
| 3 | [[03 - Structs Enums et Pattern Matching]] | Option, Result, match, if let |
| 4 | [[04 - Gestion des Erreurs en Rust]] | Result, ?, anyhow, thiserror |
| 5 | [[05 - Collections et Iterateurs]] | Vec, HashMap, closures, map/filter/fold |
| 6 | [[06 - Traits et Generiques]] | Traits, impl, generics, dyn Trait |
| 7 | [[07 - Projet Rust CLI]] | minigrep complet, clap, tests, publication |
| 8 | [[08 - Concurrence et Smart Pointers]] | Arc, Mutex, channels mpsc, Rayon, atomics, Send/Sync |
| 9 | [[09 - Async Await et Tokio]] | Future trait, runtime Tokio, I/O async, channels async, select!, Streams |
| 10 | [[10 - Macros et Metaprogrammation]] | macro_rules!, derive macros, attribute macros, proc-macro, build.rs |
| 11 | [[11 - Unsafe Rust et FFI]] | Raw pointers, extern "C", bindgen, cbindgen, MaybeUninit, repr(C) |
| 12 | [[12 - GUI avec egui et eframe]] | **Immediate mode GUI**, widgets, layouts, panels, custom painting, Tokio async, WASM |
| 13 | [[13 - Tauri Desktop Applications]] | Backend Rust + frontend web, IPC, plugins, system tray, mobile |
| 14 | [[14 - WebAssembly avec Rust]] | wasm-bindgen, web-sys, wasm-pack, trunk, Leptos, Yew, Dioxus |
| 15 | [[15 - Web Frameworks Axum et Actix]] | Axum extractors, Tower middleware, SQLx, JWT, WebSockets, Actix-web |

---

## Parcours recommandé

> [!tip] Planning semaine par semaine
> **T1 — Semaines 1-9** : Fondamentaux → C → Pointeurs → Structures → Mémoire → Data Structures (+ algo avancé) → Debugging
> **T2 — Semaines 10-23** : Python → SQL → NoSQL → HTML/CSS → JavaScript → React/Vue/Svelte → APIs (GraphQL) → Tests → CI/CD → IA
> **T3 — Semaines 24-32** : Docker avancé → Cloud → Kubernetes (+ Helm/Operators) → Réseaux → Observabilité → DevSecOps → Jenkins
> **Bachelor ML** : Mathématiques → NumPy/Pandas → Scikit-Learn → Deep Learning → MLOps → Big Data (Hadoop/Spark)
> **Bachelor Cybersec** : Fondamentaux → OSINT → Exploitation → AD → CTF/Red Team → Digital Forensics
> **En continu** : Git, Sécurité, Shell, Rust (1-15), IA Agentic, UML/Modélisation

```mermaid
graph LR
    subgraph T1[Trimestre 1 - C & Système]
        FOND[Fondamentaux Info] --> C[Langage C]
        C --> DS[Data Structures]
        DS --> ALGO[Algo Avancé<br/>DP/Graphes/NP]
        C --> DBG[Debugging]
        C --> MK[Makefiles]
    end
    subgraph T2[Trimestre 2 - Fullstack]
        PY[Python] --> SQL[SQL + PL/SQL]
        SQL --> NOSQL[NoSQL/Redis]
        PY --> API[APIs Flask/FastAPI/Django]
        HTML[HTML/CSS/Tailwind/WS] --> JS[JavaScript]
        JS --> FW[React/Vue/Svelte]
        JS --> BE[Node/PHP/Java]
        API --> GQL[GraphQL]
        API --> TEST[Tests]
        TEST --> CICD[CI/CD]
        IA[IA pour le Dev]
    end
    subgraph T3[Trimestre 3 - DevOps]
        DOCK[Docker Avancé] --> CLOUD[Cloud]
        CLOUD --> K8S[Kubernetes + Helm]
        IAC[Terraform/Ansible] --> CLOUD
        OBS[Observabilité] --> DEVSEC[DevSecOps/SRE]
        DEVSEC --> JENKINS[Jenkins/CI Alternatifs]
    end
    T1 --> T2 --> T3
    subgraph BAC[Bachelors Spécialisés]
        ML[Machine Learning x7]
        BIG[Big Data<br/>Hadoop/Spark]
        CYBER[Cybersécurité x6]
        MOB[Mobile/Flutter]
        ARCHI[Architecture/Microservices]
        UML[UML & Modélisation]
        PM[Gestion de Projet]
        CLOUDSEC[Cloud Security AWS]
    end
    T2 --> BAC
    subgraph EXTRA[Supplémentaire]
        RUST[Rust 1-15<br/>concurrence/async/FFI/GUI/WASM]
    end
    C --> RUST
```

---

*Notes générées à partir des cheatsheets Holberton School, de 43 discussions Gemini, et enrichies avec des explications approfondies.*
