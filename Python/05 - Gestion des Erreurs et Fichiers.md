# Gestion des Erreurs et Fichiers

## Qu'est-ce que la gestion des erreurs en Python ?

La gestion des erreurs est un aspect fondamental de tout programme robuste. En Python, les erreurs sont gerees par un systeme d'**exceptions** : quand une erreur survient, Python **leve** (raise) une exception qui peut etre **capturee** (caught) par le programme. Ce mecanisme est tres different de l'approche du C ou les erreurs sont gerees par des codes de retour et `errno`.

Ce cours couvre egalement les **fichiers** et l'**entree/sortie**, car la gestion des erreurs est indissociable des operations d'I/O : tout acces fichier peut echouer, et les context managers (`with`) sont essentiels pour garantir la bonne fermeture des ressources.

> [!tip] Analogie
> En C, la gestion d'erreur c'est comme conduire une voiture sans airbag : vous devez verifier chaque code de retour manuellement (`if (ret == -1) { perror("..."); }`), et si vous oubliez, le programme plante silencieusement. En Python, les exceptions sont comme des airbags : si quelque chose va mal, l'exception se declenche automatiquement, remonte la pile d'appels, et vous donne un message clair. Vous pouvez la capturer (`try/except`) ou la laisser arreter le programme proprement.

---

## Exceptions : try / except / else / finally

### Syntaxe de base

```python
# En C :
# int fd = open("fichier.txt", O_RDONLY);
# if (fd == -1) {
#     perror("Erreur");
#     return -1;
# }

# En Python :
try:
    # Code qui peut lever une exception
    resultat = 10 / 0
except ZeroDivisionError:
    # Code execute si l'exception est levee
    print("Division par zero !")
```

### try / except / else / finally

```python
def diviser(a: float, b: float) -> float | None:
    """Divise a par b avec gestion d'erreur complete."""
    try:
        # Code susceptible de lever une exception
        resultat = a / b
    except ZeroDivisionError:
        # Execute si division par zero
        print("Erreur : division par zero")
        return None
    except TypeError as e:
        # Execute si mauvais type, capture l'exception dans 'e'
        print(f"Erreur de type : {e}")
        return None
    else:
        # Execute UNIQUEMENT si aucune exception n'est levee
        print(f"Resultat : {resultat}")
        return resultat
    finally:
        # TOUJOURS execute (exception ou pas)
        print("Operation terminee")


diviser(10, 3)
# Resultat : 3.3333333333333335
# Operation terminee

diviser(10, 0)
# Erreur : division par zero
# Operation terminee

diviser("10", 3)
# Erreur de type : unsupported operand type(s) for /: 'str' and 'int'
# Operation terminee
```

```
Flux d'execution de try/except/else/finally :

try:
    code_risque()
        |
        +-- Exception levee ?
        |       |
        |       +-- Oui --> except MatchingError:
        |       |               gestion_erreur()
        |       |                   |
        |       +-- Non --> else:
        |                       code_succes()
        |                           |
        +---------------------------+
        |
        v
    finally:
        nettoyage()  (TOUJOURS execute)
```

### Capturer plusieurs exceptions

```python
# Plusieurs types d'exception
try:
    valeur = int(input("Entrez un nombre : "))
    resultat = 100 / valeur
except ValueError:
    print("Ce n'est pas un nombre")
except ZeroDivisionError:
    print("Division par zero")

# Grouper des exceptions
try:
    # ...
    pass
except (ValueError, TypeError, KeyError) as e:
    print(f"Erreur : {e}")

# Capturer TOUTE exception (deconseille en general)
try:
    # ...
    pass
except Exception as e:
    print(f"Erreur inattendue : {type(e).__name__}: {e}")
```

> [!warning] Ne jamais capturer `except:` sans type
> `except:` (sans type) capture TOUTES les exceptions, y compris `KeyboardInterrupt` et `SystemExit`. Utilisez au minimum `except Exception:` pour laisser passer les exceptions systeme.

---

## Hierarchie des Exceptions

