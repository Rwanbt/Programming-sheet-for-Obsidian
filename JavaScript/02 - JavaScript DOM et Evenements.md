# JavaScript DOM et Evenements

Le DOM (Document Object Model) est l'interface entre JavaScript et la page HTML. Il transforme le document HTML en un arbre d'objets que JavaScript peut lire, modifier, creer et supprimer. Combinee aux evenements (clics, saisies, soumissions), cette capacite de manipulation fait de JavaScript le langage incontournable du developpement web interactif.

Ce cours couvre la navigation dans le DOM, la manipulation d'elements, le systeme d'evenements, les formulaires, le stockage local, et se termine par un exemple pratique de liste de taches dynamique.

> [!tip] Analogie
> Imaginez que le HTML est le plan d'un immeuble (structure figee sur papier). Le DOM est la **maquette 3D interactive** de cet immeuble : vous pouvez ajouter des etages, deplacer des murs, changer les couleurs -- en temps reel. JavaScript est l'architecte qui manipule cette maquette. En C, vous gerez la memoire manuellement pour chaque structure. En Python avec BeautifulSoup, vous parsez du HTML mais sans interactivite. En JS dans le navigateur, le DOM est vivant.

---

## 1. Qu'est-ce que le DOM ?

Le DOM est une **API** (Application Programming Interface) fournie par le navigateur qui represente le document HTML sous forme d'un arbre de noeuds (nodes).

### Document HTML source

```html
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Ma Page</title>
  </head>
  <body>
    <h1 id="titre">Bienvenue</h1>
    <div class="contenu">
      <p>Premier paragraphe</p>
      <p>Deuxieme paragraphe</p>
    </div>
    <ul id="liste">
      <li>Element 1</li>
      <li>Element 2</li>
    </ul>
  </body>
</html>
```

### Representation en arbre (ASCII)

```
document
  └── html (lang="fr")
       ├── head
       │    └── title
       │         └── "Ma Page" (text node)
       │
       └── body
            ├── h1 (id="titre")
            │    └── "Bienvenue" (text node)
            │
            ├── div (class="contenu")
            │    ├── p
            │    │    └── "Premier paragraphe" (text node)
            │    └── p
            │         └── "Deuxieme paragraphe" (text node)
            │
            └── ul (id="liste")
                 ├── li
                 │    └── "Element 1" (text node)
                 └── li
                      └── "Element 2" (text node)
```

### Types de noeuds

```
+--------------------+----------+-------------------------------+
| Type               | nodeType | Exemple                       |
+--------------------+----------+-------------------------------+
| Element            | 1        | <div>, <p>, <h1>             |
| Text               | 3        | "Bienvenue"                  |
| Comment            | 8        | <!-- commentaire -->          |
| Document           | 9        | document                     |
| DocumentFragment   | 11       | fragment temporaire           |
+--------------------+----------+-------------------------------+
```

> [!info] DOM et memoire
> Chaque noeud du DOM est un **objet JavaScript** en memoire. Manipuler le DOM revient a modifier ces objets. Le navigateur synchronise ensuite l'affichage (rendering). C'est pourquoi les modifications massives du DOM peuvent etre couteuses en performance.

---

## 2. Acceder aux elements

### Methodes de selection

```javascript
// ─── Par ID (retourne UN element ou null) ───
const titre = document.getElementById("titre");

// ─── Par selecteur CSS (retourne le PREMIER element ou null) ───
const premierP = document.querySelector("p");
const contenu = document.querySelector(".contenu");
const titreCSS = document.querySelector("#titre");
const listeItem = document.querySelector("ul#liste > li:first-child");

// ─── Par selecteur CSS (retourne TOUS les elements, NodeList) ───
const tousLesP = document.querySelectorAll("p");
const tousLesLi = document.querySelectorAll("#liste li");

// ─── Methodes anciennes (encore courantes) ───
const parClasse = document.getElementsByClassName("contenu"); // HTMLCollection
const parTag = document.getElementsByTagName("p");            // HTMLCollection
const parNom = document.getElementsByName("email");           // NodeList
```

### NodeList vs HTMLCollection

```
+--------------------+----------------------------------+-------------------+
| Propriete          | NodeList                         | HTMLCollection    |
+--------------------+----------------------------------+-------------------+
| Retourne par       | querySelectorAll, getElementsByName | getElementsBy* |
| forEach            | Oui                              | Non (*)          |
| Statique/Live      | Statique (querySelectorAll)      | Live             |
| Conversion Array   | Array.from(nodeList)             | Array.from(coll) |
+--------------------+----------------------------------+-------------------+
(*) Convertir d'abord : Array.from(collection).forEach(...)
```

