# Deploiement Cloud et Conteneurs

## Introduction

Cette note couvre le **deploiement d'applications conteneurisees dans le cloud** : registries, orchestrateurs manages, load balancers, auto-scaling, strategies de deploiement zero-downtime et CI/CD pour conteneurs. Vous apprendrez a passer d'une image Docker locale a une application en production, scalable et resiliente.

> [!tip] Analogie
> Imaginez une **flotte de camions de livraison** :
> - Le **conteneur** est le colis standardise (votre application packagee)
> - Le **registry** est l'entrepot central ou sont stockes les colis
> - Le **service de deploiement** (ECS, GKE, ACI) est le dispatching qui envoie les camions
> - Le **load balancer** est le rond-point qui repartit le trafic entre les routes
> - L'**auto-scaling** est le directeur logistique qui ajoute ou retire des camions selon la demande
> - Le **CI/CD** est la chaine de production automatisee : fabriquer le colis, le stocker, l'expedier

---

## Pourquoi Deployer des Conteneurs dans le Cloud ?

### Les avantages cles

| Avantage | Description |
|---|---|
| **Portabilite** | Un conteneur fonctionne de la meme maniere sur votre poste, en staging et en production |
| **Scaling rapide** | Lancer 10 instances d'un conteneur prend quelques secondes, contre des minutes pour des VMs |
| **Infrastructure geree** | Le fournisseur cloud gere les serveurs, les mises a jour de l'OS, le reseau |
| **Isolation** | Chaque conteneur est isole, pas de conflits de dependances |
| **Densite** | Plus de conteneurs par serveur que de VMs, donc moins de gaspillage |
| **Reproductibilite** | L'image Docker est un artefact immutable, identique partout |

### Du developpement a la production

```
  DEVELOPPEMENT                    PRODUCTION CLOUD
  =============                    ================

  +----------------+               +----------------------------------+
  | docker-compose |               |  Cloud Provider                  |
  |   up           |               |                                  |
  | +----+ +----+  |               |  +----+ +----+ +----+ +----+    |
  | | API| | DB |  |  ========>    |  | API| | API| | API| | API|    |
  | +----+ +----+  |  Deployer     |  +----+ +----+ +----+ +----+    |
  |                 |               |         |                        |
  | 1 instance      |               |  +------+------+                |
  | Votre laptop    |               |  | Load Balancer|               |
  +----------------+               |  +--------------+                |
                                    |  Auto-scaling, monitoring,      |
                                    |  haute disponibilite            |
                                    +----------------------------------+
```

> [!info] Conteneur vs VM dans le cloud
> Vous pouvez deployer des VMs (EC2, Compute Engine) OU des conteneurs. Les conteneurs sont preferes pour les applications modernes car ils demarrent plus vite, consomment moins de ressources et sont plus faciles a automatiser. Les VMs restent utiles pour les applications legacy ou celles necessitant un controle total de l'OS.

---

## Container Registries (Registres de Conteneurs)

### Concept

Un **container registry** est un entrepot centralise pour stocker, versionner et distribuer vos images Docker. C'est l'equivalent d'un depot Git, mais pour les images de conteneurs.

### Les principaux registries

| Registry | Fournisseur | URL | Cas d'usage |
|---|---|---|---|
| **Docker Hub** | Docker Inc. | `hub.docker.com` | Images publiques, projets open-source |
| **ECR** | AWS | `<account>.dkr.ecr.<region>.amazonaws.com` | Deploiement sur ECS/EKS |
| **GCR / Artifact Registry** | GCP | `gcr.io/<project>` | Deploiement sur GKE |
| **ACR** | Azure | `<name>.azurecr.io` | Deploiement sur AKS/ACI |
| **GitHub Container Registry** | GitHub | `ghcr.io/<owner>` | Integration CI/CD GitHub |

### Workflow push/pull

```
  DEVELOPPEUR                    REGISTRY                     PRODUCTION
  ============                   ========                     ==========

  +----------+     docker push   +----------+   docker pull   +----------+
  | Build    | ================> | Registry | ===============>| Service  |
  | Image    |                   |          |                 | Cloud    |
  | locally  |                   | v1.0.0   |                 | (ECS,    |
  +----------+                   | v1.1.0   |                 |  GKE,    |
       |                         | v1.2.0   |                 |  AKS)    |
       |                         | latest   |                 +----------+
  docker build                   +----------+
  docker tag                          |
                                 Stocke les images
                                 avec des tags
                                 (versions)
```

