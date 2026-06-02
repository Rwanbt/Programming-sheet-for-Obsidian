# SASS et SCSS — Préprocesseurs CSS

SASS (Syntactically Awesome StyleSheets) est un préprocesseur CSS qui étend le langage CSS avec des fonctionnalités de programmation : variables, fonctions, boucles, modules et héritage. Il compile vers du CSS standard, compris par tous les navigateurs. Maîtriser SASS est une compétence incontournable pour tout développeur front-end professionnel travaillant sur des projets de taille réelle.

> [!info] Prérequis
> Ce cours suppose que vous maîtrisez CSS (sélecteurs, box model, flexbox/grid, media queries) et les bases de la ligne de commande. Les exemples utilisent Node.js v18+.

---

## 1. Introduction aux préprocesseurs CSS

### 1.1 Le problème que résout SASS

CSS pur présente des limitations structurelles qui deviennent douloureux dès qu'un projet dépasse quelques fichiers :

```css
/* CSS pur — douleurs réelles sur un grand projet */

/* 1. Répétition de valeurs */
.btn-primary { background-color: #3498db; }
.btn-primary:hover { background-color: #2980b9; } /* c'est quoi cette valeur ? */
.link-primary { color: #3498db; }
.header { border-bottom: 2px solid #3498db; }
/* Si on change la couleur principale → chercher/remplacer dans 40 fichiers */

/* 2. Pas de nesting → sélecteurs redondants */
.card { padding: 1rem; }
.card .card-title { font-size: 1.5rem; }
.card .card-title span { color: gray; }
.card .card-body { padding: 0.5rem; }
.card .card-footer { border-top: 1px solid #eee; }

/* 3. Pas de logique → duplication de code */
.mt-1 { margin-top: 4px; }
.mt-2 { margin-top: 8px; }
.mt-3 { margin-top: 12px; }
/* ... écrire ces 12 classes à la main */
```

Un préprocesseur CSS est un programme qui lit un langage étendu (SASS/SCSS, Less, Stylus) et produit du CSS valide en sortie. Vous écrivez dans le langage étendu, le navigateur reçoit du CSS.

```
```
Votre cerveau       Votre éditeur         Terminal          Navigateur
    │                    │                    │                  │
    │   .scss/.sass       │   sass input.scss  │                  │
    │──────────────────▶  │──────────────────▶│                  │
    │                    │                    │  style.css       │
    │                    │                    │─────────────────▶│
```
```

### 1.2 SASS vs SCSS — deux syntaxes, un seul moteur

SASS propose **deux syntaxes** pour le même moteur de compilation :

| Caractéristique | Syntaxe SASS (indentée) | Syntaxe SCSS (Sassy CSS) |
|---|---|---|
| Extension de fichier | `.sass` | `.scss` |
| Accolades `{}` | Non — indentation | Oui |
| Points-virgules `;` | Non | Oui |
| Compatibilité CSS | Non (syntaxe différente) | Oui (tout CSS valide est du SCSS valide) |
| Adoption industrie | Minoritaire | **Standard de facto** |
| Courbe d'apprentissage | Rapide si vous aimez Python | Immédiate si vous connaissez CSS |

```sass
// Syntaxe SASS (indentée) — .sass
$color-primary: #3498db

.card
  padding: 1rem
  background: white

  .card-title
    color: $color-primary
    font-size: 1.5rem
```

```scss
// Syntaxe SCSS — .scss (ce que vous utiliserez)
$color-primary: #3498db;

.card {
  padding: 1rem;
  background: white;

  .card-title {
    color: $color-primary;
    font-size: 1.5rem;
  }
}
```

> [!warning] Choix de syntaxe
> Ce cours utilise **exclusivement SCSS**. C'est le standard de l'industrie, utilisé par Bootstrap, Foundation, Material UI et la quasi-totalité des équipes professionnelles. La syntaxe SASS indentée est quasi-abandonnée dans les nouveaux projets.

### 1.3 Pourquoi utiliser SASS en 2024 ?

La question est légitime : CSS moderne a des Custom Properties (variables CSS), `calc()`, et bientôt des fonctions natives. Voici ce que SASS apporte que CSS natif n'a pas encore :

| Fonctionnalité | SASS | CSS Natif (2024) |
|---|---|---|
| Variables avec logique | ✅ `$color: darken(#blue, 10%)` | ⚠️ `var(--color)` (statique) |
| Nesting | ✅ Mature | ⚠️ Expérimental (CSS Nesting L4) |
| Modules / imports | ✅ `@use`, `@forward` | ❌ |
| Mixins (blocs réutilisables) | ✅ | ❌ |
| Boucles et conditions | ✅ | ❌ |
| Fonctions custom | ✅ | Limité (`@property`) |
| Compilation = minification | ✅ | Nécessite outil séparé |

> [!tip] CSS Nesting natif
> CSS Nesting Level 4 est disponible dans Chrome 112+, Firefox 117+, Safari 16.5+. Pour les nouveaux projets ciblant des navigateurs récents, le nesting natif peut remplacer SASS sur ce point. Mais les modules, mixins et boucles restent exclusifs à SASS.

---

## 2. Installation et configuration

### 2.1 Installation via npm

```bash
# Initialiser un projet Node (si pas déjà fait)
npm init -y

# Installer SASS comme dépendance de développement
npm install --save-dev sass

# Vérifier l'installation
npx sass --version
# → 1.69.x compiled with dart2js 3.x.x
```

> [!info] Dart Sass vs Node Sass
> Il existe deux implémentations : **Dart Sass** (recommandé, activement maintenu) et **Node Sass** (déprécié depuis 2022, basé sur LibSass C++). Installez toujours `sass` (Dart Sass), jamais `node-sass`.

### 2.2 Compilation CLI de base

```bash
# Compiler un fichier unique
npx sass src/styles/main.scss dist/css/style.css

# Mode watch (recompile à chaque modification)
npx sass --watch src/styles/main.scss dist/css/style.css

# Watcher sur un dossier entier
npx sass --watch src/styles/:dist/css/

# Compilation compressée (production)
npx sass --style=compressed src/styles/main.scss dist/css/style.min.css

# Avec source maps
npx sass --source-map src/styles/main.scss dist/css/style.css
```

### 2.3 Scripts npm

Ajoutez ces scripts dans votre `package.json` pour ne pas retaper les commandes :

```json
{
  "name": "mon-projet",
  "version": "1.0.0",
  "scripts": {
    "sass:dev": "sass --watch src/styles/main.scss:dist/css/style.css --source-map",
    "sass:build": "sass src/styles/main.scss dist/css/style.min.css --style=compressed --no-source-map",
    "sass:check": "sass --dry-run src/styles/main.scss"
  },
  "devDependencies": {
    "sass": "^1.69.0"
  }
}
```

```bash
# Utilisation
npm run sass:dev    # développement avec watch
npm run sass:build  # production
```

### 2.4 Intégration Vite

Vite (l'outil de build moderne recommandé à Holberton) supporte SASS nativement :

```bash
# Dans un projet Vite existant
npm install --save-dev sass
```

```javascript
// vite.config.js — configuration optionnelle
import { defineConfig } from 'vite';

export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        // Fichier importé automatiquement dans tous les fichiers SCSS
        // Utile pour les variables globales et les mixins
        additionalData: `@use '@/styles/abstracts/variables' as *;`
      }
    },
    // Activer les source maps en développement
    devSourcemap: true
  }
});
```

```html
<!-- index.html — importer le SCSS directement -->
<link rel="stylesheet" href="/src/styles/main.scss">
```

Vite compile automatiquement le SCSS à la volée en développement et le minifie pour la production. Aucune configuration supplémentaire n'est nécessaire.

### 2.5 Intégration Webpack

```bash
npm install --save-dev sass sass-loader css-loader style-loader
```

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.s[ac]ss$/i, // match .sass et .scss
        use: [
          'style-loader',  // 3. Injecte CSS dans le DOM
          'css-loader',    // 2. Résout @import et url()
          'sass-loader',   // 1. Compile SCSS → CSS
        ],
      },
    ],
  },
};
```

```javascript
// Dans votre JavaScript (React, Vue, etc.)
import './styles/main.scss';
```

---

## 3. Variables SASS

### 3.1 Syntaxe et déclaration

Les variables SASS commencent par le symbole `$`. Elles peuvent contenir n'importe quelle valeur CSS valide, ainsi que des types SASS spécifiques.

```scss
// ============================================
// src/styles/_variables.scss
// ============================================

// --- Couleurs ---
$color-primary:    #3498db;
$color-secondary:  #2ecc71;
$color-danger:     #e74c3c;
$color-warning:    #f39c12;
$color-dark:       #2c3e50;
$color-light:      #ecf0f1;
$color-white:      #ffffff;

// --- Typographie ---
$font-family-base:   'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
$font-family-mono:   'JetBrains Mono', 'Fira Code', monospace;
$font-size-base:     1rem;       // 16px
$font-size-sm:       0.875rem;   // 14px
$font-size-lg:       1.125rem;   // 18px
$font-size-xl:       1.25rem;    // 20px
$font-size-h1:       2.5rem;     // 40px
$font-size-h2:       2rem;       // 32px
$font-size-h3:       1.75rem;    // 28px
$line-height-base:   1.6;

// --- Espacements (système 4px) ---
$spacing-1:  0.25rem;  // 4px
$spacing-2:  0.5rem;   // 8px
$spacing-3:  0.75rem;  // 12px
$spacing-4:  1rem;     // 16px
$spacing-5:  1.25rem;  // 20px
$spacing-6:  1.5rem;   // 24px
$spacing-8:  2rem;     // 32px
$spacing-10: 2.5rem;   // 40px
$spacing-12: 3rem;     // 48px
$spacing-16: 4rem;     // 64px

// --- Bordures ---
$border-radius-sm:  0.25rem;  // 4px
$border-radius:     0.5rem;   // 8px
$border-radius-lg:  1rem;     // 16px
$border-radius-xl:  1.5rem;   // 24px
$border-radius-full: 9999px;  // pill/cercle
$border-color:      rgba(0, 0, 0, 0.12);

// --- Ombres ---
$shadow-sm:   0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.08);
$shadow-md:   0 4px 6px rgba(0, 0, 0, 0.07), 0 2px 4px rgba(0, 0, 0, 0.06);
$shadow-lg:   0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);
$shadow-xl:   0 20px 25px rgba(0, 0, 0, 0.1), 0 10px 10px rgba(0, 0, 0, 0.04);

