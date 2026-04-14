# Sécurité Mémoire en C

## Briefing de mission

> [!info] Objectif
> Apprendre à **détecter**, **analyser** et **corriger** les vulnérabilités mémoire en C. La gestion manuelle de la mémoire est la source n°1 de failles de sécurité dans les programmes C/C++.

En C, tu es responsable de chaque octet de mémoire. Il n'y a pas de garbage collector, pas de vérification automatique des bornes. Une erreur mémoire peut :
- Faire **crasher** le programme (segfault)
- Permettre à un attaquant d'**exécuter du code arbitraire** (buffer overflow exploit)
- **Fuiter des données sensibles** (lecture hors limites)
- Créer des comportements **imprévisibles** (undefined behavior)

---

## Les 3 grandes vulnérabilités

### 1. Buffer Overflow (Dépassement de tampon)

Un buffer overflow se produit quand tu écris **au-delà des limites** d'un tableau ou d'un buffer. C'est la vulnérabilité la plus exploitée dans l'histoire de la sécurité informatique.

```c
// VULNÉRABLE : gets() ne vérifie pas la taille
#include <stdio.h>

int main(void) {
    char buffer[10];
    gets(buffer);            // L'utilisateur tape 100 caractères → OVERFLOW
    printf("%s\n", buffer);
    return 0;
}
```

```c
// VULNÉRABLE : scanf sans limite
char name[20];
scanf("%s", name);           // Pas de limite → overflow possible
```

```c
// VULNÉRABLE : strcpy sans vérification
char dest[10];
char *src = "Cette chaîne est beaucoup trop longue pour le buffer";
strcpy(dest, src);           // Overflow !
```

> [!warning] Pourquoi c'est dangereux ?
> Un buffer overflow sur la pile (stack) peut écraser l'**adresse de retour** d'une fonction. Un attaquant peut alors rediriger l'exécution vers son propre code malveillant. C'est le principe des exploits classiques comme le **stack smashing**.

### 2. Dangling Pointer / Use-After-Free

Un **dangling pointer** est un pointeur qui pointe vers une zone mémoire qui a **déjà été libérée**. Utiliser ce pointeur (use-after-free) est un comportement indéfini.

```c
#include <stdlib.h>
#include <stdio.h>

int main(void) {
    int *ptr = malloc(sizeof(int));
    *ptr = 42;

    free(ptr);               // Mémoire libérée

    // VULNÉRABLE : use-after-free !
    printf("%d\n", *ptr);    // Comportement indéfini
    *ptr = 100;              // Écriture dans une zone libérée

    return 0;
}
```

> [!warning] Pourquoi c'est dangereux ?
> Après un `free`, la mémoire peut être réallouée à autre chose. Un attaquant peut contrôler ce qui est écrit dans cette zone et manipuler les données du programme.

**Correction** : Mettre le pointeur à `NULL` après `free` :

```c
free(ptr);
ptr = NULL;    // Maintenant, utiliser ptr causera un crash propre (segfault)
               // au lieu d'un comportement indéfini silencieux
```

### 3. Memory Leak (Fuite mémoire)

Une fuite mémoire se produit quand tu alloues de la mémoire sans jamais la libérer. Le programme consomme de plus en plus de mémoire jusqu'à en manquer.

```c
void leak_example(void) {
    while (1) {
        char *data = malloc(1024);
        // Utilise data...
        // Oublie de free(data) → fuite !
    }
    // Chaque itération perd 1 Ko de mémoire
}
```

> [!warning] Pourquoi c'est dangereux ?
> Un programme avec des fuites finira par consommer toute la mémoire disponible, causant un **déni de service** (DoS). Pour un serveur qui tourne 24/7, même une petite fuite est catastrophique.

---

## Fonctions dangereuses vs fonctions sûres

> [!warning] Tableau de remplacement obligatoire