Python organise les exceptions en une hierarchie de classes. Comprendre cette hierarchie est essentiel pour capturer les bonnes exceptions.

```
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- ArithmeticError
      |    +-- ZeroDivisionError
      |    +-- OverflowError
      |    +-- FloatingPointError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- MemoryError
      +-- NameError
      |    +-- UnboundLocalError
      +-- OSError
      |    +-- FileNotFoundError
      |    +-- FileExistsError
      |    +-- PermissionError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- TimeoutError
      +-- RuntimeError
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- TypeError
      +-- ValueError
      |    +-- UnicodeError
      +-- Warning
           +-- DeprecationWarning
           +-- FutureWarning
           +-- UserWarning
```

### Les exceptions les plus courantes

```python
# TypeError : mauvais type
len(42)  # TypeError: object of type 'int' has no len()

# ValueError : bonne type, mauvaise valeur
int("abc")  # ValueError: invalid literal for int()

# KeyError : cle absente dans un dictionnaire
d = {"a": 1}
d["b"]  # KeyError: 'b'

# IndexError : index hors limites
lst = [1, 2, 3]
lst[10]  # IndexError: list index out of range

# AttributeError : attribut inexistant
"hello".non_existant()  # AttributeError

# FileNotFoundError : fichier introuvable
open("inexistant.txt")  # FileNotFoundError

# NameError : variable non definie
print(variable_inconnue)  # NameError

# ImportError : module non trouve
import module_inexistant  # ModuleNotFoundError
```

> [!info] Correspondance avec le C
> | Python | C equivalent |
> |---|---|
> | `ZeroDivisionError` | Signal SIGFPE ou comportement indefini |
> | `FileNotFoundError` | `errno == ENOENT` apres `open()` |
> | `PermissionError` | `errno == EACCES` |
> | `MemoryError` | `malloc()` retourne `NULL` |
> | `RecursionError` | Stack overflow (segfault) |
> | `IndexError` | Acces hors limites (UB en C !) |

---

## Lever des Exceptions (raise)

```python
def valider_age(age: int) -> None:
    """Valide que l'age est dans un intervalle raisonnable."""
    if not isinstance(age, int):
        raise TypeError(f"L'age doit etre un entier, pas {type(age).__name__}")
    if age < 0:
        raise ValueError(f"L'age ne peut pas etre negatif : {age}")
    if age > 150:
        raise ValueError(f"L'age est trop grand : {age}")

try:
    valider_age(-5)
except ValueError as e:
    print(f"Validation echouee : {e}")

# Re-lever une exception apres traitement
try:
    # ...
    raise ValueError("erreur originale")
except ValueError:
    print("Traitement de l'erreur...")
    raise  # Re-leve la MEME exception (preserve le traceback)

# Chainer les exceptions (exception cause)
try:
    int("abc")
except ValueError as original:
    raise RuntimeError("Conversion echouee") from original
```

---

## Exceptions Personnalisees

```python
class ErreurApplication(Exception):
    """Classe de base pour les exceptions de l'application."""
    pass

class ErreurValidation(ErreurApplication):
    """Erreur de validation des donnees."""

    def __init__(self, champ: str, valeur, message: str):
        self.champ = champ
        self.valeur = valeur
        self.message = message
        super().__init__(f"Validation de '{champ}' echouee: {message} (valeur: {valeur})")

class ErreurAuthentification(ErreurApplication):
    """Erreur d'authentification."""

    def __init__(self, utilisateur: str):
        self.utilisateur = utilisateur
        super().__init__(f"Authentification echouee pour '{utilisateur}'")

class SoldeInsuffisant(ErreurApplication):
    """Le solde est insuffisant pour l'operation."""

    def __init__(self, solde: float, montant: float):
        self.solde = solde
        self.montant = montant
        self.deficit = montant - solde
        super().__init__(
            f"Solde insuffisant: {solde:.2f} EUR "
            f"(besoin de {montant:.2f} EUR, deficit de {self.deficit:.2f} EUR)"
        )


# Utilisation
class CompteBancaire:
    def __init__(self, titulaire: str, solde: float = 0.0):
        self.titulaire = titulaire
        self._solde = solde

    def retirer(self, montant: float) -> None:
        if montant <= 0:
            raise ErreurValidation("montant", montant, "doit etre positif")
        if montant > self._solde:
            raise SoldeInsuffisant(self._solde, montant)
        self._solde -= montant

try:
    compte = CompteBancaire("Alice", 100)
    compte.retirer(150)
except SoldeInsuffisant as e:
    print(f"Erreur: {e}")
    print(f"Deficit: {e.deficit:.2f} EUR")
except ErreurValidation as e:
    print(f"Validation: {e.champ} = {e.valeur}")
except ErreurApplication as e:
    print(f"Erreur application: {e}")
```

