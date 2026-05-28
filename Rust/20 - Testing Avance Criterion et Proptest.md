# 20 - Testing Avancé : Criterion et Proptest

> [!info] Prérequis
> Ce cours suppose que tu connais les bases des tests Rust. Il fait référence à [[04 - Gestion des Erreurs en Rust]] pour les `Result` dans les tests, à [[18 - Closures FnOnce FnMut Fn en Profondeur]] pour les closures utilisées par les frameworks de test, et à [[19 - Cargo Avance Workspaces et Publication]] pour les `dev-dependencies` et les profiles de bench.

Rust possède un écosystème de testing remarquablement riche : tests unitaires intégrés au compilateur, tests d'intégration, doctests auto-exécutés, benchmarks statistiques avec Criterion, property-based testing avec Proptest, fuzzing avec cargo-fuzz, et snapshot testing avec Insta. Ce cours couvre l'ensemble de cette chaîne.

---

## 1. Tests unitaires — approfondissement

### Structure de base

```rust
// src/lib.rs
pub fn additionner(a: i32, b: i32) -> i32 {
    a + b
}

pub fn diviser(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None
    } else {
        Some(a / b)
    }
}

// Convention : module tests dans le même fichier
#[cfg(test)]           // compilé uniquement en mode test
mod tests {
    use super::*;      // importer tout ce qui est dans le module parent

    #[test]
    fn test_additionner_positifs() {
        assert_eq!(additionner(2, 3), 5);
    }

    #[test]
    fn test_additionner_negatifs() {
        assert_eq!(additionner(-2, -3), -5);
    }

    #[test]
    fn test_diviser_normal() {
        let resultat = diviser(10.0, 2.0);
        assert_eq!(resultat, Some(5.0));
    }

    #[test]
    fn test_diviser_par_zero() {
        assert_eq!(diviser(10.0, 0.0), None);
    }
}
```

### Assertions complètes

```rust
#[test]
fn demo_assertions() {
    let valeur = 42i32;
    let texte = "hello world";
    let liste = vec![1, 2, 3];

    // Assertions de base
    assert!(valeur > 0);
    assert_eq!(valeur, 42);
    assert_ne!(valeur, 0);

    // Messages personnalisés (format identique à format!())
    assert!(valeur > 0, "valeur doit être positive, reçu : {}", valeur);
    assert_eq!(valeur, 42, "attendu 42 mais reçu {} (contexte : {:?})", valeur, liste);

    // Vérifications sur les collections
    assert!(liste.contains(&2));
    assert_eq!(liste.len(), 3);
    assert!(texte.contains("world"));
    assert!(texte.starts_with("hello"));

    // Comparaisons flottantes (ne jamais utiliser assert_eq! avec des f64)
    let pi = std::f64::consts::PI;
    let approx = 3.14159;
    assert!((pi - approx).abs() < 0.001, "pi approximation trop imprécise : {}", approx);
}

// assert_matches! (stabilisé en Rust 1.82)
#[test]
fn demo_assert_matches() {
    use std::assert_matches::assert_matches;

    let resultat: Result<i32, &str> = Ok(42);
    assert_matches!(resultat, Ok(n) if n > 0);

    let option: Option<&str> = Some("hello");
    assert_matches!(option, Some(s) if s.starts_with('h'));
}
```

### Tests qui retournent `Result`

```rust
use std::num::ParseIntError;

fn parser_entier(s: &str) -> Result<i32, ParseIntError> {
    s.trim().parse()
}

#[test]
fn test_parser_valide() -> Result<(), ParseIntError> {
    // On peut utiliser ? dans les tests qui retournent Result
    let n = parser_entier("42")?;
    assert_eq!(n, 42);
    Ok(())
}

#[test]
fn test_parser_invalide() -> Result<(), Box<dyn std::error::Error>> {
    // Box<dyn Error> pour accepter n'importe quel type d'erreur
    let resultat = parser_entier("abc");
    assert!(resultat.is_err(), "parser 'abc' devrait échouer");
    Ok(())
}
```

> [!tip] Tests avec `Result` vs `assert!`
> Un test qui retourne `Result<(), E>` et se termine par `Ok(())` est idiomatique quand la logique testée retourne des `Result`. L'opérateur `?` propague les erreurs comme des échecs de test avec un message lisible, sans besoin de `.unwrap()` ou de `assert!(r.is_ok())`.

### `#[should_panic]`