| Fonction dangereuse | Pourquoi | Remplacement sûr | Exemple |
|---|---|---|---|
| `gets(buf)` | Aucune limite de taille | `fgets(buf, size, stdin)` | `fgets(buf, 100, stdin);` |
| `scanf("%s", buf)` | Aucune limite | `scanf("%99s", buf)` | `scanf("%99s", buf);` |
| `strcpy(dst, src)` | Pas de vérification de taille | `strncpy(dst, src, n)` | `strncpy(dst, src, sizeof(dst) - 1);` |
| `strcat(dst, src)` | Pas de vérification | `strncat(dst, src, n)` | `strncat(dst, src, sizeof(dst) - strlen(dst) - 1);` |
| `sprintf(buf, fmt, ...)` | Pas de vérification | `snprintf(buf, size, fmt, ...)` | `snprintf(buf, 100, "Hello %s", name);` |

### Exemples de correction

**Avant (dangereux)** :
```c
char buffer[100];
gets(buffer);                    // JAMAIS !
```

**Après (sûr)** :
```c
char buffer[100];
fgets(buffer, sizeof(buffer), stdin);
// Retirer le \n si nécessaire :
buffer[strcspn(buffer, "\n")] = '\0';
```

**Avant (dangereux)** :
```c
char dest[20];
strcpy(dest, source);            // Overflow si source > 19 chars
```

**Après (sûr)** :
```c
char dest[20];
strncpy(dest, source, sizeof(dest) - 1);
dest[sizeof(dest) - 1] = '\0';  // Garantir la terminaison
```

> [!info] Note sur `strncpy`
> `strncpy` ne garantit **pas** la terminaison par `\0` si la source est plus longue que `n`. Il faut toujours ajouter `dest[n-1] = '\0'` manuellement.

---

## Détection avec Valgrind

Valgrind est ton outil principal pour détecter les vulnérabilités mémoire. Voir [[02 - Valgrind]] pour un guide complet.

### La commande de scan

```bash
gcc -Wall -Werror -Wextra -g main.c -o prog
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes --verbose ./prog
```

### Sortie Valgrind par type de vulnérabilité

#### Buffer Overflow

```
==12345== Invalid write of size 1
==12345==    at 0x4C32E0A: strcpy (vg_replace_strmem.c:510)
==12345==    by 0x10917E: main (main.c:8)
==12345==  Address 0x522d04a is 0 bytes after a block of size 10 alloc'd
```

**Diagnostic** : Écriture **après** la fin d'un bloc de 10 octets. C'est un buffer overflow.

#### Use-After-Free

```
==12345== Invalid read of size 4
==12345==    at 0x10917E: main (main.c:10)
==12345==  Address 0x522d040 is 0 bytes inside a block of size 4 free'd
==12345==    at 0x4C30D3B: free (vg_replace_malloc.c:538)
==12345==    by 0x109172: main (main.c:8)
```

**Diagnostic** : Lecture d'un bloc qui a été libéré à la ligne 8. Use-after-free.

#### Memory Leak

```
==12345== 1,024 bytes in 10 blocks are definitely lost
==12345==    at 0x4C2FB0F: malloc (vg_replace_malloc.c:309)
==12345==    by 0x10916A: process_data (main.c:15)
==12345==    by 0x1091D3: main (main.c:30)
```

**Diagnostic** : 10 allocations de `malloc` (ligne 15) n'ont jamais été libérées. Fuite mémoire.

#### Variable non initialisée

```
==12345== Conditional jump or move depends on uninitialised value(s)
==12345==    at 0x10916A: check_access (main.c:22)
==12345==  Uninitialised value was created by a stack allocation
==12345==    at 0x109155: check_access (main.c:18)
```

**Diagnostic** : La variable déclarée ligne 18 est utilisée dans une condition ligne 22 sans avoir été initialisée.

---

## Les 4 règles d'or de malloc/free

> [!warning] Mémorise ces 4 règles

### Règle 1 : Toujours vérifier le retour de malloc

