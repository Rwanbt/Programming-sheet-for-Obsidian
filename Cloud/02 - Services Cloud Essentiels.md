# Services Cloud Essentiels

## Introduction

Cette note couvre les services fondamentaux que vous retrouverez chez tous les fournisseurs cloud : **Compute**, **Storage**, **Database**, **Networking** et **IAM**. Nous utiliserons principalement la terminologie AWS avec les equivalents GCP/Azure, car les concepts sont identiques.

> [!tip] Analogie
> Si le cloud est une **ville**, alors :
> - Les **services Compute** sont les **usines** (elles executent le travail)
> - Le **Storage** est l'**entrepot** (il conserve les marchandises)
> - Les **Databases** sont les **bibliotheques** (elles organisent et permettent de retrouver l'information)
> - Le **Networking** est le **reseau routier** (il connecte tout ensemble)
> - **IAM** est le **service de securite** (il controle qui entre ou et fait quoi)

---

## Compute : Machines Virtuelles

### EC2 / Compute Engine / Azure VMs

Une **instance** est une machine virtuelle (VM) que vous louez dans le cloud. Vous choisissez la puissance (CPU, RAM, stockage) et le systeme d'exploitation.

### Types d'instances

Les instances sont organisees en **familles** selon leur usage :

| Famille | Usage | Exemple AWS | vCPU | RAM |
|---|---|---|---|---|
| **General purpose** | Equilibre CPU/RAM | t3.medium | 2 | 4 Go |
| **Compute optimized** | Calcul intensif | c5.xlarge | 4 | 8 Go |
| **Memory optimized** | Bases de donnees, cache | r5.large | 2 | 16 Go |
| **Storage optimized** | I/O intensif | i3.large | 2 | 15.25 Go |
| **GPU** | ML, rendu graphique | p3.2xlarge | 8 | 61 Go |

> [!info] Nomenclature des instances AWS
> `t3.medium` se decompose ainsi :
> - `t` : famille (general purpose, burstable)
> - `3` : generation (plus recent = meilleur rapport prix/performance)
> - `medium` : taille (nano < micro < small < medium < large < xlarge < 2xlarge...)

### Lancer une instance EC2

```bash
# Via le CLI AWS

# 1. Trouver l'AMI (Amazon Machine Image) - image de l'OS
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
  --output text
# Retourne : ami-0abcdef1234567890

# 2. Creer une paire de cles SSH
aws ec2 create-key-pair \
  --key-name ma-cle \
  --query 'KeyMaterial' \
  --output text > ma-cle.pem

chmod 400 ma-cle.pem   # Restreindre les permissions

# 3. Lancer l'instance
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name ma-cle \
  --security-group-ids sg-xxxxxxxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=mon-serveur}]'

# Equivalent GCP
gcloud compute instances create mon-serveur \
  --machine-type e2-micro \
  --zone europe-west1-b \
  --image-family debian-11 \
  --image-project debian-cloud

# Equivalent Azure
az vm create \
  --resource-group mon-groupe \
  --name mon-serveur \
  --image UbuntuLTS \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys
```

### Se connecter en SSH

```bash
# Connexion SSH a une instance EC2
ssh -i ma-cle.pem ec2-user@<adresse-ip-publique>

# Pour une instance Ubuntu
ssh -i ma-cle.pem ubuntu@<adresse-ip-publique>

# GCP (gcloud gere les cles automatiquement)
gcloud compute ssh mon-serveur --zone europe-west1-b

# Azure
ssh azureuser@<adresse-ip-publique>
```

### Security Groups (Groupes de securite)

Un **Security Group** est un **firewall virtuel** qui controle le trafic entrant et sortant d'une instance.

```bash
# Creer un security group
aws ec2 create-security-group \
  --group-name mon-sg \
  --description "Acces SSH et HTTP"

# Autoriser SSH (port 22) depuis votre IP
aws ec2 authorize-security-group-ingress \
  --group-name mon-sg \
  --protocol tcp \
  --port 22 \
  --cidr $(curl -s ifconfig.me)/32

# Autoriser HTTP (port 80) depuis partout
aws ec2 authorize-security-group-ingress \
  --group-name mon-sg \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

> [!warning] Ne jamais ouvrir SSH (port 22) a 0.0.0.0/0
> Ouvrir SSH a tout Internet expose votre instance a des attaques par force brute. Restreignez toujours l'acces SSH a votre adresse IP (`/32`) ou utilisez un bastion host.

### User Data (scripts de demarrage)

Un script execute automatiquement au **premier demarrage** de l'instance. Utile pour automatiser l'installation de dependances et le deploiement :

```bash
#!/bin/bash
# startup-script.sh - passe via --user-data file://startup-script.sh
yum update -y && yum install -y python3 python3-pip
pip3 install flask
mkdir -p /opt/app
cat > /opt/app/app.py << 'SCRIPT'
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
    return "Hello depuis le Cloud!"
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
SCRIPT
python3 /opt/app/app.py &
```

---

## Compute : Serverless (Functions as a Service)

### Lambda / Cloud Functions / Azure Functions

Le **serverless** vous permet d'executer du code **sans gerer de serveur**. Vous deployez une fonction, le fournisseur s'occupe de tout le reste (provisionning, scaling, disponibilite).

> [!tip] Analogie
> Imaginez un **taxi** vs une **voiture personnelle** :
> - **VM (EC2)** = voiture personnelle : vous la possedez, payez l'assurance et le parking meme quand elle ne roule pas
> - **Serverless (Lambda)** = taxi : vous payez uniquement quand vous roulez, pas de parking, pas d'entretien

### Concept

```
  Evenement (trigger)    -->    Fonction Lambda    -->    Resultat
                                    |
                         Le cloud gere tout :
                         - Provisionning
                         - Scaling (0 a des milliers)
                         - Haute disponibilite
                         - Facturation a la milliseconde
```

### Triggers (declencheurs)

| Trigger | Cas d'usage |
|---|---|
| **HTTP (API Gateway)** | API REST, webhooks |
| **S3 Event** | Traiter un fichier uploade (resize image, parse CSV) |
| **Schedule (cron)** | Tache planifiee (nettoyage, rapport) |
| **Queue (SQS/Pub-Sub)** | Traitement asynchrone de messages |
| **Database Event** | Reagir a une modification en base |

### Exemple : Lambda en Python

```python
# lambda_function.py

import json

def lambda_handler(event, context):
    """
    Point d'entree de la fonction Lambda.
    
    Args:
        event: Les donnees de l'evenement declencheur
        context: Informations sur l'execution (timeout, memoire, etc.)
    
    Returns:
        Reponse HTTP (si derriere API Gateway)
    """
    # Extraire le nom depuis les parametres de la requete
    name = event.get('queryStringParameters', {}).get('name', 'World')
    
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': json.dumps({
            'message': f'Hello, {name}!',
            'event': event
        })
    }
