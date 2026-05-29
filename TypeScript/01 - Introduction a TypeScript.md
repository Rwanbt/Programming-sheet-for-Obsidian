# Introduction a TypeScript

> [!tip] Analogie
> Imagine que tu ecris un roman. En JavaScript, tu peux mettre n'importe quoi dans les variables — comme un carnet de notes sans regles. En TypeScript, tu ajoutes un contrat : "cette variable est un titre de chapitre, elle sera toujours une chaine de caracteres". Si tu essaies d'y coller un nombre ou `undefined`, l'editeur te crie dessus **avant meme que tu sauvegardes**. TypeScript, c'est JavaScript avec un contrat de qualite — et ce contrat est verifie a la compilation, pas au runtime.

---

## Qu'est-ce que TypeScript ?

TypeScript est un **superset de JavaScript** developpe par Microsoft depuis 2012 et rendu open-source. La phrase "superset" signifie une chose simple mais fondamentale :

```
Tout code JavaScript valide est aussi du code TypeScript valide.
```

Tu n'as pas a tout reecrire. Tu peux migrer progressivement, fichier par fichier.

La grande difference : TypeScript ajoute un **systeme de types statiques**. En JavaScript, les types sont evalues au moment de l'execution (runtime). En TypeScript, ils sont verifies au moment de la **compilation** (avant que le code tourne).

### Le cycle de vie du code TypeScript

```
Ton fichier .ts
      |
      v
  Compilateur tsc
  (verifie les types)
      |
      |--- Erreur de type ? --> Message d'erreur clair, compilation stoppee
      |
      v
  Fichier .js genere
  (TypeScript "efface" les types)
      |
      v
  Node.js / Navigateur
  (execute le JavaScript normal)
```

TypeScript n'existe **pas** au runtime. Une fois compile, il ne reste que du JavaScript pur. Les types sont comme un echafaudage de construction : indispensables pendant la construction, retires a la livraison.

### Comparaison directe

Voici le meme code en JavaScript et en TypeScript :

```javascript
// JavaScript — aucune contrainte
function additionner(a, b) {
    return a + b;
}

additionner(5, 3);       // 8
additionner("5", 3);     // "53" — bug silencieux !
additionner(null, 3);    // 3 — comportement inattendu !
```

```typescript
// TypeScript — contrat explicite
function additionner(a: number, b: number): number {
    return a + b;
}

additionner(5, 3);       // OK : 8
additionner("5", 3);     // ERREUR a la compilation
additionner(null, 3);    // ERREUR a la compilation
```

Le bug qui aurait mis 2 heures a debugger en production est detecte en 2 secondes dans ton editeur.

---

## Pourquoi TypeScript ?

### 1. Les erreurs apparaissent au bon moment

En JavaScript, les bugs de types se manifestent au runtime — parfois en production, sur les donnees d'un vrai utilisateur. En TypeScript, ils sont detectes **pendant que tu codes**.

```
JS sans TypeScript :
  Ecriture du code -> Tests -> Deploiement -> BUG EN PRODUCTION -> Debug
                                                                   (2h de sueur)

JS avec TypeScript :
  Ecriture du code -> ERREUR IMMEDIATE -> Correction -> Tests -> Deploiement
                      (2 secondes)
```

### 2. L'autocompletion devient intelligente

Avec TypeScript, ton editeur (VS Code, WebStorm, etc.) sait **exactement** quelles proprietes et methodes sont disponibles sur chaque objet. Il peut te les suggerer sans que tu aies a consulter la documentation.

```typescript
interface Utilisateur {
    nom: string;
    age: number;
    email: string;
}

const user: Utilisateur = { nom: "Alice", age: 22, email: "alice@example.com" };

user.  // <- l'editeur propose : nom, age, email
       // Plus de fautes de frappe sur les noms de proprietes
```

### 3. Le refactoring devient sur

Renommer une fonction dans un projet JavaScript de 50 fichiers ? Bonne chance pour trouver tous les appels. En TypeScript, le compilateur te dit exactement quels fichiers utilisent quoi. Un renommage devient fiable.

### 4. La documentation est dans le code

Les types **sont** la documentation. Quand tu lis `function creerSession(userId: number, options: SessionOptions): Promise<Session>`, tu sais immediatement ce que la fonction attend et ce qu'elle retourne. Sans commentaire supplementaire.

> [!info] TypeScript dans l'industrie
> En 2024, TypeScript est utilise par la majorite des grandes entreprises tech (Microsoft, Google, Airbnb, Slack, Discord...). Le sondage Stack Overflow le classe regulierement dans les 5 langages les plus aimes. Maitriser TypeScript est aujourd'hui un critere de recrutement explicite pour les postes frontend et backend Node.js.

### Tableau de comparaison

| Critere              | JavaScript                        | TypeScript                             |
|----------------------|-----------------------------------|----------------------------------------|
| Detection d'erreurs  | Runtime (execution)               | Compilation (avant execution)          |
| Autocompletion       | Limitee (inference de base)       | Precise et complete                    |
| Refactoring          | Risque (recherche manuelle)       | Sur (compilateur guide)                |
| Courbe d'apprentissage | Immediate                       | Quelques jours supplementaires         |
| Compatibilite        | Navigateur / Node natif           | Necessite compilation vers JS          |
| Taille des projets   | OK pour petits projets            | Indispensable pour projets moyens/grands |

