# Requetes Avancees SQL

Apres avoir maitrise les bases du SQL (SELECT, WHERE, INSERT, UPDATE, DELETE), il est temps de passer aux requetes avancees. Ce cours couvre les outils qui transforment le SQL d'un simple langage d'interrogation en un veritable moteur d'analyse de donnees : fonctions d'agregation, jointures, sous-requetes, expressions conditionnelles et fonctions de fenetrage.

Ces techniques sont essentielles pour tout developpeur ou analyste de donnees. Elles permettent de repondre a des questions complexes en une seule requete, sans avoir a traiter les donnees cote application.

> [!tip] Analogie
> Si les requetes de base (cours 01) sont comme **lire des fiches individuelles** dans un classeur, les requetes avancees sont comme **produire des rapports statistiques** : combien de fiches par categorie, quelle est la moyenne, quels classeurs ont des fiches en commun, etc. Vous passez de la consultation a l'analyse.

---

## Base de Donnees de Travail

Pour ce cours, nous utilisons une base `entreprise` avec les tables suivantes :

```sql
CREATE TABLE departements (
    id   INT PRIMARY KEY AUTO_INCREMENT,
    nom  VARCHAR(50) NOT NULL UNIQUE,
    ville VARCHAR(50) NOT NULL
);

CREATE TABLE employes (
    id              INT PRIMARY KEY AUTO_INCREMENT,
    prenom          VARCHAR(50) NOT NULL,
    nom             VARCHAR(50) NOT NULL,
    email           VARCHAR(100) UNIQUE,
    date_embauche   DATE NOT NULL,
    departement_id  INT,
    manager_id      INT,
    FOREIGN KEY (departement_id) REFERENCES departements(id),
    FOREIGN KEY (manager_id)     REFERENCES employes(id)
);

CREATE TABLE salaires (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    employe_id  INT NOT NULL,
    montant     DECIMAL(10,2) NOT NULL,
    date_debut  DATE NOT NULL,
    date_fin    DATE,
    FOREIGN KEY (employe_id) REFERENCES employes(id)
);

-- Donnees
INSERT INTO departements (nom, ville) VALUES
    ('Ingenierie',   'Paris'),
    ('Marketing',    'Lyon'),
    ('RH',           'Paris'),
    ('Finance',      'Marseille'),
    ('R&D',          'Toulouse');

INSERT INTO employes (prenom, nom, email, date_embauche, departement_id, manager_id) VALUES
    ('Alice',   'Dupont',  'alice@corp.fr',   '2020-03-15', 1, NULL),
    ('Bob',     'Martin',  'bob@corp.fr',     '2019-07-01', 1, 1),
    ('Charlie', 'Bernard', 'charlie@corp.fr', '2021-01-10', 2, 1),
    ('Diana',   'Petit',   'diana@corp.fr',   '2018-06-20', 3, NULL),
    ('Eve',     'Robert',  'eve@corp.fr',     '2022-09-05', 1, 1),
    ('Frank',   'Moreau',  'frank@corp.fr',   '2020-11-30', 2, 3),
    ('Grace',   'Laurent', 'grace@corp.fr',   '2017-04-12', 4, NULL),
    ('Hugo',    'Simon',   'hugo@corp.fr',    '2023-02-28', NULL, 4),
    ('Iris',    'Michel',  'iris@corp.fr',    '2021-08-15', 3, 4),
    ('Jean',    'Garcia',  'jean@corp.fr',    '2019-12-01', 1, 2);

INSERT INTO salaires (employe_id, montant, date_debut, date_fin) VALUES
    (1, 55000, '2020-03-15', '2022-12-31'),
    (1, 62000, '2023-01-01', NULL),
    (2, 48000, '2019-07-01', '2021-12-31'),
    (2, 53000, '2022-01-01', NULL),
    (3, 42000, '2021-01-10', NULL),
    (4, 51000, '2018-06-20', '2023-06-30'),
    (4, 58000, '2023-07-01', NULL),
    (5, 45000, '2022-09-05', NULL),
    (6, 39000, '2020-11-30', NULL),
    (7, 67000, '2017-04-12', NULL),
    (8, 35000, '2023-02-28', NULL),
    (9, 46000, '2021-08-15', NULL),
    (10, 50000, '2019-12-01', NULL);
```

---

## 1. Fonctions d'Agregation

Les fonctions d'agregation calculent une valeur unique a partir d'un ensemble de lignes.

### 1.1 Les cinq fonctions de base

