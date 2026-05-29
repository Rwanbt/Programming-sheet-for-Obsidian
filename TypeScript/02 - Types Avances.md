# Types Avances en TypeScript

> [!tip] Analogie
> Imagine que TypeScript de base, c'est une boite a outils avec marteau, tournevis, cles. Les types avances, c'est l'atelier complet : fraiseuse, tour, decoupeuse laser. Tu peux fabriquer des pieces sur mesure qui s'emboitent parfaitement, avec des garanties que le marteau seul ne peut pas offrir. Les generiques, les types conditionnels et les mapped types te permettent de **programmer sur les types eux-memes** — pas seulement de les utiliser.

> [!info] Prerequis
> Cette note suppose que tu connais deja les bases : annotations de type, interfaces, enums, unions simples (`string | number`), le compilateur `tsc`. Si ce n'est pas le cas, commence par [[TypeScript/01 - Introduction a TypeScript]].

---

## Pourquoi les types avances ?

Avec les types de base, tu peux typer des variables, des parametres, des retours de fonction. C'est deja excellent. Mais rapidement tu rencontres des situations ou tu veux exprimer des contraintes plus subtiles :

- "Cette fonction accepte n'importe quel type, mais retourne **le meme** type que ce qu'elle recoit"
- "Je veux une version partielle de cette interface, sans la reedire a la main"
- "Selon la valeur d'une propriete, l'objet a une forme differente"
- "Je veux parcourir toutes les cles d'un type et transformer leurs valeurs"

Les types de base ne peuvent pas exprimer ca. Les types avances le peuvent. Et la recompense est immense : des erreurs detectees a la compilation que tu n'aurais vues qu'en production.

```
Types de base          Types avances
─────────────────      ──────────────────────────────
string                 Template literal : `${string}Id`
number                 Conditional : T extends U ? X : Y
boolean                Mapped : { [K in keyof T]: ... }
interface fixe         Generique : interface Box<T>
union manuelle         Extract<T, U>, Exclude<T, U>
```

---

## 1. Generiques `<T>`

### Le probleme qu'ils resolvent

Sans generiques, si tu veux une fonction `identity` qui retourne son argument, tu ecris :

```typescript
function identityString(x: string): string { return x; }
function identityNumber(x: number): number { return x; }
// ... une version par type → copie de code, aucun interet
```

Ou tu utilises `any`, ce qui desactive le typage :

```typescript
function identity(x: any): any { return x; } // perd toute information de type
```

Les generiques resolvent ca proprement.

### Fonctions generiques

```typescript
function identity<T>(x: T): T {
  return x;
}

const s = identity("bonjour"); // TypeScript infere T = string, retour : string
const n = identity(42);         // T = number
const b = identity(true);       // T = boolean

// Appel explicite (rarement necessaire, l'inference suffit) :
const s2 = identity<string>("bonjour");
```

> [!info] L'inference generique
> TypeScript infere le type `T` automatiquement a partir de l'argument passe. Tu n'as presque jamais besoin d'ecrire `<string>` explicitement. L'inference est l'une des forces majeures du compilateur.

Exemple plus realiste : une fonction qui retourne le premier element d'un tableau :

```typescript
function premier<T>(tableau: T[]): T | undefined {
  return tableau[0];
}

const premierNom = premier(["Alice", "Bob", "Charlie"]); // type : string | undefined
const premierNombre = premier([10, 20, 30]);              // type : number | undefined
const vide = premier([]);                                  // type : undefined
```

### Interfaces et types generiques

```typescript
interface Boite<T> {
  contenu: T;
  etiquette: string;
  vider(): void;
  remplir(valeur: T): void;
}

// Utilisation :
const boiteATexte: Boite<string> = {
  contenu: "document important",
  etiquette: "Archives 2026",
  vider() { /* ... */ },
  remplir(valeur: string) { /* ... */ }
};

// Type alias generique :
type Paire<A, B> = {
  premier: A;
  second: B;
};

const coordonnees: Paire<number, number> = { premier: 48.8566, second: 2.3522 };
const entree: Paire<string, number> = { premier: "age", second: 30 };
```

### Classes generiques

```typescript
class Pile<T> {
  private elements: T[] = [];

  empiler(element: T): void {
    this.elements.push(element);
  }

  depiler(): T | undefined {
    return this.elements.pop();
  }

  get taille(): number {
    return this.elements.length;
  }

  estVide(): boolean {
    return this.elements.length === 0;
  }
}

const pileDeNombres = new Pile<number>();
pileDeNombres.empiler(1);
pileDeNombres.empiler(2);
pileDeNombres.empiler(3);
console.log(pileDeNombres.depiler()); // 3

const pileDeChaines = new Pile<string>();
pileDeChaines.empiler("premier");
// pileDeChaines.empiler(42); // ERREUR : Argument of type 'number' is not assignable to 'string'
```

### Contraintes avec `extends`

Parfois tu veux que `T` respecte un contrat minimal. Sans contrainte, tu ne peux appeler aucune methode sur `T` car TypeScript ne sait pas ce que c'est.

```typescript
// Sans contrainte : ERREUR
function afficherLongueur<T>(x: T): number {
  return x.length; // Erreur : Property 'length' does not exist on type 'T'
}

// Avec contrainte : OK
interface AvecLongueur {
  length: number;
}

function afficherLongueur<T extends AvecLongueur>(x: T): number {
  return x.length; // OK : T a garantiment une propriete length
}

afficherLongueur("bonjour");   // OK : string a .length
afficherLongueur([1, 2, 3]);   // OK : array a .length
afficherLongueur({ length: 5, nom: "test" }); // OK : objet avec .length
// afficherLongueur(42);       // ERREUR : number n'a pas .length
```

Contrainte avec `keyof` (tres utile) :

