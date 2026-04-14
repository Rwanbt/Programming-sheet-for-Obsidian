# Python et Bases de Donnees

Connecter une application Python a une base de donnees est une etape essentielle dans le developpement de tout projet serieux. Plutot que d'ecrire du SQL brut a la main et de gerer manuellement les connexions, les ORM (Object-Relational Mapping) permettent de manipuler les donnees comme des objets Python. **SQLAlchemy** est l'ORM le plus puissant et le plus utilise dans l'ecosysteme Python.

Ce cours couvre l'ensemble du workflow : connexion a la base, definition de modeles, operations CRUD, requetes avancees, relations entre tables, et migrations avec Alembic.

> [!tip] Analogie
> Imaginez que vous parlez a un **traducteur** entre deux langues. D'un cote, vous parlez Python (objets, classes, attributs). De l'autre cote, la base de donnees parle SQL (tables, colonnes, requetes). L'ORM est ce traducteur : vous lui decrivez ce que vous voulez en Python, et il traduit en SQL pour la base de donnees. Vous n'avez jamais besoin de parler SQL directement (mais vous pouvez si necessaire).

---

## 1. Pourquoi utiliser un ORM ?

### Le probleme du SQL brut

Ecrire du SQL directement dans le code Python fonctionne, mais pose des problemes a grande echelle :

```python
# Approche SQL brut (sans ORM)
import sqlite3

conn = sqlite3.connect("blog.db")
cursor = conn.cursor()

# Creer une table
cursor.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT NOT NULL UNIQUE,
        email TEXT NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
""")

# Inserer un utilisateur (attention aux injections SQL !)
username = "alice"
email = "alice@mail.com"
cursor.execute(
    "INSERT INTO users (username, email) VALUES (?, ?)",
    (username, email)
)
conn.commit()

# Lire les utilisateurs
cursor.execute("SELECT * FROM users WHERE username = ?", ("alice",))
row = cursor.fetchone()
# row = (1, 'alice', 'alice@mail.com', '2025-01-15 10:30:00')
# C'est un tuple : row[0], row[1]... pas tres lisible

conn.close()
```

> [!warning] Problemes du SQL brut
> - **Injections SQL** : risque si on concatene des chaines au lieu d'utiliser les placeholders
> - **Pas d'objets** : les resultats sont des tuples/dicts, pas des objets avec des attributs
> - **SQL specifique** : le SQL varie entre SQLite, PostgreSQL, MySQL...
> - **Pas de migration** : modifier la structure est manuel et risque
> - **Repetitif** : chaque operation necessite du code boilerplate

### Comparaison SQL brut vs ORM

```
┌──────────────────────┬──────────────────────────┬──────────────────────────┐
│ Critere              │ SQL brut                 │ ORM (SQLAlchemy)         │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Securite             │ Risque injection SQL     │ Parametres echappes      │
│                      │ si mal ecrit             │ automatiquement          │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Lisibilite           │ SQL dans des strings     │ Code Python pur          │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Portabilite          │ SQL specifique au SGBD   │ Abstrait le SGBD         │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Resultats            │ Tuples / dicts           │ Objets Python            │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Relations            │ JOIN manuels             │ Accesseurs automatiques  │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Migrations           │ Scripts SQL manuels      │ Alembic (auto)           │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Performance          │ Controle total           │ Leger overhead, mais     │
│                      │                          │ optimisable              │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Courbe apprentissage │ Connaitre SQL suffit     │ Apprendre l'ORM + SQL    │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Quand choisir        │ Requetes tres complexes, │ La plupart des projets,  │
│                      │ scripts ponctuels        │ APIs, applications web   │
└──────────────────────┴──────────────────────────┴──────────────────────────┘
```

---

## 2. Installation de SQLAlchemy

```bash
# Installer SQLAlchemy
pip install sqlalchemy

# Optionnel : drivers pour differentes bases
pip install psycopg2-binary   # PostgreSQL
pip install pymysql            # MySQL
pip install aiosqlite          # SQLite async

# Pour les migrations
pip install alembic
```

> [!info] Versions
> Ce cours utilise **SQLAlchemy 2.0+**, la version moderne avec le style declaratif. L'ancienne syntaxe (1.x) est encore repandue dans les tutoriels en ligne, mais la 2.0 est recommandee pour tout nouveau projet.

---

## 3. Engine et Connexion

L'**engine** est le point d'entree vers la base de donnees. Il gere le pool de connexions et traduit les operations Python en SQL.

### Chaines de connexion

```python
from sqlalchemy import create_engine

# SQLite (fichier local) - ideal pour le developpement
engine = create_engine("sqlite:///ma_base.db", echo=True)
# echo=True affiche le SQL genere dans la console (debug)

# SQLite en memoire (disparait a la fermeture)
engine = create_engine("sqlite:///:memory:")

# PostgreSQL
engine = create_engine("postgresql://user:password@localhost:5432/ma_base")

# MySQL
engine = create_engine("mysql+pymysql://user:password@localhost:3306/ma_base")

# Format general :
# dialect+driver://username:password@host:port/database
```

