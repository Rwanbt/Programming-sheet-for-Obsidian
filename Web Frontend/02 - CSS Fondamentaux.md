# CSS Fondamentaux

Le **CSS** (Cascading Style Sheets) est le langage qui controle l'apparence visuelle de vos pages web. Si le HTML est le squelette, le CSS est la peau, les vetements et le maquillage. Il definit les couleurs, les polices, les espacements, la disposition des elements et bien plus encore.

CSS est un langage declaratif : vous decrivez **a quoi** les choses doivent ressembler, et le navigateur s'occupe du rendu. Ce cours couvre l'ensemble des fondamentaux CSS, de la syntaxe de base jusqu'au responsive design avec Flexbox et Grid.

> [!tip] Analogie
> Imaginez une maison. Le HTML est la structure : les murs, les pieces, les portes et les fenetres. Le CSS est la **decoration interieure** : la peinture des murs, le parquet ou le carrelage, les rideaux, la disposition des meubles, l'eclairage. Une meme maison (meme HTML) peut avoir des looks completement differents selon la decoration (differents CSS).

---

## Les 3 facons d'ajouter du CSS

### 1. CSS en ligne (inline)

Directement dans l'attribut `style` d'un element :

```html
<p style="color: red; font-size: 18px;">Texte rouge en 18px</p>
```

### 2. CSS interne (internal)

Dans une balise `<style>` a l'interieur du `<head>` :

```html
<head>
    <style>
        p {
            color: blue;
            font-size: 16px;
        }
    </style>
</head>
```

### 3. CSS externe (external)

Dans un fichier `.css` separe, lie via `<link>` :

```html
<head>
    <link rel="stylesheet" href="styles.css">
</head>
```

```css
/* styles.css */
p {
    color: green;
    font-size: 16px;
}
```

### Comparaison des 3 methodes

| Methode | Avantages | Inconvenients |
|---------|-----------|---------------|
| Inline | Rapide pour tester | Pas reutilisable, melange HTML/CSS, specifite tres haute |
| Interne | Tout dans un fichier | Pas reutilisable entre pages, alourdit le HTML |
| **Externe** | **Reutilisable, cache par le navigateur, separation propre** | Requete HTTP supplementaire |

> [!warning] Bonne pratique
> Utilisez **toujours** un fichier CSS externe en production. Le CSS inline et interne ne devrait servir que pour le prototypage rapide ou des cas tres specifiques (emails HTML, styles critiques above-the-fold).

---

## Syntaxe CSS

```css
/* Structure d'une regle CSS */
selecteur {
    propriete: valeur;
    autre-propriete: autre-valeur;
}

/* Exemple concret */
h1 {
    color: #333333;
    font-size: 2rem;
    margin-bottom: 1rem;
}
```

```
Anatomie d'une regle CSS :

    selecteur ──► h1 {
    propriete ──►     color: #333333;  ◄── valeur
                      font-size: 2rem;
    declaration ──►   margin-bottom: 1rem;
                  }
    
    Chaque paire propriete: valeur s'appelle une DECLARATION.
    L'ensemble selecteur + declarations forme une REGLE.
```

---

## Les selecteurs

Les selecteurs determinent **quels** elements HTML sont affectes par les regles CSS.

### Selecteurs de base

```css
/* Selecteur d'element (type) */
p { color: black; }
h1 { font-size: 2rem; }

/* Selecteur de classe (.) */
.important { font-weight: bold; }
.btn-primary { background: blue; }

/* Selecteur d'identifiant (#) */
#header { height: 80px; }
#main-content { max-width: 1200px; }

/* Selecteur universel (*) */
* { margin: 0; padding: 0; box-sizing: border-box; }
```

```html
<!-- En HTML -->
<p class="important">Texte important</p>
<p>Texte normal</p>
<div id="header">En-tete</div>
```

> [!info] Classes vs IDs
> - Une **classe** (`.`) peut etre utilisee sur plusieurs elements
> - Un **ID** (`#`) doit etre unique dans la page
> - Preferez les classes pour le CSS. Reservez les IDs pour les ancres et le JavaScript.

### Selecteurs de groupement

```css
/* Groupement : meme style pour plusieurs selecteurs */
h1, h2, h3 {
    font-family: 'Georgia', serif;
    color: #222;
}
```

### Selecteurs de combinaison

```css
/* Descendant (espace) : tout <a> a l'interieur d'un <nav> */
nav a {
    text-decoration: none;
}

/* Enfant direct (>) : seulement les <li> enfants directs de <ul> */
ul > li {
    list-style: square;
}

/* Adjacent (+) : le premier <p> juste apres un <h2> */
h2 + p {
    font-size: 1.2rem;
    font-weight: 500;
}

/* Frere general (~) : tous les <p> apres un <h2> au meme niveau */
h2 ~ p {
    color: #555;
}
```

