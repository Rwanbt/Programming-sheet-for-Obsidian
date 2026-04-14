# SQL Avance et Administration

Ce cours couvre les aspects avances du SQL et l'administration de bases de donnees : transactions et proprietes ACID, vues, procedures stockees, fonctions, triggers, securite (en particulier les injections SQL), gestion des utilisateurs et optimisation des requetes. Ces competences sont essentielles pour tout developpeur qui travaille avec des bases de donnees en production.

Apres avoir maitrise la conception et les requetes, il faut comprendre comment **proteger**, **automatiser** et **optimiser** une base de donnees.

> [!tip] Analogie
> Si la base de donnees est un **coffre-fort** contenant vos donnees, les transactions sont les **serrures** qui garantissent la coherence, les vues sont des **vitres teintees** qui montrent seulement ce qu'on veut, les procedures stockees sont des **robots internes** qui automatisent les taches, et la securite est le **systeme d'alarme** qui empeche les intrus d'entrer. L'optimisation, c'est l'**organisation interne** pour retrouver rapidement n'importe quel objet.

---

## 1. Transactions

Une **transaction** est un ensemble d'operations SQL qui doivent etre executees **comme un tout indivisible**. Soit toutes les operations reussissent, soit aucune ne s'applique.

### 1.1 Syntaxe de base

```sql
-- Demarrer une transaction
BEGIN;
-- ou START TRANSACTION; (MySQL)

-- Operations SQL
UPDATE comptes SET solde = solde - 500 WHERE id = 1;
UPDATE comptes SET solde = solde + 500 WHERE id = 2;

-- Valider (appliquer definitivement)
COMMIT;

-- OU annuler (tout defaire)
ROLLBACK;
```

> [!example] Cas concret : virement bancaire
> ```sql
> -- Sans transaction : DANGER
> UPDATE comptes SET solde = solde - 500 WHERE id = 1;
> -- Si le serveur plante ICI, l'argent a disparu !
> UPDATE comptes SET solde = solde + 500 WHERE id = 2;
>
> -- Avec transaction : SUR
> BEGIN;
>     UPDATE comptes SET solde = solde - 500 WHERE id = 1;
>     -- Si erreur ici, ROLLBACK automatique : rien n'a change
>     UPDATE comptes SET solde = solde + 500 WHERE id = 2;
> COMMIT;
> -- Les deux updates s'appliquent ensemble ou pas du tout
> ```

### 1.2 SAVEPOINT

```sql
BEGIN;

UPDATE inventaire SET stock = stock - 10 WHERE produit_id = 1;
SAVEPOINT avant_deuxieme_produit;

UPDATE inventaire SET stock = stock - 5 WHERE produit_id = 2;
-- Oops, erreur sur le 2e produit seulement
ROLLBACK TO SAVEPOINT avant_deuxieme_produit;
-- Le 1er UPDATE est preserve

-- Continuer avec une alternative
UPDATE inventaire SET stock = stock - 3 WHERE produit_id = 3;
COMMIT;
```

---

## 2. Proprietes ACID

Les transactions garantissent les proprietes **ACID**, fondement de la fiabilite des bases relationnelles.

```
┌────────────────────────────────────────────────────────────────┐
│                    PROPRIETES ACID                              │
│                                                                │
│  A - Atomicite (Atomicity)                                     │
│  ─────────────────────────                                     │
│  Tout ou rien. Une transaction est indivisible.                │
│  Si une seule operation echoue, toutes sont annulees.          │
│                                                                │
│  Exemple : un virement debite ET credite, ou ne fait rien.     │
│                                                                │
│  C - Coherence (Consistency)                                   │
│  ──────────────────────────                                    │
│  La base passe d'un etat valide a un autre etat valide.        │
│  Toutes les contraintes (CHECK, FK, UNIQUE) sont respectees.   │
│                                                                │
│  Exemple : le solde total des comptes reste constant apres     │
│  un virement.                                                  │
│                                                                │
│  I - Isolation                                                 │
│  ────────────                                                  │
│  Les transactions concurrentes ne s'interferent pas.           │
│  Chaque transaction "voit" la base comme si elle etait seule.  │
│                                                                │
│  Exemple : deux virements simultanes ne causent pas            │
│  d'incoherence.                                                │
│                                                                │
│  D - Durabilite (Durability)                                   │
│  ──────────────────────────                                    │
│  Une fois COMMIT effectue, les donnees sont permanentes,       │
│  meme en cas de panne de courant ou crash du serveur.          │
│                                                                │
│  Exemple : apres COMMIT, le virement est enregistre sur        │
│  disque, pas seulement en memoire.                             │
└────────────────────────────────────────────────────────────────┘
```

