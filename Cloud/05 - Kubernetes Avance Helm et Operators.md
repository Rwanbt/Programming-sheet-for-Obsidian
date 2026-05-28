# 05 - Kubernetes Avancé — Helm et Operators

> [!info] Objectifs
> Aller au-delà des ressources K8s de base : gérer des applications complexes avec Helm, déployer des workloads stateful, automatiser avec des Operators, sécuriser avec RBAC et NetworkPolicy, et mettre en place GitOps.

## 1. Rappel K8s — Socle nécessaire

Avant d'aborder le niveau avancé, les ressources suivantes doivent être maîtrisées :
- **Pod** : unité atomique, un ou plusieurs containers
- **Deployment** : gestion des pods stateless avec rolling updates
- **Service** : exposition réseau stable (ClusterIP, NodePort, LoadBalancer)
- **ConfigMap / Secret** : configuration et secrets externalisés
- **Namespace** : isolation logique des ressources

Voir [[04 - Kubernetes Introduction]] pour ces bases.

---

## 2. Helm — Le gestionnaire de packages Kubernetes

### Concept

Helm est le "apt-get" de Kubernetes. Un **chart** Helm est un paquet réutilisable qui décrit une application K8s (templates de manifests + valeurs configurables).

```
Sans Helm : appliquer manuellement 15 fichiers YAML interdépendants
Avec Helm : helm install my-app bitnami/postgresql --set auth.password=secret
```

### Structure d'un chart

```
my-chart/
├── Chart.yaml          # Métadonnées (nom, version, description, dépendances)
├── values.yaml         # Valeurs par défaut configurables
├── charts/             # Charts de dépendances
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl    # Fonctions réutilisables (partials)
│   └── NOTES.txt       # Message post-installation
└── .helmignore
```

**Chart.yaml** :
```yaml
apiVersion: v2
name: my-webapp
description: A web application with Redis cache
type: application      # application ou library
version: 0.3.1         # Version du chart (SemVer)
appVersion: "2.1.0"    # Version de l'application déployée
dependencies:
  - name: redis
    version: "17.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
```

