# JavaScript Asynchrone

JavaScript est un langage **single-threaded** : il n'execute qu'une seule instruction a la fois. Pourtant, il gere parfaitement les operations longues (requetes reseau, lectures fichier, timers) sans bloquer l'interface utilisateur. Comment ? Grace a un modele **asynchrone evenementiel** base sur l'event loop.

Ce cours explore pourquoi l'asynchrone est essentiel, comment fonctionne l'event loop, l'evolution des patterns (callbacks, Promises, async/await), et l'utilisation pratique de l'API fetch pour communiquer avec des serveurs.

> [!tip] Analogie
> Imaginez un restaurant avec un seul serveur (single-threaded). Ce serveur ne cuisine pas : il prend les commandes et les transmet en cuisine (API Web). Pendant que la cuisine prepare les plats (operations asynchrones), le serveur continue de prendre d'autres commandes. Quand un plat est pret, la cuisine le place sur le passe (callback queue), et le serveur le livre au bon moment (event loop). En C, vous gerez les threads manuellement (pthreads). En Python, vous avez asyncio avec async/await. En JS, l'asynchrone est au coeur du langage depuis le debut.

---

## 1. Pourquoi l'asynchrone ?

### Le probleme du code bloquant (synchrone)

```javascript
// ─── Code SYNCHRONE : chaque ligne attend la precedente ───
console.log("1. Debut");
// Imaginons une requete reseau qui prend 3 secondes...
// const data = requeteSynchrone("https://api.example.com"); // BLOQUE 3s
console.log("2. Donnees recues");
console.log("3. Suite du programme");

// Pendant ces 3 secondes :
// - L'interface utilisateur est GELEE
// - Aucun clic, aucun scroll, aucune animation
// - Le navigateur peut afficher "Page ne repond pas"
```

### La solution asynchrone

```javascript
// ─── Code ASYNCHRONE : les operations longues ne bloquent pas ───
console.log("1. Debut");

fetch("https://api.example.com/data")
    .then(response => response.json())
    .then(data => console.log("2. Donnees recues :", data));

console.log("3. Suite du programme (s'execute AVANT les donnees !)");

// Sortie :
// 1. Debut
// 3. Suite du programme
// 2. Donnees recues : { ... }    (arrive plus tard)
```

### Comparaison : synchrone vs asynchrone

```
SYNCHRONE (bloquant) :
─────────────────────
  Tache A ████████████████
                          Tache B ████████████████
                                                  Tache C ████
  Temps total = A + B + C

ASYNCHRONE (non-bloquant) :
───────────────────────────
  Tache A ████████████████
  Tache B ████████████████     (en parallele dans le systeme)
  Tache C ████                 (en parallele dans le systeme)
  Temps total = max(A, B, C)
```

> [!info] Single-threaded ne signifie pas lent
> JavaScript n'execute qu'un fil d'execution a la fois, mais les operations I/O (reseau, fichiers, timers) sont deleguees au systeme d'exploitation qui, lui, est multi-threaded. JS ne fait qu'orchestrer ces operations.

---

## 2. L'Event Loop

L'event loop est le mecanisme central qui coordonne l'execution du code JavaScript.

### Les composants

```
┌─────────────────────────────────────────────────────────────────────┐
│                         JAVASCRIPT ENGINE                          │
│                                                                     │
│  ┌────────────────┐         ┌────────────────────────────────┐     │
│  │   Call Stack    │         │         Web APIs               │     │
│  │  (pile d'appels)│         │  (fournies par le navigateur   │     │
│  │                │    ───>  │   ou Node.js)                  │     │
│  │  fonction()    │         │                                │     │
│  │  main()        │         │  setTimeout()                  │     │
│  │                │         │  fetch()                       │     │
│  └────────┬───────┘         │  addEventListener()            │     │
│           │                 │  XMLHttpRequest                │     │
│           │                 └──────────┬─────────────────────┘     │
│           │                            │                           │
│    ┌──────┴──────────────────────────┐ │                           │
│    │          EVENT LOOP             │ │  Quand une operation      │
│    │  (boucle infinie qui verifie    │<┘  async est terminee,      │
│    │   les queues)                   │    son callback est place   │
│    └──────┬───────────┬──────────────┘    dans une queue          │
│           │           │                                            │
│  ┌────────┴───┐  ┌────┴──────────────┐                            │
│  │ Microtask  │  │  Macrotask Queue   │                            │
│  │  Queue     │  │  (Callback Queue)  │                            │
│  │ (priorite) │  │                    │                            │
│  │            │  │  setTimeout cb     │                            │
│  │ Promise.then│  │  setInterval cb   │                            │
│  │ queueMicro │  │  I/O callbacks     │                            │
│  │ MutationObs│  │  UI rendering      │                            │
│  └────────────┘  └────────────────────┘                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Algorithme de l'Event Loop

```
1. Executer tout le code synchrone dans le Call Stack
2. Le Call Stack est vide ?
   ├── Oui :
   │   a. Vider ENTIEREMENT la Microtask Queue (Promises)
   │   b. Executer UNE tache de la Macrotask Queue (setTimeout, etc.)
   │   c. Retour a l'etape 2
   └── Non : continuer l'execution
