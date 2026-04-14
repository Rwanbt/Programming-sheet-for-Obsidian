# Tests Unitaires et TDD

Les tests sont la fondation de tout logiciel fiable. Un code sans tests est un code qui **fonctionne par chance**. Ce cours couvre les tests unitaires avec **pytest**, le framework de test le plus populaire en Python, et le **TDD** (Test-Driven Development), une methodologie qui consiste a ecrire les tests *avant* le code.

Tester, c'est investir du temps maintenant pour en economiser enormement plus tard.

> [!tip] Analogie
> Imaginez que vous construisez une maison. Les tests unitaires sont les **verifications de chaque brique** avant de les poser : est-elle solide ? A-t-elle les bonnes dimensions ? Le TDD, c'est comme dessiner le **plan de controle qualite** avant meme de commander les briques. Vous savez exactement ce que chaque piece doit faire avant de la construire.

---

## 1. Pourquoi Tester ?

### Le cout des bugs

Plus un bug est decouvert tard, plus il coute cher a corriger :

```
Cout relatif de correction d'un bug selon la phase de decouverte :

Phase            Cout      Barre
─────────────────────────────────────────────────────────
Developpement    $1        █
Tests unitaires  $5        █████
Integration      $15       ███████████████
QA / Staging     $50       ██████████████████████████████████████
Production       $100+     ██████████████████████████████████████████████████████
                           ──────────────────────────────────────────────────>
                           Plus c'est tard, plus c'est cher !
```

> [!info] Statistiques
> - Un bug decouvert en developpement coute **$1** a corriger
> - Le meme bug en production coute **$100+** (rollback, hotfix, impact utilisateurs, reputation)
> - Les projets avec une bonne couverture de tests ont **40% moins de bugs** en production
> - Le temps passe a ecrire des tests est recupere **3 a 5 fois** en maintenance reduite

### Ce que les tests garantissent

- **Confiance** : le code fonctionne comme prevu
- **Regression** : les modifications ne cassent pas l'existant
- **Documentation** : les tests montrent comment utiliser le code
- **Refactoring** : on peut modifier le code en toute securite
- **Collaboration** : les nouveaux developeurs comprennent le comportement attendu

---

## 2. La Pyramide des Tests

```
                          /\
                         /  \
                        / E2E\           Peu de tests
                       / (UI) \          Lents, couteux
                      /--------\         Haute confiance
                     /Integration\       Nombre moyen
                    / (API, DB)   \      Vitesse moyenne
                   /---------------\
                  /  Tests Unitaires \   Beaucoup de tests
                 /   (fonctions,     \  Tres rapides
                /     classes)        \ Faible cout
               /________________________\

  ← Vitesse    ████████████████████████████  Rapide
  ← Cout       █████                         Pas cher
  ← Nombre     ████████████████████████████  Beaucoup
  ← Portee     ███                           Petite (1 fonction)
  ← Confiance  ████████                      Moderee
```

> [!info] La regle 70/20/10
> Une distribution courante :
> - **70%** de tests unitaires (rapides, isoles, nombreux)
> - **20%** de tests d'integration (API, base de donnees)
> - **10%** de tests E2E (parcours utilisateur complet)

---

## 3. Anatomie d'un Test

Tout test suit un pattern en 3 etapes :

### Arrange-Act-Assert (AAA)

```python
def test_addition():
    # Arrange (preparer) - mise en place des donnees
    a = 5
    b = 3

    # Act (agir) - executer l'action a tester
    result = a + b

    # Assert (verifier) - verifier le resultat
    assert result == 8
```

### Given-When-Then (equivalent BDD)

```python
def test_user_can_login():
    # Given (etant donne) - un utilisateur existant
    user = User(username="alice", password="secret123")

    # When (quand) - il tente de se connecter
    result = login(username="alice", password="secret123")

    # Then (alors) - la connexion reussit
    assert result.success is True
    assert result.user.username == "alice"
```

> [!tip] Conseil
> Gardez toujours ces 3 sections distinctes dans vos tests. Si un test devient confus, revenez a AAA : "Qu'est-ce que je prepare ? Qu'est-ce que j'execute ? Qu'est-ce que je verifie ?"

---

## 4. pytest : Installation et Decouverte

### Installation

