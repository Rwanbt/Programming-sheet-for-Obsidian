# Git et GitHub - Le Contrôle de Version

## Qu'est-ce que Git ? Qu'est-ce que GitHub ?

**Git** est un **système de contrôle de version distribué**. Il enregistre l'historique de toutes les modifications de tes fichiers, te permet de revenir en arrière, de travailler en parallèle sur différentes fonctionnalités, et de collaborer avec d'autres développeurs.

**GitHub** est une **plateforme en ligne** qui héberge des dépôts Git. C'est un service cloud pour stocker et partager ton code.

> [!tip] Analogie Gaming
> - **Git** = le système de **sauvegarde** de ton jeu. Tu peux créer des points de sauvegarde (commits), charger une ancienne sauvegarde (checkout), avoir plusieurs sauvegardes parallèles (branches).
> - **GitHub** = le **cloud save**. Tes sauvegardes sont aussi stockées en ligne, accessibles depuis n'importe où, et tu peux les partager avec d'autres joueurs.
> - **Commit** = un point de sauvegarde avec une description ("Après avoir battu le boss du niveau 3")
> - **Branch** = une sauvegarde alternative ("Et si j'avais choisi l'autre chemin ?")
> - **Merge** = fusionner deux parties pour combiner le meilleur des deux chemins
> - **Push** = envoyer ta sauvegarde locale vers le cloud
> - **Pull** = télécharger la dernière sauvegarde depuis le cloud

---

## Le flux de travail Git

```
 Working Directory  →  Staging Area  →  Local Repo  →  Remote Repo
   (tes fichiers)      (git add)       (git commit)    (git push)
                                                    ←
                                                   (git pull)
```

| Zone | Description |
|---|---|
| **Working Directory** | Tes fichiers tels que tu les vois dans ton éditeur |
| **Staging Area** (Index) | La "zone de préparation" : les fichiers que tu as marqués pour le prochain commit |
| **Local Repository** | L'historique complet sur ta machine (dossier `.git`) |
| **Remote Repository** | L'historique sur GitHub (le cloud) |

> [!info] La Staging Area est le concept clé
> Contrairement à un simple Ctrl+S, Git te permet de **choisir** quels fichiers inclure dans chaque commit. C'est comme préparer un colis : tu mets dedans exactement ce que tu veux envoyer.

---

## Installation et première configuration

### Installation

```bash
# Ubuntu / Debian
sudo apt update
sudo apt install git

# Vérification
git --version
```

### Configuration initiale (à faire une seule fois)

```bash
# Ton identité (OBLIGATOIRE)
git config --global user.name "Ton Nom"
git config --global user.email "ton.email@example.com"

# Éditeur par défaut
git config --global core.editor "code --wait"   # VS Code
# ou
git config --global core.editor "nano"           # Nano

# Branche par défaut
git config --global init.defaultBranch main

# Vérifier ta config
git config --list
```

> [!warning] L'email doit correspondre à ton compte GitHub
> Si tu utilises un email différent, tes commits ne seront pas liés à ton profil GitHub.

---

## Créer un dépôt

### Méthode 1 : Initialiser un nouveau dépôt

```bash
mkdir mon-projet
cd mon-projet
git init
```

Cela crée un dossier caché `.git/` qui contient tout l'historique.

### Méthode 2 : Cloner un dépôt existant

```bash
git clone https://github.com/user/repo.git
cd repo
```

### Méthode 3 : Connecter un dossier existant à GitHub

```bash
cd mon-projet
git init
git remote add origin https://github.com/user/mon-projet.git
git branch -M main
git push -u origin main
```

| Commande | Description |
|---|---|
| `git remote add origin URL` | Associer le remote "origin" à l'URL |
| `git remote -v` | Voir les remotes configurés |
| `git remote remove origin` | Supprimer le remote |

---

## Le workflow quotidien

### 1. Vérifier l'état

```bash
git status
```

```
On branch main
Changes not staged for commit:
  modified:   main.c

Untracked files:
  utils.c
```

- **modified** : fichier modifié mais pas encore ajouté au staging
- **Untracked** : nouveau fichier que Git ne suit pas encore

### 2. Voir les modifications

```bash
git diff                    # Différences non stagées
git diff --staged           # Différences stagées (prêtes à commit)
git diff main..feature      # Différences entre deux branches
```

### 3. Ajouter au staging

