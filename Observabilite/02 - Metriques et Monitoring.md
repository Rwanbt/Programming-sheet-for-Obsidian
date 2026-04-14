# Metriques et Monitoring

## Qu'est-ce qu'une Metrique ?

Une **metrique** est une **mesure numerique** capturee au cours du temps. Contrairement aux logs qui enregistrent des evenements textuels, les metriques sont des **donnees structurees** sous forme de chiffres : un nombre de requetes, un pourcentage d'utilisation CPU, un temps de reponse en millisecondes.

```
  Valeur
    ^
 95 |                              *
 90 |                         *         *
 85 |                    *                    *
 80 |               *                              *
 75 |          *
 70 |     *
 65 | *
    +----+----+----+----+----+----+----+----+----+----> Temps
    10h  10h  10h  10h  10h  10h  10h  10h  10h  10h
    00   05   10   15   20   25   30   35   40   45

    Exemple : utilisation CPU (%) sur 45 minutes
    Chaque point = une mesure a un instant precis (un "sample")
```

> [!tip] Analogie
> Imaginez le **tableau de bord d'une voiture** :
> - Le **compteur de vitesse** = une metrique qui change en temps reel (gauge)
> - Le **compteur kilometrique** = une metrique qui ne fait qu'augmenter (counter)
> - L'**indicateur de temperature moteur** = une metrique avec des seuils d'alerte
>
> Vous ne lisez pas un roman pour savoir si votre voiture va bien. Vous regardez des **chiffres** sur un tableau de bord. Les metriques, c'est exactement ca pour vos applications.

---

## Metriques vs Logs

Les metriques et les logs sont complementaires mais fondamentalement differents :

| Aspect | Metriques | Logs |
|---|---|---|
| **Format** | Nombres structures | Texte (structure ou non) |
| **Volume** | Faible (echantillons) | Eleve (chaque evenement) |
| **Cout de stockage** | Faible | Eleve |
| **Requetage** | Tres rapide (series temporelles) | Plus lent (recherche texte) |
| **Aggregation** | Naturelle (sum, avg, percentile) | Complexe |
| **Cardinalite** | Attention a la cardinalite | Pas de limite |
| **Usage principal** | Alerting, dashboards, tendances | Investigation, debugging |
| **Duree de retention** | Mois/annees (compression) | Jours/semaines (volume) |
| **Question typique** | "Combien de req/s en ce moment ?" | "Pourquoi cette requete a echoue ?" |

> [!info] Quand utiliser quoi ?
> - **Metriques** : pour savoir **si** quelque chose ne va pas (detection)
> - **Logs** : pour comprendre **pourquoi** ca ne va pas (investigation)
> - **Traces** : pour comprendre **ou** ca ne va pas dans un systeme distribue (localisation)

---

## Les 4 Golden Signals (Google SRE)

Le livre **Site Reliability Engineering** de Google definit les **4 signaux essentiels** que tout systeme doit surveiller. Si vous ne pouvez mesurer que 4 choses, mesurez celles-ci :

```
                    LES 4 GOLDEN SIGNALS
    ┌──────────────────────────────────────────────┐
    │                                              │
    │   ┌──────────┐          ┌──────────┐         │
    │   │ LATENCE  │          │  TRAFIC  │         │
    │   │          │          │          │         │
    │   │ Combien  │          │ Combien  │         │
    │   │ de temps │          │ de       │         │
    │   │ ca prend │          │ demandes │         │
    │   └──────────┘          └──────────┘         │
    │                                              │
    │   ┌──────────┐          ┌──────────────┐     │
    │   │ ERREURS  │          │  SATURATION  │     │
    │   │          │          │              │     │
    │   │ Combien  │          │ A quel point │     │
    │   │ echouent │          │ c'est plein  │     │
    │   └──────────┘          └──────────────┘     │
    │                                              │
    └──────────────────────────────────────────────┘
```

### 1. Latence (Latency)

Le **temps** qu'il faut pour traiter une requete. Il faut distinguer la latence des requetes reussies et des requetes echouees.

```python
# Exemples de metriques de latence
http_request_duration_seconds{status="200"}  # Latence des succes
http_request_duration_seconds{status="500"}  # Latence des erreurs

# Indicateurs cles :
# - Mediane (P50) : 50% des requetes sont plus rapides
# - P95 : 95% des requetes sont plus rapides
# - P99 : 99% des requetes sont plus rapides
```

> [!warning] Ne regardez pas que la moyenne !
> La moyenne masque les extremes. Si 99% des requetes prennent 10ms et 1% prennent 10 secondes, la moyenne dit "110ms" et vous pensez que tout va bien. Le **P99** revele les vrais problemes.

### 2. Trafic (Traffic)

Le **volume de demandes** sur votre systeme. Selon le type de service, ca peut etre :

