# Collections et Iterateurs

Les collections sont les structures de donnees dynamiques qui permettent de stocker et manipuler des ensembles de valeurs. Si vous venez du C, vous avez l'habitude de `malloc`, `realloc`, de gerer la taille manuellement, et de parcourir avec des boucles `for` indexees. Rust offre des collections generiques, type-safe, avec gestion automatique de la memoire, et un systeme d'iterateurs qui remplace elegamment les boucles manuelles tout en gardant les memes performances.

Ce chapitre couvre les trois collections les plus utilisees (Vec, String, HashMap), les collections secondaires, puis plonge dans le systeme d'iterateurs et les closures qui les alimentent. A chaque etape, nous comparons avec l'equivalent C pour comprendre ce que Rust apporte.

---

## Vec\<T\> : le tableau dynamique

### Creer et manipuler un Vec

```rust
fn main() {
    // Creation vide avec annotation de type
    let mut nombres: Vec<i32> = Vec::new();

    // Creation avec la macro vec!
    let scores = vec![100, 95, 88, 72, 60];

    // Creation avec une valeur repetee
    let zeros = vec![0; 10];  // [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

    // Ajout d'elements
    nombres.push(1);
    nombres.push(2);
    nombres.push(3);
    // nombres = [1, 2, 3]

    // Suppression du dernier element
    let dernier: Option<i32> = nombres.pop();  // Some(3)

    // Insertion a un index
    nombres.insert(0, 0);  // [0, 1, 2]

    // Suppression a un index
    let retire = nombres.remove(0);  // 0, nombres = [1, 2]

    // Taille
    println!("Taille : {}", nombres.len());      // 2
    println!("Est vide : {}", nombres.is_empty()); // false
}
```

### Acces aux elements

```rust
fn main() {
    let v = vec![10, 20, 30, 40, 50];

    // Acces direct avec [] : panique si hors limites
    let troisieme = v[2];  // 30
    // let hors = v[10];   // PANIQUE !

    // Acces securise avec get() : retourne Option<&T>
    match v.get(2) {
        Some(val) => println!("Element 2 : {}", val),  // 30
        None => println!("Index hors limites"),
    }

    // Modification (le Vec doit etre mut)
    let mut v = vec![1, 2, 3];
    v[0] = 100;             // [100, 2, 3]

    if let Some(elem) = v.get_mut(1) {
        *elem = 200;         // [100, 200, 3]
    }
}
```

### Capacity vs Length

```
Vec en memoire :

   Stack                        Heap
+----------+               +---+---+---+---+---+---+---+---+
| ptr   ---|-------------->| 1 | 2 | 3 | ? | ? | ? | ? | ? |
| len: 3   |               +---+---+---+---+---+---+---+---+
| cap: 8   |                 ^           ^                   ^
+----------+                 |           |                   |
                          donnees    len = 3            cap = 8
                          utilisees                  espace alloue

Quand on push() et len == cap :
  1. Rust alloue un nouveau buffer 2x plus grand
  2. Copie les elements existants
  3. Libere l'ancien buffer
  (Exactement comme realloc en C, mais automatique et sur)
```

```rust
fn main() {
    let mut v = Vec::new();
    println!("len: {}, cap: {}", v.len(), v.capacity());  // 0, 0

    v.push(1);
    println!("len: {}, cap: {}", v.len(), v.capacity());  // 1, 4

    for i in 2..=4 {
        v.push(i);
    }
    println!("len: {}, cap: {}", v.len(), v.capacity());  // 4, 4

    v.push(5);  // Reallocation !
    println!("len: {}, cap: {}", v.len(), v.capacity());  // 5, 8

    // Pre-allouer pour eviter les reallocations
    let mut v2 = Vec::with_capacity(1000);
    // len = 0, cap = 1000
    // Les 1000 premiers push() ne reallqueront pas
}
```

### Slices : &[T]

Un slice est une reference vers une portion contigue de memoire. C'est la vue non-owning sur un Vec (ou un tableau fixe).

```rust
fn somme(donnees: &[i32]) -> i32 {
    // Accepte un Vec, un tableau, ou un slice
    donnees.iter().sum()
}

fn main() {
    let v = vec![1, 2, 3, 4, 5];

    // Tout le Vec comme slice
    let total = somme(&v);  // 15

    // Sous-slice
    let milieu = &v[1..4];  // [2, 3, 4]
    let debut = &v[..3];    // [1, 2, 3]
    let fin = &v[2..];      // [3, 4, 5]

    // Tableau fixe comme slice
    let tab = [10, 20, 30];
    let total = somme(&tab);  // 60
}
```

### Comparaison avec les tableaux dynamiques en C

