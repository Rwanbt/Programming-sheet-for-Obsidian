# Projet HBnB Evolution -- Clone AirBnB

> [!tip] Analogie
> Imagine que tu construis un **hotel en miniature** piece par piece. D'abord les fondations (le modele de donnees), puis les murs porteurs (la logique metier), puis la facade (l'API), puis la decoration interieure (le frontend), et enfin le toit (la securite). Chaque couche s'appuie sur la precedente. Si les fondations sont bancales, tout s'effondre. C'est exactement la philosophie de ce projet : construire une application web complete, couche par couche.

---

## 1. Presentation du Projet

### 1.1 Qu'est-ce que HBnB ?

**HBnB** (Holberton BnB) est un **clone simplifie d'AirBnB** -- une plateforme de reservation d'hebergements. C'est le projet capstone du Trimestre 2 : il reunit tout ce que tu as appris (Python, POO, API REST, bases de donnees, Docker, tests) dans une seule application fonctionnelle.

L'objectif n'est pas de reproduire AirBnB a l'identique (qui est un monstre de millions de lignes de code), mais d'en implementer les **concepts fondamentaux** :

- **Utilisateurs** : inscription, connexion, profils
- **Hebergements (Places)** : creation, description, prix, localisation
- **Avis (Reviews)** : noter et commenter un hebergement
- **Equipements (Amenities)** : Wi-Fi, piscine, parking, etc.
- **Reservation** : lier un utilisateur a un hebergement pour des dates donnees

### 1.2 Fonctionnalites Attendues

```
+-------------------------------------------------------------------+
|                   FONCTIONNALITES HBnB                            |
+-------------------------------------------------------------------+
|                                                                   |
|  UTILISATEURS                                                     |
|  - Inscription (email, mot de passe, nom, prenom)                 |
|  - Connexion (authentification JWT)                                |
|  - Profil utilisateur (consultation, modification)                 |
|  - Roles : admin / utilisateur normal                              |
|                                                                   |
|  HEBERGEMENTS (PLACES)                                             |
|  - Creer une annonce (titre, description, prix, coordonnees)      |
|  - Lister les annonces (avec filtres : prix, localisation)        |
|  - Consulter le detail d'une annonce                               |
|  - Modifier / supprimer (proprietaire uniquement)                  |
|                                                                   |
|  AVIS (REVIEWS)                                                    |
|  - Poster un avis sur un hebergement (texte + note)               |
|  - Consulter les avis d'un hebergement                             |
|  - Modifier / supprimer son propre avis                            |
|                                                                   |
|  EQUIPEMENTS (AMENITIES)                                           |
|  - Creer un equipement (admin)                                     |
|  - Associer des equipements a un hebergement                       |
|  - Filtrer les hebergements par equipement                         |
|                                                                   |
+-------------------------------------------------------------------+
```

### 1.3 Architecture Globale

Le projet suit une **architecture en couches** (layered architecture). Chaque couche a une responsabilite unique et communique uniquement avec ses voisines directes.

```
+===================================================================+
|                    ARCHITECTURE HBnB                              |
+===================================================================+
|                                                                   |
|   Client (Navigateur / Postman / curl / Frontend JS)              |
|       |                                                           |
|       |  Requetes HTTP (GET, POST, PUT, DELETE)                   |
|       v                                                           |
|   +-------------------------------------------------------+      |
|   |            COUCHE PRESENTATION (API Layer)             |      |
|   |   Flask / FastAPI                                      |      |
|   |   Routes, serialisation JSON, validation des entrees   |      |
|   |   Authentification (JWT)                                |      |
|   +-------------------------------------------------------+      |
|       |                                                           |
|       |  Appels au Service / Facade                               |
|       v                                                           |
|   +-------------------------------------------------------+      |
|   |            COUCHE LOGIQUE METIER (Business Logic)      |      |
|   |   Services / Facade                                    |      |
|   |   Regles metier, validations, orchestration            |      |
|   |   Ex: "un utilisateur ne peut pas noter son propre     |      |
|   |        hebergement"                                     |      |
|   +-------------------------------------------------------+      |
|       |                                                           |
|       |  Appels au Repository                                     |
|       v                                                           |
|   +-------------------------------------------------------+      |
|   |            COUCHE PERSISTANCE (Data Layer)             |      |
|   |   Repository Pattern                                   |      |
|   |   SQLAlchemy ORM / Base de donnees                     |      |
|   |   In-Memory (dev) ou PostgreSQL/SQLite (prod)          |      |
|   +-------------------------------------------------------+      |
|       |                                                           |
|       v                                                           |
|   [Base de Donnees : PostgreSQL / SQLite]                         |
|                                                                   |
+===================================================================+
```

---

## 2. Architecture en Couches

### 2.1 Pourquoi une architecture en couches ?

L'architecture en couches est un **patron d'architecture logicielle** qui separe l'application en niveaux de responsabilite. Chaque couche ne connait que la couche immediatement en dessous.

**Avantages :**

| Avantage | Explication |
|----------|-------------|
| **Separation des responsabilites** | Chaque couche fait UNE chose. L'API ne sait pas comment les donnees sont stockees. La BDD ne sait pas comment les donnees sont presentees. |
| **Testabilite** | On peut tester chaque couche independamment. Les tests unitaires de la logique metier n'ont pas besoin d'une BDD reelle. |
| **Maintenabilite** | Changer de BDD (SQLite → PostgreSQL) n'affecte que la couche persistance. L'API et la logique metier restent identiques. |
| **Reutilisabilite** | La logique metier peut servir a une API REST, une CLI, une interface graphique, etc. |
| **Travail en equipe** | Un membre peut travailler sur l'API pendant qu'un autre travaille sur la persistance. |

### 2.2 Le Pattern Facade

Le **Facade** est un patron de conception (design pattern) qui fournit une **interface simplifiee** a un ensemble de sous-systemes complexes. Dans HBnB, la Facade est le point d'entree unique entre la couche API et la couche logique metier.

```
+-------------------------------------------------------------------+
|                   PATTERN FACADE                                  |
+-------------------------------------------------------------------+
|                                                                   |
|   SANS Facade :                                                   |
|                                                                   |
|   API Route /users  ---> UserService                              |
|   API Route /places ---> PlaceService ---> UserService            |
|   API Route /reviews --> ReviewService --> PlaceService            |
|                          --> UserService                           |
|                                                                   |
|   L'API doit connaitre TOUS les services et les coordonner.       |
|   C'est complexe et fragile.                                      |
|                                                                   |
|   AVEC Facade :                                                   |
|                                                                   |
|   API Route /users  ---> HBnBFacade.create_user(data)             |
|   API Route /places ---> HBnBFacade.create_place(data)            |
|   API Route /reviews --> HBnBFacade.create_review(data)           |
|                                                                   |
|   L'API appelle UNE seule interface. La Facade coordonne          |
|   les services en interne.                                        |
|                                                                   |
+-------------------------------------------------------------------+
```

```python
class HBnBFacade:
    """
    Point d'entree unique pour toute la logique metier.
    Coordonne les differents repositories et applique les regles metier.
    """

    def __init__(self, user_repo, place_repo, review_repo, amenity_repo):
        self.user_repo = user_repo
        self.place_repo = place_repo
        self.review_repo = review_repo
        self.amenity_repo = amenity_repo

    # --- Utilisateurs ---
    def create_user(self, user_data: dict) -> 'User':
        """Cree un utilisateur apres validation."""
        # Verifier que l'email n'existe pas deja
        existing = self.user_repo.get_by_email(user_data['email'])
        if existing:
            raise ValueError("Email already registered")

        user = User(**user_data)
        user.hash_password(user_data['password'])
        self.user_repo.add(user)
        return user

    def get_user(self, user_id: str) -> 'User':
        """Recupere un utilisateur par son ID."""
        user = self.user_repo.get(user_id)
        if not user:
            raise ValueError("User not found")
        return user

    # --- Places ---
    def create_place(self, place_data: dict) -> 'Place':
        """Cree un hebergement. Verifie que le proprietaire existe."""
        owner = self.user_repo.get(place_data['owner_id'])
        if not owner:
            raise ValueError("Owner not found")

        place = Place(**place_data)
        self.place_repo.add(place)
        return place

    # --- Reviews ---
    def create_review(self, review_data: dict) -> 'Review':
        """Cree un avis. Verifie les regles metier."""
        user = self.user_repo.get(review_data['user_id'])
        place = self.place_repo.get(review_data['place_id'])

        if not user or not place:
            raise ValueError("User or Place not found")

        # Regle metier : on ne peut pas noter son propre hebergement
        if place.owner_id == user.id:
            raise ValueError("Cannot review your own place")

        review = Review(**review_data)
        self.review_repo.add(review)
        return review
```

> [!tip] Facade et injection de dependances
> La Facade recoit ses repositories en parametre (injection de dependances). Cela signifie qu'on peut lui passer un `InMemoryUserRepo` en dev et un `SQLUserRepo` en production, sans changer une seule ligne de code dans la Facade.