```

### Tarification serverless

```
  Cout Lambda = Nombre de requetes x Duree x Memoire allouee

  Free tier : 1 million de requetes/mois + 400 000 Go-secondes
  
  Exemple :
    10 millions de requetes/mois
    Duree moyenne : 200ms
    Memoire : 256 Mo
    
    Cout requetes : (10M - 1M gratuit) x 0.20$/million = 1.80$
    Cout compute  : 9M x 0.2s x 0.25Go x 0.0000166667$/Go-s = 7.50$
    Total : ~9.30$/mois
    
  Compare a une instance EC2 t3.micro 24/7 : ~8.50$/mois
  (mais la Lambda scale a 0 quand il n'y a pas de trafic)
```

### Cold Start (demarrage a froid)

> [!warning] Le cold start
> Quand une fonction Lambda n'a pas ete appelee depuis un moment, le cloud doit provisionner un nouvel environnement d'execution. Ce **demarrage a froid** ajoute de la latence (100ms a plusieurs secondes selon le langage et la taille du package).
>
> **Mitigation** :
> - Utiliser des langages legers (Python, Node.js) plutot que Java
> - Garder les packages petits
> - Utiliser **Provisioned Concurrency** (AWS) pour maintenir des instances chaudes
> - Eventuellement, faire un "ping" periodique (warming)

---

## Storage : Stockage Objet

### S3 / Cloud Storage / Blob Storage

Le **stockage objet** est le service le plus utilise du cloud. Il stocke des fichiers (appeles **objets**) dans des **buckets** (conteneurs).

Un **bucket** est un conteneur d'objets. Chaque **objet** a une cle (chemin, ex: `images/logo.png`), des donnees (le fichier), des metadonnees et des permissions.

### Operations de base

```bash
# Creer un bucket
aws s3 mb s3://mon-bucket-unique-12345

# Uploader un fichier
aws s3 cp monimage.png s3://mon-bucket-unique-12345/images/

# Uploader un dossier entier
aws s3 sync ./mon-site/ s3://mon-bucket-unique-12345/

# Lister le contenu
aws s3 ls s3://mon-bucket-unique-12345/
aws s3 ls s3://mon-bucket-unique-12345/images/

# Telecharger un fichier
aws s3 cp s3://mon-bucket-unique-12345/images/monimage.png ./

# Supprimer un objet
aws s3 rm s3://mon-bucket-unique-12345/images/monimage.png

# Supprimer un bucket (doit etre vide, ou utiliser --force)
aws s3 rb s3://mon-bucket-unique-12345 --force
```

### Classes de stockage

| Classe (AWS) | Usage | Disponibilite | Cout stockage | Cout acces |
|---|---|---|---|---|
| **S3 Standard** | Acces frequent | 99.99% | $$$ | $ |
| **S3 Infrequent Access** | Acces rare | 99.9% | $$ | $$ |
| **S3 Glacier** | Archivage | 99.99% | $ | $$$ (delai) |
| **S3 Glacier Deep Archive** | Archivage long terme | 99.99% | ¢ | $$$$ (12h delai) |

### Lifecycle Policies (politiques de cycle de vie)

Automatiser le deplacement des objets entre les classes de stockage :

```
  Cycle de vie d'un objet (configure via une Lifecycle Policy JSON) :
  
  Jour 0        Jour 30           Jour 90          Jour 365       Jour 730
    |              |                 |                 |               |
    v              v                 v                 v               v
  Standard --> Infrequent --> Glacier --> Deep Archive --> Supprime
  (acces      Access         (archivage)  (archivage       
   frequent)  (acces rare)                long terme)
```

### Hebergement de site statique

S3 peut heberger un site web statique (HTML, CSS, JS) sans serveur :

```bash
# 1. Creer le bucket et activer l'hebergement
aws s3 mb s3://mon-site-web.example.com
aws s3 website s3://mon-site-web.example.com \
  --index-document index.html --error-document error.html

# 2. Rendre le bucket public (politique d'acces)
aws s3api put-bucket-policy --bucket mon-site-web.example.com \
  --policy '{"Version":"2012-10-17","Statement":[{"Effect":"Allow",
  "Principal":"*","Action":"s3:GetObject",
  "Resource":"arn:aws:s3:::mon-site-web.example.com/*"}]}'

