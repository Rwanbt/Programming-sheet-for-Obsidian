# Tests Integration et E2E

Les tests unitaires verifient que chaque piece fonctionne isolement. Mais une application est un **assemblage** de pieces : base de donnees, API, interface utilisateur. Les tests d'integration verifient que ces pieces fonctionnent **ensemble**, et les tests end-to-end (E2E) simulent un **parcours utilisateur complet**.

Ce cours couvre les tests d'integration pour les APIs (Flask, FastAPI), les tests E2E avec Selenium et Playwright, la couverture de code, et l'integration dans un pipeline CI/CD.

> [!tip] Analogie
> Imaginez une chaine de montage automobile. Les tests **unitaires** verifient chaque piece individuellement (le moteur tourne, les freins serrent). Les tests **d'integration** verifient que les pieces fonctionnent ensemble (le moteur fait avancer la voiture, les freins l'arretent). Les tests **E2E** prennent la voiture finie et la conduisent sur une route : demarrage, acceleration, virage, freinage, stationnement. Chaque niveau de test attrape des bugs differents.

---

## 1. Unite vs Integration vs E2E

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Portee des differents tests                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Unitaire          Integration              E2E                     │
│  ┌───────┐        ┌─────────────┐          ┌──────────────────────┐│
│  │ func()│        │ API + DB    │          │ Navigateur + API +   ││
│  │       │        │             │          │ DB + Auth + Email    ││
│  └───────┘        └─────────────┘          └──────────────────────┘│
│                                                                     │
│  1 fonction       Plusieurs composants     Le systeme entier       │
│  Millisecondes    Secondes                 Minutes                  │
│  Mocks partout    Vrais services (ou       Vrais services          │
│                   conteneurs Docker)       + vrai navigateur       │
└─────────────────────────────────────────────────────────────────────┘
```

```
┌──────────────┬─────────────────┬──────────────────┬──────────────────┐
│              │ Unitaire        │ Integration      │ E2E              │
├──────────────┼─────────────────┼──────────────────┼──────────────────┤
│ Portee       │ 1 fonction/     │ Plusieurs        │ Parcours         │
│              │ classe          │ composants       │ utilisateur      │
├──────────────┼─────────────────┼──────────────────┼──────────────────┤
│ Vitesse      │ Tres rapide     │ Moyen            │ Lent             │
│              │ (< 10ms)        │ (100ms - 2s)     │ (5s - 60s)       │
├──────────────┼─────────────────┼──────────────────┼──────────────────┤
│ Dependances  │ Mockees         │ Reelles (DB,     │ Tout reel        │
│              │                 │ API locale)      │                  │
├──────────────┼─────────────────┼──────────────────┼──────────────────┤
│ Confiance    │ Moderee         │ Elevee           │ Tres elevee      │
├──────────────┼─────────────────┼──────────────────┼──────────────────┤
│ Maintenance  │ Faible          │ Moyenne          │ Elevee           │
├──────────────┼─────────────────┼──────────────────┼──────────────────┤
│ Debug        │ Facile (isole)  │ Moyen            │ Difficile        │
├──────────────┼─────────────────┼──────────────────┼──────────────────┤
│ Proportion   │ ~70%            │ ~20%             │ ~10%             │
└──────────────┴─────────────────┴──────────────────┴──────────────────┘
```

> [!info] Quel bug chaque type attrape ?
> - **Unitaire** : erreur de logique dans une fonction (mauvais calcul, condition inversee)
> - **Integration** : le code fonctionne en isolation mais la requete SQL est fausse, le format JSON est incorrect, l'authentification echoue
> - **E2E** : le formulaire d'inscription fonctionne mais le bouton "Valider" est cache par un element CSS

---

## 2. Tests d'Integration : API Flask

### Test client Flask

Flask fournit un client de test integre qui simule des requetes HTTP sans lancer de serveur.

```python
# src/app.py
from flask import Flask, jsonify, request

app = Flask(__name__)

# Simulons une base de donnees en memoire
users_db = {}
next_id = 1

@app.route("/users", methods=["GET"])
def list_users():
    return jsonify(list(users_db.values()))

