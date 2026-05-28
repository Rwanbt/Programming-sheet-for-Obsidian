# Introduction a Java

## Qu'est-ce que Java ?

Java est un langage de programmation **compile et interprete**, **fortement type**, **oriente objet** et **portable**, cree par James Gosling chez Sun Microsystems en 1995. Son slogan historique est *"Write Once, Run Anywhere"* : un programme Java compile une seule fois s'execute sur n'importe quelle plateforme disposant d'une JVM (Java Virtual Machine).

Java est aujourd'hui l'un des langages les plus utilises dans le monde professionnel :
- Le **developpement backend** et les **microservices** (Spring Boot, Quarkus)
- Les **applications Android**
- Les **systemes distribues** et les **Big Data** (Hadoop, Kafka, Spark)
- Les **applications d'entreprise** (banques, assurances, logistique)
- Les **APIs REST** haute performance

> [!tip] Positionnement par rapport a ce que vous connaissez
> Si le **[[01 - Introduction au C et Compilation|C]]** est un couteau suisse brut (puissance maximale, controle total, complexite maximale) et **[[01 - Introduction a Python|Python]]** un robot de cuisine (productivite maximale, magie implicite), Java est une **cuisine professionnelle bien equipee** : structure rigoureuse, securite forte, performance elevee, mais verbose. Vous ecrivez plus de code qu'en [[01 - Introduction a Python|Python]], mais le compilateur vous protege de nombreuses erreurs avant meme d'executer le programme.

---

## Java et la JVM : la portabilite par le bytecode

### Le pipeline de compilation Java

En [[01 - Introduction au C et Compilation|C]], la compilation produit un binaire natif pour une architecture specifique (x86, ARM...). En Java, la compilation produit du **bytecode** — un langage intermediaire neutre — que la JVM execute sur n'importe quelle plateforme.

```
+------------------+     +-------------------+     +------------------+
|   Code source    | --> |   Compilateur     | --> |    Bytecode      |
|   (Main.java)    |     |   javac           |     |   (Main.class)   |
+------------------+     +-------------------+     +------------------+
                                                          |
                          +---------+---------+           |
                          |         |         |           v
                     +---------+ +-----+ +-------+  +----------+
                     | JVM     | | JVM | | JVM   |  | JVM lit  |
                     | Windows | | Mac | | Linux |  | le meme  |
                     +---------+ +-----+ +-------+  | .class   |
                                                     +----------+
```

### Le JIT Compiler : optimisation a la volee

La JVM ne se contente pas d'interpreter le bytecode. Elle inclut un **JIT (Just-In-Time) Compiler** qui analyse le code pendant l'execution et compile en natif les parties les plus utilisees (les "hot spots"). C'est pourquoi Java peut approcher les performances du C sur les applications longue duree.

> [!info] JRE vs JDK
> - **JRE** (Java Runtime Environment) : contient uniquement la JVM pour *executer* du Java
> - **JDK** (Java Development Kit) : contient la JVM + `javac` + les outils de developpement pour *developper* du Java
> - En tant que developpeur, vous avez toujours besoin du **JDK**

### Les versions de Java

Java suit un cycle de releases tous les 6 mois depuis Java 9 (2017). Certaines versions sont **LTS** (Long-Term Support) et recueillent les corrections pendant 8+ ans :

| Version | Date | Statut | Nouveautes cles |
|---------|------|--------|-----------------|
| Java 8 | 2014 | LTS (fin 2030) | Lambdas, Streams, Optional |
| Java 11 | 2018 | LTS | var local, HTTP Client |
| Java 17 | 2021 | LTS | Records, Sealed Classes, Pattern Matching |
| Java 21 | 2023 | LTS (recommande) | Virtual Threads, Sequenced Collections |
| Java 24 | 2025 | Standard | Unnamed Variables, Flexible Constructor Bodies |

> [!warning] Choisir la bonne version
> En enterprise, Java 17 ou Java 21 LTS sont les cibles standard. Evitez Java 8 pour les nouveaux projets — il manque des fonctionnalites modernes essentielles. Ce cours utilise **Java 21**.