# 3. Uploader le site
aws s3 sync ./site/ s3://mon-site-web.example.com/
# URL : http://mon-site-web.example.com.s3-website.eu-west-3.amazonaws.com
```

### Pre-signed URLs (URLs presignees)

Generer une URL temporaire pour acceder a un objet prive :

```bash
# Generer une URL valide 1 heure (3600 secondes)
aws s3 presign s3://mon-bucket/fichier-confidentiel.pdf \
  --expires-in 3600
# https://mon-bucket.s3.amazonaws.com/fichier-confidentiel.pdf?X-Amz-...
```

```python
# En Python avec boto3
import boto3

s3_client = boto3.client('s3')

url = s3_client.generate_presigned_url(
    'get_object',
    Params={
        'Bucket': 'mon-bucket',
        'Key': 'fichier-confidentiel.pdf'
    },
    ExpiresIn=3600  # 1 heure
)
print(url)
```

> [!example] Cas d'usage des pre-signed URLs
> - Permettre a un utilisateur de telecharger un fichier prive sans lui donner un acces permanent
> - Generer un lien de telechargement temporaire dans un email
> - Permettre un upload direct vers S3 depuis un navigateur (pre-signed POST)

---

## Storage : Stockage Bloc et Fichier

### Stockage Bloc : EBS / Persistent Disk / Managed Disks

Le **stockage bloc** est l'equivalent d'un **disque dur** attache a une VM. Necessaire pour le systeme de fichiers de l'OS et les applications qui ont besoin d'un acces disque rapide.

| Caracteristique | Stockage Objet (S3) | Stockage Bloc (EBS) |
|---|---|---|
| **Acces** | Via HTTP/API | Monte comme un disque |
| **Performance** | Bonne pour gros fichiers | Haute performance I/O |
| **Attache a** | Aucune instance | Une instance specifique |
| **Cas d'usage** | Fichiers, backups, media | OS, bases de donnees |

### Stockage Fichier : EFS / Filestore / Azure Files

Le **stockage fichier** fournit un systeme de fichiers partage (NFS) accessible par **plusieurs instances** simultanement.

```
  Stockage Bloc (EBS)          Stockage Fichier (EFS)
  
  +------+    +------+         +------+    +------+
  | VM A |    | VM B |         | VM A |    | VM B |
  +--+---+    +--+---+         +--+---+    +--+---+
     |           |                 \          /
  +--+---+    +--+---+          +--+--------+--+
  | EBS  |    | EBS  |          |     EFS      |
  | (A)  |    | (B)  |          | (partage)    |
  +------+    +------+          +--------------+
  
  Chaque EBS est              Un EFS est monte par
  attache a UNE VM            PLUSIEURS VMs
