# Introduction a React

React est une bibliotheque JavaScript cree par Meta (Facebook) en 2013 pour construire des interfaces utilisateur. Ce n'est pas un framework complet : React se concentre exclusivement sur la couche vue (UI) et laisse les autres decisions (routage, gestion d'etat, requetes reseau) a d'autres bibliotheques. C'est ce qui lui confere une flexibilite remarquable et une longevite exceptionnelle.

Ce cours introduit les fondations de React : le Virtual DOM, JSX, les composants fonctionnels, les props, le rendu conditionnel, les listes, les evenements, et la mise en place d'un projet avec Vite. Il est le point de depart de la serie et fait suite a [[05 - SPA et Frameworks Introduction]].

> [!tip] Analogie
> Imaginez une interface utilisateur comme un arbre genealogique. Chaque noeud est un composant : il peut avoir des enfants (sous-composants) et recevoir des informations de son parent (props). React gere cet arbre et ne met a jour que les noeuds qui ont change — comme un document Word qui ne reimprime que les pages modifiees, pas le livre entier.

---

## 1. Pourquoi React ?

### Le probleme que React resout

Avant React, manipuler le DOM directement avec JavaScript devenait rapidement un cauchemar sur des interfaces complexes.

```javascript
// ─── Sans React : synchronisation manuelle etat/DOM ───
let utilisateur = { nom: "Alice", connecte: true };

// On doit manuellement mettre a jour CHAQUE element du DOM
function mettreAJourUI() {
    document.getElementById("nom").textContent = utilisateur.nom;
    document.getElementById("status").textContent = utilisateur.connecte
        ? "Connecte"
        : "Deconnecte";
    document.getElementById("avatar").className = utilisateur.connecte
        ? "avatar avatar--online"
        : "avatar avatar--offline";
    // ... et des dizaines d'autres elements
}

// Chaque modification d'etat necessite un appel manuel
utilisateur.nom = "Bob";
mettreAJourUI(); // Oublie = bug silencieux
```

React resout ce probleme en declarant **ce que l'interface doit ressembler** selon l'etat, pas **comment la modifier**.

```jsx
// ─── Avec React : declaratif, synchronisation automatique ───
function Profil({ utilisateur }) {
    return (
        <div>
            <span id="nom">{utilisateur.nom}</span>
            <span className={`avatar avatar--${utilisateur.connecte ? "online" : "offline"}`}>
                {utilisateur.connecte ? "Connecte" : "Deconnecte"}
            </span>
        </div>
    );
}
// Quand utilisateur change, React recalcule et met a jour le DOM automatiquement
```

### Caracteristiques cles de React

```
┌─────────────────────────────────────────────────────────┐
│  LES 4 PILIERS DE REACT                                  │
│                                                          │
│  1. DECLARATIF                                           │
│     Vous decrivez QUOI afficher.                         │
│     React decide COMMENT mettre a jour le DOM.          │
│                                                          │
│  2. COMPOSANTS                                           │
│     L'interface = assemblage de blocs reutilisables.     │
│     Chaque composant gere sa logique et son affichage.  │
│                                                          │
│  3. UNIDIRECTIONNEL                                      │
│     Les donnees coulent de parent vers enfant (props).  │
│     Pas de binding bidirectionnel implicite.            │
│                                                          │
│  4. VIRTUAL DOM                                          │
│     React calcule les differences en memoire.           │
│     N'applique que les changements necessaires au DOM.  │
└─────────────────────────────────────────────────────────┘
```

---

## 2. MPA vs SPA — Rappel contextuel

React est concu pour les SPA (Single Page Applications). Pour comprendre l'interet, voyons la difference fondamentale avec les MPA.

### Multi-Page Application (MPA)

```
Utilisateur           Navigateur             Serveur
    │                     │                     │
    │── Clic "Profil" ──>│                     │
    │                     │── GET /profil ─────>│
    │                     │                     │── Genere HTML complet
    │                     │<── HTML (40 KB) ────│
    │                     │── Detruit la page   │
    │                     │── Parse le nouveau  │
    │<── [ecran blanc]    │   HTML              │
    │<── Affichage ───────│                     │
    │   (~200-500ms)      │                     │
```

### Single Page Application (SPA) avec React

```
Utilisateur           React (JS)             API / Serveur
    │                     │                     │
    │                     │── GET index.html ──>│ (une seule fois au debut)
    │                     │<── HTML + React ────│
    │                     │                     │
    │── Clic "Profil" ──>│                     │
    │                     │── Calcule le VDOM   │
    │                     │── MAJ URL           │
    │<── Affichage ───────│   (pas de requete)  │
    │   (~16ms, 60fps)    │                     │
    │                     │── GET /api/profil ──>│ (optionnel: donnees)
    │                     │<── { json } ────────│
    │<── MAJ donnees ─────│                     │
```

