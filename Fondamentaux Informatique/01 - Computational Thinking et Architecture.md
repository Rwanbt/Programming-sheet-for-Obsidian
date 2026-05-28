# 01 - Computational Thinking et Architecture

> [!info] Fondations du raisonnement informatique
> Ce cours couvre les deux socles indispensables avant d'écrire la première ligne de code : apprendre à **penser comme une machine** (computational thinking) et comprendre **comment une machine fonctionne vraiment** (architecture). Sans ces fondations, on code en aveugle.

---

## Partie 1 — Computational Thinking

### Les 4 piliers fondamentaux

Le computational thinking n'est pas une compétence réservée aux programmeurs. C'est une façon de décomposer et d'attaquer n'importe quel problème complexe avec méthode. Jeannette Wing, chercheuse chez Microsoft Research, a popularisé ce concept en 2006. Il repose sur 4 piliers :

```
┌──────────────────────────────────────────────────────────────────┐
│                    COMPUTATIONAL THINKING                        │
│                                                                  │
│  1. DÉCOMPOSITION        2. RECONNAISSANCE DE PATTERNS           │
│  Découper un problème    Identifier des similarités entre        │
│  complexe en sous-       problèmes pour réutiliser des           │
│  problèmes solubles      solutions connues                       │
│                                                                  │
│  3. ABSTRACTION          4. ALGORITHMIQUE                        │
│  Ignorer les détails     Définir une suite d'étapes              │
│  non pertinents pour     précises et non ambiguës                │
│  se concentrer sur       pour résoudre un problème               │
│  l'essentiel             de manière reproductible                │
└──────────────────────────────────────────────────────────────────┘
```

---

### Pilier 1 — Décomposition

**Définition** : Diviser un problème complexe en sous-problèmes plus petits, chacun solvable indépendamment.

**Exemple concret : Construire un jeu vidéo**

Sans décomposition, "créer un jeu" semble écrasant. Avec décomposition :

```
PROBLÈME : Créer un jeu de plateforme
│
├── Affichage
│   ├── Charger les sprites
│   ├── Dessiner l'arrière-plan
│   └── Afficher le personnage
│
├── Physique
│   ├── Gravité (le personnage tombe)
│   ├── Collision avec le sol
│   └── Collision avec les murs
│
├── Contrôles
│   ├── Détecter l'appui sur une touche
│   ├── Déplacer le personnage horizontalement
│   └── Faire sauter le personnage
│
└── Score
    ├── Incrémenter quand un ennemi est éliminé
    ├── Afficher le score à l'écran
    └── Sauvegarder le meilleur score
```

Chaque feuille de cet arbre est un problème simple. Résoudre les feuilles = résoudre l'arbre entier.

**Règle d'or** : Si un sous-problème vous paraît encore trop complexe, décomposez-le encore. Continuez jusqu'à obtenir des tâches qu'un programmeur débutant peut coder en moins d'une heure.

---

### Pilier 2 — Reconnaissance de patterns

**Définition** : Identifier des similarités, des régularités ou des structures répétitives entre problèmes ou à l'intérieur d'un même problème.

**Pourquoi c'est puissant** : Si vous reconnaissez qu'un nouveau problème ressemble à un problème déjà résolu, vous pouvez adapter la solution existante plutôt que de repartir de zéro.

**Exemple : Trier des données**

| Situation | Pattern reconnu | Solution réutilisée |
|-----------|-----------------|---------------------|
| Trier une liste de noms | Comparaison et échange | Algorithme de tri |
| Trier des produits par prix | Comparaison et échange | Même algorithme, critère différent |
| Trier des fichiers par date | Comparaison et échange | Même algorithme, critère différent |
| Ordonner un tournoi sportif | Comparaison et échange | Bracket = variante du même pattern |

**Autre exemple : Les boucles**

Dès que vous voyez "faire X pour chaque élément d'une liste", c'est un pattern de boucle. Peu importe si X est "afficher", "additionner", "filtrer" ou "transformer" — la structure de contrôle est toujours la même.

---

### Pilier 3 — Abstraction

**Définition** : Se concentrer sur les informations essentielles en ignorant délibérément les détails non pertinents pour le problème à résoudre.

> [!tip] Analogie
> Quand vous conduisez une voiture, vous utilisez le volant, l'accélérateur et le frein. Vous ne pensez pas aux cylindres, aux pistons, aux arbres à cames. L'ingénieur automobile vous a fourni une **abstraction** : une interface simple qui cache la complexité mécanique. C'est exactement ce que fait l'abstraction en programmation.

**Niveaux d'abstraction en informatique :**

