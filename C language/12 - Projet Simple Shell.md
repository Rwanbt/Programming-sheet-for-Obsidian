# Projet Simple Shell

> [!tip] Analogie
> Imagine un **guichet d'accueil** dans un grand immeuble de bureaux. Tu arrives avec une demande ("je veux voir le comptable"), le guichetier (le **shell**) comprend ta requete, trouve le bon bureau (le **programme**), t'y envoie, puis attend ton retour pour t'accueillir a nouveau. Le shell est exactement ce guichetier : il lit tes commandes, trouve les programmes correspondants, les execute, puis revient te presenter le prompt. Sans lui, tu devrais parler directement au noyau du systeme d'exploitation -- autant essayer de communiquer en binaire avec le processeur.

---

## 1. Qu'est-ce qu'un Shell ?

### 1.1 Definition

Un **shell** (coquille en anglais) est un **interpreteur de commandes** qui sert d'interface entre l'utilisateur et le noyau (kernel) du systeme d'exploitation. C'est un programme comme un autre, mais son role est special : il lit des commandes, les interprete, et demande au kernel de les executer.

```
+-------------------------------------------------------------------+
|                     ARCHITECTURE SIMPLIFIEE                       |
+-------------------------------------------------------------------+
|                                                                   |
|   Utilisateur                                                     |
|       |                                                           |
|       |  "ls -la /home"                                           |
|       v                                                           |
|   +--------+                                                      |
|   | SHELL  |  <-- Interpreteur de commandes (notre projet!)       |
|   +--------+                                                      |
|       |                                                           |
|       |  fork() + execve("/bin/ls", ["ls","-la","/home"], env)    |
|       v                                                           |
|   +--------+                                                      |
|   | KERNEL |  <-- Noyau du systeme d'exploitation                 |
|   +--------+                                                      |
|       |                                                           |
|       |  Instructions machine, acces disque, etc.                 |
|       v                                                           |
|   +----------+                                                    |
|   | HARDWARE |  <-- CPU, RAM, Disque, peripheriques               |
|   +----------+                                                    |
|                                                                   |
+-------------------------------------------------------------------+
```

> [!info] En resume
> Le shell est un **programme utilisateur** qui sert de **pont** entre toi et le kernel. Il ne fait PAS partie du kernel lui-meme.

### 1.2 Mode Interactif vs Non-Interactif

Le shell peut fonctionner de deux facons :

**Mode interactif** : le shell est connecte a un terminal. Il affiche un prompt (`$ `), attend ta commande, l'execute, puis reaffiche le prompt. C'est ce que tu fais quand tu ouvres un terminal.

```bash
$ ls -la        <-- tu tapes ca
total 48        <-- le shell execute et affiche le resultat
drwxr-xr-x ...
$ _             <-- prompt a nouveau, le shell attend
```

**Mode non-interactif** : le shell lit des commandes depuis un pipe ou un fichier, pas depuis un terminal. Il n'affiche pas de prompt.

```bash
echo "ls -la" | ./hsh          # pipe : non-interactif
cat commands.txt | ./hsh       # fichier via pipe : non-interactif
```

> [!tip] Detection du mode
> On utilise `isatty(STDIN_FILENO)` pour savoir si l'entree standard est un terminal. Si oui, on est en mode interactif. Sinon, on est en mode non-interactif.

### 1.3 Histoire des Shells

| Annee | Shell | Auteur | Contribution |
|-------|-------|--------|-------------|
| 1971 | Thompson Shell | Ken Thompson | Premier shell UNIX, tres simple |
| 1977 | Bourne Shell (sh) | Stephen Bourne | Variables, structures de controle, scripts |
| 1978 | C Shell (csh) | Bill Joy | Syntaxe inspiree du C |
| 1983 | Korn Shell (ksh) | David Korn | Combine Bourne + C Shell |
| 1989 | Bash | Brian Fox | Shell GNU, le plus repandu aujourd'hui |
| 2005 | Zsh | Paul Falstad | Completion avancee, themes |

> [!info] Le projet Simple Shell
> Notre shell s'inspire du **Bourne Shell (sh)**. On doit reproduire son comportement de base. C'est pourquoi on compare souvent la sortie de notre shell avec `/bin/sh`.

### 1.4 Le Principe REPL

Tout shell fonctionne selon le modele **REPL** (Read-Eval-Print-Loop) :

```
+-----------------------------------------------+
|                BOUCLE REPL                     |
+-----------------------------------------------+
|                                                |
|   +--------+     +--------+     +---------+   |
|   |  READ  | --> |  EVAL  | --> |  PRINT  |   |
|   | (lire) |     |(evaluer)|    |(afficher)|   |
|   +--------+     +--------+     +---------+   |
|       ^                              |         |
|       |          +--------+          |         |
|       +--------- |  LOOP  | <-------+         |
|                  |(boucle)|                    |
|                  +--------+                    |
|                                                |
+-----------------------------------------------+
```

1. **Read** : lire la commande de l'utilisateur (getline)
2. **Eval** : analyser et executer la commande (parse + fork/execve)
3. **Print** : afficher le resultat (la commande executee ecrit elle-meme)
4. **Loop** : recommencer depuis le debut

---

## 2. Concepts Systeme Fondamentaux

### 2.1 PID et PPID

Chaque processus dans un systeme UNIX possede un identifiant unique : le **PID** (Process ID). Il possede aussi un **PPID** (Parent Process ID) qui identifie le processus qui l'a cree.

```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    printf("Mon PID  : %d\n", getpid());
    printf("Mon PPID : %d\n", getppid());
    return (0);
}
```

> [!info] Arbre des processus
> Tous les processus forment un **arbre** dont la racine est `init` (PID 1) ou `systemd` sur les systemes modernes. Quand tu ouvres un terminal, le terminal cree un processus shell. Quand le shell execute une commande, il cree un processus fils. Et ainsi de suite.

```
systemd (PID 1)
├── sshd (PID 450)
│   └── bash (PID 1200)        <-- ton shell
│       └── ls (PID 1201)      <-- commande en cours
├── cron (PID 500)
└── nginx (PID 600)
    ├── nginx worker (PID 601)
    └── nginx worker (PID 602)
```

Les fonctions cles :

| Fonction | Prototype | Description |
|----------|-----------|-------------|
| `getpid()` | `pid_t getpid(void)` | Retourne le PID du processus courant |
| `getppid()` | `pid_t getppid(void)` | Retourne le PID du processus parent |

### 2.2 Environnement

L'**environnement** est un ensemble de variables sous la forme `CLE=VALEUR` que chaque processus herite de son parent. C'est un tableau de chaines de caracteres (comme `argv`), termine par `NULL`.

```c
/* Methode 1 : variable globale extern */
extern char **environ;

void print_env(void)
{
    int i = 0;

    while (environ[i] != NULL)
    {
        printf("%s\n", environ[i]);
        i++;
    }
}
```

```c
/* Methode 2 : getenv() pour une variable specifique */
#include <stdlib.h>

void show_path(void)
{
    char *path = getenv("PATH");

    if (path != NULL)
        printf("PATH = %s\n", path);
    else
        printf("PATH non defini\n");
}
```

> [!warning] Passage de l'environnement
> Quand un processus appelle `fork()`, le fils **herite** de l'environnement du parent. Quand on appelle `execve()`, on doit **explicitement** passer l'environnement en 3eme argument. Si on passe `NULL`, le nouveau programme n'aura AUCUNE variable d'environnement !

```
+---------------------------------------------+
|     HERITAGE DE L'ENVIRONNEMENT             |
+---------------------------------------------+
|                                             |
|  Parent (environ)                           |
|  +---------------------------+              |
|  | PATH=/usr/bin:/bin        |              |
|  | HOME=/home/user           |              |
|  | LANG=fr_FR.UTF-8         |              |
|  +---------------------------+              |
|       |                                     |
|       | fork() --> copie exacte              |
|       v                                     |
|  Fils (environ identique)                   |
|  +---------------------------+              |
|  | PATH=/usr/bin:/bin        |              |
|  | HOME=/home/user           |              |
|  | LANG=fr_FR.UTF-8         |              |
|  +---------------------------+              |
|       |                                     |
|       | execve(cmd, argv, environ)           |
|       v                                     |
|  Nouveau programme (recoit environ)         |
|                                             |
+---------------------------------------------+
```

### 2.3 Fonctions vs Appels Systeme

C'est une distinction fondamentale a comprendre pour ce projet.

> [!tip] Analogie
> Une **fonction de bibliotheque**, c'est comme demander a ton assistant de faire un calcul sur son bloc-notes (espace utilisateur). Un **appel systeme**, c'est comme remplir un formulaire officiel et le soumettre au bureau du gouvernement (noyau). Le formulaire passe par un guichet securise (le **context switch**), le gouvernement traite la demande, et renvoie le resultat.

| Fonction (bibliotheque) | Appel systeme (kernel) | Difference |
|------------------------|----------------------|------------|
| `printf()` | `write()` | printf formate puis appelle write |
| `malloc()` | `brk()` / `mmap()` | malloc gere un pool, appelle brk si besoin |
| `fopen()` | `open()` | fopen ajoute du buffering |
| `fgets()` | `read()` | fgets bufferise les lectures |
| `getline()` | `read()` | getline alloue et lit dynamiquement |
| `strtok()` | -- | Pur espace utilisateur, pas d'appel systeme |
| -- | `fork()` | Appel systeme pur (creation de processus) |
| -- | `execve()` | Appel systeme pur (remplacement de processus) |
| -- | `wait()` | Appel systeme pur (attente de processus fils) |

```
+-------------------------------------------------------------------+
|                    ESPACE UTILISATEUR                              |
|                                                                   |
|   ton_programme.c                                                 |
|       |                                                           |
|       | appelle printf("hello")                                   |
|       v                                                           |
|   libc (bibliotheque C)                                           |
|       |                                                           |
|       | printf formate, bufferise, puis appelle write()           |
|       v                                                           |
+===================================================================+
|   --- FRONTIERE : appel systeme (interruption / syscall) ---      |
+===================================================================+
|                                                                   |
|                    ESPACE NOYAU (KERNEL)                          |
|                                                                   |
|   write() du noyau : ecrit sur le file descriptor                 |
|       |                                                           |
|       v                                                           |
|   pilote de peripherique (driver terminal, etc.)                  |
|                                                                   |
+-------------------------------------------------------------------+
```

### 2.4 Le PATH