### Comparaison pratique

| Critere | MPA | SPA (React) |
|---|---|---|
| Chargement initial | Rapide | Plus lent (bundle JS) |
| Navigations suivantes | Lent (rechargement) | Instantane |
| Experience utilisateur | Saccadee | Fluide |
| SEO | Excellent | Difficile (SSR requis) |
| Complexite frontend | Faible | Elevee |
| Cas d'usage | Blog, site vitrine | Dashboard, outil collaboratif |

> [!warning] React n'est pas toujours la reponse
> Un site vitrine, un blog ou une page de documentation sont souvent mieux servis par un generateur de site statique (Astro, Hugo) qu'une SPA React. React brille pour les applications interactives : tableaux de bord, outils, applications temps reel.

---

## 3. Le Virtual DOM

### Le cout du DOM reel

```javascript
// ─── Manipuler le DOM reel est couteux ───
// Chaque modification peut declencher :
//   Recalcul des styles (style recalculation)
//   Recalcul du layout (reflow)
//   Repeinture (repaint)
//   Composition des layers

// Mauvais : 1000 operations = 1000 reflows potentiels
const liste = document.getElementById("liste");
for (let i = 0; i < 1000; i++) {
    const li = document.createElement("li");
    li.textContent = items[i].nom;
    liste.appendChild(li); // Reflow a chaque iteration !
}
```

### Le principe du Virtual DOM

```
Cycle de rendu React :

1. Un etat (state) change
         │
         v
2. React cree un NOUVEAU Virtual DOM
   (arbre d'objets JS — rapide a creer)
         │
         v
3. Algorithme de RECONCILIATION (diffing)
   Compare ancien VDOM vs nouveau VDOM

   Ancien VDOM          Nouveau VDOM
   <ul>                 <ul>
     <li>Alice</li>       <li>Alice</li>   ← identique
     <li>Bob</li>         <li>Bob</li>     ← identique
     <li>Carol</li>       <li>Carol</li>   ← identique
   </ul>                  <li>David</li>  ← NOUVEAU
                        </ul>

   Diff calcule = "ajouter 1 element <li>David</li>"
         │
         v
4. React applique UNIQUEMENT les changements au DOM reel
   (1 seule operation au lieu de reconstruire toute la liste)
```

```javascript
// ─── Le VDOM est un objet JS simple ───
// Ce que React cree en memoire :

const vdomElement = {
    type: "div",
    props: {
        className: "carte",
        onClick: handleClick,
    },
    children: [
        {
            type: "h2",
            props: {},
            children: ["Alice Martin"]
        },
        {
            type: "p",
            props: { className: "email" },
            children: ["alice@example.com"]
        }
    ]
};
// Creer cet objet JS = microseconds
// Modifier le DOM reel = millisecondes (x100 plus lent)
```

> [!info] VDOM vs DOM reel
> Le Virtual DOM n'est pas magiquement plus rapide que le DOM reel. Son avantage est qu'il permet a React de **calculer en memoire** le minimum de changements a appliquer, puis de les appliquer en **une seule passe optimisee**. Sur des interfaces complexes avec peu de changements, le gain est significatif.

---

## 4. JSX — JavaScript + XML

### Qu'est-ce que JSX ?

JSX est une extension de syntaxe JavaScript qui ressemble a du HTML. Ce n'est pas du HTML et ce n'est pas reconnu directement par les navigateurs. Babel (ou le compilateur Vite) le transforme en appels `React.createElement()`.

```jsx
// ─── JSX (ce que vous ecrivez) ───
const element = <h1 className="titre">Bonjour !</h1>;

// ─── Ce que le compilateur genere ───
const element = React.createElement(
    "h1",
    { className: "titre" },
    "Bonjour !"
);

// ─── Ce que React.createElement retourne ───
const element = {
    type: "h1",
    props: { className: "titre", children: "Bonjour !" }
};
```

### Differences JSX vs HTML

```jsx
// ─── Attributs : camelCase en JSX ───
// HTML :    <div class="carte" tabindex="0" onclick="...">
// JSX :
<div className="carte" tabIndex={0} onClick={handleClick}>

// ─── Style : objet JS en JSX ───
// HTML :    <div style="color: red; font-size: 16px">
// JSX :
<div style={{ color: "red", fontSize: "16px" }}>
//                           ^^^ camelCase pour font-size !

// ─── Balises auto-fermantes obligatoires en JSX ───
// HTML :    <img src="...">  <input type="text">
// JSX :
<img src="photo.jpg" alt="Photo" />
<input type="text" />

// ─── for en HTML → htmlFor en JSX ───
// HTML :    <label for="email">
// JSX :
<label htmlFor="email">Email</label>
```

