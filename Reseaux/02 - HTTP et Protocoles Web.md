# 02 - HTTP et Protocoles Web

## Qu'est-ce que HTTP ?

**HTTP** (HyperText Transfer Protocol) est le protocole de la couche Application qui permet la communication entre un **client** (navigateur, application mobile, script curl) et un **serveur** web. Il fonctionne selon un modele **requete-reponse** : le client envoie une requete, le serveur renvoie une reponse.

> [!info] HTTP en bref
> - Protocole de la couche 7 (Application) du modele OSI
> - Fonctionne au-dessus de **TCP** (HTTP/1.1 et HTTP/2) ou **QUIC/UDP** (HTTP/3)
> - **Sans etat** (stateless) : chaque requete est independante, le serveur ne "se souvient" pas des requetes precedentes
> - Port par defaut : **80** (HTTP) ou **443** (HTTPS)

> [!tip] Analogie
> HTTP fonctionne comme un **guichet de restaurant**. Le client (vous) passe une commande precise (requete) au comptoir, le cuisinier (serveur) prepare le plat et vous le renvoie (reponse). Le serveur ne se souvient pas de vous entre deux commandes, sauf si vous presentez une carte de fidelite (cookie/token).

---

## Anatomie d'un message HTTP

### Requete HTTP

```
┌─────────────────────────────────────────────────────────────┐
│  REQUETE HTTP                                               │
├─────────────────────────────────────────────────────────────┤
│  Ligne de requete :                                         │
│  GET /api/users?page=2 HTTP/1.1                             │
│  └┬┘ └──────┬────────┘ └──┬───┘                             │
│  Methode    URI          Version                            │
├─────────────────────────────────────────────────────────────┤
│  En-tetes (headers) :                                       │
│  Host: api.example.com                                      │
│  Accept: application/json                                   │
│  Authorization: Bearer eyJhbGci...                          │
│  User-Agent: Mozilla/5.0                                    │
│  Accept-Language: fr-FR                                     │
├─────────────────────────────────────────────────────────────┤
│  (ligne vide separant headers et body)                      │
├─────────────────────────────────────────────────────────────┤
│  Corps (body) :                                             │
│  (vide pour GET, contient des donnees pour POST/PUT)        │
└─────────────────────────────────────────────────────────────┘
```

### Reponse HTTP

```
┌─────────────────────────────────────────────────────────────┐
│  REPONSE HTTP                                               │
├─────────────────────────────────────────────────────────────┤
│  Ligne de statut :                                          │
│  HTTP/1.1 200 OK                                            │
│  └──┬───┘ └┬┘ └┬┘                                           │
│  Version  Code Message                                      │
├─────────────────────────────────────────────────────────────┤
│  En-tetes (headers) :                                       │
│  Content-Type: application/json; charset=utf-8              │
│  Content-Length: 256                                         │
│  Cache-Control: max-age=3600                                │
│  Set-Cookie: session=abc123; HttpOnly; Secure               │
├─────────────────────────────────────────────────────────────┤
│  (ligne vide)                                               │
├─────────────────────────────────────────────────────────────┤
│  Corps (body) :                                             │
│  {"users": [{"id": 1, "name": "Alice"}, ...]}              │
└─────────────────────────────────────────────────────────────┘
```

---

## Les methodes HTTP

Chaque requete HTTP utilise une **methode** (ou verbe) qui indique l'**action** souhaitee sur la ressource :

| Methode   | Action                           | Corps requis | Idempotent | Safe |
| --------- | -------------------------------- | ------------ | ---------- | ---- |
| `GET`     | Lire / recuperer une ressource   | Non          | Oui        | Oui  |
| `POST`    | Creer une nouvelle ressource     | Oui          | Non        | Non  |
| `PUT`     | Remplacer entierement une ressource | Oui       | Oui        | Non  |
| `PATCH`   | Modifier partiellement une ressource | Oui      | Non*       | Non  |
| `DELETE`  | Supprimer une ressource          | Optionnel    | Oui        | Non  |
| `HEAD`    | Comme GET mais sans le body      | Non          | Oui        | Oui  |
| `OPTIONS` | Decouvrir les methodes autorisees| Non          | Oui        | Oui  |

