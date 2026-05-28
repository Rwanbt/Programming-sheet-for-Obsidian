# 02 - Green IT et Optimisation

> [!info] Écrire du code responsable
> Le numérique n'est pas "immatériel". Chaque requête HTTP, chaque boucle inefficace, chaque allocation mémoire inutile consomme de l'énergie réelle, produite en partie par des centrales électriques. Ce cours vous donne les outils pour mesurer, comprendre et réduire l'impact environnemental de votre code — sans sacrifier la fonctionnalité.

---

## Partie 1 — Impact environnemental du numérique

### Les chiffres clés (2024-2025)

Le secteur numérique est souvent présenté comme "dématérialisé". Les chiffres racontent une autre histoire :

| Indicateur | Valeur | Comparaison |
|------------|--------|-------------|
| Part du numérique dans les émissions mondiales de CO₂ | **3,5%** (2024) | Comparable à l'aviation civile (2,5%) |
| Consommation électrique mondiale des datacenters | **~460 TWh/an** (2023) | = consommation annuelle de la France entière |
| Eau consommée par les datacenters US pour refroidissement | **~5 milliards de litres/jour** | = 5 millions de piscines olympiques/an |
| Consommation d'eau de ChatGPT pour une conversation de 25 messages | **~500 ml** | Une bouteille d'eau |
| Durée de vie moyenne d'un smartphone | **2,5 ans** (usage) vs **15+ ans** (ressources physiques) | 80% de l'impact carbone d'un smartphone vient de sa fabrication |
| Empreinte carbone d'entraîner GPT-3 | **~552 tonnes de CO₂** | = 123 voitures pendant un an |
| Email avec pièce jointe 1MB | **~50g de CO₂** | × des milliards d'emails/jour |

**Tendance inquiétante** : L'essor de l'IA générative multiplie par 4-10 la consommation énergétique d'une requête web standard. Les datacenters IA nécessitent des refroidissements intensifs en eau dans des régions parfois déjà en stress hydrique.

**Répartition de l'impact du secteur numérique :**
```
Terminaux utilisateurs (PC, smartphones, TV) : ~47%
Réseaux de communication                      : ~28%
Datacenters et cloud                           : ~25%
```

> [!warning] La fabrication domine
> Pour un smartphone, **80% de son empreinte carbone totale sur l'ensemble de son cycle de vie** est générée lors de sa fabrication (extraction des minerais rares, assemblage, transport), et seulement 20% lors de son usage. Garder ses équipements plus longtemps est l'action individuelle la plus efficace.

---

### Les ressources critiques du numérique

**Minéraux rares** essentiels aux équipements informatiques :
- **Lithium** (batteries) : 70% de la production mondiale en Australie, Chili, Argentine
- **Cobalt** (batteries) : 70% en République Démocratique du Congo
- **Néodyme** (aimants dans disques durs, haut-parleurs) : 60% en Chine
- **Indium** (écrans LCD) : disponibilité décroissante, substituts limités
- **Tantale** (condensateurs) : conflits armés pour son contrôle en Afrique centrale

L'extraction génère des dommages environnementaux et sociaux majeurs, souvent invisibles pour l'utilisateur final.

---

## Partie 2 — Green IT : définition et niveaux d'action

### Définition

Le **Green IT** (ou "Numérique Responsable") regroupe l'ensemble des pratiques visant à réduire l'impact environnemental des systèmes informatiques **tout au long de leur cycle de vie** : fabrication, usage, fin de vie.

**Trois niveaux d'action :**

```
┌────────────────────────────────────────────────────────────────┐
│  Niveau 1 — MATÉRIEL                                           │
│  Choisir des équipements écoefficaces, les conserver longtemps │
│  Labels EPEAT, Energy Star, réparabilité, reconditionnement    │
│  Impact : fort, mais hors du contrôle direct du développeur    │
├────────────────────────────────────────────────────────────────┤
│  Niveau 2 — LOGICIEL (votre zone d'action principale)          │
│  Écrire du code économe en CPU/RAM/réseau/disque               │
│  Algorithmes efficaces, caches, compression, lazy loading      │
│  Impact : direct, immédiat, sous votre contrôle               │
├────────────────────────────────────────────────────────────────┤
│  Niveau 3 — USAGE ET SERVICES                                  │
│  Hébergement vert, sobriété fonctionnelle                      │
│  Supprimer les features inutilisées, réduire les données       │
│  Impact : organisationnel, nécessite des décisions produit     │
└────────────────────────────────────────────────────────────────┘
```

