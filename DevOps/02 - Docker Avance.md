# Docker Avance

## Prerequis

Ce cours suppose une bonne maitrise des bases de Docker : images, conteneurs, Dockerfile, commandes de base (`docker run`, `docker build`, `docker ps`, `docker stop`, `docker rm`). Si ces concepts ne sont pas clairs, revoyez d'abord le cours [[01 - Docker]].

On va ici plonger dans les mecanismes avances qui font la difference entre un usage basique de Docker et une utilisation **professionnelle** en production : builds multi-etapes, reseaux, volumes avances, securite, optimisation et debugging.

> [!tip] Analogie
> Si [[01 - Docker]] c'etait apprendre a conduire une voiture (demarrer, freiner, tourner), ce cours c'est apprendre la **mecanique** : optimiser le moteur, comprendre le systeme electrique, diagnostiquer les pannes. Vous allez comprendre ce qui se passe **sous le capot** pour construire des conteneurs legers, securises et performants.

---

## Multi-Stage Builds

### Le probleme : des images trop lourdes

Quand on construit une image Docker, on inclut souvent des outils de compilation, des dependances de developpement, des fichiers temporaires qui ne sont **pas necessaires** a l'execution. Resultat : des images de 1 Go ou plus.

> [!warning] Images lourdes = problemes
> - **Temps de pull** plus long (deploiement lent)
> - **Surface d'attaque** plus grande (plus de binaires = plus de vulnerabilites)
> - **Couts de stockage** plus eleves (registries)
> - **Demarrage** plus lent dans les orchestrateurs (Kubernetes)

### La solution : les builds multi-etapes

Le principe est simple : utiliser **plusieurs `FROM`** dans un seul Dockerfile. Chaque `FROM` demarre un nouveau **stage**. On peut copier des fichiers d'un stage a l'autre avec `COPY --from=`.

```dockerfile
# Stage 1 : compilation (image lourde avec outils de build)
FROM golang:1.22 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server .

# Stage 2 : execution (image minimale)
FROM scratch
COPY --from=builder /app/server /server
EXPOSE 8080
ENTRYPOINT ["/server"]
```

> [!info] `scratch` est l'image la plus minimale possible
> C'est une image **vide** : pas de shell, pas de libc, rien. Parfaite pour des binaires statiques Go/Rust. Pour du C avec des dependances dynamiques, on utilisera `alpine` ou `distroless`.

### Exemple avec un programme C

```dockerfile
# Stage 1 : compilation avec GCC
FROM gcc:13 AS builder
WORKDIR /src
COPY . .
RUN gcc -static -O2 -o myapp main.c utils.c -lm

# Stage 2 : image minimale
FROM alpine:3.19
RUN adduser -D appuser
COPY --from=builder /src/myapp /usr/local/bin/myapp
USER appuser
ENTRYPOINT ["myapp"]
```

> [!example] Reduction de taille
> | Approche | Taille |
> |---|---|
> | `FROM gcc:13` (tout dans une seule image) | **~1.3 Go** |
> | Multi-stage avec `alpine` | **~12 Mo** |
> | Multi-stage avec `scratch` (binaire statique) | **~2 Mo** |

### Exemple avec une application Python

```dockerfile
# Stage 1 : installer les dependances dans un venv
FROM python:3.12-slim AS builder
WORKDIR /app
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2 : image finale legere
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY . .
EXPOSE 5000
CMD ["gunicorn", "-b", "0.0.0.0:5000", "app:app"]
```

### Builder pattern avance : targets nommees

On peut cibler un stage specifique avec `--target` :

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM base AS development
COPY . .
CMD ["npm", "run", "dev"]

FROM base AS build
COPY . .
RUN npm run build

FROM nginx:alpine AS production
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```bash
docker build --target development -t myapp:dev .   # Stage de dev uniquement
docker build -t myapp:prod .                        # Stage final (production)
```

---

## Docker Networking

### Les drivers reseau

Docker propose plusieurs **drivers reseau** qui determinent comment les conteneurs communiquent entre eux et avec l'exterieur.

