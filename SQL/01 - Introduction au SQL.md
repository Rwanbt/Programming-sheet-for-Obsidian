# Introduction au SQL

Le **SQL** (Structured Query Language) est le langage universel de communication avec les bases de donnees relationnelles. Depuis sa creation dans les annees 1970 par IBM, il est devenu le standard incontournable pour stocker, manipuler et interroger des donnees structurees. Que vous developpiez une application web, une application mobile ou un systeme d'entreprise, la maitrise du SQL est une competence fondamentale.

Ce cours couvre les bases essentielles : comprendre ce qu'est une base de donnees, installer un SGBD, creer des tables, inserer des donnees et ecrire vos premieres requetes.

> [!tip] Analogie
> Imaginez une base de donnees comme une **bibliotheque**. Les **tables** sont les rayons thematiques, les **colonnes** sont les categories d'information sur chaque livre (titre, auteur, annee), et les **lignes** sont les fiches individuelles de chaque livre. Le SQL est le **langage** que vous utilisez pour demander au bibliothecaire de trouver, ajouter ou modifier des fiches.

---

## 1. Qu'est-ce qu'une Base de Donnees ?

Une **base de donnees** (database) est un ensemble organise de donnees stockees electroniquement. Une base de donnees **relationnelle** organise les donnees en **tables** (relations) composees de **lignes** (enregistrements/rows) et de **colonnes** (champs/columns).

> [!info] Vocabulaire cle
> - **Table** : structure qui contient des donnees organisees en lignes et colonnes
> - **Ligne (row)** : un enregistrement unique dans une table
> - **Colonne (column)** : un attribut/champ decrivant une propriete
> - **Schema** : la structure globale de la base (tables, colonnes, relations)
> - **SGBD** : Systeme de Gestion de Base de Donnees (le logiciel qui gere la base)

### Representation visuelle d'une table

```
Table: etudiants
+----+-----------+-----------+-----+-------------------+
| id | prenom    | nom       | age | email             |
+----+-----------+-----------+-----+-------------------+
|  1 | Alice     | Dupont    |  22 | alice@mail.com    |
|  2 | Bob       | Martin    |  25 | bob@mail.com      |
|  3 | Charlie   | Bernard   |  20 | charlie@mail.com  |
+----+-----------+-----------+-----+-------------------+
  ^       ^          ^         ^          ^
  |       |          |         |          |
 colonne colonne   colonne  colonne    colonne
```

---

## 2. Les SGBD : Comparaison

Un **SGBD** (Systeme de Gestion de Base de Donnees) est le logiciel qui permet de creer, gerer et interroger les bases de donnees.

### Tableau comparatif des principaux SGBD relationnels

```
+---------------+------------+----------+-----------+------------------+------------------+
| SGBD          | Licence    | Poids    | Serveur   | Cas d'usage      | Popularite       |
+---------------+------------+----------+-----------+------------------+------------------+
| MySQL         | Open/Comm. | Moyen    | Oui       | Web, startups    | Tres populaire   |
| PostgreSQL    | Open Source | Moyen    | Oui       | Entreprise, GIS  | En forte hausse  |
| SQLite        | Domaine    | Tres     | Non       | Mobile, embarque | Tres repandu     |
|               | public     | leger    | (fichier) | prototypage      |                  |
| MariaDB       | Open Source | Moyen    | Oui       | Fork de MySQL    | Populaire        |
| SQL Server    | Commercial | Lourd    | Oui       | Entreprise MS    | Populaire        |
| Oracle DB     | Commercial | Lourd    | Oui       | Grandes entpr.   | Standard corpo.  |
+---------------+------------+----------+-----------+------------------+------------------+
```

> [!tip] Conseil pour debuter
> - **SQLite** : ideal pour apprendre, aucune installation serveur requise
> - **PostgreSQL** : meilleur choix pour un usage professionnel, tres respectueux du standard SQL
> - **MySQL** : tres repandu dans le web (WordPress, PHP)

---

## 3. Categories du langage SQL

Le SQL se divise en plusieurs sous-langages selon le type d'operation :