### Expressions JSX

```jsx
const nom = "Alice";
const age = 28;
const estAdmin = true;
const items = ["React", "Vue", "Svelte"];

function Exemples() {
    return (
        <div>
            {/* ─── Expressions JavaScript entre accolades ─── */}
            <p>Bonjour, {nom} !</p>
            <p>Dans 10 ans : {age + 10} ans</p>
            <p>Role : {estAdmin ? "Administrateur" : "Utilisateur"}</p>

            {/* ─── Rendu conditionnel avec && ─── */}
            {estAdmin && <span className="badge">Admin</span>}

            {/* ─── Rendu d'une liste ─── */}
            <ul>
                {items.map((item, index) => (
                    <li key={index}>{item}</li>
                ))}
            </ul>

            {/* ─── Expressions complexes ─── */}
            <p>Total : {[1, 2, 3, 4, 5].reduce((a, b) => a + b, 0)}</p>
        </div>
    );
}
```

> [!warning] Ce qu'on NE peut PAS faire en JSX
> - Pas d'instructions (`if`, `for`, `while`) — uniquement des expressions
> - Pas de `console.log()` directement dans le JSX
> - Pas de commentaires HTML `<!-- -->` — utiliser `{/* ... */}`
> - Pas de 2 elements racines sans wrapper

### Fragments

Un composant React doit retourner un seul element racine. Pour eviter d'ajouter des `<div>` inutiles, utiliser les Fragments.

```jsx
// ─── Probleme : deux elements racines ───
// ❌ Interdit
function Mauvais() {
    return (
        <h1>Titre</h1>
        <p>Paragraphe</p>  // Erreur ! Deux elements racines
    );
}

// ─── Solution 1 : wrapper div (ajoute un element au DOM) ───
function AvecDiv() {
    return (
        <div>
            <h1>Titre</h1>
            <p>Paragraphe</p>
        </div>
    );
}

// ─── Solution 2 : Fragment (n'ajoute RIEN au DOM) ───
import { Fragment } from "react";

function AvecFragment() {
    return (
        <Fragment>
            <h1>Titre</h1>
            <p>Paragraphe</p>
        </Fragment>
    );
}

// ─── Solution 3 : syntaxe courte <> </> ───
function AvecSyntaxeCourte() {
    return (
        <>
            <h1>Titre</h1>
            <p>Paragraphe</p>
        </>
    );
}
```

---

## 5. Composants

### Composant fonctionnel

Un composant React est une **fonction** qui retourne du JSX. Son nom commence toujours par une majuscule.

```jsx
// ─── Composant fonctionnel minimal ───
function Salutation() {
    return <h1>Bonjour le monde !</h1>;
}

// ─── Utilisation ───
// React distingue <salutation> (tag HTML inconnu)
// de <Salutation> (composant React)
function App() {
    return <Salutation />;
}
```

### Structure d'un composant complet

```jsx
// ─── Anatomie d'un composant fonctionnel ───
import { useState } from "react"; // Import des hooks

// 1. Declaration du composant (majuscule obligatoire)
function CarteProduit({ nom, prix, disponible, onAjouter }) {
    // 2. Hooks (useState, useEffect...) — en haut du corps de fonction
    const [quantite, setQuantite] = useState(1);

    // 3. Logique derivee (calculs, transformations)
    const prixTotal = (prix * quantite).toFixed(2);
    const labelDisponibilite = disponible ? "En stock" : "Rupture";

    // 4. Gestionnaires d'evenements
    function handleAjouter() {
        onAjouter({ nom, quantite, prixTotal });
    }

    // 5. Rendu JSX (return)
    return (
        <div className={`carte ${!disponible ? "carte--indisponible" : ""}`}>
            <h3>{nom}</h3>
            <p className="prix">{prix} €</p>
            <span className="stock">{labelDisponibilite}</span>

            {disponible && (
                <div className="actions">
                    <input
                        type="number"
                        min={1}
                        value={quantite}
                        onChange={(e) => setQuantite(Number(e.target.value))}
                    />
                    <p>Total : {prixTotal} €</p>
                    <button onClick={handleAjouter}>Ajouter au panier</button>
                </div>
            )}
        </div>
    );
}

export default CarteProduit;
```

### Composant classe (legacy)

