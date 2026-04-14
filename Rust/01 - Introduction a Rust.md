# Introduction a Rust

Rust est un langage de programmation systeme qui garantit la **securite memoire** sans recourir a un ramasse-miettes (garbage collector). Concu pour offrir des performances equivalentes au C et au C++, il elimine a la compilation des categories entieres de bugs qui hantent les developpeurs systeme depuis des decennies : dangling pointers, double free, data races, buffer overflows.

Si vous connaissez le C, vous allez retrouver la meme philosophie de controle bas-niveau, mais avec un filet de securite tisse dans le compilateur lui-meme. Rust ne vous empeche pas de faire des choses dangereuses --- il vous oblige simplement a les rendre **explicites**.

---

## Pourquoi Rust ?

### Les trois piliers

1. **Securite memoire sans garbage collector** --- Le systeme d'ownership (possession) verifie a la compilation que chaque zone memoire a exactement un proprietaire. Pas de `free()` oublie, pas de double liberation.

2. **Abstractions a cout zero** (zero-cost abstractions) --- Les iterateurs, les generiques, les traits sont compiles vers du code machine aussi efficace que du C ecrit a la main. Aucun cout a l'execution.

3. **Concurrence sans peur** (fearless concurrency) --- Le compilateur interdit les data races. Si votre code compile, il est garanti exempt de courses de donnees.

> [!tip] Analogie
> Imaginez le C comme conduire une voiture de course sans ceinture de securite : puissant, rapide, mais une erreur et c'est le crash. Rust, c'est la meme voiture de course, mais avec un systeme de securite integre qui **refuse de demarrer** si vous n'avez pas boucle votre ceinture. Le cout ? Quelques secondes au demarrage (la compilation). Le gain ? Vous ne mourrez pas sur la piste.

### Comparaison rapide : C vs Rust

```
+---------------------------+-----------------------------+-------------------------------+
|         Aspect            |            C                |            Rust               |
+---------------------------+-----------------------------+-------------------------------+
| Gestion memoire           | malloc/free manuels         | Ownership automatique         |
| Securite memoire          | Aucune garantie             | Garantie a la compilation     |
| Null                      | NULL pointer partout        | Option<T> (pas de null)       |
| Gestion d'erreurs         | return -1, errno            | Result<T,E> explicite         |
| Strings                   | char* (piege permanent)     | String / &str (safe)          |
| Concurrence               | pthread + prieres           | Garantie sans data race       |
| Compilation               | cc / gcc / clang            | rustc via cargo               |
| Gestionnaire de paquets   | Aucun standard              | cargo (integre)               |
| Temps de compilation      | Rapide                      | Plus lent (analyse poussee)   |
| Performance execution     | Excellente                  | Equivalente                   |
+---------------------------+-----------------------------+-------------------------------+
```

> [!info] Performance
> Rust compile vers du code natif via LLVM, le meme backend que Clang. Les benchmarks montrent des performances quasi identiques au C, parfois meilleures grace aux optimisations permises par les garanties supplementaires du compilateur.

---

## Histoire de Rust

Rust est ne en **2006** comme projet personnel de **Graydon Hoare**, employe chez Mozilla. Le nom vient d'une famille de champignons (les rouilles) connus pour leur resilience.

```
Chronologie :
                                                                      
2006 ---- Projet personnel de Graydon Hoare
  |
2009 ---- Mozilla sponsorise officiellement le projet
  |
2012 ---- Premier compilateur auto-heberge (rustc compile rustc)
  |
2015 ---- Rust 1.0 : premiere version stable
  |          Promesse de retrocompatibilite
  |
2018 ---- Edition 2018 : async/await, NLL (Non-Lexical Lifetimes)
  |
2020 ---- Rust Foundation creee (AWS, Google, Huawei, Microsoft, Mozilla)
  |
2021 ---- Edition 2021 : closures ameliorees, format! dans panic!
  |
2024 ---- Edition 2024 : nouvelles ameliorations du langage
  |
2025 ---- Adoption massive : Linux kernel, Android, Windows, Chromium
```

> [!info] Le saviez-vous ?
> Rust a ete elu "langage le plus aime" sur le sondage Stack Overflow pendant **8 annees consecutives** (2016-2023). Il est desormais utilise dans le noyau Linux depuis la version 6.1 (decembre 2022).

---

## Installation de Rust

### rustup : le gestionnaire de toolchain

En C, vous installez `gcc` ou `clang` et vous vous debrouillez. En Rust, un seul outil gere tout : **rustup**.

