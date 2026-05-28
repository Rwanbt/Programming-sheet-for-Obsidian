# 05 - Oracle et PL/SQL

> [!info] Objectifs
> Maîtriser les spécificités Oracle par rapport aux autres SGBD, programmer en PL/SQL (procédures, fonctions, triggers, packages), optimiser les requêtes avec les outils Oracle et gérer les erreurs professionnellement.

## 1. Oracle vs PostgreSQL vs MySQL

### Différences clés

| Fonctionnalité | Oracle | PostgreSQL | MySQL |
|---------------|--------|-----------|-------|
| Licence | Commercial ($$$) | Open Source | Open Source / Commercial |
| Langage procédural | PL/SQL | PL/pgSQL | Routines SQL |
| NULL dans index | Non indexé | Indexé | Indexé |
| Séquences | `CREATE SEQUENCE` | `SERIAL` / `CREATE SEQUENCE` | `AUTO_INCREMENT` |
| Pagination | `ROWNUM`, `FETCH FIRST` | `LIMIT`/`OFFSET`, `FETCH FIRST` | `LIMIT`/`OFFSET` |
| Outer join (old) | `(+)` syntaxe propriétaire | ANSI JOIN uniquement | ANSI JOIN uniquement |
| Hints SQL | Oui (`/*+ INDEX(...) */`) | Limité | Limité |
| Hiérarchique | `CONNECT BY` | Recursive CTEs | Recursive CTEs |
| Partitionnement | Avancé (native) | Déclaratif | Basique |
| Flashback | Oui (native) | Non | Non |
| RAC | Oracle Real Application Clusters | Non | Non |

### La table DUAL

Oracle dispose d'une table fictive à une ligne/colonne utilisée pour les expressions scalaires :

```sql
-- Calculer sans table source
SELECT 2 + 2 FROM DUAL;
SELECT SYSDATE FROM DUAL;
SELECT USER FROM DUAL;
SELECT SYS_GUID() FROM DUAL;  -- Générer un UUID

-- En PostgreSQL/MySQL : SELECT 2 + 2; (pas besoin de DUAL)
```

### ROWNUM vs ROW_NUMBER()

```sql
-- ❌ ROWNUM : dangereux avec ORDER BY (numérotation AVANT le tri)
SELECT * FROM employees WHERE ROWNUM <= 10;  -- OK, premiers 10
SELECT * FROM (SELECT * FROM employees ORDER BY salary DESC) WHERE ROWNUM <= 10;  -- Correct

-- ✅ ROW_NUMBER() (SQL:2003, préféré)
SELECT *
FROM (
    SELECT e.*, ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn
    FROM employees e
)
WHERE rn BETWEEN 1 AND 10;

-- FETCH FIRST (Oracle 12c+, syntaxe standard)
SELECT * FROM employees ORDER BY salary DESC FETCH FIRST 10 ROWS ONLY;
SELECT * FROM employees ORDER BY salary DESC OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;
```

### MERGE Statement (Oracle)

```sql
-- MERGE = INSERT + UPDATE + DELETE conditionnel en une seule instruction
MERGE INTO target_table t
USING source_table s
ON (t.employee_id = s.employee_id)
WHEN MATCHED THEN
    UPDATE SET
        t.salary = s.salary,
        t.department_id = s.department_id
    WHERE t.salary <> s.salary        -- Condition optionnelle sur la branche MATCHED
    DELETE WHERE s.active = 'N'       -- Supprimer les inactifs
WHEN NOT MATCHED THEN
    INSERT (employee_id, first_name, last_name, salary, department_id)
    VALUES (s.employee_id, s.first_name, s.last_name, s.salary, s.department_id);
```

### CONNECT BY — Données hiérarchiques

```sql
-- Arbre organisationnel : manager_id référence employee_id du supérieur
SELECT
    LEVEL,                              -- Profondeur dans la hiérarchie (1 = racine)
    LPAD(' ', 2*(LEVEL-1)) || first_name || ' ' || last_name AS name,
    employee_id,
    manager_id,
    SYS_CONNECT_BY_PATH(first_name, '/') AS path  -- Chemin depuis la racine
FROM employees
START WITH manager_id IS NULL           -- Racine : le PDG (pas de manager)
CONNECT BY PRIOR employee_id = manager_id  -- Relier parent → enfant
ORDER SIBLINGS BY last_name;            -- Trier les siblings alphabétiquement

-- CONNECT_BY_ROOT : valeur de la racine
-- CONNECT_BY_ISLEAF : 1 si le nœud est une feuille (pas d'enfants)
SELECT
    first_name,
    CONNECT_BY_ROOT first_name AS root_manager,
    CONNECT_BY_ISLEAF AS is_leaf
FROM employees
START WITH manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id;
```