@app.route("/users", methods=["POST"])
def create_user():
    global next_id
    data = request.get_json()
    if not data or "username" not in data:
        return jsonify({"error": "username requis"}), 400
    user = {"id": next_id, "username": data["username"]}
    users_db[next_id] = user
    next_id += 1
    return jsonify(user), 201

@app.route("/users/<int:user_id>", methods=["GET"])
def get_user(user_id):
    user = users_db.get(user_id)
    if not user:
        return jsonify({"error": "Utilisateur non trouve"}), 404
    return jsonify(user)

@app.route("/users/<int:user_id>", methods=["DELETE"])
def delete_user(user_id):
    if user_id not in users_db:
        return jsonify({"error": "Utilisateur non trouve"}), 404
    del users_db[user_id]
    return "", 204
```

```python
# tests/test_app.py
import pytest
from src.app import app, users_db

@pytest.fixture
def client():
    """Client de test Flask"""
    app.config["TESTING"] = True
    with app.test_client() as client:
        yield client

@pytest.fixture(autouse=True)
def reset_db():
    """Reinitialiser la base avant chaque test"""
    users_db.clear()
    yield
    users_db.clear()


class TestCreateUser:
    def test_create_user_success(self, client):
        response = client.post("/users", json={"username": "alice"})
        assert response.status_code == 201
        data = response.get_json()
        assert data["username"] == "alice"
        assert "id" in data

    def test_create_user_missing_username(self, client):
        response = client.post("/users", json={})
        assert response.status_code == 400
        assert "error" in response.get_json()

    def test_create_user_no_body(self, client):
        response = client.post("/users")
        assert response.status_code == 400


class TestGetUser:
    def test_get_existing_user(self, client):
        # Creer d'abord
        client.post("/users", json={"username": "alice"})
        # Puis lire
        response = client.get("/users/1")
        assert response.status_code == 200
        assert response.get_json()["username"] == "alice"

    def test_get_nonexistent_user(self, client):
        response = client.get("/users/999")
        assert response.status_code == 404


class TestListUsers:
    def test_empty_list(self, client):
        response = client.get("/users")
        assert response.status_code == 200
        assert response.get_json() == []

    def test_list_after_creation(self, client):
        client.post("/users", json={"username": "alice"})
        client.post("/users", json={"username": "bob"})
        response = client.get("/users")
        data = response.get_json()
        assert len(data) == 2


class TestDeleteUser:
    def test_delete_existing(self, client):
        client.post("/users", json={"username": "alice"})
        response = client.delete("/users/1")
        assert response.status_code == 204
        # Verifier qu'il n'existe plus
        assert client.get("/users/1").status_code == 404

    def test_delete_nonexistent(self, client):
        response = client.delete("/users/999")
        assert response.status_code == 404
```

---

## 3. Tests d'Integration : API FastAPI

### Test client FastAPI

FastAPI utilise `TestClient` de Starlette, qui est synchrone meme pour les endpoints async.

```python
# src/main.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class UserCreate(BaseModel):
    username: str
    email: str

class UserResponse(BaseModel):
    id: int
    username: str
    email: str

users_db: dict[int, dict] = {}
next_id = 1

@app.post("/users", response_model=UserResponse, status_code=201)
def create_user(user: UserCreate):
    global next_id
    new_user = {"id": next_id, "username": user.username, "email": user.email}
    users_db[next_id] = new_user
    next_id += 1
    return new_user

@app.get("/users/{user_id}", response_model=UserResponse)
def get_user(user_id: int):
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail="Utilisateur non trouve")
    return users_db[user_id]

@app.get("/users", response_model=list[UserResponse])
def list_users(skip: int = 0, limit: int = 10):
    all_users = list(users_db.values())
    return all_users[skip:skip + limit]
```

```python
# tests/test_main.py
import pytest
from fastapi.testclient import TestClient
from src.main import app, users_db

@pytest.fixture
def client():
    return TestClient(app)

@pytest.fixture(autouse=True)
def reset_db():
    users_db.clear()
    yield
    users_db.clear()


