# Redis et Caches

**Redis** (Remote Dictionary Server) est une base de donnees **in-memory** ultra-rapide, open-source, ecrite en C. Elle stocke toutes les donnees en RAM, ce qui lui permet d'atteindre des performances que les bases sur disque ne peuvent pas egaler : **100 000+ operations par seconde** avec une latence inferieure a la milliseconde.

Redis n'est pas qu'un simple cache : c'est un outil multi-usage que vous trouverez dans presque toutes les architectures backend modernes.

> [!tip] Analogie
> Imaginez votre base de donnees comme une grande bibliotheque (rangement parfait, mais aller chercher un livre prend du temps). Redis est le bureau juste devant vous : les documents les plus utilises sont la, a portee de main, accessibles en une seconde. Quand vous avez fini de les utiliser, ils retournent a la bibliotheque — mais pendant que vous travaillez, vous avez la vitesse du bureau.

---

## Pourquoi le Cache ?

Une requete SQL sur une base de donnees peut prendre 10 a 500ms. Multipliee par 1000 utilisateurs simultanees, c'est votre serveur qui sature.

```
SANS CACHE :
Utilisateur → API → Base SQL → 150ms → Reponse

AVEC CACHE :
Utilisateur → API → Redis (cache HIT) → 0.5ms → Reponse
                         ↓ cache MISS
                    Base SQL → 150ms → Redis (mise en cache) → Reponse
```

> [!info] Impact concret du cache
> | Scenario | Sans cache | Avec cache |
> |---|---|---|
> | 1 requete SQL lourde | 200ms | 0.5ms (apres 1er appel) |
> | 1000 utilisateurs simultanes | Serveur sature | Charge SQL reduite de 90%+ |
> | Pic de trafic (soldes, lancement) | Crash possible | Redis absorbe le pic |
> | Cout infrastructure | Gros serveur SQL | SQL modeste + Redis |

---

## Installation et Connexion

```bash
# Docker (recommande pour le cours)
docker run -d --name redis -p 6379:6379 redis:7.2

# Ubuntu/Debian
sudo apt-get install redis-server
sudo systemctl start redis

# Tester la connexion
redis-cli ping
# → PONG
```

```bash
# Interface en ligne de commande
redis-cli

# Avec authentification
redis-cli -h localhost -p 6379 -a monmotdepasse

# Voir toutes les cles (attention : ne jamais en prod avec des millions de cles !)
KEYS *

# Stats du serveur
INFO server
INFO memory
```

---

## Structures de Donnees Redis

Redis n'est pas juste un store cle-string. Il supporte **6 structures de donnees** natives.

### 1. String

