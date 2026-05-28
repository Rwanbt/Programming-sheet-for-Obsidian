# Méthodes Agiles : Scrum et Kanban

## Pourquoi l'Agilité ?

Avant l'Agile, les projets logiciels suivaient le modèle **Waterfall** (cascade) : on analysait tout, on concevait tout, on développait tout, on testait tout, on livrait tout — dans cet ordre, sans retour en arrière. Le problème ? Le client ne voyait le produit qu'après 18 mois de développement, et ce qu'il voyait n'était jamais ce qu'il voulait vraiment.

> [!tip] Le problème classique du développement logiciel
> "Le client voulait une balançoire. Voilà ce que chaque département a livré :
> - Ce que le client a décrit → une balançoire
> - Ce que le chef de projet a compris → un canapé accroché à un arbre
> - Ce que le développeur a codé → une corde attachée à une branche
> - Ce que le consultant a facturé → l'arbre entier
> - Ce dont le client avait vraiment besoin → juste une chaise de jardin confortable"
>
> L'Agile résout ce problème en montrant le produit au client **toutes les 2 semaines** et en s'adaptant à son feedback.

---

## Le Manifeste Agile (2001)

En février 2001, 17 développeurs se réunissent dans une station de ski de l'Utah. Ils publient le **Manifeste Agile** — 68 mots qui transformeront l'industrie du logiciel.

### Les 4 Valeurs Fondamentales

```
Nous découvrons de meilleures façons de développer des logiciels
par la pratique et en aidant les autres à le faire.
Ces expériences nous ont amenés à valoriser :

1. Les INDIVIDUS et leurs INTERACTIONS
   plus que les processus et les outils

2. Un LOGICIEL OPÉRATIONNEL
   plus qu'une documentation exhaustive

3. La COLLABORATION avec le CLIENT
   plus que la négociation contractuelle

4. L'ADAPTATION AU CHANGEMENT
   plus que le suivi d'un plan

Cela signifie que, bien qu'il y ait de la valeur dans les éléments
à droite, nous valorisons davantage les éléments à gauche.
```

> [!warning] Malentendu courant
> Agile ne signifie **pas** "pas de documentation", "pas de plan", "pas de contrat". Cela signifie que ces choses sont moins valorisées que les livrables et la collaboration. Un projet Agile a toujours une architecture, des tests, une documentation — mais ces artefacts sont créés en fonction des besoins, pas à l'avance pour tout.

### Les 12 Principes Agiles

| # | Principe | En pratique |
|---|---|---|
| 1 | Livrer de la valeur tôt et continuellement | Pas de big bang release |
| 2 | Bienvenir les changements tardifs | Le changement est un avantage compétitif |
| 3 | Livrer fréquemment (2 sem. à 2 mois) | Sprints, releases régulières |
| 4 | Collaboration quotidienne business + devs | Pas de mur entre IT et métier |
| 5 | Construire autour de personnes motivées | Confiance, environnement, support |
| 6 | Conversation face-à-face | Le mode de communication le plus efficace |
| 7 | Logiciel fonctionnel = mesure de progrès | Pas les Gantt charts ou les docs |
| 8 | Développement durable | Pas de sprints mortels sur la durée |
| 9 | Excellence technique en continu | Refactoring, tests, code propre |
| 10 | Simplicité — maximiser le travail non fait | YAGNI : You Aren't Gonna Need It |
| 11 | Équipes auto-organisées | Pas de micro-management |
| 12 | Réflexion régulière sur l'amélioration | Les rétrospectives |

---

## Waterfall vs Agile

| Dimension | Waterfall (Cascade) | Agile |
|---|---|---|
| **Planification** | Complète au départ | Continue et adaptative |
| **Durée type** | 12-24 mois | 2-4 semaines (par sprint) |
| **Retour client** | En fin de projet | Toutes les 2 semaines |
| **Changements** | Coûteux et résistés | Bienvenus |
| **Documentation** | Exhaustive et préalable | Juste suffisante |
| **Mesure de succès** | Respect du plan initial | Valeur livrée au client |
| **Risque** | Découvert tard (en fin) | Découvert tôt (en sprint) |
| **Adapté pour** | Projets stables (construction) | Logiciel, produits incertains |
| **Exemples** | Centrale nucléaire, pont | App mobile, site e-commerce |

