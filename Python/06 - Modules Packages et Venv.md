# Modules, Packages et Environnements Virtuels

## Qu'est-ce qu'un module ?

En Python, un **module** est simplement un fichier `.py`. Tout fichier Python est un module qui peut etre importe par d'autres fichiers. Les modules permettent d'organiser le code, de le reutiliser et d'eviter les repetitions.

C'est radicalement different du C ou l'inclusion de code passe par le preprocesseur (`#include`) qui fait du copier-coller textuel. En Python, `import` est une **operation runtime** qui execute le fichier importe et donne acces a ses objets via un namespace.

> [!tip] Analogie
> En C, `#include "utils.h"` c'est comme photocopier une page et la coller dans votre document : le preprocesseur insere litteralement le contenu du fichier. En Python, `import utils` c'est comme **referencer un livre dans une bibliotheque** : le livre existe independamment, vous y accedez par son nom (`utils.ma_fonction()`), et il n'est charge qu'une seule fois meme si plusieurs personnes le demandent.

---

## Les differentes formes d'import

### import classique

```python
# Importe le module entier
import math

# On accede aux fonctions via le namespace du module
resultat = math.sqrt(16)    # 4.0
pi = math.pi                # 3.141592653589793
```

### from ... import

```python
# Importe des elements specifiques dans le namespace courant
from math import sqrt, pi

resultat = sqrt(16)    # 4.0  (pas besoin de math.sqrt)
print(pi)              # 3.141592653589793
```

### from ... import * (deconseille)

```python
# Importe TOUT le contenu du module dans le namespace courant
from math import *

# Fonctionne mais DANGEREUX :
# - On ne sait plus d'ou viennent les fonctions
# - Risque de collision de noms
# - Impossible pour les outils d'analyse de detecter les erreurs
resultat = sqrt(16)
```

> [!warning] Evitez `from module import *`
> C'est acceptable dans un shell interactif pour experimenter, mais **jamais** dans du code de production. Si deux modules definissent une fonction `connect()`, le dernier import ecrase le premier silencieusement.

### import avec alias

```python
# Renommer un module pour plus de lisibilite ou brievete
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Conventions standard que tout le monde utilise
data = np.array([1, 2, 3])
df = pd.DataFrame(data)
```

### Imports relatifs (dans un package)

```python
# Depuis un fichier dans un package, on peut importer relativement
# mypackage/
#   __init__.py
#   module_a.py
#   module_b.py
#   sub/
#     __init__.py
#     module_c.py

# Dans module_b.py :
from . import module_a              # Import relatif (meme niveau)
from .module_a import ma_fonction   # Import specifique relatif
from .sub import module_c           # Import depuis un sous-package
from .. import autre_module         # Import parent (un niveau au dessus)
```

> [!info] Imports relatifs vs absolus
> - **Absolu** : `from mypackage.module_a import func` - toujours clair, recommande
> - **Relatif** : `from . import module_a` - plus court, mais peut etre confus
> - Les imports relatifs ne fonctionnent que **dans un package** (pas dans un script execute directement)

---

## `__name__` et `"__main__"`

### Le mecanisme

Chaque module Python a une variable speciale `__name__` :
- Si le fichier est **execute directement** : `__name__ == "__main__"`
- Si le fichier est **importe** : `__name__ == "nom_du_module"`

```python
# fichier : utils.py

def calculer(x):
    return x * 2

print(f"__name__ = {__name__}")

if __name__ == "__main__":
    # Ce bloc s'execute UNIQUEMENT si on lance : python utils.py
    print("Execution directe !")
    print(calculer(21))
```

```bash
# Execution directe
python utils.py
# __name__ = __main__
# Execution directe !
# 42

# Import depuis un autre fichier
python -c "import utils"
# __name__ = utils
# (le bloc if __name__ ne s'execute PAS)
```

### Pourquoi c'est important

