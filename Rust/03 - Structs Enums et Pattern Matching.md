# Structs, Enums et Pattern Matching

Les structs et les enums sont les briques fondamentales pour creer vos propres types en Rust. Si vous venez du C, vous connaissez deja les `struct` et les `enum`. Mais Rust pousse ces concepts beaucoup plus loin : les structs peuvent avoir des **methodes attachees**, les enums peuvent contenir des **donnees**, et le pattern matching permet de les deconstruire de maniere elegante et sure.

Ce chapitre montre comment Rust resout deux problemes majeurs du C : l'absence de methodes sur les structs (fonctions separees, conventions de nommage) et l'absence de type somme (union taguee manuelle et fragile).

---

## Structs

### Definir et instancier une struct

```rust
// Definition
struct Utilisateur {
    nom: String,
    email: String,
    age: u32,
    actif: bool,
}

fn main() {
    // Instanciation : TOUS les champs doivent etre remplis
    let user1 = Utilisateur {
        nom: String::from("Alice"),
        email: String::from("alice@example.com"),
        age: 30,
        actif: true,
    };

    // Acces aux champs
    println!("{} a {} ans", user1.nom, user1.age);

    // Pour modifier, la variable ENTIERE doit etre mut
    let mut user2 = Utilisateur {
        nom: String::from("Bob"),
        email: String::from("bob@example.com"),
        age: 25,
        actif: true,
    };
    user2.age = 26;  // OK : user2 est mut
}
```

### Comparaison avec les structs C

```c
// En C : struct sans methodes, convention de nommage manuelle
typedef struct {
    char *nom;      // Qui possede cette memoire ? Mystere.
    char *email;    // Faut-il free() ? Qui sait.
    int age;
    int actif;      // Pas de bool en C89, on utilise int
} Utilisateur;

// Pas de constructeur, pas de methodes
Utilisateur creer_utilisateur(const char *nom, const char *email, int age) {
    Utilisateur u;
    u.nom = strdup(nom);    // malloc implicite
    u.email = strdup(email); // malloc implicite
    u.age = age;
    u.actif = 1;
    return u;
}

// Il faut une fonction separee pour liberer
void detruire_utilisateur(Utilisateur *u) {
    free(u->nom);
    free(u->email);
}
// Et si on oublie de l'appeler ? Memory leak.
```

```
Comparaison de l'organisation :

C :                                    Rust :
struct Utilisateur { ... };            struct Utilisateur { ... }
utilisateur_creer(...)                 impl Utilisateur {
utilisateur_detruire(...)                  fn new(...) -> Self { ... }
utilisateur_afficher(...)                  fn afficher(&self) { ... }
utilisateur_modifier_age(...)              fn modifier_age(&mut self, ...) { ... }
                                       }  // Methodes ATTACHEES au type
                                       // drop() automatique, pas de detruire()
```

> [!tip] Analogie
> En C, une struct est comme une boite sans mode d'emploi : vous devez ecrire des fonctions separees pour l'assembler, l'utiliser et la demonter. En Rust, une struct est comme un objet avec son mode d'emploi integre : les methodes sont attachees au type, et le destructeur est automatique.

### Field init shorthand

Quand le nom du parametre correspond au nom du champ :

```rust
fn creer_utilisateur(nom: String, email: String) -> Utilisateur {
    Utilisateur {
        nom,        // Raccourci : equivalent a nom: nom
        email,      // Raccourci : equivalent a email: email
        age: 0,
        actif: true,
    }
}
```

### Struct update syntax (..)

Creer une struct a partir d'une autre en ne changeant que certains champs :

```rust
fn main() {
    let user1 = Utilisateur {
        nom: String::from("Alice"),
        email: String::from("alice@example.com"),
        age: 30,
        actif: true,
    };

    // Creer user2 a partir de user1, en changeant seulement email et age
    let user2 = Utilisateur {
        email: String::from("alice.new@example.com"),
        age: 31,
        ..user1  // Prend les champs restants de user1
    };
    // ATTENTION : user1.nom a ete MOVE dans user2
    // println!("{}", user1.nom);  // ERREUR : value moved
    // Mais user1.age est toujours valide (i32 implemente Copy)
    println!("{}", user1.age);  // OK : 30
}
```

> [!warning] Move partiel avec ..
> La syntaxe `..other` **deplace** les champs qui n'implementent pas `Copy`. Dans l'exemple ci-dessus, `user1.nom` (un `String`) est move vers `user2`. Apres cela, `user1` ne peut plus etre utilise en entier, mais ses champs `Copy` (`age`, `actif`) restent accessibles.

### Tuple structs

