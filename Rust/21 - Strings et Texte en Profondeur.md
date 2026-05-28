# 21 - Strings et Texte en Profondeur

> [!info] Prérequis
> Ce cours suppose que tu maîtrises [[02 - Ownership et Borrowing]] — les strings sont l'exemple canonique du système d'ownership. Les patterns de lifetimes dans `&str` sont approfondis dans [[17 - Lifetimes Avances Pin et Unpin]]. Les itérateurs sur les strings font appel à [[05 - Collections et Iterateurs]].

Les strings en Rust sont souvent perçues comme compliquées. En réalité, leur conception est rigoureuse et répond à des contraintes précises : correctness UTF-8 garantie, zero-copy quand possible, et intégration propre avec le système d'ownership. Ce cours démonte ces types de fond en comble.

---

## 1. `String` vs `&str` — la distinction fondamentale

### Anatomie interne

```
String (owned, heap-allocated)
┌──────────────────────────────────────┐
│ ptr → [ h | e | l | l | o ]  (heap) │
│ len = 5                              │
│ capacity = 8                         │
└──────────────────────────────────────┘

&str (borrowed slice)
┌──────────────────────────────────────┐
│ ptr → pointe vers n'importe où       │
│ len = 5                              │
└──────────────────────────────────────┘
         ↓
   peut pointer vers :
   - la heap (slice d'un String)
   - le segment .rodata (string literal)
   - la stack (rare, via str en array)
```

```rust
// String — owned, heap-allocated, growable
let mut s: String = String::from("hello");
s.push_str(", world");           // alloue si nécessaire
println!("{}", s.capacity());    // peut être > len()

// &str — borrowed, zero-copy
let slice: &str = &s[0..5];     // emprunte une portion de s
let literal: &str = "hello";    // pointe vers le segment .rodata du binaire
let static_str: &'static str = "je vis pour toujours"; // lifetime statique explicite
```

> [!info] `String` est analogue à `Vec<u8>`
> En interne, `String` est défini exactement comme `struct String(Vec<u8>)` avec l'invariant que les bytes forment une séquence UTF-8 valide. De même, `&str` est analogue à `&[u8]` — un fat pointer (pointeur + longueur) — avec la même garantie UTF-8.

### Pourquoi cette distinction existe

```rust
// SANS la distinction (style C/Java) : copies inutiles partout
fn afficher_nom_java(nom: String) {
    // Java copie la string à chaque appel — coût invisible
    println!("{}", nom);
}

// AVEC la distinction Rust : zero-copy pour les lectures
fn afficher_nom(nom: &str) {
    // Emprunte juste un slice — pas d'allocation, pas de copie
    println!("{}", nom);
}

fn main() {
    let owned: String = String::from("Alice");
    let literal: &str = "Bob";

    afficher_nom(&owned);    // String → &str via deref coercion, zéro copie
    afficher_nom(literal);   // &str direct
    afficher_nom("Charlie"); // &'static str → &str, zéro copie
}
```

### Conversions dans tous les sens

```rust
// &str → String (allocation obligatoire)
let s1: String = "hello".to_string();          // via Display
let s2: String = "hello".to_owned();           // sémantiquement "je prends possession"
let s3: String = String::from("hello");        // idiomatique
let s4: String = format!("{}", "hello");       // via macro format

// String → &str (emprunt, pas d'allocation)
let owned = String::from("hello");
let slice1: &str = &owned;                     // deref coercion
let slice2: &str = owned.as_str();             // explicite
let slice3: &str = &owned[..];                 // slice complète

// Accès à une portion
let slice4: &str = &owned[1..4];              // "ell" — bytes 1, 2, 3
```

---

## 2. Deref coercion — pourquoi `&String` se comporte comme `&str`

```rust
use std::ops::Deref;

// String implémente Deref<Target = str>
// Rust applique automatiquement &String → &str quand nécessaire

fn longueur(s: &str) -> usize {
    s.len()
}

let owned = String::from("hello");
longueur(&owned);         // &String → &str automatiquement (deref coercion)
longueur("world");        // &str direct
longueur(&owned[1..3]);   // &str (slice)
```

> [!tip] Convention universelle
> Pour toute fonction qui lit une string sans la modifier ni en prendre possession : **toujours utiliser `&str`**. Cela accepte automatiquement `&String`, `&"literal"`, et tout ce qui peut être dereffé en `str`. Ne jamais écrire `fn f(s: &String)` sauf cas très spécifiques.

