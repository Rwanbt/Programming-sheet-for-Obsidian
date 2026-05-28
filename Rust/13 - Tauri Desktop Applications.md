# 13 - Tauri Desktop Applications

> [!info] Prérequis
> Ce cours suppose la maîtrise de [[06 - Traits et Generiques|Rust intermédiaire]], de l'[[09 - Async Await et Tokio|async/await]], et des bases d'un framework frontend (Svelte, React ou Vue). Voir [[12 - GUI avec egui et eframe]] pour l'alternative tout-Rust.

---

## 1. Tauri — Vue d'ensemble

Tauri est un framework open-source permettant de créer des applications desktop cross-platform (Windows, macOS, Linux) avec un **backend Rust** et un **frontend web** (HTML/CSS/JS, React, Vue, Svelte, etc.).

### 1.1 Tauri vs Electron — Tableau comparatif

| Critère | Tauri 2.0 | Electron |
|---|---|---|
| **Runtime** | WebView système (WKWebView, WebView2, WebKitGTK) | Chromium embarqué |
| **Taille bundle** | ~8–12 MB | ~120–200 MB |
| **RAM (idle)** | ~30–60 MB | ~150–300 MB |
| **Performances** | Native (Rust backend) | Node.js (JavaScript) |
| **Sécurité** | Permissions granulaires, sandbox, CSP strict | Moins restrictif par défaut |
| **Multi-fenêtres** | Oui, natif Tauri 2.0 | Oui |
| **Mobile** | iOS + Android (Tauri 2.0) | Non |
| **Langages backend** | Rust uniquement | Node.js (JavaScript/TypeScript) |
| **Courbe apprentissage** | Rust requis | Moins abrupte (JS full-stack) |
| **Ecosystème plugins** | En croissance | Très mature |
| **Licence** | MIT/Apache-2.0 | MIT |

### 1.2 Tauri 2.0 vs 1.x

| Aspect | Tauri 1.x | Tauri 2.0 |
|---|---|---|
| **Mobile** | Non | iOS + Android natif |
| **Permissions** | `allowlist` dans tauri.conf.json | Système de permissions granulaires par plugin |
| **Plugins** | `tauri-plugin-*` v1 | Réécriture complète v2 |
| **IPC** | `invoke()` + `listen()` | Inchangé, plus rapide |
| **Multi-window** | Partiel | Viewports natifs |
| **Breaking changes** | — | Majeurs vs v1 |

> [!warning] Versions
> Ce cours couvre **Tauri 2.0** (stable depuis octobre 2024). La syntaxe des permissions est radicalement différente de Tauri 1.x.

---

## 2. Architecture Tauri

```
┌─────────────────────────────────────────────────────┐
│                  Application Tauri                  │
│                                                     │
│  ┌──────────────────┐      ┌─────────────────────┐  │
│  │   Frontend Web   │ IPC  │   Backend Rust      │  │
│  │                  │◄────►│                     │  │
│  │  React/Svelte/Vue│      │  Tauri Runtime      │  │
│  │  HTML + CSS + JS │      │  (Rust + OS APIs)   │  │
│  │                  │      │                     │  │
│  │  WebView natif   │      │  src-tauri/src/     │  │
│  │  (WKWebView,     │      │  main.rs            │  │
│  │   WebView2,      │      │  lib.rs             │  │
│  │   WebKitGTK)     │      │  commands.rs        │  │
│  └──────────────────┘      └─────────────────────┘  │
│                                                     │
│              Capacités OS natives                   │
│    Fichiers · Notifications · Tray · Updater        │
└─────────────────────────────────────────────────────┘
```

### IPC (Inter-Process Communication)

L'IPC Tauri repose sur deux mécanismes :
- **Commands** : appels frontend → backend (request/response, comme un RPC)
- **Events** : messages backend → frontend (ou bidirectionnels)

---

## 3. Setup et Structure du Projet

### 3.1 Créer un nouveau projet

```bash
# Prérequis
rustup update
cargo install create-tauri-app

# Créer le projet
npm create tauri-app@latest
# Répondre aux questions :
# > Nom du projet : mon-app
# > Framework : Svelte (ou React, Vue, Vanilla...)
# > Gestionnaire de paquets : npm / pnpm / yarn
# > Langage frontend : TypeScript

# Ou avec les choix inline
npm create tauri-app@latest mon-app -- --template svelte-ts

cd mon-app
npm install
npm run tauri dev  # Lancer en mode développement
```

