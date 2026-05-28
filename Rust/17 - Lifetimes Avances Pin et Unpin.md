# 17 - Lifetimes Avancés, Pin et Unpin

> [!info] Pré-requis
> Ce cours approfondit [[02 - Ownership et Borrowing]] et suppose la maîtrise de [[06 - Traits et Generiques]]. La section Pin/Unpin est indispensable pour comprendre [[09 - Async Await et Tokio]] en profondeur et l'interaction avec [[11 - Unsafe Rust et FFI]].

---

## 1. Rappel — Lifetimes de base

Une **lifetime** est une annotation qui indique au compilateur pendant combien de temps une référence est valide. Elle n'existe que dans le code source — aucun impact runtime.

```rust
// Lifetime explicite : 'a vit au moins aussi longtemps que la référence retournée
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

// Pourquoi cette signature ? Sans 'a, le compilateur ne sait pas
// si le retour vient de x ou de y — donc quelle durée de vie garantir.

fn main() {
    let s1 = String::from("long string");
    let result;
    {
        let s2 = String::from("short");
        result = longest(s1.as_str(), s2.as_str());
        println!("{result}"); // OK : s2 vit encore ici
    }
    // println!("{result}"); // ERREUR : s2 est mort, result pourrait pointer sur lui
}
```

### `'static`

`'static` signifie "valide pour toute la durée du programme". Les string literals `"hello"` ont la lifetime `'static` car elles sont stockées dans le segment data du binaire.

```rust
// Deux usages distincts de 'static :

// 1. Référence statique — pointe vers des données permanentes
fn greeting() -> &'static str { "Hello, world!" }

// 2. Bound sur un type — T ne contient aucune référence non-statique
// (il peut être envoyé entre threads sans souci de lifetime)
fn spawn_task<T: Send + 'static>(task: T) {
    std::thread::spawn(move || { /* task */ });
}
```

---

## 2. Règles d'élision des lifetimes

Le compilateur Rust peut inférer les lifetimes dans de nombreux cas grâce à trois règles appliquées dans l'ordre.

### Règle 1 — Chaque paramètre reçoit sa propre lifetime

```rust
// Code écrit :
fn first_word(s: &str) -> &str { /* ... */ }

// Ce que le compilateur voit :
fn first_word<'a>(s: &'a str) -> &'a str { /* ... */ }
// Une lifetime 'a est assignée au paramètre &str
```

### Règle 2 — Un seul input lifetime → il s'applique aux outputs

```rust
// Code écrit :
fn trim(s: &str) -> &str { s.trim() }

// Après règle 1 : fn trim<'a>(s: &'a str) -> &str
// Après règle 2 : fn trim<'a>(s: &'a str) -> &'a str ✓
// Élision réussie — pas d'annotation nécessaire
```

### Règle 3 — Si `&self` ou `&mut self`, sa lifetime s'applique aux outputs

```rust
struct Config { value: String }

impl Config {
    // Code écrit :
    fn get_value(&self) -> &str { &self.value }

    // Ce que le compilateur voit :
    fn get_value<'a>(&'a self) -> &'a str { &self.value }
    // Règle 3 : la lifetime de &self s'applique au retour
}
```

### Quand l'élision échoue

```rust
// Deux paramètres de référence → le compilateur ne sait pas lequel alimente le retour
// → annotation obligatoire
fn pick(x: &str, y: &str) -> &str { // ERREUR : ambiguïté
    if x.len() > 0 { x } else { y }
}

// Correction :
fn pick<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > 0 { x } else { y }
}
```

> [!tip] Heuristique pratique
> Si votre fonction retourne une référence et prend plusieurs références en paramètre, les lifetimes sont probablement nécessaires. Si la référence retournée ne peut venir que d'un seul paramètre, annotez uniquement ce paramètre.

---

## 3. Lifetimes dans les structs et les implémentations

### Struct avec référence

