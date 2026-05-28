# 11 - Unsafe Rust et FFI

> [!info] Prérequis
> Ce module demande une maîtrise solide de [[02 - Ownership et Borrowing]] et de [[08 - Concurrence et Smart Pointers]]. L'unsafe Rust ne désactive pas le compilateur — il transfère la responsabilité de certaines garanties au développeur.

---

## 1. Qu'est-ce qu'Unsafe Rust ?

Rust garantit la sécurité mémoire et l'absence de data races **à la compilation**. Ces garanties ont un coût : certaines opérations correctes sont impossibles à exprimer sans aide. `unsafe` est l'échappatoire contrôlée.

> [!warning] La règle d'or de l'unsafe
> `unsafe` **isole** l'unsafety — il ne l'élimine pas. Un bloc `unsafe` signifie "le compilateur ne peut pas vérifier ceci, je certifie que c'est correct". La responsabilité de maintenir les invariants de mémoire appartient alors entièrement au développeur.

### Les 5 super-pouvoirs d'unsafe

| Super-pouvoir | Description |
|---------------|-------------|
| **Déréférencer un raw pointer** | `*const T` et `*mut T` sans garanties de validité |
| **Appeler une fonction unsafe** | Toute fn marquée `unsafe fn` ou fonction FFI |
| **Implémenter un unsafe trait** | `unsafe impl Send for T`, `unsafe impl Sync for T` |
| **Accéder à un `static mut`** | Variables globales mutables |
| **Accéder aux champs d'une union** | Les unions n'ont pas de discriminant |

```rust
// Bloc unsafe minimal
unsafe {
    // Les 5 super-pouvoirs sont disponibles ici
}

// Fonction unsafe : appeler cette fn nécessite un bloc unsafe
unsafe fn fonction_dangereuse() {
    // ...
}

// Unsafe trait : implementer ce trait nécessite unsafe impl
unsafe trait TraitDangereux {}
unsafe impl TraitDangereux for i32 {}
```

---

## 2. Raw Pointers — `*const T` et `*mut T`

Les raw pointers sont l'équivalent Rust des pointeurs C. Le compilateur ne garantit rien sur leur validité.

### Créer des raw pointers

```rust
fn main() {
    let mut x = 42i32;

    // Créer des raw pointers depuis des références — toujours sûr
    let ptr_const: *const i32 = &x;
    let ptr_mut: *mut i32 = &mut x;

    // Créer depuis une adresse mémoire brute — dangereux
    let ptr_arbitraire: *const i32 = 0x12345678 as *const i32;

    // Vérifier la nullité
    println!("ptr_const est null : {}", ptr_const.is_null());   // false
    println!("ptr_arbitraire est null : {}", ptr_arbitraire.is_null()); // false

    // Déréférencement — UNSAFE
    unsafe {
        println!("Valeur via ptr_const : {}", *ptr_const);     // 42
        *ptr_mut = 100;
        println!("Valeur après mutation : {}", *ptr_const);    // 100
    }
}
```

### Pointer arithmetic

```rust
fn main() {
    let tableau = [10i32, 20, 30, 40, 50];
    let ptr = tableau.as_ptr(); // *const i32 vers le premier élément

    unsafe {
        for i in 0..5 {
            let valeur = *ptr.add(i); // ptr.add(i) = ptr + i * size_of::<i32>()
            println!("tableau[{}] = {}", i, valeur);
        }
    }
}
```

### Slices depuis raw pointers

```rust
fn main() {
    let mut données = vec![1i32, 2, 3, 4, 5];
    let ptr = données.as_mut_ptr();
    let len = données.len();

    // Créer une slice depuis un raw pointer — UNSAFE
    let slice: &mut [i32] = unsafe {
        std::slice::from_raw_parts_mut(ptr, len)
    };

    // Manipuler comme une slice normale
    slice[0] = 100;
    println!("{:?}", données); // [100, 2, 3, 4, 5]
}
```

---

## 3. Invariants à Maintenir — Éviter l'Undefined Behavior

`unsafe` vous donne la responsabilité de maintenir des invariants que Rust ne peut pas vérifier.

### Aliasing rules

