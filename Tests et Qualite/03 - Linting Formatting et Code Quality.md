# Linting, Formatting et Code Quality

## Pourquoi la qualite du code est importante ?

La qualite du code n'est pas un luxe, c'est une **necessite**. Un code propre est un code qui survit dans le temps.

> [!tip] Analogie
> Imagine une **cuisine de restaurant** :
> - Un chef qui range ses outils et nettoie au fur et a mesure → il est **efficace**, retrouve tout, et ne fait pas d'erreurs
> - Un chef qui laisse tout en vrac → il perd du temps, fait des erreurs, et le prochain cuisinier ne comprend rien
>
> Le code, c'est pareil. Tu ne codes pas pour toi aujourd'hui. Tu codes pour **toi dans 6 mois** et pour **ton equipe**.

### Les 3 piliers de la qualite du code

| Pilier | Probleme resolu | Outil |
|---|---|---|
| **Consistance** | "Chaque fichier a un style different" | Formatters (Black, Prettier) |
| **Lisibilite** | "Je ne comprends pas ce code" | Linters (pylint, ESLint) |
| **Correction** | "Ce code compile mais contient des bugs" | Type checkers (mypy, TypeScript) |

---

## Linting vs Formatting

> [!warning] LA distinction fondamentale
> Ce sont deux choses **differentes** et **complementaires**. Ne les confonds pas.

| Aspect | Linting | Formatting |
|---|---|---|
| **But** | Trouver des **erreurs** et problemes de style | **Reformater** automatiquement le code |
| **Ce qu'il fait** | Signale les problemes, tu dois corriger | Reecrit le fichier avec le bon style |
| **Exemples** | Variable inutilisee, import manquant, complexite | Indentation, espaces, guillemets, sauts de ligne |
| **Outils Python** | pylint, flake8, mypy | Black, isort |
| **Outils JS** | ESLint | Prettier |
| **Outils C** | Betty, cppcheck | clang-format |

```
Linting (analyse)                    Formatting (correction)
┌─────────────────────┐              ┌─────────────────────┐
│  Code source        │              │  Code source        │
│         │           │              │         │           │
│         v           │              │         v           │
│  ┌─────────────┐    │              │  ┌─────────────┐    │
│  │  Analyseur  │    │              │  │  Formateur  │    │
│  └──────┬──────┘    │              │  └──────┬──────┘    │
│         │           │              │         │           │
│         v           │              │         v           │
│  Liste d'erreurs    │              │  Code reformate     │
│  (a corriger)       │              │  (automatiquement)  │
└─────────────────────┘              └─────────────────────┘
```

---

## Outils Python

### pylint - Le linter le plus complet

**pylint** analyse ton code Python et attribue une **note sur 10**.

```bash
# Installation
pip install pylint

# Utilisation
pylint mon_module.py
pylint src/

# Generer un fichier de configuration
pylint --generate-rcfile > .pylintrc
```

> [!example] Exemple de sortie pylint
> ```
> ************* Module calculator
> src/calculator.py:1:0: C0114: Missing module docstring (missing-module-docstring)
> src/calculator.py:3:0: C0116: Missing function docstring (missing-function-docstring)
> src/calculator.py:7:4: W0612: Unused variable 'result' (unused-variable)
> src/calculator.py:12:0: C0301: Line too long (105/100) (line-too-long)
>
> Your code has been rated at 6.25/10
> ```

#### Configuration `.pylintrc`

```ini
[MAIN]
# Nombre de processus paralleles
jobs=4

[MESSAGES CONTROL]
# Desactiver des regles specifiques
disable=
    C0114,  # missing-module-docstring
    C0115,  # missing-class-docstring
    R0903,  # too-few-public-methods

[FORMAT]
# Longueur maximale d'une ligne
max-line-length=88

# Nombre max d'arguments d'une fonction
max-args=6

[BASIC]
# Convention de nommage des variables
variable-rgx=[a-z_][a-z0-9_]{0,30}$
```

#### Desactiver une regle inline

```python
# Desactiver pour une ligne
x = 42  # pylint: disable=invalid-name

# Desactiver pour un bloc
# pylint: disable=missing-docstring
def my_function():
    pass
# pylint: enable=missing-docstring

# Desactiver pour tout le fichier (en haut du fichier)
# pylint: disable=line-too-long
```

---

### flake8 - Le linter leger et rapide