**Quatre Rs du numérique responsable :**
- **Réduire** : moins de fonctionnalités inutiles, moins de données collectées
- **Réutiliser** : matériel reconditionné, composants logiciels partagés
- **Recycler** : filières DEEE (Déchets d'Équipements Électriques et Électroniques)
- **Réparer** : droit à la réparation, logiciels supportant de vieux matériels

---

## Partie 3 — Green C : Code C économe en énergie

### Pourquoi le C est particulièrement impacté

Le C donne un accès direct aux ressources machine. Un mauvais code C peut :
- Allouer 100 MB de mémoire pour stocker 1 KB de données utiles
- Effectuer 1 000 accès mémoire aléatoires là où 10 accès séquentiels suffiraient
- Appeler `malloc/free` dans une boucle à 1 000 Hz (hot path)

**Et un bon code C peut :**
- Tourner 50x plus vite que le même algorithme en Python
- Utiliser 10x moins de RAM que le même traitement en Java (pas de JVM, pas de GC overhead)

### Préférer la pile au tas : éviter les allocations inutiles

```c
/* ❌ Allocation dynamique inutile — overhead malloc + fragmentation */
void traiter_nom(const char *input)
{
    char *buffer = malloc(256);   /* appel syscall, fragmentation heap */
    if (buffer == NULL) return;
    strncpy(buffer, input, 255);
    buffer[255] = '\0';
    /* traitement... */
    free(buffer);                 /* ne pas oublier ! */
}

/* ✓ Allocation sur la pile — zéro overhead, zéro risque de leak */
void traiter_nom(const char *input)
{
    char buffer[256];             /* allocation sur stack : une soustraction de registre */
    strncpy(buffer, input, 255);
    buffer[255] = '\0';
    /* traitement... */
    /* libération automatique à la fin de la fonction */
}
```

**Règle** : utilisez la pile pour les données dont la taille maximale est connue à la compilation et qui restent dans leur scope. Le tas est pour les données dont la taille est dynamique ou dont la durée de vie dépasse le scope.

### Accès mémoire séquentiels : cache-friendly code

Le principe de **localité spatiale** : le CPU charge des blocs de 64 octets (cache lines) à chaque accès mémoire. Accéder à des données contiguës = chaque cache line chargée est pleinement exploitée. Accéder à des données dispersées = beaucoup de cache lines chargées pour peu de données utilisées.

```c
/* Tableau 2D en C : stockage ROW-MAJOR (ligne par ligne) */
int matrix[1000][1000];

/* ❌ Parcours COLONNE par COLONNE : cache-hostile */
/* À chaque itération interne, on saute 1000 ints (4000 octets) en mémoire */
for (int col = 0; col < 1000; col++) {
    for (int row = 0; row < 1000; row++) {
        matrix[row][col] += 1;  /* saut de 4000 octets entre chaque accès */
    }
}

/* ✓ Parcours LIGNE par LIGNE : cache-friendly */
/* Les éléments consécutifs sont en mémoire contiguë */
for (int row = 0; row < 1000; row++) {
    for (int col = 0; col < 1000; col++) {
        matrix[row][col] += 1;  /* accès séquentiel */
    }
}
```

**Impact mesuré** : sur un matrix de 4096×4096, le parcours cache-friendly est **4 à 10 fois plus rapide** que le parcours column-major. La différence vient entièrement des cache misses.

**Structures de données cache-friendly :**

```c
/* ❌ Structure of Arrays vs Array of Structures */
/* AoS (Array of Structures) : pour chaque particule, ses 3 champs sont proches */
/* MAIS si on ne traite qu'une coordonnée à la fois, on charge des données inutiles */
typedef struct {
    float x, y, z;      /* 12 octets */
    float vx, vy, vz;   /* 12 octets */
    float mass;          /* 4 octets  */
    /* total : 28 octets + padding = 32 octets */
} Particle;

Particle particles[1000000];

/* Traiter seulement les positions x : charge 32 octets par accès, utilise 4 */
for (int i = 0; i < 1000000; i++) {
    particles[i].x += particles[i].vx * dt;
}

/* ✓ SoA (Structure of Arrays) : toutes les x sont contiguës */
struct {
    float *x, *y, *z;
    float *vx, *vy, *vz;
    float *mass;
} particles;

/* Traiter seulement les positions x : accès 100% utiles */
for (int i = 0; i < 1000000; i++) {
    particles.x[i] += particles.vx[i] * dt;
}
```

### Éviter les branches imprévisibles (branch prediction)

Les CPUs modernes **prédisent** quelle branche d'un `if/else` sera prise et commencent à exécuter les instructions à l'avance. Si la prédiction est mauvaise : **branch misprediction** = ~15 cycles perdus à vider le pipeline.

```c
#include <stdlib.h>

/* ❌ Données aléatoires → le prédicteur de branche échoue ~50% du temps */
void trier_puis_traiter_mauvais(int *data, int n)
{
    /* Sans tri préalable, les données sont aléatoires → branche imprévisible */
    for (int i = 0; i < n; i++) {
        if (data[i] >= 128) {    /* 50% de chance de chaque côté */
            data[i] *= 2;
        }
    }
}

/* ✓ Trier d'abord → le prédicteur voit d'abord N valeurs < 128, puis N valeurs >= 128 */
void trier_puis_traiter_bon(int *data, int n)
{
    /* qsort de stdlib.h */
    qsort(data, n, sizeof(int), compare_ints);

    for (int i = 0; i < n; i++) {
        if (data[i] >= 128) {    /* prévisible : d'abord toujours false, puis toujours true */
            data[i] *= 2;
        }
    }
}

/* ✓ Encore mieux : branchless (pas de branche du tout) */
void traiter_branchless(int *data, int n)
{
    for (int i = 0; i < n; i++) {
        int mask = -(data[i] >= 128);   /* 0x00000000 ou 0xFFFFFFFF */
        data[i] += data[i] & mask;      /* += data[i] si >= 128, += 0 sinon */
    }
}
```

### Mesurer la performance en C

**1. `clock_gettime` — mesure de temps précise**

```c
#include <time.h>
#include <stdio.h>

double mesurer_temps_ms(void (*fonction)(int *, int), int *data, int n)
{
    struct timespec debut, fin;

    clock_gettime(CLOCK_MONOTONIC, &debut);
    fonction(data, n);
    clock_gettime(CLOCK_MONOTONIC, &fin);

    double ms = (fin.tv_sec - debut.tv_sec) * 1000.0
               + (fin.tv_nsec - debut.tv_nsec) / 1e6;
    return ms;
}

int main(void)
{
    int n = 1000000;
    int *data = malloc(n * sizeof(int));
    /* remplir data... */

    double t = mesurer_temps_ms(trier_puis_traiter_mauvais, data, n);
    printf("Version naïve : %.2f ms\n", t);

    t = mesurer_temps_ms(traiter_branchless, data, n);
    printf("Version branchless : %.2f ms\n", t);

    free(data);
    return 0;
}
```

**2. `perf stat` — compteurs hardware Linux**

```bash
# Compiler avec les symboles de debug
gcc -O2 -g -o mon_programme mon_programme.c

# Mesurer les métriques hardware
perf stat ./mon_programme

# Sortie typique :
#  Performance counter stats for './mon_programme':
#    1,234,567  cache-misses         # 12.3% of all cache refs
#   10,045,678  cache-references
#    5,678,901  branch-misses        # 8.5% of all branches
#   66,789,012  branches
#          1.23 seconds time elapsed
```

**3. `valgrind --tool=callgrind` — profiling d'appels**

```bash
# Compiler sans optimisation pour un profiling précis
gcc -O0 -g -o mon_programme mon_programme.c

# Profiler
valgrind --tool=callgrind ./mon_programme

# Visualiser avec KCachegrind (Linux) ou QCachegrind (Windows/Mac)
kcachegrind callgrind.out.<PID>
```

Callgrind simule le CPU et compte les instructions exécutées par chaque fonction. Parfait pour trouver les hot spots.

**4. `gprof` — profiling d'échantillonnage**

```bash
# Compiler avec l'instrumentation gprof
gcc -pg -O2 -o mon_programme mon_programme.c

# Exécuter le programme (génère gmon.out)
./mon_programme

# Analyser
gprof mon_programme gmon.out | head -50

# Sortie :
#  %   cumulative   self              self     total
# time   seconds   seconds    calls  ms/call  ms/call  name
# 67.3       2.45     2.45   500000     0.00     0.00  calculer_distance
# 21.2       3.22     0.77   500000     0.00     0.00  trier_points
#  8.5       3.53     0.31        1   310.00   310.00  main
```

### L'impact énergétique réel des algorithmes

La complexité algorithmique n'est pas qu'une notion théorique — elle a des implications énergétiques directes et mesurables.

```
Exemple : traiter 1 000 000 d'éléments

O(n) — Lecture séquentielle :
  1 000 000 opérations × ~1 ns = ~1 ms
  Énergie ≈ 0.000001 Joule

O(n log n) — Tri efficace (quicksort, mergesort) :
  1 000 000 × 20 = 20 000 000 opérations × ~1 ns = ~20 ms
  Énergie ≈ 0.00002 Joule

O(n²) — Tri naïf (bubblesort, selection sort) :
  1 000 000² = 10¹² opérations
  À 10⁹ opérations/s : ~1000 secondes = 16 minutes
  Énergie ≈ 30 Joules (vs 0.00002 J pour O(n log n))
  Ratio : 1 500 000x plus énergivore !
```

| Complexité | n = 1 000 | n = 1 000 000 | n = 1 000 000 000 |
|------------|-----------|---------------|-------------------|
| O(1) | 1 | 1 | 1 |
| O(log n) | 10 | 20 | 30 |
| O(n) | 1 000 | 1 000 000 | 1 000 000 000 |
| O(n log n) | 10 000 | 20 000 000 | 30 000 000 000 |
| O(n²) | 1 000 000 | 10¹² | 10¹⁸ |

> [!warning] Le choix d'algorithme prime sur toute micro-optimisation
> Passer de O(n²) à O(n log n) vous fait gagner un facteur million pour n=10⁶. Aucune micro-optimisation (branch prediction, SIMD, accès cache) ne peut compenser un mauvais choix algorithmique.

---

## Partie 4 — Green Python

Python est interprété et beaucoup plus lent que C, mais des outils existent pour profiler et optimiser.

### Line Profiler — identifier les lignes lentes

```bash
pip install line_profiler
```

```python
# Décorer la fonction à profiler avec @profile
@profile
def traiter_fichier(nom_fichier):
    with open(nom_fichier) as f:
        lignes = f.readlines()           # charge tout en RAM (potentiellement problématique)

    resultats = []
    for ligne in lignes:
        mot = ligne.strip()
        if len(mot) > 3:
            resultats.append(mot.upper())   # crée une nouvelle chaîne à chaque fois
    return resultats
```

```bash
# Exécution
kernprof -l -v mon_script.py

# Sortie :
# Line #  Hits     Time  Per Hit   % Time  Line Contents
# =====================================================
#      3     1  45213.0  45213.0      2.1  lignes = f.readlines()
#      6  10000   8234.0      0.8      0.4  for ligne in lignes:
#      8  10000  32456.0      3.2      1.5  if len(mot) > 3:
#      9   8500 1845234.0    217.1     85.4  resultats.append(mot.upper())
```

La ligne 9 prend 85% du temps. On peut optimiser.

### Memory Profiler — détecter les fuites mémoire Python

```bash
pip install memory_profiler
```

```python
from memory_profiler import profile

@profile
def analyse_gros_fichier(nom_fichier):
    # ❌ Charge tout le fichier en RAM d'un coup
    with open(nom_fichier) as f:
        contenu = f.read()   # 10 GB en RAM !
    lignes = contenu.split('\n')
    # traitement...
```

```bash
python -m memory_profiler mon_script.py

# Sortie :
# Line #    Mem usage    Increment   Line Contents
# ================================================
#      5   47.3 MiB     47.3 MiB    with open(nom_fichier) as f:
#      6  10287.4 MiB  10240.1 MiB      contenu = f.read()   ← PROBLÈME
#      7  12847.5 MiB   2560.1 MiB      lignes = contenu.split('\n')
```

### Générateurs vs listes — lazy evaluation

```python
# ❌ Version liste : charge TOUS les éléments en mémoire
def carre_liste(n):
    return [x**2 for x in range(n)]   # crée une liste de n éléments en mémoire

result = carre_liste(1_000_000)       # ~8 MB en mémoire (1M entiers × 8 bytes)
for val in result:
    pass

# ✓ Version générateur : calcule UNE valeur à la fois, mémoire constante O(1)
def carre_generateur(n):
    for x in range(n):
        yield x**2                    # yield suspend la fonction, reprend au prochain appel

gen = carre_generateur(1_000_000)    # objet générateur : quelques dizaines d'octets
for val in gen:
    pass
```

```python
# Exemples pratiques de générateurs pour fichiers volumineux

# ❌ Charge tout le fichier de log en RAM
def analyser_erreurs_mauvais(nom_fichier):
    with open(nom_fichier) as f:
        lignes = f.readlines()          # tout en mémoire
    return [l for l in lignes if 'ERROR' in l]

# ✓ Lit ligne par ligne, mémoire constante quelle que soit la taille du fichier
def analyser_erreurs_bon(nom_fichier):
    with open(nom_fichier) as f:
        for ligne in f:                 # itérateur de fichier = lecture lazy
            if 'ERROR' in ligne:
                yield ligne             # retourne sans tout charger

# Compter les erreurs sans mémoriser toutes les lignes
count = sum(1 for _ in analyser_erreurs_bon('app.log'))
```

### NumPy et Numba — vectorisation et compilation JIT

```python
import numpy as np
import numba
import time

# ❌ Python pur : boucle lente
def distance_python(x1, y1, x2, y2):
    n = len(x1)
    distances = []
    for i in range(n):
        d = ((x2[i] - x1[i])**2 + (y2[i] - y1[i])**2) ** 0.5
        distances.append(d)
    return distances

# ✓ NumPy : opérations vectorisées (implémentées en C/Fortran)
def distance_numpy(x1, y1, x2, y2):
    return np.sqrt((x2 - x1)**2 + (y2 - y1)**2)

# ✓ Numba : compilation JIT, proche des performances C
@numba.jit(nopython=True)
def distance_numba(x1, y1, x2, y2):
    n = len(x1)
    distances = np.empty(n)
    for i in range(n):
        distances[i] = ((x2[i] - x1[i])**2 + (y2[i] - y1[i])**2) ** 0.5
    return distances

# Benchmark
n = 1_000_000
x1 = np.random.rand(n)
y1 = np.random.rand(n)
x2 = np.random.rand(n)
y2 = np.random.rand(n)

t = time.perf_counter()
distance_python(x1.tolist(), y1.tolist(), x2.tolist(), y2.tolist())
print(f"Python pur : {time.perf_counter() - t:.2f}s")

t = time.perf_counter()
distance_numpy(x1, y1, x2, y2)
print(f"NumPy : {time.perf_counter() - t:.3f}s")

t = time.perf_counter()
distance_numba(x1, y1, x2, y2)    # 1ère exécution inclut la compilation JIT
print(f"Numba (warmup) : {time.perf_counter() - t:.3f}s")

t = time.perf_counter()
distance_numba(x1, y1, x2, y2)    # 2ème exécution : compilé, rapide
print(f"Numba (2ème run) : {time.perf_counter() - t:.3f}s")

# Résultats typiques :
# Python pur : 0.85s
# NumPy      : 0.008s  (100x plus rapide)
# Numba      : 0.003s  (280x plus rapide après JIT)
```

---

## Partie 5 — Green Web

### Éco-index — mesurer l'impact d'une page web

**EcoIndex** (ecoindex.fr) est un outil open-source qui attribue une note de A à G à une page web en fonction de :
- Le nombre d'éléments DOM (complexité HTML)
- Le poids de la page (ressources téléchargées)
- Le nombre de requêtes HTTP

```
Score EcoIndex → Note
  81-100 : A  (très sobre)
  61-80  : B
  41-60  : C
  21-40  : D
  11-20  : E
   6-10  : F
    0-5  : G  (très énergivore)
```

**Outil CLI :**
```bash
npm install -g ecoindex-cli
ecoindex-cli --url https://monsite.com --output ecoindex-report.json
```

### Optimisation des images

Les images représentent souvent **50 à 80%** du poids d'une page web.

```html
<!-- ❌ Image non optimisée -->
<img src="photo.png" width="300" height="200">
<!-- PNG 24bits, 2.3 MB, une seule résolution pour tous les écrans -->

<!-- ✓ Image optimisée : format moderne + responsive + lazy loading -->
<picture>
  <!-- WebP pour les navigateurs modernes (30-50% plus léger que JPEG) -->
  <source
    type="image/webp"
    srcset="photo-300.webp 300w, photo-600.webp 600w, photo-900.webp 900w"
    sizes="(max-width: 600px) 300px, (max-width: 900px) 600px, 900px">

  <!-- JPEG comme fallback pour les anciens navigateurs -->
  <source
    type="image/jpeg"
    srcset="photo-300.jpg 300w, photo-600.jpg 600w, photo-900.jpg 900w"
    sizes="(max-width: 600px) 300px, (max-width: 900px) 600px, 900px">

  <!-- loading="lazy" : ne charge que quand l'image entre dans le viewport -->
  <img
    src="photo-300.jpg"
    alt="Description précise"
    width="300"
    height="200"
    loading="lazy">
</picture>
```

**Outils de conversion :**
```bash
# Convertir en WebP avec ImageMagick
convert photo.png -quality 80 photo.webp

# Générer plusieurs résolutions avec Sharp (Node.js)
npx sharp-cli input.jpg --width 300 -o output-300.webp
npx sharp-cli input.jpg --width 600 -o output-600.webp
npx sharp-cli input.jpg --width 900 -o output-900.webp

# Optimiser les JPEG avec mozjpeg
cjpeg -quality 80 -outfile photo-opt.jpg photo.jpg

# Optimiser les PNG avec pngquant
pngquant --quality=65-80 photo.png
```

### CSS et JavaScript : minification et tree shaking

```html
<!-- ❌ Développement : fichiers non minifiés -->
<link rel="stylesheet" href="styles.css">         <!-- 250 KB -->
<script src="app.js"></script>                     <!-- 450 KB -->

<!-- ✓ Production : minifiés, compressés, optimisés -->
<link rel="stylesheet" href="styles.min.css">     <!-- 85 KB (-66%) -->
<script src="app.min.js" defer></script>           <!-- 120 KB (-73%) -->
```

**Minification et bundling avec Vite (moderne, rapide) :**

```javascript
// vite.config.js
import { defineConfig } from 'vite'

export default defineConfig({
    build: {
        minify: 'terser',          // compression JS agressive
        cssMinify: true,           // compression CSS
        rollupOptions: {
            output: {
                // Tree shaking automatique : supprime le code mort
                // Si vous n'importez que lodash.merge, seul cette fonction est incluse
            }
        }
    }
})
```

**Tree shaking** — importer seulement ce dont vous avez besoin :

```javascript
// ❌ Importe toute la bibliothèque lodash (~100 KB gzippé)
import _ from 'lodash'
const result = _.merge(obj1, obj2)

// ✓ Tree shaking : seule la fonction merge est incluse (~3 KB)
import merge from 'lodash/merge'
const result = merge(obj1, obj2)

// ✓ Encore mieux avec les bibliothèques ES modules (tree-shaking natif)
import { merge } from 'lodash-es'
const result = merge(obj1, obj2)
```

### HTTP Caching — éviter les téléchargements inutiles

Le caching HTTP permet au navigateur de stocker les ressources localement et de ne pas les re-télécharger à chaque visite.

```
Sans cache :
  Visite 1 : télécharge index.html (5 KB) + styles.css (85 KB) + app.js (120 KB)
  Visite 2 : re-télécharge tout = 210 KB de plus
  Visite 10: 2,1 MB téléchargés en tout

Avec cache :
  Visite 1 : télécharge tout = 210 KB
  Visite 2 : vérifie index.html (304 Not Modified) + CSS/JS en cache local
  Visite 10: 5 KB (seul index.html vérifié, le reste vient du cache)
```

**Headers HTTP essentiels :**

```nginx
# Configuration Nginx — caching optimal

# Fichiers statiques avec hash dans le nom (vite, webpack les génèrent automatiquement)
# Ex: styles.a3b7f2.css → peut être caché très longtemps car le nom change si le contenu change
location ~* \.(js|css|woff2)$ {
    add_header Cache-Control "public, max-age=31536000, immutable";
    # max-age=31536000 = 1 an. "immutable" = le navigateur ne revalide pas.
}

# Images optimisées avec versioning
location ~* \.(webp|avif|jpg|png|svg)$ {
    add_header Cache-Control "public, max-age=2592000";  # 30 jours
}

# HTML : toujours re-valider (contient les hashes vers les assets)
location ~* \.html$ {
    add_header Cache-Control "no-cache";
    add_header ETag "on";  # nginx génère automatiquement l'ETag
}
```

**ETag — validation conditionnelle :**

```
Navigateur →    GET /styles.css
                If-None-Match: "abc123"   (hash du contenu stocké)

Serveur →  Si contenu inchangé : 304 Not Modified  (0 octets transférés ✓)
           Si contenu modifié  : 200 OK + nouveau contenu + nouveau ETag
```

**`Cache-Control` — référence des directives :**

| Directive | Signification |
|-----------|---------------|
| `max-age=3600` | Cache valide 3600 secondes (1 heure) |
| `public` | Stockable par tous les caches intermédiaires (CDN, proxies) |
| `private` | Stockable uniquement par le navigateur utilisateur (ex: page perso) |
| `no-cache` | Doit revalider avec le serveur avant chaque utilisation |
| `no-store` | Ne pas stocker du tout (données sensibles, données en temps réel) |
| `immutable` | Le contenu ne changera jamais → jamais de revalidation |
| `stale-while-revalidate=60` | Sert l'ancienne version pendant 60s en revalidant en arrière-plan |

### Hébergement vert

**PUE (Power Usage Effectiveness)** : ratio de l'énergie totale consommée par un datacenter par rapport à l'énergie utilisée par les serveurs seuls.

```
PUE = Énergie totale datacenter / Énergie serveurs

PUE = 1.0 : idéal théorique (100% pour les serveurs)
PUE = 1.2 : très efficace (Google, Facebook : ~1.1-1.2)
PUE = 1.5 : efficacité moyenne
PUE = 2.0 : datacenter peu efficace (50% perdu en refroidissement, UPS...)
```

**Hébergeurs certifiés verts :**

| Hébergeur | Certification | Note |
|-----------|---------------|------|
| Infomaniak | ISO 14001, 100% renouvelable | Pionnier européen, Suisse |
| OVHcloud | ISO 50001, PUE ~1.2 | Refroidissement adiabatique |
| Scaleway | 100% renouvelable, PUE ~1.3 | Datacenter DC2 Paris (refroidissement eau chaude recyclée) |
| Hetzner | 100% renouvelable | Datacenter en Finlande (refroidissement naturel) |
| Google Cloud | 100% renouvelable 24/7 | Carbon-free energy depuis 2030 (objectif) |
| AWS | 100% renouvelable (objectif 2025) | Progrès variables selon région |

**Choisir la bonne région cloud :**

La même application hébergée dans différentes régions peut avoir des empreintes carbone très différentes :
- **Finlande, Suède, Norvège** : électricité quasi 100% renouvelable (hydro, éolien)
- **France** : ~90% nucléaire (faible CO₂ mais controversé)
- **Pologne, Australie (NSW)** : forte proportion charbon/gaz
- **Texas (US-east)** : mix variable, pointes de gaz

---

## Partie 6 — Performance Budgets et Web Vitals

### Core Web Vitals (2024-2025)

Google utilise ces métriques pour le classement SEO. Mais elles sont aussi des indicateurs d'expérience utilisateur et de green web.

| Métrique | Signification | Seuil "Bon" | Seuil "À améliorer" | "Mauvais" |
|----------|---------------|-------------|---------------------|-----------|
| **LCP** — Largest Contentful Paint | Temps avant que le plus grand élément visible soit rendu | < 2.5s | 2.5-4s | > 4s |
| **INP** — Interaction to Next Paint | Délai de réponse aux interactions utilisateur | < 200ms | 200-500ms | > 500ms |
| **CLS** — Cumulative Layout Shift | Stabilité visuelle (éléments qui sautent) | < 0.1 | 0.1-0.25 | > 0.25 |
| **FCP** — First Contentful Paint | Premier pixel de contenu visible | < 1.8s | 1.8-3s | > 3s |
| **TTFB** — Time to First Byte | Temps serveur pour répondre | < 800ms | 800ms-1.8s | > 1.8s |

**Mesurer les Core Web Vitals :**

```bash
# PageSpeed Insights CLI
npx pagespeed-insights https://monsite.com --strategy mobile

# Lighthouse
npx lighthouse https://monsite.com --output html --output-path rapport.html

# WebPageTest (plus détaillé)
# Interface web : webpagetest.org
# CLI : wpt test https://monsite.com
```

**Performance budget** — définir des limites à ne pas dépasser :

```json
// budget.json pour Lighthouse CI
{
    "budgets": [
        {
            "path": "/*",
            "timings": [
                { "metric": "largest-contentful-paint", "budget": 2500 },
                { "metric": "cumulative-layout-shift",  "budget": 100 },
                { "metric": "total-blocking-time",      "budget": 300 }
            ],
            "resourceSizes": [
                { "resourceType": "script",     "budget": 200 },
                { "resourceType": "image",      "budget": 300 },
                { "resourceType": "stylesheet", "budget": 100 },
                { "resourceType": "total",      "budget": 800 }
            ]
        }
    ]
}
```

---

## Partie 7 — Télémétrie et métriques système

### Télémétrie en C — `time` et `/proc/stat`

```bash
# Mesurer le temps et les ressources d'un programme
time ./mon_programme

# Sortie :
# real    0m1.234s   (temps "mural", incluant les I/O)
# user    0m1.180s   (temps CPU en mode utilisateur)
# sys     0m0.054s   (temps CPU en appels système)

# Plus détaillé avec /usr/bin/time
/usr/bin/time -v ./mon_programme

# Sortie :
# Maximum resident set size (kbytes): 24576   (mémoire max utilisée)
# Major (I/O) page faults: 0
# Minor (reclaiming a frame) page faults: 1243
# Voluntary context switches: 12
# Involuntary context switches: 8
# Elapsed (wall clock) time: 0:01.23
```

**Lire `/proc/stat` pour les métriques CPU Linux :**

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    long user, nice, system, idle, iowait, irq, softirq;
} CpuStats;

