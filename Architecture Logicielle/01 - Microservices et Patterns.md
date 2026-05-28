# Microservices et Patterns d'Architecture

## Pourquoi l'Architecture Logicielle ?

L'architecture logicielle, c'est l'art de prendre des décisions structurelles **avant** d'écrire du code. Une mauvaise architecture se paye très cher : des équipes qui se bloquent mutuellement, des déploiements qui durent des heures, des bugs qui touchent tout le système, et des fonctionnalités qui deviennent exponentiellement difficiles à ajouter.

> [!tip] Analogie
> L'architecture d'une ville. Si vous planifiez mal (routes qui convergent toutes vers le centre, pas de zones piétonnes, parking insuffisant), vous ne pouvez pas "réparer" ça facilement. En logiciel, c'est pareil : les décisions architecturales initiales conditionnent tout ce qui vient après. Contrairement à une ville, on peut tout de même refactorer — mais le coût est élevé.

---

## Monolithe vs Microservices

### Le Monolithe

Un monolithe est une application où **tout le code est déployé ensemble** comme une seule unité.

```
Architecture Monolithique :
┌─────────────────────────────────────────────────────────┐
│                    APPLICATION                           │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │   Auth   │  │ Produits │  │  Panier  │  │Paiement│  │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Base de données unique               │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                         ↓ déploiement
                  TOUT ou RIEN
```

**Avantages du monolithe :**
- Simple à développer au début (pas de réseau entre services)
- Débogage facile (un seul processus, stack traces complètes)
- Transactions ACID triviales (une seule base)
- Tests d'intégration simples
- Déploiement simple (un artefact)
- Latence faible (appels de fonction, pas HTTP)

**Inconvénients du monolithe :**
- **Scaling difficile** : on doit scaler toute l'app même si seul un module est surchargé
- **Déploiement risqué** : un bug dans le module Paiement peut faire crasher toute l'app
- **Couplage** : les modules deviennent interdépendants, les changements cassent d'autres modules
- **Technologie unique** : toute l'app en Java, impossible de mettre un module en Python
- **Équipes bloquées** : 10 développeurs sur le même repo = conflits permanents
- **Temps de build** : 20 min pour compiler et démarrer l'app entière

### Les Microservices

Une architecture en microservices décompose l'application en **petits services indépendants**, chacun responsable d'une capacité métier.

```
Architecture Microservices :

  Client (Web / Mobile)
         ↓
  ┌──────────────┐
  │  API Gateway │  ← Point d'entrée unique
  └──────┬───────┘
         ↓ routing
┌────────┬────────┬────────┬────────┐
│Service │Service │Service │Service │
│  Auth  │Produits│ Panier │Paiement│
│  :3001 │  :3002 │  :3003 │  :3004 │
└───┬────┴───┬────┴───┬────┴───┬────┘
    │        │        │        │
  DB Auth  DB Prod  DB Cart  DB Pay
(Postgres)(MongoDB)(Redis) (Postgres)
```

**Avantages :**
- **Scaling indépendant** : si le service Produits est surchargé → scaler que lui
- **Déploiement indépendant** : update Paiement sans toucher le reste
- **Isolation des pannes** : un service down n'affecte pas les autres (avec circuit breaker)
- **Technologie adaptée** : service ML en Python, service API en Go, service Auth en Node
- **Équipes autonomes** : chaque équipe possède son service de bout en bout
- **Code plus petit** : chaque service est compréhensible seul

**Inconvénients :**
- **Complexité opérationnelle** : 20 services à déployer, monitorer, déboguer
- **Réseau** : chaque appel inter-service = latence + risque de panne réseau
- **Transactions distribuées** : impossible de faire un COMMIT atomique entre services
- **Cohérence des données** : la même donnée peut être obsolète dans un service
- **Overhead de développement** : API contracts, versioning, discovery
- **Debugging difficile** : un bug peut traverser 5 services — nécessite du tracing distribué (voir [[03 - Tracing et Debugging Distribue]])

> [!warning] Ne pas se précipiter vers les microservices
> Martin Fowler (inventeur du terme "microservices") recommande de **commencer par un monolithe modulaire**. Les microservices sont justifiés quand : (1) les équipes sont bloquées mutuellement, (2) certains modules nécessitent un scaling indépendant, (3) vous avez besoin de différentes technologies. Pas avant.