Les composants classe sont l'ancienne approche, avant les hooks (React < 16.8). Vous les rencontrerez dans des codebases existantes.

```jsx
// ─── Composant CLASSE — approche LEGACY ───
import { Component } from "react";

class Compteur extends Component {
    // Etat initial
    constructor(props) {
        super(props);
        this.state = { compte: 0 };
        // Liaison manuelle du contexte (ancienne syntaxe)
        this.incrementer = this.incrementer.bind(this);
    }

    // Methodes du cycle de vie
    componentDidMount() {
        console.log("Monte dans le DOM");
    }

    componentDidUpdate(prevProps, prevState) {
        if (prevState.compte !== this.state.compte) {
            document.title = `Compteur : ${this.state.compte}`;
        }
    }

    componentWillUnmount() {
        console.log("Retire du DOM");
    }

    // Methodes
    incrementer() {
        this.setState((prevState) => ({ compte: prevState.compte + 1 }));
    }

    // Rendu obligatoire
    render() {
        return (
            <div>
                <p>{this.state.compte}</p>
                <button onClick={this.incrementer}>+1</button>
            </div>
        );
    }
}
```

> [!info] Composants classe vs fonctionnels
> Depuis React 16.8 (2019), les composants fonctionnels avec hooks sont la norme. Les composants classe ne disparaissent pas (React les supporte toujours), mais tout nouveau code devrait utiliser les composants fonctionnels. Ils sont plus courts, plus lisibles et plus faciles a tester.

---

## 6. Props

### Passage de props

Les props (properties) sont le mecanisme pour passer des donnees d'un composant parent vers un composant enfant. Le flux est **unidirectionnel** : parent → enfant uniquement.

```jsx
// ─── Definition : le composant declare ses props ───
function Badge({ texte, couleur, taille }) {
    return (
        <span
            style={{
                backgroundColor: couleur,
                fontSize: taille === "grand" ? "18px" : "12px",
                padding: "4px 8px",
                borderRadius: "4px",
                color: "white",
            }}
        >
            {texte}
        </span>
    );
}

// ─── Utilisation : le parent passe les props ───
function App() {
    return (
        <div>
            {/* Props passees comme attributs HTML */}
            <Badge texte="Nouveau" couleur="blue" taille="petit" />
            <Badge texte="Promo" couleur="red" taille="grand" />
            <Badge texte="Epuise" couleur="gray" />
        </div>
    );
}
```

### Types de valeurs en props

```jsx
function ExemplesProps({
    // String (pas besoin d'accolades si c'est du texte pur)
    nom,
    // Nombre (accolades obligatoires)
    age,
    // Booleen (sans valeur = true, avec = false possible)
    actif,
    // Objet
    adresse,
    // Tableau
    competences,
    // Fonction (callback)
    onCliquer,
    // JSX / ReactNode (children)
    children,
}) {
    return (
        <div>
            <p>{nom}, {age} ans</p>
            <p>Statut : {actif ? "Actif" : "Inactif"}</p>
            <p>Ville : {adresse.ville}</p>
            <ul>
                {competences.map((c, i) => <li key={i}>{c}</li>)}
            </ul>
            <button onClick={onCliquer}>Action</button>
            <div className="contenu">{children}</div>
        </div>
    );
}

// ─── Utilisation avec toutes les props ───
<ExemplesProps
    nom="Alice"
    age={28}
    actif
    adresse={{ ville: "Paris", cp: "75001" }}
    competences={["React", "TypeScript", "Node"]}
    onCliquer={() => console.log("clique !")}
>
    <p>Contenu enfant passe via children</p>
</ExemplesProps>
```

### Valeurs par defaut (defaultProps et parametre par defaut)

```jsx
// ─── Methode moderne : valeurs par defaut via destructuration ───
function Bouton({
    texte = "Cliquer",
    variante = "primaire",
    taille = "normal",
    desactive = false,
    onClick = () => {},
}) {
    return (
        <button
            className={`btn btn--${variante} btn--${taille}`}
            disabled={desactive}
            onClick={onClick}
        >
            {texte}
        </button>
    );
}

// Tous ces usages sont valides :
<Bouton />                            // Texte "Cliquer", variante "primaire"
<Bouton texte="Valider" />            // Variante "primaire" par defaut
<Bouton texte="Supprimer" variante="danger" taille="grand" />
<Bouton texte="Chargement..." desactive />
```

### PropTypes — validation de props