```
Illustration des combinateurs :

<div>                        div p     → selectionne TOUS les <p> descendants
  <p>A</p>                  div > p   → selectionne A, B (enfants directs)
  <section>                  h2 + p    → selectionne C (adjacent a h2)
    <h2>Titre</h2>          h2 ~ p    → selectionne C et D (freres de h2)
    <p>C</p>
    <p>D</p>
  </section>
  <p>B</p>
</div>
```

### Selecteurs d'attribut

```css
/* Element ayant l'attribut */
[title] { cursor: help; }

/* Attribut avec valeur exacte */
[type="email"] { border-color: blue; }

/* Attribut commencant par */
[href^="https"] { color: green; }

/* Attribut finissant par */
[href$=".pdf"] { color: red; }

/* Attribut contenant */
[class*="btn"] { cursor: pointer; }
```

### Pseudo-classes

Les pseudo-classes selectionnent des elements dans un **etat** particulier :

```css
/* Survol de la souris */
a:hover {
    color: red;
    text-decoration: underline;
}

/* Element qui a le focus (clavier, clic) */
input:focus {
    outline: 2px solid blue;
    border-color: blue;
}

/* Premier enfant */
li:first-child {
    font-weight: bold;
}

/* Dernier enfant */
li:last-child {
    border-bottom: none;
}

/* Nieme enfant */
tr:nth-child(even) {
    background-color: #f5f5f5;
}

tr:nth-child(odd) {
    background-color: white;
}

/* Nieme enfant avec formule */
li:nth-child(3n) {
    color: red;  /* Chaque 3e element */
}

/* Negation */
p:not(.special) {
    color: gray;
}

/* Liens visites */
a:visited { color: purple; }

/* Liens actifs (clic en cours) */
a:active { color: orange; }

/* Champs de formulaire */
input:required { border-left: 3px solid red; }
input:valid { border-color: green; }
input:invalid { border-color: red; }
input:disabled { opacity: 0.5; }
input:checked + label { font-weight: bold; }
```

### Pseudo-elements

Les pseudo-elements permettent de styler une **partie** d'un element ou d'inserer du contenu :

```css
/* Premier ligne d'un paragraphe */
p::first-line {
    font-weight: bold;
    font-size: 1.1em;
}

/* Premiere lettre */
p::first-letter {
    font-size: 3em;
    float: left;
    line-height: 1;
    margin-right: 0.1em;
}

/* Contenu genere avant l'element */
.required::before {
    content: "* ";
    color: red;
}

/* Contenu genere apres l'element */
a[href^="http"]::after {
    content: " ↗";
    font-size: 0.8em;
}

/* Selection de texte par l'utilisateur */
::selection {
    background: #ffeb3b;
    color: black;
}

/* Placeholder des inputs */
::placeholder {
    color: #999;
    font-style: italic;
}
```

> [!info] `::before` et `::after`
> Ces pseudo-elements creent des "faux elements" avant ou apres le contenu d'un element. Ils necessitent la propriete `content` (meme vide : `content: ""`). Ils sont tres utiles pour les decorations sans polluer le HTML.

---

## La specificite

La specificite determine quelle regle CSS "gagne" quand plusieurs regles ciblent le meme element.

### Calcul de la specificite

```
Specificite = (Inline, IDs, Classes, Elements)

Inline style     →  1, 0, 0, 0  (1000 points)
#id              →  0, 1, 0, 0  (100 points)
.class           →  0, 0, 1, 0  (10 points)
element          →  0, 0, 0, 1  (1 point)
```

> [!example] Exemples de calcul
> ```css
> p                    →  0,0,0,1  =   1
> p.intro              →  0,0,1,1  =  11
> #content p.intro     →  0,1,1,1  = 111
> div#content p.intro  →  0,1,1,2  = 112
> style="..."          →  1,0,0,0  = 1000
> ```

```css
/* Specificite : 0,0,0,1 (1 point) */
p { color: black; }

/* Specificite : 0,0,1,0 (10 points) — GAGNE sur le precedent */
.texte-rouge { color: red; }

/* Specificite : 0,1,0,0 (100 points) — GAGNE sur les deux precedents */
#introduction { color: blue; }
```

> [!warning] `!important` : a eviter
> `!important` ecrase toute specificite. C'est l'arme nucleaire du CSS :
> ```css
> p { color: red !important; } /* Gagne contre tout, sauf un autre !important */
> ```
> Evitez-le autant que possible. Son usage excessif rend le CSS inmaintenable. Les seuls cas acceptables sont les overrides de librairies tierces.