```bash
# Installer pytest
pip install pytest

# Verifier la version
pytest --version
# pytest 8.x.x
```

### Convention de nommage (decouverte automatique)

pytest decouvre automatiquement les tests selon ces regles :

```
mon_projet/
├── src/
│   ├── calculator.py
│   └── user.py
├── tests/                    # Dossier "tests" (ou "test")
│   ├── __init__.py           # Optionnel (recommande)
│   ├── conftest.py           # Fixtures partagees
│   ├── test_calculator.py    # Fichier prefixe "test_"
│   └── test_user.py
├── pyproject.toml
└── pytest.ini                # Optionnel : configuration

Regles de decouverte :
- Fichiers : test_*.py ou *_test.py
- Fonctions : test_*
- Classes : Test* (sans __init__)
- Methodes : test_*
```

### Lancer les tests

```bash
# Tous les tests
pytest

# Mode verbose (detail de chaque test)
pytest -v

# Un fichier specifique
pytest tests/test_calculator.py

# Un test specifique
pytest tests/test_calculator.py::test_addition

# Une classe specifique
pytest tests/test_calculator.py::TestCalculator

# Filtrer par nom (mot-cle)
pytest -k "addition"
pytest -k "addition or subtraction"
pytest -k "not slow"

# Arreter au premier echec
pytest -x

# Arreter apres N echecs
pytest --maxfail=3

# Afficher les print() dans les tests
pytest -s

# Traceback court / long / ligne seule
pytest --tb=short
pytest --tb=long
pytest --tb=line
pytest --tb=no          # Pas de traceback

# Combinaisons courantes
pytest -v -x --tb=short    # Verbose, arret au 1er echec, traceback court
```

---

## 5. Ecrire des Tests

### Tests avec des fonctions

```python
# src/calculator.py
def add(a, b):
    return a + b

def divide(a, b):
    if b == 0:
        raise ValueError("Division par zero impossible")
    return a / b

def is_even(n):
    return n % 2 == 0
```

```python
# tests/test_calculator.py
from src.calculator import add, divide, is_even

def test_add_positive_numbers():
    assert add(2, 3) == 5

def test_add_negative_numbers():
    assert add(-1, -1) == -2

def test_add_mixed_numbers():
    assert add(-1, 1) == 0

def test_add_floats():
    result = add(0.1, 0.2)
    assert result == pytest.approx(0.3)  # Pour les flottants !

def test_divide_normal():
    assert divide(10, 2) == 5.0

def test_divide_returns_float():
    result = divide(7, 2)
    assert result == 3.5
    assert isinstance(result, float)

def test_is_even_true():
    assert is_even(4) is True

def test_is_even_false():
    assert is_even(3) is False

def test_is_even_zero():
    assert is_even(0) is True
```

### Tests avec des classes

```python
# tests/test_calculator_class.py
class TestAdd:
    """Regrouper les tests par fonctionnalite"""

    def test_positive_numbers(self):
        assert add(2, 3) == 5

    def test_negative_numbers(self):
        assert add(-1, -1) == -2

    def test_with_zero(self):
        assert add(5, 0) == 5


class TestDivide:
    def test_integer_division(self):
        assert divide(10, 2) == 5.0

    def test_float_result(self):
        assert divide(7, 2) == 3.5
```

### Assertions detaillees

```python
import pytest

def test_assertions_variees():
    # Egalite
    assert add(2, 3) == 5

    # Inegalite
    assert add(2, 3) != 6

    # Comparaison
    assert add(2, 3) > 4
    assert add(2, 3) >= 5
    assert add(2, 3) <= 5

    # Booleen
    assert is_even(4)           # True
    assert not is_even(3)       # Not False

    # Appartenance
    assert "alice" in ["alice", "bob"]
    assert "charlie" not in ["alice", "bob"]

    # Type
    assert isinstance(add(1, 2), int)

    # None
    result = None
    assert result is None
    assert add(1, 2) is not None

    # Flottants (precision)
    assert 0.1 + 0.2 == pytest.approx(0.3)
    assert add(0.1, 0.2) == pytest.approx(0.3, rel=1e-9)

    # Chaines
    assert "hello world".startswith("hello")
    assert "error" in "an error occurred"
```

---

## 6. Fixtures