```sql
-- COUNT : compter les lignes
SELECT COUNT(*) AS nombre_employes FROM employes;
-- Resultat : 10

-- COUNT sur une colonne (ignore les NULL)
SELECT COUNT(departement_id) AS avec_departement FROM employes;
-- Resultat : 9 (Hugo n'a pas de departement)

-- COUNT DISTINCT : compter les valeurs uniques
SELECT COUNT(DISTINCT departement_id) AS nb_departements FROM employes;
-- Resultat : 4

-- SUM : somme
SELECT SUM(montant) AS masse_salariale_totale
FROM salaires
WHERE date_fin IS NULL;  -- Salaires actuels uniquement

-- AVG : moyenne
SELECT AVG(montant) AS salaire_moyen
FROM salaires
WHERE date_fin IS NULL;

-- MIN et MAX
SELECT
    MIN(montant) AS salaire_min,
    MAX(montant) AS salaire_max,
    MAX(montant) - MIN(montant) AS ecart
FROM salaires
WHERE date_fin IS NULL;
```

> [!warning] COUNT(*) vs COUNT(colonne)
> - `COUNT(*)` : compte **toutes** les lignes, y compris celles avec des NULL
> - `COUNT(colonne)` : compte uniquement les lignes ou `colonne` n'est **pas** NULL
> - `COUNT(DISTINCT colonne)` : compte les valeurs **uniques** non NULL
>
> ```sql
> -- Avec la table employes (Hugo a departement_id = NULL) :
> SELECT COUNT(*) FROM employes;              -- 10
> SELECT COUNT(departement_id) FROM employes; -- 9
> SELECT COUNT(DISTINCT departement_id) FROM employes; -- 4
> ```

---

## 2. GROUP BY

`GROUP BY` regroupe les lignes partageant une meme valeur et permet d'appliquer des fonctions d'agregation par groupe.

```sql
-- Nombre d'employes par departement
SELECT
    d.nom AS departement,
    COUNT(e.id) AS nb_employes
FROM employes e
JOIN departements d ON e.departement_id = d.id
GROUP BY d.nom;
```

> [!example] Resultat
> ```
> +-------------+-------------+
> | departement | nb_employes |
> +-------------+-------------+
> | Ingenierie  |           4 |
> | Marketing   |           2 |
> | RH          |           2 |
> | Finance     |           1 |
> +-------------+-------------+
> ```

```sql
-- Salaire moyen actuel par departement
SELECT
    d.nom AS departement,
    COUNT(e.id) AS nb_employes,
    ROUND(AVG(s.montant), 2) AS salaire_moyen,
    MIN(s.montant) AS salaire_min,
    MAX(s.montant) AS salaire_max
FROM employes e
JOIN departements d ON e.departement_id = d.id
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
GROUP BY d.nom
ORDER BY salaire_moyen DESC;

-- GROUP BY sur plusieurs colonnes
SELECT
    d.ville,
    d.nom AS departement,
    COUNT(e.id) AS nb_employes
FROM employes e
JOIN departements d ON e.departement_id = d.id
GROUP BY d.ville, d.nom
ORDER BY d.ville, d.nom;
```

---

## 3. HAVING vs WHERE

> [!info] Difference fondamentale
> - **WHERE** filtre les **lignes individuelles** avant le regroupement
> - **HAVING** filtre les **groupes** apres le regroupement
>
> ```
> Ordre d'execution :
> FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
>         ↑                    ↑
>    Filtre les            Filtre les
>    lignes                groupes
> ```

```sql
-- WHERE : filtre avant GROUP BY
-- "Parmi les employes embauches apres 2020, combien par departement ?"
SELECT
    d.nom AS departement,
    COUNT(e.id) AS nb_employes
FROM employes e
JOIN departements d ON e.departement_id = d.id
WHERE e.date_embauche > '2020-01-01'   -- filtre les lignes
GROUP BY d.nom;

-- HAVING : filtre apres GROUP BY
-- "Quels departements ont plus de 2 employes ?"
SELECT
    d.nom AS departement,
    COUNT(e.id) AS nb_employes
FROM employes e
JOIN departements d ON e.departement_id = d.id
GROUP BY d.nom
HAVING COUNT(e.id) > 2;               -- filtre les groupes

-- Combiner WHERE et HAVING
-- "Parmi les employes embauches apres 2019,
--  quels departements ont un salaire moyen > 50000 ?"
SELECT
    d.nom AS departement,
    COUNT(e.id) AS nb_employes,
    ROUND(AVG(s.montant), 2) AS salaire_moyen
FROM employes e
JOIN departements d ON e.departement_id = d.id
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
WHERE e.date_embauche > '2019-01-01'
GROUP BY d.nom
HAVING AVG(s.montant) > 50000
ORDER BY salaire_moyen DESC;
```

> [!warning] Erreur frequente
> On ne peut **pas** utiliser une fonction d'agregation dans WHERE :
> ```sql
> -- ERREUR !
> SELECT departement_id, AVG(montant)
> FROM salaires
> WHERE AVG(montant) > 50000   -- Interdit !
> GROUP BY departement_id;
>
> -- CORRECT : utiliser HAVING
> SELECT departement_id, AVG(montant)
> FROM salaires
> GROUP BY departement_id
> HAVING AVG(montant) > 50000;
> ```

