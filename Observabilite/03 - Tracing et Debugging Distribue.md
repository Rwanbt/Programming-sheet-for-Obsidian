# Tracing et Debugging Distribue

## Le Probleme : Ou est le Bug ?

Dans une architecture **monolithique**, une requete traverse une seule application. Le debugging est simple : un stack trace suffit. Dans une architecture **microservices**, une seule requete utilisateur peut traverser **10, 20 ou 50 services** differents. Quand ca ralentit ou quand ca casse, comment savoir **ou** est le probleme ?

```
  REQUETE UTILISATEUR : "Afficher la page produit"
  
  Navigateur
      │
      ▼
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │   API    │────>│  Auth    │────>│  Cache   │
  │ Gateway  │     │ Service  │     │  Redis   │
  └────┬─────┘     └──────────┘     └──────────┘
       │
       ├─────────────────────┐
       │                     │
       ▼                     ▼
  ┌──────────┐         ┌──────────┐
  │ Product  │         │  Price   │
  │ Service  │         │ Service  │
  └────┬─────┘         └────┬─────┘
       │                    │
       ▼                    ▼
  ┌──────────┐         ┌──────────┐
  │ Inventory│         │ Discount │
  │ Service  │         │ Service  │
  └────┬─────┘         └────┬─────┘
       │                    │
       ▼                    ▼
  ┌──────────┐         ┌──────────┐
  │ Postgres │         │ MongoDB  │
  └──────────┘         └──────────┘

  La requete a traverse 8 composants.
  Temps total : 2.3 secondes (trop lent !)
  
  QUESTION : quel service est responsable du ralentissement ?
  SANS TRACING : on ne sait pas. On devine. On blames le mauvais service.
  AVEC TRACING : on voit exactement que Inventory Service -> Postgres
                 prend 1.8s a cause d'une requete SQL sans index.
```

> [!tip] Analogie
> Imaginez un **colis postal** qui doit passer par 8 bureaux de poste avant d'arriver chez vous. Le colis est en retard. Sans tracing, vous appelez chaque bureau un par un en demandant "c'est vous le probleme ?". Avec tracing, chaque bureau **tamponnes le colis** avec l'heure d'arrivee et de depart. En regardant les tampons, vous voyez instantanement ou le colis est reste bloque.

---

## Qu'est-ce que le Tracing Distribue ?

Le **tracing distribue** est la capacite a suivre le **parcours complet** d'une requete a travers tous les services d'un systeme distribue. Chaque etape du parcours est enregistree avec :

- **Quel** service a ete appele
- **Combien de temps** ca a pris
- **Quel a ete le resultat** (succes/erreur)
- Des **metadonnees** supplementaires (requete SQL, parametres, etc.)

