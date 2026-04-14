# Introduction au Cloud Computing

## Qu'est-ce que le Cloud Computing ?

Le **Cloud Computing** (informatique en nuage) designe l'acces a des ressources informatiques (serveurs, stockage, bases de donnees, reseaux, logiciels) via Internet, fournies par un prestataire externe. Au lieu de posseder et gerer votre propre infrastructure physique, vous **louez** des ressources a la demande aupres d'un fournisseur cloud.

> [!tip] Analogie
> Pensez a l'**electricite**. Avant les reseaux electriques, chaque usine devait posseder son propre generateur : l'acheter, l'installer, le maintenir, le reparer, le remplacer. Aujourd'hui, vous branchez une prise et vous payez ce que vous consommez. Le cloud fait la meme chose pour l'informatique : au lieu de gerer vos propres serveurs, vous "branchez" votre application sur l'infrastructure d'un fournisseur et vous payez a l'usage.

---

## On-Premise vs Cloud

### Le modele traditionnel : On-Premise

**On-Premise** (sur site) signifie que l'entreprise possede et gere tout elle-meme : les serveurs physiques, le reseau, la climatisation, la securite physique, les licences logicielles, les mises a jour.

### Comparaison detaillee

| Critere | On-Premise | Cloud |
|---|---|---|
| **Cout initial** | Tres eleve (CAPEX) | Faible ou nul (OPEX) |
| **Delai de deploiement** | Semaines/mois | Minutes |
| **Scalabilite** | Limitee, planification requise | Quasi-illimitee, a la demande |
| **Maintenance** | A votre charge | Geree par le fournisseur |
| **Controle** | Total | Partiel (selon le modele) |
| **Securite physique** | A votre charge | Geree par le fournisseur |
| **Disponibilite** | Depend de votre infrastructure | SLA 99.9%+ |
| **Mise a jour** | Manuelle | Automatique (selon le service) |

> [!info] CAPEX vs OPEX
> - **CAPEX** (Capital Expenditure) : depenses d'investissement. Vous achetez du materiel (serveurs, switches, baies de stockage). L'argent est depense d'avance.
> - **OPEX** (Operational Expenditure) : depenses operationnelles. Vous payez un abonnement mensuel ou a l'usage. Pas de gros investissement initial.
>
> Le cloud transforme le CAPEX en OPEX, ce qui est souvent plus flexible pour les entreprises.

```
  ON-PREMISE                              CLOUD
  ==========                              =====

  +------------------+                    +------------------+
  | Votre Application|                    | Votre Application|
  +------------------+                    +------------------+
  | Middleware       |                    |                  |
  +------------------+                    |   Tout ceci est  |
  | OS               |                    |   gere par le    |
  +------------------+    Vous gerez     |   fournisseur    |
  | Virtualisation   |    TOUT  ====>    |   cloud          |
  +------------------+                    |   (selon le      |
  | Serveurs         |                    |    modele)       |
  +------------------+                    |                  |
  | Stockage         |                    +------------------+
  +------------------+                    | Infrastructure   |
  | Reseau           |                    | du fournisseur   |
  +------------------+                    +------------------+
```

---

## Les 3 Modeles de Service

Le cloud propose trois niveaux d'abstraction. Plus vous montez dans la pile, moins vous avez a gerer vous-meme.

### IaaS - Infrastructure as a Service

Vous louez l'infrastructure de base : machines virtuelles, stockage, reseau. Vous gerez tout le reste (OS, middleware, application).

**Exemples** : AWS EC2, Google Compute Engine, Azure Virtual Machines

**Cas d'usage** : Migration "lift-and-shift", besoin de controle total sur l'OS, applications legacy.

### PaaS - Platform as a Service

Le fournisseur gere l'infrastructure ET la plateforme (OS, runtime, middleware). Vous deployez votre code directement.

**Exemples** : Google App Engine, AWS Elastic Beanstalk, Azure App Service, Heroku

