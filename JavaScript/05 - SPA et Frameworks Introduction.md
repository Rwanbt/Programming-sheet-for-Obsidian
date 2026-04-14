# SPA et Frameworks Introduction

Le web moderne a evolue d'un modele ou chaque interaction declenchait un rechargement complet de la page vers des applications fluides et reactives. Les Single Page Applications (SPA) et les frameworks JavaScript qui les soutiennent ont transforme la facon dont nous concevons et construisons des interfaces utilisateur.

Ce cours couvre la distinction fondamentale entre MPA et SPA, le routage cote client, le concept de composants, la gestion d'etat, le DOM virtuel, les trois grands frameworks (React, Vue, Svelte), les outils de build, et les considerations pratiques pour choisir sa stack.

> [!tip] Analogie
> Imaginez un restaurant. Dans un MPA, chaque fois que vous commandez un plat, vous devez quitter le restaurant, revenir, vous reinstaller et attendre qu'on vous serve. Dans un SPA, vous restez assis a votre table : le serveur (JavaScript) apporte et debarrasse les plats sans jamais vous demander de partir. Le menu (le HTML initial) est charge une seule fois, et tout le reste se passe a votre table (le navigateur).

---

## 1. Web Traditionnel (MPA) vs SPA

### Multi-Page Application (MPA)

Dans le modele classique, chaque page est un document HTML distinct genere par le serveur.

```
Utilisateur          Navigateur            Serveur
    │                    │                    │
    │── Clic lien ──────>│                    │
    │                    │── GET /about ──────>│
    │                    │                    │── Genere HTML
    │                    │<── HTML complet ───│
    │                    │── Parse + Render ──>│
    │<── Affichage ──────│                    │
    │                    │                    │
    │── Clic lien ──────>│                    │
    │                    │── GET /contact ────>│
    │                    │                    │── Genere HTML
    │                    │<── HTML complet ───│
    │                    │── Parse + Render ──>│
    │<── Ecran blanc     │                    │
    │<── Affichage ──────│                    │
```

Chaque navigation implique :
1. Une requete HTTP au serveur
2. Le serveur genere une page HTML complete
3. Le navigateur detruit la page actuelle
4. Le navigateur parse et affiche la nouvelle page
5. L'utilisateur voit un **ecran blanc** entre les pages

### Single Page Application (SPA)

Dans une SPA, une seule page HTML est chargee. JavaScript gere toute la navigation et le rendu.

```
Utilisateur          Navigateur (JS)        Serveur / API
    │                    │                    │
    │                    │── GET index.html ──>│
    │                    │<── HTML + JS ──────│   (chargement initial)
    │                    │── Parse + Execute ─>│
    │<── Affichage ──────│                    │
    │                    │                    │
    │── Clic lien ──────>│                    │
    │                    │── JS intercepte ──>│
    │                    │── MAJ URL (pushState)
    │                    │── Render composant │
    │                    │── GET /api/data ───>│   (donnees JSON)
    │                    │<── { ... } ────────│
    │<── Affichage ──────│                    │
    │                    │                    │
    │  (pas d'ecran blanc, transition fluide) │
```

> [!info] Precision importante
> Une SPA ne signifie pas "une seule vue". L'application peut avoir des dizaines de pages/vues differentes. Le terme "single page" fait reference au fait qu'un **seul document HTML** est charge depuis le serveur.

### Comparaison MPA vs SPA

```
+----------------------------+-----------------------------------+-----------------------------------+
| Critere                    | MPA (Multi-Page)                  | SPA (Single-Page)                 |
+----------------------------+-----------------------------------+-----------------------------------+
| Chargement initial         | Rapide (HTML leger)               | Plus lent (JS a telecharger)      |
| Navigations suivantes      | Lent (rechargement complet)       | Instantane (rendu local)          |
| Experience utilisateur     | Saccadee (ecrans blancs)          | Fluide (transitions douces)       |
| SEO                        | Excellent (HTML statique)         | Difficile (contenu dynamique)     |
| Complexite backend         | Elevee (genere les vues)          | Faible (API JSON)                 |
| Complexite frontend        | Faible (HTML/CSS/peu de JS)       | Elevee (framework, state, routing)|
| Consommation memoire       | Faible (nouvelle page = reset)    | Plus elevee (tout en memoire)     |
| Mode hors-ligne            | Impossible                        | Possible (Service Workers)        |
| Etat de l'application      | Perdu a chaque navigation         | Persiste entre les vues           |
| Temps de dev initial       | Court                             | Plus long                         |
| Exemples typiques          | WordPress, sites institutionnels  | Gmail, Trello, Spotify Web        |
+----------------------------+-----------------------------------+-----------------------------------+
```

> [!warning] SPA n'est pas toujours la reponse
> Un blog, un site vitrine ou un site a fort besoin SEO sera souvent mieux servi par un MPA ou une approche hybride (SSR/SSG). Les SPA brillent pour les applications interactives de type "dashboard" ou "outil".

---

## 2. Routage Cote Client

