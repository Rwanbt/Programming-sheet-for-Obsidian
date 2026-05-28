# Gestion de Projet Classique et ITIL

## Pourquoi la Gestion de Projet Classique ?

Les méthodes Agile (voir [[01 - Methodes Agiles Scrum et Kanban]]) ne sont pas universelles. Certains projets — infrastructures, migrations critiques, systèmes embarqués, projets gouvernementaux — nécessitent une planification détaillée, un suivi rigoureux et une documentation contractuelle. La gestion de projet classique fournit ce cadre.

> [!info] Complémentarité
> Un développeur qui ne connaît que l'Agile sera perdu dans une grande entreprise qui utilise ITIL pour sa gouvernance IT, PRINCE2 pour ses projets, et CMMI pour mesurer sa maturité. Ces référentiels coexistent dans le monde professionnel.

---

## Planification de Projet

### WBS — Work Breakdown Structure

La WBS (Décomposition Structurée des Travaux) est la décomposition hiérarchique du périmètre total du projet en livrables plus petits et mieux gérables.

```
PROJET : Déploiement Application Mobile v2.0
│
├── 1. GESTION DE PROJET
│   ├── 1.1 Plan de projet
│   ├── 1.2 Rapports de suivi hebdomadaires
│   └── 1.3 Clôture du projet
│
├── 2. ANALYSE ET CONCEPTION
│   ├── 2.1 Recueil des besoins
│   │   ├── 2.1.1 Interviews parties prenantes
│   │   └── 2.1.2 Benchmark concurrentiel
│   ├── 2.2 Spécifications fonctionnelles
│   └── 2.3 Architecture technique
│
├── 3. DÉVELOPPEMENT
│   ├── 3.1 Backend API
│   │   ├── 3.1.1 Service Authentification
│   │   ├── 3.1.2 Service Données utilisateur
│   │   └── 3.1.3 Service Notifications
│   ├── 3.2 Application Mobile iOS
│   └── 3.3 Application Mobile Android
│
├── 4. TESTS ET VALIDATION
│   ├── 4.1 Tests unitaires
│   ├── 4.2 Tests d'intégration
│   ├── 4.3 Tests de recette (UAT)
│   └── 4.4 Tests de performance
│
└── 5. DÉPLOIEMENT
    ├── 5.1 Déploiement production backend
    ├── 5.2 Soumission App Store
    └── 5.3 Formation utilisateurs
```

**Règle 100 % :** la WBS doit couvrir **tout** le périmètre du projet — ni plus, ni moins. Un livrable absent de la WBS ne sera pas fait (ou sera un travail "hors scope" facturé en supplément).

**Règle du niveau de décomposition :** s'arrêter quand le livrable est assez petit pour être :
- Estimable précisément
- Assignable à une personne ou équipe
- Livrable en moins de 2 semaines

### PBS — Product Breakdown Structure

La PBS décompose le **produit** (pas le travail). Utile pour clarifier ce qu'on livre.

```
PRODUIT : Système de gestion de stock
│
├── Application Web
│   ├── Interface d'administration
│   │   ├── Tableau de bord stocks
│   │   └── Gestion des fournisseurs
│   └── Interface entrepôt (tablette)
│
├── API Backend
│   ├── API REST
│   └── Webhooks
│
├── Base de données
│   ├── Schéma PostgreSQL
│   └── Procédures de migration
│
└── Infrastructure
    ├── Serveurs de production
    └── Système de sauvegarde
```

---

## Diagramme de Gantt

Le diagramme de Gantt représente les tâches du projet sur un axe temporel, avec leurs dépendances.

```
Projet Mobile v2.0 — Gantt simplifié

Tâche                    Semaine 1  2  3  4  5  6  7  8  9  10
─────────────────────────────────────────────────────────────────
1. Analyse besoins       ████████
2. Architecture          ████████████
3. Backend - Auth                ████████
4. Backend - Data                    ████████
5. Backend - Notif                       ████████
6. Mobile iOS                        ████████████████
7. Mobile Android                        ████████████████
8. Tests intégration                             ████████
9. Recette (UAT)                                     ████
10. Déploiement                                         ██

Jalons :
  ▲ Fin analyse (S2)
                ▲ Architecture validée (S3)
                                        ▲ Backend livré (S7)
                                                    ▲ Mise en production (S10)

Chemin critique : 1 → 2 → 3 → 4 → 5 → 8 → 9 → 10
```