```rust
// Une struct qui emprunte des données — doit annoter la lifetime
struct StrSplit<'a> {
    remainder: &'a str,
    delimiter: &'a str,
}

// impl doit répéter les lifetimes
impl<'a> StrSplit<'a> {
    fn new(input: &'a str, delimiter: &'a str) -> Self {
        Self { remainder: input, delimiter }
    }

    // La lifetime de la méthode est liée à celle de la struct
    fn next_token(&mut self) -> Option<&'a str> {
        if let Some(pos) = self.remainder.find(self.delimiter) {
            let token = &self.remainder[..pos];
            self.remainder = &self.remainder[pos + self.delimiter.len()..];
            Some(token)
        } else if self.remainder.is_empty() {
            None
        } else {
            let token = self.remainder;
            self.remainder = "";
            Some(token)
        }
    }
}
```

### Lifetime bounds

```rust
// 'a: 'b signifie "'a outlive 'b" (a vit au moins aussi longtemps que b)
fn longer_ref<'a, 'b>(x: &'a str, _y: &'b str) -> &'a str
where
    'a: 'b,  // 'a doit outlive 'b
{
    x
}

// T: 'a signifie "toutes les références dans T durent au moins 'a"
struct Cache<'a, T: 'a> {
    data: &'a T,
}
// Depuis Rust 2018, T: 'a est souvent déduit automatiquement dans ce contexte
```

### Plusieurs lifetimes — quand les distinguer

```rust
// Cas où une seule lifetime est suffisante
fn first<'a>(list: &'a [&'a str]) -> &'a str {
    list[0]
}

// Cas où deux lifetimes sont nécessaires :
// le contenu de la slice et la slice elle-même peuvent avoir des durées différentes
fn get_item<'list, 'item>(list: &'list [&'item str]) -> &'item str {
    list[0]
    // Ici 'item peut outlive 'list — la référence retournée vit
    // indépendamment de la slice qui la contient
}
```

---

## 4. Higher-Rank Trait Bounds (HRTB)

### Le problème

```rust
// On veut une fonction qui prend n'importe quelle closure
// qui accepte une &str et retourne une &str
// mais la lifetime de l'input et de l'output doivent être liées

// Ceci ne compile pas — quelle lifetime utiliser pour 'a ?
fn apply_to_str<F: Fn(&'a str) -> &'a str>(f: F, s: &str) -> String {
    f(s).to_string()
}
```

### La syntaxe `for<'a>`

```rust
// HRTB : "pour toute lifetime 'a, F doit implémenter Fn(&'a str) -> &'a str"
// La closure est valide quelle que soit la lifetime choisie par l'appelant
fn apply_to_str<F>(f: F, s: &str) -> String
where
    F: for<'a> Fn(&'a str) -> &'a str,
{
    f(s).to_string()
}

// Exemple d'utilisation
let result = apply_to_str(|s| s.trim(), "  hello  ");
assert_eq!(result, "hello");
```

### HRTB dans les traits objets

```rust
// Un trait object qui accepte n'importe quelle lifetime
trait Parser {
    fn parse<'a>(&self, input: &'a str) -> Option<&'a str>;
}

// Equivalent avec HRTB explicite (nécessaire parfois pour le compilateur)
fn use_parser(p: &dyn for<'a> Fn(&'a str) -> Option<&'a str>, input: &str) {
    if let Some(result) = p(input) {
        println!("Parsed: {result}");
    }
}

// Parsers sans état — closures sans capture
use_parser(&|s| s.strip_prefix("hello: "), "hello: world");
```

### HRTB en pratique — callbacks avec lifetime

```rust
struct EventHandler {
    handlers: Vec<Box<dyn for<'a> Fn(&'a Event) -> bool>>,
}

struct Event {
    name: String,
    data: Vec<u8>,
}

impl EventHandler {
    fn register<F>(&mut self, f: F)
    where
        F: for<'a> Fn(&'a Event) -> bool + 'static,
    {
        self.handlers.push(Box::new(f));
    }

    fn dispatch(&self, event: &Event) -> bool {
        self.handlers.iter().all(|h| h(event))
    }
}
```

> [!info] HRTB et le compilateur
> Dans la plupart des cas, Rust infère les HRTB automatiquement pour les closures. Vous en avez explicitement besoin principalement lors de l'utilisation de trait objects (`dyn Fn(...)`) avec des lifetimes non triviales.

---

## 5. Variance

La variance décrit comment les sous-types se propagent à travers les types génériques.

