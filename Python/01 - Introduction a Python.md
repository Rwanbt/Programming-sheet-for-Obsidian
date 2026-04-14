# Introduction a Python

## Qu'est-ce que Python ?

Python est un langage de programmation **interprete**, **multi-paradigme** et **dynamiquement type**, cree par Guido van Rossum en 1991. Contrairement au C que vous avez etudie au premier trimestre, Python privilegie la **lisibilite** et la **productivite** du developpeur plutot que la performance brute.

Python est aujourd'hui l'un des langages les plus utilises au monde, avec des applications dans :
- Le **developpement web** (Django, Flask, FastAPI)
- La **data science** et le **machine learning** (NumPy, pandas, scikit-learn, TensorFlow)
- L'**automatisation** et le **scripting**
- Le **developpement d'APIs**
- L'**intelligence artificielle**

> [!tip] Analogie
> Si le C est un couteau suisse ou chaque outil doit etre deploye manuellement (allocation memoire, gestion des pointeurs, compilation), Python est un robot de cuisine : vous lui dites ce que vous voulez, et il gere les details techniques pour vous. Vous perdez un peu de controle fin, mais vous gagnez enormement en vitesse de developpement.

---

## Pourquoi apprendre Python apres le C ?

Avoir appris le C en premier est un **avantage enorme**. Vous comprenez deja :
- Comment la memoire fonctionne (pile, tas)
- Les pointeurs et references
- La compilation et le linking
- Les types de donnees de bas niveau

Python vous permettra de :
1. **Developper plus rapidement** des prototypes et des applications
2. **Manipuler des donnees** facilement
3. **Acceder a un ecosysteme** de bibliotheques immense
4. **Comprendre les concepts de haut niveau** (POO, programmation fonctionnelle)

> [!info] Le saviez-vous ?
> CPython, l'implementation de reference de Python, est ecrit en C ! Votre connaissance du C vous aidera a comprendre ce qui se passe "sous le capot".

---

## Interprete vs Compile : Comparaison avec le C

### Le pipeline de compilation en C

En C, le code passe par plusieurs etapes avant de devenir un executable :

```
+------------------+     +----------------+     +------------------+
|   Code source    | --> | Preprocesseur  | --> | Code preprocesse |
|   (.c, .h)       |     | (cpp)          |     | (.i)             |
+------------------+     +----------------+     +------------------+
                                                        |
                                                        v
+------------------+     +----------------+     +------------------+
|   Executable     | <-- |   Linker       | <-- |  Code objet      |
|   (a.out/.exe)   |     | (ld)           |     |  (.o)            |
+------------------+     +----------------+     +------------------+
                                                        ^
                                                        |
                                              +------------------+
                                              |  Compilateur     |
                                              |  (gcc/cc1)       |
                                              +------------------+
```

### Le pipeline d'interpretation en Python

Python fonctionne differemment :

```
+------------------+     +------------------+     +------------------+
|   Code source    | --> |   Compilateur    | --> |    Bytecode      |
|   (.py)          |     |   Python         |     |    (.pyc)        |
+------------------+     +------------------+     +------------------+
                                                        |
                                                        v
                                              +------------------+
                                              | Machine Virtuelle|
                                              | Python (PVM)     |
                                              +------------------+
                                                        |
                                                        v
                                              +------------------+
                                              |    Execution     |
                                              +------------------+
```

> [!warning] Attention
> Python n'est pas "purement interprete". Le code source est d'abord compile en **bytecode** (fichiers `.pyc` dans `__pycache__`), puis ce bytecode est interprete par la PVM. C'est une distinction importante.

### Tableau comparatif

| Aspect | C | Python |
|---|---|---|
| Type | Compile | Interprete (avec bytecode) |
| Typage | Statique, fort | Dynamique, fort |
| Memoire | Manuelle (malloc/free) | Automatique (garbage collector) |
| Vitesse d'execution | Tres rapide | Plus lent (10-100x) |
| Vitesse de developpement | Lente | Tres rapide |
| Portabilite | Recompilation necessaire | Bytecode portable |
| Fichier resultat | Binaire natif | Script interprete |

---

## Installation de Python

### Windows