Les **fixtures** sont le mecanisme de pytest pour preparer les donnees et l'environnement des tests. Elles remplacent les `setUp` et `tearDown` de unittest.

### Fixture de base

```python
import pytest
from src.user import User

@pytest.fixture
def sample_user():
    """Cree un utilisateur pour les tests"""
    return User(username="alice", email="alice@test.com")

@pytest.fixture
def admin_user():
    return User(username="admin", email="admin@test.com", role="admin")

# Utiliser la fixture : simplement la passer en parametre
def test_user_has_username(sample_user):
    assert sample_user.username == "alice"

def test_user_has_email(sample_user):
    assert sample_user.email == "alice@test.com"

def test_admin_role(admin_user):
    assert admin_user.role == "admin"
```

### Scope des fixtures

```python
@pytest.fixture(scope="function")   # Defaut : recree pour chaque test
def user():
    return User("alice")

@pytest.fixture(scope="class")      # Recree pour chaque classe de test
def db_connection():
    conn = create_connection()
    yield conn
    conn.close()

@pytest.fixture(scope="module")     # Recree pour chaque fichier de test
def api_client():
    return TestClient(app)

@pytest.fixture(scope="session")    # Cree une seule fois pour toute la session
def database():
    db = setup_database()
    yield db
    teardown_database(db)
```

```
Portee des fixtures :

session  ──────────────────────────────────────────────────>
           module A                    module B
module   ──────────────────>  ─────────────────────>
             class X    class Y
class    ─────────>  ──────>  ─────────────────────>
           test1 test2 test3    test4 test5
function ──> ──> ──> ──> ──>  ──> ──> ──> ──> ──>
```

### Yield fixtures (setup + teardown)

```python
@pytest.fixture
def temp_file():
    # SETUP : avant le yield
    import tempfile, os
    fd, path = tempfile.mkstemp(suffix=".txt")
    os.close(fd)
    with open(path, "w") as f:
        f.write("contenu de test")

    yield path  # <- le test recoit 'path'

    # TEARDOWN : apres le yield (toujours execute)
    os.unlink(path)


def test_file_exists(temp_file):
    import os
    assert os.path.exists(temp_file)

def test_file_content(temp_file):
    with open(temp_file) as f:
        assert f.read() == "contenu de test"
```

> [!info] yield vs return
> - `return` : pas de teardown, la fixture cree et retourne
> - `yield` : la partie avant `yield` est le setup, la partie apres est le teardown. Le teardown s'execute meme si le test echoue.

### conftest.py

Le fichier `conftest.py` contient les fixtures partagees entre plusieurs fichiers de test. pytest le detecte automatiquement.

```python
# tests/conftest.py
import pytest
from src.database import Database
from src.user import User

@pytest.fixture(scope="session")
def db():
    """Base de donnees de test pour toute la session"""
    database = Database(":memory:")
    database.create_tables()
    yield database
    database.close()

@pytest.fixture
def sample_user(db):
    """Utilisateur de test (depend de la fixture db)"""
    user = User(username="test_user", email="test@mail.com")
    db.add(user)
    yield user
    db.delete(user)
```

```
Structure des conftest.py (hierarchie) :

tests/
├── conftest.py              # Fixtures globales (db, client)
├── test_models.py
├── api/
│   ├── conftest.py          # Fixtures specifiques aux tests API
│   ├── test_users_api.py
│   └── test_posts_api.py
└── unit/
    ├── conftest.py          # Fixtures specifiques aux tests unitaires
    ├── test_calculator.py
    └── test_utils.py
```

---

## 7. Parametrize

`@pytest.mark.parametrize` permet de lancer le meme test avec differentes donnees d'entree.

```python
import pytest
from src.calculator import add, is_even, divide

# Un seul parametre
@pytest.mark.parametrize("number, expected", [
    (0, True),
    (1, False),
    (2, True),
    (3, False),
    (-2, True),
    (100, True),
])
def test_is_even(number, expected):
    assert is_even(number) == expected

# Plusieurs parametres
@pytest.mark.parametrize("a, b, expected", [
    (1, 1, 2),
    (0, 0, 0),
    (-1, 1, 0),
    (100, 200, 300),
    (0.1, 0.2, pytest.approx(0.3)),
])
def test_add(a, b, expected):
    assert add(a, b) == expected

# Avec des IDs pour identifier chaque cas
@pytest.mark.parametrize("a, b, expected", [
    pytest.param(10, 2, 5.0, id="division-entiere"),
    pytest.param(7, 2, 3.5, id="division-decimale"),
    pytest.param(-10, 2, -5.0, id="division-negatif"),
    pytest.param(0, 5, 0.0, id="zero-divise"),
], ids=None)
def test_divide(a, b, expected):
    assert divide(a, b) == expected
```

