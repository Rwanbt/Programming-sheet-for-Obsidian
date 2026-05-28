# 16 - Serde : Sérialisation et Désérialisation

> [!info] Pré-requis
> Ce cours suppose la maîtrise de [[06 - Traits et Generiques]], des derive macros ([[10 - Macros et Metaprogrammation]]) et une familiarité avec [[04 - Gestion des Erreurs en Rust]]. Serde est omniprésent dans l'écosystème Rust — APIs web, configs, stockage, IPC.

---

## 1. Qu'est-ce que la sérialisation ?

La **sérialisation** (ou marshalling) est le processus de conversion d'une structure de données en mémoire vers un format transportable ou stockable : JSON, TOML, binaire, CSV, MessagePack, etc.

La **désérialisation** est l'opération inverse : reconstruire la structure depuis le format sérialisé.

### Pourquoi Serde est le standard Rust

Serde (Serialize + Deserialize) s'impose pour plusieurs raisons :

| Critère | Serde | Alternatives naïves |
|---|---|---|
| **Performance** | Zero-copy possible, zero-alloc sur certains formats | Allocations systématiques |
| **Ergonomie** | `#[derive]` génère tout le code | Implémentation manuelle fastidieuse |
| **Extensibilité** | Tout format peut implémenter le backend Serde | Format fixé |
| **Composabilité** | Structs, enums, génériques, tous supportés | Cas spéciaux à traiter manuellement |
| **Écosystème** | 1 500+ crates compatibles Serde | Fragmentation |

> [!tip] Philosophie Serde
> Serde sépare le **modèle de données** (vos types Rust) du **format** (JSON, TOML, bincode…). Vous écrivez les derives une fois, et votre struct devient compatible avec tous les formats Serde sans modification.

---

## 2. Architecture Serde

### Les deux traits fondamentaux

```rust
// Côté sérialisation
pub trait Serialize {
    fn serialize<S: Serializer>(&self, serializer: S) -> Result<S::Ok, S::Error>;
}

// Côté désérialisation
pub trait Deserialize<'de>: Sized {
    fn deserialize<D: Deserializer<'de>>(deserializer: D) -> Result<Self, D::Error>;
}
```

### Le data model Serde

Serde définit **29 types fondamentaux** qui forment un modèle de données universel. Tout format Serde mappe ses concepts vers ces types :

| Catégorie | Types |
|---|---|
| **Primitifs** | bool, i8-i64, u8-u64, i128, u128, f32, f64, char |
| **Texte** | string, str (borrowed), bytes, byte_buf |
| **Séquences** | seq, tuple, tuple_struct |
| **Maps** | map, struct |
| **Enums** | unit_variant, newtype_variant, tuple_variant, struct_variant |
| **Option** | option (None / Some) |
| **Unité** | unit, unit_struct, newtype_struct |

### Le Visitor Pattern

La désérialisation utilise le pattern Visitor pour permettre au format d'appeler la méthode appropriée selon le token rencontré :

```rust
struct DurationVisitor;

impl<'de> Visitor<'de> for DurationVisitor {
    type Value = Duration;

    fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
        formatter.write_str("a duration string like '1h30m'")
    }

    fn visit_str<E: de::Error>(self, value: &str) -> Result<Duration, E> {
        parse_duration(value).map_err(E::custom)
    }
}
```

---

## 3. Setup et formats disponibles

### Cargo.toml de base

```toml
[dependencies]
# Serde avec les derive macros (obligatoire)
serde = { version = "1", features = ["derive"] }

# Formats — ajouter selon les besoins
serde_json = "1"           # JSON (le plus courant)
toml = "0.8"               # TOML (configs Rust)
serde_yaml = "0.9"         # YAML (configs DevOps)
bincode = "1"              # Binaire compact (IPC, cache)
rmp-serde = "1"            # MessagePack (binaire + schéma flexible)
csv = "1"                  # CSV (données tabulaires)
serde_with = "3"           # Helpers avancés
```

### Tableau comparatif des formats