Le `PATH` est une variable d'environnement qui contient une liste de repertoires separes par `:`. Quand tu tapes une commande sans chemin absolu (ex: `ls` au lieu de `/bin/ls`), le shell cherche l'executable dans chaque repertoire du PATH, dans l'ordre.

```
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

Le shell fait ceci pour trouver `ls` :

```
1. Verifier /usr/local/bin/ls  --> existe pas
2. Verifier /usr/bin/ls        --> existe pas
3. Verifier /bin/ls            --> EXISTE ! Executable ? Oui.
   --> Executer /bin/ls
```

> [!warning] Cas speciaux
> - Si la commande contient un `/` (ex: `./monprog` ou `/bin/ls`), on ne cherche PAS dans le PATH. C'est deja un chemin.
> - Si `PATH` n'est pas defini ou est vide, on ne cherche nulle part (ou on cherche dans le repertoire courant selon l'implementation).
> - Si le fichier existe mais n'est pas executable, on continue la recherche.

Pour verifier si un fichier existe et est executable :

```c
#include <sys/stat.h>

/**
 * file_exists - verifie si un fichier existe et est executable
 * @path: chemin complet du fichier
 *
 * Return: 1 si le fichier existe et est executable, 0 sinon
 */
int file_exists(const char *path)
{
    struct stat st;

    if (stat(path, &st) == 0)
        return (1);  /* le fichier existe */
    return (0);
}
```

Ou plus simplement avec `access()` :

```c
if (access(full_path, X_OK) == 0)
{
    /* Le fichier existe et est executable */
}
```

---

## 3. Les Appels Systeme Cles

### 3.1 fork() -- Creer un Processus Fils

`fork()` est l'appel systeme qui cree un **nouveau processus** (le fils) qui est une copie quasi-exacte du processus appelant (le parent).

```c
#include <sys/types.h>
#include <unistd.h>

pid_t fork(void);
```

**Valeurs de retour :**
- Dans le **parent** : retourne le PID du fils (un nombre positif)
- Dans le **fils** : retourne `0`
- En cas d'**erreur** : retourne `-1` (aucun fils n'est cree)

```
+-----------------------------------------------------------------+
|                    FONCTIONNEMENT DE FORK()                     |
+-----------------------------------------------------------------+
|                                                                 |
|   Avant fork() :                                                |
|                                                                 |
|   Processus parent (PID 1200)                                   |
|   +---------------------------+                                 |
|   | code + donnees + pile     |                                 |
|   | variables, file desc, env |                                 |
|   +---------------------------+                                 |
|               |                                                 |
|               | fork()                                          |
|               |                                                 |
|   Apres fork() :                                                |
|               |                                                 |
|       +-------+-------+                                         |
|       |               |                                         |
|       v               v                                         |
|   Parent          Fils (PID 1201)                               |
|   (PID 1200)     +---------------------------+                  |
|   fork()=1201    | COPIE du code + donnees   |                  |
|   continue...    | COPIE des variables        |                 |
|                  | COPIE des file descriptors |                  |
|                  +---------------------------+                  |
|                  fork()=0                                       |
|                  continue...                                    |
|                                                                 |
+-----------------------------------------------------------------+
```

> [!info] Copy-On-Write (COW)
> En realite, le kernel ne copie pas toute la memoire immediatement. Il utilise le mecanisme de **Copy-On-Write** : les pages memoire sont partagees entre parent et fils jusqu'a ce que l'un des deux les modifie. C'est une optimisation importante car souvent le fils va immediatement appeler `execve()` et remplacer toute sa memoire.

Exemple complet :

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main(void)
{
    pid_t pid;

    printf("Avant fork. PID = %d\n", getpid());

    pid = fork();

    if (pid == -1)
    {
        perror("fork");
        return (1);
    }

    if (pid == 0)
    {
        /* Code execute UNIQUEMENT par le FILS */
        printf("Je suis le FILS.  PID = %d, PPID = %d\n",
               getpid(), getppid());
    }
    else
    {
        /* Code execute UNIQUEMENT par le PARENT */
        printf("Je suis le PARENT. PID = %d, fils PID = %d\n",
               getpid(), pid);
    }

    return (0);
}
```

> [!danger] Attention
> Apres `fork()`, les deux processus executent le meme code a partir de la ligne suivante. La seule difference est la valeur de retour de `fork()`. C'est grace a cette valeur qu'on differencie parent et fils avec un `if/else`.

### 3.2 execve() -- Remplacer le Processus Courant

`execve()` remplace **completement** le processus courant par un nouveau programme. Le PID reste le meme, mais le code, les donnees, la pile, tout est remplace.

```c
#include <unistd.h>

int execve(const char *pathname, char *const argv[], char *const envp[]);
```

**Parametres :**
- `pathname` : chemin absolu de l'executable (ex: `"/bin/ls"`)
- `argv` : tableau d'arguments (ex: `{"ls", "-la", NULL}`)
- `envp` : tableau d'environnement (ex: `environ`)

**Valeurs de retour :**
- En cas de **succes** : `execve` ne retourne **JAMAIS** (le processus est devenu autre chose)
- En cas d'**erreur** : retourne `-1` et `errno` est positionne

```
+-----------------------------------------------------------------+
|              FONCTIONNEMENT DE EXECVE()                          |
+-----------------------------------------------------------------+
|                                                                 |
|   Processus fils (PID 1201)                                     |
|   +---------------------------+                                 |
|   | Code du shell (copie)     |                                 |
|   | Variables du shell        |                                 |
|   +---------------------------+                                 |
|               |                                                 |
|               | execve("/bin/ls", ["ls","-la",NULL], environ)    |
|               |                                                 |
|               v                                                 |
|   Processus fils (PID 1201)  <-- MEME PID !                    |
|   +---------------------------+                                 |
|   | Code de /bin/ls           |  <-- TOUT est remplace          |
|   | Variables de ls           |                                 |
|   +---------------------------+                                 |
|               |                                                 |
|               | ls s'execute et affiche les fichiers             |
|               |                                                 |
|               v                                                 |
|           exit(0)  --> le processus 1201 se termine             |
|                                                                 |
+-----------------------------------------------------------------+
```

Exemple :

```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    char *argv[] = {"ls", "-la", "/tmp", NULL};

    printf("Avant execve\n");
    execve("/bin/ls", argv, environ);

    /* Si on arrive ici, execve a echoue */
    perror("execve");
    return (1);
}
```

> [!warning] Point crucial
> Si `execve` reussit, **tout le code apres l'appel est inaccessible** car le processus a ete remplace. C'est pour cela qu'on appelle toujours `execve` dans le **fils** (apres `fork`), jamais dans le parent. Sinon, le shell lui-meme serait remplace par la commande !

### 3.3 wait() et waitpid() -- Attendre le Fils

Quand le parent cree un fils avec `fork()`, il doit **attendre** que le fils termine. Sinon :
1. Le fils devient un **processus zombie** (termine mais pas nettoye)
2. Le parent ne sait pas si la commande a reussi ou echoue

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *status);
pid_t waitpid(pid_t pid, int *status, int options);
```

**wait()** : attend n'importe quel fils.
**waitpid()** : attend un fils specifique (par PID).

Le parametre `status` permet de recuperer des informations sur la terminaison du fils :

| Macro | Description |
|-------|-------------|
| `WIFEXITED(status)` | Vrai si le fils s'est termine normalement (avec `exit()`) |
| `WEXITSTATUS(status)` | Code de sortie du fils (0-255) si `WIFEXITED` est vrai |
| `WIFSIGNALED(status)` | Vrai si le fils a ete tue par un signal |
| `WTERMSIG(status)` | Numero du signal qui a tue le fils |

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(void)
{
    pid_t pid;
    int status;

    pid = fork();
    if (pid == -1)
    {
        perror("fork");
        return (1);
    }

    if (pid == 0)
    {
        /* Fils : execute /bin/ls */
        char *argv[] = {"ls", NULL};

        execve("/bin/ls", argv, environ);
        perror("execve");
        exit(127);
    }
    else
    {
        /* Parent : attend le fils */
        waitpid(pid, &status, 0);

        if (WIFEXITED(status))
            printf("Le fils s'est termine avec le code %d\n",
                   WEXITSTATUS(status));
    }

    return (0);
}
```

> [!danger] Processus zombies
> Si le parent ne fait PAS `wait()`, le fils termine et devient un **zombie** : il occupe toujours une entree dans la table des processus du kernel. Accumule assez de zombies et le systeme ne pourra plus creer de nouveaux processus. Toujours appeler `wait()` ou `waitpid()` apres chaque `fork()`.

### 3.4 Le Pattern fork + execve + wait

C'est **LE** patron de conception fondamental de tout shell UNIX. Chaque commande est executee selon ce schema :

```
+-------------------------------------------------------------------+
|           LE PATTERN FORK + EXECVE + WAIT                         |
+-------------------------------------------------------------------+
|                                                                   |
|  Parent (shell, PID 100)          Fils (PID 101)                  |
|       |                                |                          |
|       |------- fork() --------------->|                           |
|       |                                |                          |
|       |                                |-- execve("/bin/ls",      |
|       |                                |          argv, environ)  |
|       |                                |                          |
|       |                                |   (le fils EST /bin/ls   |
|       |                                |    maintenant)           |
|       |                                |                          |
|       |-- waitpid(101, &status, 0)     |   ls s'execute...        |
|       |   (parent BLOQUE ici)          |   affiche fichiers...    |
|       |                                |                          |
|       |                                |-- exit(0)                |
|       |                                |                          |
|       |<----- fils termine ------------|                          |
|       |                                                           |
|       |   WEXITSTATUS(status) == 0                                |
|       |                                                           |
|       |-- affiche prompt ($)                                      |
|       |                                                           |
|       v                                                           |
|   (attend la prochaine commande)                                  |
|                                                                   |
+-------------------------------------------------------------------+
```

> [!tip] Pourquoi ce pattern ?
> On ne peut pas juste appeler `execve` dans le shell car cela **remplacerait** le shell par la commande. Le shell disparaitrait ! Avec `fork()`, on cree une copie du shell. C'est la copie qui est remplacee par `execve`. Le shell original reste intact et attend tranquillement que la commande finisse.

Voici le pattern en code :