> [!example] Sortie avec -v
> ```
> tests/test_calculator.py::test_is_even[0-True] PASSED
> tests/test_calculator.py::test_is_even[1-False] PASSED
> tests/test_calculator.py::test_is_even[2-True] PASSED
> tests/test_calculator.py::test_divide[division-entiere] PASSED
> tests/test_calculator.py::test_divide[division-decimale] PASSED
> ```

---

## 8. Markers

Les **markers** permettent de categoriser et filtrer les tests.

```python
import pytest
import sys

# Skip : toujours ignorer
@pytest.mark.skip(reason="Fonctionnalite pas encore implementee")
def test_feature_future():
    pass

# Skip conditionnel
@pytest.mark.skipif(
    sys.platform == "win32",
    reason="Ne fonctionne pas sur Windows"
)
def test_unix_only():
    import os
    os.symlink("/tmp/a", "/tmp/b")

# Expected failure (on sait que ca va echouer)
@pytest.mark.xfail(reason="Bug #123 connu, sera corrige en v2.0")
def test_known_bug():
    assert divide(1, 0) == float("inf")  # Leve ValueError pour l'instant

# Markers personnalises
@pytest.mark.slow
def test_heavy_computation():
    result = compute_primes(1_000_000)
    assert len(result) > 0

@pytest.mark.integration
def test_database_connection():
    db = connect_to_real_database()
    assert db.is_connected()
```

```ini
# pytest.ini ou pyproject.toml - Declarer les markers personnalises
[tool.pytest.ini_options]
markers = [
    "slow: tests lents (> 1s)",
    "integration: tests necessitant une base de donnees",
    "e2e: tests end-to-end",
]
```

```bash
# Lancer uniquement les tests marques "slow"
pytest -m slow

# Exclure les tests lents
pytest -m "not slow"

# Combinaisons
pytest -m "not slow and not integration"
pytest -m "slow or integration"
```

---

## 9. Mocking

Le **mocking** consiste a remplacer des dependances externes (API, base de donnees, fichiers...) par des objets simules. Cela permet de tester du code de maniere isolee.

### Mock et MagicMock

```python
from unittest.mock import Mock, MagicMock

# Mock basique
mock_db = Mock()
mock_db.get_user.return_value = {"id": 1, "name": "alice"}

result = mock_db.get_user(1)
assert result == {"id": 1, "name": "alice"}

# Verifier que la methode a ete appelee
mock_db.get_user.assert_called_once()
mock_db.get_user.assert_called_with(1)

# MagicMock : Mock avec les methodes magiques (__len__, __iter__, etc.)
mock_list = MagicMock()
mock_list.__len__.return_value = 5
assert len(mock_list) == 5
```

### patch : remplacer un objet temporairement

```python
from unittest.mock import patch
import requests

# src/weather.py
def get_temperature(city):
    response = requests.get(f"https://api.weather.com/{city}")
    data = response.json()
    return data["temperature"]
```

```python
# tests/test_weather.py
from unittest.mock import patch, Mock
from src.weather import get_temperature

# Methode 1 : patch comme decorateur
@patch("src.weather.requests.get")
def test_get_temperature(mock_get):
    # Configurer le mock
    mock_response = Mock()
    mock_response.json.return_value = {"temperature": 22.5}
    mock_get.return_value = mock_response

    # Tester
    result = get_temperature("Paris")

    # Verifier
    assert result == 22.5
    mock_get.assert_called_once_with("https://api.weather.com/Paris")

# Methode 2 : patch comme context manager
def test_get_temperature_with_context():
    with patch("src.weather.requests.get") as mock_get:
        mock_response = Mock()
        mock_response.json.return_value = {"temperature": 15.0}
        mock_get.return_value = mock_response

        result = get_temperature("Lyon")
        assert result == 15.0
```