| Driver | Description | Cas d'usage |
|---|---|---|
| `bridge` | Reseau prive interne (par defaut) | Conteneurs sur un seul hote |
| `host` | Partage le reseau de la machine hote | Performance maximale, pas d'isolation |
| `none` | Pas de reseau du tout | Securite, batch jobs |
| `overlay` | Reseau multi-hotes | Docker Swarm, communication inter-noeuds |
| `macvlan` | Adresse MAC dediee par conteneur | Integration reseau legacy |

### Le reseau bridge par defaut

Quand on lance un conteneur sans specifier de reseau, il est connecte au **bridge par defaut** (`docker0`).

```bash
# Lancer deux conteneurs sur le bridge par defaut
docker run -d --name web nginx
docker run -d --name api node:20-alpine

# Verifier le reseau
docker network inspect bridge
```

> [!warning] Limite du bridge par defaut
> Sur le bridge par defaut, les conteneurs ne peuvent **pas** se resoudre par nom DNS. Il faut utiliser les adresses IP, ce qui est fragile et non recommande. **Creez toujours un reseau personnalise.**

### Creer un reseau personnalise

```bash
# Creer un reseau bridge personnalise
docker network create mon-reseau

# Lancer des conteneurs sur ce reseau
docker run -d --name web --network mon-reseau nginx
docker run -d --name api --network mon-reseau node:20-alpine

# Connecter un conteneur existant a un reseau
docker network connect mon-reseau mon-conteneur

# Deconnecter
docker network disconnect mon-reseau mon-conteneur
```

### Resolution DNS entre conteneurs

> [!info] DNS automatique sur les reseaux personnalises
> Sur un reseau **personnalise** (user-defined bridge), Docker fournit un serveur DNS integre. Chaque conteneur est accessible par **son nom** (`--name`).

```bash
# Depuis le conteneur 'api', on peut joindre 'web' par son nom
docker exec api ping web
# PING web (172.18.0.2): 56 data bytes
# 64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.089 ms

# Ou utiliser le nom dans du code applicatif
docker exec api curl http://web:80
```

```python
# Dans le code Python du conteneur 'api' :
import requests

# 'db' est le nom du conteneur PostgreSQL sur le meme reseau
response = requests.get("http://web:80/api/data")

# Connexion a la base de donnees par nom de conteneur
DATABASE_URL = "postgresql://user:pass@db:5432/mydb"
```

### Reseau host

```bash
# Le conteneur partage directement le reseau de la machine
docker run -d --network host nginx
# Nginx est accessible sur localhost:80 de la machine hote
# Pas de mapping de port necessaire (-p est ignore)
```

> [!warning] `--network host` ne fonctionne pas sur Docker Desktop (macOS/Windows)
> Docker Desktop utilise une VM Linux. Le mode `host` partage le reseau de cette VM, pas celui de votre machine. Cela ne fonctionne veritablement que sous **Linux natif**.

### Isolation reseau : frontend/backend

```bash
# Creer deux reseaux isoles
docker network create frontend
docker network create backend

# Nginx sur frontend, DB sur backend, API sur les deux
docker run -d --name nginx --network frontend -p 80:80 nginx
docker run -d --name api --network frontend node:20-alpine
docker network connect backend api
docker run -d --name db --network backend postgres:16

# nginx -> api (OK), api -> db (OK), nginx -> db (IMPOSSIBLE)
```

```
  ┌─────────── frontend ────────────┐
  │  ┌───────┐       ┌───────┐     │
  │  │ nginx │◄─────►│  api  │──┐  │
  │  └───────┘       └───────┘  │  │
  └─────────────────────────────│──┘
  ┌─────────── backend ─────────│──┐
  │                  ┌───────┐  │  │
  │                  │  api  │◄─┘  │
  │                  └───┬───┘     │
  │                  ┌───▼───┐     │
  │                  │  db   │     │
  │                  └───────┘     │
  └────────────────────────────────┘
```

---

## Volumes Avances

### Types de montages

Docker propose trois types de montages pour persister des donnees :

