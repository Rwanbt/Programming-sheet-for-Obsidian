# Gestion des Erreurs en Rust

La gestion des erreurs est un domaine ou Rust se demarque radicalement du C et de la plupart des autres langages. Pas d'exceptions (comme Java/Python), pas de `NULL` omnipresent (comme C), pas de codes d'erreur oublies silencieusement. Rust force le programmeur a **confronter chaque erreur possible** au moment de la compilation. Le resultat : des programmes plus robustes, ou les bugs lies aux erreurs non gerees disparaissent completement.

Ce chapitre explore les deux categories d'erreurs en Rust, les outils pour les gerer, et comment creer vos propres types d'erreur. Vous verrez pourquoi le systeme de Rust est considere comme l'un des meilleurs qui existe, tout en restant performant a l'execution.

---

## Philosophie de Rust : ni exceptions, ni null

> [!tip] Analogie
> Imaginez un service postal. En C, quand un colis est perdu, le facteur pose silencieusement un papier vide dans votre boite sans vous prevenir. En Java, le facteur lance le colis par la fenetre et espere que quelqu'un l'attrape (exceptions). En Rust, le facteur vous remet un colis scelle portant une etiquette claire : "Colis recu" ou "Echec de livraison : raison X". Vous **devez** lire l'etiquette avant d'ouvrir.

```
Philosophie comparee :

C :         Retourne -1 ou NULL, errno global
            -> Le programmeur oublie de verifier ? Crash silencieux.

C++ / Java : Leve une exception
            -> L'exception traverse la pile d'appels invisiblement.
            -> On ne sait pas quelle fonction peut throw sans lire le code.

Rust :      Retourne Result<T, E> ou Option<T>
            -> Le compilateur REFUSE de continuer si l'erreur n'est pas geree.
            -> Le type de retour DOCUMENTE les erreurs possibles.
```

> [!info] Pas de null en Rust
> Rust n'a pas de valeur `null`. Tony Hoare, l'inventeur de `null`, l'a appele "mon erreur a un milliard de dollars". Rust utilise `Option<T>` a la place : soit `Some(valeur)`, soit `None`. C'est un enum, pas une valeur speciale, et le compilateur vous force a gerer les deux cas.

---

## Deux categories d'erreurs

Rust separe clairement les erreurs en deux familles :

```
                    Erreurs en Rust
                         |
            +------------+------------+
            |                         |
    Irrecuperables              Recuperables
        panic!()               Result<T, E>
            |                         |
    +-------+-------+        +-------+-------+
    |               |        |               |
  Bug dans       Etat       Fichier        Input
  le code      impossible   absent        invalide
    |               |        |               |
  Index hors     Invariant   On peut        On peut
  limites        viole       demander       redemander
                             un autre       a l'user
```

> [!warning] Regle d'or
> Utilisez `panic!` pour les **bugs** (situations qui ne devraient jamais arriver). Utilisez `Result` pour les **erreurs attendues** (situations qui peuvent legitimement se produire).

---

## panic! : erreurs irrecuperables

### Qu'est-ce que panic! ?

`panic!` arrete l'execution du thread courant. C'est l'equivalent d'un `abort()` en C, mais avec plus de controle.

```rust
fn main() {
    // panic! explicite avec un message
    panic!("Quelque chose de terrible s'est passe !");
}
```

```
Sortie :
thread 'main' panicked at 'Quelque chose de terrible s'est passe !', src/main.rs:2:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

### Paniques implicites

Certaines operations paniquent automatiquement quand elles echouent :

```rust
fn main() {
    let v = vec![1, 2, 3];

    // Acces hors limites : PANIQUE
    let n = v[10];  // thread 'main' panicked at 'index out of bounds'

    // En C, v[10] accede a de la memoire aleatoire (UB)
    // En Rust, c'est une panique immediate et propre
}
```

### Stack unwinding vs abort

Quand un `panic!` se produit, Rust a deux strategies possibles :

```
Strategy 1 : Unwinding (par defaut)
+-----------------------------------+
| Rust remonte la pile d'appels     |
| et appelle drop() sur chaque      |
| variable pour liberer la memoire  |
|                                   |
| main() -> foo() -> bar() -> PANIC |
|           drop     drop           |
|   drop                            |
+-----------------------------------+
Avantage : nettoyage propre
Inconvenient : binaire plus gros (~10%)