**Cas d'usage** : Developpement rapide d'applications web, APIs, sans se soucier de l'infrastructure.

### SaaS - Software as a Service

Le fournisseur gere absolument tout. Vous utilisez un logiciel pret a l'emploi via un navigateur.

**Exemples** : Gmail, Slack, Salesforce, Microsoft 365, Dropbox

**Cas d'usage** : Utilisation d'outils sans aucune gestion technique.

### Diagramme de la pile : qui gere quoi ?

```
                  On-Premise    IaaS         PaaS         SaaS
                 +----------+----------+----------+----------+
  Applications   | VOUS     | VOUS     | VOUS     | PROVIDER |
                 +----------+----------+----------+----------+
  Donnees        | VOUS     | VOUS     | VOUS     | PROVIDER |
                 +----------+----------+----------+----------+
  Runtime        | VOUS     | VOUS     | PROVIDER | PROVIDER |
                 +----------+----------+----------+----------+
  Middleware     | VOUS     | VOUS     | PROVIDER | PROVIDER |
                 +----------+----------+----------+----------+
  OS             | VOUS     | VOUS     | PROVIDER | PROVIDER |
                 +----------+----------+----------+----------+
  Virtualisation | VOUS     | PROVIDER | PROVIDER | PROVIDER |
                 +----------+----------+----------+----------+
  Serveurs       | VOUS     | PROVIDER | PROVIDER | PROVIDER |
                 +----------+----------+----------+----------+
  Stockage       | VOUS     | PROVIDER | PROVIDER | PROVIDER |
                 +----------+----------+----------+----------+
  Reseau         | VOUS     | PROVIDER | PROVIDER | PROVIDER |
                 +----------+----------+----------+----------+

  VOUS     = Vous gerez cette couche
  PROVIDER = Le fournisseur cloud gere cette couche
```

> [!example] Analogie immobiliere
> - **On-Premise** = Construire sa propre maison. Vous gerez tout : terrain, construction, plomberie, electricite.
> - **IaaS** = Louer un appartement vide. Les murs et la plomberie sont la, mais vous amenagez tout.
> - **PaaS** = Louer un appartement meuble. Vous apportez juste vos affaires personnelles.
> - **SaaS** = Aller a l'hotel. Vous arrivez, tout est pret, vous utilisez.

---

## Modeles de Deploiement

### Cloud Public

Les ressources sont partagees entre plusieurs clients (multi-tenant) sur l'infrastructure du fournisseur.

- **Avantages** : Cout reduit, scalabilite, pas de maintenance
- **Inconvenients** : Moins de controle, donnees sur l'infrastructure d'un tiers
- **Exemples** : AWS, GCP, Azure

### Cloud Prive

L'infrastructure cloud est dediee a une seule organisation, soit sur site, soit hebergee par un tiers.

- **Avantages** : Controle total, securite renforcee, conformite reglementaire
- **Inconvenients** : Cout eleve, maintenance a votre charge
- **Exemples** : VMware vSphere, OpenStack, Azure Stack

### Cloud Hybride

Combinaison de cloud public et prive, avec orchestration entre les deux environnements.

- **Avantages** : Flexibilite, migration progressive, donnees sensibles en prive
- **Inconvenients** : Complexite de gestion, integration entre les environnements
- **Cas d'usage** : Banques, sante (donnees sensibles en prive, reste en public)

### Multi-Cloud

Utilisation de **plusieurs fournisseurs** de cloud public simultanement (ex : AWS + GCP).

- **Avantages** : Eviter le vendor lock-in, best-of-breed, resilience
- **Inconvenients** : Complexite operationnelle, couts de gestion
- **Cas d'usage** : Grandes entreprises voulant eviter la dependance a un seul fournisseur

