# Les Terminaux : Guide Complet

---

## Introduction

Ce guide couvre **tout** ce qu'il faut savoir sur les terminaux : qu'est-ce qu'un terminal, quels shells existent, quels outils utiliser selon ton OS, et comment choisir le bon setup pour ton workflow de developpement.

> [!info] Prerequis
> Aucun prerequis technique. Ce guide part de zero. Si tu veux apprendre les commandes elles-memes, consulte [[01 - Shell et Commandes Linux]].

---

## 1. Qu'est-ce qu'un terminal ?

### 1.1 Definitions : Terminal vs Shell vs Console vs Emulateur

Ces termes sont souvent utilises de maniere interchangeable, mais ils designent des choses **differentes**.

| Terme | Definition | Exemple |
|---|---|---|
| **Terminal** | Historiquement, un appareil physique (ecran + clavier) connecte a un ordinateur | VT100, Teletype |
| **Emulateur de terminal** | Un logiciel qui **simule** un terminal physique | GNOME Terminal, iTerm2, Windows Terminal |
| **Shell** | Le **programme** qui interprete tes commandes | bash, zsh, fish, PowerShell |
| **Console** | Le terminal physique de la machine (ecran + clavier directement connectes) | TTY1-TTY6 sur Linux |

> [!tip] Analogie simple
> L'emulateur de terminal, c'est la **fenetre**. Le shell, c'est le **programme** qui tourne a l'interieur.
> Tu peux changer de shell sans changer de terminal, et vice versa.

### 1.2 Diagramme : Comment ca s'emboite

```
+-------------------------------------------------------------------+
|                         UTILISATEUR                                |
|                    (tape des commandes)                            |
+-------------------------------------------------------------------+
                             |
                             v
+-------------------------------------------------------------------+
|                  EMULATEUR DE TERMINAL                             |
|          (Windows Terminal, GNOME Terminal, iTerm2)                |
|                                                                   |
|  - Affiche le texte (rendu graphique)                             |
|  - Capture les frappes clavier                                    |
|  - Gere les couleurs, polices, onglets                            |
+-------------------------------------------------------------------+
                             |
                             | (stdin / stdout / stderr)
                             v
+-------------------------------------------------------------------+
|                          SHELL                                     |
|               (bash, zsh, fish, PowerShell)                       |
|                                                                   |
|  - Interprete les commandes                                       |
|  - Gere les variables, redirections, pipes                        |
|  - Execute les programmes                                         |
+-------------------------------------------------------------------+
                             |
                             | (appels systeme : fork, exec, read, write)
                             v
+-------------------------------------------------------------------+
|                         KERNEL                                     |
|                (Linux, Windows NT, macOS/XNU)                     |
|                                                                   |
|  - Gere les processus                                             |
|  - Gere le systeme de fichiers                                    |
|  - Gere le materiel                                               |
+-------------------------------------------------------------------+
                             |
                             v
+-------------------------------------------------------------------+
|                        HARDWARE                                    |
|            (CPU, RAM, disque, peripheriques)                       |
+-------------------------------------------------------------------+
```

### 1.3 Le flux concret d'une commande

Quand tu tapes `ls -la` et appuies sur Entree :

```
1. L'emulateur de terminal capture "ls -la\n"
2. Il envoie ces caracteres au shell via stdin
3. Le shell parse la commande :
   - Programme : "ls"
   - Arguments : ["-la"]
4. Le shell fait un fork() + exec() pour lancer /usr/bin/ls
5. ls fait des appels systeme (opendir, readdir, stat)
6. Le kernel accede au systeme de fichiers sur le disque
7. ls ecrit le resultat sur stdout
8. Le shell transmet stdout a l'emulateur
9. L'emulateur affiche le texte a l'ecran
```

---

## 2. L'histoire des terminaux

### 2.1 Les TTY physiques (1960-1970)

```
+------------------+          cable serie          +------------------+
|   TELETYPE       | ============================> |   ORDINATEUR     |
|   (TTY)          |                               |   CENTRAL        |
|                  | <============================ |   (mainframe)    |
|  [clavier]       |                               |                  |
|  [imprimante]    |                               |                  |
+------------------+                               +------------------+
```

- **Teletype (TTY)** : machine a ecrire connectee a un ordinateur
- Pas d'ecran ! Le resultat etait **imprime sur papier**
- C'est pourquoi on dit encore "print" en programmation
- Le terme **TTY** est reste dans Linux (`/dev/tty0`, commande `tty`)

### 2.2 Les terminaux video (1970-1990)

```
+------------------+
|   VT100 (1978)   |
|                  |
|  +-----------+   |
|  |           |   |
|  |  ecran    |   |
|  |  CRT      |   |
|  |           |   |
|  +-----------+   |
|  [  clavier  ]   |
+------------------+
```

- **VT100** de DEC (1978) : le terminal de reference
- Introduction des **codes d'echappement ANSI** (couleurs, positionnement du curseur)
- Ces codes sont encore utilises aujourd'hui : `\033[31m` = rouge

### 2.3 Les terminaux virtuels (1990-present)

- Avec les interfaces graphiques (GUI), plus besoin de terminaux physiques
- Les **emulateurs de terminal** simulent un terminal dans une fenetre
- Linux conserve des TTY virtuels (Ctrl+Alt+F1 a F6)
- On a aujourd'hui des terminaux GPU-acceleres (Alacritty, Kitty)