Sans le guard, les effets de bord du module s'executent a chaque `import` (prints, connexions, tests). Avec le guard, seul le code dans le bloc `if` s'execute quand on lance le fichier directement.

> [!tip] Equivalent en C
> En Python, `if __name__ == "__main__"` joue le role de `int main(void)` en C : c'est le point d'entree quand le fichier est execute directement.

---

## Creer ses propres modules

### Module simple

```python
# fichier : geometry.py

"""Module de calculs geometriques."""

import math

PI = math.pi

def aire_cercle(rayon: float) -> float:
    """Calcule l'aire d'un cercle."""
    return PI * rayon ** 2

def perimetre_cercle(rayon: float) -> float:
    """Calcule le perimetre d'un cercle."""
    return 2 * PI * rayon

def aire_rectangle(longueur: float, largeur: float) -> float:
    """Calcule l'aire d'un rectangle."""
    return longueur * largeur
```

```python
# fichier : main.py
import geometry

print(geometry.aire_cercle(5))         # 78.53981633974483
print(geometry.perimetre_cercle(5))    # 31.41592653589793

from geometry import aire_rectangle
print(aire_rectangle(4, 3))            # 12
```

---

## Packages

### Qu'est-ce qu'un package ?

Un **package** est un dossier contenant un fichier `__init__.py` (peut etre vide). C'est un moyen d'organiser les modules en hierarchie.

```
mon_projet/
    main.py
    mypackage/
        __init__.py          # Fait de ce dossier un package
        module_a.py
        module_b.py
        sous_package/
            __init__.py      # Sous-package
            module_c.py
            module_d.py
```

### `__init__.py`

Le fichier `__init__.py` est execute quand on importe le package. Il peut etre vide ou contenir du code d'initialisation :

```python
# mypackage/__init__.py

# Re-exporter les elements importants pour simplifier les imports
from .module_a import ClasseA
from .module_b import fonction_utile

# Definir ce qui est exporte avec "from mypackage import *"
__all__ = ["ClasseA", "fonction_utile", "module_a", "module_b"]

# Version du package
__version__ = "1.0.0"
```

```python
# Grace au __init__.py, on peut faire :
from mypackage import ClasseA           # Direct, sans passer par module_a
from mypackage import fonction_utile

# Au lieu de :
from mypackage.module_a import ClasseA  # Plus explicite mais plus long
```

### `__all__`

La variable `__all__` controle ce qui est exporte par `from package import *` :

```python
# mypackage/module_a.py

__all__ = ["fonction_publique", "ClassePublique"]

def fonction_publique():
    return "accessible"

def _fonction_privee():
    return "convention privee"

def fonction_interne():
    return "pas dans __all__, pas exportee par import *"

class ClassePublique:
    pass
```

### Sous-packages

```python
from mypackage.sous_package import module_c
from mypackage.sous_package.module_d import MaClasse
```

> [!example] Structure d'un vrai package : `requests`
> ```
> requests/
>     __init__.py     # from .api import get, post, put, ...
>     api.py          # Fonctions principales
>     models.py       # Request, Response
>     sessions.py     # Session
>     exceptions.py   # ConnectionError, Timeout, ...
> ```

---

## La Bibliotheque Standard

Python est livre avec une bibliotheque standard extremement riche ("batteries included"). Voici les modules les plus utiles :

### os et sys

```python
import os, sys

os.getcwd()                             # Repertoire courant
os.listdir(".")                         # Lister les fichiers
os.makedirs("a/b/c", exist_ok=True)     # Creer des repertoires
os.environ["HOME"]                      # Variables d'environnement
os.getenv("API_KEY", "default")         # Avec valeur par defaut

sys.argv          # Arguments de la ligne de commande
sys.path          # Chemins de recherche des modules
sys.platform      # 'linux', 'darwin', 'win32'
sys.exit(1)       # Quitter avec un code de retour
```

### pathlib (recommande sur os.path)