> [!info] Idempotent vs Safe
> - **Idempotent** : appeler N fois produit le meme resultat qu'appeler 1 fois. `DELETE /users/42` appellee 10 fois donne le meme resultat : l'utilisateur 42 est supprime.
> - **Safe** (sure) : ne modifie pas l'etat du serveur. `GET` ne doit jamais modifier de donnees.
> - `PATCH` peut etre rendue idempotente selon l'implementation, mais ce n'est pas garanti par la spec.

### Exemples concrets avec curl

```bash
# GET : recuperer une liste
curl https://api.example.com/users

# GET : recuperer un element specifique
curl https://api.example.com/users/42

# POST : creer un utilisateur
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'

# PUT : remplacer un utilisateur
curl -X PUT https://api.example.com/users/42 \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice B.", "email": "alice.b@example.com"}'

# PATCH : modifier partiellement
curl -X PATCH https://api.example.com/users/42 \
  -H "Content-Type: application/json" \
  -d '{"email": "nouveau@example.com"}'

# DELETE : supprimer
curl -X DELETE https://api.example.com/users/42

# HEAD : verifier sans telecharger le body
curl -I https://example.com

# OPTIONS : decouvrir les methodes autorisees (preflight CORS)
curl -X OPTIONS https://api.example.com/users \
  -H "Origin: https://frontend.com"
```

---

## Codes de statut HTTP

Le code de statut indique le **resultat** de la requete. Il est compose de 3 chiffres, classes en 5 categories :

### 1xx : Information

| Code | Signification                          |
| ---- | -------------------------------------- |
| 100  | Continue (le serveur attend le body)   |
| 101  | Switching Protocols (ex: WebSocket)    |

### 2xx : Succes

| Code | Signification                                          |
| ---- | ------------------------------------------------------ |
| 200  | **OK** : requete reussie                               |
| 201  | **Created** : ressource creee (reponse a POST)         |
| 204  | **No Content** : succes mais pas de body (souvent DELETE) |

### 3xx : Redirection

| Code | Signification                                          |
| ---- | ------------------------------------------------------ |
| 301  | **Moved Permanently** : la ressource a change d'URL definitivement |
| 302  | **Found** : redirection temporaire                     |
| 304  | **Not Modified** : utiliser la version en cache        |
| 307  | **Temporary Redirect** : comme 302 mais conserve la methode |
| 308  | **Permanent Redirect** : comme 301 mais conserve la methode |

### 4xx : Erreur client

| Code | Signification                                          |
| ---- | ------------------------------------------------------ |
| 400  | **Bad Request** : requete malformee                    |
| 401  | **Unauthorized** : authentification requise            |
| 403  | **Forbidden** : authentifie mais pas autorise          |
| 404  | **Not Found** : ressource introuvable                  |
| 405  | **Method Not Allowed** : methode non supportee         |
| 409  | **Conflict** : conflit avec l'etat actuel              |
| 413  | **Payload Too Large** : body trop volumineux           |
| 422  | **Unprocessable Entity** : donnees invalides (validation) |
| 429  | **Too Many Requests** : rate limiting                  |

### 5xx : Erreur serveur

| Code | Signification                                          |
| ---- | ------------------------------------------------------ |
| 500  | **Internal Server Error** : erreur generique du serveur|
| 502  | **Bad Gateway** : le proxy/reverse proxy a recu une reponse invalide |
| 503  | **Service Unavailable** : serveur surcharge ou en maintenance |
| 504  | **Gateway Timeout** : le proxy n'a pas recu de reponse a temps |

> [!tip] Astuce pour s'en souvenir
> - **2xx** : tout va bien
> - **3xx** : va voir ailleurs
> - **4xx** : c'est la faute du client
> - **5xx** : c'est la faute du serveur