```typescript
function getPropriete<T, K extends keyof T>(objet: T, cle: K): T[K] {
  return objet[cle];
}

const utilisateur = { nom: "Alice", age: 30, actif: true };

const nom = getPropriete(utilisateur, "nom");   // type : string
const age = getPropriete(utilisateur, "age");   // type : number
// getPropriete(utilisateur, "email");          // ERREUR : "email" n'existe pas
```

### Plusieurs parametres de type

```typescript
function fusionner<T, U>(objet1: T, objet2: U): T & U {
  return { ...objet1, ...objet2 } as T & U;
}

const resultat = fusionner(
  { nom: "Alice" },
  { age: 30, actif: true }
);
// type infere : { nom: string } & { age: number; actif: boolean }
console.log(resultat.nom);   // "Alice"
console.log(resultat.age);   // 30
```

> [!warning] Generiques vs `any`
> `any` desactive completement la verification de type. `T` **preserve** le type : si tu passes un `string`, TypeScript sait que le retour est un `string`. C'est toute la difference. N'utilise jamais `any` la ou un generique suffit.

---

## 2. Utility Types

TypeScript inclut une bibliotheque de types utilitaires preconstruits qui transforment des types existants. Ils sont implementes en interne avec les mecanismes que tu verras ensuite (mapped types, conditional types), mais tu peux les utiliser directement.

```
Utility Types — Vue d'ensemble
─────────────────────────────────────────────────────────────────
Partial<T>         Toutes les proprietes deviennent optionnelles
Required<T>        Toutes les proprietes deviennent obligatoires
Readonly<T>        Toutes les proprietes deviennent en lecture seule
Record<K, V>       Dictionnaire avec cles de type K et valeurs V
Pick<T, K>         Garde seulement les proprietes K de T
Omit<T, K>         Retire les proprietes K de T
Exclude<T, U>      Retire les membres de T qui sont dans U (unions)
Extract<T, U>      Garde seulement les membres de T qui sont dans U
NonNullable<T>     Retire null et undefined de T
ReturnType<F>      Type de retour d'une fonction F
Parameters<F>      Tuple des parametres d'une fonction F
InstanceType<C>    Type de l'instance d'une classe C
─────────────────────────────────────────────────────────────────
```

### `Partial<T>` — tout rendre optionnel

```typescript
interface Utilisateur {
  id: number;
  nom: string;
  email: string;
  age: number;
}

// Tres utile pour les fonctions de mise a jour (PATCH) :
function mettreAJourUtilisateur(id: number, modifications: Partial<Utilisateur>): void {
  // modifications peut n'avoir que certaines proprietes
  console.log(`Mise a jour ${id}:`, modifications);
}

mettreAJourUtilisateur(1, { nom: "Alice" });          // OK
mettreAJourUtilisateur(2, { email: "x@y.com", age: 25 }); // OK
mettreAJourUtilisateur(3, {});                         // OK : tout est optionnel
```

### `Required<T>` — tout rendre obligatoire

```typescript
interface ConfigOptionnelle {
  timeout?: number;
  retries?: number;
  baseUrl?: string;
}

// Apres validation, toutes les valeurs sont presentes :
type ConfigComplete = Required<ConfigOptionnelle>;
// Equivalent a : { timeout: number; retries: number; baseUrl: string }

function demarrerClient(config: ConfigComplete): void {
  // Ici on sait que tout est defini, pas besoin de verifier undefined
  console.log(`Connecting to ${config.baseUrl} with timeout ${config.timeout}ms`);
}
```

### `Readonly<T>` — immutabilite

```typescript
interface Point {
  x: number;
  y: number;
}

const point: Readonly<Point> = { x: 10, y: 20 };
// point.x = 5; // ERREUR : Cannot assign to 'x' because it is a read-only property

// Tres utile pour les props React, les constantes de configuration :
const CONFIG: Readonly<{ apiUrl: string; version: string }> = {
  apiUrl: "https://api.example.com",
  version: "2.0"
};
```

### `Record<K, V>` — dictionnaire type

```typescript
type Role = "admin" | "editor" | "viewer";

const permissions: Record<Role, string[]> = {
  admin: ["read", "write", "delete"],
  editor: ["read", "write"],
  viewer: ["read"]
};

// Dictionnaire avec cles dynamiques :
type CacheNombres = Record<string, number>;
const cache: CacheNombres = {};
cache["resultat_42"] = 1764;
```

> [!example] Record vs Index Signature
> `Record<string, number>` et `{ [key: string]: number }` sont equivalents. Mais `Record<"a" | "b", number>` garantit que les cles `"a"` et `"b"` **sont presentes** — ce que l'index signature ne peut pas exprimer.

### `Pick<T, K>` et `Omit<T, K>` — selection et exclusion

```typescript
interface Article {
  id: number;
  titre: string;
  contenu: string;
  auteur: string;
  dateCreation: Date;
  dateModification: Date;
  publie: boolean;
}

// Pour une liste d'articles (on n'a pas besoin du contenu complet) :
type ArticleResume = Pick<Article, "id" | "titre" | "auteur" | "publie">;
// { id: number; titre: string; auteur: string; publie: boolean }

// Pour la creation (pas d'id ni de dates encore) :
type NouvelArticle = Omit<Article, "id" | "dateCreation" | "dateModification">;
// { titre: string; contenu: string; auteur: string; publie: boolean }
```

> [!tip] Pick vs Omit
> Utilise `Pick` quand tu veux **quelques proprietes** d'un grand type.
> Utilise `Omit` quand tu veux **presque tout** sauf quelques proprietes.
> Le code est plus lisible ainsi.

### `Exclude<T, U>` et `Extract<T, U>` — operations sur les unions

Ces deux types operent sur des **unions**, pas sur des objets.