```bash
# Installation sur Linux/macOS/WSL
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Sur Windows, telecharger rustup-init.exe depuis https://rustup.rs

# Verifier l'installation
rustc --version       # Le compilateur
cargo --version       # Le gestionnaire de projet / build
rustup --version      # Le gestionnaire de toolchain
```

### Les trois outils principaux

```
+-------------------+----------------------------------+---------------------------+
|      Outil        |          Role                    |    Equivalent C           |
+-------------------+----------------------------------+---------------------------+
| rustc             | Compilateur                      | gcc / clang               |
| cargo             | Build + deps + test + doc        | make + pkg-config + ...   |
| rustup            | Gere les versions de Rust        | (rien d'equivalent)       |
+-------------------+----------------------------------+---------------------------+
```

> [!tip] Conseil pratique
> En pratique, vous n'appellerez presque jamais `rustc` directement. `cargo` fait tout pour vous. C'est comme si `make` connaissait automatiquement vos dependances, savait compiler, tester, documenter et publier votre code.

### Mettre a jour Rust

```bash
rustup update          # Met a jour toutes les toolchains
rustup default stable  # Utiliser la version stable
rustup default nightly # Utiliser la version nightly (experimental)
```

---

## Cargo : le couteau suisse

### Commandes essentielles

```bash
# Creer un nouveau projet
cargo new mon_projet          # Cree un projet binaire (executable)
cargo new ma_lib --lib        # Cree un projet bibliotheque

# Compiler
cargo build                   # Compile en mode debug (rapide, pas optimise)
cargo build --release         # Compile en mode release (optimise, lent)

# Executer
cargo run                     # Compile ET execute
cargo run --release           # Compile en release ET execute

# Verifier sans produire de binaire (plus rapide)
cargo check                   # Verifie que le code compile

# Tests
cargo test                    # Lance tous les tests

# Documentation
cargo doc                     # Genere la documentation HTML
cargo doc --open              # Genere et ouvre dans le navigateur

# Ajouter une dependance
cargo add serde               # Ajoute serde a Cargo.toml
cargo add serde --features derive
```

> [!warning] cargo check vs cargo build
> Pendant le developpement, utilisez `cargo check` plutot que `cargo build`. Il verifie la validite du code sans produire de binaire, ce qui est **beaucoup plus rapide**. Reservez `cargo build` pour quand vous avez besoin de l'executable.

### Comparaison avec le workflow C

```
En C :                              En Rust :
                                    
1. Ecrire le code                   1. Ecrire le code
2. Ecrire un Makefile               2. (cargo s'en charge)
3. Installer les deps manuellement  3. cargo add <dep>
4. make                             4. cargo build
5. ./mon_programme                  5. cargo run
6. Ecrire des tests separement      6. cargo test (tests integres)
7. Generer la doc ? Doxygen ?       7. cargo doc
```

---

## Structure d'un projet Rust

```bash
cargo new mon_projet
```

Produit :

```
mon_projet/
+-- Cargo.toml          # Manifeste du projet (equivalent package.json / Makefile)
+-- Cargo.lock          # Versions exactes des dependances (auto-genere)
+-- src/
|   +-- main.rs         # Point d'entree pour un binaire
+-- target/             # Dossier de build (auto-genere, dans .gitignore)
    +-- debug/          # Binaires de debug
    +-- release/        # Binaires optimises
```

Pour une bibliotheque :

```
ma_lib/
+-- Cargo.toml
+-- src/
|   +-- lib.rs          # Point d'entree pour une bibliotheque
+-- tests/              # Tests d'integration
+-- examples/           # Exemples d'utilisation
+-- benches/            # Benchmarks
```

### Cargo.toml

```toml
[package]
name = "mon_projet"
version = "0.1.0"
edition = "2021"           # Edition du langage Rust

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }

[dev-dependencies]
criterion = "0.5"          # Dependances pour les tests uniquement
```

> [!info] Editions
> Les editions Rust (2015, 2018, 2021, 2024) permettent d'introduire des changements incompatibles de maniere controlee. Chaque projet declare son edition dans `Cargo.toml`. Les crates d'editions differentes peuvent coexister dans le meme projet.

---

## Hello World

### En C

```c
// hello.c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

```bash
gcc hello.c -o hello && ./hello
```

### En Rust

```rust
// src/main.rs
fn main() {
    println!("Hello, World!");
}
```

```bash
cargo run
```

### Differences notables

```
C :    printf("Hello, %s!\n", name);    // fonction de la libc
Rust : println!("Hello, {}!", name);    // macro du langage (note le !)