### La cascade : ordre de priorite

Quand la specificite est egale, c'est l'ordre qui determine le gagnant :

1. Feuilles de style du navigateur (user-agent)
2. Feuilles de style de l'utilisateur
3. Feuilles de style de l'auteur (votre CSS)
4. Declarations `!important` de l'auteur
5. Declarations `!important` de l'utilisateur

A specificite egale et meme origine, **la derniere regle declaree gagne**.

### L'heritage

Certaines proprietes se transmettent automatiquement aux elements enfants :

```css
/* Proprietes heritees (principales) :
   color, font-family, font-size, line-height, text-align,
   visibility, cursor, list-style */

body {
    color: #333;           /* Tous les textes seront #333 */
    font-family: Arial;    /* Tous les textes seront en Arial */
    line-height: 1.6;      /* Interligne pour tout le document */
}

/* Proprietes NON heritees (principales) :
   margin, padding, border, background, width, height,
   display, position, overflow */
```

```css
/* Forcer ou bloquer l'heritage */
.enfant {
    color: inherit;   /* Force l'heritage */
    border: initial;  /* Remet la valeur par defaut */
    margin: unset;    /* Herite si la propriete est naturellement heritee,
                         sinon initial */
}
```

---

## Le modele de boite (Box Model)

**Chaque element HTML est une boite rectangulaire.** Le box model definit les dimensions de cette boite :

```
┌─────────────────────────────────────────────────────┐
│                      MARGIN                         │
│   ┌─────────────────────────────────────────────┐   │
│   │                  BORDER                     │   │
│   │   ┌─────────────────────────────────────┐   │   │
│   │   │              PADDING                │   │   │
│   │   │   ┌─────────────────────────────┐   │   │   │
│   │   │   │                             │   │   │   │
│   │   │   │          CONTENT            │   │   │   │
│   │   │   │     (largeur x hauteur)     │   │   │   │
│   │   │   │                             │   │   │   │
│   │   │   └─────────────────────────────┘   │   │   │
│   │   │                                     │   │   │
│   │   └─────────────────────────────────────┘   │   │
│   │                                             │   │
│   └─────────────────────────────────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

```css
.box {
    /* Contenu */
    width: 300px;
    height: 200px;

    /* Espacement interne */
    padding: 20px;

    /* Bordure */
    border: 2px solid black;

    /* Espacement externe */
    margin: 10px;
}

/* Largeur TOTALE par defaut (content-box) :
   300 + 20*2 + 2*2 + 10*2 = 364px  (contenu + padding + border + margin) */
```

> [!warning] `box-sizing: border-box` — indispensable
> Par defaut, `width` ne definit que la largeur du **contenu**. Le padding et le border s'ajoutent en plus, ce qui rend les calculs penibles.
>
> Avec `box-sizing: border-box`, `width` inclut le padding et le border :
> ```css
> /* Reset universel recommande */
> *, *::before, *::after {
>     box-sizing: border-box;
> }
>
> .box {
>     width: 300px;
>     padding: 20px;
>     border: 2px solid black;
>     /* Largeur TOTALE = 300px (le padding et border sont inclus) */
>     /* Le contenu fait 300 - 20*2 - 2*2 = 256px */
> }
> ```

### Marges : comportements speciaux

```css
/* Centrer un element bloc horizontalement */
.container {
    width: 800px;
    margin: 0 auto;  /* marges laterales automatiques = centrage */
}

/* Raccourci margin/padding */
margin: 10px;                /* 4 cotes identiques */
margin: 10px 20px;           /* vertical | horizontal */
margin: 10px 20px 30px;      /* haut | horizontal | bas */
margin: 10px 20px 30px 40px; /* haut | droite | bas | gauche (sens horaire) */
```

> [!info] Fusion des marges (margin collapsing)
> Les marges verticales de deux elements adjacents **fusionnent** : seule la plus grande est appliquee.
> ```css
> .paragraphe1 { margin-bottom: 20px; }
> .paragraphe2 { margin-top: 30px; }
> /* L'espace entre eux sera 30px, pas 50px */
> ```
> Ce comportement ne s'applique qu'aux marges **verticales** et uniquement dans le flux normal.

---

## Display

La propriete `display` controle comment un element se comporte dans le flux :

```css
/* Block : prend toute la largeur, commence sur nouvelle ligne */
/* Elements block par defaut : div, p, h1-h6, section, article, form... */
.block { display: block; }

/* Inline : s'insere dans le flux du texte, pas de width/height */
/* Elements inline par defaut : span, a, strong, em, img... */
.inline { display: inline; }