```javascript
// ─── Iterer sur les resultats ───
const paragraphes = document.querySelectorAll("p");

// Methode 1 : forEach (NodeList le supporte)
paragraphes.forEach((p, index) => {
    console.log(`Paragraphe ${index}: ${p.textContent}`);
});

// Methode 2 : for...of
for (const p of paragraphes) {
    console.log(p.textContent);
}

// Methode 3 : convertir en Array pour utiliser map, filter, etc.
const textes = Array.from(paragraphes).map(p => p.textContent);
```

> [!warning] querySelector vs getElementById
> `querySelector("#id")` et `getElementById("id")` retournent le meme element, mais `getElementById` est legerement plus rapide. Pour la selection par ID, les deux sont acceptables. Pour les selecteurs CSS complexes, seul `querySelector` fonctionne.

---

## 3. Modifier le contenu

### textContent, innerHTML, innerText

```javascript
const titre = document.getElementById("titre");

// ─── textContent : texte brut (plus sur, plus rapide) ───
titre.textContent = "Nouveau titre";
console.log(titre.textContent); // "Nouveau titre"

// ─── innerHTML : contenu HTML (attention aux failles XSS !) ───
titre.innerHTML = "Titre en <em>italique</em>";
// Le navigateur parse le HTML et cree les noeuds

// ─── innerText : texte visible (prend en compte le CSS) ───
// Si un element est display:none, innerText l'ignore
// textContent retourne tout le texte, visible ou non
```

### Difference textContent vs innerHTML vs innerText

```
                    textContent              innerHTML               innerText
                    ───────────              ─────────               ─────────
Retourne :          Texte brut               HTML complet            Texte visible
Parse HTML :        Non                      Oui                     Non
Securite :          Sur (pas de XSS)         Risque XSS (*)          Sur
Performance :       Rapide                   Plus lent (parsing)     Lent (layout)
Elements caches :   Inclus                   Inclus                  Exclus

(*) Ne JAMAIS injecter des donnees utilisateur avec innerHTML !
```

> [!warning] Faille XSS avec innerHTML
> ```javascript
> // DANGEREUX : injection de code malveillant
> const saisie = '<img src=x onerror="alert(document.cookie)">';
> element.innerHTML = saisie; // Execute le JavaScript malveillant !
>
> // SECURISE : utiliser textContent
> element.textContent = saisie; // Affiche le texte brut, pas d'execution
> ```

---

## 4. Modifier les attributs

```javascript
const lien = document.querySelector("a");

// ─── Lire un attribut ───
const href = lien.getAttribute("href");
const classe = lien.getAttribute("class");

// ─── Definir un attribut ───
lien.setAttribute("href", "https://example.com");
lien.setAttribute("target", "_blank");
lien.setAttribute("data-id", "42");   // attributs personnalises (data-*)

// ─── Supprimer un attribut ───
lien.removeAttribute("target");

// ─── Verifier l'existence ───
lien.hasAttribute("href"); // true
```

### Manipuler les classes avec classList

```javascript
const div = document.querySelector(".contenu");

// ─── Ajouter une ou plusieurs classes ───
div.classList.add("active");
div.classList.add("theme-dark", "animate");

// ─── Supprimer une classe ───
div.classList.remove("animate");

// ─── Toggle (ajouter si absente, retirer si presente) ───
div.classList.toggle("visible");
// Retourne true si ajoutee, false si retiree

// ─── Toggle conditionnel ───
div.classList.toggle("active", estActif); // force l'ajout/retrait

// ─── Verifier la presence ───
div.classList.contains("active"); // true ou false

// ─── Remplacer ───
div.classList.replace("theme-dark", "theme-light");
```

> [!tip] Analogie
> `classList` est comme un systeme d'etiquettes (tags) sur un dossier. Vous pouvez en ajouter (`add`), en retirer (`remove`), verifier si une etiquette est presente (`contains`), ou basculer (`toggle`) -- exactement comme les tags dans une application de notes.

---

## 5. Modifier les styles

### Style inline (propriete style)

```javascript
const boite = document.querySelector(".boite");

// ─── Definir des styles (camelCase !) ───
boite.style.backgroundColor = "blue";      // background-color => backgroundColor
boite.style.fontSize = "20px";             // font-size => fontSize
boite.style.marginTop = "10px";            // margin-top => marginTop
boite.style.display = "flex";
boite.style.border = "2px solid red";

// ─── Lire un style inline ───
console.log(boite.style.backgroundColor);  // "blue"

// ─── Supprimer un style inline ───
boite.style.backgroundColor = "";          // Remet a la valeur CSS par defaut

// ─── Definir plusieurs styles d'un coup ───
boite.style.cssText = "color: white; padding: 10px; border-radius: 5px;";
```

### Lire les styles calcules (getComputedStyle)