> [!tip] Analogie
> Les exceptions personnalisees sont comme des formulaires d'erreur specifiques dans une entreprise. Au lieu d'un simple "Erreur", vous avez un formulaire "Solde Insuffisant" qui contient les details : le solde actuel, le montant demande et le deficit. Cela permet au code appelant de reagir de maniere intelligente.

### Bonnes pratiques pour les exceptions personnalisees

```python
# 1. Creer une hierarchie coherente
class MonAppException(Exception):
    """Base de toutes les exceptions de l'application."""

class MonAppErreurReseau(MonAppException):
    """Erreurs reseau."""

class MonAppErreurBDD(MonAppException):
    """Erreurs base de donnees."""

# 2. Toujours heriter d'Exception (ou d'une sous-classe)
# Jamais de BaseException directement

# 3. Fournir un message clair et des attributs utiles
```

---

## EAFP vs LBYL

Deux philosophies de gestion d'erreur coexistent en programmation.

### LBYL : Look Before You Leap (Regarder avant de sauter)

C'est l'approche typique du **C** : verifier les conditions avant d'agir.

```python
# Style LBYL (style C)
def obtenir_valeur_lbyl(dico: dict, cle: str) -> str:
    if cle in dico:                    # Verification d'abord
        return dico[cle]
    else:
        return "defaut"

# Pour les fichiers
import os
if os.path.exists("fichier.txt"):     # Verification d'abord
    with open("fichier.txt") as f:
        contenu = f.read()
```

### EAFP : Easier to Ask Forgiveness than Permission (Plus facile de demander pardon)

C'est l'approche **pythonique** : essayer et gerer l'erreur si elle survient.

```python
# Style EAFP (style Python)
def obtenir_valeur_eafp(dico: dict, cle: str) -> str:
    try:                               # Essayer d'abord
        return dico[cle]
    except KeyError:
        return "defaut"

# Pour les fichiers
try:
    with open("fichier.txt") as f:     # Essayer d'abord
        contenu = f.read()
except FileNotFoundError:
    contenu = ""
```

> [!info] Pourquoi EAFP est prefere en Python ?
> 1. **Atomicite** : avec LBYL, la condition peut changer entre la verification et l'action (race condition)
> 2. **Performance** : si l'erreur est rare, `try/except` est plus rapide que les verifications
> 3. **Lisibilite** : le code est plus direct et "pythonique"
> 4. **Duck typing** : on ne verifie pas le type, on essaie d'utiliser l'objet

```
LBYL (style C) :
  "Le fichier existe-t-il ?"
  "Oui" -> lire le fichier
  "Non" -> gerer l'erreur

EAFP (style Python) :
  Essayer de lire le fichier
  Ca a marche ? -> continuer
  Exception ? -> gerer l'erreur
```

---

## Context Managers et l'instruction with

Le context manager est un pattern qui garantit le **nettoyage** des ressources. C'est l'equivalent Python des blocs `goto cleanup` en C.

### Pourquoi with est essentiel

```python
# MAUVAIS : si une exception survient, le fichier n'est pas ferme
f = open("data.txt", "r")
contenu = f.read()
# Si une exception se produit ici, f.close() n'est jamais appele !
f.close()

# MOYEN : try/finally
f = open("data.txt", "r")
try:
    contenu = f.read()
finally:
    f.close()

# BON : with (context manager)
with open("data.txt", "r") as f:
    contenu = f.read()
# f.close() est appele automatiquement, meme en cas d'exception
```

