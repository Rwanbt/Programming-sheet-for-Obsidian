# File I/O et Appels Systeme

> [!info] Contexte
> Ce chapitre marque le passage du cote "bas niveau" de la programmation C. On quitte le confort de `printf` et `fopen` pour utiliser directement les **appels systeme** (`open`, `read`, `write`, `close`) qui communiquent avec le noyau (Kernel). C'est une etape cle pour comprendre comment un programme interagit reellement avec le systeme d'exploitation.

---

## 1. Fonctions de bibliotheque vs Appels Systeme

| Aspect | Fonction de bibliotheque | Appel Systeme |
|--------|-------------------------|---------------|
| Exemples | `printf`, `fopen`, `fread` | `write`, `open`, `read`, `close` |
| Section du man | Section 3 (`man 3 printf`) | Section 2 (`man 2 open`) |
| Buffer interne | Oui (memoire tampon) | Non (acces direct) |
| Performance | Plus lente (overhead du buffer) | Plus rapide (direct au Kernel) |
| Facilite d'usage | Plus simple, formatage automatique | Plus complexe, bytes bruts |
| Portabilite | Tres portable (standard C) | Specifique POSIX (Unix/Linux) |

```
Ton code C
    |
    |-- printf("Hello")     --> [Bibliotheque C (libc)]  --> buffer --> write()
    |                                                                      |
    |-- write(1, "Hello", 5) ------------------------------------------->|
    |                                                                      |
    v                                                                      v
                                                                    [Kernel]
                                                                       |
                                                                    [Hardware]
```

> [!tip] Quand utiliser quoi ?
> - **printf/fopen** : Pour du code applicatif general, formatage, portabilite
> - **open/read/write** : Pour du code systeme, des projets bas niveau, ou quand la lib standard est interdite (comme chez Holberton)

---

## 2. Les File Descriptors (Descripteurs de fichier)

Un **File Descriptor (FD)** est simplement un **entier positif**. Quand tu ouvres un fichier, le systeme te donne un numero. C'est ton "ticket" pour interagir avec ce fichier.

### Les 3 FD standard

Par defaut, chaque processus possede trois fichiers deja ouverts :

| FD | Constante POSIX | Role |
|----|----------------|------|
| **0** | `STDIN_FILENO` | Entree standard (clavier) |
| **1** | `STDOUT_FILENO` | Sortie standard (ecran) |
| **2** | `STDERR_FILENO` | Sortie d'erreur (ecran, pour les erreurs) |

```c
#include <unistd.h>

/* Ces deux lignes font la meme chose : */
write(1, "Hello\n", 6);
write(STDOUT_FILENO, "Hello\n", 6);  /* Plus lisible ! */
```

> [!tip] Preference Holberton
> Toujours preferer les constantes symboliques (POSIX) aux nombres :
> `read(STDIN_FILENO, ...)` plutot que `read(0, ...)`

### Allocation des FD

Le systeme attribue toujours le **plus petit FD disponible** :

```
FD 0 : stdin  (toujours ouvert)
FD 1 : stdout (toujours ouvert)
FD 2 : stderr (toujours ouvert)
FD 3 : premier fichier que tu ouvres avec open()
FD 4 : deuxieme fichier...
...
```

---

## 3. open() : Ouvrir ou creer un fichier

```c
#include <fcntl.h>    /* Pour les flags O_RDONLY, etc. */
#include <sys/stat.h> /* Pour les permissions S_IRUSR, etc. */

int fd = open(filename, flags);
int fd = open(filename, flags, mode);  /* Si O_CREAT est utilise */
```

### Les flags (drapeaux)

| Flag | Description |
|------|-------------|
| `O_RDONLY` | Lecture seule (valeur : 0) |
| `O_WRONLY` | Ecriture seule (valeur : 1) |
| `O_RDWR` | Lecture ET ecriture (valeur : 2) |
| `O_CREAT` | Cree le fichier s'il n'existe pas |
| `O_TRUNC` | Ecrase le contenu si le fichier existe |
| `O_APPEND` | Ecrit a la fin du fichier |
| `O_EXCL` | Echoue si le fichier existe deja (avec O_CREAT) |

> [!warning] On combine les flags avec `|` (OR binaire)
> ```c
> /* Ouvrir en ecriture, creer si inexistant, ecraser si existant */
> fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
> ```

### Les permissions (mode)

Le troisieme argument n'est necessaire que si `O_CREAT` est utilise.

| Constante | Octal | Description |
|-----------|-------|-------------|
| `S_IRUSR` | 0400 | Lecture proprietaire |
| `S_IWUSR` | 0200 | Ecriture proprietaire |
| `S_IXUSR` | 0100 | Execution proprietaire |
| `S_IRGRP` | 0040 | Lecture groupe |
| `S_IWGRP` | 0020 | Ecriture groupe |
| `S_IROTH` | 0004 | Lecture autres |