---

## 3. Niveaux d'Isolation

Le niveau d'isolation definit comment les transactions concurrentes interagissent. Plus le niveau est eleve, plus l'isolation est forte, mais plus les performances diminuent.

```
┌────────────────────┬──────────────┬───────────────┬──────────────┐
│ Niveau             │ Dirty Read   │Non-Repeatable │ Phantom Read │
│                    │(lecture sale)│ Read          │(lecture fant.)│
├────────────────────┼──────────────┼───────────────┼──────────────┤
│ READ UNCOMMITTED   │   Possible   │   Possible    │   Possible   │
│ (le plus permissif)│              │               │              │
├────────────────────┼──────────────┼───────────────┼──────────────┤
│ READ COMMITTED     │   Impossible │   Possible    │   Possible   │
│ (defaut PostgreSQL)│              │               │              │
├────────────────────┼──────────────┼───────────────┼──────────────┤
│ REPEATABLE READ    │   Impossible │   Impossible  │   Possible   │
│ (defaut MySQL)     │              │               │              │
├────────────────────┼──────────────┼───────────────┼──────────────┤
│ SERIALIZABLE       │   Impossible │   Impossible  │   Impossible │
│ (le plus strict)   │              │               │              │
└────────────────────┴──────────────┴───────────────┴──────────────┘
```

