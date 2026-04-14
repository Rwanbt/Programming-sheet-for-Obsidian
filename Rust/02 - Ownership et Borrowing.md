# Ownership et Borrowing

L'ownership (possession) est **le** concept fondamental de Rust. C'est ce qui permet au langage de garantir la securite memoire a la compilation, sans garbage collector. Si vous comprenez ce chapitre, vous comprenez Rust. Si vous ne le comprenez pas, rien d'autre n'aura de sens.

Pour un developpeur C, l'ownership est la reponse a une question que vous vous posez depuis toujours : **comment gerer la memoire correctement, a chaque fois, sans exception ?** En C, la reponse est "faites attention". En Rust, la reponse est "le compilateur le fait pour vous".

---

## Le probleme que Rust resout

### Les bugs memoire du C

En C, la gestion memoire est manuelle. Et les humains font des erreurs. Voici les categories de bugs que Rust elimine **a la compilation** :

#### 1. Dangling pointer (pointeur pendant)

```c
// C : le compilateur ne dit RIEN
#include <stdlib.h>
#include <stdio.h>

int *creer_nombre(void) {
    int x = 42;
    return &x;  // DANGER : x est detruit quand la fonction retourne
}               // Le pointeur retourne pointe vers de la memoire invalide

int main(void) {
    int *p = creer_nombre();
    printf("%d\n", *p);  // Comportement indefini ! Peut afficher 42,
                          // ou 0, ou crasher, ou lancer des missiles
    return 0;
}
```

#### 2. Double free

```c
// C : compile sans probleme, crash a l'execution (parfois)
#include <stdlib.h>

int main(void) {
    int *p = malloc(sizeof(int));
    *p = 42;
    free(p);
    free(p);  // Double free ! Corruption du heap, crash, faille de securite
    return 0;
}
```

#### 3. Use-after-free

```c
// C : une des failles de securite les plus exploitees
#include <stdlib.h>
#include <stdio.h>

int main(void) {
    int *p = malloc(sizeof(int));
    *p = 42;
    free(p);
    printf("%d\n", *p);  // Use-after-free ! p pointe vers de la memoire liberee
    *p = 100;            // Ecrit dans de la memoire potentiellement reallouee
    return 0;
}
```

#### 4. Memory leak (fuite memoire)

```c
// C : le programme consomme de plus en plus de memoire
#include <stdlib.h>

void traiter(void) {
    int *data = malloc(1000 * sizeof(int));
    // ... utilisation de data ...
    if (erreur) {
        return;  // OUPS : on a oublie free(data) dans ce chemin !
    }
    free(data);
}
```

#### 5. Buffer overflow

```c
// C : la faille de securite classique
#include <string.h>

int main(void) {
    char buffer[10];
    strcpy(buffer, "cette chaine est beaucoup trop longue pour le buffer");
    // Ecrase la memoire adjacente, potentiellement l'adresse de retour
    // -> execution de code arbitraire (exploit)
    return 0;
}
```

> [!warning] Impact reel
> Ces bugs ne sont pas theoriques. Selon Microsoft, **~70% des vulnerabilites de securite** dans leurs produits sont des bugs de securite memoire. Google rapporte des chiffres similaires pour Chromium. Rust elimine ces categories entieres de bugs.

---

## Les 3 regles de l'ownership

Voici les trois regles fondamentales. Memorisez-les.

```
+-----------------------------------------------------------------------+
|                    LES 3 REGLES DE L'OWNERSHIP                        |
|                                                                       |
|  1. Chaque valeur en Rust a exactement UN proprietaire (owner)        |
|                                                                       |
|  2. Il ne peut y avoir qu'UN SEUL proprietaire a la fois              |
|                                                                       |
|  3. Quand le proprietaire sort du scope, la valeur est liberee (drop) |
+-----------------------------------------------------------------------+
```

> [!tip] Analogie
> Pensez a un livre physique. Un livre ne peut avoir qu'un seul possesseur a la fois. Vous pouvez le **donner** (move) a quelqu'un d'autre -- mais alors vous ne l'avez plus. Vous pouvez le **preter** (borrow) -- l'autre personne peut le lire, mais vous restez le proprietaire et vous le recupererez. Quand le proprietaire meurt (sort du scope), le livre est detruit.

### Regle 1 et 3 : un proprietaire, liberation automatique

```rust
fn main() {
    {
        let s = String::from("hello");  // s est le proprietaire de "hello"
        println!("{}", s);              // s est valide ici
    }  // s sort du scope -> Rust appelle automatiquement drop(s)
       // La memoire heap de "hello" est liberee ICI, pas besoin de free()

    // println!("{}", s);  // ERREUR : s n'existe plus
}
```

Comparaison avec C :

