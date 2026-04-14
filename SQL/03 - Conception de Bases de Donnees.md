# Conception de Bases de Donnees

La conception d'une base de donnees est l'etape la plus critique d'un projet. Une base bien concue est performante, maintenable et evolutive. Une base mal concue entraine des anomalies de donnees, des performances degradees et une dette technique considerable. Ce cours couvre les principes fondamentaux : modelisation entite-relation, normalisation, cles, indexes et contraintes d'integrite.

Avant d'ecrire la moindre requete SQL, il faut **penser la structure**. Ce cours vous donne les outils pour le faire correctement.

> [!tip] Analogie
> Concevoir une base de donnees, c'est comme concevoir les **plans d'un immeuble** avant de le construire. On ne commence pas par couler le beton : on definit d'abord les pieces (tables), les portes entre elles (relations), et on s'assure que la structure est solide (normalisation). Modifier les fondations une fois l'immeuble construit est extremement couteux — tout comme restructurer une base en production.

---

## 1. Pourquoi la Conception est Essentielle

### Les problemes d'une mauvaise conception

```
┌──────────────────────────────────────────────────────────┐
│           PROBLEMES D'UNE BASE MAL CONCUE                │
│                                                          │
│  1. Redondance des donnees                               │
│     → Meme information stockee a plusieurs endroits      │
│     → Gaspillage d'espace, incoherences                  │
│                                                          │
│  2. Anomalies de mise a jour                             │
│     → Modifier une info oblige a toucher N lignes        │
│     → Risque d'oublier une occurrence                    │
│                                                          │
│  3. Anomalies de suppression                             │
│     → Supprimer une ligne peut perdre des infos utiles   │
│                                                          │
│  4. Anomalies d'insertion                                │
│     → Impossible d'inserer une info sans donnees liees   │
│                                                          │
│  5. Performances degradees                               │
│     → Requetes lentes, indexes inutiles ou manquants     │
└──────────────────────────────────────────────────────────┘
```

> [!example] Exemple de mauvaise conception
> ```
> Table: commandes_mauvaise
> +----+--------+--------+---------+----------+------------+
> | id | client | email  | produit | prix     | date       |
> +----+--------+--------+---------+----------+------------+
> |  1 | Alice  | a@m.fr | Laptop  | 999.99   | 2025-01-10 |
> |  2 | Alice  | a@m.fr | Souris  |  29.99   | 2025-01-10 |
> |  3 | Bob    | b@m.fr | Laptop  | 999.99   | 2025-01-12 |
> +----+--------+--------+---------+----------+------------+
> ```
> Problemes : "Alice" et "a@m.fr" sont repetes. Si Alice change d'email, il faut mettre a jour toutes les lignes. Le prix du "Laptop" est repete — et s'il change ?

---

## 2. Entites et Relations

### 2.1 Identifier les entites

Une **entite** est un objet ou concept distinct que l'on souhaite stocker. Chaque entite devient une table.

```
Domaine : Universite

Entites identifiees :
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Etudiant    │  │    Cours     │  │  Professeur  │
│              │  │              │  │              │
│ - id         │  │ - id         │  │ - id         │
│ - nom        │  │ - code       │  │ - nom        │
│ - prenom     │  │ - nom        │  │ - specialite │
│ - email      │  │ - credits    │  │ - email      │
│ - date_naiss │  │ - departement│  │              │
└──────────────┘  └──────────────┘  └──────────────┘
```

### 2.2 Identifier les relations

Les **relations** decrivent comment les entites interagissent entre elles.

```
Etudiant ──< s'inscrit a >── Cours        (N:M)
Professeur ──< enseigne >── Cours          (1:N)
Cours ──< appartient a >── Departement     (N:1)
```

---

## 3. Diagrammes Entite-Relation (ER)

### 3.1 Notation

