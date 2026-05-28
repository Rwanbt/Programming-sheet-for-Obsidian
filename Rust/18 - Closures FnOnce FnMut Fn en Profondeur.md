# 18 - Closures : FnOnce, FnMut, Fn en Profondeur

> [!info] Pré-requis
> Ce cours suppose la maîtrise de [[05 - Collections et Iterateurs]] (usage des closures avec `map`, `filter`) et de [[06 - Traits et Generiques]] (bounds, dispatch statique vs dynamique). La section sur les captures avancées se connecte à [[17 - Lifetimes Avances Pin et Unpin]] et [[08 - Concurrence et Smart Pointers]].

---

## 1. Anatomie d'une closure

### Qu'est-ce qu'une closure ?

Une closure est une **fonction anonyme qui capture son environnement**. En Rust, ce n'est pas un type primitif : le compilateur génère une struct unique pour chaque closure, portant les variables capturées comme champs.

```rust
fn main() {
    let threshold = 10;           // variable de l'environnement
    let is_big = |n: i32| n > threshold;  // closure qui capture threshold

    println!("{}", is_big(5));    // false
    println!("{}", is_big(15));   // true
}
```

### Désucrage par le compilateur

```rust
// Ce que vous écrivez :
let threshold = 10;
let is_big = |n: i32| n > threshold;

// Ce que le compilateur génère approximativement :
struct IsBigClosure {
    threshold: i32,  // variable capturée
}

impl FnOnce<(i32,)> for IsBigClosure {
    type Output = bool;
    extern "rust-call" fn call_once(self, args: (i32,)) -> bool {
        args.0 > self.threshold
    }
}

impl FnMut<(i32,)> for IsBigClosure {
    extern "rust-call" fn call_mut(&mut self, args: (i32,)) -> bool {
        args.0 > self.threshold
    }
}

impl Fn<(i32,)> for IsBigClosure {
    extern "rust-call" fn call(&self, args: (i32,)) -> bool {
        args.0 > self.threshold
    }
}

// is_big est une valeur de type IsBigClosure (anonyme, unique)
let is_big = IsBigClosure { threshold };
```

> [!tip] Le type d'une closure est unique
> Chaque closure a un type unique et opaque, même si deux closures ont exactement la même signature et le même corps. C'est pourquoi on utilise des génériques ou des trait objects pour les passer en paramètre.

---

## 2. Les 3 modes de capture

Rust choisit le mode de capture **le moins restrictif possible** pour chaque variable.

### Capture par référence `&T`

```rust
let data = vec![1, 2, 3];

// data est capturé par &Vec<i32> — lecture seule
let sum = || data.iter().sum::<i32>();

println!("{}", sum());  // 6
println!("{data:?}");   // data est toujours accessible — non consommé
```

### Capture par référence mutable `&mut T`

```rust
let mut count = 0;

// count est capturé par &mut i32 — modifiable
let mut increment = || { count += 1; };

increment();
increment();
// println!("{count}");  // ERREUR ici : increment tient &mut count
drop(increment);
println!("{count}");  // OK maintenant : 2
```

### Capture par valeur (move)

```rust
let data = vec![1, 2, 3];

// data est capturé par valeur — ownership transféré dans la closure
let owns_data = || println!("{data:?}");

// println!("{data:?}");  // ERREUR : data a été moved dans la closure
owns_data();  // [1, 2, 3]
```

### Comment Rust choisit le mode

Le compilateur inspecte **comment chaque variable est utilisée** dans le corps de la closure :

```rust
let s = String::from("hello");
let n = 42;

// Rust choisit pour chaque variable indépendamment :
let closure = || {
    println!("{n}");    // n est Copy → capturé par copie (équivalent &n puis copie)
    println!("{s}");    // s n'est pas Copy, lecture seule → &s
};

// Ordre de priorité (du moins au plus restrictif) :
// &T (si seule la lecture est nécessaire)
// &mut T (si la modification est nécessaire)
// T (move, si la valeur doit être consommée)
```

### `move` — forcer la capture par valeur

```rust
let message = String::from("hello from thread");

// ERREUR sans move : le thread pourrait outlive la variable locale
let handle = std::thread::spawn(move || {
    println!("{message}");  // message est maintenant owned par la closure
});

handle.join().unwrap();
// println!("{message}");  // ERREUR : message a été moved
```

