# 08 - Concurrence et Smart Pointers

> [!info] Prérequis
> Ce module suppose une maîtrise de [[02 - Ownership et Borrowing]] et des [[06 - Traits et Generiques]]. La concurrence en Rust repose entièrement sur le système de types — comprendre ownership est indispensable.

---

## 1. Smart Pointers — Vue d'ensemble

En Rust, une **référence** (`&T`) emprunte de la donnée sans en être propriétaire. Un **smart pointer** est une structure qui *possède* la donnée et implémente `Deref` (et souvent `Drop`). Ils permettent des patterns d'ownership impossibles avec les références simples.

### Tableau comparatif — Quand utiliser quoi

| Type | Thread-safe | Mutabilité | Ownership | Cas d'usage principal |
|------|-------------|------------|-----------|----------------------|
| `Box<T>` | Oui (si T: Send) | Via `&mut` | Unique | Heap allocation, trait objects, récursion |
| `Rc<T>` | **Non** | Via `RefCell<T>` | Partagé | Graphes single-thread, cycles UI |
| `Arc<T>` | **Oui** | Via `Mutex<T>` | Partagé | Données partagées entre threads |
| `RefCell<T>` | Non | Borrow check runtime | Unique | Mutabilité intérieure sans `mut` |
| `Cell<T>` | Non | `get()`/`set()` | Unique | Copy types, overhead nul |
| `Mutex<T>` | Oui | Lock exclusif | - | Mutation partagée multi-thread |
| `RwLock<T>` | Oui | Lecture multi / écriture exclusive | - | Données lues souvent, écrites rarement |
| `Weak<T>` | - | Dépend de la cible | Non-owning | Éviter les cycles de référence |

---

## 2. `Box<T>` — Allocation sur le heap

`Box<T>` est le smart pointer le plus simple : il alloue `T` sur le heap et en est le propriétaire unique. Quand le `Box` sort de scope, la mémoire est libérée automatiquement.

### Cas d'usage 1 : Types de taille inconnue à la compilation

```rust
// ❌ Impossible — taille inconnue à la compilation
// enum List { Cons(i32, List), Nil }

// ✅ Box interrompt la récursion infinie
#[derive(Debug)]
enum List {
    Cons(i32, Box<List>),
    Nil,
}

fn main() {
    let list = List::Cons(1,
        Box::new(List::Cons(2,
            Box::new(List::Cons(3,
                Box::new(List::Nil)
            ))
        ))
    );
    println!("{:?}", list);
}
```

### Cas d'usage 2 : Trait objects (`dyn Trait`)

```rust
trait Animal {
    fn parler(&self) -> &str;
    fn nom(&self) -> &str;
}

struct Chien { nom: String }
struct Chat { nom: String }

impl Animal for Chien {
    fn parler(&self) -> &str { "Ouaf!" }
    fn nom(&self) -> &str { &self.nom }
}

impl Animal for Chat {
    fn parler(&self) -> &str { "Miaou!" }
    fn nom(&self) -> &str { &self.nom }
}

fn main() {
    // Vec de trait objects — dispatch dynamique via vtable
    let animaux: Vec<Box<dyn Animal>> = vec![
        Box::new(Chien { nom: "Rex".to_string() }),
        Box::new(Chat { nom: "Felix".to_string() }),
        Box::new(Chien { nom: "Lassie".to_string() }),
    ];

    for animal in &animaux {
        println!("{} dit : {}", animal.nom(), animal.parler());
    }
}
```

> [!tip] `Box<dyn Trait>` vs Generics
> Utilisez les generics (`impl Trait` ou `<T: Trait>`) quand tous les types sont connus à la compilation — c'est plus rapide (dispatch statique, monomorphisation). Utilisez `Box<dyn Trait>` quand les types sont déterminés au runtime ou quand vous avez besoin d'hétérogénéité dans une collection.

### Déréférencement et `Deref` coercions

```rust
fn main() {
    let x = Box::new(42);
    
    // Déréférencement explicite
    println!("Valeur : {}", *x);
    
    // Deref coercion : Box<String> → &String → &str
    let s = Box::new(String::from("hello"));
    let len = calculer_longueur(&s); // &Box<String> se coerce en &str
    println!("Longueur : {}", len);
}

fn calculer_longueur(s: &str) -> usize {
    s.len()
}
```