```c
#include <stdlib.h>
#include <string.h>

// En C : gestion manuelle TOTALE
typedef struct {
    int *data;
    size_t len;
    size_t cap;
} VecInt;

VecInt vec_new() {
    VecInt v = { NULL, 0, 0 };
    return v;
}

int vec_push(VecInt *v, int valeur) {
    if (v->len == v->cap) {
        size_t new_cap = v->cap == 0 ? 4 : v->cap * 2;
        int *new_data = realloc(v->data, new_cap * sizeof(int));
        if (new_data == NULL) return -1;  // Erreur d'allocation
        v->data = new_data;
        v->cap = new_cap;
    }
    v->data[v->len++] = valeur;
    return 0;
}

int vec_get(const VecInt *v, size_t index) {
    // Pas de verification de bornes !
    // Si index >= len : comportement indefini
    return v->data[index];
}

void vec_free(VecInt *v) {
    free(v->data);
    v->data = NULL;
    v->len = 0;
    v->cap = 0;
    // Et si on oublie ? Memory leak.
    // Et si on l'appelle deux fois ? Double free.
}
```

```
Comparaison Vec<T> vs tableau dynamique C :

Critere              C (manuel)              Rust (Vec<T>)
-------------------------------------------------------------
Creation             malloc + init           Vec::new() ou vec![]
Ajout                realloc manuel          push()
Acces securise       Non (UB si hors)        get() -> Option
Liberation           free() (oubli = leak)   Automatique (Drop)
Type safety          void* = aucune          Generique <T>
Thread safety        Aucune garantie         Ownership + borrowing
Taille dynamique     len/cap manuels         len()/capacity()
```

> [!tip] Analogie
> Un `Vec` en Rust est comme un classeur extensible avec un employe dedie qui gere les pages : il ajoute des pages au besoin, refuse de vous montrer une page qui n'existe pas, et detruit le classeur quand vous n'en avez plus besoin. En C, c'est vous l'employe, et personne ne vous previent si vous cherchez la page 500 dans un classeur de 3 pages.

---

## String : les chaines de caracteres

### Creation et manipulation

```rust
fn main() {
    // Creation
    let mut s = String::new();                     // Chaine vide
    let s2 = String::from("Bonjour");              // Depuis un literal
    let s3 = "Bonjour".to_string();                // Equivalent
    let s4 = format!("{} {}", "Bonjour", "monde"); // Interpolation

    // Ajout
    s.push_str("Hello");   // Ajouter un &str
    s.push(' ');            // Ajouter un char
    s.push_str("world");
    // s = "Hello world"

    // Concatenation avec +
    let hello = String::from("Hello");
    let world = String::from(" world");
    let salut = hello + &world;  // hello est CONSOMME (move) !
    // println!("{}", hello);    // ERREUR : hello a ete move

    // Concatenation avec format! (pas de move)
    let prenom = String::from("Alice");
    let nom = String::from("Dupont");
    let complet = format!("{} {}", prenom, nom);
    // prenom et nom sont toujours utilisables
}
```

### Le probleme de l'indexation : UTF-8

> [!warning] On ne peut PAS indexer une String avec \[\]
> `"Bonjour"[0]` ne compile pas en Rust. Pourquoi ? Parce qu'une `String` est encodee en UTF-8, et un "caractere" peut occuper 1 a 4 octets. L'indexation par numero d'octet serait dangereuse (on pourrait couper un caractere en deux), et l'indexation par numero de caractere serait O(n) au lieu de O(1).

```rust
fn main() {
    let texte = String::from("cafe\u{0301}");  // "cafe" avec accent : cafe
    // En memoire UTF-8 : [99, 97, 102, 101, 204, 129]
    //                      c   a   f    e    accent combine

    // Trois visions du meme texte :
    println!("Octets : {:?}", texte.as_bytes());
    // [99, 97, 102, 101, 204, 129]  -- 6 octets

    println!("Chars : {:?}", texte.chars().collect::<Vec<_>>());
    // ['c', 'a', 'f', 'e', '\u{0301}']  -- 5 code points

    // Graphemes (clusters visuels) : necessite le crate unicode-segmentation
    // ["c", "a", "f", "e\u{0301}"]  -- 4 graphemes visuels
}
```

```
Les 3 niveaux d'une String UTF-8 :

Texte visuel :     n   o   e   l       (4 graphemes)
                   |   |   |   |
Code points :      n   o   e   ¨   l   (5 scalar values)
                   |   |   |  / \  |
Octets UTF-8 :   6E  6F  65 CC 88 6C   (6 bytes)

String::len()    = 6  (octets)
.chars().count() = 5  (code points)
graphemes        = 4  (ce que l'humain voit)
```