```

> [!warning] Priorite des microtasks
> Les microtasks (Promises) sont **toujours** executees avant les macrotasks (setTimeout). Meme un `setTimeout(fn, 0)` s'execute apres toutes les Promises en attente.

### Demonstration de l'ordre d'execution

```javascript
console.log("1 - Synchrone");

setTimeout(() => {
    console.log("2 - setTimeout (macrotask)");
}, 0);

Promise.resolve().then(() => {
    console.log("3 - Promise (microtask)");
});

Promise.resolve().then(() => {
    console.log("4 - Promise 2 (microtask)");
});

console.log("5 - Synchrone");

// Sortie :
// 1 - Synchrone       (call stack)
// 5 - Synchrone       (call stack)
// 3 - Promise          (microtask queue - prioritaire)
// 4 - Promise 2        (microtask queue - prioritaire)
// 2 - setTimeout       (macrotask queue - apres les microtasks)
```

### Visualisation pas a pas

```
Etape 1 : Call Stack execute le code synchrone
  Call Stack:    [console.log("1"), setTimeout(...), Promise.resolve()..., console.log("5")]
  Microtask Q:   []
  Macrotask Q:   []
  Sortie:        "1 - Synchrone"

Etape 2 : setTimeout place son callback dans la Macrotask Queue
  Call Stack:    [suite du code]
  Microtask Q:   []
  Macrotask Q:   [cb_setTimeout]
  Sortie:        "1 - Synchrone"

Etape 3 : Promise.then place son callback dans la Microtask Queue
  Call Stack:    [suite du code]
  Microtask Q:   [cb_promise1, cb_promise2]
  Macrotask Q:   [cb_setTimeout]
  Sortie:        "1 - Synchrone", "5 - Synchrone"

Etape 4 : Call Stack vide -> Vider la Microtask Queue
  Call Stack:    [cb_promise1]
  Microtask Q:   [cb_promise2]
  Macrotask Q:   [cb_setTimeout]
  Sortie:        + "3 - Promise", "4 - Promise 2"

Etape 5 : Microtask Queue vide -> Executer 1 Macrotask
  Call Stack:    [cb_setTimeout]
  Microtask Q:   []
  Macrotask Q:   []
  Sortie:        + "2 - setTimeout"
```

---

## 3. Callbacks

Un **callback** est une fonction passee en argument a une autre fonction, qui sera appelee plus tard (quand l'operation asynchrone est terminee).

### Concept de base

```javascript
// ─── Callback synchrone (pour comparaison) ───
const nombres = [3, 1, 4, 1, 5];
nombres.sort((a, b) => a - b);  // Le callback est appele pour chaque comparaison

// ─── Callback asynchrone ───
console.log("Avant");

setTimeout(() => {
    console.log("Pendant (apres 1 seconde)");
}, 1000);

console.log("Apres (s'execute avant 'Pendant')");
```

### Exemple : lire un fichier (Node.js)

```javascript
// ─── C : fopen + fread (synchrone, bloquant) ───
// FILE *f = fopen("data.txt", "r");
// fread(buffer, 1, size, f);   // Bloque jusqu'a la lecture complete