---

## 3. `Rc<T>` — Reference Counting (single-thread)

`Rc<T>` (Reference Counted) permet à plusieurs parties du code de posséder la même donnée. Le compteur de références s'incrémente à chaque `clone()` et se décrémente quand un `Rc` sort de scope. La donnée est libérée quand le compteur atteint 0.

> [!warning] Single-thread uniquement
> `Rc<T>` n'implémente pas `Send` — il ne peut pas être envoyé entre threads. Pour le multi-thread, utilisez `Arc<T>`.

```rust
use std::rc::Rc;

#[derive(Debug)]
struct Noeud {
    valeur: i32,
    enfants: Vec<Rc<Noeud>>,
}

fn main() {
    let feuille = Rc::new(Noeud {
        valeur: 5,
        enfants: vec![],
    });

    println!("Compteur après création : {}", Rc::strong_count(&feuille)); // 1

    let branche = Rc::new(Noeud {
        valeur: 3,
        enfants: vec![Rc::clone(&feuille)], // clone incrémente le compteur
    });

    println!("Compteur après branche : {}", Rc::strong_count(&feuille)); // 2

    {
        let autre_ref = Rc::clone(&feuille);
        println!("Compteur dans le bloc : {}", Rc::strong_count(&feuille)); // 3
    } // autre_ref sort de scope → compteur décrémenté

    println!("Compteur après le bloc : {}", Rc::strong_count(&feuille)); // 2
}
```

---

## 4. `Arc<T>` — Atomic Reference Counting (multi-thread)