```
+------+-----------------------------------+----------------------------+
| Cat. | Nom complet                       | Commandes principales      |
+------+-----------------------------------+----------------------------+
| DDL  | Data Definition Language          | CREATE, ALTER, DROP,       |
|      | (definition de structure)         | TRUNCATE, RENAME           |
+------+-----------------------------------+----------------------------+
| DML  | Data Manipulation Language        | INSERT, UPDATE, DELETE,    |
|      | (manipulation de donnees)         | MERGE                      |
+------+-----------------------------------+----------------------------+
| DQL  | Data Query Language               | SELECT                     |
|      | (interrogation de donnees)        |                            |
+------+-----------------------------------+----------------------------+
| DCL  | Data Control Language             | GRANT, REVOKE              |
|      | (controle des droits)             |                            |
+------+-----------------------------------+----------------------------+
| TCL  | Transaction Control Language      | BEGIN, COMMIT, ROLLBACK,   |
|      | (gestion des transactions)        | SAVEPOINT                  |
+------+-----------------------------------+----------------------------+
```

> [!info] DDL vs DML
> **DDL** modifie la **structure** (le contenant) : creer une table, ajouter une colonne.
> **DML** modifie les **donnees** (le contenu) : ajouter une ligne, modifier une valeur.
> Cette distinction est fondamentale pour comprendre le SQL.

---

## 4. Installation et Connexion

### 4.1 Installer MySQL

```bash
# Sur Ubuntu/Debian
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation

# Sur macOS avec Homebrew
brew install mysql
brew services start mysql

# Sur Windows : telecharger l'installeur depuis mysql.com
# ou utiliser Chocolatey :
choco install mysql
```

### 4.2 Installer PostgreSQL

```bash
# Sur Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql

# Sur macOS avec Homebrew
brew install postgresql@16
brew services start postgresql@16

# Sur Windows : telecharger depuis postgresql.org
# ou utiliser Chocolatey :
choco install postgresql
```

### 4.3 Se connecter en ligne de commande

```bash
# MySQL
mysql -u root -p
# Saisir le mot de passe

# PostgreSQL
sudo -u postgres psql
# ou directement si configure
psql -U mon_utilisateur -d ma_base
```

### 4.4 Outils graphiques (GUI)

```
+-------------+------------------+----------------------------------+
| Outil       | SGBD supportes   | Points forts                     |
+-------------+------------------+----------------------------------+
| DBeaver     | Tous (universel) | Gratuit, open-source, complet    |
| pgAdmin     | PostgreSQL       | Outil officiel PostgreSQL        |
| MySQL       | MySQL            | Outil officiel MySQL             |
| Workbench   |                  |                                  |
| DataGrip    | Tous             | Payant (JetBrains), tres puissant|
| HeidiSQL    | MySQL, MariaDB,  | Leger, Windows                   |
|             | PostgreSQL       |                                  |
+-------------+------------------+----------------------------------+
```

> [!tip] Recommandation
> **DBeaver Community Edition** est le choix le plus polyvalent pour debuter. Il supporte tous les SGBD majeurs, propose l'autocompletion SQL et permet de visualiser les schemas graphiquement.

---

## 5. Creer une Base de Donnees

### 5.1 CREATE DATABASE

```sql
-- Creer une base de donnees
CREATE DATABASE universite;

-- Avec des options d'encodage (MySQL)
CREATE DATABASE universite
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- PostgreSQL avec encodage
CREATE DATABASE universite
  ENCODING 'UTF8'
  LC_COLLATE 'fr_FR.UTF-8';

-- Creer seulement si elle n'existe pas
CREATE DATABASE IF NOT EXISTS universite;
```

### 5.2 USE (selectionner une base)

```sql
-- MySQL : selectionner la base active
USE universite;

-- PostgreSQL : on se connecte directement
-- \c universite   (commande psql)
```

### 5.3 Supprimer une base

```sql
-- Supprimer une base de donnees
DROP DATABASE universite;

-- Supprimer seulement si elle existe
DROP DATABASE IF EXISTS universite;
```

> [!warning] Attention
> `DROP DATABASE` supprime **definitivement** toute la base et ses donnees.
> Il n'y a **aucun moyen de recuperation** sans sauvegarde prealable.
> Toujours verifier deux fois avant d'executer cette commande.

---

## 6. Types de Donnees

### Tableau des types de donnees courants

