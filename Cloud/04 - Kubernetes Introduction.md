# Kubernetes Introduction

## Qu'est-ce que Kubernetes ?

**Kubernetes** (souvent abrege **K8s**) est une plateforme open-source d'**orchestration de conteneurs**. Elle automatise le deploiement, la mise a l'echelle et la gestion d'applications conteneurisees.

Kubernetes a ete cree par **Google** en 2014, base sur plus de 15 ans d'experience avec leur systeme interne Borg. Il est maintenant maintenu par la **CNCF** (Cloud Native Computing Foundation) et est devenu le standard de facto pour l'orchestration de conteneurs.

> [!tip] Analogie
> Imaginez un **chef d'orchestre** dirigeant un ensemble de musiciens :
> - Les **conteneurs** sont les musiciens (chacun joue sa partition)
> - **Kubernetes** est le chef d'orchestre (il coordonne tout le monde)
> - Si un musicien fait un malaise (crash), le chef fait immediatement entrer un remplacant (self-healing)
> - Si le public est plus nombreux que prevu, le chef ajoute des musiciens (scaling)
> - Le chef peut changer la partition en cours de concert sans que le public ne remarque d'interruption (rolling update)

---

## Pourquoi Kubernetes ?

### Le probleme

Sans orchestrateur, gerer 50+ conteneurs sur 10+ serveurs devient un cauchemar : quel conteneur va sur quel serveur ? Que faire quand un conteneur plante ? Comment mettre a jour sans downtime ? Kubernetes automatise tout cela.

### Les capacites de Kubernetes

| Capacite | Description |
|---|---|
| **Self-healing** | Un conteneur plante ? Kubernetes le relance automatiquement |
| **Scaling horizontal** | Ajouter/retirer des instances selon la charge |
| **Rolling updates** | Mise a jour sans interruption de service |
| **Rollback** | Retour a une version precedente en une commande |
| **Service discovery** | Les conteneurs se trouvent entre eux automatiquement (DNS interne) |
| **Load balancing** | Distribution du trafic entre les instances |
| **Configuration management** | Injection de config et secrets sans rebuild d'image |
| **Storage orchestration** | Montage automatique de volumes persistants |

> [!info] Kubernetes n'est pas toujours la reponse
> Kubernetes est puissant mais complexe. Pour une petite application avec 2-3 conteneurs, Docker Compose ou un service comme AWS ECS/Fargate est souvent plus adapte. Kubernetes brille a partir de **10+ services** ou quand vous avez besoin de portabilite multi-cloud.

---

## Architecture de Kubernetes

### Vue d'ensemble

```
  +=====================================================================+
  |                       CLUSTER KUBERNETES                             |
  |                                                                      |
  |  CONTROL PLANE (Master)                                             |
  |  +---------------------------------------------------------------+  |
  |  | +------------+ +--------+ +-----------+ +------------------+  |  |
  |  | | API Server | | etcd   | | Scheduler | | Controller Mgr   |  |  |
  |  | | (entree    | | (base  | | (place    | | (maintient       |  |  |
  |  | |  kubectl)  | |  K/V)  | |  les pods)| |  l'etat desire)  |  |  |
  |  | +------------+ +--------+ +-----------+ +------------------+  |  |
  |  +---------------------------------------------------------------+  |
  |           | kubectl / API REST                                       |
  |  WORKER NODES                                                        |
  |  +---------------------------+    +---------------------------+      |
  |  | NODE 1                    |    | NODE 2                    |      |
  |  | +------+ +------+        |    | +------+ +------+        |      |
  |  | |Pod A | |Pod B |        |    | |Pod C | |Pod D |        |      |
  |  | +------+ +------+        |    | +------+ +------+        |      |
  |  | [kubelet] [kube-proxy]   |    | [kubelet] [kube-proxy]   |      |
  |  | [containerd]             |    | [containerd]             |      |
  |  +---------------------------+    +---------------------------+      |
  +=====================================================================+
```

### Composants du Control Plane