```c
/**
 * execute_command - execute une commande avec fork + execve + wait
 * @argv: tableau d'arguments (argv[0] = commande)
 * @shell_name: nom du shell (pour les messages d'erreur)
 *
 * Return: code de sortie de la commande
 */
int execute_command(char **argv, char *shell_name)
{
    pid_t pid;
    int status;

    pid = fork();
    if (pid == -1)
    {
        perror("fork");
        return (1);
    }

    if (pid == 0)
    {
        /* FILS : executer la commande */
        if (execve(argv[0], argv, environ) == -1)
        {
            fprintf(stderr, "%s: 1: %s: not found\n",
                    shell_name, argv[0]);
            exit(127);
        }
    }
    else
    {
        /* PARENT : attendre le fils */
        waitpid(pid, &status, 0);
        if (WIFEXITED(status))
            return (WEXITSTATUS(status));
    }

    return (0);
}
```

---

## 4. Architecture du Shell -- Vue d'Ensemble

### 4.1 La Boucle Principale

Notre shell suit cette logique globale :

```
+===================================================================+
|                ORGANIGRAMME COMPLET DU SHELL                      |
+===================================================================+
|                                                                   |
|   START                                                           |
|     |                                                             |
|     v                                                             |
|   [stdin est un terminal ?]                                       |
|     |           |                                                 |
|    OUI         NON                                                |
|     |           |                                                 |
|     v           |                                                 |
|   [Afficher     |                                                 |
|    prompt]      |                                                 |
|     |           |                                                 |
|     +-----+-----+                                                |
|           |                                                       |
|           v                                                       |
|   [Lire une ligne (getline)]                                      |
|           |                                                       |
|           v                                                       |
|   [getline retourne -1 ?] ----OUI----> [free & exit]             |
|           |                            (EOF ou erreur)            |
|          NON                                                      |
|           |                                                       |
|           v                                                       |
|   [Tokeniser la ligne en argv]                                    |
|           |                                                       |
|           v                                                       |
|   [argv vide / ligne vide ?] ----OUI----> [retour boucle]        |
|           |                                                       |
|          NON                                                      |
|           |                                                       |
|           v                                                       |
|   [argv[0] est un builtin ?]                                      |
|     |           |                                                 |
|    OUI         NON                                                |
|     |           |                                                 |
|     v           v                                                 |
|   [Executer   [argv[0] contient '/' ?]                            |
|    builtin]     |           |                                     |
|     |          OUI         NON                                    |
|     |           |           |                                     |
|     |           v           v                                     |
|     |   [Verifier que   [Chercher dans PATH]                      |
|     |    le fichier       |           |                           |
|     |    existe]         TROUVE    PAS TROUVE                     |
|     |        |            |           |                           |
|     |        v            v           v                           |
|     |   [fork+execve] [fork+execve] [Erreur:                     |
|     |        |            |          not found]                   |
|     |        v            v           |                           |
|     |   [wait fils]  [wait fils]      |                           |
|     |        |            |           |                           |
|     +--------+-----+-----+-----------+                           |
|                     |                                             |
|                     v                                             |
|              [Retour boucle] ---------> (recommencer)             |
|                                                                   |
+===================================================================+
```

### 4.2 Structure des Donnees

Pour notre shell simple, on n'a pas besoin de structures complexes. Voici ce qu'on manipule principalement :

```
+-------------------------------------------------------------------+
|                   DONNEES PRINCIPALES                             |
+-------------------------------------------------------------------+
|                                                                   |
|   char *line          --> "ls -la /tmp\n"   (de getline)          |
|                                                                   |
|   char **argv         --> argv[0] = "ls"                          |
|                           argv[1] = "-la"                         |
|                           argv[2] = "/tmp"                        |
|                           argv[3] = NULL                          |
|                                                                   |
|   char *full_path     --> "/bin/ls"         (apres recherche)     |
|                                                                   |
|   char **environ      --> environ[0] = "PATH=/usr/bin:/bin"       |
|                           environ[1] = "HOME=/home/user"          |
|                           environ[2] = "LANG=fr_FR.UTF-8"        |
|                           environ[3] = NULL                       |
|                                                                   |
|   int status          --> code de retour du dernier processus     |
|   int line_count      --> numero de ligne (pour erreurs)          |
|                                                                   |
+-------------------------------------------------------------------+
```

---

## 5. Implementation Etape par Etape

### Etape 0 : Le Header (shell.h)

Le header centralise toutes les declarations. C'est le premier fichier a creer.

```c
#ifndef SHELL_H
#define SHELL_H

/* === Includes === */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <signal.h>
#include <errno.h>

/* === Variables globales externes === */
extern char **environ;

/* === Macros === */
#define PROMPT "$ "
#define BUFSIZE 1024
#define DELIMITERS " \t\n"

/* === Prototypes : boucle principale === */
void shell_loop(char *shell_name);

/* === Prototypes : parsing === */
char **tokenize(char *line);

/* === Prototypes : execution === */
int execute(char **argv, char *shell_name, int line_count);

/* === Prototypes : resolution PATH === */
char *find_in_path(const char *command);

/* === Prototypes : builtins === */
int handle_builtin(char **argv, char *line);
int builtin_exit(char **argv, char *line);
int builtin_env(void);

/* === Prototypes : utilitaires === */
char *_strdup(const char *str);
int _strlen(const char *s);
char *_strcat_path(const char *dir, const char *file);
int _strcmp(const char *s1, const char *s2);
void free_argv(char **argv);

#endif /* SHELL_H */
```

> [!info] Pourquoi `extern char **environ` ?
> `environ` est une variable globale definie par le systeme (la libc). En la declarant `extern`, on dit au compilateur "cette variable existe quelque part, je veux y acceder". On ne la definit pas, on la reference.

> [!tip] Organisation des prototypes
> Regrouper les prototypes par fichier/module rend le header plus lisible. Ajouter un commentaire pour chaque groupe.

### Etape 1 : La Boucle Principale et la Lecture (Simple Shell 0.1)

C'est le coeur du shell : la boucle REPL.

```c
/* main.c */

#include "shell.h"

/**
 * main - point d'entree du simple shell
 * @argc: nombre d'arguments (non utilise)
 * @argv: tableau d'arguments (argv[0] = nom du shell)
 *
 * Return: 0 en cas de succes, code d'erreur sinon
 */
int main(int argc __attribute__((unused)), char *argv[])
{
    shell_loop(argv[0]);
    return (0);
}

/**
 * sigint_handler - gere le signal SIGINT (Ctrl+C)
 * @signum: numero du signal
 *
 * Description: au lieu de quitter, on affiche simplement
 * un nouveau prompt
 */
void sigint_handler(int signum __attribute__((unused)))
{
    write(STDOUT_FILENO, "\n", 1);
    write(STDOUT_FILENO, PROMPT, 2);
}

/**
 * shell_loop - boucle principale du shell (REPL)
 * @shell_name: nom du programme (pour messages d'erreur)
 */
void shell_loop(char *shell_name)
{
    char *line = NULL;
    size_t len = 0;
    ssize_t nread;
    char **argv;
    int status = 0;
    int line_count = 0;
    int interactive;

    interactive = isatty(STDIN_FILENO);
    signal(SIGINT, sigint_handler);

    while (1)
    {
        /* Afficher le prompt en mode interactif */
        if (interactive)
            write(STDOUT_FILENO, PROMPT, 2);

        /* Lire une ligne */
        nread = getline(&line, &len, stdin);

        /* Gerer EOF (Ctrl+D ou fin de fichier) */
        if (nread == -1)
        {
            if (interactive)
                write(STDOUT_FILENO, "\n", 1);
            free(line);
            exit(status);
        }

        line_count++;

        /* Supprimer le newline final */
        if (nread > 0 && line[nread - 1] == '\n')
            line[nread - 1] = '\0';

        /* Ignorer les lignes vides */
        if (line[0] == '\0')
            continue;

        /* Tokeniser la ligne */
        argv = tokenize(line);
        if (argv == NULL || argv[0] == NULL)
        {
            free(argv);
            continue;
        }

        /* Verifier les builtins */
        if (handle_builtin(argv, line) == 1)
        {
            free(argv);
            continue;
        }

        /* Executer la commande */
        status = execute(argv, shell_name, line_count);

        free(argv);
    }

    free(line);
}
```

> [!warning] getline et la memoire
> `getline()` alloue dynamiquement la memoire pour stocker la ligne lue. Elle peut reallouer si la ligne est plus grande que le buffer actuel. Il faut **toujours** liberer `line` a la fin (meme si on passe `NULL` initialement, `getline` alloue). Le pointeur `line` est reutilise a chaque iteration -- `getline` gere elle-meme la reallocation.

> [!example] getline en detail
> ```c
> char *line = NULL;   /* pointeur initialise a NULL */
> size_t len = 0;      /* taille du buffer = 0 */
> ssize_t nread;       /* nombre de caracteres lus */
> 
> nread = getline(&line, &len, stdin);
> /* getline fait :                                     */
> /* 1. Si line == NULL : alloue un buffer              */
> /* 2. Lit depuis stdin jusqu'a '\n' ou EOF            */
> /* 3. Stocke la ligne dans line (avec '\n' inclus)    */
> /* 4. Met a jour len avec la taille du buffer         */
> /* 5. Retourne le nombre de chars lus, ou -1 si EOF   */
> ```

**Gestion de Ctrl+C et Ctrl+D :**

```
+-------------------------------------------------------------------+
|                SIGNAUX CLAVIER                                    |
+-------------------------------------------------------------------+
|                                                                   |
|   Ctrl+C  -->  Signal SIGINT                                      |
|                Le shell NE DOIT PAS quitter                       |
|                Solution : installer un handler qui affiche         |
|                juste un nouveau prompt                             |
|                                                                   |
|   Ctrl+D  -->  EOF (End Of File)                                  |
|                getline retourne -1                                 |
|                Le shell DOIT quitter proprement                    |
|                Solution : free(line), exit(status)                 |
|                                                                   |
+-------------------------------------------------------------------+
```

> [!danger] Ne pas confondre Ctrl+C et Ctrl+D
> - `Ctrl+C` envoie le signal SIGINT. Par defaut, il tue le processus. On installe un handler pour l'ignorer dans le shell (mais le processus fils, lui, sera bien tue).
> - `Ctrl+D` ne produit pas de signal. Il ferme le flux d'entree (EOF). `getline` retourne `-1`.

### Etape 2 : Le Parsing (Tokenisation)

La tokenisation consiste a decouper la ligne en mots (tokens). Par exemple :