### Context manager avec contextlib

```python
from contextlib import contextmanager

@contextmanager
def ouvrir_connexion(hote: str):
    """Context manager simplifie avec un generateur."""
    print(f"Connexion a {hote}...")
    connexion = {"hote": hote, "active": True}
    try:
        yield connexion  # Le code du bloc 'with' s'execute ici
    except Exception as e:
        print(f"Erreur pendant la connexion: {e}")
        raise
    finally:
        connexion["active"] = False
        print(f"Deconnexion de {hote}")


with ouvrir_connexion("serveur.local") as conn:
    print(f"Utilisation de la connexion: {conn}")
```

### Plusieurs context managers

```python
# Python 3.10+ : parentheses pour les context managers multiples
with (
    open("input.txt", "r") as f_in,
    open("output.txt", "w") as f_out,
):
    for ligne in f_in:
        f_out.write(ligne.upper())

# Avant Python 3.10
with open("input.txt", "r") as f_in:
    with open("output.txt", "w") as f_out:
        for ligne in f_in:
            f_out.write(ligne.upper())
```

---

## Lecture et Ecriture de Fichiers

### Ouvrir un fichier : modes

```python
# Syntaxe : open(chemin, mode, encoding)

# Modes principaux :
# "r"  : lecture (defaut). Erreur si le fichier n'existe pas.
# "w"  : ecriture. Cree le fichier ou ECRASE le contenu existant.
# "a"  : ajout. Cree le fichier ou ajoute a la fin.
# "x"  : creation exclusive. Erreur si le fichier existe deja.

# Modificateurs :
# "b"  : mode binaire (bytes). Ex: "rb", "wb"
# "t"  : mode texte (defaut). Ex: "rt" == "r"
# "+"  : lecture ET ecriture. Ex: "r+", "w+"
```

> [!warning] Toujours specifier l'encoding
> En Python 3, le mode texte utilise l'encoding du systeme par defaut. Sur Windows, c'est souvent `cp1252`, pas `utf-8`. Specifiez toujours l'encoding :
> ```python
> with open("fichier.txt", "r", encoding="utf-8") as f:
>     contenu = f.read()
> ```

### Lecture

```python
# Lire tout le contenu
with open("data.txt", "r", encoding="utf-8") as f:
    contenu = f.read()         # Retourne une seule chaine
    print(contenu)

# Lire ligne par ligne (econome en memoire)
with open("data.txt", "r", encoding="utf-8") as f:
    for ligne in f:            # Iterateur : une ligne a la fois
        print(ligne, end="")   # end="" car la ligne contient deja \n

# Lire une seule ligne
with open("data.txt", "r", encoding="utf-8") as f:
    premiere_ligne = f.readline()       # Lit une ligne
    deuxieme_ligne = f.readline()       # Lit la suivante

# Lire toutes les lignes dans une liste
with open("data.txt", "r", encoding="utf-8") as f:
    lignes = f.readlines()     # Liste de toutes les lignes
    print(lignes)              # ["ligne 1\n", "ligne 2\n", ...]

# Lire les N premiers caracteres
with open("data.txt", "r", encoding="utf-8") as f:
    debut = f.read(100)        # Lit 100 caracteres
```

### Ecriture

```python
# Ecrire (ecrase le fichier existant)
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Premiere ligne\n")
    f.write("Deuxieme ligne\n")

# Ecrire plusieurs lignes
lignes = ["Ligne 1\n", "Ligne 2\n", "Ligne 3\n"]
with open("output.txt", "w", encoding="utf-8") as f:
    f.writelines(lignes)       # N'ajoute PAS de \n automatiquement

# Ajouter a la fin (append)
with open("log.txt", "a", encoding="utf-8") as f:
    f.write("Nouvelle entree de log\n")

# Utiliser print() pour ecrire dans un fichier
with open("output.txt", "w", encoding="utf-8") as f:
    print("Bonjour", file=f)
    print(f"Valeur: {42}", file=f)
```

