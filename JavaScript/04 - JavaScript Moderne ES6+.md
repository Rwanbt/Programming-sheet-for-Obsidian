# JavaScript Moderne ES6+

ECMAScript 2015 (ES6) a revolutionne JavaScript en introduisant une multitude de fonctionnalites qui rendent le code plus lisible, plus sur et plus expressif. Les editions suivantes (ES2016 a ES2024) ont continue d'enrichir le langage avec des ajouts pratiques.

Ce cours couvre les fonctionnalites modernes essentielles : destructuring, spread/rest, arrow functions en profondeur, optional chaining, nullish coalescing, iterateurs et generateurs, Map/Set, modules, et classes.

> [!tip] Analogie
> Si JavaScript classique (ES5) est une boite a outils de base (marteau, tournevis, pince), ES6+ est un atelier complet avec des outils electriques, des gabarits, et des systemes de rangement. Vous pouvez toujours construire la meme chose, mais avec moins d'effort, moins d'erreurs, et un resultat plus propre. Python a eu cette philosophie ergonomique des le depart. C, fidele a sa nature, reste minimaliste.

---

## 1. Destructuring

Le destructuring permet d'extraire des valeurs d'un tableau ou d'un objet directement dans des variables distinctes.

### Destructuring de tableaux

```javascript
// ─── Extraction basique ───
const couleurs = ["rouge", "vert", "bleu", "jaune"];
const [premier, deuxieme] = couleurs;
console.log(premier);   // "rouge"
console.log(deuxieme);  // "vert"

// ─── Ignorer des elements ───
const [, , troisieme] = couleurs;
console.log(troisieme); // "bleu"

// ─── Rest element (capturer le reste) ───
const [tete, ...reste] = couleurs;
console.log(tete);      // "rouge"
console.log(reste);     // ["vert", "bleu", "jaune"]

// ─── Valeurs par defaut ───
const [a, b, c, d, e = "noir"] = couleurs;
console.log(e);         // "noir" (la 5e valeur n'existe pas)

// ─── Echange de variables (swap) ───
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y);     // 2, 1  (sans variable temporaire !)
```

### Destructuring d'objets

```javascript
const utilisateur = {
    nom: "Alice",
    age: 25,
    ville: "Paris",
    role: "admin"
};

// ─── Extraction basique (le nom doit correspondre a la cle) ───
const { nom, age } = utilisateur;
console.log(nom);   // "Alice"
console.log(age);   // 25

// ─── Renommer les variables ───
const { nom: prenom, ville: localisation } = utilisateur;
console.log(prenom);       // "Alice"
console.log(localisation); // "Paris"

// ─── Valeurs par defaut ───
const { email = "non renseigne" } = utilisateur;
console.log(email);  // "non renseigne"

// ─── Renommer + valeur par defaut ───
const { email: mail = "inconnu" } = utilisateur;
console.log(mail);   // "inconnu"

// ─── Rest properties ───
const { nom: n, ...infos } = utilisateur;
console.log(infos);  // { age: 25, ville: "Paris", role: "admin" }
```

### Destructuring imbrique (nested)

```javascript
const entreprise = {
    nom: "TechCorp",
    adresse: {
        rue: "42 rue du Code",
        ville: "Paris",
        codePostal: "75001"
    },
    employes: [
        { prenom: "Alice", poste: "Dev" },
        { prenom: "Bob", poste: "Design" }
    ]
};

// ─── Acces imbrique ───
const {
    nom: nomEntreprise,
    adresse: { ville, codePostal },
    employes: [premierEmploye, deuxiemeEmploye]
} = entreprise;

console.log(nomEntreprise);            // "TechCorp"
console.log(ville);                    // "Paris"
console.log(premierEmploye.prenom);    // "Alice"
```

### Destructuring dans les parametres de fonction

```javascript
// ─── Sans destructuring ───
function afficherOld(utilisateur) {
    console.log(`${utilisateur.nom} a ${utilisateur.age} ans`);
}

// ─── Avec destructuring ───
function afficher({ nom, age, ville = "inconnue" }) {
    console.log(`${nom} a ${age} ans, habite a ${ville}`);
}

afficher({ nom: "Alice", age: 25 });
// "Alice a 25 ans, habite a inconnue"

// ─── Avec des tableaux ───
function premierEtDernier([premier, ...reste]) {
    const dernier = reste[reste.length - 1] || premier;
    return { premier, dernier };
}

console.log(premierEtDernier([1, 2, 3, 4])); // { premier: 1, dernier: 4 }
```

### Comparaison avec Python

