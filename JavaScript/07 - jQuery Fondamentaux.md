# jQuery Fondamentaux

jQuery est une bibliothèque JavaScript légère et rapide, publiée en 2006 par John Resig, qui a transformé la façon dont les développeurs web manipulent le DOM, gèrent les événements et effectuent des requêtes AJAX. Pendant plus d'une décennie, elle a été la bibliothèque JavaScript la plus utilisée au monde, présente sur plus de 75 % des sites web du top million Alexa.

Même si JavaScript vanilla moderne (ES6+) et les frameworks comme React ou Vue ont réduit son utilisation dans les nouveaux projets, jQuery reste incontournable pour maintenir des bases de code legacy, comprendre des millions de projets existants, et maîtriser certaines bibliothèques qui en dépendent encore (Bootstrap 4, Slick, DataTables, Select2…).

> [!info] Prérequis
> Ce cours suppose que vous maîtrisez les bases de JavaScript (DOM, événements, fonctions, closures) couvertes dans [[02 - JavaScript DOM et Evenements]] et [[03 - JavaScript Asynchrone]]. Les comparaisons avec JavaScript moderne font référence aux fonctionnalités vues dans [[04 - JavaScript Moderne ES6+]].

---

## 1. Introduction à jQuery

### 1.1 Pourquoi jQuery a révolutionné le web

En 2006, écrire du JavaScript compatible avec tous les navigateurs était un cauchemar. Internet Explorer 6, Firefox 1.5, Safari 2 et Opera avaient chacun leurs bizarreries, leurs bugs et leurs APIs propriétaires. Ajouter un simple gestionnaire d'événement nécessitait du code défensif complexe :

```javascript
// JavaScript vanilla en 2006 — compatibilité cross-browser
function addEventListenerCrossB(element, event, handler) {
    if (element.addEventListener) {
        // Standard W3C (Firefox, Safari, Opera)
        element.addEventListener(event, handler, false);
    } else if (element.attachEvent) {
        // Internet Explorer 6-8
        element.attachEvent("on" + event, handler);
    } else {
        // Dernier recours
        element["on" + event] = handler;
    }
}
```

jQuery a résolu ce problème en une seule ligne :

```javascript
// jQuery — fonctionne partout, même IE6
$(element).on("click", handler);
```

Les problèmes que jQuery résolvait nativement :

| Problème 2006 | Solution jQuery |
|---|---|
| APIs événements différentes par navigateur | `.on()` unifié |
| `XMLHttpRequest` vs `ActiveXObject` (IE) | `$.ajax()` unifié |
| `querySelectorAll` n'existait pas | Sélecteurs CSS puissants via Sizzle |
| Animations CSS non supportées | `.animate()`, `.fadeIn()`, `.slideDown()` |
| `NodeList` non itérable facilement | Collections jQuery avec méthodes built-in |
| JSONP pour les requêtes cross-origin | `$.getJSON()` avec support JSONP |

### 1.2 Histoire et versions

```
2006 — jQuery 1.0    : John Resig publie jQuery à BarCamp NYC
2009 — jQuery 1.3    : Moteur de sélection Sizzle intégré
2011 — jQuery 1.7    : .on() et .off() remplacent .bind()/.live()/.delegate()
2013 — jQuery 2.0    : Abandon du support IE 6-8 (branche 1.x maintenue)
2016 — jQuery 3.0    : Promises/A+ compliance, performances améliorées
2024 — jQuery 3.7.1  : Dernière version stable
```

> [!info] Branches actives
> **jQuery 3.x** — branche principale, IE 9+ minimum.
> **jQuery Migrate** — plugin pour migrer code 1.x/2.x vers 3.x sans tout réécrire.

### 1.3 CDN vs npm

**Via CDN** (solution la plus rapide pour prototyper) :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Mon projet jQuery</title>
</head>
<body>
    <!-- Votre HTML -->

    <!-- jQuery depuis jsDelivr (CDN recommandé) -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.7.1/dist/jquery.min.js"></script>

    <!-- Ou depuis le CDN officiel jQuery -->
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"
            integrity="sha256-/JqT3SQfawRcv/BIHPThkBvs0OEvtFFmqPF/lYI/Cxo="
            crossorigin="anonymous"></script>

    <script>
        // Votre code jQuery ici
        $(document).ready(function() {
            console.log("jQuery prêt !");
        });
    </script>
</body>
</html>
```

**Via npm** (projets Node.js, bundlers Webpack/Vite) :

```bash
# Installation
npm install jquery

# Ou avec yarn
yarn add jquery
```

```javascript
// Import ES module (avec bundler)
import $ from "jquery";

// Import CommonJS (Node.js / anciens bundlers)
const $ = require("jquery");
```

> [!tip] Choisir le bon build
> jQuery propose plusieurs builds :
> - `jquery.js` — version complète non minifiée (developpement, ~290 KB)
> - `jquery.min.js` — version minifiée (production, ~87 KB)
> - `jquery.slim.js` — sans AJAX et effets (si vous n'en avez pas besoin, ~70 KB)
> - `jquery.slim.min.js` — slim minifié (~67 KB)

### 1.4 `$()` et `jQuery()` — la fonction fondamentale

`$` est simplement un alias pour `jQuery`. Les deux sont strictement équivalents :

```javascript
// Ces deux lignes sont identiques
$("#monBouton").hide();
jQuery("#monBouton").hide();
```

`$()` est une **fonction** qui peut recevoir plusieurs types d'arguments :

```javascript
// 1. Un sélecteur CSS (string) → cherche dans le DOM
const paragraphes = $("p");
const titre = $("#titre-principal");
const boutons = $(".btn-primary");

// 2. Un élément DOM → le "wrappe" dans un objet jQuery
const elementNatif = document.getElementById("app");
const elementJQuery = $(elementNatif);

// 3. Du HTML (string commençant par <) → crée un nouvel élément
const nouveauDiv = $("<div>").addClass("conteneur").text("Bonjour");

// 4. Une fonction → raccourci pour $(document).ready()
$(function() {
    console.log("DOM prêt !");
});

// 5. Un tableau ou NodeList → conversion en collection jQuery
const nodeList = document.querySelectorAll(".item");
const collection = $(nodeList);
```

> [!warning] Conflit de noms — `$.noConflict()`
> Certaines bibliothèques (Prototype.js, MooTools) utilisent aussi `$`. En cas de conflit :
> ```javascript
> // Libère $ pour l'autre bibliothèque
> const jq = $.noConflict();
> jq("#monElement").hide(); // Utiliser jq à la place de $
>
> // Ou utiliser une IIFE pour garder $ localement
> (function($) {
>     // Ici $ = jQuery, même si d'autres libs utilisent $ globalement
>     $("#monElement").hide();
> })(jQuery);
> ```

---

## 2. Sélecteurs jQuery

### 2.1 Sélecteurs de base

jQuery utilise le moteur **Sizzle** qui supporte l'intégralité des sélecteurs CSS3 plus des extensions propriétaires.

```javascript
// ─── Sélecteurs CSS standards ───

// Par ID (retourne 0 ou 1 élément)
const header = $("#header");

// Par classe (retourne tous les éléments avec cette classe)
const cards = $(".card");

// Par tag
const liens = $("a");

// Par attribut
const inputsTexte = $("input[type='text']");
const liensExternes = $("a[href^='https']");   // href commence par https
const images = $("img[src$='.png']");           // src finit par .png
const data = $("[data-user]");                  // a l'attribut data-user (quelle que soit sa valeur)

// Combinaisons
const menuActif = $("nav .active");
const titresDansSections = $("section > h2");
const champPrenom = $("form#inscription input[name='prenom']");

// Multiples sélecteurs (union)
const boutons = $("button, .btn, input[type='submit']");
```

### 2.2 Pseudo-sélecteurs CSS vs extensions jQuery

```javascript
// ─── Pseudo-sélecteurs CSS (performants, utilisent querySelectorAll) ───
const premierLi = $("li:first-child");
const dernierLi = $("li:last-child");
const liPairs   = $("li:nth-child(even)");
const liImpairs = $("li:nth-child(odd)");
const checked   = $("input:checked");
const disabled  = $("input:disabled");
const focus     = $("input:focus");
const hover     = $(":hover"); // peu utile en JS

// ─── Extensions jQuery (plus lentes — forcent Sizzle, pas querySelectorAll) ───
const premierJq = $("li:first");          // équiv. :first-child mais différent
const dernierJq = $("li:last");
const pairJq    = $("li:even");           // index 0-based, donc 0, 2, 4...
const impairJq  = $("li:odd");            // index 1, 3, 5...
const visible   = $("div:visible");       // éléments visibles (display != none)
const caches    = $("div:hidden");        // éléments cachés
const animsEnCours = $(":animated");      // éléments en cours d'animation
const contient  = $("p:contains('jQuery')"); // texte contient "jQuery"
const hasChild  = $("div:has(p)");        // div contenant au moins un p
const eq3       = $("li:eq(2)");          // 3e élément (index 0-based)
const gt2       = $("li:gt(2)");          // éléments après l'index 2
const lt2       = $("li:lt(2)");          // éléments avant l'index 2
```

> [!warning] Performances des sélecteurs
> Les extensions jQuery (`:first`, `:even`, `:visible`, `:contains`…) ne peuvent pas être déléguées à `querySelectorAll` et sont donc **plus lentes**. Préférez les équivalents CSS natifs quand ils existent.
>
> ```javascript
> // ❌ Lent — Sizzle doit tout parcourir
> $("li:even")
>
> // ✅ Plus rapide — utilise querySelectorAll
> $("li:nth-child(even)")
>
> // ✅ Encore mieux — filtrer après sélection initiale rapide
> $("li").filter(":even")
> ```

### 2.3 Comparaison : sélecteurs jQuery vs vanilla ES6

```javascript
// ─── Sélection d'un élément par ID ───
// jQuery
const el = $("#monId");