### Comparaison avec le C

```c
// C : ouverture et lecture de fichier
#include <stdio.h>

int main() {
    FILE *f = fopen("data.txt", "r");
    if (f == NULL) {
        perror("Erreur ouverture");
        return 1;
    }

    char buffer[256];
    while (fgets(buffer, sizeof(buffer), f) != NULL) {
        printf("%s", buffer);
    }

    fclose(f);  // IMPORTANT : ne pas oublier !
    return 0;
}
```

```python
# Python : equivalent
with open("data.txt", "r", encoding="utf-8") as f:
    for ligne in f:
        print(ligne, end="")
# Fermeture automatique, gestion d'erreur native
```

> [!example] Tableau de correspondance C / Python pour les fichiers
> | C | Python | Description |
> |---|---|---|
> | `fopen("f", "r")` | `open("f", "r")` | Ouvrir en lecture |
> | `fopen("f", "w")` | `open("f", "w")` | Ouvrir en ecriture |
> | `fopen("f", "a")` | `open("f", "a")` | Ouvrir en ajout |
> | `fopen("f", "rb")` | `open("f", "rb")` | Ouvrir en binaire |
> | `fread(buf, 1, n, f)` | `f.read(n)` | Lire n octets/caracteres |
> | `fgets(buf, n, f)` | `f.readline()` | Lire une ligne |
> | `fprintf(f, "...")` | `f.write("...")` | Ecrire |
> | `fclose(f)` | `f.close()` ou `with` | Fermer |
> | `fseek(f, 0, SEEK_SET)` | `f.seek(0)` | Repositionner |
> | `ftell(f)` | `f.tell()` | Position actuelle |
> | `feof(f)` | Pas d'equivalent direct | Fin de fichier |

---

## pathlib : Gestion Moderne des Chemins

Le module `pathlib` (Python 3.4+) fournit une API orientee objet pour manipuler les chemins de fichiers. C'est l'alternative moderne a `os.path`.

```python
from pathlib import Path

# Creer un chemin
dossier = Path("documents")
fichier = Path("documents/notes/cours.txt")

# Chemin absolu
absolu = Path.cwd() / "documents" / "notes"
home = Path.home()

# Proprietes du chemin
p = Path("/home/user/documents/rapport.pdf")
print(p.name)      # "rapport.pdf"
print(p.stem)      # "rapport"
print(p.suffix)    # ".pdf"
print(p.parent)    # PosixPath("/home/user/documents")
print(p.parents[1]) # PosixPath("/home/user")
print(p.parts)     # ("/", "home", "user", "documents", "rapport.pdf")
```

### Operations courantes

```python
from pathlib import Path

# Verifier l'existence
p = Path("data.txt")
print(p.exists())      # True/False
print(p.is_file())     # True si c'est un fichier
print(p.is_dir())      # True si c'est un dossier

# Creer des dossiers
dossier = Path("nouveau/sous_dossier")
dossier.mkdir(parents=True, exist_ok=True)
# parents=True : cree les dossiers intermediaires
# exist_ok=True : pas d'erreur si le dossier existe deja

# Lire et ecrire (raccourcis)
p = Path("data.txt")
p.write_text("Contenu du fichier", encoding="utf-8")
contenu = p.read_text(encoding="utf-8")

p_bin = Path("data.bin")
p_bin.write_bytes(b"\x00\x01\x02")
octets = p_bin.read_bytes()

# Renommer / Deplacer
p.rename("nouveau_nom.txt")

# Supprimer
Path("fichier_temp.txt").unlink(missing_ok=True)  # missing_ok: pas d'erreur si absent
Path("dossier_vide").rmdir()  # Seulement si vide
```

### Parcourir des fichiers (glob)

```python
from pathlib import Path

dossier = Path(".")

# Lister les fichiers Python dans le dossier courant
for fichier in dossier.glob("*.py"):
    print(fichier)

# Recherche recursive (tous les sous-dossiers)
for fichier in dossier.rglob("*.py"):
    print(fichier)

# Lister tout le contenu d'un dossier
for element in dossier.iterdir():
    type_elem = "D" if element.is_dir() else "F"
    print(f"[{type_elem}] {element.name}")
```