```
  +------------------------------------------------------------------+
  |                    MODELES DE DEPLOIEMENT                         |
  |                                                                  |
  |  PUBLIC           PRIVE           HYBRIDE          MULTI-CLOUD   |
  |  +--------+      +--------+      +--------+       +--------+    |
  |  |AWS/GCP/|      |  Votre |      | Public |       |  AWS   |    |
  |  | Azure  |      | infra  |      +---+----+       +---+----+    |
  |  |Partage |      |Dedie a |          |                |         |
  |  | entre  |      |  vous  |      +---+----+       +---+----+    |
  |  |clients |      |  seul  |      | Prive  |       |  GCP   |    |
  |  +--------+      +--------+      +--------+       +--------+    |
  +------------------------------------------------------------------+
```

---

## Les 3 Grands Fournisseurs Cloud

### AWS (Amazon Web Services)

- **Leader du marche** (~32% de part de marche)
- Lance en 2006, le plus ancien et le plus mature
- **Plus de 200 services** : compute, stockage, ML, IoT, bases de donnees...
- Ecosysteme immense, communaute tres active
- Complexe mais extremement complet
- Certifications tres valorisees (Solutions Architect, Developer, SysOps)

### Google Cloud Platform (GCP)

- ~10% de part de marche
- **Force** : Big Data, Machine Learning, Kubernetes (cree par Google)
- Infrastructure du moteur de recherche Google
- Reseau mondial extremement performant
- BigQuery pour l'analyse de donnees a grande echelle
- Interface generalement consideree comme plus intuitive

### Microsoft Azure

- ~23% de part de marche, 2e plus grand fournisseur
- **Force** : Integration avec l'ecosysteme Microsoft (Active Directory, Office 365, Windows Server)
- Tres populaire en **entreprise** (migration depuis des environnements Microsoft existants)
- Excellente solution **hybride** (Azure Stack, Azure Arc)
- Support .NET natif

> [!info] Pourquoi connaitre les 3 ?
> En entreprise, vous rencontrerez probablement l'un des trois. Les concepts sont les memes, seuls les noms des services changent. Apprendre l'un facilite l'apprentissage des autres.

---

## Tableau de Correspondance des Services

Les trois fournisseurs offrent des services equivalents avec des noms differents :

| Service | AWS | GCP | Azure |
|---|---|---|---|
| **Machines virtuelles** | EC2 | Compute Engine | Virtual Machines |
| **Serverless (fonctions)** | Lambda | Cloud Functions | Azure Functions |
| **Stockage objet** | S3 | Cloud Storage | Blob Storage |
| **Stockage bloc** | EBS | Persistent Disk | Managed Disks |
| **Base relationnelle** | RDS | Cloud SQL | Azure SQL Database |
| **NoSQL document** | DynamoDB | Firestore | Cosmos DB |
| **Cache** | ElastiCache | Memorystore | Azure Cache |
| **Conteneurs** | ECS / EKS | GKE | AKS |
| **Reseau virtuel** | VPC | VPC | VNet |
| **DNS** | Route 53 | Cloud DNS | Azure DNS |
| **CDN** | CloudFront | Cloud CDN | Azure CDN |
| **Monitoring** | CloudWatch | Cloud Monitoring | Azure Monitor |
| **IAM** | IAM | IAM | Azure AD / RBAC |
| **CLI** | `aws` | `gcloud` | `az` |

> [!tip] Astuce pour apprendre
> Maitrisez **un** fournisseur en profondeur (AWS est le plus demande sur le marche du travail). Les concepts etant transferables, vous apprendrez les autres facilement ensuite.

---

## Regions et Zones de Disponibilite

### Concept

Les fournisseurs cloud possedent des **datacenters** repartis dans le monde entier, organises en une hierarchie :