Des structs sans noms de champs, utiles pour creer des types distincts :

```rust
// Deux types distincts, meme si la structure interne est identique
struct Couleur(u8, u8, u8);
struct Point(f64, f64, f64);

fn main() {
    let rouge = Couleur(255, 0, 0);
    let origine = Point(0.0, 0.0, 0.0);

    // Acces par index
    println!("R = {}", rouge.0);
    println!("x = {}", origine.0);

    // ERREUR : types incompatibles meme si la structure est identique
    // let p: Point = rouge;  // Ne compile pas !
}
```

> [!info] Newtype pattern
> Un pattern courant est le "newtype" : une tuple struct avec un seul champ, pour creer un type distinct qui encapsule un type existant.
> ```rust
> struct Metres(f64);
> struct Kilometres(f64);
>
> // On ne peut pas accidentellement additionner des metres et des kilometres
> // Le systeme de types l'interdit
> ```
> En C, `typedef double Metres;` et `typedef double Kilometres;` sont des alias : le compilateur les considere comme le meme type. Aucune protection.

### Unit structs

Des structs sans aucun champ, utiles comme marqueurs :

```rust
struct AlwaysEqual;

fn main() {
    let _sujet = AlwaysEqual;
    // Utile pour implementer des traits sans donnees
}
```

---

## Methodes et blocs impl

### Definir des methodes

```rust
#[derive(Debug)]
struct Rectangle {
    largeur: f64,
    hauteur: f64,
}

impl Rectangle {
    // Methode : prend &self (reference immutable vers l'instance)
    fn aire(&self) -> f64 {
        self.largeur * self.hauteur
    }

    // Methode : prend &mut self (reference mutable)
    fn agrandir(&mut self, facteur: f64) {
        self.largeur *= facteur;
        self.hauteur *= facteur;
    }

    // Methode : prend self (consomme l'instance, la detruit)
    fn en_carre(self) -> Rectangle {
        let cote = (self.largeur * self.hauteur).sqrt();
        Rectangle {
            largeur: cote,
            hauteur: cote,
        }
    }

    // Fonction associee (pas de self) : comme un constructeur
    fn new(largeur: f64, hauteur: f64) -> Self {
        Self { largeur, hauteur }
    }

    // Autre fonction associee
    fn carre(cote: f64) -> Self {
        Self {
            largeur: cote,
            hauteur: cote,
        }
    }
}

fn main() {
    let mut rect = Rectangle::new(30.0, 50.0);  // Fonction associee (::)
    println!("Aire = {}", rect.aire());          // Methode (.)
    rect.agrandir(2.0);
    println!("Nouvelle aire = {}", rect.aire());

    let carre = Rectangle::carre(10.0);
    println!("Carre : {:?}", carre);

    // en_carre() consomme rect2
    let rect2 = Rectangle::new(4.0, 9.0);
    let carre2 = rect2.en_carre();  // rect2 est move
    // println!("{:?}", rect2);  // ERREUR : rect2 a ete consume
    println!("{:?}", carre2);
}
```

### Les trois formes de self

```
+------------------+-------------------------------------------+------------------------+
|   Parametre      |   Signification                           |   Equivalent C         |
+------------------+-------------------------------------------+------------------------+
| &self            | Emprunte l'instance (lecture seule)        | const Rect *self       |
| &mut self        | Emprunte l'instance (lecture-ecriture)     | Rect *self             |
| self             | Prend possession (consomme l'instance)    | Rect self (par valeur) |
+------------------+-------------------------------------------+------------------------+
```

```
En C, la convention est de passer un pointeur en premier argument :

// C : convention manuelle, rien ne la force
void rect_afficher(const Rectangle *self) { ... }
void rect_agrandir(Rectangle *self, double facteur) { ... }

// Rust : le compilateur encode la semantique dans le type de self
impl Rectangle {
    fn afficher(&self) { ... }          // Lit seulement
    fn agrandir(&mut self, f: f64) { }  // Modifie
}
```

> [!example] Plusieurs blocs impl
> Vous pouvez avoir plusieurs blocs `impl` pour le meme type. C'est utile pour organiser le code :
> ```rust
> impl Rectangle {
>     fn aire(&self) -> f64 { self.largeur * self.hauteur }
>     fn perimetre(&self) -> f64 { 2.0 * (self.largeur + self.hauteur) }
> }
>
> impl Rectangle {
>     fn est_carre(&self) -> bool { self.largeur == self.hauteur }
>     fn contient(&self, autre: &Rectangle) -> bool {
>         self.largeur >= autre.largeur && self.hauteur >= autre.hauteur
>     }
> }
> ```

---

## Derive macros