> [!warning] 401 vs 403
> - **401 Unauthorized** : le client n'est **pas authentifie** (pas de token, token expire). Le nom est trompeur, ca devrait s'appeler "Unauthenticated".
> - **403 Forbidden** : le client est **authentifie** mais n'a pas les **permissions** pour acceder a cette ressource.

---

## Les en-tetes HTTP (Headers)

Les headers transportent des **metadonnees** sur la requete ou la reponse. Ils sont au format `Cle: Valeur`.

### Headers de requete importants

| Header            | Role                                          | Exemple                                  |
| ----------------- | --------------------------------------------- | ---------------------------------------- |
| `Host`            | Nom du serveur cible                          | `Host: api.example.com`                  |
| `Accept`          | Types de contenu acceptes par le client        | `Accept: application/json`               |
| `Content-Type`    | Type du body envoye                           | `Content-Type: application/json`         |
| `Authorization`   | Credentials d'authentification                | `Authorization: Bearer eyJ...`           |
| `User-Agent`      | Identification du client                      | `User-Agent: Mozilla/5.0 ...`            |
| `Accept-Language` | Langues preferees                             | `Accept-Language: fr-FR, en`             |
| `Cookie`          | Cookies envoyes au serveur                    | `Cookie: session=abc123`                 |
| `If-None-Match`   | Cache conditionnel (ETag)                     | `If-None-Match: "etag123"`               |

### Headers de reponse importants

| Header              | Role                                        | Exemple                                  |
| ------------------- | ------------------------------------------- | ---------------------------------------- |
| `Content-Type`      | Type du body retourne                       | `Content-Type: text/html; charset=utf-8` |
| `Content-Length`    | Taille du body en octets                     | `Content-Length: 1024`                   |
| `Cache-Control`     | Politique de mise en cache                   | `Cache-Control: max-age=3600`            |
| `Set-Cookie`        | Definir un cookie chez le client             | `Set-Cookie: id=abc; HttpOnly`           |
| `Location`          | URL de redirection (avec 3xx)                | `Location: /new-page`                    |
| `ETag`              | Identifiant de version de la ressource       | `ETag: "33a64df551425fcc55e"`            |

### Headers CORS (Cross-Origin Resource Sharing)

Quand un frontend sur `https://app.com` appelle une API sur `https://api.com`, le navigateur bloque la requete par defaut (politique **same-origin**). Les headers CORS permettent au serveur d'autoriser ces requetes cross-origin :

```
# Le serveur repond avec ces headers :
Access-Control-Allow-Origin: https://app.com    (ou * pour tout le monde)
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

> [!warning] CORS est une protection du navigateur
> CORS ne protege **que** les navigateurs web. Un script Python, curl ou Postman ignorent totalement CORS. C'est pourquoi CORS ne remplace pas l'authentification et l'autorisation cote serveur.

---

## HTTP/1.1 vs HTTP/2 vs HTTP/3

### Evolution du protocole HTTP

```
1991   HTTP/0.9  →  Texte brut, GET uniquement
1996   HTTP/1.0  →  Headers, POST, codes de statut
1997   HTTP/1.1  →  Keep-alive, chunked, Host header
2015   HTTP/2    →  Multiplexage, compression headers, push serveur
2022   HTTP/3    →  QUIC (UDP), 0-RTT, plus de head-of-line blocking
```

### Comparaison detaillee

| Caracteristique         | HTTP/1.1               | HTTP/2                  | HTTP/3                  |
| ----------------------- | ---------------------- | ----------------------- | ----------------------- |
| Transport               | TCP                    | TCP                     | **QUIC (UDP)**          |
| Multiplexage            | Non (1 req/connexion)  | **Oui** (streams)       | **Oui** (streams)       |
| Compression headers     | Non                    | **HPACK**               | **QPACK**               |
| Chiffrement             | Optionnel (HTTPS)      | Pratiquement obligatoire| **Toujours** (TLS 1.3)  |
| Server Push             | Non                    | Oui (peu utilise)       | Non (retire)            |
| Head-of-line blocking   | Oui (TCP + HTTP)       | TCP seulement           | **Aucun**               |
| Temps de connexion      | 1-2 RTT (TCP+TLS)     | 1-2 RTT                 | **0-1 RTT**             |

### Le probleme de HTTP/1.1

```
HTTP/1.1 : une requete a la fois par connexion