```javascript
const boite = document.querySelector(".boite");

// style.* ne retourne que les styles INLINE
// Pour lire les styles appliques par CSS :
const styles = getComputedStyle(boite);
console.log(styles.width);           // "200px" (valeur calculee)
console.log(styles.backgroundColor); // "rgb(0, 0, 255)"
console.log(styles.display);         // "flex"
```

> [!info] Bonne pratique : classes CSS plutot que styles inline
> Plutot que de modifier les styles un par un en JavaScript, ajoutez/retirez des classes CSS. C'est plus maintenable, plus performant, et separe la logique (JS) de la presentation (CSS).
> ```javascript
> // Preferer :
> element.classList.add("erreur");
> // Plutot que :
> element.style.color = "red";
> element.style.border = "1px solid red";
> element.style.fontWeight = "bold";
> ```

---

## 6. Creer et supprimer des elements

### Creer des elements

```javascript
// ─── Creer un element ───
const nouvelElement = document.createElement("div");
nouvelElement.textContent = "Je suis nouveau !";
nouvelElement.classList.add("carte");
nouvelElement.setAttribute("data-id", "1");

// ─── Creer un noeud texte (rarement necessaire) ───
const texte = document.createTextNode("Du texte brut");

// ─── Creer un fragment (pour des insertions multiples) ───
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
    const li = document.createElement("li");
    li.textContent = `Element ${i}`;
    fragment.appendChild(li);
}
// Une seule insertion dans le DOM (performant !)
document.getElementById("liste").appendChild(fragment);
```

### Inserer des elements

```javascript
const parent = document.querySelector(".contenu");
const nouveau = document.createElement("p");
nouveau.textContent = "Nouveau paragraphe";

// ─── appendChild : ajoute a la FIN des enfants ───
parent.appendChild(nouveau);

// ─── append : ajoute a la fin (accepte texte + plusieurs elements) ───
parent.append(nouveau, " et du texte", document.createElement("br"));

// ─── prepend : ajoute au DEBUT ───
parent.prepend(nouveau);

// ─── before / after : inserer AVANT ou APRES un element ───
const existant = document.querySelector("p");
existant.before(nouveau);  // Insere avant
existant.after(nouveau);   // Insere apres

// ─── insertBefore : methode classique ───
parent.insertBefore(nouveau, existant);

// ─── insertAdjacentHTML : inserer du HTML a une position precise ───
existant.insertAdjacentHTML("beforebegin", "<p>Avant l'element</p>");
existant.insertAdjacentHTML("afterbegin", "<strong>Debut interne</strong>");
existant.insertAdjacentHTML("beforeend", "<em>Fin interne</em>");
existant.insertAdjacentHTML("afterend", "<p>Apres l'element</p>");
```

### Positions de insertAdjacentHTML

```
<!-- beforebegin -->
<p>
  <!-- afterbegin -->
  Contenu existant
  <!-- beforeend -->
</p>
<!-- afterend -->
```

### Supprimer et remplacer des elements

```javascript
// ─── remove() : methode moderne ───
const element = document.querySelector(".a-supprimer");
element.remove();

// ─── removeChild : methode classique ───
const parent = document.querySelector(".contenu");
const enfant = parent.querySelector("p");
parent.removeChild(enfant);

// ─── replaceChild : remplacer un enfant ───
const ancien = document.querySelector(".ancien");
const nouveau = document.createElement("div");
nouveau.textContent = "Remplacant";
ancien.parentNode.replaceChild(nouveau, ancien);

// ─── replaceWith : methode moderne ───
ancien.replaceWith(nouveau);
```

### Cloner un element

```javascript
const original = document.querySelector(".carte");

// ─── Clone superficiel (sans enfants) ───
const copie1 = original.cloneNode(false);

// ─── Clone profond (avec tous les enfants) ───
const copie2 = original.cloneNode(true);

document.body.appendChild(copie2);
```

---

## 7. Naviguer dans le DOM

```javascript
const element = document.querySelector(".contenu");

// ─── Parents ───
element.parentNode;          // Noeud parent (peut etre un text node)
element.parentElement;       // Element parent (toujours un Element)
element.closest(".wrapper"); // Ancetre le plus proche qui correspond au selecteur

// ─── Enfants ───
element.children;            // HTMLCollection des enfants Element
element.childNodes;          // NodeList de TOUS les noeuds enfants (texte inclus)
element.firstElementChild;   // Premier enfant Element
element.lastElementChild;    // Dernier enfant Element
element.childElementCount;   // Nombre d'enfants Element

// ─── Freres et soeurs (siblings) ───
element.nextElementSibling;      // Element suivant
element.previousElementSibling;  // Element precedent
element.nextSibling;             // Noeud suivant (peut etre texte/commentaire)
element.previousSibling;         // Noeud precedent
```

### Schema de navigation

```
                    parentNode / parentElement
                           │
      previousElementSibling ── [ELEMENT] ── nextElementSibling
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
              firstElementChild  children[1]  lastElementChild
              children[0]                    children[length-1]
```