// --- Breakpoints ---
$breakpoint-sm:  576px;
$breakpoint-md:  768px;
$breakpoint-lg:  992px;
$breakpoint-xl:  1200px;
$breakpoint-xxl: 1400px;

// --- Z-index ---
$z-dropdown:  1000;
$z-sticky:    1020;
$z-fixed:     1030;
$z-modal:     1040;
$z-tooltip:   1060;
$z-toast:     1080;

// --- Transitions ---
$transition-fast:   0.15s ease;
$transition-base:   0.3s ease;
$transition-slow:   0.5s ease;
```

### 3.2 Portée des variables (scope)

```scss
// Variable globale
$color-primary: #3498db;

.card {
  // Variable locale — visible uniquement dans .card et ses enfants
  $card-padding: 1.5rem;
  $card-bg: white;

  padding: $card-padding;
  background: $card-bg;

  .card-header {
    // $card-padding est accessible ici (scope parent)
    padding: $card-padding * 0.5;
  }
}

// ERREUR — $card-padding n'existe pas à ce niveau
.footer {
  padding: $card-padding; // ❌ Undefined variable
}
```

### 3.3 Flag `!default` — valeurs par défaut

Le flag `!default` définit une valeur **seulement si la variable n'est pas déjà définie**. C'est le mécanisme de customisation des bibliothèques SASS (comme Bootstrap).

```scss
// _config.scss — valeurs par défaut d'une bibliothèque
$color-primary: #3498db !default;
$font-size-base: 1rem !default;
$border-radius: 0.5rem !default;

// Dans le fichier principal de l'utilisateur
// En important VOS variables AVANT la bibliothèque, vous surchargez les defaults
$color-primary: #e74c3c; // Votre couleur → écrase le !default
@use 'librairie'; // Le !default de la lib ne s'applique plus
```

### 3.4 SASS Variables vs CSS Custom Properties

C'est une question fréquente en entretien. Comprendre la différence est essentiel :

| Aspect | Variables SASS `$var` | CSS Custom Properties `--var` |
|---|---|---|
| Résolution | **Compilation** → disparaissent dans le CSS final | **Runtime** → présentes dans le CSS livré |
| Manipulation JS | ❌ Impossible | ✅ `element.style.setProperty('--color', 'red')` |
| Héritage DOM | ❌ Statique | ✅ Cascade et héritage CSS |
| Media queries | ✅ Valeur dynamique à la compilation | ❌ Valeur fixe (sauf re-déclaration) |
| Calculs | ✅ `darken($color, 10%)` | Limité à `calc()` |
| Theming dark mode | Compilation multiple | ✅ `[data-theme="dark"] { --bg: #000; }` |

```scss
// Stratégie hybride recommandée — le meilleur des deux mondes
// Variables SASS pour les calculs et la logique de compilation
$color-primary-raw: #3498db;
$color-primary-dark: darken($color-primary-raw, 10%);

// Export en Custom Properties pour la flexibilité runtime
:root {
  --color-primary:      #{$color-primary-raw};
  --color-primary-dark: #{$color-primary-dark};
  --font-size-base:     #{$font-size-base};
}

// Utilisation dans les composants
.btn {
  // Utilise la Custom Property → modifiable par JS et dark mode
  background-color: var(--color-primary);
  
  &:hover {
    background-color: var(--color-primary-dark);
  }
}
```

> [!tip] Interpolation `#{}`
> Pour injecter une variable SASS dans un contexte qui attend une chaîne de caractères (comme la valeur d'une Custom Property), utilisez `#{}`. Sans interpolation, `$variable` serait lu comme une valeur SASS, pas comme une valeur CSS.

---

## 4. Nesting — Règles imbriquées

### 4.1 Principe du nesting

Le nesting permet d'écrire des règles CSS dans le corps d'autres règles, reflétant la structure HTML. SASS "déplie" ces règles à la compilation.

```scss
// SCSS — ce que vous écrivez
nav {
  background: $color-dark;
  padding: $spacing-4;

  ul {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    gap: $spacing-4;
  }

  li {
    // Pas de style par défaut
  }

  a {
    color: $color-white;
    text-decoration: none;
    padding: $spacing-2 $spacing-4;
    border-radius: $border-radius;
    transition: background $transition-fast;

    &:hover {
      background: rgba(255, 255, 255, 0.1);
    }

    &.active {
      background: $color-primary;
    }
  }
}
```

```css
/* CSS compilé — ce que le navigateur reçoit */
nav {
  background: #2c3e50;
  padding: 1rem;
}

nav ul {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  gap: 1rem;
}

nav a {
  color: #ffffff;
  text-decoration: none;
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  transition: background 0.15s ease;
}

nav a:hover {
  background: rgba(255, 255, 255, 0.1);
}

nav a.active {
  background: #3498db;
}
```

### 4.2 Le sélecteur parent `&`

Le `&` est remplacé par le sélecteur parent au moment de la compilation. C'est l'un des outils les plus puissants de SASS.

```scss
.btn {
  display: inline-flex;
  align-items: center;
  padding: $spacing-2 $spacing-6;
  border: none;
  border-radius: $border-radius;
  font-size: $font-size-base;
  font-weight: 600;
  cursor: pointer;
  transition: all $transition-fast;

  // &:pseudo-classe — états interactifs
  &:hover   { opacity: 0.9; transform: translateY(-1px); }
  &:active  { transform: translateY(0); }
  &:focus   { outline: 2px solid $color-primary; outline-offset: 2px; }
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    pointer-events: none;
  }

  // &.modificateur — BEM ou variantes
  &.btn-primary   { background: $color-primary; color: white; }
  &.btn-secondary { background: $color-secondary; color: white; }
  &.btn-danger    { background: $color-danger; color: white; }
  &.btn-outline {
    background: transparent;
    border: 2px solid currentColor;
    color: $color-primary;

    &:hover {
      background: $color-primary;
      color: white;
    }
  }

  // &--modificateur BEM
  &--sm { padding: $spacing-1 $spacing-3; font-size: $font-size-sm; }
  &--lg { padding: $spacing-3 $spacing-8; font-size: $font-size-lg; }

  // Contexte parent — "quand .btn est dans .card"
  .card & {
    width: 100%; // Bouton pleine largeur dans une carte
  }

  // Combinaison parent + pseudo
  .dark-theme & {
    &:hover { background: lighten($color-primary, 10%); }
  }
}
```

```css
/* Compilation de .btn--sm et .btn dans .card */
.btn--sm {
  padding: 0.25rem 0.75rem;
  font-size: 0.875rem;
}

.card .btn {
  width: 100%;
}
```

### 4.3 Nesting des media queries

SASS permet d'écrire les media queries **à l'intérieur** du sélecteur, ce qui améliore enormément la lisibilité et la maintenabilité.

```scss
.hero {
  padding: $spacing-8;
  font-size: $font-size-xl;

  // Media query imbriquée — reste dans le contexte .hero
  @media (max-width: $breakpoint-md) {
    padding: $spacing-4;
    font-size: $font-size-lg;
  }

  @media (max-width: $breakpoint-sm) {
    padding: $spacing-3;
    font-size: $font-size-base;
  }

  &__title {
    font-size: $font-size-h1;
    line-height: 1.2;

    @media (max-width: $breakpoint-md) {
      font-size: $font-size-h2;
    }
  }
}
```

```css
/* CSS compilé — les media queries sont extraites et remontées */
.hero {
  padding: 2rem;
  font-size: 1.25rem;
}

@media (max-width: 768px) {
  .hero {
    padding: 1rem;
    font-size: 1.125rem;
  }
}

@media (max-width: 576px) {
  .hero {
    padding: 0.75rem;
    font-size: 1rem;
  }
}

.hero__title {
  font-size: 2.5rem;
  line-height: 1.2;
}

@media (max-width: 768px) {
  .hero__title {
    font-size: 2rem;
  }
}
```

### 4.4 Bonnes pratiques : règle des 3 niveaux

> [!warning] Règle des 3 niveaux maximum
> Un nesting au-delà de 3 niveaux génère des sélecteurs CSS trop spécifiques, difficiles à surcharger et révélateurs d'un HTML mal structuré. Si vous vous retrouvez à 4+ niveaux, revenez au niveau 0 et utilisez une nouvelle classe.

```scss
// ❌ Nesting trop profond — mauvaise pratique
.sidebar {
  .sidebar-section {
    .sidebar-section-header {
      .sidebar-section-header-icon {
        .icon-svg { // 5 niveaux — horreur
          fill: $color-primary;
        }
      }
    }
  }
}
/* Compile vers : .sidebar .sidebar-section .sidebar-section-header
   .sidebar-section-header-icon .icon-svg { ... } — trop spécifique ! */

// ✅ Bonne pratique — reprendre depuis le niveau 0
.sidebar {
  // Niveau 1 — OK
}

.sidebar-section {
  // Niveau 1 — reprend au début
  
  &__header {
    // Niveau 2 — BEM, reste lisible
  }
}

.sidebar-icon {
  // Classe autonome, réutilisable
  fill: $color-primary;
}
```

```scss
// ✅ Exemple complet bien structuré — carte produit
.product-card {
  // Niveau 1 : l'élément racine
  display: flex;
  flex-direction: column;
  border-radius: $border-radius-lg;
  overflow: hidden;
  box-shadow: $shadow-md;
  transition: transform $transition-base, box-shadow $transition-base;

  &:hover {
    // Niveau 2 : état — acceptable
    transform: translateY(-4px);
    box-shadow: $shadow-xl;
  }

  &__image {
    // Niveau 2 : élément BEM
    width: 100%;
    aspect-ratio: 4/3;
    object-fit: cover;
  }

  &__body {
    // Niveau 2 : élément BEM
    padding: $spacing-4;
    flex: 1;

    @media (max-width: $breakpoint-sm) {
      // Niveau 3 : media query dans un élément — limite à respecter
      padding: $spacing-3;
    }
  }

  &__title {
    // Niveau 2 — nouveau & depuis la racine
    font-size: $font-size-lg;
    font-weight: 700;
    margin-bottom: $spacing-2;
  }

  &--featured {
    // Niveau 2 : modificateur BEM
    border: 2px solid $color-primary;
    
    .product-card__title {
      // Niveau 3 : dernier niveau autorisé
      color: $color-primary;
    }
  }
}
```

---

## 5. Partials et organisation en modules

### 5.1 Partials — fichiers fragmentés

Un **partial** est un fichier SCSS dont le nom commence par un underscore `_`. SASS ne compilera pas directement ce fichier en CSS ; il est destiné à être importé dans d'autres fichiers.

```
```
src/styles/
├── main.scss              ← fichier d'entrée (compilé)
├── abstracts/
│   ├── _variables.scss    ← partial (NOT compilé directement)
│   ├── _mixins.scss       ← partial
│   └── _functions.scss    ← partial
├── base/
│   ├── _reset.scss        ← partial
│   └── _typography.scss   ← partial
└── components/
    ├── _button.scss       ← partial
    └── _card.scss         ← partial
```
```

### 5.2 `@use` — le système de modules moderne

`@use` est le remplaçant moderne de `@import` (déprécié depuis Dart Sass 1.23). Il charge un fichier SASS comme un **module** avec un espace de noms.

```scss
// src/styles/abstracts/_variables.scss
$color-primary: #3498db;
$spacing-4: 1rem;
$font-size-base: 1rem;
```

```scss
// src/styles/components/_button.scss

// @use avec namespace automatique (nom du fichier)
@use '../abstracts/variables';

.btn {
  // Accès via namespace "variables."
  color: variables.$color-primary;
  padding: variables.$spacing-4;
  font-size: variables.$font-size-base;
}
```

```scss
// Alias pour raccourcir
@use '../abstracts/variables' as vars;

.btn {
  color: vars.$color-primary; // Alias "vars" au lieu de "variables"
}
```

```scss
// Wildcard — importer dans le namespace global (déconseillé sauf dans main.scss)
@use '../abstracts/variables' as *;

.btn {
  color: $color-primary; // Pas de namespace — risque de collisions
}
```

### 5.3 `@forward` — réexporter des modules

`@forward` permet de créer un **fichier index** qui réexporte plusieurs modules, simplifiant les imports dans les fichiers consommateurs.

```scss
// src/styles/abstracts/_index.scss
// Ce fichier réexporte tout le dossier abstracts en un seul point d'entrée

@forward 'variables';
@forward 'mixins';
@forward 'functions';
```

```scss
// src/styles/components/_button.scss
// Avec @forward, un seul import suffit pour tout abstracts/

@use '../abstracts' as abstracts; // Utilise abstracts/_index.scss

.btn {
  color: abstracts.$color-primary;
  // Accès aux mixins aussi : @include abstracts.flex-center;
}
```

```scss
// @forward avec configuration — personnaliser les variables d'un module
// src/styles/abstracts/_index.scss

@forward 'variables' with (
  $color-primary: #e74c3c !default,  // Surcharger la valeur par défaut
  $border-radius: 0.25rem !default
);
@forward 'mixins';
```

```scss
// @forward avec préfixe — éviter les collisions de noms
@forward 'button-variables' as btn-*;
// Toutes les variables de button-variables seront préfixées : $btn-color, $btn-size...
```

### 5.4 Différence entre `@use` et `@import` (déprécié)

> [!warning] `@import` est déprécié
> `@import` sera supprimé dans une future version de Dart Sass. N'utilisez jamais `@import` dans du nouveau code. Les différences clés :

| Aspect | `@import` (déprécié) | `@use` (moderne) |
|---|---|---|
| Espaces de noms | Global — tout dans le même scope | Module — chaque fichier a son namespace |
| Exécution | Peut être importé plusieurs fois | Exécuté **une seule fois** |
| Visibilité | Tout est global | Variables privées avec `$-` |
| Performance | Moins efficace | Optimisé par le compilateur |

```scss
// Variables privées — préfixées par -
// Ne sont pas accessibles via @use depuis l'extérieur
$-internal-spacing: 0.75rem; // Privée
$public-spacing: 1rem;       // Publique

// Fonction privée
@function -calculate-ratio($value) {
  @return $value * 1.618;
}
```

---

## 6. Mixins

### 6.1 Définition et usage de base

Un mixin est un bloc de CSS réutilisable, similaire à une fonction qui produit des déclarations CSS.

```scss
// src/styles/abstracts/_mixins.scss

// Mixin simple — sans paramètres
@mixin reset-list {
  list-style: none;
  margin: 0;
  padding: 0;
}

// Utilisation
nav ul {
  @include reset-list;
  display: flex;
  gap: 1rem;
}

// CSS compilé
// nav ul {
//   list-style: none;
//   margin: 0;
//   padding: 0;
//   display: flex;
//   gap: 1rem;
// }
```

### 6.2 Mixins avec paramètres

```scss
// Mixin avec paramètres obligatoires
@mixin flex-layout($direction, $justify, $align) {
  display: flex;
  flex-direction: $direction;
  justify-content: $justify;
  align-items: $align;
}

// Mixin avec valeurs par défaut
@mixin flex-center($direction: row, $gap: 0) {
  display: flex;
  flex-direction: $direction;
  justify-content: center;
  align-items: center;
  gap: $gap;
}

// Utilisations
.hero {
  @include flex-center(column, 2rem);
}

.nav {
  @include flex-center; // Utilise les valeurs par défaut
}

.sidebar {
  @include flex-layout(column, flex-start, stretch);
}
```

### 6.3 Mixins avancés — media queries responsive

```scss
// Mixin breakpoints — le mixin le plus utilisé sur tout projet
@mixin respond-to($breakpoint) {
  @if $breakpoint == 'sm' {
    @media (max-width: 576px) { @content; }
  } @else if $breakpoint == 'md' {
    @media (max-width: 768px) { @content; }
  } @else if $breakpoint == 'lg' {
    @media (max-width: 992px) { @content; }
  } @else if $breakpoint == 'xl' {
    @media (max-width: 1200px) { @content; }
  } @else {
    @warn "Breakpoint `#{$breakpoint}` non reconnu.";
  }
}