Connexion 1: ──[GET /page]──────[reponse]──[GET /style.css]──[reponse]──
Connexion 2: ──[GET /script.js]──[reponse]──[GET /image.png]──[reponse]──
Connexion 3: ──[GET /font.woff]──[reponse]──

Probleme : les navigateurs ouvrent 6 connexions TCP en parallele
           = overhead de handshakes TCP + TLS

HTTP/2 : multiplexage sur une seule connexion

Connexion unique:
  Stream 1: ──[GET /page]────[reponse]──
  Stream 2: ──[GET /style.css]──[reponse]──
  Stream 3: ──[GET /script.js]────[reponse]──
  Stream 4: ──[GET /image.png]──────[reponse]──
  (tout en parallele sur la meme connexion TCP)
```

> [!info] HTTP/3 et QUIC
> HTTP/3 utilise **QUIC** au lieu de TCP. QUIC est un protocole de transport construit au-dessus d'**UDP** par Google. Son avantage principal : si un paquet est perdu dans un stream, il ne bloque pas les autres streams (pas de head-of-line blocking au niveau transport). De plus, QUIC integre TLS 1.3 directement, ce qui reduit le nombre d'aller-retours pour etablir la connexion.

---

## HTTPS et TLS

### Qu'est-ce que TLS ?

**TLS** (Transport Layer Security, successeur de SSL) est un protocole de chiffrement qui protege les communications HTTP. HTTPS = HTTP + TLS.

TLS fournit trois garanties :
1. **Confidentialite** : les donnees sont chiffrees (personne ne peut les lire en transit)
2. **Integrite** : les donnees ne peuvent pas etre modifiees sans detection
3. **Authentification** : le serveur prouve son identite via un certificat

### La chaine de certificats

```
┌──────────────────────────────────┐
│   Autorite de Certification      │   Certificat racine (CA root)
│   racine (Root CA)               │   Pre-installe dans le navigateur/OS
│   ex: DigiCert, Let's Encrypt    │
└────────────┬─────────────────────┘
             │ signe
             ↓
┌──────────────────────────────────┐
│   Autorite intermediaire         │   Certificat intermediaire
│   (Intermediate CA)              │
└────────────┬─────────────────────┘
             │ signe
             ↓