```typescript
type Evenement = "click" | "mouseenter" | "mouseleave" | "keydown" | "keyup" | "submit";

// Exclure tous les evenements souris :
type EvenementsClavier = Exclude<Evenement, "click" | "mouseenter" | "mouseleave">;
// "keydown" | "keyup" | "submit"

// Garder seulement les evenements souris :
type EvenementsSouris = Extract<Evenement, "click" | "mouseenter" | "mouseleave">;
// "click" | "mouseenter" | "mouseleave"

// Cas classique : retirer null/undefined d'un type nullable
type MaybeString = string | null | undefined;
type CertainementString = Exclude<MaybeString, null | undefined>;
// string
```

### `NonNullable<T>`

```typescript
type MaybeNombre = number | null | undefined;
type NombreCertain = NonNullable<MaybeNombre>; // number

// Equivalent a Exclude<T, null | undefined> — juste plus lisible
```

### `ReturnType<F>` et `Parameters<F>`

Ces deux types extraient des informations depuis une signature de fonction :

```typescript
function creerUtilisateur(nom: string, age: number, admin: boolean) {
  return { id: Math.random(), nom, age, admin, creeA: new Date() };
}

// Le type de retour, sans avoir a le redeclarer :
type Utilisateur = ReturnType<typeof creerUtilisateur>;
// { id: number; nom: string; age: number; admin: boolean; creeA: Date }

// Les parametres sous forme de tuple :
type ParamsCreation = Parameters<typeof creerUtilisateur>;
// [nom: string, age: number, admin: boolean]

// Utile pour les wrappers :
function journaliserAppel<F extends (...args: any[]) => any>(
  fn: F,
  ...args: Parameters<F>
): ReturnType<F> {
  console.log("Appel avec :", args);
  const resultat = fn(...args);
  console.log("Resultat :", resultat);
  return resultat;
}

const utilisateur = journaliserAppel(creerUtilisateur, "Alice", 30, false);
```

---

## 3. `keyof` et `typeof`

### `keyof` — extraire les cles d'un type

`keyof T` produit une union des noms de proprietes de `T`.

```typescript
interface Produit {
  id: number;
  nom: string;
  prix: number;
  disponible: boolean;
}

type ClesProduit = keyof Produit;
// "id" | "nom" | "prix" | "disponible"

// Utilisation pratique : acces typesafe a une propriete
function trier<T>(tableau: T[], cle: keyof T): T[] {
  return [...tableau].sort((a, b) => {
    if (a[cle] < b[cle]) return -1;
    if (a[cle] > b[cle]) return 1;
    return 0;
  });
}

const produits = [
  { id: 3, nom: "Clavier", prix: 89, disponible: true },
  { id: 1, nom: "Souris", prix: 29, disponible: false },
  { id: 2, nom: "Ecran", prix: 299, disponible: true }
];

trier(produits, "prix");  // OK : trie par prix
trier(produits, "nom");   // OK : trie par nom
// trier(produits, "couleur"); // ERREUR : "couleur" n'est pas une cle de Produit
```

### `typeof` — extraire le type d'une valeur

`typeof` en TypeScript (pas le `typeof` JavaScript) extrait le type statique d'une variable ou d'une expression.

```typescript
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  features: {
    darkMode: true,
    notifications: false
  }
};

// Extraire le type de config sans le redeclarer :
type Config = typeof config;
// {
//   apiUrl: string;
//   timeout: number;
//   features: { darkMode: boolean; notifications: boolean };
// }

// Tres utile avec des fonctions :
const operations = {
  addition: (a: number, b: number) => a + b,
  soustraction: (a: number, b: number) => a - b,
  multiplication: (a: number, b: number) => a * b
};

type TypeOperations = typeof operations;
type NomOperation = keyof typeof operations; // "addition" | "soustraction" | "multiplication"
```

> [!info] `keyof typeof` — le duo classique
> `keyof typeof monObjet` est un pattern tres frequent. Il te donne les cles d'un objet **valeur** (pas d'un type) sous forme d'union de string literals. Indispensable pour les lookups typsafe dans des maps de configuration ou de traduction.

---

## 4. Index Signatures

Une index signature permet de dire "cet objet peut avoir n'importe quelle cle de ce type, et les valeurs seront de ce type".

```typescript
interface Dictionnaire {
  [cle: string]: string;
}

const traductions: Dictionnaire = {
  bonjour: "hello",
  merci: "thank you",
  aurevoir: "goodbye"
};

// Cles dynamiques
traductions["bonsoir"] = "good evening"; // OK
// traductions["nombre"] = 42;           // ERREUR : la valeur doit etre string

// Index signatures avec types de cle :
interface CacheNumerique {
  [id: number]: { valeur: string; timestamp: number };
}

// Melanger proprietes fixes et index signature :
interface CompteRendu {
  titre: string;       // propriete fixe obligatoire
  date: Date;          // propriete fixe obligatoire
  [section: string]: string | Date; // sections dynamiques
  // Note : le type des valeurs fixes doit etre compatible avec l'index signature
}
```

> [!warning] Piege avec index signatures et proprietes fixes
> Si tu as un index signature `[key: string]: SomeType`, toutes les proprietes fixes doivent avoir un type compatible avec `SomeType`. C'est une contrainte de TypeScript : l'index signature "couvre" toutes les cles possibles, y compris les proprietes nommees.

---

## 5. Types Conditionnels

La syntaxe est : `T extends U ? X : Y`

Cela se lit : "Si `T` est assignable a `U`, alors le type est `X`, sinon le type est `Y`."

### Cas basiques