> [!tip] Analogie
> `pathlib` est comme un GPS pour votre systeme de fichiers. Au lieu de manipuler des chaines de caracteres brutes (`os.path.join(a, b)`), vous utilisez l'operateur `/` (`Path(a) / b`). C'est plus lisible, plus sur et multiplateforme.

### Construction de chemins

```python
from pathlib import Path

# Avec l'operateur / (bien plus lisible que os.path.join)
base = Path("/home/user")
rapport = base / "documents" / "rapport.pdf"

# Changer l'extension
nouveau = rapport.with_suffix(".docx")
print(nouveau)  # /home/user/documents/rapport.docx

# Changer le nom
autre = rapport.with_name("notes.pdf")
print(autre)  # /home/user/documents/notes.pdf
```

---

## JSON : Lecture et Ecriture

JSON (JavaScript Object Notation) est le format d'echange de donnees le plus utilise. Python le gere nativement.

```python
import json

# Dictionnaire Python -> JSON (serialisation)
donnees = {
    "nom": "Alice",
    "age": 25,
    "notes": [15.5, 12.0, 18.0],
    "adresse": {
        "ville": "Paris",
        "code_postal": "75001"
    },
    "actif": True,
    "commentaire": None
}

# Convertir en chaine JSON
json_str = json.dumps(donnees, indent=2, ensure_ascii=False)
print(json_str)
# {
#   "nom": "Alice",
#   "age": 25,
#   "notes": [15.5, 12.0, 18.0],
#   "adresse": {
#     "ville": "Paris",
#     "code_postal": "75001"
#   },
#   "actif": true,
#   "commentaire": null
# }

# JSON -> dictionnaire Python (deserialisation)
recupere = json.loads(json_str)
print(recupere["nom"])  # "Alice"
```

### Fichiers JSON

```python
import json
from pathlib import Path

# Ecrire dans un fichier JSON
donnees = {"etudiants": [{"nom": "Alice", "note": 15.5}]}

with open("etudiants.json", "w", encoding="utf-8") as f:
    json.dump(donnees, f, indent=2, ensure_ascii=False)

# Lire un fichier JSON
with open("etudiants.json", "r", encoding="utf-8") as f:
    donnees = json.load(f)

print(donnees["etudiants"][0]["nom"])  # "Alice"
```

### Correspondance des types Python <-> JSON

| Python | JSON |
|---|---|
| `dict` | `object {}` |
| `list`, `tuple` | `array []` |
| `str` | `string ""` |
| `int`, `float` | `number` |
| `True` | `true` |
| `False` | `false` |
| `None` | `null` |

> [!warning] Types non serialisables
> JSON ne supporte pas nativement : `set`, `datetime`, `Path`, les classes personnalisees. Vous devez fournir un serialiseur custom :
> ```python
> import json
> from datetime import datetime
> 
> class MonEncodeur(json.JSONEncoder):
>     def default(self, obj):
>         if isinstance(obj, datetime):
>             return obj.isoformat()
>         if isinstance(obj, set):
>             return list(obj)
>         return super().default(obj)
> 
> json.dumps({"date": datetime.now()}, cls=MonEncodeur)
> ```

---

## CSV : Lecture et Ecriture

CSV (Comma-Separated Values) est un format tabulaire simple, tres utilise pour les donnees.

### csv.reader et csv.writer

```python
import csv

# Ecrire un fichier CSV
with open("notes.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerow(["Nom", "Matiere", "Note"])  # En-tete
    writer.writerow(["Alice", "Math", 15.5])
    writer.writerow(["Bob", "Math", 12.0])
    writer.writerow(["Alice", "Info", 18.0])

# Ecrire plusieurs lignes d'un coup
lignes = [
    ["Charlie", "Math", 14.0],
    ["Charlie", "Info", 16.5],
]
with open("notes.csv", "a", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerows(lignes)

# Lire un fichier CSV
with open("notes.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.reader(f)
    en_tete = next(reader)  # Premiere ligne = en-tete
    print(en_tete)  # ["Nom", "Matiere", "Note"]
    for ligne in reader:
        nom, matiere, note = ligne
        print(f"{nom} - {matiere}: {note}")
```