Rust permet de generer automatiquement des implementations de traits courants avec `#[derive(...)]`.

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}

fn main() {
    let p1 = Point { x: 1.0, y: 2.0 };

    // Debug : permet {:?} et {:#?}
    println!("{:?}", p1);         // Point { x: 1.0, y: 2.0 }
    println!("{:#?}", p1);        // Affichage "pretty" sur plusieurs lignes

    // Clone : copie profonde explicite
    let p2 = p1.clone();

    // PartialEq : comparaison avec ==
    assert_eq!(p1, p2);

    let p3 = Point { x: 3.0, y: 4.0 };
    assert_ne!(p1, p3);
}
```

### Derives courants

```
+-------------------+----------------------------------------------+-------------------+
| Derive            | Ce que ca genere                             | Equiv C           |
+-------------------+----------------------------------------------+-------------------+
| Debug             | Affichage avec {:?} (pour le debug)          | (rien, printf %?) |
| Clone             | .clone() pour copie profonde                 | memcpy + deep     |
| Copy              | Copie implicite (stack only)                 | memcpy            |
| PartialEq         | == et !=                                     | (fonction custom)  |
| Eq                | Egalite totale (complete PartialEq)          | (fonction custom)  |
| PartialOrd        | <, >, <=, >=                                 | (fonction custom)  |
| Ord               | Ordre total (pour le tri)                    | qsort comparator  |
| Hash              | Hashable (pour HashMap/HashSet)              | hash function      |
| Default           | Valeur par defaut                            | memset(0) ?        |
+-------------------+----------------------------------------------+-------------------+
```

> [!info] Pourquoi derive et pas automatique ?
> En C, `memcpy` copie n'importe quelle struct bit a bit, meme si c'est dangereux (copie superficielle de pointeurs). En Rust, le compilateur refuse de copier une struct qui contient des `String` ou des `Vec` sans `Clone` explicite, car copier les metadonnees stack sans dupliquer les donnees heap serait un double free.

---

## Enums

### Enums basiques

```rust
// En C :
// enum Direction { NORD, SUD, EST, OUEST };
// -> Juste des entiers deguises (0, 1, 2, 3)

// En Rust :
enum Direction {
    Nord,
    Sud,
    Est,
    Ouest,
}

fn main() {
    let dir = Direction::Nord;

    match dir {
        Direction::Nord => println!("On va vers le nord"),
        Direction::Sud => println!("On va vers le sud"),
        Direction::Est => println!("On va vers l'est"),
        Direction::Ouest => println!("On va vers l'ouest"),
    }
}
```

### Enums avec donnees : la vraie puissance

Contrairement au C, les variantes d'un enum Rust peuvent **contenir des donnees**.

```rust
enum Message {
    Quitter,                         // Pas de donnees (comme un enum C)
    Deplacer { x: i32, y: i32 },     // Struct anonyme
    Ecrire(String),                  // Contient un String
    ChangerCouleur(u8, u8, u8),      // Contient un tuple
}

fn traiter_message(msg: Message) {
    match msg {
        Message::Quitter => {
            println!("Fin du programme");
        }
        Message::Deplacer { x, y } => {
            println!("Deplacement vers ({}, {})", x, y);
        }
        Message::Ecrire(texte) => {
            println!("Message : {}", texte);
        }
        Message::ChangerCouleur(r, g, b) => {
            println!("Couleur : ({}, {}, {})", r, g, b);
        }
    }
}

fn main() {
    let msg = Message::Ecrire(String::from("bonjour"));
    traiter_message(msg);

    let msg2 = Message::Deplacer { x: 10, y: 20 };
    traiter_message(msg2);
}
```

### Comparaison avec les tagged unions en C

```c
// En C, pour faire la meme chose, il faut une union taguee manuelle :
typedef enum { QUITTER, DEPLACER, ECRIRE, CHANGER_COULEUR } MessageType;

typedef struct {
    MessageType type;
    union {
        struct { int x; int y; } deplacer;
        char *ecrire;  // Qui possede cette memoire ?
        struct { unsigned char r, g, b; } couleur;
    } data;
} Message;

void traiter_message(Message *msg) {
    switch (msg->type) {
        case QUITTER:
            printf("Fin\n");
            break;
        case DEPLACER:
            printf("(%d, %d)\n", msg->data.deplacer.x, msg->data.deplacer.y);
            break;
        case ECRIRE:
            printf("%s\n", msg->data.ecrire);
            break;
        case CHANGER_COULEUR:
            printf("(%d, %d, %d)\n", msg->data.couleur.r,
                   msg->data.couleur.g, msg->data.couleur.b);
            break;
        // Si on oublie un cas ? Le compilateur C ne dit RIEN.
        // Si on accede au mauvais champ de l'union ? Comportement indefini.
    }
}
```

```
Comparaison structurelle :