### `AsRef<str>` — encore plus générique

```rust
use std::convert::AsRef;

// Accepte String, &str, &String, et tout type qui implémente AsRef<str>
fn traiter<S: AsRef<str>>(s: S) {
    let slice: &str = s.as_ref();
    println!("Longueur : {}", slice.len());
}

traiter("literal");
traiter(String::from("owned"));
traiter(&String::from("ref to owned"));

// Cas d'usage : std::path::Path implémente aussi AsRef<str> (si valid UTF-8)
use std::path::Path;
let chemin = Path::new("/etc/hosts");
// traiter(chemin);  // compilerait si Path: AsRef<str>, mais ce n'est pas le cas
```

### `impl Into<String>` vs `impl AsRef<str>`

```rust
// AsRef<str> : lecture seule, zero-copy — préférer pour les fonctions de lecture
fn afficher(s: impl AsRef<str>) {
    println!("{}", s.as_ref());
}

// Into<String> : conversion avec possession possible — pour les fonctions qui stockent
struct Cache {
    donnees: String,
}

impl Cache {
    // Accepte &str, String, &String — converti en String si nécessaire
    fn nouveau(valeur: impl Into<String>) -> Self {
        Self { donnees: valeur.into() }
    }
}

Cache::nouveau("literal");                // &str → String (allocation)
Cache::nouveau(String::from("owned"));   // String → String (move, pas d'allocation)
```

---

## 3. Encodage UTF-8 — la réalité des caractères

### Bytes, codepoints, graphème clusters

```rust
let emoji = "héllo 🦀";

// BYTES : taille en mémoire
println!("bytes : {}", emoji.len());           // 12 (é=2 bytes, 🦀=4 bytes)

// CHARS : codepoints Unicode (U+XXXX)
println!("chars : {}", emoji.chars().count()); // 8 (h, é, l, l, o, espace, 🦀 = 7 codepoints + mais é est 1)

// Pour les graphème clusters (perçus comme "1 caractère" visuellement)
// nécessite la crate unicode-segmentation
// use unicode_segmentation::UnicodeSegmentation;
// emoji.graphemes(true).count()  // compte les "caractères" visibles
```

```rust
// Itération correcte
let texte = "café";

// PAR BYTES (u8) — valeurs brutes, peut couper des codepoints
for b in texte.bytes() {
    print!("{:02x} ", b);  // 63 61 66 c3 a9
}

// PAR CHARS (char) — codepoints Unicode, toujours valides
for c in texte.chars() {
    println!("'{}' (U+{:04X})", c, c as u32);
}
// 'c' (U+0063)
// 'a' (U+0061)
// 'f' (U+0066)
// 'é' (U+00E9)

// AVEC INDICES — byte index + char
for (byte_index, caractere) in texte.char_indices() {
    println!("byte[{}] = '{}'", byte_index, caractere);
}
```

> [!warning] L'indexation directe est interdite pour une bonne raison
> `texte[1]` est une erreur de compilation en Rust. `texte[0..2]` est valide à la compilation mais **panique à l'exécution** si les bytes 0..2 ne forment pas un codepoint UTF-8 complet. Utiliser `.char_indices()` ou `.chars().nth(n)` pour un accès sûr.

### Validation UTF-8 à la frontière

```rust
// Convertir des bytes en &str — vérifie l'UTF-8
let bytes: &[u8] = b"hello";
let s: Result<&str, std::str::Utf8Error> = std::str::from_utf8(bytes);
let s = s.expect("bytes invalides");

// Version owned
let bytes: Vec<u8> = vec![104, 101, 108, 108, 111];
let s: Result<String, std::string::FromUtf8Error> = String::from_utf8(bytes);

// Version "infaillible" avec remplacement des bytes invalides par U+FFFD (replacement character)
let bytes: Vec<u8> = vec![0xff, 104, 101, 108];
let s = String::from_utf8_lossy(&bytes);  // Retourne Cow<str>
println!("{}", s);  // "hello" avec un caractère de remplacement au début

// UNSAFE : skip the validation (dangereux — uniquement si tu garantis l'UTF-8)
// let s = unsafe { std::str::from_utf8_unchecked(bytes) };
```