**Concepts clés :**

- **Dépendances** : Fin-Début (FD), Début-Début (DD), Fin-Fin (FF), Début-Fin (DF)
- **Jalons** (Milestones) : événements sans durée qui marquent des dates importantes
- **Chemin critique** : séquence de tâches dont le retard retarde toute la fin du projet
- **Marge** (float) : temps de retard possible sans impacter la fin du projet

> [!warning] Le Gantt n'est pas une vérité absolue
> En pratique, aucun projet ne suit le Gantt initial à la lettre. Le Gantt est un **plan de référence** (baseline) — on le met à jour régulièrement et on compare l'avancement réel vs. planifié. Un Gantt qui n'est jamais mis à jour est inutile.

**Outils pour les diagrammes de Gantt :**
- Microsoft Project (standard entreprise, payant)
- GanttProject (gratuit, open-source)
- Smartsheet (cloud, payant)
- Jira Advanced Roadmaps (intégré à Jira)

---

## Méthode PERT / MPM

### PERT (Program Evaluation and Review Technique)

PERT est utilisé pour les projets à durée incertaine. Il utilise 3 estimations :

```
Pour chaque tâche :
  O = durée Optimiste (tout se passe parfaitement)
  M = durée la plus vraiseMblable (Most Likely)
  P = durée Pessimiste (tout se passe mal)

Durée PERT = (O + 4M + P) / 6

Exemple — Développement de l'API :
  O = 3 semaines (tout va bien)
  M = 5 semaines (scénario normal)
  P = 10 semaines (problèmes inattendus)
  
  Durée PERT = (3 + 4×5 + 10) / 6 = (3 + 20 + 10) / 6 = 33/6 = 5,5 semaines

Variance = ((P - O) / 6)² = ((10 - 3) / 6)² = (7/6)² = 1,36 semaines²
```

### MPM — Méthode des Potentiels Métra

Représentation du réseau de tâches avec les dépendances :

```
      ┌─────── B (4j) ──────┐
  ┌───┤                     ├───┐
  │   └─────── C (6j) ──────┘   │
A (3j)                           ├─ F (2j) ─ FIN
  │   ┌─────── D (5j) ──────┐   │
  └───┤                     ├───┘
      └─────── E (8j) ──────┘

Chemin critique = A → E → F = 3 + 8 + 2 = 13 jours
(E est critique : aucune marge)
B a 4 jours de marge (13 - 3 - 4 - 2 = 4)
```

---

## Estimation Financière

### ROI — Return on Investment

```
ROI = (Bénéfices nets - Coûts) / Coûts × 100

Exemple :
  Coût du projet : 150 000 €
  Économies annuelles estimées (automatisation) : 80 000 €/an
  Durée d'amortissement : 150 000 / 80 000 = 1,875 ans ≈ 22 mois
  ROI sur 3 ans : ((80 000 × 3 - 150 000) / 150 000) × 100 = 60 %
```

### TCO — Total Cost of Ownership

Le TCO inclut **tous** les coûts sur la durée de vie du système :

```
TCO sur 5 ans d'un système logiciel :

Coûts initiaux (CAPEX) :
  Développement          150 000 €
  Licences logicielles    20 000 €
  Infrastructure initiale 15 000 €
  Formation               10 000 €
  TOTAL INITIAL          195 000 €

Coûts récurrents (OPEX) × 5 ans :
  Maintenance (15 % du dev/an) × 5  112 500 €
  Licences logicielles/an × 5        50 000 €
  Infrastructure cloud/an × 5        60 000 €
  Support utilisateurs/an × 5        25 000 €
  TOTAL RÉCURRENTS                  247 500 €

TCO TOTAL SUR 5 ANS = 195 000 + 247 500 = 442 500 €
```

