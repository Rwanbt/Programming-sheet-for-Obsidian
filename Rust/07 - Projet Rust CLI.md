# Projet Rust CLI : minigrep

Construire un projet complet est la meilleure facon de consolider toutes les notions vues jusqu'ici : ownership, gestion d'erreurs, traits, iterateurs, et tests. Dans ce chapitre, nous allons creer **minigrep**, un outil en ligne de commande qui recherche du texte dans des fichiers, similaire a `grep` sous Unix. Ce projet couvre l'ensemble du cycle de vie d'un programme Rust : de `cargo new` jusqu'a `cargo publish`.

Chaque section construit incrementalement sur la precedente. A la fin, vous aurez un outil fonctionnel, teste, documente, et publiable. Si vous venez du C, vous decouvrirez a quel point Rust simplifie des taches qui demandent beaucoup de code manuel en C : parsing d'arguments, gestion d'erreurs, et tests automatises.

> [!tip] Analogie
> Construire minigrep, c'est comme construire une maison piece par piece. D'abord les fondations (structure du projet), puis la plomberie (lecture de fichiers), l'electricite (logique de recherche), la peinture (sortie coloree), et enfin l'inspection finale (tests). Chaque etape est independante mais contribue au tout.

---

## Structure du projet

### Initialisation avec Cargo

```bash
# Creer le projet
cargo new minigrep
cd minigrep

# Structure initiale generee par cargo
tree minigrep/
```

```
minigrep/
|-- Cargo.toml              # Manifeste du projet (dependances, metadata)
|-- src/
|   |-- main.rs             # Point d'entree
|   |-- lib.rs              # Logique metier (on va le creer)
|-- tests/
|   |-- integration.rs      # Tests d'integration (on va le creer)
```

> [!info] Pourquoi separer main.rs et lib.rs ?
> La separation est une convention forte en Rust :
> - `main.rs` : point d'entree, parsing des arguments, appel de la logique, gestion de la sortie
> - `lib.rs` : toute la logique metier, testable independamment
> 
> Cela permet de tester la logique sans lancer le binaire, et de reutiliser le code comme bibliotheque.

### Cargo.toml : le manifeste

```toml
[package]
name = "minigrep"
version = "0.1.0"
edition = "2021"
authors = ["Votre Nom <votre@email.com>"]
description = "Un outil de recherche de texte dans les fichiers, comme grep"
license = "MIT"
readme = "README.md"
repository = "https://github.com/vous/minigrep"
keywords = ["cli", "grep", "search", "text"]
categories = ["command-line-utilities"]

[dependencies]
clap = { version = "4", features = ["derive"] }
colored = "2"

[dev-dependencies]
assert_cmd = "2"
predicates = "3"
tempfile = "3"
```

```
Architecture du projet complet :

minigrep/
|-- Cargo.toml
|-- Cargo.lock                  # Genere automatiquement (versions exactes)
|-- src/
|   |-- main.rs                 # Point d'entree + parsing CLI
|   |-- lib.rs                  # Config, recherche, logique metier
|-- tests/
|   |-- integration.rs          # Tests end-to-end
|-- target/                     # Genere par cargo (binaires, cache)
|   |-- debug/minigrep          # Binaire debug
|   |-- release/minigrep        # Binaire optimise

Comparaison avec un projet C equivalent :

minigrep_c/
|-- Makefile                    # A ecrire manuellement
|-- src/
|   |-- main.c                  # Tout dans le meme fichier souvent
|   |-- search.c                # Pas de convention standard
|   |-- search.h                # Headers a maintenir manuellement
|-- tests/
|   |-- test_search.c           # Framework de test a installer soi-meme
```

---

## Parsing des arguments avec clap

### Pourquoi clap ?

En C, parser les arguments demande soit `getopt` (limité), soit du parsing manuel avec `argv/argc`. En Rust, le crate `clap` genere automatiquement l'aide, la validation, et les sous-commandes a partir de simples annotations.

```
Parsing d'arguments : C vs Rust

C (getopt / manuel)                    Rust (clap derive)
+--------------------------------------+--------------------------------------+
| int main(int argc, char *argv[]) {   | #[derive(Parser)]                    |
|   char *query = NULL;                | struct Args {                        |
|   char *filename = NULL;             |     query: String,                   |
|   int ignore_case = 0;              |     filename: PathBuf,               |
|   int opt;                           |     #[arg(short, long)]              |
|   while ((opt = getopt(argc, argv,   |     ignore_case: bool,               |
|           "i")) != -1) {            | }                                    |
|     switch (opt) {                   |                                      |
|       case 'i': ignore_case=1;break; | // clap genere automatiquement :     |
|       default: usage(); exit(1);     | // --help, --version, validation,    |
|     }                                | // messages d'erreur, completion     |
|   }                                  |                                      |
|   // Validation manuelle...          | let args = Args::parse();            |
|   // Messages d'erreur manuels...    | // C'est tout !                      |
| }                                    |                                      |
+--------------------------------------+--------------------------------------+
```

### Definition de la CLI avec derive macro

```rust
// src/main.rs
use clap::{Parser, Subcommand};
use std::path::PathBuf;

/// minigrep - Recherche de texte dans les fichiers
#[derive(Parser, Debug)]
#[command(
    name = "minigrep",
    version,
    author,
    about = "Recherche un motif dans un fichier",
    long_about = "minigrep est un outil en ligne de commande qui recherche \
                  un motif (texte ou regex) dans un ou plusieurs fichiers. \
                  Similaire a grep, mais ecrit en Rust."
)]
struct Args {
    /// Le motif a rechercher
    #[arg(value_name = "MOTIF")]
    query: String,

    /// Le fichier dans lequel chercher
    #[arg(value_name = "FICHIER")]
    filename: PathBuf,

    /// Ignorer la casse lors de la recherche
    #[arg(short, long)]
    ignore_case: bool,

    /// Afficher les numeros de ligne
    #[arg(short = 'n', long)]
    line_numbers: bool,

    /// Compter le nombre de correspondances au lieu de les afficher
    #[arg(short, long)]
    count: bool,

    /// Sous-commande optionnelle
    #[command(subcommand)]
    command: Option<Commands>,
}

#[derive(Subcommand, Debug)]
enum Commands {
    /// Afficher des statistiques sur le fichier
    Stats {
        /// Le fichier a analyser
        #[arg(value_name = "FICHIER")]
        filename: PathBuf,
    },
}

fn main() {
    let args = Args::parse();
    println!("{:?}", args);
}
```