```
+-------------+---------------------------+----------------------------+--------------------+
| Type        | Description               | Exemple                    | Taille memoire     |
+-------------+---------------------------+----------------------------+--------------------+
| INT         | Entier signe              | 42, -100, 0               | 4 octets           |
| BIGINT      | Grand entier              | 9223372036854775807        | 8 octets           |
| SMALLINT    | Petit entier              | -32768 a 32767             | 2 octets           |
| FLOAT       | Decimal approx.           | 3.14 (precision ~7 chif.) | 4 octets           |
| DOUBLE      | Decimal approx.           | 3.14159265 (precision ~15)| 8 octets           |
| DECIMAL(p,s)| Decimal exact             | DECIMAL(10,2) -> 99999.99 | Variable           |
| VARCHAR(n)  | Chaine variable           | VARCHAR(100) -> max 100 c.| n + 1-2 octets     |
| CHAR(n)     | Chaine fixe               | CHAR(5) -> toujours 5 c.  | n octets           |
| TEXT        | Texte long                | Articles, descriptions     | Variable (max ~1Go)|
| DATE        | Date                      | '2025-01-15'               | 3 octets           |
| DATETIME    | Date et heure             | '2025-01-15 14:30:00'     | 8 octets           |
| TIMESTAMP   | Date/heure UTC            | '2025-01-15 14:30:00'     | 4 octets           |
| BOOLEAN     | Vrai/Faux                 | TRUE, FALSE                | 1 octet            |
| BLOB        | Donnees binaires          | Images, fichiers           | Variable           |
+-------------+---------------------------+----------------------------+--------------------+
```

> [!info] FLOAT vs DECIMAL
> - **FLOAT/DOUBLE** : stockage approche, rapide mais imprecis pour les calculs financiers
>   ```sql
>   -- FLOAT peut donner : 0.1 + 0.2 = 0.30000000000000004
>   ```
> - **DECIMAL(p,s)** : stockage exact, ideal pour les montants d'argent
>   ```sql
>   -- DECIMAL(10,2) : 10 chiffres au total, 2 apres la virgule
>   -- Parfait pour les prix : 12345678.99
>   ```

> [!warning] VARCHAR vs TEXT
> - **VARCHAR(255)** : limite explicite, peut etre indexe directement
> - **TEXT** : pas de limite pratique, mais indexation plus complexe
> Privilegiez VARCHAR quand vous connaissez la taille maximale.

---

## 7. CREATE TABLE : Creer des Tables

### 7.1 Syntaxe de base

```sql
CREATE TABLE etudiants (
    id          INT PRIMARY KEY AUTO_INCREMENT,  -- Cle primaire auto-incrementee
    prenom      VARCHAR(50) NOT NULL,            -- Obligatoire
    nom         VARCHAR(50) NOT NULL,
    email       VARCHAR(100) UNIQUE,             -- Valeur unique
    age         INT CHECK (age >= 16 AND age <= 99),  -- Contrainte de verification
    date_naissance DATE,
    actif       BOOLEAN DEFAULT TRUE,            -- Valeur par defaut
    date_inscription DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

> [!info] PostgreSQL vs MySQL
> PostgreSQL utilise `SERIAL` ou `GENERATED ALWAYS AS IDENTITY` au lieu de `AUTO_INCREMENT` :
> ```sql
> -- PostgreSQL
> CREATE TABLE etudiants (
>     id    SERIAL PRIMARY KEY,
>     -- ou (standard SQL moderne)
>     id    INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
>     prenom VARCHAR(50) NOT NULL
> );
> ```

### 7.2 Les contraintes (constraints)

```sql
CREATE TABLE cours (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    code        VARCHAR(10) NOT NULL UNIQUE,    -- Combine NOT NULL + UNIQUE
    nom         VARCHAR(100) NOT NULL,
    credits     INT NOT NULL DEFAULT 3,
    departement VARCHAR(50),
    capacite    INT CHECK (capacite > 0 AND capacite <= 500),
    date_debut  DATE NOT NULL,
    date_fin    DATE,

    -- Contrainte CHECK sur plusieurs colonnes
    CONSTRAINT chk_dates CHECK (date_fin IS NULL OR date_fin > date_debut)
);
```

### Recapitulatif des contraintes

```
+----------------+------------------------------------------------+
| Contrainte     | Description                                    |
+----------------+------------------------------------------------+
| PRIMARY KEY    | Identifiant unique de la ligne (NOT NULL +     |
|                | UNIQUE implicites). Une seule par table.       |
+----------------+------------------------------------------------+
| NOT NULL       | La colonne ne peut pas contenir NULL           |
+----------------+------------------------------------------------+
| UNIQUE         | Aucune valeur dupliquee dans cette colonne     |
+----------------+------------------------------------------------+
| DEFAULT        | Valeur par defaut si aucune n'est fournie      |
+----------------+------------------------------------------------+
| CHECK          | Verifie une condition sur la valeur            |
+----------------+------------------------------------------------+
| AUTO_INCREMENT | Genere automatiquement un entier croissant     |
| (SERIAL)       | (MySQL: AUTO_INCREMENT, PG: SERIAL)            |
+----------------+------------------------------------------------+
| FOREIGN KEY    | Reference vers une cle primaire d'une autre    |
|                | table (voir cours 03)                          |
+----------------+------------------------------------------------+
```

### 7.3 Table d'inscription (relation entre tables)

```sql
CREATE TABLE inscriptions (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    etudiant_id INT NOT NULL,
    cours_id    INT NOT NULL,
    note        DECIMAL(4,2) CHECK (note >= 0 AND note <= 20),
    date_inscription DATE DEFAULT (CURRENT_DATE),

    -- Cles etrangeres (expliquees en detail dans le cours 03)
    FOREIGN KEY (etudiant_id) REFERENCES etudiants(id),
    FOREIGN KEY (cours_id)    REFERENCES cours(id),

    -- Empecher un etudiant de s'inscrire deux fois au meme cours
    UNIQUE (etudiant_id, cours_id)
);
```

---

## 8. INSERT INTO : Inserer des Donnees

### 8.1 Insertion d'une seule ligne

```sql
-- Insertion complete (toutes les colonnes sauf AUTO_INCREMENT)
INSERT INTO etudiants (prenom, nom, email, age, date_naissance, actif)
VALUES ('Alice', 'Dupont', 'alice@universite.fr', 22, '2003-03-15', TRUE);