---

## Installation du JDK

### Option 1 : SDKMAN (recommande sur Linux/Mac)

```bash
# Installer SDKMAN
curl -s "https://get.sdkman.io" | bash

# Installer Java 21 LTS (Temurin = build open source recommandee)
sdk install java 21.0.3-tem

# Verifier l'installation
java -version
javac -version
```

### Option 2 : Installation directe (Windows)

Telecharger le JDK depuis [adoptium.net](https://adoptium.net) (Eclipse Temurin) et ajouter `JAVA_HOME` aux variables d'environnement.

### Verifier l'installation

```bash
$ java -version
openjdk version "21.0.3" 2024-04-16
OpenJDK Runtime Environment Temurin-21.0.3+9 (build 21.0.3+9)
OpenJDK 64-Bit Server VM Temurin-21.0.3+9 (build 21.0.3+9, mixed mode)

$ javac -version
javac 21.0.3
```

---

## Comparaison avec C et Python

| Caracteristique | C | Python | Java |
|----------------|---|--------|------|
| **Typage** | Statique, faible | Dynamique, fort | Statique, fort |
| **Compilation** | Natif | Bytecode Python | Bytecode JVM |
| **Gestion memoire** | Manuelle (malloc/free) — voir [[01 - Introduction au C et Compilation]] | Automatique (GC) | Automatique (GC) |
| **Performance** | Tres haute | Moyenne | Haute (JIT) |
| **Portabilite** | Faible (recompiler par OS) | Haute | Tres haute (JVM) |
| **Verbosité** | Moyenne | Faible | Haute |
| **POO** | Non (structs) | Multi-paradigme | Oblige |
| **Securite types** | Faible | Moyenne | Tres haute |
| **Ecosysteme** | Bibliotheques systeme | Scientifique/Web | Enterprise/Web |
| **Premier programme** | 5 lignes | 1 ligne | 7 lignes |

> [!info] Quand choisir Java ?
> - Applications **enterprise** avec des contraintes de fiabilite elevees
> - **APIs REST** et microservices a haute disponibilite
> - Applications **Android** (via Kotlin, derivee de Java)
> - Systemes ou la **maintenance sur 10+ ans** est une contrainte

---

## Votre premier programme Java

```java
// Fichier : HelloWorld.java
// Le nom du fichier DOIT correspondre au nom de la classe publique

public class HelloWorld {
    
    // Point d'entree du programme
    // Equivalent du int main(void) en [[01 - Introduction au C et Compilation|C]] ou if __name__ == '__main__' en [[01 - Introduction a Python|Python]]
    public static void main(String[] args) {
        System.out.println("Bonjour, Holberton !");
        
        // args contient les arguments de la ligne de commande
        if (args.length > 0) {
            System.out.println("Argument recu : " + args[0]);
        }
    }
}
```

```bash
# Compilation
javac HelloWorld.java
# Produit HelloWorld.class (le bytecode)

# Execution
java HelloWorld
# Output : Bonjour, Holberton !
```

> [!tip] Comparaison avec [[01 - Introduction a Python|Python]] et [[01 - Introduction au C et Compilation|C]]
> ```python
> # Python : 1 ligne (voir [[01 - Introduction a Python]])
> print("Bonjour, Holberton !")
> ```
> ```c
> // C : 5 lignes (voir [[01 - Introduction au C et Compilation]])
> #include <stdio.h>
> int main(void) {
>     printf("Bonjour, Holberton !\n");
>     return 0;
> }
> ```
> Java est plus verbeux, mais cette structure apporte de la rigueur : tout code appartient a une classe, le point d'entree est clairement identifie, les types sont declares.

---

## Types primitifs vs Objets

### Les 8 types primitifs

Java distingue les **types primitifs** (stockes sur la pile, pas d'objet) des **types objets** (stockes sur le tas, instances de classes) :

```java
// Types primitifs — valeur stockee directement
byte  b  = 42;           // 8 bits,  -128 a 127
short s  = 1000;         // 16 bits, -32768 a 32767
int   i  = 2_000_000;   // 32 bits  (separateur _ pour lisibilite)
long  l  = 9_000_000_000L; // 64 bits, suffixe L obligatoire

float  f = 3.14f;        // 32 bits flottant, suffixe f obligatoire
double d = 3.14159265;   // 64 bits flottant (defaut pour les decimaux)

boolean ok  = true;      // true ou false (PAS 0/1 comme en C)
char    c   = 'A';       // 16 bits Unicode (pas ASCII comme en C)
```

> [!warning] Differences avec [[01 - Introduction au C et Compilation|C]]
> En [[01 - Introduction au C et Compilation|C]], `int` varie selon l'architecture (32 ou 64 bits). En Java, `int` est **toujours** 32 bits. Il n'y a pas de `unsigned` en Java. Le `char` Java est Unicode (16 bits), pas ASCII (8 bits).

### Les types objets (wrapper classes)

Chaque type primitif a un **wrapper** objet correspondant. Ces wrappers sont utilises avec les collections (qui ne peuvent stocker que des objets) :

```java
// Autoboxing : Java convertit automatiquement primitif <-> objet
Integer objInt = 42;           // autoboxing : int -> Integer
int     primInt = objInt;      // unboxing : Integer -> int

// Les wrappers fournissent des methodes utiles
int max = Integer.MAX_VALUE;   // 2147483647
int min = Integer.MIN_VALUE;   // -2147483648
int parsed = Integer.parseInt("42");  // convertit String -> int
String s = Integer.toString(42);      // convertit int -> String

// Attention au piegage null avec les wrappers
Integer nullable = null;
int valeur = nullable; // NullPointerException au runtime !
```

| Primitif | Wrapper | Taille |
|----------|---------|--------|
| `int` | `Integer` | 32 bits |
| `long` | `Long` | 64 bits |
| `double` | `Double` | 64 bits |
| `float` | `Float` | 32 bits |
| `boolean` | `Boolean` | 1 bit |
| `char` | `Character` | 16 bits |
| `byte` | `Byte` | 8 bits |
| `short` | `Short` | 16 bits |

---

## Les Strings : immutabilite et StringBuilder

### String est immutable

En Java, un objet `String` ne peut **jamais** etre modifie apres creation. Toute "modification" cree un nouvel objet :

```java
String s = "Holberton";

// Ces operations NE modifient PAS s, elles creent de nouveaux objets
String upper = s.toUpperCase();    // "HOLBERTON" (nouveau String)
String concat = s + " School";    // "Holberton School" (nouveau String)

// s est toujours "Holberton"
System.out.println(s);            // Holberton

// Methodes utiles de String
String texte = "  Bonjour le monde  ";
System.out.println(texte.trim());           // "Bonjour le monde"
System.out.println(texte.trim().length());  // 17
System.out.println(texte.contains("monde")); // true
System.out.println(texte.replace("o", "0")); // "  B0nj0ur le m0nde  "
System.out.println(texte.split(" ").length); // nombre de mots

// Comparaison : TOUJOURS utiliser .equals(), jamais ==
String a = "hello";
String b = "hello";
System.out.println(a == b);       // Potentiellement true (interning) - NE PAS UTILISER
System.out.println(a.equals(b));  // true - CORRECT
System.out.println(a.equalsIgnoreCase("HELLO")); // true
```

> [!warning] Le piegeon de == avec les Strings
> En [[01 - Introduction au C et Compilation|C]], on compare des `char*` avec `strcmp()`. En Java, `==` compare les **references** (l'adresse memoire), pas le contenu. Deux Strings avec le meme contenu peuvent etre deux objets differents. **Utilisez toujours `.equals()`** pour comparer le contenu.

### StringBuilder : modifications efficaces

Quand vous devez construire une chaine par concatenations repetees (dans une boucle par exemple), `String` est inefficace — chaque `+` cree un nouvel objet. `StringBuilder` mutate le meme objet :

```java
// INEFFICACE : cree N nouveaux objets String en memoire
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i + ", ";   // 1000 allocations !
}

// EFFICACE : un seul objet modifie en place
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
    sb.append(", ");
}
String result = sb.toString();  // un seul String final

// Methodes de StringBuilder
StringBuilder sb2 = new StringBuilder("Hello");
sb2.append(" World");      // "Hello World"
sb2.insert(5, ",");        // "Hello, World"
sb2.delete(5, 6);          // "Hello World"
sb2.reverse();             // "dlroW olleH"
sb2.length();              // longueur
```

---

## Tableaux et ArrayList

### Tableaux Java (array)

Les tableaux Java ont une **taille fixe** definie a la creation, comme en C :

```java
// Declaration et initialisation
int[] nombres = new int[5];        // [0, 0, 0, 0, 0]
int[] notes = {20, 15, 18, 12, 9}; // initialisation directe

// Acces par index (0-based, comme en C)
notes[0] = 19;                     // modification
System.out.println(notes[2]);      // 18

// Iteration
for (int i = 0; i < notes.length; i++) {
    System.out.println(notes[i]);
}

// For-each (equivalent Python for x in liste)
for (int note : notes) {
    System.out.println(note);
}

// Tableaux 2D
int[][] matrice = new int[3][4];    // 3 lignes, 4 colonnes
int[][] grille = {{1, 2}, {3, 4}, {5, 6}};

// Utilitaires : java.util.Arrays
import java.util.Arrays;
Arrays.sort(notes);                  // tri en place
System.out.println(Arrays.toString(notes)); // affichage lisible [9, 12, 15, 18, 19]
```

### ArrayList : tableaux dynamiques

`ArrayList` est l'equivalent de la liste [[01 - Introduction a Python|Python]] (voir aussi [[02 - Structures de Donnees Python]]) — taille variable, methodes riches :

```java
import java.util.ArrayList;
import java.util.List;

// Declaration : List (interface) = bonne pratique, ArrayList = implementation
List<String> prenoms = new ArrayList<>();

// Ajout
prenoms.add("Alice");
prenoms.add("Bob");
prenoms.add(1, "Charlie");  // insere a l'index 1

// Acces
System.out.println(prenoms.get(0));   // "Alice"
System.out.println(prenoms.size());   // 3

// Suppression
prenoms.remove("Bob");         // par valeur
prenoms.remove(0);             // par index

// Verification
prenoms.contains("Alice");     // true
prenoms.isEmpty();             // false

// Iteration
for (String prenom : prenoms) {
    System.out.println(prenom);
}

// Conversion array <-> ArrayList
String[] tableau = prenoms.toArray(new String[0]);
List<String> fromArray = new ArrayList<>(Arrays.asList(tableau));
```

> [!tip] Tableau vs ArrayList — quand choisir ?
> - **Tableau** : taille connue et fixe, performances maximales (primitifs), API bas niveau
> - **ArrayList** : taille variable, methodes riches, integration avec les Collections/Streams
> En pratique moderne : preferez `ArrayList` (ou `List`) dans 90% des cas.

---

## Controle de flux

### if / else if / else

```java
int score = 75;

if (score >= 90) {
    System.out.println("Excellent");
} else if (score >= 75) {
    System.out.println("Bien");
} else if (score >= 60) {
    System.out.println("Passable");
} else {
    System.out.println("Insuffisant");
}
```

### Switch expression (Java 14+)

Java 14 a modernise le `switch` en **expression** (retourne une valeur, pas de fall-through) :

```java
// Ancien switch (avant Java 14) — dangereux (oubli de break)
switch (jour) {
    case "Lundi":
    case "Mardi":
        System.out.println("Debut de semaine");
        break;
    case "Vendredi":
        System.out.println("Fin de semaine");
        break;
    default:
        System.out.println("Milieu de semaine");
}

// Nouveau switch expression (Java 14+) — prefere
String message = switch (jour) {
    case "Lundi", "Mardi" -> "Debut de semaine";
    case "Vendredi"       -> "Fin de semaine";
    default               -> "Milieu de semaine";
};

// Switch avec yield pour blocs multi-lignes
int nombreJours = switch (mois) {
    case "Janvier", "Mars", "Mai" -> 31;
    case "Avril", "Juin"          -> 30;
    case "Fevrier" -> {
        boolean bisextile = (annee % 4 == 0);
        yield bisextile ? 29 : 28;
    }
    default -> throw new IllegalArgumentException("Mois inconnu : " + mois);
};
```

### Boucles

```java
// for classique (identique a C)
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

// while
int n = 10;
while (n > 0) {
    System.out.println(n);
    n--;
}

// do-while (s'execute au moins une fois)
int compteur = 0;
do {
    compteur++;
} while (compteur < 5);

// for-each (equivalent Python for x in iterable)
List<String> langages = List.of("Java", "Python", "C");
for (String langage : langages) {
    System.out.println(langage);
}

// break et continue (meme comportement qu'en C)
for (int i = 0; i < 10; i++) {
    if (i == 3) continue;  // saute l'iteration 3
    if (i == 7) break;     // arrete la boucle a 7
    System.out.println(i);
}
```

---

## Methodes : overloading et varargs

### Definition d'une methode

```java
public class Calculatrice {
    
    // Methode simple : modificateur + type retour + nom + parametres
    public static int additionner(int a, int b) {
        return a + b;
    }
    
    // Methode void : ne retourne rien
    public static void afficherResultat(String label, int valeur) {
        System.out.println(label + " : " + valeur);
    }
}
```

### Overloading (surcharge)

Java permet plusieurs methodes avec le **meme nom** mais des **signatures differentes** (types ou nombre de parametres) :

```java
public class Formateur {
    
    // Version 1 : un String
    public static String formater(String texte) {
        return "[" + texte + "]";
    }
    
    // Version 2 : un int
    public static String formater(int nombre) {
        return String.format("%06d", nombre);  // 000042
    }
    
    // Version 3 : deux parametres
    public static String formater(String texte, int largeur) {
        return String.format("%-" + largeur + "s", texte);
    }
}

// Appel — Java choisit automatiquement la bonne version
Formateur.formater("Holberton");     // [Holberton]
Formateur.formater(42);              // 000042
Formateur.formater("Java", 10);     // "Java      "
```

### Varargs (nombre variable d'arguments)

```java
// ... indique un nombre variable d'arguments (toujours en dernier)
public static int somme(int... nombres) {
    int total = 0;
    for (int n : nombres) {  // nombres est un tableau int[]
        total += n;
    }
    return total;
}

// Appel avec 0, 1, ou N arguments
System.out.println(somme());           // 0
System.out.println(somme(5));          // 5
System.out.println(somme(1, 2, 3, 4)); // 10

// Equivalent Python : def somme(*nombres)
```

---

## Packages et imports

Les **packages** Java organisent le code en namespaces, similaires aux modules [[01 - Introduction a Python|Python]] :

```java
// Declaration du package (premiere ligne du fichier)
package com.holberton.cours.java;

// Import de classes specifiques (recommande)
import java.util.ArrayList;
import java.util.List;
import java.util.HashMap;

// Import de tout un package (deconseille — pollue le namespace)
import java.util.*;

// Import static : acces direct aux membres statiques
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;

public class Geometrie {
    public static double aireCercle(double rayon) {
        return PI * rayon * rayon;  // pas besoin de Math.PI
    }
    
    public static double hypotenuse(double a, double b) {
        return sqrt(a * a + b * b);  // pas besoin de Math.sqrt
    }
}
```

> [!info] Convention de nommage des packages
> Les packages Java suivent la convention **domaine inverse** : `com.entreprise.projet.module`. Cela garantit l'unicite mondiale. Pour Holberton : `com.holberton.cours.java`.

---

## Conventions de nommage Java

| Element | Convention | Exemple |
|---------|-----------|---------|
| **Classes** | PascalCase | `EtudiantManager` |
| **Methodes** | camelCase | `calculerMoyenne()` |
| **Variables** | camelCase | `nombreEtudiants` |
| **Constantes** | UPPER_SNAKE_CASE | `MAX_TAILLE_TABLEAU` |
| **Packages** | lowercase | `com.holberton.java` |
| **Interfaces** | PascalCase (souvent adjectif) | `Serializable`, `Comparable` |

```java
// Exemple applique
public class GestionEtudiants {
    
    private static final int CAPACITE_MAX = 100;  // constante
    private List<String> listeEtudiants;          // variable instance
    
    public void ajouterEtudiant(String nomEtudiant) {  // methode
        if (listeEtudiants.size() < CAPACITE_MAX) {
            listeEtudiants.add(nomEtudiant);
        }
    }
}
```

---

## Gestion des exceptions

### Le mecanisme try/catch/finally

En [[01 - Introduction au C et Compilation|C]], les erreurs sont gerees par codes de retour. En Java, les erreurs sont des **exceptions** — des objets que l'on "lance" (throw) et "attrape" (catch) — pour comparer avec Python, voir [[05 - Gestion des Erreurs et Fichiers]] :

```java
public class ExempleExceptions {
    
    public static void main(String[] args) {
        // Bloc try : code susceptible de lever une exception
        try {
            int[] tableau = {1, 2, 3};
            System.out.println(tableau[10]); // ArrayIndexOutOfBoundsException
            
            String texte = null;
            texte.length();                   // NullPointerException
            
        } catch (ArrayIndexOutOfBoundsException e) {
            // Attrape uniquement cette exception
            System.out.println("Index invalide : " + e.getMessage());
            
        } catch (NullPointerException e) {
            System.out.println("Objet null : " + e.getMessage());
            
        } catch (Exception e) {
            // Attrape toutes les exceptions (toujours en dernier)
            System.out.println("Erreur inattendue : " + e.getMessage());
            e.printStackTrace();  // Affiche la trace complete
            
        } finally {
            // TOUJOURS execute, avec ou sans exception
            // Utilise pour liberer des ressources (fichiers, connexions...)
            System.out.println("Nettoyage effectue");
        }
    }
}
```

### Checked vs Unchecked exceptions

Java distingue deux categories d'exceptions :

```
Exception (classe mere)
├── RuntimeException (unchecked — non obligatoirement capturees)
│   ├── NullPointerException
│   ├── ArrayIndexOutOfBoundsException
│   ├── IllegalArgumentException
│   └── ClassCastException
└── Checked exceptions (doivent etre declarees avec throws ou capturees)
    ├── IOException
    ├── SQLException
    └── FileNotFoundException
```

```java
// Checked exception : le compilateur OBLIGE a la gerer
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

// Option 1 : capturer avec try/catch
public static String lireFichier(String chemin) {
    try {
        return Files.readString(Path.of(chemin));
    } catch (IOException e) {
        System.err.println("Erreur lecture : " + e.getMessage());
        return "";
    }
}

// Option 2 : propager avec throws (le caller doit gerer)
public static String lireFichierOuFail(String chemin) throws IOException {
    return Files.readString(Path.of(chemin));
}
```

### Creer ses propres exceptions

```java
// Exception metier personnalisee
public class NoteInvalideException extends RuntimeException {
    
    private final int noteInvalide;
    
    public NoteInvalideException(int note) {
        super("Note invalide : " + note + " (doit etre entre 0 et 20)");
        this.noteInvalide = note;
    }
    
    public int getNoteInvalide() {
        return noteInvalide;
    }
}

// Utilisation
public static void validerNote(int note) {
    if (note < 0 || note > 20) {
        throw new NoteInvalideException(note);
    }
}
```

---

## System.out.println vs Logging

`System.out.println` est pratique pour le debug rapide, mais **deconseille en production**. Utilisez un framework de logging :

```java
// Logging avec SLF4J + Logback (standard Java)
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MonService {
    
    // Logger associe a la classe (convention standard)
    private static final Logger log = LoggerFactory.getLogger(MonService.class);
    
    public void traiterDonnees(String input) {
        log.info("Debut traitement pour : {}", input);  // {} = placeholder
        
        try {
            // ... traitement ...
            log.debug("Donnees parsees : {}", input);
        } catch (Exception e) {
            log.error("Erreur lors du traitement de {}", input, e);
        }
        
        log.info("Traitement termine");
    }
}
```

| Niveau | Quand l'utiliser |
|--------|-----------------|
| `ERROR` | Erreur grave, le service ne peut pas continuer |
| `WARN` | Situation anormale mais recuperable |
| `INFO` | Evenements importants du cycle de vie |
| `DEBUG` | Details utiles en developpement (desactive en prod) |
| `TRACE` | Trace ultra-fine (tres rare) |

---

## Premier projet complet avec Maven

**Maven** est le gestionnaire de dependances et build system standard en Java (equivalent de `pip` + `Makefile` en [[01 - Introduction a Python|Python]] — pour la comparaison avec les systemes de build bas niveau, voir [[11 - Makefiles]]).

### Structure d'un projet Maven

```
mon-projet/
├── pom.xml                    # Configuration Maven (dependances, plugins)
└── src/
    ├── main/
    │   └── java/
    │       └── com/holberton/
    │           └── App.java   # Code source
    └── test/
        └── java/
            └── com/holberton/
                └── AppTest.java # Tests
```

### pom.xml minimal

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Identifiant unique du projet -->
    <groupId>com.holberton</groupId>
    <artifactId>introduction-java</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <properties>
        <!-- Version Java cible -->
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    
    <dependencies>
        <!-- Logging -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.4.14</version>
        </dependency>
        
        <!-- Tests -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

```bash
# Commandes Maven essentielles
mvn compile          # Compiler le projet
mvn test             # Executer les tests
mvn package          # Creer le JAR
mvn clean package    # Nettoyer + compiler + packager
mvn clean install    # Installer dans le repo local Maven

# Creer un nouveau projet depuis un archetype
mvn archetype:generate \
  -DgroupId=com.holberton \
  -DartifactId=mon-projet \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DarchetypeVersion=1.4
```

---

## Programme complet : Calculatrice de notes

Voici un programme complet integrant tous les concepts vus dans ce cours :

```java
package com.holberton.java;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;

/**
 * Calculatrice de notes pour une promotion Holberton.
 * Illustre : types, String, ArrayList, methodes, exceptions, switch expression.
 */
public class CalculatriceNotes {
    
    private static final double NOTE_MIN = 0.0;
    private static final double NOTE_MAX = 20.0;
    
    // Valide et ajoute une note a la liste
    public static void ajouterNote(List<Double> notes, double note) {
        if (note < NOTE_MIN || note > NOTE_MAX) {
            throw new IllegalArgumentException(
                "Note invalide : " + note + " (attendu entre " + NOTE_MIN + " et " + NOTE_MAX + ")"
            );
        }
        notes.add(note);
    }
    
    // Calcule la moyenne
    public static double calculerMoyenne(List<Double> notes) {
        if (notes.isEmpty()) {
            throw new IllegalStateException("Impossible de calculer la moyenne : aucune note");
        }
        double somme = 0;
        for (double note : notes) {
            somme += note;
        }
        return somme / notes.size();
    }
    
    // Retourne la mention selon la moyenne
    public static String getMention(double moyenne) {
        return switch ((int) (moyenne / 2)) {
            case 10 -> "Mention Parfaite";
            case 9  -> "Tres Bien";
            case 8  -> "Bien";
            case 7  -> "Assez Bien";
            case 6  -> "Passable";
            default -> moyenne >= NOTE_MIN ? "Insuffisant" : "Invalide";
        };
    }
    
    public static void main(String[] args) {
        List<Double> notes = new ArrayList<>();
        Scanner scanner = new Scanner(System.in);
        
        System.out.println("=== Calculatrice de notes Holberton ===");
        System.out.println("Entrez vos notes (0-20). Tapez 'fin' pour terminer.");
        
        while (scanner.hasNextLine()) {
            String saisie = scanner.nextLine().trim();
            
            if (saisie.equalsIgnoreCase("fin")) {
                break;
            }
            
            try {
                double note = Double.parseDouble(saisie);
                ajouterNote(notes, note);
                System.out.println("Note ajoutee : " + note);
            } catch (NumberFormatException e) {
                System.out.println("Saisie invalide, entrez un nombre ou 'fin'");
            } catch (IllegalArgumentException e) {
                System.out.println("Erreur : " + e.getMessage());
            }
        }
        
        if (!notes.isEmpty()) {
            Collections.sort(notes);
            double moyenne = calculerMoyenne(notes);
            String mention = getMention(moyenne);
            
            StringBuilder rapport = new StringBuilder();
            rapport.append("\n=== Rapport ===\n");
            rapport.append("Nombre de notes : ").append(notes.size()).append("\n");
            rapport.append("Notes triees : ").append(notes).append("\n");
            rapport.append(String.format("Moyenne : %.2f/20%n", moyenne));
            rapport.append("Note la plus basse : ").append(notes.get(0)).append("\n");
            rapport.append("Note la plus haute : ").append(notes.get(notes.size() - 1)).append("\n");
            rapport.append("Mention : ").append(mention);
            
            System.out.println(rapport.toString());
        }
        
        scanner.close();
    }
}
```

---

## Exercices pratiques

> [!tip] Methode recommandee
> Creez un projet Maven pour ces exercices. Testez chaque exercice avec `mvn compile && mvn exec:java`.

### Exercice 1 — FizzBuzz Java

Ecrire un programme Java qui affiche les nombres de 1 a 100 en remplacant les multiples de 3 par "Fizz", les multiples de 5 par "Buzz", et les multiples des deux par "FizzBuzz". Utilisez le switch expression de Java 14+.

### Exercice 2 — Analyseur de chaines

Ecrire une methode `analyserTexte(String texte)` qui retourne un rapport sous forme de String contenant : le nombre de mots, le nombre de voyelles, le mot le plus long, et le texte en majuscules. Gerez le cas `null` avec une exception appropriee.

### Exercice 3 — Tableau de temperatures

Creer un programme qui :
1. Stocke dans un `ArrayList<Double>` les temperatures de la semaine (saisie utilisateur)
2. Calcule min, max, moyenne
3. Affiche un graphique ASCII (une barre `*` par degre)
4. Lance une `IllegalArgumentException` si une temperature est hors de [-50, 60]

### Exercice 4 — Convertisseur avec varargs

Implementer une classe `Convertisseur` avec des methodes surchargees :
- `convertir(double celsius)` -> Fahrenheit
- `convertir(double valeur, String unite)` -> conversion selon l'unite ("F", "K", "R")
- `convertirTous(double... temperatures)` -> retourne `double[]` de toutes les conversions

### Exercice 5 — Projet Maven complet

Creer un projet Maven `carnet-adresses` avec :
- Une classe `Contact` (nom, prenom, email, telephone)
- Une classe `CarnetAdresses` avec `List<Contact>` et methodes : ajouter, supprimer, rechercher
- La recherche leve une `ContactNotFoundException` (exception personnalisee) si rien n'est trouve
- Un `main` interactif avec Scanner
- Un fichier `pom.xml` correct avec Java 21

---

## Notes liées

- [[01 - Introduction a Python]] — comparaison syntaxe, types, modules Python vs Java
- [[01 - Introduction au C et Compilation]] — comparaison compilation, types, gestion memoire
- [[05 - Gestion des Erreurs et Fichiers]] — exceptions Python a comparer avec try/catch Java
- [[02 - Structures de Donnees Python]] — listes Python vs ArrayList Java
- [[11 - Makefiles]] — build systems bas niveau vs Maven
- [[02 - Java POO et Collections]] — suite : POO, heritage, Collections Framework
- [[03 - Spring Boot et API REST]] — application web complete en Java
- [[01 - Tests Unitaires et TDD]] — TDD et JUnit 5 pour les tests Java