### Aide generee automatiquement

```bash
$ minigrep --help
```

```
minigrep - Recherche de texte dans les fichiers

Usage: minigrep [OPTIONS] <MOTIF> <FICHIER>
       minigrep [OPTIONS] <COMMAND>

Arguments:
  <MOTIF>    Le motif a rechercher
  <FICHIER>  Le fichier dans lequel chercher

Options:
  -i, --ignore-case    Ignorer la casse lors de la recherche
  -n, --line-numbers   Afficher les numeros de ligne
  -c, --count          Compter le nombre de correspondances
  -h, --help           Print help (see more with '--help')
  -V, --version        Print version

Commands:
  stats  Afficher des statistiques sur le fichier
  help   Print this message or the help of the given subcommand(s)
```

> [!info] Attributs clap importants
> - `#[command(version)]` : utilise la version de `Cargo.toml`
> - `#[arg(short, long)]` : genere `-i` et `--ignore-case`
> - `#[arg(value_name = "NOM")]` : nom affiche dans l'aide
> - `#[arg(default_value = "...")]` : valeur par defaut
> - `#[command(subcommand)]` : sous-commandes (comme `git commit`, `git push`)

---

## Lecture de fichiers

### Lire un fichier avec gestion d'erreurs

```rust
use std::fs;
use std::path::Path;

/// Lit le contenu d'un fichier et retourne le texte
/// 
/// # Errors
/// Retourne une erreur si le fichier n'existe pas ou n'est pas lisible
fn lire_fichier(chemin: &Path) -> Result<String, Box<dyn std::error::Error>> {
    let contenu = fs::read_to_string(chemin)?;
    Ok(contenu)
}
```

```
Flux de lecture d'un fichier :

    Programme                    Systeme de fichiers
       |                               |
       |  fs::read_to_string(path)     |
       |------------------------------>|
       |                               |
       |  Result<String, io::Error>    |
       |<------------------------------|
       |                               |
       |-- Ok(contenu) --> utiliser le texte
       |-- Err(e) ------> afficher l'erreur et quitter
```

### Comparaison avec C

```c
// En C : lecture de fichier = beaucoup de code, beaucoup d'erreurs possibles
#include <stdio.h>
#include <stdlib.h>

char* lire_fichier(const char *chemin) {
    FILE *f = fopen(chemin, "r");
    if (!f) return NULL;               // Erreur 1 : fichier introuvable

    fseek(f, 0, SEEK_END);
    long taille = ftell(f);
    if (taille < 0) {                  // Erreur 2 : ftell echoue
        fclose(f);
        return NULL;
    }
    rewind(f);

    char *contenu = malloc(taille + 1);
    if (!contenu) {                     // Erreur 3 : malloc echoue
        fclose(f);
        return NULL;
    }

    size_t lu = fread(contenu, 1, taille, f);
    contenu[lu] = '\0';                // Erreur 4 : oublier le \0 = UB
    fclose(f);                         // Erreur 5 : oublier fclose = fuite
    return contenu;                    // Erreur 6 : appelant doit free() !
}
```

```rust
// En Rust : une seule ligne, toutes les erreurs gerees
fn lire_fichier(chemin: &Path) -> Result<String, std::io::Error> {
    std::fs::read_to_string(chemin)  // C'est tout. Pas de malloc, pas de fclose.
}
```

> [!warning] Fichiers volumineux
> `read_to_string` charge tout le fichier en memoire. Pour les tres gros fichiers, utilisez `BufReader` avec `lines()` pour lire ligne par ligne sans tout charger.

---

## La structure Config

### Encapsuler la configuration

```rust
// src/lib.rs
use std::path::PathBuf;

/// Configuration de la recherche
pub struct Config {
    pub query: String,
    pub filename: PathBuf,
    pub ignore_case: bool,
    pub line_numbers: bool,
    pub count_only: bool,
}

impl Config {
    /// Cree une Config a partir des arguments parses
    pub fn new(
        query: String,
        filename: PathBuf,
        ignore_case: bool,
        line_numbers: bool,
        count_only: bool,
    ) -> Config {
        Config {
            query,
            filename,
            ignore_case,
            line_numbers,
            count_only,
        }
    }
}
```

---

## Logique de recherche

### Recherche sensible a la casse

```rust
/// Recherche les lignes contenant `query` dans `contenu`
/// Retourne un vecteur de tuples (numero_ligne, ligne)
pub fn search<'a>(query: &str, contenu: &'a str) -> Vec<(usize, &'a str)> {
    contenu
        .lines()
        .enumerate()
        .filter(|(_, ligne)| ligne.contains(query))
        .map(|(num, ligne)| (num + 1, ligne))  // Lignes numerotees a partir de 1
        .collect()
}
```

### Recherche insensible a la casse

```rust
/// Recherche insensible a la casse
pub fn search_case_insensitive<'a>(query: &str, contenu: &'a str) -> Vec<(usize, &'a str)> {
    let query_lower = query.to_lowercase();
    contenu
        .lines()
        .enumerate()
        .filter(|(_, ligne)| ligne.to_lowercase().contains(&query_lower))
        .map(|(num, ligne)| (num + 1, ligne))
        .collect()
}
```