---

## 2. PL/SQL — Langage procédural Oracle

### Structure de base

```sql
-- Bloc PL/SQL anonyme
DECLARE
    -- Section de déclaration des variables
    v_employee_name  VARCHAR2(100);
    v_salary         NUMBER(10, 2);
    v_hire_date      DATE;
    v_count          INTEGER := 0;  -- Initialisation
    c_bonus_rate     CONSTANT NUMBER := 0.15;

BEGIN
    -- Section d'exécution (obligatoire)
    SELECT first_name || ' ' || last_name, salary, hire_date
    INTO v_employee_name, v_salary, v_hire_date
    FROM employees
    WHERE employee_id = 100;

    v_salary := v_salary * (1 + c_bonus_rate);

    DBMS_OUTPUT.PUT_LINE('Employé: ' || v_employee_name);
    DBMS_OUTPUT.PUT_LINE('Nouveau salaire: ' || TO_CHAR(v_salary, '999,999.99'));

EXCEPTION
    -- Section de gestion des erreurs (optionnelle)
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Employé non trouvé.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Erreur: ' || SQLERRM);
        RAISE;  -- Re-lever l'exception
END;
/   -- Le slash exécute le bloc en SQL*Plus
```

### Types de données PL/SQL

```sql
DECLARE
    -- Types scalaires
    v_name     VARCHAR2(100);
    v_salary   NUMBER(12, 2);
    v_age      INTEGER;
    v_active   BOOLEAN := TRUE;
    v_date     DATE := SYSDATE;
    v_ts       TIMESTAMP WITH TIME ZONE;

    -- %TYPE : même type qu'une colonne (recommandé, s'adapte aux changements de schéma)
    v_emp_name employees.first_name%TYPE;
    v_emp_sal  employees.salary%TYPE;

    -- %ROWTYPE : enregistrement avec la structure d'une ligne de table
    v_emp_row  employees%ROWTYPE;

    -- Record personnalisé
    TYPE t_address IS RECORD (
        street    VARCHAR2(100),
        city      VARCHAR2(50),
        zip_code  VARCHAR2(10)
    );
    v_addr t_address;

    -- Collections
    TYPE t_salary_list IS TABLE OF NUMBER INDEX BY PLS_INTEGER;  -- Associative array
    v_salaries t_salary_list;

BEGIN
    -- Utilisation de %ROWTYPE
    SELECT * INTO v_emp_row FROM employees WHERE employee_id = 100;
    DBMS_OUTPUT.PUT_LINE(v_emp_row.first_name || ' earns ' || v_emp_row.salary);

    -- Utilisation du record
    v_addr.street := '1 Main Street';
    v_addr.city := 'Paris';

    -- Utilisation de la collection
    v_salaries(1) := 50000;
    v_salaries(2) := 75000;
END;
/
```

### Structures de contrôle

```sql
DECLARE
    v_salary   NUMBER := 80000;
    v_bonus    NUMBER;
    v_counter  INTEGER;
BEGIN
    -- IF / ELSIF / ELSE
    IF v_salary >= 100000 THEN
        v_bonus := v_salary * 0.20;
    ELSIF v_salary >= 50000 THEN
        v_bonus := v_salary * 0.15;
    ELSE
        v_bonus := v_salary * 0.10;
    END IF;

    -- CASE
    v_bonus := CASE
        WHEN v_salary >= 100000 THEN v_salary * 0.20
        WHEN v_salary >= 50000  THEN v_salary * 0.15
        ELSE                         v_salary * 0.10
    END;

    -- LOOP simple (avec EXIT WHEN)
    v_counter := 1;
    LOOP
        DBMS_OUTPUT.PUT_LINE('Iteration: ' || v_counter);
        v_counter := v_counter + 1;
        EXIT WHEN v_counter > 5;
    END LOOP;

    -- WHILE LOOP
    v_counter := 1;
    WHILE v_counter <= 5 LOOP
        DBMS_OUTPUT.PUT_LINE('While: ' || v_counter);
        v_counter := v_counter + 1;
    END WHILE;

    -- FOR LOOP (bornes incluses)
    FOR i IN 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE('For: ' || i);
    END LOOP;

    -- FOR LOOP en sens inverse
    FOR i IN REVERSE 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE('Reverse: ' || i);
    END LOOP;
END;
/
```