```
Entree :  "ls -la /tmp"

Sortie :  argv[0] = "ls"
          argv[1] = "-la"
          argv[2] = "/tmp"
          argv[3] = NULL
```

**Methode 1 : avec strtok (plus simple)**

```c
/* parser.c */

#include "shell.h"

/**
 * tokenize - decoupe une ligne en tokens
 * @line: la ligne a tokeniser
 *
 * Return: tableau de tokens (char **), ou NULL si erreur
 *         Le tableau est termine par NULL
 *
 * Note: les tokens pointent a l'interieur de @line
 *       (strtok modifie la chaine originale)
 */
char **tokenize(char *line)
{
    char **argv = NULL;
    char *token;
    int count = 0;
    int capacity = BUFSIZE;

    argv = malloc(sizeof(char *) * capacity);
    if (argv == NULL)
        return (NULL);

    token = strtok(line, DELIMITERS);

    while (token != NULL)
    {
        argv[count] = token;
        count++;

        /* Agrandir le tableau si necessaire */
        if (count >= capacity)
        {
            capacity += BUFSIZE;
            argv = realloc(argv, sizeof(char *) * capacity);
            if (argv == NULL)
                return (NULL);
        }

        token = strtok(NULL, DELIMITERS);
    }

    argv[count] = NULL;
    return (argv);
}
```

> [!warning] strtok modifie la chaine !
> `strtok` remplace les delimiteurs par `'\0'` dans la chaine originale. Apres `strtok("ls -la /tmp", " ")`, la chaine devient `"ls\0-la\0/tmp"`. Les tokens retournes pointent dans la chaine originale. C'est pourquoi on ne doit PAS free les tokens individuellement -- ils ne sont pas alloues separement.

> [!info] Fonctionnement de strtok
> ```
> Chaine originale : "ls -la /tmp"
> 
> Premier appel  : strtok(line, " ")   --> "ls"
>   line = "ls\0-la /tmp"
>              ^
>              strtok se souvient de cette position (variable statique interne)
> 
> Deuxieme appel : strtok(NULL, " ")   --> "-la"
>   line = "ls\0-la\0/tmp"
>                    ^
> 
> Troisieme appel : strtok(NULL, " ")   --> "/tmp"
>   line = "ls\0-la\0/tmp"
> 
> Quatrieme appel : strtok(NULL, " ")   --> NULL (plus de tokens)
> ```

**Methode 2 : tokenizer maison (plus de controle)**

```c
/* parser_custom.c */

#include "shell.h"

/**
 * count_tokens - compte le nombre de tokens dans une ligne
 * @line: la ligne
 * @delim: caractere delimiteur
 *
 * Return: nombre de tokens
 */
int count_tokens(const char *line, char delim)
{
    int count = 0;
    int in_token = 0;
    int i;

    for (i = 0; line[i] != '\0'; i++)
    {
        if (line[i] != delim && line[i] != '\t' && line[i] != '\n')
        {
            if (!in_token)
            {
                count++;
                in_token = 1;
            }
        }
        else
        {
            in_token = 0;
        }
    }

    return (count);
}

/**
 * tokenize_custom - decoupe une ligne en tokens (version maison)
 * @line: la ligne a tokeniser
 *
 * Return: tableau de tokens alloues dynamiquement, ou NULL
 */
char **tokenize_custom(char *line)
{
    char **argv;
    int i = 0, j, k, nb;

    nb = count_tokens(line, ' ');
    if (nb == 0)
        return (NULL);

    argv = malloc(sizeof(char *) * (nb + 1));
    if (argv == NULL)
        return (NULL);

    k = 0;
    while (line[i] != '\0' && k < nb)
    {
        /* Sauter les espaces */
        while (line[i] == ' ' || line[i] == '\t' || line[i] == '\n')
            i++;

        if (line[i] == '\0')
            break;

        /* Trouver la fin du token */
        j = i;
        while (line[j] != '\0' && line[j] != ' '
               && line[j] != '\t' && line[j] != '\n')
            j++;

        /* Copier le token */
        argv[k] = malloc(sizeof(char) * (j - i + 1));
        if (argv[k] == NULL)
        {
            /* Liberer ce qu'on a deja alloue */
            while (k > 0)
                free(argv[--k]);
            free(argv);
            return (NULL);
        }
        strncpy(argv[k], line + i, j - i);
        argv[k][j - i] = '\0';

        k++;
        i = j;
    }

    argv[k] = NULL;
    return (argv);
}
```

> [!tip] Quelle methode choisir ?
> Pour le projet Simple Shell de base, `strtok` est suffisant et plus simple. La version maison est utile si tu veux gerer des cas speciaux (guillemets, echappement) ou si tu veux eviter les problemes de la variable statique interne de `strtok`.

### Etape 3 : L'Execution Sans PATH (Simple Shell 0.1)

A cette etape, on n'execute que les commandes avec un chemin absolu ou relatif.

```c
/* executor.c */

#include "shell.h"

/**
 * execute - execute une commande
 * @argv: tableau d'arguments (argv[0] = commande)
 * @shell_name: nom du shell (pour messages d'erreur)
 * @line_count: numero de la ligne (pour messages d'erreur)
 *
 * Return: code de sortie de la commande executee
 */
int execute(char **argv, char *shell_name, int line_count)
{
    pid_t pid;
    int status;

    /* Verifier si la commande existe (si c'est un chemin) */
    if (argv[0][0] == '/' || argv[0][0] == '.')
    {
        if (access(argv[0], X_OK) != 0)
        {
            fprintf(stderr, "%s: %d: %s: not found\n",
                    shell_name, line_count, argv[0]);
            return (127);
        }
    }

    pid = fork();

    if (pid == -1)
    {
        perror("fork");
        return (1);
    }

    if (pid == 0)
    {
        /* FILS */
        if (execve(argv[0], argv, environ) == -1)
        {
            fprintf(stderr, "%s: %d: %s: not found\n",
                    shell_name, line_count, argv[0]);
            exit(127);
        }
    }
    else
    {
        /* PARENT */
        waitpid(pid, &status, 0);

        if (WIFEXITED(status))
            return (WEXITSTATUS(status));
    }

    return (0);
}
```

> [!example] Test de l'etape 3
> ```bash
> $ gcc -Wall -Werror -Wextra -pedantic -std=gnu89 *.c -o hsh
> $ ./hsh
> $ /bin/ls
> hsh  main.c  parser.c  executor.c  shell.h
> $ /bin/ls -la
> total 48
> drwxr-xr-x  2 user user 4096 ...
> ...
> $ /bin/echo hello
> hello
> $ qwerty
> ./hsh: 4: qwerty: not found
> $ 
> ```

### Etape 4 : Gestion des Arguments (Simple Shell 0.2)

Bonne nouvelle : si tu as bien implemente le tokenizer (Etape 2), les arguments sont deja geres ! Le tableau `argv` contient tous les tokens, y compris les arguments.

```
Commande : "/bin/ls -la /tmp"

Apres tokenization :
  argv[0] = "/bin/ls"    (la commande)
  argv[1] = "-la"        (premier argument)
  argv[2] = "/tmp"       (deuxieme argument)
  argv[3] = NULL          (terminateur)

execve("/bin/ls", argv, environ) recoit tout automatiquement.
```

> [!info] C'est la beaute de execve
> `execve` prend `argv` directement. Le programme execute (ici `ls`) recevra exactement ce tableau dans son `main(int argc, char *argv[])`. C'est pour ca que `argv[0]` est toujours le nom de la commande : c'est une convention UNIX.

### Etape 5 : Resolution du PATH (Simple Shell 0.3)

C'est l'etape la plus complexe. Quand l'utilisateur tape `ls` au lieu de `/bin/ls`, on doit trouver ou se trouve l'executable.

```c
/* path.c */

#include "shell.h"

/**
 * _strcat_path - concatene un repertoire et un nom de fichier
 * @dir: le repertoire (ex: "/usr/bin")
 * @file: le nom du fichier (ex: "ls")
 *
 * Return: chaine allouee "dir/file", ou NULL si erreur
 *         L'appelant doit liberer la memoire
 */
char *_strcat_path(const char *dir, const char *file)
{
    char *full_path;
    int dir_len, file_len;

    dir_len = strlen(dir);
    file_len = strlen(file);

    full_path = malloc(dir_len + 1 + file_len + 1);
    if (full_path == NULL)
        return (NULL);

    strcpy(full_path, dir);
    full_path[dir_len] = '/';
    strcpy(full_path + dir_len + 1, file);

    return (full_path);
}

/**
 * find_in_path - cherche un executable dans le PATH
 * @command: nom de la commande (ex: "ls")
 *
 * Return: chemin complet alloue (ex: "/bin/ls"), ou NULL si pas trouve
 *         L'appelant doit liberer la memoire
 */
char *find_in_path(const char *command)
{
    char *path_env;
    char *path_copy;
    char *dir;
    char *full_path;
    struct stat st;

    /* Recuperer la valeur de PATH */
    path_env = getenv("PATH");
    if (path_env == NULL || path_env[0] == '\0')
        return (NULL);

    /* Copier PATH car strtok va le modifier */
    path_copy = strdup(path_env);
    if (path_copy == NULL)
        return (NULL);

    /* Parcourir chaque repertoire du PATH */
    dir = strtok(path_copy, ":");

    while (dir != NULL)
    {
        /* Construire le chemin complet : dir/command */
        full_path = _strcat_path(dir, command);
        if (full_path == NULL)
        {
            free(path_copy);
            return (NULL);
        }

        /* Verifier si le fichier existe */
        if (stat(full_path, &st) == 0)
        {
            free(path_copy);
            return (full_path);  /* TROUVE ! */
        }

        free(full_path);
        dir = strtok(NULL, ":");
    }

    free(path_copy);
    return (NULL);  /* PAS TROUVE */
}
```

Maintenant on met a jour la fonction `execute` pour utiliser le PATH :

