# Introduction au NoSQL

Le **NoSQL** (Not Only SQL) designe une famille de systemes de gestion de bases de donnees qui s'eloignent du modele relationnel classique. Apparus dans les annees 2000 pour repondre aux besoins des geants du web (Google, Amazon, Facebook), ils sont aujourd'hui indispensables a tout developpeur backend.

Ce n'est pas que le SQL soit mauvais — c'est qu'il ne resout pas **tous** les problemes. Le NoSQL repond a des contraintes specifiques : volumes massifs de donnees, flexibilite du schema, scalabilite horizontale, latence ultra-faible.

> [!tip] Analogie
> Pensez au SQL comme a un classeur parfaitement organise avec des dossiers indexes, des formulaires standardises et des regles strictes. Le NoSQL, c'est une boite flexible : vous pouvez y mettre des documents de formes differentes, des listes, des graphes. Moins rigide, mais taille pour certains usages specifiques.

---

## Pourquoi le NoSQL ?

### Les limites du SQL face aux nouveaux usages

Le modele relationnel a ete concu dans les annees 1970 pour des donnees **structurees et stables**. Mais le web moderne genere des donnees :
- **Heterogenes** : un profil utilisateur n'a pas toujours les memes champs
- **Massives** : milliards de documents, petabytes de donnees
- **Distribuees** : reparties sur des dizaines de serveurs dans le monde
- **A haute frequence** : millions de lectures/ecritures par seconde

> [!info] Les problemes concrets
> | Probleme SQL | Impact |
> |---|---|
> | Schema rigide (ALTER TABLE) | Difficile de faire evoluer une structure en production |
> | Scalabilite verticale | Ajouter de la RAM/CPU a un seul serveur a des limites |
> | Jointures couteuses | JOIN sur des milliards de lignes = lenteur |
> | Transactions distribuees | Complexe et lent sur plusieurs serveurs |

### Les avantages cles du NoSQL

1. **Flexibilite du schema** : pas besoin de definir une structure rigide a l'avance
2. **Scalabilite horizontale** : ajouter des serveurs (sharding) plutot que d'upgrader un seul
3. **Performance** : lecture/ecriture optimisees pour des patterns specifiques
4. **Modeles de donnees adaptes** : documents, graphes, paires cle-valeur selon l'usage

---

## Les 4 grands types de bases NoSQL

### 1. Bases Document (Document Stores)

Stockent des **documents JSON/BSON** (ou XML). Chaque document est une entite complete avec sa propre structure.