C (union taguee manuelle) :           Rust (enum natif) :

struct Message {                       enum Message {
    enum Type type;    // tag             Quitter,
    union {            // donnees         Deplacer { x: i32, y: i32 },
        struct { ... };                   Ecrire(String),
        char *s;                          ChangerCouleur(u8, u8, u8),
        struct { ... };                }
    };
};
                                       Le tag est AUTOMATIQUE et INVISIBLE
Le tag est MANUEL                      Le compilateur VERIFIE l'exhaustivite
Rien ne verifie la coherence           Impossible d'acceder au mauvais variant
```

> [!warning] Securite des enums
> En C, rien ne vous empeche d'acceder a `msg->data.deplacer` quand `msg->type == ECRIRE`. C'est un comportement indefini. En Rust, le pattern matching **oblige** a deconstruire la bonne variante. Il est litteralement impossible d'acceder aux donnees de `Deplacer` quand vous avez un `Ecrire`.

---

## Option\<T\> : la fin du NULL

### Le probleme de NULL en C

```c
// En C : NULL est un piege partout
#include <stdio.h>
#include <stdlib.h>

// Cette fonction peut retourner NULL. Le compilateur ne vous previent pas.
char *trouver_utilisateur(int id) {
    if (id == 42) {
        return "Alice";
    }
    return NULL;  // Pas trouve
}

int main(void) {
    char *user = trouver_utilisateur(99);
    printf("Longueur : %lu\n", strlen(user));  // CRASH : NULL dereference
    // Le compilateur n'a rien dit. Le programme crashe a l'execution.
    return 0;
}
```

Tony Hoare, l'inventeur de NULL, l'a appele sa **"billion-dollar mistake"** (erreur a un milliard de dollars).

### La solution Rust : Option\<T\>

```rust
// Option est defini dans la bibliotheque standard :
// enum Option<T> {
//     Some(T),    // Il y a une valeur
//     None,       // Il n'y a pas de valeur
// }

fn trouver_utilisateur(id: u32) -> Option<String> {
    if id == 42 {
        Some(String::from("Alice"))
    } else {
        None  // Explicite : pas de valeur
    }
}

fn main() {
    let user = trouver_utilisateur(99);

    // IMPOSSIBLE d'utiliser user directement comme un String
    // println!("{}", user);  // ERREUR : Option<String> n'implemente pas Display

    // Il FAUT gerer le cas None
    match user {
        Some(nom) => println!("Trouve : {}", nom),
        None => println!("Utilisateur non trouve"),
    }
}
```

```
Comparaison conceptuelle :

C :    char *result = find(...);
       // result est PEUT-ETRE NULL
       // Le compilateur ne vous oblige pas a verifier
       // -> NULL dereference = crash

Rust : let result: Option<String> = find(...);
       // result est EXPLICITEMENT "peut-etre vide"
       // Le compilateur REFUSE de vous laisser utiliser
       // la valeur sans gerer le cas None
       // -> NULL dereference = IMPOSSIBLE
```

### Methodes utiles sur Option

```rust
fn main() {
    let nombre: Option<i32> = Some(42);
    let vide: Option<i32> = None;

    // unwrap : extrait la valeur ou PANIQUE si None
    let n = nombre.unwrap();  // 42
    // let v = vide.unwrap();  // PANIQUE ! N'utilisez qu'en debug/prototypage

    // unwrap_or : fournit une valeur par defaut
    let n = vide.unwrap_or(0);  // 0

    // unwrap_or_else : calcule la valeur par defaut avec une closure
    let n = vide.unwrap_or_else(|| calcul_couteux());

    // is_some / is_none
    if nombre.is_some() {
        println!("Il y a une valeur");
    }

    // map : transforme la valeur si elle existe
    let double: Option<i32> = nombre.map(|n| n * 2);  // Some(84)
    let double: Option<i32> = vide.map(|n| n * 2);    // None

    // and_then : chaine les operations qui retournent Option (flatmap)
    let resultat = nombre
        .map(|n| n + 10)      // Some(52)
        .filter(|&n| n > 50)  // Some(52) car 52 > 50
        .map(|n| n * 2);      // Some(104)

    // if let : extraction concise
    if let Some(n) = nombre {
        println!("Valeur : {}", n);
    }
}
```

> [!tip] Analogie
> `Option<T>` est comme un colis : `Some(valeur)` c'est un colis avec quelque chose dedans, `None` c'est un colis vide. En C, on recoit juste un pointeur et on espere qu'il pointe vers quelque chose. En Rust, le type du colis vous FORCE a verifier s'il est vide avant de l'ouvrir.

---

## Result\<T, E\> : la fin du "return -1"

### Le probleme de la gestion d'erreurs en C

```c
// En C : les erreurs sont des valeurs magiques, rien ne force a les gerer
#include <stdio.h>
#include <stdlib.h>