```c
/* executor.c -- version mise a jour */

#include "shell.h"

/**
 * execute - execute une commande (avec resolution PATH)
 * @argv: tableau d'arguments
 * @shell_name: nom du shell
 * @line_count: numero de ligne
 *
 * Return: code de sortie
 */
int execute(char **argv, char *shell_name, int line_count)
{
    pid_t pid;
    int status;
    char *full_path = NULL;

    /* Si la commande contient un '/', c'est un chemin direct */
    if (strchr(argv[0], '/') != NULL)
    {
        full_path = strdup(argv[0]);
    }
    else
    {
        /* Chercher dans le PATH */
        full_path = find_in_path(argv[0]);
    }

    /* Commande non trouvee ? */
    if (full_path == NULL || access(full_path, X_OK) != 0)
    {
        fprintf(stderr, "%s: %d: %s: not found\n",
                shell_name, line_count, argv[0]);
        free(full_path);
        return (127);
    }

    /* NE PAS fork si la commande n'existe pas (on a deja verifie) */
    pid = fork();

    if (pid == -1)
    {
        perror("fork");
        free(full_path);
        return (1);
    }

    if (pid == 0)
    {
        /* FILS : executer avec le chemin complet */
        if (execve(full_path, argv, environ) == -1)
        {
            perror(shell_name);
            free(full_path);
            exit(127);
        }
    }
    else
    {
        /* PARENT : attendre le fils */
        waitpid(pid, &status, 0);
        free(full_path);

        if (WIFEXITED(status))
            return (WEXITSTATUS(status));
    }

    return (0);
}
```

> [!warning] Points importants pour le PATH
> 1. **Copier** la valeur de PATH avant d'utiliser `strtok` (car `strtok` modifie la chaine)
> 2. **Liberer** la copie de PATH apres utilisation
> 3. **Liberer** `full_path` dans tous les cas (succes ou echec)
> 4. **Ne pas fork** si la commande n'existe pas -- verifier AVANT le fork
> 5. `strchr(argv[0], '/')` detecte si la commande contient un slash (chemin absolu ou relatif)

> [!example] Deroulement pour "ls -la"
> ```
> 1. argv[0] = "ls" (pas de '/')
> 2. find_in_path("ls")
>    PATH = "/usr/local/bin:/usr/bin:/bin"
>    
>    Tentative 1 : stat("/usr/local/bin/ls") --> ECHEC
>    Tentative 2 : stat("/usr/bin/ls")       --> ECHEC
>    Tentative 3 : stat("/bin/ls")           --> SUCCES !
>    
>    Retourne : "/bin/ls"
> 3. fork()
> 4. Fils : execve("/bin/ls", ["ls", "-la", NULL], environ)
> 5. Parent : waitpid(...)
> ```

### Etape 6 : Builtin exit (Simple Shell 0.4)

Les **builtins** sont des commandes executees directement par le shell, sans `fork` + `execve`. `exit` est le builtin le plus important : il termine le shell.

> [!tip] Pourquoi pas execve pour exit ?
> Si `exit` etait un programme externe, il terminerait le processus fils, pas le shell parent. On a besoin que le shell lui-meme se termine. C'est pourquoi les builtins sont executes dans le processus du shell directement.

```c
/* builtins.c */

#include "shell.h"

/**
 * handle_builtin - verifie si la commande est un builtin et l'execute
 * @argv: tableau d'arguments
 * @line: la ligne originale (pour la liberer si necessaire)
 *
 * Return: 1 si c'etait un builtin (a ete gere), 0 sinon
 */
int handle_builtin(char **argv, char *line)
{
    if (strcmp(argv[0], "exit") == 0)
    {
        builtin_exit(argv, line);
        return (1);  /* ne sera jamais atteint si exit reussit */
    }

    if (strcmp(argv[0], "env") == 0)
    {
        builtin_env();
        return (1);
    }

    return (0);  /* pas un builtin */
}

/**
 * builtin_exit - quitte le shell proprement
 * @argv: tableau d'arguments (argv[1] peut contenir le code de sortie)
 * @line: la ligne a liberer avant de quitter
 */
int builtin_exit(char **argv, char *line)
{
    int exit_status = 0;

    /* Si un argument est fourni, l'utiliser comme code de sortie */
    if (argv[1] != NULL)
        exit_status = atoi(argv[1]);

    /* Liberer la memoire avant de quitter */
    free(argv);
    free(line);

    exit(exit_status);
}
```

> [!warning] Liberer la memoire avant exit
> Quand on appelle `exit()`, le systeme libere automatiquement toute la memoire du processus. Cependant, pour passer les tests de `valgrind` et avoir un code propre, il est recommande de liberer explicitement `line` et `argv` avant d'appeler `exit()`.

### Etape 7 : Builtin env (Simple Shell 1.0)

Le builtin `env` affiche toutes les variables d'environnement.

```c
/* builtins.c -- ajout */

/**
 * builtin_env - affiche l'environnement
 *
 * Return: 0 toujours
 */
int builtin_env(void)
{
    int i = 0;

    while (environ[i] != NULL)
    {
        printf("%s\n", environ[i]);
        /* Alternative avec write : */
        /* write(STDOUT_FILENO, environ[i], strlen(environ[i])); */
        /* write(STDOUT_FILENO, "\n", 1);                        */
        i++;
    }

    return (0);
}
```

> [!example] Sortie attendue de env
> ```bash
> $ env
> PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
> HOME=/home/user
> USER=user
> SHELL=/bin/bash
> TERM=xterm-256color
> LANG=en_US.UTF-8
> PWD=/home/user/simple_shell
> ...
> $ 
> ```

---

## 6. Gestion de la Memoire

> [!danger] Les fuites memoire sont un probleme majeur
> Le shell tourne en boucle infinie. Si tu as une fuite a chaque iteration, la memoire consommee augmente sans cesse. C'est inacceptable. Valgrind est ton meilleur ami.

### 6.1 Quoi liberer ?

```
+-------------------------------------------------------------------+
|               MEMOIRE A GERER                                     |
+-------------------------------------------------------------------+
|                                                                   |
|   Allocation           | Quand liberer          | Qui libere     |
|   ---------------------|------------------------|----------------|
|   line (getline)       | A la fin (exit/EOF)    | shell_loop     |
|   argv (tokenize)      | Apres chaque commande  | shell_loop     |
|   full_path (find_*)   | Apres execve/erreur    | execute        |
|   path_copy (strdup)   | Apres recherche PATH   | find_in_path   |
|                                                                   |
+-------------------------------------------------------------------+
```

> [!info] Cycle de vie de line
> `line` est alloue par `getline` au premier appel. A chaque appel suivant, `getline` reutilise (et potentiellement realloue) le meme buffer. Il ne faut donc PAS liberer `line` a chaque iteration -- seulement a la sortie du shell.

### 6.2 Strategie de Liberation

```c
/*
 * Patron typique de la boucle :
 */
while (1)
{
    /* line est reutilise par getline (pas de free ici) */
    nread = getline(&line, &len, stdin);

    if (nread == -1)
    {
        free(line);      /* <-- liberer avant exit */
        exit(status);
    }

    argv = tokenize(line);      /* allocation */

    /* ... utiliser argv ... */

    free(argv);          /* <-- liberer apres chaque commande */
    /* NE PAS free les elements argv[i] si on utilise strtok */
    /* (ils pointent dans line qui est gere par getline) */
}
```

### 6.3 Utiliser Valgrind

```bash
# Verifier les fuites memoire
valgrind --leak-check=full --show-leak-kinds=all ./hsh

# Mode non-interactif (plus facile pour tester)
echo "ls" | valgrind --leak-check=full ./hsh

# Plusieurs commandes
echo -e "ls\nenv\nexit" | valgrind --leak-check=full ./hsh
```

> [!example] Sortie valgrind propre
> ```
> ==12345== HEAP SUMMARY:
> ==12345==     in use at exit: 0 bytes in 0 blocks
> ==12345==   total heap usage: 10 allocs, 10 frees, 2,048 bytes allocated
> ==12345== 
> ==12345== All heap blocks were freed -- no leaks are possible
> ==12345== 
> ==12345== ERROR SUMMARY: 0 errors from 0 contexts
> ```

### 6.4 Erreurs Frequentes de Memoire

```
+-------------------------------------------------------------------+
|               PIEGES MEMOIRE                                      |
+-------------------------------------------------------------------+
|                                                                   |
|  PIEGE 1 : Oublier de free line quand getline retourne -1        |
|  ---------------------------------------------------------        |
|  if (nread == -1)                                                 |
|      exit(status);    /* FUITE : line n'est pas libere ! */       |
|                                                                   |
|  CORRECTION :                                                     |
|  if (nread == -1)                                                 |
|  {                                                                |
|      free(line);                                                  |
|      exit(status);                                                |
|  }                                                                |
|                                                                   |
|  PIEGE 2 : Oublier de free full_path apres echec d'execve        |
|  -----------------------------------------------------------      |
|  /* Dans le fils, apres un execve qui echoue : */                 |
|  execve(full_path, argv, environ);                                |
|  perror("execve");                                                |
|  exit(127);    /* FUITE : full_path pas libere */                 |
|                                                                   |
|  CORRECTION :                                                     |
|  execve(full_path, argv, environ);                                |
|  perror("execve");                                                |
|  free(full_path);                                                 |
|  exit(127);                                                       |
|                                                                   |
|  PIEGE 3 : Free les tokens individuels avec strtok                |
|  -----------------------------------------------------------      |
|  /* Les tokens pointent DANS line, pas de malloc individuel */    |
|  free(argv[0]);   /* ERREUR : double free ou corruption ! */      |
|                                                                   |
|  CORRECTION : ne free que le tableau argv lui-meme                |
|  free(argv);      /* OK */                                        |
|                                                                   |
+-------------------------------------------------------------------+
```

---

## 7. Mode Non-Interactif

### 7.1 Detection

```c
int interactive = isatty(STDIN_FILENO);
```

`isatty()` retourne `1` si le file descriptor est connecte a un terminal, `0` sinon.

```
+-------------------------------------------------------------------+
|           INTERACTIF VS NON-INTERACTIF                            |
+-------------------------------------------------------------------+
|                                                                   |
|   INTERACTIF (terminal) :                                         |
|   +---------------------------+                                   |
|   | Terminal  --->  Shell     |   isatty(0) == 1                  |
|   | (clavier)      ./hsh     |   Afficher le prompt              |
|   +---------------------------+                                   |
|                                                                   |
|   NON-INTERACTIF (pipe) :                                         |
|   +---------------------------+                                   |
|   | echo "ls" | Shell        |   isatty(0) == 0                  |
|   | (pipe)        ./hsh      |   PAS de prompt                   |
|   +---------------------------+                                   |
|                                                                   |
|   NON-INTERACTIF (fichier) :                                      |
|   +---------------------------+                                   |
|   | cat cmds | Shell         |   isatty(0) == 0                  |
|   | (pipe)       ./hsh       |   PAS de prompt                   |
|   +---------------------------+                                   |
|                                                                   |
+-------------------------------------------------------------------+
```

