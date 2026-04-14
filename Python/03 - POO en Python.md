# POO en Python

## Qu'est-ce que la Programmation Orientee Objet ?

La Programmation Orientee Objet (POO) est un **paradigme de programmation** qui organise le code autour d'**objets** plutot que de fonctions et de logique procedurale. Un objet combine des **donnees** (attributs) et des **comportements** (methodes) en une seule entite.

Python est un langage **multi-paradigme** : il supporte la programmation procedurale, fonctionnelle et orientee objet. Contrairement au C qui est purement procedural (pas de classes natives), Python a ete concu des le depart avec la POO comme concept central. En fait, **tout en Python est un objet** : les entiers, les chaines, les fonctions, meme les classes elles-memes.

> [!tip] Analogie
> Imaginez que vous concevez un jeu video. En C (procedural), vous auriez des `struct` pour les donnees et des fonctions separees pour les manipuler : `struct Joueur`, `deplacer_joueur()`, `attaquer_joueur()`. En POO, tout est regroupe dans une **classe** `Joueur` qui sait se deplacer et attaquer elle-meme. C'est comme la difference entre un plan d'architecte (struct + fonctions separees) et une maison LEGO avec des instructions integrees (classe avec methodes).

---

## Classes vs Structs : la Transition depuis le C

### En C : struct + fonctions

```c
// C : les donnees et les fonctions sont separees
#include <stdio.h>
#include <string.h>

typedef struct {
    char nom[50];
    int age;
    float note;
} Etudiant;

// Les fonctions operent sur la struct
void afficher_etudiant(const Etudiant *e) {
    printf("Nom: %s, Age: %d, Note: %.1f\n", e->nom, e->age, e->note);
}

void modifier_note(Etudiant *e, float nouvelle_note) {
    e->note = nouvelle_note;
}

int main() {
    Etudiant alice = {"Alice", 20, 15.5};
    afficher_etudiant(&alice);
    modifier_note(&alice, 17.0);
    return 0;
}
```

### En Python : classe

```python
# Python : donnees et fonctions sont regroupees dans une classe
class Etudiant:
    def __init__(self, nom: str, age: int, note: float):
        self.nom = nom
        self.age = age
        self.note = note

    def afficher(self) -> None:
        print(f"Nom: {self.nom}, Age: {self.age}, Note: {self.note:.1f}")

    def modifier_note(self, nouvelle_note: float) -> None:
        self.note = nouvelle_note

# Utilisation
alice = Etudiant("Alice", 20, 15.5)
alice.afficher()
alice.modifier_note(17.0)
```

> [!info] Differences fondamentales
> | Aspect | C (struct) | Python (class) |
> |---|---|---|
> | Donnees | Champs de struct | Attributs d'instance |
> | Fonctions | Fonctions externes | Methodes integrees |
> | Heritage | Non supporte | Supporte (simple et multiple) |
> | Encapsulation | Pas de mecanisme | Convention et name mangling |
> | Polymorphisme | Via pointeurs de fonctions | Natif (duck typing) |
> | Memoire | Allocation manuelle | Garbage collector |

---

## Definition d'une Classe

### Syntaxe de base

```python
class NomDeLaClasse:
    """Docstring de la classe."""

    # Attribut de classe (partage par toutes les instances)
    compteur = 0

    def __init__(self, param1, param2):
        """Constructeur : initialise une instance."""
        # Attributs d'instance (propres a chaque objet)
        self.param1 = param1
        self.param2 = param2
        NomDeLaClasse.compteur += 1

    def methode(self):
        """Une methode d'instance."""
        return f"{self.param1} - {self.param2}"
```

### Le constructeur `__init__`