| Type | Syntaxe | Gere par Docker | Cas d'usage |
|---|---|---|---|
| **Named volume** | `-v mydata:/data` | Oui | Donnees persistantes (BDD, uploads) |
| **Bind mount** | `-v /host/path:/data` | Non | Dev, fichiers de config |
| **tmpfs** | `--tmpfs /tmp` | Non (en RAM) | Donnees temporaires sensibles |

### Named volumes

```bash
# Creer un volume nomme
docker volume create pgdata

# Utiliser le volume
docker run -d --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:16

# Lister les volumes
docker volume ls

# Inspecter un volume
docker volume inspect pgdata
# Affiche le chemin reel sur la machine hote (Mountpoint)

# Supprimer un volume (ATTENTION : perte de donnees)
docker volume rm pgdata

# Supprimer tous les volumes non utilises
docker volume prune
```

> [!info] Les named volumes survivent aux conteneurs
> Quand on fait `docker rm postgres`, le volume `pgdata` **reste**. Les donnees sont preservees. On peut creer un nouveau conteneur avec le meme volume et retrouver toutes ses donnees.

### tmpfs : stockage en memoire

```bash
# Montage tmpfs (uniquement en RAM, jamais ecrit sur disque)
docker run -d --name secure-app \
  --tmpfs /tmp:rw,noexec,size=100m \
  --tmpfs /run/secrets:ro,size=1m \
  myapp
```

> [!tip] Quand utiliser tmpfs ?
> - **Donnees sensibles temporaires** (tokens, cles de session) : jamais ecrites sur disque
> - **Cache applicatif** : rapide et automatiquement nettoye a l'arret du conteneur
> - **Fichiers temporaires** : evite d'ecrire sur la couche filesystem du conteneur

### Backup et restore de volumes

```bash
# Sauvegarder un volume dans une archive tar
docker run --rm \
  -v pgdata:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/pgdata-backup.tar.gz -C /source .

# Restaurer un volume depuis une archive
docker volume create pgdata-restored
docker run --rm \
  -v pgdata-restored:/target \
  -v $(pwd):/backup:ro \
  alpine tar xzf /backup/pgdata-backup.tar.gz -C /target
```

> [!example] Script de backup automatise
> ```bash
> #!/bin/bash
> VOLUME_NAME="pgdata"
> BACKUP_DIR="/backups"
> DATE=$(date +%Y%m%d_%H%M%S)
> BACKUP_FILE="${BACKUP_DIR}/${VOLUME_NAME}-${DATE}.tar.gz"
>
> docker run --rm \
>   -v ${VOLUME_NAME}:/source:ro \
>   -v ${BACKUP_DIR}:/backup \
>   alpine tar czf /backup/$(basename ${BACKUP_FILE}) -C /source .
>
> echo "Backup cree : ${BACKUP_FILE}"
>
> # Garder uniquement les 7 derniers backups
> ls -t ${BACKUP_DIR}/${VOLUME_NAME}-*.tar.gz | tail -n +8 | xargs rm -f
> ```

### Volume drivers

Pour des besoins avances, on peut utiliser des **drivers de volumes** tiers (NFS, etc.) :

```bash
# Volume NFS (partage reseau)
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/shared/data \
  nfs-volume
```

---

## Docker Secrets et Configs

> [!warning] Les variables d'environnement ne sont pas securisees
> Les `env vars` passees avec `-e` sont visibles dans `docker inspect`, dans `/proc/*/environ` et potentiellement dans les logs. Utilisez les **secrets** pour les donnees sensibles.

### Docker Secrets (Swarm mode)

Les secrets sont chiffres au repos et montes en fichiers dans `/run/secrets/` :

```bash
# Creer un secret (necessite Swarm : docker swarm init)
echo "super_secret_password" | docker secret create db_password -

# Utiliser dans un service
docker service create --name postgres \
  --secret db_password \
  -e POSTGRES_PASSWORD_FILE=/run/secrets/db_password \
  postgres:16
```

