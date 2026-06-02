# Webpack et Bundlers Modernes

Les applications web modernes sont composées de dizaines, voire de centaines de fichiers JavaScript, CSS, images et autres ressources. Un **bundler** est l'outil qui assemble ces fichiers épars en un ensemble cohérent, optimisé et prêt pour la production. Webpack est le bundler le plus utilisé dans l'industrie, mais l'écosystème a évolué vers des alternatives plus rapides comme Vite, Rollup ou esbuild. Ce cours couvre l'intégralité du workflow de bundling, de la configuration de base aux optimisations avancées.

---

## 1. Pourquoi un bundler ?

### 1.1 Le problème des modules dans le navigateur

Pendant longtemps, le navigateur n'avait aucun système de modules natif. Tout le JavaScript était chargé via des balises `<script>` dans l'ordre, partageant un même espace global :

```html
<!-- Avant les bundlers : fragile et non maintenable -->
<script src="jquery.js"></script>
<script src="lodash.js"></script>
<script src="utils.js"></script>
<script src="components.js"></script>
<script src="app.js"></script>
```

> [!warning] Les problèmes de cette approche
> - **Pollution du scope global** : chaque fichier peut écraser les variables des autres
> - **Ordre manuel obligatoire** : si `app.js` dépend de `utils.js`, il faut maintenir cet ordre à la main
> - **Pas d'encapsulation** : impossible de savoir quelles fonctions sont "privées"
> - **Performances** : des dizaines de requêtes HTTP pour charger tous les scripts

### 1.2 L'arrivée des systèmes de modules

Node.js a introduit CommonJS (CJS) en 2009, puis le standard ECMAScript Modules (ESM) a été finalisé en 2015 :

```javascript
// CommonJS (Node.js, avant 2015)
const lodash = require('lodash');
module.exports = { maFonction };

// ESM - ECMAScript Modules (standard actuel)
import { debounce } from 'lodash';
export const maFonction = () => {};
```

Le problème : les navigateurs anciens ne comprenaient pas ces syntaxes. Un bundler résout ce problème en transformant ce graphe de dépendances en un (ou plusieurs) fichiers que n'importe quel navigateur peut charger.

### 1.3 Ce que fait concrètement un bundler

```
Fichiers source (modules)          Bundler              Sortie (navigateur)
─────────────────────────          ───────              ───────────────────

src/
├── index.js  ─────────────┐
│   └── import './app.js'  │
├── app.js  ───────────────┤      ┌─────────┐           dist/
│   └── import './utils'   ├─────▶│ WEBPACK │──────────▶├── bundle.js (minifié)
├── utils.js ───────────────┤      │  VITE   │           ├── styles.css
│   └── import 'lodash'    │      │  ROLLUP │           └── index.html
├── styles.css ─────────────┤      └─────────┘
└── logo.png ───────────────┘
     ↑
node_modules/
└── lodash/
    └── lodash.js
```

Les responsabilités d'un bundler moderne :

| Responsabilité | Description |
|---|---|
| **Résolution de dépendances** | Construire le graphe complet des imports |
| **Transpilation** | Convertir TypeScript, JSX, ES2022+ en JS compatible |
| **Bundling** | Assembler tous les modules en un ou plusieurs fichiers |
| **Optimisation** | Minification, tree shaking, code splitting |
| **Assets** | Gérer les images, polices, CSS |
| **Dev Server** | Serveur local avec rechargement automatique |

---

## 2. Webpack 5 — Installation et configuration de base

### 2.1 Installation

```bash
# Initialiser un projet Node
mkdir mon-projet && cd mon-projet
npm init -y

# Installer Webpack et son CLI
npm install --save-dev webpack webpack-cli

# Vérifier la version
npx webpack --version
# webpack: 5.x.x
# webpack-cli: 5.x.x
```

### 2.2 Premier bundle sans configuration

Webpack 5 fonctionne sans configuration grâce à des conventions par défaut :

```
# Structure de projet minimale
mon-projet/
├── src/
│   └── index.js    ← entry point par défaut
├── package.json
└── node_modules/
```

```javascript
// src/index.js
import { saluer } from './utils.js';

const message = saluer('Holberton');
console.log(message);
```

```javascript
// src/utils.js
export function saluer(nom) {
  return `Bonjour, ${nom} !`;
}
```

```bash
# Bundler sans config (mode production par défaut)
npx webpack

# Résultat : dist/main.js (minifié)
```

### 2.3 webpack.config.js — Structure complète

Le fichier de configuration est un module CommonJS qui exporte un objet :

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  // ─── Entrée ───────────────────────────────────────────────────
  // Le ou les points d'entrée du graphe de dépendances
  entry: './src/index.js',

  // ─── Sortie ───────────────────────────────────────────────────
  output: {
    // Dossier de sortie (chemin absolu obligatoire)
    path: path.resolve(__dirname, 'dist'),

    // Nom du fichier bundle principal
    // [contenthash] = hash basé sur le contenu (cache-busting)
    filename: '[name].[contenthash].js',

    // Vider le dossier dist avant chaque build
    clean: true,
  },

  // ─── Mode ─────────────────────────────────────────────────────
  // 'development' | 'production' | 'none'
  mode: 'development',

  // ─── Résolution ───────────────────────────────────────────────
  resolve: {
    // Extensions résolues automatiquement (dans l'ordre)
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],

    // Alias pour éviter les imports relatifs trop longs
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
    },
  },

  // ─── Loaders ──────────────────────────────────────────────────
  module: {
    rules: [
      // Les rules sont définies ici (section suivante)
    ],
  },

  // ─── Plugins ──────────────────────────────────────────────────
  plugins: [
    // Les plugins sont définis ici (section 4)
  ],
};
```

### 2.4 Entry points multiples

Pour des applications multi-pages ou des bibliothèques :

```javascript
// webpack.config.js — Entry points multiples
module.exports = {
  entry: {
    // Clé = nom du chunk, valeur = fichier d'entrée
    app: './src/app.js',
    admin: './src/admin.js',
    vendeurs: ['./src/polyfills.js', './src/vendors.js'], // tableau
  },

  output: {
    // [name] sera remplacé par la clé (app, admin, vendeurs)
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

### 2.5 Comprendre le graphe de dépendances

Webpack part du ou des entry points et construit un graphe en suivant récursivement tous les imports :

```
entry: src/index.js
       │
       ├── import './components/Header.jsx'
       │         │
       │         ├── import './Logo.png'
       │         └── import '../styles/header.css'
       │
       ├── import './services/api.js'
       │         │
       │         └── import 'axios'  ← node_modules
       │
       └── import './utils/format.js'
```

Webpack ne bundle que ce qui est réellement importé — les fichiers non utilisés sont ignorés.

---

## 3. Loaders

### 3.1 Principe des loaders

Webpack ne comprend nativement que le JavaScript et le JSON. Les **loaders** permettent de transformer n'importe quel type de fichier en module JavaScript :

```
Fichier .css ─── css-loader ──▶ Module JS (représente le CSS)
              └── style-loader ▶ Injection dans le DOM

Fichier .jsx ─── babel-loader ▶ Module JS (ES5 compatible)

Fichier .png ─── file-loader ──▶ Module JS (retourne l'URL)
```

> [!info] Ordre d'exécution des loaders
> Les loaders s'exécutent **de droite à gauche** (ou de bas en haut dans un tableau). `use: ['style-loader', 'css-loader']` signifie : d'abord `css-loader`, ensuite `style-loader`.

### 3.2 css-loader et style-loader

```bash
npm install --save-dev css-loader style-loader
```

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        // Tester l'extension du fichier avec une regex
        test: /\.css$/,

        // Ordre d'exécution : css-loader d'abord, style-loader ensuite
        use: [
          'style-loader',  // 2. Injecte le CSS dans des balises <style> dans le DOM
          'css-loader',    // 1. Résout les @import et url() dans le CSS
        ],
      },
    ],
  },
};
```

```css
/* src/styles/main.css */
.container {
  max-width: 1200px;
  margin: 0 auto;
  background: url('../images/bg.png'); /* url() résolu par css-loader */
}
```

```javascript
// src/index.js
// Le CSS est importé comme un module !
import './styles/main.css';
```

> [!tip] En production
> `style-loader` injecte le CSS en JavaScript (augmente la taille du bundle JS). En production, utiliser `MiniCssExtractPlugin` à la place (voir section 4) pour extraire le CSS dans un fichier séparé.

### 3.3 SASS/SCSS avec sass-loader

```bash
npm install --save-dev sass-loader sass
```

```javascript
// webpack.config.js
{
  test: /\.(scss|sass)$/,
  use: [
    'style-loader',   // 3. Injecte dans le DOM
    'css-loader',     // 2. Résout les imports CSS
    'sass-loader',    // 1. Compile SCSS → CSS
  ],
}
```

```scss
// src/styles/main.scss
$primary-color: #3498db;
$font-size-base: 16px;