---

## 4. Les Jointures (JOINs)

Les jointures combinent des lignes de deux tables ou plus selon une condition de relation. C'est l'un des concepts les plus importants du SQL relationnel.

### 4.1 INNER JOIN

Retourne uniquement les lignes ayant une correspondance dans **les deux** tables.

```
Table A            Table B
+----+------+     +----+------+
| id | nom  |     | id | a_id |
+----+------+     +----+------+
|  1 | AAA  |     | 10 |  1   |
|  2 | BBB  |     | 20 |  3   |
|  3 | CCC  |     | 30 |  1   |
+----+------+     +----+------+

INNER JOIN (A.id = B.a_id) :
+----+-----+------+------+
| A  | nom | B.id | a_id |
+----+-----+------+------+
|  1 | AAA |  10  |  1   |  ← A.1 matche B.10
|  1 | AAA |  30  |  1   |  ← A.1 matche B.30
|  3 | CCC |  20  |  3   |  ← A.3 matche B.20
+----+-----+------+------+
BBB (id=2) est exclu : aucune correspondance dans B
```

```sql
-- Employes avec leur departement
SELECT
    e.prenom,
    e.nom,
    d.nom AS departement
FROM employes e
INNER JOIN departements d ON e.departement_id = d.id;
-- Hugo (departement_id = NULL) n'apparait pas
```

### 4.2 LEFT JOIN (LEFT OUTER JOIN)

Retourne **toutes** les lignes de la table de gauche, meme sans correspondance dans la table de droite (NULL dans ce cas).

```
LEFT JOIN (A.id = B.a_id) :
+----+-----+------+------+
| A  | nom | B.id | a_id |
+----+-----+------+------+
|  1 | AAA |  10  |  1   |
|  1 | AAA |  30  |  1   |
|  2 | BBB | NULL | NULL |  ← BBB garde, pas de match
|  3 | CCC |  20  |  3   |
+----+-----+------+------+
Toutes les lignes de A sont presentes
```

```sql
-- TOUS les employes, meme ceux sans departement
SELECT
    e.prenom,
    e.nom,
    COALESCE(d.nom, 'Non assigne') AS departement
FROM employes e
LEFT JOIN departements d ON e.departement_id = d.id;
-- Hugo apparait avec "Non assigne"

-- Trouver les employes SANS departement
SELECT e.prenom, e.nom
FROM employes e
LEFT JOIN departements d ON e.departement_id = d.id
WHERE d.id IS NULL;
```

### 4.3 RIGHT JOIN (RIGHT OUTER JOIN)

Retourne **toutes** les lignes de la table de droite, meme sans correspondance dans la table de gauche.

```
RIGHT JOIN (A.id = B.a_id) :
+------+------+------+------+
| A.id | nom  | B.id | a_id |
+------+------+------+------+
|  1   | AAA  |  10  |  1   |
|  1   | AAA  |  30  |  1   |
| NULL | NULL |  20  |  3   |  ← Si A n'avait pas id=3
+------+------+------+------+
Toutes les lignes de B sont presentes
```

```sql
-- Tous les departements, meme ceux sans employe
SELECT
    d.nom AS departement,
    e.prenom,
    e.nom
FROM employes e
RIGHT JOIN departements d ON e.departement_id = d.id;
-- "R&D" apparait avec NULL, NULL (aucun employe)
```

> [!tip] LEFT JOIN vs RIGHT JOIN
> En pratique, on utilise presque toujours `LEFT JOIN` et on reorganise les tables pour que la table "principale" soit a gauche. Un `RIGHT JOIN` peut toujours etre reecrit en `LEFT JOIN` en inversant l'ordre des tables.

### 4.4 FULL OUTER JOIN

Retourne **toutes** les lignes des **deux** tables, avec NULL la ou il n'y a pas de correspondance.

```
FULL OUTER JOIN (A.id = B.a_id) :
+------+------+------+------+
| A.id | nom  | B.id | a_id |
+------+------+------+------+
|  1   | AAA  |  10  |  1   |
|  1   | AAA  |  30  |  1   |
|  2   | BBB  | NULL | NULL |  ← Pas de match dans B
|  3   | CCC  |  20  |  3   |
| NULL | NULL |  40  |  5   |  ← Pas de match dans A
+------+------+------+------+
```

```sql
-- Tous les employes ET tous les departements
SELECT
    e.prenom,
    e.nom,
    d.nom AS departement
FROM employes e
FULL OUTER JOIN departements d ON e.departement_id = d.id;
-- Note : MySQL ne supporte pas FULL OUTER JOIN directement.
-- On simule avec UNION :

SELECT e.prenom, e.nom, d.nom AS departement
FROM employes e
LEFT JOIN departements d ON e.departement_id = d.id
UNION
SELECT e.prenom, e.nom, d.nom AS departement
FROM employes e
RIGHT JOIN departements d ON e.departement_id = d.id;
```