```
Python (unpacking, similaire) :
─────────────────────────────
a, b, c = [1, 2, 3]                    # Tuple/list unpacking
a, *reste = [1, 2, 3, 4]               # Star expression
premier, _, troisieme = [1, 2, 3]      # Ignorer avec _

JavaScript (destructuring) :
────────────────────────────
const [a, b, c] = [1, 2, 3];
const [a, ...reste] = [1, 2, 3, 4];
const [premier, , troisieme] = [1, 2, 3];

Python n'a PAS de destructuring d'objets/dicts :
────────────────────────────────────────────────
d = {"nom": "Alice", "age": 25}
nom = d["nom"]                    # Acces manuel
age = d["age"]

JavaScript :
────────────
const { nom, age } = { nom: "Alice", age: 25 };  // Direct !
```

---

## 2. Spread Operator (...)

L'operateur spread (`...`) "etale" les elements d'un iterable (tableau, objet, string).

### Spread avec les tableaux

```javascript
// ─── Copier un tableau (shallow copy) ───
const original = [1, 2, 3];
const copie = [...original];
copie.push(4);
console.log(original); // [1, 2, 3] (non modifie)
console.log(copie);    // [1, 2, 3, 4]

// ─── Fusionner des tableaux ───
const a = [1, 2];
const b = [3, 4];
const fusionne = [...a, ...b];     // [1, 2, 3, 4]
const avec = [...a, 99, ...b];     // [1, 2, 99, 3, 4]

// ─── Convertir un iterable en tableau ───
const lettres = [..."hello"];      // ["h", "e", "l", "l", "o"]
const unique = [...new Set([1, 1, 2, 3, 3])]; // [1, 2, 3]

// ─── Passer des elements comme arguments ───
const nombres = [3, 1, 4, 1, 5];
Math.max(...nombres);  // 5 (au lieu de Math.max.apply(null, nombres))
```

### Spread avec les objets

```javascript
// ─── Copier un objet (shallow copy) ───
const user = { nom: "Alice", age: 25 };
const copie = { ...user };

// ─── Fusionner des objets (le dernier gagne en cas de conflit) ───
const defauts = { theme: "clair", langue: "fr", taille: 14 };
const prefs = { theme: "sombre", taille: 16 };
const config = { ...defauts, ...prefs };
// { theme: "sombre", langue: "fr", taille: 16 }

// ─── Ajouter/modifier des proprietes ───
const updated = { ...user, age: 26, email: "alice@mail.com" };
// { nom: "Alice", age: 26, email: "alice@mail.com" }
```

> [!warning] Spread fait une copie superficielle (shallow copy)
> ```javascript
> const original = { a: 1, nested: { b: 2 } };
> const copie = { ...original };
> copie.nested.b = 99;
> console.log(original.nested.b); // 99 ! (meme reference)
>
> // Pour une copie profonde :
> const deep = structuredClone(original); // ES2022
> ```

---

## 3. Rest Parameters (...args)

L'operateur rest (`...`) dans les parametres collecte les arguments restants dans un tableau.

```javascript
// ─── Collecter tous les arguments ───
function somme(...nombres) {
    return nombres.reduce((acc, n) => acc + n, 0);
}
somme(1, 2, 3);       // 6
somme(1, 2, 3, 4, 5); // 15

// ─── Collecter les arguments restants ───
function log(niveau, ...messages) {
    const prefix = `[${niveau.toUpperCase()}]`;
    console.log(prefix, ...messages);
}
log("info", "Serveur", "demarre", "sur le port", 3000);
// [INFO] Serveur demarre sur le port 3000

// ─── Comparaison avec C et Python ───
// C :     void f(int n, ...) { va_list args; ... }  // Fastidieux
// Python : def f(*args, **kwargs): ...                // *args = positionnels
// JS :     function f(...args) { }                    // Un seul rest, en dernier
```

> [!info] Rest vs arguments
> Avant ES6, on utilisait l'objet special `arguments` (array-like, pas un vrai Array). `...rest` est un vrai Array avec toutes ses methodes. Preferez toujours `...rest`.

### Spread vs Rest : meme syntaxe, contextes differents

```
SPREAD (etaler) :  utilise dans les appels/litteraux
─────────────────
  Math.max(...nombres)       // Etale le tableau en arguments
  const copie = [...arr]     // Etale dans un nouveau tableau
  const obj = { ...source }  // Etale dans un nouvel objet

REST (collecter) : utilise dans les declarations/parametres
──────────────────
  function f(...args) {}     // Collecte les arguments
  const [a, ...reste] = arr  // Collecte le reste du destructuring
  const { x, ...other } = obj // Collecte le reste des proprietes
```

---

## 4. Arrow Functions en profondeur

### Le probleme de this

