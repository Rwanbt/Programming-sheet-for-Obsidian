# Logs et Journalisation

## Qu'est-ce que l'Observabilite ?

L'**observabilite** est la capacite a comprendre l'etat interne d'un systeme en observant ses sorties. En production, vous ne pouvez pas attacher un debugger ou mettre des `print()` partout. L'observabilite vous donne les outils pour comprendre ce qui se passe, diagnostiquer les problemes et reagir aux incidents.

L'observabilite repose sur **trois piliers** :

```
                      OBSERVABILITE
                           /|\
                          / | \
                         /  |  \
                        /   |   \
                       /    |    \
                      /     |     \
                     /      |      \
                    /       |       \
                   /        |        \
                  +-------- +--------+
                  |         |        |
                LOGS     METRIQUES  TRACES
                  |         |        |
             Evenements  Chiffres  Parcours
             textuels    mesurables d'une requete
             horodates   dans le   a travers les
                         temps     services
                  
  Logs    : "Erreur 500 a 14:32:05 sur /api/users"
  Metrics : "CPU a 87%, 342 requetes/seconde, latence P99 = 1.2s"
  Traces  : "Requete #abc123 : API (50ms) -> Auth (12ms) -> DB (200ms)"
```

> [!tip] Analogie
> Imaginez que vous etes **medecin** :
> - Les **logs** sont le **carnet de sante** du patient : chaque evenement medical est note avec une date
> - Les **metriques** sont les **constantes vitales** : temperature, rythme cardiaque, tension (des chiffres dans le temps)
> - Les **traces** sont l'**IRM** : vous voyez exactement le trajet d'un signal a travers le corps
>
> Chaque pilier seul est utile. Les trois ensemble vous donnent une vision complete.

---

## Pourquoi l'Observabilite est Essentielle

### En developpement vs en production

| Situation | Developpement | Production |
|---|---|---|
| **Debugger** | Oui (breakpoints, step) | Non (impossible) |
| **print()** | Oui | Non (pas de terminal) |
| **Etat du systeme** | Visible | Opaque sans observabilite |
| **Reproduction** | Facile (meme machine) | Difficile (charge, concurrence) |
| **Nombre d'instances** | 1 | 10, 100, 1000 |

### Cas d'usage concrets

- **Debugging en production** : un utilisateur rapporte une erreur. Les logs vous montrent exactement ce qui s'est passe.
- **Incident response** : le site est lent. Les metriques montrent que la base de donnees est saturee. Les traces revelent qu'une requete specifique fait un scan complet de table.
- **Post-mortem** : apres un incident, les logs permettent de reconstruire la chronologie exacte des evenements.
- **Alerting** : les metriques declenchent une alerte automatique quand le temps de reponse depasse un seuil.

> [!warning] Sans observabilite, vous etes aveugle
> En production, chaque minute d'indisponibilite coute de l'argent et de la reputation. Sans logs ni metriques, vous ne pouvez ni detecter ni diagnostiquer les problemes. L'observabilite n'est pas un luxe, c'est une necessite.

---

## Les Logs : Fondamentaux

### Qu'est-ce qu'un log ?

Un **log** est un enregistrement horodate d'un evenement survenu dans le systeme. C'est la forme la plus ancienne et la plus basique d'observabilite.

```
2025-03-15 14:32:05 INFO  [api.users] Requete GET /api/users - 200 - 45ms
2025-03-15 14:32:06 WARN  [api.auth] Token expire pour user_id=42
2025-03-15 14:32:07 ERROR [api.users] Echec connexion DB: timeout apres 5s
2025-03-15 14:32:07 ERROR [api.users] Traceback (most recent call last):
                          File "app.py", line 42, in get_users
                            results = db.query("SELECT * FROM users")
                          psycopg2.OperationalError: connection timed out
```

### Quoi logger