---

## 4. Opérations sur les strings

### Itération et découpe

```rust
let texte = "  hello,world,foo  \n  bar  ";

// Lignes
for ligne in texte.lines() {
    println!("'{}'", ligne);
    // "  hello,world,foo  "
    // "  bar  "
}

// Split standard
let parties: Vec<&str> = "a,b,c,d".split(',').collect();  // ["a", "b", "c", "d"]

// Split avec limite
let parties: Vec<&str> = "a,b,c,d".splitn(3, ',').collect();  // ["a", "b", "c,d"]

// Split depuis la fin
let parties: Vec<&str> = "a,b,c,d".rsplitn(2, ',').collect();  // ["d", "a,b,c"]

// Split une fois
if let Some((gauche, droite)) = "cle=valeur".split_once('=') {
    println!("clé: {}, valeur: {}", gauche, droite);  // clé: cle, valeur: valeur
}

// Split sur une condition
let mots: Vec<&str> = "hello world\tfoo\nbar".split_whitespace().collect();
// ["hello", "world", "foo", "bar"]

// Split sur plusieurs patterns
let parties: Vec<&str> = "un,deux;trois|quatre".split(&[',', ';', '|'][..]).collect();
```

### Trim et nettoyage

```rust
let s = "  \t hello \n  ";

println!("'{}'", s.trim());         // 'hello'
println!("'{}'", s.trim_start());   // 'hello \n  '
println!("'{}'", s.trim_end());     // '  \t hello'

// Trim sur des caractères spécifiques
let s = "###hello###";
println!("{}", s.trim_matches('#'));          // "hello"
println!("{}", s.trim_start_matches('#'));    // "hello###"
println!("{}", s.trim_end_matches('#'));      // "###hello"

// Trim sur une condition
let s = "12345abc678";
let s = s.trim_matches(|c: char| c.is_numeric());  // "abc"
```

### Recherche et remplacement

```rust
let texte = "le chat noir et le chat blanc";

// Recherche
println!("{}", texte.contains("chat"));          // true
println!("{}", texte.starts_with("le"));         // true
println!("{}", texte.ends_with("blanc"));        // true

// Position (byte index)
println!("{:?}", texte.find("chat"));            // Some(3)
println!("{:?}", texte.rfind("chat"));           // Some(19)

// Toutes les occurrences
for (i, _) in texte.match_indices("chat") {
    println!("Trouvé à l'index {}", i);          // 3, puis 19
}

// Remplacement
let modifie = texte.replace("chat", "chien");
println!("{}", modifie);  // "le chien noir et le chien blanc"

let modifie = texte.replacen("chat", "chien", 1);
println!("{}", modifie);  // "le chien noir et le chat blanc"

// Casse
println!("{}", "hElLo".to_lowercase());   // "hello"
println!("{}", "hElLo".to_uppercase());   // "HELLO"
```

### Parsing avec `FromStr`

```rust
use std::str::FromStr;
use std::num::ParseIntError;

// Utilisation de parse() — retourne Result<T, T::Err>
let n: i32 = "42".parse().unwrap();
let n: i32 = "42".parse::<i32>().unwrap();  // turbofish si le type n'est pas inféré
let r: Result<i32, _> = "abc".parse::<i32>();  // Err(...)

// Implémenter FromStr pour ses propres types
#[derive(Debug, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}

impl FromStr for Point {
    type Err = String;

    fn from_str(s: &str) -> Result<Self, Self::Err> {
        // Attend le format "x,y"
        let parties: Vec<&str> = s.split(',').collect();
        if parties.len() != 2 {
            return Err(format!("format invalide : '{}' (attendu 'x,y')", s));
        }

        let x = parties[0].trim().parse::<f64>()
            .map_err(|e| format!("x invalide : {}", e))?;
        let y = parties[1].trim().parse::<f64>()
            .map_err(|e| format!("y invalide : {}", e))?;

        Ok(Point { x, y })
    }
}

let p: Point = "3.14, 2.71".parse().unwrap();
assert_eq!(p, Point { x: 3.14, y: 2.71 });
```

---

## 5. Construction efficace de strings

### `String::with_capacity` — éviter les réallocations