> [!example] Parcours complet d'un arbre DOM
> ```javascript
> function afficherArbre(element, profondeur = 0) {
>     const indent = "  ".repeat(profondeur);
>     const tag = element.tagName?.toLowerCase() || "#text";
>     const id = element.id ? `#${element.id}` : "";
>     const cls = element.className ? `.${element.className}` : "";
>     console.log(`${indent}${tag}${id}${cls}`);
>
>     for (const enfant of element.children || []) {
>         afficherArbre(enfant, profondeur + 1);
>     }
> }
> afficherArbre(document.body);
> ```

---

## 8. Evenements

### addEventListener

```javascript
const bouton = document.querySelector("#mon-bouton");

// ─── Syntaxe de base ───
bouton.addEventListener("click", function(event) {
    console.log("Bouton clique !");
    console.log("Element clique :", event.target);
});

// ─── Avec arrow function ───
bouton.addEventListener("click", (e) => {
    console.log("Clique sur :", e.target.textContent);
});

// ─── Avec une fonction nommee (necessaire pour removeEventListener) ───
function gererClic(e) {
    console.log("Gestion du clic");
}

bouton.addEventListener("click", gererClic);
bouton.removeEventListener("click", gererClic); // Fonctionne !

// ─── Options ───
bouton.addEventListener("click", gererClic, {
    once: true,      // Se declenche UNE seule fois
    capture: false,  // Phase de capture (voir section bubbling)
    passive: true    // N'appellera pas preventDefault (performance scroll)
});
```

> [!warning] removeEventListener necessite la meme reference
> ```javascript
> // NE FONCTIONNE PAS (fonctions anonymes differentes en memoire)
> bouton.addEventListener("click", () => console.log("A"));
> bouton.removeEventListener("click", () => console.log("A")); // Pas la meme reference !
>
> // FONCTIONNE (meme reference de fonction)
> const handler = () => console.log("B");
> bouton.addEventListener("click", handler);
> bouton.removeEventListener("click", handler); // OK, meme reference
> ```

### Comparaison avec d'autres approches

```
C :       Pas d'evenements natifs (signal handlers pour OS)
Python :  tkinter : bouton.bind("<Button-1>", callback)
          PyQt :    bouton.clicked.connect(callback)
JS :      element.addEventListener("click", callback)
```

---

## 9. Types d'evenements courants

### Tableau des evenements principaux

```
+-------------------+-------------------------------+----------------------------+
| Categorie         | Evenement                     | Declencheur                |
+-------------------+-------------------------------+----------------------------+
| Souris            | click                         | Clic gauche                |
|                   | dblclick                      | Double-clic                |
|                   | mousedown / mouseup           | Bouton presse / relache    |
|                   | mouseover / mouseout          | Survol entre / sortie      |
|                   | mouseenter / mouseleave       | Comme over/out sans bubble |
|                   | mousemove                     | Deplacement souris         |
|                   | contextmenu                   | Clic droit                 |
+-------------------+-------------------------------+----------------------------+
| Clavier           | keydown                       | Touche pressee             |
|                   | keyup                         | Touche relachee            |
|                   | keypress (deprecie)           | Caractere genere           |
+-------------------+-------------------------------+----------------------------+
| Formulaire        | submit                        | Soumission formulaire      |
|                   | input                         | Valeur change (temps reel) |
|                   | change                        | Valeur change (perte focus)|
|                   | focus / blur                  | Gain / perte de focus      |
|                   | reset                         | Reset du formulaire        |
+-------------------+-------------------------------+----------------------------+
| Document/Window   | DOMContentLoaded              | HTML parse (sans CSS/img)  |
|                   | load                          | Page entiere chargee       |
|                   | resize                        | Fenetre redimensionnee     |
|                   | scroll                        | Defilement                 |
|                   | beforeunload                  | Avant fermeture page       |
+-------------------+-------------------------------+----------------------------+
| Drag & Drop       | dragstart / dragend           | Debut / fin du drag        |
|                   | dragover / drop               | Survol / depot             |
+-------------------+-------------------------------+----------------------------+
```

### Exemples pratiques

```javascript
// ─── Clavier : detecter les touches ───
document.addEventListener("keydown", (e) => {
    console.log(`Touche : ${e.key}, Code : ${e.code}`);
    if (e.key === "Escape") {
        fermerModal();
    }
    if (e.ctrlKey && e.key === "s") {
        e.preventDefault(); // Empeche le "Enregistrer" du navigateur
        sauvegarder();
    }
});

// ─── Souris : position du curseur ───
document.addEventListener("mousemove", (e) => {
    console.log(`Position : (${e.clientX}, ${e.clientY})`);
});

// ─── Chargement de la page ───
document.addEventListener("DOMContentLoaded", () => {
    console.log("DOM pret ! (images pas encore chargees)");
    initialiserApplication();
});