// @content — le mixin accepte un bloc de contenu entre {} 
.hero {
  font-size: 3rem;
  padding: 4rem;

  @include respond-to('md') {
    font-size: 2rem;
    padding: 2rem;
  }

  @include respond-to('sm') {
    font-size: 1.5rem;
    padding: 1rem;
  }
}
```

```scss
// Version avec map (plus maintenable)
$breakpoints: (
  'sm':  576px,
  'md':  768px,
  'lg':  992px,
  'xl':  1200px,
  'xxl': 1400px
);

@mixin respond-to($breakpoint) {
  $value: map.get($breakpoints, $breakpoint);

  @if $value == null {
    @error "Breakpoint `#{$breakpoint}` introuvable dans $breakpoints.";
  }

  @media (max-width: $value) {
    @content;
  }
}
```

### 6.4 Bibliothèque complète de mixins pratiques

```scss
// src/styles/abstracts/_mixins.scss
@use 'sass:math';
@use 'variables' as vars;

// ——— Typographie ———

@mixin text-truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

@mixin text-truncate-multiline($lines: 2) {
  display: -webkit-box;
  -webkit-line-clamp: $lines;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

@mixin font-smoothing {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

// ——— Positionnement ———

@mixin absolute-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

@mixin absolute-fill {
  position: absolute;
  inset: 0; // top: 0; right: 0; bottom: 0; left: 0;
}

@mixin sticky-top($offset: 0) {
  position: sticky;
  top: $offset;
  z-index: vars.$z-sticky;
}

// ——— Visuels ———

@mixin card-base {
  background: white;
  border-radius: vars.$border-radius-lg;
  box-shadow: vars.$shadow-md;
  overflow: hidden;
}

@mixin hover-lift($distance: 4px, $shadow: vars.$shadow-xl) {
  transition:
    transform vars.$transition-base,
    box-shadow vars.$transition-base;

  &:hover {
    transform: translateY(-#{$distance});
    box-shadow: $shadow;
  }
}

@mixin gradient-text($from, $to, $direction: 135deg) {
  background: linear-gradient($direction, $from, $to);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

// ——— Accessibilité ———

@mixin visually-hidden {
  // Masqué visuellement mais lisible par les lecteurs d'écran
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

@mixin focus-visible-ring($color: vars.$color-primary, $offset: 2px) {
  &:focus-visible {
    outline: 2px solid $color;
    outline-offset: $offset;
  }

  &:focus:not(:focus-visible) {
    outline: none;
  }
}

// ——— Utilitaires ———

@mixin clearfix {
  &::after {
    content: '';
    display: table;
    clear: both;
  }
}

@mixin aspect-ratio($width: 16, $height: 9) {
  aspect-ratio: math.div($width, $height);
  
  // Fallback pour navigateurs anciens
  @supports not (aspect-ratio: 1) {
    &::before {
      content: '';
      display: block;
      padding-top: math.div($height, $width) * 100%;
    }
  }
}
```

### 6.5 Mixins avec `@content` — blocs dynamiques

`@content` permet de passer un bloc CSS entier à un mixin, le rendant extrêmement flexible.

```scss
// Mixin pour les animations
@mixin keyframes($name) {
  @keyframes #{$name} {
    @content;
  }
}

// Utilisation
@include keyframes(fade-in) {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

// Mixin pour dark mode
@mixin dark-mode {
  @media (prefers-color-scheme: dark) {
    @content;
  }

  [data-theme="dark"] & {
    @content;
  }
}

// Utilisation
.card {
  background: white;
  color: black;

  @include dark-mode {
    background: #1e1e2e;
    color: #cdd6f4;
  }
}
```

---

## 7. Fonctions SASS

### 7.1 Définir des fonctions avec `@function`

Contrairement aux mixins qui produisent des **déclarations CSS**, les fonctions retournent une **valeur** utilisable dans une propriété.

```scss
@use 'sass:math';
@use 'sass:color';

// Convertir px en rem
@function px-to-rem($px, $base: 16) {
  @return math.div($px, $base) * 1rem;
}

// Utilisation
.hero {
  font-size: px-to-rem(48);  // → 3rem
  padding: px-to-rem(32);    // → 2rem
  border-radius: px-to-rem(8); // → 0.5rem
}

// Fonction de contraste — choisit noir ou blanc selon le fond
@function contrast-color($background) {
  $lightness: color.lightness($background);
  @if $lightness > 50 {
    @return #000000;
  } @else {
    @return #ffffff;
  }
}

// Utilisation
$btn-bg: #3498db;
.btn {
  background: $btn-bg;
  color: contrast-color($btn-bg); // → #ffffff (fond foncé)
}

// Fonction d'espacement — système cohérent
@function spacing($multiplier) {
  $base: 0.25rem; // 4px
  @return $base * $multiplier;
}

.card {
  padding: spacing(4);      // 1rem
  margin-bottom: spacing(8); // 2rem
  gap: spacing(2);           // 0.5rem
}
```

### 7.2 Fonctions du module `sass:math`

```scss
@use 'sass:math';

// math.div — division (remplace l'opérateur / déprécié)
.container {
  width: math.div(100%, 3); // 33.3333%
}

// math.round, math.ceil, math.floor
$value: 1.7rem;
.element {
  font-size: math.round($value);  // 2rem
  padding: math.floor($value);    // 1rem
  margin: math.ceil($value);      // 2rem
}

// math.min, math.max, math.clamp (SASS 1.31+)
.responsive-text {
  font-size: math.clamp(1rem, 2vw + 0.5rem, 2rem);
}

// math.pow, math.sqrt, math.log, math.abs...
$golden-ratio: 1.618;
@function golden-size($base) {
  @return $base * $golden-ratio;
}
```

### 7.3 Fonctions du module `sass:color`

```scss
@use 'sass:color';

$primary: #3498db;

// Ajuster une couleur
.btn-hover {
  background: color.adjust($primary, $lightness: -10%);  // plus sombre
}

.badge-light {
  background: color.adjust($primary, $lightness: 40%);   // plus clair
  color: color.adjust($primary, $lightness: -30%);       // texte encore plus sombre
}

// Mélanger deux couleurs
$mixed: color.mix($primary, white, 80%); // 80% primary, 20% white

// Changer la teinte/saturation
$analogous: color.adjust($primary, $hue: 30deg);     // +30° sur la roue chromatique
$desaturated: color.adjust($primary, $saturation: -20%); // Moins saturé

// Obtenir des informations
$hue: color.hue($primary);           // → 204deg
$sat: color.saturation($primary);    // → 70%
$light: color.lightness($primary);   // → 54%

// Rendre transparent
$semi: color.adjust($primary, $alpha: -0.5);  // 50% opaque
// ou
$semi: rgba($primary, 0.5);                   // équivalent

// Ancien API (toujours fonctionnel mais moins précis)
// darken($primary, 10%)   → color.adjust($primary, $lightness: -10%)
// lighten($primary, 10%)  → color.adjust($primary, $lightness: 10%)
// transparentize($c, 0.5) → color.adjust($c, $alpha: -0.5)
```

### 7.4 Fonctions du module `sass:string`

```scss
@use 'sass:string';

// Concaténation
$prefix: 'btn';
$modifier: 'primary';
$class: $prefix + '-' + $modifier; // "btn-primary"

// string.length, string.slice, string.index, string.to-upper-case
$name: 'hello-world';
$upper: string.to-upper-case($name); // "HELLO-WORLD"
$length: string.length($name);       // 11
$sub: string.slice($name, 1, 5);    // "hello"
```

---

## 8. Extends et Placeholders

### 8.1 `@extend` — partager des styles

`@extend` permet à un sélecteur d'hériter des styles d'un autre. SASS regroupera les sélecteurs dans le CSS compilé.

```scss
// Classe de base
.message {
  padding: $spacing-3 $spacing-4;
  border-radius: $border-radius;
  font-size: $font-size-sm;
  font-weight: 500;
}

// Extensions — héritent de .message
.message-success {
  @extend .message;
  background: #d4edda;
  color: #155724;
  border-left: 4px solid $color-secondary;
}

.message-error {
  @extend .message;
  background: #f8d7da;
  color: #721c24;
  border-left: 4px solid $color-danger;
}

.message-warning {
  @extend .message;
  background: #fff3cd;
  color: #856404;
  border-left: 4px solid $color-warning;
}
```

```css
/* CSS compilé — SASS regroupe les sélecteurs */
.message, .message-success, .message-error, .message-warning {
  padding: 0.75rem 1rem;
  border-radius: 0.5rem;
  font-size: 0.875rem;
  font-weight: 500;
}

.message-success {
  background: #d4edda;
  color: #155724;
  border-left: 4px solid #2ecc71;
}
/* ... */
```

### 8.2 Placeholders `%` — étendre sans classe fantôme

Un placeholder `%` définit un ensemble de styles qui n'est **jamais compilé** s'il n'est pas étendu. Il évite de générer une classe CSS inutile dans le HTML.

```scss
// ✅ Placeholder — n'apparaît PAS dans le CSS final si non étendu
%flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

%card-base {
  background: white;
  border-radius: $border-radius-lg;
  box-shadow: $shadow-md;
  overflow: hidden;
}

%visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
}

// Utilisation — les styles sont injectés là où on les étend
.hero {
  @extend %flex-center;
  flex-direction: column;
  min-height: 100vh;
}

.modal-overlay {
  @extend %flex-center;
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
}

.product-card {
  @extend %card-base;
  // Styles spécifiques...
}

.sr-only {
  @extend %visually-hidden;
}
```

### 8.3 Mixin vs Extend vs Placeholder — quand utiliser quoi ?

C'est l'une des questions les plus fréquentes. Voici le tableau de décision :

| Critère | Mixin `@include` | Extend `@extend` | Placeholder `%` + `@extend` |
|---|---|---|---|
| Paramètres | ✅ Oui | ❌ Non | ❌ Non |
| CSS généré | Une copie par `@include` | Sélecteurs regroupés | Sélecteurs regroupés (sans classe fantôme) |
| Gonflement CSS | Possible | Non | Non |
| Utilisation dans `@media` | ✅ Oui | ❌ Restrictions | ❌ Restrictions |
| **Cas d'usage idéal** | Comportements paramétrables, media queries | **Éviter** en général | Styles partagés sans paramètre |

```scss
// ——— Règle pratique ———

// 1. Vous avez besoin de paramètres → MIXIN
@mixin button-variant($bg, $color: white) {
  background: $bg;
  color: $color;
  &:hover { background: darken($bg, 8%); }
}

// 2. Styles identiques, pas de paramètres, pas de @media → PLACEHOLDER
%btn-reset {
  border: none;
  cursor: pointer;
  font-family: inherit;
}

// 3. @extend depuis une classe existante → ÉVITER
// (risque de sélecteurs non désirés dans le CSS)
```

> [!warning] Limitation de @extend
> `@extend` ne peut pas être utilisé à l'intérieur d'une `@media` query. Si vous avez besoin de partager des styles dans un contexte responsive, utilisez un mixin avec `@content` ou un placeholder défini en dehors des media queries.

---

## 9. Boucles SASS

### 9.1 `@each` — itérer sur une liste

```scss
// Liste simple
$colors: red, green, blue, yellow;

@each $color in $colors {
  .text-#{$color} {
    color: $color;
  }
}
// Génère : .text-red, .text-green, .text-blue, .text-yellow

// Liste de paires
$icons: (
  'home' '\f015',
  'user' '\f007',
  'bell' '\f0f3',
  'cart' '\f07a'
);

@each $name, $unicode in $icons {
  .icon-#{$name}::before {
    content: $unicode;
    font-family: 'FontAwesome';
  }
}

// Map (dictionnaire) — voir section 11 pour les maps
$theme-colors: (
  'primary':   #3498db,
  'secondary': #2ecc71,
  'danger':    #e74c3c,
  'warning':   #f39c12
);

@each $name, $color in $theme-colors {
  .btn-#{$name} {
    background: $color;
    color: white;

    &:hover {
      background: color.adjust($color, $lightness: -8%);
    }
  }

  .badge-#{$name} {
    background: color.adjust($color, $lightness: 35%);
    color: color.adjust($color, $lightness: -20%);
  }

  .text-#{$name} {
    color: $color;
  }
}
```

### 9.2 `@for` — boucle numérique

```scss
@use 'sass:math';