### 7.2 Comportement Attendu

En mode non-interactif :
1. **Pas de prompt** affiché
2. Le shell lit les commandes de stdin jusqu'a EOF
3. Chaque ligne est executee normalement
4. A la fin (EOF), le shell se termine

```bash
# Test 1 : une seule commande
echo "/bin/ls" | ./hsh

# Test 2 : plusieurs commandes
echo -e "/bin/ls\n/bin/pwd\n/bin/echo hello" | ./hsh

# Test 3 : depuis un fichier
cat << 'EOF' > commands.txt
/bin/ls
/bin/pwd
/bin/echo hello world
EOF
cat commands.txt | ./hsh

# Test 4 : comparer avec /bin/sh
echo "/bin/ls" | ./hsh
echo "/bin/ls" | /bin/sh
# Les deux sorties doivent etre identiques
```

---

## 8. Gestion des Erreurs -- Format Exact

### 8.1 Format des Messages d'Erreur

Le format attendu est **strictement** celui de `/bin/sh`. Les checkers comparent la sortie caractere par caractere.

```
Format : <nom_du_shell>: <numero_ligne>: <commande>: not found
```

Exemples :

```bash
# Si le shell s'appelle ./hsh et qu'on tape "qwerty" a la ligne 3 :
./hsh: 3: qwerty: not found

# Si le shell est lance avec un chemin different :
/home/user/simple_shell/hsh: 1: blabla: not found
```

> [!danger] Le format DOIT etre exact
> Un espace en trop ou en moins et le checker echoue. Compare toujours ta sortie avec celle de `/bin/sh` :
> ```bash
> echo "qwerty" | /bin/sh 2>&1
> echo "qwerty" | ./hsh 2>&1
> # Les messages d'erreur doivent etre identiques
> # (sauf le nom du shell qui sera different)
> ```

### 8.2 Code de Sortie (Exit Status)

Le shell doit retourner le code de sortie de la derniere commande executee.

| Situation | Code de sortie |
|-----------|---------------|
| Commande reussie | `0` |
| Commande echouee | Le code retourne par la commande |
| Commande non trouvee | `127` |
| Permission refusee | `126` |
| Erreur de syntaxe | `2` |

```c
/* Recuperer le code de sortie du fils */
waitpid(pid, &status, 0);

if (WIFEXITED(status))
    last_status = WEXITSTATUS(status);
```

### 8.3 Ecrire sur stderr

Les messages d'erreur doivent aller sur **stderr** (fd 2), pas sur stdout (fd 1). Cela permet de separer la sortie normale des erreurs.

```c
/* Methode 1 : fprintf sur stderr */
fprintf(stderr, "%s: %d: %s: not found\n",
        shell_name, line_count, argv[0]);

/* Methode 2 : write sur STDERR_FILENO */
/* Plus complexe mais n'utilise pas de buffering */
write(STDERR_FILENO, shell_name, strlen(shell_name));
write(STDERR_FILENO, ": ", 2);
/* ... etc ... */
```

> [!tip] fprintf(stderr, ...) vs perror()
> - `perror("message")` affiche `message: <description de errno>`. Utile pour les appels systeme qui positionnent errno (fork, execve, malloc).
> - `fprintf(stderr, ...)` est plus flexible pour formater les messages comme on veut.

---

## 9. Structure du Projet Recommandee

```
simple_shell/
│
├── shell.h                 # Header principal
│   └── includes, extern, prototypes, macros
│
├── main.c                  # Point d'entree + boucle REPL
│   ├── main()              # Appelle shell_loop
│   ├── shell_loop()        # La boucle principale
│   └── sigint_handler()    # Gestion de Ctrl+C
│
├── parser.c                # Tokenisation
│   └── tokenize()          # Decoupe line en argv
│
├── executor.c              # Execution des commandes
│   └── execute()           # fork + execve + wait
│
├── path.c                  # Resolution du PATH
│   ├── find_in_path()      # Cherche commande dans PATH
│   └── _strcat_path()      # Concatene repertoire + fichier
│
├── builtins.c              # Commandes internes
│   ├── handle_builtin()    # Dispatch vers le bon builtin
│   ├── builtin_exit()      # Quitte le shell
│   └── builtin_env()       # Affiche l'environnement
│
├── helpers.c               # Fonctions utilitaires
│   ├── _strlen()           # Longueur d'une chaine
│   ├── _strcmp()            # Comparer deux chaines
│   └── _strdup()           # Dupliquer une chaine
│
├── man_1_simple_shell      # Page de manuel
├── AUTHORS                 # Noms des auteurs
└── README.md               # Description du projet
```

> [!warning] Regles Betty
> - Maximum **5 fonctions par fichier** `.c`
> - Maximum **40 lignes par fonction**
> - Pas de variables globales (sauf `environ` qui est `extern`)
> - Pas plus de **5 variables locales** par fonction
> - Commentaire de documentation pour chaque fonction
> - Style de code Betty (indentation, accolades, etc.)

> [!info] Compilation
> ```bash
> gcc -Wall -Werror -Wextra -pedantic -std=gnu89 *.c -o hsh
> ```
> Le flag `-std=gnu89` autorise certaines extensions GNU (comme `getline`) tout en restant proche du C89. Tous les warnings sont actives et traites comme des erreurs.

---

## 10. Tests

### 10.1 Tests Interactifs

Lance le shell et teste manuellement :

```bash
$ ./hsh
$ ls
AUTHORS  builtins.c  executor.c  helpers.c  hsh  main.c  parser.c  path.c  shell.h
$ ls -la
total 56
drwxr-xr-x  2 user user  4096 ...
-rw-r--r--  1 user user   500 ... AUTHORS
...
$ /bin/echo hello world
hello world
$ env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOME=/home/user
...
$ qwerty
./hsh: 5: qwerty: not found
$ exit
$
```

### 10.2 Tests Non-Interactifs

```bash
# Commande simple
echo "ls" | ./hsh

# Plusieurs commandes
echo -e "ls\npwd\necho hello" | ./hsh

# Commande avec chemin absolu
echo "/bin/ls -la" | ./hsh

# Commande inexistante
echo "qwerty" | ./hsh

# Comparer avec /bin/sh
echo "ls" | ./hsh > out_hsh 2>&1
echo "ls" | /bin/sh > out_sh 2>&1
diff out_hsh out_sh
```

### 10.3 Tests de Cas Limites

```bash
# Ligne vide (juste Enter)
echo "" | ./hsh

# Espaces seulement
echo "    " | ./hsh

# Ctrl+D (EOF)
echo -n "" | ./hsh

# Commande avec beaucoup d'espaces
echo "  ls   -la   /tmp  " | ./hsh

# PATH non defini
env -i ./hsh
# puis taper "ls" --> devrait afficher "not found"

# Commande avec chemin relatif
echo "./hsh" | ./hsh
```

### 10.4 Tests Valgrind

```bash
# Test basique
echo "ls" | valgrind --leak-check=full ./hsh

# Test avec plusieurs commandes
echo -e "ls\nenv\nexit" | valgrind --leak-check=full ./hsh

# Test avec commande inexistante
echo "qwerty" | valgrind --leak-check=full ./hsh

# Test sans PATH
echo "ls" | env -i valgrind --leak-check=full ./hsh
```

> [!tip] Script de test automatise
> ```bash
> #!/bin/bash
> # test_shell.sh - Tests automatiques
> 
> SHELL="./hsh"
> REF="/bin/sh"
> PASS=0
> FAIL=0
> 
> run_test() {
>     local cmd="$1"
>     local desc="$2"
>     
>     out_hsh=$(echo "$cmd" | $SHELL 2>&1)
>     out_sh=$(echo "$cmd" | $REF 2>&1)
>     
>     if [ "$out_hsh" = "$out_sh" ]; then
>         echo "[PASS] $desc"
>         ((PASS++))
>     else
>         echo "[FAIL] $desc"
>         echo "  Attendu : $out_sh"
>         echo "  Obtenu  : $out_hsh"
>         ((FAIL++))
>     fi
> }
> 
> run_test "/bin/ls" "ls avec chemin absolu"
> run_test "/bin/echo hello" "echo avec argument"
> run_test "ls" "ls sans chemin (PATH)"
> run_test "ls -la /tmp" "ls avec arguments"
> 
> echo ""
> echo "Resultats: $PASS passes, $FAIL echoues"
> ```

---

## 11. Erreurs Classiques

### 11.1 Liste des Pieges