```javascript
// ─── Probleme classique avec les fonctions normales ───
const compteur = {
    valeur: 0,
    incrementer() {
        // Ici, this = compteur (methode d'objet)
        setTimeout(function() {
            // Ici, this = window (ou undefined en strict mode)
            // PAS compteur !
            this.valeur++;  // NE FONCTIONNE PAS
            console.log(this.valeur); // NaN
        }, 1000);
    }
};

// ─── Solution ES5 : sauvegarder this ───
const compteur2 = {
    valeur: 0,
    incrementer() {
        const self = this;  // Sauvegarde
        setTimeout(function() {
            self.valeur++;  // Utilise self au lieu de this
        }, 1000);
    }
};

// ─── Solution ES6 : arrow function ───
const compteur3 = {
    valeur: 0,
    incrementer() {
        setTimeout(() => {
            // Arrow function : this = this du contexte parent = compteur3
            this.valeur++;
            console.log(this.valeur); // 1
        }, 1000);
    }
};
```

### Regles du this dans les arrow functions

```
Fonction normale :     this est determine a L'APPEL
                       (depend de comment la fonction est appelee)

Arrow function :       this est determine a la CREATION
                       (herite du this du scope englobant, LEXICAL)
```

```
┌──────────────────────────────────────────────────────────────┐
│  const obj = {                                               │
│      valeur: 42,                                             │
│      methode() {              // this = obj                  │
│          │                                                   │
│          ├── function() {}    // this = ?? (depend de l'appel)│
│          │                                                   │
│          └── () => {}         // this = obj (herite)         │
│      }                                                       │
│  };                                                          │
└──────────────────────────────────────────────────────────────┘
```

### Quand NE PAS utiliser les arrow functions

```javascript
// ─── 1. Methodes d'objet ───
const personne = {
    nom: "Alice",
    // MAUVAIS : arrow function, this = contexte global
    saluer: () => {
        console.log(`Bonjour, je suis ${this.nom}`); // undefined !
    },
    // BON : methode shorthand
    saluer() {
        console.log(`Bonjour, je suis ${this.nom}`); // "Alice"
    }
};

// ─── 2. Constructeurs ───
// Les arrow functions ne peuvent PAS etre utilisees avec new
const Personne = (nom) => { this.nom = nom; };
// new Personne("Alice"); // TypeError: Personne is not a constructor

// ─── 3. Quand vous avez besoin de arguments ───
// Les arrow functions n'ont PAS leur propre objet arguments
const f = () => {
    // console.log(arguments); // ReferenceError
};
// Utilisez ...rest a la place : const f = (...args) => { }

// ─── 4. Prototypes ───
function Animal(nom) { this.nom = nom; }
// MAUVAIS
Animal.prototype.parler = () => {
    console.log(this.nom); // undefined
};
// BON
Animal.prototype.parler = function() {
    console.log(this.nom); // Correct
};
```

> [!warning] Resume : quand utiliser quoi ?
> - **Arrow function** : callbacks, `.map()`, `.filter()`, `.then()`, event handlers simples, fonctions imbriquees
> - **Function normale** : methodes d'objet, constructeurs, quand vous avez besoin d'un `this` dynamique

---

## 5. Template Literals avances (Tagged Templates)

```javascript
// ─── Tagged templates : fonctions appliquees aux template literals ───
function majuscules(strings, ...valeurs) {
    // strings = parties statiques : ["Bonjour ", " ! Tu as ", " ans."]
    // valeurs = expressions interpolees : ["Alice", 25]
    return strings.reduce((result, str, i) => {
        const val = valeurs[i] !== undefined
            ? String(valeurs[i]).toUpperCase()
            : "";
        return result + str + val;
    }, "");
}

const nom = "Alice";
const age = 25;
const msg = majuscules`Bonjour ${nom} ! Tu as ${age} ans.`;
// "Bonjour ALICE ! Tu as 25 ans."

// ─── Cas d'usage pratique : echappement HTML ───
function html(strings, ...valeurs) {
    const echapper = (str) => String(str)
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;");

    return strings.reduce((result, str, i) => {
        return result + str + (valeurs[i] !== undefined ? echapper(valeurs[i]) : "");
    }, "");
}

const userInput = '<script>alert("xss")</script>';
const safe = html`<p>Message : ${userInput}</p>`;
// "<p>Message : &lt;script&gt;alert(&quot;xss&quot;)&lt;/script&gt;</p>"

// ─── Cas d'usage : requetes SQL parametrees ───
function sql(strings, ...valeurs) {
    const query = strings.join("?");
    return { query, params: valeurs };
}

const userId = 42;
const result = sql`SELECT * FROM users WHERE id = ${userId} AND active = ${true}`;
// { query: "SELECT * FROM users WHERE id = ? AND active = ?", params: [42, true] }
```

---

## 6. Optional Chaining (?.)

L'optional chaining permet d'acceder a des proprietes profondes sans verifier chaque niveau.