```c
// MAUVAIS
char *ptr = malloc(100);
strcpy(ptr, "Hello");     // Si malloc a échoué, ptr est NULL → crash

// BON
char *ptr = malloc(100);
if (ptr == NULL) {
    perror("malloc failed");
    return -1;
}
strcpy(ptr, "Hello");
```

### Règle 2 : Toujours initialiser après allocation

```c
// MAUVAIS
int *arr = malloc(10 * sizeof(int));
// arr contient des valeurs aléatoires (garbage)

// BON - Option 1 : calloc
int *arr = calloc(10, sizeof(int));   // Initialisé à 0

// BON - Option 2 : memset
int *arr = malloc(10 * sizeof(int));
if (arr)
    memset(arr, 0, 10 * sizeof(int));
```

### Règle 3 : Un malloc = un free

Chaque allocation doit avoir exactement un free correspondant. Pas zéro (fuite). Pas deux (double free).

### Règle 4 : Mettre le pointeur à NULL après free

```c
free(ptr);
ptr = NULL;    // Protection contre use-after-free
```

---

## Le concept d'Ownership (Propriété)

> [!info] Qui est responsable du `free()` ?

En C, il n'y a pas de garbage collector. Pour chaque bloc de mémoire alloué, tu dois définir clairement **qui est le propriétaire**, c'est-à-dire qui est responsable de le libérer.

### Règles d'ownership

1. **Celui qui alloue est propriétaire par défaut**
2. **La propriété peut être transférée** (documentée dans les commentaires ou le nom de la fonction)
3. **Un seul propriétaire à la fois**

```c
// La fonction crée et RETOURNE la mémoire
// → L'appelant devient propriétaire et doit free
char *create_greeting(const char *name) {
    char *greeting = malloc(100);
    if (!greeting)
        return NULL;
    snprintf(greeting, 100, "Hello, %s!", name);
    return greeting;   // Transfert de propriété à l'appelant
}

int main(void) {
    char *msg = create_greeting("Alice");  // main est propriétaire
    if (msg) {
        printf("%s\n", msg);
        free(msg);     // main libère (il est propriétaire)
    }
    return 0;
}
```

```c
// La fonction UTILISE la mémoire sans en prendre possession
// → L'appelant reste propriétaire
void print_greeting(const char *greeting) {  // const = "je ne modifie pas"
    printf("%s\n", greeting);
    // PAS de free ici ! On n'est pas propriétaire
}
```

> [!tip] Convention
> - Si une fonction **retourne** un pointeur alloué → elle transfère la propriété
> - Si une fonction **prend un pointeur en paramètre** → elle l'emprunte (pas de free)
> - Utilise `const` pour indiquer qu'un paramètre est en lecture seule

---

## Règles CERT C

Le CERT (Computer Emergency Response Team) publie des standards de codage sécurisé pour C. Voici les règles les plus importantes pour la mémoire.

### EXP33-C : Ne pas utiliser de variables non initialisées

```c
// VIOLATION
int x;
if (x > 0)          // x contient du garbage
    do_something();

// CONFORME
int x = 0;
if (x > 0)
    do_something();
```

### EXP34-C : Ne pas déréférencer un pointeur NULL

```c
// VIOLATION
char *ptr = get_data();
printf("%s\n", ptr);     // ptr pourrait être NULL

// CONFORME
char *ptr = get_data();
if (ptr != NULL)
    printf("%s\n", ptr);
```

### MEM30-C : Ne pas accéder à la mémoire libérée

```c
// VIOLATION
free(ptr);
*ptr = 42;               // Use-after-free

// CONFORME
free(ptr);
ptr = NULL;
```

### ENV30-C : Ne pas modifier la valeur retournée par getenv()

```c
// VIOLATION
char *path = getenv("PATH");
path[0] = '/';            // Modifie la variable d'environnement !

// CONFORME
char *path = getenv("PATH");
if (path) {
    char *copy = strdup(path);
    if (copy) {
        copy[0] = '/';    // Modifie la copie
        free(copy);
    }
}
```

---

## Flags de compilation pour la sécurité