def test_create_user(client):
    response = client.post("/users", json={
        "username": "alice",
        "email": "alice@test.com"
    })
    assert response.status_code == 201
    data = response.json()
    assert data["username"] == "alice"
    assert data["email"] == "alice@test.com"
    assert "id" in data


def test_create_user_validation(client):
    """Pydantic valide automatiquement"""
    response = client.post("/users", json={"username": "alice"})
    assert response.status_code == 422  # Validation error


def test_get_user(client):
    # Creer
    client.post("/users", json={"username": "bob", "email": "bob@test.com"})
    # Lire
    response = client.get("/users/1")
    assert response.status_code == 200
    assert response.json()["username"] == "bob"


def test_get_user_not_found(client):
    response = client.get("/users/999")
    assert response.status_code == 404
    assert response.json()["detail"] == "Utilisateur non trouve"


def test_list_users_pagination(client):
    # Creer 5 utilisateurs
    for i in range(5):
        client.post("/users", json={
            "username": f"user{i}",
            "email": f"user{i}@test.com"
        })
    # Pagination
    response = client.get("/users?skip=2&limit=2")
    data = response.json()
    assert len(data) == 2
    assert data[0]["username"] == "user2"
```

---

## 4. Tests avec une Vraie Base de Donnees

Pour les tests d'integration serieux, on utilise une vraie base de donnees (souvent SQLite en memoire ou un conteneur Docker).

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from src.models import Base
from src.main import app, get_db

# Base de donnees de test en memoire
TEST_DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(TEST_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSession = sessionmaker(bind=engine)


@pytest.fixture(scope="session")
def setup_database():
    """Creer les tables une seule fois"""
    Base.metadata.create_all(bind=engine)
    yield
    Base.metadata.drop_all(bind=engine)


@pytest.fixture
def db_session(setup_database):
    """Session de test avec rollback automatique"""
    session = TestingSession()
    yield session
    session.rollback()
    session.close()


@pytest.fixture
def client(db_session):
    """Client de test avec la session de test injectee"""
    def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db
    from fastapi.testclient import TestClient
    with TestClient(app) as c:
        yield c
    app.dependency_overrides.clear()
```

```python
# tests/test_integration_db.py
def test_create_and_retrieve_user(client):
    """Test d'integration : creation et lecture via API + DB"""
    # Creer
    resp = client.post("/users", json={
        "username": "alice",
        "email": "alice@blog.com"
    })
    assert resp.status_code == 201
    user_id = resp.json()["id"]

    # Verifier en DB via l'API
    resp = client.get(f"/users/{user_id}")
    assert resp.status_code == 200
    assert resp.json()["username"] == "alice"


def test_create_duplicate_username(client):
    """Test d'integration : unicite en base"""
    client.post("/users", json={
        "username": "alice", "email": "alice1@test.com"
    })
    resp = client.post("/users", json={
        "username": "alice", "email": "alice2@test.com"
    })
    assert resp.status_code == 409  # Conflict
```

> [!warning] Base de test vs base de production
> Ne testez **jamais** contre la base de production. Utilisez :
> - SQLite en memoire (`:memory:`) pour les tests rapides
> - Un conteneur Docker avec PostgreSQL pour les tests realistes
> - Une base de test dediee avec des donnees jetables

---

## 5. Tests API avec requests et httpx

Pour tester une API **deployee** (pas en local), on utilise `requests` ou `httpx`.

### Avec requests

```python
import requests
import pytest

BASE_URL = "http://localhost:8000"

@pytest.fixture(scope="session")
def api_url():
    """URL de l'API de test"""
    return BASE_URL

def test_health_check(api_url):
    response = requests.get(f"{api_url}/health")
    assert response.status_code == 200
    assert response.json()["status"] == "ok"

def test_create_user(api_url):
    response = requests.post(f"{api_url}/users", json={
        "username": "test_user",
        "email": "test@api.com"
    })
    assert response.status_code == 201
    data = response.json()
    assert "id" in data
    assert data["username"] == "test_user"
```

### Avec httpx (async)

```python
import httpx
import pytest

@pytest.mark.asyncio
async def test_async_api():
    async with httpx.AsyncClient(base_url="http://localhost:8000") as client:
        response = await client.get("/users")
        assert response.status_code == 200
        assert isinstance(response.json(), list)
```