.container {
  color: $primary-color;
  font-size: $font-size-base;

  &:hover {
    color: darken($primary-color, 10%);
  }
}
```

### 3.4 babel-loader — Transpilation JavaScript

Babel transforme le JavaScript moderne (ES2022+, JSX, TypeScript) en JavaScript compatible avec les anciens navigateurs.

```bash
npm install --save-dev babel-loader @babel/core @babel/preset-env @babel/preset-react
```

```javascript
// webpack.config.js
{
  test: /\.(js|jsx)$/,

  // Exclure node_modules (déjà compilé par les auteurs des packages)
  exclude: /node_modules/,

  use: {
    loader: 'babel-loader',
    options: {
      // Les presets peuvent aussi être dans babel.config.js ou .babelrc
      presets: [
        [
          '@babel/preset-env',
          {
            // Cibler les navigateurs supportés
            targets: '> 0.25%, not dead',

            // Ajouter les polyfills automatiquement selon les targets
            useBuiltIns: 'usage',
            corejs: 3,
          },
        ],
        // Transpile JSX (React)
        '@babel/preset-react',
      ],
    },
  },
}
```

```javascript
// babel.config.js (alternative : définir la config Babel ici)
module.exports = {
  presets: [
    ['@babel/preset-env', { targets: '> 0.25%, not dead' }],
    '@babel/preset-react',
  ],
  plugins: [
    // Décorateurs, class properties, etc.
    '@babel/plugin-proposal-class-properties',
  ],
};
```

### 3.5 file-loader et url-loader (approche Webpack 4)

> [!warning] Obsolescence partielle
> En Webpack 5, `file-loader` et `url-loader` sont remplacés par les **Asset Modules** natifs (section 9). Cette section est utile pour comprendre les projets existants Webpack 4.

```bash
npm install --save-dev file-loader url-loader
```

```javascript
// webpack.config.js (style Webpack 4)
{
  // Images
  test: /\.(png|jpg|gif|svg)$/,
  use: [
    {
      loader: 'url-loader',
      options: {
        // Si l'image est < 8KB : encoder en base64 (inline)
        // Si >= 8KB : déléguer à file-loader (fichier séparé)
        limit: 8192,
        name: 'images/[name].[hash:8].[ext]',
      },
    },
  ],
},
{
  // Polices de caractères
  test: /\.(woff|woff2|eot|ttf|otf)$/,
  use: [
    {
      loader: 'file-loader',
      options: {
        name: 'fonts/[name].[hash:8].[ext]',
        outputPath: 'assets/',
      },
    },
  ],
},
```

### 3.6 Écrire un loader personnalisé

Un loader est une fonction qui reçoit le contenu du fichier et retourne du JavaScript :

```javascript
// loaders/markdown-loader.js
const { marked } = require('marked');

// Le loader reçoit le contenu brut du fichier
module.exports = function markdownLoader(source) {
  // Transformer le Markdown en HTML
  const html = marked.parse(source);

  // Retourner du JavaScript valide (export du résultat)
  return `module.exports = ${JSON.stringify(html)};`;
};
```

```javascript
// webpack.config.js
{
  test: /\.md$/,
  use: [
    // Chemin vers notre loader personnalisé
    path.resolve(__dirname, 'loaders/markdown-loader.js'),
  ],
}
```

```javascript
// Utilisation dans le code
import readme from './README.md';
document.getElementById('content').innerHTML = readme;
```

---

## 4. Plugins

### 4.1 Différence Loaders / Plugins

| | Loaders | Plugins |
|---|---|---|
| **Rôle** | Transformer un type de fichier | Intervenir dans le processus de build |
| **Portée** | Fichier par fichier | Build global |
| **Syntaxe** | `module.rules` | `plugins: [new Plugin()]` |
| **Exemples** | `babel-loader`, `css-loader` | `HtmlWebpackPlugin`, `DefinePlugin` |

Les plugins ont accès aux **hooks** du cycle de vie de Webpack et peuvent modifier le comportement à n'importe quelle étape.

### 4.2 HtmlWebpackPlugin

Génère automatiquement un fichier `index.html` qui inclut les bons bundles (avec les hashes de cache-busting).

```bash
npm install --save-dev html-webpack-plugin
```

```javascript
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      // Template de base (optionnel mais recommandé)
      template: './src/index.html',

      // Nom du fichier généré dans dist/
      filename: 'index.html',

      // Titre de la page
      title: 'Mon Application',

      // Injecter les scripts automatiquement
      // 'body' = avant </body>, 'head' = dans <head>
      inject: 'body',

      // Minifier le HTML en production
      minify: process.env.NODE_ENV === 'production' ? {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
      } : false,

      // Variables disponibles dans le template EJS
      meta: {
        viewport: 'width=device-width, initial-scale=1',
        description: 'Application Holberton School',
      },
    }),
  ],
};
```

```html
<!-- src/index.html (template EJS) -->
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title><%= htmlWebpackPlugin.options.title %></title>
  <!-- Les CSS seront injectées ici automatiquement -->
</head>
<body>
  <div id="root"></div>
  <!-- Les scripts seront injectés ici automatiquement -->
</body>
</html>
```

### 4.3 MiniCssExtractPlugin

Extrait le CSS en fichiers séparés au lieu de l'injecter dans le JS via `style-loader` :

```bash
npm install --save-dev mini-css-extract-plugin
```

```javascript
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

const isProduction = process.env.NODE_ENV === 'production';

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // En prod : extraire dans un fichier CSS séparé
          // En dev : injecter via style-loader (plus rapide pour HMR)
          isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
          'css-loader',
        ],
      },
    ],
  },

  plugins: [
    ...(isProduction ? [
      new MiniCssExtractPlugin({
        // Nom du fichier CSS extrait
        filename: 'styles/[name].[contenthash:8].css',
        // CSS des chunks chargés dynamiquement
        chunkFilename: 'styles/[id].[contenthash:8].css',
      }),
    ] : []),
  ],
};
```

### 4.4 CopyWebpackPlugin

Copie des fichiers statiques dans le dossier de sortie (images, fonts, fichiers public/) :

```bash
npm install --save-dev copy-webpack-plugin
```

```javascript
// webpack.config.js
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
  plugins: [
    new CopyWebpackPlugin({
      patterns: [
        {
          // Source : dossier public/
          from: path.resolve(__dirname, 'public'),

          // Destination : racine du dossier dist/
          to: path.resolve(__dirname, 'dist'),

          // Exclure certains fichiers
          globOptions: {
            ignore: ['**/.DS_Store', '**/Thumbs.db'],
          },
        },
        {
          // Copier un fichier spécifique
          from: path.resolve(__dirname, 'src/assets/robots.txt'),
          to: path.resolve(__dirname, 'dist/robots.txt'),
        },
      ],
    }),
  ],
};
```

### 4.5 DefinePlugin — Variables d'environnement

Injecte des constantes globales au moment du build (remplacées textuellement dans le code) :

```javascript
// webpack.config.js
const { DefinePlugin } = require('webpack');

module.exports = {
  plugins: [
    new DefinePlugin({
      // IMPORTANT : les valeurs sont du code JS brut
      // Pour une chaîne, il faut JSON.stringify()
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
      'process.env.API_URL': JSON.stringify(process.env.API_URL || 'http://localhost:3000'),

      // Constante booléenne (pas de guillemets = expression JS directe)
      '__DEV__': process.env.NODE_ENV !== 'production',

      // Version de l'application
      '__APP_VERSION__': JSON.stringify(require('./package.json').version),
    }),
  ],
};
```

```javascript
// Dans le code source — Webpack remplace la valeur au build
if (__DEV__) {
  console.log('Mode développement activé');
}