/* Inline-block : comme inline, mais accepte width/height */
.inline-block { display: inline-block; }

/* None : l'element disparait completement (pas d'espace reserve) */
.cache { display: none; }
```

```
Block vs Inline vs Inline-block :

BLOCK :
┌──────────────────────────────────────┐
│ Prend toute la largeur               │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│ Chaque bloc sur une nouvelle ligne   │
└──────────────────────────────────────┘

INLINE :
Ce texte contient [un lien] et [un autre] sur la meme ligne.
(pas de width/height possible)

INLINE-BLOCK :
┌──────┐ ┌──────┐ ┌──────┐
│ Box1 │ │ Box2 │ │ Box3 │  ← sur la meme ligne
└──────┘ └──────┘ └──────┘     mais avec width/height
```

---

## Positionnement

```css
/* STATIC (defaut) : flux normal, pas de top/left/right/bottom */
.static { position: static; }

/* RELATIVE : decale par rapport a sa position normale
   L'espace original est conserve */
.relative {
    position: relative;
    top: 10px;
    left: 20px;
}

/* ABSOLUTE : sort du flux, se positionne par rapport
   au premier ancetre positionne (relative/absolute/fixed) */
.absolute {
    position: absolute;
    top: 0;
    right: 0;
}

/* FIXED : sort du flux, fixe par rapport a la fenetre
   (reste en place lors du scroll) */
.fixed {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
}

/* STICKY : se comporte comme relative, puis comme fixed
   quand on scrolle au-dela d'un seuil */
.sticky {
    position: sticky;
    top: 0;  /* Se fixe quand il atteint le haut de la fenetre */
}
```

> [!example] Cas d'usage courants
> - **`relative`** : decalage visuel leger, sert souvent de conteneur de reference pour un `absolute`
> - **`absolute`** : badges, tooltips, menus deroulants, superposition d'elements
> - **`fixed`** : barre de navigation fixe, bouton "retour en haut"
> - **`sticky`** : en-tete de tableau qui reste visible, sidebar qui suit le scroll

```
Position absolute avec ancetre relative :

┌─────── .parent (position: relative) ───────┐
│                                             │
│                            ┌─── .badge ───┐ │
│                            │ position:    │ │
│                            │ absolute;    │ │
│                            │ top: 0;      │ │
│                            │ right: 0;    │ │
│                            └──────────────┘ │
│                                             │
│   Le badge se positionne dans le coin       │
│   superieur droit du parent.                │
│                                             │
└─────────────────────────────────────────────┘
```

### z-index

Quand des elements se chevauchent, `z-index` controle l'ordre d'empilement :

```css
.arriere-plan { position: relative; z-index: 1; }
.contenu      { position: relative; z-index: 2; }
.popup        { position: absolute; z-index: 100; }
```

> [!warning] z-index ne fonctionne qu'avec position
> `z-index` n'a d'effet que sur les elements avec `position` autre que `static` (ou dans un contexte flex/grid).

---

## Couleurs

```css
/* Mot-cle nomme */
color: red;
color: tomato;
color: cornflowerblue;

/* Hexadecimal */
color: #ff0000;          /* Rouge */
color: #f00;             /* Rouge (raccourci) */
color: #ff000080;        /* Rouge semi-transparent */

/* RGB */
color: rgb(255, 0, 0);           /* Rouge */
color: rgba(255, 0, 0, 0.5);     /* Rouge 50% transparent */

/* HSL (Hue, Saturation, Lightness) */
color: hsl(0, 100%, 50%);        /* Rouge */
color: hsla(0, 100%, 50%, 0.5);  /* Rouge 50% transparent */

/* Variables CSS (custom properties) */
:root {
    --couleur-primaire: #3498db;
    --couleur-secondaire: #2ecc71;
    --couleur-texte: #333;
}
color: var(--couleur-primaire);
```

> [!tip] HSL : le format le plus intuitif
> - **H (Hue)** : la teinte sur la roue chromatique (0=rouge, 120=vert, 240=bleu)
> - **S (Saturation)** : 0% = gris, 100% = couleur vive
> - **L (Lightness)** : 0% = noir, 50% = couleur pure, 100% = blanc
>
> C'est le format le plus facile pour creer des variations d'une meme couleur :
> ```css
> --bleu-clair: hsl(210, 80%, 70%);
> --bleu:       hsl(210, 80%, 50%);
> --bleu-fonce: hsl(210, 80%, 30%);
> ```

---

## Typographie

```css
body {
    /* Famille de polices (avec fallbacks) */
    font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;

    /* Taille de police */
    font-size: 16px;    /* Valeur absolue */
    font-size: 1rem;    /* Relative a la taille racine (html) */
    font-size: 1.2em;   /* Relative a la taille du parent */

    /* Graisse */
    font-weight: 400;   /* Normal */
    font-weight: 700;   /* Gras (= bold) */

    /* Interligne */
    line-height: 1.6;   /* 1.6x la taille de police — sans unite */

    /* Alignement */
    text-align: left;      /* Gauche (defaut LTR) */
    text-align: center;    /* Centre */
    text-align: right;     /* Droite */
    text-align: justify;   /* Justifie */
}