```rust
// MAL : Vec/String se réallouent à chaque push (factor 2 — log2(n) réallocations)
let mut s = String::new();
for i in 0..100 {
    s.push_str(&i.to_string());  // peut réallouer plusieurs fois
}

// BIEN : réserver la capacité dès le départ
let mut s = String::with_capacity(300);  // estimation de la taille finale
for i in 0..100 {
    s.push_str(&i.to_string());  // pas de réallocation si estimation juste
    s.push(' ');
}

// Vérifier
println!("len={}, capacity={}", s.len(), s.capacity());
```

### Les différentes façons de construire

```rust
// format! — le plus lisible, toujours alloue
let s = format!("Bonjour {} ! Tu as {} ans.", nom, age);

// push_str / push — efficace en boucle
let mut s = String::with_capacity(100);
s.push_str("Hello");
s.push(',');
s.push_str(" world");
s.push('!');

// Opérateur + (consomme le premier argument)
let s1 = String::from("Hello");
let s2 = String::from(" world");
let s3 = s1 + &s2;  // s1 est déplacé (move), s2 est emprunté
// s1 n'est plus utilisable après cette ligne

// Concaténation efficace via collect
let parties = vec!["a", "b", "c", "d"];
let s: String = parties.join(", ");              // "a, b, c, d"
let s: String = parties.iter().cloned().collect::<Vec<_>>().join("-"); // "a-b-c-d"
let s: String = parties.into_iter().collect();   // "abcd" (concat directe)

// write! sur String — sans allocation intermédiaire
use std::fmt::Write;
let mut s = String::with_capacity(50);
write!(s, "valeur = {:.2}", 3.14159).unwrap();
writeln!(s, ", status = {}", "ok").unwrap();
```

> [!tip] `write!` vs `format!`
> `format!("val={:.2}", x)` crée toujours une nouvelle `String`. `write!(&mut buf, "val={:.2}", x)` écrit dans un buffer existant sans allocation intermédiaire. En boucle sur des milliers d'itérations, la différence est significative.

---

## 6. `Cow<'a, str>` — Clone on Write

```rust
use std::borrow::Cow;

// Cow<'a, str> est un enum :
// - Borrowed(&'a str) : aucune allocation
// - Owned(String) : alloue si nécessaire

fn nettoyer_espaces(s: &str) -> Cow<str> {
    if s.contains("  ") {
        // Nécessite une modification : on alloue un String
        let nettoye = s.split_whitespace().collect::<Vec<_>>().join(" ");
        Cow::Owned(nettoye)
    } else {
        // Pas de modification : on emprunte la string originale
        Cow::Borrowed(s)
    }
}

let s1 = "hello world";          // pas d'espaces doubles
let s2 = "hello  world";         // espaces doubles

let r1 = nettoyer_espaces(s1);  // Borrowed — zéro allocation
let r2 = nettoyer_espaces(s2);  // Owned — allocation nécessaire

println!("{}", r1);
println!("{}", r2);

// Cow s'utilise comme &str (implémente Deref<Target=str>)
assert_eq!(r1.len(), 11);
assert_eq!(r2.len(), 11);
```

```rust
use std::borrow::Cow;

// Cas d'usage classique : builder une string conditionnellement
fn preparer_url<'a>(base: &'a str, chemin: &str) -> Cow<'a, str> {
    if chemin.is_empty() {
        Cow::Borrowed(base)  // pas de modification, emprunter
    } else {
        Cow::Owned(format!("{}/{}", base, chemin))  // construire
    }
}

// Conversion
let cow: Cow<str> = Cow::Borrowed("hello");
let owned: String = cow.into_owned();   // clone si Borrowed, move si Owned

let mut cow: Cow<str> = Cow::Borrowed("hello");
let mutable: &mut String = cow.to_mut();  // convertit en Owned si nécessaire
mutable.push_str(" world");
```

---

## 7. Autres types de strings — la famille complète

### `OsString` / `OsStr` — strings système

```rust
use std::ffi::{OsStr, OsString};
use std::path::Path;

// OsString peut représenter des séquences d'octets non-UTF-8 (sur Linux)
// Sur Windows, c'est du WTF-16 (UTF-16 étendu)
let os_str: &OsStr = OsStr::new("hello");
let os_string: OsString = OsString::from("hello");

// Conversion vers &str (peut échouer si non-UTF-8)
if let Some(s) = os_str.to_str() {
    println!("{}", s);
}

// Conversion infaillible (lossy)
let s = os_str.to_string_lossy();  // Cow<str>

// Principaux cas d'usage : arguments de programme, variables d'environnement
use std::env;
for (cle, valeur) in env::vars_os() {
    // cle: OsString, valeur: OsString
    let cle_str = cle.to_string_lossy();
    println!("{} = ...", cle_str);
}
```