### 3.2 Structure du projet

```
mon-app/
├── src/                    # Frontend (Svelte/React/etc.)
│   ├── App.svelte
│   ├── main.ts
│   └── app.css
├── src-tauri/              # Backend Rust
│   ├── Cargo.toml
│   ├── build.rs
│   ├── icons/              # Icônes de l'app (plusieurs tailles)
│   ├── capabilities/       # Permissions Tauri 2.0
│   │   └── default.json
│   └── src/
│       ├── main.rs         # Point d'entrée (très minimal)
│       ├── lib.rs          # Logique principale + setup
│       └── commands.rs     # Commandes Tauri
├── package.json
├── vite.config.ts
└── tauri.conf.json         # Configuration principale
```

### 3.3 main.rs et lib.rs

```rust
// src-tauri/src/main.rs — Aussi simple que possible
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

fn main() {
    mon_app_lib::run()
}
```

```rust
// src-tauri/src/lib.rs — La logique principale
use tauri::Manager;

mod commands;
use commands::*;

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        // Plugins
        .plugin(tauri_plugin_fs::init())
        .plugin(tauri_plugin_dialog::init())
        .plugin(tauri_plugin_notification::init())
        .plugin(tauri_plugin_store::Builder::default().build())
        // State partagé
        .manage(AppState::default())
        // Commandes
        .invoke_handler(tauri::generate_handler![
            obtenir_salutation,
            sauvegarder_fichier,
            charger_config,
        ])
        // Setup initial
        .setup(|app| {
            // Accès au handle de l'app au démarrage
            let handle = app.handle();
            // Initialiser la base de données, charger la config, etc.
            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("Erreur lors du lancement de Tauri");
}

// État partagé entre commandes
#[derive(Default)]
struct AppState {
    compteur: std::sync::Mutex<i32>,
}
```

---

## 4. Configuration — tauri.conf.json

```json
{
  "$schema": "https://schema.tauri.app/config/2",
  "productName": "Mon Application",
  "version": "1.0.0",
  "identifier": "com.monentreprise.mon-app",
  "build": {
    "frontendDist": "../dist",
    "devUrl": "http://localhost:5173",
    "beforeDevCommand": "npm run dev",
    "beforeBuildCommand": "npm run build"
  },
  "app": {
    "windows": [
      {
        "title": "Mon Application",
        "width": 1200,
        "height": 800,
        "minWidth": 800,
        "minHeight": 600,
        "resizable": true,
        "center": true,
        "decorations": true,
        "transparent": false,
        "fullscreen": false
      }
    ],
    "security": {
      "csp": "default-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:;"
    }
  },
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "windows": {
      "certificateThumbprint": null,
      "digestAlgorithm": "sha256",
      "timestampUrl": ""
    },
    "macOS": {
      "entitlements": null,
      "exceptionDomain": "",
      "signingIdentity": null,
      "providerShortName": null
    }
  }
}
```

### Permissions Tauri 2.0 — capabilities/default.json

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "default",
  "description": "Permissions par défaut",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:allow-read-text-file",
    "fs:allow-write-text-file",
    "fs:allow-mkdir",
    "dialog:allow-open",
    "dialog:allow-save",
    "notification:allow-notify",
    "shell:allow-open"
  ]
}
```

---

## 5. Commands Rust

Les commands sont des fonctions Rust appelables depuis le JavaScript via `invoke()`.

### 5.1 Commandes simples

```rust
// src-tauri/src/commands.rs

// Commande synchrone basique
#[tauri::command]
fn obtenir_salutation(nom: String) -> String {
    format!("Bonjour, {} !", nom)
}

