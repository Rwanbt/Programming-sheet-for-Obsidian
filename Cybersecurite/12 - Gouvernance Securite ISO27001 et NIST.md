# Gouvernance Sécurité — ISO 27001 et NIST CSF

La gouvernance de la sécurité de l'information est le socle sur lequel repose toute organisation qui prend au sérieux la protection de ses actifs numériques. Ce cours couvre les deux référentiels dominants dans l'industrie — l'ISO 27001 et le NIST Cybersecurity Framework — ainsi que les structures opérationnelles (RSSI, SOC) et les matrices d'attaque qui permettent de passer de la théorie à la pratique. À l'issue de ce cours, vous serez capable de comprendre une politique de sécurité, de conduire une analyse de risques de base, et de situer un incident dans un cadre de réponse structuré.

---

## 1. ISO 27001 — Le Système de Management de la Sécurité de l'Information

### 1.1 Présentation et objectifs

L'ISO/IEC 27001 est une norme internationale publiée par l'Organisation Internationale de Normalisation (ISO) et la Commission Électrotechnique Internationale (IEC). Sa première version date de 2005 ; la version actuelle est **ISO/IEC 27001:2022**.

Elle définit les exigences pour établir, mettre en œuvre, maintenir et améliorer en continu un **Système de Management de la Sécurité de l'Information** (SMSI). Un SMSI n'est pas un outil logiciel ni une checklist : c'est un **cadre organisationnel** qui englobe les personnes, les processus et la technologie.

> [!info] Définition — SMSI
> Un Système de Management de la Sécurité de l'Information (SMSI) est l'ensemble des politiques, procédures, directives, ressources et activités qu'une organisation gère collectivement dans le but de protéger ses informations. Il repose sur une approche basée sur les **risques** : on identifie ce qui peut mal tourner, on évalue la probabilité et l'impact, puis on décide comment y répondre.

**Pourquoi la certification ISO 27001 ?**

| Bénéfice | Pour qui |
|---|---|
| Réduire les incidents de sécurité | Équipe technique et direction |
| Gagner la confiance des clients | Commercial et partenariats |
| Répondre aux exigences contractuelles | Juridique et conformité |
| Accéder à certains appels d'offres publics | Direction générale |
| Structurer la gestion des risques | RSSI et DSI |

La norme est **certifiable** : un organisme accrédité audite l'organisation et délivre un certificat valable 3 ans (avec audits de surveillance annuels).

### 1.2 Structure de la norme ISO 27001:2022

La norme est organisée en **clauses** (sections obligatoires) et une **Annexe A** (contrôles de référence).

```
ISO/IEC 27001:2022
│
├── Clauses 1–3 : Domaine d'application, références, termes
├── Clause 4  : Contexte de l'organisation
├── Clause 5  : Leadership
├── Clause 6  : Planification (dont analyse de risques)
├── Clause 7  : Support
├── Clause 8  : Réalisation (mise en œuvre)
├── Clause 9  : Évaluation des performances
├── Clause 10 : Amélioration
│
└── Annexe A  : 93 contrôles de sécurité (référence, non exhaustif)
```

> [!warning] Clauses obligatoires vs Annexe A
> Les clauses 4 à 10 sont **toutes obligatoires** pour la certification. L'Annexe A liste des contrôles possibles, mais l'organisation doit justifier dans une **Déclaration d'Applicabilité (SoA — Statement of Applicability)** pourquoi elle applique ou exclut chaque contrôle.

#### L'Annexe A — 93 contrôles répartis en 4 domaines

La version 2022 a restructuré les contrôles (anciennement 114 en 2013) en 4 grands domaines :

| Domaine | Nombre de contrôles | Exemples |
|---|---|---|
| **Organisationnel** (5.x) | 37 | Politiques, gestion des risques, relations fournisseurs |
| **Humain** (6.x) | 8 | Sensibilisation, formation, sanctions |
| **Physique** (7.x) | 14 | Contrôle d'accès locaux, protection équipements |
| **Technologique** (8.x) | 34 | Cryptographie, journalisation, gestion des vulnérabilités |

**Exemples de contrôles clés :**

```
Contrôle 5.15 — Contrôle d'accès
  → L'accès aux informations doit être restreint selon la politique.

Contrôle 6.3 — Sensibilisation, éducation et formation à la sécurité
  → Tout le personnel doit recevoir une formation adaptée.

Contrôle 7.1 — Périmètres de sécurité physique
  → Des périmètres doivent protéger les zones abritant des informations.

Contrôle 8.8 — Gestion des vulnérabilités techniques
  → Les vulnérabilités doivent être identifiées et corrigées en temps opportun.

Contrôle 8.24 — Utilisation de la cryptographie
  → Des règles doivent encadrer l'usage de la cryptographie.
```

> [!tip] Nouveautés 2022
> L'ISO 27001:2022 ajoute 11 contrôles absents de la version 2013, dont :
> - **5.7** — Threat intelligence
> - **5.23** — Sécurité de l'information pour l'utilisation des services cloud
> - **8.9** — Gestion de la configuration
> - **8.10** — Suppression des informations
> - **8.11** — Masquage des données
> - **8.12** — Prévention des fuites de données (DLP)

---

## 2. Le Cycle PDCA appliqué à la sécurité

### 2.1 Présentation du cycle PDCA

Le PDCA (Plan-Do-Check-Act), aussi appelé **roue de Deming**, est le moteur d'amélioration continue sur lequel repose l'ISO 27001. Il s'applique à l'ensemble du SMSI.

```
          ┌─────────────────────────────────────────┐
          │                                         │
     ┌────▼────┐                          ┌─────────▼───┐
     │  PLAN   │                          │     ACT     │
     │         │                          │             │
     │ Définir │◄─────── Améliorer ───────│  Corriger   │
     │ le SMSI │                          │  Améliorer  │
     └────┬────┘                          └─────────────┘
          │                                         ▲
          │ Établir                       Auditer   │
          ▼                                         │
     ┌────────────┐                       ┌─────────┴───┐
     │     DO     │                       │    CHECK    │
     │            │                       │             │
     │  Mettre en ├──────── Mesurer ──────►  Surveiller │
     │  œuvre     │                       │  Évaluer    │
     └────────────┘                       └─────────────┘
```

### 2.2 PLAN — Planifier le SMSI

**Objectif :** Définir le périmètre, identifier les risques, choisir les contrôles.

Activités clés :
- Définir le **périmètre du SMSI** (tous les systèmes ? un département ? un produit ?)
- Analyser le **contexte** (interne : culture, ressources ; externe : réglementation, concurrents)
- Identifier les **parties intéressées** (clients, régulateurs, actionnaires)
- Réaliser l'**analyse de risques**
- Rédiger le **Plan de Traitement des Risques (PTR)**
- Produire la **Déclaration d'Applicabilité (SoA)**

### 2.3 DO — Mettre en œuvre

**Objectif :** Déployer ce qui a été planifié.

Activités clés :
- Implémenter les contrôles de l'Annexe A retenus
- Former et sensibiliser le personnel
- Déployer les outils techniques (SIEM, DLP, IAM, etc.)
- Gérer les incidents au quotidien
- Documenter les procédures opérationnelles

### 2.4 CHECK — Vérifier

**Objectif :** Mesurer l'efficacité du SMSI.

Activités clés :
- Réaliser des **audits internes** (au moins une fois par an)
- Suivre les **indicateurs de performance** (KPI sécurité)
- Effectuer des **tests d'intrusion** et revues de vulnérabilités
- Organiser des **revues de direction**

```python
# Exemple d'indicateurs de performance sécurité (KPI)
kpi_securite = {
    "delai_moyen_correction_critique": "< 72h",
    "taux_completion_formation": "> 95%",
    "nombre_incidents_par_mois": "< 5",
    "disponibilite_systemes_critiques": "> 99.9%",
    "taux_couverture_sauvegardes": "100%",
    "delai_detection_intrusion": "< 30 min",
}
```

### 2.5 ACT — Agir / Améliorer

**Objectif :** Corriger les non-conformités, améliorer en continu.