```typescript
type EstChaine<T> = T extends string ? true : false;

type A = EstChaine<string>;  // true
type B = EstChaine<number>;  // false
type C = EstChaine<"hello">; // true (literal string extends string)

// Cas pratique : rendre un type nullable
type Nullable<T> = T extends null | undefined ? T : T | null;
```

### Distribution sur les unions

Quand `T` est une union, le type conditionnel se **distribue** sur chaque membre :

```typescript
type EstObjet<T> = T extends object ? true : false;

type D = EstObjet<string | number | object>;
// Distribue : EstObjet<string> | EstObjet<number> | EstObjet<object>
// = false | false | true
// = boolean (simplifie)

// Exemple tres utile :
type Flatten<T> = T extends Array<infer Item> ? Item : T;

type E = Flatten<string[]>;      // string
type F = Flatten<number[][]>;    // number[] (un seul niveau)
type G = Flatten<boolean>;       // boolean (pas un tableau)
```

### `infer` — extraire un sous-type

`infer` introduit une variable de type dans la branche `extends`. C'est l'outil le plus puissant des types conditionnels.

```typescript
// Extraire le type de retour d'une fonction :
type MonReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type H = MonReturnType<() => string>;           // string
type I = MonReturnType<(x: number) => boolean>; // boolean
type J = MonReturnType<string>;                 // never (pas une fonction)

// Extraire le type des elements d'un tableau :
type ElementDe<T> = T extends (infer E)[] ? E : never;

type K = ElementDe<string[]>;          // string
type L = ElementDe<Array<{ id: number }>>; // { id: number }

// Extraire le type du premier parametre :
type PremierParam<T> = T extends (premier: infer P, ...reste: any[]) => any ? P : never;

type M = PremierParam<(id: number, nom: string) => void>; // number
```

> [!tip] `infer` en pratique
> `infer` est utilise en interne par `ReturnType<F>`, `Parameters<F>`, `InstanceType<C>`. Si tu comprends les exemples ci-dessus, tu comprends comment ces utility types sont implementes. La bibliotheque standard TypeScript est entierement ecrite avec ces mecanismes.

### Types conditionnels imbriques

```typescript
type TypeDeValeur<T> =
  T extends string ? "chaine" :
  T extends number ? "nombre" :
  T extends boolean ? "booleen" :
  T extends null ? "null" :
  T extends undefined ? "indefini" :
  T extends Function ? "fonction" :
  "objet";

type N = TypeDeValeur<string>;    // "chaine"
type O = TypeDeValeur<42>;        // "nombre"
type P = TypeDeValeur<() => void>; // "fonction"
type Q = TypeDeValeur<{ x: number }>; // "objet"
```

---

## 6. Mapped Types

Les mapped types permettent de **transformer** tous les membres d'un type existant. La syntaxe de base :

```typescript
type TransformerType<T> = {
  [K in keyof T]: /* nouveau type pour chaque propriete */
};
```

### Transformation basique

```typescript
interface Personne {
  nom: string;
  age: number;
  actif: boolean;
}

// Rendre toutes les proprietes optionnelles (implementation de Partial) :
type MonPartial<T> = {
  [K in keyof T]?: T[K];
};

// Rendre toutes les proprietes readonly (implementation de Readonly) :
type MonReadonly<T> = {
  readonly [K in keyof T]: T[K];
};

// Rendre toutes les proprietes nullable :
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

type PersonneNullable = Nullable<Personne>;
// { nom: string | null; age: number | null; actif: boolean | null }
```

### Modificateurs `+` et `-`

Les modificateurs `+` et `-` ajoutent ou retirent `readonly` et `?` :

```typescript
// Retirer readonly (inverse de Readonly) :
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};

// Retirer l'optionnalite (implementation de Required) :
type MonRequired<T> = {
  [K in keyof T]-?: T[K];
};

const personne: Readonly<Personne> = { nom: "Alice", age: 30, actif: true };
// personne.nom = "Bob"; // ERREUR

const personneModifiable: Mutable<typeof personne> = { ...personne };
personneModifiable.nom = "Bob"; // OK maintenant
```

### Mapped types avec conditions

On peut combiner mapped types et types conditionnels :

```typescript
// Garder seulement les proprietes d'un certain type :
type FiltrerParType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface Formulaire {
  nom: string;
  age: number;
  email: string;
  score: number;
  actif: boolean;
}

type ChampsTexte = FiltrerParType<Formulaire, string>;
// { nom: string; email: string }

type ChampsNumeriques = FiltrerParType<Formulaire, number>;
// { age: number; score: number }
```

### Remapping de cles avec `as`

```typescript
// Ajouter un prefixe a toutes les cles :
type AvecPrefixe<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K];
};

type PersonneAvecGet = AvecPrefixe<Personne, "get">;
// { getNom: string; getAge: number; getActif: boolean }

// Renommer les cles en camelCase → snake_case (exemple simplifie) :
type VersSnakeCase<T> = {
  [K in keyof T as K extends string ? `${Lowercase<K>}` : K]: T[K];
};
```

---

## 7. Template Literal Types

Les template literal types etendent les string literals avec une syntaxe similaire aux template strings JavaScript.

```typescript
type Evenement = "click" | "focus" | "blur";
type Ecouteur = `on${Capitalize<Evenement>}`;
// "onClick" | "onFocus" | "onBlur"

type Direction = "top" | "right" | "bottom" | "left";
type Padding = `padding-${Direction}`;
// "padding-top" | "padding-right" | "padding-bottom" | "padding-left"
```

### Combinaisons d'unions

Quand plusieurs unions sont combinees, TypeScript produit le produit cartesien :

```typescript
type Couleur = "rouge" | "vert" | "bleu";
type Taille = "petit" | "grand";

type Variante = `${Couleur}-${Taille}`;
// "rouge-petit" | "rouge-grand" | "vert-petit" | "vert-grand" | "bleu-petit" | "bleu-grand"
```