a {
    /* Decoration de texte */
    text-decoration: none;        /* Supprime le soulignement */
    text-decoration: underline;   /* Souligne */
    text-decoration: line-through; /* Barre */
}

h1 {
    /* Transformation */
    text-transform: uppercase;    /* MAJUSCULES */
    text-transform: lowercase;    /* minuscules */
    text-transform: capitalize;   /* Premiere Lettre En Majuscule */

    /* Espacement des lettres */
    letter-spacing: 2px;

    /* Espacement des mots */
    word-spacing: 4px;
}
```

> [!info] rem vs em vs px
> - **`px`** : valeur absolue, ne change pas. Simple mais pas flexible.
> - **`em`** : relative au parent. 1.5em = 1.5x la taille du parent. Peut se composer et devenir imprevisible.
> - **`rem`** : relative au `<html>`. 1rem = taille definie sur `html`. Predictible et recommande.
>
> Bonne pratique : utilisez `rem` pour la plupart des tailles, `em` pour les espacements relatifs a la taille du texte courant.

### Google Fonts

```html
<!-- Dans le <head> du HTML -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap"
      rel="stylesheet">
```

```css
/* Dans le CSS */
body {
    font-family: 'Inter', sans-serif;
}
```

---

## Arriere-plans

```css
.element {
    /* Couleur de fond */
    background-color: #f5f5f5;

    /* Image de fond */
    background-image: url('images/fond.jpg');
    background-repeat: no-repeat;    /* no-repeat | repeat-x | repeat-y */
    background-size: cover;          /* cover | contain | 100px 200px */
    background-position: center;     /* center | top left | 50% 50% */
    background-attachment: fixed;    /* fixed | scroll */

    /* Raccourci */
    background: #f5f5f5 url('fond.jpg') no-repeat center/cover;

    /* Degrades */
    background: linear-gradient(to right, #3498db, #2ecc71);
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    background: radial-gradient(circle, #fff, #000);

    /* Degrades multiples */
    background:
        linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.5)),
        url('image.jpg') center/cover;
}
```

---

## Bordures et ombres

```css
.element {
    /* Bordures */
    border: 2px solid #333;
    border-top: 1px dashed red;
    border-radius: 8px;             /* Coins arrondis */
    border-radius: 50%;             /* Cercle parfait (si carre) */
    border-radius: 20px 0 20px 0;   /* Coins individuels */

    /* Ombre de boite */
    box-shadow: 2px 4px 8px rgba(0, 0, 0, 0.2);
    /*          offset-x  offset-y  blur  couleur */

    /* Ombre interne */
    box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);

    /* Ombres multiples */
    box-shadow:
        0 1px 3px rgba(0,0,0,0.12),
        0 1px 2px rgba(0,0,0,0.24);

    /* Ombre de texte */
    text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}
```

---

## Flexbox

Flexbox est un systeme de mise en page **unidimensionnel** (une ligne OU une colonne). C'est l'outil ideal pour aligner et distribuer l'espace entre les elements.

### Activer Flexbox

```css
.container {
    display: flex;
}
```

```
Avant flex :                 Apres display: flex :
┌──────────────────┐         ┌──────┐┌──────┐┌──────┐
│     Item 1       │         │Item 1││Item 2││Item 3│
├──────────────────┤   →     └──────┘└──────┘└──────┘
│     Item 2       │
├──────────────────┤
│     Item 3       │
└──────────────────┘
```

### Proprietes du conteneur flex

```css
.container {
    display: flex;

    /* Direction de l'axe principal */
    flex-direction: row;            /* Horizontal (defaut) → */
    flex-direction: row-reverse;    /* Horizontal inverse ← */
    flex-direction: column;         /* Vertical ↓ */
    flex-direction: column-reverse; /* Vertical inverse ↑ */

    /* Retour a la ligne */
    flex-wrap: nowrap;   /* Tout sur une ligne (defaut) */
    flex-wrap: wrap;     /* Retour a la ligne si necessaire */

    /* Alignement sur l'axe principal */
    justify-content: flex-start;    /* Debut (defaut) */
    justify-content: flex-end;      /* Fin */
    justify-content: center;        /* Centre */
    justify-content: space-between; /* Espace entre les items */
    justify-content: space-around;  /* Espace autour des items */
    justify-content: space-evenly;  /* Espace egal partout */

    /* Alignement sur l'axe secondaire */
    align-items: stretch;     /* Etire (defaut) */
    align-items: flex-start;  /* Haut */
    align-items: flex-end;    /* Bas */
    align-items: center;      /* Centre vertical */
    align-items: baseline;    /* Ligne de base du texte */

    /* Espacement entre items */
    gap: 20px;           /* Espace entre tous les items */
    row-gap: 20px;       /* Espace vertical uniquement */
    column-gap: 10px;    /* Espace horizontal uniquement */
}
```

```
justify-content (axe principal, ici horizontal) :