```
Niveau 7 : Application (votre logiciel)
    ↑
Niveau 6 : API / Bibliothèques (functions prêtes à l'emploi)
    ↑
Niveau 5 : Système d'exploitation (gestion des ressources)
    ↑
Niveau 4 : Langage de programmation (C, Python...)
    ↑
Niveau 3 : Assembleur (instructions lisibles par l'humain)
    ↑
Niveau 2 : Code machine (0 et 1)
    ↑
Niveau 1 : Circuits logiques (transistors, portes AND/OR)
    ↑
Niveau 0 : Physique (électrons, tensions)
```

Quand vous écrivez `printf("Hello")`, vous opérez au niveau 6-7. Vous ne pensez pas aux transistors.

**Abstraction en design de code :**

```c
// MAUVAIS : aucune abstraction, la logique métier est noyée dans les détails
int main() {
    // 200 lignes de code entremêlées : lecture fichier, calcul, affichage
    FILE *f = fopen("data.csv", "r");
    char line[256];
    int total = 0;
    while (fgets(line, sizeof(line), f)) {
        // parser CSV manuellement...
        // calculer...
        // afficher...
    }
}

// BON : chaque fonction est une abstraction du problème
int main() {
    Data *data = load_csv("data.csv");    // détails de lecture cachés
    int total = compute_total(data);      // détails de calcul cachés
    display_result(total);               // détails d'affichage cachés
    free_data(data);
    return 0;
}
```

---

### Pilier 4 — Algorithmique

**Définition** : Définir une séquence d'étapes précises, non ambiguës et reproductibles qui mène à la solution d'un problème.

Un algorithme valide doit être :
- **Fini** : il s'arrête en un nombre fini d'étapes
- **Défini** : chaque étape est précise et sans ambiguïté
- **Efficace** : chaque étape est réalisable (pas "diviser par zéro")
- **Entrées/Sorties** : il produit un résultat à partir d'entrées

**Exemple : Chercher un mot dans un dictionnaire**

```
Algorithme : Recherche par dichotomie dans un dictionnaire papier

ENTRÉE : un dictionnaire D, un mot M à chercher
SORTIE : la page où se trouve M, ou "absent"

1. Ouvrir le dictionnaire à la page du MILIEU
2. Lire le mot sur cette page (notons-le W)
3. SI W = M :
       → RETOURNER le numéro de page  (trouvé !)
   SINON SI M vient AVANT W alphabétiquement :
       → Ignorer la moitié droite du dictionnaire
       → Reprendre à l'étape 1 sur la moitié gauche
   SINON (M vient APRÈS W) :
       → Ignorer la moitié gauche du dictionnaire
       → Reprendre à l'étape 1 sur la moitié droite
4. SI le dictionnaire restant est vide :
       → RETOURNER "absent"
```

> [!tip] Pourquoi c'est efficace ?
> Un dictionnaire de 100 000 mots : la recherche linéaire (page par page) nécessite jusqu'à 100 000 comparaisons. La dichotomie n'en nécessite que log₂(100 000) ≈ **17 comparaisons**. Un facteur 6000x de différence !

---

### Exercices de pensée computationnelle sans ordinateur

> [!warning] L'objectif de ces exercices
> Avant de toucher un clavier, entraînez votre cerveau à penser algorithmiquement. Ces exercices se font sur papier, en groupe, ou mentalement.

**Exercice 1 : Trier un jeu de cartes**

Prenez 10 cartes d'un jeu de 52. Mélangez-les. Objectif : les trier par valeur en un minimum de comparaisons.

Questions à se poser :
1. Quel est votre algorithme ? Décrivez-le étape par étape en français.
2. Combien de comparaisons avez-vous effectuées ?
3. Si vous doublez le nombre de cartes (20), combien de comparaisons prévoyez-vous ?
4. Quelle est votre stratégie optimale ?

**Réflexion** : Vous venez d'implémenter mentalement un algorithme de tri. Peut-être le tri à bulles (comparer deux à deux, échanger si nécessaire, répéter), peut-être le tri par insertion (insérer chaque carte à sa bonne position dans une main déjà triée). Chacun a ses caractéristiques de performance.

**Exercice 2 : La recette comme algorithme**

Prenez n'importe quelle recette de cuisine. Identifiez :
1. Les **entrées** (ingrédients, quantités)
2. Les **étapes** (chaque instruction)
3. Les **conditions** ("si la pâte est trop collante, ajouter de la farine")
4. Les **boucles** ("remuer pendant 5 minutes")
5. Les **sorties** (le plat final)

Réécrivez-la en pseudo-code structuré :

