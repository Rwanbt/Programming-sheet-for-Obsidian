# 09 - Async Await et Tokio

> [!info] Prérequis
> Ce module nécessite de comprendre [[08 - Concurrence et Smart Pointers]] (threads, channels) et [[04 - Gestion des Erreurs en Rust]] (Result, ?). L'async Rust est une alternative aux threads OS — pas un remplacement.

---

## 1. Pourquoi l'async ?

### I/O Bound vs CPU Bound

| Type de tâche | Exemple | Solution recommandée |
|---------------|---------|---------------------|
| **I/O Bound** | Requêtes HTTP, lecture fichier, base de données | Async/await avec Tokio |
| **CPU Bound** | Calcul numérique, compression, hachage | Threads OS + Rayon |
| **Mixed** | Serveur web avec calculs | Tokio + `spawn_blocking` |

### Threads OS vs Async Tasks

```
Thread OS :
- Mémoire stack : ~8 MB par thread (fixe, allouée à la création)
- Scheduling : par le kernel OS (context switch coûteux ~1-10 μs)
- Limite pratique : quelques milliers de threads

Async Task :
- Mémoire : ~100 bytes par task (taille exacte de la state machine)
- Scheduling : par le runtime Rust (context switch ~quelques ns)
- Limite pratique : des millions de tasks simultanées
```

> [!tip] La règle d'or
> Serveur traitant 10 000 connexions simultanées en attente de DB : async. Compilation de 1000 fichiers en parallèle : threads OS + Rayon. Ne pas mélanger le code CPU-intensif dans les async tasks sans `spawn_blocking`.

---

## 2. Le trait `Future`

Toute fonction `async` compile vers une **state machine** qui implémente le trait `Future`.

```rust
// Le trait Future simplifié
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    Ready(T),   // Le futur est terminé, voici le résultat
    Pending,    // Pas encore prêt, réveille-moi via le Waker
}
```

### Comment `async fn` est compilé

```rust
// Ce que vous écrivez
async fn attendre_et_retourner(x: u32) -> u32 {
    tokio::time::sleep(std::time::Duration::from_secs(1)).await;
    x + 1
}

// Ce que le compilateur génère (approximation)
// Une state machine avec un état par point .await
enum AttendreEtRetournerFuture {
    État0 { x: u32 },
    État1 { x: u32, sleep_future: SleepFuture },
    Terminé,
}

impl Future for AttendreEtRetournerFuture {
    type Output = u32;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<u32> {
        // Gère les transitions d'états et délègue le poll aux sous-futurs
        todo!()
    }
}
```

> [!info] `Pin<&mut Self>`
> `Pin` empêche de déplacer une valeur en mémoire après qu'elle a été "pinned". C'est nécessaire car les state machines async stockent des références vers leurs propres données internes — les déplacer casserait ces références. En pratique, `Box::pin()` et `pin!()` gèrent ça pour vous.

---

## 3. Tokio Runtime

### Setup

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

### `#[tokio::main]`

```rust
// Macro qui configure et lance le runtime Tokio
#[tokio::main]
async fn main() {
    println!("Dans le runtime Tokio !");
    
    // Équivalent manuel :
    // let runtime = tokio::runtime::Runtime::new().unwrap();
    // runtime.block_on(async { ... });
}
```

### Runtime multi-thread vs current-thread

```rust
// Multi-thread (défaut) — utilise tous les cœurs disponibles
#[tokio::main]
async fn main() { /* ... */ }

// Équivalent explicite
#[tokio::main(flavor = "multi_thread", worker_threads = 4)]
async fn main() { /* ... */ }

// Single-thread — pour les environnements embarqués ou WASM
#[tokio::main(flavor = "current_thread")]
async fn main() { /* ... */ }
```

---

## 4. `tokio::spawn` — Lancer des tâches

```rust
use tokio::task::JoinHandle;
use std::time::Duration;

#[tokio::main]
async fn main() {
    // spawn retourne un JoinHandle — similaire à thread::spawn
    let handle: JoinHandle<u32> = tokio::spawn(async {
        tokio::time::sleep(Duration::from_millis(100)).await;
        42
    });

    println!("Tâche lancée, on continue...");

    // Attendre le résultat — JoinHandle<T> est lui-même un Future
    let résultat = handle.await.unwrap(); // unwrap : gère l'erreur si la tâche a paniqué
    println!("Résultat : {}", résultat);
}
```