// Commande avec multiple paramètres
#[tauri::command]
fn calculer(a: f64, b: f64, operation: String) -> Result<f64, String> {
    match operation.as_str() {
        "add" => Ok(a + b),
        "sub" => Ok(a - b),
        "mul" => Ok(a * b),
        "div" => {
            if b == 0.0 { Err("Division par zéro".to_string()) }
            else { Ok(a / b) }
        }
        _ => Err(format!("Opération inconnue : {}", operation)),
    }
}
```

### 5.2 Commandes async

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct ArticleMeteo {
    ville: String,
    temperature: f32,
    description: String,
}

// Commande async — ne bloque pas le thread UI
#[tauri::command]
async fn obtenir_meteo(ville: String) -> Result<ArticleMeteo, String> {
    // reqwest ou ureq pour les requêtes HTTP
    let url = format!(
        "https://api.openweathermap.org/data/2.5/weather?q={}&appid=CLE&units=metric",
        ville
    );

    let response = reqwest::get(&url)
        .await
        .map_err(|e| e.to_string())?;

    if !response.status().is_success() {
        return Err(format!("Erreur API : {}", response.status()));
    }

    // Parsing JSON simplifié
    let data: serde_json::Value = response.json().await.map_err(|e| e.to_string())?;
    
    Ok(ArticleMeteo {
        ville:       data["name"].as_str().unwrap_or("?").to_string(),
        temperature: data["main"]["temp"].as_f64().unwrap_or(0.0) as f32,
        description: data["weather"][0]["description"].as_str().unwrap_or("?").to_string(),
    })
}
```

### 5.3 Accès au State et au Handle

```rust
use tauri::State;

// Accéder à l'état partagé
#[tauri::command]
fn incrementer(state: State<AppState>) -> i32 {
    let mut compteur = state.compteur.lock().unwrap();
    *compteur += 1;
    *compteur
}

// Accéder à l'AppHandle (pour émettre des events, gérer les fenêtres...)
#[tauri::command]
async fn tache_longue(app: tauri::AppHandle) -> Result<(), String> {
    for i in 0..100 {
        tokio::time::sleep(tokio::time::Duration::from_millis(50)).await;
        // Émettre un event de progression
        app.emit("progression", serde_json::json!({ "valeur": i }))
            .map_err(|e| e.to_string())?;
    }
    Ok(())
}

// Accéder à la fenêtre courante
#[tauri::command]
fn minimiser_fenetre(window: tauri::Window) {
    window.minimize().unwrap();
}
```

### 5.4 Gestion d'erreurs typée

```rust
use serde::Serialize;
use thiserror::Error;

// Erreur custom sérialisable (envoyée au frontend comme JSON)
#[derive(Debug, Error, Serialize)]
pub enum ErreurApp {
    #[error("Fichier introuvable : {0}")]
    FichierIntrouvable(String),
    
    #[error("Permission refusée")]
    PermissionRefusee,
    
    #[error("Erreur réseau : {0}")]
    ErreurReseau(String),
    
    #[error("Données invalides : {0}")]
    DonneesInvalides(String),
}

#[tauri::command]
async fn lire_fichier_config(chemin: String) -> Result<String, ErreurApp> {
    if chemin.is_empty() {
        return Err(ErreurApp::DonneesInvalides("Chemin vide".to_string()));
    }
    
    tokio::fs::read_to_string(&chemin)
        .await
        .map_err(|e| match e.kind() {
            std::io::ErrorKind::NotFound => ErreurApp::FichierIntrouvable(chemin),
            std::io::ErrorKind::PermissionDenied => ErreurApp::PermissionRefusee,
            _ => ErreurApp::ErreurReseau(e.to_string()),
        })
}
```

---

## 6. Appel des Commands depuis JavaScript/TypeScript

```typescript
// src/lib/tauri.ts
import { invoke } from '@tauri-apps/api/core';

// Appel simple
const salutation = await invoke<string>('obtenir_salutation', { nom: 'Monde' });

// Appel avec gestion d'erreur
try {
    const resultat = await invoke<number>('calculer', {
        a: 10.0,
        b: 3.0,
        operation: 'div'
    });
    console.log('Résultat :', resultat);
} catch (erreur) {
    // erreur est l'objet Rust sérialisé en JSON
    console.error('Erreur :', erreur);
}

// Types TypeScript pour les erreurs Rust custom
interface ErreurApp {
    FichierIntrouvable?: string;
    PermissionRefusee?: null;
    ErreurReseau?: string;
    DonneesInvalides?: string;
}

async function chargerConfig(chemin: string): Promise<string> {
    try {
        return await invoke<string>('lire_fichier_config', { chemin });
    } catch (e) {
        const erreur = e as ErreurApp;
        if (erreur.FichierIntrouvable) {
            throw new Error(`Fichier introuvable : ${erreur.FichierIntrouvable}`);
        }
        throw new Error('Erreur inconnue');
    }
}
```

