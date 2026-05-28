# 10 - Macros et Métaprogrammation

> [!info] Prérequis
> Ce module demande une bonne maîtrise de [[06 - Traits et Generiques]] et de [[03 - Structs Enums et Pattern Matching]]. Les macros procédurales nécessitent aussi de connaître les concepts d'AST (Abstract Syntax Tree).

---

## 1. Vue d'ensemble des macros Rust

Rust offre deux familles de macros, fondamentalement différentes dans leur mécanisme.

| Type | Syntaxe | Moment d'exécution | Cas d'usage |
|------|---------|-------------------|-------------|
| **Déclaratives** (`macro_rules!`) | `mon_macro!(...)` | Compilation (expansion textuelle) | Patterns répétitifs simples, DSLs |
| **Procédurales `derive`** | `#[derive(MonTrait)]` | Compilation (AST → AST) | Implémenter des traits automatiquement |
| **Procédurales `attribute`** | `#[mon_attribut]` | Compilation (AST → AST) | Transformer des items (fonctions, structs) |
| **Procédurales `function-like`** | `mon_macro!(...)` | Compilation (tokens → AST) | Génération de code complexe avec accès à tout l'AST |

> [!tip] Macros vs Generics
> Les generics résolvent la réutilisation de code pour des types différents. Les macros résolvent la réutilisation de *syntaxe* — des patterns impossibles à exprimer avec les types seuls (nombre d'arguments variable, code conditionnel à la compilation, génération de code à partir de structs).

---

## 2. Macros Déclaratives — `macro_rules!`

### Syntaxe de base

```rust
// Déclaration — peut être utilisée après dans le même module ou sous-modules
macro_rules! dire_bonjour {
    // Règle sans argument
    () => {
        println!("Bonjour !");
    };
    // Règle avec un argument
    ($nom:expr) => {
        println!("Bonjour, {} !", $nom);
    };
    // Règle avec plusieurs arguments
    ($prenom:expr, $nom:expr) => {
        println!("Bonjour, {} {} !", $prenom, $nom);
    };
}

fn main() {
    dire_bonjour!();                   // → "Bonjour !"
    dire_bonjour!("Alice");            // → "Bonjour, Alice !"
    dire_bonjour!("Alice", "Dupont");  // → "Bonjour, Alice Dupont !"
}
```

### Fragments — les types de captures

| Fragment | Description | Exemple |
|----------|-------------|---------|
| `expr` | Expression | `1 + 2`, `"hello"`, `ma_fonction()` |
| `stmt` | Statement (instruction) | `let x = 1;` |
| `ty` | Type | `i32`, `Vec<String>`, `impl Trait` |
| `ident` | Identifiant | `mon_var`, `MaStruct`, `do_thing` |
| `pat` | Pattern | `Some(x)`, `42`, `(a, b)` |
| `tt` | Token tree (1 token ou arbre) | N'importe quoi |
| `literal` | Littéral | `42`, `"hello"`, `3.14` |
| `path` | Chemin de module | `std::collections::HashMap` |
| `meta` | Méta-attribut | `derive(Debug)`, `cfg(test)` |
| `item` | Item Rust | `fn`, `struct`, `impl`, `mod` |
| `block` | Bloc de code | `{ ... }` |
| `lifetime` | Durée de vie | `'a`, `'static` |
| `vis` | Visibilité | `pub`, `pub(crate)` |

### Répétitions

```rust
// $(...)* : zéro ou plusieurs fois
// $(...)+ : une ou plusieurs fois
// $(...)? : zéro ou une fois

macro_rules! mon_vec {
    // Version avec zéro ou plusieurs éléments
    ($($element:expr),* $(,)?) => {
        {
            let mut v = Vec::new();
            $(
                v.push($element); // Répété pour chaque $element
            )*
            v
        }
    };
}

fn main() {
    let v1: Vec<i32> = mon_vec![];
    let v2 = mon_vec![1, 2, 3];
    let v3 = mon_vec![1, 2, 3,]; // La virgule finale est OK grâce à $(,)?

    println!("{:?}", v1); // []
    println!("{:?}", v2); // [1, 2, 3]
    println!("{:?}", v3); // [1, 2, 3]
}
```

### Exemple progressif — macro `assoc!`

```rust
// Créer un HashMap en une ligne
macro_rules! assoc {
    ($($clé:expr => $valeur:expr),* $(,)?) => {
        {
            use std::collections::HashMap;
            let mut map = HashMap::new();
            $(
                map.insert($clé, $valeur);
            )*
            map
        }
    };
}

fn main() {
    let scores = assoc! {
        "Alice" => 100,
        "Bob" => 85,
        "Charlie" => 92,
    };
    println!("{:?}", scores);
}
```

### Exemple avancé — macro de logging structuré

```rust
macro_rules! log_structuré {
    ($niveau:ident, $message:expr $(, $clé:ident = $valeur:expr)*) => {
        {
            print!("[{}] {}", stringify!($niveau), $message);
            $(
                print!(" | {}={:?}", stringify!($clé), $valeur);
            )*
            println!();
        }
    };
}

fn main() {
    log_structuré!(INFO, "Connexion établie", ip = "192.168.1.1", port = 8080);
    log_structuré!(ERROR, "Échec DB", code = 500, table = "users");
    log_structuré!(DEBUG, "Simple message");
}
// Sortie :
// [INFO] Connexion établie | ip="192.168.1.1" | port=8080
// [ERROR] Échec DB | code=500 | table="users"
// [DEBUG] Simple message
```

### Macro récursive — implémenter un mini-DSL

```rust
// Parser de règles de validation
macro_rules! valider {
    // Cas de base : aucune règle
    ($valeur:expr) => { true };

    // Règle non-null
    ($valeur:expr, non_vide $(, $reste:tt)*) => {
        {
            let v = &$valeur;
            !v.is_empty() && valider!(v.to_string(), $($reste),*)
        }
    };

    // Règle de longueur minimale
    ($valeur:expr, min_len = $n:expr $(, $reste:tt)*) => {
        {
            let v = &$valeur;
            v.len() >= $n && valider!(v.to_string(), $($reste),*)
        }
    };
}

fn main() {
    let email = "alice@example.com";
    let valide = valider!(email, non_vide, min_len = 5);
    println!("Email valide : {}", valide); // true

    let vide = "";
    println!("Vide valide : {}", valider!(vide, non_vide)); // false
}
```

---

## 3. Hygiène des macros

Les macros Rust sont **hygiéniques** : les identifiants introduits dans une macro n'interfèrent pas avec ceux du code environnant.

```rust
macro_rules! créer_variable {
    ($valeur:expr) => {
        let x = $valeur; // Ce `x` est hygiénique — isolé dans la macro
        x * 2
    };
}

fn main() {
    let x = 100; // Ce `x` et celui de la macro sont distincts
    let résultat = créer_variable!(5);
    println!("x = {}, résultat = {}", x, résultat); // x = 100, résultat = 10
    // x n'a pas été modifié par la macro
}
```

> [!info] Cas où l'hygiène ne s'applique pas
> Les identifiants passés *en argument* à la macro peuvent être utilisés dans le code généré. C'est intentionnel et permet de créer des macros qui introduisent des variables dans le scope appelant (pattern `$nom:ident`).

---

## 4. Macros Procédurales — Introduction

Les macros procédurales sont des **fonctions** qui reçoivent du Rust sous forme de `TokenStream` et retournent un `TokenStream`. Elles ont accès à l'AST complet et peuvent donc faire bien plus que les macros déclaratives.

### Setup du projet

```toml
# Cargo.toml du workspace
[workspace]
members = ["mon-app", "mon-derive"]

# mon-derive/Cargo.toml
[package]
name = "mon-derive"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true  # OBLIGATOIRE pour les macros procédurales

[dependencies]
syn = { version = "2", features = ["full"] }
quote = "1"
proc-macro2 = "1"
```

---

## 5. Derive Macros

### Implémenter `#[derive(Description)]`

```rust
// mon-derive/src/lib.rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput, Data, Fields};

#[proc_macro_derive(Description)]
pub fn derive_description(input: TokenStream) -> TokenStream {
    // Parser l'input en AST Rust
    let input = parse_macro_input!(input as DeriveInput);
    let nom = &input.ident; // Nom de la struct/enum

    // Générer l'implémentation
    let output = quote! {
        impl Description for #nom {
            fn description(&self) -> String {
                format!("Instance de {}", stringify!(#nom))
            }
        }
    };

    output.into()
}
```

### Derive plus complet — accéder aux champs

```rust
// Derive qui génère un Builder pattern
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput, Data, Fields, Type};

#[proc_macro_derive(Builder)]
pub fn derive_builder(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let nom = &input.ident;
    let nom_builder = syn::Ident::new(&format!("{}Builder", nom), nom.span());

    // Extraire les champs de la struct
    let champs = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(fields) => &fields.named,
            _ => panic!("Builder supporte uniquement les structs avec champs nommés"),
        },
        _ => panic!("Builder supporte uniquement les structs"),
    };

    let noms_champs: Vec<_> = champs.iter()
        .map(|f| f.ident.as_ref().unwrap())
        .collect();

    let types_champs: Vec<_> = champs.iter()
        .map(|f| &f.ty)
        .collect();

    let output = quote! {
        // Struct Builder avec tous les champs en Option
        pub struct #nom_builder {
            #(#noms_champs: Option<#types_champs>),*
        }

        impl #nom_builder {
            pub fn new() -> Self {
                #nom_builder {
                    #(#noms_champs: None),*
                }
            }

            // Méthodes setter pour chaque champ
            #(
                pub fn #noms_champs(mut self, valeur: #types_champs) -> Self {
                    self.#noms_champs = Some(valeur);
                    self
                }
            )*

            pub fn build(self) -> Result<#nom, String> {
                Ok(#nom {
                    #(
                        #noms_champs: self.#noms_champs
                            .ok_or_else(|| format!("Champ '{}' manquant", stringify!(#noms_champs)))?
                    ),*
                })
            }
        }

        impl #nom {
            pub fn builder() -> #nom_builder {
                #nom_builder::new()
            }
        }
    };

    output.into()
}
```

```rust
// Utilisation dans mon-app/src/main.rs
use mon_derive::Builder;

#[derive(Debug, Builder)]
struct Serveur {
    host: String,
    port: u16,
    max_connexions: u32,
}

fn main() {
    let serveur = Serveur::builder()
        .host("localhost".to_string())
        .port(8080)
        .max_connexions(100)
        .build()
        .expect("Configuration invalide");

    println!("{:?}", serveur);
}
```

---

## 6. Attribute Macros

Les attribute macros transforment l'item sur lequel elles sont appliquées. Elles reçoivent deux TokenStream : les arguments de l'attribut et l'item annoté.

### Macro d'instrumentation automatique

```rust
// mon-derive/src/lib.rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, ItemFn, FnArg, ReturnType};

#[proc_macro_attribute]
pub fn instrumener(_attrs: TokenStream, item: TokenStream) -> TokenStream {
    let fonction = parse_macro_input!(item as ItemFn);
    let nom = &fonction.sig.ident;
    let nom_str = nom.to_string();
    let bloc = &fonction.block;
    let signature = &fonction.sig;
    let visibilité = &fonction.vis;

    let output = quote! {
        #visibilité #signature {
            let __début = std::time::Instant::now();
            println!("[TRACE] Entrée dans {}", #nom_str);
            let __résultat = (|| #bloc)();
            println!("[TRACE] Sortie de {} en {:?}", #nom_str, __début.elapsed());
            __résultat
        }
    };

    output.into()
}
```

```rust
// Utilisation
use mon_derive::instrumener;

#[instrumener]
fn calculer_factorial(n: u64) -> u64 {
    if n == 0 { 1 } else { n * calculer_factorial(n - 1) }
}

fn main() {
    println!("{}", calculer_factorial(10));
    // [TRACE] Entrée dans calculer_factorial
    // [TRACE] Sortie de calculer_factorial en 2.1µs
    // 3628800
}
```

### Macro d'attribut avec arguments

```rust
#[proc_macro_attribute]
pub fn retry(attrs: TokenStream, item: TokenStream) -> TokenStream {
    // Parser les attributs : #[retry(max = 3, délai_ms = 100)]
    let args = parse_macro_input!(attrs as syn::AttributeArgs);
    let fonction = parse_macro_input!(item as ItemFn);

    // Extraire max et délai des arguments (simplification)
    let max_tentatives = 3u32; // Extraction réelle des args omise pour clarté
    let délai_ms = 100u64;

    let nom = &fonction.sig.ident;
    let visibilité = &fonction.vis;
    let signature = &fonction.sig;
    let bloc = &fonction.block;

    let output = quote! {
        #visibilité #signature {
            let mut tentative = 0u32;
            loop {
                let résultat = (|| #bloc)();
                match résultat {
                    Ok(val) => return Ok(val),
                    Err(e) if tentative < #max_tentatives => {
                        tentative += 1;
                        eprintln!("Tentative {} échouée : {:?}", tentative, e);
                        std::thread::sleep(std::time::Duration::from_millis(#délai_ms));
                    }
                    Err(e) => return Err(e),
                }
            }
        }
    };

    output.into()
}
```

---

## 7. Function-like Macros Procédurales

Syntaxiquement identiques aux macros déclaratives (`macro!(...)`) mais disposent de toute la puissance des macros procédurales.

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    // Valider et transformer une query SQL à la compilation
    let input_str = input.to_string();

    // Vérification statique simplifiée
    if !input_str.trim_matches('"').to_uppercase().starts_with("SELECT") {
        return quote! {
            compile_error!("Seules les requêtes SELECT sont autorisées")
        }.into();
    }

    // Retourner la string validée avec des métadonnées
    let query = input_str.trim_matches('"');
    quote! {
        {
            // En pratique : pré-compilation, validation de schéma, etc.
            #query
        }
    }.into()
}
```

---

## 8. Build Scripts — `build.rs`

Le fichier `build.rs` à la racine du projet est exécuté par Cargo avant la compilation. Il permet de générer du code, compiler des bibliothèques C, etc.

```rust
// build.rs
use std::env;
use std::fs;
use std::path::Path;

