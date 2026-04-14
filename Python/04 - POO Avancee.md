# POO Avancee

## Qu'est-ce que la POO avancee ?

Ce cours explore les mecanismes avances de la programmation orientee objet en Python. Apres avoir vu les bases (classes, heritage, polymorphisme) dans la note precedente, nous allons maintenant aborder les outils qui permettent d'ecrire du code **modulaire**, **extensible** et **maintenable** : decorateurs, classes abstraites, descripteurs, design patterns et principes SOLID.

Ces concepts sont essentiels pour les projets de taille reelle et sont omni-presents dans les frameworks Python populaires (Django, Flask, FastAPI, SQLAlchemy).

> [!tip] Analogie
> Si la POO de base vous a appris a construire des murs et des toits (classes, heritage), la POO avancee vous apprend les techniques d'architecture : comment concevoir un batiment entier qui soit solide, extensible et facile a maintenir. Les decorateurs sont comme des couches de peinture (ajout de fonctionnalites sans modifier la structure), les classes abstraites sont des plans d'architecte (contrats que les sous-classes doivent respecter), et les design patterns sont des plans eprouves pour des problemes courants.

---

## Decorateurs (Decorators)

Un decorateur est une **fonction qui prend une fonction en argument et retourne une nouvelle fonction**. C'est un moyen elegant d'ajouter du comportement a des fonctions ou des classes existantes sans les modifier.

### Rappel : les fonctions sont des objets

```python
# En Python, les fonctions sont des objets de premiere classe
def saluer(nom):
    return f"Bonjour, {nom} !"

# On peut assigner une fonction a une variable
dire_bonjour = saluer
print(dire_bonjour("Alice"))  # "Bonjour, Alice !"

# On peut passer une fonction en argument
def appliquer(func, valeur):
    return func(valeur)

print(appliquer(saluer, "Bob"))  # "Bonjour, Bob !"

# On peut definir une fonction dans une fonction (closure)
def creer_multiplicateur(facteur):
    def multiplier(x):
        return x * facteur
    return multiplier

doubler = creer_multiplicateur(2)
tripler = creer_multiplicateur(3)
print(doubler(5))   # 10
print(tripler(5))   # 15
```

> [!info] Comparaison avec le C
> En C, les pointeurs de fonctions permettent de passer des fonctions en arguments (`int (*f)(int)`), mais il n'y a pas de closures ni de fonctions imbriquees (sauf avec les extensions GNU). Les decorateurs Python n'ont pas d'equivalent direct en C.

### Decorateur simple

```python
import time

def mesurer_temps(func):
    """Decorateur qui mesure le temps d'execution d'une fonction."""
    def wrapper(*args, **kwargs):
        debut = time.time()
        resultat = func(*args, **kwargs)
        fin = time.time()
        print(f"{func.__name__} a pris {fin - debut:.4f} secondes")
        return resultat
    return wrapper

# Application avec la syntaxe @
@mesurer_temps
def tri_lent(liste):
    """Trie une liste (pour demo)."""
    return sorted(liste)

# Equivalent a : tri_lent = mesurer_temps(tri_lent)
resultat = tri_lent([3, 1, 4, 1, 5, 9, 2, 6])
# "tri_lent a pris 0.0001 secondes"
```

```
Comment fonctionne un decorateur :

   @mesurer_temps
   def tri_lent(liste):
       ...

   est equivalent a :

   def tri_lent(liste):
       ...
   tri_lent = mesurer_temps(tri_lent)

   Quand on appelle tri_lent(liste) :

   tri_lent(liste)
       |
       v
   wrapper(liste)        <-- la fonction wrapper du decorateur
       |
       +-- debut = time.time()
       +-- resultat = func(liste)   <-- appelle la vraie tri_lent
       +-- fin = time.time()
       +-- print(duree)
       +-- return resultat
```

### @functools.wraps : preserver les metadonnees