| Format | Taille relative | Vitesse | Lisible | Schéma | Cas d'usage |
|---|---|---|---|---|---|
| **JSON** | 100% (référence) | Moyen | ✅ Oui | Non | APIs REST, configs, debug |
| **TOML** | ~90% | Moyen | ✅ Oui | Non | Configs Rust (Cargo.toml) |
| **YAML** | ~95% | Lent | ✅ Oui | Non | Configs DevOps, Kubernetes |
| **Bincode** | ~30% | ✅ Très rapide | ❌ Non | Non | IPC, cache local, jeux |
| **MessagePack** | ~50% | Rapide | ❌ Non | Optionnel | APIs binaires, mobile |
| **CSV** | Variable | Rapide | ✅ Oui | Non | Données tabulaires, export |

> [!warning] YAML et serde_yaml
> `serde_yaml` v0.9 a changé son API. La crate est en maintenance minimale — pour les nouveaux projets, préférez `serde_json` pour les configs ou `toml` si vous voulez de la lisibilité.

---

## 4. `#[derive(Serialize, Deserialize)]`

### Structs simples

```rust
use serde::{Deserialize, Serialize};

// Struct nommée — le cas le plus courant
#[derive(Debug, Serialize, Deserialize)]
struct User {
    id: u64,
    name: String,
    email: String,
    active: bool,
}

// Tuple struct — sérialisée comme un tableau JSON
#[derive(Debug, Serialize, Deserialize)]
struct Point(f64, f64);

// Newtype struct — sérialisée comme la valeur intérieure
#[derive(Debug, Serialize, Deserialize)]
struct UserId(u64);

fn main() {
    let user = User {
        id: 1,
        name: "Alice".to_string(),
        email: "alice@example.com".to_string(),
        active: true,
    };

    let json = serde_json::to_string_pretty(&user).unwrap();
    println!("{json}");
    // {
    //   "id": 1,
    //   "name": "Alice",
    //   "email": "alice@example.com",
    //   "active": true
    // }

    let point = Point(3.14, 2.71);
    println!("{}", serde_json::to_string(&point).unwrap()); // [3.14,2.71]

    let uid = UserId(42);
    println!("{}", serde_json::to_string(&uid).unwrap()); // 42
}
```

### Enums

```rust
#[derive(Debug, Serialize, Deserialize)]
enum Shape {
    Circle { radius: f64 },          // struct variant
    Rectangle(f64, f64),             // tuple variant
    Triangle,                        // unit variant
}

// Sérialisation par défaut (format "externally tagged") :
// Circle    → {"Circle": {"radius": 5.0}}
// Rectangle → {"Rectangle": [3.0, 4.0]}
// Triangle  → "Triangle"
```

### Génériques avec bounds

```rust
#[derive(Debug, Serialize, Deserialize)]
struct ApiResponse<T>
where
    T: Serialize,  // bound requis pour la sérialisation
{
    data: T,
    status: u16,
    message: String,
}

// Serde ajoute automatiquement T: Deserialize<'de> pour la désérialisation
// via le derive macro
```

---

## 5. Attributs `#[serde(...)]` — référence complète

### Renommage

```rust
#[derive(Serialize, Deserialize)]
struct Config {
    // Renommer un champ spécifique
    #[serde(rename = "max_connections")]
    max_conn: u32,

    // Renommer toute la struct avec une convention
    // (s'applique à tous les champs non renommés explicitement)
}

// Conventions disponibles pour rename_all :
// "lowercase", "UPPERCASE", "camelCase", "PascalCase",
// "snake_case", "SCREAMING_SNAKE_CASE", "kebab-case", "SCREAMING-KEBAB-CASE"
#[derive(Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
struct ApiUser {
    user_id: u64,       // → "userId" en JSON
    first_name: String, // → "firstName" en JSON
    last_name: String,  // → "lastName" en JSON
}
```

### Skip et champs optionnels