fn main() {
    // Variables disponibles dans le code via env!()
    println!("cargo:rustc-env=BUILD_TIMESTAMP={}", chrono::Utc::now().to_rfc3339());

    // Déclencher la re-exécution si un fichier change
    println!("cargo:rerun-if-changed=src/schema.json");

    // Générer du code Rust depuis un fichier externe
    let out_dir = env::var("OUT_DIR").unwrap();
    let dest_path = Path::new(&out_dir).join("generated.rs");

    let schema_json = fs::read_to_string("src/schema.json")
        .expect("schema.json introuvable");

    let code_généré = générer_code_depuis_schema(&schema_json);
    fs::write(&dest_path, code_généré).unwrap();
}

fn générer_code_depuis_schema(json: &str) -> String {
    // Parser le JSON et générer des structs Rust correspondantes
    // (Simplifié pour l'exemple)
    format!(r#"
pub const SCHEMA_VERSION: &str = "1.0";
pub fn valider_schema() -> bool {{ true }}
    "#)
}
```

```rust
// src/main.rs — utiliser le code généré
include!(concat!(env!("OUT_DIR"), "/generated.rs"));

fn main() {
    println!("Version schema : {}", SCHEMA_VERSION);
    println!("Build timestamp : {}", env!("BUILD_TIMESTAMP"));
}
```

---

## 9. Macros utiles de la stdlib

```rust
fn main() {
    // dbg! — affiche fichier, ligne, expression et valeur
    let x = dbg!(2 + 3); // [src/main.rs:2] 2 + 3 = 5
    println!("{}", x);   // 5

    // todo! — marque du code non implémenté
    fn future_feature() -> i32 {
        todo!("Implémenter le calcul de score")
    }

    // unimplemented! — similaire à todo! mais sémantique "ne sera jamais fait ici"
    fn méthode_abstraite() -> String {
        unimplemented!()
    }

    // compile_error! — erreur personnalisée à la compilation
    // #[cfg(not(target_os = "linux"))]
    // compile_error!("Ce module ne supporte que Linux");

    // cfg! — vérification de configuration
    if cfg!(target_os = "windows") {
        println!("Windows détecté");
    } else if cfg!(target_os = "linux") {
        println!("Linux détecté");
    }

    // include_str! — inclure un fichier texte au moment de la compilation
    let contenu_readme = include_str!("../README.md");
    println!("README length : {}", contenu_readme.len());

    // concat! — concaténer des littéraux à la compilation
    const VERSION: &str = concat!("v", env!("CARGO_PKG_VERSION"));
    println!("Version : {}", VERSION);

    // stringify! — convertir une expression en &str
    let nom_macro = stringify!(Vec<HashMap<String, i32>>);
    println!("Type : {}", nom_macro);
}
```

### Macros de format avancées

```rust
fn main() {
    // format_args! — arguments de format sans allocation
    let args = format_args!("Hello {}", "world");
    print!("{}", args);

    // write! / writeln! — écriture dans un buffer
    use std::fmt::Write;
    let mut buf = String::new();
    write!(buf, "Valeur : {}", 42).unwrap();
    writeln!(buf, " — terminé").unwrap();
    println!("{}", buf);

    // eprintln! / eprint! — écrire sur stderr
    eprintln!("Erreur : quelque chose s'est mal passé");

    // assert_eq! avec message personnalisé
    let a = 2 + 2;
    assert_eq!(a, 4, "Arithmétique de base brisée : {} != {}", a, 4);

    // matches! — tester un pattern sans binding
    let val = Some(42);
    assert!(matches!(val, Some(x) if x > 0));
}
```

---

## 10. Macros Conditionnelles et `cfg`

```rust
// Attributs cfg
#[cfg(debug_assertions)]
fn vérification_debug() {
    println!("Mode debug actif");
}

#[cfg(feature = "experimental")]
mod fonctionnalités_expérimentales {
    pub fn nouvelle_feature() { todo!() }
}

// Définitions conditionnelles selon l'OS
#[cfg(target_os = "windows")]
fn chemin_config() -> &'static str { "C:\\AppData\\config.toml" }

#[cfg(not(target_os = "windows"))]
fn chemin_config() -> &'static str { "~/.config/config.toml" }

// Tester plusieurs conditions
#[cfg(all(target_os = "linux", target_arch = "x86_64"))]
fn feature_linux_x86_64() {}

#[cfg(any(feature = "json", feature = "toml"))]
fn parser_config() {}

fn main() {
    #[cfg(debug_assertions)]
    vérification_debug();

    println!("Config : {}", chemin_config());
}
```

---

## 11. Patterns Avancés de Macros Déclaratives

### Macro avec compteur d'index

```rust
// Générer des variantes d'un enum avec des indices
macro_rules! enum_indexé {
    ($nom:ident { $($variante:ident),* $(,)? }) => {
        #[derive(Debug, Clone, Copy, PartialEq, Eq)]
        enum $nom {
            $($variante),*
        }

        impl $nom {
            fn index(&self) -> usize {
                let mut i = 0;
                $(
                    if *self == $nom::$variante { return i; }
                    i += 1;
                )*
                unreachable!()
            }

            fn depuis_index(idx: usize) -> Option<Self> {
                let mut i = 0;
                $(
                    if i == idx { return Some($nom::$variante); }
                    i += 1;
                )*
                None
            }
        }
    };
}

enum_indexé!(Couleur { Rouge, Vert, Bleu, Jaune });

fn main() {
    let c = Couleur::Vert;
    println!("Index de Vert : {}", c.index()); // 1
    println!("Couleur à l'index 2 : {:?}", Couleur::depuis_index(2)); // Some(Bleu)
}
```

### Macro pour implémenter des traits sur des tuples

```rust
// Implémenter un trait sur des tuples de différentes arités
macro_rules! impl_somme_tuple {
    ($($T:ident),*) => {
        impl<$($T: std::ops::Add<Output = $T> + Default + Copy),*> Sommable for ($($T,)*) {
            // Implementation simplifiée
        }
    };
}

trait Sommable {
    fn somme_éléments(&self) -> String;
}
```

---

## 12. Exercices Pratiques

> [!tip] Exercices progressifs — de `macro_rules!` aux macros procédurales

### Exercice 1 — Macro `my_map!` (Débutant)
Implémentez une macro `my_map!` qui crée un `HashMap` :
```rust
let m = my_map!{ "a" => 1, "b" => 2, "c" => 3 };
```
La macro doit accepter 0 ou plusieurs paires clé-valeur, avec virgule finale optionnelle.

### Exercice 2 — Macro de tests paramétrés (Intermédiaire)
Créez une macro `test_cases!` qui génère plusieurs fonctions de test :
```rust
test_cases! {
    test_addition: { addition(2, 3) == 5 }
    test_soustraction: { soustraction(10, 4) == 6 }
    test_multiplication: { multiplication(3, 4) == 12 }
}
// Génère #[test] fn test_addition() { assert!(addition(2, 3) == 5); } etc.
```

### Exercice 3 — Derive `Validator` (Intermédiaire+)
Créez une macro procédurale `#[derive(Validator)]` qui génère une méthode `valider(&self) -> Result<(), Vec<String>>` :
- Un champ `String` annoté `#[validation(non_vide)]` vérifie que la string n'est pas vide
- Un champ `u32` annoté `#[validation(min = 1, max = 100)]` vérifie la plage

```rust
#[derive(Validator)]
struct Formulaire {
    #[validation(non_vide)]
    nom: String,
    #[validation(min = 1, max = 100)]
    age: u32,
}
```

### Exercice 4 — Attribute macro `#[memoize]` (Avancé)
Implémentez une macro d'attribut `#[memoize]` qui ajoute de la mise en cache automatique à une fonction pure :
- La fonction annotée doit retourner un type `Clone + Eq + Hash`
- Les arguments doivent aussi être `Hash + Eq + Clone`
- Utiliser un `HashMap` thread-local pour le cache
- Générer le wrapper automatiquement

### Exercice 5 — Build script DSL (Avancé)
Créez un `build.rs` qui lit un fichier `messages.toml` et génère automatiquement des constantes Rust :
```toml
# messages.toml
[fr]
bonjour = "Bonjour !"
au_revoir = "Au revoir !"

[en]
bonjour = "Hello!"
au_revoir = "Goodbye!"
```
Le code généré doit exposer `pub fn message(lang: &str, clé: &str) -> &'static str`.

---

## Notes liées

- [[06 - Traits et Generiques]] — Les macros `derive` génèrent des implémentations de traits
- [[03 - Structs Enums et Pattern Matching]] — `syn` parse les structs/enums pour les derive macros
- [[02 - Ownership et Borrowing]] — Les macros procédurales manipulent des `TokenStream` (ownership standard)
- [[04 - Gestion des Erreurs en Rust]] — `compile_error!`, erreurs de compilation depuis les macros
- [[05 - Collections et Iterateurs]] — `vec!`, `format!` sont des exemples de macros déclaratives
- [[08 - Concurrence et Smart Pointers]] — `thread_local!` est une macro déclarative pour le TLS
- [[11 - Unsafe Rust et FFI]] — Les macros `bindgen`/`cbindgen` sont des cas pratiques de build scripts