> [!warning] `move` est tout-ou-rien
> `move` force toutes les captures à être par valeur. Vous ne pouvez pas demander "capture x par valeur mais y par référence". Pour capturer y par référence dans une closure `move`, créez d'abord une référence à y et capturez la référence.
>
> ```rust
> let data = vec![1, 2, 3];
> let data_ref = &data;  // référence explicite
> let closure = move || println!("{data_ref:?}");
> // data_ref (la référence) est capturée par move
> // mais data lui-même n'est pas moved
> ```

---

## 3. Les 3 traits de closure

### `FnOnce` — peut être appelée au moins une fois

Toute closure implémente `FnOnce`. La closure consomme (`move`) ses captures ou les capture par valeur.

```rust
fn consume<F: FnOnce() -> String>(f: F) -> String {
    f()  // f est consommé ici
    // f()  // ERREUR : f a été consumed par le premier appel
}

let greeting = String::from("hello");
// Cette closure déplace greeting → FnOnce seulement
let say_hello = move || greeting;  // greeting est returned = moved out

let result = consume(say_hello);
println!("{result}");
// consume(say_hello);  // ERREUR : say_hello a été consumed
```

### `FnMut` — peut être appelée plusieurs fois en modifiant les captures

```rust
fn apply_twice<F: FnMut()>(mut f: F) {
    f();
    f();
}

let mut count = 0;
// Cette closure modifie count → FnMut (mais pas Fn)
apply_twice(|| count += 1);
println!("{count}");  // 2
```

### `Fn` — peut être appelée plusieurs fois sans modifier les captures

```rust
fn apply_n_times<F: Fn(i32) -> i32>(f: F, x: i32, n: u32) -> i32 {
    (0..n).fold(x, |acc, _| f(acc))
}

let double = |x| x * 2;  // Fn : ne modifie rien, peut être appelée N fois
let result = apply_n_times(double, 1, 10);
println!("{result}");  // 1024
```

### Hiérarchie et sous-traits

```
Fn : FnMut : FnOnce

Tout type Fn implémente aussi FnMut et FnOnce.
Tout type FnMut implémente aussi FnOnce.
```

```rust
// Une fonction qui accepte FnOnce accepte TOUTES les closures
fn call_once<F: FnOnce()>(f: F) { f(); }

// Une fonction qui accepte Fn est la plus restrictive
// Elle n'accepte que les closures qui peuvent être appelées plusieurs fois
// sans modification (pas de &mut capture, pas de move-out)
fn call_repeatedly<F: Fn()>(f: F) {
    for _ in 0..3 { f(); }
}

// Comment savoir quel trait une closure implémente ?
// → En regardant ce qu'elle fait avec ses captures :

let x = 5;
let read_only = || println!("{x}");        // Fn (lecture seule)
let mut n = 0;
let modify = || { n += 1; };               // FnMut (modification)
let s = String::from("hi");
let consume_s = || drop(s);               // FnOnce (consomme s)
```

> [!info] Tableau récapitulatif
>
> | Trait | Reçoit | Corps autorisé | Appelable |
> |---|---|---|---|
> | `Fn` | `&self` | Lecture des captures | N fois |
> | `FnMut` | `&mut self` | Lecture + écriture des captures | N fois |
> | `FnOnce` | `self` | Move-out possible | 1 fois |

---

## 4. Closures comme paramètres

### Bounds génériques — dispatch statique (monomorphisation)

```rust
// La forme la plus performante : le compilateur génère une version
// spécialisée pour chaque type de closure (inline possible)
fn apply<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(x)
}

// Syntaxe équivalente avec impl Trait (préférée pour la lisibilité)
fn apply_impl(f: impl Fn(i32) -> i32, x: i32) -> i32 {
    f(x)
}
```

### Trait objects — dispatch dynamique

```rust
// Pas de monomorphisation — dispatch via vtable (légèrement plus lent)
// Nécessaire quand le type exact de la closure n'est pas connu à la compilation
fn apply_dyn(f: &dyn Fn(i32) -> i32, x: i32) -> i32 {
    f(x)
}

// Cas d'usage : stocker des closures hétérogènes dans une collection
let handlers: Vec<Box<dyn Fn(i32) -> i32>> = vec![
    Box::new(|x| x + 1),
    Box::new(|x| x * 2),
    Box::new(|x| x - 3),
];

let result = handlers.iter().fold(10, |acc, f| f(acc));
println!("{result}");  // (10 + 1) * 2 - 3 = 19
```

