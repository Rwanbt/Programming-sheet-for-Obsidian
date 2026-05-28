# MLOps et Déploiement de Modèles

## Qu'est-ce que MLOps ?

**MLOps** (Machine Learning Operations) est l'application des pratiques DevOps au cycle de vie des modèles de Machine Learning. C'est la discipline qui permet de passer d'un modèle entraîné dans un notebook Jupyter à un **service en production, fiable, scalable et maintenable**.

> [!tip] Le problème que MLOps résout
> Un data scientist entraîne un modèle avec 94 % d'accuracy. Formidable. Maintenant :
> - Comment le déployer en production ?
> - Comment garantir qu'il se comporte pareil sur les vraies données ?
> - Que faire quand les données changent et que les performances se dégradent ?
> - Comment gérer 12 versions du modèle avec différents hyperparamètres ?
> - Comment servir 10 000 prédictions par seconde ?
> MLOps répond à toutes ces questions.

### Pourquoi MLOps est Différent du DevOps Classique

| Dimension | DevOps classique | MLOps |
|---|---|---|
| **Artefact à déployer** | Code (déterministe) | Code + Données + Modèle (3 composantes) |
| **Tests** | Tests unitaires/intégration | Tests + validation des performances du modèle |
| **Reproductibilité** | Git suffit | Git + DVC (données et modèles sont trop lourds pour Git) |
| **Monitoring** | Latence, erreurs HTTP | Latence + data drift + model drift + fairness |
| **Rollback** | Revenir au commit précédent | Revenir au modèle précédent + ses données d'entraînement |
| **Déclencheur de redéploiement** | Nouveau code | Nouveau code OU nouvelles données OU drift détecté |
| **Expertise requise** | Dev + Ops | Dev + Ops + Data Science + MLE |

```
Écart classique (le "ML Tax") :
┌─────────────────────────────────────────────────────────┐
│                 Système ML en production                 │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Code ML (le modèle)                │    │
│  │                 ~5 % du système                 │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  Le reste (95 %) :                                       │
│  Configuration, monitoring, feature engineering,         │
│  data collection, serving infrastructure,                │
│  process management, resource management...              │
└─────────────────────────────────────────────────────────┘
```

---

## Cycle de Vie d'un Modèle ML

```
1. DÉVELOPPEMENT
   ├── Exploration des données (EDA)
   ├── Feature engineering
   ├── Sélection du modèle
   └── Entraînement + optimisation hyperparamètres

2. VALIDATION
   ├── Métriques de performance (accuracy, F1, AUC...)
   ├── Tests de fairness (biais par groupe)
   ├── Tests de robustesse (données adversariales)
   └── Validation métier (le modèle prédit ce qui a du sens ?)

3. STAGING
   ├── Déploiement en environnement de pré-production
   ├── Tests d'intégration avec le système cible
   ├── Tests de charge (peut-il supporter le trafic ?)
   └── Validation par les parties prenantes

4. PRODUCTION
   ├── Déploiement (blue/green, canary, A/B test)
   ├── Serving des prédictions
   └── Logging des inputs/outputs

5. MONITORING
   ├── Data drift : les données d'entrée ont-elles changé ?
   ├── Concept drift : la relation input→output a-t-elle changé ?
   ├── Performance dégradée ? (si les labels sont disponibles)
   └── Latence, throughput, erreurs

6. RETRAIN
   ├── Déclenché par drift, performance ou schedule
   ├── Nouvelles données d'entraînement
   └── Retour à l'étape 1

Durée de vie typique d'un modèle en production : 3-12 mois avant dégradation notable
```

---

## MLflow — Tracking et Gestion des Modèles

MLflow est la plateforme MLOps open-source la plus répandue. Elle couvre 4 fonctionnalités principales.

### MLflow Tracking — Journaliser les Expériences

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, f1_score, roc_auc_score
import pandas as pd

# Configurer le serveur MLflow
mlflow.set_tracking_uri("http://localhost:5000")
mlflow.set_experiment("classification-fraude")

