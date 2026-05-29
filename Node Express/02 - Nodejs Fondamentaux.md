# Node.js Fondamentaux

> [!tip] Analogie
> Imagine un serveur de restaurant ultra-efficace. Contrairement a un serveur classique qui reste debout devant toi pendant que le cuisinier prepare ton plat (bloquant), Node.js fonctionne comme un serveur qui prend ta commande, la transmet en cuisine, puis va immediatement prendre la commande d'une autre table. Quand ton plat est pret, le cuisinier lui fait signe et il te l'apporte. Un seul serveur, des dizaines de tables servies simultanement — c'est le modele non-bloquant de Node.js.

---

## Qu'est-ce que Node.js ?

Node.js est un **environnement d'execution JavaScript cote serveur**, construit sur le moteur V8 de Google Chrome. Il permet d'ecrire du code serveur en JavaScript — un langage que tu connais deja du navigateur.

Avant Node.js (sorti en 2009 par Ryan Dahl), JavaScript etait confine au navigateur. Node.js a brise cette barriere en extrayant V8 et en l'encapsulant dans un runtime capable d'interagir avec le systeme d'exploitation : fichiers, reseau, processus.

### Les trois piliers de Node.js

| Pilier | Description | Pourquoi ca compte |
|--------|-------------|-------------------|
| **Moteur V8** | Compile JS en code machine natif | Performances proches du C dans certains cas |
| **Single-threaded** | Un seul thread d'execution principal | Pas de concurrence complexe, pas de deadlocks |
| **Non-blocking I/O** | Les operations lentes ne bloquent pas le fil | Gerer des milliers de connexions avec peu de RAM |

### V8 : le moteur sous le capot

V8 est le moteur JavaScript developpe par Google pour Chrome. Il ne se contente pas d'interpreter le JavaScript — il le **compile a la volee** (JIT compilation) en code machine optimise. C'est pour ca que JavaScript moderne est rapide.

Node.js embarque V8 et y ajoute :
- Des **bindings C++** pour acceder au systeme de fichiers, reseau, etc.
- La bibliotheque **libuv** qui fournit la boucle evenementielle et le pool de threads I/O
- Un ensemble de **modules natifs** (fs, http, path, crypto...)

```
  JavaScript Code
        |
        v
  [ V8 Engine ]  <-- compile JS en bytecode, puis en machine code
        |
  [ Node.js Bindings ] (C++)
        |
  [ libuv ]  <-- event loop, thread pool, async I/O
        |
  [ Systeme d'Exploitation ]
```

### Pourquoi single-threaded ?

Le modele multi-thread traditionnel (Apache, Java EE) cree un thread par connexion. C'est puissant mais couteux : chaque thread consomme de la memoire (~1-2 MB), et les changements de contexte CPU s'accumulent.

Node.js prend le pari inverse : **un seul thread**, mais jamais bloque. Au lieu d'attendre une reponse base de donnees, il enregistre un callback et passe a la tache suivante. Quand la BDD repond, le callback est execute.