### Tableau comparatif — quand utiliser quoi

| Syntaxe | Dispatch | Taille binaire | Inline | Cas d'usage |
|---|---|---|---|---|
| `F: Fn(...)` générique | Statique | Augmente (monomorphisation) | ✅ Oui | Performance critique, une closure à la fois |
| `impl Fn(...)` | Statique | Augmente | ✅ Oui | Même que générique, syntaxe plus lisible |
| `&dyn Fn(...)` | Dynamique | Stable | ❌ Non | Collections, callbacks runtime |
| `Box<dyn Fn(...)>` | Dynamique | Stable | ❌ Non | Owned, stocké dans une struct |

### Multiple closures avec bounds

```rust
// Composer deux closures
fn compose<A, B, C, F, G>(f: F, g: G) -> impl Fn(A) -> C
where
    F: Fn(A) -> B,
    G: Fn(B) -> C,
{
    move |x| g(f(x))
}

let add_one = |x: i32| x + 1;
let double = |x: i32| x * 2;
let add_then_double = compose(add_one, double);

println!("{}", add_then_double(5));  // (5 + 1) * 2 = 12
```

---

## 5. Retourner des closures

### Le problème fondamental

```rust
// ERREUR : type de la closure inconnu à la compilation
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n  // OK : une seule closure possible
}

// ERREUR : deux branches, deux types de closure différents
fn make_op(add: bool) -> impl Fn(i32) -> i32 {
    if add {
        move |x| x + 1  // type A
    } else {
        move |x| x - 1  // type B — incompatible avec type A !
    }
}
```

### Solution — `Box<dyn Fn(...)>`

```rust
// Box<dyn Fn> efface le type concret → dispatch dynamique
fn make_op(add: bool) -> Box<dyn Fn(i32) -> i32> {
    if add {
        Box::new(move |x| x + 1)
    } else {
        Box::new(move |x| x - 1)
    }
}

let op = make_op(true);
println!("{}", op(5));  // 6
```

### Closures retournées et variables locales

```rust
// ERREUR : la closure retourne une référence vers une variable locale
fn make_greeting(name: String) -> impl Fn() -> String {
    move || format!("Hello, {name}!")
    // name est moved dans la closure — OK, car la closure l'own
}

let greet = make_greeting("Alice".to_string());
println!("{}", greet());  // Hello, Alice!
println!("{}", greet());  // Fonctionne plusieurs fois car Fn
```

### `impl Fn` vs `Box<dyn Fn>` — choisir

```rust
// impl Fn : une seule closure possible (en pratique)
fn filter_factory(min: i32) -> impl Fn(i32) -> bool {
    move |x| x >= min
}

// Box<dyn Fn> : plusieurs closures possibles, ou besoin d'ownership dynamique
fn make_pipeline(steps: Vec<String>) -> Box<dyn Fn(String) -> String> {
    Box::new(move |mut s| {
        for step in &steps {
            match step.as_str() {
                "upper" => s = s.to_uppercase(),
                "trim"  => s = s.trim().to_string(),
                _       => {}
            }
        }
        s
    })
}
```

---

## 6. Closures et les traits standard

### Iterateurs — le cas d'usage le plus courant

```rust
let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// map : Fn(T) -> U — transforme chaque élément
let doubled: Vec<i32> = numbers.iter().map(|&x| x * 2).collect();

// filter : Fn(&T) -> bool — garde les éléments qui satisfont le prédicat
let evens: Vec<&i32> = numbers.iter().filter(|&&x| x % 2 == 0).collect();

// flat_map : Fn(T) -> IntoIterator — aplatit les résultats imbriqués
let words = vec!["hello world", "foo bar"];
let individual: Vec<&str> = words.iter().flat_map(|s| s.split(' ')).collect();

// fold/reduce : FnMut(Acc, T) -> Acc — accumulation
let sum: i32 = numbers.iter().fold(0, |acc, &x| acc + x);

// for_each : FnMut(T) — effet de bord pour chaque élément
numbers.iter().for_each(|x| print!("{x} "));

// take_while / skip_while : Fn(&T) -> bool
let prefix: Vec<&i32> = numbers.iter().take_while(|&&x| x < 5).collect();
```

