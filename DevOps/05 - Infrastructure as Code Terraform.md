# Infrastructure as Code avec Terraform

## Qu'est-ce que l'Infrastructure as Code (IaC) ?

L'**Infrastructure as Code** est une pratique qui consiste a gerer et provisionner l'infrastructure informatique (serveurs, reseaux, bases de donnees, load balancers) a travers du **code** plutot que par des processus manuels.

> [!tip] Analogie
> Imagine la construction d'une **maison** :
> - **Sans IaC** = tu construis ta maison **a la main**, brique par brique, sans plan. Si tu veux en construire une deuxieme identique, tu dois tout refaire de memoire. Si tu fais une erreur, tu dois casser et reconstruire.
> - **Avec IaC** = tu as un **plan d'architecte** (le code). Tu donnes le plan a un robot (Terraform) et il construit la maison exactement comme decrit. Tu veux 10 maisons identiques ? Tu lances le plan 10 fois.
>
> L'IaC, c'est le **plan d'architecte** de ton infrastructure cloud.

---

## Infrastructure manuelle vs automatisee

### Sans IaC (l'enfer des clics)

```
Administrateur : "J'ai cree le serveur en cliquant dans la console AWS"
Collegue : "Tu as utilise quelle taille d'instance ?"
Administrateur : "Euh... je ne me souviens plus"
Chef : "Il faut recreer le meme environnement en staging"
Administrateur : "Ca va me prendre 3 jours..."
```

### Avec IaC (le paradis du code)

| Aspect | Manuel | IaC |
|---|---|---|
| **Reproductibilite** | Impossible a reproduire exactement | Identique a chaque execution |
| **Documentation** | "Le serveur est configure comme ca... je crois" | Le code EST la documentation |
| **Versionnage** | Pas d'historique | Git : qui a change quoi et quand |
| **Vitesse** | Des heures de clics | Quelques minutes d'execution |
| **Erreurs** | Frequentes (oubli, faute de frappe) | Reduites (le code est verifie) |
| **Scalabilite** | 1 serveur = 1 heure, 100 serveurs = 100 heures | 1 ou 100 serveurs = meme code |

---

## Approches : imperatif vs declaratif

```
┌─────────────────────────────────────────────────────────────────────┐
│                     IMPERATIF vs DECLARATIF                         │
├──────────────────────────────┬──────────────────────────────────────┤
│         IMPERATIF            │           DECLARATIF                 │
│    "Comment le faire"        │      "Ce que je veux"               │
│                              │                                      │
│  1. Cree un VPC              │  Je veux :                          │
│  2. Cree un sous-reseau      │  - 1 VPC                            │
│  3. Cree un security group   │  - 1 sous-reseau                    │
│  4. Cree une instance EC2    │  - 1 security group                 │
│  5. Attache le security group│  - 1 instance EC2                   │
│     a l'instance             │                                      │
│                              │  → Terraform s'occupe du comment    │
│  Exemples : scripts Bash,    │  Exemples : Terraform, CloudForm.   │
│  Ansible, Pulumi (Python)    │                                      │
└──────────────────────────────┴──────────────────────────────────────┘
```

> [!info] Terraform est declaratif
> Tu decris l'**etat final desire** de ton infrastructure. Terraform calcule les etapes necessaires pour y arriver. Si l'infrastructure existe deja dans le bon etat, Terraform ne fait **rien**.

---

## Pourquoi Terraform ?

### Les forces de Terraform

| Avantage | Explication |
|---|---|
| **Multi-cloud** | Un seul outil pour AWS, GCP, Azure, et 3000+ providers |
| **Declaratif** | Tu decris ce que tu veux, pas comment le faire |
| **Plan avant execution** | `terraform plan` montre ce qui va changer AVANT d'appliquer |
| **State management** | Terraform sait ce qui existe deja et ne recree rien inutilement |
| **Ecosysteme riche** | Des milliers de modules reutilisables sur le Terraform Registry |
| **Open source** | Gratuit et communautaire |
| **Idempotent** | Relancer le meme code produit le meme resultat |

> [!warning] Terraform ne gere pas la configuration des serveurs
> Terraform **cree** l'infrastructure (serveurs, reseaux, BDD). Pour **configurer** ce qu'il y a **dans** les serveurs (installer des paquets, deployer du code), on utilise **Ansible**, **Chef** ou **Puppet**. Ce sont des outils complementaires, pas concurrents.

