# 06 - Cloud Security AWS

> [!info] Objectifs
> Maîtriser la sécurité AWS : modèle de responsabilité partagée, IAM, réseau VPC, chiffrement KMS, audit CloudTrail, détection des menaces avec GuardDuty, et réponse aux incidents dans AWS.

## 1. Shared Responsibility Model

### La frontière de responsabilité AWS

```
┌─────────────────────────────────────────────────────┐
│          RESPONSABILITÉ DU CLIENT                    │
│  - Données (chiffrement au repos et en transit)      │
│  - Applications et code                              │
│  - Identity and Access Management (IAM)              │
│  - Configuration des services (Security Groups, etc.)│
│  - Système d'exploitation des EC2 (patches OS)       │
│  - Network configuration (VPC, sous-réseaux)         │
├─────────────────────────────────────────────────────┤
│          RESPONSABILITÉ AWS                          │
│  - Hyperviseur et infrastructure physique            │
│  - Réseau physique des data centers                  │
│  - Hardware (serveurs, stockage, réseau)              │
│  - Sécurité physique des data centers                │
│  - Infrastructure des services managés               │
│    (ex: la gestion du cluster RDS sous-jacent)        │
└─────────────────────────────────────────────────────┘
```

**Nuance importante** pour les services managés :

| Service | OS patching | Runtime | Votre responsabilité |
|---------|------------|---------|---------------------|
| EC2 | Client | Client | Tout le stack applicatif |
| ECS (Fargate) | AWS | AWS | Code, configs, secrets |
| RDS | AWS | AWS | Données, schéma, auth utilisateurs DB |
| Lambda | AWS | AWS | Code de la fonction |
| S3 | AWS | AWS | Politiques de bucket, chiffrement, ACLs |

---

## 2. IAM — Identity and Access Management

### Hiérarchie des identités

```
Organization (AWS Organizations)
├── Management Account
│   └── Service Control Policies (SCPs) → appliquées à tous les comptes
├── Production Account
│   ├── IAM Users (alice, bob)  ← Entités humaines
│   ├── IAM Groups (developers, admins)
│   ├── IAM Roles (ec2-role, lambda-role, ci-role)
│   └── IAM Policies (permissions)
└── Development Account
```

### Format ARN

```
arn:partition:service:region:account-id:resource-type/resource-id

Exemples :
arn:aws:iam::123456789:user/alice
arn:aws:iam::123456789:role/lambda-execution-role
arn:aws:s3:::my-bucket/data/*                    # S3 n'a pas de région ni compte dans l'ARN
arn:aws:ec2:eu-west-1:123456789:instance/i-abc123
arn:aws:kms:us-east-1:123456789:key/mrk-xxx
```