// ─── Python : open (synchrone par defaut) ───
// with open("data.txt") as f:
//     data = f.read()          # Bloque aussi

// ─── JavaScript Node.js : callback asynchrone ───
const fs = require("fs");

fs.readFile("data.txt", "utf8", (err, data) => {
    if (err) {
        console.error("Erreur :", err);
        return;
    }
    console.log("Contenu :", data);
});
console.log("Lecture lancee (pas encore terminee)");
```

### Le probleme : Callback Hell (Pyramid of Doom)

```javascript
// ─── Enchainer des operations asynchrones avec des callbacks ───
getUtilisateur(userId, (err, user) => {
    if (err) { handleError(err); return; }

    getCommandes(user.id, (err, commandes) => {
        if (err) { handleError(err); return; }

        getDetails(commandes[0].id, (err, details) => {
            if (err) { handleError(err); return; }

            getProduit(details.produitId, (err, produit) => {
                if (err) { handleError(err); return; }

                afficher(produit);
                // ... et ca continue ──>
            });
        });
    });
});
```

```
Visualisation du "Callback Hell" :

getUtilisateur ─────┐
                     └─ getCommandes ─────┐
                                          └─ getDetails ─────┐
                                                              └─ getProduit ──┐
                                                                              └─ afficher
Indentation croissante ──────────────────────────────────────────────────────────>
Lisibilite decroissante ──────────────────────────────────────────────────────────>
Maintenabilite nulle ────────────────────────────────────────────────────────────>
```

> [!warning] Pourquoi le callback hell est problematique
> - **Lisibilite** : le code va toujours plus a droite (pyramide)
> - **Gestion d'erreurs** : chaque niveau doit gerer ses propres erreurs
> - **Maintenabilite** : inserer une etape au milieu est un cauchemar
> - **Tests** : presque impossible a tester unitairement

---

## 4. Promises

Une **Promise** (promesse) est un objet qui represente le resultat futur d'une operation asynchrone. Elle peut etre dans l'un de trois etats :

```
┌──────────────────────────────────────────────────────┐
│                    Promise                           │
│                                                      │
│  ┌──────────┐    resolve(valeur)    ┌────────────┐  │
│  │ PENDING  │ ──────────────────>   │ FULFILLED  │  │
│  │ (en      │                       │ (tenue)    │  │
│  │ attente) │                       │ .then()    │  │
│  │          │    reject(erreur)     ├────────────┤  │
│  │          │ ──────────────────>   │ REJECTED   │  │
│  │          │                       │ (rejetee)  │  │
│  └──────────┘                       │ .catch()   │  │
│                                     └────────────┘  │
│                                                      │
│  Une fois settled (fulfilled ou rejected),           │
│  l'etat ne change PLUS.                              │
└──────────────────────────────────────────────────────┘
```

### Creer une Promise

```javascript
const maPromesse = new Promise((resolve, reject) => {
    // Operation asynchrone simulee
    const succes = true;

    setTimeout(() => {
        if (succes) {
            resolve("Donnees recues !");   // Passe a l'etat FULFILLED
        } else {
            reject(new Error("Echec !"));  // Passe a l'etat REJECTED
        }
    }, 1000);
});
```

### Consommer une Promise : .then(), .catch(), .finally()

```javascript
maPromesse
    .then(resultat => {
        console.log("Succes :", resultat);  // "Donnees recues !"
        return resultat.toUpperCase();       // Retourne une nouvelle valeur
    })
    .then(resultatModifie => {
        console.log("Modifie :", resultatModifie);
    })
    .catch(erreur => {
        console.error("Erreur :", erreur.message);
    })
    .finally(() => {
        console.log("Termine (succes ou echec)");
    });
```

### Chaining (enchainement)

```javascript
// ─── Callback hell transforme en chaine de Promises ───
getUtilisateur(userId)
    .then(user => getCommandes(user.id))
    .then(commandes => getDetails(commandes[0].id))
    .then(details => getProduit(details.produitId))
    .then(produit => afficher(produit))
    .catch(err => handleError(err));  // UNE SEULE gestion d'erreur !
```

```
Comparaison visuelle :