---

## Installation et Configuration

### Prerequis

Tu dois avoir Node.js et npm installes. Voir [[JavaScript/01 - Introduction a JavaScript]] pour le setup de base.

### Installer TypeScript

```bash
# Installation globale (disponible partout sur ta machine)
npm install -g typescript

# Verifier l'installation
tsc --version
# Affiche : Version 5.x.x

# Ou installation locale au projet (recommande en equipe)
npm install --save-dev typescript
```

### Ton premier fichier TypeScript

Cree un fichier `bonjour.ts` :

```typescript
// bonjour.ts
const message: string = "Bonjour, TypeScript !";
console.log(message);

function saluer(prenom: string): string {
    return `Bonjour, ${prenom} !`;
}

console.log(saluer("Holberton"));
```

Compile et execute :

```bash
# Compilation : genere bonjour.js
tsc bonjour.ts

# Execution du fichier JS genere
node bonjour.js

# Ou en une seule commande avec ts-node (voir plus bas)
npx ts-node bonjour.ts
```

### Le fichier tsconfig.json

Pour un vrai projet, on ne compile pas fichier par fichier. On configure tout dans `tsconfig.json` a la racine du projet.

```bash
# Genere un tsconfig.json avec les options commentees
tsc --init
```

Le fichier genere contient des dizaines d'options. Voici les plus importantes :

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

| Option                        | Ce qu'elle fait                                                    |
|-------------------------------|---------------------------------------------------------------------|
| `target`                      | Version JS de sortie (ES5, ES2020, ESNext...)                      |
| `module`                      | Systeme de modules (`commonjs` pour Node, `esnext` pour bundlers)  |
| `strict`                      | Active TOUS les checks stricts (recommande fortement)              |
| `outDir`                      | Dossier de sortie des fichiers `.js` compiles                      |
| `rootDir`                     | Dossier source des fichiers `.ts`                                   |
| `esModuleInterop`             | Imports compatibles avec les modules CommonJS                       |
| `skipLibCheck`                | Ignore les erreurs dans les `.d.ts` des librairies tierces          |

> [!warning] Active `strict: true` des le debut
> L'option `strict` active un ensemble de verifications qui peuvent sembler contraignantes au debut, mais qui capturent les bugs les plus courants. Si tu ne l'actives pas maintenant, tu devras refactoriser plus tard quand le projet sera gros. C'est beaucoup plus douloureux. Demarre strict, reste strict.

### Structure de projet recommandee

```
mon-projet/
├── src/
│   ├── index.ts
│   ├── utils/
│   │   └── calculs.ts
│   └── types/
│       └── modeles.ts
├── dist/           (genere par tsc, ne pas committer)
├── tsconfig.json
├── package.json
└── .gitignore      (inclure dist/)
```

### ts-node — execution directe

`ts-node` execute les fichiers TypeScript directement sans etape de compilation intermediaire. Ideal pour le developpement.

```bash
# Installation
npm install --save-dev ts-node

# Execution directe
npx ts-node src/index.ts

# Mode watch (re-execute a chaque modification)
npx ts-node-dev src/index.ts
```

---

## Les Types Primitifs

TypeScript propose les memes types primitifs que JavaScript, plus quelques types supplementaires specifiques au systeme de types.

### Vue d'ensemble