---

## 3. Le Modele de Donnees

### 3.1 Diagramme Entite-Relation

```
+===================================================================+
|               DIAGRAMME ENTITE-RELATION (ERD)                     |
+===================================================================+
|                                                                   |
|   +----------+       +----------+       +----------+              |
|   |   USER   |       |  PLACE   |       |  REVIEW  |              |
|   +----------+       +----------+       +----------+              |
|   | id (PK)  |<--+   | id (PK)  |<--+   | id (PK)  |             |
|   | email    |   |   | title    |   |   | text     |              |
|   | password |   +---| owner_id |   +---| place_id |              |
|   | first_   |   |   | descript.|       | user_id  |--+           |
|   |   name   |   |   | price    |       | rating   |  |           |
|   | last_    |   |   | latitude |       | created  |  |           |
|   |   name   |   |   | longitude|       | updated  |  |           |
|   | is_admin |   |   | created  |       +----------+  |           |
|   | created  |   |   | updated  |                      |           |
|   | updated  |   |   +----------+                      |           |
|   +----------+   |       |                             |           |
|        ^         |       | M:N                         |           |
|        |         |       |                             |           |
|        +---------+  +----+------+                      |           |
|                     | PLACE_    |                      |           |
|                     | AMENITY   |                      |           |
|                     +-----------+                      |           |
|                     | place_id  |                      |           |
|                     | amenity_id|                      |           |
|                     +-----------+                      |           |
|                          |                             |           |
|                     +----------+                       |           |
|                     | AMENITY  |                       |           |
|                     +----------+                       |           |
|                     | id (PK)  |                       |           |
|                     | name     |                       |           |
|                     | created  |                       |           |
|                     | updated  |                       |           |
|                     +----------+                       |           |
|                                                        |           |
|   Relations :                                          |           |
|   - User 1---N Place    (un user possede N places)     |           |
|   - User 1---N Review   (un user ecrit N reviews) -----+           |
|   - Place 1---N Review  (une place recoit N reviews)              |
|   - Place M---N Amenity (via table d'association)                 |
|                                                                   |
+===================================================================+
```

### 3.2 BaseModel -- La classe parente

Toutes les entites du projet heritent d'un `BaseModel` qui fournit les attributs communs : un identifiant unique (UUID), et les timestamps de creation et de mise a jour.

```python
import uuid
from datetime import datetime


class BaseModel:
    """
    Classe de base pour toutes les entites du projet.
    Fournit : id (UUID), created_at, updated_at.
    """

    def __init__(self, **kwargs):
        """
        Initialise une nouvelle entite.
        Si 'id' est fourni dans kwargs, on l'utilise (utile pour
        recharger depuis la BDD). Sinon, on genere un nouvel UUID.
        """
        self.id = kwargs.get('id', str(uuid.uuid4()))
        self.created_at = kwargs.get('created_at', datetime.utcnow())
        self.updated_at = kwargs.get('updated_at', datetime.utcnow())

    def save(self):
        """Met a jour le timestamp updated_at."""
        self.updated_at = datetime.utcnow()

    def to_dict(self):
        """
        Convertit l'objet en dictionnaire pour la serialisation JSON.
        Les dates sont converties en format ISO 8601.
        """
        result = self.__dict__.copy()
        result['created_at'] = self.created_at.isoformat()
        result['updated_at'] = self.updated_at.isoformat()
        return result
```

> [!info] Pourquoi UUID plutot qu'un auto-increment ?
> Avec un auto-increment (1, 2, 3...), un attaquant peut deviner les IDs des autres ressources. Avec un UUID (`550e8400-e29b-41d4-a716-446655440000`), c'est impossible. De plus, les UUID sont uniques **globalement**, ce qui facilite la fusion de bases de donnees ou les systemes distribues.

### 3.3 User -- L'utilisateur

```python
import bcrypt
import re
from app.models.base_model import BaseModel


class User(BaseModel):
    """
    Represente un utilisateur de la plateforme.
    
    Attributs:
        email: adresse email unique
        password_hash: hash bcrypt du mot de passe (jamais le mdp en clair)
        first_name: prenom
        last_name: nom de famille
        is_admin: True si administrateur
    """

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.email = kwargs.get('email', '')
        self.password_hash = kwargs.get('password_hash', '')
        self.first_name = kwargs.get('first_name', '')
        self.last_name = kwargs.get('last_name', '')
        self.is_admin = kwargs.get('is_admin', False)

        self.validate()

    def validate(self):
        """Valide les donnees de l'utilisateur."""
        if not self.email or not self._is_valid_email(self.email):
            raise ValueError("Invalid email address")
        if not self.first_name or len(self.first_name) > 50:
            raise ValueError("First name is required (max 50 chars)")
        if not self.last_name or len(self.last_name) > 50:
            raise ValueError("Last name is required (max 50 chars)")

    @staticmethod
    def _is_valid_email(email: str) -> bool:
        """Verifie le format de l'email avec une regex simple."""
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return re.match(pattern, email) is not None

    def hash_password(self, password: str):
        """
        Hash le mot de passe avec bcrypt.
        bcrypt genere automatiquement un salt unique.
        """
        if len(password) < 8:
            raise ValueError("Password must be at least 8 characters")
        self.password_hash = bcrypt.hashpw(
            password.encode('utf-8'),
            bcrypt.gensalt()
        ).decode('utf-8')

    def check_password(self, password: str) -> bool:
        """Verifie si le mot de passe correspond au hash stocke."""
        return bcrypt.checkpw(
            password.encode('utf-8'),
            self.password_hash.encode('utf-8')
        )

    def to_dict(self):
        """Surcharge pour exclure le password_hash de la serialisation."""
        result = super().to_dict()
        del result['password_hash']  # NE JAMAIS exposer le hash
        return result
```

> [!danger] Securite des mots de passe
> On ne stocke **jamais** un mot de passe en clair. On stocke un **hash** (empreinte irreversible). `bcrypt` est l'algorithme recommande car il est volontairement lent (resistant au brute-force) et inclut un **salt** (valeur aleatoire) pour empecher les attaques par rainbow table.

### 3.4 Place -- L'hebergement

```python
from app.models.base_model import BaseModel


class Place(BaseModel):
    """
    Represente un hebergement sur la plateforme.
    
    Attributs:
        title: titre de l'annonce
        description: description detaillee
        price: prix par nuit (en dollars/euros)
        latitude: coordonnee GPS (-90 a 90)
        longitude: coordonnee GPS (-180 a 180)
        owner_id: UUID de l'utilisateur proprietaire (FK)
    """

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.title = kwargs.get('title', '')
        self.description = kwargs.get('description', '')
        self.price = kwargs.get('price', 0.0)
        self.latitude = kwargs.get('latitude', 0.0)
        self.longitude = kwargs.get('longitude', 0.0)
        self.owner_id = kwargs.get('owner_id', '')
        self.amenity_ids = kwargs.get('amenity_ids', [])

        self.validate()

    def validate(self):
        """Valide les donnees de l'hebergement."""
        if not self.title or len(self.title) > 100:
            raise ValueError("Title is required (max 100 chars)")
        if self.price < 0:
            raise ValueError("Price must be a positive number")
        if not (-90 <= self.latitude <= 90):
            raise ValueError("Latitude must be between -90 and 90")
        if not (-180 <= self.longitude <= 180):
            raise ValueError("Longitude must be between -180 and 180")
        if not self.owner_id:
            raise ValueError("Owner ID is required")
```

### 3.5 Review -- L'avis

```python
from app.models.base_model import BaseModel


class Review(BaseModel):
    """
    Represente un avis laisse par un utilisateur sur un hebergement.
    
    Attributs:
        text: contenu de l'avis
        rating: note de 1 a 5
        user_id: UUID de l'auteur (FK)
        place_id: UUID de l'hebergement (FK)
    """

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.text = kwargs.get('text', '')
        self.rating = kwargs.get('rating', 0)
        self.user_id = kwargs.get('user_id', '')
        self.place_id = kwargs.get('place_id', '')

        self.validate()

    def validate(self):
        """Valide les donnees de l'avis."""
        if not self.text:
            raise ValueError("Review text is required")
        if not (1 <= self.rating <= 5):
            raise ValueError("Rating must be between 1 and 5")
        if not self.user_id:
            raise ValueError("User ID is required")
        if not self.place_id:
            raise ValueError("Place ID is required")
```

### 3.6 Amenity -- L'equipement

```python
from app.models.base_model import BaseModel


class Amenity(BaseModel):
    """
    Represente un equipement disponible dans un hebergement.
    Exemples : Wi-Fi, Piscine, Parking, Climatisation.
    
    Attributs:
        name: nom de l'equipement (unique)
    """

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.name = kwargs.get('name', '')

        self.validate()

    def validate(self):
        """Valide les donnees de l'equipement."""
        if not self.name or len(self.name) > 50:
            raise ValueError("Amenity name is required (max 50 chars)")
```

### 3.7 Modeles SQLAlchemy avec Relations