### `CString` / `CStr` — interop C

```rust
use std::ffi::{CStr, CString};

// CString : string null-terminated pour FFI C
// Important : alloue, possède sa mémoire, s'assure qu'il n'y a pas de \0 intérieur
let c_str = CString::new("hello").expect("pas de null intérieur autorisé");

// Passer à une fonction C (unsafe car FFI)
extern "C" {
    fn puts(s: *const std::os::raw::c_char) -> std::os::raw::c_int;
}
unsafe {
    puts(c_str.as_ptr());
}

// CStr : slice empruntée, null-terminated (analogue à &str pour CString)
let c_str_ref: &CStr = c_str.as_c_str();

// Convertir un CStr reçu de C en &str
let resultat_de_c: &CStr = unsafe { CStr::from_ptr(c_str.as_ptr()) };
if let Ok(s) = resultat_de_c.to_str() {
    println!("Reçu de C : {}", s);
}
```

> [!warning] Safety de CStr
> `CStr::from_ptr()` est `unsafe` car le pointeur doit rester valide pour toute la durée de vie du `CStr` créé, et pointer vers une séquence de bytes terminée par `\0`. Voir [[11 - Unsafe Rust et FFI]] pour les patterns FFI complets.

### `PathBuf` / `Path` — chemins de fichiers

```rust
use std::path::{Path, PathBuf};

// Path : slice empruntée (analogue à &str)
// PathBuf : owned (analogue à String)

let chemin = Path::new("/etc/nginx/nginx.conf");
println!("{:?}", chemin.extension());      // Some("conf")
println!("{:?}", chemin.file_name());      // Some("nginx.conf")
println!("{:?}", chemin.file_stem());      // Some("nginx")
println!("{:?}", chemin.parent());         // Some("/etc/nginx")

// Construction de chemins
let mut pb = PathBuf::from("/home/erwan");
pb.push("documents");
pb.push("fichier.txt");
println!("{}", pb.display());              // /home/erwan/documents/fichier.txt

// join retourne un PathBuf
let chemin = Path::new("/home/erwan").join("documents/fichier.txt");

// Conversion
if let Some(s) = chemin.to_str() {       // &str (peut échouer si non-UTF-8)
    println!("{}", s);
}
let s = chemin.display().to_string();    // toujours une String (lossy)
```

### `Rc<str>` et `Arc<str>` — strings partagées

```rust
use std::rc::Rc;
use std::sync::Arc;

// Arc<str> : string immuable partagée entre threads — moins lourd que Arc<String>
// String : 24 bytes (ptr + len + cap)
// Arc<str> : 16 bytes (ptr + len) — pas de champ capacity
let shared: Arc<str> = Arc::from("hello world");
let clone1 = Arc::clone(&shared);
let clone2 = Arc::clone(&shared);
// Les trois pointent vers le même buffer — zéro copie

// Conversion depuis &str et String
let from_literal: Arc<str> = "hello".into();
let from_string: Arc<str> = String::from("hello").into();

// Utilisation en cache ou en structure de données immuable partagée
use std::collections::HashMap;
let mut cache: HashMap<u64, Arc<str>> = HashMap::new();
cache.insert(1, Arc::from("valeur_longue_répétée_souvent"));
```

---

## 8. Formatage avancé

### Spécifications de format complètes