```

---

## Database : Services de Bases de Donnees Gerees

### RDS / Cloud SQL / Azure SQL Database

Un service de **base de donnees relationnelle geree**. Le fournisseur s'occupe de : l'installation, les mises a jour, les backups, la replication, le failover.

```bash
# Creer une instance RDS PostgreSQL
aws rds create-db-instance --db-instance-identifier ma-base \
  --db-instance-class db.t3.micro --engine postgres --engine-version 15 \
  --master-username admin --master-user-password MonMotDePasse123! \
  --allocated-storage 20

# Equivalent GCP
gcloud sql instances create ma-base \
  --database-version POSTGRES_15 --tier db-f1-micro --region europe-west1

# Obtenir l'endpoint de connexion
aws rds describe-db-instances --db-instance-identifier ma-base \
  --query 'DBInstances[0].Endpoint.Address'
```

> [!info] Pourquoi utiliser RDS plutot qu'installer PostgreSQL sur une VM ?
> - **Backups automatiques** (retention configurable, point-in-time recovery)
> - **Mises a jour** automatiques du moteur
> - **Multi-AZ** : basculement automatique en cas de panne
> - **Read Replicas** : repliques en lecture pour distribuer la charge
> - **Monitoring** integre
>
> Vous payez un peu plus, mais vous economisez des heures d'administration.

### DynamoDB / Firestore / Cosmos DB

Les bases **NoSQL gerees** pour les applications necessitant une latence tres faible et un scaling massif.

```
  Base Relationnelle (RDS)        Base NoSQL (DynamoDB)
  
  +----+--------+--------+       +----------------------------------+
  | id | nom    | email  |       | {                                |
  +----+--------+--------+       |   "id": "u1",                   |
  | 1  | Alice  | a@b.fr |       |   "nom": "Alice",               |
  | 2  | Bob    | b@c.fr |       |   "email": "a@b.fr",            |
  +----+--------+--------+       |   "preferences": {              |
                                  |     "theme": "dark",            |
  Schema fixe                    |     "lang": "fr"                |
  Relations (JOIN)               |   }                              |
  Transactions ACID              | }                                |
                                  +----------------------------------+
                                  
                                  Schema flexible
                                  Pas de JOIN (denormalise)
                                  Latence < 10ms a n'importe quelle echelle