CpuStats lire_cpu_stats(void)
{
    CpuStats stats = {0};
    FILE *f = fopen("/proc/stat", "r");
    if (!f) return stats;

    char line[256];
    fgets(line, sizeof(line), f);  /* première ligne : stats CPU total */
    fclose(f);

    /* Format : "cpu  user nice system idle iowait irq softirq ..." */
    sscanf(line, "cpu  %ld %ld %ld %ld %ld %ld %ld",
           &stats.user, &stats.nice, &stats.system, &stats.idle,
           &stats.iowait, &stats.irq, &stats.softirq);
    return stats;
}

double calculer_usage_cpu(CpuStats avant, CpuStats apres)
{
    long total_avant = avant.user + avant.nice + avant.system + avant.idle
                     + avant.iowait + avant.irq + avant.softirq;
    long total_apres = apres.user + apres.nice + apres.system + apres.idle
                     + apres.iowait + apres.irq + apres.softirq;

    long idle_avant = avant.idle + avant.iowait;
    long idle_apres = apres.idle + apres.iowait;

    long delta_total = total_apres - total_avant;
    long delta_idle  = idle_apres - idle_avant;

    if (delta_total == 0) return 0.0;
    return (1.0 - (double)delta_idle / delta_total) * 100.0;
}
```

**Performance counters hardware en C (Linux perf_event) :**

```c
#include <linux/perf_event.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <string.h>
#include <stdio.h>