Activités clés :
- Traiter les **non-conformités** détectées lors des audits
- Lancer des **actions correctives** (cause racine + correction)
- Mettre à jour les politiques et procédures
- Redémarrer le cycle PDCA

---

## 3. Analyse de Risques — Méthodologie complète

### 3.1 Vocabulaire fondamental

Avant de conduire une analyse de risques, il faut maîtriser le vocabulaire :

| Terme | Définition | Exemple |
|---|---|---|
| **Actif** | Tout ce qui a de la valeur pour l'organisation | Base de données clients, code source, réputat. |
| **Menace** | Cause potentielle d'un incident indésirable | Ransomware, employé malveillant, incendie |
| **Vulnérabilité** | Faiblesse exploitable par une menace | Patch manquant, mot de passe faible |
| **Impact** | Conséquence si la menace se réalise | Perte de données, arrêt d'activité, amende |
| **Vraisemblance** | Probabilité que la menace se réalise | Faible / Moyen / Élevé |
| **Risque** | Combinaison impact × vraisemblance | Score quantitatif ou qualitatif |
| **Contrôle** | Mesure qui réduit le risque | Antivirus, formation, chiffrement |
| **Risque résiduel** | Risque restant après application des contrôles | Ce qui subsiste après mitigation |

### 3.2 Inventaire des actifs

La première étape est de **recenser tous les actifs informationnels**. On les classe en catégories :

```
ACTIFS INFORMATIONNELS
│
├── Actifs primaires (ce qu'on protège)
│   ├── Informations : données clients, contrats, R&D
│   └── Processus : facturation, production, support
│
└── Actifs de support (ce qui fait tourner les actifs primaires)
    ├── Matériel : serveurs, postes, routeurs
    ├── Logiciels : OS, applications, bases de données
    ├── Réseau : LAN, WAN, liens internet
    ├── Personnel : administrateurs, développeurs
    └── Sites : datacenters, bureaux
```

**Exemple de tableau d'inventaire d'actifs :**

| ID | Actif | Catégorie | Propriétaire | Confidentialité | Intégrité | Disponibilité |
|---|---|---|---|---|---|---|
| A01 | Base clients PostgreSQL | Donnée | DPO | Élevée | Élevée | Élevée |
| A02 | Serveur web NGINX | Matériel | SysAdmin | Moyenne | Élevée | Élevée |
| A03 | Code source Git | Logiciel | CTO | Élevée | Élevée | Moyenne |
| A04 | Documentation interne | Information | RH | Faible | Moyenne | Faible |

> [!info] Critères CIA
> Chaque actif est évalué selon la **triade CIA** :
> - **Confidentialité** : seules les personnes autorisées y accèdent
> - **Intégrité** : les données n'ont pas été altérées sans autorisation
> - **Disponibilité** : l'actif est accessible quand on en a besoin

### 3.3 Identification des menaces et vulnérabilités

Pour chaque actif, on identifie :

**Exemples de menaces par catégorie :**

```
MENACES TECHNIQUES
├── Malwares (ransomware, spyware, rootkit)
├── Attaques réseau (DDoS, MITM, sniffing)
├── Exploits (CVE, 0-day, injection SQL)
└── Accès non autorisé (brute force, credential stuffing)

MENACES HUMAINES
├── Erreur involontaire (mauvaise config, suppression accidentelle)
├── Malveillance interne (sabotage, exfiltration)
├── Ingénierie sociale (phishing, vishing, pretexting)
└── Espionnage industriel

MENACES PHYSIQUES / ENVIRONNEMENTALES
├── Incendie, inondation, tremblement de terre
├── Panne électrique
├── Vol d'équipement
└── Panne matérielle (disque dur, PSU)
```

**Exemples de vulnérabilités associées :**

| Menace | Vulnérabilité associée |
|---|---|
| Ransomware | Patchs OS non appliqués, sauvegardes absentes |
| Phishing | Personnel non formé, absence de MFA |
| DDoS | Absence de CDN/WAF, plan de capacité insuffisant |
| Vol de données | Chiffrement absent sur disques, DLP non configuré |
| Accès non autorisé | Mots de passe faibles, pas de revue des accès |

### 3.4 Calcul du niveau de risque

La méthode la plus répandue (compatible ISO 27005) utilise une **matrice risque** :

```
Vraisemblance
    │  4 │  M  │  H  │  H  │  C  │
    │  3 │  L  │  M  │  H  │  H  │
    │  2 │  L  │  L  │  M  │  H  │
    │  1 │  N  │  L  │  L  │  M  │
    └────┴──────┴──────┴──────┴──────
              1     2     3     4    Impact
```

**Légende :**
- `N` = Négligeable (risque acceptable sans action)
- `L` = Faible (surveiller)
- `M` = Moyen (traitement planifié)
- `H` = Élevé (traitement prioritaire)
- `C` = Critique (action immédiate)

**Formule simplifiée :**

```python
def calculer_risque(vraisemblance: int, impact: int) -> str:
    """
    Calcule le niveau de risque.
    
    Args:
        vraisemblance: 1 (rare) à 4 (quasi-certain)
        impact: 1 (négligeable) à 4 (catastrophique)
    
    Returns:
        Niveau de risque : "Négligeable", "Faible", "Moyen", "Élevé", "Critique"
    """
    score = vraisemblance * impact
    
    if score <= 2:
        return "Négligeable"
    elif score <= 4:
        return "Faible"
    elif score <= 8:
        return "Moyen"
    elif score <= 12:
        return "Élevé"
    else:
        return "Critique"

# Exemples
print(calculer_risque(3, 4))  # Ransomware sur DB critique → "Critique"
print(calculer_risque(1, 3))  # Incendie datacenter → "Moyen"
print(calculer_risque(4, 1))  # Spam quotidien → "Faible"
```

### 3.5 Exemple complet d'analyse de risques

```
Actif   : Base de données clients (A01)
Menace  : Ransomware via email de phishing
Vulnér. : Pas de filtre email avancé, 30% du personnel non formé

Vraisemblance : 3 (probable — secteur ciblé régulièrement)
Impact        : 4 (catastrophique — RGPD + arrêt activité + réputation)
Score brut    : 12 → Niveau "Élevé"

Contrôles existants :
  - Antivirus à jour (réduit vraisemblance de 1)
  - Sauvegardes quotidiennes (réduit impact de 1)

Score résiduel : Vraisemblance=2, Impact=3 → Score=6 → "Moyen"

Traitement décidé : RÉDUIRE
  Actions :
    1. Déployer un filtre anti-phishing avancé (deadline: J+30)
    2. Former 100% du personnel (deadline: J+60)
    3. Implémenter MFA sur tous les comptes (deadline: J+15)
```

---

## 4. Plan de Traitement des Risques

### 4.1 Les quatre options de traitement

Face à un risque identifié, l'organisation dispose de **quatre options** :

```
                    ┌─────────────────────────────────────────────┐
                    │         PLAN DE TRAITEMENT DES RISQUES      │
                    └──────────────────┬──────────────────────────┘
                                       │
           ┌───────────────────────────┼───────────────────────────┐
           │                           │                           │
    ┌──────▼──────┐           ┌────────▼──────┐           ┌────────▼──────┐
    │   RÉDUIRE   │           │  TRANSFÉRER   │           │    ACCEPTER   │
    │  (Mitiger)  │           │  (Partager)   │           │               │
    └──────┬──────┘           └────────┬──────┘           └────────┬──────┘
           │                           │                           │
   Mettre en place            Assurance cyber,          Risque résiduel
   des contrôles              sous-traitance            jugé acceptable
                              responsabilité
                                       │
                              ┌────────▼──────┐
                              │    ÉVITER     │
                              │               │
                              │  Supprimer    │
                              │  l'activité   │
                              └───────────────┘
```

**Détail de chaque option :**

**RÉDUIRE** — Mettre en place des contrôles pour diminuer la vraisemblance et/ou l'impact.

```bash
# Exemple concret : réduire le risque d'accès non autorisé
# Action 1 : Activation du MFA sur tous les comptes admin
aws iam update-account-password-policy \
  --require-symbols \
  --require-numbers \
  --minimum-password-length 14

# Action 2 : Revue des droits d'accès
aws iam generate-credential-report
aws iam get-credential-report
```