### Lancer plusieurs tâches et attendre toutes

```rust
use tokio::task;

#[tokio::main]
async fn main() {
    let tâches: Vec<_> = (0..10)
        .map(|i| {
            task::spawn(async move {
                tokio::time::sleep(std::time::Duration::from_millis(10 * i)).await;
                i * 2
            })
        })
        .collect();

    let résultats: Vec<u64> = futures::future::join_all(tâches)
        .await
        .into_iter()
        .map(|r| r.unwrap())
        .collect();

    println!("Résultats : {:?}", résultats);
}
```

### `spawn_blocking` — Code CPU-intensif dans un contexte async

```rust
#[tokio::main]
async fn main() {
    // ❌ Ne jamais bloquer le thread async avec du code CPU-intensif
    // let résultat = calcul_lourd(); // Bloque tout le runtime !

    // ✅ Déléguer vers un thread dédié
    let résultat = tokio::task::spawn_blocking(|| {
        // Ce code tourne dans un thread séparé, pas dans le runtime async
        calcul_lourd()
    }).await.unwrap();

    println!("Résultat : {}", résultat);
}

fn calcul_lourd() -> u64 {
    (0u64..1_000_000).sum()
}
```

---

## 5. I/O Async avec Tokio

### Fichiers asynchrones

```rust
use tokio::fs;
use tokio::io::{self, AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> io::Result<()> {
    // Lecture complète d'un fichier
    let contenu = fs::read_to_string("config.toml").await?;
    println!("Config : {}", contenu);

    // Écriture asynchrone
    fs::write("output.txt", b"bonjour async").await?;

    // Lecture par chunks avec un buffer
    let mut fichier = fs::File::open("data.bin").await?;
    let mut buffer = vec![0u8; 1024];
    let n = fichier.read(&mut buffer).await?;
    println!("{} octets lus", n);

    // Écriture avec BufWriter pour la performance
    let mut sortie = io::BufWriter::new(fs::File::create("sortie.txt").await?);
    sortie.write_all(b"ligne 1\n").await?;
    sortie.write_all(b"ligne 2\n").await?;
    sortie.flush().await?;

    Ok(())
}
```

### Réseau asynchrone — TCP

```rust
use tokio::net::{TcpListener, TcpStream};
use tokio::io::{AsyncReadExt, AsyncWriteExt};

// Serveur TCP simple
async fn serveur() {
    let listener = TcpListener::bind("127.0.0.1:8080").await.unwrap();
    println!("Serveur en écoute sur :8080");

    loop {
        let (mut socket, addr) = listener.accept().await.unwrap();
        println!("Connexion depuis : {}", addr);

        // Chaque connexion dans sa propre tâche
        tokio::spawn(async move {
            let mut buffer = vec![0u8; 1024];
            loop {
                let n = match socket.read(&mut buffer).await {
                    Ok(0) => break,           // Connexion fermée
                    Ok(n) => n,
                    Err(_) => break,
                };
                // Echo : renvoyer ce qui a été reçu
                socket.write_all(&buffer[..n]).await.unwrap();
            }
        });
    }
}

// Client TCP
async fn client() {
    let mut stream = TcpStream::connect("127.0.0.1:8080").await.unwrap();
    stream.write_all(b"bonjour serveur").await.unwrap();

    let mut réponse = vec![0u8; 1024];
    let n = stream.read(&mut réponse).await.unwrap();
    println!("Réponse : {}", String::from_utf8_lossy(&réponse[..n]));
}
```

---

## 6. Primitives de Synchronisation Async

> [!warning] Ne pas utiliser `std::sync::Mutex` dans du code async
> `std::sync::Mutex::lock()` est bloquant — il bloque le thread OS, empêchant d'autres tâches async de s'exécuter. Utilisez `tokio::sync::Mutex` qui `await` au lieu de bloquer.

### Mutex async

```rust
use tokio::sync::Mutex;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let compteur = Arc::new(Mutex::new(0u32));
    let mut handles = vec![];

    for _ in 0..100 {
        let c = Arc::clone(&compteur);
        handles.push(tokio::spawn(async move {
            let mut guard = c.lock().await; // await — ne bloque pas le thread
            *guard += 1;
        }));
    }

    for h in handles { h.await.unwrap(); }
    println!("Compteur : {}", *compteur.lock().await); // 100
}
```