---

## 7. Events (Événements)

### 7.1 Émettre et écouter côté Rust

```rust
// Émettre depuis Rust vers TOUS les frontend
app.emit("mise-a-jour", serde_json::json!({
    "version": "2.1.0",
    "notes": "Nouvelles fonctionnalités"
})).unwrap();

// Émettre vers une fenêtre spécifique
let fenetre = app.get_webview_window("main").unwrap();
fenetre.emit("notification", "Message pour cette fenêtre").unwrap();

// Écouter un event frontend côté Rust
app.listen("action-utilisateur", |event| {
    let payload: String = event.payload().to_string();
    println!("Action reçue : {}", payload);
});
```

### 7.2 Émettre et écouter côté JavaScript

```typescript
import { emit, listen, once } from '@tauri-apps/api/event';

// Écouter un event Rust → JS (abonnement permanent)
const deabonnement = await listen<{ version: string }>('mise-a-jour', (event) => {
    console.log('Mise à jour disponible :', event.payload.version);
    console.log('Fenêtre source :', event.windowLabel);
});

// Écouter une seule fois
await once('initialisation-terminee', () => {
    console.log('App prête !');
});

// Émettre depuis JS vers Rust (ou d'autres fenêtres)
await emit('action-utilisateur', { type: 'clic', element: 'bouton-sauvegarder' });

// Se désabonner (important pour éviter les memory leaks)
onDestroy(() => {
    deabonnement(); // Appeler la fonction retournée par listen()
});
```

### 7.3 Pattern Progress avec Events

```typescript
// Svelte — écouter la progression d'une tâche longue
<script lang="ts">
import { invoke } from '@tauri-apps/api/core';
import { listen } from '@tauri-apps/api/event';
import { onMount, onDestroy } from 'svelte';

let progression = 0;
let deabonnement: (() => void) | null = null;

onMount(async () => {
    deabonnement = await listen<{ valeur: number }>('progression', (e) => {
        progression = e.payload.valeur;
    });
});

onDestroy(() => deabonnement?.());

async function lancerTache() {
    progression = 0;
    await invoke('tache_longue');
}
</script>

<button on:click={lancerTache}>Lancer</button>
<progress value={progression} max={100}></progress>
```

---

## 8. State Management Rust

```rust
// Types d'état recommandés
use std::sync::{Arc, Mutex, RwLock};

// Pour les données simples et rares
struct Config {
    api_key: Mutex<String>,
    debug_mode: std::sync::atomic::AtomicBool,
}

// Pour les données en lecture intensive
struct Cache {
    donnees: RwLock<std::collections::HashMap<String, String>>,
}

// Enregistrement dans le setup
.manage(Arc::new(Config {
    api_key: Mutex::new(String::new()),
    debug_mode: std::sync::atomic::AtomicBool::new(false),
}))
.manage(Arc::new(Cache {
    donnees: RwLock::new(std::collections::HashMap::new()),
}))

// Utilisation dans les commandes
#[tauri::command]
fn definir_cle_api(cle: String, config: State<Arc<Config>>) {
    *config.api_key.lock().unwrap() = cle;
}

#[tauri::command]
fn lire_depuis_cache(cle: String, cache: State<Arc<Cache>>) -> Option<String> {
    cache.donnees.read().unwrap().get(&cle).cloned()
}
```

---

## 9. Plugins Tauri Officiels

### 9.1 Filesystem (tauri-plugin-fs)

```toml
# Cargo.toml
[dependencies]
tauri-plugin-fs = "2"
```