### Anatomie d'une chaine de connexion

```
postgresql+psycopg2://admin:s3cret@db.example.com:5432/production
    │         │        │     │          │           │       │
    │         │        │     │          │           │       └─ nom de la base
    │         │        │     │          │           └─ port
    │         │        │     │          └─ host / adresse du serveur
    │         │        │     └─ mot de passe
    │         │        └─ nom d'utilisateur
    │         └─ driver Python (optionnel)
    └─ dialecte (type de SGBD)
```

### Options de l'engine

```python
engine = create_engine(
    "postgresql://user:pass@localhost/db",
    echo=True,            # Afficher le SQL genere
    pool_size=5,          # Nombre de connexions dans le pool
    max_overflow=10,      # Connexions supplementaires si pool plein
    pool_timeout=30,      # Secondes d'attente pour une connexion
    pool_recycle=1800,    # Recycler les connexions apres 30 min
)
```

> [!tip] Pool de connexions
> Ouvrir une connexion a la base est couteux. L'engine maintient un **pool** de connexions reutilisables. Quand votre code demande une connexion, il en obtient une du pool. Quand il a fini, la connexion retourne dans le pool au lieu d'etre fermee.

---

## 4. ORM : Base Declarative

SQLAlchemy offre deux approches : le **Core** (bas niveau, proche du SQL) et l'**ORM** (haut niveau, objets Python). Nous utilisons l'ORM.

### Creer la base declarative

```python
from sqlalchemy.orm import DeclarativeBase

# SQLAlchemy 2.0+ : classe de base moderne
class Base(DeclarativeBase):
    pass

# Toutes les classes de modeles heriteront de Base
# Base contient un registre (metadata) de toutes les tables
```

> [!info] Ancienne syntaxe (1.x)
> Avant la 2.0, on utilisait `declarative_base()` :
> ```python
> from sqlalchemy.orm import declarative_base
> Base = declarative_base()  # Style 1.x, encore fonctionnel
> ```
> Les deux fonctionnent, mais le style 2.0 (classe) est recommande.

---

## 5. Definir des Modeles

Un **modele** est une classe Python qui represente une table dans la base de donnees. Chaque attribut de classe correspond a une colonne.

### Types de colonnes

```python
from sqlalchemy import (
    Column, Integer, String, Text, Float, Boolean,
    DateTime, ForeignKey, Table
)
from sqlalchemy.orm import DeclarativeBase, relationship
from datetime import datetime

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"  # Nom de la table en base

    # Colonnes
    id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String(50), unique=True, nullable=False)
    email = Column(String(120), unique=True, nullable=False)
    bio = Column(Text, nullable=True)          # Texte long, sans limite
    score = Column(Float, default=0.0)         # Nombre decimal
    is_active = Column(Boolean, default=True)  # Vrai/Faux
    created_at = Column(DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f"<User(id={self.id}, username='{self.username}')>"
```

### Tableau des types de colonnes

```
┌──────────────┬──────────────────────┬──────────────────────────┐
│ Type SQLAlch.│ Type SQL genere      │ Type Python equivalent   │
├──────────────┼──────────────────────┼──────────────────────────┤
│ Integer      │ INTEGER              │ int                      │
│ String(n)    │ VARCHAR(n)           │ str                      │
│ Text         │ TEXT                 │ str (long)               │
│ Float        │ FLOAT                │ float                    │
│ Boolean      │ BOOLEAN              │ bool                     │
│ DateTime     │ DATETIME / TIMESTAMP │ datetime.datetime        │
│ Date         │ DATE                 │ datetime.date            │
│ Time         │ TIME                 │ datetime.time            │
│ LargeBinary  │ BLOB                 │ bytes                    │
│ Numeric(p,s) │ NUMERIC(p,s)         │ decimal.Decimal          │
│ Enum         │ ENUM / VARCHAR       │ enum.Enum                │
└──────────────┴──────────────────────┴──────────────────────────┘
```

### Options des colonnes

```python
# Les options les plus courantes pour Column()
Column(Integer, primary_key=True)           # Cle primaire
Column(String(50), nullable=False)          # NOT NULL (obligatoire)
Column(String(100), unique=True)            # Valeur unique
Column(Float, default=0.0)                  # Valeur par defaut (Python)
Column(DateTime, server_default=func.now()) # Valeur par defaut (SQL)
Column(String(50), index=True)              # Creer un index (perf.)
Column(String, comment="Description")       # Commentaire en base
```

### Creer les tables en base

