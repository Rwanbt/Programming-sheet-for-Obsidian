# Message Queues et Files d'Attente

Les **message queues** (files d'attente de messages) sont un mécanisme fondamental de l'architecture logicielle moderne qui permet à des systèmes de communiquer de façon **asynchrone** et **découplée**. Elles sont omniprésentes dans les applications web à fort trafic : envoi d'emails, traitement d'images, notifications push, intégrations webhook. Ce cours couvre les concepts théoriques, les outils principaux de l'écosystème Node.js (BullMQ) et Python (Celery), le broker RabbitMQ, et les Redis Streams.

---

## 1. Pourquoi les Message Queues ?

### 1.1 Le problème du traitement synchrone

Dans une architecture web classique, chaque requête HTTP déclenche un traitement **synchrone** : l'utilisateur attend la réponse complète avant de continuer. Cela pose des problèmes dès que le traitement devient long ou incertain.

```
Client → [Requête HTTP] → Serveur → [Traitement 10s] → [Réponse]
                                         ↑
                               L'utilisateur attend 10 secondes
                               La connexion reste ouverte
                               Le serveur est bloqué
```

Exemples concrets de traitements coûteux à ne **jamais** faire de façon synchrone :
- Envoi d'email via SMTP (latence réseau, serveur SMTP surchargé)
- Redimensionnement / compression d'images uploadées
- Génération de PDF ou rapports
- Appels à des APIs tierces peu fiables
- Mise à jour d'index de recherche
- Envoi de notifications push à des milliers d'utilisateurs

### 1.2 Le découplage producteur / consommateur

Une queue introduit un **intermédiaire** entre celui qui produit le travail et celui qui l'exécute :

```
                    ┌─────────────┐
 Producteur ───────▶│   QUEUE     │───────▶ Consommateur
(Serveur web)       │  [job1]     │         (Worker)
                    │  [job2]     │
                    │  [job3]     │
                    └─────────────┘
```

**Avantages du découplage :**
- Le serveur web répond immédiatement à l'utilisateur (`202 Accepted`)
- Le worker traite les jobs à son rythme
- Si le worker crashe, les jobs ne sont pas perdus (ils restent en queue)
- On peut scaler les workers indépendamment du serveur web

### 1.3 Absorption des pics de charge

Sans queue, un pic de trafic submerge directement le serveur :

```
Trafic normal  : ████ ████ ████  →  Serveur OK
Pic de trafic  : ████████████████  →  Serveur surchargé / crash

Avec une queue :
Pic de trafic  : ████████████████  →  Queue absorbe
                                       ↓
                                   Workers traitent progressivement
                                   ████ ████ ████ ████  →  Stable
```

> [!info] Analogie du restaurant
> La queue fonctionne comme la salle d'attente d'un restaurant. Les clients (jobs) arrivent à leur rythme. Les cuisiniers (workers) traitent les commandes dans l'ordre, sans être submergés. Si un cuisinier tombe malade, les autres continuent. Si beaucoup de clients arrivent, on ajoute des cuisiniers temporairement.

### 1.4 Retry automatique et résilience

Les queues permettent de **retenter automatiquement** les jobs échoués, ce qui est impossible en synchrone :

```
Job envoyé → Worker tente de traiter
                  ↓
             [Erreur réseau]
                  ↓
             Retry dans 30s → [Erreur réseau]
                  ↓
             Retry dans 60s → [Succès]
```

Sans queue, une erreur réseau temporaire lors de l'envoi d'un email ferait simplement rater l'email. Avec une queue, le job est retentée automatiquement jusqu'au succès.

### 1.5 Résumé des cas d'usage

| Cas d'usage | Pourquoi une queue ? |
|---|---|
| Envoi d'emails transactionnels | SMTP lent, séparation des responsabilités |
| Traitement d'images uploadées | CPU-intensif, long |
| Notifications push | Fan-out vers des milliers d'utilisateurs |
| Import CSV en masse | Long, peut échouer à mi-chemin |
| Webhooks sortants | API tierce peut être down |
| Génération de rapports | Résultat récupéré plus tard |
| Synchronisation entre microservices | Découplement fort |

---

## 2. Patterns de Message Queues

### 2.1 Work Queue (File de travail)

Le pattern le plus simple : une queue, plusieurs workers. Chaque message est traité par **un seul worker** (compétition).

```
                         ┌──────────┐
                    ┌───▶│ Worker 1 │
Producteur ──▶ Queue│    └──────────┘
                    │    ┌──────────┐
                    └───▶│ Worker 2 │
                         └──────────┘
```

**Cas d'usage :** traitement d'images, envoi d'emails, jobs CPU-intensifs.

**Comportement :** les jobs sont distribués en round-robin entre les workers disponibles. Si un worker est occupé, le job va au suivant.

### 2.2 Pub/Sub (Publication / Abonnement)

Un message est publié sur un **canal** (topic). **Tous les abonnés** à ce canal reçoivent une copie du message.

```
                    ┌─────────────────┐
                    │   Subscriber 1  │ (Service Email)
Producteur ──▶ Topic│   Subscriber 2  │ (Service Analytics)
                    │   Subscriber 3  │ (Service Audit)
                    └─────────────────┘
```

**Cas d'usage :** broadcast d'événements système, mises à jour en temps réel, synchronisation de caches.

**Différence avec Work Queue :** dans pub/sub, **tous** les abonnés reçoivent le message. Dans une work queue, **un seul** worker traite chaque message.

### 2.3 Fan-Out

Variante du pub/sub où un message est distribué à **plusieurs queues** (et non directement à des consommateurs).

```
                    ┌──▶ Queue A ──▶ Workers A
Producteur ──▶ Exchange
                    └──▶ Queue B ──▶ Workers B
```

**Cas d'usage :** un événement `order.placed` doit déclencher à la fois l'envoi d'un email de confirmation ET la mise à jour du stock ET la génération d'une facture.

### 2.4 Topic Exchange

Variante avancée du fan-out avec **routage par pattern**. Les messages ont une routing key, et les queues s'abonnent à des patterns.

```
Routing key: "order.placed.fr"
─────────────────────────────────────────────────────
Pattern "order.#"     ──▶ Queue Commandes    ✓ (match)
Pattern "order.*.fr"  ──▶ Queue France       ✓ (match)
Pattern "user.#"      ──▶ Queue Utilisateurs ✗ (no match)
```

**Wildcards RabbitMQ :**
- `*` remplace exactement un mot
- `#` remplace zéro ou plusieurs mots

### 2.5 RPC over Queue (Remote Procedure Call)

La queue peut simuler un appel de fonction distant : le producteur envoie un job et **attend le résultat** via une queue de réponse.

```
Client ──▶ [request queue] ──▶ Server
  ▲                               │
  └──── [reply queue] ◀───────────┘
```

> [!warning] Attention au RPC over Queue
> Ce pattern réintroduit la synchronicité. À utiliser uniquement quand le résultat est strictement nécessaire immédiatement. Dans la plupart des cas, il vaut mieux utiliser un result backend (stocker le résultat et le récupérer plus tard avec l'ID du job).

---

## 3. Garanties de Livraison

### 3.1 At-Most-Once (Au plus une fois)

Le message est envoyé une seule fois. S'il est perdu en route, tant pis.

```
Producteur ──▶ [Message] ──▶ Consommateur
                    ↑
              Peut être perdu
              Jamais dupliqué
```

**Quand l'utiliser :** métriques non critiques, logs de débogage, données facilement recalculables.

**Implémentation Redis :** `LPUSH` sans confirmation de traitement.

### 3.2 At-Least-Once (Au moins une fois)

Le message est garanti d'être livré, mais peut être livré **plusieurs fois** en cas de crash du worker avant l'acquittement.

```
Producteur ──▶ [Message] ──▶ Consommateur
                                  │
                            [Traitement...]
                                  │
                            [CRASH avant ACK]
                                  ↓
                    [Message remis en queue] ──▶ Nouveau Worker
```

**Quand l'utiliser :** la majorité des cas. Traitement d'emails, d'images, webhooks.

**Contrainte :** le traitement doit être **idempotent** (exécuter deux fois ne cause pas de problème). Exemple : "envoyer un email avec ID xxx" est idempotent si on vérifie l'ID avant d'envoyer.

> [!tip] Idempotence
> Un traitement est idempotent si l'exécuter plusieurs fois donne le même résultat qu'une seule exécution. Pour garantir l'idempotence : stocker l'ID du job traité en base, vérifier avant traitement si l'ID existe déjà.

### 3.3 Exactly-Once (Exactement une fois)

Le message est traité **exactement une fois**, même en cas de crash. C'est le plus difficile à implémenter.

```
Producteur ──▶ [Message + ID unique] ──▶ Consommateur
                                              │
                                    [Vérifier si ID traité]
                                              │
                                    [Traiter + Marquer ID]
                                              │
                                    [ACK atomique]
```

**Implémentation :** transaction base de données englobant le traitement ET l'acquittement. Ou utilisation de Kafka avec `enable.idempotence=true`.

**Coût :** performances réduites, complexité accrue.

| Garantie | Performance | Complexité | Duplication possible | Perte possible |
|---|---|---|---|---|
| At-most-once | Très haute | Faible | Non | Oui |
| At-least-once | Haute | Moyenne | Oui | Non |
| Exactly-once | Faible | Haute | Non | Non |

> [!info] Recommandation pratique
> Dans 90% des cas, **at-least-once avec idempotence** est le bon choix. Les systèmes exactly-once ont un coût en performance et complexité rarement justifié. La vraie question est : "Est-ce que mon traitement est idempotent ?"

---

## 4. BullMQ — Queues Node.js avec Redis

### 4.1 Présentation et Architecture

**BullMQ** est la version moderne de Bull, une bibliothèque Node.js de gestion de queues basée sur **Redis**. Redis stocke les jobs sous forme de sorted sets et lists.

```
                    ┌──────────────────────────────┐
                    │           REDIS               │
                    │  bull:myqueue:waiting  [list] │
                    │  bull:myqueue:active   [list] │
                    │  bull:myqueue:failed   [list] │
                    │  bull:myqueue:completed[list] │
                    └──────────────────────────────┘
                           ▲              ▲
                           │              │
              ┌────────────┴─┐     ┌──────┴──────────┐
              │   Queue      │     │    Worker        │
              │  (producteur)│     │  (consommateur)  │
              └──────────────┘     └─────────────────-┘
```

**Installation :**

```bash
npm install bullmq
npm install ioredis  # optionnel, BullMQ utilise ioredis en interne

# Redis doit tourner localement ou via Docker
docker run -d -p 6379:6379 redis:alpine
```

### 4.2 Création d'une Queue

```javascript
// queues/emailQueue.js
import { Queue } from 'bullmq';

// Configuration de la connexion Redis
const redisConnection = {
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT) || 6379,
  password: process.env.REDIS_PASSWORD || undefined,
};

// Création de la queue
export const emailQueue = new Queue('email-queue', {
  connection: redisConnection,
  defaultJobOptions: {
    // Options par défaut pour tous les jobs de cette queue
    attempts: 3,           // Nombre de tentatives max
    backoff: {
      type: 'exponential', // Délai exponentiel entre les tentatives
      delay: 1000,         // Délai initial : 1s, puis 2s, 4s...
    },
    removeOnComplete: {
      age: 3600,   // Garder les jobs complétés pendant 1 heure
      count: 1000, // Garder au max 1000 jobs complétés
    },
    removeOnFail: {
      age: 24 * 3600, // Garder les jobs échoués pendant 24h pour audit
    },
  },
});

// Nettoyage propre à l'arrêt de l'application
process.on('SIGTERM', async () => {
  await emailQueue.close();
});
```

### 4.3 Ajout de Jobs — Options Complètes

```javascript
// Ajout simple
await emailQueue.add('send-welcome', {
  to: 'user@example.com',
  name: 'Alice',
  template: 'welcome',
});

// Ajout avec options spécifiques au job
await emailQueue.add('send-invoice', {
  to: 'client@example.com',
  invoiceId: 'INV-2024-001',
  amount: 150.00,
}, {
  // --- Priorité (1 = haute, 100 = basse) ---
  priority: 1,

  // --- Délai avant traitement ---
  delay: 5 * 60 * 1000, // Traiter dans 5 minutes

  // --- Tentatives ---
  attempts: 5,
  backoff: {
    type: 'exponential',
    delay: 2000, // 2s, 4s, 8s, 16s, 32s
  },

  // --- Job unique (évite les doublons) ---
  jobId: `invoice-${invoiceId}`, // ID custom = pas de doublon si même ID

  // --- Timeout ---
  timeout: 30000, // Échouer si le worker prend plus de 30s

  // --- Répétition (CRON) ---
  // repeat: { cron: '0 9 * * 1-5' }, // Lundi-Vendredi à 9h
  // repeat: { every: 60000 },         // Toutes les 60 secondes
});

// Ajout groupé (bulk)
const jobs = [
  { name: 'send-newsletter', data: { userId: 1 } },
  { name: 'send-newsletter', data: { userId: 2 } },
  { name: 'send-newsletter', data: { userId: 3 } },
];
await emailQueue.addBulk(jobs);
```

> [!tip] Job IDs uniques
> Utiliser un `jobId` custom est la façon la plus simple d'éviter les doublons. Si vous ajoutez deux fois un job avec le même `jobId`, le second est ignoré (ou remplace le premier selon la config). Très utile pour les opérations "upsert" ou les jobs déclenchés par des webhooks.

### 4.4 Worker — Traitement des Jobs

```javascript
// workers/emailWorker.js
import { Worker } from 'bullmq';
import nodemailer from 'nodemailer';

const redisConnection = {
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT) || 6379,
};

// Simulateur d'envoi d'email
const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST,
  port: 587,
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS,
  },
});

// Définition du processeur de jobs
async function processEmailJob(job) {
  // job.name  = nom du job ('send-welcome', 'send-invoice', etc.)
  // job.data  = données passées lors de l'ajout
  // job.id    = identifiant unique du job
  // job.attemptsMade = nombre de tentatives déjà effectuées

  console.log(`[Worker] Traitement job ${job.id} (${job.name}), tentative ${job.attemptsMade + 1}`);

  // Mise à jour du progrès (visible dans Bull Board)
  await job.updateProgress(10);

  // Routage par nom de job
  switch (job.name) {
    case 'send-welcome':
      await sendWelcomeEmail(job.data);
      break;
    case 'send-invoice':
      await sendInvoiceEmail(job.data);
      break;
    case 'send-newsletter':
      await sendNewsletterEmail(job.data);
      break;
    default:
      throw new Error(`Type de job inconnu : ${job.name}`);
  }

  await job.updateProgress(100);

  // La valeur de retour est stockée comme résultat du job
  return { success: true, sentAt: new Date().toISOString() };
}

async function sendWelcomeEmail({ to, name, template }) {
  await transporter.sendMail({
    from: '"Mon App" <noreply@monapp.com>',
    to,
    subject: `Bienvenue, ${name} !`,
    html: `<h1>Bonjour ${name}</h1><p>Merci de vous être inscrit.</p>`,
  });
}

async function sendInvoiceEmail({ to, invoiceId, amount }) {
  // Logique d'envoi de facture...
  await transporter.sendMail({
    from: '"Facturation" <billing@monapp.com>',
    to,
    subject: `Votre facture ${invoiceId}`,
    html: `<p>Montant : ${amount}€</p>`,
  });
}

async function sendNewsletterEmail({ userId }) {
  // Récupérer l'email depuis la BDD, envoyer la newsletter...
  console.log(`Newsletter envoyée à userId ${userId}`);
}

// Création du worker
const emailWorker = new Worker('email-queue', processEmailJob, {
  connection: redisConnection,
  concurrency: 5,  // Traiter jusqu'à 5 jobs simultanément

  // Limiter la vitesse de traitement (rate limiting)
  limiter: {
    max: 100,      // Max 100 jobs
    duration: 60000, // par minute
  },
});

// Événements du worker
emailWorker.on('completed', (job, result) => {
  console.log(`✅ Job ${job.id} complété :`, result);
});

emailWorker.on('failed', (job, error) => {
  console.error(`❌ Job ${job?.id} échoué (tentative ${job?.attemptsMade}) :`, error.message);
});

emailWorker.on('progress', (job, progress) => {
  console.log(`⏳ Job ${job.id} : ${progress}%`);
});

emailWorker.on('error', (error) => {
  // Erreur de connexion Redis, etc.
  console.error('Erreur worker :', error);
});

// Nettoyage propre
process.on('SIGTERM', async () => {
  console.log('Arrêt du worker en cours...');
  await emailWorker.close();
  console.log('Worker arrêté proprement.');
});

console.log('Worker email démarré, en attente de jobs...');
```

### 4.5 QueueScheduler et Jobs Répétés

```javascript
// workers/scheduledJobs.js
import { Queue } from 'bullmq';

const reportQueue = new Queue('reports', { connection: redisConnection });

// Job récurrent avec cron
await reportQueue.add(
  'daily-report',
  { type: 'daily', recipients: ['admin@monapp.com'] },
  {
    repeat: {
      cron: '0 8 * * *', // Tous les jours à 8h
      tz: 'Europe/Paris', // Fuseau horaire
    },
  }
);

// Job répété toutes les heures
await reportQueue.add(
  'hourly-stats',
  { metric: 'active-users' },
  {
    repeat: {
      every: 60 * 60 * 1000, // 1 heure en ms
      limit: 24,              // Maximum 24 répétitions
    },
  }
);

// Lister les jobs récurrents
const repeatableJobs = await reportQueue.getRepeatableJobs();
console.log('Jobs récurrents :', repeatableJobs);

// Supprimer un job récurrent
await reportQueue.removeRepeatableByKey(repeatableJobs[0].key);
```

> [!warning] QueueScheduler retiré dans BullMQ v3+
> Dans BullMQ v3 et supérieur, le `QueueScheduler` a été supprimé. Ses fonctionnalités sont maintenant intégrées directement dans le `Worker`. Il n'est plus nécessaire de l'instancier séparément. Si vous voyez du code avec `new QueueScheduler(...)`, c'est du code BullMQ v1/v2.

### 4.6 Bull Board — Dashboard Visuel

**Bull Board** est une interface web pour visualiser et gérer les queues BullMQ.

```bash
npm install @bull-board/api @bull-board/express
```

```javascript
// app.js (ou server.js)
import express from 'express';
import { createBullBoard } from '@bull-board/api';
import { BullMQAdapter } from '@bull-board/api/bullMQAdapter.js';
import { ExpressAdapter } from '@bull-board/express';
import { emailQueue } from './queues/emailQueue.js';
import { reportQueue } from './queues/reportQueue.js';

const app = express();

// Configuration de Bull Board
const serverAdapter = new ExpressAdapter();
serverAdapter.setBasePath('/admin/queues');

createBullBoard({
  queues: [
    new BullMQAdapter(emailQueue),
    new BullMQAdapter(reportQueue),
  ],
  serverAdapter,
});

// Monter le dashboard (protéger en production !)
app.use('/admin/queues', serverAdapter.getRouter());

// Protection basique par mot de passe
app.use('/admin', (req, res, next) => {
  const authHeader = req.headers['authorization'];
  if (!authHeader || authHeader !== `Bearer ${process.env.ADMIN_TOKEN}`) {
    return res.status(401).json({ error: 'Non autorisé' });
  }
  next();
});

app.listen(3000, () => {
  console.log('Dashboard disponible sur http://localhost:3000/admin/queues');
});
```

**Fonctionnalités de Bull Board :**
- Voir les jobs en attente, actifs, complétés, échoués
- Relancer manuellement un job échoué
- Vider une queue
- Voir les logs et données de chaque job
- Visualiser le progrès en temps réel

### 4.7 Événements de Queue et Monitoring

```javascript
// Écouter les événements de la queue (dans le serveur principal)
import { QueueEvents } from 'bullmq';

const emailQueueEvents = new QueueEvents('email-queue', {
  connection: redisConnection,
});

emailQueueEvents.on('completed', ({ jobId, returnvalue }) => {
  console.log(`Job ${jobId} terminé avec résultat :`, returnvalue);
});

emailQueueEvents.on('failed', ({ jobId, failedReason }) => {
  console.error(`Job ${jobId} a échoué :`, failedReason);
  // Alerter via Sentry, Slack, etc.
});

emailQueueEvents.on('waiting', ({ jobId }) => {
  console.log(`Job ${jobId} en attente`);
});

emailQueueEvents.on('active', ({ jobId }) => {
  console.log(`Job ${jobId} en cours de traitement`);
});

// Attendre la complétion d'un job spécifique (pattern RPC)
const job = await emailQueue.add('send-welcome', { to: 'user@example.com' });
const result = await job.waitUntilFinished(emailQueueEvents, 30000); // timeout 30s
console.log('Job terminé :', result);
```

---

## 5. Exemple Pratique BullMQ — Traitement d'Images

### 5.1 Architecture

```
Utilisateur ──▶ POST /upload ──▶ Serveur Express ──▶ imageQueue.add()
                                                              │
                                                     Redis stocke le job
                                                              │
                                                    imageWorker récupère
                                                              │
                                               sharp() redimensionne l'image
                                                              │
                                               Fichier sauvegardé sur disque/S3
```

### 5.2 Code Complet

```javascript
// routes/upload.js
import express from 'express';
import multer from 'multer';
import { Queue } from 'bullmq';
import path from 'path';
import fs from 'fs';

const router = express.Router();
const upload = multer({ dest: 'uploads/temp/' });

const imageQueue = new Queue('image-processing', {
  connection: { host: 'localhost', port: 6379 },
});

router.post('/upload', upload.single('image'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'Aucune image fournie' });
  }

  try {
    // Ajouter le job de traitement
    const job = await imageQueue.add('resize', {
      originalPath: req.file.path,
      filename: req.file.filename,
      originalName: req.file.originalname,
      sizes: [
        { width: 1920, height: 1080, suffix: 'full' },
        { width: 800, height: 600, suffix: 'medium' },
        { width: 200, height: 200, suffix: 'thumbnail' },
      ],
    });

    res.status(202).json({
      message: 'Image en cours de traitement',
      jobId: job.id,
      statusUrl: `/jobs/${job.id}/status`,
    });
  } catch (error) {
    // Nettoyer le fichier temporaire en cas d'erreur
    fs.unlinkSync(req.file.path);
    res.status(500).json({ error: 'Erreur lors de la mise en queue' });
  }
});

// Endpoint pour vérifier le statut d'un job
router.get('/jobs/:jobId/status', async (req, res) => {
  const job = await imageQueue.getJob(req.params.jobId);
  if (!job) return res.status(404).json({ error: 'Job introuvable' });

  const state = await job.getState(); // waiting, active, completed, failed
  const progress = job.progress;

  res.json({
    jobId: job.id,
    state,
    progress,
    result: state === 'completed' ? job.returnvalue : null,
    error: state === 'failed' ? job.failedReason : null,
  });
});

export default router;
```

```javascript
// workers/imageWorker.js
import { Worker } from 'bullmq';
import sharp from 'sharp';
import path from 'path';
import fs from 'fs';

const OUTPUT_DIR = 'uploads/processed';
fs.mkdirSync(OUTPUT_DIR, { recursive: true });

async function processImageJob(job) {
  const { originalPath, filename, sizes } = job.data;
  const results = [];

  await job.updateProgress(0);

  for (let i = 0; i < sizes.length; i++) {
    const { width, height, suffix } = sizes[i];
    const outputFilename = `${filename}_${suffix}.webp`;
    const outputPath = path.join(OUTPUT_DIR, outputFilename);

    // Traitement avec sharp
    await sharp(originalPath)
      .resize(width, height, {
        fit: 'cover',       // Rogner pour remplir
        position: 'center', // Centrer le rognage
      })
      .webp({ quality: 85 }) // Convertir en WebP
      .toFile(outputPath);

    results.push({
      suffix,
      path: outputPath,
      url: `/images/${outputFilename}`,
    });

    // Mettre à jour le progrès
    await job.updateProgress(Math.round(((i + 1) / sizes.length) * 100));
  }

  // Supprimer le fichier temporaire
  fs.unlinkSync(originalPath);

  return { processedImages: results };
}

const imageWorker = new Worker('image-processing', processImageJob, {
  connection: { host: 'localhost', port: 6379 },
  concurrency: 3, // 3 images en parallèle max
});

imageWorker.on('completed', (job) => {
  console.log(`✅ Image ${job.data.originalName} traitée`);
});

imageWorker.on('failed', (job, error) => {
  console.error(`❌ Échec traitement ${job?.data?.originalName} :`, error.message);
  // Nettoyer le fichier temp si le job a complètement échoué
  if (job?.data?.originalPath && fs.existsSync(job.data.originalPath)) {
    fs.unlinkSync(job.data.originalPath);
  }
});
```

---

## 6. Celery — Queues Python

### 6.1 Architecture Celery

Celery est le système de traitement de tâches asynchrones de référence en Python. Il est très utilisé avec Django et Flask.

```
                    ┌──────────────────────────────────────┐
                    │          BROKER (Redis/RabbitMQ)      │
                    │  celery:default (queue principale)    │
                    │  celery:email   (queue emails)        │
                    │  celery:heavy   (queue CPU-intensif)  │
                    └──────────────────────────────────────┘
                           ▲                    ▲
                           │                    │
              ┌────────────┴──┐        ┌────────┴───────────┐
              │ Application   │        │   Workers Celery   │
              │ (Flask/Django)│        │  (processus séparés)│
              └───────────────┘        └────────────────────┘
                                                 │
                                    ┌────────────▼──────────┐
                                    │  RESULT BACKEND       │
                                    │  (Redis/PostgreSQL)   │
                                    └───────────────────────┘
```

**Installation :**

```bash
pip install celery redis
# ou avec RabbitMQ
pip install celery "celery[rabbitmq]"

# Pour le monitoring Flower
pip install flower

# Redis comme broker et result backend
docker run -d -p 6379:6379 redis:alpine
```

### 6.2 Configuration de l'Application Celery

```python
# celery_app.py
from celery import Celery
import os

def create_celery_app():
    app = Celery('mon_projet')
    
    app.config_from_object({
        # --- Broker (où les tâches sont envoyées) ---
        'broker_url': os.environ.get('CELERY_BROKER_URL', 'redis://localhost:6379/0'),
        
        # --- Result Backend (où les résultats sont stockés) ---
        'result_backend': os.environ.get('CELERY_RESULT_BACKEND', 'redis://localhost:6379/1'),
        
        # --- Sérialisation ---
        'task_serializer': 'json',
        'result_serializer': 'json',
        'accept_content': ['json'],
        
        # --- Fuseaux horaires ---
        'timezone': 'Europe/Paris',
        'enable_utc': True,
        
        # --- Concurrence ---
        'worker_concurrency': 4,  # 4 processus workers
        'worker_prefetch_multiplier': 1,  # Un seul job à la fois par worker (pour les tâches longues)
        
        # --- Expiration des résultats ---
        'result_expires': 3600,  # 1 heure
        
        # --- Retry par défaut ---
        'task_acks_late': True,        # Acquitter après traitement (at-least-once)
        'task_reject_on_worker_lost': True,  # Remettre en queue si le worker crash
        
        # --- Routage des tâches vers des queues spécifiques ---
        'task_routes': {
            'tasks.send_email': {'queue': 'email'},
            'tasks.process_image': {'queue': 'heavy'},
            'tasks.*': {'queue': 'default'},  # Toutes les autres tâches
        },
        
        # --- Priorités ---
        'task_queue_max_priority': 10,
        'task_default_priority': 5,
    })
    
    return app

celery_app = create_celery_app()
```

### 6.3 Définir des Tâches

```python
# tasks/email_tasks.py
from celery_app import celery_app
import smtplib
from email.mime.text import MIMEText
import logging
import time

logger = logging.getLogger(__name__)

# --- Décorateur @app.task ---
@celery_app.task(
    name='tasks.send_email',       # Nom explicite de la tâche
    bind=True,                      # 'self' = accès à l'instance de la tâche
    max_retries=3,                  # Max 3 tentatives
    default_retry_delay=60,         # 60 secondes entre les tentatives
    queue='email',                  # Queue dédiée aux emails
    rate_limit='100/m',             # Max 100 emails par minute
)
def send_email(self, to: str, subject: str, body: str, from_addr: str = 'noreply@monapp.com'):
    """
    Envoie un email de façon asynchrone.
    
    Args:
        to: Adresse email destinataire
        subject: Sujet de l'email
        body: Corps de l'email (HTML)
        from_addr: Adresse expéditeur
    """
    try:
        # Mise à jour du statut
        self.update_state(state='PROGRESS', meta={'step': 'Connexion SMTP'})
        
        # Logique d'envoi
        msg = MIMEText(body, 'html')
        msg['Subject'] = subject
        msg['From'] = from_addr
        msg['To'] = to
        
        with smtplib.SMTP('localhost', 1025) as server:  # MailHog en dev
            server.send_message(msg)
        
        logger.info(f"Email envoyé à {to} : {subject}")
        return {'success': True, 'to': to, 'subject': subject}
    
    except smtplib.SMTPException as exc:
        # Retry avec backoff exponentiel
        logger.warning(f"Échec envoi email (tentative {self.request.retries + 1}) : {exc}")
        raise self.retry(
            exc=exc,
            countdown=2 ** self.request.retries * 60  # 60s, 120s, 240s
        )
    except Exception as exc:
        # Erreur non récupérable
        logger.error(f"Erreur critique envoi email : {exc}")
        raise  # Ne pas retenter


# --- @shared_task (pour Django, évite les imports circulaires) ---
from celery import shared_task

@shared_task(bind=True, max_retries=5)
def process_image(self, image_path: str, user_id: int):
    """
    Tâche de traitement d'image.
    Utiliser @shared_task dans les projets Django pour éviter
    d'importer l'instance celery directement dans les modules Django.
    """
    try:
        from PIL import Image
        import os
        
        self.update_state(state='PROGRESS', meta={'progress': 0, 'step': 'Chargement'})
        
        with Image.open(image_path) as img:
            # Thumbnail
            thumbnail = img.copy()
            thumbnail.thumbnail((200, 200))
            thumbnail_path = image_path.replace('.', '_thumb.')
            thumbnail.save(thumbnail_path)
            
            self.update_state(state='PROGRESS', meta={'progress': 50, 'step': 'Thumbnail créé'})
            
            # Medium
            medium = img.copy()
            medium.thumbnail((800, 600))
            medium_path = image_path.replace('.', '_medium.')
            medium.save(medium_path)
        
        return {
            'original': image_path,
            'thumbnail': thumbnail_path,
            'medium': medium_path,
            'user_id': user_id,
        }
    
    except FileNotFoundError as exc:
        raise  # Ne pas retenter — le fichier n'existe pas
    except Exception as exc:
        raise self.retry(exc=exc, countdown=30)
```

### 6.4 Lancer des Tâches

```python
# Dans votre application Flask / Django / script

# --- .delay() : raccourci le plus simple ---
result = send_email.delay(
    to='user@example.com',
    subject='Bienvenue !',
    body='<h1>Bienvenue sur notre plateforme</h1>',
)
print(f"Job ID : {result.id}")  # Identifiant du job pour suivre le résultat

# --- .apply_async() : contrôle complet ---
result = send_email.apply_async(
    args=[],                           # Arguments positionnels
    kwargs={                           # Arguments nommés
        'to': 'user@example.com',
        'subject': 'Réinitialisation de mot de passe',
        'body': '<p>Voici votre lien...</p>',
    },
    countdown=300,                     # Délai de 5 minutes
    eta=datetime(2024, 1, 15, 9, 0),  # OU exécuter à une heure précise
    priority=8,                        # Priorité haute (sur 10)
    queue='email',                     # Queue spécifique
    retry=True,
    retry_policy={
        'max_retries': 3,
        'interval_start': 0,
        'interval_step': 0.2,
        'interval_max': 0.5,
    },
)

# --- Récupérer le résultat ---
# ATTENTION : .get() est BLOQUANT — ne jamais appeler dans une requête web !
# Utiliser uniquement dans des scripts, des tâches asynchrones, ou des tests
try:
    result_data = result.get(timeout=10)  # Attendre max 10 secondes
    print("Résultat :", result_data)
except Exception as exc:
    print("Tâche échouée :", exc)

# --- Vérifier le statut sans bloquer ---
print(result.state)   # PENDING, STARTED, PROGRESS, SUCCESS, FAILURE
print(result.ready()) # True si terminé (succès ou échec)
print(result.successful()) # True si SUCCESS

# --- Révoquer (annuler) un job ---
result.revoke(terminate=True)  # Annuler même si en cours
```

### 6.5 Chaining — Workflows Complexes

Celery propose des primitives puissantes pour composer des workflows :

```python
from celery import chain, group, chord

# --- chain : séquence (résultat du premier = argument du suivant) ---
# Exemple : valider l'image → la traiter → envoyer une notification
workflow = chain(
    validate_image.s('uploads/photo.jpg'),   # .s() = signature (args sans dispatch)
    process_image.s(),                         # Reçoit le résultat de validate_image
    notify_user.s(user_id=42),               # Reçoit le résultat de process_image
)
result = workflow.apply_async()

# --- group : parallèle (toutes les tâches en parallèle) ---
# Exemple : envoyer une newsletter à 1000 utilisateurs en parallèle
emails = [
    send_email.s(to=f'user{i}@example.com', subject='Newsletter', body='...')
    for i in range(1000)
]
group_result = group(emails).apply_async()

# Attendre que tous soient finis
group_result.get()  # Liste des résultats

# --- chord : group + callback (parallèle puis agrégation) ---
# Exemple : traiter 5 images en parallèle, puis générer un rapport une fois que toutes sont traitées
image_paths = ['img1.jpg', 'img2.jpg', 'img3.jpg', 'img4.jpg', 'img5.jpg']

chord_result = chord(
    group([process_image.s(path) for path in image_paths]),  # Partie parallèle
    generate_report.s()  # Callback avec la liste de tous les résultats
).apply_async()

# --- Exemple complet : inscription utilisateur ---
def register_user_workflow(user_id: int, email: str):
    """
    Workflow complet après inscription :
    1. Envoyer l'email de bienvenue
    2. Créer le profil par défaut
    Ces deux tâches sont indépendantes → group
    Puis : 3. Déclencher l'onboarding une fois les deux tâches terminées → chord
    """
    return chord(
        group(
            send_email.s(to=email, subject='Bienvenue !', body='...'),
            create_default_profile.s(user_id=user_id),
        ),
        trigger_onboarding.s(user_id=user_id),
    ).apply_async()
```

### 6.6 Celery Beat — Tâches Planifiées

```python
# celery_beat.py (ou dans celery_app.py)
from celery.schedules import crontab

celery_app.conf.beat_schedule = {
    # --- Rapport quotidien à 8h ---
    'daily-report': {
        'task': 'tasks.generate_daily_report',
        'schedule': crontab(hour=8, minute=0),
        'args': (),
        'kwargs': {'report_type': 'daily'},
    },
    
    # --- Nettoyage toutes les heures ---
    'cleanup-temp-files': {
        'task': 'tasks.cleanup_temp_files',
        'schedule': crontab(minute=0),  # Chaque heure à :00
    },
    
    # --- Synchronisation toutes les 30 minutes en semaine ---
    'sync-external-data': {
        'task': 'tasks.sync_data',
        'schedule': crontab(minute='*/30', hour='9-18', day_of_week='mon-fri'),
    },
    
    # --- Simple intervalle : toutes les 60 secondes ---
    'heartbeat': {
        'task': 'tasks.heartbeat',
        'schedule': 60.0,  # en secondes
    },
}
```

**Démarrage des workers et du beat :**

```bash
# Worker classique
celery -A celery_app worker --loglevel=info

# Worker dédié à la queue email
celery -A celery_app worker -Q email --loglevel=info --concurrency=10

# Worker dédié aux tâches lourdes (un seul à la fois)
celery -A celery_app worker -Q heavy --loglevel=info --concurrency=1

# Celery Beat (scheduler de tâches planifiées)
celery -A celery_app beat --loglevel=info

# Les deux ensemble (développement uniquement)
celery -A celery_app worker --beat --loglevel=info
```

### 6.7 Flower — Dashboard Monitoring Python

```bash
# Installer et lancer Flower
pip install flower
celery -A celery_app flower --port=5555

# Avec authentification
celery -A celery_app flower \
  --port=5555 \
  --basic_auth=admin:monmotdepasse \
  --url_prefix=flower
```

**Fonctionnalités de Flower :**
- Dashboard temps réel des workers actifs
- Historique des tâches (succès, échecs, durée)
- Voir les tâches en cours d'exécution
- Révoquer des tâches
- Visualiser les files d'attente
- API REST pour l'intégration

---

## 7. RabbitMQ

### 7.1 Architecture AMQP

RabbitMQ implémente le protocole **AMQP** (Advanced Message Queuing Protocol). Son architecture est plus riche que celle de Redis-based queues.

```
                    ┌─────────────────────────────────────────┐
                    │              RABBITMQ                    │
                    │                                          │
Producer ──▶ [Exchange]                                       │
                    │  ┌──────────────────────────┐           │
                    │  │     Binding (rules)       │           │
                    │  │  routing_key = "email.*"  │           │
                    │  └──────────────────────────┘           │
                    │         │              │                  │
                    │    [Queue A]       [Queue B]             │
                    │         │              │                  │
                    └─────────┼──────────────┼──────────────--┘
                              │              │
                         Consumer 1     Consumer 2
```

**Composants clés :**
- **Producer** : envoie des messages à un exchange (jamais directement à une queue)
- **Exchange** : reçoit les messages et les route vers les queues selon des règles
- **Binding** : règle qui lie un exchange à une queue (avec une routing key optionnelle)
- **Queue** : stocke les messages jusqu'à leur traitement
- **Consumer** : écoute une queue et traite les messages

### 7.2 Types d'Exchanges

**Direct Exchange** — routage exact par routing key :
```
Message avec routing_key="email" → Queue "email-queue"
Message avec routing_key="sms"   → Queue "sms-queue"
```

**Fanout Exchange** — broadcast à toutes les queues liées :
```
Message → Exchange fanout → Queue A, Queue B, Queue C (toutes reçoivent)
```

**Topic Exchange** — routage par pattern :
```
"order.placed.fr"  → match "order.#" et "order.*.fr"
"order.cancelled"  → match "order.#" mais pas "order.*.fr"
```

**Headers Exchange** — routage par attributs du message (rarement utilisé) :
```
Message avec {type: "pdf", priority: "high"} → Queue "pdf-high"
```

### 7.3 Concepts AMQP Critiques

**Message Acknowledgement (ACK/NACK) :**

```
Consumer reçoit message
    │
    ├── Traitement réussi → ACK  → RabbitMQ supprime le message
    │
    ├── Traitement échoué (récupérable) → NACK + requeue=true → Message remis en queue
    │
    └── Traitement échoué (définitif) → NACK + requeue=false → Message envoyé à Dead Letter Queue
```

**Prefetch (QoS) :**
```
Sans prefetch : RabbitMQ envoie 1000 messages à un seul consumer
               → Consumer saturé pendant que les autres sont inactifs

Avec prefetch=1 : RabbitMQ n'envoie un nouveau message que quand le précédent est ACKé
                → Distribution équitable entre consumers
```

> [!warning] Importance de prefetch=1
> Sans configurer le prefetch, un consumer lent peut recevoir des centaines de messages en même temps, causant des timeouts et une distribution inégale. **Toujours configurer `prefetch: 1`** pour les tâches longues.

### 7.4 Installation et Configuration RabbitMQ

```bash
# Via Docker (recommandé)
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=motdepasse \
  rabbitmq:3-management

# Interface de gestion : http://localhost:15672
# AMQP port : 5672
```

```bash
npm install amqplib
```

### 7.5 Work Queue avec amqplib

```javascript
// rabbitmq/connection.js
import amqplib from 'amqplib';

let connection = null;
let channel = null;

export async function getRabbitMQChannel() {
  if (channel) return channel;
  
  connection = await amqplib.connect({
    hostname: process.env.RABBITMQ_HOST || 'localhost',
    port: parseInt(process.env.RABBITMQ_PORT) || 5672,
    username: process.env.RABBITMQ_USER || 'admin',
    password: process.env.RABBITMQ_PASS || 'motdepasse',
    vhost: '/',
    heartbeat: 60, // Ping toutes les 60s pour maintenir la connexion
  });
  
  connection.on('error', (err) => {
    console.error('Erreur connexion RabbitMQ :', err.message);
  });
  
  channel = await connection.createChannel();
  return channel;
}

export async function closeRabbitMQ() {
  if (channel) await channel.close();
  if (connection) await connection.close();
}
```

```javascript
// rabbitmq/producer.js — Producteur Work Queue
import { getRabbitMQChannel } from './connection.js';

const QUEUE_NAME = 'task-queue';

export async function sendTask(taskData) {
  const channel = await getRabbitMQChannel();
  
  // Déclarer la queue (idempotent — pas d'erreur si elle existe déjà)
  await channel.assertQueue(QUEUE_NAME, {
    durable: true, // Survit au redémarrage de RabbitMQ
  });
  
  // Sérialiser les données
  const message = Buffer.from(JSON.stringify(taskData));
  
  // Publier le message
  channel.sendToQueue(QUEUE_NAME, message, {
    persistent: true,  // Message persisté sur disque (survit au redémarrage)
    priority: taskData.priority || 5,
  });
  
  console.log(`[Producteur] Message envoyé :`, taskData);
}

// Utilisation
await sendTask({ type: 'process-order', orderId: 123, userId: 456 });
```

```javascript
// rabbitmq/consumer.js — Consommateur Work Queue
import { getRabbitMQChannel } from './connection.js';

const QUEUE_NAME = 'task-queue';

export async function startConsumer() {
  const channel = await getRabbitMQChannel();
  
  // Déclarer la queue
  await channel.assertQueue(QUEUE_NAME, { durable: true });
  
  // Prefetch : ne pas donner plus d'un message à la fois à ce worker
  await channel.prefetch(1);
  
  console.log(`[Consumer] En attente de messages sur ${QUEUE_NAME}...`);
  
  // Consommer les messages
  channel.consume(QUEUE_NAME, async (message) => {
    if (!message) return;
    
    let taskData;
    try {
      taskData = JSON.parse(message.content.toString());
      console.log('[Consumer] Traitement :', taskData);
      
      // Traitement du job
      await processTask(taskData);
      
      // ACK : traitement réussi, supprimer le message de la queue
      channel.ack(message);
      console.log('[Consumer] Tâche terminée :', taskData);
      
    } catch (error) {
      console.error('[Consumer] Erreur :', error.message);
      
      // NACK avec requeue selon la nature de l'erreur
      const isTransient = error.message.includes('ECONNREFUSED') || 
                          error.message.includes('timeout');
      
      channel.nack(message, false, isTransient); // requeue=true si erreur transitoire
    }
  }, {
    noAck: false, // Activer les ACK manuels (at-least-once delivery)
  });
}

async function processTask(task) {
  switch (task.type) {
    case 'process-order':
      // Logique de traitement de commande
      await new Promise(resolve => setTimeout(resolve, 1000)); // Simulation
      break;
    default:
      throw new Error(`Type de tâche inconnu : ${task.type}`);
  }
}

startConsumer().catch(console.error);
```

### 7.6 Pub/Sub avec RabbitMQ

```javascript
// rabbitmq/publisher.js — Pub/Sub avec Fanout Exchange
import { getRabbitMQChannel } from './connection.js';

const EXCHANGE_NAME = 'events';

export async function publishEvent(event, data) {
  const channel = await getRabbitMQChannel();
  
  // Déclarer l'exchange fanout
  await channel.assertExchange(EXCHANGE_NAME, 'fanout', {
    durable: true,
  });
  
  const message = JSON.stringify({ event, data, timestamp: new Date().toISOString() });
  
  // Publier vers l'exchange (routing key vide pour fanout)
  channel.publish(EXCHANGE_NAME, '', Buffer.from(message), {
    persistent: true,
  });
  
  console.log(`[Publisher] Événement publié : ${event}`);
}

// Utilisation
await publishEvent('user.registered', { userId: 123, email: 'user@example.com' });
```

```javascript
// rabbitmq/subscriber.js — Abonné Pub/Sub
import { getRabbitMQChannel } from './connection.js';

const EXCHANGE_NAME = 'events';

export async function subscribeToEvents(serviceName, handler) {
  const channel = await getRabbitMQChannel();
  
  // Déclarer l'exchange
  await channel.assertExchange(EXCHANGE_NAME, 'fanout', { durable: true });
  
  // Créer une queue exclusive pour ce service
  // ('' = nom généré automatiquement, exclusive = supprimée à la déconnexion)
  const { queue } = await channel.assertQueue('', {
    exclusive: true,  // Queue privée à cette connexion
  });
  
  // Lier la queue à l'exchange
  await channel.bindQueue(queue, EXCHANGE_NAME, '');
  
  console.log(`[${serviceName}] Abonné aux événements sur ${queue}`);
  
  channel.consume(queue, (message) => {
    if (!message) return;
    
    try {
      const data = JSON.parse(message.content.toString());
      handler(data);
      channel.ack(message);
    } catch (error) {
      console.error(`[${serviceName}] Erreur :`, error);
      channel.nack(message, false, false); // Rejeter sans requeue
    }
  }, { noAck: false });
}

// Service email : abonné aux événements utilisateur
await subscribeToEvents('email-service', ({ event, data }) => {
  if (event === 'user.registered') {
    console.log(`Envoi email de bienvenue à ${data.email}`);
  }
});

// Service analytics : abonné aux mêmes événements
await subscribeToEvents('analytics-service', ({ event, data }) => {
  console.log(`Analytics : enregistrement événement ${event}`, data);
});
```

### 7.7 Dead Letter Queue (DLQ)

La DLQ reçoit les messages rejetés (trop de tentatives ou NACK définitif) :

```javascript
// Configurer une queue avec Dead Letter Exchange
await channel.assertExchange('dlx', 'direct', { durable: true }); // Dead Letter Exchange
await channel.assertQueue('dead-letter-queue', { durable: true }); // DLQ
await channel.bindQueue('dead-letter-queue', 'dlx', 'dead');

// Queue principale avec DLX configuré
await channel.assertQueue('main-queue', {
  durable: true,
  arguments: {
    'x-dead-letter-exchange': 'dlx',
    'x-dead-letter-routing-key': 'dead',
    'x-message-ttl': 30000,   // Messages expirés après 30s → DLQ
    'x-max-retries': 3,       // Max 3 tentatives → DLQ
  },
});
```

---

## 8. Redis Streams

### 8.1 Présentation

Les **Redis Streams** sont une structure de données Redis (disponible depuis Redis 5.0) qui combine les avantages d'un log append-only et d'une queue de messages avec consumer groups.

**Avantages des Redis Streams vs Redis Lists (BullMQ) :**
- Persistance et relectabilité : les messages ne sont pas supprimés après consommation
- Consumer groups : plusieurs groupes de consommateurs indépendants
- Message IDs déterministes basés sur le temps
- Parfait pour l'audit et le replay d'événements

```
Redis Stream "events"

ID              Data
─────────────── ─────────────────────────────────────
1640000000000-0 type=order.placed orderId=123 userId=42
1640000001000-0 type=user.registered userId=43 email=...
1640000002000-0 type=order.shipped orderId=123 carrier=DHL
     ↑
     └── Format : <timestamp_ms>-<sequence>
```

### 8.2 Commandes Redis Streams

```bash
# Ajouter un message (XADD)
# * = générer un ID automatique
XADD events * type order.placed orderId 123 userId 42

# Lire les N derniers messages (XREAD)
XREAD COUNT 10 STREAMS events 0-0

# Lire les messages depuis un ID (streaming, bloquant)
XREAD COUNT 10 BLOCK 0 STREAMS events $

# Longueur du stream
XLEN events

# Lire une plage de messages
XRANGE events 1640000000000-0 1640000002000-0

# Trim le stream (garder les 1000 derniers)
XTRIM events MAXLEN ~ 1000
```

**Consumer Groups :**

```bash
# Créer un consumer group ($ = commencer à partir des nouveaux messages)
XGROUP CREATE events email-group $ MKSTREAM

# Créer un deuxième groupe indépendant
XGROUP CREATE events analytics-group $

# Lire les messages pour un consumer (XREADGROUP)
# > = messages non encore assignés à ce consumer
XREADGROUP GROUP email-group worker-1 COUNT 1 BLOCK 0 STREAMS events >

# ACK un message traité
XACK events email-group 1640000001000-0

# Voir les messages en attente (PEL - Pending Entry List)
XPENDING events email-group - + 10
```

### 8.3 Implémentation Node.js avec Redis Streams

```javascript
// redis-streams/producer.js
import { createClient } from 'redis';

const client = createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379',
});

await client.connect();

export async function publishEvent(streamName, event) {
  // XADD : ajouter un événement au stream
  const messageId = await client.xAdd(
    streamName,     // Nom du stream
    '*',            // ID automatique
    {               // Données (clé/valeur plat)
      type: event.type,
      data: JSON.stringify(event.data),
      timestamp: Date.now().toString(),
    },
    {
      TRIM: {
        strategy: 'MAXLEN',  // Limiter la taille du stream
        strategyModifier: '~', // Approximatif (plus performant)
        threshold: 10000,     // Garder ~10 000 messages
      },
    }
  );
  
  console.log(`[Stream] Événement publié, ID : ${messageId}`);
  return messageId;
}

// Exemples
await publishEvent('app-events', {
  type: 'order.placed',
  data: { orderId: 123, userId: 42, amount: 99.90 },
});

await publishEvent('app-events', {
  type: 'user.registered',
  data: { userId: 43, email: 'new@example.com' },
});
```

```javascript
// redis-streams/consumer.js
import { createClient } from 'redis';

const client = createClient({ url: process.env.REDIS_URL || 'redis://localhost:6379' });
await client.connect();

const STREAM = 'app-events';
const GROUP = 'email-service';
const CONSUMER = `worker-${process.pid}`; // ID unique par processus

// Créer le consumer group s'il n'existe pas
async function ensureGroup() {
  try {
    await client.xGroupCreate(STREAM, GROUP, '$', { MKSTREAM: true });
    console.log(`[Stream] Groupe ${GROUP} créé`);
  } catch (err) {
    if (!err.message.includes('BUSYGROUP')) throw err;
    console.log(`[Stream] Groupe ${GROUP} existant, reconnexion`);
  }
}

async function processMessage(messageId, fields) {
  const eventType = fields.type;
  const eventData = JSON.parse(fields.data);
  
  console.log(`[${CONSUMER}] Traitement ${eventType} (ID: ${messageId})`);
  
  switch (eventType) {
    case 'order.placed':
      // Envoyer email de confirmation
      console.log(`Email confirmation commande ${eventData.orderId}`);
      break;
    case 'user.registered':
      // Envoyer email de bienvenue
      console.log(`Email bienvenue à ${eventData.email}`);
      break;
    default:
      console.log(`Événement ignoré : ${eventType}`);
  }
}

async function consumeMessages() {
  await ensureGroup();
  
  console.log(`[Stream] Consumer ${CONSUMER} prêt sur ${STREAM}/${GROUP}`);
  
  // Boucle de consommation
  while (true) {
    try {
      // XREADGROUP : lire les nouveaux messages (> = non assignés)
      const messages = await client.xReadGroup(
        GROUP,
        CONSUMER,
        [{ key: STREAM, id: '>' }],
        { COUNT: 10, BLOCK: 2000 } // Bloquer 2s si pas de messages
      );
      
      if (!messages) continue; // Timeout, re-boucler
      
      for (const { name, messages: streamMessages } of messages) {
        for (const { id, message } of streamMessages) {
          try {
            await processMessage(id, message);
            
            // ACK : message traité avec succès
            await client.xAck(STREAM, GROUP, id);
            
          } catch (error) {
            console.error(`[Stream] Erreur traitement ${id} :`, error.message);
            // Pas d'ACK → message reste en "pending"
            // Sera récupéré par le mécanisme de réclamation (voir ci-dessous)
          }
        }
      }
      
    } catch (error) {
      console.error('[Stream] Erreur de lecture :', error.message);
      await new Promise(resolve => setTimeout(resolve, 1000)); // Attendre avant retry
    }
  }
}

// Réclamation des messages en attente (messages orphelins)
async function reclaimPendingMessages() {
  const IDLE_TIMEOUT = 30000; // 30 secondes sans ACK → reclaim
  
  const pending = await client.xPending(
    STREAM,
    GROUP,
    { start: '-', end: '+', count: 100 }
  );
  
  for (const entry of pending) {
    if (entry.millisecondsSinceDelivery > IDLE_TIMEOUT) {
      // Récupérer le message pour le retraiter
      const claimed = await client.xAutoClaim(
        STREAM,
        GROUP,
        CONSUMER,
        IDLE_TIMEOUT,
        entry.id
      );
      
      for (const { id, message } of claimed.messages) {
        console.log(`[Stream] Récupération message orphelin ${id}`);
        try {
          await processMessage(id, message);
          await client.xAck(STREAM, GROUP, id);
        } catch (error) {
          console.error(`[Stream] Échec récupération ${id} :`, error);
        }
      }
    }
  }
}

// Lancer la consommation
consumeMessages().catch(console.error);

// Récupérer les orphelins toutes les 60 secondes
setInterval(reclaimPendingMessages, 60000);
```

### 8.4 Redis Streams vs BullMQ

| Critère | Redis Streams | BullMQ |
|---|---|---|
| Relectabilité | Oui (messages conservés) | Non (supprimés après traitement) |
| Consumer groups | Oui (natif) | Oui (via workers) |
| Monitoring | Via XPENDING, XINFO | Bull Board |
| Priorités | Non | Oui |
| Delay/Cron | Non | Oui |
| Retry automatique | Manuel | Automatique |
| Courbe d'apprentissage | Moyenne | Faible |
| Cas d'usage | Event sourcing, audit | Jobs asynchrones |

> [!info] Quand utiliser Redis Streams vs BullMQ ?
> - **BullMQ** : jobs à traiter une fois, retry automatique, cron, monitoring visuel. Idéal pour les tâches métier (emails, images, webhooks).
> - **Redis Streams** : événements à rejouer, audit trail, plusieurs services indépendants consommant le même flux. Idéal pour l'event sourcing et la communication entre microservices.

---

## 9. Cas Pratique E2E — File d'Attente d'Emails avec BullMQ

### 9.1 Architecture Complète

```
                    ┌─────────────────┐
    POST /register  │   API Express   │
───────────────────▶│                 │
                    │  1. Créer user  │
                    │  2. Queue email │◀──────┐
                    │  3. Réponse 201 │       │
                    └─────────────────┘       │
                                              │
                    ┌─────────────────┐       │
                    │     REDIS       │       │
                    │  email:queue    │───────┘
                    │  [job-001]      │
                    │  [job-002]      │
                    └─────────────────┘
                              │
                    ┌─────────▼───────┐
                    │  Email Worker   │
                    │  (Node.js)      │
                    │                 │
                    │  1. Prendre job │
                    │  2. Envoyer via │
                    │     Nodemailer  │
                    │  3. ACK Redis   │
                    └─────────────────┘
```

### 9.2 Structure du Projet

```
email-queue-demo/
├── package.json
├── .env
├── src/
│   ├── server.js          # Serveur Express
│   ├── queues/
│   │   └── emailQueue.js  # Définition des queues
│   ├── workers/
│   │   └── emailWorker.js # Worker de traitement
│   ├── routes/
│   │   └── auth.js        # Routes d'authentification
│   └── services/
│       └── emailService.js # Logique d'envoi d'email
├── Dockerfile
└── docker-compose.yml
```

### 9.3 Code Complet

```javascript
// src/queues/emailQueue.js
import { Queue } from 'bullmq';

const connection = {
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT) || 6379,
};

export const emailQueue = new Queue('emails', {
  connection,
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 2000 },
    removeOnComplete: { age: 3600, count: 500 },
    removeOnFail: { age: 86400 },
  },
});

// Raccourcis pour les types d'emails courants
export const EmailQueue = {
  async sendWelcome(user) {
    return emailQueue.add('welcome', { user }, { priority: 5 });
  },
  
  async sendPasswordReset(user, token) {
    return emailQueue.add('password-reset', { user, token }, {
      priority: 1,     // Haute priorité
      attempts: 5,     // Plus de tentatives pour les emails importants
      delay: 0,        // Immédiat
    });
  },
  
  async sendNewsletter(users, campaignId) {
    const jobs = users.map(user => ({
      name: 'newsletter',
      data: { user, campaignId },
      opts: { priority: 10, delay: Math.random() * 5000 }, // Étaler sur 5 secondes
    }));
    return emailQueue.addBulk(jobs);
  },
};
```

```javascript
// src/services/emailService.js
import nodemailer from 'nodemailer';

// En développement : MailHog (npm run mailhog ou docker)
// En production : SendGrid, Mailgun, SES, etc.
const createTransport = () => {
  if (process.env.NODE_ENV === 'production') {
    return nodemailer.createTransport({
      host: process.env.SMTP_HOST,
      port: 587,
      secure: false,
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS,
      },
    });
  }
  // MailHog pour le développement
  return nodemailer.createTransport({
    host: 'localhost',
    port: 1025,
    ignoreTLS: true,
  });
};

const transporter = createTransport();

export async function sendEmail({ to, subject, html, from = 'app@example.com' }) {
  const info = await transporter.sendMail({ from, to, subject, html });
  return { messageId: info.messageId, accepted: info.accepted };
}

// Templates d'emails
export const templates = {
  welcome: ({ name, verificationUrl }) => ({
    subject: `Bienvenue, ${name} !`,
    html: `
      <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
        <h1>Bienvenue sur notre plateforme, ${name} !</h1>
        <p>Votre compte a été créé avec succès.</p>
        <p>
          <a href="${verificationUrl}" 
             style="background: #007bff; color: white; padding: 12px 24px; 
                    text-decoration: none; border-radius: 4px;">
            Vérifier mon email
          </a>
        </p>
      </div>
    `,
  }),
  
  passwordReset: ({ name, resetUrl, expiresIn = '1 heure' }) => ({
    subject: 'Réinitialisation de votre mot de passe',
    html: `
      <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
        <h2>Réinitialisation de mot de passe</h2>
        <p>Bonjour ${name},</p>
        <p>Une demande de réinitialisation de mot de passe a été effectuée pour votre compte.</p>
        <p>
          <a href="${resetUrl}"
             style="background: #dc3545; color: white; padding: 12px 24px;
                    text-decoration: none; border-radius: 4px;">
            Réinitialiser mon mot de passe
          </a>
        </p>
        <p><small>Ce lien expire dans ${expiresIn}. Si vous n'êtes pas à l'origine de cette demande, ignorez cet email.</small></p>
      </div>
    `,
  }),
};
```

```javascript
// src/workers/emailWorker.js
import { Worker } from 'bullmq';
import { sendEmail, templates } from '../services/emailService.js';

const connection = {
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT) || 6379,
};

async function processJob(job) {
  const { name, data } = job;
  
  switch (name) {
    case 'welcome': {
      const { user } = data;
      const verificationUrl = `${process.env.APP_URL}/verify?token=${user.verificationToken}`;
      const { subject, html } = templates.welcome({ name: user.name, verificationUrl });
      
      await job.updateProgress(50);
      const result = await sendEmail({ to: user.email, subject, html });
      await job.updateProgress(100);
      
      return result;
    }
    
    case 'password-reset': {
      const { user, token } = data;
      const resetUrl = `${process.env.APP_URL}/reset-password?token=${token}`;
      const { subject, html } = templates.passwordReset({ name: user.name, resetUrl });
      
      await job.updateProgress(50);
      const result = await sendEmail({ to: user.email, subject, html });
      await job.updateProgress(100);
      
      return result;
    }
    
    case 'newsletter': {
      const { user, campaignId } = data;
      // Récupérer le contenu de la newsletter depuis la BDD ou un service
      const { subject, html } = await getNewsletterContent(campaignId, user);
      
      await job.updateProgress(50);
      const result = await sendEmail({ to: user.email, subject, html });
      await job.updateProgress(100);
      
      return result;
    }
    
    default:
      throw new Error(`Type d'email inconnu : ${name}`);
  }
}