> [!info] Les 3 piliers de l'observabilite
> - **Logs** : "que s'est-il passe ?" (evenements textuels)
> - **Metriques** : "combien et a quelle vitesse ?" (chiffres agreges)
> - **Traces** : "ou et dans quel ordre ?" (parcours d'une requete)
>
> Les traces sont le **troisieme pilier**, celui qui donne la vision bout-en-bout.

---

## Concepts Cles

### Trace

Une **trace** represente le **parcours complet** d'une requete, de son point d'entree jusqu'a la reponse finale. C'est un arbre de spans.

### Span

Un **span** represente **une operation** au sein d'une trace. Chaque span a :
- Un **nom** (ex: `GET /api/products`)
- Un **debut** et une **duree**
- Un **statut** (OK, ERROR)
- Des **attributs** (cles/valeurs)
- Des **evenements** (logs attaches au span)
- Un **parent** (sauf le span racine)

```
  TRACE : abc-123-def (requete "Afficher produit #42")
  
  Temps ──────────────────────────────────────────────>
  
  ├── API Gateway: GET /products/42 ─────────────────────────┤ 450ms
  │   │                                                       │
  │   ├── Auth Service: verify_token ────┤ 15ms               │
  │   │                                                       │
  │   ├── Product Service: get_product ──────────────────┤    │
  │   │   │                                    350ms     │    │
  │   │   ├── Cache: GET product:42 ─┤ 2ms               │    │
  │   │   │   (cache MISS)                               │    │
  │   │   │                                              │    │
  │   │   ├── Postgres: SELECT * FROM... ───────┤ 320ms  │    │
  │   │   │   ^^^^ BOTTLENECK ICI !                      │    │
  │   │   │                                              │    │
  │   │   └── Cache: SET product:42 ─┤ 3ms               │    │
  │   │                                                       │
  │   └── Price Service: get_price ──────┤ 45ms               │
  │       │                                                   │
  │       └── Discount: check_promo ─┤ 12ms                   │
  │                                                           │
  └───────────────────────────────────────────────────────────┘
  
  DIAGNOSTIC : la requete Postgres prend 320ms sur 450ms total (71%)
               => ajouter un index sur la colonne product_id
```

### Span Context

Le **span context** est l'information qui voyage entre les services pour les relier ensemble :

```
  Span Context = {
      trace_id  : "abc123def456789"    # Identifiant unique de la trace
      span_id   : "span-789"          # Identifiant unique du span actuel
      trace_flags: "01"               # Flags (ex: sampled = oui/non)
  }
```

### Propagation

La **propagation** est le mecanisme par lequel le span context est transmis d'un service a l'autre. Sans propagation, chaque service genererait des traces separees et independantes.

```
  SERVICE A                          SERVICE B
  ┌─────────────────┐               ┌─────────────────┐
  │                 │               │                 │
  │  Span "call B"  │               │  Span "handle"  │
  │  trace_id: abc  │──── HTTP ────>│  trace_id: abc  │
  │  span_id: s1    │   Header:     │  span_id: s2    │
  │                 │  traceparent  │  parent_id: s1  │
  │                 │               │                 │
  └─────────────────┘               └─────────────────┘
  
  Le header HTTP "traceparent" transporte le contexte.
  Service B sait qu'il fait partie de la meme trace "abc"
  et que son parent est le span "s1" de Service A.
```

> [!info] Formats de propagation
> - **W3C Trace Context** : le standard (recommande)
>   `traceparent: 00-abc123def456789-span789-01`
> - **B3** : format Zipkin (plus ancien)
>   `X-B3-TraceId: abc123def456789`
> - **Jaeger** : format proprietaire Jaeger
>   `uber-trace-id: abc123:span789:0:1`

---

## OpenTelemetry

### Qu'est-ce que c'est ?

**OpenTelemetry** (OTel) est le **standard unifie** pour l'observabilite. C'est un projet CNCF qui fusionne OpenTracing et OpenCensus. Il fournit des APIs, des SDKs et des outils pour collecter des traces, des metriques et des logs.

```
  AVANT OPENTELEMETRY :
  
  Application ──> SDK Jaeger ──> Jaeger
  Application ──> SDK Zipkin ──> Zipkin
  Application ──> SDK Datadog ──> Datadog
  
  Probleme : changer de backend = re-instrumenter toute l'application
  
  AVEC OPENTELEMETRY :
  
                                     ┌──> Jaeger
  Application ──> SDK OTel ──> OTel  ├──> Zipkin
                               Coll. ├──> Tempo
                                     ├──> Datadog
                                     └──> New Relic
  
  Avantage : instrumenter UNE FOIS, envoyer vers N'IMPORTE QUEL backend
```

### Composants d'OpenTelemetry

| Composant | Role |
|---|---|
| **API** | Interface pour instrumenter le code (stable, ne change pas) |
| **SDK** | Implementation de l'API (configure les exporters, samplers, etc.) |
| **Collector** | Composant standalone qui recoit, traite et exporte les donnees |
| **Auto-instrumentation** | Instrumente automatiquement les frameworks populaires |
| **Exporters** | Envoie les donnees vers un backend (Jaeger, OTLP, Zipkin, etc.) |

### Langages supportes

Python, Java, Go, JavaScript/TypeScript, .NET, Ruby, PHP, Rust, C++, Swift, Erlang/Elixir.

---

## Instrumenter une Application Python

### Installation

```bash
# SDK et API de base
pip install opentelemetry-api opentelemetry-sdk

# Exporter OTLP (protocol standard)
pip install opentelemetry-exporter-otlp

# Auto-instrumentation pour frameworks populaires
pip install opentelemetry-instrumentation-flask
pip install opentelemetry-instrumentation-fastapi
pip install opentelemetry-instrumentation-requests
pip install opentelemetry-instrumentation-sqlalchemy

# Ou installer tout d'un coup
pip install opentelemetry-distro
opentelemetry-bootstrap -a install
```

### Configuration de base

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource

# -------------------------------------------------------
# 1. DEFINIR LA RESSOURCE (qui suis-je ?)
# -------------------------------------------------------
resource = Resource.create({
    "service.name": "product-service",
    "service.version": "1.2.0",
    "deployment.environment": "production",
})

# -------------------------------------------------------
# 2. CONFIGURER LE TRACER PROVIDER
# -------------------------------------------------------
provider = TracerProvider(resource=resource)

# Exporter les spans vers le Collector OTel (ou directement vers Jaeger)
otlp_exporter = OTLPSpanExporter(
    endpoint="http://localhost:4317",  # OTel Collector gRPC
)
provider.add_span_processor(BatchSpanProcessor(otlp_exporter))

# Enregistrer le provider globalement
trace.set_tracer_provider(provider)

# -------------------------------------------------------
# 3. OBTENIR UN TRACER
# -------------------------------------------------------
tracer = trace.get_tracer("product-service", "1.2.0")
```

### Auto-instrumentation Flask

```python
from flask import Flask
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.instrumentation.sqlalchemy import SQLAlchemyInstrumentor

app = Flask(__name__)

# Auto-instrumenter Flask (chaque requete HTTP = un span automatique)
FlaskInstrumentor().instrument_app(app)

# Auto-instrumenter les appels HTTP sortants (requests)
RequestsInstrumentor().instrument()

# Auto-instrumenter SQLAlchemy (chaque requete SQL = un span)
# SQLAlchemyInstrumentor().instrument(engine=db_engine)
```

### Instrumentation manuelle

```python
from opentelemetry import trace

tracer = trace.get_tracer("product-service")

# -------------------------------------------------------
# SPAN SIMPLE
# -------------------------------------------------------
def get_product(product_id: int):
    with tracer.start_as_current_span("get_product") as span:
        # Ajouter des attributs
        span.set_attribute("product.id", product_id)
        span.set_attribute("db.system", "postgresql")
        
        product = db.query(f"SELECT * FROM products WHERE id = {product_id}")
        
        if product is None:
            span.set_status(trace.StatusCode.ERROR, "Product not found")
            span.set_attribute("error.type", "NotFound")
            raise ProductNotFound(product_id)
        
        span.set_attribute("product.name", product.name)
        span.set_attribute("product.category", product.category)
        return product


# -------------------------------------------------------
# SPANS IMBRIQUES (parent/enfant automatique)
# -------------------------------------------------------
def process_order(order_data: dict):
    with tracer.start_as_current_span("process_order") as span:
        span.set_attribute("order.items_count", len(order_data["items"]))
        
        # Ce span sera automatiquement enfant de "process_order"
        with tracer.start_as_current_span("validate_order"):
            validate(order_data)
        
        # Celui-ci aussi
        with tracer.start_as_current_span("charge_payment") as payment_span:
            payment_span.set_attribute("payment.method", order_data["payment_method"])
            payment_span.set_attribute("payment.amount", order_data["total"])
            charge(order_data)
        
        with tracer.start_as_current_span("send_confirmation"):
            send_email(order_data["email"])


# -------------------------------------------------------
# AJOUTER DES EVENEMENTS (logs dans un span)
# -------------------------------------------------------
def risky_operation():
    with tracer.start_as_current_span("risky_operation") as span:
        span.add_event("Starting operation", {"attempt": 1})
        
        try:
            result = do_something_risky()
            span.add_event("Operation succeeded", {"result_size": len(result)})
        except Exception as e:
            span.add_event("Operation failed", {"error": str(e)})
            span.set_status(trace.StatusCode.ERROR, str(e))
            span.record_exception(e)
            raise
```

> [!warning] Ne tracez pas tout !
> Chaque span a un cout (memoire, reseau, stockage). Ne creez pas de spans pour des operations triviales (addition de deux nombres, formatage de string). Tracez les operations **significatives** : appels reseau, requetes DB, operations I/O, logique metier importante.

---

## Propagation du Contexte (W3C Trace Context)

### Le header `traceparent`

```
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
             ││ │                                │ │                │ ││
             ││ │         trace-id (128 bits)     │ │   span-id     │ ││
             ││ └────────────────────────────────┘ └──(64 bits)────┘ ││
             │└── version (toujours "00")                            │└── flags
             │                                                       │    01 = sampled
             └───────────────────────────────────────────────────────┘
```

### Comment ca marche entre services HTTP

```python
import requests
from opentelemetry import trace
from opentelemetry.propagate import inject

tracer = trace.get_tracer("service-a")

def call_service_b(user_id: int):
    with tracer.start_as_current_span("call_service_b") as span:
        span.set_attribute("user.id", user_id)
        
        # Les headers de propagation sont injectes automatiquement
        # si vous avez instrumente la librairie requests
        response = requests.get(
            f"http://service-b:8001/api/users/{user_id}"
        )
        # Le header traceparent est ajoute automatiquement :
        # traceparent: 00-<trace_id>-<span_id>-01
        
        return response.json()
```

```
  SERVICE A                           SERVICE B
  ┌────────────────┐                 ┌────────────────┐
  │                │                 │                │
  │  with span:    │                 │  Flask recoit: │
  │  "call svc B"  │    HTTP GET     │  traceparent   │
  │                │ ──────────────> │  header        │
  │  trace: t1     │  traceparent:   │                │
  │  span: s1      │  00-t1-s1-01   │  Cree span:    │
  │                │                 │  "handle GET"  │
  │                │                 │  trace: t1     │
  │                │                 │  span: s2      │
  │                │  <───────────── │  parent: s1    │
  │                │    200 OK       │                │
  └────────────────┘                 └────────────────┘
  
  Meme trace_id "t1" dans les deux services !
  Le span s2 a pour parent s1 => on voit la relation.
```

---

## OpenTelemetry Collector

Le **Collector** est un composant intermediaire qui recoit, traite et exporte les donnees de telemetrie. C'est le **hub central** de votre pipeline d'observabilite.

```
  APPLICATIONS                    COLLECTOR                    BACKENDS
  
  ┌──────────┐               ┌─────────────────────────────┐
  │ Service A│──┐            │                             │
  └──────────┘  │            │  ┌──────────┐               │    ┌────────┐
                │ OTLP/gRPC  │  │RECEIVERS │               │    │ Jaeger │
  ┌──────────┐  ├──────────> │  │          │               ├──> │        │
  │ Service B│──┘            │  │ - OTLP   │  ┌──────────┐│    └────────┘
  └──────────┘               │  │ - Jaeger │─>│PROCESSORS││
                             │  │ - Zipkin │  │          ││    ┌────────┐
  ┌──────────┐  Prometheus   │  └──────────┘  │ - Batch  ││    │Promethe│
  │ Node     │──scrape─────> │                │ - Filter │├──> │   us   │
  │ Exporter │               │                │ - Sample ││    └────────┘
  └──────────┘               │                │ - Enrich ││
                             │                └──────────┘│    ┌────────┐
                             │                    │       │    │  Tempo │
                             │                    ▼       ├──> │        │
                             │  ┌──────────────┐  │       │    └────────┘
                             │  │  EXPORTERS   │<─┘       │
                             │  │              │          │    ┌────────┐
                             │  │ - OTLP       │          │    │Datadog │
                             │  │ - Jaeger     │          ├──> │        │
                             │  │ - Prometheus │          │    └────────┘
                             │  └──────────────┘          │
                             │                             │
                             └─────────────────────────────┘
```

### Configuration du Collector

```yaml
# otel-collector-config.yml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  # Regrouper les spans avant export (performance)
  batch:
    timeout: 5s
    send_batch_size: 1024

  # Echantillonnage : ne garder qu'un % des traces
  probabilistic_sampler:
    sampling_percentage: 10    # 10% des traces

  # Ajouter des attributs a tous les spans
  resource:
    attributes:
      - key: environment
        value: production
        action: upsert

exporters:
  # Exporter vers Jaeger
  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true

  # Exporter les metriques vers Prometheus
  prometheus:
    endpoint: 0.0.0.0:8889

  # Logs de debug (utile en developpement)
  logging:
    loglevel: debug

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch, resource]
      exporters: [otlp/jaeger, logging]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]
