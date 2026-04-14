# Docker Compose en Pratique

## Pourquoi Docker Compose ?

Dans la realite, une application n'est presque jamais un seul conteneur. Une application web typique necessite au minimum :
- Un **serveur web** (Nginx, Apache)
- Un **backend** (Flask, Express, Django)
- Une **base de donnees** (PostgreSQL, MySQL)
- Un **cache** (Redis, Memcached)

Lancer et configurer ces conteneurs un par un avec `docker run` est fastidieux, source d'erreurs, et impossible a reproduire de maniere fiable. **Docker Compose** permet de definir et gerer une application **multi-conteneurs** dans un seul fichier YAML.

> [!tip] Analogie
> Imaginez que vous organisez un concert. Sans Compose, vous devez appeler chaque musicien individuellement, leur donner l'adresse, l'heure, les instructions techniques. Avec Compose, vous ecrivez **une seule fiche technique** qui decrit tout : qui joue quoi, sur quelle scene, avec quel materiel, dans quel ordre. Un seul `docker compose up` et tout le monde est en place.

---

## Le fichier de configuration

### docker-compose.yml vs compose.yaml

> [!info] Nomenclature
> - `docker-compose.yml` : nom historique, toujours supporte
> - `compose.yaml` : nom recommande depuis Compose V2
> - `compose.yml` : egalement supporte
>
> Docker Compose cherche dans cet ordre : `compose.yaml`, `compose.yml`, `docker-compose.yml`, `docker-compose.yaml`

### Le champ `version` est obsolete

```yaml
# ANCIEN (plus necessaire depuis Compose V2)
version: "3.8"
services:
  web:
    image: nginx

# MODERNE (pas de champ version)
services:
  web:
    image: nginx
```

> [!warning] Ne mettez plus `version:`
> Depuis Docker Compose V2, le champ `version` est **ignore** et genere un avertissement. Docker Compose determine automatiquement les fonctionnalites disponibles.

---

## Structure de base

```yaml
# compose.yaml
services:
  # Definition des conteneurs
  web:
    image: nginx
  api:
    build: ./api
  db:
    image: postgres:16

volumes:
  # Volumes nommes
  pgdata:
  redis-data:

networks:
  # Reseaux personnalises
  frontend:
  backend:
```

Les trois sections principales :
- **services** : les conteneurs a lancer (obligatoire)
- **volumes** : les volumes nommes (optionnel)
- **networks** : les reseaux personnalises (optionnel)

---

## Exemple Complet : Flask + PostgreSQL + Redis + Nginx

### Architecture

```
                    ┌──────────┐
                    │  Client  │
                    └────┬─────┘
                         │ :80
                    ┌────▼─────┐
                    │  Nginx   │ (reverse proxy)
                    │  :80     │
                    └────┬─────┘
                         │ :5000
                    ┌────▼─────┐
                    │  Flask   │ (API Python)
                    │  :5000   │
                    └──┬────┬──┘
                       │    │
              ┌────────┘    └────────┐
              │ :5432                │ :6379
        ┌─────▼──────┐       ┌──────▼─────┐
        │ PostgreSQL  │       │   Redis    │
        │  :5432      │       │   :6379    │
        └─────────────┘       └────────────┘
```

### compose.yaml

```yaml
services:
  # ─── Reverse Proxy ─────────────────────────────
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      api:
        condition: service_healthy
    networks:
      - frontend
    restart: unless-stopped

  # ─── API Python Flask ──────────────────────────
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    expose:
      - "5000"
    environment:
      - FLASK_ENV=production
      - DATABASE_URL=postgresql://appuser:secret@db:5432/myapp
      - REDIS_URL=redis://redis:6379/0
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s
    networks:
      - frontend
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "1.0"

  # ─── Base de donnees ───────────────────────────
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 1G

  # ─── Cache Redis ───────────────────────────────
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --maxmemory 256mb
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    networks:
      - backend
    restart: unless-stopped

volumes:
  pgdata:
    name: myapp-pgdata
  redis-data:
    name: myapp-redis

networks:
  frontend:
    name: myapp-frontend
  backend:
    name: myapp-backend
    internal: true   # Pas d'acces internet pour le backend
```