```
┌─────────────────────────────────────────────────┐
│       NOTATION DES DIAGRAMMES ER                │
│                                                 │
│  ┌──────────┐                                   │
│  │ Entite   │  Rectangle = Table                │
│  └──────────┘                                   │
│                                                 │
│  ──────────── Ligne = Relation                  │
│                                                 │
│  Cardinalites :                                 │
│    ──|────  1 (exactement un)                   │
│    ──|──<   1:N (un vers plusieurs)             │
│    ──>──<   N:M (plusieurs vers plusieurs)      │
│    ──o────  0 ou 1 (optionnel)                  │
│    ──o──<   0 ou plusieurs                      │
│                                                 │
│  PK = Primary Key (cle primaire)                │
│  FK = Foreign Key (cle etrangere)               │
└─────────────────────────────────────────────────┘
```

### 3.2 Exemple : Systeme universitaire

```
┌──────────────┐       ┌─────────────────┐       ┌──────────────┐
│  etudiants   │       │  inscriptions   │       │    cours     │
│──────────────│       │─────────────────│       │──────────────│
│ PK id        │──|──<─│ PK id           │─>──|──│ PK id        │
│    prenom    │       │ FK etudiant_id  │       │    code      │
│    nom       │       │ FK cours_id     │       │    nom       │
│    email     │       │    note         │       │    credits   │
│    age       │       │    date_inscr   │       │ FK prof_id   │
└──────────────┘       └─────────────────┘       └──────┬───────┘
                                                        │
                                                   ─>──|──
                                                        │
                                                 ┌──────▼───────┐
                                                 │ professeurs  │
                                                 │──────────────│
                                                 │ PK id        │
                                                 │    nom       │
                                                 │    prenom    │
                                                 │    specialite│
                                                 └──────────────┘
```

---

## 4. Cardinalite des Relations

### 4.1 Relation 1:1 (un a un)

Chaque entite A est liee a exactement une entite B, et inversement.

```sql
-- Exemple : chaque employe a un seul badge, et chaque badge appartient a un seul employe
CREATE TABLE employes (
    id      INT PRIMARY KEY AUTO_INCREMENT,
    nom     VARCHAR(50) NOT NULL,
    prenom  VARCHAR(50) NOT NULL
);

CREATE TABLE badges (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    code        VARCHAR(20) UNIQUE NOT NULL,
    employe_id  INT UNIQUE NOT NULL,  -- UNIQUE garantit le 1:1
    FOREIGN KEY (employe_id) REFERENCES employes(id)
);
```

> [!info] Quand utiliser 1:1 ?
> - Separer des donnees rarement utilisees (optimisation)
> - Informations sensibles dans une table separee (securite)
> - Extensions optionnelles d'une entite
> Dans beaucoup de cas, une relation 1:1 peut etre fusionnee en une seule table.

### 4.2 Relation 1:N (un a plusieurs)

L'entite la plus courante. Un departement a plusieurs employes, mais un employe n'a qu'un seul departement.

```sql
CREATE TABLE departements (
    id   INT PRIMARY KEY AUTO_INCREMENT,
    nom  VARCHAR(50) NOT NULL
);

CREATE TABLE employes (
    id              INT PRIMARY KEY AUTO_INCREMENT,
    nom             VARCHAR(50) NOT NULL,
    departement_id  INT,
    FOREIGN KEY (departement_id) REFERENCES departements(id)
);
-- La cle etrangere est dans la table du cote "N"
```

```
┌──────────────┐         ┌──────────────┐
│ departements │──|──<───│  employes    │
│              │  1    N │              │
│ PK id        │         │ PK id        │
│    nom       │         │    nom       │
│              │         │ FK dept_id   │
└──────────────┘         └──────────────┘
```

### 4.3 Relation N:M (plusieurs a plusieurs)

Un etudiant peut s'inscrire a plusieurs cours, et un cours peut avoir plusieurs etudiants. On utilise une **table de jonction** (junction table / table associative).

```sql
CREATE TABLE etudiants (
    id     INT PRIMARY KEY AUTO_INCREMENT,
    nom    VARCHAR(50) NOT NULL
);

CREATE TABLE cours (
    id     INT PRIMARY KEY AUTO_INCREMENT,
    nom    VARCHAR(100) NOT NULL
);

-- Table de jonction
CREATE TABLE inscriptions (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    etudiant_id INT NOT NULL,
    cours_id    INT NOT NULL,
    note        DECIMAL(4,2),
    FOREIGN KEY (etudiant_id) REFERENCES etudiants(id),
    FOREIGN KEY (cours_id)    REFERENCES cours(id),
    UNIQUE (etudiant_id, cours_id)  -- empeche les doublons
);
```