Callbacks :                          Promises :
─────────────────                    ─────────────
fn(a, (err, r1) => {                fn(a)
  fn(b, (err, r2) => {                .then(r1 => fn(b))
    fn(c, (err, r3) => {               .then(r2 => fn(c))
      fn(d, (err, r4) => {              .then(r3 => fn(d))
        // resultat final                .then(r4 => resultat)
      });                                .catch(err => erreur);
    });
  });
});
```

> [!tip] Analogie
> Une Promise est comme un ticket de commande au restaurant. Quand vous commandez (creez la Promise), on vous donne un ticket (pending). Soit le plat arrive (fulfilled = .then), soit la cuisine vous informe qu'un ingredient manque (rejected = .catch). Dans tous les cas, le serveur passe a la table suivante (finally).

### Creer des Promises resolues/rejetees immediatement

```javascript
// ─── Promise deja resolue ───
const resolue = Promise.resolve(42);
resolue.then(val => console.log(val));  // 42

// ─── Promise deja rejetee ───
const rejetee = Promise.reject(new Error("Echec"));
rejetee.catch(err => console.log(err.message));  // "Echec"
```

### Methodes statiques utilitaires

```javascript
const p1 = fetch("/api/users");
const p2 = fetch("/api/posts");
const p3 = fetch("/api/comments");

// ─── Promise.all : TOUTES doivent reussir ───
Promise.all([p1, p2, p3])
    .then(([users, posts, comments]) => {
        console.log("Tout recu !");
    })
    .catch(err => {
        console.log("Au moins UNE a echoue");
    });

// ─── Promise.allSettled : attend TOUTES, sans echouer ───
Promise.allSettled([p1, p2, p3])
    .then(resultats => {
        resultats.forEach(r => {
            if (r.status === "fulfilled") {
                console.log("Succes :", r.value);
            } else {
                console.log("Echec :", r.reason);
            }
        });
    });

// ─── Promise.race : la PREMIERE terminee (succes ou echec) ───
Promise.race([p1, p2, p3])
    .then(premierResultat => {
        console.log("Le plus rapide :", premierResultat);
    });

// ─── Promise.any : la PREMIERE qui REUSSIT ───
Promise.any([p1, p2, p3])
    .then(premierSucces => {
        console.log("Premier succes :", premierSucces);
    })
    .catch(err => {
        console.log("TOUTES ont echoue");  // AggregateError
    });
```

### Tableau comparatif des methodes statiques

```
+-------------------+------------------+--------------------+------------------+
| Methode           | Attend           | Resolve quand      | Reject quand     |
+-------------------+------------------+--------------------+------------------+
| Promise.all       | Toutes           | Toutes fulfilled   | 1 rejected       |
| Promise.allSettled| Toutes           | Toutes settled     | Jamais           |
| Promise.race      | La 1ere          | 1ere settled       | 1ere rejected    |
| Promise.any       | La 1ere reussie  | 1ere fulfilled     | Toutes rejected  |
+-------------------+------------------+--------------------+------------------+
```

---

## 5. async / await

`async/await` est du **sucre syntaxique** au-dessus des Promises. Le code asynchrone ressemble a du code synchrone, ce qui le rend beaucoup plus lisible.

### Syntaxe de base

```javascript
// ─── Avec Promises (.then) ───
function getUser(id) {
    return fetch(`/api/users/${id}`)
        .then(response => response.json())
        .then(data => data);
}

// ─── Avec async/await (equivalent) ───
async function getUser(id) {
    const response = await fetch(`/api/users/${id}`);
    const data = await response.json();
    return data;
}
```

### Regles

```javascript
// 1. await ne peut etre utilise que dans une fonction async
async function exemple() {
    const resultat = await unePromesse();
    return resultat;
}

// 2. Une fonction async retourne TOUJOURS une Promise
const resultat = exemple();  // Promise, pas la valeur directe !
resultat.then(val => console.log(val));

// 3. await "pause" l'execution de la fonction async
//    MAIS ne bloque PAS le reste du programme
async function demo() {
    console.log("A");
    await new Promise(r => setTimeout(r, 1000));  // Pause 1s
    console.log("B");  // Execute apres 1s
}
demo();
console.log("C");  // S'execute IMMEDIATEMENT, avant B

