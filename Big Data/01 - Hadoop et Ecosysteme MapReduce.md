# 01 - Hadoop et Écosystème MapReduce

> [!info] Objectifs
> Comprendre l'architecture Hadoop, le paradigme MapReduce et l'écosystème complet qui gravite autour de HDFS pour traiter des pétaoctets de données.

## 1. Big Data — Les 3V et les enjeux

### Les 3V fondamentaux

| V | Définition | Exemple |
|---|-----------|---------|
| **Volume** | Quantité de données (TB → PB → EB) | Logs serveur, données IoT, séquences génomiques |
| **Vélocité** | Vitesse de génération et de traitement | Flux Twitter (500 000 tweets/min), transactions bancaires |
| **Variété** | Diversité des formats | JSON, images, vidéos, logs bruts, données relationnelles |

Certains ajoutent 2 V supplémentaires :
- **Véracité** : qualité et fiabilité des données
- **Valeur** : ROI et extraction d'insights utiles

### Scalabilité : horizontale vs verticale

```
Verticale (Scale Up)          Horizontale (Scale Out)
┌────────────────────┐        ┌──────┐  ┌──────┐  ┌──────┐
│  Serveur énorme    │   vs   │ Node │  │ Node │  │ Node │
│  RAM: 2 TB         │        │  1   │  │  2   │  │  3   │
│  CPU: 128 cœurs    │        └──────┘  └──────┘  └──────┘
│  Coût: $$$$$$      │        Coût : $ × N   (commodity hardware)
└────────────────────┘
```

La scalabilité horizontale est le paradigme Hadoop : des milliers de machines bon marché plutôt qu'un seul super-serveur. C'est plus tolérant aux pannes (une machine tombe → les autres continuent) et linéairement extensible.

### Batch vs Streaming vs Micro-batch

| Mode | Description | Latence | Outils |
|------|-----------|---------|-------|
| **Batch** | Traitement de volumes immenses à intervalles réguliers | Minutes → Heures | Hadoop MapReduce, Hive |
| **Streaming** | Traitement événement par événement en temps réel | Millisecondes | Apache Kafka Streams, Flink |
| **Micro-batch** | Mini-batches toutes les N secondes (compromis) | Secondes | Spark Structured Streaming, Storm |

> [!tip] Quand choisir quoi ?
> - **Batch** : rapports mensuels, ETL nuit, ML training
> - **Streaming** : détection de fraude, alertes temps réel, dashboards live
> - **Micro-batch** : dashboards quasi-temps réel, agrégations courtes fenêtres

---

## 2. HDFS — Hadoop Distributed File System

### Architecture HDFS

```
                    ┌──────────────────┐
                    │    NameNode      │  ← Maître (métadonnées)
                    │  (métadonnées)   │    Namespace, inode tree
                    │  - fsimage       │    Localisation des blocs
                    │  - edit log      │
                    └──────┬───────────┘
                           │ heartbeat + block reports
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
   ┌──────────┐    ┌──────────┐    ┌──────────┐
   │DataNode 1│    │DataNode 2│    │DataNode 3│  ← Esclaves (données réelles)
   │ blk_001  │    │ blk_001  │    │ blk_002  │
   │ blk_003  │    │ blk_002  │    │ blk_003  │
   └──────────┘    └──────────┘    └──────────┘
```

**NameNode** : cerveau du cluster
- Stocke le namespace (arbre de fichiers/dossiers)
- Gère la localisation de chaque bloc sur les DataNodes
- Point unique de défaillance → Secondary NameNode (checkpoint) ou HA NameNode (ZooKeeper)

**DataNode** : muscles du cluster
- Stocke les blocs de données réels
- Envoie des heartbeats au NameNode toutes les 3 secondes
- Envoie un block report toutes les heures

### Blocs et réplication

```
Fichier 500 MB → divisé en blocs de 128 MB (défaut)
                  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐
Fichier original: │ Bloc 1 │  │ Bloc 2 │  │ Bloc 3 │  │ Bloc 4 │
                  │ 128 MB │  │ 128 MB │  │ 128 MB │  │ 116 MB │
                  └────────┘  └────────┘  └────────┘  └────────┘

Réplication factor = 3 (défaut)
Bloc 1 → DataNode 1, DataNode 3, DataNode 5
Bloc 2 → DataNode 2, DataNode 1, DataNode 4
...
```