```
  +---------------------------------------------------------------+
  |                      MONDE                                     |
  |                                                                |
  |  Region : us-east-1 (Virginie)     Region : eu-west-1 (Paris) |
  |  +---------------------------+    +---------------------------+|
  |  |                           |    |                           ||
  |  | AZ: us-east-1a            |    | AZ: eu-west-1a            ||
  |  | +----------+              |    | +----------+              ||
  |  | |Datacenter|              |    | |Datacenter|              ||
  |  | +----------+              |    | +----------+              ||
  |  |                           |    |                           ||
  |  | AZ: us-east-1b            |    | AZ: eu-west-1b            ||
  |  | +----------+              |    | +----------+              ||
  |  | |Datacenter|              |    | |Datacenter|              ||
  |  | +----------+              |    | +----------+              ||
  |  |                           |    |                           ||
  |  | AZ: us-east-1c            |    | AZ: eu-west-1c            ||
  |  | +----------+              |    | +----------+              ||
  |  | |Datacenter|              |    | |Datacenter|              ||
  |  | +----------+              |    | +----------+              ||
  |  +---------------------------+    +---------------------------+|
  +---------------------------------------------------------------+
```

- **Region** : Zone geographique (ex : `eu-west-3` = Paris). Chaque region est **independante** des autres.
- **Zone de disponibilite (AZ)** : Un ou plusieurs datacenters physiquement separes au sein d'une region, connectes par un reseau a faible latence.

### Pourquoi c'est important

1. **Latence** : Placez vos serveurs pres de vos utilisateurs. Un utilisateur a Paris aura une meilleure experience avec un serveur a `eu-west-3` qu'a `us-east-1`.

2. **Redondance** : Deployer dans plusieurs AZ protege contre la panne d'un datacenter. Si `eu-west-3a` tombe, `eu-west-3b` prend le relais.

3. **Conformite** : Certaines reglementations (RGPD en Europe) imposent que les donnees restent dans une region geographique specifique.

4. **Cout** : Les prix varient selon les regions. `us-east-1` est generalement la moins chere pour AWS.

> [!warning] Attention a la region selectionnee
> La region est souvent le premier choix a faire. Si vous creez des ressources dans la mauvaise region, vos serveurs seront loin de vos utilisateurs et vos couts peuvent augmenter. Verifiez toujours la region dans votre console ou CLI avant de deployer.

---

## Free Tiers : Apprendre Gratuitement

Chaque fournisseur propose un **niveau gratuit** pour permettre l'apprentissage et l'experimentation :

### AWS Free Tier

- **12 mois gratuits** : EC2 (t2.micro, 750h/mois), S3 (5 Go), RDS (db.t2.micro, 750h/mois)
- **Toujours gratuit** : Lambda (1M requetes/mois), DynamoDB (25 Go), CloudWatch (basique)
- **Essais** : Certains services gratuits pour une periode limitee

### GCP Free Tier

- **300$ de credits** pendant 90 jours (pour tout essayer)
- **Toujours gratuit** : Compute Engine (e2-micro), Cloud Storage (5 Go), Cloud Functions (2M invocations/mois), BigQuery (1 To de requetes/mois)
- Generalement considere comme le plus genereux pour debuter

### Azure Free Tier

- **200$ de credits** pendant 30 jours
- **12 mois gratuits** : VM (B1S, 750h/mois), Blob Storage (5 Go), SQL Database (250 Go)
- **Toujours gratuit** : Azure Functions (1M executions/mois), Cosmos DB (1000 RU/s)

> [!warning] Surveillez votre facturation !
> Meme avec un free tier, il est possible de depasser les limites et d'etre facture. Configurez **toujours** des alertes de facturation (billing alerts) des la creation de votre compte. Ne laissez jamais de ressources tourner inutilement.

---

## Modeles de Tarification

### On-Demand (a la demande)

Vous payez a l'heure ou a la seconde, sans engagement. Le plus flexible mais le plus cher.

```
Cout = Duree d'utilisation x Prix horaire

Exemple : EC2 t3.medium
  Prix : ~0.0416 $/heure (us-east-1)
  24h/jour x 30 jours = 720 heures
  720 x 0.0416 = ~30 $/mois
```