```python
# Creer toutes les tables definies dans Base
engine = create_engine("sqlite:///blog.db", echo=True)
Base.metadata.create_all(engine)

# Cela genere le SQL :
# CREATE TABLE users (
#     id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
#     username VARCHAR(50) NOT NULL UNIQUE,
#     email VARCHAR(120) NOT NULL UNIQUE,
#     bio TEXT,
#     score FLOAT DEFAULT 0.0,
#     is_active BOOLEAN DEFAULT 1,
#     created_at DATETIME
# )
```

---

## 6. Relations entre Tables

Les bases de donnees relationnelles tirent leur puissance des **relations** entre tables. SQLAlchemy les gere elegamment.

### Relation One-to-Many (un a plusieurs)

Un utilisateur peut ecrire plusieurs articles. Chaque article appartient a un seul utilisateur.

```python
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True, nullable=False)
    email = Column(String(120), nullable=False)

    # Relation : un user a plusieurs posts
    posts = relationship("Post", back_populates="author")

    def __repr__(self):
        return f"<User(username='{self.username}')>"


class Post(Base):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)

    # Cle etrangere : reference vers users.id
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)

    # Relation inverse : chaque post a un author
    author = relationship("User", back_populates="posts")

    def __repr__(self):
        return f"<Post(title='{self.title}')>"
```

```
Schema de la relation One-to-Many :

  users                          posts
  ┌─────────────────┐           ┌─────────────────────┐
  │ id (PK)         │──────┐    │ id (PK)             │
  │ username        │      │    │ title               │
  │ email           │      │    │ content             │
  └─────────────────┘      │    │ created_at          │
                           └───>│ author_id (FK) ─────│──> users.id
                                └─────────────────────┘

  User.posts  ──>  [Post, Post, Post]    (liste)
  Post.author ──>  User                  (objet unique)
```

> [!info] back_populates vs backref
> - `back_populates` : explicite, chaque cote declare la relation. Recommande.
> - `backref` : implicite, une seule declaration suffit. Plus court mais moins lisible.
> ```python
> # Avec backref (equivalent mais implicite)
> posts = relationship("Post", backref="author")
> # Pas besoin de declarer 'author' dans Post
> ```

### Relation Many-to-Many (plusieurs a plusieurs)

Un article peut avoir plusieurs tags, et un tag peut etre associe a plusieurs articles. Cela necessite une **table d'association**.

```python
# Table d'association (pas de classe, juste une table)
post_tags = Table(
    "post_tags",
    Base.metadata,
    Column("post_id", Integer, ForeignKey("posts.id"), primary_key=True),
    Column("tag_id", Integer, ForeignKey("tags.id"), primary_key=True),
)


class Tag(Base):
    __tablename__ = "tags"

    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)

    # Relation Many-to-Many via la table d'association
    posts = relationship("Post", secondary=post_tags, back_populates="tags")

    def __repr__(self):
        return f"<Tag(name='{self.name}')>"


class Post(Base):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(Text)
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)

    author = relationship("User", back_populates="posts")
    tags = relationship("Tag", secondary=post_tags, back_populates="posts")

    def __repr__(self):
        return f"<Post(title='{self.title}')>"
```

```
Schema Many-to-Many :

  posts              post_tags            tags
  ┌──────────┐      ┌─────────────┐     ┌──────────┐
  │ id (PK)  │<─────│ post_id (FK)│     │ id (PK)  │
  │ title    │      │ tag_id (FK) │────>│ name     │
  │ content  │      └─────────────┘     └──────────┘
  └──────────┘
      │                                     │
      └── post.tags = [Tag, Tag] ───────────┘
                                   tag.posts = [Post, Post]
```

### Relation One-to-One

```python
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    username = Column(String(50))

    # uselist=False : un seul profil, pas une liste
    profile = relationship("Profile", back_populates="user", uselist=False)


class Profile(Base):
    __tablename__ = "profiles"
    id = Column(Integer, primary_key=True)
    avatar_url = Column(String(255))
    user_id = Column(Integer, ForeignKey("users.id"), unique=True)

    user = relationship("User", back_populates="profile")
```

---

## 7. Sessions

La **session** est l'interface principale pour interagir avec la base de donnees. Elle gere les transactions, le cache d'identite et la persistence des objets.

### Creer et utiliser une session

```python
from sqlalchemy.orm import sessionmaker

# Creer une fabrique de sessions liee a l'engine
Session = sessionmaker(bind=engine)

# Methode 1 : session manuelle
session = Session()
try:
    # ... operations ...
    session.commit()
except Exception:
    session.rollback()
    raise
finally:
    session.close()

# Methode 2 : context manager (recommande)
with Session() as session:
    with session.begin():
        # Tout ce qui est ici est dans une transaction
        # commit automatique a la fin du bloc
        # rollback automatique en cas d'exception
        session.add(User(username="alice", email="alice@mail.com"))
    # La session est fermee automatiquement
```