```bash
git add main.c              # Ajouter un fichier spécifique
git add main.c utils.c      # Ajouter plusieurs fichiers
git add *.c                 # Ajouter tous les fichiers .c
git add .                   # Ajouter TOUT (attention !)
```

> [!warning] `git add .` ajoute TOUT
> Fais toujours un `git status` avant pour vérifier que tu n'ajoutes pas de fichiers indésirables (.env, binaires, etc.).

### 4. Créer un commit

```bash
git commit -m "Add user authentication feature"
```

### 5. Envoyer sur GitHub

```bash
git push                    # Si la branche est déjà liée au remote
git push -u origin main     # Première fois : lier et pousser
```

### 6. Récupérer les modifications

```bash
git pull                    # Télécharger ET fusionner
git fetch                   # Télécharger SANS fusionner (plus sûr)
```

> [!info] `pull` = `fetch` + `merge`
> `git fetch` télécharge les modifications mais ne modifie pas tes fichiers. Tu peux inspecter avant de fusionner. `git pull` fait les deux d'un coup.

---

## Conventional Commits

Les **Conventional Commits** sont un standard pour écrire des messages de commit clairs et cohérents.

### Format

```
type(scope): description courte

Corps optionnel avec plus de détails.
```

### Les types

| Type | Quand l'utiliser | Exemple |
|---|---|---|
| `feat` | Nouvelle fonctionnalité | `feat: add login page` |
| `fix` | Correction de bug | `fix: resolve null pointer in parser` |
| `docs` | Documentation | `docs: update README with install steps` |
| `style` | Formatage (pas de changement de logique) | `style: fix indentation in main.c` |
| `refactor` | Refactoring (pas de nouvelle fonctionnalité ni fix) | `refactor: extract validation into helper` |
| `test` | Ajout/modification de tests | `test: add unit tests for linked list` |
| `chore` | Maintenance (build, CI, deps) | `chore: update Makefile` |
| `perf` | Amélioration de performance | `perf: optimize sort algorithm` |

### Exemples

```bash
git commit -m "feat: add user registration endpoint"
git commit -m "fix: handle empty string in parser"
git commit -m "docs: add API documentation"
git commit -m "refactor: split main into modules"
git commit -m "chore: add .gitignore for build files"
```

> [!tip] Règles d'or pour les messages de commit
> - Utiliser l'**impératif** : "add" pas "added" ni "adding"
> - Première ligne : **max 72 caractères**
> - Décrire le **pourquoi**, pas le **quoi** (le diff montre le quoi)
> - Un commit = **une** modification logique

---

## Git log - Explorer l'historique

```bash
git log                         # Historique complet
git log --oneline               # Une ligne par commit
git log --oneline -5            # Les 5 derniers commits
git log --graph --oneline       # Avec graphe des branches
git log --author="Alice"        # Commits d'un auteur
git log --since="2024-01-01"    # Depuis une date
git log -- main.c               # Historique d'un fichier
git log -p                      # Avec les diffs
```

### Exemple de sortie

```bash
git log --oneline --graph
```

```
* a1b2c3d (HEAD -> main) feat: add user validation
* e4f5g6h fix: handle NULL input
* i7j8k9l refactor: extract parser module
|\
| * m0n1o2p feat: add logging
|/
* q3r4s5t chore: initial commit
```

---

## .gitignore

Le fichier `.gitignore` dit à Git quels fichiers **ignorer** (ne pas suivre).

### Créer un .gitignore

```bash
# Créer à la racine du projet
touch .gitignore
```

### Que mettre dedans

```gitignore
# Fichiers compilés
*.o
*.out
a.out

# Exécutables du projet
prog
my_program

# Fichiers d'éditeur
*.swp
*.swo
*~
.vscode/
.idea/

# Fichiers système
.DS_Store
Thumbs.db

# Secrets et config locale
.env
*.key
credentials.json

# Dossiers de build
build/
dist/
node_modules/

# Fichiers Valgrind
vgcore.*
valgrind.log
```

> [!warning] Ajoute `.gitignore` AVANT ton premier commit
> Si tu ajoutes un fichier dans `.gitignore` alors qu'il est déjà suivi par Git, il faut le retirer du tracking :
> ```bash
> git rm --cached fichier_a_ignorer
> ```

---

## Branches

### Concept

Une branche est une **ligne de développement indépendante**. Tu peux travailler sur une fonctionnalité sans affecter le code principal.

```
main     ──●──●──●──●──●──●
                \         /
feature   ──────●──●──●──
```

### Commandes de base