### Utilisation avec Docker Hub et ECR

```bash
# === Docker Hub ===

# 1. Se connecter, construire, taguer, pousser
docker login
docker build -t mon-api:1.0.0 .
docker tag mon-api:1.0.0 votre-username/mon-api:1.0.0
docker push votre-username/mon-api:1.0.0

# Recuperer l'image (depuis n'importe ou)
docker pull votre-username/mon-api:1.0.0

# === AWS ECR ===

# 1. Creer un depot et se connecter
aws ecr create-repository --repository-name mon-api
aws ecr get-login-password --region eu-west-3 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.eu-west-3.amazonaws.com

# 2. Taguer et pousser
docker tag mon-api:1.0.0 \
  123456789.dkr.ecr.eu-west-3.amazonaws.com/mon-api:1.0.0
docker push 123456789.dkr.ecr.eu-west-3.amazonaws.com/mon-api:1.0.0

# === GCP Artifact Registry ===

gcloud auth configure-docker europe-west1-docker.pkg.dev
docker tag mon-api:1.0.0 \
  europe-west1-docker.pkg.dev/mon-projet/mon-repo/mon-api:1.0.0
docker push europe-west1-docker.pkg.dev/mon-projet/mon-repo/mon-api:1.0.0
```

> [!warning] Bonnes pratiques pour les tags
> - **Ne deployez jamais `latest` en production** : ce tag est mutable et vous ne savez pas quelle version tourne
> - Utilisez des tags semantiques (`v1.2.3`) ou le SHA du commit Git (`abc123f`)
> - Activez le **scan de vulnerabilites** sur votre registry (ECR, GCR et ACR le proposent nativement)
> - Configurez une politique de retention pour supprimer les anciennes images et economiser du stockage

---

## Deployer des Conteneurs : AWS ECS

### Elastic Container Service (ECS) - Vue d'ensemble

**ECS** est le service d'orchestration de conteneurs natif d'AWS. Il permet de deployer, gerer et scaler des conteneurs Docker sans installer Kubernetes.

### Les composants ECS

```
  ECS - Architecture
  ==================

  +---------------------------------------------------------------+
  |  CLUSTER ECS                                                   |
  |                                                                |
  |  +---------------------------+  +---------------------------+  |
  |  | SERVICE A (API)           |  | SERVICE B (Worker)        |  |
  |  | Desired count: 3          |  | Desired count: 2          |  |
  |  |                           |  |                           |  |
  |  | +------+ +------+ +------+| | +------+ +------+        |  |
  |  | |Task 1| |Task 2| |Task 3|| | |Task 1| |Task 2|        |  |
  |  | |      | |      | |      || | |      | |      |        |  |
  |  | |[API] | |[API] | |[API] || | |[Wrkr]| |[Wrkr]|        |  |
  |  | +------+ +------+ +------+| | +------+ +------+        |  |
  |  +---------------------------+  +---------------------------+  |
  |                                                                |
  |  Infrastructure : Fargate (serverless) OU EC2 (vos instances) |
  +---------------------------------------------------------------+
```

| Composant | Description |
|---|---|
| **Cluster** | Regroupement logique de services et de taches |
| **Task Definition** | Blueprint d'un conteneur (image, CPU, RAM, ports, variables d'env) |
| **Task** | Instance en cours d'execution d'une Task Definition |
| **Service** | Maintient un nombre desire de tasks en execution, gere le scaling et le load balancing |

### Task Definition (definition de tache)

La Task Definition est un fichier JSON qui decrit comment executer votre conteneur :