### Methodes utiles sur String

```rust
fn main() {
    let s = String::from("  Bonjour le monde  ");

    // Decoupage
    let mots: Vec<&str> = s.split_whitespace().collect();
    // ["Bonjour", "le", "monde"]

    let parties: Vec<&str> = "a,b,c,d".split(',').collect();
    // ["a", "b", "c", "d"]

    // Recherche
    let contient = s.contains("monde");       // true
    let commence = s.starts_with("  Bon");    // true
    let finit = s.ends_with("monde  ");       // true

    // Transformation
    let sans_espaces = s.trim();              // "Bonjour le monde"
    let majuscules = s.to_uppercase();
    let remplacement = s.replace("monde", "Rust");

    // Iteration sur les caracteres
    for c in "Rust".chars() {
        print!("{} ", c);  // R u s t
    }

    // Slicing (par indices d'octets, attention !)
    let bonjour = &s.trim()[..7];  // "Bonjour" (7 octets = 7 chars ASCII)
    // let invalide = &"cafe\u{0301}"[..5];  // PANIQUE si coupe au milieu d'un char
}
```

### Comparaison avec char* en C

```c
#include <string.h>
#include <stdlib.h>

// En C : chaines = pointeurs bruts
char *concatener(const char *a, const char *b) {
    size_t len_a = strlen(a);      // O(n) a chaque appel !
    size_t len_b = strlen(b);
    char *result = malloc(len_a + len_b + 1);
    if (result == NULL) return NULL;  // Gestion d'erreur manuelle
    strcpy(result, a);
    strcat(result, b);
    return result;
    // L'appelant DOIT free(result). S'il oublie : memory leak.
}

// Problemes classiques :
void dangers() {
    char buf[10];
    strcpy(buf, "Cette chaine est beaucoup trop longue");
    // BUFFER OVERFLOW : ecrase la memoire adjacente

    char *s = "hello";
    s[0] = 'H';  // COMPORTEMENT INDEFINI (literal en memoire read-only)
}
```

```
Comparaison String vs char* :

Critere              C (char*)               Rust (String)
-------------------------------------------------------------
Encodage             Ambigu (ASCII?)         UTF-8 garanti
Taille               strlen() O(n)           len() O(1)
Concatenation        strcat (buffer ?)       push_str / + / format!
Buffer overflow      Possible                Impossible
Memoire              free() manuel           Automatique (Drop)
Null terminator      Obligatoire (\0)        Non necessaire
Slicing              Pointeur arithmetique   &str avec verif UTF-8
```

> [!info] &str vs String
> `String` est un type possede (owner) : il alloue sur le heap et libere a la destruction. `&str` est une reference (emprunt) vers des donnees UTF-8 quelque part en memoire. Preferez `&str` comme parametre de fonction et `String` quand vous avez besoin de posseder la chaine.

---

## HashMap\<K, V\> : la table de hachage

### Operations de base

```rust
use std::collections::HashMap;

fn main() {
    // Creation
    let mut scores: HashMap<String, i32> = HashMap::new();

    // Insertion
    scores.insert(String::from("Alice"), 95);
    scores.insert(String::from("Bob"), 87);
    scores.insert(String::from("Claire"), 92);

    // Acces
    let score_alice = scores.get("Alice");  // Option<&i32> -> Some(&95)
    let score_dave = scores.get("Dave");    // None

    // Acces avec valeur par defaut
    let s = scores.get("Dave").copied().unwrap_or(0);  // 0

    // Suppression
    let ancien = scores.remove("Bob");  // Some(87)

    // Taille
    println!("Nombre d'entrees : {}", scores.len());  // 2

    // Verification de presence
    if scores.contains_key("Alice") {
        println!("Alice a un score");
    }
}
```

### L'API Entry : inserer ou modifier

```rust
use std::collections::HashMap;

fn main() {
    let mut scores: HashMap<String, Vec<i32>> = HashMap::new();

    // or_insert : insere si absent, retourne une reference mutable
    scores.entry(String::from("Alice")).or_insert(vec![]);
    scores.entry(String::from("Alice")).or_insert(vec![]);  // Pas d'effet

    // or_insert_with : calcul paresseux
    scores.entry(String::from("Bob"))
        .or_insert_with(|| {
            println!("Creation du vecteur pour Bob");
            Vec::new()
        });

    // Modifier la valeur existante
    scores.entry(String::from("Alice"))
        .or_insert(vec![])
        .push(95);

    scores.entry(String::from("Alice"))
        .or_insert(vec![])
        .push(88);

    // Alice a maintenant [95, 88]
    println!("{:?}", scores.get("Alice"));  // Some([95, 88])
}
```