```bash
# Créer une branche
git branch feature-login

# Changer de branche
git switch feature-login
# ou (ancien style)
git checkout feature-login

# Créer ET changer en une commande
git switch -c feature-login
# ou
git checkout -b feature-login

# Lister les branches
git branch              # Branches locales
git branch -a           # Toutes (locales + remote)

# Renommer une branche
git branch -m ancien-nom nouveau-nom

# Supprimer une branche
git branch -d feature-login       # Seulement si déjà fusionnée
git branch -D feature-login       # Forcer la suppression

# Pousser une branche sur GitHub
git push -u origin feature-login
```

### Conventions de nommage

| Préfixe | Usage | Exemple |
|---|---|---|
| `feature/` | Nouvelle fonctionnalité | `feature/user-auth` |
| `fix/` | Correction de bug | `fix/null-pointer-crash` |
| `hotfix/` | Correction urgente | `hotfix/security-patch` |
| `refactor/` | Refactoring | `refactor/database-layer` |
| `docs/` | Documentation | `docs/api-reference` |
| `test/` | Tests | `test/unit-tests` |

---

## Merge (Fusion)

### Fast-forward merge

Quand la branche `main` n'a **pas bougé** depuis la création de la branche feature :

```
AVANT :
main     ──●──●
                \
feature         ●──●──●

APRÈS (fast-forward) :
main     ──●──●──●──●──●
                         (feature pointe ici aussi)
```

```bash
git switch main
git merge feature-login
```

Git avance simplement le pointeur `main`. Pas de commit de merge.

### Merge avec commit (--no-ff)

Force un **commit de merge** même si un fast-forward est possible. Cela préserve l'historique de la branche.

```
AVANT :
main     ──●──●
                \
feature         ●──●──●

APRÈS (--no-ff) :
main     ──●──●────────●  (commit de merge)
                \      /
feature         ●──●──●
```

```bash
git switch main
git merge --no-ff feature-login
```

> [!tip] Quand utiliser `--no-ff` ?
> - En **équipe** : toujours `--no-ff` pour garder la trace de quelle branche a apporté quoi
> - **Seul** sur un petit projet : le fast-forward suffit

---

## Résolution de conflits

Un conflit arrive quand **deux branches ont modifié la même ligne** du même fichier.

### Les 4 étapes

**Étape 1** : Git te prévient

```bash
git merge feature
# CONFLICT (content): Merge conflict in main.c
# Automatic merge failed; fix conflicts and then commit the result.
```

**Étape 2** : Ouvrir le fichier en conflit

Git insère des marqueurs dans le fichier :

```c
<<<<<<< HEAD
    printf("Version de main\n");
=======
    printf("Version de feature\n");
>>>>>>> feature
```