// Après compilation (en production), devient :
if (false) {
  console.log('...');
}
// Et le minifier élimine ce bloc mort (dead code elimination)
```

> [!tip] dotenv-webpack
> Pour charger automatiquement les variables depuis un fichier `.env`, utiliser `dotenv-webpack` :
> ```bash
> npm install --save-dev dotenv-webpack
> ```
> ```javascript
> const Dotenv = require('dotenv-webpack');
> plugins: [new Dotenv()]
> ```

### 4.6 BundleAnalyzerPlugin — Visualiser la taille du bundle

```bash
npm install --save-dev webpack-bundle-analyzer
```

```javascript
// webpack.config.js
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
  plugins: [
    // Ouvre une interface web avec la carte des modules
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',         // Génère un HTML statique
      reportFilename: 'bundle-report.html',
      openAnalyzer: false,            // Ne pas ouvrir auto (CI friendly)
    }),
  ],
};
```

```
# Lancer uniquement pour analyser
ANALYZE=true npx webpack --mode production
```

---

## 5. Mode development vs production

### 5.1 Différences fondamentales

| Fonctionnalité | Development | Production |
|---|---|---|
| **Vitesse de build** | Rapide | Lent (optimisations) |
| **Taille du bundle** | Grande | Minimale (minification) |
| **Source maps** | Complètes (`eval-source-map`) | Absentes ou externes |
| **Tree shaking** | Partiel | Complet |
| **Minification** | Non | Oui (Terser) |
| **Console.log** | Conservés | Supprimables |
| **HMR** | Activé | Désactivé |
| **Noms lisibles** | Oui | Hachés/minifiés |

### 5.2 Webpack config par environnement — Pattern recommandé

```
webpack/
├── webpack.common.js    ← Configuration partagée
├── webpack.dev.js       ← Configuration développement
└── webpack.prod.js      ← Configuration production
```

```bash
npm install --save-dev webpack-merge
```

```javascript
// webpack/webpack.common.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',

  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: '[name].[contenthash:8].js',
    clean: true,
  },

  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
    alias: {
      '@': path.resolve(__dirname, '../src'),
    },
  },

  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: 'babel-loader',
      },
      {
        test: /\.(png|jpg|webp|gif)$/,
        type: 'asset',
        parser: {
          dataUrlCondition: { maxSize: 8 * 1024 },
        },
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name][ext]',
        },
      },
    ],
  },

  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
    }),
  ],
};
```

```javascript
// webpack/webpack.dev.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'development',

  // Source maps en développement
  // eval-source-map : rapide, précis, lisible
  devtool: 'eval-source-map',

  // Serveur de développement
  devServer: {
    port: 3000,
    hot: true,          // HMR activé
    open: true,         // Ouvrir le navigateur automatiquement
    historyApiFallback: true, // SPA : renvoyer index.html pour toutes les routes
  },

  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
});
```

```javascript
// webpack/webpack.prod.js
const { merge } = require('webpack-merge');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'production',

  // Source maps en production : fichier séparé, non envoyé au client
  devtool: 'source-map',

  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader'],
      },
    ],
  },

  plugins: [
    new MiniCssExtractPlugin({
      filename: 'styles/[name].[contenthash:8].css',
    }),
  ],

  optimization: {
    // Activer la minification
    minimize: true,

    minimizer: [
      // Minifier le JavaScript (Terser)
      new TerserPlugin({
        terserOptions: {
          compress: {
            // Supprimer les console.log en production
            drop_console: true,
            drop_debugger: true,
          },
          format: {
            // Supprimer les commentaires
            comments: false,
          },
        },
        extractComments: false,
      }),

      // Minifier le CSS
      new CssMinimizerPlugin(),
    ],
  },
});
```

```json
// package.json — Scripts NPM
{
  "scripts": {
    "dev": "webpack serve --config webpack/webpack.dev.js",
    "build": "webpack --config webpack/webpack.prod.js",
    "build:analyze": "ANALYZE=true webpack --config webpack/webpack.prod.js",
    "build:stats": "webpack --config webpack/webpack.prod.js --json > stats.json"
  }
}
```

### 5.3 Source maps

Les source maps permettent de déboguer le code compilé en le reliant au code source original :

```javascript
// Options de devtool par ordre de vitesse / précision
module.exports = {
  devtool: 'eval',                    // Très rapide, pas de source map réelle
  devtool: 'eval-cheap-source-map',   // Rapide, colonnes inexactes
  devtool: 'eval-source-map',         // ← DEV recommandé : lent au premier build, précis
  devtool: 'source-map',              // ← PROD : fichier .map séparé
  devtool: 'hidden-source-map',       // Fichier .map mais pas référencé dans le bundle
  devtool: false,                     // Pas de source map
};
```

---

## 6. Code Splitting

### 6.1 Pourquoi splitter le code ?

Sans code splitting, toute l'application est dans un seul bundle. Un utilisateur qui visite la page d'accueil charge aussi le code du tableau de bord admin.

```
Sans code splitting       Avec code splitting
─────────────────────     ─────────────────────

bundle.js (2 MB)          main.js (200 KB)        ← chargé immédiatement
 ├── Page Accueil          ├── Accueil chunk        ← lazy-loaded au besoin
 ├── Page Dashboard        ├── Dashboard chunk      ← lazy-loaded au besoin
 ├── Page Admin            └── Admin chunk          ← lazy-loaded au besoin
 └── Bibliothèques         vendors.js (500 KB)     ← mis en cache longtemps
```

### 6.2 import() dynamique — Lazy Loading

La syntaxe `import()` dynamique est la clé du code splitting en JavaScript moderne :

```javascript
// src/router.js — Charger les pages à la demande

// ❌ Import statique : tout est bundlé ensemble
import DashboardPage from './pages/Dashboard';

// ✅ Import dynamique : chunk séparé, chargé à la demande
const loadDashboard = () => import('./pages/Dashboard');

// Exemple avec React Router
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Admin = React.lazy(() => import('./pages/Admin'));

function App() {
  return (
    <Suspense fallback={<div>Chargement...</div>}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/admin" element={<Admin />} />
      </Routes>
    </Suspense>
  );
}
```

```javascript
// Nommer les chunks avec les "magic comments" Webpack
const loadChart = () => import(
  /* webpackChunkName: "chart-library" */    // Nom du chunk
  /* webpackPrefetch: true */                // Précharger après idle
  /* webpackPreload: true */                 // Précharger en parallèle
  'chart.js'
);
```

> [!info] Prefetch vs Preload
> - **`webpackPrefetch`** : ressource probablement nécessaire dans le futur. Chargée pendant l'idle du navigateur. Ajoute `<link rel="prefetch">`.
> - **`webpackPreload`** : ressource nécessaire pendant le chargement actuel. Priorité haute. Ajoute `<link rel="preload">`.

### 6.3 SplitChunksPlugin — Configuration avancée

SplitChunksPlugin est intégré dans Webpack et extrait automatiquement les modules partagés :

```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      // 'all' : splitter les chunks synchrones et asynchrones
      // 'async' : uniquement les imports dynamiques (défaut)
      // 'initial' : uniquement les chunks initiaux
      chunks: 'all',

      // Taille minimale pour créer un chunk (en octets)
      minSize: 20000,

      // Taille minimale de réduction pour qu'un split soit effectué
      minRemainingSize: 0,

      // Nombre minimal d'imports pour extraire un module
      minChunks: 1,

      // Nombre max de requêtes asynchrones parallèles
      maxAsyncRequests: 30,

      // Nombre max de requêtes initiales parallèles
      maxInitialRequests: 30,

      // Délimiteur dans le nom du chunk généré
      automaticNameDelimiter: '~',

      // Groupes de cache (règles de splitting)
      cacheGroups: {
        // Groupe "vendors" : bibliothèques de node_modules
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: -10,
        },

        // Groupe séparé pour React et React-DOM
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
          name: 'react',
          chunks: 'all',
          priority: 20, // Priorité plus haute que "vendors"
        },

        // Modules partagés entre plusieurs chunks
        common: {
          name: 'common',
          minChunks: 2,  // Au moins 2 chunks utilisent ce module
          priority: -20,
          reuseExistingChunk: true, // Réutiliser si déjà extrait
        },
      },
    },

    // Extrait le runtime Webpack dans un chunk séparé
    // (évite d'invalider le cache vendors à chaque build)
    runtimeChunk: 'single',
  },
};
```

### 6.4 Visualiser le résultat du code splitting

Après un build, vérifier les chunks générés :

```bash
npx webpack --mode production --json | npx webpack-bundle-analyzer

# Ou avec le plugin BundleAnalyzerPlugin configuré
npm run build:analyze
```

```
Résultat typique d'un build avec splitting :

dist/
├── index.html
├── main.abc12345.js         (200 KB) ← code applicatif
├── vendors.def67890.js      (500 KB) ← bibliothèques tierces
├── react.ghi11223.js        (130 KB) ← React + React-DOM
├── common.jkl44556.js       (50 KB)  ← modules partagés
├── dashboard.mno77889.js    (80 KB)  ← lazy chunk
└── admin.pqr00112.js        (60 KB)  ← lazy chunk
```

---

## 7. Tree Shaking

### 7.1 Principe

Le tree shaking ("secouage d'arbre") élimine le code non utilisé du bundle final. Une importation partielle d'une bibliothèque ne doit pas inclure toute la bibliothèque.

```javascript
// ❌ Sans tree shaking — importe TOUT lodash (70KB+)
import _ from 'lodash';
const result = _.debounce(fn, 300);