> [!info] En dehors de Swarm
> Simulez les secrets avec des bind mounts en lecture seule :
> ```bash
> docker run -d \
>   -v ./secrets/db_password.txt:/run/secrets/db_password:ro \
>   -e POSTGRES_PASSWORD_FILE=/run/secrets/db_password \
>   postgres:16
> ```

Les **Docker Configs** fonctionnent de la meme maniere mais sans chiffrement, pour des fichiers de configuration (nginx.conf, etc.).

---

## Healthchecks

### Pourquoi des healthchecks ?

Un conteneur peut etre en etat `running` mais l'application a l'interieur peut etre **plantee** (boucle infinie, deadlock, connexion BDD perdue). Les healthchecks permettent a Docker de verifier que l'application fonctionne **reellement**.

### Instruction HEALTHCHECK dans le Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt

# Healthcheck : verifie que l'API repond
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1

EXPOSE 5000
CMD ["gunicorn", "-b", "0.0.0.0:5000", "app:app"]
```

| Parametre | Description | Defaut |
|---|---|---|
| `--interval` | Frequence des verifications | 30s |
| `--timeout` | Temps max pour une verification | 30s |
| `--start-period` | Delai avant la premiere verification | 0s |
| `--retries` | Nombre d'echecs avant status `unhealthy` | 3 |
| `--start-interval` | Frequence pendant le start-period | 5s |

### Exemples de healthchecks

```dockerfile
# API HTTP
HEALTHCHECK --interval=15s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# PostgreSQL
HEALTHCHECK --interval=10s --timeout=5s --retries=5 \
  CMD pg_isready -U postgres || exit 1

# Redis
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD redis-cli ping | grep -q PONG || exit 1
```

### Healthcheck au runtime

```bash
# Override le healthcheck du Dockerfile
docker run -d \
  --health-cmd="curl -f http://localhost:5000/health || exit 1" \
  --health-interval=15s --health-timeout=3s --health-retries=3 \
  myapp

# Verifier le status
docker ps          # Colonne STATUS affiche (healthy)/(unhealthy)
docker inspect --format='{{json .State.Health}}' mon-conteneur
```

> [!tip] Healthcheck sans curl
> ```dockerfile
> HEALTHCHECK CMD wget -qO- http://localhost:8080/health || exit 1       # Alpine
> HEALTHCHECK CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')"
> HEALTHCHECK CMD nc -z localhost 8080 || exit 1                          # TCP only
> ```

---

## Optimisation des Images

### Ordre des couches et cache

> [!info] Chaque instruction du Dockerfile cree une couche (layer)
> Docker met en cache chaque couche. Si une couche n'a pas change, Docker la reutilise. **Mais** si une couche change, toutes les couches suivantes sont reconstruites.

```dockerfile
# MAUVAIS : le cache est invalide a chaque changement de code
FROM python:3.12-slim
WORKDIR /app
COPY . .                           # Change a chaque modif de code
RUN pip install -r requirements.txt # Reinstalle tout a chaque fois !
CMD ["python", "app.py"]

# BON : les dependances sont mises en cache separement
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .            # Change rarement
RUN pip install -r requirements.txt # Mis en cache tant que requirements.txt ne change pas
COPY . .                           # Seul ce layer est reconstruit
CMD ["python", "app.py"]
```

```
  Ordre optimal des couches :

  1. Instructions systeme (apt-get, apk add)     ← Change tres rarement
  2. Fichiers de dependances (requirements.txt)   ← Change parfois
  3. Installation des dependances (pip install)   ← Cache si #2 n'a pas change
  4. Code source (COPY . .)                       ← Change souvent
  5. Configuration runtime (CMD, EXPOSE)          ← Change rarement
