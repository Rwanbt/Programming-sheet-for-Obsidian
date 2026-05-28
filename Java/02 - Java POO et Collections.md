# Java POO et Collections

## La Programmation Orientee Objet en Java

Java est un langage **exclusivement oriente objet** : tout code (hormis les types primitifs) vit dans une classe. Contrairement a [[01 - Introduction a Python|Python]] ou la POO est optionnelle (voir [[03 - POO en Python]]), en Java elle est obligatoire.

> [!tip] Analogie avec ce que vous connaissez
> En [[01 - Introduction au C et Compilation|C]], vous utilisez des `struct` pour regrouper des donnees, et des fonctions qui operent dessus. En [[01 - Introduction a Python|Python]], une `class` regroupe donnees et comportements (voir [[03 - POO en Python]]). Java fait pareil, mais avec des regles plus strictes : tout est explicitement type, les niveaux d'acces sont declares, et le compilateur verifie la coherence avant meme de lancer le programme.

---

## Classes et Objets

### Anatomie d'une classe Java

```java
package com.holberton.poo;

/**
 * Classe representant un etudiant Holberton.
 * Illustre la structure complete d'une classe Java.
 */
public class Etudiant {
    
    // ========================
    //  CHAMPS (attributs)
    // ========================
    
    // Constante de classe (static final = partagee par toutes les instances)
    private static final int PROMOTION_ACTUELLE = 2025;
    
    // Compteur de classe (partage entre toutes les instances)
    private static int nombreEtudiants = 0;
    
    // Champs d'instance (prives par convention)
    private String prenom;
    private String nom;
    private int age;
    private double moyenne;
    
    // ========================
    //  CONSTRUCTEURS
    // ========================
    
    // Constructeur principal
    public Etudiant(String prenom, String nom, int age) {
        this.prenom = prenom;  // this = l'instance courante
        this.nom = nom;
        this.age = age;
        this.moyenne = 0.0;
        nombreEtudiants++;     // mise a jour du compteur de classe
    }
    
    // Constructeur par defaut (age = 18 si non specifie)
    public Etudiant(String prenom, String nom) {
        this(prenom, nom, 18);  // delegation au constructeur principal
    }
    
    // ========================
    //  METHODES
    // ========================
    
    // Methode d'instance
    public void afficherInfos() {
        System.out.printf("Etudiant : %s %s, age %d ans, moyenne %.1f/20%n",
                          prenom, nom, age, moyenne);
    }
    
    // Methode de classe (static)
    public static int getNombreEtudiants() {
        return nombreEtudiants;
    }
    
    // toString : utilise par println() automatiquement
    @Override
    public String toString() {
        return String.format("Etudiant{%s %s, %d ans}", prenom, nom, age);
    }
}
```

### Instanciation et utilisation

```java
// Creer des objets avec new
Etudiant alice = new Etudiant("Alice", "Martin", 22);
Etudiant bob   = new Etudiant("Bob", "Dupont");  // age = 18

alice.afficherInfos();
// Etudiant : Alice Martin, age 22 ans, moyenne 0.0/20

System.out.println(alice);   // appelle toString() automatiquement
// Etudiant{Alice Martin, 22 ans}

System.out.println(Etudiant.getNombreEtudiants()); // 2 (methode de classe)
```

---

## Encapsulation : getters et setters

L'encapsulation protege les donnees internes en les rendant `private` et en exposant des methodes publiques controlees :

```java
public class CompteBancaire {
    
    private String titulaire;
    private double solde;
    private String numeroCompte;
    
    public CompteBancaire(String titulaire, String numeroCompte, double soldeInitial) {
        this.titulaire = titulaire;
        this.numeroCompte = numeroCompte;
        // Validation dans le setter
        setSolde(soldeInitial);
    }
    
    // Getter simple (lecture)
    public String getTitulaire() {
        return titulaire;
    }
    
    // Getter avec logique (masquage partiel du numero)
    public String getNumeroCompte() {
        return "****" + numeroCompte.substring(numeroCompte.length() - 4);
    }
    
    public double getSolde() {
        return solde;
    }
    
    // Setter avec validation
    private void setSolde(double solde) {
        if (solde < 0) {
            throw new IllegalArgumentException("Le solde ne peut pas etre negatif");
        }
        this.solde = solde;
    }
    
    // Methodes metier (pas juste des setters mecaniques)
    public void deposer(double montant) {
        if (montant <= 0) throw new IllegalArgumentException("Montant invalide");
        this.solde += montant;
        System.out.printf("Depot de %.2f€. Nouveau solde : %.2f€%n", montant, solde);
    }
    
    public void retirer(double montant) {
        if (montant <= 0) throw new IllegalArgumentException("Montant invalide");
        if (montant > solde) throw new IllegalStateException("Solde insuffisant");
        this.solde -= montant;
    }
}
```