┌──────────────────────────────────┐
│   Certificat du serveur          │   Certificat feuille (leaf)
│   example.com                    │   Contient la cle publique du serveur
└──────────────────────────────────┘
```

### Le handshake TLS 1.3 (simplifie)

```
Client                                    Serveur
  │                                          │
  │── ClientHello ─────────────────────────→│
  │   (versions TLS, suites de chiffrement,  │
  │    cle publique ephemere DH)             │
  │                                          │
  │←─ ServerHello + Certificat + Finish ────│
  │   (cle publique DH du serveur,           │
  │    certificat, preuve d'identite)        │
  │                                          │
  │── Finish ──────────────────────────────→│
  │   (confirmation du client)               │
  │                                          │
  │     DONNEES CHIFFREES (1 RTT)           │
  │←──────────────────────────────────────→│
```

### Let's Encrypt

**Let's Encrypt** est une autorite de certification **gratuite** et **automatisee**. Elle a democratise HTTPS en rendant l'obtention de certificats triviale :

```bash
# Installer certbot
sudo apt install certbot python3-certbot-nginx

# Obtenir un certificat pour un domaine
sudo certbot --nginx -d example.com -d www.example.com

# Renouvellement automatique (les certificats expirent apres 90 jours)
sudo certbot renew --dry-run

# Verifier les certificats installes
sudo certbot certificates
```

> [!warning] HTTPS n'est pas optionnel
> En 2026, il n'y a **aucune raison** de servir un site en HTTP simple. Les navigateurs modernes marquent les sites HTTP comme "non securises". Let's Encrypt rend HTTPS gratuit et facile. Utilisez-le toujours.

---

## Cookies

Les cookies sont des petites donnees que le serveur envoie au client via le header `Set-Cookie`. Le navigateur les stocke et les renvoie automatiquement a chaque requete vers le meme domaine.

### Fonctionnement

```
1. Le serveur envoie un cookie :
   HTTP/1.1 200 OK
   Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Lax; Max-Age=86400

2. Le navigateur stocke le cookie

3. A chaque requete suivante vers le meme domaine :
   GET /api/profile HTTP/1.1
   Cookie: session_id=abc123
```

### Attributs des cookies

| Attribut     | Role                                                          |
| ------------ | ------------------------------------------------------------- |
| `HttpOnly`   | Le cookie n'est **pas** accessible en JavaScript (`document.cookie`). Protege contre le vol de session par XSS. |
| `Secure`     | Le cookie n'est envoye que sur les connexions **HTTPS**.       |
| `SameSite`   | Controle quand le cookie est envoye dans les requetes cross-site. Valeurs : `Strict`, `Lax`, `None`. |
| `Max-Age`    | Duree de vie en secondes. `Max-Age=0` supprime le cookie.     |
| `Expires`    | Date d'expiration (ancienne methode, preferer `Max-Age`).      |
| `Domain`     | Domaine pour lequel le cookie est valide.                      |
| `Path`       | Chemin pour lequel le cookie est valide.                       |

> [!example] Bonnes pratiques pour les cookies
> ```
> Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax; Max-Age=86400; Path=/
> ```
> - **HttpOnly** : toujours pour les cookies de session (anti-XSS)
> - **Secure** : toujours (vous etes en HTTPS, n'est-ce pas ?)
> - **SameSite=Lax** : bon compromis securite/usabilite
> - **Max-Age** raisonnable : pas de session eternelle

---

## Sessions vs Tokens

Deux approches pour gerer l'**authentification** dans les applications web :

### Sessions (Stateful)

```
┌──────────┐     1. Login (email/password)      ┌──────────┐
│  Client  │ ──────────────────────────────────→│  Serveur │
│          │                                     │          │
│          │     2. Set-Cookie: session=abc123   │  Stocke  │
│          │ ←──────────────────────────────────│  abc123  │
│          │                                     │  en RAM  │
│          │     3. Cookie: session=abc123       │  ou DB   │
│          │ ──────────────────────────────────→│          │
│          │     4. Le serveur cherche abc123    │          │
│          │        dans son store de sessions   │          │
└──────────┘                                     └──────────┘
```

### Tokens JWT (Stateless)

```
┌──────────┐     1. Login (email/password)      ┌──────────┐
│  Client  │ ──────────────────────────────────→│  Serveur │
│          │                                     │          │
│          │     2. {"token": "eyJhbGci..."}    │  Signe   │
│          │ ←──────────────────────────────────│  le JWT  │
│          │                                     │  (rien a │
│          │     3. Authorization: Bearer eyJ... │  stocker)│
│          │ ──────────────────────────────────→│          │
│          │     4. Le serveur VERIFIE la        │          │
│          │        signature du JWT (pas de DB) │          │
└──────────┘                                     └──────────┘
```

### Comparaison

| Aspect            | Sessions (cookies)              | Tokens (JWT)                     |
| ----------------- | ------------------------------- | -------------------------------- |
| Stockage serveur  | Oui (RAM, Redis, DB)            | Non (stateless)                  |
| Revocation        | Facile (supprimer la session)   | Difficile (attendre expiration)  |
| Scalabilite       | Complexe (sessions partagees)   | Simple (rien a partager)         |
| Taille            | Petit cookie                    | Token plus gros (payload encode) |
| Securite XSS      | HttpOnly protege le cookie      | Token en localStorage = vulnerable |
| CSRF              | Vulnerable (cookie automatique) | Non vulnerable (header manuel)   |

> [!tip] En pratique
> Beaucoup d'applications modernes utilisent un **hybride** : un JWT stocke dans un cookie HttpOnly. Cela combine la securite des cookies (pas d'acces JavaScript) avec la nature stateless du JWT.

---

## WebSocket

HTTP est un protocole **unidirectionnel** : le client envoie une requete, le serveur repond. Pour les applications en temps reel (chat, jeux, dashboards), on a besoin de communication **bidirectionnelle**. C'est le role de **WebSocket**.

### Le handshake d'upgrade

WebSocket commence par une requete HTTP classique avec un header `Upgrade` :

```
Client → Serveur (requete HTTP) :
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

Serveur → Client (reponse HTTP 101) :
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

Apres le handshake, la connexion TCP reste ouverte :
Client ←──── Messages bidirectionnels ────→ Serveur
  "Bonjour"  →
              ←  "Hello!"
  "Nouvelle commande"  →
              ←  "Notification: commande recue"
              ←  "Notification: commande expediee"  (push serveur)
```

### Cas d'usage de WebSocket

- **Chat en temps reel** (Slack, Discord, WhatsApp Web)
- **Jeux multijoueurs** en ligne
- **Dashboards en temps reel** (monitoring, trading)
- **Editeurs collaboratifs** (Google Docs, Figma)
- **Notifications push** dans le navigateur

> [!info] WebSocket vs HTTP
> N'utilisez pas WebSocket pour tout. Si vous avez juste besoin de recuperer des donnees (CRUD classique), HTTP REST est plus simple et mieux outille. WebSocket est pour le **temps reel bidirectionnel**.

---

## Server-Sent Events (SSE)

SSE est une alternative plus simple a WebSocket quand vous avez besoin uniquement de **push du serveur vers le client** (unidirectionnel).

```
Client → Serveur :
GET /events HTTP/1.1
Accept: text/event-stream

Serveur → Client (flux continu) :
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache

data: {"type": "notification", "message": "Nouveau message"}

data: {"type": "update", "progress": 45}

event: error
data: {"message": "Connexion interrompue"}
```

### Comparaison WebSocket vs SSE

| Aspect          | WebSocket               | SSE                         |
| --------------- | ----------------------- | --------------------------- |
| Direction       | Bidirectionnel          | Serveur → Client uniquement |
| Protocole       | ws:// ou wss://         | HTTP classique              |
| Reconnexion     | A gerer manuellement    | Automatique (EventSource)   |
| Navigateur      | API WebSocket           | API EventSource             |
| Complexite      | Plus complexe           | Plus simple                 |
| Cas d'usage     | Chat, jeux, collab      | Notifications, feeds, logs  |

```javascript
// Cote client (JavaScript)
const source = new EventSource('/api/events');

source.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('Recu:', data);
};

