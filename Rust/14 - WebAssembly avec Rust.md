# 14 - WebAssembly avec Rust

> [!info] Prérequis
> Maîtrise des [[06 - Traits et Generiques|Traits et Génériques Rust]], bases du DOM/JavaScript. Voir [[12 - GUI avec egui et eframe]] pour l'approche tout-Rust en WASM, et [[13 - Tauri Desktop Applications]] pour les apps desktop.

---

## 1. Qu'est-ce que WebAssembly ?

WebAssembly (WASM) est un format d'instruction binaire portable, conçu comme cible de compilation pour des langages de haut niveau (C/C++, Rust, Go...). Il s'exécute dans un sandbox sécurisé à des performances proches du natif.

### WASM vs JavaScript — Tableau comparatif

| Critère | WebAssembly | JavaScript |
|---|---|---|
| **Format** | Binaire compact (.wasm) | Texte (source ou minifié) |
| **Parsing** | Très rapide (format binaire) | Plus lent (parse + JIT) |
| **Performances** | Proches du natif (1.1x–2x vs C) | Variable (JIT) |
| **Typage** | Strict (i32, i64, f32, f64 + SIMD) | Dynamique |
| **Mémoire** | Mémoire linéaire explicite | GC automatique |
| **GC** | Pas natif (proposal en cours) | Intégré |
| **Threads** | Via SharedArrayBuffer + Atomics | Worker Threads |
| **DOM access** | Via JS bridge (indirect) | Direct |
| **Sandbox** | Oui (même origin policy) | Oui |
| **Portabilité** | Navigateurs, Node.js, Deno, WASI runtimes | Navigateurs, Node.js |
| **Cas d'usage** | Calcul intensif, gaming, audio/video, cryptographie | Logique UI, DOM, réseau |

> [!tip] WASM ne remplace pas JS
> WASM et JavaScript sont **complémentaires**. WASM excelle pour les calculs lourds, la manipulation de buffers, les algos DSP. JavaScript reste supérieur pour les interactions DOM, le réseau asynchrone, et la logique d'UI simple.

---

## 2. Toolchain Rust → WASM

### 2.1 Installation

```bash
# Target WASM pour Rust
rustup target add wasm32-unknown-unknown

# wasm-pack — l'outil principal (build, test, publish)
cargo install wasm-pack

# trunk — alternative pour les SPA egui/leptos (équivalent Vite pour WASM)
cargo install trunk

# wasm-bindgen CLI (souvent géré automatiquement par wasm-pack)
cargo install wasm-bindgen-cli

# Vérifier l'installation
wasm-pack --version
trunk --version
```

### 2.2 Cargo.toml minimal

```toml
[package]
name    = "mon-wasm"
version = "0.1.0"
edition = "2021"

# IMPORTANT : lib avec crate-type cdylib pour WASM
[lib]
crate-type = ["cdylib", "rlib"]
# cdylib = bibliothèque dynamique C (pour WASM)
# rlib   = bibliothèque Rust normale (pour les tests natifs)

[dependencies]
wasm-bindgen = "0.2"
web-sys = { version = "0.3", features = [
    # Activer uniquement les APIs DOM nécessaires (réduire la taille)
    "console",
    "Window",
    "Document",
    "Element",
    "HtmlElement",
    "HtmlCanvasElement",
    "CanvasRenderingContext2d",
    "EventTarget",
    "MouseEvent",
    "KeyboardEvent",
] }
js-sys = "0.3"
console_error_panic_hook = "0.1" # Afficher les panics Rust dans la console JS

[dev-dependencies]
wasm-bindgen-test = "0.3" # Tests WASM dans le navigateur ou Node.js

[profile.release]
opt-level = "s"   # Optimiser pour la taille (s) ou vitesse (3)
lto = true
```

---

## 3. wasm-bindgen — Interface Rust/JavaScript

`wasm-bindgen` génère automatiquement le "glue code" JavaScript qui permet d'appeler du Rust depuis JS et vice-versa.

### 3.1 Fonctions exposées à JavaScript