```javascript
const user = {
    nom: "Alice",
    adresse: {
        ville: "Paris"
    }
    // pas de propriete 'preferences'
};

// ─── Sans optional chaining (verbeux et fragile) ───
const theme = user.preferences && user.preferences.theme
    ? user.preferences.theme
    : "defaut";

// ─── Avec optional chaining ───
const theme2 = user.preferences?.theme;  // undefined (pas d'erreur !)
const ville = user.adresse?.ville;       // "Paris"

// ─── Sur des methodes ───
user.saluer?.();          // undefined si saluer n'existe pas, sinon l'appelle

// ─── Sur des tableaux ───
const premier = user.commandes?.[0];     // undefined si commandes n'existe pas

// ─── Enchainement profond ───
const codePostal = user.adresse?.details?.codePostal?.toString();
// undefined a chaque niveau si la propriete n'existe pas

// ─── Equivalent en Python ───
// Python n'a pas de ?. natif
// theme = getattr(getattr(user, 'preferences', None), 'theme', 'defaut')
// Ou avec un try/except
```

```
Sans ?. :                              Avec ?. :
─────────                              ─────────
if (user &&                            user.adresse?.details?.codePostal
    user.adresse &&
    user.adresse.details &&
    user.adresse.details.codePostal)
```

---

## 7. Nullish Coalescing (??) et Short-Circuit

### Nullish Coalescing (??)

```javascript
// ?? retourne le cote droit SEULEMENT si le gauche est null ou undefined
// (PAS pour 0, "", false, NaN)

const a = null ?? "defaut";      // "defaut"
const b = undefined ?? "defaut"; // "defaut"
const c = 0 ?? "defaut";        // 0         (0 n'est pas null/undefined)
const d = "" ?? "defaut";       // ""        ("" n'est pas null/undefined)
const e = false ?? "defaut";    // false     (false n'est pas null/undefined)

// ─── Comparaison avec || ───
const f = 0 || "defaut";        // "defaut"  (|| traite 0 comme falsy !)
const g = 0 ?? "defaut";        // 0         (?? ne traite que null/undefined)

const h = "" || "defaut";       // "defaut"  (|| traite "" comme falsy)
const i = "" ?? "defaut";       // ""        (?? garde la chaine vide)
```

> [!tip] Analogie
> `||` est un gardien strict qui rejette tout ce qui lui semble "faux" (0, "", false, null, undefined). `??` est un gardien qui ne rejette que les "absents" (null, undefined). Si un utilisateur a explicitement choisi la valeur 0 ou "", `??` la respecte.

### Tableau comparatif || vs ??

```
+------------------+-------------------+-------------------+
| Expression       | || "defaut"       | ?? "defaut"       |
+------------------+-------------------+-------------------+
| null             | "defaut"          | "defaut"          |
| undefined        | "defaut"          | "defaut"          |
| 0                | "defaut"          | 0                 |
| ""               | "defaut"          | ""                |
| false            | "defaut"          | false             |
| NaN              | "defaut"          | NaN               |
| "hello"          | "hello"           | "hello"           |
| 42               | 42                | 42                |
+------------------+-------------------+-------------------+
```

### Short-circuit evaluation (&&, ||)

```javascript
// ─── && : retourne la premiere valeur falsy, ou la derniere ───
const a = true && "hello";       // "hello"
const b = false && "hello";      // false
const c = "foo" && "bar" && 0;   // 0

// Utilisation courante : execution conditionnelle
const user = { isAdmin: true };
user.isAdmin && afficherPanelAdmin();  // Execute si isAdmin est truthy

// ─── || : retourne la premiere valeur truthy, ou la derniere ───
const d = "" || "defaut";        // "defaut"
const e = "hello" || "defaut";   // "hello"

// Utilisation courante : valeur par defaut (attention aux falsy !)
const nom = inputNom || "Anonyme";

// ─── Combiner ?. et ?? ───
const theme = user.preferences?.theme ?? "clair";
// Si user.preferences est undefined -> undefined ?? "clair" -> "clair"
// Si user.preferences.theme est null -> null ?? "clair" -> "clair"
// Si user.preferences.theme est "" -> "" (pas null/undefined)
```

---

## 8. Enhanced Object Literals

```javascript
// ─── Raccourci de propriete (shorthand) ───
const nom = "Alice";
const age = 25;

// Ancien
const user1 = { nom: nom, age: age };
// Moderne (si le nom de la variable = le nom de la cle)
const user2 = { nom, age };

// ─── Raccourci de methode ───
// Ancien
const obj1 = {
    saluer: function() { return "Bonjour"; }
};
// Moderne
const obj2 = {
    saluer() { return "Bonjour"; }
};

// ─── Noms de proprietes dynamiques (computed properties) ───
const champ = "email";
const obj3 = {
    [champ]: "alice@mail.com",           // { email: "alice@mail.com" }
    [`get${champ.charAt(0).toUpperCase() + champ.slice(1)}`]() {
        return this[champ];               // getEmail()
    }
};
console.log(obj3.email);       // "alice@mail.com"
console.log(obj3.getEmail());  // "alice@mail.com"
```

