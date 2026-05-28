# MongoDB Avance

Ce cours approfondit MongoDB au-dela du CRUD basique. On explore les outils qui font la difference entre une application qui "fonctionne" et une application **performante et maintenable** : indexation, aggregation pipeline, transactions, et patterns de schema design.

Prerequis : maitriser [[01 - Introduction au NoSQL]] (CRUD de base, connexion pymongo).

> [!tip] Analogie — L'aggregation pipeline
> Imaginez une chaine de production industrielle. La matiere premiere (vos documents) entre dans la chaine. A chaque etape ($match, $group, $sort...), elle est transformee, filtree, calculee. En sortie, vous obtenez exactement le rapport dont vous avez besoin. Pas de requete magique — une sequence d'etapes clairement definies.

---

## Indexation et Performance

Sans index, MongoDB fait un **collection scan** : il lit **chaque document** pour trouver ceux qui correspondent. Sur 10 millions de documents, c'est catastrophique.

### Comprendre les index

Un index est une structure de donnees (B-Tree) qui maintient une copie triee des valeurs d'un ou plusieurs champs, avec des pointeurs vers les documents correspondants.

```
Sans index : MongoDB lit 1 000 000 documents pour trouver "Alice"
Avec index sur "nom" : MongoDB consulte le B-Tree, trouve le pointeur, lit 1 document
```

> [!info] Cout des index
> | Avantage | Inconvenient |
> |---|---|
> | Lectures tres rapides | Ecritures plus lentes (index mis a jour) |
> | Tri instantane | Espace disque supplementaire |
> | Queries complexes optimisees | Trop d'index = overhead inutile |

### Creer des index avec mongosh

```javascript
// Index simple (ascendant)
db.etudiants.createIndex({ nom: 1 })

// Index descendant
db.etudiants.createIndex({ date_inscription: -1 })

// Index compose (sur plusieurs champs)
db.etudiants.createIndex({ formation: 1, age: -1 })

// Index unique (empeche les doublons)
db.etudiants.createIndex({ email: 1 }, { unique: true })

// Index sparse (ignore les documents sans ce champ)
db.etudiants.createIndex({ github_url: 1 }, { sparse: true })

// Index TTL : supprime automatiquement les documents apres N secondes
// Parfait pour les sessions, les tokens temporaires
db.sessions.createIndex({ created_at: 1 }, { expireAfterSeconds: 3600 })

// Voir tous les index d'une collection
db.etudiants.getIndexes()

// Supprimer un index
db.etudiants.dropIndex({ nom: 1 })
```

### Analyser les performances avec explain()

```javascript
// explain() montre comment MongoDB execute une requete
db.etudiants.find({ formation: "FullStack" }).explain("executionStats")

// Resultats importants a observer :
// - "stage": "COLLSCAN" → pas d'index (mauvais)
// - "stage": "IXSCAN"   → utilise un index (bien)
// - "nReturned"         → documents retournes
// - "totalDocsExamined" → documents examines (doit etre proche de nReturned)
```

### Index avec pymongo

```python
from pymongo import MongoClient, ASCENDING, DESCENDING, IndexModel

client = MongoClient("mongodb://localhost:27017/")
db = client["holberton_school"]
etudiants = db["etudiants"]

# Creer un index simple
etudiants.create_index([("nom", ASCENDING)])

# Index unique
etudiants.create_index([("email", ASCENDING)], unique=True)

# Index compose
etudiants.create_index([("formation", ASCENDING), ("age", DESCENDING)])

# Plusieurs index en une seule operation
index1 = IndexModel([("nom", ASCENDING)])
index2 = IndexModel([("email", ASCENDING)], unique=True)
index3 = IndexModel([("date_inscription", DESCENDING)], expireAfterSeconds=86400)

etudiants.create_indexes([index1, index2, index3])

# Voir les index existants
for index in etudiants.list_indexes():
    print(index)

# Verifier les performances
import pprint
plan = etudiants.find({"formation": "FullStack"}).explain()
pprint.pprint(plan["executionStats"])
```

---

## Aggregation Pipeline

L'aggregation pipeline est le moteur analytique de MongoDB. Une sequence d'**etapes** ($stages) transforme les documents, chacune recevant la sortie de la precedente.

```
Collection → [$match] → [$project] → [$group] → [$sort] → [$limit] → Resultats
```

### Etapes fondamentales

#### $match — Filtrer

```javascript
// Equivalent d'un WHERE SQL
{ $match: { formation: "FullStack", age: { $gte: 21 } } }
```

#### $project — Selectionner et transformer les champs