```json
{
  "_id": "user_42",
  "nom": "Alice Martin",
  "email": "alice@example.com",
  "adresses": [
    {"type": "domicile", "ville": "Paris", "cp": "75011"},
    {"type": "travail", "ville": "Lyon", "cp": "69001"}
  ],
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

**Exemples** : MongoDB, CouchDB, Firestore
**Cas d'usage** : catalogues produits, profils utilisateurs, CMS, applications mobiles

---

### 2. Bases Cle-Valeur (Key-Value Stores)

La forme la plus simple : une **cle unique** pointe vers une **valeur** (string, number, blob, structure serialisee).

```
cle                    →  valeur
"session:user42"       →  {"token": "abc123", "expires": 1700000000}
"counter:page_views"   →  42891
"cache:product:789"    →  "<html>...</html>"
```

**Exemples** : Redis, DynamoDB, Memcached
**Cas d'usage** : cache, sessions utilisateur, file d'attente, rate limiting

---

### 3. Bases Colonne (Column Stores / Wide-Column)

Organisent les donnees en **familles de colonnes** plutot qu'en lignes. Optimisees pour les requetes analytiques sur des colonnes specifiques de milliards de lignes.

```
Row Key     │ info:nom      │ info:email         │ analytics:vues │
────────────┼───────────────┼────────────────────┼────────────────┤
"user:42"   │ "Alice"       │ "alice@example.com" │ 1520           │
"user:43"   │ "Bob"         │ "bob@example.com"   │ 892            │
```

**Exemples** : Apache Cassandra, HBase, Google Bigtable
**Cas d'usage** : time-series, IoT, analytics, logs a haute frequence

---

### 4. Bases Graphe (Graph Databases)

Modelisent les donnees comme un **graphe** : noeuds (entites) et aretes (relations). Optimisees pour les requetes traversant de nombreuses relations.

```
(Alice) --[AMIS_AVEC]--> (Bob)
(Alice) --[TRAVAILLE_A]--> (TechCorp)
(Bob)   --[AIME]--> (Python)
(Alice) --[AIME]--> (Python)
```

**Exemples** : Neo4j, Amazon Neptune, ArangoDB
**Cas d'usage** : reseaux sociaux, moteurs de recommandation, detection de fraude, cartographie

---

## Comparaison SQL vs NoSQL

> [!info] Tableau comparatif
> | Critere | SQL (Relationnel) | NoSQL |
> |---|---|---|
> | **Schema** | Fixe, declare a l'avance | Flexible, evolutif |
> | **Scalabilite** | Verticale (+ gros serveur) | Horizontale (+ serveurs) |
> | **Transactions ACID** | Oui, natives | Variable selon la base |
> | **Langage de requete** | SQL standardise | Specifique a chaque base |
> | **Relations** | Jointures (JOIN) | Denormalisation / embed |
> | **Coherence** | Forte (ACID) | Eventuelle souvent |
> | **Cas d'usage** | Donnees structurees, transactions | Big data, flexibilite, perf |

> [!warning] Le mythe SQL vs NoSQL
> Ce n'est pas un combat. La plupart des applications modernes utilisent **les deux** :
> - PostgreSQL pour les donnees metier critiques (commandes, paiements)
> - Redis pour le cache et les sessions
> - MongoDB pour les contenus flexibles (articles, produits)
> - Elasticsearch pour la recherche full-text

---

## Le theoreme CAP

Tout systeme distribue doit faire des compromis entre trois proprietes :

```
           Consistance (C)
          /               \
         /   Impossible    \
        /   d'avoir les 3  \
       /    simultanement   \
      /                     \
Disponibilite (A) --------- Tolerance (P)
```

- **C - Consistency** : chaque lecture recoit la donnee la plus recente ou une erreur
- **A - Availability** : chaque requete recoit une reponse (peut etre perimee)
- **P - Partition Tolerance** : le systeme fonctionne meme si des noeuds sont coupes

> [!info] Positionnement des bases
> | Base | CAP choisi | Compromis |
> |---|---|---|
> | MongoDB | CP | Peut etre indisponible en cas de partition |
> | Cassandra | AP | Peut retourner des donnees perimees |
> | Redis | CP | Priorite a la consistance |
> | DynamoDB | AP | Disponibilite maximale |
> | PostgreSQL | CA | Pas tolerable aux partitions (mono-noeud) |

---

## MongoDB : Introduction

MongoDB est la base de donnees document la plus populaire. Elle stocke les donnees en **BSON** (Binary JSON), un format binaire proche du JSON.

### Concepts fondamentaux

```
SQL              →  MongoDB
─────────────────────────────
Base de donnees  →  Database
Table            →  Collection
Ligne            →  Document
Colonne          →  Field (champ)
JOIN             →  Embedded document / $lookup
PRIMARY KEY      →  _id (genere automatiquement)
```

### Installation

**Methode recommandee pour le cours (Docker)** :
```bash
# Lancer MongoDB dans Docker
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  mongo:7.0

# Verifier que le conteneur tourne
docker ps
```

**Installation directe (Ubuntu/Debian)** :
```bash
# Importer la cle GPG
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Ajouter le depot
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Installer
sudo apt-get update && sudo apt-get install -y mongodb-org

# Demarrer le service
sudo systemctl start mongod
```

---

## CRUD avec mongosh (shell interactif)

```bash
# Se connecter au shell MongoDB
mongosh "mongodb://localhost:27017"

# Ou avec authentification
mongosh "mongodb://admin:password@localhost:27017"
```

### Create - Inserer des documents

```javascript
// Selectionner (ou creer) une base de donnees
use holberton_school

// Inserer un document
db.etudiants.insertOne({
  nom: "Alice Martin",
  age: 22,
  formation: "FullStack",
  langages: ["Python", "JavaScript", "C"],
  notes: {
    algorithmique: 18,
    web: 16,
    systeme: 14
  },
  date_inscription: new Date()
})

