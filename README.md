# Programming Sheet for Obsidian

**85 cours de programmation complets, en francais, au format Obsidian.**

> Un coffre Obsidian couvrant 3 trimestres de formation en developpement logiciel — du langage C jusqu'au deploiement cloud avec Kubernetes, en passant par Python, JavaScript, SQL, Rust, et DevOps.

---

## Chiffres cles

| | |
|---|---|
| **Notes** | 85 fichiers Markdown |
| **Lignes** | ~89 000 lignes de contenu |
| **Langages** | C, Python, JavaScript, SQL, Rust, HTML/CSS, Bash, HCL, YAML |
| **Langue** | Francais (termes techniques en anglais) |
| **Format** | Obsidian Markdown avec `[[wikilinks]]`, callouts, diagrammes ASCII |

---

## Pour qui ?

- Etudiants en ecole de programmation (Holberton School, 42, etc.)
- Developpeurs debutants qui veulent des cours structures et progressifs
- Toute personne souhaitant apprendre la programmation de zero a deployement en production

Chaque note est concue pour qu'un **novice complet** puisse, apres lecture, **ecrire et mettre en pratique** toutes les connaissances abordees.

---

## Installation

### Option 1 : Cloner et ouvrir dans Obsidian

```bash
git clone https://github.com/Rwanbt/Programming-sheet-for-Obsidian.git
```

Ouvrir Obsidian > **Open folder as vault** > selectionner le dossier clone.

### Option 2 : Telecharger le ZIP

Cliquer sur **Code > Download ZIP** sur GitHub, extraire, puis ouvrir dans Obsidian.

### Navigation

Commencer par le fichier **`00 - Index Programmation.md`** qui sert de table des matieres avec des liens vers toutes les notes, organise par trimestre.

---

## Structure du coffre

```
Programming-sheet-for-Obsidian/
|
|-- 00 - Index Programmation.md        <- Point d'entree, navigation globale
|
|-- C language/                         (16 notes)
|   |-- 00b - Le Preprocesseur C
|   |-- 01 - Introduction au C et Compilation
|   |-- 02 - Bases Numeriques (Binaire, Hex, Decimal)
|   |-- 03 - Structures et Typedef
|   |-- 03b - Les Pointeurs
|   |-- 03c - Chaines de Caracteres
|   |-- 04 - Pointeurs et Memoire (malloc, free)
|   |-- 05 - Pointeurs de Fonctions
|   |-- 06 - Fonctions Variadiques
|   |-- 07 - Recursion
|   |-- 08 - Arguments Ligne de Commande (argc, argv)
|   |-- 09 - File IO et Appels Systeme
|   |-- 10 - Projet Printf
|   |-- 11 - Makefiles
|   +-- 12 - Projet Simple Shell          <- Projet capstone T1
|
|-- Structures de Donnees/              (5 notes)
|   |-- 01 - Listes Chainees
|   |-- 02 - Listes Doublement Chainees
|   |-- 03 - Tables de Hachage
|   |-- 04 - Arbres Binaires
|   +-- 05 - Tri et Complexite Algorithmique (Big O)
|
|-- Python/                             (11 notes)
|   |-- 01 - Introduction a Python
|   |-- 02 - Structures de Donnees Python
|   |-- 03 - POO en Python
|   |-- 04 - POO Avancee (decorateurs, patterns)
|   |-- 05 - Gestion des Erreurs et Fichiers
|   |-- 06 - Modules Packages et Venv
|   |-- 07 - Python Async et Concurrence
|   |-- 08 - APIs REST avec Flask
|   |-- 09 - APIs REST avec FastAPI
|   |-- 10 - Python et Bases de Donnees (SQLAlchemy)
|   +-- 11 - Projet HBnB                  <- Projet capstone T2 (clone AirBnB)
|
|-- SQL/                                (4 notes)
|   |-- 01 - Introduction au SQL
|   |-- 02 - Requetes Avancees (JOIN, Window Functions)
|   |-- 03 - Conception de Bases de Donnees
|   +-- 04 - SQL Avance et Administration
|
|-- JavaScript/                         (6 notes)
|   |-- 01 - Introduction a JavaScript
|   |-- 02 - JavaScript DOM et Evenements
|   |-- 03 - JavaScript Asynchrone (Promises, async/await)
|   |-- 04 - JavaScript Moderne ES6+
|   |-- 05 - SPA et Frameworks Introduction
|   +-- 06 - Projet JavaScript Interactif (TODO app)
|
|-- Web Frontend/                       (4 notes)
|   |-- 01 - HTML Fondamentaux
|   |-- 02 - CSS Fondamentaux (Flexbox, Grid)
|   |-- 03 - CSS Avance (Animations, Tailwind, BEM)
|   +-- 04 - Projet Web Statique (Portfolio)
|
|-- Rust/                               (7 notes)
|   |-- 01 - Introduction a Rust
|   |-- 02 - Ownership et Borrowing
|   |-- 03 - Structs Enums et Pattern Matching
|   |-- 04 - Gestion des Erreurs en Rust
|   |-- 05 - Collections et Iterateurs
|   |-- 06 - Traits et Generiques
|   +-- 07 - Projet Rust CLI (minigrep)
|
|-- DevOps/                             (8 notes)
|   |-- 01 - Docker
|   |-- 02 - Docker Avance (multi-stage, networks)
|   |-- 03 - Docker Compose en Pratique
|   |-- 04 - CI-CD avec GitHub Actions
|   |-- 05 - Infrastructure as Code (Terraform)
|   |-- 06 - Automatisation avec Ansible
|   |-- 07 - DevSecOps
|   +-- 08 - Projet Capstone DevOps        <- Projet capstone T3
|
|-- Cloud/                              (4 notes)
|   |-- 01 - Introduction au Cloud
|   |-- 02 - Services Cloud Essentiels
|   |-- 03 - Deploiement Cloud et Conteneurs
|   +-- 04 - Kubernetes Introduction
|
|-- Reseaux/                            (3 notes)
|   |-- 01 - Fondamentaux Reseaux (OSI, TCP/IP)
|   |-- 02 - HTTP et Protocoles Web
|   +-- 03 - Securite Reseau
|
|-- Observabilite/                      (3 notes)
|   |-- 01 - Logs et Journalisation
|   |-- 02 - Metriques et Monitoring (Prometheus, Grafana)
|   +-- 03 - Tracing et Debugging Distribue
|
|-- Debugging/                          (2 notes)
|   |-- 01 - GDB
|   +-- 02 - Valgrind
|
|-- Tests et Qualite/                   (3 notes)
|   |-- 01 - Tests Unitaires et TDD (pytest)
|   |-- 02 - Tests Integration et E2E
|   +-- 03 - Linting Formatting et Code Quality
|
|-- Secutrity/                          (2 notes)
|   |-- 01 - Securite Memoire en C
|   +-- 02 - Securite Web OWASP (Top 10)
|
|-- Shell/                              (2 notes)
|   |-- 01 - Shell et Commandes Linux
|   +-- 02 - Shell Scripting
|
|-- Terminal/                           (2 notes)
|   |-- 01 - Archives et Compression
|   +-- 02 - Les Terminaux Guide Complet (Bash, Zsh, CMD, PowerShell, WSL)
|
|-- Git/                                (1 note)
|   +-- 01 - Git et GitHub
|
+-- IA pour le Dev/                     (2 notes)
    |-- 01 - IA Assistants de Code
    +-- 02 - IA et Productivite Dev
```