```python
from functools import wraps

def mon_decorateur(func):
    @wraps(func)  # Preserve __name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        """Wrapper interne."""
        print(f"Appel de {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@mon_decorateur
def ma_fonction():
    """Ma docstring originale."""
    pass

# Sans @wraps :
# print(ma_fonction.__name__)  # "wrapper"
# print(ma_fonction.__doc__)   # "Wrapper interne."

# Avec @wraps :
print(ma_fonction.__name__)  # "ma_fonction"
print(ma_fonction.__doc__)   # "Ma docstring originale."
```

> [!warning] Toujours utiliser @wraps
> Sans `@wraps`, le decorateur ecrase les metadonnees de la fonction originale (`__name__`, `__doc__`, `__module__`). Cela cause des problemes de debogage et de documentation. Utilisez toujours `@wraps(func)`.

### Decorateur avec arguments

```python
from functools import wraps

def repeter(n: int):
    """Decorateur qui repete l'execution n fois."""
    def decorateur(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            resultats = []
            for _ in range(n):
                resultats.append(func(*args, **kwargs))
            return resultats
        return wrapper
    return decorateur

@repeter(3)
def dire_bonjour(nom):
    print(f"Bonjour, {nom} !")
    return nom

# Equivalent a : dire_bonjour = repeter(3)(dire_bonjour)
dire_bonjour("Alice")
# Bonjour, Alice !
# Bonjour, Alice !
# Bonjour, Alice !
```

```
Structure d'un decorateur avec arguments :

repeter(3)              <-- retourne le decorateur
    |
    v
decorateur(func)        <-- recoit la fonction originale
    |
    v
wrapper(*args, **kwargs) <-- remplace la fonction originale
```

### Decorateurs pratiques courants

```python
from functools import wraps

# 1. Cache / Memoisation
def cache(func):
    """Met en cache les resultats des appels precedents."""
    _cache = {}

    @wraps(func)
    def wrapper(*args):
        if args not in _cache:
            _cache[args] = func(*args)
        return _cache[args]
    return wrapper

@cache
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(100))  # Instantane grace au cache

# Note : Python fournit @functools.lru_cache pour cela
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci_v2(n: int) -> int:
    if n < 2:
        return n
    return fibonacci_v2(n - 1) + fibonacci_v2(n - 2)


# 2. Validation d'arguments
def valider_types(*types):
    """Verifie les types des arguments."""
    def decorateur(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for arg, type_attendu in zip(args, types):
                if not isinstance(arg, type_attendu):
                    raise TypeError(
                        f"Attendu {type_attendu.__name__}, "
                        f"recu {type(arg).__name__}"
                    )
            return func(*args, **kwargs)
        return wrapper
    return decorateur

@valider_types(str, int)
def creer_profil(nom: str, age: int) -> dict:
    return {"nom": nom, "age": age}

creer_profil("Alice", 25)  # OK
# creer_profil("Alice", "25")  # TypeError !


# 3. Retry (nouvelle tentative)
def retry(max_tentatives: int = 3, delai: float = 1.0):
    """Reessaie l'appel en cas d'exception."""
    def decorateur(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for tentative in range(max_tentatives):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if tentative == max_tentatives - 1:
                        raise
                    print(f"Tentative {tentative + 1} echouee: {e}")
                    import time
                    time.sleep(delai)
        return wrapper
    return decorateur

@retry(max_tentatives=3, delai=0.5)
def appel_api():
    """Simule un appel API instable."""
    import random
    if random.random() < 0.7:
        raise ConnectionError("Serveur indisponible")
    return {"status": "ok"}
```

### Decorateurs de classe

```python
from functools import wraps

# Decorateur qui s'applique a une classe
def singleton(cls):
    """Garantit qu'une seule instance de la classe existe."""
    instances = {}

    @wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return get_instance

@singleton
class Configuration:
    def __init__(self):
        self.parametres = {}

    def set(self, cle: str, valeur) -> None:
        self.parametres[cle] = valeur

    def get(self, cle: str, defaut=None):
        return self.parametres.get(cle, defaut)


c1 = Configuration()
c1.set("debug", True)

c2 = Configuration()
print(c2.get("debug"))  # True (c'est le MEME objet)
print(c1 is c2)          # True
```

---

## Classes Abstraites (ABC)

Les classes abstraites definissent des **interfaces** que les sous-classes doivent implementer. Elles ne peuvent pas etre instanciees directement.