/* Ouvrir un compteur de cache misses L1 */
int ouvrir_compteur_cache_miss(void)
{
    struct perf_event_attr attr;
    memset(&attr, 0, sizeof(attr));
    attr.type           = PERF_TYPE_HARDWARE;
    attr.config         = PERF_COUNT_HW_CACHE_MISSES;
    attr.disabled       = 1;
    attr.exclude_kernel = 1;
    attr.exclude_hv     = 1;

    return syscall(SYS_perf_event_open, &attr, 0, -1, -1, 0);
}

/* Utilisation */
int fd = ouvrir_compteur_cache_miss();
ioctl(fd, PERF_EVENT_IOC_RESET, 0);
ioctl(fd, PERF_EVENT_IOC_ENABLE, 0);

/* ... votre code à mesurer ... */

ioctl(fd, PERF_EVENT_IOC_DISABLE, 0);
long long count;
read(fd, &count, sizeof(count));
printf("Cache misses L1 : %lld\n", count);
close(fd);
```

---

## Exercices pratiques

### Exercice 1 — Profiling C (45 min)

Compilez et profilez le programme suivant. Identifiez le bottleneck, puis optimisez :

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* Recherche naïve d'une sous-chaîne dans un grand texte */
int chercher_naif(const char *texte, int n, const char *pattern, int m)
{
    int occurrences = 0;
    for (int i = 0; i <= n - m; i++) {
        int match = 1;
        for (int j = 0; j < m; j++) {
            if (texte[i + j] != pattern[j]) {
                match = 0;
                break;
            }
        }
        if (match) occurrences++;
    }
    return occurrences;
}

int main(void)
{
    int n = 10000000;
    char *texte = malloc(n + 1);
    /* remplir avec des données pseudo-aléatoires */
    for (int i = 0; i < n; i++)
        texte[i] = 'a' + (rand() % 26);
    texte[n] = '\0';

    int count = chercher_naif(texte, n, "hello", 5);
    printf("Occurrences : %d\n", count);

    free(texte);
    return 0;
}
```

