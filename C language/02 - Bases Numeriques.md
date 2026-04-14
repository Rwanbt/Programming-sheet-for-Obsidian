# 02 - Bases Numériques

## Les 3 systèmes de numération essentiels

En informatique, on travaille principalement avec **trois bases** :

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    LES 3 BASES ESSENTIELLES                            │
├─────────────────┬──────────────────┬────────────────────────────────────┤
│   BINAIRE       │   DÉCIMAL        │   HEXADÉCIMAL                     │
│   Base 2        │   Base 10        │   Base 16                         │
│   Chiffres: 0,1 │   Chiffres: 0-9  │   Chiffres: 0-9, A-F             │
│   Préfixe: 0b   │   (aucun)        │   Préfixe: 0x                     │
│                 │                  │                                    │
│   Le langage    │   Le langage     │   Le langage                      │
│   des machines  │   des humains    │   raccourci du binaire            │
└─────────────────┴──────────────────┴────────────────────────────────────┘
```

> [!info] Pourquoi ces bases ?
> - **Binaire (base 2)** : c'est le seul langage que le processeur comprend (courant / pas courant → 1 / 0)
> - **Décimal (base 10)** : le système naturel pour les humains (10 doigts)
> - **Hexadécimal (base 16)** : un raccourci compact pour écrire du binaire (1 chiffre hex = 4 bits)

---

## Table de correspondance complète (valeurs clés)

| Décimal | Binaire      | Hexadécimal | Octal | Signification                     |
| ------- | ------------ | ----------- | ----- | --------------------------------- |
| 0       | `0000 0000`  | `0x00`      | `000` | Zéro, NULL, faux                  |
| 1       | `0000 0001`  | `0x01`      | `001` | Un, vrai                          |
| 2       | `0000 0010`  | `0x02`      | `002` | 2^1                               |
| 4       | `0000 0100`  | `0x04`      | `004` | 2^2                               |
| 7       | `0000 0111`  | `0x07`      | `007` | Permissions rwx (chmod)           |
| 8       | `0000 1000`  | `0x08`      | `010` | 2^3                               |
| 10      | `0000 1010`  | `0x0A`      | `012` | Newline `\n` en ASCII             |
| 15      | `0000 1111`  | `0x0F`      | `017` | Demi-octet max (nibble)           |
| 16      | `0001 0000`  | `0x10`      | `020` | 2^4                               |
| 32      | `0010 0000`  | `0x20`      | `040` | Espace ASCII, 2^5                 |
| 48      | `0011 0000`  | `0x30`      | `060` | Caractère '0' en ASCII            |
| 64      | `0100 0000`  | `0x40`      | `100` | 2^6, '@' en ASCII                 |
| 65      | `0100 0001`  | `0x41`      | `101` | 'A' en ASCII                      |
| 97      | `0110 0001`  | `0x61`      | `141` | 'a' en ASCII                      |
| 127     | `0111 1111`  | `0x7F`      | `177` | Max signed char, DEL ASCII        |
| 128     | `1000 0000`  | `0x80`      | `200` | Min signed char (négatif), 2^7    |
| 255     | `1111 1111`  | `0xFF`      | `377` | Max unsigned char, octet plein    |

---

## Anatomie d'un octet (byte)

Un **octet** = **8 bits**. C'est l'unité fondamentale de stockage.

```
        MSB                                          LSB
  (Most Significant Bit)                    (Least Significant Bit)
        ▼                                              ▼
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │ Bit7│ Bit6│ Bit5│ Bit4│ Bit3│ Bit2│ Bit1│ Bit0│
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    128    64    32    16     8     4     2     1      ← Poids (2^n)

  Exemple : 0b01001101 = 0 + 64 + 0 + 0 + 8 + 4 + 0 + 1 = 77
```

| Terme        | Définition                                 |
| ------------ | ------------------------------------------ |
| **bit**      | 0 ou 1, unité minimale d'information       |
| **nibble**   | 4 bits (un chiffre hexadécimal)            |
| **octet**    | 8 bits = 1 byte                            |
| **word**     | 16, 32 ou 64 bits selon l'architecture     |
| **MSB**      | Bit de poids le plus fort (le plus à gauche) |
| **LSB**      | Bit de poids le plus faible (le plus à droite) |

---

## Méthodes de conversion

### Décimal → Binaire (divisions successives par 2)

On divise par 2 et on note les **restes** de bas en haut :

```
  77 ÷ 2 = 38  reste 1  ↑
  38 ÷ 2 = 19  reste 0  │
  19 ÷ 2 =  9  reste 1  │  Lire de bas en haut
   9 ÷ 2 =  4  reste 1  │
   4 ÷ 2 =  2  reste 0  │
   2 ÷ 2 =  1  reste 0  │
   1 ÷ 2 =  0  reste 1  │

  77 en binaire = 1001101
  Sur 8 bits    = 01001101