FILE *ouvrir_fichier(const char *chemin) {
    FILE *f = fopen(chemin, "r");
    // Retourne NULL en cas d'erreur. Pourquoi ? errno le sait peut-etre.
    return f;
}

int lire_entier(const char *s) {
    // atoi retourne 0 si l'input est invalide
    // MAIS retourne aussi 0 si l'input est "0"
    // Comment distinguer une erreur d'un zero valide ? On ne peut pas.
    return atoi(s);
}

int main(void) {
    FILE *f = ouvrir_fichier("data.txt");
    // Si on oublie de verifier f == NULL : CRASH a la prochaine utilisation
    char buf[100];
    fgets(buf, 100, f);  // Si f est NULL : comportement indefini
    return 0;
}
```

### La solution Rust : Result\<T, E\>

```rust
// Result est defini dans la bibliotheque standard :
// enum Result<T, E> {
//     Ok(T),      // Succes : contient la valeur
//     Err(E),     // Erreur : contient l'erreur
// }

use std::fs;
use std::num::ParseIntError;

fn lire_fichier(chemin: &str) -> Result<String, std::io::Error> {
    fs::read_to_string(chemin)  // Retourne Result<String, io::Error>
}

fn parser_entier(s: &str) -> Result<i32, ParseIntError> {
    s.parse::<i32>()  // Retourne Result<i32, ParseIntError>
}

fn main() {
    // Il FAUT gerer les deux cas
    match lire_fichier("data.txt") {
        Ok(contenu) => println!("Contenu : {}", contenu),
        Err(erreur) => println!("Erreur : {}", erreur),
    }

    match parser_entier("42") {
        Ok(n) => println!("Nombre : {}", n),
        Err(e) => println!("Erreur de parsing : {}", e),
    }

    match parser_entier("pas_un_nombre") {
        Ok(n) => println!("Nombre : {}", n),
        Err(e) => println!("Erreur : {}", e),  // Affiche l'erreur
    }
}
```

```
Comparaison gestion d'erreurs :

C :                                    Rust :

int fd = open("f.txt", O_RDONLY);      let f = File::open("f.txt");
if (fd < 0) {                         match f {
    perror("open");                        Ok(file) => { /* utiliser file */ }
    return -1;                             Err(e) => { /* gerer l'erreur */ }
}                                      }
// Si on oublie le if ?               // Si on oublie le match ?
// Le compilateur ne dit RIEN          // Le compilateur REFUSE de compiler
                                       // (Result doit etre utilise)
```

> [!info] #[must_use]
> `Result` est marque `#[must_use]` en Rust. Si vous ignorez un `Result` sans le traiter, le compilateur emet un **warning**. En C, ignorer la valeur de retour de `fopen()` ou `malloc()` ne genere aucun avertissement par defaut.

### Methodes utiles sur Result

```rust
fn main() {
    let ok: Result<i32, String> = Ok(42);
    let err: Result<i32, String> = Err(String::from("echec"));

    // unwrap : extrait ou panique
    let n = ok.unwrap();  // 42
    // let n = err.unwrap();  // PANIQUE avec le message d'erreur

    // expect : comme unwrap mais avec un message personnalise
    let n = ok.expect("devrait avoir une valeur");  // 42

    // unwrap_or / unwrap_or_else
    let n = err.unwrap_or(0);  // 0
    let n = err.unwrap_or_else(|e| {
        println!("Erreur ignoree : {}", e);
        0
    });

    // map / map_err : transformer le succes ou l'erreur
    let double = ok.map(|n| n * 2);       // Ok(84)
    let msg = err.map_err(|e| format!("FATAL: {}", e));

    // is_ok / is_err
    if ok.is_ok() {
        println!("Succes !");
    }

    // and_then : chainage (retourne Result)
    let resultat = ok
        .map(|n| n + 10)              // Ok(52)
        .and_then(|n| if n > 50 {
            Ok(n)
        } else {
            Err(String::from("trop petit"))
        });                            // Ok(52)
}
```

### L'operateur ? : propagation d'erreurs

L'operateur `?` est un raccourci pour propager les erreurs vers l'appelant.