### Cycle de vie d'un objet dans la session

```
                    ┌──────────┐
                    │ Transient│  Objet cree, pas encore dans la session
                    └────┬─────┘
                         │ session.add()
                         v
                    ┌──────────┐
                    │ Pending  │  Dans la session, pas encore en base
                    └────┬─────┘
                         │ session.flush() / session.commit()
                         v
                    ┌──────────┐
                    │Persistent│  En session ET en base
                    └────┬─────┘
                         │ session.expunge() / session.close()
                         v
                    ┌──────────┐
                    │ Detached │  Plus dans la session, mais a un id en base
                    └──────────┘
```

> [!warning] Session et threads
> Une session n'est **pas thread-safe**. En application web, utilisez une session par requete HTTP. Avec FastAPI, utilisez `Depends()` pour injecter une session par requete.

### sessionmaker avancee

```python
from sqlalchemy.orm import sessionmaker

# Configuration courante pour une application
SessionLocal = sessionmaker(
    bind=engine,
    autocommit=False,   # Commit explicite requis
    autoflush=False,    # Flush explicite (meilleur controle)
    expire_on_commit=True  # Re-fetch les objets apres commit
)

# Pattern pour FastAPI / Flask
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

## 8. Operations CRUD

### Create (creer)

```python
with Session() as session:
    with session.begin():
        # Creer un seul objet
        user = User(username="alice", email="alice@mail.com")
        session.add(user)

        # Creer plusieurs objets
        tags = [
            Tag(name="python"),
            Tag(name="sqlalchemy"),
            Tag(name="orm"),
        ]
        session.add_all(tags)

    # Apres commit, l'objet a un id
    print(user.id)  # 1
```

### Read (lire)

```python
with Session() as session:
    # Obtenir par cle primaire
    user = session.get(User, 1)  # SQLAlchemy 2.0
    # Ancienne syntaxe : session.query(User).get(1)

    # Tous les utilisateurs
    users = session.query(User).all()
    # -> [<User(username='alice')>, <User(username='bob')>]

    # Premier resultat
    first_user = session.query(User).first()
    # -> <User(username='alice')>  ou  None

    # Exactement un resultat (leve une exception sinon)
    try:
        unique_user = session.query(User).filter_by(username="alice").one()
    except Exception:
        print("Pas exactement un resultat !")

    # Filtrer avec filter_by (egalite simple)
    alice = session.query(User).filter_by(username="alice").first()

    # Filtrer avec filter (expressions complexes)
    active_users = session.query(User).filter(
        User.is_active == True,
        User.score > 50.0
    ).all()

    # Filtres avances
    from sqlalchemy import or_, and_, not_

    results = session.query(User).filter(
        or_(
            User.username == "alice",
            User.username == "bob"
        )
    ).all()

    # LIKE
    results = session.query(User).filter(
        User.email.like("%@gmail.com")
    ).all()

    # IN
    results = session.query(User).filter(
        User.username.in_(["alice", "bob", "charlie"])
    ).all()

    # IS NULL / IS NOT NULL
    results = session.query(User).filter(User.bio.is_(None)).all()
    results = session.query(User).filter(User.bio.isnot(None)).all()

    # BETWEEN
    results = session.query(User).filter(
        User.score.between(10.0, 90.0)
    ).all()
```

### Update (modifier)

```python
with Session() as session:
    with session.begin():
        # Methode 1 : modifier l'objet directement
        user = session.query(User).filter_by(username="alice").first()
        if user:
            user.email = "new_alice@mail.com"
            user.score = 95.0
            # Le commit sauvegarde les changements

        # Methode 2 : update en masse (plus performant)
        session.query(User).filter(
            User.is_active == False
        ).update(
            {"score": 0.0},
            synchronize_session="evaluate"
        )
```

### Delete (supprimer)

```python
with Session() as session:
    with session.begin():
        # Methode 1 : supprimer un objet
        user = session.query(User).filter_by(username="alice").first()
        if user:
            session.delete(user)

        # Methode 2 : suppression en masse
        session.query(User).filter(
            User.is_active == False
        ).delete(synchronize_session="evaluate")
```

> [!example] CRUD complet en action
> ```python
> with Session() as session:
>     with session.begin():
>         # Create
>         u = User(username="demo", email="demo@test.com")
>         session.add(u)
>     
>     # Read
>     u = session.query(User).filter_by(username="demo").first()
>     print(u.email)  # demo@test.com
>     
>     with session.begin():
>         # Update
>         u.email = "updated@test.com"
>     
>     with session.begin():
>         # Delete
>         session.delete(u)
> ```

---

## 9. Requetes Avancees

### Tri et limitation

```python
# ORDER BY
users = session.query(User).order_by(User.username).all()
users = session.query(User).order_by(User.score.desc()).all()