```bash
# Compilation avec protections maximales
gcc -Wall -Werror -Wextra -g \
    -fsanitize=address \
    -fstack-protector-strong \
    -D_FORTIFY_SOURCE=2 \
    -O2 \
    main.c -o prog
```

| Flag | Protection |
|---|---|
| `-Wall -Werror -Wextra` | Warnings maximum, traités comme erreurs |
| `-g` | Symboles de debug (pour Valgrind et GDB) |
| `-fsanitize=address` | AddressSanitizer : détecte buffer overflow, use-after-free |
| `-fsanitize=undefined` | UBSan : détecte les comportements indéfinis |
| `-fstack-protector-strong` | Canary : détecte le stack smashing |
| `-D_FORTIFY_SOURCE=2` | Vérifie les fonctions de la libc (strcpy, sprintf, etc.) |
| `-O2` | Optimisations (requis pour `_FORTIFY_SOURCE`) |
| `-Wformat-security` | Avertit sur les chaînes de format non sûres |
| `-pie -fPIE` | Position Independent Executable (ASLR) |

---

## Rapport technique de sécurité

Pour chaque bug mémoire trouvé, réponds à ces 4 questions :

### Structure du rapport

> [!info] Les 4 questions par bug
> 1. **Quoi ?** - Quel type de vulnérabilité ? (Buffer overflow, use-after-free, leak, etc.)
> 2. **Où ?** - Quel fichier, quelle ligne, quelle fonction ?
> 3. **Pourquoi ?** - Quelle est la cause racine ? (buffer trop petit, oubli de free, pas de vérification NULL, etc.)
> 4. **Comment corriger ?** - Quelle est la solution précise ?

### Exemple de rapport

```
Bug #1 : Buffer Overflow
- Quoi : Écriture hors limites (Invalid write of size 1)
- Où : main.c:8, dans la fonction copy_input()
- Pourquoi : malloc(strlen(src)) au lieu de malloc(strlen(src) + 1),
  le '\0' est écrit après la fin du buffer
- Correction : Changer en malloc(strlen(src) + 1)

Bug #2 : Use-After-Free
- Quoi : Lecture d'un bloc libéré (Invalid read of size 4)
- Où : main.c:22, dans la fonction process_node()
- Pourquoi : Le noeud est libéré à la ligne 20, puis son champ
  ->next est lu à la ligne 22
- Correction : Sauvegarder node->next dans une variable temporaire
  avant de free(node)
```

---

## AddressSanitizer (ASan) - Alternative à Valgrind

AddressSanitizer est un outil de détection intégré au compilateur. Il est plus rapide que Valgrind mais nécessite une recompilation.

### Utilisation

```bash
# Compiler avec ASan
gcc -Wall -Werror -Wextra -g -fsanitize=address main.c -o prog

# Lancer normalement
./prog
```

ASan arrête le programme **immédiatement** quand il détecte un problème et affiche un rapport détaillé.

### Exemple de sortie ASan

```
=================================================================
==12345==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60200000001a
WRITE of size 1 at 0x60200000001a thread T0
    #0 0x7f... in strcpy (/lib/x86_64-linux-gnu/libc.so.6+0x...)
    #1 0x401234 in copy_string /home/user/main.c:8
    #2 0x401300 in main /home/user/main.c:15

0x60200000001a is located 0 bytes to the right of 10-byte region
[0x602000000010,0x60200000001a) allocated by thread T0 here:
    #0 0x7f... in malloc (/usr/lib/x86_64-linux-gnu/libasan.so.6+0x...)
    #1 0x401210 in copy_string /home/user/main.c:6
```

### Valgrind vs AddressSanitizer

| Aspect | Valgrind | AddressSanitizer |
|---|---|---|
| Recompilation nécessaire | Non | Oui (`-fsanitize=address`) |
| Ralentissement | ~20x | ~2x |
| Détection fuites | Excellent | Bon (avec `ASAN_OPTIONS=detect_leaks=1`) |
| Détection overflow | Bon | Excellent |
| Détection use-after-free | Bon | Excellent |
| Variables non initialisées | Oui (avec `--track-origins`) | Non (utiliser MSan) |
| Disponibilité | Linux | Linux, macOS, Windows |