source.onerror = (error) => {
    console.error('Erreur SSE:', error);
};
```

---

## REST vs GraphQL

### REST (Representational State Transfer)

REST est un **style d'architecture** pour les APIs, base sur les ressources et les methodes HTTP :

```
GET    /api/users          → Liste des utilisateurs
GET    /api/users/42       → Utilisateur 42
POST   /api/users          → Creer un utilisateur
PUT    /api/users/42       → Modifier l'utilisateur 42
DELETE /api/users/42       → Supprimer l'utilisateur 42
GET    /api/users/42/posts → Posts de l'utilisateur 42
```

### GraphQL

GraphQL utilise un **seul endpoint** et le client specifie exactement les donnees qu'il veut :

```graphql
# Un seul endpoint : POST /graphql

# Le client demande exactement ce qu'il veut
query {
  user(id: 42) {
    name
    email
    posts(limit: 5) {
      title
      createdAt
    }
  }
}

# Reponse : exactement la structure demandee
{
  "data": {
    "user": {
      "name": "Alice",
      "email": "alice@example.com",
      "posts": [
        {"title": "Mon premier post", "createdAt": "2026-01-15"},
        ...
      ]
    }
  }
}
```

### Comparaison

| Aspect              | REST                              | GraphQL                         |
| ------------------- | --------------------------------- | ------------------------------- |
| Endpoints           | Multiples (`/users`, `/posts`)    | Un seul (`/graphql`)            |
| Over-fetching       | Oui (recoit tout meme si inutile) | Non (on demande exactement ce qu'on veut) |
| Under-fetching      | Oui (N+1 requetes)               | Non (une seule requete)         |
| Typage              | Non standard (OpenAPI optionnel)  | Schema type natif               |
| Cache               | Facile (URLs = cles de cache)     | Plus complexe (un seul endpoint)|
| Courbe d'apprentissage | Faible                         | Moyenne                         |
| Outillage           | curl, Postman, fetch              | Apollo, Relay, GraphiQL         |
| Cas d'usage ideal   | CRUD simple, microservices        | Frontend complexe, mobile       |

> [!tip] Quand choisir quoi ?
> - **REST** : APIs publiques, CRUD simple, microservices, quand le cache HTTP est important
> - **GraphQL** : applications mobiles (minimiser la bande passante), frontends complexes avec beaucoup de vues differentes, quand l'over-fetching est un vrai probleme

---

## gRPC

**gRPC** (Google Remote Procedure Call) est un framework de communication hautes performances qui utilise **Protocol Buffers** (protobuf) comme format de serialisation binaire.

### Fonctionnement

```
1. Definir le contrat dans un fichier .proto :

   syntax = "proto3";
   
   service UserService {
     rpc GetUser (UserRequest) returns (UserResponse);
     rpc ListUsers (Empty) returns (stream UserResponse);  // streaming
   }
   
   message UserRequest {
     int32 id = 1;
   }
   
   message UserResponse {
     int32 id = 1;
     string name = 2;
     string email = 3;
   }