```c
// En C, VOUS devez liberer :
void exemple(void) {
    char *s = malloc(6);
    strcpy(s, "hello");
    printf("%s\n", s);
    free(s);            // Si vous oubliez cette ligne = memory leak
}                       // Si vous l'appelez deux fois = double free
```

> [!info] RAII
> Ce pattern s'appelle **RAII** (Resource Acquisition Is Initialization). Il vient du C++ mais Rust le rend obligatoire et infaillible. La ressource est acquise a la creation de la variable et liberee a sa destruction. Pas d'exception possible, pas d'oubli possible.

---

## Move semantics (semantique de deplacement)

C'est ici que Rust diverge radicalement du C. Quand vous assignez une valeur a une autre variable, il y a un **deplacement** (move), pas une copie.

### Le probleme en C

```c
// En C : deux pointeurs vers la meme memoire = DANGER
#include <stdlib.h>
#include <string.h>

int main(void) {
    char *s1 = malloc(6);
    strcpy(s1, "hello");

    char *s2 = s1;  // s2 pointe vers la MEME memoire que s1
                     // Les deux sont valides. Qui libere ? Quand ?

    free(s1);        // s1 libere la memoire
    printf("%s", s2); // Use-after-free ! s2 pointe vers de la memoire liberee
    free(s2);        // Double free !
    return 0;
}
```

```
Apres "char *s2 = s1;" en C :

Stack                   Heap
s1: [ptr] ------+
                 +----> [h|e|l|l|o|\0]
s2: [ptr] ------+
                        ^
                 DEUX pointeurs vers la MEME memoire
                 Qui libere ? -> Bug garanti
```

### La solution Rust : le move

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // MOVE : s1 est INVALIDE a partir d'ici

    // println!("{}", s1);  // ERREUR DE COMPILATION :
    //                      // "value borrowed here after move"
    println!("{}", s2);     // OK : s2 est le seul proprietaire
}
```

```
Avant le move :                    Apres let s2 = s1 :

Stack          Heap                Stack          Heap
s1: [ptr] ---> [h|e|l|l|o]        s1: [INVALIDE]
    [len:5]                        
    [cap:5]                        s2: [ptr] ---> [h|e|l|l|o]
                                       [len:5]
                                       [cap:5]

s1 est INVALIDE. Le compilateur refuse tout acces a s1.
Un seul proprietaire = un seul free = zero bugs.
```

> [!warning] Move != copie superficielle
> En C, `s2 = s1` pour des pointeurs fait une copie **du pointeur** (pas des donnees). Les deux pointeurs sont valides et c'est un piege. En Rust, `let s2 = s1` fait un move : les metadonnees (ptr, len, cap) sont copiees sur la stack, mais l'ancien proprietaire est **invalide**. C'est comme un transfert de propriete legal.

### Visualisation detaillee du move

```
Etape 1 : let s1 = String::from("hello");

     Stack (frame main)              Heap
     +------------------+
s1:  | ptr  : 0xABC0    |--------->  +---+---+---+---+---+
     | len  : 5          |           | h | e | l | l | o |
     | cap  : 5          |           +---+---+---+---+---+
     +------------------+            Adresse 0xABC0


Etape 2 : let s2 = s1;   (MOVE)

     Stack (frame main)              Heap
     +------------------+
s1:  | xxxxxxxxxxxxxxxx  |  (INVALIDE - le compilateur marque s1 comme "moved")
     | xxxxxxxxxxxxxxxx  |
     | xxxxxxxxxxxxxxxx  |
     +------------------+
     +------------------+
s2:  | ptr  : 0xABC0    |--------->  +---+---+---+---+---+
     | len  : 5          |           | h | e | l | l | o |
     | cap  : 5          |           +---+---+---+---+---+
     +------------------+            Adresse 0xABC0

     Un seul proprietaire -> drop() appele UNE seule fois quand s2 sort du scope
```

---

## Clone : copie profonde explicite

Si vous avez vraiment besoin de deux copies independantes, utilisez `.clone()` :

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();  // Copie profonde : nouvelle allocation heap

    println!("{}", s1);  // OK ! s1 est toujours valide
    println!("{}", s2);  // OK ! s2 a sa propre copie
}
```

```
Apres clone() :

Stack                  Heap
s1: [ptr] ----------> [h|e|l|l|o]    (allocation 1)
    [len:5]
    [cap:5]

s2: [ptr] ----------> [h|e|l|l|o]    (allocation 2, independante)
    [len:5]
    [cap:5]

Deux allocations separees. Deux free() independants. Aucun probleme.
```

> [!warning] Cout du clone
> `.clone()` fait une **allocation heap** et copie les donnees. C'est l'equivalent de `malloc` + `memcpy` en C. Ne clonez pas par reflexe pour faire taire le compilateur. C'est souvent le signe qu'il faut repenser le design avec des references.