**Questions** :
1. Mesurez le temps d'exécution avec `time`
2. Compilez avec `-O2` et re-mesurez. Quelle différence ?
3. Profilez avec `gprof` ou `perf stat`. Où se passe le temps ?
4. Implémentez l'algorithme de Knuth-Morris-Pratt (O(n+m)) et comparez

### Exercice 2 — Générateurs Python (30 min)

```python
# Un fichier de log de 100 MB (simulé) contient des entrées de la forme :
# "2024-01-15 10:23:45 ERROR Database connection timeout"
# "2024-01-15 10:23:46 INFO User login successful"

# 1. Générez un fichier de log simulé de 1 million de lignes
import random
import time

niveaux = ['DEBUG', 'INFO', 'WARNING', 'ERROR']
messages = [
    'Database connection timeout', 'User login successful',
    'Cache miss', 'File not found', 'Request processed',
    'Memory threshold exceeded', 'Query executed'
]

with open('simulation.log', 'w') as f:
    for i in range(1_000_000):
        niveau = random.choice(niveaux)
        msg = random.choice(messages)
        f.write(f"2024-01-15 {10 + i//3600:02d}:{(i//60)%60:02d}:{i%60:02d} {niveau} {msg}\n")

# 2. Implémentez deux versions d'extraction des erreurs :
#    - Version liste (tout en mémoire)
#    - Version générateur (lazy)
# Mesurez la différence de mémoire avec memory_profiler
# Mesurez la différence de temps avec time.perf_counter()
```

