# Introduction a JavaScript

JavaScript est le langage de programmation du Web. Contrairement au C qui compile en code machine et a Python qui s'interprete ligne par ligne, JavaScript a ete concu pour s'executer dans un navigateur web, transformant des pages statiques en applications interactives. Aujourd'hui, grace a Node.js, il s'execute aussi cote serveur, en ligne de commande, et meme sur des microcontroleurs.

Ce cours pose les fondations : histoire, syntaxe, types, variables, structures de controle, fonctions, portee, hoisting et closures. Si vous connaissez deja C et Python, vous verrez que JavaScript emprunte a chacun tout en ajoutant ses propres surprises.

> [!tip] Analogie
> Imaginez que C est un moteur de Formule 1 (puissant, precis, mais vous gerez tout vous-meme), Python est une voiture automatique (confortable, lisible), et JavaScript est un moteur hybride : il s'adapte au contexte (navigateur ou serveur), change de comportement selon le mode (strict ou non), et possede un systeme de types tres flexible -- parfois trop.

---

## 1. Qu'est-ce que JavaScript ?

JavaScript (souvent abrege JS) est un langage :
- **Interprete** (ou compile a la volee via un moteur JIT)
- **Multi-paradigme** : imperatif, fonctionnel, oriente objet (prototypes)
- **Dynamiquement type** : les types sont determines a l'execution
- **Single-threaded** avec un modele evenementiel (event loop)

> [!info] Ne pas confondre Java et JavaScript
> Malgre la similitude de nom (raison marketing de 1995), Java et JavaScript sont des langages completement differents. Java est statiquement type, compile en bytecode, et oriente objet par classes. JavaScript est dynamiquement type, interprete, et oriente objet par prototypes.

### Comparaison rapide : C vs Python vs JavaScript

```
+--------------------+----------------+----------------+------------------+
| Caracteristique    | C              | Python         | JavaScript       |
+--------------------+----------------+----------------+------------------+
| Typage             | Statique fort  | Dynamique fort | Dynamique faible |
| Compilation        | Compile        | Interprete     | JIT (V8, etc.)   |
| Paradigme          | Imperatif      | Multi          | Multi            |
| Gestion memoire    | Manuelle       | GC automatique | GC automatique   |
| Point-virgule      | Obligatoire    | Non            | Optionnel (ASI)  |
| Accolades          | Oui            | Non (indent.)  | Oui              |
| Execution          | Natif OS       | Interpreteur   | Navigateur/Node  |
| Entree/Sortie      | printf/scanf   | print/input    | console.log      |
+--------------------+----------------+----------------+------------------+
```

---

## 2. Bref historique

### La naissance (1995)

- **Brendan Eich** cree JavaScript en **10 jours** chez Netscape
- Nom original : Mocha, puis LiveScript, puis JavaScript
- Objectif : ajouter de l'interactivite aux pages web

### L'evolution

```
1995 ── JavaScript 1.0 (Netscape Navigator 2.0)
  │
1997 ── ECMAScript 1 (standardisation ECMA-262)
  │
1999 ── ECMAScript 3 (regex, try/catch, etc.)
  │
2009 ── ECMAScript 5 (strict mode, JSON, Array methods)
  │
2015 ── ECMAScript 6 / ES2015 (let/const, arrow, class, Promise...)
  │        └── Revolution majeure, le "JS moderne" commence ici
  │
2016 ── ES2016 (Array.includes, **)
  │
2017 ── ES2017 (async/await, Object.entries)
  │
2020 ── ES2020 (optional chaining, nullish coalescing)
  │
2024 ── ES2024 (Array grouping, Promise.withResolvers...)
  │
  v
  Aujourd'hui : mise a jour annuelle
```

> [!info] ECMAScript vs JavaScript
> ECMAScript est la **specification** (le standard). JavaScript est l'**implementation** la plus connue de ce standard. Quand on dit "ES6", on parle de la 6e edition de la specification ECMAScript, publiee en 2015.

---

## 3. Ou s'execute JavaScript ?

### Dans le navigateur

Chaque navigateur possede un **moteur JavaScript** :

```
+-------------------+-------------------+
| Navigateur        | Moteur JS         |
+-------------------+-------------------+
| Chrome / Edge     | V8                |
| Firefox           | SpiderMonkey      |
| Safari            | JavaScriptCore    |
+-------------------+-------------------+
```

Le navigateur fournit aussi des **API Web** : DOM, fetch, localStorage, etc.

### Avec Node.js (cote serveur)

Node.js utilise le moteur **V8** de Chrome en dehors du navigateur.

```
┌──────────────────────────────────────────┐
│              Navigateur                  │
│  ┌──────────┐  ┌──────────────────────┐  │
│  │ Moteur V8│  │ API Web (DOM, fetch) │  │
│  └──────────┘  └──────────────────────┘  │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│              Node.js                     │
│  ┌──────────┐  ┌──────────────────────┐  │
│  │ Moteur V8│  │ API Node (fs, http)  │  │
│  └──────────┘  └──────────────────────┘  │
└──────────────────────────────────────────┘
```