```

### .dockerignore

Le fichier `.dockerignore` empeche des fichiers inutiles d'etre envoyes dans le **build context** :

```dockerignore
# .dockerignore
.git
.gitignore
__pycache__
*.pyc
.env
.venv
node_modules
*.md
docker-compose*.yml
Dockerfile
.dockerignore
tests/
docs/
.coverage
.pytest_cache
```

> [!warning] Sans `.dockerignore`, le dossier `.git` et `node_modules` sont envoyes au daemon Docker
> Cela peut ajouter des **centaines de Mo** au build context et rallonger considerablement le temps de build.

### Choix de l'image de base

| Image | Taille | Shell | Package Manager | Usage |
|---|---|---|---|---|
| `ubuntu:24.04` | ~78 Mo | bash | apt | Dev, besoin d'outils |
| `debian:bookworm-slim` | ~52 Mo | bash | apt | Production generique |
| `alpine:3.19` | ~7 Mo | sh | apk | Production legere |
| `distroless` | ~2-20 Mo | Non | Non | Production securisee |
| `scratch` | 0 Mo | Non | Non | Binaires statiques |

> [!tip] Alpine vs Distroless
> - **Alpine** : ideal quand on a besoin d'un shell pour le debugging. Utilise `musl` au lieu de `glibc` (peut causer des problemes de compatibilite).
> - **Distroless** (Google) : pas de shell, pas de package manager, pas d'utilitaires. Uniquement le runtime. Surface d'attaque minimale mais debugging plus difficile.
> ```dockerfile
> # Image distroless pour Python
> FROM gcr.io/distroless/python3-debian12
> COPY --from=builder /app /app
> WORKDIR /app
> CMD ["app.py"]
> ```

### Reduire le nombre de layers

```dockerfile
# MAUVAIS : 3 layers separees
RUN apt-get update
RUN apt-get install -y curl wget git
RUN apt-get clean

# BON : 1 seule layer, nettoyage inclus
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl \
      wget \
      git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

> [!info] `--no-install-recommends`
> Ce flag evite d'installer les paquets "recommandes" (pas strictement necessaires), reduisant la taille de l'image.

---

## Securite Docker

### Executer en tant que non-root

> [!warning] Par defaut, les conteneurs tournent en root
> Si un attaquant s'echappe du conteneur, il est root sur la machine hote. **Toujours** creer et utiliser un utilisateur non-root.

```dockerfile
FROM python:3.12-slim
WORKDIR /app

# Creer un utilisateur non-root
RUN groupadd -r appgroup && useradd -r -g appgroup -d /app -s /sbin/nologin appuser

# Installer les dependances en root
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code et changer le proprietaire
COPY --chown=appuser:appgroup . .

# Basculer vers l'utilisateur non-root
USER appuser

EXPOSE 5000
CMD ["gunicorn", "-b", "0.0.0.0:5000", "app:app"]
```

### Filesystem en lecture seule

```bash
# Lancer un conteneur avec un filesystem en lecture seule
docker run -d \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /var/run \
  -v appdata:/data \
  myapp
```

### Limites de ressources

```bash
# Limiter la memoire et le CPU
docker run -d \
  --memory=512m \
  --memory-swap=512m \
  --cpus=1.5 \
  --pids-limit=100 \
  myapp

# Pas de privilege supplementaire
docker run -d \
  --security-opt=no-new-privileges \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  myapp
```

> [!example] Recapitulatif des bonnes pratiques securite
> ```dockerfile
> # Image minimale
> FROM python:3.12-slim
>
> # Pas de cache pip (reduit la taille ET evite des donnees sensibles en cache)
> ENV PIP_NO_CACHE_DIR=1
>
> # Utilisateur non-root
> RUN useradd -r -s /sbin/nologin appuser
>
> WORKDIR /app
> COPY requirements.txt .
> RUN pip install -r requirements.txt
> COPY --chown=appuser:appuser . .
>
> # Pas de root
> USER appuser
>
> # Healthcheck
> HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
>   CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')"
>
> EXPOSE 5000
> CMD ["gunicorn", "-b", "0.0.0.0:5000", "app:app"]
> ```

### Scanner les vulnerabilites avec Trivy

```bash
trivy image myapp:latest                                    # Scanner une image
trivy image --severity CRITICAL,HIGH myapp:latest           # Critiques seulement
trivy image --exit-code 1 --severity CRITICAL myapp:latest  # Fail en CI/CD
trivy config Dockerfile                                      # Scanner un Dockerfile
```

---

## Docker Buildx : Builds Multi-Architecture

### Pourquoi le multi-architecture ?