> [!example] Compteur de mots avec Entry
> ```rust
> use std::collections::HashMap;
> 
> fn compter_mots(texte: &str) -> HashMap<&str, usize> {
>     let mut compteur = HashMap::new();
>     for mot in texte.split_whitespace() {
>         let count = compteur.entry(mot).or_insert(0);
>         *count += 1;
>     }
>     compteur
> }
> 
> fn main() {
>     let texte = "le chat mange le poisson et le chat dort";
>     let compteur = compter_mots(texte);
>     for (mot, count) in &compteur {
>         println!("{} : {}", mot, count);
>     }
>     // le : 3, chat : 2, mange : 1, poisson : 1, et : 1, dort : 1
> }
> ```

### Iteration sur HashMap

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert("pomme", 3);
    map.insert("banane", 5);
    map.insert("cerise", 12);

    // Iteration par reference
    for (cle, valeur) in &map {
        println!("{} : {}", cle, valeur);
    }

    // Iteration mutable
    for (_, valeur) in &mut map {
        *valeur *= 2;  // Doubler toutes les valeurs
    }

    // Iteration qui consomme (ownership)
    for (cle, valeur) in map {
        println!("{} -> {}", cle, valeur);
    }
    // map n'est plus utilisable ici
}
```

### Comparaison avec les tables de hachage en C

```c
// En C : pas de table de hachage integree
// Il faut soit ecrire la sienne, soit utiliser une bibliotheque

typedef struct Entry {
    char *key;
    int value;
    struct Entry *next;  // Chainage pour collisions
} Entry;

typedef struct {
    Entry **buckets;
    size_t nb_buckets;
    size_t count;
} HashMap;

// Il faut implementer :
// - hash_string() : fonction de hachage
// - hashmap_new() : allocation
// - hashmap_insert() : insertion avec gestion des collisions
// - hashmap_get() : recherche
// - hashmap_remove() : suppression avec re-chainage
// - hashmap_free() : liberation de TOUTE la memoire
// Environ 150-200 lignes de code... pour chaque type de cle/valeur
```

> [!info] En Rust, `HashMap` est generique
> `HashMap<K, V>` fonctionne avec n'importe quel type `K` qui implemente `Eq + Hash`, et n'importe quel type `V`. En C, il faut reecrire la structure (ou utiliser `void*` avec des casts dangereux) pour chaque combinaison de types.

---

## Autres collections utiles

### HashSet\<T\>

```rust
use std::collections::HashSet;

fn main() {
    let mut langages: HashSet<String> = HashSet::new();

    langages.insert(String::from("Rust"));
    langages.insert(String::from("C"));
    langages.insert(String::from("Python"));
    langages.insert(String::from("Rust"));  // Doublon ignore

    println!("Nombre : {}", langages.len());  // 3
    println!("Contient Rust : {}", langages.contains("Rust"));  // true

    // Operations ensemblistes
    let systeme: HashSet<String> = ["C", "Rust", "C++"]
        .iter().map(|s| s.to_string()).collect();

    let script: HashSet<String> = ["Python", "JavaScript", "Rust"]
        .iter().map(|s| s.to_string()).collect();

    // Intersection
    for lang in systeme.intersection(&script) {
        println!("Les deux : {}", lang);  // Rust
    }

    // Union, difference, symetric_difference aussi disponibles
}
```

### BTreeMap et BTreeSet : versions ordonnees

```rust
use std::collections::BTreeMap;

fn main() {
    let mut scores = BTreeMap::new();
    scores.insert("Zebra", 10);
    scores.insert("Alpha", 30);
    scores.insert("Mango", 20);

    // Iteration dans l'ordre des cles (alphabetique ici)
    for (cle, val) in &scores {
        println!("{} : {}", cle, val);
    }
    // Alpha : 30
    // Mango : 20
    // Zebra : 10

    // Range queries
    for (cle, val) in scores.range("Alpha"..="Mango") {
        println!("{} : {}", cle, val);
    }
}
```

```
Comparaison des collections :