```rust
pub fn indice_tableau(v: &[i32], i: usize) -> i32 {
    v[i]  // panique si i >= v.len()
}

#[test]
#[should_panic]
fn test_out_of_bounds() {
    indice_tableau(&[1, 2, 3], 10);
}

#[test]
#[should_panic(expected = "index out of bounds")]
fn test_out_of_bounds_message() {
    // Vérifie que le message de panique CONTIENT la chaîne expected
    indice_tableau(&[1, 2, 3], 10);
}

// Attention : should_panic réussit même si le code panique pour une mauvaise raison
// Toujours utiliser expected = "..." pour être précis
```

### `#[ignore]` et filtrage

```rust
#[test]
#[ignore = "trop lent pour le CI — lancer manuellement"]
fn test_integration_lent() {
    // teste quelque chose qui prend 30 secondes
}

#[test]
#[ignore]
fn test_necessite_hardware_special() {
    // nécessite une carte son
}
```

```bash
# Lancer uniquement les tests ignorés
cargo test -- --ignored

# Lancer tous les tests, y compris les ignorés
cargo test -- --include-ignored

# Filtrage par nom (tous les tests dont le nom contient "parser")
cargo test parser

# Filtrage par module
cargo test tests::unitaires::

# Lancer en séquentiel (nécessaire si les tests partagent un état global)
cargo test -- --test-threads=1

# Voir les println! même si les tests réussissent
cargo test -- --nocapture

# Combiner les options
cargo test -- --test-threads=1 --nocapture parser
```

---

## 2. Tests d'intégration

### Structure du dossier `tests/`

```
mon_crate/
├── src/
│   └── lib.rs
├── tests/
│   ├── integration_principale.rs   ← crate de test séparée
│   ├── integration_parseur.rs      ← autre crate de test séparée
│   └── common/
│       └── mod.rs                  ← code partagé entre les fichiers de test
```

> [!info] Chaque fichier dans `tests/` est un crate séparé
> Contrairement aux tests unitaires (dans `src/`), chaque fichier de `tests/` est compilé comme son propre crate, qui dépend du crate testé. Cela signifie que seules les APIs publiques sont testables depuis `tests/` — exactement comme un utilisateur externe.

### Code commun aux tests d'intégration

```rust
// tests/common/mod.rs (ou tests/common.rs pour l'édition 2018+)
pub fn initialiser_environnement() {
    // setup commun : créer des fichiers temporaires, initialiser la DB, etc.
    let _ = env_logger::try_init();
}

pub struct Fixture {
    pub dir: tempfile::TempDir,
}

impl Fixture {
    pub fn nouvelle() -> Self {
        Self {
            dir: tempfile::tempdir().expect("impossible de créer le dossier temporaire"),
        }
    }

    pub fn chemin_fichier(&self, nom: &str) -> std::path::PathBuf {
        self.dir.path().join(nom)
    }
}
```

```rust
// tests/integration_principale.rs
mod common;  // importer le module commun

use mon_crate::MaStruct;

#[test]
fn test_creation_fichier() {
    common::initialiser_environnement();
    let fixture = common::Fixture::nouvelle();
    let chemin = fixture.chemin_fichier("output.txt");

    MaStruct::ecrire_fichier(&chemin, "contenu").unwrap();

    assert!(chemin.exists());
    let contenu = std::fs::read_to_string(&chemin).unwrap();
    assert_eq!(contenu, "contenu");
}

// La fixture est automatiquement nettoyée quand elle sort du scope (TempDir::drop)
```

```bash
# Lancer uniquement les tests d'intégration d'un fichier
cargo test --test integration_principale

# Lancer tous les tests d'intégration (tous les fichiers de tests/)
cargo test --tests
```

### Unit tests vs Integration tests — quand utiliser quoi

| Critère | Unit tests | Integration tests |
|---------|-----------|-------------------|
| Scope | Une fonction / un module | API publique complète |
| Accès | API privée (`use super::*`) | API publique uniquement |
| Vitesse | Très rapides | Plus lents (I/O possible) |
| Localisation | `src/` | `tests/` |
| But | Logique interne, edge cases | Comportement utilisateur |

---

## 3. Doctests — documentation exécutable

### Syntaxe de base

```rust
/// Calcule la somme de deux entiers.
///
/// # Exemples
///
/// ```rust
/// use mon_crate::additionner;
///
/// assert_eq!(additionner(2, 3), 5);
/// assert_eq!(additionner(-1, 1), 0);
/// ```
pub fn additionner(a: i32, b: i32) -> i32 {
    a + b
}