// Vanilla
const el = document.getElementById("monId");
const el = document.querySelector("#monId");

// ─── Sélection de plusieurs éléments ───
// jQuery
const items = $(".item");

// Vanilla
const items = document.querySelectorAll(".item");

// ─── Vérifier si un élément existe ───
// jQuery
if ($("#monId").length > 0) { /* existe */ }
if ($("#monId").length)     { /* existe */ }

// Vanilla
if (document.getElementById("monId")) { /* existe */ }

// ─── Filtrer une collection ───
// jQuery
const actifs = $(".btn").filter(".active");
const nonActifs = $(".btn").not(".active");

// Vanilla
const actifs = [...document.querySelectorAll(".btn")]
    .filter(el => el.classList.contains("active"));
```

### 2.4 Méthodes de filtrage et d'indexation

```javascript
const items = $("li"); // Collection de 5 éléments

// Accéder à un élément par index
items.eq(0);     // 1er élément en objet jQuery
items.eq(-1);    // dernier élément en objet jQuery
items[0];        // 1er élément en DOM natif (pas jQuery)
items.get(0);    // idem — DOM natif
items.get();     // tableau de tous les éléments DOM natifs

// Vérifier le contenu
items.length;           // nombre d'éléments
items.is(".active");    // au moins un élément correspond au sélecteur ?
items.is(":visible");   // au moins un est visible ?

// Filtrage
items.filter(".active");              // garde ceux qui matchent
items.filter(function(index, el) {    // filtrage par fonction
    return $(this).text().length > 5;
});
items.not(".disabled");               // exclut ceux qui matchent

// Slice
items.slice(1, 3);  // éléments d'index 1 et 2

// Itération
items.each(function(index, element) {
    console.log(index, $(element).text());
    // 'this' = l'élément DOM courant
    console.log($(this).text());
});
```

---

## 3. Manipulation du DOM

### 3.1 Lire et écrire le contenu

```javascript
const div = $("#contenu");

// ─── .html() — contenu HTML innerHTML ───
console.log(div.html());                     // lecture
div.html("<strong>Nouveau contenu</strong>"); // écriture
div.html(function(index, ancienHtml) {        // fonction de transformation
    return ancienHtml + " <em>(modifié)</em>";
});

// ─── .text() — contenu texte textContent ───
console.log(div.text());             // lecture — renvoie le texte sans balises
div.text("Texte brut <pas de HTML>"); // écriture — échappe automatiquement le HTML
// → affiché littéralement : Texte brut <pas de HTML>

// ─── .val() — valeur d'un champ de formulaire ───
const valeur = $("input#prenom").val();           // lecture
$("input#prenom").val("Marie");                   // écriture
$("select#pays").val("FR");                       // sélectionne une option
$("input[type='checkbox']").val();                // valeur de l'attribut value
$("input[type='checkbox']").prop("checked");      // état coché (true/false)

// Comparaison vanilla
div.innerHTML;           // jQuery .html()
div.textContent;         // jQuery .text()
input.value;             // jQuery .val()
```

### 3.2 Attributs et propriétés

```javascript
const img = $("img#avatar");
const checkbox = $("input#accepter");

// ─── .attr() — attributs HTML ───
img.attr("src");                          // lecture : "avatar.jpg"
img.attr("src", "nouveau-avatar.jpg");    // écriture
img.attr("alt", "Photo de profil");       // écriture
img.attr({                                // écriture multiple
    src: "avatar.jpg",
    alt: "Avatar",
    width: "100"
});
img.removeAttr("width");                  // suppression

// ─── .prop() — propriétés DOM (différent des attributs HTML) ───
checkbox.prop("checked");          // true/false (état courant)
checkbox.prop("checked", true);    // cocher
checkbox.prop("disabled", false);  // activer
$("input[type='text']").prop("required", true);

// Différence importante attr() vs prop()
// attr("checked") → "checked" si présent dans le HTML source, undefined sinon
// prop("checked") → true/false selon l'état actuel de l'interface

// ─── .data() — attributs data-* ───
// HTML : <div id="user" data-id="42" data-nom="Marie">
const userId = $("#user").data("id");       // 42 (converti en number automatiquement)
const userName = $("#user").data("nom");    // "Marie"
$("#user").data("role", "admin");           // stockage (pas dans le DOM HTML)
$("#user").data({ score: 100, actif: true }); // stockage multiple

// Comparaison vanilla
img.getAttribute("src");             // .attr("src")
img.setAttribute("src", "val");      // .attr("src", "val")
img.removeAttribute("width");        // .removeAttr("width")
checkbox.checked;                    // .prop("checked")
div.dataset.id;                      // .data("id")
```

> [!info] `.attr()` vs `.prop()`
> Utilisez **`.prop()`** pour les états booléens qui peuvent changer dynamiquement :
> `checked`, `disabled`, `selected`, `readOnly`, `multiple`
>
> Utilisez **`.attr()`** pour les attributs HTML statiques :
> `src`, `href`, `id`, `class`, `name`, `type`, `placeholder`

### 3.3 Styles CSS

```javascript
const boite = $(".boite");

// ─── .css() — styles inline ───
boite.css("color");                     // lecture : "rgb(0, 0, 0)"
boite.css("background-color");          // camelCase ou kebab-case
boite.css("backgroundColor");          // les deux fonctionnent
boite.css("color", "red");             // écriture
boite.css("font-size", "18px");
boite.css({                            // écriture multiple
    color: "white",
    backgroundColor: "#333",
    fontSize: "16px",
    padding: "10px 20px"
});

// ─── Dimensions ───
boite.width();           // largeur content (sans padding ni border) en pixels
boite.height();
boite.innerWidth();      // width + padding
boite.innerHeight();
boite.outerWidth();      // width + padding + border
boite.outerHeight();
boite.outerWidth(true);  // width + padding + border + margin

boite.width(200);        // définir la largeur
boite.height("50%");

// ─── Position et défilement ───
boite.offset();          // { top: 150, left: 75 } — relative au document
boite.position();        // { top: 10, left: 20 } — relative au parent positionné
$(window).scrollTop();   // position de défilement vertical
$(window).scrollLeft();
$("html, body").scrollTop(500); // scroller jusqu'à 500px
```

### 3.4 Classes CSS

```javascript
const btn = $(".bouton");

// ─── Ajout et suppression ───
btn.addClass("active");
btn.addClass("active highlight animate");   // plusieurs classes
btn.removeClass("active");
btn.removeClass("active highlight");

// ─── Toggle ───
btn.toggleClass("active");                  // ajoute si absent, supprime si présent
btn.toggleClass("active", true);            // force ajout
btn.toggleClass("active", false);           // force suppression
btn.toggleClass("active", condition);       // conditionnel

// ─── Vérification ───
btn.hasClass("active");   // true/false

// ─── Remplacer toutes les classes ───
btn.attr("class", "nouvelle-classe");

// Comparaison vanilla
el.classList.add("active");
el.classList.remove("active");
el.classList.toggle("active");
el.classList.contains("active");
```

---

## 4. Traversal — Navigation dans le DOM

### 4.1 Parents

```javascript
const item = $("li.actif");

// Remonter d'un niveau
item.parent();                    // parent direct
item.parent(".menu");             // parent direct s'il a la classe .menu

// Remonter de plusieurs niveaux
item.parents();                   // TOUS les ancêtres jusqu'à <html>
item.parents("section");          // tous les ancêtres <section>
item.parentsUntil("body");        // ancêtres jusqu'à (sans inclure) <body>
item.parentsUntil(".conteneur", "div"); // ancêtres div jusqu'à .conteneur

// Trouver l'ancêtre le plus proche qui correspond
item.closest("section");          // remonte et trouve le 1er <section> correspondant
item.closest("[data-component]"); // remonte jusqu'au composant parent
// Note : contrairement à parents(), closest() s'arrête au premier match
```

> [!tip] `.closest()` vs `.parents()`
> - `.parents("X")` retourne **tous** les ancêtres correspondant à X (collection)
> - `.closest("X")` retourne **le plus proche** ancêtre correspondant à X (0 ou 1 élément)
>
> `.closest()` est presque toujours préféré pour la délégation d'événements et la navigation semantique.

### 4.2 Enfants et descendants

```javascript
const liste = $("ul#menu");

// Enfants directs uniquement
liste.children();           // tous les enfants directs
liste.children("li");       // enfants directs qui sont des <li>
liste.children(".active");  // enfants directs avec classe .active

// Tous les descendants
liste.find("a");                    // tous les <a> dans la liste (profondeur illimitée)
liste.find("[data-id]");            // tous les éléments avec data-id
liste.find("span.badge");

// Contenu (inclut les noeuds texte)
liste.contents();           // enfants directs + noeuds texte + commentaires

// Comparaison vanilla
el.parentElement;                           // .parent()
el.closest("section");                      // .closest("section")
el.children;                                // .children() — HTMLCollection
el.querySelectorAll("a");                   // .find("a")
[...el.children].filter(c => c.matches(".active")); // .children(".active")
```

### 4.3 Frères et sœurs (siblings)

```javascript
const item = $("li.actif");

// Frères et sœurs
item.siblings();            // tous les frères/sœurs (pas l'élément lui-même)
item.siblings("li");        // frères/sœurs qui sont des <li>
item.siblings(".important");

// Frères adjacents
item.next();                // frère suivant immédiat
item.next("li");            // frère suivant s'il est un <li>
item.nextAll();             // tous les frères suivants
item.nextAll(".item");
item.nextUntil(".separateur"); // frères suivants jusqu'à (sans inclure) .separateur