```
Types TypeScript
|
|-- Types primitifs JS
|   |-- string
|   |-- number
|   |-- boolean
|   |-- null
|   `-- undefined
|
|-- Types specifiques TypeScript
|   |-- any        (desactive la verification de types)
|   |-- unknown    (inconnu mais sur)
|   |-- never      (n'arrive jamais)
|   `-- void       (pas de valeur de retour)
|
`-- Types complexes (voir section suivante)
    |-- object
    |-- array
    `-- tuple
```

### string

```typescript
const prenom: string = "Alice";
const salutation: string = `Bonjour, ${prenom} !`;
const vide: string = "";

// Erreur : ne peut pas assigner un nombre a une string
// const age: string = 42; // ERREUR
```

### number

En TypeScript (comme en JavaScript), il n'y a qu'un seul type `number` pour les entiers ET les decimaux. Pas de `int`, `float`, `double` distincts.

```typescript
const age: number = 22;
const prix: number = 19.99;
const temperature: number = -5;
const infini: number = Infinity;
const notANumber: number = NaN;    // NaN est de type number en JS/TS
const hex: number = 0xFF;          // 255
const binaire: number = 0b1010;    // 10
```

### boolean

```typescript
const estConnecte: boolean = true;
const estAdmin: boolean = false;

// Attention : ce n'est PAS la meme chose que les valeurs "truthy/falsy" JS
// TypeScript est strict sur les types booleens
// const mauvais: boolean = 1;    // ERREUR
// const mauvais: boolean = "";   // ERREUR
```

### null et undefined

```typescript
let valeurNull: null = null;
let valeurUndef: undefined = undefined;

// Avec strict: true, null et undefined ne sont PAS assignables
// aux autres types sans union explicite
// let nom: string = null;  // ERREUR avec strict: true
let nomOuNull: string | null = null;  // OK
```

> [!info] null vs undefined en pratique
> La distinction entre `null` et `undefined` est une source de confusion. La convention la plus repandue : `undefined` signifie "pas encore initialise / absent", `null` signifie "explicitement vide / absent intentionnel". TypeScript te force a gerer les deux cas explicitement avec `strict: true`.

### any

`any` desactive completement la verification de types pour une variable. C'est une porte de sortie — utile pour migrer du code JS progressivement, mais a eviter.

```typescript
let nimporteQuoi: any = 42;
nimporteQuoi = "une chaine";    // OK
nimporteQuoi = true;            // OK
nimporteQuoi = { foo: "bar" };  // OK
nimporteQuoi.methodeInventee(); // OK (pas d'erreur, mais bug potentiel)

// any "contamine" : si tu passes un any a une fonction typee,
// TypeScript ne verifie plus rien
```

> [!warning] `any` est une dette technique
> Chaque `any` dans ton code est un trou dans ton filet de securite. Il annule tous les benefices de TypeScript pour cette variable. Utilise `any` uniquement comme etape transitoire lors d'une migration. La regle en equipe pro : zero `any` en production, et les occurrences restantes sont commentees avec une justification.

### unknown

`unknown` est le type "je ne sais pas encore ce que c'est, mais je vais le verifier avant de l'utiliser". C'est la version **sure** de `any`.

```typescript
let valeurInconnue: unknown = "peut-etre une chaine";

// On ne peut PAS utiliser directement une valeur unknown
// valeurInconnue.toUpperCase(); // ERREUR

// On doit d'abord verifier le type (type guard)
if (typeof valeurInconnue === "string") {
    console.log(valeurInconnue.toUpperCase()); // OK, TypeScript sait que c'est une string
}

// Cas d'usage typique : donnees venues de l'exterieur (API, JSON.parse)
const donneesBrutes: unknown = JSON.parse('{"nom": "Alice"}');
```

### never

`never` represente une valeur qui **n'arrive jamais**. C'est le type de retour d'une fonction qui soit lance toujours une erreur, soit boucle infiniment.

```typescript
// Fonction qui ne retourne jamais (leve toujours une exception)
function erreurFatale(message: string): never {
    throw new Error(message);
}

// Boucle infinie
function boucleInfinie(): never {
    while (true) {
        // ...
    }
}

// Cas avance : type "exhaustiveness check"
type Forme = "cercle" | "carre";

function aire(forme: Forme): number {
    if (forme === "cercle") return 3.14;
    if (forme === "carre") return 1;
    // Si on oublie un cas, TypeScript detecte l'erreur ici
    const _exhaustif: never = forme; // ERREUR si forme a encore des cas non traites
    return _exhaustif;
}
```

### void

`void` est le type de retour d'une fonction qui ne retourne rien (ou retourne `undefined`).

```typescript
function logMessage(message: string): void {
    console.log(message);
    // Pas de return (ou return; sans valeur)
}

// void est different de undefined :
// - void = "je ne retourne rien d'utile"
// - undefined = "la valeur est undefined"
// En pratique, la difference est subtile mais importante pour les callbacks
```

### Tableau recapitulatif

| Type        | Description                            | A utiliser quand...                          |
|-------------|----------------------------------------|----------------------------------------------|
| `string`    | Texte                                  | Toujours pour du texte                       |
| `number`    | Nombre (entier ou decimal)             | Calculs, compteurs, indices                  |
| `boolean`   | Vrai/Faux                              | Conditions, flags                            |
| `null`      | Absence intentionnelle de valeur       | Valeur explicitement vide                    |
| `undefined` | Variable non initialisee               | Valeur absente / pas encore assignee         |
| `any`       | Desactive les types                    | Migration JS → TS (transitoire uniquement)   |
| `unknown`   | Type inconnu mais sur                  | Donnees externes (API, JSON), inputs         |
| `never`     | N'arrive jamais                        | Fonctions qui throws, boucles infinies       |
| `void`      | Pas de valeur de retour                | Fonctions a effets de bord                   |

---

## Type Inference vs Annotation Explicite

TypeScript est suffisamment intelligent pour **deviner** le type d'une variable a partir de sa valeur d'initialisation. C'est l'inference de types.

### L'inference automatique

```typescript
// TypeScript infere automatiquement le type
const age = 22;           // TypeScript sait que c'est un number
const nom = "Alice";      // TypeScript sait que c'est une string
const actif = true;       // TypeScript sait que c'est un boolean

// Ces deux lignes sont equivalentes :
const a: number = 42;
const b = 42;             // TypeScript infere number
```

### Quand annoter explicitement ?

L'inference fonctionne bien pour les initialisations simples. Mais il y a des cas ou l'annotation explicite est necessaire ou recommandee :

```typescript
// 1. Variables declarees sans valeur initiale
let compteur: number;     // Pas encore de valeur → annotation obligatoire
compteur = 0;

// 2. Parametres de fonctions (TypeScript ne peut pas les inferer)
function multiplier(a: number, b: number): number {
    return a * b;
}

// 3. Quand l'inference donne un type trop large
const statuts = ["actif", "inactif", "en-attente"];
// TypeScript infere : string[]
// Si on veut contraindre a ces valeurs uniquement :
const statutsPrecis: ("actif" | "inactif" | "en-attente")[] = ["actif", "inactif"];

// 4. Objets avec structure complexe (pour la lisibilite)
const utilisateur: { nom: string; age: number } = {
    nom: "Bob",
    age: 30
};
```

> [!tip] La regle pratique
> Laisse TypeScript inferer quand c'est clair et evident (initialisations simples). Annote explicitement les signatures de fonctions, les variables non initialisees, et les cas ou l'inference donnerait un type trop large ou ambigu. En equipe, un style coherent est plus important que d'annoter absolument tout.

### Inference dans les fonctions

```typescript
// TypeScript infere le type de retour a partir du corps de la fonction
function double(x: number) {
    return x * 2; // TypeScript infere : retourne number
}

// Mais annoter le type de retour explicitement est une bonne pratique
// pour les fonctions publiques d'un module
function doubleExplicite(x: number): number {
    return x * 2;
}

// Si tu te trompes, TypeScript te previent :
function mauvaisRetour(x: number): string {
    return x * 2; // ERREUR : number n'est pas assignable a string
}
```

---

## Interfaces et Type Alias

Pour typer des structures complexes (objets, fonctions), TypeScript offre deux outils principaux : `interface` et `type`. Ce sont les deux facon de definir la "forme" d'un objet.

### Interface

Une interface definit le contrat qu'un objet doit respecter.

```typescript
interface Utilisateur {
    id: number;
    nom: string;
    email: string;
    age?: number;        // Propriete optionnelle (le ? la rend optionnelle)
    readonly creeLe: Date; // Propriete en lecture seule
}

const alice: Utilisateur = {
    id: 1,
    nom: "Alice",
    email: "alice@example.com",
    creeLe: new Date()
};

// age est optionnel, on peut l'omettre
// creeLe est readonly, on ne peut pas le modifier apres creation
// alice.creeLe = new Date(); // ERREUR
```

### Extension d'interfaces

Les interfaces peuvent s'etendre les unes les autres — comme l'heritage en POO.

```typescript
interface Personne {
    nom: string;
    age: number;
}

interface Employe extends Personne {
    poste: string;
    salaire: number;
}

// Employe a : nom, age, poste, salaire
const employe: Employe = {
    nom: "Bob",
    age: 35,
    poste: "Developpeur",
    salaire: 45000
};

// On peut etendre plusieurs interfaces
interface Manager extends Employe {
    equipe: Employe[];
}
```

### Type Alias

Un type alias cree un nouveau nom pour n'importe quel type — pas seulement les objets.

```typescript
// Alias pour un type primitif
type Identifiant = number;
type NomUtilisateur = string;

// Alias pour un objet (meme syntaxe qu'interface, avec =)
type Point = {
    x: number;
    y: number;
};

// Alias pour un union type (impossible avec interface)
type ResultatOperation = "succes" | "echec" | "en-attente";

// Alias pour une fonction
type FonctionComparaison = (a: number, b: number) => boolean;

const estPlusGrand: FonctionComparaison = (a, b) => a > b;
```

### Interface vs Type Alias : quand choisir ?

```
+---------------------------+------------------+-------------------+
|          Capacite         |    interface     |    type alias     |
+---------------------------+------------------+-------------------+
| Typer un objet            |       OUI        |        OUI        |
| Extension / heritage      |  extends (clair) |  intersection &   |
| Union types (A | B)       |       NON        |        OUI        |
| Tuples                    |       NON        |        OUI        |
| Declaration merging       |       OUI        |        NON        |
| Primitives et fonctions   |       NON        |        OUI        |
+---------------------------+------------------+-------------------+
```

> [!info] Declaration merging
> Une particularite d'`interface` : on peut declarer la meme interface plusieurs fois, et TypeScript les fusionne. Utile pour etendre des types de librairies tierces. `type` ne supporte pas ca — une deuxieme declaration du meme nom = erreur.

```typescript
// Declaration merging avec interface
interface Animal {
    nom: string;
}

interface Animal {
    age: number;
}

// Animal a maintenant : nom ET age
const chat: Animal = { nom: "Felix", age: 3 }; // OK
```

**Regle de decision simple :**
- Tu types un **objet** et tu veux pouvoir l'etendre / l'implementer dans une classe → `interface`
- Tu as besoin de **unions, tuples, ou types complexes** → `type alias`
- En cas de doute : `interface` pour les objets publics, `type` pour tout le reste

---

## Enums

Un enum (enumeration) est un ensemble de constantes nommees. Plutot qu'eparpiller des valeurs magiques dans le code, on les regroupe sous un nom logique.

### Enum numerique (defaut)

```typescript
enum Direction {
    Haut,       // 0
    Bas,        // 1
    Gauche,     // 2
    Droite      // 3
}

let direction: Direction = Direction.Haut;
console.log(direction);           // 0
console.log(Direction[0]);        // "Haut" (mapping inverse)

// On peut specifier les valeurs de depart
enum StatusCode {
    OK = 200,
    Cree = 201,
    BadRequest = 400,
    NonAutorise = 401,
    NotFound = 404,
    Erreur = 500
}
```

### Enum de chaines (string enum)

```typescript
enum Couleur {
    Rouge = "ROUGE",
    Vert = "VERT",
    Bleu = "BLEU"
}

// Plus lisible en debug que les valeurs numeriques
console.log(Couleur.Rouge);  // "ROUGE" au lieu de 0
```

### const enum — version optimisee

Un `const enum` est remplace **inline** a la compilation : aucun objet enum n'est genere dans le JS de sortie. Plus performant, mais pas de mapping inverse.

```typescript
const enum Permission {
    Lecture = 1,
    Ecriture = 2,
    Admin = 4
}

// Le JS genere remplace les references par les valeurs directement
// Permission.Lecture → 1 (pas d'objet Permission dans le JS)
const maPermission: Permission = Permission.Lecture;
```

> [!tip] Quand utiliser const enum ?
> Utilise `const enum` quand tu n'as pas besoin de mapper les valeurs en sens inverse (valeur → nom) et que la performance compte. Pour tout le reste, l'enum standard suffit. Certaines configurations (Babel, isolatedModules) sont incompatibles avec `const enum` — a verifier dans ton projet.

### Alternative moderne : union de litteraux

Pour des cas simples, les unions de litteraux remplacent souvent avantageusement les enums :

```typescript
// Avec enum
enum Statut {
    Actif = "actif",
    Inactif = "inactif"
}

// Avec union — plus leger, plus flexible
type Statut = "actif" | "inactif";

// Les deux approches sont valides ; les unions sont devenues la preference
// dans la communaute TypeScript moderne car plus legeres et composables
```

---

## Arrays et Tuples

### Arrays (tableaux)

TypeScript permet de typer le contenu d'un tableau de deux facons equivalentes :

```typescript
// Syntaxe 1 : Type[]
const nombres: number[] = [1, 2, 3, 4, 5];
const noms: string[] = ["Alice", "Bob", "Charlie"];

// Syntaxe 2 : Array<Type> (syntaxe generique)
const entiers: Array<number> = [10, 20, 30];

// Tableau de types complexes
interface Produit {
    id: number;
    nom: string;
    prix: number;
}

const catalogue: Produit[] = [
    { id: 1, nom: "Clavier", prix: 79 },
    { id: 2, nom: "Souris", prix: 45 }
];

// Tableau en lecture seule
const constantes: readonly number[] = [1, 2, 3];
// constantes.push(4); // ERREUR
```

### Tuples

Un tuple est un tableau avec un nombre **fixe** d'elements et des types specifiques pour chaque position.

```typescript
// Un tuple [string, number] : toujours une string en position 0, un number en position 1
let coordonnees: [number, number] = [48.8566, 2.3522]; // Paris

// Tuple nommee (TypeScript 4+)
type Entree = [nom: string, age: number];
const utilisateur: Entree = ["Alice", 22];

// Acces par index
console.log(utilisateur[0]); // "Alice"
console.log(utilisateur[1]); // 22

// Tuple avec element optionnel
type Plage = [debut: number, fin: number, pas?: number];
const plage1: Plage = [0, 100];
const plage2: Plage = [0, 100, 5];
```

> [!example] Cas d'usage concret des tuples
> Les tuples sont parfaits pour les retours multiples de fonctions (alternative a retourner un objet) et pour modeliser des donnees tabulaires.
>
> ```typescript
> // Fonction qui retourne [valeur, erreur] — pattern populaire inspire de Go
> function diviser(a: number, b: number): [number | null, string | null] {
>     if (b === 0) return [null, "Division par zero"];
>     return [a / b, null];
> }
>
> const [resultat, erreur] = diviser(10, 2);
> if (erreur) {
>     console.error(erreur);
> } else {
>     console.log(resultat); // 5
> }
> ```

---

## Union Types et Intersection Types

### Union Types — le "ou"

Un union type exprime qu'une valeur peut etre **l'un ou l'autre** des types listes.

```typescript
// string OU number
let identifiant: string | number;
identifiant = "abc123";  // OK
identifiant = 42;        // OK
identifiant = true;      // ERREUR

// Fonction qui accepte plusieurs types
function afficher(valeur: string | number | boolean): void {
    console.log(String(valeur));
}

// Union avec null (pattern tres courant)
function trouverUtilisateur(id: number): Utilisateur | null {
    // retourne l'utilisateur ou null si non trouve
}
```

### Narrowing — affiner le type dans un bloc

Quand on a un union type, TypeScript exige qu'on verifie quel type on a avant d'utiliser des methodes specifiques. C'est le **narrowing** (affinement de type).

```typescript
function traiter(valeur: string | number): string {
    if (typeof valeur === "string") {
        // Ici TypeScript sait que valeur est une string
        return valeur.toUpperCase();
    } else {
        // Ici TypeScript sait que valeur est un number
        return valeur.toFixed(2);
    }
}

// Narrowing avec instanceof
function formater(date: Date | string): string {
    if (date instanceof Date) {
        return date.toLocaleDateString();
    }
    return date; // TypeScript sait que c'est une string ici
}
```

### Intersection Types — le "et"

Un intersection type combine plusieurs types : la valeur doit satisfaire **tous** les types en meme temps.

```typescript
interface Identifiable {
    id: number;
}

interface Nomme {
    nom: string;
}

// Un type qui est Identifiable ET Nomme
type EntiteNommee = Identifiable & Nomme;

const produit: EntiteNommee = {
    id: 1,
    nom: "Clavier"
    // Doit avoir les deux proprietes
};

// Utile pour fusionner des types au lieu d'etendre des interfaces
type UtilisateurAvecPermissions = Utilisateur & { permissions: string[] };
```

> [!info] Union vs Intersection : le moyen de s'en souvenir
> - `A | B` = "soit A soit B" → **union** (ensemble plus grand)
> - `A & B` = "a la fois A et B" → **intersection** (ensemble plus contraint)
>
> Contre-intuitivement, l'intersection de types produit un type avec **plus** de proprietes (pas moins) — parce que la valeur doit satisfaire les contraintes des deux types.

---

## Fonctions Typees

Voir aussi [[JavaScript/04 - JavaScript Moderne ES6+]] pour les fonctions fleche et les defaults ES6 que TypeScript etend.

### Syntaxe de base

```typescript
// Parametres types + type de retour
function additionner(a: number, b: number): number {
    return a + b;
}

// Fonction fleche typee
const multiplier = (a: number, b: number): number => a * b;

// TypeScript infere le type de retour, mais l'annotation explicite est recommandee
// pour les fonctions publiques
```

### Parametres optionnels et valeurs par defaut

```typescript
// Parametre optionnel avec ?
// Le parametre peut etre omis ou passe comme undefined
function saluer(nom: string, titre?: string): string {
    if (titre) {
        return `Bonjour, ${titre} ${nom} !`;
    }
    return `Bonjour, ${nom} !`;
}

saluer("Alice");           // "Bonjour, Alice !"
saluer("Alice", "Dr.");    // "Bonjour, Dr. Alice !"

// Valeur par defaut (comme ES6, mais TypeScript infere le type)
function creerProfil(nom: string, role: string = "utilisateur"): string {
    return `${nom} (${role})`;
}

creerProfil("Bob");             // "Bob (utilisateur)"
creerProfil("Alice", "admin");  // "Alice (admin)"
```

### Parametres rest

```typescript
// ...args capture tous les arguments restants dans un tableau
function somme(premier: number, ...reste: number[]): number {
    return reste.reduce((acc, val) => acc + val, premier);
}

somme(1, 2, 3, 4, 5); // 15
```

### Surcharge de fonctions (function overloading)

TypeScript permet de declarer plusieurs signatures pour une meme fonction. Le corps de la fonction implemente toutes les surcharges.

```typescript
// Declarations de surcharge (signatures)
function combiner(a: string, b: string): string;
function combiner(a: number, b: number): number;

// Implementation (non visible de l'exterieur)
function combiner(a: string | number, b: string | number): string | number {
    if (typeof a === "string" && typeof b === "string") {
        return a + b;
    }
    if (typeof a === "number" && typeof b === "number") {
        return a + b;
    }
    throw new Error("Types incompatibles");
}

combiner("Hello, ", "World");  // "Hello, World"
combiner(10, 20);              // 30
combiner("10", 20);            // ERREUR (pas de surcharge pour string + number)
```

### Typer les callbacks et fonctions comme valeurs

```typescript
// Type d'une fonction comme parametre
function appliquer(valeurs: number[], fn: (x: number) => number): number[] {
    return valeurs.map(fn);
}

appliquer([1, 2, 3], x => x * 2); // [2, 4, 6]

// Interface pour une fonction
interface Comparateur {
    (a: number, b: number): number;
}

const trierCroissant: Comparateur = (a, b) => a - b;
```

---

## Type Assertions et Non-Null Assertion

### Type Assertion (`as`)

Parfois, tu en sais plus que TypeScript sur le type d'une valeur. La type assertion permet de dire "fais-moi confiance, c'est ce type".

```typescript
// L'element peut etre null selon TypeScript, mais tu sais qu'il existe
const canvas = document.getElementById("monCanvas") as HTMLCanvasElement;
const ctx = canvas.getContext("2d"); // Maintenant TypeScript sait que c'est un HTMLCanvasElement

// Syntaxe alternative (ancienne, evitee dans les fichiers .tsx)
const canvas2 = <HTMLCanvasElement>document.getElementById("monCanvas");

// Cas courant : JSON.parse retourne any, on l'asserte
interface Config {
    host: string;
    port: number;
}
const config = JSON.parse('{"host":"localhost","port":3000}') as Config;
```

> [!warning] La type assertion n'est pas un cast
> En C++ ou Java, un cast peut convertir une valeur. En TypeScript, `as` ne fait **rien** au runtime. Ca dit juste au compilateur "traite cette valeur comme ce type". Si la valeur n'est pas reellement de ce type au runtime, tu auras des bugs. Utilise `as` avec parcimonie, uniquement quand tu as une certitude que le compilateur ne peut pas voir.

### Double assertion

Si TypeScript refuse une assertion directe (les types sont trop eloignes), on peut passer par `unknown` :

```typescript
// Parfois necessaire, toujours suspect : a eviter si possible
const valeurForce = someValue as unknown as AutreType;
```

### Non-Null Assertion (`!`)

Le `!` postfixe dit a TypeScript "cette valeur n'est ni `null` ni `undefined`, je te le garantis".

```typescript
// TypeScript pense que querySelector peut retourner null
const bouton = document.querySelector("#submit")!; // ! = "je garantis que c'est non-null"
bouton.addEventListener("click", () => {}); // OK sans verification

// Sans le !, TypeScript exigerait une verification :
const boutonSur = document.querySelector("#submit");
if (boutonSur) {
    boutonSur.addEventListener("click", () => {});
}
```

> [!warning] `!` est dangereux
> Comme `as`, le `!` bypass les protections de TypeScript. Si l'element n'existe pas au runtime, tu auras un `TypeError: Cannot read properties of null`. Prefere toujours la verification explicite. Reserve `!` aux cas ou tu es absolument certain et ou la verification alourdirait inutilement le code (ex : tests unitaires).

---

## Optional Chaining et Nullish Coalescing

Ces deux operateurs viennent de JavaScript ES2020 et sont particulierement utiles avec TypeScript.

### Optional Chaining (`?.`)

Permet d'acceder a une propriete ou d'appeler une methode sans lancer d'erreur si la valeur est `null` ou `undefined`. L'expression retourne `undefined` au lieu de planter.

```typescript
interface Adresse {
    rue: string;
    ville: string;
}

interface Utilisateur {
    nom: string;
    adresse?: Adresse; // Optionnel
}

const user: Utilisateur = { nom: "Alice" };

// Sans optional chaining — verbose et risque
const ville1 = user.adresse !== undefined ? user.adresse.ville : undefined;

// Avec optional chaining — elegant
const ville2 = user.adresse?.ville; // undefined si adresse est absent

// Fonctionne aussi pour les appels de methodes
const longueur = user.adresse?.rue?.length;

// Et pour les appels de fonctions
type Handler = (() => void) | undefined;
const onClick: Handler = undefined;
onClick?.(); // Rien ne se passe, pas d'erreur

// Et pour les acces par index
const premier = user?.adresse?.rue?.[0];
```

> [!example] Cas pratique : donnees d'API
> Les reponses d'API ont souvent des structures profondement imbriquees avec des valeurs optionnelles. `?.` remplace une pyramide de conditions.
>
> ```typescript
> // Sans ?.
> const pays = reponse
>     && reponse.data
>     && reponse.data.utilisateur
>     && reponse.data.utilisateur.adresse
>     && reponse.data.utilisateur.adresse.pays;
>
> // Avec ?.
> const pays = reponse?.data?.utilisateur?.adresse?.pays;
> ```

### Nullish Coalescing (`??`)

Retourne le cote droit si le cote gauche est `null` ou `undefined`. Contrairement a `||`, il ne considere **pas** `0`, `""`, `false` comme des valeurs "manquantes".

```typescript
// Le probleme avec || :
const volume = userVolume || 50;
// Si userVolume = 0 (silencieux), || donne 50 — bug !
// 0 est falsy en JS, donc || l'ignore

// La solution avec ?? :
const volumeCorrect = userVolume ?? 50;
// Si userVolume = 0, ?? retourne 0 (0 n'est pas null/undefined)
// Si userVolume = null ou undefined, ?? retourne 50

// Combinaison avec ?. (tres courant)
const prenom = user?.profil?.prenom ?? "Anonyme";
```

| Operateur | Retourne le droit si gauche est... |
|-----------|-------------------------------------|
| `\|\|`    | Falsy (`false`, `0`, `""`, `null`, `undefined`, `NaN`) |
| `??`      | `null` ou `undefined` uniquement    |

### Nullish Assignment (`??=`)

```typescript
// Assigne la valeur de droite uniquement si la variable est null/undefined
let pseudo: string | null = null;
pseudo ??= "Anonyme"; // pseudo devient "Anonyme"

let pseudo2 = "Alice";
pseudo2 ??= "Anonyme"; // pseudo2 reste "Alice"
```

---

## Compilation avec tsc et ts-node

### Compilation avec tsc

```bash
# Compiler un fichier unique
tsc fichier.ts

# Compiler tout le projet (utilise tsconfig.json)
tsc

# Mode watch : recompile a chaque modification
tsc --watch
# ou
tsc -w

# Verifier les types sans generer de fichiers
tsc --noEmit

# Cibler une version JS specifique
tsc --target ES2020 fichier.ts
```

### Mode strict et flags utiles

```bash
# Activer le mode strict pour ce build
tsc --strict

# Specifier le dossier de sortie
tsc --outDir dist

# Voir la version
tsc --version

# Afficher l'aide
tsc --help
```

### ts-node — execution directe

```bash
# Installation dans le projet
npm install --save-dev ts-node @types/node

# Execution directe d'un fichier .ts
npx ts-node src/index.ts

# Mode REPL TypeScript interactif
npx ts-node

# ts-node-dev : comme nodemon mais pour TypeScript
npm install --save-dev ts-node-dev
npx ts-node-dev --respawn src/index.ts
```

### Workflow de developpement type

```
Developpement local                     Production
|                                       |
| src/index.ts  →  npx ts-node          | src/index.ts
|                  (direct, rapide)     |     |
|                                       |    tsc
|                                       |     |
|                                       | dist/index.js
|                                       |     |
|                                       |  node dist/index.js
```

### Les fichiers de declaration `.d.ts`

Quand tu utilises une librairie JavaScript tierce, TypeScript a besoin de connaitre ses types. Les fichiers `.d.ts` contiennent uniquement des declarations de types (pas de code executable).

```bash
# Installer les types pour une librairie (ex: Node.js)
npm install --save-dev @types/node

# Pour Express
npm install --save-dev @types/express

# Pour lodash
npm install --save-dev @types/lodash
```

Les librairies modernes incluent souvent leurs propres types. Chercher sur https://www.npmjs.com/package/@types/NOM si tu as des erreurs de type sur une librairie tierce.

---

## Carte Mentale

```
                    TYPESCRIPT
                        |
        +---------------+----------------+
        |               |                |
   POURQUOI ?       SETUP               TYPES
        |               |                |
  Erreurs a la      tsconfig.json    Primitifs
  compilation       |     |          |       |
        |           tsc   ts-node  string  number
  Autocompletion        |          boolean  null
  precise           strict:true   undefined void
        |                          any   unknown
  Refactoring                      never
  sur                                |
                              Types complexes
                              |            |
                          Interface      Array
                          Type alias     Tuple
                              |
                         Union |          & Intersection
                         A | B            A & B
                                |
                          FONCTIONS
                          |        |
                    Optionnels   Surcharge
                    Defaults     Callbacks
                          |
                    OPERATEURS
                    |         |
                   ?.        ??
              Optional    Nullish
              Chaining    Coalescing
                    |
               ASSERTIONS
               |         |
              as          !
           Type       Non-Null
         Assertion   Assertion
```

---

## Exercices Pratiques

### Exercice 1 — Typer un systeme d'utilisateurs

Cree un fichier `utilisateurs.ts` qui :

1. Definit une interface `Utilisateur` avec : `id` (number), `nom` (string), `email` (string), `age` (number optionnel), `role` (un enum `Role` avec les valeurs `Admin`, `Editeur`, `Lecteur`)
2. Cree un type `NouvelUtilisateur` qui est `Omit<Utilisateur, 'id'>` (Utilisateur sans l'id)
3. Ecrit une fonction `creerUtilisateur(data: NouvelUtilisateur): Utilisateur` qui ajoute un id auto-incremente
4. Ecrit une fonction `trouverParEmail(utilisateurs: Utilisateur[], email: string): Utilisateur | undefined`
5. Teste les deux fonctions

**Ce que tu dois verifier** : que TypeScript t'empeche d'assigner un role invalide, d'oublier un champ obligatoire, ou d'utiliser directement le resultat de `trouverParEmail` sans verifier s'il est `undefined`.

### Exercice 2 — Calculatrice avec surcharge

Cree un fichier `calculatrice.ts` avec une fonction `calculer` qui :

- Avec `(a: number, b: number, operation: "addition" | "soustraction" | "multiplication")` → retourne un `number`
- Avec `(a: string, b: string)` → retourne une `string` (concatenation)
- Utilise la surcharge TypeScript pour typer les deux signatures separement
- Inclut une fonction `afficherResultat(valeur: number | string): void` qui utilise le narrowing pour formater differemment selon le type

### Exercice 3 — Config avec types stricts

Cree un fichier `config.ts` qui :

1. Definit une interface `ConfigServeur` avec : `host`, `port`, `ssl` (boolean), `timeout` (optionnel, en ms), `maxConnexions` (optionnel)
2. Cree une fonction `chargerConfig(donneesBrutes: unknown): ConfigServeur` qui :
   - Valide que `donneesBrutes` est bien un objet
   - Extrait et valide les champs obligatoires
   - Utilise `??` pour les valeurs par defaut des champs optionnels
   - Lance une erreur typee si la config est invalide
3. Utilise `?.` pour acceder a `timeout` et `maxConnexions` de facon sure

### Exercice 4 — Refactoring d'un fichier JavaScript

Prends ce code JavaScript et migre-le vers TypeScript en ajoutant tous les types necessaires :

```javascript
// a-migrer.js
function calculerPanier(articles, codePromo) {
    let total = 0;
    
    for (const article of articles) {
        total += article.prix * article.quantite;
    }
    
    if (codePromo) {
        const remise = codePromo.type === "pourcentage"
            ? total * (codePromo.valeur / 100)
            : codePromo.valeur;
        total -= remise;
    }
    
    return {
        sousTotal: total,
        tva: total * 0.2,
        total: total * 1.2
    };
}

function formaterPrix(montant, devise) {
    return new Intl.NumberFormat("fr-FR", {
        style: "currency",
        currency: devise || "EUR"
    }).format(montant);
}
```

**Objectifs** : creer des interfaces pour `Article`, `CodePromo`, `ResultatPanier`, typer tous les parametres et retours, utiliser `??` pour la valeur par defaut de `devise`.

---

## Pour Aller Plus Loin

Les concepts abordes ici forment la base de TypeScript. Une fois maitrise, explore :

- **Generics** — fonctions et interfaces parametrees par des types (`Array<T>`, `Promise<T>`, et tes propres generics)
- **Utility Types** — `Partial<T>`, `Required<T>`, `Readonly<T>`, `Pick<T, K>`, `Omit<T, K>`, `Record<K, V>` et bien d'autres types utilitaires fournis par TypeScript
- **Conditional Types** — `T extends U ? X : Y` pour des types qui dependent d'autres types
- **Mapped Types** — transformer chaque propriete d'un type existant
- **Template Literal Types** — manipuler des types de chaines comme des templates

Tout ca est dans [[TypeScript/02 - Types Avances]].

> [!info] La philosophie TypeScript
> TypeScript n'est pas la pour rendre le code plus verbeux — il est la pour rendre le code plus **sur** et plus **lisible**. Si tu te retrouves a ecrire des annotations partout sans en tirer de valeur, c'est souvent un signe que l'inference peut faire le travail. Le bon TypeScript est du TypeScript ou les types sont presents la ou ils apportent de la valeur (interfaces publiques, donnees ambigues, frontiere systeme) et absents la ou ils sont redondants (variables locales evidentes).

---

*Note creee dans le cadre du cursus Holberton School — Programmation Web.*
*Prerequisites : [[JavaScript/01 - Introduction a JavaScript]], [[JavaScript/04 - JavaScript Moderne ES6+]]*
*Suite : [[TypeScript/02 - Types Avances]]*