C :    int main(void) { ... return 0; } // return explicite
Rust : fn main() { ... }               // pas de return necessaire

C :    #include <stdio.h>               // inclusion manuelle
Rust : println! est disponible partout  // prelude automatique
```

> [!warning] Le point d'exclamation !
> `println!` n'est pas une fonction, c'est une **macro**. Le `!` l'indique. Les macros en Rust sont beaucoup plus puissantes et sures que les macros du preprocesseur C (`#define`). Elles sont verifiees a la compilation et hygieniques (pas de collision de noms).

---

## Variables

### Declaration

En C, toute variable est mutable par defaut. En Rust, c'est l'inverse : tout est **immutable** par defaut.

```rust
fn main() {
    // Immutable par defaut
    let x = 5;
    // x = 6;  // ERREUR : cannot assign twice to immutable variable

    // Mutable explicitement
    let mut y = 10;
    y = 20;  // OK

    // Constante : evaluee a la compilation, type obligatoire
    const MAX_POINTS: u32 = 100_000;
}
```

### Comparaison avec C

```
C :                                 Rust :

int x = 5;          (mutable)      let x = 5;          (immutable)
const int x = 5;    (immutable)    let mut x = 5;      (mutable)
#define MAX 100      (macro)       const MAX: u32 = 100; (constante typee)
```

> [!tip] Analogie
> En C, la mutabilite est la regle et `const` est l'exception. En Rust, l'immutabilite est la regle et `mut` est l'exception. C'est comme un coffre-fort : en C, il est ouvert par defaut et vous devez penser a le fermer. En Rust, il est ferme par defaut et vous devez explicitement l'ouvrir.

### Constantes vs variables immutables

```rust
// Variable immutable : calculee a l'execution
let x = compute_something();

// Constante : DOIT etre connue a la compilation, type explicite obligatoire
const PI: f64 = 3.14159265358979;
const MAX_SIZE: usize = 1024 * 1024;  // Expressions constantes OK

// Les constantes n'ont pas d'adresse fixe en memoire :
// le compilateur les "inline" partout ou elles sont utilisees
```

---

## Shadowing

Le shadowing est un concept **unique a Rust** qui n'existe pas en C. Il permet de redeclarer une variable avec le meme nom, meme en changeant son type.

```rust
fn main() {
    let x = 5;
    let x = x + 1;      // Nouvelle variable x, l'ancienne est "masquee"
    let x = x * 2;      // Encore une nouvelle variable x
    println!("{}", x);   // Affiche 12

    // On peut meme changer le type !
    let espaces = "   ";          // &str
    let espaces = espaces.len();  // usize  (meme nom, type different)
    println!("{}", espaces);      // Affiche 3
}
```

> [!warning] Shadowing vs mutabilite
> Le shadowing cree une **nouvelle variable**. `mut` modifie la **meme variable**.
> ```rust
> // Shadowing : OK pour changer de type
> let x = "hello";
> let x = x.len();  // OK : nouvelle variable de type usize
>
> // mut : IMPOSSIBLE de changer de type
> let mut x = "hello";
> // x = x.len();   // ERREUR : expected &str, found usize
> ```

### Pourquoi le shadowing est utile

```rust
fn main() {
    // En C, on ferait :
    // const char *input_str = get_input();
    // int input_num = atoi(input_str);
    // Deux noms differents pour la meme donnee conceptuelle

    // En Rust, le shadowing permet :
    let input = "42";            // &str
    let input = input.trim();    // &str (nettoyee)
    let input: i32 = input.parse().unwrap();  // i32
    // Un seul nom logique pour la meme donnee a differents stades
}
```

---

## Types de donnees

### Types scalaires