```python
class Voiture:
    """Represente une voiture."""

    def __init__(self, marque: str, modele: str, annee: int, km: float = 0.0):
        """
        Initialise une nouvelle voiture.

        Args:
            marque: La marque de la voiture.
            modele: Le modele.
            annee: L'annee de fabrication.
            km: Le kilometrage (0 par defaut).
        """
        self.marque = marque
        self.modele = modele
        self.annee = annee
        self.km = km
        self.en_marche = False

    def demarrer(self) -> None:
        """Demarre la voiture."""
        if not self.en_marche:
            self.en_marche = True
            print(f"{self.marque} {self.modele} demarre.")
        else:
            print("La voiture est deja en marche.")

    def rouler(self, distance: float) -> None:
        """Fait rouler la voiture sur une distance donnee."""
        if self.en_marche:
            self.km += distance
            print(f"Parcours de {distance} km. Total: {self.km} km")
        else:
            print("Demarrez d'abord la voiture !")

    def arreter(self) -> None:
        """Arrete la voiture."""
        self.en_marche = False
        print(f"{self.marque} {self.modele} s'arrete.")


# Utilisation
ma_voiture = Voiture("Peugeot", "208", 2023)
ma_voiture.demarrer()
ma_voiture.rouler(150.5)
ma_voiture.arreter()
```

> [!warning] `__init__` n'est PAS un constructeur au sens strict
> En Python, `__init__` est un **initialiseur**. Le vrai constructeur est `__new__` (rarement utilise directement). `__init__` recoit l'instance deja creee (`self`) et initialise ses attributs.

---

## self : la Reference a l'Instance

`self` est l'equivalent Python du pointeur `this` en C++ ou du pointeur vers la struct en C.

```python
class Point:
    def __init__(self, x: float, y: float):
        self.x = x  # self.x est l'attribut, x est le parametre
        self.y = y

    def distance_origine(self) -> float:
        return (self.x ** 2 + self.y ** 2) ** 0.5

    def deplacer(self, dx: float, dy: float) -> None:
        self.x += dx
        self.y += dy

# Quand vous appelez :
p = Point(3, 4)
p.distance_origine()  # Python traduit en : Point.distance_origine(p)
```

> [!info] Pourquoi `self` est explicite ?
> En C++, `this` est implicite. En Python, `self` doit etre le premier parametre de chaque methode d'instance. C'est une decision de design : "Explicit is better than implicit" (Zen of Python). Le nom `self` est une **convention**, pas un mot-cle, mais ne jamais utiliser un autre nom.

```
Appel de methode :
  p.deplacer(1, 2)
  
Python traduit en :
  Point.deplacer(p, 1, 2)
                 ^
                 |
               self
```

---

## Variables d'Instance vs Variables de Classe

```python
class Etudiant:
    # Variable de CLASSE (partagee par toutes les instances)
    universite = "Universite de Paris"
    nombre_etudiants = 0

    def __init__(self, nom: str, age: int):
        # Variables d'INSTANCE (propres a chaque objet)
        self.nom = nom
        self.age = age
        Etudiant.nombre_etudiants += 1

    def __del__(self):
        Etudiant.nombre_etudiants -= 1


# Demonstration
alice = Etudiant("Alice", 20)
bob = Etudiant("Bob", 22)

# Variables d'instance : differentes pour chaque objet
print(alice.nom)  # "Alice"
print(bob.nom)    # "Bob"

# Variable de classe : partagee
print(alice.universite)  # "Universite de Paris"
print(bob.universite)    # "Universite de Paris"
print(Etudiant.nombre_etudiants)  # 2

# Attention au piege !
alice.universite = "Sorbonne"  # Cree un attribut d'INSTANCE pour alice
print(alice.universite)  # "Sorbonne" (attribut d'instance)
print(bob.universite)    # "Universite de Paris" (attribut de classe)
```

```
Memoire :

Classe Etudiant
+----------------------------+
| universite = "U. de Paris" |
| nombre_etudiants = 2       |
+----------------------------+
       ^            ^
       |            |
+------+------+  +------+------+
| Instance:   |  | Instance:   |
| alice       |  | bob         |
| nom="Alice" |  | nom="Bob"   |
| age=20      |  | age=22      |
+-------------+  +-------------+
```

> [!warning] Piege des attributs de classe mutables
> ```python
> class Mauvais:
>     notes = []  # DANGER : partage entre toutes les instances !
> 
>     def ajouter_note(self, note):
>         self.notes.append(note)
> 
> a = Mauvais()
> b = Mauvais()
> a.ajouter_note(15)
> print(b.notes)  # [15] !! b voit la note de a
> 
> class Bon:
>     def __init__(self):
>         self.notes = []  # Chaque instance a sa propre liste
> ```

---

## Properties : @property et @setter