---

## Le trait Copy : types qui vivent sur la stack

Les types simples qui vivent entierement sur la stack n'ont pas besoin de move. Ils implementent le trait `Copy` et sont automatiquement copies.

```rust
fn main() {
    // Les entiers implementent Copy
    let x = 5;
    let y = x;  // COPIE (pas un move) : x est toujours valide
    println!("x = {}, y = {}", x, y);  // OK !

    // Pourquoi ? Parce que copier un i32 (4 octets sur la stack)
    // est aussi rapide que faire un move. Pas besoin de semantique
    // de deplacement pour des donnees si petites.
}
```

### Types qui implementent Copy

```
+-------------------------+----------------------------+
| Implementent Copy       | N'implementent PAS Copy    |
+-------------------------+----------------------------+
| i8, i16, i32, i64, i128| String                     |
| u8, u16, u32, u64, u128| Vec<T>                     |
| f32, f64                | Box<T>                     |
| bool                    | Tout type avec des donnees |
| char                    |   sur le heap              |
| (i32, bool)  (tuples    | HashMap<K,V>               |
|  de types Copy)         |                            |
| [i32; 5]  (arrays de   |                            |
|  types Copy)            |                            |
+-------------------------+----------------------------+
```

> [!info] Regle generale
> Si un type ne necessite **aucune allocation heap** et que sa copie bit-a-bit est valide, il peut implementer `Copy`. Les types qui gerent des ressources (memoire heap, fichiers, connexions) ne le peuvent pas car copier leurs metadonnees sans dupliquer la ressource serait dangereux.

---

## Ownership et fonctions

Passer une valeur a une fonction suit les memes regles : ca **deplace** (move) la valeur.

### Le move dans les fonctions

```rust
fn prend_possession(s: String) {
    println!("{}", s);
}  // s est dropped ici, la memoire est liberee

fn donne_possession() -> String {
    let s = String::from("hello");
    s  // La possession est transferee a l'appelant
}

fn main() {
    let s1 = String::from("hello");
    prend_possession(s1);
    // println!("{}", s1);  // ERREUR : s1 a ete MOVE dans la fonction

    let s2 = donne_possession();
    println!("{}", s2);  // OK : on a recu la possession

    // Pattern "prendre et rendre" (lourd, on verra mieux apres)
    fn traiter_et_rendre(s: String) -> String {
        println!("Traitement de : {}", s);
        s  // On rend la possession
    }

    let s3 = String::from("bonjour");
    let s3 = traiter_et_rendre(s3);  // Shadowing avec la valeur rendue
    println!("{}", s3);  // OK
}
```

### Comparaison avec C

```c
// En C : aucune notion de propriete, c'est le far west
void traiter(char *s) {
    printf("%s\n", s);
    free(s);  // Est-ce que cette fonction est censee liberer ? Qui sait ?
}

int main(void) {
    char *s = malloc(6);
    strcpy(s, "hello");
    traiter(s);
    // free(s);  // Double free si traiter() a deja libere !
    //           // Mais comment savoir sans lire le code de traiter() ?
    return 0;
}
```

> [!tip] Analogie
> En C, donner un pointeur a une fonction c'est comme donner les cles de votre maison a quelqu'un sans contrat. Qui est responsable ? En Rust, le systeme de types EST le contrat : si la fonction prend un `String` (pas une reference), elle prend possession. C'est clair, explicite, verifie a la compilation.

---

## References et borrowing (emprunt)

Le pattern "prendre et rendre" est lourd. La solution : les **references**. Emprunter une valeur sans en prendre possession.

### References immutables : `&T`

```rust
fn calculer_longueur(s: &String) -> usize {
    s.len()
}  // s sort du scope, mais comme c'est une reference,
   // la valeur pointee N'EST PAS liberee

fn main() {
    let s1 = String::from("hello");
    let len = calculer_longueur(&s1);  // On prete s1, on ne la donne pas
    println!("'{}' fait {} caracteres", s1, len);  // s1 est toujours valide !
}
```

```
Emprunt immutable (&String) :

Stack                        Heap
s1: [ptr] ----------------> [h|e|l|l|o]
    [len:5]                  ^
    [cap:5]                  |
                             |
&s1 (dans calculer_longueur):
    [ptr vers s1] ----------+
                             (pointe vers les memes donnees)
                             (mais NE POSSEDE PAS)
                             (pas de free quand &s1 sort du scope)
```

### References mutables : `&mut T`

```rust
fn ajouter_monde(s: &mut String) {
    s.push_str(", world!");
}

fn main() {
    let mut s = String::from("hello");
    ajouter_monde(&mut s);
    println!("{}", s);  // "hello, world!"
}
```

### Les 2 regles du borrowing