item.prev();                // frère précédent immédiat
item.prevAll();             // tous les frères précédents
item.prevUntil(".separateur");
```

### 4.4 Méthodes de manipulation de collection

```javascript
// Ajouter des éléments à la sélection
$("li.actif").add("li.important");   // union des deux sélecteurs
$("li").add(document.getElementById("special"));

// Revenir à la sélection précédente après traversal
$("ul")
    .find("li")          // sélection devient les <li>
    .addClass("item")
    .end()               // revient à la sélection $("ul")
    .addClass("liste");

// Premier et dernier
$("li").first();         // premier élément
$("li").last();          // dernier élément
```

---

## 5. Création et insertion d'éléments

### 5.1 Créer des éléments

```javascript
// ─── Création via string HTML ───
const p = $("<p>");                                    // <p></p>
const p = $("<p>Bonjour le monde</p>");               // avec contenu
const p = $("<p>", { class: "intro", id: "premier" }); // avec attributs

// ─── Création plus complexe (méthodes chaînées) ───
const carte = $("<div>")
    .addClass("card")
    .attr("data-id", 42)
    .css("border", "1px solid #ccc")
    .html(`
        <h3>Titre de la carte</h3>
        <p>Description de la carte</p>
    `);

// ─── Création programmatique ───
function creerBouton(texte, classe) {
    return $("<button>")
        .addClass("btn " + classe)
        .text(texte)
        .attr("type", "button");
}
const btnValider = creerBouton("Valider", "btn-success");
const btnAnnuler = creerBouton("Annuler", "btn-danger");
```

### 5.2 Insérer dans le DOM

```javascript
const liste = $("ul#resultats");
const nouvelItem = $("<li>").text("Nouvel élément").addClass("item");

// ─── Insérer À L'INTÉRIEUR d'un élément ───

// En dernier enfant
liste.append(nouvelItem);               // cible.append(contenu)
nouvelItem.appendTo(liste);             // contenu.appendTo(cible) — même résultat

// En premier enfant
liste.prepend(nouvelItem);
nouvelItem.prependTo(liste);

// ─── Insérer À L'EXTÉRIEUR (autour) d'un élément ───

// Après l'élément (frère suivant)
liste.after("<p>Fin de la liste</p>");
$("<p>Fin de la liste</p>").insertAfter(liste);

// Avant l'élément (frère précédent)
liste.before("<h2>Résultats</h2>");
$("<h2>Résultats</h2>").insertBefore(liste);

// ─── Envelopper ───
$("a.lien-externe").wrap('<span class="lien-container"></span>');
// → <span class="lien-container"><a class="lien-externe">...</a></span>

$("li").wrapAll('<div class="liste-wrapper"></div>');
// → enveloppe TOUS les li dans un seul div

$("li").wrapInner('<span class="item-inner"></span>');
// → enveloppe le CONTENU de chaque li
```

### 5.3 Supprimer et remplacer

```javascript
// ─── Suppression ───
$(".temporaire").remove();          // supprime du DOM (avec événements et data)
$(".temporaire").detach();          // supprime mais conserve événements/data
                                    // utile si on réinsère plus tard

$("ul").empty();                    // vide le contenu (supprime tous les enfants)
// Note : .empty() garde l'élément lui-même, .remove() le supprime

// ─── Remplacement ───
$(".ancien").replaceWith('<div class="nouveau">Nouveau contenu</div>');
$('<div class="nouveau">').replaceAll(".ancien");

// ─── Clone ───
const original = $(".carte:first");
const copie = original.clone();       // clone sans les événements
const copieAvecEvents = original.clone(true); // clone AVEC les événements
copie.appendTo(".galerie");
```

### 5.4 Comparaison avec vanilla ES6

```javascript
// ─── Créer et insérer ───
// jQuery
$("ul").append("<li>Élément</li>");

// Vanilla
const li = document.createElement("li");
li.textContent = "Élément";
document.querySelector("ul").appendChild(li);

// Vanilla moderne (insertAdjacentHTML)
document.querySelector("ul").insertAdjacentHTML("beforeend", "<li>Élément</li>");

// ─── Supprimer ───
// jQuery
$(".temp").remove();

// Vanilla
document.querySelector(".temp").remove(); // Modern DOM API — fonctionne partout depuis ~2015

// ─── Avant/Après ───
// jQuery
$(".cible").before("<p>Avant</p>").after("<p>Après</p>");

// Vanilla
cible.insertAdjacentHTML("beforebegin", "<p>Avant</p>");
cible.insertAdjacentHTML("afterend", "<p>Après</p>");
```

---

## 6. Événements jQuery

### 6.1 `.on()` — la méthode universelle

Depuis jQuery 1.7, `.on()` est la méthode recommandée pour tous les types d'événements. Les anciennes méthodes `.bind()`, `.live()`, `.delegate()` sont dépréciées.

```javascript
// ─── Syntaxe de base ───
$(selector).on(evenement, handler);
$(selector).on(evenement, [selectorDelegation], [data], handler);

// ─── Exemples fondamentaux ───
$("button#envoyer").on("click", function() {
    console.log("Bouton cliqué !");
    console.log(this);          // l'élément DOM cliqué
    console.log($(this));       // l'élément jQuery
});

// ─── L'objet event ───
$("input#recherche").on("keyup", function(event) {
    console.log(event.type);          // "keyup"
    console.log(event.which);         // code de la touche (déprecated, utiliser event.key)
    console.log(event.key);           // "Enter", "a", "Backspace"…
    console.log(event.target);        // l'élément qui a déclenché l'événement
    console.log(event.currentTarget); // l'élément sur lequel .on() est attaché
    console.log(event.pageX);         // position X dans la page
    console.log(event.pageY);
});

// ─── Empêcher les comportements par défaut ───
$("form#contact").on("submit", function(event) {
    event.preventDefault();   // empêche l'envoi du formulaire
    // traitement AJAX ici
});

$("a.no-follow").on("click", function(event) {
    event.preventDefault();   // empêche la navigation
    event.stopPropagation();  // empêche la propagation vers les parents
    // ou les deux en un : return false;
});
```

### 6.2 Raccourcis d'événements courants

```javascript
// Ces raccourcis existent toujours (non dépréciés) mais .on() est préféré
// pour la cohérence et la délégation.

$("button").click(handler);            // .on("click", handler)
$("input").change(handler);            // .on("change", handler)
$("input").focus(handler);             // .on("focus", handler)
$("input").blur(handler);              // .on("blur", handler)
$("input").keyup(handler);             // .on("keyup", handler)
$("input").keydown(handler);           // .on("keydown", handler)
$("form").submit(handler);             // .on("submit", handler)
$("select").change(handler);           // .on("change", handler)
$(window).resize(handler);             // .on("resize", handler)
$(window).scroll(handler);             // .on("scroll", handler)
$(document).ready(handler);            // handler exécuté quand le DOM est prêt

// ─── Événements souris spéciaux ───
$(".menu-item").hover(
    function() { $(this).addClass("survole"); },     // mouseenter
    function() { $(this).removeClass("survole"); }   // mouseleave
);
// Équivalent :
$(".menu-item").on("mouseenter", function() { $(this).addClass("survole"); });
$(".menu-item").on("mouseleave", function() { $(this).removeClass("survole"); });
```

### 6.3 Plusieurs événements sur un seul `.on()`

```javascript
// ─── Plusieurs événements, même handler ───
$("input").on("focus blur", function(event) {
    const estFocus = event.type === "focus";
    $(this).toggleClass("actif", estFocus);
});

// ─── Plusieurs événements, handlers différents ───
$(".zone-upload").on({
    "dragenter": function() { $(this).addClass("drag-over"); },
    "dragleave": function() { $(this).removeClass("drag-over"); },
    "drop":      function(event) {
        event.preventDefault();
        $(this).removeClass("drag-over");
        // traitement du fichier déposé
    }
});
```

### 6.4 `.off()` — supprimer des gestionnaires

```javascript
// ─── Supprimer tous les handlers d'un événement ───
$("button").off("click");

// ─── Supprimer un handler spécifique (doit être une référence nommée) ───
function handleClick() { console.log("cliqué"); }
$("button").on("click", handleClick);
$("button").off("click", handleClick);

// ─── Namespacing — très utile pour off() ciblé ───
$("button").on("click.monPlugin", handleClick);
$("button").on("click.autrePlugin", autreHandler);

$("button").off("click.monPlugin"); // supprime uniquement le handler de monPlugin
$("button").off(".monPlugin");      // supprime TOUS les handlers du namespace monPlugin

// ─── .one() — handler exécuté une seule fois ───
$("button#intro").one("click", function() {
    alert("Vous avez vu ce message une seule fois !");
    // Le handler est automatiquement supprimé après la première exécution
});
```

### 6.5 Déclencher des événements manuellement

```javascript
// ─── .trigger() ───
$("form#contact").trigger("submit");      // déclenche + exécute le handler
$("input#date").trigger("change");

// ─── .triggerHandler() ───
$("input").triggerHandler("focus");       // exécute le handler SANS le comportement natif
// (ne met pas vraiment le focus sur l'input)

// Passer des données supplémentaires
$("button").trigger("click", [42, "extra-data"]);
$("button").on("click", function(event, id, data) {
    console.log(id, data); // 42, "extra-data"
});

// Événements personnalisés
$(".panier").trigger("produitAjoute", [{ id: 5, nom: "T-shirt", prix: 29 }]);
$(".panier").on("produitAjoute", function(event, produit) {
    console.log(`${produit.nom} ajouté au panier`);
});
```

---

## 7. Event Delegation

### 7.1 Le problème sans délégation

```javascript
// Scénario : liste de tâches où l'utilisateur peut ajouter des items dynamiquement