/// Parse une chaîne en entier.
///
/// # Exemples
///
/// ```
/// # use mon_crate::parser_entier;
/// // La ligne commençant par # est cachée dans la doc mais exécutée
/// # // Cela permet d'avoir des imports sans polluer la documentation
/// let n = parser_entier("42").unwrap();
/// assert_eq!(n, 42);
/// ```
///
/// Retourne une erreur si la chaîne n'est pas un entier valide :
///
/// ```
/// # use mon_crate::parser_entier;
/// assert!(parser_entier("abc").is_err());
/// ```
pub fn parser_entier(s: &str) -> Result<i32, std::num::ParseIntError> {
    s.parse()
}
```

### Attributs de doctests

````rust
/// Cette fonction est en cours de développement.
///
/// ```rust,ignore
/// // Ce code ne sera pas compilé ni exécuté
/// ma_fonction_incomplète();
/// ```
///
/// Exemple qui compile mais ne s'exécute pas (I/O, réseau) :
///
/// ```rust,no_run
/// let contenu = std::fs::read_to_string("/chemin/vers/fichier").unwrap();
/// println!("{}", contenu);
/// ```
///
/// Exemple qui doit paniquer :
///
/// ```rust,should_panic
/// let v: Vec<i32> = vec![];
/// v[0]; // panique !
/// ```
pub fn exemple_attributs() {}
````

> [!tip] L'avantage majeur des doctests
> Les doctests garantissent que les exemples de documentation compilent toujours et correspondent à l'API actuelle. Un doctest cassé est une erreur de compilation — il est impossible d'avoir de la documentation mensongère. C'est un avantage énorme sur les README ou les commentaires narratifs.

```bash
# Lancer uniquement les doctests
cargo test --doc

# Tout à la fois
cargo test   # unit + integration + doctests
```

---

## 4. Mocking

### `mockall` — mocker automatiquement les traits

```toml
[dev-dependencies]
mockall = "0.12"
```

```rust
// src/lib.rs
use std::io;

// Le trait qu'on veut mocker
pub trait BaseDeDonnees {
    fn chercher_utilisateur(&self, id: u64) -> Result<String, io::Error>;
    fn sauvegarder(&mut self, id: u64, nom: &str) -> Result<(), io::Error>;
}

pub struct ServiceUtilisateur<DB: BaseDeDonnees> {
    db: DB,
}

impl<DB: BaseDeDonnees> ServiceUtilisateur<DB> {
    pub fn nouveau(db: DB) -> Self {
        Self { db }
    }

    pub fn obtenir_nom(&self, id: u64) -> String {
        match self.db.chercher_utilisateur(id) {
            Ok(nom) => nom,
            Err(_) => "Inconnu".to_string(),
        }
    }
}
```

```rust
// src/lib.rs ou tests/mock_tests.rs
#[cfg(test)]
mod tests {
    use super::*;
    use mockall::predicate::*;
    use mockall::mock;

    // Générer automatiquement un mock pour le trait
    mockall::mock! {
        BaseDeDonneesImpl {}

        impl BaseDeDonnees for BaseDeDonneesImpl {
            fn chercher_utilisateur(&self, id: u64) -> Result<String, std::io::Error>;
            fn sauvegarder(&mut self, id: u64, nom: &str) -> Result<(), std::io::Error>;
        }
    }

    #[test]
    fn test_obtenir_nom_existant() {
        let mut mock = MockBaseDeDonneesImpl::new();

        // Configurer les attentes : chercher_utilisateur(42) retourne "Alice"
        mock.expect_chercher_utilisateur()
            .with(eq(42u64))           // uniquement pour l'argument 42
            .times(1)                  // doit être appelé exactement 1 fois
            .returning(|_| Ok("Alice".to_string()));

        let service = ServiceUtilisateur::nouveau(mock);
        assert_eq!(service.obtenir_nom(42), "Alice");
        // Le mock vérifie automatiquement que toutes les attentes sont satisfaites (drop)
    }

    #[test]
    fn test_obtenir_nom_absent() {
        let mut mock = MockBaseDeDonneesImpl::new();

        mock.expect_chercher_utilisateur()
            .returning(|_| Err(std::io::Error::new(std::io::ErrorKind::NotFound, "not found")));

        let service = ServiceUtilisateur::nouveau(mock);
        assert_eq!(service.obtenir_nom(99), "Inconnu");
    }

    #[test]
    fn test_sauvegarder_appele() {
        let mut mock = MockBaseDeDonneesImpl::new();

        // Vérifier qu'une méthode est appelée avec des arguments spécifiques
        mock.expect_sauvegarder()
            .with(eq(1u64), eq("Bob"))
            .times(1)
            .returning(|_, _| Ok(()));

        mock.sauvegarder(1, "Bob").unwrap();
    }
}
```