```
+------------------------------------------------------------------------+
|                    LES 2 REGLES DU BORROWING                           |
|                                                                        |
|  A tout instant, vous pouvez avoir SOIT :                              |
|                                                                        |
|    - Autant de references immutables (&T) que vous voulez              |
|                                                                        |
|  SOIT :                                                                |
|                                                                        |
|    - Exactement UNE reference mutable (&mut T)                         |
|                                                                        |
|  JAMAIS les deux en meme temps.                                        |
+------------------------------------------------------------------------+
```

```rust
fn main() {
    let mut s = String::from("hello");

    // OK : plusieurs references immutables
    let r1 = &s;
    let r2 = &s;
    println!("{} et {}", r1, r2);
    // r1 et r2 ne sont plus utilisees apres cette ligne (NLL)

    // OK : une reference mutable (apres que r1 et r2 ne sont plus utilisees)
    let r3 = &mut s;
    r3.push_str("!");
    println!("{}", r3);

    // ERREUR : reference immutable ET mutable en meme temps
    // let r4 = &s;
    // let r5 = &mut s;
    // println!("{} {}", r4, r5);  // Ne compile pas !
}
```

> [!tip] Analogie
> Pensez a un document partage. Plusieurs personnes peuvent le **lire** en meme temps (references immutables). Mais si quelqu'un veut l'**editer** (reference mutable), il doit avoir un acces exclusif --- sinon les lecteurs pourraient voir des donnees incoherentes. C'est exactement le principe des locks lecture/ecriture (rwlock), mais verifie a la compilation.

### Pourquoi cette regle ?

```rust
// Sans cette regle, ce bug serait possible :
fn main() {
    let mut v = vec![1, 2, 3];
    let premier = &v[0];  // Reference immutable vers le premier element

    v.push(4);  // ERREUR ! push() peut reallouer le Vec en memoire
                 // Si reallocation : `premier` pointerait vers de la memoire liberee
                 // C'est EXACTEMENT un dangling pointer !

    // println!("{}", premier);  // Aurait ete un use-after-free
}
```

```
Avant push(4) :                      Apres push(4) avec reallocation :

Stack          Heap                   Stack          Heap
v: [ptr] ----> [1|2|3| ]             v: [ptr] ----> [1|2|3|4| | | | ]
   [len:3]     capacite 4               [len:4]     NOUVELLE adresse !
   [cap:4]                              [cap:8]

premier: [ptr] --> (ancien emplacement)   premier: [ptr] --> MEMOIRE LIBEREE !
                    ^                                         ^^^^^^^^^^^^^^^
                    OK avant push                      DANGLING POINTER !
```

> [!warning] En C, ce bug est silencieux
> ```c
> // Ce code C compile et semble fonctionner... jusqu'au jour ou le realloc deplace le buffer
> #include <stdlib.h>
> int main(void) {
>     int *v = malloc(3 * sizeof(int));
>     v[0] = 1; v[1] = 2; v[2] = 3;
>     int *premier = &v[0];
>     v = realloc(v, 8 * sizeof(int));  // Peut deplacer le buffer !
>     printf("%d\n", *premier);          // Dangling pointer si realloc a deplace
>     return 0;
> }
> ```
> Rust detecte ce bug a la compilation. En C, il peut passer inapercu pendant des annees.

---

## References pendantes (dangling references)

Rust interdit les dangling references a la compilation :

```rust
// ERREUR DE COMPILATION
fn dangling() -> &String {
    let s = String::from("hello");
    &s  // On retourne une reference vers s...
}       // ...mais s est detruit ici ! La reference pointerait vers rien.

// Le compilateur dit :
// "this function's return type contains a borrowed value,
//  but there is no value for it to be borrowed from"

// Solution : retourner la valeur elle-meme (transfert de propriete)
fn pas_dangling() -> String {
    let s = String::from("hello");
    s  // On DONNE la propriete a l'appelant
}
```

Comparaison avec C :

```c
// C compile sans broncher
char *dangling(void) {
    char buffer[100];
    strcpy(buffer, "hello");
    return buffer;  // Retourne un pointeur vers la stack de dangling()
}                   // buffer est detruit, le pointeur est invalide
// GCC donne un warning, mais compile quand meme
```

---

## Lifetimes (durees de vie)

Les lifetimes sont la facon dont Rust raisonne sur la duree de validite des references. La plupart du temps, le compilateur les infere automatiquement. Mais parfois, vous devez les annoter.

### Pourquoi les lifetimes existent

```rust
// Le compilateur ne peut pas determiner quelle reference est retournee
fn le_plus_long(x: &str, y: &str) -> &str {  // ERREUR !
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
// Le compilateur demande : "la reference retournee vit combien de temps ?"
// Est-ce qu'elle vit aussi longtemps que x ? Que y ? Il ne peut pas savoir.
```

### Annotations de lifetime