```rust
fn main() {
    let mut x = 42i32;

    // ❌ UB : Deux pointeurs mutables qui alias — violer les règles d'aliasing
    let ptr1: *mut i32 = &mut x;
    // let ptr2: *mut i32 = &mut x; // Illégal même en unsafe si used simultanément
    // *ptr1 = 1; *ptr2 = 2; // UB : deux accès mutables simultanés

    // ✅ Correct : un seul accès mutable à la fois
    unsafe {
        *ptr1 = 100;
    }
    println!("{}", x); // 100
}
```

### Liste des UB courants à éviter

```rust
// ❌ Déréférencer un pointeur null
unsafe {
    // let ptr: *const i32 = std::ptr::null();
    // let _ = *ptr; // UB : null dereference
}

// ❌ Pointeur dangling (vers une valeur drop)
unsafe {
    // let ptr = {
    //     let x = Box::new(42);
    //     x.as_ptr()
    //     // x est drop ici — ptr est dangling
    // };
    // let _ = *ptr; // UB : use after free
}

// ❌ Accès out-of-bounds
unsafe {
    // let tableau = [1, 2, 3];
    // let _ = *tableau.as_ptr().add(100); // UB : hors des bornes
}

// ❌ Mauvais alignement
unsafe {
    // let données: [u8; 5] = [0; 5];
    // let ptr = données.as_ptr().add(1) as *const u32;
    // let _ = *ptr; // UB : u32 requiert un alignement de 4 bytes
}

// ❌ Valeurs invalides pour un type
unsafe {
    // let invalide: u8 = 2;
    // let _: bool = std::mem::transmute(invalide); // UB : bool doit être 0 ou 1
}
```

---

## 4. `MaybeUninit<T>` — Initialisation sécurisée

`mem::uninitialized()` est **deprecated et UB**. `MaybeUninit<T>` est le remplacement sûr.

```rust
use std::mem::MaybeUninit;

fn main() {
    // Allouer un tableau non initialisé (pattern courant en code système)
    let mut tableau: [MaybeUninit<i32>; 5] = unsafe {
        MaybeUninit::uninit().assume_init()
    };

    // Initialiser chaque élément
    for (i, elem) in tableau.iter_mut().enumerate() {
        elem.write(i as i32 * 2); // Écrit la valeur — pas d'UB
    }

    // Lire les valeurs initialisées
    let tableau_init: [i32; 5] = unsafe {
        // SAFETY: Tous les éléments ont été initialisés via .write()
        std::mem::transmute(tableau)
    };

    println!("{:?}", tableau_init); // [0, 2, 4, 6, 8]
}

// Pattern idiomatique : initialisation d'une struct complexe
fn initialiser_struct() -> SomeStruct {
    let mut uninit: MaybeUninit<SomeStruct> = MaybeUninit::uninit();
    
    unsafe {
        let ptr = uninit.as_mut_ptr();
        // Initialiser chaque champ individuellement
        std::ptr::addr_of_mut!((*ptr).champ1).write(42);
        std::ptr::addr_of_mut!((*ptr).champ2).write("hello".to_string());
        
        // SAFETY: Tous les champs ont été initialisés
        uninit.assume_init()
    }
}

struct SomeStruct {
    champ1: i32,
    champ2: String,
}
```

---

## 5. FFI avec C — `extern "C"`

La FFI (Foreign Function Interface) permet d'appeler du code C depuis Rust et vice versa.

### Appeler une fonction C

```rust
// Déclarer des fonctions C externes
extern "C" {
    fn abs(input: i32) -> i32;
    fn sqrt(x: f64) -> f64;
    fn strlen(s: *const std::os::raw::c_char) -> usize;
}

fn main() {
    unsafe {
        println!("abs(-42) = {}", abs(-42)); // 42
        println!("sqrt(9.0) = {}", sqrt(9.0)); // 3.0
    }
}
```

### Linkage statique vs dynamique

```rust
// Linkage dynamique — la lib doit être disponible au runtime
#[link(name = "m")] // -lm sur Linux
extern "C" {
    fn sin(x: f64) -> f64;
}

// Linkage statique — inclus dans le binaire
#[link(name = "ma_lib_c", kind = "static")]
extern "C" {
    fn ma_fonction_c(x: i32) -> i32;
}

// Via build.rs pour des libs custom :
// println!("cargo:rustc-link-lib=static=ma_lib_c");
// println!("cargo:rustc-link-search=native=/path/to/lib");
```