# LIMIT et OFFSET (pagination)
page = 2
per_page = 10
users = session.query(User).order_by(User.id).offset(
    (page - 1) * per_page
).limit(per_page).all()
```

### Jointures

```python
# JOIN implicite (via relationship)
# Quand on accede a user.posts, SQLAlchemy fait le JOIN automatiquement
user = session.query(User).filter_by(username="alice").first()
for post in user.posts:  # SELECT ... FROM posts WHERE author_id = ?
    print(post.title)

# JOIN explicite
from sqlalchemy import func

results = session.query(
    User.username,
    func.count(Post.id).label("nb_posts")
).join(Post, User.id == Post.author_id).group_by(
    User.username
).all()
# -> [('alice', 5), ('bob', 3)]

# LEFT JOIN
results = session.query(User, Post).outerjoin(
    Post, User.id == Post.author_id
).all()
```

### Agregation (GROUP BY, HAVING)

```python
from sqlalchemy import func

# COUNT
nb_users = session.query(func.count(User.id)).scalar()

# Nombre de posts par auteur
stats = session.query(
    User.username,
    func.count(Post.id).label("nb_posts"),
    func.max(Post.created_at).label("dernier_post")
).join(Post).group_by(User.username).all()

# HAVING : filtrer apres GROUP BY
prolific = session.query(
    User.username,
    func.count(Post.id).label("nb_posts")
).join(Post).group_by(
    User.username
).having(
    func.count(Post.id) > 5
).all()
```

### Sous-requetes

```python
from sqlalchemy import select

# Sous-requete : utilisateurs qui ont plus de 3 posts
subq = (
    select(Post.author_id)
    .group_by(Post.author_id)
    .having(func.count(Post.id) > 3)
    .subquery()
)

active_authors = session.query(User).filter(
    User.id.in_(select(subq.c.author_id))
).all()
```

### Eager vs Lazy Loading

```python
from sqlalchemy.orm import joinedload, selectinload, lazyload

# Lazy loading (defaut) : charge les relations a la demande
# Chaque acces a user.posts fait une requete SQL supplementaire
user = session.query(User).first()
print(user.posts)  # -> SELECT * FROM posts WHERE author_id = 1

# Eager loading avec joinedload : un seul JOIN
users = session.query(User).options(
    joinedload(User.posts)
).all()
# -> SELECT users.*, posts.* FROM users LEFT JOIN posts ON ...
# Tout est charge en une seule requete !

# Eager loading avec selectinload : deux requetes separees
users = session.query(User).options(
    selectinload(User.posts)
).all()
# -> SELECT * FROM users
# -> SELECT * FROM posts WHERE author_id IN (1, 2, 3, ...)
```

> [!warning] Le probleme N+1
> Le **N+1 problem** survient quand on itere sur N objets et que chaque acces a une relation genere une requete supplementaire :
> ```python
> # MAUVAIS : N+1 requetes (1 pour les users + N pour les posts)
> users = session.query(User).all()  # 1 requete
> for user in users:
>     print(user.posts)  # N requetes supplementaires !
>
> # BON : 1 seule requete avec joinedload
> users = session.query(User).options(joinedload(User.posts)).all()
> for user in users:
>     print(user.posts)  # Deja charge, pas de requete !
> ```

---

## 10. Migrations avec Alembic

Quand la structure de la base change (ajout de colonne, nouvelle table...), il faut **migrer** la base existante. **Alembic** est l'outil de migration officiel de SQLAlchemy.

### Initialisation

```bash
# Initialiser Alembic dans le projet
alembic init alembic

# Structure creee :
# alembic/
#   env.py         <- configuration de l'environnement
#   versions/      <- dossier des fichiers de migration
# alembic.ini      <- configuration principale
```

### Configuration

```python
# alembic/env.py - Importer les modeles pour l'autogeneration
from models import Base  # Votre fichier de modeles

target_metadata = Base.metadata  # Remplacer None par ceci
```

```ini
# alembic.ini - Configurer la connexion
sqlalchemy.url = sqlite:///blog.db
# Ou : postgresql://user:pass@localhost/blog
```

### Creer et appliquer des migrations

```bash
# Generer une migration automatiquement (detecte les changements)
alembic revision --autogenerate -m "creation tables users et posts"

# Appliquer toutes les migrations en attente
alembic upgrade head

# Voir l'historique des migrations
alembic history

# Voir la migration actuelle
alembic current

# Revenir en arriere d'une migration
alembic downgrade -1

# Revenir a une revision specifique
alembic downgrade abc123

# Revenir au debut (aucune migration)
alembic downgrade base
```

### Exemple de fichier de migration

```python
"""creation tables users et posts

Revision ID: a1b2c3d4e5f6
Revises:
Create Date: 2025-06-15 14:30:00.000000
"""
from alembic import op
import sqlalchemy as sa