```

```python
# Exemple DynamoDB avec boto3
import boto3
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('utilisateurs')

table.put_item(Item={
    'user_id': 'u1', 'nom': 'Alice', 'email': 'alice@example.com',
    'preferences': {'theme': 'dark', 'lang': 'fr'}
})

response = table.get_item(Key={'user_id': 'u1'})
print(response['Item']['nom'])  # Alice
```

### ElastiCache / Memorystore / Azure Cache

Service de cache en memoire gere, compatible **Redis** ou Memcached. Utilise pour accelerer les lectures frequentes.

```
  Sans cache :
  Client --> Application --> Base de donnees (50ms)
  
  Avec cache Redis :
  Client --> Application --> Redis (1ms)   [cache hit]
                         --> Base de donnees (50ms)  [cache miss]
                         --> Redis (ecriture en cache)
```

---

## Networking : VPC (Virtual Private Cloud)

### Concept

Un **VPC** est votre **reseau prive** dans le cloud. C'est un espace isole ou vous deployez vos ressources, avec un controle total sur l'adressage IP, les sous-reseaux, les tables de routage et les passerelles.

> [!tip] Analogie
> Un VPC est comme un **immeuble de bureaux securise** :
> - Le **VPC** = l'immeuble entier (votre espace prive)
> - Les **sous-reseaux** = les etages (public = rez-de-chaussee ouvert aux visiteurs, prive = etages a acces restreint)
> - L'**Internet Gateway** = la porte d'entree principale
> - Le **NAT Gateway** = un coursier : les gens a l'interieur peuvent envoyer des colis dehors, mais personne ne peut entrer par cette voie
> - Les **Security Groups** = les badges d'acces individuels
> - Les **NACLs** = les regles de l'immeuble (qui peut entrer a quel etage)

### Architecture VPC

```
  +---------------------------------------------------------------+
  |  VPC : 10.0.0.0/16                                           |
  |                                                               |
  |         +------+------+                                       |
  |         | Internet GW |  <-- Porte d'entree vers Internet    |
  |         +------+------+                                       |
  |                |                                              |
  |         +------+------+                                       |
  |         | Load Balancer|                                      |
  |         +------+------+                                       |
  |           /         \                                         |
  |  +----------------+  +----------------+                       |
  |  | PUBLIC 10.0.1.0|  | PUBLIC 10.0.2.0|                      |
  |  | AZ: eu-west-3a |  | AZ: eu-west-3b |                      |
  |  | +----+ +----+  |  | +----+         |                      |
  |  | |Web1| |Web2|  |  | |Web3|         |                      |
  |  | +----+ +----+  |  | +----+         |                      |
  |  +----------------+  +----------------+                       |
  |                                                               |
  |  +----------------+  +----------------+                       |
  |  | PRIVE 10.0.3.0 |  | PRIVE 10.0.4.0 |                     |
  |  | AZ: eu-west-3a |  | AZ: eu-west-3b |                      |
  |  | +----+ +----+  |  | +----+ +----+  |                      |
  |  | |API1| |API2|  |  | |API3| | DB |  |                      |
  |  | +----+ +----+  |  | +----+ |(RDS)| |                      |
  |  | +----------+   |  |        +----+  |                      |
  |  | |NAT Gateway|  |  |                |                      |
  |  | +----------+   |  |                |                      |
  |  +----------------+  +----------------+                       |
  +---------------------------------------------------------------+