| Type d'evenement | Exemple | Pourquoi |
|---|---|---|
| **Requetes entrantes** | `GET /api/users 200 45ms` | Comprendre le trafic et les performances |
| **Erreurs** | `ConnectionError: DB timeout` | Diagnostiquer les pannes |
| **Changements d'etat** | `User 42 role changed: user -> admin` | Audit, securite |
| **Demarrage/arret** | `Server started on port 8000` | Savoir quand le service a redemarre |
| **Evenements metier** | `Order #1234 completed, total: 59.99` | Comprendre l'activite |
| **Decisions du systeme** | `Rate limit applied to IP 1.2.3.4` | Comprendre le comportement |

### Quoi NE PAS logger

> [!warning] Donnees sensibles : JAMAIS dans les logs
> - **Mots de passe** (meme haches)
> - **Tokens** d'authentification (JWT, API keys)
> - **Donnees personnelles** (PII) : email, telephone, adresse, numero de carte
> - **Cles de chiffrement** ou secrets
>
> Les logs sont souvent stockes en clair, partages entre equipes, et conserves longtemps. Une donnee sensible dans un log est une faille de securite.

```python
# MAUVAIS
logger.info(f"Login attempt: user={username}, password={password}")
logger.info(f"API call with token: {auth_token}")
logger.info(f"Payment for card: {card_number}")

# BON
logger.info(f"Login attempt: user={username}")
logger.info(f"API call with token: {auth_token[:8]}...")  # Tronquer
logger.info(f"Payment for card: ****{card_number[-4:]}")  # Masquer
```

---

## Niveaux de Log

Les niveaux de log permettent de **filtrer** les messages par gravite :

```
  CRITICAL  ████████████████████████  Le systeme est en panne
  ERROR     ██████████████████████    Quelque chose a echoue
  WARNING   ████████████████████      Attention, probleme potentiel
  INFO      ██████████████████        Tout va bien, evenement normal
  DEBUG     ████████████████          Detail technique pour le dev

  En production : generalement INFO ou WARNING
  En developpement : DEBUG
```

| Niveau | Quand l'utiliser | Exemple |
|---|---|---|
| **DEBUG** | Detail technique pour le developpeur | `DEBUG: Query SQL: SELECT * FROM users WHERE id=42` |
| **INFO** | Evenement normal attendu | `INFO: Server started on port 8000` |
| **WARNING** | Situation anormale mais geree | `WARNING: Disk usage at 85%` |
| **ERROR** | Erreur qui affecte une requete/operation | `ERROR: Failed to process payment for order #1234` |
| **CRITICAL** | Le systeme est en panne ou inutilisable | `CRITICAL: Database connection pool exhausted` |

> [!example] Choisir le bon niveau
> ```python
> # Un utilisateur se connecte -> INFO
> logger.info("User %s logged in successfully", user_id)
> 
> # Le cache est plein, on fonctionne sans -> WARNING
> logger.warning("Cache full, falling back to database")
> 
> # Une requete API echoue -> ERROR
> logger.error("Payment API returned 500 for order %s", order_id)
> 
> # La base de donnees ne repond plus -> CRITICAL
> logger.critical("All database connections failed, service unavailable")
> 
> # Detail d'une requete SQL -> DEBUG
> logger.debug("Executing query: %s with params: %s", query, params)
> ```

---

## Le Module logging de Python

### Configuration de base

```python
import logging

# Configuration la plus simple
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s %(levelname)-8s [%(name)s] %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

# Utilisation directe
logging.info("Application demarree")
logging.warning("Disque presque plein")
logging.error("Erreur de connexion a la base")
```

```
# Sortie :
2025-03-15 14:32:05 INFO     [root] Application demarree
2025-03-15 14:32:05 WARNING  [root] Disque presque plein
2025-03-15 14:32:05 ERROR    [root] Erreur de connexion a la base
```

### getLogger : le logger nomme

```python
import logging

# Creer un logger nomme (convention : nom du module)
logger = logging.getLogger(__name__)

# __name__ = "mon_package.mon_module" -> permet d'identifier l'origine

def process_order(order_id: int) -> None:
    logger.info("Traitement de la commande %d", order_id)
    try:
        # ... traitement ...
        logger.info("Commande %d traitee avec succes", order_id)
    except Exception as e:
        logger.error("Echec traitement commande %d: %s", order_id, e)
        raise
```