### Templates Helm

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-webapp.fullname" . }}
  labels:
    {{- include "my-webapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-webapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-webapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          ports:
            - containerPort: {{ .Values.service.port }}
          env:
            {{- if .Values.redis.enabled }}
            - name: REDIS_URL
              value: "redis://{{ include "my-webapp.fullname" . }}-redis:6379"
            {{- end }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

```yaml
# values.yaml
replicaCount: 2

image:
  repository: my-registry/my-webapp
  tag: ""    # Override pour forcer une version spécifique

service:
  type: ClusterIP
  port: 8080

ingress:
  enabled: false
  hostname: myapp.example.com

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

redis:
  enabled: true
  auth:
    password: ""   # À surcharger dans prod
```

### Commandes Helm essentielles

```bash
# Ajouter un repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable
helm repo update  # Mettre à jour l'index local

# Rechercher un chart
helm search repo postgresql
helm search hub nginx  # Recherche sur ArtifactHub

# Installer / mettre à jour
helm install my-release my-chart/ -f custom-values.yaml
helm install postgres bitnami/postgresql \
    --namespace databases \
    --create-namespace \
    --set auth.postgresPassword=mysecret \
    --set primary.persistence.size=20Gi

helm upgrade my-release my-chart/ --set image.tag=v1.2.0
helm upgrade --install my-release my-chart/  # Install si absent, upgrade sinon

# Rollback
helm rollback my-release 2   # Revenir à la révision 2
helm history my-release       # Voir l'historique des révisions

# Debug et validation
helm template my-release my-chart/ -f values.yaml  # Rendre les templates sans deployer
helm lint my-chart/                                  # Valider la syntaxe
helm diff upgrade my-release my-chart/  # (plugin helm-diff)

# Désinstaller
helm uninstall my-release
helm uninstall my-release --keep-history  # Conserver l'historique

# Packaging et distribution
helm package my-chart/               # Crée my-chart-0.3.1.tgz
helm push my-chart-0.3.1.tgz oci://my-registry/charts/
```

### Helmfile — Multi-releases déclaratif

```yaml
# helmfile.yaml
repositories:
  - name: bitnami
    url: https://charts.bitnami.com/bitnami

releases:
  - name: postgresql
    namespace: databases
    chart: bitnami/postgresql
    version: 12.x.x
    values:
      - values/postgresql-prod.yaml

  - name: redis
    namespace: databases
    chart: bitnami/redis
    values:
      - values/redis-prod.yaml

  - name: my-webapp
    namespace: production
    chart: ./charts/my-webapp
    needs:
      - databases/postgresql
      - databases/redis
    values:
      - values/webapp-prod.yaml
```

```bash
helmfile sync          # Appliquer toutes les releases
helmfile diff          # Voir les changements
helmfile destroy       # Supprimer toutes les releases
```

---

## 3. StatefulSets — Applications avec état

### Pourquoi pas un Deployment ?

Les Deployments créent des pods **interchangeables**. Pour les bases de données, les queues ou tout système stateful, on a besoin de :
- **Identités stables** : pod-0, pod-1, pod-2 (pas pod-abc123)
- **Ordre de démarrage/arrêt garanti** : pod-0 démarre avant pod-1
- **Stockage persistant individuel** : chaque pod a son propre PVC

### Exemple : PostgreSQL HA avec StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql
spec:
  serviceName: postgresql-headless    # Headless service (DNS direct vers chaque pod)
  replicas: 3
  selector:
    matchLabels:
      app: postgresql
  template:
    metadata:
      labels:
        app: postgresql
    spec:
      initContainers:
        - name: init-replica
          image: postgres:15
          command: ["/bin/bash", "-c"]
          args:
            - |
              if [ "$HOSTNAME" != "postgresql-0" ]; then
                pg_basebackup -h postgresql-0.postgresql-headless -U repl -D /data/pgdata
              fi
      containers:
        - name: postgresql
          image: postgres:15
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            - name: PGDATA
              value: /data/pgdata
          volumeMounts:
            - name: data
              mountPath: /data
          ports:
            - containerPort: 5432
  volumeClaimTemplates:              # Un PVC créé automatiquement par pod
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: gp2
        resources:
          requests:
            storage: 100Gi
---
# Headless Service (pas de ClusterIP — DNS résout directement vers les pods)
apiVersion: v1
kind: Service
metadata:
  name: postgresql-headless
spec:
  clusterIP: None          # Headless !
  selector:
    app: postgresql
  ports:
    - port: 5432
```

**DNS stable** : `postgresql-0.postgresql-headless.namespace.svc.cluster.local`
Même si le pod redémarre → même hostname → même stockage via PVC.

---

## 4. DaemonSets — Un pod par nœud

```yaml
# Déployer un agent de monitoring sur tous les nœuds
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true            # Accès au réseau du nœud
      hostPID: true                # Accès aux processus du nœud
      tolerations:                 # Déployer aussi sur les nœuds master (tainted)
        - key: node-role.kubernetes.io/control-plane
          effect: NoSchedule
      containers:
        - name: node-exporter
          image: prom/node-exporter:latest
          securityContext:
            privileged: true
          volumeMounts:
            - name: proc
              mountPath: /host/proc
              readOnly: true
            - name: sys
              mountPath: /host/sys
              readOnly: true
      volumes:
        - name: proc
          hostPath:
            path: /proc
        - name: sys
          hostPath:
            path: /sys
```

**Use cases DaemonSet** : agents de monitoring (Prometheus node-exporter), collecteurs de logs (Fluentd, Filebeat), plugins CNI réseau (Calico, Cilium), drivers GPU.

---

## 5. Jobs et CronJobs

```yaml
# Job one-shot : migration de base de données
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  parallelism: 1          # Pods à lancer en parallèle
  completions: 1          # Succès requis pour considérer le job terminé
  backoffLimit: 3         # Nombre de retries avant échec
  activeDeadlineSeconds: 600  # Timeout global
  template:
    spec:
      restartPolicy: OnFailure  # Never ou OnFailure (pas Always)
      containers:
        - name: migration
          image: my-app:latest
          command: ["python", "manage.py", "migrate"]
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: database-url
---
# CronJob : rapport quotidien
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-report
spec:
  schedule: "0 6 * * *"          # Tous les jours à 6h UTC
  concurrencyPolicy: Forbid       # Ne pas lancer si le précédent tourne encore
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: reporter
              image: my-reporter:latest
              command: ["python", "generate_report.py"]
```

---

## 6. Persistent Volumes — Stockage persistant

### PV / PVC / StorageClass

```
StorageClass (admin cluster) → PV (volume physique) → PVC (demande de volume) → Pod
```

```yaml
# StorageClass (dynamic provisioning AWS)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3-encrypted
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
  throughput: "125"
reclaimPolicy: Retain   # Retain, Delete, Recycle
volumeBindingMode: WaitForFirstConsumer  # Créer le volume dans la même AZ que le pod
---
# PVC (demande par le développeur)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-data
spec:
  accessModes:
    - ReadWriteOnce     # RWO: un nœud; ROX: lecture multi-nœuds; RWX: écriture multi-nœuds
  storageClassName: gp3-encrypted
  resources:
    requests:
      storage: 50Gi
---
# Utilisation dans un Pod
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-data
```

### Access Modes

| Mode | Description | Typique |
|------|-----------|--------|
| `ReadWriteOnce` (RWO) | Un seul nœud en lecture/écriture | AWS EBS, GCP PD |
| `ReadOnlyMany` (ROX) | Plusieurs nœuds en lecture seule | NFS, CephFS |
| `ReadWriteMany` (RWX) | Plusieurs nœuds en lecture/écriture | NFS, CephFS, AWS EFS |

---

## 7. NetworkPolicy — Pare-feu Kubernetes

```yaml
# Stratégie : deny all par défaut, puis allow explicitement
---
# Bloquer tout le trafic entrant sur le namespace production
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}    # Sélectionne tous les pods
  policyTypes:
    - Ingress

---
# Autoriser uniquement l'accès depuis le frontend vers l'API
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api              # Règle appliquée aux pods API
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend   # Seulement depuis les pods frontend
        - namespaceSelector:
            matchLabels:
              name: monitoring  # Et depuis le namespace monitoring (Prometheus)
      ports:
        - protocol: TCP
          port: 8080
```

> [!warning] CNI requis
> Les NetworkPolicy ne fonctionnent que si le CNI plugin les supporte : **Calico**, **Cilium** (le plus puissant), **Weave**. Flannel et kube-proxy seuls NE supportent PAS les NetworkPolicy.

---

## 8. RBAC — Contrôle d'accès basé sur les rôles

```yaml
# Role (namespace-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: staging
rules:
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "watch", "update", "patch"]
  - apiGroups: [""]
    resources: ["pods", "pods/logs"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list", "watch", "create", "update"]
---
# RoleBinding : lier le Role à un utilisateur ou ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: staging
subjects:
  - kind: User
    name: alice
    apiGroup: rbac.authorization.k8s.io
  - kind: ServiceAccount
    name: ci-bot
    namespace: ci
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
---
# ClusterRole (cluster-wide) + ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
```

```bash
# Vérifier les permissions
kubectl auth can-i create pods --namespace=production
kubectl auth can-i create pods --as=alice --namespace=production
kubectl auth can-i '*' '*' --all-namespaces  # Est-ce qu'on est admin ?
```

---

## 9. Autoscaling

### HPA — Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70     # Scale up si CPU > 70%
    - type: Resource
      resource:
        name: memory
        target:
          type: AverageValue
          averageValue: 500Mi
    - type: Pods
      pods:
        metric:
          name: requests_per_second  # Métrique custom (via Prometheus Adapter)
        target:
          type: AverageValue
          averageValue: 1000
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 30
      policies:
        - type: Percent
          value: 100         # Max doubler les replicas
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300  # Attendre 5 min avant de scale down
```

### KEDA — Event-Driven Autoscaling

```yaml
# Scale depuis 0 à N basé sur la longueur d'une queue Kafka
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: kafka-consumer-scaler
spec:
  scaleTargetRef:
    name: kafka-consumer
  minReplicaCount: 0      # Scale to zero !
  maxReplicaCount: 50
  triggers:
    - type: kafka
      metadata:
        bootstrapServers: kafka:9092
        consumerGroup: my-group
        topic: my-topic
        lagThreshold: "100"   # 1 replica pour 100 messages en retard
```

---

## 10. CRDs et Operators

### Custom Resource Definitions

```yaml
# Définir une nouvelle ressource K8s : "Database"
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.mycompany.com
spec:
  group: mycompany.com
  names:
    kind: Database
    plural: databases
    singular: database
    shortNames: ["db"]
  scope: Namespaced
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine:
                  type: string
                  enum: ["postgresql", "mysql"]
                version:
                  type: string
                size:
                  type: string
                  enum: ["small", "medium", "large"]
```

```yaml
# Utilisation de la CRD "Database"
apiVersion: mycompany.com/v1alpha1
kind: Database
metadata:
  name: production-db
spec:
  engine: postgresql
  version: "15"
  size: large
```

### Pattern Operator — Reconciliation Loop

```
┌──────────────────────────────────────────────────────────┐
│                    Controller Loop                        │
│                                                          │
│  Watch(Database CR) → Observe desired state             │
│         ↓                                               │
│  Get current state (StatefulSet, Services, Secrets...)  │
│         ↓                                               │
│  Compute diff (desired ↔ current)                       │
│         ↓                                               │
│  Act (create/update/delete K8s resources)               │
│         ↓                                               │
│  Update CR Status                                        │
│         ↓                                               │
│  ←────────── Boucle infinie ──────────────────         │
└──────────────────────────────────────────────────────────┘
```

**Opérateurs populaires** :
- **Zalando PostgreSQL Operator** : PostgreSQL HA automatisé (réplication, failover, backups)
- **Strimzi** : Apache Kafka sur K8s (topics, users, connectors comme CRDs)
- **Prometheus Operator** : `ServiceMonitor`, `PrometheusRule` comme CRDs
- **cert-manager** : gestion automatique des certificats TLS (Let's Encrypt)
- **Argo CD** : déploiement GitOps
- **Crossplane** : provisionner des ressources cloud (AWS RDS, S3) comme CRDs K8s

---

## 11. Service Mesh — Istio

### Pourquoi un Service Mesh ?

Un service mesh déplace la logique réseau complexe (mTLS, retry, circuit breaker, observabilité) hors du code applicatif, dans des sidecars Envoy injectés automatiquement.

```
Sans Istio :                    Avec Istio (Envoy sidecar) :
  App A → App B                  App A → Envoy A → [mTLS] → Envoy B → App B
  (HTTP en clair)                (automatique, sans code)
```

### Ressources Istio clés

```yaml
# VirtualService : règles de routing du trafic
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-api
spec:
  hosts:
    - my-api
  http:
    - match:
        - headers:
            x-canary:
              exact: "true"
      route:
        - destination:
            host: my-api
            subset: v2
    - route:                    # Trafic par défaut : 90% v1, 10% v2
        - destination:
            host: my-api
            subset: v1
          weight: 90
        - destination:
            host: my-api
            subset: v2
          weight: 10
---
# DestinationRule : définit les subsets et les politiques de connexion
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-api
spec:
  host: my-api
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL    # mTLS automatique
    connectionPool:
      http:
        retries:
          attempts: 3
          retryOn: "5xx,reset,connect-failure"
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

**Linkerd** : alternative plus légère à Istio, certificats mTLS automatiques, footprint mémoire 5x plus faible. Recommandé pour les clusters de taille modérée.

---

## 12. GitOps — Argo CD

### Principe GitOps

```
Git repository (source of truth)
       │
       │ Webhook / Poll
       ▼
   Argo CD (détecte les diffs)
       │
       │ Sync
       ▼
Kubernetes cluster (état réconcilié avec Git)
```

**Avantages** : auditabilité totale (git log = historique des déploiements), rollback trivial (`git revert`), séparation CI (build) / CD (deploy).

```yaml
# Application Argo CD
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-webapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/k8s-configs.git
    targetRevision: main
    path: production/my-webapp   # Dossier dans le repo
    helm:
      valueFiles:
        - values-prod.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true       # Supprimer les ressources K8s qui ne sont plus dans Git
      selfHeal: true    # Re-synchroniser si quelqu'un modifie K8s manuellement
    syncOptions:
      - CreateNamespace=true
```

**App of Apps pattern** : une Application Argo CD qui déploie d'autres Applications — permet de gérer tout un cluster depuis un seul dépôt Git.

---

## 13. K8s Production Checklist

### Ressources et limites (obligatoires en production)

```yaml
resources:
  requests:               # Garantis par K8s pour le scheduling
    cpu: "100m"           # 0.1 vCPU
    memory: "128Mi"
  limits:                 # Maximum absolu (OOM kill si dépassé pour mémoire)
    cpu: "500m"
    memory: "512Mi"
```

> [!warning] Pas de limits CPU en production
> Les limits CPU provoquent du CPU throttling, pas de kill. Une application qui dépasse sa CPU limit est ralentie artificiellement. Mieux : pas de limit CPU + HPA pour scaler. La memory limit est en revanche obligatoire (OOM protection).

### Probes — Santé des pods

```yaml
livenessProbe:            # Pod mort → K8s redémarre le container
  httpGet:
    path: /health/live
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:           # Pod pas prêt → K8s ne lui envoie pas de trafic
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 3

startupProbe:             # Pour les apps à démarrage lent (évite liveness kills prématurés)
  httpGet:
    path: /health/live
    port: 8080
  failureThreshold: 30    # Attend jusqu'à 5 minutes (30 × 10s)
  periodSeconds: 10
```

### Pod Disruption Budget

```yaml
# Garantit qu'au moins 80% des pods sont disponibles pendant une éviction
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  minAvailable: "80%"   # Ou: maxUnavailable: "20%"
  selector:
    matchLabels:
      app: api
```

### Priority Classes

```yaml
# Pods critiques survivent aux évictions de ressources
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "Pour les services critiques de production"
---
# Dans le pod spec :
priorityClassName: high-priority
```

---

## 14. Wikilinks et ressources

- [[04 - Kubernetes Introduction]] — Bases K8s (Pods, Deployments, Services)
- [[04 - Microservices avec Spring]] — Architecture microservices déployée sur K8s
- [[07 - MLOps et Deploiement de Modeles]] — Déploiement de modèles ML sur K8s
- [[07 - DevSecOps]] — Sécurité K8s (Pod Security Standards, image scanning)

---

## Exercices Pratiques

### Exercice 1 — Créer un chart Helm
```bash
# 1. helm create mon-app (structure de base)
# 2. Modifier values.yaml pour une app Node.js avec Redis
# 3. Ajouter Redis comme dépendance via Chart.yaml
# 4. helm template pour valider les manifests générés
# 5. helm install --dry-run pour simuler le déploiement
# 6. Déployer sur minikube et vérifier avec kubectl get all
```

### Exercice 2 — StatefulSet Redis Cluster
Déployer un Redis Cluster à 3 nœuds avec StatefulSet :
1. StatefulSet avec 3 replicas, stockage 1Gi par pod
2. Headless Service pour le DNS stable
3. ConfigMap pour la configuration Redis
4. Initialisation du cluster dans un initContainer
5. Tester la persistance : `kubectl delete pod redis-0` → les données survivent

### Exercice 3 — RBAC multi-équipes
Architecture : namespace `frontend`, namespace `backend`, namespace `databases`
1. Role `frontend-dev` : peut lister/get les pods et logs dans `frontend`, ne peut PAS modifier les secrets
2. Role `backend-dev` : peut créer/modifier les deployments dans `backend`
3. ClusterRole `db-reader` : peut lire les StatefulSets dans tout le cluster
4. Tester avec `kubectl auth can-i` pour chaque role

### Exercice 4 — Argo CD GitOps
1. Installer Argo CD sur minikube
2. Créer un repo Git avec la configuration K8s de votre app
3. Créer une Application Argo CD pointant vers ce repo
4. Modifier le tag de l'image dans Git → observer Argo CD synchroniser automatiquement
5. Simuler un rollback : `git revert` → Argo CD revient à l'état précédent

> [!tip] Checklist production K8s résumée
> - Resources requests/limits sur tous les containers
> - Liveness + Readiness probes configurées
> - PodDisruptionBudget pour les services critiques
> - NetworkPolicy deny-all + allow explicit
> - RBAC avec least privilege
> - ServiceAccount dédié par application (pas de default SA)
> - Images scannées et signées
> - Secrets via Vault ou External Secrets Operator (pas de secrets en clair dans Git)