> [!warning] Attention : single-threaded n'est pas magique
> Si tu executes du code CPU-intensif (chiffrement, calcul complexe, traitement d'image), tu **bloques** le seul thread, et personne d'autre ne peut etre servi pendant ce temps. Pour ces cas, utilise les Worker Threads (voir [[Node Express/03 - Nodejs Avance]]).

---

## L'Event Loop expliquee visuellement

C'est le mecanisme central de Node.js. Comprendre l'event loop, c'est comprendre POURQUOI Node.js fonctionne comme il fonctionne.

### Les composants

```
  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │   TON CODE JAVASCRIPT                               │
  │                                                     │
  │   ┌──────────────────┐                              │
  │   │   Call Stack      │  <-- une seule pile         │
  │   │                  │      d'execution             │
  │   │  main()          │                              │
  │   │  setTimeout cb   │                              │
  │   │  fs.readFile cb  │                              │
  │   └────────┬─────────┘                              │
  │            │                                        │
  └────────────┼────────────────────────────────────────┘
               │
               │ quand le call stack est VIDE
               │
  ┌────────────▼────────────────────────────────────────┐
  │                                                     │
  │   EVENT LOOP                                        │
  │                                                     │
  │   Phases (dans l'ordre) :                           │
  │                                                     │
  │   1. timers          --> setTimeout, setInterval    │
  │   2. pending I/O     --> erreurs reseau reportees   │
  │   3. idle, prepare   --> usage interne              │
  │   4. poll            --> attente de nouveaux I/O    │
  │   5. check           --> setImmediate               │
  │   6. close callbacks --> socket.on('close', ...)    │
  │                                                     │
  │   Entre chaque phase :                              │
  │   ┌─ Microtask Queue (PRIORITAIRE) ─────────────┐  │
  │   │  process.nextTick()                          │  │
  │   │  Promise.then() / async/await                │  │
  │   └──────────────────────────────────────────────┘  │
  └─────────────────────────────────────────────────────┘
               │
  ┌────────────▼────────────────────────────────────────┐
  │                                                     │
  │   libuv Thread Pool (4 threads par defaut)          │
  │                                                     │
  │   fs.readFile   DNS lookup   crypto.pbkdf2          │
  │   fs.writeFile  zlib         ...                    │
  │                                                     │
  └─────────────────────────────────────────────────────┘
```

### Trace pas a pas d'un exemple

```javascript
console.log('1 - debut');

setTimeout(() => {
  console.log('4 - setTimeout (macrotask)');
}, 0);

Promise.resolve().then(() => {
  console.log('3 - Promise.then (microtask)');
});

process.nextTick(() => {
  console.log('2 - nextTick (microtask prioritaire)');
});

console.log('5 - fin du script');
// Attends... ce console.log s'affiche AVANT le 3 et le 4 ?
```

Resultat d'execution :
```
1 - debut
5 - fin du script
2 - nextTick (microtask prioritaire)
3 - Promise.then (microtask)
4 - setTimeout (macrotask)
```

Voici pourquoi :

```
ETAPE 1 : Execution synchrone du script principal
  Call Stack: [main script]
  → console.log('1') s'execute
  → setTimeout enregistre un callback dans le timer OS (pas dans le call stack)
  → Promise.resolve().then() enregistre dans la microtask queue
  → process.nextTick() enregistre dans la microtask queue (haute priorite)
  → console.log('5') s'execute
  Call Stack vide.

ETAPE 2 : Microtasks (AVANT de passer a la phase suivante de l'event loop)
  → process.nextTick callback : console.log('2')
  → Promise.then callback : console.log('3')
  Microtask queue vide.

ETAPE 3 : Phase timers de l'event loop
  → setTimeout callback (timer expire) : console.log('4')
```

> [!info] nextTick vs Promise
> `process.nextTick()` est execute AVANT les Promises, meme si elles sont deja resolues. C'est un mecanisme Node.js-specifique pour diferer une execution a la fin de l'operation courante, avant tout autre I/O. Utilise-le avec parcimonie — une recursion `nextTick` peut affamer la boucle.

### Pourquoi le setTimeout 0ms n'est pas instantane

`setTimeout(() => {}, 0)` ne signifie pas "execute immediatement". Ca signifie "execute au minimum apres 0ms". En pratique, la phase `timers` n'est atteinte qu'apres que tout le code synchrone et toutes les microtasks aient ete traites.

---

## Modules : CommonJS vs ES Modules

Node.js supporte deux systemes de modules. Comprendre leur difference evite des heures de confusion.

### CommonJS (CJS) — le systeme historique

CommonJS est le systeme original de Node.js. Chaque fichier est un module independant avec son propre scope.

```javascript
// math.js
const PI = 3.14159;

function aire(rayon) {
  return PI * rayon * rayon;
}

function perimetre(rayon) {
  return 2 * PI * rayon;
}

// Exporter plusieurs elements
module.exports = {
  aire,
  perimetre,
  PI
};
```

```javascript
// app.js
const math = require('./math');
// ou avec destructuring
const { aire, perimetre } = require('./math');

console.log(aire(5));     // 78.53975
console.log(perimetre(5)); // 31.4159

// require peut aussi charger des modules natifs
const fs = require('fs');
const path = require('path');

// ou des modules npm
const axios = require('axios'); // apres npm install axios
```

**Caracteristiques CJS :**
- `require()` est **synchrone** — le fichier est charge et execute immediatement
- Les modules sont **caches** apres le premier `require()` — re-importer le meme module retourne l'objet deja charge
- Fonctionne bien pour les scripts CLI et le code serveur classique
- Extension de fichier `.js` par defaut (ou `.cjs` pour forcer CJS)

```javascript
// Verifier le cache
const a = require('./math');
const b = require('./math');
console.log(a === b); // true — meme reference, module charge une seule fois
```

### ES Modules (ESM) — le standard moderne

ES Modules est le systeme standardise par ECMAScript 2015. Il est supporte nativement dans Node.js depuis la v12 (stable v14+).

```javascript
// math.mjs (ou .js avec "type": "module" dans package.json)
export const PI = 3.14159;

export function aire(rayon) {
  return PI * rayon * rayon;
}

export function perimetre(rayon) {
  return 2 * PI * rayon;
}

// Export par defaut
export default function info() {
  return 'Module de geometrie circulaire';
}
```

```javascript
// app.mjs
import { aire, perimetre, PI } from './math.mjs';
import info from './math.mjs'; // import par defaut

console.log(info());
console.log(aire(5));

// Import dynamique (utile pour le lazy loading)
const module = await import('./math.mjs');
console.log(module.aire(3));
```

### Tableau comparatif CJS vs ESM

| Critere | CommonJS | ES Modules |
|---------|----------|-----------|
| Syntaxe | `require()` / `module.exports` | `import` / `export` |
| Chargement | Synchrone | Asynchrone (permet l'optimisation) |
| Top-level await | Non | Oui |
| Tree-shaking | Non (bundlers ne peuvent pas analyser) | Oui (analyse statique possible) |
| Extension par defaut | `.js` | `.mjs` ou `.js` avec `"type": "module"` |
| `__dirname`, `__filename` | Disponibles | Non disponibles (utiliser `import.meta.url`) |
| Interop | Peut importer ESM avec `import()` dynamique | Peut importer CJS avec `import` statique |
| Usage aujourd'hui | Legacy, scripts CLI | Projets modernes, bibliotheques |

```javascript
// ESM : equivalent de __dirname
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

> [!warning] Melanger CJS et ESM
> Tu ne peux pas utiliser `require()` dans un fichier ESM, ni `import` statique dans un fichier CJS. Node.js detecte le type selon l'extension (`.mjs` = ESM, `.cjs` = CJS) ou via le champ `"type"` dans `package.json`. Dans un projet existant, verifie ce champ avant d'adopter ESM.

---

## npm et package.json

npm (Node Package Manager) est l'outil de gestion de dependances de Node.js. Avec plus de 2 millions de packages, c'est le plus grand registre de logiciels open source au monde.

### Initialiser un projet

```bash
# Creer un nouveau dossier et initialiser
mkdir mon-projet
cd mon-projet
npm init          # mode interactif (pose des questions)
npm init -y       # accepter toutes les valeurs par defaut
```

### Anatomie d'un package.json

```json
{
  "name": "mon-projet",
  "version": "1.0.0",
  "description": "Un projet Node.js de demonstration",
  "main": "index.js",
  "type": "commonjs",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "build": "tsc",
    "lint": "eslint src/"
  },
  "keywords": ["node", "demo"],
  "author": "Ton Nom <email@example.com>",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.2",
    "axios": "^1.4.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.6.1",
    "eslint": "^8.45.0"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### Installer des packages

```bash
# Installer une dependance de production
npm install express
npm i express        # raccourci

# Installer une dependance de developpement uniquement
npm install --save-dev nodemon
npm i -D nodemon     # raccourci

# Installer globalement (outils CLI)
npm install -g typescript
npm i -g ts-node

# Installer toutes les dependances d'un projet existant
npm install

# Supprimer un package
npm uninstall express

# Mettre a jour les packages
npm update
npm update express

# Voir les packages installes
npm list
npm list --depth=0   # seulement le niveau racine
```

### Comprendre le versioning semver

npm utilise la **Semantic Versioning** (semver) : `MAJOR.MINOR.PATCH`

```
1.4.2
│ │ └── PATCH : correction de bug (retrocompatible)
│ └──── MINOR : nouvelle fonctionnalite (retrocompatible)
└────── MAJOR : changement cassant (breaking change)
```

| Prefixe | Signification | Exemple | Installe |
|---------|--------------|---------|---------|
| `^` | Compatible avec la MINOR | `^4.18.2` | >= 4.18.2 < 5.0.0 |
| `~` | Compatible avec le PATCH | `~4.18.2` | >= 4.18.2 < 4.19.0 |
| `*` | N'importe quelle version | `*` | La plus recente |
| (rien) | Exactement cette version | `4.18.2` | 4.18.2 uniquement |

```bash
# package-lock.json : verrouille les versions exactes installees
# Toujours committer ce fichier en equipe (garantit le meme environnement)
# Ne jamais committer node_modules/
```

### Scripts npm

Les scripts dans `package.json` sont des raccourcis de commandes :

```json
"scripts": {
  "start": "node src/index.js",
  "dev": "nodemon src/index.js --watch src",
  "test": "jest --coverage",
  "test:watch": "jest --watch",
  "build": "tsc && cp -r public dist/",
  "lint": "eslint src/ --ext .js,.ts",
  "format": "prettier --write src/"
}
```

```bash
npm run start    # ou npm start (exception pour "start")
npm run dev
npm run test     # ou npm test (exception pour "test")
npm run build
```

> [!info] npx — executer sans installer globalement
> `npx create-react-app mon-app` telecharge et execute le package une seule fois, sans l'installer globalement. Utile pour les outils utilises rarement. Node.js 18+ inclut npx par defaut.

---

## Module `fs` : interagir avec le systeme de fichiers

Le module `fs` (File System) permet de lire, ecrire, supprimer et manipuler des fichiers et dossiers.

### Trois API dans le meme module

```javascript
const fs = require('fs');         // callbacks
const fsSync = require('fs');     // methodes synchrones (meme module, suffixe Sync)
const fsPromises = require('fs/promises'); // Promises (Node.js 14+)
```

### Lire un fichier

```javascript
const fs = require('fs');
const fsPromises = require('fs/promises');
const path = require('path');

// --- VERSION CALLBACK (ancienne) ---
fs.readFile(path.join(__dirname, 'data.txt'), 'utf8', (err, contenu) => {
  if (err) {
    console.error('Erreur de lecture:', err.message);
    return;
  }
  console.log(contenu);
});

// --- VERSION SYNCHRONE (bloquante — eviter en production) ---
try {
  const contenu = fs.readFileSync(path.join(__dirname, 'data.txt'), 'utf8');
  console.log(contenu);
} catch (err) {
  console.error('Erreur:', err.message);
}

// --- VERSION PROMISES (recommandee) ---
async function lireFichier() {
  try {
    const contenu = await fsPromises.readFile(
      path.join(__dirname, 'data.txt'),
      'utf8'
    );
    console.log(contenu);
  } catch (err) {
    if (err.code === 'ENOENT') {
      console.error('Fichier introuvable');
    } else {
      throw err;
    }
  }
}

lireFichier();
```

### Ecrire un fichier

```javascript
const fsPromises = require('fs/promises');

async function exempleEcriture() {
  // Creer ou remplacer un fichier
  await fsPromises.writeFile('output.txt', 'Bonjour Node.js !', 'utf8');
  
  // Ajouter du contenu sans effacer (append)
  await fsPromises.appendFile('log.txt', `[${new Date().toISOString()}] Action\n`);
  
  // Ecrire du JSON
  const data = { nom: 'Alice', age: 30, actif: true };
  await fsPromises.writeFile('data.json', JSON.stringify(data, null, 2));
}
```

### Lister et manipuler le systeme de fichiers

```javascript
const fsPromises = require('fs/promises');

async function operations() {
  // Lister un dossier
  const fichiers = await fsPromises.readdir('./src');
  console.log(fichiers);

  // Lister avec types (fichier ou dossier)
  const entrees = await fsPromises.readdir('./src', { withFileTypes: true });
  entrees.forEach(entree => {
    const type = entree.isDirectory() ? 'DOSSIER' : 'FICHIER';
    console.log(`[${type}] ${entree.name}`);
  });

  // Creer un dossier
  await fsPromises.mkdir('./logs', { recursive: true }); // recursive: evite l'erreur si existe

  // Renommer / deplacer
  await fsPromises.rename('./ancien.txt', './nouveau.txt');

  // Supprimer un fichier
  await fsPromises.unlink('./temp.txt');

  // Supprimer un dossier (vide)
  await fsPromises.rmdir('./vieux-dossier');

  // Supprimer un dossier et son contenu
  await fsPromises.rm('./build', { recursive: true, force: true });

  // Obtenir des informations sur un fichier
  const stats = await fsPromises.stat('./package.json');
  console.log('Taille:', stats.size, 'octets');
  console.log('Modifie le:', stats.mtime);
  console.log('Est un dossier:', stats.isDirectory());
}
```

> [!warning] sync vs async en production
> N'utilise JAMAIS les methodes synchrones (`readFileSync`, `writeFileSync`) dans un serveur en production. Elles bloquent le thread principal — pendant qu'un fichier est lu, TOUTES les autres requetes attendent. Reserve les methodes sync au demarrage de l'application (chargement de config) ou aux scripts CLI one-shot.

---

## Module `path` : manipuler les chemins de fichiers

Le module `path` resout un probleme classique : les separateurs de chemin different entre Windows (`\`) et Unix (`/`). `path` gere ca automatiquement.

```javascript
const path = require('path');

// --- path.join : concatener des segments ---
const chemin1 = path.join('users', 'alice', 'documents', 'fichier.txt');
// → 'users/alice/documents/fichier.txt' (Unix)
// → 'users\alice\documents\fichier.txt' (Windows)

const chemin2 = path.join(__dirname, '..', 'config', 'app.json');
// Monte d'un niveau depuis le fichier courant, puis va dans config/

// --- path.resolve : chemin absolu ---
const abs = path.resolve('fichier.txt');
// → '/home/alice/projet/fichier.txt' (chemin absolu depuis le cwd)

const absExplicite = path.resolve('/base', 'sous-dossier', 'fichier.txt');
// → '/base/sous-dossier/fichier.txt'

// --- path.dirname : dossier parent ---
console.log(path.dirname('/home/alice/documents/fichier.txt'));
// → '/home/alice/documents'

console.log(path.dirname(__filename)); // identique a __dirname

// --- path.basename : nom du fichier ---
console.log(path.basename('/home/alice/documents/fichier.txt'));
// → 'fichier.txt'

console.log(path.basename('/home/alice/documents/fichier.txt', '.txt'));
// → 'fichier' (sans l'extension)

// --- path.extname : extension ---
console.log(path.extname('image.jpg'));    // → '.jpg'
console.log(path.extname('archive.tar.gz')); // → '.gz'
console.log(path.extname('README'));       // → '' (pas d'extension)

// --- path.parse : decomposer un chemin ---
const info = path.parse('/home/alice/documents/fichier.txt');
console.log(info);
// {
//   root: '/',
//   dir: '/home/alice/documents',
//   base: 'fichier.txt',
//   ext: '.txt',
//   name: 'fichier'
// }

// --- path.format : reconstruire depuis les composants ---
const chemin = path.format({
  dir: '/home/alice/documents',
  name: 'fichier',
  ext: '.txt'
});
// → '/home/alice/documents/fichier.txt'

// --- path.sep : separateur du systeme ---
console.log(path.sep); // '/' sur Unix, '\' sur Windows

// --- Cas pratique : construire des chemins robustement ---
const CONFIG_PATH = path.join(__dirname, '..', 'config', 'database.json');
const LOGS_DIR = path.resolve(process.cwd(), 'logs');
```

> [!info] __dirname et __filename
> `__dirname` : chemin absolu du **dossier** contenant le fichier courant.
> `__filename` : chemin absolu du **fichier** courant lui-meme.
> Ces variables sont magiquement injectees par Node.js dans chaque module CJS — elles n'existent pas en ESM (utiliser `import.meta.url` a la place).

---

## Streams et Buffers

Les streams sont l'un des concepts les plus puissants et les plus mal compris de Node.js. Ils permettent de traiter des donnees en **morceaux** plutot qu'en une seule fois.

### Pourquoi les streams ?

```
SANS STREAM (probleme pour les gros fichiers) :
  Fichier 2 GB
       │
       ▼
  fs.readFile()  --> RAM : TOUT le fichier en memoire (2 GB)
       │
       ▼
  Traitement puis envoi

AVEC STREAM (efficace) :
  Fichier 2 GB
       │
       ▼
  Chunk 64 KB --> Traitement --> Envoi
  Chunk 64 KB --> Traitement --> Envoi
  Chunk 64 KB --> Traitement --> Envoi
  ...
  RAM utilisee : ~64 KB seulement
```

### Les 4 types de streams

| Type | Description | Exemple |
|------|-------------|---------|
| **Readable** | Source de donnees | `fs.createReadStream()`, `http.IncomingMessage` |
| **Writable** | Destination de donnees | `fs.createWriteStream()`, `http.ServerResponse` |
| **Duplex** | Readable ET Writable | `net.Socket` (TCP) |
| **Transform** | Duplex qui transforme les donnees | `zlib.createGzip()`, `crypto.createCipher()` |

### Readable Stream — lire en flux

```javascript
const fs = require('fs');

const readable = fs.createReadStream('./gros-fichier.csv', {
  encoding: 'utf8',
  highWaterMark: 64 * 1024 // taille des chunks : 64 KB
});

let lignesCount = 0;

readable.on('data', (chunk) => {
  // Appele pour chaque chunk
  lignesCount += chunk.split('\n').length;
  console.log(`Chunk recu : ${chunk.length} caracteres`);
});

readable.on('end', () => {
  console.log(`Lecture terminee. Lignes estimees : ${lignesCount}`);
});

readable.on('error', (err) => {
  console.error('Erreur de lecture:', err.message);
});
```

### Writable Stream — ecrire en flux

```javascript
const fs = require('fs');

const writable = fs.createWriteStream('./output.txt', { flags: 'a' }); // 'a' = append

// Ecrire des donnees
writable.write('Premiere ligne\n');
writable.write('Deuxieme ligne\n');

// Signaler la fin
writable.end('Derniere ligne\n', () => {
  console.log('Ecriture terminee');
});

writable.on('error', (err) => {
  console.error('Erreur ecriture:', err.message);
});
```

### pipe() — connecter des streams

`pipe()` est la methode la plus elegante pour connecter un readable a un writable.

```javascript
const fs = require('fs');
const zlib = require('zlib');

// Compresser un fichier en streaming — aucun fichier entier en RAM
const source = fs.createReadStream('./archive.tar');
const gzip = zlib.createGzip();           // Transform stream
const destination = fs.createWriteStream('./archive.tar.gz');

// Connecter : source --> compression --> fichier compresse
source
  .pipe(gzip)           // passe les chunks dans le compresseur
  .pipe(destination)    // ecrit les chunks comprimes
  .on('finish', () => {
    console.log('Compression terminee !');
  })
  .on('error', (err) => {
    console.error('Erreur:', err.message);
  });
```

### Transform Stream personnalise

```javascript
const { Transform } = require('stream');

// Un transform stream qui met tout en majuscules
class MajusculesTransform extends Transform {
  _transform(chunk, encoding, callback) {
    // Transformer le chunk et le passer au stream suivant
    const transforme = chunk.toString().toUpperCase();
    this.push(transforme);
    callback(); // signale que la transformation est terminee
  }
}

// Utilisation
const fs = require('fs');
const majuscules = new MajusculesTransform();

fs.createReadStream('./texte.txt')
  .pipe(majuscules)
  .pipe(fs.createWriteStream('./texte-maj.txt'));
```

### Buffers : les donnees brutes

Un `Buffer` est une zone de memoire fixe contenant des octets bruts. C'est la representation basse-niveau des donnees binaires en Node.js.

```javascript
// Creer un buffer
const buf1 = Buffer.from('Bonjour'); // depuis une string
const buf2 = Buffer.from([0x42, 0x6f, 0x6e]); // depuis des octets hexa
const buf3 = Buffer.alloc(10); // buffer de 10 octets, initialise a 0
const buf4 = Buffer.allocUnsafe(10); // buffer non initialise (plus rapide, moins sur)

// Lire un buffer
console.log(buf1.toString());          // 'Bonjour'
console.log(buf1.toString('hex'));     // '426f6e6a6f7572'
console.log(buf1.toString('base64')); // 'Qm9uam91cg=='

// Taille
console.log(buf1.length); // 7 (octets, pas caracteres)

// Convertir
const str = buf1.toString('utf8');
const bufFromStr = Buffer.from(str, 'utf8');

// Les streams travaillent avec des Buffers
// Quand tu lis un stream sans encoding, tu recois des Buffers
const readable = fs.createReadStream('./image.png'); // pas d'encoding
readable.on('data', (chunk) => {
  console.log(chunk instanceof Buffer); // true
  console.log(chunk.length); // taille en octets
});
```

> [!example] Cas pratique : parser un CSV en streaming
> ```javascript
> const fs = require('fs');
> const readline = require('readline');
> 
> async function parserCSV(chemin) {
>   const stream = fs.createReadStream(chemin);
>   const rl = readline.createInterface({ input: stream });
>   
>   let premiereLigne = true;
>   let headers = [];
>   const resultats = [];
>   
>   for await (const ligne of rl) {
>     const colonnes = ligne.split(',');
>     if (premiereLigne) {
>       headers = colonnes;
>       premiereLigne = false;
>     } else {
>       const objet = {};
>       headers.forEach((h, i) => objet[h] = colonnes[i]);
>       resultats.push(objet);
>     }
>   }
>   
>   return resultats;
> }
> 
> parserCSV('./data.csv').then(data => console.log(data));
> ```

---

## EventEmitter : le patron observateur

`EventEmitter` est la classe de base qui alimente une grande partie de l'ecosysteme Node.js. Les streams, les serveurs HTTP, les sockets TCP — tous heritent d'`EventEmitter`.

### API de base

```javascript
const { EventEmitter } = require('events');

// Creer une instance
const emitter = new EventEmitter();

// Ecouter un evenement
emitter.on('message', (data) => {
  console.log('Message recu:', data);
});

// Ecouter une seule fois
emitter.once('connexion', (userId) => {
  console.log('Premiere connexion de:', userId);
  // Ce handler sera automatiquement supprime apres le premier appel
});

// Emettre un evenement
emitter.emit('message', 'Bonjour !');
emitter.emit('message', { texte: 'World', timestamp: Date.now() });
emitter.emit('connexion', 'alice');
emitter.emit('connexion', 'bob'); // le handler once n'est plus la

// Supprimer un listener
function monHandler(data) {
  console.log(data);
}
emitter.on('data', monHandler);
emitter.removeListener('data', monHandler); // supprimer un handler specifique
emitter.off('data', monHandler); // alias de removeListener

// Supprimer tous les listeners d'un evenement
emitter.removeAllListeners('message');

// Lister les evenements enregistres
console.log(emitter.eventNames());
```

### Creer sa propre classe avec EventEmitter

```javascript
const { EventEmitter } = require('events');

class Minuterie extends EventEmitter {
  constructor(dureeMs) {
    super();
    this.dureeMs = dureeMs;
    this.compteur = 0;
    this._intervalId = null;
  }

  demarrer() {
    if (this._intervalId) return;

    this._intervalId = setInterval(() => {
      this.compteur++;
      this.emit('tick', this.compteur);

      if (this.compteur * 1000 >= this.dureeMs) {
        this.emit('fin', this.compteur);
        this.arreter();
      }
    }, 1000);

    this.emit('debut');
    return this; // chaining
  }

  arreter() {
    if (this._intervalId) {
      clearInterval(this._intervalId);
      this._intervalId = null;
    }
    return this;
  }
}

// Utilisation
const timer = new Minuterie(5000); // 5 secondes

timer
  .on('debut', () => console.log('Minuterie demarree'))
  .on('tick', (n) => console.log(`Seconde ${n}`))
  .on('fin', (total) => console.log(`Termine apres ${total} secondes`));

timer.demarrer();
```

### Gestion des erreurs avec EventEmitter

```javascript
const { EventEmitter } = require('events');

const emitter = new EventEmitter();

// IMPORTANT : si 'error' est emis sans listener, Node.js CRASHE
emitter.on('error', (err) => {
  console.error('Erreur capturee:', err.message);
});

// Simuler une erreur
emitter.emit('error', new Error('Quelque chose a mal tourne'));
```

> [!warning] L'evenement 'error' est special
> Si tu emets un evenement `'error'` sur un EventEmitter qui n'a pas de listener pour `'error'`, Node.js lance une exception non capturee et **plante le processus**. Toujours enregistrer un handler `error` sur tes emitters critiques.

---

## Module `http` : creer un serveur sans Express

Comprendre `http` natif te rend independant de tout framework et t'aide a comprendre ce que fait Express sous le capot.

### Serveur basique

```javascript
const http = require('http');

const serveur = http.createServer((req, res) => {
  // req : http.IncomingMessage (Readable stream)
  // res : http.ServerResponse (Writable stream)

  console.log(`${req.method} ${req.url}`);

  // Definir les headers de reponse
  res.setHeader('Content-Type', 'text/plain; charset=utf-8');
  res.setHeader('X-Custom-Header', 'Node.js natif');

  // Envoyer la reponse
  res.writeHead(200); // code HTTP (optionnel si pas de headers supplementaires)
  res.end('Bonjour depuis Node.js !');
});

serveur.listen(3000, '127.0.0.1', () => {
  console.log('Serveur ecoute sur http://127.0.0.1:3000');
});
```

### Routing manuel et parsing du corps

```javascript
const http = require('http');
const { URL } = require('url');
const fsPromises = require('fs/promises');
const path = require('path');

const serveur = http.createServer(async (req, res) => {
  // Parser l'URL avec query strings
  const url = new URL(req.url, `http://${req.headers.host}`);
  const pathname = url.pathname;
  const method = req.method;

  // Helper : envoyer du JSON
  function sendJSON(statusCode, data) {
    const body = JSON.stringify(data);
    res.writeHead(statusCode, {
      'Content-Type': 'application/json',
      'Content-Length': Buffer.byteLength(body)
    });
    res.end(body);
  }

  // Helper : lire le corps de la requete
  async function readBody() {
    return new Promise((resolve, reject) => {
      let body = '';
      req.on('data', (chunk) => { body += chunk.toString(); });
      req.on('end', () => {
        try {
          resolve(JSON.parse(body));
        } catch {
          resolve(body);
        }
      });
      req.on('error', reject);
    });
  }

  // --- Routing ---
  try {
    if (pathname === '/' && method === 'GET') {
      sendJSON(200, { message: 'API Node.js natif', version: '1.0.0' });

    } else if (pathname === '/utilisateurs' && method === 'GET') {
      const utilisateurs = [
        { id: 1, nom: 'Alice' },
        { id: 2, nom: 'Bob' }
      ];
      sendJSON(200, utilisateurs);

    } else if (pathname === '/utilisateurs' && method === 'POST') {
      const body = await readBody();
      const nouvelUtilisateur = { id: Date.now(), ...body };
      sendJSON(201, nouvelUtilisateur);

    } else if (pathname.startsWith('/static/')) {
      // Servir des fichiers statiques
      const fichierPath = path.join(__dirname, 'public', pathname.replace('/static', ''));
      try {
        const contenu = await fsPromises.readFile(fichierPath);
        res.writeHead(200);
        res.end(contenu);
      } catch {
        sendJSON(404, { erreur: 'Fichier introuvable' });
      }

    } else {
      sendJSON(404, { erreur: 'Route introuvable', path: pathname });
    }

  } catch (err) {
    console.error('Erreur serveur:', err);
    sendJSON(500, { erreur: 'Erreur interne du serveur' });
  }
});

serveur.on('error', (err) => {
  if (err.code === 'EADDRINUSE') {
    console.error('Port deja utilise');
  } else {
    throw err;
  }
});

serveur.listen(3000, () => {
  console.log('Serveur : http://localhost:3000');
});
```

> [!info] C'est ce que fait Express
> Express.js n'est pas magique — il enveloppe exactement ce code `http.createServer()` avec un systeme de routing plus elegant, des middlewares et des helpers `res.json()`, `res.send()`. Voir [[Node Express/01 - Express.js et NestJS]] pour la suite.

---

## Module `process`

`process` est un objet global disponible dans tout fichier Node.js sans `require()`. Il represente le processus Node.js en cours d'execution.

```javascript
// --- process.argv : arguments de la ligne de commande ---
// node script.js --port 3000 --env production
console.log(process.argv);
// [
//   '/usr/bin/node',       // [0] chemin de l'executable Node
//   '/home/alice/script.js', // [1] chemin du script
//   '--port',              // [2] premier argument
//   '3000',                // [3]
//   '--env',               // [4]
//   'production'           // [5]
// ]

// Parser les arguments simplement
const args = process.argv.slice(2); // supprimer node et script path
const portIndex = args.indexOf('--port');
const port = portIndex !== -1 ? parseInt(args[portIndex + 1]) : 3000;

// --- process.env : variables d'environnement ---
const NODE_ENV = process.env.NODE_ENV || 'development';
const PORT = parseInt(process.env.PORT) || 3000;
const DB_URL = process.env.DATABASE_URL;

if (!DB_URL) {
  console.error('DATABASE_URL manquante dans les variables d\'environnement');
  process.exit(1);
}

// --- process.exit : quitter le processus ---
process.exit(0);  // succes (code 0 = OK)
process.exit(1);  // echec (code non-zero = erreur)

// --- process.stdout / process.stderr ---
process.stdout.write('Message sans saut de ligne');
process.stderr.write('Erreur\n');

// console.log utilise process.stdout en interne
// console.error utilise process.stderr

// --- process.cwd() : repertoire de travail courant ---
console.log(process.cwd());
// '/home/alice/projets/mon-app'
// Differend de __dirname (dossier du fichier) si lance depuis ailleurs

// --- process.platform : systeme d'exploitation ---
console.log(process.platform); // 'linux', 'darwin', 'win32'

// --- process.version / process.versions ---
console.log(process.version);  // 'v20.5.1'
console.log(process.versions); // { node: '20.5.1', v8: '11.3.244.8', ... }

// --- process.memoryUsage() ---
const mem = process.memoryUsage();
console.log(`Heap utilise : ${Math.round(mem.heapUsed / 1024 / 1024)} MB`);
console.log(`Heap total : ${Math.round(mem.heapTotal / 1024 / 1024)} MB`);

// --- Signaux OS ---
process.on('SIGTERM', () => {
  console.log('Signal SIGTERM recu — arret propre');
  // Fermer les connexions, vider les buffers...
  process.exit(0);
});

process.on('SIGINT', () => {
  console.log('\nCtrl+C detecte — arret propre');
  process.exit(0);
});

// --- Erreurs non capturees ---
process.on('uncaughtException', (err) => {
  console.error('Exception non capturee:', err);
  process.exit(1); // Redemarrer proprement via un process manager
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Promise non geree:', promise, 'raison:', reason);
  process.exit(1);
});
```

> [!example] Script CLI avec process.argv
> ```javascript
> // greeter.js
> const nom = process.argv[2] || 'monde';
> const fois = parseInt(process.argv[3]) || 1;
> 
> for (let i = 0; i < fois; i++) {
>   console.log(`Bonjour, ${nom} !`);
> }
> // node greeter.js Alice 3
> // → Bonjour, Alice !
> // → Bonjour, Alice !
> // → Bonjour, Alice !
> ```

---

## Node.js vs Navigateur

Meme si les deux executent JavaScript via V8, les environnements sont tres differents.

### Ce qui n'existe que dans le navigateur

| API | Description |
|-----|-------------|
| `window` / `document` | DOM, manipulation du HTML |
| `localStorage` / `sessionStorage` | Stockage cote client |
| `fetch` (historiquement) | Maintenant disponible Node.js 18+ |
| `XMLHttpRequest` | Requetes HTTP navigateur |
| `navigator` | Informations navigateur/appareil |
| `alert()`, `confirm()`, `prompt()` | Dialogues UI |
| `requestAnimationFrame()` | Boucle de rendu graphique |
| `canvas` / `WebGL` | Rendu graphique 2D/3D |
| `ServiceWorker` | Cache offline, push notifications |

### Ce qui n'existe que dans Node.js

| API / Module | Description |
|--------------|-------------|
| `fs` | Acces au systeme de fichiers |
| `http` / `https` | Serveur HTTP natif |
| `net` / `dgram` | TCP / UDP bas niveau |
| `child_process` | Lancer des sous-processus |
| `cluster` | Multi-processus sur plusieurs CPUs |
| `worker_threads` | Multi-threading JavaScript |
| `os` | Informations systeme (CPU, memoire, interfaces reseau) |
| `crypto` | Cryptographie (hash, chiffrement, HMAC) |
| `process` | Controle du processus, env variables |
| `__dirname`, `__filename` | Chemins du fichier courant (CJS) |
| `Buffer` | Donnees binaires brutes |
| `require()` | Systeme de modules CJS |

### Ce qui existe dans LES DEUX

| API | Note |
|-----|------|
| `console` | API identique |
| `setTimeout`, `setInterval`, `clearTimeout` | Comportement equivalent |
| `Promise`, `async/await` | Identique |
| `JSON` | Identique |
| `Math`, `Date`, `RegExp` | Identique |
| `fetch` | Node.js 18+ nativement (compatible browsers) |
| `URL`, `URLSearchParams` | Identique depuis Node.js 10 |
| `crypto.getRandomValues()` | Disponible dans les deux (impl differente) |
| `AbortController` | Node.js 15+ |
| `EventTarget` | Node.js 14.5+ (alternative a EventEmitter) |

```javascript
// Code isomorphique (fonctionne dans les deux environnements)
async function fetchDonnees(url) {
  const reponse = await fetch(url);
  if (!reponse.ok) {
    throw new Error(`HTTP ${reponse.status}`);
  }
  return reponse.json();
}

// Ce code fonctionne dans Node.js 18+ ET dans un navigateur moderne
const data = await fetchDonnees('https://api.example.com/data');
```

> [!info] Pourquoi ces differences ?
> Le navigateur protege le systeme de l'utilisateur — il serait dangereux qu'une page web puisse lire tes fichiers ou lancer des processus. Node.js tourne sous ton controle en serveur, donc il a acces complet au systeme. C'est un choix de securite, pas une limitation technique.

---

## Asynchronisme : patterns disponibles

Pour approfondir les Promises et async/await, consulte [[JavaScript/03 - JavaScript Asynchrone]].

### Resume des 3 patterns

```javascript
// 1. CALLBACKS (ancien — eviter dans le nouveau code)
fs.readFile('./data.txt', 'utf8', (err, data) => {
  if (err) { /* gerer */ return; }
  fs.writeFile('./output.txt', data.toUpperCase(), (err) => {
    if (err) { /* gerer */ return; }
    console.log('Termine');
  });
});
// Probleme : callback hell si on chaine plusieurs operations

// 2. PROMISES (moderne)
fsPromises.readFile('./data.txt', 'utf8')
  .then(data => fsPromises.writeFile('./output.txt', data.toUpperCase()))
  .then(() => console.log('Termine'))
  .catch(err => console.error(err));

// 3. ASYNC/AWAIT (syntaxe la plus lisible)
async function traiter() {
  try {
    const data = await fsPromises.readFile('./data.txt', 'utf8');
    await fsPromises.writeFile('./output.txt', data.toUpperCase());
    console.log('Termine');
  } catch (err) {
    console.error(err);
  }
}

// Operations en parallele avec Promise.all
async function traiterEnParallele(fichiers) {
  const contenus = await Promise.all(
    fichiers.map(f => fsPromises.readFile(f, 'utf8'))
  );
  return contenus.join('\n---\n');
}
```

---

## Carte Mentale

```
                        ┌─────────────────────────────────────────────┐
                        │              NODE.JS                        │
                        │   Runtime JavaScript cote serveur           │
                        └───────────────────┬─────────────────────────┘
                                            │
              ┌─────────────────────────────┼──────────────────────────────┐
              │                             │                              │
   ┌──────────▼──────────┐    ┌─────────────▼────────────┐   ┌────────────▼────────────┐
   │   MOTEUR & RUNTIME  │    │     EVENT LOOP           │   │      MODULES NATIFS     │
   │                     │    │                          │   │                         │
   │  V8 (compile JS)    │    │  Call Stack              │   │  fs   (fichiers)        │
   │  libuv (async I/O)  │    │  Microtask Queue         │   │  http (serveur)         │
   │  Node bindings C++  │    │    nextTick (prio haute) │   │  path (chemins)         │
   └─────────────────────┘    │    Promise.then          │   │  events (EventEmitter)  │
                              │  Macrotask Queue         │   │  stream (streams)       │
                              │    setTimeout            │   │  crypto (chiffrement)   │
                              │    setInterval           │   │  os    (systeme)        │
                              │    I/O callbacks         │   │  child_process          │
                              │  Thread Pool (libuv)     │   └─────────────────────────┘
                              │    fs, dns, crypto...    │
                              └──────────────────────────┘

   ┌─────────────────────────────────────────────────────────────────────────────────┐
   │                           SYSTEME DE MODULES                                   │
   │                                                                                 │
   │   CommonJS (CJS)                    ES Modules (ESM)                           │
   │   require('./module')               import { fn } from './module.mjs'          │
   │   module.exports = { ... }          export function fn() { ... }               │
   │   synchrone                         asynchrone, tree-shakable                  │
   │   __dirname disponible              import.meta.url a utiliser                 │
   └─────────────────────────────────────────────────────────────────────────────────┘

   ┌─────────────────────────────────────────────────────────────────────────────────┐
   │                              NPM ECOSYSTEM                                     │
   │                                                                                 │
   │   package.json                                                                  │
   │   ├── dependencies      (production)                                           │
   │   ├── devDependencies   (build, test, lint)                                    │
   │   ├── scripts           (npm run <script>)                                     │
   │   └── version           (semver : MAJOR.MINOR.PATCH)                          │
   │                                                                                 │
   │   npm install / npm i           Installer les dependances                     │
   │   npm install -D <pkg>          Dev dependency                                 │
   │   npm run <script>              Executer un script                             │
   └─────────────────────────────────────────────────────────────────────────────────┘

   ┌───────────────────────────────────────┐  ┌─────────────────────────────────────┐
   │           STREAMS                     │  │          ASYNCHRONISME              │
   │                                       │  │                                     │
   │   Readable  <── source de donnees     │  │   1. Callbacks (ancien)             │
   │   Writable  ──> destination           │  │   2. Promises .then/.catch          │
   │   Transform ──> transforme en passant │  │   3. async/await (recommande)       │
   │   Duplex    ──> read + write          │  │   4. Promise.all (parallele)        │
   │                                       │  │   5. for await...of (streams)       │
   │   pipe() : source.pipe(destination)   │  └─────────────────────────────────────┘
   └───────────────────────────────────────┘

   ┌─────────────────────────────────────────────────────────────────────────────────┐
   │                           EVENTEMIITTER PATTERN                                │
   │                                                                                 │
   │   emitter.on('event', handler)    Ecouter                                      │
   │   emitter.emit('event', data)     Declencher                                   │
   │   emitter.once('event', handler)  Ecouter une seule fois                       │
   │   emitter.off('event', handler)   Supprimer un listener                        │
   │                                                                                 │
   │   Heritent d'EventEmitter :  fs.ReadStream, http.Server, net.Socket...         │
   └─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Exercices Pratiques

### Exercice 1 : Logger de fichiers

Cree un script `logger.js` qui :
1. Accepte un message en argument CLI (`node logger.js "Mon message"`)
2. Ajoute une ligne dans `logs/app.log` avec timestamp, PID et message
3. Cree le dossier `logs/` s'il n'existe pas
4. Affiche le nombre total de lignes dans le log

```javascript
// Squelette de depart
const fsPromises = require('fs/promises');
const path = require('path');

async function log(message) {
  const logsDir = path.join(__dirname, 'logs');
  const logFile = path.join(logsDir, 'app.log');

  // TODO 1 : creer le dossier logs s'il n'existe pas (mkdir recursive)
  // TODO 2 : formater la ligne : "[TIMESTAMP] [PID:xxxx] message\n"
  // TODO 3 : ajouter la ligne dans app.log (appendFile)
  // TODO 4 : lire le fichier et compter les lignes
  // TODO 5 : afficher "Log enregistre. Total : N lignes."
}

const message = process.argv[2];
if (!message) {
  console.error('Usage : node logger.js "Mon message"');
  process.exit(1);
}

log(message).catch(console.error);
```

<details>
<summary>Solution</summary>

```javascript
const fsPromises = require('fs/promises');
const path = require('path');

async function log(message) {
  const logsDir = path.join(__dirname, 'logs');
  const logFile = path.join(logsDir, 'app.log');

  await fsPromises.mkdir(logsDir, { recursive: true });

  const timestamp = new Date().toISOString();
  const ligne = `[${timestamp}] [PID:${process.pid}] ${message}\n`;

  await fsPromises.appendFile(logFile, ligne, 'utf8');

  const contenu = await fsPromises.readFile(logFile, 'utf8');
  const nbLignes = contenu.split('\n').filter(l => l.trim()).length;

  console.log(`Log enregistre. Total : ${nbLignes} ligne(s).`);
}

const message = process.argv[2];
if (!message) {
  console.error('Usage : node logger.js "Mon message"');
  process.exit(1);
}

log(message).catch(console.error);
```
</details>

---

### Exercice 2 : Serveur HTTP avec compteur

Cree un serveur HTTP minimal avec 3 routes :
- `GET /` → `{ status: 'ok', visites: N }` ou N est le nombre de requetes recues
- `GET /reset` → remet le compteur a 0
- Toute autre route → 404 avec `{ erreur: 'Route inconnue' }`

```javascript
const http = require('http');

let compteur = 0;

const serveur = http.createServer((req, res) => {
  // TODO : implementer les 3 routes
  // Helper sendJSON disponible
  function sendJSON(status, data) {
    const body = JSON.stringify(data);
    res.writeHead(status, { 'Content-Type': 'application/json' });
    res.end(body);
  }

  // ...
});

serveur.listen(3000, () => console.log('http://localhost:3000'));
```

**Pour tester** :
```bash
curl http://localhost:3000/
curl http://localhost:3000/
curl http://localhost:3000/reset
curl http://localhost:3000/
curl http://localhost:3000/inconnu
```

---

### Exercice 3 : Event Bus avec EventEmitter

Implemente un systeme de notifications simple :
1. Cree une classe `NotificationBus` heritant d'`EventEmitter`
2. Methode `publier(canal, message)` qui emet un evenement du nom du canal
3. Methode `abonner(canal, handler)` qui ecoute le canal
4. Methode `desabonner(canal, handler)`
5. Methode `historique()` qui retourne les 10 derniers messages emis (toutes canaux)

```javascript
const { EventEmitter } = require('events');

class NotificationBus extends EventEmitter {
  constructor() {
    super();
    this._historique = [];
  }

  publier(canal, message) {
    // TODO : ajouter a l'historique (garder seulement les 10 derniers)
    // TODO : emettre l'evenement canal avec { canal, message, timestamp }
  }

  abonner(canal, handler) {
    // TODO
  }

  desabonner(canal, handler) {
    // TODO
  }

  historique() {
    // TODO : retourner une copie des 10 derniers messages
  }
}

// Test
const bus = new NotificationBus();

bus.abonner('chat', (data) => {
  console.log(`[CHAT] ${data.message}`);
});

bus.abonner('alerte', (data) => {
  console.log(`[ALERTE] ${data.message}`);
});

bus.publier('chat', 'Bonjour tout le monde');
bus.publier('alerte', 'Serveur en surcharge !');
bus.publier('chat', 'Comment ca va ?');

console.log('\nHistorique:', bus.historique());
```

---

### Exercice 4 : Pipeline de transformation de fichier

Cree un script qui utilise des streams pour transformer un fichier CSV en JSON, ligne par ligne, sans charger tout le fichier en memoire.

**Format CSV d'entree** (`data.csv`) :
```
nom,age,ville
Alice,30,Paris
Bob,25,Lyon
Charlie,35,Marseille
```

**Format JSON de sortie** (`output.json`) :
```json
[
  { "nom": "Alice", "age": "30", "ville": "Paris" },
  { "nom": "Bob", "age": "25", "ville": "Lyon" },
  { "nom": "Charlie", "age": "35", "ville": "Marseille" }
]
```

```javascript
const fs = require('fs');
const readline = require('readline');
const fsPromises = require('fs/promises');

async function csvVersJSON(inputPath, outputPath) {
  const stream = fs.createReadStream(inputPath, 'utf8');
  const rl = readline.createInterface({ input: stream });

  let headers = null;
  const resultats = [];

  for await (const ligne of rl) {
    // TODO : premiere ligne = headers, suivantes = donnees
    // Indice : ligne.split(',')
  }

  // TODO : ecrire le tableau JSON dans outputPath (indente, lisible)
  // Indice : JSON.stringify(resultats, null, 2)

  console.log(`Conversion terminee : ${resultats.length} enregistrements`);
  return resultats;
}

csvVersJSON('./data.csv', './output.json').catch(console.error);
```

---

## Liens et Suite

- [[JavaScript/03 - JavaScript Asynchrone]] — Approfondir les Promises, async/await et la gestion d'erreurs asynchrone
- [[JavaScript/04 - JavaScript Moderne ES6+]] — Destructuring, spread, modules, classes et tout ce qu'on utilise en Node.js moderne
- [[Node Express/01 - Express.js et NestJS]] — Passer a un vrai framework HTTP avec routing, middlewares et REST API
- [[Node Express/03 - Nodejs Avance]] — Worker Threads, Cluster, streams avances, performance et production

---

*Note creee pour le cursus Holberton — Trimestre 3 : Backend Node.js*