Strategy 2 : Abort
+-----------------------------------+
| Le programme s'arrete             |
| IMMEDIATEMENT.                    |
| L'OS recupere la memoire.         |
+-----------------------------------+
Avantage : binaire plus petit
Inconvenient : pas de nettoyage
```

Pour configurer le mode abort dans `Cargo.toml` :

```toml
[profile.release]
panic = "abort"   # Plus petit binaire, souvent utilise en embarque
```

### Comparaison avec C

```c
// En C : acces hors limites = comportement indefini
#include <stdio.h>

int main(void) {
    int tab[3] = {1, 2, 3};

    // COMPORTEMENT INDEFINI : peut retourner n'importe quoi,
    // crasher, ou meme sembler fonctionner
    printf("%d\n", tab[10]);

    // Verifier manuellement ?
    // Pas de mecanisme integre. Il faut passer la taille partout.
    return 0;
}
```

```rust
fn main() {
    let tab = vec![1, 2, 3];

    // Acces securise avec get() : retourne Option<&T>
    match tab.get(10) {
        Some(val) => println!("Valeur : {}", val),
        None => println!("Index hors limites !"),
    }

    // Acces direct avec [] : panique si hors limites
    // Au moins c'est DETERMINISTE, pas un comportement indefini
    // let val = tab[10];  // panic!
}
```

### unwrap et expect sur Option

```rust
fn main() {
    let nombres = vec![1, 2, 3];

    // unwrap : extrait la valeur ou panique
    let premier = nombres.get(0).unwrap();  // 1
    // let hors = nombres.get(10).unwrap();  // PANIQUE !

    // expect : comme unwrap avec un message personnalise
    let premier = nombres.get(0).expect("La liste ne devrait pas etre vide");

    // unwrap est acceptable dans :
    // - Les tests
    // - Les prototypes rapides
    // - Quand on SAIT que la valeur est presente (invariant prouve)
}
```

> [!warning] Ne parsemez pas votre code de `unwrap()`
> Chaque `unwrap()` est un `panic!` potentiel. En production, preferez `match`, `if let`, ou l'operateur `?`. Reservez `unwrap()` aux cas ou vous pouvez **prouver** que l'erreur est impossible.

---

## Result\<T, E\> : erreurs recuperables

### Le type Result

`Result` est un enum defini dans la bibliotheque standard :

```rust
enum Result<T, E> {
    Ok(T),    // Succes : contient la valeur de type T
    Err(E),   // Erreur : contient l'erreur de type E
}
```

```
Result<String, io::Error>
         |          |
         |          +-- Type de l'erreur
         +------------- Type du succes

Exemples concrets :
Result<String, io::Error>     -- Lecture de fichier
Result<i32, ParseIntError>    -- Parsing d'entier
Result<(), SqlError>          -- Insertion en BDD
Result<Config, ConfigError>   -- Chargement de config
```

### Matching sur Result

```rust
use std::fs;
use std::io;

fn main() {
    let resultat: Result<String, io::Error> = fs::read_to_string("config.txt");

    match resultat {
        Ok(contenu) => {
            println!("Fichier lu avec succes :");
            println!("{}", contenu);
        }
        Err(erreur) => {
            // On peut inspecter le type d'erreur
            match erreur.kind() {
                io::ErrorKind::NotFound => {
                    println!("Fichier non trouve, creation d'un fichier par defaut...");
                }
                io::ErrorKind::PermissionDenied => {
                    println!("Permission refusee !");
                }
                _ => {
                    println!("Erreur inattendue : {}", erreur);
                }
            }
        }
    }
}
```

### Methodes utiles sur Result

```rust
fn main() {
    let ok: Result<i32, String> = Ok(42);
    let err: Result<i32, String> = Err(String::from("echec"));

    // --- Extraction ---

    // unwrap : extrait Ok ou panique sur Err
    let n = ok.unwrap();  // 42
    // let n = err.unwrap();  // PANIQUE

    // expect : comme unwrap avec message personnalise
    let n = ok.expect("devrait etre Ok");  // 42

    // unwrap_or : valeur par defaut si Err
    let n = err.unwrap_or(0);  // 0

    // unwrap_or_else : calcul paresseux si Err
    let n = err.unwrap_or_else(|e| {
        eprintln!("Erreur ignoree : {}", e);
        -1
    });  // -1

    // unwrap_or_default : Default::default() si Err
    let n: Result<i32, String> = Err("oops".into());
    let n = n.unwrap_or_default();  // 0 (default de i32)

    // --- Transformation ---

    // map : transforme Ok, laisse Err intact
    let double = ok.map(|n| n * 2);  // Ok(84)

    // map_err : transforme Err, laisse Ok intact
    let err2 = err.map_err(|e| format!("ERREUR: {}", e));

    // and_then : chainage (retourne Result)
    let resultat = ok.and_then(|n| {
        if n > 0 { Ok(n * 10) } else { Err(String::from("negatif")) }
    });  // Ok(420)

    // --- Inspection ---

    // is_ok / is_err
    assert!(ok.is_ok());
    assert!(err.is_err());

    // ok() : convertit Result<T,E> en Option<T>
    let opt: Option<i32> = ok.ok();  // Some(42)
}
```

> [!tip] Analogie
> Pensez a `Result` comme un colis avec deux compartiments. `unwrap()` ouvre le colis brutalement : si c'est le mauvais compartiment, tout explose. `match` ouvre delicatement et verifie quel compartiment contient quelque chose. `map` transforme le contenu sans ouvrir le colis. `?` transmet le colis a votre chef si c'est une erreur.

```
Methodes sur Result<T, E> :