```
+---------------+--------+----------+------------------------------------------+
| Type Rust     | Taille | Equiv C  | Plage                                    |
+---------------+--------+----------+------------------------------------------+
| i8            | 1 oct  | int8_t   | -128 a 127                               |
| i16           | 2 oct  | int16_t  | -32_768 a 32_767                         |
| i32           | 4 oct  | int32_t  | -2_147_483_648 a 2_147_483_647           |
| i64           | 8 oct  | int64_t  | -9.2 * 10^18 a 9.2 * 10^18              |
| i128          | 16 oct | (aucun)  | Tres grand                               |
| isize         | ptr    | intptr_t | Depend de l'architecture (32 ou 64 bits) |
+---------------+--------+----------+------------------------------------------+
| u8            | 1 oct  | uint8_t  | 0 a 255                                  |
| u16           | 2 oct  | uint16_t | 0 a 65_535                               |
| u32           | 4 oct  | uint32_t | 0 a 4_294_967_295                        |
| u64           | 8 oct  | uint64_t | 0 a 18.4 * 10^18                         |
| u128          | 16 oct | (aucun)  | Tres grand                               |
| usize         | ptr    | size_t   | Depend de l'architecture                 |
+---------------+--------+----------+------------------------------------------+
| f32           | 4 oct  | float    | ~6-7 chiffres significatifs              |
| f64           | 8 oct  | double   | ~15-16 chiffres significatifs            |
+---------------+--------+----------+------------------------------------------+
| bool          | 1 oct  | _Bool    | true / false                             |
| char          | 4 oct  | (aucun)  | Un point de code Unicode (U+0000-10FFFF) |
+---------------+--------+----------+------------------------------------------+
```

> [!warning] char en Rust vs C
> En C, `char` fait **1 octet** et represente un caractere ASCII. En Rust, `char` fait **4 octets** et represente un point de code Unicode complet. Cela signifie qu'un `char` Rust peut contenir des emojis, des caracteres chinois, des symboles mathematiques, etc.
> ```rust
> let c: char = 'z';
> let coeur: char = '❤';
> let kanji: char = '漢';
> println!("Taille de char : {} octets", std::mem::size_of::<char>()); // 4
> ```

### Litteraux numeriques

```rust
fn main() {
    let decimal = 98_222;       // Underscores comme separateurs visuels
    let hex = 0xff;             // Hexadecimal (comme en C)
    let octal = 0o77;           // Octal (C utilise 077, Rust utilise 0o77)
    let binaire = 0b1111_0000;  // Binaire (pas d'equivalent standard en C)
    let octet: u8 = b'A';      // Byte literal (equivalent au char C)
}
```

### Tuples

Les tuples n'existent pas en C. Ils regroupent des valeurs de types differents.

```rust
fn main() {
    // Declaration
    let tup: (i32, f64, char) = (500, 6.4, 'x');

    // Destructuration
    let (x, y, z) = tup;
    println!("y = {}", y);  // 6.4

    // Acces par index
    let cinq_cents = tup.0;
    let six_quatre = tup.1;

    // Tuple vide = "unit type" (equivalent du void en C)
    let vide: () = ();
}
```

### Tableaux (arrays)

```rust
fn main() {
    // Tableau a taille fixe (sur la stack, comme en C)
    let a: [i32; 5] = [1, 2, 3, 4, 5];

    // Initialisation uniforme
    let zeros = [0; 10];  // [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

    // Acces
    let premier = a[0];
    let dernier = a[4];

    // DIFFERENCE CRUCIALE avec C :
    // let boom = a[10];  // PANIQUE a l'execution (bounds check)
    // En C : a[10] = comportement indefini silencieux !
}
```

> [!example] Buffer overflow : C vs Rust
> ```c
> // En C : DANGER - aucune verification
> int a[5] = {1, 2, 3, 4, 5};
> int x = a[10];  // Comportement indefini ! Lit n'importe quoi en memoire
> a[10] = 42;     // Ecrase de la memoire aleatoire -> faille de securite
> ```
> ```rust
> // En Rust : securise - verification a l'execution
> let a = [1, 2, 3, 4, 5];
> let x = a[10];  // PANIQUE : "index out of bounds"
>                  // Le programme s'arrete proprement, pas de corruption
> ```

---

## Inference de type et casting

### Inference

Rust deduit les types quand c'est possible, comme `auto` en C++ (mais C n'a pas d'equivalent) :

```rust
fn main() {
    let x = 42;        // i32 par defaut
    let y = 3.14;      // f64 par defaut
    let z = true;      // bool
    let s = "hello";   // &str

    // Parfois il faut aider le compilateur
    let parsed: i32 = "42".parse().unwrap();
    // ou
    let parsed = "42".parse::<i32>().unwrap();  // "turbofish" syntax
}
```

### Casting avec `as`

```rust
fn main() {
    let x: i32 = 42;
    let y: f64 = x as f64;       // i32 -> f64
    let z: u8 = 255;
    let w: i8 = z as i8;         // Attention : 255_u8 -> -1_i8 (overflow)

    let c: char = 'A';
    let code: u32 = c as u32;    // 65
    let byte: u8 = b'A';         // 65

    // En C : les casts sont implicites et dangereux
    // int x = 3.14;  // Compile sans warning ! Perte silencieuse.
    // En Rust : let x: i32 = 3.14;  // ERREUR : type mismatch
}
```