flex-start:      [A][B][C]                    
flex-end:                          [A][B][C]
center:                [A][B][C]
space-between:   [A]        [B]        [C]
space-around:     [A]     [B]     [C]
space-evenly:      [A]      [B]      [C]

align-items (axe secondaire, ici vertical) :

flex-start:  ┌─[A]─[B]─[C]─────────┐
             │                      │
             └──────────────────────┘

center:      ┌──────────────────────┐
             │  [A]─[B]─[C]        │
             └──────────────────────┘

flex-end:    ┌──────────────────────┐
             │                      │
             └─[A]─[B]─[C]─────────┘
```

### Proprietes des items flex

```css
.item {
    /* Alignement individuel (override align-items) */
    align-self: center;

    /* Croissance : combien l'item s'etend pour remplir l'espace */
    flex-grow: 1;    /* L'item prend l'espace disponible */
    flex-grow: 0;    /* L'item garde sa taille naturelle (defaut) */

    /* Retrecissement : combien l'item peut retrecir */
    flex-shrink: 1;  /* L'item peut retrecir (defaut) */
    flex-shrink: 0;  /* L'item ne retrecit jamais */

    /* Taille de base */
    flex-basis: 200px;  /* Taille initiale avant grow/shrink */
    flex-basis: auto;   /* Utilise width/height (defaut) */

    /* Raccourci flex (grow, shrink, basis) */
    flex: 1;           /* flex: 1 1 0 — prend tout l'espace equitablement */
    flex: 0 0 200px;   /* Taille fixe de 200px, ne grandit ni ne retrecit */

    /* Ordre d'affichage (visuel uniquement, ne change pas le DOM) */
    order: -1;  /* Affiche en premier */
    order: 0;   /* Position par defaut */
    order: 1;   /* Affiche en dernier */
}
```

> [!example] Cas d'usage classique : navigation
> ```css
> .navbar {
>     display: flex;
>     justify-content: space-between;
>     align-items: center;
>     padding: 1rem 2rem;
> }
>
> .navbar .logo { flex-shrink: 0; }
> .navbar .nav-links { display: flex; gap: 2rem; }
> .navbar .auth-buttons { display: flex; gap: 1rem; }
> ```

> [!tip] Centrage parfait avec Flexbox
> ```css
> .centre-parfait {
>     display: flex;
>     justify-content: center;
>     align-items: center;
>     min-height: 100vh;
> }
> ```
> C'est la methode la plus simple et la plus fiable pour centrer un element horizontalement ET verticalement.

---

## CSS Grid

Grid est un systeme de mise en page **bidimensionnel** (lignes ET colonnes). Ideal pour les mises en page complexes.

### Activer Grid

```css
.container {
    display: grid;

    /* Definir les colonnes */
    grid-template-columns: 200px 1fr 200px;  /* 3 colonnes */
    grid-template-columns: repeat(3, 1fr);    /* 3 colonnes egales */
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));  /* Responsive ! */

    /* Definir les lignes */
    grid-template-rows: auto 1fr auto;

    /* Espacement */
    gap: 20px;
    row-gap: 20px;
    column-gap: 10px;
}
```

```
Exemple : grid-template-columns: 200px 1fr 200px;

┌──────────┬────────────────────────────┬──────────┐
│  200px   │            1fr             │  200px   │
│ (fixe)   │    (espace restant)        │ (fixe)   │
└──────────┴────────────────────────────┴──────────┘
```

### L'unite `fr`

L'unite `fr` (fraction) represente une part de l'espace **disponible** :

```css
grid-template-columns: 1fr 2fr 1fr;
/* 1re colonne = 25%, 2e = 50%, 3e = 25% */