# Identifiants de revision
revision = 'a1b2c3d4e5f6'
down_revision = None
branch_labels = None
depends_on = None


def upgrade():
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), primary_key=True),
        sa.Column('username', sa.String(50), unique=True, nullable=False),
        sa.Column('email', sa.String(120), nullable=False),
    )
    op.create_table(
        'posts',
        sa.Column('id', sa.Integer(), primary_key=True),
        sa.Column('title', sa.String(200), nullable=False),
        sa.Column('author_id', sa.Integer(), sa.ForeignKey('users.id')),
    )


def downgrade():
    op.drop_table('posts')
    op.drop_table('users')
```

> [!tip] Bonnes pratiques Alembic
> - Toujours ecrire le `downgrade()` pour pouvoir revenir en arriere
> - Commitez les fichiers de migration dans git
> - Testez les migrations sur une copie de la base avant de les appliquer en production
> - Utilisez `--autogenerate` mais **relisez toujours** le fichier genere

---

## 11. SQL Brut dans SQLAlchemy

Parfois, une requete est trop complexe pour l'ORM. SQLAlchemy permet d'executer du SQL brut.

```python
from sqlalchemy import text

with engine.connect() as conn:
    # Requete simple
    result = conn.execute(text("SELECT * FROM users WHERE is_active = :active"),
                          {"active": True})
    for row in result:
        print(row.username, row.email)

    # Requete avec plusieurs parametres
    result = conn.execute(
        text("""
            SELECT u.username, COUNT(p.id) as nb_posts
            FROM users u
            LEFT JOIN posts p ON u.id = p.author_id
            WHERE u.created_at > :date
            GROUP BY u.username
            HAVING COUNT(p.id) > :min_posts
        """),
        {"date": "2025-01-01", "min_posts": 5}
    )

# Dans une session ORM
with Session() as session:
    result = session.execute(
        text("SELECT * FROM users WHERE score > :score"),
        {"score": 80.0}
    )
    rows = result.fetchall()
```

> [!warning] Securite
> Utilisez **toujours** `text()` avec des parametres nommes (`:param`). Ne concatenez **jamais** de variables dans une chaine SQL :
> ```python
> # DANGEREUX - Injection SQL possible !
> session.execute(text(f"SELECT * FROM users WHERE id = {user_id}"))
>
> # SECURISE - Parametres echappes
> session.execute(text("SELECT * FROM users WHERE id = :id"), {"id": user_id})
> ```

---

## 12. Exemple Pratique : Modeles d'un Blog

Voici un exemple complet de modeles pour une API de blog avec utilisateurs, articles, commentaires et tags.

```python
from sqlalchemy import (
    Column, Integer, String, Text, Boolean, DateTime,
    ForeignKey, Table, create_engine, func
)
from sqlalchemy.orm import (
    DeclarativeBase, relationship, sessionmaker
)
from datetime import datetime


# === Base ===
class Base(DeclarativeBase):
    pass


# === Table d'association pour Many-to-Many ===
post_tags = Table(
    "post_tags",
    Base.metadata,
    Column("post_id", Integer, ForeignKey("posts.id", ondelete="CASCADE"),
           primary_key=True),
    Column("tag_id", Integer, ForeignKey("tags.id", ondelete="CASCADE"),
           primary_key=True),
)


# === Modeles ===
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True, nullable=False, index=True)
    email = Column(String(120), unique=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    bio = Column(Text, default="")
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow,
                        onupdate=datetime.utcnow)

    # Relations
    posts = relationship("Post", back_populates="author",
                         cascade="all, delete-orphan")
    comments = relationship("Comment", back_populates="author",
                            cascade="all, delete-orphan")

    def __repr__(self):
        return f"<User(username='{self.username}')>"


class Post(Base):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False, index=True)
    slug = Column(String(200), unique=True, nullable=False)
    content = Column(Text, nullable=False)
    excerpt = Column(String(500))
    is_published = Column(Boolean, default=False)
    view_count = Column(Integer, default=0)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow,
                        onupdate=datetime.utcnow)
    published_at = Column(DateTime, nullable=True)

    # Cle etrangere
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)

    # Relations
    author = relationship("User", back_populates="posts")
    comments = relationship("Comment", back_populates="post",
                            cascade="all, delete-orphan",
                            order_by="Comment.created_at")
    tags = relationship("Tag", secondary=post_tags,
                        back_populates="posts")

    def __repr__(self):
        return f"<Post(title='{self.title}')>"


class Comment(Base):
    __tablename__ = "comments"

    id = Column(Integer, primary_key=True)
    content = Column(Text, nullable=False)
    is_approved = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)

    # Cles etrangeres
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    post_id = Column(Integer, ForeignKey("posts.id"), nullable=False)

    # Relations
    author = relationship("User", back_populates="comments")
    post = relationship("Post", back_populates="comments")

    def __repr__(self):
        return f"<Comment(id={self.id}, post_id={self.post_id})>"