> [!warning] Pas de conversion implicite
> Rust n'effectue **jamais** de conversion de type implicite. En C, `int x = 3.14;` compile silencieusement avec perte de precision. En Rust, vous devez ecrire `let x: i32 = 3.14 as i32;` pour rendre la troncature explicite.

---

## Strings : la grande confusion

C'est probablement le sujet qui deroute le plus les developpeurs C. Rust a **deux types principaux** de chaines.

### En C : le chaos

```c
// En C, une "string" c'est juste un pointeur vers des octets termines par \0
char *s1 = "hello";           // Pointeur vers memoire en lecture seule
char s2[] = "hello";          // Tableau de 6 chars sur la stack
char *s3 = malloc(6);         // Pointeur vers le tas (heap)
strcpy(s3, "hello");
// Qui libere s3 ? Quand ? On ne sait pas.
// s1[0] = 'H';  // Comportement indefini !
// strlen() parcourt TOUTE la chaine a chaque appel (O(n))
```

### En Rust : deux types clairs

```
+---------------------+----------------------------+-----------------------------+
|                     |   String                   |   &str                      |
+---------------------+----------------------------+-----------------------------+
| Stockage            | Heap (tas)                 | Emprunt (reference)         |
| Mutable             | Oui (si mut)               | Non                         |
| Proprietaire        | Oui (possede les donnees)  | Non (vue sur des donnees)   |
| Taille connue ?     | A l'execution (dynamique)  | A l'execution (fat pointer) |
| Equivalent C        | char* + malloc + strlen    | const char* (mais safe)     |
| Utilisation typique | Construire/modifier        | Lire/passer en parametre    |
+---------------------+----------------------------+-----------------------------+
```

```rust
fn main() {
    // &str : reference vers une chaine (souvent un litteral)
    let s1: &str = "hello";  // Pointe vers la memoire du programme (statique)

    // String : chaine sur le heap, modifiable
    let mut s2: String = String::from("hello");
    s2.push_str(", world!");
    s2.push('!');

    // Conversion
    let s3: String = s1.to_string();    // &str -> String
    let s4: String = "hello".into();    // &str -> String (via Into trait)
    let s5: &str = &s3;                 // String -> &str (deref coercion)

    // En parametre de fonction, preferez &str pour accepter les deux :
    fn afficher(s: &str) {
        println!("{}", s);
    }
    afficher(&s2);   // String -> &str automatiquement
    afficher(s1);    // &str directement
}
```

### Modele memoire d'un String

```
Stack                          Heap
+--------+--------+--------+
| ptr    |--------|------->| h | e | l | l | o |
+--------+--------+--------+  (5 octets de donnees)
| len: 5 |
+--------+
| cap: 5 |        
+--------+

Comparaison avec C :
C :    char *s = "hello";  -> juste un pointeur, pas de len ni cap
       strlen(s) doit parcourir pour trouver le \0
Rust : String stocke ptr + len + cap
       .len() est O(1), pas besoin de \0
```

> [!tip] Regle d'or pour les fonctions
> Si votre fonction a juste besoin de **lire** une chaine, prenez `&str` en parametre. Elle acceptera aussi bien des `String` (par deref coercion) que des `&str`. Ne prenez `String` que si vous avez besoin de **prendre possession** de la chaine.

---

## Affichage et formatage

### println! et format!

```rust
fn main() {
    let nom = "Rust";
    let version = 2025;

    // Affichage basique
    println!("Hello, {}!", nom);             // Hello, Rust!
    println!("{} version {}", nom, version); // Rust version 2025

    // Arguments nommes
    println!("{lang} v{ver}", lang = nom, ver = version);

    // Formatage de nombres
    println!("{:>10}", 42);        //         42  (aligne a droite, largeur 10)
    println!("{:<10}", 42);        // 42          (aligne a gauche)
    println!("{:0>5}", 42);        // 00042       (padding avec des zeros)
    println!("{:.3}", 3.14159);    // 3.142       (3 decimales)

    // Bases
    println!("{:b}", 42);          // 101010      (binaire)
    println!("{:o}", 42);          // 52          (octal)
    println!("{:x}", 255);         // ff          (hexadecimal minuscule)
    println!("{:X}", 255);         // FF          (hexadecimal majuscule)
    println!("{:#x}", 255);        // 0xff        (avec prefixe)
    println!("{:#b}", 42);         // 0b101010    (avec prefixe)

    // Debug
    let tableau = [1, 2, 3];
    println!("{:?}", tableau);     // [1, 2, 3]        (format Debug)
    println!("{:#?}", tableau);    // [                 (format Debug "pretty")
                                   //     1,
                                   //     2,
                                   //     3,
                                   // ]
}
```