### Types FFI-safe

```rust
use std::os::raw::{
    c_int,     // int C
    c_uint,    // unsigned int C
    c_long,    // long C
    c_ulong,   // unsigned long C
    c_char,    // char C
    c_uchar,   // unsigned char C
    c_float,   // float C
    c_double,  // double C
    c_void,    // void* C
    c_size_t,  // size_t C
};

// Alternativement, via libc crate (recommandé)
// extern crate libc;
// use libc::{c_int, c_char, size_t};
```

### Strings C — `CString` et `CStr`

```rust
use std::ffi::{CString, CStr};
use std::os::raw::c_char;

extern "C" {
    fn puts(s: *const c_char) -> c_int;
    fn afficher_message(s: *const c_char);
}

fn main() {
    // Rust String → CString → *const c_char
    let message = CString::new("Bonjour depuis Rust !").unwrap();
    // CString garantit un null-terminator
    unsafe {
        puts(message.as_ptr());
    }

    // *const c_char → &CStr → &str
    let c_str_ptr = message.as_ptr(); // Simuler un retour de fonction C
    let rust_str = unsafe {
        CStr::from_ptr(c_str_ptr) // SAFETY: ptr valide et null-terminé
            .to_str()             // Result<&str, Utf8Error>
            .expect("String C invalide UTF-8")
    };
    println!("Reçu de C : {}", rust_str);

    // Gérer les erreurs de null dans la string
    match CString::new("contient\0un null") {
        Ok(_) => println!("OK"),
        Err(e) => println!("Erreur : {:?}", e), // NulError
    }
}
```

---

## 6. `bindgen` — Générer des Bindings Automatiquement

`bindgen` lit des headers C/C++ et génère les déclarations Rust correspondantes.

```toml
# Cargo.toml
[build-dependencies]
bindgen = "0.70"
```

```rust
// build.rs
use bindgen;
use std::env;
use std::path::PathBuf;

fn main() {
    println!("cargo:rustc-link-lib=sqlite3");

    let bindings = bindgen::Builder::default()
        .header("wrapper.h")         // Header C à parser
        .allowlist_function("sqlite3_.*") // Filtrer les fonctions
        .allowlist_type("sqlite3.*")
        .parse_callbacks(Box::new(bindgen::CargoCallbacks::new()))
        .generate()
        .expect("Impossible de générer les bindings");

    let out_path = PathBuf::from(env::var("OUT_DIR").unwrap());
    bindings
        .write_to_file(out_path.join("bindings.rs"))
        .expect("Impossible d'écrire les bindings");
}
```

```c
// wrapper.h
#include <sqlite3.h>
```

```rust
// src/lib.rs
#![allow(non_upper_case_globals)]
#![allow(non_camel_case_types)]
#![allow(non_snake_case)]

include!(concat!(env!("OUT_DIR"), "/bindings.rs"));
```

---

## 7. Ownership en FFI — Qui alloue, qui libère

La règle fondamentale : **l'allocateur qui alloue doit libérer**.

```rust
use std::os::raw::c_char;
use std::ffi::CString;

// Cas 1 : Rust passe une string à C — Rust conserve l'ownership
fn passer_string_à_c() {
    let s = CString::new("hello").unwrap();
    unsafe {
        // C reçoit un pointeur — NE DOIT PAS libérer via free()
        // La durée de vie est garantie par `s` qui est encore en scope
        quelque_chose_avec_string(s.as_ptr());
    }
    // s est drop ici → mémoire libérée par Rust
}

// Cas 2 : Rust retourne un pointeur à C — C doit appeler notre deleter
#[no_mangle]
pub extern "C" fn allouer_struct() -> *mut MonStruct {
    let s = Box::new(MonStruct { valeur: 42 });
    Box::into_raw(s) // Convertit Box en raw pointer — Rust ne libère plus
}

#[no_mangle]
pub extern "C" fn libérer_struct(ptr: *mut MonStruct) {
    if !ptr.is_null() {
        unsafe {
            // SAFETY: ptr a été créé par allouer_struct(), appelé une seule fois
            drop(Box::from_raw(ptr)); // Reconvertit en Box → libère la mémoire
        }
    }
}

// Cas 3 : C retourne un pointeur que Rust doit libérer via la fonction C
extern "C" {
    fn c_allouer_buffer(taille: usize) -> *mut u8;
    fn c_libérer_buffer(ptr: *mut u8);
}

struct BufferC {
    ptr: *mut u8,
    taille: usize,
}

impl Drop for BufferC {
    fn drop(&mut self) {
        if !self.ptr.is_null() {
            unsafe {
                c_libérer_buffer(self.ptr); // Libéré via l'allocateur C
            }
        }
    }
}

struct MonStruct { valeur: i32 }

extern "C" {
    fn quelque_chose_avec_string(s: *const c_char);
}
```