```typescript
import { readTextFile, writeTextFile, mkdir, BaseDirectory, readDir } from '@tauri-apps/plugin-fs';

// Lire un fichier
const contenu = await readTextFile('config.json', {
    baseDir: BaseDirectory.AppConfig // $APPCONFIG/config.json
});

// Écrire un fichier
await writeTextFile('notes.txt', 'Mon contenu', {
    baseDir: BaseDirectory.Document
});

// Créer un dossier
await mkdir('sous-dossier', {
    baseDir: BaseDirectory.AppData,
    recursive: true
});

// Lister un dossier
const fichiers = await readDir('', { baseDir: BaseDirectory.AppData });
for (const f of fichiers) {
    console.log(f.name, f.isDirectory ? '(dossier)' : '');
}
```

### 9.2 Dialog (tauri-plugin-dialog)

```typescript
import { open, save, message, ask, confirm } from '@tauri-apps/plugin-dialog';

// Dialogue d'ouverture de fichier
const fichier = await open({
    multiple: false,
    filters: [
        { name: 'JSON', extensions: ['json'] },
        { name: 'Tous les fichiers', extensions: ['*'] }
    ],
    defaultPath: '/home/user/documents'
});
if (fichier) {
    console.log('Chemin sélectionné :', fichier);
}

// Dialogue de sauvegarde
const chemin = await save({
    filters: [{ name: 'Texte', extensions: ['txt', 'md'] }],
    defaultPath: 'mon-fichier.txt'
});

// Boîtes de dialogue système
await message('Opération réussie', { title: 'Succès', kind: 'info' });
const confirme = await confirm('Voulez-vous vraiment supprimer ?', {
    title: 'Confirmer la suppression',
    kind: 'warning'
});
const réponse = await ask('Entrez votre nom :', { title: 'Identification' });
```

### 9.3 Store (tauri-plugin-store) — Persistence key-value

```typescript
import { load } from '@tauri-apps/plugin-store';

// Ouvrir/créer un store
const store = await load('preferences.json', { autoSave: true });

// Lire
const theme = await store.get<string>('theme') ?? 'sombre';
const volume = await store.get<number>('volume') ?? 0.8;

// Écrire
await store.set('theme', 'clair');
await store.set('dernier-projet', '/chemin/vers/projet.mpj');
await store.save(); // Forcer la sauvegarde immédiate (si autoSave = false)

// Supprimer
await store.delete('cle-temporaire');

// Réinitialiser
await store.clear();
```

### 9.4 Notification (tauri-plugin-notification)

```typescript
import { sendNotification, isPermissionGranted, requestPermission } from '@tauri-apps/plugin-notification';

async function notifier(titre: string, corps: string) {
    let permis = await isPermissionGranted();
    if (!permis) {
        const permission = await requestPermission();
        permis = permission === 'granted';
    }
    if (permis) {
        sendNotification({ title: titre, body: corps, icon: 'icons/icon.png' });
    }
}
```

### 9.5 Shell (tauri-plugin-shell) — Ouvrir des liens/fichiers

```typescript
import { open } from '@tauri-apps/plugin-shell';

// Ouvrir une URL dans le navigateur par défaut
await open('https://tauri.app');

// Ouvrir un fichier avec son application par défaut
await open('/chemin/vers/document.pdf');

// Ouvrir le gestionnaire de fichiers à un dossier
await open('/chemin/vers/dossier');
```

---

## 10. System Tray

```rust
// src-tauri/src/lib.rs
use tauri::tray::{TrayIconBuilder, TrayIconEvent, MouseButton, MouseButtonState};
use tauri::menu::{Menu, MenuItem};

.setup(|app| {
    let quitter = MenuItem::with_id(app, "quitter", "Quitter", true, None::<&str>)?;
    let afficher = MenuItem::with_id(app, "afficher", "Afficher", true, None::<&str>)?;
    let menu = Menu::with_items(app, &[&afficher, &quitter])?;

    TrayIconBuilder::new()
        .icon(app.default_window_icon().unwrap().clone())
        .menu(&menu)
        .menu_on_left_click(false)
        .on_menu_event(|app, event| match event.id.as_ref() {
            "quitter" => app.exit(0),
            "afficher" => {
                if let Some(w) = app.get_webview_window("main") {
                    w.show().unwrap();
                    w.set_focus().unwrap();
                }
            }
            _ => {}
        })
        .on_tray_icon_event(|tray, event| {
            if let TrayIconEvent::Click {
                button: MouseButton::Left,
                button_state: MouseButtonState::Up,
                ..
            } = event {
                let app = tray.app_handle();
                if let Some(w) = app.get_webview_window("main") {
                    if w.is_visible().unwrap_or(false) {
                        w.hide().unwrap();
                    } else {
                        w.show().unwrap();
                        w.set_focus().unwrap();
                    }
                }
            }
        })
        .build(app)?;
    Ok(())
})
```