Extraction :
  unwrap()            -> T        (panique si Err)
  expect("msg")       -> T        (panique si Err, avec message)
  unwrap_or(val)      -> T        (val si Err)
  unwrap_or_else(f)   -> T        (f(e) si Err)
  unwrap_or_default() -> T        (T::default() si Err)

Transformation :
  map(f)              -> Result<U, E>    (f(t) si Ok)
  map_err(f)          -> Result<T, F>    (f(e) si Err)
  and_then(f)         -> Result<U, E>    (f(t) si Ok, ou Err)

Conversion :
  ok()                -> Option<T>       (Some(t) si Ok, None si Err)
  err()               -> Option<E>       (Some(e) si Err, None si Ok)
```

---

## L'operateur ? : propagation elegante

### Le probleme : la pyramide de match

```rust
use std::fs;
use std::io;

// Sans ? : verbeux et repetitif
fn lire_config_v1(chemin: &str) -> Result<u32, Box<dyn std::error::Error>> {
    let contenu = match fs::read_to_string(chemin) {
        Ok(c) => c,
        Err(e) => return Err(Box::new(e)),
    };
    let premiere_ligne = match contenu.lines().next() {
        Some(l) => l,
        None => return Err("Fichier vide".into()),
    };
    let nombre = match premiere_ligne.trim().parse::<u32>() {
        Ok(n) => n,
        Err(e) => return Err(Box::new(e)),
    };
    Ok(nombre)
}
```

### La solution : l'operateur ?

```rust
use std::fs;

// Avec ? : concis et lisible
fn lire_config_v2(chemin: &str) -> Result<u32, Box<dyn std::error::Error>> {
    let contenu = fs::read_to_string(chemin)?;
    let premiere_ligne = contenu.lines().next().ok_or("Fichier vide")?;
    let nombre = premiere_ligne.trim().parse::<u32>()?;
    Ok(nombre)
}

// Encore plus concis avec chainage
fn lire_config_v3(chemin: &str) -> Result<u32, Box<dyn std::error::Error>> {
    Ok(fs::read_to_string(chemin)?
        .lines()
        .next()
        .ok_or("Fichier vide")?
        .trim()
        .parse::<u32>()?)
}
```

```
Fonctionnement de l'operateur ? :

    let valeur = expression?;

    est equivalent a :

    let valeur = match expression {
        Ok(v)  => v,                   // Succes : extraire la valeur
        Err(e) => return Err(e.into()),  // Erreur : convertir et retourner
    };
                             ^^^^^
                             |
                 Conversion automatique via From trait !
```

### Conversion automatique avec From

L'operateur `?` appelle automatiquement `From::from()` pour convertir le type d'erreur. C'est ce qui permet de melanger differents types d'erreurs :

```rust
use std::fs;
use std::io;
use std::num::ParseIntError;

// Cette fonction retourne Box<dyn Error> qui accepte n'importe quelle erreur
fn traiter(chemin: &str) -> Result<i32, Box<dyn std::error::Error>> {
    let contenu = fs::read_to_string(chemin)?;    // io::Error -> Box<dyn Error>
    let nombre = contenu.trim().parse::<i32>()?;   // ParseIntError -> Box<dyn Error>
    Ok(nombre * 2)
}