# Charger les données
df = pd.read_csv("data/transactions.csv")
X = df.drop('fraude', axis=1)
y = df['fraude']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Entraînement avec tracking MLflow
with mlflow.start_run(run_name="rf-experiment-v3"):
    # Log des paramètres
    params = {
        "n_estimators": 200,
        "max_depth": 10,
        "min_samples_split": 5,
        "class_weight": "balanced",
    }
    mlflow.log_params(params)
    
    # Entraînement
    model = RandomForestClassifier(**params, random_state=42)
    model.fit(X_train, y_train)
    
    # Évaluation
    y_pred = model.predict(X_test)
    y_proba = model.predict_proba(X_test)[:, 1]
    
    # Log des métriques
    mlflow.log_metrics({
        "accuracy": accuracy_score(y_test, y_pred),
        "f1_score": f1_score(y_test, y_pred),
        "roc_auc": roc_auc_score(y_test, y_proba),
        "train_size": len(X_train),
    })
    
    # Log du modèle
    mlflow.sklearn.log_model(
        model,
        artifact_path="model",
        registered_model_name="DetecteurFraude",
        input_example=X_test.head(5),  # exemple d'input pour la doc
    )
    
    # Log d'artefacts supplémentaires
    import matplotlib.pyplot as plt
    from sklearn.metrics import ConfusionMatrixDisplay
    
    fig, ax = plt.subplots()
    ConfusionMatrixDisplay.from_predictions(y_test, y_pred, ax=ax)
    fig.savefig("confusion_matrix.png")
    mlflow.log_artifact("confusion_matrix.png")
    
    print(f"Run ID: {mlflow.active_run().info.run_id}")

# Lancer l'UI MLflow
# mlflow ui --port 5000  → http://localhost:5000
```

### MLflow Model Registry — Versioning des Modèles

```python
from mlflow.tracking import MlflowClient

client = MlflowClient()

# Promouvoir un modèle de Staging à Production
client.transition_model_version_stage(
    name="DetecteurFraude",
    version=3,
    stage="Production",
    archive_existing_versions=True,  # archive la version précédente
)

# Charger le modèle en production
model = mlflow.pyfunc.load_model("models:/DetecteurFraude/Production")
prediction = model.predict(pd.DataFrame([new_transaction]))

# Comparer deux runs
run_a = client.get_run("abc123")
run_b = client.get_run("def456")
print(f"Run A AUC: {run_a.data.metrics['roc_auc']:.4f}")
print(f"Run B AUC: {run_b.data.metrics['roc_auc']:.4f}")
```

```bash
# Démarrer le serveur MLflow avec PostgreSQL et S3
mlflow server \
  --backend-store-uri postgresql://user:password@localhost/mlflow \
  --default-artifact-root s3://mon-bucket-mlflow/artifacts \
  --host 0.0.0.0 \
  --port 5000
```

---

## DVC — Data Version Control

DVC versionne les données et les modèles avec Git, en stockant les gros fichiers dans un stockage distant (S3, GCS, Azure Blob, serveur SSH).

```bash
# Installation et initialisation
pip install dvc dvc-s3  # + connecteur cloud
dvc init  # crée .dvc/ dans le repo Git

# Configurer le stockage distant
dvc remote add -d myremote s3://mon-bucket-ml/dvc-store
dvc remote modify myremote region eu-west-1

# Versionner un dataset (le fichier lourd va sur S3, le .dvc sur Git)
dvc add data/transactions.csv
git add data/transactions.csv.dvc .gitignore
git commit -m "feat: add fraud detection dataset v1"
dvc push  # upload vers S3

# Versionner un modèle entraîné
dvc add models/rf_model.pkl
git add models/rf_model.pkl.dvc
git commit -m "feat: train fraud detector v1 - AUC 0.943"
dvc push

# Reproduire le pipeline sur une autre machine
git clone <repo>
dvc pull  # télécharge les données depuis S3
```

### Pipelines DVC

```yaml
# dvc.yaml — pipeline reproductible
stages:
  preprocess:
    cmd: python src/preprocess.py
    deps:
      - data/raw/transactions.csv
      - src/preprocess.py
    params:
      - params.yaml:
          - preprocess.test_size
          - preprocess.random_state
    outs:
      - data/processed/X_train.parquet
      - data/processed/X_test.parquet

  train:
    cmd: python src/train.py
    deps:
      - data/processed/X_train.parquet
      - src/train.py
    params:
      - params.yaml:
          - model.n_estimators
          - model.max_depth
    outs:
      - models/model.pkl
    metrics:
      - metrics.json:
          cache: false

  evaluate:
    cmd: python src/evaluate.py
    deps:
      - models/model.pkl
      - data/processed/X_test.parquet
    metrics:
      - metrics.json:
          cache: false