```rust
#[derive(Serialize, Deserialize)]
struct Document {
    title: String,
    content: String,

    // Jamais sérialisé ni désérialisé
    #[serde(skip)]
    cache: Option<String>,

    // Seulement ignoré à la sérialisation (présent en désérialisation)
    #[serde(skip_serializing)]
    internal_id: u64,

    // Seulement ignoré à la désérialisation
    #[serde(skip_deserializing)]
    computed_hash: String,

    // Omis de la sortie JSON si None (évite les "field": null)
    #[serde(skip_serializing_if = "Option::is_none")]
    description: Option<String>,

    // Omis si le vecteur est vide
    #[serde(skip_serializing_if = "Vec::is_empty")]
    tags: Vec<String>,
}
```

### Valeurs par défaut

```rust
fn default_port() -> u16 { 8080 }
fn default_timeout() -> u64 { 30 }

#[derive(Deserialize)]
struct ServerConfig {
    host: String,

    // Utilise Default::default() si le champ est absent du JSON
    #[serde(default)]
    debug: bool,  // → false si absent

    // Utilise une fonction custom
    #[serde(default = "default_port")]
    port: u16,  // → 8080 si absent

    #[serde(default = "default_timeout")]
    timeout_secs: u64,
}
```

### Flatten — fusionner des structs

```rust
#[derive(Serialize, Deserialize)]
struct Pagination {
    page: u32,
    per_page: u32,
    total: u64,
}

#[derive(Serialize, Deserialize)]
struct UserListResponse {
    users: Vec<User>,

    // Les champs de Pagination sont "injectés" directement
    // dans UserListResponse au lieu d'être imbriqués
    #[serde(flatten)]
    pagination: Pagination,
}

// JSON résultant :
// {
//   "users": [...],
//   "page": 1,
//   "per_page": 20,
//   "total": 150
// }
// (pas de clé "pagination" intermédiaire)
```

### Enums polymorphiques — tagging

```rust
// 1. Externally tagged (défaut) — peu pratique pour les APIs
// {"Circle": {"radius": 5.0}}

// 2. Internally tagged — le plus courant pour les APIs REST
#[derive(Serialize, Deserialize)]
#[serde(tag = "type")]
enum Shape {
    Circle { radius: f64 },
    // → {"type": "Circle", "radius": 5.0}

    Rectangle { width: f64, height: f64 },
    // → {"type": "Rectangle", "width": 3.0, "height": 4.0}
}

// 3. Adjacently tagged — type et données séparés
#[derive(Serialize, Deserialize)]
#[serde(tag = "kind", content = "payload")]
enum Event {
    Click { x: i32, y: i32 },
    // → {"kind": "Click", "payload": {"x": 10, "y": 20}}

    KeyPress(char),
    // → {"kind": "KeyPress", "payload": "a"}
}

// 4. Untagged — Serde tente chaque variant, prend le premier qui marche
// Attention : plus lent, moins précis sur les erreurs
#[derive(Serialize, Deserialize)]
#[serde(untagged)]
enum StringOrInt {
    Str(String),
    Int(i64),
}
```

### Alias et conversions custom

```rust
#[derive(Deserialize)]
struct LegacyCompat {
    // Accepte "username" OU "user_name" OU "login" en entrée
    #[serde(alias = "user_name", alias = "login")]
    username: String,
}

// serialize_with / deserialize_with : custom sur un champ spécifique
mod timestamp_serde {
    use serde::{Deserialize, Deserializer, Serialize, Serializer};
    use std::time::{SystemTime, UNIX_EPOCH};

    pub fn serialize<S: Serializer>(time: &SystemTime, s: S) -> Result<S::Ok, S::Error> {
        let secs = time.duration_since(UNIX_EPOCH).unwrap().as_secs();
        s.serialize_u64(secs)
    }

    pub fn deserialize<'de, D: Deserializer<'de>>(d: D) -> Result<SystemTime, D::Error> {
        let secs = u64::deserialize(d)?;
        Ok(UNIX_EPOCH + std::time::Duration::from_secs(secs))
    }
}

#[derive(Serialize, Deserialize)]
struct LogEntry {
    message: String,
    #[serde(with = "timestamp_serde")]
    created_at: SystemTime,
}
```