> [!tip] Analogie
> Le moteur V8 est comme un moteur de voiture. Dans le navigateur, il est monte dans une berline (avec GPS = DOM, vitres electriques = API Web). Dans Node.js, il est monte dans un camion (avec remorque = systeme de fichiers, radio CB = reseau serveur).

---

## 4. Ajouter JavaScript a une page HTML

### Methode 1 : script en ligne (inline)

```javascript
<!DOCTYPE html>
<html>
<body>
  <h1>Ma page</h1>
  <script>
    console.log("Hello depuis le script inline !");
  </script>
</body>
</html>
```

### Methode 2 : fichier externe (recommande)

```javascript
// fichier : script.js
console.log("Hello depuis un fichier externe !");
```

```html
<!DOCTYPE html>
<html>
<head>
  <script src="script.js"></script>
</head>
<body>
  <h1>Ma page</h1>
</body>
</html>
```

### Attributs defer et async

```
Sans attribut :     HTML ──────■ STOP ■ telecharge JS ■ execute JS ■──── HTML
                                  │          │              │
                                  └──────────┴──────────────┘
                                       Bloque le rendu

defer :             HTML ──────────────────────────────────────── HTML terminé
                         telecharge JS en parallele ───┐          │
                                                       └─ execute JS
                    (Execute apres le parsing HTML, ordre garanti)

async :             HTML ──────────────■ STOP ■ execute JS ■──── HTML
                         telecharge JS ──┘
                    (Execute des que telecharge, ordre NON garanti)
```

> [!warning] Placement du script
> Si votre script manipule le DOM et que vous le placez dans `<head>` sans `defer`, il s'executera **avant** que le HTML ne soit charge. Le DOM n'existera pas encore et votre code echouera. Utilisez `defer` ou placez le `<script>` juste avant `</body>`.

### Comparaison avec C et Python

```
C :       gcc main.c -o main && ./main     (compilation puis execution)
Python :  python script.py                 (interpretation directe)
JS :      <script src="app.js"></script>   (dans le navigateur)
          node app.js                      (avec Node.js)
```

---

## 5. La console du navigateur

La console est l'outil de debogage principal du developpeur JavaScript.

### Ouvrir la console

- **Chrome / Edge** : `F12` ou `Ctrl + Shift + J`
- **Firefox** : `F12` ou `Ctrl + Shift + K`
- **Safari** : `Cmd + Option + C`

### Methodes principales de console

```javascript
console.log("Message standard");          // Affichage simple
console.warn("Attention !");              // Avertissement (jaune)
console.error("Erreur critique !");       // Erreur (rouge)
console.table([1, 2, 3]);                // Affichage en tableau
console.time("timer");                   // Demarre un chronometre
// ... code a mesurer ...
console.timeEnd("timer");               // Affiche le temps ecoule
console.dir(document.body);             // Affiche les proprietes d'un objet
console.group("Groupe");                // Debut d'un groupe
console.log("Element 1");
console.log("Element 2");
console.groupEnd();                     // Fin du groupe
```

> [!example] Equivalent dans d'autres langages
> ```
> C :       printf("Hello %s\n", name);   // #include <stdio.h>
> Python :  print(f"Hello {name}")
> JS :      console.log(`Hello ${name}`); // template literal
> ```

---

## 6. Variables : var, let et const

### Declaration

```javascript
var ancienneMethode = "evitez var";    // portee fonction, hoisted
let variable = "peut changer";         // portee bloc, pas de re-declaration
const constante = "ne change pas";     // portee bloc, pas de re-assignation
```

### Tableau comparatif var vs let vs const

```
+-------------------+----------+----------+----------+
| Propriete         | var      | let      | const    |
+-------------------+----------+----------+----------+
| Portee            | Fonction | Bloc     | Bloc     |
| Hoisting          | Oui (*)  | Oui (**) | Oui (**) |
| Re-declaration    | Oui      | Non      | Non      |
| Re-assignation    | Oui      | Oui      | Non      |
| Valeur initiale   | undefined| TDZ      | TDZ      |
| Utilisation       | Legacy   | Variable | Constante|
+-------------------+----------+----------+----------+
(*) Hoisted avec valeur undefined
(**) Hoisted mais dans la Temporal Dead Zone (TDZ) = ReferenceError si acces avant declaration
```

### Exemples de portee

```javascript
// ─── var : portee FONCTION ───
function exempleVar() {
    if (true) {
        var x = 10;
    }
    console.log(x); // 10 -- var "sort" du bloc if
}

// ─── let : portee BLOC ───
function exempleLet() {
    if (true) {
        let y = 20;
    }
    // console.log(y); // ReferenceError: y is not defined
}

// ─── const : reference constante ───
const tableau = [1, 2, 3];
tableau.push(4);          // OK ! Le contenu peut changer
// tableau = [5, 6];      // TypeError: Assignment to constant variable
```