---

## 9. Symbols

Un Symbol est un **identifiant unique et immuable**, souvent utilise comme cle de propriete d'objet.

```javascript
// ─── Creation ───
const sym1 = Symbol("description");
const sym2 = Symbol("description");
console.log(sym1 === sym2);  // false (chaque Symbol est unique !)

// ─── Utilisation comme cle de propriete ───
const ID = Symbol("id");
const user = {
    nom: "Alice",
    [ID]: 42    // Propriete "cachee" avec un Symbol comme cle
};

console.log(user[ID]);       // 42
console.log(user.nom);       // "Alice"

// Les Symbols ne sont PAS enumeres par for...in ou Object.keys
for (let cle in user) {
    console.log(cle);  // "nom" seulement
}
console.log(Object.keys(user));  // ["nom"]

// Pour acceder aux Symbols :
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id)]

// ─── Symbols globaux (partages) ───
const s1 = Symbol.for("app.id");  // Cree ou recupere dans le registre global
const s2 = Symbol.for("app.id");
console.log(s1 === s2);           // true (meme Symbol global)
```

### Well-known Symbols (Symbols systeme)

```javascript
// ─── Symbol.iterator : rendre un objet iterable ───
const range = {
    start: 1,
    end: 5,
    [Symbol.iterator]() {
        let current = this.start;
        const end = this.end;
        return {
            next() {
                if (current <= end) {
                    return { value: current++, done: false };
                }
                return { done: true };
            }
        };
    }
};

for (const n of range) {
    console.log(n);  // 1, 2, 3, 4, 5
}
console.log([...range]);  // [1, 2, 3, 4, 5]

// ─── Symbol.toPrimitive : personnaliser la conversion ───
const money = {
    value: 42,
    currency: "EUR",
    [Symbol.toPrimitive](hint) {
        if (hint === "string") return `${this.value} ${this.currency}`;
        return this.value;  // "number" ou "default"
    }
};

console.log(`${money}`);   // "42 EUR"
console.log(money + 8);    // 50
```

---

## 10. Iterateurs et Generateurs

### Iterateurs (Iterator Protocol)

Un iterateur est un objet qui implemente la methode `next()`, retournant `{ value, done }`.

```javascript
// ─── Iterateur manuel ───
function creerIterateur(tableau) {
    let index = 0;
    return {
        next() {
            if (index < tableau.length) {
                return { value: tableau[index++], done: false };
            }
            return { value: undefined, done: true };
        }
    };
}

const it = creerIterateur(["a", "b", "c"]);
console.log(it.next()); // { value: "a", done: false }
console.log(it.next()); // { value: "b", done: false }
console.log(it.next()); // { value: "c", done: false }
console.log(it.next()); // { value: undefined, done: true }
```

### Generateurs (function*)

Les generateurs simplifient la creation d'iterateurs avec le mot-cle `yield`.

```javascript
// ─── Generateur basique ───
function* compteur(debut, fin) {
    for (let i = debut; i <= fin; i++) {
        yield i;  // "pause" et retourne la valeur
    }
}

const gen = compteur(1, 5);
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }

// Iterable avec for...of
for (const n of compteur(1, 5)) {
    console.log(n); // 1, 2, 3, 4, 5
}

// Spread
console.log([...compteur(1, 5)]); // [1, 2, 3, 4, 5]
```

```javascript
// ─── Generateur infini ───
function* fibonacci() {
    let a = 0, b = 1;
    while (true) {
        yield a;
        [a, b] = [b, a + b];
    }
}

// Prendre les 10 premiers nombres de Fibonacci
const fib = fibonacci();
const dix = [];
for (let i = 0; i < 10; i++) {
    dix.push(fib.next().value);
}
console.log(dix); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// ─── yield* : deleguer a un autre generateur ───
function* genA() {
    yield 1;
    yield 2;
}

function* genB() {
    yield* genA();  // Delegue a genA
    yield 3;
    yield 4;
}

console.log([...genB()]); // [1, 2, 3, 4]
```

### Comparaison avec Python

```
Python :                          JavaScript :
────────                          ────────────
def compteur(n):                  function* compteur(n) {
    for i in range(n):                for (let i = 0; i < n; i++) {
        yield i                           yield i;
                                      }
                                  }

# Utilisation identique :
for x in compteur(5):             for (const x of compteur(5)) {
    print(x)                          console.log(x);
                                  }
```

> [!info] Les generateurs sont paresseux (lazy)
> Comme en Python, les generateurs JavaScript ne calculent les valeurs que lorsqu'on les demande (`next()`). C'est ideal pour les sequences infinies ou les grandes collections.