// @for ... through — inclut la borne supérieure
@for $i from 1 through 12 {
  .col-#{$i} {
    width: math.div(100%, 12) * $i;
  }
}
// Génère : .col-1 (8.33%), .col-2 (16.66%), ... .col-12 (100%)

// @for ... to — exclut la borne supérieure
@for $i from 1 to 5 {
  .opacity-#{$i * 25} {
    opacity: math.div($i * 25, 100);
  }
}
// Génère : .opacity-25, .opacity-50, .opacity-75 (pas .opacity-100)

// Système de spacings utilitaires
$spacings: (1, 2, 3, 4, 5, 6, 8, 10, 12, 16);

@each $size in $spacings {
  .mt-#{$size} { margin-top:    $size * 0.25rem; }
  .mb-#{$size} { margin-bottom: $size * 0.25rem; }
  .ml-#{$size} { margin-left:   $size * 0.25rem; }
  .mr-#{$size} { margin-right:  $size * 0.25rem; }
  .mx-#{$size} { margin-left:   $size * 0.25rem; margin-right:  $size * 0.25rem; }
  .my-#{$size} { margin-top:    $size * 0.25rem; margin-bottom: $size * 0.25rem; }
  .m-#{$size}  { margin:        $size * 0.25rem; }

  .pt-#{$size} { padding-top:    $size * 0.25rem; }
  .pb-#{$size} { padding-bottom: $size * 0.25rem; }
  .pl-#{$size} { padding-left:   $size * 0.25rem; }
  .pr-#{$size} { padding-right:  $size * 0.25rem; }
  .px-#{$size} { padding-left:   $size * 0.25rem; padding-right:  $size * 0.25rem; }
  .py-#{$size} { padding-top:    $size * 0.25rem; padding-bottom: $size * 0.25rem; }
  .p-#{$size}  { padding:        $size * 0.25rem; }
}
```

### 9.3 `@while` — boucle conditionnelle

```scss
// @while est rare en pratique — @for et @each couvrent la plupart des cas
// Utilisation typique : générations non-linéaires

$i: 1;
@while $i <= 8 {
  .z-#{$i * 10} {
    z-index: $i * 10;
  }
  $i: $i + 1;
}

// Générer une échelle typographique modulaire (ratio)
$base-size: 1rem;
$ratio: 1.25; // "Major Third"
$steps: 6;
$current: $base-size;
$step: 1;