// Avec un type d'erreur personnalise
#[derive(Debug)]
enum MonErreur {
    Io(io::Error),
    Parse(ParseIntError),
}

impl From<io::Error> for MonErreur {
    fn from(e: io::Error) -> Self {
        MonErreur::Io(e)
    }
}

impl From<ParseIntError> for MonErreur {
    fn from(e: ParseIntError) -> Self {
        MonErreur::Parse(e)
    }
}

fn traiter_v2(chemin: &str) -> Result<i32, MonErreur> {
    let contenu = fs::read_to_string(chemin)?;    // io::Error -> MonErreur via From
    let nombre = contenu.trim().parse::<i32>()?;   // ParseIntError -> MonErreur via From
    Ok(nombre * 2)
}
```

### ? fonctionne aussi avec Option

```rust
fn premier_mot(texte: &str) -> Option<&str> {
    let ligne = texte.lines().next()?;       // None -> return None
    let mot = ligne.split_whitespace().next()?;  // None -> return None
    Some(mot)
}
```

> [!info] L'operateur ? dans main()
> Par defaut, `main()` retourne `()`. Pour utiliser `?` dans main, declarez-la ainsi :
> ```rust
> fn main() -> Result<(), Box<dyn std::error::Error>> {
>     let contenu = std::fs::read_to_string("config.txt")?;
>     println!("{}", contenu);
>     Ok(())
> }
> ```
> Si le `Result` est `Err`, le programme affiche l'erreur et retourne un code de sortie non-zero.

---

## Types d'erreur personnalises

### Enum d'erreurs

```rust
use std::fmt;
use std::io;
use std::num::ParseIntError;

#[derive(Debug)]
enum ConfigError {
    FichierManquant(String),
    FormatInvalide { ligne: usize, message: String },
    ValeurHorsLimites { cle: String, valeur: i64, min: i64, max: i64 },
    IoError(io::Error),
    ParseError(ParseIntError),
}

// Implementer Display pour les messages lisibles
impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            ConfigError::FichierManquant(chemin) => {
                write!(f, "Fichier de configuration manquant : {}", chemin)
            }
            ConfigError::FormatInvalide { ligne, message } => {
                write!(f, "Format invalide a la ligne {} : {}", ligne, message)
            }
            ConfigError::ValeurHorsLimites { cle, valeur, min, max } => {
                write!(f, "Valeur {} = {} hors limites [{}, {}]", cle, valeur, min, max)
            }
            ConfigError::IoError(e) => write!(f, "Erreur I/O : {}", e),
            ConfigError::ParseError(e) => write!(f, "Erreur de parsing : {}", e),
        }
    }
}

// Implementer Error pour l'interoperabilite
impl std::error::Error for ConfigError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            ConfigError::IoError(e) => Some(e),
            ConfigError::ParseError(e) => Some(e),
            _ => None,
        }
    }
}

// Implementer From pour la conversion automatique avec ?
impl From<io::Error> for ConfigError {
    fn from(e: io::Error) -> Self {
        ConfigError::IoError(e)
    }
}

impl From<ParseIntError> for ConfigError {
    fn from(e: ParseIntError) -> Self {
        ConfigError::ParseError(e)
    }
}
```

> [!info] Le trait std::error::Error
> Le trait `Error` requiert `Display` + `Debug`. La methode optionnelle `source()` permet de chainer les erreurs pour afficher la cause racine. C'est l'equivalent d'une "error chain" dans d'autres langages.

### Utilisation du type personnalise

```rust
use std::fs;