Les properties permettent de controler l'acces aux attributs, comme des getters/setters mais avec une syntaxe d'attribut.

```python
class Temperature:
    """Represente une temperature en Celsius."""

    def __init__(self, celsius: float = 0.0):
        self._celsius = celsius  # Attribut "prive" par convention

    @property
    def celsius(self) -> float:
        """Getter pour la temperature en Celsius."""
        return self._celsius

    @celsius.setter
    def celsius(self, valeur: float) -> None:
        """Setter avec validation."""
        if valeur < -273.15:
            raise ValueError("Temperature en dessous du zero absolu !")
        self._celsius = valeur

    @property
    def fahrenheit(self) -> float:
        """Propriete calculee : conversion en Fahrenheit."""
        return self._celsius * 9 / 5 + 32

    @fahrenheit.setter
    def fahrenheit(self, valeur: float) -> None:
        """Setter pour Fahrenheit : convertit en Celsius."""
        self.celsius = (valeur - 32) * 5 / 9


# Utilisation : syntaxe d'attribut, mais avec controle
t = Temperature(25)
print(t.celsius)      # 25 (appelle le getter)
print(t.fahrenheit)   # 77.0 (propriete calculee)

t.celsius = 100       # Appelle le setter (avec validation)
# t.celsius = -300    # ValueError !

t.fahrenheit = 32     # Modifie via Fahrenheit
print(t.celsius)      # 0.0
```

> [!tip] Analogie
> Les properties sont comme un guichet automatique. Vous interagissez avec l'interface simple (inserer carte, taper code), mais derriere, il y a des verifications complexes (solde suffisant, carte valide). En C, c'est comme encapsuler l'acces a un champ de struct derriere des fonctions `get_xxx()` et `set_xxx()`, mais avec la syntaxe agreable d'un acces direct.

---

## Heritage

L'heritage permet de creer des classes qui **heritent** des attributs et methodes d'une classe parente. C'est l'un des piliers de la POO, et cela n'existe pas en C.

### Heritage simple

```python
class Animal:
    """Classe de base pour les animaux."""

    def __init__(self, nom: str, age: int):
        self.nom = nom
        self.age = age

    def parler(self) -> str:
        return "..."

    def presenter(self) -> str:
        return f"{self.nom}, {self.age} ans"


class Chien(Animal):
    """Un chien herite d'Animal."""

    def __init__(self, nom: str, age: int, race: str):
        super().__init__(nom, age)  # Appel au constructeur parent
        self.race = race

    def parler(self) -> str:
        return "Ouaf !"

    def presenter(self) -> str:
        base = super().presenter()  # Reutilise la methode parente
        return f"{base}, race: {self.race}"


class Chat(Animal):
    """Un chat herite d'Animal."""

    def __init__(self, nom: str, age: int, interieur: bool = True):
        super().__init__(nom, age)
        self.interieur = interieur

    def parler(self) -> str:
        return "Miaou !"


# Utilisation
rex = Chien("Rex", 5, "Berger Allemand")
minou = Chat("Minou", 3)

print(rex.presenter())    # "Rex, 5 ans, race: Berger Allemand"
print(rex.parler())       # "Ouaf !"
print(minou.parler())     # "Miaou !"

# Verification d'heritage
print(isinstance(rex, Chien))    # True
print(isinstance(rex, Animal))   # True
print(isinstance(rex, Chat))     # False
print(issubclass(Chien, Animal)) # True
```

```
Hierarchie d'heritage :

        Animal
       /      \
    Chien     Chat
```

### Heritage multiple

Python supporte l'heritage multiple (contrairement a beaucoup de langages).

```python
class Volant:
    """Mixin pour les objets volants."""

    def voler(self) -> str:
        return f"{self.nom} vole dans les airs"


class Nageant:
    """Mixin pour les objets nageants."""

    def nager(self) -> str:
        return f"{self.nom} nage dans l'eau"


class Canard(Animal, Volant, Nageant):
    """Un canard peut parler, voler et nager."""

    def parler(self) -> str:
        return "Coin coin !"


donald = Canard("Donald", 2)
print(donald.parler())    # "Coin coin !"
print(donald.voler())     # "Donald vole dans les airs"
print(donald.nager())     # "Donald nage dans l'eau"
print(donald.presenter()) # "Donald, 2 ans"
```