> [!warning] const ne signifie pas immuable
> `const` empeche la **re-assignation** de la variable, pas la **mutation** de la valeur. Un objet ou tableau `const` peut etre modifie. Pour rendre un objet immuable, utilisez `Object.freeze()`.

### Comparaison avec C et Python

```
C :       int x = 5;        // Type explicite, portee bloc
          const int Y = 10; // Vraie constante (valeur figee)

Python :  x = 5             // Pas de mot-cle, portee fonction/module
          Y = 10            // Convention MAJUSCULES, pas de vrai const

JS :      let x = 5;        // Portee bloc
          const Y = 10;     // Reference constante (pas la valeur)
```

---

## 7. Types de donnees

JavaScript possede **8 types** : 7 types primitifs et 1 type objet.

### Types primitifs

```javascript
// ─── number ───
let entier = 42;
let flottant = 3.14;
let infini = Infinity;
let pasUnNombre = NaN;          // "Not a Number" (mais typeof = "number" !)

// ─── string ───
let simple = 'Guillemets simples';
let double = "Guillemets doubles";
let backtick = `Template literal ${entier}`;

// ─── boolean ───
let vrai = true;
let faux = false;

// ─── null ───
let vide = null;                // Absence intentionnelle de valeur

// ─── undefined ───
let nonDefini;                  // Variable declaree mais pas initialisee
let aussi = undefined;          // Explicitement undefined

// ─── symbol (ES6) ───
let sym = Symbol("description"); // Identifiant unique et immuable

// ─── bigint (ES2020) ───
let grand = 9007199254740991n;  // Entier de precision arbitraire
let aussi_grand = BigInt("123456789012345678901234567890");
```

### Type objet (reference)

```javascript
// ─── object ───
let objet = { nom: "Alice", age: 25 };
let tableau = [1, 2, 3];             // Array (sous-type d'objet)
let fn = function() {};               // Function (sous-type d'objet)
let date = new Date();                // Date (sous-type d'objet)
let regex = /pattern/g;               // RegExp (sous-type d'objet)
```

### Hierarchie des types

```
Types JavaScript
├── Primitifs (valeur, immuables, compares par valeur)
│   ├── number      42, 3.14, NaN, Infinity
│   ├── string      "hello", 'world', `template`
│   ├── boolean     true, false
│   ├── null        null
│   ├── undefined   undefined
│   ├── symbol      Symbol("id")
│   └── bigint      123n
│
└── Objet (reference, muable, compare par reference)
    ├── Object      { cle: "valeur" }
    ├── Array       [1, 2, 3]
    ├── Function    function() {}
    ├── Date        new Date()
    ├── RegExp      /pattern/
    ├── Map, Set    new Map(), new Set()
    └── ...         (tout le reste)
```

### Comparaison des types avec C et Python

```
+------------+-------------------+-------------------+-------------------+
| Concept    | C                 | Python            | JavaScript        |
+------------+-------------------+-------------------+-------------------+
| Entier     | int, long         | int (illimite)    | number / bigint   |
| Flottant   | float, double     | float             | number            |
| Caractere  | char              | str (len 1)       | string (len 1)    |
| Chaine     | char[]            | str               | string            |
| Booleen    | int (0/1)         | bool              | boolean           |
| Null       | NULL (pointeur)   | None              | null / undefined  |
| Tableau    | int arr[]         | list              | Array             |
| Dict/Map   | (pas natif)       | dict              | Object / Map      |
+------------+-------------------+-------------------+-------------------+
```

---

## 8. typeof et verification de type

```javascript
typeof 42;              // "number"
typeof "hello";         // "string"
typeof true;            // "boolean"
typeof undefined;       // "undefined"
typeof null;            // "object"     // BUG historique !
typeof Symbol();        // "symbol"
typeof 42n;             // "bigint"
typeof {};              // "object"
typeof [];              // "object"     // Array est un objet
typeof function(){};    // "function"   // Cas special
typeof NaN;             // "number"     // NaN est un... number
```

> [!warning] Le bug de typeof null
> `typeof null` retourne `"object"` au lieu de `"null"`. C'est un bug present depuis la toute premiere version de JavaScript (1995) qui n'a jamais ete corrige pour des raisons de compatibilite. Pour verifier null, utilisez `=== null`.

### Verifications fiables

```javascript
// Verifier null
let val = null;
val === null;                    // true

// Verifier un Array
Array.isArray([1, 2, 3]);       // true
Array.isArray({});               // false

// Verifier NaN
Number.isNaN(NaN);              // true
Number.isNaN("hello");          // false (contrairement a isNaN())

// Verifier un entier
Number.isInteger(42);           // true
Number.isInteger(42.0);         // true
Number.isInteger(42.5);         // false
```