Dans une SPA, la navigation ne passe pas par le serveur. C'est JavaScript qui intercepte les changements d'URL et affiche le bon contenu.

### L'API History

```javascript
// ─── pushState : ajouter une entree dans l'historique ───
// Signature : history.pushState(state, title, url)
history.pushState({ page: "about" }, "", "/about");
// L'URL change dans la barre d'adresse
// MAIS aucune requete n'est envoyee au serveur !

// ─── replaceState : remplacer l'entree actuelle ───
history.replaceState({ page: "about-v2" }, "", "/about");
// Remplace au lieu d'ajouter

// ─── Naviguer dans l'historique ───
history.back();     // Equivalent au bouton retour
history.forward();  // Equivalent au bouton suivant
history.go(-2);     // Reculer de 2 pages
```

### Detecter la navigation (popstate)

```javascript
// ─── popstate : declenche quand l'utilisateur navigue (retour/avance) ───
window.addEventListener("popstate", (event) => {
    console.log("Navigation vers :", window.location.pathname);
    console.log("State :", event.state);
    // Afficher le bon contenu selon l'URL
    routerVers(window.location.pathname);
});
```

### Mini-routeur SPA

```javascript
// ─── Routeur minimaliste ───
class Router {
    constructor() {
        this.routes = {};
        window.addEventListener("popstate", () => this.resolve());
    }

    // Enregistrer une route
    on(chemin, callback) {
        this.routes[chemin] = callback;
        return this; // Chaining
    }

    // Naviguer vers un chemin
    naviguer(chemin) {
        history.pushState(null, "", chemin);
        this.resolve();
    }

    // Resoudre la route actuelle
    resolve() {
        const chemin = window.location.pathname;
        const handler = this.routes[chemin];
        if (handler) {
            handler();
        } else {
            this.routes["/404"]?.();
        }
    }
}

// ─── Utilisation ───
const router = new Router();

router
    .on("/", () => {
        document.getElementById("app").innerHTML = "<h1>Accueil</h1>";
    })
    .on("/about", () => {
        document.getElementById("app").innerHTML = "<h1>A propos</h1>";
    })
    .on("/404", () => {
        document.getElementById("app").innerHTML = "<h1>Page non trouvee</h1>";
    });

// Intercepter les clics sur les liens
document.addEventListener("click", (e) => {
    if (e.target.matches("a[data-spa]")) {
        e.preventDefault();
        router.naviguer(e.target.getAttribute("href"));
    }
});
```

```html
<!-- Les liens SPA utilisent un attribut data pour etre interceptes -->
<nav>
    <a href="/" data-spa>Accueil</a>
    <a href="/about" data-spa>A propos</a>
    <a href="/contact" data-spa>Contact</a>
</nav>
<div id="app"></div>
```

### Hash routing vs History routing

```
+-------------------+----------------------------------+----------------------------------+
| Aspect            | Hash (#)                         | History API                      |
+-------------------+----------------------------------+----------------------------------+
| URL               | example.com/#/about              | example.com/about                |
| Serveur           | Aucune config necessaire         | Doit rediriger vers index.html   |
| Esthetique        | Moins propre                     | URLs propres                     |
| SEO               | Mauvais (# ignore par moteurs)   | Meilleur                         |
| Compatibilite     | Tous les navigateurs             | Navigateurs modernes             |
| Evenement         | hashchange                       | popstate                         |
+-------------------+----------------------------------+----------------------------------+
```

> [!info] Configuration serveur pour History routing
> Avec le History API, si l'utilisateur rafraichit `/about`, le serveur doit retourner `index.html` (et non chercher un fichier `/about`). C'est le "fallback" SPA. Avec Nginx : `try_files $uri /index.html;`. Avec Apache : `.htaccess` RewriteRule.

---

## 3. Concept de Composants

### Qu'est-ce qu'un composant ?

Un composant est un **morceau reutilisable d'interface utilisateur** qui encapsule sa structure (HTML), son style (CSS) et son comportement (JS).

> [!tip] Analogie
> Les composants sont comme des **briques LEGO**. Chaque brique a une forme et une fonction specifiques. Vous assemblez des petites briques pour former des briques plus grandes, et ces grandes briques forment une construction complete. Un bouton est une brique. Un formulaire est un assemblage de briques (inputs, labels, boutons). Une page est un assemblage de formulaires, listes, en-tetes, etc.

```
┌──────────────────────────────────────────────────────┐
│  Application                                          │
│  ┌────────────────────────────────────────────────┐  │
│  │  Header                                         │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐     │  │
│  │  │   Logo   │  │  NavItem │  │  NavItem │     │  │
│  │  └──────────┘  └──────────┘  └──────────┘     │  │
│  └────────────────────────────────────────────────┘  │
│  ┌──────────────────────┐  ┌──────────────────────┐  │
│  │  Sidebar             │  │  MainContent          │  │
│  │  ┌────────────────┐  │  │  ┌────────────────┐  │  │
│  │  │  FilterGroup   │  │  │  │  Card          │  │  │
│  │  │  ┌──────────┐  │  │  │  │  ┌──────────┐ │  │  │
│  │  │  │ Checkbox │  │  │  │  │  │  Button  │ │  │  │
│  │  │  └──────────┘  │  │  │  │  └──────────┘ │  │  │
│  │  └────────────────┘  │  │  └────────────────┘  │  │
│  └──────────────────────┘  └──────────────────────┘  │
└──────────────────────────────────────────────────────┘
```