```

```bash
# Exécuter le pipeline (uniquement les stages dont les deps ont changé)
dvc repro

# Comparer deux expériences
dvc metrics diff HEAD~1 HEAD

# Afficher les métriques
dvc metrics show
```

---

## Serving de Modèles

### FastAPI — Servir un Modèle scikit-learn

```python
# api/main.py
import pickle
from pathlib import Path

import numpy as np
import pandas as pd
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, validator
import mlflow

app = FastAPI(
    title="API Détection de Fraude",
    version="1.0.0",
    description="Service de prédiction de fraude bancaire"
)

# Charger le modèle au démarrage (une seule fois)
MODEL = None

@app.on_event("startup")
async def charger_modele():
    global MODEL
    # Option 1 : depuis MLflow Model Registry
    MODEL = mlflow.sklearn.load_model("models:/DetecteurFraude/Production")
    # Option 2 : depuis un fichier local
    # with open("models/model.pkl", "rb") as f:
    #     MODEL = pickle.load(f)

class Transaction(BaseModel):
    montant: float
    heure: int                    # 0-23
    pays_origine: str
    type_transaction: str
    nb_transactions_24h: int
    
    @validator('montant')
    def montant_positif(cls, v):
        if v <= 0:
            raise ValueError('Le montant doit être positif')
        return v
    
    @validator('heure')
    def heure_valide(cls, v):
        if not 0 <= v <= 23:
            raise ValueError('Heure entre 0 et 23')
        return v

class PredictionResponse(BaseModel):
    fraude: bool
    probabilite: float
    score_risque: str  # "FAIBLE", "MOYEN", "ÉLEVÉ", "CRITIQUE"
    
def score_to_label(probabilite: float) -> str:
    if probabilite < 0.3:   return "FAIBLE"
    if probabilite < 0.6:   return "MOYEN"
    if probabilite < 0.85:  return "ÉLEVÉ"
    return "CRITIQUE"

@app.post("/predict", response_model=PredictionResponse)
async def predire(transaction: Transaction):
    if MODEL is None:
        raise HTTPException(status_code=503, detail="Modèle non chargé")
    
    # Préparer les features
    features = pd.DataFrame([{
        'montant': transaction.montant,
        'heure': transaction.heure,
        'pays_origine_encoded': hash(transaction.pays_origine) % 100,
        'type_encoded': hash(transaction.type_transaction) % 10,
        'nb_transactions_24h': transaction.nb_transactions_24h,
    }])
    
    probabilite = float(MODEL.predict_proba(features)[0, 1])
    fraude = probabilite > 0.5
    
    return PredictionResponse(
        fraude=fraude,
        probabilite=round(probabilite, 4),
        score_risque=score_to_label(probabilite),
    )

@app.get("/health")
async def health():
    return {"status": "ok", "model_loaded": MODEL is not None}
```

```bash
# Lancer le serveur
uvicorn api.main:app --host 0.0.0.0 --port 8080 --workers 4

# Test
curl -X POST http://localhost:8080/predict \
  -H "Content-Type: application/json" \
  -d '{"montant": 15000, "heure": 3, "pays_origine": "NG", "type_transaction": "VIREMENT", "nb_transactions_24h": 12}'
```

### BentoML — Serving Simplifié

```python
# service.py
import bentoml
from bentoml.io import JSON
import numpy as np

# Sauvegarder le modèle dans le store BentoML
bentoml.sklearn.save_model("detecteur_fraude", model, metadata={"auc": 0.943})

# Définir le service
svc = bentoml.Service("detecteur_fraude_service")

runner = bentoml.sklearn.get("detecteur_fraude:latest").to_runner()
svc.add_runner(runner)

@svc.api(input=JSON(), output=JSON())
async def predict(input_data: dict) -> dict:
    features = np.array([[
        input_data["montant"],
        input_data["heure"],
        input_data["nb_transactions_24h"],
    ]])
    prob = await runner.predict_proba.async_run(features)
    return {"probabilite": float(prob[0, 1])}

# Construire et servir
# bentoml build
# bentoml serve service:svc --port 3000
```

---

## Conteneurisation des Modèles ML

```dockerfile
# Dockerfile.ml
FROM python:3.11-slim

# Installer les dépendances système
RUN apt-get update && apt-get install -y \
    libgomp1 \  # requis par LightGBM/XGBoost
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copier et installer les dépendances Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copier le modèle et le code
COPY models/model.pkl models/
COPY api/ api/