```
ALGORITHME : Faire des crêpes

ENTRÉES :
    farine = 250g
    oeufs = 3
    lait = 500ml
    sel = 1 pincée

ÉTAPES :
1. Mettre farine dans un saladier
2. Faire un puits au centre
3. Casser les œufs dans le puits
4. RÉPÉTER (mélanger farine + œufs) JUSQU'À (mélange homogène)
5. Ajouter le lait petit à petit
6. RÉPÉTER (mélanger) JUSQU'À (pâte lisse sans grumeaux)
7. Ajouter le sel
8. Laisser reposer 30 minutes
9. TANT QUE (il reste de la pâte) FAIRE :
       a. Chauffer poêle
       b. Verser louche de pâte
       c. ATTENDRE 2 minutes
       d. SI (bords dorés) ALORS retourner la crêpe
       e. ATTENDRE 1 minute supplémentaire
       f. Retirer la crêpe

SORTIE : crêpes prêtes à déguster
```

---

### Introduction à Scratch — La logique visuelle

Scratch (scratch.mit.edu) est un environnement de programmation visuelle développé par le MIT. Les programmes sont construits en assemblant des **blocs colorés** qui s'emboîtent comme des pièces de puzzle, éliminant les erreurs de syntaxe pour se concentrer sur la logique.

**Les grandes catégories de blocs :**

| Catégorie | Couleur | Rôle | Exemples |
|-----------|---------|------|---------|
| Événements | Jaune | Déclencher une action | "Quand ⚑ cliqué", "Quand touche [espace] pressée" |
| Contrôle | Orange | Structure du programme | `répéter 10 fois`, `si...alors...sinon`, `attendre 1 secondes` |
| Mouvement | Bleu | Déplacer le sprite | `avancer de 10 pas`, `aller à x:0 y:0`, `tourner de 90°` |
| Apparence | Violet | Modifier l'affichage | `dire "Bonjour!"`, `basculer sur costume`, `montrer` |
| Son | Rose | Jouer des sons | `jouer son [miaou]`, `arrêter tous les sons` |
| Variables | Orange foncé | Stocker des données | `mettre [score] à 0`, `ajouter 1 à [score]` |
| Opérateurs | Vert | Calculs et comparaisons | `+ - × /`, `= < >`, `et`, `ou`, `non` |

**Exemple : Personnage qui rebondit sur les bords**

```
Quand ⚑ cliqué
  répéter indéfiniment
    avancer de 5 pas
    rebondir si le bord est atteint
  fin répéter
```

**Exemple : Compteur de score**

```
Quand ⚑ cliqué
  mettre [score] à 0   ← initialisation

Quand je reçois [ennemi touché]
  ajouter 10 à [score]   ← incrémentation
  dire [score] pendant 1 secondes
```

**Structure si/sinon/si-sinon :**

```
Quand ⚑ cliqué
  répéter indéfiniment
    si <touche [droite] pressée ?> alors
      ajouter 5 à x
    sinon
      si <touche [gauche] pressée ?> alors
        ajouter -5 à x
      fin si
    fin si
  fin répéter
```

**Broadcasting (messages entre sprites) :**

Le broadcasting permet à un sprite d'envoyer un message que d'autres sprites peuvent recevoir. C'est l'équivalent des **événements** en programmation réelle.

```
Sprite Joueur :
  si <touche [ennemi] ?> alors
    envoyer à tous [collision]
  fin si

Sprite Ennemi :
  quand je reçois [collision]
    cacher
    ajouter 1 à [vie_perdue]

Sprite Interface :
  quand je reçois [collision]
    jouer son [ouch]
    dire "Aïe!" pendant 0.5 secondes
```

**Fonctions personnalisées (Mes blocs) :**

Scratch permet de créer ses propres blocs pour éviter la répétition de code :

```
Définir [dessiner carré de taille (côté)]
  répéter 4 fois
    avancer de (côté) pas
    tourner de 90 degrés
  fin répéter

─────────────────────────────────

Quand ⚑ cliqué
  dessiner carré de taille 50
  aller à x:100 y:0
  dessiner carré de taille 100
  aller à x:-100 y:0
  dessiner carré de taille 75
```

---

### Du Scratch au pseudo-code au code réel

Le même problème exprimé dans les 3 niveaux de formalisme :

**Problème : Deviner un nombre entre 1 et 100**

**En Scratch (blocs visuels) :**
```
Quand ⚑ cliqué
  mettre [secret] à (nombre aléatoire entre 1 et 100)
  mettre [tentatives] à 0
  répéter indéfiniment
    demander [Devinez un nombre entre 1 et 100 :] et attendre
    ajouter 1 à [tentatives]
    si <(réponse) = (secret)> alors
      dire [Bravo ! Trouvé en ] + (tentatives) + [ coups !] pendant 2 secondes
      arrêter [ce script]
    sinon
      si <(réponse) < (secret)> alors
        dire [Trop petit !] pendant 1 secondes
      sinon
        dire [Trop grand !] pendant 1 secondes
      fin si
    fin si
  fin répéter
```