@while $step <= $steps {
  .text-scale-#{$step} {
    font-size: $current;
  }
  $current: $current * $ratio;
  $step: $step + 1;
}
// text-scale-1: 1rem, text-scale-2: 1.25rem, text-scale-3: 1.5625rem...
```

---

## 10. Conditions — `@if`, `@else if`, `@else`

```scss
// Conditions simples
@mixin button-size($size) {
  @if $size == 'sm' {
    padding: $spacing-1 $spacing-3;
    font-size: $font-size-sm;
  } @else if $size == 'md' {
    padding: $spacing-2 $spacing-4;
    font-size: $font-size-base;
  } @else if $size == 'lg' {
    padding: $spacing-3 $spacing-6;
    font-size: $font-size-lg;
  } @else {
    @error "Taille `#{$size}` invalide. Valeurs acceptées : sm, md, lg.";
  }
}

// Opérateurs de comparaison
// == égal    != différent
// <  inférieur  >  supérieur
// <= inférieur ou égal  >= supérieur ou égal
// and  or  not

@mixin responsive-grid($columns, $gap: 1rem) {
  display: grid;
  gap: $gap;

  @if $columns >= 4 and $columns <= 12 {
    grid-template-columns: repeat($columns, 1fr);
  } @else if $columns < 4 {
    grid-template-columns: repeat($columns, 1fr);
    @warn "Grille avec #{$columns} colonnes — peu commun.";
  } @else {
    @error "Trop de colonnes : #{$columns}. Maximum 12.";
  }
}

// @if avec type checking
@mixin set-color($color) {
  @if type-of($color) != 'color' {
    @error "#{$color} n'est pas une couleur valide.";
  }
  color: $color;
}

// Conditions dans les fonctions
@function clamp-value($value, $min, $max) {
  @if $value < $min {
    @return $min;
  } @else if $value > $max {
    @return $max;
  } @else {
    @return $value;
  }
}
```

### 10.1 `@warn` et `@error` — messages de débogage

```scss
// @warn — affiche un avertissement mais continue la compilation
@mixin deprecated-mixin() {
  @warn "Ce mixin est déprécié. Utilisez `new-mixin()` à la place.";
  // ... code legacy
}

// @error — stoppe la compilation avec un message
@function get-color($name) {
  $available: ('primary', 'secondary', 'danger');
  @if not index($available, $name) {
    @error "Couleur `#{$name}` inconnue. Disponibles : #{$available}";
  }
  // ... retourner la couleur
}

// @debug — affiche une valeur pendant le développement (supprimer avant prod)
$test-value: 1.5rem * 2;
@debug "Valeur calculée : #{$test-value}"; // Affiche dans le terminal
```

---

## 11. Maps SASS — dictionnaires

### 11.1 Création et accès

```scss
@use 'sass:map';

// Déclaration d'une map
$theme-colors: (
  'primary':   #3498db,
  'secondary': #2ecc71,
  'success':   #27ae60,
  'danger':    #e74c3c,
  'warning':   #f39c12,
  'info':      #17a2b8,
  'light':     #f8f9fa,
  'dark':      #343a40
);

// Accès à une valeur
.btn-primary {
  background: map.get($theme-colors, 'primary');
}

// Vérifier l'existence d'une clé
@if map.has-key($theme-colors, 'primary') {
  .debug { content: "primary existe"; }
}

// Fusionner deux maps
$extra-colors: ('brand': #ff6b6b, 'accent': #ffd93d);
$all-colors: map.merge($theme-colors, $extra-colors);

// Obtenir toutes les clés / valeurs
$keys:   map.keys($theme-colors);    // ('primary', 'secondary', ...)
$values: map.values($theme-colors);  // (#3498db, #2ecc71, ...)
```

### 11.2 Maps imbriquées — configurations complexes

```scss
// Map de configuration d'un système de design
$typography: (
  'h1': (
    'size':        2.5rem,
    'weight':      800,
    'line-height': 1.2,
    'letter-spacing': -0.02em
  ),
  'h2': (
    'size':        2rem,
    'weight':      700,
    'line-height': 1.25,
    'letter-spacing': -0.01em
  ),
  'h3': (
    'size':        1.75rem,
    'weight':      600,
    'line-height': 1.3,
    'letter-spacing': 0
  ),
  'body': (
    'size':        1rem,
    'weight':      400,
    'line-height': 1.6,
    'letter-spacing': 0
  )
);

// Accès à une map imbriquée
@function get-typography($tag, $property) {
  $tag-config: map.get($typography, $tag);
  @if $tag-config == null {
    @error "Tag `#{$tag}` introuvable dans $typography.";
  }
  @return map.get($tag-config, $property);
}

// Générer les styles typographiques depuis la map
@each $tag, $config in $typography {
  #{$tag} {
    font-size:      map.get($config, 'size');
    font-weight:    map.get($config, 'weight');
    line-height:    map.get($config, 'line-height');
    letter-spacing: map.get($config, 'letter-spacing');
  }
}
```

### 11.3 Maps de breakpoints — pattern professionnel

```scss
@use 'sass:map';

// Map centralisée des breakpoints
$grid: (
  'columns':    12,
  'gutter':     1.5rem,
  'breakpoints': (
    'xs':  0,
    'sm':  576px,
    'md':  768px,
    'lg':  992px,
    'xl':  1200px,
    'xxl': 1400px
  ),
  'containers': (
    'sm':  540px,
    'md':  720px,
    'lg':  960px,
    'xl':  1140px,
    'xxl': 1320px
  )
);

// Fonction d'accès au breakpoint
@function breakpoint($name) {
  $bps: map.get($grid, 'breakpoints');
  @return map.get($bps, $name);
}

// Mixin responsive utilisant la map
@mixin media-up($name) {
  @media (min-width: breakpoint($name)) {
    @content;
  }
}

@mixin media-down($name) {
  @media (max-width: breakpoint($name) - 0.02px) {
    @content;
  }
}

// Utilisation élégante
.hero-title {
  font-size: 1.5rem;

  @include media-up('md') {
    font-size: 2rem;
  }

  @include media-up('xl') {
    font-size: 3rem;
  }
}
```

---

## 12. Architecture SCSS — Pattern 7-1

### 12.1 Présentation de l'architecture 7-1

Le pattern **7-1** est la convention d'organisation la plus répandue pour les projets SCSS de taille professionnelle. Son nom vient de **7 dossiers, 1 fichier d'entrée**.

```
```
src/styles/
│
├── abstracts/          # Pas de CSS généré directement
│   ├── _index.scss     # @forward de tout le dossier
│   ├── _variables.scss # Variables globales
│   ├── _mixins.scss    # Mixins
│   ├── _functions.scss # Fonctions
│   └── _placeholders.scss # %placeholders
│
├── base/               # Styles de base, reset, typographie
│   ├── _index.scss
│   ├── _reset.scss     # CSS reset / normalize
│   ├── _typography.scss # Règles typographiques globales
│   └── _accessibility.scss
│
├── components/         # Composants UI réutilisables
│   ├── _index.scss
│   ├── _button.scss
│   ├── _card.scss
│   ├── _modal.scss
│   ├── _form.scss
│   ├── _navbar.scss
│   ├── _badge.scss
│   └── _alert.scss
│
├── layout/             # Structure de la page
│   ├── _index.scss
│   ├── _header.scss
│   ├── _footer.scss
│   ├── _sidebar.scss
│   ├── _grid.scss
│   └── _container.scss
│
├── pages/              # Styles spécifiques à chaque page
│   ├── _home.scss
│   ├── _about.scss
│   ├── _contact.scss
│   └── _blog.scss
│
├── themes/             # Thèmes visuels (dark mode, marques)
│   ├── _dark.scss
│   └── _light.scss
│
├── vendors/            # CSS de bibliothèques tierces
│   ├── _bootstrap-custom.scss
│   └── _swiper.scss
│
└── main.scss           ← SEUL fichier compilé directement
```
```

### 12.2 Fichier d'entrée `main.scss`

```scss
// src/styles/main.scss
// ============================================================
// ORDRE CRITIQUE : abstracts → vendors → base → layout →
//                  components → pages → themes
// ============================================================

// 1. Abstracts — doit être en premier (ne génère aucun CSS)
@use 'abstracts' as *;
// ou, si abstracts/_index.scss existe avec @forward :
// (le as * importe dans le scope global pour les autres fichiers)

// 2. Vendors — reset avant tout style
@use 'vendors/bootstrap-custom';

// 3. Base — reset et styles fondamentaux
@use 'base/reset';
@use 'base/typography';
@use 'base/accessibility';

// 4. Layout — structure avant composants
@use 'layout/container';
@use 'layout/grid';
@use 'layout/header';
@use 'layout/footer';
@use 'layout/sidebar';

// 5. Components — composants réutilisables
@use 'components/button';
@use 'components/card';
@use 'components/form';
@use 'components/modal';
@use 'components/navbar';
@use 'components/badge';
@use 'components/alert';

// 6. Pages — surcharges spécifiques aux pages
@use 'pages/home';
@use 'pages/about';
@use 'pages/blog';

// 7. Themes — en dernier pour surcharger
@use 'themes/dark';
```

### 12.3 Exemple de composant complet — `_button.scss`

```scss
// src/styles/components/_button.scss
@use '../abstracts' as *;
@use 'sass:color';

// ==============================
// Variables locales au composant
// ==============================
$btn-padding-y:      $spacing-2;
$btn-padding-x:      $spacing-6;
$btn-font-size:      $font-size-base;
$btn-font-weight:    600;
$btn-border-radius:  $border-radius;
$btn-transition:     all #{$transition-fast};
$btn-disabled-opacity: 0.6;

