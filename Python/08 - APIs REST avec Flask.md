# APIs REST avec Flask

## Qu'est-ce qu'une API ?

Une **API** (Application Programming Interface) est un ensemble de regles qui permet a deux logiciels de communiquer entre eux. Une **API web** permet a un client (navigateur, application mobile, autre serveur) d'interagir avec un serveur via le protocole HTTP.

> [!tip] Analogie
> Une API c'est comme le **menu d'un restaurant**. Le client ne va pas en cuisine : il consulte le menu (la documentation), passe une commande au serveur (une requete HTTP), et recoit son plat (la reponse). Le client n'a pas besoin de savoir comment le plat est prepare, seulement ce qu'il peut commander et sous quel format il recevra sa reponse.

En C, il n'existe pas de framework web integre. Pour creer un serveur HTTP en C, il faudrait gerer les sockets, parser les requetes HTTP manuellement, gerer les connexions... Des centaines de lignes pour un simple "Hello World". Python et Flask rendent cela trivial.

---

## Les principes REST

**REST** (Representational State Transfer) est un style d'architecture pour les APIs web. Ce n'est pas un protocole rigide, mais un ensemble de **contraintes** de conception :

```
┌─────────────────────────────────────────────────────────────────┐
│                    Principes REST                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Client-Serveur                                              │
│     Le client et le serveur sont independants.                  │
│     Le client ne connait pas le fonctionnement interne          │
│     du serveur et vice-versa.                                   │
│                                                                 │
│  2. Sans etat (Stateless)                                       │
│     Chaque requete contient TOUTE l'information necessaire.     │
│     Le serveur ne garde aucune session entre les requetes.      │
│                                                                 │
│  3. Interface uniforme                                          │
│     Les ressources sont identifiees par des URLs.               │
│     On utilise les methodes HTTP standard (GET, POST...).       │
│     Les reponses sont auto-descriptives (Content-Type).         │
│                                                                 │
│  4. Ressources                                                  │
│     Tout est une ressource identifiee par une URL :             │
│     /users, /users/42, /users/42/posts                          │
│                                                                 │
│  5. Representations                                             │
│     Une meme ressource peut avoir plusieurs formats :           │
│     JSON, XML, HTML... (JSON est le standard de facto)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

> [!info] REST vs RPC
> - **REST** : on manipule des **ressources** avec des verbes HTTP (`GET /users/42`)
> - **RPC** (Remote Procedure Call) : on appelle des **fonctions** (`POST /getUser?id=42`)
> - REST est dominant pour les APIs publiques. GraphQL et gRPC sont des alternatives modernes.

---

## Les methodes HTTP

Les methodes HTTP correspondent aux operations **CRUD** (Create, Read, Update, Delete) :

```
┌──────────┬──────────────┬─────────────────┬──────────┬────────────┐
│ Methode  │ CRUD         │ Description     │ Corps ?  │ Idempotent │
├──────────┼──────────────┼─────────────────┼──────────┼────────────┤
│ GET      │ Read         │ Lire une ou     │ Non      │ Oui        │
│          │              │ plusieurs       │          │            │
│          │              │ ressources      │          │            │
├──────────┼──────────────┼─────────────────┼──────────┼────────────┤
│ POST     │ Create       │ Creer une       │ Oui      │ Non        │
│          │              │ nouvelle        │          │            │
│          │              │ ressource       │          │            │
├──────────┼──────────────┼─────────────────┼──────────┼────────────┤
│ PUT      │ Update       │ Remplacer une   │ Oui      │ Oui        │
│          │ (complet)    │ ressource       │          │            │
│          │              │ entierement     │          │            │
├──────────┼──────────────┼─────────────────┼──────────┼────────────┤
│ PATCH    │ Update       │ Modifier        │ Oui      │ Non*       │
│          │ (partiel)    │ partiellement   │          │            │
│          │              │ une ressource   │          │            │
├──────────┼──────────────┼─────────────────┼──────────┼────────────┤
│ DELETE   │ Delete       │ Supprimer une   │ Optionnel│ Oui        │
│          │              │ ressource       │          │            │
└──────────┴──────────────┴─────────────────┴──────────┴────────────┘