async function getNewsletterContent(campaignId, user) {
  // En production : récupérer depuis la BDD
  return {
    subject: `Notre newsletter #${campaignId}`,
    html: `<p>Bonjour ${user.name}, voici notre newsletter...</p>`,
  };
}

// Démarrage du worker
const worker = new Worker('emails', processJob, {
  connection,
  concurrency: 10, // 10 emails en parallèle
  limiter: {
    max: 200,         // Max 200 emails
    duration: 60000,  // par minute (respecter les limites SMTP)
  },
});

worker.on('completed', (job, result) => {
  console.log(`✅ Email ${job.name} envoyé (job ${job.id}) :`, result?.messageId);
});

worker.on('failed', (job, error) => {
  console.error(`❌ Échec email ${job?.name} (job ${job?.id}) :`, error.message);
  // En production : alerter via Sentry, PagerDuty, etc.
  if (job?.attemptsMade >= job?.opts?.attempts) {
    console.error(`⚠️  Job ${job.id} a épuisé toutes ses tentatives → Dead Letter`);
    // Notifier l'équipe / stocker en BDD pour investigation
  }
});

worker.on('error', console.error);

process.on('SIGTERM', () => worker.close());

console.log('Worker email démarré.');
```

```javascript
// src/server.js
import express from 'express';
import { createBullBoard } from '@bull-board/api';
import { BullMQAdapter } from '@bull-board/api/bullMQAdapter.js';
import { ExpressAdapter } from '@bull-board/express';
import { emailQueue, EmailQueue } from './queues/emailQueue.js';
import crypto from 'crypto';