> [!info] `newline=""` dans open()
> Pour les fichiers CSV, specifiez toujours `newline=""` dans `open()`. Le module `csv` gere lui-meme les fins de ligne. Sans cela, vous risquez d'avoir des lignes vides supplementaires sur Windows.

### DictReader et DictWriter

```python
import csv

# DictWriter : ecrire avec des noms de colonnes
with open("contacts.csv", "w", newline="", encoding="utf-8") as f:
    champs = ["nom", "email", "telephone"]
    writer = csv.DictWriter(f, fieldnames=champs)
    writer.writeheader()
    writer.writerow({"nom": "Alice", "email": "alice@mail.com", "telephone": "0601020304"})
    writer.writerow({"nom": "Bob", "email": "bob@mail.com", "telephone": "0605060708"})

# DictReader : lire comme des dictionnaires
with open("contacts.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for ligne in reader:
        # Chaque ligne est un OrderedDict
        print(f"Nom: {ligne['nom']}, Email: {ligne['email']}")

# DictReader est ideal pour les fichiers avec en-tete
```

### CSV avec separateur personnalise

```python
import csv

# Fichier avec point-virgule (courant en France)
with open("donnees.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f, delimiter=";")
    writer.writerow(["Nom", "Prix", "Quantite"])
    writer.writerow(["Pomme", "1,50", 10])  # Virgule dans le prix

# Lire avec point-virgule
with open("donnees.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.reader(f, delimiter=";")
    for ligne in reader:
        print(ligne)
```

---

## Exemple Complet : Application de Gestion

Voici un exemple complet combinant tous les concepts du cours.

```python
"""
Gestionnaire de notes etudiant.
Demonstre : exceptions, fichiers, JSON, pathlib.
"""

import json
from pathlib import Path
from dataclasses import dataclass, field, asdict


# --- Exceptions personnalisees ---

class ErreurGestionNotes(Exception):
    """Classe de base pour les erreurs du gestionnaire."""

class EtudiantNonTrouve(ErreurGestionNotes):
    def __init__(self, nom: str):
        super().__init__(f"Etudiant non trouve: {nom}")
        self.nom = nom

class NoteInvalide(ErreurGestionNotes):
    def __init__(self, note: float):
        super().__init__(f"Note invalide: {note} (doit etre entre 0 et 20)")
        self.note = note


# --- Modeles de donnees ---

@dataclass
class Etudiant:
    nom: str
    notes: dict[str, float] = field(default_factory=dict)

    def ajouter_note(self, matiere: str, note: float) -> None:
        if not 0 <= note <= 20:
            raise NoteInvalide(note)
        self.notes[matiere] = note

    @property
    def moyenne(self) -> float | None:
        if not self.notes:
            return None
        return sum(self.notes.values()) / len(self.notes)


# --- Gestionnaire avec persistance ---

class GestionnaireNotes:
    def __init__(self, chemin_donnees: Path):
        self.chemin = chemin_donnees
        self.etudiants: dict[str, Etudiant] = {}
        self._charger()

    def _charger(self) -> None:
        """Charge les donnees depuis le fichier JSON."""
        try:
            with open(self.chemin, "r", encoding="utf-8") as f:
                donnees = json.load(f)
            for nom, info in donnees.items():
                self.etudiants[nom] = Etudiant(nom=nom, notes=info["notes"])
        except FileNotFoundError:
            self.etudiants = {}
        except json.JSONDecodeError as e:
            raise ErreurGestionNotes(f"Fichier JSON corrompu: {e}") from e

    def sauvegarder(self) -> None:
        """Sauvegarde les donnees dans le fichier JSON."""
        self.chemin.parent.mkdir(parents=True, exist_ok=True)
        donnees = {
            nom: asdict(etud) for nom, etud in self.etudiants.items()
        }
        with open(self.chemin, "w", encoding="utf-8") as f:
            json.dump(donnees, f, indent=2, ensure_ascii=False)

    def ajouter_etudiant(self, nom: str) -> Etudiant:
        if nom in self.etudiants:
            raise ErreurGestionNotes(f"L'etudiant '{nom}' existe deja")
        etudiant = Etudiant(nom=nom)
        self.etudiants[nom] = etudiant
        self.sauvegarder()
        return etudiant

    def obtenir(self, nom: str) -> Etudiant:
        try:
            return self.etudiants[nom]
        except KeyError:
            raise EtudiantNonTrouve(nom) from None

    def noter(self, nom: str, matiere: str, note: float) -> None:
        etudiant = self.obtenir(nom)
        etudiant.ajouter_note(matiere, note)
        self.sauvegarder()


def main() -> None:
    chemin = Path("donnees") / "notes.json"
    gestionnaire = GestionnaireNotes(chemin)

    try:
        gestionnaire.ajouter_etudiant("Alice")
        gestionnaire.noter("Alice", "Math", 15.5)
        gestionnaire.noter("Alice", "Info", 18.0)

        alice = gestionnaire.obtenir("Alice")
        print(f"Moyenne de {alice.nom}: {alice.moyenne:.1f}")

    except ErreurGestionNotes as e:
        print(f"Erreur: {e}")


if __name__ == "__main__":
    main()
```