Avec l'arrivee des puces **ARM** (Apple Silicon M1/M2/M3, AWS Graviton, Raspberry Pi), une image construite sur une architecture ne fonctionne pas forcement sur une autre.

```bash
# Creer un builder multi-architecture
docker buildx create --name multiarch --driver docker-container --use

# Verifier les plateformes supportees
docker buildx inspect multiarch --bootstrap

# Construire pour plusieurs architectures et pousser
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t monregistry/myapp:latest \
  --push \
  .

# Construire et charger localement (une seule plateforme)
docker buildx build \
  --platform linux/arm64 \
  -t myapp:arm64 \
  --load \
  .
```

> [!info] Comment ca marche ?
> Buildx utilise **QEMU** pour emuler les architectures non-natives. C'est plus lent qu'une compilation native, mais ca permet de creer des images multi-arch depuis une seule machine.

```dockerfile
# Dockerfile compatible multi-architecture (utilise les ARG automatiques)
FROM --platform=$BUILDPLATFORM golang:1.22 AS builder
ARG TARGETOS TARGETARCH
WORKDIR /app
COPY . .
RUN GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -o /app/server .

FROM alpine:3.19
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]
```

---

## Debugging de Conteneurs

### docker exec : executer des commandes dans un conteneur

```bash
# Ouvrir un shell interactif dans un conteneur en cours
docker exec -it mon-conteneur /bin/bash

# Si pas de bash (Alpine)
docker exec -it mon-conteneur /bin/sh

# Executer une commande specifique
docker exec mon-conteneur cat /etc/nginx/nginx.conf

# Executer en tant qu'utilisateur specifique
docker exec -u root mon-conteneur apt-get update
```

### docker logs : consulter les logs

```bash
# Voir tous les logs
docker logs mon-conteneur

# Suivre les logs en temps reel (comme tail -f)
docker logs -f mon-conteneur

# Derniers 100 logs
docker logs --tail 100 mon-conteneur

# Logs depuis une date
docker logs --since 2024-01-01T00:00:00 mon-conteneur

# Logs des dernieres 30 minutes
docker logs --since 30m mon-conteneur

# Avec timestamps
docker logs -t mon-conteneur
```

### docker inspect : details complets

```bash
# Toutes les informations du conteneur (JSON)
docker inspect mon-conteneur

# Extraire l'adresse IP
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mon-conteneur

# Extraire les volumes montes
docker inspect -f '{{json .Mounts}}' mon-conteneur | python -m json.tool

# Extraire les variables d'environnement
docker inspect -f '{{json .Config.Env}}' mon-conteneur | python -m json.tool

# Status du healthcheck
docker inspect -f '{{json .State.Health}}' mon-conteneur | python -m json.tool
```

### docker stats et nsenter

```bash
# Monitoring temps reel
docker stats                     # Tous les conteneurs
docker stats --no-stream         # Snapshot ponctuel

# nsenter : entrer dans le namespace d'un conteneur (utile pour distroless/scratch)
PID=$(docker inspect -f '{{.State.Pid}}' mon-conteneur)
sudo nsenter -t $PID -m -u -i -n -p -- /bin/bash
```

> [!tip] Debugging de conteneurs sans shell
> Pour les images `distroless` ou `scratch`, utilisez un **conteneur sidecar** :
> ```bash
> docker run -it --rm \
>   --pid=container:mon-conteneur \
>   --network=container:mon-conteneur \
>   nicolaka/netshoot
> ```

### docker cp : copier des fichiers

```bash
docker cp mon-conteneur:/var/log/app.log ./app.log   # Conteneur -> hote
docker cp ./config.json mon-conteneur:/app/config.json # Hote -> conteneur
```

---

## Dockerfile : Developpement vs Production

> [!example] Comparaison complete

### Dockerfile de developpement