// ✅ Avec tree shaking — n'importe que debounce
import { debounce } from 'lodash-es';
const result = debounce(fn, 300);
```

### 7.2 Conditions pour le tree shaking

> [!warning] ESM obligatoire
> Le tree shaking ne fonctionne qu'avec les modules **ESM** (`import`/`export`). Les modules CommonJS (`require`/`module.exports`) ne peuvent pas être tree-shaked efficacement car les imports sont dynamiques à l'exécution.

```javascript
// ✅ ESM — tree-shakeable
export function utilise() { return 'utilisé'; }
export function inutile() { return 'pas importé'; }

// src/app.js
import { utilise } from './utils'; // 'inutile' sera éliminé
```

```javascript
// ❌ CommonJS — pas de tree shaking
exports.utilise = function() { return 'utilisé'; };
exports.inutile = function() { return 'pas importé'; };

// src/app.js
const { utilise } = require('./utils'); // 'inutile' reste dans le bundle !
```

### 7.3 sideEffects dans package.json

Webpack doit savoir si l'importation d'un module a des **effets de bord** (modification du DOM, polyfills, CSS, etc.) :

```json
// package.json — Déclarer que le package n'a pas d'effets de bord
{
  "name": "ma-librairie",
  "sideEffects": false
}
```

```json
// package.json — Déclarer les fichiers AVEC effets de bord
{
  "name": "mon-app",
  "sideEffects": [
    "*.css",          // Les imports CSS ont des effets de bord
    "*.scss",
    "./src/polyfills.js"
  ]
}
```

> [!warning] Attention aux CSS
> Si vous déclarez `"sideEffects": false`, Webpack peut éliminer vos imports CSS (`import './styles.css'`), car techniquement un import CSS sans valeur de retour ressemble à du dead code. Toujours lister les CSS dans `sideEffects`.

### 7.4 Activer le tree shaking dans Webpack

```javascript
// webpack.config.js
module.exports = {
  mode: 'production', // Active automatiquement tree shaking + minification

  optimization: {
    // Activer le tree shaking (activé par défaut en mode production)
    usedExports: true,

    // Concaténer les modules (scope hoisting) — réduit la taille et améliore les perf
    concatenateModules: true,

    // Active la minification (nécessaire pour éliminer physiquement le dead code)
    minimize: true,
  },
};
```

### 7.5 Vérifier le tree shaking

```javascript
// src/math.js — Module avec plusieurs exports
export const addition = (a, b) => a + b;        // Utilisé
export const soustraction = (a, b) => a - b;    // Utilisé
export const multiplication = (a, b) => a * b;  // NON utilisé
export const division = (a, b) => a / b;        // NON utilisé

// src/index.js
import { addition, soustraction } from './math';
console.log(addition(1, 2), soustraction(5, 3));
```

```bash
# Build et vérifier que multiplication/division ne sont pas dans le bundle
npx webpack --mode production
grep "multiplication" dist/main.js  # Ne doit rien retourner
```

---

## 8. Hot Module Replacement (HMR)

### 8.1 Principe

Le HMR remplace les modules modifiés dans le navigateur **sans rechargement complet de la page**. L'état de l'application est préservé.

```
Sans HMR                    Avec HMR
──────────────────────      ──────────────────────────────
Modifier un fichier         Modifier un fichier
        ↓                           ↓
Rechargement complet        Webpack recompile le module
de la page                          ↓
        ↓                   Envoi du module via WebSocket
État de l'app perdu                 ↓
(formulaires, scroll,       Remplacement à chaud
navigation...)              État préservé
```

### 8.2 webpack-dev-server

```bash
npm install --save-dev webpack-dev-server
```

```javascript
// webpack.dev.js
module.exports = {
  devServer: {
    // Port du serveur
    port: 3000,

    // Activer HMR
    hot: true,

    // Activer le live reload (fallback si HMR échoue)
    liveReload: true,

    // Ouvrir le navigateur au démarrage
    open: true,

    // Pour les SPAs : retourner index.html pour toutes les routes inconnues
    historyApiFallback: true,

    // Proxy des requêtes API pour éviter les problèmes CORS
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true,
        pathRewrite: { '^/api': '' },
      },
    },

    // Servir des fichiers statiques
    static: {
      directory: path.join(__dirname, 'public'),
    },

    // HTTPS (utile pour tester PWA, Service Workers)
    https: false,

    // Compresser les réponses
    compress: true,

    // Configuration des headers HTTP
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  },
};
```

```json
// package.json
{
  "scripts": {
    "dev": "webpack serve --config webpack/webpack.dev.js",
    "dev:hot": "webpack serve --config webpack/webpack.dev.js --hot"
  }
}
```

### 8.3 HMR avec React — React Fast Refresh

```bash
npm install --save-dev @pmmmwh/react-refresh-webpack-plugin react-refresh
```

```javascript
// webpack.dev.js
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');

module.exports = merge(common, {
  mode: 'development',
  devServer: { hot: true },

  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            plugins: [
              // Activer React Fast Refresh en développement
              'react-refresh/babel',
            ],
          },
        },
      },
    ],
  },

  plugins: [
    new ReactRefreshWebpackPlugin(),
  ],
});
```

### 8.4 HMR API — Mise à jour manuelle de modules

Pour les modules sans framework qui ne supportent pas le HMR automatiquement :

```javascript
// src/counter.js
export let count = 0;
export function increment() { count++; }

// Indicateur HMR : le module accepte ses propres mises à jour
if (module.hot) {
  module.hot.accept('./counter', () => {
    // Callback appelé quand counter.js est modifié
    console.log('counter.js mis à jour !');
    // Ré-initialiser si nécessaire
  });

  // Préserver l'état entre les rechargements
  module.hot.dispose((data) => {
    data.count = count; // Sauvegarder l'état
  });

  // Restaurer l'état après rechargement
  if (module.hot.data) {
    count = module.hot.data.count;
  }
}
```

---

## 9. Asset Modules (Webpack 5)

### 9.1 Présentation

Webpack 5 intègre nativement la gestion des assets sans `file-loader` ni `url-loader` :

| Type | Description | Équivalent Webpack 4 |
|---|---|---|
| `asset/resource` | Fichier émis séparé, URL retournée | `file-loader` |
| `asset/inline` | Base64 encodé dans le bundle | `url-loader` (limit=∞) |
| `asset/source` | Contenu brut comme string | `raw-loader` |
| `asset` | Automatique selon la taille | `url-loader` avec `limit` |

### 9.2 Configuration complète des Asset Modules

```javascript
// webpack.config.js
module.exports = {
  output: {
    // Nom par défaut des assets émis
    assetModuleFilename: 'assets/[hash][ext][query]',
  },

  module: {
    rules: [
      // ─── Images ───────────────────────────────────────────────────
      {
        test: /\.(png|jpg|jpeg|gif|svg|webp)$/,

        // 'asset' : Webpack décide selon la taille
        type: 'asset',

        // Seuil : < 8KB → inline, >= 8KB → fichier séparé
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024, // 8 KB
          },
        },

        // Surcharger le nom pour ce type
        generator: {
          filename: 'images/[name].[hash:8][ext]',
        },
      },

      // ─── Polices ──────────────────────────────────────────────────
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,

        // Toujours un fichier séparé
        type: 'asset/resource',

        generator: {
          filename: 'fonts/[name].[hash:8][ext]',
        },
      },

      // ─── SVG inline (pour les icônes) ─────────────────────────────
      {
        test: /\.svg$/,

        // Toujours inliné en base64
        type: 'asset/inline',
      },

      // ─── Fichiers texte bruts ─────────────────────────────────────
      {
        test: /\.txt$/,

        // Retourne le contenu comme string
        type: 'asset/source',
      },
    ],
  },
};
```

```javascript
// Utilisation dans le code
import logo from './assets/logo.png';     // URL string
import font from './assets/roboto.woff2'; // URL string
import icon from './assets/star.svg';     // Base64 data URL
import text from './README.txt';          // String brut