> [!info] Explication des anomalies
> - **Dirty Read** : lire des donnees modifiees par une transaction non encore committee
> - **Non-Repeatable Read** : relire la meme ligne donne un resultat different (une autre transaction l'a modifiee entre-temps)
> - **Phantom Read** : re-executer une requete retourne des lignes supplementaires (une autre transaction a insere entre-temps)

```sql
-- Changer le niveau d'isolation
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- PostgreSQL : pour la session
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- MySQL : pour la session
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

---

## 4. Vues (Views)

Une **vue** est une requete nommee et sauvegardee. Elle se comporte comme une table virtuelle.

### 4.1 Creer une vue

```sql
-- Vue simple : employes avec leur departement
CREATE VIEW v_employes_departement AS
SELECT
    e.id,
    e.prenom,
    e.nom,
    e.email,
    d.nom AS departement,
    d.ville
FROM employes e
LEFT JOIN departements d ON e.departement_id = d.id;

-- Utiliser la vue comme une table
SELECT * FROM v_employes_departement WHERE ville = 'Paris';

-- Vue avec agregation
CREATE VIEW v_stats_departement AS
SELECT
    d.nom AS departement,
    COUNT(e.id) AS nb_employes,
    ROUND(AVG(s.montant), 2) AS salaire_moyen,
    MIN(s.montant) AS salaire_min,
    MAX(s.montant) AS salaire_max
FROM departements d
LEFT JOIN employes e ON e.departement_id = d.id
LEFT JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
GROUP BY d.nom;

-- Requeter les stats simplement
SELECT * FROM v_stats_departement ORDER BY salaire_moyen DESC;
```

### 4.2 Modifier et supprimer une vue

```sql
-- Modifier une vue (la remplacer)
CREATE OR REPLACE VIEW v_employes_departement AS
SELECT
    e.id,
    e.prenom,
    e.nom,
    CONCAT(e.prenom, ' ', e.nom) AS nom_complet,
    d.nom AS departement
FROM employes e
LEFT JOIN departements d ON e.departement_id = d.id;

-- Supprimer une vue
DROP VIEW IF EXISTS v_employes_departement;
```

### 4.3 Vues modifiables et WITH CHECK OPTION

```sql
-- Vue des employes actifs
CREATE VIEW v_employes_actifs AS
SELECT id, prenom, nom, email, actif
FROM employes
WHERE actif = TRUE
WITH CHECK OPTION;

-- On peut inserer/modifier via la vue, MAIS :
-- WITH CHECK OPTION empeche d'inserer des lignes qui ne satisfont pas le WHERE

-- OK : inserer un employe actif
INSERT INTO v_employes_actifs (prenom, nom, email, actif)
VALUES ('Test', 'User', 'test@corp.fr', TRUE);

-- ERREUR : actif = FALSE viole la condition de la vue
INSERT INTO v_employes_actifs (prenom, nom, email, actif)
VALUES ('Test', 'Inactif', 'test2@corp.fr', FALSE);
-- Erreur : CHECK OPTION failed
```

> [!tip] Quand utiliser les vues ?
> - **Simplification** : cacher la complexite des jointures derriere un nom simple
> - **Securite** : donner acces a une vue filtree plutot qu'a la table complete
> - **Abstraction** : si la structure change, il suffit de modifier la vue
> - **Reutilisation** : eviter de repeter des requetes complexes

---

## 5. Procedures Stockees (Stored Procedures)

Une procedure stockee est un bloc de code SQL nomme, stocke dans la base et executable a la demande.

### 5.1 Creer une procedure

```sql
-- Procedure simple sans parametre
DELIMITER //
CREATE PROCEDURE sp_liste_employes()
BEGIN
    SELECT
        e.prenom,
        e.nom,
        d.nom AS departement,
        s.montant AS salaire
    FROM employes e
    LEFT JOIN departements d ON e.departement_id = d.id
    LEFT JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
    ORDER BY e.nom;
END //
DELIMITER ;

-- Appeler la procedure
CALL sp_liste_employes();
```

### 5.2 Parametres IN, OUT, INOUT

```sql
-- Parametre IN : valeur passee a la procedure
DELIMITER //
CREATE PROCEDURE sp_employes_par_departement(
    IN p_departement VARCHAR(50)
)
BEGIN
    SELECT e.prenom, e.nom, s.montant
    FROM employes e
    JOIN departements d ON e.departement_id = d.id
    JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
    WHERE d.nom = p_departement
    ORDER BY s.montant DESC;
END //
DELIMITER ;

CALL sp_employes_par_departement('Ingenierie');

-- Parametre OUT : valeur retournee par la procedure
DELIMITER //
CREATE PROCEDURE sp_salaire_moyen(
    IN  p_departement VARCHAR(50),
    OUT p_moyenne DECIMAL(10,2)
)
BEGIN
    SELECT AVG(s.montant) INTO p_moyenne
    FROM employes e
    JOIN departements d ON e.departement_id = d.id
    JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL
    WHERE d.nom = p_departement;
END //
DELIMITER ;

-- Utilisation
CALL sp_salaire_moyen('Ingenierie', @moy);
SELECT @moy AS salaire_moyen_ingenierie;

-- Parametre INOUT : valeur modifiee
DELIMITER //
CREATE PROCEDURE sp_augmenter_salaire(
    IN    p_employe_id INT,
    INOUT p_montant DECIMAL(10,2)
)
BEGIN
    -- p_montant contient le pourcentage d'augmentation
    -- On le remplace par le nouveau salaire
    DECLARE ancien_salaire DECIMAL(10,2);

    SELECT montant INTO ancien_salaire
    FROM salaires
    WHERE employe_id = p_employe_id AND date_fin IS NULL;

    SET p_montant = ancien_salaire * (1 + p_montant / 100);

    -- Cloturer l'ancien salaire
    UPDATE salaires SET date_fin = CURDATE()
    WHERE employe_id = p_employe_id AND date_fin IS NULL;

    -- Inserer le nouveau salaire
    INSERT INTO salaires (employe_id, montant, date_debut)
    VALUES (p_employe_id, p_montant, CURDATE());
END //
DELIMITER ;

-- Augmenter de 10%
SET @augmentation = 10.00;
CALL sp_augmenter_salaire(1, @augmentation);
SELECT @augmentation AS nouveau_salaire;
```

### 5.3 PostgreSQL : syntaxe PL/pgSQL

```sql
-- PostgreSQL utilise une syntaxe differente
CREATE OR REPLACE PROCEDURE sp_virement(
    p_de_compte INT,
    p_vers_compte INT,
    p_montant DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE comptes SET solde = solde - p_montant WHERE id = p_de_compte;
    UPDATE comptes SET solde = solde + p_montant WHERE id = p_vers_compte;

    -- Verifier que le solde est positif
    IF (SELECT solde FROM comptes WHERE id = p_de_compte) < 0 THEN
        RAISE EXCEPTION 'Solde insuffisant';
    END IF;
END;
$$;

CALL sp_virement(1, 2, 500.00);
```

---

## 6. Fonctions Utilisateur (User-Defined Functions)

A la difference des procedures, les fonctions **retournent une valeur** et peuvent etre utilisees dans des requetes SELECT.

```sql
-- MySQL
DELIMITER //
CREATE FUNCTION fn_anciennete(p_date_embauche DATE)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, p_date_embauche, CURDATE());
END //
DELIMITER ;

-- Utilisation dans une requete
SELECT
    prenom,
    nom,
    date_embauche,
    fn_anciennete(date_embauche) AS annees_anciennete
FROM employes
ORDER BY annees_anciennete DESC;

-- PostgreSQL
CREATE OR REPLACE FUNCTION fn_anciennete(p_date DATE)
RETURNS INT
LANGUAGE plpgsql
IMMUTABLE
AS $$
BEGIN
    RETURN EXTRACT(YEAR FROM age(CURRENT_DATE, p_date));
END;
$$;
```

> [!info] DETERMINISTIC / IMMUTABLE
> - **DETERMINISTIC** (MySQL) / **IMMUTABLE** (PostgreSQL) : la fonction retourne toujours le meme resultat pour les memes parametres. Le SGBD peut optimiser en mettant en cache.
> - **NOT DETERMINISTIC** / **VOLATILE** : le resultat peut varier (ex: utilise CURDATE(), RANDOM()).

```sql
-- Fonction plus complexe : calculer une prime
DELIMITER //
CREATE FUNCTION fn_prime(
    p_salaire DECIMAL(10,2),
    p_anciennete INT
)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE prime DECIMAL(10,2);

    IF p_anciennete >= 10 THEN
        SET prime = p_salaire * 0.15;
    ELSEIF p_anciennete >= 5 THEN
        SET prime = p_salaire * 0.10;
    ELSEIF p_anciennete >= 2 THEN
        SET prime = p_salaire * 0.05;
    ELSE
        SET prime = 0;
    END IF;

    RETURN prime;
END //
DELIMITER ;

-- Utiliser dans une requete
SELECT
    e.prenom,
    e.nom,
    s.montant AS salaire,
    fn_anciennete(e.date_embauche) AS anciennete,
    fn_prime(s.montant, fn_anciennete(e.date_embauche)) AS prime
FROM employes e
JOIN salaires s ON s.employe_id = e.id AND s.date_fin IS NULL;
```

---

## 7. Triggers (Declencheurs)

Un **trigger** est un bloc de code execute **automatiquement** en reponse a un evenement (INSERT, UPDATE, DELETE) sur une table.

### 7.1 Syntaxe et types

```
┌────────────────────────────────────────────────────┐
│              TYPES DE TRIGGERS                      │
│                                                    │
│  BEFORE INSERT  : avant l'insertion                │
│  AFTER INSERT   : apres l'insertion                │
│  BEFORE UPDATE  : avant la modification            │
│  AFTER UPDATE   : apres la modification            │
│  BEFORE DELETE  : avant la suppression             │
│  AFTER DELETE   : apres la suppression             │
│                                                    │
│  Variables speciales :                             │
│  NEW.colonne : la nouvelle valeur (INSERT/UPDATE)  │
│  OLD.colonne : l'ancienne valeur (UPDATE/DELETE)   │
└────────────────────────────────────────────────────┘
```

### 7.2 Exemples pratiques

```sql
-- Trigger BEFORE INSERT : valider et formater les donnees
DELIMITER //
CREATE TRIGGER trg_employes_before_insert
BEFORE INSERT ON employes
FOR EACH ROW
BEGIN
    -- Mettre le nom en majuscules
    SET NEW.nom = UPPER(NEW.nom);
    -- Mettre la premiere lettre du prenom en majuscule
    SET NEW.prenom = CONCAT(
        UPPER(LEFT(NEW.prenom, 1)),
        LOWER(SUBSTRING(NEW.prenom, 2))
    );
    -- Email en minuscules
    SET NEW.email = LOWER(NEW.email);
END //
DELIMITER ;

-- Trigger AFTER INSERT : creer un log
CREATE TABLE audit_log (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    table_nom   VARCHAR(50),
    operation   VARCHAR(10),
    enreg_id    INT,
    details     TEXT,
    date_action DATETIME DEFAULT CURRENT_TIMESTAMP,
    utilisateur VARCHAR(100)
);

DELIMITER //
CREATE TRIGGER trg_employes_after_insert
AFTER INSERT ON employes
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_nom, operation, enreg_id, details, utilisateur)
    VALUES (
        'employes',
        'INSERT',
        NEW.id,
        CONCAT('Nouvel employe : ', NEW.prenom, ' ', NEW.nom),
        CURRENT_USER()
    );
