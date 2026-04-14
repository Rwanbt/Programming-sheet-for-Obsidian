# APIs REST avec FastAPI

## Qu'est-ce que FastAPI ?

**FastAPI** est un framework web Python moderne, concu pour creer des APIs rapidement avec des performances elevees. Il repose sur deux piliers : les **type hints** Python et la programmation **asynchrone**.

> [!tip] Analogie
> Si Flask est une **boite a outils** ou vous choisissez et assemblez chaque outil, FastAPI est un **atelier complet** : les outils sont la, organises, et ils fonctionnent ensemble automatiquement. Vous declarez ce que vous voulez (types, modeles), et FastAPI genere la validation, la documentation et la serialisation pour vous.

Ses avantages principaux :
- **Rapide** : performances comparables a Node.js et Go (grace a Starlette et uvicorn)
- **Typage** : utilise les type hints Python pour la validation automatique
- **Documentation** : genere automatiquement Swagger UI et ReDoc
- **Async natif** : supporte nativement `async`/`await`
- **Validation** : utilise Pydantic pour la validation des donnees

---

## Installation

```python
# Creer un environnement virtuel
# python -m venv venv
# source venv/bin/activate  (Linux/Mac)
# venv\Scripts\activate     (Windows)

# Installer FastAPI et uvicorn (serveur ASGI)
# pip install fastapi uvicorn

# uvicorn est le serveur qui execute l'application FastAPI
# (equivalent de flask run, mais pour les applications ASGI/async)
```

> [!info] WSGI vs ASGI
> Flask utilise **WSGI** (Web Server Gateway Interface), le standard synchrone. FastAPI utilise **ASGI** (Asynchronous Server Gateway Interface), le standard asynchrone. ASGI peut gerer des milliers de connexions simultanees grace a `async`/`await`, tandis que WSGI bloque un worker par requete.

---

## Premiere application

```python
from fastapi import FastAPI

# Creer l'application
app = FastAPI(
    title="Mon API",
    description="Une API de demonstration avec FastAPI",
    version="1.0.0"
)

@app.get("/")
def accueil():
    return {"message": "Bienvenue sur FastAPI !"}

@app.get("/bonjour/{nom}")
def bonjour(nom: str):
    return {"message": f"Bonjour {nom} !"}
```

```
# Lancer le serveur :
# uvicorn main:app --reload
#
# main   = nom du fichier (main.py)
# app    = nom de la variable FastAPI
# --reload = rechargement automatique en dev
#
# Sortie :
# INFO:     Uvicorn running on http://127.0.0.1:8000
# INFO:     Started reloader process
#
# URLs importantes :
# http://127.0.0.1:8000          -> L'API
# http://127.0.0.1:8000/docs     -> Swagger UI (documentation interactive)
# http://127.0.0.1:8000/redoc    -> ReDoc (documentation alternative)
```

---

## Parametres de chemin (Path Parameters)

```python
from fastapi import FastAPI, Path

app = FastAPI()

# Parametre simple avec type hint
@app.get("/users/{user_id}")
def get_user(user_id: int):
    # FastAPI convertit et valide automatiquement !
    # GET /users/42   -> user_id = 42 (int)
    # GET /users/abc  -> 422 Unprocessable Entity (erreur de validation)
    return {"user_id": user_id}

# Parametre avec validation via Path()
@app.get("/items/{item_id}")
def get_item(
    item_id: int = Path(..., ge=1, le=10000, description="ID de l'item")
):
    return {"item_id": item_id}

# Enum pour limiter les valeurs possibles
from enum import Enum

class Categorie(str, Enum):
    fiction = "fiction"
    science = "science"
    histoire = "histoire"

@app.get("/livres/{categorie}")
def livres_par_categorie(categorie: Categorie):
    return {"categorie": categorie.value}
    # GET /livres/fiction -> OK, GET /livres/cuisine -> 422
```

---

## Parametres de requete (Query Parameters)