// Dans React
function Header() {
  return (
    <header>
      <img src={logo} alt="Logo" />
    </header>
  );
}
```

---

## 10. Module Federation (Webpack 5)

### 10.1 Qu'est-ce que le Module Federation ?

Module Federation est la fonctionnalité la plus innovante de Webpack 5. Elle permet à des **applications distinctes** de partager du code à l'exécution, sans build commun. C'est le fondement technique des **micro-frontends**.

```
Application Shell (host)
├── Charge l'app principale
└── Consomme des modules distants

        ┌─────────────────────────┐
        │   APP SHELL             │
        │   localhost:3000        │
        │                         │
        │  ┌────────┐             │
        │  │ Header │◄────────────┼──── Remote: Header/Header
        │  └────────┘             │           localhost:3001
        │                         │
        │  ┌────────┐             │
        │  │ Cart   │◄────────────┼──── Remote: Cart/CartWidget
        │  └────────┘             │           localhost:3002
        └─────────────────────────┘
```

### 10.2 Configuration d'un Remote (fournisseur)

```javascript
// app-header/webpack.config.js — Application qui EXPOSE des modules
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  mode: 'development',
  devServer: { port: 3001 },

  plugins: [
    new ModuleFederationPlugin({
      // Nom unique de cette application fedérée
      name: 'headerApp',

      // Fichier d'entrée remote (chargé par les hosts)
      filename: 'remoteEntry.js',

      // Modules exposés aux autres applications
      exposes: {
        // './Header' = alias utilisé par les consommateurs
        // './src/components/Header' = chemin réel dans ce projet
        './Header': './src/components/Header',
        './Navigation': './src/components/Navigation',
      },

      // Dépendances partagées (évite de charger React deux fois)
      shared: {
        react: {
          singleton: true,         // Une seule instance dans toute la page
          requiredVersion: '^18.0.0',
          eager: false,            // Chargé à la demande
        },
        'react-dom': {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
      },
    }),
  ],
};
```

### 10.3 Configuration d'un Host (consommateur)

```javascript
// app-shell/webpack.config.js — Application qui CONSOMME des modules distants
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  mode: 'development',
  devServer: { port: 3000 },

  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',

      // Modules distants consommés
      remotes: {
        // 'headerApp' = nom local utilisé dans les imports
        // 'headerApp@URL/remoteEntry.js' = nom du remote @ URL du remoteEntry
        headerApp: 'headerApp@http://localhost:3001/remoteEntry.js',
        cartApp: 'cartApp@http://localhost:3002/remoteEntry.js',
      },

      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

```javascript
// app-shell/src/App.js — Consommer un module fédéré
import React, { Suspense } from 'react';

// Import dynamique du composant distant
const Header = React.lazy(() => import('headerApp/Header'));
const CartWidget = React.lazy(() => import('cartApp/CartWidget'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Chargement du header...</div>}>
        <Header title="Mon App" />
      </Suspense>

      <main>
        <Suspense fallback={<div>Chargement du panier...</div>}>
          <CartWidget userId={user.id} />
        </Suspense>
      </main>
    </div>
  );
}
```

> [!tip] Module Federation en production
> En production, les URLs des remotes doivent pointer vers des CDN ou des serveurs stables. Utiliser des variables d'environnement ou une configuration dynamique pour gérer les différents environnements (dev/staging/prod).

---

## 11. Vite — L'alternative ultra-rapide

### 11.1 Architecture de Vite

Vite prend une approche radicalement différente de Webpack :

```
Webpack (approche bundle-based)        Vite (approche native ESM)
────────────────────────────────       ────────────────────────────
Au démarrage :                         Au démarrage :
  Analyser tous les modules              Ne rien bundler
  Transpiler tout le code                Démarrer instantanément
  Créer le bundle complet                       ↓
  Démarrer le serveur          →         Navigateur demande /src/index.js
       ↓                                 Vite transforme à la volée
  Lent (10-60s sur gros projets)         Uniquement ce module
                                         Très rapide (<1s)
En prod :
  Rollup bundle l'application
  (optimisé, ESM natif)
```

### 11.2 Installation et configuration

```bash
# Créer un projet Vite
npm create vite@latest mon-projet -- --template react
cd mon-projet
npm install
npm run dev
```

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  // ─── Plugins ─────────────────────────────────────────────────────
  plugins: [
    react({
      // Fast Refresh activé par défaut
      fastRefresh: true,
    }),
  ],

  // ─── Résolution ───────────────────────────────────────────────────
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
    },
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
  },

  // ─── Serveur de développement ─────────────────────────────────────
  server: {
    port: 3000,
    open: true,
    hmr: true,

    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },

  // ─── Build (utilise Rollup) ───────────────────────────────────────
  build: {
    outDir: 'dist',
    sourcemap: false,

    rollupOptions: {
      output: {
        // Manual chunks pour contrôler le splitting
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },

    // Taille limite avant warning
    chunkSizeWarningLimit: 1000,
  },

  // ─── Variables d'environnement ────────────────────────────────────
  // Vite charge automatiquement .env, .env.local, .env.[mode]
  // Les variables doivent commencer par VITE_ pour être exposées
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
  },

  // ─── Pré-bundling des dépendances (esbuild) ───────────────────────
  optimizeDeps: {
    // Forcer le pré-bundling de certains packages
    include: ['react', 'react-dom'],
    // Exclure du pré-bundling
    exclude: ['ma-lib-locale'],
  },
});
```

```
# Fichiers .env avec Vite
.env                # Toujours chargé
.env.local          # Toujours chargé, ignoré par git
.env.development    # Chargé en mode dev
.env.production     # Chargé en mode prod

# Contenu
VITE_API_URL=https://api.exemple.com  # Accessible via import.meta.env.VITE_API_URL
API_SECRET=xyz                         # NE SERA PAS exposé (pas de préfixe VITE_)
```

### 11.3 Comparaison avec Webpack

```javascript
// Webpack : config JS obligatoire
module.exports = { ... }

// Vite : config optionnelle, zéro config pour les cas simples
// vite.config.js n'est pas obligatoire pour un projet React basique
```

| Aspect | Webpack 5 | Vite 5 |
|---|---|---|
| Démarrage dev | Lent (10-60s) | Ultra-rapide (<1s) |
| HMR | Rapide | Instantané |
| Config | Verbeuse mais flexible | Concise et bien documentée |
| Écosystème | Mature, très large | Jeune, croissance rapide |
| Build prod | Webpack bundle | Rollup bundle |
| Support TypeScript | Via babel-loader ou ts-loader | Natif (esbuild) |
| CSS Modules | Loader requis | Natif |
| Migration | Point de départ | Migration possible |

---

## 12. Rollup — Pour les bibliothèques

### 12.1 Positionnement

Rollup est spécialisé dans la création de **bibliothèques JavaScript** avec un output ESM propre. Son tree shaking est plus efficace que Webpack pour les libs.

```bash
npm install --save-dev rollup
# Plugins essentiels
npm install --save-dev @rollup/plugin-node-resolve @rollup/plugin-commonjs @rollup/plugin-babel
```

```javascript
// rollup.config.js
import { nodeResolve } from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import terser from '@rollup/plugin-terser';
import pkg from './package.json' assert { type: 'json' };

export default [
  // ─── Build ESM (pour bundlers modernes) ───────────────────────────
  {
    input: 'src/index.js',

    output: {
      file: pkg.module,       // ex: dist/index.esm.js
      format: 'esm',
      sourcemap: true,
    },

    // Externaliser les dépendances (ne pas les bundler dans la lib)
    external: ['react', 'react-dom'],

    plugins: [
      nodeResolve({ extensions: ['.js', '.jsx'] }),
      babel({
        babelHelpers: 'bundled',
        presets: ['@babel/preset-react'],
        exclude: 'node_modules/**',
      }),
      terser(),
    ],
  },

  // ─── Build CJS (pour Node.js et bundlers anciens) ─────────────────
  {
    input: 'src/index.js',

    output: {
      file: pkg.main,         // ex: dist/index.cjs.js
      format: 'cjs',
      exports: 'named',
      sourcemap: true,
    },

    external: ['react', 'react-dom'],

    plugins: [
      nodeResolve({ extensions: ['.js', '.jsx'] }),
      commonjs(),
      babel({
        babelHelpers: 'bundled',
        presets: ['@babel/preset-react'],
        exclude: 'node_modules/**',
      }),
      terser(),
    ],
  },
];
```

```json
// package.json — Exposer plusieurs formats
{
  "name": "ma-librairie",
  "main": "dist/index.cjs.js",
  "module": "dist/index.esm.js",
  "exports": {
    ".": {
      "import": "./dist/index.esm.js",
      "require": "./dist/index.cjs.js"
    }
  },
  "sideEffects": false
}
```

---

## 13. esbuild — Performance extrême

### 13.1 Positionnement

esbuild est écrit en Go et peut être **10 à 100 fois plus rapide** que les bundlers JavaScript. Il est utilisé comme transformateur dans Vite et comme bundler standalone.

```bash
npm install --save-dev esbuild
```

```javascript
// build.js — esbuild API Node.js
const esbuild = require('esbuild');

