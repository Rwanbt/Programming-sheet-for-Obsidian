# 02 - Apache Spark et PySpark

> [!info] Objectifs
> Maîtriser Apache Spark depuis les RDDs jusqu'aux DataFrames, le streaming, MLlib et les optimisations avancées. PySpark est l'API Python de Spark — production-grade dans l'industrie Data.

## 1. Spark vs MapReduce

### Pourquoi Spark a remplacé MapReduce

```
MapReduce :
  Input HDFS → Map → Write disk → Shuffle → Read disk → Reduce → Write HDFS
                    ↑ BOTTLENECK : chaque étape intermédiaire écrit sur disque

Spark :
  Input HDFS → Stage 1 → Stage 2 → Stage 3 → Write HDFS
               ↑ Tout en mémoire (RAM). Disque uniquement si spill ou checkpoint.
```

**Chiffres clés** :
- 10x plus rapide que MapReduce sur disque
- 100x plus rapide en mémoire (si les données tiennent)
- Même API pour batch, streaming, ML et graph

### Avantages de Spark

| Fonctionnalité | MapReduce | Spark |
|---------------|-----------|-------|
| In-memory processing | Non | Oui |
| DAG optimizer | Non | Oui (Catalyst) |
| Streaming | Non (micro-batch avec Storm) | Natif |
| Machine Learning | Non | MLlib |
| Graph computing | Non | GraphX |
| API languages | Java | Scala, Java, Python, R, SQL |
| Latence itérative (ML) | Très élevée | Faible (réutilise les données cachées) |

**DAG Optimizer (Catalyst)** : Spark construit un plan logique, l'optimise (push-down predicates, projection pruning, join reordering), génère un plan physique optimal avant l'exécution. C'est l'équivalent de l'optimizer SQL d'une base relationnelle, mais pour le Big Data distribué.

---

## 2. Architecture Spark

### Composants principaux

```
                      ┌─────────────────────────┐
                      │        Driver           │
                      │  - SparkContext/Session  │
                      │  - DAG Scheduler         │
                      │  - Task Scheduler        │
                      │  - Code utilisateur      │
                      └────────────┬────────────┘
                                   │
                     ┌─────────────▼──────────────┐
                     │      Cluster Manager        │
                     │  (YARN / K8s / Standalone)  │
                     └──┬──────────────────────┬───┘
                        │                      │
               ┌────────▼─────┐      ┌─────────▼────┐
               │  Executor 1  │      │  Executor 2  │
               │  - Tasks     │      │  - Tasks     │
               │  - Cache     │      │  - Cache     │
               │  - Shuffle   │      │  - Shuffle   │
               └──────────────┘      └──────────────┘
```

**Driver** : processus principal qui héberge votre code Spark
- Crée le SparkContext (API bas niveau) ou SparkSession (API unifiée)
- Traduit les transformations en DAG de stages
- Schedule les tasks sur les executors via le Cluster Manager