// Inserer plusieurs documents
db.etudiants.insertMany([
  {
    nom: "Bob Dupont",
    age: 24,
    formation: "DevOps",
    langages: ["Python", "Bash", "Go"],
    notes: { algorithmique: 15, web: 12, systeme: 19 }
  },
  {
    nom: "Claire Leclerc",
    age: 21,
    formation: "FullStack",
    langages: ["JavaScript", "TypeScript", "Python"],
    notes: { algorithmique: 17, web: 19, systeme: 13 }
  }
])
```

### Read - Lire des documents

```javascript
// Lire tous les documents
db.etudiants.find()

// Lire avec un filtre
db.etudiants.find({ formation: "FullStack" })

// Filtre avance : age > 21 ET formation FullStack
db.etudiants.find({
  age: { $gt: 21 },
  formation: "FullStack"
})

// Operateurs de comparaison
// $gt: greater than (>)      $gte: >= 
// $lt: less than (<)         $lte: <=
// $eq: egal (=)              $ne: different (!=)
// $in: dans une liste        $nin: pas dans une liste

// Trouver les etudiants qui connaissent Python
db.etudiants.find({ langages: "Python" })

// Projection : selectionner uniquement certains champs (1 = inclure, 0 = exclure)
db.etudiants.find(
  { formation: "FullStack" },
  { nom: 1, "notes.web": 1, _id: 0 }
)

// Trier par age decroissant
db.etudiants.find().sort({ age: -1 })

// Pagination : sauter 2, prendre 5
db.etudiants.find().skip(2).limit(5)

// Compter
db.etudiants.countDocuments({ formation: "FullStack" })

// Trouver un seul document
db.etudiants.findOne({ nom: "Alice Martin" })
```

### Update - Modifier des documents

```javascript
// Modifier un document (le premier trouve)
db.etudiants.updateOne(
  { nom: "Alice Martin" },                    // filtre
  { $set: { age: 23, formation: "DevOps" } } // modification
)

// Incrementer un champ numerique
db.etudiants.updateOne(
  { nom: "Bob Dupont" },
  { $inc: { age: 1 } }
)

// Ajouter un element a un tableau
db.etudiants.updateOne(
  { nom: "Claire Leclerc" },
  { $push: { langages: "Rust" } }
)

// Modifier plusieurs documents
db.etudiants.updateMany(
  { formation: "FullStack" },
  { $set: { statut: "actif" } }
)

// Upsert : creer si n'existe pas
db.etudiants.updateOne(
  { nom: "David Martin" },
  { $set: { age: 25, formation: "Data" } },
  { upsert: true }
)
```

### Delete - Supprimer des documents

```javascript
// Supprimer un document
db.etudiants.deleteOne({ nom: "David Martin" })

// Supprimer plusieurs documents
db.etudiants.deleteMany({ formation: "DevOps" })

// Supprimer tous les documents d'une collection (attention !)
db.etudiants.deleteMany({})

// Supprimer une collection entiere
db.etudiants.drop()
```

---

## CRUD avec Python (pymongo)

### Installation

```bash
pip install pymongo
```

### Connexion et configuration

```python
from pymongo import MongoClient
from pymongo.errors import ConnectionFailure, DuplicateKeyError
import os
from datetime import datetime

def get_mongo_client():
    """Cree et retourne un client MongoDB."""
    host = os.getenv("MONGO_HOST", "localhost")
    port = int(os.getenv("MONGO_PORT", 27017))
    username = os.getenv("MONGO_USER", None)
    password = os.getenv("MONGO_PASSWORD", None)
    
    if username and password:
        uri = f"mongodb://{username}:{password}@{host}:{port}/"
    else:
        uri = f"mongodb://{host}:{port}/"
    
    try:
        client = MongoClient(uri, serverSelectionTimeoutMS=5000)
        # Verifier la connexion
        client.admin.command('ping')
        print("Connexion MongoDB etablie.")
        return client
    except ConnectionFailure as e:
        print(f"Impossible de se connecter a MongoDB : {e}")
        raise

# Utilisation
client = get_mongo_client()
db = client["holberton_school"]
etudiants = db["etudiants"]
```

### Create avec pymongo

```python
def inserer_etudiant(collection, etudiant: dict) -> str:
    """Insere un etudiant et retourne son _id."""
    etudiant["date_inscription"] = datetime.utcnow()
    result = collection.insert_one(etudiant)
    return str(result.inserted_id)