### Principes des composants

```javascript
// ─── Un composant en pseudo-code ───
// Composant = fonction(donnees) => interface
function Bouton(props) {
    // props = donnees d'entree (comme des parametres)
    return `
        <button class="btn btn--${props.variante}" onclick="${props.onClick}">
            ${props.texte}
        </button>
    `;
}

// Reutilisable avec differentes donnees
Bouton({ texte: "Valider", variante: "primary", onClick: "soumettre()" });
Bouton({ texte: "Annuler", variante: "secondary", onClick: "annuler()" });
Bouton({ texte: "Supprimer", variante: "danger", onClick: "supprimer()" });
```

### Composition vs Heritage

```
Approche privilegiee dans les frameworks modernes : COMPOSITION

Heritage (eviter) :              Composition (preferer) :
─────────────────               ────────────────────────
BaseComponent                    Page
  └── PageComponent                ├── Header
        └── DashboardPage          │     ├── Logo
              └── AdminDashboard   │     └── Navigation
                                   ├── Sidebar
                                   │     └── FilterList
                                   └── Content
                                         └── CardGrid
                                               └── Card (x N)
```

---

## 4. Gestion d'Etat (State Management)

### Qu'est-ce que l'etat ?

L'etat (state) est l'ensemble des **donnees dynamiques** qui determinent ce que l'utilisateur voit et comment l'application se comporte a un instant T.

```javascript
// Exemples d'etat dans une application
const etat = {
    // Etat de l'interface
    menuOuvert: false,
    themeActuel: "sombre",
    ongletActif: "profil",

    // Etat des donnees
    utilisateur: { nom: "Alice", email: "alice@mail.com" },
    articles: [/* ... */],
    panier: [/* ... */],

    // Etat du formulaire
    champRecherche: "JavaScript",
    filtresActifs: ["recent", "populaire"],

    // Etat reseau
    chargement: true,
    erreur: null,
};
```

### Etat local vs global

```
┌────────────────────────────────────────────────────────┐
│                    ETAT GLOBAL                          │
│   (utilisateur connecte, theme, langue, panier)        │
│                                                         │
│   ┌───────────────┐  ┌───────────────┐                 │
│   │ Composant A   │  │ Composant B   │                 │
│   │               │  │               │                 │
│   │ Etat local :  │  │ Etat local :  │                 │
│   │ - menuOuvert  │  │ - ongletActif │                 │
│   │ - tooltip     │  │ - formulaire  │                 │
│   │               │  │               │                 │
│   │ ┌───────────┐ │  │ ┌───────────┐ │                 │
│   │ │ Enfant A1 │ │  │ │ Enfant B1 │ │                 │
│   │ │ Etat :    │ │  │ │ Etat :    │ │                 │
│   │ │ - hover   │ │  │ │ - filtre  │ │                 │
│   │ └───────────┘ │  │ └───────────┘ │                 │
│   └───────────────┘  └───────────────┘                 │
└────────────────────────────────────────────────────────┘

Etat LOCAL = propre a un composant (menu ouvert, input en cours)
Etat GLOBAL = partage entre plusieurs composants (utilisateur, theme)
```

### Pourquoi la gestion d'etat devient complexe

```
Probleme du "prop drilling" :         Solution : store centralise
──────────────────────────           ──────────────────────────

     App (state: user)                      ┌───────────┐
      │                                     │   Store   │
      ├── Header (props: user)              │  (global) │
      │    └── Avatar (props: user)         └─────┬─────┘
      │         └── Initiales (user.nom)      ┌───┼───┐
      │                                       │   │   │
      ├── Sidebar (props: user)             App Header Sidebar
      │    └── Menu (props: user.role)          │
      │                                       Avatar
      └── Content (props: user)
           └── Profile (props: user)      Chaque composant accede
                                          directement au store
L'etat doit traverser TOUS                 sans "passer" les props
les composants intermediaires              a travers toute la hierarchie
```

> [!warning] Ne pas tout globaliser
> Ce n'est pas parce qu'un store existe que tout doit y aller. L'etat d'un menu dropdown, d'un champ de formulaire en cours de saisie, ou d'un tooltip doit rester local. Globaliser inutilement rend le debug plus difficile.

---

## 5. Le DOM Virtuel

### Le probleme du DOM reel

```javascript
// ─── Manipuler le DOM reel est LENT ───
// Chaque modification declenche potentiellement :
// 1. Recalcul des styles (style recalculation)
// 2. Recalcul du layout (reflow)
// 3. Repeinture (repaint)

// Mauvaise approche : 100 modifications = 100 reflows potentiels
for (let i = 0; i < 100; i++) {
    const li = document.createElement("li");
    li.textContent = `Item ${i}`;
    liste.appendChild(li);  // Reflow a chaque ajout !
}

// Meilleure approche (sans framework) : DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
    const li = document.createElement("li");
    li.textContent = `Item ${i}`;
    fragment.appendChild(li);  // Pas de reflow
}
liste.appendChild(fragment);   // Un seul reflow
```