> [!warning] L'erreur classique
> Estimer seulement les coûts de développement. En réalité, les coûts de maintenance représentent souvent **70-80 % du coût total de vie** d'un logiciel. Ne jamais vendre/approuver un projet sans inclure le TCO complet.

### Budget Projet et Réserves

```
Estimation des coûts de base (bottom-up) :
  Phase 1 — Analyse : 15 000 €
  Phase 2 — Dev Backend : 40 000 €
  Phase 3 — Dev Mobile : 55 000 €
  Phase 4 — Tests : 20 000 €
  Phase 5 — Déploiement : 10 000 €
  TOTAL BASE : 140 000 €

  + Réserve de contingence (risques identifiés) : 10 % → 14 000 €
  + Réserve de management (risques inconnus) : 5 % → 7 000 €
  
  BUDGET TOTAL AUTORISÉ : 161 000 €
```

---

## Gestion des Risques

### Registre des Risques

Un registre des risques liste, évalue et planifie la réponse à chaque risque identifié.

| ID | Risque | Probabilité | Impact | Score (P×I) | Stratégie | Responsable |
|---|---|---|---|---|---|---|
| R01 | Départ d'un développeur clé | Moyen (3) | Élevé (4) | 12 | Mitigation : documentation, pair programming | Chef projet |
| R02 | Retard API tierce (paiement) | Élevé (4) | Élevé (4) | 16 | Contournement : mock de l'API en parallèle | Tech Lead |
| R03 | Rejet App Store iOS | Faible (2) | Élevé (4) | 8 | Mitigation : review préliminaire des guidelines | Dev iOS |
| R04 | Dépassement budget serveur | Moyen (3) | Moyen (3) | 9 | Surveillance : alertes budget cloud | DevOps |
| R05 | Retard client pour la recette | Élevé (4) | Moyen (3) | 12 | Atténuation : planifier tôt, rappels | Chef projet |

**Matrice Probabilité × Impact :**
```
Impact
  │
4 │  R03(8)  R02(16)
  │
3 │  R04(9)  R01(12) R05(12)
  │
2 │
  │
1 │
  └──────────────────────
     1    2    3    4     Probabilité

Zone rouge (>12) : R02 → action immédiate
Zone orange (6-12) : R01, R04, R05, R03 → surveillance
Zone verte (<6) : acceptable
```

**4 Stratégies de réponse aux risques :**
1. **Éviter** : modifier le plan pour éliminer le risque (changer de fournisseur)
2. **Transférer** : passer le risque à une autre partie (assurance, sous-traitance)
3. **Atténuer** : réduire la probabilité ou l'impact (prototype, tests précoces)
4. **Accepter** : décider de ne rien faire (risque mineur ou trop coûteux à traiter)

---

## ITIL v4 — IT Infrastructure Library

ITIL est le référentiel de bonnes pratiques pour la **gestion des services IT**. ITIL v4 (2019) est la version actuelle.

> [!info] ITIL n'est pas un framework de développement
> ITIL concerne la **gestion des services IT en production** : comment répondre à un incident, comment gérer un changement sur un système en production, comment mesurer la qualité du service. C'est le quotidien des équipes IT Ops, Service Desk, et DevOps.

### Concept Fondamental : La Chaîne de Valeur des Services

```
Demande / Opportunité
         ↓
┌────────────────────────────────────────────────────────┐
│              CHAÎNE DE VALEUR DES SERVICES              │
│                                                         │
│  Engager ──> Planifier ──> Concevoir/Transitionner     │
│       ↓                           ↓                    │
│  Obtenir/Construire ──────> Fournir/Soutenir            │
│       └────────────────────────────┘                   │
│                    ↑                                    │
│             Améliorer (continu)                         │
└────────────────────────────────────────────────────────┘
         ↓
  Valeur pour le client
```

### Les 4 Dimensions ITIL v4