```javascript
// Equivalent d'un SELECT avec colonnes calculees
{
  $project: {
    _id: 0,
    nom: 1,
    moyenne: { $avg: ["$notes.algorithmique", "$notes.web", "$notes.systeme"] },
    nom_majuscule: { $toUpper: "$nom" }
  }
}
```

#### $group — Agregation et calculs

```javascript
// Equivalent d'un GROUP BY avec fonctions agregat
{
  $group: {
    _id: "$formation",          // Grouper par ce champ
    total: { $sum: 1 },         // Compter les documents
    age_moyen: { $avg: "$age" },
    note_max: { $max: "$notes.web" },
    tous_les_noms: { $push: "$nom" }
  }
}
```

#### $sort — Trier

```javascript
// 1 = ASC, -1 = DESC
{ $sort: { age_moyen: -1 } }
```

#### $limit et $skip — Pagination

```javascript
{ $skip: 10 },
{ $limit: 5 }
```

#### $unwind — Decomposer un tableau

```javascript
// Transforme chaque element d'un tableau en un document separe
// ["Python", "JS", "C"] → 3 documents
{ $unwind: "$langages" }
```

#### $lookup — Jointure avec une autre collection

```javascript
// Equivalent d'un JOIN SQL
{
  $lookup: {
    from: "formations",          // Collection a joindre
    localField: "formation_id",  // Champ local
    foreignField: "_id",         // Champ distant
    as: "infos_formation"        // Nom du tableau resultant
  }
}
```

---

### Exemples complets d'aggregation

```javascript
// Exemple 1 : Statistiques par formation
db.etudiants.aggregate([
  // Etape 1 : exclure les etudiants sans notes
  { $match: { "notes.web": { $exists: true } } },
  // Etape 2 : calculer la moyenne par etudiant
  { $project: {
    nom: 1,
    formation: 1,
    moyenne: { $avg: ["$notes.algorithmique", "$notes.web", "$notes.systeme"] }
  }},
  // Etape 3 : grouper par formation
  { $group: {
    _id: "$formation",
    nb_etudiants: { $sum: 1 },
    moyenne_promo: { $avg: "$moyenne" },
    meilleure_moyenne: { $max: "$moyenne" }
  }},
  // Etape 4 : trier par moyenne descendante
  { $sort: { moyenne_promo: -1 } }
])

// Exemple 2 : Langages les plus populaires
db.etudiants.aggregate([
  // Decomposer le tableau de langages
  { $unwind: "$langages" },
  // Compter par langage
  { $group: { _id: "$langages", count: { $sum: 1 } } },
  // Trier par popularite
  { $sort: { count: -1 } },
  // Prendre le top 5
  { $limit: 5 }
])
```

### Aggregation avec pymongo

```python
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017/")
db = client["holberton_school"]
etudiants = db["etudiants"]


def stats_par_formation(collection) -> list:
    """Calcule les statistiques par formation."""
    pipeline = [
        {"$match": {"notes.web": {"$exists": True}}},
        {"$project": {
            "nom": 1,
            "formation": 1,
            "moyenne": {"$avg": ["$notes.algorithmique", "$notes.web", "$notes.systeme"]}
        }},
        {"$group": {
            "_id": "$formation",
            "nb_etudiants": {"$sum": 1},
            "moyenne_promo": {"$avg": "$moyenne"},
            "meilleure_moyenne": {"$max": "$moyenne"}
        }},
        {"$sort": {"moyenne_promo": -1}}
    ]
    return list(collection.aggregate(pipeline))


def top_langages(collection, top_n: int = 5) -> list:
    """Retourne les N langages les plus populaires."""
    pipeline = [
        {"$unwind": "$langages"},
        {"$group": {"_id": "$langages", "count": {"$sum": 1}}},
        {"$sort": {"count": -1}},
        {"$limit": top_n},
        {"$project": {"_id": 0, "langage": "$_id", "nb_etudiants": "$count"}}
    ]
    return list(collection.aggregate(pipeline))


def etudiants_avec_formation(collection) -> list:
    """Jointure etudiants + formations (lookup)."""
    pipeline = [
        {"$lookup": {
            "from": "formations_detail",
            "localField": "formation",
            "foreignField": "code",
            "as": "detail_formation"
        }},
        {"$unwind": {"path": "$detail_formation", "preserveNullAndEmptyArrays": True}},
        {"$project": {
            "_id": 0,
            "nom": 1,
            "formation": 1,
            "duree_mois": "$detail_formation.duree_mois",
            "responsable": "$detail_formation.responsable"
        }}
    ]
    return list(collection.aggregate(pipeline))


# Appels
print("=== Stats par formation ===")
for stat in stats_par_formation(etudiants):
    print(f"  {stat['_id']}: {stat['nb_etudiants']} etudiants, moy={stat['moyenne_promo']:.1f}")

print("\n=== Top 5 langages ===")
for lang in top_langages(etudiants):
    print(f"  {lang['langage']}: {lang['nb_etudiants']} etudiants")
```