// ==============================
// Base du bouton
// ==============================
.btn {
  // Reset et base
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: $spacing-2;

  // Typographie
  font-family: $font-family-base;
  font-size: $btn-font-size;
  font-weight: $btn-font-weight;
  line-height: 1.5;
  text-align: center;
  text-decoration: none;
  white-space: nowrap;

  // Apparence
  padding: $btn-padding-y $btn-padding-x;
  border: 2px solid transparent;
  border-radius: $btn-border-radius;
  cursor: pointer;
  user-select: none;

  // Animation
  transition: $btn-transition;
  vertical-align: middle;

  // États
  @include focus-visible-ring($color-primary);

  &:disabled,
  &.disabled {
    opacity: $btn-disabled-opacity;
    cursor: not-allowed;
    pointer-events: none;
  }

  // ==============================
  // Variantes de couleur
  // ==============================
  $variants: (
    'primary':   ($color-primary, white),
    'secondary': ($color-secondary, white),
    'danger':    ($color-danger, white),
    'warning':   ($color-warning, $color-dark),
    'dark':      ($color-dark, white),
    'light':     ($color-light, $color-dark)
  );

  @each $name, $values in $variants {
    $bg:    nth($values, 1);
    $color: nth($values, 2);

    &.btn-#{$name} {
      background: $bg;
      color: $color;
      border-color: $bg;

      &:hover:not(:disabled) {
        background: color.adjust($bg, $lightness: -8%);
        border-color: color.adjust($bg, $lightness: -8%);
        transform: translateY(-1px);
        box-shadow: 0 4px 12px rgba($bg, 0.4);
      }

      &:active:not(:disabled) {
        transform: translateY(0);
        box-shadow: none;
      }
    }

    &.btn-outline-#{$name} {
      background: transparent;
      color: $bg;
      border-color: $bg;

      &:hover:not(:disabled) {
        background: $bg;
        color: $color;
      }
    }
  }

  // ==============================
  // Tailles
  // ==============================
  &.btn-sm {
    padding: $spacing-1 $spacing-3;
    font-size: $font-size-sm;
    border-radius: $border-radius-sm;
  }

  &.btn-lg {
    padding: $spacing-3 $spacing-8;
    font-size: $font-size-lg;
    border-radius: $border-radius-lg;
  }

  &.btn-block {
    width: 100%;
  }

  // ==============================
  // Contextes
  // ==============================
  .card & {
    width: 100%;
  }

  // Icône seule
  &.btn-icon {
    padding: $spacing-2;
    aspect-ratio: 1;
    border-radius: $border-radius-full;
  }
}
```

### 12.4 `_reset.scss` et `_typography.scss` de base

```scss
// src/styles/base/_reset.scss
@use '../abstracts' as *;

// Modern CSS Reset (inspiré de Andy Bell)
*,
*::before,
*::after {
  box-sizing: border-box;
}

* {
  margin: 0;
  padding: 0;
}

html {
  font-size: 16px;
  -webkit-text-size-adjust: 100%;
  scroll-behavior: smooth;

  @media (prefers-reduced-motion: reduce) {
    scroll-behavior: auto;
  }
}

body {
  font-family: $font-family-base;
  font-size: $font-size-base;
  line-height: $line-height-base;
  color: $color-dark;
  background: $color-white;
  @include font-smoothing;
}

img,
picture,
video,
canvas,
svg {
  display: block;
  max-width: 100%;
}

input,
button,
textarea,
select {
  font: inherit;
}

p,
h1,
h2,
h3,
h4,
h5,
h6 {
  overflow-wrap: break-word;
}

a {
  color: $color-primary;
  text-decoration: underline;
  text-underline-offset: 2px;

  &:hover {
    text-decoration: none;
  }
}
```

---

## 13. Intégration Bootstrap personnalisée avec SASS

### 13.1 Pourquoi personnaliser Bootstrap via SASS

Bootstrap expose toutes ses variables SASS avec le flag `!default`. En surchargeant ces variables **avant** d'importer Bootstrap, vous personnalisez tout le framework sans écrire une ligne de CSS en surcharge.

```bash
npm install bootstrap
```

### 13.2 Configuration complète

```scss
// src/styles/vendors/_bootstrap-custom.scss

// ============================================================
// ÉTAPE 1 — Surcharger les variables Bootstrap
// Toutes les variables Bootstrap utilisent !default
// Voir : node_modules/bootstrap/scss/_variables.scss
// ============================================================

// Couleurs
$primary:   #6c63ff;
$secondary: #2ec4b6;
$success:   #2ecc71;
$danger:    #e74c3c;
$warning:   #f39c12;
$info:      #3498db;
$light:     #f8f9fa;
$dark:      #1a1a2e;

// Typographie
$font-family-sans-serif: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
$font-size-base:         1rem;
$line-height-base:       1.6;
$font-weight-bold:       700;

// Espacements
$spacer: 1rem;
$spacers: (
  0: 0,
  1: $spacer * 0.25,   // 4px
  2: $spacer * 0.5,    // 8px
  3: $spacer,          // 16px
  4: $spacer * 1.5,    // 24px
  5: $spacer * 2,      // 32px
  6: $spacer * 3,      // 48px
  7: $spacer * 4,      // 64px
);

// Bordures
$border-radius:    0.5rem;
$border-radius-sm: 0.25rem;
$border-radius-lg: 1rem;
$border-radius-pill: 50rem;

// Ombres
$box-shadow:    0 4px 6px rgba(0, 0, 0, 0.07), 0 2px 4px rgba(0, 0, 0, 0.06);
$box-shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.12);
$box-shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);

// Boutons
$btn-padding-y: 0.5rem;
$btn-padding-x: 1.25rem;
$btn-font-weight: 600;
$btn-border-radius: $border-radius;

// Navbar
$navbar-padding-y: 1rem;
$navbar-dark-color: rgba(255, 255, 255, 0.85);

// Cards
$card-border-radius: $border-radius;
$card-box-shadow: $box-shadow;
$card-border-width: 0; // Pas de bordure — ombre uniquement

// Modals
$modal-content-border-radius: $border-radius-lg;

// Grid
$grid-gutter-width: 1.5rem;

// ============================================================
// ÉTAPE 2 — Importer uniquement les modules Bootstrap nécessaires
// (tree-shaking manuel — réduire le CSS final)
// ============================================================

// Obligatoires (fonctions et mixins Bootstrap)
@import 'bootstrap/scss/functions';
@import 'bootstrap/scss/variables';
@import 'bootstrap/scss/variables-dark';
@import 'bootstrap/scss/maps';
@import 'bootstrap/scss/mixins';
@import 'bootstrap/scss/utilities';

// Layout
@import 'bootstrap/scss/root';
@import 'bootstrap/scss/reboot';
@import 'bootstrap/scss/grid';
@import 'bootstrap/scss/containers';

// Composants — importer seulement ce dont vous avez besoin
@import 'bootstrap/scss/buttons';
@import 'bootstrap/scss/card';
@import 'bootstrap/scss/modal';
@import 'bootstrap/scss/nav';
@import 'bootstrap/scss/navbar';
@import 'bootstrap/scss/forms';
@import 'bootstrap/scss/badge';
@import 'bootstrap/scss/alert';
@import 'bootstrap/scss/dropdown';
@import 'bootstrap/scss/spinners';

// Helpers et utilitaires
@import 'bootstrap/scss/helpers';
@import 'bootstrap/scss/utilities/api'; // Génère les classes utilitaires

// ❌ NE PAS importer (non utilisé dans ce projet)
// @import 'bootstrap/scss/carousel';
// @import 'bootstrap/scss/offcanvas';
// @import 'bootstrap/scss/accordion';
```

### 13.3 Étendre Bootstrap avec vos propres composants

```scss
// src/styles/components/_custom-cards.scss
// Étendre les cards Bootstrap avec vos propres variantes

@use '../abstracts' as *;

// Card avec gradient
.card-gradient {
  @extend .card;
  background: linear-gradient(135deg, $color-primary, $color-secondary);
  color: white;
  border: none;

  .card-title,
  .card-text {
    color: inherit;
  }
}

// Card statistique (dashboard)
.card-stat {
  @extend .card;
  text-align: center;
  padding: $spacing-6;

  &__value {
    font-size: 3rem;
    font-weight: 800;
    color: $color-primary;
    line-height: 1;
    margin-bottom: $spacing-2;
  }

  &__label {
    font-size: $font-size-sm;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: rgba($color-dark, 0.6);
  }

  &__trend {
    margin-top: $spacing-2;
    font-size: $font-size-sm;
    font-weight: 600;

    &.up   { color: $color-secondary; }
    &.down { color: $color-danger; }
  }
}
```

---

## 14. Comparaison Tailwind CSS vs SASS

### 14.1 Philosophies opposées

| Critère | SASS | Tailwind CSS |
|---|---|---|
| **Paradigme** | CSS sémantique + abstraction | Utility-first (classes atomiques) |
| **Courbe d'apprentissage** | Progressive (CSS étendu) | Nécessite mémoriser les classes |
| **CSS bundle** | Maîtrisé (ce que vous écrivez) | Très petit avec PurgeCSS (JIT) |
| **Nommage** | Vous définissez vos classes | Classes prédéfinies (`flex`, `mt-4`) |
| **Composants** | Dans le SCSS | Via `@apply` ou components JS |
| **Thèmes** | Variables SASS + CSS vars | `tailwind.config.js` |
| **Customisation fine** | Total — tout est possible | Limité aux tokens Tailwind |
| **Prototypage** | Plus lent | Très rapide |
| **Maintenabilité** | Facile à comprendre | HTML verbeux |

### 14.2 Exemple — même composant dans les deux approches

```html
<!-- Tailwind — Card produit -->
<div class="bg-white rounded-xl shadow-md overflow-hidden hover:-translate-y-1
            hover:shadow-xl transition-all duration-300 cursor-pointer">
  <img class="w-full h-48 object-cover" src="product.jpg" alt="Produit">
  <div class="p-6">
    <span class="text-xs font-semibold uppercase tracking-widest text-blue-500
                 bg-blue-50 px-2 py-1 rounded-full">Nouveau</span>
    <h3 class="mt-3 text-xl font-bold text-gray-900">Nom du produit</h3>
    <p class="mt-2 text-sm text-gray-500 line-clamp-2">Description courte...</p>
    <div class="mt-4 flex items-center justify-between">
      <span class="text-2xl font-extrabold text-gray-900">49,99 €</span>
      <button class="bg-blue-600 hover:bg-blue-700 text-white font-semibold
                     py-2 px-4 rounded-lg transition-colors">
        Ajouter
      </button>
    </div>
  </div>