### Covariance

Un type `F<T>` est **covariant** en `T` si `T: U` implique `F<T>: F<U>`.

```rust
// &'a T est covariant en 'a :
// Si 'long: 'short (long outlive short), alors &'long T: &'short T
// = une référence longue peut être utilisée là où une référence courte est attendue
fn covariant_example<'long, 'short>(x: &'long str) -> &'short str
where
    'long: 'short,
{
    x  // OK : 'long → 'short, covariance
}

// Les Box<T>, Vec<T>, Option<T> sont covariants en T
// (si T outlives U, Box<T> peut être coercé en Box<U>... en théorie)
```

### Contravariance

Un type `F<T>` est **contravariant** en `T` si `T: U` implique `F<U>: F<T>` (inversion).

```rust
// fn(T) est contravariant en T :
// Une fonction qui accepte un type "plus large" peut remplacer
// une fonction qui accepte un type "plus précis"

// fn(&'static str) peut être utilisé où fn(&'a str) est attendu
// car 'static est plus "précis" que 'a, mais pour les paramètres,
// c'est l'inverse qui compte
```

### Invariance

Un type est **invariant** si aucune substitution de lifetime n'est permise.

```rust
// &mut T est INVARIANT en T — c'est crucial pour la sécurité mémoire
fn invariant_problem() {
    let mut s: &'static str = "hello";
    let r: &mut &'static str = &mut s;

    // Si &mut T était covariant, on pourrait faire :
    // let r: &mut &'short str = r;  // "réduire" la lifetime
    // *r = "another";  // assigner une référence courte
    // drop(r);
    // println!("{s}");  // DANGLING ! s pointerait sur une string morte
    // → L'invariance de &mut T empêche exactement ça
}

// Cell<T> et UnsafeCell<T> sont aussi invariants en T
```

### `PhantomData<T>` — contrôler la variance

```rust
use std::marker::PhantomData;

// Un type qui "possède" un T mais ne le stocke pas directement
// (utile pour les types qui gèrent de la mémoire manuellement)
struct MyVec<T> {
    ptr: *mut T,
    len: usize,
    cap: usize,
    // Sans PhantomData, Rust considère MyVec comme invariant en T
    // (car *mut T est invariant)
    _marker: PhantomData<T>,  // Rend MyVec covariant en T (comme Vec<T>)
}

// Marquer qu'un type "contient" une lifetime sans vraiment la stocker
struct Slice<'a, T> {
    ptr: *const T,
    len: usize,
    _lifetime: PhantomData<&'a T>,  // Covariant en 'a et T
}

// Marquer qu'un type ne doit PAS être Send/Sync
struct NotSend<T> {
    data: T,
    _not_send: PhantomData<*mut T>,  // *mut T n'est pas Send
}
```

> [!warning] PhantomData et drop check
> Si votre type "possède" des T (les libère dans son `drop`), utilisez `PhantomData<T>`. Si votre type emprunte des T, utilisez `PhantomData<&'a T>` ou `PhantomData<&'a mut T>`. Le drop checker utilise cette information pour vérifier que les T ne sont pas utilisés après leur liberation.

---

## 6. `Pin<T>` — le problème fondamental

### Pourquoi Pin existe

Les futures `async`/`await` en Rust sont des **state machines auto-référentielles** : après le premier `poll()`, la future peut contenir des pointeurs vers ses propres champs. Si on déplace la future en mémoire, ces pointeurs deviennent invalides.

```rust
// Une future auto-référentielle (simplifiée)
async fn auto_ref_example() {
    let data = vec![1, 2, 3];
    let reference = &data[0];  // référence vers le champ 'data' de la future

    // Entre deux points de suspension (await), la future est une struct
    // qui contient BOTH 'data' AND 'reference' (pointeur vers data[0])
    // Si la future est déplacée, 'reference' devient invalide !

    some_async_fn().await;  // Point de suspension

    println!("{reference}");  // Utilise la référence après le point de suspension
}
```

### La structure de `Pin`