### Curseurs

```sql
DECLARE
    -- Curseur explicite
    CURSOR c_high_earners IS
        SELECT employee_id, first_name, last_name, salary
        FROM employees
        WHERE salary > 10000
        ORDER BY salary DESC;

    -- Variable de curseur
    v_emp c_high_earners%ROWTYPE;

BEGIN
    -- Méthode 1 : OPEN / FETCH / CLOSE (contrôle total)
    OPEN c_high_earners;
    LOOP
        FETCH c_high_earners INTO v_emp;
        EXIT WHEN c_high_earners%NOTFOUND;  -- Plus de lignes
        DBMS_OUTPUT.PUT_LINE(v_emp.first_name || ': $' || v_emp.salary);
    END LOOP;
    DBMS_OUTPUT.PUT_LINE('Lignes traitées: ' || c_high_earners%ROWCOUNT);
    CLOSE c_high_earners;

    -- Méthode 2 : Cursor FOR loop (recommandée, plus simple, ferme automatiquement)
    FOR rec IN c_high_earners LOOP
        DBMS_OUTPUT.PUT_LINE(rec.first_name || ': $' || rec.salary);
    END LOOP;

    -- Méthode 3 : Curseur inline dans le FOR loop
    FOR rec IN (SELECT first_name, salary FROM employees WHERE salary > 10000) LOOP
        DBMS_OUTPUT.PUT_LINE(rec.first_name || ': $' || rec.salary);
    END LOOP;

    -- Curseur avec paramètres
    DECLARE
        CURSOR c_dept_emp(p_dept_id NUMBER) IS
            SELECT * FROM employees WHERE department_id = p_dept_id;
    BEGIN
        FOR rec IN c_dept_emp(10) LOOP
            DBMS_OUTPUT.PUT_LINE(rec.first_name);
        END LOOP;
    END;
END;
/
```

---

## 3. Procédures stockées

```sql
-- Création
CREATE OR REPLACE PROCEDURE give_raise(
    p_employee_id IN  employees.employee_id%TYPE,
    p_raise_pct   IN  NUMBER,
    p_new_salary  OUT employees.salary%TYPE,
    p_status      OUT VARCHAR2
) AS
    v_old_salary employees.salary%TYPE;
    v_max_salary jobs.max_salary%TYPE;
BEGIN
    -- Récupérer le salaire actuel et le maximum du poste
    SELECT e.salary, j.max_salary
    INTO v_old_salary, v_max_salary
    FROM employees e
    JOIN jobs j ON e.job_id = j.job_id
    WHERE e.employee_id = p_employee_id;

    p_new_salary := v_old_salary * (1 + p_raise_pct / 100);

    IF p_new_salary > v_max_salary THEN
        p_new_salary := v_max_salary;
        p_status := 'CAPPED';
    ELSE
        p_status := 'OK';
    END IF;

    UPDATE employees
    SET salary = p_new_salary,
        last_modified = SYSDATE
    WHERE employee_id = p_employee_id;

    COMMIT;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        p_status := 'ERROR: Employee not found';
        ROLLBACK;
    WHEN OTHERS THEN
        p_status := 'ERROR: ' || SQLERRM;
        ROLLBACK;
        RAISE;
END give_raise;
/

-- Appel depuis SQL*Plus
DECLARE
    v_new_sal employees.salary%TYPE;
    v_status  VARCHAR2(100);
BEGIN
    give_raise(
        p_employee_id => 101,
        p_raise_pct   => 10,
        p_new_salary  => v_new_sal,
        p_status      => v_status
    );
    DBMS_OUTPUT.PUT_LINE('Nouveau salaire: ' || v_new_sal || ' Status: ' || v_status);
END;
/
```

```python
# Appel depuis Python avec oracledb (successeur de cx_Oracle)
import oracledb

connection = oracledb.connect(
    user="admin",
    password="secret",
    dsn="hostname:1521/ORCL"
)

cursor = connection.cursor()

# Appeler la procédure
new_salary = cursor.var(oracledb.NUMBER)
status = cursor.var(oracledb.STRING)

cursor.callproc("give_raise", [101, 10, new_salary, status])

print(f"Nouveau salaire: {new_salary.getvalue()}")
print(f"Status: {status.getvalue()}")

connection.commit()
cursor.close()
connection.close()
```

---

## 4. Fonctions