```jsx
import PropTypes from "prop-types"; // npm install prop-types

function CarteProduit({ nom, prix, note, categories, onAcheter }) {
    return (
        <div>
            <h3>{nom}</h3>
            <p>{prix} €</p>
            <p>Note : {note}/5</p>
            <ul>{categories.map((c, i) => <li key={i}>{c}</li>)}</ul>
            <button onClick={onAcheter}>Acheter</button>
        </div>
    );
}

// ─── Declaration des types attendus ───
CarteProduit.propTypes = {
    nom: PropTypes.string.isRequired,
    prix: PropTypes.number.isRequired,
    note: PropTypes.number,
    categories: PropTypes.arrayOf(PropTypes.string),
    onAcheter: PropTypes.func.isRequired,
};

// ─── Valeurs par defaut ───
CarteProduit.defaultProps = {
    note: 0,
    categories: [],
};
```

> [!tip] PropTypes vs TypeScript
> PropTypes detecte les erreurs de type **a l'execution** (dans la console navigateur). TypeScript les detecte **a la compilation** (dans votre editeur). Sur les nouveaux projets, TypeScript est preferable car les erreurs sont detectees plus tot. PropTypes reste utile sur les projets JavaScript purs.

---

## 7. Rendu Conditionnel

```jsx
function Tableau({ donnees, chargement, erreur, utilisateur }) {
    // ─── Technique 1 : if/return (guard clause) ───
    if (chargement) {
        return <div className="spinner">Chargement...</div>;
    }

    if (erreur) {
        return (
            <div className="erreur">
                <h2>Une erreur est survenue</h2>
                <p>{erreur.message}</p>
                <button onClick={() => window.location.reload()}>Reessayer</button>
            </div>
        );
    }

    if (!donnees || donnees.length === 0) {
        return <p className="vide">Aucune donnee disponible.</p>;
    }

    // ─── Technique 2 : operateur ternaire dans JSX ───
    return (
        <div>
            {utilisateur.estAdmin ? (
                <button className="btn-admin">Modifier tout</button>
            ) : (
                <button className="btn-user">Voir seulement</button>
            )}

            {/* ─── Technique 3 : && court-circuit ─── */}
            {utilisateur.notifications.length > 0 && (
                <div className="badge-notif">
                    {utilisateur.notifications.length}
                </div>
            )}

            {/* ─── Technique 4 : variable intermediaire ─── */}
            {(() => {
                // Pour de la logique complexe, extraire dans une variable
                const contenu = donnees.map(item => (
                    <tr key={item.id}>
                        <td>{item.nom}</td>
                        <td>{item.valeur}</td>
                    </tr>
                ));
                return <table><tbody>{contenu}</tbody></table>;
            })()}
        </div>
    );
}
```

---

## 8. Listes et Keys

### Rendre une liste

```jsx
const produits = [
    { id: 1, nom: "Clavier", prix: 89, stock: 15 },
    { id: 2, nom: "Souris", prix: 45, stock: 0 },
    { id: 3, nom: "Ecran", prix: 349, stock: 7 },
];

function ListeProduits() {
    return (
        <ul>
            {/* ─── Methode map() sur le tableau ─── */}
            {produits.map((produit) => (
                // KEY est obligatoire sur le premier element du map
                <li key={produit.id} className={produit.stock === 0 ? "rupture" : ""}>
                    <strong>{produit.nom}</strong> — {produit.prix} €
                    {produit.stock === 0 && <span> (Epuise)</span>}
                </li>
            ))}
        </ul>
    );
}
```

### Pourquoi les keys sont essentielles

```
Sans key, React ne sait pas quel element a change :

Liste initiale :   [Alice, Bob, Carol]
Liste apres ajout: [David, Alice, Bob, Carol]  ← David insere au debut

Sans key : React croit que TOUS les elements ont change
  → Rerender de toute la liste (inefficace)

Avec key (id unique) : React identifie que seul David est nouveau
  → Rerender de David seulement (optimal)
```

```jsx
// ─── BONNE pratique : key = identifiant metier stable ───
{utilisateurs.map(user => (
    <CarteProfil key={user.id} nom={user.nom} email={user.email} />
))}

// ─── MAUVAISE pratique : key = index du tableau ───
// Problematique si la liste peut etre reordonnee/filtree
{utilisateurs.map((user, index) => (
    <CarteProfil key={index} nom={user.nom} />  // ← a eviter
))}
```

> [!warning] Key = index uniquement en dernier recours
> Utiliser l'index comme key est acceptable UNIQUEMENT si la liste est statique (jamais triee, filtrée ou reordonnee). Sinon, React peut associer le mauvais composant a la mauvaise key, causant des bugs visuels subtils (valeurs d'input qui ne se deplacent pas avec l'element).

---

## 9. Gestion des Evenements