Collection       Ordre      Recherche    Insertion     Utilisation
-------------------------------------------------------------------
HashMap<K,V>     Non        O(1) moy.    O(1) moy.    Table rapide
BTreeMap<K,V>    Oui        O(log n)     O(log n)     Cles ordonnees
HashSet<T>       Non        O(1) moy.    O(1) moy.    Ensemble rapide
BTreeSet<T>      Oui        O(log n)     O(log n)     Ensemble ordonne
Vec<T>           Insertion  O(n)         O(1) amort.  Liste sequentielle
VecDeque<T>      Insertion  O(n)         O(1) amort.  File double
LinkedList<T>    Insertion  O(n)         O(1)         Rarement utile
```

---

## Closures

Les closures sont des fonctions anonymes qui capturent leur environnement. Elles sont essentielles pour utiliser les iterateurs efficacement.

### Syntaxe

```rust
fn main() {
    // Fonction normale
    fn ajouter_fn(a: i32, b: i32) -> i32 { a + b }

    // Closure equivalente
    let ajouter = |a: i32, b: i32| -> i32 { a + b };

    // Closure avec inference de types (plus courant)
    let ajouter = |a, b| a + b;
    let resultat: i32 = ajouter(3, 4);  // 7

    // Closure sans arguments
    let saluer = || println!("Bonjour !");
    saluer();

    // Closure multiligne
    let traiter = |x: i32| {
        let double = x * 2;
        let carre = double * double;
        carre  // Pas de ; = valeur de retour
    };
    println!("{}", traiter(3));  // 36
}
```

### Capture de l'environnement

```rust
fn main() {
    let facteur = 10;
    let message = String::from("Resultat");

    // Capture par reference (&facteur) - trait Fn
    let multiplier = |x| x * facteur;
    println!("{}", multiplier(5));  // 50
    println!("facteur toujours valide : {}", facteur);

    // Capture par reference mutable (&mut) - trait FnMut
    let mut compteur = 0;
    let mut incrementer = || {
        compteur += 1;  // Emprunte &mut compteur
        compteur
    };
    println!("{}", incrementer());  // 1
    println!("{}", incrementer());  // 2

    // Capture par move (ownership) - trait FnOnce
    let nom = String::from("Alice");
    let saluer = move || {
        println!("Bonjour, {}", nom);  // nom est MOVE dans la closure
    };
    saluer();
    // println!("{}", nom);  // ERREUR : nom a ete move
}
```

```
Les 3 traits de closure :

FnOnce  :  Peut etre appelee AU MOINS une fois
            (consomme les captures)
   ^
   |
FnMut   :  Peut etre appelee plusieurs fois
            (modifie les captures)
   ^
   |
Fn      :  Peut etre appelee plusieurs fois
            (lit les captures, ne modifie pas)

Hierarchie : Fn implemente FnMut, FnMut implemente FnOnce
Donc une closure Fn peut etre passee la ou FnMut ou FnOnce est attendu.
```

> [!tip] Analogie
> Une closure est comme un employe qui peut emporter des documents du bureau. `Fn` : il les photographie (lecture seule). `FnMut` : il prend le crayon et annote l'original (modification). `FnOnce` : il emporte le document original (consommation). Le mot-cle `move` dit : "emporte tout ce dont tu as besoin".

---

## Iterateurs

### Le trait Iterator

```rust
// Definition simplifiee du trait Iterator
trait Iterator {
    type Item;  // Le type des elements produits

    fn next(&mut self) -> Option<Self::Item>;
    // Retourne Some(element) ou None quand c'est fini
}
```

```
Fonctionnement d'un iterateur :

  iter.next() -> Some(1)
  iter.next() -> Some(2)
  iter.next() -> Some(3)
  iter.next() -> None      <- fin de l'iteration
  iter.next() -> None      <- reste None pour toujours
```

### Les trois methodes d'iteration

```rust
fn main() {
    let v = vec![1, 2, 3];

    // iter() : emprunte &T (le Vec reste utilisable)
    for val in v.iter() {
        // val est de type &i32
        println!("{}", val);
    }
    println!("v est toujours la : {:?}", v);

    // iter_mut() : emprunte &mut T (modification en place)
    let mut v = vec![1, 2, 3];
    for val in v.iter_mut() {
        // val est de type &mut i32
        *val *= 2;
    }
    println!("v modifie : {:?}", v);  // [2, 4, 6]

    // into_iter() : consomme le Vec (ownership des elements)
    let v = vec![String::from("a"), String::from("b")];
    for val in v.into_iter() {
        // val est de type String (ownership transfere)
        println!("possede : {}", val);
    }
    // println!("{:?}", v);  // ERREUR : v a ete consomme
}
```

```
Les 3 types d'iteration :

Methode        Produit    Ownership        Apres la boucle
--------------------------------------------------------------
.iter()        &T         Emprunt          Vec toujours valide
.iter_mut()    &mut T     Emprunt mut      Vec modifie
.into_iter()   T          Consomme         Vec detruit

Note : for val in &v    est equivalent a  for val in v.iter()
       for val in &mut v est equivalent a  for val in v.iter_mut()
       for val in v      est equivalent a  for val in v.into_iter()