### Le concept de DOM virtuel (VDOM)

```
Etat change            DOM Virtuel              DOM Reel
──────────            ──────────────           ──────────

1. Nouveau state  ──> 2. Cree un nouveau      4. Applique SEULEMENT
                         VDOM (arbre JS)          les changements
                                                   necessaires
                      3. Compare (diff)
                         ancien VDOM vs
                         nouveau VDOM
                         (algorithme de
                          reconciliation)

                         Ancien:          Nouveau:
                         <ul>             <ul>
                           <li>A</li>       <li>A</li>
                           <li>B</li>       <li>B</li>
                           <li>C</li>       <li>C</li>  (inchange)
                         </ul>              <li>D</li>  (ajoute !)
                                          </ul>

                         Diff = "ajouter <li>D</li>"
```

```javascript
// ─── Representation simplifiee d'un VDOM ───
// Le VDOM est un arbre d'objets JavaScript simples

// Noeud VDOM
function creerVNode(type, props, ...enfants) {
    return { type, props: props || {}, enfants };
}

// Exemple : representer <div class="card"><h2>Titre</h2><p>Texte</p></div>
const vdom = creerVNode("div", { class: "card" },
    creerVNode("h2", null, "Titre"),
    creerVNode("p", null, "Texte")
);

// Resultat : un objet JS simple (rapide a creer et comparer)
// {
//     type: "div",
//     props: { class: "card" },
//     enfants: [
//         { type: "h2", props: {}, enfants: ["Titre"] },
//         { type: "p", props: {}, enfants: ["Texte"] }
//     ]
// }
```

### Algorithme de diffing (simplifie)

```javascript
// ─── Diff simplifie ───
function diff(ancienVNode, nouveauVNode) {
    // Cas 1 : noeud supprime
    if (!nouveauVNode) return { type: "REMOVE" };

    // Cas 2 : noeud ajoute
    if (!ancienVNode) return { type: "ADD", vnode: nouveauVNode };

    // Cas 3 : type different => remplacer entierement
    if (ancienVNode.type !== nouveauVNode.type) {
        return { type: "REPLACE", vnode: nouveauVNode };
    }

    // Cas 4 : texte different
    if (typeof nouveauVNode === "string") {
        if (ancienVNode !== nouveauVNode) {
            return { type: "TEXT", texte: nouveauVNode };
        }
        return null; // Pas de changement
    }

    // Cas 5 : meme type => comparer les props et enfants
    return {
        type: "UPDATE",
        props: diffProps(ancienVNode.props, nouveauVNode.props),
        enfants: diffEnfants(ancienVNode.enfants, nouveauVNode.enfants)
    };
}
```

> [!info] React et Vue utilisent le VDOM, Svelte non
> Svelte adopte une approche differente : il compile les composants en JavaScript imperatif qui manipule directement le DOM reel. Il n'y a donc pas de surcharge liee au VDOM. Chaque approche a ses avantages.

---

## 6. React

### Presentation

React est une bibliotheque (pas un framework complet) creee par Facebook (Meta) en 2013. Elle se concentre sur la couche vue (UI) et utilise un DOM virtuel.

### JSX : HTML dans JavaScript

```jsx
// ─── JSX = syntaxe qui melange HTML et JavaScript ───
// Ce n'est pas du HTML : c'est du JavaScript transforme par un compilateur

// JSX :
const element = <h1 className="titre">Bonjour, {nom}!</h1>;

// Compile en :
const element = React.createElement("h1", { className: "titre" }, "Bonjour, ", nom, "!");

// ─── Expressions dans JSX ───
const liste = (
    <ul>
        {items.map(item => (
            <li key={item.id}>{item.nom}</li>
        ))}
    </ul>
);

// ─── Rendu conditionnel ───
const message = (
    <div>
        {estConnecte ? <p>Bienvenue !</p> : <p>Connectez-vous</p>}
        {erreur && <p className="erreur">{erreur}</p>}
    </div>
);
```

### Composants fonctionnels

```jsx
// ─── Composant = fonction qui retourne du JSX ───
function Salutation({ nom, age }) {
    return (
        <div className="salutation">
            <h2>Bonjour, {nom} !</h2>
            <p>Vous avez {age} ans.</p>
        </div>
    );
}

// Utilisation (comme un tag HTML)
<Salutation nom="Alice" age={25} />
```

### Hooks : useState et useEffect