**Stratégie de placement des répliques** (Rack Awareness) :
1. Première réplique : nœud local (ou rack local si distant)
2. Deuxième réplique : rack différent
3. Troisième réplique : même rack que la 2e, nœud différent

Cette stratégie optimise à la fois la tolérance aux pannes (rack entier défaillant) et la bande passante réseau.

### Fault Tolerance

- **DataNode défaillant** : NameNode détecte l'absence de heartbeat après 10 min, re-réplique les blocs orphelins sur d'autres nœuds
- **Corruption de bloc** : checksum MD5 vérifié à chaque lecture, le bloc corrompu est remplacé par une réplique saine
- **NameNode défaillant** : avec HA, un Standby NameNode prend le relais via ZooKeeper (failover automatique < 30 secondes)

### Commandes HDFS essentielles

```bash
# Navigation et listing
hdfs dfs -ls /                          # Lister la racine HDFS
hdfs dfs -ls -R /data/                  # Listing récursif
hdfs dfs -du -s -h /data/logs/          # Taille du dossier (human-readable)

# Création de répertoires
hdfs dfs -mkdir /data/raw
hdfs dfs -mkdir -p /data/processed/2024/01   # Crée les parents si nécessaire

# Transfert de fichiers
hdfs dfs -put localfile.csv /data/raw/       # Upload local → HDFS
hdfs dfs -put -f localfile.csv /data/raw/    # Forcer l'écrasement
hdfs dfs -get /data/raw/file.csv ./          # Download HDFS → local
hdfs dfs -copyFromLocal data/ /data/raw/     # Équivalent -put

# Lecture et manipulation
hdfs dfs -cat /data/raw/file.csv            # Afficher le contenu
hdfs dfs -tail /data/logs/app.log           # Dernières lignes
hdfs dfs -cp /data/raw/ /data/backup/       # Copier dans HDFS
hdfs dfs -mv /data/raw/old/ /data/archive/  # Déplacer
hdfs dfs -rm /data/raw/file.csv             # Supprimer
hdfs dfs -rm -r /data/tmp/                  # Supprimer récursivement

# Administration
hdfs dfs -count -q /data/                   # Quotas et utilisation
hdfs fsck /data/raw/ -files -blocks         # Vérifier l'intégrité
hdfs dfs -chmod 755 /data/shared/           # Permissions Unix
hdfs dfs -chown user:group /data/           # Propriétaire
```

---

## 3. MapReduce — Le paradigme de calcul distribué

### Le paradigme Map → Shuffle/Sort → Reduce

```
Input Data (HDFS)
       │
       ▼
┌─────────────┐
│   MAPPING   │  Chaque mapper traite UN bloc localement
│             │  (data locality → calcul où la donnée est)
│ Map(k1,v1)  │
│ → list(k2,v2)│
└─────────────┘
       │
       ▼
┌─────────────┐
│SHUFFLE/SORT │  Framework automatique : tri par clé,
│             │  envoi réseau vers les reducers appropriés
│  Regrouper  │
│  par clé    │
└─────────────┘
       │
       ▼
┌─────────────┐
│   REDUCING  │
│             │  Reduce(k2, list(v2)) → list(v3)
│  Agréger    │
│  par clé    │
└─────────────┘
       │
       ▼
Output (HDFS)
```

### Word Count classique en Java

```java
// Mapper : lit les lignes, émet (mot, 1)
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    @Override
    public void map(LongWritable key, Text value, Context context)
            throws IOException, InterruptedException {
        String[] words = value.toString().toLowerCase().split("\\s+");
        for (String w : words) {
            if (!w.isEmpty()) {
                word.set(w);
                context.write(word, one);  // Émet ("hadoop", 1), ("est", 1)...
            }
        }
    }
}

// Reducer : reçoit ("hadoop", [1,1,1,1]), somme → ("hadoop", 4)
public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    public void reduce(Text key, Iterable<IntWritable> values, Context context)
            throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}

// Driver
public class WordCount {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "Word Count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(WordCountMapper.class);
        job.setCombinerClass(WordCountReducer.class);  // Optimisation locale
        job.setReducerClass(WordCountReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

### Word Count avec Hadoop Streaming (Python)

```python
# mapper.py
import sys

for line in sys.stdin:
    line = line.strip().lower()
    words = line.split()
    for word in words:
        print(f"{word}\t1")