### Validation de reponses

```python
def test_response_structure(client):
    """Verifier la structure JSON de la reponse"""
    response = client.post("/users", json={
        "username": "alice",
        "email": "alice@test.com"
    })
    data = response.json()

    # Verifier que tous les champs attendus sont presents
    required_fields = {"id", "username", "email"}
    assert required_fields.issubset(data.keys())

    # Verifier les types
    assert isinstance(data["id"], int)
    assert isinstance(data["username"], str)
    assert isinstance(data["email"], str)

    # Verifier les valeurs
    assert data["id"] > 0
    assert "@" in data["email"]
```

### Codes de statut HTTP a tester

```
┌───────┬──────────────────────────┬──────────────────────────────┐
│ Code  │ Signification            │ Quand le tester              │
├───────┼──────────────────────────┼──────────────────────────────┤
│ 200   │ OK                       │ GET reussi, PUT reussi       │
│ 201   │ Created                  │ POST qui cree une ressource  │
│ 204   │ No Content               │ DELETE reussi                │
│ 400   │ Bad Request              │ Donnees invalides            │
│ 401   │ Unauthorized             │ Pas de token / token expire  │
│ 403   │ Forbidden                │ Token valide, droits insuff. │
│ 404   │ Not Found                │ Ressource inexistante        │
│ 409   │ Conflict                 │ Doublon (email unique, etc.) │
│ 422   │ Unprocessable Entity     │ Validation Pydantic echouee  │
│ 500   │ Internal Server Error    │ Bug dans le serveur          │
└───────┴──────────────────────────┴──────────────────────────────┘
```

---

## 6. Tests E2E : Concepts

Les tests E2E simulent un utilisateur reel interagissant avec l'application dans un navigateur. Ils testent le **parcours complet** : interface, API, base de donnees.

```
Parcours E2E typique :

Navigateur         API              Base de donnees
    │                │                    │
    │  1. Ouvrir /register               │
    │────────────>   │                    │
    │  2. Remplir le formulaire          │
    │  3. Cliquer "S'inscrire"           │
    │────────────>   │                    │
    │                │  4. INSERT user    │
    │                │──────────────────> │
    │                │  5. OK             │
    │                │<────────────────── │
    │  6. Redirect /login                │
    │<────────────   │                    │
    │  7. Remplir identifiants           │
    │  8. Cliquer "Connexion"            │
    │────────────>   │                    │
    │                │  9. SELECT user    │
    │                │──────────────────> │
    │  10. Redirect /dashboard           │
    │<────────────   │                    │
    │  11. Verifier "Bienvenue alice"    │
    v                v                    v
```

---

## 7. Selenium

**Selenium** est l'outil le plus ancien et le plus repandu pour les tests E2E. Il controle un vrai navigateur (Chrome, Firefox...).

### Installation

```bash
pip install selenium

# Installer aussi un driver (ou utiliser webdriver-manager)
pip install webdriver-manager
```

### Exemple complet

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import pytest


@pytest.fixture
def browser():
    """Lancer et fermer le navigateur"""
    options = webdriver.ChromeOptions()
    options.add_argument("--headless")       # Pas d'interface graphique
    options.add_argument("--no-sandbox")
    options.add_argument("--disable-gpu")

    service = Service(ChromeDriverManager().install())
    driver = webdriver.Chrome(service=service, options=options)
    driver.implicitly_wait(10)  # Attente implicite de 10s

    yield driver

    driver.quit()


def test_google_search(browser):
    """Test E2E : recherche Google"""
    # Ouvrir la page
    browser.get("https://www.google.com")

    # Trouver la barre de recherche
    search_box = browser.find_element(By.NAME, "q")

    # Taper une recherche
    search_box.send_keys("Python pytest")
    search_box.send_keys(Keys.RETURN)

    # Attendre les resultats
    WebDriverWait(browser, 10).until(
        EC.presence_of_element_located((By.ID, "search"))
    )

    # Verifier qu'il y a des resultats
    results = browser.find_elements(By.CSS_SELECTOR, "div.g")
    assert len(results) > 0