---

## Tableau récapitulatif des remplacements sûrs

| Opération | Dangereux | Sûr |
|---|---|---|
| Lire une ligne | `gets(buf)` | `fgets(buf, size, stdin)` |
| Lire un mot | `scanf("%s", buf)` | `scanf("%99s", buf)` ou `fgets` |
| Copier une chaîne | `strcpy(d, s)` | `strncpy(d, s, n); d[n-1]='\0';` |
| Concaténer | `strcat(d, s)` | `strncat(d, s, n)` |
| Formatter | `sprintf(buf, ...)` | `snprintf(buf, size, ...)` |
| Allouer | `malloc(n)` | `malloc(n)` + vérifier retour |
| Allouer initialisé | `malloc(n)` + memset | `calloc(count, size)` |
| Libérer | `free(ptr)` | `free(ptr); ptr = NULL;` |
| Réallouer | `ptr = realloc(ptr, n)` | `tmp = realloc(ptr, n); if(tmp) ptr = tmp;` |

---

## Etude de cas : Secure Input & Memory Lab

Cette section presente un cas pratique complet issu d'un projet Holberton, avec des vulnerabilites reelles et leurs corrections.

### Architecture du programme vulnerable

```
src/
├── main.c          ← Point d'entree, orchestration
├── user_input.c    ← Lecture de l'entree utilisateur
├── user_input.h    ← Declaration de read_username()
├── session.c       ← Gestion de session (create/print/destroy)
└── session.h       ← Definition du type session_t
```

Flux de donnees :
```
[stdin] → read_username() → malloc() → session_create() → session_print()
       → session_destroy() → [free] → free(username)
```

### Vulnerabilite 1 : Buffer Overflow sur la pile

**Fichier** : `user_input.c`

```c
/* VULNERABLE */
char *read_username(void)
{
	char buffer[32];
	char *username = NULL;

	scanf("%s", buffer);                  /* DANGEREUX : pas de limite ! */
	username = malloc(strlen(buffer) + 1);
	if (username == NULL)
		return (NULL);
	strcpy(username, buffer);
	return (username);
}
```

**Probleme** : `buffer` fait 32 octets sur la pile. `scanf("%s", ...)` peut lire un input arbitrairement long. Si l'utilisateur entre 33+ caracteres, le buffer deborde et ecrase des donnees adjacentes sur la pile (variables locales, adresse de retour).

**Regles CERT violees** : STR31-C, ARR30-C, FIO42-C

```c
/* CORRIGE */
char *read_username(void)
{
	char buffer[32];
	char *username = NULL;
	size_t len;

	if (fgets(buffer, sizeof(buffer), stdin) == NULL)
		return (NULL);

	len = strlen(buffer);
	if (len > 0 && buffer[len - 1] == '\n')
		buffer[len - 1] = '\0';

	username = malloc(strlen(buffer) + 1);
	if (username == NULL)
		return (NULL);
	strcpy(username, buffer);
	return (username);
}
```

> [!tip] Pourquoi `fgets` est mieux que `scanf`
> - `fgets(buffer, sizeof(buffer), stdin)` lit au maximum `sizeof(buffer) - 1` caracteres
> - Garantit la null-termination
> - Preserve les espaces (contrairement a `scanf("%s", ...)`)
> - Retourne `NULL` en cas d'erreur (detectable)

### Vulnerabilite 2 : Use-After-Free et Double-Free

**Fichier** : `main.c`

```c
/* VULNERABLE */
int main(void)
{
	char *username = read_username();
	session_t *session = session_create(username);

	session_print(session);
	session_destroy(session);        /* free(session->user) = free(username) ! */

	printf("Goodbye %s\n", username); /* USE-AFTER-FREE : username deja libere */
	free(username);                   /* DOUBLE-FREE : deja libere dans destroy */

	return (0);
}
```