**En pseudo-code :**
```
ALGORITHME : Devine un nombre

secret ← nombre_aléatoire(1, 100)
tentatives ← 0

RÉPÉTER INDÉFINIMENT :
    afficher "Devinez un nombre entre 1 et 100 :"
    guess ← lire_entier()
    tentatives ← tentatives + 1

    SI guess = secret ALORS
        afficher "Bravo ! Trouvé en " + tentatives + " coups !"
        ARRÊTER
    SINON SI guess < secret ALORS
        afficher "Trop petit !"
    SINON
        afficher "Trop grand !"
    FIN SI
FIN RÉPÉTER
```

**En C :**
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main(void)
{
    srand(time(NULL));
    int secret = (rand() % 100) + 1;  /* nombre entre 1 et 100 */
    int tentatives = 0;
    int guess;

    do {
        printf("Devinez un nombre entre 1 et 100 : ");
        scanf("%d", &guess);
        tentatives++;

        if (guess == secret) {
            printf("Bravo ! Trouvé en %d coups !\n", tentatives);
            return 0;
        } else if (guess < secret) {
            printf("Trop petit !\n");
        } else {
            printf("Trop grand !\n");
        }
    } while (1);
}
```

**En Python :**
```python
import random

def jeu_devine_nombre():
    secret = random.randint(1, 100)
    tentatives = 0

    while True:
        guess = int(input("Devinez un nombre entre 1 et 100 : "))
        tentatives += 1

        if guess == secret:
            print(f"Bravo ! Trouvé en {tentatives} coups !")
            break
        elif guess < secret:
            print("Trop petit !")
        else:
            print("Trop grand !")

jeu_devine_nombre()
```

> [!tip] Observation clé
> La **structure logique** est identique dans les 3 versions. Seule la **syntaxe** change. Une fois la logique maîtrisée en pseudo-code, passer d'un langage à l'autre est une question de syntaxe, pas de réflexion.

---

## Partie 2 — Architecture d'un système informatique

### Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────┐
│                    SYSTÈME INFORMATIQUE                         │
│                                                                 │
│  ┌──────────────────┐    ┌─────────────────────────────────┐   │
│  │       CPU        │    │           MÉMOIRE               │   │
│  │  ┌─────────────┐ │    │  ┌──────┐  ┌──────┐  ┌──────┐  │   │
│  │  │     ALU     │ │◄──►│  │Cache │  │ RAM  │  │ ROM  │  │   │
│  │  │ (calculs)   │ │    │  │L1/L2 │  │(temp)│  │(perm)│  │   │
│  │  ├─────────────┤ │    │  └──────┘  └──────┘  └──────┘  │   │
│  │  │     CU      │ │    └─────────────────────────────────┘   │
│  │  │ (contrôle)  │ │                    │                     │
│  │  ├─────────────┤ │         ┌──────────┴──────────┐          │
│  │  │  Registres  │ │         │   BUS SYSTÈME        │          │
│  │  └─────────────┘ │         │ (données/adresses/   │          │
│  └──────────────────┘         │  contrôle)           │          │
│           │                   └──────────┬──────────┘          │
│           └───────────────────────────────┤                    │
│                                           │                    │
│  ┌────────────────┐  ┌────────────────────┤──────────────────┐ │
│  │   STOCKAGE     │  │   PÉRIPHÉRIQUES    │                  │ │
│  │  HDD/SSD/NVMe  │  │  Clavier, souris,  │  Écran, imprim.  │ │
│  └────────────────┘  └────────────────────┴──────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

### Le CPU — Cerveau du système

#### Structure interne

**ALU (Arithmetic Logic Unit)** : effectue les opérations mathématiques (addition, soustraction, multiplication, division) et logiques (AND, OR, NOT, XOR, comparaisons). C'est ici que se passe le "calcul" à proprement parler.

**CU (Control Unit)** : orchestre l'exécution des instructions. Il lit chaque instruction depuis la mémoire, la décode, coordonne l'ALU et la mémoire pour l'exécuter, puis passe à l'instruction suivante.

**Registres** : mémoire ultra-rapide intégrée directement dans le CPU. Quelques dizaines à quelques centaines de registres, chacun stockant une valeur (32 ou 64 bits). Les calculs se font toujours dans les registres — une donnée en RAM doit d'abord être chargée dans un registre.

| Type de registre | Rôle |
|------------------|------|
| Accumulateur (ACC) | Résultat des opérations ALU |
| Program Counter (PC) | Adresse de la prochaine instruction |
| Stack Pointer (SP) | Sommet de la pile d'exécution |
| Status Register (SR) | Flags : zero, carry, overflow, négatif |
| Registres généraux | Données temporaires (RAX, RBX... sur x86) |

**Cache CPU** : mémoire très rapide intégrée dans le CPU, plus lente que les registres mais bien plus rapide que la RAM.

```
Hiérarchie mémoire par vitesse d'accès :