// ❌ PROBLÈME : les handlers sont attachés aux éléments existants au moment du .on()
// Les nouveaux éléments ajoutés dynamiquement n'ont PAS ces handlers
$(".tache").on("click", function() {
    $(this).toggleClass("terminee");
});

// Si on ajoute dynamiquement un nouvel item...
$("#ajouter-tache").on("click", function() {
    $("<li>").addClass("tache").text("Nouvelle tâche").appendTo("ul#liste");
    // La nouvelle tâche n'aura PAS le handler .click() !
});
```

### 7.2 La solution : délégation d'événements

```javascript
// ✅ SOLUTION : attacher le handler à un élément PARENT stable
// La syntaxe .on(event, selector, handler) active la délégation
$("ul#liste").on("click", ".tache", function() {
    $(this).toggleClass("terminee");
    // 'this' = l'élément .tache cliqué (pas ul#liste)
});

// Maintenant, même les .tache ajoutés dynamiquement répondront au clic !
// Le mécanisme :
// 1. Le clic se propage depuis .tache jusqu'à ul#liste (bubbling)
// 2. jQuery vérifie si l'élément cible correspond au sélecteur ".tache"
// 3. Si oui, exécute le handler avec this = l'élément .tache
```

```
┌─────────────────────────────────────────┐
│  ul#liste  ← handler attaché ici         │
│  ┌───────────────────────────────────┐   │
│  │ li.tache   ← clic ici             │   │
│  │ "Acheter du lait"                 │   │
│  └───────────────────────────────────┘   │
│  ┌───────────────────────────────────┐   │
│  │ li.tache.terminee                 │   │
│  │ "Faire la vaisselle" ✓            │   │
│  └───────────────────────────────────┘   │
│                                         │
│  Bubbling : clic ──▶ li.tache ──▶ ul    │
│  jQuery : "li.tache correspond au       │
│  sélecteur ? oui → exécuter handler"    │
└─────────────────────────────────────────┘
```

### 7.3 Avantages de la délégation

```javascript
// Avantage 1 : éléments dynamiques (vu ci-dessus)

// Avantage 2 : performance — UN seul handler pour N éléments
// ❌ Inefficace : 1000 handlers
$("tr.ligne-donnee").on("click", handler); // attache 1000 handlers si 1000 lignes

// ✅ Efficace : 1 seul handler
$("tbody#donnees").on("click", "tr.ligne-donnee", handler);

// Avantage 3 : handlers toujours actifs même après remplacement du contenu
$("div#resultats").on("click", ".btn-detail", function() {
    const id = $(this).data("id");
    afficherDetail(id);
});

// Même si on remplace tout le HTML de #resultats avec $.ajax,
// les boutons .btn-detail dans les nouveaux résultats fonctionneront toujours.

// ─── Délégation sur document (fallback ultime) ───
// Quand aucun parent stable n'existe, déléguer sur document
$(document).on("click", ".modal-close", function() {
    $(this).closest(".modal").hide();
});
// Note : éviter de trop déléguer sur document — tous les clics remontent jusqu'à lui
```

> [!warning] Ne pas déléguer inutilement
> La délégation sur `document` ou `body` fait remonter TOUS les événements jusqu'en haut. Pour de bonnes performances, déléguer sur le **parent le plus proche et stable** de vos éléments dynamiques.

---

## 8. Effets et Animations

### 8.1 Afficher et cacher

```javascript
// ─── Basique ───
$(".panneau").show();                 // display: block (ou valeur précédente)
$(".panneau").hide();                 // display: none
$(".panneau").toggle();               // alterne show/hide

// ─── Avec durée (en millisecondes ou "slow"/"fast") ───
$(".panneau").show(400);              // 400ms
$(".panneau").hide("slow");           // 600ms
$(".panneau").toggle("fast");         // 200ms
$(".panneau").show(600, "swing");     // durée + easing
$(".panneau").show(600, "linear");

// ─── Avec callback (exécuté à la fin de l'animation) ───
$(".modal").show(300, function() {
    $(this).find("input:first").focus();
    console.log("Modal affichée !");
});
```

### 8.2 Fondu (Fade)

```javascript
// ─── Fondu complet ───
$(".image").fadeIn(400);         // opacité 0 → 1
$(".image").fadeOut(400);        // opacité 1 → 0 + display:none
$(".image").fadeToggle(400);

// ─── Fondu vers une opacité spécifique ───
$(".overlay").fadeTo(500, 0.7);  // fondu vers opacité 0.7 (l'élément reste visible)
$(".overlay").fadeTo(500, 0);    // fondu vers 0 (mais PAS display:none)

// ─── Avec callback ───
$(".alerte").fadeOut(600, function() {
    $(this).remove(); // supprime après le fondu
});
```

### 8.3 Glissement (Slide)

```javascript
// ─── Accordéon vertical ───
$(".contenu-accordeon").slideDown(400);   // hauteur 0 → auto
$(".contenu-accordeon").slideUp(400);     // hauteur auto → 0 + display:none
$(".contenu-accordeon").slideToggle(400); // alterne

// Exemple pratique : accordéon FAQ
$(".faq-question").on("click", function() {
    const reponse = $(this).next(".faq-reponse");
    reponse.slideToggle(300);
    $(this).toggleClass("ouvert");
});
```

### 8.4 `.animate()` — animations personnalisées

```javascript
// ─── Syntaxe ───
$(selector).animate(proprietes, [duree], [easing], [callback]);

// ─── Exemples ───
$(".boite").animate({
    width: "300px",
    height: "200px",
    opacity: 0.8,
    marginLeft: "50px"
}, 600);

// ─── Valeurs relatives ───
$(".compteur").animate({
    left: "+=100px",   // déplacer de 100px vers la droite
    top: "-=50px"      // remonter de 50px
}, 400);

// ─── Easing ───
// jQuery core : "swing" (défaut, légèrement courbé) ou "linear"
// jQuery UI ajoute : "easeInOutBack", "easeInBounce", etc.
$(".element").animate({ left: "200px" }, 500, "linear");

// ─── Séquence d'animations ───
$(".balle")
    .animate({ left: "200px" }, 400)
    .animate({ top: "100px" }, 400)
    .animate({ left: "0px" }, 400)
    .animate({ top: "0px" }, 400);
// Chaque .animate() est mis en file d'attente (queue)

// ─── Arrêter les animations ───
$(".element").stop();           // arrête l'animation en cours
$(".element").stop(true);       // vide la file + arrête
$(".element").stop(true, true); // vide la file + saute à la fin
$(".element").finish();         // termine immédiatement toutes les animations en attente
```

> [!tip] Préférer CSS pour les animations en 2024
> Pour les animations de performance, utilisez les **transitions CSS** et les **animations CSS** — elles s'exécutent sur le GPU via le compositor du navigateur, alors que `.animate()` modifie le DOM via JavaScript (thread principal, moins performant).
>
> ```javascript
> // ❌ Moins performant
> $(".carte").animate({ transform: "scale(1.05)" }, 200);
>
> // ✅ Plus performant
> $(".carte").addClass("agrandie");
> // CSS : .agrandie { transform: scale(1.05); transition: transform 200ms ease; }
> ```

### 8.5 Contrôle de la file d'attente

```javascript
// jQuery maintient une file d'attente "fx" pour les animations
$(".el").queue();           // voir la file
$(".el").dequeue();         // exécuter le prochain élément de la file
$(".el").clearQueue();      // vider la file

// Insérer une pause dans une séquence
$(".el")
    .animate({ left: "200px" }, 400)
    .delay(1000)             // pause de 1 seconde
    .animate({ left: "0px" }, 400);

// Insérer une fonction personnalisée dans la file
$(".el")
    .animate({ left: "200px" }, 400)
    .queue(function(next) {
        $(this).addClass("etape-2");
        next(); // IMPORTANT : appeler next() pour continuer la file
    })
    .animate({ opacity: 0.5 }, 300);
```

---

## 9. AJAX avec jQuery

### 9.1 `$.ajax()` — la méthode complète

```javascript
// ─── Syntaxe complète ───
$.ajax({
    url: "https://api.exemple.com/utilisateurs",
    method: "GET",                  // "GET", "POST", "PUT", "DELETE"…
    dataType: "json",               // type de données attendu en retour
    contentType: "application/json", // type des données envoyées
    data: { page: 1, limite: 10 }, // paramètres de la requête
    headers: {
        "Authorization": "Bearer " + token,
        "X-Custom-Header": "valeur"
    },
    timeout: 5000,                  // timeout en ms
    beforeSend: function(xhr) {
        // Appelé avant l'envoi — utile pour ajouter un spinner
        $("#spinner").show();
    },
    success: function(data, statut, xhr) {
        // data = réponse parsée (objet JS si dataType:"json")
        console.log("Succès :", data);
        afficherUtilisateurs(data);
    },
    error: function(xhr, statut, erreur) {
        console.error("Erreur HTTP :", xhr.status, erreur);
        console.error("Corps de la réponse :", xhr.responseText);
        afficherErreur(`Erreur ${xhr.status} : ${erreur}`);
    },
    complete: function(xhr, statut) {
        // Toujours appelé, succès ou erreur
        $("#spinner").hide();
    }
});
```

### 9.2 Méthodes raccourcies

```javascript
// ─── $.get() ───
$.get("https://api.exemple.com/produits", function(data) {
    console.log(data);
});

// Avec paramètres
$.get("https://api.exemple.com/produits", { categorie: "tech", page: 1 }, function(data) {
    afficherProduits(data);
});

// ─── $.post() ───
$.post("https://api.exemple.com/utilisateurs", {
    nom: "Marie",
    email: "marie@example.com",
    motDePasse: "secret123"
}, function(data) {
    console.log("Utilisateur créé :", data);
});

// ─── $.getJSON() ───
$.getJSON("https://api.exemple.com/config.json", function(config) {
    initialiserApp(config);
});