### MRO (Method Resolution Order)

Quand une methode existe dans plusieurs classes parentes, Python utilise le MRO pour determiner laquelle appeler.

```python
class A:
    def methode(self):
        return "A"

class B(A):
    def methode(self):
        return "B"

class C(A):
    def methode(self):
        return "C"

class D(B, C):
    pass

d = D()
print(d.methode())  # "B" (B est avant C dans l'heritage)

# Voir le MRO
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

```
MRO de D (algorithme C3 linearization) :

     A
    / \
   B   C
    \ /
     D

Ordre de resolution : D -> B -> C -> A -> object
```

> [!info] L'algorithme C3
> Python utilise l'algorithme C3 linearization pour calculer le MRO. L'ordre respecte deux regles : une classe est toujours avant ses parents, et l'ordre d'heritage est preserve. Si ces regles sont contradictoires, Python leve une `TypeError`.

---

## super()

`super()` permet d'appeler des methodes de la classe parente.

```python
class Base:
    def __init__(self, x: int):
        self.x = x
        print(f"Base.__init__(x={x})")

class Derivee(Base):
    def __init__(self, x: int, y: int):
        super().__init__(x)  # Appelle Base.__init__
        self.y = y
        print(f"Derivee.__init__(y={y})")

d = Derivee(1, 2)
# Base.__init__(x=1)
# Derivee.__init__(y=2)
```

> [!warning] Toujours utiliser super()
> Ne jamais appeler directement `Base.__init__(self, x)`. Utilisez `super().__init__(x)`. Cela garantit le bon fonctionnement avec l'heritage multiple et le MRO.

### super() avec heritage multiple

```python
class A:
    def __init__(self):
        print("A.__init__")
        super().__init__()

class B(A):
    def __init__(self):
        print("B.__init__")
        super().__init__()

class C(A):
    def __init__(self):
        print("C.__init__")
        super().__init__()

class D(B, C):
    def __init__(self):
        print("D.__init__")
        super().__init__()

d = D()
# D.__init__
# B.__init__
# C.__init__
# A.__init__
```

---

## Polymorphisme

Le polymorphisme permet d'utiliser des objets de types differents de maniere uniforme.

```python
class Forme:
    def aire(self) -> float:
        raise NotImplementedError("Sous-classes doivent implementer aire()")

    def perimetre(self) -> float:
        raise NotImplementedError

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

class Triangle(Forme):
    def __init__(self, base: float, hauteur: float, cote1: float,
                 cote2: float, cote3: float):
        self.base = base
        self.hauteur = hauteur
        self.cote1 = cote1
        self.cote2 = cote2
        self.cote3 = cote3

    def aire(self) -> float:
        return 0.5 * self.base * self.hauteur

    def perimetre(self) -> float:
        return self.cote1 + self.cote2 + self.cote3


# Polymorphisme en action
formes: list[Forme] = [
    Rectangle(5, 3),
    Cercle(2),
    Triangle(4, 3, 3, 4, 5)
]

for forme in formes:
    # Meme interface, comportement different
    print(f"Aire: {forme.aire():.2f}, Perimetre: {forme.perimetre():.2f}")
```

> [!tip] Analogie
> Le polymorphisme c'est comme une telecommande universelle. Que vous pointiez vers une TV Samsung, LG ou Sony, le bouton "Volume+" fonctionne. Chaque TV l'implemente differemment en interne, mais l'interface est la meme. En C, vous obteniez un effet similaire avec des pointeurs de fonctions dans les structs, mais c'etait beaucoup plus complexe a mettre en place.

### Duck Typing

Python utilise le **duck typing** : "Si ca marche comme un canard et que ca fait coin-coin, c'est un canard."

```python
# Pas besoin d'heritage pour le polymorphisme !
class Fichier:
    def lire(self) -> str:
        return "Contenu du fichier"

class BaseDeDonnees:
    def lire(self) -> str:
        return "Donnees de la BDD"

class API:
    def lire(self) -> str:
        return "Reponse de l'API"

def afficher_donnees(source) -> None:
    """Fonctionne avec TOUT objet ayant une methode lire()."""
    print(source.lire())