END //
DELIMITER ;

-- Trigger BEFORE UPDATE : empecher certaines modifications
DELIMITER //
CREATE TRIGGER trg_salaires_before_update
BEFORE UPDATE ON salaires
FOR EACH ROW
BEGIN
    -- Empecher de diminuer un salaire de plus de 10%
    IF NEW.montant < OLD.montant * 0.9 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Impossible de reduire le salaire de plus de 10%';
    END IF;
END //
DELIMITER ;

-- Trigger AFTER DELETE : archiver les donnees supprimees
CREATE TABLE employes_archives (
    id          INT,
    prenom      VARCHAR(50),
    nom         VARCHAR(50),
    email       VARCHAR(100),
    date_suppression DATETIME DEFAULT CURRENT_TIMESTAMP
);

DELIMITER //
CREATE TRIGGER trg_employes_after_delete
AFTER DELETE ON employes
FOR EACH ROW
BEGIN
    INSERT INTO employes_archives (id, prenom, nom, email)
    VALUES (OLD.id, OLD.prenom, OLD.nom, OLD.email);
END //
DELIMITER ;
```

> [!warning] Precautions avec les triggers
> - Les triggers s'executent **implicitement** — il est facile d'oublier leur existence
> - Ils peuvent **degrader les performances** (surtout sur des INSERT/UPDATE massifs)
> - Les triggers imbriques (un trigger qui declenche un autre trigger) sont difficiles a debugger
> - Documentez toujours vos triggers et limitez leur nombre par table

---

## 8. Securite : Injection SQL

L'injection SQL est l'une des vulnerabilites les plus dangereuses et les plus courantes dans les applications web.

### 8.1 Comment fonctionne une injection SQL

```
┌─────────────────────────────────────────────────────────────┐
│                 INJECTION SQL                                │
│                                                             │
│  Formulaire de connexion :                                  │
│  ┌─────────────────────────────┐                            │
│  │ Username: admin             │                            │
│  │ Password: ' OR '1'='1      │                            │
│  └─────────────────────────────┘                            │
│                                                             │
│  Code vulnerable (Python) :                                 │
│  query = "SELECT * FROM users                               │
│           WHERE username='" + username + "'                  │
│           AND password='" + password + "'"                   │
│                                                             │
│  Requete generee :                                          │
│  SELECT * FROM users                                        │
│  WHERE username='admin'                                     │
│  AND password='' OR '1'='1'                                 │
│           ^^^^^^^^^^^^^^^^^                                 │
│           Toujours vrai ! → Acces accorde sans mot de passe │
│                                                             │
│  Pire scenario :                                            │
│  Password: '; DROP TABLE users; --                          │
│  → SELECT * FROM users WHERE ... AND password='';           │
│    DROP TABLE users;                                        │
│    --'                                                      │
│  → LA TABLE USERS EST SUPPRIMEE !                           │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 Exemples de code vulnerable