### Option et Result

```rust
let opt: Option<i32> = Some(5);

// map : transforme la valeur si Some
let doubled = opt.map(|x| x * 2);  // Some(10)

// and_then : chaîne des opérations qui peuvent échouer (= flatMap)
let result = opt
    .and_then(|x| if x > 0 { Some(x) } else { None })
    .map(|x| x.to_string());

// unwrap_or_else : valeur par défaut calculée lazily (évite l'évaluation si Some)
let value = opt.unwrap_or_else(|| expensive_computation());

// filter : None si le prédicat est faux
let filtered = opt.filter(|&x| x > 3);  // Some(5)

// Result — même patterns
let res: Result<i32, String> = Ok(42);
let mapped = res.map(|x| x * 2);
let chained = res.and_then(|x| if x > 0 { Ok(x) } else { Err("negative".into()) });
let error_mapped = res.map_err(|e| format!("Error: {e}"));
```

### Tri avec closures

```rust
let mut people = vec![
    ("Alice", 30),
    ("Bob", 25),
    ("Charlie", 35),
];

// sort_by : FnMut(&T, &T) -> Ordering
people.sort_by(|a, b| a.1.cmp(&b.1));  // par âge croissant

// sort_by_key : FnMut(&T) -> K (plus lisible pour les cas simples)
people.sort_by_key(|&(name, _)| name);  // alphabétique

// sort_by avec ordre inversé
people.sort_by(|a, b| b.1.cmp(&a.1));  // âge décroissant
```

### `std::thread::spawn` — les 3 contraintes

```rust
use std::thread;

fn spawn_task<F, T>(f: F) -> thread::JoinHandle<T>
where
    F: FnOnce() -> T,  // FnOnce : la closure est appelée une fois dans le thread
    F: Send,            // Send : la closure peut être envoyée vers un autre thread
    F: 'static,         // 'static : pas de références non-statiques
                        // (le thread pourrait outlive toute référence locale)
    T: Send,            // T : le résultat peut être envoyé vers le thread principal
{
    thread::spawn(f)
}

// Pourquoi 'static ?
// Le thread peut survivre à la fonction qui l'a créé
// → toutes les données capturées doivent être owned ou 'static
let data = vec![1, 2, 3];
thread::spawn(move || {  // move : data est owned par la closure
    println!("{data:?}");
});
```

---

## 7. Captures avancées

### Capture champ par champ

```rust
struct Point {
    x: f64,
    y: f64,
    label: String,
}

fn main() {
    let p = Point { x: 1.0, y: 2.0, label: String::from("A") };

    // Rust capture chaque champ indépendamment
    // x et y sont Copy → copiés
    // label est String → borrowed (&label)
    let describe = || format!("({}, {}): {}", p.x, p.y, p.label);

    println!("{}", describe());
    println!("{}", describe());
    // p est toujours accessible car label n'est que borrowed

    // Pour capturer label par valeur (move partiel non supporté directement) :
    let label = p.label;  // move label hors de p
    let describe_owned = move || label.clone();  // move label dans la closure
}
```

### Closures récursives — le problème du type infini

```rust
// ERREUR : le type de factorial est défini en termes de lui-même
// let factorial = |n: u64| if n == 0 { 1 } else { n * factorial(n - 1) };

// Solution 1 : fn pointer (pas de capture, pas de récursion via closure)
fn factorial(n: u64) -> u64 {
    if n == 0 { 1 } else { n * factorial(n - 1) }
}
let f: fn(u64) -> u64 = factorial;

// Solution 2 : Box (allocation + indirection, coupe le type infini)
use std::cell::RefCell;
use std::rc::Rc;

let factorial: Rc<dyn Fn(u64) -> u64> = {
    let rc: Rc<RefCell<Option<Rc<dyn Fn(u64) -> u64>>>> = Rc::new(RefCell::new(None));
    let rc_clone = Rc::clone(&rc);
    let f: Rc<dyn Fn(u64) -> u64> = Rc::new(move |n| {
        if n == 0 { 1 } else { n * rc_clone.borrow().as_ref().unwrap()(n - 1) }
    });
    *rc.borrow_mut() = Some(Rc::clone(&f));
    f
};

println!("{}", factorial(5));  // 120
```