### Sémaphore — limiter la concurrence

```rust
use tokio::sync::Semaphore;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    // Maximum 5 opérations simultanées (ex: connexions DB)
    let sémaphore = Arc::new(Semaphore::new(5));
    let mut handles = vec![];

    for i in 0..20 {
        let sem = Arc::clone(&sémaphore);
        handles.push(tokio::spawn(async move {
            let _permit = sem.acquire().await.unwrap(); // Attend un slot
            println!("Tâche {} en cours ({} disponibles)", i,
                sem.available_permits());
            tokio::time::sleep(std::time::Duration::from_millis(100)).await;
            // _permit droppé ici → slot libéré
        }));
    }

    for h in handles { h.await.unwrap(); }
}
```

### `Notify` — signalement

```rust
use tokio::sync::Notify;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let notif = Arc::new(Notify::new());
    let n1 = Arc::clone(&notif);

    tokio::spawn(async move {
        println!("En attente de notification...");
        n1.notified().await; // Attend
        println!("Notification reçue !");
    });

    tokio::time::sleep(std::time::Duration::from_millis(100)).await;
    notif.notify_one(); // Réveille un waiter
}
```

---

## 7. Canaux Async — `tokio::sync`

### Tableau comparatif des canaux

| Canal | Producteurs | Consommateurs | Buffer | Cas d'usage |
|-------|-------------|---------------|--------|-------------|
| `mpsc` | Multiple | Single | Oui (bounded) ou Non (unbounded) | Pattern classique producteur/consommateur |
| `broadcast` | Single | Multiple | Oui (ring buffer) | Événements diffusés à tous les abonnés |
| `oneshot` | Single | Single | 1 message | Réponse à une requête unique |
| `watch` | Single | Multiple | 1 (dernier) | État partagé mis à jour |

### `mpsc` async

```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel(32); // Buffer de 32 messages

    // Plusieurs producteurs
    for i in 0..5 {
        let tx = tx.clone();
        tokio::spawn(async move {
            tx.send(format!("message_{}", i)).await.unwrap();
        });
    }
    drop(tx); // Fermer le channel côté émetteur

    while let Some(msg) = rx.recv().await {
        println!("Reçu : {}", msg);
    }
    println!("Channel fermé");
}
```

### `oneshot` — Requête/Réponse

```rust
use tokio::sync::oneshot;

async fn faire_calcul(tx: oneshot::Sender<u32>, valeur: u32) {
    let résultat = valeur * 2;
    tx.send(résultat).unwrap(); // Envoyer une seule fois
}

#[tokio::main]
async fn main() {
    let (tx, rx) = oneshot::channel();
    tokio::spawn(faire_calcul(tx, 21));
    let résultat = rx.await.unwrap();
    println!("Résultat : {}", résultat); // 42
}
```

### `broadcast` — Diffusion à tous les abonnés

```rust
use tokio::sync::broadcast;

#[tokio::main]
async fn main() {
    let (tx, _) = broadcast::channel(16); // Ring buffer de 16

    // Créer 3 abonnés
    let mut rx1 = tx.subscribe();
    let mut rx2 = tx.subscribe();
    let mut rx3 = tx.subscribe();

    tokio::spawn(async move {
        for i in 0..3 {
            tx.send(format!("événement_{}", i)).unwrap();
        }
    });

    // Chaque abonné reçoit tous les messages
    while let Ok(msg) = rx1.recv().await {
        println!("Abonné 1 : {}", msg);
    }
}
```

### `watch` — État partagé

```rust
use tokio::sync::watch;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = watch::channel("initial");

    tokio::spawn(async move {
        tokio::time::sleep(std::time::Duration::from_millis(100)).await;
        tx.send("mise à jour 1").unwrap();
        tokio::time::sleep(std::time::Duration::from_millis(100)).await;
        tx.send("mise à jour 2").unwrap();
    });

    // Attendre les changements
    while rx.changed().await.is_ok() {
        println!("Nouvel état : {}", *rx.borrow());
    }
}
```

---

## 8. Timers et Délais