---

## 9. Coercion de type (type coercion)

La coercion est la conversion automatique (implicite) ou manuelle (explicite) entre types.

### Coercion implicite (a eviter autant que possible)

```javascript
// ─── String + number => concatenation ───
"5" + 3;           // "53"  (number converti en string)
"5" + true;        // "5true"

// ─── Autres operateurs => conversion en number ───
"5" - 3;           // 2
"5" * 3;           // 15
"5" / "2";         // 2.5
true + true;       // 2     (true = 1)
false + 1;         // 1     (false = 0)

// ─── Comparaisons ───
"5" == 5;          // true  (coercion !)
null == undefined; // true  (cas special)
"" == false;       // true  (les deux deviennent 0)
```

> [!tip] Analogie
> La coercion implicite en JS est comme un traducteur automatique qui essaie de deviner ce que vous voulez dire. Parfois il a raison (`"5" - 3 = 2`), parfois il produit du charabia (`"5" + 3 = "53"`). En C, le compilateur vous previent. En Python, il refuse simplement (`TypeError`). En JS, il essaie toujours... et se trompe souvent.

### Coercion explicite (recommandee)

```javascript
// ─── Vers Number ───
Number("42");       // 42
Number("hello");    // NaN
Number(true);       // 1
Number(false);      // 0
Number(null);       // 0
Number(undefined);  // NaN
parseInt("42px");   // 42  (lit jusqu'au premier non-chiffre)
parseFloat("3.14"); // 3.14

// ─── Vers String ───
String(42);         // "42"
String(true);       // "true"
String(null);       // "null"
(42).toString();    // "42"

// ─── Vers Boolean ───
Boolean(0);         // false
Boolean("");        // false
Boolean(null);      // false
Boolean(undefined); // false
Boolean(NaN);       // false
Boolean("hello");   // true
Boolean(42);        // true
Boolean([]);        // true   (tableau vide = truthy !)
Boolean({});        // true   (objet vide = truthy !)
```

### Valeurs falsy et truthy

```
Valeurs FALSY (converties en false) :
────────────────────────────────────
false, 0, -0, 0n, "", null, undefined, NaN

Valeurs TRUTHY (tout le reste, y compris) :
────────────────────────────────────────────
true, 42, "0", "false", [], {}, function(){}
```

> [!warning] Pieges classiques
> En JavaScript, `[]` (tableau vide) et `{}` (objet vide) sont **truthy**, contrairement a Python ou une liste vide `[]` et un dict vide `{}` sont **falsy**. C'est une source frequente de bugs.

---

## 10. Egalite : == vs === (table de verite)

### == (egalite abstraite, avec coercion)

```javascript
5 == "5";          // true   (string converti en number)
0 == false;        // true   (false converti en 0)
"" == false;       // true   (les deux deviennent 0)
null == undefined; // true   (cas special de la spec)
0 == null;         // false  (null ne se convertit qu'avec undefined)
NaN == NaN;        // false  (NaN n'est egal a rien, meme pas a lui-meme)
```

### === (egalite stricte, sans coercion)

```javascript
5 === "5";          // false  (types differents)
0 === false;        // false
"" === false;       // false
null === undefined; // false
```

### Table de verite partielle avec ==

```
+-------------+-------+------+-------+-----------+------+------+--------+
|      ==     | true  |  1   |  "1"  | [1]       | "0"  |  0   | false  |
+-------------+-------+------+-------+-----------+------+------+--------+
| true        |   T   |  T   |   T   |   T       |  F   |  F   |   F    |
| 1           |   T   |  T   |   T   |   T       |  F   |  F   |   F    |
| "1"         |   T   |  T   |   T   |   T       |  F   |  F   |   F    |
| [1]         |   T   |  T   |   T   |   F(*)    |  F   |  F   |   F    |
| "0"         |   F   |  F   |   F   |   F       |  T   |  T   |   T    |
| 0           |   F   |  F   |   F   |   F       |  T   |  T   |   T    |
| false       |   F   |  F   |   F   |   F       |  T   |  T   |   T    |
+-------------+-------+------+-------+-----------+------+------+--------+
(*) Deux objets differents ne sont jamais == entre eux (comparaison par reference)
```

> [!warning] Regle d'or
> **Utilisez TOUJOURS `===` et `!==`**. L'egalite abstraite `==` est une source inepuisable de bugs. La seule exception courante : `val == null` pour tester a la fois `null` et `undefined`.

---

## 11. Template literals