### `#[automock]` — attribut de macro automatique

```rust
// Plus concis que mock! — directement sur le trait
#[cfg_attr(test, mockall::automock)]
pub trait EmailService {
    fn envoyer(&self, destinataire: &str, sujet: &str) -> Result<(), String>;
}

// Dans les tests : MockEmailService est disponible automatiquement
```

### `mockito` et `wiremock` pour HTTP

```toml
[dev-dependencies]
mockito = "1"           # serveur HTTP local pour mocker les appels réseau
wiremock = "0.6"        # alternative plus ergonomique
```

```rust
#[cfg(test)]
mod tests {
    use mockito::Server;

    #[tokio::test]
    async fn test_appel_api() {
        let mut server = Server::new_async().await;

        // Configurer le mock HTTP
        let mock = server.mock("GET", "/utilisateurs/1")
            .with_status(200)
            .with_header("content-type", "application/json")
            .with_body(r#"{"id": 1, "nom": "Alice"}"#)
            .create_async()
            .await;

        let url = server.url();
        let client = reqwest::Client::new();
        let reponse = client
            .get(format!("{}/utilisateurs/1", url))
            .send()
            .await
            .unwrap();

        assert_eq!(reponse.status(), 200);
        mock.assert_async().await;  // vérifie que le mock a été appelé
    }
}
```

---

## 5. Benchmarks avec Criterion

### Configuration

```toml
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

# Déclarer chaque fichier de benchmark
[[bench]]
name = "mes_benchmarks"
harness = false    # désactiver le harness de test par défaut (obligatoire avec Criterion)
```

```
mon_crate/
├── benches/
│   └── mes_benchmarks.rs   ← fichier de benchmark
├── src/
│   └── lib.rs
```

### Benchmark simple

```rust
// benches/mes_benchmarks.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use mon_crate::trier;

fn benchmark_tri(c: &mut Criterion) {
    // Données de test
    let mut donnees: Vec<i32> = (0..1000).rev().collect();

    c.bench_function("tri_1000_elements", |b| {
        b.iter(|| {
            let mut v = donnees.clone();
            // black_box empêche le compilateur d'optimiser le code benchmarké
            // (sans ça, il pourrait détecter que le résultat n'est pas utilisé et tout supprimer)
            trier(black_box(&mut v));
            black_box(v)
        })
    });
}

criterion_group!(benches, benchmark_tri);
criterion_main!(benches);
```

> [!warning] `black_box` est essentiel
> Sans `black_box()`, LLVM peut optimiser en supprimant entièrement le code dont le résultat n'est pas utilisé, rendant le benchmark inutile (temps ≈ 0). `black_box()` crée un barrière d'optimisation opaque au compilateur.

### Comparer des implémentations

```rust
use criterion::{black_box, criterion_group, criterion_main, BenchmarkId, Criterion};

fn fibonacci_recursif(n: u64) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci_recursif(n - 1) + fibonacci_recursif(n - 2),
    }
}

fn fibonacci_iteratif(n: u64) -> u64 {
    let mut a = 0u64;
    let mut b = 1u64;
    for _ in 0..n {
        let temp = a;
        a = b;
        b = temp + b;
    }
    a
}

fn benchmark_fibonacci(c: &mut Criterion) {
    let mut groupe = c.benchmark_group("fibonacci");

    // Tester plusieurs valeurs de n
    for n in [10u64, 15, 20].iter() {
        groupe.bench_with_input(BenchmarkId::new("récursif", n), n, |b, &n| {
            b.iter(|| fibonacci_recursif(black_box(n)))
        });

        groupe.bench_with_input(BenchmarkId::new("itératif", n), n, |b, &n| {
            b.iter(|| fibonacci_iteratif(black_box(n)))
        });
    }

    groupe.finish();
}

criterion_group!(benches, benchmark_fibonacci);
criterion_main!(benches);
```

### Configuration avancée de Criterion

```rust
use criterion::{criterion_group, criterion_main, Criterion};
use std::time::Duration;

fn benchmark_configure(c: &mut Criterion) {
    // Configurer les paramètres de mesure
    let mut groupe = c.benchmark_group("traitement_lent");

    groupe
        .sample_size(50)                         // nombre de mesures (défaut : 100)
        .measurement_time(Duration::from_secs(10)) // durée totale de mesure
        .warm_up_time(Duration::from_secs(3));   // phase de chauffe

    groupe.bench_function("operation_lente", |b| {
        b.iter(|| {
            // opération qui prend quelques millisecondes
        })
    });

    groupe.finish();
}

criterion_group!(benches, benchmark_configure);
criterion_main!(benches);
```