Registres    │ ~0.3 ns  │ quelques KB  │ Dans le CPU
Cache L1     │ ~1 ns    │ 32-64 KB     │ Dans le CPU (par cœur)
Cache L2     │ ~4 ns    │ 256KB-1MB    │ Dans le CPU (par cœur)
Cache L3     │ ~10 ns   │ 4-64 MB      │ Dans le CPU (partagé)
RAM (DDR5)   │ ~60 ns   │ 8-128 GB     │ Sur la carte mère
SSD NVMe     │ ~100 µs  │ 500GB-8TB    │ Disque
HDD          │ ~5 ms    │ 1-20 TB      │ Disque
```

> [!warning] L'impact du cache sur les performances
> Accéder à la RAM est 200x plus lent qu'accéder au cache L1. Un programme qui génère beaucoup de "cache misses" (données non trouvées en cache) sera catastrophiquement lent. C'est pourquoi l'accès séquentiel à la mémoire est toujours préférable à l'accès aléatoire (voir [[02 - Green IT et Optimisation]]).

**Pipeline d'exécution** : Les CPUs modernes n'exécutent pas les instructions une par une séquentiellement. Le pipeline divise l'exécution en étapes (Fetch → Decode → Execute → Writeback) et traite plusieurs instructions simultanément à différentes étapes.

**Cores et threads** :
- Un **cœur** (core) est une unité d'exécution indépendante dans le CPU
- Un CPU 8 cœurs peut exécuter 8 instructions véritablement en parallèle
- **HyperThreading** (Intel) / **SMT** (AMD) : chaque cœur physique présente 2 cœurs logiques au système d'exploitation, partageant les ressources d'exécution

---

### La mémoire

#### RAM — Random Access Memory

La RAM est la mémoire de travail de l'ordinateur. Volatile (s'efface à l'extinction), rapide, relativement chère par GB.

**Adressage** : Chaque octet en RAM a une adresse unique (un entier). Un système 64 bits peut théoriquement adresser 2^64 octets = 16 exaoctets.

```
Adresses mémoire (exemple simplifié 8 bits) :
┌────────┬──────────┐
│ Adresse│  Valeur  │
├────────┼──────────┤
│ 0x0000 │ 01001000 │  ← 'H'
│ 0x0001 │ 01100101 │  ← 'e'
│ 0x0002 │ 01101100 │  ← 'l'
│ 0x0003 │ 01101100 │  ← 'l'
│ 0x0004 │ 01101111 │  ← 'o'
│ ...    │ ...      │
└────────┴──────────┘
```

**Localité spatiale** : Les programmes accèdent souvent à des données proches les unes des autres en mémoire. Le CPU charge des blocs (cache lines, ~64 octets) en anticipation.

**Localité temporelle** : Les données récemment accédées ont une forte probabilité d'être accédées à nouveau bientôt. Le cache les conserve.

#### Pile (Stack) vs Tas (Heap)

```
Disposition mémoire d'un programme en C :
┌──────────────────────┐  ← adresses hautes
│       STACK          │  Variables locales, paramètres de fonctions
│   (empile vers bas)  │  Gérée automatiquement (LIFO)
│          ↓           │
│                      │
│          ↑           │
│    (grandit vers haut)│
│       HEAP           │  malloc(), new, calloc()
│  (allocation manuelle)│  Gérée par le programmeur (ou GC)
├──────────────────────┤
│   DATA (initialisées)│  Variables globales et statiques initialisées
├──────────────────────┤
│   BSS (non init.)    │  Variables globales et statiques non initialisées
├──────────────────────┤
│       TEXT           │  Code machine du programme (lecture seule)
└──────────────────────┘  ← adresses basses
```

```c
int global_var = 42;      /* DATA segment */
int uninit_global;        /* BSS segment */