---

## 8. `repr(C)` — Layout Compatible C

Par défaut, Rust peut réorganiser les champs d'une struct pour optimiser. `#[repr(C)]` impose le layout C.

```rust
// ❌ Sans repr(C) : l'ordre des champs peut être modifié par le compilateur
struct NePasPasser {
    a: u8,
    b: u32,
    c: u16,
}

// ✅ Avec repr(C) : layout garanti, compatible avec les structs C
#[repr(C)]
struct CompatibleC {
    a: u8,
    _padding: [u8; 3], // Souvent implicite — mais visible avec repr(C)
    b: u32,
    c: u16,
    _padding2: [u8; 2],
}

// Vérification des tailles
fn main() {
    use std::mem;
    println!("Taille NePasPasser : {}", mem::size_of::<NePasPasser>());
    println!("Taille CompatibleC : {}", mem::size_of::<CompatibleC>());
    println!("Alignement CompatibleC : {}", mem::align_of::<CompatibleC>());
}

// Unions en Rust (nécessitent repr(C) pour FFI)
#[repr(C)]
union ValeurC {
    entier: i32,
    flottant: f32,
    octets: [u8; 4],
}

fn main_union() {
    let u = ValeurC { entier: 42 };
    unsafe {
        println!("Comme entier : {}", u.entier);
        println!("Comme octets : {:?}", u.octets);
    }
}
```

---

## 9. Exposer Rust vers C avec `cbindgen`

`cbindgen` génère des headers C depuis du code Rust, permettant à du code C/C++ d'appeler des bibliothèques Rust.

```toml
# Cargo.toml
[lib]
crate-type = ["cdylib", "staticlib"]  # Bibliothèque C dynamique ou statique

[build-dependencies]
cbindgen = "0.26"
```

```rust
// build.rs
fn main() {
    let crate_dir = std::env::var("CARGO_MANIFEST_DIR").unwrap();
    cbindgen::Builder::new()
        .with_crate(crate_dir)
        .with_language(cbindgen::Language::C)
        .generate()
        .expect("Impossible de générer les bindings C")
        .write_to_file("include/ma_lib.h");
}
```

```rust
// src/lib.rs — API publique exposée à C
#[repr(C)]
pub struct Point {
    pub x: f64,
    pub y: f64,
}

/// Calcule la distance entre deux points.
///
/// # Safety
/// `a` et `b` doivent être des pointeurs valides et non-null.
#[no_mangle]
pub extern "C" fn distance(a: *const Point, b: *const Point) -> f64 {
    // SAFETY: L'appelant garantit que les pointeurs sont valides
    let a = unsafe { &*a };
    let b = unsafe { &*b };
    let dx = a.x - b.x;
    let dy = a.y - b.y;
    (dx * dx + dy * dy).sqrt()
}

#[no_mangle]
pub extern "C" fn point_nouveau(x: f64, y: f64) -> *mut Point {
    Box::into_raw(Box::new(Point { x, y }))
}

#[no_mangle]
pub extern "C" fn point_libérer(ptr: *mut Point) {
    if !ptr.is_null() {
        unsafe { drop(Box::from_raw(ptr)); }
    }
}
```

---

## 10. Wrapper sécurisé autour d'une lib C

Le pattern idiomatique : wrapper unsafe FFI derrière une API safe.