```

### Sous-reseaux publics vs prives

| Caracteristique | Sous-reseau PUBLIC | Sous-reseau PRIVE |
|---|---|---|
| **Acces Internet entrant** | Oui (via Internet Gateway) | Non |
| **Acces Internet sortant** | Oui (direct) | Oui (via NAT Gateway) |
| **IP publique** | Oui (automatique ou Elastic IP) | Non |
| **Usage** | Load balancers, bastions | Serveurs d'application, bases de donnees |

### Security Groups vs NACLs

| Caracteristique | Security Group | NACL |
|---|---|---|
| **Niveau** | Instance (interface reseau) | Sous-reseau |
| **Type** | Stateful | Stateless |
| **Regles** | Autoriser uniquement | Autoriser ET refuser |
| **Evaluation** | Toutes les regles | Regles evaluees dans l'ordre |
| **Par defaut** | Tout refuse (entrant) | Tout autorise |

> [!info] Stateful vs Stateless
> - **Stateful** (Security Group) : si vous autorisez le trafic entrant sur le port 80, la reponse sortante est **automatiquement** autorisee. Pas besoin de creer une regle sortante.
> - **Stateless** (NACL) : vous devez creer des regles **pour les deux directions**. Si vous autorisez l'entrant sur le port 80, vous devez aussi autoriser le sortant sur les ports ephemeres (1024-65535).

### Route Tables (tables de routage)

```
  Route Table du sous-reseau PUBLIC :
  +------------------+------------------+
  | Destination      | Cible            |
  +------------------+------------------+
  | 10.0.0.0/16      | local            |  <-- Trafic interne au VPC
  | 0.0.0.0/0        | igw-xxxxxxxx     |  <-- Tout le reste → Internet Gateway
  +------------------+------------------+

  Route Table du sous-reseau PRIVE :
  +------------------+------------------+
  | Destination      | Cible            |
  +------------------+------------------+
  | 10.0.0.0/16      | local            |  <-- Trafic interne au VPC
  | 0.0.0.0/0        | nat-xxxxxxxx     |  <-- Tout le reste → NAT Gateway
  +------------------+------------------+
```

---

## IAM : Identity and Access Management

### Concept

**IAM** controle **qui** (authentification) peut faire **quoi** (autorisation) sur **quelles ressources**.

```
  +-------+     Authentification      +-------+     Autorisation      +----------+
  | Qui ? | -----------------------> | IAM   | --------------------> | Ressource|
  | User  |  (login, cle, role)      |       |  (politique/policy)  | (EC2, S3)|
  +-------+                          +-------+                      +----------+
```

### Les composants IAM

```
  IAM
   |
   +-- Utilisateurs (Users)
   |     |
   |     +-- Cles d'acces (pour le CLI)
   |     +-- Mot de passe (pour la console)
   |     +-- MFA (second facteur)
   |
   +-- Groupes (Groups)
   |     |
   |     +-- Developpeurs
   |     +-- Administrateurs
   |     +-- Lecture-seule
   |
   +-- Roles (Roles)
   |     |
   |     +-- Role EC2 (permissions pour une instance)
   |     +-- Role Lambda (permissions pour une fonction)
   |     +-- Role Cross-Account (acces entre comptes)
   |
   +-- Politiques (Policies)
         |
         +-- AWS Managed (pre-definies par AWS)
         +-- Customer Managed (creees par vous)
         +-- Inline (attachees directement a un user/role)
```

### Exemple de politique IAM

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::mon-bucket/*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::mon-bucket/*"
    }
  ]
}
```

> [!warning] Principe du moindre privilege (Least Privilege)
> Ne donnez **jamais** plus de permissions que necessaire. Commencez avec zero permission et ajoutez uniquement ce qui est requis. Une politique trop permissive (ex: `"Action": "*"`, `"Resource": "*"`) est une faille de securite majeure.