> [!info] Waterfall n'est pas toujours mauvais
> Pour construire un pont ou une centrale nucléaire, on ne fait pas d'"itérations" — les spécifications sont définies à l'avance et les changements en cours de construction sont catastrophiques. Waterfall est adapté aux projets où les exigences sont stables et le domaine bien compris. En logiciel, c'est rarement le cas.

---

## Scrum

Scrum est le framework Agile le plus utilisé. Ce n'est pas une méthode complète, mais un **cadre** qui définit des rôles, des artefacts et des cérémonies.

### Les 3 Rôles Scrum

#### Product Owner (PO)

**Responsabilité unique :** maximiser la valeur du produit.

- Représente les parties prenantes (clients, métier, direction)
- Gère le **Product Backlog** : priorise, ajoute, supprime des items
- Décide de ce qui entre dans chaque sprint (avec l'équipe)
- Accepte ou rejette le travail lors de la Sprint Review
- Il dit **QUOI** construire — jamais **COMMENT**

> [!warning] Le PO n'est pas un chef de projet
> Le Product Owner ne gère pas les tâches techniques, n'assigne pas le travail aux développeurs, et ne dicte pas les délais de chaque tâche. Son rôle est de représenter la valeur business. Un PO qui passe son temps à créer des sous-tâches Jira est un anti-pattern.

#### Scrum Master

**Responsabilité unique :** servir l'équipe et protéger le processus Scrum.

- Facilite les cérémonies (planning, standup, review, rétro)
- Élimine les **impediments** (obstacles qui bloquent l'équipe)
- Coach l'équipe sur les pratiques Agile
- Protège l'équipe des interruptions externes pendant le sprint
- Il n'est **pas** chef d'équipe — il est au service de l'équipe

#### Development Team (Équipe de Développement)

- 3 à 9 personnes (idéalement 5-7)
- **Auto-organisée** : décide comment atteindre l'objectif du sprint
- **Pluridisciplinaire** : devs, QA, designers, ops — tout ce qu'il faut pour livrer
- Collectivement responsable du résultat (pas "c'est le bug du dev X")
- Estime l'effort des User Stories

### Les 3 Artefacts Scrum

#### Product Backlog

La liste ordonnée de tout ce qui pourrait être fait dans le produit.

```
Product Backlog (exemple app de commande de repas) :
┌─────────────────────────────────────────────────────┐
│  PRIORITÉ HAUTE                                      │
│  [US-001] Créer un compte     📊 8 pts  ✅ affinée   │
│  [US-002] Se connecter        📊 3 pts  ✅ affinée   │
│  [US-003] Voir les restaurants 📊 5 pts ✅ affinée  │
│                                                      │
│  PRIORITÉ MOYENNE                                    │
│  [US-004] Passer une commande 📊 13 pts ⚠️ à raffiner│
│  [US-005] Suivre la livraison 📊 ?  pts ❓ floue    │
│                                                      │
│  BASSE PRIORITÉ (épic, pas encore découpée)          │
│  [EPIC] Programme de fidélité                        │
│  [EPIC] Mode hors ligne                              │
└─────────────────────────────────────────────────────┘
```

- **Ordonné** (pas "priorisé" par couleur) : le PO décide de l'ordre
- **Vivant** : change à chaque sprint (nouvelles idées, bugs, changements de cap)
- **Estimé** en haut de la liste (les items flous du bas ne sont pas estimés)

#### Sprint Backlog

La liste des tâches sélectionnées pour le sprint en cours + le plan pour les réaliser.

```
Sprint 7 — Objectif : "Un utilisateur peut s'inscrire et parcourir les restaurants"
Duration : 14 jours (12 mai → 26 mai)

┌──────────────────────────────────┬───────┬────────────────┐
│ User Story / Tâche               │ Points│ Statut         │
├──────────────────────────────────┼───────┼────────────────┤
│ US-001 : Créer un compte         │   8   │ En cours       │
│   - Formulaire inscription       │       │ ✅ Done        │
│   - Validation email             │       │ ✅ Done        │
│   - Envoi email confirmation     │       │ 🔄 In Progress │
│   - Tests unitaires              │       │ ⬜ Todo        │
├──────────────────────────────────┼───────┼────────────────┤
│ US-002 : Se connecter            │   3   │ Todo           │
│ US-003 : Voir les restaurants    │   5   │ Todo           │
└──────────────────────────────────┴───────┴────────────────┘
```

#### Increment

Le produit **potentiellement livrable** à la fin du sprint. Chaque incrément s'ajoute aux précédents. Pour être "Done", l'incrément doit respecter la **Definition of Done** (DoD).

**Exemple de Definition of Done :**
- Code committé sur main
- Tests unitaires écrits et passants (coverage > 80 %)
- Tests d'intégration passants
- Code review approuvée par un pair
- Documentation mise à jour si nécessaire
- Déployé en environnement de staging
- PO a validé le comportement

### Les 5 Cérémonies Scrum

#### Sprint Planning (Planification du Sprint)

```
Qui : PO + Scrum Master + Dev Team
Durée : 2h max pour un sprint de 2 semaines (règle : 1h par semaine de sprint)
Quand : Début de chaque sprint

Agenda :
1. PO présente l'objectif du sprint (Sprint Goal) — 15 min
2. L'équipe sélectionne les User Stories du Product Backlog — 30 min
3. L'équipe découpe les stories en tâches techniques — 1h
4. L'équipe confirme sa capacité (vélocité, absences) — 15 min

Output : Sprint Backlog + Sprint Goal défini
```

#### Daily Standup (Mêlée Quotidienne)

```
Qui : Dev Team (PO et SM peuvent observer, pas obligatoire)
Durée : 15 minutes MAX
Quand : Chaque jour, même heure, même lieu
Format : Debout (pour rester bref)

3 questions par personne :
  ✓ "Qu'est-ce que j'ai fait hier ?"
  ✓ "Qu'est-ce que je vais faire aujourd'hui ?"
  ✓ "Y a-t-il des obstacles qui me bloquent ?"

Anti-patterns :
  ✗ Rapport au Scrum Master (c'est une sync entre pairs)
  ✗ Résolution de problèmes en réunion (noter → offline)
  ✗ Dépasse 15 minutes
  ✗ "J'ai travaillé sur..." sans mentionner l'User Story
```

#### Sprint Review (Revue de Sprint)

```
Qui : PO + Dev Team + parties prenantes (clients, direction)
Durée : 1h max pour sprint de 2 semaines
Quand : Avant-dernière heure du sprint

Agenda :
1. Dev Team démontre le logiciel fonctionnel (pas de slides !)
2. PO valide ou rejette chaque User Story contre les critères d'acceptation
3. Parties prenantes donnent du feedback
4. PO met à jour le Product Backlog en conséquence

Ce n'est PAS une réunion de validation bureaucratique.
C'est une démonstration de logiciel qui marche.
```

#### Sprint Retrospective

```
Qui : Dev Team + Scrum Master (PO optionnel)
Durée : 1h30 pour sprint de 2 semaines
Quand : Dernière cérémonie du sprint

Objectif : améliorer le processus, pas livrer plus

Format classique "Start / Stop / Continue" :
  START (quoi commencer) :
    - "On devrait faire du pair programming sur les tickets complexes"
    - "Écrire les tests avant le code"

  STOP (quoi arrêter) :
    - "Les interruptions pendant le standup pour des questions hors-scope"
    - "Commencer des stories qu'on ne finira pas dans le sprint"

  CONTINUE (quoi garder) :
    - "Le code review en 24h max, ça marche bien"
    - "Les Friday demos informelles"

Output : 1-3 actions concrètes pour le prochain sprint
```

#### Backlog Refinement (Grooming) — La 5e cérémonie

```
Qui : PO + Dev Team
Durée : max 10 % du sprint (2h pour sprint 2 semaines)
Quand : Mi-sprint (pas une cérémonie officielle Scrum mais universellement pratiquée)

Objectif : s'assurer que le haut du backlog est prêt pour le prochain sprint
  - Découper les épics en stories
  - Estimer les stories non estimées
  - Raffiner les critères d'acceptation
  - Identifier les dépendances
```

---

## User Stories

### Format Standard

```
En tant que [PERSONA],
Je veux [ACTION / FONCTIONNALITÉ],
Afin de [BÉNÉFICE / VALEUR MÉTIER].
```

**Exemples :**

```
En tant qu'acheteur,
Je veux pouvoir sauvegarder des articles dans une wishlist,
Afin de les retrouver facilement lors de ma prochaine visite.

En tant qu'administrateur,
Je veux pouvoir désactiver un compte utilisateur,
Afin de gérer les abus sans supprimer définitivement le compte.
```

### Critères INVEST pour une bonne User Story

| Lettre | Signification | Question à se poser |
|---|---|---|
| **I** | Independent | La story peut être développée sans dépendre d'une autre ? |
| **N** | Negotiable | Les détails peuvent être affinés avec le PO ? |
| **V** | Valuable | Elle apporte de la valeur à l'utilisateur final ? |
| **E** | Estimable | L'équipe peut l'estimer en story points ? |
| **S** | Small | Elle tient dans un sprint (pas un mois de travail) ? |
| **T** | Testable | On peut écrire des tests d'acceptation dessus ? |

### Critères d'Acceptation

Les critères d'acceptation définissent **exactement** quand une story est terminée.

```
User Story : "En tant qu'utilisateur, je veux me connecter avec mon email/password"

Critères d'acceptation (format Gherkin Given/When/Then) :

SCENARIO 1 - Connexion réussie :
  GIVEN un utilisateur avec email "alice@test.com" et password "Secret123"
  WHEN il soumet le formulaire de connexion
  THEN il est redirigé vers son dashboard
  AND un cookie de session est créé

SCENARIO 2 - Mauvais password :
  GIVEN un utilisateur avec email "alice@test.com"
  WHEN il soumet le formulaire avec un mauvais password
  THEN il voit le message "Email ou mot de passe incorrect"
  AND le formulaire ne révèle pas si c'est l'email ou le password qui est faux

SCENARIO 3 - Blocage après 5 tentatives :
  GIVEN 5 tentatives échouées en 10 minutes
  WHEN il soumet une 6e tentative (même si correct)
  THEN il voit "Compte temporairement bloqué. Réessayez dans 15 minutes."
```

### Story Points et Planning Poker

Les **story points** mesurent la complexité relative, pas le temps.

**Échelle de Fibonacci** (suite partielle) : 1, 2, 3, 5, 8, 13, 21, 40, 100

```
Planning Poker :
1. Le PO présente une User Story
2. Chaque développeur choisit une carte en secret
3. Révélation simultanée (évite l'influence)
4. Discussion sur les écarts importants
5. Nouvelle estimation si nécessaire
6. Consensus → story estimée

Pourquoi Fibonacci et pas 1-2-3-... ?
→ L'écart entre les chiffres croît avec la complexité.
  Une story à 5 points est 5x plus complexe qu'une à 1 point.
  La différence entre 8 et 13 est volontairement floue
  (on ne peut pas être précis sur des grosses stories).
```

---

## Kanban

Kanban vient du japonais "tableau visuel". Issu du système Toyota, adapté au développement logiciel par David J. Anderson en 2007.

### Différences fondamentales avec Scrum

| Dimension | Scrum | Kanban |
|---|---|---|
| **Itérations** | Sprints fixes (1-4 semaines) | Flux continu, pas d'itérations |
| **Rôles** | PO, SM, Dev Team | Aucun rôle prescrit |
| **Engagement** | Sprint backlog figé | Priorisation continue |
| **Changements mid-sprint** | Interdits | Autorisés à tout moment |
| **Métriques** | Vélocité, burn-down | Cycle time, throughput |
| **Adapté pour** | Développement produit | Support, maintenance, ops |

### Le Tableau Kanban

```
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│  BACKLOG │  ANALYSE │  DEV     │  REVIEW  │  DONE    │
│          │  (max:2) │  (max:3) │  (max:2) │          │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ • Bug #12│ • US-045 │ • US-041 │ • US-038 │ • US-035 │
│ • US-046 │ • US-044 │ • US-042 │          │ • US-036 │
│ • US-047 │          │ • Bug #11│          │ • Bug #10│
│ • US-048 │          │          │          │          │
│ • ...    │          │          │          │          │
└──────────┴──────────┴──────────┴──────────┴──────────┘
                         ↑
              WIP Limit = 3 : si une 4e carte 
              veut entrer en DEV, elle ATTEND
```

### WIP Limits (Limites de Travail en Cours)

La règle fondamentale de Kanban : **limiter le travail en cours**.

```
Sans WIP Limits (anti-pattern) :
  Dev A : commence ticket 1, bloqué → commence ticket 2,
          bloqué → commence ticket 3...
  Résultat : 10 tickets "in progress", 0 terminés

Avec WIP Limits :
  WIP Limit = 2 en DEV
  Dev A : commence ticket 1, bloqué
  → STOP : n'en commence pas un nouveau
  → Aide Dev B à finir son ticket
  → Ou supprime le blocage
  Résultat : moins de tickets simultanés, mais finis plus vite
```

> [!tip] "Stop starting, start finishing"
> Le mantra Kanban. Un ticket terminé a 100x plus de valeur qu'un ticket à moitié fait. Les WIP limits forcent l'équipe à finir avant de commencer.

### Métriques Kanban

**Cycle Time** : temps entre le début du développement d'un ticket et sa livraison.
```
Cycle Time = Date de livraison - Date de début DEV
Objectif : minimiser et rendre prévisible
```

**Lead Time** : temps entre la demande du client et la livraison.
```
Lead Time = Date de livraison - Date d'entrée dans le backlog
(inclut le temps d'attente dans le backlog)
```

**Throughput** : nombre de tickets livrés par unité de temps.
```
Throughput = 23 tickets livrés cette semaine
Utile pour prévoir : "si le throughput est de 20/semaine,
les 60 tickets restants seront livrés en 3 semaines"
```

**CFD (Cumulative Flow Diagram) :**
```
Tickets
  ^
  │    ╭────────────────────
  │   ╭│──────────────────
  │  ╭─│────────────────
  │ ╭──│──────────────
  │╭───│────────────
  └──────────────────────> Temps
  
  L'épaisseur de chaque bande = WIP dans cette colonne
  Une bande qui s'épaissit = goulot d'étranglement
```

---

## Scrumban

Hybride Scrum + Kanban, utilisé par de nombreuses équipes en pratique :
- Garder les cérémonies Scrum (planning, rétro) mais sans sprint figé
- Ajouter les WIP limits de Kanban
- Flux continu avec points de synchronisation réguliers

---

## Extreme Programming (XP)

XP est une méthode Agile plus prescriptive sur les **pratiques techniques**.

### Pratiques XP Clés

**Test-Driven Development (TDD) :**
```
Cycle RED → GREEN → REFACTOR :

1. RED : Écrire un test qui échoue (le test décrit le comportement souhaité)
2. GREEN : Écrire le minimum de code pour faire passer le test
3. REFACTOR : Améliorer le code sans changer son comportement

→ voir [[01 - Tests Unitaires et TDD]] pour les détails
```

**Pair Programming :**
```
Driver : écrit le code
Navigator : réfléchit à la conception, revoit, pose des questions

→ Rôles changent régulièrement (toutes les 25 min par exemple)
→ Bénéfices : revue de code en temps réel, transfer de connaissance,
  moins de bugs, meilleure conception
→ Contre-intuitif : 2 devs sur 1 clavier = presque aussi rapide que 2 séparés
  mais avec une bien meilleure qualité
```

**Continuous Integration (CI) :**
- Chaque développeur merge son code sur main plusieurs fois par jour
- Un pipeline automatique build et teste à chaque merge
- Si ça casse → priorité absolue de réparer avant toute autre chose

**Refactoring permanent :**
- Le code est toujours amélioré lors du cycle RED-GREEN-REFACTOR
- La dette technique est traitée en continu, pas accumulée

---

## Frameworks Agiles à l'Échelle

Pour les grandes organisations avec plusieurs équipes Scrum :

| Framework | Nb d'équipes | Complexité | Usage |
|---|---|---|---|
| **Scrum of Scrums** | 3-10 équipes | Faible | Coordination inter-équipes |
| **LeSS** (Large-Scale Scrum) | 2-8 équipes | Moyenne | Proche du Scrum classique |
| **SAFe** (Scaled Agile Framework) | 5-100+ équipes | Élevée | Grandes entreprises |
| **Spotify Model** | Variable | Spécifique | Inspiré Spotify (non copier) |

> [!warning] Le Spotify Model
> Beaucoup d'entreprises ont essayé de copier le modèle d'organisation de Spotify (squads, tribes, chapters, guilds). Spotify lui-même a déclaré en 2020 que le "Spotify Model" tel que décrit dans des présentations de 2012 n'existe plus. N'adoptez pas un framework organisationnel sans comprendre le contexte qui l'a créé.

---

## Outils

| Outil | Usage | Prix | Points forts |
|---|---|---|---|
| **Jira** | Gestion backlog, sprints, reporting | Payant (à partir de 8$/user) | Très complet, intégrations |
| **Linear** | Gestion d'issues moderne | Payant (gratuit limité) | UI excellent, rapide |
| **GitHub Projects** | Kanban + Issues GitHub | Gratuit | Intégré au code |
| **Notion** | Wiki + Kanban léger | Freemium | Flexible, tout-en-un |
| **Trello** | Kanban visuel simple | Freemium | Simplicité |
| **Azure DevOps** | Backlog + CI/CD Microsoft | Freemium | Intégration Microsoft |

---

## Métriques Agiles

### Vélocité

```
Vélocité = story points complétés par sprint

Sprint 1 : 23 pts
Sprint 2 : 31 pts
Sprint 3 : 28 pts
Sprint 4 : 29 pts
Vélocité moyenne : 28 pts

Utilisation : prévoir combien de sprints pour terminer le backlog
  Backlog restant : 150 pts ÷ 28 pts/sprint = ~5,4 sprints
```

> [!warning] La vélocité n'est pas une mesure de productivité
> Comparer la vélocité entre équipes est absurde — chaque équipe estime différemment. La vélocité d'une équipe n'est utile que pour **prévoir ses futures livraisons**, pas pour la comparer à une autre équipe ou pour "pousser" l'équipe à faire plus.

### Burn-Down Chart

```
Story Points
^
50│ •
45│   •
40│       •  ← idéal
35│         •
30│           •  ←  réel (en retard)
25│           •
20│           •
15│             •
10│               •
 5│                 •
 0└───────────────────────> Jours du sprint
  0  2  4  6  8  10  12  14

Ligne idéale : diagonal parfait du total au 0
Ligne réelle : indique si l'équipe est en avance ou en retard
```

---

## Formats de Rétrospective

### Start / Stop / Continue (classique)

Décrit ci-dessus dans la section cérémonies.

### 4Ls

```
LIKED         (ce qu'on a aimé)
LEARNED       (ce qu'on a appris)
LACKED        (ce qui manquait)
LONGED FOR    (ce dont on avait envie)
```

### Mad / Sad / Glad

```
MAD   (ce qui nous a rendus frustrés / en colère)
SAD   (ce qui nous a déçus / attristés)
GLAD  (ce qui nous a rendus contents / fiers)
```

### Étoile de Mer

```
        CONTINUER
           │
PLUS ──────┼────── COMMENCER
           │
        ARRÊTER
        
(+ une zone "MOINS DE" entre Continuer et Arrêter)
```

---

## Exercices Pratiques

### Exercice 1 — Rédiger des User Stories (30 min)
Pour une application de réservation de restaurant, rédigez 5 User Stories complètes avec :
- Le format "En tant que / Je veux / Afin de"
- 3 critères d'acceptation minimum par story (format Gherkin)
- Une estimation en story points (avec justification)
Vérifiez chaque story contre les critères INVEST.

### Exercice 2 — Sprint Planning (45 min)
Votre équipe a une vélocité de 30 points. Le sprint dure 2 semaines. Voici le backlog ordonné :
- US-01 : Inscription (8 pts)
- US-02 : Connexion (5 pts)
- US-03 : Profil utilisateur (8 pts)
- US-04 : Recherche de restaurant (13 pts)
- US-05 : Voir le menu (8 pts)
- US-06 : Ajouter au panier (5 pts)

Questions :
1. Quelles stories entrent dans le sprint ?
2. Quel Sprint Goal proposez-vous ?
3. Rédigez la Definition of Done pour ce sprint
4. Que faites-vous si, en milieu de sprint, le PO veut ajouter une story d'urgence ?

### Exercice 3 — Tableau Kanban (1h)
Créez un tableau Kanban pour votre projet de groupe à Holberton :
1. Définissez les colonnes (au minimum 4)
2. Définissez les WIP limits pour chaque colonne (justifiez)
3. Listez 10 tâches réelles et placez-les dans les bonnes colonnes
4. Identifiez un potentiel goulot d'étranglement et expliquez comment le résoudre

### Exercice 4 — Rétrospective (30 min)
Faites une rétrospective sur votre dernier projet ou votre semaine de formation :
1. Format Start / Stop / Continue
2. Au minimum 2 items par colonne (soyez honnêtes)
3. Choisissez 2 actions concrètes et assignez-les à une personne avec une date
4. Comparez avec le format 4Ls — qu'est-ce que chaque format révèle de différent ?

> [!info] Aller plus loin
> - **Livre** : "Scrum : The Art of Doing Twice the Work in Half the Time" - Jeff Sutherland (co-créateur de Scrum)
> - **Livre** : "Kanban" - David J. Anderson
> - **Certification** : Professional Scrum Master I (PSM I) — scrum.org (gratuit pour l'exam en ligne, 150 $)
> - **Simulation** : Scrum Poker en ligne (planning poker online) pour pratiquer l'estimation
> - **Outil gratuit** : GitHub Projects — parfait pour pratiquer Kanban sur vos projets Holberton
