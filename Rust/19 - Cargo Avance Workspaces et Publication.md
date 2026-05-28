# 19 - Cargo Avancé : Workspaces et Publication

> [!info] Prérequis
> Ce cours suppose que tu maîtrises les bases de Rust et de Cargo. Il est conseillé d'avoir pratiqué [[07 - Projet Rust CLI]] avant d'aborder les workspaces. Les macros de build sont abordées en lien avec [[10 - Macros et Metaprogrammation]].

Cargo est bien plus qu'un gestionnaire de paquets : c'est le système de build, de test, de publication et d'organisation de projets Rust. Ce cours couvre les fonctionnalités avancées qui deviennent indispensables dès qu'un projet dépasse quelques fichiers.

---

## 1. `Cargo.toml` en détail — toutes les sections

### Section `[package]`

La section `[package]` décrit les métadonnées du crate publié ou utilisé localement.

```toml
[package]
name = "mon_crate"
version = "1.2.3"
edition = "2021"
authors = ["Erwan Barat <barat.erwan@gmail.com>"]
description = "Un crate qui fait des choses utiles"
license = "MIT OR Apache-2.0"
repository = "https://github.com/rwanbt/mon_crate"
homepage = "https://mon_crate.rs"
documentation = "https://docs.rs/mon_crate"
readme = "README.md"
keywords = ["parser", "text", "utility"]   # 5 max sur crates.io
categories = ["text-processing"]           # liste officielle crates.io
rust-version = "1.70"                      # MSRV (Minimum Supported Rust Version)
exclude = ["benches/", "tests/fixtures/large/"]
include = ["src/**/*", "Cargo.toml", "README.md", "LICENSE"]
```

> [!tip] `exclude` vs `include`
> `include` est prioritaire sur `exclude`. Si les deux sont présents, seul `include` est pris en compte. Pour un crate publié sur crates.io, minimiser la taille du paquet avec `exclude` accélère le téléchargement pour tous les utilisateurs.

### Section `[dependencies]`

```toml
[dependencies]
# Version semver standard
serde = "1"                    # ^1.0.0 — compatible avec >=1.0.0, <2.0.0
tokio = "~1.28"                # ~1.28.0 — compatible avec >=1.28.0, <1.29.0
regex = "=1.9.1"               # exactement 1.9.1
rand = "*"                     # n'importe quelle version (déconseillé en production)
clap = ">=4.0, <5.0"          # plage explicite

# Dépendance git
ma_lib = { git = "https://github.com/user/ma_lib", branch = "main" }
ma_lib2 = { git = "https://github.com/user/ma_lib2", rev = "a1b2c3d" }
ma_lib3 = { git = "https://github.com/user/ma_lib3", tag = "v1.2.0" }

# Dépendance par chemin (développement local ou workspace)
mon_autre_crate = { path = "../mon_autre_crate" }

# Dépendance optionnelle (activable via feature)
serde = { version = "1", optional = true }

# Activer des features d'une dépendance
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }

# Registry personnalisé (crates.io privé)
ma_lib_privee = { version = "1", registry = "mon-registry" }
```

> [!warning] Le caret `^` est implicite
> `serde = "1"` est équivalent à `serde = "^1.0.0"`. Le caret autorise les mises à jour mineure et patch, mais jamais la majeure. C'est le comportement par défaut de Cargo — il garantit la compatibilité semver.

### Sections `[dev-dependencies]` et `[build-dependencies]`

```toml
[dev-dependencies]
# Uniquement pour les tests et benchmarks — non incluses dans le binaire final
criterion = { version = "0.5", features = ["html_reports"] }
proptest = "1"
mockall = "0.12"
insta = "1"
tempfile = "3"

[build-dependencies]
# Uniquement pour build.rs — ne peuvent pas être utilisées dans src/
cc = "1"
bindgen = "0.69"
prost-build = "0.12"
```