```
┌──────────┐       ┌──────────────┐       ┌──────────┐
│etudiants │──|──<─│ inscriptions │─>──|──│  cours   │
│          │ 1   N │              │ N    1 │          │
│ PK id    │       │ PK id        │       │ PK id    │
│    nom   │       │ FK etud_id   │       │    nom   │
│          │       │ FK cours_id  │       │          │
│          │       │    note      │       │          │
└──────────┘       └──────────────┘       └──────────┘
```

> [!tip] Table de jonction
> La table de jonction peut porter des **attributs propres** a la relation.
> Par exemple, `inscriptions` contient la `note` — qui n'appartient ni a l'etudiant seul ni au cours seul, mais a leur combinaison.

---

## 5. Cles Primaires et Etrangeres

### 5.1 Cle primaire (PRIMARY KEY)

Identifiant **unique** et **non NULL** de chaque ligne dans une table.

```
+------------------+-------------------------------------------+
| Type             | Description                               |
+------------------+-------------------------------------------+
| Cle naturelle    | Attribut existant (email, code ISBN,      |
|                  | numero secu). Risque de changement.       |
+------------------+-------------------------------------------+
| Cle surrogate    | ID genere artificiellement (AUTO_INCREMENT|
| (artificielle)   | ou UUID). Stable, performant.             |
+------------------+-------------------------------------------+
| Cle composee     | Combinaison de colonnes                   |
|                  | (etudiant_id + cours_id)                  |
+------------------+-------------------------------------------+
```

> [!tip] Naturelle vs Surrogate
> Dans la majorite des cas, preferez une **cle surrogate** (INT AUTO_INCREMENT ou UUID) :
> - Les cles naturelles peuvent changer (email, nom, code postal)
> - Les cles surrogate sont stables et performantes pour les jointures
> - Exception : tables de reference avec des codes stables (code pays ISO, code devise)

### 5.2 Cle etrangere (FOREIGN KEY)

Une cle etrangere cree un lien entre deux tables. Elle garantit l'**integrite referentielle** : on ne peut pas referencer un enregistrement qui n'existe pas.

```sql
CREATE TABLE commandes (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    client_id   INT NOT NULL,
    date_commande DATE NOT NULL,

    -- Syntaxe complete avec actions de reference
    CONSTRAINT fk_commandes_clients
        FOREIGN KEY (client_id)
        REFERENCES clients(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

### 5.3 Actions de reference (ON DELETE / ON UPDATE)

```
+-------------+---------------------------------------------+
| Action      | Comportement                                |
+-------------+---------------------------------------------+
| CASCADE     | Supprime/modifie les lignes enfants         |
|             | automatiquement                             |
+-------------+---------------------------------------------+
| SET NULL    | Met la cle etrangere a NULL dans les        |
|             | lignes enfants                              |
+-------------+---------------------------------------------+
| SET DEFAULT | Met la cle etrangere a la valeur par defaut |
+-------------+---------------------------------------------+
| RESTRICT    | Empeche la suppression/modification si des  |
|             | lignes enfants existent (par defaut)        |
+-------------+---------------------------------------------+
| NO ACTION   | Similaire a RESTRICT (verification differee)|
+-------------+---------------------------------------------+
```

```sql
-- CASCADE : supprimer un auteur supprime tous ses livres
CREATE TABLE livres (
    id        INT PRIMARY KEY AUTO_INCREMENT,
    titre     VARCHAR(200) NOT NULL,
    auteur_id INT NOT NULL,
    FOREIGN KEY (auteur_id) REFERENCES auteurs(id)
        ON DELETE CASCADE
);

-- SET NULL : supprimer un departement met les employes a NULL
CREATE TABLE employes (
    id              INT PRIMARY KEY AUTO_INCREMENT,
    nom             VARCHAR(50) NOT NULL,
    departement_id  INT,
    FOREIGN KEY (departement_id) REFERENCES departements(id)
        ON DELETE SET NULL
);