```rust
use tokio::time::{sleep, timeout, interval, Duration, Instant};

#[tokio::main]
async fn main() {
    // Délai simple
    sleep(Duration::from_millis(100)).await;
    println!("100ms écoulées");

    // Timeout — annuler si trop lent
    let résultat = timeout(Duration::from_millis(50), opération_lente()).await;
    match résultat {
        Ok(val) => println!("Succès : {}", val),
        Err(_) => println!("Timeout !"),
    }

    // Intervalle — tâche périodique
    let mut tick = interval(Duration::from_millis(100));
    let début = Instant::now();
    for _ in 0..5 {
        tick.tick().await; // Premier tick immédiat
        println!("Tick à {:?}", début.elapsed());
    }
}

async fn opération_lente() -> &'static str {
    sleep(Duration::from_millis(200)).await;
    "résultat"
}
```

---

## 9. La macro `select!` — Attendre plusieurs Futures

`select!` attend plusieurs futures et retourne dès que l'une d'elles se termine.

```rust
use tokio::select;
use tokio::time::{sleep, Duration};
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel(1);

    tokio::spawn(async move {
        sleep(Duration::from_millis(200)).await;
        tx.send("message").await.unwrap();
    });

    // Attendre message OU timeout
    select! {
        msg = rx.recv() => {
            println!("Message reçu : {:?}", msg);
        }
        _ = sleep(Duration::from_millis(100)) => {
            println!("Timeout avant le message !");
        }
    }
}
```

### `select!` en boucle avec cancellation

```rust
use tokio::select;
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (cmd_tx, mut cmd_rx) = mpsc::channel(1);
    let (data_tx, mut data_rx) = mpsc::channel(10);

    // Envoyer données
    tokio::spawn(async move {
        for i in 0..5 {
            data_tx.send(i).await.unwrap();
            tokio::time::sleep(std::time::Duration::from_millis(50)).await;
        }
    });

    // Arrêter après 150ms
    tokio::spawn(async move {
        tokio::time::sleep(std::time::Duration::from_millis(150)).await;
        cmd_tx.send("stop").await.unwrap();
    });

    loop {
        select! {
            // biased; // Décommenter pour prioriser la première branche
            data = data_rx.recv() => {
                match data {
                    Some(val) => println!("Données : {}", val),
                    None => break,
                }
            }
            cmd = cmd_rx.recv() => {
                println!("Commande reçue : {:?}", cmd);
                break;
            }
        }
    }
}
```

> [!warning] Biais de `select!`
> Par défaut, `select!` choisit aléatoirement parmi les branches prêtes (pour éviter la famine). Avec `biased;`, il évalue les branches dans l'ordre — utile quand certaines branches ont une priorité absolue (ex. commandes d'arrêt).

---

## 10. Streams

Un `Stream` est l'équivalent async d'un `Iterator` — il produit des éléments de manière asynchrone.

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
tokio-stream = "0.1"
futures = "0.3"
async-stream = "0.3"
```

```rust
use tokio_stream::StreamExt;
use futures::stream;

#[tokio::main]
async fn main() {
    // Créer un stream depuis un itérateur
    let mut s = stream::iter(vec![1, 2, 3, 4, 5]);
    while let Some(val) = s.next().await {
        println!("Valeur : {}", val);
    }

    // Transformations de stream
    let somme: i32 = stream::iter(0..100)
        .filter(|&x| async move { x % 2 == 0 })
        .map(|x| async move { x * x })
        .buffered(10) // Jusqu'à 10 futures en parallèle
        .collect::<Vec<_>>()
        .await
        .into_iter()
        .sum();

    println!("Somme des carrés pairs : {}", somme);
}
```

### Créer un stream custom avec `async-stream`

```rust
use async_stream::stream;
use tokio_stream::StreamExt;

fn nombres_pairs(jusqu_a: u32) -> impl futures::Stream<Item = u32> {
    stream! {
        for i in 0..jusqu_a {
            if i % 2 == 0 {
                tokio::time::sleep(std::time::Duration::from_millis(10)).await;
                yield i;
            }
        }
    }
}

#[tokio::main]
async fn main() {
    let mut s = nombres_pairs(20);
    while let Some(n) = s.next().await {
        print!("{} ", n);
    }
    println!();
}
```

---

## 11. HTTP Async avec `reqwest`

```toml
[dependencies]
reqwest = { version = "0.12", features = ["json"] }
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
```

```rust
use serde::{Deserialize, Serialize};
use reqwest::Client;