---

## Contenu par trimestre

### Trimestre 1 — Fondations (C, Systeme, Outils)

Les bases de la programmation avec le langage C, les structures de donnees, les outils de debug, le terminal et Git.

| Domaine | Notes | Sujets cles |
|---------|-------|-------------|
| Langage C | 16 | Compilation, types, pointeurs, memoire, structs, recursion, file I/O, Makefiles |
| Structures de Donnees | 5 | Listes chainees, hash tables, arbres binaires, algorithmes de tri, Big O |
| Debugging | 2 | GDB (breakpoints, backtrace, TUI), Valgrind (fuites memoire, acces invalides) |
| Shell & Terminal | 4 | Commandes Linux, scripting bash, guide des terminaux, archives |
| Git | 1 | Workflow complet, branches, merge, conflits, GitHub Flow |
| Docker | 1 | Conteneurs, images, Dockerfile, volumes |
| Securite | 1 | Securite memoire en C, buffer overflow, dangling pointers |
| **Projet** | **Simple Shell** | **Construire un interpreteur de commandes UNIX en C** |

### Trimestre 2 — Fullstack & Web

Backend Python, frontend HTML/CSS/JS, bases de donnees SQL, tests, CI/CD et securite web.

| Domaine | Notes | Sujets cles |
|---------|-------|-------------|
| Python | 11 | POO, async, Flask, FastAPI, SQLAlchemy, decorateurs, design patterns |
| SQL | 4 | SELECT/JOIN/GROUP BY, normalisation, transactions ACID, injections SQL |
| JavaScript | 6 | DOM, events, async/await, ES6+, SPA, frameworks (React/Vue/Svelte) |
| Web Frontend | 4 | HTML semantique, CSS Flexbox/Grid, responsive, animations, projet portfolio |
| Tests | 3 | pytest, TDD, integration, E2E (Selenium/Playwright), linting |
| DevOps | 3 | Docker avance, Docker Compose, CI/CD GitHub Actions |
| Securite Web | 1 | OWASP Top 10, XSS, CSRF, CORS, HTTPS, JWT |
| IA pour le Dev | 2 | Copilot, Claude, prompt engineering, agents, MCP |
| **Projet** | **HBnB** | **Clone AirBnB complet (API + frontend + auth + DB)** |