---

## 6. serde_json en détail

### Sérialisation

```rust
use serde_json;

let user = User { id: 1, name: "Alice".to_string(), /* ... */ };

// → String
let s = serde_json::to_string(&user)?;

// → String indenté (debug / fichiers de config)
let pretty = serde_json::to_string_pretty(&user)?;

// → Vec<u8> (évite une String intermédiaire)
let bytes: Vec<u8> = serde_json::to_vec(&user)?;

// → Writer quelconque (fichier, socket, buffer)
let file = std::fs::File::create("user.json")?;
serde_json::to_writer(file, &user)?;
serde_json::to_writer_pretty(std::io::stdout(), &user)?;
```

### Désérialisation

```rust
// Depuis &str
let user: User = serde_json::from_str(r#"{"id":1,"name":"Alice"}"#)?;

// Depuis &[u8]
let bytes = b"{\"id\":1,\"name\":\"Alice\"}";
let user: User = serde_json::from_slice(bytes)?;

// Depuis un Reader (fichier, réseau)
let file = std::fs::File::open("user.json")?;
let user: User = serde_json::from_reader(file)?;
```

### `serde_json::Value` — JSON dynamique

Quand la structure du JSON n'est pas connue à la compilation :

```rust
use serde_json::{json, Value};

// Construire du JSON dynamique avec la macro json!
let payload = json!({
    "name": "Alice",
    "scores": [10, 20, 30],
    "metadata": {
        "version": 2,
        "active": true
    }
});

// Navigation
let name = &payload["name"];              // → Value::String("Alice")
let score = &payload["scores"][0];        // → Value::Number(10)
let version = payload["metadata"]["version"].as_u64(); // → Some(2)

// get() retourne Option<&Value>
if let Some(v) = payload.get("missing_field") {
    println!("Présent: {v}");
}

// Mutation d'un objet JSON
let mut obj = json!({"a": 1});
if let Value::Object(map) = &mut obj {
    map.insert("b".to_string(), json!(2));
}

// Conversion Value → type typé
let value: Value = serde_json::from_str(input)?;
let user: User = serde_json::from_value(value)?;

// Type vers Value (utile pour manipuler avant de sérialiser)
let value = serde_json::to_value(&user)?;
```

### `RawValue` — parsing différé

```rust
use serde_json::value::RawValue;

// Utile quand vous recevez une enveloppe JSON et voulez
// passer la partie "data" à un autre système sans la re-parser
#[derive(Deserialize)]
struct Envelope<'a> {
    event_type: String,
    #[serde(borrow)]
    payload: &'a RawValue,  // Garde le JSON brut sans le parser
}

let input = r#"{"event_type":"click","payload":{"x":10,"y":20}}"#;
let envelope: Envelope = serde_json::from_str(input)?;
println!("Type: {}", envelope.event_type);
println!("Payload brut: {}", envelope.payload.get());
// Vous pouvez maintenant parser envelope.payload selon event_type
```

---

## 7. TOML avec la crate `toml`

```toml
# Cargo.toml
[dependencies]
toml = "0.8"
serde = { version = "1", features = ["derive"] }
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct AppConfig {
    database: DatabaseConfig,
    server: ServerConfig,
    features: Vec<String>,
}

#[derive(Debug, Serialize, Deserialize)]
struct DatabaseConfig {
    url: String,
    max_connections: u32,
    #[serde(default)]
    timeout_ms: u64,
}

#[derive(Debug, Serialize, Deserialize)]
struct ServerConfig {
    host: String,
    port: u16,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Lire depuis un fichier TOML
    let content = std::fs::read_to_string("config.toml")?;
    let config: AppConfig = toml::from_str(&content)?;

    // Écrire en TOML
    let toml_str = toml::to_string_pretty(&config)?;
    std::fs::write("config_backup.toml", toml_str)?;

    Ok(())
}
```

Fichier `config.toml` correspondant :

```toml
features = ["auth", "metrics", "cache"]

[database]
url = "postgres://localhost/mydb"
max_connections = 10

[server]
host = "0.0.0.0"
port = 8080
```