```rust
// src/lib.rs
use wasm_bindgen::prelude::*;

// Initialiser les panics lisibles (à appeler au début)
#[wasm_bindgen(start)]
pub fn init_panic_hook() {
    console_error_panic_hook::set_once();
}

// Fonction simple — types primitifs supportés nativement
#[wasm_bindgen]
pub fn additionner(a: i32, b: i32) -> i32 {
    a + b
}

// Fonction avec String
#[wasm_bindgen]
pub fn saluer(nom: &str) -> String {
    format!("Bonjour, {} !", nom)
}

// Fonction qui retourne un Result → throw JS en cas d'erreur
#[wasm_bindgen]
pub fn diviser(a: f64, b: f64) -> Result<f64, JsValue> {
    if b == 0.0 {
        Err(JsValue::from_str("Division par zéro"))
    } else {
        Ok(a / b)
    }
}

// Fonction avec Vec<T> — converti en Uint8Array/Float32Array etc.
#[wasm_bindgen]
pub fn trier_tableau(mut v: Vec<i32>) -> Vec<i32> {
    v.sort();
    v
}

// Log dans la console JS
#[wasm_bindgen]
pub fn log_exemple() {
    web_sys::console::log_1(&"Message depuis Rust !".into());
    web_sys::console::log_2(&"Valeur :".into(), &JsValue::from(42));
}
```

### 3.2 Structs exposées à JavaScript

```rust
use wasm_bindgen::prelude::*;

// Struct exportée — instanciable depuis JS comme new Point(x, y)
#[wasm_bindgen]
pub struct Point {
    x: f64,
    y: f64,
}

#[wasm_bindgen]
impl Point {
    // Constructeur JS
    #[wasm_bindgen(constructor)]
    pub fn new(x: f64, y: f64) -> Point {
        Point { x, y }
    }

    // Getters/Setters JS
    #[wasm_bindgen(getter)]
    pub fn x(&self) -> f64 { self.x }
    #[wasm_bindgen(getter)]
    pub fn y(&self) -> f64 { self.y }
    #[wasm_bindgen(setter)]
    pub fn set_x(&mut self, x: f64) { self.x = x; }

    // Méthodes
    pub fn distance(&self, autre: &Point) -> f64 {
        let dx = self.x - autre.x;
        let dy = self.y - autre.y;
        (dx * dx + dy * dy).sqrt()
    }

    pub fn deplacer(&mut self, dx: f64, dy: f64) {
        self.x += dx;
        self.y += dy;
    }

    // Méthode qui retourne une valeur JS complexe
    pub fn to_object(&self) -> JsValue {
        let obj = js_sys::Object::new();
        js_sys::Reflect::set(&obj, &"x".into(), &self.x.into()).unwrap();
        js_sys::Reflect::set(&obj, &"y".into(), &self.y.into()).unwrap();
        obj.into()
    }
}
```

### 3.3 Appeler des fonctions JavaScript depuis Rust

```rust
use wasm_bindgen::prelude::*;

// Déclarer une fonction JS externe
#[wasm_bindgen]
extern "C" {
    // Fonctions globales JS
    fn alert(s: &str);
    
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
    
    #[wasm_bindgen(js_namespace = console, js_name = log)]
    fn log_valeur(v: &JsValue);
    
    // Accéder à Math.random()
    #[wasm_bindgen(js_namespace = Math)]
    fn random() -> f64;
    
    // Appeler une fonction JS définie dans la page HTML
    fn ma_fonction_js(payload: &str);
}

#[wasm_bindgen]
pub fn exemple_appel_js() {
    let r = random();
    log(&format!("Nombre aléatoire : {}", r));
    alert("Hello depuis Rust !");
    ma_fonction_js("{\"type\": \"ping\"}");
}
```

---

## 4. web-sys — Accéder au DOM depuis Rust

`web-sys` fournit des bindings pour TOUTES les APIs Web (DOM, Canvas, WebGL, Fetch, WebSocket, etc.). Chaque type/méthode est derrière une feature Cargo.