```python
from abc import ABC, abstractmethod

class Forme(ABC):
    """Classe abstraite pour les formes geometriques."""

    @abstractmethod
    def aire(self) -> float:
        """Calcule l'aire de la forme."""
        ...

    @abstractmethod
    def perimetre(self) -> float:
        """Calcule le perimetre de la forme."""
        ...

    def description(self) -> str:
        """Methode concrete (avec implementation)."""
        return f"{self.__class__.__name__}: aire={self.aire():.2f}"


# Impossible d'instancier une classe abstraite
# f = Forme()  # TypeError: Can't instantiate abstract class

class Rectangle(Forme):
    def __init__(self, largeur: float, hauteur: float):
        self.largeur = largeur
        self.hauteur = hauteur

    def aire(self) -> float:
        return self.largeur * self.hauteur

    def perimetre(self) -> float:
        return 2 * (self.largeur + self.hauteur)


class Cercle(Forme):
    def __init__(self, rayon: float):
        self.rayon = rayon

    def aire(self) -> float:
        import math
        return math.pi * self.rayon ** 2

    def perimetre(self) -> float:
        import math
        return 2 * math.pi * self.rayon


# Si on oublie une methode abstraite :
# class Incomplet(Forme):
#     def aire(self):
#         return 0
#     # perimetre() manque !
# i = Incomplet()  # TypeError !

r = Rectangle(5, 3)
print(r.description())  # "Rectangle: aire=15.00"
```

> [!tip] Analogie
> Une classe abstraite est comme un **contrat** ou un **cahier des charges**. Elle dit : "Toute forme doit pouvoir calculer son aire et son perimetre." Elle ne dit pas **comment**, c'est aux classes concretes de decider. C'est similaire aux fichiers header `.h` en C qui declarent les prototypes de fonctions sans les implementer.

### Abstract Properties

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @property
    @abstractmethod
    def son(self) -> str:
        """Le son que fait l'animal."""
        ...

    @abstractmethod
    def se_deplacer(self) -> str:
        ...

class Chien(Animal):
    @property
    def son(self) -> str:
        return "Ouaf"

    def se_deplacer(self) -> str:
        return "Le chien court"
```

---

## Protocols (Structural Subtyping)

Les Protocols (Python 3.8+) definissent des interfaces basees sur la **structure** plutot que l'heritage. C'est du duck typing formalise.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Lisible(Protocol):
    """Tout objet qui a une methode lire() -> str."""
    def lire(self) -> str:
        ...

class Fichier:
    def lire(self) -> str:
        return "contenu du fichier"

class BaseDeDonnees:
    def lire(self) -> str:
        return "donnees BDD"

# Pas besoin d'heritage !
def traiter(source: Lisible) -> None:
    print(source.lire())

traiter(Fichier())        # OK
traiter(BaseDeDonnees())  # OK

# Verification a l'execution (grace a @runtime_checkable)
print(isinstance(Fichier(), Lisible))  # True
```

> [!info] ABC vs Protocol
> | Aspect | ABC | Protocol |
> |---|---|---|
> | Type | Heritage nominal | Sous-typage structurel |
> | Heritage requis | Oui (`class X(ABC)`) | Non |
> | Verification | A l'instanciation | Par les outils de type (mypy) |
> | Style | Java/C++ | Go interfaces, duck typing |
> | Quand utiliser | Contrat fort, hierarchie | Interfaces legeres, interoperabilite |

---

## Descripteurs

Les descripteurs sont le mecanisme sous-jacent des `@property`, `@classmethod` et `@staticmethod`. Ils controlent l'acces aux attributs.

```python
class Valide:
    """Descripteur qui valide qu'une valeur est dans un intervalle."""

    def __init__(self, minimum: float, maximum: float):
        self.minimum = minimum
        self.maximum = maximum

    def __set_name__(self, owner, name):
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, None)

    def __set__(self, obj, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f"{self.name} doit etre un nombre")
        if not self.minimum <= value <= self.maximum:
            raise ValueError(
                f"{self.name} doit etre entre {self.minimum} et {self.maximum}"
            )
        setattr(obj, self.private_name, value)


class Etudiant:
    age = Valide(16, 99)
    note = Valide(0, 20)

    def __init__(self, nom: str, age: int, note: float):
        self.nom = nom
        self.age = age    # Passe par Valide.__set__
        self.note = note  # Passe par Valide.__set__


e = Etudiant("Alice", 20, 15.5)  # OK
# e = Etudiant("Bob", 10, 15.5)  # ValueError: age doit etre entre 16 et 99
# e.note = 25                     # ValueError: note doit etre entre 0 et 20
```