def test_login_flow(browser):
    """Test E2E : parcours de connexion"""
    browser.get("http://localhost:5000/login")

    # Remplir le formulaire
    username_input = browser.find_element(By.ID, "username")
    password_input = browser.find_element(By.ID, "password")
    submit_button = browser.find_element(By.CSS_SELECTOR, "button[type='submit']")

    username_input.send_keys("alice")
    password_input.send_keys("secret123")
    submit_button.click()

    # Attendre la redirection
    WebDriverWait(browser, 10).until(
        EC.url_contains("/dashboard")
    )

    # Verifier le contenu
    welcome = browser.find_element(By.CLASS_NAME, "welcome-message")
    assert "Bienvenue alice" in welcome.text


def test_screenshot_on_failure(browser):
    """Prendre une capture d'ecran en cas d'echec"""
    browser.get("http://localhost:5000")
    try:
        element = browser.find_element(By.ID, "nonexistent")
    except Exception:
        browser.save_screenshot("screenshots/failure.png")
        raise
```

### Localiser les elements

```python
# Par ID
element = browser.find_element(By.ID, "username")

# Par nom
element = browser.find_element(By.NAME, "email")

# Par classe CSS
element = browser.find_element(By.CLASS_NAME, "btn-primary")

# Par selecteur CSS (le plus flexible)
element = browser.find_element(By.CSS_SELECTOR, "form#login input[name='email']")
elements = browser.find_elements(By.CSS_SELECTOR, "ul.results li")

# Par XPath
element = browser.find_element(By.XPATH, "//button[text()='Valider']")

# Par texte du lien
element = browser.find_element(By.LINK_TEXT, "Se connecter")
element = browser.find_element(By.PARTIAL_LINK_TEXT, "connecter")

# Par tag
elements = browser.find_elements(By.TAG_NAME, "input")
```

### Attentes (waits)

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(browser, 10)  # Max 10 secondes

# Attendre qu'un element soit present dans le DOM
element = wait.until(EC.presence_of_element_located((By.ID, "results")))

# Attendre qu'un element soit visible
element = wait.until(EC.visibility_of_element_located((By.ID, "modal")))

# Attendre qu'un element soit cliquable
button = wait.until(EC.element_to_be_clickable((By.ID, "submit")))

# Attendre que le titre contienne un texte
wait.until(EC.title_contains("Dashboard"))

# Attendre qu'un element disparaisse
wait.until(EC.invisibility_of_element_located((By.ID, "spinner")))

# Attendre une URL
wait.until(EC.url_contains("/success"))
```

> [!warning] Attente implicite vs explicite
> - **Implicite** (`driver.implicitly_wait(10)`) : attend globalement pour tout `find_element`. Simple mais imprecis.
> - **Explicite** (`WebDriverWait`) : attend une condition specifique. Plus precis, recommande.
> - Ne melangez pas les deux.

---

## 8. Playwright : Alternative Moderne

**Playwright** est une alternative moderne a Selenium, developpee par Microsoft. Plus rapide, plus fiable, avec auto-attente.

### Installation

```bash
pip install playwright
playwright install  # Telecharge les navigateurs (Chromium, Firefox, WebKit)
```

### Exemple complet

```python
from playwright.sync_api import sync_playwright, expect
import pytest

@pytest.fixture
def page():
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        yield page
        browser.close()


def test_login_playwright(page):
    """Meme test de login, en Playwright"""
    page.goto("http://localhost:5000/login")

    # Remplir (auto-attente integree !)
    page.fill("#username", "alice")
    page.fill("#password", "secret123")
    page.click("button[type='submit']")

    # Attendre la navigation
    page.wait_for_url("**/dashboard")

    # Verifier
    welcome = page.locator(".welcome-message")
    expect(welcome).to_contain_text("Bienvenue alice")


def test_form_validation(page):
    page.goto("http://localhost:5000/register")

    # Soumettre un formulaire vide
    page.click("button[type='submit']")

    # Verifier le message d'erreur
    error = page.locator(".error-message")
    expect(error).to_be_visible()
    expect(error).to_have_text("Le nom d'utilisateur est requis")
```