# Variable pour la version du modèle (traceable)
ARG MODEL_VERSION=unknown
ENV MODEL_VERSION=$MODEL_VERSION

EXPOSE 8080
CMD ["uvicorn", "api.main:app", "--host", "0.0.0.0", "--port", "8080"]
```

```yaml
# docker-compose.yml
services:
  ml-api:
    build:
      context: .
      dockerfile: Dockerfile.ml
      args:
        MODEL_VERSION: "v1.3.2"
    ports:
      - "8080:8080"
    environment:
      - MLFLOW_TRACKING_URI=http://mlflow:5000
    volumes:
      - ./models:/app/models  # montage pour mise à jour sans rebuild
    restart: unless-stopped
    
  mlflow:
    image: ghcr.io/mlflow/mlflow:v2.13.0
    ports:
      - "5000:5000"
    environment:
      - MLFLOW_BACKEND_STORE_URI=postgresql://mlflow:mlflow@db/mlflow
    volumes:
      - mlflow-artifacts:/mlflow/artifacts
```

---

## Kubernetes pour le ML Serving

Voir [[04 - Kubernetes Introduction]] pour les bases. Voici les spécificités ML :

```yaml
# deployment-ml.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: detecteur-fraude
  labels:
    app: detecteur-fraude
    model-version: "1.3.2"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: detecteur-fraude
  template:
    metadata:
      labels:
        app: detecteur-fraude
    spec:
      containers:
      - name: api
        image: monregistry/detecteur-fraude:1.3.2
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30  # laisser le temps de charger le modèle
          periodSeconds: 10
        env:
        - name: MODEL_VERSION
          value: "1.3.2"

---
# HPA — scaling automatique selon la latence ou le CPU
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: detecteur-fraude-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: detecteur-fraude
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods
    pods:
      metric:
        name: request_latency_p99
      target:
        type: AverageValue
        averageValue: 100m  # 100ms p99
```

---

## Monitoring de Modèles

### Data Drift avec Evidently

```python
# Détecter si les données de production ont changé
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset, DataQualityPreset
from evidently.metrics import *
import pandas as pd

# Données d'entraînement (référence)
train_data = pd.read_parquet("data/train.parquet")
# Données de production du dernier mois
prod_data = pd.read_parquet("data/prod_last_month.parquet")

# Rapport de drift
rapport = Report(metrics=[
    DataDriftPreset(),
    DataQualityPreset(),
    # Métriques spécifiques
    ColumnDriftMetric(column_name="montant"),
    ColumnDriftMetric(column_name="heure"),
    DatasetDriftMetric(drift_share=0.3),  # alerte si 30 % des features ont drifté
])

rapport.run(
    reference_data=train_data,
    current_data=prod_data,
)

rapport.save_html("drift_report.html")

# Intégration dans un pipeline de monitoring
results = rapport.as_dict()
drift_detected = results["metrics"][0]["result"]["dataset_drift"]

if drift_detected:
    print("⚠️ DRIFT DÉTECTÉ — planifier un retrain")
    # Déclencher une alerte (PagerDuty, Slack, email)
    alerting_service.send_alert("Data drift détecté sur DetecteurFraude")
```

### Concept Drift

```python
# Concept drift = la relation X→y a changé (pas les données, les labels)
# Exemple : les fraudeurs changent de technique → le modèle ne les reconnaît plus

# Mesurer : comparer les métriques du modèle sur des fenêtres temporelles

from datetime import datetime, timedelta
import pandas as pd

def calculer_performance_fenetre(
    predictions_df: pd.DataFrame,
    labels_df: pd.DataFrame,
    fenetre_jours: int = 7
) -> dict:
    """
    Calcule les métriques sur les 'fenetre_jours' derniers jours.
    Nécessite d'avoir les vrais labels (possible si on observe les fraudes réelles).
    """
    date_debut = datetime.now() - timedelta(days=fenetre_jours)
    
    mask = predictions_df['timestamp'] >= date_debut
    preds = predictions_df[mask]
    labels = labels_df[labels_df['transaction_id'].isin(preds['transaction_id'])]
    
    if len(labels) < 100:
        return {"status": "insufficient_data"}
    
    from sklearn.metrics import f1_score, roc_auc_score
    
    return {
        "f1": f1_score(labels['fraude'], preds['prediction']),
        "auc": roc_auc_score(labels['fraude'], preds['proba']),
        "nb_samples": len(labels),
        "fenetre_jours": fenetre_jours,
    }