```sql
-- Fonction qui retourne une valeur utilisable dans des requêtes SQL
CREATE OR REPLACE FUNCTION get_employee_age(
    p_employee_id IN employees.employee_id%TYPE
) RETURN NUMBER
AS
    v_hire_date employees.hire_date%TYPE;
    v_years     NUMBER;
BEGIN
    SELECT hire_date INTO v_hire_date
    FROM employees
    WHERE employee_id = p_employee_id;

    v_years := TRUNC(MONTHS_BETWEEN(SYSDATE, v_hire_date) / 12);
    RETURN v_years;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN NULL;
END get_employee_age;
/

-- Utilisation dans une requête SQL
SELECT
    employee_id,
    first_name,
    get_employee_age(employee_id) AS years_of_service,
    ROUND(salary / 12, 2) AS monthly_salary
FROM employees
WHERE get_employee_age(employee_id) >= 5
ORDER BY years_of_service DESC;
```

---

## 5. Triggers

```sql
-- Trigger BEFORE INSERT : générer une PK automatique
CREATE OR REPLACE TRIGGER trg_employees_bi
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    IF :NEW.employee_id IS NULL THEN
        :NEW.employee_id := seq_employees.NEXTVAL;
    END IF;
    :NEW.created_at := SYSDATE;
    :NEW.created_by := USER;
END;
/

-- Trigger AFTER INSERT/UPDATE/DELETE : audit log
CREATE OR REPLACE TRIGGER trg_employees_audit
AFTER INSERT OR UPDATE OR DELETE ON employees
FOR EACH ROW
BEGIN
    IF INSERTING THEN
        INSERT INTO employees_audit (action, employee_id, new_salary, changed_by, changed_at)
        VALUES ('INSERT', :NEW.employee_id, :NEW.salary, USER, SYSTIMESTAMP);

    ELSIF UPDATING THEN
        INSERT INTO employees_audit (action, employee_id, old_salary, new_salary, changed_by, changed_at)
        VALUES ('UPDATE', :OLD.employee_id, :OLD.salary, :NEW.salary, USER, SYSTIMESTAMP);

    ELSIF DELETING THEN
        INSERT INTO employees_audit (action, employee_id, old_salary, changed_by, changed_at)
        VALUES ('DELETE', :OLD.employee_id, :OLD.salary, USER, SYSTIMESTAMP);
    END IF;
END;
/

-- Trigger de niveau STATEMENT (pas FOR EACH ROW) : une seule exécution par DML
CREATE OR REPLACE TRIGGER trg_protect_weekends
BEFORE INSERT OR UPDATE OR DELETE ON salary_changes
BEGIN
    IF TO_CHAR(SYSDATE, 'DY', 'NLS_DATE_LANGUAGE=ENGLISH') IN ('SAT', 'SUN') THEN
        RAISE_APPLICATION_ERROR(-20001, 'Salary changes not allowed on weekends');
    END IF;
END;
/

-- INSTEAD OF trigger sur une vue (pour rendre une vue complexe modifiable)
CREATE OR REPLACE TRIGGER trg_employee_dept_view
INSTEAD OF INSERT ON employee_dept_view   -- Vue joignant employees + departments
FOR EACH ROW
BEGIN
    INSERT INTO employees (employee_id, first_name, last_name, salary, department_id)
    VALUES (:NEW.employee_id, :NEW.first_name, :NEW.last_name, :NEW.salary, :NEW.department_id);
END;
/
```

> [!warning] :NEW et :OLD
> `:NEW` : nouvelles valeurs (disponible en INSERT et UPDATE)
> `:OLD` : anciennes valeurs (disponible en UPDATE et DELETE)
> En INSERT : `:OLD` est NULL. En DELETE : `:NEW` est NULL.

---

## 6. Packages