### 4.5 CROSS JOIN

Produit cartesien : chaque ligne de A est combinee avec chaque ligne de B.

```sql
-- Toutes les combinaisons employe-departement
SELECT e.prenom, d.nom
FROM employes e
CROSS JOIN departements d;
-- 10 employes × 5 departements = 50 lignes

-- Utilisation pratique : generer une grille
SELECT
    m.mois,
    d.nom AS departement
FROM (
    SELECT 1 AS mois UNION SELECT 2 UNION SELECT 3
    UNION SELECT 4 UNION SELECT 5 UNION SELECT 6
) m
CROSS JOIN departements d
ORDER BY m.mois, d.nom;
```

> [!warning] Attention au CROSS JOIN
> Le produit cartesien peut generer un nombre **enorme** de lignes.
> 1000 lignes × 1000 lignes = 1 000 000 de lignes resultantes.
> Ne l'utilisez que quand vous en avez reellement besoin.

### 4.6 Self-Join (auto-jointure)

Joindre une table avec elle-meme. Utile pour les relations hierarchiques.

```sql
-- Trouver chaque employe avec le nom de son manager
SELECT
    e.prenom AS employe_prenom,
    e.nom    AS employe_nom,
    m.prenom AS manager_prenom,
    m.nom    AS manager_nom
FROM employes e
LEFT JOIN employes m ON e.manager_id = m.id;
```

> [!example] Resultat
> ```
> +---------+----------+---------+----------+
> | emp_prn | emp_nom  | mgr_prn | mgr_nom  |
> +---------+----------+---------+----------+
> | Alice   | Dupont   | NULL    | NULL     |  ← pas de manager
> | Bob     | Martin   | Alice   | Dupont   |
> | Charlie | Bernard  | Alice   | Dupont   |
> | Diana   | Petit    | NULL    | NULL     |  ← pas de manager
> | Eve     | Robert   | Alice   | Dupont   |
> | Frank   | Moreau   | Charlie | Bernard  |
> | Grace   | Laurent  | NULL    | NULL     |
> | Hugo    | Simon    | Diana   | Petit    |
> | Iris    | Michel   | Diana   | Petit    |
> | Jean    | Garcia   | Bob     | Martin   |
> +---------+----------+---------+----------+
> ```

### 4.7 Jointure sur 3+ tables

```sql
-- Employes avec leur departement ET leur salaire actuel
SELECT
    e.prenom,
    e.nom,
    d.nom       AS departement,
    d.ville,
    s.montant   AS salaire_actuel
FROM employes e
JOIN departements d ON e.departement_id = d.id
JOIN salaires s     ON s.employe_id = e.id
WHERE s.date_fin IS NULL
ORDER BY s.montant DESC;
```

### Diagramme recapitulatif des JOINs

```
     ┌─────────────────────────────────────────────────────────┐
     │              RECAPITULATIF DES JOINTURES                │
     │                                                         │
     │  INNER JOIN        LEFT JOIN         RIGHT JOIN         │
     │   ┌───┐              ┌───┐              ┌───┐          │
     │  ┌┤   ├┐            ┌┤###├┐            ┌┤   ├┐         │
     │  │└─┬─┘│           │└─┬─┘│            │└─┬─┘│         │
     │  │ ┌┴┐ │           │ ┌┴┐ │            │ ┌┴┐ │         │
     │  │ │█│ │           │ │█│ │            │ │█│ │         │
     │  │ └┬┘ │           │ └┬┘ │            │ └┬┘ │         │
     │  │┌─┴─┐│           │┌─┴─┐│            │┌─┴─┐│         │
     │  └┤   ├┘            └┤   ├┘            └┤###├┘         │
     │   └───┘              └───┘              └───┘          │
     │  A ∩ B             A (tous)           B (tous)         │
     │                    + B si match       + A si match     │
     │                                                         │
     │  FULL OUTER JOIN     CROSS JOIN                         │
     │   ┌───┐              A × B                              │
     │  ┌┤###├┐             Chaque ligne de A                  │
     │  │└─┬─┘│            combinee avec chaque               │
     │  │ ┌┴┐ │            ligne de B                          │
     │  │ │█│ │                                                │
     │  │ └┬┘ │                                                │
     │  │┌─┴─┐│                                                │
     │  └┤###├┘                                                │
     │   └───┘                                                 │
     │  A + B (tous)                                           │
     │                                                         │
     │  █ = lignes avec correspondance                         │
     │  # = lignes SANS correspondance (completes avec NULL)   │
     └─────────────────────────────────────────────────────────┘
```

---

## 5. Sous-requetes (Subqueries)