void fonction(int param)  /* param est sur la STACK */
{
    int local = 10;       /* local est sur la STACK */
    int *ptr = malloc(sizeof(int) * 100);  /* allocation sur le HEAP */
    /* ... */
    free(ptr);            /* libération manuelle obligatoire en C */
}
```

> [!warning] Stack overflow
> La pile a une taille limitée (typiquement 1-8 MB). Une récursion infinie (une fonction qui s'appelle indéfiniment) remplit la pile et provoque un "stack overflow" — le programme crashe.

#### ROM — Read-Only Memory

Mémoire non volatile (persiste sans alimentation). Contient le firmware/BIOS/UEFI — le premier code exécuté au démarrage de la machine, avant que le système d'exploitation ne soit chargé.

---

### Stockage

#### HDD vs SSD

| Caractéristique | HDD (Disque dur) | SSD SATA | SSD NVMe |
|-----------------|------------------|----------|----------|
| Vitesse lecture | ~150 MB/s | ~550 MB/s | ~7000 MB/s |
| Latence | ~5-10 ms | ~0.1 ms | ~0.02 ms |
| IOPS | ~100-200 | ~100 000 | ~1 000 000 |
| Prix/TB | Faible | Moyen | Plus élevé |
| Durabilité chocs | Faible (pièces mobiles) | Haute | Haute |
| Technologie | Plateaux magnétiques + tête de lecture | Flash NAND via SATA | Flash NAND via PCIe |

#### Systèmes de fichiers

Le **système de fichiers** organise et indexe les données sur le support de stockage.

| Système de fichiers | OS principal | Caractéristiques |
|--------------------|--------------|------------------|
| ext4 | Linux | Journal, inodes, robuste |
| NTFS | Windows | Journalisation, permissions ACL, compression |
| APFS | macOS | Snapshots, chiffrement natif, SSD-optimisé |
| FAT32 | Universel (clés USB) | Simple, compatible partout, limite 4GB/fichier |
| exFAT | Universel (clés USB) | Comme FAT32 mais sans limite de taille de fichier |
| ZFS | Serveurs | Checksums, RAID intégré, snapshots |
| Btrfs | Linux moderne | Snapshots, compression, sous-volumes |

---

### Bus et communication

Les **bus** sont des canaux de communication qui transfèrent des données entre les composants.

```
Trois types de bus :

Bus de DONNÉES     : transfère les données elles-mêmes (8/16/32/64 bits en parallèle)
Bus d'ADRESSES     : spécifie où lire/écrire en mémoire
Bus de CONTRÔLE    : signaux de synchronisation (lecture/écriture, interruptions, horloge)
```

**Interfaces modernes** :
- **PCIe** (PCI Express) : GPU, SSD NVMe, cartes réseau haute performance. PCIe 5.0 x16 = ~128 GB/s
- **USB** : périphériques externes. USB 3.2 = 20 Gbit/s
- **SATA** : disques durs et SSD SATA. Limité à ~600 MB/s
- **DDR5** : bus mémoire RAM. ~51 GB/s par canal

---

### Représentation binaire

#### Rappel binaire et pourquoi

Les ordinateurs utilisent le binaire (base 2) car les transistors ont deux états : **passant** (1) ou **bloqué** (0). Physiquement simple et robuste face au bruit électrique.

```
Conversions :
Décimal → Binaire
  13 = 8 + 4 + 1 = 1101₂

Binaire → Décimal
  1011₂ = 1×8 + 0×4 + 1×2 + 1×1 = 11

Hexadécimal (base 16) : notation compacte du binaire
  0xFF = 1111 1111₂ = 255
  0x41 = 0100 0001₂ = 65 = 'A' en ASCII
```

#### Entiers signés — Complément à 2

Pour représenter des nombres négatifs, les CPUs utilisent le **complément à 2** :

```
Sur 8 bits :
  +5  =  0000 0101
  -5  =  1111 1011   (complément à 2)

Comment calculer le complément à 2 de N :
  1. Écrire N en binaire :         0000 0101  (+5)
  2. Inverser tous les bits :      1111 1010  (complément à 1)
  3. Ajouter 1 :                   1111 1011  (complément à 2 = -5)

Vérification : +5 + (-5) = 0
  0000 0101
+ 1111 1011
─────────────
  0000 0000   (la retenue finale est ignorée) ✓

Plage sur 8 bits : -128 à +127
Plage sur 16 bits : -32768 à +32767
Plage sur 32 bits : -2 147 483 648 à +2 147 483 647
```

> [!warning] Integer overflow
> ```c
> int8_t x = 127;   /* maximum pour int8_t */
> x = x + 1;        /* résultat : -128 ! overflow silencieux */
> ```
> Ce comportement est une source fréquente de bugs de sécurité.

#### IEEE 754 — Flottants

Les nombres décimaux sont représentés selon la norme IEEE 754 :

```
Format float 32 bits :
┌───┬────────────────┬─────────────────────────────────────────┐
│ S │   Exposant (8) │           Mantisse (23)                 │
└───┴────────────────┴─────────────────────────────────────────┘
  1 bit          8 bits                    23 bits

Valeur = (-1)^S × 1.Mantisse × 2^(Exposant - 127)