```sql
-- Package SPEC (interface publique — comme un header C)
CREATE OR REPLACE PACKAGE pkg_hr_utils AS

    -- Constante publique
    c_max_raise_pct CONSTANT NUMBER := 30;

    -- Déclarations des procédures et fonctions publiques
    PROCEDURE give_raise(
        p_employee_id IN  NUMBER,
        p_raise_pct   IN  NUMBER,
        p_new_salary  OUT NUMBER
    );

    FUNCTION get_department_budget(p_dept_id IN NUMBER) RETURN NUMBER;

    -- Surcharge (overloading) : même nom, paramètres différents
    FUNCTION format_salary(p_salary NUMBER) RETURN VARCHAR2;
    FUNCTION format_salary(p_salary NUMBER, p_currency VARCHAR2) RETURN VARCHAR2;

END pkg_hr_utils;
/

-- Package BODY (implémentation)
CREATE OR REPLACE PACKAGE BODY pkg_hr_utils AS

    -- Variable de package (persistante pour la session)
    g_call_count NUMBER := 0;

    -- Procédure privée (non déclarée dans la spec)
    PROCEDURE log_action(p_action VARCHAR2) AS
    BEGIN
        INSERT INTO action_log VALUES (SYSDATE, USER, p_action);
    END log_action;

    -- Implémentation des membres publics
    PROCEDURE give_raise(
        p_employee_id IN  NUMBER,
        p_raise_pct   IN  NUMBER,
        p_new_salary  OUT NUMBER
    ) AS
        v_salary NUMBER;
    BEGIN
        g_call_count := g_call_count + 1;

        IF p_raise_pct > c_max_raise_pct THEN
            RAISE_APPLICATION_ERROR(-20010, 'Raise exceeds maximum ' || c_max_raise_pct || '%');
        END IF;

        SELECT salary INTO v_salary FROM employees WHERE employee_id = p_employee_id;
        p_new_salary := v_salary * (1 + p_raise_pct / 100);

        UPDATE employees SET salary = p_new_salary WHERE employee_id = p_employee_id;
        log_action('RAISE employee_id=' || p_employee_id);
    END give_raise;

    FUNCTION get_department_budget(p_dept_id IN NUMBER) RETURN NUMBER AS
        v_budget NUMBER;
    BEGIN
        SELECT SUM(salary) INTO v_budget FROM employees WHERE department_id = p_dept_id;
        RETURN NVL(v_budget, 0);
    END get_department_budget;

    FUNCTION format_salary(p_salary NUMBER) RETURN VARCHAR2 AS
    BEGIN
        RETURN TO_CHAR(p_salary, 'FM$999,999,990.00');
    END format_salary;

    FUNCTION format_salary(p_salary NUMBER, p_currency VARCHAR2) RETURN VARCHAR2 AS
    BEGIN
        RETURN TO_CHAR(p_salary, 'FM999,999,990.00') || ' ' || p_currency;
    END format_salary;

END pkg_hr_utils;
/

-- Utilisation
BEGIN
    pkg_hr_utils.give_raise(101, 10, :new_sal);
    DBMS_OUTPUT.PUT_LINE(pkg_hr_utils.format_salary(75000, 'EUR'));
END;
/
```

**DBMS_OUTPUT** : paquet système pour déboguer (équivalent de `print()`)
```sql
SET SERVEROUTPUT ON SIZE UNLIMITED  -- Activer dans SQL*Plus
DBMS_OUTPUT.PUT_LINE('Debug: ' || v_variable);
DBMS_OUTPUT.PUT(v_part1);           -- Sans saut de ligne
DBMS_OUTPUT.NEW_LINE;               -- Saut de ligne
```

---

## 7. Gestion des erreurs

### Exceptions prédéfinies

```sql
BEGIN
    SELECT salary INTO v_salary FROM employees WHERE employee_id = p_id;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        -- SELECT INTO sans résultat
        DBMS_OUTPUT.PUT_LINE('No employee found');

    WHEN TOO_MANY_ROWS THEN
        -- SELECT INTO avec plusieurs lignes
        DBMS_OUTPUT.PUT_LINE('Multiple rows returned');

    WHEN DUP_VAL_ON_INDEX THEN
        -- Violation de contrainte UNIQUE
        DBMS_OUTPUT.PUT_LINE('Duplicate value');

    WHEN VALUE_ERROR THEN
        -- Erreur de conversion ou dépassement de capacité
        DBMS_OUTPUT.PUT_LINE('Value error');

    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Division by zero');

    WHEN OTHERS THEN
        -- Attrape tout le reste
        DBMS_OUTPUT.PUT_LINE('Unexpected error: ' || SQLCODE || ' - ' || SQLERRM);
        -- SQLCODE : code d'erreur Oracle (négatif pour les erreurs Oracle)
        -- SQLERRM : message d'erreur Oracle
        RAISE;  -- Re-propager l'exception (ne pas avaler silencieusement)
END;
```

### Exceptions utilisateur