grid-template-columns: 200px 1fr 1fr;
/* 200px fixe, puis l'espace restant divise en 2 parts egales */
```

### Placer les items sur la grille

```css
.item-a {
    grid-column: 1 / 3;     /* De la ligne 1 a la ligne 3 (occupe 2 colonnes) */
    grid-row: 1 / 2;        /* Premiere ligne */
}

.item-b {
    grid-column: span 2;     /* Occupe 2 colonnes */
    grid-row: span 3;        /* Occupe 3 lignes */
}
```

### Grid areas : nommer les zones

```css
.container {
    display: grid;
    grid-template-columns: 200px 1fr;
    grid-template-rows: auto 1fr auto;
    grid-template-areas:
        "header  header"
        "sidebar content"
        "footer  footer";
    gap: 20px;
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.footer  { grid-area: footer; }
```

```
Resultat visuel :

┌──────────────────────────────────┐
│             header               │
├──────────┬───────────────────────┤
│          │                       │
│ sidebar  │       content         │
│          │                       │
├──────────┴───────────────────────┤
│             footer               │
└──────────────────────────────────┘
```

### Grille responsive automatique

```css
/* La grille la plus utile : items de minimum 250px
   qui remplissent automatiquement l'espace */
.grid-responsive {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 1.5rem;
}
```

```
Sur grand ecran (1200px) :
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│ Card │ │ Card │ │ Card │ │ Card │
└──────┘ └──────┘ └──────┘ └──────┘

Sur ecran moyen (800px) :
┌─────────┐ ┌─────────┐
│  Card   │ │  Card   │
├─────────┤ ├─────────┤
│  Card   │ │  Card   │
└─────────┘ └─────────┘

Sur petit ecran (400px) :
┌────────────────┐
│     Card       │
├────────────────┤
│     Card       │
├────────────────┤
│     Card       │
├────────────────┤
│     Card       │
└────────────────┘
```

> [!info] `auto-fit` vs `auto-fill`
> - **`auto-fit`** : les items s'etirent pour remplir l'espace (les colonnes vides s'effondrent)
> - **`auto-fill`** : les colonnes vides sont conservees (l'espace reste vide)
>
> En pratique, `auto-fit` est le plus utilise.

### Quand utiliser Flexbox vs Grid ?

| Critere | Flexbox | Grid |
|---------|---------|------|
| Dimension | 1D (ligne ou colonne) | 2D (lignes et colonnes) |
| Contenu | Le contenu definit la taille | La grille definit la taille |
| Usage | Navigation, boutons, alignements | Mises en page, grilles de cartes |
| Approche | Du contenu vers l'exterieur | Du conteneur vers l'interieur |

> [!tip] Regle simple
> - Alignement sur **une seule direction** → Flexbox
> - Placement sur **une grille 2D** → Grid
> - En cas de doute, commencez par Flexbox. Passez a Grid si ca devient complique.

---

## Responsive Design

Le responsive design permet a votre site de s'adapter a toutes les tailles d'ecran.

### Le viewport meta

```html
<!-- OBLIGATOIRE pour le responsive -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### Media queries

```css
/* Approche mobile-first (recommandee) :
   Le CSS de base est pour mobile,
   les media queries ajoutent les styles pour les grands ecrans */

/* Mobile (styles par defaut, pas de media query) */
.container {
    padding: 1rem;
}

.grid {
    display: grid;
    grid-template-columns: 1fr;  /* 1 colonne sur mobile */
    gap: 1rem;
}

/* Tablette (a partir de 768px) */
@media (min-width: 768px) {
    .container {
        padding: 2rem;
        max-width: 768px;
        margin: 0 auto;
    }

    .grid {
        grid-template-columns: repeat(2, 1fr);  /* 2 colonnes */
    }
}

/* Desktop (a partir de 1024px) */
@media (min-width: 1024px) {
    .container {
        max-width: 1200px;
    }

    .grid {
        grid-template-columns: repeat(3, 1fr);  /* 3 colonnes */
    }
}

/* Grand ecran (a partir de 1440px) */
@media (min-width: 1440px) {
    .container {
        max-width: 1400px;
    }

    .grid {
        grid-template-columns: repeat(4, 1fr);  /* 4 colonnes */
    }
}
```

### Breakpoints courants

```
Breakpoints standards :

Mobile         Tablette       Desktop         Grand ecran
│◄── 0-767 ──►│◄── 768-1023 ►│◄── 1024-1439 ►│◄── 1440+ ──►│

Breakpoints couramment utilises :
- 480px  : petits telephones
- 768px  : tablettes (portrait)
- 1024px : tablettes (paysage) / petits laptops
- 1200px : desktops
- 1440px : grands ecrans
```

> [!warning] Mobile-first vs Desktop-first
> - **Mobile-first** (`min-width`) : on part du mobile et on ajoute. Recommande car les styles mobiles sont plus simples.
> - **Desktop-first** (`max-width`) : on part du desktop et on reduit. Plus intuitif au debut mais produit souvent plus de CSS.

---

## Variables CSS (Custom Properties)

```css
:root {
    /* Definition des variables (sur l'element racine) */
    --couleur-primaire: #3498db;
    --couleur-secondaire: #2ecc71;
    --couleur-danger: #e74c3c;
    --couleur-texte: #333;
    --couleur-fond: #ffffff;

    --police-principale: 'Inter', sans-serif;
    --police-titre: 'Georgia', serif;

    --espacement-sm: 0.5rem;
    --espacement-md: 1rem;
    --espacement-lg: 2rem;

    --rayon-bordure: 8px;
    --ombre: 0 2px 8px rgba(0, 0, 0, 0.1);

    --largeur-max: 1200px;
}

/* Utilisation */
.btn-primary {
    background-color: var(--couleur-primaire);
    color: white;
    padding: var(--espacement-sm) var(--espacement-md);
    border-radius: var(--rayon-bordure);
    font-family: var(--police-principale);
}

/* Valeur de repli (fallback) */
.element {
    color: var(--couleur-custom, #333);  /* #333 si la variable n'existe pas */
}

/* Override dans un scope specifique */
.section-sombre {
    --couleur-texte: #fff;
    --couleur-fond: #1a1a1a;
}
```

> [!tip] Pourquoi utiliser des variables CSS ?
> - **Coherence** : une seule source de verite pour les couleurs, espacements, etc.
> - **Maintenabilite** : changez une variable, tout le site se met a jour
> - **Themes** : facilitent le dark mode et les changements de theme
> - **Lisibilite** : `var(--couleur-primaire)` est plus parlant que `#3498db`

---

## Carte Mentale

```
                              CSS FONDAMENTAUX
                                    │
        ┌──────────┬────────┬───────┼───────┬──────────┬──────────┐
        │          │        │       │       │          │          │
   Selecteurs  Specificite  Box   Display  Layout   Responsive  Variables
        │          │       Model    │       │          │          │
   ┌────┼────┐    │        │       │    ┌──┼──┐       │       :root
   │    │    │  1000=inline │    block  │     │   @media      --var()
element │  pseudo  100=id   │   inline Flex  Grid  min-width
 class  │  :hover   10=cls │  inline-  │     │   breakpoints
  #id   │  ::before  1=el  │  block  ┌─┼─┐  │
  [ ]   │            │     │        │ │ │  template
  > + ~ │         cascade  │   direction │  areas
        │         heritage │   justify  gap  fr
     groupement     │      │   align     │   repeat
                    │  ┌───┼───┐  wrap  minmax
                    │  │   │   │  gap   auto-fit
                    │ content  │
                    │ padding  │
                    │ border   │
                    │ margin   │
                    │          │
                 box-sizing: border-box
```

---

## Exercices

### Exercice 1 : Systeme de design minimal
Creez un fichier CSS avec des variables CSS pour un systeme de design complet :
- 5 couleurs (primaire, secondaire, succes, danger, neutre) en HSL
- 3 tailles de police (sm, md, lg) en rem
- 4 espacements (xs, sm, md, lg) en rem
- 1 ombre et 1 rayon de bordure
- Appliquez ces variables a une page avec des boutons, des cartes et du texte

### Exercice 2 : Layout Flexbox
Creez une barre de navigation responsive avec Flexbox :
- Logo a gauche, liens au centre, bouton a droite
- Sur mobile : tout empile verticalement avec un menu hamburger (CSS only)
- Les liens ont un effet `:hover` avec transition
- Le lien actif est visuellement different

### Exercice 3 : Grille de cartes avec Grid
Creez une grille de 6 cartes "projets" :
- Chaque carte a une image, un titre, une description et un bouton
- Utilisez `grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))`
- Ajoutez des hover effects (scale, ombre)
- Responsive sans media query (grace a minmax)

### Exercice 4 : Page complete responsive
Creez une page "landing page" complete :
- Header avec navigation (Flexbox)
- Section hero centree (Flexbox)
- Section features en grille (Grid)
- Footer sur 3 colonnes qui passe a 1 colonne sur mobile
- Mobile-first avec breakpoints a 768px et 1024px
- Utilisez uniquement des variables CSS pour les couleurs

---

## Liens

- [[01 - HTML Fondamentaux]] - Revoir la structure HTML
- [[03 - CSS Avance]] - Animations, transitions et methodologies CSS