const app = express();
app.use(express.json());

// ── Bull Board ──
const serverAdapter = new ExpressAdapter();
serverAdapter.setBasePath('/admin/queues');
createBullBoard({ queues: [new BullMQAdapter(emailQueue)], serverAdapter });
app.use('/admin/queues', serverAdapter.getRouter());

// ── Routes ──
app.post('/register', async (req, res) => {
  const { name, email, password } = req.body;
  
  if (!name || !email || !password) {
    return res.status(400).json({ error: 'Champs requis : name, email, password' });
  }
  
  try {
    // Simuler la création en BDD
    const user = {
      id: crypto.randomUUID(),
      name,
      email,
      verificationToken: crypto.randomBytes(32).toString('hex'),
    };
    
    // Ajouter le job d'email de bienvenue (NON-BLOQUANT)
    const job = await EmailQueue.sendWelcome(user);
    
    res.status(201).json({
      message: 'Compte créé. Un email de vérification a été envoyé.',
      userId: user.id,
      emailJobId: job.id, // Permet au client de vérifier le statut
    });
    
  } catch (error) {
    console.error('Erreur inscription :', error);
    res.status(500).json({ error: 'Erreur interne' });
  }
});

app.post('/forgot-password', async (req, res) => {
  const { email } = req.body;
  
  // Simuler la recherche en BDD
  const user = { id: '123', name: 'Alice', email };
  const resetToken = crypto.randomBytes(32).toString('hex');
  
  // Sauvegarder le token en BDD (non simulé ici)
  
  await EmailQueue.sendPasswordReset(user, resetToken);
  
  // IMPORTANT : toujours répondre de la même façon (ne pas divulguer si l'email existe)
  res.json({ message: 'Si cet email existe, un lien de réinitialisation a été envoyé.' });
});