### Reserved Instances (instances reservees)

Engagement sur 1 ou 3 ans en echange d'une reduction significative (jusqu'a 72%).

```
On-Demand : 30 $/mois = 360 $/an
Reserved 1 an : ~216 $/an (40% de reduction)
Reserved 3 ans : ~108 $/an (70% de reduction)
```

**Cas d'usage** : Charge de travail previsible et stable (serveur de production qui tourne 24/7).

### Spot / Preemptible Instances

Vous utilisez la **capacite inutilisee** du fournisseur a prix reduit (jusqu'a 90% moins cher). En contrepartie, le fournisseur peut **reprendre** l'instance a tout moment (avec un preavis de 2 minutes).

| Fournisseur | Nom | Reduction |
|---|---|---|
| AWS | Spot Instances | Jusqu'a 90% |
| GCP | Preemptible / Spot VMs | Jusqu'a 91% |
| Azure | Spot VMs | Jusqu'a 90% |

**Cas d'usage** : Calculs par lots (batch processing), CI/CD, entrainement ML, taches tolerantes aux interruptions.

> [!tip] Strategie de cout optimale
> En production, combinez les modeles :
> - **Reserved** pour la charge de base (ce qui tourne toujours)
> - **On-Demand** pour les pics previsibles
> - **Spot** pour les taches non-critiques et les calculs lourds

---

## Console Web vs CLI

### Console Web (Interface Graphique)

Chaque fournisseur propose une **interface web** pour gerer les ressources :

- **AWS Console** : `console.aws.amazon.com`
- **GCP Console** : `console.cloud.google.com`
- **Azure Portal** : `portal.azure.com`

La console est ideale pour **decouvrir** les services, visualiser les ressources et faire des operations ponctuelles.

### CLI (Command Line Interface)

Pour l'automatisation et la productivite, chaque fournisseur propose un outil en ligne de commande :

| Fournisseur | CLI | Installation |
|---|---|---|
| AWS | `aws` | `pip install awscli` ou installeur officiel |
| GCP | `gcloud` | Google Cloud SDK |
| Azure | `az` | `pip install azure-cli` ou installeur officiel |

> [!info] Pourquoi utiliser la CLI ?
> - **Reproductibilite** : les commandes peuvent etre scriptes
> - **Automatisation** : integration dans des pipelines CI/CD
> - **Rapidite** : plus rapide que de naviguer dans une interface graphique
> - **Infrastructure as Code** : les commandes documentent votre infrastructure

---

## Configurer le CLI

### AWS CLI

```bash
# Installation
pip install awscli

# OU via l'installeur officiel (recommande)
# https://aws.amazon.com/cli/

# Configuration
aws configure
# AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region name [None]: eu-west-3
# Default output format [None]: json

# Verifier la configuration
aws sts get-caller-identity
```

### GCP CLI (gcloud)

```bash
# Installation : telecharger le Google Cloud SDK
# https://cloud.google.com/sdk/docs/install

# Initialisation
gcloud init
# Cela ouvre un navigateur pour l'authentification

# Configurer le projet par defaut
gcloud config set project mon-projet-id

# Configurer la region par defaut
gcloud config set compute/region europe-west1

# Verifier la configuration
gcloud config list
```

### Azure CLI

```bash
# Installation
pip install azure-cli

# OU via l'installeur officiel
# https://learn.microsoft.com/cli/azure/install-azure-cli

# Connexion
az login
# Cela ouvre un navigateur pour l'authentification

# Configurer l'abonnement par defaut
az account set --subscription "Mon-Abonnement"

# Verifier
az account show
```

> [!warning] Securite des credentials
> **Ne commitez JAMAIS** vos cles d'acces dans un depot Git. Utilisez des variables d'environnement ou des fichiers de configuration locaux ignores par `.gitignore`. Les cles exposees publiquement sont detectees en quelques minutes par des bots malveillants.

```bash
# Bonnes pratiques pour les credentials

# 1. Variables d'environnement
export AWS_ACCESS_KEY_ID="votre-cle"
export AWS_SECRET_ACCESS_KEY="votre-secret"

# 2. Fichier de configuration (cree par aws configure)
# ~/.aws/credentials  (ne jamais commiter ce fichier)

# 3. Roles IAM (meilleure pratique en production)
# Pas de cles du tout, l'instance recoit les permissions automatiquement
```

---

## Concepts Cles du Cloud

### Elasticite

La capacite a **augmenter ou reduire** automatiquement les ressources en fonction de la demande.

```
  Charge utilisateur          Ressources allouees (avec elasticite)

  ^                           ^
  |     ___                   |     ___
  |    /   \                  |    /   \
  |   /     \      ===>      |   /     \
  |  /       \___             |  /       \___
  | /                         | /
  +------------>              +------------>
       Temps                       Temps

  Sans elasticite : vous payez pour la capacite maximale en permanence
  Avec elasticite : les ressources suivent la demande
```

### Scalabilite

| Type | Description | Exemple |
|---|---|---|
| **Verticale** (Scale Up) | Augmenter la puissance d'une machine (plus de CPU, RAM) | Passer de t3.medium a t3.xlarge |
| **Horizontale** (Scale Out) | Ajouter plus de machines | Passer de 1 a 5 instances derriere un load balancer |

```
  SCALABILITE VERTICALE          SCALABILITE HORIZONTALE

  +--------+                     +----+ +----+ +----+ +----+
  |        |                     |    | |    | |    | |    |
  |        |                     | VM | | VM | | VM | | VM |
  | GROS   |                     |    | |    | |    | |    |
  | SERVEUR|                     +----+ +----+ +----+ +----+
  |        |                            |
  |        |                     +------+------+
  +--------+                     | Load Balancer|
                                 +--------------+
```

> [!tip] Privilegiez la scalabilite horizontale
> La scalabilite verticale a des limites physiques (il y a un maximum de RAM dans un serveur). La scalabilite horizontale est theoriquement illimitee et offre une meilleure resilience : si une machine tombe, les autres continuent.

### Haute Disponibilite (High Availability)

La capacite d'un systeme a rester operationnel meme en cas de panne d'un composant. Se mesure en pourcentage de disponibilite :

| Disponibilite | Temps d'indisponibilite/an | Nom courant |
|---|---|---|
| 99% | 3.65 jours | "Deux neufs" |
| 99.9% | 8.77 heures | "Trois neufs" |
| 99.99% | 52.6 minutes | "Quatre neufs" |
| 99.999% | 5.26 minutes | "Cinq neufs" |

### Tolerance aux Pannes (Fault Tolerance)

La capacite a continuer de fonctionner **sans interruption** meme lorsqu'un composant tombe en panne. Plus strict que la haute disponibilite : il n'y a **aucune** interruption perceptible.

### Pay-as-you-go

Vous ne payez que ce que vous consommez. Pas de cout initial, pas d'engagement (en mode on-demand).

```
  Modele traditionnel :
  Cout = Prix_serveur + Maintenance + Electricite + Personnel (fixe)

  Modele cloud :
  Cout = Ressources_consommees x Prix_unitaire (variable)
```

---

## Principes Cloud-Native : Resume du 12-Factor App

Le **12-Factor App** est une methodologie pour construire des applications adaptees au cloud. Voici un resume des 12 facteurs :

| # | Facteur | Principe |
|---|---|---|
| 1 | **Codebase** | Un seul depot de code, plusieurs deploiements |
| 2 | **Dependencies** | Declarer et isoler explicitement les dependances |
| 3 | **Config** | Stocker la config dans l'environnement (variables d'env) |
| 4 | **Backing services** | Traiter les services externes comme des ressources attachables |
| 5 | **Build, release, run** | Separer strictement build, release et execution |
| 6 | **Processes** | Executer l'app comme des processus sans etat (stateless) |
| 7 | **Port binding** | Exporter les services via un port |
| 8 | **Concurrency** | Scalabilite via des processus (scale out) |
| 9 | **Disposability** | Demarrage rapide, arret gracieux |
| 10 | **Dev/prod parity** | Garder dev, staging et prod aussi similaires que possible |
| 11 | **Logs** | Traiter les logs comme des flux d'evenements |
| 12 | **Admin processes** | Executer les taches d'admin comme des processus ponctuels |