> [!tip] Pattern config app
> Combinez TOML pour la config de base + variables d'environnement pour les secrets (via la crate `config` ou `figment`). Le TOML va dans le dépôt Git ; les secrets comme `DATABASE_PASSWORD` viennent de l'env.

---

## 8. Sérialisation custom — implémenter les traits manuellement

### Implémenter `Serialize` manuellement

```rust
use serde::{ser::SerializeStruct, Serialize, Serializer};

struct Color {
    r: u8,
    g: u8,
    b: u8,
}

impl Serialize for Color {
    fn serialize<S: Serializer>(&self, serializer: S) -> Result<S::Ok, S::Error> {
        // Sérialiser comme un objet JSON à 3 champs
        let mut state = serializer.serialize_struct("Color", 3)?;
        state.serialize_field("r", &self.r)?;
        state.serialize_field("g", &self.g)?;
        state.serialize_field("b", &self.b)?;
        state.end()
    }
}

// Ou sérialiser comme une string "#RRGGBB"
struct HexColor(u8, u8, u8);

impl Serialize for HexColor {
    fn serialize<S: Serializer>(&self, serializer: S) -> Result<S::Ok, S::Error> {
        let hex = format!("#{:02X}{:02X}{:02X}", self.0, self.1, self.2);
        serializer.serialize_str(&hex)
    }
}
```

### Implémenter `Deserialize` manuellement — Visitor pattern complet

Exemple : parser une durée au format "1h30m" en `std::time::Duration` :

```rust
use serde::{
    de::{self, Visitor},
    Deserialize, Deserializer,
};
use std::fmt;
use std::time::Duration;

struct HumanDuration(Duration);

struct HumanDurationVisitor;

impl<'de> Visitor<'de> for HumanDurationVisitor {
    type Value = HumanDuration;

    fn expecting(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.write_str("une durée comme '1h30m', '45s', '2h', '30m15s'")
    }

    fn visit_str<E: de::Error>(self, value: &str) -> Result<Self::Value, E> {
        let mut remaining = value;
        let mut total_secs: u64 = 0;

        // Parser les heures
        if let Some(h_pos) = remaining.find('h') {
            let hours: u64 = remaining[..h_pos]
                .parse()
                .map_err(|_| E::custom(format!("heures invalides dans '{value}'")))?;
            total_secs += hours * 3600;
            remaining = &remaining[h_pos + 1..];
        }

        // Parser les minutes
        if let Some(m_pos) = remaining.find('m') {
            let mins: u64 = remaining[..m_pos]
                .parse()
                .map_err(|_| E::custom(format!("minutes invalides dans '{value}'")))?;
            total_secs += mins * 60;
            remaining = &remaining[m_pos + 1..];
        }

        // Parser les secondes
        if let Some(s_pos) = remaining.find('s') {
            let secs: u64 = remaining[..s_pos]
                .parse()
                .map_err(|_| E::custom(format!("secondes invalides dans '{value}'")))?;
            total_secs += secs;
        }

        Ok(HumanDuration(Duration::from_secs(total_secs)))
    }
}

impl<'de> Deserialize<'de> for HumanDuration {
    fn deserialize<D: Deserializer<'de>>(deserializer: D) -> Result<Self, D::Error> {
        deserializer.deserialize_str(HumanDurationVisitor)
    }
}

// Utilisation
#[derive(Deserialize)]
struct RateLimit {
    requests: u32,
    #[serde(deserialize_with = "deserialize_duration")]
    window: Duration,
}

fn deserialize_duration<'de, D: Deserializer<'de>>(d: D) -> Result<Duration, D::Error> {
    let hd = HumanDuration::deserialize(d)?;
    Ok(hd.0)
}
```

---

## 9. Patterns avancés

### Zero-copy avec `&'de str` et `Cow`