// Sortie : A, C, (1s) B
```

### Gestion d'erreurs avec try/catch

```javascript
async function chargerDonnees() {
    try {
        const response = await fetch("/api/data");

        if (!response.ok) {
            throw new Error(`Erreur HTTP : ${response.status}`);
        }

        const data = await response.json();
        return data;

    } catch (erreur) {
        console.error("Echec du chargement :", erreur.message);
        return null;  // Valeur de secours
    } finally {
        console.log("Chargement termine (succes ou echec)");
    }
}
```

### Execution parallele avec async/await

```javascript
// ─── MAUVAIS : sequentiel (lent) ───
async function chargerSequentiel() {
    const users = await fetch("/api/users").then(r => r.json());     // 1s
    const posts = await fetch("/api/posts").then(r => r.json());     // 1s
    const comments = await fetch("/api/comments").then(r => r.json());// 1s
    // Total : ~3 secondes
    return { users, posts, comments };
}

// ─── BON : parallele (rapide) ───
async function chargerParallele() {
    const [users, posts, comments] = await Promise.all([
        fetch("/api/users").then(r => r.json()),       // ┐
        fetch("/api/posts").then(r => r.json()),       // ├── en parallele
        fetch("/api/comments").then(r => r.json()),    // ┘
    ]);
    // Total : ~1 seconde (le plus lent des trois)
    return { users, posts, comments };
}
```

```
Sequentiel :
  users    ████████
                    posts    ████████
                                      comments ████████
  Temps : ─────────────────────────────────────────────> 3s

Parallele (Promise.all) :
  users    ████████
  posts    ████████
  comments ████████
  Temps : ──────────> 1s
```

> [!warning] await dans une boucle
> ```javascript
> // MAUVAIS : chaque iteration attend la precedente
> for (const url of urls) {
>     const data = await fetch(url);  // Sequentiel !
> }
>
> // BON : lancer toutes les requetes puis attendre
> const promises = urls.map(url => fetch(url));
> const resultats = await Promise.all(promises);
> ```

### Comparaison avec Python asyncio

```
Python asyncio :                    JavaScript async/await :
────────────────                    ────────────────────────
import asyncio                      // Natif, pas d'import

async def get_data():               async function getData() {
    response = await fetch(url)         const response = await fetch(url);
    return await response.json()        return await response.json();
                                    }

asyncio.run(get_data())             getData().then(console.log);

# Parallele :                       // Parallele :
await asyncio.gather(               await Promise.all([
    task1(),                            task1(),
    task2(),                            task2(),
    task3()                             task3()
)                                   ]);
```

> [!info] Difference cle avec Python
> En Python, vous avez besoin d'une **boucle evenementielle** explicite (`asyncio.run()`). En JavaScript, l'event loop tourne **toujours** en arriere-plan. Vous n'avez pas besoin de la demarrer.

---

## 6. L'API fetch

`fetch` est l'API moderne pour effectuer des requetes HTTP en JavaScript. Elle retourne une Promise.

### Requete GET (lire des donnees)

```javascript
// ─── GET simple ───
const response = await fetch("https://jsonplaceholder.typicode.com/posts");
const posts = await response.json();
console.log(posts);

// ─── Verifier le statut ───
async function fetchJSON(url) {
    const response = await fetch(url);

    if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return await response.json();
}
```

### Requete POST (envoyer des donnees)

```javascript
const nouvelArticle = {
    title: "Mon article",
    body: "Contenu de l'article",
    userId: 1
};

const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify(nouvelArticle)
});

const resultat = await response.json();
console.log("Cree :", resultat);
```

### Requetes PUT et DELETE

```javascript
// ─── PUT : mise a jour complete ───
const response = await fetch("https://api.example.com/posts/1", {
    method: "PUT",
    headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer mon-token-jwt"
    },
    body: JSON.stringify({
        title: "Titre modifie",
        body: "Nouveau contenu",
        userId: 1
    })
});

// ─── PATCH : mise a jour partielle ───
const response2 = await fetch("https://api.example.com/posts/1", {
    method: "PATCH",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ title: "Seulement le titre" })
});

// ─── DELETE : suppression ───
const response3 = await fetch("https://api.example.com/posts/1", {
    method: "DELETE"
});