### Exercice 3 — Optimisation Web (60 min)

Créez une page HTML "non optimisée" avec :
- Une image PNG de 2 MB (créez-en une de test)
- Un fichier CSS de 50 KB non minifié
- Un fichier JavaScript de 100 KB non minifié
- Pas de headers de cache

Puis optimisez-la :
1. Convertissez l'image en WebP avec différentes qualités (80%, 60%, 40%)
2. Ajoutez le lazy loading et le `srcset`
3. Minifiez CSS avec `cssnano` et JS avec `terser`
4. Configurez les headers de cache appropriés (simulez avec un serveur Express)
5. Mesurez avec Lighthouse avant/après

Livrable : rapport de gains (poids avant/après, score Lighthouse avant/après)

### Exercice 4 — Cache-friendly en C (30 min)

```c
#define N 2048

float matrix_a[N][N];
float matrix_b[N][N];
float matrix_c[N][N];

/* Implémenter 3 versions de multiplication matricielle :
   1. Version naïve ijk
   2. Version ikj (transposer la boucle pour accès cache-friendly)
   3. Version avec tiling (blocs de 64x64)

   Mesurer le temps de chacune avec clock_gettime.
   Expliquer pourquoi les performances diffèrent. */
```

### Exercice 5 — Mesure d'empreinte (15 min)

Utilisez l'outil Éco-index (ecoindex.fr) ou l'extension de navigateur pour analyser 3 sites web de votre choix :
1. Un site "lourd" (e-commerce, médias)
2. Un site "sobre" (Wikipedia, documentation)
3. Votre propre projet web (si disponible)

Pour chacun, relevez : score EcoIndex, poids de la page, nombre de requêtes HTTP, nombre d'éléments DOM. Identifiez les 3 principales améliorations possibles pour le site le moins bien noté.

---

> [!info] Pour aller plus loin
> - [[07 - DevSecOps]] — intégrer les contrôles de performance en CI/CD
> - [[08 - Projet Capstone DevOps]] — appliquer le green thinking à un projet réel
> - Livre : *Sustainable Web Design* — Tom Greenwood (A Book Apart)
> - Outil : Website Carbon Calculator (websitecarbon.com)
> - RFC : Green Software Foundation Spec — Software Carbon Intensity (SCI)