```
+===================================================================+
|                 TOP 15 DES ERREURS CLASSIQUES                     |
+===================================================================+
|                                                                   |
|  1. OUBLIER environ DANS execve                                   |
|     ---------------------------------                              |
|     FAUX :  execve(path, argv, NULL);                             |
|     JUSTE : execve(path, argv, environ);                          |
|     Impact : le programme execute n'a aucune variable             |
|              d'environnement (PATH, HOME, etc.)                   |
|                                                                   |
|  2. NE PAS GERER LE RETOUR DE getline (-1)                       |
|     ---------------------------------                              |
|     Si getline retourne -1 (EOF), il faut free et exit.           |
|     Sinon : boucle infinie ou comportement indefini.              |
|                                                                   |
|  3. FUITES MEMOIRE A LA SORTIE                                    |
|     ---------------------------------                              |
|     Oublier de free(line) avant exit().                           |
|     Oublier de free(argv) avant exit().                           |
|                                                                   |
|  4. NE PAS VERIFIER LE RETOUR DE fork()                          |
|     ---------------------------------                              |
|     fork() peut echouer et retourner -1.                          |
|     Toujours verifier.                                            |
|                                                                   |
|  5. UTILISER system() AU LIEU DE execve                           |
|     ---------------------------------                              |
|     system() est INTERDIT dans ce projet.                         |
|     Elle appelle elle-meme fork+exec+wait en interne.             |
|                                                                   |
|  6. OUBLIER LE NULL A LA FIN DE argv                              |
|     ---------------------------------                              |
|     argv doit se terminer par NULL.                               |
|     Sans ca, execve a un comportement indefini.                   |
|                                                                   |
|  7. NE PAS LIBERER LES CHAINES DE PATH                           |
|     ---------------------------------                              |
|     find_in_path alloue full_path.                                |
|     Il faut le free apres utilisation (succes ou echec).          |
|                                                                   |
|  8. PATH VIDE OU NON DEFINI                                      |
|     ---------------------------------                              |
|     getenv("PATH") peut retourner NULL.                           |
|     Ou PATH peut etre vide ("").                                  |
|     Gerer ces cas.                                                |
|                                                                   |
|  9. LE SHELL QUITTE SUR Ctrl+C                                   |
|     ---------------------------------                              |
|     Par defaut, SIGINT tue le processus.                          |
|     Installer un handler avec signal().                           |
|                                                                   |
|  10. EXECVE SUR LE PARENT (PAS LE FILS)                          |
|      --------------------------------                              |
|      Appeler execve sans fork remplace le shell !                 |
|      Toujours fork d'abord, execve dans le fils.                  |
|                                                                   |
|  11. FORK MEME SI LA COMMANDE N'EXISTE PAS                       |
|      --------------------------------                              |
|      Verifier l'existence AVANT de fork.                          |
|      Fork inutile = processus gaspille.                           |
|                                                                   |
|  12. MAUVAIS FORMAT D'ERREUR                                     |
|      --------------------------------                              |
|      Doit etre : "nom_shell: ligne: cmd: not found\n"            |
|      Comparer avec /bin/sh.                                       |
|                                                                   |
|  13. NE PAS COPIER PATH AVANT strtok                             |
|      --------------------------------                              |
|      getenv("PATH") retourne un pointeur vers                     |
|      l'environnement. strtok le modifierait !                     |
|      Toujours strdup avant strtok.                                |
|                                                                   |
|  14. ECRIRE LES ERREURS SUR stdout AU LIEU DE stderr             |
|      --------------------------------                              |
|      Les messages d'erreur vont sur stderr (fd 2).                |
|      Utiliser fprintf(stderr, ...) ou write(2, ...).              |
|                                                                   |
|  15. PROCESSUS ZOMBIES                                            |
|      --------------------------------                              |
|      Oublier wait/waitpid apres fork.                             |
|      Le fils termine mais n'est pas "recolte".                    |
|                                                                   |
+===================================================================+
```

### 11.2 Debugging Pratique

```bash
# Compiler avec symboles de debug
gcc -g -Wall -Werror -Wextra -pedantic -std=gnu89 *.c -o hsh

# Debugger avec gdb
gdb ./hsh
(gdb) run
$ ls
(gdb) break execute   # point d'arret dans execute
(gdb) continue
$ ls
(gdb) print argv[0]   # inspecter la commande
(gdb) print full_path  # inspecter le chemin

# Tracer les appels systeme
strace ./hsh
# Affiche chaque appel systeme (fork, execve, wait, write, ...)

# Tracer uniquement certains appels
strace -e trace=process ./hsh
# Affiche seulement fork, execve, wait, exit, etc.
```

---

## 12. Fonctions Autorisees -- Rappel

Voici la liste **complete** des fonctions autorisees dans le projet :

| Fonction | Section man | Description | Usage dans le shell |
|----------|-------------|-------------|---------------------|
| `access` | man 2 | Verifie les permissions d'un fichier | Verifier si executable existe |
| `chdir` | man 2 | Change le repertoire courant | Builtin `cd` (avance) |
| `close` | man 2 | Ferme un file descriptor | Nettoyage |
| `closedir` | man 3 | Ferme un repertoire | Parcours repertoires |
| `execve` | man 2 | Execute un programme | **Execution commandes** |
| `exit` | man 3 | Termine le processus | Builtin `exit` |
| `_exit` | man 2 | Termine sans cleanup | Sortie du fils si execve echoue |
| `fflush` | man 3 | Vide le buffer de sortie | Avant fork |
| `fork` | man 2 | Cree un processus fils | **Creation processus** |
| `free` | man 3 | Libere la memoire | Partout |
| `getcwd` | man 3 | Obtient le repertoire courant | Builtin `cd` (avance) |
| `getline` | man 3 | Lit une ligne depuis un flux | **Lecture commandes** |
| `getpid` | man 2 | Obtient le PID | Debug |
| `isatty` | man 3 | Teste si fd est un terminal | **Detection mode interactif** |
| `kill` | man 2 | Envoie un signal | Gestion signaux |
| `malloc` | man 3 | Alloue de la memoire | Partout |
| `open` | man 2 | Ouvre un fichier | Lecture fichiers |
| `opendir` | man 3 | Ouvre un repertoire | Parcours repertoires |
| `perror` | man 3 | Affiche message d'erreur | Messages d'erreur |
| `read` | man 2 | Lit depuis un fd | Alternative a getline |
| `readdir` | man 3 | Lit entree de repertoire | Parcours repertoires |
| `signal` | man 2 | Gestion de signaux | **Ctrl+C** |
| `stat` | man 2 | Info sur un fichier | **Verifier existence** |
| `lstat` | man 2 | Info sur un lien symbolique | Liens symboliques |
| `fstat` | man 2 | Info sur un fd | Info fichiers |
| `strtok` | man 3 | Decoupe une chaine | **Tokenisation** |
| `wait` | man 2 | Attend un processus fils | **Attente commande** |
| `waitpid` | man 2 | Attend un fils specifique | **Attente commande** |
| `wait3` | man 2 | Wait avec info ressources | Alternative wait |
| `wait4` | man 2 | Waitpid avec info ressources | Alternative waitpid |
| `write` | man 2 | Ecrit sur un fd | **Affichage prompt/erreurs** |
| `fprintf` | man 3 | Ecriture formatee sur flux | Messages d'erreur |
| `printf` | man 3 | Ecriture formatee sur stdout | Affichage |
| `sprintf` | man 3 | Ecriture formatee dans chaine | Construction chaines |
| `putchar` | man 3 | Ecrit un caractere | Affichage |
| `getenv` | man 3 | Obtient variable d'environnement | **Lire PATH** |