| Composant | Role |
|---|---|
| **API Server** | Point d'entree unique pour toutes les operations. Recoit les commandes `kubectl`, valide et traite les requetes |
| **etcd** | Base de donnees cle-valeur distribuee. Stocke tout l'etat du cluster (la "source de verite") |
| **Scheduler** | Decide sur quel node placer un nouveau pod, en fonction des ressources disponibles, des contraintes et des affinites |
| **Controller Manager** | Ensemble de controleurs qui surveillent l'etat du cluster et agissent pour atteindre l'etat desire (ex: si un pod manque, en creer un nouveau) |

### Composants des Worker Nodes

| Composant | Role |
|---|---|
| **kubelet** | Agent qui tourne sur chaque node. Communique avec l'API Server et s'assure que les conteneurs des pods tournent correctement |
| **kube-proxy** | Gere les regles reseau sur chaque node. Permet la communication entre les pods et l'exposition des services |
| **Container Runtime** | Le moteur qui execute les conteneurs (containerd, CRI-O). Docker n'est plus directement supporte depuis K8s 1.24 |

> [!warning] Docker n'est plus le runtime par defaut
> Depuis Kubernetes 1.24, le support de Docker (dockershim) a ete retire. Kubernetes utilise maintenant **containerd** ou **CRI-O** comme runtime. Vos images Docker fonctionnent toujours parfaitement (elles respectent le standard OCI), seul le runtime d'execution a change.

---

## kubectl : L'outil en ligne de commande

### Installation

```bash
# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# macOS (avec Homebrew)
brew install kubectl

# Windows (avec Chocolatey)
choco install kubernetes-cli

# Verifier l'installation
kubectl version --client
```

### kubeconfig et contextes

Le fichier `~/.kube/config` contient les informations de connexion aux clusters (adresse du serveur, certificats, tokens). Un **contexte** associe un cluster, un utilisateur et un namespace par defaut.

```bash
# Lister les contextes disponibles
kubectl config get-contexts

# Changer de contexte (changer de cluster)
kubectl config use-context mon-cluster-prod

# Voir le contexte courant
kubectl config current-context
```

### Commandes essentielles

```bash
# === LIRE ===
kubectl get pods                    # Lister les pods
kubectl get pods -o wide            # Avec details (IP, node)
kubectl get all                     # Toutes les ressources
kubectl get pods -n kube-system     # Pods dans un namespace specifique
kubectl describe pod mon-pod        # Details complets d'un pod
kubectl logs mon-pod                # Logs d'un pod
kubectl logs mon-pod -f --tail=50   # Suivre les 50 dernieres lignes

# === CREER / MODIFIER ===
kubectl apply -f deployment.yaml    # Creer ou mettre a jour
kubectl apply -f ./manifests/       # Appliquer tous les YAML d'un dossier
kubectl delete -f deployment.yaml   # Supprimer via le manifeste

# === DEBUG ===
kubectl exec -it mon-pod -- /bin/bash      # Shell interactif
kubectl port-forward pod/mon-pod 8080:8000 # Acceder depuis votre poste
kubectl port-forward svc/mon-service 8080:80
```

---

## Concepts Fondamentaux

### Pods : L'unite de base

Un **Pod** est la plus petite unite deployable dans Kubernetes. Il contient **un ou plusieurs conteneurs** qui partagent le reseau et le stockage.

```
  POD SIMPLE (cas le plus courant)     POD MULTI-CONTENEURS (sidecar)

  +-------------------+                +----------------------------+
  | Pod               |                | Pod                        |
  |                   |                |                            |
  | +---------------+ |                | +----------+ +----------+ |
  | | Conteneur API | |                | | Conteneur| | Sidecar  | |
  | | Port 8000     | |                | | API      | | (logs,   | |
  | +---------------+ |                | | Port 8000| | proxy,   | |
  |                   |                | +----------+ | metrics) | |
  | IP: 10.244.1.5   |                | +----------+ +----------+ |
  +-------------------+                |                            |
                                       | IP: 10.244.1.6            |
                                       | (IP partagee entre        |
                                       |  les conteneurs du pod)   |
                                       +----------------------------+
```

> [!info] Pourquoi des pods multi-conteneurs ?
> Le **pattern sidecar** est courant : un conteneur principal (votre API) accompagne d'un conteneur auxiliaire qui ajoute une fonctionnalite transverse (collecte de logs, proxy reseau, cache local). Les conteneurs dans un meme pod communiquent via `localhost` et partagent les volumes.