```
Timeline :
1960     1970      1980       1990       2000       2010       2020
  |        |         |          |          |          |          |
  TTY    VT100    xterm     GNOME       iTerm    Hyper     Alacritty
 papier  ecran    X Window  Terminal     macOS    Electron   GPU
```

---

## 3. Les Shells disponibles

### 3.1 Bash (Bourne Again Shell)

> [!info] Le standard
> Bash est le shell par defaut sur la plupart des distributions Linux. C'est **le** shell a connaitre en priorite.

**Origine** : Cree en 1989 par Brian Fox pour le projet GNU, en remplacement du Bourne Shell (sh).

**Fichiers de configuration** :

```
+-------------------------------------------------------+
|  Login shell              Non-login shell              |
|  (ssh, tty, su -)        (ouvrir un terminal)         |
|                                                        |
|  /etc/profile             /etc/bash.bashrc             |
|       |                        |                       |
|  ~/.bash_profile          ~/.bashrc                    |
|  (ou ~/.bash_login)                                    |
|  (ou ~/.profile)                                       |
+-------------------------------------------------------+
```

| Fichier | Quand il est lu | Usage |
|---|---|---|
| `/etc/profile` | Login shell (systeme) | Config globale |
| `~/.bash_profile` | Login shell (user) | Variables d'env, PATH |
| `~/.bashrc` | Shell interactif non-login | Aliases, prompt, fonctions |
| `~/.bash_logout` | Fermeture d'un login shell | Nettoyage |

> [!warning] Piege classique
> Sur beaucoup de systemes, `.bash_profile` **source** `.bashrc` pour que les aliases soient disponibles partout. Si tes aliases ne marchent pas en SSH, verifie ca !

**Exemple de `.bashrc`** :

```bash
# Aliases utiles
alias ll='ls -la'
alias gs='git status'
alias gc='git commit'
alias gp='git push'

# Prompt personnalise avec couleurs et branche git
parse_git_branch() {
    git branch 2>/dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
export PS1="\[\033[32m\]\u@\h\[\033[00m\]:\[\033[34m\]\w\[\033[33m\]\$(parse_git_branch)\[\033[00m\]\$ "

# PATH
export PATH="$HOME/.local/bin:$PATH"
```

**Forces** :
- Disponible partout (Linux, macOS, WSL, Git Bash)
- Enorme documentation et communaute
- Standard pour les scripts systeme
- Compatible POSIX (en mode `--posix`)

**Faiblesses** :
- Autocompletion basique comparee a zsh/fish
- Syntaxe parfois archaique (`[[ ]]` vs `[ ]`, `$(( ))`)
- Pas de suggestions en temps reel

### 3.2 Zsh (Z Shell)

> [!info] Le shell moderne
> Zsh est le shell par defaut sur macOS depuis Catalina (2019). Il est compatible bash a 95% mais avec plein d'ameliorations.

**Ameliorations vs Bash** :

| Feature | Bash | Zsh |
|---|---|---|
| Autocompletion | Basique (Tab) | Avancee (Tab + menu interactif) |
| Correction ortho | Non | `setopt CORRECT` |
| Globbing etendu | Limite | `**/*.c` natif, qualificateurs |
| Themes de prompt | Manuel | Oh My Zsh themes |
| Partage d'historique | Par shell | Entre tous les shells |
| Plugins | Non | Framework de plugins |

**Oh My Zsh** : Le framework qui rend Zsh incontournable.

```bash
# Installation
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

**Plugins populaires** :

```bash
# Dans ~/.zshrc
plugins=(
    git                 # Aliases git (gst, gco, gcm, gp...)
    zsh-autosuggestions # Suggestions basees sur l'historique
    zsh-syntax-highlighting  # Coloration syntaxique en temps reel
    z                   # Navigation rapide (z project → cd ~/dev/project)
    docker              # Autocompletion Docker
    kubectl             # Autocompletion Kubernetes
)
```

**Themes populaires** : `robbyrussell` (defaut), `agnoster`, `powerlevel10k` (le plus puissant).

### 3.3 Fish (Friendly Interactive Shell)

> [!warning] Pas POSIX-compatible
> La syntaxe de Fish est differente de bash/zsh. Les scripts bash ne fonctionnent **pas** dans Fish. C'est un shell interactif, pas un shell de scripting.

**Ce qui rend Fish unique** :
- Autosuggestions en temps reel (basees sur l'historique et les man pages)
- Coloration syntaxique **native** (pas besoin de plugin)
- Interface web de configuration : `fish_config`
- Aide inline : quand tu tapes une commande, Fish te montre la doc

```bash
# Syntaxe Fish (differente de bash !)
# Variables
set myvar "hello"            # pas de $, pas de =

# Conditions
if test -f /etc/passwd
    echo "fichier existe"
end                          # pas de fi, mais end

# Boucles
for f in *.txt
    echo $f
end

# Fonctions
function greet
    echo "Hello, $argv[1]"