---

## 11. Map et Set

### Map (dictionnaire ordonne)

```javascript
// ─── Creer un Map ───
const map = new Map();
map.set("nom", "Alice");
map.set(42, "un nombre comme cle");
map.set(true, "un boolean comme cle");

const objCle = { id: 1 };
map.set(objCle, "un objet comme cle"); // Les objets peuvent etre des cles !

// ─── Lire ───
map.get("nom");     // "Alice"
map.get(42);        // "un nombre comme cle"
map.has("nom");     // true
map.size;           // 4

// ─── Supprimer ───
map.delete(42);
map.clear();        // Tout supprimer

// ─── Iterer ───
const users = new Map([
    ["alice", { age: 25 }],
    ["bob", { age: 30 }]
]);

for (const [cle, valeur] of users) {
    console.log(`${cle}: ${valeur.age}`);
}

users.forEach((valeur, cle) => {
    console.log(`${cle}: ${valeur.age}`);
});

// ─── Conversion ───
const obj = Object.fromEntries(users);  // Map => Object
const map2 = new Map(Object.entries(obj)); // Object => Map
```

### Map vs Object

```
+--------------------+----------------------------+----------------------------+
| Aspect             | Object                     | Map                        |
+--------------------+----------------------------+----------------------------+
| Cles               | String / Symbol seulement  | N'importe quel type        |
| Ordre              | Non garanti (*)            | Ordre d'insertion          |
| Taille             | Object.keys(o).length      | map.size (O(1))            |
| Iteration          | for...in (+ prototype !)   | for...of (direct)          |
| Performance        | Bon pour petits objets     | Meilleur pour ajout/suppr. |
| Serialisation JSON | JSON.stringify(o)          | Pas directe                |
| Prototype          | Herite de Object.prototype | Pas de pollution           |
+--------------------+----------------------------+----------------------------+
(*) En pratique, l'ordre est garanti depuis ES2015 pour les cles non-entieres
```

### Set (ensemble de valeurs uniques)

```javascript
// ─── Creer un Set ───
const set = new Set([1, 2, 3, 3, 2, 1]);
console.log(set);      // Set(3) { 1, 2, 3 }  (doublons elimines)

// ─── Ajouter / Supprimer ───
set.add(4);
set.add(1);            // Ignore (deja present)
set.delete(2);
set.has(3);            // true
set.size;              // 3

// ─── Cas d'usage : dedoublonner un tableau ───
const tableau = [1, 2, 2, 3, 3, 3, 4];
const unique = [...new Set(tableau)];  // [1, 2, 3, 4]

// ─── Operations ensemblistes ───
const a = new Set([1, 2, 3, 4]);
const b = new Set([3, 4, 5, 6]);

// Union
const union = new Set([...a, ...b]);          // {1, 2, 3, 4, 5, 6}

// Intersection
const inter = new Set([...a].filter(x => b.has(x)));  // {3, 4}

// Difference
const diff = new Set([...a].filter(x => !b.has(x)));  // {1, 2}

// ES2025 (propose) : methodes natives
// a.union(b), a.intersection(b), a.difference(b)
```

### WeakMap et WeakSet

```javascript
// Les cles d'un WeakMap sont des REFERENCES FAIBLES
// Si l'objet cle n'est plus reference ailleurs, il peut etre garbage-collected

let obj = { id: 1 };
const weakMap = new WeakMap();
weakMap.set(obj, "donnees associees");

obj = null;  // L'entree dans le WeakMap sera aussi nettoyee (GC)

// Cas d'usage : stocker des metadonnees sans empecher le garbage collection
// Pas de .size, pas iterable, pas de .clear()
```

> [!example] Comparaison avec Python
> ```
> Python :                    JavaScript :
> ────────                    ────────────
> d = {"cle": "val"}         const m = new Map([["cle", "val"]]);
> s = {1, 2, 3}              const s = new Set([1, 2, 3]);
> s1 & s2  (intersection)    new Set([...s1].filter(x => s2.has(x)))
> s1 | s2  (union)           new Set([...s1, ...s2])
> s1 - s2  (difference)      new Set([...s1].filter(x => !s2.has(x)))
> ```

---

## 12. Modules (import / export)

### Exporter

```javascript
// ─── fichier : mathUtils.js ───

// Export nomme (named export)
export const PI = 3.14159;

export function addition(a, b) {
    return a + b;
}

export function multiplication(a, b) {
    return a * b;
}

// Export par defaut (un seul par fichier)
export default class Calculatrice {
    static calculer(expression) {
        return eval(expression); // Simplifie pour l'exemple
    }
}
```

### Importer