Une sous-requete est une requete imbriquee dans une autre requete.

### 5.1 Sous-requete dans WHERE

```sql
-- Employes gagnant plus que la moyenne
SELECT e.prenom, e.nom, s.montant
FROM employes e
JOIN salaires s ON s.employe_id = e.id
WHERE s.date_fin IS NULL
  AND s.montant > (SELECT AVG(montant) FROM salaires WHERE date_fin IS NULL);

-- Employes du meme departement qu'Alice
SELECT prenom, nom
FROM employes
WHERE departement_id = (
    SELECT departement_id
    FROM employes
    WHERE prenom = 'Alice' AND nom = 'Dupont'
);

-- Employes dans des departements a Paris
SELECT prenom, nom
FROM employes
WHERE departement_id IN (
    SELECT id FROM departements WHERE ville = 'Paris'
);
```

### 5.2 Sous-requete dans FROM (table derivee)

```sql
-- Salaire moyen par departement, puis filtrer
SELECT dept_stats.departement, dept_stats.salaire_moyen
FROM (
    SELECT
        d.nom AS departement,
        AVG(s.montant) AS salaire_moyen
    FROM employes e
    JOIN departements d ON e.departement_id = d.id
    JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
    GROUP BY d.nom
) AS dept_stats
WHERE dept_stats.salaire_moyen > 45000
ORDER BY dept_stats.salaire_moyen DESC;
```

### 5.3 Sous-requete dans SELECT

```sql
-- Pour chaque employe, afficher le salaire moyen de son departement
SELECT
    e.prenom,
    e.nom,
    s.montant AS salaire,
    (SELECT AVG(s2.montant)
     FROM salaires s2
     JOIN employes e2 ON s2.employe_id = e2.id
     WHERE e2.departement_id = e.departement_id
       AND s2.date_fin IS NULL
    ) AS moyenne_departement
FROM employes e
JOIN salaires s ON s.employe_id = e.id
WHERE s.date_fin IS NULL;
```

### 5.4 Sous-requete correlee

Une sous-requete **correlee** fait reference a la requete externe. Elle est re-executee pour chaque ligne de la requete externe.

```sql
-- Employes dont le salaire est superieur a la moyenne de leur departement
SELECT e.prenom, e.nom, s.montant
FROM employes e
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
WHERE s.montant > (
    SELECT AVG(s2.montant)
    FROM salaires s2
    JOIN employes e2 ON s2.employe_id = e2.id
    WHERE e2.departement_id = e.departement_id  -- Reference a la requete externe
      AND s2.date_fin IS NULL
);
```

> [!info] Performance des sous-requetes correlees
> Les sous-requetes correlees sont executees **une fois par ligne** de la requete externe. Elles peuvent etre lentes sur de grandes tables. Preferez les JOINs ou les CTE quand c'est possible.

---

## 6. EXISTS et NOT EXISTS

`EXISTS` teste si une sous-requete retourne au moins une ligne.

```sql
-- Departements qui ont au moins un employe
SELECT d.nom
FROM departements d
WHERE EXISTS (
    SELECT 1 FROM employes e WHERE e.departement_id = d.id
);

-- Departements SANS employe
SELECT d.nom
FROM departements d
WHERE NOT EXISTS (
    SELECT 1 FROM employes e WHERE e.departement_id = d.id
);
-- Resultat : R&D (aucun employe dans ce departement)

-- Employes ayant eu au moins une augmentation
SELECT DISTINCT e.prenom, e.nom
FROM employes e
WHERE EXISTS (
    SELECT 1
    FROM salaires s
    WHERE s.employe_id = e.id
    GROUP BY s.employe_id
    HAVING COUNT(*) > 1
);
```

> [!tip] EXISTS vs IN
> - `EXISTS` est souvent plus performant que `IN` pour les grandes tables
> - `EXISTS` s'arrete des qu'une correspondance est trouvee
> - `IN` charge l'ensemble complet de la sous-requete en memoire
> ```sql
> -- Ces deux requetes donnent le meme resultat :
> -- Version IN
> SELECT * FROM employes WHERE departement_id IN (SELECT id FROM departements WHERE ville = 'Paris');
> -- Version EXISTS (souvent plus rapide)
> SELECT * FROM employes e WHERE EXISTS (SELECT 1 FROM departements d WHERE d.id = e.departement_id AND d.ville = 'Paris');
> ```

---

## 7. UNION, INTERSECT, EXCEPT

### 7.1 UNION / UNION ALL