```

### Adaptateurs d'iterateurs

Les adaptateurs transforment un iterateur en un autre iterateur. Ils sont **paresseux** : rien ne s'execute tant qu'on ne consomme pas l'iterateur.

```rust
fn main() {
    let nombres = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // map : transformer chaque element
    let doubles: Vec<i32> = nombres.iter().map(|x| x * 2).collect();
    // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

    // filter : garder les elements qui satisfont un predicat
    let pairs: Vec<&i32> = nombres.iter().filter(|x| *x % 2 == 0).collect();
    // [2, 4, 6, 8, 10]

    // enumerate : ajouter l'index
    for (i, val) in nombres.iter().enumerate() {
        println!("[{}] = {}", i, val);
    }

    // zip : combiner deux iterateurs
    let noms = vec!["Alice", "Bob", "Claire"];
    let ages = vec![30, 25, 28];
    let personnes: Vec<_> = noms.iter().zip(ages.iter()).collect();
    // [("Alice", 30), ("Bob", 25), ("Claire", 28)]

    // take / skip
    let trois_premiers: Vec<&i32> = nombres.iter().take(3).collect();  // [1, 2, 3]
    let sans_trois: Vec<&i32> = nombres.iter().skip(3).collect();      // [4..10]

    // chain : enchainer deux iterateurs
    let a = vec![1, 2, 3];
    let b = vec![4, 5, 6];
    let tous: Vec<&i32> = a.iter().chain(b.iter()).collect();  // [1..6]

    // flat_map : map + aplatir
    let phrases = vec!["hello world", "foo bar"];
    let mots: Vec<&str> = phrases.iter()
        .flat_map(|s| s.split_whitespace())
        .collect();
    // ["hello", "world", "foo", "bar"]
}
```

### Consommateurs d'iterateurs

Les consommateurs declenchent l'execution de la chaine d'iterateurs.

```rust
fn main() {
    let nombres = vec![1, 2, 3, 4, 5];

    // collect : rassembler dans une collection
    let v: Vec<i32> = nombres.iter().copied().collect();

    // sum : additionner
    let total: i32 = nombres.iter().sum();  // 15

    // count : compter les elements
    let nb: usize = nombres.iter().count();  // 5

    // min / max
    let minimum = nombres.iter().min();  // Some(&1)
    let maximum = nombres.iter().max();  // Some(&5)

    // fold : accumulation personnalisee
    let produit = nombres.iter().fold(1, |acc, x| acc * x);  // 120

    // reduce : comme fold mais utilise le premier element comme accumulateur
    let somme = nombres.iter().copied().reduce(|acc, x| acc + x);  // Some(15)

    // any / all
    let a_pair = nombres.iter().any(|x| x % 2 == 0);   // true
    let tous_pos = nombres.iter().all(|x| *x > 0);      // true

    // find : premier element correspondant
    let premier_pair = nombres.iter().find(|x| *x % 2 == 0);  // Some(&2)

    // position : index du premier element correspondant
    let pos = nombres.iter().position(|x| *x == 3);  // Some(2)
}
```

### Chainage : la puissance des iterateurs

```rust
fn main() {
    let donnees = vec![
        "Alice:95", "Bob:87", "Claire:92", "Dave:abc", "Eve:78", ""
    ];

    // Pipeline : filtrer, parser, transformer, collecter
    let scores: Vec<(String, u32)> = donnees.iter()
        .filter(|s| !s.is_empty())           // Ignorer les lignes vides
        .filter_map(|s| {                     // Parser et filtrer les erreurs
            let parts: Vec<&str> = s.split(':').collect();
            if parts.len() == 2 {
                parts[1].parse::<u32>().ok().map(|score| {
                    (parts[0].to_string(), score)
                })
            } else {
                None
            }
        })
        .filter(|(_, score)| *score >= 80)    // Garder les scores >= 80
        .collect();

    // scores = [("Alice", 95), ("Bob", 87), ("Claire", 92)]
    println!("{:?}", scores);

    // Calcul de moyenne en une chaine
    let notes = vec![15, 12, 18, 9, 14, 16];
    let (somme, count) = notes.iter()
        .fold((0, 0), |(s, c), &n| (s + n, c + 1));
    let moyenne = somme as f64 / count as f64;
    println!("Moyenne : {:.1}", moyenne);  // 14.0
}
```

### Comparaison : boucles C vs iterateurs Rust

```c
// C : boucle manuelle pour filtrer et transformer
int scores[100];
int scores_len = 0;
for (int i = 0; i < nb_etudiants; i++) {
    if (etudiants[i].note >= 10) {
        scores[scores_len++] = etudiants[i].note * 2;
    }
}
// Problemes : taille fixe, pas de verification de bornes,
// logique melee (filtre + transform + stockage dans la meme boucle)
```

```rust
// Rust : pipeline declaratif
let scores: Vec<i32> = etudiants.iter()
    .filter(|e| e.note >= 10)
    .map(|e| e.note * 2)
    .collect();