```rust
use std::fs;
use std::io;

// Sans ? : verbeux
fn lire_nom_v1(chemin: &str) -> Result<String, io::Error> {
    let contenu = match fs::read_to_string(chemin) {
        Ok(c) => c,
        Err(e) => return Err(e),  // Propager l'erreur
    };
    Ok(contenu.trim().to_string())
}

// Avec ? : concis et elegant
fn lire_nom_v2(chemin: &str) -> Result<String, io::Error> {
    let contenu = fs::read_to_string(chemin)?;  // ? = si Err, return Err
    Ok(contenu.trim().to_string())
}

// Chainage avec ?
fn lire_et_parser(chemin: &str) -> Result<i32, Box<dyn std::error::Error>> {
    let contenu = fs::read_to_string(chemin)?;
    let nombre = contenu.trim().parse::<i32>()?;
    Ok(nombre)
}
```

> [!warning] L'operateur ? ne peut etre utilise que dans les fonctions qui retournent Result (ou Option). Il ne fonctionne PAS dans `fn main()` par defaut, sauf si vous declarez `fn main() -> Result<(), Error>`.

---

## Pattern Matching

### Match : expressions exhaustives

```rust
enum Piece {
    Roi,
    Reine,
    Tour,
    Fou,
    Cavalier,
    Pion,
}

fn valeur(piece: &Piece) -> u32 {
    match piece {
        Piece::Roi => 0,        // Le roi n'a pas de valeur (infinie)
        Piece::Reine => 9,
        Piece::Tour => 5,
        Piece::Fou => 3,
        Piece::Cavalier => 3,
        Piece::Pion => 1,
    }
    // Si on oublie un variant : ERREUR DE COMPILATION
    // "non-exhaustive patterns: `Pion` not covered"
}
```

### Patterns avec binding

```rust
fn decrire_nombre(n: i32) -> String {
    match n {
        0 => String::from("zero"),
        1 => String::from("un"),
        2..=9 => format!("petit nombre : {}", n),    // Range pattern
        10 | 20 | 30 => format!("dizaine ronde : {}", n),  // Or pattern
        x if x < 0 => format!("negatif : {}", x),    // Guard
        x => format!("nombre : {}", x),               // Catch-all avec binding
    }
}
```

### Patterns imbriques (nested patterns)

```rust
enum Forme {
    Cercle(f64),
    Rectangle(f64, f64),
    Triangle { base: f64, hauteur: f64 },
}

fn aire(forme: &Forme) -> f64 {
    match forme {
        Forme::Cercle(rayon) => std::f64::consts::PI * rayon * rayon,
        Forme::Rectangle(l, h) => l * h,
        Forme::Triangle { base, hauteur } => 0.5 * base * hauteur,
    }
}

// Pattern matching sur des Option imbriquees
fn decrire(val: Option<Option<i32>>) {
    match val {
        Some(Some(n)) => println!("Valeur : {}", n),
        Some(None) => println!("Valeur presente mais vide"),
        None => println!("Rien du tout"),
    }
}
```

### Destructuration dans match et let

```rust
#[derive(Debug)]
struct Point {
    x: f64,
    y: f64,
}

fn main() {
    let point = Point { x: 3.0, y: 4.0 };

    // Destructuration dans let
    let Point { x, y } = point;
    println!("x = {}, y = {}", x, y);

    // Destructuration dans match
    let point2 = Point { x: 0.0, y: 5.0 };
    match point2 {
        Point { x: 0.0, y } => println!("Sur l'axe Y, y = {}", y),
        Point { x, y: 0.0 } => println!("Sur l'axe X, x = {}", x),
        Point { x, y } => println!("Point ({}, {})", x, y),
    }

    // Destructuration de tuples
    let (a, b, c) = (1, 2.0, "trois");
    println!("{} {} {}", a, b, c);

    // Ignorer des valeurs avec _
    let (premier, _, troisieme) = (10, 20, 30);
    println!("{} {}", premier, troisieme);

    // Ignorer le reste avec ..
    let (head, ..) = (1, 2, 3, 4, 5);
    println!("Premier : {}", head);
}
```

### Le wildcard _ et le catch-all

```rust
fn main() {
    let nombre = 42;

    // _ : ignore la valeur, ne cree pas de binding
    match nombre {
        1 => println!("un"),
        _ => println!("autre"),  // _ ne cree pas de variable
    }

    // variable : cree un binding (catch-all nomme)
    match nombre {
        1 => println!("un"),
        n => println!("autre : {}", n),  // n contient la valeur
    }

    // Attention : _ dans un pattern de destructuration
    let (_x, _) = (1, 2);  // _x est cree mais _ est ignore
}
```

---

## if let et while let

### if let : match sur un seul pattern