Exemple : 3.14 ≈ 0 10000000 10010001111010111000011
```

> [!warning] Erreurs de précision flottante
> ```c
> printf("%.20f\n", 0.1 + 0.2);
> /* Affiche : 0.30000000000000004441 */
> /* PAS 0.3 ! */
> ```
> 0.1 et 0.2 ne sont pas représentables exactement en binaire. Jamais comparer des flottants avec `==`. Utiliser `fabs(a - b) < epsilon`.

#### ASCII et UTF-8

**ASCII** (7 bits) : 128 caractères. Lettres latines, chiffres, ponctuation, caractères de contrôle.

```
'A' = 65 = 0x41 = 0100 0001
'a' = 97 = 0x61 = 0110 0001
'0' = 48 = 0x30 = 0011 0000
```

**UTF-8** : encodage variable (1-4 octets) couvrant tous les caractères de tous les systèmes d'écriture (Unicode). Rétrocompatible avec ASCII pour les 128 premiers caractères.

```
'A'  (U+0041) → 1 octet  : 0100 0001
'é'  (U+00E9) → 2 octets : 11000011 10101001
'中' (U+4E2D) → 3 octets : 11100100 10111000 10101101
'🎵' (U+1F3B5)→ 4 octets : 11110000 10011111 10001110 10110101
```

---

### Système d'exploitation

#### Rôle du SE

Le système d'exploitation est le chef d'orchestre entre le matériel et les applications.

```
Applications utilisateur
         │
         ▼
┌────────────────────────┐
│   SYSTÈME D'EXPLOITATION│
│  ┌──────────┐           │
│  │  Kernel  │           │
│  │          │           │
│  │ Gestion  │ Gestion   │
│  │processus │ mémoire   │
│  │          │           │
│  │ Drivers  │ Système   │
│  │ matériel │ fichiers  │
│  └──────────┘           │
└────────────────────────┘
         │
         ▼
      Matériel (CPU, RAM, Disque, Réseau...)
```

**Fonctions principales du SE** :
1. **Gestion des processus** : création, ordonnancement, terminaison
2. **Gestion de la mémoire** : allocation, protection, mémoire virtuelle
3. **Gestion des fichiers** : organisation, accès, permissions
4. **Gestion des I/O** : pilotes pour les périphériques
5. **Sécurité** : authentification, isolation des processus

#### Processus vs Threads

| | Processus | Thread |
|--|-----------|--------|
| Définition | Programme en cours d'exécution | Unité d'exécution dans un processus |
| Mémoire | Espace mémoire isolé | Partage la mémoire du processus |
| Communication | IPC (pipes, sockets, shared mem) | Variables partagées (attention aux races) |
| Création | Lent (fork()) | Rapide (pthread_create()) |
| Isolation | Fort | Faible (un thread peut corrompre les autres) |
| Exemple | Deux instances de Firefox | Les onglets d'un même Firefox |

```c
#include <pthread.h>
#include <stdio.h>

void *thread_function(void *arg) {
    int thread_id = *(int *)arg;
    printf("Thread %d s'exécute\n", thread_id);
    return NULL;
}

int main(void) {
    pthread_t threads[4];
    int ids[4] = {1, 2, 3, 4};

    for (int i = 0; i < 4; i++) {
        pthread_create(&threads[i], NULL, thread_function, &ids[i]);
    }
    for (int i = 0; i < 4; i++) {
        pthread_join(threads[i], NULL);
    }
    return 0;
}
```

#### Ordonnanceur (Scheduler)

L'ordonnanceur décide quel processus/thread s'exécute sur quel cœur CPU à quel moment. Sur un système avec 8 cœurs mais 200 processus actifs, il doit distribuer équitablement le temps CPU.

**Politiques d'ordonnancement courantes** :
- **Round Robin** : chaque processus reçoit un quantum de temps, puis passe au suivant
- **Priority Scheduling** : les processus de haute priorité passent en premier
- **CFS** (Linux) : Completely Fair Scheduler — partage "équitable" pondéré par priorité

#### Appels système (Syscalls)

Les appels système sont l'interface entre le code utilisateur et le kernel.

```c
/* Ces fonctions C masquent des appels système */
FILE *f = fopen("data.txt", "r");  /* syscall open() */
fread(buffer, 1, 100, f);          /* syscall read()  */
fclose(f);                          /* syscall close() */
printf("Bonjour\n");               /* syscall write() */
```

**Mode User vs Mode Kernel** :

```
Mode USER :
  - Votre programme C, Python, navigateur web
  - Accès mémoire limité à l'espace utilisateur
  - Pas d'accès direct au matériel
  - Ne peut pas crasher le système entier

Mode KERNEL :
  - Code du système d'exploitation
  - Accès complet à tout le matériel
  - Gère les interruptions hardware
  - Un bug ici = kernel panic / blue screen

Transition user → kernel : via syscall (interruption logicielle)
  coût : ~200-500 cycles CPU (relativement cher)
```

---

### Architecture Von Neumann — Fetch-Decode-Execute

John von Neumann a défini en 1945 l'architecture qui reste la base de pratiquement tous les ordinateurs modernes : le programme et les données sont stockés dans **la même mémoire**.

```
Cycle Fetch-Decode-Execute :