-- Insertion partielle (les colonnes non specifiees prennent la valeur DEFAULT ou NULL)
INSERT INTO etudiants (prenom, nom, email, age)
VALUES ('Bob', 'Martin', 'bob@universite.fr', 25);
```

### 8.2 Insertion de plusieurs lignes

```sql
-- Inserer plusieurs etudiants d'un coup (plus performant)
INSERT INTO etudiants (prenom, nom, email, age, date_naissance) VALUES
    ('Charlie', 'Bernard',  'charlie@universite.fr', 20, '2005-07-22'),
    ('Diana',   'Petit',    'diana@universite.fr',   23, '2002-11-08'),
    ('Eve',     'Robert',   'eve@universite.fr',     21, '2004-05-30'),
    ('Frank',   'Moreau',   'frank@universite.fr',   24, '2001-09-12'),
    ('Grace',   'Laurent',  'grace@universite.fr',   22, '2003-02-28');
```

### 8.3 Inserer des cours

```sql
INSERT INTO cours (code, nom, credits, departement, capacite, date_debut, date_fin) VALUES
    ('INF101', 'Introduction a la Programmation', 6, 'Informatique', 200, '2025-09-01', '2026-01-15'),
    ('INF201', 'Bases de Donnees',                5, 'Informatique', 150, '2025-09-01', '2026-01-15'),
    ('MAT101', 'Algebre Lineaire',                4, 'Mathematiques', 180, '2025-09-01', '2026-01-15'),
    ('MAT201', 'Analyse Numerique',               4, 'Mathematiques', 120, '2025-09-01', '2026-01-15'),
    ('PHY101', 'Mecanique Classique',             5, 'Physique',      100, '2025-09-01', '2026-01-15');

-- Inserer des inscriptions
INSERT INTO inscriptions (etudiant_id, cours_id, note) VALUES
    (1, 1, 16.50),
    (1, 2, 18.00),
    (2, 1, 12.75),
    (2, 3, 14.00),
    (3, 2, 15.25),
    (3, 4, NULL),    -- Note pas encore attribuee
    (4, 1, 11.00),
    (4, 5, 17.50);
```

> [!warning] Erreurs courantes avec INSERT
> ```sql
> -- ERREUR : violation de NOT NULL
> INSERT INTO etudiants (prenom) VALUES ('Alice');
> -- nom est NOT NULL, il manque !
>
> -- ERREUR : violation de UNIQUE
> INSERT INTO etudiants (prenom, nom, email)
> VALUES ('Alice', 'Autre', 'alice@universite.fr');
> -- cet email existe deja !
>
> -- ERREUR : violation de CHECK
> INSERT INTO etudiants (prenom, nom, age)
> VALUES ('Test', 'Test', 10);
> -- age doit etre >= 16
> ```

---

## 9. SELECT : Interroger les Donnees

### 9.1 Selection de base

```sql
-- Selectionner toutes les colonnes et toutes les lignes
SELECT * FROM etudiants;

-- Selectionner des colonnes specifiques
SELECT prenom, nom, age FROM etudiants;

-- Utiliser des alias (AS) pour renommer les colonnes dans le resultat
SELECT
    prenom AS "Prenom",
    nom    AS "Nom de famille",
    age    AS "Age (annees)"
FROM etudiants;

-- Expressions calculees
SELECT
    prenom,
    nom,
    2025 - YEAR(date_naissance) AS age_calcule