```python
# PYTHON - CODE VULNERABLE (NE JAMAIS FAIRE CA !)
username = request.form['username']
password = request.form['password']

# Concatenation de chaines : DANGEREUX
query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
cursor.execute(query)
```

```javascript
// JAVASCRIPT (Node.js) - CODE VULNERABLE
const username = req.body.username;
const query = `SELECT * FROM users WHERE username='${username}'`;
// DANGEREUX : injection possible
```

```c
// C - CODE VULNERABLE (similaire a l'usage incorrect de snprintf)
char query[512];
// NE JAMAIS FAIRE CA avec des entrees utilisateur :
snprintf(query, sizeof(query),
    "SELECT * FROM users WHERE username='%s' AND password='%s'",
    username, password);
// L'utilisateur peut injecter des guillemets et du SQL arbitraire
// snprintf protege contre le buffer overflow mais PAS contre l'injection SQL
```

### 8.3 La solution : requetes parametrees (prepared statements)

```python
# PYTHON - CODE SECURISE avec requetes parametrees
username = request.form['username']
password = request.form['password']

# Le SGBD traite les parametres comme des DONNEES, jamais comme du SQL
query = "SELECT * FROM users WHERE username = %s AND password = %s"
cursor.execute(query, (username, password))
# Meme si username contient "' OR '1'='1", il sera traite comme un texte
```

```javascript
// JAVASCRIPT (Node.js) - CODE SECURISE
const query = 'SELECT * FROM users WHERE username = $1 AND password = $2';
const result = await pool.query(query, [username, password]);
```

```java
// JAVA - CODE SECURISE avec PreparedStatement
String query = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement stmt = connection.prepareStatement(query);
stmt.setString(1, username);
stmt.setString(2, password);
ResultSet rs = stmt.executeQuery();
```

```c
// C - Approche securisee avec MySQL C API
MYSQL_STMT *stmt = mysql_stmt_init(conn);
const char *query = "SELECT * FROM users WHERE username = ? AND password = ?";
mysql_stmt_prepare(stmt, query, strlen(query));

MYSQL_BIND bind[2];
memset(bind, 0, sizeof(bind));
bind[0].buffer_type = MYSQL_TYPE_STRING;
bind[0].buffer = username;
bind[0].buffer_length = strlen(username);
bind[1].buffer_type = MYSQL_TYPE_STRING;
bind[1].buffer = password;
bind[1].buffer_length = strlen(password);

mysql_stmt_bind_param(stmt, bind);
mysql_stmt_execute(stmt);
```