### Cles d'acces vs Roles

| Methode | Usage | Securite |
|---|---|---|
| **Cles d'acces** | CLI depuis votre poste | Moyen (cles statiques, peuvent fuiter) |
| **Roles IAM** | Services cloud (EC2, Lambda) | Eleve (credentials temporaires, rotation auto) |

```bash
# MAUVAIS : stocker des cles dans le code
AWS_ACCESS_KEY_ID="AKIA..."      # NE FAITES JAMAIS CA
AWS_SECRET_ACCESS_KEY="wJalr..."  # DANS DU CODE

# BON : attacher un role IAM a l'instance EC2
# L'instance recoit automatiquement des credentials temporaires
# Aucune cle a gerer, aucun risque de fuite
aws ec2 associate-iam-instance-profile \
  --instance-id i-1234567890abcdef0 \
  --iam-instance-profile Name=mon-role-ec2
```

### MFA (Multi-Factor Authentication)

```
  Authentification standard :      Authentification MFA :
  
  +----------+                     +----------+
  | Password | ---> Acces          | Password |
  +----------+                     +-----+----+
                                         |
                                   +-----+----+
                                   | Code MFA | ---> Acces
                                   | (6 digits)|
                                   +----------+
                                   
  1 facteur = risque               2 facteurs = securise
```

> [!tip] Activez MFA PARTOUT
> MFA devrait etre active sur :
> 1. Le compte root (obligatoire)
> 2. Tous les utilisateurs IAM avec acces a la console
> 3. Les operations sensibles (suppression de ressources critiques)

---

## Pratique : Deployer une Application Flask sur EC2

### Objectif

Deployer une API Flask accessible depuis Internet sur une instance EC2.

### Etape 1 : Creer l'infrastructure

```bash
# 1. Security group : autoriser SSH (votre IP) et port 5000 (Flask)
aws ec2 create-security-group --group-name flask-app-sg --description "Flask App"
aws ec2 authorize-security-group-ingress --group-id sg-xxx \
  --protocol tcp --port 22 --cidr $(curl -s ifconfig.me)/32
aws ec2 authorize-security-group-ingress --group-id sg-xxx \
  --protocol tcp --port 5000 --cidr 0.0.0.0/0

# 2. Cle SSH
aws ec2 create-key-pair --key-name flask-key \
  --query 'KeyMaterial' --output text > flask-key.pem
chmod 400 flask-key.pem

# 3. Lancer l'instance (t3.micro = free tier)
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 --instance-type t3.micro \
  --key-name flask-key --security-group-ids sg-xxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=flask-app}]'

# 4. Recuperer l'IP publique
aws ec2 describe-instances --filters "Name=tag:Name,Values=flask-app" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' --output text
```

### Etape 2 : Deployer l'application

```bash
# 1. Se connecter a l'instance
ssh -i flask-key.pem ec2-user@<IP_PUBLIQUE>

# 2. Installer Python et configurer l'application
sudo yum update -y && sudo yum install -y python3 python3-pip
mkdir -p ~/flask-app && cd ~/flask-app

# 3. Creer app.py (API Flask avec 3 endpoints : /, /tasks, /health)
cat > app.py << 'EOF'
from flask import Flask, jsonify
app = Flask(__name__)
tasks = [
    {"id": 1, "title": "Apprendre le Cloud", "done": False},
    {"id": 2, "title": "Deployer sur EC2", "done": False},
]
@app.route('/')
def index():
    return jsonify({"message": "API de taches - Cloud Demo"})
@app.route('/tasks')
def get_tasks():
    return jsonify({"tasks": tasks})
@app.route('/health')
def health():
    return jsonify({"status": "healthy"})
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# 4. Installer et lancer avec gunicorn
pip3 install flask gunicorn
gunicorn --bind 0.0.0.0:5000 app:app --daemon

# 5. Tester localement puis depuis votre navigateur
curl http://localhost:5000/health    # {"status": "healthy"}
# Navigateur : http://<IP_PUBLIQUE>:5000/tasks
```