end
```

### 3.4 sh (Bourne Shell) et dash

**sh (Bourne Shell)** :
- Cree par Stephen Bourne aux Bell Labs (1979)
- Le shell **original** d'Unix
- Sur la plupart des systemes modernes, `/bin/sh` pointe vers bash ou dash
- Standard POSIX : les scripts ecrits pour `sh` sont les plus portables

**dash (Debian Almquist Shell)** :
- Implementation minimaliste de sh
- **Beaucoup plus rapide** que bash pour executer des scripts
- Utilise comme `/bin/sh` sur Debian/Ubuntu
- Ne supporte PAS les bashismes (`[[ ]]`, `$(( ))`, arrays)

```bash
# Ce script fonctionne en sh/dash (POSIX)
#!/bin/sh
if [ -f "/etc/hostname" ]; then
    hostname=$(cat /etc/hostname)
    echo "Machine: $hostname"
fi

# Ce script NE fonctionne PAS en sh/dash (bashisme)
#!/bin/sh
array=(1 2 3)       # ERREUR : les arrays n'existent pas en sh
[[ $x == "test" ]]  # ERREUR : [[ ]] n'existe pas en sh
```

### 3.5 Tableau comparatif des shells

| | **bash** | **zsh** | **fish** | **sh/dash** |
|---|---|---|---|---|
| **POSIX** | Oui (mode posix) | Oui (quasi) | **Non** | Oui |
| **Autocompletion** | Basique | Avancee | Excellente | Aucune |
| **Suggestions** | Non | Plugin | **Native** | Non |
| **Scripting** | Excellent | Excellent | Syntaxe differente | Minimal mais portable |
| **Config** | `.bashrc` | `.zshrc` | `config.fish` | `/etc/profile` |
| **Par defaut sur** | Linux | macOS | Aucun | Scripts systeme |
| **Courbe** | Moyenne | Moyenne | Facile | Difficile |
| **Plugins** | Non | Oh My Zsh | Fisher / Oh My Fish | Non |

---

## 4. Les Terminaux sous Windows

### 4.1 CMD (Command Prompt)

> [!warning] Heritage DOS
> CMD est l'heritier de `COMMAND.COM` de MS-DOS (1981). Il est toujours present dans Windows mais ses capacites sont **tres limitees** comparees a un shell Unix.

**Commandes de base** :

| CMD | Equivalent Linux | Description |
|---|---|---|
| `dir` | `ls` | Lister les fichiers |
| `cd` | `cd` | Changer de repertoire |
| `cls` | `clear` | Effacer l'ecran |
| `copy` | `cp` | Copier un fichier |
| `move` | `mv` | Deplacer/renommer |
| `del` | `rm` | Supprimer un fichier |
| `type` | `cat` | Afficher le contenu |
| `echo` | `echo` | Afficher du texte |
| `mkdir` | `mkdir` | Creer un dossier |
| `rmdir /s` | `rm -r` | Supprimer un dossier |
| `findstr` | `grep` | Chercher du texte |
| `set` | `export` | Variable d'environnement |

**Limites de CMD** :
- Pas de pipe puissant (pas d'equivalent de `awk`, `sed`, `xargs`)
- Pas de scripting serieux (batch est tres limite)
- Pas d'autocompletion avancee
- Pas de gestion des couleurs native (avant Windows 10)
- Pas de SSH integre (avant Windows 10)

**Syntaxe batch** (fichiers `.bat` ou `.cmd`) :

```bash
@echo off
REM Ceci est un commentaire en batch
set NAME=John
echo Hello, %NAME%!

REM Boucle
for %%f in (*.txt) do (
    echo Fichier: %%f
)

REM Condition
if exist "myfile.txt" (
    echo Le fichier existe
) else (
    echo Le fichier n'existe pas
)
```

### 4.2 PowerShell

> [!tip] Le vrai shell de Windows
> PowerShell est un shell **oriente objet**. Quand tu fais un pipe, tu ne passes pas du texte brut comme en bash, mais des **objets .NET** avec des proprietes et des methodes.

**Paradigme objet : la difference fondamentale**

```
Bash (pipe texte) :
  ps aux | grep firefox | awk '{print $2}' | xargs kill
  ^^^^^^^^   ^^^^^^^^^^^   ^^^^^^^^^^^^^^^^   ^^^^^^^^^^
  texte       filtre        decoupe texte     action
  brut        regex         colonnes          sur texte

PowerShell (pipe objet) :
  Get-Process firefox | Stop-Process
  ^^^^^^^^^^^^^^^^^^^^^^^   ^^^^^^^^^^^^
  objets Process avec       methode Stop()
  .Name, .Id, .CPU...      sur chaque objet
```

**Cmdlets essentiels** :

```bash
# Lister les processus
Get-Process
Get-Process | Where-Object { $_.CPU -gt 100 }
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10

# Fichiers
Get-ChildItem -Recurse -Filter "*.log"
Get-Content myfile.txt | Select-String "error"

# Systeme
Get-Service | Where-Object { $_.Status -eq "Running" }
Get-ComputerInfo

# Reseau
Test-NetConnection google.com -Port 443
Invoke-WebRequest https://api.example.com/data | ConvertFrom-Json
```

**Profils PowerShell** :

```
$PROFILE                          # Affiche le chemin du profil
$PROFILE.CurrentUserCurrentHost   # ~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
```

**Quand utiliser PowerShell** :
- Administration systeme Windows
- Automatisation de taches Windows (Active Directory, Exchange, Azure)
- Quand tu as besoin de manipuler des objets structures (JSON, CSV, XML)
- Quand tu travailles avec des API REST

### 4.3 Git Bash

> [!info] Unix sur Windows (light)
> Git Bash est un emulateur de terminal fourni avec Git for Windows. Il est base sur **MinGW/MSYS2** et fournit un environnement bash minimal sur Windows.

**Ce qu'il inclut** :

```
Git Bash = MinGW/MSYS2 + bash + outils GNU
                                    |
                +-------------------+-------------------+
                |         |         |         |         |
              bash      grep      sed       awk       ssh
              git       find      sort      curl      scp
              ls        cat       head      tail      wc
              vim       less      diff      patch     tar