```

---

## A/B Testing de Modèles

```python
# A/B testing : comparer deux versions de modèles en production

import random
from fastapi import FastAPI, Request
import mlflow

app = FastAPI()

# Charger les deux versions
MODEL_A = mlflow.sklearn.load_model("models:/DetecteurFraude/1")  # version actuelle
MODEL_B = mlflow.sklearn.load_model("models:/DetecteurFraude/2")  # nouvelle version

@app.post("/predict")
async def predict(transaction: dict, request: Request):
    # Assigner aléatoirement 20 % du trafic au modèle B
    use_model_b = random.random() < 0.20
    
    model = MODEL_B if use_model_b else MODEL_A
    version = "B" if use_model_b else "A"
    
    features = prepare_features(transaction)
    proba = model.predict_proba(features)[0, 1]
    
    # Logger pour l'analyse ultérieure
    log_prediction({
        "transaction_id": transaction["id"],
        "model_version": version,
        "prediction": proba > 0.5,
        "proba": proba,
        "timestamp": datetime.utcnow().isoformat(),
    })
    
    return {"fraude": proba > 0.5, "proba": proba}

# Après N jours, comparer les métriques des deux groupes
# Si B est meilleur → promouvoir B à 100 % du trafic
```

---

## Feature Store — Feast

Un feature store centralise et partage les features entre les équipes data.

```python
# Définir un feature store avec Feast
from feast import FeatureStore, Entity, FeatureView, Field
from feast.types import Float32, Int64

# Entité
transaction = Entity(
    name="transaction_id",
    description="Identifiant unique de transaction",
)

# FeatureView — features pré-calculées
stats_utilisateur = FeatureView(
    name="stats_utilisateur_30j",
    entities=[transaction],
    ttl=timedelta(days=1),
    schema=[
        Field(name="montant_moyen_30j", dtype=Float32),
        Field(name="nb_transactions_30j", dtype=Int64),
        Field(name="pays_les_plus_frequents", dtype=String),
    ],
    source=my_offline_source,  # table Parquet ou BigQuery
)

# Dans l'API de serving : récupérer les features à la volée
store = FeatureStore(repo_path="feature_repo/")

features = store.get_online_features(
    features=["stats_utilisateur_30j:montant_moyen_30j"],
    entity_rows=[{"transaction_id": "txn_123"}],
).to_dict()
```

---

## Pipelines ML avec Prefect

```python
# pipeline_retrain.py
from prefect import flow, task
import mlflow
import pandas as pd

@task(retries=3, retry_delay_seconds=60)
def charger_donnees(source: str) -> pd.DataFrame:
    """Charger les nouvelles données depuis la source."""
    return pd.read_parquet(source)

@task
def preprocesser(df: pd.DataFrame) -> tuple:
    """Nettoyer et préparer les features."""
    from sklearn.model_selection import train_test_split
    X = df.drop('fraude', axis=1)
    y = df['fraude']
    return train_test_split(X, y, test_size=0.2)

@task
def entrainer(X_train, y_train) -> object:
    """Entraîner un nouveau modèle."""
    from sklearn.ensemble import RandomForestClassifier
    model = RandomForestClassifier(n_estimators=200, random_state=42)
    model.fit(X_train, y_train)
    return model

@task
def evaluer_et_enregistrer(model, X_test, y_test) -> float:
    """Évaluer et enregistrer dans MLflow si meilleur."""
    from sklearn.metrics import roc_auc_score
    proba = model.predict_proba(X_test)[:, 1]
    auc = roc_auc_score(y_test, proba)
    
    if auc > 0.93:  # seuil minimum pour enregistrer
        with mlflow.start_run():
            mlflow.log_metric("auc", auc)
            mlflow.sklearn.log_model(model, "model",
                registered_model_name="DetecteurFraude")
        print(f"Nouveau modèle enregistré : AUC = {auc:.4f}")
    else:
        print(f"Modèle rejeté : AUC = {auc:.4f} < seuil 0.93")
    
    return auc

@flow(name="pipeline-retrain-fraude")
def pipeline_retrain(source: str = "s3://data/transactions_new.parquet"):
    """Pipeline complet de retrain."""
    df = charger_donnees(source)
    X_train, X_test, y_train, y_test = preprocesser(df)
    model = entrainer(X_train, y_train)
    auc = evaluer_et_enregistrer(model, X_test, y_test)
    return auc