1. Telecharger Python depuis [python.org](https://www.python.org/downloads/)
2. **Cocher "Add Python to PATH"** lors de l'installation
3. Verifier dans un terminal :

```python
# Dans le terminal (pas dans Python)
python --version
# Python 3.12.x

pip --version
# pip 24.x
```

### Linux (Ubuntu/Debian)

```python
# Python est souvent pre-installe
python3 --version

# Sinon :
# sudo apt update
# sudo apt install python3 python3-pip
```

### macOS

```python
# Avec Homebrew :
# brew install python3

python3 --version
```

> [!info] Python 2 vs Python 3
> Python 2 est **obsolete** depuis le 1er janvier 2020. Utilisez **toujours** Python 3. Sur certains systemes, la commande `python` pointe encore vers Python 2. Utilisez `python3` pour etre sur.

---

## Le REPL (Read-Eval-Print Loop)

Le REPL est un environnement interactif ou vous pouvez tester du code Python ligne par ligne. C'est quelque chose qui n'existe pas en C (ou il faut recompiler a chaque modification).

```python
# Lancer le REPL :
# $ python3

>>> 2 + 3
5

>>> "Bonjour" + " " + "Python"
'Bonjour Python'

>>> type(42)
<class 'int'>

>>> help(print)
# Affiche la documentation de print

>>> exit()
# Quitte le REPL
```

> [!tip] Analogie
> Le REPL est comme une calculatrice programmable. Vous tapez une expression, Python l'evalue et vous montre le resultat immediatement. En C, il faudrait ecrire un fichier .c, le compiler, puis l'executer pour chaque test.

### Comparaison du workflow

```
C :    editer -> sauvegarder -> compiler -> corriger erreurs -> executer
Python : editer -> sauvegarder -> executer (ou tester dans le REPL)
```

---

## L'Indentation : le Coeur de Python

### En C : les accolades

```c
// C utilise des accolades pour delimiter les blocs
if (x > 0) {
    printf("Positif\n");
    if (x > 100) {
        printf("Grand nombre\n");
    }
}
```

### En Python : l'indentation

```python
# Python utilise l'indentation (4 espaces par convention)
if x > 0:
    print("Positif")
    if x > 100:
        print("Grand nombre")
```

> [!warning] Attention
> L'indentation n'est pas cosmetique en Python, elle est **syntaxique**. Une mauvaise indentation provoque une `IndentationError`. Utilisez **4 espaces** (pas des tabulations) par niveau d'indentation. Configurez votre editeur en consequence.

```python
# ERREUR : indentation incorrecte
if True:
print("Erreur!")  # IndentationError: expected an indented block

# ERREUR : indentation inconsistante
if True:
    print("Ligne 1")
      print("Ligne 2")  # IndentationError: unexpected indent
```

### Convention PEP 8

PEP 8 est le guide de style officiel de Python. Les regles principales :
- **4 espaces** par niveau d'indentation
- **79 caracteres** maximum par ligne (120 pour certains projets)
- **2 lignes vides** entre les definitions de fonctions de niveau superieur
- **1 ligne vide** entre les methodes d'une classe
- **snake_case** pour les fonctions et variables : `ma_fonction`, `mon_variable`
- **PascalCase** pour les classes : `MaClasse`
- **UPPER_SNAKE_CASE** pour les constantes : `MA_CONSTANTE`

---

## Variables et Typage Dynamique

### En C : typage statique

```c
// C : le type est declare explicitement et fixe
int age = 25;
float prix = 19.99;
char nom[] = "Alice";
// age = "vingt-cinq";  // ERREUR de compilation
```

### En Python : typage dynamique

```python
# Python : le type est infere automatiquement
age = 25           # int
prix = 19.99       # float
nom = "Alice"      # str

# La variable peut changer de type (mais ce n'est pas recommande)
age = "vingt-cinq"  # Pas d'erreur, age est maintenant un str
```

> [!info] Typage dynamique vs statique
> En C, le **compilateur** verifie les types. En Python, les types sont verifies a l'**execution**. Cela signifie que certaines erreurs de type ne seront detectees qu'au moment ou le code problematique est execute.

### Pas de declaration sans affectation

```python
# En C : int x;  // declaration sans initialisation (valeur indeterminee)
# En Python : impossible de declarer sans affecter
# x  # NameError: name 'x' is not defined

x = None  # Equivalent le plus proche : affecter None
```

### Affectation multiple

```python
# Affectation multiple
a, b, c = 1, 2, 3

# Echange de valeurs (pas besoin de variable temporaire comme en C)
a, b = b, a  # a=2, b=1

# Meme valeur
x = y = z = 0
```

---

## Les Types de Donnees Fondamentaux

### int (entier)

```python
# Pas de limite de taille ! (contrairement au C : int = 32 bits)
x = 42
grand_nombre = 123_456_789_012_345  # Les underscores ameliorent la lisibilite
negatif = -17

# Bases differentes
binaire = 0b1010      # 10 en decimal
octal = 0o17          # 15 en decimal
hexadecimal = 0xFF    # 255 en decimal

# Operations
print(10 / 3)    # 3.3333... (division flottante, DIFFERENT du C !)
print(10 // 3)   # 3 (division entiere, comme en C)
print(10 % 3)    # 1 (modulo, comme en C)
print(2 ** 10)   # 1024 (puissance, pas d'equivalent direct en C)
```

> [!warning] Division en Python vs C
> En C, `10 / 3` donne `3` (division entiere entre entiers). En Python, `10 / 3` donne `3.3333...` (toujours flottant). Pour la division entiere en Python, utilisez `//`.

### float (nombre a virgule flottante)

```python
pi = 3.14159
temperature = -40.0
scientifique = 1.5e-3   # 0.0015

# Attention aux erreurs d'arrondi (meme probleme qu'en C)
print(0.1 + 0.2)        # 0.30000000000000004
print(0.1 + 0.2 == 0.3) # False !

# Solution : utiliser math.isclose()
import math
print(math.isclose(0.1 + 0.2, 0.3))  # True
```

### str (chaine de caracteres)

```python
# En C : char* ou char[] (tableau de caracteres termine par '\0')
# En Python : type str, immutable, Unicode natif

simple = 'Bonjour'
double = "Bonjour"
multi_ligne = """Ceci est
une chaine
sur plusieurs lignes"""

# Les chaines sont IMMUTABLES (contrairement aux char[] en C)
nom = "Alice"
# nom[0] = 'a'  # TypeError: 'str' object does not support item assignment

# Longueur
print(len(nom))  # 5 (en C : strlen(nom))

# Concatenation
prenom = "Alice"
message = "Bonjour " + prenom  # Concatenation avec +
# En C : strcat() ou snprintf()

# Repetition
ligne = "-" * 40  # "----------------------------------------"

# Indexation (comme un tableau en C)
mot = "Python"
print(mot[0])    # 'P' (premier caractere)
print(mot[-1])   # 'n' (dernier caractere, pas d'equivalent en C)

# Slicing (decoupe) - pas d'equivalent en C
print(mot[0:3])  # 'Pyt' (de l'index 0 a 2)
print(mot[2:])   # 'thon' (de l'index 2 a la fin)
print(mot[:2])   # 'Py' (du debut a l'index 1)
print(mot[::-1]) # 'nohtyP' (inversion)
```

### Methodes de chaines utiles

```python
texte = "  Bonjour le Monde  "

print(texte.strip())       # "Bonjour le Monde" (supprime espaces)
print(texte.lower())       # "  bonjour le monde  "
print(texte.upper())       # "  BONJOUR LE MONDE  "
print(texte.replace("Monde", "Python"))  # "  Bonjour le Python  "
print(texte.split())       # ['Bonjour', 'le', 'Monde']
print(",".join(["a", "b", "c"]))  # "a,b,c"
print("Python" in texte)   # False
print(texte.find("Monde")) # 14 (index, -1 si non trouve)
print(texte.count("o"))    # 3
print(texte.startswith("  B"))  # True
print(texte.endswith("  "))     # True
```

### bool (booleen)

```python
# En C : 0 = false, tout le reste = true (ou stdbool.h)
# En Python : True et False (avec majuscule)

actif = True
termine = False

# Valeurs "falsy" (evaluees a False) :
# False, 0, 0.0, "", [], {}, (), set(), None

# Valeurs "truthy" (evaluees a True) :
# Tout le reste

print(bool(0))      # False
print(bool(42))     # True
print(bool(""))     # False
print(bool("abc"))  # True
print(bool([]))     # False
print(bool([1]))    # True
```

### None (valeur nulle)

```python
# En C : NULL (pointeur nul)
# En Python : None (objet singleton)

resultat = None

# Tester None avec 'is', pas '=='
if resultat is None:
    print("Pas de resultat")

# Utilisation courante : valeur par defaut
def chercher(valeur, liste):
    for element in liste:
        if element == valeur:
            return element
    return None  # Explicite : rien trouve
```

---

## Les f-strings (Formatted String Literals)

Les f-strings sont la methode moderne (Python 3.6+) pour formater les chaines.

```python
# En C : printf("Nom: %s, Age: %d\n", nom, age);
# En Python :

nom = "Alice"
age = 25
note = 15.667

# f-string basique
print(f"Nom: {nom}, Age: {age}")

# Expressions dans les f-strings
print(f"Dans 5 ans : {age + 5}")
print(f"Nom en majuscules : {nom.upper()}")

# Formatage numerique
print(f"Note : {note:.2f}")        # "Note : 15.67" (2 decimales)
print(f"Pourcentage : {0.856:.1%}") # "Pourcentage : 85.6%"
print(f"Hexa : {255:#x}")          # "Hexa : 0xff"
print(f"Binaire : {10:#b}")        # "Binaire : 0b1010"

# Alignement et remplissage
print(f"{'gauche':<20}")   # "gauche              "
print(f"{'droite':>20}")   # "              droite"
print(f"{'centre':^20}")   # "       centre       "
print(f"{'padded':*^20}")  # "*******padded*******"

# Debug (Python 3.8+)
x = 42
print(f"{x=}")  # "x=42"
print(f"{x + 1=}")  # "x + 1=43"
```

> [!tip] Analogie
> Les f-strings sont comme `printf()` en C, mais en beaucoup plus puissant et lisible. Au lieu de `printf("Nom: %s, Age: %d", nom, age)`, vous ecrivez `f"Nom: {nom}, Age: {age}"`. Les expressions sont directement dans la chaine !

### Anciennes methodes (pour reference)

```python
# Methode .format() (Python 2.6+)
print("Nom: {}, Age: {}".format(nom, age))
print("Nom: {0}, Age: {1}".format(nom, age))

# Operateur % (style C, deconseille)
print("Nom: %s, Age: %d" % (nom, age))
```

---

## Operateurs

### Operateurs arithmetiques

```python
a, b = 17, 5

print(a + b)   # 22  Addition
print(a - b)   # 12  Soustraction
print(a * b)   # 85  Multiplication
print(a / b)   # 3.4 Division (toujours float !)
print(a // b)  # 3   Division entiere
print(a % b)   # 2   Modulo
print(a ** b)  # 1419857 Puissance (pas en C sans pow())
```

### Operateurs de comparaison

```python
x, y = 10, 20

print(x == y)   # False  Egal
print(x != y)   # True   Different
print(x < y)    # True   Inferieur
print(x > y)    # False  Superieur
print(x <= y)   # True   Inferieur ou egal
print(x >= y)   # False  Superieur ou egal

# Comparaisons chainees (pas possible en C !)
age = 25
print(18 <= age < 65)  # True (equivalent a : 18 <= age and age < 65)
```

### Operateurs logiques

```python
# En C : &&, ||, !
# En Python : and, or, not

a, b = True, False

print(a and b)  # False (C : a && b)
print(a or b)   # True  (C : a || b)
print(not a)    # False (C : !a)

# Court-circuit (comme en C)
# Python evalue de gauche a droite et s'arrete des que le resultat est determine
print(False and print("jamais execute"))  # False
print(True or print("jamais execute"))    # True
```

### Operateurs d'identite et d'appartenance

```python
# Operateurs d'identite (pas d'equivalent en C)
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)   # True  (meme valeur)
print(a is b)   # False (objets differents en memoire)
print(a is c)   # True  (meme objet en memoire)

# Operateurs d'appartenance (pas d'equivalent en C)
print(2 in [1, 2, 3])      # True
print(4 not in [1, 2, 3])  # True
print("Py" in "Python")    # True
```

> [!warning] `is` vs `==`
> `==` compare les **valeurs**. `is` compare les **identites** (meme objet en memoire). Utilisez `is` uniquement pour comparer avec `None`, `True` et `False`.

---

## Structures de Controle

### if / elif / else

```python
# En C :
# if (age >= 18) {
#     printf("Majeur\n");
# } else if (age >= 16) {
#     printf("Presque majeur\n");
# } else {
#     printf("Mineur\n");
# }

# En Python :
age = 17

if age >= 18:
    print("Majeur")
elif age >= 16:
    print("Presque majeur")
else:
    print("Mineur")
```

> [!info] Differences avec le C
> - Pas de parentheses obligatoires autour de la condition
> - Deux-points `:` apres chaque condition
> - `elif` au lieu de `else if`
> - Indentation au lieu d'accolades
> - Pas de `switch/case` classique (mais `match/case` depuis Python 3.10)

### Expression ternaire

```python
# En C : int max = (a > b) ? a : b;
# En Python :
a, b = 10, 20
maximum = a if a > b else b  # 20
```

### match / case (Python 3.10+)

```python
# Equivalent du switch/case en C, mais beaucoup plus puissant
commande = "quitter"

match commande:
    case "demarrer":
        print("Demarrage...")
    case "arreter" | "quitter":  # Plusieurs valeurs
        print("Arret...")
    case _:  # default
        print("Commande inconnue")
```

### Boucle while

```python
# En C :
# int i = 0;
# while (i < 5) {
#     printf("%d\n", i);
#     i++;
# }

# En Python :
i = 0
while i < 5:
    print(i)
    i += 1  # Pas de ++ en Python !

# while avec else (specifique a Python)
i = 0
while i < 5:
    if i == 3:
        break
    i += 1
else:
    # Execute UNIQUEMENT si la boucle se termine normalement (sans break)
    print("Boucle terminee sans break")
```

> [!warning] Pas d'operateur ++ ou --
> Python n'a pas les operateurs `++` et `--` du C. Utilisez `i += 1` et `i -= 1`.

### Boucle for

```python
# En C : for (int i = 0; i < 5; i++) { ... }
# En Python : for itere sur des SEQUENCES, pas avec un compteur

# Iterer sur un range
for i in range(5):       # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 8):    # 2, 3, 4, 5, 6, 7
    print(i)

for i in range(0, 10, 2): # 0, 2, 4, 6, 8 (pas de 2)
    print(i)

# Iterer sur une liste
fruits = ["pomme", "banane", "cerise"]
for fruit in fruits:
    print(fruit)

# Iterer avec index (enumerate)
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
# 0: pomme
# 1: banane
# 2: cerise

# Iterer sur une chaine
for caractere in "Python":
    print(caractere)

# for avec else (comme while)
for i in range(10):
    if i == 5:
        break
else:
    print("Jamais execute car break a ete atteint")
```

> [!tip] Analogie
> La boucle `for` en Python est comme un `foreach` dans d'autres langages. Elle parcourt directement les elements d'une collection, sans besoin d'index. C'est comme si en C, au lieu de `for(int i=0; i<n; i++) { printf("%d", arr[i]); }`, vous pouviez ecrire `pour chaque element de arr : afficher(element)`.

### break, continue, pass

```python
# break : sort de la boucle (comme en C)
for i in range(10):
    if i == 5:
        break
    print(i)  # 0, 1, 2, 3, 4

# continue : passe a l'iteration suivante (comme en C)
for i in range(10):
    if i % 2 == 0:
        continue
    print(i)  # 1, 3, 5, 7, 9

# pass : ne fait rien (placeholder, pas d'equivalent en C)
for i in range(10):
    pass  # TODO: implementer plus tard

if True:
    pass  # Bloc vide (en C : {} ou ;)
```

---

## Fonctions

### Definition et appel

```python
# En C :
# int addition(int a, int b) {
#     return a + b;
# }

# En Python :
def addition(a, b):
    return a + b

resultat = addition(3, 5)
print(resultat)  # 8
```

### Retour multiple (pas possible en C sans struct/pointeurs)

```python
def min_max(liste):
    return min(liste), max(liste)

minimum, maximum = min_max([3, 1, 4, 1, 5, 9])
print(f"Min: {minimum}, Max: {maximum}")  # Min: 1, Max: 9
```

### Arguments par defaut

```python
def saluer(nom, salutation="Bonjour"):
    return f"{salutation}, {nom} !"

print(saluer("Alice"))            # "Bonjour, Alice !"
print(saluer("Bob", "Salut"))     # "Salut, Bob !"
```

> [!warning] Piege classique : argument mutable par defaut
> Ne jamais utiliser une liste ou un dictionnaire comme valeur par defaut !
> ```python
> # MAUVAIS :
> def ajouter(element, liste=[]):
>     liste.append(element)
>     return liste
> 
> print(ajouter(1))  # [1]
> print(ajouter(2))  # [1, 2] !! La meme liste est reutilisee
> 
> # BON :
> def ajouter(element, liste=None):
>     if liste is None:
>         liste = []
>     liste.append(element)
>     return liste
> ```

### Arguments nommes (keyword arguments)

```python
def creer_profil(nom, age, ville="Paris"):
    return f"{nom}, {age} ans, {ville}"

# Appel avec arguments nommes (ordre libre)
print(creer_profil(age=25, nom="Alice", ville="Lyon"))
```

### *args et **kwargs

```python
# *args : nombre variable d'arguments positionnels
def somme(*args):
    """Calcule la somme de tous les arguments."""
    total = 0
    for nombre in args:
        total += nombre
    return total

print(somme(1, 2, 3))      # 6
print(somme(1, 2, 3, 4, 5)) # 15

# **kwargs : nombre variable d'arguments nommes
def afficher_info(**kwargs):
    """Affiche toutes les informations passees en argument."""
    for cle, valeur in kwargs.items():
        print(f"{cle}: {valeur}")

afficher_info(nom="Alice", age=25, ville="Paris")
# nom: Alice
# age: 25
# ville: Paris

# Combinaison
def fonction_complete(obligatoire, *args, defaut="valeur", **kwargs):
    print(f"Obligatoire: {obligatoire}")
    print(f"Args: {args}")
    print(f"Defaut: {defaut}")
    print(f"Kwargs: {kwargs}")

fonction_complete("a", "b", "c", defaut="X", extra1=1, extra2=2)
# Obligatoire: a
# Args: ('b', 'c')
# Defaut: X
# Kwargs: {'extra1': 1, 'extra2': 2}
```

> [!info] Ordre des parametres
> L'ordre des parametres doit etre : `positionnels, *args, keyword-only, **kwargs`.
> ```
> def f(pos1, pos2, *args, kw_only1, kw_only2="def", **kwargs):
> ```

---

## Docstrings et Type Hints

### Docstrings

```python
def calculer_imc(poids: float, taille: float) -> float:
    """
    Calcule l'Indice de Masse Corporelle (IMC).

    Args:
        poids: Le poids en kilogrammes.
        taille: La taille en metres.

    Returns:
        L'IMC calcule.

    Raises:
        ValueError: Si le poids ou la taille est negatif.

    Examples:
        >>> calculer_imc(70, 1.75)
        22.857142857142858
    """
    if poids <= 0 or taille <= 0:
        raise ValueError("Le poids et la taille doivent etre positifs")
    return poids / (taille ** 2)

# Acceder a la docstring
print(calculer_imc.__doc__)
help(calculer_imc)
```

### Type Hints (annotations de type)

```python
# Les type hints sont OPTIONNELS et ne sont PAS verifies a l'execution
# Ils servent a la documentation et aux outils d'analyse statique (mypy)

def saluer(nom: str) -> str:
    return f"Bonjour, {nom} !"

def additionner(a: int, b: int) -> int:
    return a + b

# Types complexes
from typing import Optional, Union

def chercher(cle: str) -> Optional[int]:
    """Retourne un int ou None."""
    pass

def traiter(valeur: Union[int, str]) -> str:
    """Accepte un int ou un str."""
    return str(valeur)

# Python 3.10+ : syntaxe simplifiee
def chercher_v2(cle: str) -> int | None:
    pass

def traiter_v2(valeur: int | str) -> str:
    return str(valeur)
```

> [!example] Comparaison type hints Python vs C
> ```c
> // C : les types sont obligatoires et verifies a la compilation
> int addition(int a, int b) {
>     return a + b;
> }
> ```
> ```python
> # Python : les type hints sont optionnels et informatifs
> def addition(a: int, b: int) -> int:
>     return a + b
> 
> # Ceci fonctionne quand meme (pas d'erreur a l'execution) :
> addition("hello", " world")  # "hello world"
> ```

---

## Apercu des Collections

Python offre plusieurs types de collections integres. Ils seront detailles dans la note suivante.

```python
# Liste (equivalent approximatif d'un tableau dynamique en C)
fruits = ["pomme", "banane", "cerise"]
fruits.append("orange")
print(fruits[0])  # "pomme"

# Dictionnaire (equivalent d'une table de hachage)
ages = {"Alice": 25, "Bob": 30}
print(ages["Alice"])  # 25

# Ensemble (set) - pas d'equivalent direct en C
unique = {1, 2, 3, 2, 1}
print(unique)  # {1, 2, 3}

# Tuple (comme une liste, mais immutable)
coordonnees = (48.8566, 2.3522)
```

---

## Entrees Utilisateur

```python
# En C : scanf("%d", &age);
# En Python :

nom = input("Entrez votre nom : ")  # Toujours une chaine !
print(f"Bonjour, {nom}")

# Pour obtenir un entier, il faut convertir
age = int(input("Entrez votre age : "))
print(f"Vous avez {age} ans")

# Pour un float
taille = float(input("Entrez votre taille (m) : "))

# Gestion d'erreur basique
try:
    nombre = int(input("Entrez un nombre : "))
except ValueError:
    print("Ce n'est pas un nombre valide !")
```

> [!warning] input() retourne toujours un str
> Contrairement a `scanf()` en C qui parse directement selon le format, `input()` retourne **toujours** une chaine. Vous devez explicitement convertir avec `int()`, `float()`, etc.

---

## Le Pattern main

### En C

```c
// C : le point d'entree est la fonction main()
#include <stdio.h>

void saluer(const char* nom) {
    printf("Bonjour, %s!\n", nom);
}

int main(int argc, char* argv[]) {
    saluer("Monde");
    return 0;
}
```

### En Python

```python
# Python : le pattern if __name__ == "__main__"

def saluer(nom: str) -> None:
    """Salue la personne donnee."""
    print(f"Bonjour, {nom} !")

def main() -> None:
    """Point d'entree principal du programme."""
    saluer("Monde")

if __name__ == "__main__":
    main()
```

> [!info] Pourquoi `if __name__ == "__main__"` ?
> Quand un fichier Python est **execute directement**, `__name__` vaut `"__main__"`.
> Quand il est **importe** comme module, `__name__` vaut le nom du module.
> Ce pattern permet au fichier de fonctionner a la fois comme script et comme module importable.

```python
# fichier: utils.py
def carre(x):
    return x ** 2

if __name__ == "__main__":
    # Ce code s'execute UNIQUEMENT si on lance : python utils.py
    print(carre(5))  # 25

# fichier: main.py
from utils import carre  # Importe la fonction, n'execute PAS le bloc main
print(carre(3))  # 9
```

---

## Premier Programme Complet

Voici un programme complet illustrant les concepts vus dans cette note :

```python
"""
Programme de calcul de notes.
Demonstre les concepts de base de Python.
"""


def calculer_moyenne(notes: list[float]) -> float:
    """
    Calcule la moyenne d'une liste de notes.

    Args:
        notes: Liste de notes (float).

    Returns:
        La moyenne des notes.

    Raises:
        ValueError: Si la liste est vide.
    """
    if not notes:
        raise ValueError("La liste de notes est vide")
    return sum(notes) / len(notes)


def obtenir_mention(moyenne: float) -> str:
    """Retourne la mention correspondant a la moyenne."""
    if moyenne >= 16:
        return "Tres Bien"
    elif moyenne >= 14:
        return "Bien"
    elif moyenne >= 12:
        return "Assez Bien"
    elif moyenne >= 10:
        return "Passable"
    else:
        return "Insuffisant"


def afficher_bulletin(nom: str, notes: dict[str, float]) -> None:
    """Affiche le bulletin de notes d'un etudiant."""
    print(f"\n{'=' * 50}")
    print(f"  BULLETIN DE NOTES - {nom.upper()}")
    print(f"{'=' * 50}")

    toutes_notes = []
    for matiere, note in notes.items():
        print(f"  {matiere:<20} : {note:>5.1f}/20")
        toutes_notes.append(note)

    moyenne = calculer_moyenne(toutes_notes)
    mention = obtenir_mention(moyenne)

    print(f"{'=' * 50}")
    print(f"  {'MOYENNE':<20} : {moyenne:>5.1f}/20")
    print(f"  {'MENTION':<20} : {mention}")
    print(f"{'=' * 50}")


def main() -> None:
    """Point d'entree du programme."""
    notes = {
        "Mathematiques": 15.5,
        "Physique": 12.0,
        "Informatique": 18.0,
        "Anglais": 14.5,
        "Francais": 11.0,
    }

    afficher_bulletin("Alice Dupont", notes)

    # Interaction utilisateur
    print("\n--- Ajout d'une note ---")
    matiere = input("Matiere : ")
    try:
        note = float(input("Note (/20) : "))
        if 0 <= note <= 20:
            notes[matiere] = note
            afficher_bulletin("Alice Dupont", notes)
        else:
            print("La note doit etre entre 0 et 20")
    except ValueError:
        print("Veuillez entrer un nombre valide")


if __name__ == "__main__":
    main()
```

---

## Carte Mentale ASCII

```
                        PYTHON - Introduction
                               |
        +----------+-----------+-----------+----------+
        |          |           |           |          |
    Interprete  Variables   Controle   Fonctions   Types
        |          |           |           |          |
   +----+----+   Dynamique  +--+--+    def        +--+--+--+--+
   |         |   Type hints |     |    return     |  |  |  |  |
  REPL    Bytecode          if   for   *args     int fl str bo None
   |         |             elif while  **kwargs       at    ol
  PVM    .pyc files        else range  docstring
                           match break
                                continue
```

---

## Exercices

### Exercice 1 : Convertisseur de temperature

Ecrivez un programme qui demande une temperature en Celsius et affiche la conversion en Fahrenheit et Kelvin. Utilisez des fonctions separees pour chaque conversion, avec des type hints et des docstrings.

```python
# Formules :
# F = C * 9/5 + 32
# K = C + 273.15
```

### Exercice 2 : FizzBuzz

Ecrivez le classique FizzBuzz : pour les nombres de 1 a 100, affichez "Fizz" si le nombre est divisible par 3, "Buzz" si divisible par 5, "FizzBuzz" si divisible par les deux, sinon le nombre lui-meme. Comparez votre solution Python avec ce que vous auriez ecrit en C.

### Exercice 3 : Calculatrice interactive

Creez une calculatrice interactive en ligne de commande qui :
- Demande deux nombres et une operation (+, -, *, /)
- Affiche le resultat avec 2 decimales
- Gere la division par zero
- Continue jusqu'a ce que l'utilisateur tape "quitter"
- Utilise le pattern `if __name__ == "__main__"`

### Exercice 4 : Analyse de texte

Ecrivez un programme qui prend une phrase en entree et affiche :
- Le nombre de mots
- Le nombre de caracteres (avec et sans espaces)
- Le mot le plus long
- La phrase inversee
Utilisez les methodes de chaines et les f-strings.

---

## Liens

- [[01 - Introduction au C et Compilation]] - Comparaison avec le pipeline C
- [[02 - Structures de Donnees Python]] - Les collections Python en detail
- [[03 - POO en Python]] - Programmation orientee objet
- [[04 - POO Avancee]] - Concepts avances de POO
- [[05 - Gestion des Erreurs et Fichiers]] - Exceptions et fichiers