### Syntaxe de base

```jsx
// ─── Evenements React vs DOM natif ───
// DOM natif :    element.addEventListener("click", handler)
// HTML :         <button onclick="handler()">
// JSX :          <button onClick={handler}>

function ExemplesEvenements() {
    // ─── Handler comme fonction nommee (recommande) ───
    function handleClick() {
        console.log("Bouton clique");
    }

    // ─── Handler avec parametres ───
    function handleSupprimer(id, event) {
        event.stopPropagation(); // Empeche la propagation
        console.log(`Supprimer l'element ${id}`);
    }

    // ─── Handler inline (pour la logique simple uniquement) ───
    return (
        <div onClick={() => console.log("div cliquee")}>
            {/* Passer la reference, PAS l'appel */}
            <button onClick={handleClick}>Cliquer</button>

            {/* Pour passer des arguments, utiliser une arrow function */}
            <button onClick={() => handleSupprimer(42)}>Supprimer</button>

            {/* Recuperer l'objet event ───────────────────────── */}
            <input
                onChange={(event) => console.log(event.target.value)}
                placeholder="Taper quelque chose"
            />

            {/* Empecher le comportement par defaut */}
            <a
                href="https://example.com"
                onClick={(e) => {
                    e.preventDefault();
                    console.log("Lien intercepte");
                }}
            >
                Lien
            </a>
        </div>
    );
}
```

### Evenements courants en React

| Evenement JSX | Equivalent DOM | Quand s'execute |
|---|---|---|
| `onClick` | `click` | Clic souris / tap tactile |
| `onChange` | `change` / `input` | Modification d'input |
| `onSubmit` | `submit` | Soumission de formulaire |
| `onKeyDown` | `keydown` | Touche pressee |
| `onKeyUp` | `keyup` | Touche relachee |
| `onFocus` | `focus` | Element recoit le focus |
| `onBlur` | `blur` | Element perd le focus |
| `onMouseEnter` | `mouseenter` | Souris entre dans l'element |
| `onMouseLeave` | `mouseleave` | Souris quitte l'element |
| `onScroll` | `scroll` | Scroll dans l'element |
| `onDragStart` | `dragstart` | Debut de drag |
| `onDrop` | `drop` | Drop d'un element |

---

## 10. Setup avec Vite

### Creer un projet React avec Vite

```bash
# ─── Creer un nouveau projet React ───
npm create vite@latest mon-app -- --template react
# ou avec TypeScript (recommande pour les nouveaux projets)
npm create vite@latest mon-app -- --template react-ts

# ─── Installer les dependances ───
cd mon-app
npm install

# ─── Lancer le serveur de developpement ───
npm run dev
# → http://localhost:5173

# ─── Scripts disponibles ───
npm run dev       # Serveur dev avec HMR (Hot Module Replacement)
npm run build     # Build de production dans dist/
npm run preview   # Previsualiser le build de production
```

### Structure d'un projet React/Vite

```
mon-app/
│
├── public/                    ← Fichiers statiques (favicon, images publiques)
│   └── vite.svg
│
├── src/                       ← Votre code source
│   ├── assets/                ← Images, fonts importees par JS
│   │   └── react.svg
│   │
│   ├── components/            ← Composants reutilisables (a creer)
│   │   ├── Bouton.jsx
│   │   ├── Header.jsx
│   │   └── ...
│   │
│   ├── pages/                 ← Composants de page (a creer)
│   │   ├── Accueil.jsx
│   │   └── Profil.jsx
│   │
│   ├── App.jsx                ← Composant racine
│   ├── App.css                ← Styles globaux de l'app
│   ├── index.css              ← Styles de base (reset, variables CSS)
│   └── main.jsx               ← Point d'entree — monte React dans le DOM
│
├── index.html                 ← Page HTML unique (le "shell" SPA)
├── vite.config.js             ← Configuration Vite
├── package.json               ← Dependances et scripts
└── .gitignore                 ← Exclut node_modules, dist, etc.
```

### Fichiers cles expliques

```jsx
// ─── main.jsx : point d'entree ───
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";
import App from "./App.jsx";

// Trouver l'element racine dans index.html
const racine = document.getElementById("root");

// Monter React dans cet element
createRoot(racine).render(
    // StrictMode : detecte les problemes potentiels en developpement
    // Double les renders en dev pour detecter les effets de bord
    <StrictMode>
        <App />
    </StrictMode>
);
```

```html
<!-- index.html : le shell SPA -->
<!doctype html>
<html lang="fr">
    <head>
        <meta charset="UTF-8" />
        <link rel="icon" type="image/svg+xml" href="/vite.svg" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Mon App React</title>
    </head>
    <body>
        <!-- React va "s'accrocher" ici -->
        <div id="root"></div>
        <!-- Vite injecte automatiquement le script -->
        <script type="module" src="/src/main.jsx"></script>
    </body>