```bash
# Lancer les benchmarks
cargo bench

# Avec cargo-criterion (rapport HTML amélioré)
cargo install cargo-criterion
cargo criterion

# Lancer un benchmark spécifique
cargo bench -- fibonacci

# Sauvegarder la baseline pour comparaison future
cargo bench -- --save-baseline main

# Comparer avec la baseline sauvegardée
cargo bench -- --baseline main
```

> [!info] Rapport HTML
> Avec `features = ["html_reports"]`, Criterion génère un rapport HTML dans `target/criterion/` avec graphiques de distribution, historique et comparaisons. Ouvrir `target/criterion/report/index.html`.

---

## 6. Property-based testing avec Proptest

### Principe fondamental

Contrairement aux tests classiques (cas précis → résultat attendu), le property-based testing génère des milliers d'inputs aléatoires et vérifie des **propriétés invariantes** qui doivent toujours être vraies.

```toml
[dev-dependencies]
proptest = "1"
```

### Premier test avec Proptest

```rust
// Dans les tests unitaires ou d'intégration
use proptest::prelude::*;

fn trier_vec(mut v: Vec<i32>) -> Vec<i32> {
    v.sort();
    v
}

proptest! {
    // Proptest génère automatiquement des Vec<i32> aléatoires
    #[test]
    fn tri_idempotent(v in any::<Vec<i32>>()) {
        // Propriété : trier deux fois = trier une fois
        let une_fois = trier_vec(v.clone());
        let deux_fois = trier_vec(une_fois.clone());
        prop_assert_eq!(une_fois, deux_fois);
    }

    #[test]
    fn tri_conserve_longueur(v in any::<Vec<i32>>()) {
        let longueur_avant = v.len();
        let trie = trier_vec(v);
        prop_assert_eq!(trie.len(), longueur_avant);
    }

    #[test]
    fn tri_produit_vecteur_ordonne(v in any::<Vec<i32>>()) {
        let trie = trier_vec(v);
        // Vérifier que chaque paire consécutive est ordonnée
        for paire in trie.windows(2) {
            prop_assert!(paire[0] <= paire[1],
                "Ordre incorrect : {} > {}", paire[0], paire[1]);
        }
    }
}
```

### Stratégies — générer des types complexes

```rust
use proptest::prelude::*;

proptest! {
    // Entiers dans une plage
    #[test]
    fn test_division(
        numerateur in -1000i32..=1000,
        denominateur in 1i32..=100,   // jamais zéro
    ) {
        let resultat = numerateur / denominateur;
        prop_assert!(resultat.abs() <= numerateur.abs());
    }

    // Chaînes correspondant à un pattern regex
    #[test]
    fn test_identifiant(id in "[a-z][a-z0-9_]{0,19}") {
        prop_assert!(id.len() <= 20);
        prop_assert!(id.chars().next().map(|c| c.is_alphabetic()).unwrap_or(false));
    }

    // Vec avec taille contrôlée
    #[test]
    fn test_liste_courte(v in proptest::collection::vec(any::<i32>(), 0..50)) {
        prop_assert!(v.len() <= 50);
    }

    // Option
    #[test]
    fn test_option(opt in proptest::option::of(any::<i32>())) {
        match opt {
            Some(n) => prop_assert!(n != i32::MIN),  // juste une propriété exemple
            None => prop_assert!(true),
        }
    }
}
```

### Propriétés classiques à tester

```rust
use proptest::prelude::*;
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
struct Utilisateur {
    nom: String,
    age: u8,
}

// Propriété 1 : round-trip de sérialisation
proptest! {
    #[test]
    fn serialisation_roundtrip(
        nom in "[A-Za-z ]{1,50}",
        age in 0u8..=120,
    ) {
        let utilisateur = Utilisateur { nom, age };
        let json = serde_json::to_string(&utilisateur).unwrap();
        let reparse: Utilisateur = serde_json::from_str(&json).unwrap();
        prop_assert_eq!(utilisateur, reparse, "round-trip sérialization échoué");
    }
}

// Propriété 2 : associativité
fn concatener(a: &str, b: &str) -> String {
    format!("{}{}", a, b)
}

proptest! {
    #[test]
    fn concatenation_longueur(s1 in ".*", s2 in ".*") {
        let resultat = concatener(&s1, &s2);
        prop_assert_eq!(
            resultat.len(),
            s1.len() + s2.len(),
            "La concaténation ne devrait pas changer la longueur totale"
        );
    }
}
```