```javascript
let nom = "Alice";
let age = 25;

// ─── Concatenation classique (ancienne methode) ───
let msg1 = "Bonjour " + nom + ", tu as " + age + " ans.";

// ─── Template literal (methode moderne) ───
let msg2 = `Bonjour ${nom}, tu as ${age} ans.`;

// ─── Expressions dans ${ } ───
let msg3 = `Dans 5 ans, tu auras ${age + 5} ans.`;

// ─── Multi-ligne ───
let html = `
  <div class="card">
    <h2>${nom}</h2>
    <p>Age : ${age}</p>
  </div>
`;

// ─── Appel de fonction ───
let msg4 = `Nom en majuscules : ${nom.toUpperCase()}`;
```

### Comparaison

```
C :       sprintf(buf, "Bonjour %s, tu as %d ans.", nom, age);
Python :  f"Bonjour {nom}, tu as {age} ans."      (f-string)
JS :      `Bonjour ${nom}, tu as ${age} ans.`      (backticks !)
```

---

## 12. Operateurs

### Operateurs arithmetiques

```javascript
let a = 10, b = 3;
a + b;    // 13    Addition
a - b;    // 7     Soustraction
a * b;    // 30    Multiplication
a / b;    // 3.333 Division (toujours flottante !)
a % b;    // 1     Modulo
a ** b;   // 1000  Exponentiation (ES2016)
```

> [!info] Pas de division entiere native
> En C, `10 / 3` donne `3` (division entiere entre int). En Python, `10 / 3` donne `3.333` et `10 // 3` donne `3`. En JS, `10 / 3` donne toujours `3.333...`. Pour une division entiere : `Math.floor(10 / 3)` ou `Math.trunc(10 / 3)`.

### Operateurs logiques

```javascript
// ET logique (short-circuit)
true && "hello";    // "hello" (retourne la derniere valeur truthy)
false && "hello";   // false   (retourne la premiere valeur falsy)

// OU logique (short-circuit)
"" || "default";    // "default" (retourne la premiere valeur truthy)
"hello" || "world"; // "hello"

// NON logique
!true;              // false
!0;                 // true
!!"hello";          // true  (double negation = conversion en boolean)
```

### Operateurs d'assignation composes

```javascript
let x = 10;
x += 5;   // x = 15
x -= 3;   // x = 12
x *= 2;   // x = 24
x /= 4;   // x = 6
x %= 4;   // x = 2
x **= 3;  // x = 8
x++;       // x = 9  (post-increment)
++x;       // x = 10 (pre-increment)
```

---

## 13. Structures de controle

### if / else if / else

```javascript
let score = 75;

if (score >= 90) {
    console.log("Excellent");
} else if (score >= 70) {
    console.log("Bien");
} else if (score >= 50) {
    console.log("Passable");
} else {
    console.log("Insuffisant");
}
```

### Operateur ternaire

```javascript
let age = 20;
let statut = age >= 18 ? "majeur" : "mineur";

// Equivalent en C : identique
// Equivalent en Python : "majeur" if age >= 18 else "mineur"
```

### switch

```javascript
let jour = "lundi";

switch (jour) {
    case "lundi":
    case "mardi":
    case "mercredi":
    case "jeudi":
    case "vendredi":
        console.log("Jour ouvrable");
        break;              // IMPORTANT : sans break, cascade !
    case "samedi":
    case "dimanche":
        console.log("Week-end");
        break;
    default:
        console.log("Jour inconnu");
}
```

> [!warning] N'oubliez pas le break !
> En C et en JS, sans `break`, l'execution "tombe" dans le case suivant (fall-through). Python n'a pas ce probleme car `match/case` (3.10+) ne fait pas de fall-through.

### Boucle for

```javascript
// ─── for classique (comme en C) ───
for (let i = 0; i < 5; i++) {
    console.log(i);  // 0, 1, 2, 3, 4
}

// ─── for...of (iterer sur les VALEURS, comme Python's for...in) ───
let fruits = ["pomme", "banane", "cerise"];
for (let fruit of fruits) {
    console.log(fruit);  // "pomme", "banane", "cerise"
}

// ─── for...in (iterer sur les CLES/INDEX) ───
let personne = { nom: "Alice", age: 25 };
for (let cle in personne) {
    console.log(`${cle}: ${personne[cle]}`);
}
// "nom: Alice", "age: 25"
```

> [!warning] for...in vs for...of
> `for...in` itere sur les **cles** (proprietes enumerables), `for...of` itere sur les **valeurs** (objets iterables). N'utilisez **jamais** `for...in` sur un tableau (il itere sur les index sous forme de strings et inclut les proprietes du prototype).

### Boucles while et do...while

```javascript
// ─── while ───
let compteur = 0;
while (compteur < 5) {
    console.log(compteur);
    compteur++;
}

// ─── do...while (execute au moins une fois) ───
let reponse;
do {
    reponse = prompt("Tapez 'oui' pour continuer");
} while (reponse !== "oui");
```

### Comparaison des boucles