**Executors** : processus JVM sur les worker nodes
- Exécutent les tasks (unité atomique de travail = traitement d'une partition)
- Stockent les données en cache (mémoire + disque si overflow)
- Gèrent les shuffles (échange de données entre stages)

**Cluster Manager** : allocation des ressources (CPU + RAM)
- Standalone : cluster Spark natif (développement, petits déploiements)
- YARN : intégration Hadoop (production Hadoop)
- Kubernetes : déploiement cloud-native (tendance actuelle)
- Mesos : moins utilisé, legacy

### SparkSession — Point d'entrée

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MonApplication") \
    .master("local[*]") \                      # local[*] = tous les cœurs locaux
    .config("spark.executor.memory", "4g") \
    .config("spark.executor.cores", "2") \
    .config("spark.sql.shuffle.partitions", "200") \
    .getOrCreate()

# Accéder au SparkContext sous-jacent
sc = spark.sparkContext

# Arrêter proprement
spark.stop()
```

---

## 3. RDDs — Resilient Distributed Datasets

### Concept fondamental

Un RDD est une collection distribuée immuable, partitionnée sur les nœuds du cluster. L'API bas niveau de Spark.

**Propriétés** :
- **Resilient** : reconstruction automatique depuis le lineage en cas de perte de partition
- **Distributed** : les partitions sont sur différents nœuds
- **Dataset** : collection typée d'objets Python/Scala/Java

```python
# Créer un RDD
rdd_from_list = sc.parallelize([1, 2, 3, 4, 5], numSlices=4)  # 4 partitions
rdd_from_file = sc.textFile("/data/logs/*.log")                 # Une partition par bloc HDFS

print(rdd_from_list.getNumPartitions())  # 4
```

### Transformations (lazy)

Les transformations ne s'exécutent pas immédiatement — elles construisent le DAG.

```python
# map : applique une fonction à chaque élément, retourne 1 élément
squared = rdd_from_list.map(lambda x: x * x)
# [1, 4, 9, 16, 25]

# filter : garde les éléments qui satisfont le prédicat
evens = rdd_from_list.filter(lambda x: x % 2 == 0)
# [2, 4]

# flatMap : applique une fonction qui retourne plusieurs éléments, aplatit le résultat
words = sc.textFile("/data/text.txt").flatMap(lambda line: line.split(" "))

# reduceByKey : agrège les valeurs par clé (paires k,v → paires k,v)
word_counts = words.map(lambda w: (w, 1)).reduceByKey(lambda a, b: a + b)

# groupByKey : groupe toutes les valeurs par clé (moins efficace que reduceByKey !)
grouped = pairs_rdd.groupByKey()  # Attention : transfère TOUTES les valeurs en réseau

# sortByKey : tri par clé
sorted_rdd = word_counts.sortByKey(ascending=False)

# join : jointure de deux RDDs de paires (k,v)
rdd1 = sc.parallelize([("a", 1), ("b", 2)])
rdd2 = sc.parallelize([("a", "x"), ("b", "y"), ("c", "z")])
joined = rdd1.join(rdd2)  # [("a", (1, "x")), ("b", (2, "y"))]

# union, intersection, subtract, cartesian
combined = rdd1.union(rdd2)
```

> [!warning] `groupByKey` vs `reduceByKey`
> `groupByKey` transfère TOUTES les valeurs sur le réseau puis agrège. `reduceByKey` fait une agrégation locale (comme un combiner) puis une agrégation finale. Préférer `reduceByKey` quand possible : 10x moins de données réseau.

### Actions (déclenchent l'exécution)

```python
# collect : ramène toutes les données dans le driver (DANGER avec gros RDDs)
result = word_counts.collect()  # → list Python dans le driver

# count : nombre d'éléments
n = word_counts.count()

# take : prendre les N premiers éléments
first_5 = word_counts.take(5)

# first : premier élément
first = word_counts.first()

# reduce : agrégation globale
total = rdd_from_list.reduce(lambda a, b: a + b)  # 15

# saveAsTextFile : écrire sur HDFS/S3
word_counts.saveAsTextFile("/data/output/wordcount/")

# countByKey : dict du nombre de valeurs par clé
counts = pairs_rdd.countByKey()

# foreach : appliquer une fonction sur chaque élément (effets de bord)
word_counts.foreach(lambda x: print(x))  # Exécuté sur les executors, pas driver
```

### Lazy Evaluation et Lineage Graph

```python
# Aucun calcul ne se passe ici :
rdd1 = sc.textFile("/data/huge_file.txt")    # Lazy
rdd2 = rdd1.filter(lambda l: "ERROR" in l)  # Lazy
rdd3 = rdd2.map(lambda l: l.split("|"))     # Lazy

# L'exécution se lance uniquement ici :
result = rdd3.count()  # Action → déclenche le calcul
```

**Lineage Graph** : Spark sait comment reconstruire n'importe quelle partition perdue en rejouant les transformations depuis la source. C'est le mécanisme de fault tolerance sans checkpoint disque.

---

## 4. DataFrames et Datasets

### DataFrame : l'API structurée

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, avg, count, when, lit, regexp_extract
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DoubleType

spark = SparkSession.builder.appName("DFExample").getOrCreate()

# Lecture de données
df = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("/data/sales.csv")

# Schéma explicite (préférable pour la production)
schema = StructType([
    StructField("sale_id", IntegerType(), nullable=False),
    StructField("customer_id", IntegerType(), nullable=True),
    StructField("product", StringType(), nullable=True),
    StructField("amount", DoubleType(), nullable=True),
    StructField("sale_date", StringType(), nullable=True)
])
df = spark.read.csv("/data/sales.csv", schema=schema, header=True)

df.printSchema()
df.show(5, truncate=False)
df.describe().show()
```

### Opérations essentielles

```python
# select : choisir des colonnes
df.select("customer_id", "amount").show()
df.select(col("amount") * 1.2).alias("amount_ttc").show()

# filter / where : filtrer les lignes
df.filter(col("amount") > 100).show()
df.where((col("amount") > 100) & (col("product") != "PROMO")).show()

# withColumn : ajouter/modifier une colonne
df = df.withColumn("amount_eur", col("amount") * 0.92)
df = df.withColumn("is_high_value", when(col("amount") > 500, True).otherwise(False))

# groupBy + agg
from pyspark.sql.functions import sum, max, min, countDistinct

df.groupBy("product") \
  .agg(
      count("*").alias("nb_sales"),
      sum("amount").alias("total_revenue"),
      avg("amount").alias("avg_ticket"),
      countDistinct("customer_id").alias("unique_customers")
  ) \
  .orderBy(col("total_revenue").desc()) \
  .show()

# join
customers_df = spark.read.parquet("/data/customers/")
result = df.join(customers_df, on="customer_id", how="left")
# how: "inner", "left", "right", "full", "left_semi", "left_anti", "cross"

# drop, rename
df = df.drop("obsolete_column")
df = df.withColumnRenamed("old_name", "new_name")

# distinct, dropDuplicates
df_unique = df.dropDuplicates(["customer_id", "product"])
```

### SQL avec Spark

```python
# Enregistrer un DataFrame comme vue temporaire
df.createOrReplaceTempView("sales")

# Requête SQL standard
result = spark.sql("""
    SELECT
        product,
        COUNT(*) as nb_sales,
        SUM(amount) as revenue,
        AVG(amount) as avg_ticket,
        RANK() OVER (ORDER BY SUM(amount) DESC) as revenue_rank
    FROM sales
    WHERE sale_date >= '2024-01-01'
    GROUP BY product
    HAVING COUNT(*) > 100
    ORDER BY revenue DESC
    LIMIT 20
""")
result.show()

# Vue globale (accessible entre SparkSessions)
df.createGlobalTempView("global_sales")
spark.sql("SELECT * FROM global_temp.global_sales").show()
```

---

## 5. PySpark pratique — Lecture et écriture

### Formats de fichiers

```python
# CSV
df = spark.read.csv("/data/file.csv", header=True, sep=";", encoding="UTF-8",
                    nullValue="N/A", inferSchema=True)

# JSON
df = spark.read.json("/data/events/")  # Chaque ligne = un objet JSON

# Parquet (format columnar recommandé)
df = spark.read.parquet("/data/processed/")
df.write.parquet("/data/output/", mode="overwrite")  # mode: overwrite, append, ignore, error

# ORC (alternative Parquet, optimisé pour Hive)
df = spark.read.orc("/data/hive/table/")

# JDBC (base de données relationnelle)
df = spark.read.jdbc(
    url="jdbc:postgresql://host:5432/mydb",
    table="orders",
    properties={"user": "admin", "password": "secret", "driver": "org.postgresql.Driver"},
    numPartitions=10,
    partitionColumn="order_id",
    lowerBound=1,
    upperBound=1000000
)

# Delta Lake (ACID sur Parquet)
df = spark.read.format("delta").load("/data/delta/sales/")
df.write.format("delta").mode("append").save("/data/delta/sales/")
```

### Parquet — Pourquoi c'est le standard

```
CSV (row-based):          Parquet (column-based):
┌────┬────┬────┐          ┌──────────┐ ┌──────────┐ ┌──────────┐
│ A  │ B  │ C  │          │Col A: 1,2│ │Col B: x,y│ │Col C: π,e│
│ 1  │ x  │ π  │    →     │  3 bytes │ │  3 bytes │ │  3 bytes │
│ 2  │ y  │ e  │          └──────────┘ └──────────┘ └──────────┘
└────┴────┴────┘
                          SELECT A FROM table → lit SEULEMENT la colonne A
                          Compression ~10x (données similaires dans une colonne)
```

---

## 6. Optimisation Spark

### `explain()` — Comprendre le plan d'exécution

```python
# Plan logique et physique
df.groupBy("product").agg(sum("amount")).explain()
df.groupBy("product").agg(sum("amount")).explain(extended=True)  # Détails complets
df.groupBy("product").agg(sum("amount")).explain(mode="formatted")  # Lisible

# Exemple de sortie :
# == Physical Plan ==
# AdaptiveSparkPlan isFinalPlan=false
# +- HashAggregate(keys=[product], functions=[sum(amount)])
#    +- Exchange hashpartitioning(product, 200)  ← Shuffle !
#       +- HashAggregate(keys=[product], functions=[partial_sum(amount)])
#          +- FileScan parquet [product, amount]  ← Lecture sélective colonnes
```

### Caching et Persistence

```python
from pyspark import StorageLevel

# Cache en mémoire (défaut)
df.cache()
df.count()  # Déclenche le calcul et met en cache

# Persistence avec niveau explicite
df.persist(StorageLevel.MEMORY_AND_DISK)  # Mémoire, puis disque si overflow
df.persist(StorageLevel.DISK_ONLY)         # Seulement sur disque
df.persist(StorageLevel.MEMORY_ONLY_2)    # Mémoire, répliqué sur 2 nœuds

# Libérer le cache
df.unpersist()
spark.catalog.clearCache()  # Vider tout le cache
```

> [!tip] Quand mettre en cache ?
> Mettre en cache un DataFrame qui est réutilisé plusieurs fois dans votre pipeline. Ne pas cacher les DataFrames volumineux utilisés une seule fois — cela consomme de la mémoire inutilement.

### Broadcast Join

```python
from pyspark.sql.functions import broadcast

# Table de dimension petite (< 10 MB par défaut)
countries_df = spark.read.csv("/data/countries.csv", header=True)  # ~200 lignes

# Sans broadcast : shuffle des deux côtés du join (cher)
result = large_df.join(countries_df, on="country_code")

# Avec broadcast : envoie countries_df sur tous les executors, zero shuffle
result = large_df.join(broadcast(countries_df), on="country_code")

# Configuration automatique (AQE peut aussi décider de broadcaster)
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", "100m")  # Seuil auto-broadcast
```

### Repartition vs Coalesce

```python
# repartition : répartit les données sur N partitions (déclenche un shuffle)
df = df.repartition(100)                           # 100 partitions équilibrées
df = df.repartition(50, "country_code")            # 50 partitions, par colonne

# coalesce : réduit le nombre de partitions SANS shuffle (fusionne des partitions locales)
df = df.coalesce(10)   # Réduire de 100 à 10 partitions
# Utiliser coalesce pour réduire le nombre de fichiers à l'écriture

df.coalesce(1).write.csv("/data/output/single_file/")  # Un seul fichier (attention : tout au driver)

# Vérifier le nombre de partitions
print(df.rdd.getNumPartitions())
```

### Data Skew — Gestion des déséquilibres

```python
# Problème : une clé a 90% des données → un executor fait tout le travail
# Exemple : un seul pays représente 90% des commandes

# Solution 1 : Salting (ajouter un préfixe aléatoire)
from pyspark.sql.functions import concat, lit, (col, monotonically_increasing_id
import random

# Diviser la grosse clé en sous-clés
def add_salt(df, n=10):
    return df.withColumn("salted_key",
        concat(col("skewed_key"), lit("_"), (monotonically_increasing_id() % n).cast("string"))
    )

# Solution 2 : Broadcast si la petite table est < seuil
# Solution 3 : AQE (Adaptive Query Execution) — automatique depuis Spark 3.x
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
```

### AQE — Adaptive Query Execution (Spark 3.x)

```python
# Activer AQE (par défaut depuis Spark 3.2)
spark.conf.set("spark.sql.adaptive.enabled", "true")

# AQE features :
# 1. Nombre de partitions post-shuffle dynamique (évite 200 fichiers minuscules)
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")

# 2. Conversion automatique sort-merge join → broadcast join si possible
spark.conf.set("spark.sql.adaptive.localShuffleReader.enabled", "true")

# 3. Gestion automatique du skew (divise les partitions larges)
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes", "256m")
```

---

## 7. Spark Structured Streaming

### Micro-batch vs Continuous Processing

```
Micro-batch (défaut) :
  Source → Trigger (every 5s) → Process batch → Sink → Checkpoint
  Latence : secondes. Garanties : exactly-once.

Continuous Processing (expérimental) :
  Source → Process record-by-record → Sink
  Latence : <1ms. Garanties : at-least-once seulement.
```

### Pipeline Streaming de base

```python
# Lecture depuis Kafka
kafka_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "kafka1:9092,kafka2:9092") \
    .option("subscribe", "events-topic") \
    .option("startingOffsets", "latest") \
    .load()

# Parser le message JSON
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StructField, StringType, LongType

event_schema = StructType([
    StructField("user_id", StringType()),
    StructField("event_type", StringType()),
    StructField("timestamp", LongType())
])

events_df = kafka_df.select(
    from_json(col("value").cast("string"), event_schema).alias("data")
).select("data.*")

# Transformation
events_per_type = events_df.groupBy("event_type").count()

# Écriture avec watermark et fenêtres temporelles
from pyspark.sql.functions import window, to_timestamp

events_df = events_df.withColumn("ts", to_timestamp(col("timestamp") / 1000))

windowed = events_df \
    .withWatermark("ts", "10 minutes") \
    .groupBy(window("ts", "5 minutes"), "event_type") \
    .count()

# Sink vers console (debug)
query = windowed.writeStream \
    .outputMode("update") \        # update, append, complete
    .format("console") \
    .trigger(processingTime="5 seconds") \
    .start()

# Sink vers Kafka
query = events_per_type.writeStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "kafka1:9092") \
    .option("topic", "output-topic") \
    .option("checkpointLocation", "/data/checkpoints/events/") \
    .start()

query.awaitTermination()
```

### `foreachBatch` — Logique custom à chaque micro-batch

```python
def process_batch(batch_df, batch_id):
    """Appelée pour chaque micro-batch."""
    # Écrire vers PostgreSQL
    batch_df.write.jdbc(
        url="jdbc:postgresql://host/db",
        table="streaming_results",
        mode="append",
        properties={"user": "admin", "password": "secret"}
    )
    # Mettre en cache pour multiple sinks
    batch_df.cache()
    batch_df.write.parquet(f"/data/streaming/{batch_id}/")
    batch_df.unpersist()

query = windowed.writeStream \
    .foreachBatch(process_batch) \
    .option("checkpointLocation", "/data/checkpoints/") \
    .start()
```

---

## 8. MLlib — Machine Learning distribué

### Pipelines ML

```python
from pyspark.ml import Pipeline
from pyspark.ml.feature import StringIndexer, VectorAssembler, StandardScaler
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.evaluation import BinaryClassificationEvaluator
from pyspark.ml.tuning import CrossValidator, ParamGridBuilder

# Charger les données
df = spark.read.parquet("/data/ml/train/")

# Étape 1 : Encoder les catégories
indexer = StringIndexer(inputCols=["category", "region"], outputCols=["cat_idx", "reg_idx"])

# Étape 2 : Assembler les features en un seul vecteur
assembler = VectorAssembler(
    inputCols=["cat_idx", "reg_idx", "amount", "duration", "age"],
    outputCol="features_raw"
)

# Étape 3 : Normaliser
scaler = StandardScaler(inputCol="features_raw", outputCol="features")

# Modèle
rf = RandomForestClassifier(
    featuresCol="features",
    labelCol="is_fraud",
    numTrees=100,
    maxDepth=10
)

# Pipeline = séquence d'estimateurs et transformers
pipeline = Pipeline(stages=[indexer, assembler, scaler, rf])

# Train/test split
train, test = df.randomSplit([0.8, 0.2], seed=42)

# Cross-validation
param_grid = ParamGridBuilder() \
    .addGrid(rf.numTrees, [50, 100, 200]) \
    .addGrid(rf.maxDepth, [5, 10]) \
    .build()

evaluator = BinaryClassificationEvaluator(labelCol="is_fraud", metricName="areaUnderROC")

cv = CrossValidator(
    estimator=pipeline,
    estimatorParamMaps=param_grid,
    evaluator=evaluator,
    numFolds=3
)

# Entraîner
model = cv.fit(train)

# Évaluer
predictions = model.transform(test)
auc = evaluator.evaluate(predictions)
print(f"AUC: {auc:.4f}")

# Sauvegarder
model.write().overwrite().save("/models/fraud_detection_v1/")
```

### Algorithmes disponibles dans MLlib

| Catégorie | Algorithmes |
|-----------|------------|
| Classification | LogisticRegression, RandomForest, GBTClassifier, NaiveBayes, LinearSVC |
| Régression | LinearRegression, RandomForestRegressor, GBTRegressor |
| Clustering | KMeans, BisectingKMeans, GaussianMixture, LDA (topic modeling) |
| Recommandation | ALS (Collaborative Filtering) |
| Feature extraction | TF-IDF, Word2Vec, PCA, StandardScaler, MinMaxScaler |
| Réduction dim. | PCA, SVD |

---

## 9. GraphX — Graphes distribués (intro)

```python
# GraphX est une API Scala/Java — en Python, utiliser GraphFrames (bibliothèque externe)
# pip install graphframes sur le cluster, ou --packages graphframes:graphframes:0.8.2-spark3.2

from graphframes import GraphFrame

vertices = spark.createDataFrame([
    ("a", "Alice", 34),
    ("b", "Bob", 36),
    ("c", "Charlie", 30)
], ["id", "name", "age"])

edges = spark.createDataFrame([
    ("a", "b", "friend"),
    ("b", "c", "follow"),
    ("c", "b", "follow")
], ["src", "dst", "relationship"])

g = GraphFrame(vertices, edges)

# PageRank
results = g.pageRank(resetProbability=0.15, tol=0.01)
results.vertices.select("id", "pagerank").show()

# Connected Components
g.connectedComponents().show()

# Shortest Paths
g.shortestPaths(landmarks=["a", "c"]).show()
```

---

## 10. Delta Lake — ACID sur Parquet

### Pourquoi Delta Lake ?

Le problème avec Parquet brut sur un data lake :
- Pas de transactions → lecture partielle pendant l'écriture
- Pas de rollback en cas d'erreur
- Pas d'historique des modifications
- Schema evolution difficile

```python
# Écrire en Delta (ACID)
df.write.format("delta").mode("overwrite").save("/data/delta/sales/")

# Lire
df = spark.read.format("delta").load("/data/delta/sales/")

# Upsert avec MERGE (idéal pour CDC - Change Data Capture)
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "/data/delta/sales/")

updates_df = spark.read.parquet("/data/updates/")

delta_table.alias("target").merge(
    updates_df.alias("source"),
    "target.sale_id = source.sale_id"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()

# Time Travel — requêter une version passée
df_v3 = spark.read.format("delta") \
    .option("versionAsOf", 3) \
    .load("/data/delta/sales/")

df_yesterday = spark.read.format("delta") \
    .option("timestampAsOf", "2024-01-14 00:00:00") \
    .load("/data/delta/sales/")

# Historique des transactions
delta_table.history().show()

# Schema Evolution (autoriser l'ajout de nouvelles colonnes)
df_new.write.format("delta") \
    .option("mergeSchema", "true") \
    .mode("append") \
    .save("/data/delta/sales/")
```

---

## 11. Déploiement Spark

### spark-submit

```bash
# Mode local (développement)
spark-submit \
    --master local[4] \
    --driver-memory 2g \
    my_script.py

# YARN cluster mode (production Hadoop)
spark-submit \
    --master yarn \
    --deploy-mode cluster \              # cluster = driver sur le cluster
    --num-executors 10 \
    --executor-memory 8g \
    --executor-cores 4 \
    --driver-memory 4g \
    --conf spark.sql.shuffle.partitions=400 \
    --jars /jars/postgresql-jdbc.jar \
    --py-files dependencies.zip \
    my_pipeline.py --input /data/input/ --output /data/output/

# Kubernetes
spark-submit \
    --master k8s://https://k8s-api-server:6443 \
    --deploy-mode cluster \
    --conf spark.kubernetes.container.image=my-registry/spark:3.4.0 \
    --conf spark.kubernetes.namespace=spark-jobs \
    local:///opt/spark/work-dir/my_script.py
```

### Databricks Community Edition

Databricks est la plateforme cloud officielle pour Spark (créée par les auteurs de Spark).

```python
# Spécificités Databricks
# Display enrichi avec visualisations
display(df)

# Magic commands
%sql SELECT * FROM my_table LIMIT 10
%sh ls /dbfs/mnt/my-bucket/
%fs ls /mnt/my-bucket/

# Monter un bucket S3
dbutils.fs.mount(
    source="s3a://my-bucket/",
    mount_point="/mnt/my-bucket",
    extra_configs={"fs.s3a.access.key": "KEY", "fs.s3a.secret.key": "SECRET"}
)
```

---

## 12. Wikilinks et ressources

- [[01 - Hadoop et Ecosysteme MapReduce]] — HDFS, MapReduce, Hive
- [[03 - ML Supervise avec Scikit-Learn]] — ML classique pour comparaison
- [[04 - Kubernetes Introduction]] — Déploiement Spark sur Kubernetes

---

## Exercices Pratiques

### Exercice 1 — DataFrame transformations
```python
# Dataset : ventes e-commerce (sale_id, customer_id, product, category, amount, country, date)
# 1. Charger le CSV avec schéma explicite
# 2. Calculer le top 5 des catégories par CA (chiffre d'affaires)
# 3. Trouver les clients ayant commandé dans plus de 3 pays différents
# 4. Calculer le panier moyen par mois et pays
# 5. Identifier les produits dont les ventes ont augmenté de plus de 20% en MoM
```

### Exercice 2 — Optimisation
```python
# Vous avez ce code qui prend 45 minutes :
result = large_df.join(medium_df, on="product_id") \
                 .join(small_lookup_df, on="category_id") \
                 .groupBy("region", "category") \
                 .agg(sum("amount"))

# 1. Identifier les optimisations possibles
# 2. Utiliser explain() pour analyser le plan
# 3. Appliquer broadcast pour small_lookup_df
# 4. Tester l'impact de spark.sql.shuffle.partitions (200 → 50 → 400)
# 5. Activer AQE et mesurer l'amélioration
```

### Exercice 3 — Streaming
Créer un pipeline qui :
1. Lit des events JSON depuis un socket (`nc -lk 9999`)
2. Parse le JSON : `{"user": "alice", "action": "click", "page": "home"}`
3. Compte les actions par page toutes les 30 secondes
4. Affiche en mode `complete` sur la console
5. Ajouter un watermark de 1 minute et passer en fenêtres glissantes de 1 min / step 30s

### Exercice 4 — MLlib Pipeline
```python
# Dataset : Titanic (survived, pclass, sex, age, sibsp, parch, fare, embarked)
# 1. Nettoyer les valeurs nulles (fillna pour age, mode pour embarked)
# 2. Encoder sex et embarked avec StringIndexer
# 3. Assembler toutes les features avec VectorAssembler
# 4. Tester LogisticRegression et RandomForest
# 5. CrossValidator avec 5 folds et ParamGrid sur maxDepth [3, 5, 10]
# 6. Évaluer sur le test set avec AUC-ROC
```

> [!tip] Performance Spark — Récapitulatif
> 1. Préférer Parquet/ORC à CSV pour la lecture
> 2. Utiliser les filtres tôt (push-down predicates)
> 3. Broadcaster les petites tables de dimension
> 4. Activer AQE pour les optimisations automatiques
> 5. Utiliser `explain()` avant d'optimiser
> 6. Mettre en cache les DataFrames réutilisés
> 7. Éviter `collect()` sur de gros datasets
> 8. Partitionner les données de sortie par colonnes de filtre fréquentes