```rust
// Exemple : wrapper autour de zlib pour la compression
use std::ffi::c_void;
use std::os::raw::{c_int, c_ulong};

// Déclarations FFI brutes (générées par bindgen normalement)
extern "C" {
    fn compress(
        dest: *mut u8,
        dest_len: *mut c_ulong,
        source: *const u8,
        source_len: c_ulong,
    ) -> c_int;

    fn uncompress(
        dest: *mut u8,
        dest_len: *mut c_ulong,
        source: *const u8,
        source_len: c_ulong,
    ) -> c_int;
}

// Codes de retour zlib
const Z_OK: c_int = 0;

#[derive(Debug)]
pub enum ErreurCompression {
    BufferInsuffisant,
    DonnéesInvalides,
    ErreurInterne(c_int),
}

/// API publique safe — cache toute la FFI unsafe
pub fn compresser(données: &[u8]) -> Result<Vec<u8>, ErreurCompression> {
    let source_len = données.len() as c_ulong;
    // zlib recommande ce calcul pour la taille du buffer de sortie
    let mut dest_len = (source_len as f64 * 1.01 + 12.0) as c_ulong;
    let mut sortie = vec![0u8; dest_len as usize];

    let résultat = unsafe {
        // SAFETY:
        // - sortie.as_mut_ptr() est valide et a la taille dest_len
        // - données.as_ptr() est valide et a la taille source_len
        // - dest_len contient la capacité du buffer de sortie
        compress(
            sortie.as_mut_ptr(),
            &mut dest_len,
            données.as_ptr(),
            source_len,
        )
    };

    match résultat {
        Z_OK => {
            sortie.truncate(dest_len as usize);
            Ok(sortie)
        }
        -5 => Err(ErreurCompression::BufferInsuffisant),
        code => Err(ErreurCompression::ErreurInterne(code)),
    }
}
```

---

## 11. `std::mem` — Utilitaires mémoire

```rust
use std::mem;

fn main() {
    // Taille et alignement des types
    println!("size_of::<i32>() = {}", mem::size_of::<i32>()); // 4
    println!("size_of::<String>() = {}", mem::size_of::<String>()); // 24 (ptr+len+cap)
    println!("align_of::<f64>() = {}", mem::align_of::<f64>()); // 8

    // mem::swap — échanger deux valeurs
    let mut a = String::from("hello");
    let mut b = String::from("world");
    mem::swap(&mut a, &mut b);
    println!("{} {}", a, b); // "world hello"

    // mem::replace — remplacer une valeur et récupérer l'ancienne
    let mut opt = Some(String::from("valeur"));
    let ancienne = mem::replace(&mut opt, None);
    println!("Ancienne : {:?}, Nouvelle : {:?}", ancienne, opt);

    // mem::forget — supprimer la valeur SANS appeler Drop
    let s = String::from("je ne serai pas drop");
    mem::forget(s); // La mémoire est "leakée" intentionnellement
    // Utile pour transférer l'ownership à du code FFI C

    // mem::size_of_val — taille d'une valeur concrète
    let v: Vec<i32> = vec![1, 2, 3];
    println!("size_of_val(&v) = {}", mem::size_of_val(&v)); // 24 (le Vec lui-même, pas le contenu)
}
```

### `mem::transmute` — Reinterprétation dangereuse

```rust
fn main() {
    // ❌ transmute est l'opération la plus dangereuse de Rust
    // Préférez toujours une alternative sûre

    // Convertir u32 → f32 (même taille, OK si valeur valide)
    let bits: u32 = 0x3F800000; // bits IEEE 754 de 1.0f32
    let flottant: f32 = unsafe { std::mem::transmute(bits) };
    println!("{}", flottant); // 1.0

    // ✅ Alternative sûre via méthodes de la lib standard
    let flottant_safe = f32::from_bits(bits);
    println!("{}", flottant_safe); // 1.0 — même résultat, zero unsafe

    // ✅ Convertir une slice de bytes en slice de u32
    // Au lieu de transmute, utiliser bytemuck ou un cast aligné
    let données: &[u8] = &[0x01, 0x02, 0x03, 0x04];
    // let u32s: &[u32] = unsafe { std::mem::transmute(données) }; // risqué
}
```

---

## 12. Documentation Safety — Convention Rust

La convention Rust pour documenter le code unsafe est stricte et universelle.

### Commentaires `// SAFETY:`