```

**Pourquoi Git Bash existe** :
- Git a ete cree pour Linux (par Linus Torvalds)
- Beaucoup de fonctionnalites de Git dependent d'outils Unix (diff, patch, ssh)
- Git Bash embarque ces outils pour que Git fonctionne sur Windows

**Limites (ce n'est PAS un vrai Linux)** :
- Pas de `apt`, `yum`, `pacman` (pas de gestionnaire de paquets complet)
- Pas de support complet des signaux Unix
- Performances reduites sur les operations filesystem (traduction de chemins)
- Pas de Docker, pas de systemd, pas de crontab
- Chemins Windows vs Unix : `/c/Users/john` au lieu de `C:\Users\john`

```bash
# Dans Git Bash, les chemins sont traduits
pwd
# /c/Users/john/Desktop

# Tu peux acceder aux lecteurs Windows avec /c/, /d/, etc.
ls /d/Projects/
```

### 4.4 WSL (Windows Subsystem for Linux)

> [!tip] Un vrai Linux dans Windows
> WSL est la solution recommandee pour faire du developpement Linux sur Windows. WSL2 execute un **vrai kernel Linux** dans une VM legere.

**WSL1 vs WSL2** :

```
+------------------------------------------------------------------+
|                         WSL1 (2016)                               |
|                                                                   |
|  +---------------------------+  +-----------------------------+   |
|  |      Application Linux    |  |     Application Linux       |   |
|  +---------------------------+  +-----------------------------+   |
|  +---------------------------+  +-----------------------------+   |
|  |     Appels systeme Linux  |  |    Appels systeme Linux     |   |
|  +---------------------------+  +-----------------------------+   |
|                    |                          |                    |
|                    v                          v                    |
|  +------------------------------------------------------------+  |
|  |           COUCHE DE TRADUCTION (lxcore.sys)                 |  |
|  |   Traduit les syscalls Linux → syscalls Windows NT          |  |
|  +------------------------------------------------------------+  |
|  +------------------------------------------------------------+  |
|  |              KERNEL WINDOWS NT                              |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+

+------------------------------------------------------------------+
|                         WSL2 (2019)                               |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |              VM LEGERE (Hyper-V)                             |  |
|  |                                                              |  |
|  |  +------------------------+  +--------------------------+   |  |
|  |  |   Application Linux    |  |    Application Linux     |   |  |
|  |  +------------------------+  +--------------------------+   |  |
|  |  +------------------------------------------------------+   |  |
|  |  |            VRAI KERNEL LINUX                          |   |  |
|  |  |      (compatibilite syscall 100%)                     |   |  |
|  |  +------------------------------------------------------+   |  |
|  +------------------------------------------------------------+  |
|  +------------------------------------------------------------+  |
|  |              KERNEL WINDOWS NT + Hyper-V                    |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

| | **WSL1** | **WSL2** |
|---|---|---|
| **Architecture** | Couche de traduction | VM avec vrai kernel |
| **Compatibilite syscall** | ~80% | ~100% |
| **Performance FS Windows** | Rapide (natif) | Lent (reseau 9P) |
| **Performance FS Linux** | Lent | **Tres rapide** |
| **Docker** | Non | **Oui** |
| **Memoire** | Partagee avec Windows | VM (configurable) |
| **Reseau** | Meme IP que Windows | IP propre (bridge) |

**Installation** :

```bash
# Dans PowerShell (admin)
wsl --install                    # Installe WSL2 + Ubuntu
wsl --install -d Debian          # Installer une autre distro
wsl --list --online              # Voir les distros disponibles
wsl --set-default-version 2     # WSL2 par defaut
```

**Acces aux fichiers** :

```bash
# Depuis WSL : acceder aux fichiers Windows
ls /mnt/c/Users/john/Desktop/
cd /mnt/d/Projects/

# Depuis Windows : acceder aux fichiers WSL
# Dans l'explorateur : \\wsl$\Ubuntu\home\john\
# Ou : \\wsl.localhost\Ubuntu\home\john\
```

> [!warning] Performance des fichiers
> Travaille **toujours** sur le filesystem natif !
> - Projet Linux → le mettre dans `~/projects/` (filesystem ext4 de WSL)
> - Projet Windows → le mettre dans `/mnt/c/...` (NTFS)
> Acceder aux fichiers Windows depuis WSL (ou inversement) est **lent**.

**Integration VS Code** :

```bash
# Depuis WSL, ouvrir VS Code sur le projet courant
code .

# VS Code installe automatiquement l'extension "Remote - WSL"
# et se connecte au filesystem Linux
```

**Git Bash vs WSL : quand utiliser quoi ?**