```c
/* Permission 0644 : rw-r--r-- */
fd = open("file.txt", O_WRONLY | O_CREAT, S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);
/* Equivalent a : */
fd = open("file.txt", O_WRONLY | O_CREAT, 0644);
```

### Retour de open()

- **Succes** : retourne le FD (un entier >= 0)
- **Erreur** : retourne **-1** (et `errno` est positionne)

```c
int fd = open("inexistant.txt", O_RDONLY);
if (fd == -1)
{
    perror("Erreur open");  /* Affiche : "Erreur open: No such file or directory" */
    return (-1);
}
```

> [!info] Valeurs de O_RDONLY, O_WRONLY, O_RDWR
> Sur Ubuntu/Linux :
> - `O_RDONLY` = 0
> - `O_WRONLY` = 1
> - `O_RDWR` = 2
>
> Attention : `O_RDWR` n'est **PAS** `O_RDONLY | O_WRONLY` (0 | 1 = 1, pas 2 !)
> Ce sont des valeurs **exclusives**, pas des bits independants.

---

## 4. close() : Fermer un descripteur

```c
#include <unistd.h>

close(fd);
```

> [!warning] Toujours fermer les FD !
> Le systeme limite le nombre de FD ouverts simultanement.
> Ne pas fermer = fuite de ressources (comme ne pas `free` apres `malloc`).

---

## 5. read() : Lire des octets

```c
#include <unistd.h>

ssize_t bytes_read = read(fd, buffer, count);
```

| Parametre | Description |
|-----------|-------------|
| `fd` | Le descripteur de fichier a lire |
| `buffer` | Le tableau ou stocker les octets lus |
| `count` | Le nombre maximum d'octets a lire |

### Valeurs de retour

| Retour | Signification |
|--------|--------------|
| `> 0` | Nombre d'octets effectivement lus |
| `0` | **Fin de fichier (EOF)** atteinte |
| `-1` | **Erreur** (verifier `errno`) |

```c
char buffer[1024];
ssize_t n_read;

n_read = read(fd, buffer, 1024);
if (n_read == -1)
    perror("Erreur read");
else if (n_read == 0)
    printf("Fin du fichier\n");
else
    printf("Lu %zd octets\n", n_read);
```

> [!warning] read() ne met PAS de '\0' a la fin !
> Contrairement a `fgets`, `read` lit des octets bruts. Si tu veux traiter
> le buffer comme une chaine, ajoute manuellement le `'\0'` :
> ```c
> buffer[n_read] = '\0';
> ```

---

## 6. write() : Ecrire des octets

```c
#include <unistd.h>

ssize_t bytes_written = write(fd, buffer, count);
```

| Parametre | Description |
|-----------|-------------|
| `fd` | Le descripteur de fichier ou ecrire |
| `buffer` | Les donnees a ecrire |
| `count` | Le nombre d'octets a ecrire |

### Valeurs de retour

| Retour | Signification |
|--------|--------------|
| `>= 0` | Nombre d'octets effectivement ecrits |
| `-1` | **Erreur** (disque plein, pipe casse, etc.) |

```c
char *msg = "Hello, World!\n";
ssize_t written;

written = write(STDOUT_FILENO, msg, 14);
if (written == -1)
    perror("Erreur write");
```

---

## 7. lseek() : Repositionner le curseur

```c
#include <unistd.h>

off_t offset = lseek(fd, offset, whence);
```

| Whence | Description |
|--------|-------------|
| `SEEK_SET` | Depuis le **debut** du fichier |
| `SEEK_CUR` | Depuis la **position courante** |
| `SEEK_END` | Depuis la **fin** du fichier |

```c
/* Retourner au debut du fichier */
lseek(fd, 0, SEEK_SET);

/* Avancer de 10 octets */
lseek(fd, 10, SEEK_CUR);

/* Aller a la fin du fichier */
lseek(fd, 0, SEEK_END);

/* Connaitre la taille d'un fichier */
off_t size = lseek(fd, 0, SEEK_END);
lseek(fd, 0, SEEK_SET);  /* Revenir au debut */
```

---

## 8. Gestion des erreurs : errno et perror

Quand un appel systeme echoue (retourne -1), la variable globale `errno` est positionnee avec un code d'erreur.

```c
#include <errno.h>
#include <stdio.h>  /* Pour perror */
#include <string.h> /* Pour strerror */

int fd = open("inexistant.txt", O_RDONLY);
if (fd == -1)
{
    perror("open");
    /* Affiche : "open: No such file or directory" */

    /* Ou manuellement : */
    printf("Erreur %d : %s\n", errno, strerror(errno));
}
```

> [!tip] Toujours verifier les retours !
> Chaque appel a `open`, `read`, `write` et `close` peut echouer.
> Un programme robuste verifie **systematiquement** la valeur de retour.

---

## 9. Exemple complet : Copier un fichier