```javascript
// ─── fichier : app.js ───

// Import nomme (accolades obligatoires)
import { addition, PI } from "./mathUtils.js";

// Import par defaut (pas d'accolades, nom au choix)
import Calculatrice from "./mathUtils.js";

// Import mixte
import Calc, { addition as add, PI } from "./mathUtils.js";

// Import de tout
import * as MathUtils from "./mathUtils.js";
MathUtils.addition(2, 3);
MathUtils.PI;
MathUtils.default; // La classe Calculatrice

// Re-export
export { addition, PI } from "./mathUtils.js";
export { default as Calc } from "./mathUtils.js";
```

### Import dynamique (chargement conditionnel)

```javascript
// ─── Chargement a la demande (code splitting) ───
async function chargerModule() {
    if (condition) {
        const { default: Module, helperFn } = await import("./monModule.js");
        Module.init();
    }
}

// Utile pour :
// - Charger un module uniquement si necessaire
// - Reduire la taille du bundle initial
// - Charger conditionnellement selon la plateforme
```

### Utilisation dans le HTML

```html
<!-- type="module" active les imports/exports -->
<script type="module" src="app.js"></script>

<!-- Les modules sont automatiquement en mode strict -->
<!-- Les modules sont deferes par defaut (comme defer) -->
<!-- Les modules ont leur propre scope (pas de pollution globale) -->
```

### Comparaison des systemes de modules

```
+------------------+--------------------------------+-----------------------------+
| Systeme          | Syntaxe                        | Contexte                    |
+------------------+--------------------------------+-----------------------------+
| ES Modules       | import/export                  | Standard JS (navigateur+Node)|
| CommonJS         | require() / module.exports     | Node.js (historique)        |
| Python           | import / from...import         | Standard Python             |
| C                | #include                       | Preprocesseur (pas de module)|
+------------------+--------------------------------+-----------------------------+
```

> [!warning] CommonJS vs ES Modules dans Node.js
> Node.js supporte les deux systemes. Les fichiers `.mjs` ou avec `"type": "module"` dans package.json utilisent ES Modules. Les fichiers `.cjs` ou par defaut utilisent CommonJS. Preferez ES Modules pour les nouveaux projets.

---

## 13. Classes

Les classes JavaScript sont du **sucre syntaxique** au-dessus du systeme de prototypes.

### Syntaxe de base

```javascript
class Animal {
    // ─── Constructeur ───
    constructor(nom, age) {
        this.nom = nom;
        this.age = age;
    }

    // ─── Methode ───
    parler() {
        console.log(`${this.nom} fait un bruit.`);
    }

    // ─── Getter ───
    get info() {
        return `${this.nom} (${this.age} ans)`;
    }

    // ─── Setter ───
    set nouveauNom(nom) {
        if (nom.length < 2) throw new Error("Nom trop court");
        this.nom = nom;
    }

    // ─── Methode statique ───
    static creer(nom, age) {
        return new Animal(nom, age);
    }

    // ─── Champ statique ───
    static especes = [];

    // ─── toString personnalise ───
    toString() {
        return this.info;
    }
}

const chat = new Animal("Felix", 3);
chat.parler();          // "Felix fait un bruit."
console.log(chat.info); // "Felix (3 ans)" (getter, pas de ())
chat.nouveauNom = "Minou"; // setter

const chien = Animal.creer("Rex", 5); // methode statique
```

### Heritage (extends / super)

```javascript
class Chat extends Animal {
    constructor(nom, age, couleur) {
        super(nom, age);       // Appelle le constructeur parent
        this.couleur = couleur;
    }

    // ─── Override de methode ───
    parler() {
        console.log(`${this.nom} dit : Miaou !`);
    }

    // ─── Methode specifique ───
    ronronner() {
        console.log(`${this.nom} ronronne...`);
    }

    // ─── Appeler la methode parent ───
    sePresenter() {
        super.parler();  // Appelle Animal.parler()
        console.log(`Je suis un chat ${this.couleur}`);
    }
}

const minou = new Chat("Minou", 2, "roux");
minou.parler();       // "Minou dit : Miaou !"
minou.ronronner();    // "Minou ronronne..."
minou.sePresenter();  // "Minou fait un bruit." puis "Je suis un chat roux"

// ─── Verification d'instance ───
minou instanceof Chat;   // true
minou instanceof Animal; // true
```

### Proprietes et methodes privees (#)