```rust
// Pin est un wrapper sur un pointeur (P) qui garantit que la valeur pointée
// ne sera jamais déplacée (moved) tant que le Pin existe
pub struct Pin<P> {
    pointer: P,  // champ privé — inaccessible directement
}

// Méthodes clés :
impl<P: Deref> Pin<P> {
    // Créer un Pin depuis un type Unpin (sûr car les types Unpin peuvent être déplacés)
    pub fn new(pointer: P) -> Pin<P>
    where P::Target: Unpin { /* ... */ }

    // Obtenir une référence immutable (toujours sûr)
    pub fn as_ref(&self) -> Pin<&P::Target> { /* ... */ }
}

impl<P: DerefMut> Pin<P> {
    // Obtenir une référence mutable (sûr uniquement si la cible est Unpin)
    pub fn as_mut(&mut self) -> Pin<&mut P::Target> { /* ... */ }

    // UNSAFE : obtenir une &mut même pour un type !Unpin
    // Contrat : vous devez garantir de ne jamais déplacer la valeur
    pub unsafe fn get_unchecked_mut(self) -> &'a mut P::Target { /* ... */ }
}
```

---

## 7. `Unpin` — le trait marqueur

```rust
// Unpin est un trait auto-implémenté par le compilateur
// pour les types qui peuvent être déplacés en toute sécurité
pub auto trait Unpin {}

// La quasi-totalité des types Rust sont Unpin :
// i32, String, Vec<T>, structs sans auto-références, etc.

// Les futures générées par async/await sont !Unpin
// (elles peuvent contenir des auto-références)

// Vous pouvez opt-out manuellement :
use std::marker::PhantomPinned;

struct SelfReferential {
    data: String,
    ptr: *const String,  // pointe vers self.data
    _pin: PhantomPinned, // Rend le type !Unpin
}
```

### Visualisation de la hiérarchie

```
Tous les types
├── Unpin (peuvent être déplacés librement)
│   ├── i32, String, Vec<T>, Box<T>, ...
│   └── → Pin::new() fonctionne directement
└── !Unpin (ne peuvent PAS être déplacés après pinnage)
    ├── futures async/await
    ├── types avec PhantomPinned
    └── → Besoin de Box::pin() ou pin!() macro
```

---

## 8. Créer des valeurs pinnées

### `Box::pin()` — pin sur le heap

```rust
use std::future::Future;
use std::pin::Pin;

// Pin sur le heap — la valeur est allouée et ne peut plus être déplacée
let pinned: Pin<Box<dyn Future<Output = i32>>> = Box::pin(async { 42 });

// Utilisation typique pour stocker des futures dans des structs
struct FutureRunner {
    future: Pin<Box<dyn Future<Output = ()> + Send>>,
}

impl FutureRunner {
    fn new<F: Future<Output = ()> + Send + 'static>(f: F) -> Self {
        Self { future: Box::pin(f) }
    }
}
```

### `pin!()` macro — pin sur la stack (Rust 1.68+)

```rust
use std::pin::pin;

async fn my_async_fn() {
    // Pin une future sur la stack sans allocation heap
    let future = pin!(some_async_operation());
    // 'future' est maintenant Pin<&mut SomeAsyncOperation>

    // Utile pour les runtimes custom ou les select! macro
    tokio::select! {
        result = future => println!("Terminé: {result:?}"),
        _ = tokio::time::sleep(Duration::from_secs(1)) => println!("Timeout"),
    }
}
```

### `Pin::new_unchecked()` — unsafe

```rust
// Pour créer un Pin sur un type !Unpin sans Box
// UNSAFE : vous devez garantir que la valeur ne sera jamais déplacée
let mut data = SelfReferential { /* ... */ };
let pinned = unsafe { Pin::new_unchecked(&mut data) };
// Désormais, data ne doit JAMAIS être déplacé tant que pinned vit
```

---

## 9. Projections Pin — accéder aux champs

Le problème : si `Pin<&mut Struct>`, comment accéder à `Pin<&mut FieldType>` ?

```toml
# Cargo.toml
[dependencies]
pin-project = "1"
# ou la version lite (sans proc-macro, mais plus limitée)
pin-project-lite = "0.2"
```

### Avec `pin-project`