# Planifier le retrain hebdomadaire
from prefect.deployments import Deployment
from prefect.server.schemas.schedules import CronSchedule

deployment = Deployment.build_from_flow(
    flow=pipeline_retrain,
    name="retrain-hebdomadaire",
    schedule=CronSchedule(cron="0 2 * * 1"),  # Lundi 2h du matin
    work_queue_name="ml-workers",
)
deployment.apply()
```

---

## Coûts Cloud ML

```
Estimation coûts AWS pour un modèle de fraude (100 000 req/jour) :

SERVING (EC2 ou EKS) :
  m5.xlarge (4 vCPU, 16 GB) × 2 instances : ~150 $/mois
  → Option Spot instances : -70 % = ~45 $/mois (pour le dev/staging)
  → SageMaker Endpoint : ~200 $/mois (géré, mais plus cher)

STOCKAGE :
  S3 (modèles + datasets ~100 GB) : ~2 $/mois
  RDS PostgreSQL (MLflow backend) : ~50 $/mois

ENTRAÎNEMENT (à la demande) :
  m5.2xlarge × 2h de training/semaine : ~3 $/mois
  GPU p3.2xlarge (deep learning) × 4h : ~25 $/mois

MONITORING :
  CloudWatch ou Grafana Cloud : 0-30 $/mois

TOTAL ESTIMÉ : 150-300 $/mois
```

**Optimisations coûts :**
- **Spot instances** : -60-70 % pour les workloads tolérants aux interruptions (training)
- **Savings Plans** : -30-40 % pour les instances serving toujours actives
- **Distillation de modèle** : réduire la taille du modèle → moins de compute
- **Quantisation** : FP32 → INT8 = 4x moins de mémoire, 2-4x plus rapide
- **Model caching** : ne pas recharger le modèle à chaque requête

---

## Cas Pratique Complet — Détecteur de Fraude

### Structure du Projet

```
fraud-detection-mlops/
├── data/
│   ├── raw/
│   └── processed/
├── models/
├── src/
│   ├── preprocess.py
│   ├── train.py
│   ├── evaluate.py
│   └── features.py
├── api/
│   ├── main.py
│   └── schemas.py
├── monitoring/
│   └── drift_check.py
├── pipelines/
│   └── retrain.py
├── tests/
│   ├── test_api.py
│   └── test_model.py
├── dvc.yaml
├── params.yaml
├── Dockerfile
└── docker-compose.yml
```

### Pipeline Complet (résumé)

```bash
# 1. Setup initial
git init && dvc init
dvc remote add -d s3remote s3://mon-bucket/dvc

# 2. Versioner les données initiales
dvc add data/raw/transactions.csv
git add data/raw/transactions.csv.dvc
git commit -m "feat: add initial dataset v1"
dvc push

# 3. Entraîner et tracker avec MLflow
mlflow server &  # démarrer MLflow en arrière-plan
python src/train.py  # génère un run MLflow

# 4. Vérifier les résultats
mlflow ui  # ouvrir http://localhost:5000

# 5. Construire l'image Docker
docker build -t fraud-detector:v1 .

# 6. Déployer en local (test)
docker-compose up -d

# 7. Tester l'API
curl -X POST http://localhost:8080/predict \
  -d '{"montant": 500, "heure": 14, "pays_origine": "FR", \
       "type_transaction": "CB", "nb_transactions_24h": 2}'

# 8. Vérifier le monitoring
python monitoring/drift_check.py

# 9. Déployer en production sur Kubernetes
kubectl apply -f k8s/
kubectl rollout status deployment/detecteur-fraude
```

### Tests du Modèle

```python
# tests/test_model.py
import pytest
import pandas as pd
import numpy as np
import mlflow

@pytest.fixture
def model():
    return mlflow.sklearn.load_model("models:/DetecteurFraude/Staging")

def test_model_performance_minimale(model):
    """Le modèle doit avoir un AUC > 0.93 sur le jeu de test de référence."""
    from sklearn.metrics import roc_auc_score
    X_test = pd.read_parquet("data/test_reference.parquet")
    y_test = pd.read_csv("data/test_labels.csv")['fraude']
    
    proba = model.predict_proba(X_test)[:, 1]
    auc = roc_auc_score(y_test, proba)
    
    assert auc > 0.93, f"AUC {auc:.4f} < seuil minimum 0.93"