```rust
// Syntaxe : {[argument][':'[[fill]align][sign][#]['0'][width]['.' precision][type]]}

// Alignement
println!("{:>10}",  "droite");    // "    droite"  (droite)
println!("{:<10}",  "gauche");    // "gauche    "  (gauche)
println!("{:^10}",  "centre");    // "  centre  "  (centré)
println!("{:*>10}", "rempli");    // "****rempli"  (remplissage avec *)
println!("{:0>5}",  42);          // "00042"        (padding zéros)

// Nombres
println!("{:b}",   42);           // "101010"       (binaire)
println!("{:o}",   42);           // "52"           (octal)
println!("{:x}",   255);          // "ff"           (hexadécimal minuscule)
println!("{:X}",   255);          // "FF"           (hexadécimal majuscule)
println!("{:#x}",  255);          // "0xff"         (avec préfixe)
println!("{:#010x}", 255);        // "0x000000ff"   (préfixe + padding)
println!("{:.3}",  3.14159);      // "3.142"        (précision)
println!("{:+}",   42);           // "+42"          (signe explicite)
println!("{:e}",   1234567.0);    // "1.234567e6"   (notation scientifique)

// Arguments nommés et positionnels
let (nom, age) = ("Alice", 30);
println!("{nom} a {age} ans");         // Rust 1.58+ : noms de variables directs
println!("{0} et {1} et {0}", "a", "b"); // "a et b et a" (positionnels)
let pi = std::f64::consts::PI;
println!("{pi:.5}");                   // "3.14159" (variable + précision)

// Debug vs Display
#[derive(Debug)]
struct Point { x: f64, y: f64 }

let p = Point { x: 1.0, y: 2.0 };
println!("{:?}",  p);   // Point { x: 1.0, y: 2.0 }
println!("{:#?}", p);   // pretty-print multi-lignes
```

### Implémenter `Display` et `Debug`

```rust
use std::fmt;

struct Couleur {
    r: u8,
    g: u8,
    b: u8,
}

// Display : sortie lisible pour l'utilisateur final
impl fmt::Display for Couleur {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "#{:02X}{:02X}{:02X}", self.r, self.g, self.b)
    }
}

// Debug : sortie pour le développeur (dérivable en général)
impl fmt::Debug for Couleur {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        // f.debug_struct facilite le formatage
        f.debug_struct("Couleur")
            .field("r", &self.r)
            .field("g", &self.g)
            .field("b", &self.b)
            .finish()
    }
}

// Debug manuel utile pour les types avec données sensibles
struct MotDePasse(String);

impl fmt::Debug for MotDePasse {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        f.write_str("MotDePasse(***)")  // ne jamais afficher le vrai mot de passe
    }
}

let rouge = Couleur { r: 255, g: 0, b: 0 };
println!("{}", rouge);    // #FF0000
println!("{:?}", rouge);  // Couleur { r: 255, g: 0, b: 0 }
```

### `fmt::Write` — écrire dans un buffer sans allocation

```rust
use std::fmt::Write;

fn construire_csv(donnees: &[(String, i32)]) -> String {
    let capacite_estimee = donnees.len() * 30;
    let mut buffer = String::with_capacity(capacite_estimee);

    writeln!(buffer, "nom,valeur").unwrap();  // write! dans un String ne peut pas échouer
    for (nom, valeur) in donnees {
        writeln!(buffer, "{},{}", nom, valeur).unwrap();
    }

    buffer
}

// Le trait fmt::Write est aussi implémenté par les types custom
struct LimitedWriter {
    buf: String,
    max: usize,
}

impl fmt::Write for LimitedWriter {
    fn write_str(&mut self, s: &str) -> fmt::Result {
        if self.buf.len() + s.len() <= self.max {
            self.buf.push_str(s);
            Ok(())
        } else {
            Err(fmt::Error)  // erreur si dépassement
        }
    }
}
```

---

## 9. Expressions régulières avec `regex`

```toml
[dependencies]
regex = "1"
once_cell = "1"   # pour les regex globales
```

```rust
use regex::Regex;

// Compilation d'une regex (peut échouer si le pattern est invalide)
let re = Regex::new(r"(\d{4})-(\d{2})-(\d{2})").unwrap();

// is_match
assert!(re.is_match("2024-01-15"));

// find — première occurrence
if let Some(m) = re.find("Date : 2024-01-15") {
    println!("Trouvé : {} (positions {}-{})", m.as_str(), m.start(), m.end());
}

// captures — groupes de capture
if let Some(caps) = re.captures("Date : 2024-01-15") {
    println!("Année : {}", &caps[1]);    // "2024"
    println!("Mois : {}", &caps[2]);     // "01"
    println!("Jour : {}", &caps[3]);     // "15"
}

// captures_iter — toutes les occurrences
let texte = "2024-01-15 et 2024-12-31";
for caps in re.captures_iter(texte) {
    println!("Date : {}-{}-{}", &caps[1], &caps[2], &caps[3]);
}

// Captures nommées
let re_nommee = Regex::new(r"(?P<annee>\d{4})-(?P<mois>\d{2})-(?P<jour>\d{2})").unwrap();
if let Some(caps) = re_nommee.captures("2024-01-15") {
    println!("Année : {}", caps.name("annee").unwrap().as_str());
}

// Remplacement
let resultat = re.replace_all(texte, "DATE_CACHÉE");
println!("{}", resultat);

// Bytes (pour les données non-UTF-8)
use regex::bytes::Regex as BytesRegex;
let re_bytes = BytesRegex::new(r"\xFF\xFE").unwrap();
```