### Politiques IAM

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnSpecificBucket",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket",
        "s3:GetObjectVersion"
      ],
      "Resource": [
        "arn:aws:s3:::my-data-bucket",
        "arn:aws:s3:::my-data-bucket/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "eu-west-1"
        },
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    },
    {
      "Sid": "DenyDeleteS3",
      "Effect": "Deny",
      "Action": [
        "s3:DeleteObject",
        "s3:DeleteBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

### Types de politiques

| Type | Attachée à | Scope |
|------|-----------|-------|
| **Identity-based** | User, Group, Role | Définit ce que l'identité PEUT faire |
| **Resource-based** | Ressource (S3 bucket policy, KMS key policy) | Définit QUI peut accéder à la ressource |
| **SCP** (Service Control Policy) | OU / Account AWS Organizations | Limite maximale pour tout le compte |
| **Session policy** | Assume-role avec restriction supplémentaire | Réduit les perms d'un role au moment de l'assumer |
| **Permissions boundary** | User ou Role | Limite maximale des perms accordables |

### Logique d'évaluation des politiques

```
Requête API AWS
       │
       ▼
1. Existe-t-il un Deny explicite (dans N'IMPORTE quelle policy) ?
       │ OUI → DENY
       │ NON → continuer
       ▼
2. Existe-t-il un SCP dans l'Organization qui bloque ?
       │ OUI → DENY
       │ NON → continuer
       ▼
3. Existe-t-il un Allow explicite correspondant ?
       │ OUI → ALLOW
       │ NON → DENY implicite (tout ce qui n'est pas explicitement autorisé est refusé)
```

> [!warning] Deny explicite prioritaire absolu
> Un `"Effect": "Deny"` dans n'importe quelle politique (SCP, resource-based, identity-based) écrase TOUS les Allow. Il n'existe pas de "super-Allow" pour contourner un Deny explicite.

### AssumeRole — Cross-Account Access

```json
// Trust Policy du role (qui peut assumer ce role ?)
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::111111111111:root"  // Compte source
    },
    "Action": "sts:AssumeRole",
    "Condition": {
      "StringEquals": {"sts:ExternalId": "unique-secret-123"},  // Protection contre confused deputy
      "Bool": {"aws:MultiFactorAuthPresent": "true"}
    }
  }]
}
```

```bash
# Assumer un role cross-account
aws sts assume-role \
    --role-arn arn:aws:iam::222222222:role/ReadOnlyAccess \
    --role-session-name audit-session \
    --external-id unique-secret-123

# Utiliser les credentials temporaires
export AWS_ACCESS_KEY_ID=ASIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
```

### iam:PassRole — Danger critique

`iam:PassRole` permet à un utilisateur de donner à un service AWS le droit d'assumer un rôle. Si un utilisateur a `iam:PassRole` + `ec2:RunInstances`, il peut lancer une EC2 avec un rôle Admin et escalader ses privilèges.

```json
// Limiter PassRole à des roles spécifiques
{
  "Effect": "Allow",
  "Action": "iam:PassRole",
  "Resource": "arn:aws:iam::123456:role/ec2-limited-*",
  "Condition": {
    "StringEquals": {"iam:PassedToService": "ec2.amazonaws.com"}
  }
}
```

---

## 3. Security Groups et NACLs

### Security Groups — Pare-feu Stateful

```
Security Group (attaché à une ENI/instance) :
- Stateful : la réponse est automatiquement autorisée (pas besoin de règle de retour)
- Seulement des Allow (pas de Deny explicite possible)
- Par défaut : tout bloqué en entrée, tout autorisé en sortie

Exemple de Security Group pour un web server :
Inbound :
  Type: HTTP    Protocol: TCP   Port: 80    Source: 0.0.0.0/0
  Type: HTTPS   Protocol: TCP   Port: 443   Source: 0.0.0.0/0
  Type: SSH     Protocol: TCP   Port: 22    Source: 10.0.0.0/8 (réseau interne seulement)

Outbound :
  All traffic : 0.0.0.0/0 (autoriser les réponses et les appels externes)
```

**Référencer un autre Security Group** (la bonne pratique) :

```bash
# Au lieu de mettre l'IP du serveur DB, référencer son Security Group
aws ec2 authorize-security-group-ingress \
    --group-id sg-database-id \
    --protocol tcp \
    --port 5432 \
    --source-group sg-webapp-id  # Tout ce qui est dans sg-webapp peut accéder
```

### NACLs — Pare-feu Stateless (niveau sous-réseau)

```
NACL (Network Access Control List) :
- Stateless : les règles aller ET retour doivent être définies
- Allow ET Deny possibles
- Appliqué au sous-réseau (pas à l'instance)
- Évaluées dans l'ordre croissant des numéros de règle (la première qui match gagne)

Règles recommandées pour un subnet public :
Inbound :
  100  ALLOW  TCP   0.0.0.0/0   80     (HTTP)
  110  ALLOW  TCP   0.0.0.0/0   443    (HTTPS)
  120  ALLOW  TCP   10.0.0.0/8  22     (SSH depuis le réseau interne)
  130  ALLOW  TCP   0.0.0.0/0   1024-65535  (ports éphémères pour les réponses !)
  *    DENY   ALL   0.0.0.0/0   ALL

Outbound :
  100  ALLOW  ALL   0.0.0.0/0   ALL
  *    DENY   ALL   0.0.0.0/0   ALL
```

> [!tip] SG vs NACL
> **Security Groups** en première ligne (toujours). **NACLs** en deuxième couche de défense (à configurer en production pour ajouter des Deny explicites que les SGs ne peuvent pas faire, ex: bloquer une IP malveillante au niveau réseau).

---

## 4. VPC Security — Architecture réseau

### Architecture VPC sécurisée

```
VPC 10.0.0.0/16
├── Public Subnet 10.0.1.0/24 (eu-west-1a)
│   ├── Internet Gateway (IGW)   ← Route vers internet
│   ├── NAT Gateway              ← Permet aux private subnets d'accéder à internet
│   └── Bastion Host / Jump Server
│
├── Private Subnet 10.0.2.0/24 (eu-west-1a)
│   ├── Application Servers (EC2, ECS)
│   └── Route → NAT Gateway (pour les updates OS, appels API AWS)
│
└── Private Subnet 10.0.3.0/24 (eu-west-1b)
    └── Database Servers (RDS, ElastiCache)
        Route → NAT Gateway seulement si nécessaire
```

### VPC Endpoints — Éviter le trafic public

```bash
# Sans VPC Endpoint : EC2 → Internet (via IGW ou NAT) → S3 API
# Avec VPC Endpoint : EC2 → VPC Endpoint → S3 (réseau AWS privé, pas d'internet)

# Créer un Gateway Endpoint pour S3 (gratuit)
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-xxx \
    --service-name com.amazonaws.eu-west-1.s3 \
    --route-table-ids rtb-xxx

# Interface Endpoint pour DynamoDB, SQS, SNS, SSM (coût par heure)
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-xxx \
    --vpc-endpoint-type Interface \
    --service-name com.amazonaws.eu-west-1.secretsmanager \
    --subnet-ids subnet-xxx \
    --security-group-ids sg-xxx
```

**VPC Peering** : connexion privée entre deux VPCs (même compte ou cross-account, même région ou cross-region). Pas de transit (si A↔B et B↔C, A ne peut pas joindre C via B).

**Transit Gateway** : routeur central pour connecter plusieurs VPCs et réseaux on-premise. Remplace les VPC Peerings maillés complexes.

---

## 5. Encryption — KMS et gestion des clés

### KMS — Key Management Service

```
Envelope Encryption :
┌─────────────────────────────────────────────────────┐
│ 1. KMS génère une Data Encryption Key (DEK) de 256 bits│
│ 2. Chiffre vos données avec la DEK (AES-256)         │
│ 3. Chiffre la DEK avec votre CMK (Customer Master Key)│
│ 4. Stocke la DEK chiffrée avec les données            │
│ 5. La CMK ne quitte JAMAIS les HSMs de KMS           │
└─────────────────────────────────────────────────────┘
```

```bash
# Créer une CMK symétrique
aws kms create-key \
    --description "Production database encryption key" \
    --key-usage ENCRYPT_DECRYPT \
    --key-spec SYMMETRIC_DEFAULT

# Chiffrer des données directement (≤ 4 KB)
aws kms encrypt \
    --key-id alias/prod-db-key \
    --plaintext fileb://secret.txt \
    --output text --query CiphertextBlob | base64 -d > secret.enc

# Déchiffrer
aws kms decrypt \
    --ciphertext-blob fileb://secret.enc \
    --output text --query Plaintext | base64 -d

# Générer une DEK pour envelope encryption
aws kms generate-data-key \
    --key-id alias/prod-db-key \
    --key-spec AES_256

# Rotation automatique des clés
aws kms enable-key-rotation --key-id alias/prod-db-key
```

### Options de chiffrement S3

| Mode | Description | Clé gérée par |
|------|-----------|-------------|
| **SSE-S3** | AES-256, AWS gère tout | AWS |
| **SSE-KMS** | Utilise votre CMK KMS, audit CloudTrail de chaque accès | Vous (dans KMS) |
| **SSE-C** | Vous fournissez la clé à chaque requête (AWS n'a jamais la clé) | Vous (jamais stockée AWS) |
| **Client-side** | Chiffrement avant d'uploader, AWS ne voit que les données chiffrées | Vous (hors AWS) |

```bash
# Forcer le chiffrement SSE-KMS sur un bucket (bucket policy)
aws s3api put-bucket-policy --bucket my-bucket --policy '{
  "Statement": [{
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:PutObject",
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
      "StringNotEquals": {
        "s3:x-amz-server-side-encryption": "aws:kms"
      }
    }
  }]
}'
```

### CloudHSM — Modules de sécurité matériels dédiés

Pour les exigences de conformité FIPS 140-2 Level 3 (secteur financier, santé) :
- Vos propres HSMs physiques dans le datacenter AWS
- AWS n'a PAS accès à vos clés
- Vous seul pouvez gérer les clés (et les perdre si vous perdez le PIN admin)
- 10x plus cher que KMS → uniquement si compliance l'impose

---

## 6. Audit et Compliance

### CloudTrail — Journalisation des API

```bash
# Vérifier que CloudTrail est actif
aws cloudtrail describe-trails

# Créer un trail multi-région
aws cloudtrail create-trail \
    --name org-audit-trail \
    --s3-bucket-name my-audit-bucket \
    --is-multi-region-trail \
    --enable-log-file-validation \
    --cloud-watch-logs-log-group-arn arn:aws:logs:...:log-group:cloudtrail
```

**Events importants à surveiller** :
```
Management events (contrôle plane) - actifs par défaut :
  - IAM : CreateUser, AttachUserPolicy, CreateAccessKey
  - EC2 : RunInstances, TerminateInstances, ModifySecurityGroup
  - KMS : DisableKey, ScheduleKeyDeletion
  - S3 : DeleteBucket, PutBucketPolicy, PutBucketAcl

Data events (data plane) - désactivés par défaut (coût !) :
  - S3 : GetObject, PutObject, DeleteObject (par bucket)
  - Lambda : InvokeFunction
  - DynamoDB : GetItem, PutItem

CloudTrail Insights : détection automatique d'anomalies (pic d'API calls)
```

### AWS Config — Compliance as Code

```python
# Règle Config : vérifier que toutes les EC2 ont un tag "Environment"
import boto3

def lambda_handler(event, context):
    config = boto3.client('config')
    ec2 = boto3.client('ec2')

    # L'instance à évaluer
    instance_id = event['configurationItem']['resourceId']
    instance = ec2.describe_instances(InstanceIds=[instance_id])

    tags = {tag['Key']: tag['Value']
            for tag in instance['Reservations'][0]['Instances'][0].get('Tags', [])}

    if 'Environment' not in tags:
        compliance = 'NON_COMPLIANT'
        annotation = f"Instance {instance_id} manque le tag 'Environment'"
    else:
        compliance = 'COMPLIANT'
        annotation = "Tag Environment présent"

    config.put_evaluations(
        Evaluations=[{
            'ComplianceResourceType': 'AWS::EC2::Instance',
            'ComplianceResourceId': instance_id,
            'ComplianceType': compliance,
            'Annotation': annotation,
            'OrderingTimestamp': event['configurationItem']['configurationItemCaptureTime']
        }],
        ResultToken=event['resultToken']
    )
```

**Règles Config AWS managées** (exemples) :
- `encrypted-volumes` : volumes EBS chiffrés
- `root-account-mfa-enabled` : MFA sur le compte root
- `s3-bucket-public-read-prohibited` : pas de buckets S3 publics
- `vpc-flow-logs-enabled` : VPC Flow Logs actifs
- `restricted-ssh` : SSH (port 22) pas ouvert sur 0.0.0.0/0

### GuardDuty — Détection de menaces ML

GuardDuty analyse les VPC Flow Logs, CloudTrail, DNS Logs et Route53 avec des modèles ML pour détecter les menaces sans configuration manuelle.

**Types de findings** :

| Catégorie | Finding | Signification |
|-----------|---------|-------------|
| Recon | `Recon:EC2/Portscan` | Votre EC2 scanne des ports |
| Backdoor | `Backdoor:EC2/C&CActivity` | Connexion à un serveur C2 connu |
| CryptoCurrency | `CryptoCurrency:EC2/BitcoinTool` | Mining crypto sur votre EC2 |
| UnauthorizedAccess | `UnauthorizedAccess:IAMUser/ConsoleLoginSuccess.B` | Connexion depuis Tor |
| Persistence | `Persistence:IAMUser/UserPermissions` | Escalade de privilèges |
| Exfiltration | `Exfiltration:S3/ObjectRead.Unusual` | Volume de lecture S3 anormal |

```bash
# Activer GuardDuty
aws guardduty create-detector --enable

# Lister les findings critiques
aws guardduty list-findings \
    --detector-id xxx \
    --finding-criteria '{"Criterion": {"severity": {"Gte": 7}}}'
```

### Security Hub — Agrégateur CSPM

Security Hub aggrège les findings de GuardDuty, Inspector, Macie, Config et des outils tiers, et les normalise au format ASFF (Amazon Security Finding Format).

```bash
# Activer Security Hub avec les standards de sécurité
aws securityhub enable-security-hub \
    --enable-default-standards  # CIS AWS Benchmark + AWS Foundational Security Best Practices
```

### Macie — Classification S3

Macie analyse automatiquement le contenu des buckets S3 pour détecter des données sensibles (numéros de carte de crédit, SSN, données médicales, credentials).

---

## 7. Secrets Management

### AWS Secrets Manager

```python
import boto3
import json

def get_secret(secret_name, region="eu-west-1"):
    client = boto3.client("secretsmanager", region_name=region)

    response = client.get_secret_value(SecretId=secret_name)

    if 'SecretString' in response:
        return json.loads(response['SecretString'])
    else:
        return response['SecretBinary']

# Usage
db_creds = get_secret("prod/myapp/db")
conn = psycopg2.connect(
    host=db_creds["host"],
    database=db_creds["dbname"],
    user=db_creds["username"],
    password=db_creds["password"]
)
```

**Rotation automatique** :
```bash
# Rotation automatique du mot de passe RDS toutes les 30 jours
aws secretsmanager rotate-secret \
    --secret-id prod/myapp/db \
    --rotation-lambda-arn arn:aws:lambda:...:function:SecretsManagerRDSRotation \
    --rotation-rules AutomaticallyAfterDays=30
```

### SSM Parameter Store

```bash
# Stocker un secret chiffré avec KMS
aws ssm put-parameter \
    --name "/prod/myapp/api-key" \
    --value "sk-1234567890" \
    --type SecureString \
    --key-id alias/prod-key

# Lire
aws ssm get-parameter \
    --name "/prod/myapp/api-key" \
    --with-decryption

# Lire tous les params d'un chemin
aws ssm get-parameters-by-path \
    --path "/prod/myapp/" \
    --with-decryption
```

| Critère | Secrets Manager | SSM Parameter Store |
|---------|----------------|-------------------|
| Rotation automatique | Oui (native Lambda) | Non |
| Coût | 0.40$/secret/mois | Gratuit (Standard) |
| Cross-region replication | Oui | Non |
| RDS integration | Oui | Non |
| Audit CloudTrail | Oui | Oui |

---

## 8. Container Security

### ECR Image Scanning

```bash
# Activer le scan automatique sur push
aws ecr put-image-scanning-configuration \
    --repository-name my-app \
    --image-scanning-configuration scanOnPush=true

# Déclencher un scan manuel
aws ecr start-image-scan \
    --repository-name my-app \
    --image-id imageTag=latest

# Lire les résultats
aws ecr describe-image-scan-findings \
    --repository-name my-app \
    --image-id imageTag=latest \
    --query 'imageScanFindings.findings[?severity==`CRITICAL`]'
```

### EKS IRSA — IAM Roles for Service Accounts

Sans IRSA : tous les pods du nœud partagent le rôle IAM de l'instance EC2 (moindre isolation).
Avec IRSA : chaque ServiceAccount K8s obtient son propre rôle IAM (least privilege par pod).

```bash
# Activer OIDC pour le cluster EKS
eksctl utils associate-iam-oidc-provider \
    --cluster my-cluster --approve

# Créer un service account K8s avec un rôle IAM associé
eksctl create iamserviceaccount \
    --name s3-reader \
    --namespace default \
    --cluster my-cluster \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
    --approve
```

```yaml
# Le pod peut maintenant accéder à S3 sans credentials statiques
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: s3-reader  # Le pod hérite du rôle IAM via IRSA
  containers:
    - name: app
      image: my-app:latest
```

---

## 9. WAF et Shield

### AWS WAF — Application Firewall

```json
// Web ACL avec règles communes
{
  "Rules": [
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 1,
      "OverrideAction": {"None": {}},
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet"
        }
      }
    },
    {
      "Name": "RateLimitRule",
      "Priority": 2,
      "Action": {"Block": {}},
      "Statement": {
        "RateBasedStatement": {
          "Limit": 2000,
          "AggregateKeyType": "IP"
        }
      }
    },
    {
      "Name": "GeoBlockRule",
      "Priority": 3,
      "Action": {"Block": {}},
      "Statement": {
        "GeoMatchStatement": {
          "CountryCodes": ["RU", "CN", "KP"]
        }
      }
    }
  ]
}
```

**Managed Rule Groups AWS** :
- `AWSManagedRulesCommonRuleSet` : OWASP Top 10 (XSS, SQLi...)
- `AWSManagedRulesSQLiRuleSet` : SQL injection spécifique
- `AWSManagedRulesBotControlRuleSet` : détection des bots
- `AWSManagedRulesKnownBadInputsRuleSet` : inputs malveillants connus

### Shield Standard vs Advanced

| Feature | Shield Standard | Shield Advanced |
|---------|----------------|----------------|
| Coût | Gratuit | 3000$/mois + data transfer |
| Protection L3/L4 | Oui | Oui, renforcée |
| Protection L7 (DDoS app) | Non | Oui (avec WAF) |
| 24/7 DDoS Response Team | Non | Oui |
| Credits de coût DDoS | Non | Oui (remboursement des surcoûts EC2/ELB) |
| Advanced monitoring | Non | Oui (Health-based detection) |

---

## 10. Incident Response AWS

### Processus IR dans AWS

```
1. DETECT   → GuardDuty finding / CloudWatch alarm / utilisateur qui signale
2. CONTAIN  → Isoler l'instance, révoquer les credentials IAM compromis
3. ANALYZE  → CloudTrail logs, VPC Flow Logs, Memory dump via SSM
4. ERADICATE → Supprimer les ressources compromises, corriger la vulnérabilité
5. RECOVER  → Restaurer depuis des snapshots sains
6. LESSONS  → Postmortem, amélioration des détections
```

### Investigation avec CloudWatch Logs Insights

```sql
-- Trouver toutes les actions d'un utilisateur suspect dans CloudTrail
fields @timestamp, eventName, sourceIPAddress, userAgent, errorCode
| filter userIdentity.userName = "alice"
| filter @timestamp > ago(24h)
| sort @timestamp desc
| limit 200

-- Détecter les tentatives de création de backdoor IAM
fields @timestamp, userIdentity.userName, requestParameters.userName
| filter eventName in ["CreateUser", "CreateAccessKey", "AttachUserPolicy"]
| filter userIdentity.type = "AssumedRole"
| sort @timestamp desc

-- Analyser le trafic réseau suspect (VPC Flow Logs)
fields @timestamp, srcAddr, dstAddr, dstPort, action, bytes
| filter action = "REJECT" and bytes > 0
| filter dstPort not in [80, 443, 22]
| stats sum(bytes) as totalBytes by srcAddr, dstAddr, dstPort
| sort totalBytes desc
```

### Isolation d'une instance compromise

```bash
# 1. Créer un Security Group "quarantine" (rien en entrée, rien en sortie)
aws ec2 create-security-group \
    --group-name quarantine \
    --description "Quarantine: no traffic allowed"

# 2. Attacher le SG de quarantaine à l'instance (en remplaçant tous les autres SGs)
aws ec2 modify-instance-attribute \
    --instance-id i-compromised \
    --groups sg-quarantine-id

# 3. Créer un snapshot pour analyse forensique
aws ec2 create-snapshot \
    --volume-id vol-xxx \
    --description "Forensic snapshot - incident 2024-01-15"

# 4. Révoquer les credentials IAM associés à l'instance
aws iam delete-access-key --access-key-id AKIA...

# 5. Désactiver le role IAM de l'instance (plus d'appels API)
aws iam put-role-policy \
    --role-name compromised-role \
    --policy-name DenyAll \
    --policy-document '{"Statement":[{"Effect":"Deny","Action":"*","Resource":"*"}]}'
```

### AWS Detective — Investigation graphique

Detective construit automatiquement un graphe de comportement à partir de CloudTrail, VPC Flow Logs et GuardDuty findings. Permet de visualiser les connexions entre entités (quelles instances ont interagi avec cet IP, quels utilisateurs ont utilisé ce rôle).

---

## 11. Wikilinks et ressources

- [[02 - Services Cloud Essentiels]] — S3, EC2, IAM, VPC bases
- [[03 - Deploiement Cloud et Conteneurs]] — ECS, EKS, Docker en prod
- [[07 - DevSecOps]] — Sécurité dans le pipeline CI/CD
- [[01 - Fondamentaux Hacking Ethique]] — Perspective attaquant pour mieux défendre

---

## Exercices Pratiques

### Exercice 1 — IAM Least Privilege
```bash
# Créer un utilisateur de CI/CD qui peut seulement :
# - Pousser des images dans ECR (dans le repo my-app uniquement)
# - Mettre à jour un service ECS spécifique
# - Lire les logs CloudWatch du cluster ECS

# 1. Rédiger la policy JSON avec les permissions minimales
# 2. Tester avec aws iam simulate-principal-policy
# 3. Vérifier que les accès non autorisés sont bien refusés

aws iam simulate-principal-policy \
    --policy-source-arn arn:aws:iam::123:user/ci-bot \
    --action-names "ecr:PutImage" "s3:DeleteBucket" \
    --resource-arns "arn:aws:ecr:eu-west-1:123:repository/my-app" "*"
```

### Exercice 2 — Détection CloudTrail
Écrire une CloudWatch Logs Metric Filter + Alarm qui déclenche une alerte SNS quand :
1. Root account se connecte à la console AWS
2. Une politique IAM est modifiée hors des heures ouvrées (18h-8h)
3. Un bucket S3 passe en accès public
4. Une Security Group rule ouvre le port 22 à 0.0.0.0/0

### Exercice 3 — Architecture sécurisée
Concevoir une architecture AWS pour une application web RGPD avec :
- Base de données PostgreSQL (données personnelles)
- API backend (Python FastAPI)
- Frontend React sur CloudFront + S3

Exigences : chiffrement au repos + en transit, pas d'accès internet direct à la DB, logs d'accès complets, rotation des credentials DB automatique, backups chiffrés.

Livrable : diagramme d'architecture avec les services AWS choisis et justifications de chaque décision de sécurité.

### Exercice 4 — GuardDuty Response
```python
# Lambda déclenchée par un GuardDuty finding de criticité >= HIGH
# Elle doit automatiquement :
# 1. Récupérer l'instance EC2 concernée
# 2. Lui attacher un Security Group de quarantaine
# 3. Créer un snapshot EBS pour analyse forensique
# 4. Envoyer une notification Slack avec le finding
# 5. Créer un ticket dans Jira via l'API

import boto3
import json

def handler(event, context):
    finding = event['detail']
    severity = finding['severity']

    if severity >= 7:  # HIGH ou CRITICAL
        instance_id = finding['resource']['instanceDetails']['instanceId']
        # À compléter...
```

> [!tip] Règles d'or sécurité AWS
> 1. Jamais de credentials AWS dans le code ou les variables d'environnement — utiliser IAM Roles
> 2. MFA obligatoire sur le compte root et les comptes admin humains
> 3. CloudTrail actif dans toutes les régions avec log file validation
> 4. GuardDuty actif : le ROI est immédiat pour le coût ($)
> 5. Pas de Security Group 0.0.0.0/0 sur le port 22 ou 3389 — utiliser SSM Session Manager
> 6. Chiffrement SSE-KMS sur tous les buckets S3 contenant des données sensibles
> 7. VPC Endpoints pour éviter que les données traversent internet