**flake8** est plus simple que pylint : il combine `pycodestyle` (style PEP 8) + `pyflakes` (erreurs logiques) + `mccabe` (complexite).

```bash
# Installation
pip install flake8

# Utilisation
flake8 mon_fichier.py
flake8 src/ --max-line-length=88 --max-complexity=10

# Avec statistiques
flake8 src/ --count --show-source --statistics
```

#### Configuration `.flake8`

```ini
[flake8]
max-line-length = 88
max-complexity = 10
exclude =
    .git,
    __pycache__,
    venv,
    build
ignore =
    E203,   # whitespace before ':'  (conflit avec Black)
    W503,   # line break before binary operator
per-file-ignores =
    __init__.py:F401   # imported but unused dans les __init__
```

#### Plugins utiles flake8

```bash
pip install flake8-bugbear      # Detecte des bugs subtils
pip install flake8-comprehensions  # Ameliore les comprehensions
pip install flake8-docstrings   # Verifie les docstrings
```

---

### mypy - Le verificateur de types statique

**mypy** verifie que les **types** de ton code Python sont coherents, **sans l'executer**.

```bash
# Installation
pip install mypy

# Utilisation
mypy mon_fichier.py
mypy src/ --ignore-missing-imports

# Mode strict (plus de verifications)
mypy src/ --strict
```

> [!example] Exemple d'erreur detectee par mypy
> ```python
> def add(a: int, b: int) -> int:
>     return a + b
>
> result = add("hello", 5)  # mypy: error: Argument 1 has incompatible type "str"; expected "int"
> ```

#### Configuration dans `pyproject.toml`

```toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
ignore_missing_imports = true
disallow_untyped_defs = true

# Configuration par module
[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false
```

#### Ignorer des erreurs specifiques

```python
# Ignorer une ligne
result = some_untyped_function()  # type: ignore

# Ignorer un type d'erreur specifique
x: int = some_function()  # type: ignore[assignment]
```

> [!info] Pourquoi utiliser mypy ?
> Python est un langage a typage dynamique. Les bugs de type ne sont decouverts qu'a l'execution. mypy les detecte **avant** l'execution, comme un compilateur le ferait pour C ou Java.

---

### Black - Le formateur opiniatre

**Black** est un formateur de code Python qui ne laisse **aucun choix** de style. Il reformate automatiquement tout le code de la meme maniere.

```bash
# Installation
pip install black

# Formatter un fichier
black mon_fichier.py

# Formatter un dossier
black src/

# Verifier sans modifier (mode check pour le CI)
black --check src/

# Voir les changements sans modifier (mode diff)
black --diff src/
```

> [!tip] Analogie
> Black, c'est comme un **uniforme scolaire** : plus personne ne debat de ce qu'il faut porter. Tout le monde a le meme style, fin de la discussion. Ca evite les guerres de style dans les code reviews.

#### Configuration dans `pyproject.toml`

```toml
[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'
extend-exclude = '''
/(
    migrations
    | venv
)/
'''
```

> [!example] Avant / Apres Black
> ```python
> # AVANT (style inconsistant)
> def my_function( x,y, z ):
>     result = {'key1': x,'key2':y,
>         'key3': z}
>     return result
>
> # APRES Black (style uniforme)
> def my_function(x, y, z):
>     result = {"key1": x, "key2": y, "key3": z}
>     return result
> ```

---

### isort - Le trieur d'imports

**isort** trie et organise automatiquement tes imports Python selon les conventions PEP 8.

```bash
# Installation
pip install isort

# Trier les imports
isort mon_fichier.py
isort src/

# Verifier sans modifier
isort --check-only src/

# Voir les changements
isort --diff src/
```

#### Configuration dans `pyproject.toml`

```toml
[tool.isort]
profile = "black"       # Compatible avec Black
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
line_length = 88
known_first_party = ["mon_projet"]
```

> [!example] Avant / Apres isort
> ```python
> # AVANT (imports en vrac)
> import os
> from mon_projet.utils import helper
> import sys
> from collections import defaultdict
> import json
> from flask import Flask, request
>
> # APRES isort (imports organises)
> import json
> import os
> import sys
> from collections import defaultdict
>
> from flask import Flask, request
>
> from mon_projet.utils import helper
> ```
> Les imports sont groupes : standard library → third party → local.

---

## Outils JavaScript

### ESLint - Le linter JavaScript/TypeScript

**ESLint** est LE linter standard pour JavaScript et TypeScript.