---

## Embedded Documents vs References

C'est **la** decision de modelisation la plus importante en MongoDB.

### Embedded Documents (imbrication)

```javascript
// Tout dans un seul document
{
  "_id": ObjectId("..."),
  "titre": "Apprendre Python",
  "auteur": {
    "nom": "Alice Martin",
    "email": "alice@example.com",
    "bio": "Developpeuse depuis 10 ans"
  },
  "commentaires": [
    {"user": "Bob", "texte": "Excellent cours !", "date": ISODate("2025-01-15")},
    {"user": "Claire", "texte": "Tres clair.", "date": ISODate("2025-01-16")}
  ]
}
```

**Avantages** : une seule lecture, pas de jointure, atomicite garantie
**Inconvenients** : duplication si l'auteur est dans 100 articles, document peut devenir enorme

### References (normalisation)

```javascript
// Collection articles
{
  "_id": ObjectId("article_001"),
  "titre": "Apprendre Python",
  "auteur_id": ObjectId("user_042"),      // Reference vers la collection users
  "commentaire_ids": [ObjectId("com_1"), ObjectId("com_2")]
}

// Collection users
{
  "_id": ObjectId("user_042"),
  "nom": "Alice Martin",
  "email": "alice@example.com"
}
```

**Avantages** : pas de duplication, documents legers
**Inconvenients** : necessiter un $lookup (jointure), plusieurs lectures

> [!info] Regle de decision
> | Critere | Embedded | Reference |
> |---|---|---|
> | Relation | 1-a-peu | 1-a-beaucoup ou N-a-N |
> | Donnees changent ? | Rarement | Souvent |
> | Lecture typique | Ensemble | Separement |
> | Taille max document | Reste < 16 MB | Illimitee |
> | Exemple | Adresses d'un user | Commandes d'un user |

---

## Schema Design Patterns

### Pattern 1 : Subset Pattern

Quand un document a beaucoup de sous-documents, stocker un **sous-ensemble** dans le document principal et le reste dans une collection separee.

```javascript
// Film avec les 3 meilleurs commentaires inline, les autres dans une collection
{
  "_id": "film_001",
  "titre": "Inception",
  "top_commentaires": [  // Les 3 meilleurs seulement
    {"note": 5, "texte": "Chef-d'oeuvre absolu"},
    {"note": 5, "texte": "Magistral"},
    {"note": 4, "texte": "Tres bien"}
  ],
  "nb_total_commentaires": 15420
}
// + collection separate "commentaires_films" pour les 15417 autres
```

### Pattern 2 : Bucket Pattern (pour les time-series)

Regrouper des mesures temporelles dans des "buckets" pour eviter trop de petits documents.

```javascript
// Plutot que 1 document par mesure de capteur (mauvais)
// On regroupe par heure (bon)
{
  "capteur_id": "temp_001",
  "date_heure": ISODate("2025-01-15T14:00:00Z"),
  "mesures": [
    {"minute": 0, "valeur": 22.3},
    {"minute": 1, "valeur": 22.4},
    // ... 58 autres mesures
    {"minute": 59, "valeur": 22.1}
  ],
  "stats": {"min": 21.8, "max": 22.7, "avg": 22.2}
}
```

### Pattern 3 : Outlier Pattern

Pour les cas exceptionnels (un celebrite avec 10M followers vs un user normal avec 200) :

```javascript
// Document standard
{
  "_id": "user_normal",
  "followers": ["user_1", "user_2", ..., "user_200"],  // Embedded
  "has_overflow": false
}

// Document outlier
{
  "_id": "user_celeb",
  "followers": ["user_1", ..., "user_100"],  // Premier batch seulement
  "has_overflow": true  // Flag indiquant qu'il y a plus dans la collection overflow
}
```

---

## Transactions ACID en MongoDB

Depuis MongoDB 4.0, les **transactions multi-documents** sont disponibles (et multi-collections depuis 4.2).

> [!warning] Performances
> Les transactions ont un cout en performance. Les utiliser uniquement quand l'atomicite est vraiment necessaire. Pour les operations sur un seul document, MongoDB est toujours atomique sans transaction.