**Analyse du probleme d'ownership** :
1. `read_username()` alloue `username` → proprietaire initial : `main`
2. `session_create()` stocke le **meme pointeur** dans `session->user` (pas de copie) → ownership ambigu
3. `session_destroy()` libere `session->user` → considere qu'elle est proprietaire
4. `main()` utilise puis libere `username` → considere qu'elle est proprietaire
5. **Resultat** : deux proprietaires pour la meme allocation = double-free

**Correction A** - `session_create` fait une copie (ownership dans session) :

```c
session_t *session_create(char *username)
{
	session_t *session = malloc(sizeof(session_t));

	if (session == NULL)
		return (NULL);
	session->user = malloc(strlen(username) + 1);
	if (session->user == NULL)
	{
		free(session);
		return (NULL);
	}
	strcpy(session->user, username);
	session->access_level = 1;
	return (session);
}
```

**Correction B** - `session_destroy` ne libere PAS `session->user` (ownership dans main) :

```c
void session_destroy(session_t *session)
{
	if (session == NULL)
		return;
	/* NE PAS free(session->user) — main() garde l'ownership */
	free(session);
}
```

### Secure Data Handling : le store

Dans un programme avec un store (stockage en memoire), les memes problemes se repetent a plus grande echelle :

| Situation | Question d'ownership | Risque |
|---|---|---|
| `store_add(session)` echoue | Qui libere `session` ? | Memory leak si personne, double-free si les deux |
| `store_delete(id)` | Le store libere-t-il les membres de la session ? | Leak si non, dangling pointer si oui et qu'un autre pointeur existe |
| `store_clear()` | Tous les pointeurs externes sont-ils invalides ? | Use-after-free si on y accede apres |

> [!warning] Regle d'or du patching
> Un **bon** patch :
> - Enforce les invariants de taille (borne l'input)
> - Definit un owner unique pour chaque allocation
> - Garantit qu'une seule entite appelle `free()` sur chaque pointeur
>
> Un **mauvais** patch :
> - Ajoute des `free()` aleatoires
> - Agrandit les buffers sans limiter l'input
> - Corrige le symptome (l'erreur Valgrind) sans la cause racine

### Ce que Valgrind rapporte sur ce code

Avec un input normal (ex: "alice") :
```
==PID== Invalid read of size 1          ← use-after-free sur username
==PID== Address X is 8 bytes inside a block of size 9 free'd
         ← free'd dans session_destroy
==PID== Invalid free() / delete[]       ← double-free de username
```

Avec un input long (33+ caracteres) :
```
==PID== Invalid write of size 1         ← stack overflow dans buffer[32]
Segmentation fault
```

> [!tip] Conseil cle
> Se concentrer sur la **premiere** erreur dans le log Valgrind. Les erreurs suivantes sont souvent des effets en cascade du premier probleme.

---

## Exercices

### Exercice 1 : Identifier les vulnérabilités

Trouve **toutes** les vulnérabilités dans ce code :

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void process_input(void) {
    char buffer[10];
    int authenticated;

    printf("Enter password: ");
    gets(buffer);

    if (authenticated)
        printf("Access granted!\n");
}

int main(void) {
    char *data = malloc(50);
    strcpy(data, "Sensitive information");

    char *copy = data;
    free(data);
    printf("Data: %s\n", copy);

    return 0;
}
```

### Exercice 2 : Corriger le code

Réécris le code de l'exercice 1 en utilisant uniquement des fonctions sûres et en suivant toutes les règles de sécurité mémoire.

### Exercice 3 : Rapport de sécurité

Compile le code de l'exercice 1 et analyse-le avec :
1. Valgrind
2. AddressSanitizer (`-fsanitize=address`)
3. Rédige un rapport technique avec les 4 questions pour chaque bug trouvé.

---

**Liens connexes** : [[02 - Valgrind]] | [[04 - Pointeurs et Memoire]]