| Dimension | Description |
|---|---|
| **Organisations et personnes** | Structure, culture, rôles, responsabilités |
| **Information et technologie** | Systèmes d'information, outils, données |
| **Partenaires et fournisseurs** | Contrats, SLA, relations externes |
| **Flux de valeur et processus** | Comment le travail est accompli |

### Les Pratiques ITIL Clés (sur 34 au total)

#### Incident Management (Gestion des Incidents)

Un **incident** est une interruption non planifiée ou une dégradation d'un service IT.

```
Processus de gestion d'incident :

1. DÉTECTION
   Monitoring automatique, appel utilisateur, signalement
   
2. ENREGISTREMENT
   Ticket créé avec : heure, impact, priorité, description

3. CLASSIFICATION & PRIORISATION
   P1 (Critique) : service complètement down, tous les utilisateurs impactés
   P2 (Élevée)   : dégradation majeure, partie des utilisateurs impactés
   P3 (Moyenne)  : impact limité, workaround disponible
   P4 (Basse)    : cosmétique, peu d'impact

4. DIAGNOSTIC & RÉSOLUTION
   Level 1 : Service Desk (solutions connues, base de connaissance)
   Level 2 : Équipe technique (analyse approfondie)
   Level 3 : Expert / éditeur (cas complexes)

5. CLÔTURE
   Vérification que le service est restauré
   Satisfaction utilisateur confirmée

SLA typiques :
  P1 : réponse < 15 min, résolution < 4h
  P2 : réponse < 30 min, résolution < 8h
  P3 : réponse < 2h, résolution < 3 jours
  P4 : réponse < 24h, résolution < 1 semaine
```

#### Problem Management (Gestion des Problèmes)

Un **problème** est la cause sous-jacente d'un ou plusieurs incidents.

```
Incident ≠ Problème :
  Incident : "Le site est lent ce matin"
  Problème : "Pourquoi est-il lent ? Fuites mémoire dans le serveur X"

Incident Management = éteindre l'incendie (rapide)
Problem Management  = trouver ce qui a mis le feu (analyse)

Méthode des 5 Pourquoi :
  Pourquoi le site est lent ?
  → La base de données est surchargée
  Pourquoi est-elle surchargée ?
  → Des requêtes non optimisées tournent en boucle
  Pourquoi tournent-elles en boucle ?
  → Le cache Redis est tombé
  Pourquoi est-il tombé ?
  → Le disque est plein
  Pourquoi est-il plein ?
  → Les logs ne sont pas purgés automatiquement (CAUSE RACINE)
  
  Solution durable : ajouter une politique de purge automatique des logs
```

#### Change Management (Gestion des Changements)

Tout changement sur un système en production doit être contrôlé pour minimiser les risques.

```
Types de changements :

Standard (pre-approuvé) :
  Procédure connue, risque faible
  Exemple : ajout d'un utilisateur, mise à jour de mot de passe
  → Pas de CAB, procédure documentée

Normal :
  Doit passer par le Change Advisory Board (CAB)
  Exemple : déploiement d'une nouvelle version, migration de DB
  → RFC (Request for Change) → revue CAB → approbation → déploiement
  
Urgent (Emergency Change) :
  Correction critique, ne peut pas attendre la réunion CAB
  → eCAB (Emergency CAB) ou approbation accélérée
  → Documentation après coup obligatoire

RFC (Request for Change) contient :
  - Description du changement
  - Raison du changement
  - Impact potentiel
  - Plan de retour arrière (rollback plan)
  - Planning
  - Tests prévus
```

#### Service Desk

Point de contact unique entre les utilisateurs et l'IT.

```
Types de Service Desk :
  Local       : physiquement sur site
  Centralisé  : un seul centre pour toute l'organisation
  Virtuel     : équipes distribuées géographiquement
  Follow the Sun : 24/7 via relais de fuseaux horaires

Canaux :
  Téléphone, email, portail self-service, chat, sur site

KPIs Service Desk :
  FCR (First Call Resolution) : % incidents résolus au premier contact → cible : 70-80 %
  MTTR (Mean Time to Resolve) : temps moyen de résolution
  NPS (Net Promoter Score) : satisfaction utilisateur
  Volume d'incidents par mois : tendance à la baisse = amélioration du service
```