```
+------------------+-----------------------------+---------------------------+
| Type             | C                           | Python       | JavaScript |
+------------------+-----------------------------+---------------------------+
| Compteur         | for(int i=0;i<n;i++)        | for i in range(n)         |
|                  |                             |              | for(let i=0;i<n;i++)     |
| Iterer valeurs   | (pas natif)                 | for x in lst | for(x of lst)            |
| Iterer cles      | (pas natif)                 | for k in d   | for(k in obj)            |
| Tant que         | while(cond)                 | while cond:  | while(cond)              |
| Au moins 1 fois  | do{...}while(cond);         | (pas natif)  | do{...}while(cond);      |
+------------------+-----------------------------+---------------------------+
```

---

## 14. Fonctions

### Declaration de fonction (function declaration)

```javascript
function saluer(nom) {
    return `Bonjour ${nom} !`;
}
console.log(saluer("Alice"));  // "Bonjour Alice !"
```

### Expression de fonction (function expression)

```javascript
const saluer = function(nom) {
    return `Bonjour ${nom} !`;
};
console.log(saluer("Bob"));
```

### Fonction flechee (arrow function, ES6)

```javascript
// ─── Syntaxe complete ───
const addition = (a, b) => {
    return a + b;
};

// ─── Retour implicite (une seule expression) ───
const addition2 = (a, b) => a + b;

// ─── Un seul parametre : parentheses optionnelles ───
const double = x => x * 2;

// ─── Pas de parametre ───
const salut = () => "Bonjour !";
```

### Parametres par defaut

```javascript
function creerUtilisateur(nom, role = "visiteur", actif = true) {
    return { nom, role, actif };
}

creerUtilisateur("Alice");               // { nom: "Alice", role: "visiteur", actif: true }
creerUtilisateur("Bob", "admin");        // { nom: "Bob", role: "admin", actif: true }
creerUtilisateur("Charlie", "mod", false);
```

### IIFE (Immediately Invoked Function Expression)

```javascript
// ─── Fonction executee immediatement ───
(function() {
    let secret = "invisible dehors";
    console.log("IIFE executee !");
})();

// ─── Version arrow ───
(() => {
    console.log("IIFE arrow !");
})();
```

> [!info] A quoi sert une IIFE ?
> Avant `let` et `const` (ES6), les IIFE servaient a creer un scope isole pour eviter de polluer le scope global. Aujourd'hui, les modules remplissent ce role. On les rencontre encore dans du code legacy.

### Comparaison

```
C :       int add(int a, int b) { return a + b; }     // Typage explicite
Python :  def add(a, b): return a + b                  # Pas d'accolades
JS :      const add = (a, b) => a + b;                 // Arrow function
```

---

## 15. Portee (Scope)

### Les trois niveaux de portee

```javascript
// ─── 1. Portee globale ───
var globaleVar = "visible partout";        // var dans le global
let globale = "aussi globale";             // let dans le global

// ─── 2. Portee de fonction ───
function maFonction() {
    var localeFn = "visible dans la fonction";
    let aussiFn = "aussi dans la fonction";
    // globale est accessible ici
}
// localeFn et aussiFn ne sont PAS accessibles ici

// ─── 3. Portee de bloc (let/const seulement) ───
if (true) {
    let localeBloc = "visible dans le if";
    var pasBlocScope = "sort du if !";     // var ignore la portee de bloc
}
// localeBloc NON accessible ici
// pasBlocScope EST accessible ici (var !)
```

### Schema visuel de la portee

```
┌─── Portee Globale ──────────────────────────────────────┐
│  let a = 1;                                             │
│                                                         │
│  ┌─── Portee Fonction f() ────────────────────────┐     │
│  │  let b = 2;                                    │     │
│  │  // a est accessible (portee parente)          │     │
│  │                                                │     │
│  │  ┌─── Portee Bloc (if) ─────────────────┐     │     │
│  │  │  let c = 3;                          │     │     │
│  │  │  // a et b sont accessibles          │     │     │
│  │  └──────────────────────────────────────┘     │     │
│  │  // c n'est PAS accessible ici               │     │
│  └────────────────────────────────────────────────┘     │
│  // b et c ne sont PAS accessibles ici                  │
└─────────────────────────────────────────────────────────┘
```

---

## 16. Hoisting (remontee des declarations)

Le hoisting est le mecanisme par lequel JavaScript "remonte" les declarations en haut de leur portee **avant l'execution**.

### var : hoisted avec undefined

```javascript
console.log(x); // undefined (pas d'erreur !)
var x = 5;
console.log(x); // 5

// Le moteur JS interprete comme :
// var x;            <-- declaration remontee
// console.log(x);  <-- undefined
// x = 5;           <-- assignation reste en place
// console.log(x);  <-- 5
```

### let / const : hoisted mais TDZ (Temporal Dead Zone)