### Monolithe Modulaire — Le meilleur des deux mondes

```
Monolithe Modulaire :
┌────────────────────────────────────────────┐
│                APPLICATION                  │
│  ┌───────────┐  ┌───────────┐              │
│  │  Module   │  │  Module   │              │
│  │   Auth    │  │ Produits  │              │
│  │           │  │           │              │
│  │  Bounded  │  │  Bounded  │  ← APIs     │
│  │  Context  │  │  Context  │    explicites│
│  └───────────┘  └───────────┘              │
│  Communication via interfaces, pas directs │
└────────────────────────────────────────────┘
         ↓ quand le besoin se présente
     Extraire en microservice
```

---

## Domain-Driven Design (DDD) — Bases

Le DDD est une approche de conception qui centre l'architecture sur le **domaine métier** (les règles business) plutôt que sur la technique.

### Bounded Context (Contexte Borné)

Un bounded context est la frontière dans laquelle un modèle est cohérent et valide.

```
Exemple e-commerce :

Contexte "Catalogue" :          Contexte "Commande" :
┌──────────────────┐           ┌──────────────────┐
│  Produit         │           │  ProduitCommandé  │
│  - id            │           │  - idProduit      │
│  - nom           │           │  - quantite       │
│  - description   │           │  - prixAuMoment   │
│  - stock         │           │  (snapshot !)     │
└──────────────────┘           └──────────────────┘

Le "Produit" n'a pas le même sens dans chaque contexte !
Dans Commande, on stocke le prix AU MOMENT de la commande
(le prix catalogue peut changer, la commande doit rester stable)
```

### Ubiquitous Language (Langage Omniprésent)

Utiliser **les mêmes termes métier** dans le code, les discussions, la documentation et avec les experts métier. Pas de traduction entre le vocabulaire business et le vocabulaire technique.

```python
# ❌ Mauvais — vocabulaire technique, pas métier
class OrderRecord:
    def update_status_flag(self, flag: int): ...

# ✅ Bon — vocabulaire du domaine
class Commande:
    def confirmer(self): ...
    def expedier(self): ...
    def annuler(self, raison: str): ...
```

### Entités vs Value Objects

```python
# ENTITÉ : identité unique, mutable, suit son cycle de vie
class Utilisateur:
    def __init__(self, id: str, email: str):
        self.id = id  # identité stable
        self.email = email
    
    # Deux utilisateurs sont identiques si même ID
    def __eq__(self, other):
        return isinstance(other, Utilisateur) and self.id == other.id

# VALUE OBJECT : défini par ses attributs, immuable, pas d'identité
@dataclass(frozen=True)  # immuable en Python
class Adresse:
    rue: str
    ville: str
    code_postal: str
    pays: str
    # Deux adresses sont identiques si mêmes attributs
    # On ne "change" pas une adresse, on en crée une nouvelle
```

### Aggregate

Un aggregate est un cluster d'entités et value objects traité comme une unité. Il a une **racine** (Aggregate Root) qui est le seul point d'entrée.

```python
# Aggregate Root : Commande
class Commande:  # Aggregate Root
    def __init__(self, id: str, client_id: str):
        self._id = id
        self._client_id = client_id
        self._lignes: list[LigneCommande] = []
        self._statut = StatutCommande.EN_ATTENTE
    
    # Toutes les modifications passent par la racine
    def ajouter_produit(self, produit_id: str, quantite: int, prix: float):
        if self._statut != StatutCommande.EN_ATTENTE:
            raise ValueError("Impossible d'ajouter à une commande confirmée")
        self._lignes.append(LigneCommande(produit_id, quantite, prix))
    
    @property
    def total(self) -> float:
        return sum(ligne.sous_total for ligne in self._lignes)

class LigneCommande:  # Entité interne, pas accessible directement
    def __init__(self, produit_id: str, quantite: int, prix_unitaire: float):
        self.produit_id = produit_id
        self.quantite = quantite
        self.prix_unitaire = prix_unitaire
    
    @property
    def sous_total(self) -> float:
        return self.quantite * self.prix_unitaire
```