if (response3.ok) {
    console.log("Supprime avec succes");
}
```

### L'objet Response

```javascript
const response = await fetch(url);

// ─── Proprietes ───
response.ok;          // true si status 200-299
response.status;      // 200, 404, 500, etc.
response.statusText;  // "OK", "Not Found", etc.
response.headers;     // Headers de la reponse
response.url;         // URL finale (apres redirections)
response.redirected;  // true si redirige

// ─── Methodes de lecture du corps (une seule fois !) ───
const json = await response.json();       // Parse JSON
const texte = await response.text();      // Texte brut
const blob = await response.blob();       // Fichier binaire
const formData = await response.formData(); // Donnees de formulaire
const buffer = await response.arrayBuffer(); // Buffer binaire
```

> [!warning] Le corps de la Response ne peut etre lu qu'UNE fois
> Apres avoir appele `.json()`, `.text()`, ou `.blob()`, le stream est consomme. Pour lire le corps plusieurs fois, clonez la reponse : `const clone = response.clone();`

### Gestion robuste des erreurs

```javascript
async function fetchAvecErreurs(url, options = {}) {
    try {
        const response = await fetch(url, options);

        // fetch ne rejette PAS sur les erreurs HTTP (404, 500, etc.)
        // Il ne rejette que sur les erreurs reseau (pas d'internet, DNS fail)
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }

        const contentType = response.headers.get("content-type");
        if (contentType && contentType.includes("application/json")) {
            return await response.json();
        } else {
            return await response.text();
        }

    } catch (erreur) {
        if (erreur instanceof TypeError) {
            // Erreur reseau (pas de connexion, CORS, etc.)
            console.error("Erreur reseau :", erreur.message);
        } else {
            // Erreur HTTP ou autre
            console.error("Erreur :", erreur.message);
        }
        throw erreur;  // Re-lancer pour que l'appelant puisse gerer
    }
}
```

> [!warning] fetch ne rejette PAS sur les erreurs HTTP !
> Contrairement a `axios` ou `XMLHttpRequest`, `fetch` considere qu'une reponse 404 ou 500 est un **succes** (la requete a abouti, le serveur a repondu). Vous devez verifier `response.ok` manuellement.

---

## 7. Exemple CRUD complet avec une API REST

```javascript
const API_URL = "https://jsonplaceholder.typicode.com/todos";

// ─── CREATE (POST) ───
async function creerTodo(titre) {
    const response = await fetch(API_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            title: titre,
            completed: false,
            userId: 1
        })
    });
    if (!response.ok) throw new Error(`Erreur: ${response.status}`);
    return await response.json();
}

// ─── READ (GET) ───
async function lireTodos(limite = 10) {
    const response = await fetch(`${API_URL}?_limit=${limite}`);
    if (!response.ok) throw new Error(`Erreur: ${response.status}`);
    return await response.json();
}

async function lireTodo(id) {
    const response = await fetch(`${API_URL}/${id}`);
    if (!response.ok) throw new Error(`Erreur: ${response.status}`);
    return await response.json();
}