```c
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

#define BUFFER_SIZE 1024

/**
 * copy_file - Copie le contenu d'un fichier vers un autre.
 * @src: Nom du fichier source.
 * @dest: Nom du fichier destination.
 * Return: 0 si succes, -1 si erreur.
 */
int copy_file(const char *src, const char *dest)
{
    int fd_src, fd_dest;
    char buffer[BUFFER_SIZE];
    ssize_t n_read, n_written;

    /* Ouvrir le fichier source en lecture */
    fd_src = open(src, O_RDONLY);
    if (fd_src == -1)
    {
        perror("Erreur ouverture source");
        return (-1);
    }

    /* Creer/ouvrir le fichier destination en ecriture */
    fd_dest = open(dest, O_WRONLY | O_CREAT | O_TRUNC, 0664);
    if (fd_dest == -1)
    {
        perror("Erreur ouverture destination");
        close(fd_src);
        return (-1);
    }

    /* Boucle de lecture/ecriture */
    while ((n_read = read(fd_src, buffer, BUFFER_SIZE)) > 0)
    {
        n_written = write(fd_dest, buffer, n_read);
        if (n_written == -1)
        {
            perror("Erreur ecriture");
            close(fd_src);
            close(fd_dest);
            return (-1);
        }
    }

    if (n_read == -1)
        perror("Erreur lecture");

    /* Toujours fermer les descripteurs */
    close(fd_src);
    close(fd_dest);

    return (0);
}
```

---

## 10. Exemple : Lire un fichier ligne par ligne

```c
#include <unistd.h>
#include <fcntl.h>

/**
 * read_char_by_char - Lit un fichier caractere par caractere
 *                     et ecrit chaque ligne sur stdout.
 * @filename: Nom du fichier.
 */
void read_char_by_char(const char *filename)
{
    int fd;
    char c;
    ssize_t n;

    fd = open(filename, O_RDONLY);
    if (fd == -1)
    {
        perror("open");
        return;
    }

    while ((n = read(fd, &c, 1)) > 0)
    {
        write(STDOUT_FILENO, &c, 1);
    }

    close(fd);
}
```

> [!warning] Lire 1 octet a la fois est lent !
> Chaque appel a `read` est un appel systeme (passage User -> Kernel -> User).
> Pour de meilleures performances, lire par blocs de 1024 ou 4096 octets.

---

## 11. Comparaison : Appels systeme vs Bibliotheque

| Operation | Appel systeme (bas niveau) | Bibliotheque (haut niveau) |
|-----------|---------------------------|----------------------------|
| Ouvrir | `open("f.txt", O_RDONLY)` | `fopen("f.txt", "r")` |
| Lire | `read(fd, buf, size)` | `fread(buf, 1, size, fp)` |
| Ecrire | `write(fd, buf, size)` | `fwrite(buf, 1, size, fp)` |
| Fermer | `close(fd)` | `fclose(fp)` |
| Retour | `int` (file descriptor) | `FILE *` (pointeur de structure) |
| Buffer | Non (direct au Kernel) | Oui (buffer interne gere par la libc) |
| Formatage | Non | Oui (`fprintf`, `fscanf`) |

> [!info] `is open a function or a system call?`
> C'est les deux ! Et c'est aussi un **library call** :
> - **System Call** : demande au Kernel d'ouvrir un fichier (Section 2 du man)
> - **Function** : a un prototype, des arguments, une valeur de retour
> - **Library Call** : tu appelles une fonction de la libc qui "enveloppe" l'appel systeme

---

## 12. Exercices

> [!example] Exercice 1 : Lire et afficher
> Ecrire une fonction qui lit un fichier texte et affiche son contenu sur la sortie standard.
> Prototype : `int read_textfile(const char *filename, size_t letters);`
> La fonction doit lire au maximum `letters` caracteres.

> [!example] Exercice 2 : Creer un fichier
> Ecrire une fonction qui cree un fichier avec un contenu donne.
> Prototype : `int create_file(const char *filename, char *text_content);`
> Permissions : `rw-------` (0600)

> [!example] Exercice 3 : Ajouter a un fichier
> Ecrire une fonction qui ajoute du texte a la fin d'un fichier.
> Prototype : `int append_text_to_file(const char *filename, char *text_content);`
> Indice : utiliser `O_WRONLY | O_APPEND`

> [!example] Exercice 4 : Copier un fichier (programme complet)
> Ecrire un programme `cp` qui copie le contenu d'un fichier vers un autre.
> Usage : `cp file_from file_to`
> Gerer toutes les erreurs avec des messages sur stderr et des codes de sortie specifiques.

---

## 13. Liens

- [[01 - Introduction au C et Compilation]] - Les bases du C et de gcc
- [[04 - Pointeurs et Memoire]] - Comprendre les buffers et la gestion memoire
- [[01 - Shell et Commandes Linux]] - Les redirections (>, <, |) reposent sur les FD
- [[03 - Structures et Typedef]] - Les structures FILE utilisees par fopen
- [[10 - Projet Printf]] - Le projet _printf utilise write() pour l'affichage