---

## CMMI — Capability Maturity Model Integration

CMMI mesure la **maturité des processus** d'une organisation logicielle sur 5 niveaux.

```
NIVEAU 5 — OPTIMISANT
  Amélioration continue, innovation et optimisation des processus
  "On mesure, on améliore, on innove continuellement"

NIVEAU 4 — GÉRÉ QUANTITATIVEMENT
  Processus mesurés et contrôlés statistiquement
  "On mesure tout et on pilote avec des données"

NIVEAU 3 — DÉFINI
  Processus standards définis à l'échelle de l'organisation
  "Tout le monde suit le même processus documenté"

NIVEAU 2 — GÉRÉ
  Processus planifiés, exécutés, mesurés, contrôlés
  "Les projets ont des processus de base définis"

NIVEAU 1 — INITIAL
  Processus ad hoc, chaotiques, succès individuel
  "Ça dépend des individus, pas des processus"
```

En pratique, la plupart des organisations visent le niveau 3. Les niveaux 4 et 5 sont réservés aux organisations avec des exigences de qualité très élevées (aérospatiale, défense, médical).

---

## ISO 20000 et Gouvernance IT

**ISO 20000** : norme internationale pour la gestion des services IT. C'est ITIL formalisé et certifiable.

**COBIT (Control Objectives for Information Technologies)** : framework de gouvernance IT centré sur la valeur business et le risque. Complète ITIL avec la dimension gouvernance.

```
Différence ITIL vs COBIT :

ITIL = Comment opérer les services IT (opérationnel)
COBIT = Comment gouverner les investissements IT (stratégique)

ITIL répond à : "Comment gérer un incident ?"
COBIT répond à : "Nos investissements IT créent-ils de la valeur ? Les risques sont-ils maîtrisés ?"
```

---

## RGPD pour les Développeurs

Le **RGPD** (Règlement Général sur la Protection des Données, GDPR en anglais) est obligatoire pour toute organisation qui traite des données de résidents de l'Union Européenne. Entré en vigueur le 25 mai 2018.

> [!warning] Le RGPD s'applique à VOUS
> Même si votre startup est aux USA, si vous traitez des données d'utilisateurs européens, vous êtes soumis au RGPD. La méconnaissance de la loi n'est pas une excuse devant la CNIL.

### Données Personnelles

**Donnée personnelle** = toute information permettant d'identifier directement ou indirectement une personne physique.

```
DONNÉES PERSONNELLES :
  Directes : nom, prénom, email, numéro de téléphone, adresse IP
  Indirectes : cookie, identifiant de session, identifiant publicitaire
  Sensibles (catégorie spéciale, protection renforcée) :
    → origine ethnique, opinions politiques, croyances religieuses
    → données de santé, données génétiques, données biométriques
    → orientation sexuelle, données judiciaires
```

### Les Droits des Utilisateurs