> [!warning] Ne deployez jamais des pods directement
> En production, n'utilisez **jamais** `kind: Pod` seul. Utilisez un **Deployment** qui gere les pods pour vous (replicas, rolling updates, rollback). Un pod seul ne sera pas relance en cas de crash ou de suppression du node.

### ReplicaSets et Self-Healing

Un **ReplicaSet** s'assure qu'un nombre defini de pods identiques est toujours en cours d'execution. Si un pod disparait, il en cree un nouveau automatiquement. En pratique, vous ne creez **jamais** un ReplicaSet directement : vous creez un **Deployment** qui gere les ReplicaSets pour vous.

```
  ReplicaSet : desired = 3

  Etat normal :                      Un pod plante -> auto-recreation :
  +------+ +------+ +------+        +------+ +------+ +------+
  | Pod  | | Pod  | | Pod  |  --->  | Pod  | | Pod  | | NEW  |
  | (OK) | | (OK) | | (OK) |        | (OK) | | (OK) | | (OK) |
  +------+ +------+ +------+        +------+ +------+ +------+
```

### Deployments : Le controleur principal

Un **Deployment** est la ressource la plus utilisee dans Kubernetes. Il declare l'etat desire de votre application et Kubernetes s'assure de l'atteindre.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mon-api                    # Nom du deployment
  labels:
    app: mon-api                   # Labels pour identifier cette ressource
spec:
  replicas: 3                      # Nombre d'instances souhaitees
  
  selector:                        # Comment le Deployment trouve ses pods
    matchLabels:
      app: mon-api                 # Doit matcher les labels du template
  
  strategy:                        # Strategie de mise a jour
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1                  # Max 1 pod en plus pendant la MAJ
      maxUnavailable: 0            # Aucun pod indisponible pendant la MAJ
  
  template:                        # Template du pod (ce qui sera cree)
    metadata:
      labels:
        app: mon-api               # Labels du pod (doit matcher selector)
        version: "1.0.0"
    spec:
      containers:
        - name: api
          image: mon-registry/mon-api:1.0.0
          ports:
            - containerPort: 8000
          
          # Ressources (important pour le scheduling et l'auto-scaling)
          resources:
            requests:              # Minimum garanti
              cpu: "100m"
              memory: "128Mi"
            limits:                # Maximum autorise
              cpu: "500m"
              memory: "256Mi"
          
          # Variables d'environnement
          env:
            - name: ENV
              value: "production"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: url
          
          # Health checks (voir section dediee)
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 10
            periodSeconds: 30
          
          readinessProbe:
            httpGet:
              path: /ready
              port: 8000
            initialDelaySeconds: 5
            periodSeconds: 10
```

### Operations courantes sur les Deployments

```bash
kubectl apply -f deployment.yaml                   # Creer / mettre a jour
kubectl rollout status deployment/mon-api          # Statut du deploiement
kubectl rollout history deployment/mon-api         # Historique des versions
kubectl rollout undo deployment/mon-api            # Rollback version precedente
kubectl rollout undo deployment/mon-api --to-revision=2  # Rollback specifique
kubectl set image deployment/mon-api api=mon-api:2.0.0   # MAJ image
kubectl scale deployment/mon-api --replicas=5      # Scaler manuellement
```

---

## Services : Exposer les Applications

Les pods sont **ephemeres** : leur IP change a chaque recreation. Un **Service** est un point d'acces stable (IP fixe + DNS interne) vers un ensemble de pods, resolvant ce probleme.

### Types de Services

```
  ClusterIP (defaut)              NodePort                   LoadBalancer
  Accessible seulement            Accessible via             Accessible via
  depuis le cluster               IP du node + port          IP publique (cloud)

  +---Cluster---------+          +---Cluster---------+      +---Cloud----------+
  |                    |          |                    |      |                   |
  | Pod A --> ClusterIP|          | :30080 sur chaque |      | +---LB (public)--+|
  |          10.96.1.1 |          | node               |      | | IP: 34.56.78.90||
  |             |      |          |     |              |      | +-------+--------+|
  |          +--+--+   |          |  +--+--+           |      |         |         |
  |          | Pods|   |          |  | Pods|           |      |      +--+--+      |
  |          +-----+   |          |  +-----+           |      |      | Pods|      |
  +--------------------+          +--------------------+      |      +-----+      |
                                                              +-------------------+
  Usage : communication           Usage : dev/test,           Usage : production,
  interne entre services          acces direct aux nodes      exposition publique