```rust
use std::borrow::Cow;

// Emprunter la string du buffer d'entrée — zéro allocation !
#[derive(Deserialize)]
struct ZeroCopyUser<'a> {
    // &'a str emprunte directement depuis le buffer JSON
    // Fonctionne avec from_str(), échoue avec from_reader()
    #[serde(borrow)]
    name: &'a str,

    // Cow : emprunte si possible, alloue si nécessaire
    // Fonctionne avec tous les backends
    email: Cow<'a, str>,
}

fn parse_users(json: &str) -> Vec<ZeroCopyUser> {
    // Les strings dans les structs pointent vers `json`, pas de copie
    serde_json::from_str(json).unwrap()
}
```

> [!warning] Limite du zero-copy
> `&'de str` fonctionne uniquement avec des inputs qui vivent assez longtemps (comme `from_str`). Avec `from_reader` (streaming), Serde ne peut pas garantir que le buffer reste valide — il faudra `String` ou `Cow<str>`.

### Champ absent vs `null`

```rust
// Ces deux structs se comportent différemment !

#[derive(Deserialize)]
struct WithOption {
    // null → Some(None) ... non, c'est juste None
    // absent → None
    // "value" → Some("value")
    field: Option<String>,  // Même comportement pour absent ET null
}

// Pour distinguer "absent" de "null", utiliser serde_with ou un type custom
use serde_with::serde_as;

#[serde_as]
#[derive(Deserialize)]
struct WithExplicitNull {
    // None = absent, Some(None) = null, Some(Some(v)) = valeur
    #[serde_as(as = "Option<Option<_>>")]
    field: Option<Option<String>>,
}
```

### `serde_with` — conversions pratiques

```rust
use serde_with::{serde_as, DurationSeconds, DisplayFromStr, base64::Base64};
use std::time::Duration;

#[serde_as]
#[derive(Serialize, Deserialize)]
struct Config {
    // Duration ↔ nombre de secondes en JSON
    #[serde_as(as = "DurationSeconds<u64>")]
    timeout: Duration,

    // Tout type Display/FromStr ↔ string JSON
    #[serde_as(as = "DisplayFromStr")]
    port: std::net::SocketAddr,

    // Vec<u8> ↔ base64 string
    #[serde_as(as = "Base64")]
    secret_key: Vec<u8>,

    // HashMap avec clés non-string (Serde normal ne supporte pas)
    #[serde_as(as = "Vec<(_, _)>")]
    map_with_int_keys: std::collections::HashMap<i32, String>,
}
```

---

## 10. Intégrations framework

### Axum — JSON extractors et responses

```rust
use axum::{
    extract::Json,
    routing::post,
    Router,
};
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

#[derive(Serialize)]
struct UserCreated {
    id: u64,
    name: String,
}

async fn create_user(Json(payload): Json<CreateUser>) -> Json<UserCreated> {
    // payload est déjà désérialisé par Axum via Serde
    Json(UserCreated {
        id: 42,
        name: payload.name,
    })
}
// Voir [[15 - Web Frameworks Axum et Actix]] pour plus de détails
```

### Configuration multi-format avec `figment`

```rust
use figment::{
    providers::{Env, Format, Toml},
    Figment,
};
use serde::Deserialize;

#[derive(Deserialize)]
struct Config {
    database_url: String,
    port: u16,
    debug: bool,
}

fn load_config() -> Config {
    Figment::new()
        .merge(Toml::file("config.toml"))      // Base : fichier TOML
        .merge(Env::prefixed("APP_"))           // Override : APP_PORT=9000, etc.
        .extract()
        .expect("Configuration invalide")
}
```

---

## 11. Gestion des erreurs Serde

Voir [[04 - Gestion des Erreurs en Rust]] pour le contexte général.

```rust
use serde_json::Error as JsonError;

fn parse_config(input: &str) -> Result<AppConfig, JsonError> {
    serde_json::from_str(input)
    // L'erreur contient la ligne et la colonne du problème
}

// Convertir les erreurs Serde en erreurs custom
#[derive(Debug, thiserror::Error)]
enum AppError {
    #[error("Configuration invalide : {0}")]
    Config(#[from] serde_json::Error),

    #[error("IO error : {0}")]
    Io(#[from] std::io::Error),
}

fn load_config_file(path: &str) -> Result<AppConfig, AppError> {
    let content = std::fs::read_to_string(path)?;  // AppError::Io
    let config = serde_json::from_str(&content)?;   // AppError::Config
    Ok(config)
}
```