```
Flux de la recherche avec iterateurs :

    "Bonjour le monde\nbonjour Rust\nHello World"
                    |
                .lines()
                    |
    ["Bonjour le monde", "bonjour Rust", "Hello World"]
                    |
              .enumerate()
                    |
    [(0, "Bonjour le monde"), (1, "bonjour Rust"), (2, "Hello World")]
                    |
          .filter(contient "bonjour" ignore case)
                    |
    [(0, "Bonjour le monde"), (1, "bonjour Rust")]
                    |
          .map(num + 1)
                    |
    [(1, "Bonjour le monde"), (2, "bonjour Rust")]
                    |
              .collect()
                    |
    Vec[(1, "Bonjour le monde"), (2, "bonjour Rust")]
```

> [!tip] Analogie
> Les iterateurs fonctionnent comme une chaine de montage dans une usine. Chaque etape (lines, enumerate, filter, map, collect) est un poste de travail. Les pieces (les lignes) passent d'un poste a l'autre, et seules celles qui passent tous les controles arrivent a la fin. Rien n'est stocke en memoire entre les etapes : c'est du traitement **lazy** (paresseux).

### Comparaison : recherche en C vs Rust

```c
// En C : recherche manuelle avec strstr et boucles
#include <string.h>
#include <stdio.h>
#include <ctype.h>

// Recherche case-insensitive en C = cauchemar
char* strcasestr_manual(const char *haystack, const char *needle) {
    // Il faut l'ecrire soi-meme sur certains systemes
    size_t nlen = strlen(needle);
    for (; *haystack; haystack++) {
        if (strncasecmp(haystack, needle, nlen) == 0)
            return (char*)haystack;
    }
    return NULL;
}

void search(const char *query, const char *contenu, int ignore_case) {
    const char *ptr = contenu;
    int line_num = 1;
    while (*ptr) {
        const char *eol = strchr(ptr, '\n');
        if (!eol) eol = ptr + strlen(ptr);

        // Copier la ligne dans un buffer temporaire
        size_t len = eol - ptr;
        char line[4096];  // Buffer fixe = danger de debordement
        if (len >= sizeof(line)) len = sizeof(line) - 1;
        strncpy(line, ptr, len);
        line[len] = '\0';

        // Recherche
        int found = ignore_case
            ? (strcasestr_manual(line, query) != NULL)
            : (strstr(line, query) != NULL);

        if (found) {
            printf("%d: %s\n", line_num, line);
        }

        line_num++;
        ptr = (*eol) ? eol + 1 : eol;
    }
}
```

```rust
// En Rust : equivalent en quelques lignes, sans buffer overflow possible
pub fn search<'a>(query: &str, contenu: &'a str, ignore_case: bool) -> Vec<(usize, &'a str)> {
    let query_cmp = if ignore_case { query.to_lowercase() } else { query.to_string() };
    contenu
        .lines()
        .enumerate()
        .filter(|(_, ligne)| {
            let ligne_cmp = if ignore_case { ligne.to_lowercase() } else { ligne.to_string() };
            ligne_cmp.contains(&query_cmp)
        })
        .map(|(num, ligne)| (num + 1, ligne))
        .collect()
}
```

---

## Variables d'environnement

### Lire IGNORE_CASE

```rust
use std::env;

impl Config {
    /// Cree une Config en tenant compte des variables d'environnement
    pub fn from_args_and_env(
        query: String,
        filename: PathBuf,
        ignore_case_flag: bool,
        line_numbers: bool,
        count_only: bool,
    ) -> Config {
        // Le flag CLI a priorite, sinon on regarde la variable d'environnement
        let ignore_case = ignore_case_flag || env::var("IGNORE_CASE").is_ok();

        Config {
            query,
            filename,
            ignore_case,
            line_numbers,
            count_only,
        }
    }
}
```

```bash
# Utilisation :
$ minigrep bonjour poeme.txt           # Sensible a la casse
$ minigrep -i bonjour poeme.txt        # Ignore la casse (flag)
$ IGNORE_CASE=1 minigrep bonjour poeme.txt  # Ignore la casse (env)
```

> [!info] std::env::var
> `env::var("NOM")` retourne `Result<String, VarError>`. Si la variable n'existe pas, c'est `Err`. On utilise `.is_ok()` pour juste verifier sa presence, ou `.unwrap_or("defaut".to_string())` pour une valeur par defaut.

---

## Sortie coloree

### Utiliser le crate colored

```rust
use colored::*;

/// Affiche une ligne avec le motif mis en surbrillance
fn afficher_resultat(
    num_ligne: usize,
    ligne: &str,
    query: &str,
    afficher_num: bool,
) {
    // Mettre en surbrillance le motif trouve
    let ligne_coloree = ligne.replace(
        query,
        &query.red().bold().to_string(),
    );

    if afficher_num {
        print!("{}: ", num_ligne.to_string().green());
    }
    println!("{}", ligne_coloree);
}
```

```
Sortie coloree dans le terminal :

    $ minigrep -n "Rust" fichier.txt

    3:  J'apprends [Rust] depuis 2 semaines     (Rust en rouge gras)
    7:  [Rust] est un langage systeme            (3 et 7 en vert)
    15: Le compilateur [Rust] est strict

    $ minigrep -c "Rust" fichier.txt
    3 correspondances trouvees
```

---

## Sortie stderr et codes de sortie

### Erreurs vers stderr avec eprintln!

```rust
use std::process;

fn main() {
    let args = Args::parse();

    // Construire la config
    let config = Config::from_args_and_env(
        args.query,
        args.filename,
        args.ignore_case,
        args.line_numbers,
        args.count,
    );

    // Executer et gerer les erreurs
    if let Err(e) = run(config) {
        eprintln!("Erreur : {}", e);   // stderr, pas stdout
        process::exit(1);               // Code de sortie non-zero = erreur
    }
}
```

> [!warning] println! vs eprintln!
> - `println!` ecrit sur **stdout** : les resultats normaux. Peut etre redirige avec `>`.
> - `eprintln!` ecrit sur **stderr** : les erreurs. Visible meme avec `> fichier.txt`.
> 
> ```bash
> # L'erreur apparait dans le terminal, pas dans output.txt
> $ minigrep motif inexistant.txt > output.txt
> Erreur : Le fichier 'inexistant.txt' n'existe pas
> ```