```

```yaml
# service-loadbalancer.yaml - Exemple complet
apiVersion: v1
kind: Service
metadata:
  name: mon-api-public
spec:
  type: LoadBalancer         # Provisionne un LB cloud automatiquement
  selector:
    app: mon-api             # Selectionne les pods avec ce label
  ports:
    - port: 80               # Port du service
      targetPort: 8000       # Port du conteneur
```

```bash
# DNS automatique entre services (dans le meme namespace)
# curl http://mon-api          -> ClusterIP du service mon-api
# curl http://mon-api.staging  -> Service dans le namespace staging
# Format complet : <service>.<namespace>.svc.cluster.local
```

Le type **ExternalName** permet de creer un alias DNS vers un service externe (ex: `externalName: ma-base.abc123.rds.amazonaws.com`).

---

## Namespaces : Isolation Logique

Un **Namespace** isole logiquement les ressources dans un cluster. Cas d'usage : par environnement (`dev`, `staging`, `prod`), par equipe ou par application.

```bash
kubectl get namespaces                     # Lister (default, kube-system, etc.)
kubectl create namespace staging           # Creer un namespace
kubectl apply -f deployment.yaml -n staging  # Deployer dans un namespace
kubectl config set-context --current --namespace=staging  # Changer le defaut
```

---

## ConfigMaps et Secrets

### ConfigMaps : Configuration non-sensible

```bash
# Creer un ConfigMap depuis des valeurs literales ou un fichier
kubectl create configmap app-config \
  --from-literal=ENV=production \
  --from-literal=LOG_LEVEL=info
kubectl create configmap nginx-config --from-file=nginx.conf
```

### Secrets : Donnees sensibles

```bash
# Creer un secret
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=MonMotDePasse123
```

> [!warning] Les Secrets Kubernetes ne sont PAS chiffres par defaut
> Les Secrets sont encodes en **base64** (pas de chiffrement). Quiconque a acces au cluster peut les lire. Pour une vraie securite :
> - Activez le **chiffrement au repos** (encryption at rest) dans etcd
> - Utilisez un **gestionnaire de secrets externe** (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager)
> - Limitez l'acces avec RBAC (Role-Based Access Control)

### Injection dans les pods

Trois methodes pour injecter ConfigMaps/Secrets dans un pod :

```yaml
spec:
  containers:
    - name: api
      # Methode 1 : Variable d'env individuelle
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
      # Methode 2 : Toutes les cles en variables d'env
      envFrom:
        - configMapRef:
            name: app-config
      # Methode 3 : Monter comme fichier dans /etc/config
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

---

## Persistent Volumes et Claims

Par defaut, les donnees d'un conteneur sont perdues quand il est detruit. Pour les bases de donnees, il faut du **stockage persistant** via PersistentVolumes (PV) et PersistentVolumeClaims (PVC).

```
  StorageClass (gp2)    auto-provision    PV (cree auto)    mount    Pod
  +---------+         ===============>   +---------+      ======>  +------+
  | Defini  |                            | Volume  |               | /data|
  | par     |  <--- PVC (50 Go, SSD) ---|  reel   |               |      |
  | admin   |       cree par le dev      | (EBS)   |               +------+
  +---------+                            +---------+
```

Avec le **provisionning dynamique** (StorageClass), le developpeur cree un PVC et Kubernetes provisionne automatiquement le volume sous-jacent.

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
spec:
  accessModes:
    - ReadWriteOnce          # Un seul node peut monter en ecriture
  storageClassName: gp2      # StorageClass (varie selon le cloud)
  resources:
    requests:
      storage: 10Gi