---

## 11. Auto-Updater

```toml
# Cargo.toml
[dependencies]
tauri-plugin-updater = "2"
```

```typescript
// src/updater.ts
import { check } from '@tauri-apps/plugin-updater';
import { ask } from '@tauri-apps/plugin-dialog';

export async function verifierMisesAJour() {
    try {
        const update = await check();
        if (!update?.available) return;

        const souhaite = await ask(
            `Version ${update.version} disponible.\n\n${update.body}\n\nInstaller maintenant ?`,
            { title: 'Mise à jour disponible', kind: 'info' }
        );

        if (souhaite) {
            let telechargé = 0;
            let taille = 0;
            await update.downloadAndInstall((event) => {
                switch (event.event) {
                    case 'Started':
                        taille = event.data.contentLength ?? 0;
                        console.log(`Téléchargement : ${taille} octets`);
                        break;
                    case 'Progress':
                        telechargé += event.data.chunkLength;
                        console.log(`${Math.round(telechargé / taille * 100)}%`);
                        break;
                    case 'Finished':
                        console.log('Installation terminée, redémarrage...');
                        break;
                }
            });
        }
    } catch (e) {
        console.error('Erreur vérification mise à jour :', e);
    }
}
```

```json
// tauri.conf.json — configuration de l'updater
{
  "plugins": {
    "updater": {
      "active": true,
      "endpoints": ["https://releases.monapp.com/update/{{target}}/{{arch}}/{{current_version}}"],
      "dialog": false,
      "pubkey": "CLEF_PUBLIQUE_TAURI_UPDATER"
    }
  }
}
```

---

## 12. Sécurité

```json
// tauri.conf.json — CSP stricte
{
  "app": {
    "security": {
      "csp": "default-src 'self'; connect-src 'self' https://api.monservice.com; img-src 'self' data: https:; style-src 'self' 'unsafe-inline'"
    }
  }
}
```

> [!warning] Règles de sécurité Tauri
> 1. **Ne jamais exposer de commandes Rust sans validation** des paramètres reçus
> 2. **CSP stricte** — ne jamais utiliser `'unsafe-eval'`
> 3. **Permissions minimales** — accorder uniquement ce qui est nécessaire
> 4. **Valider côté Rust** tout input venant du frontend — le frontend est un vecteur d'attaque potentiel (XSS → RCE via commandes)
> 5. **Ne pas exposer de secrets** (clés API, tokens) via les commandes sans contrôle d'accès

---

## 13. Build et Distribution

```bash
# Build pour la plateforme courante
npm run tauri build

# Fichiers générés dans src-tauri/target/release/bundle/
# Windows : .msi + .exe (NSIS installer)
# macOS   : .app + .dmg
# Linux   : .deb + .AppImage + .rpm
```

### Dockerfile CI/CD (build Linux)

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    curl build-essential \
    libwebkit2gtk-4.1-dev libssl-dev libayatana-appindicator3-dev librsvg2-dev \
    nodejs npm

RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
ENV PATH="/root/.cargo/bin:$PATH"

WORKDIR /app
COPY . .
RUN npm install
RUN npm run tauri build
```

### GitHub Actions

```yaml
# .github/workflows/release.yml
name: Release Tauri
on:
  push:
    tags: ['v*']