```python
from pathlib import Path

p = Path("dossier") / "sous-dossier" / "fichier.txt"

p.name       # "fichier.txt"
p.stem       # "fichier"
p.suffix     # ".txt"
p.parent     # Path("dossier/sous-dossier")
p.exists()   # True/False

p.read_text()                          # Lire le contenu
p.write_text("hello")                  # Ecrire
p.mkdir(parents=True, exist_ok=True)   # Creer des dossiers

list(Path(".").rglob("*.py"))          # Tous les .py (recursif)
Path(__file__).parent.resolve()        # Dossier du script courant
```

> [!tip] Preferez `pathlib` a `os.path`
> `pathlib` est plus lisible, supporte l'operateur `/` pour construire des chemins. Standard moderne depuis Python 3.4.

### datetime

```python
from datetime import datetime, date, timedelta

maintenant = datetime.now()
noel = datetime(2024, 12, 25)

maintenant.strftime("%Y-%m-%d %H:%M:%S")        # Formatage
dt = datetime.strptime("2024-01-15", "%Y-%m-%d") # Parsing
demain = date.today() + timedelta(days=1)         # Arithmetique
```

### json

```python
import json

# Python dict -> JSON string
data = {"nom": "Alice", "age": 30, "scores": [95, 87, 92]}
json_str = json.dumps(data, indent=2, ensure_ascii=False)

# JSON string -> Python dict
data = json.loads('{"nom": "Alice", "age": 30}')

# Lire/ecrire depuis un fichier
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

with open("data.json") as f:
    data = json.load(f)
```

### re (expressions regulieres)

```python
import re

texte = "Mon email est alice@example.com et bob@test.fr"

re.search(r"\w+@\w+\.\w+", texte).group()   # "alice@example.com"
re.findall(r"\w+@\w+\.\w+", texte)           # ["alice@example.com", "bob@test.fr"]
re.sub(r"\w+@\w+\.\w+", "[EMAIL]", texte)    # "Mon email est [EMAIL] et [EMAIL]"

# Compilation (plus efficace si utilise plusieurs fois)
pattern = re.compile(r"(\d{2})/(\d{2})/(\d{4})")
```

### collections, itertools, functools

```python
from collections import Counter, defaultdict, deque

Counter("abracadabra")           # {'a': 5, 'b': 2, 'r': 2, ...}
Counter("abracadabra").most_common(2)  # [('a', 5), ('b', 2)]

scores = defaultdict(list)
scores["Alice"].append(95)       # Pas de KeyError si cle absente

file = deque(maxlen=5)           # File double (rapide des deux cotes)
```

```python
from itertools import chain, product, combinations
list(chain([1, 2], [3, 4]))              # [1, 2, 3, 4]
list(product("AB", "12"))                 # [('A','1'),('A','2'),('B','1'),('B','2')]
list(combinations("ABCD", 2))             # [('A','B'),('A','C'),...]
```

```python
from functools import lru_cache, partial

@lru_cache(maxsize=128)    # Memoization automatique
def fibonacci(n):
    if n < 2: return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(100)  # Instantane grace au cache
```

### typing

```python
from typing import Optional, TypeAlias, Any

# Annotations de type (pas de verification a l'execution)
def saluer(nom: str, age: int = 0) -> str:
    return f"Bonjour {nom}, {age} ans"

Matrice: TypeAlias = list[list[float]]

def chercher(nom: str) -> str | None:    # equivalent a Optional[str]
    return None
```

---

## pip : Le Gestionnaire de Paquets

### Commandes essentielles

```bash
# Installer un package
pip install requests
pip install flask==3.0.0           # Version exacte
pip install "django>=4.0,<5.0"     # Contrainte de version
pip install git+https://github.com/user/repo.git  # Depuis Git

# Desinstaller
pip uninstall requests

# Lister les packages installes
pip list

# Voir les details d'un package
pip show requests

# Exporter les dependances
pip freeze > requirements.txt

# Installer depuis un fichier requirements
pip install -r requirements.txt

# Mettre a jour un package
pip install --upgrade requests

# Installer en mode editable (dev)
pip install -e .
pip install -e ".[dev,test]"     # Avec extras
```