```jsx
import { useState, useEffect } from "react";

function Compteur() {
    // ─── useState : gerer l'etat local ───
    // Retourne [valeurActuelle, fonctionDeModification]
    const [compte, setCompte] = useState(0);
    const [nom, setNom] = useState("Alice");

    // ─── useEffect : effets de bord (side effects) ───
    // S'execute apres le rendu
    useEffect(() => {
        // Mise a jour du titre de la page
        document.title = `Compteur : ${compte}`;

        // Fonction de nettoyage (optionnelle)
        return () => {
            document.title = "React App";
        };
    }, [compte]); // Ne re-execute que si 'compte' change

    // useEffect avec tableau vide = componentDidMount
    useEffect(() => {
        console.log("Composant monte !");
        return () => console.log("Composant demonte !");
    }, []);

    return (
        <div>
            <p>{nom} : {compte}</p>
            <button onClick={() => setCompte(compte + 1)}>+1</button>
            <button onClick={() => setCompte(c => c - 1)}>-1</button>
            <button onClick={() => setCompte(0)}>Reset</button>
        </div>
    );
}
```

### Ecosysteme React

```
React (UI)
  ├── React Router ─── Routage SPA
  ├── Redux / Zustand ─── Gestion d'etat global
  ├── React Query (TanStack) ─── Gestion des requetes API
  ├── Next.js ─── Framework full-stack (SSR, SSG, API routes)
  ├── Styled Components / Tailwind ─── Styling
  ├── React Hook Form ─── Formulaires
  └── Framer Motion ─── Animations
```

---

## 7. Vue.js

### Presentation

Vue.js est un framework progressif cree par Evan You en 2014. Il est concu pour etre adoptable incrementalement : on peut l'utiliser comme une simple bibliotheque ou comme un framework complet.

### Syntaxe template (Options API)

```html
<template>
    <!-- Le template utilise une syntaxe declarative -->
    <div class="compteur">
        <h2>{{ titre }}</h2>
        <p>Compteur : {{ compte }}</p>
        <p>Double : {{ double }}</p>

        <button @click="incrementer">+1</button>
        <button @click="compte--">-1</button>

        <!-- v-if : rendu conditionnel -->
        <p v-if="compte > 10" class="alerte">C'est beaucoup !</p>
        <p v-else-if="compte > 5">Ca monte...</p>
        <p v-else>Encore petit</p>

        <!-- v-for : boucle -->
        <ul>
            <li v-for="item in items" :key="item.id">
                {{ item.nom }}
            </li>
        </ul>

        <!-- v-model : binding bidirectionnel -->
        <input v-model="recherche" placeholder="Rechercher..." />
    </div>
</template>

<script>
export default {
    data() {
        return {
            titre: "Mon Compteur",
            compte: 0,
            recherche: "",
            items: [
                { id: 1, nom: "Item A" },
                { id: 2, nom: "Item B" }
            ]
        };
    },
    computed: {
        // Proprietes calculees (mises en cache)
        double() {
            return this.compte * 2;
        }
    },
    watch: {
        // Observer les changements d'une donnee
        compte(nouvelleValeur, ancienneValeur) {
            console.log(`Compteur : ${ancienneValeur} -> ${nouvelleValeur}`);
        }
    },
    methods: {
        incrementer() {
            this.compte++;
        }
    },
    mounted() {
        console.log("Composant monte !");
    }
};
</script>
```

### Composition API (Vue 3)

```html
<script setup>
import { ref, computed, watch, onMounted } from "vue";

// ref() = equivalent de useState en React
const compte = ref(0);
const recherche = ref("");

// computed = propriete calculee reactive
const double = computed(() => compte.value * 2);

// watch = observer un changement
watch(compte, (nouveau, ancien) => {
    console.log(`${ancien} -> ${nouveau}`);
});

// Cycle de vie
onMounted(() => {
    console.log("Monte !");
});

function incrementer() {
    compte.value++;
}
</script>

<template>
    <p>{{ compte }} (double : {{ double }})</p>
    <button @click="incrementer">+1</button>
    <input v-model="recherche" />
</template>
```

### Ecosysteme Vue

```
Vue (UI)
  ├── Vue Router ─── Routage SPA
  ├── Pinia (ex-Vuex) ─── Gestion d'etat global
  ├── Nuxt.js ─── Framework full-stack (SSR, SSG)
  ├── Vuetify / PrimeVue ─── Bibliotheques de composants
  ├── VueUse ─── Collection de composables utilitaires
  └── Vite ─── Outil de build (cree par Evan You)
```

---

## 8. Svelte

### Presentation

Svelte est un compilateur cree par Rich Harris en 2016. Contrairement a React et Vue, Svelte deplace le travail du **navigateur** vers l'etape de **compilation**. Le resultat est du JavaScript imperatif pur, sans runtime framework.

### Syntaxe reactive