```

---

## Backends de Tracing

### Jaeger

**Jaeger** est le backend open-source le plus populaire, cree par Uber et donne a la CNCF.

```bash
# Lancer Jaeger en local (tout-en-un)
docker run -d --name jaeger \
  -p 16686:16686 \    # UI web
  -p 4317:4317 \      # OTLP gRPC
  -p 4318:4318 \      # OTLP HTTP
  jaegertracing/all-in-one:1.55
```

Interface Jaeger (`http://localhost:16686`) :
- **Search** : trouver des traces par service, operation, duree, tags
- **Trace Detail** : vue en cascade (waterfall) de tous les spans
- **Compare** : comparer deux traces cote a cote
- **Dependencies** : graphe des dependances entre services

### Comparaison des backends

| Backend | Type | Stockage | Avantage |
|---|---|---|---|
| **Jaeger** | Open source | Cassandra, ES, Badger | Mature, reference CNCF |
| **Zipkin** | Open source | Cassandra, ES, MySQL | Simple, leger |
| **Tempo** | Open source (Grafana) | Object storage (S3) | Pas d'index, cout faible |
| **Datadog** | Commercial | Cloud | APM complet, correlation logs/traces |
| **New Relic** | Commercial | Cloud | Observabilite tout-en-un |
| **Honeycomb** | Commercial | Cloud | Exploration interactive des traces |