| Besoin | Git Bash | WSL |
|---|---|---|
| Juste utiliser Git | Oui | Overkill |
| Commandes Unix simples (grep, sed) | Oui | Oui |
| Compiler du C avec gcc | Limite | **Oui** |
| Docker | Non | **Oui** |
| Valgrind, strace, gdb | Non | **Oui** |
| Serveur web local (nginx, Apache) | Non | **Oui** |
| Scripts bash complexes | Limite | **Oui** |
| Projet Holberton | Non | **Oui** |

### 4.5 Windows Terminal

> [!tip] Le terminal moderne de Microsoft
> Windows Terminal est un emulateur de terminal qui peut heberger **n'importe quel shell** : CMD, PowerShell, Git Bash, WSL. C'est la **fenetre**, pas le shell.

**Features** :

```
+-----------------------------------------------------+
|  Windows Terminal                                    |
|  +-------+--------+---------+---------+              |
|  |  CMD  | PS7    | Ubuntu  | Git Bash|  ← onglets  |
|  +-------+--------+---------+---------+              |
|  |                                     |             |
|  |  $ ls -la                           |             |
|  |  total 32                           |             |
|  |  drwxr-xr-x  5 user user 4096      |             |
|  |  -rw-r--r--  1 user user  234      |             |
|  |                                     |             |
|  +-------------------------------------+             |
+-----------------------------------------------------+
```

- **Onglets** : plusieurs shells dans une fenetre
- **Split panes** : diviser la fenetre (Alt+Shift+D)
- **Profils** : configuration par shell (couleurs, police, image de fond)
- **GPU-accelere** : rendu rapide
- **Raccourcis** : Ctrl+Shift+T (nouvel onglet), Ctrl+Shift+W (fermer)
- **JSON config** : `settings.json` pour tout personnaliser

```bash
# Installer via winget
winget install Microsoft.WindowsTerminal
```

### 4.6 Tableau comparatif Windows

| | **CMD** | **PowerShell** | **Git Bash** | **WSL** | **Windows Terminal** |
|---|---|---|---|---|---|
| **Type** | Shell | Shell | Shell + env | OS (Linux) | Emulateur |
| **Commandes** | DOS | Cmdlets .NET | Unix (bash) | Unix (bash/zsh) | N/A (heberge des shells) |
| **Scripting** | .bat/.cmd | .ps1 | .sh | .sh | N/A |
| **POSIX** | Non | Non | Partiel | **Oui** | N/A |
| **Outils dev** | Tres limite | Moyen | Git + outils GNU | **Complet** | N/A |
| **Docker** | Non | Oui (Docker Desktop) | Non | **Oui (natif)** | N/A |
| **Cas d'usage** | Scripts legacy | Admin Windows | Git au quotidien | Dev Linux/C | Interface unifiee |

---

## 5. Les Terminaux sous macOS

### 5.1 Terminal.app

- Le terminal natif fourni avec macOS
- Simple, leger, fait le travail
- Supporte les onglets et les profils de couleur
- Utilise **zsh** par defaut depuis macOS Catalina (10.15)

### 5.2 iTerm2

> [!tip] Le terminal de reference sur macOS
> Si tu developpes sur macOS, iTerm2 est quasi indispensable.

**Features principales** :

| Feature | Description |
|---|---|
| **Split panes** | Diviser horizontalement/verticalement (Cmd+D, Cmd+Shift+D) |
| **Search** | Ctrl+Shift+F pour chercher dans le terminal |
| **Autocomplete** | Cmd+; pour autocompletion basee sur l'historique |
| **Profiles** | Profils differents par projet/serveur |
| **Triggers** | Actions automatiques basees sur le texte affiche |
| **Shell integration** | Marquer les commandes, navigation entre commandes |
| **Hotkey window** | Terminal en overlay (style Quake) avec un raccourci |
| **tmux integration** | Integration native avec tmux |

### 5.3 Le shell par defaut

```bash
# Verifier le shell courant
echo $SHELL

# Avant Catalina (< 10.15) : bash
/bin/bash

# Depuis Catalina (>= 10.15) : zsh
/bin/zsh

# Raison : la licence GPL v3 de bash (Apple ne veut pas distribuer du GPL v3)
# macOS avait bash 3.2 (2007!) qui est la derniere version GPL v2
```

---

## 6. Les Terminaux sous Linux

### 6.1 Emulateurs populaires

| Terminal | Bureau | Particularite |
|---|---|---|
| **GNOME Terminal** | GNOME | Defaut Ubuntu, simple et efficace |
| **Konsole** | KDE | Tres configurable, signets, SSH manager |
| **Alacritty** | Aucun | GPU-accelere, ultra rapide, config YAML |
| **Kitty** | Aucun | GPU-accelere, images inline, ligatures |
| **Terminator** | Aucun | Split panes natif, groupes |
| **xterm** | X11 | Le plus ancien, ultra leger |
| **st** | Aucun | Suckless, minimaliste, configure a la compilation |

```bash
# Installer Alacritty (GPU-accelere, config en YAML)
sudo apt install alacritty

# Installer Kitty (GPU-accelere, images dans le terminal)
sudo apt install kitty

# Installer Terminator (split panes)
sudo apt install terminator
```

### 6.2 TTY virtuels

Linux fournit **6 TTY virtuels** accessibles sans interface graphique :