### side_effect

```python
from unittest.mock import patch, Mock

# side_effect : definir un comportement dynamique
@patch("src.weather.requests.get")
def test_api_error(mock_get):
    # Simuler une exception
    mock_get.side_effect = ConnectionError("Pas de connexion")

    with pytest.raises(ConnectionError):
        get_temperature("Paris")

# side_effect avec une fonction
@patch("src.weather.requests.get")
def test_different_cities(mock_get):
    def fake_get(url):
        mock_resp = Mock()
        if "Paris" in url:
            mock_resp.json.return_value = {"temperature": 20.0}
        elif "Lyon" in url:
            mock_resp.json.return_value = {"temperature": 18.0}
        return mock_resp

    mock_get.side_effect = fake_get

    assert get_temperature("Paris") == 20.0
    assert get_temperature("Lyon") == 18.0

# side_effect avec une liste (valeurs successives)
@patch("src.weather.requests.get")
def test_retries(mock_get):
    mock_ok = Mock()
    mock_ok.json.return_value = {"temperature": 22.0}

    mock_get.side_effect = [
        ConnectionError("Echec 1"),
        ConnectionError("Echec 2"),
        mock_ok,  # Reussite au 3e essai
    ]
```

> [!warning] Ou patcher ?
> Patchez toujours a l'endroit ou l'objet est **utilise**, pas ou il est **defini** :
> ```python
> # src/weather.py importe requests
> # Patcher dans "src.weather.requests" (ou il est utilise)
> @patch("src.weather.requests.get")  # CORRECT
> @patch("requests.get")               # INCORRECT (patche globalement)
> ```

---

## 10. Tester les Exceptions

```python
import pytest
from src.calculator import divide

# Verifier qu'une exception est levee
def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(10, 0)

# Verifier le message de l'exception
def test_divide_by_zero_message():
    with pytest.raises(ValueError, match="Division par zero"):
        divide(10, 0)

# Capturer l'exception pour l'inspecter
def test_divide_by_zero_details():
    with pytest.raises(ValueError) as exc_info:
        divide(10, 0)

    assert str(exc_info.value) == "Division par zero impossible"
    assert exc_info.type is ValueError
```

---

## 11. Couverture de Code (Coverage)

La couverture de code mesure quel pourcentage du code est execute par les tests.

### Installation et utilisation

```bash
# Installer
pip install pytest-cov

# Lancer avec couverture
pytest --cov=src

# Rapport detaille dans le terminal
pytest --cov=src --cov-report=term-missing

# Generer un rapport HTML interactif
pytest --cov=src --cov-report=html
# Ouvre htmlcov/index.html dans le navigateur

# Rapport XML (pour CI/CD)
pytest --cov=src --cov-report=xml
```

> [!example] Sortie de coverage
> ```
> ---------- coverage: platform linux, python 3.12 ----------
> Name                    Stmts   Miss  Cover   Missing
> -------------------------------------------------------
> src/calculator.py          12      2    83%   15-16
> src/user.py                45      8    82%   30-35, 50-52
> src/weather.py             20      5    75%   25-29
> -------------------------------------------------------
> TOTAL                      77     15    81%
> ```

### Que viser ?

```
┌──────────────────────┬──────────┬──────────────────────────────┐
│ Couverture           │ Qualite  │ Commentaire                  │
├──────────────────────┼──────────┼──────────────────────────────┤
│ < 50%                │ Faible   │ Risque eleve de regression   │
│ 50% - 70%            │ Correct  │ Minimum acceptable           │
│ 70% - 85%            │ Bon      │ Standard pour la plupart     │
│ 85% - 95%            │ Tres bon │ Projets critiques            │
│ 95% - 100%           │ Excessif │ Cout/benefice diminue        │
└──────────────────────┴──────────┴──────────────────────────────┘
```

> [!warning] La couverture n'est pas tout
> 100% de couverture ne signifie pas 0 bug. La couverture mesure les **lignes executees**, pas la **qualite des assertions**. Un test sans `assert` a 100% de couverture mais ne verifie rien.

---

## 12. TDD : Test-Driven Development

Le TDD est une methodologie ou l'on ecrit les tests **avant** le code de production. Le cycle est strict : Red, Green, Refactor.

### Le cycle TDD