FROM etudiants;
```

> [!example] Resultat de `SELECT prenom, nom, age FROM etudiants`
> ```
> +---------+---------+-----+
> | prenom  | nom     | age |
> +---------+---------+-----+
> | Alice   | Dupont  |  22 |
> | Bob     | Martin  |  25 |
> | Charlie | Bernard |  20 |
> | Diana   | Petit   |  23 |
> | Eve     | Robert  |  21 |
> +---------+---------+-----+
> ```

### 9.2 SELECT DISTINCT

```sql
-- Eliminer les doublons
SELECT DISTINCT departement FROM cours;

-- Resultat : 'Informatique', 'Mathematiques', 'Physique'
```

---

## 10. WHERE : Filtrer les Resultats

### 10.1 Operateurs de comparaison

```sql
-- Egalite
SELECT * FROM etudiants WHERE age = 22;

-- Inegalite
SELECT * FROM etudiants WHERE age != 22;
-- ou
SELECT * FROM etudiants WHERE age <> 22;

-- Superieur, inferieur
SELECT * FROM etudiants WHERE age > 21;
SELECT * FROM etudiants WHERE age >= 22;
SELECT * FROM etudiants WHERE age < 23;
SELECT * FROM etudiants WHERE age <= 22;
```

### 10.2 BETWEEN, IN, LIKE

```sql
-- BETWEEN : intervalle inclusif
SELECT * FROM etudiants
WHERE age BETWEEN 20 AND 23;
-- Equivalent a : age >= 20 AND age <= 23

-- IN : liste de valeurs
SELECT * FROM cours
WHERE departement IN ('Informatique', 'Physique');

-- NOT IN : exclure des valeurs
SELECT * FROM cours
WHERE departement NOT IN ('Mathematiques');

-- LIKE : recherche par motif
SELECT * FROM etudiants WHERE nom LIKE 'M%';     -- commence par M
SELECT * FROM etudiants WHERE nom LIKE '%t';      -- finit par t
SELECT * FROM etudiants WHERE nom LIKE '%ar%';    -- contient "ar"
SELECT * FROM etudiants WHERE prenom LIKE '___';  -- exactement 3 caracteres
-- % = n'importe quelle sequence de caracteres (0 ou plus)
-- _ = exactement un caractere
```

### 10.3 IS NULL / IS NOT NULL

```sql
-- Trouver les inscriptions sans note
SELECT * FROM inscriptions WHERE note IS NULL;

-- Trouver les inscriptions avec une note
SELECT * FROM inscriptions WHERE note IS NOT NULL;
```

> [!warning] Piege avec NULL
> ```sql
> -- INCORRECT : ceci ne fonctionne PAS !
> SELECT * FROM inscriptions WHERE note = NULL;
> -- NULL n'est pas une valeur, c'est l'absence de valeur.
> -- On ne peut pas comparer avec = ou !=
>
> -- CORRECT :
> SELECT * FROM inscriptions WHERE note IS NULL;
> ```

### 10.4 Combiner les conditions (AND, OR, NOT)

```sql
-- AND : les deux conditions doivent etre vraies
SELECT * FROM etudiants
WHERE age >= 21 AND actif = TRUE;

-- OR : au moins une condition doit etre vraie
SELECT * FROM cours
WHERE departement = 'Informatique' OR credits >= 5;

-- NOT : inverse la condition
SELECT * FROM etudiants
WHERE NOT (age > 23);

-- Combinaison complexe (utiliser des parentheses !)
SELECT * FROM etudiants
WHERE (age >= 20 AND age <= 23)
   OR (nom LIKE 'M%' AND actif = TRUE);
```

> [!tip] Priorite des operateurs
> L'ordre de priorite est : `NOT` > `AND` > `OR`.
> Utilisez **toujours des parentheses** pour clarifier vos intentions :
> ```sql
> -- Ambigu :
> WHERE a = 1 OR b = 2 AND c = 3
> -- Equivalent a : WHERE a = 1 OR (b = 2 AND c = 3)
>
> -- Clair :
> WHERE (a = 1 OR b = 2) AND c = 3
> ```

---

## 11. ORDER BY : Trier les Resultats

```sql
-- Tri croissant (par defaut)
SELECT * FROM etudiants ORDER BY nom ASC;

-- Tri decroissant
SELECT * FROM etudiants ORDER BY age DESC;

-- Tri sur plusieurs colonnes
SELECT * FROM etudiants
ORDER BY nom ASC, prenom ASC;

-- Tri par numero de colonne (deconseille mais possible)
SELECT prenom, nom, age FROM etudiants
ORDER BY 3 DESC;  -- trie par la 3e colonne (age)