def inserer_etudiants(collection, liste: list) -> list:
    """Insere plusieurs etudiants."""
    result = collection.insert_many(liste)
    return [str(id) for id in result.inserted_ids]

# Exemple d'utilisation
nouvel_etudiant = {
    "nom": "Emma Bernard",
    "age": 23,
    "formation": "FullStack",
    "langages": ["Python", "JavaScript"],
    "notes": {"algorithmique": 17, "web": 18, "systeme": 14}
}

id_insere = inserer_etudiant(etudiants, nouvel_etudiant)
print(f"Document insere avec l'ID : {id_insere}")
```

### Read avec pymongo

```python
from bson import ObjectId

def trouver_tous(collection) -> list:
    """Retourne tous les documents."""
    return list(collection.find({}, {"_id": 0}))  # Exclure _id

def trouver_par_formation(collection, formation: str) -> list:
    """Retourne les etudiants d'une formation donnee."""
    return list(collection.find(
        {"formation": formation},
        {"nom": 1, "age": 1, "notes": 1, "_id": 0}
    ).sort("nom", 1))  # Trier par nom ASC

def trouver_par_id(collection, doc_id: str) -> dict | None:
    """Retourne un document par son _id."""
    return collection.find_one({"_id": ObjectId(doc_id)})

def rechercher_avance(collection, age_min: int, langages: list) -> list:
    """Recherche avec plusieurs criteres."""
    filtre = {
        "age": {"$gte": age_min},
        "langages": {"$in": langages}
    }
    return list(collection.find(filtre, {"_id": 0}))

# Exemples
tous = trouver_tous(etudiants)
fullstack = trouver_par_formation(etudiants, "FullStack")
pythoniens = rechercher_avance(etudiants, 21, ["Python"])

for etudiant in fullstack:
    print(f"  {etudiant['nom']} - {etudiant['notes']}")
```

### Update avec pymongo

```python
def mettre_a_jour_etudiant(collection, nom: str, mises_a_jour: dict) -> int:
    """Met a jour un etudiant. Retourne le nombre de documents modifies."""
    result = collection.update_one(
        {"nom": nom},
        {"$set": mises_a_jour}
    )
    return result.modified_count

def ajouter_langage(collection, nom: str, langage: str) -> int:
    """Ajoute un langage a la liste d'un etudiant."""
    result = collection.update_one(
        {"nom": nom},
        {"$push": {"langages": langage}}
    )
    return result.modified_count

def mettre_a_jour_tous(collection, formation: str, champs: dict) -> int:
    """Met a jour tous les etudiants d'une formation."""
    result = collection.update_many(
        {"formation": formation},
        {"$set": champs}
    )
    return result.modified_count

# Exemples
nb_modifies = mettre_a_jour_etudiant(etudiants, "Emma Bernard", {"age": 24})
print(f"{nb_modifies} document(s) mis a jour.")

nb_modifies = ajouter_langage(etudiants, "Emma Bernard", "TypeScript")
print(f"Langage ajoute : {nb_modifies} document(s) modifie(s).")
```

### Delete avec pymongo

```python
def supprimer_etudiant(collection, nom: str) -> int:
    """Supprime un etudiant par son nom. Retourne le nombre de suppressions."""
    result = collection.delete_one({"nom": nom})
    return result.deleted_count

def supprimer_par_formation(collection, formation: str) -> int:
    """Supprime tous les etudiants d'une formation."""
    result = collection.delete_many({"formation": formation})
    return result.deleted_count

# Exemple
nb = supprimer_etudiant(etudiants, "Emma Bernard")
print(f"{nb} document(s) supprime(s).")
```

---

## Script complet : exemple de projet

```python
"""
Exemple complet : gestion d'un catalogue de livres avec pymongo.
"""
from pymongo import MongoClient, ASCENDING, DESCENDING
from datetime import datetime
import pprint