### Utilitaires de manipulation de chaines

TypeScript fournit des types utilitaires pour manipuler les chaines :

```typescript
type R = Uppercase<"bonjour">;    // "BONJOUR"
type S = Lowercase<"BONJOUR">;    // "bonjour"
type T2 = Capitalize<"bonjour">;   // "Bonjour"
type U2 = Uncapitalize<"Bonjour">; // "bonjour"
```

### Application pratique : typed event emitter

```typescript
type EventMap = {
  userConnected: { userId: string; timestamp: Date };
  messageSent: { from: string; to: string; content: string };
  fileUploaded: { filename: string; size: number };
};

type EventName = keyof EventMap;
type HandlerName = `on${Capitalize<EventName>}`;
// "onUserConnected" | "onMessageSent" | "onFileUploaded"

type Handlers = {
  [K in EventName as `on${Capitalize<K>}`]: (payload: EventMap[K]) => void;
};
// {
//   onUserConnected: (payload: { userId: string; timestamp: Date }) => void;
//   onMessageSent: (payload: { from: string; ... }) => void;
//   onFileUploaded: (payload: { filename: string; size: number }) => void;
// }
```

> [!example] Template literal types + mapped types = API generation
> Ce pattern est utilise dans des librairies comme `tRPC`, `Prisma`, et `Zod` pour generer automatiquement des types d'API a partir de schemas. C'est ce qui leur permet de te proposer une autocompletion precise sans ecrire un seul type manuellement.

---

## 8. Type Guards

### Le probleme

TypeScript sait qu'une valeur peut etre `string | number`. Mais dans une branche specifique du code, tu veux travailler avec la certitude que c'est un `string`. C'est le role des type guards : **affiner le type dans un bloc de code**.

### `typeof` guard

```typescript
function traiter(valeur: string | number | boolean): string {
  if (typeof valeur === "string") {
    // Ici TypeScript sait que valeur : string
    return valeur.toUpperCase();
  }
  if (typeof valeur === "number") {
    // Ici TypeScript sait que valeur : number
    return valeur.toFixed(2);
  }
  // Ici TypeScript sait que valeur : boolean
  return valeur ? "vrai" : "faux";
}
```

### `instanceof` guard

```typescript
class Cercle {
  constructor(public rayon: number) {}
  aire(): number { return Math.PI * this.rayon ** 2; }
}

class Rectangle {
  constructor(public largeur: number, public hauteur: number) {}
  aire(): number { return this.largeur * this.hauteur; }
}

type Forme = Cercle | Rectangle;

function calculerAire(forme: Forme): number {
  if (forme instanceof Cercle) {
    // TypeScript sait que forme : Cercle
    return forme.aire(); // acces a rayon possible aussi
  }
  // TypeScript sait que forme : Rectangle
  return forme.aire();
}
```

### `in` guard

```typescript
interface Chien {
  type: "chien";
  aboyer(): void;
  manger(): void;
}

interface Chat {
  type: "chat";
  miauler(): void;
  manger(): void;
}

function faireParler(animal: Chien | Chat): void {
  if ("aboyer" in animal) {
    // TypeScript sait que animal : Chien
    animal.aboyer();
  } else {
    // TypeScript sait que animal : Chat
    animal.miauler();
  }
}
```

### Fonctions de type guard `x is T`

Quand `typeof`, `instanceof` et `in` ne suffisent pas, tu peux ecrire ta propre fonction de type guard :

```typescript
interface UtilisateurConnecte {
  id: string;
  nom: string;
  token: string;
}

interface UtilisateurAnonyme {
  sessionId: string;
}

type Visiteur = UtilisateurConnecte | UtilisateurAnonyme;

// La syntaxe "parametre is Type" dans le retour fait de cette fonction un type guard
function estConnecte(visiteur: Visiteur): visiteur is UtilisateurConnecte {
  return "token" in visiteur;
}

function afficherProfil(visiteur: Visiteur): void {
  if (estConnecte(visiteur)) {
    // TypeScript sait que visiteur : UtilisateurConnecte
    console.log(`Bonjour ${visiteur.nom}, token: ${visiteur.token}`);
  } else {
    // TypeScript sait que visiteur : UtilisateurAnonyme
    console.log(`Session anonyme: ${visiteur.sessionId}`);
  }
}
```

> [!warning] Type guards personnalises : responsabilite du developpeur
> TypeScript fait confiance a ton predicat. Si tu ecris `function estString(x: unknown): x is string { return true; }`, TypeScript te croira meme si tu mens. La responsabilite de l'exactitude revient au developpeur. C'est le seul endroit en TypeScript ou le type checking peut "mentir".

### `asserts x is T` — type assertion functions

```typescript
function assertEstNombre(valeur: unknown): asserts valeur is number {
  if (typeof valeur !== "number") {
    throw new Error(`Attendu un nombre, recu : ${typeof valeur}`);
  }
}

function traiterValeur(x: unknown): void {
  assertEstNombre(x);
  // Ici TypeScript sait que x : number
  console.log(x.toFixed(2));
}
```

---

## 9. Discriminated Unions

C'est l'un des patterns les plus utilises en TypeScript, particulierement dans les applications React, Redux et les systemes a etats.

### Principe

Une discriminated union (ou tagged union) est une union de types qui partagent une **propriete discriminante** — une propriete litterale qui identifie uniquement chaque membre.

```
Requete HTTP avec union discriminee :

  ┌──────────────────────────────────────────────────┐
  │  type EtatRequete =                              │
  │    | { etat: "en_attente" }                      │
  │    | { etat: "succes"; donnees: Data }           │
  │    | { etat: "erreur"; message: string }         │
  └──────────────────────────────────────────────────┘
         ↑
    La propriete "etat" est le discriminant
    TypeScript l'utilise pour affiner le type
    dans chaque branche du switch/if
```