2. Generer le code client/serveur automatiquement (protoc)
3. Appeler les methodes comme des fonctions locales
```

### Quand utiliser gRPC ?

| Aspect        | REST/JSON            | gRPC/Protobuf             |
| ------------- | -------------------- | ------------------------- |
| Format        | JSON (texte)         | Protobuf (binaire)        |
| Performance   | Correcte             | Excellente (10x plus compact) |
| Streaming     | SSE ou WebSocket     | Natif (4 types de streaming) |
| Contrat       | OpenAPI (optionnel)  | .proto (obligatoire)      |
| Navigateur    | Natif                | Necessite grpc-web        |
| Cas d'usage   | APIs publiques       | Communication inter-services |

> [!info] gRPC en microservices
> gRPC est ideal pour la communication **entre microservices** au sein d'un cluster, ou la performance est critique et ou le navigateur n'est pas implique. Pour les APIs publiques face aux navigateurs, REST ou GraphQL restent plus pratiques.

---

## Bonnes pratiques de design d'API

### Versioning

```
# Via l'URL (le plus courant)
GET /api/v1/users
GET /api/v2/users

# Via un header
GET /api/users
Accept: application/vnd.api.v2+json
```

### Pagination

```bash
# Offset-based (simple)
GET /api/users?page=2&per_page=20

# Reponse avec metadonnees :
{
  "data": [...],
  "pagination": {
    "page": 2,
    "per_page": 20,
    "total": 150,
    "total_pages": 8
  }
}

# Cursor-based (meilleures performances sur de gros datasets)
GET /api/users?cursor=eyJpZCI6NDJ9&limit=20
```

### Filtrage et tri

```bash
# Filtrage
GET /api/users?status=active&role=admin

# Tri
GET /api/users?sort=created_at&order=desc