> [!info] Pourquoi `logging.getLogger(__name__)` ?
> - Chaque module a son propre logger nomme
> - Les noms suivent la hierarchie des packages : `mon_app.api.users`
> - Vous pouvez configurer des niveaux differents par module
> - En lisant un log, vous savez immediatement d'ou il vient

### Handlers : ou envoyer les logs

Les **handlers** definissent la destination des logs :

```python
import logging
from logging.handlers import RotatingFileHandler

logger = logging.getLogger("mon_app")
logger.setLevel(logging.DEBUG)

# Handler 1 : Console (StreamHandler)
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)  # Console : INFO et au-dessus
console_format = logging.Formatter(
    '%(asctime)s %(levelname)-8s %(message)s'
)
console_handler.setFormatter(console_format)

# Handler 2 : Fichier (FileHandler)
file_handler = logging.FileHandler('app.log')
file_handler.setLevel(logging.DEBUG)  # Fichier : tout y compris DEBUG
file_format = logging.Formatter(
    '%(asctime)s %(levelname)-8s [%(name)s:%(lineno)d] %(message)s'
)
file_handler.setFormatter(file_format)

# Handler 3 : Fichier rotatif (RotatingFileHandler)
rotating_handler = RotatingFileHandler(
    'app_rotating.log',
    maxBytes=10_000_000,  # 10 Mo max par fichier
    backupCount=5          # Garder 5 fichiers de backup
)
rotating_handler.setLevel(logging.WARNING)
rotating_handler.setFormatter(file_format)

# Attacher les handlers au logger
logger.addHandler(console_handler)
logger.addHandler(file_handler)
logger.addHandler(rotating_handler)
```

```
  Architecture des handlers :

  logger.info("message")
         |
         v
  +------+------+
  |   Logger    |  (niveau: DEBUG)
  | "mon_app"   |
  +------+------+
         |
    +----+----+----+
    |         |    |
    v         v    v
  Console   File  Rotating
  (INFO+)  (DEBUG+) (WARNING+)
    |         |    |
    v         v    v
  stdout   app.log  app_rotating.log
                    app_rotating.log.1
                    app_rotating.log.2
                    ...
```

### Formatters : personnaliser l'affichage

| Variable | Description | Exemple |
|---|---|---|
| `%(asctime)s` | Horodatage | `2025-03-15 14:32:05` |
| `%(levelname)s` | Niveau | `INFO`, `ERROR` |
| `%(name)s` | Nom du logger | `mon_app.api.users` |
| `%(module)s` | Nom du module | `users` |
| `%(funcName)s` | Nom de la fonction | `get_users` |
| `%(lineno)d` | Numero de ligne | `42` |
| `%(message)s` | Le message | `User connected` |
| `%(process)d` | PID du processus | `12345` |
| `%(thread)d` | ID du thread | `140234` |

### Propagation et hierarchie

```python
import logging

# Hierarchie des loggers :
#
#   root
#    |
#    +-- mon_app
#         |
#         +-- mon_app.api
#         |    |
#         |    +-- mon_app.api.users
#         |    +-- mon_app.api.orders
#         |
#         +-- mon_app.db

# Configurer le logger parent
app_logger = logging.getLogger("mon_app")
app_logger.setLevel(logging.INFO)
app_logger.addHandler(logging.StreamHandler())

# Les loggers enfants heritent de la configuration du parent
api_logger = logging.getLogger("mon_app.api")
# api_logger herite du handler et du niveau de "mon_app"

users_logger = logging.getLogger("mon_app.api.users")
# users_logger herite aussi

# Logger pour le module DB avec un niveau different
db_logger = logging.getLogger("mon_app.db")
db_logger.setLevel(logging.WARNING)  # Moins verbeux pour la DB
```

> [!warning] Attention a la propagation
> Par defaut, `propagate=True`. Un log emis par `mon_app.api.users` remonte a `mon_app.api`, puis a `mon_app`, puis a `root`. Si chaque logger a un handler, le message sera affiche **plusieurs fois**.
> ```python
> # Desactiver la propagation si necessaire
> users_logger.propagate = False
> ```

---

## Logging Structure (JSON)

### Pourquoi le format structure ?