### Codes de sortie

```rust
use std::process;

// Convention Unix pour les codes de sortie :
// 0 = succes
// 1 = erreur generale
// 2 = mauvaise utilisation de la commande

fn main() {
    let args = Args::parse();

    let config = Config::from_args_and_env(
        args.query,
        args.filename,
        args.ignore_case,
        args.line_numbers,
        args.count,
    );

    match run(config) {
        Ok(count) => {
            if count == 0 {
                process::exit(1);  // Aucun resultat (comme grep)
            }
            process::exit(0);      // Succes
        }
        Err(e) => {
            eprintln!("Erreur : {}", e);
            process::exit(2);      // Erreur
        }
    }
}
```

---

## Implementation complete

### src/lib.rs

```rust
use std::fs;
use std::path::PathBuf;
use std::error::Error;
use colored::*;

/// Configuration de la recherche minigrep
pub struct Config {
    pub query: String,
    pub filename: PathBuf,
    pub ignore_case: bool,
    pub line_numbers: bool,
    pub count_only: bool,
}

impl Config {
    /// Cree une Config a partir des arguments et variables d'environnement
    pub fn new(
        query: String,
        filename: PathBuf,
        ignore_case: bool,
        line_numbers: bool,
        count_only: bool,
    ) -> Config {
        let ignore_case = ignore_case || std::env::var("IGNORE_CASE").is_ok();
        Config { query, filename, ignore_case, line_numbers, count_only }
    }
}

/// Resultat d'une recherche : (numero de ligne, contenu de la ligne)
pub type SearchResult<'a> = Vec<(usize, &'a str)>;

/// Recherche sensible a la casse
pub fn search<'a>(query: &str, contenu: &'a str) -> SearchResult<'a> {
    contenu
        .lines()
        .enumerate()
        .filter(|(_, ligne)| ligne.contains(query))
        .map(|(num, ligne)| (num + 1, ligne))
        .collect()
}

/// Recherche insensible a la casse
pub fn search_case_insensitive<'a>(query: &str, contenu: &'a str) -> SearchResult<'a> {
    let query_lower = query.to_lowercase();
    contenu
        .lines()
        .enumerate()
        .filter(|(_, ligne)| ligne.to_lowercase().contains(&query_lower))
        .map(|(num, ligne)| (num + 1, ligne))
        .collect()
}

/// Affiche une ligne de resultat avec coloration
fn afficher_ligne(num: usize, ligne: &str, query: &str, afficher_num: bool) {
    let coloree = ligne.replace(query, &query.red().bold().to_string());
    if afficher_num {
        print!("{}: ", num.to_string().green());
    }
    println!("{}", coloree);
}

/// Execute la recherche complete
/// Retourne le nombre de resultats trouves, ou une erreur
pub fn run(config: Config) -> Result<usize, Box<dyn Error>> {
    let contenu = fs::read_to_string(&config.filename)
        .map_err(|e| format!(
            "Impossible de lire '{}' : {}",
            config.filename.display(), e
        ))?;

    let resultats = if config.ignore_case {
        search_case_insensitive(&config.query, &contenu)
    } else {
        search(&config.query, &contenu)
    };

    if config.count_only {
        println!("{} correspondance(s) trouvee(s)", resultats.len());
    } else {
        for (num, ligne) in &resultats {
            afficher_ligne(*num, ligne, &config.query, config.line_numbers);
        }
    }

    Ok(resultats.len())
}
```

### src/main.rs

```rust
use clap::{Parser, Subcommand};
use std::path::PathBuf;
use std::process;

use minigrep::Config;

/// minigrep - Recherche de texte dans les fichiers
#[derive(Parser, Debug)]
#[command(name = "minigrep", version, author, about = "Recherche un motif dans un fichier")]
struct Args {
    /// Le motif a rechercher
    #[arg(value_name = "MOTIF")]
    query: String,

    /// Le fichier dans lequel chercher
    #[arg(value_name = "FICHIER")]
    filename: PathBuf,

    /// Ignorer la casse
    #[arg(short, long)]
    ignore_case: bool,

    /// Afficher les numeros de ligne
    #[arg(short = 'n', long)]
    line_numbers: bool,

    /// Compter les correspondances
    #[arg(short, long)]
    count: bool,

    /// Sous-commande
    #[command(subcommand)]
    command: Option<Commands>,
}

#[derive(Subcommand, Debug)]
enum Commands {
    /// Statistiques sur un fichier
    Stats {
        #[arg(value_name = "FICHIER")]
        filename: PathBuf,
    },
}

fn main() {
    let args = Args::parse();

    // Gerer les sous-commandes
    if let Some(Commands::Stats { filename }) = &args.command {
        match std::fs::read_to_string(filename) {
            Ok(contenu) => {
                let lignes = contenu.lines().count();
                let mots: usize = contenu.lines().map(|l| l.split_whitespace().count()).sum();
                let caracteres = contenu.len();
                println!("Fichier : {}", filename.display());
                println!("Lignes  : {}", lignes);
                println!("Mots    : {}", mots);
                println!("Octets  : {}", caracteres);
            }
            Err(e) => {
                eprintln!("Erreur : {}", e);
                process::exit(2);
            }
        }
        return;
    }

    // Recherche normale
    let config = Config::new(
        args.query,
        args.filename,
        args.ignore_case,
        args.line_numbers,
        args.count,
    );

    match minigrep::run(config) {
        Ok(0) => process::exit(1),     // Aucun resultat
        Ok(_) => process::exit(0),     // Succes
        Err(e) => {
            eprintln!("Erreur : {}", e);
            process::exit(2);
        }
    }
}
```

---

## Tests

### Tests unitaires dans lib.rs

Les tests unitaires vivent dans le meme fichier que le code qu'ils testent, dans un module `#[cfg(test)]`.