> [!info] `expose` vs `ports`
> - **`ports: "80:80"`** : mappe le port sur la machine hote (accessible depuis l'exterieur)
> - **`expose: "5000"`** : rend le port accessible uniquement aux autres conteneurs du meme reseau (pas de mapping sur l'hote)

---

## Options de Service en Detail

### image et build

```yaml
services:
  # Utiliser une image existante
  db:
    image: postgres:16-alpine

  # Construire depuis un Dockerfile
  api:
    build: ./api

  # Build avec options avancees
  api-advanced:
    build:
      context: ./api
      dockerfile: Dockerfile.prod
      args:
        PYTHON_VERSION: "3.12"
        BUILD_DATE: "2024-01-15"
      target: production
      cache_from:
        - myapp:cache
    image: myapp:latest   # Tag de l'image construite
```

### ports, volumes, environment

```yaml
services:
  web:
    ports:
      - "80:80"                          # HOST:CONTAINER
      - "127.0.0.1:8080:80"             # Ecouter uniquement sur localhost
      - "5000"                           # Port aleatoire sur l'hote

  api:
    volumes:
      - pgdata:/var/lib/postgresql/data  # Volume nomme
      - ./src:/app/src                   # Bind mount
      - ./config/app.conf:/etc/app.conf:ro  # Lecture seule

    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - SECRET_KEY=${SECRET_KEY}         # Depuis le shell ou .env
    env_file:
      - .env
```

### depends_on avec conditions

```yaml
services:
  api:
    depends_on:
      # Simple (attend que le conteneur demarre)
      redis:
        condition: service_started

      # Attend que le healthcheck passe
      db:
        condition: service_healthy

      # Attend que le conteneur se termine avec succes
      migrations:
        condition: service_completed_successfully
```

> [!warning] `depends_on` sans condition ne garantit pas que le service est pret
> `service_started` signifie seulement que le conteneur a demarre, pas que l'application est prete. Pour PostgreSQL par exemple, le conteneur peut etre "started" alors que le serveur de base de donnees est encore en cours d'initialisation. **Utilisez toujours `condition: service_healthy`** avec un healthcheck.

### restart, networks, command, deploy

```yaml
services:
  api:
    restart: unless-stopped   # "no" | "always" | "on-failure" | "unless-stopped"
    command: ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "app:app"]
    networks:
      - frontend
      - backend
    deploy:
      resources:
        limits:
          cpus: "2.0"
          memory: 1G
      replicas: 3

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true      # Pas d'acces a internet
```

---

## Fichier .env et Substitution de Variables

### Le fichier .env

Docker Compose charge automatiquement un fichier `.env` place a cote du `compose.yaml` :

```bash
# .env
POSTGRES_USER=appuser
POSTGRES_PASSWORD=supersecret
POSTGRES_DB=myapp
APP_PORT=5000
REDIS_VERSION=7
ENVIRONMENT=production
```

```yaml
# compose.yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}

  api:
    ports:
      - "${APP_PORT}:5000"

  redis:
    image: redis:${REDIS_VERSION}-alpine
```

### Syntaxe de substitution

| Syntaxe | Comportement |
|---|---|
| `${VAR}` | Valeur de VAR, vide si absente |
| `${VAR:-default}` | default si VAR absente ou vide |
| `${VAR:?message}` | Erreur avec message si VAR absente ou vide |

```yaml
services:
  api:
    environment:
      - SECRET_KEY=${SECRET_KEY:?SECRET_KEY must be set}
      - LOG_LEVEL=${LOG_LEVEL:-info}
```

> [!warning] Ne commitez jamais le fichier `.env` contenant des secrets
> Ajoutez `.env` au `.gitignore` et fournissez un `.env.example` avec des valeurs fictives.

---

## Volumes Nommes pour la Persistance

```yaml
services:
  db:
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    volumes:
      - redis-data:/data

volumes:
  pgdata:
    name: myapp-pgdata     # Nom explicite (sinon prefixe par le nom du projet)
    driver: local
    labels:
      com.myapp.description: "PostgreSQL data"

  redis-data:
    name: myapp-redis
    driver: local
```

> [!tip] Cycle de vie des volumes
> ```bash
> # docker compose down : NE supprime PAS les volumes
> docker compose down
>
> # docker compose down -v : SUPPRIME les volumes
> docker compose down -v    # ATTENTION : perte de donnees !
>
> # Verifier les volumes existants
> docker volume ls | grep myapp
> ```
> C'est un piege classique : `down -v` detruit vos donnees PostgreSQL. Utilisez-le uniquement pour un reset complet.

---

## Reseaux Personnalises

### Isolation frontend / backend

```yaml
services:
  nginx:
    networks:
      - frontend       # Accessible depuis l'exterieur

  api:
    networks:
      - frontend       # Communique avec nginx
      - backend        # Communique avec db et redis

  db:
    networks:
      - backend        # Isole, pas d'acces internet

  redis:
    networks:
      - backend        # Isole, pas d'acces internet

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true     # Bloque l'acces internet
```

> [!info] Reseau par defaut
> Si aucun reseau n'est defini, Compose cree un reseau bridge `<projet>_default` et y connecte tous les services. Tous les conteneurs peuvent se joindre par leur nom de service.

Resultat : `nginx` ne peut joindre que `api`, `api` peut joindre tout le monde, `db` et `redis` sont isoles du frontend.

---

## Profiles : Dev vs Test vs Prod

Les profiles permettent de definir quels services demarrer selon le contexte :

```yaml
services:
  api:
    build: ./api
    # Pas de profile = toujours demarre

  db:
    image: postgres:16-alpine
    # Pas de profile = toujours demarre

  redis:
    image: redis:7-alpine
    # Pas de profile = toujours demarre

  # Outils de dev uniquement
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@local.dev
      PGADMIN_DEFAULT_PASSWORD: admin
    profiles:
      - dev
      - debug

  # Monitoring (prod uniquement)
  prometheus:
    image: prom/prometheus
    profiles:
      - prod
      - monitoring

  grafana:
    image: grafana/grafana
    profiles:
      - prod
      - monitoring

  # Tests
  test-runner:
    build:
      context: ./api
      target: test
    command: pytest -v
    profiles:
      - test
```

```bash
# Demarrer les services sans profile (api, db, redis)
docker compose up -d

# Demarrer avec le profil dev (api, db, redis + pgadmin)
docker compose --profile dev up -d

# Demarrer avec plusieurs profils
docker compose --profile dev --profile monitoring up -d

# Lancer les tests
docker compose --profile test run --rm test-runner
```

---

## Commandes Docker Compose

### Commandes essentielles

```bash
# Demarrer tous les services en arriere-plan
docker compose up -d

# Demarrer et forcer le rebuild des images
docker compose up -d --build

# Arreter et supprimer les conteneurs
docker compose down

# Arreter et supprimer conteneurs + volumes + images
docker compose down -v --rmi all

# Voir les conteneurs en cours
docker compose ps

# Voir les conteneurs (y compris arretes)
docker compose ps -a

# Suivre les logs de tous les services
docker compose logs -f

# Logs d'un service specifique
docker compose logs -f api

# Logs des 50 dernieres lignes
docker compose logs --tail 50 api

# Executer une commande dans un service
docker compose exec api bash
docker compose exec db psql -U appuser -d myapp

# Lancer un conteneur one-shot (supprime apres execution)
docker compose run --rm api python manage.py migrate

# Reconstruire les images
docker compose build

# Reconstruire sans cache
docker compose build --no-cache

# Telecharger les images
docker compose pull

# Redemarrer un service
docker compose restart api

# Voir les processus dans les conteneurs
docker compose top
```

### Commandes avancees

```bash
docker compose up -d --scale api=3     # Plusieurs instances d'un service
docker compose config                   # Config finale (apres substitution)
docker compose config --quiet           # Valider le fichier
docker compose images                   # Images utilisees
docker compose events                   # Evenements en temps reel
```

---

## Override Files

### Principe

Docker Compose charge automatiquement les fichiers dans cet ordre :
1. `compose.yaml` (ou `docker-compose.yml`)
2. `compose.override.yaml` (ou `docker-compose.override.yml`)

Le fichier override **fusionne** ses valeurs avec le fichier principal.

### Exemple : override pour le developpement

```yaml
# compose.override.yaml (dev - charge automatiquement)
services:
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.dev
    ports:
      - "5000:5000"        # Exposer pour le dev
      - "5678:5678"        # Port debugger
    volumes:
      - ./api/src:/app/src  # Hot reload
    environment:
      - DEBUG=true
      - FLASK_DEBUG=1

  db:
    ports:
      - "5432:5432"        # Exposer pour les outils locaux
    environment:
      POSTGRES_PASSWORD: devpassword
```

### Fichier de production explicite

```yaml
# compose.prod.yaml
services:
  api:
    build: { context: ./api, target: production }
    restart: always
    deploy:
      resources: { limits: { memory: 1G, cpus: "2.0" } }
      replicas: 3
  db:
    restart: always
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD:?DB_PASSWORD is required}
  nginx:
    image: nginx:alpine
    ports: ["80:80", "443:443"]
    volumes: ["./nginx/prod.conf:/etc/nginx/nginx.conf:ro"]
    restart: always
```

```bash
docker compose up -d                                        # Dev (auto-override)
docker compose -f compose.yaml -f compose.prod.yaml up -d   # Production
```

---

## Docker Compose Watch

> [!info] `docker compose watch` (Compose >= 2.22)
> Le mode **watch** surveille les fichiers locaux et synchronise automatiquement les modifications dans les conteneurs, sans rebuild complet. Ideal pour le developpement avec hot reload.

```yaml
# compose.yaml
services:
  api:
    build: ./api
    develop:
      watch:
        # Sync : copie les fichiers modifies dans le conteneur
        - action: sync
          path: ./api/src
          target: /app/src
          ignore:
            - __pycache__/
            - "*.pyc"

        # Sync + Restart : copie et redemarre le conteneur
        - action: sync+restart
          path: ./api/requirements.txt
          target: /app/requirements.txt

        # Rebuild : reconstruit l'image completement
        - action: rebuild
          path: ./api/Dockerfile
```

```bash
# Lancer en mode watch
docker compose watch

# Ou combiner avec up
docker compose up --watch
```

| Action | Comportement | Cas d'usage |
|---|---|---|
| `sync` | Copie les fichiers dans le conteneur | Code source (avec hot reload) |
| `sync+restart` | Copie + redemarre le conteneur | Config, dependances |
| `rebuild` | Reconstruit l'image | Dockerfile, systeme |

---

## Patterns Courants

### Wait-for-it : attendre qu'un service soit pret

```yaml
services:
  api:
    depends_on:
      db: { condition: service_healthy }
    command: >
      sh -c "./wait-for-it.sh db:5432 --timeout=30 --strict && python app.py"
```

> [!tip] Alternatives a wait-for-it
> - **`depends_on` avec healthcheck** : solution native recommandee
> - **Retry dans le code** : meilleure approche pour la production
> ```python
> def connect_with_retry(url, max_retries=10, delay=2):
>     for attempt in range(max_retries):
>         try:
>             return psycopg2.connect(url)
>         except psycopg2.OperationalError:
>             print(f"DB not ready, retry {attempt+1}/{max_retries}")
>             time.sleep(delay)
>     raise Exception("Could not connect to database")
> ```

### Hot reload en dev

```yaml
# compose.override.yaml (dev) - monter le code source en bind mount
services:
  api:       # Flask
    volumes: ["./api/src:/app/src"]
    environment: [FLASK_DEBUG=1]
    command: flask run --host=0.0.0.0 --reload
  frontend:  # Node.js
    volumes: ["./frontend/src:/app/src"]
    command: npx nodemon --watch src src/index.js
```

### Healthchecks dans Compose

```yaml
services:
  db:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  redis:
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 15s
      timeout: 5s
      retries: 3
```

---

## Carte Mentale

```
                     Docker Compose
                          │
      ┌──────────┬────────┼────────┬──────────┐
      │          │        │        │          │
  compose.yaml  Services  Reseaux  Volumes  Commandes
      │          │        │        │          │
 ┌────┤     ┌────┤    ┌───┤    ┌───┤     ┌────┤
 │    │     │    │    │   │    │   │     │    │
.env  │   image  │  bridge │  named │   up -d  │
 │    │     │    │    │   │    │   │     │    │
override  build  │  internal  bind  down -v
 │    │     │    │    │   │          │    │
profiles ports   │  frontend       logs -f
      │     │    │    │   │          │    │
  watch  volumes │  backend       exec
         │    │                    │    │
      depends_on                build
         │    │
      healthcheck
         │
      restart
```

---

## Exercices

### Exercice 1 : Stack Web Complete

Creez un `compose.yaml` pour une application avec :
- Un frontend **Nginx** servant des fichiers statiques (port 80)
- Un backend **Node.js** (Express) sur le port 3000
- Une base **PostgreSQL** avec un volume nomme
- Un **Redis** pour le cache
- Deux reseaux : `frontend` et `backend` avec isolation correcte
- Des healthchecks pour chaque service

### Exercice 2 : Override Dev/Prod

A partir de l'exercice 1, creez :
1. Un `compose.override.yaml` qui expose les ports des BDD, monte le code en bind mount, active le debug
2. Un `compose.prod.yaml` qui limite les ressources, active les restart policies, et n'expose que le port 80

### Exercice 3 : Profiles

Ajoutez a votre stack :
- Un service `pgadmin` (profil `dev`)
- Un service `test-runner` qui execute les tests (profil `test`)
- Demarrez les services avec le profil `dev`, puis lancez les tests avec `run`

### Exercice 4 : Variables et Secrets

Creez un fichier `.env.example` documentant toutes les variables necessaires, puis :
1. Utilisez `${VAR:-default}` pour les valeurs optionnelles
2. Utilisez `${VAR:?message}` pour les valeurs obligatoires en production
3. Testez avec `docker compose config` que la substitution fonctionne

---

## Liens

- [[01 - Docker]] - Les bases de Docker
- [[02 - Docker Avance]] - Concepts avances (multi-stage, networking, securite)
- [[04 - CI-CD avec GitHub Actions]] - Integration et deploiement continu