### Codegen : generer des tests automatiquement

```bash
# Playwright enregistre vos actions et genere le code !
playwright codegen http://localhost:5000
# Un navigateur s'ouvre. Cliquez, tapez... le code Python est genere.
```

### Playwright vs Selenium

```
┌──────────────────────┬────────────────────────┬────────────────────────┐
│ Aspect               │ Selenium               │ Playwright             │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Attentes             │ Manuelles (WebDriver   │ Auto-attente integree  │
│                      │ Wait + EC)             │ (plus fiable)          │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Navigateurs          │ Chrome, Firefox, Edge, │ Chromium, Firefox,     │
│                      │ Safari (via drivers)   │ WebKit (inclus)        │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Installation         │ Driver externe requis  │ playwright install     │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ API                  │ Verbose, ancienne      │ Moderne, concise       │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Async                │ Non natif              │ Natif (sync + async)   │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Codegen              │ Non                    │ Oui (enregistreur)     │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Vitesse              │ Moyen                  │ Rapide                 │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Maturite             │ Tres mature (2004)     │ Recent (2020)          │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Communaute           │ Enorme                 │ En croissance rapide   │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Recommandation       │ Projets existants      │ Nouveaux projets       │
└──────────────────────┴────────────────────────┴────────────────────────┘
```

> [!tip] Conseil
> Pour un **nouveau projet**, choisissez Playwright. L'auto-attente elimine la majorite des tests "flaky" (qui echouent aleatoirement), et l'API est beaucoup plus concise.

---

## 9. Organisation des Tests

### Structure de dossiers

```
mon_projet/
├── src/
│   ├── app.py
│   ├── models.py
│   └── services.py
├── tests/
│   ├── conftest.py              # Fixtures globales
│   ├── unit/                    # Tests unitaires
│   │   ├── conftest.py
│   │   ├── test_models.py
│   │   └── test_services.py
│   ├── integration/             # Tests d'integration
│   │   ├── conftest.py          # Fixtures DB, client API
│   │   ├── test_api_users.py
│   │   └── test_api_posts.py
│   └── e2e/                     # Tests end-to-end
│       ├── conftest.py          # Fixtures navigateur
│       ├── test_login_flow.py
│       └── test_registration.py
├── pytest.ini
└── pyproject.toml
```

### Configuration pytest

```ini
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["tests"]
markers = [
    "unit: tests unitaires",
    "integration: tests d'integration",
    "e2e: tests end-to-end",
    "slow: tests lents",
]
addopts = "-v --tb=short"

# Lancer par categorie :
# pytest -m unit
# pytest -m integration
# pytest -m "not e2e"       (tout sauf E2E)
# pytest -m "not slow"      (tout sauf les lents)
```

### Conventions de nommage

```
┌──────────────────────┬──────────────────────────────────────┐
│ Convention           │ Exemple                              │
├──────────────────────┼──────────────────────────────────────┤
│ Fichier              │ test_users.py                        │
│ Fonction             │ test_create_user_with_valid_data     │
│ Classe               │ TestUserCreation                     │
│ Fixture              │ sample_user, db_session              │
│ Conftest             │ tests/conftest.py (un par niveau)    │
├──────────────────────┼──────────────────────────────────────┤
│ Nommage descriptif   │ test_login_fails_with_wrong_password │
│ (pas)                │ test_login_1, test_login_2           │
└──────────────────────┴──────────────────────────────────────┘
```

> [!info] Noms descriptifs
> Un bon nom de test doit decrire **ce qu'il teste** et **le resultat attendu**. Quand un test echoue, le nom seul doit suffire a comprendre ce qui ne va pas :
> - `test_divide_by_zero_raises_value_error` (clair)
> - `test_divide_3` (pas clair du tout)

---

## 10. Couverture de Code (coverage.py)

### Installation et utilisation

```bash
# Installer
pip install coverage
# Ou avec le plugin pytest
pip install pytest-cov
```

### Lancer les rapports

```bash
# Avec pytest-cov
pytest --cov=src --cov-report=term-missing

# Rapport HTML interactif
pytest --cov=src --cov-report=html

# Branch coverage (recommande)
pytest --cov=src --cov-branch --cov-report=term-missing
```