jobs:
  release:
    strategy:
      matrix:
        os: [ubuntu-22.04, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - uses: actions/setup-node@v4
        with: { node-version: 20 }

      - name: Dépendances Linux
        if: matrix.os == 'ubuntu-22.04'
        run: |
          sudo apt-get update
          sudo apt-get install -y libwebkit2gtk-4.1-dev libayatana-appindicator3-dev librsvg2-dev

      - run: npm install
      - uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          TAURI_SIGNING_PRIVATE_KEY: ${{ secrets.TAURI_PRIVATE_KEY }}
        with:
          tagName: v__VERSION__
          releaseName: 'Mon App v__VERSION__'
          releaseBody: 'Voir CHANGELOG.md pour les nouveautés.'
          releaseDraft: true
```

---

## 14. Mobile — Tauri 2.0 (iOS + Android)

```bash
# Prérequis Android
rustup target add aarch64-linux-android armv7-linux-androideabi x86_64-linux-android i686-linux-android
cargo install tauri-cli --version "^2"

# Prérequis iOS (macOS uniquement)
rustup target add aarch64-apple-ios x86_64-apple-ios aarch64-apple-ios-sim

# Initialiser les targets mobiles
cargo tauri android init
cargo tauri ios init

# Développement
cargo tauri android dev
cargo tauri ios dev

# Build
cargo tauri android build
cargo tauri ios build
```

```rust
// Différences API mobile
// Les plugins fs, dialog, notification ont des comportements légèrement différents
// Vérifier les capabilities mobiles séparément

// capabilities/mobile.json
{
  "$schema": "../gen/schemas/mobile-schema.json",
  "identifier": "mobile-default",
  "platforms": ["android", "ios"],
  "windows": ["main"],
  "permissions": [
    "core:default",
    "notification:allow-notify"
    // fs et dialog ont des chemins différents sur mobile
  ]
}
```

---

## 15. Projet Complet — App Notes Cross-Platform

### Backend Rust avec SQLite

```toml
# src-tauri/Cargo.toml
[dependencies]
tauri = { version = "2", features = [] }
rusqlite = { version = "0.31", features = ["bundled"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
// src-tauri/src/db.rs
use rusqlite::{Connection, Result, params};
use serde::{Deserialize, Serialize};
use std::sync::Mutex;

#[derive(Serialize, Deserialize, Clone)]
pub struct Note {
    pub id:      i64,
    pub titre:   String,
    pub contenu: String,
    pub cree_le: String,
}

pub struct BddState(pub Mutex<Connection>);

impl BddState {
    pub fn new(chemin: &str) -> Result<Self> {
        let conn = Connection::open(chemin)?;
        conn.execute_batch("
            CREATE TABLE IF NOT EXISTS notes (
                id      INTEGER PRIMARY KEY AUTOINCREMENT,
                titre   TEXT NOT NULL,
                contenu TEXT NOT NULL DEFAULT '',
                cree_le TEXT NOT NULL DEFAULT (datetime('now'))
            );
        ")?;
        Ok(Self(Mutex::new(conn)))
    }
}

// Commands
#[tauri::command]
pub fn lister_notes(state: tauri::State<BddState>) -> Result<Vec<Note>, String> {
    let conn = state.0.lock().map_err(|e| e.to_string())?;
    let mut stmt = conn.prepare(
        "SELECT id, titre, contenu, cree_le FROM notes ORDER BY cree_le DESC"
    ).map_err(|e| e.to_string())?;

    let notes: Result<Vec<Note>, _> = stmt.query_map([], |row| {
        Ok(Note {
            id:      row.get(0)?,
            titre:   row.get(1)?,
            contenu: row.get(2)?,
            cree_le: row.get(3)?,
        })
    }).map_err(|e| e.to_string())?.collect();
    
    notes.map_err(|e| e.to_string())
}

#[tauri::command]
pub fn creer_note(titre: String, contenu: String, state: tauri::State<BddState>) -> Result<i64, String> {
    let conn = state.0.lock().map_err(|e| e.to_string())?;
    conn.execute(
        "INSERT INTO notes (titre, contenu) VALUES (?1, ?2)",
        params![titre, contenu],
    ).map_err(|e| e.to_string())?;
    Ok(conn.last_insert_rowid())
}

#[tauri::command]
pub fn supprimer_note(id: i64, state: tauri::State<BddState>) -> Result<(), String> {
    let conn = state.0.lock().map_err(|e| e.to_string())?;
    conn.execute("DELETE FROM notes WHERE id = ?1", params![id])
        .map_err(|e| e.to_string())?;
    Ok(())
}
```

### Frontend Svelte

```svelte
<!-- src/App.svelte -->
<script lang="ts">
import { invoke } from '@tauri-apps/api/core';
import { onMount } from 'svelte';

interface Note { id: number; titre: string; contenu: string; cree_le: string; }

let notes: Note[] = [];
let selection: Note | null = null;
let titre = '';
let contenu = '';

onMount(chargerNotes);

async function chargerNotes() {
    notes = await invoke<Note[]>('lister_notes');
}

async function creerNote() {
    if (!titre.trim()) return;
    const id = await invoke<number>('creer_note', { titre, contenu });
    titre = ''; contenu = '';
    await chargerNotes();
    selection = notes.find(n => n.id === id) ?? null;
}

async function supprimerNote(id: number) {
    await invoke('supprimer_note', { id });
    if (selection?.id === id) selection = null;
    await chargerNotes();
}
</script>

<div class="app">
    <aside>
        <button on:click={() => { selection = null; titre = ''; contenu = ''; }}>+ Nouvelle note</button>
        {#each notes as note}
            <div class="note-item" class:actif={selection?.id === note.id}
                on:click={() => selection = note}>
                <strong>{note.titre}</strong>
                <small>{note.cree_le}</small>
            </div>
        {/each}
    </aside>

    <main>
        {#if selection}
            <h2>{selection.titre}</h2>
            <pre>{selection.contenu}</pre>
            <button on:click={() => supprimerNote(selection!.id)}>Supprimer</button>
        {:else}
            <input bind:value={titre} placeholder="Titre de la note" />
            <textarea bind:value={contenu} placeholder="Contenu..." rows={15}></textarea>
            <button on:click={creerNote}>Créer la note</button>
        {/if}
    </main>
</div>
```

---

## 16. Récapitulatif

| Concept | Outil Tauri |
|---|---|
| Appel Rust → JS | `app.emit("event", payload)` |
| Appel JS → Rust | `invoke('command', args)` |
| État partagé | `app.manage(T)` + `State<T>` dans les commands |
| Fichiers | `tauri-plugin-fs` |
| Dialogs OS | `tauri-plugin-dialog` |
| Persistence | `tauri-plugin-store` |
| Notifications | `tauri-plugin-notification` |
| Tray | `TrayIconBuilder` |
| Mise à jour | `tauri-plugin-updater` |
| Build | `npm run tauri build` |
| Mobile | `cargo tauri android/ios dev/build` |

> [!tip] Ressources
> - [tauri.app](https://tauri.app) — documentation officielle
> - [docs.rs/tauri](https://docs.rs/tauri) — API Rust complète
> - [awesome-tauri](https://github.com/tauri-apps/awesome-tauri) — exemples de la communauté

---

## Notes liées

### Backend Rust
- [[09 - Async Await et Tokio]] — async/await dans les commandes Tauri, `spawn_blocking`
- [[08 - Concurrence et Smart Pointers]] — `Arc<Mutex<T>>` pour le state partagé entre commandes
- [[11 - Unsafe Rust et FFI]] — accès aux APIs système natifs depuis Tauri
- [[04 - Gestion des Erreurs en Rust]] — gestion des erreurs dans les commandes `Result<T, String>`
- [[12 - GUI avec egui et eframe]] — alternative : UI en Rust natif sans HTML/CSS

### Frontend web pour Tauri
- [[01 - Svelte du Debutant a l'Expert]] — SvelteKit : frontend recommandé pour Tauri 2.0
- [[01 - Introduction a React]] — React : alternative populaire pour le frontend Tauri
- [[01 - Introduction a VueJS]] — Vue 3 : autre option frontend pour Tauri
- [[06 - Tailwind CSS Complet]] — Tailwind pour styler l'interface Tauri

### Déploiement et build
- [[04 - CI-CD avec GitHub Actions]] — pipeline CI/CD pour builder et releaser sur macOS/Windows/Linux
- [[14 - WebAssembly avec Rust]] — WASM : alternative pour apps web (sans desktop natif)
- [[01 - Docker]] — conteneurisation pour le build CI cross-platform

### Base de données et backend
- [[01 - Introduction au SQL]] — SQLite via `rusqlite` dans le backend Tauri
- [[15 - Web Frameworks Axum et Actix]] — Axum : serveur local embarqué dans Tauri si besoin
- [[02 - Securite Web OWASP]] — sécurité CSP, permissions Tauri, XSS prevention