### Le fichier requirements.txt

```
# requirements.txt
flask==3.0.0
requests>=2.31.0,<3.0.0
sqlalchemy~=2.0.0          # Compatible ~= (2.0.x)
gunicorn>=21.0.0
```

> [!warning] `pip freeze` capture TOUTES les dependances
> `pip freeze` liste aussi les dependances transitives. Pour un fichier propre, listez manuellement vos dependances directes ou utilisez `pip-compile` (de `pip-tools`).

---

## Environnements Virtuels (venv)

### Pourquoi les environnements virtuels ?

> [!warning] Sans venv, c'est le chaos
> Si vous installez tous les packages globalement :
> - **Projet A** a besoin de `flask==2.3` et **Projet B** de `flask==3.0` → conflit !
> - Mettre a jour un package pour un projet peut **casser** un autre projet
> - Impossible de savoir quels packages appartiennent a quel projet
> - Les droits admin sont necessaires pour installer globalement sur certains systemes

Un **environnement virtuel** (virtual environment) est un dossier isole contenant sa propre copie de Python et ses propres packages. Chaque projet a son propre venv.

### Creer, activer, desactiver

```bash
python -m venv .venv                   # Creer le venv

# Activation selon l'OS
source .venv/bin/activate              # Linux / macOS
.venv\Scripts\Activate.ps1             # Windows (PowerShell)
source .venv/Scripts/activate          # Windows (Git Bash)

which python                           # Verifier (doit pointer vers .venv/)
deactivate                             # Quitter le venv
```

### Workflow complet

```bash
# Creer le projet et le venv
mkdir mon-projet && cd mon-projet
python -m venv .venv
source .venv/bin/activate          # Linux/macOS
pip install flask requests sqlalchemy
pip freeze > requirements.txt
deactivate                         # Quitter le venv

# Sur une autre machine (apres git clone)
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt    # Dependances exactes
```

> [!info] Ne commitez jamais le dossier .venv
> ```gitignore
> # .gitignore
> .venv/
> __pycache__/
> *.pyc
> .env
> ```

---

## Structure de Projet Recommandee

### Layout standard

```
mon-projet/
├── pyproject.toml          # Configuration du projet
├── .gitignore
├── requirements.txt
├── src/
│   └── mon_package/
│       ├── __init__.py
│       ├── main.py
│       ├── models.py
│       └── services/
│           ├── __init__.py
│           └── database.py
├── tests/
│   ├── conftest.py         # Fixtures pytest
│   ├── test_main.py
│   └── test_models.py
└── docs/
```

> [!tip] Le layout `src/`
> Placer le code dans `src/mon_package/` evite les imports accidentels du code local au lieu du package installe. Convention recommandee par la communaute Python.

### pyproject.toml

Le fichier `pyproject.toml` est le standard moderne pour configurer un projet Python. Il remplace `setup.py`, `setup.cfg`, et centralise la configuration de multiples outils :

```toml
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "mon-package"
version = "1.0.0"
description = "Description courte du projet"
requires-python = ">=3.10"

dependencies = [
    "flask>=3.0.0",
    "requests>=2.31.0",
    "sqlalchemy>=2.0.0",
]

[project.optional-dependencies]
dev = ["pytest>=7.0.0", "black", "ruff", "mypy"]

[project.scripts]
mon-cli = "mon_package.main:cli"

# ─── Configuration des outils ────────────

[tool.pytest.ini_options]
testpaths = ["tests"]

[tool.ruff]
line-length = 88
target-version = "py312"

[tool.mypy]
python_version = "3.12"
strict = true
```

```bash
pip install -e .          # Installer en mode editable
pip install -e ".[dev]"   # Avec les extras de dev
python -m build           # Construire le package
```