// Endpoint de statut des jobs
app.get('/jobs/:jobId', async (req, res) => {
  const job = await emailQueue.getJob(req.params.jobId);
  if (!job) return res.status(404).json({ error: 'Job introuvable' });
  
  res.json({
    id: job.id,
    name: job.name,
    state: await job.getState(),
    progress: job.progress,
    attempts: job.attemptsMade,
    result: job.returnvalue,
    error: job.failedReason,
    createdAt: new Date(job.timestamp).toISOString(),
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Serveur démarré sur http://localhost:${PORT}`);
  console.log(`Bull Board : http://localhost:${PORT}/admin/queues`);
});
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"   # SMTP
      - "8025:8025"   # Interface web

  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - SMTP_HOST=mailhog
      - SMTP_PORT=1025
      - APP_URL=http://localhost:3000
      - NODE_ENV=development
    depends_on:
      - redis
      - mailhog

  worker:
    build: .
    command: node src/workers/emailWorker.js
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - SMTP_HOST=mailhog
      - SMTP_PORT=1025
      - NODE_ENV=development
    depends_on:
      - redis
      - mailhog
    deploy:
      replicas: 2  # 2 instances du worker

volumes:
  redis_data:
```

```json
// package.json
{
  "name": "email-queue-demo",
  "type": "module",
  "scripts": {
    "start": "node src/server.js",
    "worker": "node src/workers/emailWorker.js",
    "dev": "nodemon src/server.js",
    "docker": "docker-compose up --build"
  },
  "dependencies": {
    "bullmq": "^5.0.0",
    "@bull-board/api": "^5.0.0",
    "@bull-board/express": "^5.0.0",
    "express": "^4.18.0",
    "nodemailer": "^6.9.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
```

---

## 10. Bonnes Pratiques et Pièges Courants

### 10.1 Idempotence — Règle Absolue

Tout job **doit être idempotent**. Si votre worker crashe après le traitement mais avant l'ACK, le job sera relancé. Si votre traitement n'est pas idempotent, vous enverrez deux emails, débiterez deux fois, etc.

```javascript
// ❌ Non idempotent — charge la BDD deux fois si le job crashe entre les deux
async function processOrder(job) {
  await db.createInvoice(job.data.orderId);  // Crée la facture
  await emailQueue.add('send-invoice', { orderId: job.data.orderId }); // Peut être ajouté deux fois
}

// ✅ Idempotent — vérification avant action
async function processOrder(job) {
  const { orderId } = job.data;
  
  // Vérifier si la facture existe déjà
  const existing = await db.findInvoice({ orderId });
  if (existing) {
    console.log(`Facture ${orderId} déjà créée, skip`);
    return { skipped: true, invoiceId: existing.id };
  }
  
  // Créer et marquer comme fait dans une transaction
  const invoice = await db.transaction(async (trx) => {
    const inv = await trx.createInvoice(orderId);
    await trx.markOrderProcessed(orderId);
    return inv;
  });
  
  return { invoiceId: invoice.id };
}
```

### 10.2 Ne Jamais Bloquer sur `.get()` dans une Requête Web

```javascript
// ❌ CATASTROPHIQUE — bloque le thread Node.js jusqu'à la fin du job
app.post('/send', async (req, res) => {
  const job = await emailQueue.add('send', req.body);
  const result = await job.waitUntilFinished(queueEvents); // BLOQUANT
  res.json(result);
});

// ✅ Pattern correct — répondre immédiatement, le client peut vérifier le statut
app.post('/send', async (req, res) => {
  const job = await emailQueue.add('send', req.body);
  res.status(202).json({
    message: 'Traitement en cours',
    jobId: job.id,
    statusUrl: `/jobs/${job.id}`,
  });
});
```

### 10.3 Séparer les Queues par Priorité/Type

```javascript
// ❌ Une seule queue mélange des jobs urgents et lents
await queue.add('weekly-report', reportData);     // 30 minutes
await queue.add('password-reset', userData);       // Urgent, 2 secondes

// La réinitialisation de mot de passe attend derrière le rapport

// ✅ Queues séparées
const urgentQueue = new Queue('urgent');           // Workers dédiés, priorité max
const backgroundQueue = new Queue('background');  // Workers séparés, moins nombreux

await urgentQueue.add('password-reset', userData);
await backgroundQueue.add('weekly-report', reportData);
```

### 10.4 Gérer la Backpressure

```javascript
// Eviter de saturer la queue avec des millions de jobs d'un coup
// ✅ Traitement par batches avec pause

async function scheduleNewsletterInBatches(userIds, campaignId) {
  const BATCH_SIZE = 100;
  const DELAY_BETWEEN_BATCHES = 1000; // 1 seconde entre chaque batch
  
  for (let i = 0; i < userIds.length; i += BATCH_SIZE) {
    const batch = userIds.slice(i, i + BATCH_SIZE);
    
    await emailQueue.addBulk(
      batch.map(userId => ({
        name: 'newsletter',
        data: { userId, campaignId },
        opts: { delay: Math.floor(i / BATCH_SIZE) * DELAY_BETWEEN_BATCHES },
      }))
    );
    
    // Pause courte pour ne pas saturer Redis
    await new Promise(resolve => setTimeout(resolve, 100));
    
    console.log(`Batch ${Math.floor(i / BATCH_SIZE) + 1} planifié`);
  }
}
```

### 10.5 Logging et Observabilité

```javascript
// Intégrer les événements de queue avec votre système de logging
import { QueueEvents } from 'bullmq';

const events = new QueueEvents('emails', { connection });

events.on('failed', ({ jobId, failedReason, prev }) => {
  // Envoyer à Sentry
  Sentry.captureException(new Error(failedReason), {
    extra: { jobId, previousState: prev },
    tags: { queue: 'emails' },
  });
  
  // Métrique Prometheus/Datadog
  metrics.increment('queue.job.failed', { queue: 'emails' });
});

events.on('completed', ({ jobId, returnvalue }) => {
  metrics.increment('queue.job.completed', { queue: 'emails' });
  metrics.timing('queue.job.duration', Date.now() - returnvalue?.startedAt);
});
```

### 10.6 Tableau Récapitulatif des Outils

| Outil | Langage | Broker | Points forts | Limites |
|---|---|---|---|---|
| **BullMQ** | Node.js | Redis | Simple, Bull Board, cron | Pas de fan-out natif |
| **Celery** | Python | Redis/RabbitMQ | Workflows complexes, Django | Configuration verbieuse |
| **amqplib** | Node.js | RabbitMQ | AMQP complet, fan-out, DLQ | Plus complexe |
| **Redis Streams** | Node.js/Python | Redis | Event sourcing, replay | Monitoring rudimentaire |
| **Kafka** | Tous | Kafka | Très haute performance | Infrastructure lourde |
| **SQS (AWS)** | Tous | AWS | Managé, auto-scale | Payant, vendor lock-in |

---

## 11. Exercices Pratiques

### Exercice 1 — File d'Attente d'Emails (Débutant)

Mettez en place une API d'inscription utilisateur avec envoi d'email différé :

1. `POST /register` crée l'utilisateur en base et ajoute un job `welcome-email` dans BullMQ
2. Le worker prend le job et simule l'envoi (simple `console.log`)
3. `GET /jobs/:id` retourne l'état du job
4. Tester avec 3 workers lancés en parallèle

**Bonus :** ajouter Bull Board et observer la distribution des jobs.

### Exercice 2 — Retry et Dead Letter (Intermédiaire)

Créez un worker qui échoue aléatoirement (30% de chance d'erreur) :

1. Configurer 3 tentatives avec backoff exponentiel
2. Après 3 échecs, déplacer le job vers une queue `dead-letter`
3. Ajouter un endpoint `POST /dead-letter/:id/retry` pour relancer manuellement
4. Observer dans Bull Board les jobs en failed

### Exercice 3 — Pub/Sub avec RabbitMQ (Intermédiaire)

Simuler un système d'événements e-commerce :

1. Créer un exchange `orders` de type `topic`
2. Queue `email-service` : abonnée à `order.#`
3. Queue `analytics-service` : abonnée à `order.placed`
4. Queue `stock-service` : abonnée à `order.placed` et `order.cancelled`
5. Publier des événements et vérifier que chaque service reçoit les bons messages

### Exercice 4 — Workflow Celery (Avancé)

Implémenter un pipeline de traitement de données avec Celery :

1. Tâche `fetch_data(source_url)` : télécharger un CSV
2. Tâche `validate_data(csv_content)` : valider les données
3. Tâche `import_records(validated_data)` : importer en BDD
4. Tâche `send_report(import_result)` : envoyer un rapport par email

Enchaîner ces 4 tâches avec `chain()`. Ajouter une gestion d'erreur : si `validate_data` échoue, appeler `notify_error()` au lieu de continuer.

### Exercice 5 — Benchmark de Charge (Avancé)

Mesurer les performances de votre queue :

1. Ajouter 10 000 jobs d'un coup
2. Mesurer :
   - Temps pour vider la queue avec 1 worker
   - Temps pour vider la queue avec 5 workers
   - Taux de traitement (jobs/seconde)
3. Trouver le nombre optimal de workers pour maximiser le débit sans saturer Redis

> [!tip] Challenge supplémentaire
> Implémenter un système de **circuit breaker** : si plus de 50% des jobs échouent en 5 minutes, mettre la queue en pause automatiquement et envoyer une alerte Slack.

---

## 12. Récapitulatif

```
┌─────────────────────────────────────────────────────────────────┐
│                    MESSAGE QUEUES — RÉSUMÉ                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  POURQUOI ?                                                       │
│  • Découplage producteur/consommateur                            │
│  • Absorption des pics de charge                                 │
│  • Retry automatique                                             │
│  • Scalabilité indépendante                                      │
│                                                                   │
│  PATTERNS                                                         │
│  Work Queue  → 1 job = 1 worker (compétition)                   │
│  Pub/Sub     → 1 message = N abonnés (broadcast)                │
│  Fan-out     → 1 exchange = N queues                            │
│  Topic       → routage par pattern (*.fr, order.#)              │
│                                                                   │
│  GARANTIES                                                        │
│  At-most-once  → rapide, peut perdre                            │
│  At-least-once → safe, peut dupliquer (idempotence requise)     │
│  Exactly-once  → complexe, lent                                  │
│                                                                   │
│  OUTILS                                                           │
│  BullMQ   (Node.js + Redis)  → jobs asynchrones web             │
│  Celery   (Python)           → workflows complexes              │
│  RabbitMQ (tous)             → AMQP complet, entreprise         │
│  Redis Streams               → event sourcing, replay           │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

> [!info] Lien avec les autres cours
> - **Node.js Avancé** : event loop, streams, workers threads — le contexte d'exécution des workers BullMQ
> - **Microservices et Patterns** : les queues sont la colonne vertébrale de la communication asynchrone entre services
> - **Redis** : structure de données utilisée par BullMQ et Redis Streams
> - **Docker et DevOps** : déploiement multi-workers avec docker-compose `replicas`