### Regex globales avec `once_cell` ou `std::sync::LazyLock`

```rust
// Compiler une regex une seule fois (Rust < 1.80)
use once_cell::sync::Lazy;
use regex::Regex;

static RE_EMAIL: Lazy<Regex> = Lazy::new(|| {
    Regex::new(r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}").unwrap()
});

fn est_email_valide(s: &str) -> bool {
    RE_EMAIL.is_match(s)
}

// Rust 1.80+ : std::sync::LazyLock (sans dépendance externe)
use std::sync::LazyLock;

static RE_URL: LazyLock<Regex> = LazyLock::new(|| {
    Regex::new(r"https?://[^\s]+").unwrap()
});
```

> [!warning] Ne jamais compiler une regex dans une boucle
> `Regex::new(...)` compile le pattern en un automate fini — c'est coûteux (~microsecondes à millisecondes). Toujours stocker la `Regex` dans une `static Lazy` ou la passer en paramètre. Une regex compilée dans une boucle de 1M itérations peut facilement coûter 10x plus que la logique utile.

---

## 10. Parsing avancé — `nom`, `pest`, `logos`

### `nom` — parser combinator

```toml
[dependencies]
nom = "7"
```

```rust
use nom::{
    IResult,
    bytes::complete::{tag, take_while1},
    character::complete::{digit1, multispace0},
    combinator::map_res,
    sequence::{preceded, separated_pair},
    branch::alt,
};

// Parser un entier
fn entier(input: &str) -> IResult<&str, i64> {
    map_res(digit1, str::parse)(input)
}

// Parser une paire clé=valeur
fn paire_cle_valeur(input: &str) -> IResult<&str, (&str, i64)> {
    separated_pair(
        take_while1(|c: char| c.is_alphabetic()),  // clé : lettres
        tag("="),
        entier,                                    // valeur : entier
    )(input)
}

// Test
let (reste, (cle, valeur)) = paire_cle_valeur("timeout=30").unwrap();
assert_eq!(cle, "timeout");
assert_eq!(valeur, 30);
assert_eq!(reste, "");
```

### `logos` — lexer rapide par derive macro

```toml
[dependencies]
logos = "0.14"
```

```rust
use logos::Logos;

#[derive(Logos, Debug, PartialEq)]
enum Token {
    #[token("fn")]
    Fn,

    #[token("let")]
    Let,

    #[regex("[a-zA-Z_][a-zA-Z0-9_]*")]
    Identifiant,

    #[regex("[0-9]+", |lex| lex.slice().parse::<i64>().ok())]
    Entier(i64),

    #[token("=")]
    Egal,

    #[token(";")]
    PtVirgule,

    #[regex(r"[ \t\n\f]+", logos::skip)]  // ignorer les espaces
    #[error]
    Erreur,
}

let code = "let x = 42;";
let tokens: Vec<_> = Token::lexer(code)
    .spanned()
    .collect();

for (token, span) in &tokens {
    println!("{:?} at {:?}", token, span);
}
```

---

## 11. Cas pratiques — recettes courantes

### Construire une table ASCII alignée

```rust
fn afficher_table(donnees: &[(&str, i32, f64)]) -> String {
    use std::fmt::Write;
    let mut buf = String::with_capacity(donnees.len() * 40);

    writeln!(buf, "{:<20} {:>8} {:>10}", "Nom", "Quantité", "Prix").unwrap();
    writeln!(buf, "{}", "-".repeat(40)).unwrap();

    for (nom, qte, prix) in donnees {
        writeln!(buf, "{:<20} {:>8} {:>10.2}€", nom, qte, prix).unwrap();
    }

    buf
}

let donnees = vec![
    ("Pomme", 42, 1.5),
    ("Banane", 100, 0.75),
    ("Fraise", 5, 3.99),
];

println!("{}", afficher_table(&donnees));
```