```rust
use wasm_bindgen::prelude::*;
use web_sys::{window, Document, HtmlElement, HtmlCanvasElement, CanvasRenderingContext2d};

// Accéder au document
fn obtenir_document() -> Document {
    window()
        .expect("Pas de window")
        .document()
        .expect("Pas de document")
}

// Manipuler le DOM
#[wasm_bindgen]
pub fn modifier_dom() -> Result<(), JsValue> {
    let document = obtenir_document();
    
    // Obtenir un élément par ID
    let element = document
        .get_element_by_id("mon-div")
        .ok_or_else(|| JsValue::from_str("Élément introuvable"))?;
    
    let html_el: HtmlElement = element.dyn_into::<HtmlElement>()?;
    html_el.set_inner_text("Modifié par Rust !");
    html_el.style().set_property("color", "green")?;
    html_el.style().set_property("font-size", "24px")?;
    
    // Créer un nouvel élément
    let p = document.create_element("p")?;
    p.set_inner_html("<strong>Paragraphe</strong> créé par Rust");
    document.body().unwrap().append_child(&p)?;
    
    Ok(())
}

// Event listeners
use wasm_bindgen::closure::Closure;
use web_sys::EventTarget;

#[wasm_bindgen]
pub fn ajouter_listener() -> Result<(), JsValue> {
    let document = obtenir_document();
    let bouton = document.get_element_by_id("mon-bouton").unwrap();
    
    // La closure doit vivre aussi longtemps que le listener
    // forget() la fait vivre pour toujours (attention aux leaks !)
    let closure = Closure::<dyn Fn()>::new(|| {
        web_sys::console::log_1(&"Bouton cliqué !".into());
    });
    
    let bouton: EventTarget = bouton.dyn_into()?;
    bouton.add_event_listener_with_callback("click", closure.as_ref().unchecked_ref())?;
    closure.forget(); // Laisser vivre la closure côté JS
    
    Ok(())
}
```

### 4.1 Canvas 2D

```rust
use web_sys::{HtmlCanvasElement, CanvasRenderingContext2d};
use wasm_bindgen::prelude::*;

fn obtenir_contexte_canvas(id: &str) -> Result<CanvasRenderingContext2d, JsValue> {
    let document = web_sys::window().unwrap().document().unwrap();
    let canvas = document.get_element_by_id(id).unwrap();
    let canvas: HtmlCanvasElement = canvas.dyn_into::<HtmlCanvasElement>()?;
    let ctx = canvas
        .get_context("2d")?
        .unwrap()
        .dyn_into::<CanvasRenderingContext2d>()?;
    Ok(ctx)
}

#[wasm_bindgen]
pub fn dessiner_cercle(x: f64, y: f64, rayon: f64) -> Result<(), JsValue> {
    let ctx = obtenir_contexte_canvas("mon-canvas")?;
    
    ctx.begin_path();
    ctx.arc(x, y, rayon, 0.0, std::f64::consts::TAU)?; // TAU = 2π
    ctx.set_fill_style_str("rgba(100, 200, 100, 0.8)");
    ctx.fill();
    ctx.set_stroke_style_str("#00ff00");
    ctx.set_line_width(2.0);
    ctx.stroke();
    
    Ok(())
}

#[wasm_bindgen]
pub fn effacer_canvas(largeur: f64, hauteur: f64) -> Result<(), JsValue> {
    let ctx = obtenir_contexte_canvas("mon-canvas")?;
    ctx.clear_rect(0.0, 0.0, largeur, hauteur);
    Ok(())
}
```

---

## 5. js-sys — Types JavaScript en Rust

```rust
use js_sys::{Array, Object, Promise, Date, Math, Reflect};
use wasm_bindgen::prelude::*;
use wasm_bindgen_futures::JsFuture;

// Manipuler un Array JS
#[wasm_bindgen]
pub fn traiter_tableau_js(arr: &Array) -> Array {
    let resultat = Array::new();
    for i in 0..arr.length() {
        let val = arr.get(i);
        if let Some(n) = val.as_f64() {
            resultat.push(&JsValue::from(n * 2.0));
        }
    }
    resultat
}

// Construire un Object JS
#[wasm_bindgen]
pub fn creer_objet_js() -> Object {
    let obj = Object::new();
    Reflect::set(&obj, &"nom".into(), &"Rust".into()).unwrap();
    Reflect::set(&obj, &"version".into(), &"1.80".into()).unwrap();
    Reflect::set(&obj, &"rapide".into(), &JsValue::TRUE).unwrap();
    obj
}

// Utiliser Math JS
#[wasm_bindgen]
pub fn nombre_aleatoire_entre(min: f64, max: f64) -> f64 {
    Math::random() * (max - min) + min
}

// Attendre une Promise JS depuis Rust (avec async)
#[wasm_bindgen]
pub async fn fetch_json(url: String) -> Result<JsValue, JsValue> {
    let window = web_sys::window().unwrap();
    let promise = window.fetch_with_str(&url);
    let response = JsFuture::from(promise).await?;
    let response: web_sys::Response = response.dyn_into()?;
    JsFuture::from(response.json()?).await
}
```