-- RESTRICT : empeche de supprimer un client qui a des commandes
CREATE TABLE commandes (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    client_id   INT NOT NULL,
    FOREIGN KEY (client_id) REFERENCES clients(id)
        ON DELETE RESTRICT
);
```

> [!warning] Choisir la bonne action
> - **CASCADE** : quand l'enfant n'a pas de sens sans le parent (commentaires d'un post supprime)
> - **SET NULL** : quand l'enfant peut exister sans le parent (employe sans departement)
> - **RESTRICT** : quand la suppression doit etre explicitement geree (client avec des commandes)
> En cas de doute, utilisez RESTRICT (le defaut) — c'est le plus sur.

---

## 6. Normalisation

La normalisation est le processus de structuration d'une base pour **eliminer la redondance** et les **anomalies de donnees**.

### 6.1 Premiere Forme Normale (1NF)

> Chaque cellule contient une **valeur atomique** (indivisible). Pas de listes, pas de groupes repetitifs.

```
AVANT (viole 1NF) :
+----+-------+-----------------------------+
| id | nom   | telephones                  |
+----+-------+-----------------------------+
|  1 | Alice | 0601020304, 0611223344      |  ← Liste !
|  2 | Bob   | 0655667788                  |
+----+-------+-----------------------------+

APRES (1NF respectee) :

Option 1 : Colonnes separees (si nombre fixe et petit)
+----+-------+--------------+--------------+
| id | nom   | tel_principal| tel_second   |
+----+-------+--------------+--------------+
|  1 | Alice | 0601020304   | 0611223344   |
|  2 | Bob   | 0655667788   | NULL         |
+----+-------+--------------+--------------+

Option 2 : Table separee (recommande si nombre variable)
Table: contacts          Table: telephones
+----+-------+          +----+------------+------------+
| id | nom   |          | id | contact_id | numero     |
+----+-------+          +----+------------+------------+
|  1 | Alice |          |  1 |     1      | 0601020304 |
|  2 | Bob   |          |  2 |     1      | 0611223344 |
+----+-------+          |  3 |     2      | 0655667788 |
                        +----+------------+------------+
```

### 6.2 Deuxieme Forme Normale (2NF)

> En 1NF + chaque attribut non-cle depend de **toute** la cle primaire (pas de dependance partielle). S'applique uniquement aux tables avec une cle composee.

```
AVANT (viole 2NF) :
Cle composee : (etudiant_id, cours_id)

+-------------+----------+------+------------------+----------+
| etudiant_id | cours_id | note | nom_etudiant     | nom_cours|
+-------------+----------+------+------------------+----------+
|      1      |    101   | 15   | Alice Dupont     | SQL      |
|      1      |    102   | 18   | Alice Dupont     | Python   |
|      2      |    101   | 12   | Bob Martin       | SQL      |
+-------------+----------+------+------------------+----------+

Probleme :
- nom_etudiant depend SEULEMENT de etudiant_id (dependance partielle)
- nom_cours depend SEULEMENT de cours_id (dependance partielle)

APRES (2NF respectee) : 3 tables separees

etudiants:                 cours:                 inscriptions:
+----+--------------+     +-----+--------+       +--------+--------+------+
| id | nom          |     | id  | nom    |       | etu_id | crs_id | note |
+----+--------------+     +-----+--------+       +--------+--------+------+
|  1 | Alice Dupont |     | 101 | SQL    |       |   1    |  101   |  15  |
|  2 | Bob Martin   |     | 102 | Python |       |   1    |  102   |  18  |
+----+--------------+     +-----+--------+       |   2    |  101   |  12  |
                                                  +--------+--------+------+
```

### 6.3 Troisieme Forme Normale (3NF)

> En 2NF + aucun attribut non-cle ne depend d'un autre attribut non-cle (pas de dependance transitive).

```
AVANT (viole 3NF) :
+----+-------+----------------+--------------+
| id | nom   | departement_id | dept_ville   |
+----+-------+----------------+--------------+
|  1 | Alice |       1        | Paris        |
|  2 | Bob   |       1        | Paris        |
|  3 | Eve   |       2        | Lyon         |
+----+-------+----------------+--------------+

Probleme : dept_ville depend de departement_id, pas directement de id
  id → departement_id → dept_ville  (dependance transitive)
  Si Paris devient "Paris Cedex", il faut modifier toutes les lignes.