Les logs textuels classiques sont faciles a lire pour un humain, mais **difficiles a parser** par une machine :

```
# Log textuel classique - difficile a parser
2025-03-15 14:32:05 ERROR [api.users] Failed to get user 42: connection timeout

# Log structure JSON - facile a parser et a indexer
{"timestamp":"2025-03-15T14:32:05Z","level":"ERROR","logger":"api.users",
 "message":"Failed to get user","user_id":42,"error":"connection timeout",
 "duration_ms":5000}
```

Le format JSON permet :
- **Recherche** : trouver tous les logs ou `user_id=42`
- **Filtrage** : afficher uniquement les `level=ERROR`
- **Agregation** : compter les erreurs par endpoint
- **Alertes** : declencher une alerte quand `duration_ms > 3000`

### Avec la bibliotheque structlog

```python
# pip install structlog

import structlog

# Configuration de structlog
structlog.configure(
    processors=[
        structlog.stdlib.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.format_exc_info,
        structlog.processors.JSONRenderer()
    ],
    wrapper_class=structlog.stdlib.BoundLogger,
    logger_factory=structlog.stdlib.LoggerFactory(),
)

logger = structlog.get_logger("api.users")

logger.info("user_login", user_id=42, ip="192.168.1.1")
# {"user_id": 42, "ip": "192.168.1.1", "event": "user_login",
#  "level": "info", "timestamp": "2025-03-15T14:32:05Z"}

logger.error("db_query_failed", duration_ms=5000, error="connection timeout")
# {"duration_ms": 5000, "error": "connection timeout",
#  "event": "db_query_failed", "level": "error", "timestamp": "..."}
```

### Bound loggers (contexte persistant)

```python
# Ajouter du contexte qui persiste entre les appels
logger = structlog.get_logger()
log = logger.bind(request_id="abc-123", user_id=42)

log.info("processing_request")
# {"request_id": "abc-123", "user_id": 42, "event": "processing_request", ...}

log.info("fetching_data", table="users")
# {"request_id": "abc-123", "user_id": 42, "table": "users",
#  "event": "fetching_data", ...}

# Le request_id et user_id sont automatiquement inclus dans chaque log
```

---

## Agregation de Logs : Centraliser les Logs

### Le probleme

```
  Sans agregation :                 Avec agregation :
  
  +--------+                        +--------+
  | App 1  |---> app1.log           | App 1  |---+
  +--------+                        +--------+   |
                                                  |   +-------------+
  +--------+                        +--------+   +-->| Systeme     |
  | App 2  |---> app2.log           | App 2  |---+  | centralise  |
  +--------+                        +--------+   |  | de logs     |
                                                  |  | (recherche, |
  +--------+                        +--------+   |  |  alertes,   |
  | App 3  |---> app3.log           | App 3  |---+  |  dashboards)|
  +--------+                        +--------+      +-------------+
  
  Vous devez SSH sur                Un seul endroit
  chaque machine et                 pour tout chercher
  faire "tail -f" manuellement      et analyser
```

> [!info] Pourquoi centraliser ?
> Avec des dizaines ou centaines d'instances, il est impossible de se connecter a chaque machine pour lire les logs. L'agregation centralise tous les logs dans un systeme unique avec recherche, filtrage et visualisation.

---

## La Stack ELK (Elasticsearch + Logstash + Kibana)

### Architecture

```
  +----------+     +----------+     +--------------+     +---------+
  | App 1    |     |          |     |              |     |         |
  | App 2    |---->| Logstash |---->| Elasticsearch|---->| Kibana  |
  | App 3    |     |          |     |              |     |         |
  | Serveurs |     | Pipeline |     | Indexation   |     | Visu    |
  +----------+     | de       |     | et           |     | et      |
       |           | traitement|     | recherche    |     | tableaux|
   Logs bruts     +----------+     +--------------+     +---------+
                       |                  |                  |
                    Parsing           Stockage           Dashboards
                    Filtrage          Requetes            Alertes
                    Enrichissement    Full-text           Rapports
                    Transformation    Agregation
```

### Elasticsearch