// ─── .load() — charger du HTML dans un élément ───
$("#contenu-dynamique").load("fragments/sidebar.html");
// Avec sélecteur pour extraire une partie du document chargé
$("#contenu-dynamique").load("page.html #section-principale");
// Avec callback
$("#contenu-dynamique").load("fragments/sidebar.html", function(response, status) {
    if (status === "error") {
        $(this).html("<p>Erreur de chargement</p>");
    }
});
```

### 9.3 `.done()`, `.fail()`, `.always()` — interface Promise

Depuis jQuery 1.5, `$.ajax()` retourne un objet **Deferred** compatible avec l'interface Promise :

```javascript
const requete = $.ajax({
    url: "https://jsonplaceholder.typicode.com/posts/1",
    method: "GET",
    dataType: "json"
});

requete
    .done(function(data) {
        console.log("Article chargé :", data.title);
        $("#titre").text(data.title);
        $("#corps").text(data.body);
    })
    .fail(function(xhr, statut, erreur) {
        console.error("Échec :", erreur);
        $("#erreur").text(`Impossible de charger l'article : ${erreur}`).show();
    })
    .always(function() {
        $("#chargement").hide();
    });

// ─── Chaining de requêtes séquentielles ───
$.get("/api/utilisateur/1")
    .done(function(utilisateur) {
        return $.get(`/api/commandes?userId=${utilisateur.id}`);
    })
    .done(function(commandes) {
        afficherCommandes(commandes);
    })
    .fail(function(xhr) {
        afficherErreur(xhr.status);
    });
```

### 9.4 Exemple concret — Formulaire AJAX

```javascript
$("#formulaire-contact").on("submit", function(event) {
    event.preventDefault();

    const formulaire = $(this);
    const boutonEnvoi = formulaire.find("button[type='submit']");
    const messageResultat = $("#message-resultat");

    // Sérialiser le formulaire automatiquement
    const donnees = formulaire.serialize();
    // → "nom=Marie&email=marie%40example.com&message=Bonjour"

    // Ou en objet
    const donneesObj = formulaire.serializeArray();
    // → [{ name: "nom", value: "Marie" }, { name: "email", value: "..." }]

    // Désactiver le bouton et afficher le loader
    boutonEnvoi.prop("disabled", true).text("Envoi en cours…");
    messageResultat.removeClass("succes erreur").hide();

    $.ajax({
        url: "/api/contact",
        method: "POST",
        data: donnees,
        dataType: "json"
    })
    .done(function(reponse) {
        messageResultat
            .addClass("succes")
            .text(reponse.message || "Message envoyé avec succès !")
            .fadeIn(300);
        formulaire[0].reset(); // réinitialiser le formulaire natif
    })
    .fail(function(xhr) {
        let message = "Une erreur est survenue. Veuillez réessayer.";
        if (xhr.status === 422) {
            const erreurs = xhr.responseJSON?.erreurs;
            if (erreurs) message = Object.values(erreurs).join(", ");
        }
        messageResultat
            .addClass("erreur")
            .text(message)
            .fadeIn(300);
    })
    .always(function() {
        boutonEnvoi.prop("disabled", false).text("Envoyer");
    });
});
```

### 9.5 Configuration globale AJAX

```javascript
// Réglages par défaut pour toutes les requêtes $.ajax()
$.ajaxSetup({
    headers: {
        "X-CSRF-Token": $('meta[name="csrf-token"]').attr("content")
    },
    timeout: 10000,
    dataType: "json"
});

// ─── Hooks globaux AJAX ───
$(document)
    .ajaxStart(function() {
        $("#spinner-global").show(); // au moins 1 requête en cours
    })
    .ajaxStop(function() {
        $("#spinner-global").hide(); // toutes les requêtes terminées
    })
    .ajaxError(function(event, xhr, settings, erreur) {
        console.error(`Erreur AJAX sur ${settings.url} :`, erreur);
        if (xhr.status === 401) {
            window.location.href = "/connexion";
        }
    });
```

---

## 10. Promises et Deferred Objects (héritage)

### 10.1 Deferred — créer vos propres promises jQuery

```javascript
// $.Deferred() est l'implémentation pre-ES6 des Promises
// Permet de créer des opérations asynchrones personnalisées

function chargerImage(url) {
    const deferred = $.Deferred();
    const img = new Image();

    img.onload = function() {
        deferred.resolve(img);         // équiv. Promise resolve
    };
    img.onerror = function() {
        deferred.reject("Impossible de charger : " + url); // équiv. Promise reject
    };
    img.src = url;

    return deferred.promise(); // expose .done()/.fail()/.always() sans .resolve()/.reject()
}

chargerImage("https://exemple.com/photo.jpg")
    .done(function(img) {
        $("body").append(img);
        console.log("Image chargée :", img.width + "x" + img.height);
    })
    .fail(function(erreur) {
        console.error(erreur);
    });
```

### 10.2 `$.when()` — attendre plusieurs opérations

```javascript
// ─── Attendre plusieurs requêtes AJAX ───
$.when(
    $.get("/api/utilisateurs"),
    $.get("/api/produits"),
    $.get("/api/config")
).done(function(reponseUsers, reponseProduits, reponseConfig) {
    // Chaque argument est [data, status, xhr]
    const users    = reponseUsers[0];
    const produits = reponseProduits[0];
    const config   = reponseConfig[0];

    initialiserApp(users, produits, config);
}).fail(function(xhr) {
    console.error("Au moins une requête a échoué :", xhr.status);
});

// ─── Avec des Deferreds personnalisés ───
const animationTerminee = $.Deferred();
const donneesChargees   = $.Deferred();

$(".splash").fadeOut(1000, function() {
    animationTerminee.resolve();
});

$.get("/api/donnees-initiales").done(function(data) {
    traiterDonnees(data);
    donneesChargees.resolve();
});

$.when(animationTerminee, donneesChargees).done(function() {
    // Les deux sont terminés — afficher l'application
    $(".app").fadeIn(300);
});
```

> [!info] Deferred vs Promises natives ES6
> Depuis jQuery 3.0, les Deferreds sont alignés sur la spécification **Promises/A+**, ce qui les rend interopérables avec les Promises natives et `async/await`.
>
> ```javascript
> // Convertir une Promise native en compatible jQuery
> const promiseNative = fetch("/api/data").then(r => r.json());
> $.when(promiseNative).done(function(data) { ... });
>
> // Utiliser await sur un Deferred jQuery
> const data = await $.get("/api/data");
> ```

---

## 11. Plugins jQuery

### 11.1 Utiliser un plugin existant

Les plugins jQuery étendent le prototype `$.fn` pour ajouter des méthodes à toutes les collections jQuery. L'écosystème est immense — plus de 3 000 plugins sur le registre officiel.

```html
<!-- Structure typique : jQuery d'abord, plugin ensuite -->
<script src="jquery.min.js"></script>
<script src="slick.min.js"></script>
<link rel="stylesheet" href="slick.css">

<script>
    // Initialisation du plugin
    $(".carousel").slick({
        slidesToShow: 3,
        slidesToScroll: 1,
        autoplay: true,
        autoplaySpeed: 2000,
        arrows: true,
        dots: true,
        responsive: [
            {
                breakpoint: 768,
                settings: { slidesToShow: 1 }
            }
        ]
    });
</script>
```

### 11.2 Plugins populaires en 2024

| Plugin | Usage | Taille | Statut |
|---|---|---|---|
| **Slick** | Carrousel/slider | 25 KB | Actif |
| **DataTables** | Tableaux interactifs | 80 KB | Actif |
| **Select2** | Champs select avancés | 70 KB | Actif |
| **Chosen** | Champs select | 30 KB | Déprécié |
| **Fancybox** | Lightbox images/vidéos | 150 KB | Actif |
| **jQuery UI** | Widgets UI complets | 250 KB | Maintenance |
| **jQuery Validate** | Validation de formulaires | 22 KB | Actif |
| **Colorbox** | Lightbox légère | 10 KB | Maintenance |
| **Magnific Popup** | Popup responsive | 20 KB | Maintenance |
| **Waypoints** | Scroll triggers | 7 KB | Actif |

### 11.3 Exemple complet : DataTables

```html
<link rel="stylesheet" href="https://cdn.datatables.net/1.13.7/css/jquery.dataTables.min.css">

<table id="tableau-employes">
    <thead>
        <tr>
            <th>Nom</th>
            <th>Poste</th>
            <th>Bureau</th>
            <th>Salaire</th>
        </tr>
    </thead>
</table>

<script src="jquery.min.js"></script>
<script src="https://cdn.datatables.net/1.13.7/js/jquery.dataTables.min.js"></script>
<script>
    $("#tableau-employes").DataTable({
        ajax: "/api/employes",           // chargement AJAX
        columns: [
            { data: "nom" },
            { data: "poste" },
            { data: "bureau" },
            {
                data: "salaire",
                render: function(data) {
                    return data.toLocaleString("fr-FR", {
                        style: "currency",
                        currency: "EUR"
                    });
                }
            }
        ],
        pageLength: 25,
        language: {
            url: "//cdn.datatables.net/plug-ins/1.13.7/i18n/fr-FR.json"
        },
        order: [[3, "desc"]],   // tri par salaire décroissant par défaut
        responsive: true
    });