```rust
fn main() {
    let config_max: Option<u32> = Some(100);

    // Avec match (verbeux quand on ne veut qu'un cas)
    match config_max {
        Some(max) => println!("Max = {}", max),
        _ => {},
    }

    // Avec if let (concis)
    if let Some(max) = config_max {
        println!("Max = {}", max);
    }

    // if let avec else
    if let Some(max) = config_max {
        println!("Max = {}", max);
    } else {
        println!("Pas de max configure");
    }

    // Avec Result
    let resultat: Result<i32, String> = Ok(42);
    if let Ok(valeur) = resultat {
        println!("Valeur = {}", valeur);
    }
}
```

### while let : boucle tant qu'un pattern matche

```rust
fn main() {
    let mut pile: Vec<i32> = vec![1, 2, 3, 4, 5];

    // pop() retourne Option<i32>
    // while let continue tant que c'est Some
    while let Some(sommet) = pile.pop() {
        println!("Depile : {}", sommet);
    }
    // Affiche : 5, 4, 3, 2, 1
    // S'arrete quand pop() retourne None
}
```

> [!example] Combinaison courante
> ```rust
> // Traiter une file de messages
> fn traiter_messages(messages: &mut Vec<Message>) {
>     while let Some(msg) = messages.pop() {
>         if let Message::Ecrire(texte) = msg {
>             println!("Texte : {}", texte);
>         }
>     }
> }
> ```

---

## Implementer le trait Display

En C, pour afficher une struct avec `printf`, il faut ecrire une fonction d'affichage personnalisee. En Rust, on implemente le trait `Display`.

```rust
use std::fmt;

struct Point {
    x: f64,
    y: f64,
}

// Implementer Display permet d'utiliser {} dans println!
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

// Comparaison :
// C :    void point_print(const Point *p) { printf("(%f, %f)", p->x, p->y); }
//        point_print(&p);
// Rust : println!("{}", p);  // Utilise l'impl Display automatiquement

fn main() {
    let p = Point { x: 3.0, y: 4.0 };
    println!("Point : {}", p);          // Point : (3, 4)
    let s = format!("Le point est {}", p);  // Marche aussi avec format!
    println!("{}", s);
}
```

### Display pour les enums

```rust
use std::fmt;

enum Couleur {
    Rouge,
    Vert,
    Bleu,
    Custom(u8, u8, u8),
}

impl fmt::Display for Couleur {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            Couleur::Rouge => write!(f, "Rouge (#FF0000)"),
            Couleur::Vert => write!(f, "Vert (#00FF00)"),
            Couleur::Bleu => write!(f, "Bleu (#0000FF)"),
            Couleur::Custom(r, g, b) => write!(f, "Custom (#{:02X}{:02X}{:02X})", r, g, b),
        }
    }
}

fn main() {
    let c = Couleur::Custom(128, 64, 255);
    println!("{}", c);  // Custom (#8040FF)
}
```

> [!info] Debug vs Display
> - `Debug` (`{:?}`) : pour les developpeurs, derive automatiquement avec `#[derive(Debug)]`
> - `Display` (`{}`) : pour les utilisateurs, doit etre implemente manuellement
> - En C, il n'y a pas cette distinction. `printf` ne sait rien de vos types personnalises.

---

## Recapitulatif : enums comme types somme

```
Types en programmation :

TYPE PRODUIT (struct) :              TYPE SOMME (enum) :
"A ET B ET C"                        "A OU B OU C"

struct Personne {                    enum Resultat {
    nom: String,      // ET              Succes(i32),     // OU
    age: u32,         // ET              Echec(String),   // OU
    email: String,    // ET              EnCours,         // OU
}                                    }

En C :                               En C :
struct = type produit OK             enum + union = type somme FRAGILE
                                     Pas de verification par le compilateur

En Rust :
struct = type produit OK
enum = type somme SAFE (verifie par le compilateur)
```

```
Option<T> et Result<T,E> sont des enums :

    Option<T>                        Result<T, E>
    +--------+                       +--------+
    | Some(T)|                       | Ok(T)  |
    +--------+                       +--------+
    | None   |                       | Err(E) |
    +--------+                       +--------+

    Remplace NULL en C               Remplace return -1 / errno en C
    Force a gerer l'absence          Force a gerer l'erreur
```

---

## Carte Mentale ASCII