### Parser un fichier de configuration simple

```rust
use std::collections::HashMap;
use std::str::FromStr;

#[derive(Debug)]
struct Config(HashMap<String, String>);

impl FromStr for Config {
    type Err = String;

    fn from_str(s: &str) -> Result<Self, Self::Err> {
        let mut map = HashMap::new();

        for (numero_ligne, ligne) in s.lines().enumerate() {
            let ligne = ligne.trim();

            // Ignorer commentaires et lignes vides
            if ligne.is_empty() || ligne.starts_with('#') {
                continue;
            }

            let (cle, valeur) = ligne.split_once('=').ok_or_else(|| {
                format!("ligne {} : séparateur '=' manquant", numero_ligne + 1)
            })?;

            map.insert(cle.trim().to_string(), valeur.trim().to_string());
        }

        Ok(Config(map))
    }
}

let config_str = "
    # Configuration
    host = localhost
    port = 8080
    debug = true
";

let config: Config = config_str.parse().unwrap();
println!("{:?}", config.0.get("port"));  // Some("8080")
```

---

## Exercices pratiques

### Exercice 1 — Analyse de strings UTF-8

Écris une fonction `analyser_texte(s: &str) -> StatistiquesTexte` qui retourne :
- Nombre de bytes
- Nombre de codepoints Unicode (chars)
- Nombre de mots (séparés par des espaces)
- Nombre de lignes
- Le caractère le plus fréquent

Teste avec un texte contenant des emojis et des caractères accentués.

### Exercice 2 — Cow intelligent

Implémente `normaliser_identifiant(s: &str) -> Cow<str>` qui :
- Retourne `Cow::Borrowed(s)` si `s` est déjà valide (lowercase ASCII, tirets uniquement)
- Retourne `Cow::Owned(...)` en remplaçant les espaces par des tirets et en mettant en lowercase

Teste que les strings déjà valides ne font aucune allocation.

### Exercice 3 — Mini-template engine

Implémente `interpoler(template: &str, variables: &HashMap<&str, &str>) -> String` qui remplace `{nom}` par la valeur correspondante dans la map. Utilise `String::with_capacity` et `fmt::Write`. Voir [[18 - Closures FnOnce FnMut Fn en Profondeur]] pour les patterns de callback si tu veux rendre le remplacement extensible.

### Exercice 4 — Parseur CSV robuste avec `nom`

Implémente un parseur CSV avec `nom` qui gère :
- Les champs entre guillemets avec des virgules à l'intérieur
- Les guillemets échappés (`""` dans un champ guillemété)
- Les fins de ligne `\r\n` et `\n`

Lis [[16 - Serde Serialisation et Deserialisation]] pour comparer avec le crate `csv` existant.

### Exercice 5 — Formatage personnalisé

Crée un type `Bytes(usize)` qui implémente `Display` pour afficher :
- < 1024 : `"123 B"`
- < 1024² : `"12.3 KB"`
- < 1024³ : `"1.23 MB"`
- etc.

Teste avec `format!("{}", Bytes(1_500_000))` → `"1.43 MB"`.

---

## Notes liées

### Ownership et types fondamentaux
- [[02 - Ownership et Borrowing]] — `String` vs `&str` comme exemple canonique de l'ownership
- [[17 - Lifetimes Avances Pin et Unpin]] — lifetimes de `&str`, `Cow<'a, str>`, `impl Trait`
- [[05 - Collections et Iterateurs]] — itérateurs `.chars()`, `.lines()`, `.split()` chaînés

### Sérialisation et parsing
- [[16 - Serde Serialisation et Deserialisation]] — `FromStr`, `Display`, intégration serde avec strings
- [[07 - Projet Rust CLI]] — parsing d'arguments texte, formatage de la sortie

### Interop et unsafe
- [[11 - Unsafe Rust et FFI]] — `CString`/`CStr` pour l'interop C, strings non-UTF-8

### Closures et génériques
- [[18 - Closures FnOnce FnMut Fn en Profondeur]] — closures dans `.split()`, `.map()`, `.filter()` sur strings
- [[06 - Traits et Generiques]] — `AsRef<str>`, `Into<String>`, `Display`, `FromStr`