```

---

## Exemple Complet : API Python + PostgreSQL

### Architecture cible

```
  +---Cluster Kubernetes--------------------------------+
  |                                                      |
  |  +-----------+      +-----------+      +-----------+|
  |  | Service   |      | Service   |      | ConfigMap ||
  |  | mon-api   |      | postgres  |      | app-config||
  |  | (LB)      |      | (ClusterIP)     | + Secret  ||
  |  +-----+-----+      +-----+-----+      +-----+-----+|
  |        |                   |                  |      |
  |  +-----+-----+      +-----+-----+            |      |
  |  | Deployment |      | Deployment |           |      |
  |  | mon-api    |      | postgres   |           |      |
  |  | replicas:3 |      | replicas:1 |           |      |
  |  +-----+-----+      +-----+-----+            |      |
  |        |                   |                  |      |
  |  +---+-+--+--+      +-----+-----+            |      |
  |  |Pod|Pod|Pod|      |   Pod     |            |      |
  |  |   |   |   |      |  [PG 16] |            |      |
  |  +---+---+---+      +-----+-----+            |      |
  |                            |                  |      |
  |                      +-----+-----+            |      |
  |                      |    PVC    |            |      |
  |                      | 10 Go SSD |            |      |
  |                      +-----------+            |      |
  +----------------------------------------------+------+
```

### Manifestes YAML

```yaml
# all-in-one.yaml (namespace: mon-app)
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: mon-app
data:
  ENV: "production"
  DB_HOST: "postgres"
  DB_PORT: "5432"
  DB_NAME: "monapp"
---
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: mon-app
type: Opaque
stringData:                    # stringData = pas besoin d'encoder en base64
  DB_USER: "appuser"
  DB_PASSWORD: "S3cur3P@ssw0rd!"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
  namespace: mon-app
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: gp2
  resources:
    requests:
      storage: 10Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: mon-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16-alpine
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_DB
              valueFrom:
                configMapKeyRef: { name: app-config, key: DB_NAME }
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef: { name: db-credentials, key: DB_USER }
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef: { name: db-credentials, key: DB_PASSWORD }
          volumeMounts:
            - name: pg-storage
              mountPath: /var/lib/postgresql/data
          resources:
            requests: { cpu: "250m", memory: "256Mi" }
            limits:   { cpu: "1000m", memory: "512Mi" }
      volumes:
        - name: pg-storage
          persistentVolumeClaim:
            claimName: postgres-data
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: mon-app
spec:
  type: ClusterIP
  selector: { app: postgres }
  ports:
    - port: 5432
      targetPort: 5432
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mon-api
  namespace: mon-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mon-api
  strategy:
    type: RollingUpdate
    rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }
  template:
    metadata:
      labels:
        app: mon-api
    spec:
      containers:
        - name: api
          image: mon-registry/mon-api:1.0.0
          ports:
            - containerPort: 8000
          envFrom:
            - configMapRef: { name: app-config }
            - secretRef: { name: db-credentials }
          livenessProbe:
            httpGet: { path: /health, port: 8000 }
            initialDelaySeconds: 15
            periodSeconds: 30
          readinessProbe:
            httpGet: { path: /ready, port: 8000 }
            initialDelaySeconds: 5
            periodSeconds: 10
          resources:
            requests: { cpu: "100m", memory: "128Mi" }
            limits:   { cpu: "500m", memory: "256Mi" }
---
apiVersion: v1
kind: Service
metadata:
  name: mon-api
  namespace: mon-app
spec:
  type: LoadBalancer
  selector: { app: mon-api }
  ports:
    - port: 80
      targetPort: 8000
```

```bash
# Deployer l'ensemble
kubectl create namespace mon-app
kubectl apply -f all-in-one.yaml

# Verifier le deploiement
kubectl get all -n mon-app
kubectl get pvc -n mon-app

# Obtenir l'URL du LoadBalancer
kubectl get svc mon-api -n mon-app \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

---

## Scaling et Auto-Scaling

### Scaling manuel

```bash
# Augmenter le nombre de replicas
kubectl scale deployment/mon-api --replicas=5 -n mon-app

# Verifier
kubectl get pods -n mon-app -l app=mon-api
# NAME                       READY   STATUS    AGE
# mon-api-7d9f8b6c4-abc12   1/1     Running   2m
# mon-api-7d9f8b6c4-def34   1/1     Running   2m
# mon-api-7d9f8b6c4-ghi56   1/1     Running   2m
# mon-api-7d9f8b6c4-jkl78   1/1     Running   30s
# mon-api-7d9f8b6c4-mno90   1/1     Running   30s
```