// ─── UPDATE (PUT) ───
async function mettreAJourTodo(id, donnees) {
    const response = await fetch(`${API_URL}/${id}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(donnees)
    });
    if (!response.ok) throw new Error(`Erreur: ${response.status}`);
    return await response.json();
}

// ─── DELETE ───
async function supprimerTodo(id) {
    const response = await fetch(`${API_URL}/${id}`, {
        method: "DELETE"
    });
    if (!response.ok) throw new Error(`Erreur: ${response.status}`);
    return true;
}

// ─── Utilisation ───
async function demo() {
    try {
        // Creer
        const nouveau = await creerTodo("Apprendre async/await");
        console.log("Cree :", nouveau);

        // Lire
        const todos = await lireTodos(5);
        console.log("Liste :", todos);

        // Modifier
        const modifie = await mettreAJourTodo(1, {
            title: "Titre modifie",
            completed: true,
            userId: 1
        });
        console.log("Modifie :", modifie);

        // Supprimer
        await supprimerTodo(1);
        console.log("Supprime !");

    } catch (erreur) {
        console.error("Erreur CRUD :", erreur.message);
    }
}

demo();
```

---

## 8. XMLHttpRequest (legacy)

`XMLHttpRequest` (XHR) est l'ancienne methode pour effectuer des requetes HTTP. Vous la rencontrerez dans du code legacy.

```javascript
// ─── XMLHttpRequest : ancienne methode ───
function getDataXHR(url, callback) {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);

    xhr.onload = function() {
        if (xhr.status === 200) {
            const data = JSON.parse(xhr.responseText);
            callback(null, data);
        } else {
            callback(new Error(`HTTP ${xhr.status}`));
        }
    };

    xhr.onerror = function() {
        callback(new Error("Erreur reseau"));
    };

    xhr.send();
}

// ─── Equivalent moderne avec fetch ───
async function getData(url) {
    const response = await fetch(url);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
}
```

> [!info] Quand rencontrer XMLHttpRequest ?
> - Code ecrit avant 2015 (avant ES6 et la generalisation de fetch)
> - Upload de fichiers avec suivi de progression (`xhr.upload.onprogress`)
> - Certaines bibliotheques (jQuery.ajax utilise XHR en interne)
> Aujourd'hui, preferez `fetch` ou des bibliotheques comme `axios`.

---

## 9. JSON : parse et stringify

JSON (JavaScript Object Notation) est le format standard d'echange de donnees sur le web.

```javascript
// ─── Objet JS => String JSON ───
const obj = { nom: "Alice", age: 25, hobbies: ["lecture", "code"] };
const json = JSON.stringify(obj);
console.log(json);
// '{"nom":"Alice","age":25,"hobbies":["lecture","code"]}'

// ─── String JSON => Objet JS ───
const parsed = JSON.parse(json);
console.log(parsed.nom);  // "Alice"

// ─── Formatage lisible (pretty print) ───
const pretty = JSON.stringify(obj, null, 2);
console.log(pretty);
// {
//   "nom": "Alice",
//   "age": 25,
//   "hobbies": [
//     "lecture",
//     "code"
//   ]
// }

// ─── Filtrer les proprietes ───
const partiel = JSON.stringify(obj, ["nom", "age"]);
// '{"nom":"Alice","age":25}'

// ─── Transformer avec une fonction ───
const custom = JSON.stringify(obj, (cle, valeur) => {
    if (typeof valeur === "string") return valeur.toUpperCase();
    return valeur;
});
// '{"nom":"ALICE","age":25,"hobbies":["LECTURE","CODE"]}'
```

> [!warning] Limitations de JSON
> JSON ne supporte **pas** : `undefined`, `function`, `Symbol`, `NaN`, `Infinity`, les dates (converties en string), les references circulaires. `JSON.stringify` les ignore silencieusement ou lance une erreur (references circulaires).

---

## 10. Timers : setTimeout et setInterval

### setTimeout (executer une fois apres un delai)

```javascript
// ─── Executer apres 2 secondes ───
const timerId = setTimeout(() => {
    console.log("Execute apres 2 secondes");
}, 2000);

// ─── Annuler avant l'execution ───
clearTimeout(timerId);

// ─── setTimeout(fn, 0) : pas "immediatement" ! ───
setTimeout(() => console.log("B"), 0);
console.log("A");
// Sortie : A, B (setTimeout 0 passe par la macrotask queue)
```

### setInterval (executer periodiquement)

```javascript
// ─── Executer toutes les secondes ───
let compteur = 0;
const intervalId = setInterval(() => {
    compteur++;
    console.log(`Tick ${compteur}`);

    if (compteur >= 5) {
        clearInterval(intervalId);  // Arreter apres 5 ticks
        console.log("Intervalle arrete");
    }
}, 1000);
```

### Pattern : setTimeout recursif (plus precis que setInterval)

```javascript
// setInterval ne garantit pas un intervalle EXACT entre executions
// setTimeout recursif attend la fin de chaque execution

async function pollAPI() {
    try {
        const data = await fetch("/api/status").then(r => r.json());
        mettreAJour(data);
    } catch (e) {
        console.error("Erreur polling :", e);
    }

    // Replanifier APRES la fin de l'execution
    setTimeout(pollAPI, 5000);
}

pollAPI(); // Lancer le premier appel
```

### Comparaison

```
setInterval :  |──1s──|──1s──|──1s──|──1s──|
               exec   exec   exec   exec   exec
               (si exec prend du temps, les appels se chevauchent)

setTimeout recursif :  |──1s──|exec|──1s──|exec|──1s──|exec|
                       (toujours 1s APRES la fin de l'execution)
```

---

## 11. Comparaison Python asyncio vs JavaScript async

```
+------------------------+----------------------------+----------------------------+
| Aspect                 | Python (asyncio)           | JavaScript                 |
+------------------------+----------------------------+----------------------------+
| Event loop             | Explicite (asyncio.run)    | Implicite (toujours actif) |
| Syntaxe async          | async def / await          | async function / await     |
| Parallele              | asyncio.gather(...)        | Promise.all([...])         |
| Timeout                | asyncio.wait_for(coro, t)  | Promise.race + setTimeout  |
| HTTP client            | aiohttp (externe)          | fetch (natif)              |
| Callbacks              | Rares                      | Fondamentaux (DOM events)  |
| Sleep                  | await asyncio.sleep(1)     | await new Promise(r =>     |
|                        |                            |   setTimeout(r, 1000))     |
| Multithreading         | threading / multiprocessing| Web Workers                |
| Gestion erreurs        | try/except                 | try/catch                  |
+------------------------+----------------------------+----------------------------+
```

---

## Carte Mentale ASCII

```
                     JavaScript Asynchrone
                             │
       ┌─────────┬───────────┼───────────┬──────────────┐
       │         │           │           │              │
   Event Loop  Callbacks  Promises   async/await     fetch API
       │         │           │           │              │
   Call Stack  Pattern    new Promise  async function  GET
   Web APIs    Callback    resolve     await           POST
   Microtask Q  Hell      reject      try/catch       PUT/DELETE
   Macrotask Q           .then()      Promise.all     Headers
       │                 .catch()     parallele       Response
       │                 .finally()                   .json()
       │                     │                        .ok
       │              ┌──────┴──────┐
       │              │             │
       │          Promise.all   Promise.race
       │          allSettled    Promise.any
       │
   ┌───┴───────────┐
   │               │
  Timers          JSON
   │               │
  setTimeout     parse
  setInterval    stringify
  clearTimeout   format
  clearInterval  limitations
```

---

## Exercices

### Exercice 1 : Deviner l'ordre d'execution

Sans executer le code, predisez l'ordre de sortie, puis verifiez :

```javascript
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve()
    .then(() => console.log("C"))
    .then(() => console.log("D"));

setTimeout(() => console.log("E"), 0);

Promise.resolve().then(() => {
    console.log("F");
    setTimeout(() => console.log("G"), 0);
});

console.log("H");
```

### Exercice 2 : Wrapper fetch robuste

Creez une fonction `fetchRetry(url, options, maxRetries = 3)` qui :
- Effectue une requete fetch
- Si la requete echoue (erreur reseau ou status >= 500), reessaie jusqu'a `maxRetries` fois
- Attend un delai croissant entre chaque tentative (1s, 2s, 4s... = backoff exponentiel)
- Retourne les donnees JSON en cas de succes
- Lance une erreur apres epuisement des tentatives

### Exercice 3 : Chargeur d'images parallele

Creez une fonction `chargerImages(urls)` qui :
- Prend un tableau d'URLs d'images
- Charge toutes les images en parallele avec `Promise.all`
- Retourne un tableau de resultats `{ url, status: "ok" | "error", element? }`
- Utilisez `Promise.allSettled` pour ne pas echouer si une image est introuvable
- Affichez une barre de progression (pourcentage) au fur et a mesure

### Exercice 4 : Mini-application meteo

Creez une page qui :
- Demande a l'utilisateur de saisir une ville
- Utilise fetch pour appeler une API meteo (ex: wttr.in ou OpenWeatherMap)
- Affiche la temperature, la description et une icone
- Gere les erreurs (ville introuvable, probleme reseau)
- Affiche un indicateur de chargement pendant la requete
- Utilise async/await et try/catch

---

## Liens

- [[01 - Introduction a JavaScript]]
- [[02 - JavaScript DOM et Evenements]]
- [[04 - JavaScript Moderne ES6+]]
- [[07 - Python Async et Concurrence]]