### Trimestre 3 — DevOps & Cloud

Infrastructure as Code, cloud, reseaux, Kubernetes, monitoring et DevSecOps.

| Domaine | Notes | Sujets cles |
|---------|-------|-------------|
| DevOps | 3 | Terraform (IaC), Ansible, DevSecOps (SAST/DAST, secrets) |
| Cloud | 4 | AWS/GCP/Azure, compute/storage/IAM, deploiement conteneurs, Kubernetes |
| Reseaux | 3 | OSI/TCP-IP, HTTP/TLS, securite reseau, SSH, VPN |
| Observabilite | 3 | Logs structures, Prometheus/Grafana, tracing distribue (OpenTelemetry) |
| **Projet** | **Capstone DevOps** | **Application web complete avec pipeline CI/CD et deploiement cloud** |

### Supplementaire — Rust

Un parcours complet pour apprendre Rust en parallele, avec des comparaisons constantes avec le C.

| Notes | Sujets cles |
|-------|-------------|
| 7 | Ownership, borrowing, lifetimes, structs/enums, pattern matching, traits, generiques, iterateurs, projet CLI complet |

---

## Ce que contient chaque note

Chaque note suit un format pedagogique coherent :

- **Introduction** avec definition et analogie concrete
- **Theorie** expliquee progressivement, du simple au complexe
- **Exemples de code** complets et commentes (copier-coller et tester)
- **Diagrammes ASCII** pour visualiser la memoire, les architectures, les flux
- **Tableaux recapitulatifs** des commandes, fonctions, comparaisons
- **Callouts Obsidian** : `> [!tip]`, `> [!warning]`, `> [!info]`, `> [!example]`
- **Carte mentale ASCII** resumant tous les concepts
- **Exercices pratiques** (2-4 par note) pour mettre en application
- **Liens `[[wikilinks]]`** vers les notes liees pour naviguer dans le coffre
- **Comparaisons entre langages** (C vs Python vs JS vs Rust)

---

## Parcours recommande

```
Semaine 1-2    : Terminal + Git + Introduction au C
Semaine 3-4    : Pointeurs + Chaines + Structures + Memoire (malloc/free)
Semaine 5-6    : Recursion + Listes chainees + Hash tables + GDB + Valgrind
Semaine 7-8    : Arbres binaires + Tri/Big O + Printf + Makefiles
Semaine 9      : PROJET SIMPLE SHELL
Semaine 10-11  : Python (bases, POO, fichiers, modules)
Semaine 12-13  : SQL + HTML/CSS + JavaScript (bases, DOM, async)
Semaine 14-15  : APIs (Flask/FastAPI) + SQLAlchemy + Tests
Semaine 16-17  : Docker avance + CI/CD + Securite OWASP
Semaine 18     : PROJET HBnB (clone AirBnB)
Semaine 19-20  : Terraform + Ansible + Cloud
Semaine 21-22  : Reseaux + Kubernetes + Monitoring
Semaine 23-24  : DevSecOps + Observabilite avancee
Semaine 25-27  : PROJET CAPSTONE DEVOPS
En continu     : Rust (en parallele), IA pour le dev
```

---

## Contribuer

Les contributions sont les bienvenues ! Si tu trouves une erreur, un concept mal explique, ou si tu veux ajouter une note :

1. Fork le repo
2. Cree une branche (`git checkout -b fix/nom-correction`)
3. Fais tes modifications en respectant le format existant
4. Ouvre une Pull Request

### Format a respecter

- Francais, termes techniques en anglais quand c'est standard
- Titre H1, puis introduction "Qu'est-ce que..."
- Au moins un callout `> [!tip] Analogie`
- Code blocks avec tags de langue (` ```python `, ` ```c `, etc.)
- Finir par : Carte Mentale ASCII + Exercices (2-4) + Liens (wikilinks)
- 600-1200 lignes par note

---

## Licence

Ce projet est partage librement pour l'education. Utilisez-le, partagez-le, ameliorez-le.

---

## Auteurs

- **Erwan Barat** ([@Rwanbt](https://github.com/Rwanbt)) — Etudiant Holberton School
- **Claude Opus 4.6** — Assistant IA (generation et structuration du contenu)
- **Gemini** — Assistant IA (discussions d'apprentissage et exercices)

---

> *"La meilleure facon d'apprendre, c'est d'enseigner."* — Richard Feynman
>
> Ce coffre est ne de cette philosophie : structurer ses connaissances pour les rendre accessibles a tous.
