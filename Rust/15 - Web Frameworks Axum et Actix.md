# 15 - Web Frameworks Axum et Actix

> [!info] Prérequis
> Ce cours requiert la maîtrise de l'[[09 - Async Await et Tokio|async/await et Tokio]], des [[06 - Traits et Generiques|Traits et Génériques]], et de la [[04 - Gestion des Erreurs en Rust|gestion d'erreurs]]. Des notions d'API REST sont utiles (voir [[03 - Spring Boot et API REST]]).

---

## 1. Panorama des frameworks web Rust

| Framework | Modèle | Tokio | Stars (2025) | Points forts |
|---|---|---|---|---|
| **Axum** | Tower middleware | Oui | ★★★★★ | Ergonomie, type-safety, écosystème Tower |
| **Actix-web** | Acteur + async | Tokio optionnel | ★★★★★ | Performance, maturité, ecosystème riche |
| **Rocket** | Macro-driven | Tokio | ★★★★ | Ergonomie, zero boilerplate, forme |
| **Warp** | Filtres composables | Tokio | ★★★ | Composabilité fonctionnelle |
| **Poem** | Proc-macro OpenAPI | Tokio | ★★★ | Génération OpenAPI intégrée |

> [!tip] Choix recommandé 2025
> **Axum** pour les nouveaux projets — intégration native avec l'écosystème Tower (middleware universel), excellent support Tokio, ergonomie supérieure. **Actix-web** pour les projets à très haute performance ou avec une équipe existante sur actix.

---

## 2. Axum — Architecture et Concepts