# reducer.py
import sys
from collections import defaultdict

word_count = defaultdict(int)

for line in sys.stdin:
    word, count = line.strip().split('\t')
    word_count[word] += int(count)

for word, count in sorted(word_count.items()):
    print(f"{word}\t{count}")
```

```bash
# Lancer avec Hadoop Streaming
hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-*.jar \
    -input /data/input/ \
    -output /data/output/ \
    -mapper mapper.py \
    -reducer reducer.py \
    -file mapper.py \
    -file reducer.py
```

### Combiner et Partitioner

**Combiner** : mini-reducer local qui réduit le trafic réseau avant le shuffle. Fonctionne uniquement si l'opération est commutative et associative (somme, max, min — pas la moyenne).

```
Sans combiner: Mapper → réseau → Reducer
  ("hadoop",1), ("hadoop",1), ("hadoop",1) → réseau → Reducer reçoit 3 paires

Avec combiner: Mapper → Combiner local → réseau → Reducer
  ("hadoop",1), ("hadoop",1), ("hadoop",1) → Combiner → ("hadoop",3) → réseau
```

**Partitioner** : détermine quel reducer reçoit quelle clé. Le `HashPartitioner` par défaut : `hash(key) % numReducers`. Partitioner personnalisé pour éviter le data skew.

---

## 4. YARN — Yet Another Resource Negotiator

### Architecture YARN

```
┌──────────────────────────────────────┐
│         ResourceManager              │
│  - Scheduler (FIFO/Capacity/Fair)    │
│  - ApplicationsManager               │
└──────────┬───────────────────────────┘
           │
    ┌──────┴──────┐
    │             │
┌───▼───┐    ┌───▼───┐
│Node   │    │Node   │   NodeManagers sur chaque machine
│Manager│    │Manager│   Gèrent les containers (CPU + RAM)
│       │    │       │
│ App   │    │ App   │   ApplicationMasters (un par job)
│Master │    │Master │   Négocient les resources, suivent le job
└───────┘    └───────┘
```

**ResourceManager** : allocateur de ressources global du cluster
- **Scheduler** : décide qui obtient des containers
- **ApplicationsManager** : démarre les ApplicationMasters, gère leur cycle de vie

**NodeManager** : agent sur chaque worker node
- Lance et surveille les containers (JVM isolées avec ressources limitées)
- Rapporte la disponibilité des ressources au ResourceManager

**ApplicationMaster** : un par job soumis
- Négocie les containers avec le ResourceManager
- Surveille l'avancement des tâches, gère les relances en cas d'échec

### Schedulers YARN

| Scheduler | Comportement | Cas d'usage |
|-----------|-------------|-------------|
| **FIFO** | Premier arrivé, premier servi — simple | Développement, cluster mono-utilisateur |
| **Capacity Scheduler** | Queues avec capacités garanties (ex: 60% prod, 40% dev) | Multi-tenant entreprise (défaut Hadoop) |
| **Fair Scheduler** | Resources partagées équitablement entre jobs actifs | Multi-utilisateurs avec priorités dynamiques |

---

## 5. Hive — SQL sur Hadoop

### HiveQL — SQL-like sur HDFS

```sql
-- Créer une base de données
CREATE DATABASE analytics;
USE analytics;