```

### Binaire → Décimal (somme des poids)

```
  01001101

  0×128 + 1×64 + 0×32 + 0×16 + 1×8 + 1×4 + 0×2 + 1×1
  =  0   +  64  +   0  +   0  +  8  +  4  +  0  +  1
  = 77
```

### Décimal → Hexadécimal (divisions successives par 16)

```
  255 ÷ 16 = 15  reste 15 (F)  ↑
   15 ÷ 16 =  0  reste 15 (F)  │  Lire de bas en haut

  255 en hexadécimal = 0xFF
```

```
  77 ÷ 16 = 4  reste 13 (D)  ↑
   4 ÷ 16 = 0  reste  4 (4)  │

  77 en hexadécimal = 0x4D
```

### Binaire ↔ Hexadécimal (groupes de 4 bits)

C'est la conversion la plus rapide ! Chaque chiffre hex correspond à **exactement 4 bits** :

```
  Binaire → Hex : regrouper par 4 depuis la droite
  01001101  →  0100  1101
                4      D     → 0x4D

  Hex → Binaire : remplacer chaque chiffre par 4 bits
  0xFF  →  F     F
           1111  1111  → 11111111
```

**Table de conversion rapide (à mémoriser)** :

| Hex | Bin    | Hex | Bin    |
| --- | ------ | --- | ------ |
| 0   | `0000` | 8   | `1000` |
| 1   | `0001` | 9   | `1001` |
| 2   | `0010` | A   | `1010` |
| 3   | `0011` | B   | `1011` |
| 4   | `0100` | C   | `1100` |
| 5   | `0101` | D   | `1101` |
| 6   | `0110` | E   | `1110` |
| 7   | `0111` | F   | `1111` |

---

## Addition binaire et overflow

### Addition binaire

Les règles sont simples :
```
  0 + 0 = 0
  0 + 1 = 1
  1 + 0 = 1
  1 + 1 = 10  (0, retenue 1)
  1 + 1 + 1 = 11  (1, retenue 1)
```

Exemple : `77 + 42 = 119`
```
    01001101   (77)
  + 00101010   (42)
  ──────────
    01110111   (119) ✓
```

### Overflow (dépassement sur 8 bits)

Sur un `unsigned char` (8 bits), la valeur maximale est `255`. Si on dépasse :

```
    11111111   (255)
  + 00000001   (  1)
  ──────────
  1 00000000   → Le 9ème bit est perdu ! Résultat = 0

  255 + 1 = 0  (overflow !)
```

> [!warning] L'overflow est silencieux en C !
> Le C ne signale **pas** d'erreur en cas d'overflow. La valeur "boucle" simplement. C'est une source fréquente de bugs graves (failles de sécurité).
>
> ```c
> unsigned char x = 255;
> x = x + 1;  /* x vaut maintenant 0 ! */
> ```

---

## Opérateurs bit à bit (bitwise)

Les opérateurs bit à bit travaillent sur **chaque bit individuellement**.

### AND (`&`) — Garde les bits communs

```
  1101  (13)          Règle :  1 & 1 = 1
& 1010  (10)                   1 & 0 = 0
──────                         0 & 1 = 0
  1000  ( 8)                   0 & 0 = 0
```

> [!tip] AND sert de **masque** : il "éteint" les bits qu'on ne veut pas voir.

### OR (`|`) — Allume les bits

```
  1101  (13)          Règle :  1 | 1 = 1
| 1010  (10)                   1 | 0 = 1
──────                         0 | 1 = 1
  1010  (15)                   0 | 0 = 0
```

### XOR (`^`) — Inverse quand différent

```
  1101  (13)          Règle :  1 ^ 1 = 0
^ 1010  (10)                   1 ^ 0 = 1
──────                         0 ^ 1 = 1
  0111  ( 7)                   0 ^ 0 = 0
```

> [!info] Propriété magique du XOR
> `A ^ A = 0` et `A ^ 0 = A`. Cela permet d'échanger deux variables sans variable temporaire !

### NOT (`~`) — Inverse tous les bits

```
~ 00001101  (13)
──────────
  11110010  (242 en unsigned, ou -14 en signed)