-- Combiner avec WHERE
SELECT * FROM etudiants
WHERE actif = TRUE
ORDER BY date_inscription DESC;
```

---

## 12. LIMIT et OFFSET : Pagination

```sql
-- Les 3 premiers etudiants
SELECT * FROM etudiants
ORDER BY nom ASC
LIMIT 3;

-- Sauter les 2 premiers, prendre les 3 suivants (pagination)
SELECT * FROM etudiants
ORDER BY nom ASC
LIMIT 3 OFFSET 2;
-- Affiche les etudiants 3, 4, 5

-- PostgreSQL supporte aussi la syntaxe standard SQL :
-- FETCH FIRST 3 ROWS ONLY
-- OFFSET 2 ROWS FETCH NEXT 3 ROWS ONLY
```

> [!example] Pagination pour une application web
> ```sql
> -- Page 1 (elements 1-10)
> SELECT * FROM etudiants ORDER BY id LIMIT 10 OFFSET 0;
>
> -- Page 2 (elements 11-20)
> SELECT * FROM etudiants ORDER BY id LIMIT 10 OFFSET 10;
>
> -- Page N (formule generale)
> -- LIMIT taille_page OFFSET (numero_page - 1) * taille_page
> ```

---

## 13. UPDATE : Modifier des Donnees

```sql
-- Modifier une seule colonne pour un etudiant specifique
UPDATE etudiants
SET age = 23
WHERE id = 1;

-- Modifier plusieurs colonnes
UPDATE etudiants
SET email = 'nouvelle.alice@universite.fr',
    actif = FALSE
WHERE id = 1;

-- Modifier avec une expression calculee
UPDATE inscriptions
SET note = note + 1
WHERE note < 10;
-- Ajouter 1 point a toutes les notes en dessous de 10

-- Modifier plusieurs lignes selon une condition
UPDATE etudiants
SET actif = FALSE
WHERE date_inscription < '2024-01-01';
```

> [!warning] TOUJOURS utiliser WHERE avec UPDATE
> ```sql
> -- DANGER : sans WHERE, TOUTES les lignes sont modifiees !
> UPDATE etudiants SET actif = FALSE;
> -- Ceci desactive TOUS les etudiants !
>
> -- Bonne pratique : tester d'abord avec SELECT
> SELECT * FROM etudiants WHERE date_inscription < '2024-01-01';
> -- Verifier les lignes concernees, puis :
> UPDATE etudiants SET actif = FALSE
> WHERE date_inscription < '2024-01-01';
> ```

---

## 14. DELETE : Supprimer des Donnees

```sql
-- Supprimer un etudiant specifique
DELETE FROM etudiants WHERE id = 7;

-- Supprimer selon une condition
DELETE FROM inscriptions
WHERE note IS NULL AND date_inscription < '2024-06-01';