APRES (3NF respectee) :

employes:                     departements:
+----+-------+--------+     +----+-----------+--------+
| id | nom   | dep_id |     | id | nom       | ville  |
+----+-------+--------+     +----+-----------+--------+
|  1 | Alice |   1    |     |  1 | Ingenierie| Paris  |
|  2 | Bob   |   1    |     |  2 | Marketing | Lyon   |
|  3 | Eve   |   2    |     +----+-----------+--------+
+----+-------+--------+
```

> [!info] Faut-il toujours normaliser en 3NF ?
> En general, oui. Mais il existe des cas ou une **denormalisation volontaire** est justifiee :
> - **Performance de lecture** : eviter des jointures couteuses sur de tres grandes tables
> - **Data warehousing** : les schemas en etoile (star schema) sont volontairement denormalises
> - **Caching** : stocker un total precalcule plutot que de le recalculer a chaque fois
>
> La regle : **normaliser d'abord, denormaliser ensuite si et seulement si mesure de performance le justifie**.

---

## 7. Les Index

Un index est une structure de donnees qui accelere la recherche dans une table, au prix d'un cout supplementaire en ecriture et en espace disque.

### 7.1 Concept du B-tree

```
Sans index (full table scan) :
┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
│ 1│ 2│ 3│ 4│ 5│ 6│ 7│ 8│ 9│10│  → Parcourir TOUTES les lignes
└──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘    pour trouver id = 7
Complexite : O(n)

Avec index B-tree :
                    ┌─────┐
                    │  5  │        Niveau 1 : 7 > 5 → droite
                    └──┬──┘
              ┌────────┴────────┐
           ┌──┴──┐          ┌──┴──┐
           │ 2,3 │          │ 7,8 │  Niveau 2 : 7 trouve !
           └──┬──┘          └──┬──┘
          ┌───┴───┐        ┌───┴───┐
        ┌─┴─┐   ┌─┴─┐   ┌─┴─┐   ┌─┴─┐
        │1,2│   │3,4│   │5,6│   │7,8│  Feuilles → donnees
        └───┘   └───┘   └───┘   └───┘
Complexite : O(log n)

Pour 1 million de lignes :
- Sans index : ~1 000 000 comparaisons
- Avec index : ~20 comparaisons (log2(1M) ≈ 20)
```

### 7.2 Creer des index

```sql
-- Index simple sur une colonne
CREATE INDEX idx_employes_nom ON employes(nom);