```
+-----------------------------------------------------------+
|  Ctrl+Alt+F1  →  TTY1 (souvent l'interface graphique)     |
|  Ctrl+Alt+F2  →  TTY2 (terminal texte)                    |
|  Ctrl+Alt+F3  →  TTY3 (terminal texte)                    |
|  Ctrl+Alt+F4  →  TTY4 (terminal texte)                    |
|  Ctrl+Alt+F5  →  TTY5 (terminal texte)                    |
|  Ctrl+Alt+F6  →  TTY6 (terminal texte)                    |
+-----------------------------------------------------------+
```

> [!example] Quand utiliser un TTY ?
> - L'interface graphique a plante → Ctrl+Alt+F2 pour acceder a un TTY et `sudo systemctl restart gdm`
> - Tu veux un environnement completement isole
> - Tu fais du depannage systeme

---

## 7. Les terminaux integres aux IDE

### 7.1 VS Code

> [!tip] Le terminal integre le plus utilise
> VS Code integre un terminal complet accessible avec **Ctrl+`** (backtick).

```
+----------------------------------------------------------+
|  VS Code                                                  |
|  +----------------------------------------------------+  |
|  |  Editeur de code                                    |  |
|  |  main.c                                             |  |
|  |  +----------------------------------------------+  |  |
|  |  | #include <stdio.h>                           |  |  |
|  |  | int main(void) {                             |  |  |
|  |  |     printf("Hello\n");                       |  |  |
|  |  |     return 0;                                |  |  |
|  |  | }                                            |  |  |
|  |  +----------------------------------------------+  |  |
|  +----------------------------------------------------+  |
|  +----------------------------------------------------+  |
|  | TERMINAL  PROBLEMS  OUTPUT  DEBUG CONSOLE           |  |
|  | +--------+------+--------+                          |  |
|  | | bash   | zsh  | PS7    |  ← profils multiples     |  |
|  | +--------+------+--------+                          |  |
|  | $ gcc -Wall -Werror main.c -o main                  |  |
|  | $ ./main                                            |  |
|  | Hello                                               |  |
|  +----------------------------------------------------+  |
+----------------------------------------------------------+
```

**Configuration du shell dans VS Code** :

```json
// settings.json
{
    "terminal.integrated.defaultProfile.windows": "Git Bash",
    "terminal.integrated.profiles.windows": {
        "PowerShell": {
            "path": "pwsh",
            "icon": "terminal-powershell"
        },
        "Git Bash": {
            "path": "C:\\Program Files\\Git\\bin\\bash.exe",
            "icon": "terminal-bash"
        },
        "WSL": {
            "path": "wsl.exe",
            "icon": "terminal-linux"
        },
        "CMD": {
            "path": "cmd.exe"
        }
    }
}
```

**Raccourcis** :

| Raccourci | Action |
|---|---|
| Ctrl+` | Ouvrir/fermer le terminal |
| Ctrl+Shift+` | Nouveau terminal |
| Ctrl+Shift+5 | Split terminal |
| Ctrl+PageUp/Down | Naviguer entre terminaux |

### 7.2 JetBrains (CLion, IntelliJ, PyCharm)

- Terminal integre avec Alt+F12
- Supporte plusieurs profils de shell
- Integration avec les outils de build du projet

### 7.3 Terminal dans Docker

```bash
# Ouvrir un shell dans un container Docker
docker exec -it mon_container bash

# Lancer un container avec un shell interactif
docker run -it ubuntu:22.04 bash

# Le terminal dans Docker est un vrai shell Linux
# mais isole dans un container
```

---

## 8. Quel terminal/shell choisir ?

### 8.1 Flowchart decisionnel

```
Tu es sur quel OS ?
    |
    +---> Windows
    |       |
    |       +---> Tu fais du dev C / Linux ?
    |       |       |
    |       |       +---> OUI → WSL2 + Windows Terminal
    |       |       |
    |       |       +---> NON → Tu fais quoi ?
    |       |               |
    |       |               +---> Juste Git → Git Bash
    |       |               |
    |       |               +---> Admin Windows → PowerShell
    |       |               |
    |       |               +---> Web dev (Node, Python) → WSL2
    |       |
    |       +---> Emulateur → Windows Terminal (obligatoire)
    |
    +---> macOS
    |       |
    |       +---> Terminal → iTerm2
    |       +---> Shell → zsh (deja par defaut) + Oh My Zsh
    |
    +---> Linux
            |
            +---> Shell → bash (debutant) ou zsh + Oh My Zsh (avance)
            +---> Terminal → celui de ton DE ou Alacritty/Kitty
```

### 8.2 Recommandations par profil

> [!example] Etudiant Holberton
> - **OS** : WSL2 sur Windows OU Linux natif
> - **Terminal** : Windows Terminal (Windows) ou GNOME Terminal (Linux)
> - **Shell** : bash (c'est ce qui est utilise dans les projets et le checker)
> - **IDE** : VS Code avec terminal integre (bash via WSL)
> - **Pourquoi** : Les projets Holberton ciblent Linux. Tu as besoin de gcc, valgrind, Betty, etc.

> [!example] Developpeur Web (JavaScript/Python)
> - **Shell** : zsh + Oh My Zsh (plugins git, node, python, docker)
> - **Terminal** : iTerm2 (macOS) ou Windows Terminal (Windows)
> - **Pourquoi** : L'autocompletion zsh et les plugins font gagner beaucoup de temps

> [!example] Sysadmin / DevOps
> - **Shell** : bash (standard sur les serveurs) + zsh en local
> - **Terminal** : avec tmux pour les sessions persistantes
> - **Pourquoi** : Tu te connectes en SSH a des serveurs qui ont bash. tmux te permet de garder des sessions actives.

> [!example] Data Scientist
> - **Shell** : bash ou zsh
> - **Outils** : Jupyter Notebook + terminal pour la gestion d'environnements (conda, pip)
> - **Terminal** : celui de l'IDE (VS Code ou JupyterLab)

---

## 9. Configuration et personnalisation

### 9.1 Changer le shell par defaut

```bash
# Voir les shells disponibles
cat /etc/shells