// Avantages : redimensionnement auto, expressif, et...
// MEME PERFORMANCE que la boucle C grace aux zero-cost abstractions
```

> [!info] Zero-cost abstractions
> Le compilateur Rust transforme les chaines d'iterateurs en code machine equivalent a une boucle `for` manuelle. Il n'y a AUCUN overhead a l'execution. Les iterateurs sont souvent meme PLUS rapides que les boucles indexees car le compilateur peut eliminer les verifications de bornes grace aux informations de type.

---

## Exemples pratiques

### Compteur de frequence de mots

```rust
use std::collections::HashMap;

fn frequence_mots(texte: &str) -> Vec<(String, usize)> {
    let mut compteur: HashMap<String, usize> = HashMap::new();

    texte.split_whitespace()
        .map(|mot| mot.to_lowercase())
        .map(|mot| mot.trim_matches(|c: char| !c.is_alphanumeric()).to_string())
        .filter(|mot| !mot.is_empty())
        .for_each(|mot| {
            *compteur.entry(mot).or_insert(0) += 1;
        });

    let mut resultat: Vec<(String, usize)> = compteur.into_iter().collect();
    resultat.sort_by(|a, b| b.1.cmp(&a.1));  // Tri par frequence decroissante
    resultat
}

fn main() {
    let texte = "Le chat mange le poisson. Le chat dort. Le poisson nage.";
    let freq = frequence_mots(texte);
    for (mot, count) in &freq {
        println!("{:>12} : {}", mot, count);
    }
    // le : 4, chat : 2, poisson : 2, mange : 1, dort : 1, nage : 1
}
```

### Pipeline de donnees

```rust
#[derive(Debug)]
struct Mesure {
    capteur: String,
    valeur: f64,
    timestamp: u64,
}

fn analyser(mesures: &[Mesure]) {
    // Moyenne par capteur
    let mut par_capteur: std::collections::HashMap<&str, Vec<f64>> =
        std::collections::HashMap::new();

    mesures.iter().for_each(|m| {
        par_capteur.entry(&m.capteur)
            .or_insert_with(Vec::new)
            .push(m.valeur);
    });

    for (capteur, valeurs) in &par_capteur {
        let moyenne = valeurs.iter().sum::<f64>() / valeurs.len() as f64;
        let min = valeurs.iter().cloned().reduce(f64::min).unwrap();
        let max = valeurs.iter().cloned().reduce(f64::max).unwrap();
        println!("{}: moyenne={:.1}, min={:.1}, max={:.1}",
            capteur, moyenne, min, max);
    }

    // Les 5 mesures les plus elevees
    let mut top: Vec<&Mesure> = mesures.iter().collect();
    top.sort_by(|a, b| b.valeur.partial_cmp(&a.valeur).unwrap());
    println!("\nTop 5 :");
    for m in top.iter().take(5) {
        println!("  {} = {:.1} (capteur {})", m.timestamp, m.valeur, m.capteur);
    }
}
```

---

## Carte Mentale

```
                        COLLECTIONS ET ITERATEURS
                                  |
            +---------------------+---------------------+
            |                     |                     |
        COLLECTIONS           CLOSURES             ITERATEURS
            |                     |                     |
    +-------+-------+        |args| body          Trait Iterator
    |       |       |             |                next() -> Option
  Vec<T>  String  HashMap       Fn                     |
    |       |       |          FnMut            +------+------+
  push    UTF-8   entry       FnOnce           |             |
  pop     chars   or_insert    move        Adaptateurs  Consommateurs
  get     bytes                               |             |
  slice   split               map, filter    collect, sum
    |       |                 enumerate      count, fold
  &[T]    &str               zip, take       min, max
                             chain            find, any
                             flat_map         reduce

  C : malloc/realloc         C : pointeurs    C : for(i=0;i<n;i++)
      void*                      de fonctions      boucles manuelles
      free()                                        aucune composition
```

---

## Exercices

### Exercice 1 : Statistiques sur un Vec

Implementez une fonction qui calcule la moyenne, la mediane et le mode (valeur la plus frequente) d'un `Vec<i32>`. Utilisez les iterateurs et HashMap.

```rust
use std::collections::HashMap;

fn statistiques(nombres: &[i32]) -> Option<(f64, f64, i32)> {
    // Retourne (moyenne, mediane, mode) ou None si vide
    // Indice : pour la mediane, triez une copie
    // Indice : pour le mode, utilisez HashMap + entry
    todo!()
}