### HorizontalPodAutoscaler (HPA)

Le **HPA** ajuste automatiquement le nombre de replicas en fonction des metriques (CPU, memoire, metriques custom).

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: mon-api-hpa
  namespace: mon-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mon-api
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60   # Cible : 60% de CPU moyen
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 75
```

```bash
kubectl apply -f hpa.yaml
kubectl get hpa -n mon-app
# NAME          REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS
# mon-api-hpa   Deployment/mon-api  23%/60%   2         20        3
```

> [!info] Pre-requis pour le HPA
> Le HPA necessite le **Metrics Server** (pre-installe sur EKS, GKE, AKS ; sur minikube : `minikube addons enable metrics-server`). Les pods doivent avoir des `resources.requests` definis pour que le calcul de pourcentage fonctionne.

---

## Health Checks (Sondes de Sante)

### Les trois types de probes

| Probe | Question posee | Si echec |
|---|---|---|
| **livenessProbe** | "Le conteneur est-il vivant ?" | Kubernetes **redmarre** le conteneur |
| **readinessProbe** | "Le conteneur est-il pret a recevoir du trafic ?" | Le pod est **retire du Service** (plus de trafic) |
| **startupProbe** | "Le conteneur a-t-il fini de demarrer ?" | Les autres probes sont **desactivees** jusqu'au succes |

```yaml
# Exemple complet de health checks
spec:
  containers:
    - name: api
      image: mon-api:1.0.0
      startupProbe:            # L'app a-t-elle fini de demarrer ?
        httpGet: { path: /health, port: 8000 }
        failureThreshold: 30   # 30 x 2s = 60s max pour demarrer
        periodSeconds: 2
      livenessProbe:           # L'app est-elle vivante ?
        httpGet: { path: /health, port: 8000 }
        periodSeconds: 30
        failureThreshold: 3    # 3 echecs = restart du conteneur
      readinessProbe:          # L'app peut-elle recevoir du trafic ?
        httpGet: { path: /ready, port: 8000 }
        periodSeconds: 10
        failureThreshold: 3    # 3 echecs = retire du service
```

> [!tip] Differenciez /health et /ready
> - `/health` (liveness) : verifie que le processus est vivant. Ne pas verifier les dependances ici (sinon un crash de la base relance tous les pods inutilement).
> - `/ready` (readiness) : verifie que l'application peut reellement traiter des requetes (connexion a la base, au cache, etc.). Si la base est down, le pod est retire du service mais pas redemarre.

---

## Helm : Le Gestionnaire de Packages

### Concept

**Helm** est le gestionnaire de packages de Kubernetes. Il permet de packager, versionner et deployer des applications complexes sous forme de **charts** (packages).

Un chart Helm contient : `Chart.yaml` (metadonnees), `values.yaml` (configuration par defaut), `templates/` (manifestes YAML avec variables Go).

### Commandes Helm essentielles

```bash
# Installer Helm : https://helm.sh/docs/intro/install/
brew install helm              # macOS
choco install kubernetes-helm  # Windows

# Ajouter un repo et installer un chart
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install ma-base bitnami/postgresql \
  --namespace mon-app \
  --set auth.postgresPassword=MonMotDePasse

# Installer un chart local avec des valeurs custom
helm install mon-app ./mon-chart --values custom-values.yaml -n mon-app

# Gestion des releases
helm list -n mon-app                              # Lister
helm upgrade mon-app ./mon-chart -n mon-app       # Mettre a jour
helm rollback mon-app 1 -n mon-app                # Rollback
helm uninstall mon-app -n mon-app                 # Desinstaller
```

### Exemple de values.yaml

```yaml
replicaCount: 3
image:
  repository: mon-registry/mon-api
  tag: "1.0.0"
service:
  type: LoadBalancer
  port: 80
resources:
  requests: { cpu: "100m", memory: "128Mi" }
  limits:   { cpu: "500m", memory: "256Mi" }
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 20
  targetCPUUtilization: 60