</div>
```

```html
<!-- SASS — même carte, HTML sémantique -->
<div class="product-card">
  <img class="product-card__image" src="product.jpg" alt="Produit">
  <div class="product-card__body">
    <span class="badge badge-primary">Nouveau</span>
    <h3 class="product-card__title">Nom du produit</h3>
    <p class="product-card__description">Description courte...</p>
    <div class="product-card__footer">
      <span class="product-card__price">49,99 €</span>
      <button class="btn btn-primary btn-sm">Ajouter</button>
    </div>
  </div>
</div>
```

```scss
// SASS — le composant est défini une seule fois, propre et maintenable
.product-card {
  @include card-base;
  @include hover-lift;
  cursor: pointer;

  &__image {
    width: 100%;
    height: 12rem;
    object-fit: cover;
  }

  &__body {
    padding: $spacing-6;
  }

  &__title {
    margin-top: $spacing-3;
    font-size: $font-size-xl;
    font-weight: 700;
    color: $color-dark;
  }

  &__description {
    margin-top: $spacing-2;
    font-size: $font-size-sm;
    color: rgba($color-dark, 0.6);
    @include text-truncate-multiline(2);
  }

  &__footer {
    margin-top: $spacing-4;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  &__price {
    font-size: 1.5rem;
    font-weight: 800;
    color: $color-dark;
  }
}
```

### 14.3 Quand choisir quoi ?

| Contexte | Recommandation |
|---|---|
| Prototypage rapide, startup early-stage | **Tailwind** |
| Application React/Vue avec composants réutilisables | **Tailwind** (+ CSS Modules) ou **SASS Modules** |
| Site vitrine, landing page | Les deux fonctionnent |
| Bibliothèque de composants (design system) | **SASS** — contrôle total |
| Personnalisation profonde de Bootstrap | **SASS** — obligatoire |
| Équipe déjà formée SASS | **SASS** — ne pas changer pour changer |
| Équipe déjà formée Tailwind | **Tailwind** |
| Projet Holberton B2/B3 | **SASS** — apprentissage des fondamentaux |

> [!tip] L'approche hybride
> Sur un projet React/Vue moderne, il est courant de combiner : Tailwind pour les utilitaires rapides (espacements, couleurs de base) + SASS Modules pour les composants complexes qui nécessitent de la logique. Ce n'est pas une opposition binaire.

---

## 15. Compilation, source maps et production

### 15.1 Modes de compilation

```bash
# Développement — lisible + source maps (défaut)
npx sass src/styles/main.scss dist/css/style.css

# Production — minifié, sans source maps
npx sass src/styles/main.scss dist/css/style.min.css \
  --style=compressed \
  --no-source-map

# Style expanded (défaut, lisible)
npx sass --style=expanded main.scss style.css

# Style compressed (production)
npx sass --style=compressed main.scss style.min.css
```

### 15.2 Source maps — déboguer le SCSS dans le navigateur

Les source maps sont des fichiers `.css.map` qui permettent aux DevTools du navigateur de montrer le fichier SCSS source au lieu du CSS compilé.

```
```
DevTools affiche :              Au lieu de :
────────────────                ─────────────────
components/_button.scss:42      style.css:1847
  background: #3498db;            background: #3498db;
```
```

```bash
# Source maps activées par défaut en mode non-compressé
npx sass main.scss style.css
# Génère style.css ET style.css.map

# Source maps intégrées dans le CSS (un seul fichier, plus pratique pour le dev)
npx sass --source-map-urls=absolute main.scss style.css

# Désactiver les source maps (production)
npx sass --no-source-map --style=compressed main.scss style.min.css
```

Le fichier `.map` contient la correspondance ligne par ligne entre le CSS compilé et les fichiers SCSS sources. **Ne jamais déployer les `.map` en production** (ils exposent votre code source).

### 15.3 Configuration Vite complète pour la production

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import { resolve } from 'path';

export default defineConfig(({ command, mode }) => ({
  resolve: {
    alias: {
      '@styles': resolve(__dirname, 'src/styles'),
    }
  },

  css: {
    preprocessorOptions: {
      scss: {
        // Injecté en tête de chaque fichier SCSS
        // Permet d'utiliser variables et mixins sans @use explicite
        additionalData: `
          @use '@styles/abstracts/variables' as *;
          @use '@styles/abstracts/mixins' as *;
        `,
        // Activer les warnings SASS
        quietDeps: false
      }
    },
    // Source maps en développement uniquement
    devSourcemap: mode === 'development',
    // Autoprefixer via PostCSS
    postcss: {
      plugins: [
        // npm install --save-dev autoprefixer
        require('autoprefixer')
      ]
    }
  },

  build: {
    // Minification CSS en production (intégré à Vite via esbuild)
    cssMinify: true,
    // Séparer le CSS dans son propre fichier
    cssCodeSplit: false,
  }
}));
```

### 15.4 Autoprefixer — compatibilité navigateurs

SASS ne gère pas les préfixes vendeurs (`-webkit-`, `-moz-`). Combiné à **PostCSS + Autoprefixer**, votre SCSS cible les navigateurs modernes et Autoprefixer ajoute les préfixes nécessaires à la compilation.

```bash
npm install --save-dev postcss autoprefixer
```

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('autoprefixer')
  ]
};
```

```json
// package.json — définir les navigateurs cibles
{
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not dead",
    "not ie 11"
  ]
}
```

```scss
// Vous écrivez
.container {
  display: grid;
  backdrop-filter: blur(10px);
  user-select: none;
}

/* Autoprefixer génère */
.container {
  display: grid;
  -webkit-backdrop-filter: blur(10px);
  backdrop-filter: blur(10px);
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none;
}
```

### 15.5 Optimisation CSS en production — checklist

```scss
// ✅ Bonne pratique — imports structurés avec @use
@use 'sass:color';
@use 'sass:math';
// Pas de @import (déprécié et moins optimisé)

// ✅ Éviter les sélecteurs universels coûteux
// ❌ * { box-sizing: border-box; } dans un composant
// ✅ *,*::before,*::after dans _reset.scss une seule fois

// ✅ Utiliser @extend avec parcimonie
// Préférer les mixins pour éviter les sélecteurs groupés inattendus

// ✅ Limiter la génération de classes utilitaires
// Ne générer que ce qui est réellement utilisé
$spacings-used: (2, 4, 6, 8); // Pas tous les spacings 1-20

// ✅ Variables CSS pour les valeurs dynamiques (dark mode)
:root {
  --color-bg: #{$color-white};
  --color-text: #{$color-dark};
}

[data-theme="dark"] {
  --color-bg: #1e1e2e;
  --color-text: #cdd6f4;
}
```

---

## 16. Projet pratique — Système de design complet

### 16.1 Structure du projet

Voici un projet complet illustrant toutes les notions du cours. L'objectif : construire la page d'accueil d'une application SaaS fictive.

```
```
src/styles/
├── abstracts/
│   ├── _index.scss
│   ├── _variables.scss
│   ├── _mixins.scss
│   └── _functions.scss
├── base/
│   ├── _index.scss
│   ├── _reset.scss
│   └── _typography.scss
├── components/
│   ├── _index.scss
│   ├── _button.scss
│   ├── _card.scss
│   └── _badge.scss
├── layout/
│   ├── _index.scss
│   ├── _navbar.scss
│   └── _section.scss
├── pages/
│   └── _home.scss
└── main.scss
```
```

### 16.2 `_functions.scss`

```scss
// src/styles/abstracts/_functions.scss
@use 'sass:math';
@use 'sass:color';
@use 'variables' as vars;

// Convertir px en rem
@function rem($px) {
  @return math.div($px, 16) * 1rem;
}

// Convertir px en em (relatif à une base donnée)
@function em($px, $context: 16) {
  @return math.div($px, $context) * 1em;
}

// Obtenir une couleur avec opacité
@function alpha($color, $opacity) {
  @return rgba($color, $opacity);
}

// Générer une couleur de fond légère depuis une couleur principale
@function tint($color, $weight: 90%) {
  @return color.mix(white, $color, $weight);
}

// Générer une couleur sombre depuis une couleur principale
@function shade($color, $weight: 20%) {
  @return color.mix(black, $color, $weight);
}