-- Table externe (données restent dans HDFS, DROP ne supprime pas les fichiers)
CREATE EXTERNAL TABLE logs (
    timestamp STRING,
    user_id   INT,
    action    STRING,
    duration  FLOAT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/data/raw/logs/';

-- Table interne (données gérées par Hive, DROP supprime tout)
CREATE TABLE user_summary AS
SELECT user_id, COUNT(*) AS actions, AVG(duration) AS avg_duration
FROM logs
GROUP BY user_id;
```

### Partitionnement et Bucketing

```sql
-- Table partitionnée (dossiers séparés par date → pruning des scans)
CREATE TABLE events (
    event_id BIGINT,
    user_id  INT,
    event    STRING
)
PARTITIONED BY (year INT, month INT, day INT)
STORED AS ORC;

-- Insérer dans une partition spécifique
INSERT INTO events PARTITION (year=2024, month=1, day=15)
SELECT event_id, user_id, event FROM events_raw WHERE ...;

-- Bucketing (distribution dans N fichiers par hash d'une colonne)
CREATE TABLE users_bucketed (
    user_id INT,
    name    STRING
)
CLUSTERED BY (user_id) INTO 32 BUCKETS
STORED AS ORC;
```

### Hive vs SQL traditionnel

| Aspect | Hive | PostgreSQL/MySQL |
|--------|------|-----------------|
| Latence | Minutes (batch) | Millisecondes |
| Volume | Pétaoctets | Gigaoctets → Téraoctets |
| ACID | Limité (ORC + transactions) | Complet |
| Schéma | Schema-on-read | Schema-on-write |
| Scalabilité | Horizontale (cluster) | Verticale surtout |
| Mise à jour | Difficile (immutable files) | Native |

**Metastore** : base de données relationnelle (Derby ou MySQL/PostgreSQL) qui stocke les métadonnées des tables Hive. Partageable entre Hive, Spark et Presto.

---

## 6. Pig — Langage Dataflow

### Pourquoi Pig ?

Pig est un langage de transformation de données de plus haut niveau que MapReduce, mais plus flexible que Hive SQL. Idéal pour les transformations complexes multi-étapes difficiles à exprimer en SQL.

```pig
-- Charger des données
raw_logs = LOAD '/data/logs/' USING PigStorage(',')
           AS (timestamp:chararray, user_id:int, action:chararray, duration:float);

-- Filtrer
valid_logs = FILTER raw_logs BY duration > 0 AND user_id IS NOT NULL;

-- Grouper et agréger
grouped = GROUP valid_logs BY user_id;
user_stats = FOREACH grouped GENERATE
    group AS user_id,
    COUNT(valid_logs) AS total_actions,
    AVG(valid_logs.duration) AS avg_duration,
    MAX(valid_logs.duration) AS max_duration;

-- Trier et limiter
top_users = ORDER user_stats BY total_actions DESC;
top_100 = LIMIT top_users 100;

-- Stocker
STORE top_100 INTO '/data/output/top_users/' USING PigStorage('\t');
```

**Grunt Shell** : REPL interactif pour tester des scripts Pig ligne par ligne.

| Outil | Quand utiliser |
|-------|---------------|
| **Hive** | Requêtes SQL, analytics, rapports, équipes SQL |
| **Pig** | Transformations complexes, ETL multi-étapes, logique non-SQL |
| **Spark** | Les deux + ML + streaming + performance |

---

## 7. HBase — NoSQL Column-Family sur HDFS

### Modèle de données HBase

```
Table: user_activity
├── Row Key: "user_001|2024-01-15"   ← Clé principale (bytewise sorted)
│   ├── Column Family: "info"
│   │   ├── info:name    = "Alice"
│   │   └── info:email   = "alice@example.com"
│   └── Column Family: "events"
│       ├── events:login  = "2024-01-15T08:00:00"
│       └── events:logout = "2024-01-15T18:00:00"
└── Row Key: "user_002|2024-01-15"
    └── ...
```

### Row Key Design — Critique pour les performances

```
❌ Mauvais : row key séquentiel → hot spot (tout écrit sur le même Region Server)
user_00001, user_00002, user_00003...

✅ Bon : row key avec salt ou reverse timestamp pour distribuer
# Salt : hash(user_id) % 10 préfixé
3|user_00001, 7|user_00002, 1|user_00003

# Reverse timestamp pour les données time-series (dernière donnée en premier)
user_001|(Long.MAX_VALUE - timestamp)
```

### LSM Trees et Compaction

HBase utilise les **Log-Structured Merge trees** :
1. Les écritures vont dans **MemStore** (RAM) + **WAL** (Write-Ahead Log sur HDFS pour durabilité)
2. Quand MemStore est plein → **flush** vers un HFile sur HDFS
3. Accumulation de HFiles → **compaction** (Minor = fusionner quelques HFiles, Major = tout fusionner + supprimer les données expirées)

```java
// Java API HBase
Configuration conf = HBaseConfiguration.create();
Connection connection = ConnectionFactory.createConnection(conf);
Table table = connection.getTable(TableName.valueOf("user_activity"));

// Put (écriture)
Put put = new Put(Bytes.toBytes("user_001|2024-01-15"));
put.addColumn(
    Bytes.toBytes("info"),
    Bytes.toBytes("name"),
    Bytes.toBytes("Alice")
);
table.put(put);

// Get (lecture d'une ligne)
Get get = new Get(Bytes.toBytes("user_001|2024-01-15"));
Result result = table.get(get);
byte[] name = result.getValue(Bytes.toBytes("info"), Bytes.toBytes("name"));

// Scan (lecture de plusieurs lignes)
Scan scan = new Scan();
scan.setStartRow(Bytes.toBytes("user_001"));
scan.setStopRow(Bytes.toBytes("user_002"));
ResultScanner scanner = table.getScanner(scan);
for (Result r : scanner) { /* traiter */ }
```

> [!warning] Scans vs Gets
> Un **Get** est O(1) par row key. Un **Scan** avec filtre sur une colonne fait un full scan de la plage — préférer un row key bien conçu qui encode les critères de filtre pour limiter la plage scannée.

---

## 8. Écosystème Hadoop complet

### Sqoop — Import/Export RDBMS ↔ HDFS

```bash
# Import depuis MySQL vers HDFS
sqoop import \
    --connect jdbc:mysql://db-server/mydb \
    --username admin --password secret \
    --table customers \
    --target-dir /data/customers/ \
    --split-by customer_id \           # Colonne pour paralléliser
    --num-mappers 4 \                  # 4 mappers en parallèle
    --fields-terminated-by ',' \
    --as-parquetfile                   # Format Parquet directement

# Export depuis HDFS vers MySQL
sqoop export \
    --connect jdbc:mysql://db-server/mydb \
    --username admin --password secret \
    --table results \
    --export-dir /data/output/results/
```

### Flume — Ingestion de logs

```properties
# Configuration agent Flume pour logs Apache → HDFS
agent.sources = apache_log
agent.channels = memory_channel
agent.sinks = hdfs_sink

agent.sources.apache_log.type = exec
agent.sources.apache_log.command = tail -F /var/log/apache2/access.log

agent.channels.memory_channel.type = memory
agent.channels.memory_channel.capacity = 10000

agent.sinks.hdfs_sink.type = hdfs
agent.sinks.hdfs_sink.hdfs.path = /data/logs/%Y/%m/%d/
agent.sinks.hdfs_sink.hdfs.rollInterval = 3600
agent.sinks.hdfs_sink.hdfs.fileType = DataStream
```

### Oozie — Orchestration de workflows

```xml
<!-- Workflow Oozie : Sqoop import → MapReduce → notification email -->
<workflow-app name="daily-etl" xmlns="uri:oozie:workflow:0.5">
    <start to="sqoop-import"/>

    <action name="sqoop-import">
        <sqoop xmlns="uri:oozie:sqoop-action:0.2">
            <command>import --connect ... --table orders ...</command>
        </sqoop>
        <ok to="mapreduce-transform"/>
        <error to="fail"/>
    </action>

    <action name="mapreduce-transform">
        <map-reduce>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property><name>mapred.mapper.class</name><value>com.example.Mapper</value></property>
            </configuration>
        </map-reduce>
        <ok to="end"/>
        <error to="fail"/>
    </action>

    <kill name="fail"><message>ETL failed: ${wf:errorMessage(wf:lastErrorNode())}</message></kill>
    <end name="end"/>
</workflow-app>
```

### Zookeeper — Coordination distribuée

ZooKeeper est le service de coordination pour les clusters distribués. Utilisé par HBase, Kafka, HDFS HA.

**Fonctionnalités** :
- **Leader election** : qui est le NameNode actif ?
- **Configuration distribuée** : partage de config entre nœuds
- **Service discovery** : quels services sont disponibles ?
- **Distributed locks** : synchronisation entre processus

**Znodes** : arbre de données hiérarchique (comme un filesystem). Les clients peuvent créer des znodes éphémères (disparaissent à la déconnexion) pour le lock/election.

### Ambari — Cluster Management

Interface web pour :
- Installer et configurer les services Hadoop
- Monitoring (métriques CPU, RAM, HDFS usage)
- Alertes automatiques
- Gestion des configurations centralisée

---

## 9. Cloud Hadoop — Services managés

### Comparatif des offres cloud

| Service | Provider | Particularité |
|---------|---------|--------------|
| **AWS EMR** | Amazon | Intégration native S3, auto-scaling, spot instances, Livy REST API |
| **Google Dataproc** | GCP | Démarrage en 90 secondes, intégration GCS et BigQuery, Jupyter Hub |
| **Azure HDInsight** | Microsoft | Intégration Azure Data Lake Storage Gen2, Active Directory |

### AWS EMR — Exemple de cluster

```bash
# Créer un cluster EMR avec Spark + Hive
aws emr create-cluster \
    --name "Analytics Cluster" \
    --release-label emr-6.10.0 \
    --applications Name=Spark Name=Hive Name=Hadoop \
    --instance-type m5.xlarge \
    --instance-count 5 \
    --ec2-attributes KeyName=my-key-pair \
    --use-default-roles \
    --auto-terminate    # Cluster s'arrête quand les steps sont terminés

# Soumettre un job Spark
aws emr add-steps \
    --cluster-id j-XXXXX \
    --steps Type=Spark,Name="Word Count",\
            ActionOnFailure=CONTINUE,\
            Args=[--class,WordCount,s3://my-bucket/wordcount.jar,\
                  s3://my-bucket/input/,s3://my-bucket/output/]
```

### Self-hosted vs Managed

| Critère | Self-hosted | Cloud Managed |
|---------|------------|--------------|
| Contrôle | Total | Limité |
| Maintenance | Équipe ops dédiée | Gérée par le provider |
| Coût (long terme) | Moins cher | Pay-as-you-go (peut coûter cher) |
| Démarrage | Semaines | Minutes |
| Sécurité | Maîtrisée | Partagée avec le provider |

> [!tip] Tendance actuelle
> La tendance est au **découplage stockage/calcul** : stocker les données sur S3/GCS (pas dans HDFS) et créer des clusters temporaires pour le calcul (EMR, Dataproc). Le cluster ne tourne que pendant le job → économies importantes.

---

## 10. Wikilinks et ressources

- [[02 - Apache Spark et PySpark]] — Le successeur de MapReduce, 100x plus rapide
- [[01 - Introduction au NoSQL]] — Contexte NoSQL dont HBase
- [[02 - Services Cloud Essentiels]] — AWS S3, EMR, et les services managés

---

## Exercices Pratiques

### Exercice 1 — Manipulation HDFS
```bash
# 1. Créer la structure de dossiers suivante dans HDFS :
#    /tp/input/
#    /tp/output/
# 2. Uploader 3 fichiers texte locaux dans /tp/input/
# 3. Vérifier la réplication (hdfs fsck /tp/input/ -files -blocks -locations)
# 4. Calculer l'espace utilisé
```

### Exercice 2 — MapReduce Python
Écrire un job Hadoop Streaming qui :
1. Lit des fichiers de logs Apache (format Combined Log)
2. Compte le nombre de requêtes par code de statut HTTP (200, 404, 500...)
3. Trie les résultats par nombre de requêtes décroissant

```python
# Template mapper.py — à compléter
import sys
import re

LOG_PATTERN = r'(\d+\.\d+\.\d+\.\d+).*".*" (\d+) (\d+)'

for line in sys.stdin:
    match = re.search(LOG_PATTERN, line)
    if match:
        status_code = match.group(2)
        # Émettre la clé et la valeur appropriées
        print(f"???")
```

### Exercice 3 — HiveQL
```sql
-- Créer une table partitionnée par mois pour des transactions e-commerce
-- Chaque ligne : transaction_id, customer_id, product_id, amount, timestamp
-- 1. Créer la table externe pointant vers /data/ecommerce/
-- 2. Calculer le CA mensuel par client
-- 3. Identifier les top 10 clients sur l'année
-- 4. Calculer le panier moyen par mois
```

### Exercice 4 — Architecture
Concevoir une architecture Hadoop pour ce besoin :
> Une e-commerce génère 50 GB de logs/jour (clics, pages vues, achats). L'équipe Data veut : (1) un rapport quotidien du top 100 produits, (2) une détection de fraude en quasi temps réel, (3) un modèle ML pour les recommandations.

Identifier quels composants de l'écosystème Hadoop utiliser pour chaque besoin et justifier.

> [!tip] Corrigé Exercice 4 (partiel)
> - Rapport quotidien → Flume (ingestion) + HDFS + Hive (SQL) ou Spark (batch)
> - Fraude quasi temps réel → Kafka + Spark Structured Streaming (micro-batch)
> - Recommandations ML → Spark MLlib + données stockées en Parquet/ORC