```javascript
// console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 10;

// let y est hoisted, mais dans la TDZ entre le debut du bloc
// et la ligne de declaration. Tout acces dans la TDZ = erreur.
```

### Fonctions : entierement hoisted (declarations seulement)

```javascript
// ─── Function declaration : entierement hoisted ───
saluer();  // "Bonjour !" (fonctionne avant la declaration)

function saluer() {
    console.log("Bonjour !");
}

// ─── Function expression : PAS hoisted ───
// direAuRevoir(); // TypeError: direAuRevoir is not a function

const direAuRevoir = function() {
    console.log("Au revoir !");
};
```

### Resumé du hoisting

```
+---------------------+---------------------------+------------------+
| Declaration         | Hoisted ?                 | Avant la ligne   |
+---------------------+---------------------------+------------------+
| var x = 5;          | Oui (var x;)              | undefined        |
| let x = 5;          | Oui mais TDZ              | ReferenceError   |
| const x = 5;        | Oui mais TDZ              | ReferenceError   |
| function f() {}     | Oui (complete)            | Utilisable       |
| const f = () => {}  | Oui mais TDZ (const)      | ReferenceError   |
+---------------------+---------------------------+------------------+
```

> [!tip] Analogie
> Imaginez un professeur qui recoit les copies d'examen (votre code). Avant de les corriger, il fait un premier passage pour lister tous les noms des etudiants (declarations). Les `var` sont inscrits avec la mention "present" (`undefined`), les `let/const` sont inscrits mais avec un cadenas (TDZ) qui ne s'ouvrira qu'a la bonne ligne, et les `function` declarations sont inscrites avec la copie complete.

---

## 17. Closures

Une **closure** est une fonction qui "se souvient" de l'environnement dans lequel elle a ete creee, meme apres que cet environnement a cesse d'exister.

### Concept de base

```javascript
function creerCompteur() {
    let compte = 0;              // variable locale

    return function() {          // cette fonction "capture" compte
        compte++;
        return compte;
    };
}

const compteur = creerCompteur();
console.log(compteur());  // 1
console.log(compteur());  // 2
console.log(compteur());  // 3
// 'compte' n'est pas accessible directement, mais la closure y accede
```

### Visualisation

```
creerCompteur() retourne une fonction
   │
   │  ┌─── Environnement capture (closure) ───┐
   │  │  compte = 0                            │
   │  │  ┌─── Fonction retournee ───────┐      │
   │  │  │  compte++;                   │      │
   │  │  │  return compte;              │      │
   │  │  └──────────────────────────────┘      │
   │  └────────────────────────────────────────┘
   │
   └──> const compteur = [fonction + son environnement]
```

### Exemples pratiques

```javascript
// ─── 1. Fonction d'usine (factory) ───
function creerMultiplicateur(facteur) {
    return function(nombre) {
        return nombre * facteur;
    };
}

const double = creerMultiplicateur(2);
const triple = creerMultiplicateur(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15

// ─── 2. Donnees privees (encapsulation) ───
function creerBanque() {
    let solde = 0;  // "prive" grace a la closure

    return {
        deposer(montant) {
            solde += montant;
            return `Depot de ${montant}. Solde : ${solde}`;
        },
        retirer(montant) {
            if (montant > solde) return "Fonds insuffisants";
            solde -= montant;
            return `Retrait de ${montant}. Solde : ${solde}`;
        },
        consulter() {
            return `Solde : ${solde}`;
        }
    };
}

const compte = creerBanque();
console.log(compte.deposer(100));   // "Depot de 100. Solde : 100"
console.log(compte.retirer(30));    // "Retrait de 30. Solde : 70"
console.log(compte.consulter());    // "Solde : 70"
// compte.solde est undefined (pas accessible directement)
```

### Le piege classique de la closure dans une boucle

```javascript
// ─── PROBLEME avec var ───
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i);   // Affiche 3, 3, 3 (pas 0, 1, 2)
    }, 100);
}
// Toutes les callbacks partagent le MEME i (var = portee fonction)

// ─── SOLUTION avec let ───
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i);   // Affiche 0, 1, 2
    }, 100);
}
// let cree un nouveau i pour chaque iteration du bloc
```

> [!info] Closures dans d'autres langages
> En C, il n'y a pas de closures (pas de fonctions imbriquees standard). En Python, les closures existent mais les variables capturees sont en lecture seule sauf avec le mot-cle `nonlocal`. En JS, les closures sont naturelles et omni-presentes.

---

## 18. Strict mode

Le mode strict (`"use strict"`) active un mode d'execution plus rigoureux de JavaScript.

```javascript
"use strict";

// ─── Empeche les variables non declarees ───
// x = 10;              // ReferenceError (sans strict : creerait une globale !)

// ─── Empeche la suppression de variables ───
let y = 20;
// delete y;            // SyntaxError

// ─── Empeche les parametres dupliques ───
// function f(a, a) {}  // SyntaxError

// ─── this vaut undefined dans les fonctions simples ───
function test() {
    console.log(this);   // undefined (sans strict : window)
}
```