**TRANSFÉRER** — Confier le risque à un tiers (assurance, sous-traitant spécialisé).

Exemples :
- Souscrire une **assurance cyber** (couvre les pertes financières post-incident)
- Externaliser l'hébergement vers un **cloud provider certifié ISO 27001**
- Faire appel à un **SOC externalisé** pour la surveillance

**ACCEPTER** — Décider de ne rien faire si le risque résiduel est jugé acceptable.

> [!warning] Acceptation de risque
> L'acceptation doit être **formellement documentée** et **validée par la direction**. Ce n'est pas "ignorer" le risque : c'est une décision consciente et tracée. Tout risque accepté doit être réévalué à chaque cycle PDCA.

**ÉVITER** — Supprimer l'activité qui génère le risque.

Exemple : une société décide de **ne pas stocker de données de carte bancaire** et d'utiliser uniquement un prestataire de paiement (Stripe, Adyen) — elle évite ainsi tout risque PCI-DSS lié au stockage des cartes.

### 4.2 Formalisation du Plan de Traitement

```python
# Modèle de Plan de Traitement des Risques (PTR)
plan_traitement = {
    "R001": {
        "risque": "Ransomware via phishing",
        "actif": "Base de données clients",
        "score_brut": {"vraisemblance": 3, "impact": 4, "niveau": "Élevé"},
        "traitement": "RÉDUIRE",
        "actions": [
            {
                "id": "A001",
                "description": "Déployer filtre anti-phishing (Proofpoint)",
                "responsable": "SysAdmin",
                "deadline": "2024-03-01",
                "statut": "En cours",
                "cout": 3500
            },
            {
                "id": "A002",
                "description": "Formation phishing pour 100% des collaborateurs",
                "responsable": "RSSI",
                "deadline": "2024-04-01",
                "statut": "Planifié",
                "cout": 1200
            }
        ],
        "score_residuel": {"vraisemblance": 1, "impact": 3, "niveau": "Moyen"},
        "validation_direction": "2024-02-15",
        "proprietaire_risque": "DSI"
    }
}
```

---

## 5. Processus de Certification ISO 27001

### 5.1 Les étapes de la certification

```
Phase 0 : Préparation (3-12 mois selon maturité)
│
├── Définir le périmètre du SMSI
├── Nommer un RSSI / chef de projet
├── Conduire l'analyse de risques
├── Rédiger les politiques et procédures
├── Implémenter les contrôles
└── Former le personnel

Phase 1 : Audit de certification — Revue documentaire
│
├── L'auditeur examine la documentation du SMSI
├── Vérifie l'existence des politiques obligatoires
├── Identifie les écarts majeurs à corriger
└── Durée : 1-3 jours (selon périmètre)

Phase 2 : Audit de certification — Audit terrain
│
├── L'auditeur visite les locaux, interroge le personnel
├── Teste l'efficacité réelle des contrôles
├── Rédige son rapport avec constatations
└── Durée : 2-5 jours

Décision de certification
│
├── Conformité → Certificat délivré (valable 3 ans)
├── Non-conformités mineures → Correction sous 90 jours
└── Non-conformités majeures → Nouveau cycle d'audit requis

Audits de surveillance (années 2 et 3)
│
└── Vérification annuelle du maintien du SMSI

Renouvellement (tous les 3 ans)
└── Nouvel audit complet de recertification
```

### 5.2 Documents obligatoires pour la certification

> [!info] Documents exigés par la norme
> Ces documents doivent exister ET être maintenus à jour pour obtenir la certification :

| Document | Clause ISO | Contenu |
|---|---|---|
| Politique de sécurité de l'information | 5.2 | Engagement de la direction, objectifs |
| Périmètre du SMSI | 4.3 | Frontières du SMSI documentées |
| Analyse de risques | 6.1.2 | Méthodologie, actifs, risques identifiés |
| Plan de Traitement des Risques | 6.1.3 | Actions, responsables, délais |
| Déclaration d'Applicabilité (SoA) | 6.1.3 | Justification des 93 contrôles |
| Objectifs de sécurité | 6.2 | KPI mesurables et tracés |
| Rapports d'audits internes | 9.2 | Résultats des audits périodiques |
| Revue de direction | 9.3 | Décisions de la direction sur le SMSI |
| Non-conformités et actions correctives | 10.1 | Traçabilité des problèmes et corrections |

### 5.3 Exemples de politiques de sécurité

#### Politique de classification de l'information

```markdown
# Politique de Classification de l'Information
Version : 2.1 | Date : 2024-01-15 | Propriétaire : RSSI

## 1. Niveaux de classification

| Niveau | Label | Description | Exemples |
|--------|-------|-------------|---------|
| C0 | PUBLIC | Destiné au public | Site web, plaquettes |
| C1 | INTERNE | Usage interne uniquement | Procédures, organigramme |
| C2 | CONFIDENTIEL | Accès restreint aux ayants droit | Données clients, contrats |
| C3 | SECRET | Accès exceptionnel | R&D, données stratégiques |

## 2. Obligations par niveau

### C2 — CONFIDENTIEL
- Chiffrement obligatoire en transit (TLS 1.2+) et au repos (AES-256)
- Partage externe : signature NDA préalable requise
- Stockage : uniquement sur systèmes approuvés par la DSI
- Destruction : écrasement sécurisé 7 passes (DoD 5220.22-M)
```

#### Politique BYOD (Bring Your Own Device)

```markdown
# Politique BYOD — Appareils Personnels en Entreprise
Version : 1.3 | Propriétaire : RSSI / DRH

## Périmètre
Tous les collaborateurs souhaitant utiliser un appareil personnel
(smartphone, tablette, laptop) pour accéder aux ressources de l'entreprise.

## Conditions d'utilisation

### Prérequis techniques (contrôlés par MDM)
- [ ] OS à jour (max 2 versions majeures de retard toléré)
- [ ] Chiffrement du stockage activé
- [ ] Code PIN / biométrie activé (min. 6 chiffres)
- [ ] Solution antivirus approuvée installée
- [ ] Jailbreak / root : INTERDIT

### Accès autorisés
- Email professionnel (via conteneur chiffré)
- Applications métier approuvées (liste blanche MDM)
- VPN obligatoire pour tout accès au réseau interne

### Séparation vie pro / vie perso
- Conteneur MDM isolé : données pro dans un espace chiffré séparé
- L'IT ne peut accéder qu'au conteneur pro, jamais aux données perso
- En cas de perte/vol : effacement à distance du conteneur pro uniquement
```

#### Politique de contrôle d'accès

```bash
# Exemple d'implémentation technique : principe du moindre privilège
# Script de revue des accès (Linux)

#!/bin/bash
# Audit des comptes avec accès sudo
echo "=== Comptes avec privilèges sudo ==="
getent group sudo | cut -d: -f4 | tr ',' '\n' | while read user; do
    last_login=$(lastlog -u "$user" | tail -1 | awk '{print $4, $5, $6, $7, $8, $9}')
    echo "Utilisateur: $user | Dernière connexion: $last_login"
done

# Comptes sans connexion depuis 90 jours → désactiver
echo ""
echo "=== Comptes inactifs depuis 90 jours ==="
lastlog | awk -v limit=$(date -d '90 days ago' +%s) '
NR>1 && $5 != "**Never" {
    cmd = "date -d \"" $4" "$5" "$6" "$7" "$8"\" +%s"
    cmd | getline ts
    close(cmd)
    if (ts < limit) print $1, "inactif depuis", $4, $5, $6
}'
```

---

## 6. NIST Cybersecurity Framework (CSF)

### 6.1 Présentation et contexte

Le **NIST Cybersecurity Framework** est publié par le National Institute of Standards and Technology (NIST), agence du gouvernement américain. La version actuelle est **NIST CSF 2.0** (publiée en février 2024).

Contrairement à l'ISO 27001 qui est une norme certifiable, le NIST CSF est un **cadre volontaire** conçu pour :
- Améliorer la communication entre les équipes techniques et la direction
- Structurer la posture de cybersécurité d'une organisation
- S'appliquer à toute organisation quelle que soit sa taille ou son secteur