# Pas besoin que les classes heritent d'une meme base
afficher_donnees(Fichier())
afficher_donnees(BaseDeDonnees())
afficher_donnees(API())
```

---

## Encapsulation

Python n'a pas de mots-cles `private` ou `protected` comme C++ ou Java. L'encapsulation repose sur des **conventions**.

### Convention avec underscore

```python
class CompteBancaire:
    def __init__(self, titulaire: str, solde: float = 0.0):
        self.titulaire = titulaire    # Public
        self._solde = solde           # "Protege" (convention : ne pas acceder directement)
        self.__historique = []        # "Prive" (name mangling)

    @property
    def solde(self) -> float:
        return self._solde

    def deposer(self, montant: float) -> None:
        if montant <= 0:
            raise ValueError("Le montant doit etre positif")
        self._solde += montant
        self.__historique.append(f"+{montant}")

    def retirer(self, montant: float) -> None:
        if montant <= 0:
            raise ValueError("Le montant doit etre positif")
        if montant > self._solde:
            raise ValueError("Solde insuffisant")
        self._solde -= montant
        self.__historique.append(f"-{montant}")

    def afficher_historique(self) -> None:
        for operation in self.__historique:
            print(operation)


compte = CompteBancaire("Alice", 1000)
compte.deposer(500)
compte.retirer(200)

# Acces public : OK
print(compte.titulaire)  # "Alice"

# Acces "protege" : fonctionne mais deconseille
print(compte._solde)  # 1300

# Acces "prive" : name mangling
# print(compte.__historique)  # AttributeError !
print(compte._CompteBancaire__historique)  # Fonctionne mais JAMAIS faire ca
```

> [!info] Niveaux d'acces en Python
> | Prefixe | Convention | Acces | Equivalent C++ |
> |---|---|---|---|
> | `nom` | Public | Libre | `public:` |
> | `_nom` | Protege | Deconseille hors classe/sous-classes | `protected:` |
> | `__nom` | Prive (name mangling) | Modifie en `_Classe__nom` | `private:` |
> | `__nom__` | Dunder (magic method) | Utilise par Python | Special |

---

## Methodes Dunder (Magic Methods)

Les methodes dunder (double underscore) permettent de definir le comportement des operateurs et des fonctions built-in pour vos classes.

### `__str__` et `__repr__`

```python
class Point:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __str__(self) -> str:
        """Representation lisible (pour l'utilisateur). Appelee par print()."""
        return f"({self.x}, {self.y})"

    def __repr__(self) -> str:
        """Representation technique (pour le developpeur). Appelee dans le REPL."""
        return f"Point(x={self.x}, y={self.y})"


p = Point(3, 4)
print(p)      # (3, 4)        -> appelle __str__
print(repr(p)) # Point(x=3, y=4) -> appelle __repr__
print([p])    # [Point(x=3, y=4)] -> dans une liste, __repr__ est utilise
```

> [!tip] Analogie
> `__str__` est comme la carte de visite que vous donnez a un client (presentation soignee). `__repr__` est comme votre badge employe (informations techniques completes). En regle generale, `__repr__` devrait idealement retourner une chaine qui, evaluee, recreerait l'objet.

### Operateurs de comparaison

```python
class Note:
    def __init__(self, valeur: float, matiere: str):
        self.valeur = valeur
        self.matiere = matiere

    def __eq__(self, other) -> bool:
        """Egalite : =="""
        if not isinstance(other, Note):
            return NotImplemented
        return self.valeur == other.valeur

    def __lt__(self, other) -> bool:
        """Inferieur : <"""
        if not isinstance(other, Note):
            return NotImplemented
        return self.valeur < other.valeur

    def __le__(self, other) -> bool:
        """Inferieur ou egal : <="""
        if not isinstance(other, Note):
            return NotImplemented
        return self.valeur <= other.valeur

    def __repr__(self) -> str:
        return f"Note({self.valeur}, '{self.matiere}')"


n1 = Note(15, "Math")
n2 = Note(12, "Info")
n3 = Note(15, "Anglais")