```sql
CREATE OR REPLACE PROCEDURE check_salary(p_salary NUMBER) AS
    e_salary_too_high EXCEPTION;               -- Déclaration d'exception custom
    PRAGMA EXCEPTION_INIT(e_salary_too_high, -20001);  -- Lier à un code ORA-

    e_negative_salary EXCEPTION;               -- Exception non liée à un code ORA-
BEGIN
    IF p_salary < 0 THEN
        RAISE e_negative_salary;               -- Lever l'exception
    END IF;

    IF p_salary > 1000000 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Salary ' || p_salary || ' exceeds maximum');
        -- RAISE_APPLICATION_ERROR : range -20000 à -20999 réservé pour le code applicatif
    END IF;

EXCEPTION
    WHEN e_negative_salary THEN
        DBMS_OUTPUT.PUT_LINE('Erreur : salaire négatif interdit');
    WHEN e_salary_too_high THEN
        DBMS_OUTPUT.PUT_LINE('Erreur : ' || SQLERRM);
END;
/
```

---

## 8. Performance Oracle

### EXPLAIN PLAN et DBMS_XPLAN

```sql
-- Générer le plan d'exécution
EXPLAIN PLAN FOR
SELECT e.first_name, e.last_name, d.department_name, e.salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 10000
ORDER BY e.salary DESC;

-- Afficher le plan
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY(FORMAT => 'ALL'));

-- Plan d'une requête déjà en mémoire (plus précis, avec statistiques réelles)
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR(format => 'ALLSTATS LAST'));
```

### Types d'index Oracle

```sql
-- Index B-tree (défaut, pour les colonnes à haute cardinalité)
CREATE INDEX idx_emp_salary ON employees(salary);
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

-- Index composé (ordre des colonnes = ordre des filtres fréquents)
CREATE INDEX idx_emp_dept_sal ON employees(department_id, salary);

-- Index Bitmap (pour les colonnes à faible cardinalité : statut, sexe, département)
CREATE BITMAP INDEX idx_emp_status ON employees(status);
-- ⚠ Bitmap : excellent en lecture, catastrophique pour les écritures concurrentes (DML)

-- Index basé sur une fonction
CREATE INDEX idx_emp_name_upper ON employees(UPPER(last_name));
-- Permet d'utiliser l'index sur : WHERE UPPER(last_name) = 'SMITH'

-- Index de type : INVISIBLE (ne sera pas utilisé par l'optimizer, pour tester)
ALTER INDEX idx_emp_salary INVISIBLE;
ALTER INDEX idx_emp_salary VISIBLE;
```

### Hints SQL

```sql
-- Forcer l'utilisation d'un index
SELECT /*+ INDEX(e idx_emp_salary) */
    e.first_name, e.salary
FROM employees e
WHERE e.salary > 10000;

-- Forcer un Full Table Scan (bypasser les index)
SELECT /*+ FULL(e) */
    e.first_name
FROM employees e
WHERE e.salary > 10000;

-- Hints de join
SELECT /*+ USE_NL(e d) */      -- Nested Loops (pour petits volumes ou avec index)
    e.first_name, d.department_name
FROM employees e, departments d
WHERE e.department_id = d.department_id;

SELECT /*+ USE_HASH(e d) */    -- Hash Join (pour gros volumes)
    e.first_name, d.department_name
FROM employees e, departments d
WHERE e.department_id = d.department_id;

-- Hint parallèle
SELECT /*+ PARALLEL(e, 4) */ COUNT(*) FROM employees e;

-- Forcer l'ordre des tables dans la jointure
SELECT /*+ ORDERED */ ...
FROM employees e, departments d, locations l
WHERE e.department_id = d.department_id ...;
```

### Partitionnement

```sql
-- Partitionnement RANGE (par plage de valeurs)
CREATE TABLE sales (
    sale_id      NUMBER,
    sale_date    DATE,
    amount       NUMBER,
    customer_id  NUMBER
)
PARTITION BY RANGE (sale_date) (
    PARTITION p_2022 VALUES LESS THAN (DATE '2023-01-01'),
    PARTITION p_2023 VALUES LESS THAN (DATE '2024-01-01'),
    PARTITION p_2024 VALUES LESS THAN (DATE '2025-01-01'),
    PARTITION p_future VALUES LESS THAN (MAXVALUE)
);

-- Partitionnement HASH (distribution uniforme)
CREATE TABLE employees_hash (
    employee_id NUMBER,
    first_name  VARCHAR2(50)
)
PARTITION BY HASH (employee_id) PARTITIONS 8;

-- Partitionnement LIST (par valeurs discrètes)
CREATE TABLE sales_regional (
    sale_id NUMBER,
    region  VARCHAR2(20)
)
PARTITION BY LIST (region) (
    PARTITION p_north VALUES ('NORTH', 'NORTHWEST', 'NORTHEAST'),
    PARTITION p_south VALUES ('SOUTH', 'SOUTHWEST', 'SOUTHEAST'),
    PARTITION p_other VALUES (DEFAULT)
);

-- Composite (RANGE-HASH, RANGE-LIST...)
CREATE TABLE sales_composite (
    sale_id   NUMBER,
    sale_date DATE,
    region    VARCHAR2(20)
)
PARTITION BY RANGE (sale_date)
SUBPARTITION BY HASH (region) SUBPARTITIONS 4 (
    PARTITION p_2024 VALUES LESS THAN (DATE '2025-01-01'),
    PARTITION p_2025 VALUES LESS THAN (DATE '2026-01-01')
);
```