> [!warning] Getters/Setters mecaniques : le piege
> Evitez de creer systematiquement un getter ET un setter pour chaque champ — ca casse l'encapsulation autant que de rendre le champ `public`. Exposez uniquement ce dont les clients ont besoin. Preferez des **methodes metier** (`deposer`, `retirer`) plutot que `setSolde()`.

---

## Heritage

### extends et super

```java
// Classe parente (base)
public class Vehicule {
    
    protected String marque;  // protected : accessible aux sous-classes
    protected int vitesseMax;
    
    public Vehicule(String marque, int vitesseMax) {
        this.marque = marque;
        this.vitesseMax = vitesseMax;
    }
    
    public void demarrer() {
        System.out.println(marque + " demarre.");
    }
    
    public String getDescription() {
        return marque + " (max " + vitesseMax + " km/h)";
    }
    
    @Override
    public String toString() {
        return "Vehicule{" + marque + "}";
    }
}

// Classe enfant (derivee)
public class Voiture extends Vehicule {
    
    private int nombrePortes;
    
    public Voiture(String marque, int vitesseMax, int nombrePortes) {
        super(marque, vitesseMax);  // OBLIGATOIRE : appel du constructeur parent
        this.nombrePortes = nombrePortes;
    }
    
    // Surcharge de la methode parente
    @Override
    public void demarrer() {
        super.demarrer();  // appel de la version parente
        System.out.println("Moteur de voiture enclenche.");
    }
    
    // Surchage de getDescription
    @Override
    public String getDescription() {
        return super.getDescription() + ", " + nombrePortes + " portes";
    }
    
    // Nouvelle methode specifique aux voitures
    public void klaxonner() {
        System.out.println(marque + " : TUT TUT !");
    }
}

// Classe petit-enfant
public class VoitureElectrique extends Voiture {
    
    private int autonomieKm;
    
    public VoitureElectrique(String marque, int vitesseMax, int autonomieKm) {
        super(marque, vitesseMax, 4);
        this.autonomieKm = autonomieKm;
    }
    
    @Override
    public void demarrer() {
        System.out.println(marque + " : demarrage electrique silencieux.");
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", autonomie " + autonomieKm + " km";
    }
}
```

### @Override : annotation importante

L'annotation `@Override` est **fortement recommandee** sur toute methode qui en surcharge une autre. Elle force le compilateur a verifier que la methode existe bien dans la classe parente — protege contre les fautes de frappe :

```java
// Sans @Override : si on ecrit "demarerr" par erreur, c'est une NOUVELLE methode
public void demarerr() { ... }  // Silencieux, mais bug !

// Avec @Override : le compilateur detecte l'erreur
@Override
public void demarerr() { ... }  // ERREUR de compilation : methode introuvable dans le parent
```

---

## Polymorphisme

Le polymorphisme permet de traiter des objets de types differents de facon uniforme via un type commun :

```java
// Un tableau de Vehicule peut contenir des Voiture, VoitureElectrique, etc.
List<Vehicule> garage = new ArrayList<>();
garage.add(new Voiture("Renault", 180, 5));
garage.add(new VoitureElectrique("Tesla", 250, 600));
garage.add(new Vehicule("Velo", 30));  // vehicule de base

// Polymorphisme : la BONNE methode est appelee selon le TYPE REEL de l'objet
for (Vehicule v : garage) {
    v.demarrer();  // appelle demarrer() de Voiture, VoitureElectrique ou Vehicule
}

// instanceof : verifier le type reel (utiliser avec moderation)
for (Vehicule v : garage) {
    if (v instanceof Voiture voiture) {  // pattern matching Java 16+
        voiture.klaxonner();  // ok, car on sait que c'est une Voiture
    }
}
```