### Branch coverage

La couverture de branches est plus precise que la couverture de lignes :

```python
def categorize(score):
    if score >= 90:
        return "A"
    elif score >= 70:
        return "B"
    else:
        return "C"

# Couverture lignes : test_categorize(95) couvre 3 lignes sur 6 (50%)
# Couverture branches : couvre 1 branche sur 3 (33%)
# Il faut tester les 3 cas pour avoir 100% de branch coverage
```

### Configuration

```ini
# pyproject.toml
[tool.coverage.run]
source = ["src"]
branch = true
omit = [
    "*/tests/*",
    "*/migrations/*",
    "*/__pycache__/*",
    "*/venv/*",
]

[tool.coverage.report]
show_missing = true
fail_under = 80           # Echoue si < 80% de couverture
exclude_lines = [
    "pragma: no cover",
    "if __name__ == .__main__.",
    "raise NotImplementedError",
    "pass",
]

[tool.coverage.html]
directory = "htmlcov"
```

### Interpreter les resultats

```
---------- coverage ----------
Name                    Stmts   Miss Branch BrPart  Cover   Missing
-------------------------------------------------------------------
src/app.py                 45      3     12      2    91%   67-69
src/models.py              30      0      6      0   100%
src/services.py            60     12     20      5    78%   45-52, 80-85
-------------------------------------------------------------------
TOTAL                     135     15     38      7    87%

Stmts   = Nombre total de lignes executables
Miss    = Lignes non executees par les tests
Branch  = Nombre total de branches (if/else)
BrPart  = Branches partiellement couvertes
Cover   = Pourcentage de couverture
Missing = Numeros des lignes non couvertes
```

### Badges de couverture

```bash
# Generer un badge SVG (pour le README GitHub)
pip install coverage-badge
coverage-badge -o coverage.svg
```

---

## 11. Integration CI/CD (GitHub Actions)

### Configuration de base

```yaml
# .github/workflows/tests.yml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ["3.11", "3.12"]

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements*.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt

      - name: Run unit tests
        run: pytest tests/unit -v --tb=short

      - name: Run integration tests
        run: pytest tests/integration -v --tb=short

      - name: Run coverage
        run: |
          pytest --cov=src --cov-report=xml --cov-report=term-missing
          # Echoue si la couverture est < 80%

      - name: Upload coverage report
        uses: codecov/codecov-action@v4
        with:
          file: coverage.xml
```