> [!warning] Regles d'or contre l'injection SQL
> 1. **TOUJOURS** utiliser des requetes parametrees / prepared statements
> 2. **JAMAIS** concatener des entrees utilisateur dans une requete SQL
> 3. Valider et nettoyer les entrees cote serveur (validation, whitelisting)
> 4. Utiliser un ORM (SQLAlchemy, Sequelize, Hibernate) qui genere des requetes parametrees
> 5. Appliquer le **principe du moindre privilege** : l'utilisateur de la base de l'application ne devrait pas avoir les droits DROP ou ALTER

> [!info] Comparaison avec snprintf en C
> `snprintf` protege contre les **buffer overflows** (debordements de memoire) en limitant la taille d'ecriture. Mais il ne protege **pas** contre l'injection SQL : il formate la chaine telle quelle, guillemets et SQL malveillant inclus. Les requetes parametrees sont au SQL ce que `snprintf` est aux buffers : une protection specifique contre une classe de vulnerabilite specifique.

---

## 9. Gestion des Utilisateurs et Privileges

### 9.1 Creer des utilisateurs

```sql
-- MySQL
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'mot_de_passe_fort';
CREATE USER 'app_user'@'%' IDENTIFIED BY 'mdp_fort';  -- Acces depuis n'importe ou

-- PostgreSQL
CREATE USER app_user WITH PASSWORD 'mot_de_passe_fort';
CREATE ROLE app_readonly WITH LOGIN PASSWORD 'mdp_fort';
```

### 9.2 GRANT : accorder des privileges

```sql
-- Accorder tous les privileges sur une base
GRANT ALL PRIVILEGES ON entreprise.* TO 'admin'@'localhost';

-- Accorder des privileges specifiques
GRANT SELECT, INSERT, UPDATE ON entreprise.employes TO 'app_user'@'localhost';
GRANT SELECT ON entreprise.v_stats_departement TO 'app_user'@'localhost';

-- Privilege d'executer une procedure
GRANT EXECUTE ON PROCEDURE entreprise.sp_liste_employes TO 'app_user'@'localhost';

-- PostgreSQL : privileges sur schema
GRANT USAGE ON SCHEMA public TO app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;

-- Appliquer les changements (MySQL)
FLUSH PRIVILEGES;
```

### 9.3 REVOKE : retirer des privileges

```sql
-- Retirer un privilege specifique
REVOKE INSERT, UPDATE ON entreprise.employes FROM 'app_user'@'localhost';

-- Retirer tous les privileges
REVOKE ALL PRIVILEGES ON entreprise.* FROM 'app_user'@'localhost';

-- Supprimer un utilisateur
DROP USER 'app_user'@'localhost';
```

> [!tip] Principe du moindre privilege
> Chaque utilisateur/application ne devrait avoir que les privileges **strictement necessaires** :
> - L'application web : SELECT, INSERT, UPDATE, DELETE sur les tables metier
> - Le rapport BI : SELECT uniquement, idealement via des vues
> - L'admin : tous les privileges
> - Le backup : SELECT + LOCK TABLES + RELOAD
>
> ```sql
> -- Utilisateur pour une application web (pas de DROP, ALTER, GRANT !)
> CREATE USER 'webapp'@'10.0.0.%' IDENTIFIED BY 'mdp_complexe';
> GRANT SELECT, INSERT, UPDATE, DELETE ON monapp.* TO 'webapp'@'10.0.0.%';
>
> -- Utilisateur en lecture seule pour les rapports
> CREATE USER 'reporting'@'localhost' IDENTIFIED BY 'mdp_reporting';
> GRANT SELECT ON monapp.* TO 'reporting'@'localhost';
> ```

---

## 10. EXPLAIN : Analyser les Requetes

`EXPLAIN` montre le **plan d'execution** d'une requete : comment le SGBD compte la resoudre.

### 10.1 Utilisation de base

```sql
-- MySQL
EXPLAIN SELECT e.prenom, e.nom, d.nom
FROM employes e
JOIN departements d ON e.departement_id = d.id
WHERE d.ville = 'Paris';
```

> [!example] Resultat typique d'EXPLAIN (MySQL)
> ```
> +----+-------+--------+------+---------------+------+---------+------+------+----------+
> | id | type  | table  | type | possible_keys | key  | key_len | ref  | rows | Extra    |
> +----+-------+--------+------+---------------+------+---------+------+------+----------+
> |  1 | SIMPLE| d      | ALL  | PRIMARY       | NULL | NULL    | NULL |    5 | Using w. |
> |  1 | SIMPLE| e      | ref  | idx_dept      | idx  | 5       | d.id |    2 | NULL     |
> +----+-------+--------+------+---------------+------+---------+------+------+----------+
> ```