```bash
# Installation
npm install --save-dev eslint

# Initialisation interactive
npx eslint --init

# Utilisation
npx eslint src/
npx eslint src/ --ext .js,.jsx,.ts,.tsx

# Corriger automatiquement ce qui est corrigeable
npx eslint src/ --fix
```

#### Configuration `.eslintrc.json`

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true,
    "jest": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module",
    "ecmaFeatures": {
      "jsx": true
    }
  },
  "plugins": [
    "react",
    "@typescript-eslint"
  ],
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "error",
    "semi": ["error", "always"],
    "quotes": ["error", "double"],
    "indent": ["error", 2],
    "no-var": "error",
    "prefer-const": "error",
    "eqeqeq": ["error", "always"]
  }
}
```

> [!info] Niveaux de severite ESLint
> - `"off"` ou `0` : regle desactivee
> - `"warn"` ou `1` : avertissement (ne bloque pas le CI)
> - `"error"` ou `2` : erreur (bloque le CI)

#### Ignorer des fichiers ou des lignes

```javascript
// Ignorer la ligne suivante
// eslint-disable-next-line no-console
console.log("Debug");

// Ignorer un bloc
/* eslint-disable no-alert */
alert("Hello");
alert("World");
/* eslint-enable no-alert */
```

```
# .eslintignore
node_modules/
build/
dist/
*.min.js
```

---

### Prettier - Le formateur universel

**Prettier** formate automatiquement JavaScript, TypeScript, CSS, JSON, Markdown, HTML et bien plus.

```bash
# Installation
npm install --save-dev prettier

# Formatter
npx prettier --write "src/**/*.{js,jsx,ts,tsx,json,css}"

# Verifier sans modifier (pour le CI)
npx prettier --check "src/**/*.{js,jsx,ts,tsx,json,css}"
```

#### Configuration `.prettierrc`

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "always"
}
```

#### Integration ESLint + Prettier

```bash
# Installer les plugins de compatibilite
npm install --save-dev eslint-config-prettier eslint-plugin-prettier
```

```json
{
  "extends": [
    "eslint:recommended",
    "plugin:prettier/recommended"
  ]
}
```

> [!warning] Conflit ESLint / Prettier
> ESLint et Prettier peuvent avoir des regles **contradictoires** (ex: guillemets simples vs doubles). `eslint-config-prettier` desactive les regles ESLint qui sont en conflit avec Prettier. **Prettier gagne toujours** pour le formatage.

---

## Outils C

### Betty Linter (Holberton)

**Betty** est le linter utilise a Holberton School. Il verifie le respect du coding style Holberton.

```bash
# Installation
git clone https://github.com/hs-hq/Betty.git
cd Betty
sudo ./install.sh

# Utilisation
betty mon_fichier.c
betty mon_fichier.h
betty *.c *.h
```

> [!warning] Regles Betty importantes
> - Max **40 lignes** par fonction
> - Max **5 variables** par fonction
> - Max **80 caracteres** par ligne
> - Pas de `//` pour les commentaires (utiliser `/* */`)
> - Accolade ouvrante sur une **nouvelle ligne** pour les fonctions

### clang-format

**clang-format** est le formateur standard pour C/C++.

```bash
# Installation (Ubuntu)
sudo apt-get install clang-format

# Formatter un fichier
clang-format -i mon_fichier.c

# Verifier sans modifier
clang-format --dry-run -Werror mon_fichier.c

# Generer un fichier de configuration
clang-format -style=llvm -dump-config > .clang-format
```

#### Configuration `.clang-format`

```yaml
BasedOnStyle: LLVM
IndentWidth: 4
TabWidth: 4
UseTab: ForIndentation
BreakBeforeBraces: Allman
ColumnLimit: 80
AllowShortFunctionsOnASingleLine: None
AllowShortIfStatementsOnASingleLine: false
```

---

## Pre-commit Hooks

Les **pre-commit hooks** sont des scripts qui s'executent **automatiquement avant chaque commit**. Si le hook echoue, le commit est bloque.

> [!tip] Analogie
> C'est comme un **vigile a l'entree d'un club** : avant que ton code entre dans le repository, il doit passer le controle. Pas de code mal formate, pas d'erreurs de linting, pas de fichiers oublies.

### Installation et configuration

```bash
# Installation
pip install pre-commit

# Installer les hooks dans le repo
pre-commit install

# Lancer manuellement sur tous les fichiers
pre-commit run --all-files

# Mettre a jour les hooks
pre-commit autoupdate
```