```python
from pymongo import MongoClient
from pymongo.errors import PyMongoError

client = MongoClient("mongodb://localhost:27017/")
db = client["ecommerce"]


def transfert_stock(produit_source: str, produit_dest: str, quantite: int) -> bool:
    """
    Transfere du stock d'un produit a un autre.
    Operation atomique : les deux mises a jour reussissent ou aucune.
    """
    with client.start_session() as session:
        try:
            with session.start_transaction():
                # Verifier le stock disponible
                source = db.produits.find_one(
                    {"_id": produit_source, "stock": {"$gte": quantite}},
                    session=session
                )
                if not source:
                    raise ValueError(f"Stock insuffisant pour {produit_source}")
                
                # Debiter le stock source
                db.produits.update_one(
                    {"_id": produit_source},
                    {"$inc": {"stock": -quantite}},
                    session=session
                )
                
                # Crediter le stock destination
                db.produits.update_one(
                    {"_id": produit_dest},
                    {"$inc": {"stock": quantite}},
                    session=session
                )
                
                # Log de l'operation
                db.historique_stocks.insert_one({
                    "source": produit_source,
                    "destination": produit_dest,
                    "quantite": quantite,
                    "timestamp": __import__("datetime").datetime.utcnow()
                }, session=session)
                
                # Si on arrive ici sans exception : commit automatique
                print(f"Transfert de {quantite} unites reussi.")
                return True
                
        except (PyMongoError, ValueError) as e:
            print(f"Transaction annulee : {e}")
            # Le rollback est automatique en cas d'exception
            return False
```

---

## Projection et Pagination avancee

```python
from pymongo import MongoClient, ASCENDING, DESCENDING
from bson import ObjectId


def paginer(collection, filtre: dict, page: int, par_page: int, tri: list = None) -> dict:
    """
    Pagination robuste avec metadata.
    
    Args:
        filtre: Criteres de filtrage MongoDB
        page: Numero de page (commence a 1)
        par_page: Nombre de resultats par page
        tri: Liste de tuples [(champ, direction), ...]
    
    Returns:
        Dict avec 'resultats', 'total', 'page', 'pages_totales'
    """
    if tri is None:
        tri = [("_id", ASCENDING)]
    
    total = collection.count_documents(filtre)
    pages_totales = (total + par_page - 1) // par_page
    
    resultats = list(
        collection.find(filtre, {"_id": 0})
        .sort(tri)
        .skip((page - 1) * par_page)
        .limit(par_page)
    )
    
    return {
        "resultats": resultats,
        "total": total,
        "page": page,
        "par_page": par_page,
        "pages_totales": pages_totales,
        "a_suivant": page < pages_totales,
        "a_precedent": page > 1
    }


def cursor_based_pagination(collection, dernier_id: str | None, limite: int) -> dict:
    """
    Pagination par curseur — plus performante que skip() sur gros volumes.
    Utilise l'_id comme curseur (toujours croissant avec ObjectId).
    """
    filtre = {}
    if dernier_id:
        filtre["_id"] = {"$gt": ObjectId(dernier_id)}
    
    resultats = list(
        collection.find(filtre)
        .sort("_id", ASCENDING)
        .limit(limite)
    )
    
    prochain_curseur = str(resultats[-1]["_id"]) if resultats else None
    
    return {
        "resultats": resultats,
        "prochain_curseur": prochain_curseur,
        "a_suivant": prochain_curseur is not None
    }
```

---

## MongoDB Atlas

MongoDB Atlas est la solution **cloud managee** officielle de MongoDB. Elle elimine la gestion des serveurs et offre des fonctionnalites avancees.

### Connexion a Atlas depuis Python

```python
from pymongo import MongoClient
import os

# La connection string est disponible dans le dashboard Atlas
# Format : mongodb+srv://user:password@cluster.mongodb.net/
ATLAS_URI = os.getenv("MONGODB_ATLAS_URI")

client = MongoClient(
    ATLAS_URI,
    serverSelectionTimeoutMS=5000,
    tls=True,           # TLS/SSL active par defaut sur Atlas
    tlsAllowInvalidCertificates=False
)

db = client["ma_base"]
print("Connecte a Atlas :", client.server_info()["version"])
```

### Atlas Search — recherche full-text

```python
# Atlas offre une recherche full-text avancee via des index de recherche
# Configurer l'index dans le dashboard Atlas, puis utiliser $search

resultats = list(db.articles.aggregate([
    {
        "$search": {
            "index": "articles_search",
            "text": {
                "query": "Python programmation asynchrone",
                "path": ["titre", "contenu"],
                "fuzzy": {"maxEdits": 1}  # Tolerant aux fautes de frappe
            }
        }
    },
    {"$limit": 10},
    {"$project": {"titre": 1, "score": {"$meta": "searchScore"}, "_id": 0}}
]))
```