```

---

## Kubernetes Manage : EKS, GKE, AKS

### Comparaison

| Caracteristique | **EKS** (AWS) | **GKE** (GCP) | **AKS** (Azure) |
|---|---|---|---|
| **Cout control plane** | ~73$/mois | Gratuit (Autopilot) ou ~73$/mois | Gratuit |
| **Mise a jour K8s** | Manuelle (assistee) | Automatique (Autopilot) | Semi-automatique |
| **Networking** | VPC-native, Calico | VPC-native, Calico, Dataplane V2 | Azure CNI, Kubenet |
| **Stockage** | EBS, EFS | Persistent Disk, Filestore | Azure Disk, Azure Files |
| **Registry integre** | ECR | Artifact Registry | ACR |
| **Monitoring** | CloudWatch + Container Insights | Cloud Monitoring (integre) | Azure Monitor |
| **Particularite** | Integration IAM, Fargate profile | Autopilot (serverless nodes) | Integration Azure AD |

```bash
# EKS (AWS)
eksctl create cluster --name mon-cluster --region eu-west-3 \
  --nodegroup-name workers --node-type t3.medium --nodes 3

# GKE (GCP)
gcloud container clusters create mon-cluster --zone europe-west1-b \
  --num-nodes 3 --machine-type e2-medium \
  --enable-autoscaling --min-nodes 1 --max-nodes 10

# AKS (Azure)
az aks create --resource-group mon-groupe --name mon-cluster \
  --node-count 3 --node-vm-size Standard_B2s \
  --enable-cluster-autoscaler --min-count 1 --max-count 10
```

> [!tip] Conseil pour debuter
> **GKE Autopilot** est le moyen le plus simple de commencer : pas de nodes a gerer, couts optimises, control plane gratuit. Pour AWS, **EKS avec Fargate profiles** offre une experience similaire.

---

## Kubernetes vs Docker Compose

### Quand utiliser quoi ?

| Critere | **Docker Compose** | **Kubernetes** |
|---|---|---|
| **Complexite** | Faible (1 jour a apprendre) | Elevee (1-3 mois) |
| **Nombre de services** | 1-10 | 10-1000+ |
| **Environnement** | Dev local, CI, petit projet | Staging, production |
| **Scaling** | Limite | Natif, automatique (HPA) |
| **Self-healing** | Non | Oui (liveness/readiness) |
| **Multi-machine** | Non (un seul host) | Oui (cluster de nodes) |
| **Monitoring** | Manuel | Integre (metrics, logs) |

> [!example] Progression typique d'un projet
> 1. **Debut** : `docker compose up` sur le laptop du developpeur
> 2. **MVP** : deploiement sur un seul serveur avec Docker Compose
> 3. **Croissance** : migration vers ECS Fargate ou Cloud Run (simplicite)
> 4. **Scale** : migration vers Kubernetes quand la complexite le justifie (10+ services, multi-cloud)

---

## Developpement Local avec Kubernetes

### Options pour apprendre et developper

| Outil | Description | Ressources requises | Cas d'usage |
|---|---|---|---|
| **minikube** | Cluster K8s local dans une VM ou conteneur | 2 CPU, 2 Go RAM | Apprentissage, tests |
| **kind** | K8s dans Docker (Kubernetes IN Docker) | 2 CPU, 2 Go RAM | CI/CD, tests rapides |
| **k3s** | K8s leger (Rancher) | 512 Mo RAM | Edge, IoT, dev |
| **Docker Desktop** | K8s integre a Docker Desktop | 4 CPU, 4 Go RAM | Dev quotidien (Mac/Win) |

```bash
# === MINIKUBE (recommande pour debuter) ===
brew install minikube              # macOS / choco install minikube (Windows)
minikube start --cpus 2 --memory 4096
minikube addons enable metrics-server
minikube addons enable dashboard
minikube dashboard                 # Interface graphique
minikube tunnel                    # Acceder aux services LoadBalancer
minikube stop && minikube delete   # Nettoyer

# === KIND (ideal pour CI/CD) ===
brew install kind
kind create cluster --name mon-cluster
kind load docker-image mon-api:1.0.0 --name mon-cluster  # Image locale
kind delete cluster --name mon-cluster