`Arc<T>` est l'équivalent thread-safe de `Rc<T>`. Les opérations sur le compteur sont atomiques, ce qui le rend plus lent que `Rc<T>` mais sûr en contexte concurrent.

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let donnees = Arc::new(vec![1, 2, 3, 4, 5]);
    let mut handles = vec![];

    for i in 0..5 {
        let donnees_clone = Arc::clone(&donnees); // clone atomique
        let handle = thread::spawn(move || {
            println!("Thread {} voit : {:?}", i, donnees_clone);
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }
}
```

> [!tip] `Arc<T>` est immuable par défaut
> Pour partager des données mutables entre threads, combinez `Arc<Mutex<T>>` ou `Arc<RwLock<T>>`.

---

## 5. `RefCell<T>` — Mutabilité intérieure

`RefCell<T>` déplace les vérifications du borrow checker du **compile time** au **runtime**. Cela permet la mutabilité à l'intérieur d'une valeur immutable — le pattern dit de **interior mutability**.

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Noeud {
    valeur: i32,
    parent: Option<Rc<RefCell<Noeud>>>,
    enfants: Vec<Rc<RefCell<Noeud>>>,
}

impl Noeud {
    fn new(valeur: i32) -> Rc<RefCell<Self>> {
        Rc::new(RefCell::new(Noeud {
            valeur,
            parent: None,
            enfants: vec![],
        }))
    }
}

fn main() {
    let parent = Noeud::new(1);
    let enfant = Noeud::new(2);

    // Mutation via RefCell même si parent est immutable
    parent.borrow_mut().enfants.push(Rc::clone(&enfant));
    enfant.borrow_mut().parent = Some(Rc::clone(&parent));

    println!("Parent valeur : {}", parent.borrow().valeur);
    println!("Enfant valeur : {}", enfant.borrow().valeur);

    // borrow() → Ref<T> (immutable borrow)
    // borrow_mut() → RefMut<T> (mutable borrow)
    // Panique au runtime si règles violées
}
```

> [!warning] Paniques runtime avec `RefCell`
> `borrow()` panique si une référence mutable existe déjà. `borrow_mut()` panique si n'importe quelle référence (mutable ou non) existe. Préférez `try_borrow()` et `try_borrow_mut()` qui retournent un `Result` si vous ne voulez pas de paniques.

---

## 6. `Cell<T>` — Mutabilité intérieure pour Copy types

`Cell<T>` est une version simplifiée de `RefCell<T>` pour les types `Copy`. Pas de borrow dynamique — on copie la valeur directement. Overhead nul.

```rust
use std::cell::Cell;

struct Compteur {
    valeur: Cell<i32>,
}

impl Compteur {
    fn new() -> Self {
        Compteur { valeur: Cell::new(0) }
    }

    // &self (immutable) mais on peut modifier via Cell
    fn incrementer(&self) {
        self.valeur.set(self.valeur.get() + 1);
    }

    fn valeur(&self) -> i32 {
        self.valeur.get()
    }
}

fn main() {
    let c = Compteur::new();
    c.incrementer();
    c.incrementer();
    c.incrementer();
    println!("Compteur : {}", c.valeur()); // 3
}
```

---

## 7. `Mutex<T>` et `RwLock<T>` — Exclusion mutuelle

### `Mutex<T>`

Un `Mutex<T>` protège une donnée avec un verrou exclusif. Pour accéder à la donnée, vous devez d'abord acquérir le lock. Si un autre thread détient le lock, le thread courant bloque.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let compteur = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let compteur = Arc::clone(&compteur);
        let handle = thread::spawn(move || {
            let mut valeur = compteur.lock().unwrap(); // bloque si lock pris
            *valeur += 1;
            // MutexGuard libère le lock ici (RAII)
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Résultat : {}", *compteur.lock().unwrap()); // 10
}
```

> [!warning] Mutex empoisonné (poisoned mutex)
> Si un thread panique en détenant un lock, le `Mutex` devient "empoisonné". Les appels suivants à `lock()` retournent `Err`. `unwrap()` propagera la panique. Gérez cela explicitement avec `match` ou `lock().unwrap_or_else(|e| e.into_inner())`.

### `RwLock<T>`

Permet plusieurs lecteurs simultanés ou un seul écrivain. Idéal pour les données lues fréquemment mais modifiées rarement.

```rust
use std::sync::{Arc, RwLock};
use std::thread;

fn main() {
    let cache = Arc::new(RwLock::new(vec![1, 2, 3]));

    // Plusieurs lecteurs en simultané
    let mut handles = vec![];
    for i in 0..5 {
        let cache = Arc::clone(&cache);
        handles.push(thread::spawn(move || {
            let data = cache.read().unwrap(); // read lock — plusieurs simultanés OK
            println!("Thread {} lit : {:?}", i, *data);
        }));
    }

    // Un seul écrivain
    let cache_write = Arc::clone(&cache);
    handles.push(thread::spawn(move || {
        let mut data = cache_write.write().unwrap(); // write lock — exclusif
        data.push(4);
        println!("Écriture : {:?}", *data);
    }));

    for h in handles { h.join().unwrap(); }
}
```

---

## 8. `Weak<T>` — Éviter les cycles de référence

`Rc<T>` et `Arc<T>` ne libèrent jamais la mémoire si des cycles de références existent. `Weak<T>` est une référence non-possessive qui ne contribue pas au compteur fort.

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Noeud {
    valeur: i32,
    parent: RefCell<Weak<Noeud>>,   // Weak : pas de cycle
    enfants: RefCell<Vec<Rc<Noeud>>>,
}

fn main() {
    let feuille = Rc::new(Noeud {
        valeur: 3,
        parent: RefCell::new(Weak::new()),
        enfants: RefCell::new(vec![]),
    });

    let branche = Rc::new(Noeud {
        valeur: 5,
        parent: RefCell::new(Weak::new()),
        enfants: RefCell::new(vec![Rc::clone(&feuille)]),
    });

    // Assigner le parent (Weak — pas de cycle)
    *feuille.parent.borrow_mut() = Rc::downgrade(&branche);

    // Accéder au parent via upgrade() → Option<Rc<T>>
    if let Some(parent) = feuille.parent.borrow().upgrade() {
        println!("Parent de feuille : {}", parent.valeur);
    }

    println!("Strong count branche : {}", Rc::strong_count(&branche)); // 1
    println!("Weak count branche : {}", Rc::weak_count(&branche));     // 1
}
```

---

## 9. Threads — `std::thread`

### Spawn et join

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..=5 {
            println!("Thread spawné : {}", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..=3 {
        println!("Thread principal : {}", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap(); // Attendre la fin du thread
}
```

### `move` closures — capturer l'environnement

```rust
use std::thread;

fn main() {
    let message = String::from("bonjour depuis le thread principal");

    // `move` transfère la propriété de `message` dans le thread
    let handle = thread::spawn(move || {
        println!("{}", message); // message est maintenant dans ce thread
    });

    // println!("{}", message); // ❌ Erreur : message a été moved

    handle.join().unwrap();
}
```

### Thread local storage

```rust
use std::cell::RefCell;
use std::thread;

// Chaque thread a sa propre copie de THREAD_ID
thread_local! {
    static THREAD_DATA: RefCell<Vec<String>> = RefCell::new(vec![]);
}

fn ajouter_log(message: &str) {
    THREAD_DATA.with(|data| {
        data.borrow_mut().push(message.to_string());
    });
}

fn afficher_logs() {
    THREAD_DATA.with(|data| {
        println!("Logs de ce thread : {:?}", data.borrow());
    });
}

fn main() {
    let handle = thread::spawn(|| {
        ajouter_log("Message A");
        ajouter_log("Message B");
        afficher_logs();
    });

    ajouter_log("Message principal");
    afficher_logs();

    handle.join().unwrap();
}
```

---

## 10. Channels — Communication inter-threads

### `mpsc::channel` (Multiple Producer, Single Consumer)

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    // Cloner le sender pour plusieurs producteurs
    let tx1 = tx.clone();
    let tx2 = tx;

    thread::spawn(move || {
        tx1.send(String::from("Message du thread 1")).unwrap();
        tx1.send(String::from("Deuxième message du thread 1")).unwrap();
    });

    thread::spawn(move || {
        tx2.send(String::from("Message du thread 2")).unwrap();
    });

    // rx reçoit tous les messages (bloque jusqu'à réception)
    for message in rx {
        println!("Reçu : {}", message);
    }
    // La boucle se termine quand tous les Sender sont drop
}
```

### `mpsc::sync_channel` — Canal borné (backpressure)

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    // Buffer de 3 messages maximum
    let (tx, rx) = mpsc::sync_channel(3);

    let handle = thread::spawn(move || {
        for i in 0..10 {
            println!("Envoi de {}", i);
            tx.send(i).unwrap(); // Bloque si le buffer est plein
        }
    });

    thread::sleep(std::time::Duration::from_millis(100));

    for valeur in rx {
        println!("Reçu : {}", valeur);
    }

    handle.join().unwrap();
}
```

### Pattern producteur/consommateur complet

```rust
use std::sync::mpsc;
use std::thread;

#[derive(Debug)]
enum Travail {
    Tache(String),
    Terminer,
}

fn main() {
    let (tx, rx) = mpsc::channel::<Travail>();

    // Consommateur
    let consommateur = thread::spawn(move || {
        loop {
            match rx.recv().unwrap() {
                Travail::Tache(t) => println!("Traitement : {}", t),
                Travail::Terminer => {
                    println!("Consommateur terminé");
                    break;
                }
            }
        }
    });

    // Producteurs
    for i in 0..5 {
        tx.send(Travail::Tache(format!("tache_{}", i))).unwrap();
    }
    tx.send(Travail::Terminer).unwrap();

    consommateur.join().unwrap();
}
```

---

## 11. Rayon — Data Parallelism

Rayon est la bibliothèque de référence pour le parallélisme de données en Rust. Elle distribue automatiquement le travail sur tous les cœurs disponibles via un algorithme de **work-stealing**.

### Installation

```toml
# Cargo.toml
[dependencies]
rayon = "1.10"
```

### `par_iter()` — Itérateurs parallèles

```rust
use rayon::prelude::*;

fn main() {
    let nombres: Vec<i64> = (0..1_000_000).collect();

    // Séquentiel
    let somme_seq: i64 = nombres.iter().sum();

    // Parallèle — identique en syntaxe, distribué sur tous les cœurs
    let somme_par: i64 = nombres.par_iter().sum();

    assert_eq!(somme_seq, somme_par);

    // Transformation parallèle
    let carres: Vec<i64> = nombres
        .par_iter()
        .map(|&n| n * n)
        .filter(|&n| n % 2 == 0)
        .collect();

    println!("Premiers carrés pairs : {:?}", &carres[..5]);
}
```

### `join()` — Deux tâches en parallèle

```rust
use rayon;

fn fibonacci(n: u64) -> u64 {
    if n <= 1 { return n; }
    // Les deux appels s'exécutent en parallèle si rayon le juge pertinent
    let (a, b) = rayon::join(
        || fibonacci(n - 1),
        || fibonacci(n - 2),
    );
    a + b
}

fn main() {
    println!("Fibonacci(30) = {}", fibonacci(30));
}
```

### Configuration du thread pool

```rust
use rayon::ThreadPoolBuilder;

fn main() {
    // Pool global par défaut = nombre de cœurs logiques
    // Créer un pool personnalisé
    let pool = ThreadPoolBuilder::new()
        .num_threads(4)
        .build()
        .unwrap();

    pool.install(|| {
        let resultat: Vec<_> = (0..100)
            .into_par_iter()
            .map(|i| i * 2)
            .collect();
        println!("Résultat (4 threads) : {} éléments", resultat.len());
    });
}
```

> [!tip] Quand utiliser Rayon
> Rayon excelle sur les **opérations CPU-bound** avec des données larges (> quelques milliers d'éléments). Pour des tâches I/O-bound, préférez async/await avec Tokio (voir [[09 - Async Await et Tokio]]). Pour de très petits datasets, le sequential est plus rapide à cause de l'overhead de distribution.

---

## 12. Atomics — Synchronisation sans verrou

Les atomics sont des primitives bas niveau permettant des opérations thread-safe sans Mutex. Utiles pour les compteurs, flags et structures lock-free.

```rust
use std::sync::atomic::{AtomicUsize, AtomicBool, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let compteur = Arc::new(AtomicUsize::new(0));
    let actif = Arc::new(AtomicBool::new(true));

    let mut handles = vec![];

    for _ in 0..10 {
        let c = Arc::clone(&compteur);
        let a = Arc::clone(&actif);
        handles.push(thread::spawn(move || {
            while a.load(Ordering::Relaxed) {
                c.fetch_add(1, Ordering::SeqCst);
                break; // Juste un incrément pour l'exemple
            }
        }));
    }

    for h in handles { h.join().unwrap(); }

    actif.store(false, Ordering::SeqCst);
    println!("Compteur final : {}", compteur.load(Ordering::SeqCst)); // 10
}
```

### Ordres de mémoire

| Ordering | Description | Quand utiliser |
|----------|-------------|----------------|
| `Relaxed` | Aucune synchronisation de mémoire | Compteurs sans coordination inter-threads |
| `Acquire` | Garantit que les lectures suivantes voient les écritures précédant le `Release` | Acquisition d'un lock |
| `Release` | Garantit que les écritures précédentes sont visibles avant ce point | Libération d'un lock |
| `AcqRel` | Acquire + Release combinés | Opérations read-modify-write (fetch_add) |
| `SeqCst` | Ordre total cohérent entre tous les threads | Quand vous n'êtes pas sûr, au détriment des performances |

> [!warning] Ordering est complexe
> Mal utiliser les ordres de mémoire introduit des bugs de concurrence extrêmement difficiles à déboguer. En cas de doute : `SeqCst` est toujours correct (mais plus lent). `Relaxed` est uniquement sûr pour des compteurs indépendants.

---

## 13. `Send` et `Sync` — Traits de marqueur

Ces deux traits définissent la sémantique de sécurité des threads au niveau du système de types.

- **`Send`** : un type `T: Send` peut être transféré (moved) vers un autre thread.
- **`Sync`** : un type `T: Sync` peut être référencé depuis plusieurs threads simultanément (i.e., `&T: Send`).

### Pourquoi `Rc<T>` n'est pas `Send`

```rust
use std::rc::Rc;
use std::thread;

fn main() {
    let rc = Rc::new(42);

    // ❌ Erreur de compilation :
    // `Rc<i32>` cannot be sent between threads safely
    // thread::spawn(move || {
    //     println!("{}", rc);
    // });

    // Le compilateur nous protège : Rc utilise des compteurs non-atomiques.
    // Si deux threads modifiaient le compteur simultanément → data race.
}
```

### Implémenter `Send` et `Sync` pour des types custom

```rust
use std::ptr::NonNull;

// Type wrapping un raw pointer
struct MonPtr<T>(NonNull<T>);

// ❌ Par défaut, les raw pointers ne sont ni Send ni Sync
// On peut les marquer manuellement si on garantit la sécurité

// SAFETY: MonPtr est Send si T est Send (nous gérons l'accès exclusif)
unsafe impl<T: Send> Send for MonPtr<T> {}

// SAFETY: MonPtr est Sync si T est Sync (accès en lecture seule partageable)
unsafe impl<T: Sync> Sync for MonPtr<T> {}
```

> [!warning] `unsafe impl Send/Sync`
> Ces implémentations sont marquées `unsafe` car vous promettez au compilateur que votre code est sûr. Assurez-vous de comprendre tous les invariants avant d'ajouter ces implémentations.

---

## 14. Deadlocks — Prévention

Un deadlock se produit quand deux threads attendent chacun un lock détenu par l'autre.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let lock_a = Arc::new(Mutex::new("A"));
    let lock_b = Arc::new(Mutex::new("B"));

    let la = Arc::clone(&lock_a);
    let lb = Arc::clone(&lock_b);

    // ❌ Risque de deadlock si les deux threads s'exécutent simultanément
    // Thread 1 : lock_a → lock_b
    // Thread 2 : lock_b → lock_a

    // ✅ Solution : toujours acquérir dans le même ordre
    // Ici on assure : lock_a TOUJOURS avant lock_b
    let h1 = thread::spawn(move || {
        let _a = la.lock().unwrap();
        let _b = lb.lock().unwrap();
        println!("Thread 1 a les deux locks");
    });

    let la2 = Arc::clone(&lock_a);
    let lb2 = Arc::clone(&lock_b);

    let h2 = thread::spawn(move || {
        let _a = la2.lock().unwrap(); // Même ordre : a avant b
        let _b = lb2.lock().unwrap();
        println!("Thread 2 a les deux locks");
    });

    h1.join().unwrap();
    h2.join().unwrap();
}
```

### `parking_lot` — Alternative performante

La crate `parking_lot` fournit des Mutex, RwLock et Condvar plus rapides que la stdlib, avec des fonctionnalités additionnelles (try_lock avec timeout, pas d'empoisonnement).

```toml
[dependencies]
parking_lot = "0.12"
```

```rust
use parking_lot::Mutex;
use std::sync::Arc;

fn main() {
    let data = Arc::new(Mutex::new(vec![1, 2, 3]));
    let d = Arc::clone(&data);

    let handle = std::thread::spawn(move || {
        d.lock().push(4); // Pas de .unwrap() nécessaire
    });

    handle.join().unwrap();
    println!("{:?}", *data.lock()); // [1, 2, 3, 4]
}
```

---

## 15. Patterns de Concurrence

### Thread Pool manuel

```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;

type Job = Box<dyn FnOnce() + Send + 'static>;

struct ThreadPool {
    workers: Vec<thread::JoinHandle<()>>,
    sender: mpsc::Sender<Job>,
}

impl ThreadPool {
    fn new(taille: usize) -> Self {
        let (tx, rx) = mpsc::channel::<Job>();
        let rx = Arc::new(Mutex::new(rx));

        let workers = (0..taille)
            .map(|id| {
                let rx = Arc::clone(&rx);
                thread::spawn(move || loop {
                    let job = rx.lock().unwrap().recv();
                    match job {
                        Ok(tache) => {
                            println!("Worker {} exécute une tâche", id);
                            tache();
                        }
                        Err(_) => break, // Channel fermé
                    }
                })
            })
            .collect();

        ThreadPool { workers, sender: tx }
    }

    fn execute<F>(&self, f: F)
    where F: FnOnce() + Send + 'static
    {
        self.sender.send(Box::new(f)).unwrap();
    }
}

fn main() {
    let pool = ThreadPool::new(4);

    for i in 0..8 {
        pool.execute(move || {
            println!("Tâche {} exécutée", i);
        });
    }

    thread::sleep(std::time::Duration::from_millis(100));
}
```

### Actor Pattern avec channels

```rust
use std::sync::mpsc;
use std::thread;

enum Message {
    IncrémenterDe(i32),
    Obtenir(mpsc::Sender<i32>),
    Arrêter,
}

struct Acteur {
    sender: mpsc::Sender<Message>,
}

impl Acteur {
    fn nouveau() -> Self {
        let (tx, rx) = mpsc::channel();
        thread::spawn(move || {
            let mut etat = 0i32;
            for msg in rx {
                match msg {
                    Message::IncrémenterDe(n) => etat += n,
                    Message::Obtenir(réponse) => réponse.send(etat).unwrap(),
                    Message::Arrêter => break,
                }
            }
        });
        Acteur { sender: tx }
    }

    fn incrementer(&self, n: i32) {
        self.sender.send(Message::IncrémenterDe(n)).unwrap();
    }

    fn obtenir(&self) -> i32 {
        let (tx, rx) = mpsc::channel();
        self.sender.send(Message::Obtenir(tx)).unwrap();
        rx.recv().unwrap()
    }
}

fn main() {
    let acteur = Acteur::nouveau();
    acteur.incrementer(10);
    acteur.incrementer(5);
    println!("État : {}", acteur.obtenir()); // 15
}
```

---

## 16. Exercices Pratiques

> [!tip] Exercices progressifs — du plus simple au plus avancé

### Exercice 1 — Box et trait objects (Débutant)
Créez une calculatrice avec le pattern strategy :
- Trait `Operation` avec méthode `calculer(a: f64, b: f64) -> f64`
- Structs `Addition`, `Soustraction`, `Multiplication`, `Division`
- Struct `Calculatrice` contenant un `Vec<Box<dyn Operation>>`
- Fonction qui applique toutes les opérations sur une paire de nombres

### Exercice 2 — Arc et Mutex (Intermédiaire)
Implémentez un compteur de votes thread-safe :
- Struct `SystemeVote` avec `Arc<Mutex<HashMap<String, u32>>>`
- Méthodes : `voter(candidat: &str)`, `résultats() -> HashMap<String, u32>`
- Lancer 100 threads qui votent chacun pour un candidat aléatoire
- Vérifier que le total des votes = 100

### Exercice 3 — Channels producteur/consommateur (Intermédiaire)
Implémentez un pipeline de traitement d'images fictif :
- Stage 1 : producteur génère des "images" (juste des nombres)
- Stage 2 : filtre qui rejette les valeurs paires
- Stage 3 : transformateur qui double les valeurs
- Stage 4 : consommateur qui accumule les résultats
- Chaque stage tourne dans son propre thread, communicant via channels

### Exercice 4 — Rayon (Intermédiaire)
Comparez les performances séquentielles et parallèles :
- Calcul des 1 000 000 premiers nombres premiers (crible d'Ératosthène parallèle)
- Benchmark avec `std::time::Instant`
- Affichage du speedup obtenu

### Exercice 5 — Actor pattern (Avancé)
Implémentez un système de cache distribué :
- `ActeurCache` qui gère un `HashMap<String, String>` en mémoire
- Messages : `Set(clé, valeur)`, `Get(clé, sender)`, `Supprimer(clé)`, `Vider`
- 5 threads producteurs qui écrivent des données
- 10 threads lecteurs qui lisent de façon concurrente
- Vérification de la cohérence finale

---

## Notes liées

- [[02 - Ownership et Borrowing]] — Comprendre ownership est prérequis à tous les smart pointers
- [[06 - Traits et Generiques]] — `Deref`, `Drop`, `Send`, `Sync` sont des traits
- [[09 - Async Await et Tokio]] — Alternative async aux threads OS pour l'I/O concurrente
- [[03 - Structs Enums et Pattern Matching]] — Structs utilisées dans tous les patterns de ce module
- [[04 - Gestion des Erreurs en Rust]] — Gestion des erreurs de lock (empoisonnement, timeouts)
- [[05 - Collections et Iterateurs]] — `par_iter()` de Rayon étend les itérateurs standards
- [[07 - Projet Rust CLI]] — Exemple pratique d'application multi-thread