</html>
```

```jsx
// ─── App.jsx : composant racine ───
import { useState } from "react";
import Header from "./components/Header";
import "./App.css";

function App() {
    const [compteur, setCompteur] = useState(0);

    return (
        <>
            <Header titre="Mon Application" />
            <main>
                <h1>Bienvenue dans React</h1>
                <p>Compteur : {compteur}</p>
                <button onClick={() => setCompteur(c => c + 1)}>
                    Incrementer
                </button>
            </main>
        </>
    );
}

export default App;
```

---

## 11. Premier Composant Complet

Voici un exemple de composant autonome et realiste qui combine tout ce que nous avons vu.

```jsx
// ─── src/components/ListeTaches.jsx ───
import { useState } from "react";

// Composant enfant : une tache individuelle
function ItemTache({ tache, onToggle, onSupprimer }) {
    return (
        <li
            className={`tache ${tache.terminee ? "tache--terminee" : ""}`}
            style={{ listStyle: "none", padding: "8px", marginBottom: "4px" }}
        >
            <input
                type="checkbox"
                checked={tache.terminee}
                onChange={() => onToggle(tache.id)}
            />
            <span style={{ marginLeft: "8px", textDecoration: tache.terminee ? "line-through" : "none" }}>
                {tache.texte}
            </span>
            <button
                onClick={() => onSupprimer(tache.id)}
                style={{ marginLeft: "auto", float: "right" }}
            >
                ×
            </button>
        </li>
    );
}

// Composant parent : la liste complete
function ListeTaches() {
    const [taches, setTaches] = useState([
        { id: 1, texte: "Apprendre React", terminee: true },
        { id: 2, texte: "Comprendre les hooks", terminee: false },
        { id: 3, texte: "Creer un projet", terminee: false },
    ]);
    const [nouvelleTache, setNouvelleTache] = useState("");
    const [filtre, setFiltre] = useState("toutes"); // "toutes" | "actives" | "terminees"

    // Ajouter une tache
    function handleAjouter(e) {
        e.preventDefault();
        if (!nouvelleTache.trim()) return;

        setTaches([
            ...taches,
            {
                id: Date.now(), // Identifiant unique simple
                texte: nouvelleTache.trim(),
                terminee: false,
            },
        ]);
        setNouvelleTache(""); // Vider le champ
    }

    // Basculer l'etat termine/non-termine
    function handleToggle(id) {
        setTaches(taches.map(t =>
            t.id === id ? { ...t, terminee: !t.terminee } : t
        ));
    }

    // Supprimer une tache
    function handleSupprimer(id) {
        setTaches(taches.filter(t => t.id !== id));
    }

    // Filtrer les taches selon le filtre actif
    const tachesFiltrees = taches.filter(t => {
        if (filtre === "actives") return !t.terminee;
        if (filtre === "terminees") return t.terminee;
        return true; // "toutes"
    });

    const nbRestantes = taches.filter(t => !t.terminee).length;

    return (
        <div style={{ maxWidth: "400px", margin: "2rem auto", fontFamily: "sans-serif" }}>
            <h1>Liste de taches</h1>
            <p>{nbRestantes} tache(s) restante(s)</p>

            {/* Formulaire d'ajout */}
            <form onSubmit={handleAjouter} style={{ display: "flex", gap: "8px" }}>
                <input
                    type="text"
                    value={nouvelleTache}
                    onChange={(e) => setNouvelleTache(e.target.value)}
                    placeholder="Nouvelle tache..."
                    style={{ flex: 1, padding: "8px" }}
                />
                <button type="submit">Ajouter</button>
            </form>

            {/* Filtres */}
            <div style={{ margin: "1rem 0", display: "flex", gap: "8px" }}>
                {["toutes", "actives", "terminees"].map(f => (
                    <button
                        key={f}
                        onClick={() => setFiltre(f)}
                        style={{ fontWeight: filtre === f ? "bold" : "normal" }}
                    >
                        {f.charAt(0).toUpperCase() + f.slice(1)}
                    </button>
                ))}
            </div>

            {/* Liste */}
            {tachesFiltrees.length === 0 ? (
                <p>Aucune tache dans cette categorie.</p>
            ) : (
                <ul style={{ padding: 0 }}>
                    {tachesFiltrees.map(tache => (
                        <ItemTache
                            key={tache.id}
                            tache={tache}
                            onToggle={handleToggle}
                            onSupprimer={handleSupprimer}
                        />
                    ))}
                </ul>
            )}
        </div>
    );
}