> [!tip] Analogie
> - **Jaeger** = l'**appareil photo professionnel** : puissant, configurable, open source
> - **Tempo** = le **telephone** : moins de fonctions mais stockage quasi-illimite et pas cher
> - **Datadog** = le **studio photo tout-inclus** : tout est fait pour vous, mais ca coute cher

---

## Correlation : Lier Logs et Traces

La vraie puissance de l'observabilite vient quand on **lie** les traces aux logs. Un log devient beaucoup plus utile quand on sait a quelle trace il appartient.

### Structured Logging avec trace_id

```python
import logging
import json
from opentelemetry import trace

class TraceContextFilter(logging.Filter):
    """Ajoute trace_id et span_id a chaque log automatiquement."""
    
    def filter(self, record):
        span = trace.get_current_span()
        ctx = span.get_span_context()
        
        if ctx.is_valid:
            record.trace_id = format(ctx.trace_id, '032x')
            record.span_id = format(ctx.span_id, '016x')
        else:
            record.trace_id = "0" * 32
            record.span_id = "0" * 16
        
        return True

# Configurer le logger
logger = logging.getLogger("product-service")
logger.addFilter(TraceContextFilter())

formatter = logging.Formatter(
    '%(asctime)s %(levelname)s [trace_id=%(trace_id)s span_id=%(span_id)s] '
    '%(name)s - %(message)s'
)

# Resultat dans les logs :
# 2025-03-15 14:32:05 ERROR [trace_id=4bf92f3577b34da6 span_id=00f067aa0ba902b7]
#   product-service - Product #42 not found in database
#
# En copiant le trace_id, on peut retrouver la trace complete dans Jaeger !
```