```

### Décalages (shifts)

```c
/* Décalage à gauche : multiplie par 2^n */
5 << 1  →  1010   /* 5 × 2 = 10 */
5 << 3  →  101000 /* 5 × 8 = 40 */

/* Décalage à droite : divise par 2^n */
40 >> 2  →  1010  /* 40 / 4 = 10 */
```

```
  Décalage gauche << 1 :
  ┌─┬─┬─┬─┬─┬─┬─┬─┐
  │0│0│0│0│0│1│0│1│  = 5
  └─┴─┴─┴─┴─┴─┴─┴─┘
         ← tout décale d'1 position, 0 entre à droite
  ┌─┬─┬─┬─┬─┬─┬─┬─┐
  │0│0│0│0│1│0│1│0│  = 10
  └─┴─┴─┴─┴─┴─┴─┴─┘
```

---

## Table des astuces bitwise

| Opération                   | Expression C              | Explication                         |
| --------------------------- | ------------------------- | ----------------------------------- |
| Tester le bit N             | `(x >> N) & 1`            | Décale puis masque                  |
| Mettre le bit N à 1 (set)  | `x \| (1 << N)`          | OR avec un masque                   |
| Mettre le bit N à 0 (clear)| `x & ~(1 << N)`          | AND avec masque inversé             |
| Inverser le bit N (toggle)  | `x ^ (1 << N)`           | XOR avec un masque                  |
| Vérifier pair/impair        | `x & 1`                  | 0 = pair, 1 = impair               |
| Multiplier par 2^n          | `x << n`                 | Plus rapide que `x * pow(2,n)`      |
| Diviser par 2^n             | `x >> n`                 | Division entière par puissance de 2 |
| Swap sans temp (XOR swap)   | `a^=b; b^=a; a^=b;`     | Échange via triple XOR              |

### Exemples en code C

```c
#include <stdio.h>

int main(void)
{
    unsigned char x = 0b00001101;  /* 13 */

    /* Tester le bit 3 */
    printf("Bit 3 = %d\n", (x >> 3) & 1);  /* 1 */

    /* Mettre le bit 1 à 1 */
    x = x | (1 << 1);  /* x = 0b00001111 = 15 */

    /* Mettre le bit 0 à 0 */
    x = x & ~(1 << 0);  /* x = 0b00001110 = 14 */

    /* Inverser le bit 2 */
    x = x ^ (1 << 2);  /* x = 0b00001010 = 10 */

    /* Vérifier pair/impair */
    if (x & 1)
        printf("Impair\n");
    else
        printf("Pair\n");  /* 10 est pair */

    /* XOR swap */
    int a = 5, b = 3;
    a ^= b;  /* a = 6 */
    b ^= a;  /* b = 5 */
    a ^= b;  /* a = 3 */
    printf("a = %d, b = %d\n", a, b);  /* a=3, b=5 */

    return (0);
}
```

---

## Applications dans le monde réel

### Adresses mémoire (hexadécimal)

```c
int x = 42;
printf("Adresse de x : %p\n", (void *)&x);
/* Affiche par ex. : 0x7ffd5e8c3a4c */
```

Les adresses mémoire sont affichées en hexadécimal car c'est compact et aligné sur les frontières de mots.

### Couleurs CSS (hexadécimal)

```
  #FF5733
   ││││││
   ││││└┘── Bleu  = 0x33 = 51
   ││└┘──── Vert  = 0x57 = 87
   └┘────── Rouge = 0xFF = 255
```

### Adresses IP et masques de sous-réseau

```
  IP:     192.168.1.10
  Binaire: 11000000.10101000.00000001.00001010

  Masque: 255.255.255.0   (/24)
  Binaire: 11111111.11111111.11111111.00000000

  Réseau = IP & Masque :
          11000000.10101000.00000001.00000000  = 192.168.1.0
```

### Permissions Unix (chmod) — octal

```
  chmod 755 fichier

  7 = 111 = rwx (propriétaire)
  5 = 101 = r-x (groupe)
  5 = 101 = r-x (autres)

  ┌───┬───┬───┐
  │ r │ w │ x │
  │ 4 │ 2 │ 1 │  ← Poids de chaque permission
  └───┴───┴───┘
```

### Chiffrement XOR

```c
char message[] = "HELLO";
char cle = 0x5A;

/* Chiffrement */
for (int i = 0; message[i]; i++)
    message[i] ^= cle;
/* message est maintenant chiffré (illisible) */

/* Déchiffrement (même opération !) */
for (int i = 0; message[i]; i++)
    message[i] ^= cle;