```javascript
class CompteBancaire {
    // ─── Champs prives (ES2022) ───
    #solde;
    #historique = [];

    constructor(proprietaire, soldeInitial) {
        this.proprietaire = proprietaire; // Public
        this.#solde = soldeInitial;       // Prive
    }

    // ─── Methode privee ───
    #enregistrer(operation) {
        this.#historique.push({
            ...operation,
            date: new Date().toISOString()
        });
    }

    deposer(montant) {
        if (montant <= 0) throw new Error("Montant invalide");
        this.#solde += montant;
        this.#enregistrer({ type: "depot", montant });
        return this;  // Pour le chaining
    }

    retirer(montant) {
        if (montant > this.#solde) throw new Error("Solde insuffisant");
        this.#solde -= montant;
        this.#enregistrer({ type: "retrait", montant });
        return this;
    }

    get solde() {
        return this.#solde;
    }

    get releve() {
        return [...this.#historique]; // Copie pour eviter la mutation
    }
}

const compte = new CompteBancaire("Alice", 1000);
compte.deposer(500).retirer(200);  // Chaining
console.log(compte.solde);         // 1300
console.log(compte.releve);        // [{type: "depot",...}, {type: "retrait",...}]
// compte.#solde;                   // SyntaxError: Private field
```

### Comparaison avec Python

```
Python :                                JavaScript :
────────                                ────────────
class Animal:                           class Animal {
    especes = []    # class var             static especes = [];

    def __init__(self, nom, age):           constructor(nom, age) {
        self.nom = nom                          this.nom = nom;
        self.__secret = 42  # "prive"           this.#secret = 42; // Vraiment prive
                                            }
    def parler(self):                       parler() {
        print(f"{self.nom} bruit")              console.log(`${this.nom} bruit`);
                                            }
    @property                               get info() {
    def info(self):                             return `${this.nom}`;
        return f"{self.nom}"                }
                                        }
    @staticmethod                       // En Python : name mangling (_Animal__secret)
    def creer(nom, age):                // En JS : vraiment inaccessible (#secret)
        return Animal(nom, age)

class Chat(Animal):                     class Chat extends Animal {
    def __init__(self, nom, age, c):        constructor(nom, age, c) {
        super().__init__(nom, age)              super(nom, age);
        self.couleur = c                        this.couleur = c;
                                            }
    def parler(self):                       parler() {
        print(f"{self.nom}: Miaou!")            console.log(`${this.nom}: Miaou!`);
                                            }
                                        }
```

> [!info] Heritage multiple ?
> JavaScript ne supporte **pas** l'heritage multiple (contrairement a Python). Pour ajouter des comportements a plusieurs classes, on utilise des **mixins** (fonctions qui prennent une classe et retournent une classe etendue).

---

## Carte Mentale ASCII

```
                      JavaScript Moderne ES6+
                              │
       ┌──────────┬───────────┼───────────┬──────────────┐
       │          │           │           │              │
  Destructuring Spread/Rest Nouvelles  Collections    Classes
       │          │        Syntaxes        │              │
   Tableaux    ...array    ?.             Map           constructor
   Objets      ...object   ??             Set           extends/super
   Imbrique    ...args     && ||          WeakMap       private #
   Parametres  Copie       Enhanced obj   WeakSet       static
   Defauts     Fusion      Symbols                     get/set
       │                      │                          │
       │               ┌──────┴──────┐              Modules
       │               │             │                  │
       │          Iterateurs    Generateurs         import/export
       │          next()        function*           named/default
       │          {value,done}  yield               dynamic import()
       │                        yield*
       │
   Arrow Functions
       │
   this lexical
   Quand utiliser
   Quand eviter
   Tagged templates
```

---

## Exercices

### Exercice 1 : Gestionnaire de configuration

Creez une classe `Config` qui :
- Utilise un `Map` interne (prive avec `#`) pour stocker les paires cle-valeur
- Supporte le destructuring dans le constructeur : `new Config({ theme: "sombre", lang: "fr" })`
- Methode `get(cle, defaut)` avec optional chaining et nullish coalescing
- Methode `merge(autreConfig)` avec spread sur les entries
- Methode `toJSON()` pour la serialisation
- Est iterable (implementez `[Symbol.iterator]()`)

### Exercice 2 : Pipeline de transformations avec generateurs

Creez des generateurs pour creer un pipeline de traitement de donnees :
- `function* range(start, end, step)` : genere une sequence
- `function* map(iterable, fn)` : applique une transformation
- `function* filter(iterable, fn)` : filtre les valeurs
- `function* take(iterable, n)` : prend les n premiers
- Combinez-les : `take(filter(map(range(0, 100), x => x ** 2), x => x % 3 === 0), 5)`

### Exercice 3 : Mini-framework de composants

Creez un systeme de composants UI simple :
- Classe de base `Component` avec `render()`, `setState()`, `mount(container)`
- Classe `Button extends Component` avec des props (texte, onClick, style)
- Classe `List extends Component` qui affiche un tableau d'items
- Utilisez les modules (export/import), les proprietes privees, les destructuring dans les constructeurs
- Le `setState` doit re-render le composant automatiquement

---

## Liens

- [[01 - Introduction a JavaScript]]
- [[02 - JavaScript DOM et Evenements]]
- [[03 - JavaScript Asynchrone]]
- [[03 - POO en Python]]