Le **moteur de recherche et d'indexation**. Il stocke les logs et permet des recherches extremement rapides sur des teraoctets de donnees.

```
  Requete Elasticsearch (Kibana Query Language) :
  
  level:ERROR AND service:api AND message:"timeout"
  
  Resultat : tous les logs ERROR du service API contenant "timeout"
  
  Requete plus avancee :
  level:ERROR AND response_time:>5000 AND NOT path:"/health"
```

### Logstash

Le **pipeline de traitement**. Il recoit les logs bruts, les transforme et les envoie a Elasticsearch. Un pipeline Logstash a 3 sections : `input` (source), `filter` (transformation) et `output` (destination).

```ruby
# logstash.conf (simplifie)
input {
  tcp { port => 5000; codec => json }
  file { path => "/var/log/app/*.log" }
}
filter {
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} \[%{DATA:logger}\] %{GREEDYDATA:msg}"
    }
  }
  date { match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]; target => "@timestamp" }
}
output {
  elasticsearch { hosts => ["http://elasticsearch:9200"]; index => "app-logs-%{+YYYY.MM.dd}" }
}
```

### Kibana

L'**interface de visualisation**. Permet de creer des tableaux de bord, des graphiques et des alertes a partir des donnees indexees dans Elasticsearch.

Fonctionnalites principales :
- **Discover** : recherche et exploration des logs en temps reel
- **Dashboards** : graphiques (erreurs/heure, top erreurs, repartition par service)
- **Alertes** : notification quand un seuil est depasse (ex: plus de 100 erreurs/minute)

---

## Alternatives a ELK

### Loki + Grafana

- **Loki** : systeme de logs par Grafana Labs, inspire de Prometheus
- Indexe uniquement les **labels** (pas le contenu complet), donc beaucoup plus leger qu'Elasticsearch
- S'integre parfaitement avec **Grafana** pour la visualisation
- Ideal pour les environnements Kubernetes

### CloudWatch (AWS)

- Service de logs integre a AWS
- Chaque service AWS envoie ses logs automatiquement
- Recherche, filtrage, alertes, metriques basees sur les logs
- Pas besoin de gerer l'infrastructure

### Datadog

- Plateforme SaaS d'observabilite complete (logs + metriques + traces)
- Facile a mettre en place, mais couteuse a grande echelle
- Excellentes alertes et dashboards

| Solution | Type | Cout | Complexite | Meilleur pour |
|---|---|---|---|---|
| **ELK** | Self-hosted | Infra | Elevee | Controle total, gros volumes |
| **Loki+Grafana** | Self-hosted | Infra | Moyenne | Kubernetes, logs legers |
| **CloudWatch** | SaaS (AWS) | Usage | Faible | Ecosysteme AWS |
| **Datadog** | SaaS | Usage | Faible | Observabilite complete |

---

## journald : Logs Systeme Linux

### Concept

**systemd-journald** est le systeme de journalisation de systemd. Il collecte les logs du noyau, des services systemd et des applications.

### Commandes journalctl essentielles

```bash
# Logs d'un service specifique
journalctl -u mon-app.service

# Suivre en temps reel (comme tail -f)
journalctl -f -u mon-app.service

# Filtrer par date et priorite
journalctl --since "1 hour ago" --until "now"
journalctl -p err                # ERROR et au-dessus

# Logs du boot actuel / precedent
journalctl -b                    # Boot actuel
journalctl -b -1                 # Boot precedent

# Format JSON (pour parser avec jq)
journalctl -u mon-app -o json | jq '.'

# Gestion de l'espace disque
journalctl --disk-usage
sudo journalctl --vacuum-time=7d   # Garder 7 jours
sudo journalctl --vacuum-size=500M # Limiter a 500 Mo
```

---

## Log Rotation avec logrotate

### Concept

Les fichiers de log grossissent indefiniment. **logrotate** automatise la rotation (archivage et compression des anciens fichiers, suppression des plus vieux).

### Configuration

