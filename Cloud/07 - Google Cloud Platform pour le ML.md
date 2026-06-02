# Google Cloud Platform pour le Machine Learning

Google Cloud Platform (GCP) est l'une des trois grandes plateformes cloud du marché, et elle offre un écosystème ML parmi les plus complets grâce à des années de R&D interne chez Google (TensorFlow, TPUs, AlphaGo). Ce cours couvre l'ensemble de la chaîne MLOps sur GCP, depuis le stockage des données brutes jusqu'au déploiement d'un modèle en production, avec les outils concrets que vous rencontrerez en entreprise.

> [!info] Prérequis
> - Bases Python (niveau B1 Holberton)
> - Notions de Machine Learning (régression, classification, évaluation)
> - Familiarité avec les commandes shell Linux
> - Un compte Google (création d'un compte GCP gratuite avec 300 $ de crédits offerts)

---

## 1. Introduction à Google Cloud Platform

### 1.1 Architecture générale de GCP

GCP est organisé autour de **régions** et de **zones** géographiques. Comprendre cette hiérarchie est fondamental pour maîtriser les coûts et la latence.

```
Monde GCP
├── Région (ex: europe-west1 = Belgique)
│   ├── Zone A (europe-west1-a)
│   ├── Zone B (europe-west1-b)
│   └── Zone C (europe-west1-c)
├── Région (ex: us-central1 = Iowa)
└── Région (ex: asia-east1 = Taiwan)
```

> [!info] Choisir sa région
> Pour les projets ML en Europe, préférez `europe-west4` (Pays-Bas) ou `europe-west1` (Belgique). Toujours choisir la région **la plus proche de vos utilisateurs finaux** pour réduire la latence, et vérifiez que les services Vertex AI y sont disponibles.

### 1.2 La Console GCP

La console web est accessible sur [console.cloud.google.com](https://console.cloud.google.com). Elle permet de gérer tous les services visuellement. L'interface est organisée par **produits** (Storage, BigQuery, Vertex AI, etc.) accessibles depuis le menu hamburger.

**Navigation rapide :**
- `Ctrl + K` (ou `Cmd + K`) : recherche universelle dans la console
- La barre de recherche en haut accepte des noms de service, de ressource ou de page
- Les **pins** permettent de fixer ses services favoris dans la barre latérale

### 1.3 Projets GCP

Tout dans GCP s'organise dans des **projets**. Un projet est une unité d'isolation : facturation, permissions, quotas et ressources sont séparés par projet.

```
Organisation (ex: holberton.io)
├── Dossier: Étudiants B2
│   ├── Projet: ml-project-alice-2024
│   ├── Projet: ml-project-bob-2024
│   └── Projet: ml-shared-datasets
└── Dossier: Production
    └── Projet: holberton-prod
```

**Créer un projet via CLI :**

```bash
# Créer un nouveau projet
gcloud projects create mon-projet-ml-2024 \
  --name="Mon Projet ML" \
  --labels="env=dev,equipe=holberton"

# Lister tous les projets
gcloud projects list

# Définir le projet actif par défaut
gcloud config set project mon-projet-ml-2024

# Vérifier la configuration actuelle
gcloud config list
```

### 1.4 Installation et configuration de gcloud CLI

`gcloud` est l'outil en ligne de commande officiel de GCP. C'est votre meilleur ami pour l'automatisation.

```bash
# Installation sur Ubuntu/Debian
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# Initialisation (ouvre un navigateur pour l'authentification)
gcloud init

# Authentification seule (sans reconfigurer le projet)
gcloud auth login

# Authentification pour les SDKs Python (Application Default Credentials)
gcloud auth application-default login

# Vérifier l'identité connectée
gcloud auth list

# Mettre à jour gcloud
gcloud components update
```

> [!warning] Application Default Credentials (ADC)
> Le SDK Python `google-cloud-*` utilise les **ADC** pour s'authentifier. Après avoir installé gcloud, vous devez impérativement lancer `gcloud auth application-default login` pour que vos scripts Python trouvent les credentials. Sans ça, vous obtiendrez des erreurs `DefaultCredentialsError`.

**Structure de configuration gcloud :**

```bash
# Créer un profil de configuration nommé (utile si vous gérez plusieurs projets)
gcloud config configurations create holberton-ml
gcloud config set project mon-projet-ml-2024
gcloud config set compute/region europe-west1
gcloud config set compute/zone europe-west1-b

# Lister les configurations
gcloud config configurations list

# Activer une configuration
gcloud config configurations activate holberton-ml
```

### 1.5 IAM — Identity and Access Management

IAM contrôle **qui** peut faire **quoi** sur **quelles ressources**. C'est le système de permissions de GCP.

**Concepts clés :**

| Concept | Description | Exemple |
|---|---|---|
| **Principal** | Entité qui accède aux ressources | Utilisateur, Service Account, Groupe |
| **Role** | Ensemble de permissions | `roles/storage.objectViewer` |
| **Policy** | Association Principal ↔ Role sur une ressource | "Alice peut lire les buckets du projet X" |
| **Permission** | Action atomique | `storage.objects.get` |

**Types de rôles :**
- **Roles primitifs** (hérités) : `Owner`, `Editor`, `Viewer` — trop larges, éviter en production
- **Roles prédéfinis** : `roles/bigquery.dataViewer`, `roles/aiplatform.user` — granulaires
- **Roles personnalisés** : vous définissez exactement les permissions

```bash
# Voir les rôles d'un projet
gcloud projects get-iam-policy mon-projet-ml-2024

# Ajouter un rôle à un utilisateur
gcloud projects add-iam-policy-binding mon-projet-ml-2024 \
  --member="user:alice@holberton.io" \
  --role="roles/aiplatform.user"

# Créer un Service Account (pour les scripts automatisés)
gcloud iam service-accounts create ml-pipeline-sa \
  --display-name="ML Pipeline Service Account"

# Donner des permissions au Service Account
gcloud projects add-iam-policy-binding mon-projet-ml-2024 \
  --member="serviceAccount:ml-pipeline-sa@mon-projet-ml-2024.iam.gserviceaccount.com" \
  --role="roles/aiplatform.admin"

# Générer une clé JSON pour le Service Account
gcloud iam service-accounts keys create ./sa-key.json \
  --iam-account=ml-pipeline-sa@mon-projet-ml-2024.iam.gserviceaccount.com
```

> [!warning] Sécurité des clés Service Account
> Ne commitez **jamais** un fichier `sa-key.json` dans un repo Git. Ajoutez-le immédiatement à `.gitignore`. En production, utilisez **Workload Identity** ou les métadonnées de VM plutôt que des fichiers de clé.

### 1.6 Activation des APIs

Chaque service GCP doit être **activé** avant utilisation. C'est un mécanisme de contrôle et de facturation.

```bash
# Activer plusieurs APIs d'un coup (pour un projet ML complet)
gcloud services enable \
  storage.googleapis.com \
  bigquery.googleapis.com \
  aiplatform.googleapis.com \
  containerregistry.googleapis.com \
  cloudbuild.googleapis.com \
  run.googleapis.com \
  notebooks.googleapis.com

# Vérifier les APIs activées
gcloud services list --enabled

# Désactiver une API (attention : supprime les ressources associées parfois)
gcloud services disable containerregistry.googleapis.com
```

---

## 2. Google Cloud Storage (GCS)

### 2.1 Concept de buckets

GCS est un service de **stockage objet** (similaire à S3 d'AWS). Les données sont stockées dans des **buckets** (seaux), accessibles via HTTP ou les APIs Google.

```
Bucket: gs://mon-projet-ml-datasets/
├── raw/
│   ├── images/
│   │   ├── train/
│   │   │   ├── cat_001.jpg
│   │   │   └── dog_001.jpg
│   │   └── test/
│   └── tabular/
│       └── transactions.csv
├── processed/
│   └── features.parquet
└── models/
    └── v1/
        ├── model.pkl
        └── metrics.json
```

**Classes de stockage :**

| Classe | Usage | Prix stockage | Prix accès |
|---|---|---|---|
| **Standard** | Données fréquemment accédées | ~$0.020/GB/mois | Gratuit |
| **Nearline** | Accès < 1 fois/mois | ~$0.010/GB/mois | $0.01/GB lu |
| **Coldline** | Accès < 1 fois/trimestre | ~$0.004/GB/mois | $0.02/GB lu |
| **Archive** | Backups longue durée | ~$0.001/GB/mois | $0.05/GB lu |

> [!tip] Optimisation des coûts
> Pour les datasets ML, utilisez **Standard** pendant l'entraînement actif, puis migrez vers **Nearline** ou **Coldline** une fois le projet terminé. GCS supporte des politiques de cycle de vie automatiques pour faire ça sans intervention manuelle.

### 2.2 Créer et gérer des buckets

```bash
# Créer un bucket (le nom doit être GLOBALEMENT unique sur tout GCS)
gsutil mb -p mon-projet-ml-2024 \
         -c STANDARD \
         -l europe-west1 \
         gs://holberton-ml-datasets-2024/

# Avec gcloud (nouvelle syntaxe)
gcloud storage buckets create gs://holberton-ml-datasets-2024 \
  --project=mon-projet-ml-2024 \
  --location=europe-west1 \
  --default-storage-class=STANDARD

# Lister les buckets du projet
gsutil ls
gcloud storage buckets list

# Voir les détails d'un bucket
gsutil ls -L -b gs://holberton-ml-datasets-2024/
```

### 2.3 Opérations avec gsutil

```bash
# Copier un fichier local vers GCS
gsutil cp ./mon_dataset.csv gs://holberton-ml-datasets-2024/raw/

# Copier un dossier entier (flag -r pour récursif)
gsutil cp -r ./images/ gs://holberton-ml-datasets-2024/raw/images/

# Copie parallèle multi-thread (beaucoup plus rapide pour les gros datasets)
gsutil -m cp -r ./images/ gs://holberton-ml-datasets-2024/raw/images/

# Synchroniser un dossier local avec GCS (comme rsync)
gsutil -m rsync -r -d ./local_data/ gs://holberton-ml-datasets-2024/raw/

# Télécharger depuis GCS
gsutil cp gs://holberton-ml-datasets-2024/raw/mon_dataset.csv ./

# Lister le contenu d'un bucket
gsutil ls gs://holberton-ml-datasets-2024/
gsutil ls -la gs://holberton-ml-datasets-2024/raw/  # avec tailles et dates

# Supprimer
gsutil rm gs://holberton-ml-datasets-2024/raw/fichier_inutile.csv
gsutil rm -r gs://holberton-ml-datasets-2024/raw/dossier_inutile/

# Voir la taille totale d'un bucket
gsutil du -sh gs://holberton-ml-datasets-2024/

# Copier entre deux buckets GCS (côté serveur, sans passer par votre machine)
gsutil cp gs://source-bucket/data.csv gs://destination-bucket/data.csv
```

### 2.4 Utiliser GCS depuis Python

```python
from google.cloud import storage
import pandas as pd
import io

# Initialiser le client (utilise les ADC automatiquement)
client = storage.Client(project="mon-projet-ml-2024")

# Récupérer un bucket
bucket = client.bucket("holberton-ml-datasets-2024")

# Uploader un fichier
def upload_file(local_path: str, gcs_path: str) -> None:
    """Uploade un fichier local vers GCS.
    
    Args:
        local_path: Chemin local du fichier
        gcs_path: Chemin dans le bucket (ex: 'raw/data.csv')
    """
    blob = bucket.blob(gcs_path)
    blob.upload_from_filename(local_path)
    print(f"Fichier uploadé: gs://holberton-ml-datasets-2024/{gcs_path}")

# Télécharger un fichier
def download_file(gcs_path: str, local_path: str) -> None:
    blob = bucket.blob(gcs_path)
    blob.download_to_filename(local_path)

# Lire un CSV directement en mémoire (sans écrire sur disque)
def read_csv_from_gcs(gcs_path: str) -> pd.DataFrame:
    blob = bucket.blob(gcs_path)
    data = blob.download_as_bytes()
    return pd.read_csv(io.BytesIO(data))

# Lister les fichiers d'un préfixe
def list_files(prefix: str) -> list[str]:
    blobs = client.list_blobs("holberton-ml-datasets-2024", prefix=prefix)
    return [blob.name for blob in blobs]

# Exemple d'utilisation
df = read_csv_from_gcs("raw/transactions.csv")
print(f"Dataset chargé: {df.shape}")

# Uploader un DataFrame directement (sans fichier intermédiaire)
def upload_dataframe(df: pd.DataFrame, gcs_path: str) -> None:
    blob = bucket.blob(gcs_path)
    blob.upload_from_string(df.to_csv(index=False), content_type="text/csv")

# Uploader un modèle pickle
import pickle
def upload_model(model, gcs_path: str) -> None:
    blob = bucket.blob(gcs_path)
    blob.upload_from_string(pickle.dumps(model), content_type="application/octet-stream")
```

### 2.5 Contrôle d'accès aux buckets

```bash
# Rendre un objet public (attention : vraiment public sur internet)
gsutil acl ch -u AllUsers:R gs://holberton-ml-datasets-2024/public/demo_data.csv

# Donner accès à un Service Account spécifique
gsutil iam ch \
  serviceAccount:ml-pipeline-sa@mon-projet-ml-2024.iam.gserviceaccount.com:objectViewer \
  gs://holberton-ml-datasets-2024/

# Signed URL (accès temporaire sans credentials, valable 1 heure)
gsutil signurl -d 1h sa-key.json gs://holberton-ml-datasets-2024/modele/v1/model.pkl
```

---

## 3. BigQuery

### 3.1 Qu'est-ce que BigQuery ?

BigQuery est un **entrepôt de données** (data warehouse) serverless et massivement parallèle. Il peut analyser des téraoctets de données en quelques secondes grâce à son architecture columnar distribuée.

```
Architecture BigQuery
┌─────────────────────────────────────────┐
│ Projet GCP                              │
│  ├── Dataset: ml_features               │
│  │    ├── Table: transactions_2024      │
│  │    ├── Table: user_profiles          │
│  │    └── Vue: fraud_candidates         │
│  └── Dataset: model_results             │
│       └── Table: predictions_v2         │
└─────────────────────────────────────────┘
```

**Particularités importantes :**
- **Serverless** : pas de cluster à gérer, pas de provisionnement
- **Columnar** : les colonnes sont stockées séparément — requêtes sur quelques colonnes = très rapide
- **Facturation à la requête** : vous payez pour les données scannées (pas pour le temps de calcul)
- **Séparation stockage/calcul** : les données restent dans Colossus (système de fichiers Google)

### 3.2 Premiers pas avec BigQuery

```bash
# Créer un dataset
bq mk --dataset \
  --location=EU \
  --description="Datasets ML Holberton" \
  mon-projet-ml-2024:ml_features

# Charger un CSV depuis GCS dans une table
bq load \
  --source_format=CSV \
  --skip_leading_rows=1 \
  --autodetect \
  mon-projet-ml-2024:ml_features.transactions \
  gs://holberton-ml-datasets-2024/raw/transactions.csv

# Lancer une requête SQL simple
bq query --use_legacy_sql=false \
  'SELECT COUNT(*) as nb_lignes FROM `mon-projet-ml-2024.ml_features.transactions`'

# Exporter une table vers GCS
bq extract \
  --destination_format=CSV \
  mon-projet-ml-2024:ml_features.transactions \
  gs://holberton-ml-datasets-2024/exports/transactions_*.csv
```

### 3.3 SQL dans BigQuery

BigQuery utilise un dialecte SQL standard (SQL:2011) avec des extensions Google.

```sql
-- Requête basique : analyse des transactions
SELECT
  DATE(timestamp) AS jour,
  categorie,
  COUNT(*) AS nb_transactions,
  AVG(montant) AS montant_moyen,
  SUM(montant) AS chiffre_affaires
FROM `mon-projet-ml-2024.ml_features.transactions`
WHERE timestamp >= TIMESTAMP('2024-01-01')
  AND montant > 0
GROUP BY jour, categorie
ORDER BY jour DESC, chiffre_affaires DESC
LIMIT 100;

-- Fenêtres glissantes (très utiles pour les features temporelles en ML)
SELECT
  user_id,
  timestamp,
  montant,
  AVG(montant) OVER (
    PARTITION BY user_id
    ORDER BY timestamp
    ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
  ) AS moyenne_mobile_30j,
  COUNT(*) OVER (
    PARTITION BY user_id
    ORDER BY timestamp
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS nb_transactions_7j
FROM `mon-projet-ml-2024.ml_features.transactions`;

-- Détection d'anomalies basique
WITH stats AS (
  SELECT
    AVG(montant) AS moyenne,
    STDDEV(montant) AS ecart_type
  FROM `mon-projet-ml-2024.ml_features.transactions`
)
SELECT
  t.*,
  (t.montant - s.moyenne) / s.ecart_type AS z_score
FROM `mon-projet-ml-2024.ml_features.transactions` t
CROSS JOIN stats s
WHERE ABS((t.montant - s.moyenne) / s.ecart_type) > 3
ORDER BY ABS((t.montant - s.moyenne) / s.ecart_type) DESC;
```

### 3.4 BigQuery ML — Entraîner des modèles en SQL

BigQuery ML (BQML) permet d'entraîner des modèles directement dans BigQuery, **sans exporter les données** ni écrire une seule ligne de Python.

```sql
-- 1. Créer un modèle de régression logistique pour détecter la fraude
CREATE OR REPLACE MODEL `mon-projet-ml-2024.ml_features.modele_fraude`
OPTIONS(
  model_type = 'LOGISTIC_REG',
  input_label_cols = ['est_fraude'],
  max_iterations = 50,
  l2_reg = 0.01,
  data_split_method = 'AUTO_SPLIT'  -- séparation train/test automatique
) AS
SELECT
  montant,
  heure_journee,
  jour_semaine,
  pays_origine,
  categorie_marchand,
  nb_transactions_7j,
  ecart_montant_habituel,
  est_fraude  -- colonne label (0 ou 1)
FROM `mon-projet-ml-2024.ml_features.transactions_features`;

-- 2. Évaluer le modèle
SELECT *
FROM ML.EVALUATE(MODEL `mon-projet-ml-2024.ml_features.modele_fraude`,
  (SELECT * FROM `mon-projet-ml-2024.ml_features.transactions_features` WHERE split = 'TEST')
);
-- Retourne: precision, recall, accuracy, f1_score, roc_auc, log_loss

-- 3. Faire des prédictions
SELECT
  transaction_id,
  predicted_est_fraude,
  predicted_est_fraude_probs[OFFSET(1)].prob AS probabilite_fraude
FROM ML.PREDICT(MODEL `mon-projet-ml-2024.ml_features.modele_fraude`,
  (SELECT * FROM `mon-projet-ml-2024.ml_features.nouvelles_transactions`)
)
ORDER BY probabilite_fraude DESC;

-- 4. Autres types de modèles disponibles en BQML
-- model_type options:
-- 'LINEAR_REG'         → régression linéaire
-- 'LOGISTIC_REG'       → classification binaire/multi-classe
-- 'KMEANS'             → clustering non-supervisé
-- 'RANDOM_FOREST_CLASSIFIER'  → forêt aléatoire
-- 'BOOSTED_TREE_CLASSIFIER'   → XGBoost/Gradient Boosting
-- 'DNN_CLASSIFIER'     → réseau de neurones
-- 'MATRIX_FACTORIZATION' → système de recommandation
-- 'ARIMA_PLUS'         → prévision de séries temporelles
-- 'TENSORFLOW'         → importer un modèle TF depuis GCS

-- Exemple avec gradient boosting (souvent plus performant)
CREATE OR REPLACE MODEL `mon-projet-ml-2024.ml_features.modele_fraude_xgb`
OPTIONS(
  model_type = 'BOOSTED_TREE_CLASSIFIER',
  input_label_cols = ['est_fraude'],
  num_parallel_tree = 4,
  max_tree_depth = 6,
  subsample = 0.8,
  data_split_method = 'AUTO_SPLIT',
  enable_global_explain = TRUE  -- active les feature importances
) AS
SELECT * FROM `mon-projet-ml-2024.ml_features.transactions_features`;

-- Feature importances
SELECT * FROM ML.GLOBAL_EXPLAIN(MODEL `mon-projet-ml-2024.ml_features.modele_fraude_xgb`);
```

### 3.5 Utiliser BigQuery depuis Python

```python
from google.cloud import bigquery
import pandas as pd

# Initialiser le client
client = bigquery.Client(project="mon-projet-ml-2024")

# Requête simple → DataFrame pandas
def query_to_dataframe(sql: str) -> pd.DataFrame:
    """Exécute une requête BQ et retourne un DataFrame."""
    return client.query(sql).to_dataframe()

# Exemple : charger les features pour l'entraînement
df_features = query_to_dataframe("""
    SELECT
        montant,
        heure_journee,
        jour_semaine,
        nb_transactions_7j,
        est_fraude
    FROM `mon-projet-ml-2024.ml_features.transactions_features`
    WHERE DATE(timestamp) BETWEEN '2024-01-01' AND '2024-06-30'
""")

print(f"Shape: {df_features.shape}")
print(df_features.head())

# Écrire un DataFrame dans BigQuery
def upload_dataframe_to_bq(
    df: pd.DataFrame,
    table_id: str,
    write_disposition: str = "WRITE_TRUNCATE"
) -> None:
    """
    Uploade un DataFrame vers une table BigQuery.
    
    Args:
        df: Le DataFrame à uploader
        table_id: Format 'dataset.table_name'
        write_disposition: 'WRITE_TRUNCATE' (remplacer) ou 'WRITE_APPEND' (ajouter)
    """
    job_config = bigquery.LoadJobConfig(
        write_disposition=write_disposition,
        autodetect=True  # détecter le schéma automatiquement
    )
    
    full_table_id = f"mon-projet-ml-2024.{table_id}"
    job = client.load_table_from_dataframe(df, full_table_id, job_config=job_config)
    job.result()  # attendre la fin du job
    print(f"Table {full_table_id} mise à jour: {len(df)} lignes")

# Requête paramétrée (recommandé pour éviter les injections SQL)
def get_transactions_by_user(user_id: str) -> pd.DataFrame:
    query = """
        SELECT *
        FROM `mon-projet-ml-2024.ml_features.transactions`
        WHERE user_id = @user_id
        ORDER BY timestamp DESC
        LIMIT 1000
    """
    job_config = bigquery.QueryJobConfig(
        query_parameters=[
            bigquery.ScalarQueryParameter("user_id", "STRING", user_id)
        ]
    )
    return client.query(query, job_config=job_config).to_dataframe()
```

> [!tip] Optimiser les coûts BigQuery
> - Toujours utiliser `SELECT col1, col2` au lieu de `SELECT *` — vous payez pour les colonnes scannées
> - Partitionnez vos tables par date (`PARTITION BY DATE(timestamp)`) et filtrez toujours sur la partition
> - Utilisez `LIMIT` en développement pour tester vos requêtes à moindre coût
> - L'onglet "Validator" dans la console estime le coût **avant** d'exécuter

---

## 4. Vertex AI — La Plateforme ML Unifiée

### 4.1 Vue d'ensemble de Vertex AI

Vertex AI est la plateforme ML unifiée de Google Cloud, lancée en 2021. Elle centralise tout ce qui était éparpillé dans AI Platform, AutoML, et d'autres services.

```
Vertex AI
├── Data
│   ├── Datasets (gestion des données labelisées)
│   └── Feature Store (entrepôt de features ML)
├── Train
│   ├── AutoML (no-code)
│   └── Custom Training (code Python/containers)
├── Deploy
│   ├── Model Registry (versioning des modèles)
│   └── Endpoints (serving online et batch)
├── MLOps
│   ├── Pipelines (Kubeflow)
│   ├── Experiments (tracking)
│   └── Metadata (lignée des artéfacts)
└── Explainability AI (XAI)
```

### 4.2 Datasets Vertex AI

Les Datasets Vertex AI sont des **conteneurs de données labelisées** qui s'intègrent avec AutoML et les jobs d'entraînement.

**Types de datasets :**

| Type | Usage | Format accepté |
|---|---|---|
| **Image** | Classification, détection, segmentation | JPEG, PNG, BMP, GIF |
| **Texte** | Classification, extraction d'entités, Q&A | TXT, CSV |
| **Tabulaire** | Classification, régression | CSV, BigQuery |
| **Vidéo** | Classification, tracking d'objets | MP4, AVI, MOV |

```python
from google.cloud import aiplatform

# Initialiser Vertex AI
aiplatform.init(
    project="mon-projet-ml-2024",
    location="europe-west4"  # vérifier que Vertex AI est dispo dans cette région
)

# Créer un dataset tabulaire depuis GCS
dataset = aiplatform.TabularDataset.create(
    display_name="transactions-fraude-2024",
    gcs_source="gs://holberton-ml-datasets-2024/raw/transactions.csv"
)
print(f"Dataset créé: {dataset.resource_name}")

# Créer un dataset depuis BigQuery
dataset_bq = aiplatform.TabularDataset.create(
    display_name="transactions-fraude-bq",
    bq_source="bq://mon-projet-ml-2024.ml_features.transactions_features"
)

# Créer un dataset image
dataset_image = aiplatform.ImageDataset.create(
    display_name="chats-chiens-classifier",
    gcs_source="gs://holberton-ml-datasets-2024/raw/images/import_file.csv",
    import_schema_uri=aiplatform.schema.dataset.ioformat.image.single_label_classification
)
```

**Format du fichier d'import pour les images (CSV) :**
```csv
gs://holberton-ml-datasets-2024/raw/images/train/chat_001.jpg,chat
gs://holberton-ml-datasets-2024/raw/images/train/chat_002.jpg,chat
gs://holberton-ml-datasets-2024/raw/images/train/chien_001.jpg,chien
gs://holberton-ml-datasets-2024/raw/images/test/chat_003.jpg,chat
```

### 4.3 AutoML — No-Code ML

AutoML est la fonctionnalité la plus accessible de Vertex AI. Elle permet d'entraîner des modèles sans écrire une ligne de code de ML, en choisissant simplement les données et la tâche.

**AutoML Image Classification :**

```python
from google.cloud import aiplatform

aiplatform.init(project="mon-projet-ml-2024", location="europe-west4")

# Récupérer le dataset existant
dataset = aiplatform.ImageDataset("projects/mon-projet-ml-2024/locations/europe-west4/datasets/DATASET_ID")

# Lancer un job AutoML image
job = aiplatform.AutoMLImageTrainingJob(
    display_name="automl-chats-chiens-v1",
    prediction_type="classification",  # ou "object_detection"
    multi_label=False,  # True si une image peut avoir plusieurs labels
    model_type="CLOUD",  # CLOUD (meilleure précision) ou MOBILE_TF_LOW_LATENCY_1 (mobile)
    base_model=None  # None = entraîner from scratch, ou model existant pour fine-tuning
)

# Lancer l'entraînement (peut prendre des heures)
model = job.run(
    dataset=dataset,
    model_display_name="modele-chats-chiens-v1",
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    budget_milli_node_hours=8000,  # budget max en millièmes d'heure noeud (8h = 8000)
    sync=True  # attendre la fin (False pour lancer en arrière-plan)
)

print(f"Modèle entraîné: {model.resource_name}")
```

**AutoML Tabulaire :**

```python
# Job AutoML pour données tabulaires (classification ou régression)
job = aiplatform.AutoMLTabularTrainingJob(
    display_name="automl-fraude-detection-v1",
    optimization_prediction_type="classification",  # ou "regression"
    optimization_objective="maximize-au-prc",  # maximiser l'area under PR curve (fraude = données déséquilibrées)
    column_transformations=[
        {"numeric": {"column_name": "montant"}},
        {"numeric": {"column_name": "nb_transactions_7j"}},
        {"categorical": {"column_name": "categorie_marchand"}},
        {"categorical": {"column_name": "pays_origine"}},
        {"timestamp": {"column_name": "timestamp"}},
    ]
)

model = job.run(
    dataset=dataset_bq,
    target_column="est_fraude",
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    budget_milli_node_hours=1000,  # ~1h pour un petit dataset
    model_display_name="fraude-automl-v1",
    disable_early_stopping=False,  # arrêt précoce si pas d'amélioration
    sync=True
)
```

> [!info] Quand utiliser AutoML vs Custom Training ?
> - **AutoML** : vous avez des données propres, vous voulez un résultat rapide, vous n'avez pas besoin de contrôle fin sur l'architecture. Idéal pour prototyper.
> - **Custom Training** : vous avez une architecture spécifique (BERT, ResNet personnalisé, etc.), vous devez optimiser finement (learning rate scheduling, loss custom), ou vous travaillez sur un problème non standard.

### 4.4 Custom Training

Le Custom Training permet d'entraîner vos propres modèles sur l'infrastructure GCP avec n'importe quel framework (TensorFlow, PyTorch, scikit-learn, JAX...).

**Structure d'un script d'entraînement :**

```python
# trainer/task.py — script d'entraînement standard
import argparse
import os
import pandas as pd
import numpy as np
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, roc_auc_score
import pickle
from google.cloud import storage

def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, required=True,
                        help="Chemin GCS vers le dataset CSV")
    parser.add_argument("--model-dir", type=str, required=True,
                        help="Dossier GCS pour sauvegarder le modèle")
    parser.add_argument("--n-estimators", type=int, default=100)
    parser.add_argument("--learning-rate", type=float, default=0.1)
    parser.add_argument("--max-depth", type=int, default=4)
    return parser.parse_args()

def load_data(gcs_path: str) -> pd.DataFrame:
    """Charge les données depuis GCS."""
    # Vertex AI Custom Training peut accéder directement aux URIs gs://
    return pd.read_csv(gcs_path)

def train(args):
    print(f"Chargement des données depuis {args.data_path}")
    df = load_data(args.data_path)
    
    # Préparation
    FEATURE_COLS = ["montant", "heure_journee", "jour_semaine",
                    "nb_transactions_7j", "ecart_montant_habituel"]
    LABEL_COL = "est_fraude"
    
    X = df[FEATURE_COLS].values
    y = df[LABEL_COL].values
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42, stratify=y
    )
    
    # Entraînement
    print(f"Entraînement GBM: n_estimators={args.n_estimators}, lr={args.learning_rate}")
    model = GradientBoostingClassifier(
        n_estimators=args.n_estimators,
        learning_rate=args.learning_rate,
        max_depth=args.max_depth,
        random_state=42
    )
    model.fit(X_train, y_train)
    
    # Évaluation
    y_pred = model.predict(X_test)
    y_prob = model.predict_proba(X_test)[:, 1]
    
    print("\n=== Métriques ===")
    print(classification_report(y_test, y_pred))
    print(f"AUC-ROC: {roc_auc_score(y_test, y_prob):.4f}")
    
    # Sauvegarder le modèle
    model_path = "/tmp/model.pkl"
    with open(model_path, "wb") as f:
        pickle.dump(model, f)
    
    # Uploader vers GCS
    gcs_model_path = os.path.join(args.model_dir, "model.pkl")
    client = storage.Client()
    bucket_name, blob_path = gcs_model_path.replace("gs://", "").split("/", 1)
    bucket = client.bucket(bucket_name)
    blob = bucket.blob(blob_path)
    blob.upload_from_filename(model_path)
    print(f"Modèle sauvegardé: {gcs_model_path}")

if __name__ == "__main__":
    args = parse_args()
    train(args)
```

**Lancer un Custom Training Job :**

```python
from google.cloud import aiplatform

aiplatform.init(project="mon-projet-ml-2024", location="europe-west4")

# Option 1 : Utiliser un container pre-built de Google (le plus simple)
job = aiplatform.CustomTrainingJob(
    display_name="gbm-fraude-training-v1",
    script_path="trainer/task.py",           # script local
    container_uri="europe-docker.pkg.dev/vertex-ai/training/scikit-learn-cpu.1-0:latest",
    requirements=["pandas>=1.3.0", "scikit-learn>=1.0.0"],
    model_serving_container_image_uri="europe-docker.pkg.dev/vertex-ai/prediction/sklearn-cpu.1-0:latest"
)

model = job.run(
    args=[
        "--data-path", "gs://holberton-ml-datasets-2024/raw/transactions.csv",
        "--model-dir", "gs://holberton-ml-datasets-2024/models/v1/",
        "--n-estimators", "200",
        "--learning-rate", "0.05",
    ],
    replica_count=1,
    machine_type="n1-standard-4",   # 4 vCPU, 15 GB RAM
    # Pour GPU: machine_type="n1-standard-8", accelerator_type="NVIDIA_TESLA_T4", accelerator_count=1
    base_output_dir="gs://holberton-ml-datasets-2024/training-output/",
    sync=True
)
```

**Liste des containers pre-built disponibles :**

| Framework | CPU URI | GPU URI |
|---|---|---|
| TensorFlow 2.11 | `tf-cpu.2-11` | `tf-gpu.2-11` |
| PyTorch 1.13 | `pytorch-cpu.1-13` | `pytorch-gpu.1-13` |
| scikit-learn 1.0 | `sklearn-cpu.1-0` | N/A |
| XGBoost 1.6 | `xgboost-cpu.1-6` | N/A |

```bash
# Voir tous les containers disponibles
gcloud ai custom-jobs local-run --help
# Ou sur la doc : https://cloud.google.com/vertex-ai/docs/training/pre-built-containers
```

### 4.5 Hyperparameter Tuning

Vertex AI intègre un service d'optimisation d'hyperparamètres basé sur Bayesian Optimization.

```python
from google.cloud.aiplatform import hyperparameter_tuning as hpt

# Le script d'entraînement doit reporter les métriques via hypertune
# pip install cloudml-hypertune
# Dans task.py:
# import hypertune
# hpt_client = hypertune.HyperTune()
# hpt_client.report_hyperparameter_tuning_metric(
#     hyperparameter_metric_tag="auc_roc",
#     metric_value=roc_auc_score(y_test, y_prob),
#     global_step=args.n_estimators
# )

job = aiplatform.HyperparameterTuningJob(
    display_name="gbm-hparam-tuning",
    custom_job=aiplatform.CustomJob(
        display_name="gbm-hparam-base",
        worker_pool_specs=[{
            "machine_spec": {"machine_type": "n1-standard-4"},
            "replica_count": 1,
            "python_package_spec": {
                "executor_image_uri": "europe-docker.pkg.dev/vertex-ai/training/scikit-learn-cpu.1-0:latest",
                "package_uris": ["gs://holberton-ml-datasets-2024/trainer-0.1.tar.gz"],
                "python_module": "trainer.task"
            }
        }]
    ),
    metric_spec={"auc_roc": "maximize"},  # métrique à optimiser + direction
    parameter_spec={
        "learning-rate": hpt.DoubleParameterSpec(min=0.001, max=0.5, scale="log"),
        "n-estimators": hpt.IntegerParameterSpec(min=50, max=500, scale="linear"),
        "max-depth": hpt.IntegerParameterSpec(min=2, max=8, scale="linear"),
    },
    max_trial_count=20,          # nombre maximum d'essais
    parallel_trial_count=4,      # essais parallèles (coût x4 mais 4x plus rapide)
    search_algorithm=None        # None = Bayesian Optimization (recommandé)
)

job.run(sync=True)

# Récupérer le meilleur trial
best_trial = min(job.trials, key=lambda t: -t.final_measurement.metrics[0].value)
print(f"Meilleurs hyperparamètres: {best_trial.parameters}")
```

### 4.6 Model Registry

Le Model Registry centralise tous vos modèles entraînés avec leurs métadonnées, métriques et versions.

```python
from google.cloud import aiplatform

aiplatform.init(project="mon-projet-ml-2024", location="europe-west4")

# Uploader un modèle depuis GCS manuellement
model = aiplatform.Model.upload(
    display_name="fraude-gbm-v2",
    artifact_uri="gs://holberton-ml-datasets-2024/models/v2/",
    serving_container_image_uri="europe-docker.pkg.dev/vertex-ai/prediction/sklearn-cpu.1-0:latest",
    serving_container_predict_route="/predict",
    serving_container_health_route="/health",
    description="GBM entraîné sur transactions 2024, AUC-ROC=0.94",
    labels={"version": "2", "framework": "sklearn", "team": "holberton"}
)

print(f"Modèle enregistré: {model.resource_name}")
print(f"Version: {model.version_id}")

# Lister tous les modèles
models = aiplatform.Model.list(
    filter="labels.framework=sklearn",
    order_by="update_time desc"
)
for m in models:
    print(f"{m.display_name} | version {m.version_id} | {m.update_time}")

# Créer une nouvelle version d'un modèle existant
new_version = aiplatform.Model.upload(
    parent_model=model.resource_name,  # lier à un modèle existant
    display_name="fraude-gbm-v3",
    artifact_uri="gs://holberton-ml-datasets-2024/models/v3/",
    serving_container_image_uri="europe-docker.pkg.dev/vertex-ai/prediction/sklearn-cpu.1-0:latest",
    is_default_version=True  # cette version devient la version par défaut
)
```

### 4.7 Endpoints — Déploiement et Prédiction

Un **Endpoint** est un serveur de prédiction managé. Il gère automatiquement le scaling, l'autoscaling et la haute disponibilité.

```python
# Créer un endpoint
endpoint = aiplatform.Endpoint.create(
    display_name="fraude-detection-endpoint",
    description="Endpoint de production pour la détection de fraude",
    labels={"env": "production"}
)

# Déployer un modèle sur l'endpoint
endpoint.deploy(
    model=model,
    deployed_model_display_name="fraude-gbm-v2-deployed",
    machine_type="n1-standard-2",
    min_replica_count=1,      # au moins 1 réplique active
    max_replica_count=5,      # scale jusqu'à 5 répliques si charge élevée
    traffic_split={"0": 100}  # 100% du trafic sur ce modèle
)

# Prédiction online (une requête à la fois ou petit batch)
predictions = endpoint.predict(
    instances=[
        {
            "montant": 1500.0,
            "heure_journee": 3,
            "jour_semaine": 0,
            "nb_transactions_7j": 15,
            "ecart_montant_habituel": 4.2
        },
        {
            "montant": 25.0,
            "heure_journee": 14,
            "jour_semaine": 2,
            "nb_transactions_7j": 3,
            "ecart_montant_habituel": 0.1
        }
    ]
)
print(predictions.predictions)

# Prédiction batch (pour des millions d'instances)
batch_prediction_job = model.batch_predict(
    job_display_name="batch-fraude-juin-2024",
    gcs_source="gs://holberton-ml-datasets-2024/batch_input/juin_2024.jsonl",
    gcs_destination_prefix="gs://holberton-ml-datasets-2024/batch_output/juin_2024/",
    instances_format="jsonl",       # ou "csv", "bigquery"
    predictions_format="jsonl",
    machine_type="n1-standard-4",
    starting_replica_count=2,
    max_replica_count=8,
    sync=True
)

# Test du déploiement A/B (traffic splitting)
# Déployer une deuxième version et partager le trafic
endpoint.deploy(
    model=new_version_model,
    deployed_model_display_name="fraude-gbm-v3-deployed",
    machine_type="n1-standard-2",
    min_replica_count=1,
    max_replica_count=3,
    traffic_split={
        "deployed_model_v2_id": 80,  # 80% sur v2
        "0": 20                       # 20% sur le nouveau modèle
    }
)
```

> [!warning] Coûts des Endpoints
> Un endpoint avec `min_replica_count=1` tourne **en permanence**, même sans requêtes. Le coût minimum est environ **$0.10-0.30/heure** selon le `machine_type`. En développement, pensez à supprimer vos endpoints (`endpoint.delete()`) quand vous ne les utilisez pas.

### 4.8 Vertex AI Pipelines

Les Pipelines permettent d'orchestrer des workflows ML complexes sous forme de DAG (Directed Acyclic Graph).

```
Pipeline: fraud_detection_pipeline
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  data_ingestion │──▶│  feature_eng │──▶│   training   │
└──────────────┘    └──────────────┘    └──────┬───────┘
                                               │
                                        ┌──────▼───────┐
                                        │  evaluation  │
                                        └──────┬───────┘
                                               │ (si AUC > 0.90)
                                        ┌──────▼───────┐
                                        │  deployment  │
                                        └──────────────┘
```

```python
from kfp import dsl
from kfp.v2 import compiler
from google.cloud.aiplatform import pipeline_jobs

# Définir un composant (fonction Python → step du pipeline)
@dsl.component(
    packages_to_install=["pandas", "scikit-learn", "google-cloud-storage"],
    base_image="python:3.9-slim"
)
def preprocess_data(
    input_gcs_path: str,
    output_gcs_path: dsl.Output[dsl.Dataset]
):
    """Composant de prétraitement des données."""
    import pandas as pd
    from sklearn.preprocessing import StandardScaler
    import pickle
    
    df = pd.read_csv(input_gcs_path)
    
    # Normalisation
    scaler = StandardScaler()
    numeric_cols = ["montant", "nb_transactions_7j", "ecart_montant_habituel"]
    df[numeric_cols] = scaler.fit_transform(df[numeric_cols])
    
    df.to_csv(output_gcs_path.path, index=False)

@dsl.component(
    packages_to_install=["pandas", "scikit-learn"],
    base_image="python:3.9-slim"
)
def train_model(
    input_data: dsl.Input[dsl.Dataset],
    model_output: dsl.Output[dsl.Model],
    n_estimators: int = 100,
    learning_rate: float = 0.1
) -> dsl.Metrics:
    import pandas as pd
    import pickle
    from sklearn.ensemble import GradientBoostingClassifier
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import roc_auc_score
    
    df = pd.read_csv(input_data.path)
    X = df.drop("est_fraude", axis=1).values
    y = df["est_fraude"].values
    
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
    
    model = GradientBoostingClassifier(
        n_estimators=n_estimators,
        learning_rate=learning_rate
    )
    model.fit(X_train, y_train)
    
    auc = roc_auc_score(y_test, model.predict_proba(X_test)[:, 1])
    
    with open(model_output.path, "wb") as f:
        pickle.dump(model, f)
    
    # Retourner les métriques (visibles dans l'UI Vertex AI)
    metrics = dsl.Metrics()
    metrics.log_metric("auc_roc", auc)
    return metrics

# Définir le pipeline (DAG)
@dsl.pipeline(
    name="fraude-detection-pipeline",
    description="Pipeline complet de détection de fraude"
)
def fraude_pipeline(
    input_data_path: str = "gs://holberton-ml-datasets-2024/raw/transactions.csv",
    n_estimators: int = 200,
    learning_rate: float = 0.1
):
    # Step 1: Prétraitement
    preprocess_task = preprocess_data(input_gcs_path=input_data_path)
    
    # Step 2: Entraînement (dépend du step 1)
    train_task = train_model(
        input_data=preprocess_task.outputs["output_gcs_path"],
        n_estimators=n_estimators,
        learning_rate=learning_rate
    )
    
    # Les dépendances sont inférées automatiquement des outputs/inputs

# Compiler le pipeline en JSON
compiler.Compiler().compile(
    pipeline_func=fraude_pipeline,
    package_path="fraude_pipeline.json"
)

# Lancer le pipeline
job = pipeline_jobs.PipelineJob(
    display_name="fraude-pipeline-run-001",
    template_path="fraude_pipeline.json",
    parameter_values={
        "input_data_path": "gs://holberton-ml-datasets-2024/raw/transactions.csv",
        "n_estimators": 300,
        "learning_rate": 0.05
    },
    enable_caching=True  # réutilise les résultats si les inputs n'ont pas changé
)

job.run(sync=False)  # lancer en arrière-plan
print(f"Pipeline lancé: {job.resource_name}")
```

---

## 5. Vertex AI Workbench (JupyterLab Managé)

### 5.1 Présentation

Vertex AI Workbench remplace l'ancien "AI Platform Notebooks". C'est un service **JupyterLab entièrement managé** qui tourne sur une VM GCP avec accès natif à tous les services Google Cloud.

**Deux types d'instances :**

| Type | Description | Usage |
|---|---|---|
| **Managed Notebooks** | Entièrement managé par Google, autoscaling jusqu'à 0 | Exploration, collaboration |
| **User-Managed Notebooks** | VM configurable (GPU, RAM, disk), contrôle total | Entraînements intensifs |

### 5.2 Créer une instance Workbench

```bash
# Via gcloud (User-Managed Notebooks)
gcloud notebooks instances create mon-notebook-ml \
  --vm-image-project=deeplearning-platform-release \
  --vm-image-family=tf-latest-cpu \
  --machine-type=n1-standard-4 \
  --location=europe-west4-a \
  --no-public-ip  # sécurité: sans IP publique (accès via IAP)

# Avec GPU T4
gcloud notebooks instances create notebook-gpu \
  --vm-image-project=deeplearning-platform-release \
  --vm-image-family=pytorch-latest-gpu \
  --machine-type=n1-standard-8 \
  --accelerator-type=NVIDIA_TESLA_T4 \
  --accelerator-core-count=1 \
  --location=europe-west4-a

# Lister les instances
gcloud notebooks instances list --location=europe-west4-a

# Arrêter une instance (arrêter la facturation sans supprimer)
gcloud notebooks instances stop mon-notebook-ml \
  --location=europe-west4-a

# Démarrer
gcloud notebooks instances start mon-notebook-ml \
  --location=europe-west4-a
```

> [!tip] Économiser sur les notebooks
> Arrêtez toujours votre instance Workbench quand vous ne travaillez pas. Une VM `n1-standard-4` coûte ~$0.19/heure, soit ~$140/mois si oubliée allumée. Configurez une alerte de budget dans GCP pour être notifié.

### 5.3 Images pré-configurées disponibles

```bash
# Voir toutes les images disponibles
gcloud compute images list \
  --project=deeplearning-platform-release \
  --no-standard-images \
  --filter="name:tf OR name:pytorch OR name:sklearn"
```

Les images populaires incluent :
- `tf-latest-cpu` / `tf-latest-gpu` — TensorFlow avec CUDA
- `pytorch-latest-cpu` / `pytorch-latest-gpu` — PyTorch avec CUDA
- `common-cpu` / `common-gpu` — Base Jupyter avec conda
- `r-latest-cpu` — R Studio Server

### 5.4 Accès et utilisation

```python
# Dans un notebook Workbench, les credentials GCP sont automatiquement disponibles
# Pas besoin de gcloud auth application-default login

from google.cloud import bigquery
from google.cloud import storage

# Ces clients fonctionnent directement sans configuration supplémentaire
bq_client = bigquery.Client()  # utilise automatiquement le SA de la VM
gcs_client = storage.Client()

# Accès à GCS avec le chemin complet
import pandas as pd
df = pd.read_csv("gs://holberton-ml-datasets-2024/raw/transactions.csv")
print(df.head())
```

---

## 6. Cloud Run — Déployer une API ML Containerisée

### 6.1 Cloud Run en deux mots

Cloud Run est un service **serverless de containers** : vous déployez une image Docker et Google gère automatiquement l'infrastructure, le scaling (y compris jusqu'à 0 instance), et le HTTPS.

```
Requête HTTP
     │
     ▼
Cloud Run (gère le routing, TLS, autoscaling)
     │
     ▼
Container FastAPI + modèle ML
     │
     ├── /predict  → inférence
     └── /health   → health check
```

**Pourquoi Cloud Run pour le ML ?**
- Gratuit jusqu'à 2 millions de requêtes/mois (free tier)
- Scale à zéro (aucun coût sans trafic)
- Déploiement en quelques minutes depuis une image
- Idéal pour des APIs d'inférence à faible/moyen volume

### 6.2 Construire l'API avec FastAPI

```python
# app/main.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, validator
import pickle
import numpy as np
from google.cloud import storage
import os
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(
    title="API Détection de Fraude",
    description="Modèle GBM entraîné sur Vertex AI",
    version="1.0.0"
)

# Schéma de requête avec validation Pydantic
class TransactionFeatures(BaseModel):
    montant: float
    heure_journee: int
    jour_semaine: int
    nb_transactions_7j: int
    ecart_montant_habituel: float
    
    @validator("montant")
    def montant_must_be_positive(cls, v):
        if v < 0:
            raise ValueError("Le montant doit être positif")
        return v
    
    @validator("heure_journee")
    def heure_valide(cls, v):
        if not 0 <= v <= 23:
            raise ValueError("L'heure doit être entre 0 et 23")
        return v

class PredictionResponse(BaseModel):
    est_fraude: bool
    probabilite_fraude: float
    version_modele: str

# Charger le modèle au démarrage (une seule fois)
model = None
MODEL_GCS_PATH = os.environ.get(
    "MODEL_GCS_PATH",
    "gs://holberton-ml-datasets-2024/models/v2/model.pkl"
)
MODEL_VERSION = os.environ.get("MODEL_VERSION", "v2")

@app.on_event("startup")
async def load_model():
    """Charge le modèle depuis GCS au démarrage du container."""
    global model
    logger.info(f"Chargement du modèle depuis {MODEL_GCS_PATH}")
    
    try:
        client = storage.Client()
        bucket_name, blob_path = MODEL_GCS_PATH.replace("gs://", "").split("/", 1)
        bucket = client.bucket(bucket_name)
        blob = bucket.blob(blob_path)
        
        model_bytes = blob.download_as_bytes()
        model = pickle.loads(model_bytes)
        logger.info("Modèle chargé avec succès")
    except Exception as e:
        logger.error(f"Erreur chargement modèle: {e}")
        raise

@app.get("/health")
async def health_check():
    """Health check pour Cloud Run."""
    if model is None:
        raise HTTPException(status_code=503, detail="Modèle non chargé")
    return {"status": "healthy", "model_version": MODEL_VERSION}

@app.post("/predict", response_model=PredictionResponse)
async def predict(transaction: TransactionFeatures):
    """Prédit si une transaction est frauduleuse."""
    if model is None:
        raise HTTPException(status_code=503, detail="Modèle non disponible")
    
    # Préparer les features dans le bon ordre
    features = np.array([[
        transaction.montant,
        transaction.heure_journee,
        transaction.jour_semaine,
        transaction.nb_transactions_7j,
        transaction.ecart_montant_habituel
    ]])
    
    # Prédiction
    proba = model.predict_proba(features)[0, 1]
    is_fraud = bool(proba > 0.5)
    
    logger.info(f"Prédiction: fraude={is_fraud}, proba={proba:.3f}")
    
    return PredictionResponse(
        est_fraude=is_fraud,
        probabilite_fraude=float(proba),
        version_modele=MODEL_VERSION
    )
```

### 6.3 Dockerfile

```dockerfile
# Dockerfile
FROM python:3.10-slim

# Installer les dépendances
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code
COPY app/ .

# Cloud Run expose le port 8080 par défaut
EXPOSE 8080

# Variable d'environnement pour uvicorn
ENV PORT=8080

# Lancer l'API
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```

```txt
# requirements.txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.0
scikit-learn==1.3.2
numpy==1.24.4
google-cloud-storage==2.13.0
```

### 6.4 Build et déploiement

```bash
# Option 1 : Build local + push vers Artifact Registry
# Créer un repository Docker dans Artifact Registry
gcloud artifacts repositories create ml-apis \
  --repository-format=docker \
  --location=europe-west4

# Build de l'image
docker build -t europe-west4-docker.pkg.dev/mon-projet-ml-2024/ml-apis/fraude-api:v1 .

# Authentifier Docker avec GCP
gcloud auth configure-docker europe-west4-docker.pkg.dev

# Pousser l'image
docker push europe-west4-docker.pkg.dev/mon-projet-ml-2024/ml-apis/fraude-api:v1

# Option 2 : Cloud Build (build dans le cloud, sans Docker local)
gcloud builds submit --tag europe-west4-docker.pkg.dev/mon-projet-ml-2024/ml-apis/fraude-api:v2 .

# Déployer sur Cloud Run
gcloud run deploy fraude-api \
  --image europe-west4-docker.pkg.dev/mon-projet-ml-2024/ml-apis/fraude-api:v1 \
  --region europe-west4 \
  --platform managed \
  --memory 512Mi \
  --cpu 1 \
  --concurrency 80 \
  --min-instances 0 \
  --max-instances 10 \
  --set-env-vars MODEL_GCS_PATH=gs://holberton-ml-datasets-2024/models/v2/model.pkl,MODEL_VERSION=v2 \
  --service-account ml-pipeline-sa@mon-projet-ml-2024.iam.gserviceaccount.com \
  --allow-unauthenticated  # pour un accès public (enlever en prod)

# Obtenir l'URL de l'API déployée
gcloud run services describe fraude-api \
  --region europe-west4 \
  --format "value(status.url)"
```

**Tester l'API :**

```bash
# Health check
curl https://fraude-api-xxxx-ew.a.run.app/health

# Prédiction
curl -X POST https://fraude-api-xxxx-ew.a.run.app/predict \
  -H "Content-Type: application/json" \
  -d '{
    "montant": 1500.0,
    "heure_journee": 3,
    "jour_semaine": 0,
    "nb_transactions_7j": 15,
    "ecart_montant_habituel": 4.2
  }'
```

```python
# Appel depuis Python
import requests

API_URL = "https://fraude-api-xxxx-ew.a.run.app"

response = requests.post(
    f"{API_URL}/predict",
    json={
        "montant": 1500.0,
        "heure_journee": 3,
        "jour_semaine": 0,
        "nb_transactions_7j": 15,
        "ecart_montant_habituel": 4.2
    }
)
print(response.json())
# → {"est_fraude": true, "probabilite_fraude": 0.87, "version_modele": "v2"}
```

---

## 7. Google Colab et Colab Enterprise

### 7.1 Google Colab (gratuit)

Colab standard est gratuit et accessible à tous via [colab.research.google.com](https://colab.research.google.com).

**Limitations du Colab gratuit :**
- Sessions limitées à 12h maximum
- Déconnexion si inactif ~90 minutes
- GPU/TPU limités et non garantis (accès si ressources disponibles)
- RAM : ~12 GB RAM, stockage temporaire (~70 GB)

```python
# Dans un notebook Colab : monter Google Drive
from google.colab import drive
drive.mount('/content/drive')

# Accéder aux fichiers Drive
import pandas as pd
df = pd.read_csv('/content/drive/MyDrive/datasets/transactions.csv')

# Authentification GCP depuis Colab
from google.colab import auth
auth.authenticate_user()

# Maintenant les SDK GCP fonctionnent
from google.cloud import storage
client = storage.Client(project="mon-projet-ml-2024")

# Copier depuis GCS
!gsutil cp gs://holberton-ml-datasets-2024/raw/transactions.csv /content/

# Vérifier le GPU disponible
!nvidia-smi
import torch
print(f"GPU disponible: {torch.cuda.is_available()}")
print(f"GPU: {torch.cuda.get_device_name(0)}")
```

### 7.2 Colab Pro et Colab Pro+

| Feature | Gratuit | Pro ($10/mois) | Pro+ ($50/mois) |
|---|---|---|---|
| Temps de session | 12h | 24h | 24h |
| GPU premium (A100) | Non | Oui | Priorité |
| RAM | ~12 GB | ~25 GB | ~52 GB |
| Stockage | Temporaire | Temporaire | Temporaire |
| Sessions en arrière-plan | Non | Non | Oui |

### 7.3 Colab Enterprise

Colab Enterprise est intégré dans Vertex AI et offre des notebooks Colab avec des ressources GCP dédiées.

**Avantages vs Colab standard :**
- Exécution sur vos propres VMs GCP (contrôle des coûts, GPU dédiés)
- Intégration native avec BigQuery, GCS, Vertex AI
- Sécurité entreprise (VPC, IAM, logs d'audit)
- Pas de déconnexion intempestive

```bash
# Activer Colab Enterprise dans votre projet
gcloud services enable colab.googleapis.com

# Accessible via console.cloud.google.com/colab ou directement depuis Colab
# en basculant sur "Connect to a GCP VM"
```

---

## 8. Comparaison GCP vs AWS vs Azure pour le ML

### 8.1 Tableau comparatif global

| Dimension | GCP (Vertex AI) | AWS (SageMaker) | Azure (Azure ML) |
|---|---|---|---|
| **Plateforme ML** | Vertex AI | SageMaker | Azure ML |
| **Stockage objet** | Cloud Storage (GCS) | S3 | Azure Blob Storage |
| **Notebooks** | Vertex Workbench | SageMaker Studio | Azure ML Studio |
| **AutoML** | AutoML (image, texte, tabulaire) | SageMaker Autopilot (tabulaire) | AutoML (tabulaire) |
| **Pipelines** | Kubeflow Pipelines | SageMaker Pipelines | Azure ML Pipelines |
| **Serving** | Vertex Endpoints | SageMaker Endpoints | Azure ML Endpoints |
| **Feature Store** | Vertex Feature Store | SageMaker Feature Store | Azure ML Feature Store |
| **Data Warehouse** | BigQuery | Redshift | Synapse Analytics |
| **GPU propriétaire** | TPU v4 (JAX/TF) | Trainium/Inferentia | Non |
| **Part de marché cloud** | ~11% | ~31% | ~23% |

### 8.2 Comparaison des expériences développeur

**Facilité de démarrage :**

```
GCP     : ████████░░  8/10 — Console claire, SDK Python cohérent, docs excellentes
AWS     : ██████░░░░  6/10 — Puissant mais complexe, trop d'abstractions imbriquées
Azure   : ███████░░░  7/10 — Bon si vous êtes Microsoft-shop, MLflow intégré
```

**Points forts de chaque plateforme :**

| Plateforme | Points forts ML |
|---|---|
| **GCP** | BigQuery ML, TPUs, intégration TensorFlow/JAX native, Explainability AI |
| **AWS** | Écosystème le plus large, SageMaker Studio très complet, Ground Truth pour labeling |
| **Azure** | Intégration MLflow native, Azure DevOps, fort en enterprise, OpenAI partnership |

### 8.3 Mapping des services équivalents

```
Concept         │ GCP                    │ AWS                    │ Azure
────────────────┼────────────────────────┼────────────────────────┼──────────────────────
Stockage objet  │ GCS                    │ S3                     │ Blob Storage
Container reg.  │ Artifact Registry      │ ECR                    │ ACR
Serverless fn.  │ Cloud Functions        │ Lambda                 │ Azure Functions
Container srv.  │ Cloud Run              │ App Runner / ECS Fargate│ Container Apps
Managed K8s     │ GKE                    │ EKS                    │ AKS
Data warehouse  │ BigQuery               │ Redshift               │ Synapse Analytics
Spark managé    │ Dataproc               │ EMR                    │ HDInsight / Synapse Spark
ML Pipelines    │ Vertex Pipelines (KFP) │ SageMaker Pipelines    │ Azure ML Pipelines
Experiment track│ Vertex Experiments     │ SageMaker Experiments  │ MLflow (natif Azure ML)
Model registry  │ Vertex Model Registry  │ SageMaker Model Reg.   │ Azure ML Model Reg.
```

> [!tip] Quel cloud choisir pour votre projet ?
> - Vous êtes une startup, vous utilisez Google Workspace, ou votre data est dans BigQuery → **GCP**
> - Votre entreprise est AWS-first (très fréquent) ou vous avez besoin de l'écosystème le plus large → **AWS**
> - Votre entreprise utilise Microsoft 365/Azure AD, ou vous intégrez des services Microsoft → **Azure**
> - Vous voulez éviter le vendor lock-in → utilisez des outils open-source (MLflow, Kubeflow) sur n'importe quel cloud

---

## 9. Cas Pratique E2E — Classification sur Vertex AI

Ce cas pratique guide la création d'un pipeline complet de classification de fleurs Iris, depuis la préparation des données jusqu'au déploiement et à la consommation de l'endpoint.

### 9.1 Mise en place

```bash
# Créer le projet et activer les APIs
gcloud projects create holberton-iris-demo
gcloud config set project holberton-iris-demo
gcloud services enable aiplatform.googleapis.com storage.googleapis.com

# Créer le bucket
gsutil mb -l europe-west4 gs://holberton-iris-demo-data/

# Installer les dépendances Python
pip install google-cloud-aiplatform google-cloud-storage \
            scikit-learn pandas numpy fastapi uvicorn
```

### 9.2 Préparer et uploader les données

```python
# prepare_data.py
from sklearn.datasets import load_iris
import pandas as pd
from google.cloud import storage

def prepare_iris_dataset():
    """Prépare le dataset Iris et l'uploade sur GCS."""
    iris = load_iris()
    
    df = pd.DataFrame(
        iris.data,
        columns=["sepal_length", "sepal_width", "petal_length", "petal_width"]
    )
    df["species"] = iris.target
    df["species_name"] = df["species"].map({
        0: "setosa", 1: "versicolor", 2: "virginica"
    })
    
    # Sauvegarder localement
    df.to_csv("/tmp/iris_full.csv", index=False)
    print(f"Dataset: {df.shape}, distribution:\n{df['species_name'].value_counts()}")
    
    # Uploader sur GCS
    client = storage.Client(project="holberton-iris-demo")
    bucket = client.bucket("holberton-iris-demo-data")
    blob = bucket.blob("raw/iris.csv")
    blob.upload_from_filename("/tmp/iris_full.csv")
    print("Dataset uploadé: gs://holberton-iris-demo-data/raw/iris.csv")
    
    return df

df = prepare_iris_dataset()
```

### 9.3 Créer le script d'entraînement

```python
# trainer/task.py
import argparse
import pandas as pd
import numpy as np
import pickle
import json
import os
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import LabelEncoder
from sklearn.metrics import classification_report, accuracy_score
from google.cloud import storage

def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, required=True)
    parser.add_argument("--model-dir", type=str, required=True)
    parser.add_argument("--n-estimators", type=int, default=100)
    parser.add_argument("--max-depth", type=int, default=5)
    return parser.parse_args()

def upload_to_gcs(local_path: str, gcs_path: str):
    """Uploade un fichier local vers GCS."""
    client = storage.Client()
    bucket_name, blob_path = gcs_path.replace("gs://", "").split("/", 1)
    bucket = client.bucket(bucket_name)
    blob = bucket.blob(blob_path)
    blob.upload_from_filename(local_path)
    print(f"Uploadé: {gcs_path}")

def main():
    args = parse_args()
    
    # Charger les données
    df = pd.read_csv(args.data_path)
    print(f"Données chargées: {df.shape}")
    
    FEATURE_COLS = ["sepal_length", "sepal_width", "petal_length", "petal_width"]
    X = df[FEATURE_COLS].values
    y = df["species"].values
    
    # Split
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42, stratify=y
    )
    
    # Entraînement
    model = RandomForestClassifier(
        n_estimators=args.n_estimators,
        max_depth=args.max_depth,
        random_state=42,
        n_jobs=-1
    )
    model.fit(X_train, y_train)
    
    # Évaluation
    y_pred = model.predict(X_test)
    accuracy = accuracy_score(y_test, y_pred)
    cv_scores = cross_val_score(model, X, y, cv=5, scoring="accuracy")
    
    print(f"\n=== Résultats ===")
    print(f"Accuracy test: {accuracy:.4f}")
    print(f"CV Accuracy: {cv_scores.mean():.4f} ± {cv_scores.std():.4f}")
    print(f"\nRapport détaillé:")
    print(classification_report(y_test, y_pred,
                                target_names=["setosa", "versicolor", "virginica"]))
    
    # Feature importances
    for feat, imp in zip(FEATURE_COLS, model.feature_importances_):
        print(f"  {feat}: {imp:.4f}")
    
    # Sauvegarder le modèle et les métriques
    os.makedirs("/tmp/model_artifacts", exist_ok=True)
    
    with open("/tmp/model_artifacts/model.pkl", "wb") as f:
        pickle.dump(model, f)
    
    metrics = {
        "accuracy_test": float(accuracy),
        "cv_accuracy_mean": float(cv_scores.mean()),
        "cv_accuracy_std": float(cv_scores.std()),
        "n_estimators": args.n_estimators,
        "max_depth": args.max_depth
    }
    with open("/tmp/model_artifacts/metrics.json", "w") as f:
        json.dump(metrics, f, indent=2)
    
    # Uploader les artifacts
    model_gcs_path = os.path.join(args.model_dir, "model.pkl")
    metrics_gcs_path = os.path.join(args.model_dir, "metrics.json")
    
    upload_to_gcs("/tmp/model_artifacts/model.pkl", model_gcs_path)
    upload_to_gcs("/tmp/model_artifacts/metrics.json", metrics_gcs_path)
    
    print(f"\nModèle sauvegardé dans: {args.model_dir}")

if __name__ == "__main__":
    main()
```

### 9.4 Lancer l'entraînement sur Vertex AI

```python
# train_on_vertex.py
from google.cloud import aiplatform

PROJECT = "holberton-iris-demo"
REGION = "europe-west4"
BUCKET = "gs://holberton-iris-demo-data"

aiplatform.init(project=PROJECT, location=REGION)

job = aiplatform.CustomTrainingJob(
    display_name="iris-rf-training-v1",
    script_path="trainer/task.py",
    container_uri=f"{REGION}-docker.pkg.dev/vertex-ai/training/scikit-learn-cpu.1-0:latest",
    requirements=["pandas>=1.3.0", "scikit-learn>=1.0.0", "google-cloud-storage>=2.0.0"],
    model_serving_container_image_uri=f"{REGION}-docker.pkg.dev/vertex-ai/prediction/sklearn-cpu.1-0:latest"
)

model = job.run(
    args=[
        "--data-path", f"{BUCKET}/raw/iris.csv",
        "--model-dir", f"{BUCKET}/models/iris-rf-v1/",
        "--n-estimators", "150",
        "--max-depth", "7",
    ],
    replica_count=1,
    machine_type="n1-standard-2",
    base_output_dir=f"{BUCKET}/training-output/",
    model_display_name="iris-random-forest-v1",
    sync=True
)

print(f"Modèle enregistré dans Vertex AI: {model.resource_name}")
```

### 9.5 Déployer et tester l'endpoint

```python
# deploy_and_test.py
from google.cloud import aiplatform
import json

PROJECT = "holberton-iris-demo"
REGION = "europe-west4"

aiplatform.init(project=PROJECT, location=REGION)

# Récupérer le modèle (depuis le step précédent ou par nom)
models = aiplatform.Model.list(filter="display_name=iris-random-forest-v1")
model = models[0]
print(f"Modèle trouvé: {model.display_name} ({model.resource_name})")

# Créer l'endpoint
endpoint = aiplatform.Endpoint.create(
    display_name="iris-classifier-endpoint",
    description="Classification Iris - Demo Holberton"
)

# Déployer le modèle
endpoint.deploy(
    model=model,
    deployed_model_display_name="iris-rf-v1-deployed",
    machine_type="n1-standard-2",
    min_replica_count=1,
    max_replica_count=3,
    traffic_percentage=100
)

print(f"Endpoint déployé: {endpoint.resource_name}")

# Test de prédiction
test_instances = [
    [5.1, 3.5, 1.4, 0.2],   # setosa (espèce 0)
    [6.7, 3.1, 4.7, 1.5],   # versicolor (espèce 1)
    [6.3, 2.7, 4.9, 1.8],   # virginica (espèce 2)
]

species_map = {0: "setosa", 1: "versicolor", 2: "virginica"}

predictions = endpoint.predict(instances=test_instances)
print("\n=== Prédictions ===")
for i, (instance, pred) in enumerate(zip(test_instances, predictions.predictions)):
    espece_predite = species_map[int(pred)]
    print(f"Instance {i+1}: {instance} → {espece_predite}")

# Nettoyer (IMPORTANT pour éviter les coûts)
# endpoint.undeploy_all()
# endpoint.delete()
```

### 9.6 Monitoring et logs

```bash
# Voir les logs d'un training job
gcloud logging read \
  'resource.type="ml_job"' \
  --project=holberton-iris-demo \
  --limit=50 \
  --format="value(textPayload)"

# Voir les métriques d'un endpoint (requêtes, latence)
gcloud monitoring metrics-descriptors list \
  --filter="metric.type:aiplatform"

# Voir les prédictions dans les logs
gcloud logging read \
  'resource.type="aiplatform.googleapis.com/Endpoint"' \
  --project=holberton-iris-demo \
  --limit=20
```

---

## 10. Coûts — Estimation, Free Tier et Optimisation

### 10.1 Free Tier GCP

Google offre plusieurs niveaux de gratuité :

**Crédit de démarrage :**
- **300 $ de crédits** valables 90 jours pour les nouveaux comptes
- Suffisant pour suivre entièrement ce cours

**Free Tier permanent (always free) :**

| Service | Quota gratuit mensuel |
|---|---|
| **GCS** | 5 GB stockage, 1 GB/mois trafic sortant vers certaines régions |
| **BigQuery** | 10 GB stockage, 1 TB de requêtes |
| **Cloud Run** | 2 millions de requêtes, 360 000 GB-secondes |
| **Cloud Build** | 120 minutes de build |
| **Artifact Registry** | 0.5 GB stockage |

### 10.2 Estimation des coûts d'un projet ML type

```
Scénario : Projet ML académique (6 semaines)

Stockage GCS
  - 10 GB dataset + modèles (Standard)   → 0.20 $/mois × 1.5 mois = 0.30 $

BigQuery
  - Développement : 50 Go de requêtes    → 50 × 0.005 $ = 0.25 $
  (les 1 premiers TB/mois sont gratuits → 0 $ si < 1 TB)

Vertex AI Training
  - 3 training jobs × 2h × n1-standard-4 (0.19 $/h) → 1.14 $
  - 1 training job avec GPU T4 × 4h (0.35 $/h)       → 1.40 $

Vertex AI Endpoint
  - n1-standard-2 × 10h de test (0.09 $/h)           → 0.90 $
  ⚠️ À supprimer après les tests !

Vertex Workbench
  - n1-standard-4 × 20h (0.19 $/h)                   → 3.80 $
  ⚠️ Arrêter quand inactif !

Cloud Run
  - Inclus dans le free tier → 0 $

TOTAL ESTIMÉ : ~7.79 $ pour tout le cours
Avec 300 $ de crédits → largement couvert
```

### 10.3 Outils de gestion des coûts

```bash
# Voir la facture actuelle
gcloud billing projects list

# Créer une alerte de budget (RECOMMANDÉ dès le premier jour)
# Via Console : Billing → Budgets & Alerts → Create Budget
# Mettre une alerte à 50 $ pour ne jamais dépasser les crédits

# Voir la consommation par service
gcloud billing accounts list
```

```python
# Estimer le coût d'une requête BigQuery AVANT de l'exécuter
from google.cloud import bigquery

client = bigquery.Client(project="holberton-iris-demo")

# dry_run=True n'exécute pas la requête mais calcule les bytes à scanner
job_config = bigquery.QueryJobConfig(dry_run=True, use_query_cache=False)

query = """
    SELECT *
    FROM `mon-projet-ml-2024.ml_features.transactions`
    WHERE DATE(timestamp) >= '2024-01-01'
"""

job = client.query(query, job_config=job_config)
bytes_scanned = job.total_bytes_processed
cost_usd = (bytes_scanned / 1e12) * 5  # $5 par TB

print(f"Bytes à scanner : {bytes_scanned / 1e9:.2f} GB")
print(f"Coût estimé : ${cost_usd:.4f}")
```

### 10.4 Optimisations coûts avancées

```bash
# 1. Utiliser des Spot VMs (= preemptible) pour les training jobs (70% moins cher)
# Les Spot VMs peuvent être interrompues par Google, mais les training jobs Vertex AI
# reprennent automatiquement depuis le dernier checkpoint

# Dans le CustomTrainingJob, ajouter:
# worker_pool_specs=[{
#   ...
#   "machine_spec": {
#     "machine_type": "n1-standard-4",
#     "accelerator_type": "NVIDIA_TESLA_T4",
#     "accelerator_count": 1
#   },
#   "disk_spec": {"boot_disk_type": "PD_SSD", "boot_disk_size_gb": 100}
# }]
# Et dans le CustomJob: enable_web_access=True, scheduling={"timeout": "3600s"}

# 2. Committed Use Discounts pour les ressources permanentes
# -57% sur les VMs avec engagement 3 ans (inutile pour les étudiants)

# 3. Politique de cycle de vie sur GCS (nettoyer automatiquement les vieux fichiers)
gsutil lifecycle set lifecycle.json gs://holberton-iris-demo-data/
```

```json
// lifecycle.json
{
  "rule": [
    {
      "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
      "condition": {
        "age": 30,
        "matchesPrefix": ["training-output/"]
      }
    },
    {
      "action": {"type": "Delete"},
      "condition": {
        "age": 180,
        "matchesPrefix": ["tmp/"]
      }
    }
  ]
}
```

---

## 11. Bonnes Pratiques et Pièges à Éviter

### 11.1 Sécurité

> [!warning] Les 5 erreurs de sécurité les plus fréquentes sur GCP
> 1. **Bucket GCS public** — ne mettre `allUsers` en lecture que sur des données vraiment publiques
> 2. **Clés Service Account commitées** — utiliser `.gitignore` dès le premier jour
> 3. **Permissions trop larges** — éviter `roles/owner` sur les SA, préférer des rôles granulaires
> 4. **Endpoint sans authentification** — `--allow-unauthenticated` est ok en dev, jamais en prod
> 5. **Pas d'alerte de budget** — sans alerte, une boucle infinie peut vous coûter des centaines d'euros

### 11.2 Checklist avant un Training Job

```
☐ Le dataset est dans GCS (et non sur votre machine locale) ?
☐ Le machine_type est adapté au workload (pas un n1-highmem-32 pour un CSV de 10 MB) ?
☐ sync=False si le job prend plus de 30 min (ne pas bloquer votre terminal) ?
☐ Le model_dir dans GCS existe et est accessible ?
☐ Les métriques sont loggées pour être visibles dans Vertex AI Experiments ?
☐ Un budget max est défini (budget_milli_node_hours) pour AutoML ?
```

### 11.3 Checklist avant un déploiement Endpoint

```
☐ Le modèle est dans le Model Registry avec ses métadonnées ?
☐ Le serving container est compatible avec l'artefact du modèle ?
☐ min_replica_count=0 en dev pour éviter les coûts permanents ?
☐ Un health check est implémenté sur /health ?
☐ L'endpoint sera supprimé après les tests ?
☐ Les logs sont activés pour monitorer les prédictions ?
```

---

## 12. Exercices Pratiques

### Exercice 1 — Prise en main GCP (1h)

**Objectif :** Configurer votre environnement GCP et tester les outils de base.

1. Créer un projet GCP nommé `holberton-ml-[votre-prenom]`
2. Activer les APIs : Storage, BigQuery, Vertex AI
3. Créer un bucket GCS dans `europe-west4`
4. Télécharger le dataset Titanic depuis Kaggle et l'uploader dans GCS avec gsutil
5. Créer une table BigQuery depuis ce CSV et compter le taux de survie par classe

**Livrable :** Screenshot de la table BigQuery avec les résultats de la requête.

### Exercice 2 — BigQuery ML (2h)

**Objectif :** Entraîner un modèle de classification sans Python.

1. Charger le dataset Titanic dans BigQuery
2. Créer un modèle `LOGISTIC_REG` pour prédire la survie
3. Évaluer le modèle avec `ML.EVALUATE`
4. Comparer avec un `BOOSTED_TREE_CLASSIFIER`
5. Faire des prédictions sur 5 nouvelles instances avec `ML.PREDICT`

```sql
-- Point de départ
CREATE OR REPLACE MODEL `votre-projet.titanic.modele_survie`
OPTIONS(
  model_type = 'LOGISTIC_REG',
  input_label_cols = ['survived']
) AS
SELECT
  -- À compléter : choisir les features pertinentes
  pclass,
  -- ...
  survived
FROM `votre-projet.titanic.passagers`;
```

### Exercice 3 — Custom Training + Endpoint (3h)

**Objectif :** Pipeline complet d'un bout à l'autre.

1. Prendre le dataset Wine Quality (UCI ML Repository)
2. Écrire un script d'entraînement (`trainer/task.py`) avec scikit-learn
3. Lancer un Custom Training Job sur Vertex AI (n1-standard-2, sans GPU)
4. Vérifier que le modèle apparaît dans le Model Registry
5. Déployer sur un Endpoint et tester avec 3 instances de test
6. **Important :** Supprimer l'endpoint à la fin pour éviter les frais

### Challenge — Pipeline Complet avec Kubeflow (4h)

**Objectif :** Construire un pipeline Vertex AI avec 3 composants.

Construire un pipeline qui :
1. **Composant 1 :** Charge et nettoie les données depuis GCS (supprime les NaN, normalise)
2. **Composant 2 :** Entraîne un Random Forest et sauvegarde le modèle + les métriques
3. **Composant 3 :** Vérifie si l'accuracy > 0.85, et si oui déploie sur un endpoint

Utiliser `enable_caching=True` et vérifier que relancer le pipeline réutilise bien le cache pour les étapes non modifiées.

---

## Récapitulatif des Commandes Essentielles

```bash
# CONFIGURATION
gcloud config set project MON_PROJET
gcloud auth application-default login

# GCS
gsutil mb -l europe-west4 gs://MON_BUCKET/
gsutil -m cp -r ./data/ gs://MON_BUCKET/raw/
gsutil du -sh gs://MON_BUCKET/

# BIGQUERY
bq mk --dataset MON_PROJET:MON_DATASET
bq load --autodetect MON_PROJET:MON_DATASET.MA_TABLE gs://MON_BUCKET/data.csv
bq query --use_legacy_sql=false 'SELECT COUNT(*) FROM `MON_PROJET.MON_DATASET.MA_TABLE`'

# VERTEX AI
gcloud ai custom-jobs list --region=europe-west4
gcloud ai endpoints list --region=europe-west4
gcloud ai models list --region=europe-west4

# CLOUD RUN
gcloud run deploy MON_SERVICE --image IMAGE_URI --region europe-west4
gcloud run services list --region europe-west4
gcloud run services delete MON_SERVICE --region europe-west4

# NETTOYAGE (économiser de l'argent)
gcloud ai endpoints delete ENDPOINT_ID --region=europe-west4
gcloud notebooks instances stop MON_NOTEBOOK --location=europe-west4-a
```

> [!tip] Ressources pour aller plus loin
> - **Documentation officielle Vertex AI** : cloud.google.com/vertex-ai/docs
> - **Codelabs GCP ML** : codelabs.developers.google.com (chercher "Vertex AI")
> - **Qwiklabs / Cloud Skills Boost** : coursera des labs pratiques GCP, souvent inclus dans les programmes académiques
> - **GitHub google-cloud-python** : exemples officiels du SDK Python
> - **TFX (TensorFlow Extended)** : si vous plongez dans les pipelines ML avancés avec TensorFlow