```rust
// On annote : la reference retournee vit au moins aussi longtemps
// que la plus courte des durees de vie de x et y
fn le_plus_long<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("longue chaine");
    let resultat;
    {
        let string2 = String::from("xyz");
        resultat = le_plus_long(string1.as_str(), string2.as_str());
        println!("Le plus long : {}", resultat);  // OK ici
    }
    // println!("{}", resultat);  // ERREUR si on deplace cette ligne ici
    // string2 est mort, et resultat pourrait pointer vers string2
}
```

> [!info] Syntaxe des lifetimes
> `'a` (se prononce "tick a" ou "lifetime a") est un **parametre generique de duree de vie**. C'est comme un parametre de type generique `<T>`, mais pour les durees de vie. Il ne change pas le comportement du code, il aide le compilateur a verifier que les references sont valides.

### Lifetimes dans les structs

Si une struct contient une reference, il faut annoter le lifetime :

```rust
// Cette struct contient une reference, donc elle ne peut pas
// vivre plus longtemps que la donnee referencee
struct Extrait<'a> {
    contenu: &'a str,
}

fn main() {
    let roman = String::from("Il etait une fois...");
    let extrait = Extrait {
        contenu: &roman[0..17],
    };
    println!("{}", extrait.contenu);
}
// Si roman etait detruit avant extrait -> dangling reference
// Le lifetime 'a garantit que ca ne peut pas arriver
```

### Regles d'elision des lifetimes

Le compilateur infere les lifetimes automatiquement dans la plupart des cas grace a trois regles :

```
Regle 1 : Chaque parametre reference recoit son propre lifetime
          fn f(x: &str, y: &str) -> fn f<'a, 'b>(x: &'a str, y: &'b str)

Regle 2 : S'il n'y a qu'un parametre reference, son lifetime
          est assigne a toutes les references en sortie
          fn f(x: &str) -> &str  ->  fn f<'a>(x: &'a str) -> &'a str

Regle 3 : S'il y a un parametre &self ou &mut self, son lifetime
          est assigne a toutes les references en sortie
          fn f(&self, x: &str) -> &str  utilise le lifetime de self
```

```rust
// Ces trois signatures sont equivalentes grace a l'elision :
fn premier_mot(s: &str) -> &str { /* ... */ }
fn premier_mot<'a>(s: &'a str) -> &'a str { /* ... */ }
// Le compilateur applique la regle 2 automatiquement
```

> [!example] Quand faut-il annoter ?
> ```rust
> // PAS BESOIN : un seul parametre reference (regle 2)
> fn longueur(s: &str) -> usize { s.len() }
>
> // PAS BESOIN : methode avec &self (regle 3)
> impl Livre {
>     fn titre(&self) -> &str { &self.titre }
> }
>
> // BESOIN : deux parametres reference et on retourne une reference
> // Le compilateur ne peut pas deviner lequel
> fn le_plus_long<'a>(x: &'a str, y: &'a str) -> &'a str { /* ... */ }
> ```

### Le lifetime 'static

`'static` signifie que la reference est valide pour toute la duree du programme.

```rust
// Les litteraux de chaine ont le lifetime 'static
let s: &'static str = "je vis eternellement";
// La chaine est incluse dans le binaire, donc toujours valide

// Les constantes aussi
static GLOBAL: i32 = 42;
let r: &'static i32 = &GLOBAL;
```

> [!warning] N'abusez pas de 'static
> Quand le compilateur demande un lifetime et que vous ne savez pas quoi mettre, la tentation est de mettre `'static` partout. C'est presque toujours le mauvais reflexe. `'static` signifie "vit pour toujours", ce qui est rarement ce que vous voulez. Cherchez plutot la bonne duree de vie.

---

## Le borrow checker en pratique

Le borrow checker est la partie du compilateur qui verifie les regles d'ownership et de borrowing. Voici les erreurs les plus courantes et comment les resoudre.

### Erreur 1 : use of moved value

```rust
fn main() {
    let s = String::from("hello");
    let s2 = s;
    println!("{}", s);  // ERREUR : value borrowed here after move
}
```

Solutions :

```rust
// Solution 1 : utiliser une reference
fn main() {
    let s = String::from("hello");
    let s2 = &s;  // Emprunt, pas move
    println!("{} {}", s, s2);  // OK
}

// Solution 2 : cloner si necessaire
fn main() {
    let s = String::from("hello");
    let s2 = s.clone();
    println!("{} {}", s, s2);  // OK, mais allocation supplementaire
}