</script>
```

### 11.4 Créer votre propre plugin jQuery

```javascript
// ─── Structure d'un plugin jQuery robuste ───
(function($) {
    "use strict";

    // Nom du plugin
    const PLUGIN_NAME = "compteurAnimé";
    const DEFAULT_OPTIONS = {
        debut: 0,
        duree: 2000,
        separateurMilliers: " ",
        callback: null
    };

    // Définition du plugin
    $.fn[PLUGIN_NAME] = function(options) {
        // Fusion des options avec les défauts
        const settings = $.extend({}, DEFAULT_OPTIONS, options);

        // Itérer sur chaque élément de la collection (this = collection jQuery)
        return this.each(function() {
            const $element = $(this);
            const valeurFinale = parseFloat($element.text().replace(/\s/g, "")) || 0;

            $({ compteur: settings.debut }).animate(
                { compteur: valeurFinale },
                {
                    duration: settings.duree,
                    easing: "swing",
                    step: function() {
                        const valeur = Math.ceil(this.compteur);
                        $element.text(
                            valeur.toLocaleString("fr-FR").replace(/ /g, settings.separateurMilliers)
                        );
                    },
                    complete: function() {
                        $element.text(
                            valeurFinale.toLocaleString("fr-FR").replace(/ /g, settings.separateurMilliers)
                        );
                        if (typeof settings.callback === "function") {
                            settings.callback.call($element[0]);
                        }
                    }
                }
            );
        });
    };

})(jQuery);

// ─── Utilisation ───
// HTML : <span class="compteur">4250</span>
$(".compteur").compteurAnimé({
    debut: 0,
    duree: 3000,
    callback: function() {
        $(this).addClass("animation-terminee");
    }
});
```

---

## 12. Chaining (Enchaînement)

### 12.1 Principe du chaining

La majorité des méthodes jQuery retournent l'**objet jQuery lui-même**, ce qui permet d'enchaîner plusieurs méthodes sur la même sélection sans répéter le sélecteur.

```javascript
// ─── Sans chaining (répétitif) ───
const $btn = $(".bouton-principal");
$btn.addClass("actif");
$btn.css("color", "white");
$btn.css("background", "#007bff");
$btn.text("Cliquez ici");
$btn.show();

// ─── Avec chaining (élégant) ───
$(".bouton-principal")
    .addClass("actif")
    .css({ color: "white", background: "#007bff" })
    .text("Cliquez ici")
    .show();
```

### 12.2 Chaining avec traversal

```javascript
// Manipuler l'élément ET ses voisins en une chaîne
$(".alerte")
    .text("Erreur de connexion")
    .addClass("alerte-danger")
    .parent()
        .addClass("a-une-erreur")
    .end()          // revient à .alerte
    .show(300);

// ─── Exemple real-world : mise à jour d'un formulaire ───
$("#form-login")
    .find("input")
        .val("")
        .prop("disabled", false)
    .end()
    .find(".message-erreur")
        .text("")
        .hide()
    .end()
    .find("button[type='submit']")
        .prop("disabled", false)
        .text("Se connecter");
```

### 12.3 Quand NE PAS chaîner

```javascript
// ❌ Chaining illisible — trop de logique dans une chaîne
$("ul").find("li").not(".disabled").filter(function() {
    return $(this).data("role") === "admin";
}).each(function() { ... }).addClass("selectionne").end().end().toggleClass("liste-admin");

// ✅ Préférer des variables intermédiaires quand la chaîne devient complexe
const $liste = $("ul");
const $itemsActifs = $liste.find("li").not(".disabled");
const $admins = $itemsActifs.filter(function() {
    return $(this).data("role") === "admin";
});
$admins.each(function() { /* traitement */ });
$admins.addClass("selectionne");
$liste.toggleClass("liste-admin");
```

---

## 13. `$(document).ready()` vs DOMContentLoaded

### 13.1 Le problème

Quand le navigateur parse le HTML, les scripts se chargent et s'exécutent avant que tout le DOM ne soit construit. Si votre script essaie d'accéder à `#mon-bouton` mais que cet élément n'est pas encore dans le DOM → `null`, erreur.

```javascript
// ❌ Risque : le script s'exécute avant que le DOM soit complet
// Si ce script est dans <head>, #mon-bouton n'existe pas encore
document.getElementById("mon-bouton").addEventListener("click", handler); // TypeError: null
```

### 13.2 Solutions : 4 approches comparées

```javascript
// ─── 1. jQuery (syntaxe longue) ───
$(document).ready(function() {
    // Exécuté quand le DOM est construit (pas les images, CSS)
    $("#mon-bouton").on("click", handler);
});

// ─── 2. jQuery (syntaxe courte recommandée) ───
$(function() {
    $("#mon-bouton").on("click", handler);
});

// ─── 3. Vanilla — DOMContentLoaded ───
document.addEventListener("DOMContentLoaded", function() {
    document.getElementById("mon-bouton").addEventListener("click", handler);
});

// ─── 4. Script en bas de page (avant </body>) ───
// La solution la plus simple et la plus performante :
// Les éléments DOM au-dessus existent déjà quand le script s'exécute
<body>
    <div id="app">...</div>
    <!-- Tout le HTML d'abord -->
    <script>
        document.getElementById("mon-bouton").addEventListener("click", handler);
    </script>
</body>

// ─── 5. Attribut defer sur le script (recommandé ES6) ───
// <script src="app.js" defer></script>
// defer = exécuter après le parsing HTML, avant DOMContentLoaded
// Équivalent fonctionnel à mettre le script en bas de page
```

### 13.3 Différence entre ready() et window.onload

```javascript
// $(document).ready() / DOMContentLoaded
// → DOM prêt, mais les images/CSS/iframes peuvent encore charger
// → Se déclenche tôt — idéal pour attacher des handlers

// $(window).on("load") / window.onload
// → TOUT est chargé (DOM + images + CSS + iframes)
// → Se déclenche tard — utile pour les calculs basés sur les images

$(window).on("load", function() {
    const hauteurImage = $("img#hero").height(); // correct — image chargée
    ajusterLayout(hauteurImage);
});

// ─── Enregistrement multiple ───
// jQuery .ready() peut être appelé plusieurs fois — tous s'exécutent
$(function() { initialiserNavigation(); });
$(function() { initialiserFormulaireContact(); });
$(function() { initialiserCarousel(); });

// DOMContentLoaded ne peut être déclenché qu'une fois — mais on peut
// ajouter autant d'écouteurs qu'on veut :
document.addEventListener("DOMContentLoaded", initialiserNavigation);
document.addEventListener("DOMContentLoaded", initialiserFormulaireContact);
```

---

## 14. Migration vers JavaScript Vanilla Moderne

### 14.1 Tableau d'équivalences complet

| jQuery | Vanilla ES6+ | Notes |
|---|---|---|
| `$(selector)` | `document.querySelector(selector)` | 1 élément |
| `$(selector)` | `document.querySelectorAll(selector)` | Tous les éléments |
| `$(el)` | *(déjà un élément DOM)* | Wrapping inutile |
| `$("<div>")` | `document.createElement("div")` | |
| `.html(val)` | `el.innerHTML = val` | |
| `.html()` | `el.innerHTML` | |
| `.text(val)` | `el.textContent = val` | |
| `.text()` | `el.textContent` | |
| `.val(val)` | `el.value = val` | |
| `.val()` | `el.value` | |
| `.attr("src", v)` | `el.setAttribute("src", v)` | |
| `.attr("src")` | `el.getAttribute("src")` | |
| `.prop("checked")` | `el.checked` | |
| `.css("color", v)` | `el.style.color = v` | |
| `.css("color")` | `getComputedStyle(el).color` | |
| `.addClass("x")` | `el.classList.add("x")` | |
| `.removeClass("x")` | `el.classList.remove("x")` | |
| `.toggleClass("x")` | `el.classList.toggle("x")` | |
| `.hasClass("x")` | `el.classList.contains("x")` | |
| `.append(el)` | `parent.appendChild(el)` | |
| `.append(html)` | `parent.insertAdjacentHTML("beforeend", html)` | |
| `.prepend(html)` | `parent.insertAdjacentHTML("afterbegin", html)` | |
| `.before(html)` | `el.insertAdjacentHTML("beforebegin", html)` | |
| `.after(html)` | `el.insertAdjacentHTML("afterend", html)` | |
| `.remove()` | `el.remove()` | Modern DOM |
| `.empty()` | `el.innerHTML = ""` | |
| `.clone()` | `el.cloneNode(true)` | |
| `.parent()` | `el.parentElement` | |
| `.children()` | `el.children` | HTMLCollection |
| `.find("x")` | `el.querySelectorAll("x")` | |
| `.closest("x")` | `el.closest("x")` | Natif depuis 2017 |
| `.next()` | `el.nextElementSibling` | |
| `.prev()` | `el.previousElementSibling` | |
| `.siblings()` | `[...el.parentElement.children].filter(c => c !== el)` | |
| `.on("click", fn)` | `el.addEventListener("click", fn)` | |
| `.off("click", fn)` | `el.removeEventListener("click", fn)` | |
| `.trigger("click")` | `el.dispatchEvent(new Event("click"))` | |
| `.each(fn)` | `els.forEach(fn)` | après `[...nodeList]` |
| `.filter(sel)` | `[...els].filter(el => el.matches(sel))` | |
| `.is(sel)` | `el.matches(sel)` | |
| `$.ajax(opts)` | `fetch(url, opts)` | |
| `$.get(url, fn)` | `fetch(url).then(r=>r.json()).then(fn)` | |
| `$.post(url, data, fn)` | `fetch(url, {method:"POST", body: JSON.stringify(data)}).then(...)` | |
| `$.extend(a, b)` | `{ ...a, ...b }` ou `Object.assign({}, a, b)` | |
| `$.isArray(v)` | `Array.isArray(v)` | |
| `$.trim(str)` | `str.trim()` | |
| `$.type(v)` | `typeof v` / `Array.isArray(v)` | |
| `$(document).ready(fn)` | `document.addEventListener("DOMContentLoaded", fn)` | ou `defer` |

### 14.2 AJAX : jQuery vs Fetch API