### Stratégies personnalisées

```rust
use proptest::prelude::*;
use proptest::strategy::Strategy;

#[derive(Debug, Clone)]
struct Intervalle {
    debut: i32,
    fin: i32,
}

// Stratégie custom : générer un intervalle valide (debut <= fin)
fn intervalle_valide() -> impl Strategy<Value = Intervalle> {
    (any::<i32>(), any::<i32>())
        .prop_map(|(a, b)| {
            if a <= b {
                Intervalle { debut: a, fin: b }
            } else {
                Intervalle { debut: b, fin: a }
            }
        })
}

proptest! {
    #[test]
    fn test_intervalle(i in intervalle_valide()) {
        prop_assert!(i.debut <= i.fin);
    }
}
```

### `proptest_derive` — Arbitrary automatique

```toml
[dev-dependencies]
proptest = "1"
proptest-derive = "0.4"
```

```rust
use proptest_derive::Arbitrary;
use proptest::prelude::*;

#[derive(Debug, Clone, Arbitrary)]
struct Configuration {
    #[proptest(strategy = "1u16..=65535")]
    port: u16,
    #[proptest(strategy = "\"[a-z]{3,20}\"")]
    hote: String,
    actif: bool,
    #[proptest(strategy = "proptest::option::of(any::<u32>())")]
    timeout_ms: Option<u32>,
}

proptest! {
    #[test]
    fn test_configuration(config in any::<Configuration>()) {
        prop_assert!(config.port > 0);
        prop_assert!(config.hote.len() >= 3);
    }
}
```

> [!tip] Shrinking automatique
> Quand Proptest trouve un cas d'échec, il **réduit automatiquement** l'input au cas minimal qui reproduit l'échec. C'est une des fonctionnalités les plus puissantes : au lieu de voir un Vec de 500 éléments qui échoue, tu vois le plus petit Vec (souvent 1-2 éléments) qui reproduit le bug.

---

## 7. `quickcheck` comme alternative

```toml
[dev-dependencies]
quickcheck = "1"
quickcheck_macros = "1"
```

```rust
use quickcheck::quickcheck;
use quickcheck_macros::quickcheck;

fn reverse<T: Clone>(v: Vec<T>) -> Vec<T> {
    let mut r = v.clone();
    r.reverse();
    r
}

#[quickcheck]
fn double_reverse_est_identique(v: Vec<i32>) -> bool {
    reverse(reverse(v.clone())) == v
}

// Avec la macro quickcheck! pour des propriétés plus complexes
quickcheck! {
    fn longueur_preservee(v: Vec<String>) -> bool {
        reverse(v.clone()).len() == v.len()
    }
}
```

**Différences Proptest vs Quickcheck :**

| Aspect | Proptest | Quickcheck |
|--------|---------|------------|
| Définition des stratégies | Explicite, composable | Implicite via `Arbitrary` |
| Shrinking | Intégré, sophistiqué | Via `Arbitrary::shrink()` — souvent basique |
| API | `proptest!` macro | `quickcheck!` macro ou attribut `#[quickcheck]` |
| Ergonomie types custom | `#[derive(Arbitrary)]` | Impl manuelle |

---

## 8. Fuzzing avec `cargo-fuzz`

Le fuzzing génère des inputs mutés pour trouver des crashs, panics, ou comportements indéfinis — idéal pour les parseurs et les fonctions qui traitent des données non fiables.

```bash
# Installer cargo-fuzz (nécessite nightly pour libFuzzer)
cargo install cargo-fuzz

# Initialiser dans un projet
cargo fuzz init

# Créer un nouveau target de fuzzing
cargo fuzz add mon_parseur
```

```rust
// fuzz/fuzz_targets/mon_parseur.rs
#![no_main]
use libfuzzer_sys::fuzz_target;

fuzz_target!(|data: &[u8]| {
    // Le fuzzer appelle cette closure avec des inputs mutés
    // Elle ne doit JAMAIS paniquer sur des inputs bien formés
    // (les panics sont des crashs détectés par le fuzzer)
    if let Ok(s) = std::str::from_utf8(data) {
        // Tester le parseur avec n'importe quelle chaîne UTF-8
        let _ = mon_crate::parser_config(s);
        // Si cette ligne panique → bug trouvé !
    }
});
```

```bash
# Lancer le fuzzer (Ctrl+C pour arrêter)
cargo fuzz run mon_parseur

# Lancer avec des seeds (inputs initiaux)
cargo fuzz run mon_parseur corpus/mon_parseur/

# Rejouer un crash trouvé
cargo fuzz run mon_parseur artifacts/mon_parseur/crash-xxx
```