---

## 12. Cargo.toml complet — projet serde

```toml
[package]
name = "serde-demo"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1", features = ["derive", "rc"] }  # "rc" pour Rc/Arc
serde_json = { version = "1", features = ["raw_value", "float_roundtrip"] }
toml = "0.8"
bincode = "1"
serde_with = { version = "3", features = ["base64", "chrono"] }
thiserror = "1"

[dev-dependencies]
criterion = "0.5"  # Benchmarks de sérialisation
```

---

## 13. Exercices pratiques

### Exercice 1 — API REST basique

Modélisez ces types JSON avec Serde :

```json
{
  "id": 1,
  "title": "Buy groceries",
  "completed": false,
  "created_at": 1716825600,
  "tags": ["personal", "urgent"],
  "assignee": null
}
```

Exigences :
- `created_at` doit être un `SystemTime` en Rust (utiliser `serialize_with`/`deserialize_with`)
- `assignee` : `Option<String>`, absent du JSON si `null` (utiliser `skip_serializing_if`)
- `tags` : absent du JSON si vide
- Le champ JSON s'appelle `created_at` mais le champ Rust s'appelle `created`

### Exercice 2 — Enum polymorphique

Implémentez un type `Notification` avec plusieurs variantes :

```json
{"type": "email", "to": "alice@example.com", "subject": "Hello"}
{"type": "sms", "phone": "+33612345678", "text": "Votre code: 1234"}
{"type": "push", "device_token": "abc123", "title": "Nouveau message", "body": "..."}
```

Utilisez `#[serde(tag = "type")]` et vérifiez la sérialisation/désérialisation dans les deux sens.

### Exercice 3 — Désérialisation custom

Implémentez un `Visitor` pour désérialiser des couleurs depuis plusieurs formats :
- `"red"` → couleur nommée
- `"#FF0000"` → hex RGB
- `[255, 0, 0]` → tableau RGB
- `{"r": 255, "g": 0, "b": 0}` → objet

Le `Visitor` doit implémenter `visit_str`, `visit_seq`, et `visit_map`.

### Exercice 4 — Configuration app

Créez un système de configuration pour une application web :
- Fichier `config.toml` avec les valeurs par défaut
- Variables d'environnement `APP_*` pour les overrides
- Struct `Config` avec validation (port entre 1024-65535, URL valide)
- Fonction `Config::load()` qui retourne `Result<Config, AppError>`

### Exercice 5 — Zero-copy benchmark

Comparez les performances de :
1. `String` dans les champs (toujours allocation)
2. `Cow<'a, str>` (allocation conditionnelle)
3. `&'a str` (jamais d'allocation)

Sur un payload JSON de 1000 objets. Utilisez `criterion` pour mesurer.

---

## Notes liées

- [[01 - Introduction a Rust]] — bases du langage nécessaires
- [[02 - Ownership et Borrowing]] — lifetimes et zero-copy avec `&'de str`
- [[04 - Gestion des Erreurs en Rust]] — `?`, `thiserror`, erreurs Serde
- [[06 - Traits et Generiques]] — `Serialize`/`Deserialize` sont des traits, bounds `where T: Serialize`
- [[09 - Async Await et Tokio]] — sérialisation dans les handlers async
- [[10 - Macros et Metaprogrammation]] — comment `#[derive(Serialize)]` fonctionne
- [[11 - Unsafe Rust et FFI]] — zero-copy et raw bytes
- [[15 - Web Frameworks Axum et Actix]] — `Json<T>` extractor, réponses JSON
- [[17 - Lifetimes Avances Pin et Unpin]] — `'de` lifetime dans `Deserialize<'de>`
- [[19 - Cargo Avance Workspaces et Publication]] — déclarer les features Serde dans Cargo.toml