**Partition Pruning** : l'optimizer Oracle n'accède qu'aux partitions pertinentes selon les filtres de la requête.

---

## 9. SQL Oracle avancé

### Window Functions

```sql
-- Similaires à PostgreSQL
SELECT
    employee_id,
    first_name,
    department_id,
    salary,
    -- Rang dans le département
    RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dept_rank,
    DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dept_dense_rank,
    -- Comparaison avec voisins
    LAG(salary, 1, 0) OVER (PARTITION BY department_id ORDER BY hire_date) AS prev_salary,
    LEAD(salary) OVER (PARTITION BY department_id ORDER BY hire_date) AS next_salary,
    -- Agrégations glissantes
    AVG(salary) OVER (PARTITION BY department_id
                      ORDER BY hire_date
                      ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_avg,
    SUM(salary) OVER (PARTITION BY department_id) AS dept_total
FROM employees;
```

### PIVOT / UNPIVOT

```sql
-- Transformer des lignes en colonnes (PIVOT)
SELECT *
FROM (
    SELECT department_id, job_id, salary
    FROM employees
)
PIVOT (
    AVG(salary) AS avg_sal
    FOR job_id IN (
        'IT_PROG' AS programmer,
        'SA_MAN' AS sales_manager,
        'AD_ASST' AS admin_asst
    )
);

-- UNPIVOT : transformer des colonnes en lignes
SELECT employee_id, quarter, sales_amount
FROM quarterly_sales
UNPIVOT (
    sales_amount FOR quarter IN (q1 AS 'Q1', q2 AS 'Q2', q3 AS 'Q3', q4 AS 'Q4')
);
```

### LISTAGG — Agrégation en chaîne

```sql
-- Lister les employés d'un département sur une ligne
SELECT
    department_id,
    LISTAGG(last_name, ', ') WITHIN GROUP (ORDER BY last_name) AS employee_list
FROM employees
GROUP BY department_id;

-- Oracle 19c+ : LISTAGG avec ON OVERFLOW TRUNCATE (évite ORA-01489)
SELECT
    department_id,
    LISTAGG(last_name, ', ' ON OVERFLOW TRUNCATE '...' WITH COUNT)
        WITHIN GROUP (ORDER BY last_name) AS employee_list
FROM employees
GROUP BY department_id;
```

### Flashback Query

```sql
-- Voir les données d'il y a 1 heure (undo data doit être disponible)
SELECT * FROM employees AS OF TIMESTAMP (SYSDATE - INTERVAL '1' HOUR);

-- Voir les données à un SCN (System Change Number) précis
SELECT * FROM employees AS OF SCN 12345678;

-- Récupérer des lignes supprimées accidentellement
INSERT INTO employees
SELECT * FROM employees AS OF TIMESTAMP (SYSDATE - INTERVAL '30' MINUTE)
WHERE employee_id = 999;

-- Flashback Table (restaurer toute la table à un moment passé)
FLASHBACK TABLE employees TO TIMESTAMP (SYSDATE - INTERVAL '1' HOUR);
```

### CTEs (WITH clause)

```sql
-- Récursif avec Oracle (équivalent de CONNECT BY)
WITH dept_hierarchy (dept_id, parent_id, name, level_num) AS (
    -- Ancre : départements racines
    SELECT department_id, parent_department_id, department_name, 1
    FROM departments
    WHERE parent_department_id IS NULL

    UNION ALL

    -- Partie récursive
    SELECT d.department_id, d.parent_department_id, d.department_name, h.level_num + 1
    FROM departments d
    JOIN dept_hierarchy h ON d.parent_department_id = h.dept_id
)
SELECT LPAD(' ', 2*(level_num-1)) || name AS department
FROM dept_hierarchy
ORDER BY level_num, name;
```

---

## 10. SQL*Plus et SQL Developer