> [!info] Pourquoi le 12-Factor App ?
> Ces principes garantissent que votre application est **portable**, **scalable** et **maintenable** dans un environnement cloud. Si votre application les respecte, elle se deploiera facilement sur n'importe quel fournisseur cloud.

---

## Pratique : Creer un Compte et Configurer le CLI

### Etape 1 : Creer un compte Free Tier

Choisissez un fournisseur (AWS recommande pour debuter) :

1. Allez sur `https://aws.amazon.com/free/`
2. Cliquez sur "Create a Free Account"
3. Renseignez vos informations (une carte bancaire est requise mais ne sera pas debitee tant que vous restez dans les limites du free tier)
4. Activez **MFA** (Multi-Factor Authentication) immediatement

> [!warning] Securite du compte root
> Le compte que vous venez de creer est le compte **root** (administrateur supreme). Ne l'utilisez **jamais** pour les operations quotidiennes. Creez un utilisateur IAM avec les permissions necessaires et utilisez celui-ci.

### Etape 2 : Configurer les alertes de facturation

```
AWS Console > Billing > Budgets > Create Budget
  Type : Zero spend budget (alerte des le premier centime)
  OU : Monthly cost budget (seuil personnalise, ex: 5$)
  Notification : votre email
```

### Etape 3 : Creer un utilisateur IAM