> [!tip] Closures récursives — recommandation
> En pratique, utilisez une fonction `fn` ordinaire pour la récursion. Les closures récursives via `Rc<RefCell<...>>` sont pédagogiquement intéressantes mais rarement justifiées en production.

### Mémoïsation avec closures

```rust
use std::collections::HashMap;

struct Memoize<F, A, R> {
    cache: HashMap<A, R>,
    f: F,
}

impl<F, A, R> Memoize<F, A, R>
where
    F: Fn(A) -> R,
    A: std::hash::Hash + Eq + Clone,
    R: Clone,
{
    fn new(f: F) -> Self {
        Self { cache: HashMap::new(), f }
    }

    fn call(&mut self, arg: A) -> R {
        if let Some(cached) = self.cache.get(&arg) {
            return cached.clone();
        }
        let result = (self.f)(arg.clone());
        self.cache.insert(arg, result.clone());
        result
    }
}

fn main() {
    let mut fib = Memoize::new(|n: u64| {
        // Naïf — une vraie mémoïsation récursive nécessite Rc<RefCell<...>>
        (0..n).fold((0u64, 1u64), |(a, b), _| (b, a + b)).0
    });
    println!("{}", fib.call(10));  // 55
}
```

---

## 8. Function pointers `fn`

### `fn` vs `Fn` — la différence fondamentale

```rust
// fn(i32) -> i32 : pointeur de fonction
// → pas de capture, taille fixe (un pointeur), Copy
// → peut pointer vers des fonctions nommées ET des closures sans capture

fn add_one(x: i32) -> i32 { x + 1 }

let f: fn(i32) -> i32 = add_one;  // fonction nommée → fn pointer
let g: fn(i32) -> i32 = |x| x + 1;  // closure sans capture → fn pointer aussi
// let h: fn(i32) -> i32 = |x| x + threshold;  // ERREUR : a une capture

// Fn(i32) -> i32 : trait — implémenté par fn pointers ET closures
// fn pointers implémentent Fn, FnMut, FnOnce automatiquement
```

### Coercions utiles

```rust
// Passer des fonctions nommées là où une closure est attendue
let numbers = vec![1, 2, 3];

fn is_even(n: &i32) -> bool { n % 2 == 0 }
let evens: Vec<&i32> = numbers.iter().filter(is_even).collect();
// filter attend Fn(&i32) -> bool → is_even coercé automatiquement

// fn pointers dans des tableaux (tous le même type fn(...) -> ...)
let ops: [fn(i32, i32) -> i32; 4] = [
    |a, b| a + b,
    |a, b| a - b,
    |a, b| a * b,
    |a, b| if b != 0 { a / b } else { 0 },
];
```

### FFI — closures et function pointers

```rust
// En FFI (voir [[11 - Unsafe Rust et FFI]]), seules les fn pointers passent
// car les closures ont un type opaque et une taille variable

extern "C" {
    fn qsort(
        base: *mut std::ffi::c_void,
        num: usize,
        size: usize,
        compare: Option<unsafe extern "C" fn(*const std::ffi::c_void, *const std::ffi::c_void) -> i32>,
    );
}

// Callback C safe : une fn extern "C"
unsafe extern "C" fn compare_ints(a: *const std::ffi::c_void, b: *const std::ffi::c_void) -> i32 {
    let a = *(a as *const i32);
    let b = *(b as *const i32);
    a - b
}
```

---

## 9. Patterns pratiques

### Builder pattern avec closures

```rust
struct QueryBuilder {
    table: String,
    conditions: Vec<Box<dyn Fn(&str) -> bool>>,
    limit: Option<usize>,
}

impl QueryBuilder {
    fn new(table: &str) -> Self {
        Self {
            table: table.to_string(),
            conditions: Vec::new(),
            limit: None,
        }
    }

    fn filter<F: Fn(&str) -> bool + 'static>(mut self, condition: F) -> Self {
        self.conditions.push(Box::new(condition));
        self
    }

    fn limit(mut self, n: usize) -> Self {
        self.limit = Some(n);
        self
    }

    fn matches(&self, row: &str) -> bool {
        self.conditions.iter().all(|f| f(row))
    }
}

fn main() {
    let query = QueryBuilder::new("users")
        .filter(|row| row.contains("active"))
        .filter(|row| !row.contains("banned"))
        .limit(10);

    let rows = vec!["alice active", "bob banned active", "charlie active"];
    let results: Vec<&str> = rows.iter()
        .filter(|&&row| query.matches(row))
        .take(query.limit.unwrap_or(usize::MAX))
        .copied()
        .collect();
}
```