```sql
-- SQL*Plus : interface ligne de commande Oracle
-- Connexion
sqlplus admin/password@hostname:1521/ORCL
sqlplus / as sysdba                  -- Connexion admin OS

-- Commandes SQL*Plus
SET SERVEROUTPUT ON SIZE UNLIMITED   -- Activer DBMS_OUTPUT
SET LINESIZE 200                     -- Largeur d'affichage
SET PAGESIZE 50                      -- Lignes par page
SET TIMING ON                        -- Afficher le temps d'exécution

DESCRIBE employees;                  -- Voir la structure d'une table
SHOW ERRORS;                         -- Voir les erreurs de compilation PL/SQL
EXEC pkg_hr_utils.give_raise(101, 10, :v);  -- Exécuter une procédure

-- Exécuter un script SQL
@/path/to/script.sql
@@relative_script.sql                -- Relatif au script courant

-- Sauvegarder la sortie
SPOOL /tmp/output.txt
SELECT * FROM employees;
SPOOL OFF

-- Variables de substitution
DEFINE employee_id = 101
SELECT * FROM employees WHERE employee_id = &employee_id;
```

---

## 11. Wikilinks et ressources

- [[01 - Introduction au SQL]] — Bases SQL communes
- [[02 - Requetes Avancees SQL]] — Jointures, sous-requêtes, window functions
- [[04 - SQL Avance et Administration]] — Indexes, optimisation, transactions

---

## Exercices Pratiques

### Exercice 1 — PL/SQL procédural
```sql
-- Écrire une procédure CALCULATE_BONUS qui :
-- 1. Prend en entrée : employee_id, evaluation_score (1-5)
-- 2. Calcule le bonus selon : score 5 = 20%, 4 = 15%, 3 = 10%, 1-2 = 0%
-- 3. Vérifie que le bonus ne dépasse pas le plafond du poste (jobs.max_salary)
-- 4. Met à jour la table employees
-- 5. Insère dans bonus_history (à créer : employee_id, bonus_amount, reason, date)
-- 6. Gère les exceptions : employee non trouvé, score invalide, plafond dépassé
-- 7. Appeler depuis Python avec oracledb
```

### Exercice 2 — Package complet
Créer `PKG_REPORT` avec :
1. `get_top_earners(p_n IN NUMBER)` : retourne un cursor REF CURSOR
2. `dept_salary_report` : affiche via DBMS_OUTPUT le rapport de chaque département
3. `export_to_file(p_path IN VARCHAR2)` : exporte le rapport via UTL_FILE
4. Variable de package comptant les appels (`g_report_count`)

### Exercice 3 — Triggers audit complet
```sql
-- Créer un système d'audit pour la table ORDERS :
-- 1. Trigger BEFORE INSERT : générer order_id depuis une séquence, valider les champs
-- 2. Trigger AFTER INSERT/UPDATE/DELETE : logguer dans orders_audit
--    (colonnes : action, order_id, old_amount, new_amount, changed_by, changed_at)
-- 3. Trigger BEFORE UPDATE : interdire la modification des commandes livrées
-- 4. Vérifier avec des SELECT que l'audit fonctionne
```

### Exercice 4 — Optimisation
```sql
-- Requête lente (>30 secondes sur 10 millions de lignes) à optimiser :
SELECT c.customer_name,
       COUNT(o.order_id) AS order_count,
       SUM(o.total_amount) AS lifetime_value
FROM customers c
JOIN orders o ON UPPER(c.customer_email) = UPPER(o.customer_email)  -- Problème !
WHERE o.order_date >= TO_DATE('2023-01-01', 'YYYY-MM-DD')
GROUP BY c.customer_name
HAVING SUM(o.total_amount) > 10000;

-- 1. Identifier les problèmes avec EXPLAIN PLAN
-- 2. Créer les index appropriés (dont function-based)
-- 3. Réécrire la requête si nécessaire
-- 4. Mesurer l'amélioration avec SET TIMING ON
```

> [!tip] Bonnes pratiques PL/SQL
> - Toujours utiliser `%TYPE` et `%ROWTYPE` pour les variables — évite les refactorings douloureux
> - Préférer le Cursor FOR Loop au OPEN/FETCH/CLOSE — plus concis, moins d'erreurs
> - RAISE_APPLICATION_ERROR pour les erreurs métier (-20000 à -20999)
> - Ne jamais laisser un `WHEN OTHERS` vide — toujours logger ou RAISE
> - Préférer les procédures dans des packages — meilleure encapsulation et performance (invalide moins de code à la recompilation)