fn charger_config(chemin: &str) -> Result<Config, ConfigError> {
    let contenu = fs::read_to_string(chemin)
        .map_err(|_| ConfigError::FichierManquant(chemin.to_string()))?;

    let mut config = Config::default();

    for (i, ligne) in contenu.lines().enumerate() {
        let parts: Vec<&str> = ligne.splitn(2, '=').collect();
        if parts.len() != 2 {
            return Err(ConfigError::FormatInvalide {
                ligne: i + 1,
                message: format!("Attendu 'cle=valeur', trouve : '{}'", ligne),
            });
        }

        let cle = parts[0].trim();
        let valeur = parts[1].trim();

        match cle {
            "port" => {
                let port: i64 = valeur.parse()
                    .map_err(|e| ConfigError::ParseError(e))?;
                if port < 1 || port > 65535 {
                    return Err(ConfigError::ValeurHorsLimites {
                        cle: "port".to_string(),
                        valeur: port,
                        min: 1,
                        max: 65535,
                    });
                }
                config.port = port as u16;
            }
            _ => {} // Ignorer les cles inconnues
        }
    }

    Ok(config)
}
```

---

## Le crate anyhow : erreurs simples pour les applications

Le crate `anyhow` simplifie la gestion d'erreurs dans les **applications** (pas les bibliotheques).

```toml
# Cargo.toml
[dependencies]
anyhow = "1"
```

```rust
use anyhow::{Result, Context, anyhow, bail, ensure};
use std::fs;

// anyhow::Result<T> = Result<T, anyhow::Error>
// anyhow::Error accepte TOUTE erreur (comme Box<dyn Error> mais mieux)

fn charger_config(chemin: &str) -> Result<Config> {
    let contenu = fs::read_to_string(chemin)
        .context(format!("Impossible de lire le fichier '{}'", chemin))?;
    //   ^^^^^^^ Ajoute du contexte a l'erreur

    let config: Config = serde_json::from_str(&contenu)
        .context("Le fichier de configuration n'est pas un JSON valide")?;

    // bail! = return Err(anyhow!(...))
    if config.port == 0 {
        bail!("Le port ne peut pas etre 0");
    }

    // ensure! = if !condition { bail!(...) }
    ensure!(config.port <= 65535, "Port {} hors limites", config.port);

    Ok(config)
}

fn main() -> Result<()> {
    let config = charger_config("config.json")?;
    println!("Port : {}", config.port);
    Ok(())
}
```

```
Sortie en cas d'erreur avec context() :

Error: Impossible de lire le fichier 'config.json'

Caused by:
    No such file or directory (os error 2)