### Configuration `.pre-commit-config.yaml`

```yaml
repos:
  # Hooks generiques (tout langage)
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace       # Supprime les espaces en fin de ligne
      - id: end-of-file-fixer         # Ajoute un saut de ligne en fin de fichier
      - id: check-yaml                # Verifie la syntaxe YAML
      - id: check-json                # Verifie la syntaxe JSON
      - id: check-added-large-files   # Empeche les gros fichiers
        args: ['--maxkb=500']
      - id: check-merge-conflict      # Detecte les marqueurs de conflit oublies

  # Black (Python formatter)
  - repo: https://github.com/psf/black
    rev: 24.3.0
    hooks:
      - id: black
        language_version: python3.11

  # isort (Python import sorter)
  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort
        args: ["--profile", "black"]

  # flake8 (Python linter)
  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
        args: ["--max-line-length=88"]

  # mypy (Python type checker)
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.9.0
    hooks:
      - id: mypy
        additional_dependencies: [types-requests]

  # ESLint (JavaScript linter)
  - repo: https://github.com/pre-commit/mirrors-eslint
    rev: v9.0.0
    hooks:
      - id: eslint
        files: \.[jt]sx?$
        types: [file]
```

### Fonctionnement

```
git commit -m "mon message"
         |
         v
┌────────────────────────────┐
│    PRE-COMMIT HOOKS        │
│                            │
│  [1] trailing-whitespace   │──── OK
│  [2] end-of-file-fixer     │──── OK (fixe automatiquement)
│  [3] black                 │──── FAIL (reformate le fichier)
│  [4] flake8                │──── Pas execute (black a echoue)
│                            │
│  RESULTAT : ECHEC          │
│  Le commit est BLOQUE      │
│                            │
│  → black a reformate les   │
│    fichiers automatiquement│
│  → git add + git commit    │
│    a nouveau               │
└────────────────────────────┘
```

> [!info] Comportement important
> Quand un hook comme **Black** echoue, c'est souvent parce qu'il a **modifie** les fichiers. Il suffit de refaire `git add` puis `git commit`. Le deuxieme essai passera car les fichiers sont maintenant bien formates.

---

## EditorConfig

**EditorConfig** garantit que tous les membres de l'equipe utilisent les memes **reglages d'editeur**, quel que soit leur IDE.

### Configuration `.editorconfig`

```ini
# Fichier racine
root = true

# Regles pour tous les fichiers
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 4

# Regles specifiques Python
[*.py]
indent_size = 4
max_line_length = 88

# Regles specifiques JavaScript/TypeScript
[*.{js,jsx,ts,tsx}]
indent_size = 2

# Regles specifiques YAML
[*.{yml,yaml}]
indent_size = 2

# Regles specifiques JSON
[*.json]
indent_size = 2

# Regles specifiques C
[*.{c,h}]
indent_style = tab
indent_size = 4

# Makefile (tabulations obligatoires)
[Makefile]
indent_style = tab
```

> [!info] Support EditorConfig
> La plupart des IDE modernes supportent EditorConfig nativement ou via un plugin :
> - **VS Code** : extension "EditorConfig for VS Code"
> - **JetBrains (PyCharm, WebStorm)** : support natif
> - **Vim** : plugin editorconfig-vim

---

## Integration avec le CI/CD

Ajouter les checks de qualite dans ton pipeline GitHub Actions :

```yaml
name: Code Quality

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  quality-python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Installer les outils
        run: pip install flake8 black isort mypy pylint

      - name: Flake8
        run: flake8 src/ --count --show-source --statistics

      - name: Black (check)
        run: black --check src/

      - name: isort (check)
        run: isort --check-only src/

      - name: mypy
        run: mypy src/ --ignore-missing-imports

  quality-javascript:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Installer les dependances
        run: npm ci

      - name: ESLint
        run: npx eslint src/ --ext .js,.jsx,.ts,.tsx

      - name: Prettier (check)
        run: npx prettier --check "src/**/*.{js,jsx,ts,tsx,json,css}"
```

> [!warning] Pre-commit local + CI distant = double securite
> - **Pre-commit** empeche le developpeur de committer du code mal formate
> - **CI** verifie que le pre-commit n'a pas ete contourne (`git commit --no-verify`)
> - Les deux sont necessaires pour une qualite maximale