---

## Comparaison avec le C

| Aspect | C | Python |
|---|---|---|
| **Inclusion** | `#include` (copier-coller textuel) | `import` (chargement runtime) |
| **Declaration** | Header `.h` + `.c` | Tout dans un `.py` |
| **Packages** | Aucun gestionnaire standard | `pip` + PyPI (400k+) |
| **Dependances** | `Makefile` manuels | `requirements.txt`, `pyproject.toml` |
| **Isolation** | Difficile | `venv` |
| **Standard lib** | Minimale (libc) | Tres riche ("batteries included") |

```c
// En C : header + implementation + compilation manuelle
// utils.h
#ifndef UTILS_H
#define UTILS_H
double aire_cercle(double rayon);
#endif

// utils.c
#include "utils.h"
#include <math.h>
double aire_cercle(double rayon) { return M_PI * rayon * rayon; }

// Compilation : gcc -c utils.c -o utils.o -lm && gcc utils.o main.o -o programme -lm
```

```python
# En Python : import direct, pas de compilation
from utils import aire_cercle
print(f"Aire : {aire_cercle(5.0)}")   # python main.py
```

> [!info] En C, tout est manuel
> Pas de `pip install`, pas de venv. Des outils comme **vcpkg** ou **Conan** tentent de combler ce manque, mais restent plus complexes que pip.

---

## Carte Mentale

```
              Modules, Packages et Venv
                        │
        ┌───────────┬───┴───┬───────────┐
        │           │       │           │
     Modules    Packages   pip        venv
        │           │       │           │
   ┌────┤      ┌────┤    ┌──┤      ┌────┤
   │    │      │    │    │  │      │    │
 import │  __init__  │ install │  create
   │    │      │    │    │  │      │    │
  from  │  __all__   │ freeze  │ activate
   │    │      │    │    │  │      │    │
  as    │   sub-   │ requirements deactivate
   │    │  packages  │    │  │      │
__name__ │           │ -e .  │  requirements
 main    │           │       │    .txt
         │           │       │
   stdlib│       pyproject   │
         │        .toml      │
    ┌────┤                   │
    │    │              structure
   os  pathlib           projet
    │    │                  │
  sys  json            ┌───┤
    │    │             │   │
datetime re          src/ tests/
    │    │
collections
  itertools
  functools
   typing
```

---

## Exercices

### Exercice 1 : Module et __name__

Creez un module `converter.py` avec des fonctions `celsius_to_fahrenheit()` et `fahrenheit_to_celsius()`. Ajoutez un bloc `if __name__ == "__main__"` qui teste les fonctions. Verifiez que les tests ne s'executent pas quand vous importez le module depuis un autre fichier.

### Exercice 2 : Package complet

Creez un package `mathtools/` avec :
- `__init__.py` qui exporte les fonctions principales
- `geometry.py` (aire_cercle, perimetre_cercle, aire_rectangle)
- `stats.py` (moyenne, mediane, ecart_type)
- `__all__` correctement defini
- Un `main.py` qui utilise le package avec differentes formes d'import

### Exercice 3 : Venv et requirements

1. Creez un nouveau dossier de projet avec un venv
2. Installez `requests`, `flask` et `python-dotenv`
3. Gelez les dependances dans `requirements.txt`
4. Supprimez le venv, recreez-le, et reinstallez depuis `requirements.txt`
5. Verifiez que tout fonctionne

### Exercice 4 : Structure de projet

Creez un mini-projet avec la structure `src/` recommandee, un `pyproject.toml` minimal, et un test pytest. Installez le projet en mode editable (`pip install -e .`) et verifiez que les imports fonctionnent depuis n'importe quel repertoire.

---

## Liens

- [[05 - Gestion des Erreurs et Fichiers]] - Exceptions, fichiers et context managers
- [[07 - Python Async et Concurrence]] - Programmation asynchrone et parallele