/* message est redevenu "HELLO" */
```

> [!info] XOR est sa propre inverse
> `(A ^ K) ^ K = A`. C'est la base du chiffrement XOR : appliquer deux fois la même clé redonne le message original.

### Magic bytes / signatures de fichiers

Les fichiers commencent par des octets magiques qui identifient leur format :

| Format | Magic bytes (hex)       | ASCII        |
| ------ | ----------------------- | ------------ |
| PDF    | `25 50 44 46`           | `%PDF`       |
| PNG    | `89 50 4E 47`           | `.PNG`       |
| ELF    | `7F 45 4C 46`           | `.ELF`       |
| JPEG   | `FF D8 FF`              | —            |
| ZIP    | `50 4B 03 04`           | `PK..`       |

---

## Notations C et affichage

### Notation dans le code source

```c
int dec = 42;        /* Décimal (pas de préfixe) */
int hex = 0x2A;      /* Hexadécimal (préfixe 0x) */
int oct = 052;       /* Octal (préfixe 0) */
int bin = 0b101010;  /* Binaire (préfixe 0b, extension GCC) */

/* Tous valent 42 ! */
```

> [!warning] Le piège du zéro initial
> ```c
> int x = 010;  /* Ce n'est PAS 10 ! C'est 8 en octal ! */
> ```
> Un nombre commençant par `0` est interprété en **octal** par le compilateur C.

### Affichage avec `printf`

```c
int val = 255;

printf("Décimal :      %d\n", val);   /* 255 */
printf("Hexadécimal :  %x\n", val);   /* ff */
printf("Hex majuscule: %X\n", val);   /* FF */
printf("Hex avec 0x :  %#x\n", val);  /* 0xff */
printf("Octal :        %o\n", val);   /* 377 */
printf("Octal avec 0 : %#o\n", val);  /* 0377 */
```

### Fonction `print_binary` (pas de `%b` en standard !)

Il n'existe pas de spécificateur `printf` pour le binaire en C standard. Voici comment en écrire un :

```c
#include <stdio.h>

/**
 * print_binary - Affiche un nombre en binaire
 * @n: Le nombre à afficher
 */
void print_binary(unsigned int n)
{
    int i;
    int started = 0;

    for (i = 31; i >= 0; i--)
    {
        if ((n >> i) & 1)
        {
            putchar('1');
            started = 1;
        }
        else if (started)
        {
            putchar('0');
        }
    }
    if (!started)
        putchar('0');
}

int main(void)
{
    print_binary(42);    /* Affiche : 101010 */
    printf("\n");
    print_binary(255);   /* Affiche : 11111111 */
    printf("\n");
    print_binary(0);     /* Affiche : 0 */
    printf("\n");

    return (0);
}
```

### Version compacte sur 8 bits

```c
/**
 * print_byte - Affiche un octet en binaire (8 bits)
 * @byte: L'octet à afficher
 */
void print_byte(unsigned char byte)
{
    int i;

    for (i = 7; i >= 0; i--)
        putchar(((byte >> i) & 1) + '0');
}
```

---

## Valeurs clés à mémoriser

### Puissances de 2

| 2^n   | Valeur       | Hex        |
| ----- | ------------ | ---------- |
| 2^0   | 1            | 0x1        |
| 2^1   | 2            | 0x2        |
| 2^2   | 4            | 0x4        |
| 2^3   | 8            | 0x8        |
| 2^4   | 16           | 0x10       |
| 2^7   | 128          | 0x80       |
| 2^8   | 256          | 0x100      |
| 2^10  | 1 024 (1 Ki) | 0x400      |
| 2^16  | 65 536       | 0x10000    |
| 2^20  | 1 048 576 (1 Mi) | 0x100000 |
| 2^32  | 4 294 967 296| 0x100000000|

### Valeurs sentinelles

| Valeur | Hex    | Signification                                  |
| ------ | ------ | ---------------------------------------------- |
| 0      | `0x00` | NULL, faux, fin de chaîne `\0`                 |
| 127    | `0x7F` | Max `signed char`                              |
| 128    | `0x80` | Bit de signe sur un octet                      |
| 255    | `0xFF` | Max `unsigned char`, octet plein               |
| 65535  | `0xFFFF` | Max `unsigned short`                         |
| -1     | `0xFFFFFFFF` | Tous les bits à 1 (32 bits, en complément à 2) |

---

## Manipulation avancée des bits

### Bit Fields en C

Les bit fields permettent de définir des champs de taille précise en bits dans une structure :

```c
/* Registre de flags CPU simulé */
typedef struct {
    unsigned int carry    : 1;  /* 1 bit */
    unsigned int zero     : 1;  /* 1 bit */
    unsigned int negative : 1;  /* 1 bit */
    unsigned int overflow : 1;  /* 1 bit */
    unsigned int reserved : 4;  /* 4 bits de padding */
} flags_t;