> [!danger] Fonctions INTERDITES
> - `system()` -- elle fait fork+exec+wait elle-meme (c'est de la triche)
> - `exec*()` famille sauf `execve()` -- seul `execve` est autorise
> - Toute autre fonction non listee ci-dessus

---

## 13. Code Complet Recapitulatif

> [!info] Vue d'ensemble
> Voici un recapitulatif organise de tous les fichiers du projet, pour avoir une vision globale. Chaque fichier respecte la regle Betty des 5 fonctions maximum.

### shell.h (complet)

```c
#ifndef SHELL_H
#define SHELL_H

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <signal.h>
#include <errno.h>

extern char **environ;

#define PROMPT "$ "
#define BUFSIZE 1024
#define DELIMITERS " \t\n"

/* main.c */
void shell_loop(char *shell_name);
void sigint_handler(int signum);

/* parser.c */
char **tokenize(char *line);

/* executor.c */
int execute(char **argv, char *shell_name, int line_count);

/* path.c */
char *find_in_path(const char *command);
char *_strcat_path(const char *dir, const char *file);

/* builtins.c */
int handle_builtin(char **argv, char *line);
int builtin_exit(char **argv, char *line);
int builtin_env(void);

#endif /* SHELL_H */
```

### main.c (complet)

```c
#include "shell.h"

/**
 * main - point d'entree du simple shell
 * @argc: nombre d'arguments (non utilise)
 * @argv: tableau d'arguments
 *
 * Return: 0 en cas de succes
 */
int main(int argc __attribute__((unused)), char *argv[])
{
    shell_loop(argv[0]);
    return (0);
}

/**
 * sigint_handler - gere Ctrl+C (SIGINT)
 * @signum: numero du signal (non utilise)
 */
void sigint_handler(int signum __attribute__((unused)))
{
    write(STDOUT_FILENO, "\n", 1);
    write(STDOUT_FILENO, PROMPT, 2);
}

/**
 * shell_loop - boucle principale du shell
 * @shell_name: nom du programme pour les erreurs
 */
void shell_loop(char *shell_name)
{
    char *line = NULL;
    size_t len = 0;
    ssize_t nread;
    char **argv;
    int status = 0, line_count = 0, interactive;

    interactive = isatty(STDIN_FILENO);
    signal(SIGINT, sigint_handler);

    while (1)
    {
        if (interactive)
            write(STDOUT_FILENO, PROMPT, 2);

        nread = getline(&line, &len, stdin);
        if (nread == -1)
        {
            if (interactive)
                write(STDOUT_FILENO, "\n", 1);
            free(line);
            exit(status);
        }

        line_count++;
        if (nread > 0 && line[nread - 1] == '\n')
            line[nread - 1] = '\0';
        if (line[0] == '\0')
            continue;

        argv = tokenize(line);
        if (argv == NULL || argv[0] == NULL)
        {
            free(argv);
            continue;
        }

        if (handle_builtin(argv, line) == 1)
        {
            free(argv);
            continue;
        }

        status = execute(argv, shell_name, line_count);
        free(argv);
    }
    free(line);
}
```

### parser.c (complet)

```c
#include "shell.h"

/**
 * tokenize - decoupe une ligne en tokens
 * @line: la ligne a decouper
 *
 * Return: tableau de char* termine par NULL, ou NULL si erreur
 */
char **tokenize(char *line)
{
    char **argv = NULL;
    char *token;
    int count = 0;
    int capacity = BUFSIZE;

    argv = malloc(sizeof(char *) * capacity);
    if (argv == NULL)
        return (NULL);

    token = strtok(line, DELIMITERS);
    while (token != NULL)
    {
        argv[count] = token;
        count++;
        if (count >= capacity)
        {
            capacity += BUFSIZE;
            argv = realloc(argv, sizeof(char *) * capacity);
            if (argv == NULL)
                return (NULL);
        }
        token = strtok(NULL, DELIMITERS);
    }

    argv[count] = NULL;
    return (argv);
}
```

### executor.c (complet)

```c
#include "shell.h"

/**
 * execute - execute une commande externe
 * @argv: tableau d'arguments (argv[0] = commande)
 * @shell_name: nom du shell pour les erreurs
 * @line_count: numero de ligne pour les erreurs
 *
 * Return: code de sortie de la commande
 */
int execute(char **argv, char *shell_name, int line_count)
{
    pid_t pid;
    int status;
    char *full_path = NULL;

    if (strchr(argv[0], '/') != NULL)
        full_path = strdup(argv[0]);
    else
        full_path = find_in_path(argv[0]);

    if (full_path == NULL || access(full_path, X_OK) != 0)
    {
        fprintf(stderr, "%s: %d: %s: not found\n",
                shell_name, line_count, argv[0]);
        free(full_path);
        return (127);
    }

    pid = fork();
    if (pid == -1)
    {
        perror("fork");
        free(full_path);
        return (1);
    }

    if (pid == 0)
    {
        if (execve(full_path, argv, environ) == -1)
        {
            perror(shell_name);
            free(full_path);
            _exit(127);
        }
    }
    else
    {
        waitpid(pid, &status, 0);
        free(full_path);
        if (WIFEXITED(status))
            return (WEXITSTATUS(status));
    }

    return (0);
}
```

### path.c (complet)

```c
#include "shell.h"

/**
 * _strcat_path - concatene repertoire et nom de fichier avec '/'
 * @dir: le repertoire
 * @file: le nom de fichier
 *
 * Return: chaine allouee "dir/file", ou NULL
 */
char *_strcat_path(const char *dir, const char *file)
{
    char *path;
    int dlen, flen;

    dlen = strlen(dir);
    flen = strlen(file);
    path = malloc(dlen + 1 + flen + 1);
    if (path == NULL)
        return (NULL);

    strcpy(path, dir);
    path[dlen] = '/';
    strcpy(path + dlen + 1, file);

    return (path);
}

/**
 * find_in_path - cherche un executable dans PATH
 * @command: nom de la commande
 *
 * Return: chemin complet alloue, ou NULL si pas trouve
 */
char *find_in_path(const char *command)
{
    char *path_env, *path_copy, *dir, *full;
    struct stat st;

    path_env = getenv("PATH");
    if (path_env == NULL || path_env[0] == '\0')
        return (NULL);

    path_copy = strdup(path_env);
    if (path_copy == NULL)
        return (NULL);

    dir = strtok(path_copy, ":");
    while (dir != NULL)
    {
        full = _strcat_path(dir, command);
        if (full == NULL)
        {
            free(path_copy);
            return (NULL);
        }
        if (stat(full, &st) == 0)
        {
            free(path_copy);
            return (full);
        }
        free(full);
        dir = strtok(NULL, ":");
    }

    free(path_copy);
    return (NULL);
}
```

### builtins.c (complet)

```c
#include "shell.h"

/**
 * handle_builtin - verifie et execute un builtin
 * @argv: tableau d'arguments
 * @line: ligne originale (pour free si exit)
 *
 * Return: 1 si builtin gere, 0 sinon
 */
int handle_builtin(char **argv, char *line)
{
    if (strcmp(argv[0], "exit") == 0)
    {
        builtin_exit(argv, line);
        return (1);
    }
    if (strcmp(argv[0], "env") == 0)
    {
        builtin_env();
        return (1);
    }
    return (0);
}

/**
 * builtin_exit - quitte le shell
 * @argv: arguments (argv[1] = code de sortie optionnel)
 * @line: ligne a liberer
 *
 * Return: ne retourne jamais
 */
int builtin_exit(char **argv, char *line)
{
    int status = 0;

    if (argv[1] != NULL)
        status = atoi(argv[1]);

    free(argv);
    free(line);
    exit(status);
}

/**
 * builtin_env - affiche l'environnement
 *
 * Return: 0
 */
int builtin_env(void)
{
    int i = 0;

    while (environ[i] != NULL)
    {
        printf("%s\n", environ[i]);
        i++;
    }
    return (0);
}
```

---

## 14. Resume du Flux d'Execution

```
+===================================================================+
|                                                                   |
|   RESUME : LE CHEMIN D'UNE COMMANDE DANS LE SHELL                |
|                                                                   |
|   Utilisateur tape : "ls -la /tmp"                                |
|                                                                   |
|   1. [shell_loop]  isatty() ? oui --> affiche "$ "               |
|                                                                   |
|   2. [shell_loop]  getline() lit "ls -la /tmp\n"                  |
|                                                                   |
|   3. [shell_loop]  supprime le '\n' --> "ls -la /tmp"             |
|                                                                   |
|   4. [tokenize]    strtok decoupe :                               |
|                    argv[0] = "ls"                                  |
|                    argv[1] = "-la"                                 |
|                    argv[2] = "/tmp"                                |
|                    argv[3] = NULL                                  |
|                                                                   |
|   5. [handle_builtin]  "ls" != "exit" et != "env"                |
|                         --> pas un builtin (retourne 0)           |
|                                                                   |
|   6. [execute]     "ls" ne contient pas '/'                       |
|                    --> appelle find_in_path("ls")                 |
|                                                                   |
|   7. [find_in_path]  PATH="/usr/local/bin:/usr/bin:/bin"          |
|                       stat("/usr/local/bin/ls") --> ECHEC          |
|                       stat("/usr/bin/ls") --> ECHEC                |
|                       stat("/bin/ls") --> SUCCES                   |
|                       retourne "/bin/ls"                           |
|                                                                   |
|   8. [execute]     access("/bin/ls", X_OK) == 0 --> OK            |
|                                                                   |
|   9. [execute]     fork() --> PID fils = 4567                     |
|                                                                   |
|   10. [fils 4567]  execve("/bin/ls",                              |
|                           ["ls","-la","/tmp",NULL],               |
|                           environ)                                |
|                    --> le fils DEVIENT /bin/ls                     |
|                    --> ls affiche le contenu de /tmp               |
|                    --> ls termine avec exit(0)                     |
|                                                                   |
|   11. [parent]     waitpid(4567, &status, 0)                     |
|                    WEXITSTATUS(status) == 0                       |
|                    free(full_path)                                 |
|                    retourne 0                                      |
|                                                                   |
|   12. [shell_loop] free(argv)                                     |
|                    --> retour au debut de la boucle                |
|                    --> affiche "$ "                                |
|                                                                   |
+===================================================================+
```

---

## Carte Mentale

```
                          SIMPLE SHELL
                              |
            +-----------------+-----------------+
            |                 |                 |
       CONCEPTS           BOUCLE           IMPLEMENTATION
       SYSTEME             REPL
            |                 |                 |
    +-------+------+    +----+----+    +-------+-------+
    |       |      |    |    |    |    |       |       |
   PID   fork() PATH  Read Eval Print shell.h  Fichiers
   PPID  execve  env   |    |    |    main.c   Tests
   env   wait()       getline parse  parser.c  Valgrind
              |             tokenize executor.c
              |             builtin  path.c
              v             execute  builtins.c
    +-------------------+
    |  FORK+EXECVE+WAIT |
    |  = coeur du shell |
    +-------------------+
            |
    +-------+-------+
    |               |
  BUILTINS      COMMANDES
  (dans le      EXTERNES
   parent)      (fork+exec)
    |               |
  exit           find_in_path
  env            execve
                 waitpid
```

---

## Exercices

### Exercice 1 : Comprendre fork

Ecris un programme qui appelle `fork()` trois fois de suite (sans conditions). Combien de processus existent a la fin ? Dessine l'arbre des processus.

> [!example] Indice
> Apres chaque `fork()`, le nombre de processus double. Donc : 1 --> 2 --> 4 --> 8 processus.
> ```
> Apres fork 1 :  P1 --- P2
> Apres fork 2 :  P1 --- P2     P3 --- P4
> Apres fork 3 :  P1-P2  P3-P4  P5-P6  P7-P8  (8 processus)
> ```

### Exercice 2 : Tokenizer Manuel

Ecris une fonction `char **my_split(char *str, char delim)` qui decoupe une chaine selon un delimiteur, SANS utiliser `strtok`. La fonction doit :
1. Compter le nombre de tokens
2. Allouer le tableau de pointeurs
3. Allouer et copier chaque token individuellement
4. Terminer par NULL

Teste avec : `my_split("hello:world:foo", ':')` --> `{"hello", "world", "foo", NULL}`

### Exercice 3 : Mini-Shell en 50 Lignes

Ecris un shell minimal qui :
- Affiche un prompt `#cisfun$ `
- Lit une commande avec `getline`
- Execute la commande avec chemin absolu seulement (pas de PATH)
- Gere EOF (Ctrl+D)
- Boucle jusqu'a EOF

Ce shell ne gere PAS : les arguments (juste la commande seule), le PATH, les builtins, les erreurs elegantes. C'est l'exercice de base pour comprendre le pattern fork+execve+wait.

### Exercice 4 : Deboguer un Shell Bugge

Le code suivant contient **5 bugs**. Trouve-les et corrige-les :

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main(void)
{
    char *line;          /* BUG 1 : ??? */
    size_t len = 0;
    char *argv[64];
    int i;
    pid_t pid;

    while (1)
    {
        printf("$ ");
        getline(&line, &len, stdin);  /* BUG 2 : ??? */

        argv[0] = strtok(line, " \n");
        i = 1;
        while ((argv[i] = strtok(NULL, " \n")) != NULL)
            i++;

        pid = fork();
        execve(argv[0], argv, NULL);  /* BUG 3, 4 : ??? */
        wait(NULL);                   /* BUG 5 : ??? */
    }

    return (0);
}
```

> [!tip] Indices
> - BUG 1 : `line` doit etre initialise a quoi pour que `getline` alloue ?
> - BUG 2 : que se passe-t-il si `getline` retourne -1 ?
> - BUG 3 : `execve` est appele par qui (parent ou fils) ?
> - BUG 4 : le 3eme argument de `execve` devrait etre quoi ?
> - BUG 5 : `wait` est appele au mauvais endroit (et par qui ?)

> [!example] Solution
> ```c
> /* BUG 1 */ char *line = NULL;         /* initialiser a NULL */
> /* BUG 2 */ if (getline(...) == -1) { free(line); exit(0); }
> /* BUG 3 */ if (pid == 0) execve(...); /* seulement dans le fils */
> /* BUG 4 */ execve(argv[0], argv, environ); /* pas NULL */
> /* BUG 5 */ else waitpid(pid, NULL, 0);  /* dans le parent */
> ```

---

## Liens

- [[09 - File IO et Appels Systeme]] -- Les appels systeme read, write, open, close
- [[04 - Pointeurs et Memoire]] -- malloc, free, pointeurs doubles (char **)
- [[03c - Chaines de Caracteres]] -- Manipulation de chaines, strtok, strcpy
- [[05 - Pointeurs de Fonctions]] -- Utile pour la table de dispatch des builtins
- [[08 - Arguments Ligne de Commande]] -- argc, argv, comment un programme recoit ses arguments
- [[../Shell/01 - Shell et Commandes Linux]] -- Les commandes de base du shell (utilisateur)