export default ListeTaches;
```

---

## 12. Organisation d'un Projet React

### Convention de nommage et de structure

```
src/
├── components/          ← Composants reutilisables (pas de logique metier)
│   ├── ui/              ← Composants tres generiques (Button, Input, Card...)
│   │   ├── Bouton.jsx
│   │   ├── Input.jsx
│   │   └── Carte.jsx
│   └── features/        ← Composants lies a une feature
│       ├── auth/
│       │   ├── FormulaireConnexion.jsx
│       │   └── BoutonDeconnexion.jsx
│       └── produits/
│           ├── ListeProduits.jsx
│           └── CarteProduit.jsx
│
├── pages/               ← Composants de page (1 par route)
│   ├── Accueil.jsx
│   ├── Profil.jsx
│   └── PasDeTrouve.jsx
│
├── hooks/               ← Custom hooks (prefixe "use")
│   ├── useLocalStorage.js
│   └── useFetch.js
│
├── services/            ← Logique metier, appels API
│   └── api.js
│
├── utils/               ← Fonctions utilitaires pures
│   └── formatDate.js
│
└── constants/           ← Constantes de l'application
    └── routes.js
```

> [!tip] Commencer simple, refactorer ensuite
> Ne pas creer toute cette structure des le debut. Commencez avec `components/` et `pages/` uniquement. Extractez `hooks/` et `services/` quand le code se repete. La sur-structuration prematuree est aussi nocive que l'absence de structure.

---

## Carte Mentale ASCII

```
                        REACT — FONDAMENTAUX
                                │
        ┌───────────────────────┼──────────────────────┐
        │                       │                      │
    CONCEPT                    JSX                 COMPOSANTS
        │                       │                      │
   Declaratif             HTML + JS              Fonctionnel
   Virtual DOM            Expressions            Props
   Unidirectionnel        Fragments              Children
   Composants             className              DefaultProps
                          camelCase              PropTypes
        │                                             │
    EVENEMENTS                                  RENDU COND.
        │                                             │
    onClick               LISTES                ternaire ?:
    onChange              map() + key           && court-circuit
    onSubmit              Identifiant stable     if/return
    preventDefault        Eviter index           switch/case
        │
    SETUP VITE
        │
    npm create vite
    src/main.jsx
    src/App.jsx
    public/index.html
    npm run dev
```

---

## Exercices

### Exercice 1 : Composant Carte de profil

Creez un composant `CarteProfil` qui affiche :
- Une photo (utiliser une URL d'image placeholder)
- Un nom, un titre professionnel et une ville
- Une liste de competences (tableau de strings en props)
- Un bouton "Contacter" qui affiche une alerte avec le nom
- Si la propriete `premium` est `true`, afficher un badge "Premium"
- Valeurs par defaut pour toutes les props

### Exercice 2 : Liste filtrable

Creez un composant `ListeFiltrableIngredients` avec :
- Un tableau d'ingredients en props (`{ id, nom, categorie, calories }`)
- Un champ de recherche qui filtre par nom en temps reel
- Des boutons de filtre par categorie ("Tous", "Legumes", "Viandes", "Feculents")
- L'affichage du nombre de resultats
- Un message "Aucun resultat" si la liste filtree est vide
- Mettre en evidence le terme recherche dans les resultats (envelopper dans `<mark>`)

### Exercice 3 : Composant d'accordeon

Creez un composant `Accordeon` qui :
- Recoit en props un tableau `{ id, titre, contenu }` de sections
- N'affiche qu'une section ouverte a la fois
- Anime l'ouverture/fermeture via une transition CSS
- Affiche une icone `+` / `-` selon l'etat ouvert/ferme
- Expose une prop `ouvertParDefaut` pour definir la section initiale

### Exercice 4 : Compteur de mots

Creez un composant `AnalyseurTexte` avec :
- Un `<textarea>` pour saisir du texte
- En temps reel : nombre de mots, caracteres, caracteres sans espaces, phrases
- Une barre de progression qui se remplit selon une limite configurable (prop `limite`, defaut 200 mots)
- La barre devient rouge quand on depasse 80% de la limite
- Un bouton "Effacer" qui vide le textarea et remet tous les compteurs a zero

---

## Liens

- [[05 - SPA et Frameworks Introduction]]
- [[02 - Hooks et Gestion d'Etat]]
- [[03 - React Router et Formulaires]]
- [[04 - React Avance et Tests]]