Axum est construit sur [Tower](https://docs.rs/tower), un framework de middleware générique. Cela signifie que **tous les middlewares Tower fonctionnent avec Axum** (et vice-versa avec tout service Tower).

```toml
[dependencies]
axum            = { version = "0.8", features = ["macros"] }
tokio           = { version = "1", features = ["full"] }
tower           = "0.5"
tower-http      = { version = "0.6", features = ["cors", "trace", "compression-gzip"] }
serde           = { version = "1", features = ["derive"] }
serde_json      = "1"
tracing         = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
```

### 2.1 Application minimale

```rust
use axum::{routing::get, Router};
use std::net::SocketAddr;

#[tokio::main]
async fn main() {
    // Initialiser le logging
    tracing_subscriber::fmt::init();

    let app = Router::new()
        .route("/",       get(racine))
        .route("/sante",  get(sante))
        .route("/bonjour/:nom", get(bonjour));

    let adresse: SocketAddr = "0.0.0.0:3000".parse().unwrap();
    tracing::info!("Serveur démarré sur http://{}", adresse);

    let ecouteur = tokio::net::TcpListener::bind(adresse).await.unwrap();
    axum::serve(ecouteur, app).await.unwrap();
}

async fn racine() -> &'static str {
    "Bonjour depuis Axum !"
}

async fn sante() -> axum::http::StatusCode {
    axum::http::StatusCode::OK
}

// Extractor de paramètre de chemin
async fn bonjour(axum::extract::Path(nom): axum::extract::Path<String>) -> String {
    format!("Bonjour, {} !", nom)
}
```

---

## 3. Handlers et Types de Réponse

Un handler Axum est une fonction async qui retourne `impl IntoResponse`. Presque tout implémente `IntoResponse`.

```rust
use axum::{
    response::{IntoResponse, Response, Html},
    http::{StatusCode, header},
    Json,
};
use serde::{Deserialize, Serialize};

#[derive(Serialize)]
struct MessageReponse {
    message: String,
    code:    u32,
}

// Retourner du texte
async fn texte() -> &'static str { "Texte brut" }

// Retourner du HTML
async fn page_html() -> Html<&'static str> {
    Html("<h1>Bonjour HTML</h1>")
}

// Retourner du JSON
async fn json_handler() -> Json<MessageReponse> {
    Json(MessageReponse { message: "OK".to_string(), code: 200 })
}

// Retourner avec StatusCode
async fn cree() -> (StatusCode, Json<MessageReponse>) {
    (
        StatusCode::CREATED,
        Json(MessageReponse { message: "Créé".to_string(), code: 201 }),
    )
}

// Retourner avec headers custom
async fn avec_headers() -> impl IntoResponse {
    let headers = [(header::CONTENT_TYPE, "application/json")];
    let corps = r#"{"custom": true}"#;
    (StatusCode::OK, headers, corps)
}

// Retourner une Response complète
async fn response_complete() -> Response {
    Response::builder()
        .status(StatusCode::OK)
        .header(header::CONTENT_TYPE, "text/plain")
        .header("X-Custom-Header", "valeur")
        .body("Corps de la réponse".into())
        .unwrap()
}

// Retourner du binaire (fichier, image...)
async fn telecharger_fichier() -> impl IntoResponse {
    let contenu = b"Contenu du fichier";
    (
        StatusCode::OK,
        [(header::CONTENT_TYPE, "application/octet-stream"),
         (header::CONTENT_DISPOSITION, "attachment; filename=\"fichier.bin\"")],
        contenu.as_slice(),
    )
}
```

---

## 4. Extractors — Lire les Données de la Requête

Les extractors sont des types qui extraient des données de la requête HTTP. Ils s'ajoutent comme paramètres du handler.

### 4.1 Paramètres de chemin

```rust
use axum::extract::Path;
use std::collections::HashMap;

// Un seul paramètre
async fn user_par_id(Path(id): Path<u64>) -> String {
    format!("User #{}", id)
}

// Plusieurs paramètres
async fn article(Path((auteur, slug)): Path<(String, String)>) -> String {
    format!("Article {} de {}", slug, auteur)
}

// Avec une struct
#[derive(Deserialize)]
struct ParamsArticle { auteur: String, slug: String }

async fn article_struct(Path(params): Path<ParamsArticle>) -> String {
    format!("{}/{}", params.auteur, params.slug)
}

// Configuration des routes correspondantes
Router::new()
    .route("/users/:id",                     get(user_par_id))
    .route("/articles/:auteur/:slug",         get(article))
    .route("/articles2/:auteur/:slug",        get(article_struct));
```

### 4.2 Query Parameters

```rust
use axum::extract::Query;

#[derive(Deserialize)]
struct ParamsRecherche {
    q:     Option<String>,
    page:  Option<u32>,
    limit: Option<u32>,
}

async fn rechercher(Query(params): Query<ParamsRecherche>) -> Json<serde_json::Value> {
    let q     = params.q.unwrap_or_default();
    let page  = params.page.unwrap_or(1);
    let limit = params.limit.unwrap_or(20).min(100); // Sécurité : limiter à 100

    Json(serde_json::json!({
        "requete":  q,
        "page":     page,
        "limite":   limit,
        "resultats": []
    }))
}
// GET /rechercher?q=rust&page=2&limit=10
```

### 4.3 Corps JSON

```rust
use axum::extract::Json;

#[derive(Deserialize)]
struct CreerUser {
    nom:   String,
    email: String,
    age:   Option<u32>,
}

#[derive(Serialize)]
struct UserCree {
    id:    u64,
    nom:   String,
    email: String,
}

async fn creer_user(Json(payload): Json<CreerUser>) -> (StatusCode, Json<UserCree>) {
    // Validation basique
    if payload.email.is_empty() || payload.nom.is_empty() {
        // Sera amélioré avec les erreurs custom (§7)
    }
    
    let user = UserCree {
        id:    42, // En réalité : insérer en BDD et retourner l'ID
        nom:   payload.nom,
        email: payload.email,
    };
    
    (StatusCode::CREATED, Json(user))
}
```

### 4.4 State partagé

```rust
use axum::extract::State;
use std::sync::Arc;

// L'état de l'application
#[derive(Clone)]
struct AppState {
    db:     sqlx::PgPool,
    config: Arc<Config>,
}

#[derive(Clone)]
struct Config {
    api_key: String,
}

// Setup dans main()
async fn main() {
    let pool = sqlx::PgPool::connect("postgresql://localhost/madb").await.unwrap();
    let state = AppState {
        db:     pool,
        config: Arc::new(Config { api_key: "secret".to_string() }),
    };

    let app = Router::new()
        .route("/users", get(lister_users))
        .with_state(state); // Attacher le state
    
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}

async fn lister_users(State(state): State<AppState>) -> Json<Vec<String>> {
    // Utiliser state.db pour les requêtes
    Json(vec!["Alice".to_string(), "Bob".to_string()])
}
```

### 4.5 Headers, Form, Multipart

```rust
use axum::{
    extract::{Multipart, Form},
    http::HeaderMap,
    TypedHeader,
};
use axum_extra::headers::Authorization;
use axum_extra::headers::authorization::Bearer;

// Lire les headers
async fn avec_headers(headers: HeaderMap) -> String {
    let user_agent = headers
        .get("user-agent")
        .and_then(|v| v.to_str().ok())
        .unwrap_or("inconnu");
    format!("User-Agent : {}", user_agent)
}

// Formulaire HTML (application/x-www-form-urlencoded)
#[derive(Deserialize)]
struct FormulaireLogin { nom: String, mdp: String }

async fn login(Form(data): Form<FormulaireLogin>) -> String {
    format!("Login tenté pour : {}", data.nom)
}

// Upload de fichier (multipart/form-data)
async fn upload(mut multipart: Multipart) -> Result<String, String> {
    while let Some(field) = multipart.next_field().await.map_err(|e| e.to_string())? {
        let nom_champ = field.name().unwrap_or("?").to_string();
        let nom_fichier = field.file_name().unwrap_or("?").to_string();
        let data = field.bytes().await.map_err(|e| e.to_string())?;
        
        tracing::info!("Reçu champ '{}', fichier '{}', taille {}", nom_champ, nom_fichier, data.len());
        
        // Sauvegarder le fichier
        tokio::fs::write(format!("uploads/{}", nom_fichier), data)
            .await.map_err(|e| e.to_string())?;
    }
    Ok("Upload réussi".to_string())
}
```

---

## 5. Router — Organisation des Routes

```rust
use axum::{routing::{get, post, put, delete, patch}, Router};

// Sous-routeur réutilisable — bonne pratique pour les grandes apps
fn routes_users() -> Router<AppState> {
    Router::new()
        .route("/",           get(lister_users).post(creer_user))
        .route("/:id",        get(obtenir_user).put(modifier_user).delete(supprimer_user))
        .route("/:id/avatar", post(upload_avatar))
}

fn routes_articles() -> Router<AppState> {
    Router::new()
        .route("/",       get(lister_articles).post(creer_article))
        .route("/:slug",  get(obtenir_article).patch(modifier_article))
}

fn routes_api() -> Router<AppState> {
    Router::new()
        .nest("/users",    routes_users())
        .nest("/articles", routes_articles())
}

async fn main_setup(state: AppState) -> Router {
    Router::new()
        .nest("/api/v1", routes_api())
        .route("/health", get(|| async { "OK" }))
        .with_state(state)
}
```

---

## 6. Middleware avec Tower

```rust
use tower::ServiceBuilder;
use tower_http::{
    cors::{CorsLayer, Any},
    trace::{TraceLayer, DefaultMakeSpan, DefaultOnResponse},
    compression::CompressionLayer,
    limit::RequestBodyLimitLayer,
    timeout::TimeoutLayer,
};
use std::time::Duration;

async fn configurer_middleware(app: Router) -> Router {
    let cors = CorsLayer::new()
        .allow_origin(Any)
        .allow_methods([axum::http::Method::GET, axum::http::Method::POST,
                        axum::http::Method::PUT, axum::http::Method::DELETE])
        .allow_headers(Any);

    app.layer(
        ServiceBuilder::new()
            // Timeout global
            .layer(TimeoutLayer::new(Duration::from_secs(30)))
            // Limite la taille du corps (5 MB)
            .layer(RequestBodyLimitLayer::new(5 * 1024 * 1024))
            // Compression automatique (gzip, brotli)
            .layer(CompressionLayer::new())
            // CORS
            .layer(cors)
            // Logging automatique de chaque requête
            .layer(TraceLayer::new_for_http()
                .make_span_with(DefaultMakeSpan::new().level(tracing::Level::INFO))
                .on_response(DefaultOnResponse::new().level(tracing::Level::INFO))
            ),
    )
}
```

### Middleware personnalisé

```rust
use axum::{
    middleware::{self, Next},
    extract::Request,
    response::Response,
};

// Middleware d'authentification basique
async fn middleware_auth(
    State(state): State<AppState>,
    mut req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    let token = req.headers()
        .get("Authorization")
        .and_then(|v| v.to_str().ok())
        .and_then(|v| v.strip_prefix("Bearer "));

    match token {
        Some(t) if valider_token(t, &state).await => {
            // Injecter l'utilisateur dans les extensions de la requête
            // req.extensions_mut().insert(user_info);
            Ok(next.run(req).await)
        }
        _ => Err(StatusCode::UNAUTHORIZED),
    }
}

async fn valider_token(token: &str, state: &AppState) -> bool {
    // Vérifier le JWT, consulter la BDD, etc.
    !token.is_empty() // Simplifié
}

// Appliquer le middleware à certaines routes seulement
let routes_protegees = Router::new()
    .route("/profil", get(obtenir_profil))
    .route_layer(middleware::from_fn_with_state(state.clone(), middleware_auth));

let app = Router::new()
    .route("/login", post(login))
    .merge(routes_protegees)
    .with_state(state);
```

---

## 7. Gestion des Erreurs

```rust
use axum::{
    response::{IntoResponse, Response},
    http::StatusCode,
    Json,
};
use serde_json::json;

// Erreur applicative typée
#[derive(Debug)]
pub enum ErreurApp {
    NonTrouve(String),
    NonAutorise,
    DonneesInvalides(String),
    BDD(sqlx::Error),
    Interne(anyhow::Error),
}

// Convertir l'erreur en réponse HTTP
impl IntoResponse for ErreurApp {
    fn into_response(self) -> Response {
        let (statut, message) = match self {
            ErreurApp::NonTrouve(msg)         => (StatusCode::NOT_FOUND, msg),
            ErreurApp::NonAutorise            => (StatusCode::UNAUTHORIZED, "Non autorisé".to_string()),
            ErreurApp::DonneesInvalides(msg)  => (StatusCode::BAD_REQUEST, msg),
            ErreurApp::BDD(e) => {
                tracing::error!("Erreur BDD : {:?}", e);
                (StatusCode::INTERNAL_SERVER_ERROR, "Erreur de base de données".to_string())
            }
            ErreurApp::Interne(e) => {
                tracing::error!("Erreur interne : {:?}", e);
                (StatusCode::INTERNAL_SERVER_ERROR, "Erreur interne".to_string())
            }
        };

        let corps = json!({
            "erreur": message,
            "code":   statut.as_u16(),
        });

        (statut, Json(corps)).into_response()
    }
}

// Convertions pratiques
impl From<sqlx::Error> for ErreurApp {
    fn from(e: sqlx::Error) -> Self {
        match e {
            sqlx::Error::RowNotFound => ErreurApp::NonTrouve("Enregistrement introuvable".to_string()),
            _ => ErreurApp::BDD(e),
        }
    }
}

// Usage dans les handlers — propre grâce à l'opérateur ?
async fn obtenir_user_bdd(
    Path(id): Path<i64>,
    State(state): State<AppState>,
) -> Result<Json<User>, ErreurApp> {
    if id <= 0 {
        return Err(ErreurApp::DonneesInvalides("ID invalide".to_string()));
    }

    let user = sqlx::query_as!(User, "SELECT * FROM users WHERE id = $1", id)
        .fetch_one(&state.db)
        .await?; // Convertit sqlx::Error → ErreurApp via From

    Ok(Json(user))
}
```

---

## 8. SQLx avec Axum — Base de Données

```toml
[dependencies]
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "uuid", "chrono", "migrate"] }
uuid = { version = "1", features = ["v4", "serde"] }
chrono = { version = "0.4", features = ["serde"] }
```

### 8.1 Connexion et Pool

```rust
use sqlx::PgPool;

async fn creer_pool() -> PgPool {
    let database_url = std::env::var("DATABASE_URL")
        .expect("DATABASE_URL doit être défini");

    PgPool::connect_with(
        database_url.parse::<sqlx::postgres::PgConnectOptions>().unwrap()
            .application_name("mon-app")
    )
    .await
    .expect("Impossible de se connecter à PostgreSQL")
}
```

### 8.2 Migrations

```bash
# Structure des migrations
migrations/
├── 20240101000001_creer_users.sql
├── 20240101000002_creer_articles.sql
└── 20240201000001_ajouter_colonne_bio.sql
```

```sql
-- migrations/20240101000001_creer_users.sql
CREATE TABLE users (
    id         BIGSERIAL PRIMARY KEY,
    uuid       UUID NOT NULL DEFAULT gen_random_uuid(),
    nom        VARCHAR(100) NOT NULL,
    email      VARCHAR(255) NOT NULL UNIQUE,
    mdp_hash   VARCHAR(255) NOT NULL,
    bio        TEXT,
    cree_le    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    modifie_le TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_users_email ON users(email);
```

```rust
// Appliquer les migrations au démarrage
sqlx::migrate!("./migrations")
    .run(&pool)
    .await
    .expect("Erreur lors des migrations");
```

### 8.3 Requêtes typées

```rust
use sqlx::FromRow;
use chrono::{DateTime, Utc};
use uuid::Uuid;

#[derive(Debug, FromRow, Serialize)]
struct User {
    id:         i64,
    uuid:       Uuid,
    nom:        String,
    email:      String,
    bio:        Option<String>,
    cree_le:    DateTime<Utc>,
    modifie_le: DateTime<Utc>,
}

// query_as! — vérification de type à la compilation
async fn obtenir_user(pool: &PgPool, id: i64) -> sqlx::Result<User> {
    sqlx::query_as!(
        User,
        "SELECT id, uuid, nom, email, bio, cree_le, modifie_le FROM users WHERE id = $1",
        id
    )
    .fetch_one(pool)
    .await
}

// Lister avec pagination
async fn lister_users(pool: &PgPool, page: i64, limite: i64) -> sqlx::Result<Vec<User>> {
    let offset = (page - 1) * limite;
    sqlx::query_as!(
        User,
        r#"
        SELECT id, uuid, nom, email, bio, cree_le, modifie_le
        FROM users
        ORDER BY cree_le DESC
        LIMIT $1 OFFSET $2
        "#,
        limite, offset
    )
    .fetch_all(pool)
    .await
}

// INSERT avec retour de l'ID
async fn inserer_user(pool: &PgPool, nom: &str, email: &str, mdp_hash: &str) -> sqlx::Result<i64> {
    let row = sqlx::query!(
        "INSERT INTO users (nom, email, mdp_hash) VALUES ($1, $2, $3) RETURNING id",
        nom, email, mdp_hash
    )
    .fetch_one(pool)
    .await?;
    Ok(row.id)
}

// Transaction
async fn transfert_credits(pool: &PgPool, de: i64, vers: i64, montant: i64) -> sqlx::Result<()> {
    let mut tx = pool.begin().await?;
    
    sqlx::query!("UPDATE users SET credits = credits - $1 WHERE id = $2", montant, de)
        .execute(&mut *tx).await?;
    sqlx::query!("UPDATE users SET credits = credits + $1 WHERE id = $2", montant, vers)
        .execute(&mut *tx).await?;
    
    tx.commit().await?;
    Ok(())
}
```

---

## 9. API REST Complète avec JWT

```toml
[dependencies]
jsonwebtoken = "9"
bcrypt       = "0.15"
validator    = { version = "0.18", features = ["derive"] }
```

```rust
// === Authentication JWT ===
use jsonwebtoken::{encode, decode, Header, Algorithm, Validation, EncodingKey, DecodingKey};
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct ClaimsJWT {
    sub:  String,  // subject = user ID
    exp:  usize,   // expiration (timestamp Unix)
    iat:  usize,   // issued at
    role: String,
}

fn creer_token(user_id: i64, role: &str, secret: &str) -> Result<String, jsonwebtoken::errors::Error> {
    let maintenant  = std::time::SystemTime::now().duration_since(std::time::UNIX_EPOCH).unwrap().as_secs() as usize;
    let expiration  = maintenant + 24 * 3600; // 24h

    let claims = ClaimsJWT {
        sub:  user_id.to_string(),
        exp:  expiration,
        iat:  maintenant,
        role: role.to_string(),
    };

    encode(&Header::default(), &claims, &EncodingKey::from_secret(secret.as_bytes()))
}

fn verifier_token(token: &str, secret: &str) -> Result<ClaimsJWT, jsonwebtoken::errors::Error> {
    let mut validation = Validation::new(Algorithm::HS256);
    validation.validate_exp = true;

    let donnees = decode::<ClaimsJWT>(
        token,
        &DecodingKey::from_secret(secret.as_bytes()),
        &validation,
    )?;
    Ok(donnees.claims)
}

// Handler de login
#[derive(Deserialize)]
struct DemandeLogin { email: String, mdp: String }

#[derive(Serialize)]
struct ReponseLogin { token: String, expire_dans: u64 }

async fn login_handler(
    State(state): State<AppState>,
    Json(payload): Json<DemandeLogin>,
) -> Result<Json<ReponseLogin>, ErreurApp> {
    // Chercher l'utilisateur
    let user = sqlx::query!(
        "SELECT id, mdp_hash, role FROM users WHERE email = $1",
        payload.email
    )
    .fetch_optional(&state.db)
    .await
    .map_err(ErreurApp::BDD)?
    .ok_or_else(|| ErreurApp::NonAutorise)?;

    // Vérifier le mot de passe
    let valide = bcrypt::verify(&payload.mdp, &user.mdp_hash)
        .map_err(|e| ErreurApp::Interne(e.into()))?;

    if !valide {
        return Err(ErreurApp::NonAutorise);
    }

    // Générer le JWT
    let token = creer_token(user.id, &user.role, &state.config.jwt_secret)
        .map_err(|e| ErreurApp::Interne(e.into()))?;

    Ok(Json(ReponseLogin { token, expire_dans: 86400 }))
}

// Extractor JWT personnalisé
use axum::{async_trait, extract::FromRequestParts, http::request::Parts};

struct UtilisateurAuthentifie {
    id:   i64,
    role: String,
}

#[async_trait]
impl<S> FromRequestParts<S> for UtilisateurAuthentifie
where
    S: Send + Sync,
    AppState: axum::extract::FromRef<S>,
{
    type Rejection = ErreurApp;

    async fn from_request_parts(parts: &mut Parts, state: &S) -> Result<Self, Self::Rejection> {
        let app_state = AppState::from_ref(state);
        
        let token = parts
            .headers
            .get("Authorization")
            .and_then(|v| v.to_str().ok())
            .and_then(|v| v.strip_prefix("Bearer "))
            .ok_or(ErreurApp::NonAutorise)?;

        let claims = verifier_token(token, &app_state.config.jwt_secret)
            .map_err(|_| ErreurApp::NonAutorise)?;

        Ok(UtilisateurAuthentifie {
            id:   claims.sub.parse().map_err(|_| ErreurApp::NonAutorise)?,
            role: claims.role,
        })
    }
}

// Utiliser l'extractor dans un handler
async fn profil(
    utilisateur: UtilisateurAuthentifie,
    State(state): State<AppState>,
) -> Result<Json<User>, ErreurApp> {
    let user = obtenir_user(&state.db, utilisateur.id).await?;
    Ok(Json(user))
}
```

---

## 10. WebSockets avec Axum

```toml
[dependencies]
axum = { version = "0.8", features = ["ws"] }
tokio = { version = "1", features = ["full"] }
futures = "0.3"
```

```rust
use axum::{
    extract::ws::{WebSocket, WebSocketUpgrade, Message},
    response::Response,
};
use futures::{StreamExt, SinkExt};
use std::sync::Arc;
use tokio::sync::broadcast;

// État partagé pour le broadcast
#[derive(Clone)]
struct EtatWs {
    sender: broadcast::Sender<String>,
}

// Handler WebSocket
async fn ws_handler(
    ws:    WebSocketUpgrade,
    State(state): State<Arc<EtatWs>>,
) -> Response {
    ws.on_upgrade(move |socket| gerer_connexion(socket, state))
}

async fn gerer_connexion(socket: WebSocket, state: Arc<EtatWs>) {
    let (mut sender, mut receiver) = socket.split();
    let mut abonnement = state.sender.subscribe();

    // Task d'envoi — reçoit les messages broadcast et les envoie au client
    let mut task_envoi = tokio::spawn(async move {
        while let Ok(msg) = abonnement.recv().await {
            if sender.send(Message::Text(msg.into())).await.is_err() {
                break; // Client déconnecté
            }
        }
    });

    // Task de réception — lit les messages du client
    let state_clone   = state.clone();
    let mut task_reception = tokio::spawn(async move {
        while let Some(Ok(msg)) = receiver.next().await {
            match msg {
                Message::Text(texte) => {
                    tracing::info!("Message reçu : {}", texte);
                    // Diffuser à tous les connectés
                    let _ = state_clone.sender.send(texte.to_string());
                }
                Message::Binary(data) => {
                    tracing::info!("Binaire reçu : {} octets", data.len());
                }
                Message::Close(_) => break,
                Message::Ping(data) => {
                    // Le pong est envoyé automatiquement
                    tracing::debug!("Ping reçu");
                }
                _ => {}
            }
        }
    });

    // Attendre que l'une des deux tasks se termine
    tokio::select! {
        _ = &mut task_envoi     => task_reception.abort(),
        _ = &mut task_reception => task_envoi.abort(),
    }

    tracing::info!("Client WebSocket déconnecté");
}

// Setup dans main
async fn main() {
    let (tx, _) = broadcast::channel(100);
    let etat_ws = Arc::new(EtatWs { sender: tx });

    let app = Router::new()
        .route("/ws", get(ws_handler))
        .with_state(etat_ws);
    
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

---

## 11. Actix-web

```toml
[dependencies]
actix-web  = "4"
actix-rt   = "2"
serde      = { version = "1", features = ["derive"] }
serde_json = "1"
```

### 11.1 Structure de base

```rust
use actix_web::{web, App, HttpServer, HttpResponse, middleware};
use actix_web::web::{Data, Json, Path, Query};

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap(actix_web::middleware::Logger::default())
            .wrap(actix_cors::Cors::default().allow_any_origin())
            // State partagé
            .app_data(Data::new(mon_etat()))
            // Routes
            .service(
                web::scope("/api")
                    .route("/users",     web::get().to(lister_users))
                    .route("/users",     web::post().to(creer_user))
                    .route("/users/{id}", web::get().to(obtenir_user))
            )
    })
    .bind("0.0.0.0:3000")?
    .run()
    .await
}