---

## `__slots__`

Par defaut, Python stocke les attributs d'instance dans un dictionnaire (`__dict__`). `__slots__` optimise la memoire en declarant les attributs a l'avance.

```python
class PointNormal:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

class PointOptimise:
    __slots__ = ("x", "y")

    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y


p1 = PointNormal(3, 4)
p2 = PointOptimise(3, 4)

# PointNormal a un __dict__
print(p1.__dict__)  # {"x": 3, "y": 4}
p1.z = 5           # On peut ajouter des attributs dynamiquement

# PointOptimise n'a PAS de __dict__
# print(p2.__dict__)  # AttributeError
# p2.z = 5            # AttributeError: 'PointOptimise' has no attribute 'z'
```

> [!info] Quand utiliser `__slots__` ?
> Utilisez `__slots__` quand vous creez **beaucoup d'instances** (millions) d'une classe avec peu d'attributs. L'economie de memoire peut etre significative (30-50%). Inconvenient : pas de creation d'attributs dynamiques, l'heritage avec `__slots__` est delicat.

---

## Metaclasses (apercu)

Les metaclasses sont des "classes de classes". En Python, **tout est un objet**, y compris les classes elles-memes. La metaclasse par defaut est `type`.

```python
# type est la metaclasse par defaut
print(type(42))        # <class 'int'>
print(type(int))       # <class 'type'>
print(type(type))      # <class 'type'>

# On peut creer une classe avec type()
MaClasse = type("MaClasse", (object,), {"x": 42, "saluer": lambda self: "Bonjour"})
obj = MaClasse()
print(obj.x)        # 42
print(obj.saluer())  # "Bonjour"

# Metaclasse personnalisee (usage avance)
class AutoRepr(type):
    """Metaclasse qui ajoute automatiquement __repr__ aux classes."""
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if "__repr__" not in namespace:
            def auto_repr(self):
                attrs = ", ".join(
                    f"{k}={v!r}" for k, v in self.__dict__.items()
                )
                return f"{name}({attrs})"
            cls.__repr__ = auto_repr
        return cls

class Personne(metaclass=AutoRepr):
    def __init__(self, nom, age):
        self.nom = nom
        self.age = age

p = Personne("Alice", 25)
print(p)  # Personne(nom='Alice', age=25)
```

> [!warning] Metaclasses : a utiliser avec parcimonie
> Les metaclasses sont un outil puissant mais complexe. La plupart du temps, les decorateurs de classe ou les `__init_subclass__` suffisent. La regle : "Si vous vous demandez si vous avez besoin de metaclasses, vous n'en avez probablement pas besoin."

---

## Design Patterns

Les design patterns sont des **solutions eprouvees** a des problemes de conception recurrents. Voici les plus courants en Python.

### Singleton

Garantit qu'une classe n'a qu'une seule instance.

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, valeur: str = ""):
        # Attention : __init__ est appele a chaque fois
        if not hasattr(self, "_initialise"):
            self.valeur = valeur
            self._initialise = True


s1 = Singleton("premier")
s2 = Singleton("deuxieme")
print(s1 is s2)       # True
print(s1.valeur)      # "premier" (pas ecrase grace au guard)
```

### Factory (Fabrique)

Cree des objets sans specifier la classe exacte.

```python
from abc import ABC, abstractmethod

class Notification(ABC):
    @abstractmethod
    def envoyer(self, message: str) -> None:
        ...

class EmailNotification(Notification):
    def envoyer(self, message: str) -> None:
        print(f"[EMAIL] {message}")

class SMSNotification(Notification):
    def envoyer(self, message: str) -> None:
        print(f"[SMS] {message}")