---

## Installer Terraform

### Sur macOS

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

### Sur Ubuntu/Debian

```bash
# Ajouter la cle GPG et le repo HashiCorp
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install terraform
```

### Sur Windows

```bash
# Avec Chocolatey
choco install terraform

# Ou avec winget
winget install Hashicorp.Terraform
```

### Verifier l'installation

```bash
terraform version
# Terraform v1.9.x on ...
```

---

## Le langage HCL (HashiCorp Configuration Language)

Terraform utilise **HCL**, un langage declaratif concu specifiquement pour decrire de l'infrastructure.

### Structure de base : les blocs

```hcl
# Un bloc a un TYPE, des LABELS et un CORPS
type "label1" "label2" {
  argument1 = "valeur1"
  argument2 = 42

  bloc_imbrique {
    argument3 = true
  }
}
```

### Les elements du langage

| Element | Description | Exemple |
|---|---|---|
| **Bloc** | Conteneur avec un type et des labels | `resource "aws_instance" "web" { }` |
| **Argument** | Assignation d'une valeur | `ami = "ami-0c55b159cbfafe1f0"` |
| **Expression** | Valeur qui peut etre calculee | `"Hello, ${var.name}"` |
| **Commentaire** | Ligne ignoree | `# Ceci est un commentaire` |

### Exemples de syntaxe HCL

```hcl
# Commentaire sur une ligne
/* Commentaire
   sur plusieurs lignes */

# Types de valeurs
nom        = "mon-serveur"         # string
count      = 3                      # number
actif      = true                   # bool
tags       = ["web", "prod"]        # list
config     = {                      # map
  port     = 8080
  protocol = "tcp"
}
```

---

## Les Providers

### Qu'est-ce qu'un provider ?

Un **provider** est un plugin qui permet a Terraform de communiquer avec une API externe (AWS, GCP, Azure, GitHub, Docker, Kubernetes...).

```
┌──────────────────────────────────────────────────────────────┐
│                         TERRAFORM                             │
│                            |                                  │
│              ┌─────────────┼─────────────┐                    │
│              |             |             |                     │
│         ┌────▼────┐  ┌────▼────┐  ┌────▼────┐                │
│         │   AWS   │  │   GCP   │  │  Azure  │                │
│         │Provider │  │Provider │  │Provider │                │
│         └────┬────┘  └────┬────┘  └────┬────┘                │
│              |             |             |                     │
│         ┌────▼────┐  ┌────▼────┐  ┌────▼────┐                │
│         │ AWS API │  │ GCP API │  │Azure API│                │
│         └─────────┘  └─────────┘  └─────────┘                │
└──────────────────────────────────────────────────────────────┘
```

### Le bloc provider

```hcl
# Configuration du provider AWS
provider "aws" {
  region  = "eu-west-3"    # Paris
  profile = "default"       # Profil AWS CLI
}

# Configuration du provider GCP
provider "google" {
  project = "mon-projet-gcp"
  region  = "europe-west1"
}
```

### terraform init

La commande `terraform init` **telecharge les providers** necessaires. C'est la premiere commande a executer dans un projet Terraform.

```bash
terraform init
# Initializing the backend...
# Initializing provider plugins...
# - Finding hashicorp/aws versions matching "~> 5.0"...
# - Installing hashicorp/aws v5.31.0...
# Terraform has been successfully initialized!
```

> [!info] Le dossier .terraform/
> `terraform init` cree un dossier `.terraform/` qui contient les plugins telecharges. Ce dossier est **lourd** et ne doit **jamais** etre commite dans Git (ajoute-le a `.gitignore`).