```rust
use pin_project::pin_project;
use std::pin::Pin;
use std::future::Future;
use std::task::{Context, Poll};

#[pin_project]
struct TimedFuture<F: Future> {
    #[pin]  // Ce champ doit être projeté comme Pin<&mut F>
    inner: F,

    // Ce champ n'a pas besoin d'être pinned (Unpin)
    started_at: std::time::Instant,
}

impl<F: Future> Future for TimedFuture<F> {
    type Output = (F::Output, std::time::Duration);

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        let this = self.project();  // Génère les projections sûres
        // this.inner est Pin<&mut F> (projeté)
        // this.started_at est &mut Instant (non projeté, car Unpin)

        match this.inner.poll(cx) {
            Poll::Ready(output) => {
                let elapsed = this.started_at.elapsed();
                Poll::Ready((output, elapsed))
            }
            Poll::Pending => Poll::Pending,
        }
    }
}
```

### Avec `pin-project-lite`

```rust
use pin_project_lite::pin_project;

pin_project! {
    struct Stream<S> {
        #[pin]
        inner: S,
        buffer: Vec<u8>,
    }
}
```

---

## 10. `Future` et `Poll` — la signature complète

```rust
use std::future::Future;
use std::task::{Context, Poll};
use std::pin::Pin;

pub trait Future {
    type Output;

    // POURQUOI Pin<&mut Self> ?
    // → &mut Self permettrait de déplacer la future (mem::replace, etc.)
    // → Pin<&mut Self> garantit que la future ne sera jamais déplacée
    //   entre deux appels à poll()
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

// Implémenter Future manuellement — utile pour des primitives d'async customs
struct ReadyFuture<T>(Option<T>);

impl<T: Unpin> Future for ReadyFuture<T> {
    type Output = T;

    fn poll(mut self: Pin<&mut Self>, _cx: &mut Context<'_>) -> Poll<T> {
        // get_mut() fonctionne car T: Unpin
        Poll::Ready(self.get_mut().0.take().expect("polled after completion"))
    }
}
```

---

## 11. Futures avec lifetimes dans `async fn`

```rust
// async fn génère implicitement une future avec des lifetimes

// Ce code :
async fn process<'a>(data: &'a str) -> &'a str {
    tokio::time::sleep(std::time::Duration::from_millis(10)).await;
    data.trim()
}

// Est approximativement désucré en :
fn process_explicit<'a>(data: &'a str) -> impl Future<Output = &'a str> + 'a {
    async move { data.trim() }
}

// La future capture `data` et la lifetime 'a est propagée au type de retour
// Le + 'a est crucial : sans lui, la future ne pourrait pas référencer data
```

### Stocker une future qui emprunte des données

```rust
use std::pin::Pin;
use std::future::Future;

// Problème : impossible de stocker une future qui emprunte self
// directement dans une struct (lifetime circulaire)

// Solution : Box<dyn Future> avec une lifetime explicite
struct Processor {
    data: String,
}

impl Processor {
    // Retourne une future qui emprunte &self — OK pour l'appelant
    async fn run(&self) -> usize {
        tokio::time::sleep(std::time::Duration::from_millis(10)).await;
        self.data.len()
    }

    // Si vous devez stocker la future : Pin<Box<dyn Future + '_>>
    fn run_boxed(&self) -> Pin<Box<dyn Future<Output = usize> + '_>> {
        Box::pin(async move { self.data.len() })
    }
}
```

---

## 12. Patterns pratiques — récapitulatif

### Pattern 1 — Async trait avec Pin (pré-`async fn` in traits)

```rust
use std::future::Future;
use std::pin::Pin;

// Avant Rust 1.75, les async fn dans les traits n'existaient pas
// → on utilisait Pin<Box<dyn Future>> explicitement
trait AsyncProcessor {
    fn process<'a>(&'a self, input: &'a str)
        -> Pin<Box<dyn Future<Output = String> + Send + 'a>>;
}

struct MyProcessor;

impl AsyncProcessor for MyProcessor {
    fn process<'a>(&'a self, input: &'a str)
        -> Pin<Box<dyn Future<Output = String> + Send + 'a>>
    {
        Box::pin(async move { input.to_uppercase() })
    }
}

// Depuis Rust 1.75, vous pouvez écrire directement :
trait ModernAsyncProcessor {
    async fn process(&self, input: &str) -> String;
}
```