// Contraste automatique
@function auto-contrast($bg) {
  $lightness: color.lightness($bg);
  @return if($lightness > 55%, #111827, #ffffff);
}

// Espacement selon un indice
@function space($multiplier) {
  @return vars.$spacing-1 * $multiplier;
}
```

### 16.3 `_navbar.scss`

```scss
// src/styles/layout/_navbar.scss
@use '../abstracts' as *;

.navbar {
  position: sticky;
  top: 0;
  z-index: $z-sticky;
  background: rgba(white, 0.85);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid $border-color;

  &__inner {
    display: flex;
    align-items: center;
    justify-content: space-between;
    max-width: 1200px;
    margin: 0 auto;
    padding: $spacing-4 $spacing-6;

    @include respond-to('md') {
      padding: $spacing-3 $spacing-4;
    }
  }

  &__logo {
    font-size: $font-size-xl;
    font-weight: 800;
    text-decoration: none;
    @include gradient-text($color-primary, $color-secondary);
  }

  &__links {
    @include reset-list;
    display: flex;
    gap: $spacing-1;

    @include respond-to('md') {
      display: none; // Menu hamburger en mobile (hors scope de ce cours)
    }
  }

  &__link {
    display: block;
    padding: $spacing-2 $spacing-3;
    color: $color-dark;
    text-decoration: none;
    border-radius: $border-radius;
    font-size: $font-size-sm;
    font-weight: 500;
    transition: background $transition-fast, color $transition-fast;

    &:hover {
      background: alpha($color-primary, 0.08);
      color: $color-primary;
    }

    &.active {
      color: $color-primary;
      font-weight: 600;
    }
  }

  &__actions {
    display: flex;
    gap: $spacing-3;
    align-items: center;

    @include respond-to('sm') {
      display: none;
    }
  }
}
```

### 16.4 `_home.scss`

```scss
// src/styles/pages/_home.scss
@use '../abstracts' as *;
@use 'sass:math';

// ==============================
// Hero Section
// ==============================
.hero {
  min-height: 90vh;
  display: flex;
  align-items: center;
  position: relative;
  overflow: hidden;

  // Fond dégradé subtil
  background: linear-gradient(
    135deg,
    tint($color-primary, 95%) 0%,
    white 50%,
    tint($color-secondary, 92%) 100%
  );

  // Cercle décoratif
  &::before {
    content: '';
    @include absolute-fill;
    background:
      radial-gradient(circle at 80% 20%, alpha($color-primary, 0.05) 0%, transparent 50%),
      radial-gradient(circle at 20% 80%, alpha($color-secondary, 0.05) 0%, transparent 50%);
    pointer-events: none;
  }

  &__container {
    max-width: 1200px;
    margin: 0 auto;
    padding: $spacing-16 $spacing-6;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: $spacing-12;
    align-items: center;

    @include respond-to('lg') {
      grid-template-columns: 1fr;
      text-align: center;
    }
  }

  &__badge {
    display: inline-flex;
    align-items: center;
    gap: $spacing-2;
    background: alpha($color-primary, 0.1);
    color: $color-primary;
    padding: $spacing-1 $spacing-3;
    border-radius: $border-radius-full;
    font-size: $font-size-sm;
    font-weight: 600;
    margin-bottom: $spacing-4;

    &::before {
      content: '✦';
    }
  }

  &__title {
    font-size: clamp(2rem, 4vw + 1rem, 3.5rem);
    font-weight: 800;
    line-height: 1.1;
    letter-spacing: -0.03em;
    margin-bottom: $spacing-4;

    span {
      @include gradient-text($color-primary, $color-secondary);
    }
  }

  &__subtitle {
    font-size: $font-size-lg;
    color: rgba($color-dark, 0.65);
    line-height: 1.7;
    margin-bottom: $spacing-8;
    max-width: 40ch;

    @include respond-to('lg') {
      margin: 0 auto $spacing-8;
    }
  }

  &__cta {
    display: flex;
    gap: $spacing-3;
    flex-wrap: wrap;

    @include respond-to('lg') {
      justify-content: center;
    }
  }

  &__image {
    border-radius: $border-radius-xl;
    box-shadow: $shadow-xl;
    width: 100%;

    @include respond-to('lg') {
      max-width: 600px;
      margin: 0 auto;
    }
  }
}

// ==============================
// Features Section
// ==============================
.features {
  padding: $spacing-16 $spacing-6;

  &__container {
    max-width: 1200px;
    margin: 0 auto;
  }

  &__header {
    text-align: center;
    margin-bottom: $spacing-12;
  }

  &__title {
    font-size: $font-size-h2;
    font-weight: 800;
    margin-bottom: $spacing-3;
  }

  &__subtitle {
    font-size: $font-size-lg;
    color: rgba($color-dark, 0.6);
    max-width: 50ch;
    margin: 0 auto;
  }

  &__grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: $spacing-6;

    @include respond-to('lg') {
      grid-template-columns: repeat(2, 1fr);
    }

    @include respond-to('sm') {
      grid-template-columns: 1fr;
    }
  }
}

// Feature Card
.feature-card {
  padding: $spacing-6;
  border-radius: $border-radius-lg;
  border: 1px solid $border-color;
  background: white;
  transition: all $transition-base;

  &:hover {
    border-color: alpha($color-primary, 0.3);
    box-shadow: $shadow-lg;
    transform: translateY(-4px);
  }

  &__icon {
    width: 3rem;
    height: 3rem;
    border-radius: $border-radius;
    background: tint($color-primary, 88%);
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.5rem;
    margin-bottom: $spacing-4;
  }

  &__title {
    font-size: $font-size-lg;
    font-weight: 700;
    margin-bottom: $spacing-2;
  }

  &__description {
    font-size: $font-size-sm;
    color: rgba($color-dark, 0.65);
    line-height: 1.7;
  }
}
```

---

## 17. Exercices pratiques

> [!info] Exercices progressifs
> Ces exercices vont du niveau débutant (A) au niveau avancé (C). Faites-les dans l'ordre.

### Exercice A1 — Variables et nesting (débutant)

**Objectif** : convertir un CSS existant en SCSS avec variables et nesting.

```css
/* CSS à convertir */
.card {
  background: white;
  border-radius: 8px;
  padding: 24px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}
.card h2 {
  color: #3498db;
  font-size: 20px;
  margin-bottom: 12px;
}
.card p {
  color: #666;
  font-size: 14px;
  line-height: 1.6;
}
.card .btn {
  background: #3498db;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
}
.card .btn:hover {
  background: #2980b9;
}
```

**Attendu** : créer `_variables.scss` avec les couleurs, un fichier `_card.scss` avec nesting, et un `main.scss` qui les importe.

### Exercice A2 — Mixin responsive (débutant-intermédiaire)

**Objectif** : créer un mixin `respond-to` et l'utiliser sur un composant.

Créer un composant `.pricing-card` qui affiche 3 colonnes sur desktop, 2 sur tablette et 1 sur mobile, en utilisant le mixin.

### Exercice B1 — Boucle et map (intermédiaire)

**Objectif** : générer un système de couleurs complet avec `@each`.

```scss
// Créer cette map et générer automatiquement :
// - .bg-{color}
// - .text-{color}
// - .border-{color}
// - .btn-{color} (avec :hover)

$palette: (
  'blue':   #3498db,
  'green':  #2ecc71,
  'red':    #e74c3c,
  'orange': #e67e22,
  'purple': #9b59b6
);
```

### Exercice B2 — Architecture 7-1 (intermédiaire)

**Objectif** : restructurer un projet existant selon le pattern 7-1.

Vous avez un fichier `style.css` de 600 lignes. Identifiez les catégories (variables, reset, composants, layout) et répartissez-les dans la structure 7-1.

### Exercice C1 — Système de design (avancé)

**Objectif** : construire un système de design complet utilisable sur un projet réel.

Réaliser :
1. `_variables.scss` avec toutes les tokens (couleurs, typo, espacements, ombres)
2. `_mixins.scss` avec minimum 8 mixins utiles
3. `_functions.scss` avec `rem()`, `tint()`, `auto-contrast()`
4. Un composant `.data-table` responsive avec états hover et alternance de lignes
5. Un composant `.toast-notification` avec 4 variantes (success, error, warning, info)

### Exercice C2 — Bootstrap personnalisé (avancé)

**Objectif** : personnaliser Bootstrap pour une marque fictive "PurpleWave".

Contraintes :
- Couleur primaire : `#7c3aed`
- Police : Google Fonts "Poppins"
- Border-radius de tous les composants : `0.75rem`
- Ombres plus prononcées que Bootstrap par défaut
- Supprimer les composants Bootstrap non utilisés (carousel, accordion)

---

## 18. Bonnes pratiques et erreurs courantes

### 18.1 Les erreurs les plus fréquentes

```scss
// ❌ Erreur 1 — Nesting trop profond
.nav .nav-list .nav-item .nav-link .nav-link-icon {
  color: red; // Sélecteur cauchemardesque
}

// ❌ Erreur 2 — @extend dans une media query (interdit)
@media (max-width: 768px) {
  .mobile-card {
    @extend .card; // ❌ Erreur de compilation
  }
}

// ❌ Erreur 3 — Utiliser @import (déprécié)
@import 'variables'; // ❌ Utiliser @use à la place

// ❌ Erreur 4 — Interpolation inutile
$color: red;
.btn {
  color: #{$color}; // ❌ Inutile ici
  color: $color;    // ✅ Suffisant
}

// ❌ Erreur 5 — Magic numbers sans variable
.hero {
  padding: 73px 48px; // ❌ D'où viennent ces valeurs ?
}

// ✅ Correct
.hero {
  padding: space(18) space(12); // ✅ Cohérent avec le système
}

// ❌ Erreur 6 — Mixin pour styles sans paramètres (utiliser un placeholder)
@mixin reset-list { list-style: none; margin: 0; padding: 0; }
// → Utiliser %reset-list si pas de paramètres

// ❌ Erreur 7 — Redéfinir une variable globale dans un composant
// src/styles/components/_button.scss
$color-primary: #ff0000; // ❌ Casse tous les autres composants !
```

### 18.2 Checklist avant livraison

```
```
Checklist SCSS — avant de livrer votre code
──────────────────────────────────────────

Variables
  □ Toutes les couleurs sont dans _variables.scss (pas de hex en dur dans les composants)
  □ Pas de magic numbers — tout dans des variables nommées
  □ Convention de nommage cohérente ($color-primary, $spacing-4...)

Structure
  □ Pattern 7-1 respecté (ou structure équivalente définie)
  □ Un seul fichier compilé (main.scss)
  □ Tous les autres fichiers sont des partials (_prefix.scss)
  □ @use utilisé, pas @import

Qualité du code
  □ Nesting ≤ 3 niveaux
  □ Mixins pour tout code répété avec paramètres
  □ Placeholders %... pour styles partagés sans paramètres
  □ Boucles @each pour générer les classes utilitaires

Performance
  □ Aucune classe CSS inutilisée générée
  □ Source maps désactivées en production
  □ CSS minifié en production
  □ Autoprefixer configuré
```
```

---

## Récapitulatif des modules SASS natifs

| Module | Import | Fonctions clés |
|---|---|---|
| `sass:math` | `@use 'sass:math'` | `math.div()`, `math.round()`, `math.floor()`, `math.ceil()`, `math.abs()`, `math.clamp()` |
| `sass:color` | `@use 'sass:color'` | `color.adjust()`, `color.mix()`, `color.hue()`, `color.lightness()`, `color.change()` |
| `sass:string` | `@use 'sass:string'` | `string.length()`, `string.slice()`, `string.to-upper-case()`, `string.index()` |
| `sass:list` | `@use 'sass:list'` | `list.length()`, `list.nth()`, `list.append()`, `list.join()`, `list.index()` |
| `sass:map` | `@use 'sass:map'` | `map.get()`, `map.set()`, `map.merge()`, `map.keys()`, `map.values()`, `map.has-key()` |
| `sass:selector` | `@use 'sass:selector'` | `selector.nest()`, `selector.extend()`, `selector.replace()` |
| `sass:meta` | `@use 'sass:meta'` | `meta.type-of()`, `meta.inspect()`, `meta.call()`, `meta.module-variables()` |

> [!tip] Pour aller plus loin
> - Documentation officielle Dart Sass : https://sass-lang.com/documentation/
> - Bootstrap SASS source : https://github.com/twbs/bootstrap/tree/main/scss
> - Pattern 7-1 détaillé : https://sass-guidelin.es/#the-7-1-pattern
> - Sass Guidelines (Hugo Giraudel) : https://sass-guidelin.es/