#[derive(Debug, Deserialize)]
struct Post {
    id: u32,
    title: String,
    body: String,
}

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = Client::new();

    // GET simple avec désérialisation JSON
    let posts: Vec<Post> = client
        .get("https://jsonplaceholder.typicode.com/posts")
        .send()
        .await?
        .json()
        .await?;

    println!("Premier post : {}", posts[0].title);

    // Requêtes parallèles
    let urls = vec![
        "https://jsonplaceholder.typicode.com/posts/1",
        "https://jsonplaceholder.typicode.com/posts/2",
        "https://jsonplaceholder.typicode.com/posts/3",
    ];

    let futures: Vec<_> = urls.iter()
        .map(|url| {
            let client = client.clone();
            async move {
                client.get(*url).send().await?.json::<Post>().await
            }
        })
        .collect();

    let résultats = futures::future::join_all(futures).await;
    for r in résultats {
        if let Ok(post) = r {
            println!("Post {} : {}", post.id, post.title);
        }
    }

    Ok(())
}
```

---

## 12. Introduction à Axum — Serveur HTTP

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
```

```rust
use axum::{
    routing::{get, post},
    Router, Json, extract::Path,
    response::IntoResponse,
    http::StatusCode,
};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct Utilisateur {
    id: u32,
    nom: String,
}

// Handler GET simple
async fn racine() -> &'static str {
    "Bonjour depuis Axum !"
}

// Handler avec paramètre de chemin
async fn obtenir_utilisateur(Path(id): Path<u32>) -> impl IntoResponse {
    let utilisateur = Utilisateur { id, nom: format!("User_{}", id) };
    Json(utilisateur)
}

// Handler POST avec corps JSON
async fn créer_utilisateur(Json(payload): Json<Utilisateur>) -> impl IntoResponse {
    (StatusCode::CREATED, Json(payload))
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(racine))
        .route("/utilisateurs/:id", get(obtenir_utilisateur))
        .route("/utilisateurs", post(créer_utilisateur));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Serveur sur http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}
```

---

## 13. Gestion des erreurs en async

```rust
use anyhow::Result; // Ajouter `anyhow = "1"` dans Cargo.toml

async fn lire_config(chemin: &str) -> Result<String> {
    let contenu = tokio::fs::read_to_string(chemin).await?; // ? fonctionne en async fn
    Ok(contenu.trim().to_string())
}

async fn traiter() -> Result<()> {
    let config = lire_config("config.toml").await?;
    println!("Config : {}", config);

    // Enchaîner des opérations async avec ?
    let client = reqwest::Client::new();
    let réponse = client.get("https://example.com")
        .send()
        .await?
        .text()
        .await?;

    println!("Réponse : {} bytes", réponse.len());
    Ok(())
}

#[tokio::main]
async fn main() {
    if let Err(e) = traiter().await {
        eprintln!("Erreur : {:#}", e);
        std::process::exit(1);
    }
}
```

---

## 14. Tests Async

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tokio::time::{timeout, Duration};

    // Test async basique
    #[tokio::test]
    async fn test_basique() {
        let résultat = opération_async().await;
        assert_eq!(résultat, 42);
    }

    // Test avec timeout
    #[tokio::test]
    async fn test_avec_timeout() {
        let résultat = timeout(
            Duration::from_millis(100),
            opération_rapide()
        ).await;
        assert!(résultat.is_ok(), "Devrait terminer en moins de 100ms");
    }

    // Contrôler le temps avec `tokio::time::pause()`
    #[tokio::test]
    async fn test_timer_simulé() {
        tokio::time::pause(); // Pause le temps automatique

        let handle = tokio::spawn(async {
            tokio::time::sleep(Duration::from_secs(3600)).await; // 1 heure !
            "réveil"
        });

        // Avancer le temps manuellement
        tokio::time::advance(Duration::from_secs(3600)).await;

        let résultat = handle.await.unwrap();
        assert_eq!(résultat, "réveil"); // Instantané dans le test
    }

    async fn opération_async() -> u32 { 42 }
    async fn opération_rapide() -> u32 { 1 }
}
```

---

## 15. Patterns Avancés

### Connection Pool avec `Semaphore`

```rust
use tokio::sync::{Semaphore, Mutex};
use std::sync::Arc;
use std::collections::VecDeque;