```
                    Structs, Enums et Pattern Matching
                                  |
            +---------------------+---------------------+
            |                     |                     |
         STRUCTS               ENUMS              PATTERN MATCHING
            |                     |                     |
    +-------+-------+     +------+------+       +------+------+
    |       |       |     |      |      |       |      |      |
  champs  methods  derive  data   Option  Result  match    if let
    |       |       |     variants  <T>    <T,E>  |      while let
    |   impl Self   |     |      |      |       |
    |   &self       +- Debug     Some   Ok      +- exhaustif
    |   &mut self   +- Clone     None   Err     +- guards
    |   self        +- PartialEq               +- bindings
    |               +- Copy                    +- nested
    +- field init                              +- destructure
    +- update (..)                             +- wildcards _
    +- tuple struct                            +- ranges
    +- unit struct                             +- | (or)

    vs C :
    - struct C : pas de methodes, free() manuel
    - enum C : juste des int, pas de donnees
    - switch C : pas exhaustif, fall-through
    - NULL C : pas de verification forcee
    - errno C : facile a ignorer
```

---

## Exercices

### Exercice 1 : Struct avec methodes

Creez une struct `Vecteur2D` avec `x: f64` et `y: f64`. Implementez :
- `new(x, y)` : constructeur
- `norme(&self)` : longueur du vecteur (sqrt(x^2 + y^2))
- `ajouter(&self, autre: &Vecteur2D)` : retourne un nouveau Vecteur2D
- `normaliser(&self)` : retourne un vecteur de norme 1
- `Display` : affichage sous forme `(x, y)`

```rust
struct Vecteur2D {
    x: f64,
    y: f64,
}

impl Vecteur2D {
    // A completer
}

impl std::fmt::Display for Vecteur2D {
    // A completer
}
```

### Exercice 2 : Enum et pattern matching

Creez un enum `Expression` qui represente des expressions arithmetiques simples :
- `Nombre(f64)`
- `Addition(Box<Expression>, Box<Expression>)`
- `Multiplication(Box<Expression>, Box<Expression>)`
- `Negation(Box<Expression>)`

Implementez une fonction `evaluer(expr: &Expression) -> f64` qui evalue l'expression recursivement.

```rust
enum Expression {
    // A completer (utiliser Box pour la recursion)
}

fn evaluer(expr: &Expression) -> f64 {
    // A completer avec match
    todo!()
}

fn main() {
    // (3 + 4) * 2 = 14
    let expr = Expression::Multiplication(
        Box::new(Expression::Addition(
            Box::new(Expression::Nombre(3.0)),
            Box::new(Expression::Nombre(4.0)),
        )),
        Box::new(Expression::Nombre(2.0)),
    );
    println!("{}", evaluer(&expr));  // 14.0
}
```

### Exercice 3 : Option et Result en pratique

Creez une fonction `diviser(a: f64, b: f64) -> Result<f64, String>` qui retourne une erreur si b est zero. Puis creez une fonction `inverse_safe(v: &[f64]) -> Vec<Result<f64, String>>` qui retourne l'inverse de chaque element.

```rust
fn diviser(a: f64, b: f64) -> Result<f64, String> {
    // A completer
    todo!()
}

fn inverse_safe(v: &[f64]) -> Vec<Result<f64, String>> {
    // Indice : .iter().map(|&x| diviser(1.0, x)).collect()
    todo!()
}

fn main() {
    let nombres = vec![2.0, 0.0, 4.0, 0.0, 5.0];
    let inverses = inverse_safe(&nombres);
    for (i, res) in inverses.iter().enumerate() {
        match res {
            Ok(val) => println!("1/{} = {:.2}", nombres[i], val),
            Err(e) => println!("1/{} = ERREUR: {}", nombres[i], e),
        }
    }
}
```

### Exercice 4 : Reecrivez une union taguee C

Traduisez cette union taguee C en Rust idiomatique avec un enum. Implementez `Display` pour chaque variante.

```c
// Code C a traduire en Rust :
typedef enum { ENTIER, FLOTTANT, CHAINE, BOOLEEN } TypeValeur;

typedef struct {
    TypeValeur type;
    union {
        int entier;
        double flottant;
        char *chaine;
        int booleen;
    } data;
} Valeur;

void afficher_valeur(const Valeur *v) {
    switch (v->type) {
        case ENTIER:   printf("%d", v->data.entier); break;
        case FLOTTANT: printf("%f", v->data.flottant); break;
        case CHAINE:   printf("%s", v->data.chaine); break;
        case BOOLEEN:  printf("%s", v->data.booleen ? "true" : "false"); break;
    }
}
```

```rust
// A completer : definir l'enum Valeur et implementer Display
enum Valeur {
    // ...
}
```

---

## Liens

- [[03 - Structures et Typedef]] --- Les structs et typedef en C (comparaison directe)
- [[01 - Introduction a Rust]] --- Les bases du langage
- [[02 - Ownership et Borrowing]] --- Ownership, references et lifetimes