> [!info] Séparation des dépendances
> Cette séparation existe pour la performance : les `dev-dependencies` ne sont pas compilées pour les releases. Les `build-dependencies` sont compilées dans un contexte différent (l'hôte, pas la cible) pour que `build.rs` puisse s'exécuter même en cross-compilation.

### Section `[features]`

Voir la section dédiée ci-dessous (section 2).

### Section `[profile.*]`

Voir la section dédiée (section 4).

### Section `[workspace]`

Voir la section dédiée (section 3).

### Sections `[patch]` et `[replace]`

```toml
# [patch] remplace une dépendance par une version locale ou git — SANS changer les Cargo.toml des dépendants
[patch.crates-io]
rand = { path = "../rand-fork" }
serde = { git = "https://github.com/serde-rs/serde", branch = "fix-my-bug" }

# [replace] est déprécié depuis Rust 1.31 — utiliser [patch] à la place
# [replace]
# "rand:0.8.5" = { path = "../rand-fork" }
```

> [!tip] Usage de `[patch]`
> `[patch]` est précieux pour tester un correctif dans une dépendance avant qu'il soit publié sur crates.io. Il s'applique à tout le graphe de dépendances de façon transitive, sans modifier le `Cargo.toml` de chaque dépendant.

---

## 2. Features — système de fonctionnalités conditionnelles

### Définir des features

```toml
[features]
# Feature par défaut — active automatiquement "json" et "logging"
default = ["json", "logging"]

# Features simples
json = []
logging = []
async = []

# Feature qui active une dépendance optionnelle
serde-support = ["dep:serde"]

# Feature qui en active une autre (features additives)
full = ["json", "logging", "async", "serde-support"]

# Feature qui active les features d'une dépendance
json-schema = ["serde/derive", "serde_json/schema"]
```

```toml
[dependencies]
serde = { version = "1", optional = true }
serde_json = { version = "1", optional = true }
log = { version = "0.4", optional = true }
```

> [!warning] Features additives — jamais soustractives
> Rust garantit que les features ne peuvent qu'activer du code, jamais en désactiver. Si deux dépendances activent chacune une feature différente d'un crate commun, **les deux features seront actives** (feature unification). Il est donc impossible de désactiver la feature `default` d'un crate si une de tes dépendances l'active. C'est un choix de conception fondamental.

### Utiliser les features dans le code

```rust
// Conditionnellement compiler tout un module
#[cfg(feature = "json")]
pub mod json_support {
    // ...
}

// Conditionnellement compiler une fonction
#[cfg(feature = "serde-support")]
use serde::{Deserialize, Serialize};

#[cfg(feature = "serde-support")]
#[derive(Serialize, Deserialize)]
pub struct MonType {
    pub valeur: i32,
}

#[cfg(not(feature = "serde-support"))]
pub struct MonType {
    pub valeur: i32,
}

// Dans une fonction
pub fn traiter(data: &[u8]) -> String {
    #[cfg(feature = "logging")]
    log::debug!("Traitement de {} bytes", data.len());

    // code commun...
    String::from("résultat")
}

// Condition combinée
#[cfg(all(feature = "json", feature = "async"))]
pub async fn charger_json() { /* ... */ }

#[cfg(any(feature = "json", feature = "yaml"))]
pub fn charger_config() { /* ... */ }
```

### Activer les features depuis la ligne de commande

```bash
# Features spécifiques en plus des defaults
cargo build --features "json async"

# Toutes les features
cargo build --all-features

# Sans les features par défaut
cargo build --no-default-features

# Combiner : sans defaults, avec des features précises
cargo build --no-default-features --features "json"

# Pour un membre de workspace
cargo build -p mon_crate --features "json"
```

### Feature unification en workspace

> [!warning] Comportement surprenant en workspace
> Si ton workspace contient `crate_a` (dépend de `serde` sans features) et `crate_b` (dépend de `serde` avec `features = ["derive"]`), lors d'un `cargo build --workspace`, Serde sera compilé **avec** le feature `derive` pour **tous** les membres. Les features sont unifiées au niveau du graphe de dépendances entier.
>
> C'est intentionnel pour éviter de compiler la même crate deux fois, mais ça peut masquer des erreurs si tu testes `crate_a` seul sans le workspace.

---

## 3. Workspaces — monorepo Rust

### Pourquoi utiliser un workspace

Un workspace regroupe plusieurs crates dans un seul dépôt avec :
- Un seul `Cargo.lock` partagé (versions cohérentes entre tous les membres)
- Un répertoire `target/` partagé (pas de recompilation entre crates)
- Build parallèle automatique selon le graphe de dépendances
- Gestion centralisée des métadonnées et des versions de dépendances

### Structure d'un workspace

```
mon-projet/
├── Cargo.toml          ← workspace root
├── Cargo.lock          ← un seul lock file partagé
├── crates/
│   ├── core/
│   │   ├── Cargo.toml
│   │   └── src/lib.rs
│   ├── cli/
│   │   ├── Cargo.toml
│   │   └── src/main.rs
│   └── web/
│       ├── Cargo.toml
│       └── src/main.rs
└── target/             ← partagé entre tous les membres
```

```toml
# Cargo.toml racine (workspace root)
[workspace]
members = [
    "crates/core",
    "crates/cli",
    "crates/web",
]
exclude = ["examples/standalone"]   # exclure certains sous-dossiers
resolver = "2"                      # résolveur de features recommandé

# Métadonnées partagées (Rust 1.64+)
[workspace.package]
version = "0.5.0"
authors = ["Erwan Barat <barat.erwan@gmail.com>"]
license = "MIT"
edition = "2021"
repository = "https://github.com/rwanbt/mon-projet"

# Dépendances partagées (Rust 1.64+)
[workspace.dependencies]
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
anyhow = "1"
tracing = "0.1"
```

```toml
# crates/core/Cargo.toml — hériter des métadonnées du workspace
[package]
name = "mon-projet-core"
version.workspace = true           # hérite de workspace.package.version
edition.workspace = true
license.workspace = true

[dependencies]
serde.workspace = true             # hérite de workspace.dependencies.serde
anyhow.workspace = true
# On peut surcharger avec des features supplémentaires :
tokio = { workspace = true, features = ["rt-multi-thread"] }
```

```toml
# crates/cli/Cargo.toml
[package]
name = "mon-projet-cli"
version.workspace = true
edition.workspace = true

[dependencies]
mon-projet-core = { path = "../core" }   # dépendance entre membres
clap = { version = "4", features = ["derive"] }
anyhow.workspace = true
```

### Commandes workspace

```bash
# Builder tout le workspace
cargo build --workspace

# Builder un seul membre
cargo build -p mon-projet-core
cargo build -p mon-projet-cli

# Tester tout le workspace
cargo test --workspace

# Tester un membre spécifique
cargo test -p mon-projet-core

# Lancer le binaire d'un membre
cargo run -p mon-projet-cli -- --help

# Linter tout
cargo clippy --workspace --all-targets

# Vérifier tout sans compiler les artefacts
cargo check --workspace
```

### Resolver v2 — différences avec v1

```toml
# À la racine du workspace ou dans [package]
resolver = "2"
```

Le resolver v2 (défaut avec edition 2021) diffère du v1 sur les features :

- **v1** : les features de `[dev-dependencies]` s'appliquent globalement, même en build normal
- **v2** : les `[dev-dependencies]` et `[build-dependencies]` ont leurs propres features, séparées des `[dependencies]`

> [!info] Conseil
> Toujours utiliser `resolver = "2"`. Il évite des situations où une dépendance de dev active une feature lourde (ex. `tokio/full`) dans le build de production.

---

## 4. Profiles — optimiser le build

### Profiles standard

```toml
# Profile de développement (cargo build, cargo run)
[profile.dev]
opt-level = 0          # 0 = pas d'optimisation, rapide à compiler
debug = true           # symboles de debug (true = niveau 2)
debug-assertions = true
overflow-checks = true
lto = false            # Link-Time Optimization — désactivé en dev
codegen-units = 256    # parallélisme de compilation maximal
incremental = true     # compilation incrémentale
panic = "unwind"       # dépile la stack en cas de panic (nécessaire pour catch_unwind)

# Profile de release (cargo build --release)
[profile.release]
opt-level = 3          # optimisation maximale
debug = false          # pas de symboles (true pour profiler en release)
debug-assertions = false
overflow-checks = false
lto = "thin"           # LTO thin — bon compromis perf/temps de link
codegen-units = 1      # 1 seule unité = meilleures optimisations inter-fonctions
strip = true           # retirer les symboles du binaire final (taille réduite)
panic = "abort"        # panic = arrêt immédiat (pas de stack unwinding)
```

> [!tip] LTO : thin vs fat
> - `lto = false` : pas de LTO, link rapide
> - `lto = "thin"` : LTO partiel, bon équilibre perf/temps (recommandé pour la production)
> - `lto = true` ou `lto = "fat"` : LTO complet, meilleure perf mais link très lent

### `panic = "abort"` vs `panic = "unwind"`

| Mode | Comportement | Avantages | Inconvénients |
|------|-------------|-----------|---------------|
| `"unwind"` | Dépile la stack, exécute les `Drop` | Nécessaire pour `catch_unwind`, tests robustes | Binaire plus lourd, code de cleanup présent |
| `"abort"` | Arrêt immédiat du process | Binaire plus petit, pas de code d'unwinding | Impossble d'utiliser `catch_unwind`, pertes de ressources possibles |

> [!warning] `panic = "abort"` en WASM et embedded
> Pour les cibles sans OS (WebAssembly, microcontrôleurs), `panic = "abort"` est souvent obligatoire. L'unwinding nécessite un runtime système qui n'existe pas sur ces plateformes.

### Profiles personnalisés

```toml
# Profile pour le profiling CPU (perf, flamegraph)
[profile.profiling]
inherits = "release"   # part du profile release
debug = true           # ajouter les symboles pour que flamegraph affiche les noms
strip = false          # ne pas retirer les symboles

# Profile avec optimisations debug (debug mais plus rapide)
[profile.dev.package."*"]
opt-level = 1          # optimiser toutes les dépendances en dev (build plus lent, exec plus rapide)

[profile.dev.package."mon_crate_lent"]
opt-level = 3          # optimisation maximale pour un crate spécifique seulement
```

```bash
# Utiliser un profile personnalisé
cargo build --profile profiling
```

---

## 5. Build scripts — `build.rs`

Le fichier `build.rs` à la racine du crate est compilé et exécuté **avant** la compilation du crate lui-même. Il permet d'orchestrer des étapes de build complexes.

### Mise en place

```toml
[package]
build = "build.rs"     # implicite si le fichier existe, explicite pour désactiver avec build = false

[build-dependencies]
cc = "1"               # compilateur C — uniquement disponible dans build.rs
bindgen = "0.69"
```

### Variables d'environnement `cargo:` (instructions de build)

```rust
// build.rs
fn main() {
    // Recompiler si ces fichiers changent (sinon Cargo met en cache le résultat)
    println!("cargo:rerun-if-changed=src/wrapper.h");
    println!("cargo:rerun-if-changed=build.rs");
    println!("cargo:rerun-if-env-changed=CC");   // recompiler si la variable d'env change

    // Lier une bibliothèque système
    println!("cargo:rustc-link-lib=ssl");        // dynamique : -lssl
    println!("cargo:rustc-link-lib=static=mylib"); // statique
    println!("cargo:rustc-link-search=native=/usr/local/lib");  // répertoire de recherche

    // Passer un cfg au compilateur Rust
    println!("cargo:rustc-cfg=has_feature_x");   // utilisable avec #[cfg(has_feature_x)]

    // Définir une variable d'environnement visible dans src/ via env!()
    println!("cargo:rustc-env=BUILD_TIMESTAMP={}", chrono::Utc::now().to_rfc3339());
}
```

### Générer du code avec `build.rs`

```rust
// build.rs — générer un fichier Rust à partir de données
use std::env;
use std::fs;
use std::path::Path;

fn main() {
    let out_dir = env::var("OUT_DIR").unwrap();
    let dest_path = Path::new(&out_dir).join("generated.rs");

    // Générer des constantes depuis un fichier de config
    let config = fs::read_to_string("config.json").expect("config.json manquant");
    let version_str = extract_version(&config);

    fs::write(&dest_path, format!(
        r#"
        pub const GENERATED_VERSION: &str = "{version_str}";
        pub const BUILD_TARGET: &str = "{target}";
        "#,
        version_str = version_str,
        target = env::var("TARGET").unwrap_or_default(),
    )).unwrap();

    println!("cargo:rerun-if-changed=config.json");
}

fn extract_version(config: &str) -> &str {
    "1.0.0" // simplifié
}
```

```rust
// src/lib.rs — inclure le code généré
include!(concat!(env!("OUT_DIR"), "/generated.rs"));

pub fn get_version() -> &'static str {
    GENERATED_VERSION
}
```

### Détecter la version du compilateur dans `build.rs`

```rust
// build.rs
fn main() {
    // Activer un cfg si le compilateur est assez récent
    let version = rustc_version::version().unwrap();
    if version.minor >= 75 {
        println!("cargo:rustc-cfg=has_new_api");
    }
}
```

```toml
[build-dependencies]
rustc-version = "0.4"
```

### Lier une bibliothèque C avec `cc`

```rust
// build.rs
fn main() {
    cc::Build::new()
        .file("src/native/helper.c")
        .file("src/native/utils.c")
        .flag("-O2")
        .compile("myhelper");  // produit libmyhelper.a

    println!("cargo:rerun-if-changed=src/native/helper.c");
    println!("cargo:rerun-if-changed=src/native/utils.c");
}
```

> [!info] Variables d'environnement Cargo disponibles dans `build.rs`
> - `OUT_DIR` : répertoire de sortie pour les artefacts générés
> - `CARGO_MANIFEST_DIR` : répertoire du `Cargo.toml`
> - `TARGET` : triple cible (ex. `x86_64-unknown-linux-gnu`)
> - `HOST` : triple de l'hôte de compilation
> - `PROFILE` : `debug` ou `release`
> - `CARGO_PKG_VERSION` : version du package

---

## 6. Outils Cargo essentiels

### `cargo-expand` — inspecter les macros

```bash
cargo install cargo-expand

# Voir le code après expansion de toutes les macros
cargo expand

# Expansion d'un module spécifique
cargo expand mon::module::path

# Expansion avec un feature activé
cargo expand --features "serde-support"
```

Indispensable pour déboguer [[10 - Macros et Metaprogrammation]] et comprendre ce que `#[derive(...)]` génère réellement.

### `cargo-watch` — rebuilder automatiquement

```bash
cargo install cargo-watch

cargo watch -x check                    # vérifier à chaque changement
cargo watch -x test                     # relancer les tests
cargo watch -x "run -- --arg valeur"    # relancer le programme
cargo watch -x "clippy -- -D warnings" # linter en continu
cargo watch -c -x test                  # -c pour clear le terminal
```

### `cargo-flamegraph` — profiling CPU

```bash
cargo install flamegraph

# Générer un flamegraph du binaire
cargo flamegraph -- --input data.txt

# Sur Linux, nécessite perf :
sudo cargo flamegraph --bin mon_binaire
```

### `cargo-audit` — CVE scan

```bash
cargo install cargo-audit

# Scanner le Cargo.lock contre la base RustSec
cargo audit

# En CI : échouer si une vulnérabilité est trouvée
cargo audit --deny warnings
```

### `cargo-deny` — gouvernance supply-chain

```bash
cargo install cargo-deny

# Initialiser la configuration
cargo deny init

# Vérifier licenses, bans, CVE, duplicatas
cargo deny check
```

```toml
# .cargo/deny.toml (généré par cargo deny init)
[licenses]
allow = ["MIT", "Apache-2.0", "BSD-3-Clause"]
deny = ["GPL-3.0"]

[bans]
multiple-versions = "deny"   # interdire les versions dupliquées d'un crate

[advisories]
ignore = []   # ignorer des CVE spécifiques (avec justification)
```

### `cargo-nextest` — runner de tests amélioré

```bash
cargo install cargo-nextest

# Remplace `cargo test` — plus rapide, meilleur output
cargo nextest run

# Avec filtrage
cargo nextest run --test-threads 4
cargo nextest run -E "test(mon_test)"
```

### `cargo-udeps` et `cargo-machete` — dépendances inutilisées

```bash
cargo install cargo-udeps
cargo +nightly udeps   # nécessite nightly

# Alternative stable
cargo install cargo-machete
cargo machete
```

### `cargo-bloat` — analyser la taille du binaire

```bash
cargo install cargo-bloat

cargo bloat --release --crates    # taille par crate
cargo bloat --release -n 10       # top 10 fonctions les plus lourdes
```

### `sccache` — cache de compilation partagé

```bash
# Windows
winget install Mozilla.sccache

# Configurer dans .cargo/config.toml
# [build]
# rustc-wrapper = "sccache"
```

### `cargo-semver-checks` — vérifier les breaking changes

```bash
cargo install cargo-semver-checks

# Vérifier que la nouvelle version ne brise pas la compatibilité semver
cargo semver-checks check-release
```

---

## 7. Publier sur crates.io

### Préparer la publication

```bash
# Se connecter avec le token API de crates.io
cargo login                    # ouvre le navigateur ou demande le token

# Vérifier avant de publier (simule sans envoyer)
cargo publish --dry-run

# Vérifier le contenu du paquet
cargo package --list

# Publier
cargo publish
```

### Checklist avant publication

```toml
[package]
name = "mon_crate"
version = "1.0.0"
description = "Description courte et précise (obligatoire)"
license = "MIT"                    # ou "MIT OR Apache-2.0"
# license-file = "LICENSE"         # si le fichier n'est pas standard
repository = "https://..."        # très utile pour les utilisateurs
readme = "README.md"               # affiché sur crates.io
keywords = ["keyword1", "keyword2"]  # 5 max, lowercase, tirets
categories = ["network-programming"] # liste officielle
rust-version = "1.70"              # MSRV si applicable
```

> [!warning] La publication est permanente (presque)
> Une version publiée ne peut pas être supprimée, seulement "yankée". Un crate yanké ne peut plus être téléchargé par les nouveaux projets, mais les projets qui l'utilisent déjà peuvent continuer.

### Versionning semver et breaking changes

| Type de changement | Version |
|---|---|
| Correction de bug sans changement d'API | Patch (`1.0.0` → `1.0.1`) |
| Nouvelle fonctionnalité rétrocompatible | Minor (`1.0.0` → `1.1.0`) |
| Changement d'API incompatible | Major (`1.0.0` → `2.0.0`) |

```bash
# Vérifier automatiquement les breaking changes semver
cargo semver-checks check-release --baseline-version 1.0.0
```

### Yanker une version

```bash
# Retirer une version défectueuse (ne la supprime pas, empêche les nouveaux téléchargements)
cargo yank --version 1.0.1

# Annuler un yank
cargo yank --version 1.0.1 --undo
```

---

## 8. Configuration Cargo — `.cargo/config.toml`

```toml
# .cargo/config.toml (local au projet) ou ~/.cargo/config.toml (global)

[alias]
t = "test"
b = "build"
r = "run"
rr = "run --release"
c = "check"
cl = "clippy -- -D warnings"
ex = "expand"    # nécessite cargo-expand

[build]
jobs = 8                              # parallélisme
rustc-wrapper = "sccache"            # cache de compilation
target = "x86_64-unknown-linux-gnu"  # cible par défaut

[net]
retry = 3                            # tentatives réseau
offline = false                      # ne pas exiger le réseau

[registries]
mon-registry = { index = "https://my-registry.example.com/index" }

[target.x86_64-unknown-linux-gnu]
linker = "clang"
rustflags = ["-C", "link-arg=-fuse-ld=lld"]
```

### Variables d'environnement Cargo au runtime

```rust
// Disponibles via env!() au compile time
const VERSION: &str = env!("CARGO_PKG_VERSION");
const NAME: &str = env!("CARGO_PKG_NAME");
const AUTHORS: &str = env!("CARGO_PKG_AUTHORS");
const MANIFEST_DIR: &str = env!("CARGO_MANIFEST_DIR");

// option_env! pour les variables optionnelles (retourne Option<&str>)
const CI_BUILD: Option<&str> = option_env!("CI");

fn main() {
    println!("Version : {}", VERSION);
    if CI_BUILD.is_some() {
        println!("Build en CI");
    }
}
```

---

## 9. Résumé des commandes Cargo essentielles

```bash
# Développement
cargo new mon_projet --lib          # nouveau crate bibliothèque
cargo new mon_projet --bin          # nouveau crate binaire
cargo init                          # initialiser dans un dossier existant
cargo check                         # vérifier sans compiler
cargo build                         # compiler en mode debug
cargo build --release               # compiler en mode release
cargo run -- --arg valeur           # compiler et lancer
cargo test                          # lancer tous les tests
cargo test nom_test                 # lancer un test spécifique
cargo bench                         # lancer les benchmarks
cargo doc --open                    # générer et ouvrir la documentation

# Dépendances
cargo add serde --features derive   # ajouter une dépendance (cargo-edit)
cargo remove serde                  # retirer une dépendance (cargo-edit)
cargo update                        # mettre à jour les dépendances
cargo tree                          # afficher l'arbre de dépendances
cargo tree -d                       # afficher les dépendances dupliquées

# Qualité
cargo clippy                        # linter
cargo fmt                           # formater le code
cargo fix                           # appliquer les corrections automatiques
cargo audit                         # CVE scan
cargo deny check                    # gouvernance supply-chain

# Publication
cargo login                         # s'authentifier sur crates.io
cargo package                       # créer l'archive sans publier
cargo publish --dry-run             # simulation de publication
cargo publish                       # publier sur crates.io
cargo yank --version 1.0.0         # yanker une version
```

---

## Exercices pratiques

### Exercice 1 — Créer un workspace

Crée un workspace `mon-app` avec trois membres :
- `crates/core` : bibliothèque avec une fonction `saluer(nom: &str) -> String`
- `crates/cli` : binaire qui appelle `core::saluer`
- `crates/utils` : fonctions utilitaires partagées

Exigences :
- Version partagée via `workspace.package`
- `serde` déclaré une seule fois dans `workspace.dependencies`
- `cargo test --workspace` doit passer

### Exercice 2 — Features conditionnelles

Dans un crate `formateur`, crée :
- Feature `json` : active une fonction `to_json(data: &str) -> String`
- Feature `yaml` : active une fonction `to_yaml(data: &str) -> String`
- Feature `full` : active les deux
- Feature `default` : vide (rien par défaut)

Teste avec `--features "json"`, `--all-features`, `--no-default-features`.

### Exercice 3 — Build script

Crée un `build.rs` qui :
1. Génère un fichier `generated.rs` avec `pub const BUILD_TIME: &str = "..."` (timestamp actuel)
2. Détecte si la variable d'environnement `ENABLE_FEATURE_X` est définie et émet `cargo:rustc-cfg=feature_x`
3. Dans `src/main.rs`, affiche `BUILD_TIME` et compile conditionnellement du code avec `#[cfg(feature_x)]`

### Exercice 4 — Publication (simulation)

Prépare un crate `hello-rwanbt` pour la publication :
- `cargo publish --dry-run` doit passer sans erreur
- Tous les champs obligatoires de `[package]` doivent être présents
- `cargo package --list` ne doit pas inclure les fichiers de bench ou de fixtures volumineuses

---

## Notes liées

### Fondamentaux
- [[01 - Introduction a Rust]] — installation de Rust et Cargo, premier projet
- [[02 - Ownership et Borrowing]] — concepts de propriété sous-jacents
- [[07 - Projet Rust CLI]] — mise en pratique dans un vrai projet CLI

### Fonctionnalités avancées
- [[10 - Macros et Metaprogrammation]] — `cargo expand` pour inspecter les macros
- [[11 - Unsafe Rust et FFI]] — `build.rs` pour lier des bibliothèques C
- [[16 - Serde Serialisation et Deserialisation]] — feature optionnelle `serde-support` typique
- [[17 - Lifetimes Avances Pin et Unpin]] — patterns avancés dans les bibliothèques publiées

### Tests et qualité
- [[20 - Testing Avance Criterion et Proptest]] — `dev-dependencies`, `cargo-nextest`, benchmarks
- [[04 - Gestion des Erreurs en Rust]] — gestion d'erreurs dans les build scripts

### Écosystème web
- [[15 - Web Frameworks Axum et Actix]] — workspaces pour les projets web multi-crates
- [[09 - Async Await et Tokio]] — features Tokio dans un workspace