### Exemple complet

```typescript
// Definition de la union discriminee :
type ResultatRequete<T> =
  | { statut: "en_attente" }
  | { statut: "succes"; donnees: T; codeHttp: number }
  | { statut: "erreur"; message: string; codeHttp: number; peutReessayer: boolean };

// Utilisation :
function afficherResultat<T>(resultat: ResultatRequete<T>): string {
  switch (resultat.statut) {
    case "en_attente":
      // TypeScript sait que resultat : { statut: "en_attente" }
      return "Chargement...";

    case "succes":
      // TypeScript sait que resultat : { statut: "succes"; donnees: T; codeHttp: number }
      return `Succes (${resultat.codeHttp}): ${JSON.stringify(resultat.donnees)}`;

    case "erreur":
      // TypeScript sait que resultat.peutReessayer existe ici
      const action = resultat.peutReessayer ? "Reessayer" : "Abandonner";
      return `Erreur ${resultat.codeHttp}: ${resultat.message}. ${action}`;
  }
}
```

### Exhaustivite avec `never`

Un des avantages majeurs : TypeScript peut verifier que tu as traite **tous les cas** :

```typescript
type Forme =
  | { type: "cercle"; rayon: number }
  | { type: "rectangle"; largeur: number; hauteur: number }
  | { type: "triangle"; base: number; hauteur: number };

function calculerAire(forme: Forme): number {
  switch (forme.type) {
    case "cercle":
      return Math.PI * forme.rayon ** 2;
    case "rectangle":
      return forme.largeur * forme.hauteur;
    case "triangle":
      return (forme.base * forme.hauteur) / 2;
    default:
      // Cette branche est atteinte si un cas n'est pas traite.
      // Si on ajoute "losange" a Forme sans l'ajouter ici,
      // TypeScript va lever une erreur via le type never :
      const exhaustif: never = forme;
      throw new Error(`Forme inconnue : ${JSON.stringify(exhaustif)}`);
  }
}
```

> [!tip] Le pattern `never` pour l'exhaustivite
> En assignant `forme` a une variable de type `never` dans le `default`, tu demandes a TypeScript de verifier que cette branche est **inaccessible**. Si tu ajoutes un nouveau membre a l'union sans mettre a jour le switch, TypeScript generera une erreur a la compilation. C'est une technique de protection contre les oublis dans les unions qui evoluent.

### Application Redux-style

```typescript
// Actions typees avec unions discriminees :
type ActionCompteur =
  | { type: "INCREMENTER" }
  | { type: "DECREMENTER" }
  | { type: "REINITIALISER" }
  | { type: "DEFINIR"; valeur: number };

interface EtatCompteur {
  valeur: number;
}

function reducerCompteur(etat: EtatCompteur, action: ActionCompteur): EtatCompteur {
  switch (action.type) {
    case "INCREMENTER":
      return { valeur: etat.valeur + 1 };
    case "DECREMENTER":
      return { valeur: etat.valeur - 1 };
    case "REINITIALISER":
      return { valeur: 0 };
    case "DEFINIR":
      // action.valeur est accessible uniquement ici
      return { valeur: action.valeur };
  }
}
```

---

## 10. `as const` et Readonly Profond

### `as const` — inferrer les types les plus precis

Sans `as const`, TypeScript infere des types larges pour les litteraux :

```typescript
const config = {
  url: "https://api.example.com",
  port: 3000,
  features: ["darkMode", "notifications"]
};
// TypeScript infere :
// { url: string; port: number; features: string[] }
// Les valeurs peuvent "changer" selon TypeScript

const configConst = {
  url: "https://api.example.com",
  port: 3000,
  features: ["darkMode", "notifications"]
} as const;
// TypeScript infere :
// {
//   readonly url: "https://api.example.com";
//   readonly port: 3000;  ← literal type, pas juste number
//   readonly features: readonly ["darkMode", "notifications"];
// }
```

### Utilisation pratique avec les enums

`as const` est souvent prefere aux enums en TypeScript moderne :

```typescript
// Avec enum :
enum Direction {
  Nord = "NORD",
  Sud = "SUD",
  Est = "EST",
  Ouest = "OUEST"
}

// Avec as const (prefere) :
const Direction = {
  Nord: "NORD",
  Sud: "SUD",
  Est: "EST",
  Ouest: "OUEST"
} as const;

// Extraire le type union des valeurs :
type Direction = typeof Direction[keyof typeof Direction];
// "NORD" | "SUD" | "EST" | "OUEST"

function deplacer(direction: Direction): void {
  console.log(`Deplacement vers : ${direction}`);
}

deplacer(Direction.Nord); // OK
deplacer("NORD");         // OK (la valeur litterale est compatible)
// deplacer("nord");      // ERREUR : "nord" n'est pas dans la union
```

### `readonly` profond avec `DeepReadonly`

Le type `Readonly<T>` de TypeScript est seulement en surface (shallow). Pour un readonly profond, il faut l'implementer :

```typescript
type DeepReadonly<T> =
  T extends (infer R)[] ? DeepReadonlyArray<R> :
  T extends object ? DeepReadonlyObject<T> :
  T;

interface DeepReadonlyArray<T> extends ReadonlyArray<DeepReadonly<T>> {}

type DeepReadonlyObject<T> = {
  readonly [K in keyof T]: DeepReadonly<T[K]>;
};

// Exemple :
interface StructureImbriquee {
  niveau1: {
    niveau2: {
      valeur: number;
    };
  };
}

const obj: DeepReadonly<StructureImbriquee> = {
  niveau1: {
    niveau2: { valeur: 42 }
  }
};

// obj.niveau1.niveau2.valeur = 100; // ERREUR a tous les niveaux
```