| Droit | Description | Délai de réponse |
|---|---|---|
| **Accès** | Obtenir une copie de toutes ses données | 1 mois |
| **Rectification** | Corriger des données inexactes | 1 mois |
| **Effacement** (droit à l'oubli) | Supprimer ses données | 1 mois |
| **Portabilité** | Recevoir ses données dans un format structuré | 1 mois |
| **Opposition** | Refuser le traitement à des fins marketing | Immédiat |
| **Limitation** | Limiter le traitement en cas de litige | Variable |

### Implémentation Technique du RGPD

```python
# Exemple : API de suppression de compte (droit à l'oubli)
from datetime import datetime

def supprimer_compte_utilisateur(user_id: str) -> dict:
    """
    Implémente le droit à l'oubli RGPD.
    Pseudonymise les données d'audit au lieu de supprimer pour la traçabilité légale.
    """
    
    # 1. Supprimer les données personnelles directes
    user = db.users.find_one({'_id': user_id})
    
    # 2. Pseudonymiser les logs d'audit (obligation légale de conservation)
    db.audit_logs.update_many(
        {'user_id': user_id},
        {'$set': {
            'user_id': f"DELETED_{hash(user_id)}",
            'user_email': '[SUPPRIMÉ]',
            'deletion_date': datetime.utcnow()
        }}
    )
    
    # 3. Supprimer le compte principal
    db.users.delete_one({'_id': user_id})
    
    # 4. Supprimer les données dans les services tiers
    newsletter_service.unsubscribe(user['email'])
    analytics_service.delete_user_data(user_id)
    
    # 5. Logger la suppression (traçabilité RGPD)
    db.gdpr_log.insert_one({
        'action': 'ACCOUNT_DELETED',
        'user_id_hash': hash(user_id),
        'timestamp': datetime.utcnow(),
        'request_source': 'user_request'
    })
    
    return {'status': 'deleted', 'timestamp': datetime.utcnow().isoformat()}
```

**DPO (Data Protection Officer)** : obligatoire pour les organisations qui traitent des données sensibles à grande échelle. Responsable de la conformité RGPD.

### Privacy by Design

Les 7 principes de Privacy by Design (intégrer la vie privée dès la conception) :

```
1. PROACTIF, PAS RÉACTIF
   Anticiper les problèmes de vie privée avant qu'ils n'arrivent

2. PAR DÉFAUT
   Le paramètre le plus protecteur doit être le paramètre par défaut
   ❌ Opt-out (données partagées par défaut, case à décocher)
   ✅ Opt-in (données non partagées par défaut, case à cocher)

3. INTÉGRÉ À LA CONCEPTION
   La protection des données fait partie de l'architecture, pas ajoutée après

4. FONCTIONNALITÉ TOTALE
   Privacy ET service : pas obligé de choisir

5. SÉCURITÉ DE BOUT EN BOUT
   Protection pendant toute la durée de conservation

6. VISIBILITÉ ET TRANSPARENCE
   Les pratiques sont transparentes et vérifiables

7. RESPECT DE LA VIE PRIVÉE DE L'UTILISATEUR
   Centrée sur l'utilisateur, pas sur l'organisation
```

**Sanctions RGPD :**
- Violations mineures : jusqu'à 10 millions € ou 2 % du CA mondial
- Violations graves : jusqu'à 20 millions € ou 4 % du CA mondial
- Exemples réels : Meta — 1,2 milliard € (2023), Amazon — 746 millions € (2021)

---

## Green IT et Éco-Conception

L'industrie numérique représente **3-4 % des émissions mondiales de CO2** (comparable à l'aviation). En France, c'est 2,5 % des émissions et la tendance est à la hausse.

### Impact Carbone du Numérique

```
Sources d'émissions numériques :

FABRICATION : 45-70 % (terminaux utilisateurs)
  Smartphone (fabrication) : ~70 kg CO2e
  Ordinateur portable : ~300 kg CO2e
  Serveur : ~2 000 kg CO2e

USAGE : 30-55 %
  1 email avec pièce jointe : ~50 g CO2e
  1h de streaming vidéo HD : ~36 g CO2e
  1 recherche Google : 0,3 g CO2e
  Entraînement GPT-3 : ~300 tonnes CO2e
```

### Techniques d'Éco-Conception Logicielle

**Lazy Loading :**
```html
<!-- ❌ Charger toutes les images immédiatement -->
<img src="photo.jpg" alt="Photo">

<!-- ✅ Charger uniquement quand visible -->
<img src="photo.jpg" alt="Photo" loading="lazy">
```

**Optimisation des Images :**
```bash
# Convertir en WebP (30-50 % plus léger que JPEG)
cwebp -q 80 photo.jpg -o photo.webp

# Optimiser les JPEG
jpegoptim --max=85 photo.jpg

# Utiliser les bons formats :
# Photos → WebP / JPEG
# Icônes → SVG
# Sprites → CSS sprites
```

**Cache Efficace :**
```nginx
# nginx.conf — cache agressif pour les assets statiques
location /static/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    # "immutable" = le navigateur ne re-vérifie jamais (hash dans le nom de fichier)
}
```