```
         ┌──────────────┐
         │   1. RED      │  Ecrire un test qui ECHOUE
         │   (rouge)     │  (le code n'existe pas encore)
         └──────┬───────┘
                │
                v
         ┌──────────────┐
         │   2. GREEN    │  Ecrire le MINIMUM de code
         │   (vert)      │  pour que le test passe
         └──────┬───────┘
                │
                v
         ┌──────────────┐
         │  3. REFACTOR  │  Ameliorer le code
         │  (refactoriser)│  SANS casser les tests
         └──────┬───────┘
                │
                └──────> Retour a 1. RED
                         (prochain test)

Temps :  RED ──> GREEN ──> REFACTOR ──> RED ──> GREEN ──> ...
Tests :   X       V          V          X        V
          echoue  passe      passe      echoue   passe
```

### Regles du TDD

1. **N'ecrivez PAS de code de production** tant qu'il n'y a pas un test qui echoue
2. **N'ecrivez PAS plus de test** que necessaire pour provoquer un echec
3. **N'ecrivez PAS plus de code** que necessaire pour faire passer le test

> [!tip] Pourquoi ca marche ?
> Le TDD force a :
> - Definir le comportement avant l'implementation
> - Garder le code minimal et sans superflu
> - Avoir une couverture de tests proche de 100%
> - Concevoir des interfaces claires et testables

---

## 13. Exemple Pratique TDD : Classe Calculator

Construisons une classe `Calculator` pas a pas avec le TDD.

### Etape 1 : RED - Premier test

```python
# tests/test_calculator_tdd.py

def test_calculator_exists():
    """Le calculateur doit exister"""
    calc = Calculator()
    assert calc is not None
```

```bash
pytest tests/test_calculator_tdd.py -v
# FAILED - NameError: name 'Calculator' is not defined
# -> RED : le test echoue, c'est normal !
```

### Etape 2 : GREEN - Minimum de code

```python
# src/calculator.py
class Calculator:
    pass
```

```python
# tests/test_calculator_tdd.py
from src.calculator import Calculator

def test_calculator_exists():
    calc = Calculator()
    assert calc is not None
```

```bash
pytest -v  # PASSED -> GREEN
```

### Etape 3 : RED - Test d'addition

```python
def test_add_two_numbers():
    calc = Calculator()
    assert calc.add(2, 3) == 5
```

```bash
pytest -v  # FAILED - AttributeError: 'Calculator' has no attribute 'add'
```

### Etape 4 : GREEN - Implementer add

```python
class Calculator:
    def add(self, a, b):
        return a + b
```

```bash
pytest -v  # PASSED
```

### Etape 5 : RED - Test de soustraction

```python
def test_subtract():
    calc = Calculator()
    assert calc.subtract(10, 3) == 7
```

### Etape 6 : GREEN - Implementer subtract

```python
class Calculator:
    def add(self, a, b):
        return a + b

    def subtract(self, a, b):
        return a - b
```

### Etape 7 : RED - Division et cas d'erreur

```python
def test_divide():
    calc = Calculator()
    assert calc.divide(10, 2) == 5.0

def test_divide_by_zero():
    calc = Calculator()
    with pytest.raises(ValueError, match="zero"):
        calc.divide(10, 0)
```

### Etape 8 : GREEN

```python
class Calculator:
    def add(self, a, b):
        return a + b

    def subtract(self, a, b):
        return a - b

    def divide(self, a, b):
        if b == 0:
            raise ValueError("Division par zero impossible")
        return a / b
```

### Etape 9 : REFACTOR - Fixture et plus de tests

```python
# tests/test_calculator_tdd.py
import pytest
from src.calculator import Calculator

@pytest.fixture
def calc():
    return Calculator()

class TestAdd:
    def test_positive_numbers(self, calc):
        assert calc.add(2, 3) == 5

    def test_negative_numbers(self, calc):
        assert calc.add(-1, -1) == -2

    def test_floats(self, calc):
        assert calc.add(0.1, 0.2) == pytest.approx(0.3)

class TestSubtract:
    def test_basic(self, calc):
        assert calc.subtract(10, 3) == 7

    def test_negative_result(self, calc):
        assert calc.subtract(3, 10) == -7

class TestDivide:
    def test_integer_division(self, calc):
        assert calc.divide(10, 2) == 5.0

    def test_float_division(self, calc):
        assert calc.divide(7, 2) == 3.5

    def test_divide_by_zero(self, calc):
        with pytest.raises(ValueError, match="zero"):
            calc.divide(10, 0)

class TestHistory:
    """Nouvelle fonctionnalite : historique des operations"""

    def test_history_starts_empty(self, calc):
        assert calc.history == []

    def test_add_records_history(self, calc):
        calc.add(2, 3)
        assert len(calc.history) == 1
        assert calc.history[0] == "2 + 3 = 5"
```