# Recherche
GET /api/users?search=alice
```

### HATEOAS (Hypermedia as the Engine of Application State)

HATEOAS est un concept REST ou les reponses incluent des **liens** vers les actions possibles :

```json
{
  "id": 42,
  "name": "Alice",
  "email": "alice@example.com",
  "_links": {
    "self": {"href": "/api/users/42"},
    "posts": {"href": "/api/users/42/posts"},
    "delete": {"href": "/api/users/42", "method": "DELETE"}
  }
}
```

> [!example] Checklist d'une bonne API REST
> - Utiliser les **methodes HTTP** correctement (GET pour lire, POST pour creer, etc.)
> - Retourner les **codes de statut** semantiquement corrects
> - Utiliser des **noms de ressources au pluriel** : `/users`, pas `/user`
> - Supporter la **pagination** des que la liste peut etre longue
> - Documenter avec **OpenAPI/Swagger**
> - Versionner l'API des le debut
> - Retourner des **messages d'erreur** clairs et structures
> - Limiter le debit (**rate limiting**) pour eviter les abus

---

## Carte Mentale ASCII

```
                         HTTP ET PROTOCOLES WEB
                                 │
        ┌──────────┬─────────────┼──────────┬──────────────┐
        │          │             │          │              │
     Messages   Methodes     Versions    Securite     Temps reel
        │          │             │          │              │
   ┌────┴────┐  GET  POST   ┌───┴───┐   HTTPS        ┌───┴───┐
   │         │  PUT  DELETE  │       │   TLS 1.3      │       │
  Requete  Reponse PATCH    HTTP/1.1│   Certificats  WebSocket SSE
  Headers  Status  HEAD     HTTP/2  │   Let's Encrypt  │
  Body     codes   OPTIONS  HTTP/3  │                  Chat
           1xx-5xx          QUIC    │                  Jeux
                                    │
                              ┌─────┴─────┐
                              │           │
                            Cookies    Sessions
                            HttpOnly   vs Tokens
                            Secure     JWT
                            SameSite   Stateless

        ┌──────────┬───────────┐
        │          │           │
      REST     GraphQL      gRPC
    Ressources  Schema     Protobuf
    CRUD        1 endpoint  Streaming
    Cache HTTP  Typage     Inter-service
```

---

## Exercices

### Exercice 1 : Explorer les requetes HTTP

Utilisez `curl -v` pour envoyer des requetes vers `https://httpbin.org` :
1. `GET /get` : observez les headers de requete et de reponse
2. `POST /post` avec un body JSON : observez le code 200 et le body retourne
3. `GET /status/404` : observez le code de statut
4. `GET /redirect/3` : observez les redirections (avec `-L`)
5. `GET /headers` : observez vos headers tels que le serveur les voit

### Exercice 2 : Comparer les protocoles

Pour chaque scenario, indiquez quel protocole/technologie est le plus adapte (REST, GraphQL, gRPC, WebSocket, SSE) et justifiez :
1. Une API publique de catalogue de produits
2. Un systeme de chat entre utilisateurs
3. Communication entre 50 microservices internes
4. Un dashboard qui affiche des metriques en temps reel
5. Une application mobile qui affiche un profil utilisateur complexe (avec posts, amis, photos)

### Exercice 3 : Analyser un certificat TLS

```bash
# Utilisez cette commande pour analyser le certificat d'un site :
openssl s_client -connect example.com:443 -showcerts < /dev/null 2>/dev/null \
  | openssl x509 -text -noout
```
1. Qui est l'emetteur (Issuer) du certificat ?
2. Pour quel domaine le certificat est-il valide (Subject) ?
3. Quand expire-t-il ?
4. Quel algorithme de signature est utilise ?

---

## Liens

- [[01 - Fondamentaux Reseaux]] -- Modele OSI/TCP-IP, adresses IP, TCP/UDP, DNS
- [[03 - Securite Reseau]] -- Firewalls, SSH, VPN, TLS en profondeur
- [[08 - APIs REST avec Flask]] -- Construire une API REST en Python avec Flask