```rust
// A la fin de src/lib.rs

#[cfg(test)]
mod tests {
    use super::*;

    // Contenu de test reutilisable
    const POEME: &str = "\
Rust est rapide et sur,
le compilateur est strict.
rust n'est pas un langage facile,
mais Rust vaut l'effort.
RUST pour les systemes !";

    #[test]
    fn recherche_sensible_casse() {
        let resultats = search("Rust", POEME);
        assert_eq!(resultats.len(), 2);
        assert_eq!(resultats[0], (1, "Rust est rapide et sur,"));
        assert_eq!(resultats[1], (4, "mais Rust vaut l'effort."));
    }

    #[test]
    fn recherche_insensible_casse() {
        let resultats = search_case_insensitive("rust", POEME);
        assert_eq!(resultats.len(), 4);
        assert_eq!(resultats[0].0, 1);  // Ligne 1 : "Rust est..."
        assert_eq!(resultats[1].0, 3);  // Ligne 3 : "rust n'est..."
        assert_eq!(resultats[2].0, 4);  // Ligne 4 : "mais Rust..."
        assert_eq!(resultats[3].0, 5);  // Ligne 5 : "RUST pour..."
    }

    #[test]
    fn recherche_aucun_resultat() {
        let resultats = search("Python", POEME);
        assert!(resultats.is_empty());
    }

    #[test]
    fn recherche_mot_partiel() {
        let resultats = search("compil", POEME);
        assert_eq!(resultats.len(), 1);
        assert_eq!(resultats[0].1, "le compilateur est strict.");
    }

    #[test]
    fn recherche_chaine_vide() {
        let resultats = search("", POEME);
        // Chaque ligne contient la chaine vide
        assert_eq!(resultats.len(), 5);
    }

    #[test]
    fn recherche_dans_contenu_vide() {
        let resultats = search("Rust", "");
        assert!(resultats.is_empty());
    }
}
```