> [!example] Workflow de debugging avec correlation
> 1. Alerte : "taux d'erreur > 1%" (metrique)
> 2. Dashboard Grafana : pic d'erreurs sur `product-service` (metrique)
> 3. Logs : filtrer par `status=500` sur `product-service` -> voir les `trace_id` (log)
> 4. Jaeger : coller le `trace_id` -> voir le parcours complet (trace)
> 5. Diagnostic : le span `Postgres: SELECT` montre un timeout de 30s
> 6. Resolution : ajouter un index, le P99 redescend

---

## Exemple Pratique : Tracer 2 Services Python

### Service A (API Gateway)

```python
# service_a.py
from flask import Flask, jsonify
import requests
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor

# Configuration OTel
resource = Resource.create({"service.name": "api-gateway"})
provider = TracerProvider(resource=resource)
provider.add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="http://localhost:4317"))
)
trace.set_tracer_provider(provider)

tracer = trace.get_tracer("api-gateway")

app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()

@app.route("/api/products/<int:product_id>")
def get_product(product_id):
    with tracer.start_as_current_span("fetch_product_details") as span:
        span.set_attribute("product.id", product_id)
        
        # Appeler Service B - le traceparent est injecte automatiquement
        response = requests.get(
            f"http://localhost:8001/internal/products/{product_id}"
        )
        
        if response.status_code == 200:
            return jsonify(response.json())
        else:
            span.set_status(trace.StatusCode.ERROR)
            return jsonify({"error": "Product not found"}), 404

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

### Service B (Product Service)

```python
# service_b.py
from flask import Flask, jsonify
import time
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource
from opentelemetry.instrumentation.flask import FlaskInstrumentor