| Marqueur | Signification |
|---|---|
| `<<<<<<< HEAD` | Début de la version actuelle (ta branche) |
| `=======` | Séparation |
| `>>>>>>> feature` | Fin de la version entrante (l'autre branche) |

**Étape 3** : Résoudre manuellement

Choisis la bonne version (ou combine les deux) et supprime les marqueurs :

```c
    printf("Version fusionnée\n");
```

**Étape 4** : Marquer comme résolu et commit

```bash
git add main.c
git commit -m "fix: resolve merge conflict in main.c"
```

> [!warning] Ne laisse JAMAIS les marqueurs `<<<<<<<` dans ton code !
> Git add + commit = "j'ai résolu ce conflit". Si tu laisses les marqueurs, ton code ne compilera pas.

---

## Revenir en arrière

### `git reset` - Réécrire l'historique

> [!warning] `reset` modifie l'historique. Ne l'utilise **JAMAIS** sur des commits déjà poussés et partagés.

| Mode | Ce qui se passe | Commande |
|---|---|---|
| `--soft` | Annule le commit, garde les fichiers en staging | `git reset --soft HEAD~1` |
| `--mixed` | Annule le commit, remet les fichiers en working dir (défaut) | `git reset HEAD~1` |
| `--hard` | Annule le commit, **supprime** toutes les modifications | `git reset --hard HEAD~1` |

```
HEAD~1 = un commit en arrière
HEAD~3 = trois commits en arrière
```

**Schéma** :

```
                        --soft          --mixed         --hard
Commit annulé             ✓               ✓               ✓
Fichiers en staging       ✓               ✗               ✗
Fichiers en working dir   ✓               ✓               ✗
```

### `git revert` - Annuler proprement

`revert` crée un **nouveau commit** qui annule les changements d'un ancien commit. L'historique est préservé.

```bash
git revert abc1234       # Annule le commit abc1234
git revert HEAD          # Annule le dernier commit
```

> [!tip] Quand utiliser reset vs revert ?
> - **reset** : code **pas encore poussé** (en local uniquement). Tu veux "effacer" comme si le commit n'avait jamais existé.
> - **revert** : code **déjà poussé et partagé**. Tu veux annuler proprement sans casser l'historique des autres.

---

## Autres commandes utiles

### `git stash` - Mettre de côté temporairement

Tu travailles sur quelque chose mais tu dois changer de branche en urgence.

```bash
git stash                    # Mettre de côté les modifications
git stash list               # Voir les stash sauvegardés
git stash pop                # Récupérer le dernier stash
git stash drop               # Supprimer le dernier stash
git stash apply              # Appliquer sans supprimer
```

### `git restore` - Annuler des modifications

```bash
git restore main.c           # Annuler les modifications non stagées
git restore --staged main.c  # Retirer du staging (unstage)
```

### `git tag` - Marquer une version

```bash
git tag v1.0.0                        # Tag léger
git tag -a v1.0.0 -m "Version 1.0"   # Tag annoté
git push origin v1.0.0                # Pousser un tag
git push origin --tags                # Pousser tous les tags
```

### `git cherry-pick` - Copier un commit

```bash
git cherry-pick abc1234       # Copier le commit abc1234 sur la branche actuelle
```

---

## Collaboration

### Ajouter des collaborateurs sur GitHub

1. Aller dans **Settings** → **Collaborators**
2. Cliquer **Add people**
3. Entrer le nom d'utilisateur GitHub

### Chaque personne sur sa branche

```bash
# Alice
git switch -c feature/auth
# travaille, commit, push
git push -u origin feature/auth

# Bob
git switch -c feature/api
# travaille, commit, push
git push -u origin feature/api
```

Puis chacun fait une **Pull Request** pour fusionner dans `main`.

---

## GitHub Flow : Le workflow professionnel

### Les 6 étapes

```
1. Créer une branche
        ↓
2. Faire des commits
        ↓
3. Pousser la branche
        ↓
4. Ouvrir une Pull Request
        ↓
5. Code review + discussion
        ↓
6. Merge dans main
```

**Étape 1** : Créer une branche depuis `main`
```bash
git switch main
git pull
git switch -c feature/login-page
```

**Étape 2** : Travailler et committer
```bash
# ... modifications ...
git add login.c login.h
git commit -m "feat: add login page skeleton"
# ... plus de modifications ...
git commit -m "feat: add form validation"
```

**Étape 3** : Pousser la branche
```bash
git push -u origin feature/login-page
```

**Étape 4** : Ouvrir une Pull Request sur GitHub
- Aller sur GitHub → bouton "Compare & pull request"
- Écrire un titre clair et une description
- Assigner des reviewers

**Étape 5** : Code review
- Les reviewers commentent le code
- Tu fais les corrections demandées et push à nouveau

**Étape 6** : Merge
- Une fois approuvée, cliquer "Merge pull request"
- Supprimer la branche feature

```bash
# Après le merge, en local :
git switch main
git pull
git branch -d feature/login-page
```

---

## Référence complète des commandes

| Catégorie | Commande | Description |
|---|---|---|
| **Config** | `git config --global user.name "Nom"` | Définir ton nom |
| | `git config --global user.email "email"` | Définir ton email |
| | `git config --list` | Voir la configuration |
| **Créer** | `git init` | Initialiser un dépôt |
| | `git clone URL` | Cloner un dépôt distant |
| **Quotidien** | `git status` | Voir l'état des fichiers |
| | `git diff` | Voir les modifications |
| | `git add fichier` | Ajouter au staging |
| | `git commit -m "msg"` | Créer un commit |
| | `git push` | Envoyer sur le remote |
| | `git pull` | Récupérer depuis le remote |
| | `git fetch` | Télécharger sans fusionner |
| **Branches** | `git branch` | Lister les branches |
| | `git switch -c nom` | Créer et changer de branche |
| | `git switch nom` | Changer de branche |
| | `git branch -d nom` | Supprimer une branche |
| | `git merge branche` | Fusionner une branche |
| | `git merge --no-ff branche` | Fusionner avec commit de merge |
| **Historique** | `git log --oneline` | Historique condensé |
| | `git log --graph` | Historique avec graphe |
| | `git show abc123` | Détails d'un commit |
| **Annuler** | `git restore fichier` | Annuler les modifications |
| | `git restore --staged fichier` | Unstage un fichier |
| | `git reset --soft HEAD~1` | Annuler commit, garder staging |
| | `git reset HEAD~1` | Annuler commit, garder fichiers |
| | `git reset --hard HEAD~1` | Annuler commit, tout supprimer |
| | `git revert abc123` | Créer un commit d'annulation |
| **Divers** | `git stash` | Mettre de côté |
| | `git stash pop` | Récupérer le stash |
| | `git tag v1.0` | Créer un tag |
| | `git cherry-pick abc123` | Copier un commit |
| | `git remote -v` | Voir les remotes |

---

## Scénarios de panique et solutions

### "J'ai commit sur la mauvaise branche !"

```bash
# 1. Sauvegarder le commit
git log --oneline -1          # Noter le hash (ex: abc1234)

# 2. Annuler le commit sur la mauvaise branche
git reset --soft HEAD~1

# 3. Stash les modifications
git stash

# 4. Aller sur la bonne branche
git switch bonne-branche

# 5. Récupérer les modifications
git stash pop
git add .
git commit -m "feat: le bon message"
```

### "J'ai écrit un mauvais message de commit !"

```bash
# Si c'est le DERNIER commit et PAS encore poussé :
git commit --amend -m "Le bon message"
```

> [!warning] Ne fais **jamais** `--amend` sur un commit déjà poussé ! Utilise `revert` + nouveau commit à la place.

### "Mon push est rejeté !"

```
! [rejected]  main -> main (fetch first)
error: failed to push some refs
```

```bash
# Quelqu'un a poussé avant toi. Solution :
git pull --rebase         # Récupérer + rebase tes commits par-dessus
git push                  # Maintenant ça passe
```

### "J'ai supprimé un fichier par accident !"

```bash
# Si pas encore commité :
git restore fichier_supprime.c

# Si déjà commité, récupérer depuis un ancien commit :
git checkout HEAD~1 -- fichier_supprime.c
```

### "J'ai commité un secret (.env, clé API) !"

> [!warning] URGENCE : même si tu supprimes le fichier dans un nouveau commit, il reste dans l'historique !

```bash
# 1. Ajouter au .gitignore immédiatement
echo ".env" >> .gitignore

# 2. Retirer du tracking
git rm --cached .env

# 3. Commit
git commit -m "chore: remove secret file from tracking"

# 4. Changer TOUS tes mots de passe / clés API
# L'historique Git garde tout. Si c'est public, c'est compromis.
```

Pour vraiment supprimer de l'historique, il faut des outils comme `git filter-branch` ou `BFG Repo Cleaner`, mais c'est complexe et dangereux.

---

## Gerer les fichiers non suivis dans VS Code

Quand des fichiers apparaissent en **rouge** avec un **U** dans VS Code, cela signifie qu'ils sont **Untracked** : Git a detecte de nouveaux fichiers qui ne font pas encore partie de l'historique.

### Lexique visuel de VS Code

| Lettre | Couleur | Signification |
|---|---|---|
| **U** | Rouge/Gris | Untracked (nouveau fichier, pas encore suivi) |
| **M** | Jaune/Orange | Modified (fichier connu de Git, modifie) |
| **A** | Vert | Added (fichier pret a etre commite) |
| **D** | Rouge | Deleted (fichier supprime) |

### Cas 1 : Sauvegarder ces fichiers (le plus courant)

1. Aller dans l'onglet **Source Control** (`Ctrl+Maj+G`)
2. Les fichiers Untracked apparaissent sous "Changes"
3. Cliquer sur le **+** a cote du fichier (ou a cote de "Changes" pour tout ajouter)
4. Les fichiers passent en vert avec un **A** (Added)
5. Taper un message de commit et cliquer **Commit**

### Cas 2 : Ignorer ces fichiers (`.gitignore`)

Si ce sont des fichiers temporaires, des executables ou des dossiers de dependances :

```bash
# Ajouter au .gitignore a la racine du projet
*.exe
*.o
a.out
node_modules/
```

> [!tip] Regle d'or en C
> Ne **jamais** commiter les fichiers executables (compiles). Les ajouter systematiquement au `.gitignore`.

### Cas 3 : Supprimer les fichiers non voulus

```bash
# Attention : suppression definitive, pas de retour en arriere via Git
git clean -fd    # Supprime tous les fichiers non suivis
```

> [!warning] `git clean -fd` est irreversible
> Ne l'utiliser que si on est sur de ne pas avoir cree de nouveaux fichiers importants sans les avoir ajoutes au staging.

### Pourquoi tout devient rouge apres compilation ?

Si on compile des fichiers C avec `gcc`, les executables generes sont marques comme **Modified** ou **Untracked** par Git. C'est normal : meme un seul octet change dans un binaire et Git le detecte. La solution est d'ajouter les executables au `.gitignore`.

Si beaucoup de fichiers dans des dossiers parents apparaissent, c'est parce que Git surveille **tout l'arbre** depuis la racine du depot, meme si on est dans un sous-dossier. Pour annuler des modifications non voulues :

```bash
git restore ../              # Annuler les changements dans les dossiers parents
```

---

## Creer un profil GitHub professionnel (README)

GitHub permet de creer un **profil README** : un fichier `README.md` dans un depot qui porte le meme nom que ton username. Ce fichier s'affiche directement sur ta page de profil.

### Etapes de creation

1. Creer un nouveau depot sur GitHub avec **exactement** ton nom d'utilisateur (ex: `Rwanbt/Rwanbt`)
2. Cocher "Add a README file"
3. Editer le `README.md` avec le contenu souhaite

### Elements d'un profil premium

#### Texte anime (typing SVG)

```html
<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?size=25&duration=4000&color=2F80ED&center=true&vCenter=true&width=600&lines=Creative+Software+Developer;Holberton+School+Student;Design+%2B+Code+%2B+Sound" />
</p>
```

#### Badges de technologies (shields.io)

```markdown
![C](https://img.shields.io/badge/C-00599C?style=for-the-badge&logo=c&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
```

#### Statistiques GitHub

```html
<p align="center">
  <img src="https://github-readme-stats.vercel.app/api?username=TON_USERNAME&show_icons=true&hide_border=true&bg_color=00000000" alt="GitHub Stats" />
</p>
```

> [!tip] Si les stats ne s'affichent pas
> Les serveurs Vercel gratuits peuvent etre en pause. Utiliser un miroir alternatif :
> ```
> https://github-readme-stats.zcy.dev/api?username=TON_USERNAME&show_icons=true&hide_border=true&bg_color=00000000
> ```

#### Structure recommandee

```markdown
# Hi, I'm [Ton Nom]

## About Me
[Description courte de ton parcours et ta transition vers le dev]

## Current Focus
- [Technologies en cours d'apprentissage]

## Tech Stack
### Learning
[Badges des langages en cours]

### Tools
[Badges des outils maitrises]

## GitHub Stats
[Widget de statistiques]

## Vision
[Ta vision professionnelle en une phrase]
```

> [!info] Bonnes pratiques
> - Garder le README **concis** et **visuel** (pas un mur de texte)
> - Mettre en avant ses **projets concrets** plutot que de lister des competences theoriques
> - Ajouter des liens vers LinkedIn, portfolio, etc.
> - Mettre a jour regulierement au fil de la progression

---

## Checklist Pro

> [!tip] Checklist avant chaque session de travail
> - [ ] `git pull` sur `main` pour être à jour
> - [ ] Créer une branche pour ta feature : `git switch -c feature/...`
> - [ ] Travailler, commiter régulièrement avec des messages clairs
> - [ ] `git status` avant chaque commit pour vérifier
> - [ ] Vérifier le `.gitignore` (pas de secrets, pas de binaires)
> - [ ] Pousser ta branche : `git push -u origin feature/...`
> - [ ] Ouvrir une Pull Request sur GitHub
> - [ ] Après merge : supprimer la branche et pull `main`

---

## Exercices

### Exercice 1 : Premier dépôt

1. Crée un dossier `mon-premier-repo`
2. Initialise Git
3. Crée un fichier `hello.c` avec un Hello World
4. Fais ton premier commit
5. Crée un dépôt sur GitHub et pousse ton code

### Exercice 2 : Branches et merge

1. Crée une branche `feature/goodbye`
2. Ajoute une fonction `goodbye()` dans un nouveau fichier
3. Fais un commit
4. Retourne sur `main`
5. Fais un merge avec `--no-ff`

### Exercice 3 : Résoudre un conflit

1. Sur `main`, modifie la ligne 5 de `hello.c`
2. Crée une branche `test-conflict`, modifie aussi la ligne 5 différemment
3. Essaie de merger → conflit !
4. Résous le conflit manuellement
5. Commite la résolution