- Requetes HTTP par seconde
- Transactions par seconde
- Messages dans une queue
- Sessions actives

### 3. Erreurs (Errors)

Le **taux de requetes qui echouent**, explicitement (HTTP 500) ou implicitement (HTTP 200 avec contenu incorrect, ou reponse trop lente).

### 4. Saturation (Saturation)

A quel point votre systeme est **"plein"**. Ca mesure les ressources les plus contraintes :
- CPU utilise a 90%
- Memoire disponible a 5%
- Disque plein a 95%
- File d'attente avec 10 000 messages en attente

> [!example] Les 4 Golden Signals pour une API web
> | Signal | Metrique | Seuil d'alerte |
> |---|---|---|
> | **Latence** | `http_request_duration_seconds` P99 | > 500ms |
> | **Trafic** | `http_requests_total` rate/s | < 10 req/s (trop bas = probleme) |
> | **Erreurs** | `http_requests_total{status=~"5.."}` / total | > 1% |
> | **Saturation** | `container_memory_usage_bytes` / limit | > 85% |

---

## Types de Metriques

Toutes les metriques ne se comportent pas de la meme facon. Il existe 4 types fondamentaux :

### Counter (Compteur)

Un **counter** est une valeur qui ne fait que **monter** (ou se reinitialiser a zero au redemarrage). On ne s'interesse pas a sa valeur absolue, mais a sa **vitesse de changement** (rate).

```
  Valeur
    ^
 500|                                          *
 400|                                    *
 300|                              *
 200|                       *
 100|              *
  50|        *
  10|  *
    +----+----+----+----+----+----+----+-------> Temps
    
    Exemple : nombre total de requetes HTTP recues
    La valeur ne descend JAMAIS (sauf redemarrage)
    
    Ce qui nous interesse : rate() = combien par seconde
```

**Exemples** : nombre total de requetes, nombre total d'erreurs, octets envoyes, taches completees.

### Gauge (Jauge)

Un **gauge** est une valeur qui peut **monter et descendre**. C'est un instantane de l'etat actuel.

```
  Valeur
    ^
 85 |        *              *
 75 |   *         *              *
 60 |                  *              *
 45 |                                      *
 30 | *                                         *
    +----+----+----+----+----+----+----+----+----> Temps
    
    Exemple : utilisation CPU en %
    La valeur monte et descend en temps reel
```

**Exemples** : temperature, utilisation memoire, nombre de goroutines actives, taille d'une file d'attente.

### Histogram (Histogramme)

Un **histogram** mesure la **distribution** des valeurs en les repartissant dans des "buckets" (intervalles). Essentiel pour les latences.

```
  Nombre de
  requetes
    ^
 250|  ████
 200|  ████  ████
 150|  ████  ████
 100|  ████  ████  ████
  50|  ████  ████  ████  ████
  20|  ████  ████  ████  ████  ████
    +------+------+------+------+------+
    0-50ms 50-100 100-200 200-500 500ms+
    
    Bucket (intervalle de latence)
    
    On voit que la majorite des requetes sont < 100ms
    mais quelques-unes depassent 500ms
```

Un histogram genere automatiquement **3 series** :
- `_bucket` : nombre d'observations dans chaque intervalle
- `_sum` : somme totale des observations
- `_count` : nombre total d'observations

### Summary (Resume)

Similaire a l'histogramme mais calcule les **quantiles** (percentiles) cote client. Moins flexible car les quantiles ne peuvent pas etre re-agreges entre instances.

| Type | Monte/Descend | Usage principal | Fonction PromQL |
|---|---|---|---|
| **Counter** | Monte seulement | Nombre d'evenements | `rate()`, `increase()` |
| **Gauge** | Monte et descend | Etat actuel | `avg()`, `min()`, `max()` |
| **Histogram** | Monte seulement (buckets) | Distribution | `histogram_quantile()` |
| **Summary** | Monte seulement (quantiles) | Distribution (pre-calculee) | Directement lisible |

---

## Prometheus : Architecture

**Prometheus** est le systeme de monitoring open-source de reference. Cree chez SoundCloud, donne a la CNCF (Cloud Native Computing Foundation), c'est le standard de facto pour le monitoring des applications cloud-native.