---

## Communication Entre Services

### REST Synchrone

```
Service A ──────HTTP POST────────> Service B
          <─────200 OK / réponse──
```

- **Quand** : opérations qui nécessitent une réponse immédiate (création d'un utilisateur, vérification de stock)
- **Problème** : couplage temporel — si B est down, A échoue

### gRPC

Protocol Buffers + HTTP/2 — plus rapide que REST pour la communication inter-services.

```protobuf
// commande.proto
syntax = "proto3";

service CommandeService {
  rpc CreerCommande (CreerCommandeRequest) returns (Commande);
  rpc GetCommande (GetCommandeRequest) returns (Commande);
  rpc StreamStatut (GetCommandeRequest) returns (stream StatutUpdate);
}

message Commande {
  string id = 1;
  string client_id = 2;
  float total = 3;
  string statut = 4;
}
```

| Critère | REST | gRPC |
|---|---|---|
| **Format** | JSON (texte) | Protobuf (binaire) |
| **Performance** | Modérée | Élevée (3-10× plus rapide) |
| **Browser support** | Natif | Requiert un proxy (grpc-web) |
| **Streaming** | Limité | Natif (bidirectionnel) |
| **Tooling** | Très mûr | Mature mais plus complexe |
| **Usage idéal** | APIs publiques, browser | Communication inter-services |

### Message Queue (Asynchrone)

```
Service A ──────publish(event)──> Message Broker (Kafka/RabbitMQ)
                                        │
                              ┌─────────┼──────────┐
                              ↓         ↓          ↓
                         Service B  Service C  Service D
                         (chacun consomme à son rythme)
```

**Quand utiliser l'asynchrone :**
- Opérations qui n'ont pas besoin d'une réponse immédiate (email de confirmation, mise à jour de stock)
- Événements qui intéressent plusieurs services (commande créée → 3 services réagissent)
- Découplage : Service A ne sait même pas que B, C, D existent

```python
# Exemple : publier un événement (avec Kafka + confluent-kafka)
from confluent_kafka import Producer
import json

producer = Producer({'bootstrap.servers': 'kafka:9092'})

def commande_creee(commande_id: str, total: float):
    evenement = {
        'type': 'COMMANDE_CREEE',
        'commande_id': commande_id,
        'total': total,
        'timestamp': '2025-05-28T10:00:00Z'
    }
    producer.produce(
        topic='commandes',
        key=commande_id,
        value=json.dumps(evenement),
    )
    producer.flush()

# Service B (Notifications) consomme cet événement
# Service C (Inventaire) consomme cet événement
# Service D (Analytics) consomme cet événement
```

---

## Patterns de Résilience

### Circuit Breaker

```
État FERMÉ (normal) :
  Requêtes passent → Service OK → Circuit reste fermé

État OUVERT (panne) :
  Trop d'échecs détectés → Circuit s'ouvre → Requêtes bloquées immédiatement
  (au lieu d'attendre le timeout à chaque fois)

État DEMI-OUVERT (récupération) :
  Après un délai → Laisser passer une requête test
  Succès → Revenir à FERMÉ
  Échec → Retourner à OUVERT

Diagramme :
FERMÉ ──(trop d'échecs)──> OUVERT
  ^                            │
  │                        (délai timeout)
  │                            ↓
  └──(test réussi)──────> DEMI-OUVERT
```

```python
# Implémentation simple de Circuit Breaker
from datetime import datetime, timedelta
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(self, seuil_echecs=5, timeout_secondes=30):
        self.seuil_echecs = seuil_echecs
        self.timeout = timeout_secondes
        self.echecs = 0
        self.etat = CircuitState.CLOSED
        self.derniere_ouverture = None
    
    def appeler(self, fonction, *args, **kwargs):
        if self.etat == CircuitState.OPEN:
            if datetime.now() - self.derniere_ouverture > timedelta(seconds=self.timeout):
                self.etat = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit ouvert - service indisponible")
        
        try:
            resultat = fonction(*args, **kwargs)
            self._succes()
            return resultat
        except Exception as e:
            self._echec()
            raise
    
    def _succes(self):
        self.echecs = 0
        self.etat = CircuitState.CLOSED
    
    def _echec(self):
        self.echecs += 1
        if self.echecs >= self.seuil_echecs:
            self.etat = CircuitState.OPEN
            self.derniere_ouverture = datetime.now()
```

### Retry avec Backoff Exponentiel

```python
import time
import random

def retry_avec_backoff(fonction, max_tentatives=3, base_delai=1.0):
    for tentative in range(max_tentatives):
        try:
            return fonction()
        except Exception as e:
            if tentative == max_tentatives - 1:
                raise  # Dernière tentative → propager l'erreur
            
            # Backoff exponentiel avec jitter (bruit aléatoire)
            # Évite que tous les clients réessaient en même temps
            delai = base_delai * (2 ** tentative) + random.uniform(0, 1)
            print(f"Tentative {tentative + 1} échouée. Retry dans {delai:.1f}s")
            time.sleep(delai)

# Tentative 1 : délai 1.0-2.0s
# Tentative 2 : délai 2.0-3.0s
# Tentative 3 : délai 4.0-5.0s
```

### Timeout

```python
import asyncio

async def appel_service_avec_timeout(url: str, timeout_s: float = 5.0):
    try:
        async with asyncio.timeout(timeout_s):
            # Faire l'appel HTTP
            response = await http_client.get(url)
            return response
    except asyncio.TimeoutError:
        # Fallback : retourner des données en cache ou une réponse par défaut
        return obtenir_cache_fallback(url)
```

### Bulkhead (Cloisons Étanches)

```
Sans bulkhead :
  Tous les services partagent le même pool de threads
  → Service lent épuise tous les threads
  → Tous les autres services sont impactés

Avec bulkhead :
  Service A → Pool dédié (10 threads max)
  Service B → Pool dédié (10 threads max)
  Service C → Pool dédié (5 threads max)
  
  Si Service A est lent → seul son pool est saturé
  Service B et C continuent de fonctionner
```

---

## API Gateway Pattern

L'API Gateway est le **point d'entrée unique** pour tous les clients. Il évite que les clients doivent connaître l'adresse de chaque microservice.

```
Client Mobile ──┐
Client Web   ──>│  API Gateway        Service Auth
Client IoT  ──┘ │  - Authentification  ← vérifie les tokens
                 │  - Rate limiting     ← 100 req/min par IP
                 │  - Routing           ← /users → User Service
                 │  - Load balancing    ← round-robin
                 │  - SSL termination   ← HTTPS → HTTP interne
                 │  - Request agregation← combine plusieurs appels
                 ↓
         ┌───────┴───────┐
     User Service   Product Service
         └───────────────┘
```

**Solutions populaires :** Kong, AWS API Gateway, Nginx (avec config), Traefik, Envoy.

---

## Saga Pattern — Transactions Distribuées

Dans un monolithe, on utilise une transaction de base de données. Dans les microservices, c'est impossible (chaque service a sa propre DB). La Saga coordonne une séquence d'opérations locales.

### Orchestration (un chef d'orchestre central)

```
Saga Orchestrateur (ex: création de commande)

Orchestrateur
    │
    ├──> Service Commande : créer commande (état: PENDING)
    │         ← OK
    ├──> Service Paiement : débiter client
    │         ← OK
    ├──> Service Inventaire : réserver stock
    │         ← ECHEC (stock insuffisant)
    │
    │    → COMPENSATION :
    ├──> Service Paiement : rembourser client     ← annuler
    └──> Service Commande : annuler commande      ← annuler
```

### Chorégraphie (chaque service réagit aux événements)

```
Commande créée (event) ──────────────────────────────>
                        │                             │
               Service Paiement               Service Inventaire
               Paiement OK (event)            Réserve stock (event)
                        │
               Service Livraison
               Livraison planifiée (event)
                        ...
```

| Aspect | Orchestration | Chorégraphie |
|---|---|---|
| **Complexité** | Logique centralisée | Logique distribuée |
| **Debugging** | Facile (un seul endroit) | Difficile (tracer les events) |
| **Couplage** | Orchestrateur connaît tout | Services découplés |
| **Risque** | SPOF si orchestrateur down | Pas de SPOF |
| **Quand** | Workflows complexes avec conditions | Événements simples, nombreux consumers |

---

## CQRS — Command Query Responsibility Segregation

Séparer les opérations de **lecture** et d'**écriture** dans des modèles distincts.

```
SANS CQRS :
  Même modèle pour lire et écrire → compromis → sous-optimal pour les deux

AVEC CQRS :
  ┌─────────────────┐        ┌─────────────────┐
  │   WRITE SIDE    │        │    READ SIDE     │
  │                 │        │                  │
  │  Commands       │ event  │  Queries         │
  │  (create,       │──────> │  (optimisé pour  │
  │   update,       │        │   la lecture,    │
  │   delete)       │        │   dénormalisé)   │
  │                 │        │                  │
  │  Write DB       │        │  Read DB         │
  │  (Postgres)     │        │  (Elasticsearch, │
  └─────────────────┘        │   Redis, etc.)   │
                              └─────────────────┘
```

**Bénéfices :**
- La base de lecture peut être optimisée pour les requêtes (indexation, dénormalisation)
- Scalabilité indépendante : les lectures sont souvent 90 % du trafic
- Possibilité de plusieurs vues de lecture différentes

---

## Event Sourcing

Au lieu de stocker l'état actuel, on stocke la **séquence d'événements** qui a mené à cet état.

```python
# APPROCHE CLASSIQUE (stocker l'état)
# Table comptes: {id: 1, solde: 150.00}
# ← on ne sait pas COMMENT on est arrivé à 150€

# EVENT SOURCING (stocker les événements)
events = [
    {'type': 'COMPTE_CREE', 'solde_initial': 0, 'timestamp': '...'},
    {'type': 'DEPOT', 'montant': 200, 'timestamp': '...'},
    {'type': 'RETRAIT', 'montant': 50, 'timestamp': '...'},
    # Rejouer → solde = 0 + 200 - 50 = 150€
]
```

**Avantages :** audit complet, capacité de "rembobiner" l'état, debug facilité, base du CQRS.

**Inconvénients :** complexité, les snapshots sont nécessaires pour les longues séquences.

---

## Service Mesh (Istio)

Un service mesh est une couche d'infrastructure qui gère la communication entre services, **sans modifier le code applicatif**.

```
Avec Istio (sidecar pattern) :
┌─────────────────────────────────────┐
│  Pod Service A                       │
│  ┌──────────────┐  ┌─────────────┐  │
│  │   Service A  │  │   Envoy     │  │
│  │   (app code) │<─│  (sidecar)  │  │
│  └──────────────┘  └──────┬──────┘  │
└─────────────────────────────│────────┘
                              │ mTLS automatique
                              │ retry, circuit breaker
                              │ tracing, metrics
┌─────────────────────────────│────────┐
│  Pod Service B              │        │
│  ┌──────────────┐  ┌────────┴─────┐  │
│  │   Service B  │  │   Envoy      │  │
│  │   (app code) │<─│  (sidecar)   │  │
│  └──────────────┘  └─────────────┘  │
└─────────────────────────────────────┘
```

Istio fournit automatiquement : mTLS entre services, circuit breaking, retries, timeouts, load balancing, observabilité (métriques, traces, logs). Le code applicatif ne sait pas qu'Istio existe.

---

## Observabilité des Microservices

Avec 20+ services, il faut des outils pour comprendre ce qui se passe. Les 3 piliers :

```
┌────────────────┬────────────────────┬───────────────────────────┐
│  LOGS          │  MÉTRIQUES         │  TRACES                   │
│                │                    │                           │
│  "Que s'est-il │  "Comment le       │  "Quel chemin a suivi     │
│  passé ?"      │  système se        │  cette requête entre      │
│                │  porte-t-il ?"     │  les services ?"          │
│                │                    │                           │
│  ELK Stack     │  Prometheus        │  Jaeger                   │
│  Loki          │  Grafana           │  Zipkin                   │
│  CloudWatch    │  DataDog           │  Tempo                    │
└────────────────┴────────────────────┴───────────────────────────┘
```

Voir [[03 - Tracing et Debugging Distribue]] pour le détail du distributed tracing.

---

## Décomposition des Services

### Par Capacité Métier

```
E-commerce décomposé par capacité métier :

  Gestion des utilisateurs  → Service Identité
  Catalogue produits        → Service Catalogue
  Gestion du panier         → Service Panier
  Traitement des commandes  → Service Commandes
  Paiement                  → Service Paiement
  Logistique                → Service Expédition
  Notifications             → Service Notifications
  Recherche                 → Service Recherche
```

### Règles de Décomposition

| Règle | Description |
|---|---|
| **Single Responsibility** | Un service = une capacité métier unique |
| **Loose Coupling** | Les services communiquent via APIs, pas de partage de DB |
| **High Cohesion** | La logique qui change ensemble reste ensemble |
| **Bounded Context** | Chaque service a son propre modèle de données |
| **Independent Deployability** | Déployer un service sans redéployer les autres |

> [!warning] Anti-patterns courants
> - **Service trop petit** (nano-services) : un service par table de base de données → overhead colossal
> - **DB partagée** : deux services qui lisent/écrivent la même table → couplage fort déguisé
> - **Chattiness** : service A appelle B 10 fois par requête → latence accumulée
> - **Monolithe distribué** : tous les services déploient ensemble → worst of both worlds

---

## Déploiement Microservices

Les microservices s'orchestrent avec Kubernetes — voir [[04 - Kubernetes Introduction]] pour le détail.

En résumé :
- Chaque service = un ou plusieurs Pods Docker
- Service Kubernetes pour la découverte de service et le load balancing
- ConfigMap et Secrets pour la configuration
- HPA (Horizontal Pod Autoscaler) pour le scaling automatique
- Ingress ou API Gateway pour le routage externe

---

## Exercices Pratiques

### Exercice 1 — Décomposer un monolithe (45 min)
Prenez un système de réservation d'hôtel (gérant : hôtels, chambres, réservations, clients, paiements, emails, reviews). Dessinez :
1. Les bounded contexts
2. Les services que vous extrairiez et dans quel ordre
3. Les événements échangés entre services (au minimum 5)
4. Quelle base de données serait adaptée à chaque service

### Exercice 2 — Implémenter un Circuit Breaker (1h)
Implémentez en Python ou JavaScript un circuit breaker complet avec :
- Les 3 états (CLOSED, OPEN, HALF_OPEN)
- Configuration du seuil d'échecs et du timeout
- Logging de chaque transition d'état
- Tests unitaires pour vérifier chaque état
Testez en simulant une API qui retourne des erreurs 50 % du temps.

### Exercice 3 — Saga Choreography (1h30)
Simulez en Python une saga de création de commande avec chorégraphie :
1. `CommandeService` : crée la commande, publie `COMMANDE_CREEE`
2. `PaiementService` : écoute `COMMANDE_CREEE`, débite, publie `PAIEMENT_OK` ou `PAIEMENT_ECHOUE`
3. `StockService` : écoute `PAIEMENT_OK`, réserve, publie `STOCK_RESERVE`
4. `CommandeService` : écoute `STOCK_RESERVE`, confirme la commande
5. Gérer le cas où le paiement échoue (compensation)

Utiliser des dictionnaires Python pour simuler le message broker.

### Exercice 4 — API Gateway simple (1h)
Créez un API Gateway en Python (Flask ou FastAPI) qui :
1. Route `/users/*` vers `http://localhost:3001`
2. Route `/products/*` vers `http://localhost:3002`
3. Vérifie un token Bearer dans l'en-tête Authorization
4. Ajoute un header `X-Request-ID` unique à chaque requête
5. Limite à 10 requêtes par minute par IP (avec un simple dict en mémoire)

> [!info] Aller plus loin
> - **Livre** : "Building Microservices" de Sam Newman — la référence
> - **Livre** : "Designing Distributed Systems" de Brendan Burns (gratuit en ligne)
> - **Blog** : martinfowler.com — articles fondamentaux sur les microservices et DDD
> - **Cours** : "Microservices with Node.js and React" sur Udemy (Stephen Grider)
> - **Hands-on** : Déployer une app multi-services sur Kubernetes avec [[04 - Kubernetes Introduction]]