// Handler avec extractors actix
async fn obtenir_user(
    path:  Path<u64>,
    state: Data<AppState>,
) -> HttpResponse {
    let id = path.into_inner();
    // Simuler une requête BDD
    if id == 0 {
        return HttpResponse::NotFound().json(serde_json::json!({ "erreur": "Non trouvé" }));
    }
    HttpResponse::Ok().json(serde_json::json!({ "id": id, "nom": "Alice" }))
}

async fn creer_user(
    body:  Json<CreerUser>,
    state: Data<AppState>,
) -> HttpResponse {
    HttpResponse::Created().json(serde_json::json!({ "id": 1, "nom": body.nom }))
}
```

### 11.2 Axum vs Actix-web — Comparaison directe

| Aspect | Axum 0.8 | Actix-web 4 |
|---|---|---|
| **Runtime** | Tokio uniquement | Tokio (actix-rt) |
| **Middleware** | Tower (universel) | Actix middleware (propriétaire) |
| **Ergonomie** | Type-safe, extractors composables | Macro `#[get]` optionnelle, très ergonomique aussi |
| **Performances** | Très élevées | Légèrement supérieures (benchmarks TechEmpower) |
| **Taille ecosystem** | En croissance | Mature, plus de plugins |
| **WebSocket** | Intégré (feature ws) | `actix-ws` crate |
| **Acteur model** | Non | Historique (optionnel en v4) |
| **State** | `with_state()` + extractor | `Data<T>` (Arc implicite) |
| **Erreurs** | `impl IntoResponse` | `ResponseError` trait |
| **Compilation** | Rapide | Plus lente (macros) |