### Tests avec services (PostgreSQL)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    env:
      DATABASE_URL: postgresql://test:test@localhost:5432/test_db

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: pytest tests/ -v --cov=src
```

---

## 12. Bonnes Pratiques

### Ce qu'il faut tester (et ne pas tester)

```
┌────────────────────────────┬──────────────────────────────────┐
│ TESTER                     │ NE PAS TESTER                    │
├────────────────────────────┼──────────────────────────────────┤
│ Le comportement            │ L'implementation                 │
│ (quoi, pas comment)        │ (comment c'est code)             │
├────────────────────────────┼──────────────────────────────────┤
│ Les cas limites            │ Les getters/setters triviaux     │
│ (vide, null, max, negatif) │                                  │
├────────────────────────────┼──────────────────────────────────┤
│ Les cas d'erreur           │ Le framework lui-meme            │
│ (exception, timeout)       │ (Flask/Django gere les routes)   │
├────────────────────────────┼──────────────────────────────────┤
│ Les regles metier          │ Les librairies tierces           │
│ (validation, calculs)      │ (requests, SQLAlchemy)           │
├────────────────────────────┼──────────────────────────────────┤
│ Les contrats d'API         │ Les details CSS/HTML             │
│ (status codes, schemas)    │ (sauf E2E critiques)             │
└────────────────────────────┴──────────────────────────────────┘
```

### Les principes FIRST

```
F - Fast      : les tests doivent etre rapides (quelques secondes max)
I - Isolated  : chaque test est independant (pas d'ordre, pas d'etat partage)
R - Repeatable: memes resultats a chaque execution (pas de random, pas de date)
S - Self-validating: le test dit clairement PASS ou FAIL (pas de verification manuelle)
T - Timely    : ecrits en meme temps que le code (ou avant, en TDD)
```

> [!warning] Tests fragiles ("flaky")
> Un test flaky est un test qui echoue aleatoirement. Causes courantes :
> - Dependance a l'heure/date (`datetime.now()` -> mocker !)
> - Dependance a l'ordre des tests (etat partage)
> - Attentes reseau (timeouts trop courts)
> - Tests E2E avec des animations CSS
>
> Solution : isoler, mocker les dependances externes, utiliser des attentes explicites.

### Noms significatifs

```python
# MAUVAIS : on ne sait pas ce que ca teste
def test_user_1():
    ...

def test_edge_case():
    ...

# BON : le nom decrit le scenario et le resultat attendu
def test_create_user_with_duplicate_email_returns_409():
    ...

def test_login_with_expired_token_returns_401():
    ...

def test_empty_cart_total_is_zero():
    ...
```

### Garder les tests rapides

```python
# LENT : chaque test cree une base complete
def test_query(self):
    db = create_full_database()  # 5 secondes
    result = db.query(...)

# RAPIDE : fixture avec scope "session" + rollback
@pytest.fixture(scope="session")
def db():
    return create_database_once()

@pytest.fixture
def session(db):
    s = db.create_session()
    yield s
    s.rollback()  # Rapide, pas de recreation
```

---

## Carte Mentale

```
                     Tests Integration et E2E
                              │
        ┌──────────┬──────────┼──────────┬──────────┐
        │          │          │          │          │
   Integration   E2E      Coverage    CI/CD    Bonnes
        │          │          │          │     Pratiques
   ┌────┤     ┌────┤     ┌────┤     ┌────┤        │
   │    │     │    │     │    │     │    │    ┌────┤
  Flask │  Selenium│  --cov   │  GitHub │  FIRST  │
  test  │     │    │     │    │  Actions│     │    │
  client│  Playwright  branch│     │    │   Noms  │
   │    │     │    │     │    │  matrix │ descriptifs
 FastAPI│  codegen │   html   │     │    │     │
 TestClient   │    │  report  │  cache  │  Pas de │
   │    │  headless│     │    │     │    │  flaky  │
 conftest│  auto-  │   80%    │ services│     │
 .py    │  wait   │   cible  │  (DB)   │  Tester │
   │    │         │          │         │  le     │
  DB    │  screenshot        │ coverage│  compor-│
  test  │         │          │  upload │  tement │
        │    locators        │         │
     fixtures     │          │
     (session)  waits
                  │
          explicit > implicit
```

---

## Exercices

### Exercice 1 : Tests d'integration API

Creez une API FastAPI de gestion de taches (todo) avec : `POST /tasks`, `GET /tasks`, `GET /tasks/{id}`, `PUT /tasks/{id}`, `DELETE /tasks/{id}`. Ecrivez des tests d'integration complets avec le `TestClient`, couvrant tous les status codes (200, 201, 204, 400, 404). Utilisez des fixtures pour reinitialiser les donnees entre chaque test.

### Exercice 2 : Tests avec base de donnees

Reprenez l'API de l'exercice 1 avec SQLAlchemy (SQLite en memoire). Ecrivez des tests d'integration qui utilisent une vraie base de donnees. Configurez un `conftest.py` avec les fixtures `engine`, `db_session` et `client`. Verifiez que les donnees sont bien persistees et que le rollback fonctionne entre les tests.

### Exercice 3 : CI/CD Pipeline

Ecrivez un fichier `.github/workflows/tests.yml` qui :
1. Lance les tests unitaires et d'integration sur Python 3.11 et 3.12
2. Genere un rapport de couverture
3. Echoue si la couverture est inferieure a 75%
4. Cache les dependances pip entre les executions

---

## Liens

- [[01 - Tests Unitaires et TDD]] - Tests unitaires et TDD avec pytest
- [[03 - Linting Formatting et Code Quality]] - Qualite de code et linting
- [[04 - CI-CD avec GitHub Actions]] - Pipelines CI/CD complets