### Pattern 2 — Select sur plusieurs futures

```rust
use tokio::pin;

async fn race_two_operations() {
    let op1 = async { tokio::time::sleep(Duration::from_millis(100)).await; "op1" };
    let op2 = async { tokio::time::sleep(Duration::from_millis(200)).await; "op2" };

    // pin! évite l'allocation Box
    pin!(op1);
    pin!(op2);

    tokio::select! {
        result = op1 => println!("Gagnant: {result}"),
        result = op2 => println!("Gagnant: {result}"),
    }
}
```

### Pattern 3 — Diagnostiquer les erreurs Pin

```rust
// Erreur fréquente :
// error[E0277]: `F` cannot be unpinned
// = help: consider using `Box::pin`

// Quand vous voyez cette erreur sur un Future :
// 1. Wrappez avec Box::pin(future) si vous avez besoin de l'allouer
// 2. Utilisez pin!(future) si vous voulez rester sur la stack
// 3. Ajoutez + Unpin bound si votre type n'a pas besoin d'être !Unpin

fn poll_future<F: Future<Output = ()> + Unpin>(f: &mut F) {
    // Unpin permet d'utiliser Pin::new directement
    let pinned = Pin::new(f);
    // ...
}
```

---

## 13. Exercices pratiques

### Exercice 1 — Annotations de lifetimes

Annotez ces fonctions avec les lifetimes appropriées (certaines peuvent être élidées) :

```rust
// À annoter :
fn find_in(haystack: &str, needle: &str) -> Option<&str>;
fn join<'??>(a: &str, b: &str, sep: &str) -> String;
fn first_and_last(items: &[String]) -> (&str, &str);

struct Cache {
    data: HashMap<String, String>,
}
impl Cache {
    fn get(&self, key: &str) -> Option<&str>;
    fn get_or_default<'??>(cache: &Cache, key: &str, default: &str) -> &str;
}
```

### Exercice 2 — HRTB

Implémentez une fonction `map_strings` qui prend une slice de `&str` et une closure `F: for<'a> Fn(&'a str) -> &'a str` et retourne un `Vec<&str>`. Vérifiez qu'elle fonctionne avec `.trim()` et `.split_at(3).0`.

### Exercice 3 — PhantomData et variance

Créez un type `TypedId<T>` qui wraps un `u64` mais est marqué comme "appartenant à" un type `T`. Le type doit être :
- Covariant en T
- `Copy + Clone`
- Affichable (impl Display)
- Non convertible entre `TypedId<User>` et `TypedId<Post>` sans conversion explicite

### Exercice 4 — Future manuelle

Implémentez une future `Delay` qui se comporte comme `tokio::time::sleep` mais sans utiliser Tokio — utilisez `std::thread::sleep` dans un thread séparé et un `std::sync::mpsc::channel` pour signaler la completion. La future doit implémenter `Future<Output = ()>`.

### Exercice 5 — pin-project

Créez un combinateur de future `WithTimeout<F>` qui :
1. Poll la future intérieure `F`
2. Si elle n'est pas terminée après `deadline`, retourne `Err(TimeoutError)`
3. Sinon, retourne `Ok(F::Output)`

Utilisez `pin-project` pour projeter correctement les champs.

---

## Notes liées

- [[01 - Introduction a Rust]] — bases du langage
- [[02 - Ownership et Borrowing]] — lifetimes de base, borrow checker
- [[03 - Structs Enums et Pattern Matching]] — structs avec références
- [[04 - Gestion des Erreurs en Rust]] — erreurs dans les futures
- [[06 - Traits et Generiques]] — bounds de lifetime, HRTB
- [[08 - Concurrence et Smart Pointers]] — `Arc`, `Mutex`, et les lifetimes thread-safe
- [[09 - Async Await et Tokio]] — Pin et Unpin sont au cœur du modèle async
- [[11 - Unsafe Rust et FFI]] — PhantomData, variance, unsafe Pin
- [[16 - Serde Serialisation et Deserialisation]] — `'de` lifetime dans `Deserialize<'de>`
- [[18 - Closures FnOnce FnMut Fn en Profondeur]] — closures et lifetimes de capture