```rust
fn déréférencer<T>(ptr: *const T) -> &'static T {
    // SAFETY: Cette fonction requiert que le caller garantisse que :
    // 1. ptr est non-null
    // 2. ptr pointe vers une valeur valide initialisée de type T
    // 3. La valeur est valide pour la durée de vie 'static (ou au moins
    //    aussi longtemps que la référence retournée sera utilisée)
    // 4. Aucun autre accès mutable n'existe simultanément
    unsafe { &*ptr }
}
```

### Section `# Safety` dans les doc comments

```rust
/// Crée une référence depuis un raw pointer.
///
/// # Safety
///
/// Cette fonction est sûre si et seulement si :
/// - `ptr` est non-null
/// - `ptr` est correctement aligné pour le type `T`
/// - `ptr` pointe vers une valeur initialisée de type `T`
/// - La durée de vie de la référence retournée ne dépasse pas
///   celle de la valeur pointée
/// - Aucune référence mutable vers la même valeur n'existe pendant
///   la durée de vie de la référence retournée
///
/// # Exemple
///
/// ```rust
/// let x = 42i32;
/// let ptr = &x as *const i32;
/// let référence: &i32 = unsafe { ref_depuis_ptr(ptr) };
/// assert_eq!(*référence, 42);
/// ```
pub unsafe fn ref_depuis_ptr<'a, T>(ptr: *const T) -> &'a T {
    // SAFETY: Vérifié par le contrat documenté ci-dessus
    &*ptr
}
```

---

## 13. Appels Système Directs

```rust
// Appels système Linux sans libc (pattern rare mais existant)
#[cfg(target_os = "linux")]
mod syscall {
    use std::os::raw::{c_long, c_int};

    // Numéros d'appels système x86_64
    const SYS_WRITE: c_long = 1;
    const STDOUT: c_int = 1;

    pub fn écrire(données: &[u8]) -> isize {
        unsafe {
            // SAFETY: SYS_WRITE avec des arguments valides
            // données.as_ptr() valide pour la durée de l'appel
            std::arch::asm!(
                "syscall",
                inlateout("rax") SYS_WRITE as usize => _,
                in("rdi") STDOUT as usize,
                in("rsi") données.as_ptr() as usize,
                in("rdx") données.len(),
                lateout("rcx") _,
                lateout("r11") _,
            );
        }
        données.len() as isize
    }
}
```

---

## 14. Patterns de Sécurité

### Abstraire unsafe derrière une API sûre

```rust
/// Buffer circulaire lock-free (simplifié pour illustration)
pub struct CircularBuffer<T, const N: usize> {
    données: [std::mem::MaybeUninit<T>; N],
    head: std::sync::atomic::AtomicUsize,
    tail: std::sync::atomic::AtomicUsize,
}

impl<T: Send, const N: usize> CircularBuffer<T, N> {
    pub fn new() -> Self {
        CircularBuffer {
            données: unsafe { std::mem::MaybeUninit::uninit().assume_init() },
            head: std::sync::atomic::AtomicUsize::new(0),
            tail: std::sync::atomic::AtomicUsize::new(0),
        }
    }

    /// Écrit un élément dans le buffer.
    /// Retourne false si le buffer est plein.
    pub fn push(&self, val: T) -> bool {
        let tail = self.tail.load(std::sync::atomic::Ordering::Relaxed);
        let next_tail = (tail + 1) % N;

        if next_tail == self.head.load(std::sync::atomic::Ordering::Acquire) {
            return false; // Plein
        }

        unsafe {
            // SAFETY: tail est dans [0, N) par construction (modulo N)
            // Aucun autre thread n'écrit à cette position simultanément
            // (garanti par la logique du circular buffer)
            (self.données[tail].as_ptr() as *mut T).write(val);
        }

        self.tail.store(next_tail, std::sync::atomic::Ordering::Release);
        true
    }
}
```

### Wrapper RAII pour ressources FFI

```rust
extern "C" {
    fn ma_lib_ouvrir(chemin: *const std::os::raw::c_char) -> *mut std::os::raw::c_void;
    fn ma_lib_fermer(handle: *mut std::os::raw::c_void);
    fn ma_lib_lire(handle: *mut std::os::raw::c_void, buf: *mut u8, n: usize) -> isize;
}

/// Wrapper RAII — ferme automatiquement le handle à la fin
pub struct Handle(*mut std::os::raw::c_void);