```sql
-- UNION : combine les resultats de deux requetes (supprime les doublons)
SELECT prenom, nom, 'Manager' AS role
FROM employes
WHERE id IN (SELECT DISTINCT manager_id FROM employes WHERE manager_id IS NOT NULL)
UNION
SELECT prenom, nom, 'Employe' AS role
FROM employes
WHERE id NOT IN (SELECT DISTINCT manager_id FROM employes WHERE manager_id IS NOT NULL);

-- UNION ALL : conserve les doublons (plus rapide)
SELECT nom, ville FROM departements WHERE ville = 'Paris'
UNION ALL
SELECT nom, ville FROM departements WHERE nom LIKE '%e%';
-- Si un departement parisien contient 'e', il apparait deux fois
```

> [!warning] Regles pour UNION
> Les deux requetes doivent avoir :
> - Le **meme nombre** de colonnes
> - Des **types compatibles** dans le meme ordre
> ```sql
> -- ERREUR : nombre de colonnes different
> SELECT prenom, nom FROM employes
> UNION
> SELECT nom FROM departements;
> ```

### 7.2 INTERSECT et EXCEPT

```sql
-- INTERSECT : lignes presentes dans LES DEUX requetes
-- (Supporte par PostgreSQL, pas par MySQL avant 8.0.31)
SELECT departement_id FROM employes WHERE date_embauche < '2020-01-01'
INTERSECT
SELECT departement_id FROM employes WHERE date_embauche >= '2020-01-01';
-- Departements ayant des employes anciens ET recents

-- EXCEPT (ou MINUS en Oracle) : lignes de la 1ere absentes de la 2nde
SELECT id FROM departements
EXCEPT
SELECT DISTINCT departement_id FROM employes WHERE departement_id IS NOT NULL;
-- IDs de departements sans aucun employe
```

---

## 8. CASE WHEN : Expressions Conditionnelles

`CASE WHEN` est l'equivalent du if/else en SQL.

```sql
-- Classification des salaires
SELECT
    e.prenom,
    e.nom,
    s.montant,
    CASE
        WHEN s.montant >= 60000 THEN 'Senior'
        WHEN s.montant >= 45000 THEN 'Confirme'
        WHEN s.montant >= 35000 THEN 'Junior'
        ELSE 'Stagiaire'
    END AS niveau
FROM employes e
JOIN salaires s ON s.employe_id = e.id
WHERE s.date_fin IS NULL
ORDER BY s.montant DESC;

-- CASE dans un GROUP BY pour creer des categories
SELECT
    CASE
        WHEN YEAR(date_embauche) < 2020 THEN 'Avant 2020'
        WHEN YEAR(date_embauche) < 2022 THEN '2020-2021'
        ELSE '2022 et apres'
    END AS periode_embauche,
    COUNT(*) AS nb_employes
FROM employes
GROUP BY
    CASE
        WHEN YEAR(date_embauche) < 2020 THEN 'Avant 2020'
        WHEN YEAR(date_embauche) < 2022 THEN '2020-2021'
        ELSE '2022 et apres'
    END;

-- CASE dans un agrege : comptage conditionnel (pivot)
SELECT
    d.nom AS departement,
    COUNT(*) AS total,
    SUM(CASE WHEN s.montant >= 50000 THEN 1 ELSE 0 END) AS salaire_50k_plus,
    SUM(CASE WHEN s.montant < 50000  THEN 1 ELSE 0 END) AS salaire_sous_50k
FROM employes e
JOIN departements d ON e.departement_id = d.id
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
GROUP BY d.nom;
```

> [!example] Resultat du pivot
> ```
> +-------------+-------+------------------+------------------+
> | departement | total | salaire_50k_plus | salaire_sous_50k |
> +-------------+-------+------------------+------------------+
> | Ingenierie  |     4 |                2 |                2 |
> | Marketing   |     2 |                0 |                2 |
> | RH          |     2 |                1 |                1 |
> | Finance     |     1 |                1 |                0 |
> +-------------+-------+------------------+------------------+
> ```

---

## 9. Fonctions de Fenetrage (Window Functions)

Les fonctions de fenetrage effectuent des calculs sur un ensemble de lignes liees a la ligne courante, **sans reduire le nombre de lignes** (contrairement a GROUP BY).

### 9.1 Syntaxe generale

```sql
fonction_fenetre() OVER (
    [PARTITION BY colonne1, colonne2, ...]
    [ORDER BY colonne3 [ASC|DESC], ...]
)
```

- **PARTITION BY** : divise les lignes en groupes (comme GROUP BY, mais sans agregation)
- **ORDER BY** : definit l'ordre dans chaque partition

### 9.2 ROW_NUMBER, RANK, DENSE_RANK

```sql
-- ROW_NUMBER : numero sequentiel unique
SELECT
    e.prenom,
    e.nom,
    d.nom AS departement,
    s.montant,
    ROW_NUMBER() OVER (ORDER BY s.montant DESC) AS classement
FROM employes e
JOIN departements d ON e.departement_id = d.id
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL;

-- Avec PARTITION BY : classement par departement
SELECT
    e.prenom,
    e.nom,
    d.nom AS departement,
    s.montant,
    ROW_NUMBER() OVER (PARTITION BY d.id ORDER BY s.montant DESC) AS rang_dept
FROM employes e
JOIN departements d ON e.departement_id = d.id
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL;
```

