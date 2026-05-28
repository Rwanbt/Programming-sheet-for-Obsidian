# 09 - Incident Response et Postmortems

> [!info] La résilience comme discipline d'ingénierie
> Les incidents de production ne sont pas une exception — ils sont inévitables. La différence entre une équipe mature et une équipe qui court après les incendies : la première a un **process défini**, une **culture blame-free**, et tire des apprentissages systématiques de chaque incident. Ce cours couvre la réponse aux incidents, le postmortem blameless, le SBOM et le Threat Modeling.

---

## Partie 1 — SRE : Site Reliability Engineering

### Philosophie Google

Le **Site Reliability Engineering** (SRE) est une discipline née chez Google en 2003, popularisée par le livre *Site Reliability Engineering* de 2016. L'idée centrale : traiter les opérations comme un **problème d'ingénierie logicielle**, pas comme un art opaque géré par des "admins système".

**La tension fondamentale :**

```
Développement (Dev)           Operations (Ops)
───────────────               ───────────────
Veut déployer vite ──────────── Veut la stabilité
Innover, changer               Ne pas casser
Nouvelles features             Pas d'incidents

SRE résout cette tension avec :
→ Un contrat explicite (Error Budget)
→ Des métriques objectives (SLI/SLO/SLA)
→ Toil automatisé (moins de travail manuel répétitif)
```

### SLI, SLO, SLA — les différences

Ces trois acronymes sont souvent confondus. Voici leurs rôles distincts :

**SLI — Service Level Indicator**

Un SLI est une **mesure quantitative** d'un aspect du service. C'est la donnée brute.

Exemples de SLI :
- Disponibilité : `(requêtes réussies / total requêtes) × 100%`
- Latence : `p99 de la latence des requêtes`
- Débit : `requêtes traitées par seconde`
- Taux d'erreur : `(requêtes en erreur / total requêtes) × 100%`
- Fraîcheur des données : `temps écoulé depuis la dernière mise à jour`

**SLO — Service Level Objective**

Un SLO est un **objectif interne** pour un SLI. C'est votre cible.

Exemples :
- "La disponibilité sera ≥ 99.5% sur 30 jours glissants" → SLI: disponibilité, SLO: 99.5%
- "99% des requêtes auront une latence < 200ms" → SLI: latence p99, SLO: 200ms
- "Le taux d'erreur sera < 0.1%" → SLI: error rate, SLO: 0.1%

**SLA — Service Level Agreement**

Un SLA est un **contrat légal** avec les clients, incluant des pénalités financières en cas de non-respect.

```
SLO vs SLA :
  SLO : 99.9% de disponibilité (objectif interne ambitieux)
  SLA : 99.5% de disponibilité (engagement contractuel avec pénalités)

Le SLO est plus strict que le SLA → marge de sécurité avant les pénalités.
Si le SLO est violé, l'équipe réagit AVANT que le SLA ne soit violé.
```

| | SLI | SLO | SLA |
|--|-----|-----|-----|
| Définition | Indicateur de mesure | Objectif interne | Contrat avec pénalités |
| Public | Équipe technique | Équipe produit + tech | Clients |
| Conséquence si violé | Alerte interne | Revue / action corrective | Pénalités financières, crédits |
| Tyique | p99 latence = 187ms | p99 latence < 200ms | p99 latence < 500ms |

### Error Budget

L'error budget est la quantité d'indisponibilité "tolérée" selon le SLO sur une période donnée.

```
SLO : 99.9% de disponibilité sur 30 jours

Temps total disponible : 30 jours × 24h × 60min = 43 200 minutes

Error Budget = (100% - 99.9%) × 43 200 = 0.1% × 43 200 = 43.2 minutes

Interprétation :
  → L'équipe a "droit" à 43 minutes d'indisponibilité par mois
  → Si l'error budget est épuisé : STOP les déploiements de nouvelles features
  → Si l'error budget est intact : accélérer les déploiements (on peut se permettre le risque)
```

**L'error budget aligne Dev et Ops** : les deux équipes partagent le même budget. Si les développeurs déploient trop vite et causent des incidents, le budget s'épuise et les déploiements s'arrêtent — pénalité pour les deux.

### Toil — le travail à éliminer

Le **toil** (ou "labeur") désigne le travail opérationnel manuel, répétitif, automatisable, qui ne génère pas de valeur durable.

**Caractéristiques du toil :**
- Manuel : un humain fait ce qu'un script pourrait faire
- Répétitif : même action effectuée encore et encore
- Automatisable : il existe une solution technique
- Tactique : ne réduit pas le backlog futur, ne fait que "tenir"
- Sans valeur durable : quand c'est fini, rien n'a changé structurellement

**Exemples de toil :**
- Relancer manuellement un service qui crashe toutes les nuits
- Copier-coller des logs pour créer un rapport hebdomadaire
- Répondre manuellement aux mêmes alertes PagerDuty en appliquant le même runbook

**Règle SRE Google** : les SREs ne doivent pas passer plus de 50% de leur temps en toil. L'autre 50% est dédié à l'ingénierie (automatisation, amélioration de la fiabilité).

---

## Partie 2 — Cycle de vie d'un incident

### Les 6 phases