fn main() {
    let donnees = vec![3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5];
    if let Some((moy, med, mode)) = statistiques(&donnees) {
        println!("Moyenne : {:.2}", moy);
        println!("Mediane : {:.1}", med);
        println!("Mode    : {}", mode);
    }
}
```

### Exercice 2 : Transformation de texte pipeline

Ecrivez une fonction qui prend un texte, retire la ponctuation, met en minuscules, filtre les mots de moins de 3 lettres, et retourne les mots uniques tries par ordre alphabetique. Utilisez uniquement des iterateurs (pas de boucle `for`).

```rust
use std::collections::HashSet;

fn mots_significatifs(texte: &str) -> Vec<String> {
    // Pipeline d'iterateurs :
    // 1. split_whitespace
    // 2. map : retirer la ponctuation (trim_matches)
    // 3. map : to_lowercase
    // 4. filter : longueur >= 3
    // 5. collect dans un HashSet (pour l'unicite)
    // 6. Convertir en Vec et trier
    todo!()
}

fn main() {
    let texte = "Le petit chat est sur le tapis. Le chat dort sur le tapis !";
    let mots = mots_significatifs(texte);
    println!("{:?}", mots);
    // Attendu : ["chat", "dort", "est", "petit", "sur", "tapis"]
}
```

### Exercice 3 : Gestionnaire de scores

Creez une structure `TableauScores` qui utilise un `HashMap<String, Vec<u32>>` pour stocker les scores de joueurs. Implementez :
- `ajouter_score(nom, score)` : ajoute un score
- `moyenne(nom)` : retourne la moyenne d'un joueur (Option)
- `meilleur_joueur()` : retourne le joueur avec la meilleure moyenne
- `top_n(n)` : retourne les n meilleurs scores tous joueurs confondus

```rust
use std::collections::HashMap;

struct TableauScores {
    scores: HashMap<String, Vec<u32>>,
}

impl TableauScores {
    fn new() -> Self {
        TableauScores { scores: HashMap::new() }
    }

    fn ajouter_score(&mut self, nom: &str, score: u32) {
        // Utiliser l'API entry
        todo!()
    }

    fn moyenne(&self, nom: &str) -> Option<f64> {
        // Retourner None si le joueur n'existe pas ou n'a pas de scores
        todo!()
    }

    fn meilleur_joueur(&self) -> Option<(&str, f64)> {
        // Retourner le nom et la moyenne du meilleur joueur
        // Indice : iter().map(...).max_by(...)
        todo!()
    }

    fn top_n(&self, n: usize) -> Vec<(&str, u32)> {
        // Retourner les n meilleurs scores (nom, score) tries decroissant
        // Indice : flat_map pour aplatir les scores de chaque joueur
        todo!()
    }
}

fn main() {
    let mut tableau = TableauScores::new();
    tableau.ajouter_score("Alice", 95);
    tableau.ajouter_score("Alice", 88);
    tableau.ajouter_score("Bob", 92);
    tableau.ajouter_score("Bob", 97);
    tableau.ajouter_score("Claire", 100);

    println!("Moyenne Alice : {:?}", tableau.moyenne("Alice"));
    println!("Meilleur : {:?}", tableau.meilleur_joueur());
    println!("Top 3 : {:?}", tableau.top_n(3));
}
```

### Exercice 4 : Reimplementer des fonctions d'iterateurs

Reimplementez `map`, `filter` et `fold` sans utiliser les methodes standard. Cela vous aidera a comprendre comment les iterateurs fonctionnent en interne.

```rust
fn mon_map<T, U, F>(v: &[T], f: F) -> Vec<U>
where
    F: Fn(&T) -> U,
{
    // Utiliser une boucle for et push()
    todo!()
}

fn mon_filter<T, F>(v: &[T], predicat: F) -> Vec<&T>
where
    F: Fn(&T) -> bool,
{
    todo!()
}

fn mon_fold<T, U, F>(v: &[T], init: U, f: F) -> U
where
    F: Fn(U, &T) -> U,
{
    todo!()
}

fn main() {
    let nombres = vec![1, 2, 3, 4, 5];

    let doubles = mon_map(&nombres, |x| x * 2);
    println!("map : {:?}", doubles);  // [2, 4, 6, 8, 10]

    let pairs = mon_filter(&nombres, |x| *x % 2 == 0);
    println!("filter : {:?}", pairs);  // [2, 4]

    let somme = mon_fold(&nombres, 0, |acc, x| acc + x);
    println!("fold : {}", somme);  // 15
}
```

---

## Liens

- [[02 - Ownership et Borrowing]] --- Les regles d'ownership s'appliquent aux collections et aux closures
- [[06 - Traits et Generiques]] --- Les traits Iterator, Fn, et les generiques qui rendent les collections possibles