resource = Resource.create({"service.name": "product-service"})
provider = TracerProvider(resource=resource)
provider.add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="http://localhost:4317"))
)
trace.set_tracer_provider(provider)

tracer = trace.get_tracer("product-service")

app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)

PRODUCTS = {
    1: {"id": 1, "name": "Laptop Pro", "price": 1299.99},
    2: {"id": 2, "name": "Wireless Mouse", "price": 49.99},
}

@app.route("/internal/products/<int:product_id>")
def get_product(product_id):
    with tracer.start_as_current_span("db_query") as span:
        span.set_attribute("db.system", "postgresql")
        span.set_attribute("db.statement", f"SELECT * FROM products WHERE id = {product_id}")
        
        time.sleep(0.05)  # Simule une requete DB
        
        product = PRODUCTS.get(product_id)
        if product:
            span.set_attribute("db.rows_affected", 1)
            return jsonify(product)
        else:
            span.set_status(trace.StatusCode.ERROR, "Not found")
            return jsonify({"error": "Not found"}), 404

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8001)
```

### Docker Compose pour tout lancer

```yaml
version: '3.8'
services:
  jaeger:
    image: jaegertracing/all-in-one:1.55
    ports:
      - "16686:16686"   # UI
      - "4317:4317"     # OTLP gRPC

  service-a:
    build: ./service-a
    ports:
      - "8000:8000"
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://jaeger:4317

  service-b:
    build: ./service-b
    ports:
      - "8001:8001"
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://jaeger:4317
```

---

## Debugging avec les Traces

### Trouver les spans lents

Dans l'interface Jaeger :
1. Chercher les traces avec une **duree elevee** (ex: > 1s)
2. Ouvrir la vue **waterfall** : chaque span est une barre horizontale
3. Les **spans les plus longs** sont les bottlenecks
4. Verifier les **attributs** du span (requete SQL, URL appelee, etc.)

### Identifier les erreurs

```
  TRACE AVEC ERREUR :
  
  ├── API Gateway: GET /checkout ───────────────────────┤ 2.1s
  │   │                                                  │
  │   ├── Cart Service: get_cart ────────┤ 120ms          │
  │   │   └── Redis: GET cart:user42 ─┤ 3ms              │
  │   │                                                  │
  │   ├── Payment Service: charge ──────────────┤ 1.8s   │
  │   │   │                                     │        │
  │   │   ├── Stripe API: create_charge ────┤ 1.5s      │
  │   │   │   status: ERROR                 │            │
  │   │   │   error: "Card declined"        │            │
  │   │   │   ^^^^ CAUSE RACINE             │            │
  │   │   │                                              │
  │   │   └── Retry: create_charge ─────────┤ 250ms     │
  │   │       status: ERROR                              │
  │   │                                                  │
  │   └── status: ERROR (propage depuis Payment)         │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

> [!warning] Erreurs en cascade
> Dans un systeme distribue, une erreur dans un service se propage souvent en cascade. Le tracing vous montre la **cause racine** et non les symptomes. Sans tracing, vous verriez "API Gateway 500" et vous chercheriez le bug dans le gateway, alors que le vrai probleme est dans Stripe.

---

## Bonnes Pratiques

### Echantillonnage (Sampling)

En production, tracer **100% des requetes** est souvent trop couteux. L'echantillonnage permet de ne garder qu'une fraction representative :

| Strategie | Description | Usage |
|---|---|---|
| **Head-based** | Decision au debut de la trace (ex: 10%) | Simple, previsible |
| **Tail-based** | Decision a la fin (garder les erreurs, les lentes) | Plus intelligent, plus complexe |
| **Rate limiting** | Maximum N traces par seconde | Protege contre les pics |

```python
from opentelemetry.sdk.trace.sampling import TraceIdRatioBased

# Garder 10% des traces
sampler = TraceIdRatioBased(0.1)
provider = TracerProvider(resource=resource, sampler=sampler)
```

### Que tracer ?

- **Toujours** : appels HTTP entre services, requetes DB, operations de cache, appels a des APIs externes
- **Souvent** : logique metier importante, jobs asynchrones, messages dans les queues
- **Jamais** : operations triviales (calculs en memoire, formatage de texte)