// Solution 3 : repenser le design (souvent la meilleure option)
```

### Erreur 2 : cannot borrow as mutable

```rust
fn main() {
    let s = String::from("hello");
    s.push_str(" world");  // ERREUR : cannot borrow `s` as mutable
}
// Solution : declarer mut
fn main() {
    let mut s = String::from("hello");
    s.push_str(" world");  // OK
}
```

### Erreur 3 : cannot borrow as mutable because also borrowed as immutable

```rust
fn main() {
    let mut v = vec![1, 2, 3];
    let first = &v[0];     // emprunt immutable
    v.push(4);             // ERREUR : emprunt mutable alors que first existe
    println!("{}", first);
}
// Solution : utiliser first AVANT de modifier v
fn main() {
    let mut v = vec![1, 2, 3];
    let first = &v[0];
    println!("{}", first);  // Utiliser first ici
    // first n'est plus utilise (NLL - Non-Lexical Lifetimes)
    v.push(4);             // OK maintenant
}
```

### Erreur 4 : does not live long enough

```rust
fn main() {
    let r;
    {
        let s = String::from("hello");
        r = &s;  // ERREUR : s ne vit pas assez longtemps
    }  // s est detruit ici
    // println!("{}", r);  // r serait un dangling reference
}
// Solution : s'assurer que la donnee vit assez longtemps
fn main() {
    let s = String::from("hello");
    let r = &s;
    println!("{}", r);  // OK : s et r ont le meme scope
}
```

> [!tip] Strategie face aux erreurs du borrow checker
> 1. **Lisez l'erreur** : les messages de Rust sont excellents, avec des suggestions.
> 2. **Ne clonez pas par reflexe** : c'est un patch, pas une solution.
> 3. **Reorganisez le code** : souvent, reordonner les operations suffit.
> 4. **Utilisez des references** : `&T` au lieu de `T` dans les parametres.
> 5. **Reduisez la portee** : utilisez des blocs `{}` pour limiter la duree de vie des emprunts.

---

## Stack vs Heap en Rust

### Rappel : stack et heap en C

```c
// Stack (pile) : rapide, automatique, taille fixe
int x = 42;              // Sur la stack
int tableau[100];         // Sur la stack (taille connue a la compilation)

// Heap (tas) : lent, manuel, taille dynamique
int *p = malloc(100 * sizeof(int));  // Sur le heap
// ... utilisation ...
free(p);                              // Liberation MANUELLE
```

### En Rust : meme modele, gestion differente

```rust
fn main() {
    // Stack : types a taille fixe connue a la compilation
    let x: i32 = 42;               // 4 octets sur la stack
    let tableau: [i32; 5] = [0; 5]; // 20 octets sur la stack

    // Heap : types a taille dynamique
    let s = String::from("hello");  // Metadonnees sur la stack,
                                    // donnees sur le heap
    let v = vec![1, 2, 3];         // Pareil : metadonnees stack, donnees heap

    // Box<T> : allocation explicite sur le heap
    let b = Box::new(42);          // Alloue un i32 sur le heap
    println!("{}", b);             // 42 (auto-deref)
}  // Tout est libere automatiquement ici. TOUT.
```

### Box\<T\> : l'equivalent de malloc

```rust
fn main() {
    // En C : int *p = malloc(sizeof(int)); *p = 42; ... free(p);
    // En Rust :
    let b = Box::new(42);
    println!("{}", *b);  // Dereferencement (comme *p en C)
    // Pas besoin de free : drop est appele automatiquement
}

// Box est utile pour :
// 1. Types de taille inconnue a la compilation (types recursifs)
enum Liste {
    Cons(i32, Box<Liste>),
    Vide,
}

// 2. Transferer la propriete de grandes donnees sans copier
fn transferer(data: Box<[u8; 1_000_000]>) {
    // On a deplace UN pointeur (8 octets), pas 1 Mo de donnees
}
```

---

## Modele memoire du String : en detail

```
Comparaison detaillee : C char* vs Rust String

=== En C ===

char *s = "hello";        Stack          Code (read-only)
                           s: [ptr] ---> [h|e|l|l|o|\0]
                           (juste un pointeur, 8 octets)
                           strlen() = O(n), doit chercher le \0
                           Pas d'info sur la capacite
                           s[0] = 'H' -> UNDEFINED BEHAVIOR

char *s = malloc(10);     Stack          Heap
strcpy(s, "hello");        s: [ptr] ---> [h|e|l|l|o|\0|?|?|?|?]
                           (juste un pointeur, 8 octets)
                           Capacite ? Vous devez la suivre vous-meme
                           Debordement ? Aucune protection


=== En Rust ===

let s = String::from("hello");

    Stack (24 octets)          Heap (5 octets de donnees)
    +-------------------+
    | ptr  : 0x7F2A     |---->  +---+---+---+---+---+
    | len  : 5           |      | h | e | l | l | o |
    | cap  : 5           |      +---+---+---+---+---+
    +-------------------+
                                Pas de \0 terminal !
                                len = longueur actuelle    (O(1) pour .len())
                                cap = capacite allouee     (pour push_str)
                                Toute operation verifie les bornes