print(n1 == n3)  # True (meme valeur)
print(n1 > n2)   # True (Python deduit __gt__ de __lt__)
print(sorted([n1, n2, n3]))  # [Note(12, 'Info'), Note(15, 'Math'), Note(15, 'Anglais')]
```

> [!info] `functools.total_ordering`
> Au lieu de definir tous les operateurs de comparaison, vous pouvez utiliser `@functools.total_ordering` et ne definir que `__eq__` et un autre (`__lt__`, `__gt__`, `__le__` ou `__ge__`). Python deduira les autres.

### Operateurs arithmetiques

```python
class Vecteur:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __add__(self, other: "Vecteur") -> "Vecteur":
        """Addition : v1 + v2"""
        return Vecteur(self.x + other.x, self.y + other.y)

    def __sub__(self, other: "Vecteur") -> "Vecteur":
        """Soustraction : v1 - v2"""
        return Vecteur(self.x - other.x, self.y - other.y)

    def __mul__(self, scalaire: float) -> "Vecteur":
        """Multiplication par un scalaire : v * 3"""
        return Vecteur(self.x * scalaire, self.y * scalaire)

    def __rmul__(self, scalaire: float) -> "Vecteur":
        """Multiplication inverse : 3 * v"""
        return self.__mul__(scalaire)

    def __abs__(self) -> float:
        """Norme : abs(v)"""
        return (self.x ** 2 + self.y ** 2) ** 0.5

    def __repr__(self) -> str:
        return f"Vecteur({self.x}, {self.y})"


v1 = Vecteur(3, 4)
v2 = Vecteur(1, 2)

print(v1 + v2)     # Vecteur(4, 6)
print(v1 - v2)     # Vecteur(2, 2)
print(v1 * 2)      # Vecteur(6, 8)
print(3 * v1)      # Vecteur(9, 12)
print(abs(v1))     # 5.0
```

### `__len__`, `__getitem__`, `__iter__`

```python
class Playlist:
    def __init__(self, nom: str):
        self.nom = nom
        self._chansons: list[str] = []

    def ajouter(self, chanson: str) -> None:
        self._chansons.append(chanson)

    def __len__(self) -> int:
        """len(playlist)"""
        return len(self._chansons)

    def __getitem__(self, index: int) -> str:
        """playlist[0], playlist[-1], supporte aussi le slicing"""
        return self._chansons[index]

    def __iter__(self):
        """for chanson in playlist:"""
        return iter(self._chansons)

    def __contains__(self, chanson: str) -> bool:
        """chanson in playlist"""
        return chanson in self._chansons

    def __repr__(self) -> str:
        return f"Playlist('{self.nom}', {len(self)} chansons)"


pl = Playlist("Ma playlist")
pl.ajouter("Bohemian Rhapsody")
pl.ajouter("Stairway to Heaven")
pl.ajouter("Hotel California")

print(len(pl))        # 3
print(pl[0])          # "Bohemian Rhapsody"
print(pl[-1])         # "Hotel California"

for chanson in pl:    # Iterable grace a __iter__
    print(f"  - {chanson}")

print("Hotel California" in pl)  # True
```

### Tableau des methodes dunder principales

| Methode | Operateur/Fonction | Description |
|---|---|---|
| `__init__` | `Classe()` | Constructeur |
| `__str__` | `str()`, `print()` | Representation lisible |
| `__repr__` | `repr()` | Representation technique |
| `__len__` | `len()` | Longueur |
| `__getitem__` | `obj[key]` | Acces par index/cle |
| `__setitem__` | `obj[key] = val` | Affectation par index/cle |
| `__delitem__` | `del obj[key]` | Suppression par index/cle |
| `__iter__` | `for x in obj` | Iteration |
| `__next__` | `next()` | Element suivant |
| `__contains__` | `x in obj` | Appartenance |
| `__eq__` | `==` | Egalite |
| `__lt__` | `<` | Inferieur |
| `__le__` | `<=` | Inferieur ou egal |
| `__add__` | `+` | Addition |
| `__sub__` | `-` | Soustraction |
| `__mul__` | `*` | Multiplication |
| `__truediv__` | `/` | Division |
| `__floordiv__` | `//` | Division entiere |
| `__mod__` | `%` | Modulo |
| `__pow__` | `**` | Puissance |
| `__bool__` | `bool()` | Conversion booleenne |
| `__hash__` | `hash()` | Hash pour dict/set |
| `__call__` | `obj()` | Appel comme fonction |
| `__enter__`/`__exit__` | `with` | Context manager |