---

## GridFS : stocker de gros fichiers

MongoDB limite les documents a **16 MB**. GridFS permet de stocker des fichiers plus gros en les decoupant en chunks.

```python
import gridfs
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017/")
db = client["media_storage"]
fs = gridfs.GridFS(db)


def uploader_fichier(chemin_local: str, metadata: dict = None) -> str:
    """Upload un fichier dans GridFS."""
    with open(chemin_local, "rb") as f:
        file_id = fs.put(
            f,
            filename=chemin_local.split("/")[-1],
            metadata=metadata or {}
        )
    return str(file_id)


def telecharger_fichier(file_id: str, chemin_destination: str) -> None:
    """Telecharge un fichier depuis GridFS."""
    from bson import ObjectId
    grid_out = fs.get(ObjectId(file_id))
    with open(chemin_destination, "wb") as f:
        f.write(grid_out.read())
    print(f"Fichier sauvegarde : {chemin_destination}")


def lister_fichiers() -> list:
    """Liste tous les fichiers stockes."""
    return [
        {"id": str(f._id), "nom": f.filename, "taille": f.length}
        for f in fs.find()
    ]


# Exemple d'utilisation
id_image = uploader_fichier("photo_profil.jpg", {"user": "alice", "type": "avatar"})
print(f"Fichier uploade avec l'ID : {id_image}")
telecharger_fichier(id_image, "/tmp/avatar_recupere.jpg")
```

---

## Comparaison MongoDB vs PostgreSQL JSONB

PostgreSQL peut stocker du JSON avec le type JSONB et offre des operations similaires a MongoDB.

> [!info] Quand choisir lequel ?
> | Critere | MongoDB | PostgreSQL JSONB |
> |---|---|---|
> | Schema principal | Flexible (tout JSON) | Mixte (colonnes + JSON) |
> | Requetes relationnelles | $lookup (moins naturel) | JOIN natif (SQL) |
> | Performance JSON | Optimisee | Tres bonne |
> | Transactions | Multi-doc depuis 4.0 | ACID complet toujours |
> | Scalabilite horizontale | Native (sharding) | Via extensions (Citus) |
> | Indexation JSON | Indexes MongoDB | GIN index |
> | Communaute | Grande | Tres grande |

Voir [[01 - Introduction au SQL]] et [[03 - Conception de Bases de Donnees]] pour les cas ou PostgreSQL est preferable.

---

## Exercices Pratiques

### Exercice 1 : Indexation
1. Creez une collection `produits` avec 10 000 documents (utiliser une boucle Python)
2. Mesurez le temps d'une requete sans index (`time.time()`)
3. Creez un index sur le champ `categorie`
4. Mesurez de nouveau — quelle amelioration ?
5. Utilisez `explain("executionStats")` pour confirmer l'utilisation de l'index

### Exercice 2 : Aggregation Pipeline
Avec la collection `etudiants` de l'exercice precedent :
1. Calculez la moyenne generale de chaque etudiant (moyenne de ses notes)
2. Trouvez le langage de programmation le plus populaire
3. Identifiez les etudiants avec une moyenne > 15 ET ayant Python dans leurs langages
4. Generez un rapport : par formation, nombre d'etudiants + moyenne + meilleure note

### Exercice 3 : Schema Design
Vous concevez une plateforme e-commerce. Modelisez :
- `utilisateurs` (profil, adresses multiples)
- `produits` (categories, variantes, stock)
- `commandes` (produits, statut, historique)

Questions :
1. Quels champs mettez-vous en embedded vs reference ?
2. Quels index creez-vous ?
3. Pour quelle operation utiliseriez-vous une transaction ?

### Exercice 4 : Pagination
Implementez une API de pagination pour une collection de 50 000 articles :
1. Methode `skip/limit` avec metadata (page courante, total pages)
2. Methode cursor-based pour les gros volumes
3. Comparez les performances des deux approches sur 50 000 documents

> [!warning] A retenir
> - Un index mal place ralentit les ecritures sans accelerer les lectures utiles — analyser avec `explain()`
> - L'aggregation pipeline = transformation sequentielle de documents, etape par etape
> - Embedded quand les donnees sont toujours lues ensemble ; Reference quand elles evoluent independamment
> - Les transactions MongoDB existent mais ont un cout : les eviter si un seul document suffit
> - GridFS pour les fichiers > 16 MB
> - Voir [[08 - APIs REST avec Flask]] pour integrer MongoDB dans une API REST