class Tag(Base):
    __tablename__ = "tags"

    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False, index=True)
    description = Column(String(200))

    # Relation Many-to-Many
    posts = relationship("Post", secondary=post_tags,
                         back_populates="tags")

    def __repr__(self):
        return f"<Tag(name='{self.name}')>"


# === Initialisation ===
engine = create_engine("sqlite:///blog.db", echo=False)
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
```

### Utiliser les modeles

```python
with Session() as session:
    with session.begin():
        # Creer un utilisateur
        alice = User(
            username="alice",
            email="alice@blog.com",
            password_hash="hashed_password_here"
        )
        session.add(alice)

    with session.begin():
        # Creer des tags
        python_tag = Tag(name="python", description="Langage Python")
        web_tag = Tag(name="web", description="Developpement Web")
        session.add_all([python_tag, web_tag])

    with session.begin():
        # Creer un article avec tags
        post = Post(
            title="Introduction a SQLAlchemy",
            slug="intro-sqlalchemy",
            content="SQLAlchemy est un ORM Python...",
            excerpt="Decouvrez SQLAlchemy",
            author=alice,
            tags=[python_tag, web_tag],
            is_published=True,
            published_at=datetime.utcnow()
        )
        session.add(post)

    with session.begin():
        # Ajouter un commentaire
        comment = Comment(
            content="Super article !",
            author=alice,
            post=post
        )
        session.add(comment)

    # Lire avec relations
    user = session.query(User).filter_by(username="alice").first()
    print(f"{user.username} a {len(user.posts)} article(s)")
    for p in user.posts:
        print(f"  - {p.title} ({len(p.tags)} tags, {len(p.comments)} com.)")
        for tag in p.tags:
            print(f"    Tag: {tag.name}")
```

---

## 13. Bonnes Pratiques

### Gestion du cycle de vie des sessions

```python
# Pattern recommande pour les applications web
from contextlib import contextmanager

@contextmanager
def get_session():
    session = Session()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

# Utilisation
with get_session() as session:
    user = User(username="test", email="test@mail.com", password_hash="hash")
    session.add(user)
# Commit automatique si pas d'erreur, rollback sinon
```

### Eviter le probleme N+1

```python
from sqlalchemy.orm import joinedload, selectinload

# Charger les posts avec leurs tags et commentaires en une requete
posts = session.query(Post).options(
    joinedload(Post.author),           # JOIN sur l'auteur
    selectinload(Post.tags),           # IN query pour les tags
    selectinload(Post.comments)        # IN query pour les commentaires
        .joinedload(Comment.author)    # JOIN sur l'auteur du commentaire
).filter(Post.is_published == True).all()
```

### Indexation

```python
from sqlalchemy import Index

class Post(Base):
    __tablename__ = "posts"
    # ...

    # Index compose pour les requetes frequentes
    __table_args__ = (
        Index("idx_posts_author_published", "author_id", "is_published"),
        Index("idx_posts_created_at", "created_at"),
    )
```

> [!tip] Quand ajouter un index ?
> - Sur les colonnes utilisees dans `WHERE`, `JOIN`, `ORDER BY`
> - Sur les cles etrangeres (SQLAlchemy ne les indexe pas automatiquement)
> - **Ne pas** indexer les colonnes rarement filtrees ou les tables tres petites
> - Trop d'index ralentit les ecritures (`INSERT`, `UPDATE`)

### Separation des concerns

```
Structure recommandee d'un projet :

mon_projet/
├── models/
│   ├── __init__.py       # Exporte Base et les modeles
│   ├── base.py           # DeclarativeBase, engine, Session
│   ├── user.py           # Modele User
│   ├── post.py           # Modele Post
│   └── tag.py            # Modele Tag
├── repositories/
│   ├── user_repo.py      # Fonctions CRUD pour User
│   └── post_repo.py      # Fonctions CRUD pour Post
├── alembic/
│   ├── env.py
│   └── versions/
├── alembic.ini
└── main.py
```

---

## 14. Comparaison avec C

En C, il n'existe pas d'ORM standard. L'acces aux bases de donnees se fait via des bibliotheques bas niveau.

```c
// En C : acces a SQLite avec la bibliotheque C native
#include <stdio.h>
#include <sqlite3.h>