```bash
# Lancer les tests
$ cargo test

running 6 tests
test tests::recherche_sensible_casse ... ok
test tests::recherche_insensible_casse ... ok
test tests::recherche_aucun_resultat ... ok
test tests::recherche_mot_partiel ... ok
test tests::recherche_chaine_vide ... ok
test tests::recherche_dans_contenu_vide ... ok

test result: ok. 6 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

```
Organisation des tests en Rust :

                          Tests
                            |
            +---------------+---------------+
            |                               |
     Tests unitaires                Tests d'integration
     (dans src/*.rs)                (dans tests/*.rs)
            |                               |
    #[cfg(test)]                    Utilisent l'API publique
    mod tests { ... }               comme un utilisateur externe
            |                               |
    Acces aux fonctions             N'ont acces qu'a pub
    privees du module               (comme une crate externe)
            |                               |
    Rapides, ciblent une            Testent le comportement
    seule fonction                  global du programme
```

### Tests d'integration

```rust
// tests/integration.rs
use std::fs;
use tempfile::NamedTempFile;
use std::io::Write;

// Les tests d'integration importent la crate comme un utilisateur externe
use minigrep;
use minigrep::Config;
use std::path::PathBuf;

/// Cree un fichier temporaire avec le contenu donne et retourne son chemin
fn creer_fichier_temp(contenu: &str) -> (NamedTempFile, PathBuf) {
    let mut fichier = NamedTempFile::new().expect("Impossible de creer fichier temp");
    write!(fichier, "{}", contenu).expect("Impossible d'ecrire");
    let chemin = fichier.path().to_path_buf();
    (fichier, chemin)
}

#[test]
fn integration_recherche_basique() {
    let contenu = "premiere ligne\ndeuxieme ligne avec Rust\ntroisieme ligne";
    let (_f, chemin) = creer_fichier_temp(contenu);

    let config = Config::new(
        "Rust".to_string(),
        chemin,
        false,   // case sensitive
        false,   // pas de numeros
        false,   // pas de comptage
    );

    let resultat = minigrep::run(config);
    assert!(resultat.is_ok());
    assert_eq!(resultat.unwrap(), 1);  // 1 correspondance
}

#[test]
fn integration_fichier_inexistant() {
    let config = Config::new(
        "test".to_string(),
        PathBuf::from("fichier_qui_nexiste_pas.txt"),
        false, false, false,
    );

    let resultat = minigrep::run(config);
    assert!(resultat.is_err());
}

#[test]
fn integration_ignore_case() {
    let contenu = "Hello World\nhello rust\nHELLO THERE";
    let (_f, chemin) = creer_fichier_temp(contenu);

    let config = Config::new(
        "hello".to_string(),
        chemin,
        true,    // ignore case
        false, false,
    );

    let resultat = minigrep::run(config);
    assert!(resultat.is_ok());
    assert_eq!(resultat.unwrap(), 3);  // Les 3 lignes correspondent
}

#[test]
fn integration_aucune_correspondance() {
    let contenu = "abc\ndef\nghi";
    let (_f, chemin) = creer_fichier_temp(contenu);

    let config = Config::new(
        "xyz".to_string(),
        chemin,
        false, false, false,
    );

    let resultat = minigrep::run(config);
    assert!(resultat.is_ok());
    assert_eq!(resultat.unwrap(), 0);
}
```

```bash
# Lancer seulement les tests d'integration
$ cargo test --test integration

# Lancer un test specifique
$ cargo test recherche_sensible

# Lancer les tests avec la sortie affichee
$ cargo test -- --nocapture
```

> [!example] Organisation des tests
> ```
> cargo test                          # Tous les tests
> cargo test --lib                    # Tests unitaires seulement
> cargo test --test integration       # Tests d'integration seulement
> cargo test recherche_sensible       # Tests dont le nom contient ce motif
> cargo test -- --nocapture           # Voir println! pendant les tests
> cargo test -- --test-threads=1      # Tests sequentiels (pas en parallele)
> ```

---

## Documentation

### Doc comments avec ///

```rust
/// Recherche les lignes contenant `query` dans `contenu`.
///
/// La recherche est sensible a la casse. Pour une recherche
/// insensible, utilisez [`search_case_insensitive`].
///
/// # Arguments
///
/// * `query` - Le motif a rechercher
/// * `contenu` - Le texte dans lequel chercher
///
/// # Returns
///
/// Un vecteur de tuples `(numero_ligne, texte_ligne)` pour chaque
/// ligne contenant le motif.
///
/// # Examples
///
/// ```
/// use minigrep::search;
///
/// let contenu = "Rust est genial\nPython aussi\nRust est rapide";
/// let resultats = search("Rust", contenu);
///
/// assert_eq!(resultats.len(), 2);
/// assert_eq!(resultats[0], (1, "Rust est genial"));
/// assert_eq!(resultats[1], (3, "Rust est rapide"));
/// ```
///
/// Recherche sans resultat :
///
/// ```
/// use minigrep::search;
///
/// let resultats = search("Java", "Rust est genial");
/// assert!(resultats.is_empty());
/// ```
pub fn search<'a>(query: &str, contenu: &'a str) -> Vec<(usize, &'a str)> {
    contenu
        .lines()
        .enumerate()
        .filter(|(_, ligne)| ligne.contains(query))
        .map(|(num, ligne)| (num + 1, ligne))
        .collect()
}
```

```bash
# Generer la documentation HTML
$ cargo doc --open

# Les exemples dans /// sont aussi des tests !
$ cargo test --doc
```

> [!info] Les exemples de doc sont des tests
> Les blocs de code dans les commentaires `///` sont compiles et executes par `cargo test --doc`. C'est une garantie que la documentation reste synchronisee avec le code. Si un exemple ne compile plus, le test echoue.

---

## Publication sur crates.io

### Preparer la publication

```bash
# 1. Verifier que tout compile et que les tests passent
$ cargo build
$ cargo test
$ cargo doc

# 2. Verifier le packaging (ce qui sera inclus)
$ cargo package --list

# 3. Se connecter a crates.io (necessite un compte)
$ cargo login <votre-token-api>

# 4. Publier (irreversible pour cette version !)
$ cargo publish

# 5. Publier une nouvelle version
# Modifier version dans Cargo.toml : "0.1.0" -> "0.1.1"
$ cargo publish
```

```
Versioning semantique (SemVer) :

    MAJOR.MINOR.PATCH
      |     |     |
      |     |     +-- Corrections de bugs (retro-compatible)
      |     +-------- Nouvelles fonctionnalites (retro-compatible)
      +-------------- Changements incompatibles (breaking changes)

    0.1.0 -> 0.1.1   Correction de bug
    0.1.1 -> 0.2.0   Nouvelle fonctionnalite
    0.2.0 -> 1.0.0   Premiere version stable / breaking change
    1.0.0 -> 1.0.1   Correction de bug
    1.0.1 -> 1.1.0   Nouvelle fonctionnalite retro-compatible
    1.1.0 -> 2.0.0   Breaking change
```

> [!warning] Publication irreversible
> Une fois publiee, une version ne peut plus etre supprimee de crates.io. Vous pouvez la "yank" (la marquer comme non recommandee), mais elle restera telechargeable. Verifiez bien avant de publier : tests, documentation, metadata dans Cargo.toml.

---

## Build release et distribution

### Compiler en mode release

```bash
# Mode debug (par defaut) : compilation rapide, binaire non optimise
$ cargo build
# -> target/debug/minigrep (gros, lent, avec symboles de debug)

# Mode release : compilation lente, binaire optimise
$ cargo build --release
# -> target/release/minigrep (petit, rapide, sans symboles)
```

```
Comparaison debug vs release :

                    Debug               Release
    +-------------------------------------------+
    Compilation    Rapide (~2s)         Lent (~10s)
    Taille         ~5 Mo               ~1 Mo
    Vitesse exec.  Lent                3-10x plus rapide
    Symboles       Oui (gdb)           Non (strip)
    Assertions     debug_assert! actif  desactive
    Overflow       Panic               Wrapping (comme C)
    +-------------------------------------------+

    Utiliser debug pendant le developpement.
    Utiliser release pour la distribution.
```

### Distribution du binaire

```bash
# Le binaire Rust est statiquement lie (pas de dependance runtime)
$ ldd target/release/minigrep
    # Tres peu de dependances (libc, libpthread, etc.)

# Copier le binaire n'importe ou
$ cp target/release/minigrep /usr/local/bin/

# Cross-compilation pour d'autres plateformes
$ rustup target add x86_64-unknown-linux-musl
$ cargo build --release --target x86_64-unknown-linux-musl
# -> Binaire 100% statique, zero dependance
```

> [!info] Avantage sur C
> En C, distribuer un binaire necessite de gerer les dependances partagees (.so/.dll), ou de compiler statiquement (complique avec glibc). En Rust avec musl, un `cargo build --release --target x86_64-unknown-linux-musl` produit un binaire unique sans aucune dependance.

---

## Recapitulatif : le code complet

### Fichier src/lib.rs complet

```rust
//! # minigrep
//!
//! `minigrep` est un outil de recherche de texte dans les fichiers.
//! Il supporte la recherche sensible/insensible a la casse,
//! l'affichage des numeros de ligne, et la sortie coloree.

use std::error::Error;
use std::fs;
use std::path::PathBuf;
use colored::*;

/// Configuration pour une recherche minigrep
pub struct Config {
    /// Le motif a rechercher
    pub query: String,
    /// Le chemin du fichier a analyser
    pub filename: PathBuf,
    /// Ignorer la casse si vrai
    pub ignore_case: bool,
    /// Afficher les numeros de ligne si vrai
    pub line_numbers: bool,
    /// Compter uniquement (pas d'affichage des lignes)
    pub count_only: bool,
}

impl Config {
    /// Cree une nouvelle configuration
    ///
    /// La variable d'environnement `IGNORE_CASE` est consultee si
    /// `ignore_case` est `false`.
    ///
    /// # Examples
    ///
    /// ```
    /// use minigrep::Config;
    /// use std::path::PathBuf;
    ///
    /// let config = Config::new(
    ///     "motif".to_string(),
    ///     PathBuf::from("fichier.txt"),
    ///     false, false, false,
    /// );
    /// assert_eq!(config.query, "motif");
    /// ```
    pub fn new(
        query: String,
        filename: PathBuf,
        ignore_case: bool,
        line_numbers: bool,
        count_only: bool,
    ) -> Config {
        let ignore_case = ignore_case || std::env::var("IGNORE_CASE").is_ok();
        Config { query, filename, ignore_case, line_numbers, count_only }
    }
}

/// Type alias pour les resultats de recherche
pub type SearchResult<'a> = Vec<(usize, &'a str)>;

/// Recherche sensible a la casse
///
/// # Examples
///
/// ```
/// let resultats = minigrep::search("monde", "bonjour le monde\nhello world");
/// assert_eq!(resultats, vec![(1, "bonjour le monde")]);
/// ```
pub fn search<'a>(query: &str, contenu: &'a str) -> SearchResult<'a> {
    contenu
        .lines()
        .enumerate()
        .filter(|(_, ligne)| ligne.contains(query))
        .map(|(num, ligne)| (num + 1, ligne))
        .collect()
}

/// Recherche insensible a la casse
///
/// # Examples
///
/// ```
/// let resultats = minigrep::search_case_insensitive("rust", "Rust est genial\nPYTHON aussi");
/// assert_eq!(resultats, vec![(1, "Rust est genial")]);
/// ```
pub fn search_case_insensitive<'a>(query: &str, contenu: &'a str) -> SearchResult<'a> {
    let query_lower = query.to_lowercase();
    contenu
        .lines()
        .enumerate()
        .filter(|(_, ligne)| ligne.to_lowercase().contains(&query_lower))
        .map(|(num, ligne)| (num + 1, ligne))
        .collect()
}

/// Affiche une ligne de resultat avec coloration syntaxique
fn afficher_ligne(num: usize, ligne: &str, query: &str, afficher_num: bool) {
    let coloree = ligne.replace(query, &query.red().bold().to_string());
    if afficher_num {
        print!("{}: ", num.to_string().green());
    }
    println!("{}", coloree);
}

/// Execute la recherche et affiche les resultats
///
/// Retourne le nombre de correspondances trouvees.
///
/// # Errors
///
/// Retourne une erreur si le fichier ne peut pas etre lu.
pub fn run(config: Config) -> Result<usize, Box<dyn Error>> {
    let contenu = fs::read_to_string(&config.filename)
        .map_err(|e| format!(
            "Impossible de lire '{}' : {}",
            config.filename.display(), e
        ))?;

    let resultats = if config.ignore_case {
        search_case_insensitive(&config.query, &contenu)
    } else {
        search(&config.query, &contenu)
    };

    if config.count_only {
        println!("{} correspondance(s) trouvee(s)", resultats.len());
    } else {
        for (num, ligne) in &resultats {
            afficher_ligne(*num, ligne, &config.query, config.line_numbers);
        }
    }

    Ok(resultats.len())
}

#[cfg(test)]
mod tests {
    use super::*;

    const POEME: &str = "\
Rust est rapide et sur,
le compilateur est strict.
rust n'est pas un langage facile,
mais Rust vaut l'effort.
RUST pour les systemes !";

    #[test]
    fn recherche_sensible_casse() {
        let resultats = search("Rust", POEME);
        assert_eq!(resultats.len(), 2);
        assert_eq!(resultats[0], (1, "Rust est rapide et sur,"));
        assert_eq!(resultats[1], (4, "mais Rust vaut l'effort."));
    }

    #[test]
    fn recherche_insensible_casse() {
        let resultats = search_case_insensitive("rust", POEME);
        assert_eq!(resultats.len(), 4);
    }

    #[test]
    fn recherche_aucun_resultat() {
        let resultats = search("Python", POEME);
        assert!(resultats.is_empty());
    }

    #[test]
    fn recherche_mot_partiel() {
        let resultats = search("compil", POEME);
        assert_eq!(resultats.len(), 1);
        assert_eq!(resultats[0].1, "le compilateur est strict.");
    }

    #[test]
    fn recherche_chaine_vide() {
        let resultats = search("", POEME);
        assert_eq!(resultats.len(), 5);
    }

    #[test]
    fn recherche_dans_contenu_vide() {
        let resultats = search("Rust", "");
        assert!(resultats.is_empty());
    }
}
```

### Fichier src/main.rs complet

```rust
use clap::{Parser, Subcommand};
use std::path::PathBuf;
use std::process;

use minigrep::Config;

/// minigrep - Recherche de texte dans les fichiers
#[derive(Parser, Debug)]
#[command(name = "minigrep", version, author, about = "Recherche un motif dans un fichier")]
struct Args {
    /// Le motif a rechercher
    #[arg(value_name = "MOTIF")]
    query: String,

    /// Le fichier dans lequel chercher
    #[arg(value_name = "FICHIER")]
    filename: PathBuf,

    /// Ignorer la casse
    #[arg(short, long)]
    ignore_case: bool,

    /// Afficher les numeros de ligne
    #[arg(short = 'n', long)]
    line_numbers: bool,

    /// Compter les correspondances
    #[arg(short, long)]
    count: bool,

    #[command(subcommand)]
    command: Option<Commands>,
}

#[derive(Subcommand, Debug)]
enum Commands {
    /// Statistiques sur un fichier
    Stats {
        #[arg(value_name = "FICHIER")]
        filename: PathBuf,
    },
}

fn main() {
    let args = Args::parse();

    if let Some(Commands::Stats { filename }) = &args.command {
        match std::fs::read_to_string(filename) {
            Ok(contenu) => {
                let lignes = contenu.lines().count();
                let mots: usize = contenu.lines()
                    .map(|l| l.split_whitespace().count())
                    .sum();
                let octets = contenu.len();
                println!("Fichier : {}", filename.display());
                println!("Lignes  : {}", lignes);
                println!("Mots    : {}", mots);
                println!("Octets  : {}", octets);
            }
            Err(e) => {
                eprintln!("Erreur : {}", e);
                process::exit(2);
            }
        }
        return;
    }

    let config = Config::new(
        args.query,
        args.filename,
        args.ignore_case,
        args.line_numbers,
        args.count,
    );

    match minigrep::run(config) {
        Ok(0) => process::exit(1),
        Ok(_) => process::exit(0),
        Err(e) => {
            eprintln!("Erreur : {}", e);
            process::exit(2);
        }
    }
}
```

---

## Carte Mentale

```
                            PROJET CLI : MINIGREP
                                    |
            +-----------+-----------+-----------+-----------+
            |           |           |           |           |
        STRUCTURE     PARSING     LOGIQUE     TESTS      PUBLIER
            |           |           |           |           |
    +-------+----+   clap       +--+--+    +---+---+   Cargo.toml
    |            |   derive     |     |    |       |   metadata
  main.rs    lib.rs    |     search  run  Unit.  Integ.    |
  (entree)  (metier)   |       |     |    #[test] tests/  crates.io
    |            |   #[command] |     |      |       |   cargo publish
  Args::     Config  #[arg]  lines() |   assert!  API      |
  parse()    run()   subcmd  filter()| assert_eq! publique SemVer
    |            |       |   collect()|      |       |
  process::  Result  --help    |   colored  6 tests  4 tests
  exit()     Box<dyn>  -V    iter    eprintln!
                  |          chain      |
            env::var              stderr vs
            IGNORE_CASE           stdout

    C equivalent :                Rust :
    - getopt/manual argv         - clap (3 lignes)
    - strstr + boucles           - iterateurs (fonctionnel)
    - printf partout             - eprintln! pour erreurs
    - pas de framework de test   - cargo test integre
    - Makefile manuel            - cargo build --release
```

---

## Exercices

### Exercice 1 : Ajouter le support des expressions regulieres

Modifiez minigrep pour supporter les regex avec le crate `regex`. Ajoutez un flag `--regex` ou `-r` qui active le mode regex au lieu de la recherche textuelle simple.

```rust
// Ajouter dans Cargo.toml : regex = "1"

use regex::Regex;

/// Recherche par expression reguliere
pub fn search_regex<'a>(pattern: &str, contenu: &'a str) -> Result<SearchResult<'a>, regex::Error> {
    let re = Regex::new(pattern)?;
    let resultats = contenu
        .lines()
        .enumerate()
        .filter(|(_, ligne)| re.is_match(ligne))
        .map(|(num, ligne)| (num + 1, ligne))
        .collect();
    Ok(resultats)
}

// Ajouter dans Args :
//     #[arg(short, long)]
//     regex: bool,

// Modifier run() pour utiliser search_regex quand le flag est actif.

// Tests a ecrire :
#[test]
fn regex_chiffres() {
    let contenu = "abc 123\ndef ghi\njkl 456";
    let resultats = search_regex(r"\d+", contenu).unwrap();
    assert_eq!(resultats.len(), 2);
    // Lignes 1 et 3 contiennent des chiffres
}

#[test]
fn regex_email() {
    let contenu = "contact: user@example.com\npas d'email ici\nautre: test@rust.org";
    let resultats = search_regex(r"\w+@\w+\.\w+", contenu).unwrap();
    assert_eq!(resultats.len(), 2);
}

#[test]
fn regex_invalide() {
    let resultat = search_regex(r"[invalid", "texte");
    assert!(resultat.is_err());
}
```

### Exercice 2 : Recherche dans plusieurs fichiers

Etendez minigrep pour accepter plusieurs fichiers (ou un glob pattern). Affichez le nom du fichier devant chaque resultat quand plusieurs fichiers sont fournis.

```rust
// Modifier Args pour accepter plusieurs fichiers :
//     #[arg(value_name = "FICHIERS", num_args = 1..)]
//     filenames: Vec<PathBuf>,

// Ajouter dans Cargo.toml : glob = "0.3"

// Indice : utiliser glob::glob("src/**/*.rs") pour les patterns
// Indice : prefixer chaque ligne avec le nom du fichier en violet
//          filename.display().to_string().purple()

// Sortie attendue :
// src/main.rs:3: use minigrep::Config;
// src/lib.rs:15: pub struct Config {
// src/lib.rs:42: impl Config {
```

### Exercice 3 : Mode interactif

Ajoutez une sous-commande `interactive` qui lance un mode REPL : l'utilisateur tape un motif, minigrep cherche et affiche les resultats, puis attend un nouveau motif. `quit` pour sortir.

```rust
// Sous-commande a ajouter :
// Interactive {
//     #[arg(value_name = "FICHIER")]
//     filename: PathBuf,
// }

// Indice : utiliser std::io::stdin().read_line(&mut buffer)
// Indice : boucle loop { ... } avec break sur "quit"

use std::io::{self, Write};

fn mode_interactif(filename: &Path) -> Result<(), Box<dyn std::error::Error>> {
    let contenu = std::fs::read_to_string(filename)?;
    println!("Mode interactif - fichier : {}", filename.display());
    println!("Tapez un motif (ou 'quit' pour quitter) :");

    loop {
        print!("> ");
        io::stdout().flush()?;

        let mut input = String::new();
        io::stdin().read_line(&mut input)?;
        let input = input.trim();

        if input == "quit" || input == "exit" {
            println!("Au revoir !");
            break;
        }

        let resultats = search(input, &contenu);
        if resultats.is_empty() {
            println!("Aucun resultat pour '{}'", input);
        } else {
            for (num, ligne) in &resultats {
                // Afficher avec coloration
                todo!()
            }
            println!("--- {} resultat(s) ---", resultats.len());
        }
    }
    Ok(())
}
```

---

## Liens

- [[06 - Traits et Generiques]] --- Les traits utilises dans ce projet (Display, Error, From)
- [[05 - Collections et Iterateurs]] --- Les iterateurs sont au coeur de la logique de recherche
- [[04 - Gestion des Erreurs en Rust]] --- Result, ?, Box<dyn Error> utilises partout dans minigrep