```svelte
<script>
    // ─── Variables reactives (pas besoin de useState ou ref) ───
    let compte = 0;
    let nom = "Alice";
    let items = [
        { id: 1, texte: "Apprendre Svelte" },
        { id: 2, texte: "Creer un projet" }
    ];

    // ─── Declarations reactives ($:) ───
    // Recalcule automatiquement quand 'compte' change
    $: double = compte * 2;
    $: estGrand = compte > 10;
    $: console.log("Le compteur vaut :", compte);  // Effet de bord reactif

    // ─── Fonctions ───
    function incrementer() {
        compte++;
    }

    function ajouterItem() {
        items = [...items, { id: items.length + 1, texte: "Nouveau" }];
        // IMPORTANT : items = [...] et non items.push()
        // Svelte detecte les changements par REASSIGNATION
    }
</script>

<!-- Rendu conditionnel -->
{#if estGrand}
    <p>C'est beaucoup !</p>
{:else if compte > 5}
    <p>Ca monte...</p>
{:else}
    <p>Encore petit</p>
{/if}

<!-- Boucle -->
{#each items as item (item.id)}
    <li>{item.texte}</li>
{/each}

<!-- Binding bidirectionnel -->
<input bind:value={nom} />

<p>{nom} : {compte} (double : {double})</p>
<button on:click={incrementer}>+1</button>
```

### Compile-time : pas de virtual DOM

```
React / Vue :                          Svelte :
──────────                             ───────
1. Code source                         1. Code source (.svelte)
2. Bundle inclut le framework          2. Compilateur Svelte
   (React ~40KB, Vue ~30KB)            3. JS imperatif pur
3. Au runtime :                            (pas de framework inclus)
   - Cree le VDOM                      4. Au runtime :
   - Compare ancien/nouveau VDOM          - MAJ directe du DOM
   - Calcule les differences              (element.textContent = val)
   - Applique au DOM reel

Avantage : flexibilite                 Avantage : performance,
Inconvenient : surcharge runtime       bundle tres petit (~5KB)
                                       Inconvenient : ecosysteme
                                       plus petit
```

### Ecosysteme Svelte

```
Svelte (UI + compilateur)
  ├── SvelteKit ─── Framework full-stack (SSR, SSG, API)
  ├── Svelte Store ─── Gestion d'etat (integre)
  ├── Svelte Transition ─── Animations (integre)
  └── Svelte Action ─── Directives DOM (integre)
```

> [!tip] Analogie
> Si React et Vue sont des **interpretes** qui traduisent vos instructions en temps reel, Svelte est un **compilateur** qui traduit tout a l'avance. Le resultat : un "programme" optimise qui s'execute directement sans intermediaire. C'est un peu la difference entre Python (interprete) et C (compile).

---

## 9. Comparaison des Frameworks

```
+---------------------+------------------+------------------+------------------+
| Critere             | React            | Vue              | Svelte           |
+---------------------+------------------+------------------+------------------+
| Cree par            | Meta (Facebook)  | Evan You         | Rich Harris      |
| Annee               | 2013             | 2014             | 2016             |
| Type                | Bibliotheque     | Framework prog.  | Compilateur      |
| Langage template    | JSX              | HTML + directives| HTML + directives|
| Reactivite          | Hooks (useState) | ref() / reactive | Assignation (=)  |
| DOM Virtuel         | Oui              | Oui              | Non              |
| Taille bundle       | ~40 KB (runtime) | ~30 KB (runtime) | ~2-5 KB (compile)|
| Courbe apprent.     | Moyenne          | Douce            | Douce            |
| Marche emploi       | +++ (dominant)   | ++ (fort)        | + (croissant)    |
| Ecosysteme          | Tres riche       | Riche            | En croissance    |
| Full-stack          | Next.js          | Nuxt.js          | SvelteKit        |
| TypeScript          | Excellent        | Excellent        | Bon              |
| Mobile              | React Native     | Capacitor/Ionic  | Capacitor        |
| Entreprises         | Meta, Netflix,   | Alibaba, GitLab, | NY Times,        |
|                     | Airbnb, Discord  | Apple, Nintendo  | Spotify (partiel)|
+---------------------+------------------+------------------+------------------+
```

### Syntaxe comparee : un meme composant

```
React (JSX + Hooks) :              Vue 3 (Composition API) :
──────────────────────             ────────────────────────
function Compteur() {              <script setup>
  const [n, setN] =                import { ref } from "vue";
    useState(0);                   const n = ref(0);
  return (                         </script>
    <div>
      <p>{n}</p>                   <template>
      <button                        <p>{{ n }}</p>
        onClick={() =>               <button @click="n++">
          setN(n + 1)}>                +1
        +1                           </button>
      </button>                    </template>
    </div>
  );
}

Svelte :
────────
<script>
  let n = 0;
</script>

<p>{n}</p>
<button on:click={() => n++}>
  +1
</button>
```

> [!example] Le meme resultat, 3 philosophies
> React privilegie "tout est JavaScript" (JSX). Vue separe template et logique dans un meme fichier (.vue). Svelte utilise une syntaxe proche du HTML natif avec des superpouvoir reactifs. Aucun n'est objectivement "meilleur" ; le choix depend du contexte.

---

## 10. Outils de Build

### Pourquoi un outil de build ?

Les navigateurs ne comprennent pas nativement JSX, les fichiers `.vue`, `.svelte`, TypeScript, ni les `import` depuis `node_modules`. Un outil de build transforme le code source en fichiers que le navigateur comprend.