```
                         ARCHITECTURE PROMETHEUS
                         
  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
  │ Application │    │ Application │    │   Node      │
  │  Python     │    │  Go         │    │  Exporter   │
  │  /metrics   │    │  /metrics   │    │  /metrics   │
  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘
         │                  │                  │
         │    SCRAPE (pull) │                  │
         │    toutes les N  │                  │
         │    secondes      │                  │
         ▼                  ▼                  ▼
  ┌────────────────────────────────────────────────────┐
  │                   PROMETHEUS                        │
  │                                                    │
  │  ┌──────────┐  ┌──────────────┐  ┌──────────────┐ │
  │  │ Scraper  │  │    TSDB      │  │   PromQL     │ │
  │  │ (collecte│─>│  (stockage   │<─│  (langage    │ │
  │  │  HTTP)   │  │  series      │  │  de requete) │ │
  │  └──────────┘  │  temporelles)│  └──────┬───────┘ │
  │                └──────────────┘         │         │
  │  ┌──────────────┐                      │         │
  │  │ Alert Rules  │──────────────────────┘         │
  │  └──────┬───────┘                                │
  └─────────┼────────────────────────────────────────┘
            │                          │
            ▼                          ▼
  ┌──────────────────┐      ┌──────────────────┐
  │  ALERTMANAGER    │      │    GRAFANA        │
  │                  │      │                  │
  │  Routing         │      │  Dashboards      │
  │  Deduplication   │      │  Visualisation   │
  │  Silencing       │      │                  │
  └────────┬─────────┘      └──────────────────┘
           │
     ┌─────┼──────┐
     ▼     ▼      ▼
  Email  Slack  PagerDuty
```

> [!info] Modele Pull vs Push
> Prometheus utilise un modele **pull** : c'est Prometheus qui va **chercher** les metriques sur chaque cible (scrape). C'est l'inverse de beaucoup de systemes ou les applications **envoient** (push) leurs metriques. Le pull simplifie la configuration : les applications n'ont pas besoin de connaitre l'adresse de Prometheus.

### Concepts cles

- **Target** : une application qui expose des metriques via un endpoint HTTP (generalement `/metrics`)
- **Scrape** : l'action de Prometheus qui va lire les metriques d'une target
- **TSDB** : Time Series Database, le stockage optimise pour les series temporelles
- **PromQL** : le langage de requete puissant de Prometheus
- **Exporter** : un composant qui expose les metriques d'un systeme tiers (Node Exporter pour Linux, MySQL Exporter, etc.)

---

## Instrumenter une Application Python

### Installation

```bash
pip install prometheus-client flask
# ou avec FastAPI :
pip install prometheus-client fastapi uvicorn
```

### Exemple complet avec Flask

```python
from flask import Flask, request
from prometheus_client import (
    Counter, Gauge, Histogram, Summary,
    generate_latest, CONTENT_TYPE_LATEST
)
import time
import random

app = Flask(__name__)

# -------------------------------------------------------
# DEFINIR LES METRIQUES
# -------------------------------------------------------

# Counter : nombre total de requetes
http_requests_total = Counter(
    'http_requests_total',                    # Nom de la metrique
    'Nombre total de requetes HTTP recues',   # Description
    ['method', 'endpoint', 'status']          # Labels (dimensions)
)

# Gauge : nombre de requetes en cours de traitement
requests_in_progress = Gauge(
    'http_requests_in_progress',
    'Nombre de requetes HTTP en cours de traitement'
)

# Histogram : distribution des temps de reponse
request_duration_seconds = Histogram(
    'http_request_duration_seconds',
    'Duree des requetes HTTP en secondes',
    ['method', 'endpoint'],
    buckets=[0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0]
)

# Gauge : donnee metier
items_in_cart = Gauge(
    'items_in_cart_total',
    'Nombre total d\'articles dans tous les paniers'
)


# -------------------------------------------------------
# MIDDLEWARE : mesurer chaque requete automatiquement
# -------------------------------------------------------

@app.before_request
def before_request():
    request.start_time = time.time()
    requests_in_progress.inc()  # +1 requete en cours

@app.after_request
def after_request(response):
    # Mesurer la duree
    duration = time.time() - request.start_time
    request_duration_seconds.labels(
        method=request.method,
        endpoint=request.path
    ).observe(duration)

    # Compter la requete
    http_requests_total.labels(
        method=request.method,
        endpoint=request.path,
        status=response.status_code
    ).inc()

    requests_in_progress.dec()  # -1 requete en cours
    return response


# -------------------------------------------------------
# ENDPOINTS DE L'APPLICATION
# -------------------------------------------------------

@app.route('/api/users')
def get_users():
    time.sleep(random.uniform(0.01, 0.1))  # Simule un traitement
    return {"users": ["alice", "bob", "charlie"]}

@app.route('/api/orders', methods=['POST'])
def create_order():
    time.sleep(random.uniform(0.05, 0.3))
    items_in_cart.dec(3)  # 3 articles retires du panier
    return {"order_id": "ORD-12345"}, 201


# -------------------------------------------------------
# ENDPOINT /metrics POUR PROMETHEUS
# -------------------------------------------------------

@app.route('/metrics')
def metrics():
    return generate_latest(), 200, {'Content-Type': CONTENT_TYPE_LATEST}


if __name__ == '__main__':
    items_in_cart.set(42)  # Valeur initiale
    app.run(host='0.0.0.0', port=8000)
```

### Exemple avec FastAPI