let s2 = &s[0..3];  // Slice &str

    Stack (16 octets)          Pointe vers les donnees de s
    +-------------------+
    | ptr  : 0x7F2A     |---->  [h|e|l]  (sous-partie du buffer de s)
    | len  : 3           |
    +-------------------+
                                C'est un "fat pointer" : pointeur + longueur
                                Pas de copie des donnees !
```

> [!info] Fat pointers
> En C, un pointeur est toujours un simple entier (adresse memoire, 8 octets sur 64 bits). En Rust, les references vers des types a taille dynamique sont des **fat pointers** : pointeur + metadonnees. Un `&str` fait 16 octets (pointeur + longueur). Un `&dyn Trait` fait 16 octets (pointeur + vtable).

---

## Les slices

Un slice est une **reference vers une portion contigue de memoire**. C'est comme un pointeur C avec une longueur attachee.

### Slices de chaines : &str

```rust
fn main() {
    let s = String::from("hello world");

    let hello: &str = &s[0..5];     // "hello"
    let world: &str = &s[6..11];    // "world"
    let tout: &str = &s[..];        // "hello world" (toute la chaine)
    let debut: &str = &s[..5];      // "hello" (depuis le debut)
    let fin: &str = &s[6..];        // "world" (jusqu'a la fin)

    println!("{} {}", hello, world);
}
```

### Slices de tableaux : &[T]

```rust
fn somme(nombres: &[i32]) -> i32 {
    let mut total = 0;
    for n in nombres {
        total += n;
    }
    total
}

fn main() {
    let tableau = [1, 2, 3, 4, 5];
    let slice = &tableau[1..4];  // [2, 3, 4]
    println!("Somme du slice : {}", somme(slice));
    println!("Somme du tableau : {}", somme(&tableau));
}
```

### Comparaison avec C

```c
// En C : un "slice" c'est un pointeur + une convention
void somme(int *data, int len) {  // Deux parametres separes
    int total = 0;
    for (int i = 0; i < len; i++) {
        total += data[i];  // Aucune verification de bornes
    }
}
// Rien n'empeche d'appeler somme(data, 1000) sur un tableau de 5 elements
```

```rust
// En Rust : un slice CONTIENT la longueur
fn somme(data: &[i32]) -> i32 {  // Un seul parametre : pointeur + longueur
    data.iter().sum()
    // Impossible d'acceder hors bornes : le slice connait sa taille
}
```

> [!example] Slices en pratique
> ```rust
> fn premier_mot(s: &str) -> &str {
>     let octets = s.as_bytes();
>     for (i, &octet) in octets.iter().enumerate() {
>         if octet == b' ' {
>             return &s[0..i];
>         }
>     }
>     &s[..]
> }
>
> fn main() {
>     let phrase = String::from("hello world");
>     let mot = premier_mot(&phrase);
>     println!("Premier mot : {}", mot);
>     // phrase.clear();  // ERREUR ! mot emprunte phrase de maniere immutable
>     //                  // on ne peut pas la modifier tant que mot est utilise
>     println!("{}", mot);
> }
> ```
> En C, rien n'empecherait de modifier `phrase` pendant que `mot` pointe dedans, causant un dangling pointer.

---

## Recapitulatif : comment l'ownership remplace malloc/free

```
+-----------------------------------------------------------------------+
|              MODELE MEMOIRE : C vs RUST                               |
+-----------------------------------------------------------------------+
|                                                                       |
|  En C :                                                               |
|    1. Vous allouez avec malloc()                                      |
|    2. Vous utilisez la memoire                                        |
|    3. Vous DEVEZ appeler free() exactement UNE fois                   |
|    4. Si vous oubliez -> memory leak                                  |
|    5. Si vous appelez deux fois -> double free                        |
|    6. Si vous utilisez apres free -> use-after-free                   |
|    7. Le compilateur ne vous aide PAS                                 |
|                                                                       |
|  En Rust :                                                            |
|    1. Vous creez une valeur (String::from, Vec::new, Box::new)        |
|    2. Vous utilisez la valeur                                         |
|    3. Quand le proprietaire sort du scope -> drop automatique         |
|    4. Oublier free ? Impossible (automatique)                         |
|    5. Double free ? Impossible (un seul proprietaire)                 |
|    6. Use-after-free ? Impossible (borrow checker)                    |
|    7. Le compilateur GARANTIT la correction                           |
|                                                                       |
+-----------------------------------------------------------------------+
```

```
Flux de vie d'une valeur heap en Rust :

   Creation                    Utilisation                 Destruction
      |                            |                           |
      v                            v                           v
  let s = String::from("hi")  println!("{}", s)        } // fin du scope
      |                            |                    |
      |  s est le proprietaire     |  s est emprunte    | drop(s) appele
      |  (un seul a la fois)       |  ou lu              | automatiquement
      |                            |                    |
      +------- AUCUN free() NECESSAIRE ----------------+