```bash
# Fichier : /etc/logrotate.d/mon-app

/var/log/mon-app/*.log {
    daily                  # Rotation quotidienne
    missingok              # Pas d'erreur si le fichier n'existe pas
    rotate 14              # Garder 14 fichiers (14 jours)
    compress               # Compresser les anciens fichiers (.gz)
    delaycompress          # Ne pas compresser le fichier le plus recent
    notifempty             # Ne pas tourner si le fichier est vide
    create 0640 www-data www-data  # Permissions du nouveau fichier
    sharedscripts          # Executer les scripts une seule fois
    postrotate
        # Signaler a l'application de reouvrir les fichiers
        systemctl reload mon-app
    endscript
}
```

```
  Avant rotation :                 Apres rotation :
  
  app.log (500 Mo, actif)          app.log (0 Ko, nouveau, actif)
                                   app.log.1 (500 Mo, hier)
                                   app.log.2.gz (150 Mo, avant-hier)
                                   app.log.3.gz (140 Mo, J-3)
                                   ...
                                   app.log.14.gz (supprime apres 14 jours)
```

```bash
# Tester la configuration (dry run)
logrotate -d /etc/logrotate.d/mon-app

# Forcer une rotation manuelle
logrotate -f /etc/logrotate.d/mon-app
```

---

## Docker Logs

### Commandes de base

```bash
# Voir les logs d'un conteneur
docker logs mon-conteneur

# Suivre en temps reel
docker logs -f mon-conteneur

# Les N derniers logs
docker logs --tail 100 mon-conteneur

# Logs depuis une date
docker logs --since "2025-03-15T14:00:00" mon-conteneur

# Logs avec timestamps
docker logs -t mon-conteneur
```

### Log Drivers

Docker supporte plusieurs **drivers** pour gerer les logs :

| Driver | Description | Usage |
|---|---|---|
| **json-file** | Stocke en JSON local (defaut) | Developpement |
| **syslog** | Envoie a un serveur syslog | Integration systeme |
| **fluentd** | Envoie a Fluentd | Agregation avancee |
| **awslogs** | Envoie a CloudWatch | Environnement AWS |
| **gcplogs** | Envoie a Cloud Logging | Environnement GCP |
| **none** | Pas de logs | Performance maximum |

```bash
# Lancer un conteneur avec un driver specifique
docker run --log-driver=syslog mon-image

# Configurer le driver par defaut dans /etc/docker/daemon.json
# {
#   "log-driver": "json-file",
#   "log-opts": {
#     "max-size": "10m",
#     "max-file": "3"
#   }
# }
```

> [!warning] Le driver json-file ne fait PAS de rotation par defaut
> Sans configuration, les logs Docker en json-file grandissent indefiniment et peuvent remplir le disque. Configurez **toujours** `max-size` et `max-file` :
> ```bash
> docker run \
>   --log-opt max-size=10m \
>   --log-opt max-file=3 \
>   mon-image
> ```

---

## Bonnes Pratiques

### 1. Correlation IDs (identifiants de correlation)

Attribuer un identifiant unique a chaque requete pour la suivre a travers tous les services :

```python
import uuid
import structlog

def middleware(request, call_next):
    request_id = request.headers.get('X-Request-ID', str(uuid.uuid4()))
    logger = structlog.get_logger().bind(request_id=request_id)
    
    logger.info("request_started", method=request.method, path=request.url.path)
    response = call_next(request)
    logger.info("request_completed", status_code=response.status_code)
    
    response.headers['X-Request-ID'] = request_id
    return response
```

Le correlation ID est propage via le header `X-Request-ID` a travers tous les services. Ainsi, en cherchant `request_id=abc-123` dans vos logs centralises, vous retrouvez **tout le parcours** de la requete a travers API, Service A, Service B, etc.

### 2. Timestamps en UTC

```python
import logging

# Toujours utiliser UTC pour les timestamps
logging.Formatter.converter = time.gmtime

formatter = logging.Formatter(
    '%(asctime)s %(levelname)s %(message)s',
    datefmt='%Y-%m-%dT%H:%M:%SZ'  # Format ISO 8601 en UTC
)
```

> [!info] Pourquoi UTC ?
> Si vos serveurs sont dans differentes regions (Paris, New York, Tokyo), chacun a un fuseau horaire different. Utiliser UTC partout evite la confusion et facilite la correlation entre les logs de differents services.