La structure la plus basique. Peut stocker du texte, des nombres, ou des donnees binaires (jusqu'a 512 MB).

```bash
# Definir et lire
SET nom "Alice Martin"
GET nom
# → "Alice Martin"

# Definir avec expiration (TTL en secondes)
SET session:user42 "token_abc123" EX 3600
TTL session:user42    # Secondes restantes
# → 3598

# Incrementer/decrementer un compteur atomique
SET compteur_vues 0
INCR compteur_vues    # → 1
INCR compteur_vues    # → 2
INCRBY compteur_vues 10  # → 12
DECR compteur_vues    # → 11

# Definir seulement si la cle n'existe pas (mutex/lock)
SET lock:ressource "process_42" NX EX 30
# NX = "Not eXists", EX = expiration

# Obtenir la valeur et definir en meme temps
GETSET old_value "nouvelle_valeur"

# Definir plusieurs en une fois
MSET prenom "Alice" nom "Martin" age "23"
MGET prenom nom age
```

### 2. Hash

Un objet avec des champs, comme un document JSON plat. Ideal pour les profils utilisateurs.

```bash
# Definir des champs
HSET user:42 nom "Alice Martin" email "alice@example.com" age 23 formation "FullStack"

# Lire un champ
HGET user:42 nom
# → "Alice Martin"

# Lire tous les champs
HGETALL user:42
# → nom, Alice Martin, email, alice@example.com, age, 23, formation, FullStack

# Incrementer un champ numerique
HINCRBY user:42 age 1  # → 24

# Verifier si un champ existe
HEXISTS user:42 email  # → 1 (true)

# Supprimer un champ
HDEL user:42 formation

# Nombre de champs
HLEN user:42
```

### 3. List

Une liste doublement chainee. Optimisee pour les operations en tete/queue (O(1)).

```bash
# Ajouter a gauche (en tete) ou a droite (en queue)
LPUSH taches "Faire les tests"
LPUSH taches "Ecrire les specs"
RPUSH taches "Deployer en prod"

# Lire une plage (0 = debut, -1 = fin)
LRANGE taches 0 -1
# → ["Ecrire les specs", "Faire les tests", "Deployer en prod"]

# Extraire et supprimer (depuis la gauche = queue FIFO)
LPOP taches       # → "Ecrire les specs"
RPOP taches       # → "Deployer en prod"

# Extraire avec attente (utile pour les queues de taches)
BLPOP taches 30   # Attendre jusqu'a 30s qu'un element soit disponible

# Longueur
LLEN taches

# Trim : garder uniquement les N derniers elements
LTRIM logs:app 0 999  # Garder les 1000 derniers logs seulement
```

### 4. Set

Collection de valeurs **uniques**, non ordonnees. Supporte les operations ensemblistes.

```bash
# Ajouter des membres
SADD tags:article:42 "python" "programmation" "backend" "web"
SADD tags:article:43 "javascript" "web" "frontend" "react"

# Verifier l'appartenance
SISMEMBER tags:article:42 "python"  # → 1 (true)
SISMEMBER tags:article:42 "java"    # → 0 (false)

# Tous les membres
SMEMBERS tags:article:42

# Intersection (tags en commun)
SINTER tags:article:42 tags:article:43
# → {"web"}

# Union
SUNION tags:article:42 tags:article:43
# → {"python", "programmation", "backend", "web", "javascript", "frontend", "react"}

# Difference
SDIFF tags:article:42 tags:article:43
# → {"python", "programmation", "backend"}

# Taille
SCARD tags:article:42
```

### 5. Sorted Set (ZSet)

Comme un Set, mais chaque membre a un **score** numerique. Automatiquement trie par score.

```bash
# Ajouter avec un score
ZADD classement 95.5 "Alice"
ZADD classement 87.3 "Bob"
ZADD classement 91.0 "Claire"
ZADD classement 78.8 "David"

# Lire par range de rang (0 = meilleur score)
ZRANGE classement 0 -1 WITHSCORES  # Du plus bas au plus haut
ZREVRANGE classement 0 2 WITHSCORES  # Top 3

# Rang d'un membre
ZRANK classement "Alice"     # Position (0-indexed, ordre croissant)
ZREVRANK classement "Alice"  # Position ordre decroissant (0 = meilleur)

# Score d'un membre
ZSCORE classement "Bob"  # → "87.3"

# Incrementer un score
ZINCRBY classement 5 "David"  # David gagne 5 points

# Membres dans un range de scores
ZRANGEBYSCORE classement 85 100 WITHSCORES  # Score entre 85 et 100
```

### 6. Stream

Structure de donnees de type **journal append-only**, similaire a Kafka (voir la section sur les Streams plus bas).

---

## TTL et Expiration

Redis permet de definir une **duree de vie** sur n'importe quelle cle.

```bash
# Definir un TTL (expiration en secondes)
SET ma_cle "valeur" EX 3600         # Expire dans 1 heure
SET ma_cle "valeur" PX 3600000      # En millisecondes
SET ma_cle "valeur" EXAT 1700000000 # Timestamp Unix precis

# Verifier le TTL restant
TTL ma_cle    # Secondes restantes (-1 = pas de TTL, -2 = cle inexistante)
PTTL ma_cle   # En millisecondes

# Supprimer le TTL (rendre permanente)
PERSIST ma_cle

# Definir un TTL sur une cle existante
EXPIRE ma_cle 7200      # Secondes
EXPIREAT ma_cle 1700000000  # Timestamp Unix
```

---

## Patterns de Cache

### Cache-Aside (Lazy Loading) — le plus courant

```python
import redis
import json
from typing import Any

r = redis.Redis(host="localhost", port=6379, decode_responses=True)


def get_utilisateur(user_id: int) -> dict:
    """
    Pattern Cache-Aside :
    1. Chercher dans Redis (rapide)
    2. Si absent (cache miss), aller en base de donnees
    3. Stocker le resultat dans Redis
    4. Retourner le resultat
    """
    cache_key = f"user:{user_id}"
    
    # Etape 1 : Verifier le cache
    cached = r.get(cache_key)
    if cached:
        print(f"Cache HIT pour user:{user_id}")
        return json.loads(cached)
    
    # Etape 2 : Cache MISS → aller en base de donnees
    print(f"Cache MISS pour user:{user_id} — requete SQL...")
    # Simuler une requete SQL lente
    import time
    time.sleep(0.15)  # 150ms simulee
    user = {
        "id": user_id,
        "nom": "Alice Martin",
        "email": "alice@example.com",
        "formation": "FullStack"
    }
    
    # Etape 3 : Stocker dans le cache (TTL : 1 heure)
    r.setex(cache_key, 3600, json.dumps(user))
    
    return user


def invalider_cache_utilisateur(user_id: int) -> None:
    """Invalider le cache quand les donnees changent."""
    r.delete(f"user:{user_id}")
    print(f"Cache invalide pour user:{user_id}")
```

### Write-Through — ecriture simultanee cache + DB

```python
def mettre_a_jour_utilisateur(user_id: int, donnees: dict) -> None:
    """
    Write-Through : mettre a jour SIMULTANEMENT la DB et le cache.
    Garantit que le cache est toujours a jour.
    Inconvenient : ecriture plus lente (2 operations).
    """
    # 1. Mettre a jour la base de donnees (SQL, MongoDB, etc.)
    # db.execute("UPDATE users SET ... WHERE id = ?", (user_id,))
    print(f"Mise a jour SQL pour user:{user_id}")
    
    # 2. Mettre a jour le cache immediatement
    cache_key = f"user:{user_id}"
    r.setex(cache_key, 3600, json.dumps({"id": user_id, **donnees}))
    print(f"Cache mis a jour pour user:{user_id}")
```

### Write-Behind (Write-Back) — ecriture differee

```python
import threading
from collections import defaultdict

class WriteBehindCache:
    """
    Write-Behind : ecrire d'abord dans le cache, synchro DB en arriere-plan.
    Avantage : ecritures ultra-rapides.
    Risque : perte de donnees si crash avant la synchro.
    Cas d'usage : compteurs, metriques, analytics.
    """
    
    def __init__(self, redis_client, sync_interval: int = 5):
        self.r = redis_client
        self.dirty_keys = set()  # Cles a synchroniser
        self.sync_interval = sync_interval
        self._start_background_sync()
    
    def write(self, key: str, value: Any) -> None:
        """Ecriture instantanee dans Redis."""
        self.r.set(key, json.dumps(value))
        self.dirty_keys.add(key)
    
    def _sync_to_database(self) -> None:
        """Synchronisation periodique vers la base de donnees."""
        while True:
            import time
            time.sleep(self.sync_interval)
            if self.dirty_keys:
                print(f"Synchro DB : {len(self.dirty_keys)} cles a persister")
                for key in list(self.dirty_keys):
                    value = self.r.get(key)
                    if value:
                        # db.save(key, json.loads(value))
                        print(f"  Persiste {key}")
                self.dirty_keys.clear()
    
    def _start_background_sync(self) -> None:
        t = threading.Thread(target=self._sync_to_database, daemon=True)
        t.start()
```

---

## Sessions Utilisateur

Redis est ideal pour gerer les sessions : rapide, avec TTL automatique, partage entre plusieurs instances d'API.

```python
import redis
import json
import secrets
from datetime import datetime

r = redis.Redis(host="localhost", port=6379, decode_responses=True)

SESSION_TTL = 24 * 3600  # 24 heures


def creer_session(user_id: int, user_data: dict) -> str:
    """Cree une session et retourne le token."""
    token = secrets.token_urlsafe(32)  # Token cryptographiquement sur
    session_key = f"session:{token}"
    
    session_data = {
        "user_id": user_id,
        "created_at": datetime.utcnow().isoformat(),
        **user_data
    }
    
    # Stocker avec TTL
    r.setex(session_key, SESSION_TTL, json.dumps(session_data))
    return token


def recuperer_session(token: str) -> dict | None:
    """Recupere et renouvelle une session."""
    session_key = f"session:{token}"
    data = r.get(session_key)
    
    if not data:
        return None  # Session expiree ou invalide
    
    # Renouveler le TTL (sliding expiration)
    r.expire(session_key, SESSION_TTL)
    return json.loads(data)


def detruire_session(token: str) -> None:
    """Deconnexion : supprimer la session."""
    r.delete(f"session:{token}")


# Exemple d'utilisation
token = creer_session(42, {"nom": "Alice", "role": "etudiant"})
print(f"Token de session : {token}")

session = recuperer_session(token)
print(f"Session recuperee : {session}")

detruire_session(token)
print(f"Session apres deconnexion : {recuperer_session(token)}")  # None
```

---

## Rate Limiting avec Redis

```python
def rate_limit(r: redis.Redis, user_id: str, max_requetes: int = 100, fenetre: int = 3600) -> dict:
    """
    Rate limiting avec fenetre glissante.
    
    Args:
        user_id: Identifiant de l'utilisateur ou de l'IP
        max_requetes: Nombre maximum de requetes dans la fenetre
        fenetre: Duree de la fenetre en secondes
    
    Returns:
        Dict avec 'autorise', 'restantes', 'reset_dans'
    """
    cle = f"rate_limit:{user_id}"
    
    # Pipeline atomique : incrementer ET verifier en une seule transaction
    pipe = r.pipeline()
    pipe.incr(cle)
    pipe.ttl(cle)
    count, ttl = pipe.execute()
    
    # Definir le TTL si c'est la premiere requete
    if ttl == -1:
        r.expire(cle, fenetre)
        ttl = fenetre
    
    autorise = count <= max_requetes
    
    return {
        "autorise": autorise,
        "requetes_effectuees": count,
        "requetes_restantes": max(0, max_requetes - count),
        "reset_dans_secondes": ttl
    }


# Utilisation dans une API (Flask/FastAPI)
def middleware_rate_limit(user_id: str):
    resultat = rate_limit(r, user_id, max_requetes=100, fenetre=3600)
    
    if not resultat["autorise"]:
        raise Exception(f"429 Too Many Requests — Reset dans {resultat['reset_dans_secondes']}s")
    
    print(f"Requetes restantes : {resultat['requetes_restantes']}")
    return resultat
```

---

## Redis Pub/Sub — Message Broker

Redis peut agir comme un **broker de messages** simple pour la communication entre services.

```python
import redis
import threading
import time

r = redis.Redis(host="localhost", port=6379, decode_responses=True)


def publisher(canal: str, messages: list) -> None:
    """Publie des messages sur un canal."""
    for message in messages:
        nb_subscribers = r.publish(canal, json.dumps(message))
        print(f"Message publie sur '{canal}' ({nb_subscribers} abonnes) : {message}")
        time.sleep(1)


def subscriber(canaux: list) -> None:
    """S'abonne a des canaux et traite les messages."""
    pubsub = r.pubsub()
    pubsub.subscribe(*canaux)
    
    print(f"Abonne aux canaux : {canaux}")
    
    for message in pubsub.listen():
        if message["type"] == "message":
            data = json.loads(message["data"])
            canal = message["channel"]
            print(f"[{canal}] Recu : {data}")
            # Traiter le message ici...


# Exemple : notifications en temps reel
def lancer_exemple_pubsub():
    # Subscriber dans un thread separe
    t_sub = threading.Thread(
        target=subscriber,
        args=(["notifications", "alertes"],),
        daemon=True
    )
    t_sub.start()
    time.sleep(0.5)  # Laisser le subscriber demarrer
    
    # Publier des messages
    messages = [
        {"type": "info", "texte": "Nouveau cours disponible"},
        {"type": "alerte", "texte": "Serveur en maintenance dans 10min"},
        {"type": "info", "texte": "Votre note est disponible"}
    ]
    publisher("notifications", messages)
```

---

## Redis Streams

Les Streams sont une structure de donnees plus avancee que Pub/Sub : les messages sont **persistes** et peuvent etre consommes par plusieurs groupes de consommateurs.

```python
import redis
import json

r = redis.Redis(host="localhost", port=6379, decode_responses=True)

STREAM_NAME = "evenements_utilisateurs"


def publier_evenement(event_type: str, data: dict) -> str:
    """Publie un evenement dans le stream. Retourne l'ID de l'evenement."""
    message = {"type": event_type, **data}
    # '*' = ID genere automatiquement par Redis (timestamp + sequence)
    event_id = r.xadd(STREAM_NAME, {"payload": json.dumps(message)})
    return event_id


def creer_groupe_consommateurs(groupe: str) -> None:
    """Cree un groupe de consommateurs (pour la lecture garantie)."""
    try:
        r.xgroup_create(STREAM_NAME, groupe, id="0", mkstream=True)
        print(f"Groupe '{groupe}' cree.")
    except redis.exceptions.ResponseError as e:
        if "BUSYGROUP" in str(e):
            print(f"Groupe '{groupe}' existe deja.")


def consommer_evenements(groupe: str, consommateur: str, count: int = 10) -> list:
    """Lit les prochains messages non traites pour ce groupe."""
    messages = r.xreadgroup(
        groupname=groupe,
        consumername=consommateur,
        streams={STREAM_NAME: ">"},  # ">" = messages non encore delivres
        count=count,
        block=5000  # Attendre 5 secondes si vide
    )
    
    evenements = []
    if messages:
        for stream, entries in messages:
            for entry_id, fields in entries:
                data = json.loads(fields["payload"])
                evenements.append({"id": entry_id, "data": data})
                # Acknowledger le message (marque comme traite)
                r.xack(STREAM_NAME, groupe, entry_id)
    
    return evenements


# Exemple d'utilisation
creer_groupe_consommateurs("analytics")
creer_groupe_consommateurs("notifications")

# Publier
publier_evenement("page_vue", {"user_id": 42, "page": "/cours/python"})
publier_evenement("inscription", {"user_id": 43, "formation": "FullStack"})
publier_evenement("note_ajoutee", {"user_id": 42, "note": 18, "cours": "algorithmique"})

# Consommer depuis le groupe analytics
events = consommer_evenements("analytics", "worker_1")
for event in events:
    print(f"Analytics traite : {event['data']}")
```

---

## Utilisation depuis Python (redis-py)

### Connexion et configuration avancee

```python
import redis
from redis.retry import Retry
from redis.backoff import ExponentialBackoff

# Connection pool (a utiliser dans les applications de production)
pool = redis.ConnectionPool(
    host="localhost",
    port=6379,
    db=0,
    max_connections=20,
    decode_responses=True  # Decoder automatiquement en string Python
)

r = redis.Redis(connection_pool=pool)

# Configuration avec retry automatique
retry = Retry(ExponentialBackoff(), 3)
r_with_retry = redis.Redis(
    host="localhost",
    retry=retry,
    retry_on_error=[ConnectionError, TimeoutError]
)

# Test de connexion
def ping_redis(client: redis.Redis) -> bool:
    try:
        return client.ping()
    except redis.ConnectionError:
        return False

print("Redis connecte :", ping_redis(r))
```

### Pipeline Redis — operations atomiques groupees

```python
def incrementer_compteurs_batch(r: redis.Redis, user_ids: list) -> None:
    """
    Pipeline : envoyer plusieurs commandes en un seul aller-retour reseau.
    Bien plus efficace que N appels individuels.
    """
    with r.pipeline() as pipe:
        for user_id in user_ids:
            pipe.incr(f"vues:user:{user_id}")
            pipe.expire(f"vues:user:{user_id}", 86400)
        # Executer toutes les commandes en une seule fois
        resultats = pipe.execute()
    return resultats


def transaction_atomique(r: redis.Redis, cle: str, nouveau_min: int) -> bool:
    """
    WATCH + pipeline pour une transaction optimiste.
    Si la cle change entre WATCH et EXECUTE, la transaction est annulee.
    """
    with r.pipeline() as pipe:
        while True:
            try:
                pipe.watch(cle)  # Observer la cle
                valeur_actuelle = int(pipe.get(cle) or 0)
                
                if valeur_actuelle <= nouveau_min:
                    pipe.multi()  # Debut de la transaction
                    pipe.set(cle, nouveau_min)
                    pipe.execute()  # Commit — echoue si la cle a change
                    return True
                else:
                    pipe.reset()
                    return False  # Pas besoin de modifier
                    
            except redis.WatchError:
                # La cle a ete modifiee par un autre client → reessayer
                continue
```

---

## Redis Cluster — Bases

Redis Cluster repartit les donnees sur plusieurs noeuds via **consistent hashing**.

```python
from redis.cluster import RedisCluster

# Connexion a un cluster Redis
startup_nodes = [
    {"host": "redis-node-1", "port": 7000},
    {"host": "redis-node-2", "port": 7001},
    {"host": "redis-node-3", "port": 7002}
]

rc = RedisCluster(startup_nodes=startup_nodes, decode_responses=True)

# Utilisation identique au Redis standard
rc.set("ma_cle", "ma_valeur")
print(rc.get("ma_cle"))

# Note : les cles d'un pipeline DOIVENT etre sur le meme slot
# Utiliser les hash tags {} pour forcer la co-localisation
rc.set("{user:42}:profil", "...")
rc.set("{user:42}:session", "...")
# Ces deux cles seront sur le meme noeud grace au hash tag {user:42}
```

---

## Comparaison SSE vs WebSocket vs Long Polling

Pour comprendre quand utiliser Redis Pub/Sub ou Streams avec des clients web :

> [!info] Comparaison des techniques temps-reel
> | Technique | Direction | Latence | Connexion | Cas d'usage |
> |---|---|---|---|---|
> | **Long Polling** | Serveur → Client | ~500ms | HTTP repetees | Legacy, simple |
> | **SSE** | Serveur → Client | ~50ms | HTTP persistante | Notifications, logs |
> | **WebSocket** | Bidirectionnel | ~1ms | WS persistante | Chat, jeux, collaboration |
> | **Redis Pub/Sub** | Serveur → Serveur | <1ms | TCP interne | Entre microservices |

Voir [[05 - WebSockets]] pour l'implementation complete des WebSockets et la comparaison detaillee.

---

## Exercices Pratiques

### Exercice 1 : Cache simple
1. Installez redis et redis-py
2. Implementez une fonction `get_produit(id)` qui :
   - Cherche d'abord dans Redis (cache)
   - Si absent, "simule" une requete SQL (sleep 200ms + retourne un dict)
   - Stocke le resultat dans Redis avec un TTL de 10 minutes
3. Appelez la fonction 10 fois avec le meme ID et mesurez la difference de temps

### Exercice 2 : Structures de donnees
1. Implementez un **leaderboard** de jeu avec un Sorted Set :
   - Ajouter/modifier le score d'un joueur
   - Afficher le top 10
   - Afficher le rang d'un joueur specifique
2. Implementez un **systeme de tags** avec des Sets :
   - Ajouter des tags a des articles
   - Trouver les articles ayant le tag "python"
   - Trouver les articles ayant les tags "python" ET "web" (SINTER)

### Exercice 3 : Sessions
Implementez un systeme de sessions complet :
1. `login(username, password)` → cree une session, retourne un token
2. `get_current_user(token)` → recupere l'utilisateur de la session
3. `logout(token)` → invalide la session
4. `cleanup_expired_sessions()` → Redis gere les TTL automatiquement, pourquoi cette fonction est-elle inutile ?

### Exercice 4 : Rate Limiting
Implementez un rate limiter pour une API :
1. 100 requetes par heure par utilisateur
2. 1000 requetes par heure par adresse IP
3. Testez avec une boucle qui simule 150 requetes
4. Afficher les headers HTTP simules : `X-RateLimit-Remaining`, `X-RateLimit-Reset`

Voir [[08 - APIs REST avec Flask]] pour integrer ce rate limiter dans une vraie API.

### Exercice 5 : Pub/Sub
Creez un systeme de notifications :
1. Un publisher qui publie des evenements (inscription, note, message)
2. Deux subscribers dans des threads separes : un pour les emails, un pour les push notifications
3. Testez que les deux subscribers recoivent bien chaque evenement

> [!warning] A retenir
> - Redis = in-memory → ultra-rapide mais volatil (configurer AOF/RDB pour la persistance)
> - Cache-Aside = pattern le plus courant : lire le cache, sinon DB + remplir le cache
> - TTL obligatoire sur tout ce qui peut devenir stale (perime)
> - Utiliser un pipeline pour les operations batch (1 aller-retour reseau au lieu de N)
> - Pub/Sub = pas de persistance (message perdu si pas d'abonne) → utiliser Streams si important
> - Ne jamais stocker de donnees sensibles sans chiffrement (Redis ne chiffre pas par defaut)