```
Code source                  Build tool              Navigateur
───────────                  ──────────              ──────────
src/
  App.jsx          ─┐
  Header.vue        │      ┌──────────┐           dist/
  utils.ts          ├─────>│  Vite /  │──────>      index.html
  styles.scss       │      │ Webpack  │              bundle.js (optimise)
  images/logo.svg  ─┘      └──────────┘              styles.css
                                                      assets/
                           Transformations :
                           - JSX => JS
                           - TypeScript => JS
                           - SCSS => CSS
                           - Minification
                           - Tree shaking
                           - Code splitting
                           - Optimisation images
```

### Vite vs Webpack

```
+---------------------+----------------------------------+----------------------------------+
| Aspect              | Vite                             | Webpack                          |
+---------------------+----------------------------------+----------------------------------+
| Demarrage dev       | Quasi-instantane (ESM natif)     | Lent (bundle tout d'abord)       |
| HMR (Hot Reload)    | Tres rapide                      | Plus lent                        |
| Configuration       | Minimale (convention > config)   | Verbose (webpack.config.js)      |
| Build production    | Rollup (rapide)                  | Webpack (complet)                |
| Ecosysteme plugins  | Croissant                        | Tres riche                       |
| Utilise par         | Vue, Svelte, React (recents)     | Projets existants, Create React  |
| Cree par            | Evan You (createur de Vue)       | Tobias Koppers                   |
+---------------------+----------------------------------+----------------------------------+
```

```bash
# ─── Creer un projet avec Vite ───
npm create vite@latest mon-projet -- --template react
npm create vite@latest mon-projet -- --template vue
npm create vite@latest mon-projet -- --template svelte

cd mon-projet
npm install
npm run dev      # Serveur de developpement (http://localhost:5173)
npm run build    # Build de production (dist/)
npm run preview  # Previsualiser le build
```

---

## 11. npm et package.json

### npm : le gestionnaire de paquets

```bash
# ─── Commandes essentielles ───
npm init -y                    # Creer un package.json
npm install react react-dom    # Installer des dependances (production)
npm install -D vitest          # Installer en dev seulement
npm install                    # Installer tout depuis package.json
npm uninstall react            # Desinstaller
npm update                     # Mettre a jour
npm audit                      # Verifier les vulnerabilites
npm run dev                    # Lancer un script defini dans package.json
npx create-react-app my-app   # Executer un paquet sans l'installer
```

### package.json : le coeur du projet

```json
{
    "name": "mon-projet",
    "version": "1.0.0",
    "description": "Mon application SPA",
    "type": "module",
    "scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview",
        "test": "vitest",
        "lint": "eslint src/"
    },
    "dependencies": {
        "react": "^18.2.0",
        "react-dom": "^18.2.0",
        "react-router-dom": "^6.20.0"
    },
    "devDependencies": {
        "@vitejs/plugin-react": "^4.2.0",
        "vite": "^5.0.0",
        "vitest": "^1.0.0",
        "eslint": "^8.50.0"
    }
}
```

```
node_modules/     ← Dependances installees (NE PAS commiter)
├── react/
├── react-dom/
├── vite/
└── ... (des centaines de paquets)

package-lock.json ← Versions exactes (A COMMITER)
.gitignore        ← Doit contenir "node_modules"
```

> [!warning] node_modules
> Le dossier `node_modules` peut contenir des milliers de fichiers et peser des centaines de Mo. Il ne faut **jamais** le commiter dans git. Le `package-lock.json` (ou `yarn.lock`) garantit que `npm install` reinstallera les memes versions exactes.

### npm vs yarn vs pnpm

```
+------------------+-------------------+--------------------+--------------------+
| Aspect           | npm               | yarn               | pnpm               |
+------------------+-------------------+--------------------+--------------------+
| Inclus avec      | Node.js           | Installation sep.  | Installation sep.  |
| Vitesse           | Bonne             | Bonne              | Tres rapide        |
| Espace disque    | Copie tout        | Copie tout         | Liens symboliques  |
| Lock file        | package-lock.json | yarn.lock          | pnpm-lock.yaml     |
| Workspaces       | Oui               | Oui (pionnier)     | Oui                |
+------------------+-------------------+--------------------+--------------------+
```

---

## 12. Choisir un Framework

### Arbre de decision

```
                    Quel framework choisir ?
                            │
                 ┌──────────┴───────────┐
                 │                      │
         Projet pro / equipe     Projet perso / apprentissage
                 │                      │
         ┌───────┴───────┐        ┌─────┴──────┐
         │               │        │            │
    Equipe existante  Nouveau     Debutant   Experimente
         │            projet      web         JS
         │               │        │            │
    Utiliser ce       ┌───┴───┐   Vue         Svelte
    qu'ils             │       │  (douce      (innovant,
    connaissent     Beaucoup  Perf courbe)    performant)
                    de libs   critique?
                    tierces?     │
                       │     ┌───┴───┐
                    React    │       │
                    (ecosys  Oui    Non
                    riche)    │      │
                           Svelte  React ou Vue
                                   (selon pref)
```