**Code Efficace :**
```python
# ❌ Chargement de toutes les données en mémoire
def get_all_users():
    return db.query("SELECT * FROM users")  # 1 million de lignes en RAM

# ✅ Pagination et streaming
def get_users_page(page: int, size: int = 50):
    offset = page * size
    return db.query("SELECT * FROM users LIMIT ? OFFSET ?", (size, offset))
```

**Hébergement Vert :**

| Hébergeur | Énergie renouvelable | Label | Score Greenpeace |
|---|---|---|---|
| **OVHcloud** | 100 % renouvelable (certifié) | ISO 14001 | A |
| **Scaleway** | 100 % renouvelable | HDS | A |
| **Infomaniak** | 100 % renouvelable + neutre CO2 | ISO 14001 | A+ |
| **AWS** | 85 % (objectif 100 % en 2025) | Variable par région | B+ |
| **Google Cloud** | 100 % neutre CO2 depuis 2007 | Carbon neutral | A |

**Éco-index :** outil pour mesurer l'impact d'une page web.
```
Score Éco-index = algorithme basé sur :
  - Nombre de requêtes HTTP
  - Taille de la page (octets)
  - Nombre d'éléments DOM

A → G (comme l'étiquette énergie des appareils électroménagers)
Cible : score A ou B
```

**Liens vers [[07 - DevSecOps]]** pour la CI/CD verte et les pratiques de déploiement durable.

---

## Exercices Pratiques

### Exercice 1 — WBS et Gantt (1h)
Pour votre projet de fin de formation Holberton, créez :
1. Une WBS complète à 3 niveaux (vérifier la règle 100 %)
2. Un diagramme de Gantt pour les 4 dernières semaines
3. Identifier le chemin critique
4. Calculer les marges pour chaque tâche non critique

### Exercice 2 — Registre des risques (45 min)
Toujours pour votre projet final, créez un registre des risques avec :
1. Minimum 6 risques identifiés (techniques + humains + organisationnels)
2. Évaluation Probabilité × Impact pour chacun
3. Une stratégie de réponse pour chaque risque
4. Un responsable assigné
5. Dessinez la matrice des risques

### Exercice 3 — Simulation ITIL (30 min)
Simulez le processus de gestion d'incident pour ce scénario :
"Le lundi matin à 8h30, l'application de paie est inaccessible pour 500 employés. La paie mensuelle doit partir à midi."

Répondez à :
1. Quelle priorité assignez-vous ? Pourquoi ?
2. Qui contactez-vous et dans quel ordre ?
3. Quelles sont vos premières actions de diagnostic ?
4. Quel est votre plan de communication ?
5. Comment documentez-vous la cause racine après résolution ?

### Exercice 4 — Audit RGPD (1h)
Auditez l'application Todo développée dans le cours Flutter :
1. Listez toutes les données personnelles potentiellement collectées
2. Pour chaque donnée : quelle base légale (consentement, contrat, obligation légale, intérêt légitime) ?
3. Rédigez une politique de confidentialité simplifiée (5 sections)
4. Implémentez en pseudo-code le endpoint DELETE /account qui respecte le droit à l'oubli
5. Identifiez 3 principes de Privacy by Design non respectés et proposez des corrections

### Exercice 5 — Éco-conception (30 min)
Analysez le site web de votre école :
1. Mesurez l'Éco-index (ecoindex.fr)
2. Identifiez les 5 plus grandes sources d'inefficacité
3. Proposez 5 optimisations concrètes (avec estimation de gain)
4. Calculez le ROI environnemental si votre site reçoit 10 000 visites/mois

> [!info] Certifications professionnelles
> - **ITIL Foundation** : certification d'entrée, reconnue mondialement, ~350 €
> - **PMP** (Project Management Professional) : certification chef de projet, ~600 €, 36 mois d'expérience requis
> - **DPO Certifié** : formation CNIL ou organismes privés, ~1 500 €, obligatoire dans certaines entreprises
> - **Green IT** : label GreenIT.fr, formation éco-conception numérique responsable