---

## Carte Mentale ASCII

```
             GESTION DES ERREURS ET FICHIERS
                         |
         +-------+-------+-------+-------+
         |       |       |       |       |
     Exceptions Fichiers pathlib  JSON    CSV
         |       |       |       |       |
    +----+----+ open()  Path   dumps   reader
    |    |    | read() exists  loads   writer
   try  raise write() mkdir   dump    DictR.
  except custom modes  glob   load    DictW.
   else  hier. r/w/a  rglob
  finally EAFP  rb/wb  /operator
         LBYL  with

   Context Managers
   with statement
   __enter__/__exit__
   @contextmanager
```

---

## Exercices

### Exercice 1 : Analyseur de logs

Ecrivez un programme qui lit un fichier de logs (une entree par ligne, format : `[NIVEAU] TIMESTAMP - Message`) et produit :
- Le nombre d'entrees par niveau (INFO, WARNING, ERROR)
- Les 5 dernieres erreurs
- Un fichier resume JSON

Gerez les fichiers corrompus et les lignes malformees avec des exceptions appropriees.

### Exercice 2 : Convertisseur CSV <-> JSON

Creez un programme qui convertit un fichier CSV en JSON et vice-versa. Utilisez `pathlib` pour detecter le format d'entree (extension). Gerez les erreurs de format, les fichiers manquants et les encodings differents.

### Exercice 3 : Gestionnaire de configuration

Creez une classe `Configuration` qui :
- Charge des parametres depuis un fichier JSON
- Permet d'acceder aux parametres avec la notation par points (`config.database.host`)
- Sauvegarde automatiquement les modifications (context manager)
- Leve des exceptions personnalisees (`CleManquante`, `TypeInvalide`)
- Supporte des valeurs par defaut

### Exercice 4 : Sauvegarde de fichiers

Ecrivez un script qui :
- Prend un dossier source et un dossier de destination
- Copie tous les fichiers modifies depuis la derniere sauvegarde (comparer les dates)
- Utilise `pathlib` pour toutes les operations de chemin
- Ecrit un rapport de sauvegarde en JSON (fichiers copies, taille totale, duree)
- Gere les erreurs (permissions, espace disque, fichiers verrouilles)

---

## Liens

- [[01 - Introduction a Python]] - Bases du langage et premiers `try/except`
- [[02 - Structures de Donnees Python]] - Dictionnaires pour JSON
- [[03 - POO en Python]] - Classes pour les exceptions personnalisees
- [[04 - POO Avancee]] - Context managers et decorateurs
- [[09 - File IO et Appels Systeme]] - Comparaison avec l'I/O en C