### Comparaison avec printf en C

```
C :                                  Rust :

printf("%d\n", 42);                  println!("{}", 42);
printf("%s\n", "hello");            println!("{}", "hello");
printf("%x\n", 255);                println!("{:x}", 255);
printf("%.2f\n", 3.14);            println!("{:.2}", 3.14);
printf("%05d\n", 42);              println!("{:0>5}", 42);
sprintf(buf, "%d", 42);            let s = format!("{}", 42);
```

> [!info] Securite du formatage
> En C, `printf("%s", 42)` compile sans erreur et provoque un comportement indefini (le programme essaie de lire l'entier 42 comme une adresse de chaine). En Rust, `println!("{}", x)` verifie a la **compilation** que `x` implemente le trait `Display`. Pas de format string vulnerability possible.

---

## Fonctions

### Syntaxe de base

```rust
// En C :
// int addition(int a, int b) {
//     return a + b;
// }

// En Rust :
fn addition(a: i32, b: i32) -> i32 {
    a + b  // Pas de return, pas de ;  -> c'est une EXPRESSION
}

fn main() {
    let resultat = addition(3, 4);
    println!("{}", resultat);  // 7
}
```

### Expressions vs statements

C'est une difference fondamentale avec le C. En Rust, presque tout est une **expression** (retourne une valeur).

```rust
fn main() {
    // Un bloc {} est une expression
    let x = {
        let a = 5;
        let b = 10;
        a + b       // Pas de ; -> c'est la valeur retournee par le bloc
    };
    println!("{}", x);  // 15

    // if est une expression (comme l'operateur ternaire en C)
    let nombre = 7;
    let parite = if nombre % 2 == 0 { "pair" } else { "impair" };
    // En C : const char *parite = (nombre % 2 == 0) ? "pair" : "impair";

    // Attention au point-virgule !
    // a + b   -> expression, retourne la somme
    // a + b;  -> statement, retourne () (unit type, equivalent void)
}
```

> [!warning] Le point-virgule change tout
> ```rust
> fn bon() -> i32 {
>     42      // Expression : retourne 42. OK.
> }
>
> fn mauvais() -> i32 {
>     42;     // Statement : retourne (). ERREUR : expected i32, found ()
> }
> ```

### Types de retour et early return

```rust
// Pas de valeur de retour (retourne implicitement ())
fn saluer(nom: &str) {
    println!("Bonjour, {} !", nom);
}

// Return explicite pour sortir tot
fn valeur_absolue(x: i32) -> i32 {
    if x < 0 {
        return -x;  // Early return (comme en C)
    }
    x  // Retour implicite (derniere expression)
}

// Fonctions qui ne retournent jamais : type "never" (!)
fn boucle_infinie() -> ! {
    loop {
        // ne s'arrete jamais
    }
}
```

---

## Controle de flux

### if / else

```rust
fn main() {
    let nombre = 7;

    // Classique (pas de parentheses obligatoires autour de la condition !)
    if nombre > 5 {
        println!("Grand");
    } else if nombre > 0 {
        println!("Petit");
    } else {
        println!("Negatif ou zero");
    }

    // if comme expression (remplace le ternaire C)
    let description = if nombre > 0 { "positif" } else { "non-positif" };

    // ERREUR : pas de conversion implicite vers bool
    // if nombre { ... }  // ERREUR en Rust ! (OK en C)
    if nombre != 0 { /* ... */ }  // Il faut etre explicite
}
```

> [!info] Pas de "truthy/falsy"
> En C, `if (42)` est vrai, `if (0)` est faux, `if (ptr)` teste si le pointeur est non-null. En Rust, seul un `bool` peut etre utilise dans un `if`. C'est plus verbeux mais elimine une source de bugs.

### Boucles

```rust
fn main() {
    // loop : boucle infinie (remplace while(1) en C)
    let mut compteur = 0;
    let resultat = loop {
        compteur += 1;
        if compteur == 10 {
            break compteur * 2;  // loop peut retourner une valeur !
        }
    };
    println!("{}", resultat);  // 20

    // while
    let mut n = 3;
    while n > 0 {
        println!("{}...", n);
        n -= 1;
    }
    println!("Decollage !");

    // for : itere sur un iterateur (pas de for(i=0; i<n; i++) a la C)
    for i in 0..5 {           // 0, 1, 2, 3, 4  (range exclusive)
        println!("{}", i);
    }
    for i in 0..=5 {          // 0, 1, 2, 3, 4, 5  (range inclusive)
        println!("{}", i);
    }

    // for sur un tableau
    let arr = [10, 20, 30, 40, 50];
    for element in arr.iter() {
        println!("{}", element);
    }
    // Avec index
    for (i, val) in arr.iter().enumerate() {
        println!("[{}] = {}", i, val);
    }
}
```

### match : le switch surpuissant

`match` est l'une des fonctionnalites les plus puissantes de Rust. Pensez-y comme un `switch` en C, mais en beaucoup plus expressif.

```rust
fn main() {
    let nombre = 3;

    // Basique (comme switch en C, mais SANS fall-through !)
    match nombre {
        1 => println!("un"),
        2 => println!("deux"),
        3 => println!("trois"),
        4 | 5 => println!("quatre ou cinq"),  // Ou logique
        6..=10 => println!("entre six et dix"),  // Range
        _ => println!("autre chose"),  // Default (obligatoire si pas exhaustif)
    }

    // match comme expression
    let texte = match nombre {
        1 => "un",
        2 => "deux",
        _ => "autre",
    };

    // match avec garde (guard)
    let x = 4;
    match x {
        n if n < 0 => println!("negatif"),
        n if n % 2 == 0 => println!("{} est pair", n),
        _ => println!("impair positif"),
    }
}
```

> [!warning] Exhaustivite
> En C, un `switch` sans `default` compile sans probleme et ignore les cas non couverts. En Rust, `match` **doit** couvrir tous les cas possibles. Le compilateur refuse de compiler si un cas est oublie. C'est un filet de securite enorme.

### if let : match simplifie

```rust
fn main() {
    let valeur: Option<i32> = Some(42);

    // Au lieu de :
    match valeur {
        Some(v) => println!("Valeur : {}", v),
        None => {},  // On ne fait rien
    }

    // On peut ecrire :
    if let Some(v) = valeur {
        println!("Valeur : {}", v);
    }

    // Avec else
    if let Some(v) = valeur {
        println!("Valeur : {}", v);
    } else {
        println!("Pas de valeur");
    }
}
```

---

## Commentaires et documentation

```rust
// Commentaire simple (comme en C)

/* Commentaire
   multi-lignes (comme en C) */

/// Commentaire de documentation pour l'element suivant
/// Supporte le **Markdown** !
///
/// # Exemples
///
/// ```
/// let x = ma_fonction(42);
/// assert_eq!(x, 84);
/// ```
fn ma_fonction(x: i32) -> i32 {
    x * 2
}

//! Commentaire de documentation pour le module/crate courant
//! Place en haut de lib.rs ou main.rs
```

> [!tip] Tests dans la documentation
> Les blocs de code dans les commentaires `///` sont compiles et executes par `cargo test` ! C'est une garantie que la documentation reste a jour avec le code. En C, la documentation (Doxygen, man pages) diverge inevitablement du code.

```bash
cargo doc --open  # Genere et affiche la documentation HTML
```

---

## Tableau recapitulatif : C vs Rust

```
+---------------------------+----------------------------------+----------------------------------+
|       Concept             |          C                       |          Rust                    |
+---------------------------+----------------------------------+----------------------------------+
| Variable mutable          | int x = 5;                       | let mut x = 5;                   |
| Variable immutable        | const int x = 5;                 | let x = 5;                      |
| Constante compile-time    | #define MAX 100                  | const MAX: u32 = 100;            |
| Entier 32 bits            | int32_t x;                       | let x: i32;                     |
| Caractere                 | char c = 'A'; (1 octet)         | let c: char = 'A'; (4 octets)  |
| Chaine                    | char *s = "hello";               | let s: &str = "hello";          |
| Chaine mutable            | char s[10]; strcpy(s,"hi");      | let mut s = String::from("hi"); |
| Tableau                   | int a[5] = {1,2,3,4,5};         | let a: [i32; 5] = [1,2,3,4,5]; |
| Affichage                 | printf("%d\n", x);              | println!("{}", x);              |
| Fonction                  | int f(int a) { return a; }      | fn f(a: i32) -> i32 { a }      |
| Pas de valeur retour      | void f() {}                      | fn f() {}                       |
| Null                      | NULL                             | None (Option<T>)                |
| Boucle infinie            | while(1) {}                      | loop {}                         |
| For classique             | for(int i=0;i<n;i++)             | for i in 0..n                   |
| Switch                    | switch(x) { case 1: ... }       | match x { 1 => ... }           |
| Ternaire                  | x > 0 ? "oui" : "non"           | if x > 0 { "oui" } else {"non"}|
| Allocation heap           | malloc(n)                        | Box::new(v), Vec::new()        |
| Liberation memoire        | free(ptr)                        | Automatique (drop)             |
| Gestion erreur            | return -1; errno                 | Result<T, E>                   |
| Build                     | make / cmake                     | cargo build                    |
| Tests                     | (externe : CUnit, etc.)          | cargo test (integre)           |
| Documentation             | Doxygen / man pages              | cargo doc (integre)            |
+---------------------------+----------------------------------+----------------------------------+
```

---

## Carte Mentale ASCII

```
                            Introduction a Rust
                                    |
                +-------------------+-------------------+
                |                   |                   |
          Pourquoi Rust       Installation          Le Langage
          |                   |                     |
          +-- Securite mem.   +-- rustup            +-- Variables
          +-- Zero-cost abs.  +-- rustc             |   +-- let (immutable)
          +-- Concurrence     +-- cargo             |   +-- let mut
                              |                     |   +-- const
                              +-- new               |   +-- shadowing
                              +-- build / run       |
                              +-- check / test      +-- Types
                              +-- doc / add         |   +-- i8..i128 / u8..u128
                                                    |   +-- f32 / f64
                                                    |   +-- bool / char (4B!)
                                                    |   +-- tuples / arrays
                                                    |
                                                    +-- Strings
                                                    |   +-- String (heap, owned)
                                                    |   +-- &str (reference)
                                                    |
                                                    +-- Fonctions
                                                    |   +-- fn / -> Type
                                                    |   +-- Expressions vs statements
                                                    |
                                                    +-- Controle de flux
                                                        +-- if (expression!)
                                                        +-- loop / while / for
                                                        +-- match (exhaustif)
                                                        +-- if let
```

---

## Exercices

### Exercice 1 : Temperature

Ecrivez un programme qui convertit des temperatures Fahrenheit en Celsius et inversement. Utilisez des fonctions separees et le pattern matching pour choisir la direction de conversion.

```rust
// Squelette :
fn fahrenheit_vers_celsius(f: f64) -> f64 {
    // A completer
}

fn celsius_vers_fahrenheit(c: f64) -> f64 {
    // A completer
}

fn main() {
    let temp = 100.0;
    let direction = "F->C";  // ou "C->F"

    let resultat = match direction {
        // A completer
        _ => panic!("Direction inconnue"),
    };

    println!("{} -> {:.1}", temp, resultat);
}
```

### Exercice 2 : FizzBuzz en Rust

Implementez FizzBuzz de 1 a 100 en utilisant `match` au lieu de `if/else`. Comparez avec votre version C.

```rust
fn main() {
    for n in 1..=100 {
        let result = match (n % 3, n % 5) {
            // A completer : utilisez le tuple (modulo3, modulo5)
            _ => todo!(),
        };
        println!("{}", result);
    }
}
```

### Exercice 3 : Manipulation de Strings

Ecrivez une fonction qui prend un `&str`, inverse les mots (pas les caracteres), et retourne un `String`. Par exemple : `"hello world rust"` -> `"rust world hello"`.

```rust
fn inverser_mots(phrase: &str) -> String {
    // Indice : .split_whitespace(), .rev(), .collect::<Vec<&str>>(), .join(" ")
    todo!()
}

fn main() {
    let phrase = "hello world rust";
    let inversee = inverser_mots(phrase);
    println!("{}", inversee);  // "rust world hello"
}
```

### Exercice 4 : Tableau et statistiques

Creez un programme qui calcule la somme, la moyenne, le minimum et le maximum d'un tableau de `f64`. Comparez la concision avec une version C equivalente.

```rust
fn statistiques(data: &[f64]) -> (f64, f64, f64, f64) {
    // Retourne (somme, moyenne, min, max)
    // Indice : .iter(), .sum(), .min_by(), .max_by()
    todo!()
}
```

---

## Liens

- [[01 - Introduction au C et Compilation]] --- Comparer avec l'introduction au C
- [[02 - Ownership et Borrowing]] --- Le concept le plus important de Rust
- [[03 - Structs Enums et Pattern Matching]] --- Types personnalises et pattern matching avance