> [!tip] Comparaison avec [[03 - POO en Python|Python]]
> En [[03 - POO en Python|Python]], le polymorphisme est implicite (duck typing) : un objet peut appeler une methode si elle existe, sans notion de hierarchie obligatoire. En Java, le polymorphisme passe par l'heritage ou les interfaces — la hierarchie est explicite et verifiee par le compilateur. Pour les patterns avances (design patterns, SOLID), voir [[04 - POO Avancee]].

---

## Classes abstraites

Une **classe abstraite** ne peut pas etre instanciee — elle sert de modele commun avec du code partage et des methodes a implementer obligatoirement :

```java
// abstract : ne peut pas etre instanciee directement
public abstract class Forme {
    
    protected String couleur;
    
    public Forme(String couleur) {
        this.couleur = couleur;
    }
    
    // Methode abstraite : DOIT etre implementee par les sous-classes
    public abstract double calculerAire();
    public abstract double calculerPerimetre();
    
    // Methode concrete : partagee par toutes les formes
    public void afficher() {
        System.out.printf("%s %s : aire=%.2f, perimetre=%.2f%n",
                          couleur, getClass().getSimpleName(),
                          calculerAire(), calculerPerimetre());
    }
}

public class Cercle extends Forme {
    
    private double rayon;
    
    public Cercle(String couleur, double rayon) {
        super(couleur);
        this.rayon = rayon;
    }
    
    @Override
    public double calculerAire() {
        return Math.PI * rayon * rayon;
    }
    
    @Override
    public double calculerPerimetre() {
        return 2 * Math.PI * rayon;
    }
}

public class Rectangle extends Forme {
    
    private double largeur, hauteur;
    
    public Rectangle(String couleur, double largeur, double hauteur) {
        super(couleur);
        this.largeur = largeur;
        this.hauteur = hauteur;
    }
    
    @Override
    public double calculerAire() {
        return largeur * hauteur;
    }
    
    @Override
    public double calculerPerimetre() {
        return 2 * (largeur + hauteur);
    }
}
```

---

## Interfaces