class PushNotification(Notification):
    def envoyer(self, message: str) -> None:
        print(f"[PUSH] {message}")

class NotificationFactory:
    """Factory : cree la bonne notification selon le type."""

    _types: dict[str, type[Notification]] = {
        "email": EmailNotification,
        "sms": SMSNotification,
        "push": PushNotification,
    }

    @classmethod
    def creer(cls, type_notif: str) -> Notification:
        classe = cls._types.get(type_notif)
        if classe is None:
            raise ValueError(f"Type inconnu: {type_notif}")
        return classe()

    @classmethod
    def enregistrer(cls, nom: str, classe: type[Notification]) -> None:
        """Permet d'ajouter de nouveaux types sans modifier la factory."""
        cls._types[nom] = classe


# Utilisation
notif = NotificationFactory.creer("email")
notif.envoyer("Bienvenue !")  # [EMAIL] Bienvenue !
```

### Observer (Observateur)

Permet a des objets d'etre notifies quand un autre objet change d'etat.

```python
from abc import ABC, abstractmethod

class Observateur(ABC):
    @abstractmethod
    def notifier(self, evenement: str, donnees: dict) -> None:
        ...

class Sujet:
    """Objet observe."""

    def __init__(self):
        self._observateurs: list[Observateur] = []

    def abonner(self, obs: Observateur) -> None:
        self._observateurs.append(obs)

    def desabonner(self, obs: Observateur) -> None:
        self._observateurs.remove(obs)

    def _emettre(self, evenement: str, donnees: dict) -> None:
        for obs in self._observateurs:
            obs.notifier(evenement, donnees)


class Boutique(Sujet):
    def __init__(self):
        super().__init__()
        self._produits: dict[str, float] = {}

    def ajouter_produit(self, nom: str, prix: float) -> None:
        self._produits[nom] = prix
        self._emettre("nouveau_produit", {"nom": nom, "prix": prix})

    def modifier_prix(self, nom: str, prix: float) -> None:
        ancien = self._produits.get(nom)
        self._produits[nom] = prix
        self._emettre("prix_modifie", {
            "nom": nom, "ancien_prix": ancien, "nouveau_prix": prix
        })


class ClientEmail(Observateur):
    def __init__(self, email: str):
        self.email = email

    def notifier(self, evenement: str, donnees: dict) -> None:
        if evenement == "prix_modifie":
            print(f"[Email a {self.email}] Prix de {donnees['nom']} "
                  f"modifie: {donnees['ancien_prix']} -> {donnees['nouveau_prix']}")

class JournalLog(Observateur):
    def notifier(self, evenement: str, donnees: dict) -> None:
        print(f"[LOG] {evenement}: {donnees}")


# Utilisation
boutique = Boutique()
boutique.abonner(ClientEmail("alice@mail.com"))
boutique.abonner(JournalLog())

boutique.ajouter_produit("Laptop", 999.99)
# [LOG] nouveau_produit: {"nom": "Laptop", "prix": 999.99}

boutique.modifier_prix("Laptop", 899.99)
# [Email a alice@mail.com] Prix de Laptop modifie: 999.99 -> 899.99
# [LOG] prix_modifie: {"nom": "Laptop", ...}
```

### Strategy (Strategie)

Permet de changer l'algorithme utilise a l'execution.

```python
from abc import ABC, abstractmethod

class StrategieTri(ABC):
    @abstractmethod
    def trier(self, donnees: list) -> list:
        ...

class TriBulle(StrategieTri):
    def trier(self, donnees: list) -> list:
        arr = donnees.copy()
        n = len(arr)
        for i in range(n):
            for j in range(0, n - i - 1):
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]
        return arr