def test_modele_ne_plante_pas_sur_entrees_limites(model):
    """Le modèle ne doit pas lever d'exception sur des valeurs extrêmes."""
    cas_limites = pd.DataFrame([
        {"montant": 0.01, "heure": 0, "pays_encoded": 0, "type_encoded": 0, "nb_transactions_24h": 0},
        {"montant": 999999, "heure": 23, "pays_encoded": 99, "type_encoded": 9, "nb_transactions_24h": 1000},
        {"montant": np.nan, "heure": 12, "pays_encoded": 1, "type_encoded": 1, "nb_transactions_24h": 1},
    ]).fillna(0)
    
    predictions = model.predict(cas_limites)
    assert len(predictions) == 3
    assert all(p in [0, 1] for p in predictions)

def test_fairness_par_pays(model):
    """Le taux de faux positifs ne doit pas dépasser 10 % dans aucun groupe pays."""
    from sklearn.metrics import confusion_matrix
    
    for pays in ["FR", "DE", "US", "NG"]:
        subset = test_data[test_data['pays_origine'] == pays]
        if len(subset) < 50:
            continue
        preds = model.predict(subset.drop('fraude', axis=1))
        tn, fp, fn, tp = confusion_matrix(subset['fraude'], preds).ravel()
        fpr = fp / (fp + tn) if (fp + tn) > 0 else 0
        assert fpr <= 0.10, f"Taux de faux positifs trop élevé pour {pays}: {fpr:.2%}"
```

> [!info] Liens croisés
> - **Modèles scikit-learn** : [[03 - ML Supervise avec Scikit-Learn]]
> - **Infrastructure Kubernetes** : [[04 - Kubernetes Introduction]]
> - **CI/CD pour le pipeline ML** : [[04 - CI-CD avec GitHub Actions]]
> - **Monitoring et observabilité** : [[02 - Métriques et Monitoring]]

---

## Exercices Pratiques

### Exercice 1 — MLflow Tracking (1h)
Entraînez 3 modèles différents (RandomForest, GradientBoosting, LogisticRegression) sur le dataset Iris ou Titanic de scikit-learn :
1. Logger les paramètres, métriques et modèles de chaque run dans MLflow
2. Comparer les 3 runs dans l'UI MLflow
3. Enregistrer le meilleur modèle dans le Model Registry avec le tag "Champion"
4. Charger le modèle "Champion" depuis le registry et faire une prédiction

### Exercice 2 — API de Serving (1h30)
Créez une API FastAPI complète pour servir le modèle de l'exercice 1 :
1. Endpoint `POST /predict` avec validation Pydantic
2. Endpoint `GET /health` qui retourne la version du modèle et son statut
3. Endpoint `GET /metrics` qui retourne le nombre de prédictions, la distribution des classes prédites, et le temps de réponse moyen
4. Conteneuriser avec Docker (Dockerfile + test `docker run`)

### Exercice 3 — Monitoring Drift (1h)
Simulez un data drift et détectez-le avec Evidently :
1. Prenez 70 % du dataset comme données d'entraînement (référence)
2. Modifiez artificiellement les 30 % restants (changer la distribution d'une colonne numérique)
3. Générez un rapport Evidently HTML comparant les deux
4. Identifiez les features qui ont drifté et interprétez le rapport

### Exercice 4 — Pipeline Complet (3h)
Créez un pipeline MLOps de bout en bout pour un problème de classification (dataset au choix) :
1. DVC pour versionner les données et les modèles
2. Script d'entraînement avec MLflow tracking
3. API FastAPI pour le serving
4. Docker Compose avec l'API + MLflow
5. Script de vérification du drift (simulé)
6. Pipeline Prefect ou simple script Python qui orchestre le tout
7. Écrivez 3 tests pour le modèle (performance minimale, valeurs limites, cas dégénérés)

> [!info] Ressources MLOps
> - **Cours** : "MLOps Specialization" sur Coursera (DeepLearning.AI, Andrew Ng) — référence
> - **Livre** : "Designing Machine Learning Systems" de Chip Huyen — excellent panorama
> - **MLflow** : mlflow.org/docs — documentation officielle, très bien faite
> - **Evidently** : docs.evidentlyai.com — monitoring de drift
> - **Feast** : docs.feast.dev — feature store open-source
> - **Prefect** : docs.prefect.io — orchestration de pipelines