struct Pool<T> {
    sémaphore: Arc<Semaphore>,
    connexions: Arc<Mutex<VecDeque<T>>>,
}

impl<T: Send + 'static> Pool<T> {
    fn new(connexions: Vec<T>) -> Self {
        let capacité = connexions.len();
        Pool {
            sémaphore: Arc::new(Semaphore::new(capacité)),
            connexions: Arc::new(Mutex::new(VecDeque::from(connexions))),
        }
    }

    async fn acquérir(&self) -> (T, Arc<Mutex<VecDeque<T>>>, tokio::sync::OwnedSemaphorePermit) {
        let permit = Arc::clone(&self.sémaphore)
            .acquire_owned().await.unwrap();
        let conn = self.connexions.lock().await.pop_front().unwrap();
        (conn, Arc::clone(&self.connexions), permit)
    }
}
```

### Retry avec backoff exponentiel

```rust
use std::time::Duration;
use tokio::time::sleep;

async fn avec_retry<F, Fut, T, E>(
    mut opération: F,
    max_tentatives: u32,
) -> Result<T, E>
where
    F: FnMut() -> Fut,
    Fut: std::future::Future<Output = Result<T, E>>,
    E: std::fmt::Debug,
{
    let mut délai = Duration::from_millis(100);

    for tentative in 1..=max_tentatives {
        match opération().await {
            Ok(val) => return Ok(val),
            Err(e) if tentative < max_tentatives => {
                eprintln!("Tentative {} échouée : {:?}. Retry dans {:?}", tentative, e, délai);
                sleep(délai).await;
                délai *= 2; // Backoff exponentiel
            }
            Err(e) => return Err(e),
        }
    }
    unreachable!()
}
```

---

## 16. Exercices Pratiques

> [!tip] Exercices progressifs — mettre en pratique l'async Rust

### Exercice 1 — Premier serveur async (Débutant)
Créez un serveur TCP qui :
- Écoute sur le port 8080
- Accepte des connexions et répond "Bonjour !" à tout message reçu
- Gère 100 connexions simultanées sans problème
- Affiche un log de chaque connexion avec son adresse IP

### Exercice 2 — Téléchargeur parallèle (Intermédiaire)
Écrivez un outil qui télécharge plusieurs URLs en parallèle :
- Prend une liste d'URLs en argument
- Les télécharge toutes en parallèle avec `reqwest`
- Limite à 5 downloads simultanés (Semaphore)
- Affiche la progression et le temps total
- Gère les erreurs gracieusement (timeout de 10s par URL)

### Exercice 3 — Chat TCP multi-client (Intermédiaire)
Serveur de chat minimaliste :
- Chaque connexion TCP est un "client"
- Un message envoyé par un client est broadcasté à tous les autres
- Utiliser `tokio::sync::broadcast` pour la diffusion
- Gérer la déconnexion propre des clients
- Maximum 50 clients simultanés

### Exercice 4 — Rate limiter async (Avancé)
Implémentez un middleware de limitation de débit :
- Maximum N requêtes par seconde par IP
- Utiliser `tokio::time::interval` et des compteurs atomiques
- Intégrer avec un serveur Axum basique
- Retourner HTTP 429 (Too Many Requests) si dépassé

### Exercice 5 — Worker pool async avec priorités (Avancé)
Pool de workers avec files de priorité :
- 3 niveaux de priorité (Haute, Normale, Basse)
- N workers async qui consomment les tâches par priorité décroissante
- Métriques : temps d'attente moyen par priorité, throughput
- Arrêt gracieux : terminer les tâches en cours avant de s'arrêter

---

## Notes liées

- [[08 - Concurrence et Smart Pointers]] — Threads OS, Arc/Mutex — base de ce module
- [[04 - Gestion des Erreurs en Rust]] — `?` en contexte async, `anyhow`
- [[06 - Traits et Generiques]] — Le trait `Future`, `Stream`, `AsyncRead`, `AsyncWrite`
- [[05 - Collections et Iterateurs]] — Les streams sont des itérateurs async
- [[15 - Web Frameworks Axum et Actix]] — Suite logique de ce module pour le développement web
- [[02 - Ownership et Borrowing]] — `Pin`, lifetimes dans les async fn
- [[07 - Projet Rust CLI]] — Appliquer async dans un vrai projet CLI