# === K3S (Kubernetes leger, Linux) ===
curl -sfL https://get.k3s.io | sh -
sudo k3s kubectl get nodes
```

> [!tip] Recommandation pour debuter
> Commencez avec **minikube** pour apprendre (dashboard graphique, bonne documentation). Passez a **kind** pour les tests automatises (CI/CD) car il est plus rapide a demarrer/arreter.

---

## Parcours d'Apprentissage Kubernetes

| Niveau | Duree | Sujets |
|---|---|---|
| **1. Bases** | 2-4 semaines | minikube, Pods, Deployments, Services, kubectl |
| **2. Intermediate** | 1-2 mois | ConfigMaps, Secrets, PV, Namespaces, Health checks, HPA, Helm |
| **3. Production** | 2-4 mois | Ingress, RBAC, Network Policies, EKS/GKE/AKS, CI/CD, Prometheus + Grafana |
| **4. Expert** | continu | Operators, CRDs, Service Mesh (Istio), GitOps (ArgoCD), securite avancee |

---

## Carte Mentale ASCII

```
                           KUBERNETES (K8s)
                                 |
         +-----------+-----------+-----------+-----------+
         |           |           |           |           |
    ARCHITECTURE  WORKLOADS   RESEAU      CONFIG      OUTILS
         |           |           |           |           |
     +---+---+   +--+---+   +--+---+   +--+---+   +--+---+
     |   |   |   |  |   |   |  |   |   |  |   |   |  |   |
    API etcd     Pod Repl  Svc Ingr  CfgMap Sec  Helm  kubectl
    Srv Sched    Dep  Set  ClIP NP   PV  PVC    Chart  minikube
    Ctrl         HPA       NodeP                vals   kind
    Mgr                    LB                   repo   k3s

    MANAGED K8S :  EKS (AWS)  |  GKE (GCP)  |  AKS (Azure)
```

---

## Exercices

### Exercice 1 : Premiers pas avec minikube

1. Installez minikube et kubectl
2. Demarrez un cluster avec `minikube start`
3. Deployez un pod Nginx : `kubectl run nginx --image=nginx:latest`
4. Exposez-le : `kubectl expose pod nginx --port=80 --type=NodePort`
5. Accedez-y via `minikube service nginx`
6. Consultez les logs : `kubectl logs nginx`
7. Entrez dans le pod : `kubectl exec -it nginx -- /bin/bash`
8. Supprimez tout et arretez le cluster

### Exercice 2 : Deploiement complet

1. Ecrivez un Deployment YAML pour une API Python (FastAPI ou Flask) avec 3 replicas
2. Ajoutez un Service de type LoadBalancer
3. Ajoutez un ConfigMap avec des variables d'environnement
4. Ajoutez des health checks (liveness et readiness)
5. Deployez sur minikube et verifiez que tout fonctionne
6. Changez l'image (simulez une mise a jour) et observez le rolling update
7. Faites un rollback avec `kubectl rollout undo`

### Exercice 3 : Application multi-tier

En vous inspirant de l'exemple complet de cette note :
1. Deployez PostgreSQL avec un PVC pour le stockage persistant
2. Deployez votre API connectee a PostgreSQL via un Service ClusterIP
3. Creez un ConfigMap et un Secret pour la configuration de la base
4. Ajoutez un HPA qui scale l'API entre 2 et 10 pods
5. Verifiez que les donnees persistent apres un redemarrage du pod PostgreSQL

### Exercice 4 : Helm

1. Installez Helm
2. Deployez PostgreSQL avec le chart Bitnami : `helm install db bitnami/postgresql`
3. Explorez les valeurs configurables : `helm show values bitnami/postgresql`
4. Mettez a jour avec des valeurs custom (taille du PVC, mot de passe)
5. Faites un rollback avec `helm rollback`
6. Desinstallez proprement

---

## Liens

- [[03 - Deploiement Cloud et Conteneurs]] - Registries, ECS, load balancers, auto-scaling
- [[02 - Docker Avance]] - Construction d'images, multi-stage builds, Docker Compose
- [[04 - CI-CD avec GitHub Actions]] - Pipelines d'integration et deploiement continus
- [[01 - Introduction au Cloud]] - Concepts fondamentaux du cloud computing