---

## Dataclasses

Les dataclasses (Python 3.7+) automatisent la creation de classes qui servent principalement a stocker des donnees.

```python
from dataclasses import dataclass, field

@dataclass
class Etudiant:
    nom: str
    age: int
    note: float = 0.0
    matieres: list[str] = field(default_factory=list)

    # Les methodes sont generees automatiquement :
    # __init__, __repr__, __eq__

# Utilisation
alice = Etudiant("Alice", 20, 15.5)
bob = Etudiant("Bob", 22, 12.0)

print(alice)  # Etudiant(nom='Alice', age=20, note=15.5, matieres=[])
print(alice == bob)  # False (__eq__ genere automatiquement)
```

> [!example] Comparaison : classe manuelle vs dataclass
> ```python
> # Sans dataclass (beaucoup de code repetitif)
> class Point:
>     def __init__(self, x: float, y: float):
>         self.x = x
>         self.y = y
>     def __repr__(self):
>         return f"Point(x={self.x}, y={self.y})"
>     def __eq__(self, other):
>         if not isinstance(other, Point):
>             return NotImplemented
>         return self.x == other.x and self.y == other.y
> 
> # Avec dataclass (automatique !)
> @dataclass
> class Point:
>     x: float
>     y: float
> ```

### Options avancees des dataclasses

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)  # Immutable (comme un namedtuple)
class Coordonnees:
    latitude: float
    longitude: float

@dataclass(order=True)  # Genere __lt__, __le__, __gt__, __ge__
class NoteExamen:
    sort_index: float = field(init=False, repr=False)
    matiere: str
    valeur: float

    def __post_init__(self):
        self.sort_index = self.valeur  # Tri par valeur

notes = [
    NoteExamen("Math", 15.0),
    NoteExamen("Info", 18.0),
    NoteExamen("Anglais", 12.0),
]
print(sorted(notes))
# [NoteExamen(matiere='Anglais', valeur=12.0),
#  NoteExamen(matiere='Math', valeur=15.0),
#  NoteExamen(matiere='Info', valeur=18.0)]
```

---

## Methodes Statiques et Methodes de Classe

### @staticmethod

Une methode statique n'a pas acces a l'instance (`self`) ni a la classe (`cls`). C'est une simple fonction dans le namespace de la classe.

```python
class MathUtils:
    @staticmethod
    def est_pair(n: int) -> bool:
        return n % 2 == 0

    @staticmethod
    def pgcd(a: int, b: int) -> int:
        while b:
            a, b = b, a % b
        return a

# Appel sans creer d'instance
print(MathUtils.est_pair(4))   # True
print(MathUtils.pgcd(12, 8))   # 4
```

### @classmethod

Une methode de classe recoit la classe elle-meme (`cls`) comme premier argument. Elle est souvent utilisee comme **constructeur alternatif** (factory method).

```python
class Date:
    def __init__(self, jour: int, mois: int, annee: int):
        self.jour = jour
        self.mois = mois
        self.annee = annee

    @classmethod
    def depuis_chaine(cls, date_str: str) -> "Date":
        """Constructeur alternatif a partir d'une chaine 'JJ/MM/AAAA'."""
        jour, mois, annee = map(int, date_str.split("/"))
        return cls(jour, mois, annee)  # cls() au lieu de Date()

    @classmethod
    def aujourdhui(cls) -> "Date":
        """Constructeur alternatif pour la date du jour."""
        from datetime import date
        d = date.today()
        return cls(d.day, d.month, d.year)

    def __repr__(self) -> str:
        return f"Date({self.jour:02d}/{self.mois:02d}/{self.annee})"


# Utilisation
d1 = Date(15, 3, 2024)
d2 = Date.depuis_chaine("25/12/2024")
d3 = Date.aujourdhui()

print(d1)  # Date(15/03/2024)
print(d2)  # Date(25/12/2024)
```

> [!info] Quand utiliser quoi ?
> | Type | Acces a | Quand l'utiliser |
> |---|---|---|
> | Methode d'instance | `self` (instance) | Opere sur les donnees d'une instance |
> | @classmethod | `cls` (classe) | Constructeur alternatif, methode liee a la classe |
> | @staticmethod | Rien | Fonction utilitaire liee au concept de la classe |

---

## Composition vs Heritage

### Heritage ("est un")

```python
class Vehicule:
    def __init__(self, vitesse_max: float):
        self.vitesse_max = vitesse_max