esbuild.build({
  entryPoints: ['src/index.js'],
  bundle: true,
  minify: true,
  sourcemap: true,
  target: ['chrome90', 'firefox88', 'safari14'],
  outfile: 'dist/bundle.js',

  // Externaliser des modules
  external: ['react', 'react-dom'],

  // Définir des variables globales
  define: {
    'process.env.NODE_ENV': '"production"',
  },

  // Loader pour différents types de fichiers
  loader: {
    '.png': 'file',
    '.svg': 'text',
    '.css': 'css',
  },

  // Plugins esbuild
  plugins: [],
}).catch(() => process.exit(1));
```

```bash
# CLI directe — build ultra-rapide
npx esbuild src/index.js --bundle --minify --outfile=dist/bundle.js

# Avec watch mode
npx esbuild src/index.js --bundle --watch --outfile=dist/bundle.js
```

### 13.2 Limitations d'esbuild

> [!warning] Ce qu'esbuild ne fait pas (encore)
> - **Pas de code splitting** aussi avancé que Webpack
> - **Pas de HMR natif** (délégué à Vite ou autre)
> - **Pas de plugins riches** (écosystème plus petit)
> - **Pas de support TypeScript avec vérification de types** (transpile sans type-check)
> - **Pas de support CSS Modules** natif
> - Rollup ou Webpack restent préférables pour des configurations complexes

---

## 14. Parcel — Zero Configuration

### 14.1 Présentation

Parcel fonctionne sans aucune configuration. Il détecte automatiquement les transformations nécessaires (TypeScript, JSX, SASS) en lisant les fichiers de config existants (`tsconfig.json`, `.babelrc`, etc.).

```bash
npm install --save-dev parcel

# Lancer directement depuis le HTML
npx parcel src/index.html
```

```html
<!-- src/index.html — Parcel part de l'HTML -->
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="./styles/main.scss">
</head>
<body>
  <div id="root"></div>
  <!-- Parcel détecte JSX/TypeScript automatiquement -->
  <script type="module" src="./index.tsx"></script>
</body>
</html>
```

```bash
# Parcel installe automatiquement les dépendances manquantes !
# Si index.tsx est detecté, Parcel installe @parcel/transformer-typescript

# Build production
npx parcel build src/index.html --dist-dir dist
```

> [!info] Quand utiliser Parcel ?
> Parcel est idéal pour les **prototypes rapides** et les **petits projets** sans besoins de configuration avancée. Pour des projets en production avec des besoins spécifiques (Module Federation, configuration fine), Webpack ou Vite sont plus adaptés.

---

## 15. Tableau comparatif complet

| Critère | Webpack 5 | Vite 5 | Rollup | esbuild | Parcel 2 |
|---|---|---|---|---|---|
| **Démarrage dev** | Lent (10-60s) | Instantané (<1s) | N/A | Ultra-rapide | Rapide |
| **HMR** | Bien | Excellent | N/A | Via plugins | Bien |
| **Build prod** | Webpack | Rollup | Rollup | esbuild | Parcel |
| **Configuration** | Verbeuse | Concise | Moyenne | Minimaliste | Zéro |
| **Tree shaking** | Bon | Excellent | Excellent | Bon | Bon |
| **Code splitting** | Excellent | Bien | Partiel | Partiel | Bien |
| **Module Federation** | Oui natif | Plugin | Non | Non | Non |
| **TypeScript** | Via loader | Natif | Via plugin | Natif | Natif |
| **CSS Modules** | Via loader | Natif | Via plugin | Partiel | Natif |
| **Écosystème plugins** | Très large | Croissant | Large | Petit | Moyen |
| **Communauté** | Très grande | Grande | Grande | Croissante | Moyenne |
| **Cas d'usage principal** | Apps complexes | Apps modernes | Bibliothèques | Transforms | Prototypes |
| **Maturité** | Très mature | Mature | Très mature | Jeune | Mature |

### 15.1 Arbre de décision

```
Tu crées une bibliothèque NPM ?
        │
        ▼ Oui
    → ROLLUP (meilleur tree shaking, formats multiples)

Tu crées une bibliothèque NPM ?
        │
        ▼ Non
Tu as besoin de micro-frontends (Module Federation) ?
        │
        ▼ Oui
    → WEBPACK 5

Tu as besoin de micro-frontends ?
        │
        ▼ Non
C'est un prototype ou petit projet ?
        │
        ▼ Oui
    → PARCEL ou VITE

C'est un prototype ?
        │
        ▼ Non
Le projet est nouveau (pas de migration nécessaire) ?
        │
        ├─── Oui ──▶ VITE (développement rapide, DX excellente)
        │
        └─── Non (migration depuis Webpack) ──▶ WEBPACK (moins de friction)
```

---

## 16. Cas pratique : Webpack 5 pour React (dev + prod)

### 16.1 Structure du projet

```
mon-app-react/
├── src/
│   ├── index.js              ← Entry point
│   ├── App.jsx               ← Composant racine
│   ├── components/
│   │   ├── Header.jsx
│   │   └── Footer.jsx
│   ├── pages/
│   │   ├── Home.jsx
│   │   └── Dashboard.jsx     ← Lazy loaded
│   ├── hooks/
│   │   └── useApi.js
│   ├── services/
│   │   └── api.js
│   ├── styles/
│   │   ├── main.scss
│   │   └── variables.scss
│   └── assets/
│       ├── logo.svg
│       └── hero.jpg
├── public/
│   ├── favicon.ico
│   └── robots.txt
├── webpack/
│   ├── webpack.common.js
│   ├── webpack.dev.js
│   └── webpack.prod.js
├── babel.config.js
├── package.json
└── .env
```

### 16.2 Installation complète

```bash
# ─── Projet React ───────────────────────────────────────────────────
npm install react react-dom react-router-dom

# ─── Webpack core ───────────────────────────────────────────────────
npm install --save-dev webpack webpack-cli webpack-merge webpack-dev-server

# ─── Babel ──────────────────────────────────────────────────────────
npm install --save-dev \
  babel-loader \
  @babel/core \
  @babel/preset-env \
  @babel/preset-react \
  @babel/plugin-proposal-class-properties

# React Fast Refresh (HMR React)
npm install --save-dev \
  @pmmmwh/react-refresh-webpack-plugin \
  react-refresh

# ─── CSS/SCSS ───────────────────────────────────────────────────────
npm install --save-dev \
  css-loader \
  style-loader \
  mini-css-extract-plugin \
  css-minimizer-webpack-plugin \
  sass \
  sass-loader \
  postcss \
  postcss-loader \
  postcss-preset-env

# ─── Plugins ────────────────────────────────────────────────────────
npm install --save-dev \
  html-webpack-plugin \
  copy-webpack-plugin \
  dotenv-webpack \
  terser-webpack-plugin \
  webpack-bundle-analyzer
```

### 16.3 babel.config.js

```javascript
// babel.config.js
module.exports = (api) => {
  const isTest = api.env('test');
  const isDev = api.env('development');

  return {
    presets: [
      [
        '@babel/preset-env',
        {
          // Cibler les navigateurs modernes
          targets: '> 0.25%, not dead, not ie 11',
          useBuiltIns: 'usage',
          corejs: { version: 3, proposals: true },
          // Pas de transformation des modules en dev (native ESM)
          modules: isTest ? 'commonjs' : false,
        },
      ],
      [
        '@babel/preset-react',
        {
          // Fast Refresh en développement
          runtime: 'automatic',
          development: isDev,
        },
      ],
    ],

    plugins: [
      '@babel/plugin-proposal-class-properties',
      // Fast Refresh plugin uniquement en dev
      ...(isDev ? ['react-refresh/babel'] : []),
    ],
  };
};
```

### 16.4 webpack/webpack.common.js — Configuration partagée complète

```javascript
// webpack/webpack.common.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const Dotenv = require('dotenv-webpack');