# Changer son shell (necessite le mot de passe)
chsh -s /bin/zsh

# Verifier le changement
echo $SHELL
# Attention : il faut se deconnecter/reconnecter pour que ce soit effectif
```

### 9.2 .bashrc vs .bash_profile vs .profile

```
                   +------------------+
                   | Login shell ?    |
                   | (ssh, tty, su -) |
                   +--------+---------+
                            |
                   +--------+---------+
                   |  OUI        NON  |
                   v                  v
          +----------------+  +----------------+
          | .bash_profile  |  |   .bashrc      |
          | (ou .profile)  |  |                |
          +-------+--------+  +----------------+
                  |
                  | (bonne pratique :
                  |  source .bashrc)
                  v
          +----------------+
          |   .bashrc      |
          +----------------+

REGLE : Mets TOUT dans .bashrc et fais en sorte que .bash_profile le source.
```

```bash
# ~/.bash_profile (ou ~/.profile)
# Source .bashrc pour que les aliases marchent partout
if [ -f "$HOME/.bashrc" ]; then
    source "$HOME/.bashrc"
fi

# Variables d'environnement specifiques au login
export EDITOR=vim
export LANG=en_US.UTF-8
```

### 9.3 Prompt PS1 personnalise

```bash
# PS1 basique
export PS1="\u@\h:\w\$ "
# john@laptop:~/projects$

# Avec couleurs
export PS1="\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "
# john@laptop:~/projects$  (en vert:bleu)

# Codes PS1 :
# \u = username
# \h = hostname
# \w = repertoire courant (complet)
# \W = repertoire courant (nom seulement)
# \d = date
# \t = heure (24h)
# \$ = $ (ou # si root)
# \n = nouvelle ligne

# Codes couleur ANSI :
# \[\033[XXm\] ou X est :
# 30=noir 31=rouge 32=vert 33=jaune 34=bleu 35=magenta 36=cyan 37=blanc
# Prefixer de 01; pour gras : \033[01;32m = vert gras
# \033[00m = reset
```

**Prompt avec branche Git** :

```bash
# Fonction pour afficher la branche git
parse_git_branch() {
    git branch 2>/dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}

# Prompt avec branche git en jaune
export PS1="\[\033[32m\]\u\[\033[00m\]:\[\033[34m\]\w\[\033[33m\]\$(parse_git_branch)\[\033[00m\]\$ "
# john:~/projects/myapp (main)$
```

### 9.4 Aliases utiles

```bash
# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ll='ls -la'
alias la='ls -A'
alias l='ls -CF'

# Git
alias gs='git status'
alias ga='git add'
alias gc='git commit -m'
alias gp='git push'
alias gl='git log --oneline -20'
alias gd='git diff'
alias gco='git checkout'
alias gbr='git branch'

# Compilation C
alias gccf='gcc -Wall -Werror -Wextra -pedantic -std=gnu89'
alias val='valgrind --leak-check=full --show-leak-kinds=all'

# Securite
alias rm='rm -i'       # Demander confirmation avant de supprimer
alias cp='cp -i'
alias mv='mv -i'

# Raccourcis
alias c='clear'
alias h='history'
alias ports='netstat -tulanp'
alias myip='curl ifconfig.me'
```

---

## 10. Raccourcis clavier universels

### 10.1 Raccourcis du shell (bash/zsh)

> [!tip] Ces raccourcis viennent de Emacs
> La plupart des raccourcis du terminal sont inspires de l'editeur Emacs. Ils fonctionnent dans bash, zsh et la plupart des applications qui utilisent readline.

| Raccourci | Action | Mnemonic |
|---|---|---|
| **Ctrl+C** | Interrompre la commande en cours | **C**ancel |
| **Ctrl+D** | Fermer le shell (EOF) / supprimer caractere | **D**elete / EOF |
| **Ctrl+Z** | Suspendre le processus (fg pour reprendre) | **Z**zzz (sleep) |
| **Ctrl+L** | Effacer l'ecran (comme `clear`) | c**L**ear |
| **Ctrl+R** | Recherche inversee dans l'historique | **R**everse search |
| **Ctrl+A** | Aller au debut de la ligne | **A** = debut alphabet |
| **Ctrl+E** | Aller a la fin de la ligne | **E**nd |
| **Ctrl+U** | Supprimer du curseur au debut de la ligne | |
| **Ctrl+K** | Supprimer du curseur a la fin de la ligne | **K**ill |
| **Ctrl+W** | Supprimer le mot precedent | **W**ord |
| **Alt+B** | Reculer d'un mot | **B**ack |
| **Alt+F** | Avancer d'un mot | **F**orward |
| **Tab** | Autocompletion | |
| **Tab Tab** | Lister les completions possibles | |
| **!!** | Repeter la derniere commande | |
| **!$** | Dernier argument de la derniere commande | |

### 10.2 Raccourcis Ctrl+R (reverse search)

```
# Tu tapes Ctrl+R, puis tu tapes un bout de la commande
(reverse-i-search)`gcc': gcc -Wall -Werror -Wextra main.c -o main

# Ctrl+R encore : resultat precedent
# Enter : executer la commande trouvee
# Ctrl+G : annuler la recherche
# Fleche droite : editer la commande trouvee
```