```python
from fastapi import FastAPI, Query
from typing import Optional

app = FastAPI()

# Parametres de query = tout ce qui n'est pas dans le chemin
@app.get("/items")
def lister_items(page: int = 1, limit: int = 10, recherche: Optional[str] = None):
    # GET /items?page=2&limit=20&recherche=python
    return {"page": page, "limit": limit, "recherche": recherche}

# Avec validation avancee via Query()
@app.get("/produits")
def lister_produits(
    page: int = Query(default=1, ge=1),
    limit: int = Query(default=10, ge=1, le=100),
    q: Optional[str] = Query(default=None, min_length=2, max_length=100),
    tags: list[str] = Query(default=[])
):
    # GET /produits?page=1&limit=20&q=python&tags=web&tags=api
    return {"page": page, "limit": limit, "q": q, "tags": tags}
```

> [!info] Path vs Query
> - **Path parameters** (`/users/{id}`) : pour identifier une ressource specifique
> - **Query parameters** (`?page=1&limit=10`) : pour filtrer, trier, paginer
> - Regle : si le parametre est **obligatoire** pour trouver la ressource, c'est un path parameter. Sinon, c'est un query parameter.

---

## Corps de la requete avec Pydantic

Pydantic est le coeur de la validation dans FastAPI. Les **modeles Pydantic** definissent la structure attendue des donnees :

### Modele de base

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

# Definir un modele Pydantic
class TaskCreate(BaseModel):
    titre: str
    description: Optional[str] = None
    priorite: str = "moyenne"
    fait: bool = False

@app.post("/tasks")
def creer_task(task: TaskCreate):
    # FastAPI :
    # 1. Lit le corps JSON de la requete
    # 2. Valide les types automatiquement
    # 3. Convertit en objet TaskCreate
    # 4. Retourne 422 si la validation echoue
    return {
        "message": "Tache creee",
        "task": task.model_dump()  # Convertit en dict
    }
```

```
# Requete valide :
# POST /tasks
# {"titre": "Apprendre FastAPI", "priorite": "haute"}
# -> 200 OK

# Requete invalide (titre manquant) :
# POST /tasks
# {"priorite": "haute"}
# -> 422 Unprocessable Entity avec details de l'erreur
```

### Field : validation avancee

```python
from pydantic import BaseModel, Field
from typing import Optional

class Produit(BaseModel):
    nom: str = Field(
        ...,                    # Obligatoire
        min_length=2,
        max_length=100,
        description="Nom du produit",
        examples=["Laptop Pro 15"]
    )
    prix: float = Field(
        ...,
        gt=0,                   # Strictement positif
        le=100_000,
        description="Prix en euros"
    )
    stock: int = Field(
        default=0,
        ge=0,
        description="Quantite en stock"
    )
    description: Optional[str] = Field(
        default=None,
        max_length=1000
    )
    tags: list[str] = Field(
        default_factory=list,   # Defaut = liste vide
        max_length=10           # Max 10 tags
    )
```

### Validators personnalises

```python
from pydantic import BaseModel, field_validator, model_validator

class Utilisateur(BaseModel):
    nom: str
    email: str
    age: int
    mot_de_passe: str
    confirmation: str

    @field_validator("email")
    @classmethod
    def valider_email(cls, v):
        if "@" not in v:
            raise ValueError("L'email doit contenir un @")
        return v.lower()

    @field_validator("age")
    @classmethod
    def valider_age(cls, v):
        if not 13 <= v <= 150:
            raise ValueError("Age invalide (13-150)")
        return v

    @model_validator(mode="after")
    def verifier_mots_de_passe(self):
        if self.mot_de_passe != self.confirmation:
            raise ValueError("Les mots de passe ne correspondent pas")
        return self
```

### Modeles imbriques

```python
from pydantic import BaseModel
from typing import Optional

class Adresse(BaseModel):
    rue: str
    ville: str
    code_postal: str
    pays: str = "France"

class ProfilComplet(BaseModel):
    nom: str
    email: str
    adresse: Adresse              # Modele imbrique
    centres_interet: list[str] = []

# Le JSON attendu :
# {"nom": "Alice", "email": "a@b.com",
#  "adresse": {"rue": "12 rue X", "ville": "Paris", "code_postal": "75002"},
#  "centres_interet": ["python"]}