Une **interface** definit un contrat (un ensemble de methodes) sans implementation (sauf les `default` methods depuis Java 8). Une classe peut implementer plusieurs interfaces (contrairement a l'heritage single en Java) :

```java
// Interface = contrat
public interface Serialisable {
    String toJson();
    static Serialisable fromJson(String json) {  // methode statique d'interface
        throw new UnsupportedOperationException("Non implemente");
    }
}

public interface Comparable<T> {
    int compareTo(T autre);
}

// Default method : implementation par defaut (depuis Java 8)
public interface Loggable {
    String getNom();  // abstraite
    
    default void logInfo(String message) {  // implementation par defaut
        System.out.println("[INFO][" + getNom() + "] " + message);
    }
    
    default void logError(String message) {
        System.err.println("[ERROR][" + getNom() + "] " + message);
    }
}

// Une classe peut implementer plusieurs interfaces
public class Produit implements Serialisable, Loggable {
    
    private String nom;
    private double prix;
    
    public Produit(String nom, double prix) {
        this.nom = nom;
        this.prix = prix;
    }
    
    @Override
    public String toJson() {
        return String.format("{\"nom\":\"%s\",\"prix\":%.2f}", nom, prix);
    }
    
    @Override
    public String getNom() {
        return nom;
    }
}
```

### Functional Interfaces : les interfaces a une methode

Une **functional interface** est une interface avec une seule methode abstraite. Elle peut etre representee par une **lambda expression** :

```java
// @FunctionalInterface : annotation qui le signale au compilateur
@FunctionalInterface
public interface Calculatrice {
    double calculer(double a, double b);
}

// Utilisation avec lambda
Calculatrice addition      = (a, b) -> a + b;
Calculatrice multiplication = (a, b) -> a * b;
Calculatrice puissance     = (a, b) -> Math.pow(a, b);

System.out.println(addition.calculer(3, 4));       // 7.0
System.out.println(multiplication.calculer(3, 4)); // 12.0
```

Interfaces fonctionnelles standard de `java.util.function` :

| Interface | Signature | Usage |
|-----------|-----------|-------|
| `Function<T, R>` | `R apply(T t)` | Transformation |
| `Predicate<T>` | `boolean test(T t)` | Filtrage |
| `Consumer<T>` | `void accept(T t)` | Action sans retour |
| `Supplier<T>` | `T get()` | Creation sans parametre |
| `BiFunction<T, U, R>` | `R apply(T t, U u)` | Transformation a 2 entrees |

---

## Records (Java 16+)

Les **records** sont des classes de donnees immutables generees automatiquement. Ils remplacent les classes "POJO" verbeux :

```java
// AVANT les records (Java 15 et avant)
public final class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    public int x() { return x; }
    public int y() { return y; }
    
    @Override public boolean equals(Object o) { ... }
    @Override public int hashCode() { ... }
    @Override public String toString() { return "Point[x=" + x + ", y=" + y + "]"; }
}

// AVEC les records (Java 16+) : 1 ligne !
public record Point(int x, int y) {}

// Utilisation identique
Point p = new Point(3, 4);
System.out.println(p.x());         // 3
System.out.println(p);             // Point[x=3, y=4]
System.out.println(p.equals(new Point(3, 4))); // true

// Records avec validation dans le constructeur compact
public record Temperature(double valeur, String unite) {
    // Constructeur compact : validations sans boilerplate
    public Temperature {
        if (!unite.equals("C") && !unite.equals("F") && !unite.equals("K")) {
            throw new IllegalArgumentException("Unite invalide : " + unite);
        }
    }
}
```

---

## Generics

Les **generics** permettent d'ecrire du code parametrable par le type — comme les templates C++ mais plus safe (pattern analogue aux generics Python avec `typing.Generic`, voir [[04 - POO Avancee]]) :

```java
// Classe generique : T est le type parametre
public class Boite<T> {
    
    private T contenu;
    
    public Boite(T contenu) {
        this.contenu = contenu;
    }
    
    public T ouvrir() {
        return contenu;
    }
    
    public boolean estVide() {
        return contenu == null;
    }
    
    @Override
    public String toString() {
        return "Boite[" + contenu + "]";
    }
}

// Utilisation avec differents types
Boite<String>  boiteTexte  = new Boite<>("Holberton");
Boite<Integer> boiteNombre = new Boite<>(42);
Boite<Point>   boitePoint  = new Boite<>(new Point(3, 4));

String texte   = boiteTexte.ouvrir();   // pas de cast necessaire !
Integer nombre = boiteNombre.ouvrir();

// Methode generique
public static <T extends Comparable<T>> T maximum(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}

System.out.println(maximum(10, 20));         // 20
System.out.println(maximum("Alice", "Bob")); // Bob (ordre alphabetique)

// Wildcards (jokers)
public static void afficherListe(List<?> liste) {  // ? = n'importe quel type
    for (Object element : liste) {
        System.out.println(element);
    }
}

// Bounded wildcards
public static double somme(List<? extends Number> nombres) {  // Number ou sous-type
    return nombres.stream().mapToDouble(Number::doubleValue).sum();
}
```

---

## Collections Framework

Java fournit un ensemble riche de structures de donnees via le **Collections Framework** (pour les equivalents Python : listes → [[02 - Structures de Donnees Python]], dictionnaires → [[03 - Tables de Hachage]]) :

```
Collection (interface)
├── List (interface) — elements ordonnes, doublons autorises
│   ├── ArrayList — tableau dynamique, acces rapide par index
│   └── LinkedList — liste chainee, insertion/suppression rapides
├── Set (interface) — elements uniques, pas d'ordre garanti
│   ├── HashSet — unicite via hashCode(), ordre non garanti
│   ├── LinkedHashSet — unicite + ordre d'insertion preserve
│   └── TreeSet — unicite + tri naturel ou par Comparator
└── Queue / Deque — file, pile
    ├── ArrayDeque — pile ou file efficace
    └── PriorityQueue — file avec priorite

Map (interface) — paires cle -> valeur (pas une Collection)
├── HashMap — acces rapide, ordre non garanti
├── LinkedHashMap — ordre d'insertion preserve
└── TreeMap — cles triees
```

### List : ArrayList et LinkedList

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

// ArrayList : choix par defaut (acces O(1) par index)
List<String> langages = new ArrayList<>();
langages.add("Java");
langages.add("Python");
langages.add("Rust");
langages.add(1, "C");         // insere a l'index 1

langages.get(0);              // "Java" — O(1)
langages.remove("Python");    // par valeur
langages.remove(0);           // par index
langages.size();              // taille actuelle
langages.contains("Rust");    // true
langages.indexOf("Rust");     // index (ou -1 si absent)

// LinkedList : utile si beaucoup d'insertions/suppressions en milieu de liste
LinkedList<Integer> file = new LinkedList<>();
file.addFirst(1);   // ajoute en tete
file.addLast(2);    // ajoute en queue
file.removeFirst(); // retire de la tete

// Listes immutables (Java 9+)
List<String> constantes = List.of("A", "B", "C");  // non modifiable
```

### Set : ensembles sans doublons

```java
import java.util.HashSet;
import java.util.Set;
import java.util.TreeSet;

// HashSet : performance maximale, ordre aleatoire
Set<String> visiteurs = new HashSet<>();
visiteurs.add("Alice");
visiteurs.add("Bob");
visiteurs.add("Alice");  // ignore — deja present
System.out.println(visiteurs.size());         // 2
System.out.println(visiteurs.contains("Bob")); // true

// TreeSet : elements tries automatiquement
Set<Integer> notesTri = new TreeSet<>();
notesTri.add(15);
notesTri.add(8);
notesTri.add(20);
System.out.println(notesTri);  // [8, 15, 20]

// Operations ensemblistes
Set<String> setA = new HashSet<>(Set.of("a", "b", "c"));
Set<String> setB = new HashSet<>(Set.of("b", "c", "d"));

// Intersection
setA.retainAll(setB);  // setA devient {b, c}

// Union
setA.addAll(setB);

// Difference
setA.removeAll(setB);
```

### Map : dictionnaires

```java
import java.util.HashMap;
import java.util.Map;
import java.util.TreeMap;

// HashMap : le dictionnaire [[01 - Introduction a Python|Python]] de Java (voir [[03 - Tables de Hachage]] pour la theorie)
Map<String, Integer> notes = new HashMap<>();
notes.put("Alice", 18);
notes.put("Bob", 14);
notes.put("Charlie", 19);
notes.put("Alice", 20);  // ecrase la valeur precedente

// Acces
notes.get("Alice");                    // 20
notes.getOrDefault("David", 0);       // 0 (valeur par defaut si cle absente)
notes.containsKey("Bob");             // true
notes.containsValue(19);              // true

// Iteration sur une Map
for (Map.Entry<String, Integer> entree : notes.entrySet()) {
    System.out.println(entree.getKey() + " : " + entree.getValue());
}

// forEach avec lambda (plus concis)
notes.forEach((nom, note) -> 
    System.out.printf("%s : %d/20%n", nom, note)
);

// putIfAbsent, computeIfAbsent
notes.putIfAbsent("Eve", 15);  // n'ecrase pas si deja present

// computeIfAbsent : utile pour grouper
Map<String, List<String>> groupes = new HashMap<>();
groupes.computeIfAbsent("Groupe A", k -> new ArrayList<>()).add("Alice");
groupes.computeIfAbsent("Groupe A", k -> new ArrayList<>()).add("Bob");

// TreeMap : cles triees
Map<String, Integer> notesTriees = new TreeMap<>(notes);
// Affiche par ordre alphabetique des noms
```

---

## Streams API

Les **Streams** permettent de traiter des collections de facon **fonctionnelle** et **declarative**. Equivalent des comprehensions et fonctions [[01 - Introduction a Python|Python]] (`map`, `filter`, `reduce`) — pour la programmation fonctionnelle asynchrone en Python, voir [[07 - Python Async et Concurrence]] :

```java
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

List<Integer> nombres = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// filter : garde les elements qui satisfont le predicat
List<Integer> pairs = nombres.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// [2, 4, 6, 8, 10]

// map : transforme chaque element
List<String> chaines = nombres.stream()
    .map(n -> "N" + n)
    .collect(Collectors.toList());
// ["N1", "N2", ..., "N10"]

// Chainer les operations (pipeline)
double moyenneCarresPairs = nombres.stream()
    .filter(n -> n % 2 == 0)        // garde les pairs
    .map(n -> n * n)                 // carre chacun
    .mapToInt(Integer::intValue)     // convertit en IntStream
    .average()                       // calcule la moyenne
    .orElse(0.0);                   // valeur par defaut si stream vide
// Resultat : 44.0

// reduce : aggrégation
int produit = nombres.stream()
    .reduce(1, (acc, n) -> acc * n);
// 3628800

// collect avec Collectors
List<String> prenoms = List.of("Alice", "Bob", "Charlie", "David", "Eve");

// Grouper par longueur du prenom
Map<Integer, List<String>> parLongueur = prenoms.stream()
    .collect(Collectors.groupingBy(String::length));
// {3: [Bob, Eve], 5: [Alice, David], 7: [Charlie]}

// Joindre en une chaine
String tousLesPrenoms = prenoms.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// "[Alice, Bob, Charlie, David, Eve]"

// Statistiques
IntSummaryStatistics stats = nombres.stream()
    .mapToInt(Integer::intValue)
    .summaryStatistics();
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Moy: " + stats.getAverage());

// sorted et distinct
List<Integer> doublons = List.of(3, 1, 4, 1, 5, 9, 2, 6, 5, 3);
List<Integer> uniques = doublons.stream()
    .distinct()
    .sorted()
    .collect(Collectors.toList());
// [1, 2, 3, 4, 5, 6, 9]
```

### Streams vs comprehensions Python

```python
# Python
carres_pairs = [x**2 for x in nombres if x % 2 == 0]
```

```java
// Java — equivalent Streams
List<Integer> carresPairs = nombres.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .collect(Collectors.toList());
```

---

## Optional : eviter NullPointerException

`Optional<T>` est un conteneur qui peut contenir une valeur ou rien — force a gerer explicitement l'absence de valeur :

```java
import java.util.Optional;

public class RechercheEtudiant {
    
    private Map<String, Integer> notes = new HashMap<>();
    
    // Retourne Optional.empty() si l'etudiant n'est pas trouve
    public Optional<Integer> getNoteEtudiant(String nom) {
        return Optional.ofNullable(notes.get(nom));
    }
    
    public void utiliser() {
        Optional<Integer> note = getNoteEtudiant("Alice");
        
        // Verifier si present
        if (note.isPresent()) {
            System.out.println("Note : " + note.get());
        }
        
        // Valeur par defaut
        int noteOuZero = note.orElse(0);
        
        // Valeur calculee par defaut
        int noteOuCalcule = note.orElseGet(() -> calculerNoteMoyenne());
        
        // Lancer une exception si absent
        int noteObligatoire = note.orElseThrow(
            () -> new RuntimeException("Etudiant non trouve")
        );
        
        // Map et filter sur Optional
        Optional<String> mention = note
            .filter(n -> n >= 10)
            .map(n -> n >= 16 ? "Tres Bien" : "Passable");
        
        // ifPresent : action si valeur presente
        note.ifPresent(n -> System.out.println("Note trouvee : " + n));
        
        // ifPresentOrElse (Java 9+)
        note.ifPresentOrElse(
            n -> System.out.println("Note : " + n),
            () -> System.out.println("Etudiant non inscrit")
        );
    }
}
```

---

## Lambdas et Method References

### Expressions lambda

Une **lambda** est une fonction anonyme — similaire aux lambdas Python et aux function pointers C :

```java
// Syntaxe : (parametres) -> corps
// Un parametre : parentheses optionnelles
Runnable r = () -> System.out.println("Hello");

// Avec parametre
Consumer<String> afficher = s -> System.out.println(s);

// Avec plusieurs parametres
BiFunction<Integer, Integer, Integer> somme = (a, b) -> a + b;

// Avec bloc d'instructions
Comparator<String> parLongueur = (s1, s2) -> {
    int diff = Integer.compare(s1.length(), s2.length());
    return diff != 0 ? diff : s1.compareTo(s2);
};

// Utilisation avec Collections.sort
List<String> noms = new ArrayList<>(List.of("Charlie", "Alice", "Bob"));
noms.sort((a, b) -> a.compareTo(b));  // tri alphabetique
noms.sort((a, b) -> a.length() - b.length());  // tri par longueur
```

### Method References : raccourcis de lambdas

Quand une lambda appelle simplement une methode existante, on peut utiliser une **reference de methode** avec `::` :

```java
List<String> prenoms = List.of("alice", "bob", "charlie");

// Lambda complete
prenoms.stream()
    .map(s -> s.toUpperCase())
    .forEach(s -> System.out.println(s));

// Equivalent avec method references
prenoms.stream()
    .map(String::toUpperCase)        // reference de methode d'instance
    .forEach(System.out::println);  // reference de methode statique

// Differentes formes de method references
// 1. Methode statique : ClassName::staticMethod
Function<String, Integer> parser = Integer::parseInt;

// 2. Methode d'instance (object specifique) : instance::method
String prefixe = "Java ";
Function<String, String> ajouter = prefixe::concat;

// 3. Methode d'instance (type) : ClassName::instanceMethod
Function<String, String> majuscule = String::toUpperCase;

// 4. Constructeur : ClassName::new
Supplier<ArrayList<String>> creerListe = ArrayList::new;
```

---

## Exemple complet : Systeme de gestion d'etudiants

```java
package com.holberton.poo;

import java.util.*;
import java.util.stream.Collectors;

public record Etudiant(String prenom, String nom, List<Double> notes) {
    
    // Validation dans le constructeur compact du record
    public Etudiant {
        Objects.requireNonNull(prenom, "Le prenom est obligatoire");
        Objects.requireNonNull(nom, "Le nom est obligatoire");
        notes = new ArrayList<>(notes);  // defensive copy
    }
    
    // Constructeur sans notes
    public Etudiant(String prenom, String nom) {
        this(prenom, nom, new ArrayList<>());
    }
    
    // Methodes metier sur le record
    public double getMoyenne() {
        return notes.stream()
            .mapToDouble(Double::doubleValue)
            .average()
            .orElse(0.0);
    }
    
    public String getMention() {
        double moy = getMoyenne();
        if (moy >= 16) return "Tres Bien";
        if (moy >= 14) return "Bien";
        if (moy >= 12) return "Assez Bien";
        if (moy >= 10) return "Passable";
        return "Insuffisant";
    }
    
    public String getNomComplet() {
        return prenom + " " + nom;
    }
}

// Interface pour le rapport
@FunctionalInterface
interface GenerateurRapport {
    String generer(List<Etudiant> etudiants);
}

public class PromotionManager {
    
    private final List<Etudiant> etudiants = new ArrayList<>();
    
    public void ajouter(Etudiant e) {
        etudiants.add(Objects.requireNonNull(e));
    }
    
    public Optional<Etudiant> trouver(String prenom, String nom) {
        return etudiants.stream()
            .filter(e -> e.prenom().equalsIgnoreCase(prenom)
                      && e.nom().equalsIgnoreCase(nom))
            .findFirst();
    }
    
    public List<Etudiant> getRecu(double seuilMinimum) {
        return etudiants.stream()
            .filter(e -> e.getMoyenne() >= seuilMinimum)
            .sorted(Comparator.comparingDouble(Etudiant::getMoyenne).reversed())
            .collect(Collectors.toList());
    }
    
    public Map<String, List<Etudiant>> grouperParMention() {
        return etudiants.stream()
            .collect(Collectors.groupingBy(Etudiant::getMention));
    }
    
    public String rapport(GenerateurRapport generateur) {
        return generateur.generer(etudiants);
    }
    
    public static void main(String[] args) {
        PromotionManager promo = new PromotionManager();
        
        promo.ajouter(new Etudiant("Alice", "Martin", List.of(18.0, 16.5, 19.0)));
        promo.ajouter(new Etudiant("Bob", "Dupont", List.of(12.0, 9.5, 11.0)));
        promo.ajouter(new Etudiant("Charlie", "Durand", List.of(15.0, 14.5, 16.0)));
        
        // Rapport avec lambda
        String rapportSimple = promo.rapport(etudiants -> {
            StringBuilder sb = new StringBuilder("=== Rapport Promotion ===\n");
            etudiants.forEach(e -> sb.append(
                String.format("  %s : %.1f/20 (%s)%n",
                              e.getNomComplet(), e.getMoyenne(), e.getMention())
            ));
            return sb.toString();
        });
        
        System.out.println(rapportSimple);
        
        // Etudiants recus (moyenne >= 10)
        System.out.println("Recus :");
        promo.getRecu(10.0).forEach(e -> 
            System.out.println("  " + e.getNomComplet() + " - " + e.getMention())
        );
        
        // Repartition par mention
        System.out.println("\nRepartition par mention :");
        promo.grouperParMention().forEach((mention, liste) ->
            System.out.println("  " + mention + " : " + 
                liste.stream().map(Etudiant::getNomComplet).collect(Collectors.joining(", ")))
        );
    }
}
```

---

## Exercices pratiques

> [!tip] Conseil
> Ces exercices progressent en difficulte. Commencez par implementer sans Streams, puis refactorisez avec les Streams et lambdas.

### Exercice 1 — Hierarchie de formes

Implementer la hierarchie `Forme` -> `Cercle`, `Rectangle`, `Triangle` avec :
- Classe abstraite `Forme` avec `calculerAire()` et `calculerPerimetre()` abstraites
- Methode concrete `comparer(Forme autre)` qui retourne laquelle a la plus grande aire
- Une liste `List<Forme>` avec les 3 types, tri par aire avec Streams
- Affichage de la forme avec l'aire maximale

### Exercice 2 — Bibliotheque avec generics

Creer une classe generique `Catalogue<T>` qui stocke des elements typés avec :
- Methode `ajouter(T element)` et `supprimer(T element)`
- Methode `rechercher(Predicate<T> critere)` qui retourne `List<T>`
- Methode `trier(Comparator<T> ordre)` qui retourne une vue triee
- Tester avec `Catalogue<String>` (livres) et `Catalogue<Integer>` (annees)

### Exercice 3 — Streams avances

A partir d'une liste de 20 etudiants avec notes :
1. Filtrer ceux dont la moyenne depasse 12
2. Grouper par mention (`Collectors.groupingBy`)
3. Calculer la moyenne de la promotion entiere
4. Trouver l'etudiant avec la meilleure et la plus mauvaise moyenne (`Collectors.maxBy/minBy`)
5. Construire une Map `nom -> moyenne` (`Collectors.toMap`)

### Exercice 4 — Design pattern Observer avec interfaces

Implementer le pattern Observer (voir [[04 - POO Avancee]] pour les design patterns en Python) :
- Interface `Observateur<T>` avec methode `recevoir(T evenement)`
- Interface `Observable<T>` avec `abonner(Observateur<T> obs)`, `desabonner(...)`, `notifier(T event)`
- Classe `NoteurEvenements` qui implemente `Observable<String>`
- Plusieurs observateurs lambda qui loggent, filtrent, ou comptent les evenements

### Exercice 5 — Records et Optional

Creer un systeme de carnet de contacts :
- Record `Contact(String nom, String email, Optional<String> telephone)`
- Classe `CarnetContacts` avec `Map<String, Contact>`
- Methode `rechercher(String motCle)` : retourne `List<Contact>` contenant le motCle dans nom ou email
- Methode `exporterCsv()` avec Streams qui produit `"nom,email,telephone\n..."` en une ligne de code

### Exercice 6 — Mini framework de validation

En utilisant les functional interfaces :
- Interface `Validateur<T>` avec `boolean valider(T valeur)` et `String getMessage()`
- Methode `and(Validateur<T> autre)` retournant un nouveau Validateur combinant les deux
- Creer des validateurs : `notBlank`, `longueurMin(int)`, `correspondAuPattern(String regex)`
- Valider un formulaire de creation de compte : email, mot de passe, nom d'utilisateur

---

## Notes liées

- [[01 - Introduction a Java]] — prerequis : types, syntaxe, ArrayList
- [[03 - POO en Python]] — POO Python a comparer avec Java : heritage, polymorphisme, duck typing
- [[04 - POO Avancee]] — design patterns (Observer, Strategy, Factory), principes SOLID
- [[02 - Structures de Donnees Python]] — listes et dictionnaires Python vs Collections Java
- [[03 - Tables de Hachage]] — theorie des HashMap et HashSet
- [[07 - Python Async et Concurrence]] — programmation fonctionnelle et lambdas Python vs Streams Java
- [[03 - Spring Boot et API REST]] — suite : application web Java avec Spring Boot
- [[01 - Tests Unitaires et TDD]] — tester les services et collections avec JUnit 5