```bash
# Via la console : IAM > Users > Add User
# OU via le CLI (si deja configure avec le compte root) :

aws iam create-user --user-name mon-utilisateur-dev

# Creer une cle d'acces pour le CLI
aws iam create-access-key --user-name mon-utilisateur-dev

# Attacher une politique de permissions
aws iam attach-user-policy \
  --user-name mon-utilisateur-dev \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
```

### Etape 4 : Installer et configurer le CLI

```bash
# Installer AWS CLI v2
# Sur Linux/macOS :
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Sur Windows : telecharger l'installeur MSI depuis
# https://aws.amazon.com/cli/

# Verifier l'installation
aws --version
# aws-cli/2.x.x Python/3.x.x ...

# Configurer avec les cles de l'utilisateur IAM
aws configure
# Entrez les cles de l'utilisateur IAM (PAS du compte root)

# Tester
aws s3 ls                  # Lister les buckets S3
aws ec2 describe-regions   # Lister les regions disponibles
```

### Etape 5 : Premieres commandes

```bash
# Lister les services disponibles
aws help

# Obtenir de l'aide sur un service specifique
aws ec2 help
aws s3 help

# Exemples de commandes courantes
aws s3 ls                          # Lister les buckets
aws s3 mb s3://mon-premier-bucket  # Creer un bucket
aws s3 cp fichier.txt s3://mon-premier-bucket/  # Upload un fichier
aws s3 ls s3://mon-premier-bucket/ # Lister le contenu du bucket
aws s3 rb s3://mon-premier-bucket --force  # Supprimer le bucket

# Equivalent GCP
gcloud storage ls
gcloud storage buckets create gs://mon-premier-bucket
gcloud storage cp fichier.txt gs://mon-premier-bucket/

# Equivalent Azure
az storage account list
az storage container create --name mon-conteneur --account-name moncompte
```