-- Supprimer toutes les lignes (garder la structure de la table)
DELETE FROM inscriptions;
-- ou plus rapide (reinitialise aussi l'auto-increment) :
TRUNCATE TABLE inscriptions;
```

> [!warning] TOUJOURS utiliser WHERE avec DELETE
> Meme principe que UPDATE : sans WHERE, toutes les lignes sont supprimees.
> ```sql
> -- Bonne pratique : d'abord SELECT pour verifier
> SELECT * FROM etudiants WHERE actif = FALSE;
> -- Puis supprimer
> DELETE FROM etudiants WHERE actif = FALSE;
> ```

---

## 15. DROP TABLE et ALTER TABLE

### 15.1 Supprimer une table

```sql
-- Supprimer une table et toutes ses donnees
DROP TABLE inscriptions;

-- Supprimer seulement si elle existe
DROP TABLE IF EXISTS inscriptions;
```

### 15.2 Modifier la structure d'une table

```sql
-- Ajouter une colonne
ALTER TABLE etudiants
ADD COLUMN telephone VARCHAR(20);

-- Ajouter une colonne avec une contrainte
ALTER TABLE etudiants
ADD COLUMN ville VARCHAR(50) DEFAULT 'Paris';

-- Supprimer une colonne
ALTER TABLE etudiants
DROP COLUMN telephone;

-- Modifier le type d'une colonne (MySQL)
ALTER TABLE etudiants
MODIFY COLUMN email VARCHAR(200) NOT NULL;

-- Modifier le type d'une colonne (PostgreSQL)
ALTER TABLE etudiants
ALTER COLUMN email TYPE VARCHAR(200);

-- Renommer une colonne (MySQL 8+)
ALTER TABLE etudiants
RENAME COLUMN nom TO nom_famille;

-- Ajouter une contrainte
ALTER TABLE etudiants
ADD CONSTRAINT chk_age CHECK (age >= 16);

-- Supprimer une contrainte
ALTER TABLE etudiants
DROP CONSTRAINT chk_age;

-- Renommer la table
ALTER TABLE etudiants
RENAME TO eleves;
```

> [!info] ALTER TABLE en production
> Modifier la structure d'une table en production peut etre risque :
> - Ajouter une colonne avec DEFAULT peut verrouiller la table pendant longtemps sur de grandes tables
> - Supprimer une colonne perd les donnees de maniere irreversible
> - Modifier un type peut echouer si les donnees existantes ne sont pas compatibles
> Testez toujours sur une copie avant d'appliquer en production.

---

## 16. Ordre d'Execution d'une Requete SELECT

Il est crucial de comprendre que SQL n'execute **pas** les clauses dans l'ordre ou vous les ecrivez :

```
Ordre d'ECRITURE :          Ordre d'EXECUTION :
                            
1. SELECT                   1. FROM        (quelle table ?)
2. FROM                     2. WHERE       (filtrer les lignes)
3. WHERE                    3. GROUP BY    (regrouper)
4. GROUP BY                 4. HAVING      (filtrer les groupes)
5. HAVING                   5. SELECT      (choisir les colonnes)
6. ORDER BY                 6. ORDER BY    (trier)
7. LIMIT                    7. LIMIT       (limiter le nombre)
```

> [!tip] Pourquoi c'est important
> Cela explique pourquoi on ne peut pas utiliser un alias defini dans SELECT
> a l'interieur de WHERE :
> ```sql
> -- ERREUR : l'alias n'existe pas encore quand WHERE est execute
> SELECT prenom, age * 2 AS double_age
> FROM etudiants
> WHERE double_age > 40;
>
> -- CORRECT : repeter l'expression
> SELECT prenom, age * 2 AS double_age
> FROM etudiants
> WHERE age * 2 > 40;
> ```

---

## 17. Exemple Complet : Mini-Projet

Voici un scenario complet pour pratiquer toutes les notions :

```sql
-- 1. Creer la base
CREATE DATABASE IF NOT EXISTS universite;
USE universite;

-- 2. Creer les tables
CREATE TABLE etudiants (
    id              INT PRIMARY KEY AUTO_INCREMENT,
    prenom          VARCHAR(50) NOT NULL,
    nom             VARCHAR(50) NOT NULL,
    email           VARCHAR(100) UNIQUE NOT NULL,
    age             INT CHECK (age >= 16 AND age <= 99),
    date_naissance  DATE,
    ville           VARCHAR(50) DEFAULT 'Paris',
    actif           BOOLEAN DEFAULT TRUE,
    date_inscription DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE cours (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    code        VARCHAR(10) UNIQUE NOT NULL,
    nom         VARCHAR(100) NOT NULL,
    credits     INT NOT NULL DEFAULT 3,
    departement VARCHAR(50) NOT NULL,
    capacite    INT CHECK (capacite > 0)
);

CREATE TABLE inscriptions (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    etudiant_id INT NOT NULL,
    cours_id    INT NOT NULL,
    note        DECIMAL(4,2) CHECK (note >= 0 AND note <= 20),
    FOREIGN KEY (etudiant_id) REFERENCES etudiants(id),
    FOREIGN KEY (cours_id) REFERENCES cours(id),
    UNIQUE (etudiant_id, cours_id)
);

-- 3. Inserer des donnees
INSERT INTO etudiants (prenom, nom, email, age, date_naissance, ville) VALUES
    ('Alice',   'Dupont',   'alice@univ.fr',   22, '2003-03-15', 'Paris'),
    ('Bob',     'Martin',   'bob@univ.fr',     25, '2000-07-22', 'Lyon'),
    ('Charlie', 'Bernard',  'charlie@univ.fr', 20, '2005-11-08', 'Paris'),
    ('Diana',   'Petit',    'diana@univ.fr',   23, '2002-05-30', 'Marseille'),
    ('Eve',     'Robert',   'eve@univ.fr',     21, '2004-09-12', 'Lyon');

INSERT INTO cours (code, nom, credits, departement, capacite) VALUES
    ('INF101', 'Intro Programmation',  6, 'Informatique',  200),
    ('INF201', 'Bases de Donnees',     5, 'Informatique',  150),
    ('MAT101', 'Algebre Lineaire',     4, 'Mathematiques', 180),
    ('PHY101', 'Mecanique',            5, 'Physique',      100);

INSERT INTO inscriptions (etudiant_id, cours_id, note) VALUES
    (1, 1, 16.50), (1, 2, 18.00),
    (2, 1, 12.75), (2, 3, 14.00),
    (3, 2, 15.25), (3, 1, 13.00),
    (4, 4, 17.50), (5, 2, NULL);

-- 4. Requetes d'interrogation

-- Tous les etudiants de Paris ages de plus de 20 ans
SELECT prenom, nom, age, ville
FROM etudiants
WHERE ville = 'Paris' AND age > 20
ORDER BY nom ASC;

-- Les cours d'informatique avec plus de 4 credits
SELECT code, nom, credits
FROM cours
WHERE departement = 'Informatique' AND credits > 4;

-- Les 3 meilleures notes
SELECT e.prenom, e.nom, c.nom AS cours, i.note
FROM inscriptions i
JOIN etudiants e ON i.etudiant_id = e.id
JOIN cours c ON i.cours_id = c.id
WHERE i.note IS NOT NULL
ORDER BY i.note DESC
LIMIT 3;

-- 5. Mise a jour
UPDATE etudiants
SET ville = 'Bordeaux'
WHERE id = 3;

-- 6. Suppression
DELETE FROM inscriptions
WHERE etudiant_id = 5 AND cours_id = 2;
```

---

## Carte Mentale ASCII

```
                        ┌─────────────────────────┐
                        │   INTRODUCTION AU SQL    │
                        └────────────┬────────────┘
                                     │
            ┌────────────────────────┼────────────────────────┐
            │                        │                        │
    ┌───────▼───────┐      ┌────────▼────────┐     ┌────────▼────────┐
    │  Concepts de  │      │   Structure     │     │   Requetes      │
    │    base       │      │   (DDL)         │     │   (DML/DQL)     │
    └───────┬───────┘      └────────┬────────┘     └────────┬────────┘
            │                       │                       │
    ┌───────┼───────┐       ┌───────┼───────┐       ┌──────┼───────┐
    │       │       │       │       │       │       │      │       │
  SGBD   Types   Categ.  CREATE  ALTER   DROP   INSERT  SELECT  UPDATE
  MySQL  SQL     DDL     TABLE   TABLE   TABLE  INTO     │     DELETE
  PG     DML             │                       │       │
  SQLite DQL          Colonnes                 Lignes  WHERE
         DCL          Contraintes              Multi   ORDER BY
                      PK, NOT NULL                     LIMIT
                      UNIQUE, CHECK                    AS (alias)
                      DEFAULT                          LIKE, IN
                      AUTO_INCREMENT                   BETWEEN
                                                       IS NULL
```

---

## Exercices

### Exercice 1 : Creer une base pour une librairie
Creez une base de donnees `librairie` avec une table `livres` contenant : id (cle primaire auto-incrementee), titre (obligatoire, max 200 caracteres), auteur (obligatoire, max 100 caracteres), prix (decimal exact avec 2 decimales, positif), pages (entier positif), genre (max 50 caracteres), disponible (booleen, par defaut vrai), date_publication (date). Inserez au moins 5 livres.

### Exercice 2 : Requetes de selection
Sur la base `universite` creee dans ce cours :
1. Affichez tous les etudiants dont le prenom commence par une voyelle
2. Trouvez les cours ayant entre 4 et 6 credits (inclus)
3. Listez les inscriptions sans note, triees par etudiant_id
4. Affichez les 3 etudiants les plus jeunes avec leur ville

### Exercice 3 : Modification et suppression
1. Changez la ville de tous les etudiants de Lyon en "Lyon Cedex"
2. Augmentez de 1 credit tous les cours du departement Informatique
3. Ajoutez une colonne `moyenne_generale` (DECIMAL(4,2)) a la table etudiants
4. Supprimez les inscriptions dont la note est inferieure a 10 (s'il y en a)

### Exercice 4 : Debug SQL
Trouvez et corrigez les erreurs dans chaque requete :
```sql
-- a)
SELECT prenom nom FROM etudiants WHERE age = NULL;

-- b)
INSERT INTO etudiants (prenom, email) VALUES ('Test', 'test@mail.com');

-- c)
SELECT * FROM etudiants ORDER BY age WHERE age > 20;
```

---

## Liens

- [[02 - Requetes Avancees SQL]] - Fonctions d'agregation, JOINs, sous-requetes
- [[03 - Conception de Bases de Donnees]] - Normalisation, cles etrangeres, indexes
- [[04 - SQL Avance et Administration]] - Transactions, vues, procedures stockees