def main():
    client = MongoClient("mongodb://localhost:27017/")
    db = client["librairie"]
    livres = db["livres"]

    # Nettoyer pour l'exemple
    livres.drop()

    # Inserer des livres
    livres.insert_many([
        {
            "titre": "Fluent Python",
            "auteur": "Luciano Ramalho",
            "annee": 2022,
            "genre": "Programmation",
            "prix": 45.99,
            "tags": ["python", "avance", "idiomes"],
            "disponible": True
        },
        {
            "titre": "The Clean Coder",
            "auteur": "Robert C. Martin",
            "annee": 2011,
            "genre": "Genie logiciel",
            "prix": 32.50,
            "tags": ["pratiques", "professionnalisme"],
            "disponible": True
        },
        {
            "titre": "Designing Data-Intensive Applications",
            "auteur": "Martin Kleppmann",
            "annee": 2017,
            "genre": "Architecture",
            "prix": 55.00,
            "tags": ["databases", "distributed", "NoSQL"],
            "disponible": False
        }
    ])

    print("=== Tous les livres disponibles ===")
    for livre in livres.find({"disponible": True}, {"_id": 0}).sort("prix", ASCENDING):
        print(f"  [{livre['prix']}€] {livre['titre']} — {livre['auteur']}")

    print("\n=== Livres avec 'python' dans les tags ===")
    for livre in livres.find({"tags": "python"}, {"titre": 1, "auteur": 1, "_id": 0}):
        print(f"  {livre['titre']}")

    print("\n=== Prix moyen par genre ===")
    pipeline = [
        {"$group": {"_id": "$genre", "prix_moyen": {"$avg": "$prix"}}},
        {"$sort": {"prix_moyen": -1}}
    ]
    for result in livres.aggregate(pipeline):
        print(f"  {result['_id']}: {result['prix_moyen']:.2f}€")

    client.close()


if __name__ == "__main__":
    main()
```

---

## Cas d'usage : quand choisir MongoDB ?

> [!tip] Choisir MongoDB quand...
> - Les documents ont des structures variables (profils, articles, produits)
> - Le schema evolue frequemment pendant le developpement
> - Les donnees sont naturellement hierarchiques (embedded documents)
> - Vous avez besoin de scalabilite horizontale
> - La recherche se fait principalement dans un seul document

> [!warning] Eviter MongoDB quand...
> - Vous avez beaucoup de relations complexes entre entites
> - Les transactions ACID multi-collection sont critiques
> - Les donnees sont tres structurees et stables
> - Votre equipe maitrise mieux SQL (le migration cost n'en vaut pas la peine)
> - Voir [[01 - Introduction au SQL]] pour les cas ou SQL est preferable

---

## Exercices Pratiques

### Exercice 1 : Premier pas avec mongosh
1. Installez MongoDB (ou utilisez Docker)
2. Creez une base `ma_bibliotheque` avec une collection `livres`
3. Inserez 5 livres avec les champs : titre, auteur, annee, genre, note_perso (float)
4. Listez tous les livres publies apres 2015
5. Mettez a jour la note de 3 livres
6. Supprimez le livre avec la note la plus basse

### Exercice 2 : CRUD en Python
Ecrivez un script Python `book_manager.py` qui :
1. Se connecte a MongoDB via pymongo
2. Implemente 5 fonctions : `add_book()`, `get_all_books()`, `update_book()`, `delete_book()`, `search_by_genre()`
3. Chaque fonction doit gerer les erreurs (document non trouve, connexion echouee)
4. Affiche les resultats de maniere lisible

### Exercice 3 : Modelisation
Vous developpez une app de blog. Modelisez en MongoDB :
- Des **articles** avec titre, contenu, auteur, date, tags
- Des **commentaires** (embedded dans les articles OU collection separee ?)
- Des **utilisateurs** avec profil et articles ecrits

Justifiez votre choix de modelisation (embedded vs reference). Consultez [[02 - MongoDB Avance]] pour les criteres de decision.

### Exercice 4 : CAP Theorem
Recherchez et expliquez :
1. Pourquoi MongoDB est classe CP et non AP
2. Quel scenario concret rendrait MongoDB temporairement indisponible (CAP)
3. Donnez un exemple d'application ou AP serait preferable a CP

> [!warning] A retenir
> - NoSQL ne remplace pas SQL — chaque outil a son domaine d'excellence
> - MongoDB = base document, schema flexible, scalabilite horizontale
> - Les 4 types NoSQL : Document, Cle-Valeur, Colonne, Graphe
> - Le theoreme CAP : tout systeme distribue choisit 2 proprietes sur 3
> - pymongo est le driver Python officiel — toujours fermer la connexion (`client.close()`)