### Etape 3 : Nettoyer les ressources

```bash
# IMPORTANT : nettoyez pour eviter les couts !

# Terminer l'instance
aws ec2 terminate-instances --instance-ids i-xxxxxxxxxxxxxxxxx

# Supprimer le security group (apres la terminaison de l'instance)
aws ec2 delete-security-group --group-id sg-xxxxxxxx

# Supprimer la paire de cles
aws ec2 delete-key-pair --key-name flask-key
rm flask-key.pem
```

> [!warning] Nettoyez TOUJOURS vos ressources
> Une instance EC2 qui tourne en continu, meme une t3.micro, finit par couter de l'argent apres le free tier. Prenez l'habitude de tout nettoyer apres chaque exercice.

---

## Carte Mentale ASCII

```
                       SERVICES CLOUD ESSENTIELS
                                |
         +----------+-----------+-----------+-----------+
         |          |           |           |           |
      COMPUTE    STORAGE    DATABASE    NETWORK      IAM
         |          |           |           |           |
     +---+---+  +---+---+  +---+---+     VPC       +---+---+
     |       |  |   |   |  |   |   |      |        |   |   |
    VM    Lambda Obj Bloc Fich RDS NoSQL  +--+--+  User Grp Role
     |       |  S3  EBS  EFS  |  Dynamo  |  |  |      |
   Types  Triggers  |      Cloud  Firest Sub IGW  Policies
   AMI    Cold     Classes  SQL         Pub NAT  Least
   SSH    Start    Lifecycle             Priv SG   Privilege
   SG              PreSign              NACL Routes MFA
   UserData
```

---

## Exercices

### Exercice 1 : Deployer et exposer une API

1. Creez une instance EC2 (ou Compute Engine) avec le free tier
2. Installez Python et deployer une API FastAPI avec 3 endpoints
3. Configurez le security group pour autoriser le trafic HTTP
4. Testez l'API depuis votre navigateur et avec `curl`
5. Nettoyez toutes les ressources

### Exercice 2 : Stockage S3

1. Creez un bucket S3
2. Uploadez 5 fichiers de tailles differentes
3. Creez une politique de lifecycle qui deplace les fichiers vers Infrequent Access apres 30 jours
4. Generez une pre-signed URL pour un des fichiers (validite 5 minutes)
5. Activez l'hebergement de site statique et uploadez une page HTML
6. Supprimez le bucket

### Exercice 3 : Architecture VPC

Dessinez (sur papier ou en ASCII) une architecture VPC comprenant :
- 2 sous-reseaux publics (dans 2 AZ differentes) avec des serveurs web
- 2 sous-reseaux prives (dans 2 AZ) avec des serveurs d'application
- 1 base de donnees RDS en Multi-AZ dans les sous-reseaux prives
- Un Load Balancer devant les serveurs web
- Un NAT Gateway pour les sous-reseaux prives
Indiquez les CIDR, les security groups et les routes.

### Exercice 4 : Politique IAM

Ecrivez une politique IAM (en JSON) qui :
- Autorise la lecture et l'ecriture dans un bucket S3 specifique
- Autorise le lancement et l'arret d'instances EC2 de type t3.micro uniquement
- Refuse toute suppression de ressource
- Respecte le principe du moindre privilege

---

## Liens

- [[01 - Introduction au Cloud]] - Concepts fondamentaux du cloud computing
- [[03 - Deploiement Cloud et Conteneurs]] - Deploiement avec conteneurs et orchestration
- [[01 - Fondamentaux Reseaux]] - Bases du reseau (TCP/IP, DNS, HTTP)
- [[01 - Docker]] - Conteneurs Docker
- [[09 - APIs REST avec FastAPI]] - Construction d'APIs avec FastAPI