```python
from fastapi import FastAPI, Request
from prometheus_client import Counter, Histogram, generate_latest
from starlette.responses import Response
import time

app = FastAPI()

REQUEST_COUNT = Counter(
    'http_requests_total', 'Total HTTP requests',
    ['method', 'endpoint', 'status']
)
REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds', 'Request latency',
    ['method', 'endpoint']
)

@app.middleware("http")
async def prometheus_middleware(request: Request, call_next):
    start = time.time()
    response = await call_next(request)
    duration = time.time() - start

    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.url.path,
        status=response.status_code
    ).inc()
    REQUEST_LATENCY.labels(
        method=request.method,
        endpoint=request.url.path
    ).observe(duration)

    return response

@app.get("/api/health")
async def health():
    return {"status": "ok"}

@app.get("/metrics")
async def metrics():
    return Response(
        content=generate_latest(),
        media_type="text/plain"
    )
```

> [!warning] Attention a la cardinalite des labels !
> Chaque combinaison unique de labels cree une nouvelle serie temporelle. Si vous mettez `user_id` comme label et que vous avez 1 million d'utilisateurs, vous creez 1 million de series. Prometheus ne gere pas bien la haute cardinalite. Utilisez des labels a **faible cardinalite** : method, status, endpoint, region, env.

---

## PromQL : Langage de Requete

**PromQL** (Prometheus Query Language) est le langage pour interroger les donnees de Prometheus. C'est un langage puissant et expressif.

### Vecteurs instantanes et de plage

```promql
# Vecteur instantane : la derniere valeur de chaque serie
http_requests_total
# Retourne : http_requests_total{method="GET", endpoint="/api/users", status="200"} 1523

# Vecteur de plage : les valeurs sur une periode
http_requests_total[5m]
# Retourne les echantillons des 5 dernieres minutes pour chaque serie

# Filtrage par labels
http_requests_total{method="GET"}
http_requests_total{status=~"5.."}          # regex : tout status 5xx
http_requests_total{endpoint!="/healthz"}   # exclusion
```

### Fonctions essentielles

```promql
# -------------------------------------------------------
# rate() : vitesse de changement par seconde d'un counter
# C'EST LA FONCTION LA PLUS IMPORTANTE
# -------------------------------------------------------
rate(http_requests_total[5m])
# = nombre de requetes par seconde (moyenne sur 5 min)

# -------------------------------------------------------
# sum() : agregation
# -------------------------------------------------------
sum(rate(http_requests_total[5m]))
# = total des requetes/s tous labels confondus

# -------------------------------------------------------
# sum() + by() : agreger en gardant certaines dimensions
# -------------------------------------------------------
sum(rate(http_requests_total[5m])) by (status)
# = requetes/s regroupees par code status

sum(rate(http_requests_total[5m])) by (method, endpoint)
# = requetes/s par methode et endpoint

# -------------------------------------------------------
# avg(), min(), max() : aggregations classiques
# -------------------------------------------------------
avg(rate(http_request_duration_seconds_sum[5m]) 
    / rate(http_request_duration_seconds_count[5m]))
# = duree moyenne des requetes

# -------------------------------------------------------
# histogram_quantile() : percentiles depuis un histogram
# -------------------------------------------------------
histogram_quantile(0.99, 
    rate(http_request_duration_seconds_bucket[5m]))
# = P99 de la duree des requetes (99% sont plus rapides)

histogram_quantile(0.50, 
    rate(http_request_duration_seconds_bucket[5m]))
# = Mediane (P50)

# -------------------------------------------------------
# increase() : augmentation totale sur une periode
# -------------------------------------------------------
increase(http_requests_total[1h])
# = nombre de requetes dans la derniere heure

# -------------------------------------------------------
# Exemples avances
# -------------------------------------------------------
# Taux d'erreur en pourcentage
sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ sum(rate(http_requests_total[5m])) * 100

# Requetes/s par service
sum(rate(http_requests_total[5m])) by (job)
```

> [!tip] Analogie
> PromQL est comme **Excel pour les series temporelles** :
> - `rate()` = la derivee (vitesse de changement)
> - `sum() by (label)` = un tableau croise dynamique (GROUP BY en SQL)
> - `histogram_quantile()` = PERCENTILE.INC() d'Excel
> - Les vecteurs = les colonnes de votre feuille de calcul

---

## Grafana : Visualisation

