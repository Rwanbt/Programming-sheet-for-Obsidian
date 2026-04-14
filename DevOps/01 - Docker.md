# Docker - Les Conteneurs

## Qu'est-ce que Docker ?

**Docker** est un outil qui permet de créer des **conteneurs** : des environnements isolés et portables qui contiennent tout ce dont une application a besoin pour fonctionner (OS, bibliothèques, dépendances, code).

> [!tip] Analogie Gaming
> Pense à une **cartouche de jeu** (NES, Game Boy) :
> - La **cartouche** = une **image Docker**. C'est un template figé qui contient le jeu complet.
> - Quand tu **insères la cartouche** dans la console = `docker run`. Tu crées une **instance** du jeu (un conteneur).
> - Tu peux insérer la **même cartouche** dans n'importe quelle console compatible → ça marche pareil partout.
> - Tu peux avoir **plusieurs consoles** avec la même cartouche en même temps → plusieurs conteneurs de la même image.
> - **Éteindre la console** = `docker stop`. Le jeu est en pause.
> - **Retirer la cartouche** = `docker rm`. L'instance est supprimée.
>
> Docker garantit : "Si ça marche sur ma machine, ça marche partout."

---

## Image vs Conteneur

> [!warning] LA distinction fondamentale

| Concept | Nature | Analogie | Commande |
|---|---|---|---|
| **Image** | Statique (lecture seule) | La recette de cuisine | `docker build` / `docker pull` |
| **Conteneur** | Dynamique (en cours d'exécution) | Le plat cuisiné | `docker run` |

- Une **image** est un **modèle figé** : elle contient l'OS, les bibliothèques, le code. Elle ne change pas.
- Un **conteneur** est une **instance vivante** d'une image : il s'exécute, a son propre système de fichiers, ses propres processus.

Tu peux créer **plusieurs conteneurs** à partir d'une **seule image**, comme tu peux cuisiner plusieurs plats à partir de la même recette.

---

## Le cycle de vie complet

```
  Dockerfile     →   docker build   →   Image   →   docker run   →   Conteneur
  (instructions)     (construction)     (figée)      (lancement)     (vivant)
                                                                        |
                                                              docker stop (arrêté)
                                                                        |
                                                              docker rm  (supprimé)
```

### `stop` vs `rm` : LA confusion n°1 des débutants

> [!warning] `stop` ≠ `rm`

| Commande | Ce qui se passe | Analogie |
|---|---|---|
| `docker stop` | Le conteneur est **arrêté** mais existe encore | Éteindre la console (la partie est en pause) |
| `docker rm` | Le conteneur est **supprimé définitivement** | Jeter la console à la poubelle |

```bash
docker stop mon-conteneur     # Arrêté, visible dans docker ps -a
docker rm mon-conteneur       # Supprimé, n'existe plus du tout
```

Un conteneur **arrêté** :
- Apparaît dans `docker ps -a` (mais pas dans `docker ps`)
- Peut être **redémarré** avec `docker start`
- Prend de l'**espace disque**

Un conteneur **supprimé** :
- N'existe plus du tout
- Il faut refaire `docker run` pour en créer un nouveau

---

## VM vs Conteneur

> [!info] LA difference fondamentale
> Une **Machine Virtuelle** simule un ordinateur complet avec son propre OS. Un **Conteneur** partage le noyau de la machine hote et ne contient que l'application et ses dependances.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MACHINE VIRTUELLE (VM)                           │
│                                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                         │
│  │  App A   │  │  App B   │  │  App C   │                         │
│  ├──────────┤  ├──────────┤  ├──────────┤                         │
│  │  Bins/   │  │  Bins/   │  │  Bins/   │                         │
│  │  Libs    │  │  Libs    │  │  Libs    │                         │
│  ├──────────┤  ├──────────┤  ├──────────┤                         │
│  │  OS      │  │  OS      │  │  OS      │  ← Chaque VM a son OS  │
│  │  complet │  │  complet │  │  complet │                         │
│  └──────────┘  └──────────┘  └──────────┘                         │
│  ┌──────────────────────────────────────┐                          │
│  │         Hyperviseur (VMware, VBox)   │                          │
│  └──────────────────────────────────────┘                          │
│  ┌──────────────────────────────────────┐                          │
│  │         Materiel physique            │                          │
│  └──────────────────────────────────────┘                          │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                       CONTENEUR (Docker)                            │
│                                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                         │
│  │  App A   │  │  App B   │  │  App C   │                         │
│  ├──────────┤  ├──────────┤  ├──────────┤                         │
│  │  Bins/   │  │  Bins/   │  │  Bins/   │                         │
│  │  Libs    │  │  Libs    │  │  Libs    │                         │
│  └──────────┘  └──────────┘  └──────────┘                         │
│  ┌──────────────────────────────────────┐                          │
│  │         Docker Engine                │  ← Partage le noyau     │
│  └──────────────────────────────────────┘                          │
│  ┌──────────────────────────────────────┐                          │
│  │         OS Hote (Linux)              │                          │
│  └──────────────────────────────────────┘                          │
│  ┌──────────────────────────────────────┐                          │
│  │         Materiel physique            │                          │
│  └──────────────────────────────────────┘                          │
└─────────────────────────────────────────────────────────────────────┘
```

| Caracteristique | Machine Virtuelle (VM) | Conteneur (Docker) |
|---|---|---|
| **Isolation** | Totale (niveau materiel) | Partielle (niveau processus) |
| **OS Invite** | Oui (complet) | Non (partage le noyau de l'hote) |
| **Poids** | Lourd (plusieurs Go) | Leger (quelques Mo) |
| **Demarrage** | Lent (minutes) | Instantane (secondes) |
| **Ressources** | Gourmand (RAM/CPU reserves) | Econome (utilise juste le necessaire) |
| **Portabilite** | Moyenne (fichiers lourds) | Excellente (image legere) |
| **Outil** | Hyperviseur (VMware, VirtualBox) | Moteur de conteneur (Docker) |

> [!tip] Quand choisir quoi ?
> - **VM** : besoin de faire tourner des OS differents (ex: Windows sur Mac), ou securite exigeant une isolation physique totale
> - **Conteneur** : deploiement rapide de microservices, economies de ressources, garantie que l'app tourne de la meme maniere partout

---

## Docker + VS Code : Dev Containers

L'extension **Dev Containers** de VS Code permet de travailler directement **a l'interieur** d'un conteneur Docker, avec l'explorateur de fichiers et les extensions lies au Linux du conteneur.

### Methode 1 : Via l'extension Docker (la plus simple)

1. Installer l'extension **Docker** (de Microsoft) dans VS Code
2. Cliquer sur l'icone Baleine (Docker) dans la barre laterale gauche
3. Derouler la section **Containers**
4. Clic droit sur le conteneur → **Attach Shell**
5. Un terminal s'ouvre en bas de l'ecran, directement a l'interieur du conteneur

### Methode 2 : Via le terminal integre

```bash
docker exec -it nom_du_conteneur /bin/bash
```

- `-it` : interaction interactive
- `/bin/bash` : lance le shell Bash

### Methode 3 : Remote - Containers (recommandee)

1. Installer l'extension **Dev Containers**
2. Cliquer sur le bouton vert/bleu en bas a gauche de VS Code (icone `><`)
3. Selectionner **Attach to Running Container...**
4. Choisir le conteneur dans la liste
5. Une nouvelle fenetre VS Code s'ouvre : on code depuis **l'interieur** du conteneur

> [!tip] Conseil
> Une fois dans le conteneur, verifier que les fichiers sont bien la avec `ls -a`. Si les fichiers de code ne sont pas visibles, ils sont probablement montes dans un sous-dossier specifique.

---

## Troubleshooting Docker

### Le daemon Docker ne repond pas

```
failed to connect to the docker API at unix:///var/run/docker.sock
```

**Cause** : le service Docker n'est pas lance en arriere-plan.

**Solutions** :
```bash
# Relancer Docker Desktop (Linux)
systemctl --user start docker-desktop

# Verifier le contexte Docker
docker context use desktop-linux

# Verifier que le daemon est actif
docker info
```

### Quitter `docker stats` bloque dans le terminal

`docker stats` affiche les ressources en temps reel et accapare le terminal.

| Action | Commande |
|---|---|
| Quitter | `Ctrl + C` |
| Forcer la fermeture | `Ctrl + \` (SIGQUIT) |
| Instantane sans blocage | `docker stats --no-stream` |
| Filtrer un conteneur | `docker stats nom_du_conteneur` |

Si le terminal est completement gele, ouvrir un nouveau terminal et tuer le processus :
```bash
ps aux | grep "docker stats"
kill -9 <PID>
```

### Warnings de locale Perl (Betty linter)

```
perl: warning: Setting locale failed.
perl: warning: Falling back to the standard locale ("C").
```

**Cause** : l'environnement du conteneur n'a pas la langue/charset `en_US.UTF-8` installe. Ce n'est **pas** une erreur dans le code C.

**Fix rapide (temporaire)** :
```bash
export LC_ALL=C
```

**Fix permanent** :
```bash
sudo apt-get update && sudo apt-get install -y locales && sudo locale-gen en_US.UTF-8
```

**Astuce** : pour filtrer ces warnings et ne voir que le retour de Betty :
```bash
betty fichier.c 2>&1 | grep -v "perl: warning"
```

---

## Installation sur Ubuntu

```bash
# 1. Supprimer les anciennes versions
sudo apt remove docker docker-engine docker.io containerd runc

# 2. Installer les prérequis
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release

# 3. Ajouter la clé GPG officielle de Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 4. Ajouter le dépôt Docker
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Installer Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 6. Ajouter ton utilisateur au groupe docker (pour ne pas taper sudo)
sudo usermod -aG docker $USER
# Puis déconnecte-toi et reconnecte-toi

# 7. Vérifier
docker --version
docker run hello-world
```

---

## Gérer les images

### Rechercher et télécharger

```bash
docker search ubuntu                  # Chercher une image
docker pull ubuntu                    # Télécharger la dernière version
docker pull ubuntu:22.04              # Télécharger une version spécifique (tag)
docker pull gcc:latest                # Image avec GCC
docker pull python:3.11               # Python 3.11
```

> [!info] Les tags
> Le **tag** est la version de l'image. `ubuntu` = `ubuntu:latest`. Toujours spécifier un tag en production pour éviter les surprises :
> ```bash
> docker pull ubuntu:22.04    # Bien : version précise
> docker pull ubuntu:latest   # Risqué : peut changer
> ```

### Lister et supprimer

```bash
docker images                          # Lister toutes les images locales
docker rmi ubuntu:22.04                # Supprimer une image
docker rmi abc123                      # Supprimer par ID
docker image prune                     # Supprimer les images non utilisées
docker image prune -a                  # Supprimer TOUTES les images non utilisées
```

---

## `docker run` - Créer et lancer un conteneur

### Syntaxe de base

```bash
docker run [OPTIONS] IMAGE [COMMANDE]
```

### Les options essentielles

| Option | Description | Exemple |
|---|---|---|
| `-it` | Mode interactif + terminal (tu peux taper des commandes) | `docker run -it ubuntu bash` |
| `-d` | Mode détaché (en arrière-plan) | `docker run -d nginx` |
| `--name` | Donner un nom au conteneur | `docker run --name mon-app ubuntu` |
| `-p` | Mapper un port (hôte:conteneur) | `docker run -p 8080:80 nginx` |
| `-v` | Monter un volume (hôte:conteneur) | `docker run -v /mon/code:/app ubuntu` |
| `--rm` | Supprimer automatiquement le conteneur à l'arrêt | `docker run --rm ubuntu echo "hello"` |
| `-e` | Définir une variable d'environnement | `docker run -e MY_VAR=42 ubuntu` |
| `-w` | Définir le répertoire de travail | `docker run -w /app ubuntu` |

### Exemples concrets

```bash
# Shell interactif dans Ubuntu
docker run -it ubuntu bash

# Serveur web Nginx sur le port 8080
docker run -d --name web -p 8080:80 nginx

# Compiler un projet C
docker run --rm -v $(pwd):/app -w /app gcc:latest gcc -o prog main.c

# Lancer un programme avec des arguments
docker run --rm -it -v $(pwd):/app -w /app ubuntu ./prog arg1 arg2
```

> [!tip] `-it` = Interactif + Terminal
> - `-i` = interactive (garde STDIN ouvert)
> - `-t` = TTY (alloue un pseudo-terminal)
> - Ensemble, `-it` te donne un shell dans le conteneur.

> [!info] Mapping de ports `-p`
> ```
> -p 8080:80
>      |    |
>      |    └── Port DANS le conteneur (Nginx écoute sur 80)
>      └─────── Port sur TA machine (tu accèdes via localhost:8080)
> ```

---

## Gérer les conteneurs

### Commandes de gestion

```bash
# Lister les conteneurs
docker ps                   # Conteneurs en cours d'exécution
docker ps -a                # TOUS les conteneurs (y compris arrêtés)

# Démarrer / arrêter
docker start mon-conteneur   # Démarrer un conteneur arrêté
docker stop mon-conteneur    # Arrêter proprement (SIGTERM)
docker kill mon-conteneur    # Forcer l'arrêt (SIGKILL)
docker restart mon-conteneur # Redémarrer

# Exécuter une commande dans un conteneur en cours
docker exec -it mon-conteneur bash     # Ouvrir un shell
docker exec mon-conteneur ls /app      # Lancer une commande

# Logs et monitoring
docker logs mon-conteneur              # Voir les logs
docker logs -f mon-conteneur           # Suivre les logs en temps réel
docker stats                           # CPU, mémoire, réseau (comme top)
docker inspect mon-conteneur           # Toutes les infos (JSON)

# Copier des fichiers
docker cp fichier.txt mon-conteneur:/app/    # Hôte → Conteneur
docker cp mon-conteneur:/app/result.txt .    # Conteneur → Hôte

# Supprimer
docker rm mon-conteneur                # Supprimer un conteneur arrêté
docker rm -f mon-conteneur             # Forcer la suppression (même en cours)
```

---

## Dockerfile - Construire ses propres images

Un **Dockerfile** est un fichier texte contenant les instructions pour construire une image.

### Les instructions

| Instruction | Description | Exemple |
|---|---|---|
| `FROM` | Image de base (OBLIGATOIRE, première instruction) | `FROM ubuntu:22.04` |
| `RUN` | Exécuter une commande pendant la construction | `RUN apt update && apt install -y gcc` |
| `WORKDIR` | Définir le répertoire de travail | `WORKDIR /app` |
| `COPY` | Copier des fichiers de l'hôte vers l'image | `COPY main.c /app/` |
| `ADD` | Comme COPY, mais peut extraire des archives | `ADD archive.tar.gz /app/` |
| `ENV` | Définir une variable d'environnement | `ENV CC=gcc` |
| `EXPOSE` | Documenter le port utilisé | `EXPOSE 8080` |
| `CMD` | Commande par défaut au lancement du conteneur | `CMD ["./prog"]` |
| `ENTRYPOINT` | Point d'entrée (non remplaçable facilement) | `ENTRYPOINT ["./prog"]` |

> [!warning] CMD vs ENTRYPOINT
> - **CMD** : commande par défaut. Peut être remplacée par l'utilisateur :
>   ```bash
>   docker run mon-image bash    # Remplace CMD par bash
>   ```
> - **ENTRYPOINT** : point d'entrée fixe. L'utilisateur peut ajouter des arguments :
>   ```bash
>   docker run mon-image arg1    # arg1 est passé à ENTRYPOINT
>   ```
> - **Ensemble** : ENTRYPOINT = la commande, CMD = les arguments par défaut :
>   ```dockerfile
>   ENTRYPOINT ["./prog"]
>   CMD ["--verbose"]
>   # → docker run mon-image            → ./prog --verbose
>   # → docker run mon-image --quiet    → ./prog --quiet
>   ```

### Exemple concret : Projet C

```dockerfile
# Dockerfile pour un projet C
FROM ubuntu:22.04

# Installer les outils
RUN apt update && apt install -y \
    gcc \
    make \
    valgrind \
    gdb \
    && rm -rf /var/lib/apt/lists/*

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers source
COPY *.c *.h Makefile /app/

# Compiler le projet
RUN make

# Commande par défaut
CMD ["./prog"]
```

### `.dockerignore`

Comme `.gitignore`, mais pour Docker. Évite de copier des fichiers inutiles dans l'image.

```
# .dockerignore
.git
*.o
a.out
.env
node_modules
build/
```

### Construire l'image

```bash
# Construire avec un tag
docker build -t mon-projet:1.0 .

# Construire sans cache (force la reconstruction)
docker build --no-cache -t mon-projet:1.0 .

# Le point "." = le contexte de build (le dossier courant)
```

> [!info] Couches de cache (layers)
> Chaque instruction du Dockerfile crée une **couche**. Docker met en cache chaque couche. Si tu changes une ligne, seules les couches **à partir de cette ligne** sont reconstruites.
>
> **Bonne pratique** : mets les instructions qui changent rarement en haut (apt install) et celles qui changent souvent en bas (COPY de tes fichiers source).
>
> ```dockerfile
> # BON ORDRE :
> FROM ubuntu:22.04
> RUN apt update && apt install -y gcc    # Change rarement → en haut
> WORKDIR /app
> COPY *.c *.h Makefile /app/             # Change souvent → en bas
> RUN make
> ```

---

## Volumes - Persister les données

Par défaut, quand un conteneur est supprimé, **toutes ses données sont perdues**. Les volumes permettent de persister les données.

### Bind mounts (monter un dossier)

Tu montes un dossier de ton **hôte** dans le conteneur. Les modifications sont synchronisées en temps réel.

```bash
docker run -v /chemin/hote:/chemin/conteneur image

# Exemple : monter ton code source
docker run -it -v $(pwd):/app -w /app gcc:latest bash
```

> [!tip] Quand utiliser un bind mount ?
> - Développement : monter ton code source pour le modifier en direct
> - Partager des fichiers entre l'hôte et le conteneur

### Docker volumes (volumes gérés par Docker)

Docker gère le stockage. Plus propre et plus performant.

```bash
# Créer un volume
docker volume create mes-donnees

# Utiliser un volume
docker run -v mes-donnees:/app/data image

# Lister les volumes
docker volume ls

# Supprimer un volume
docker volume rm mes-donnees
```

> [!tip] Quand utiliser un Docker volume ?
> - Bases de données (les données survivent à la suppression du conteneur)
> - Données partagées entre plusieurs conteneurs

### Comparaison

| Aspect | Bind mount | Docker volume |
|---|---|---|
| Syntaxe | `-v /chemin/hote:/conteneur` | `-v nom-volume:/conteneur` |
| Géré par | Toi | Docker |
| Emplacement | N'importe où sur ton disque | `/var/lib/docker/volumes/` |
| Portable | Non (dépend du chemin hôte) | Oui |
| Utilisation | Développement | Production, bases de données |

---

## Docker Hub

Docker Hub est le **registre public** d'images Docker (comme GitHub pour le code).

```bash
# Se connecter
docker login

# Tagger une image pour Docker Hub
docker tag mon-image:1.0 username/mon-image:1.0

# Pousser sur Docker Hub
docker push username/mon-image:1.0

# Télécharger depuis Docker Hub
docker pull username/mon-image:1.0
```

---

## Docker Compose

Docker Compose permet de définir et gérer des applications **multi-conteneurs** avec un fichier YAML.

### Exemple : Application web + base de données

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:80"
    volumes:
      - ./src:/app
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    depends_on:
      - db

  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=myapp
    ports:
      - "5432:5432"

volumes:
  db-data:
```

### Commandes Docker Compose

```bash
# Lancer tous les services
docker compose up

# Lancer en arrière-plan
docker compose up -d

# Reconstruire les images avant de lancer
docker compose up --build

# Arrêter tous les services
docker compose down

# Arrêter et supprimer les volumes
docker compose down -v

# Voir les logs
docker compose logs
docker compose logs -f web    # Suivre les logs du service "web"

# Voir l'état des services
docker compose ps

# Exécuter une commande dans un service
docker compose exec web bash

# Redémarrer un service
docker compose restart web
```

> [!info] `docker-compose` vs `docker compose`
> - `docker-compose` (avec tiret) = ancienne version standalone (v1)
> - `docker compose` (avec espace) = nouvelle version intégrée dans Docker CLI (v2)
>
> Utilise `docker compose` (v2), c'est le standard actuel.

---

## Nettoyage

Les images, conteneurs et volumes s'accumulent vite et prennent de l'espace disque.

```bash
# Supprimer les conteneurs arrêtés
docker container prune

# Supprimer les images non utilisées
docker image prune
docker image prune -a           # TOUTES les images non utilisées par un conteneur

# Supprimer les volumes orphelins
docker volume prune

# TOUT nettoyer d'un coup (attention !)
docker system prune             # Conteneurs arrêtés + images sans tag + réseaux
docker system prune -a          # + toutes les images non utilisées
docker system prune -a --volumes  # + les volumes

# Voir l'espace utilisé
docker system df
```

> [!warning] `docker system prune -a --volumes`
> Cette commande supprime **tout** : conteneurs arrêtés, toutes les images, tous les volumes. Utilise-la seulement si tu veux vraiment repartir de zéro.

---

## Carte mentale Docker

```
                         DOCKER
                           |
           ┌───────────────┼───────────────┐
           |               |               |
        IMAGES         CONTENEURS       VOLUMES
           |               |               |
     ┌─────┼─────┐   ┌────┼────┐     ┌────┼────┐
     |     |     |   |    |    |     |    |    |
   pull  build  rmi  run  stop  rm  bind  vol  prune
     |     |         |
   tag  Dockerfile  -it -d -p -v
     |     |
   push  .dockerignore
```

### Règles essentielles

> [!tip] Les 10 règles Docker
> 1. **Image = statique**, Conteneur = dynamique
> 2. **`stop` ≠ `rm`** : stop arrête, rm supprime
> 3. Toujours donner un **`--name`** à tes conteneurs
> 4. Utiliser **`--rm`** pour les conteneurs temporaires
> 5. Spécifier un **tag** pour les images (`ubuntu:22.04`, pas `ubuntu`)
> 6. Mettre les instructions stables **en haut** du Dockerfile (cache)
> 7. Utiliser **`.dockerignore`** pour ne pas copier de fichiers inutiles
> 8. **Un conteneur = un processus** (pas un serveur avec 10 services)
> 9. Utiliser **Docker Compose** pour les applications multi-conteneurs
> 10. **Nettoyer régulièrement** : `docker system prune`

---

## Exercices

### Exercice 1 : Premier conteneur

1. Télécharge l'image `ubuntu:22.04`
2. Lance un conteneur interactif
3. Dans le conteneur, crée un fichier `hello.txt`
4. Quitte le conteneur
5. Vérifie qu'il apparaît dans `docker ps -a`
6. Redémarre-le et vérifie que `hello.txt` existe toujours
7. Supprime le conteneur

### Exercice 2 : Dockerfile pour un projet C

Crée un Dockerfile qui :
1. Part de `ubuntu:22.04`
2. Installe `gcc` et `make`
3. Copie un fichier `main.c`
4. Le compile
5. Le lance au démarrage

### Exercice 3 : Docker Compose

Crée un `docker-compose.yml` avec deux services :
1. Un service `build` qui compile ton projet C
2. Un service `test` qui lance Valgrind sur l'exécutable