### Attributs recommandes

```python
# Conventions semantiques OpenTelemetry
span.set_attribute("http.method", "GET")
span.set_attribute("http.url", "https://api.example.com/users")
span.set_attribute("http.status_code", 200)
span.set_attribute("db.system", "postgresql")
span.set_attribute("db.statement", "SELECT * FROM users WHERE id = $1")
span.set_attribute("messaging.system", "rabbitmq")
span.set_attribute("messaging.destination", "orders-queue")
```

> [!warning] Donnees sensibles
> Ne mettez **jamais** de donnees personnelles dans les attributs de spans :
> - Pas de mots de passe, tokens, cles API
> - Pas de donnees PII (nom, email, numero de carte)
> - Attention aux requetes SQL qui contiennent des valeurs utilisateur
> - Utilisez des placeholders : `SELECT * FROM users WHERE id = $1` et non `WHERE id = 42`

---

## Carte Mentale

```
                    TRACING DISTRIBUE
                          │
         ┌────────────────┼────────────────┐
         │                │                │
    CONCEPTS         OPENTELEMETRY      PRATIQUES
         │                │                │
    ┌────┼────┐     ┌────┼────┐     ┌────┼────┐
    │    │    │     │    │    │     │    │    │
  Trace Span Context API  SDK Collector Debug Sampling Correlation
    │    │   Propag.       │    │       │         │
    │  Nom     W3C     Auto-  Receivers Waterfall Logs+Traces
    │  Duree   Trace   instr. Process.  Bottleneck trace_id
    │  Status  Context        Exporters Erreurs
    │  Attrib.                          cascade
    │
  trace_id                    BACKENDS
  span_id               ┌──────┼──────┐
  flags                 │      │      │
                     Jaeger  Tempo  Commercial
                     (CNCF) (Grafana) (Datadog
                                      New Relic)
```

---

## Exercices

### Exercice 1 : Lire une trace

Voici une trace simplifiee. Identifiez le bottleneck et proposez une solution :

```
  ├── GET /api/dashboard ────────────────────────── 3.2s
  │   ├── Auth: verify_token ──── 45ms
  │   ├── UserService: get_profile ──── 80ms
  │   ├── OrderService: get_recent_orders ──────── 2.8s
  │   │   ├── MongoDB: find({user_id: 42}) ──── 2.7s
  │   │   └── Format results ──── 50ms
  │   └── NotificationService: get_unread ──── 120ms
```

> [!example] Solution
> Le bottleneck est **MongoDB: find** qui prend 2.7s sur 3.2s total (84%).
> Solutions possibles :
> - Ajouter un **index** sur le champ `user_id` dans la collection orders
> - Ajouter un **cache** (Redis) pour les commandes recentes
> - **Limiter** le nombre de resultats retournes (pagination)
> - Verifier si la requete peut etre **optimisee** (projection, etc.)

### Exercice 2 : Instrumenter une fonction

Ecrivez le code d'instrumentation OpenTelemetry pour une fonction `send_email(to, subject, body)` qui :
- Cree un span nomme "send_email"
- Ajoute des attributs : destinataire (sans l'email complet), sujet
- Gere les erreurs
- Enregistre un evenement quand l'email est envoye

### Exercice 3 : Concevoir une strategie de sampling

Pour un service qui recoit 50 000 requetes par seconde :
- Calculez le volume de donnees si on trace tout (estimation : 1 KB par span, 5 spans par trace)
- Proposez une strategie de sampling adaptee
- Expliquez comment s'assurer que les traces d'erreur sont toujours conservees

> [!example] Solution
> - Volume total : 50 000 * 5 * 1 KB = 250 MB/s = **21.6 TB/jour**
> - Avec sampling a 1% : 2.16 TB/jour (encore beaucoup)
> - Avec sampling a 0.1% : 216 GB/jour (raisonnable)
> - Strategie recommandee : **tail-based sampling** via le Collector OTel
>   - Garder 100% des traces avec erreur
>   - Garder 100% des traces lentes (> 1s)
>   - Garder 0.1% des traces normales
>   - Rate limiter a 1000 traces/s maximum

---

## Liens

- [[01 - Logs et Journalisation]] : le premier pilier de l'observabilite, les logs
- [[02 - Metriques et Monitoring]] : le deuxieme pilier, les metriques et Prometheus