---

## 6. wasm-pack — Build et Publication

```bash
# Build pour intégration avec bundler (webpack, vite, rollup)
wasm-pack build --target bundler --release

# Build pour utilisation directe dans le navigateur (ESM)
wasm-pack build --target web --release

# Build pour Node.js
wasm-pack build --target nodejs --release

# Build sans modules (vanilla JS, <script> tag)
wasm-pack build --target no-modules --release

# Structure générée dans pkg/
# pkg/
# ├── mon_wasm.js        — JS glue code
# ├── mon_wasm.d.ts      — TypeScript types
# ├── mon_wasm_bg.wasm   — Binaire WASM
# ├── mon_wasm_bg.wasm.d.ts
# └── package.json
```

### Intégration avec Vite

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import wasmPack from 'vite-plugin-wasm-pack'

export default defineConfig({
    plugins: [wasmPack('./')]
})
```

```typescript
// src/main.ts — Utilisation du module WASM
import init, { additionner, Point, saluer } from '../pkg/mon_wasm.js';

async function main() {
    // IMPORTANT : initialiser le module WASM avant toute utilisation
    await init();
    
    console.log(additionner(3, 4));        // 7
    console.log(saluer("Monde"));          // "Bonjour, Monde !"
    
    const p = new Point(10, 20);
    console.log(p.x, p.y);                // 10, 20
    p.deplacer(5, -5);
    console.log(p.x, p.y);                // 15, 15
    
    // Libérer la mémoire Rust (important pour les structs avec des ressources)
    p.free();
}

main().catch(console.error);
```

---

## 7. Performance WASM

### 7.1 Mémoire linéaire

```rust
// Partager de la mémoire entre Rust et JS sans copie
// Retourner un pointeur + longueur, JS accède directement à la mémoire WASM

#[wasm_bindgen]
pub struct Buffer {
    data: Vec<f32>,
}

#[wasm_bindgen]
impl Buffer {
    #[wasm_bindgen(constructor)]
    pub fn new(taille: usize) -> Self {
        Buffer { data: vec![0.0; taille] }
    }

    // Retourner un pointeur direct vers la mémoire (zero-copy !)
    pub fn as_ptr(&self) -> *const f32 {
        self.data.as_ptr()
    }

    pub fn len(&self) -> usize {
        self.data.len()
    }

    pub fn remplir_sinus(&mut self, frequence: f32, taux_echantillon: f32) {
        for (i, sample) in self.data.iter_mut().enumerate() {
            *sample = (2.0 * std::f32::consts::PI * frequence * i as f32 / taux_echantillon).sin();
        }
    }
}
```

```javascript
// Côté JS — accès zero-copy à la mémoire Rust
const buffer = new Buffer(44100);
buffer.remplir_sinus(440.0, 44100.0);

// Lire directement depuis la mémoire WASM
const wasmMemory = wasm.memory.buffer; // SharedArrayBuffer
const view = new Float32Array(wasmMemory, buffer.as_ptr(), buffer.len());
// `view` pointe directement sur la mémoire Rust — aucune copie !
console.log(view[0], view[1000]); // Lire les échantillons

buffer.free(); // Ne pas oublier !
```

### 7.2 SIMD WASM (parallélisme vectoriel)

```rust
// Cargo.toml — activer SIMD
[target.wasm32-unknown-unknown]
rustflags = ["-C", "target-feature=+simd128"]
```

```rust
// Utiliser les intrinsics SIMD WASM (nécessite nightly ou cfg spécifique)
#[cfg(target_arch = "wasm32")]
use std::arch::wasm32::*;