/* Permissions de fichier compactes */
typedef struct {
    unsigned int read    : 1;
    unsigned int write   : 1;
    unsigned int execute : 1;
} permission_t;

/* Usage */
flags_t flags = {0};
flags.carry = 1;
flags.zero = 0;
if (flags.carry)
    printf("Carry flag actif !\n");
```

> [!tip] Utilité
> Les bit fields sont utilisés en programmation embarquée, dans les protocoles réseau, et pour les registres matériels où chaque bit a une signification.

### Masques réseau (Subnetting)

Le subnetting utilise le AND bitwise pour déterminer si deux machines sont sur le même réseau :

```
IP        : 192.168.1.42    = 11000000.10101000.00000001.00101010
Masque /24: 255.255.255.0   = 11111111.11111111.11111111.00000000
AND       :                 = 11000000.10101000.00000001.00000000
Réseau    : 192.168.1.0     ← adresse du réseau

IP        : 192.168.1.100   = 11000000.10101000.00000001.01100100
Masque /24: 255.255.255.0   = 11111111.11111111.11111111.00000000
AND       :                 = 11000000.10101000.00000001.00000000
Réseau    : 192.168.1.0     ← même réseau ! Communication directe ✅
```

| Notation CIDR | Masque | Bits réseau | Hôtes possibles |
|---------------|--------|-------------|-----------------|
| /8 | 255.0.0.0 | 8 | 16 777 214 |
| /16 | 255.255.0.0 | 16 | 65 534 |
| /24 | 255.255.255.0 | 24 | 254 |
| /32 | 255.255.255.255 | 32 | 1 |

### Endianness (ordre des octets)

```
Valeur : 0x12345678

Big-Endian (réseau, SPARC, PowerPC) :
  Adresse : 0x00  0x01  0x02  0x03
  Valeur  :  12    34    56    78    ← octet de poids fort en premier

Little-Endian (x86, ARM, la plupart des PC) :
  Adresse : 0x00  0x01  0x02  0x03
  Valeur  :  78    56    34    12    ← octet de poids faible en premier
```

> [!warning] Attention
> L'endianness est source de bugs dans la communication réseau. Les protocoles réseau utilisent le Big-Endian ("network byte order"). Utilise `htons()`, `htonl()`, `ntohs()`, `ntohl()` pour convertir.

```c
#include <stdio.h>

/* Détecter l'endianness de ta machine */
int is_little_endian(void)
{
    unsigned int x = 1;
    return (*((unsigned char *)&x) == 1);
}
/* Si le premier octet vaut 1 → little-endian
   Si le premier octet vaut 0 → big-endian */
```

---

## Liens

- [[01 - Introduction au C et Compilation]] — Opérateurs et types de base
- [[04 - Pointeurs et Memoire]] — Adresses mémoire en hexadécimal

---

## Exercices pratiques

### Exercice 1 : Conversions manuelles
Convertir **à la main** (sans calculatrice) :
1. `42` en binaire
2. `0b11010110` en décimal
3. `0xBEEF` en binaire
4. `0b10101100` en hexadécimal
5. `chmod 644` : quelles permissions pour chaque catégorie ?

### Exercice 2 : `print_binary`
Écrire une fonction `void print_binary(unsigned long int n)` qui affiche la représentation binaire d'un nombre sans utiliser de tableau ni de `malloc`.

### Exercice 3 : Opérations bitwise
Sans exécuter le code, calculer le résultat de :
```c
unsigned char a = 0xCA;
unsigned char b = 0xFE;

printf("%X\n", a & b);
printf("%X\n", a | b);
printf("%X\n", a ^ b);
printf("%X\n", ~a & 0xFF);
printf("%X\n", a << 2 & 0xFF);
```

### Exercice 4 : Manipuler les bits
Écrire les fonctions suivantes :
```c
int get_bit(unsigned long int n, unsigned int index);
int set_bit(unsigned long int *n, unsigned int index);
int clear_bit(unsigned long int *n, unsigned int index);
unsigned int flip_bits(unsigned long int n);
```

### Exercice 5 : Convertisseur universel
Écrire un programme qui prend un nombre en argument et l'affiche en décimal, binaire, hexadécimal et octal.

### Exercice 6 : XOR chiffrement
Écrire un programme qui chiffre et déchiffre une chaîne avec une clé XOR passée en argument.