**Grafana** est la plateforme de visualisation open-source de reference. Elle se connecte a Prometheus (et dizaines d'autres sources) pour creer des dashboards interactifs.

### Connexion a Prometheus

1. Dans Grafana : **Configuration > Data Sources > Add data source**
2. Choisir **Prometheus**
3. URL : `http://prometheus:9090` (ou l'adresse de votre Prometheus)
4. Cliquer **Save & Test**

### Types de panneaux (Panels)

| Type | Usage | Exemple |
|---|---|---|
| **Time Series** | Evolution dans le temps | Requetes/s, latence, CPU |
| **Stat** | Valeur unique mise en avant | Uptime : 99.97% |
| **Gauge** | Valeur avec seuils visuels | CPU : 72% (jaune > 70%, rouge > 90%) |
| **Table** | Donnees tabulaires | Top 10 endpoints les plus lents |
| **Bar Chart** | Comparaisons | Erreurs par service |
| **Heatmap** | Distribution dans le temps | Distribution des latences |
| **Logs** | Affichage de logs (Loki) | Logs filtres par trace_id |

### Variables et templates

Les **variables** rendent les dashboards dynamiques et reutilisables :

```
# Variable "environment" : dropdown pour choisir l'environnement
Type : Query
Query : label_values(http_requests_total, env)
# Resultat : dropdown avec [production, staging, development]

# Variable "service" : dropdown pour choisir le service
Type : Query  
Query : label_values(http_requests_total{env="$environment"}, job)
# Resultat : dropdown avec [api-gateway, user-service, order-service]

# Utilisation dans les requetes du dashboard :
rate(http_requests_total{env="$environment", job="$service"}[5m])
```

### Dashboard as Code (JSON)

Les dashboards Grafana peuvent etre exportes et geres en JSON, ce qui permet le versioning avec Git :

```json
{
  "dashboard": {
    "title": "API Overview",
    "tags": ["api", "production"],
    "panels": [
      {
        "title": "Requetes par seconde",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{env=\"production\"}[5m])) by (endpoint)",
            "legendFormat": "{{endpoint}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "reqps"
          }
        }
      },
      {
        "title": "Taux d'erreur",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "steps": [
                {"value": 0, "color": "green"},
                {"value": 1, "color": "yellow"},
                {"value": 5, "color": "red"}
              ]
            }
          }
        }
      }
    ]
  }
}
```

> [!info] Grafana as Code
> Utilisez **Grafonnet** (librairie Jsonnet) ou **Terraform provider Grafana** pour generer vos dashboards de maniere programmatique. Cela permet de versionner, reviser et deployer vos dashboards comme du code.

---

## Alerting

### Alertmanager

L'**Alertmanager** gere les alertes generees par Prometheus. Il s'occupe de :

```
                    FLUX D'ALERTING
                    
  Prometheus                Alertmanager              Destinataires
  ┌──────────┐             ┌──────────────┐          ┌──────────┐
  │ Regles   │  ──fire──>  │ Regroupement │ ──────>  │  Email   │
  │ d'alerte │             │ Deduplication │          │  Slack   │
  │          │             │ Silencing     │          │ PagerDuty│
  │ Evalue   │             │ Inhibition    │          │ Webhook  │
  │ toutes   │             │ Routing       │          └──────────┘
  │ les N sec│             └──────────────┘
  └──────────┘
```

### Regles d'alerte dans Prometheus

```yaml
# prometheus/alert_rules.yml
groups:
  - name: api_alerts
    rules:
      # Alerte : taux d'erreur trop eleve
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 0.01
        for: 5m    # L'alerte doit etre vraie pendant 5 min
        labels:
          severity: critical
        annotations:
          summary: "Taux d'erreur superieur a 1%"
          description: "Le taux d'erreur 5xx est de {{ $value | humanizePercentage }} depuis 5 minutes"

      # Alerte : latence P99 trop elevee
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99,
            rate(http_request_duration_seconds_bucket[5m])) > 0.5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Latence P99 superieure a 500ms"
          description: "P99 = {{ $value }}s"

      # Alerte : instance down
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
```

### Configuration de l'Alertmanager

```yaml
# alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m

route:
  receiver: 'default-slack'
  group_by: ['alertname', 'severity']
  group_wait: 30s       # Attendre 30s avant d'envoyer (grouper)
  group_interval: 5m    # Intervalle entre groupes
  repeat_interval: 4h   # Re-notifier toutes les 4h si non resolu
  
  routes:
    # Les alertes critiques vont a PagerDuty
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
      continue: true    # Continuer a evaluer les routes suivantes
    
    # Les alertes critiques vont aussi a Slack
    - match:
        severity: critical
      receiver: 'slack-critical'
    
    # Les warnings vont a Slack seulement
    - match:
        severity: warning
      receiver: 'slack-warnings'

receivers:
  - name: 'default-slack'
    slack_configs:
      - channel: '#monitoring'
        api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'

  - name: 'slack-critical'
    slack_configs:
      - channel: '#incidents'
        api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
        title: '🚨 CRITICAL: {{ .GroupLabels.alertname }}'
        text: '{{ .CommonAnnotations.description }}'

  - name: 'slack-warnings'
    slack_configs:
      - channel: '#monitoring'
        api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: 'votre-cle-pagerduty'

# Silencing : desactiver temporairement des alertes
# (se fait via l'interface web de l'Alertmanager)

# Inhibition : supprimer certaines alertes quand d'autres sont actives
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
    # Si une alerte critical existe, supprimer les warnings
    # pour le meme alertname et la meme instance
```

> [!warning] Alert Fatigue
> Trop d'alertes = personne ne les regarde. Regles d'or :
> - Chaque alerte doit necessiter une **action humaine**
> - Si une alerte ne reveille pas quelqu'un a 3h du matin, ce n'est pas un `critical`
> - Utilisez des seuils realistes et le parametre `for` pour eviter les faux positifs
> - Revoyez regulierement vos alertes : supprimez celles qui ne servent pas

---

## SLI, SLO et SLA

Ces trois concepts forment la base de la fiabilite orientee service :

```
  ┌────────────────────────────────────────────────────┐
  │                                                    │
  │   SLI (Service Level Indicator)                    │
  │   = La METRIQUE que vous mesurez                   │
  │   "Quel pourcentage de requetes repondent < 200ms" │
  │                                                    │
  │   SLO (Service Level Objective)                    │
  │   = L'OBJECTIF que vous visez                      │
  │   "99.9% des requetes doivent repondre < 200ms"    │
  │                                                    │
  │   SLA (Service Level Agreement)                    │
  │   = Le CONTRAT avec vos clients                    │
  │   "Si < 99.5%, on vous rembourse 10%"              │
  │                                                    │
  │   Relation :                                       │
  │   SLI ──mesure──> SLO ──engagement──> SLA          │
  │   (chiffre)       (objectif interne)  (contrat)    │
  │                                                    │
  └────────────────────────────────────────────────────┘
```

### Exemples concrets

| Service | SLI | SLO | SLA |
|---|---|---|---|
| API Web | % de requetes avec latence < 200ms | 99.9% | 99.5% (remboursement si violation) |
| Base de donnees | % de temps disponible | 99.99% (52 min downtime/an) | 99.95% |
| Pipeline data | % de jobs completes avec succes | 99% | 95% |

### Error Budget (Budget d'erreur)

L'**error budget** est la quantite d'indisponibilite **autorisee** par votre SLO :

```
SLO = 99.9% de disponibilite

Error Budget = 100% - 99.9% = 0.1%
Sur 30 jours  = 30 * 24 * 60 * 0.001 = 43.2 minutes

Vous avez droit a 43.2 minutes d'indisponibilite par mois.

Si vous avez consomme 30 minutes le 15 du mois :
  Budget restant = 43.2 - 30 = 13.2 minutes
  => Ralentir les deployments, reduire les risques
  
Si le budget est epuise :
  => STOP aux deployments de features
  => Focus uniquement sur la fiabilite
```

> [!tip] Analogie
> L'error budget, c'est comme un **forfait de donnees mobiles** :
> - Vous avez un quota mensuel (43 minutes d'indisponibilite)
> - Chaque incident consomme du quota
> - Si vous avez du budget restant, vous pouvez prendre des risques (deployer des features)
> - Si le budget est epuise, vous devez etre conservateur
>
> C'est un outil puissant pour **negocier** entre l'equipe produit ("deployons vite !") et l'equipe SRE ("soyons prudents !").

---

## RED Method (pour les services)

La methode **RED** est un framework de monitoring specifique aux **services** (microservices, APIs) :

```
  R - Rate      = Nombre de requetes par seconde
  E - Errors    = Nombre de requetes echouees par seconde
  D - Duration  = Distribution du temps de reponse (histogramme)
```

```promql
# Rate : requetes par seconde
sum(rate(http_requests_total[5m])) by (service)

# Errors : taux d'erreur
sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
/ sum(rate(http_requests_total[5m])) by (service)

# Duration : P50, P90, P99
histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))
histogram_quantile(0.90, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))
```

## USE Method (pour les ressources)

La methode **USE** est un framework de monitoring specifique aux **ressources** (CPU, memoire, disque, reseau) :

```
  U - Utilization  = Pourcentage de temps ou la ressource est occupee
  S - Saturation   = Degre de travail en file d'attente
  E - Errors       = Nombre d'erreurs de la ressource
```

| Ressource | Utilization | Saturation | Errors |
|---|---|---|---|
| **CPU** | % de temps non-idle | Longueur de la run queue | Erreurs machine check |
| **Memoire** | % utilisee | Pages de swap | OOM kills |
| **Disque** | % I/O time | Longueur I/O queue | Erreurs lecture/ecriture |
| **Reseau** | % de bande passante | Paquets en attente (drops) | Erreurs de transmission |

> [!info] RED vs USE
> - **RED** = pour ce que vos **utilisateurs** voient (les services)
> - **USE** = pour ce que votre **infrastructure** subit (les ressources)
> - Utilisez les deux ensemble pour une vision complete

---

## Pratique : Prometheus + Grafana avec Docker Compose

Voici un setup complet et fonctionnel :

### Structure du projet

```
monitoring/
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml
│   └── alert_rules.yml
├── alertmanager/
│   └── alertmanager.yml
└── grafana/
    └── provisioning/
        └── datasources/
            └── prometheus.yml
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  # -------------------------------------------------------
  # PROMETHEUS : collecte et stockage des metriques
  # -------------------------------------------------------
  prometheus:
    image: prom/prometheus:v2.51.0
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/alert_rules.yml:/etc/prometheus/alert_rules.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
    restart: unless-stopped

  # -------------------------------------------------------
  # GRAFANA : visualisation et dashboards
  # -------------------------------------------------------
  grafana:
    image: grafana/grafana:10.4.0
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    depends_on:
      - prometheus
    restart: unless-stopped

  # -------------------------------------------------------
  # ALERTMANAGER : gestion des alertes
  # -------------------------------------------------------
  alertmanager:
    image: prom/alertmanager:v0.27.0
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    restart: unless-stopped

  # -------------------------------------------------------
  # NODE EXPORTER : metriques systeme (CPU, RAM, disque)
  # -------------------------------------------------------
  node-exporter:
    image: prom/node-exporter:v1.7.0
    container_name: node-exporter
    ports:
      - "9100:9100"
    restart: unless-stopped

  # -------------------------------------------------------
  # VOTRE APPLICATION (exemple)
  # -------------------------------------------------------
  app:
    build: ./app
    container_name: my-app
    ports:
      - "8000:8000"
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

### prometheus.yml

```yaml
# prometheus/prometheus.yml
global:
  scrape_interval: 15s       # Frequence de collecte
  evaluation_interval: 15s   # Frequence d'evaluation des regles
  scrape_timeout: 10s

# Regles d'alerte
rule_files:
  - "alert_rules.yml"

# Ou envoyer les alertes
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

# Quelles cibles scraper
scrape_configs:
  # Prometheus se scrape lui-meme
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Metriques systeme via Node Exporter
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  # Votre application
  - job_name: 'my-app'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['app:8000']
    # Labels supplementaires
    relabel_configs:
      - target_label: env
        replacement: production
```

### Grafana provisioning (auto-configuration)

```yaml
# grafana/provisioning/datasources/prometheus.yml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
```

### Lancement

```bash
# Demarrer tout le stack
docker compose up -d

# Verifier que tout tourne
docker compose ps

# Acceder aux interfaces :
# Prometheus : http://localhost:9090
# Grafana    : http://localhost:3000 (admin/admin)
# Alertmanager : http://localhost:9093

# Verifier les targets dans Prometheus :
# http://localhost:9090/targets
# Toutes les targets doivent etre en etat "UP"
```

---

## Bonnes Pratiques de Monitoring

### Que monitorer ?

```
  ┌─────────────────────────────────────────────────┐
  │              PYRAMIDE DU MONITORING              │
  │                                                  │
  │                    /\                             │
  │                   /  \                            │
  │                  / UX  \   Parcours utilisateur   │
  │                 / metier \  (conversion, signup)  │
  │                /──────────\                       │
  │               / Application \  RED method         │
  │              / (API, services)\  (Rate, Errors,   │
  │             /──────────────────\   Duration)      │
  │            /   Infrastructure   \  USE method     │
  │           / (CPU, RAM, disque,   \ (Utilization,  │
  │          /   reseau, conteneurs)  \  Saturation,  │
  │         /──────────────────────────\   Errors)    │
  │        /      Dependances externes  \             │
  │       / (DB, cache, queues, APIs     \            │
  │      /   tierces, DNS, CDN)           \           │
  │     /──────────────────────────────────\          │
  │                                                  │
  └─────────────────────────────────────────────────┘
```

### Conception des dashboards

| Principe | Description |
|---|---|
| **Hierarchie** | Dashboard de vue d'ensemble > Dashboard par service > Dashboard detaille |
| **5 secondes** | Un dashboard doit repondre "est-ce que ca va ?" en 5 secondes |
| **Couleurs** | Vert = OK, Jaune = attention, Rouge = critique. Pas plus. |
| **Pas de vanity metrics** | Chaque panel doit servir a prendre une decision |
| **Annotations** | Marquer les deployments, incidents, changements de config |

### Eviter l'alert fatigue

1. **Chaque alerte doit etre actionnable** : si personne ne peut rien faire, ce n'est pas une alerte
2. **Utiliser `for`** : exiger que la condition soit vraie pendant X minutes avant d'alerter
3. **Severity correcte** : `critical` = action immediate, `warning` = a traiter dans les heures
4. **Revue reguliere** : supprimer les alertes ignorees, ajuster les seuils
5. **Pages vs tickets** : les `critical` reveillent les gens, les `warning` creent des tickets

> [!example] Anti-pattern vs bonne pratique
> **Mauvais** : alerte quand CPU > 80% pendant 1 minute
> - Le CPU peut faire des pics a 90% sans consequence
> - Resultat : 50 alertes par jour, tout le monde les ignore
> 
> **Bon** : alerte quand le P99 de latence > 500ms pendant 10 minutes ET le taux d'erreur > 1%
> - Ca mesure l'**impact utilisateur**, pas la cause technique
> - Resultat : 2 alertes par mois, toujours pertinentes

---

## Carte Mentale

```
                        METRIQUES & MONITORING
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
     FONDAMENTAUX         PROMETHEUS            PRATIQUES
          │                    │                    │
    ┌─────┼─────┐        ┌────┼────┐         ┌────┼────┐
    │     │     │        │    │    │         │    │    │
  Types  Golden PromQL  Archi Instru- Alert  SLI/ RED/ Bonnes
  metric Signals        -tecture ment  -ing  SLO  USE  pratiques
    │     │              │    │    │    │     │
  Counter Latence     Pull  Python Alert  Error  Rate
  Gauge   Trafic      TSDB  Flask  Manager Budget Errors
  Histo   Erreurs     Scrape FastAPI Routing      Duration
  Summary Saturation                Silencing     Utilization
                                                  Saturation
                              GRAFANA
                                │
                        ┌───────┼───────┐
                        │       │       │
                     Panels  Variables  Dashboard
                     (graph,  (template)  as Code
                      stat,              (JSON)
                      gauge)
```

---

## Exercices

### Exercice 1 : Identifier les types de metriques

Pour chaque metrique ci-dessous, indiquez s'il s'agit d'un **Counter**, **Gauge**, **Histogram** ou **Summary** :

1. Nombre total d'emails envoyes
2. Temperature actuelle du serveur
3. Distribution des tailles de fichiers uploades
4. Nombre de connexions actives a la base de donnees
5. Nombre total d'octets transferes sur le reseau
6. Utilisation memoire actuelle en pourcentage

> [!example] Solution
> 1. **Counter** (ne fait que monter)
> 2. **Gauge** (monte et descend)
> 3. **Histogram** (distribution de valeurs)
> 4. **Gauge** (nombre qui fluctue)
> 5. **Counter** (cumul qui ne fait que monter)
> 6. **Gauge** (pourcentage qui varie)

### Exercice 2 : Ecrire des requetes PromQL

Ecrivez les requetes PromQL pour :

1. Le nombre de requetes par seconde sur les 5 dernieres minutes
2. Le taux d'erreur (5xx) en pourcentage
3. Le 95eme percentile de la latence par endpoint
4. Le nombre total de requetes dans la derniere heure

> [!example] Solution
> ```promql
> # 1. Requetes par seconde
> sum(rate(http_requests_total[5m]))
> 
> # 2. Taux d'erreur en %
> sum(rate(http_requests_total{status=~"5.."}[5m]))
> / sum(rate(http_requests_total[5m])) * 100
> 
> # 3. P95 par endpoint
> histogram_quantile(0.95,
>     sum(rate(http_request_duration_seconds_bucket[5m])) by (le, endpoint))
> 
> # 4. Total sur 1 heure
> sum(increase(http_requests_total[1h]))
> ```

### Exercice 3 : Definir des SLO

Pour un service e-commerce, definissez les SLI et SLO pour :
- La page d'accueil
- L'API de paiement
- Le service de recherche

Pensez a la latence, la disponibilite et le taux d'erreur.

> [!example] Solution
> | Service | SLI | SLO |
> |---|---|---|
> | Page d'accueil | Latence P99 | < 800ms pour 99.5% |
> | Page d'accueil | Disponibilite | 99.9% (43 min downtime/mois) |
> | API Paiement | Taux d'erreur | < 0.01% (critique : argent) |
> | API Paiement | Latence P99 | < 2s pour 99.99% |
> | API Paiement | Disponibilite | 99.99% (4.3 min/mois) |
> | Recherche | Latence P50 | < 200ms pour 99% |
> | Recherche | Pertinence | Top-3 resultats pertinents > 90% |

### Exercice 4 : Concevoir un dashboard

Dessinez (sur papier ou dans un outil) un dashboard Grafana pour un microservice API avec :
- Un panneau "Stat" pour le taux d'erreur actuel
- Un graphe temporel des requetes/s par endpoint
- Un graphe temporel du P50/P90/P99 de latence
- Un gauge pour l'utilisation memoire
- Une variable pour filtrer par environnement

Ecrivez les requetes PromQL de chaque panneau.

---

## Liens

- [[01 - Logs et Journalisation]] : le premier pilier de l'observabilite
- [[03 - Tracing et Debugging Distribue]] : le troisieme pilier, suivre une requete a travers les services
- [[03 - Docker Compose en Pratique]] : pour deployer votre stack de monitoring