┌─────────────────────────────────────────────────────────┐
│                                                         │
│   1. FETCH                                              │
│   Le CU lit l'instruction pointée par le PC            │
│   PC ← PC + 1  (instruction suivante par défaut)       │
│                    │                                    │
│                    ▼                                    │
│   2. DECODE                                             │
│   Le CU décode l'instruction :                          │
│   - Quelle opération ? (ADD, LOAD, JUMP, COMPARE...)   │
│   - Quels opérandes ? (registres, valeurs immédiates)  │
│                    │                                    │
│                    ▼                                    │
│   3. EXECUTE                                            │
│   L'ALU effectue l'opération                           │
│   Résultat stocké dans un registre ou en mémoire       │
│                    │                                    │
│                    └────────────────► retour à FETCH   │
└─────────────────────────────────────────────────────────┘
```

**Exemple concret** : instruction `ADD R1, R2, R3` (R1 = R2 + R3)

1. **Fetch** : lire l'instruction à l'adresse PC
2. **Decode** : "c'est une addition, opérandes = R2 et R3, destination = R1"
3. **Execute** : ALU additionne les valeurs de R2 et R3, stocke le résultat dans R1

---

### RISC vs CISC — Philosophies d'architecture CPU

| | RISC | CISC |
|--|------|------|
| Signification | Reduced Instruction Set Computer | Complex Instruction Set Computer |
| Instructions | Simples, de taille fixe | Complexes, taille variable |
| Exemples | ARM (smartphones), RISC-V, MIPS | x86/x64 (Intel, AMD), x87 |
| Exécution | 1 cycle par instruction (idéal) | 1 à 100+ cycles par instruction |
| Mémoire code | Plus de code pour la même tâche | Moins d'instructions (plus denses) |
| Tendance | Mobile, serveurs (ARM), IoT | PCs, laptops, serveurs x86 |

> [!info] La convergence moderne
> Les processeurs x86 modernes (Intel, AMD) décodent en interne les instructions CISC en micro-opérations RISC. La distinction hardware s'est donc en partie brouillée, mais elle reste pertinente côté compilateur et ISA (Instruction Set Architecture).

---

## Exercices pratiques

### Exercice 1 — Décomposition (30 min)

Choisissez un des problèmes suivants et décomposez-le en sous-problèmes jusqu'aux "feuilles" (tâches de moins d'une heure) :

a) Construire un moteur de blog en ligne
b) Créer un système de réservation de billets de train
c) Implémenter une calculatrice scientifique

Livrable : un arbre de décomposition avec au moins 3 niveaux.

### Exercice 2 — Pseudo-code (45 min)

Écrivez le pseudo-code des algorithmes suivants, puis implémentez-les en Python :

a) **FizzBuzz** : pour les nombres de 1 à 100, afficher "Fizz" si divisible par 3, "Buzz" si par 5, "FizzBuzz" si par les deux, sinon le nombre.

b) **Palindrome** : vérifier si une chaîne de caractères est un palindrome (ex: "radar", "madam").

c) **Fibonacci** : calculer le n-ième terme de la suite de Fibonacci.

### Exercice 3 — Représentation binaire (20 min)

Sans calculatrice :

1. Convertir 156 en binaire
2. Convertir 1011 0110₂ en décimal
3. Convertir 0xA3 en décimal et en binaire
4. Représenter -25 en complément à 2 sur 8 bits
5. Que vaut 0111 1111₂ + 0000 0001₂ sur 8 bits signé ? (Attention à l'overflow)

### Exercice 4 — Architecture mémoire (30 min)

Analysez ce programme C et identifiez pour chaque variable dans quel segment de mémoire elle se trouve (TEXT, DATA, BSS, HEAP, STACK) :

```c
int compteur_global = 0;        /* ? */
char *message;                   /* ? */
const char *NOM = "Holberton";  /* ? */

void incrementer(int valeur) {  /* ? */
    int temp = valeur + 1;       /* ? */
    compteur_global = temp;
}

int main(void) {
    int n = 10;                  /* ? */
    int *tableau = malloc(n * sizeof(int)); /* tableau lui-même : ? / contenu : ? */
    message = malloc(50);        /* ? */
    incrementer(5);
    free(tableau);
    free(message);
    return 0;
}
```

### Exercice 5 — Scratch vers C (60 min)

Implémentez en C le jeu "Pierre Feuille Ciseaux" :
1. D'abord, dessinez la logique en blocs Scratch (sur papier ou sur scratch.mit.edu)
2. Traduisez en pseudo-code
3. Implémentez en C avec `rand()` pour le choix de l'ordinateur
4. Jouez 5 manches et affichez le score final

---

> [!info] Pour aller plus loin
> - [[01 - Introduction au C et Compilation]] — première implémentation concrète en C
> - [[02 - Bases Numeriques]] — approfondissement des systèmes numériques et de l'arithmétique binaire
> - Livre recommandé : *Computer Organization and Architecture* — William Stallings
> - En ligne : *CS50x* de Harvard (gratuit sur edX) — couvre ces fondamentaux avec des exercices guidés