### Strategy pattern

```rust
// Le strategy pattern en Rust s'exprime naturellement avec des closures
// et FnMut — plus simple que des traits et des structs séparées

struct DataProcessor {
    transform: Box<dyn Fn(Vec<i32>) -> Vec<i32>>,
    filter: Box<dyn Fn(&i32) -> bool>,
}

impl DataProcessor {
    fn new(
        transform: impl Fn(Vec<i32>) -> Vec<i32> + 'static,
        filter: impl Fn(&i32) -> bool + 'static,
    ) -> Self {
        Self {
            transform: Box::new(transform),
            filter: Box::new(filter),
        }
    }

    fn process(&self, data: Vec<i32>) -> Vec<i32> {
        (self.transform)(data)
            .into_iter()
            .filter(|x| (self.filter)(x))
            .collect()
    }
}

// Strategies interchangeables
let normalize = DataProcessor::new(
    |data| data.into_iter().map(|x| x * 2).collect(),
    |&x| x > 0,
);

let deduplicate = DataProcessor::new(
    |mut data| { data.sort(); data.dedup(); data },
    |_| true,
);
```

### Event handlers / callbacks

```rust
use std::collections::HashMap;

type Handler = Box<dyn Fn(&str) + Send + Sync>;

struct EventBus {
    handlers: HashMap<String, Vec<Handler>>,
}

impl EventBus {
    fn new() -> Self {
        Self { handlers: HashMap::new() }
    }

    fn subscribe<F>(&mut self, event: &str, handler: F)
    where
        F: Fn(&str) + Send + Sync + 'static,
    {
        self.handlers
            .entry(event.to_string())
            .or_default()
            .push(Box::new(handler));
    }

    fn publish(&self, event: &str, data: &str) {
        if let Some(handlers) = self.handlers.get(event) {
            for handler in handlers {
                handler(data);
            }
        }
    }
}

fn main() {
    let mut bus = EventBus::new();

    bus.subscribe("user.login", |data| println!("Login: {data}"));
    bus.subscribe("user.login", |data| println!("Audit: {data}"));

    bus.publish("user.login", "alice");
    // Login: alice
    // Audit: alice
}
```

### `once_cell::sync::Lazy` — initialisation lazy

```rust
use once_cell::sync::Lazy;
use std::collections::HashMap;

// Initialisé une seule fois, la première fois qu'on y accède
// La closure FnOnce est appelée thread-safely
static CONFIG: Lazy<HashMap<&'static str, i32>> = Lazy::new(|| {
    let mut map = HashMap::new();
    map.insert("max_retries", 3);
    map.insert("timeout_ms", 5000);
    map
});

fn get_config(key: &str) -> Option<i32> {
    CONFIG.get(key).copied()
}
```

---

## 10. `once_cell` et closures d'initialisation

```rust
// std::sync::OnceLock (stabilisé Rust 1.70)
use std::sync::OnceLock;

static INSTANCE: OnceLock<ExpensiveResource> = OnceLock::new();

struct ExpensiveResource {
    data: Vec<u8>,
}

fn get_resource() -> &'static ExpensiveResource {
    INSTANCE.get_or_init(|| {
        // Cette closure FnOnce n'est appelée qu'une fois
        println!("Initialisation...");
        ExpensiveResource { data: vec![1, 2, 3] }
    })
}
```

---

## 11. Cas avancé — closures et async

La connexion avec [[09 - Async Await et Tokio]] et [[17 - Lifetimes Avances Pin et Unpin]] :

```rust
use std::future::Future;

// Une closure async est désucre en une closure qui retourne un Future
// Elle implémente FnOnce (souvent), rarement Fn directement

async fn fetch_user(id: u64) -> String {
    format!("user_{id}")
}

// Closure qui retourne un Future — FnMut car pas de move-out
let make_request = |id: u64| async move {
    tokio::time::sleep(std::time::Duration::from_millis(10)).await;
    fetch_user(id).await
};

// Passer une async closure comme paramètre
async fn process_ids<F, Fut>(ids: Vec<u64>, f: F) -> Vec<String>
where
    F: Fn(u64) -> Fut,
    Fut: Future<Output = String>,
{
    let mut results = Vec::new();
    for id in ids {
        results.push(f(id).await);
    }
    results
}
```