Quand on passe a la persistance en base de donnees, on transforme les classes ci-dessus en modeles SQLAlchemy. Voici l'implementation complete avec les relations :

```python
from app.extensions import db  # db = SQLAlchemy()
import uuid
from datetime import datetime


# --- Table d'association pour la relation Many-to-Many ---
place_amenity = db.Table(
    'place_amenity',
    db.Column('place_id', db.String(36), db.ForeignKey('places.id'),
              primary_key=True),
    db.Column('amenity_id', db.String(36), db.ForeignKey('amenities.id'),
              primary_key=True)
)


class User(db.Model):
    """Modele SQLAlchemy pour les utilisateurs."""
    __tablename__ = 'users'

    id = db.Column(db.String(36), primary_key=True,
                   default=lambda: str(uuid.uuid4()))
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(128), nullable=False)
    first_name = db.Column(db.String(50), nullable=False)
    last_name = db.Column(db.String(50), nullable=False)
    is_admin = db.Column(db.Boolean, default=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow,
                           onupdate=datetime.utcnow)

    # Relations
    places = db.relationship('Place', backref='owner', lazy=True,
                             cascade='all, delete-orphan')
    reviews = db.relationship('Review', backref='author', lazy=True,
                              cascade='all, delete-orphan')


class Place(db.Model):
    """Modele SQLAlchemy pour les hebergements."""
    __tablename__ = 'places'

    id = db.Column(db.String(36), primary_key=True,
                   default=lambda: str(uuid.uuid4()))
    title = db.Column(db.String(100), nullable=False)
    description = db.Column(db.Text, nullable=True)
    price = db.Column(db.Float, nullable=False)
    latitude = db.Column(db.Float, nullable=False)
    longitude = db.Column(db.Float, nullable=False)
    owner_id = db.Column(db.String(36), db.ForeignKey('users.id'),
                         nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow,
                           onupdate=datetime.utcnow)

    # Relations
    reviews = db.relationship('Review', backref='place', lazy=True,
                              cascade='all, delete-orphan')
    amenities = db.relationship('Amenity', secondary=place_amenity,
                                backref=db.backref('places', lazy=True))


class Review(db.Model):
    """Modele SQLAlchemy pour les avis."""
    __tablename__ = 'reviews'

    id = db.Column(db.String(36), primary_key=True,
                   default=lambda: str(uuid.uuid4()))
    text = db.Column(db.Text, nullable=False)
    rating = db.Column(db.Integer, nullable=False)
    user_id = db.Column(db.String(36), db.ForeignKey('users.id'),
                        nullable=False)
    place_id = db.Column(db.String(36), db.ForeignKey('places.id'),
                         nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow,
                           onupdate=datetime.utcnow)


class Amenity(db.Model):
    """Modele SQLAlchemy pour les equipements."""
    __tablename__ = 'amenities'

    id = db.Column(db.String(36), primary_key=True,
                   default=lambda: str(uuid.uuid4()))
    name = db.Column(db.String(50), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow,
                           onupdate=datetime.utcnow)
```