```json
{
  "family": "mon-api",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "api",
      "image": "123456789.dkr.ecr.eu-west-3.amazonaws.com/mon-api:1.0.0",
      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        { "name": "DATABASE_URL", "value": "postgresql://..." },
        { "name": "ENV", "value": "production" }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/mon-api",
          "awslogs-region": "eu-west-3",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8000/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

### Fargate vs EC2 Launch Type

| Caracteristique | **Fargate** (serverless) | **EC2** (instances gerees) |
|---|---|---|
| **Gestion serveurs** | Aucune (AWS gere tout) | Vous gerez les instances EC2 |
| **Scaling** | Automatique, granulaire | Vous gerez l'auto-scaling des instances |
| **Cout** | Plus cher par tache | Moins cher a grande echelle |
| **Controle** | Limite (pas d'acces SSH) | Total (acces SSH, configuration OS) |
| **Demarrage** | ~30-60 secondes | Depends de l'instance |
| **Cas d'usage** | Equipes petites, charges variables | Grandes charges, besoins specifiques |

> [!tip] Quand choisir Fargate ?
> Commencez avec **Fargate** si vous debutez. Pas de serveurs a gerer, pas de patches OS, pas de capacity planning. Vous ne payez que pour les ressources CPU/RAM de vos taches. Passez a EC2 launch type uniquement si vous avez besoin de reduire les couts a grande echelle ou d'un controle specifique sur l'infrastructure.

### Creer un service ECS avec Fargate

```bash
# 1. Creer un cluster
aws ecs create-cluster --cluster-name mon-cluster

# 2. Enregistrer la task definition
aws ecs register-task-definition \
  --cli-input-json file://task-definition.json

# 3. Creer le service avec un load balancer
aws ecs create-service \
  --cluster mon-cluster \
  --service-name mon-api-service \
  --task-definition mon-api:1 \
  --desired-count 3 \
  --launch-type FARGATE \
  --network-configuration \
    "awsvpcConfiguration={
      subnets=[subnet-xxx,subnet-yyy],
      securityGroups=[sg-xxx],
      assignPublicIp=ENABLED
    }" \
  --load-balancers \
    "targetGroupArn=arn:aws:elasticloadbalancing:...,
     containerName=api,
     containerPort=8000"

# 4. Verifier le statut
aws ecs describe-services \
  --cluster mon-cluster \
  --services mon-api-service \
  --query 'services[0].{status:status,running:runningCount,desired:desiredCount}'
```

### Integration avec ALB (Application Load Balancer)

```
  Internet
     |
     v
  +--+--+
  | ALB  |  <-- Application Load Balancer
  +--+--+      - Ecoute sur le port 443 (HTTPS)
     |         - Termine le SSL
     |         - Health checks sur /health
     |
  +--+--+--+--+
  |  |  |  |  |
  v  v  v  v  v
  +--+ +--+ +--+
  |T1| |T2| |T3|   <-- Tasks ECS (conteneurs)
  +--+ +--+ +--+       Port 8000
```

---

## Deployer des Conteneurs : Autres Plateformes

### Google Kubernetes Engine (GKE)

**GKE** est le service Kubernetes manage de Google Cloud. C'est la reference pour Kubernetes en production car Google est le createur de Kubernetes.

```bash
# Creer un cluster GKE
gcloud container clusters create mon-cluster \
  --zone europe-west1-b \
  --num-nodes 3 \
  --machine-type e2-medium

# Se connecter au cluster
gcloud container clusters get-credentials mon-cluster \
  --zone europe-west1-b

# Deployer avec kubectl (voir note Kubernetes)
kubectl apply -f deployment.yaml
```

> [!info] GKE vs ECS
> **GKE** utilise Kubernetes (standard ouvert, portable entre clouds). **ECS** est proprietaire AWS mais plus simple a configurer. Si vous voulez de la portabilite multi-cloud, choisissez Kubernetes. Si vous etes 100% AWS et voulez de la simplicite, choisissez ECS.

Pour les details complets sur Kubernetes, consultez [[04 - Kubernetes Introduction]].

### Azure Container Instances (ACI)

**ACI** est le moyen le plus simple de lancer un conteneur dans Azure. Pas de cluster a gerer, pas d'orchestrateur : vous lancez directement un conteneur.

```bash
# Lancer un conteneur en une commande
az container create \
  --resource-group mon-groupe \
  --name mon-api \
  --image mon-registry.azurecr.io/mon-api:1.0.0 \
  --cpu 1 --memory 1.5 \
  --ports 8000 \
  --dns-name-label mon-api-demo \
  --environment-variables ENV=production

# Verifier le statut
az container show --resource-group mon-groupe --name mon-api \
  --query "{status:instanceView.state,fqdn:ipAddress.fqdn}"