```javascript
// ─── GET avec jQuery ───
$.get("/api/articles")
    .done(function(articles) {
        afficher(articles);
    })
    .fail(function(xhr) {
        console.error(xhr.status);
    });

// ─── GET avec Fetch (vanilla) ───
fetch("/api/articles")
    .then(function(response) {
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    })
    .then(function(articles) {
        afficher(articles);
    })
    .catch(function(erreur) {
        console.error(erreur.message);
    });

// ─── GET avec Fetch + async/await ───
async function chargerArticles() {
    try {
        const response = await fetch("/api/articles");
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const articles = await response.json();
        afficher(articles);
    } catch (erreur) {
        console.error(erreur.message);
    }
}

// ─── POST avec jQuery ───
$.ajax({
    url: "/api/articles",
    method: "POST",
    contentType: "application/json",
    data: JSON.stringify({ titre: "Mon article", contenu: "..." }),
    dataType: "json"
}).done(function(nouvelArticle) {
    console.log("Créé :", nouvelArticle.id);
});

// ─── POST avec Fetch ───
async function creerArticle(titre, contenu) {
    const response = await fetch("/api/articles", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            "X-CSRF-Token": document.querySelector('meta[name="csrf-token"]').content
        },
        body: JSON.stringify({ titre, contenu })
    });
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
}
```

### 14.3 Événements : migration pas à pas

```javascript
// ─── Avant (jQuery) ───
$(document).ready(function() {

    // Delegation
    $(document).on("click", ".btn-supprimer", function() {
        const id = $(this).data("id");
        if (confirm("Supprimer ?")) {
            supprimerElement(id);
        }
    });

    // Formulaire
    $("#form-recherche").on("submit", function(e) {
        e.preventDefault();
        rechercherProduits($(this).find("input[name='q']").val());
    });

    // Classe dynamique
    $(".accordion-titre").on("click", function() {
        $(this).toggleClass("ouvert")
               .next(".accordion-contenu")
               .slideToggle(300);
    });
});

// ─── Après (Vanilla ES6) ───
document.addEventListener("DOMContentLoaded", () => {

    // Delegation
    document.addEventListener("click", (e) => {
        const btn = e.target.closest(".btn-supprimer");
        if (!btn) return;
        const id = btn.dataset.id;
        if (confirm("Supprimer ?")) {
            supprimerElement(id);
        }
    });

    // Formulaire
    document.getElementById("form-recherche")?.addEventListener("submit", (e) => {
        e.preventDefault();
        const q = e.currentTarget.querySelector("input[name='q']").value;
        rechercherProduits(q);
    });

    // Classe dynamique (animation CSS à la place de .slideToggle)
    document.querySelectorAll(".accordion-titre").forEach(titre => {
        titre.addEventListener("click", function() {
            this.classList.toggle("ouvert");
            const contenu = this.nextElementSibling;
            contenu.classList.toggle("ouvert");
            // CSS : .accordion-contenu { max-height: 0; overflow: hidden; transition: max-height 300ms ease; }
            //       .accordion-contenu.ouvert { max-height: 1000px; }
        });
    });
});
```

---

## 15. Quand encore utiliser jQuery en 2024

### 15.1 Contextes légitimes

```
┌─────────────────────────────────────────────────────────────────┐
│                    UTILISER JQUERY ?                            │
│                                                                 │
│  ✅ OUI — contextes valides                                     │
│  ───────────────────────────────────────────────────────────── │
│  • Projet legacy sur jQuery 1.x/2.x → maintenance              │
│  • Plugins jQuery utilisés (DataTables, Select2, Slick)         │
│    qui ne proposent pas d'alternative vanilla moderne           │
│  • Bootstrap 4 (dépend de jQuery)                              │
│  • CMS WordPress/Drupal : jQuery inclus nativement,             │
│    scripts admin/thème s'appuient dessus                        │
│  • Contrainte de compatibilité avec de très vieux navigateurs  │
│    (mais jQuery 3.x abandonne IE < 9, donc rarissime)           │
│  • Codebase d'équipe existante bien maîtrisée,                  │
│    pas de refacto planifiée                                     │
│                                                                 │
│  ❌ NON — préférer autre chose                                  │
│  ───────────────────────────────────────────────────────────── │
│  • Nouveau projet greenfield                                    │
│  • Application SPA (React/Vue/Angular gèrent le DOM)           │
│  • Performance critique (mobile, connexions lentes)             │
│  • Équipe formée en JavaScript moderne                          │
│  • Taille du bundle est une contrainte (87 KB pour rien)        │
└─────────────────────────────────────────────────────────────────┘
```

### 15.2 Statistiques d'utilisation (2024)

| Contexte | % sites utilisant jQuery |
|---|---|
| Top 1 million de sites web | ~78 % |
| Sites WordPress | ~96 % |
| Sites e-commerce | ~67 % |
| Applications SPA | ~15 % |
| Nouveaux projets 2023-2024 | ~20 % |

> [!info] Pourquoi encore 78 % ?
> La majorité du web **n'est pas** des applications SPA complexes. Ce sont des sites marketing, des blogs, des boutiques en ligne, des portails institutionnels — des projets où jQuery a été intégré il y a 5-10 ans et où un refactoring complet ne serait pas rentable.

### 15.3 Stratégie de migration progressive

Quand vous devez intervenir sur du code jQuery legacy :

```javascript
// Stratégie 1 : "Strangler Fig" — remplacer par modules
// Ne pas réécrire tout jQuery en un bloc.
// À chaque nouvelle feature ou bug fix : écrire en vanilla, cohabiter temporairement.

// Ancienne fonction jQuery (ne pas toucher)
function ancienneInitCarousel() {
    $(".carousel").slick({ /* ... */ });
}

// Nouvelle fonction vanilla (nouveau code)
function initialiserNavigationModerne() {
    document.querySelectorAll(".nav-item").forEach(item => {
        item.addEventListener("click", handleNavClick);
    });
}

// Les deux cohabitent — migration progressive sur plusieurs sprints

// Stratégie 2 : Isoler jQuery dans des modules
// Créer un wrapper qui expose une API framework-agnostique
const domUtils = {
    masquer: (selector) => $(selector).hide(),
    afficher: (selector) => $(selector).show(),
};
// Plus tard : remplacer l'implémentation sans toucher les appelants
const domUtils = {
    masquer: (selector) => document.querySelectorAll(selector)
        .forEach(el => el.style.display = "none"),
    afficher: (selector) => document.querySelectorAll(selector)
        .forEach(el => el.style.display = ""),
};
```

---

## 16. Projet Pratique — Application Todo List complète

### Cahier des charges

Créer une application Todo List complète avec :
- Ajout de tâches avec validation
- Marquage comme terminé
- Suppression (avec confirmation)
- Filtre (toutes / actives / terminées)
- Persistance en `localStorage`
- Animation à l'ajout et à la suppression