> [!info] ISO 27001 vs NIST CSF — Différence fondamentale
> - **ISO 27001** : norme prescriptive avec exigences obligatoires, certifiable par un tiers
> - **NIST CSF** : cadre descriptif, volontaire, non certifiable — outil de communication et de roadmap

### 6.2 Les 6 fonctions du NIST CSF 2.0

La version 2.0 ajoute une 6e fonction (**GOVERN**) aux 5 fonctions historiques. Ces fonctions représentent le **cycle de vie complet** de la gestion des risques cyber.

```
                    ┌─────────────────────────────────────────┐
                    │           GOVERN (Transversal)          │
                    │  Stratégie, politique, supervision,     │
                    │  gestion des risques, conformité        │
                    └──────────────────┬──────────────────────┘
                                       │ Chapeaute tout
          ┌────────────────────────────┼────────────────────────────┐
          │                            │                            │
   ┌──────▼──────┐              ┌──────▼──────┐              ┌──────▼──────┐
   │  IDENTIFY   │              │   PROTECT   │              │   DETECT    │
   │             │              │             │              │             │
   │ Inventaire  │              │ Contrôles   │              │ Surveillance│
   │ Risques     │              │ Protections │              │ Anomalies   │
   └─────────────┘              └─────────────┘              └─────────────┘
          │                            │                            │
          └────────────────────────────┼────────────────────────────┘
                                       │
          ┌────────────────────────────┼────────────────────────────┐
          │                                                         │
   ┌──────▼──────┐                                         ┌────────▼────┐
   │   RESPOND   │                                         │   RECOVER   │
   │             │                                         │             │
   │ Réponse à   │                                         │ Restauration│
   │ l'incident  │                                         │ Continuité  │
   └─────────────┘                                         └─────────────┘
```

#### Fonction 1 : GOVERN (GV) — Gouvernance

**Rôle :** Établir et surveiller la stratégie de cybersécurité, les politiques et la gestion des risques au plus haut niveau de l'organisation.

Catégories principales :
- **GV.OC** — Contexte organisationnel : comprendre la mission, les parties prenantes, les dépendances
- **GV.RM** — Stratégie de gestion des risques : appétit au risque défini, tolérance documentée
- **GV.RR** — Rôles et responsabilités : RACI de sécurité clair
- **GV.PO** — Politiques : politiques de sécurité alignées sur la stratégie
- **GV.OV** — Supervision : la direction surveille les résultats de la cybersécurité
- **GV.SC** — Supply Chain : gestion des risques chez les fournisseurs

#### Fonction 2 : IDENTIFY (ID) — Identifier

**Rôle :** Comprendre le contexte de l'organisation pour gérer le risque cyber sur les systèmes, personnes, actifs, données et capacités.

```python
# Exemple : catégories IDENTIFY implémentées
identify_checklist = {
    "ID.AM": {  # Asset Management
        "description": "Inventaire des actifs physiques et logiciels",
        "exemples": [
            "CMDB (Configuration Management Database) à jour",
            "Scan réseau automatisé hebdomadaire (Nmap, Nessus)",
            "Inventaire des données personnelles (RGPD)",
        ],
        "statut": "implémenté"
    },
    "ID.RA": {  # Risk Assessment
        "description": "Analyse de risques documentée et régulière",
        "exemples": [
            "Analyse de risques annuelle sur les actifs critiques",
            "Veille vulnérabilités (CVE, NVD)",
            "Tests d'intrusion annuels",
        ],
        "statut": "en cours"
    },
    "ID.IM": {  # Improvement
        "description": "Amélioration continue basée sur les leçons apprises",
        "exemples": [
            "Post-mortem systématique après chaque incident",
            "Benchmarking avec standards du secteur",
        ],
        "statut": "planifié"
    }
}
```

#### Fonction 3 : PROTECT (PR) — Protéger

**Rôle :** Développer et mettre en œuvre les mesures de protection appropriées pour limiter l'impact d'un incident.

Catégories principales avec exemples techniques :

```bash
# PR.AA — Authentication and Access Control
# Implémenter MFA pour tous les accès distants
# Exemple AWS CLI
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name "AdminMFA" \
  --outfile /tmp/qrcode.png \
  --bootstrap-method QRCodePNG

# PR.DS — Data Security
# Chiffrer les données sensibles en transit
# Configuration TLS stricte dans Nginx
cat << 'EOF' >> /etc/nginx/conf.d/ssl.conf
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
add_header Strict-Transport-Security "max-age=63072000" always;
EOF

# PR.PS — Platform Security
# Hardening OS — désactiver les services inutiles (CIS Benchmark)
systemctl disable avahi-daemon cups bluetooth
systemctl mask avahi-daemon cups bluetooth
```

#### Fonction 4 : DETECT (DE) — Détecter

**Rôle :** Mettre en place les activités appropriées pour identifier l'occurrence d'un événement de cybersécurité.

```python
# Exemple de règle de détection SIEM (format Sigma)
sigma_rule = """
title: Tentative de force brute SSH
id: 5f1f2c3e-1234-5678-abcd-ef0123456789
status: stable
description: Détecte les tentatives de force brute sur SSH
references:
  - https://attack.mitre.org/techniques/T1110/001/
author: SOC L2
date: 2024/01/15
logsource:
  product: linux
  service: auth
detection:
  selection:
    CommandLine|contains: 'Failed password'
  timeframe: 5m
  condition: selection | count() > 10
falsepositives:
  - Scripts de test légitimes
  - Oubli de mot de passe d'un administrateur
level: medium
tags:
  - attack.credential_access
  - attack.t1110.001
"""
```

#### Fonction 5 : RESPOND (RS) — Répondre

**Rôle :** Développer et mettre en œuvre les activités appropriées pour agir face à un incident détecté.

```markdown
# Playbook de réponse — Ransomware (simplifié)

## Phase 1 : Confinement (0-15 min)
- [ ] Isoler le(s) système(s) affecté(s) du réseau (VLAN quarantaine)
- [ ] Couper les partages réseau depuis les machines infectées
- [ ] Préserver les preuves (snapshot mémoire, logs)
- [ ] Alerter RSSI + DSI + Direction

## Phase 2 : Éradication (15 min - 4h)
- [ ] Identifier le vecteur d'entrée (logs, EDR)
- [ ] Identifier le ransomware (ID Ransomware, VirusTotal)
- [ ] Scanner l'ensemble du réseau pour détecter la propagation
- [ ] Révoquer les credentials compromis

## Phase 3 : Restauration (4h - 72h)
- [ ] Vérifier l'intégrité des sauvegardes (hors ligne !)
- [ ] Restaurer depuis la dernière sauvegarde propre
- [ ] Patcher la vulnérabilité exploitée avant reconnexion
- [ ] Surveillance renforcée 72h post-restauration

## Phase 4 : Post-mortem (J+7)
- [ ] Rapport d'incident complet
- [ ] Timeline détaillée de l'attaque
- [ ] Leçons apprises + plan d'action
- [ ] Notification CNIL si données personnelles compromises (72h légal)
```

#### Fonction 6 : RECOVER (RC) — Récupérer

**Rôle :** Développer et mettre en œuvre les activités appropriées pour maintenir les plans de résilience et restaurer les capacités ou services impactés.