window.addEventListener("load", () => {
    console.log("Page entierement chargee (images incluses)");
});
```

> [!info] DOMContentLoaded vs load
> `DOMContentLoaded` se declenche des que le HTML est parse et le DOM construit, **sans attendre** les images, CSS ou iframes. `load` attend que **tout** soit charge. Utilisez `DOMContentLoaded` pour initialiser votre JS le plus tot possible.

---

## 10. L'objet Event

```javascript
element.addEventListener("click", function(e) {
    // ─── Proprietes principales ───
    e.type;              // "click" (type de l'evenement)
    e.target;            // Element qui a DECLENCHE l'evenement
    e.currentTarget;     // Element sur lequel le listener est ATTACHE
    e.timeStamp;         // Timestamp de l'evenement
    e.isTrusted;         // true si declenche par l'utilisateur

    // ─── Position (evenements souris) ───
    e.clientX; e.clientY;    // Position dans la fenetre (viewport)
    e.pageX; e.pageY;       // Position dans la page (avec scroll)
    e.offsetX; e.offsetY;   // Position relative a l'element cible
    e.screenX; e.screenY;   // Position sur l'ecran physique

    // ─── Touches modificatrices ───
    e.ctrlKey;           // true si Ctrl est presse
    e.shiftKey;          // true si Shift est presse
    e.altKey;            // true si Alt est presse
    e.metaKey;           // true si Meta/Cmd est presse

    // ─── Methodes ───
    e.preventDefault();      // Empeche l'action par defaut
    e.stopPropagation();     // Arrete la propagation (bubbling/capturing)
    e.stopImmediatePropagation(); // Arrete aussi les autres listeners du meme element
});
```

### preventDefault : cas d'usage courants

```javascript
// ─── Empecher la soumission d'un formulaire ───
form.addEventListener("submit", (e) => {
    e.preventDefault(); // Pas de rechargement de page
    // Traiter les donnees en JavaScript
});

// ─── Empecher la navigation d'un lien ───
lien.addEventListener("click", (e) => {
    e.preventDefault(); // Le lien ne navigue pas
    // Logique personnalisee
});