### 10.3 Raccourcis par emulateur de terminal

| Raccourci | Windows Terminal | GNOME Terminal | iTerm2 |
|---|---|---|---|
| Nouvel onglet | Ctrl+Shift+T | Ctrl+Shift+T | Cmd+T |
| Fermer onglet | Ctrl+Shift+W | Ctrl+Shift+W | Cmd+W |
| Split horizontal | Alt+Shift+- | N/A (Terminator) | Cmd+Shift+D |
| Split vertical | Alt+Shift+= | N/A (Terminator) | Cmd+D |
| Copier | Ctrl+Shift+C | Ctrl+Shift+C | Cmd+C |
| Coller | Ctrl+Shift+V | Ctrl+Shift+V | Cmd+V |
| Zoom + | Ctrl+= | Ctrl+Shift++ | Cmd++ |
| Zoom - | Ctrl+- | Ctrl+- | Cmd+- |
| Recherche | Ctrl+Shift+F | Ctrl+Shift+F | Cmd+F |

---

## 11. Exercices pratiques

### Exercice 1 : Identifier ton environnement

```bash
# 1. Quel est ton shell actuel ?
echo $SHELL
echo $0

# 2. Quel est ton emulateur de terminal ?
echo $TERM
echo $TERM_PROGRAM    # sur macOS

# 3. Quel OS ?
uname -a

# 4. Ou est le binaire de ton shell ?
which bash
which zsh
```

### Exercice 2 : Configurer ton .bashrc

> [!example] A faire
> 1. Ouvre ton fichier `~/.bashrc`
> 2. Ajoute au moins 5 aliases utiles
> 3. Personnalise ton prompt PS1 avec :
>    - Ton username en vert
>    - Le repertoire courant en bleu
>    - La branche git en jaune
> 4. Source le fichier : `source ~/.bashrc`
> 5. Verifie que tes aliases fonctionnent

### Exercice 3 : Explorer les shells

```bash
# 1. Liste les shells disponibles
cat /etc/shells

# 2. Lance temporairement un autre shell
zsh        # entre dans zsh
echo $0    # verifie
exit       # reviens a ton shell

# 3. Compare les prompts et l'autocompletion
# Dans bash : tape "git " puis Tab Tab
# Dans zsh :  tape "git " puis Tab
# Note les differences
```

### Exercice 4 : Windows - Comparer les environnements

> [!example] Si tu es sur Windows
> 1. Ouvre CMD et tape : `dir`, `echo %PATH%`, `set`
> 2. Ouvre PowerShell et tape : `Get-ChildItem`, `$env:PATH`, `Get-Variable`
> 3. Ouvre Git Bash et tape : `ls -la`, `echo $PATH`, `env`
> 4. Ouvre WSL et tape les memes commandes que Git Bash
> 5. Compare : vitesse, lisibilite, fonctionnalites
> 6. Essaye `gcc --version` et `valgrind --version` dans chaque → quel est le seul ou tout marche ?

### Exercice 5 : Maitriser les raccourcis

> [!example] Pratique ces raccourcis
> 1. Tape une longue commande
> 2. Ctrl+A pour aller au debut, Ctrl+E pour la fin
> 3. Ctrl+U pour tout effacer
> 4. Tape `echo "Hello World"` puis Ctrl+C pour annuler
> 5. Utilise Ctrl+R pour retrouver une commande dans l'historique
> 6. Utilise Tab pour autocompleter un chemin de fichier
> 7. Utilise `!!` pour repeter la derniere commande
> 8. Utilise `sudo !!` pour relancer la derniere commande en root

---

## 12. Resume

```
+-----------------------------------------------------------+
|                    CHEAT SHEET TERMINAUX                    |
+-----------------------------------------------------------+
|                                                            |
|  CONCEPTS :                                                |
|  Terminal = fenetre   |  Shell = interpreteur              |
|  Emulateur = logiciel |  TTY = terminal virtuel            |
|                                                            |
|  SHELLS :                                                  |
|  bash → standard Linux, a connaitre absolument             |
|  zsh  → bash ameliore, defaut macOS, Oh My Zsh            |
|  fish → user-friendly, pas POSIX                           |
|                                                            |
|  WINDOWS :                                                 |
|  CMD → legacy, eviter si possible                          |
|  PowerShell → admin Windows, objets .NET                   |
|  Git Bash → Git + outils Unix basiques                     |
|  WSL2 → VRAI Linux, recommande pour le dev                 |
|  Windows Terminal → emulateur moderne, obligatoire         |
|                                                            |
|  CHOIX RECOMMANDE (etudiant dev) :                         |
|  Windows Terminal + WSL2 (Ubuntu) + bash + VS Code         |
|                                                            |
+-----------------------------------------------------------+
```

---

## Liens

- [[01 - Shell et Commandes Linux]] - Les commandes essentielles
- [[01 - Archives et Compression]] - Archivage et compression en ligne de commande