```dockerfile
# Dockerfile.dev
FROM python:3.12

# Outils de debug
RUN apt-get update && apt-get install -y \
    vim curl wget net-tools iputils-ping && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Installer les dependances (dev incluses)
COPY requirements.txt requirements-dev.txt ./
RUN pip install -r requirements.txt -r requirements-dev.txt

# Le code sera monte en bind mount, pas besoin de COPY
# COPY . .

# Activer le mode debug
ENV FLASK_DEBUG=1
ENV FLASK_ENV=development
ENV PYTHONDONTWRITEBYTECODE=1

EXPOSE 5000

# Hot reload avec Flask
CMD ["flask", "run", "--host=0.0.0.0", "--reload"]
```

### Dockerfile de production

```dockerfile
# Dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.12-slim
WORKDIR /app

# Utilisateur non-root
RUN groupadd -r app && useradd -r -g app -d /app app

# Copier le venv depuis le builder
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copier le code
COPY --chown=app:app . .

USER app

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')"

EXPOSE 5000
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "app:app"]
```

### Tableau comparatif

| Aspect | Developpement | Production |
|---|---|---|
| **Image de base** | `python:3.12` (complete) | `python:3.12-slim` (minimale) |
| **Multi-stage** | Non necessaire | Oui |
| **Outils de debug** | Installes (vim, curl, etc.) | Aucun |
| **Dependances** | Dev + prod | Prod uniquement |
| **Code source** | Bind mount (hot reload) | COPY dans l'image |
| **Utilisateur** | root (pratique) | Non-root (securite) |
| **Mode debug** | Active | Desactive |
| **Serveur** | Flask dev server (reload) | Gunicorn (multi-worker) |
| **Healthcheck** | Optionnel | Obligatoire |
| **Taille image** | ~1 Go | ~150 Mo |

---

## Carte Mentale

```
                         Docker Avance
                              │
        ┌─────────┬──────────┬┴──────────┬──────────┬──────────┐
        │         │          │           │          │          │
  Multi-Stage  Reseau    Volumes    Securite   Optimisation  Debug
        │         │          │           │          │          │
   ┌────┤    ┌────┤     ┌────┤      ┌────┤     ┌────┤    ┌────┤
   │    │    │    │     │    │      │    │     │    │    │    │
builder bridge named  non-root  cache   exec
   │    │    │    │     │    │      │    │   order   │    │    │
target host  tmpfs  readonly  .ignore  logs
   │    │    │    │     │    │      │    │     │    │    │    │
scratch overlay backup limits  alpine inspect
   │         │          │      │    │     │         │
distroless  DNS     drivers  Trivy  layers      nsenter
                               │
                            buildx
                         (multi-arch)
```

---

## Exercices

### Exercice 1 : Multi-stage Build C

Creez un programme C simple (`hello.c`) qui affiche "Hello from Docker!". Ecrivez un Dockerfile multi-stage qui :
1. Compile avec `gcc:13`
2. Utilise `scratch` comme image finale (compilez avec `-static`)
3. Verifiez la taille finale avec `docker images`

### Exercice 2 : Reseau et Communication

Creez un setup avec 3 conteneurs :
1. Un conteneur `nginx` sur le reseau `frontend`
2. Un conteneur `redis` sur le reseau `backend`
3. Un conteneur `alpine` connecte aux deux reseaux
4. Verifiez que `alpine` peut `ping` les deux autres, mais que `nginx` ne peut pas `ping` `redis`

### Exercice 3 : Volume Backup/Restore

1. Creez un volume `testdata`, lancez un conteneur Alpine qui ecrit un fichier dedans
2. Sauvegardez le volume dans un `.tar.gz`
3. Supprimez le volume original
4. Restaurez depuis le backup et verifiez que les donnees sont intactes

### Exercice 4 : Optimisation d'Image

Prenez ce Dockerfile inefficace et optimisez-le (objectif : passer de ~900 Mo a moins de 100 Mo) :
```dockerfile
FROM python:3.12
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
RUN apt-get update && apt-get install -y curl
CMD ["python", "app.py"]
```

---

## Liens

- [[01 - Docker]] - Les bases de Docker
- [[03 - Docker Compose en Pratique]] - Orchestration multi-conteneurs
- [[04 - CI-CD avec GitHub Actions]] - Integration et deploiement continu