@app.post("/profils")
def creer_profil(profil: ProfilComplet):
    return profil.model_dump()
```

---

## Modeles de reponse

FastAPI permet de definir le modele de la **reponse**, ce qui filtre automatiquement les champs :

```python
class UserCreate(BaseModel):
    nom: str
    email: str
    mot_de_passe: str

class UserResponse(BaseModel):  # SANS mot_de_passe !
    id: int
    nom: str
    email: str

# response_model filtre les champs retournes
@app.post("/users", response_model=UserResponse, status_code=201)
def creer_user(user: UserCreate):
    nouveau = {"id": 1, "nom": user.nom, "email": user.email,
               "mot_de_passe": user.mot_de_passe}  # NE sera PAS dans la reponse
    return nouveau
```

> [!warning] Toujours utiliser un response_model
> Sans `response_model`, FastAPI retourne tout, y compris les champs sensibles. Le `response_model` agit comme un **filtre de securite** automatique.

---

## Codes de statut

```python
from fastapi import FastAPI, status

app = FastAPI()

# Definir le code de statut par defaut
@app.post("/items", status_code=status.HTTP_201_CREATED)
def creer_item(item: dict):
    return item

@app.delete("/items/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
def supprimer_item(item_id: int):
    pass  # 204 = pas de contenu dans la reponse

# Les constantes status sont plus lisibles que les nombres :
# status.HTTP_200_OK
# status.HTTP_201_CREATED
# status.HTTP_204_NO_CONTENT
# status.HTTP_400_BAD_REQUEST
# status.HTTP_401_UNAUTHORIZED
# status.HTTP_403_FORBIDDEN
# status.HTTP_404_NOT_FOUND
# status.HTTP_422_UNPROCESSABLE_ENTITY
# status.HTTP_500_INTERNAL_SERVER_ERROR
```

---

## Documentation automatique

FastAPI genere **automatiquement** deux interfaces de documentation :

```
  /docs  -> Swagger UI (interface interactive, testable dans le navigateur)
  /redoc -> ReDoc (documentation detaillee avec schemas et exemples)

  Tout est genere automatiquement depuis les type hints,
  les modeles Pydantic et les docstrings !
```

```python
app = FastAPI(
    title="API de gestion de taches",
    description="CRUD complet pour les taches avec filtrage et pagination.",
    version="2.0.0",
)

@app.get("/tasks", summary="Lister les taches", tags=["Taches"])
def lister_tasks(page: int = 1, limit: int = 10):
    """Retourne la liste paginee de toutes les taches."""
    return {"tasks": [], "page": page, "limit": limit}
```

> [!tip] Analogie
> La documentation automatique de FastAPI, c'est comme si un architecte creait automatiquement les plans de la maison **pendant** la construction. Avec Flask, il faut dessiner les plans a la main (Swagger separement). Avec FastAPI, les plans sont toujours a jour car ils sont generes directement depuis le code.

---

## Injection de dependances

Le systeme `Depends` de FastAPI permet de partager de la logique entre plusieurs endpoints :

```python
from fastapi import FastAPI, Depends, HTTPException, Header
from typing import Optional

app = FastAPI()

# Dependance simple : verification d'authentification
async def verifier_token(authorization: Optional[str] = Header(None)):
    if authorization is None:
        raise HTTPException(status_code=401, detail="Token manquant")
    if not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Format de token invalide")
    token = authorization.replace("Bearer ", "")
    # Ici on verifierait le token en base ou via JWT
    if token != "secret123":
        raise HTTPException(status_code=401, detail="Token invalide")
    return {"user_id": 42, "role": "admin"}

# Dependance pour la pagination
def pagination(page: int = 1, limit: int = 10):
    if page < 1:
        page = 1
    if limit > 100:
        limit = 100
    return {"page": page, "limit": limit, "offset": (page - 1) * limit}

# Utilisation dans les endpoints
@app.get("/protected")
def route_protegee(user=Depends(verifier_token)):
    return {"message": f"Bienvenue utilisateur {user['user_id']}"}

@app.get("/items")
def lister_items(
    pagination_params=Depends(pagination),
    user=Depends(verifier_token)
):
    return {
        "items": [],
        "pagination": pagination_params,
        "user": user
    }
```

```python
# Dependance avec yield (nettoyage automatique)
async def get_db():
    db = DatabaseSession()
    try:
        yield db       # Le endpoint recoit la connexion
    finally:
        db.close()     # Nettoyage automatique apres la requete

@app.get("/data")
async def get_data(db=Depends(get_db)):
    return {"data": db.query("SELECT * FROM data")}
```

> [!info] Injection de dependances
> Le pattern `Depends` est similaire a l'injection de dependances en Java (Spring) ou en Angular. L'avantage : les dependances sont declarees, testables, et reutilisables. Pour les tests, on peut facilement remplacer une dependance reelle par un mock.

---

## Middleware et CORS

```python
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
import time

app = FastAPI()

# Middleware personnalise : temps de reponse
@app.middleware("http")
async def ajouter_temps_reponse(request: Request, call_next):
    debut = time.time()
    response = await call_next(request)
    response.headers["X-Process-Time"] = f"{time.time() - debut:.4f}"
    return response

# CORS : autoriser les requetes cross-origin
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://mon-site.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## Endpoints asynchrones

```python
import asyncio
import httpx  # pip install httpx (client HTTP async)

# Endpoint synchrone (FastAPI utilise un thread pool en interne)
@app.get("/sync")
def endpoint_sync():
    import time
    time.sleep(1)
    return {"mode": "sync"}

# Endpoint asynchrone (plus efficace pour I/O)
@app.get("/async")
async def endpoint_async():
    await asyncio.sleep(1)  # Non-bloquant
    return {"mode": "async"}

# Appeler une API externe de maniere async
@app.get("/meteo/{ville}")
async def get_meteo(ville: str):
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.example.com/weather/{ville}")
        return response.json()
```

> [!warning] async def vs def
> - `async def` : quand votre endpoint fait des operations async (`await`)
> - `def` : quand votre endpoint fait des operations bloquantes (FastAPI le lance dans un thread pool)
> - **Ne jamais** faire d'operations bloquantes dans un `async def` : ca bloquerait toute la boucle

---

## Gestion des erreurs

### HTTPException

```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

items = {"abc": {"nom": "Widget", "prix": 9.99}}

@app.get("/items/{item_id}")
def get_item(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item '{item_id}' non trouve",
            headers={"X-Error": "Item not found"}  # Headers optionnels
        )
    return items[item_id]
```

### Gestionnaires d'erreurs personnalises

```python
from fastapi.responses import JSONResponse

class ItemIndisponibleError(Exception):
    def __init__(self, item_id: str, message: str = ""):
        self.item_id = item_id
        self.message = message

@app.exception_handler(ItemIndisponibleError)
async def item_indisponible_handler(request, exc: ItemIndisponibleError):
    return JSONResponse(status_code=503, content={
        "erreur": "Item indisponible", "item_id": exc.item_id, "message": exc.message
    })
```

---

## Routers (APIRouter)

Les routers permettent d'organiser une grande API en modules, similaire aux Blueprints de Flask :

```
mon_api/
├── main.py             # Point d'entree + include_router
├── routers/            # Un fichier par ressource (users.py, tasks.py)
├── models/             # Modeles Pydantic
├── dependencies.py     # Dependances partagees
└── requirements.txt
```

```python
# routers/tasks.py
from fastapi import APIRouter, HTTPException, status
from pydantic import BaseModel
from typing import Optional

router = APIRouter(prefix="/tasks", tags=["Taches"])

class TaskCreate(BaseModel):
    titre: str
    description: Optional[str] = None
    priorite: str = "moyenne"

tasks_db = []
next_id = 1

@router.get("/")
def lister_tasks():
    return tasks_db

@router.post("/", status_code=status.HTTP_201_CREATED)
def creer_task(task: TaskCreate):
    global next_id
    nouvelle = {"id": next_id, **task.model_dump(), "fait": False}
    tasks_db.append(nouvelle)
    next_id += 1
    return nouvelle

@router.delete("/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
def supprimer_task(task_id: int):
    task = next((t for t in tasks_db if t["id"] == task_id), None)
    if not task:
        raise HTTPException(status_code=404, detail="Tache non trouvee")
    tasks_db.remove(task)
```

```python
# main.py
from fastapi import FastAPI
from routers import tasks, users, auth

app = FastAPI(title="Mon API", version="1.0.0")

# Enregistrer les routers
app.include_router(tasks.router)
app.include_router(users.router)
app.include_router(auth.router, prefix="/auth", tags=["Authentification"])

@app.get("/")
def root():
    return {"message": "API en ligne", "version": "1.0.0"}
```

---

## Background Tasks

FastAPI permet d'executer des taches en arriere-plan apres avoir renvoye la reponse :

```python
from fastapi import BackgroundTasks

def envoyer_email(destinataire: str, sujet: str):
    import time
    time.sleep(3)  # Simule l'envoi
    print(f"Email envoye a {destinataire}: {sujet}")

@app.post("/inscrire")
def inscrire(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(envoyer_email, email, "Bienvenue !")
    return {"message": f"Inscription de {email} en cours"}  # Reponse immediate
```

> [!example] Cas d'usage : envoi d'emails, generation de PDF, traitement d'images, notifications push, nettoyage de fichiers temporaires.

---

## Upload de fichiers

```python
from fastapi import FastAPI, UploadFile, HTTPException

app = FastAPI()

@app.post("/upload")
async def upload_fichier(fichier: UploadFile):
    contenu = await fichier.read()
    return {"nom": fichier.filename, "type": fichier.content_type, "taille": len(contenu)}

@app.post("/upload-multiple")
async def upload_plusieurs(fichiers: list[UploadFile]):
    return {"fichiers": [{"nom": f.filename, "taille": len(await f.read())} for f in fichiers]}

@app.post("/upload-image")
async def upload_image(image: UploadFile):
    if image.content_type not in ["image/jpeg", "image/png", "image/webp"]:
        raise HTTPException(400, detail="Format non supporte")
    contenu = await image.read()
    if len(contenu) > 5 * 1024 * 1024:
        raise HTTPException(400, detail="Image trop volumineuse (max 5 Mo)")
    with open(f"uploads/{image.filename}", "wb") as f:
        f.write(contenu)
    return {"chemin": f"uploads/{image.filename}", "taille": len(contenu)}
```

---

## API CRUD complete avec Pydantic

```python
from fastapi import FastAPI, HTTPException, status, Query
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime

app = FastAPI(title="API Gestion de Projets", version="1.0.0")

# ── Modeles Pydantic ──────────────────────────────

class ProjetBase(BaseModel):
    nom: str = Field(..., min_length=2, max_length=100)
    description: Optional[str] = Field(None, max_length=500)
    budget: float = Field(..., gt=0)
    statut: str = Field(default="planifie")

class ProjetCreate(ProjetBase):
    pass

class ProjetUpdate(BaseModel):
    nom: Optional[str] = Field(None, min_length=2)
    description: Optional[str] = None
    budget: Optional[float] = Field(None, gt=0)
    statut: Optional[str] = None

class ProjetResponse(ProjetBase):
    id: int
    date_creation: datetime
    date_modification: datetime

# ── "Base de donnees" ─────────────────────────────

projets_db: dict[int, dict] = {}
next_id = 1

# ── Endpoints ─────────────────────────────────────

@app.get("/projets", response_model=list[ProjetResponse], tags=["Projets"])
def lister_projets(
    page: int = Query(default=1, ge=1),
    limit: int = Query(default=10, ge=1, le=100),
    statut: Optional[str] = None,
):
    """Liste tous les projets avec pagination et filtrage."""
    resultats = list(projets_db.values())
    if statut:
        resultats = [p for p in resultats if p["statut"] == statut]
    debut = (page - 1) * limit
    return resultats[debut:debut + limit]

@app.get("/projets/{projet_id}", response_model=ProjetResponse, tags=["Projets"])
def get_projet(projet_id: int):
    """Retourne un projet par son ID."""
    if projet_id not in projets_db:
        raise HTTPException(404, detail=f"Projet {projet_id} non trouve")
    return projets_db[projet_id]

@app.post(
    "/projets",
    response_model=ProjetResponse,
    status_code=status.HTTP_201_CREATED,
    tags=["Projets"]
)
def creer_projet(projet: ProjetCreate):
    """Cree un nouveau projet."""
    global next_id
    maintenant = datetime.now()
    nouveau = {
        "id": next_id,
        **projet.model_dump(),
        "date_creation": maintenant,
        "date_modification": maintenant,
    }
    projets_db[next_id] = nouveau
    next_id += 1
    return nouveau

@app.put("/projets/{projet_id}", response_model=ProjetResponse, tags=["Projets"])
def modifier_projet(projet_id: int, projet: ProjetUpdate):
    """Modifie un projet existant (mise a jour partielle)."""
    if projet_id not in projets_db:
        raise HTTPException(404, detail=f"Projet {projet_id} non trouve")

    existant = projets_db[projet_id]
    donnees = projet.model_dump(exclude_unset=True)  # Seulement les champs fournis
    existant.update(donnees)
    existant["date_modification"] = datetime.now()

    return existant

@app.delete(
    "/projets/{projet_id}",
    status_code=status.HTTP_204_NO_CONTENT,
    tags=["Projets"]
)
def supprimer_projet(projet_id: int):
    """Supprime un projet."""
    if projet_id not in projets_db:
        raise HTTPException(404, detail=f"Projet {projet_id} non trouve")
    del projets_db[projet_id]
```

---

## Integration SQLAlchemy (apercu)

```python
# pip install sqlalchemy
from sqlalchemy import create_engine, Column, Integer, String, Float
from sqlalchemy.orm import declarative_base, sessionmaker

DATABASE_URL = "sqlite:///./projets.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class ProjetDB(Base):
    __tablename__ = "projets"
    id = Column(Integer, primary_key=True, index=True)
    nom = Column(String, nullable=False)
    budget = Column(Float, nullable=False)

Base.metadata.create_all(bind=engine)

# Dependance FastAPI pour la session DB
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/projets-db")
def lister_projets_db(db=Depends(get_db)):
    return [{"id": p.id, "nom": p.nom} for p in db.query(ProjetDB).all()]
```

> [!info] SQLAlchemy et FastAPI
> SQLAlchemy est l'ORM le plus utilise en Python. Pour une version async, utilisez `sqlalchemy[asyncio]` avec `aiosqlite` ou `asyncpg`.

---

## Authentification (concepts)

### OAuth2 et JWT

```python
# pip install python-jose[cryptography] passlib[bcrypt]
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from datetime import datetime, timedelta

SECRET_KEY = "votre_cle_secrete_tres_longue_et_aleatoire"
ALGORITHM = "HS256"
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def creer_token(data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    to_encode["exp"] = datetime.utcnow() + (expires_delta or timedelta(minutes=30))
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

@app.post("/token")
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    if form_data.username != "admin" or form_data.password != "secret":
        raise HTTPException(status_code=401, detail="Identifiants incorrects")
    token = creer_token({"sub": form_data.username})
    return {"access_token": token, "token_type": "bearer"}

async def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")
        if not username:
            raise HTTPException(status_code=401, detail="Token invalide")
    except JWTError:
        raise HTTPException(status_code=401, detail="Token invalide")
    return {"username": username}

@app.get("/profil")
def mon_profil(user=Depends(get_current_user)):
    return {"message": f"Bonjour {user['username']}"}
```

> [!warning] Securite en production
> En production : stockez les mots de passe **hashes** (bcrypt), utilisez des cles secretes aleatoires, implementez le refresh token, utilisez HTTPS, et ajoutez du rate limiting.

---

## Comparaison Flask vs FastAPI

```
┌──────────────────┬────────────────────┬────────────────────┐
│                  │ Flask              │ FastAPI            │
├──────────────────┼────────────────────┼────────────────────┤
│ Typage           │ Optionnel          │ Exploite (valid.   │
│                  │                    │ auto, docs auto)   │
├──────────────────┼────────────────────┼────────────────────┤
│ Async            │ Limite (WSGI)      │ Natif (ASGI)       │
├──────────────────┼────────────────────┼────────────────────┤
│ Documentation    │ Manuelle           │ Automatique        │
├──────────────────┼────────────────────┼────────────────────┤
│ Validation       │ Manuelle           │ Pydantic integre   │
├──────────────────┼────────────────────┼────────────────────┤
│ Performance      │ ~500 req/s         │ ~3000+ req/s       │
├──────────────────┼────────────────────┼────────────────────┤
│ Apprentissage    │ Tres doux          │ Doux (si types OK) │
├──────────────────┼────────────────────┼────────────────────┤
│ Ecosysteme       │ Tres mature        │ Croissance rapide  │
├──────────────────┼────────────────────┼────────────────────┤
│ Quand choisir    │ Prototypes,        │ APIs modernes,     │
│                  │ projets simples    │ microservices      │
└──────────────────┴────────────────────┴────────────────────┘
```

> [!example] Choix entre Flask et FastAPI
> - **Flask** : vous demarrez en Python, votre projet est petit/moyen, vous voulez un maximum de liberte
> - **FastAPI** : vous voulez de la validation automatique, de la documentation generee, du typage strict et/ou de l'async natif

---

## Carte Mentale

```
                    APIs REST avec FastAPI
                            │
        ┌──────────┬────────┼────────┬──────────┐
        │          │        │        │          │
    Pydantic   Endpoints  Docs    Routers   Securite
        │          │        │        │          │
   ┌────┤     ┌────┤    ┌───┤    ┌───┤     ┌────┤
   │    │     │    │    │   │    │   │     │    │
BaseModel│   path   │ /docs │ APIRouter│  OAuth2  │
   │    │     │    │    │   │    │   │     │    │
 Field  │  query   │ /redoc│ prefix  │   JWT    │
   │    │     │    │    │   │    │   │     │    │
validator│  body   │  auto  │  tags   │ Depends  │
   │    │     │    │  generated│   │     │    │
 nested │  async  │        │ include │ HTTPException
   │    │     │    │        │ _router │
response│  status │        │        │
 model  │  codes  │        │     BackgroundTasks
        │         │        │
   Depends     middleware  │
   (injection) │        upload
              CORS       fichiers
```

---

## Exercices

### Exercice 1 : API CRUD avec Pydantic

Creez une API FastAPI pour gerer une bibliotheque de livres. Chaque livre a : `titre` (str, 2-200 chars), `auteur` (str), `annee` (int, 1000-2030), `isbn` (str, 13 chars), `genre` (enum), `note` (float, 0-5, optionnel). Utilisez des modeles Pydantic avec validation (BookCreate, BookUpdate, BookResponse). Implementez le CRUD complet avec pagination et filtrage par genre.

### Exercice 2 : Documentation et validation

Enrichissez l'API de l'exercice 1 avec :
- Des descriptions et exemples pour chaque endpoint (docstrings, `summary`, `tags`)
- Un validator personnalise pour l'ISBN (verifier le checksum)
- Des `response_model` pour chaque endpoint
- Testez la documentation generee sur `/docs`

### Exercice 3 : Authentification

Ajoutez un systeme d'authentification a l'API :
- Endpoint `/token` qui retourne un JWT
- Dependance `get_current_user` qui verifie le token
- Certains endpoints (POST, PUT, DELETE) necessitent une authentification
- Les endpoints GET restent publics
- Ajoutez un role "admin" necessaire pour supprimer des livres

---

## Liens

- [[08 - APIs REST avec Flask]] - Flask, le micro-framework classique
- [[10 - Python et Bases de Donnees]] - Integration avec les bases de donnees
- [[01 - Introduction au SQL]] - Bases de donnees SQL