class Voiture(Vehicule):  # Une voiture EST UN vehicule
    def __init__(self, vitesse_max: float, places: int):
        super().__init__(vitesse_max)
        self.places = places
```

### Composition ("a un")

```python
class Moteur:
    def __init__(self, puissance: int):
        self.puissance = puissance
        self.en_marche = False

    def demarrer(self) -> None:
        self.en_marche = True

    def arreter(self) -> None:
        self.en_marche = False

class Roue:
    def __init__(self, taille: int):
        self.taille = taille

class Voiture:  # Une voiture A UN moteur et DES roues
    def __init__(self, puissance: int, taille_roues: int):
        self.moteur = Moteur(puissance)
        self.roues = [Roue(taille_roues) for _ in range(4)]

    def demarrer(self) -> None:
        self.moteur.demarrer()
        print("Voiture demarree")

    def arreter(self) -> None:
        self.moteur.arreter()
        print("Voiture arretee")
```

> [!warning] Privilegiez la composition
> En general, **privilegiez la composition sur l'heritage**. L'heritage cree un couplage fort entre les classes. La composition est plus flexible. Utilisez l'heritage quand il y a une vraie relation "est un" et que les sous-classes partagent du comportement.

```
Heritage :
  Vehicule --> Voiture --> VoitureElectrique
  (couplage fort, hierarchie rigide)

Composition :
  Voiture ---a un---> Moteur
          ---a des--> Roue[4]
          ---a un---> GPS
  (couplage faible, flexible)
```

---

## Carte Mentale ASCII

```
                          POO en Python
                               |
        +----------+-----------+-----------+----------+
        |          |           |           |          |
     Classes   Heritage    Polymorphisme Encapsul.  Dunder
        |          |           |           |       Methods
   +----+----+  +--+--+   Duck typing  _prive    +--+--+
   |    |    |  |     |                __mangl.   |     |
 __init__ @prop simple  Methode       @property  __str__ __add__
  self  methods multiple partagee               __repr__ __eq__
  class  static super() interface              __len__  __iter__
  vars   class  MRO                            __getitem__
         meth.
                    Dataclasses        Composition
                    @dataclass         vs Heritage
                    frozen, order      "a un" vs "est un"
```

---

## Exercices

### Exercice 1 : Classe Fraction

Creez une classe `Fraction` qui represente une fraction mathematique. Implementez :
- `__init__` avec simplification automatique (PGCD)
- `__add__`, `__sub__`, `__mul__`, `__truediv__`
- `__eq__`, `__lt__`
- `__str__` et `__repr__`
- `@classmethod depuis_chaine(cls, s)` pour parser "3/4"

### Exercice 2 : Systeme de bibliotheque

Creez un systeme avec les classes : `Livre` (titre, auteur, isbn, disponible), `Membre` (nom, id, livres_empruntes), `Bibliotheque` (nom, livres, membres). Implementez emprunter, retourner, rechercher. Utilisez `@dataclass` pour `Livre` et `Membre`.

### Exercice 3 : Hierarchie de formes geometriques

Creez une hierarchie : `Forme` (base abstraite) -> `Rectangle`, `Cercle`, `Triangle`. Chaque forme doit avoir `aire()`, `perimetre()`, `__str__`, `__repr__`, `__eq__`. Ajoutez `__add__` pour fusionner les aires. Creez une `FormeComposee` utilisant la composition.

### Exercice 4 : Carte de jeu

Creez les classes `Carte` (valeur, couleur) et `Paquet` (52 cartes). Implementez : melanger, distribuer, trier. `Paquet` doit supporter `len()`, `__getitem__`, et `__iter__`. Utilisez `@functools.total_ordering` pour les comparaisons de cartes.

---

## Liens

- [[01 - Introduction a Python]] - Les bases du langage
- [[02 - Structures de Donnees Python]] - Collections Python
- [[04 - POO Avancee]] - Decorators, ABC, design patterns
- [[03 - Structures et Typedef]] - Structures en C
