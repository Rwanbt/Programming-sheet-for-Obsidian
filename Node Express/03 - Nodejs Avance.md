# Node.js Avance

## Au-dela des bases

> [!tip] Analogie
> Node.js de base, c'est un chef cuisinier qui travaille seul mais tres vite (event loop). Node.js avance, c'est une brigade complete : le chef coordonne des commis (worker threads), envoie des sous-traitants (child processes), ouvre plusieurs restaurants en parallele (cluster), et gere un flux continu de commandes sans jamais engorger la cuisine (streams). La connaissance de l'event loop est le prerequis — ici on apprend a orchestrer toute la brigade.

Tu connais deja les fondamentaux vus dans [[Node Express/02 - Nodejs Fondamentaux]] : event loop, callbacks, Promises, async/await, les modules de base `fs`, `path`, `http`. Tu sais creer une API REST avec Express (cf. [[Node Express/01 - Express.js et NestJS]]) et tu maitrises l'asynchronisme JavaScript (cf. [[JavaScript/03 - JavaScript Asynchrone]]).

Ce cours couvre les outils qui font la difference entre une app Node.js "qui marche" et une app Node.js **en production** : parallelisme, securite, performance, monitoring.

---

## Sommaire

1. [[#Worker Threads — Parallelisme CPU]]
2. [[#Child Processes — Delegation de taches]]
3. [[#Clustering — Utiliser tous les cores]]
4. [[#Streams Avances — Traitement de flux]]
5. [[#Authentification JWT avec Express]]
6. [[#WebSockets avec ws]]
7. [[#Rate Limiting et Securite]]
8. [[#Testing avec Jest et Supertest]]
9. [[#PM2 en Production]]
10. [[#Profiling et Performance]]
11. [[#Carte Mentale]]
12. [[#Exercices Pratiques]]

---

## Worker Threads — Parallelisme CPU

### Le probleme : Node.js est single-threaded

L'event loop de Node.js execute le code JavaScript sur **un seul thread**. C'est parfait pour les operations I/O (lecture fichier, requete HTTP) car elles sont non-bloquantes. Mais si tu fais un calcul intensif (compression d'image, chiffrement, parsing JSON de 500 Mo), tu **bloques l'event loop** entier — aucune autre requete ne peut etre traitee pendant ce temps.

```
Sans Worker Threads :

Thread principal  [req1]--[CALCUL LOURD 2s]--[req2]--[req3]
                          ^^^^^^^^^^^^^^^^
                          req2 et req3 attendent 2s
```

```
Avec Worker Threads :

Thread principal  [req1]---------[req2]--[req3]
                        \
Worker Thread 1   [CALCUL LOURD 2s] -> postMessage(resultat)
```

### Creer un Worker Thread

```javascript
// fichier : main.js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');
const path = require('path');

if (isMainThread) {
  // Code du thread principal
  function runWorker(data) {
    return new Promise((resolve, reject) => {
      const worker = new Worker(__filename, {
        workerData: data   // donnees passees au worker a la creation
      });

      worker.on('message', resolve);   // worker envoie un resultat
      worker.on('error', reject);      // erreur non rattrapee dans le worker
      worker.on('exit', (code) => {
        if (code !== 0) {
          reject(new Error(`Worker stopped with exit code ${code}`));
        }
      });
    });
  }

  // Lancer 4 workers en parallele
  Promise.all([
    runWorker({ start: 1,       end: 25_000_000 }),
    runWorker({ start: 25_000_001, end: 50_000_000 }),
    runWorker({ start: 50_000_001, end: 75_000_000 }),
    runWorker({ start: 75_000_001, end: 100_000_000 }),
  ]).then(results => {
    const total = results.reduce((a, b) => a + b, 0);
    console.log('Total:', total);
  });

} else {
  // Code du worker (meme fichier, branche else)
  const { start, end } = workerData;
  let sum = 0;
  for (let i = start; i <= end; i++) {
    sum += i;
  }
  parentPort.postMessage(sum);
}
```

> [!info] `__filename` comme chemin du worker
> Quand `isMainThread === false`, le meme fichier est execute comme worker. C'est pratique pour les exemples. En production, separez le code worker dans un fichier dedie pour plus de clarte.

### SharedArrayBuffer et Atomics — Memoire partagee

Par defaut, chaque worker a sa propre memoire. Pour partager de la memoire entre threads sans copie, on utilise `SharedArrayBuffer`.

```javascript
// Partager un buffer entre le thread principal et un worker
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
  // Allouer 4 octets partages (un Int32)
  const sharedBuffer = new SharedArrayBuffer(4);
  const sharedArray = new Int32Array(sharedBuffer);
  sharedArray[0] = 0; // Compteur initial

  const worker = new Worker(__filename, {
    workerData: { sharedBuffer }
  });

  worker.on('exit', () => {
    console.log('Valeur finale:', sharedArray[0]); // Valeur incrementee par le worker
  });

} else {
  const sharedArray = new Int32Array(workerData.sharedBuffer);

  // Atomics.add : increment atomique, thread-safe
  for (let i = 0; i < 1000; i++) {
    Atomics.add(sharedArray, 0, 1);
  }
}
```

**Pourquoi `Atomics` ?** Sans Atomics, deux threads peuvent lire la meme valeur, l'incrementer chacun de leur cote, et ecrire la meme valeur — race condition classique. `Atomics.add` garantit que l'operation lecture+modification+ecriture est atomique.

| Operation Atomics | Description |
|---|---|
| `Atomics.add(array, index, value)` | Incremente et retourne l'ancienne valeur |
| `Atomics.load(array, index)` | Lecture thread-safe |
| `Atomics.store(array, index, value)` | Ecriture thread-safe |
| `Atomics.compareExchange(arr, i, expected, replacement)` | CAS (Compare-And-Swap) |
| `Atomics.wait(array, index, value)` | Bloque le thread jusqu'a changement |
| `Atomics.notify(array, index, count)` | Reveille des threads en attente |

### Pool de Workers (pattern production)

Creer un worker par requete est couteux. Le pattern professionnel est un **pool de workers** reutilisables.

```javascript
// worker-pool.js
const { Worker } = require('worker_threads');
const os = require('os');

class WorkerPool {
  constructor(workerFile, poolSize = os.cpus().length) {
    this.workers = [];
    this.queue = [];

    for (let i = 0; i < poolSize; i++) {
      this._addWorker(workerFile);
    }
  }

  _addWorker(workerFile) {
    const worker = new Worker(workerFile);
    worker.isIdle = true;

    worker.on('message', (result) => {
      worker.isIdle = true;
      worker._resolve(result);
      this._processQueue(); // Traiter la tache suivante en attente
    });

    worker.on('error', (err) => {
      worker.isIdle = true;
      worker._reject(err);
      this._processQueue();
    });

    this.workers.push(worker);
  }

  run(data) {
    return new Promise((resolve, reject) => {
      const task = { data, resolve, reject };
      const idleWorker = this.workers.find(w => w.isIdle);

      if (idleWorker) {
        this._runTask(idleWorker, task);
      } else {
        this.queue.push(task); // Tous occupes : mettre en file
      }
    });
  }

  _runTask(worker, task) {
    worker.isIdle = false;
    worker._resolve = task.resolve;
    worker._reject = task.reject;
    worker.postMessage(task.data);
  }

  _processQueue() {
    if (this.queue.length > 0) {
      const idleWorker = this.workers.find(w => w.isIdle);
      if (idleWorker) {
        this._runTask(idleWorker, this.queue.shift());
      }
    }
  }

  destroy() {
    this.workers.forEach(w => w.terminate());
  }
}

module.exports = WorkerPool;
```

> [!warning] Quand NE PAS utiliser Worker Threads
> Les Worker Threads ne sont utiles que pour le code **CPU-intensif**. Pour les operations I/O (base de donnees, fichiers, HTTP), l'event loop avec async/await est plus efficace. Utiliser des workers pour de l'I/O ajoute de la complexite sans gain.

---

## Child Processes — Delegation de taches

### exec, spawn, fork : quand utiliser quoi ?

```
exec   → commande shell simple, petite sortie (< quelques Mo)
spawn  → commande shell longue, sortie en stream (ffmpeg, grep sur gros fichier)
fork   → lancer un autre script Node.js avec canal de communication IPC
```

### exec — Simple et buffered

```javascript
const { exec } = require('child_process');

// exec(commande, callback)
exec('ls -la /tmp', { timeout: 5000 }, (error, stdout, stderr) => {
  if (error) {
    console.error('Erreur:', error.message);
    return;
  }
  if (stderr) {
    console.error('Stderr:', stderr);
  }
  console.log('Output:', stdout);
});

// Version Promise avec util.promisify
const { promisify } = require('util');
const execAsync = promisify(exec);

async function getGitLog() {
  try {
    const { stdout } = await execAsync('git log --oneline -10');
    return stdout.trim().split('\n');
  } catch (err) {
    console.error('Git non disponible:', err.message);
    return [];
  }
}
```

> [!warning] Injection de commande
> Ne jamais concatener de l'input utilisateur dans une commande `exec`. Utiliser `spawn` avec un tableau d'arguments qui n'est pas interprete par le shell.

### spawn — Streaming et longues commandes

```javascript
const { spawn } = require('child_process');

// Encoder une video avec ffmpeg (sortie en stream)
function encodeVideo(inputPath, outputPath) {
  return new Promise((resolve, reject) => {
    const ffmpeg = spawn('ffmpeg', [
      '-i', inputPath,
      '-c:v', 'libx264',
      '-preset', 'fast',
      '-y',           // ecraser si existe
      outputPath
    ]);

    // stderr de ffmpeg contient la progression
    ffmpeg.stderr.on('data', (data) => {
      const line = data.toString();
      if (line.includes('frame=')) {
        process.stdout.write('\r' + line.trim());
      }
    });

    ffmpeg.on('close', (code) => {
      if (code === 0) {
        console.log('\nEncodage termine.');
        resolve(outputPath);
      } else {
        reject(new Error(`ffmpeg exit code: ${code}`));
      }
    });

    ffmpeg.on('error', reject);
  });
}
```

### fork — Communication IPC avec un autre script Node

```javascript
// parent.js
const { fork } = require('child_process');
const path = require('path');

const child = fork(path.join(__dirname, 'worker-script.js'), [], {
  env: { ...process.env, WORKER_ID: '1' }
});

// Envoyer un message au child
child.send({ type: 'COMPUTE', payload: { n: 40 } });

// Recevoir les messages du child
child.on('message', (msg) => {
  console.log('Resultat recu:', msg);
});

child.on('exit', (code) => {
  console.log('Child process termine avec le code:', code);
});

// Terminer le child proprement apres 10s
setTimeout(() => child.kill('SIGTERM'), 10000);
```

```javascript
// worker-script.js
process.on('message', (msg) => {
  if (msg.type === 'COMPUTE') {
    const result = fibonacci(msg.payload.n);
    process.send({ type: 'RESULT', result });
  }
});

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Gerer l'arret propre
process.on('SIGTERM', () => {
  console.log('Worker: arret demande');
  process.exit(0);
});
```

| Methode | Shell ? | Sortie | IPC | Cas d'usage |
|---|---|---|---|---|
| `exec` | Oui | Buffer | Non | Commandes courtes, scripts shell |
| `execFile` | Non | Buffer | Non | Executable direct, plus sur |
| `spawn` | Non | Stream | Non | FFmpeg, grep, processus longs |
| `fork` | Non | Stream | Oui | Autre script Node.js |

---

## Clustering — Utiliser tous les cores

### Pourquoi le clustering ?

Un processus Node.js n'utilise qu'**un seul core CPU**. Sur un serveur avec 8 cores, tu gaspilles 87,5% de ta puissance de calcul. Le module `cluster` permet de lancer N processus qui partagent le meme port TCP.

```
                    ┌─────────────────────────┐
                    │      Load Balancer       │
                    │   (OS round-robin)       │
                    └────────────┬────────────┘
                                 │ :3000
              ┌──────────────────┼──────────────────┐
              │                  │                  │
         Worker 1           Worker 2           Worker 3  ...
       (PID 1001)          (PID 1002)         (PID 1003)
        Core 0               Core 1             Core 2
```

### Implementation avec le module cluster

```javascript
// server.js
const cluster = require('cluster');
const http = require('http');
const os = require('os');

const NUM_CPUS = os.cpus().length;

if (cluster.isPrimary) {
  console.log(`Master PID ${process.pid} running`);
  console.log(`Forking ${NUM_CPUS} workers...`);

  // Creer un worker par core
  for (let i = 0; i < NUM_CPUS; i++) {
    cluster.fork();
  }

  // Relancer un worker qui plante
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died (${signal || code}). Restarting...`);
    cluster.fork();
  });

  cluster.on('online', (worker) => {
    console.log(`Worker ${worker.process.pid} is online`);
  });

} else {
  // Les workers partagent le meme port
  const app = require('./app'); // ton app Express

  app.listen(3000, () => {
    console.log(`Worker ${process.pid} listening on port 3000`);
  });
}
```

> [!warning] Etat partage entre workers
> Chaque worker est un processus independant — ils ne partagent PAS la memoire. Les sessions en memoire, les caches locaux, les WebSockets in-memory ne fonctionneront pas avec le clustering. Utilise **Redis** pour partager l'etat entre workers.

### Communication entre Master et Workers

```javascript
// Master envoie une configuration aux workers
if (cluster.isPrimary) {
  for (const id in cluster.workers) {
    cluster.workers[id].send({ type: 'CONFIG', data: { rateLimit: 100 } });
  }
} else {
  process.on('message', (msg) => {
    if (msg.type === 'CONFIG') {
      console.log(`Worker ${process.pid} recu config:`, msg.data);
    }
  });
}
```

### Graceful Shutdown avec clustering

```javascript
if (cluster.isPrimary) {
  process.on('SIGTERM', () => {
    console.log('Arret en cours, notifying workers...');

    for (const id in cluster.workers) {
      cluster.workers[id].send({ type: 'SHUTDOWN' });
    }

    // Forcer la fermeture apres 30s si les workers ne s'arretent pas
    setTimeout(() => process.exit(0), 30000);
  });
}
```

---

## Streams Avances — Traitement de flux

### Rappel : les 4 types de streams

```
Readable   → source de donnees  (fs.createReadStream, HTTP req)
Writable   → destination        (fs.createWriteStream, HTTP res)
Duplex     → les deux           (TCP socket, WebSocket)
Transform  → Duplex qui modifie (compression, chiffrement, parsing)
```

### Transform Stream — Modifier des donnees a la volee

```javascript
const { Transform } = require('stream');

// Transform stream qui convertit CSV en JSON ligne par ligne
class CsvToJsonTransform extends Transform {
  constructor(options = {}) {
    super({ ...options, objectMode: true });
    this._headers = null;
    this._buffer = '';
  }

  _transform(chunk, encoding, callback) {
    this._buffer += chunk.toString();
    const lines = this._buffer.split('\n');

    // Garder la derniere ligne incomplete dans le buffer
    this._buffer = lines.pop();

    for (const line of lines) {
      if (!line.trim()) continue;

      if (!this._headers) {
        // Premiere ligne = en-tetes
        this._headers = line.split(',').map(h => h.trim());
      } else {
        const values = line.split(',').map(v => v.trim());
        const obj = {};
        this._headers.forEach((h, i) => {
          obj[h] = values[i];
        });
        this.push(obj); // Envoyer l'objet au stream suivant
      }
    }

    callback(); // Signaler que le chunk est traite
  }

  _flush(callback) {
    // Traiter le dernier fragment restant dans le buffer
    if (this._buffer.trim()) {
      const values = this._buffer.split(',').map(v => v.trim());
      const obj = {};
      this._headers.forEach((h, i) => {
        obj[h] = values[i];
      });
      this.push(obj);
    }
    callback();
  }
}

// Utilisation
const fs = require('fs');

fs.createReadStream('data.csv')
  .pipe(new CsvToJsonTransform())
  .on('data', (obj) => {
    console.log('Ligne parsee:', obj);
  })
  .on('error', (err) => console.error(err))
  .on('end', () => console.log('Traitement termine'));
```

### Backpressure — Ne pas saturer la memoire

Le backpressure est le mecanisme par lequel un stream **Writable** dit au stream **Readable** de ralentir quand il ne peut plus absorber les donnees.

```javascript
const { Writable } = require('stream');

class SlowWritable extends Writable {
  constructor() {
    super({ highWaterMark: 1024 }); // Buffer de 1 Ko seulement
  }

  _write(chunk, encoding, callback) {
    // Simuler une ecriture lente (base de donnees par exemple)
    setTimeout(() => {
      console.log(`Ecrit ${chunk.length} bytes`);
      callback(); // Appeler callback APRES avoir fini
    }, 100);
  }
}

// Gestion manuelle du backpressure
function copyWithBackpressure(readable, writable) {
  readable.on('data', (chunk) => {
    // write() retourne false si le buffer interne est plein
    const canContinue = writable.write(chunk);

    if (!canContinue) {
      readable.pause(); // Stopper la lecture
      console.log('Backpressure: lecture en pause');
    }
  });

  writable.on('drain', () => {
    readable.resume(); // Reprendre quand le writable est pret
    console.log('Drain: reprise de la lecture');
  });

  readable.on('end', () => writable.end());
}
```

### pipeline() — Gestion automatique des erreurs

`pipe()` ne propage pas les erreurs. `pipeline()` (Node 10+) gere les erreurs et le cleanup automatiquement.

```javascript
const { pipeline } = require('stream');
const { promisify } = require('util');
const fs = require('fs');
const zlib = require('zlib');

const pipelineAsync = promisify(pipeline);

async function compressFile(inputPath, outputPath) {
  try {
    await pipelineAsync(
      fs.createReadStream(inputPath),
      zlib.createGzip(),          // Transform : compression gzip
      fs.createWriteStream(outputPath)
    );
    console.log('Compression terminee:', outputPath);
  } catch (err) {
    console.error('Erreur pipeline:', err);
    // pipeline() a deja nettoye les streams automatiquement
  }
}

// Exemple avec un Transform personnalise dans le pipeline
async function processLargeFile(inputPath, outputPath) {
  await pipelineAsync(
    fs.createReadStream(inputPath),
    new CsvToJsonTransform(),                     // Transform 1 : CSV -> objet
    new Transform({                               // Transform 2 : filtrer
      objectMode: true,
      transform(obj, enc, cb) {
        if (parseInt(obj.age) >= 18) this.push(obj);
        cb();
      }
    }),
    new Transform({                               // Transform 3 : objet -> JSON line
      objectMode: true,
      transform(obj, enc, cb) {
        cb(null, JSON.stringify(obj) + '\n');
      }
    }),
    fs.createWriteStream(outputPath)
  );
}
```

> [!info] Streams en Node.js 17+
> `stream.pipeline` et `stream.Readable.from()` sont maintenant disponibles directement depuis `stream/promises`. Pas besoin de `promisify` :
> ```javascript
> const { pipeline } = require('stream/promises');
> await pipeline(source, transform, destination);
> ```

---

## Authentification JWT avec Express

### Architecture complete : inscription, connexion, refresh

```
Client                    Serveur
  |                          |
  |--- POST /register ------>|  bcrypt.hash(password)
  |<-- 201 Created ----------|
  |                          |
  |--- POST /login --------->|  bcrypt.compare() + sign(accessToken + refreshToken)
  |<-- { accessToken,        |
  |      refreshToken } -----|
  |                          |
  |--- GET /protected ------>|  verify(accessToken)
  |   Authorization: Bearer  |
  |<-- data -----------------|
  |                          |
  |--- POST /refresh ------->|  verify(refreshToken) + sign(nouveau accessToken)
  |<-- { accessToken } ------|
  |                          |
  |--- POST /logout -------->|  invalider refreshToken en DB
  |<-- 200 OK ---------------|
```

### Installation et configuration

```javascript
// npm install bcryptjs jsonwebtoken express-validator
// Configuration centralisee
// config/auth.js
module.exports = {
  ACCESS_TOKEN_SECRET: process.env.JWT_ACCESS_SECRET || 'change-me-in-production',
  REFRESH_TOKEN_SECRET: process.env.JWT_REFRESH_SECRET || 'different-secret-in-production',
  ACCESS_TOKEN_EXPIRY: '15m',   // Courte duree : si vole, expire vite
  REFRESH_TOKEN_EXPIRY: '7d',   // Longue duree : stocke en DB securisee
  SALT_ROUNDS: 12               // bcrypt : augmenter ralentit le hachage
};
```

### Modele utilisateur (exemple sans ORM)

```javascript
// models/userModel.js
const bcrypt = require('bcryptjs');
const { SALT_ROUNDS } = require('../config/auth');

// Simule une "base de donnees" en memoire (remplacer par vraie DB)
const users = new Map();
const refreshTokens = new Set(); // En production : table DB avec TTL

const UserModel = {
  async create(email, password) {
    if (users.has(email)) {
      throw new Error('EMAIL_TAKEN');
    }
    const hash = await bcrypt.hash(password, SALT_ROUNDS);
    const user = { id: Date.now().toString(), email, passwordHash: hash };
    users.set(email, user);
    return { id: user.id, email: user.email };
  },

  async findByEmail(email) {
    return users.get(email) || null;
  },

  async verifyPassword(plainPassword, hash) {
    return bcrypt.compare(plainPassword, hash);
  },

  storeRefreshToken(token) {
    refreshTokens.add(token);
  },

  isRefreshTokenValid(token) {
    return refreshTokens.has(token);
  },

  revokeRefreshToken(token) {
    refreshTokens.delete(token);
  }
};

module.exports = UserModel;
```

### Routes d'authentification

```javascript
// routes/auth.js
const express = require('express');
const jwt = require('jsonwebtoken');
const { body, validationResult } = require('express-validator');
const UserModel = require('../models/userModel');
const {
  ACCESS_TOKEN_SECRET, REFRESH_TOKEN_SECRET,
  ACCESS_TOKEN_EXPIRY, REFRESH_TOKEN_EXPIRY
} = require('../config/auth');

const router = express.Router();

// Helpers de generation de tokens
function generateAccessToken(userId, email) {
  return jwt.sign(
    { sub: userId, email },         // payload (ne pas mettre le mot de passe !)
    ACCESS_TOKEN_SECRET,
    { expiresIn: ACCESS_TOKEN_EXPIRY }
  );
}

function generateRefreshToken(userId) {
  return jwt.sign(
    { sub: userId, type: 'refresh' },
    REFRESH_TOKEN_SECRET,
    { expiresIn: REFRESH_TOKEN_EXPIRY }
  );
}

// Validation des inputs
const registerValidation = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).matches(/[A-Z]/).matches(/[0-9]/)
    .withMessage('Min 8 caracteres, 1 majuscule, 1 chiffre')
];

// POST /auth/register
router.post('/register', registerValidation, async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({ errors: errors.array() });
  }

  try {
    const { email, password } = req.body;
    const user = await UserModel.create(email, password);
    res.status(201).json({ message: 'Compte cree', user });
  } catch (err) {
    if (err.message === 'EMAIL_TAKEN') {
      return res.status(409).json({ error: 'Email deja utilise' });
    }
    res.status(500).json({ error: 'Erreur serveur' });
  }
});

// POST /auth/login
router.post('/login', async (req, res) => {
  const { email, password } = req.body;

  try {
    const user = await UserModel.findByEmail(email);

    // Meme message d'erreur que si le mot de passe est faux : evite l'enumeration
    if (!user || !(await UserModel.verifyPassword(password, user.passwordHash))) {
      return res.status(401).json({ error: 'Identifiants invalides' });
    }

    const accessToken = generateAccessToken(user.id, user.email);
    const refreshToken = generateRefreshToken(user.id);

    UserModel.storeRefreshToken(refreshToken);

    // refreshToken dans un cookie httpOnly (plus sur que localStorage)
    res.cookie('refreshToken', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000 // 7 jours en ms
    });

    res.json({ accessToken, expiresIn: ACCESS_TOKEN_EXPIRY });
  } catch (err) {
    res.status(500).json({ error: 'Erreur serveur' });
  }
});

// POST /auth/refresh
router.post('/refresh', (req, res) => {
  const refreshToken = req.cookies.refreshToken;

  if (!refreshToken) {
    return res.status(401).json({ error: 'Refresh token manquant' });
  }

  if (!UserModel.isRefreshTokenValid(refreshToken)) {
    return res.status(403).json({ error: 'Refresh token invalide ou revoque' });
  }

  try {
    const payload = jwt.verify(refreshToken, REFRESH_TOKEN_SECRET);
    const newAccessToken = generateAccessToken(payload.sub, payload.email);
    res.json({ accessToken: newAccessToken });
  } catch (err) {
    // Token expire ou malformed
    UserModel.revokeRefreshToken(refreshToken);
    res.status(403).json({ error: 'Refresh token expire' });
  }
});

// POST /auth/logout
router.post('/logout', (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  if (refreshToken) {
    UserModel.revokeRefreshToken(refreshToken);
  }
  res.clearCookie('refreshToken');
  res.json({ message: 'Deconnecte' });
});

module.exports = router;
```

### Middleware d'authentification

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const { ACCESS_TOKEN_SECRET } = require('../config/auth');

function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Token manquant' });
  }

  const token = authHeader.split(' ')[1];

  try {
    const payload = jwt.verify(token, ACCESS_TOKEN_SECRET);
    req.user = { id: payload.sub, email: payload.email };
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expire', code: 'TOKEN_EXPIRED' });
    }
    return res.status(403).json({ error: 'Token invalide' });
  }
}

// Middleware de role (exemple RBAC basique)
function requireRole(...roles) {
  return (req, res, next) => {
    if (!req.user) return res.status(401).json({ error: 'Non authentifie' });
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Permission insuffisante' });
    }
    next();
  };
}

module.exports = { authenticate, requireRole };
```

```javascript
// Utilisation dans les routes
const { authenticate, requireRole } = require('./middleware/auth');

router.get('/profile', authenticate, (req, res) => {
  res.json({ user: req.user });
});

router.delete('/admin/users/:id', authenticate, requireRole('admin'), async (req, res) => {
  // Seul un admin peut acceder ici
});
```

---

## WebSockets avec ws

### Serveur WebSocket de base

```javascript
// npm install ws
const WebSocket = require('ws');
const http = require('http');
const express = require('express');

const app = express();
const server = http.createServer(app);

// Attacher le serveur WS au meme serveur HTTP
const wss = new WebSocket.Server({ server });

// Map pour gerer les "rooms"
const rooms = new Map(); // roomId -> Set<WebSocket>

wss.on('connection', (ws, req) => {
  // Ajouter des proprietes au socket pour l'identifier
  ws.id = Math.random().toString(36).substr(2, 9);
  ws.isAlive = true;
  ws.roomId = null;

  console.log(`Client connecte: ${ws.id}`);

  ws.on('message', (rawData) => {
    try {
      const msg = JSON.parse(rawData.toString());
      handleMessage(ws, msg);
    } catch (err) {
      ws.send(JSON.stringify({ type: 'ERROR', message: 'JSON invalide' }));
    }
  });

  ws.on('close', () => {
    console.log(`Client deconnecte: ${ws.id}`);
    leaveRoom(ws);
  });

  ws.on('error', (err) => {
    console.error(`Erreur WS ${ws.id}:`, err.message);
  });

  // Pong repond au heartbeat
  ws.on('pong', () => {
    ws.isAlive = true;
  });

  ws.send(JSON.stringify({ type: 'CONNECTED', id: ws.id }));
});

// Heartbeat : deconnecter les clients morts
const heartbeatInterval = setInterval(() => {
  wss.clients.forEach((ws) => {
    if (!ws.isAlive) {
      leaveRoom(ws);
      return ws.terminate();
    }
    ws.isAlive = false;
    ws.ping(); // Le client repond avec pong
  });
}, 30000);

wss.on('close', () => clearInterval(heartbeatInterval));
```

### Gestion des rooms et broadcast

```javascript
function handleMessage(ws, msg) {
  switch (msg.type) {
    case 'JOIN_ROOM':
      joinRoom(ws, msg.roomId);
      break;

    case 'LEAVE_ROOM':
      leaveRoom(ws);
      break;

    case 'BROADCAST':
      broadcastToRoom(ws.roomId, {
        type: 'MESSAGE',
        from: ws.id,
        content: msg.content,
        timestamp: Date.now()
      }, ws); // Exclure l'expediteur
      break;

    case 'PRIVATE':
      sendPrivate(msg.targetId, {
        type: 'PRIVATE_MESSAGE',
        from: ws.id,
        content: msg.content
      });
      break;
  }
}

function joinRoom(ws, roomId) {
  leaveRoom(ws); // Quitter la room courante si besoin

  if (!rooms.has(roomId)) {
    rooms.set(roomId, new Set());
  }

  rooms.get(roomId).add(ws);
  ws.roomId = roomId;

  // Notifier les autres membres de la room
  broadcastToRoom(roomId, {
    type: 'USER_JOINED',
    userId: ws.id,
    memberCount: rooms.get(roomId).size
  }, ws);

  ws.send(JSON.stringify({
    type: 'JOINED_ROOM',
    roomId,
    memberCount: rooms.get(roomId).size
  }));
}

function leaveRoom(ws) {
  if (!ws.roomId || !rooms.has(ws.roomId)) return;

  const room = rooms.get(ws.roomId);
  room.delete(ws);

  if (room.size === 0) {
    rooms.delete(ws.roomId); // Supprimer la room vide
  } else {
    broadcastToRoom(ws.roomId, {
      type: 'USER_LEFT',
      userId: ws.id,
      memberCount: room.size
    });
  }

  ws.roomId = null;
}

function broadcastToRoom(roomId, message, excludeWs = null) {
  if (!rooms.has(roomId)) return;

  const json = JSON.stringify(message);
  rooms.get(roomId).forEach((client) => {
    if (client !== excludeWs && client.readyState === WebSocket.OPEN) {
      client.send(json);
    }
  });
}

function sendPrivate(targetId, message) {
  wss.clients.forEach((client) => {
    if (client.id === targetId && client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify(message));
    }
  });
}

server.listen(3000, () => console.log('Server on :3000'));
```

> [!example] Client JavaScript pour tester
> ```javascript
> const ws = new WebSocket('ws://localhost:3000');
> ws.onopen = () => {
>   ws.send(JSON.stringify({ type: 'JOIN_ROOM', roomId: 'general' }));
>   ws.send(JSON.stringify({ type: 'BROADCAST', content: 'Bonjour !' }));
> };
> ws.onmessage = (e) => console.log(JSON.parse(e.data));
> ```

---

## Rate Limiting et Securite

### Helmet — Headers de securite HTTP

```javascript
// npm install helmet cors express-rate-limit joi
const helmet = require('helmet');
const cors = require('cors');
const rateLimit = require('express-rate-limit');
const express = require('express');

const app = express();

// Helmet configure automatiquement une dizaine de headers de securite
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"], // Ajuster selon besoin
      styleSrc: ["'self'", "https://fonts.googleapis.com"],
      imgSrc: ["'self'", "data:", "https:"],
    }
  },
  crossOriginEmbedderPolicy: false // Desactiver si problemes avec assets externes
}));
```

### CORS — Cross-Origin Resource Sharing

```javascript
const corsOptions = {
  origin: (origin, callback) => {
    const whitelist = [
      'https://monapp.com',
      'https://www.monapp.com',
      process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : null
    ].filter(Boolean);

    if (!origin || whitelist.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error(`CORS: origine ${origin} non autorisee`));
    }
  },
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,       // Necessaire pour les cookies cross-origin
  maxAge: 86400            // Cache preflight 24h
};

app.use(cors(corsOptions));
```

### Rate Limiting — Limiter les requetes

```javascript
// Limite globale : 100 requetes / 15 min par IP
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  standardHeaders: true,    // Ajoute RateLimit-* headers
  legacyHeaders: false,
  message: {
    error: 'Trop de requetes, reessayez dans 15 minutes'
  },
  // Clef personnalisee : par IP + user ID si authentifie
  keyGenerator: (req) => {
    return req.user?.id ? `user_${req.user.id}` : req.ip;
  }
});

// Limite stricte sur l'auth : 5 tentatives / 15 min
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,  // Ne compter que les echecs
  message: { error: 'Trop de tentatives de connexion' }
});

app.use('/api/', globalLimiter);
app.use('/auth/login', authLimiter);
app.use('/auth/register', authLimiter);
```

### Validation avec Zod

```javascript
const { z } = require('zod'); // npm install zod

// Definir le schema une seule fois, reutiliser partout
const CreateUserSchema = z.object({
  email: z.string().email('Email invalide'),
  password: z.string()
    .min(8, 'Minimum 8 caracteres')
    .regex(/[A-Z]/, 'Au moins une majuscule')
    .regex(/[0-9]/, 'Au moins un chiffre'),
  age: z.number().int().min(13).max(120).optional(),
  role: z.enum(['user', 'moderator']).default('user')
});

// Middleware de validation generique
function validate(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(422).json({
        error: 'Donnees invalides',
        details: result.error.flatten().fieldErrors
      });
    }
    req.validatedBody = result.data; // Donnees nettoyees et typees
    next();
  };
}

// Utilisation
app.post('/users', validate(CreateUserSchema), async (req, res) => {
  const { email, password, role } = req.validatedBody; // Type-safe
  // ...
});
```

| Outil | Role | Note |
|---|---|---|
| `helmet` | Headers HTTP de securite | Toujours en premiere position |
| `cors` | Controle des origines | Whitelist explicite en prod |
| `express-rate-limit` | Anti-brute-force, anti-DDoS | Separater API / auth |
| `zod` | Validation et parsing des inputs | Prefere a joi pour TypeScript |
| `express-validator` | Validation chainee inline | Alternative a zod |
| `hpp` | Protection HTTP Parameter Pollution | Optionnel mais utile |

> [!warning] Securite : ce qui manque dans ce cours
> Ce cours couvre les bases. Une app en production necessite aussi : SQL injection prevention (ORM ou requetes parametrees), XSS sanitization (DOMPurify cote client), CSRF tokens, audit de dependances (`npm audit`), secrets dans des variables d'environnement (jamais dans le code).

---

## Testing avec Jest et Supertest

### Architecture de test recommandee

```
tests/
  unit/               Tests unitaires (logique pure, sans HTTP)
    userModel.test.js
    tokenUtils.test.js
  integration/        Tests d'integration (routes HTTP reelles)
    auth.test.js
    users.test.js
  fixtures/           Donnees de test reutilisables
    users.js
```

### Configuration Jest

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  setupFilesAfterFramework: ['./tests/setup.js'],
  testMatch: ['**/*.test.js']
};
```

### Tests d'integration avec Supertest

```javascript
// tests/integration/auth.test.js
const request = require('supertest'); // npm install --save-dev supertest jest
const app = require('../../app');     // Ton app Express SANS .listen()

describe('Authentication Routes', () => {
  describe('POST /auth/register', () => {
    it('devrait creer un compte avec des donnees valides', async () => {
      const response = await request(app)
        .post('/auth/register')
        .send({
          email: 'test@example.com',
          password: 'SecurePass1'
        })
        .expect('Content-Type', /json/)
        .expect(201);

      expect(response.body).toMatchObject({
        message: 'Compte cree',
        user: {
          email: 'test@example.com'
        }
      });
      expect(response.body.user).not.toHaveProperty('passwordHash');
    });

    it('devrait refuser un email invalide', async () => {
      const response = await request(app)
        .post('/auth/register')
        .send({ email: 'pas-un-email', password: 'SecurePass1' })
        .expect(422);

      expect(response.body.errors).toBeDefined();
    });

    it('devrait retourner 409 si email deja pris', async () => {
      // Creer d'abord
      await request(app)
        .post('/auth/register')
        .send({ email: 'duplicate@example.com', password: 'SecurePass1' });

      // Recreer avec le meme email
      await request(app)
        .post('/auth/register')
        .send({ email: 'duplicate@example.com', password: 'SecurePass1' })
        .expect(409);
    });
  });

  describe('Routes protegees', () => {
    let accessToken;

    // Se connecter avant chaque test de ce bloc
    beforeEach(async () => {
      await request(app)
        .post('/auth/register')
        .send({ email: 'user@test.com', password: 'SecurePass1' });

      const res = await request(app)
        .post('/auth/login')
        .send({ email: 'user@test.com', password: 'SecurePass1' });

      accessToken = res.body.accessToken;
    });

    it('devrait acceder au profil avec un token valide', async () => {
      await request(app)
        .get('/auth/profile')
        .set('Authorization', `Bearer ${accessToken}`)
        .expect(200);
    });

    it('devrait refuser sans token', async () => {
      await request(app)
        .get('/auth/profile')
        .expect(401);
    });

    it('devrait refuser avec un token expire', async () => {
      const jwt = require('jsonwebtoken');
      const expiredToken = jwt.sign(
        { sub: '123', email: 'user@test.com' },
        process.env.JWT_ACCESS_SECRET || 'change-me-in-production',
        { expiresIn: '0s' } // Expire immediatement
      );

      await request(app)
        .get('/auth/profile')
        .set('Authorization', `Bearer ${expiredToken}`)
        .expect(401);
    });
  });
});
```

### Test unitaire — Logique metier pure

```javascript
// tests/unit/tokenUtils.test.js
const jwt = require('jsonwebtoken');

// Extraire la logique en fonctions testables
const { generateAccessToken, verifyAccessToken } = require('../../utils/tokenUtils');

describe('Token Utils', () => {
  const mockUser = { id: '123', email: 'test@test.com' };

  describe('generateAccessToken', () => {
    it('devrait generer un token JWT valide', () => {
      const token = generateAccessToken(mockUser.id, mockUser.email);
      expect(typeof token).toBe('string');
      expect(token.split('.')).toHaveLength(3); // header.payload.signature
    });

    it('le payload devrait contenir id et email', () => {
      const token = generateAccessToken(mockUser.id, mockUser.email);
      const decoded = jwt.decode(token);
      expect(decoded.sub).toBe(mockUser.id);
      expect(decoded.email).toBe(mockUser.email);
    });
  });

  describe('verifyAccessToken', () => {
    it('devrait valider un token valide', () => {
      const token = generateAccessToken(mockUser.id, mockUser.email);
      const payload = verifyAccessToken(token);
      expect(payload.sub).toBe(mockUser.id);
    });

    it('devrait lever une erreur sur token invalide', () => {
      expect(() => verifyAccessToken('token.invalide.ici')).toThrow();
    });
  });
});
```

> [!info] app.js vs server.js — Pattern de test
> Pour que Supertest fonctionne, `app.js` doit exporter l'app Express **sans** appeler `app.listen()`. Le fichier `server.js` (point d'entree) fait le `.listen()`. Supertest cree son propre serveur interne.

---

## PM2 en Production

### Pourquoi PM2 ?

Sans PM2, si ton process Node.js crash, l'app est morte jusqu'a ce que tu relances manuellement. PM2 est un **process manager** qui : relance automatiquement, gere les logs, permet le hot reload, expose des metriques.

```
npm install -g pm2
```

### Commandes essentielles

```bash
# Lancer une app
pm2 start app.js --name "mon-api"

# Lancer en mode cluster (un process par core)
pm2 start app.js --name "mon-api" -i max

# Voir le statut de toutes les apps
pm2 list

# Logs en temps reel
pm2 logs mon-api

# Logs avec filtrage
pm2 logs mon-api --lines 100 --err

# Monitoring dashboard temps reel
pm2 monit

# Recharger sans downtime (0-downtime reload)
pm2 reload mon-api

# Restart avec downtime (si reload ne fonctionne pas)
pm2 restart mon-api

# Arreter
pm2 stop mon-api

# Supprimer du registre PM2
pm2 delete mon-api
```

### ecosystem.config.js — Configuration versionnee

```javascript
// ecosystem.config.js (commiter dans le repo)
module.exports = {
  apps: [
    {
      name: 'mon-api',
      script: './src/server.js',
      instances: 'max',           // Un worker par core
      exec_mode: 'cluster',       // Mode cluster

      // Variables d'environnement par environment
      env: {
        NODE_ENV: 'development',
        PORT: 3000
      },
      env_production: {
        NODE_ENV: 'production',
        PORT: 8080
      },

      // Restart automatique
      watch: false,               // Desactiver en prod (utiliser reload)
      max_memory_restart: '500M', // Restart si depasse 500 Mo (memory leak)
      restart_delay: 1000,        // Attendre 1s avant restart
      max_restarts: 10,           // Max 10 restarts en boucle
      min_uptime: '5s',           // Doit vivre 5s pour etre "stable"

      // Logs
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      error_file: './logs/error.log',
      out_file: './logs/out.log',
      merge_logs: true,           // Fusionner les logs de tous les workers

      // Graceful shutdown
      kill_timeout: 5000,         // 5s pour finir les requetes en cours
      listen_timeout: 3000,       // 3s pour que le process soit "ready"
      wait_ready: true            // Attendre process.send('ready')
    }
  ]
};
```

```javascript
// Dans server.js — signaler a PM2 que l'app est prete
const server = app.listen(PORT, () => {
  console.log(`Listening on ${PORT}`);
  if (process.send) {
    process.send('ready'); // Signal PM2 : app operationnelle
  }
});

// Graceful shutdown
process.on('SIGINT', () => {
  server.close(() => {
    console.log('HTTP server closed');
    // Fermer les connexions DB, etc.
    process.exit(0);
  });
});
```

### Demarrage automatique au boot

```bash
# Generer le script de demarrage pour ton OS
pm2 startup

# Sauvegarder l'etat actuel (apps en cours)
pm2 save

# Pour supprimer le startup
pm2 unstartup
```

| Commande PM2 | Effet |
|---|---|
| `pm2 start -i max` | Cluster avec tous les cores |
| `pm2 reload app` | 0-downtime reload (cluster uniquement) |
| `pm2 monit` | Dashboard CPU/RAM/logs live |
| `pm2 plus` | Dashboard web avec alertes (payant) |
| `pm2 logs --json` | Logs en format JSON |
| `pm2 flush` | Vider les fichiers de log |
| `pm2 reset app` | Remettre les compteurs a zero |

---

## Profiling et Performance

### --inspect — DevTools pour Node.js

```bash
# Lancer avec le debugger active
node --inspect app.js

# Ou pour attacher a un process existant
node --inspect-brk app.js   # Pause au demarrage

# Ouvrir Chrome et aller a : chrome://inspect
# Cliquer "inspect" sous le process Node.js detecte
```

Dans Chrome DevTools, onglet **Memory** :
- **Heap Snapshot** : photo de la memoire a un instant T
- **Allocation Timeline** : voir les allocations en temps reel
- **Allocation Sampling** : profiling allege (moins de surcharge)

Onglet **Profiler** (CPU) :
- Enregistrer pendant une charge pour voir quelles fonctions consomment le plus de CPU.

### Detecter les memory leaks

```javascript
// Exemple de memory leak classique : event listeners qui s'accumulent
const EventEmitter = require('events');

const emitter = new EventEmitter();

// BUG : chaque appel de setup() ajoute un listener sans jamais le retirer
function setup(userId) {
  emitter.on('data', (data) => {
    console.log(`User ${userId}: ${data}`);
  });
}

// Apres 1000 appels, 1000 listeners accumules -> WARNING: MaxListenersExceededWarning
```

```javascript
// FIX 1 : retirer le listener apres usage
function setup(userId) {
  const handler = (data) => console.log(`User ${userId}: ${data}`);
  emitter.on('data', handler);

  // Retirer apres 1 utilisation
  emitter.once('disconnect', () => emitter.off('data', handler));
}

// FIX 2 : utiliser once() si le listener ne doit s'executer qu'une fois
emitter.once('data', handler);
```

```javascript
// Surveiller la memoire programmatiquement
function logMemoryUsage() {
  const used = process.memoryUsage();
  console.log({
    heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)} MB`,
    heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)} MB`,
    rss: `${Math.round(used.rss / 1024 / 1024)} MB`,     // Total process
    external: `${Math.round(used.external / 1024 / 1024)} MB` // C++ objects
  });
}

setInterval(logMemoryUsage, 5000);
```

### clinic.js — Profiling avance

```bash
# npm install -g clinic

# Doctor : analyse globale (CPU, memoire, event loop lag)
clinic doctor -- node app.js

# Flame : flamegraph CPU (quelles fonctions consomment le plus)
clinic flame -- node app.js

# Bubbleprof : analyser l'async (ou le temps est passe entre async operations)
clinic bubbleprof -- node app.js
```

```
Flamegraph CPU (lecture) :

                         [handleRequest]
               [authenticate]  [queryDB]      [serialize]
          [jwt.verify]   [pg.query]     [findUsers]   [JSON.stringify]
     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
     Plus la barre est large, plus la fonction consomme du CPU.
     Chercher les larges barres inattendues dans des fonctions tierces.
```

### Benchmark avec autocannon

```bash
# npm install -g autocannon
# 100 connexions simultanees, 10 secondes
autocannon -c 100 -d 10 http://localhost:3000/api/users

# Avec headers (token JWT)
autocannon -c 50 -d 30 \
  -H "Authorization: Bearer <ton-token>" \
  http://localhost:3000/api/protected
```

Resultats :

```
Stat         2.5%    50%    97.5%   99%   Avg     Stddev   Max
Latency (ms)  1      3      12      18    3.5     2.1      45
Req/sec       -       -      -       -   2847.3   312.4   3102
```

### Patterns de performance courants

```javascript
// 1. Eviter les calculs repetes : memoization
const cache = new Map();

function expensiveCalc(input) {
  if (cache.has(input)) return cache.get(input);

  const result = /* calcul lourd */ input * 2;

  cache.set(input, result);
  if (cache.size > 1000) {
    // Eviter la croissance infinie
    const firstKey = cache.keys().next().value;
    cache.delete(firstKey);
  }

  return result;
}

// 2. Batch les requetes DB : eviter N+1 queries
// Mauvais : 1 requete par user
async function getUsersWithOrders_BAD(userIds) {
  const result = [];
  for (const id of userIds) {
    const user = await db.query('SELECT * FROM users WHERE id = $1', [id]);
    const orders = await db.query('SELECT * FROM orders WHERE user_id = $1', [id]);
    result.push({ user, orders });
  }
  return result;
}

// Bon : 2 requetes total, peu importe le nombre d'users
async function getUsersWithOrders_GOOD(userIds) {
  const [users, orders] = await Promise.all([
    db.query('SELECT * FROM users WHERE id = ANY($1)', [userIds]),
    db.query('SELECT * FROM orders WHERE user_id = ANY($1)', [userIds])
  ]);

  const ordersByUser = {};
  orders.rows.forEach(o => {
    ordersByUser[o.user_id] = ordersByUser[o.user_id] || [];
    ordersByUser[o.user_id].push(o);
  });

  return users.rows.map(u => ({ ...u, orders: ordersByUser[u.id] || [] }));
}
```

> [!warning] Event Loop Lag
> Si une operation synchrone prend > 10ms, elle cree un "lag" de l'event loop visible pour tous les clients. `clinic doctor` detecte ca. Solutions : deporter en Worker Thread, decouper en micro-taches avec `setImmediate`, ou utiliser un streaming approach.

---

## Carte Mentale

```
                          NODE.JS AVANCE
                               |
          ┌────────────────────┼────────────────────┐
          │                    │                    │
    PARALLELISME          PRODUCTION            SECURITE
          │                    │                    │
    ┌─────┴─────┐        ┌─────┴─────┐        ┌─────┴─────┐
    │           │        │           │        │           │
 Worker      Child     Cluster      PM2    Helmet +    JWT
 Threads   Processes    Module            CORS +     Auth
    │           │        │           │    Rate Limit    │
SharedArray  exec/spawn  fork()   ecosystem  Zod      bcrypt +
Buffer       fork()     round-   .config.js Validation refresh
Atomics      pipeline   robin                         tokens
    │
    │
 STREAMS                 TESTS               PERF
    │                    │                    │
Transform         Jest + Supertest       --inspect
pipeline()        integration         clinic.js
backpressure      unit tests         autocannon
objectMode        fixtures           flamegraph
                  beforeEach         memoryUsage()
                                     N+1 avoidance

                    WEBSOCKETS
                         │
                    ws server
                    rooms (Map)
                    broadcast
                    heartbeat (ping/pong)
                    binary messages
```

---

## Exercices Pratiques

### Exercice 1 — Worker Thread pour le hachage de mots de passe

**Contexte** : `bcrypt.hash()` est CPU-intensif. Sur un serveur avec de nombreuses inscriptions simultanees, il peut bloquer l'event loop. Deporter le hachage dans un Worker Thread.

**Objectif** :
1. Creer `workers/bcrypt-worker.js` qui recoit `{ password, saltRounds }` via `workerData`, hache le mot de passe, et retourne le hash via `parentPort.postMessage()`
2. Creer `utils/hashPassword.js` qui instancie le worker et retourne une Promise
3. Ecrire un test avec Jest qui verifie que la valeur retournee peut etre verifiee avec `bcrypt.compare()`

**Bonus** : Creer un pool de 4 workers de hachage et mesurer la difference de throughput avec autocannon sur une route `/register`.

---

### Exercice 2 — API REST securisee complete

**Objectif** : Construire une API `TODO` avec authentification JWT complete.

**Endpoints requis** :
```
POST   /auth/register     Inscription
POST   /auth/login        Connexion (retourne access + refresh token)
POST   /auth/refresh      Renouveler l'access token
POST   /auth/logout       Revoquer le refresh token
GET    /todos             Lister les todos de l'utilisateur (auth requise)
POST   /todos             Creer un todo (auth requise)
PATCH  /todos/:id         Modifier un todo (auth + owner uniquement)
DELETE /todos/:id         Supprimer (auth + owner uniquement)
```

**Contraintes** :
- Valider tous les inputs avec Zod
- Rate limit sur `/auth/login` et `/auth/register` : 5 requetes / 15 min
- Helmet et CORS configures
- Ecrire les tests Supertest pour au moins 3 endpoints

---

### Exercice 3 — Traitement de fichier CSV en streaming

**Objectif** : Creer une route `POST /import/users` qui accepte un fichier CSV et insere les utilisateurs sans charger tout le fichier en memoire.

**Format CSV** :
```
email,name,age,role
alice@test.com,Alice,28,user
bob@test.com,Bob,35,admin
```

**Etapes** :
1. Creer un `CsvParserTransform` (Transform stream) qui convert les lignes en objets
2. Creer un `ValidationTransform` qui filtre les lignes invalides (email malformed, age < 0)
3. Creer un `BatchInsertWritable` (Writable) qui accumule 100 lignes et les insere en batch (simuler avec un `console.log`)
4. Connecter les trois avec `pipeline()`
5. Retourner `{ imported: N, rejected: M }` quand le pipeline est termine

**Indicateur de succes** : le script doit traiter un CSV de 1 million de lignes en moins de 10 secondes sans depasser 100 Mo de RAM.

---

### Exercice 4 — Chat en temps reel avec WebSockets

**Objectif** : Construire un serveur de chat avec authentification JWT sur la connexion WebSocket.

**Fonctionnalites** :
1. Lors de la connexion WS, le client envoie son JWT dans le header ou en premier message `{ type: 'AUTH', token: '...' }`
2. Si le token est invalide : fermer la connexion avec le code `4001`
3. Si valide : associer `ws.userId` et `ws.email` au socket
4. Permettre de rejoindre des rooms : `{ type: 'JOIN', room: 'general' }`
5. Broadcaster les messages dans la room avec l'info de l'expediteur
6. Implémenter le heartbeat ping/pong (deconnecter les clients morts apres 60s)

**Bonus** : Persister les 50 derniers messages de chaque room en memoire et les envoyer au client qui rejoint la room (`{ type: 'HISTORY', messages: [...] }`).

---

*Notes complementaires : [[Node Express/01 - Express.js et NestJS]] | [[Node Express/02 - Nodejs Fondamentaux]] | [[JavaScript/03 - JavaScript Asynchrone]]*