> [!info] Quand utiliser strict mode ?
> Les modules ES6 (`import`/`export`) sont **automatiquement** en mode strict. Pour du code non-modulaire, ajoutez `"use strict";` en debut de fichier ou de fonction.

---

## 19. Tableau recapitulatif C vs Python vs JavaScript

```
+--------------------+------------------+------------------+--------------------+
| Aspect             | C                | Python           | JavaScript         |
+--------------------+------------------+------------------+--------------------+
| Typage             | Statique, fort   | Dynamique, fort  | Dynamique, faible  |
| Declarations       | int x = 5;       | x = 5            | let x = 5;         |
| Constantes         | const int X = 5; | X = 5 (conv.)    | const X = 5;       |
| Affichage          | printf()         | print()          | console.log()      |
| Fonctions          | int f(int x){}   | def f(x):        | function f(x) {}   |
| Lambda             | (pas natif)      | lambda x: x*2    | x => x * 2        |
| Null               | NULL             | None             | null / undefined   |
| Booleens           | 0 / 1            | True / False      | true / false       |
| Egalite            | ==               | ==               | === (strict)       |
| Strings            | char[]           | "..." / '...'    | "..." / '...' / `` |
| Interpolation      | sprintf          | f"...{x}..."     | `...${x}...`       |
| Portee             | Bloc             | Fonc./module     | Bloc (let/const)   |
| Closures           | Non              | Oui (nonlocal)   | Oui (naturelles)   |
| Compilation        | gcc => binaire   | Interprete       | JIT (V8, etc.)     |
| Point-virgule      | Obligatoire      | Non              | Optionnel          |
+--------------------+------------------+------------------+--------------------+
```

---

## Carte Mentale ASCII

```
                        JavaScript - Introduction
                                 │
          ┌──────────┬───────────┼───────────┬──────────────┐
          │          │           │           │              │
      Historique  Execution   Variables   Types        Fonctions
          │          │           │           │              │
      Eich 1995  Navigateur  var/let     Primitifs    Declaration
      ECMAScript Node.js     const       ├─ number    Expression
      ES6+ 2015  Console     Portee      ├─ string    Arrow =>
          │          │        Hoisting    ├─ boolean   IIFE
          │      <script>    TDZ         ├─ null      Closures
          │      defer/async              ├─ undefined
          │                               ├─ symbol
          │                               ├─ bigint
          │                               │
          │                              Objet
          │                               ├─ Object
          │                               ├─ Array
          │                               └─ Function
          │
      ┌───┴───────────┐
      │               │
   Controle       Operateurs
      │               │
   if/else        Arithmetiques
   switch         Logiques (&&, ||, !)
   for/while      Comparaison (=== !)
   for..of/in     Assignation (+=, etc.)
   ternaire       Coercion (== pieges)
```

---

## Exercices

### Exercice 1 : Explorateur de types

Ecrivez une fonction `analyserType(valeur)` qui retourne un objet contenant :
- `type` : le resultat de `typeof`
- `truthy` : `true` ou `false` selon que la valeur est truthy ou falsy
- `description` : une chaine decrivant le type reel (ex: "null", "array", "NaN", "number", etc.)

Testez avec : `42`, `"hello"`, `true`, `null`, `undefined`, `NaN`, `[]`, `{}`, `0`, `""`.

### Exercice 2 : Compteur configurable avec closure

Creez une fonction `creerCompteur(debut, pas)` qui retourne un objet avec :
- `suivant()` : incremente de `pas` et retourne la valeur
- `precedent()` : decremente de `pas` et retourne la valeur
- `valeur()` : retourne la valeur actuelle sans modifier
- `reinitialiser()` : remet le compteur a `debut`

### Exercice 3 : Convertisseur de types

Creez un programme qui prend une valeur et affiche un tableau montrant sa conversion en `Number`, `String`, et `Boolean`, ainsi que le resultat de `== 0`, `=== 0`, `== ""`, `=== ""`, `== null`, `=== null`.

### Exercice 4 : Quiz sur le hoisting

Sans executer le code, predisez la sortie de chaque bloc ci-dessous, puis verifiez dans la console :

```javascript
// Bloc A
console.log(a);
var a = 1;
console.log(a);

// Bloc B
console.log(b);
let b = 2;

// Bloc C
saluer();
function saluer() { console.log("Hello"); }

// Bloc D
direBonjour();
var direBonjour = function() { console.log("Bonjour"); };
```

---

## Liens

- [[02 - JavaScript DOM et Evenements]]
- [[03 - JavaScript Asynchrone]]
- [[04 - JavaScript Moderne ES6+]]
- [[01 - Introduction a Python]]
- [[01 - Introduction au C et Compilation]]
