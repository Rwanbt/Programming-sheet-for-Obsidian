# Traits et Generiques

Les generiques et les traits sont les deux piliers de l'abstraction en Rust. Ensemble, ils permettent d'ecrire du code qui fonctionne avec n'importe quel type, tout en garantissant la securite a la compilation et en maintenant les performances d'un code ecrit a la main pour chaque type specifique. Si vous venez du C, pensez aux generiques comme un remplacement type-safe de `void*`, et aux traits comme un remplacement structure des pointeurs de fonctions dans les structs.

Ce chapitre couvre les generiques (fonctions, structs, enums, implementations), les traits (definition, implementation, bounds, objets), les traits standard de la bibliotheque, et les patterns avances (surcharge d'operateurs, newtype, supertraits). A chaque etape, nous comparons avec les techniques equivalentes en C pour mettre en evidence les gains en securite et en expressivite.

---

## Generiques

### Fonctions generiques

```rust
// Sans generiques : dupliquer le code pour chaque type
fn plus_grand_i32(a: i32, b: i32) -> i32 {
    if a > b { a } else { b }
}

fn plus_grand_f64(a: f64, b: f64) -> f64 {
    if a > b { a } else { b }
}

// Avec generiques : une seule definition
fn plus_grand<T: PartialOrd>(a: T, b: T) -> T {
    if a > b { a } else { b }
}

fn main() {
    println!("{}", plus_grand(10, 20));       // 20 (i32)
    println!("{}", plus_grand(3.14, 2.71));   // 3.14 (f64)
    println!("{}", plus_grand('z', 'a'));      // 'z' (char)
}
```

```
Syntaxe des generiques :

fn nom<T>(param: T) -> T          Un parametre de type
fn nom<T, U>(a: T, b: U)          Deux parametres de type
fn nom<T: Clone>(a: T)             Avec contrainte (trait bound)
fn nom<T: Clone + Debug>(a: T)     Plusieurs contraintes
```

### Comparaison avec void* en C

```c
// En C : "generique" avec void* = AUCUNE securite de type

#include <string.h>

// Fonction "generique" de comparaison
void* plus_grand(void *a, void *b, int (*cmp)(const void*, const void*)) {
    if (cmp(a, b) > 0) return a;
    return b;
}

int cmp_int(const void *a, const void *b) {
    return *(int*)a - *(int*)b;  // Cast dangereux
}

int main(void) {
    int x = 10, y = 20;
    int *resultat = (int*)plus_grand(&x, &y, cmp_int);
    // Ca marche... mais :
    // - Rien n'empeche de passer un float* au lieu d'un int*
    // - Le cast (int*) est verifie a ZERO pourcent par le compilateur
    // - Si on passe le mauvais comparateur : UB

    // Erreur silencieuse (compile sans warning) :
    double d = 3.14;
    int *oops = (int*)plus_grand(&d, &y, cmp_int);  // UB total
    return 0;
}
```

```rust
// En Rust : le compilateur REFUSE les types incompatibles
fn plus_grand<T: PartialOrd>(a: T, b: T) -> T {
    if a > b { a } else { b }
}

fn main() {
    let r = plus_grand(10, 20);        // OK : i32 et i32
    // let r = plus_grand(10, 3.14);   // ERREUR : i32 != f64
    // let r = plus_grand(10, "abc");  // ERREUR : i32 != &str
}
```

```
Comparaison void* vs generiques :

C (void*) :                         Rust (generiques) :
+--------------------------------+  +--------------------------------+
| Aucune verification de type    |  | Type verifie a la compilation  |
| Cast manuels dangereux         |  | Pas de cast necessaire         |
| Erreurs a l'execution (UB)     |  | Erreurs a la compilation       |
| Besoin de passer la taille     |  | Taille connue via le type      |
| Pointeurs de fonctions         |  | Traits                         |
| Pas de code genere par type    |  | Monomorphisation               |
+--------------------------------+  +--------------------------------+
```

### Structs generiques

```rust
#[derive(Debug)]
struct Point<T> {
    x: T,
    y: T,
}

// Point avec deux types differents
#[derive(Debug)]
struct Paire<T, U> {
    premier: T,
    second: U,
}

fn main() {
    let entier = Point { x: 5, y: 10 };           // Point<i32>
    let flottant = Point { x: 1.0, y: 4.0 };      // Point<f64>
    // let mixte = Point { x: 5, y: 4.0 };         // ERREUR : x et y doivent etre du meme type T

    let paire = Paire { premier: "hello", second: 42 };  // Paire<&str, i32>
}
```

### Enums generiques

Vous les connaissez deja ! `Option<T>` et `Result<T, E>` sont des enums generiques :

```rust
// Definitions de la bibliotheque standard
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}

// Votre propre enum generique
#[derive(Debug)]
enum Arbre<T> {
    Feuille(T),
    Noeud {
        valeur: T,
        gauche: Box<Arbre<T>>,
        droite: Box<Arbre<T>>,
    },
    Vide,
}
```

### Blocs impl generiques

```rust
#[derive(Debug)]
struct Point<T> {
    x: T,
    y: T,
}

// Implementation generique : disponible pour tout T
impl<T> Point<T> {
    fn new(x: T, y: T) -> Self {
        Point { x, y }
    }

    fn x(&self) -> &T {
        &self.x
    }

    fn y(&self) -> &T {
        &self.y
    }
}

// Implementation specifique : seulement pour Point<f64>
impl Point<f64> {
    fn distance_origine(&self) -> f64 {
        (self.x * self.x + self.y * self.y).sqrt()
    }
}

// Implementation avec contrainte
impl<T: std::fmt::Display> Point<T> {
    fn afficher(&self) {
        println!("({}, {})", self.x, self.y);
    }
}

fn main() {
    let p = Point::new(3.0, 4.0);
    println!("Distance : {}", p.distance_origine());  // 5.0
    p.afficher();  // (3, 4)

    let q = Point::new(1, 2);
    q.afficher();  // (1, 2)
    // q.distance_origine();  // ERREUR : pas disponible pour Point<i32>
}
```

### Monomorphisation : zero-cost

> [!info] Comment Rust rend les generiques gratuits
> Le compilateur Rust effectue la **monomorphisation** : il genere une version specialisee du code pour chaque type concret utilise. `plus_grand::<i32>` et `plus_grand::<f64>` deviennent deux fonctions separees dans le binaire, chacune optimisee pour son type. Il n'y a AUCUN cout a l'execution.

```
Monomorphisation :

Code source :                     Code genere par le compilateur :

fn plus_grand<T: Ord>             fn plus_grand_i32(a: i32, b: i32) -> i32
    (a: T, b: T) -> T                 { if a > b { a } else { b } }
{                          =>
    if a > b { a }                fn plus_grand_char(a: char, b: char) -> char
    else { b }                         { if a > b { a } else { b } }
}
                                  (Chaque version est optimisee individuellement)
plus_grand(10, 20);
plus_grand('a', 'z');

Cout a l'execution : ZERO
Cout en taille du binaire : le code est duplique (compromis)
```

```
Comparaison :

C (void*) :                   Rust (generiques + monomorphisation) :
  - 1 seule fonction           - N fonctions specialisees
  - Indirection via void*      - Appel direct, pas d'indirection
  - Pas d'inlining possible    - Inlining agressif possible
  - Taille binaire minimale    - Binaire un peu plus gros
  - Plus lent a l'execution    - Performance maximale
```

---

## Traits

### Definir un trait

Un trait definit un ensemble de comportements qu'un type peut implementer.

```rust
trait Forme {
    // Methode requise (pas de corps = l'implementeur DOIT la definir)
    fn aire(&self) -> f64;
    fn perimetre(&self) -> f64;

    // Methode avec implementation par defaut
    fn description(&self) -> String {
        format!("Forme d'aire {:.2} et de perimetre {:.2}",
            self.aire(), self.perimetre())
    }
}
```

### Implementer un trait

```rust
struct Cercle {
    rayon: f64,
}

struct Rectangle {
    largeur: f64,
    hauteur: f64,
}

impl Forme for Cercle {
    fn aire(&self) -> f64 {
        std::f64::consts::PI * self.rayon * self.rayon
    }

    fn perimetre(&self) -> f64 {
        2.0 * std::f64::consts::PI * self.rayon
    }

    // description() utilise l'implementation par defaut
}

impl Forme for Rectangle {
    fn aire(&self) -> f64 {
        self.largeur * self.hauteur
    }

    fn perimetre(&self) -> f64 {
        2.0 * (self.largeur + self.hauteur)
    }

    // On peut surcharger l'implementation par defaut
    fn description(&self) -> String {
        format!("Rectangle {}x{} (aire={:.2})",
            self.largeur, self.hauteur, self.aire())
    }
}

fn main() {
    let c = Cercle { rayon: 5.0 };
    let r = Rectangle { largeur: 4.0, hauteur: 6.0 };

    println!("{}", c.description());  // Forme d'aire 78.54 et de perimetre 31.42
    println!("{}", r.description());  // Rectangle 4x6 (aire=24.00)
}
```

> [!tip] Analogie
> Un trait est comme un contrat de travail. Il specifie les competences requises (methodes sans corps) et fournit certaines competences de base (methodes par defaut). Chaque employe (type) qui signe le contrat (impl Trait for Type) doit posseder les competences requises mais herite automatiquement des competences de base, qu'il peut aussi personnaliser.

### Traits comme parametres

Il y a trois facons de passer un trait en parametre :

```rust
// 1. impl Trait (syntaxe sucree, monomorphise)
fn afficher_aire(forme: &impl Forme) {
    println!("Aire : {:.2}", forme.aire());
}

// 2. Trait bound (equivalent explicite)
fn afficher_aire_v2<T: Forme>(forme: &T) {
    println!("Aire : {:.2}", forme.aire());
}

// 3. Trait object (dispatch dynamique)
fn afficher_aire_v3(forme: &dyn Forme) {
    println!("Aire : {:.2}", forme.aire());
}

fn main() {
    let c = Cercle { rayon: 3.0 };
    let r = Rectangle { largeur: 2.0, hauteur: 5.0 };

    afficher_aire(&c);     // Monomorphise : genere afficher_aire_Cercle
    afficher_aire(&r);     // Monomorphise : genere afficher_aire_Rectangle

    afficher_aire_v3(&c);  // Dispatch dynamique : vtable
    afficher_aire_v3(&r);  // Dispatch dynamique : vtable

    // Avantage du dyn : on peut mettre des types differents dans un Vec
    let formes: Vec<&dyn Forme> = vec![&c, &r];
    for f in &formes {
        println!("{}", f.description());
    }
}
```

### Trait bounds et clauses where

```rust
// Bounds multiples avec +
fn imprimer_et_cloner<T: std::fmt::Display + Clone>(val: &T) {
    println!("{}", val);
    let copie = val.clone();
}

// Clause where : plus lisible quand il y a beaucoup de bounds
fn traiter<T, U>(t: &T, u: &U) -> String
where
    T: std::fmt::Display + Clone + PartialOrd,
    U: std::fmt::Debug + Default,
{
    format!("{} et {:?}", t, u)
}

// Retourner un impl Trait
fn creer_iterateur() -> impl Iterator<Item = i32> {
    (0..10).filter(|x| x % 2 == 0)
}

fn main() {
    let it = creer_iterateur();
    let pairs: Vec<i32> = it.collect();  // [0, 2, 4, 6, 8]
}
```

---

## Traits standard de la bibliotheque

### Display et Debug

```rust
use std::fmt;

#[derive(Debug)]  // Debug peut etre derive automatiquement
struct Couleur {
    r: u8,
    g: u8,
    b: u8,
}

// Display doit etre implemente manuellement
impl fmt::Display for Couleur {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "#{:02X}{:02X}{:02X}", self.r, self.g, self.b)
    }
}

fn main() {
    let c = Couleur { r: 255, g: 128, b: 0 };
    println!("{}", c);    // Display : #FF8000
    println!("{:?}", c);  // Debug   : Couleur { r: 255, g: 128, b: 0 }
}
```

### Clone et Copy

```rust
// Clone : duplication explicite (peut etre couteux)
#[derive(Clone, Debug)]
struct Donnees {
    valeurs: Vec<i32>,  // Vec n'est pas Copy (allocation heap)
}

// Copy : duplication implicite (doit etre cheap, stack-only)
#[derive(Copy, Clone, Debug)]
struct Point {
    x: f64,
    y: f64,
}

fn main() {
    // Clone : .clone() obligatoire
    let d1 = Donnees { valeurs: vec![1, 2, 3] };
    let d2 = d1.clone();  // Copie explicite du Vec (allocation)
    println!("{:?}", d1);  // d1 est toujours valide

    // Copy : copie implicite a chaque utilisation
    let p1 = Point { x: 1.0, y: 2.0 };
    let p2 = p1;           // Copie implicite (juste memcpy)
    println!("{:?}", p1);  // p1 est toujours valide !
    // (Sans Copy, p1 serait move et inutilisable)
}
```

> [!warning] Copy ne peut etre implemente que pour les types "stack-only"
> Un type ne peut implementer `Copy` que si TOUS ses champs sont eux-memes `Copy`. Les types qui gerent de la memoire heap (`String`, `Vec`, `Box`, etc.) ne sont PAS `Copy` car copier silencieusement une allocation serait dangereux et couteux.

### Comparaison et tri : PartialEq, Eq, PartialOrd, Ord

```rust
#[derive(Debug, PartialEq, Eq, PartialOrd, Ord)]
struct Score {
    points: u32,
    nom: String,
}

fn main() {
    let a = Score { points: 95, nom: "Alice".to_string() };
    let b = Score { points: 87, nom: "Bob".to_string() };

    // PartialEq : == et !=
    println!("{}", a == b);  // false
    println!("{}", a != b);  // true

    // PartialOrd / Ord : <, >, <=, >=, sort
    println!("{}", a > b);   // true (compare d'abord points, puis nom)

    let mut scores = vec![
        Score { points: 78, nom: "Claire".to_string() },
        Score { points: 95, nom: "Alice".to_string() },
        Score { points: 87, nom: "Bob".to_string() },
    ];
    scores.sort();  // Tri avec Ord
    for s in &scores {
        println!("{}: {}", s.nom, s.points);
    }
}
```

```
Les traits de comparaison :

PartialEq           Eq                 PartialOrd          Ord
  == !=           (= PartialEq       < > <= >=         (= PartialOrd
                   + reflexivite)                        + ordre total)
                       |                                     |
   f64 : PartialEq    |    f64 : PartialOrd                 |
   (NaN != NaN)        |    (NaN n'est pas comparable)       |
                       |                                     |
   i32 : Eq            |    i32 : Ord                        |
   (tout est           |    (tout est comparable)            |
    comparable)        |                                     |

Note : f64 implemente PartialEq et PartialOrd mais PAS Eq ni Ord
       car NaN != NaN (IEEE 754)
```

### Hash et Default

```rust
use std::collections::HashMap;

#[derive(Debug, Hash, PartialEq, Eq)]
struct Identifiant {
    categorie: String,
    numero: u32,
}

#[derive(Debug, Default)]
struct Configuration {
    port: u16,         // default = 0
    hote: String,      // default = ""
    verbose: bool,     // default = false
    max_conn: usize,   // default = 0
}

fn main() {
    // Hash : utilisable comme cle de HashMap
    let mut registre: HashMap<Identifiant, String> = HashMap::new();
    registre.insert(
        Identifiant { categorie: "user".to_string(), numero: 42 },
        "Alice".to_string(),
    );

    // Default : valeurs par defaut
    let config = Configuration::default();
    println!("{:?}", config);
    // Configuration { port: 0, hote: "", verbose: false, max_conn: 0 }

    // Struct update syntax avec Default
    let config = Configuration {
        port: 8080,
        verbose: true,
        ..Configuration::default()  // Le reste prend les valeurs par defaut
    };
}
```

### From et Into

```rust
#[derive(Debug)]
struct Celsius(f64);

#[derive(Debug)]
struct Fahrenheit(f64);

impl From<Fahrenheit> for Celsius {
    fn from(f: Fahrenheit) -> Self {
        Celsius((f.0 - 32.0) * 5.0 / 9.0)
    }
}

// Into est automatiquement implemente quand From est implemente !

fn main() {
    let f = Fahrenheit(212.0);
    let c = Celsius::from(f);             // From explicite
    println!("{:?}", c);                   // Celsius(100.0)

    let f2 = Fahrenheit(32.0);
    let c2: Celsius = f2.into();           // Into implicite
    println!("{:?}", c2);                  // Celsius(0.0)

    // Tres utile pour les conversions d'erreur (operateur ?)
    // et les constructeurs flexibles
}
```

```
Conversions standard :

From<T> for U          U::from(t)          Conversion explicite
Into<U> for T          t.into()            Conversion implicite (derive de From)
TryFrom<T> for U       U::try_from(t)      Conversion faillible -> Result
TryInto<U> for T       t.try_into()        Conversion faillible (derive de TryFrom)

Regle : implementez From, vous obtenez Into gratuitement
        implementez TryFrom, vous obtenez TryInto gratuitement
```

### Tableau recapitulatif des traits derivables

```
Trait        #[derive]   Utilite
----------------------------------------------
Debug        Oui         Affichage {:?}
Clone        Oui         Duplication explicite
Copy         Oui         Duplication implicite
PartialEq    Oui         == et !=
Eq           Oui         Egalite reflexive
PartialOrd   Oui         < > <= >=
Ord          Oui         Ordre total + sort()
Hash         Oui         Cle de HashMap/HashSet
Default      Oui         Valeurs par defaut

Display      NON         Affichage {} (impl manuelle)
From/Into    NON         Conversions (impl manuelle)
Iterator     NON         Iteration (impl manuelle)
Error        NON         Type d'erreur (impl manuelle, ou thiserror)
```

---

## Trait Objects : dispatch dynamique

### Le concept de vtable

Quand on utilise `&dyn Trait`, Rust utilise un **dispatch dynamique** : l'appel de methode est resolu a l'execution via une vtable (table de fonctions virtuelles).

```
Dispatch statique (generiques) :         Dispatch dynamique (dyn Trait) :

Compile-time :                           Runtime :
  Le compilateur sait le type exact        Le compilateur ne sait pas
  -> genere du code specifique             -> utilise un pointeur fat

fn f<T: Forme>(s: &T)                   fn f(s: &dyn Forme)
  -> f_Cercle, f_Rectangle                -> 1 seule fonction
  -> appel direct                          -> lookup dans la vtable

  +---[ &Cercle ]---+                     +---[ &dyn Forme ]---+
  | ptr vers data   |                     | ptr vers data      |
  +-----------------+                     | ptr vers vtable    |
                                          +--------------------+
                                                    |
                                          +---------v----------+
                                          | aire()    -> 0x... |
                                          | perimetre -> 0x... |
                                          | drop()    -> 0x... |
                                          +--------------------+
```

```rust
trait Animal {
    fn cri(&self) -> &str;
    fn nom(&self) -> &str;
}

struct Chat { nom: String }
struct Chien { nom: String }

impl Animal for Chat {
    fn cri(&self) -> &str { "Miaou" }
    fn nom(&self) -> &str { &self.nom }
}

impl Animal for Chien {
    fn cri(&self) -> &str { "Ouaf" }
    fn nom(&self) -> &str { &self.nom }
}

fn main() {
    // Vec de trait objects : types heterogenes dans un seul Vec
    let animaux: Vec<Box<dyn Animal>> = vec![
        Box::new(Chat { nom: "Felix".to_string() }),
        Box::new(Chien { nom: "Rex".to_string() }),
        Box::new(Chat { nom: "Garfield".to_string() }),
    ];

    for animal in &animaux {
        println!("{} fait {}", animal.nom(), animal.cri());
    }
}
```

### Comparaison avec les pointeurs de fonctions en C

```c
// En C : "polymorphisme" via pointeurs de fonctions dans les structs
typedef struct {
    const char* (*cri)(void *self);
    const char* (*nom)(void *self);
} AnimalVtable;

typedef struct {
    AnimalVtable *vtable;
    char nom[50];
} Animal;

const char* chat_cri(void *self) { return "Miaou"; }
const char* chat_nom(void *self) { return ((Animal*)self)->nom; }

const char* chien_cri(void *self) { return "Ouaf"; }
const char* chien_nom(void *self) { return ((Animal*)self)->nom; }

AnimalVtable chat_vtable = { chat_cri, chat_nom };
AnimalVtable chien_vtable = { chien_cri, chien_nom };

int main(void) {
    Animal felix = { &chat_vtable, "Felix" };
    Animal rex = { &chien_vtable, "Rex" };

    // Ca marche, mais :
    // - Aucune verification que la vtable est correcte
    // - Les casts void* peuvent etre faux
    // - Rien n'empeche d'assigner la mauvaise vtable
    printf("%s fait %s\n",
        felix.vtable->nom(&felix),
        felix.vtable->cri(&felix));
    return 0;
}
```

```
C (pointeurs de fonctions) vs Rust (dyn Trait) :

C :                                 Rust :
- Vtable manuelle                   - Vtable automatique
- void* partout                     - Types verifies
- Aucune garantie de coherence      - Le compilateur verifie tout
- Facile d'oublier un champ         - impl exige TOUTES les methodes
- Pas d'heritage des defaults       - Methodes par defaut dans le trait
```

> [!warning] Quand utiliser dyn Trait vs generiques
> Preferez les **generiques** (dispatch statique) par defaut : zero overhead, inlining possible. Utilisez `dyn Trait` quand vous avez besoin de types **heterogenes** dans une meme collection ou quand le type exact n'est pas connu a la compilation (plugins, callbacks configurables, etc.).

---

## Types associes vs generiques dans les traits

### Types associes

```rust
// Avec un type associe : UN SEUL type possible par implementation
trait Iterateur {
    type Item;  // Type associe

    fn suivant(&mut self) -> Option<Self::Item>;
}

struct Compteur {
    valeur: u32,
    max: u32,
}

impl Iterateur for Compteur {
    type Item = u32;  // Un Compteur produit des u32, point.

    fn suivant(&mut self) -> Option<u32> {
        if self.valeur < self.max {
            self.valeur += 1;
            Some(self.valeur)
        } else {
            None
        }
    }
}
```

### Generiques dans les traits

```rust
// Avec un generique : PLUSIEURS implementations possibles pour le meme type
trait Convertir<T> {
    fn convertir(&self) -> T;
}

struct Temperature(f64);  // en Celsius

impl Convertir<f64> for Temperature {
    fn convertir(&self) -> f64 {
        self.0 * 9.0 / 5.0 + 32.0  // Fahrenheit
    }
}

impl Convertir<String> for Temperature {
    fn convertir(&self) -> String {
        format!("{:.1} C", self.0)
    }
}

fn main() {
    let t = Temperature(100.0);
    let f: f64 = t.convertir();       // 212.0
    let s: String = t.convertir();     // "100.0 C"
}
```

```
Type associe vs generique dans un trait :

Type associe (type Item)           Generique (trait Foo<T>)
+-------------------------------+  +-------------------------------+
| UN type par implementation    |  | PLUSIEURS implementations     |
| impl Iterator for Vec<i32>   |  | impl From<i32> for Foo        |
|   type Item = i32             |  | impl From<String> for Foo     |
|                               |  | impl From<bool> for Foo       |
| Pas d'ambiguite a l'appel    |  | Parfois besoin de turbofish   |
| iter.next()                   |  | Foo::from::<i32>(42)          |
+-------------------------------+  +-------------------------------+

Regle : utilisez un type associe quand il n'y a qu'un choix logique
        utilisez un generique quand il peut y avoir plusieurs implementations
```

---

## Supertraits

Un supertrait est un trait qui requiert un autre trait comme prerequis.

```rust
use std::fmt;

// Affichable requiert Display (supertrait)
trait Affichable: fmt::Display {
    fn afficher_encadre(&self) {
        let texte = self.to_string();  // to_string() vient de Display
        let largeur = texte.len() + 4;
        println!("+{}+", "-".repeat(largeur - 2));
        println!("| {} |", texte);
        println!("+{}+", "-".repeat(largeur - 2));
    }
}

// Pour implementer Affichable, il FAUT aussi implementer Display
struct Message(String);

impl fmt::Display for Message {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "{}", self.0)
    }
}

impl Affichable for Message {}  // Utilise l'implementation par defaut

fn main() {
    let m = Message("Bonjour Rust !".to_string());
    m.afficher_encadre();
    // +------------------+
    // | Bonjour Rust !   |
    // +------------------+
}
```

---

## La regle orpheline (Orphan Rule)

> [!warning] Regle orpheline
> Vous ne pouvez implementer un trait pour un type que si **au moins l'un des deux** (le trait ou le type) est defini dans votre crate. Cela empeche deux crates differents de fournir des implementations conflictuelles.

```rust
// OK : notre trait, type standard
trait MonTrait {
    fn faire(&self);
}
impl MonTrait for String {  // OK : MonTrait est a nous
    fn faire(&self) { /* ... */ }
}

// OK : trait standard, notre type
struct MonType;
impl std::fmt::Display for MonType {  // OK : MonType est a nous
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "MonType")
    }
}

// INTERDIT : trait standard, type standard
// impl std::fmt::Display for Vec<i32> { ... }
// ERREUR : ni Display ni Vec<i32> ne sont definis dans notre crate
```

---

## Surcharge d'operateurs

Rust permet de surcharger les operateurs en implementant des traits du module `std::ops`.

```rust
use std::ops::{Add, Mul, Neg, Index};

#[derive(Debug, Clone, Copy, PartialEq)]
struct Vec2 {
    x: f64,
    y: f64,
}

impl Vec2 {
    fn new(x: f64, y: f64) -> Self {
        Vec2 { x, y }
    }

    fn norme(&self) -> f64 {
        (self.x * self.x + self.y * self.y).sqrt()
    }
}

// Addition : v1 + v2
impl Add for Vec2 {
    type Output = Vec2;

    fn add(self, other: Vec2) -> Vec2 {
        Vec2 {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

// Multiplication par un scalaire : v * 3.0
impl Mul<f64> for Vec2 {
    type Output = Vec2;

    fn mul(self, scalaire: f64) -> Vec2 {
        Vec2 {
            x: self.x * scalaire,
            y: self.y * scalaire,
        }
    }
}

// Negation : -v
impl Neg for Vec2 {
    type Output = Vec2;

    fn neg(self) -> Vec2 {
        Vec2 { x: -self.x, y: -self.y }
    }
}

// Indexation : v[0] et v[1]
impl Index<usize> for Vec2 {
    type Output = f64;

    fn index(&self, i: usize) -> &f64 {
        match i {
            0 => &self.x,
            1 => &self.y,
            _ => panic!("Index {} hors limites pour Vec2 (0 ou 1)", i),
        }
    }
}

// Display pour un affichage propre
impl std::fmt::Display for Vec2 {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "({:.2}, {:.2})", self.x, self.y)
    }
}

fn main() {
    let a = Vec2::new(3.0, 4.0);
    let b = Vec2::new(1.0, 2.0);

    println!("a + b = {}", a + b);          // (4.00, 6.00)
    println!("a * 2 = {}", a * 2.0);        // (6.00, 8.00)
    println!("-a    = {}", -a);              // (-3.00, -4.00)
    println!("a[0]  = {}", a[0]);            // 3.00
    println!("|a|   = {:.2}", a.norme());    // 5.00
}
```

```
Operateurs surchargables (selection) :

Trait         Operateur    Signature
------------------------------------------
Add           a + b        fn add(self, rhs) -> Output
Sub           a - b        fn sub(self, rhs) -> Output
Mul           a * b        fn mul(self, rhs) -> Output
Div           a / b        fn div(self, rhs) -> Output
Neg           -a           fn neg(self) -> Output
Not           !a           fn not(self) -> Output
Index         a[i]         fn index(&self, i) -> &Output
IndexMut      a[i] = v     fn index_mut(&mut self, i) -> &mut Output
Deref         *a           fn deref(&self) -> &Target
PartialEq     a == b       fn eq(&self, other) -> bool
PartialOrd    a < b        fn partial_cmp(&self, other) -> Option<Ordering>
```

---

## Le pattern Newtype

Le pattern newtype consiste a envelopper un type existant dans un tuple struct a un champ pour lui donner une identite distincte.

```rust
// Probleme : on peut melanger les metres et les secondes
fn mauvaise_vitesse(distance: f64, temps: f64) -> f64 {
    distance / temps  // Et si on passe les args dans le mauvais ordre ?
}

// Solution : newtype
struct Metres(f64);
struct Secondes(f64);
struct MetresParSeconde(f64);

impl std::ops::Div<Secondes> for Metres {
    type Output = MetresParSeconde;

    fn div(self, rhs: Secondes) -> MetresParSeconde {
        MetresParSeconde(self.0 / rhs.0)
    }
}

fn main() {
    let distance = Metres(100.0);
    let temps = Secondes(9.58);
    let vitesse = distance / temps;  // MetresParSeconde(10.44...)

    // let erreur = temps / distance;  // ERREUR : Div<Metres> non impl pour Secondes
}
```

> [!example] Newtype pour contourner la regle orpheline
> ```rust
> // On ne peut pas impl Display pour Vec<String> (regle orpheline)
> // Mais on peut creer un newtype :
> struct ListeNoms(Vec<String>);
> 
> impl std::fmt::Display for ListeNoms {
>     fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
>         write!(f, "{}", self.0.join(", "))
>     }
> }
> 
> fn main() {
>     let noms = ListeNoms(vec!["Alice".into(), "Bob".into(), "Claire".into()]);
>     println!("{}", noms);  // Alice, Bob, Claire
> }
> ```

---

## Comparaison globale : C vs Rust

```
C (void* + pointeurs de fonctions)     Rust (traits + generiques)
+--------------------------------------+--------------------------------------+
| void* pour le polymorphisme          | Generiques <T> pour le polymorphisme |
| Pointeurs de fonctions pour          | Traits pour les comportements        |
|   les comportements                  |                                      |
| Aucune verification de type          | Verification complete a la compil.   |
| Vtable manuelle (struct de ptrs)     | Vtable automatique (dyn Trait)       |
| Pas de methodes par defaut           | Methodes par defaut dans les traits  |
| Pas de surcharge d'operateurs        | Surcharge via traits (Add, etc.)     |
| Macros pour "generer" du code        | Monomorphisation automatique         |
| Pas de derive automatique            | #[derive(...)] pour les traits std   |
| Conventions informelles              | Contrats formels verifies            |
|                                      |                                      |
| Performance : identique              | Performance : identique              |
| (quand le code C est correct)        | (toujours correct par construction)  |
+--------------------------------------+--------------------------------------+
```

> [!tip] Analogie
> En C, le polymorphisme ressemble a un systeme de boites postales ou chaque boite a une etiquette ecrite au crayon : rien n'empeche de mettre la mauvaise lettre dans la mauvaise boite. En Rust, chaque boite a une serrure unique, et seule la cle correspondante peut l'ouvrir. Le facteur (compilateur) verifie chaque cle avant la livraison.

---

## Carte Mentale

```
                        TRAITS ET GENERIQUES
                               |
            +------------------+------------------+
            |                  |                  |
        GENERIQUES          TRAITS          TRAIT OBJECTS
            |                  |                  |
    +-------+-------+    +----+----+         dyn Trait
    |       |       |    |         |              |
  fn<T>  struct<T>  |  Definir   Impl         Vtable
         enum<T>    |  (methodes  for Type     (dispatch
         impl<T>    |   requises   |           dynamique)
            |       |   + defaut)  |              |
    Monomorphi-     |       |    Bounds      Heterogene
    sation          |    Orphan   T: Trait    Vec<Box<dyn T>>
    (zero-cost)     |    Rule     where           |
            |       |       |     +  multi    Comparaison C :
    C: void*   C: macros    |                 pointeurs de
    (danger)   (fragile)    |                 fonctions (UB)
                            |
                    +-------+-------+
                    |       |       |
                Standard  Surcharge Newtype
                traits    operateurs pattern
                    |       |       |
                Display   Add,Mul  Securite
                Clone     Index    de type
                From      Neg
                Iterator
                Default
```

---

## Exercices

### Exercice 1 : Trait Resizable avec generiques

Definissez un trait `Statistiques` qui fournit les methodes `min()`, `max()`, `moyenne()` et `ecart_type()`. Implementez-le pour `Vec<f64>` et pour votre propre struct `Echantillon<T>`.

```rust
trait Statistiques {
    fn min(&self) -> Option<f64>;
    fn max(&self) -> Option<f64>;
    fn moyenne(&self) -> Option<f64>;
    fn ecart_type(&self) -> Option<f64> {
        // Implementation par defaut utilisant moyenne()
        // ecart_type = sqrt( moyenne(xi^2) - moyenne(xi)^2 )
        todo!()
    }
}

impl Statistiques for Vec<f64> {
    fn min(&self) -> Option<f64> {
        // Indice : self.iter().cloned().reduce(f64::min)
        todo!()
    }

    fn max(&self) -> Option<f64> {
        todo!()
    }

    fn moyenne(&self) -> Option<f64> {
        todo!()
    }
}

fn afficher_stats(data: &impl Statistiques) {
    println!("Min : {:?}", data.min());
    println!("Max : {:?}", data.max());
    println!("Moy : {:?}", data.moyenne());
    println!("Std : {:?}", data.ecart_type());
}

fn main() {
    let notes = vec![12.0, 15.0, 8.0, 18.0, 14.0, 9.0];
    afficher_stats(&notes);
}
```

### Exercice 2 : Matrice avec surcharge d'operateurs

Creez une struct `Matrice2x2` et implementez les traits `Add`, `Mul`, `Display` et `PartialEq`. Ajoutez une methode `determinant()` et `inverse()`.

```rust
use std::ops::{Add, Mul};
use std::fmt;

#[derive(Debug, Clone, Copy)]
struct Matrice2x2 {
    // | a  b |
    // | c  d |
    a: f64, b: f64,
    c: f64, d: f64,
}

impl Matrice2x2 {
    fn new(a: f64, b: f64, c: f64, d: f64) -> Self {
        Matrice2x2 { a, b, c, d }
    }

    fn identite() -> Self {
        Matrice2x2::new(1.0, 0.0, 0.0, 1.0)
    }

    fn determinant(&self) -> f64 {
        todo!()
    }

    fn inverse(&self) -> Option<Matrice2x2> {
        // None si determinant == 0
        todo!()
    }
}

// Implementer Add, Mul (matrice * matrice), Display, PartialEq
// ...

fn main() {
    let a = Matrice2x2::new(1.0, 2.0, 3.0, 4.0);
    let b = Matrice2x2::new(5.0, 6.0, 7.0, 8.0);

    println!("A = {}", a);
    println!("B = {}", b);
    println!("A + B = {}", a + b);
    println!("A * B = {}", a * b);
    println!("det(A) = {}", a.determinant());
    println!("A^-1 = {:?}", a.inverse());
}
```

### Exercice 3 : Systeme d'entites avec trait objects

Creez un mini systeme d'entites de jeu video avec un trait `Entite` (methodes : `nom()`, `position()`, `mettre_a_jour()`, `afficher()`). Implementez-le pour `Joueur`, `Ennemi` et `Projectile`. Stockez-les dans un `Vec<Box<dyn Entite>>` et simulez quelques frames.

```rust
trait Entite {
    fn nom(&self) -> &str;
    fn position(&self) -> (f64, f64);
    fn mettre_a_jour(&mut self, dt: f64);
    fn est_actif(&self) -> bool { true }  // Defaut : toujours actif

    fn afficher(&self) {
        let (x, y) = self.position();
        println!("[{}] a ({:.1}, {:.1})", self.nom(), x, y);
    }
}

struct Joueur {
    nom: String,
    x: f64, y: f64,
    vitesse: f64,
}

struct Ennemi {
    x: f64, y: f64,
    hp: i32,
}

struct Projectile {
    x: f64, y: f64,
    dx: f64, dy: f64,
    actif: bool,
}

// Implementer Entite pour chaque type
// Creer un Vec<Box<dyn Entite>> et simuler une boucle de jeu

fn main() {
    let mut entites: Vec<Box<dyn Entite>> = vec![
        // Box::new(Joueur { ... }),
        // Box::new(Ennemi { ... }),
        // Box::new(Projectile { ... }),
    ];

    // Simuler 5 frames
    for frame in 0..5 {
        println!("--- Frame {} ---", frame);
        for entite in entites.iter_mut() {
            entite.mettre_a_jour(0.016);  // ~60 FPS
            if entite.est_actif() {
                entite.afficher();
            }
        }
    }
}
```

### Exercice 4 : Convertisseur universel avec From/Into

Creez un systeme de conversion entre differentes unites (Temperature, Distance, Masse) en utilisant les traits `From` et `Into`. Implementez les conversions bidirectionnelles.

```rust
#[derive(Debug, Clone, Copy)]
struct Celsius(f64);

#[derive(Debug, Clone, Copy)]
struct Fahrenheit(f64);

#[derive(Debug, Clone, Copy)]
struct Kelvin(f64);

// Implementer From pour toutes les paires de conversions
// Celsius <-> Fahrenheit
// Celsius <-> Kelvin
// Fahrenheit <-> Kelvin

// Bonus : implementer Display pour un affichage propre
// "25.00 C", "77.00 F", "298.15 K"

fn afficher_toutes_unites<T>(temp: T)
where
    T: Into<Celsius> + Into<Fahrenheit> + Into<Kelvin> + Copy,
{
    let c: Celsius = temp.into();
    let f: Fahrenheit = temp.into();
    let k: Kelvin = temp.into();
    println!("{:.2} C = {:.2} F = {:.2} K", c.0, f.0, k.0);
}

fn main() {
    let eau_bouillante = Celsius(100.0);
    afficher_toutes_unites(eau_bouillante);
    // 100.00 C = 212.00 F = 373.15 K
}
```

---

## Liens

- [[03 - Structs Enums et Pattern Matching]] --- Les structs et enums sont les types sur lesquels on implemente des traits
- [[05 - Collections et Iterateurs]] --- Le trait Iterator et les closures Fn/FnMut/FnOnce en pratique
- [[07 - Projet Rust CLI]] --- Mettre en pratique traits et generiques dans un projet complet