---

## 12. Testing

### 12.1 Tests unitaires des handlers

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use axum::{
        body::Body,
        http::{Request, StatusCode},
    };
    use tower::ServiceExt; // Pour .oneshot()
    use serde_json::Value;

    fn creer_app_test() -> Router {
        let state = AppState {
            db:     creer_pool_test().await, // Pool SQLite en mémoire pour les tests
            config: Arc::new(Config::default()),
        };
        Router::new()
            .route("/users/:id", get(obtenir_user_bdd))
            .with_state(state)
    }

    #[tokio::test]
    async fn test_sante() {
        let app = Router::new().route("/health", get(|| async { "OK" }));

        let reponse = app
            .oneshot(Request::builder()
                .uri("/health")
                .body(Body::empty()).unwrap())
            .await.unwrap();

        assert_eq!(reponse.status(), StatusCode::OK);
    }

    #[tokio::test]
    async fn test_creer_user_json_invalide() {
        let app = creer_app_test();

        let reponse = app
            .oneshot(
                Request::builder()
                    .method("POST")
                    .uri("/users")
                    .header("content-type", "application/json")
                    .body(Body::from(r#"{"email_invalide": true}"#)).unwrap()
            )
            .await.unwrap();

        assert_eq!(reponse.status(), StatusCode::UNPROCESSABLE_ENTITY);
    }

    #[tokio::test]
    async fn test_lister_users_retourne_json() {
        let app = creer_app_test();

        let reponse = app
            .oneshot(Request::builder()
                .uri("/users")
                .body(Body::empty()).unwrap())
            .await.unwrap();

        assert_eq!(reponse.status(), StatusCode::OK);
        
        let corps = axum::body::to_bytes(reponse.into_body(), usize::MAX).await.unwrap();
        let json: Value = serde_json::from_slice(&corps).unwrap();
        assert!(json.is_array());
    }
}
```

### 12.2 Tests d'intégration avec reqwest

```rust
// tests/integration_test.rs
use reqwest::Client;

#[tokio::test]
async fn test_flow_complet() {
    // Démarrer le serveur en arrière-plan
    tokio::spawn(async { demarrer_serveur("0.0.0.0:3001").await.unwrap() });
    tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
    
    let client = Client::new();
    let base = "http://localhost:3001";

    // 1. Créer un utilisateur
    let reponse = client.post(format!("{}/api/users", base))
        .json(&serde_json::json!({ "nom": "Alice", "email": "alice@test.com", "mdp": "secret" }))
        .send().await.unwrap();
    assert_eq!(reponse.status(), 201);
    let user: serde_json::Value = reponse.json().await.unwrap();
    let user_id = user["id"].as_i64().unwrap();

    // 2. Login
    let reponse = client.post(format!("{}/api/auth/login", base))
        .json(&serde_json::json!({ "email": "alice@test.com", "mdp": "secret" }))
        .send().await.unwrap();
    assert_eq!(reponse.status(), 200);
    let auth: serde_json::Value = reponse.json().await.unwrap();
    let token = auth["token"].as_str().unwrap().to_string();

    // 3. Accéder au profil avec le token
    let reponse = client.get(format!("{}/api/users/{}", base, user_id))
        .header("Authorization", format!("Bearer {}", token))
        .send().await.unwrap();
    assert_eq!(reponse.status(), 200);
}
```

---

## 13. Graceful Shutdown et Déploiement

### 13.1 Graceful Shutdown

```rust
use tokio::signal;

async fn main() {
    let app = creer_app();
    
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    
    axum::serve(listener, app)
        .with_graceful_shutdown(signal_arret())
        .await
        .unwrap();
    
    tracing::info!("Serveur arrêté proprement");
}

async fn signal_arret() {
    let ctrl_c = async {
        signal::ctrl_c().await.expect("Erreur CTRL+C");
    };
    
    #[cfg(unix)]
    let termine = async {
        signal::unix::signal(signal::unix::SignalKind::terminate())
            .expect("Erreur SIGTERM")
            .recv()
            .await;
    };
    
    #[cfg(not(unix))]
    let termine = std::future::pending::<()>();
    
    tokio::select! {
        _ = ctrl_c  => tracing::info!("CTRL+C reçu"),
        _ = termine => tracing::info!("SIGTERM reçu"),
    }
}
```

### 13.2 Dockerfile multi-stage

```dockerfile
# ====== Stage 1 : Build ======
FROM rust:1.80-slim AS builder

RUN apt-get update && apt-get install -y pkg-config libssl-dev && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY Cargo.toml Cargo.lock ./
# Cache des dépendances (évite de re-télécharger si seul le code change)
RUN mkdir src && echo "fn main(){}" > src/main.rs
RUN cargo build --release
RUN rm -f target/release/deps/mon_app*

COPY src ./src
RUN cargo build --release

# ====== Stage 2 : Runtime ======
FROM debian:bookworm-slim AS runtime

RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY --from=builder /app/target/release/mon_app .
COPY migrations ./migrations

# Utilisateur non-root
RUN useradd -ms /bin/bash appuser
USER appuser

ENV RUST_LOG=info
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:3000/health || exit 1

ENTRYPOINT ["./mon_app"]
```

---

## 14. Tracing et Observabilité

```toml
[dependencies]
tracing             = "0.1"
tracing-subscriber  = { version = "0.3", features = ["env-filter", "json"] }
tracing-opentelemetry = "0.26"  # Optionnel — OpenTelemetry
```

```rust
fn initialiser_logging() {
    // En développement — format humain
    #[cfg(debug_assertions)]
    tracing_subscriber::fmt()
        .with_env_filter(
            tracing_subscriber::EnvFilter::try_from_default_env()
                .unwrap_or_else(|_| "mon_app=debug,axum=info,tower_http=info".into())
        )
        .with_target(true)
        .with_line_number(true)
        .init();

    // En production — JSON structuré (compatible ELK, Datadog, etc.)
    #[cfg(not(debug_assertions))]
    tracing_subscriber::fmt()
        .json()
        .with_env_filter("info")
        .init();
}

// Usage dans les handlers
async fn creer_article(
    utilisateur: UtilisateurAuthentifie,
    Json(payload): Json<NouvelArticle>,
    State(state): State<AppState>,
) -> Result<Json<Article>, ErreurApp> {
    let span = tracing::info_span!("creer_article",
        user_id  = utilisateur.id,
        titre    = payload.titre.as_str()
    );
    let _guard = span.enter();

    tracing::info!("Création d'un article");
    
    let article = inserer_article(&state.db, &payload, utilisateur.id)
        .await
        .map_err(|e| {
            tracing::error!(erreur = ?e, "Échec insertion article");
            ErreurApp::BDD(e)
        })?;

    tracing::info!(article_id = article.id, "Article créé");
    Ok(Json(article))
}
```

---

## 15. Récapitulatif

### Routes Axum — cheatsheet rapide

```rust
Router::new()
    // Méthodes HTTP
    .route("/chemin", get(handler))
    .route("/chemin", post(handler))
    .route("/chemin", put(handler))
    .route("/chemin", delete(handler))
    .route("/chemin", patch(handler))
    // Même chemin, méthodes multiples
    .route("/items", get(lister).post(creer))
    .route("/items/:id", get(lire).put(modifier).delete(supprimer))
    // Nesting
    .nest("/api/v1", sous_routeur)
    // State
    .with_state(etat)
    // Middleware global
    .layer(TraceLayer::new_for_http())
    // Middleware sur certaines routes
    .route_layer(middleware::from_fn(mon_middleware))
```

### Extractors fréquents

| Extractor | Ce qu'il extrait | Exemple |
|---|---|---|
| `Path<T>` | Paramètre d'URL `:id` | `Path(id): Path<u64>` |
| `Query<T>` | Query string `?k=v` | `Query(q): Query<Params>` |
| `Json<T>` | Corps JSON | `Json(body): Json<MyStruct>` |
| `State<T>` | État partagé | `State(s): State<AppState>` |
| `HeaderMap` | Tous les headers | `headers: HeaderMap` |
| `Form<T>` | Form URL-encoded | `Form(f): Form<MyForm>` |
| `Multipart` | Upload fichier | `mut mp: Multipart` |
| `WebSocketUpgrade` | Upgrade WebSocket | `ws: WebSocketUpgrade` |

> [!tip] Ressources
> - [docs.rs/axum](https://docs.rs/axum) — API complète avec exemples
> - [Exemples axum officiels](https://github.com/tokio-rs/axum/tree/main/examples) — 50+ exemples complets
> - [SQLx cookbook](https://github.com/launchbadge/sqlx) — requêtes avancées
> - [Tower middleware guide](https://docs.rs/tower/latest/tower/) — middlewares réutilisables

---

## Notes liées

### Rust — fondations
- [[09 - Async Await et Tokio]] — runtime Tokio qui fait tourner Axum et Actix
- [[04 - Gestion des Erreurs en Rust]] — `Result`, `?`, erreurs custom avec `impl IntoResponse`
- [[08 - Concurrence et Smart Pointers]] — `Arc<T>` pour le state partagé entre handlers
- [[06 - Traits et Generiques]] — `FromRequest`, `IntoResponse`, extractors génériques Axum

### Comparaison avec autres frameworks web
- [[08 - APIs REST avec Flask]] — Flask Python : même paradigme, comparaison utile
- [[09 - APIs REST avec FastAPI]] — FastAPI : async Python, le plus proche d'Axum en ergonomie
- [[03 - Spring Boot et API REST]] — Spring Boot Java : enterprise, JPA, annotations
- [[01 - Express.js et NestJS]] — Express/NestJS : Node.js, middleware pattern similaire

### Base de données et persistence
- [[01 - Introduction au SQL]] — SQL fondamentaux, requêtes utilisées avec SQLx
- [[10 - Python et Bases de Donnees]] — SQLAlchemy vs SQLx : comparaison ORM/query builder
- [[03 - Redis et Caches]] — Redis pour le cache et les sessions côté Axum

### Sécurité et déploiement
- [[02 - Securite Web OWASP]] — OWASP Top 10, JWT, CORS, rate limiting
- [[05 - WebSockets]] — WebSockets : protocole, puis implémentation Axum
- [[01 - Docker]] — Dockerfile multi-stage pour un binaire Axum minimal
- [[04 - CI-CD avec GitHub Actions]] — pipeline CI : test → clippy → build → docker → deploy
- [[14 - WebAssembly avec Rust]] — servir du WASM depuis Axum
- [[13 - Tauri Desktop Applications]] — Axum comme backend embarqué dans une app Tauri