module.exports = {
  // ─── Entry ────────────────────────────────────────────────────────
  entry: path.resolve(__dirname, '../src/index.js'),

  // ─── Output ───────────────────────────────────────────────────────
  output: {
    path: path.resolve(__dirname, '../dist'),

    // [contenthash] pour le cache-busting en production
    filename: 'js/[name].[contenthash:8].js',
    chunkFilename: 'js/[name].[contenthash:8].chunk.js',

    // URL de base pour les assets (important pour les sous-dossiers)
    publicPath: '/',

    // Nettoyer dist/ avant chaque build
    clean: true,
  },

  // ─── Résolution ───────────────────────────────────────────────────
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],

    alias: {
      '@': path.resolve(__dirname, '../src'),
      '@components': path.resolve(__dirname, '../src/components'),
      '@pages': path.resolve(__dirname, '../src/pages'),
      '@hooks': path.resolve(__dirname, '../src/hooks'),
      '@services': path.resolve(__dirname, '../src/services'),
      '@styles': path.resolve(__dirname, '../src/styles'),
      '@assets': path.resolve(__dirname, '../src/assets'),
    },
  },

  // ─── Loaders ──────────────────────────────────────────────────────
  module: {
    rules: [
      // JavaScript / JSX
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: 'babel-loader',
      },

      // Images
      {
        test: /\.(png|jpg|jpeg|gif|webp)$/,
        type: 'asset',
        parser: {
          dataUrlCondition: { maxSize: 8 * 1024 }, // < 8KB → inline
        },
        generator: {
          filename: 'images/[name].[hash:8][ext]',
        },
      },

      // SVG
      {
        test: /\.svg$/,
        type: 'asset/resource',
        generator: {
          filename: 'images/[name].[hash:8][ext]',
        },
        // Permettre aussi l'import SVG comme composant React
        use: [
          {
            loader: '@svgr/webpack',
            options: {
              icon: true,
            },
          },
        ],
      },

      // Polices
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name].[hash:8][ext]',
        },
      },
    ],
  },

  // ─── Plugins ──────────────────────────────────────────────────────
  plugins: [
    // Générer index.html
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../src/index.html'),
      filename: 'index.html',
      inject: true,
    }),

    // Copier les fichiers public/
    new CopyWebpackPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, '../public'),
          to: path.resolve(__dirname, '../dist'),
          noErrorOnMissing: true,
          globOptions: {
            ignore: ['**/.DS_Store'],
          },
        },
      ],
    }),

    // Variables d'environnement depuis .env
    new Dotenv({
      path: path.resolve(__dirname, '../.env'),
      safe: false,
      defaults: true,
    }),
  ],

  // ─── Optimisation (base) ──────────────────────────────────────────
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Chunk React séparé (mis en cache longtemps)
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]/,
          name: 'react-vendors',
          chunks: 'all',
          priority: 20,
        },

        // Autres dépendances node_modules
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10,
        },

        // Modules partagés entre plusieurs chunks applicatifs
        common: {
          name: 'common',
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true,
        },
      },
    },

    // Runtime Webpack séparé (évite d'invalider le cache vendors)
    runtimeChunk: 'single',
  },
};
```

### 16.5 webpack/webpack.dev.js — Développement complet

```javascript
// webpack/webpack.dev.js
const path = require('path');
const { merge } = require('webpack-merge');
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  // ─── Mode ─────────────────────────────────────────────────────────
  mode: 'development',

  // ─── Source Maps ──────────────────────────────────────────────────
  // eval-source-map : bon équilibre vitesse/précision en dev
  devtool: 'eval-source-map',

  // ─── Serveur de développement ─────────────────────────────────────
  devServer: {
    port: 3000,
    hot: true,
    open: true,
    compress: true,
    historyApiFallback: true,

    static: {
      directory: path.resolve(__dirname, '../public'),
    },

    proxy: [
      {
        context: ['/api'],
        target: 'http://localhost:5000',
        changeOrigin: true,
      },
    ],

    client: {
      // Afficher les erreurs de compilation dans le navigateur
      overlay: {
        errors: true,
        warnings: false,
      },
      // Indicateur de progression dans le titre de l'onglet
      progress: true,
    },
  },

  // ─── Loaders CSS (dev : style-loader pour HMR) ────────────────────
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                // CSS Modules : pattern de nommage lisible en dev
                localIdentName: '[name]__[local]--[hash:base64:5]',
              },
            },
          },
        ],
      },
      {
        test: /\.(scss|sass)$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[name]__[local]--[hash:base64:5]',
              },
            },
          },
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: ['postcss-preset-env'],
              },
            },
          },
          'sass-loader',
        ],
      },
    ],
  },

  // ─── Plugins dev ──────────────────────────────────────────────────
  plugins: [
    // React Fast Refresh — HMR avec préservation de l'état
    new ReactRefreshWebpackPlugin(),
  ],

  // ─── Optimisation dev ─────────────────────────────────────────────
  optimization: {
    // Pas de minification en dev
    minimize: false,

    // Activer quand même l'analyse des exports pour le dev
    usedExports: true,
  },
});
```

### 16.6 webpack/webpack.prod.js — Production complète

```javascript
// webpack/webpack.prod.js
const { merge } = require('webpack-merge');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const { DefinePlugin } = require('webpack');
const common = require('./webpack.common.js');

const shouldAnalyze = process.env.ANALYZE === 'true';

module.exports = merge(common, {
  // ─── Mode ─────────────────────────────────────────────────────────
  mode: 'production',

  // ─── Source maps externes ─────────────────────────────────────────
  // source-map : fichier .map séparé, non livré au client
  devtool: 'source-map',

  // ─── Loaders CSS (prod : MiniCssExtractPlugin) ────────────────────
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              modules: {
                // Hash court en production
                localIdentName: '[hash:base64:8]',
              },
            },
          },
        ],
      },
      {
        test: /\.(scss|sass)$/,
        use: [
          MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[hash:base64:8]',
              },
            },
          },
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: ['postcss-preset-env'],
              },
            },
          },
          'sass-loader',
        ],
      },
    ],
  },

  // ─── Plugins prod ─────────────────────────────────────────────────
  plugins: [
    // Extraire le CSS dans des fichiers séparés
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css',
      chunkFilename: 'css/[id].[contenthash:8].chunk.css',
    }),

    // Définir les variables de build
    new DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify('production'),
    }),

    // Analyser le bundle (uniquement si ANALYZE=true)
    ...(shouldAnalyze ? [
      new BundleAnalyzerPlugin({
        analyzerMode: 'static',
        reportFilename: 'bundle-report.html',
        openAnalyzer: true,
      }),
    ] : []),
  ],

  // ─── Optimisation prod ────────────────────────────────────────────
  optimization: {
    minimize: true,

    minimizer: [
      // Minifier JavaScript
      new TerserPlugin({
        terserOptions: {
          parse: { ecma: 8 },
          compress: {
            ecma: 5,
            warnings: false,
            comparisons: false,
            inline: 2,
            drop_console: true,   // Supprimer console.log
            drop_debugger: true,  // Supprimer debugger
          },
          mangle: { safari10: true },
          output: {
            ecma: 5,
            comments: false,
            ascii_only: true,
          },
        },
        parallel: true,           // Paralléliser la minification
        extractComments: false,
      }),

      // Minifier CSS
      new CssMinimizerPlugin({
        minimizerOptions: {
          preset: ['default', {
            discardComments: { removeAll: true },
          }],
        },
      }),
    ],

    // Configuration du code splitting
    splitChunks: {
      chunks: 'all',
      maxInitialRequests: 25,
      maxAsyncRequests: 25,
      minSize: 20000,

      cacheGroups: {
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]/,
          name: 'react-vendors',
          chunks: 'all',
          priority: 20,
          enforce: true,
        },

        vendors: {
          test: /[\\/]node_modules[\\/]/,
          name(module) {
            // Créer un chunk par package node_modules volumineux
            const packageName = module.context.match(
              /[\\/]node_modules[\\/](.*?)([\\/]|$)/
            )[1];
            return `npm.${packageName.replace('@', '')}`;
          },
          chunks: 'all',
          priority: 10,
          minSize: 30000,
        },

        common: {
          name: 'common',
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true,
          minSize: 10000,
        },
      },
    },

    runtimeChunk: 'single',

    // Tree shaking complet
    usedExports: true,
    sideEffects: true,
    concatenateModules: true,
  },

  // ─── Performance budgets ──────────────────────────────────────────
  performance: {
    // Afficher les warnings si les assets dépassent ces seuils
    hints: 'warning',
    maxEntrypointSize: 512000,    // 500 KB
    maxAssetSize: 512000,         // 500 KB
  },
});
```

### 16.7 package.json — Scripts complets

```json
{
  "name": "mon-app-react",
  "version": "1.0.0",
  "scripts": {
    "dev": "webpack serve --config webpack/webpack.dev.js",
    "build": "webpack --config webpack/webpack.prod.js",
    "build:dev": "webpack --config webpack/webpack.dev.js",
    "build:analyze": "cross-env ANALYZE=true webpack --config webpack/webpack.prod.js",
    "build:stats": "webpack --config webpack/webpack.prod.js --json > stats.json",
    "clean": "rm -rf dist"
  },
  "sideEffects": [
    "*.css",
    "*.scss"
  ]
}
```

### 16.8 src/index.js — Entry point React

```javascript
// src/index.js
import React from 'react';
import { createRoot } from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';
import '@styles/main.scss';