---

## 11. Declaration Merging

TypeScript permet de fusionner plusieurs declarations du meme nom pour enrichir un type existant. C'est principalement utilise pour l'augmentation de types de librairies tierces.

### Fusion d'interfaces

```typescript
// Premiere declaration
interface Requete {
  url: string;
  methode: "GET" | "POST" | "PUT" | "DELETE";
}

// Deuxieme declaration — fusionne avec la premiere
interface Requete {
  headers?: Record<string, string>;
  body?: unknown;
}

// Resultat : les deux sont fusionnees
const requete: Requete = {
  url: "/api/users",
  methode: "GET",
  headers: { "Authorization": "Bearer token" }
  // body est optionnel
};
```

### Augmentation de module

C'est le cas d'usage le plus important. Par exemple, pour ajouter des proprietes a `Express.Request` :

```typescript
// Dans un fichier express.d.ts de ton projet :
declare global {
  namespace Express {
    interface Request {
      utilisateur?: {
        id: string;
        nom: string;
        roles: string[];
      };
    }
  }
}

// Maintenant dans tes middlewares :
// req.utilisateur est type correctement partout
```

### Fusion de namespaces

```typescript
// Utile pour ajouter des helpers statiques a une classe :
class Validateur {
  private valeur: string;
  constructor(valeur: string) { this.valeur = valeur; }
  estValide(): boolean { return this.valeur.length > 0; }
}

namespace Validateur {
  export function email(v: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v);
  }
  export function telephone(v: string): boolean {
    return /^\+?[0-9]{10,15}$/.test(v);
  }
}

// Utilisation :
const v = new Validateur("test");
v.estValide(); // methode instance
Validateur.email("test@example.com"); // methode statique du namespace
```

> [!warning] Declaration merging et interfaces vs types
> Le merging ne fonctionne qu'avec les `interface`, pas avec les `type`. Si tu declares deux `type` avec le meme nom, TypeScript genere une erreur. C'est l'une des differences pratiques entre `interface` et `type`.

---

## 12. Tableau Recapitulatif des Utility Types

| Type | Syntax | Usage principal |
|------|--------|-----------------|
| `Partial<T>` | `{ [K in keyof T]?: T[K] }` | Mises a jour partielles (PATCH) |
| `Required<T>` | `{ [K in keyof T]-?: T[K] }` | Apres validation, tout est present |
| `Readonly<T>` | `{ readonly [K in keyof T]: T[K] }` | Immutabilite, props React |
| `Record<K,V>` | `{ [P in K]: V }` | Dictionnaires, lookups |
| `Pick<T,K>` | `{ [P in K]: T[P] }` | Sous-ensemble de type |
| `Omit<T,K>` | `Pick<T, Exclude<keyof T, K>>` | Exclure quelques proprietes |
| `Exclude<T,U>` | Types conditionnels | Operations sur unions |
| `Extract<T,U>` | Types conditionnels | Intersection d'unions |
| `NonNullable<T>` | `Exclude<T, null \| undefined>` | Retirer les nullables |
| `ReturnType<F>` | Types conditionnels + infer | Type de retour d'une fonction |
| `Parameters<F>` | Types conditionnels + infer | Parametres d'une fonction |

---

## 13. Recapitulatif des Type Guards

| Guard | Syntax | Quand utiliser |
|-------|--------|----------------|
| `typeof` | `typeof x === "string"` | Primitifs (string, number, boolean...) |
| `instanceof` | `x instanceof Classe` | Classes et constructeurs |
| `in` | `"prop" in objet` | Verifier l'existence d'une propriete |
| Predicat | `x is Type` dans le retour | Logique de validation personnalisee |
| `asserts` | `asserts x is Type` | Validation avec throw si echec |
| Discriminant | Switch sur propriete litterale | Unions discriminees |

---

## Carte Mentale

```
                    TYPES AVANCES TYPESCRIPT
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   GENERIQUES         TRANSFORMATIONS          AFFINAGE
        │                    │                    │
   ┌────┴────┐         ┌─────┴─────┐        ┌────┴────┐
   │         │         │           │        │         │
<T>       extends  Utility     Mapped    Discrimin. Type
                   Types       Types     Unions    Guards
                     │           │
           ┌─────────┤    ┌──────┴──────┐
           │         │    │             │
        Partial    Pick  [K in        typeof
        Required   Omit  keyof T]     typeof
        Readonly  Record              instanceof
        ReturnType                    "prop" in
        Parameters                    x is T
                     │
               ┌─────┴─────┐
               │           │
          Conditional  Template
          Types        Literal
               │           │
          T ext U?   `${A}-${B}`
          X : Y      Capitalize
               │
             infer
             (extraire sous-types)

    OPERATEURS CLES
    ───────────────
    keyof T        → union des cles
    typeof valeur  → type statique
    T[K]           → type de la valeur a la cle K
    as const       → types litteraux precis + readonly
    infer R        → capture de sous-type
```

---

## Exercices Pratiques

### Exercice 1 — Generiques et contraintes

Ecris une fonction `fusionnerObjets<T, U>` qui :
- Prend deux objets de types quelconques
- Retourne leur fusion (`T & U`)
- Garantit que les deux arguments sont bien des objets (pas des primitifs)
- Teste-la avec plusieurs types differents

**Indice** : utilise `extends object` comme contrainte.

```typescript
// Solution :
function fusionnerObjets<T extends object, U extends object>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 } as T & U;
}

const resultat1 = fusionnerObjets({ nom: "Alice" }, { age: 30 });
// Type infere : { nom: string } & { age: number }

const resultat2 = fusionnerObjets(
  { x: 10, y: 20 },
  { label: "point", visible: true }
);
// resultat2.x, resultat2.label : tous accessibles et types

// fusionnerObjets("chaine", { a: 1 }); // ERREUR : string n'est pas un objet
// fusionnerObjets(42, { a: 1 });        // ERREUR : number n'est pas un objet
```