### Structure HTML

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Todo List jQuery</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
               background: #f0f2f5; min-height: 100vh; padding: 40px 20px; }
        .app { max-width: 500px; margin: 0 auto; background: white;
               border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,.1);
               overflow: hidden; }
        h1 { background: #667eea; color: white; padding: 24px;
             font-size: 1.5rem; text-align: center; }
        .saisie-zone { padding: 16px; border-bottom: 1px solid #eee; display: flex; gap: 8px; }
        #champ-tache { flex: 1; padding: 10px 14px; border: 2px solid #e2e8f0;
                       border-radius: 8px; font-size: 1rem; outline: none;
                       transition: border-color 200ms; }
        #champ-tache:focus { border-color: #667eea; }
        #btn-ajouter { padding: 10px 20px; background: #667eea; color: white;
                       border: none; border-radius: 8px; cursor: pointer;
                       font-size: 1rem; transition: background 200ms; }
        #btn-ajouter:hover { background: #5a67d8; }
        .filtres { display: flex; padding: 8px 16px; gap: 8px; border-bottom: 1px solid #eee; }
        .filtre { padding: 4px 12px; border: 1px solid #e2e8f0; border-radius: 20px;
                  cursor: pointer; font-size: .875rem; color: #718096; transition: all 200ms; }
        .filtre.actif { background: #667eea; color: white; border-color: #667eea; }
        #liste-taches { list-style: none; min-height: 100px; }
        .tache { display: flex; align-items: center; gap: 12px; padding: 14px 16px;
                 border-bottom: 1px solid #f7fafc; transition: background 150ms; }
        .tache:hover { background: #f7fafc; }
        .tache.terminee .tache-texte { text-decoration: line-through; color: #a0aec0; }
        .tache input[type="checkbox"] { width: 18px; height: 18px; accent-color: #667eea; cursor: pointer; }
        .tache-texte { flex: 1; }
        .btn-supprimer { background: none; border: none; color: #e53e3e; cursor: pointer;
                         font-size: 1.2rem; opacity: 0; transition: opacity 150ms; }
        .tache:hover .btn-supprimer { opacity: 1; }
        .pied { padding: 12px 16px; color: #718096; font-size: .875rem;
                display: flex; justify-content: space-between; }
        #message-vide { padding: 40px; text-align: center; color: #a0aec0;
                         font-style: italic; display: none; }
    </style>
</head>
<body>
    <div class="app">
        <h1>Ma Todo List</h1>

        <div class="saisie-zone">
            <input type="text" id="champ-tache" placeholder="Ajouter une tâche…" maxlength="200">
            <button id="btn-ajouter">Ajouter</button>
        </div>

        <div class="filtres">
            <button class="filtre actif" data-filtre="toutes">Toutes</button>
            <button class="filtre" data-filtre="actives">Actives</button>
            <button class="filtre" data-filtre="terminees">Terminées</button>
        </div>

        <ul id="liste-taches"></ul>
        <p id="message-vide">Aucune tâche à afficher.</p>

        <div class="pied">
            <span id="compteur">0 tâche(s) restante(s)</span>
            <button id="btn-vider-terminees" style="background:none;border:none;color:#e53e3e;cursor:pointer;display:none;">
                Supprimer terminées
            </button>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/jquery@3.7.1/dist/jquery.min.js"></script>
    <script src="todo.js"></script>
</body>
</html>
```

### Fichier `todo.js`

```javascript
$(function() {
    "use strict";

    // ─── État de l'application ───
    const CLE_STORAGE = "holberton-todos";
    let filtreActuel = "toutes";
    let taches = chargerDepuisStorage();

    // ─── Rendu ───
    function rendreTaches() {
        const $liste = $("#liste-taches");
        $liste.empty();

        const tachesFiltrees = filtrerTaches();

        if (tachesFiltrees.length === 0) {
            $("#message-vide").show();
        } else {
            $("#message-vide").hide();
            tachesFiltrees.forEach(function(tache) {
                $liste.append(creerElementTache(tache));
            });
        }

        mettreAJourCompteur();
        sauvegarderDansStorage();
    }

    function creerElementTache(tache) {
        const $li = $("<li>")
            .addClass("tache")
            .toggleClass("terminee", tache.terminee)
            .attr("data-id", tache.id);

        const $checkbox = $("<input>")
            .attr("type", "checkbox")
            .prop("checked", tache.terminee);

        const $texte = $("<span>")
            .addClass("tache-texte")
            .text(tache.texte);

        const $btnSupprimer = $("<button>")
            .addClass("btn-supprimer")
            .attr("title", "Supprimer")
            .html("&times;");

        $li.append($checkbox, $texte, $btnSupprimer);
        return $li;
    }

    function filtrerTaches() {
        switch (filtreActuel) {
            case "actives":
                return taches.filter(function(t) { return !t.terminee; });
            case "terminees":
                return taches.filter(function(t) { return t.terminee; });
            default:
                return taches;
        }
    }

    function mettreAJourCompteur() {
        const nbActives = taches.filter(function(t) { return !t.terminee; }).length;
        const nbTerminees = taches.filter(function(t) { return t.terminee; }).length;
        const mot = nbActives <= 1 ? "tâche" : "tâches";
        $("#compteur").text(`${nbActives} ${mot} restante(s)`);
        $("#btn-vider-terminees").toggle(nbTerminees > 0);
    }

    // ─── Persistance ───
    function sauvegarderDansStorage() {
        localStorage.setItem(CLE_STORAGE, JSON.stringify(taches));
    }

    function chargerDepuisStorage() {
        try {
            const donnees = localStorage.getItem(CLE_STORAGE);
            return donnees ? JSON.parse(donnees) : [];
        } catch(e) {
            return [];
        }
    }

    // ─── Actions ───
    function ajouterTache(texte) {
        const textePropre = texte.trim();
        if (textePropre.length === 0) {
            $("#champ-tache").addClass("erreur").focus();
            setTimeout(function() { $("#champ-tache").removeClass("erreur"); }, 500);
            return;
        }

        const nouvelleTache = {
            id: Date.now(),
            texte: textePropre,
            terminee: false,
            creeLe: new Date().toISOString()
        };

        taches.unshift(nouvelleTache); // Ajouter en tête de liste
        rendreTaches();

        // Animation d'entrée du premier élément
        $("#liste-taches li:first")
            .hide()
            .fadeIn(300);

        $("#champ-tache").val("").focus();
    }

    function basculerTache(id) {
        const tache = taches.find(function(t) { return t.id === id; });
        if (tache) {
            tache.terminee = !tache.terminee;
            rendreTaches();
        }
    }

    function supprimerTache(id) {
        const index = taches.findIndex(function(t) { return t.id === id; });
        if (index === -1) return;

        // Animation de sortie avant suppression
        const $li = $(`li[data-id='${id}']`);
        $li.fadeOut(250, function() {
            taches.splice(index, 1);
            rendreTaches();
        });
    }

    // ─── Événements ───

    // Ajouter via bouton
    $("#btn-ajouter").on("click", function() {
        ajouterTache($("#champ-tache").val());
    });

    // Ajouter via Entrée
    $("#champ-tache").on("keypress", function(e) {
        if (e.key === "Enter") {
            ajouterTache($(this).val());
        }
    });

    // Délégation : cocher/décocher
    $("#liste-taches").on("change", "input[type='checkbox']", function() {
        const id = parseInt($(this).closest(".tache").attr("data-id"), 10);
        basculerTache(id);
    });

    // Délégation : supprimer
    $("#liste-taches").on("click", ".btn-supprimer", function() {
        const id = parseInt($(this).closest(".tache").attr("data-id"), 10);
        supprimerTache(id);
    });

    // Filtres
    $(".filtres").on("click", ".filtre", function() {
        filtreActuel = $(this).data("filtre");
        $(".filtre").removeClass("actif");
        $(this).addClass("actif");
        rendreTaches();
    });

    // Vider les terminées
    $("#btn-vider-terminees").on("click", function() {
        taches = taches.filter(function(t) { return !t.terminee; });
        rendreTaches();
    });

    // ─── Initialisation ───
    rendreTaches();
});
```

---

## 17. Exercices et Challenges

### Exercice 1 — Sélecteurs (Niveau : Débutant)

Étant donné ce HTML :

```html
<nav id="menu-principal">
    <ul>
        <li class="item actif"><a href="/">Accueil</a></li>
        <li class="item"><a href="/blog">Blog</a></li>
        <li class="item disabled"><a href="/admin">Admin</a></li>
        <li class="item"><a href="/contact" data-section="contact">Contact</a></li>
    </ul>
</nav>
```

Écrire les sélecteurs jQuery pour :
1. Sélectionner tous les `<li>` du menu
2. Sélectionner uniquement le `<li>` actif
3. Sélectionner les `<li>` qui ne sont ni actifs ni désactivés
4. Sélectionner le lien avec `data-section="contact"`
5. Sélectionner le texte de tous les liens et les afficher dans la console

### Exercice 2 — Manipulation DOM (Niveau : Débutant)

Créer un bouton "Mode sombre" qui :
- Ajoute/retire la classe `.dark-mode` sur `<body>` au clic
- Change le texte du bouton entre "Mode sombre" et "Mode clair"
- Persiste le choix en `localStorage`
- Applique le choix au chargement de la page

### Exercice 3 — AJAX (Niveau : Intermédiaire)

Utiliser l'API publique `https://jsonplaceholder.typicode.com` :
1. Au chargement, afficher une liste de 10 utilisateurs (GET `/users`)
2. Au clic sur un utilisateur, afficher ses posts (GET `/posts?userId={id}`)
3. Gérer le chargement avec un spinner
4. Gérer les erreurs avec un message affiché à l'utilisateur

### Exercice 4 — Event Delegation (Niveau : Intermédiaire)

Créer un mini-tableau de notes étudiant :
- Chaque ligne a un bouton "Modifier" (ouvre un `<input>`) et "Supprimer"
- Bouton "Ajouter étudiant" ajoute dynamiquement une ligne
- Tous les boutons doivent fonctionner sur les lignes dynamiques (délégation)
- Le calcul de la moyenne se met à jour à chaque modification

### Exercice 5 — Plugin jQuery (Niveau : Avancé)

Créer un plugin jQuery `$.fn.tooltipSimple` qui :
- Affiche un tooltip au `mouseenter` sur l'attribut `data-tooltip`
- Positionne le tooltip au-dessus de l'élément
- L'anime avec un fadeIn/fadeOut
- Supporte les options `{ delai: 300, classe: "custom" }`
- Fonctionne sur une collection de N éléments

### Challenge — Migration (Niveau : Avancé)

Prendre le projet Todo List de la section 16 et le réécrire **entièrement en JavaScript vanilla ES6+** :
- Sans jQuery, sans aucune bibliothèque
- Mêmes fonctionnalités et animations
- Utiliser `fetch()`, `classList`, `insertAdjacentHTML`, `closest()`
- Comparer la taille du code résultant

---

## Récapitulatif

```
┌────────────────────────────────────────────────────────────────────┐
│                     JQUERY EN UN COUP D'ŒIL                       │
│                                                                    │
│  Sélection      $("sel"), .find(), .filter(), .closest()           │
│  Contenu        .html(), .text(), .val(), .attr(), .data()         │
│  Classes        .addClass(), .removeClass(), .toggleClass()        │
│  Styles         .css(), .width(), .height(), .offset()             │
│  DOM            .append(), .prepend(), .before(), .after()         │
│  Traversal      .parent(), .children(), .siblings(), .next()       │
│  Événements     .on(), .off(), .trigger(), .one()                  │
│  Délégation     .on("event", "selector", handler)                  │
│  Effets         .show/hide/toggle, .fade*, .slide*, .animate()     │
│  AJAX           $.ajax(), $.get(), $.post(), .done(), .fail()      │
│  Chaining       Chaîner les méthodes — retourne toujours $         │
│  Ready          $(fn) — exécuté après DOMContentLoaded             │
│                                                                    │
│  Quand l'utiliser ?                                                │
│  • Projets legacy, WordPress, plugins dépendants                   │
│  • Pas pour les nouveaux projets — utiliser vanilla ES6+           │
└────────────────────────────────────────────────────────────────────┘
```

> [!tip] Ressources pour aller plus loin
> - Documentation officielle jQuery : https://api.jquery.com
> - "You Might Not Need jQuery" : http://youmightnotneedjquery.com
> - jQuery Learning Center : https://learn.jquery.com
> - jQuery Source Viewer : https://j11y.io/jquery (lire le code source est formateur)
> - Bootstrap 4 source : exemple de jQuery utilisé à grande échelle dans la vraie vie