impl Handle {
    pub fn ouvrir(chemin: &str) -> Option<Self> {
        let chemin_c = std::ffi::CString::new(chemin).ok()?;
        let handle = unsafe { ma_lib_ouvrir(chemin_c.as_ptr()) };
        if handle.is_null() { None } else { Some(Handle(handle)) }
    }

    pub fn lire(&mut self, buf: &mut [u8]) -> isize {
        unsafe {
            // SAFETY: self.0 est valide (garantie par le constructeur)
            // buf est valide pour la durée de l'appel
            ma_lib_lire(self.0, buf.as_mut_ptr(), buf.len())
        }
    }
}

impl Drop for Handle {
    fn drop(&mut self) {
        if !self.0.is_null() {
            unsafe {
                // SAFETY: self.0 valide, pas encore fermé
                ma_lib_fermer(self.0);
            }
        }
    }
}

// Handle ne doit pas être Send par défaut (le handle C peut ne pas être thread-safe)
// unsafe impl Send for Handle {} // Seulement si la lib garantit la thread safety
```

---

## 15. Exercices Pratiques

> [!tip] Exercices progressifs — du plus conceptuel au plus pratique

### Exercice 1 — Raw pointers arithmétique (Débutant)
Sans utiliser de références Rust, écrivez une fonction `somme_tableau(ptr: *const i32, len: usize) -> i64` qui :
- Parcourt un tableau via pointer arithmetic
- Retourne la somme de tous les éléments
- Inclut un commentaire `// SAFETY:` complet

### Exercice 2 — CString bidirectionnel (Intermédiaire)
Écrivez une bibliothèque qui expose une API C pour manipuler des chaînes :
- `fn string_creer(s: *const c_char) -> *mut StringRust` : alloue une String Rust
- `fn string_majuscules(s: *mut StringRust) -> *mut c_char` : retourne la version en majuscules
- `fn string_libérer(s: *mut StringRust)` : libère la mémoire
- Gérer correctement l'ownership et le null-terminator

### Exercice 3 — Wrapper RAII (Intermédiaire)
Wrappez la bibliothèque C stdio via FFI :
- `fopen`, `fclose`, `fread`, `fwrite`, `fflush`
- Struct `FichierC` avec `Drop` qui appelle `fclose`
- Méthodes `lire(&mut self, buf: &mut [u8]) -> std::io::Result<usize>`
- Méthode `écrire(&mut self, données: &[u8]) -> std::io::Result<()>`
- Vérifier que le fichier est bien fermé même en cas de panique

### Exercice 4 — Buffer lock-free (Avancé)
Complétez le `CircularBuffer<T, N>` de la section 14 :
- Implémenter `pop(&self) -> Option<T>`
- Implémenter `is_empty(&self)` et `is_full(&self)`
- Écrire des tests de stress : 1 producer thread + 1 consumer thread, 1 000 000 itérations
- Vérifier qu'aucun message n'est perdu ni dupliqué

### Exercice 5 — Bindings SQLite (Avancé)
Créez un wrapper sûr autour de libsqlite3 :
- Utiliser `bindgen` pour générer les bindings bruts
- Wrapper `Database` avec `open(path: &str) -> Result<Self>`
- Méthode `execute(sql: &str) -> Result<()>`
- Méthode `query<F: FnMut(&Row)>(sql: &str, callback: F) -> Result<()>`
- Struct `Row` permettant d'accéder aux colonnes par index ou nom
- Tous les unsafe isolés dans un module `ffi` interne

---

## Notes liées

- [[02 - Ownership et Borrowing]] — Ownership et lifetimes sont la base de la sécurité FFI
- [[08 - Concurrence et Smart Pointers]] — `Send`/`Sync` unsafe impl, `Box::into_raw`
- [[06 - Traits et Generiques]] — `Drop`, `Deref` utilisés dans les wrappers RAII
- [[10 - Macros et Metaprogrammation]] — `bindgen`/`cbindgen` sont utilisés via `build.rs`
- [[04 - Gestion des Erreurs en Rust]] — Gestion des erreurs FFI (codes retour C → Result)
- [[12 - GUI avec egui et eframe]] — eframe utilise FFI sous le capot (OpenGL, platform APIs)
- [[03 - Structs Enums et Pattern Matching]] — `repr(C)`, unions, pattern matching sur les codes retour