```python
# src/calculator.py - Version finale apres refactoring
class Calculator:
    def __init__(self):
        self.history = []

    def add(self, a, b):
        result = a + b
        self._record(f"{a} + {b} = {result}")
        return result

    def subtract(self, a, b):
        result = a - b
        self._record(f"{a} - {b} = {result}")
        return result

    def divide(self, a, b):
        if b == 0:
            raise ValueError("Division par zero impossible")
        result = a / b
        self._record(f"{a} / {b} = {result}")
        return result

    def _record(self, operation):
        self.history.append(operation)
```

---

## 14. unittest : Le Module Standard

`unittest` est le module de test integre a Python (inspire de JUnit). Moins pratique que pytest, mais important a connaitre.

```python
import unittest
from src.calculator import Calculator

class TestCalculator(unittest.TestCase):
    def setUp(self):
        """Appele avant CHAQUE test (equivalent de fixture)"""
        self.calc = Calculator()

    def tearDown(self):
        """Appele apres CHAQUE test (nettoyage)"""
        pass

    def test_add(self):
        self.assertEqual(self.calc.add(2, 3), 5)

    def test_subtract(self):
        self.assertEqual(self.calc.subtract(10, 3), 7)

    def test_divide(self):
        self.assertAlmostEqual(self.calc.divide(7, 2), 3.5)

    def test_divide_by_zero(self):
        with self.assertRaises(ValueError):
            self.calc.divide(10, 0)

    def test_types(self):
        self.assertIsInstance(self.calc.add(1, 2), int)
        self.assertTrue(self.calc.add(1, 2) > 0)
        self.assertIn("alice", ["alice", "bob"])

if __name__ == "__main__":
    unittest.main()
```

### Methodes d'assertion de unittest

```
┌───────────────────────────┬────────────────────────────┐
│ Methode unittest          │ Equivalent pytest          │
├───────────────────────────┼────────────────────────────┤
│ assertEqual(a, b)         │ assert a == b              │
│ assertNotEqual(a, b)      │ assert a != b              │
│ assertTrue(x)             │ assert x                   │
│ assertFalse(x)            │ assert not x               │
│ assertIs(a, b)            │ assert a is b              │
│ assertIsNone(x)           │ assert x is None           │
│ assertIn(a, b)            │ assert a in b              │
│ assertIsInstance(a, b)    │ assert isinstance(a, b)    │
│ assertRaises(Exc)         │ pytest.raises(Exc)         │
│ assertAlmostEqual(a, b)   │ assert a == pytest.approx(b)│
└───────────────────────────┴────────────────────────────┘
```

---

## 15. pytest vs unittest

```
┌──────────────────────┬───────────────────────┬───────────────────────┐
│ Aspect               │ pytest                │ unittest              │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Style                │ Fonctions simples     │ Classes + TestCase    │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Assertions           │ assert natif Python   │ self.assertXxx()      │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Fixtures             │ @pytest.fixture       │ setUp / tearDown      │
│                      │ (flexibles, scopes)   │ (limites)             │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Parametrize          │ @pytest.mark.param.   │ Pas natif             │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Plugins              │ Ecosysteme riche      │ Limite                │
│                      │ (1000+ plugins)       │                       │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Decouverte           │ Automatique           │ Manuelle ou discover  │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Messages d'erreur    │ Tres detailles        │ Basiques              │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Compatibilite        │ Execute aussi les     │ Uniquement unittest   │
│                      │ tests unittest        │                       │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Standard Python      │ Non (pip install)     │ Oui (inclus)          │
├──────────────────────┼───────────────────────┼───────────────────────┤
│ Recommandation       │ Nouveau projet        │ Code legacy           │
└──────────────────────┴───────────────────────┴───────────────────────┘
```