> [!info] Fuzzing en CI
> Le fuzzing continu nécessite des heures/jours pour être efficace. En CI, lancer avec un timeout court (`cargo fuzz run -- -max_total_time=60`) pour détecter les crashs évidents rapidement, et avoir une session de fuzzing longue séparée en scheduled job.

---

## 9. Snapshot testing avec `insta`

```toml
[dev-dependencies]
insta = { version = "1", features = ["json", "yaml", "toml"] }
```

```rust
use insta::assert_snapshot;
use insta::assert_debug_snapshot;
use insta::assert_json_snapshot;

struct Rapport {
    titre: String,
    lignes: Vec<String>,
    total: i32,
}

fn generer_rapport(donnees: &[i32]) -> Rapport {
    Rapport {
        titre: "Rapport mensuel".to_string(),
        lignes: donnees.iter().map(|n| format!("  Valeur : {}", n)).collect(),
        total: donnees.iter().sum(),
    }
}

#[test]
fn test_snapshot_rapport() {
    let rapport = generer_rapport(&[10, 20, 30]);

    // Première exécution : crée le snapshot dans tests/snapshots/
    // Exécutions suivantes : compare avec le snapshot sauvegardé
    assert_snapshot!("rapport_simple", format!(
        "{}\n{}\nTotal: {}",
        rapport.titre,
        rapport.lignes.join("\n"),
        rapport.total
    ));
}

// Pour des structures complexes
#[derive(Debug, serde::Serialize)]
struct ResultatComplexe {
    statut: String,
    valeurs: Vec<f64>,
    metadata: std::collections::HashMap<String, String>,
}

#[test]
fn test_snapshot_json() {
    let resultat = ResultatComplexe {
        statut: "ok".to_string(),
        valeurs: vec![1.1, 2.2, 3.3],
        metadata: [("version", "1.0"), ("auteur", "test")]
            .into_iter()
            .map(|(k, v)| (k.to_string(), v.to_string()))
            .collect(),
    };

    // Compare la sérialisation JSON avec le snapshot
    assert_json_snapshot!("resultat_complexe", resultat);
}
```

```bash
# Revoir et approuver/refuser les snapshots modifiés
cargo install cargo-insta
cargo insta review

# Accepter tous les changements sans revue interactine
cargo insta accept

# Rejeter tous les changements
cargo insta reject
```