> [!info] ROW_NUMBER vs RANK vs DENSE_RANK
> ```
> Donnees : 100, 90, 90, 80
>
> ROW_NUMBER : 1, 2, 3, 4  (toujours unique, arbitraire pour les ex-aequo)
> RANK       : 1, 2, 2, 4  (saute le rang apres les ex-aequo)
> DENSE_RANK : 1, 2, 2, 3  (pas de saut apres les ex-aequo)
> ```

```sql
-- Comparaison des trois
SELECT
    e.prenom,
    s.montant,
    ROW_NUMBER() OVER (ORDER BY s.montant DESC) AS row_num,
    RANK()       OVER (ORDER BY s.montant DESC) AS rang,
    DENSE_RANK() OVER (ORDER BY s.montant DESC) AS rang_dense
FROM employes e
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL;
```

### 9.3 Fonctions d'agregation en fenetrage

```sql
-- Salaire de chaque employe compare a la moyenne de son departement
SELECT
    e.prenom,
    e.nom,
    d.nom AS departement,
    s.montant,
    ROUND(AVG(s.montant) OVER (PARTITION BY d.id), 2) AS moy_dept,
    s.montant - ROUND(AVG(s.montant) OVER (PARTITION BY d.id), 2) AS ecart_moyenne
FROM employes e
JOIN departements d ON e.departement_id = d.id
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL;

-- Somme cumulative
SELECT
    e.prenom,
    e.date_embauche,
    s.montant,
    SUM(s.montant) OVER (ORDER BY e.date_embauche) AS cumul_salaires
FROM employes e
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
ORDER BY e.date_embauche;
```

### 9.4 Utilisation pratique : Top N par groupe

```sql
-- Le mieux paye de chaque departement
SELECT * FROM (
    SELECT
        e.prenom,
        e.nom,
        d.nom AS departement,
        s.montant,
        ROW_NUMBER() OVER (PARTITION BY d.id ORDER BY s.montant DESC) AS rn
    FROM employes e
    JOIN departements d ON e.departement_id = d.id
    JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
) ranked
WHERE rn = 1;
```

---

## 10. Common Table Expressions (CTE)

Les CTE definissent des sous-requetes nommees en debut de requete avec le mot-cle `WITH`. Elles amellorent la lisibilite et permettent la reutilisation.

### 10.1 CTE simple

```sql
-- Sans CTE (sous-requete imbriquee, difficile a lire)
SELECT * FROM (
    SELECT d.nom, AVG(s.montant) AS moy
    FROM employes e
    JOIN departements d ON e.departement_id = d.id
    JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
    GROUP BY d.nom
) stats WHERE moy > 50000;

-- Avec CTE (beaucoup plus lisible)
WITH stats_dept AS (
    SELECT
        d.nom AS departement,
        AVG(s.montant) AS salaire_moyen
    FROM employes e
    JOIN departements d ON e.departement_id = d.id
    JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
    GROUP BY d.nom
)
SELECT departement, ROUND(salaire_moyen, 2)
FROM stats_dept
WHERE salaire_moyen > 50000;
```

### 10.2 CTE multiples

```sql
WITH salaires_actuels AS (
    SELECT employe_id, montant
    FROM salaires
    WHERE date_fin IS NULL
),
stats_dept AS (
    SELECT
        e.departement_id,
        d.nom AS departement,
        COUNT(*) AS nb_employes,
        AVG(sa.montant) AS salaire_moyen
    FROM employes e
    JOIN departements d ON e.departement_id = d.id
    JOIN salaires_actuels sa ON sa.employe_id = e.id
    GROUP BY e.departement_id, d.nom
)
SELECT
    sd.departement,
    sd.nb_employes,
    ROUND(sd.salaire_moyen, 2) AS salaire_moyen,
    e.prenom || ' ' || e.nom AS meilleur_paye
FROM stats_dept sd
JOIN employes e ON e.departement_id = sd.departement_id
JOIN salaires_actuels sa ON sa.employe_id = e.id
WHERE sa.montant = (
    SELECT MAX(sa2.montant)
    FROM employes e2
    JOIN salaires_actuels sa2 ON sa2.employe_id = e2.id
    WHERE e2.departement_id = sd.departement_id
);
```

### 10.3 CTE recursive