#[cfg(target_arch = "wasm32")]
pub fn additionner_vecteurs_simd(a: &[f32], b: &[f32]) -> Vec<f32> {
    assert_eq!(a.len(), b.len());
    let mut resultat = vec![0.0f32; a.len()];
    
    let chunks = a.len() / 4;
    for i in 0..chunks {
        let base = i * 4;
        // Charger 4 f32 en parallèle
        let va = unsafe { v128_load(a.as_ptr().add(base) as *const v128) };
        let vb = unsafe { v128_load(b.as_ptr().add(base) as *const v128) };
        // Additionner 4 f32 simultanément
        let vr = f32x4_add(va, vb);
        unsafe { v128_store(resultat.as_mut_ptr().add(base) as *mut v128, vr) };
    }
    // Traiter le reste
    for i in (chunks * 4)..a.len() {
        resultat[i] = a[i] + b[i];
    }
    resultat
}
```

### 7.3 Multi-threading avec Web Workers

```rust
// Nécessite : SharedArrayBuffer activé (COOP/COEP headers requis)
// wasm-bindgen-rayon ou wasm-threads
use wasm_bindgen::prelude::*;

// Configuration COOP/COEP dans vite.config.ts :
// server: { headers: { "Cross-Origin-Opener-Policy": "same-origin",
//                      "Cross-Origin-Embedder-Policy": "require-corp" } }
```

---

## 8. egui en WASM avec Trunk

C'est l'approche la plus simple pour une SPA tout-Rust.

```toml
# Cargo.toml
[dependencies]
eframe = { version = "0.29", features = ["web"] }
egui   = "0.29"
wasm-bindgen-futures = "0.4"
console_error_panic_hook = "0.1"
```

```rust
// src/main.rs
#[cfg(target_arch = "wasm32")]
fn main() {
    console_error_panic_hook::set_once();
    wasm_bindgen_futures::spawn_local(async {
        eframe::WebRunner::new()
            .start(
                "mon_canvas", // <canvas id="mon_canvas"> dans index.html
                eframe::WebOptions::default(),
                Box::new(|cc| Ok(Box::new(MonApp::new(cc)))),
            )
            .await
            .expect("Impossible de démarrer eframe web");
    });
}

#[cfg(not(target_arch = "wasm32"))]
fn main() -> eframe::Result<()> {
    eframe::run_native(
        "Mon App",
        eframe::NativeOptions::default(),
        Box::new(|cc| Ok(Box::new(MonApp::new(cc)))),
    )
}
```

```toml
# Trunk.toml
[build]
target    = "index.html"
dist      = "dist"
release   = false
public_url = "/"

[watch]
ignore = ["dist", "pkg"]
```

```bash
# Dev
trunk serve --open           # http://localhost:8080

# Production
trunk build --release        # → dist/

# Déployer sur GitHub Pages
# Copier dist/ dans gh-pages branch ou utiliser GitHub Actions
```

---

## 9. Frameworks Web Rust

### 9.1 Leptos — Signals et SSR

```toml
[dependencies]
leptos = { version = "0.7", features = ["csr"] } # csr = Client-Side Rendering
```

```rust
use leptos::prelude::*;

#[component]
fn Compteur() -> impl IntoView {
    // Signal réactif (équivalent useState React ou writable Svelte)
    let (compte, set_compte) = signal(0i32);

    view! {
        <div>
            <h1>"Compteur : " {compte}</h1>
            <button on:click=move |_| set_compte.update(|n| *n += 1)>"+1"</button>
            <button on:click=move |_| set_compte.update(|n| *n -= 1)>"-1"</button>
            <button on:click=move |_| set_compte.set(0)>"Reset"</button>
        </div>
    }
}

fn main() {
    leptos::mount::mount_to_body(Compteur);
}
```

### 9.2 Yew — Virtual DOM et Messages

```toml
[dependencies]
yew = { version = "0.21", features = ["csr"] }
```

```rust
use yew::prelude::*;

struct Compteur { compte: i32 }

enum Msg { Incrementer, Decrementer, Reset }

impl Component for Compteur {
    type Message = Msg;
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self { compte: 0 }
    }

    fn update(&mut self, _ctx: &Context<Self>, msg: Self::Message) -> bool {
        match msg {
            Msg::Incrementer  => { self.compte += 1; true }
            Msg::Decrementer  => { self.compte -= 1; true }
            Msg::Reset        => { self.compte = 0; true }
        }
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        let link = ctx.link();
        html! {
            <div>
                <h1>{ format!("Compteur : {}", self.compte) }</h1>
                <button onclick={link.callback(|_| Msg::Incrementer)}>{"+1"}</button>
                <button onclick={link.callback(|_| Msg::Decrementer)}>{"-1"}</button>
                <button onclick={link.callback(|_| Msg::Reset)}>{"Reset"}</button>
            </div>
        }
    }
}