### 3. Niveau de log en production

```python
import os

# Configurer le niveau via une variable d'environnement
log_level = os.environ.get('LOG_LEVEL', 'INFO').upper()
logging.basicConfig(level=getattr(logging, log_level))

# En production : LOG_LEVEL=INFO (ou WARNING)
# En debug d'urgence : LOG_LEVEL=DEBUG (temporaire !)
```

### 4. Ne pas logger trop... ni trop peu

```python
# TROP : noie l'information utile
for item in items:  # 10 000 items
    logger.info("Processing item %s", item.id)  # 10 000 logs !

# PAS ASSEZ : aucune visibilite
def process_batch(items):
    results = [process(item) for item in items]
    return results  # Rien n'est logue

# JUSTE BIEN : log aux points cles
def process_batch(items):
    logger.info("Starting batch processing", count=len(items))
    results = [process(item) for item in items]
    errors = [r for r in results if r.failed]
    logger.info("Batch completed",
        total=len(items),
        success=len(items) - len(errors),
        errors=len(errors)
    )
    for error in errors:
        logger.error("Item failed", item_id=error.item_id, reason=error.reason)
    return results
```

### 5. Utiliser les arguments de formatage (pas les f-strings)

```python
# MAUVAIS : la f-string est evaluee meme si le log est filtre
logger.debug(f"Processing user {user_id} with data {expensive_repr(data)}")

# BON : les arguments ne sont evalues que si le log est emis
logger.debug("Processing user %s with data %s", user_id, data)
```

---

## Carte Mentale ASCII

```
                           LOGS ET JOURNALISATION
                                    |
          +-----------+-------------+-------------+-----------+
          |           |             |             |           |
       Niveaux     Python       Structure     Agregation  Bonnes
          |        logging         |             |        Pratiques
       +--+--+       |        +---+---+     +---+---+       |
       |  |  |    +--+--+     |       |     |   |   |    +--+--+
      DBG INF WRN |  |  |   Text   JSON    ELK Loki CW  |  |  |
      ERR CRI     |  |  |          |        |       DD  Corr UTC
                 Get Hand Form  structlog  +--+--+      ID  Levels
                 Logger ler  atter          |  |  |     Fmt  Rotate
                     |     |            Elastic Log Kib
                   Named  Stream        search  stash ana
                   Hier   File
                   archy  Rotating
```

---

## Exercices

### Exercice 1 : Systeme de logging complet

Creez un module `logger.py` reutilisable qui :
1. Configure un logger avec 3 handlers : console (INFO+), fichier (DEBUG+), fichier rotatif pour les erreurs (ERROR+)
2. Utilise un format different pour la console (simple) et les fichiers (detaille avec timestamp ISO, module, ligne)
3. Expose une fonction `setup_logging(app_name, log_dir)` reutilisable
4. Testez avec un script qui genere des logs a tous les niveaux

### Exercice 2 : Logging structure avec structlog

Creez une mini-API Flask qui :
1. Utilise `structlog` pour des logs JSON
2. Implemente un middleware qui ajoute un `request_id` unique a chaque requete
3. Log chaque requete (methode, chemin, duree, code de reponse)
4. Log les erreurs avec le contexte complet (stack trace, parametres)
5. Verifiez que les logs sont parsables avec `jq`

### Exercice 3 : Analyse de logs avec journalctl

Sur une machine Linux (ou VM) :
1. Creez un service systemd simple qui ecrit des logs
2. Utilisez `journalctl` pour filtrer les logs par service, par date et par priorite
3. Exportez les logs en JSON et analysez-les avec un script Python
4. Configurez la retention des logs (vacuum)

---

## Liens

- [[02 - Metriques et Monitoring]] - Le deuxieme pilier de l'observabilite
- [[03 - Tracing et Debugging Distribue]] - Le troisieme pilier de l'observabilite
- [[05 - Gestion des Erreurs et Fichiers]] - Exceptions et gestion des erreurs en Python
- [[01 - Docker]] - Conteneurs et logs Docker
- [[02 - Services Cloud Essentiels]] - CloudWatch et services de monitoring