### 10.2 Elements cles a observer

```
┌──────────────┬──────────────────────────────────────────────┐
│ Colonne      │ Signification                                │
├──────────────┼──────────────────────────────────────────────┤
│ type         │ Type d'acces a la table :                    │
│              │ system > const > eq_ref > ref > range >      │
│              │ index > ALL (du meilleur au pire)            │
│              │ ALL = full table scan (a eviter)             │
├──────────────┼──────────────────────────────────────────────┤
│ key          │ L'index effectivement utilise (NULL = aucun) │
├──────────────┼──────────────────────────────────────────────┤
│ rows         │ Estimation du nombre de lignes examinees     │
├──────────────┼──────────────────────────────────────────────┤
│ Extra        │ Using index : index couvrant (tres bien)     │
│              │ Using where : filtrage apres lecture          │
│              │ Using filesort : tri en memoire (couteux)    │
│              │ Using temporary : table temporaire (couteux) │
└──────────────┴──────────────────────────────────────────────┘
```

```sql
-- PostgreSQL : EXPLAIN ANALYZE (execute reellement la requete)
EXPLAIN ANALYZE
SELECT e.prenom, e.nom, s.montant
FROM employes e
JOIN salaires s ON s.employe_id = e.id
WHERE s.date_fin IS NULL
ORDER BY s.montant DESC;
-- Affiche le temps reel d'execution et le nombre exact de lignes
```

---

## 11. Optimisation des Performances

### 11.1 Regles essentielles

```
┌──────────────────────────────────────────────────────────────┐
│           BONNES PRATIQUES D'OPTIMISATION                    │
│                                                              │
│  1. Eviter SELECT *                                          │
│     → Selectionner uniquement les colonnes necessaires       │
│     → Permet au SGBD d'utiliser des index couvrants          │
│                                                              │
│  2. Indexer les colonnes utilisees dans WHERE et JOIN         │
│     → Les cles etrangeres doivent toujours etre indexees     │
│     → Les colonnes de filtre frequentes aussi                │
│                                                              │
│  3. Eviter les fonctions sur les colonnes indexees            │
│     → WHERE YEAR(date) = 2025     -- N'utilise PAS l'index   │
│     → WHERE date >= '2025-01-01'  -- Utilise l'index         │
│       AND date < '2026-01-01'                                │
│                                                              │
│  4. Utiliser LIMIT quand possible                            │
│     → Surtout pour les requetes de "preview" ou pagination   │
│                                                              │
│  5. Preferer EXISTS a IN pour les grandes sous-requetes      │
│     → EXISTS s'arrete au premier match                       │
│                                                              │
│  6. Utiliser EXPLAIN pour verifier le plan d'execution       │
│     → Avant chaque requete critique en production            │
└──────────────────────────────────────────────────────────────┘
```

### 11.2 Le probleme N+1

Le probleme N+1 est l'un des pieges de performance les plus courants, surtout avec les ORM.

```python
# PROBLEME N+1 : 1 requete + N requetes supplementaires
# Requete 1 : recuperer tous les posts
posts = db.execute("SELECT * FROM posts")  # 1 requete

for post in posts:
    # Requete 2..N+1 : pour CHAQUE post, recuperer l'auteur
    author = db.execute(
        "SELECT * FROM users WHERE id = %s", post.auteur_id
    )  # N requetes supplementaires !

# Si 100 posts → 101 requetes au total !

# SOLUTION : une seule requete avec JOIN
results = db.execute("""
    SELECT p.*, u.username, u.email
    FROM posts p
    JOIN users u ON p.auteur_id = u.id
""")  # 1 seule requete, meme pour 10 000 posts
```

> [!warning] Detecter le probleme N+1
> Symptomes :
> - L'application est lente malgre des tables de taille moderee
> - Le log SQL montre des centaines de requetes similaires qui varient seulement par l'ID
> - Les performances se degradent lineairement avec le nombre d'enregistrements
>
> Solutions :
> - Utiliser des JOINs explicites
> - Activer le "eager loading" dans votre ORM (ex: `joinedload` en SQLAlchemy)
> - Utiliser `IN` pour charger en batch : `WHERE id IN (1, 2, 3, ...)`

### 11.3 Autres techniques d'optimisation