fn main() {
    yew::Renderer::<Compteur>::new().render();
}
```

---

## 10. Patterns Architecturaux

### 10.1 WASM Module pur (calcul)

Le pattern le plus simple : WASM fait le calcul, JS fait l'UI.

```
JavaScript (UI, DOM, réseau)
         ↕ invoke() / return
WebAssembly (Rust — calcul pur)
```

### 10.2 WASM + JS Bridge

```
JS App                    WASM Module
─────────────────────     ──────────────────
UI Events      ──────►   Rust Logic
JS State       ◄──────   Results
fetch/WebSocket ──────►   Data Processing
                          Zero-copy buffers
```

### 10.3 Full Rust SPA (egui/Leptos)

```
WASM (Rust — UI + logique)
        │
   WebView (DOM minimal)
   Canvas / WebGL
```

---

## 11. Tests WASM

```rust
// Tests natifs (rapides, sans navigateur)
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_additionner() {
        assert_eq!(additionner(2, 3), 5);
    }
}

// Tests dans le navigateur (via wasm-pack test)
#[cfg(test)]
mod tests_wasm {
    use wasm_bindgen_test::*;
    use super::*;

    // Exécuter dans le navigateur headless
    wasm_bindgen_test_configure!(run_in_browser);

    #[wasm_bindgen_test]
    fn test_dans_navigateur() {
        let r = diviser(10.0, 2.0);
        assert!(r.is_ok());
        assert_eq!(r.unwrap(), 5.0);
    }

    #[wasm_bindgen_test]
    fn test_division_zero() {
        let r = diviser(1.0, 0.0);
        assert!(r.is_err());
    }
}
```

```bash
# Tests natifs
cargo test

# Tests dans navigateur headless Chrome
wasm-pack test --headless --chrome

# Tests dans Node.js
wasm-pack test --node
```

---

## 12. Projet Complet — Visualiseur de Tri en Canvas

```rust
use wasm_bindgen::prelude::*;
use web_sys::{HtmlCanvasElement, CanvasRenderingContext2d, window};

#[wasm_bindgen]
pub struct VisualiseurTri {
    tableau: Vec<u32>,
    canvas:  HtmlCanvasElement,
    ctx:     CanvasRenderingContext2d,
    largeur: f64,
    hauteur: f64,
}

#[wasm_bindgen]
impl VisualiseurTri {
    #[wasm_bindgen(constructor)]
    pub fn new(canvas_id: &str, taille: usize) -> Result<VisualiseurTri, JsValue> {
        let doc    = window().unwrap().document().unwrap();
        let canvas = doc.get_element_by_id(canvas_id).unwrap();
        let canvas: HtmlCanvasElement = canvas.dyn_into()?;
        let largeur = canvas.width() as f64;
        let hauteur = canvas.height() as f64;
        let ctx = canvas.get_context("2d")?.unwrap().dyn_into()?;

        // Générer un tableau mélangé
        let mut tableau: Vec<u32> = (1..=taille as u32).collect();
        // Algorithme de Fisher-Yates simplifié (sans rand — utiliser Math.random via js-sys)
        for i in (1..taille).rev() {
            let j = (js_sys::Math::random() * (i + 1) as f64) as usize;
            tableau.swap(i, j);
        }

        Ok(VisualiseurTri { tableau, canvas, ctx, largeur, hauteur })
    }

    pub fn dessiner(&self) {
        let n = self.tableau.len() as f64;
        let larg_barre = self.largeur / n;

        // Effacer
        self.ctx.set_fill_style_str("black");
        self.ctx.fill_rect(0.0, 0.0, self.largeur, self.hauteur);

        for (i, &val) in self.tableau.iter().enumerate() {
            let haut_barre = (val as f64 / n) * self.hauteur;
            let x = i as f64 * larg_barre;
            let y = self.hauteur - haut_barre;

            // Dégradé couleur selon hauteur
            let ratio = val as f64 / n;
            let r = (ratio * 255.0) as u8;
            let b = ((1.0 - ratio) * 255.0) as u8;
            self.ctx.set_fill_style_str(&format!("rgb({}, 50, {})", r, b));
            self.ctx.fill_rect(x + 0.5, y, larg_barre - 1.0, haut_barre);
        }
    }