**Documents clés :**
- **PCA** (Plan de Continuité d'Activité) : maintenir les activités critiques pendant une crise
- **PRA** (Plan de Reprise d'Activité) : restaurer les systèmes après une interruption

```python
# Objectifs de récupération
rto_rpo = {
    "systeme": "Base de données clients",
    "criticite": "P1 - Critique",
    "RTO": "4h",      # Recovery Time Objective : délai max pour remettre en service
    "RPO": "1h",      # Recovery Point Objective : perte de données max tolérée
    "strategie": [
        "Réplication synchrone sur datacenter secondaire",
        "Sauvegardes incrémentielles toutes les heures",
        "Sauvegarde complète quotidienne hors site",
        "Test de restauration mensuel obligatoire"
    ]
}
```

### 6.3 Tiers de maturité (Implementation Tiers)

Le NIST CSF définit **4 tiers** qui décrivent la sophistication des pratiques de cybersécurité d'une organisation. Ce ne sont pas des niveaux à atteindre obligatoirement — c'est un outil de positionnement.

| Tier | Nom | Description |
|---|---|---|
| **Tier 1** | Partiel | Processus ad hoc, pas de gestion de risques formelle, peu de conscience cyber |
| **Tier 2** | Risque informé | Pratiques de sécurité approuvées mais pas systématiques, conscient des risques |
| **Tier 3** | Répétable | Politiques formelles, processus réguliers, adaptation proactive aux menaces |
| **Tier 4** | Adaptatif | Amélioration continue, partage d'information avec l'écosystème, très mature |

```
Tier 1 — Partiel
  "On fait du mieux qu'on peut quand un incident arrive"
  → Aucune politique formelle, réaction uniquement curative

Tier 2 — Risque informé
  "On sait qu'il faut faire de la sécurité et on a quelques règles"
  → Politiques existent mais appliquées de façon irrégulière

Tier 3 — Répétable
  "On a des processus écrits qu'on suit systématiquement"
  → SMSI structuré, réponse aux incidents testée régulièrement

Tier 4 — Adaptatif
  "On s'adapte en temps réel aux nouvelles menaces"
  → Threat intelligence active, partage avec le secteur (ISACs)
```

> [!tip] Quelle cible viser ?
> La plupart des organisations visent **Tier 3** comme objectif raisonnable. Le Tier 4 est atteint par les grandes organisations critiques (banques, infrastructures nationales, GAFAM). Passer de Tier 1 à Tier 2 est souvent la priorité absolue des PME.

### 6.4 Profils et plans d'action

Un **Profil NIST CSF** est la cartographie de l'état actuel vs l'état cible pour chaque sous-catégorie du framework.

```
PROFIL ACTUEL                    PROFIL CIBLE
(où on est aujourd'hui)          (où on veut être dans 12 mois)
        │                                 │
        └──────────────┬──────────────────┘
                       │
               ANALYSE DES ÉCARTS
               (Gap Analysis)
                       │
               PLAN D'ACTION
               (Priorisation des actions)
```

**Exemple de profil simplifié :**

| Fonction | Sous-catégorie | État actuel | État cible | Priorité | Échéance |
|---|---|---|---|---|---|
| IDENTIFY | ID.AM-1 (Inventaire actifs) | Tier 1 | Tier 3 | Haute | Q1 2025 |
| PROTECT | PR.AA-1 (MFA) | Tier 2 | Tier 4 | Critique | J+30 |
| DETECT | DE.CM-1 (Surveillance réseau) | Tier 1 | Tier 3 | Haute | Q2 2025 |
| RESPOND | RS.RP-1 (Plan de réponse) | Tier 1 | Tier 3 | Haute | Q1 2025 |
| RECOVER | RC.RP-1 (Plan de reprise) | Tier 2 | Tier 3 | Moyenne | Q3 2025 |

---

## 7. SIEM, SOC et Organisation de la Sécurité

### 7.1 Le RSSI — Responsable de la Sécurité des Systèmes d'Information

Le **RSSI** (ou CISO en anglais — Chief Information Security Officer) est le responsable de la politique de sécurité au sein de l'organisation.

```
Position idéale dans l'organigramme :

Direction Générale
    │
    ├── DSI (Direction des Systèmes d'Information)
    │   └── Équipes techniques
    │
    ├── RSSI ←── rattaché directement à la DG (indépendance)
    │   ├── Pilotage SMSI
    │   ├── Analyse de risques
    │   ├── Politique de sécurité
    │   ├── Sensibilisation
    │   └── Gestion des incidents (coordination)
    │
    └── Conformité / Juridique
```

**Missions principales du RSSI :**

| Mission | Activité | Fréquence |
|---|---|---|
| Stratégie | Définir et maintenir la politique de sécurité | Annuelle + événementielle |
| Risques | Piloter l'analyse et le traitement des risques | Annuelle + continue |
| Conformité | Assurer la conformité RGPD, ISO 27001, etc. | Continue |
| Incidents | Coordonner la réponse aux incidents majeurs | Événementielle |
| Sensibilisation | Animer les formations sécurité | Trimestrielle |
| Veille | Surveiller les menaces et vulnérabilités | Quotidienne |
| Reporting | Rendre compte à la direction | Mensuelle / trimestrielle |

> [!warning] RSSI ≠ Administrateur sécurité
> Le RSSI est un rôle **stratégique et organisationnel**, pas uniquement technique. Il n'opère pas les outils lui-même (c'est le rôle du SOC et des administrateurs sécurité), mais il définit la stratégie, pilote la conformité et représente la sécurité au niveau direction.

### 7.2 Le SOC — Security Operations Center

Le **SOC** est l'équipe (et souvent le lieu physique) dédiée à la **surveillance continue** de la sécurité et à la **réponse aux incidents**.

```
                        ┌─────────────────────────────────┐
                        │           SOC MANAGER           │
                        │  Coordination, reporting, KPI   │
                        └──────────────┬──────────────────┘
                                       │
               ┌───────────────────────┼───────────────────────┐
               │                       │                       │
    ┌──────────▼──────────┐  ┌──────────▼──────────┐  ┌────────▼────────────┐
    │      ANALYSTE L1    │  │      ANALYSTE L2    │  │      ANALYSTE L3    │
    │                     │  │                     │  │                     │
    │  • Tri des alertes  │  │  • Investigation    │  │  • Threat Hunting   │
    │  • 1er diagnostic   │  │  • Qualification    │  │  • Forensics        │
    │  • Escalade si L2   │  │  • Containment      │  │  • Malware analysis │
    │  • 24/7 en rotation │  │  • Escalade si L3   │  │  • Reverse eng.     │
    └─────────────────────┘  └─────────────────────┘  └─────────────────────┘
               │                       │                       │
               └───────────────────────┼───────────────────────┘
                                       │
                              ┌────────▼────────┐
                              │   THREAT INTEL  │
                              │                 │
                              │ • IOC/IOA feed  │
                              │ • CTI reports   │
                              │ • OSINT         │
                              └─────────────────┘
```

### 7.3 Rôles des analystes L1, L2, L3

#### Analyste SOC L1 — Tier 1 (Triage)

**Profil :** Junior, 0-2 ans d'expérience. Souvent en alternance ou sortie d'école.

**Responsabilités :**
- Surveiller les tableaux de bord SIEM (Splunk, QRadar, Elastic)
- Trier les alertes : vrai positif / faux positif ?
- Appliquer les runbooks de réponse standard
- Escalader les cas complexes vers L2
- Documenter chaque ticket d'incident

```
Workflow L1 :

Alerte SIEM → Analyse initiale → Faux positif ? → Fermer + documenter
                                      │
                                      Non
                                      │
                                 Runbook disponible ?
                                 ├── Oui → Appliquer runbook
                                 └── Non → Escalader L2
```

#### Analyste SOC L2 — Tier 2 (Investigation)

**Profil :** Confirmé, 2-5 ans d'expérience.

**Responsabilités :**
- Mener les investigations approfondies
- Corréler plusieurs alertes pour reconstituer un scénario d'attaque
- Décider des mesures de confinement (isoler un poste, bloquer une IP)
- Rédiger les rapports d'incidents
- Mettre à jour les règles de détection

```bash
# Exemple d'investigation L2 : analyse d'une connexion suspecte

# 1. Récupérer les logs de connexion d'un utilisateur suspect
grep "user: johndoe" /var/log/auth.log | tail -50

# 2. Analyser les processus lancés par cet utilisateur
ps aux | grep johndoe

# 3. Vérifier les connexions réseau actives
ss -tulpn | grep <PID_suspect>

# 4. Examiner les fichiers récemment accédés
find /home/johndoe -newer /home/johndoe/.bash_history -type f 2>/dev/null

# 5. Analyser les commandes bash history
cat /home/johndoe/.bash_history

# 6. Vérifier la persistance (crontabs, services)
crontab -u johndoe -l
systemctl list-units --type=service | grep -i unknown
```

#### Analyste SOC L3 — Tier 3 (Expert)

**Profil :** Expert, 5+ ans. Peut être expert malware, forensics, reverse engineering.

**Responsabilités :**
- Threat Hunting proactif (chercher des menaces cachées sans alerte)
- Analyse de malwares (reverse engineering)
- Forensics numérique
- Développement de nouvelles règles de détection
- Red Team / Purple Team

```python
# Exemple de Threat Hunting — recherche de Living-off-the-Land (LotL)
# Techniques d'attaquants qui utilisent des outils légitimes du système

# Requête SIEM (pseudo-code Splunk)
hunting_query = """
index=windows_events EventCode=4688
| where ParentImage IN (
    "C:\\Windows\\System32\\cmd.exe",
    "C:\\Windows\\System32\\powershell.exe",
    "C:\\Windows\\SysWow64\\wscript.exe"
)
| where Image IN (
    "C:\\Windows\\System32\\certutil.exe",    -- souvent abusé pour DL
    "C:\\Windows\\System32\\bitsadmin.exe",   -- download LOLBin
    "C:\\Windows\\System32\\mshta.exe",       -- execution HTA
    "C:\\Windows\\System32\\regsvr32.exe"     -- DLL hijacking
)
| stats count by _time span=1h, Image, ParentImage, CommandLine, User
| where count > 3
| sort -count
"""
```

### 7.4 SIEM — Security Information and Event Management

Le **SIEM** est l'outil central du SOC. Il collecte, normalise, corrèle et analyse les logs de tous les systèmes de l'organisation pour détecter des comportements anormaux.

```
Sources de logs → Collecte → Normalisation → Stockage → Corrélation → Alertes
    │
    ├── Firewalls (Palo Alto, Fortinet)
    ├── Serveurs (Windows Event Logs, syslog Linux)
    ├── Applications (Apache, Nginx, IIS)
    ├── Bases de données (Oracle Audit, SQL Server)
    ├── EDR (CrowdStrike, SentinelOne, Defender)
    ├── IAM (Active Directory, Okta)
    └── Cloud (AWS CloudTrail, Azure Monitor)
```

**Solutions SIEM du marché :**

| Solution | Éditeur | Type | Points forts |
|---|---|---|---|
| Splunk | Splunk Inc. | Commercial | Puissant, très répandu |
| IBM QRadar | IBM | Commercial | Excellent pour le réseau |
| Elastic SIEM | Elastic | Open source + Commercial | Flexible, Kibana |
| Microsoft Sentinel | Microsoft | Cloud (Azure) | Intégration native Azure/M365 |
| Wazuh | Wazuh Inc. | Open source | Gratuit, très complet |
| Graylog | Graylog | Open source | Logging centralisé |

---

## 8. Matrices de Détection — MITRE ATT&CK et Kill Chain

### 8.1 La Kill Chain de Lockheed Martin

La **Cyber Kill Chain** a été développée par Lockheed Martin en 2011. Elle modélise les **7 étapes séquentielles** d'une cyberattaque avancée (APT — Advanced Persistent Threat).

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CYBER KILL CHAIN                                     │
│                        Lockheed Martin, 2011                                │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
┌──────────────┐   ┌───────────────▼─────────────┐   ┌──────────────────────┐
│ 1. RECON     │   │  2. WEAPONIZATION           │   │  3. DELIVERY         │
│              │   │                             │   │                      │
│ Collecte     │──►│ Création du payload         │──►│ Envoi de l'arme      │
│ d'infos sur  │   │ (malware + exploit)         │   │ (email, USB, web)    │
│ la cible     │   │                             │   │                      │
└──────────────┘   └─────────────────────────────┘   └──────────────────────┘
                                                              │
┌──────────────┐   ┌─────────────────────────────┐   ┌───────▼──────────────┐
│ 7. ACTIONS   │   │  6. COMMAND & CONTROL       │   │  4. EXPLOITATION     │
│  ON OBJECT.  │   │                             │   │                      │
│              │◄──│ Canal de communication       │◄──│ Déclenchement de     │
│ Objectif     │   │ avec l'attaquant (C2/C&C)   │   │ l'exploit sur la     │
│ atteint      │   │                             │   │ vulnérabilité        │
└──────────────┘   └─────────────────────────────┘   └──────────────────────┘
                                                              │
                              ┌──────────────────────────────▼───────────────┐
                              │              5. INSTALLATION                  │
                              │                                               │
                              │  Persistence : backdoor, rootkit, scheduled   │
                              │  task, registry key, startup script           │
                              └───────────────────────────────────────────────┘
```

**Détail de chaque étape avec exemples :**

| Étape | Description | Techniques d'attaque | Contre-mesures |
|---|---|---|---|
| **1. Reconnaissance** | OSINT, scan de ports, LinkedIn, WHOIS | Shodan, theHarvester, Nmap | Réduire surface exposée, surveiller mentions |
| **2. Weaponization** | Création de l'outil d'attaque | Metasploit, exploits CVE, macro Office | Patch management, EDR |
| **3. Delivery** | Envoi du vecteur d'infection | Phishing, watering hole, USB | Filtre email, formation, DLP |
| **4. Exploitation** | Déclenchement de la vulnérabilité | Buffer overflow, SQLi, XSS, 0-day | WAF, MFA, patch prioritaire |
| **5. Installation** | Persistance sur la machine | Crontab, Registry, Services malveillants | EDR, surveillance intégrité |
| **6. C2** | Communication avec l'attaquant | HTTP/HTTPS covert channel, DNS tunneling | Proxy, inspection TLS, UEBA |
| **7. Actions** | Objectif final | Exfiltration, chiffrement, destruction | DLP, sauvegardes, segmentation |

> [!tip] Intérêt pour le défenseur
> La Kill Chain permet de comprendre qu'**interrompre à n'importe quelle étape suffit à bloquer l'attaque**. Plus l'interruption est précoce, moins l'impact est grave. Idéalement, bloquer dès la livraison (étape 3) avant toute exécution de code.

### 8.2 MITRE ATT&CK

**MITRE ATT&CK** (Adversarial Tactics, Techniques and Common Knowledge) est une base de connaissances publique qui référence les **tactiques et techniques utilisées par les attaquants réels**, basées sur des observations d'incidents réels.

URL : https://attack.mitre.org

```
Structure MITRE ATT&CK

TACTIQUE (POURQUOI — l'objectif tactique de l'attaquant)
    └── TECHNIQUE (COMMENT — la méthode générale)
            └── SOUS-TECHNIQUE (COMMENT précisément)
                    └── PROCÉDURE (implémentation spécifique d'un groupe APT)
```

**Les 14 tactiques ATT&CK for Enterprise :**

```
TA0043 — Reconnaissance          : Collecter des infos sur la cible
TA0042 — Resource Development    : Construire l'infrastructure d'attaque
TA0001 — Initial Access          : Entrer dans le système cible
TA0002 — Execution               : Exécuter du code malveillant
TA0003 — Persistence             : Maintenir son accès
TA0004 — Privilege Escalation    : Obtenir des droits plus élevés
TA0005 — Defense Evasion         : Éviter la détection
TA0006 — Credential Access       : Voler des identifiants
TA0007 — Discovery               : Découvrir l'environnement
TA0008 — Lateral Movement        : Se déplacer vers d'autres systèmes
TA0009 — Collection              : Rassembler les données ciblées
TA0011 — Command and Control     : Communiquer avec les systèmes compromis
TA0010 — Exfiltration            : Extraire les données volées
TA0040 — Impact                  : Perturber, détruire, chiffrer
```

**Exemple de technique et sous-technique :**

```
Tactique    : TA0006 — Credential Access
Technique   : T1110  — Brute Force
                └── T1110.001 — Password Guessing (mots de passe communs)
                └── T1110.002 — Password Cracking (hashes volés)
                └── T1110.003 — Password Spraying (1 mdp, N utilisateurs)
                └── T1110.004 — Credential Stuffing (bases leakées)

Détection   :
  - Surveiller EventCode 4625 (échec auth Windows) × 10 en < 5 min
  - Alerter si tentative depuis IP inconnue + heure inhabituelle
  - SIEM rule : count(failed_login) > 10 where timeframe=5min

Mitigation  :
  - MFA (M1032)
  - Verrouillage de compte (M1036)
  - Complexité mots de passe (M1027)
  - Credential Guard (M1043)
```

**Utilisation pratique de MITRE ATT&CK :**

```python
# Scénario : un analyste SOC reçoit une alerte sur du PowerShell suspect
# Il utilise ATT&CK pour qualifier et enrichir l'incident

incident = {
    "alerte": "PowerShell encoded command detected",
    "log": "powershell.exe -NoP -NonI -W Hidden -Enc JABjAGwAaQBlAG4AdA...",
    "hote": "DESKTOP-HR-042",
    "utilisateur": "n.martin",
    "heure": "2024-03-15 02:47:33"
}

# Mapping ATT&CK
techniques_identifiees = [
    {
        "id": "T1059.001",
        "nom": "PowerShell",
        "tactique": "Execution",
        "sous_technique": "Command and Scripting Interpreter: PowerShell",
        "indicateurs": [
            "Base64 encoding (-Enc)",
            "Exécution silencieuse (-NoP -NonI)",
            "Fenêtre cachée (-W Hidden)",
            "Heure anormale (02h47)",
        ]
    },
    {
        "id": "T1027",
        "nom": "Obfuscated Files or Information",
        "tactique": "Defense Evasion",
        "explication": "Le payload est encodé en Base64 pour éviter les signatures"
    }
]

# Sévérité et recommandation
sévérité = "HAUTE"  # Heure anormale + obfuscation + utilisateur RH = suspect
action = "ESCALADE L2 + Isolation préventive de DESKTOP-HR-042"
```

**MITRE ATT&CK Navigator** permet de visualiser les couvertures de détection :

```
Heatmap de couverture (exemple simplifié)

Tactique          │ Nb techniques │ Couverture SOC │ Lacunes
──────────────────┼───────────────┼────────────────┼──────────
Initial Access    │      9        │      7/9 (78%) │ T1195 (supply chain)
Execution         │     14        │     10/14 (71%)│ T1204 (user exec)
Persistence       │     19        │      8/19 (42%)│ LACUNE CRITIQUE
Privilege Esc.    │     13        │      9/13 (69%)│ T1068 (exploit)
Defense Evasion   │     42        │     15/42 (36%)│ LACUNE CRITIQUE
```

---

## 9. Comparaison des Référentiels de Sécurité

### 9.1 Vue d'ensemble : ISO 27001, NIST CSF, SOC 2, PCI-DSS

| Critère | ISO 27001 | NIST CSF | SOC 2 | PCI-DSS |
|---|---|---|---|---|
| **Organisme** | ISO/IEC | NIST (US Gov.) | AICPA | PCI Security Standards Council |
| **Type** | Norme certifiable | Cadre volontaire | Rapport d'audit | Norme certifiable |
| **Périmètre** | Toute organisation | Toute organisation | Prestataires de service | Toute org. traitant cartes bancaires |
| **Obligatoire ?** | Non (sauf contrat) | Non | Non (sauf contrat) | Oui si cartes bancaires |
| **Certification** | Oui (tiers accrédité) | Non | Rapport SOC 2 Type I/II | QSA (Qualified Security Assessor) |
| **Durée certificat** | 3 ans + audits annuels | N/A | Type I : instantané ; Type II : 6-12 mois | Annuel |
| **Coût** | 15 000–100 000 € | Gratuit (DIY) | 20 000–200 000 € | 20 000–500 000 € |
| **Centré sur** | Gestion des risques | Fonctions cyber | Confiance des clients | Sécurité des données cartes |
| **Adopté en priorité** | Europe, international | USA + international | SaaS, cloud, B2B | E-commerce, finance mondiale |

### 9.2 ISO 27001 vs NIST CSF — Complémentarité

Ces deux référentiels ne s'opposent pas : ils sont **complémentaires**.

```
                ISO 27001                       NIST CSF
                    │                               │
      ┌─────────────┴──────────────┐   ┌────────────┴────────────┐
      │ QUOI faire (exigences)     │   │ COMMENT penser (cadre)  │
      │ Certifiable                │   │ Communication interne   │
      │ Basé sur risques           │   │ Visualisation maturité  │
      │ 93 contrôles précis        │   │ 6 fonctions flexibles   │
      │ Audit formel               │   │ Profils et roadmap      │
      └────────────────────────────┘   └─────────────────────────┘
                    │                               │
                    └───────────────┬───────────────┘
                                    │
                         USAGE COMBINÉ RECOMMANDÉ
                         • NIST CSF : diagnostic initial et roadmap
                         • ISO 27001 : cadre d'implémentation et certification
                         • Mapping officiel NIST CSF ↔ ISO 27001 disponible sur nist.gov
```

### 9.3 SOC 2 — Type I et Type II

**SOC 2** (Service Organization Control 2) est un rapport d'audit défini par l'AICPA. Il évalue les prestataires de service selon 5 **Trust Service Criteria** :

| Critère | Description | Obligatoire ? |
|---|---|---|
| **Security** | Protection contre les accès non autorisés | OUI (toujours) |
| **Availability** | Disponibilité des systèmes selon les SLA | Non |
| **Processing Integrity** | Traitement complet, valide et en temps voulu | Non |
| **Confidentiality** | Protection des infos désignées confidentielles | Non |
| **Privacy** | Collecte et traitement des données personnelles | Non |

**Type I vs Type II :**

```
SOC 2 Type I
│
├── Photo instantanée à une date T
├── Vérifie que les contrôles EXISTENT
├── Durée : quelques semaines
└── Valeur : limitée (aucun test dans le temps)

SOC 2 Type II
│
├── Observation sur une période de 6-12 mois
├── Vérifie que les contrôles FONCTIONNENT dans le temps
├── Durée : 6-12 mois de période d'observation
└── Valeur : haute — preuve d'efficacité réelle
```

> [!info] SOC 2 pour les SaaS
> Si vous travaillez pour une startup SaaS B2B, le SOC 2 Type II sera souvent exigé par les clients enterprise avant de signer un contrat. C'est devenu le standard de confiance pour les prestataires cloud aux États-Unis.

### 9.4 PCI-DSS — Standard de sécurité des données de paiement

**PCI-DSS** (Payment Card Industry Data Security Standard) s'applique à **toute organisation** qui stocke, traite ou transmet des données de titulaires de carte bancaire.

**Les 12 exigences PCI-DSS v4.0 :**

```
1. Installer et maintenir des contrôles réseau
   → Pare-feu, segmentation réseau, documentation des flux

2. Appliquer des configurations sécurisées par défaut
   → Hardening OS, suppression des comptes/ports par défaut

3. Protéger les données de compte stockées
   → Chiffrement des PAN (numéros de carte), tokenisation

4. Protéger les données en transit
   → TLS 1.2+ obligatoire, interdiction de protocoles obsolètes

5. Se protéger contre les logiciels malveillants
   → Antivirus, protection des systèmes exposés

6. Développer et maintenir des systèmes et logiciels sécurisés
   → SDLC sécurisé, patch management, revue de code

7. Restreindre l'accès aux systèmes et données par besoin métier
   → Contrôle d'accès basé sur les rôles (RBAC)

8. Identifier les utilisateurs et authentifier l'accès
   → MFA obligatoire sur les accès aux environnements cartes

9. Restreindre l'accès physique aux données de compte
   → Contrôles d'accès physiques aux zones sensibles

10. Journaliser et surveiller tous les accès
    → Logs de tous les accès aux données cartes, SIEM

11. Tester régulièrement la sécurité des systèmes et réseaux
    → Tests d'intrusion annuels, scans trimestriels (ASV)

12. Soutenir la sécurité de l'information par des politiques
    → Politiques documentées, formation annuelle, plan de réponse
```

**Niveaux de conformité PCI-DSS selon le volume :**

| Niveau | Volume transactions/an | Obligation |
|---|---|---|
| Level 1 | > 6 millions | Audit annuel par QSA + ASV scan trimestriel |
| Level 2 | 1-6 millions | SAQ (Self-Assessment Questionnaire) + ASV scan |
| Level 3 | 20 000 - 1 million (e-commerce) | SAQ + ASV scan |
| Level 4 | < 20 000 (e-commerce) ou < 1M (autres) | SAQ recommandé |

### 9.5 Tableau de synthèse — Quand choisir quoi ?

| Contexte | Référentiel recommandé |
|---|---|
| Certification internationale pour appels d'offres | ISO 27001 |
| Structurer sa roadmap cybersécurité (PME) | NIST CSF |
| Client US exige une preuve de sécurité (SaaS) | SOC 2 Type II |
| Site e-commerce acceptant des cartes bancaires | PCI-DSS |
| Secteur santé (données médicales, US) | HIPAA |
| Services numériques en France (OIV/OSE) | NIS2 + ISO 27001 |
| Secteur public français | RGS (Référentiel Général de Sécurité) |

---

## 10. Exercices Pratiques

### Exercice 1 — Analyse de risques (niveau débutant)

**Contexte :** Vous êtes RSSI d'une startup de 20 personnes développant une application SaaS de gestion RH. L'application stocke des données personnelles de 5 000 employés clients (noms, salaires, numéros de sécurité sociale).

**Travail demandé :**

1. Identifier **5 actifs informationnels** et les classifier selon la triade CIA
2. Pour chaque actif, identifier **2 menaces** et **1 vulnérabilité** associée
3. Calculer le **score de risque brut** pour 3 scénarios de votre choix
4. Proposer un **plan de traitement** avec l'option choisie (réduire/transférer/accepter/éviter) et 2 actions concrètes par risque

**Template de rendu :**

```markdown
## Actif : [Nom]
- Confidentialité : Faible / Moyenne / Élevée
- Intégrité : Faible / Moyenne / Élevée
- Disponibilité : Faible / Moyenne / Élevée

## Scénario de risque
- Menace : [...]
- Vulnérabilité : [...]
- Vraisemblance : 1-4 (justifier)
- Impact : 1-4 (justifier)
- Score brut : V × I = [...]
- Niveau : Négligeable / Faible / Moyen / Élevé / Critique

## Plan de traitement
- Option : Réduire / Transférer / Accepter / Éviter
- Action 1 : [...] | Responsable : [...] | Délai : [...]
- Action 2 : [...] | Responsable : [...] | Délai : [...]
- Score résiduel estimé : [...]
```

---

### Exercice 2 — Mapping MITRE ATT&CK (niveau intermédiaire)

**Contexte :** Un analyste SOC L1 reçoit les alertes suivantes un lundi matin :

```
Alert 1 [08:14] — SIEM
  Source IP: 185.220.101.47 (Tor exit node)
  Dest: webserver01:443
  Pattern: 4625 × 847 en 12 minutes
  User: admin, administrator, root, sa, sysadmin (variantes)

Alert 2 [09:03] — EDR (CrowdStrike)
  Host: DESKTOP-DEV-007
  Process: powershell.exe
  Parent: outlook.exe
  Commandline: powershell -NoP -NonI -W Hidden -Enc SQBFAFgA...
  User: p.dupont

Alert 3 [10:41] — Proxy
  Host: SRV-APP-02
  Destination: pastebin.com
  Bytes out: 847 MB (anomalie : baseline = 2 MB/jour)
  User: service_account_api
```

**Travail demandé :**

1. Pour chaque alerte, identifier la **technique MITRE ATT&CK** (T-XXXX) correspondante
2. Déterminer la **priorité de traitement** (Critique/Haute/Moyenne) et la justifier
3. Décrire les **3 premières actions** que vous effectueriez en tant qu'analyste L1
4. Identifier si ces 3 alertes peuvent être **liées** à une même attaque — quelle phase de la Kill Chain représente chacune ?

---

### Exercice 3 — Rédaction d'une politique de sécurité (niveau avancé)

**Contexte :** Votre RSSI vous demande de rédiger une **Politique de Gestion des Accès Privilégiés (PAM)** pour votre organisation.

**Contraintes :**
- L'organisation a 15 administrateurs système avec accès root/admin
- Plusieurs prestataires externes accèdent ponctuellement aux serveurs
- L'audit ISO 27001 est dans 3 mois
- Contrôles à couvrir : A.5.15 (contrôle d'accès), A.8.2 (droits d'accès privilégiés), A.8.18 (utilisation de programmes utilitaires privilégiés)

**Travail demandé :**

Rédiger une politique de 1 à 2 pages couvrant :
1. Objectif et périmètre
2. Principes (moindre privilège, besoin d'en connaître, séparation des tâches)
3. Processus de demande et d'attribution des accès privilégiés
4. Exigences techniques (PAM tool, MFA, session recording, time-limited access)
5. Revue et révocation des accès
6. Sanctions en cas de non-respect

---

### Exercice 4 — Construction d'un profil NIST CSF (niveau avancé)

**Contexte :** Vous rejoignez en tant que consultant une PME de 50 personnes (cabinet d'expertise comptable). Elle n'a jamais fait d'audit de sécurité. Elle possède :
- Serveur de fichiers en local (données clients confidentielles)
- 3 serveurs Windows Server 2016 (dont 1 non patché depuis 18 mois)
- 50 postes Windows 10 (15 avec des versions non supportées)
- Accès RDP exposé directement sur Internet (port 3389 public)
- Aucun SIEM, aucun antivirus centralisé
- Aucune politique de sauvegarde formalisée
- Aucune sensibilisation sécurité

**Travail demandé :**

1. Attribuer un **Tier de maturité** (1-4) pour chacune des 6 fonctions NIST CSF
2. Identifier les **5 actions prioritaires** (Quick Wins) avec estimation d'effort et impact
3. Proposer un **plan d'action sur 12 mois** avec jalons trimestriels
4. Estimer quel **Tier** l'organisation atteindra à l'issue des 12 mois si le plan est suivi

---

## 11. Résumé et Points Clés

> [!info] À retenir absolument pour l'examen

**ISO 27001 :**
- Norme certifiable basée sur les risques, cycle PDCA
- 93 contrôles en 4 domaines (organisationnel, humain, physique, technologique)
- Documents obligatoires : SoA, analyse de risques, PTR, politique de sécurité
- 4 options de traitement du risque : réduire, transférer, accepter, éviter
- Certification valable 3 ans avec audits de surveillance annuels

**NIST CSF 2.0 :**
- Cadre volontaire, non certifiable, 6 fonctions (GOVERN, IDENTIFY, PROTECT, DETECT, RESPOND, RECOVER)
- 4 Tiers de maturité (Partiel → Adaptatif)
- Outil de communication et de roadmap, complémentaire à ISO 27001

**Organisation :**
- RSSI : rôle stratégique (politique, risques, conformité, reporting)
- SOC L1 : triage des alertes — L2 : investigation — L3 : threat hunting/forensics
- SIEM : collecte, corrélation, détection centralisée

**Matrices d'attaque :**
- Kill Chain : 7 étapes séquentielles (Recon → Actions)
- MITRE ATT&CK : 14 tactiques, 200+ techniques, base de référence du SOC

**Comparaison référentiels :**
- ISO 27001 : gouvernance globale + certification
- NIST CSF : roadmap et communication
- SOC 2 : confiance client SaaS (Type II = test dans la durée)
- PCI-DSS : données bancaires (obligatoire si traitement cartes)

> [!warning] Erreurs fréquentes à éviter
> - Confondre **menace** (agent externe) et **vulnérabilité** (faiblesse interne)
> - Croire que l'ISO 27001 exige d'implémenter les **93 contrôles** — faux, seuls les contrôles applicables au périmètre et aux risques identifiés sont requis
> - Confondre **RTO** (délai de reprise) et **RPO** (perte de données tolérée)
> - Croire que le NIST CSF Tier 4 est l'objectif universel — Tier 3 suffit pour la grande majorité des organisations
> - Traiter un risque **sans documenter** la décision — l'acceptation non documentée est une non-conformité ISO 27001