---

## Integration IDE : VS Code

### Extensions essentielles

| Extension | Langage | Role |
|---|---|---|
| **Pylint** | Python | Linting en temps reel |
| **Black Formatter** | Python | Formatage automatique |
| **mypy** | Python | Verification de types |
| **ESLint** | JavaScript | Linting en temps reel |
| **Prettier** | JavaScript | Formatage automatique |
| **EditorConfig** | Tous | Reglages editeur uniformes |
| **Error Lens** | Tous | Affiche les erreurs inline |

### Configuration `settings.json`

```json
{
  // Python
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  },
  "python.linting.pylintEnabled": true,
  "python.linting.flake8Enabled": true,
  "mypy-type-checker.args": ["--ignore-missing-imports"],

  // JavaScript / TypeScript
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  },
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],

  // General
  "editor.formatOnSave": true,
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true
}
```

> [!tip] Analogie
> Configurer ton IDE avec les bons outils, c'est comme configurer **l'auto-correct sur ton telephone** : les erreurs sont corrigees au fur et a mesure que tu tapes. Tu n'as plus besoin d'y penser.

---

## Tableau comparatif des outils

| Outil | Langage | Type | Auto-fix | Config | Difficulte |
|---|---|---|---|---|---|
| **pylint** | Python | Linter | Non | `.pylintrc` | Avance |
| **flake8** | Python | Linter | Non | `.flake8` | Simple |
| **mypy** | Python | Type checker | Non | `pyproject.toml` | Intermediaire |
| **Black** | Python | Formatter | Oui | `pyproject.toml` | Simple |
| **isort** | Python | Formatter (imports) | Oui | `pyproject.toml` | Simple |
| **ESLint** | JavaScript | Linter | Partiel (`--fix`) | `.eslintrc.json` | Intermediaire |
| **Prettier** | JavaScript+ | Formatter | Oui | `.prettierrc` | Simple |
| **Betty** | C | Linter | Non | Aucune | Simple |
| **clang-format** | C/C++ | Formatter | Oui | `.clang-format` | Intermediaire |

### Stack recommandee par langage

```
Python :   flake8 (lint) + mypy (types) + Black (format) + isort (imports)
JavaScript : ESLint (lint) + Prettier (format)
C :        Betty (lint, Holberton) + clang-format (format)
Tous :     pre-commit + EditorConfig + CI/CD
```

---

## Carte Mentale

```
                     Linting, Formatting et Code Quality
                                    |
                ┌───────────────────┼───────────────────┐
                |                   |                    |
           LINTING            FORMATTING            OUTILS
          (analyse)          (correction)           (support)
                |                   |                    |
         ┌──────┼──────┐    ┌──────┼──────┐      ┌──────┼──────┐
         |      |      |    |      |      |      |      |      |
       Python  JS     C   Python  JS     C   pre-commit EditorConfig CI/CD
         |      |      |    |      |      |
       pylint ESLint Betty Black Prettier clang-format
       flake8              isort
       mypy
```

---

## Exercices

### Exercice 1 : Configuration Python complete

Configure un projet Python avec :
1. Un fichier `.flake8` (max 88 chars, ignore E203 et W503)
2. Un `pyproject.toml` avec Black (line-length 88) et isort (profile "black")
3. Lance `flake8`, `black --check` et `isort --check-only` sur un projet existant
4. Corrige les erreurs avec `black` et `isort`

### Exercice 2 : Configuration JavaScript complete

Configure un projet JavaScript avec :
1. Un `.eslintrc.json` avec les regles recommandees + `no-console: warn` + `semi: error`
2. Un `.prettierrc` avec `semi: true`, `singleQuote: false`, `printWidth: 80`
3. Installe `eslint-config-prettier` pour eviter les conflits
4. Lance ESLint et Prettier sur le projet

### Exercice 3 : Pre-commit hooks

Configure un fichier `.pre-commit-config.yaml` qui :
1. Verifie les trailing whitespaces et la fin de fichier
2. Lance Black et isort sur les fichiers Python
3. Lance flake8 sur les fichiers Python
4. Teste que le hook bloque bien un commit avec du code mal formate

---

## Liens

- [[01 - Tests Unitaires et TDD]] - Les tests unitaires a combiner avec le linting pour une qualite maximale
- [[04 - CI-CD avec GitHub Actions]] - Integrer les outils de qualite dans un pipeline CI/CD