### Versionner les providers

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"    # >= 5.0, < 6.0
    }
  }
}
```

---

## Les Resources

### Le bloc resource

Une **resource** represente un composant d'infrastructure : un serveur, un reseau, une base de donnees, un enregistrement DNS...

```hcl
resource "type_de_resource" "nom_local" {
  argument1 = "valeur"
  argument2 = "valeur"
}
```

- **type_de_resource** : determine par le provider (ex: `aws_instance`, `google_compute_instance`)
- **nom_local** : identifiant unique dans ton code Terraform (ex: `web_server`, `database`)

### Exemple : un Security Group AWS

```hcl
resource "aws_security_group" "web_sg" {
  name        = "web-security-group"
  description = "Autorise HTTP et SSH"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Cycle de vie des resources

```
┌──────────────────────────────────────────────────────────────────┐
│                  CYCLE DE VIE D'UNE RESOURCE                     │
│                                                                   │
│   terraform apply          terraform apply         terraform      │
│   (premiere fois)          (modification)          destroy        │
│        |                        |                      |          │
│   ┌────▼────┐             ┌────▼────┐            ┌────▼────┐     │
│   │  CREATE │             │  UPDATE │            │ DESTROY │     │
│   │         │             │  ou     │            │         │     │
│   │ Cree la │             │ REPLACE │            │ Supprime│     │
│   │resource │             │         │            │ la      │     │
│   └─────────┘             └─────────┘            │resource │     │
│                                                   └─────────┘     │
│                                                                   │
│   UPDATE : modification en place (ex: changer un tag)             │
│   REPLACE : detruire + recreer (ex: changer l'AMI d'une EC2)     │
└──────────────────────────────────────────────────────────────────┘
```

> [!warning] Certaines modifications forcent un REPLACE
> Changer l'AMI d'une instance EC2 ou le type d'une instance necessite de **detruire** l'ancienne et d'en **recreer** une nouvelle. Terraform le montre clairement dans le plan avec `# forces replacement`.

---

## Premier projet : creer une instance EC2 sur AWS

### Structure du projet

```
mon-premier-projet/
├── main.tf           # Resources principales
├── variables.tf      # Declaration des variables
├── outputs.tf        # Valeurs de sortie
├── terraform.tfvars  # Valeurs des variables
└── .gitignore        # Exclure .terraform/ et *.tfstate
```

### Fichier main.tf

```hcl
# ── Configuration Terraform ──────────────────────────────
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# ── Provider AWS ─────────────────────────────────────────
provider "aws" {
  region = var.aws_region
}

# ── Security Group ───────────────────────────────────────
resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Autorise HTTP et SSH"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.my_ip]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-security-group"
  }
}

# ── Instance EC2 ─────────────────────────────────────────
resource "aws_instance" "web_server" {
  ami           = var.ami_id
  instance_type = var.instance_type

  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
    #!/bin/bash
    apt-get update -y
    apt-get install -y nginx
    systemctl start nginx
    echo "<h1>Deploye avec Terraform !</h1>" > /var/www/html/index.html
  EOF

  tags = {
    Name        = "web-server"
    Environment = "dev"
    ManagedBy   = "terraform"
  }
}
```

### Fichier variables.tf

```hcl
variable "aws_region" {
  description = "Region AWS"
  type        = string
  default     = "eu-west-3"
}

variable "instance_type" {
  description = "Type d'instance EC2"
  type        = string
  default     = "t2.micro"
}

variable "ami_id" {
  description = "AMI Ubuntu 22.04 pour eu-west-3"
  type        = string
  default     = "ami-0c55b159cbfafe1f0"
}

variable "my_ip" {
  description = "Mon adresse IP pour SSH (format CIDR)"
  type        = string
}
```

### Fichier outputs.tf

```hcl
output "instance_id" {
  description = "ID de l'instance EC2"
  value       = aws_instance.web_server.id
}

output "public_ip" {
  description = "Adresse IP publique du serveur"
  value       = aws_instance.web_server.public_ip
}

output "public_url" {
  description = "URL du serveur web"
  value       = "http://${aws_instance.web_server.public_ip}"
}
```

### Fichier terraform.tfvars

```hcl
aws_region    = "eu-west-3"
instance_type = "t2.micro"
my_ip         = "203.0.113.50/32"   # Remplace par ton IP
```

### Execution

```bash
# 1. Initialiser le projet
terraform init

# 2. Voir le plan d'execution
terraform plan

# 3. Appliquer les changements
terraform apply
# Terraform affiche le plan et demande confirmation
# Enter a value: yes

# 4. Voir les outputs
terraform output
# instance_id = "i-0abcd1234efgh5678"
# public_ip = "52.47.123.45"
# public_url = "http://52.47.123.45"

# 5. Detruire l'infrastructure (quand on a fini)
terraform destroy
```

---

## Les commandes Terraform essentielles

| Commande | Description |
|---|---|
| `terraform init` | Initialise le projet, telecharge les providers |
| `terraform plan` | Montre ce qui va etre cree/modifie/detruit |
| `terraform apply` | Applique les changements (avec confirmation) |
| `terraform destroy` | Detruit toute l'infrastructure geree |
| `terraform fmt` | Formate le code HCL proprement |
| `terraform validate` | Verifie la syntaxe et la coherence du code |
| `terraform show` | Affiche l'etat actuel de l'infrastructure |
| `terraform output` | Affiche les valeurs de sortie |
| `terraform state list` | Liste les resources dans le state |
| `terraform state show` | Affiche le detail d'une resource dans le state |

### terraform plan en detail

```bash
terraform plan
```

```
# aws_instance.web_server will be created
+ resource "aws_instance" "web_server" {
    + ami                    = "ami-0c55b159cbfafe1f0"
    + instance_type          = "t2.micro"
    + id                     = (known after apply)
    + public_ip              = (known after apply)
    + tags                   = {
        + "Name" = "web-server"
      }
  }

Plan: 2 to add, 0 to change, 0 to destroy.
```

> [!info] Les symboles du plan
> - `+` = resource a **creer**
> - `-` = resource a **detruire**
> - `~` = resource a **modifier en place**
> - `-/+` = resource a **detruire et recreer**

---

## Le State File (terraform.tfstate)

### Qu'est-ce que le state ?

Le **state** est un fichier JSON (`terraform.tfstate`) qui contient l'etat actuel de l'infrastructure geree par Terraform. C'est la **memoire** de Terraform.

```
┌─────────────────────────────────────────────────────────────────┐
│                    LE ROLE DU STATE                               │
│                                                                   │
│   Code HCL              State File            Infrastructure     │
│   (ce que tu veux)      (ce qui existe)        (la realite)      │
│        |                     |                      |             │
│        └─────────┬───────────┘                      |             │
│                  |                                   |             │
│             terraform plan                           |             │
│           (compare code vs state)                    |             │
│                  |                                   |             │
│             terraform apply ────────────────────────>|             │
│           (applique les differences)                 |             │
│                  |                                   |             │
│             terraform refresh                        |             │
│           (met a jour le state <──────────────────── |             │
│            depuis la realite)                                     │
└─────────────────────────────────────────────────────────────────┘
```

### Contenu du state file

```json
{
  "version": 4,
  "terraform_version": "1.9.0",
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "web_server",
      "instances": [
        {
          "attributes": {
            "id": "i-0abcd1234efgh5678",
            "ami": "ami-0c55b159cbfafe1f0",
            "instance_type": "t2.micro",
            "public_ip": "52.47.123.45"
          }
        }
      ]
    }
  ]
}
```

> [!warning] Le state est SENSIBLE
> Le state contient des informations sensibles (mots de passe de BDD, cles d'API). Il ne doit **JAMAIS** etre commite dans Git. Utilise un **backend distant** (S3, GCS) pour le stocker de maniere securisee.

### Remote State : backend S3

```hcl
terraform {
  backend "s3" {
    bucket         = "mon-terraform-state"
    key            = "prod/infrastructure.tfstate"
    region         = "eu-west-3"
    encrypt        = true
    dynamodb_table = "terraform-locks"   # State locking
  }
}
```

> [!info] State Locking
> Le **state locking** empeche deux personnes d'appliquer des changements en meme temps. Avec S3, on utilise une table **DynamoDB** pour le verrou. Si quelqu'un fait `terraform apply`, les autres sont bloques jusqu'a la fin.

---

## Les Variables

### Le bloc variable

```hcl
variable "nom_de_la_variable" {
  description = "Description de la variable"
  type        = string          # Type de la variable
  default     = "valeur"        # Valeur par defaut (optionnel)
  sensitive   = false           # Masquer dans les logs (optionnel)
  validation {                  # Validation personnalisee (optionnel)
    condition     = length(var.nom_de_la_variable) > 0
    error_message = "Le nom ne peut pas etre vide."
  }
}
```

### Types de variables

| Type | Exemple | Description |
|---|---|---|
| `string` | `"hello"` | Chaine de caracteres |
| `number` | `42` | Nombre entier ou decimal |
| `bool` | `true` | Booleen |
| `list(string)` | `["a", "b", "c"]` | Liste de strings |
| `map(string)` | `{key = "value"}` | Dictionnaire string -> string |
| `set(string)` | `toset(["a", "b"])` | Ensemble (pas de doublons) |
| `object({...})` | `{name = string, age = number}` | Objet structure |
| `tuple([...])` | `[string, number, bool]` | Tuple type |

### Passer des valeurs aux variables

```bash
# 1. Fichier terraform.tfvars (charge automatiquement)
# terraform.tfvars
instance_type = "t2.micro"
region        = "eu-west-3"

# 2. Flag -var sur la ligne de commande
terraform apply -var="instance_type=t2.small"

# 3. Fichier .tfvars personnalise
terraform apply -var-file="production.tfvars"

# 4. Variables d'environnement (prefixe TF_VAR_)
export TF_VAR_instance_type="t2.micro"
export TF_VAR_region="eu-west-3"
terraform apply
```

> [!info] Ordre de priorite des variables
> Du moins prioritaire au plus prioritaire :
> 1. Valeur `default` dans le bloc `variable`
> 2. Fichier `terraform.tfvars`
> 3. Fichier `*.auto.tfvars`
> 4. `-var-file` sur la ligne de commande
> 5. `-var` sur la ligne de commande
> 6. Variables d'environnement `TF_VAR_*`

---

## Les Outputs

### Le bloc output

Les **outputs** permettent d'afficher des valeurs apres un `terraform apply` et de partager des donnees entre modules.

```hcl
output "instance_ip" {
  description = "L'adresse IP publique de l'instance"
  value       = aws_instance.web_server.public_ip
}

output "db_password" {
  description = "Mot de passe de la base de donnees"
  value       = random_password.db.result
  sensitive   = true    # Ne sera pas affiche en clair
}
```

```bash
# Afficher tous les outputs
terraform output

# Afficher un output specifique
terraform output instance_ip

# Afficher en JSON (utile pour les scripts)
terraform output -json
```

---

## Les Data Sources

Un **data source** permet de **lire** des informations depuis l'infrastructure existante sans la gerer.

```hcl
# Trouver la derniere AMI Ubuntu
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]    # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

# Utiliser le resultat
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id    # Reference au data source
  instance_type = "t2.micro"
}
```

> [!example] Cas d'usage des data sources
> - Trouver la derniere AMI disponible
> - Lire les informations d'un VPC existant
> - Obtenir l'ID de la zone DNS
> - Lire un secret depuis AWS Secrets Manager
> - Recuperer les AZ disponibles dans une region

---

## Les Modules

### Pourquoi les modules ?

Un **module** est un ensemble de fichiers Terraform regroupes dans un dossier. Les modules permettent de **reutiliser** du code et d'**organiser** les projets.

```
Sans modules :                      Avec modules :
                                    
main.tf (500 lignes)               modules/
  - VPC                              ├── vpc/
  - Subnets                          │   ├── main.tf
  - Security Groups                  │   ├── variables.tf
  - EC2                              │   └── outputs.tf
  - RDS                              ├── ec2/
  - S3                               │   ├── main.tf
  - ...                              │   ├── variables.tf
                                     │   └── outputs.tf
Impossible a maintenir !             └── rds/
                                         ├── main.tf
                                         ├── variables.tf
                                         └── outputs.tf
                                    
                                    Propre et reutilisable !
```

### Creer un module

```hcl
# modules/ec2/variables.tf
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "ami_id" {
  type = string
}

variable "name" {
  type = string
}

# modules/ec2/main.tf
resource "aws_instance" "this" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = var.name
  }
}

# modules/ec2/outputs.tf
output "instance_id" {
  value = aws_instance.this.id
}

output "public_ip" {
  value = aws_instance.this.public_ip
}
```

### Utiliser un module

```hcl
# main.tf (projet racine)
module "web_server" {
  source = "./modules/ec2"

  name          = "web-server"
  ami_id        = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.small"
}

module "api_server" {
  source = "./modules/ec2"

  name          = "api-server"
  ami_id        = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

# Acceder aux outputs du module
output "web_ip" {
  value = module.web_server.public_ip
}
```

### Sources de modules

```hcl
# Module local
module "vpc" {
  source = "./modules/vpc"
}

# Module du Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"
}

# Module Git
module "vpc" {
  source = "git::https://github.com/org/terraform-vpc.git?ref=v1.0.0"
}
```

---

## Les Expressions

### Interpolation de strings

```hcl
# Interpolation simple
name = "server-${var.environment}"

# Interpolation avec fonction
name = "server-${lower(var.environment)}"
```

### Conditionnels

```hcl
# condition ? valeur_si_vrai : valeur_si_faux
instance_type = var.environment == "prod" ? "t2.large" : "t2.micro"

# Creer une resource conditionnellement
resource "aws_instance" "monitoring" {
  count = var.enable_monitoring ? 1 : 0

  ami           = var.ami_id
  instance_type = "t2.micro"
}
```

### Boucles avec for

```hcl
# Transformer une liste
upper_names = [for name in var.names : upper(name)]

# Filtrer une liste
prod_servers = [for s in var.servers : s if s.environment == "prod"]

# Creer une map
name_to_ip = { for s in var.servers : s.name => s.ip }
```

### Boucles avec for_each et count

```hcl
# count : creer N resources identiques
resource "aws_instance" "web" {
  count         = 3
  ami           = var.ami_id
  instance_type = "t2.micro"

  tags = {
    Name = "web-server-${count.index}"
  }
}

# for_each : creer des resources a partir d'une map
resource "aws_instance" "servers" {
  for_each      = var.server_config
  ami           = each.value.ami
  instance_type = each.value.type

  tags = {
    Name = each.key
  }
}
```

### Dynamic blocks

```hcl
# Au lieu de repeter les blocs ingress...
resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

# Variable associee
variable "ingress_rules" {
  default = [
    { port = 80,  cidr_blocks = ["0.0.0.0/0"] },
    { port = 443, cidr_blocks = ["0.0.0.0/0"] },
    { port = 22,  cidr_blocks = ["10.0.0.0/8"] },
  ]
}
```

---

## Les Provisioners

Les **provisioners** executent des commandes sur la machine locale ou distante apres la creation d'une resource.

### local-exec

```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> inventory.txt"
  }
}
```

### remote-exec

```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = "t2.micro"

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
    ]
  }
}
```

> [!warning] Eviter les provisioners autant que possible
> HashiCorp recommande d'eviter les provisioners car :
> - Ils rendent le code **non declaratif** (imperatif)
> - Ils ne sont **pas idempotents**
> - Les erreurs sont difficiles a gerer
> - Utilise plutot **user_data** (cloud-init) pour le bootstrap initial
> - Utilise **Ansible** pour la configuration post-creation

---

## Terraform Cloud / Enterprise

| Feature | CLI (Open Source) | Terraform Cloud (Gratuit) | Enterprise |
|---|---|---|---|
| Plan & Apply | Local | Distant | Distant |
| Remote State | Backend S3/GCS | Integre | Integre |
| State Locking | DynamoDB/manual | Automatique | Automatique |
| Policy as Code | Non | Sentinel | Sentinel |
| Private Registry | Non | Oui | Oui |
| SSO / SAML | Non | Non | Oui |

> [!info] Terraform Cloud
> **Terraform Cloud** est un service gratuit de HashiCorp qui stocke le state de maniere securisee, execute les plans a distance, et permet la collaboration en equipe. C'est un excellent point de depart avant d'investir dans des solutions plus complexes.

---

## Bonnes pratiques

| Pratique | Pourquoi |
|---|---|
| **Petits modules** | Un module = une responsabilite. Facile a tester et reutiliser |
| **Remote state** | Jamais de state local en equipe. S3 + DynamoDB minimum |
| **Version pinning** | Toujours epingler les versions des providers et modules |
| **.gitignore** | Exclure `.terraform/`, `*.tfstate`, `*.tfstate.backup`, `*.tfvars` (s'il contient des secrets) |
| **Plan avant apply** | Toujours lire le plan. Verifier ce qui sera detruit |
| **Nommer les resources** | Des noms descriptifs : `web_server`, pas `server1` |
| **Variables pour la flexibilite** | Pas de valeurs en dur dans les resources |
| **Outputs pour le partage** | Exposer les valeurs utiles (IP, ID, URL) |
| **Formater le code** | `terraform fmt` avant chaque commit |
| **Valider le code** | `terraform validate` dans le CI/CD |

### .gitignore pour Terraform

```
# Terraform
.terraform/
*.tfstate
*.tfstate.backup
*.tfstate.*.backup
.terraform.lock.hcl
crash.log

# Variables sensibles
*.tfvars
!example.tfvars
```

---

## Comparaison : Terraform vs les alternatives

| Critere | Terraform | Ansible | CloudFormation | Pulumi |
|---|---|---|---|---|
| **Type** | Provisioning | Config Management | Provisioning | Provisioning |
| **Approche** | Declaratif (HCL) | Imperatif (YAML) | Declaratif (JSON/YAML) | Imperatif (Python, Go, JS) |
| **Multi-cloud** | Oui (3000+ providers) | Oui (modules) | Non (AWS only) | Oui |
| **State** | Fichier tfstate | Pas de state | Stack dans AWS | Fichier state |
| **Langage** | HCL | YAML | JSON / YAML | Python, Go, JS, etc. |
| **Agent** | Non (API calls) | Non (SSH) | Non (API calls) | Non (API calls) |
| **Courbe d'apprentissage** | Moyenne | Facile | Moyenne | Facile (si tu connais le langage) |
| **Force** | Creer l'infra | Configurer les serveurs | Integration AWS native | Langages familiers |
| **Cas d'usage ideal** | Provisionner infra cloud | Installer/configurer apps | Infra AWS pure | Devs qui veulent coder l'infra |

> [!tip] Terraform + Ansible = le combo gagnant
> - **Terraform** cree l'infrastructure (serveurs, reseaux, BDD)
> - **Ansible** configure ce qui tourne **dans** les serveurs (paquets, fichiers, services)
> - C'est la separation des responsabilites : Terraform pour le **provisioning**, Ansible pour la **configuration**.

---

## Carte Mentale

```
                        Infrastructure as Code
                             Terraform
                                |
            ┌───────────────────┼───────────────────┐
            |                   |                    |
        Fondamentaux         Workflow             Avance
            |                   |                    |
     ┌──────┼──────┐     ┌─────┼─────┐      ┌──────┼──────┐
     |      |      |     |     |     |      |      |      |
    HCL  Providers Resources init  plan   Modules State  Expressions
     |      |      |           |           |      |      |
  Blocs   AWS    Lifecycle  apply       Registry Remote  Conditionnels
  Args    GCP    Create     destroy     Sources  S3      for_each
  Types   Azure  Update     fmt                  Locking Dynamic
          Docker Destroy    validate
                                |
                         ┌──────┼──────┐
                         |      |      |
                     Variables Outputs  Data
                         |      |      Sources
                    string/num  value
                    list/map    sensitive
                    tfvars
```

---

## Exercices

### Exercice 1 : Premier projet Terraform

Cree un projet Terraform qui :
1. Configure le provider AWS pour la region `eu-west-3`
2. Cree un Security Group qui autorise HTTP (80) et SSH (22)
3. Cree une instance EC2 `t2.micro` avec un tag `Name`
4. Definit des variables pour la region, le type d'instance et l'AMI
5. Exporte l'IP publique et l'ID de l'instance en output

### Exercice 2 : Modules reutilisables

Reprends l'exercice 1 et :
1. Cree un module `modules/ec2` avec ses propres `variables.tf`, `main.tf` et `outputs.tf`
2. Utilise ce module depuis le `main.tf` racine pour creer 2 instances (web et api)
3. Affiche les IPs publiques des deux instances en outputs

### Exercice 3 : Variables avancees et boucles

Cree un projet Terraform qui :
1. Definit une variable `servers` de type `map(object)` avec `name`, `type`, `environment`
2. Utilise `for_each` pour creer une instance par entree
3. Utilise une condition pour assigner un type d'instance different selon l'environment (prod vs dev)
4. Utilise un dynamic block pour les regles de security group

### Exercice 4 : State et backend distant

Configure un projet Terraform avec :
1. Un backend S3 pour le state distant
2. Une table DynamoDB pour le state locking
3. Un fichier `.gitignore` adapte
4. Des outputs qui affichent le nom du bucket S3 et de la table DynamoDB

---

## Liens

- [[06 - Automatisation avec Ansible]] - Configurer les serveurs crees par Terraform
- [[01 - Introduction au Cloud]] - Comprendre le cloud avant de le provisionner
- [[04 - CI-CD avec GitHub Actions]] - Automatiser les terraform plan/apply dans un pipeline CI/CD