```
┌─────────────────────────────────────────────────────────────────────┐
│                   CYCLE DE VIE D'UN INCIDENT                        │
│                                                                     │
│  1. DÉTECTION      2. TRIAGE       3. COMMUNICATION                 │
│  Alerte monitoring  Évaluer        Status page update               │
│  Rapport utilisateur l'impact      Stakeholders notifiés            │
│  → Créer l'incident → Assigner IC  → War room ouverte               │
│                                                                     │
│  4. MITIGATION     5. RÉSOLUTION   6. POSTMORTEM                    │
│  Réduire l'impact  Corriger la     Blameless review                 │
│  Rollback/workaround cause racine  Action items                     │
│  → Impact réduit   → Service OK   → Apprentissages partagés         │
└─────────────────────────────────────────────────────────────────────┘
```

**Phase 1 — Détection**

Sources de détection :
- **Alerting automatique** : Prometheus + Alertmanager, Datadog, Grafana alerts
- **Rapport utilisateur** : support ticket, tweet, Slack du client
- **Détection proactive** : Synthetic monitoring (tests automatiques qui simulent des utilisateurs)
- **Dépendance en cascade** : un autre service détecte qu'un appel API échoue

**Phase 2 — Triage**

Questions à répondre immédiatement :
1. Quel est l'impact utilisateur ? (combien d'utilisateurs, quelle fonctionnalité)
2. Quelle est la tendance ? (empire, stable, s'améliore)
3. Y a-t-il un changement récent qui pourrait être la cause ? (déploiement des 2 dernières heures)
4. Quelle est la sévérité ? → assigne la priorité

**Phase 3 — Communication** (pendant tout l'incident)

Jamais de silence radio. Communiquer même quand on n'a rien de nouveau : "Toujours en investigation, mise à jour dans 30 min".

**Phase 4 — Mitigation**

L'objectif est de **réduire l'impact**, pas nécessairement de corriger la cause racine.

Tactiques de mitigation :
- **Rollback** : revenir à la version précédente stable
- **Feature flag** : désactiver la fonctionnalité incriminée
- **Traffic shifting** : rediriger le trafic vers une instance saine
- **Rate limiting** : réduire la charge sur le système en souffrance
- **Cache** : servir du contenu en cache plutôt que la base de données

**Phase 5 — Résolution**

Confirmer que le service est revenu à la normale :
- Les métriques (SLI) sont revenues dans les seuils normaux
- Les alertes sont résolues
- Les utilisateurs confirment que le service fonctionne
- Documenter l'heure de résolution

**Phase 6 — Postmortem**

Obligatoire pour tout incident de sévérité S1/S2. Optionnel pour S3/S4 mais fortement recommandé. Voir Partie 5.

---

### Niveaux de sévérité

| Sévérité | Nom courant | Définition | Exemples concrets | Temps de réponse |
|----------|-------------|------------|-------------------|-----------------|
| **S1** | Critical | Service totalement indisponible ou perte de données | Toute l'application down, BDD corrompue, brèche de sécurité active | < 15 min |
| **S2** | Major | Fonctionnalité critique dégradée, impact large | Paiements en échec, API publique 50% d'erreurs, login impossible | < 30 min |
| **S3** | Minor | Fonctionnalité non critique impactée, workaround possible | Rapports lents, feature secondaire indisponible, erreurs isolées | < 4h |
| **S4** | Low | Impact marginal, pas d'urgence | Alerte cosmétique, log erroné, 1 utilisateur impacté sur 1M | < 24h |

---

## Partie 3 — Rôles pendant un incident

### L'Incident Commander (IC)

L'Incident Commander est le **chef d'orchestre** de la réponse à l'incident. Son rôle est de **coordonner**, pas de déboguer.

**Responsabilités de l'IC :**
- Déclarer le niveau de sévérité
- Ouvrir le war room (Zoom, Google Meet, Slack huddle)
- Assigner les rôles aux responders
- Maintenir la communication externe (status page, stakeholders)
- Prendre les décisions (rollback oui/non, escalade oui/non)
- Documenter en temps réel dans le document d'incident
- Clôturer l'incident et déclencher le postmortem

**Ce que l'IC ne fait PAS :**
- Déboguer lui-même le code
- Reproduire le bug
- Regarder les logs lui-même (il délègue)

> [!tip] Séparation des rôles
> Pendant un incident, il est impossible de déboguer efficacement ET de coordonner efficacement en même temps. L'IC doit résister à l'envie de "mettre les mains dans le cambouis".

### Les Responders

Les responders (techniciens) sont assignés par l'IC :

**Technical Lead** : responsable de l'investigation technique et de la résolution. Coordonne les ingénieurs en debug.

**Communications Lead** : rédige les mises à jour de la status page, répond aux stakeholders, gère la communication externe.

**Subject Matter Expert (SME)** : expert du domaine impacté (database admin si c'est un problème BDD, senior backend si c'est un bug applicatif).

**Scribe** : documente tout en temps réel dans le document d'incident partagé (timeline, hypothèses, actions, décisions).

---

## Partie 4 — Communication de crise

### Status Page

Une status page est la **source de vérité publique** sur l'état du service pendant un incident. Elle doit être indépendante du service principal (si le service est down, la status page doit rester accessible).

**Outils :**
- **Statuspage.io** (Atlassian) — leader du marché
- **Cachet** — open-source, auto-hébergé
- **Instatus** — moderne, moins cher
- **GitHub Pages** — pour les projets open-source

**Structure d'une mise à jour de status page :**

```
[TITRE] : Dégradation des performances — API de paiement
[STATUT] : Investigating / Identified / Monitoring / Resolved
[HEURE] : 2024-01-15 14:32 UTC

Nous enquêtons actuellement sur une dégradation des performances
affectant notre API de paiement. Les utilisateurs peuvent observer
des délais allongés lors du traitement des transactions.

Prochaine mise à jour : 15h00 UTC
```

**Cycle de mise à jour :**
- S1 : mise à jour toutes les 15 minutes minimum
- S2 : mise à jour toutes les 30 minutes
- S3/S4 : mise à jour toutes les heures

### Templates de communication

**Notification initiale (interne, Slack #incidents) :**

```
🚨 INCIDENT DÉCLARÉ — S2

Titre    : Échecs des paiements Stripe — taux d'erreur 35%
IC       : @alice
Technical: @bob @charlie
Heure    : 2024-01-15 14:28 UTC
Statut   : Investigating

War Room : [lien Zoom]
Doc      : [lien Google Doc incident]
Status P : https://status.monapp.com/incidents/42
```

**Notification externe (stakeholders, clients impactés) :**

```
Objet : Dégradation de service — Mise à jour 14:35 UTC

Nous rencontrons actuellement un problème affectant [fonctionnalité].
Nos équipes ont été alertées et travaillent activement à la résolution.

Impact : [description précise — qui est touché, qu'est-ce qui ne fonctionne pas]
Workaround : [si disponible — ex: "utilisez la méthode de paiement alternative"]
Prochaine mise à jour : [heure]

Nous vous remercions de votre patience.
[Nom], Équipe Ingénierie
```

**Annonce de résolution :**

```
✅ RÉSOLU — 2024-01-15 16:12 UTC

L'incident affectant l'API de paiement est résolu.
Durée totale : 1h44

Cause identifiée : migration de BDD ayant introduit un lock excessif
Action corrective appliquée : rollback de la migration, index recréé

Un postmortem sera publié dans 5 jours ouvrés.
Merci pour votre compréhension.
```

---

## Partie 5 — Runbooks et Playbooks

### Qu'est-ce qu'un Runbook ?

Un **runbook** est un document technique précis décrivant comment exécuter une procédure opérationnelle spécifique. Il est écrit à l'avance, en dehors de tout incident, quand tout le monde est calme.

**Caractéristiques d'un bon runbook :**
- Écrit pour une personne en situation de stress (pas pour quelqu'un qui a 2 heures)
- Pas d'ambiguïté : "run `kubectl rollout undo deployment/api`", pas "rollback le déploiement"
- Testé régulièrement (chaos engineering, game day)
- Versionné avec le code
- Liens vers les outils, dashboards, contacts

**Exemple de runbook : API haute latence**

```markdown
# Runbook : Latence API élevée (p99 > 500ms)

## Déclencheur
Alerte : `api_latency_p99_high` dans PagerDuty

## Diagnostic rapide (5 minutes max)
1. Ouvrir le dashboard Grafana : https://grafana.internal/d/api-latency
2. Vérifier si la latence est globale ou par endpoint
   - Global → problème infrastructure (BDD, réseau)
   - Par endpoint → problème code spécifique
3. Vérifier les déploiements récents :
   `kubectl rollout history deployment/api --namespace production`
4. Vérifier les métriques BDD :
   `kubectl exec -it postgres-primary-0 -- psql -U app -c "SELECT * FROM pg_stat_activity WHERE wait_event_type IS NOT NULL;"`

## Actions selon la cause identifiée
### Cas A : Déploiement récent (< 2h)
  1. Rollback immédiat :
     `kubectl rollout undo deployment/api --namespace production`
  2. Vérifier que la latence revient à la normale (< 5 min)
  3. Ouvrir un incident S2 si ce n'est pas déjà fait

### Cas B : Contention BDD
  1. Identifier les requêtes lentes :
     `SELECT query, calls, total_time/calls AS avg_time FROM pg_stat_statements ORDER BY avg_time DESC LIMIT 10;`
  2. Si requête spécifique : activer le query cache ou killer la requête
     `SELECT pg_cancel_backend(<pid>);`
  3. Si contention de locks : identifier et libérer
     `SELECT pg_terminate_backend(blocking_pid) FROM pg_blocking_pids(<waiting_pid>);`

### Cas C : Pic de trafic
  1. Vérifier le scaling automatique :
     `kubectl get hpa --namespace production`
  2. Scale manuellement si HPA ne suit pas :
     `kubectl scale deployment/api --replicas=10 --namespace production`

## Escalade
Après 20 minutes sans mitigation visible → escalader à @on-call-senior
Contacter DBA si problème BDD persiste → @dba-on-call

## Contacts
- On-call Senior: PagerDuty escalation (auto)
- DBA: @jean-dba (Slack) ou +33 6 12 34 56 78
```

### Playbook vs Runbook

| | Runbook | Playbook |
|--|---------|----------|
| Niveau | Tactique, procédural | Stratégique |
| Granularité | Étapes précises, commandes exactes | Décisions de haut niveau |
| Usage | "Comment faire X" | "Quand faire X, Y ou Z" |
| Exemple | "Commandes pour rollback Kubernetes" | "Stratégie de réponse aux incidents de sécurité" |

---

## Partie 6 — Outils de gestion d'incidents

| Outil | Spécialité | Points forts |
|-------|------------|--------------|
| **PagerDuty** | Alerting et on-call management | Escalade configurable, intégrations nombreuses, mobile excellent |
| **OpsGenie** (Atlassian) | Alerting et on-call | Intégration native Jira, alertes multi-canal |
| **VictorOps** (Splunk) | Collaboration en incident | Chronologie collaborative, interface de war room |
| **FireHydrant** | Gestion d'incident end-to-end | Automatisation runbooks, postmortem intégré |
| **Rootly** | Gestion d'incident moderne | Excellent workflow Slack-first, postmortem IA |
| **Grafana OnCall** | Open-source, self-hosted | Gratuit, intégré Grafana |

**Intégration type PagerDuty avec Prometheus :**

```yaml
# alertmanager.yml — envoi vers PagerDuty pour les alertes critiques
route:
    group_by: ['alertname', 'cluster']
    group_wait: 30s
    group_interval: 5m
    repeat_interval: 12h
    receiver: 'pagerduty-critical'

    routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
    - match:
        severity: warning
      receiver: 'slack-warnings'

receivers:
- name: 'pagerduty-critical'
  pagerduty_configs:
  - service_key: '<PAGERDUTY_SERVICE_KEY>'
    description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'
    details:
        firing: '{{ .Alerts.Firing | len }}'
        resolved: '{{ .Alerts.Resolved | len }}'
        runbook_url: '{{ .CommonAnnotations.runbook_url }}'

- name: 'slack-warnings'
  slack_configs:
  - api_url: '<SLACK_WEBHOOK_URL>'
    channel: '#alerts-warning'
    title: '⚠️ {{ .GroupLabels.alertname }}'
    text: '{{ .CommonAnnotations.description }}'
```

---

## Partie 7 — Postmortem Blameless

### Culture blame-free

La culture **blame-free** (ou "blameless postmortem") est le fondement de toute organisation qui apprend de ses erreurs.

**Le principe** : les incidents sont causés par des **systèmes défaillants**, pas par des individus mauvais. Les humains travaillent avec les informations et outils disponibles à ce moment. Si un système permet à une mauvaise action d'arriver facilement, c'est le système qu'il faut corriger, pas la personne.

> [!warning] Pourquoi la culture de la peur tue l'apprentissage
> Dans une culture de blame, les équipes :
> - Cachent les incidents mineurs (pour éviter les reproches) → les problèmes s'accumulent
> - Évitent les prises de risque → innovation stoppée
> - Ne documentent pas les vrais problèmes → même incident se répète
>
> Google, Netflix, Amazon ont tous adopté le blameless postmortem comme pilier de leur culture d'ingénierie.

**Ce que le postmortem analyse :**
- Les processus défaillants
- Les outils insuffisants
- Les alertes manquantes ou trop nombreuses
- Les manques de documentation
- Les décisions prises avec les informations disponibles (et pourquoi ces informations manquaient)

**Ce que le postmortem n'analyse PAS :**
- "Qui a fait l'erreur" (blaming)
- "Comment punir le responsable"
- "Qui n'aurait pas dû toucher à ce système"

### Reconstruction de la timeline

La timeline d'incident est la colonne vertébrale du postmortem. Elle doit être aussi précise que possible, basée sur les logs et les témoignages.

```
Exemple de timeline :

14:15 UTC — Déploiement de la version 2.4.1 en production (commit abc1234)
14:23 UTC — Première alerte Prometheus : API error rate > 1% (seuil S3)
14:28 UTC — Taux d'erreur monte à 35% → promotion à S2, PagerDuty déclenche
14:31 UTC — Alice (@IC) prend l'incident, ouvre le war room
14:34 UTC — Bob identifie une corrélation avec le déploiement 14:15
14:38 UTC — Hypothèse : nouvelle migration BDD bloque les lectures
14:42 UTC — Vérification dans les logs BDD : locks effectivement détectés
14:48 UTC — Décision : rollback de la version 2.4.1 → `kubectl rollout undo`
14:52 UTC — Rollback terminé, taux d'erreur commence à baisser
15:05 UTC — Taux d'erreur < 0.1%, latence revenue à la normale
15:10 UTC — Incident déclaré résolu. Status page mise à jour.
15:15 UTC — Analyse post-incident démarrée
```

### Méthode des 5 Pourquoi

La méthode **5 Whys** (5 Pourquoi) est une technique d'analyse de cause racine développée chez Toyota. Elle consiste à demander "pourquoi ?" répétitivement jusqu'à atteindre la cause fondamentale.

```
Symptôme : L'API de paiement retourne des erreurs 500

Pourquoi 1 : Pourquoi des erreurs 500 ?
  → La BDD retourne des timeouts

Pourquoi 2 : Pourquoi la BDD timeout ?
  → Une requête de migration bloque la table principale pendant > 30s

Pourquoi 3 : Pourquoi la migration bloque-t-elle aussi longtemps ?
  → La migration ajoute un index sur une colonne sans CONCURRENTLY

Pourquoi 4 : Pourquoi sans CONCURRENTLY ?
  → Le développeur ne connaissait pas cette option PostgreSQL

Pourquoi 5 : Pourquoi le développeur ne la connaissait pas ?
  → Il n'y a pas de documentation des bonnes pratiques de migration BDD
    et le code review n'inclut pas de checklist migrations

CAUSE RACINE : Absence de documentation et de processus de review
               spécifique aux migrations BDD

ACTIONS :
  → Créer un guide des bonnes pratiques migrations PostgreSQL
  → Ajouter une checklist migrations au template de PR
  → Ajouter un lint automatique qui détecte les migrations sans CONCURRENTLY
```

> [!tip] Limite des 5 Whys
> La méthode des 5 Whys peut converger vers différentes causes racines selon les hypothèses initiales. Elle est excellente pour des incidents simples mais insuffisante pour des incidents complexes avec plusieurs causes contributives. Dans ce cas, compléter avec le Fishbone Diagram.

### Diagramme Fishbone (Ishikawa)

Le diagramme d'Ishikawa visualise les **causes multiples** d'un problème selon 6 catégories.

```
                    CAUSE 1                    CAUSE 2
               Processus                    Personnes
              /                              \
Migration     /  Pas de checklist            \ Pas de formation
sans review  /   review migrations            \ migrations BDD
            /                                  \
           ────────────────────────────────────►  INCIDENT
                                               API down
  Pas de lint    \                        /    
  pour CONCURRENTLY\  Outils          Technologies/ Lock agressif
                    \                  /         / PostgreSQL <14
              CAUSE 3                CAUSE 4
```

**Les 6M de l'Ishikawa (manufacturing) adaptés au logiciel :**

| Catégorie | Questions |
|-----------|-----------|
| **Méthode** | Les processus sont-ils définis ? La checklist est-elle respectée ? |
| **Machine** | L'infrastructure est-elle adaptée ? Les outils fonctionnent-ils correctement ? |
| **Main-d'œuvre** (Personnes) | La formation est-elle suffisante ? La charge de travail excessive ? |
| **Matière** | Les données d'entrée sont-elles valides ? Les dépendances fiables ? |
| **Milieu** (Environnement) | Différences prod/staging ? Configuration correcte ? |
| **Mesure** | Les métriques sont-elles appropriées ? Les alertes bien calibrées ? |

### Template de Postmortem

```markdown
# Postmortem : [Titre de l'incident]

**Incident ID** : INC-2024-042
**Date** : 2024-01-15
**Durée** : 1h44 (14:28 UTC → 16:12 UTC)
**Sévérité** : S2
**IC** : Alice Martin
**Rédigé par** : Bob Dupont
**Relu par** : Charlie Lambert, DBA Team

---

## Impact

**Utilisateurs impactés** : ~12 000 (35% de la base active)
**Fonctionnalité** : API de paiement — erreurs 500 sur 35% des transactions
**Revenue** : ~8 400 € de transactions bloquées pendant 1h44
**SLO breach** : Disponibilité tombée à 65% (SLO: 99.5%) — 5.7% d'error budget consommé

---

## Timeline

| Heure UTC | Événement |
|-----------|-----------|
| 14:15 | Déploiement version 2.4.1 en production |
| 14:23 | Première alerte : error rate > 1% |
| 14:28 | Taux d'erreur > 35% → S2 déclaré |
| 14:31 | IC en place, war room ouvert |
| 14:34 | Corrélation identifiée avec le déploiement |
| 14:42 | Lock BDD confirmé dans les logs |
| 14:48 | Décision de rollback |
| 14:52 | Rollback terminé |
| 15:05 | Métriques revenues à la normale |
| 15:10 | Incident résolu |

---

## Cause Racine

La migration BDD ajoutant un index sur `payments.user_id` a été exécutée
sans l'option `CONCURRENTLY` de PostgreSQL. Sans cette option, PostgreSQL pose
un lock exclusif sur la table pendant toute la durée de la création d'index
(~110 secondes sur 50M de lignes), bloquant toutes les requêtes en lecture et écriture.

---

## Facteurs Contributifs

1. **Absence de documentation** : aucun guide de bonnes pratiques pour les migrations BDD dans le wiki
2. **Pas de checklist PR migrations** : le template de PR standard ne couvre pas les migrations
3. **Staging non représentatif** : la table `payments` en staging a 10 000 lignes (vs 50M en prod), la migration s'exécute en 0.2s en staging et est donc indétectable
4. **Monitoring incomplet** : aucune alerte sur la durée des locks BDD
5. **Pas de smoke test post-déploiement** : le déploiement est déclaré "success" sans vérification fonctionnelle des paiements

---

## Ce qui a bien fonctionné

- L'alerte Prometheus a détecté le problème en 8 minutes (bon)
- La corrélation déploiement → incident a été identifiée rapidement
- La procédure de rollback Kubernetes a fonctionné sans accroc
- La communication status page a été mise à jour en temps quasi-réel

---

## Ce qui aurait pu mieux se passer

- L'alerte S3 initiale (14:23) n'a pas généré d'alerte PagerDuty → 5 minutes perdues
- Le monitoring des locks BDD était absent
- Le staging non représentatif n'a pas détecté le problème

---

## Action Items

| Action | Responsable | Priorité | Deadline |
|--------|-------------|----------|----------|
| Créer guide bonnes pratiques migrations PostgreSQL | Bob (DBA) | P1 | J+3 |
| Ajouter checklist migrations au template PR | Alice (tech lead) | P1 | J+5 |
| Implémenter lint automatique CONCURRENTLY | Charlie | P1 | J+7 |
| Configurer alerte sur durée des locks BDD > 5s | DBA Team | P2 | J+7 |
| Représenter la table payments en staging (10M lignes minimum) | Infra | P2 | J+14 |
| Ajouter smoke test paiement dans le pipeline CD | QA + DevOps | P2 | J+14 |
| Abaisser le seuil S3→S2 de 1% à 0.5% pour les paiements | Alice | P3 | J+21 |

---

## Lessons Learned

1. Les migrations BDD doivent systématiquement utiliser CONCURRENTLY sur les tables > 1M lignes
2. Le staging doit représenter la volumétrie de production pour les tables critiques
3. Chaque déploiement doit inclure un smoke test automatique des fonctionnalités critiques
4. Les locks BDD > 5 secondes doivent générer une alerte immédiate

---

*Document finalisé le 2024-01-20. Distribué à : équipe ingénierie, produit, direction technique.*
```

---

## Partie 8 — SBOM : Software Bill of Materials

### Pourquoi le SBOM est devenu critique

Les attaques sur la **supply chain logicielle** ont explosé depuis 2020 :

```
Exemples d'attaques supply chain majeures :

SolarWinds (2020) :
  → Logiciel Orion (monitoring réseau) compromis lors de la compilation
  → 18 000 organisations infectées dont le Pentagone, Microsoft, la Commission Européenne
  → Le code malveillant était présent dans les binaires signés officiellement

Log4Shell (2021, CVE-2021-44228) :
  → Vulnérabilité dans log4j (bibliothèque Java de logging ultra-répandue)
  → Pratiquement toute application Java du monde était vulnérable
  → Déni de service + exécution de code à distance
  → Des organisations ne savaient même pas qu'elles utilisaient log4j (dépendance transitive)

XZ Utils (2024, CVE-2024-3094) :
  → Backdoor insérée dans xz-utils par un contributeur malveillant
  → Ciblait OpenSSH sur les systèmes systemd
  → Détectée de justesse avant d'atteindre les distributions majeures
```

Le problème commun : les organisations **ne savent pas exactement ce que contient leur logiciel** — surtout les dépendances transitives.

Le SBOM résout ce problème en inventoriant exhaustivement tous les composants.

### Définition et contenu d'un SBOM

Un **SBOM** (Software Bill of Materials) est une liste exhaustive et formalisée de tous les composants qui constituent un logiciel :
- Bibliothèques tierces (directes et transitives)
- Frameworks
- Outils de build
- Dépendances OS
- Leur version exacte, leur licence, leur origine

**Analogie** : la liste des ingrédients d'un produit alimentaire. Obligatoire depuis la loi européenne (EU Cyber Resilience Act 2024) pour les logiciels vendus dans l'UE.

### Formats SBOM

**SPDX (Software Package Data Exchange)** — format Linux Foundation, recommandé par le NIST US :

```json
{
    "spdxVersion": "SPDX-2.3",
    "dataLicense": "CC0-1.0",
    "SPDXID": "SPDXRef-DOCUMENT",
    "name": "mon-application",
    "packages": [
        {
            "SPDXID": "SPDXRef-Package-express",
            "name": "express",
            "version": "4.18.2",
            "downloadLocation": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
            "licenseConcluded": "MIT",
            "licenseDeclared": "MIT",
            "supplier": "Organization: TJ Holowaychuk",
            "checksums": [
                {
                    "algorithm": "SHA256",
                    "checksumValue": "abc123..."
                }
            ]
        }
    ]
}
```

**CycloneDX** — format OWASP, très utilisé dans le monde de la sécurité :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bom xmlns="http://cyclonedx.org/schema/bom/1.4">
    <metadata>
        <component type="application">
            <name>mon-application</name>
            <version>2.4.1</version>
        </component>
    </metadata>
    <components>
        <component type="library">
            <name>express</name>
            <version>4.18.2</version>
            <purl>pkg:npm/express@4.18.2</purl>
            <licenses>
                <license><id>MIT</id></license>
            </licenses>
        </component>
    </components>
</bom>
```

### Génération automatique de SBOM

**Syft — génération multi-écosystème :**

```bash
# Installer syft
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin

# Générer un SBOM pour une image Docker
syft nginx:latest -o cyclonedx-json > sbom-nginx.json

# Générer pour un répertoire de projet Node.js
syft dir:. -o spdx-json > sbom-projet.json

# Générer pour un binaire
syft packages /usr/local/bin/kubectl -o table

# Avec scan des vulnérabilités intégré via Grype
grype sbom:./sbom-projet.json
# Sortie :
# NAME         VERSION   TYPE     VULNERABILITY   SEVERITY
# express      4.18.1    npm      CVE-2024-xxxx   HIGH
```

**Trivy — scan complet et SBOM :**

```bash
# Installer trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh

# Scan d'image + génération SBOM
trivy image --format cyclonedx --output sbom.json nginx:latest

# Scan de vulnérabilités depuis SBOM
trivy sbom sbom.json

# Scan d'un projet (détecte les fichiers lock : package-lock.json, go.sum, Cargo.lock)
trivy fs .
```

### Intégration SBOM en CI/CD

```yaml
# .github/workflows/sbom.yml — génération et scan SBOM dans GitHub Actions

name: SBOM Generation and Vulnerability Scan

on:
    push:
        branches: [main]
    pull_request:
        branches: [main]

jobs:
    sbom:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v4

        - name: Build Docker image
          run: docker build -t mon-app:${{ github.sha }} .

        - name: Generate SBOM with Syft
          uses: anchore/sbom-action@v0
          with:
              image: mon-app:${{ github.sha }}
              format: cyclonedx-json
              output-file: sbom.json

        - name: Upload SBOM as artifact
          uses: actions/upload-artifact@v4
          with:
              name: sbom
              path: sbom.json

        - name: Scan SBOM for vulnerabilities with Grype
          uses: anchore/scan-action@v3
          with:
              sbom: sbom.json
              fail-build: true      # bloque le build si vulnérabilité CRITICAL
              severity-cutoff: high # ignore LOW et MEDIUM

        - name: Upload vulnerability report
          uses: github/codeql-action/upload-sarif@v3
          if: always()
          with:
              sarif_file: results.sarif
```

---

## Partie 9 — Threat Modeling

### Définition et objectif

Le **Threat Modeling** (modélisation des menaces) est un processus structuré pour identifier et prioriser les menaces de sécurité **avant** que le code soit écrit.

**La question centrale** : "Qu'est-ce qui peut mal tourner ?"

**Pourquoi faire du Threat Modeling ?**
- Trouver les failles de conception (pas de code) = 10x moins cher à corriger
- Aligner les équipes dev, ops, sécurité sur les risques
- Prioriser les contrôles de sécurité (budget limité)
- Satisfaire les audits réglementaires (GDPR, PCI-DSS, ISO 27001)

### STRIDE — Modèle de classification des menaces

STRIDE est un acronyme Microsoft (1999) qui catégorise les types de menaces :

| Lettre | Menace | Définition | Exemples |
|--------|--------|------------|---------|
| **S** | Spoofing | Usurper l'identité d'une autre entité | Faux JWT, vol de session, IP spoofing |
| **T** | Tampering | Modifier des données sans autorisation | SQL injection, modification de fichiers, man-in-the-middle |
| **R** | Repudiation | Nier avoir effectué une action | Pas de logs, logs falsifiables, pas d'audit trail |
| **I** | Information Disclosure | Exposer des données confidentielles | Logs avec données sensibles, erreurs trop détaillées, IDOR |
| **D** | Denial of Service | Rendre le service indisponible | DDoS, requests malformées, ResourceExhaustion |
| **E** | Elevation of Privilege | Obtenir des permissions supérieures | Broken access control, SSTI, Path traversal |

### Processus de Threat Modeling en 4 étapes

**Étape 1 — Diagramme de flux de données (DFD)**

```
Exemple : API REST de gestion de commandes

[Utilisateur]  →  (HTTPS)  →  [API Gateway]  →  [Service Commandes]
                                                         │
                                              [BDD PostgreSQL]
                                              [Service Email]
                                              [Service Paiement]

Composants du DFD :
  □ Entités externes : Utilisateur, Service Paiement tiers
  ○ Processus (cercles) : API Gateway, Service Commandes, Service Email
  ── Flux de données : HTTPS request, gRPC, SMTP, BDD queries
  ═══ Frontières de confiance : Internet / DMZ / Réseau interne
```

**Étape 2 — Identifier les menaces (STRIDE par composant)**

```
Pour chaque composant, appliquer STRIDE :

API Gateway :
  S - Spoofing : Un attaquant peut-il usurper l'identité d'un utilisateur ?
      → JWT mal validé, absence de vérification de signature
  T - Tampering : Peut-on modifier les requêtes en transit ?
      → HTTP au lieu de HTTPS, absence de validation du corps
  R - Repudiation : Les actions sont-elles loguées avec contexte ?
      → Logs sans user_id, timestamp non-fiable
  I - Info Disclosure : Les erreurs révèlent-elles des infos internes ?
      → Stack traces en prod, champs BDD dans les erreurs 500
  D - DoS : L'API est-elle protégée contre les abus ?
      → Pas de rate limiting, pas de timeout sur les requêtes
  E - Elevation : Un utilisateur peut-il accéder aux données d'un autre ?
      → IDOR (Insecure Direct Object Reference), RBAC manquant
```

**Étape 3 — Évaluer et prioriser avec DREAD**

DREAD est un modèle de scoring pour prioriser les menaces identifiées :

| Dimension | Question | Score 1-3 |
|-----------|----------|-----------|
| **D**amage | Quel est l'impact si la menace est exploitée ? | 1=faible, 2=moyen, 3=sévère |
| **R**eproducibility | Est-ce facile à reproduire ? | 1=difficile, 2=moyen, 3=trivial |
| **E**xploitability | Quel niveau d'expertise pour exploiter ? | 1=expert, 2=intermédiaire, 3=script kiddie |
| **A**ffected users | Combien d'utilisateurs sont impactés ? | 1=un, 2=quelques-uns, 3=tous |
| **D**iscoverability | Est-ce facile à découvrir ? | 1=difficile, 2=moyen, 3=évident |

```
Exemple : IDOR sur l'API /orders/{id}
  D - Damage       : 3 (accès aux données de commandes de n'importe quel utilisateur)
  R - Reproducibility: 3 (il suffit de changer l'ID dans l'URL)
  E - Exploitability : 3 (aucune expertise requise)
  A - Affected users : 3 (tous les utilisateurs)
  D - Discoverability: 2 (visible en explorant l'API)
  
  Score DREAD = (3+3+3+3+2) / 5 = 14/15 = CRITIQUE → corriger immédiatement
```

**Étape 4 — Mitigation**

Pour chaque menace priorisée, définir une mitigation :

```
Menace IDOR : GET /orders/{id} retourne les commandes de n'importe quel utilisateur

Mitigation :
  Code actuel (vulnérable) :
    SELECT * FROM orders WHERE id = :id

  Code corrigé (vérifie la propriété) :
    SELECT * FROM orders WHERE id = :id AND user_id = :current_user_id

  Contrôles additionnels :
    - Test automatique : une requête avec le token de user_A sur la commande de user_B doit retourner 403
    - Monitoring : alerter sur les accès massifs à /orders/{id} par le même IP
```

### OWASP Threat Dragon

**OWASP Threat Dragon** est un outil open-source (web + desktop) pour créer des diagrammes de Threat Modeling visuellement.

```bash
# Installer Threat Dragon en local (Electron)
npm install -g @owasp/threat-dragon

# Ou utiliser l'interface web
# https://www.threatdragon.com/
```

**Workflow avec Threat Dragon :**
1. Créer un nouveau diagramme de modèle
2. Ajouter les composants (acteurs, processus, datastores, frontières)
3. Ajouter les flux de données entre composants
4. Cliquer sur chaque composant → Threat Dragon suggère automatiquement des menaces STRIDE
5. Pour chaque menace : évaluer, décider (mitigation/acceptée/out of scope)
6. Exporter le rapport en HTML ou JSON

### Exemple complet : Threat Modeling d'une API REST

```
Système : API REST de gestion d'utilisateurs (CRUD)
Endpoints : POST /users, GET /users/{id}, PUT /users/{id}, DELETE /users/{id}
Auth : JWT Bearer token

─── Analyse STRIDE complète ───────────────────────────────────────

ENDPOINT : POST /users/login (authentification)

[S] Spoofing — Credential stuffing / brute force
  Menace : Attaquant teste des millions de mots de passe
  Score DREAD : 11/15 (HIGH)
  Mitigation :
    → Rate limiting : max 5 tentatives / 15 min / IP
    → Account lockout temporaire après 10 échecs
    → CAPTCHA après 3 échecs consécutifs
    → MFA obligatoire pour les comptes admin

[T] Tampering — Injection dans le corps de la requête
  Menace : POST body contient du SQL malveillant dans email/password
  Score DREAD : 12/15 (CRITICAL)
  Mitigation :
    → ORM avec requêtes paramétrées (jamais de string concatenation)
    → Validation stricte des entrées (type, longueur, format)
    → Tester avec SQLMap en pre-prod

[I] Information Disclosure — Énumération d'utilisateurs
  Menace : Réponse différente selon que l'email existe ou non
    "Email invalide" vs "Mot de passe invalide" → on sait si l'email existe
  Score DREAD : 8/15 (MEDIUM)
  Mitigation :
    → Toujours retourner le même message : "Email ou mot de passe invalide"
    → Même temps de réponse (constant time comparison)

[D] Denial of Service — Flood de l'endpoint login
  Menace : 10 000 requêtes/sec saturent le service d'authentification
  Score DREAD : 9/15 (HIGH)
  Mitigation :
    → Rate limiting global par IP (nginx: limit_req_zone)
    → Timeout sur les opérations de hashing bcrypt (ex: max 2s)
    → CDN avec protection DDoS (Cloudflare, AWS WAF)

─── Actions prioritaires ───────────────────────────────────────────

CRITIQUE (à corriger avant mise en production) :
  1. Requêtes paramétrées sur toutes les opérations BDD
  2. Rate limiting sur /login

HIGH (à corriger dans le sprint suivant) :
  3. Messages d'erreur génériques (pas d'énumération d'emails)
  4. MFA pour les comptes admin
  5. Protection DDoS au niveau CDN

MEDIUM (backlog sécurité) :
  6. CAPTCHA après 3 échecs
  7. Audit logging de toutes les authentifications
```

---

## Exercices pratiques

### Exercice 1 — Postmortem sur un incident fictif (90 min)

**Scénario** : Un soir à 22h15, l'API de votre start-up renvoie des erreurs 503. Le service est down pendant 2h12. Cause découverte : un développeur a accidentellement supprimé la variable d'environnement `DATABASE_URL` lors d'un déploiement en "refactorisant" les variables d'environnement non utilisées. Il ne savait pas que cette variable était lue dynamiquement et non visible dans le code (injectée par le Kubernetes Secret).

**Travail à faire** :
1. Rédigez la timeline complète (inventez des horaires plausibles)
2. Appliquez les 5 Pourquoi pour identifier la cause racine
3. Rédigez le postmortem complet avec le template fourni dans ce cours
4. Définissez 5 action items concrets avec responsable et deadline
5. Identifiez ce qui a bien fonctionné et ce qui aurait pu être mieux

### Exercice 2 — Threat Modeling (60 min)

Modélisez les menaces de cette application simple : **Un site web permettant aux utilisateurs de téléverser des fichiers (CV, portfolio) et de les partager avec un lien public**.

1. Dessinez le DFD (entités, processus, datastores, frontières de confiance)
2. Appliquez STRIDE à chaque composant
3. Scorez les 5 menaces les plus graves avec DREAD
4. Proposez une mitigation pour chacune

### Exercice 3 — SBOM en pratique (45 min)

1. Créez un projet Node.js minimal avec Express, axios, lodash, jsonwebtoken
2. Installez Syft et générez un SBOM en format CycloneDX
3. Installez Grype et scannez le SBOM pour des vulnérabilités
4. Identifiez et documentez les vulnérabilités trouvées
5. Mettez à jour la dépendance vulnérable et re-vérifiez

### Exercice 4 — Runbook (45 min)

Rédigez un runbook pour l'incident suivant : **"La base de données MongoDB est pleine à 95%"**. Le runbook doit inclure :
- Diagnostic (comment vérifier l'utilisation du disque, quelles collections sont les plus grosses)
- Actions immédiates (nettoyage des logs, compression, archivage)
- Actions de mitigation (étendre le disque, sharding)
- Contacts d'escalade
- Critères de résolution

---

> [!info] Pour aller plus loin
> - [[02 - Metriques et Monitoring]] — métriques SLI et alerting
> - [[03 - Tracing et Debugging Distribue]] — investigation technique pendant les incidents
> - [[07 - DevSecOps]] — intégrer la sécurité dans le cycle de déploiement
> - Livre : *Site Reliability Engineering* — Google (disponible gratuitement en ligne : sre.google/books)
> - Livre : *The DevOps Handbook* — Gene Kim, Patrick Debois, John Willis
> - OWASP Threat Modeling : owasp.org/www-community/Threat_Modeling
> - Formation : Google SRE Workbook (sre.google/workbook)