    pub fn tri_bulles_etape(&mut self) -> bool {
        // Retourne true si terminé
        let n = self.tableau.len();
        let mut modifie = false;
        for i in 0..n - 1 {
            if self.tableau[i] > self.tableau[i + 1] {
                self.tableau.swap(i, i + 1);
                modifie = true;
            }
        }
        !modifie // Terminé si aucun échange
    }

    pub fn taille(&self) -> usize {
        self.tableau.len()
    }
}
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>Visualiseur de Tri Rust WASM</title></head>
<body style="background:black;display:flex;justify-content:center;">
<canvas id="canvas" width="800" height="400"></canvas>
<script type="module">
import init, { VisualiseurTri } from './pkg/mon_wasm.js';

await init();

const viz = new VisualiseurTri('canvas', 200);
viz.dessiner();

let termine = false;
function animer() {
    if (!termine) {
        termine = viz.tri_bulles_etape();
        viz.dessiner();
        requestAnimationFrame(animer);
    }
}
document.querySelector('canvas').addEventListener('click', () => {
    termine = false;
    animer();
});
</script>
</body>
</html>
```

---

## 13. Récapitulatif — Cheatsheet WASM

| Besoin | Outil |
|---|---|
| Exposer une fn Rust à JS | `#[wasm_bindgen]` sur la fn |
| Exposer une struct | `#[wasm_bindgen]` sur struct + impl |
| Appeler JS depuis Rust | `extern "C" { fn... }` + `#[wasm_bindgen]` |
| Accéder au DOM | `web-sys` crate (features activées) |
| Types JS (Array, Object) | `js-sys` crate |
| Build | `wasm-pack build --target web` |
| Dev SPA egui | `trunk serve` |
| Mémoire zero-copy | `as_ptr()` + `wasm.memory.buffer` en JS |
| Tests navigateur | `wasm-bindgen-test` + `wasm-pack test --headless --chrome` |
| SPA réactive Rust | Leptos (signals) ou Yew (composants) |
| Async dans WASM | `wasm-bindgen-futures::spawn_local()` |
| Panic visible | `console_error_panic_hook::set_once()` |

> [!warning] Taille du bundle
> Un binaire WASM Rust peut faire 500KB–2MB. Utiliser `opt-level = "s"` + `lto = true` + `wasm-opt -Oz` (binaryen). La majorité de la taille vient du code Rust lui-même et des dépendances embarquées.

> [!tip] Ressources
> - [wasm-bindgen guide](https://rustwasm.github.io/wasm-bindgen/) — documentation officielle
> - [Rust and WebAssembly book](https://rustwasm.github.io/docs/book/) — tutoriel complet
> - [web-sys docs](https://docs.rs/web-sys) — toutes les APIs Web disponibles
> - [Leptos](https://leptos.dev/) — framework SPA Rust moderne

---

## Notes liées

### Rust — prérequis
- [[06 - Traits et Generiques]] — traits essentiels pour wasm-bindgen (`Into`, `From`, `AsRef`)
- [[09 - Async Await et Tokio]] — async/await côté WASM avec `wasm-bindgen-futures`
- [[02 - Ownership et Borrowing]] — lifetimes et borrowing avec JsValue et les types JS
- [[10 - Macros et Metaprogrammation]] — `#[wasm_bindgen]` attribute macro en détail

### Applications WASM
- [[12 - GUI avec egui et eframe]] — egui compilé en WASM : app desktop et web depuis le même code
- [[13 - Tauri Desktop Applications]] — alternative : desktop natif sans WASM
- [[15 - Web Frameworks Axum et Actix]] — backend Rust pour servir le WASM au navigateur

### JavaScript et Web
- [[01 - Introduction a JavaScript]] — JavaScript : l'hôte du WASM, interop JS↔Rust
- [[04 - JavaScript Moderne ES6+]] — modules ES6, import du bundle WASM
- [[03 - JavaScript Asynchrone]] — Promises JS ↔ async Rust via `wasm-bindgen-futures`
- [[01 - Introduction a React]] — React + WASM : composants React appelant du code Rust
- [[05 - WebSockets]] — WebSockets depuis Rust/WASM avec `web-sys`

### Déploiement
- [[04 - CI-CD avec GitHub Actions]] — pipeline `wasm-pack build` + deploy GitHub Pages
- [[01 - Docker]] — Dockerfile pour builder le WASM en CI