> [!tip] Conseil pratique
> Utilisez **pytest** pour tout nouveau projet. Il peut aussi executer les tests `unittest` existants, donc pas besoin de tout reecrire si vous migrez.

---

## 16. Patterns de Test

### Pattern Factory

```python
# Au lieu de creer des objets dans chaque test
@pytest.fixture
def user_factory():
    def _create_user(username="test", email=None, role="user"):
        if email is None:
            email = f"{username}@test.com"
        return User(username=username, email=email, role=role)
    return _create_user

def test_default_user(user_factory):
    user = user_factory()
    assert user.role == "user"

def test_admin_user(user_factory):
    admin = user_factory(username="admin", role="admin")
    assert admin.role == "admin"
```

### Pattern Builder

```python
class UserBuilder:
    def __init__(self):
        self._username = "default"
        self._email = "default@test.com"
        self._role = "user"

    def with_username(self, name):
        self._username = name
        return self

    def with_role(self, role):
        self._role = role
        return self

    def build(self):
        return User(self._username, self._email, self._role)

@pytest.fixture
def user_builder():
    return UserBuilder()

def test_admin(user_builder):
    admin = user_builder.with_username("admin").with_role("admin").build()
    assert admin.role == "admin"
```

### Tester des effets de bord

```python
def test_send_welcome_email():
    """Verifier qu'un email est envoye a l'inscription"""
    with patch("src.user.send_email") as mock_send:
        register_user("alice", "alice@test.com")

        mock_send.assert_called_once_with(
            to="alice@test.com",
            subject="Bienvenue alice !",
            body=unittest.mock.ANY  # N'importe quel contenu
        )
```

---

## Carte Mentale

```
                      Tests Unitaires et TDD
                              │
          ┌──────────┬────────┼────────┬──────────┐
          │          │        │        │          │
       pytest    Fixtures   TDD     Mocking   Coverage
          │          │        │        │          │
     ┌────┤     ┌────┤    ┌───┤    ┌───┤     ┌────┤
     │    │     │    │    │   │    │   │     │    │
   assert │  @fixture │  Red   │  Mock   │ --cov   │
     │    │     │    │    │   │    │   │     │    │
    -v    │  scope   │ Green  │ patch  │  html    │
     │    │     │    │    │   │    │   │     │    │
    -k    │  yield   │ Refactor│ side   │  term    │
     │    │     │    │    │   │  effect │ -missing │
    -x    │ conftest │       │    │     │         │
     │    │   .py    │ assert │ return │  70-85%  │
  --tb    │         │ _called│ _value │  cible   │
          │ parametrize      │        │         │
       markers      │      MagicMock │
          │    ┌────┤              │
       ┌──┤    │    │         pytest.raises
       │  │  skip  xfail
      slow │
    integration
```

---

## Exercices

### Exercice 1 : Tests d'une classe Stack

Ecrivez les tests (en TDD) pour une classe `Stack` (pile) :
- `push(item)` : ajouter un element
- `pop()` : retirer et retourner le dernier element (leve `IndexError` si vide)
- `peek()` : voir le dernier element sans le retirer
- `is_empty()` : retourne `True` si la pile est vide
- `size()` : retourne le nombre d'elements

Ecrivez d'abord les tests, puis implementez la classe.

### Exercice 2 : Tests avec Mocking

Creez une fonction `get_user_repos(username)` qui appelle l'API GitHub pour obtenir les depots d'un utilisateur. Ecrivez des tests avec `patch` pour :
- Tester le cas normal (l'API retourne une liste de repos)
- Tester le cas d'erreur 404 (utilisateur non trouve)
- Tester le cas de timeout (ConnectionError)
- Verifier que l'URL appelee est correcte

### Exercice 3 : Parametrize complet

Ecrivez une fonction `validate_password(password)` qui verifie qu'un mot de passe respecte : au moins 8 caracteres, au moins une majuscule, au moins un chiffre, au moins un caractere special. Utilisez `@pytest.mark.parametrize` pour tester au moins 10 cas (valides et invalides).

---

## Liens

- [[02 - Tests Integration et E2E]] - Tests d'integration et end-to-end
- [[08 - APIs REST avec Flask]] - Tester des APIs Flask