---

## 12. Exercices pratiques

### Exercice 1 — Analyser les traits de closure

Pour chaque closure ci-dessous, indiquez si elle implémente `Fn`, `FnMut`, `FnOnce` (ou plusieurs) et expliquez pourquoi :

```rust
let x = 5;
let c1 = || x + 1;

let mut count = 0;
let c2 = || { count += 1; count };

let s = String::from("hello");
let c3 = || { drop(s); };

let v = vec![1, 2, 3];
let c4 = move || v.len();

let mut data = vec![];
let c5 = |x: i32| data.push(x);
```

### Exercice 2 — Implémentation de `map` et `filter`

Sans utiliser les méthodes de l'itérateur standard, implémentez :

```rust
fn my_map<T, U, F: Fn(T) -> U>(input: Vec<T>, f: F) -> Vec<U>;
fn my_filter<T, F: Fn(&T) -> bool>(input: Vec<T>, f: F) -> Vec<T>;
fn my_fold<T, Acc, F: FnMut(Acc, T) -> Acc>(input: Vec<T>, init: Acc, f: F) -> Acc;
```

Testez avec `my_map(vec![1,2,3], |x| x*2)`, `my_filter(vec![1..=10], |x| x%2==0)`.

### Exercice 3 — Retourner des closures conditionnelles

Implémentez `make_validator(rule: &str) -> Box<dyn Fn(&str) -> bool>` qui crée un validateur selon la règle :
- `"non_empty"` : la string n'est pas vide
- `"email"` : contient `@` et `.`
- `"numeric"` : ne contient que des chiffres
- `"min:N"` : longueur >= N

Composez ensuite plusieurs validateurs avec `fn compose_validators(validators: Vec<Box<dyn Fn(&str) -> bool>>) -> Box<dyn Fn(&str) -> bool>`.

### Exercice 4 — Pipeline de transformation

Créez un type `Pipeline<T>` qui :
1. Stocke une `Vec<Box<dyn Fn(T) -> T>>`
2. Expose `.add_step(f: impl Fn(T) -> T + 'static) -> &mut Self`
3. Expose `.run(input: T) -> T` qui applique toutes les étapes en séquence

Testez avec un pipeline de transformation de strings : trim → lowercase → replace spaces with `-`.

### Exercice 5 — Mémoïsation générique

Implémentez `Memoize<F>` où `F: Fn(u64) -> u64` avec :
- Cache interne `HashMap<u64, u64>`
- Méthode `call(&mut self, n: u64) -> u64`
- Méthode `cache_size(&self) -> usize`
- Méthode `clear_cache(&mut self)`

Testez avec la suite de Fibonacci (même version non-récursive) et vérifiez que les appels répétés n'exécutent pas `f` deux fois pour la même entrée.

### Exercice 6 — Event system thread-safe

Étendez le `EventBus` de la section 9 pour supporter :
- Désabonnement via un `SubscriptionId` retourné par `subscribe()`
- `publish_async()` qui publie sur un `tokio::runtime` dédié
- Thread-safety : `EventBus` doit être `Send + Sync`

---

## Notes liées

- [[01 - Introduction a Rust]] — bases du langage, fonctions
- [[02 - Ownership et Borrowing]] — ownership et captures par valeur/référence
- [[05 - Collections et Iterateurs]] — `map`, `filter`, `fold` et les closures d'itérateur
- [[06 - Traits et Generiques]] — Fn, FnMut, FnOnce sont des traits, bounds, dispatch
- [[08 - Concurrence et Smart Pointers]] — `Send + Sync + 'static` pour les closures thread-safe
- [[09 - Async Await et Tokio]] — closures async, futures retournées par des closures
- [[10 - Macros et Metaprogrammation]] — macros qui génèrent des closures
- [[11 - Unsafe Rust et FFI]] — fn pointers `extern "C"`, FFI sans closures
- [[17 - Lifetimes Avances Pin et Unpin]] — lifetimes capturées dans les closures retournées
- [[16 - Serde Serialisation et Deserialisation]] — `serialize_with` utilise des fn pointers