> [!tip] Analogie
> Choisir un framework, c'est comme choisir une voiture. React est une Toyota : fiable, enorme reseau de garages (ecosysteme), tout le monde connait. Vue est une Mazda : plaisir de conduite, bien concu, un peu moins repandu. Svelte est une Tesla : innovant, performant, mais le reseau de recharge (ecosysteme) est encore en developpement.

---

## 13. Preoccupations des SPA

### SEO (Search Engine Optimization)

```
Probleme :
──────────
Moteur de recherche ──> GET /page ──> Recoit index.html VIDE
                                       <div id="app"></div>
                                       Le JS n'est pas execute !
                                       Le moteur ne voit pas le contenu.

Solutions :
───────────
1. SSR (Server-Side Rendering) ── Le serveur execute le JS et renvoie du HTML
   Outils : Next.js (React), Nuxt.js (Vue), SvelteKit

2. SSG (Static Site Generation) ── Pages pre-generees au build
   Outils : Next.js, Nuxt.js, Astro

3. Pre-rendering ── Un service genere le HTML pour les bots
   Outils : Prerender.io, rendertron

4. Hydration ── Le serveur envoie du HTML statique,
                puis le JS "hydrate" (active) la page
```

### Temps de chargement initial

```
MPA : HTML (5 KB) ─────────────────> Affichage

SPA : HTML (1 KB) + JS (200+ KB) ──> Parse JS ──> Execute ──> Affichage
      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^                 ^^^^^^^^^^^
      Temps de telechargement                      Temps d'execution
      (reseau)                                     (CPU)

Optimisations :
- Code splitting (charger le JS page par page)
- Lazy loading (import() dynamique)
- Tree shaking (eliminer le code mort)
- Compression (gzip, brotli)
- CDN pour les assets
```

### Accessibilite (a11y)

```
Problemes courants des SPA :
- Navigation sans rechargement = les lecteurs d'ecran ne detectent pas
  le changement de page
- Le focus n'est pas deplace apres navigation
- Les annonces ARIA manquantes

Solutions :
- Utiliser aria-live pour annoncer les changements
- Gerer le focus apres chaque navigation
- Utiliser les elements semantiques (<main>, <nav>, <header>)
- Tester avec un lecteur d'ecran (NVDA, VoiceOver)
```

> [!warning] L'accessibilite n'est pas optionnelle
> En France, le RGAA (Referentiel General d'Amelioration de l'Accessibilite) impose des normes legales pour les sites publics. Meme pour les sites prives, une bonne accessibilite ameliore l'experience pour tous les utilisateurs.

---

## Carte Mentale ASCII

```
                           SPA & Frameworks
                                 │
          ┌──────────┬───────────┼──────────┬──────────────┐
          │          │           │          │              │
      MPA vs SPA  Routage    Composants  State         Frameworks
          │        Client        │       Management       │
     Rechargement  │         Reutilisable    │      ┌─────┼─────┐
     vs fluide   History API  Props        Local    │     │     │
     SEO vs UX   pushState    Composition  vs      React  Vue  Svelte
                 popstate     Arbre        Global    │     │     │
                 Hash vs                   Store   JSX  Template Compile
                 History                   Redux   Hooks  ref()   $:
                                           Pinia   Next  Nuxt  SvelteKit
          ┌──────────┐                               │
          │          │                          Outils Build
      Virtual DOM  Diffing                          │
      Arbre JS     Reconciliation             ┌─────┼─────┐
      React/Vue    Minimal MAJ               Vite Webpack  npm
                                              ESM  Bundle  package.json
          ┌──────────┐
          │          │
        SEO      Performance
        SSR/SSG  Code splitting
        Hydration Lazy loading
```

---

## Exercices

### Exercice 1 : Mini-routeur SPA

Creez un routeur SPA complet en vanilla JavaScript :
- Support des routes statiques (`/about`) et dynamiques (`/user/:id`)
- Extraction des parametres d'URL (`/user/42` → `{ id: "42" }`)
- Support des query strings (`?page=2&sort=nom`)
- Middleware (ex: verifier si l'utilisateur est connecte avant d'acceder a `/admin`)
- Transitions CSS entre les vues (fade ou slide)

### Exercice 2 : Framework minimaliste

Construisez un micro-framework reactif (~100 lignes) :
- Fonction `creerEtat(initial)` qui retourne `[getter, setter]` et declenche un re-render
- Fonction `composant(template, state)` qui re-render quand l'etat change
- Systeme de props pour passer des donnees parent → enfant
- Support du `if` et du `each` dans les templates (avec des commentaires marqueurs)

### Exercice 3 : Comparatif pratique React / Vue / Svelte

Installez les 3 frameworks et creez la meme mini-application (liste de contacts) dans chacun :
- Afficher une liste de contacts (nom, email, telephone)
- Ajouter / supprimer un contact
- Recherche en temps reel
- Comparez : lignes de code, taille du bundle (`npm run build`), ressenti developpeur

---

## Liens

- [[04 - JavaScript Moderne ES6+]]
- [[06 - Projet JavaScript Interactif]]
- [[02 - JavaScript DOM et Evenements]]
- [[03 - JavaScript Asynchrone]]