```

> [!tip] Analogie finale
> En C, la gestion memoire c'est comme conduire sans GPS dans une ville inconnue : vous pouvez arriver a destination, mais vous risquez de vous perdre, de prendre des sens interdits, ou de tourner en rond. En Rust, le borrow checker est votre GPS : il connait toutes les routes, tous les sens interdits, et refuse de vous guider vers un cul-de-sac. C'est parfois frustrant quand il dit "recalcul d'itineraire", mais vous arrivez toujours a destination sans accident.

---

## Carte Mentale ASCII

```
                          Ownership et Borrowing
                                   |
              +--------------------+--------------------+
              |                    |                    |
        OWNERSHIP             BORROWING            LIFETIMES
              |                    |                    |
   +----------+-------+    +------+------+      +------+------+
   |          |       |    |             |      |      |      |
 3 regles   Move    Clone  &T           &mut T  'a     elision 'static
   |          |       |    (shared)     (excl.)
   |      s2 = s1     |    Plusieurs    Un seul   Regles :
   |      s1 mort  copie   en meme      a la      1. Chaque param -> lifetime
   |               prof.   temps        fois       2. Un param -> output lifetime
   +-- 1 owner                                     3. &self -> output lifetime
   +-- 1 a la fois    Copy trait :
   +-- drop au        i32, bool, char
       fin scope      (stack only,
                       pas de move)
              |
     +--------+--------+
     |                  |
   Stack              Heap
   let x = 5;        String, Vec, Box
   [i32; 5]          ptr + len + cap
   Copie rapide       Move par defaut
                      Drop automatique

    Slices :
    &str  = fat ptr vers chaine (ptr + len)
    &[T]  = fat ptr vers tableau (ptr + len)
```

---

## Exercices

### Exercice 1 : Predire le comportement du compilateur

Pour chaque bloc de code, indiquez si ca compile ou non. Si non, expliquez pourquoi et proposez une correction.

```rust
// Bloc A
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    println!("{} {}", s1, s2);
}

// Bloc B
fn main() {
    let x = 42;
    let y = x;
    println!("{} {}", x, y);
}

// Bloc C
fn main() {
    let mut s = String::from("hello");
    let r1 = &s;
    let r2 = &s;
    let r3 = &mut s;
    println!("{} {} {}", r1, r2, r3);
}

// Bloc D
fn main() {
    let mut v = vec![1, 2, 3];
    let first = &v[0];
    v.push(4);
    println!("{}", first);
}
```

### Exercice 2 : Reecrire du C en Rust safe

Traduisez ce code C en Rust idiomatique, en eliminant tous les problemes de memoire :

```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

char *concatener(const char *a, const char *b) {
    char *resultat = malloc(strlen(a) + strlen(b) + 1);
    strcpy(resultat, a);
    strcat(resultat, b);
    return resultat;  // L'appelant doit penser a free() !
}

int main(void) {
    char *s1 = concatener("hello", " ");
    char *s2 = concatener(s1, "world");
    printf("%s\n", s2);
    free(s1);
    free(s2);
    return 0;
}
```

### Exercice 3 : Lifetimes

Corrigez la signature de cette fonction pour qu'elle compile :

```rust
fn le_plus_court(a: &str, b: &str) -> &str {
    if a.len() <= b.len() {
        a
    } else {
        b
    }
}

fn main() {
    let s1 = String::from("hi");
    let resultat;
    {
        let s2 = String::from("hello world");
        resultat = le_plus_court(&s1, &s2);
        println!("{}", resultat);
    }
}
```

### Exercice 4 : Slice et ownership

Ecrivez une fonction `mots_longs` qui prend un `&str` et retourne un `Vec<&str>` contenant uniquement les mots de plus de 4 lettres. Faites attention aux lifetimes.

```rust
fn mots_longs(phrase: &str) -> Vec<&str> {
    // A completer
    // Indice : .split_whitespace(), .filter(), .collect()
    todo!()
}

fn main() {
    let texte = String::from("le petit chat mange des croquettes enormes");
    let longs = mots_longs(&texte);
    println!("{:?}", longs);  // ["petit", "mange", "croquettes", "enormes"]
}
```

---

## Liens

- [[04 - Pointeurs et Memoire]] --- Les pointeurs et la gestion memoire en C (les problemes que Rust resout)
- [[03b - Les Pointeurs]] --- Les pointeurs en C en detail
- [[01 - Introduction a Rust]] --- Les bases du langage Rust
- [[03 - Structs Enums et Pattern Matching]] --- Types personnalises et pattern matching