// ─── Empecher le menu contextuel ───
element.addEventListener("contextmenu", (e) => {
    e.preventDefault();
    afficherMenuPersonnalise(e.clientX, e.clientY);
});
```

---

## 11. Event Bubbling et Capturing

Quand un evenement se produit sur un element, il ne se declenche pas uniquement sur cet element. Il traverse le DOM en trois phases.

### Les trois phases

```
Phase 1 : CAPTURING (du haut vers le bas)
Phase 2 : TARGET (sur l'element cible)
Phase 3 : BUBBLING (du bas vers le haut)

         ┌───── window ─────────────────────────────────┐
         │  ┌── document ───────────────────────────┐   │
         │  │  ┌── html ───────────────────────┐    │   │
         │  │  │  ┌── body ───────────────┐    │    │   │
         │  │  │  │  ┌── div ────────┐    │    │    │   │
         │  │  │  │  │  ┌─ button ─┐ │    │    │    │   │
    1    │  │  │  │  │  │  CLICK   │ │    │    │    │   │    3
 ───────>│──│──│──│──│──│──> X <───│─│────│────│────│───│ ──────>
Capturing│  │  │  │  │  │  TARGET  │ │    │    │    │   │ Bubbling
         │  │  │  │  │  └──────────┘ │    │    │    │   │
         │  │  │  │  └───────────────┘    │    │    │   │
         │  │  │  └───────────────────────┘    │    │   │
         │  │  └───────────────────────────────┘    │   │
         │  └───────────────────────────────────────┘   │
         └──────────────────────────────────────────────┘
```

### Demonstration

```javascript
// HTML : <div id="parent"><button id="enfant">Cliquez</button></div>

const parent = document.getElementById("parent");
const enfant = document.getElementById("enfant");

// ─── Bubbling (par defaut) ───
parent.addEventListener("click", () => console.log("Parent - Bubbling"));
enfant.addEventListener("click", () => console.log("Enfant - Bubbling"));
// Clic sur le bouton affiche :
// "Enfant - Bubbling"   (phase target)
// "Parent - Bubbling"   (phase bubbling : remonte)

// ─── Capturing (3e parametre = true) ───
parent.addEventListener("click", () => console.log("Parent - Capturing"), true);
enfant.addEventListener("click", () => console.log("Enfant - Capturing"), true);
// Clic sur le bouton affiche :
// "Parent - Capturing"  (phase capturing : descend)
// "Enfant - Capturing"  (phase target)

// Ordre complet :
// 1. "Parent - Capturing"
// 2. "Enfant - Capturing"  (target)
// 3. "Enfant - Bubbling"   (target)
// 4. "Parent - Bubbling"
```

### stopPropagation

```javascript
enfant.addEventListener("click", (e) => {
    e.stopPropagation(); // L'evenement ne remonte PAS au parent
    console.log("Enfant seul !");
});

parent.addEventListener("click", () => {
    console.log("Parent : jamais appele si stopPropagation sur enfant");
});
```

---

## 12. Event Delegation

L'event delegation consiste a attacher **un seul** listener sur un parent commun plutot qu'un listener sur chaque enfant. L'evenement "bulle" vers le parent, qui identifie la cible avec `e.target`.

### Pourquoi la delegation ?

```
SANS delegation (mauvais pour les performances) :
───────────────────────────────────────────────
  <ul>
    <li> ←── addEventListener("click", ...)
    <li> ←── addEventListener("click", ...)
    <li> ←── addEventListener("click", ...)
    ... x 100 listeners
  </ul>

AVEC delegation (un seul listener) :
─────────────────────────────────────
  <ul> ←── addEventListener("click", ...)  (UN seul listener)
    <li>     e.target identifie quel <li> a ete clique
    <li>
    <li>
    ... fonctionne meme pour des elements ajoutes dynamiquement !
  </ul>
```

### Implementation

```javascript
const liste = document.getElementById("liste");

// ─── Un seul listener sur le parent ───
liste.addEventListener("click", (e) => {
    // Verifier que le clic est bien sur un <li>
    if (e.target.tagName === "LI") {
        console.log(`Element clique : ${e.target.textContent}`);
        e.target.classList.toggle("fait");
    }
});

// ─── Version plus robuste avec closest() ───
liste.addEventListener("click", (e) => {
    const li = e.target.closest("li");
    if (li && liste.contains(li)) {
        console.log(`Element : ${li.textContent}`);
    }
});

// ─── Les elements ajoutes dynamiquement fonctionnent aussi ! ───
const nouveau = document.createElement("li");
nouveau.textContent = "Nouvel element";
liste.appendChild(nouveau);
// Pas besoin d'ajouter un nouveau listener : la delegation gere deja
```

> [!tip] Analogie
> L'event delegation est comme un standard telephonique d'entreprise. Au lieu de donner un telephone a chaque employe (un listener par element), vous avez un standard central (un listener sur le parent) qui recoit tous les appels et les redirige vers le bon poste (e.target).

---

## 13. Formulaires

### Acceder aux valeurs

```javascript
// HTML :
// <form id="inscription">
//   <input type="text" name="nom" id="nom">
//   <input type="email" name="email" id="email">
//   <input type="number" name="age" id="age">
//   <select name="pays" id="pays">
//     <option value="fr">France</option>
//     <option value="be">Belgique</option>
//   </select>
//   <input type="checkbox" name="accepter" id="accepter">
//   <textarea name="message" id="message"></textarea>
//   <button type="submit">Envoyer</button>
// </form>

const form = document.getElementById("inscription");

// ─── Acceder aux champs ───
const nom = document.getElementById("nom").value;
const email = form.elements.email.value;     // Acces par name
const age = form.elements["age"].value;      // Autre syntaxe
const pays = document.getElementById("pays").value;
const accepte = document.getElementById("accepter").checked; // boolean
const message = document.getElementById("message").value;

// ─── Ecouter les changements en temps reel ───
document.getElementById("nom").addEventListener("input", (e) => {
    console.log("Valeur actuelle :", e.target.value);
});

// ─── Ecouter la perte de focus ───
document.getElementById("email").addEventListener("blur", (e) => {
    if (!e.target.value.includes("@")) {
        e.target.classList.add("erreur");
    }
});
```

### Validation et soumission

```javascript
const form = document.getElementById("inscription");

form.addEventListener("submit", (e) => {
    e.preventDefault(); // Empeche le rechargement de la page

    // ─── Recuperer les donnees ───
    const formData = new FormData(form);
    const donnees = Object.fromEntries(formData);
    console.log(donnees);
    // { nom: "Alice", email: "alice@mail.com", age: "25", pays: "fr", message: "..." }

    // ─── Validation personnalisee ───
    const erreurs = [];

    if (!donnees.nom || donnees.nom.trim().length < 2) {
        erreurs.push("Le nom doit contenir au moins 2 caracteres");
    }

    if (!donnees.email || !donnees.email.includes("@")) {
        erreurs.push("Email invalide");
    }

    const age = Number(donnees.age);
    if (isNaN(age) || age < 0 || age > 150) {
        erreurs.push("Age invalide");
    }

    if (erreurs.length > 0) {
        afficherErreurs(erreurs);
        return;
    }

    // ─── Envoyer les donnees ───
    envoyerFormulaire(donnees);
});

function afficherErreurs(erreurs) {
    const container = document.getElementById("erreurs");
    container.innerHTML = "";
    erreurs.forEach(err => {
        const p = document.createElement("p");
        p.textContent = err;
        p.classList.add("erreur");
        container.appendChild(p);
    });
}
```

---

## 14. localStorage et sessionStorage

Le Web Storage permet de stocker des donnees cle-valeur dans le navigateur.

### Differences

```
+--------------------+----------------------------+----------------------------+
| Propriete          | localStorage               | sessionStorage             |
+--------------------+----------------------------+----------------------------+
| Persistance        | Jusqu'a suppression manuelle| Fermeture onglet/fenetre  |
| Portee             | Meme origine (domaine)     | Meme onglet                |
| Capacite           | ~5-10 Mo                   | ~5-10 Mo                   |
| Partage onglets    | Oui                        | Non                        |
+--------------------+----------------------------+----------------------------+
```

### API (identique pour les deux)

```javascript
// ─── Stocker une valeur (toujours en string !) ───
localStorage.setItem("nom", "Alice");
localStorage.setItem("age", "25");          // Stocke "25" (string)

// ─── Lire une valeur ───
const nom = localStorage.getItem("nom");    // "Alice"
const age = localStorage.getItem("age");    // "25" (string !)

// ─── Supprimer une valeur ───
localStorage.removeItem("age");

// ─── Vider tout le storage ───
localStorage.clear();

// ─── Nombre d'elements ───
localStorage.length;

// ─── Acceder par index ───
localStorage.key(0); // nom de la premiere cle
```

### Stocker des objets avec JSON

```javascript
// ─── Stocker un objet ───
const utilisateur = { nom: "Alice", age: 25, roles: ["admin", "user"] };
localStorage.setItem("utilisateur", JSON.stringify(utilisateur));

// ─── Lire un objet ───
const data = localStorage.getItem("utilisateur");
const user = JSON.parse(data);
console.log(user.nom);    // "Alice"
console.log(user.roles);  // ["admin", "user"]

// ─── Pattern securise (gerer null) ───
function lireJSON(cle, defaut = null) {
    try {
        const data = localStorage.getItem(cle);
        return data ? JSON.parse(data) : defaut;
    } catch (e) {
        console.error(`Erreur lecture ${cle}:`, e);
        return defaut;
    }
}

const prefs = lireJSON("preferences", { theme: "clair", langue: "fr" });
```

> [!warning] Limitations du Web Storage
> - Stocke uniquement des **strings** (pas d'objets natifs)
> - **Synchrone** : bloque le thread principal
> - Pas de date d'expiration (contrairement aux cookies)
> - Pas de securite : accessible par tout JS sur la meme origine
> - **Ne stockez JAMAIS** de tokens, mots de passe ou donnees sensibles en localStorage

---

## 15. Exemple pratique : Todo List dynamique

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Todo List</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 500px; margin: 50px auto; }
        .todo-item { display: flex; align-items: center; padding: 8px;
                     border-bottom: 1px solid #eee; }
        .todo-item.fait { text-decoration: line-through; opacity: 0.6; }
        .todo-text { flex: 1; margin-left: 10px; }
        .btn-suppr { background: #e74c3c; color: white; border: none;
                     padding: 4px 8px; cursor: pointer; border-radius: 3px; }
        #ajout-form { display: flex; gap: 8px; margin-bottom: 20px; }
        #ajout-form input { flex: 1; padding: 8px; }
        #ajout-form button { padding: 8px 16px; }
        .compteur { color: #666; margin-top: 10px; }
    </style>
</head>
<body>
    <h1>Ma Todo List</h1>

    <form id="ajout-form">
        <input type="text" id="nouveau-todo" placeholder="Nouvelle tache..." required>
        <button type="submit">Ajouter</button>
    </form>

    <div id="liste-todos"></div>
    <p class="compteur" id="compteur"></p>

    <script>
        // ─── Etat de l'application ───
        let todos = lireJSON("todos", []);

        // ─── Elements du DOM ───
        const form = document.getElementById("ajout-form");
        const input = document.getElementById("nouveau-todo");
        const listeTodos = document.getElementById("liste-todos");
        const compteur = document.getElementById("compteur");

        // ─── Utilitaire localStorage ───
        function lireJSON(cle, defaut) {
            try {
                const data = localStorage.getItem(cle);
                return data ? JSON.parse(data) : defaut;
            } catch (e) {
                return defaut;
            }
        }

        function sauvegarder() {
            localStorage.setItem("todos", JSON.stringify(todos));
        }

        // ─── Rendu de la liste ───
        function afficherTodos() {
            listeTodos.innerHTML = "";

            todos.forEach((todo, index) => {
                const div = document.createElement("div");
                div.classList.add("todo-item");
                if (todo.fait) div.classList.add("fait");
                div.setAttribute("data-index", index);

                div.innerHTML = `
                    <input type="checkbox" ${todo.fait ? "checked" : ""}>
                    <span class="todo-text">${echapper(todo.texte)}</span>
                    <button class="btn-suppr" data-action="supprimer">X</button>
                `;

                listeTodos.appendChild(div);
            });

            mettreAJourCompteur();
        }

        // ─── Securite : echapper le HTML ───
        function echapper(texte) {
            const div = document.createElement("div");
            div.textContent = texte;
            return div.innerHTML;
        }

        // ─── Mise a jour du compteur ───
        function mettreAJourCompteur() {
            const total = todos.length;
            const faits = todos.filter(t => t.fait).length;
            compteur.textContent = `${faits}/${total} tache(s) terminee(s)`;
        }

        // ─── Ajout d'une tache ───
        form.addEventListener("submit", (e) => {
            e.preventDefault();
            const texte = input.value.trim();
            if (!texte) return;

            todos.push({ texte, fait: false, date: Date.now() });
            sauvegarder();
            afficherTodos();
            input.value = "";
            input.focus();
        });

        // ─── Event delegation sur la liste ───
        listeTodos.addEventListener("click", (e) => {
            const item = e.target.closest(".todo-item");
            if (!item) return;

            const index = Number(item.getAttribute("data-index"));

            // Checkbox : basculer l'etat
            if (e.target.type === "checkbox") {
                todos[index].fait = e.target.checked;
                sauvegarder();
                afficherTodos();
            }

            // Bouton supprimer
            if (e.target.dataset.action === "supprimer") {
                todos.splice(index, 1);
                sauvegarder();
                afficherTodos();
            }
        });

        // ─── Rendu initial ───
        afficherTodos();
    </script>
</body>
</html>
```

> [!example] Concepts utilises dans cet exemple
> - **DOM** : createElement, innerHTML, classList, getAttribute, dataset
> - **Evenements** : addEventListener, submit, click, preventDefault
> - **Event delegation** : un seul listener sur `listeTodos` pour gerer checkbox et suppression
> - **localStorage** : persistance des taches entre les sessions
> - **JSON** : stringify/parse pour stocker des objets
> - **Securite** : echappement HTML pour eviter les failles XSS

---

## Carte Mentale ASCII

```
                      DOM et Evenements
                             │
        ┌────────┬───────────┼───────────┬──────────────┐
        │        │           │           │              │
     Acceder  Modifier    Creer     Evenements     Stockage
        │        │        Supprimer      │              │
   getElementById  textContent  createElement  addEventListener  localStorage
   querySelector   innerHTML   appendChild    Event object      sessionStorage
   querySelectorAll classList  remove         e.target          JSON
        │         style      replaceWith    e.preventDefault   setItem/getItem
        │         setAttribute  cloneNode  e.stopPropagation
        │                        │              │
        │                    Fragment      Bubbling / Capturing
        │                                       │
        │                                  Event Delegation
        │                                       │
        │                                  ┌────┴────┐
        │                                  │         │
   Navigation                          Formulaires  Types
        │                                  │      d'evenements
   parentNode                         FormData      │
   children                           validation  click, submit
   nextElementSibling                             input, keydown
   closest()                                      DOMContentLoaded
```

---

## Exercices

### Exercice 1 : Galerie d'images interactive

Creez une page avec une grille de miniatures. Au clic sur une miniature :
- Affichez l'image en grand dans une zone de preview
- Ajoutez une classe "active" a la miniature selectionnee
- Utilisez l'event delegation (un seul listener sur le conteneur)

### Exercice 2 : Formulaire de contact avec validation

Creez un formulaire avec : nom, email, sujet (select), message (textarea).
- Validez chaque champ en temps reel (`input` event)
- Affichez les erreurs sous chaque champ
- Desactivez le bouton "Envoyer" tant que le formulaire n'est pas valide
- Au submit, affichez un recapitulatif dans un element `<div>`

### Exercice 3 : Bloc-notes persistant

Creez un editeur de texte simple avec :
- Un `<textarea>` dont le contenu est sauvegarde dans `localStorage` a chaque frappe
- Un compteur de mots et de caracteres mis a jour en temps reel
- Un bouton "Effacer" qui vide le texte et le localStorage
- Le texte doit etre restaure au chargement de la page

### Exercice 4 : Navigation par onglets

Creez un systeme d'onglets (tabs) :
- 3 onglets avec chacun un contenu different
- Un seul contenu visible a la fois
- L'onglet actif a une classe CSS "active"
- Utilisez l'event delegation et `data-*` attributes
- Bonus : sauvegardez l'onglet actif dans `sessionStorage`

---

## Liens

- [[01 - Introduction a JavaScript]]
- [[03 - JavaScript Asynchrone]]
- [[04 - JavaScript Moderne ES6+]]
- [[01 - HTML Fondamentaux]]