class TriRapide(StrategieTri):
    def trier(self, donnees: list) -> list:
        if len(donnees) <= 1:
            return donnees
        pivot = donnees[len(donnees) // 2]
        gauche = [x for x in donnees if x < pivot]
        milieu = [x for x in donnees if x == pivot]
        droite = [x for x in donnees if x > pivot]
        return self.trier(gauche) + milieu + self.trier(droite)

class TriPython(StrategieTri):
    def trier(self, donnees: list) -> list:
        return sorted(donnees)

class Trieur:
    def __init__(self, strategie: StrategieTri):
        self._strategie = strategie

    def changer_strategie(self, strategie: StrategieTri) -> None:
        self._strategie = strategie

    def executer(self, donnees: list) -> list:
        return self._strategie.trier(donnees)


# Utilisation
donnees = [64, 34, 25, 12, 22, 11, 90]
trieur = Trieur(TriBulle())
print(trieur.executer(donnees))  # [11, 12, 22, 25, 34, 64, 90]

trieur.changer_strategie(TriRapide())
print(trieur.executer(donnees))  # [11, 12, 22, 25, 34, 64, 90]
```

> [!tip] Analogie
> Le pattern Strategy, c'est comme un GPS qui propose plusieurs itineraires (le plus court, le plus rapide, sans autoroute). L'objectif est le meme (arriver a destination), mais l'algorithme change selon la strategie choisie.

---

## Principes SOLID

Les principes SOLID sont cinq regles de conception orientee objet.

### S - Single Responsibility Principle (Responsabilite unique)

Chaque classe ne doit avoir qu'**une seule raison de changer**.

```python
# MAUVAIS : la classe fait trop de choses
class Rapport:
    def __init__(self, donnees):
        self.donnees = donnees

    def calculer_statistiques(self):
        ...

    def generer_pdf(self):  # Responsabilite differente !
        ...

    def envoyer_email(self):  # Encore une autre !
        ...

# BON : chaque classe a une responsabilite
class Rapport:
    def __init__(self, donnees):
        self.donnees = donnees

    def calculer_statistiques(self):
        ...

class GenerateurPDF:
    def generer(self, rapport: Rapport):
        ...

class EnvoyeurEmail:
    def envoyer(self, destinataire: str, contenu: str):
        ...
```

### O - Open/Closed Principle (Ouvert/Ferme)

Les classes doivent etre **ouvertes a l'extension** mais **fermees a la modification**.

```python
# BON : on peut ajouter de nouveaux types sans modifier le code existant
from abc import ABC, abstractmethod

class Remise(ABC):
    @abstractmethod
    def calculer(self, prix: float) -> float:
        ...

class RemisePourcentage(Remise):
    def __init__(self, pourcentage: float):
        self.pourcentage = pourcentage

    def calculer(self, prix: float) -> float:
        return prix * (1 - self.pourcentage / 100)

class RemiseMontant(Remise):
    def __init__(self, montant: float):
        self.montant = montant

    def calculer(self, prix: float) -> float:
        return max(0, prix - self.montant)

# Ajout d'un nouveau type : pas besoin de modifier les classes existantes
class RemiseFidelite(Remise):
    def __init__(self, points: int):
        self.points = points

    def calculer(self, prix: float) -> float:
        reduction = min(self.points * 0.01, prix * 0.3)
        return prix - reduction
```

### L - Liskov Substitution Principle (Substitution de Liskov)

Les sous-classes doivent pouvoir remplacer leur classe parente sans casser le programme.

```python
# MAUVAIS : le Carre viole le contrat du Rectangle
class Rectangle:
    def __init__(self, largeur: float, hauteur: float):
        self._largeur = largeur
        self._hauteur = hauteur

    @property
    def largeur(self) -> float:
        return self._largeur

    @largeur.setter
    def largeur(self, valeur: float) -> None:
        self._largeur = valeur

    @property
    def hauteur(self) -> float:
        return self._hauteur

    @hauteur.setter
    def hauteur(self, valeur: float) -> None:
        self._hauteur = valeur

    def aire(self) -> float:
        return self._largeur * self._hauteur

# Ce Carre viole LSP car il change le comportement des setters
# class Carre(Rectangle):
#     def __init__(self, cote):
#         super().__init__(cote, cote)
#     @Rectangle.largeur.setter
#     def largeur(self, valeur):
#         self._largeur = valeur
#         self._hauteur = valeur  # Effet de bord inattendu !

# BON : utiliser la composition ou une hierarchie differente
class Forme(ABC):
    @abstractmethod
    def aire(self) -> float:
        ...

class Rectangle(Forme):
    def __init__(self, largeur: float, hauteur: float):
        self.largeur = largeur
        self.hauteur = hauteur

    def aire(self) -> float:
        return self.largeur * self.hauteur

class Carre(Forme):
    def __init__(self, cote: float):
        self.cote = cote

    def aire(self) -> float:
        return self.cote ** 2
```

### I - Interface Segregation Principle (Segregation d'interface)

Les clients ne doivent pas dependre d'interfaces qu'ils n'utilisent pas.

```python
# MAUVAIS : interface trop large
class Worker(ABC):
    @abstractmethod
    def travailler(self): ...
    @abstractmethod
    def manger(self): ...
    @abstractmethod
    def dormir(self): ...

# Un robot ne mange pas et ne dort pas !

# BON : interfaces segregees
class Travailleur(ABC):
    @abstractmethod
    def travailler(self): ...

class Humain(ABC):
    @abstractmethod
    def manger(self): ...
    @abstractmethod
    def dormir(self): ...

class Employe(Travailleur, Humain):
    def travailler(self): ...
    def manger(self): ...
    def dormir(self): ...

class Robot(Travailleur):
    def travailler(self): ...
```

### D - Dependency Inversion Principle (Inversion des dependances)

Les modules de haut niveau ne doivent pas dependre des modules de bas niveau. Les deux doivent dependre d'**abstractions**.

```python
# MAUVAIS : couplage fort
class MySQL:
    def requete(self, sql: str) -> list:
        ...

class ServiceUtilisateur:
    def __init__(self):
        self.db = MySQL()  # Dependance directe !

# BON : inversion de dependance
class BaseDeDonnees(ABC):
    @abstractmethod
    def requete(self, sql: str) -> list:
        ...

class MySQL(BaseDeDonnees):
    def requete(self, sql: str) -> list:
        ...

class PostgreSQL(BaseDeDonnees):
    def requete(self, sql: str) -> list:
        ...

class ServiceUtilisateur:
    def __init__(self, db: BaseDeDonnees):  # Depend de l'abstraction
        self.db = db

# Injection de dependance
service = ServiceUtilisateur(PostgreSQL())
```

---

## Mixins

Les mixins sont des classes conçues pour etre **combinees** avec d'autres classes via l'heritage multiple. Elles ajoutent des fonctionnalites sans etre des classes autonomes.

```python
class SerializableMixin:
    """Ajoute la capacite de serialisation JSON."""

    def to_dict(self) -> dict:
        return {k: v for k, v in self.__dict__.items() if not k.startswith("_")}

    def to_json(self) -> str:
        import json
        return json.dumps(self.to_dict(), default=str)

class AffichableMixin:
    """Ajoute un affichage formate."""

    def afficher(self) -> None:
        nom_classe = self.__class__.__name__
        attrs = ", ".join(f"{k}={v!r}" for k, v in self.__dict__.items()
                         if not k.startswith("_"))
        print(f"[{nom_classe}] {attrs}")

class HorodatableMixin:
    """Ajoute un horodatage automatique."""

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        original_init = cls.__init__

        def new_init(self, *args, **kw):
            from datetime import datetime
            original_init(self, *args, **kw)
            self.cree_le = datetime.now()

        cls.__init__ = new_init


class Produit(SerializableMixin, AffichableMixin):
    def __init__(self, nom: str, prix: float):
        self.nom = nom
        self.prix = prix


p = Produit("Laptop", 999.99)
p.afficher()           # [Produit] nom='Laptop', prix=999.99
print(p.to_json())     # {"nom": "Laptop", "prix": 999.99}
print(p.to_dict())     # {"nom": "Laptop", "prix": 999.99}
```

---

## Context Managers en tant que Classes

Les context managers sont utilises avec l'instruction `with`. Ils garantissent le nettoyage des ressources.

```python
class GestionnaireConnexion:
    """Context manager pour une connexion a une base de donnees."""

    def __init__(self, hote: str, port: int):
        self.hote = hote
        self.port = port
        self.connexion = None

    def __enter__(self):
        """Appele a l'entree du bloc with."""
        print(f"Connexion a {self.hote}:{self.port}...")
        self.connexion = {"hote": self.hote, "port": self.port, "active": True}
        return self.connexion

    def __exit__(self, exc_type, exc_val, exc_tb):
        """
        Appele a la sortie du bloc with (meme en cas d'exception).

        Args:
            exc_type: Type de l'exception (ou None).
            exc_val: Valeur de l'exception (ou None).
            exc_tb: Traceback (ou None).

        Returns:
            True pour supprimer l'exception, False pour la propager.
        """
        print(f"Fermeture de la connexion a {self.hote}:{self.port}")
        if self.connexion:
            self.connexion["active"] = False
        if exc_type is not None:
            print(f"Exception capturee: {exc_type.__name__}: {exc_val}")
        return False  # Propage l'exception


# Utilisation
with GestionnaireConnexion("localhost", 5432) as conn:
    print(f"Connexion active: {conn['active']}")
    # Le nettoyage est garanti meme en cas d'erreur

# Connexion a localhost:5432...
# Connexion active: True
# Fermeture de la connexion a localhost:5432
```

> [!example] Comparaison avec le C
> En C, la gestion des ressources necessite des `goto cleanup` ou des appels explicites :
> ```c
> FILE *f = fopen("data.txt", "r");
> if (f == NULL) { /* erreur */ }
> // ... utilisation ...
> fclose(f);  // Facile a oublier !
> ```
> Le context manager Python garantit que `__exit__` est **toujours** appele.

### Context manager avec un timer

```python
import time

class Timer:
    """Mesure le temps d'execution d'un bloc de code."""

    def __init__(self, nom: str = "Bloc"):
        self.nom = nom
        self.debut = 0.0
        self.duree = 0.0

    def __enter__(self):
        self.debut = time.perf_counter()
        return self

    def __exit__(self, *args):
        self.duree = time.perf_counter() - self.debut
        print(f"[{self.nom}] Duree: {self.duree:.4f}s")
        return False


with Timer("Tri") as t:
    sorted(range(1_000_000, 0, -1))
# [Tri] Duree: 0.1234s
```

---

## Carte Mentale ASCII

```
                      POO AVANCEE
                          |
        +---------+-------+-------+---------+
        |         |       |       |         |
   Decorateurs   ABC   Patterns SOLID   Divers
        |         |       |       |         |
   +----+----+  abstract +--+--+ S.R.P  +--+--+
   |    |    |  method   |     | O/C    |     |
 @wraps args  Protocols  |     | L.S.P  slots  Mixins
 classe retry           Single Factory  I.S.P  Metacl.
 cache  valid.          ton    Obs.    D.I.P
                        Strat.

   Context Managers          Descripteurs
   __enter__ / __exit__      __get__ / __set__
   with statement            __set_name__
```

---

## Exercices

### Exercice 1 : Decorateur @valider

Creez un decorateur `@valider` configurable qui verifie que les arguments d'une fonction satisfont des conditions. Exemple d'utilisation :
```python
@valider(age=lambda x: 0 < x < 150, nom=lambda x: len(x) > 0)
def creer_personne(nom: str, age: int) -> dict:
    return {"nom": nom, "age": age}
```

### Exercice 2 : Systeme de plugins avec ABC

Creez un systeme de plugins pour un editeur de texte. Definissez une ABC `Plugin` avec des methodes `nom()`, `activer()`, `desactiver()`, `executer(texte)`. Implementez : `PluginMajuscules`, `PluginCompteurMots`, `PluginRemplaceur`. Creez un `GestionnairePlugins` qui charge et execute les plugins.

### Exercice 3 : Observer pour un systeme de log

Implementez le pattern Observer pour un systeme de logging. Les evenements (info, warning, error) sont emis par un `Logger` et recus par differents `Handler` (console, fichier, email pour les erreurs critiques). Utilisez des decorateurs pour les niveaux de log.

---

## Liens

- [[03 - POO en Python]] - Bases de la POO
- [[05 - Gestion des Erreurs et Fichiers]] - Exceptions et context managers en pratique
- [[01 - Introduction a Python]] - Rappels sur les fonctions
- [[02 - Structures de Donnees Python]] - Collections utilisees dans les patterns