-> La chaine de contexte permet de comprendre POURQUOI l'erreur s'est produite.
```

> [!tip] Analogie
> `anyhow` est comme un sac poubelle universel pour les erreurs : mettez-y n'importe quelle erreur, et ajoutez des etiquettes (context) pour savoir d'ou elle vient. Ideal pour les applications. Mais pour une bibliotheque, vos utilisateurs veulent trier les erreurs par type : utilisez `thiserror` a la place.

---

## Le crate thiserror : erreurs typees pour les bibliotheques

`thiserror` genere automatiquement les implementations de `Display`, `Error` et `From` via des macros :

```toml
# Cargo.toml
[dependencies]
thiserror = "1"
```

```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum ConfigError {
    #[error("Fichier de configuration manquant : {0}")]
    FichierManquant(String),

    #[error("Format invalide a la ligne {ligne} : {message}")]
    FormatInvalide {
        ligne: usize,
        message: String,
    },

    #[error("Valeur {cle} = {valeur} hors limites [{min}, {max}]")]
    ValeurHorsLimites {
        cle: String,
        valeur: i64,
        min: i64,
        max: i64,
    },

    #[error("Erreur I/O")]
    IoError(#[from] std::io::Error),
    //      ^^^^^^ Genere impl From<io::Error> automatiquement !

    #[error("Erreur de parsing")]
    ParseError(#[from] std::num::ParseIntError),
}

// C'est TOUT. Pas besoin d'implementer Display, Error, ni From manuellement.
// thiserror genere tout a la compilation.
```

```
Comparaison : sans vs avec thiserror

Sans thiserror :                     Avec thiserror :
~50 lignes de boilerplate            ~15 lignes
  impl Display for MonError            #[derive(Error, Debug)]
  impl Error for MonError              enum MonError {
  impl From<io::Error>                     #[error("...")]
  impl From<ParseIntError>                 Io(#[from] io::Error),
  ...                                  }
```

---

## Comparaison complete : C vs Rust

```c
// GESTION D'ERREURS EN C : un champ de mines

#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>

// Probleme 1 : conventions multiples et incoherentes
int ouvrir_fichier() {
    FILE *f = fopen("data.txt", "r");
    if (f == NULL) {       // Convention : retourne NULL
        return -1;         // Convention : retourne -1
    }
    // ...
    return 0;              // Convention : retourne 0 = succes
}

// Probleme 2 : errno est global et fragile
void exemple_errno() {
    FILE *f = fopen("inexistant.txt", "r");
    // errno est positionne ici
    printf("tentative d'ouverture\n");  // printf peut ECRASER errno !
    if (f == NULL) {
        // errno est peut-etre deja modifie
        perror("fopen");   // Peut afficher la mauvaise erreur
    }
}

// Probleme 3 : rien ne force a verifier les erreurs
void danger() {
    FILE *f = fopen("data.txt", "r");
    // Pas de verification : le compilateur ne dit RIEN
    char buf[100];
    fgets(buf, 100, f);  // Si f == NULL : CRASH ou pire
}

// Probleme 4 : valeurs de retour ambigues
int lire_entier(const char *s) {
    return atoi(s);  // Retourne 0 si erreur ET si s == "0"
                     // Comment distinguer ? On ne peut pas.
}
```

```rust
// GESTION D'ERREURS EN RUST : ceinture ET bretelles

use std::fs;
use std::io;

// Avantage 1 : le type Result documente les erreurs possibles
fn ouvrir_fichier(chemin: &str) -> Result<String, io::Error> {
    fs::read_to_string(chemin)  // Le type de retour dit TOUT
}

// Avantage 2 : pas de variable globale (pas d'errno)
fn exemple_pas_errno() -> Result<(), io::Error> {
    let contenu = fs::read_to_string("inexistant.txt")?;
    // L'erreur est DANS le Result, pas dans un etat global
    // Aucune autre operation ne peut l'ecraser
    println!("{}", contenu);
    Ok(())
}

// Avantage 3 : le compilateur force la verification
fn securise() {
    let resultat = fs::read_to_string("data.txt");
    // Si on ignore resultat :
    // warning: unused `Result` that must be used
    // Le compilateur nous PREVIENT
}

// Avantage 4 : pas d'ambiguite
fn lire_entier(s: &str) -> Result<i32, std::num::ParseIntError> {
    s.parse::<i32>()
    // Ok(0) si s == "0"
    // Err(ParseIntError) si s est invalide
    // ZERO ambiguite
}
```

```
Resume de la comparaison :

Critere                  C                    Rust
---------------------------------------------------------
Signaler une erreur     return -1/NULL       Result<T, E>
Variable globale        errno                Non (dans le type)
Verif obligatoire       Non                  Oui (#[must_use])
Ambiguite               Frequente            Impossible
Type safety             Aucune               Totale
Propagation             Manuelle (if/goto)   Operateur ?
Erreurs personnalisees  Conventions ad-hoc   enum + traits
Performance             Identique            Identique (zero-cost)
```

---

## Bonnes pratiques

### Quand utiliser panic! vs Result

```
Utilisez panic! quand :                Utilisez Result quand :
+------------------------------------+------------------------------------+
| - Un invariant est viole           | - L'erreur est attendue            |
| - L'etat est corrompu              | - L'appelant peut reagir           |
| - C'est un bug dans le code        | - L'utilisateur peut corriger      |
| - Tests et prototypes              | - Code de production               |
| - Exemples et documentation        | - Bibliotheques                    |
+------------------------------------+------------------------------------+

Exemples concrets :
  panic! -> acces a un index "garanti valide" qui ne l'est pas
  panic! -> assert! dans un test
  Result -> fichier absent (l'utilisateur peut choisir un autre)
  Result -> connexion reseau echouee (on peut reessayer)
  Result -> input utilisateur invalide (on peut redemander)
```

### Gestion d'erreurs dans main

```rust
use std::process;

fn run() -> Result<(), Box<dyn std::error::Error>> {
    let config = charger_config("app.toml")?;
    let donnees = lire_donnees(&config.chemin_donnees)?;
    let resultat = traiter(&donnees)?;
    println!("Resultat : {}", resultat);
    Ok(())
}

fn main() {
    // Pattern commun : main() appelle run() et gere le code de sortie
    if let Err(e) = run() {
        eprintln!("Erreur : {}", e);

        // Afficher la chaine de causes
        let mut cause = e.source();
        while let Some(c) = cause {
            eprintln!("  Cause : {}", c);
            cause = c.source();
        }

        process::exit(1);
    }
}
```

> [!example] Codes de sortie en Rust
> ```rust
> use std::process::ExitCode;
> 
> fn main() -> ExitCode {
>     match faire_quelque_chose() {
>         Ok(_) => ExitCode::SUCCESS,      // 0
>         Err(_) => ExitCode::FAILURE,     // 1
>     }
> }
> ```

---

## Exemple pratique : parseur de fichier CSV

Voici un exemple complet qui combine tout ce que nous avons vu :

```rust
use std::fs;
use std::fmt;
use std::num::{ParseIntError, ParseFloatError};

// --- Types d'erreur ---

#[derive(Debug)]
enum CsvError {
    Io(std::io::Error),
    FichierVide,
    MauvaisNombreColonnes { ligne: usize, attendu: usize, trouve: usize },
    ParseEntier { ligne: usize, colonne: String, source: ParseIntError },
    ParseFlottant { ligne: usize, colonne: String, source: ParseFloatError },
    ValeurInvalide { ligne: usize, message: String },
}

impl fmt::Display for CsvError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            CsvError::Io(e) => write!(f, "Erreur de lecture : {}", e),
            CsvError::FichierVide => write!(f, "Le fichier CSV est vide"),
            CsvError::MauvaisNombreColonnes { ligne, attendu, trouve } => {
                write!(f, "Ligne {} : attendu {} colonnes, trouve {}", ligne, attendu, trouve)
            }
            CsvError::ParseEntier { ligne, colonne, source } => {
                write!(f, "Ligne {} colonne '{}' : {}", ligne, colonne, source)
            }
            CsvError::ParseFlottant { ligne, colonne, source } => {
                write!(f, "Ligne {} colonne '{}' : {}", ligne, colonne, source)
            }
            CsvError::ValeurInvalide { ligne, message } => {
                write!(f, "Ligne {} : {}", ligne, message)
            }
        }
    }
}

impl std::error::Error for CsvError {}

impl From<std::io::Error> for CsvError {
    fn from(e: std::io::Error) -> Self {
        CsvError::Io(e)
    }
}

// --- Structures de donnees ---

#[derive(Debug)]
struct Etudiant {
    nom: String,
    age: u32,
    note: f64,
}

// --- Parsing ---

fn parser_csv(chemin: &str) -> Result<Vec<Etudiant>, CsvError> {
    let contenu = fs::read_to_string(chemin)?;  // ? convertit io::Error -> CsvError

    let mut lignes = contenu.lines();

    // Verifier l'en-tete
    let entete = lignes.next().ok_or(CsvError::FichierVide)?;
    let nb_colonnes = entete.split(',').count();

    let mut etudiants = Vec::new();

    for (i, ligne) in lignes.enumerate() {
        let num_ligne = i + 2;  // +2 car on commence apres l'en-tete, et les humains comptent a partir de 1
        let champs: Vec<&str> = ligne.split(',').map(|s| s.trim()).collect();

        if champs.len() != nb_colonnes {
            return Err(CsvError::MauvaisNombreColonnes {
                ligne: num_ligne,
                attendu: nb_colonnes,
                trouve: champs.len(),
            });
        }

        let nom = champs[0].to_string();

        let age: u32 = champs[1].parse().map_err(|e| CsvError::ParseEntier {
            ligne: num_ligne,
            colonne: "age".to_string(),
            source: e,
        })?;

        let note: f64 = champs[2].parse().map_err(|e| CsvError::ParseFlottant {
            ligne: num_ligne,
            colonne: "note".to_string(),
            source: e,
        })?;

        if note < 0.0 || note > 20.0 {
            return Err(CsvError::ValeurInvalide {
                ligne: num_ligne,
                message: format!("Note {} hors de l'intervalle [0, 20]", note),
            });
        }

        etudiants.push(Etudiant { nom, age, note });
    }

    Ok(etudiants)
}

fn main() {
    match parser_csv("etudiants.csv") {
        Ok(etudiants) => {
            println!("{} etudiants charges :", etudiants.len());
            for e in &etudiants {
                println!("  {} (age {}) : {}/20", e.nom, e.age, e.note);
            }

            // Calculer la moyenne
            if !etudiants.is_empty() {
                let moyenne: f64 = etudiants.iter().map(|e| e.note).sum::<f64>()
                    / etudiants.len() as f64;
                println!("Moyenne : {:.2}/20", moyenne);
            }
        }
        Err(e) => {
            eprintln!("Erreur lors du parsing CSV : {}", e);
            std::process::exit(1);
        }
    }
}
```

---

## Carte Mentale

```
                    GESTION DES ERREURS EN RUST
                              |
            +-----------------+-----------------+
            |                                   |
       IRRECUPERABLE                      RECUPERABLE
        panic!()                         Result<T, E>
            |                                   |
    +-------+-------+              +------------+------------+
    |               |              |            |            |
  Quand ?        Comment ?       Ok(T)       Err(E)     Operateur ?
    |               |              |            |            |
  - Bugs          Unwinding       Valeur     Erreur      Propage
  - Tests         ou Abort        reussie    typee       auto.
  - Invariants                                |            |
                                         +----+----+    From trait
                                         |         |   (conversion)
                                     Methodes   Types
                                     utiles   personnalises
                                         |         |
                                  unwrap       enum + Display
                                  expect       + Error
                                  map          + From
                                  and_then         |
                                              +---------+
                                              |         |
                                           anyhow   thiserror
                                          (applis)  (biblio.)
```

---

## Exercices

### Exercice 1 : Convertisseur robuste

Creez une fonction `convertir_temperature(input: &str) -> Result<f64, String>` qui accepte des chaines comme `"32F"`, `"100C"`, `"0C"` et retourne la temperature en Celsius. Gerez les erreurs : format invalide, unite inconnue, valeur non parsable.

```rust
fn convertir_temperature(input: &str) -> Result<f64, String> {
    // Extraire la partie numerique et l'unite
    // "32F" -> (32.0, 'F')
    // Retourner Err si :
    //   - input est vide
    //   - l'unite n'est pas 'C' ou 'F'
    //   - la partie numerique n'est pas un nombre valide
    todo!()
}

fn main() {
    let tests = vec!["100C", "32F", "212F", "abc", "", "100X", "C"];
    for input in tests {
        match convertir_temperature(input) {
            Ok(c) => println!("{} -> {:.1}C", input, c),
            Err(e) => println!("{} -> ERREUR: {}", input, e),
        }
    }
}
```

### Exercice 2 : Chaine de propagation avec ?

Ecrivez un programme qui lit un fichier contenant un nombre par ligne, et retourne la somme. Utilisez `?` pour propager les erreurs a travers plusieurs fonctions. Definissez un type `SommeError` avec les variantes `Io`, `Parse`, et `FichierVide`.

```rust
use std::fs;

#[derive(Debug)]
enum SommeError {
    Io(std::io::Error),
    Parse { ligne: usize, source: std::num::ParseIntError },
    FichierVide,
}

// Implementer Display, Error, et From<io::Error> pour SommeError
// ...

fn lire_nombres(chemin: &str) -> Result<Vec<i64>, SommeError> {
    // Lire le fichier, parser chaque ligne en i64
    // Utiliser ? pour propager les erreurs
    todo!()
}

fn calculer_somme(chemin: &str) -> Result<i64, SommeError> {
    let nombres = lire_nombres(chemin)?;
    if nombres.is_empty() {
        return Err(SommeError::FichierVide);
    }
    Ok(nombres.iter().sum())
}

fn main() {
    match calculer_somme("nombres.txt") {
        Ok(somme) => println!("Somme = {}", somme),
        Err(e) => eprintln!("Erreur : {}", e),
    }
}
```

### Exercice 3 : Refactoring avec anyhow et thiserror

Reprenez l'exercice 2 et reecrivez-le en deux versions :
1. Version **anyhow** : remplacez le type d'erreur personnalise par `anyhow::Result`
2. Version **thiserror** : utilisez `#[derive(Error)]` pour generer les impls automatiquement

```rust
// Version anyhow
use anyhow::{Result, Context, bail};

fn lire_nombres_anyhow(chemin: &str) -> Result<Vec<i64>> {
    let contenu = std::fs::read_to_string(chemin)
        .context(format!("Impossible de lire '{}'", chemin))?;
    // A completer...
    todo!()
}

// Version thiserror
use thiserror::Error;

#[derive(Error, Debug)]
enum SommeError {
    // Utiliser #[error("...")] et #[from]
    // A completer...
}
```

---

## Liens

- [[03 - Structs Enums et Pattern Matching]] --- Les enums et le pattern matching, fondation de Result et Option
- [[05 - Collections et Iterateurs]] --- Iterateurs et methodes qui retournent des Result/Option