int main(void) {
    sqlite3 *db;
    sqlite3_stmt *stmt;

    // Ouvrir la connexion
    int rc = sqlite3_open("blog.db", &db);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Erreur: %s\n", sqlite3_errmsg(db));
        return 1;
    }

    // Executer une requete preparee (contre les injections SQL)
    const char *sql = "SELECT id, username FROM users WHERE is_active = ?";
    rc = sqlite3_prepare_v2(db, sql, -1, &stmt, NULL);

    sqlite3_bind_int(stmt, 1, 1);  // is_active = 1 (true)

    while (sqlite3_step(stmt) == SQLITE_ROW) {
        int id = sqlite3_column_int(stmt, 0);
        const char *username = (const char *)sqlite3_column_text(stmt, 1);
        printf("User %d: %s\n", id, username);
    }

    // Liberer les ressources (pas de garbage collector !)
    sqlite3_finalize(stmt);
    sqlite3_close(db);
    return 0;
}
// Compilation : gcc main.c -lsqlite3 -o main
```

```
Comparaison Python (SQLAlchemy) vs C (sqlite3) :

┌─────────────────┬────────────────────────┬────────────────────────┐
│ Aspect          │ Python + SQLAlchemy    │ C + sqlite3            │
├─────────────────┼────────────────────────┼────────────────────────┤
│ Abstraction     │ ORM : objets Python    │ API C bas niveau       │
│ Securite        │ Automatique            │ Manuelle (prepare)     │
│ Gestion memoire │ Garbage collector      │ malloc/free, finalize  │
│ Migrations      │ Alembic               │ Scripts SQL manuels    │
│ Relations       │ relationship()         │ JOIN manuels           │
│ Portabilite DB  │ Changement d'URL       │ Recompiler avec autre  │
│                 │                        │ bibliotheque           │
│ Performance     │ Bon (avec overhead)    │ Maximale               │
│ Productivite    │ Tres elevee            │ Faible                 │
└─────────────────┴────────────────────────┴────────────────────────┘
```

---

## Carte Mentale

```
                     Python et Bases de Donnees
                              │
          ┌──────────┬────────┼────────┬──────────┐
          │          │        │        │          │
       Engine    Modeles   Sessions  Requetes   Alembic
          │          │        │        │          │
     ┌────┤     ┌────┤    ┌───┤    ┌───┤     ┌────┤
     │    │     │    │    │   │    │   │     │    │
  create  │  Column   │  add   │  filter │  init   │
  _engine │     │    │    │   │    │   │     │    │
     │    │  Integer  │ commit │  join  │ revision │
     │    │     │    │    │   │    │   │     │    │
  pool    │  String   │ rollback│ group  │ upgrade │
     │    │     │    │    │   │    │   │     │    │
  echo    │  relation │  with  │ func   │ downgrade│
     │    │  ship()   │  stmt  │    │     │    │
  connect │     │    │        │ eager/  │ auto     │
  string  │  ForeignKey│       │ lazy    │ generate │
          │     │    │        │ loading │         │
       Base   back_  │        │        │
       Declar. populates     subquery  text()
       ative   │              │
              Many-to       N+1
              -Many         problem
```

---

## Exercices

### Exercice 1 : Modeles d'une application e-commerce

Creez les modeles SQLAlchemy pour une boutique en ligne :
- `Customer` : id, name, email, address, created_at
- `Product` : id, name, description, price, stock, category
- `Order` : id, customer_id (FK), total, status (enum: pending/paid/shipped/delivered), created_at
- `OrderItem` : id, order_id (FK), product_id (FK), quantity, unit_price

Implementez les relations et creez les tables dans une base SQLite. Inserez 3 clients, 5 produits, et 2 commandes avec leurs items.

### Exercice 2 : Requetes avancees

En utilisant les modeles de l'exercice 1 :
1. Listez les commandes d'un client specifique avec le detail des produits
2. Calculez le chiffre d'affaires total (somme des totaux de commandes payees)
3. Trouvez les 3 produits les plus commandes (GROUP BY + ORDER BY)
4. Listez les clients qui n'ont jamais commande (LEFT JOIN + IS NULL)

### Exercice 3 : Integration FastAPI + SQLAlchemy

Creez une mini-API FastAPI qui utilise les modeles de l'exercice 1 :
- `GET /products` : liste paginee des produits avec filtrage par categorie
- `POST /orders` : creer une commande (valider le stock disponible)
- `GET /orders/{id}` : detail d'une commande avec ses items (eager loading)
- Gerez les erreurs (produit non trouve, stock insuffisant) avec les bons codes HTTP

### Exercice 4 : Migrations Alembic

Partez des modeles de l'exercice 1. Appliquez les modifications suivantes avec Alembic :
1. Ajoutez une colonne `discount` (Float) a la table `products`
2. Ajoutez une table `Review` (id, product_id, customer_id, rating, text, created_at)
3. Generez et appliquez les migrations. Puis faites un downgrade pour verifier.

---

## Liens

- [[09 - APIs REST avec FastAPI]] - Integration avec FastAPI et Depends()
- [[01 - Introduction au SQL]] - Les bases du SQL
- [[03 - Conception de Bases de Donnees]] - Modelisation et normalisation