> [!tip] Quand utiliser Insta
> Idéal pour tester des sorties complexes (CLI output, HTML généré, JSON d'API, structures profondes) sans les écrire manuellement. Au lieu d'écrire `assert_eq!(sortie, "texte long sur 30 lignes...")`, Insta capture la sortie et te demande si elle est correcte à la première exécution.

---

## 10. Coverage

### `cargo-llvm-cov` (cross-platform)

```bash
cargo install cargo-llvm-cov

# Coverage global (affichage terminal)
cargo llvm-cov

# Rapport HTML dans target/llvm-cov/
cargo llvm-cov --html
open target/llvm-cov/html/index.html

# Format LCOV pour Codecov/Coveralls
cargo llvm-cov --lcov --output-path lcov.info

# Avec nextest (meilleur runner)
cargo llvm-cov nextest
```

### `cargo-tarpaulin` (Linux uniquement)

```bash
cargo install cargo-tarpaulin

cargo tarpaulin --out Html
cargo tarpaulin --out Lcov
```

### Intégration GitHub Actions

```yaml
# .github/workflows/coverage.yml
name: Coverage

on: [push, pull_request]

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: llvm-tools-preview
      - uses: taiki-e/install-action@cargo-llvm-cov
      - name: Générer le rapport de coverage
        run: cargo llvm-cov --all-features --lcov --output-path lcov.info
      - name: Uploader sur Codecov
        uses: codecov/codecov-action@v4
        with:
          files: lcov.info
          token: ${{ secrets.CODECOV_TOKEN }}
```

---

## 11. Stratégie de test — architecture complète

### Les 3 niveaux de tests

```
┌─────────────────────────────────────────────────────────┐
│  Niveau 3 : Tests device réel (manuels)                  │
│  Tests d'intégration avec hardware, DB réelle, réseau   │
│  → exécution manuelle avant release                     │
├─────────────────────────────────────────────────────────┤
│  Niveau 2 : Tests d'intégration (CI nightly)             │
│  API publique, mocks HTTP, fichiers temporaires          │
│  → cargo test --tests                                   │
├─────────────────────────────────────────────────────────┤
│  Niveau 1 : Tests unitaires + property-based (pre-commit)│
│  Logique pure, pas d'I/O, < 100ms total                  │
│  → cargo test --lib && cargo test --doc                 │
└─────────────────────────────────────────────────────────┘
```

### Convention de nommage

```rust
// Nommage recommandé : Composant_Scenario_ComportementAttendu
#[test]
fn parseur_config_entree_vide_retourne_defaut() { /* ... */ }

#[test]
fn parseur_config_json_invalide_retourne_erreur() { /* ... */ }

#[test]
fn parseur_config_version_2_migre_vers_version_3() { /* ... */ }

#[test]
fn service_email_envoi_echoue_retry_3_fois() { /* ... */ }

#[test]
fn tri_vecteur_vide_retourne_vide() { /* ... */ }
```

### `cargo-nextest` — runner recommandé

```bash
cargo install cargo-nextest

# Remplace cargo test
cargo nextest run

# Voir l'output en temps réel
cargo nextest run --no-capture

# Lancer un test spécifique
cargo nextest run -E "test(=parseur_config_entree_vide_retourne_defaut)"

# Parallélisme contrôlé
cargo nextest run --test-threads 4

# Format JUnit pour les CI
cargo nextest run --profile ci
```

```toml
# .config/nextest.toml
[profile.ci]
test-threads = "num-cpus"
failure-output = "immediate-final"
status-level = "all"
```

---

## Exercices pratiques

### Exercice 1 — Tests complets d'un parseur CSV

Implémente `parser_csv(contenu: &str) -> Result<Vec<Vec<String>>, ErreurCsv>`.

Exigences :
- Tests unitaires : virgule comme séparateur, guillemets, ligne vide, entête seule
- Test `#[should_panic]` : accès hors-borne
- Doctest avec 3 exemples dans la documentation
- Test d'intégration dans `tests/csv_integration.rs` qui lit un vrai fichier

### Exercice 2 — Benchmarks comparatifs

Benchmark deux implémentations de recherche dans une liste triée :
- `recherche_lineaire(v: &[i32], cible: i32) -> Option<usize>`
- `recherche_binaire(v: &[i32], cible: i32) -> Option<usize>`

Utilise `BenchmarkId` pour tester avec des listes de 100, 1000, et 10 000 éléments.

### Exercice 3 — Propriétés de sérialisation

Avec Proptest et [[16 - Serde Serialisation et Deserialisation]], teste les propriétés suivantes d'un type `Config { nom: String, valeur: u32, actif: bool }` :

1. Sérialisation JSON round-trip : `from_str(to_string(x)) == x`
2. Sérialisation TOML round-trip identique
3. La longueur de `nom` après désérialisation est identique à l'originale

### Exercice 4 — Mock d'un client HTTP

Crée un `ClientMeteo` qui appelle une API HTTP pour récupérer la météo.
- Définis un trait `ApiMeteo` avec une méthode `async fn temperature(&self, ville: &str) -> Result<f64, ErreurMeteo>`
- Mock le trait avec `mockall` pour tester `ClientMeteo::resume_journee()`
- Mock le serveur HTTP avec `mockito` pour un test d'intégration

### Exercice 5 — Snapshot de CLI

Teste la sortie de ton crate CLI (voir [[07 - Projet Rust CLI]]) avec Insta :
- Snapshot du `--help`
- Snapshot d'une commande avec des données de fixture
- Workflow : `cargo insta review` pour approuver, `cargo test` pour vérifier

---

## Notes liées

### Fondamentaux Rust
- [[04 - Gestion des Erreurs en Rust]] — `Result` dans les tests, gestion d'erreurs à tester
- [[07 - Projet Rust CLI]] — contexte de projet pour appliquer les tests
- [[18 - Closures FnOnce FnMut Fn en Profondeur]] — closures dans les stratégies Proptest

### Outils et infrastructure
- [[19 - Cargo Avance Workspaces et Publication]] — `dev-dependencies`, profiles bench, `cargo-nextest`
- [[16 - Serde Serialisation et Deserialisation]] — tests de round-trip sérialisation

### Patterns avancés
- [[05 - Collections et Iterateurs]] — stratégies Proptest sur les collections
- [[15 - Web Frameworks Axum et Actix]] — mockito/wiremock pour tester les clients HTTP
- [[08 - Concurrence et Smart Pointers]] — tests de thread safety avec `--test-threads=1`
- [[09 - Async Await et Tokio]] — `#[tokio::test]`, tests async