* PATCH peut etre idempotent selon l'implementation
```

> [!info] Idempotent ?
> Une requete est **idempotente** si l'envoyer 1 fois ou 10 fois produit le meme resultat. `GET /users/42` retourne toujours la meme chose. `DELETE /users/42` supprime l'utilisateur une fois, les appels suivants ne changent rien. `POST /users` cree un **nouvel** utilisateur a chaque appel, donc ce n'est pas idempotent.

---

## Les codes de statut HTTP

```
┌───────┬──────────────────────────────────────────────────────────┐
│ Code  │ Signification                                            │
├───────┼──────────────────────────────────────────────────────────┤
│       │ 1xx - Informationnel                                     │
│ 100   │ Continue (le serveur a recu les headers, envoyez le body)│
│ 101   │ Switching Protocols (passage a WebSocket par exemple)    │
├───────┼──────────────────────────────────────────────────────────┤
│       │ 2xx - Succes                                             │
│ 200   │ OK (requete reussie, reponse avec contenu)               │
│ 201   │ Created (ressource creee avec succes - apres POST)       │
│ 204   │ No Content (succes, mais pas de contenu a renvoyer)      │
├───────┼──────────────────────────────────────────────────────────┤
│       │ 3xx - Redirection                                        │
│ 301   │ Moved Permanently (la ressource a change d'URL)          │
│ 302   │ Found (redirection temporaire)                           │
├───────┼──────────────────────────────────────────────────────────┤
│       │ 4xx - Erreur client                                      │
│ 400   │ Bad Request (requete mal formee, donnees invalides)      │
│ 401   │ Unauthorized (authentification requise)                  │
│ 403   │ Forbidden (authentifie mais pas autorise)                │
│ 404   │ Not Found (ressource inexistante)                        │
│ 409   │ Conflict (conflit avec l'etat actuel - doublon, etc.)    │
├───────┼──────────────────────────────────────────────────────────┤
│       │ 5xx - Erreur serveur                                     │
│ 500   │ Internal Server Error (bug cote serveur)                 │
│ 502   │ Bad Gateway (le serveur intermediaire a recu une mauvaise│
│       │             reponse du serveur en amont)                 │
│ 503   │ Service Unavailable (serveur surcharge ou en maintenance)│
└───────┴──────────────────────────────────────────────────────────┘
```

> [!tip] Analogie
> Les codes HTTP sont comme les reponses d'un employe administratif :
> - **2xx** : "C'est fait !" (succes)
> - **4xx** : "Vous avez fait une erreur dans votre formulaire" (c'est la faute du client)
> - **5xx** : "Desole, nos systemes sont en panne" (c'est la faute du serveur)

---

## Le format JSON

JSON (JavaScript Object Notation) est le format standard pour les APIs REST :

```python
# JSON est natif en Python grace au module json
import json

# Dictionnaire Python -> JSON string
donnees = {
    "id": 1,
    "nom": "Alice Dupont",
    "email": "alice@example.com",
    "actif": True,
    "roles": ["admin", "user"],
    "adresse": {
        "ville": "Paris",
        "code_postal": "75001"
    }
}

# Serialiser (Python -> JSON)
json_str = json.dumps(donnees, indent=2, ensure_ascii=False)
print(json_str)

# Deserialiser (JSON -> Python)
donnees_recuperees = json.loads(json_str)
print(donnees_recuperees["nom"])  # "Alice Dupont"
```

```
Correspondance des types :
┌──────────────────┬──────────────────┐
│ Python           │ JSON             │
├──────────────────┼──────────────────┤
│ dict             │ object {}        │
│ list, tuple      │ array []         │
│ str              │ string ""        │
│ int, float       │ number           │
│ True / False     │ true / false     │
│ None             │ null             │
└──────────────────┴──────────────────┘
```

---

## Installation de Flask

```python
# Creer un environnement virtuel (voir cours precedent)
# python -m venv venv
# source venv/bin/activate  (Linux/Mac)
# venv\Scripts\activate     (Windows)

# Installer Flask
# pip install flask
```

> [!info] Flask en bref
> Flask est un **micro-framework** web Python. "Micro" signifie qu'il fournit le strict necessaire (routing, requetes, reponses) sans imposer de base de donnees, de systeme de templates ou d'ORM. On ajoute ce dont on a besoin via des extensions.

---

## Les bases de Flask

### Premiere application

```python
from flask import Flask

# Creer l'application Flask
# __name__ permet a Flask de trouver les templates et fichiers statiques
app = Flask(__name__)

# Definir une route avec un decorateur
@app.route("/")
def accueil():
    return "Bienvenue sur mon API !"

@app.route("/bonjour")
def bonjour():
    return {"message": "Bonjour le monde !"}  # Flask convertit le dict en JSON

# Lancer le serveur de developpement
if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

```
# Dans le terminal :
# python app.py
#
# Sortie :
#  * Running on http://127.0.0.1:5000
#  * Debug mode: on
#
# Ouvrir http://127.0.0.1:5000 dans le navigateur
```

> [!warning] Mode debug
> `debug=True` active le rechargement automatique (le serveur redemarrera quand vous modifiez le code) et un debugger interactif en cas d'erreur. **Ne jamais utiliser en production** car le debugger permet d'executer du code arbitraire sur le serveur.

### Routes et decorateurs

```python
from flask import Flask

app = Flask(__name__)

# Route simple
@app.route("/")
def index():
    return {"status": "ok"}

# Route avec methode specifique
@app.route("/users", methods=["GET"])
def lister_users():
    return {"users": []}

@app.route("/users", methods=["POST"])
def creer_user():
    return {"message": "User cree"}, 201  # Tuple (reponse, code_status)

# Plusieurs methodes sur la meme route
@app.route("/items", methods=["GET", "POST"])
def items():
    from flask import request
    if request.method == "GET":
        return {"items": []}
    elif request.method == "POST":
        return {"message": "Item cree"}, 201
```

### Parametres de route

```python
from flask import Flask

app = Flask(__name__)

# Parametre simple (string par defaut)
@app.route("/users/<username>")
def profil(username):
    return {"username": username}
# GET /users/alice -> {"username": "alice"}

# Parametre type avec convertisseur
@app.route("/users/<int:user_id>")
def user_par_id(user_id):
    return {"id": user_id, "type": str(type(user_id))}
# GET /users/42 -> {"id": 42, "type": "<class 'int'>"}
# GET /users/abc -> 404 (ne matche pas int)

# Autres convertisseurs disponibles
@app.route("/articles/<string:slug>")    # string (defaut)
def article(slug):
    return {"slug": slug}

@app.route("/fichiers/<path:chemin>")    # path (accepte les /)
def fichier(chemin):
    return {"chemin": chemin}
# GET /fichiers/docs/rapport.pdf -> {"chemin": "docs/rapport.pdf"}

@app.route("/prix/<float:montant>")      # float
def prix(montant):
    return {"prix": montant}
```

---

## L'objet Request

L'objet `request` de Flask contient toutes les informations sur la requete entrante :

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/demo", methods=["GET", "POST"])
def demo_request():
    # Parametres de query string : /demo?page=2&limit=10
    page = request.args.get("page", 1, type=int)
    limit = request.args.get("limit", 10, type=int)

    # Methode HTTP
    methode = request.method  # "GET", "POST", etc.

    # Headers
    content_type = request.headers.get("Content-Type")
    auth = request.headers.get("Authorization")

    # Corps de la requete (POST/PUT)
    if request.method == "POST":
        # Si Content-Type: application/json
        donnees_json = request.json  # dict ou None
        # equivalent a request.get_json()

        # Si Content-Type: application/x-www-form-urlencoded
        donnees_form = request.form.get("champ")

        # Donnees brutes
        donnees_brutes = request.data  # bytes

    return {
        "methode": methode,
        "page": page,
        "limit": limit,
        "content_type": content_type,
    }
```

> [!warning] `request.json` peut etre `None`
> Si le header `Content-Type` n'est pas `application/json`, ou si le corps n'est pas du JSON valide, `request.json` retourne `None`. Utilisez `request.get_json(force=True)` pour forcer le parsing meme sans le bon Content-Type, ou `request.get_json(silent=True)` pour eviter une erreur 400 sur du JSON invalide.

---

## Les reponses

### jsonify et make_response

```python
from flask import Flask, jsonify, make_response

app = Flask(__name__)

# Methode 1 : retourner un dict (Flask >= 1.0 le convertit en JSON)
@app.route("/v1")
def reponse_dict():
    return {"message": "hello"}  # Content-Type: application/json automatique

# Methode 2 : jsonify (plus explicite, supporte les listes)
@app.route("/v2")
def reponse_jsonify():
    return jsonify([1, 2, 3])  # Retourne un tableau JSON

# Methode 3 : tuple (reponse, code_status)
@app.route("/v3")
def reponse_tuple():
    return {"created": True}, 201

# Methode 4 : tuple (reponse, code_status, headers)
@app.route("/v4")
def reponse_headers():
    return {"data": "ok"}, 200, {"X-Custom-Header": "valeur"}

# Methode 5 : make_response (controle total)
@app.route("/v5")
def reponse_complete():
    response = make_response(jsonify({"message": "hello"}), 200)
    response.headers["X-Custom"] = "valeur"
    response.set_cookie("session_id", "abc123", httponly=True)
    return response
```

---

## API CRUD complete : Gestion de taches

Voici une API complete pour gerer des taches (todo list) :

```python
from flask import Flask, request, jsonify, abort

app = Flask(__name__)

# "Base de donnees" en memoire (pour la demo)
taches = [
    {"id": 1, "titre": "Apprendre Flask", "fait": False, "priorite": "haute"},
    {"id": 2, "titre": "Creer une API", "fait": False, "priorite": "moyenne"},
    {"id": 3, "titre": "Ecrire des tests", "fait": True, "priorite": "basse"},
]
prochain_id = 4


def trouver_tache(tache_id):
    """Trouve une tache par son ID ou retourne None."""
    return next((t for t in taches if t["id"] == tache_id), None)


# ────────────────────────────────────────────
# GET /tasks - Lister toutes les taches
# ────────────────────────────────────────────
@app.route("/tasks", methods=["GET"])
def lister_taches():
    # Filtrage optionnel par statut
    fait = request.args.get("fait")
    if fait is not None:
        fait_bool = fait.lower() in ("true", "1", "oui")
        resultat = [t for t in taches if t["fait"] == fait_bool]
    else:
        resultat = taches

    # Tri optionnel
    tri = request.args.get("tri")
    if tri == "priorite":
        ordre = {"haute": 0, "moyenne": 1, "basse": 2}
        resultat = sorted(resultat, key=lambda t: ordre.get(t["priorite"], 9))

    return jsonify({
        "taches": resultat,
        "total": len(resultat)
    })


# ────────────────────────────────────────────
# GET /tasks/<id> - Obtenir une tache
# ────────────────────────────────────────────
@app.route("/tasks/<int:tache_id>", methods=["GET"])
def obtenir_tache(tache_id):
    tache = trouver_tache(tache_id)
    if tache is None:
        abort(404)
    return jsonify(tache)


# ────────────────────────────────────────────
# POST /tasks - Creer une tache
# ────────────────────────────────────────────
@app.route("/tasks", methods=["POST"])
def creer_tache():
    global prochain_id

    if not request.json:
        abort(400, description="Le corps de la requete doit etre du JSON")

    if "titre" not in request.json:
        abort(400, description="Le champ 'titre' est obligatoire")

    if not isinstance(request.json["titre"], str) or len(request.json["titre"].strip()) == 0:
        abort(400, description="Le titre doit etre une chaine non vide")

    priorite = request.json.get("priorite", "moyenne")
    if priorite not in ("haute", "moyenne", "basse"):
        abort(400, description="La priorite doit etre 'haute', 'moyenne' ou 'basse'")

    nouvelle_tache = {
        "id": prochain_id,
        "titre": request.json["titre"].strip(),
        "fait": request.json.get("fait", False),
        "priorite": priorite,
    }
    taches.append(nouvelle_tache)
    prochain_id += 1

    return jsonify(nouvelle_tache), 201


# ────────────────────────────────────────────
# PUT /tasks/<id> - Remplacer une tache
# ────────────────────────────────────────────
@app.route("/tasks/<int:tache_id>", methods=["PUT"])
def remplacer_tache(tache_id):
    tache = trouver_tache(tache_id)
    if tache is None:
        abort(404)

    if not request.json:
        abort(400, description="Le corps de la requete doit etre du JSON")

    if "titre" not in request.json:
        abort(400, description="Le champ 'titre' est obligatoire pour PUT")

    tache["titre"] = request.json["titre"]
    tache["fait"] = request.json.get("fait", False)
    tache["priorite"] = request.json.get("priorite", "moyenne")

    return jsonify(tache)


# ────────────────────────────────────────────
# DELETE /tasks/<id> - Supprimer une tache
# ────────────────────────────────────────────
@app.route("/tasks/<int:tache_id>", methods=["DELETE"])
def supprimer_tache(tache_id):
    tache = trouver_tache(tache_id)
    if tache is None:
        abort(404)

    taches.remove(tache)
    return "", 204  # No Content


if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

---

## Gestion des erreurs

### @app.errorhandler

```python
from flask import Flask, jsonify, abort

app = Flask(__name__)

# Gestionnaire d'erreur personnalise pour 404
@app.errorhandler(404)
def not_found(error):
    return jsonify({
        "erreur": "Ressource non trouvee",
        "message": str(error.description) if hasattr(error, 'description') else "Not Found",
        "code": 404
    }), 404

# Gestionnaire pour 400
@app.errorhandler(400)
def bad_request(error):
    return jsonify({
        "erreur": "Requete invalide",
        "message": str(error.description),
        "code": 400
    }), 400

# Gestionnaire pour 500 (erreurs internes)
@app.errorhandler(500)
def internal_error(error):
    return jsonify({
        "erreur": "Erreur interne du serveur",
        "message": "Une erreur inattendue s'est produite",
        "code": 500
    }), 500

# Gestionnaire pour toutes les exceptions non gerees
@app.errorhandler(Exception)
def handle_exception(e):
    return jsonify({
        "erreur": "Erreur inattendue",
        "message": str(e),
        "code": 500
    }), 500


# Utilisation de abort() pour declencher des erreurs
@app.route("/users/<int:user_id>")
def get_user(user_id):
    users = {1: "Alice", 2: "Bob"}
    if user_id not in users:
        abort(404, description=f"Utilisateur {user_id} inexistant")
    return jsonify({"id": user_id, "nom": users[user_id]})
```

---

## Blueprints : organiser une grande application

Quand l'application grandit, on ne peut pas tout mettre dans un seul fichier. Les **Blueprints** permettent de decouper l'application en modules :

```
mon_api/
├── app.py              # Point d'entree
├── config.py           # Configuration
├── blueprints/
│   ├── __init__.py
│   ├── users.py        # Routes /users
│   ├── tasks.py        # Routes /tasks
│   └── auth.py         # Routes /auth
└── requirements.txt
```

```python
# blueprints/users.py
from flask import Blueprint, jsonify, request

# Creer le blueprint
users_bp = Blueprint("users", __name__, url_prefix="/users")

# Les routes sont definies sur le blueprint, pas sur app
@users_bp.route("/", methods=["GET"])
def lister_users():
    return jsonify({"users": []})

@users_bp.route("/<int:user_id>", methods=["GET"])
def obtenir_user(user_id):
    return jsonify({"id": user_id, "nom": "Alice"})

@users_bp.route("/", methods=["POST"])
def creer_user():
    data = request.json
    return jsonify({"message": "User cree", "data": data}), 201
```

```python
# blueprints/tasks.py
from flask import Blueprint, jsonify

tasks_bp = Blueprint("tasks", __name__, url_prefix="/tasks")

@tasks_bp.route("/", methods=["GET"])
def lister_tasks():
    return jsonify({"tasks": []})
```

```python
# app.py - Point d'entree
from flask import Flask
from blueprints.users import users_bp
from blueprints.tasks import tasks_bp

def create_app():
    """Factory pattern : cree et configure l'application."""
    app = Flask(__name__)

    # Charger la configuration
    app.config.from_object("config.Config")

    # Enregistrer les blueprints
    app.register_blueprint(users_bp)   # -> /users/...
    app.register_blueprint(tasks_bp)   # -> /tasks/...

    return app

if __name__ == "__main__":
    app = create_app()
    app.run(debug=True)
```

> [!example] Application Factory Pattern
> Le pattern `create_app()` est la maniere recommandee de structurer une application Flask. Il permet de creer plusieurs instances de l'application avec des configurations differentes (test, dev, production) et facilite les tests unitaires.

---

## Middleware

Un middleware intercepte les requetes et/ou les reponses pour ajouter un traitement transversal :

```python
from flask import Flask, request, g
import time

app = Flask(__name__)

# before_request : execute AVANT chaque requete
@app.before_request
def avant_requete():
    g.debut = time.time()  # g est un objet global par requete
    print(f"[{request.method}] {request.path}")

# after_request : execute APRES chaque requete (meme en cas d'erreur)
@app.after_request
def apres_requete(response):
    duree = time.time() - g.debut
    response.headers["X-Response-Time"] = f"{duree:.4f}s"
    print(f"  -> {response.status_code} en {duree:.4f}s")
    return response  # IMPORTANT : retourner la reponse

# teardown_request : execute apres la reponse, meme en cas d'exception
@app.teardown_request
def nettoyage(exception=None):
    if exception:
        print(f"  ERREUR : {exception}")
```

---

## CORS (Cross-Origin Resource Sharing)

Par defaut, un navigateur bloque les requetes d'un domaine vers un autre (politique same-origin). CORS permet d'autoriser ces requetes :

```python
# pip install flask-cors

from flask import Flask
from flask_cors import CORS

app = Flask(__name__)

# Autoriser toutes les origines (pratique pour le dev, DANGEREUX en prod)
CORS(app)

# Ou configurer precisement
CORS(app, resources={
    r"/api/*": {
        "origins": ["http://localhost:3000", "https://mon-frontend.com"],
        "methods": ["GET", "POST", "PUT", "DELETE"],
        "allow_headers": ["Content-Type", "Authorization"],
    }
})
```

> [!warning] CORS en production
> Ne jamais utiliser `CORS(app)` sans restriction en production. Specifiez toujours les origines autorisees. CORS est une protection **cote navigateur** : un script curl ou un autre serveur peut toujours acceder a votre API.

---

## Variables d'environnement

Ne jamais mettre de secrets (cles API, mots de passe) directement dans le code :

```python
# pip install python-dotenv

# .env (JAMAIS commite dans git !)
# FLASK_ENV=development
# SECRET_KEY=ma_cle_super_secrete_123
# DATABASE_URL=postgresql://user:pass@localhost/madb

# config.py
import os
from dotenv import load_dotenv

load_dotenv()  # Charge les variables depuis .env

class Config:
    SECRET_KEY = os.environ.get("SECRET_KEY", "cle_par_defaut")
    DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///local.db")
    DEBUG = os.environ.get("FLASK_ENV") == "development"

# app.py
from flask import Flask

app = Flask(__name__)
app.config.from_object(Config)

print(app.config["SECRET_KEY"])  # Utilise la variable d'environnement
```

```
# .gitignore - TOUJOURS exclure .env
.env
.env.local
*.pyc
__pycache__/
venv/
```

> [!warning] Secrets et Git
> Si vous commitez accidentellement un fichier `.env`, le secret est **compromis** meme si vous le supprimez ensuite (il reste dans l'historique Git). Changez immediatement tous les secrets exposes.

---

## Structure de projet Flask

```
mon_api_flask/
├── app/
│   ├── __init__.py          # create_app() factory
│   ├── config.py            # Classes de configuration
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py          # Modele User
│   │   └── task.py          # Modele Task
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── users.py         # Blueprint users
│   │   ├── tasks.py         # Blueprint tasks
│   │   └── auth.py          # Blueprint authentification
│   ├── services/
│   │   ├── __init__.py
│   │   ├── user_service.py  # Logique metier users
│   │   └── task_service.py  # Logique metier tasks
│   └── utils/
│       ├── __init__.py
│       └── validators.py    # Fonctions de validation
├── tests/
│   ├── __init__.py
│   ├── test_users.py
│   └── test_tasks.py
├── .env                     # Variables d'environnement (pas dans git !)
├── .gitignore
├── requirements.txt
├── pyproject.toml
└── run.py                   # Point d'entree : python run.py
```

```python
# app/__init__.py
from flask import Flask
from flask_cors import CORS

def create_app(config_name="development"):
    app = Flask(__name__)
    app.config.from_object(f"app.config.{config_name.capitalize()}Config")
    CORS(app)

    from app.routes.users import users_bp
    from app.routes.tasks import tasks_bp
    app.register_blueprint(users_bp, url_prefix="/api/users")
    app.register_blueprint(tasks_bp, url_prefix="/api/tasks")

    return app

# run.py : from app import create_app; create_app().run(debug=True)
```

---

## Tester l'API

### Avec curl

```
# GET - Lister les taches
curl http://localhost:5000/tasks

# GET - Obtenir une tache
curl http://localhost:5000/tasks/1

# POST - Creer une tache
curl -X POST http://localhost:5000/tasks \
  -H "Content-Type: application/json" \
  -d '{"titre": "Nouvelle tache", "priorite": "haute"}'

# PUT - Modifier une tache
curl -X PUT http://localhost:5000/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"titre": "Tache modifiee", "fait": true}'

# DELETE - Supprimer une tache
curl -X DELETE http://localhost:5000/tasks/1
```

### Avec HTTPie (plus lisible)

```
# pip install httpie

# GET
http GET localhost:5000/tasks

# POST (HTTPie detecte automatiquement le JSON)
http POST localhost:5000/tasks titre="Nouvelle tache" priorite="haute"

# PUT
http PUT localhost:5000/tasks/1 titre="Modifiee" fait:=true

# DELETE
http DELETE localhost:5000/tasks/1
```

### Avec les tests unitaires Flask

```python
# tests/test_tasks.py
import pytest
from app import create_app

@pytest.fixture
def client():
    app = create_app("testing")
    with app.test_client() as client:
        yield client

def test_lister_taches(client):
    response = client.get("/tasks")
    assert response.status_code == 200
    assert "taches" in response.get_json()

def test_creer_tache(client):
    response = client.post("/tasks", json={"titre": "Test", "priorite": "haute"})
    assert response.status_code == 201
    assert response.get_json()["titre"] == "Test"

def test_creer_tache_sans_titre(client):
    assert client.post("/tasks", json={"priorite": "haute"}).status_code == 400

def test_tache_inexistante(client):
    assert client.get("/tasks/999").status_code == 404
```

> [!example] Postman
> Postman est un outil graphique tres populaire pour tester les APIs. Il permet de sauvegarder des collections de requetes, de definir des variables d'environnement, et de creer des tests automatises. C'est un excellent complement aux tests unitaires pour l'exploration et le debugging.

---

## Comparaison avec le C

> [!info] Flask vs C pour le web
> En C, creer un serveur HTTP necessite ~200 lignes minimum (sockets, parsing HTTP, gestion memoire). Flask le fait en 5 lignes. Le C n'a pas de framework web standard : il faut des bibliotheques externes pour le JSON (cJSON), le routing manuel, et la gestion memoire. Le C est utilise pour les **serveurs web eux-memes** (Nginx) ou les proxys haute performance, pas pour la logique applicative d'une API REST.

---

## Carte Mentale

```
                      APIs REST avec Flask
                              │
          ┌───────────┬───────┼───────┬───────────┐
          │           │       │       │           │
        REST        Flask   Requete  Reponse   Structure
          │           │       │       │           │
     ┌────┤      ┌────┤    ┌──┤    ┌──┤      ┌────┤
     │    │      │    │    │  │    │  │      │    │
  Principes│   routes  │  args │ jsonify│ Blueprints
     │    │      │    │    │  │    │  │      │    │
  Methodes│ decorateurs│  json│ status │ Factory
  HTTP    │      │    │    │  │ codes  │ Pattern
     │    │    run()  │ headers│    │      │
  Codes   │      │    │    │  │ make_  │   config
  Status  │  debug    │ form  │ response│     │
     │    │      │    │    │  │    │   ├────┤
  JSON    │  config   │ method│    │   │    │
     │    │           │       │    │  .env  │
  CRUD    │         error     │    │   │ tests
          │        handler    │    │ CORS
       Ressources      │     │    │
          │          abort    │ middleware
       URLs                  │
                          before/
                          after_request
```

---

## Exercices

### Exercice 1 : API basique

Creez une API Flask pour gerer une liste de livres (`/books`). Chaque livre a un `id`, un `titre`, un `auteur` et une `annee`. Implementez les 5 methodes CRUD (GET all, GET one, POST, PUT, DELETE) avec validation des donnees et codes de statut corrects.

### Exercice 2 : Filtrage et pagination

Ajoutez a l'API de l'exercice 1 :
- Filtrage par auteur (`/books?auteur=Hugo`)
- Tri par annee (`/books?tri=annee&ordre=desc`)
- Pagination (`/books?page=2&limit=5`) avec des metadonnees dans la reponse (`total`, `page`, `pages`, `limit`)

### Exercice 3 : Blueprints et structure

Restructurez l'API en utilisant :
- Un Blueprint pour `/books` et un pour `/authors`
- Le pattern Application Factory (`create_app`)
- Un fichier de configuration avec `.env`
- Des gestionnaires d'erreurs globaux en JSON

### Exercice 4 : Tests

Ecrivez des tests pytest complets pour votre API :
- Tester chaque endpoint (happy path et cas d'erreur)
- Tester les filtres et la pagination
- Tester les validations (donnees manquantes, types invalides)
- Utiliser des fixtures pour le setup/teardown

---

## Liens

- [[06 - Modules Packages et Venv]] - Modules, packages et environnements virtuels
- [[09 - APIs REST avec FastAPI]] - FastAPI, l'alternative moderne a Flask
- [[01 - Introduction au SQL]] - Bases de donnees SQL