// Point d'entrée de l'application React
const container = document.getElementById('root');
const root = createRoot(container);

root.render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);

// Hot Module Replacement en développement
if (module.hot) {
  module.hot.accept();
}
```

```javascript
// src/App.jsx
import React, { Suspense, lazy } from 'react';
import { Routes, Route } from 'react-router-dom';
import Header from '@components/Header';

// Lazy loading des pages
const Home = lazy(() => import(/* webpackChunkName: "home" */ '@pages/Home'));
const Dashboard = lazy(() =>
  import(/* webpackChunkName: "dashboard" */ '@pages/Dashboard')
);

function App() {
  return (
    <div className="app">
      <Header />

      <Suspense fallback={<div className="loading">Chargement...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </div>
  );
}

export default App;
```

---

## 17. Exercices pratiques

### Exercice 1 — Configuration de base (Niveau débutant)

**Objectif** : Configurer Webpack from scratch sans framework.

1. Créer un projet avec cette structure :
```
exercice-1/
├── src/
│   ├── index.js
│   ├── calculator.js
│   └── styles.css
└── package.json
```

2. `calculator.js` exporte `add`, `subtract`, `multiply`
3. `index.js` importe ces fonctions et un CSS
4. Configurer Webpack avec :
   - Entry : `src/index.js`
   - Output : `dist/bundle.js`
   - `css-loader` + `style-loader`
   - `HtmlWebpackPlugin`
5. Vérifier que le build fonctionne

**Validation** : `npm run build` génère `dist/index.html` et `dist/bundle.js`

### Exercice 2 — Mode dev/prod (Niveau intermédiaire)

**Objectif** : Séparer les configurations dev et prod.

1. Reprendre l'exercice 1
2. Créer `webpack.common.js`, `webpack.dev.js`, `webpack.prod.js`
3. Dev : `style-loader`, `eval-source-map`, `webpack-dev-server`
4. Prod : `MiniCssExtractPlugin`, `source-map`, minification
5. Ajouter SASS au projet

**Validation** :
- `npm run dev` : serveur sur port 3000 avec HMR
- `npm run build` : CSS extrait dans `dist/css/`, JS minifié

### Exercice 3 — Code Splitting (Niveau avancé)

**Objectif** : Implémenter le lazy loading sur une application multi-pages.

1. Créer 3 "pages" comme modules JS
2. Implémenter un routeur simple basé sur `location.pathname`
3. Charger chaque page avec `import()` dynamique
4. Nommer les chunks avec les magic comments
5. Configurer `SplitChunksPlugin` pour extraire lodash en chunk séparé

**Validation** : 
- Bundle analysé avec `webpack-bundle-analyzer`
- Lodash dans un chunk séparé
- Chaque page dans son propre chunk
- La page courante est la seule chargée au démarrage

### Challenge — Migration Webpack vers Vite

**Objectif** : Migrer un projet Webpack 5 existant vers Vite.

1. Partir du projet de l'exercice 2 (avec SASS)
2. Créer une branche `feat/vite-migration`
3. Installer Vite et le plugin React
4. Créer `vite.config.js` équivalent à la config Webpack
5. Mettre à jour `package.json` (scripts, `type: "module"`)
6. Adapter les imports (variables d'environnement, assets)
7. Comparer les temps de démarrage

**Métriques à mesurer** :
- Temps de `npm run dev` (premier démarrage)
- Temps de `npm run build`
- Taille du bundle de production

---

## 18. Bonnes pratiques et pièges courants

### 18.1 Pièges courants Webpack

> [!warning] Pièges fréquents à éviter

**1. Oublier `path.resolve()` pour `output.path`**
```javascript
// ❌ Chemin relatif — peut casser selon le répertoire d'exécution
output: { path: 'dist' }

// ✅ Chemin absolu avec path.resolve
output: { path: path.resolve(__dirname, 'dist') }
```

**2. Ne pas exclure `node_modules` de babel-loader**
```javascript
// ❌ Transpiler node_modules = très lent et inutile
{ test: /\.js$/, use: 'babel-loader' }

// ✅ Exclure node_modules
{ test: /\.js$/, exclude: /node_modules/, use: 'babel-loader' }
```

**3. Confondre `devDependencies` et `dependencies`**
```bash
# ✅ Webpack et ses plugins en devDependencies
npm install --save-dev webpack webpack-cli

# ✅ Librairies runtime en dependencies
npm install react react-dom
```

**4. Ne pas nettoyer `dist/` entre les builds**
```javascript
// ✅ Webpack 5 : option clean intégrée
output: { clean: true }
```

**5. Oublier les `sideEffects` pour le tree shaking**
```json
// ✅ package.json — Déclarer explicitement
{
  "sideEffects": ["*.css", "*.scss"]
}
```

### 18.2 Optimisations de performance de build

```javascript
// webpack.config.js — Optimisations vitesse de build

module.exports = {
  // Cache Webpack 5 — réutilise le travail entre les builds
  cache: {
    type: 'filesystem',
    buildDependencies: {
      // Invalider le cache si webpack.config.js change
      config: [__filename],
    },
    cacheDirectory: path.resolve(__dirname, '.webpack-cache'),
  },

  // Paralléliser babel-loader
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'thread-loader', // Exécuter en parallèle
            options: {
              workers: require('os').cpus().length - 1,
            },
          },
          'babel-loader',
        ],
      },
    ],
  },

  // Réduire les résolutions inutiles
  resolve: {
    // Limiter les dossiers de recherche des modules
    modules: [path.resolve(__dirname, 'src'), 'node_modules'],

    // Réduire les extensions à résoudre
    extensions: ['.js', '.jsx'],
  },
};
```

### 18.3 Checklist de review pour une config Webpack

```
✅ Config dev et prod séparées (webpack-merge)
✅ Cache filesystem activé (Webpack 5)
✅ node_modules exclu de babel-loader
✅ HtmlWebpackPlugin configuré avec template
✅ MiniCssExtractPlugin en production
✅ source-map adapté au mode
✅ contenthash dans les noms de fichiers output
✅ output.clean: true
✅ SplitChunksPlugin configuré
✅ sideEffects déclaré dans package.json
✅ Variables d'env via DefinePlugin ou dotenv-webpack
✅ publicPath correct pour le déploiement
✅ performance.hints configuré
```

---

## Récapitulatif

```
BUNDLERS — SYNTHÈSE

┌─────────────────────────────────────────────────────────┐
│                    WEBPACK 5                            │
│  Le plus flexible et complet                            │
│  Loaders (transform) + Plugins (hooks lifecycle)        │
│  Code splitting + Module Federation                     │
│  Idéal : apps complexes, micro-frontends                │
└─────────────────────────────────────────────────────────┘
         │
         │ vs
         ▼
┌─────────────────────────────────────────────────────────┐
│                      VITE                               │
│  Ultra-rapide en dev (ESM natif + esbuild)              │
│  Build prod via Rollup                                  │
│  DX excellente, config minimale                         │
│  Idéal : nouvelles apps React/Vue/Svelte                │
└─────────────────────────────────────────────────────────┘
         │
         │ vs
         ▼
┌─────────────────────────────────────────────────────────┐
│                     ROLLUP                              │
│  Spécialisé bibliothèques NPM                           │
│  Meilleur tree shaking, formats multiples (ESM/CJS)     │
│  Idéal : écrire une lib publiée sur NPM                 │
└─────────────────────────────────────────────────────────┘
         │
         │ vs
         ▼
┌─────────────────────────────────────────────────────────┐
│                    esbuild                              │
│  10-100x plus rapide (écrit en Go)                      │
│  Limites : plugins, code splitting, CSS Modules         │
│  Idéal : transformateur dans une pipeline (Vite uses it)│
└─────────────────────────────────────────────────────────┘
```

Les concepts clés à retenir de ce cours :

| Concept | Pourquoi c'est important |
|---|---|
| **Loaders** | Transformer n'importe quel fichier en module JS |
| **Plugins** | Contrôler le cycle de build au niveau global |
| **Code splitting** | Ne charger que le code nécessaire à l'instant T |
| **Tree shaking** | Éliminer le code non utilisé (ESM + sideEffects) |
| **HMR** | Développement fluide sans perte d'état |
| **contenthash** | Cache navigateur optimal en production |
| **Module Federation** | Partager du code entre apps distinctes (micro-frontends) |
| **Vite vs Webpack** | Vite pour les nouveaux projets, Webpack pour la puissance/flexibilité |
