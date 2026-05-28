# 12 - GUI avec egui et eframe

> [!info] Prérequis
> Ce cours suppose la maîtrise des [[06 - Traits et Generiques|Traits et Génériques]], des [[08 - Concurrence et Smart Pointers|Smart Pointers]] et des bases de l'[[09 - Async Await et Tokio|async Rust]]. Connaître [[11 - Unsafe Rust et FFI]] est utile pour les sections textures avancées.

---

## 1. Immediate Mode GUI vs Retained Mode GUI

La distinction fondamentale entre les deux paradigmes GUI détermine l'architecture entière d'une application.

| Critère | Immediate Mode (egui) | Retained Mode (Qt, GTK, winit+widgets) |
|---|---|---|
| **Modèle** | Redessine tout à chaque frame | Maintient un arbre d'objets widget |
| **État UI** | Dans votre code applicatif | Dans le framework |
| **Callbacks** | Inexistants — on lit `Response` inline | Omniprésents (signaux, listeners) |
| **Complexité** | Très faible pour commencer | Élevée (sérialisation de l'état) |
| **Performance** | Plus de CPU (redraw constant) | Moins de CPU (dirty flags) |
| **Animations** | Nécessitent `request_repaint()` | Gérées nativement |
| **Threads** | Single-thread UI obligatoire | Parfois multi-thread |
| **Tests** | Difficiles (pas de scène persistante) | Plus faciles (requêtes sur l'arbre) |
| **Cas d'usage** | Outils, debuggers, GUI simple-rapide | Applications grand public complexes |

> [!tip] Quand choisir egui ?
> egui est parfait pour les outils internes, les éditeurs techniques, les dashboards de monitoring, les plugins audio (CLAP/VST), et toute app où la vitesse de développement prime sur l'UX grand public.

### Principe de fonctionnement

```rust
// À chaque frame (60fps ou sur événement) :
fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    egui::CentralPanel::default().show(ctx, |ui| {
        // Chaque appel CONSTRUIT ET AFFICHE le widget simultanément
        ui.label("Hello");
        if ui.button("Cliquer").clicked() {
            self.compteur += 1; // L'état est dans NOTRE struct
        }
        ui.label(format!("Clics : {}", self.compteur));
    });
}
```

---

## 2. Architecture egui

### Types fondamentaux

```
egui::Context  — Le contexte global (mémoire, style, input, output)
    │
    ├── egui::Ui      — Zone de layout en cours de remplissage
    │       ├── egui::Response   — Résultat de l'ajout d'un widget
    │       └── egui::Painter    — Peintre de formes 2D bas niveau
    │
    ├── egui::Style   — Tous les paramètres visuels
    └── egui::Memory  — État persistant entre frames (focus, drag...)
```

#### `egui::Context`

```rust
// Obtenu en paramètre de update() — jamais construit manuellement
let ctx: &egui::Context = ctx;

// Lire les inputs de la frame courante
ctx.input(|i| {
    let pressed_enter = i.key_pressed(egui::Key::Enter);
    let mouse_pos = i.pointer.hover_pos();
    let modifiers = i.modifiers; // .ctrl, .shift, .alt
});

// Modifier l'output (ce que le système OS va faire)
ctx.output_mut(|o| {
    o.cursor_icon = egui::CursorIcon::Grab;
    o.copied_text = "texte dans le presse-papiers".to_string();
});

// Demander un nouveau rendu
ctx.request_repaint(); // Au prochain vsync
ctx.request_repaint_after(std::time::Duration::from_millis(16)); // Dans 16ms
```

#### `egui::Ui`

```rust
// Créé implicitement par les panels/windows
// Expose tous les widgets + les primitives de layout
let ui: &mut egui::Ui = ui;

// Propriétés utiles
let available = ui.available_size(); // Vec2 — espace disponible
let max_rect = ui.max_rect();        // Rect — rectangle total disponible
let clip_rect = ui.clip_rect();      // Rect — zone de clipping effective
```

#### `egui::Response`

```rust
let response = ui.button("OK");

// États de l'interaction
response.clicked()          // Clic gauche relâché dans le widget
response.double_clicked()   // Double-clic
response.secondary_clicked()// Clic droit
response.middle_clicked()   // Clic molette
response.hovered()          // Souris au-dessus
response.dragged()          // En cours de drag
response.drag_delta()       // Vec2 — déplacement du drag cette frame
response.drag_started()     // Premier frame du drag
response.drag_released()    // Frame où le drag se termine
response.changed()          // Valeur modifiée (pour les inputs)
response.has_focus()        // Widget focusé (clavier)
response.lost_focus()       // Vient de perdre le focus
response.gained_focus()     // Vient de gagner le focus

// Enrichissement de la réponse
response.on_hover_text("Tooltip simple");
response.on_hover_ui(|ui| { ui.label("Tooltip complexe"); });
response.request_focus();   // Forcer le focus clavier
response.highlight();       // Surbrillance visuelle
```

---

## 3. Setup du projet — Cargo.toml

```toml
[package]
name    = "mon-app-gui"
version = "0.1.0"
edition = "2021"

[dependencies]
egui    = "0.29"
eframe  = { version = "0.29", default-features = false, features = [
    "accesskit",      # Accessibilité (lecteurs d'écran)
    "default_fonts",  # Polices intégrées
    "glow",           # Backend OpenGL (alternatif: "wgpu")
    "persistence",    # Sauvegarder l'état de l'app
] }
serde = { version = "1", features = ["derive"] } # Pour la persistence

# Optionnel — images
image = { version = "0.25", default-features = false, features = ["png", "jpeg"] }

# Optionnel — extras (RetainedImage, Table, etc.)
egui_extras = { version = "0.29", features = ["default", "image"] }

# Optionnel — profiling
puffin     = { version = "0.19", optional = true }
puffin_egui = { version = "0.29", optional = true }

[profile.release]
opt-level = 3
lto       = true
codegen-units = 1
strip     = true  # Réduit la taille du binaire

# Optimisation en mode debug pour la fluidité
[profile.dev]
opt-level = 1

[profile.dev.package."*"]
opt-level = 3
```

---

## 4. Le loop eframe — structure de base

```rust
use eframe::egui;

fn main() -> eframe::Result<()> {
    let native_options = eframe::NativeOptions {
        viewport: egui::ViewportBuilder::default()
            .with_title("Mon Application")
            .with_inner_size([800.0, 600.0])
            .with_min_inner_size([400.0, 300.0])
            .with_icon(/* eframe::icon_data::from_png_bytes(&[..]).unwrap() */),
        ..Default::default()
    };

    eframe::run_native(
        "Mon Application",        // Titre (aussi clé de persistence)
        native_options,
        Box::new(|cc| Ok(Box::new(MonApp::new(cc)))),
    )
}

/// L'état complet de l'application
#[derive(serde::Serialize, serde::Deserialize)]
#[serde(default)]
struct MonApp {
    nom: String,
    compteur: i32,
    sombre: bool,
    #[serde(skip)] // Ne pas persister les données volatiles
    texture: Option<egui::TextureHandle>,
}

impl Default for MonApp {
    fn default() -> Self {
        Self {
            nom: "monde".to_string(),
            compteur: 0,
            sombre: true,
            texture: None,
        }
    }
}

impl MonApp {
    fn new(cc: &eframe::CreationContext<'_>) -> Self {
        // Restaurer depuis le stockage persistant
        if let Some(storage) = cc.storage {
            return eframe::get_value(storage, eframe::APP_KEY).unwrap_or_default();
        }
        Default::default()
    }
}

impl eframe::App for MonApp {
    /// Appelée à chaque frame — cœur de l'application
    fn update(&mut self, ctx: &egui::Context, frame: &mut eframe::Frame) {
        // Thème
        if self.sombre {
            ctx.set_visuals(egui::Visuals::dark());
        } else {
            ctx.set_visuals(egui::Visuals::light());
        }

        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading(format!("Bonjour, {} !", self.nom));
            ui.text_edit_singleline(&mut self.nom);
            if ui.button("Incrémenter").clicked() {
                self.compteur += 1;
            }
            ui.label(format!("Compteur : {}", self.compteur));
            ui.checkbox(&mut self.sombre, "Mode sombre");
        });
    }

    /// Sauvegarder l'état avant fermeture
    fn save(&mut self, storage: &mut dyn eframe::Storage) {
        eframe::set_value(storage, eframe::APP_KEY, self);
    }

    /// Appelée quand l'utilisateur ferme la fenêtre
    fn on_exit(&mut self, _gl: Option<&eframe::glow::Context>) {
        println!("Au revoir !");
    }

    /// Nom de l'app (aussi identifiant de persistence)
    fn name(&self) -> &str {
        "Mon Application"
    }
}
```

---

## 5. Widgets Standards

### 5.1 Texte et typographie

```rust
// Labels simples
ui.label("Texte simple");
ui.heading("Titre H1");
ui.small("Petit texte");
ui.monospace("code_monospace");
ui.strong("Gras");
ui.weak("Atténué");

// RichText — combinaison d'attributs
use egui::{RichText, Color32, FontId, FontFamily};

ui.label(
    RichText::new("Texte stylisé")
        .color(Color32::from_rgb(100, 200, 100))
        .size(24.0)
        .strong()
        .italics()
        .underline()
        .strikethrough()
        .background_color(Color32::from_black_alpha(128))
        .font(FontId::new(20.0, FontFamily::Monospace))
);

// Texte sélectionnable (copiable)
ui.add(egui::Label::new("Texte copiable").selectable(true));

// Hyperlink
ui.hyperlink("https://egui.rs");
ui.hyperlink_to("Documentation egui", "https://docs.rs/egui");
```

### 5.2 Boutons

```rust
// Bouton standard — retourne Response
if ui.button("Cliquer").clicked() { /* action */ }

// Bouton petit format
if ui.small_button("×").clicked() { /* fermer */ }

// Bouton activé/désactivé
if ui.add_enabled(self.peut_valider, egui::Button::new("Valider")).clicked() {
    self.valider();
}

// Bouton avec image
if ui.add(egui::Button::image_and_text(
    egui::include_image!("../assets/icon.png"),
    "Avec icône"
)).clicked() { /* action */ }

// Bouton toggle (sélectionnable) — affiche l'état actif/inactif
ui.selectable_label(self.actif, "Activer");
// Version avec valeur enum
ui.selectable_value(&mut self.mode, Mode::Edition, "Édition");
ui.selectable_value(&mut self.mode, Mode::Lecture, "Lecture");

// Groupe de boutons radio horizontaux
ui.horizontal(|ui| {
    ui.radio_value(&mut self.couleur, Couleur::Rouge, "Rouge");
    ui.radio_value(&mut self.couleur, Couleur::Vert, "Vert");
    ui.radio_value(&mut self.couleur, Couleur::Bleu, "Bleu");
});
```

### 5.3 Champs de saisie

```rust
// Saisie mono-ligne (retourne Response, .changed() si modifié)
let response = ui.text_edit_singleline(&mut self.nom);
if response.lost_focus() && ui.input(|i| i.key_pressed(egui::Key::Enter)) {
    self.valider_nom();
}

// Saisie multi-ligne
ui.text_edit_multiline(&mut self.description);

// TextEdit avec options avancées
let response = ui.add(
    egui::TextEdit::singleline(&mut self.recherche)
        .hint_text("Rechercher...")    // Placeholder
        .desired_width(200.0)          // Largeur fixe
        .password(false)               // true = masquer le texte
        .font(egui::TextStyle::Monospace)
        .cursor_at_end(true)
        .interactive(true)
        .lock_focus(false)
);

// Champ mot de passe
ui.add(egui::TextEdit::singleline(&mut self.mdp).password(true));

// Saisie avec validation live
let response = ui.add(
    egui::TextEdit::singleline(&mut self.email)
        .hint_text("email@exemple.com")
);
if response.changed() {
    self.email_valide = self.email.contains('@');
}
if !self.email_valide {
    ui.colored_label(egui::Color32::RED, "Email invalide");
}
```

### 5.4 Numériques

```rust
// Slider
ui.add(egui::Slider::new(&mut self.volume, 0.0..=1.0)
    .text("Volume")
    .step_by(0.01)
    .fixed_decimals(2)
    .clamp_to_range(true)
    .show_value(true)
    .suffix(" dB")
    .logarithmic(true)    // Pour les valeurs audio
);

// Slider vertical
ui.add(egui::Slider::new(&mut self.hauteur, 0..=100)
    .vertical()
    .text("Hauteur")
);

// DragValue (compact, drag pour changer)
ui.horizontal(|ui| {
    ui.label("Taille :");
    ui.add(egui::DragValue::new(&mut self.taille)
        .speed(0.5)          // Vitesse de changement par pixel dragué
        .clamp_range(0..=9999)
        .suffix(" px")
        .fixed_decimals(0)
    );
});

// ProgressBar
ui.add(egui::ProgressBar::new(self.progression)  // 0.0..=1.0
    .text(format!("{:.0}%", self.progression * 100.0))
    .animate(true)         // Animation quand indéterminé
    .desired_width(300.0)
);

// ProgressBar indéterminé (chargement en cours)
ui.add(egui::ProgressBar::new(0.0).animate(true));
```

### 5.5 Booléens

```rust
// Checkbox
ui.checkbox(&mut self.actif, "Activer la fonctionnalité");

// Checkbox sans label
ui.add(egui::Checkbox::without_text(&mut self.cocher));

// Radio buttons
ui.radio_value(&mut self.choix, Choix::A, "Option A");
ui.radio_value(&mut self.choix, Choix::B, "Option B");

// Toggle (switch iOS-style) — via egui::toggle_ui ou crate externe
// egui natif propose ui.checkbox, pour un toggle custom voir § Custom Painting
```

### 5.6 ComboBox (liste déroulante)

```rust
let langages = ["Rust", "Python", "TypeScript", "Go", "C++"];

egui::ComboBox::from_label("Langage préféré")
    .selected_text(langages[self.langage_idx])
    .show_ui(ui, |ui| {
        for (idx, &nom) in langages.iter().enumerate() {
            ui.selectable_value(&mut self.langage_idx, idx, nom);
        }
    });

// ComboBox avec enum
#[derive(PartialEq, Clone, Copy, Debug)]
enum Theme { Clair, Sombre, Système }

egui::ComboBox::from_id_salt("theme")
    .selected_text(format!("{:?}", self.theme))
    .show_ui(ui, |ui| {
        ui.selectable_value(&mut self.theme, Theme::Clair, "Clair");
        ui.selectable_value(&mut self.theme, Theme::Sombre, "Sombre");
        ui.selectable_value(&mut self.theme, Theme::Système, "Système");
    });
```

### 5.7 Séparateurs et espacement

```rust
ui.separator();              // Ligne horizontale
ui.add_space(10.0);          // Espace vertical de 10px
ui.add(egui::Separator::default().horizontal().spacing(20.0));
ui.add(egui::Separator::default().vertical()); // Dans un layout horizontal
```

---

## 6. Système de Layout

### 6.1 Directions principales

```rust
// Horizontal — widgets côte à côte
ui.horizontal(|ui| {
    ui.label("Nom :");
    ui.text_edit_singleline(&mut self.nom);
    if ui.button("Effacer").clicked() { self.nom.clear(); }
});

// Horizontal avec wrapping (retour à la ligne automatique)
ui.horizontal_wrapped(|ui| {
    for tag in &self.tags {
        ui.button(tag);
    }
});

// Horizontal centré verticalement
ui.horizontal_centered(|ui| {
    ui.label("Centré :");
    ui.spinner(); // Widget de chargement
});

// Vertical — défaut, mais explicitable
ui.vertical(|ui| {
    ui.label("Ligne 1");
    ui.label("Ligne 2");
});

// Vertical centré horizontalement
ui.vertical_centered(|ui| {
    ui.heading("Centré");
    ui.button("Bouton centré");
});

// Vertical centré ET justifié (s'étire jusqu'aux bords)
ui.vertical_centered_justified(|ui| {
    ui.button("Pleine largeur");
});
```

### 6.2 Grid

```rust
egui::Grid::new("ma_grille")
    .num_columns(3)
    .spacing([20.0, 10.0])   // [horizontal, vertical]
    .striped(true)            // Alternance de couleurs de fond
    .min_col_width(100.0)
    .show(ui, |ui| {
        // En-têtes
        ui.strong("Nom");
        ui.strong("Valeur");
        ui.strong("Unité");
        ui.end_row(); // Obligatoire pour passer à la ligne suivante

        // Données
        for champ in &self.champs {
            ui.label(&champ.nom);
            ui.label(format!("{:.2}", champ.valeur));
            ui.label(&champ.unite);
            ui.end_row();
        }
    });
```

### 6.3 Colonnes

```rust
// Diviser en N colonnes de largeur égale
ui.columns(2, |cols| {
    cols[0].label("Colonne gauche");
    cols[0].button("Action gauche");
    
    cols[1].label("Colonne droite");
    cols[1].button("Action droite");
});
```

### 6.4 ScrollArea

```rust
// Scroll vertical simple
egui::ScrollArea::vertical()
    .max_height(300.0)
    .auto_shrink([false, false]) // Ne pas rétrécir si contenu petit
    .show(ui, |ui| {
        for i in 0..1000 {
            ui.label(format!("Ligne {}", i));
        }
    });

// Scroll horizontal uniquement
egui::ScrollArea::horizontal().show(ui, |ui| {
    ui.horizontal(|ui| {
        for i in 0..100 {
            ui.button(format!("Item {}", i));
        }
    });
});

// Scroll dans les deux directions
egui::ScrollArea::both().show(ui, |ui| {
    for y in 0..50 {
        ui.horizontal(|ui| {
            for x in 0..50 {
                ui.label(format!("({},{})", x, y));
            }
        });
    }
});

// Virtualisation — show_rows pour TRÈS grandes listes (performances optimales)
let nb_lignes = 100_000;
let hauteur_ligne = 18.0;
egui::ScrollArea::vertical()
    .show_rows(ui, hauteur_ligne, nb_lignes, |ui, range| {
        // Seules les lignes visibles sont rendues
        for i in range {
            ui.label(format!("Ligne {} sur {}", i, nb_lignes));
        }
    });
```

### 6.5 Allocation manuelle d'espace

```rust
// Réserver un espace et obtenir son Rect
let (rect, response) = ui.allocate_exact_size(
    egui::vec2(200.0, 50.0),
    egui::Sense::click_and_drag(),
);

// Dessiner dans ce rect
if ui.is_rect_visible(rect) {
    ui.painter().rect_filled(rect, 5.0, egui::Color32::DARK_BLUE);
    ui.painter().text(
        rect.center(),
        egui::Align2::CENTER_CENTER,
        "Zone custom",
        egui::FontId::default(),
        egui::Color32::WHITE,
    );
}

// Allouer un sous-Ui dans un Rect précis
let (id, rect) = ui.allocate_space(egui::vec2(300.0, 200.0));
let mut child_ui = ui.new_child(egui::UiBuilder::new().max_rect(rect));
child_ui.label("Dans le sous-Ui");
```

---

## 7. Panels et Fenêtres

### 7.1 Panels (structure principale)

```rust
fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    // Panel du haut — barre de menu/titre
    egui::TopBottomPanel::top("top_panel")
        .min_height(30.0)
        .max_height(60.0)
        .resizable(false)
        .show(ctx, |ui| {
            egui::menu::bar(ui, |ui| {
                ui.menu_button("Fichier", |ui| {
                    if ui.button("Nouveau").clicked() { self.nouveau(); ui.close_menu(); }
                    if ui.button("Ouvrir...").clicked() { self.ouvrir(); ui.close_menu(); }
                    ui.separator();
                    if ui.button("Quitter").clicked() { ctx.send_viewport_cmd(egui::ViewportCommand::Close); }
                });
                ui.menu_button("Aide", |ui| {
                    if ui.button("À propos").clicked() { self.a_propos = true; }
                });
            });
        });

    // Panel du bas — barre de statut
    egui::TopBottomPanel::bottom("status_bar").show(ctx, |ui| {
        ui.horizontal(|ui| {
            ui.label(&self.statut);
            ui.with_layout(egui::Layout::right_to_left(egui::Align::Center), |ui| {
                ui.label(format!("{:.1} fps", ctx.fps()));
            });
        });
    });

    // Panel latéral gauche
    egui::SidePanel::left("panneau_gauche")
        .resizable(true)
        .default_width(200.0)
        .width_range(150.0..=400.0)
        .show(ctx, |ui| {
            ui.heading("Navigation");
            ui.separator();
            for item in &self.navigation {
                ui.selectable_value(&mut self.page_active, item.id, &item.nom);
            }
        });

    // Panel latéral droit
    egui::SidePanel::right("panneau_droit")
        .resizable(true)
        .default_width(250.0)
        .show_animated(ctx, self.proprietes_visibles, |ui| {
            ui.heading("Propriétés");
            // Contenu conditionnel
        });

    // Panel central — tout l'espace restant
    egui::CentralPanel::default().show(ctx, |ui| {
        ui.heading(format!("Page : {}", self.page_active));
        self.afficher_page(ui);
    });
}
```

### 7.2 Windows flottantes

```rust
// Fenêtre simple
egui::Window::new("Paramètres")
    .default_pos([100.0, 100.0])  // Position initiale
    .default_size([350.0, 250.0])
    .resizable(true)
    .collapsible(true)
    .open(&mut self.params_ouverts) // bool pour fermer avec le ×
    .anchor(egui::Align2::CENTER_CENTER, [0.0, 0.0]) // Centrer à l'écran
    .show(ctx, |ui| {
        ui.label("Contenu de la fenêtre");
        ui.separator();
        ui.horizontal(|ui| {
            if ui.button("OK").clicked() { self.params_ouverts = false; }
            if ui.button("Annuler").clicked() { self.params_ouverts = false; }
        });
    });

// Fenêtre à taille fixe
egui::Window::new("Info")
    .fixed_size([400.0, 300.0])
    .collapsible(false)
    .show(ctx, |ui| { /* ... */ });

// Fenêtre sans décoration (frameless)
egui::Window::new("Splash")
    .title_bar(false)
    .resizable(false)
    .anchor(egui::Align2::CENTER_CENTER, egui::vec2(0.0, 0.0))
    .show(ctx, |ui| {
        ui.image(egui::include_image!("../assets/logo.png"));
        ui.centered_and_justified(|ui| {
            ui.spinner();
        });
    });
```

### 7.3 Area flottante (sans décoration)

```rust
// Area flottante — pour les tooltips custom, les overlays, etc.
egui::Area::new(egui::Id::new("mon_overlay"))
    .fixed_pos(egui::pos2(50.0, 50.0))
    .order(egui::Order::Foreground) // Au-dessus de tout
    .show(ctx, |ui| {
        egui::Frame::popup(ui.style()).show(ui, |ui| {
            ui.label("Overlay flottant");
        });
    });
```

### 7.4 Modal (dialog modale)

```rust
// Dialog modale — bloque l'interaction avec le reste
if self.confirmation_visible {
    egui::Modal::new(egui::Id::new("confirmation")).show(ctx, |ui| {
        ui.set_min_width(250.0);
        ui.heading("Confirmer la suppression");
        ui.separator();
        ui.label("Cette action est irréversible.");
        ui.separator();
        ui.horizontal(|ui| {
            if ui.button("Confirmer").clicked() {
                self.supprimer();
                self.confirmation_visible = false;
            }
            if ui.button("Annuler").clicked() {
                self.confirmation_visible = false;
            }
        });
    });
}
```

---

## 8. Menus et Context Menus

```rust
// Menu dans la barre de menu (voir TopBottomPanel ci-dessus)
egui::menu::bar(ui, |ui| {
    ui.menu_button("Edition", |ui| {
        if ui.add_enabled(self.peut_annuler, egui::Button::new("Annuler").shortcut_text("Ctrl+Z")).clicked() {
            self.annuler();
            ui.close_menu();
        }
        if ui.add_enabled(self.peut_refaire, egui::Button::new("Refaire").shortcut_text("Ctrl+Y")).clicked() {
            self.refaire();
            ui.close_menu();
        }
        ui.separator();
        ui.menu_button("Sous-menu", |ui| {
            if ui.button("Option A").clicked() { ui.close_menu(); }
            if ui.button("Option B").clicked() { ui.close_menu(); }
        });
    });
});

// Context menu au clic droit sur n'importe quelle Response
let response = ui.label("Clic droit ici");
response.context_menu(|ui| {
    if ui.button("Copier").clicked() {
        ctx.output_mut(|o| o.copied_text = "contenu copié".to_string());
        ui.close_menu();
    }
    if ui.button("Coller").clicked() {
        // ctx.input(|i| i.events...) pour lire le presse-papiers
        ui.close_menu();
    }
    ui.separator();
    if ui.button("Supprimer").clicked() {
        self.supprimer_selection();
        ui.close_menu();
    }
});
```

---

## 9. Interactions et Input Avancés

### 9.1 Clavier

```rust
ctx.input(|i| {
    // Touches spéciales
    if i.key_pressed(egui::Key::Escape) { self.annuler(); }
    if i.key_pressed(egui::Key::Enter)  { self.valider(); }
    if i.key_pressed(egui::Key::Delete) { self.supprimer_selection(); }
    
    // Combinaisons avec modificateurs
    if i.modifiers.ctrl && i.key_pressed(egui::Key::S) { self.sauvegarder(); }
    if i.modifiers.ctrl && i.key_pressed(egui::Key::Z) { self.annuler(); }
    if i.modifiers.ctrl && i.modifiers.shift && i.key_pressed(egui::Key::Z) { self.refaire(); }
    
    // Navigation
    if i.key_pressed(egui::Key::ArrowUp)    { self.selection_precedente(); }
    if i.key_pressed(egui::Key::ArrowDown)  { self.selection_suivante(); }
    if i.key_pressed(egui::Key::Home)       { self.aller_debut(); }
    if i.key_pressed(egui::Key::End)        { self.aller_fin(); }
    
    // Scroll de la molette
    let scroll_delta = i.raw_scroll_delta; // Vec2
    if scroll_delta.y.abs() > 0.0 {
        self.offset_scroll -= scroll_delta.y * 2.0;
    }
    
    // Zoom avec Ctrl+molette
    if i.modifiers.ctrl {
        self.zoom += i.raw_scroll_delta.y * 0.01;
        self.zoom = self.zoom.clamp(0.1, 10.0);
    }
});
```

### 9.2 Souris

```rust
ctx.input(|i| {
    // Position du pointeur
    if let Some(pos) = i.pointer.hover_pos() {
        self.pos_souris = pos; // Pos2
    }
    
    // Clics
    if i.pointer.primary_pressed()   { /* bouton gauche enfoncé */ }
    if i.pointer.primary_released()  { /* bouton gauche relâché */ }
    if i.pointer.secondary_pressed() { /* bouton droit enfoncé */ }
    
    // Drag global
    if i.pointer.is_decidedly_dragging() {
        let delta = i.pointer.delta(); // Vec2
        self.pan += delta;
    }
});
```

### 9.3 Drag & Drop

```rust
// Source de drag — attacher une payload à une Response
let response = ui.add(egui::Label::new(format!("📁 {}", self.nom)).sense(egui::Sense::drag()));
if response.drag_started() {
    response.dnd_set_drag_payload(self.id); // Stocker un identifiant
}

// Zone de dépôt — vérifier si un drag est au-dessus
let response = ui.add(egui::Label::new("Zone de dépôt").sense(egui::Sense::hover()));
if response.hovered() && ctx.drag_and_drop_payload::<usize>().is_some() {
    // Afficher un indicateur visuel
    ui.painter().rect_stroke(response.rect, 2.0, egui::Stroke::new(2.0, egui::Color32::YELLOW));
}
// Récupérer la payload au relâchement
if let Some(payload) = response.dnd_release_payload::<usize>() {
    self.deplacer_element(*payload, self.id_destination);
}
```

---

## 10. Custom Painting avec Painter

Le `Painter` permet de dessiner des formes 2D arbitraires dans le système de coordonnées d'egui.

```rust
// Obtenir un Painter pour l'UI courante
let painter = ui.painter();

// Obtenir un Painter dans un Rect spécifique
let (rect, response) = ui.allocate_exact_size(
    egui::vec2(400.0, 300.0),
    egui::Sense::click_and_drag(),
);
let painter = ui.painter_at(rect);

// Types de coordonnées
let p1 = egui::pos2(100.0, 50.0);   // Pos2 — position absolue dans l'écran
let p2 = egui::pos2(200.0, 150.0);
let v  = egui::vec2(50.0, 30.0);    // Vec2 — vecteur/taille
let r  = egui::Rect::from_min_max(p1, p2); // Rectangle

// Couleurs
let rouge     = egui::Color32::RED;
let vert      = egui::Color32::from_rgb(0, 200, 100);
let bleu_semi = egui::Color32::from_rgba_unmultiplied(0, 0, 255, 128);

// Contours (Stroke)
let contour = egui::Stroke::new(2.0, egui::Color32::WHITE);
let aucun   = egui::Stroke::NONE;

// === Formes ===

// Rectangle plein
painter.rect_filled(r, 8.0, vert);           // (rect, rayon_coins, couleur)

// Rectangle avec contour seulement
painter.rect_stroke(r, 5.0, contour, egui::StrokeKind::Outside);

// Rectangle avec remplissage ET contour
painter.rect(r, 5.0, bleu_semi, contour, egui::StrokeKind::Middle);

// Cercle plein
painter.circle_filled(p1, 30.0, rouge);       // (centre, rayon, couleur)

// Cercle contour
painter.circle_stroke(p1, 30.0, contour);

// Segment de droite
painter.line_segment([p1, p2], contour);

// Ligne brisée (polyline)
painter.add(egui::Shape::line(
    vec![p1, egui::pos2(150.0, 200.0), p2],
    contour,
));

// Flèche
painter.arrow(p1, v, contour);               // (origine, vecteur, contour)

// Texte
painter.text(
    rect.center(),                             // Position
    egui::Align2::CENTER_CENTER,               // Alignement
    "Mon texte",                               // Contenu
    egui::FontId::new(16.0, egui::FontFamily::Proportional),
    egui::Color32::WHITE,
);
```

### Mesh custom (triangles)

```rust
// Dessiner une forme quelconque avec des triangles
let mut mesh = egui::Mesh::default();

// Ajouter des sommets (position + UV + couleur)
let v0 = mesh.vertices.len() as u32;
mesh.vertices.push(egui::epaint::Vertex {
    pos: egui::pos2(100.0, 50.0),
    uv:  egui::pos2(0.0, 0.0),
    color: egui::Color32::RED,
});
mesh.vertices.push(egui::epaint::Vertex {
    pos: egui::pos2(200.0, 50.0),
    uv:  egui::pos2(1.0, 0.0),
    color: egui::Color32::GREEN,
});
mesh.vertices.push(egui::epaint::Vertex {
    pos: egui::pos2(150.0, 150.0),
    uv:  egui::pos2(0.5, 1.0),
    color: egui::Color32::BLUE,
});

// Ajouter les indices (3 par triangle)
mesh.indices.extend_from_slice(&[v0, v0 + 1, v0 + 2]);

painter.add(egui::Shape::mesh(mesh));
```

### Widget custom complet — Jauge circulaire

```rust
fn jauge_circulaire(ui: &mut egui::Ui, valeur: f32, max: f32, label: &str) -> egui::Response {
    let taille = egui::vec2(80.0, 80.0);
    let (rect, response) = ui.allocate_exact_size(taille, egui::Sense::hover());

    if ui.is_rect_visible(rect) {
        let painter = ui.painter();
        let centre = rect.center();
        let rayon = rect.width().min(rect.height()) / 2.0 - 4.0;
        let ratio = (valeur / max).clamp(0.0, 1.0);

        // Fond de la jauge
        painter.circle_filled(centre, rayon, egui::Color32::from_gray(50));

        // Arc de progression (approximation avec segments)
        let nb_segments = 64;
        let angle_debut = -std::f32::consts::PI / 2.0; // Partir du haut
        let angle_fin   = angle_debut + ratio * 2.0 * std::f32::consts::PI;
        let mut points = vec![];
        for i in 0..=nb_segments {
            let t = i as f32 / nb_segments as f32;
            let angle = angle_debut + t * (angle_fin - angle_debut);
            points.push(centre + egui::vec2(angle.cos(), angle.sin()) * (rayon - 4.0));
        }
        painter.add(egui::Shape::line(
            points,
            egui::Stroke::new(6.0, egui::Color32::from_rgb(100, 200, 100)),
        ));

        // Texte central
        painter.text(centre, egui::Align2::CENTER_CENTER,
            format!("{:.0}%", ratio * 100.0),
            egui::FontId::new(14.0, egui::FontFamily::Proportional),
            egui::Color32::WHITE,
        );

        // Label sous la jauge
        painter.text(
            centre + egui::vec2(0.0, rayon + 12.0),
            egui::Align2::CENTER_CENTER,
            label,
            egui::FontId::new(11.0, egui::FontFamily::Proportional),
            egui::Color32::GRAY,
        );
    }

    response
}
```

---

## 11. Styling et Theming

### 11.1 Thèmes intégrés

```rust
// Appliquer en début de update()
ctx.set_visuals(egui::Visuals::dark());   // Thème sombre
ctx.set_visuals(egui::Visuals::light());  // Thème clair

// Modifier un thème de base
let mut visuals = egui::Visuals::dark();
visuals.override_text_color = Some(egui::Color32::from_rgb(220, 220, 220));
visuals.window_rounding = egui::Rounding::same(8.0);
visuals.window_shadow = egui::epaint::Shadow { blur: 20, spread: 5, offset: [0, 4].into(), color: egui::Color32::from_black_alpha(100) };
ctx.set_visuals(visuals);
```

### 11.2 Style complet

```rust
ctx.style_mut(|style| {
    // Espacement
    style.spacing.item_spacing        = egui::vec2(8.0, 6.0);   // Entre widgets
    style.spacing.button_padding      = egui::vec2(12.0, 6.0);  // Padding dans les boutons
    style.spacing.indent              = 20.0;                    // Indentation
    style.spacing.slider_width        = 200.0;
    style.spacing.text_edit_width     = 280.0;
    style.spacing.interact_size       = egui::vec2(40.0, 20.0); // Taille min cliquable

    // Coins arrondis globaux
    style.visuals.window_rounding = egui::Rounding::same(10.0);
    
    // Couleurs des widgets
    style.visuals.widgets.inactive.bg_fill = egui::Color32::from_rgb(40, 40, 50);
    style.visuals.widgets.hovered.bg_fill  = egui::Color32::from_rgb(60, 60, 80);
    style.visuals.widgets.active.bg_fill   = egui::Color32::from_rgb(80, 80, 120);

    // Interactivité
    style.interaction.show_tooltips_only_when_still = true;
    style.interaction.tooltip_delay = 0.5; // secondes
    style.interaction.multi_widget_text_select = true;
});
```

### 11.3 Polices custom

```rust
fn configurer_polices(ctx: &egui::Context) {
    let mut fonts = egui::FontDefinitions::default();

    // Charger une police depuis les bytes (embarquée dans le binaire)
    fonts.font_data.insert(
        "MaPolice".to_owned(),
        egui::FontData::from_static(include_bytes!("../assets/fonts/Roboto-Regular.ttf")).into(),
    );

    // Charger une police depuis le filesystem
    let data = std::fs::read("assets/fonts/JetBrainsMono.ttf").unwrap();
    fonts.font_data.insert(
        "JetBrainsMono".to_owned(),
        egui::FontData::from_owned(data).into(),
    );

    // Définir la priorité par famille de police
    // Les polices en tête de liste ont la priorité
    fonts.families.entry(egui::FontFamily::Proportional)
        .or_default()
        .insert(0, "MaPolice".to_owned());

    fonts.families.entry(egui::FontFamily::Monospace)
        .or_default()
        .insert(0, "JetBrainsMono".to_owned());

    ctx.set_fonts(fonts);
}

// Dans MonApp::new()
configurer_polices(&cc.egui_ctx);

// Utilisation
ui.label(RichText::new("Roboto").font(
    egui::FontId::new(16.0, egui::FontFamily::Proportional)
));
ui.label(RichText::new("JetBrains Mono").font(
    egui::FontId::new(14.0, egui::FontFamily::Monospace)
));

// Police nommée (familles custom)
fonts.families.insert(
    egui::FontFamily::Name("Titre".into()),
    vec!["MaPolice".to_owned()],
);
ui.label(RichText::new("Titre").font(
    egui::FontId::new(32.0, egui::FontFamily::Name("Titre".into()))
));
```

### 11.4 Frame decorative

```rust
// Encadrer du contenu avec un style visuel
egui::Frame::none()
    .fill(egui::Color32::from_gray(25))
    .rounding(egui::Rounding::same(8.0))
    .stroke(egui::Stroke::new(1.0, egui::Color32::from_gray(60)))
    .inner_margin(egui::Margin::same(12.0))
    .shadow(egui::epaint::Shadow { blur: 8, ..Default::default() })
    .show(ui, |ui| {
        ui.heading("Contenu encadré");
        ui.label("Avec marges et coins arrondis");
    });

// Styles prédéfinis
egui::Frame::group(ui.style()).show(ui, |ui| { ui.label("Groupe"); });
egui::Frame::popup(ui.style()).show(ui, |ui| { ui.label("Popup"); });
egui::Frame::side_top_panel(ui.style()).show(ui, |ui| { ui.label("Panel"); });
```

---

## 12. Textures et Images

### 12.1 Charger une image depuis bytes

```rust
// Embarquée dans le binaire à la compilation
fn charger_texture(ctx: &egui::Context) -> egui::TextureHandle {
    // Méthode 1 : via egui::include_image! (la plus simple)
    // Utilisation directe dans ui.add(egui::Image::new(egui::include_image!("icon.png")))

    // Méthode 2 : via image crate pour plus de contrôle
    let image_bytes = include_bytes!("../assets/icon.png");
    let image = image::load_from_memory(image_bytes)
        .expect("Image invalide")
        .to_rgba8();
    let (w, h) = image.dimensions();

    ctx.load_texture(
        "mon-icon",
        egui::ColorImage::from_rgba_unmultiplied(
            [w as usize, h as usize],
            &image.into_raw(),
        ),
        egui::TextureOptions {
            magnification: egui::TextureFilter::Linear,
            minification:  egui::TextureFilter::Linear,
            ..Default::default()
        },
    )
}

// Dans new()
self.texture = Some(charger_texture(&cc.egui_ctx));

// Dans update()
if let Some(ref texture) = self.texture {
    ui.add(egui::Image::new(texture)
        .max_size(egui::vec2(200.0, 200.0))
        .fit_to_exact_size(egui::vec2(100.0, 100.0))
        .rounding(egui::Rounding::same(8.0))
        .tint(egui::Color32::WHITE)
    );

    // Bouton avec image
    if ui.add(egui::ImageButton::new(texture)).clicked() {
        // action
    }
}
```

### 12.2 Image via URI (eframe web ou fichier)

```rust
// egui gère automatiquement les URI locaux (file://) et HTTP (avec feature)
ui.add(egui::Image::new("file://assets/background.png")
    .max_size(egui::vec2(800.0, 600.0))
);

// Image intégrée avec macro (recommandé pour les binaires)
ui.add(egui::Image::new(egui::include_image!("../assets/logo.png")));
```

---

## 13. State Management et Persistence

### 13.1 Données temporaires par Id

```rust
// Stocker un état éphémère (non persisté) associé à un widget
let id = ui.id().with("mon_etat_local");
let mut valeur = ctx.data_mut(|d| d.get_temp::<f32>(id).unwrap_or(0.0));
ui.add(egui::Slider::new(&mut valeur, 0.0..=100.0));
ctx.data_mut(|d| d.insert_temp(id, valeur));
```

### 13.2 Persistence complète avec serde

```rust
// Sur le struct App — déjà montré, mais voici les détails
#[derive(serde::Serialize, serde::Deserialize, Default)]
#[serde(default)] // Valeurs par défaut si champ manquant (versions futures)
struct AppState {
    // Champs persistés
    theme: Theme,
    recent_files: Vec<String>,
    fenetre_taille: [f32; 2],
    
    // Champs non persistés
    #[serde(skip)]
    texture_cache: std::collections::HashMap<String, egui::TextureHandle>,
    #[serde(skip)]
    en_chargement: bool,
}

// Sauvegarde dans save()
fn save(&mut self, storage: &mut dyn eframe::Storage) {
    eframe::set_value(storage, eframe::APP_KEY, self);
    // Aussi possible : sauvegarder avec des clés custom
    eframe::set_value(storage, "config_avancee", &self.config);
}

// Restauration dans new()
fn new(cc: &eframe::CreationContext<'_>) -> Self {
    let state: AppState = cc.storage
        .and_then(|s| eframe::get_value(s, eframe::APP_KEY))
        .unwrap_or_default();
    state
}
```

---

## 14. Performance

```rust
// Ne repeindre que si nécessaire (économise le CPU)
// Par défaut eframe repeint à chaque événement et à 60fps
// Pour repeindre uniquement sur événement :
let native_options = eframe::NativeOptions {
    // Pas d'option directe — utiliser ctx.request_repaint() selon les besoins
    ..Default::default()
};

// Demander un repaint dans X millisecondes
ctx.request_repaint_after(std::time::Duration::from_millis(100));

// Dans un thread background qui produit des données
let ctx_clone = ctx.clone();
std::thread::spawn(move || {
    std::thread::sleep(std::time::Duration::from_secs(1));
    // Après calcul, notifier l'UI de se repeindre
    ctx_clone.request_repaint();
});

// Éviter les allocations dans update() — utiliser des buffers pré-alloués
// ❌ Alloue à chaque frame
let texte = format!("Valeur : {}", self.valeur);
ui.label(&texte);

// ✅ Réutilise le buffer
self.buffer_texte.clear();
use std::fmt::Write;
write!(&mut self.buffer_texte, "Valeur : {}", self.valeur).unwrap();
ui.label(&self.buffer_texte);
```

### Profiling avec puffin

```toml
[features]
profiling = ["dep:puffin", "dep:puffin_egui"]
```

```rust
#[cfg(feature = "profiling")]
fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    puffin::GlobalProfiler::lock().new_frame();
    puffin::profile_scope!("update");

    {
        puffin::profile_scope!("dessiner_scene");
        self.dessiner_scene(ctx);
    }

    #[cfg(feature = "profiling")]
    puffin_egui::profiler_window(ctx);
}
```

---

## 15. Intégration [[09 - Async Await et Tokio|Tokio Async]]

```rust
use std::sync::mpsc;

struct App {
    // Canal de communication thread → UI
    rx: mpsc::Receiver<String>,
    tx: mpsc::Sender<String>,
    messages: Vec<String>,
    ctx: egui::Context,
}

impl App {
    fn new(cc: &eframe::CreationContext<'_>) -> Self {
        let (tx, rx) = mpsc::channel();
        let ctx = cc.egui_ctx.clone();

        // Lancer un runtime Tokio en arrière-plan
        let tx_clone = tx.clone();
        let ctx_clone = ctx.clone();
        std::thread::spawn(move || {
            let rt = tokio::runtime::Runtime::new().unwrap();
            rt.block_on(async move {
                // Fetch HTTP async
                match reqwest::get("https://api.exemple.com/data").await {
                    Ok(resp) => {
                        let texte = resp.text().await.unwrap_or_default();
                        tx_clone.send(texte).ok();
                        ctx_clone.request_repaint(); // Réveiller l'UI
                    }
                    Err(e) => {
                        tx_clone.send(format!("Erreur : {}", e)).ok();
                        ctx_clone.request_repaint();
                    }
                }
            });
        });

        Self { rx, tx, messages: vec![], ctx }
    }
}

impl eframe::App for App {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        // Drainer les messages du canal sans bloquer
        while let Ok(msg) = self.rx.try_recv() {
            self.messages.push(msg);
        }

        egui::CentralPanel::default().show(ctx, |ui| {
            egui::ScrollArea::vertical().show(ui, |ui| {
                for msg in &self.messages {
                    ui.label(msg);
                }
            });
        });
    }
}
```

---

## 16. Target WASM

```toml
# Cargo.toml — features pour WASM
[features]
web = []

[dependencies]
eframe = { version = "0.29", features = ["web"] }
wasm-bindgen-futures = "0.4"
web-sys = "0.3"
```

```toml
# Trunk.toml — à la racine du projet
[build]
target = "index.html"
dist   = "dist"

[[proxy]]
rewrite = "/api"
backend = "http://localhost:8080"
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Mon App egui WASM</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        html, body, canvas { margin: 0; padding: 0; width: 100%; height: 100%; overflow: hidden; }
    </style>
</head>
<body>
    <!-- trunk injecte automatiquement le JS/WASM -->
</body>
</html>
```

```rust
// main.rs — point d'entrée différent selon la target
#[cfg(not(target_arch = "wasm32"))]
fn main() -> eframe::Result<()> {
    eframe::run_native(
        "Mon App",
        eframe::NativeOptions::default(),
        Box::new(|cc| Ok(Box::new(MonApp::new(cc)))),
    )
}

#[cfg(target_arch = "wasm32")]
fn main() {
    // Panique vers la console JS
    console_error_panic_hook::set_once();
    wasm_bindgen_futures::spawn_local(async {
        eframe::WebRunner::new()
            .start(
                "canvas_id", // id du <canvas> dans le HTML
                eframe::WebOptions::default(),
                Box::new(|cc| Ok(Box::new(MonApp::new(cc)))),
            )
            .await
            .expect("Échec du démarrage eframe web");
    });
}
```

```bash
# Développement
trunk serve --open

# Production
trunk build --release

# Déployer dist/ sur GitHub Pages, Netlify, Vercel...
```

---

## 17. Projet Complet 1 — App de Notes

```rust
use eframe::egui;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Clone)]
struct Note {
    id:      u64,
    titre:   String,
    contenu: String,
}

#[derive(Serialize, Deserialize, Default)]
#[serde(default)]
struct AppNotes {
    notes:          Vec<Note>,
    selection:      Option<u64>,
    prochain_id:    u64,
    mode_edition:   bool,
    #[serde(skip)]
    brouillon:      String,
    #[serde(skip)]
    titre_brouillon: String,
}

impl AppNotes {
    fn nouvelle_note(&mut self) {
        let id = self.prochain_id;
        self.prochain_id += 1;
        self.notes.push(Note {
            id,
            titre:   "Nouvelle note".to_string(),
            contenu: String::new(),
        });
        self.selection = Some(id);
        self.mode_edition = true;
        if let Some(note) = self.note_selectionnee() {
            self.brouillon       = note.contenu.clone();
            self.titre_brouillon = note.titre.clone();
        }
    }

    fn note_selectionnee(&self) -> Option<&Note> {
        self.selection.and_then(|id| self.notes.iter().find(|n| n.id == id))
    }

    fn note_selectionnee_mut(&mut self) -> Option<&mut Note> {
        self.selection.and_then(|id| self.notes.iter_mut().find(|n| n.id == id))
    }

    fn supprimer_selectionnee(&mut self) {
        if let Some(id) = self.selection {
            self.notes.retain(|n| n.id != id);
            self.selection = self.notes.last().map(|n| n.id);
        }
    }
}

impl eframe::App for AppNotes {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        ctx.set_visuals(egui::Visuals::dark());

        // Barre latérale — liste des notes
        egui::SidePanel::left("liste_notes")
            .default_width(220.0)
            .show(ctx, |ui| {
                ui.horizontal(|ui| {
                    ui.heading("Notes");
                    ui.with_layout(egui::Layout::right_to_left(egui::Align::Center), |ui| {
                        if ui.button("+").on_hover_text("Nouvelle note").clicked() {
                            self.nouvelle_note();
                        }
                    });
                });
                ui.separator();

                egui::ScrollArea::vertical().show(ui, |ui| {
                    let notes = self.notes.clone();
                    for note in &notes {
                        let actif = self.selection == Some(note.id);
                        if ui.selectable_label(actif, &note.titre).clicked() {
                            self.selection = Some(note.id);
                            self.mode_edition = false;
                            self.brouillon       = note.contenu.clone();
                            self.titre_brouillon = note.titre.clone();
                        }
                    }
                });
            });

        // Zone principale — contenu de la note
        egui::CentralPanel::default().show(ctx, |ui| {
            if let Some(id) = self.selection {
                if self.mode_edition {
                    // Mode édition
                    ui.horizontal(|ui| {
                        ui.add(egui::TextEdit::singleline(&mut self.titre_brouillon)
                            .font(egui::TextStyle::Heading)
                            .desired_width(f32::INFINITY)
                        );
                    });
                    ui.separator();

                    egui::ScrollArea::vertical().show(ui, |ui| {
                        ui.add(egui::TextEdit::multiline(&mut self.brouillon)
                            .desired_width(f32::INFINITY)
                            .desired_rows(30)
                            .hint_text("Commencez à écrire...")
                        );
                    });

                    ui.separator();
                    ui.horizontal(|ui| {
                        if ui.button("💾 Sauvegarder").clicked() {
                            if let Some(note) = self.note_selectionnee_mut() {
                                note.contenu = self.brouillon.clone();
                                note.titre   = self.titre_brouillon.clone();
                            }
                            self.mode_edition = false;
                        }
                        if ui.button("Annuler").clicked() {
                            self.mode_edition = false;
                        }
                        ui.with_layout(egui::Layout::right_to_left(egui::Align::Center), |ui| {
                            if ui.button("🗑 Supprimer").clicked() {
                                self.supprimer_selectionnee();
                            }
                        });
                    });
                } else {
                    // Mode lecture
                    if let Some(note) = self.note_selectionnee() {
                        let titre   = note.titre.clone();
                        let contenu = note.contenu.clone();
                        ui.horizontal(|ui| {
                            ui.heading(&titre);
                            ui.with_layout(egui::Layout::right_to_left(egui::Align::Center), |ui| {
                                if ui.button("✏ Éditer").clicked() {
                                    self.mode_edition = true;
                                }
                            });
                        });
                        ui.separator();
                        egui::ScrollArea::vertical().show(ui, |ui| {
                            ui.label(&contenu);
                        });
                    }
                }
            } else {
                ui.centered_and_justified(|ui| {
                    ui.label("Sélectionnez ou créez une note");
                });
            }
        });
    }

    fn save(&mut self, storage: &mut dyn eframe::Storage) {
        eframe::set_value(storage, eframe::APP_KEY, self);
    }
}
```

---

## 18. Projet Complet 2 — Color Picker Custom

```rust
#[derive(Default)]
struct ColorPicker {
    r: u8, g: u8, b: u8, a: u8,
    hex: String,
}

impl ColorPicker {
    fn couleur(&self) -> egui::Color32 {
        egui::Color32::from_rgba_unmultiplied(self.r, self.g, self.b, self.a)
    }

    fn depuis_couleur(&mut self, c: egui::Color32) {
        let [r, g, b, a] = c.to_array();
        self.r = r; self.g = g; self.b = b; self.a = a;
        self.hex = format!("#{:02X}{:02X}{:02X}", r, g, b);
    }

    fn depuis_hex(&mut self) {
        let hex = self.hex.trim_start_matches('#');
        if hex.len() == 6 {
            if let (Ok(r), Ok(g), Ok(b)) = (
                u8::from_str_radix(&hex[0..2], 16),
                u8::from_str_radix(&hex[2..4], 16),
                u8::from_str_radix(&hex[4..6], 16),
            ) {
                self.r = r; self.g = g; self.b = b;
            }
        }
    }
}

impl eframe::App for ColorPicker {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading("Color Picker");
            ui.separator();

            // Sliders RGB
            let mut modifie = false;
            modifie |= ui.add(egui::Slider::new(&mut self.r, 0..=255).text("Rouge").suffix(" R")).changed();
            modifie |= ui.add(egui::Slider::new(&mut self.g, 0..=255).text("Vert").suffix(" G")).changed();
            modifie |= ui.add(egui::Slider::new(&mut self.b, 0..=255).text("Bleu").suffix(" B")).changed();
            modifie |= ui.add(egui::Slider::new(&mut self.a, 0..=255).text("Alpha").suffix(" A")).changed();

            if modifie {
                self.hex = format!("#{:02X}{:02X}{:02X}", self.r, self.g, self.b);
            }

            ui.separator();

            // Saisie hex
            ui.horizontal(|ui| {
                ui.label("Hex :");
                let resp = ui.text_edit_singleline(&mut self.hex);
                if resp.lost_focus() { self.depuis_hex(); }
            });

            ui.separator();

            // Aperçu de la couleur
            let (rect, _) = ui.allocate_exact_size(egui::vec2(200.0, 80.0), egui::Sense::hover());
            ui.painter().rect_filled(rect, 8.0, self.couleur());
            // Damier pour l'alpha
            if self.a < 255 {
                // Afficher damier en dessous (simplifié)
                ui.painter().text(
                    rect.center(),
                    egui::Align2::CENTER_CENTER,
                    format!("Alpha: {}/255", self.a),
                    egui::FontId::default(),
                    if self.a > 128 { egui::Color32::BLACK } else { egui::Color32::WHITE },
                );
            }

            // Copier dans le presse-papiers
            if ui.button("📋 Copier hex").clicked() {
                ctx.output_mut(|o| o.copied_text = self.hex.clone());
            }
        });
    }
}
```

---

## 19. Accessibilité et Bonnes Pratiques

```rust
// Toujours ajouter des tooltips sur les contrôles non évidents
ui.button("×")
    .on_hover_text("Fermer la fenêtre (Échap)");

// Labels explicites pour les champs
ui.horizontal(|ui| {
    ui.label("Nom d'utilisateur :"); // Label AVANT le widget
    ui.text_edit_singleline(&mut self.nom);
});

// Focus clavier
if ui.button("OK").clicked() || (
    ui.input(|i| i.key_pressed(egui::Key::Enter)) && self.dialog_active
) {
    self.valider();
}

// Tab order implicite — egui gère automatiquement le Tab entre les widgets
// Pour forcer le focus sur un widget au démarrage :
let response = ui.text_edit_singleline(&mut self.recherche);
if self.premier_rendu {
    response.request_focus();
    self.premier_rendu = false;
}

// Couleurs de contraste suffisant
// Minimum WCAG AA : 4.5:1 pour le texte normal
// ✅ Blanc sur gris foncé
// ❌ Gris clair sur blanc
```

> [!warning] Thread Safety
> egui est **single-thread** côté UI. Ne jamais appeler des méthodes de `Ui` depuis un autre thread. Utiliser des canaux `mpsc` et `ctx.request_repaint()` pour communiquer depuis les threads background (voir § 15).

---

## 20. Récapitulatif — Cheatsheet

| Besoin | Solution |
|---|---|
| Widget simple | `ui.label()`, `ui.button()`, `ui.checkbox()` |
| Input texte | `ui.text_edit_singleline/multiline()` |
| Numérique | `ui.add(Slider::new())`, `ui.add(DragValue::new())` |
| Liste déroulante | `egui::ComboBox::from_label()` |
| Layout horizontal | `ui.horizontal(\|ui\| { ... })` |
| Layout grille | `egui::Grid::new("id").show(ui, \|ui\| { ... })` |
| Scroll | `egui::ScrollArea::vertical().show(ui, \|ui\| {...})` |
| Panels | `TopBottomPanel`, `SidePanel`, `CentralPanel` |
| Fenêtre flottante | `egui::Window::new("titre").show(ctx, ...)` |
| Dialog modale | `egui::Modal::new(id).show(ctx, ...)` |
| Context menu | `response.context_menu(\|ui\| { ... })` |
| Dessin custom | `ui.painter()` + `painter.rect_filled()`, etc. |
| Image | `egui::Image::new(egui::include_image!(...))` |
| Police custom | `ctx.set_fonts(FontDefinitions { ... })` |
| Couleur | `egui::Color32::from_rgb(r, g, b)` |
| Persistence | `#[derive(Serialize, Deserialize)]` + `save()` |
| Async → UI | `mpsc::channel()` + `ctx.request_repaint()` |
| WASM | `trunk serve`, `eframe::WebRunner` |
| Performance | `ctx.request_repaint_after()`, éviter les allocs |

> [!tip] Ressources
> - [egui demo live](https://www.egui.rs) — tous les widgets en action
> - [docs.rs/egui](https://docs.rs/egui) — référence API complète
> - [egui Discord](https://discord.gg/JFcEma9bJq) — communauté active
> - [egui_extras](https://docs.rs/egui_extras) — TableBuilder, RetainedImage, DatePicker

---

## Notes liées

### Rust — prérequis et compléments
- [[02 - Ownership et Borrowing]] — comprendre les lifetimes avant de gérer l'état egui
- [[06 - Traits et Generiques]] — `Widget` trait, `IntoResponse`, génériques dans les composants custom
- [[08 - Concurrence et Smart Pointers]] — `Arc<Mutex<T>>` pour partager l'état avec des threads async
- [[09 - Async Await et Tokio]] — pattern channel + `ctx.request_repaint()` pour les updates async
- [[10 - Macros et Metaprogrammation]] — macros egui comme `ui.add(...)`, `egui::widgets!`
- [[11 - Unsafe Rust et FFI]] — intégration de libs C (OpenGL, Vulkan) avec egui
- [[07 - Projet Rust CLI]] — point de départ Rust avant de passer à l'UI

### Alternatives GUI Rust
- [[13 - Tauri Desktop Applications]] — Tauri : backend Rust + frontend web (HTML/CSS/JS)
- [[14 - WebAssembly avec Rust]] — egui compilé en WASM pour le navigateur avec Trunk
- [[15 - Web Frameworks Axum et Actix]] — serveur Axum pour les apps egui client-serveur

### Comparaison avec d'autres frameworks UI
- [[01 - Introduction a React]] — React : retained mode, composants, hooks (à comparer avec immediate mode)
- [[01 - Introduction a VueJS]] — Vue : similar component model, utile pour comprendre les différences
- [[02 - CSS Fondamentaux]] — styling CSS vs theming egui (`Visuals`, `Style`)
- [[02 - Flutter Complet]] — Flutter : aussi immediate-mode-like, cross-platform

### Déploiement et qualité
- [[04 - CI-CD avec GitHub Actions]] — pipeline CI pour builder et releaser une app egui
- [[01 - Docker]] — conteneuriser une app egui Linux pour la distribution