```sql
-- Eviter les sous-requetes correlees quand un JOIN suffit
-- LENT (sous-requete correlee, executee N fois) :
SELECT e.prenom, e.nom,
    (SELECT d.nom FROM departements d WHERE d.id = e.departement_id)
FROM employes e;

-- RAPIDE (JOIN, execute une seule fois) :
SELECT e.prenom, e.nom, d.nom
FROM employes e
LEFT JOIN departements d ON e.departement_id = d.id;

-- Utiliser des index couvrants (covering index)
-- L'index contient toutes les colonnes necessaires : pas besoin d'acceder a la table
CREATE INDEX idx_covering ON employes(departement_id, nom, prenom);
-- La requete suivante n'accede QU'a l'index :
SELECT nom, prenom FROM employes WHERE departement_id = 1;

-- Partitionnement pour les tres grandes tables
CREATE TABLE logs (
    id          BIGINT AUTO_INCREMENT,
    message     TEXT,
    date_log    DATE,
    PRIMARY KEY (id, date_log)
)
PARTITION BY RANGE (YEAR(date_log)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

---

## Carte Mentale ASCII

```
                ┌───────────────────────────────────┐
                │   SQL AVANCE ET ADMINISTRATION     │
                └───────────────────┬───────────────┘
                                    │
     ┌──────────────┬───────────────┼───────────────┬──────────────┐
     │              │               │               │              │
┌────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐  ┌────▼─────┐
│Transact. │  │   Vues    │  │Procedures │  │ Securite  │  │Optimis.  │
│          │  │           │  │& Fonctions│  │           │  │          │
└────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └────┬─────┘
     │              │              │              │              │
  BEGIN          CREATE VIEW    PROCEDURE      Injection      EXPLAIN
  COMMIT         WITH CHECK     IN/OUT/INOUT   SQL            Index
  ROLLBACK       OPTION         FUNCTION       Prepared       SELECT *
  SAVEPOINT      Securite       RETURNS        Statements     N+1
     │           Abstraction    DETERMINISTIC  CREATE USER    Covering
  ACID                            │            GRANT          Index
  Atomicite                    ┌──▼──┐         REVOKE
  Coherence                    │Trigg│
  Isolation                    │ ers │
  Durabilite                   └──┬──┘
     │                            │
  Niveaux                    BEFORE/AFTER
  d'isolation                INSERT/UPDATE
  READ UNCOMMITTED           DELETE
  READ COMMITTED             NEW / OLD
  REPEATABLE READ            Audit log
  SERIALIZABLE               Validation
```

---

## Exercices

### Exercice 1 : Transactions
Ecrivez une transaction complete pour un systeme de e-commerce :
1. Creer une commande dans la table `commandes`
2. Ajouter 3 lignes dans `lignes_commande`
3. Diminuer le stock de chaque produit dans `produits`
4. Si le stock d'un produit passe en dessous de 0, annuler toute la transaction
Utilisez des SAVEPOINT pour gerer les erreurs partielles.

### Exercice 2 : Vues et securite
Sur la base du blog (cours 03) :
1. Creez une vue `v_posts_publies` qui montre les posts publies avec leur auteur et le nombre de commentaires
2. Creez une vue `v_profil_public` qui montre les infos non-sensibles des utilisateurs (pas le password_hash)
3. Creez un utilisateur `blog_reader` avec acces en lecture seule sur ces deux vues uniquement

### Exercice 3 : Procedures et triggers
1. Ecrivez une procedure `sp_publier_post(p_post_id INT)` qui change le statut d'un post en 'publie' et met a jour la date_publication
2. Ecrivez un trigger qui empeche de supprimer un utilisateur ayant des posts publies
3. Ecrivez un trigger d'audit qui enregistre chaque modification de salaire dans une table `audit_salaires`

### Exercice 4 : Optimisation
Analysez et optimisez cette requete :
```sql
SELECT *
FROM posts p
WHERE YEAR(p.date_publication) = 2025
  AND p.auteur_id IN (
    SELECT u.id FROM users u WHERE u.actif = TRUE
  )
ORDER BY p.date_publication DESC;
```
1. Identifiez les 3 problemes de performance
2. Reecrivez la requete de maniere optimisee
3. Quels index creeriez-vous pour cette requete ?

---

## Liens

- [[01 - Introduction au SQL]] - Les bases du SQL
- [[02 - Requetes Avancees SQL]] - JOINs, sous-requetes, fonctions de fenetrage
- [[03 - Conception de Bases de Donnees]] - Normalisation, cles, indexes
- [[02 - Securite Web OWASP]] - Injections SQL et autres vulnerabilites web
- [[10 - Python et Bases de Donnees]] - Utiliser SQL depuis Python