```sql
-- Hierarchie complete des managers
WITH RECURSIVE hierarchie AS (
    -- Cas de base : employes sans manager (racines)
    SELECT
        id,
        prenom,
        nom,
        manager_id,
        0 AS niveau,
        CAST(CONCAT(prenom, ' ', nom) AS CHAR(500)) AS chemin
    FROM employes
    WHERE manager_id IS NULL

    UNION ALL

    -- Cas recursif : employes avec un manager deja dans la hierarchie
    SELECT
        e.id,
        e.prenom,
        e.nom,
        e.manager_id,
        h.niveau + 1,
        CAST(CONCAT(h.chemin, ' > ', e.prenom, ' ', e.nom) AS CHAR(500))
    FROM employes e
    JOIN hierarchie h ON e.manager_id = h.id
)
SELECT
    REPEAT('  ', niveau) || prenom || ' ' || nom AS employe,
    niveau,
    chemin
FROM hierarchie
ORDER BY chemin;
```

> [!example] Resultat de la hierarchie
> ```
> employe                   | niveau | chemin
> Alice Dupont              |   0    | Alice Dupont
>   Bob Martin              |   1    | Alice Dupont > Bob Martin
>     Jean Garcia           |   2    | Alice Dupont > Bob Martin > Jean Garcia
>   Charlie Bernard         |   1    | Alice Dupont > Charlie Bernard
>     Frank Moreau          |   2    | Alice Dupont > Charlie Bernard > Frank Moreau
>   Eve Robert              |   1    | Alice Dupont > Eve Robert
> Diana Petit               |   0    | Diana Petit
>   Hugo Simon              |   1    | Diana Petit > Hugo Simon
>   Iris Michel             |   1    | Diana Petit > Iris Michel
> Grace Laurent             |   0    | Grace Laurent
> ```

---

## Carte Mentale ASCII

```
                    ┌───────────────────────────┐
                    │  REQUETES AVANCEES SQL     │
                    └─────────────┬─────────────┘
                                  │
       ┌──────────────┬───────────┼───────────┬──────────────┐
       │              │           │           │              │
  ┌────▼────┐   ┌─────▼─────┐ ┌──▼───┐  ┌───▼────┐  ┌──────▼──────┐
  │Agregation│  │  JOINs    │ │Sous- │  │CASE    │  │  Window     │
  │         │   │           │ │req.  │  │WHEN    │  │  Functions  │
  └────┬────┘   └─────┬─────┘ └──┬───┘  └───┬────┘  └──────┬──────┘
       │              │          │           │              │
  COUNT,SUM     INNER JOIN    WHERE      Conditions   ROW_NUMBER
  AVG,MIN       LEFT JOIN     FROM       Categories   RANK
  MAX           RIGHT JOIN    SELECT     Pivot        DENSE_RANK
  GROUP BY      FULL OUTER    Correlee                OVER()
  HAVING        CROSS JOIN    EXISTS                  PARTITION BY
                Self-Join                             SUM() OVER
                3+ tables                             AVG() OVER
       │              │          │
       │         ┌────▼────┐     │         ┌──────────┐
       │         │ UNION   │     │         │   CTE    │
       │         │ UNION ALL│    │         │  WITH AS │
       │         │INTERSECT│     │         │ Recursive│
       │         │ EXCEPT  │     │         └──────────┘
       │         └─────────┘     │
       │                         │
  ┌────▼─────────────────────────▼───┐
  │   WHERE filtre les LIGNES        │
  │   HAVING filtre les GROUPES      │
  │   (apres GROUP BY)               │
  └──────────────────────────────────┘
```

---

## Exercices

### Exercice 1 : Agregation et GROUP BY
Sur la base `entreprise` :
1. Calculez le salaire moyen, min et max (actuels) par ville de departement
2. Trouvez les departements ou le salaire moyen actuel depasse 50 000 euros
3. Comptez le nombre d'employes embauches par annee

### Exercice 2 : Jointures
1. Listez tous les departements avec le nombre d'employes (incluant ceux avec 0 employes)
2. Pour chaque employe, affichez son nom, son departement, son salaire actuel, et le nom de son manager
3. Trouvez les employes qui n'ont jamais eu d'augmentation (un seul enregistrement dans salaires)

### Exercice 3 : Sous-requetes et CTE
1. Trouvez les employes dont le salaire actuel est superieur a celui de leur manager
2. Avec une CTE, calculez l'anciennete de chaque employe en annees et classez-les par departement
3. Utilisez EXISTS pour trouver les departements dont aucun employe ne gagne plus de 55 000 euros

### Exercice 4 : Fonctions de fenetrage
1. Classez les employes par salaire au sein de chaque departement (RANK)
2. Pour chaque employe, affichez son salaire et le pourcentage qu'il represente par rapport au total de son departement
3. Calculez la difference de salaire entre chaque employe et le mieux paye de son departement

---

## Liens

- [[01 - Introduction au SQL]] - Les bases : SELECT, WHERE, INSERT, UPDATE, DELETE
- [[03 - Conception de Bases de Donnees]] - Normalisation, cles etrangeres, indexes
- [[04 - SQL Avance et Administration]] - Transactions, vues, procedures, securite