> [!info] `backref` et `lazy`
> - `backref='owner'` cree automatiquement un attribut `owner` sur l'objet `Place`, permettant d'acceder au `User` proprietaire sans requete manuelle.
> - `lazy=True` signifie que les relations sont chargees a la demande (quand on accede a l'attribut), pas automatiquement a chaque requete.
> - `cascade='all, delete-orphan'` supprime automatiquement les reviews/places quand l'utilisateur est supprime.

### 3.8 Regles de Validation par Entite

| Entite | Champ | Regle |
|--------|-------|-------|
| User | email | Requis, format email valide, unique |
| User | password | Min 8 caracteres, stocke en hash bcrypt |
| User | first_name | Requis, max 50 caracteres |
| User | last_name | Requis, max 50 caracteres |
| Place | title | Requis, max 100 caracteres |
| Place | price | Requis, nombre positif |
| Place | latitude | Entre -90 et 90 |
| Place | longitude | Entre -180 et 180 |
| Place | owner_id | Requis, doit referencer un User existant |
| Review | text | Requis |
| Review | rating | Entier entre 1 et 5 |
| Review | user_id | Requis, doit referencer un User existant |
| Review | place_id | Requis, doit referencer une Place existante |
| Amenity | name | Requis, max 50 caracteres, unique |

---

## 4. L'API REST -- Endpoints

### 4.1 Table Complete des Endpoints

```
+===================================================================+
|                    ENDPOINTS API REST                              |
+===================================================================+
|                                                                   |
|  USERS                                                            |
|  ------                                                           |
|  POST   /api/v1/users           Creer un utilisateur              |
|  GET    /api/v1/users           Lister tous les utilisateurs       |
|  GET    /api/v1/users/<id>      Recuperer un utilisateur           |
|  PUT    /api/v1/users/<id>      Modifier un utilisateur            |
|  DELETE /api/v1/users/<id>      Supprimer un utilisateur (admin)   |
|                                                                   |
|  PLACES                                                           |
|  -------                                                          |
|  POST   /api/v1/places          Creer un hebergement              |
|  GET    /api/v1/places          Lister (avec filtres optionnels)   |
|  GET    /api/v1/places/<id>     Recuperer un hebergement           |
|  PUT    /api/v1/places/<id>     Modifier (proprietaire seul)       |
|  DELETE /api/v1/places/<id>     Supprimer (proprietaire seul)      |
|                                                                   |
|  REVIEWS                                                          |
|  --------                                                         |
|  POST   /api/v1/places/<id>/reviews  Creer un avis                |
|  GET    /api/v1/places/<id>/reviews  Lister les avis d'une place  |
|  GET    /api/v1/reviews/<id>         Recuperer un avis             |
|  PUT    /api/v1/reviews/<id>         Modifier un avis              |
|  DELETE /api/v1/reviews/<id>         Supprimer un avis             |
|                                                                   |
|  AMENITIES                                                        |
|  ----------                                                       |
|  POST   /api/v1/amenities       Creer un equipement (admin)       |
|  GET    /api/v1/amenities       Lister tous les equipements        |
|  GET    /api/v1/amenities/<id>  Recuperer un equipement            |
|  PUT    /api/v1/amenities/<id>  Modifier (admin)                   |
|  DELETE /api/v1/amenities/<id>  Supprimer (admin)                  |
|                                                                   |
|  AUTH                                                             |
|  -----                                                            |
|  POST   /api/v1/auth/login      Connexion (retourne JWT)          |
|  POST   /api/v1/auth/register   Inscription                       |
|                                                                   |
+===================================================================+
```

### 4.2 Exemples Requete/Reponse

#### Creer un utilisateur

```
POST /api/v1/users
Content-Type: application/json

{
    "email": "john@example.com",
    "password": "securepass123",
    "first_name": "John",
    "last_name": "Doe"
}
```

```
201 Created
Content-Type: application/json

{
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "john@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "is_admin": false,
    "created_at": "2025-06-15T10:30:00",
    "updated_at": "2025-06-15T10:30:00"
}
```

#### Lister les utilisateurs

```
GET /api/v1/users
Authorization: Bearer <jwt_token>
```

```
200 OK
Content-Type: application/json

[
    {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "email": "john@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "is_admin": false
    },
    {
        "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
        "email": "jane@example.com",
        "first_name": "Jane",
        "last_name": "Smith",
        "is_admin": true
    }
]
```

#### Creer un hebergement

```
POST /api/v1/places
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
    "title": "Appartement Paris Centre",
    "description": "Bel appartement avec vue sur la Seine",
    "price": 120.00,
    "latitude": 48.8566,
    "longitude": 2.3522,
    "amenity_ids": ["wifi-uuid", "parking-uuid"]
}
```

```
201 Created
Content-Type: application/json

{
    "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "title": "Appartement Paris Centre",
    "description": "Bel appartement avec vue sur la Seine",
    "price": 120.00,
    "latitude": 48.8566,
    "longitude": 2.3522,
    "owner_id": "550e8400-e29b-41d4-a716-446655440000",
    "amenity_ids": ["wifi-uuid", "parking-uuid"],
    "created_at": "2025-06-15T11:00:00",
    "updated_at": "2025-06-15T11:00:00"
}
```

#### Creer un avis

```
POST /api/v1/places/7c9e6679-7425-40de-944b-e07fc1f90ae7/reviews
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
    "text": "Super appartement, tres bien situe !",
    "rating": 5
}
```

```
201 Created
Content-Type: application/json

{
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "text": "Super appartement, tres bien situe !",
    "rating": 5,
    "user_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "place_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "created_at": "2025-06-15T12:00:00",
    "updated_at": "2025-06-15T12:00:00"
}
```

#### Lister les equipements

```
GET /api/v1/amenities
```

```
200 OK
Content-Type: application/json

[
    {"id": "wifi-uuid", "name": "Wi-Fi"},
    {"id": "pool-uuid", "name": "Piscine"},
    {"id": "parking-uuid", "name": "Parking"},
    {"id": "ac-uuid", "name": "Climatisation"}
]
```

### 4.3 Codes de Statut HTTP

| Code | Signification | Utilisation |
|------|---------------|-------------|
| `200 OK` | Succes | GET, PUT reussis |
| `201 Created` | Ressource creee | POST reussi |
| `204 No Content` | Succes sans body | DELETE reussi |
| `400 Bad Request` | Donnees invalides | Validation echouee, JSON malformed |
| `401 Unauthorized` | Non authentifie | Token JWT absent ou expire |
| `403 Forbidden` | Non autorise | Action interdite (pas proprietaire, pas admin) |
| `404 Not Found` | Ressource introuvable | ID inexistant |
| `409 Conflict` | Conflit | Email deja utilise, doublon |
| `500 Internal Server Error` | Erreur serveur | Bug non gere |

### 4.4 Gestion des Erreurs

Toutes les erreurs retournent un JSON uniforme :

```json
{
    "error": "Description lisible de l'erreur",
    "code": 400
}
```

### 4.5 Implementation avec Flask (Blueprint)

```python
from flask import Blueprint, request, jsonify
from app.services.facade import HBnBFacade

users_bp = Blueprint('users', __name__)

# La facade est injectee via l'application factory
facade: HBnBFacade = None  # initialisee dans create_app()


def init_users_blueprint(app_facade: HBnBFacade):
    """Injecte la facade dans le blueprint."""
    global facade
    facade = app_facade


@users_bp.route('/api/v1/users', methods=['POST'])
def create_user():
    """Cree un nouvel utilisateur."""
    data = request.get_json()

    if not data:
        return jsonify({"error": "Invalid JSON"}), 400

    required = ['email', 'password', 'first_name', 'last_name']
    for field in required:
        if field not in data:
            return jsonify({"error": f"Missing field: {field}"}), 400

    try:
        user = facade.create_user(data)
        return jsonify(user.to_dict()), 201
    except ValueError as e:
        return jsonify({"error": str(e)}), 400


@users_bp.route('/api/v1/users', methods=['GET'])
def get_users():
    """Liste tous les utilisateurs."""
    users = facade.get_all_users()
    return jsonify([u.to_dict() for u in users]), 200


@users_bp.route('/api/v1/users/<user_id>', methods=['GET'])
def get_user(user_id):
    """Recupere un utilisateur par son ID."""
    try:
        user = facade.get_user(user_id)
        return jsonify(user.to_dict()), 200
    except ValueError:
        return jsonify({"error": "User not found"}), 404


@users_bp.route('/api/v1/users/<user_id>', methods=['PUT'])
def update_user(user_id):
    """Met a jour un utilisateur."""
    data = request.get_json()

    if not data:
        return jsonify({"error": "Invalid JSON"}), 400

    try:
        user = facade.update_user(user_id, data)
        return jsonify(user.to_dict()), 200
    except ValueError as e:
        return jsonify({"error": str(e)}), 400
```

### 4.6 Implementation avec FastAPI (Router)

```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel, EmailStr
from typing import List, Optional
from app.services.facade import HBnBFacade

router = APIRouter(prefix="/api/v1/users", tags=["users"])

# La facade est injectee via dependency injection
facade: HBnBFacade = None


# --- Schemas Pydantic (validation automatique) ---

class UserCreate(BaseModel):
    """Schema pour la creation d'un utilisateur."""
    email: EmailStr
    password: str
    first_name: str
    last_name: str

class UserResponse(BaseModel):
    """Schema pour la reponse (sans password_hash)."""
    id: str
    email: str
    first_name: str
    last_name: str
    is_admin: bool
    created_at: str
    updated_at: str

class UserUpdate(BaseModel):
    """Schema pour la mise a jour (champs optionnels)."""
    first_name: Optional[str] = None
    last_name: Optional[str] = None
    email: Optional[EmailStr] = None


# --- Routes ---

@router.post("/", response_model=UserResponse, status_code=201)
def create_user(user_data: UserCreate):
    """Cree un nouvel utilisateur."""
    try:
        user = facade.create_user(user_data.model_dump())
        return user.to_dict()
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))


@router.get("/", response_model=List[UserResponse])
def get_users():
    """Liste tous les utilisateurs."""
    users = facade.get_all_users()
    return [u.to_dict() for u in users]


@router.get("/{user_id}", response_model=UserResponse)
def get_user(user_id: str):
    """Recupere un utilisateur par ID."""
    try:
        user = facade.get_user(user_id)
        return user.to_dict()
    except ValueError:
        raise HTTPException(status_code=404, detail="User not found")


@router.put("/{user_id}", response_model=UserResponse)
def update_user(user_id: str, user_data: UserUpdate):
    """Met a jour un utilisateur."""
    try:
        updates = user_data.model_dump(exclude_unset=True)
        user = facade.update_user(user_id, updates)
        return user.to_dict()
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
```

> [!tip] Flask vs FastAPI
> - **Flask** : plus simple, plus de liberte, documentation manuelle. Utilise des Blueprints pour organiser les routes.
> - **FastAPI** : validation automatique (Pydantic), documentation auto-generee (Swagger), typage Python natif. Utilise des Routers.
> Les deux sont valides pour ce projet. Choisis selon les instructions de ton trimestre.

---

## 5. Authentification et Autorisation

### 5.1 Hachage des mots de passe avec bcrypt

On ne stocke **jamais** un mot de passe en clair en base de donnees. On stocke un **hash** -- une empreinte irreversible. `bcrypt` est l'algorithme recommande pour le hachage de mots de passe.

```python
import bcrypt

# --- Hacher un mot de passe ---
password = "monMotDePasse123"
salt = bcrypt.gensalt()  # genere un salt unique aleatoire
hashed = bcrypt.hashpw(password.encode('utf-8'), salt)
print(hashed)
# b'$2b$12$LJ3m4ys3Gz6xNt5xPz0Nkeeq8XzK3VZ0...'

# --- Verifier un mot de passe ---
if bcrypt.checkpw("monMotDePasse123".encode('utf-8'), hashed):
    print("Mot de passe correct")
else:
    print("Mot de passe incorrect")
```

```
+-------------------------------------------------------------------+
|               FONCTIONNEMENT DE BCRYPT                            |
+-------------------------------------------------------------------+
|                                                                   |
|   Mot de passe : "monMotDePasse123"                               |
|                                                                   |
|   1. Generer un salt aleatoire :                                  |
|      $2b$12$LJ3m4ys3Gz6xNt5xPz0Nke                              |
|                                                                   |
|   2. Combiner salt + password et hasher :                         |
|      bcrypt(salt + password) --> hash                             |
|                                                                   |
|   3. Resultat stocke en BDD :                                     |
|      $2b$12$LJ3m4ys3Gz6xNt5xPz0Nkeeq8XzK3VZ0jkR3hQ5bN2mK       |
|      ^^^^  ^^                                                     |
|      algo  cout (12 = 2^12 = 4096 iterations)                    |
|                                                                   |
|   Pour verifier : on re-hash avec le meme salt                   |
|   (le salt est inclus dans le hash stocke)                        |
|   et on compare les deux hash.                                    |
|                                                                   |
+-------------------------------------------------------------------+
```

### 5.2 JWT (JSON Web Tokens)

Un JWT est un **token** (jeton) signe cryptographiquement qui prouve l'identite d'un utilisateur. Apres la connexion, le serveur genere un JWT et le client l'envoie dans chaque requete suivante.

```
+-------------------------------------------------------------------+
|               FLUX D'AUTHENTIFICATION JWT                         |
+-------------------------------------------------------------------+
|                                                                   |
|   1. Client envoie email + password                               |
|      POST /api/v1/auth/login                                      |
|      {"email": "john@ex.com", "password": "secret123"}            |
|                                                                   |
|   2. Serveur verifie les credentials                               |
|      - Trouve l'utilisateur par email                              |
|      - bcrypt.checkpw(password, stored_hash)                      |
|                                                                   |
|   3. Si OK : serveur genere un JWT                                |
|      jwt.encode({"sub": user_id, "is_admin": false,               |
|                  "exp": now + 1h}, SECRET_KEY)                    |
|                                                                   |
|   4. Client recoit le token                                        |
|      {"access_token": "eyJhbGciOiJIUzI1NiIs..."}                 |
|                                                                   |
|   5. Client inclut le token dans chaque requete                    |
|      GET /api/v1/places                                           |
|      Authorization: Bearer eyJhbGciOiJIUzI1NiIs...               |
|                                                                   |
|   6. Serveur decode le token et identifie l'utilisateur            |
|      payload = jwt.decode(token, SECRET_KEY)                      |
|      user_id = payload["sub"]                                     |
|                                                                   |
+-------------------------------------------------------------------+
```

#### Implementation du login

```python
import jwt
from datetime import datetime, timedelta
from flask import Blueprint, request, jsonify
from app.services.facade import HBnBFacade

auth_bp = Blueprint('auth', __name__)
SECRET_KEY = "your-secret-key-change-in-production"  # en vrai, depuis .env


@auth_bp.route('/api/v1/auth/login', methods=['POST'])
def login():
    """Authentifie un utilisateur et retourne un JWT."""
    data = request.get_json()

    if not data or 'email' not in data or 'password' not in data:
        return jsonify({"error": "Email and password required"}), 400

    # Chercher l'utilisateur
    user = facade.get_user_by_email(data['email'])
    if not user:
        return jsonify({"error": "Invalid credentials"}), 401

    # Verifier le mot de passe
    if not user.check_password(data['password']):
        return jsonify({"error": "Invalid credentials"}), 401

    # Generer le JWT
    payload = {
        "sub": user.id,                                    # subject (user ID)
        "is_admin": user.is_admin,                          # role
        "exp": datetime.utcnow() + timedelta(hours=1)      # expiration
    }
    token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")

    return jsonify({
        "access_token": token,
        "token_type": "bearer"
    }), 200
```

> [!warning] Ne jamais exposer SECRET_KEY
> La cle secrete doit etre stockee dans une variable d'environnement (`.env`), jamais dans le code source. Si quelqu'un connait la cle, il peut forger de faux tokens.

### 5.3 Proteger les routes (Decorateur/Middleware)

```python
from functools import wraps
from flask import request, jsonify
import jwt


def token_required(f):
    """
    Decorateur pour proteger une route.
    Verifie que le token JWT est present et valide.
    Injecte l'utilisateur courant dans la fonction.
    """
    @wraps(f)
    def decorated(*args, **kwargs):
        token = None

        # Le token est dans le header Authorization
        auth_header = request.headers.get('Authorization')
        if auth_header and auth_header.startswith('Bearer '):
            token = auth_header.split(' ')[1]

        if not token:
            return jsonify({"error": "Token is missing"}), 401

        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
            current_user_id = payload['sub']
            is_admin = payload.get('is_admin', False)
        except jwt.ExpiredSignatureError:
            return jsonify({"error": "Token has expired"}), 401
        except jwt.InvalidTokenError:
            return jsonify({"error": "Invalid token"}), 401

        # Injecter les infos utilisateur dans kwargs
        kwargs['current_user_id'] = current_user_id
        kwargs['is_admin'] = is_admin

        return f(*args, **kwargs)

    return decorated


def admin_required(f):
    """Decorateur pour les routes admin uniquement."""
    @wraps(f)
    @token_required
    def decorated(*args, **kwargs):
        if not kwargs.get('is_admin'):
            return jsonify({"error": "Admin access required"}), 403
        return f(*args, **kwargs)
    return decorated
```

#### Utilisation sur les routes

```python
@places_bp.route('/api/v1/places', methods=['POST'])
@token_required
def create_place(current_user_id, is_admin, **kwargs):
    """Cree un hebergement. L'utilisateur connecte est le proprietaire."""
    data = request.get_json()
    data['owner_id'] = current_user_id  # le proprietaire est l'user connecte

    try:
        place = facade.create_place(data)
        return jsonify(place.to_dict()), 201
    except ValueError as e:
        return jsonify({"error": str(e)}), 400


@places_bp.route('/api/v1/places/<place_id>', methods=['PUT'])
@token_required
def update_place(place_id, current_user_id, is_admin, **kwargs):
    """Modifie un hebergement. Seul le proprietaire ou un admin peut modifier."""
    place = facade.get_place(place_id)

    if not place:
        return jsonify({"error": "Place not found"}), 404

    # Verification : proprietaire ou admin ?
    if place.owner_id != current_user_id and not is_admin:
        return jsonify({"error": "Unauthorized"}), 403

    data = request.get_json()
    try:
        updated = facade.update_place(place_id, data)
        return jsonify(updated.to_dict()), 200
    except ValueError as e:
        return jsonify({"error": str(e)}), 400
```

### 5.4 Controle d'acces par role

```
+-------------------------------------------------------------------+
|               MATRICE D'AUTORISATION                              |
+-------------------------------------------------------------------+
|                                                                   |
|   Action                  | Anonyme | User  | Owner | Admin      |
|   ----------------------- | ------- | ----- | ----- | ---------- |
|   Voir les places         |   OUI   |  OUI  |  OUI  |   OUI     |
|   Creer une place         |   NON   |  OUI  |  OUI  |   OUI     |
|   Modifier une place      |   NON   |  NON  |  OUI  |   OUI     |
|   Supprimer une place     |   NON   |  NON  |  OUI  |   OUI     |
|   Poster un avis          |   NON   |  OUI  |  NON* |   OUI     |
|   Modifier son avis       |   NON   |  OUI  |  ---  |   OUI     |
|   Supprimer un avis       |   NON   |  NON  |  NON  |   OUI     |
|   Gerer les amenities     |   NON   |  NON  |  NON  |   OUI     |
|   Supprimer un user       |   NON   |  NON  |  NON  |   OUI     |
|                                                                   |
|   * Owner = proprietaire de la place, pas de l'avis.              |
|     Un proprietaire NE PEUT PAS noter son propre hebergement.     |
|                                                                   |
+-------------------------------------------------------------------+
```

---

## 6. La Couche de Persistance

### 6.1 Le Repository Pattern

Le **Repository Pattern** abstrait l'acces aux donnees derriere une interface. Le code metier ne sait pas si les donnees viennent d'un dictionnaire en memoire, d'une base SQLite, ou de PostgreSQL. Il interagit uniquement avec l'interface du repository.

```python
from abc import ABC, abstractmethod
from typing import List, Optional


class Repository(ABC):
    """
    Interface abstraite pour tous les repositories.
    Definit le contrat : quelles operations sont disponibles.
    """

    @abstractmethod
    def add(self, obj) -> None:
        """Ajoute un objet au stockage."""
        pass

    @abstractmethod
    def get(self, obj_id: str) -> Optional[object]:
        """Recupere un objet par son ID."""
        pass

    @abstractmethod
    def get_all(self) -> List[object]:
        """Recupere tous les objets."""
        pass

    @abstractmethod
    def update(self, obj_id: str, data: dict) -> Optional[object]:
        """Met a jour un objet."""
        pass

    @abstractmethod
    def delete(self, obj_id: str) -> bool:
        """Supprime un objet. Retourne True si supprime."""
        pass
```

### 6.2 Repository In-Memory (pour le dev et les tests)

```python
from typing import List, Optional
from app.persistence.repository import Repository


class InMemoryRepository(Repository):
    """
    Implementation in-memory du Repository.
    Stocke les objets dans un dictionnaire Python.
    Ideal pour le developpement et les tests unitaires.
    """

    def __init__(self):
        self._storage = {}  # {id: object}

    def add(self, obj) -> None:
        """Ajoute un objet au dictionnaire."""
        self._storage[obj.id] = obj

    def get(self, obj_id: str) -> Optional[object]:
        """Recupere un objet par ID."""
        return self._storage.get(obj_id)

    def get_all(self) -> List[object]:
        """Retourne tous les objets."""
        return list(self._storage.values())

    def update(self, obj_id: str, data: dict) -> Optional[object]:
        """Met a jour les attributs d'un objet."""
        obj = self._storage.get(obj_id)
        if obj is None:
            return None

        for key, value in data.items():
            if hasattr(obj, key) and key != 'id':
                setattr(obj, key, value)

        obj.save()  # met a jour updated_at
        return obj

    def delete(self, obj_id: str) -> bool:
        """Supprime un objet."""
        if obj_id in self._storage:
            del self._storage[obj_id]
            return True
        return False

    def get_by_attribute(self, attr: str, value) -> Optional[object]:
        """Recherche par attribut (utile pour email, name, etc.)."""
        for obj in self._storage.values():
            if getattr(obj, attr, None) == value:
                return obj
        return None
```

### 6.3 Repository SQL (SQLAlchemy)

```python
from typing import List, Optional
from app.persistence.repository import Repository
from app.extensions import db


class SQLRepository(Repository):
    """
    Implementation SQL du Repository avec SQLAlchemy.
    Chaque entite a son propre SQLRepository parametre
    par la classe du modele.
    """

    def __init__(self, model_class):
        """
        @model_class: la classe SQLAlchemy (User, Place, etc.)
        """
        self.model_class = model_class

    def add(self, obj) -> None:
        """Ajoute a la session et commit."""
        db.session.add(obj)
        db.session.commit()

    def get(self, obj_id: str) -> Optional[object]:
        """Recupere par cle primaire."""
        return self.model_class.query.get(obj_id)

    def get_all(self) -> List[object]:
        """Recupere tous les enregistrements."""
        return self.model_class.query.all()

    def update(self, obj_id: str, data: dict) -> Optional[object]:
        """Met a jour un enregistrement."""
        obj = self.get(obj_id)
        if obj is None:
            return None

        for key, value in data.items():
            if hasattr(obj, key) and key != 'id':
                setattr(obj, key, value)

        db.session.commit()
        return obj

    def delete(self, obj_id: str) -> bool:
        """Supprime un enregistrement."""
        obj = self.get(obj_id)
        if obj is None:
            return False

        db.session.delete(obj)
        db.session.commit()
        return True

    def get_by_attribute(self, attr: str, value) -> Optional[object]:
        """Recherche par attribut."""
        return self.model_class.query.filter(
            getattr(self.model_class, attr) == value
        ).first()
```

### 6.4 Passer d'une implementation a l'autre (Injection de Dependances)

C'est ici que la magie du Repository Pattern brille : on choisit l'implementation au demarrage de l'application, et le reste du code ne change pas.

```python
# app/__init__.py  ou  app/factory.py

from flask import Flask
from app.persistence.in_memory import InMemoryRepository
from app.persistence.sql_repository import SQLRepository
from app.models.sql_models import User, Place, Review, Amenity
from app.services.facade import HBnBFacade
from app.extensions import db
import os


def create_app(config_name='development'):
    """Application factory -- cree et configure l'app Flask."""
    app = Flask(__name__)

    if config_name == 'development':
        # --- Mode developpement : stockage in-memory ---
        user_repo = InMemoryRepository()
        place_repo = InMemoryRepository()
        review_repo = InMemoryRepository()
        amenity_repo = InMemoryRepository()

    elif config_name == 'production':
        # --- Mode production : stockage SQL ---
        app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv(
            'DATABASE_URL', 'sqlite:///hbnb.db'
        )
        db.init_app(app)

        with app.app_context():
            db.create_all()

        user_repo = SQLRepository(User)
        place_repo = SQLRepository(Place)
        review_repo = SQLRepository(Review)
        amenity_repo = SQLRepository(Amenity)

    # La Facade recoit les repositories -- elle ne sait pas
    # quelle implementation est utilisee.
    facade = HBnBFacade(user_repo, place_repo, review_repo, amenity_repo)

    # Enregistrer les blueprints
    from app.api.users import users_bp, init_users_blueprint
    init_users_blueprint(facade)
    app.register_blueprint(users_bp)

    # ... autres blueprints ...

    return app
```

> [!tip] Avantage cle
> En changeant **une seule variable** (`config_name`), on passe d'un stockage en memoire (rapide, sans BDD) a un stockage SQL complet. Les tests utilisent `development`, la production utilise `production`. Zero modification dans le code metier ou l'API.

### 6.5 Migrations avec Alembic

Quand le schema de la BDD change (nouvel attribut, nouvelle table), on utilise **Alembic** pour generer et appliquer des migrations incrementales.

```bash
# Installation
pip install flask-migrate  # wrapper Flask autour d'Alembic

# Initialisation (une seule fois)
flask db init

# Generer une migration apres modification des modeles
flask db migrate -m "Add phone_number to User"
# Alembic compare les modeles SQLAlchemy avec la BDD actuelle
# et genere un script de migration dans migrations/versions/

# Appliquer la migration
flask db upgrade

# Revenir en arriere
flask db downgrade
```

```python
# Dans app/__init__.py, ajouter :
from flask_migrate import Migrate

migrate = Migrate()

def create_app():
    app = Flask(__name__)
    db.init_app(app)
    migrate.init_app(app, db)  # <-- activer les migrations
    # ...
```

> [!warning] Ne jamais modifier la BDD manuellement en production
> Utilise toujours Alembic/Flask-Migrate pour les changements de schema. Les scripts de migration sont versiones (dans git) et peuvent etre rejoues sur n'importe quel environnement.

---

## 7. Le Frontend

### 7.1 Architecture du Frontend

Le frontend peut etre implemente de deux facons :

1. **Templates Jinja2** (server-side rendering) : le serveur Flask genere le HTML complet et l'envoie au navigateur.
2. **SPA (Single Page Application)** : un frontend JavaScript independant (vanilla JS, React, Vue) qui communique avec l'API via `fetch()`.

Pour ce projet, on combine souvent les deux : pages HTML statiques avec du JavaScript qui appelle l'API.

### 7.2 Pages Attendues

```
+-------------------------------------------------------------------+
|                   PAGES DU FRONTEND                               |
+-------------------------------------------------------------------+
|                                                                   |
|  PAGE D'ACCUEIL (index.html)                                      |
|  +-----------------------------------------------------+         |
|  |  [Logo HBnB]           [Login] [Register]           |         |
|  |                                                     |         |
|  |  Filtres : Prix [___] Equipements [v] Localisation  |         |
|  |                                                     |         |
|  |  +-------------+  +-------------+  +-------------+  |         |
|  |  | Appart Paris|  | Villa Nice  |  | Studio Lyon |  |         |
|  |  | 120 EUR/nuit|  | 250 EUR/nuit|  |  80 EUR/nuit|  |         |
|  |  | **** (4.2)  |  | ***** (4.8) |  | *** (3.5)   |  |         |
|  |  +-------------+  +-------------+  +-------------+  |         |
|  +-----------------------------------------------------+         |
|                                                                   |
|  PAGE DETAIL (place.html?id=...)                                   |
|  +-----------------------------------------------------+         |
|  |  Appartement Paris Centre                            |         |
|  |  Description : Bel appartement avec vue...           |         |
|  |  Prix : 120 EUR / nuit                               |         |
|  |  Equipements : Wi-Fi, Parking                        |         |
|  |  Proprietaire : John Doe                              |         |
|  |                                                     |         |
|  |  --- Avis ---                                        |         |
|  |  Jane S. : ***** "Super appartement !"               |         |
|  |  Bob M.  : ***   "Correct mais bruyant"              |         |
|  |                                                     |         |
|  |  [Laisser un avis] (si connecte)                     |         |
|  +-----------------------------------------------------+         |
|                                                                   |
|  PAGE LOGIN (login.html)                                          |
|  +-----------------------------------------------------+         |
|  |  Email    : [_______________]                        |         |
|  |  Password : [_______________]                        |         |
|  |  [Se connecter]                                      |         |
|  |  Pas de compte ? [S'inscrire]                        |         |
|  +-----------------------------------------------------+         |
|                                                                   |
+-------------------------------------------------------------------+
```

### 7.3 Appeler l'API depuis JavaScript

```javascript
// scripts/api.js

const API_URL = 'http://localhost:5000/api/v1';

/**
 * Recupere le token JWT stocke dans localStorage.
 */
function getToken() {
    return localStorage.getItem('jwt_token');
}

/**
 * Effectue une requete authentifiee vers l'API.
 * @param {string} endpoint - ex: '/places'
 * @param {string} method - GET, POST, PUT, DELETE
 * @param {object} body - corps de la requete (optionnel)
 * @returns {Promise<object>} - reponse JSON
 */
async function apiRequest(endpoint, method = 'GET', body = null) {
    const headers = {
        'Content-Type': 'application/json'
    };

    const token = getToken();
    if (token) {
        headers['Authorization'] = `Bearer ${token}`;
    }

    const options = {
        method: method,
        headers: headers
    };

    if (body && method !== 'GET') {
        options.body = JSON.stringify(body);
    }

    const response = await fetch(`${API_URL}${endpoint}`, options);

    if (!response.ok) {
        const error = await response.json();
        throw new Error(error.error || 'API request failed');
    }

    // DELETE retourne 204 No Content (pas de JSON)
    if (response.status === 204) return null;

    return response.json();
}


// --- Fonctions specifiques ---

async function login(email, password) {
    const data = await apiRequest('/auth/login', 'POST', { email, password });
    localStorage.setItem('jwt_token', data.access_token);
    return data;
}

async function getPlaces() {
    return apiRequest('/places');
}

async function getPlace(placeId) {
    return apiRequest(`/places/${placeId}`);
}

async function createReview(placeId, text, rating) {
    return apiRequest(`/places/${placeId}/reviews`, 'POST', { text, rating });
}

async function logout() {
    localStorage.removeItem('jwt_token');
    window.location.href = '/login.html';
}
```

### 7.4 Afficher les places sur la page d'accueil

```javascript
// scripts/index.js

document.addEventListener('DOMContentLoaded', async () => {
    const container = document.getElementById('places-container');

    try {
        const places = await getPlaces();

        places.forEach(place => {
            const card = document.createElement('div');
            card.className = 'place-card';
            card.innerHTML = `
                <h3>${place.title}</h3>
                <p>${place.description.substring(0, 100)}...</p>
                <p class="price">${place.price} EUR / nuit</p>
                <a href="place.html?id=${place.id}">Voir les details</a>
            `;
            container.appendChild(card);
        });

    } catch (error) {
        container.innerHTML = '<p>Erreur lors du chargement des places.</p>';
        console.error(error);
    }
});
```

### 7.5 Design Responsive

```css
/* styles/main.css */

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Arial, sans-serif;
    background-color: #f5f5f5;
    color: #333;
}

.places-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 20px;
    padding: 20px;
    max-width: 1200px;
    margin: 0 auto;
}

.place-card {
    background: white;
    border-radius: 12px;
    padding: 20px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    transition: transform 0.2s;
}

.place-card:hover {
    transform: translateY(-4px);
}

.price {
    font-size: 1.2em;
    font-weight: bold;
    color: #e63946;
    margin: 10px 0;
}

/* Responsive */
@media (max-width: 768px) {
    .places-grid {
        grid-template-columns: 1fr;
    }
}
```

---

## 8. Tests

### 8.1 Tests Unitaires des Modeles (pytest)

```python
# tests/test_models.py

import pytest
from app.models.user import User
from app.models.place import Place
from app.models.review import Review


class TestUser:
    """Tests unitaires pour le modele User."""

    def test_create_user_valid(self):
        """Un utilisateur avec des donnees valides doit etre cree."""
        user = User(
            email="test@example.com",
            first_name="Test",
            last_name="User"
        )
        assert user.email == "test@example.com"
        assert user.first_name == "Test"
        assert user.id is not None  # UUID genere automatiquement

    def test_create_user_invalid_email(self):
        """Un email invalide doit lever une erreur."""
        with pytest.raises(ValueError, match="Invalid email"):
            User(email="not-an-email", first_name="Test", last_name="User")

    def test_create_user_missing_name(self):
        """Un prenom manquant doit lever une erreur."""
        with pytest.raises(ValueError, match="First name"):
            User(email="test@example.com", first_name="", last_name="User")

    def test_password_hashing(self):
        """Le mot de passe doit etre correctement hashe et verifiable."""
        user = User(email="test@example.com", first_name="A", last_name="B")
        user.hash_password("securepass123")

        assert user.password_hash != "securepass123"  # pas en clair
        assert user.check_password("securepass123") is True
        assert user.check_password("wrongpass") is False

    def test_to_dict_excludes_password(self):
        """to_dict ne doit jamais inclure le password_hash."""
        user = User(email="t@e.com", first_name="A", last_name="B")
        user.hash_password("pass1234")
        d = user.to_dict()

        assert 'password_hash' not in d
        assert 'email' in d


class TestPlace:
    """Tests unitaires pour le modele Place."""

    def test_create_place_valid(self):
        place = Place(
            title="Test Place",
            price=100.0,
            latitude=48.85,
            longitude=2.35,
            owner_id="some-uuid"
        )
        assert place.title == "Test Place"
        assert place.price == 100.0

    def test_create_place_negative_price(self):
        with pytest.raises(ValueError, match="positive"):
            Place(title="T", price=-10, latitude=0, longitude=0,
                  owner_id="id")

    def test_create_place_invalid_latitude(self):
        with pytest.raises(ValueError, match="Latitude"):
            Place(title="T", price=10, latitude=100, longitude=0,
                  owner_id="id")


class TestReview:
    """Tests unitaires pour le modele Review."""

    def test_create_review_valid(self):
        review = Review(
            text="Great place!",
            rating=5,
            user_id="user-uuid",
            place_id="place-uuid"
        )
        assert review.rating == 5

    def test_create_review_invalid_rating(self):
        with pytest.raises(ValueError, match="Rating"):
            Review(text="Bad", rating=6, user_id="u", place_id="p")

    def test_create_review_empty_text(self):
        with pytest.raises(ValueError, match="text"):
            Review(text="", rating=3, user_id="u", place_id="p")
```

### 8.2 Tests d'Integration de l'API

```python
# tests/test_api.py

import pytest
import json
from app import create_app


@pytest.fixture
def client():
    """Cree un client de test avec un stockage in-memory."""
    app = create_app('development')
    app.config['TESTING'] = True

    with app.test_client() as client:
        yield client


@pytest.fixture
def auth_headers(client):
    """Cree un utilisateur et retourne les headers avec le JWT."""
    # Creer un utilisateur
    client.post('/api/v1/users', json={
        "email": "auth@test.com",
        "password": "testpass123",
        "first_name": "Auth",
        "last_name": "User"
    })

    # Se connecter
    response = client.post('/api/v1/auth/login', json={
        "email": "auth@test.com",
        "password": "testpass123"
    })
    token = json.loads(response.data)['access_token']

    return {"Authorization": f"Bearer {token}"}


class TestUsersAPI:
    """Tests d'integration pour les endpoints Users."""

    def test_create_user(self, client):
        """POST /api/v1/users doit creer un utilisateur."""
        response = client.post('/api/v1/users', json={
            "email": "new@test.com",
            "password": "testpass123",
            "first_name": "New",
            "last_name": "User"
        })
        assert response.status_code == 201
        data = json.loads(response.data)
        assert data['email'] == "new@test.com"
        assert 'id' in data
        assert 'password_hash' not in data  # securite

    def test_create_user_duplicate_email(self, client):
        """POST avec un email existant doit retourner 400."""
        user_data = {
            "email": "dup@test.com",
            "password": "testpass123",
            "first_name": "Dup",
            "last_name": "User"
        }
        client.post('/api/v1/users', json=user_data)
        response = client.post('/api/v1/users', json=user_data)
        assert response.status_code == 400

    def test_get_users(self, client):
        """GET /api/v1/users doit retourner la liste."""
        client.post('/api/v1/users', json={
            "email": "list@test.com",
            "password": "pass12345",
            "first_name": "List",
            "last_name": "User"
        })
        response = client.get('/api/v1/users')
        assert response.status_code == 200
        data = json.loads(response.data)
        assert isinstance(data, list)
        assert len(data) >= 1

    def test_get_user_not_found(self, client):
        """GET /api/v1/users/<bad_id> doit retourner 404."""
        response = client.get('/api/v1/users/nonexistent-id')
        assert response.status_code == 404


class TestPlacesAPI:
    """Tests d'integration pour les endpoints Places."""

    def test_create_place(self, client, auth_headers):
        """POST /api/v1/places doit creer un hebergement."""
        response = client.post('/api/v1/places',
            headers=auth_headers,
            json={
                "title": "Test Place",
                "description": "A test place",
                "price": 100.0,
                "latitude": 48.85,
                "longitude": 2.35
            })
        assert response.status_code == 201

    def test_create_place_unauthorized(self, client):
        """POST sans token doit retourner 401."""
        response = client.post('/api/v1/places', json={
            "title": "Test",
            "price": 50
        })
        assert response.status_code == 401
```

### 8.3 Configuration des Tests

```python
# tests/conftest.py

import pytest
from app import create_app


@pytest.fixture(scope='session')
def app():
    """Cree l'application en mode test (une seule fois pour la session)."""
    app = create_app('testing')
    return app


# pytest.ini ou pyproject.toml
# [tool.pytest.ini_options]
# testpaths = ["tests"]
# python_files = "test_*.py"
# python_classes = "Test*"
# python_functions = "test_*"
```

### 8.4 Lancer les Tests et Mesurer la Couverture

```bash
# Installer les dependances de test
pip install pytest pytest-cov

# Lancer tous les tests
pytest

# Lancer avec verbosity
pytest -v

# Lancer un fichier specifique
pytest tests/test_models.py

# Lancer avec couverture
pytest --cov=app --cov-report=html

# Le rapport HTML est genere dans htmlcov/index.html
# Objectif : couverture > 80%
```

---

## 9. Docker et Deploiement

### 9.1 Dockerfile pour l'API

```dockerfile
# Dockerfile

# Image de base Python
FROM python:3.10-slim

# Repertoire de travail dans le conteneur
WORKDIR /app

# Copier les dependances d'abord (pour profiter du cache Docker)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code source
COPY . .

# Variables d'environnement
ENV FLASK_APP=app
ENV FLASK_ENV=production
ENV PORT=5000

# Exposer le port
EXPOSE 5000

# Commande de demarrage avec gunicorn (serveur de production)
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:create_app('production')"]
```

> [!info] Pourquoi gunicorn et pas `flask run` ?
> `flask run` utilise le serveur de developpement Werkzeug, qui est mono-thread et non securise. `gunicorn` est un serveur WSGI de production : multi-worker, robuste, performant.

### 9.2 docker-compose.yml

```yaml
# docker-compose.yml

version: '3.8'

services:
  # --- API Backend ---
  api:
    build: .
    container_name: hbnb-api
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://hbnb:hbnb_password@db:5432/hbnb_db
      - SECRET_KEY=${SECRET_KEY:-change-me-in-production}
      - FLASK_ENV=production
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  # --- Base de Donnees PostgreSQL ---
  db:
    image: postgres:15-alpine
    container_name: hbnb-db
    environment:
      - POSTGRES_USER=hbnb
      - POSTGRES_PASSWORD=hbnb_password
      - POSTGRES_DB=hbnb_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U hbnb"]
      interval: 5s
      timeout: 5s
      retries: 5

  # --- Reverse Proxy Nginx ---
  nginx:
    image: nginx:alpine
    container_name: hbnb-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
      - ./frontend:/usr/share/nginx/html  # fichiers statiques
    depends_on:
      - api
    restart: unless-stopped

volumes:
  postgres_data:
```

### 9.3 Configuration Nginx

```nginx
# nginx/nginx.conf

server {
    listen 80;
    server_name localhost;

    # Fichiers statiques du frontend
    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }

    # Proxy vers l'API
    location /api/ {
        proxy_pass http://api:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 9.4 Fichier .env

```bash
# .env -- NE JAMAIS commiter ce fichier dans git !

SECRET_KEY=une-cle-longue-aleatoire-genere-avec-python-secrets
DATABASE_URL=postgresql://hbnb:hbnb_password@db:5432/hbnb_db
FLASK_ENV=production
```

### 9.5 Commandes Docker Essentielles

```bash
# Construire et lancer tous les services
docker compose up --build -d

# Voir les logs
docker compose logs -f api

# Executer les migrations dans le conteneur
docker compose exec api flask db upgrade

# Creer un admin
docker compose exec api flask create-admin

# Arreter tout
docker compose down

# Arreter et supprimer les volumes (ATTENTION: perte de donnees)
docker compose down -v

# Rebuilder un seul service
docker compose build api
docker compose up -d api
```

### 9.6 Considerations de Production

```
+-------------------------------------------------------------------+
|               CHECKLIST PRODUCTION                                |
+-------------------------------------------------------------------+
|                                                                   |
|  [ ] SECRET_KEY longue et aleatoire (pas "change-me")             |
|  [ ] HTTPS active (Let's Encrypt + Nginx)                         |
|  [ ] Mot de passe BDD fort (pas "hbnb_password")                  |
|  [ ] Variables d'environnement dans .env (pas dans le code)       |
|  [ ] .env dans .gitignore                                         |
|  [ ] Gunicorn avec workers = 2 * CPU + 1                          |
|  [ ] Logs configures (fichier + rotation)                         |
|  [ ] Sauvegardes BDD automatisees (pg_dump + cron)                |
|  [ ] Rate limiting sur les endpoints sensibles (login)            |
|  [ ] CORS configure correctement                                  |
|  [ ] Pas de mode debug en production                              |
|  [ ] Images Docker minimales (slim/alpine)                         |
|                                                                   |
+-------------------------------------------------------------------+
```

---

## 10. Organisation du Travail en Equipe

### 10.1 Workflow Git

```
+-------------------------------------------------------------------+
|               GIT WORKFLOW                                        |
+-------------------------------------------------------------------+
|                                                                   |
|   main (ou master)                                                |
|     |                                                             |
|     |--- develop                                                  |
|            |                                                      |
|            |--- feature/user-model        (Membre A)              |
|            |--- feature/place-api         (Membre B)              |
|            |--- feature/auth-jwt          (Membre C)              |
|            |--- feature/frontend-places   (Membre D)              |
|            |                                                      |
|            |<-- merge via Pull Request (apres code review)        |
|            |                                                      |
|     |<---- merge develop -> main (quand stable)                   |
|                                                                   |
+-------------------------------------------------------------------+
```

**Regles :**
1. **Jamais** de commit direct sur `main` ou `develop`
2. Chaque feature dans une branche dediee (`feature/nom-descriptif`)
3. Pull Request obligatoire avec au moins 1 review
4. Les tests doivent passer avant le merge
5. Messages de commit clairs et descriptifs

```bash
# Workflow quotidien
git checkout develop
git pull origin develop
git checkout -b feature/user-model

# ... travailler ...

git add -A
git commit -m "feat: add User model with validation and password hashing"
git push origin feature/user-model

# Creer une Pull Request sur GitHub
# Apres review et approbation, merge dans develop
```

### 10.2 Structure du Projet Recommandee

```
hbnb/
├── app/
│   ├── __init__.py              # Application factory (create_app)
│   ├── extensions.py            # db = SQLAlchemy(), etc.
│   │
│   ├── models/                  # Couche Donnees
│   │   ├── __init__.py
│   │   ├── base_model.py        # BaseModel (id, timestamps)
│   │   ├── user.py              # Classe User
│   │   ├── place.py             # Classe Place
│   │   ├── review.py            # Classe Review
│   │   ├── amenity.py           # Classe Amenity
│   │   └── sql_models.py        # Modeles SQLAlchemy
│   │
│   ├── services/                # Couche Logique Metier
│   │   ├── __init__.py
│   │   └── facade.py            # HBnBFacade
│   │
│   ├── persistence/             # Couche Persistance
│   │   ├── __init__.py
│   │   ├── repository.py        # Interface abstraite
│   │   ├── in_memory.py         # Implementation in-memory
│   │   └── sql_repository.py    # Implementation SQL
│   │
│   └── api/                     # Couche Presentation
│       ├── __init__.py
│       ├── users.py             # Routes /users
│       ├── places.py            # Routes /places
│       ├── reviews.py           # Routes /reviews
│       ├── amenities.py         # Routes /amenities
│       └── auth.py              # Routes /auth
│
├── frontend/                    # Fichiers statiques
│   ├── index.html
│   ├── login.html
│   ├── place.html
│   ├── scripts/
│   │   ├── api.js
│   │   └── index.js
│   └── styles/
│       └── main.css
│
├── tests/                       # Tests
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_models.py
│   └── test_api.py
│
├── migrations/                  # Alembic (genere automatiquement)
├── nginx/
│   └── nginx.conf
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env.example                 # Template .env (sans secrets)
├── .gitignore
└── README.md
```

### 10.3 Division des Taches

Pour une equipe de 3-4 personnes, voici une repartition possible :

```
+-------------------------------------------------------------------+
|               REPARTITION DES TACHES                              |
+-------------------------------------------------------------------+
|                                                                   |
|  PHASE 1 : Fondations (Semaine 1)                                 |
|  --------------------------------                                  |
|  Tous : definir le modele de donnees ensemble                      |
|  Membre A : BaseModel + User + tests                               |
|  Membre B : Place + Review + tests                                 |
|  Membre C : Amenity + Repository interface + InMemoryRepo          |
|                                                                   |
|  PHASE 2 : API (Semaine 2)                                        |
|  -------------------------                                         |
|  Membre A : Users API + Auth (JWT)                                 |
|  Membre B : Places API + Reviews API                               |
|  Membre C : Amenities API + Facade                                 |
|                                                                   |
|  PHASE 3 : Persistance + Frontend (Semaine 3)                     |
|  ----------------------------------------                          |
|  Membre A : SQLAlchemy models + SQL Repository                     |
|  Membre B : Frontend (HTML/CSS/JS)                                 |
|  Membre C : Docker + deployment                                    |
|                                                                   |
|  PHASE 4 : Integration + Polish (Semaine 4)                       |
|  ------------------------------------------                        |
|  Tous : integration, tests end-to-end, debugging, documentation   |
|                                                                   |
+-------------------------------------------------------------------+
```

### 10.4 requirements.txt

```
# Core
flask==3.0.0
flask-cors==4.0.0
gunicorn==21.2.0

# Database
flask-sqlalchemy==3.1.1
flask-migrate==4.0.5
psycopg2-binary==2.9.9

# Auth
pyjwt==2.8.0
bcrypt==4.1.2

# Dev/Test
pytest==7.4.4
pytest-cov==4.1.0
```

### 10.5 .gitignore

```
# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.eggs/

# Environnement virtuel
venv/
env/
.venv/

# IDE
.vscode/
.idea/
*.swp

# Environnement
.env
*.db

# Tests
htmlcov/
.coverage
.pytest_cache/

# Docker
postgres_data/
```

---

## Carte Mentale

```
                            PROJET HBnB
                                |
            +-------------------+-------------------+
            |                   |                   |
       ARCHITECTURE         BACKEND             FRONTEND
       EN COUCHES
            |                   |                   |
    +-------+------+    +------+------+    +-------+------+
    |       |      |    |      |      |    |       |      |
  Presen- Logique Persis- Modeles  API   Auth  HTML/CSS  JS
  tation  Metier  tance  (ORM)  REST  JWT  Jinja2  fetch()
    |       |      |             |      |
  Flask  Facade  Repo        Routes  bcrypt
  FastAPI        Pattern     CRUD    tokens
                   |
           +-------+-------+
           |               |
       InMemory          SQL
       (dev/test)     (production)
                         |
                    +----+----+
                    |         |
                 SQLAlchemy  Alembic
                 PostgreSQL  migrations
```

---

## Liens

- [[08 - APIs REST avec Flask]] -- Construction d'API REST avec Flask, Blueprints, serialisation JSON
- [[09 - APIs REST avec FastAPI]] -- Construction d'API REST avec FastAPI, Pydantic, typage
- [[10 - Python et Bases de Donnees]] -- SQLAlchemy, ORM, requetes SQL depuis Python
- [[../SQL/03 - Conception de Bases de Donnees]] -- Modelisation relationnelle, normalisation, ERD
- [[../JavaScript/03 - JavaScript Asynchrone]] -- fetch(), Promises, async/await pour le frontend
- [[../DevOps/03 - Docker Compose en Pratique]] -- Docker, docker-compose, deploiement multi-conteneurs
- [[../Secutrity/02 - Securite Web OWASP]] -- OWASP Top 10, XSS, CSRF, injection SQL, securite JWT
- [[../Tests et Qualite/01 - Tests Unitaires et TDD]] -- pytest, TDD, couverture de code, tests d'integration