-- Index unique (garantit l'unicite)
CREATE UNIQUE INDEX idx_employes_email ON employes(email);

-- Index composite (plusieurs colonnes)
CREATE INDEX idx_employes_dept_nom ON employes(departement_id, nom);

-- Supprimer un index
DROP INDEX idx_employes_nom ON employes;  -- MySQL
DROP INDEX idx_employes_nom;              -- PostgreSQL
```

### 7.3 Quand utiliser un index ?

```
┌────────────────────────────────────────────────────────┐
│  UTILISER UN INDEX                                     │
│                                                        │
│  ✓ Colonnes frequemment dans WHERE                     │
│  ✓ Colonnes utilisees dans JOIN (cles etrangeres)      │
│  ✓ Colonnes dans ORDER BY                              │
│  ✓ Colonnes avec une grande cardinalite                │
│    (beaucoup de valeurs distinctes)                     │
│  ✓ Tables volumineuses (> 10 000 lignes)               │
│                                                        │
├────────────────────────────────────────────────────────┤
│  NE PAS UTILISER UN INDEX                              │
│                                                        │
│  ✗ Petites tables (< 1000 lignes)                      │
│  ✗ Colonnes rarement utilisees dans les requetes       │
│  ✗ Colonnes avec peu de valeurs distinctes             │
│    (ex: booleen actif/inactif sur 1M lignes)           │
│  ✗ Tables avec beaucoup d'INSERT/UPDATE/DELETE         │
│    (chaque ecriture met a jour l'index)                │
│  ✗ Trop d'index sur une meme table (ralentit les      │
│    ecritures)                                          │
└────────────────────────────────────────────────────────┘
```

> [!warning] L'ordre des colonnes dans un index composite compte
> ```sql
> CREATE INDEX idx_dept_nom ON employes(departement_id, nom);
>
> -- Cet index est utilise pour :
> WHERE departement_id = 1                        -- OUI (1ere colonne)
> WHERE departement_id = 1 AND nom = 'Alice'      -- OUI (les deux)
>
> -- Cet index N'EST PAS utilise pour :
> WHERE nom = 'Alice'                             -- NON (2eme colonne seule)
> ```
> L'index composite suit le principe du **prefixe le plus a gauche** (leftmost prefix).

### 7.4 Verifier l'utilisation des index

```sql
-- MySQL : EXPLAIN montre le plan d'execution
EXPLAIN SELECT * FROM employes WHERE nom = 'Alice';

-- PostgreSQL : EXPLAIN ANALYZE (avec les temps reels)
EXPLAIN ANALYZE SELECT * FROM employes WHERE nom = 'Alice';
```

---

## 8. Contraintes : Recapitulatif

```sql
CREATE TABLE produits (
    -- Cle primaire
    id              INT PRIMARY KEY AUTO_INCREMENT,

    -- NOT NULL : obligatoire
    nom             VARCHAR(100) NOT NULL,

    -- UNIQUE : pas de doublons
    code_barre      VARCHAR(13) UNIQUE,

    -- CHECK : condition
    prix            DECIMAL(10,2) CHECK (prix > 0),
    stock           INT DEFAULT 0 CHECK (stock >= 0),

    -- DEFAULT : valeur par defaut
    actif           BOOLEAN DEFAULT TRUE,
    date_creation   DATETIME DEFAULT CURRENT_TIMESTAMP,

    -- FOREIGN KEY : reference vers une autre table
    categorie_id    INT,
    FOREIGN KEY (categorie_id) REFERENCES categories(id)
        ON DELETE SET NULL
        ON UPDATE CASCADE,

    -- Contrainte nommee sur plusieurs colonnes
    CONSTRAINT chk_nom_non_vide CHECK (LENGTH(nom) > 0)
);
```

---

## 9. Exemple Complet : Plateforme de Blog

### 9.1 Schema conceptuel

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   users      │     │    posts     │     │  commentaires│
│──────────────│     │──────────────│     │──────────────│
│ PK id        │──|──<│ PK id       │──|──<│ PK id       │
│    username   │     │ FK auteur_id │     │ FK post_id   │
│    email     │     │    titre     │     │ FK auteur_id │
│    password  │     │    contenu   │     │    contenu   │
│    bio       │     │    statut    │     │    date_crea │
│    date_inscr│     │    date_pub  │     └──────────────┘
└──────────────┘     │    date_modif│
        │            └──────┬───────┘
        │                   │
   ─|──<──              ──>──<──
        │                   │
┌───────▼──────┐     ┌──────▼───────┐     ┌──────────────┐
│  followers   │     │  post_tags   │     │    tags      │
│──────────────│     │──────────────│     │──────────────│
│ FK follower  │     │ FK post_id   │──>──|│ PK id       │
│ FK following │     │ FK tag_id    │     │    nom       │
│    date      │     │              │     │    slug      │
└──────────────┘     └──────────────┘     └──────────────┘
```

### 9.2 Implementation SQL

```sql
-- Base de donnees
CREATE DATABASE IF NOT EXISTS blog;
USE blog;

-- Table des utilisateurs
CREATE TABLE users (
    id              INT PRIMARY KEY AUTO_INCREMENT,
    username        VARCHAR(30) NOT NULL UNIQUE,
    email           VARCHAR(100) NOT NULL UNIQUE,
    password_hash   VARCHAR(255) NOT NULL,
    bio             TEXT,
    avatar_url      VARCHAR(255),
    date_inscription DATETIME DEFAULT CURRENT_TIMESTAMP,
    actif           BOOLEAN DEFAULT TRUE
);

-- Table des posts/articles
CREATE TABLE posts (
    id              INT PRIMARY KEY AUTO_INCREMENT,
    auteur_id       INT NOT NULL,
    titre           VARCHAR(200) NOT NULL,
    slug            VARCHAR(200) NOT NULL UNIQUE,
    contenu         TEXT NOT NULL,
    extrait         VARCHAR(500),
    statut          ENUM('brouillon', 'publie', 'archive') DEFAULT 'brouillon',
    date_publication DATETIME,
    date_modification DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    date_creation   DATETIME DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (auteur_id) REFERENCES users(id)
        ON DELETE RESTRICT,

    -- Un post publie doit avoir une date de publication
    CONSTRAINT chk_publication CHECK (
        statut != 'publie' OR date_publication IS NOT NULL
    ),

    INDEX idx_posts_auteur (auteur_id),
    INDEX idx_posts_statut_date (statut, date_publication DESC)
);

-- Table des commentaires
CREATE TABLE commentaires (
    id              INT PRIMARY KEY AUTO_INCREMENT,
    post_id         INT NOT NULL,
    auteur_id       INT NOT NULL,
    contenu         TEXT NOT NULL,
    date_creation   DATETIME DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (post_id) REFERENCES posts(id)
        ON DELETE CASCADE,       -- supprimer le post supprime ses commentaires
    FOREIGN KEY (auteur_id) REFERENCES users(id)
        ON DELETE CASCADE,

    INDEX idx_commentaires_post (post_id)
);

-- Table des tags
CREATE TABLE tags (
    id      INT PRIMARY KEY AUTO_INCREMENT,
    nom     VARCHAR(50) NOT NULL UNIQUE,
    slug    VARCHAR(50) NOT NULL UNIQUE
);

-- Table de jonction post-tags (N:M)
CREATE TABLE post_tags (
    post_id INT NOT NULL,
    tag_id  INT NOT NULL,
    PRIMARY KEY (post_id, tag_id),   -- Cle composee
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id)  REFERENCES tags(id)  ON DELETE CASCADE
);

-- Table de followers (relation N:M reflexive sur users)
CREATE TABLE followers (
    follower_id     INT NOT NULL,   -- celui qui suit
    following_id    INT NOT NULL,   -- celui qui est suivi
    date_follow     DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (follower_id, following_id),
    FOREIGN KEY (follower_id)  REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (following_id) REFERENCES users(id) ON DELETE CASCADE,
    -- Empecher de se suivre soi-meme
    CONSTRAINT chk_no_self_follow CHECK (follower_id != following_id)
);
```

### 9.3 Donnees d'exemple

```sql
INSERT INTO users (username, email, password_hash, bio) VALUES
    ('alice_dev',   'alice@blog.fr',   '$2b$12$hash1...', 'Developpeuse passionnee'),
    ('bob_write',   'bob@blog.fr',     '$2b$12$hash2...', 'Ecrivain technique'),
    ('charlie_data','charlie@blog.fr', '$2b$12$hash3...', 'Data analyst');

INSERT INTO tags (nom, slug) VALUES
    ('SQL',        'sql'),
    ('Python',     'python'),
    ('DevOps',     'devops'),
    ('Tutoriel',   'tutoriel');

INSERT INTO posts (auteur_id, titre, slug, contenu, statut, date_publication) VALUES
    (1, 'Debuter avec SQL', 'debuter-sql',
     'Le SQL est un langage essentiel...', 'publie', '2025-01-15 10:00:00'),
    (1, 'JOINs SQL Expliques', 'joins-sql',
     'Les jointures permettent de...', 'publie', '2025-02-01 14:30:00'),
    (2, 'Python pour Debutants', 'python-debutants',
     'Python est un langage...', 'brouillon', NULL);

INSERT INTO post_tags (post_id, tag_id) VALUES
    (1, 1), (1, 4),  -- SQL + Tutoriel
    (2, 1),           -- SQL
    (3, 2), (3, 4);   -- Python + Tutoriel

INSERT INTO commentaires (post_id, auteur_id, contenu) VALUES
    (1, 2, 'Super article, merci !'),
    (1, 3, 'Tres clair, bien explique.'),
    (2, 3, 'J''avais justement besoin de ca.');

INSERT INTO followers (follower_id, following_id) VALUES
    (2, 1),  -- Bob suit Alice
    (3, 1),  -- Charlie suit Alice
    (3, 2);  -- Charlie suit Bob
```

### 9.4 Requetes d'exemple

```sql
-- Articles publies avec leur auteur et nombre de commentaires
SELECT
    p.titre,
    u.username AS auteur,
    p.date_publication,
    COUNT(c.id) AS nb_commentaires
FROM posts p
JOIN users u ON p.auteur_id = u.id
LEFT JOIN commentaires c ON c.post_id = p.id
WHERE p.statut = 'publie'
GROUP BY p.id, p.titre, u.username, p.date_publication
ORDER BY p.date_publication DESC;

-- Tags d'un article
SELECT p.titre, GROUP_CONCAT(t.nom ORDER BY t.nom SEPARATOR ', ') AS tags
FROM posts p
JOIN post_tags pt ON pt.post_id = p.id
JOIN tags t ON t.id = pt.tag_id
GROUP BY p.id, p.titre;

-- Nombre de followers par utilisateur
SELECT
    u.username,
    COUNT(f.follower_id) AS nb_followers
FROM users u
LEFT JOIN followers f ON f.following_id = u.id
GROUP BY u.id, u.username
ORDER BY nb_followers DESC;
```

---

## Carte Mentale ASCII

```
                  ┌──────────────────────────────┐
                  │  CONCEPTION DE BASES DE       │
                  │       DONNEES                 │
                  └──────────────┬───────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
  ┌─────▼──────┐         ┌──────▼──────┐          ┌──────▼──────┐
  │ Modelisation│        │Normalisation│          │  Physique   │
  └─────┬──────┘         └──────┬──────┘          └──────┬──────┘
        │                       │                        │
  Entites              1NF : Atomicite              Index
  Relations            2NF : Pas de dep.            B-tree
  Diagramme ER              partielles              Composite
  Cardinalite          3NF : Pas de dep.            Quand indexer
  1:1, 1:N, N:M            transitives             EXPLAIN
        │                       │                        │
  ┌─────▼──────┐         ┌──────▼──────┐          ┌──────▼──────┐
  │   Cles     │         │ Contraintes │          │   Schema    │
  └─────┬──────┘         └──────┬──────┘          │  de Blog    │
        │                       │                  └─────────────┘
  PK : naturelle         NOT NULL
       surrogate         UNIQUE                   users
       composee          CHECK                    posts
  FK : REFERENCES        DEFAULT                  commentaires
       ON DELETE         FOREIGN KEY              tags
       CASCADE                                    post_tags
       SET NULL                                   followers
       RESTRICT
```

---

## Exercices

### Exercice 1 : Modelisation
Concevez le schema d'une base pour un **systeme de reservation de billets de cinema** :
- Cinemas (nom, adresse, ville)
- Salles (numero, capacite, type : standard/IMAX/VIP)
- Films (titre, duree, genre, classification)
- Seances (film, salle, date/heure, prix)
- Clients (nom, email, date de naissance)
- Reservations (client, seance, nombre de places, montant total)

Dessinez le diagramme ER en ASCII et ecrivez le SQL de creation des tables avec toutes les contraintes appropriees.

### Exercice 2 : Normalisation
La table suivante viole les formes normales. Identifiez les violations et normalisez-la en 3NF :
```
Table: ventes
+----+--------+--------+----------+---------+-------------+-----------+
| id | client | email  | produit  | prix    | categorie   | date      |
+----+--------+--------+----------+---------+-------------+-----------+
|  1 | Alice  | a@m.fr | Laptop   | 999.99  | Electronique| 2025-01-10|
|  2 | Alice  | a@m.fr | Souris   |  29.99  | Accessoire  | 2025-01-10|
|  3 | Bob    | b@m.fr | Laptop   | 999.99  | Electronique| 2025-01-12|
+----+--------+--------+----------+---------+-------------+-----------+
```

### Exercice 3 : Index
Sur la base du blog creee dans ce cours :
1. Quels index ajouteriez-vous pour optimiser la recherche de posts par tag ?
2. Expliquez pourquoi un index sur `post_tags(tag_id, post_id)` serait utile et dans quels cas
3. Un index sur `commentaires(contenu)` serait-il pertinent ? Pourquoi ?

---

## Liens

- [[02 - Requetes Avancees SQL]] - JOINs, sous-requetes, fonctions de fenetrage
- [[04 - SQL Avance et Administration]] - Transactions, vues, procedures, securite