---

### Exercice 2 — Utility types enchainees

Tu as cette interface de schema de formulaire :

```typescript
interface SchemaFormulaire {
  nomUtilisateur: string;
  email: string;
  motDePasse: string;
  age: number;
  accepteCGU: boolean;
  codeParrainage: string | null;
}
```

Sans reedire de types en dur, construis les types suivants :
1. `FormulaireInscription` — tout est obligatoire sauf `codeParrainage` et `age`
2. `FormulaireConnexion` — seulement `email` et `motDePasse`
3. `ProfilPublic` — tout sauf `motDePasse`, `accepteCGU`, `codeParrainage`
4. `MiseAJourProfil` — comme `ProfilPublic` mais tout optionnel

```typescript
// Solution :
type FormulaireInscription = Omit<SchemaFormulaire, "codeParrainage" | "age"> & {
  codeParrainage?: string;
  age?: number;
};

type FormulaireConnexion = Pick<SchemaFormulaire, "email" | "motDePasse">;

type ProfilPublic = Omit<SchemaFormulaire, "motDePasse" | "accepteCGU" | "codeParrainage">;

type MiseAJourProfil = Partial<ProfilPublic>;
```

---

### Exercice 3 — Discriminated union avec exhaustivite

Modelise un systeme de notification avec les types suivants :
- `Email` : destinataire, sujet, corps
- `SMS` : numero, message (max 160 chars en prod — juste a documenter)
- `PushNotification` : deviceToken, titre, corps, payload optionnel

Ecris une fonction `envoyerNotification` qui traite chaque cas et verifie l'exhaustivite avec `never`. Ajoute ensuite un 4e type `Webhook` et verifie que TypeScript te force a traiter ce nouveau cas.

```typescript
// Solution partielle (ajouter Webhook est l'exercice) :
type Notification =
  | { canal: "email"; destinataire: string; sujet: string; corps: string }
  | { canal: "sms"; numero: string; message: string }
  | { canal: "push"; deviceToken: string; titre: string; corps: string; payload?: object };

function envoyerNotification(notif: Notification): Promise<void> {
  switch (notif.canal) {
    case "email":
      console.log(`Email a ${notif.destinataire}: ${notif.sujet}`);
      return Promise.resolve();
    case "sms":
      console.log(`SMS au ${notif.numero}: ${notif.message}`);
      return Promise.resolve();
    case "push":
      console.log(`Push a ${notif.deviceToken}: ${notif.titre}`);
      return Promise.resolve();
    default:
      const jamaisAtteint: never = notif;
      throw new Error(`Canal inconnu : ${JSON.stringify(jamaisAtteint)}`);
  }
}

// Pour tester l'exhaustivite : ajouter
// | { canal: "webhook"; url: string; payload: object }
// TypeScript va signaler une erreur dans le default ci-dessus
```

---

### Exercice 4 — Mapped type et template literal

Ecris un type `AvecGetters<T>` qui, pour un objet donne, genere automatiquement une version avec des methodes getter pour chaque propriete.

Par exemple, pour `{ nom: string; age: number }`, le type resultant doit etre :
`{ getNom: () => string; getAge: () => number }`

Puis ecris un type `AvecSetters<T>` qui genere des setters : `setNom: (valeur: string) => void`, etc.

```typescript
// Solution :
type AvecGetters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type AvecSetters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (valeur: T[K]) => void;
};

type AvecAccesseurs<T> = AvecGetters<T> & AvecSetters<T>;

// Test :
interface Personne {
  nom: string;
  age: number;
  actif: boolean;
}

type AccesseursPersonne = AvecAccesseurs<Personne>;
// {
//   getNom: () => string;
//   getAge: () => number;
//   getActif: () => boolean;
//   setNom: (valeur: string) => void;
//   setAge: (valeur: number) => void;
//   setActif: (valeur: boolean) => void;
// }

// Implementation (a titre d'exemple) :
function creerAccesseurs<T extends object>(objet: T): AvecAccesseurs<T> {
  const accesseurs = {} as AvecAccesseurs<T>;
  for (const cle of Object.keys(objet) as (keyof T)[]) {
    const cleCapitalisee = (cle as string)[0].toUpperCase() + (cle as string).slice(1);
    (accesseurs as any)[`get${cleCapitalisee}`] = () => objet[cle];
    (accesseurs as any)[`set${cleCapitalisee}`] = (v: T[typeof cle]) => { objet[cle] = v; };
  }
  return accesseurs;
}
```

---

## Pour aller plus loin

Ces mecanismes ouvrent la porte a des patterns tres avances utilises dans les grandes librairies :

| Concept | Librairie qui l'utilise | Pourquoi |
|---------|------------------------|---------|
| Generiques + contraintes | `Zod`, `Prisma` | Inference automatique des types de schema |
| Mapped types | `immer`, `Redux Toolkit` | Types de draft, types d'action |
| Template literal types | `tRPC`, `Hono` | Generation de routes typees |
| Discriminated unions | `xstate`, `Redux` | Machines a etats, reducers |
| `infer` | `ts-toolbelt` | Manipulation avancee de types |
| `as const` | Config, i18n | Lookup tables typesafes |

La prochaine etape naturelle est d'appliquer ces types dans un contexte framework. Voir [[TypeScript/03 - TypeScript avec React]] pour la mise en pratique avec React et les hooks types.

---

*Note creee le 2026-05-29 — Vault [[TypeScript/01 - Introduction a TypeScript]] → cette note → [[TypeScript/03 - TypeScript avec React]]*