> [!example] Votre premier test cloud
> ```bash
> # Creez un bucket S3, uploadez un fichier, recuperez-le, puis nettoyez
> aws s3 mb s3://test-cloud-$(date +%s)
> echo "Hello Cloud!" > hello.txt
> aws s3 cp hello.txt s3://test-cloud-xxx/
> aws s3 cp s3://test-cloud-xxx/hello.txt downloaded.txt
> cat downloaded.txt   # "Hello Cloud!"
> aws s3 rb s3://test-cloud-xxx --force
> ```
> Remplacez `xxx` par le timestamp affiche lors de la creation.

---

## Carte Mentale ASCII

```
                           CLOUD COMPUTING
                                 |
         +-----------+-----------+-----------+-----------+
         |           |           |           |           |
     Modeles      Deploiement  Fournisseurs Concepts   Tarification
     de service      |           |           |           |
         |       +---+---+   +--+--+    +---+---+   +---+---+
     +---+---+   |   |   |  |  |  |    |   |   |   |   |   |
     |   |   | Pub Priv Hyb AWS GCP  Elast Scala HA  On  Res Spot
    IaaS PaaS      Multi  Azure      icite bilite    Dem erve
     SaaS                                             ande

  IaaS : Infra (VM, stockage, reseau)
  PaaS : Plateforme (runtime, middleware)
  SaaS : Logiciel pret a l'emploi
```

### Regles essentielles

> [!tip] Les 10 regles du Cloud debutant
> 1. **Activez MFA** sur votre compte des la creation
> 2. **Configurez des alertes de facturation** immediatement
> 3. **N'utilisez jamais le compte root** pour les operations courantes
> 4. **Ne commitez jamais** vos cles d'acces dans Git
> 5. **Choisissez la bonne region** (pres de vos utilisateurs)
> 6. **Deployez dans plusieurs AZ** pour la resilience
> 7. **Eteignez** les ressources non utilisees (elles coutent de l'argent)
> 8. **Taguez vos ressources** pour suivre les couts
> 9. **Privilegiez la scalabilite horizontale** a la verticale
> 10. **Apprenez un fournisseur en profondeur**, les concepts sont transferables

---

## Exercices

### Exercice 1 : Creer un compte Free Tier

1. Creez un compte AWS Free Tier (ou GCP/Azure)
2. Activez MFA sur le compte root
3. Creez un budget de 0$ avec notification par email
4. Creez un utilisateur IAM avec des permissions PowerUser
5. Installez et configurez le CLI avec les cles de cet utilisateur
6. Verifiez avec `aws sts get-caller-identity`

### Exercice 2 : Explorer les services via le CLI

1. Listez les regions disponibles avec `aws ec2 describe-regions`
2. Creez un bucket S3 avec un nom unique
3. Uploadez un fichier texte dans ce bucket
4. Telechargez-le dans un autre dossier
5. Supprimez le bucket et son contenu
6. Verifiez que tout a ete nettoye

### Exercice 3 : Tableau comparatif

Recherchez et completez un tableau comparatif des 3 fournisseurs pour les services suivants :
- Stockage de fichiers volumineux (type datalake)
- File d'attente de messages (messaging queue)
- Service de notification (push notifications)
- Service de transcription audio

### Exercice 4 : Calculer un cout

Utilisez le **AWS Pricing Calculator** (`calculator.aws`) pour estimer le cout mensuel d'une infrastructure comprenant :
- 2 instances EC2 t3.medium (24/7)
- 100 Go de stockage S3
- 1 base RDS db.t3.micro
- Transfert de donnees : 50 Go/mois sortant
Comparez le cout en `us-east-1` vs `eu-west-3`.

---

## Liens

- [[02 - Services Cloud Essentiels]] - Compute, stockage, bases de donnees et reseau
- [[01 - Docker]] - Conteneurs et deploiement
- [[04 - CI-CD avec GitHub Actions]] - Integration et deploiement continus