```

### Comparaison des plateformes conteneurs manages

| Caracteristique | **AWS ECS/Fargate** | **GKE** | **Azure ACI** | **Azure AKS** |
|---|---|---|---|---|
| **Type** | Orchestrateur proprietaire | Kubernetes manage | Conteneur a la demande | Kubernetes manage |
| **Complexite** | Moyenne | Elevee | Faible | Elevee |
| **Scaling** | Auto-scaling integre | HPA Kubernetes | Manuel / Logic Apps | HPA Kubernetes |
| **Cout minimal** | ~10$/mois (Fargate) | ~70$/mois (cluster) | ~1$/mois (pay-per-use) | ~70$/mois (cluster) |
| **Portabilite** | AWS uniquement | Multi-cloud (K8s) | Azure uniquement | Multi-cloud (K8s) |
| **Cas d'usage** | Production AWS | Production multi-cloud | Dev/test, taches simples | Production Azure |
| **Courbe d'apprentissage** | Moderee | Raide | Douce | Raide |

---

## Load Balancers (Repartiteurs de Charge)

### Concept

Un **load balancer** distribue le trafic reseau entre plusieurs instances de votre application. Il est essentiel pour la haute disponibilite et la scalabilite.

> [!tip] Analogie
> Un load balancer est comme l'**hotesse d'accueil** d'un restaurant :
> - Les clients (requetes) arrivent a l'entree
> - L'hotesse les repartit entre les tables disponibles (serveurs)
> - Si une table est occupee ou indisponible, elle en choisit une autre
> - Elle verifie regulierement que les tables sont prates (health checks)

### Flux de trafic avec un load balancer

```
  Utilisateurs (Internet)
       |     |     |
       v     v     v
  +--------------------+
  |   LOAD BALANCER    |  <-- Point d'entree unique (IP fixe ou DNS)
  |                    |      - SSL termination (HTTPS)
  |  Regles de routage |      - Health checks
  |  /api/* -> API     |      - Sticky sessions (optionnel)
  |  /web/* -> Frontend|
  +----+-----+----+----+
       |     |    |
       v     v    v
  +------+ +------+ +------+
  | Inst | | Inst | | Inst |
  |  #1  | |  #2  | |  #3  |
  | (OK) | | (OK) | | (KO) |  <-- Instance #3 en panne
  +------+ +------+ +------+      Le LB ne lui envoie plus de trafic
       AZ-a    AZ-b    AZ-a
```

### Types de Load Balancers (AWS)

| Type | Couche OSI | Usage | Protocoles |
|---|---|---|---|
| **Application LB (ALB)** | Couche 7 (HTTP) | APIs REST, microservices, routage par chemin/host | HTTP, HTTPS, WebSocket |
| **Network LB (NLB)** | Couche 4 (TCP/UDP) | Performance extreme, faible latence | TCP, UDP, TLS |
| **Classic LB (CLB)** | Couche 4 et 7 | Legacy (deconseille pour les nouveaux projets) | HTTP, HTTPS, TCP |

> [!info] ALB vs NLB : lequel choisir ?
> - **ALB** : pour 90% des cas d'usage web. Il comprend HTTP et peut router selon le chemin (`/api` vers un service, `/web` vers un autre), le hostname, les headers, etc.
> - **NLB** : pour les applications necessitant des performances extremes (millions de requetes/seconde), une latence ultra-faible, ou des protocoles non-HTTP (gRPC, TCP brut, jeux video).

### Health Checks (verifications de sante)

Les health checks permettent au load balancer de savoir quelles instances sont capables de traiter des requetes. Le LB envoie periodiquement une requete HTTP (ex: `GET /health`). Si la reponse est `200 OK`, l'instance est "healthy". Apres X echecs consecutifs, elle est "unhealthy" et retiree du pool.

| Parametre | Valeur typique | Description |
|---|---|---|
| Path | `/health` | Endpoint de verification |
| Interval | 30s | Frequence des verifications |
| Timeout | 5s | Temps max pour repondre |
| Healthy threshold | 3 | Succes consecutifs pour etre healthy |
| Unhealthy threshold | 2 | Echecs consecutifs pour etre unhealthy |

### SSL Termination et regles de routage

```
  HTTPS (chiffre)          HTTP (non chiffre, reseau interne)
  
  Client ---[HTTPS 443]---> ALB ---[HTTP 8000]---> Conteneurs
                              |
                         Le ALB dechiffre le SSL
                         (SSL Termination)
                         Certificat gere par ACM (AWS Certificate Manager)

  Regles de routage ALB :
  +------------------+----------------------+
  | Condition         | Action              |
  +------------------+----------------------+
  | Path = /api/*     | Forward -> API TG   |
  | Path = /admin/*   | Forward -> Admin TG |
  | Host = api.ex.com | Forward -> API TG   |
  | Default           | Forward -> Web TG   |
  +------------------+----------------------+
  TG = Target Group (groupe de cibles)
```

```bash
# Creer un ALB avec target group et listener HTTPS
aws elbv2 create-load-balancer --name mon-alb \
  --subnets subnet-xxx subnet-yyy --security-groups sg-xxx \
  --scheme internet-facing --type application

aws elbv2 create-target-group --name mon-api-tg \
  --protocol HTTP --port 8000 --vpc-id vpc-xxx --target-type ip \
  --health-check-path /health --health-check-interval-seconds 30

aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTPS --port 443 \
  --certificates CertificateArn=arn:aws:acm:... \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...
```

---

## Auto-Scaling (Mise a l'Echelle Automatique)

### Horizontal vs Vertical Scaling

| Aspect | **Vertical** (Scale Up) | **Horizontal** (Scale Out) |
|---|---|---|
| **Methode** | Plus de CPU/RAM sur une machine | Plus de machines |
| **Limite** | Taille max d'instance | Theoriquement illimite |
| **Downtime** | Souvent necessaire (redemarrage) | Aucun |
| **Resilience** | Point unique de defaillance | Si un noeud tombe, les autres continuent |
| **Cout** | Augmente exponentiellement | Augmente lineairement |

> [!warning] Privilegiez toujours le scaling horizontal
> Le scaling vertical a des limites physiques et necessite souvent un redemarrage. Le scaling horizontal est plus resilient et peut etre automatise de maniere granulaire.

### Politiques d'auto-scaling

| Type de politique | Declencheur | Exemple |
|---|---|---|
| **Target tracking** | Maintenir une metrique a une valeur cible | CPU moyen a 60% |
| **Step scaling** | Seuils avec actions differentes | CPU > 70% : +1, CPU > 90% : +3 |
| **Scheduled** | Horaire predefini | 10 instances a 9h, 3 instances a 20h |
| **Predictive** | Historique et machine learning | Anticiper les pics du lundi matin |

### Metriques d'auto-scaling

| Metrique | Seuil scale-out | Seuil scale-in | Usage |
|---|---|---|---|
| **CPU Usage** | > 70% pendant 3 min | < 30% pendant 10 min | Le plus courant |
| **Memory Usage** | > 80% pendant 3 min | < 40% pendant 10 min | Apps gourmandes en RAM |
| **Request Count** | > 1000/min par instance | < 200/min pendant 10 min | APIs a fort trafic |
| **Queue Depth** | > 100 messages | < 10 messages | Workers asynchrones |

### Configuration auto-scaling ECS

```bash
# 1. Enregistrer la cible scalable (min 2, max 20 tasks)
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --resource-id service/mon-cluster/mon-api-service \
  --min-capacity 2 --max-capacity 20

# 2. Politique target-tracking : maintenir le CPU a 60%
aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --resource-id service/mon-cluster/mon-api-service \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 60.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    },
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60
  }'
```

### Scale-to-Zero

Le **scale-to-zero** reduit le nombre d'instances a **zero** quand il n'y a aucun trafic. Vous ne payez rien quand l'application dort. Services supportant le scale-to-zero : **AWS Lambda**, **Google Cloud Run**, **Azure Container Apps**, **Kubernetes avec KEDA**.

> [!example] Cloud Run : le meilleur du serverless et des conteneurs
> **Google Cloud Run** combine la simplicite du serverless avec la flexibilite des conteneurs : deployez une image Docker, Cloud Run gere tout (scaling, HTTPS, load balancing) et scale a zero.
> ```bash
> gcloud run deploy mon-api \
>   --image gcr.io/mon-projet/mon-api:1.0.0 \
>   --platform managed --region europe-west1 --allow-unauthenticated
> ```

---

## Strategies de Deploiement

### Blue-Green Deployment

Le **deploiement blue-green** maintient deux environnements identiques. L'un (Blue) sert le trafic en production, l'autre (Green) recoit la nouvelle version. Le basculement est instantane.

```
  ETAPE 1 : Blue sert le trafic         ETAPE 2 : Deployer sur Green
  ================================       ================================

  Utilisateurs                           Utilisateurs
       |                                      |
  +----+----+                            +----+----+
  |   LB    |                            |   LB    |
  +----+----+                            +----+----+
       |                                      |
       v                                      v
  +---------+  +---------+               +---------+  +---------+
  | BLUE    |  | GREEN   |               | BLUE    |  | GREEN   |
  | v1.0    |  | (idle)  |               | v1.0    |  | v2.0    |
  | ACTIF   |  |         |               | ACTIF   |  | TEST OK |
  +---------+  +---------+               +---------+  +---------+


  ETAPE 3 : Basculer le trafic          ETAPE 4 : Blue en standby
  ================================       ================================

  Utilisateurs                           Utilisateurs
       |                                      |
  +----+----+                            +----+----+
  |   LB    | <-- Switch !               |   LB    |
  +----+----+                            +----+----+
       |                                      |
       v                                      v
  +---------+  +---------+               +---------+  +---------+
  | BLUE    |  | GREEN   |               | BLUE    |  | GREEN   |
  | v1.0    |  | v2.0    |               | v1.0    |  | v2.0    |
  | standby |  | ACTIF   |               | rollback|  | ACTIF   |
  +---------+  +---------+               +---------+  +---------+
```

**Avantages** :
- Zero-downtime : le basculement est instantane (changement de DNS ou target group)
- Rollback immediat : il suffit de rebasculer vers Blue
- Test en conditions reelles avant de basculer

**Inconvenients** :
- Double de l'infrastructure (cout temporaire)
- Les migrations de base de donnees doivent etre compatibles avec les deux versions

### Canary Deployment (Deploiement Canari)

Le **deploiement canari** envoie progressivement un pourcentage croissant du trafic vers la nouvelle version. Si des erreurs sont detectees, le rollback est automatique.

```
  Phase 1 : 5% du trafic        Phase 2 : 25% du trafic
  vers la nouvelle version       vers la nouvelle version

  100 requetes                   100 requetes
       |                              |
  +----+----+                    +----+----+
  |   LB    |                    |   LB    |
  +--+---+--+                    +--+---+--+
     |   |                          |   |
   95%   5%                       75%  25%
     |   |                          |   |
     v   v                          v   v
  +----+ +----+                  +----+ +----+
  |v1.0| |v2.0|                  |v1.0| |v2.0|
  +----+ +----+                  +----+ +----+

  Phase 3 : 100% du trafic      Probleme detecte : rollback
  (deploiement termine)          (0% vers v2.0)

  100 requetes                   100 requetes
       |                              |
  +----+----+                    +----+----+
  |   LB    |                    |   LB    |
  +----+----+                    +----+----+
       |                              |
     100%                           100%
       |                              |
       v                              v
     +----+                        +----+
     |v2.0|                        |v1.0|  v2.0 retire
     +----+                        +----+
```

### Rolling Update (Mise a jour progressive)

Le **rolling update** remplace les instances une par une (ou par lots). A chaque etape, une ancienne instance est retiree et une nouvelle est lancee.

```
  Etat initial : 4 instances v1.0
  +------+ +------+ +------+ +------+
  | v1.0 | | v1.0 | | v1.0 | | v1.0 |
  +------+ +------+ +------+ +------+

  Etape 1 : Remplacer la premiere instance
  +------+ +------+ +------+ +------+
  | v2.0 | | v1.0 | | v1.0 | | v1.0 |
  +------+ +------+ +------+ +------+

  Etape 2 : Remplacer la deuxieme instance
  +------+ +------+ +------+ +------+
  | v2.0 | | v2.0 | | v1.0 | | v1.0 |
  +------+ +------+ +------+ +------+

  Etape 3-4 : Continuer jusqu'a la fin
  +------+ +------+ +------+ +------+
  | v2.0 | | v2.0 | | v2.0 | | v2.0 |
  +------+ +------+ +------+ +------+
```

### Comparaison des strategies

| Strategie | Downtime | Rollback | Cout infra | Complexite | Risque |
|---|---|---|---|---|---|
| **Blue-Green** | Zero | Instantane | 2x temporaire | Moyenne | Faible |
| **Canary** | Zero | Progressif | Faible | Elevee | Tres faible |
| **Rolling** | Zero | Lent | Aucun surplus | Faible | Moyen |
| **Recreate** | Oui | Manuel | Aucun surplus | Faible | Eleve |

---

## CI/CD pour Conteneurs

### Pipeline type

Le pipeline CI/CD pour conteneurs suit un flux standardise :

```
  CODE COMMIT      BUILD & TEST       PUSH           DEPLOY
  ===========      ============       ====           ======

  +----------+     +----------+     +----------+     +----------+
  | git push | --> | Build    | --> | Push     | --> | Deploy   |
  | main     |     | Docker   |     | to       |     | to       |
  +----------+     | image    |     | Registry |     | ECS/GKE  |
                   | + Tests  |     +----------+     +----------+
                   +----------+           |                |
                        |                 v                v
                   Si echec :        Registry         Nouveau
                   STOP pipeline     (ECR, GCR)       deploiement
```

### Exemple GitHub Actions

```yaml
# .github/workflows/deploy.yml

name: Build and Deploy to ECS

on:
  push:
    branches: [main]

env:
  AWS_REGION: eu-west-3
  ECR_REPOSITORY: mon-api
  ECS_CLUSTER: mon-cluster
  ECS_SERVICE: mon-api-service
  CONTAINER_NAME: api

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      # 1. Recuperer le code
      - name: Checkout
        uses: actions/checkout@v4

      # 2. Configurer les credentials AWS
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      # 3. Se connecter a ECR
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      # 4. Construire et pousser l'image
      - name: Build, tag, and push image
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      # 5. Mettre a jour la task definition
      - name: Update ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}

      # 6. Deployer sur ECS
      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

> [!warning] Securite du pipeline CI/CD
> - Stockez **toujours** les credentials (AWS keys, tokens) dans les **secrets** de votre plateforme CI/CD (GitHub Secrets, GitLab CI Variables)
> - Ne hardcodez **jamais** de credentials dans le fichier de workflow
> - Utilisez des roles IAM avec le **minimum de permissions** necessaires pour le pipeline
> - Activez le scan de vulnerabilites dans le pipeline (Trivy, Snyk, ECR scanning)

---

## Monitoring des Conteneurs Deployes

### Services de monitoring par fournisseur

| Fournisseur | Service | Fonctionnalites |
|---|---|---|
| **AWS** | CloudWatch + Container Insights | Metriques CPU/RAM, logs, traces, alertes |
| **GCP** | Cloud Monitoring + Cloud Logging | Metriques, logs, dashboards, alertes |
| **Azure** | Azure Monitor + Container Insights | Metriques, logs, diagnostics, alertes |

### Metriques essentielles a surveiller

| Metrique | Seuil d'alerte | Pourquoi |
|---|---|---|
| **CPU Utilization** | > 80% | Risque de degradation de performance |
| **Memory Usage** | > 85% | Risque d'OOM (Out Of Memory) kill |
| **Error Rate (5xx)** | > 1% | Erreurs serveur anormales |
| **Response Time (p99)** | > 500ms | Experience utilisateur degradee |
| **Task/Pod Restarts** | > 3/heure | Probleme de stabilite |
| **Health Check Fails** | > 0 | Instance potentiellement defaillante |

```bash
# Creer une alarme CloudWatch sur le CPU du service ECS
aws cloudwatch put-metric-alarm \
  --alarm-name "ecs-cpu-high" \
  --metric-name CPUUtilization --namespace AWS/ECS \
  --statistic Average --period 300 --threshold 80 \
  --comparison-operator GreaterThanThreshold --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:eu-west-3:123456789:alertes \
  --dimensions Name=ClusterName,Value=mon-cluster \
               Name=ServiceName,Value=mon-api-service

# Consulter les logs des conteneurs
aws logs get-log-events \
  --log-group-name /ecs/mon-api \
  --log-stream-name "ecs/api/task-id" --limit 50
```

---

## Optimisation des Couts

### Strategies d'optimisation

| Strategie | Economie | Difficulte | Description |
|---|---|---|---|
| **Right-sizing** | 20-40% | Faible | Ajuster CPU/RAM a l'usage reel |
| **Spot/Preemptible** | 60-90% | Moyenne | Utiliser la capacite excedentaire pour les taches tolerantes |
| **Reserved capacity** | 30-60% | Faible | Engagement 1-3 ans pour les charges stables |
| **Scale-to-zero** | Variable | Faible | Ne payer que quand il y a du trafic |
| **Architecture ARM** | 20-40% | Faible | Graviton (AWS), Tau T2A (GCP) : meilleur ratio prix/perf |
| **Multi-stage builds** | 10-20% | Faible | Images plus petites = stockage et transfert moins chers |

### Right-sizing : identifier le gaspillage

```bash
# Verifier l'utilisation reelle CPU/RAM d'un service ECS
aws cloudwatch get-metric-statistics \
  --namespace AWS/ECS \
  --metric-name CPUUtilization \
  --dimensions Name=ServiceName,Value=mon-api-service \
               Name=ClusterName,Value=mon-cluster \
  --start-time 2026-04-07T00:00:00Z \
  --end-time 2026-04-14T00:00:00Z \
  --period 86400 \
  --statistics Average Maximum

# Si l'utilisation moyenne est a 15% et le max a 40%,
# vous pouvez probablement reduire les ressources allouees
```

> [!tip] Regle du right-sizing
> Visez une utilisation moyenne de **40-60%** de CPU. Trop bas (< 20%) = vous gaspillez. Trop haut (> 80%) = vous risquez des problemes de performance. Commencez petit et augmentez selon les besoins.

### Spot Instances pour les workloads tolerants

En production, combinez les modeles : **Reserved/On-Demand** pour la base stable (min capacity) et **Spot** pour le scaling supplementaire en pic (60-90% d'economie, mais interruptibles).

---

## Carte Mentale ASCII

```
                    DEPLOIEMENT CLOUD ET CONTENEURS
                                |
         +----------+-----------+-----------+-----------+
         |          |           |           |           |
     REGISTRY    DEPLOY     LOAD BAL    SCALING     CI/CD
         |          |           |           |           |
     +---+---+  +--+---+   +--+---+    +--+---+   +--+---+
     |   |   |  |  |   |   |  |   |    |  |   |   |  |   |
    Hub ECR GCR ECS GKE   ALB NLB     Horiz Vert  Build Push
     |       |  |   ACI   Health SSL  Target Step  Test Deploy
    ACR    GHCR Fargate   Routing     Sched  Pred  GitHub
                EC2                   Zero         Actions
                |
         +------+------+
         |      |      |
      Blue-   Canary Rolling
      Green
```

---

## Exercices

### Exercice 1 : Deployer sur ECS avec Fargate

1. Creez un depot ECR et poussez une image Docker (une API simple)
2. Creez un cluster ECS
3. Redigez une Task Definition pour votre API (CPU: 256, Memory: 512)
4. Creez un service avec 2 instances
5. Configurez un ALB devant le service
6. Testez l'acces via l'URL du load balancer
7. Nettoyez toutes les ressources

### Exercice 2 : Auto-scaling

1. En reutilisant le service de l'exercice 1, configurez une politique de target tracking sur le CPU (cible : 50%)
2. Utilisez un outil de charge (comme `hey` ou `ab`) pour generer du trafic
3. Observez le scaling dans la console ECS ou via la CLI
4. Arretez la charge et verifiez que le scale-in se produit apres le cooldown

### Exercice 3 : Pipeline CI/CD

1. Creez un repository GitHub avec un Dockerfile et une API simple
2. Ecrivez un workflow GitHub Actions qui :
   - Construit l'image Docker
   - La pousse sur un registry (GHCR ou ECR)
   - Deploie sur votre service ECS (ou simulez avec un echo)
3. Faites un commit, verifiez que le pipeline s'execute
4. Ajoutez une etape de tests avant le build

### Exercice 4 : Comparaison de strategies de deploiement

Sur papier ou en document, decrivez comment vous mettriez en oeuvre chaque strategie de deploiement (blue-green, canary, rolling) pour une API REST avec une base de donnees PostgreSQL. Pour chaque strategie, indiquez :
- Les etapes de deploiement
- Comment gerer les migrations de base de donnees
- Le plan de rollback en cas de probleme
- Les avantages et inconvenients dans ce contexte

---

## Liens

- [[02 - Services Cloud Essentiels]] - Compute, stockage, bases de donnees et reseau
- [[04 - Kubernetes Introduction]] - Orchestration de conteneurs avec Kubernetes
- [[02 - Docker Avance]] - Construction d'images, multi-stage builds, Docker Compose
- [[04 - CI-CD avec GitHub Actions]] - Pipelines d'integration et deploiement continus
