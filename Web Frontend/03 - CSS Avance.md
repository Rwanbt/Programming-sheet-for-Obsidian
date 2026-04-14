# CSS Avance

Apres avoir maitrise les fondamentaux du CSS (selecteurs, box model, Flexbox, Grid), il est temps de passer au niveau superieur. Ce cours explore les techniques avancees qui donnent vie a vos interfaces : transitions fluides, animations complexes, transformations visuelles, et les methodologies qui structurent un CSS maintenable a grande echelle.

Vous decouvrirez aussi les frameworks utilitaires comme Tailwind CSS, le dark mode, et les fonctionnalites CSS les plus recentes qui changent la facon dont nous ecrivons nos styles.

> [!tip] Analogie
> Si le CSS fondamental vous a appris a **construire** et **peindre** une maison, le CSS avance vous apprend a installer l'eclairage dynamique, les portes automatiques, les systemes domotiques et a organiser le chantier pour que 50 artisans puissent travailler en parallele sans se marcher dessus.

---

## Transitions

Les transitions permettent d'animer **le changement** d'une propriete CSS d'un etat a un autre, de maniere fluide.

```css
.bouton {
    background-color: #3498db;
    color: white;
    padding: 12px 24px;
    border: none;
    border-radius: 8px;

    /* Transition : propriete | duree | timing | delai */
    transition: background-color 0.3s ease;

    /* Plusieurs proprietes */
    transition: background-color 0.3s ease,
                transform 0.2s ease,
                box-shadow 0.3s ease;

    /* Toutes les proprietes (moins performant) */
    transition: all 0.3s ease;
}

.bouton:hover {
    background-color: #2980b9;
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}
```

### Les proprietes de transition

```css
.element {
    /* Proprietes individuelles */
    transition-property: background-color, transform;
    transition-duration: 0.3s, 0.2s;
    transition-timing-function: ease, ease-out;
    transition-delay: 0s, 0.1s;
}
```

### Fonctions de timing

```css
transition-timing-function: ease;        /* Lent-rapide-lent (defaut) */
transition-timing-function: linear;      /* Vitesse constante */
transition-timing-function: ease-in;     /* Lent au debut */
transition-timing-function: ease-out;    /* Lent a la fin */
transition-timing-function: ease-in-out; /* Lent debut + fin */
transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55); /* Custom */
```

```
Visualisation des timings :

ease:         ___---‾‾‾     (acceleration douce, deceleration douce)
linear:       ____----‾‾‾‾  (vitesse constante)
ease-in:      ________---‾  (lent puis rapide)
ease-out:     ‾---________  (rapide puis lent)
ease-in-out:  ____--‾‾____  (lent, rapide, lent)
```

> [!warning] Proprietes animables
> Toutes les proprietes CSS ne sont pas animables. Les plus couramment animees :
> - `opacity`, `transform` (les plus performantes, utilisent le GPU)
> - `color`, `background-color`, `border-color`
> - `width`, `height`, `margin`, `padding` (moins performantes, provoquent un reflow)
> - `box-shadow`, `border-radius`
>
> **Impossible a animer** : `display`, `font-family`, `position`

> [!tip] Performance
> Privilegiez `transform` et `opacity` pour vos animations. Ces proprietes sont traitees par le GPU (compositing layer) et ne declenchent pas de recalcul de layout.

---

## Animations CSS

Les animations permettent des mouvements plus complexes que les transitions : plusieurs etapes, boucles infinies, controle precis du timing.

### `@keyframes` : definir l'animation

```css
/* Animation simple : apparition en fondu */
@keyframes fondu-entree {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Animation avec etapes intermediaires */
@keyframes rebond {
    0% {
        transform: translateY(0);
    }
    30% {
        transform: translateY(-30px);
    }
    50% {
        transform: translateY(0);
    }
    70% {
        transform: translateY(-15px);
    }
    100% {
        transform: translateY(0);
    }
}

/* Animation de rotation continue */
@keyframes rotation {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}

/* Animation de pulsation */
@keyframes pulsation {
    0%, 100% {
        transform: scale(1);
        opacity: 1;
    }
    50% {
        transform: scale(1.05);
        opacity: 0.8;
    }
}
```

### Appliquer une animation

```css
.element {
    /* Raccourci */
    animation: fondu-entree 0.6s ease-out forwards;

    /* Proprietes individuelles */
    animation-name: fondu-entree;
    animation-duration: 0.6s;
    animation-timing-function: ease-out;
    animation-delay: 0s;
    animation-iteration-count: 1;         /* 1, 3, infinite */
    animation-direction: normal;          /* normal, reverse, alternate */
    animation-fill-mode: forwards;        /* none, forwards, backwards, both */
    animation-play-state: running;        /* running, paused */
}

/* Rotation infinie */
.spinner {
    animation: rotation 1s linear infinite;
}

/* Pulsation continue avec alternance */
.notification {
    animation: pulsation 2s ease-in-out infinite alternate;
}

/* Plusieurs animations */
.element {
    animation: fondu-entree 0.6s ease-out,
               rebond 1s ease 0.6s;
}
```

> [!info] `animation-fill-mode`
> - `none` : l'element revient a son etat initial apres l'animation
> - `forwards` : l'element conserve l'etat de la **derniere** keyframe
> - `backwards` : l'element prend l'etat de la **premiere** keyframe pendant le delay
> - `both` : combine forwards et backwards

> [!example] Loader / Spinner CSS pur
> ```css
> .loader {
>     width: 40px;
>     height: 40px;
>     border: 4px solid #f3f3f3;
>     border-top: 4px solid #3498db;
>     border-radius: 50%;
>     animation: rotation 0.8s linear infinite;
> }
> ```

### Animation declenchee au scroll (approche CSS)

```css
/* Avec Intersection Observer en CSS natif (animation-timeline) */
@keyframes apparition {
    from {
        opacity: 0;
        transform: translateY(50px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Classes utilitaires pour animation au scroll (a declencher via JS ou scroll-timeline) */
.anime-au-scroll {
    opacity: 0;
    transform: translateY(50px);
    transition: opacity 0.6s ease, transform 0.6s ease;
}

.anime-au-scroll.visible {
    opacity: 1;
    transform: translateY(0);
}
```

---

## Transformations CSS

Les transformations modifient l'apparence visuelle d'un element sans affecter le flux du document.

```css
.element {
    /* Deplacer */
    transform: translateX(50px);
    transform: translateY(-20px);
    transform: translate(50px, -20px);   /* X et Y */

    /* Tourner */
    transform: rotate(45deg);
    transform: rotate(-90deg);

    /* Echelle (agrandir/reduire) */
    transform: scale(1.5);        /* 150% */
    transform: scale(0.8);        /* 80% */
    transform: scaleX(2);         /* 200% horizontal */

    /* Incliner */
    transform: skew(10deg);
    transform: skewX(15deg);
    transform: skewY(-5deg);

    /* Combinaisons */
    transform: translate(-50%, -50%) rotate(45deg) scale(1.2);

    /* Point d'origine de la transformation */
    transform-origin: center;      /* Defaut */
    transform-origin: top left;
    transform-origin: 50% 0%;
}
```

> [!example] Centrage absolu avec transform
> ```css
> .centre-absolu {
>     position: absolute;
>     top: 50%;
>     left: 50%;
>     transform: translate(-50%, -50%);
> }
> ```
> Cette technique est utile quand on ne connait pas les dimensions de l'element.

### Carte qui se retourne au survol (3D)

```css
.carte-container {
    perspective: 1000px;
    width: 300px;
    height: 200px;
}

.carte {
    width: 100%;
    height: 100%;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.6s ease;
}

.carte-container:hover .carte {
    transform: rotateY(180deg);
}

.carte-face, .carte-dos {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    border-radius: 12px;
}

.carte-face {
    background: #3498db;
    color: white;
}

.carte-dos {
    background: #2ecc71;
    color: white;
    transform: rotateY(180deg);
}
```

---

## Pseudo-elements : usages creatifs

Au-dela de `::before` et `::after` pour du texte, les pseudo-elements permettent des decorations complexes sans HTML supplementaire.

### Effet de soulignement anime

```css
.lien-anime {
    position: relative;
    text-decoration: none;
    color: #333;
}

.lien-anime::after {
    content: "";
    position: absolute;
    bottom: -2px;
    left: 0;
    width: 0;
    height: 2px;
    background-color: #3498db;
    transition: width 0.3s ease;
}

.lien-anime:hover::after {
    width: 100%;
}
```

### Tooltip CSS pur

```css
.tooltip {
    position: relative;
    cursor: help;
}

.tooltip::after {
    content: attr(data-tooltip);
    position: absolute;
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    padding: 8px 12px;
    background: #333;
    color: white;
    border-radius: 6px;
    font-size: 0.85rem;
    white-space: nowrap;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.3s ease, transform 0.3s ease;
    transform: translateX(-50%) translateY(5px);
}

.tooltip:hover::after {
    opacity: 1;
    transform: translateX(-50%) translateY(-5px);
}
```

```html
<span class="tooltip" data-tooltip="Ceci est un tooltip">Survolez-moi</span>
```

### Compteur CSS

```css
.liste-numerotee {
    counter-reset: etape;
}

.liste-numerotee li {
    counter-increment: etape;
    padding-left: 2.5rem;
    position: relative;
}

.liste-numerotee li::before {
    content: counter(etape);
    position: absolute;
    left: 0;
    width: 1.8rem;
    height: 1.8rem;
    background: #3498db;
    color: white;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.85rem;
    font-weight: bold;
}
```

---

## Convention de nommage BEM

BEM (Block Element Modifier) est une convention de nommage qui rend le CSS plus lisible et maintenable.

### Structure BEM

```
Block:    .card
Element:  .card__title       (partie du block, separee par __)
Modifier: .card--featured    (variation du block, separee par --)
          .card__title--large (variation de l'element)
```

```html
<!-- Exemple complet -->
<article class="card card--featured">
    <img class="card__image" src="photo.jpg" alt="Photo">
    <div class="card__content">
        <h3 class="card__title">Titre de la carte</h3>
        <p class="card__description">Description...</p>
        <a class="card__link card__link--primary" href="#">En savoir plus</a>
    </div>
</article>
```

```css
/* CSS correspondant */
.card {
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
}

.card--featured {
    border-color: #3498db;
    box-shadow: 0 4px 12px rgba(52, 152, 219, 0.2);
}

.card__image {
    width: 100%;
    height: 200px;
    object-fit: cover;
}

.card__content {
    padding: 1.5rem;
}

.card__title {
    font-size: 1.25rem;
    margin-bottom: 0.5rem;
}

.card__description {
    color: #666;
    line-height: 1.6;
}

.card__link {
    display: inline-block;
    margin-top: 1rem;
    text-decoration: none;
}

.card__link--primary {
    color: #3498db;
    font-weight: 600;
}
```

> [!tip] Avantages de BEM
> - **Pas de conflits** : chaque classe est unique et specifique
> - **Specificite plate** : tout est en une seule classe (0,0,1,0), pas de nesting
> - **Auto-documentant** : `.card__title--large` dit clairement ce que c'est
> - **Reutilisable** : les blocks sont independants et deplacables

---

## Architectures CSS

### OOCSS (Object-Oriented CSS)

Separation de la **structure** et de la **peau** (apparence) :

```css
/* Structure (reutilisable) */
.media {
    display: flex;
    gap: 1rem;
    align-items: flex-start;
}

.media__body { flex: 1; }

/* Peau (apparence, interchangeable) */
.theme-clair { background: #fff; color: #333; }
.theme-sombre { background: #1a1a1a; color: #eee; }
```

### SMACSS (Scalable and Modular Architecture for CSS)

Categorisation du CSS en 5 types :

```css
/* 1. Base : reset et styles par defaut */
html { font-size: 16px; }
a { text-decoration: none; }

/* 2. Layout : structure de la page (prefixe l-) */
.l-header { /* ... */ }
.l-sidebar { width: 250px; }

/* 3. Module : composants reutilisables */
.card { /* ... */ }
.btn { /* ... */ }

/* 4. State : etats dynamiques (prefixe is-) */
.is-active { /* ... */ }
.is-hidden { display: none; }

/* 5. Theme : variations visuelles */
.theme-dark { /* ... */ }
```

### ITCSS (Inverted Triangle CSS)

Organisation en couches, de la plus generique a la plus specifique :

```
  Settings    → Variables, config         (pas de CSS genere)
   Tools      → Mixins, fonctions         (pas de CSS genere)
    Generic   → Reset, normalize          (tres generique)
     Elements → Styles de base (h1, p, a) (elements nus)
      Objects → Patterns de layout        (pas de decoration)
       Components → Composants UI         (specifique)
        Utilities → Helpers (!important)  (le plus specifique)
```

---

## Tailwind CSS

Tailwind CSS est un framework CSS **utility-first** : au lieu d'ecrire des classes semantiques, vous composez le design directement dans le HTML avec des classes utilitaires.

### Philosophie

```html
<!-- CSS traditionnel -->
<button class="btn-primary">Cliquer</button>

<!-- Tailwind CSS -->
<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
    Cliquer
</button>
```

### Classes courantes

```html
<!-- Espacement -->
<div class="p-4">     <!-- padding: 1rem -->
<div class="px-6">    <!-- padding-left/right: 1.5rem -->
<div class="mt-8">    <!-- margin-top: 2rem -->
<div class="space-y-4"> <!-- gap vertical entre enfants -->

<!-- Flexbox -->
<div class="flex items-center justify-between gap-4">
<div class="flex-1">  <!-- flex: 1 1 0% -->

<!-- Grid -->
<div class="grid grid-cols-3 gap-6">
<div class="col-span-2">

<!-- Typographie -->
<p class="text-lg font-semibold text-gray-700 leading-relaxed">
<p class="text-sm text-center uppercase tracking-wide">

<!-- Couleurs -->
<div class="bg-white text-gray-900 border border-gray-200">
<div class="bg-gradient-to-r from-blue-500 to-purple-600">

<!-- Dimensions -->
<div class="w-full max-w-lg h-screen min-h-[400px]">

<!-- Effets -->
<div class="rounded-lg shadow-md hover:shadow-xl transition-shadow">
<div class="opacity-75 hover:opacity-100">
```

### Prefixes responsive

```html
<!-- Mobile-first : les classes sans prefixe = mobile -->
<div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
    <!-- 1 col mobile, 2 col sm(640px), 3 col md(768px), 4 col lg(1024px) -->
</div>

<p class="text-sm md:text-base lg:text-lg">
    <!-- Taille de texte responsive -->
</p>
```

> [!info] Tailwind vs CSS traditionnel
> | Aspect | Tailwind | CSS traditionnel |
> |--------|----------|-----------------|
> | Courbe d'apprentissage | Classes a memoriser | Syntaxe CSS classique |
> | Fichier CSS | Tres petit (purge) | Peut devenir volumineux |
> | HTML | Verbeux | Propre |
> | Prototypage | Tres rapide | Plus lent |
> | Maintenabilite | Composants obligatoires | BEM/methodologie necessaire |
> | Personnalisation | `tailwind.config.js` | Variables CSS |

> [!warning] Quand utiliser Tailwind ?
> Tailwind excelle dans les projets avec un framework de composants (React, Vue, Svelte) ou le HTML est reparti dans des composants reutilisables. Sans composants, le HTML devient vite illisible et la duplication est un probleme.

---

## Dark Mode

### Detection automatique avec `prefers-color-scheme`

```css
/* Strategie avec variables CSS */
:root {
    --bg-primary: #ffffff;
    --bg-secondary: #f5f5f5;
    --text-primary: #1a1a1a;
    --text-secondary: #666666;
    --border-color: #e0e0e0;
    --accent: #3498db;
}

@media (prefers-color-scheme: dark) {
    :root {
        --bg-primary: #1a1a1a;
        --bg-secondary: #2d2d2d;
        --text-primary: #e0e0e0;
        --text-secondary: #a0a0a0;
        --border-color: #404040;
        --accent: #5dade2;
    }
}

/* Les composants utilisent les variables — pas de duplication */
body {
    background-color: var(--bg-primary);
    color: var(--text-primary);
}

.card {
    background-color: var(--bg-secondary);
    border: 1px solid var(--border-color);
}
```

### Toggle manuel avec attribut data

```css
/* Theme clair (defaut) */
:root {
    --bg-primary: #ffffff;
    --text-primary: #1a1a1a;
}

/* Theme sombre via attribut */
[data-theme="dark"] {
    --bg-primary: #1a1a1a;
    --text-primary: #e0e0e0;
}
```

```html
<html data-theme="light">
<!-- Le JavaScript toggle entre "light" et "dark" sur cet attribut -->
```

> [!tip] Strategie complete pour le dark mode
> 1. Definissez toutes vos couleurs en variables CSS
> 2. Utilisez `prefers-color-scheme` pour le mode automatique
> 3. Ajoutez un toggle manuel avec `data-theme`
> 4. Sauvegardez la preference dans `localStorage`
> 5. Attention aux images : prevoyez des versions claires/sombres ou utilisez `filter: invert(1)` avec parcimonie

---

## Fonctions CSS

### `calc()`

Calculs dynamiques melangeant differentes unites :

```css
.sidebar {
    width: calc(100% - 300px);
    height: calc(100vh - 80px);
    padding: calc(1rem + 5px);
    font-size: calc(14px + 0.5vw);  /* Taille fluide */
}
```

### `min()`, `max()`, `clamp()`

```css
.container {
    /* min() : prend la plus petite valeur */
    width: min(1200px, 90%);
    /* = 90% sur petit ecran, 1200px quand l'ecran est assez grand */

    /* max() : prend la plus grande valeur */
    font-size: max(16px, 1.2vw);
    /* Jamais plus petit que 16px */

    /* clamp(minimum, ideal, maximum) */
    font-size: clamp(1rem, 2.5vw, 2rem);
    /* Au minimum 1rem, idealement 2.5vw, au maximum 2rem */

    padding: clamp(1rem, 3vw, 3rem);
}
```

> [!tip] Typographie fluide avec `clamp()`
> ```css
> h1 { font-size: clamp(1.8rem, 4vw, 3.5rem); }
> h2 { font-size: clamp(1.4rem, 3vw, 2.5rem); }
> p  { font-size: clamp(1rem, 1.5vw, 1.25rem); }
> ```
> Plus besoin de media queries pour adapter la taille du texte.

---

## Filtres CSS

```css
.image {
    /* Filtres individuels */
    filter: blur(5px);
    filter: grayscale(100%);
    filter: brightness(1.2);
    filter: contrast(1.5);
    filter: saturate(2);
    filter: sepia(80%);
    filter: hue-rotate(90deg);
    filter: opacity(50%);
    filter: drop-shadow(4px 4px 8px rgba(0,0,0,0.3));

    /* Combinaison de filtres */
    filter: brightness(1.1) contrast(1.1) saturate(1.2);
}

/* Filtre au survol */
.carte-image:hover img {
    filter: brightness(1.1);
    transition: filter 0.3s ease;
}

/* Image grise qui devient coloree au hover */
.portfolio-image {
    filter: grayscale(100%);
    transition: filter 0.4s ease;
}
.portfolio-image:hover {
    filter: grayscale(0%);
}
```

### `backdrop-filter`

Applique un filtre a l'arriere-plan **derriere** l'element (effet verre depoli) :

```css
.navbar-glass {
    background: rgba(255, 255, 255, 0.7);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);  /* Safari */
    border-bottom: 1px solid rgba(255, 255, 255, 0.3);
}

.modal-overlay {
    background: rgba(0, 0, 0, 0.3);
    backdrop-filter: blur(4px);
}
```

---

## Scroll Behavior

### Defilement fluide

```css
html {
    scroll-behavior: smooth;
}
```

Les clics sur les ancres (`<a href="#section">`) defileront maintenant de maniere fluide.

### Scroll Snap

Force le defilement a "s'accrocher" a des points precis :

```css
.carousel {
    display: flex;
    overflow-x: auto;
    scroll-snap-type: x mandatory;  /* Snap horizontal obligatoire */
    gap: 1rem;
}

.carousel-item {
    scroll-snap-align: start;       /* S'aligne au debut */
    flex: 0 0 100%;                 /* Chaque item = 100% de largeur */
}

/* Scroll vertical type "pages" */
.sections-plein-ecran {
    height: 100vh;
    overflow-y: auto;
    scroll-snap-type: y mandatory;
}

.section-page {
    height: 100vh;
    scroll-snap-align: start;
}
```

---

## CSS Moderne : fonctionnalites recentes

### Container Queries

Adapter le style d'un composant en fonction de la taille de **son conteneur** (pas du viewport) :

```css
.card-container {
    container-type: inline-size;
    container-name: card;
}

@container card (min-width: 400px) {
    .card {
        display: flex;
        flex-direction: row;
    }
}

@container card (max-width: 399px) {
    .card {
        display: flex;
        flex-direction: column;
    }
}
```

### Le selecteur `:has()`

Le "selecteur parent" tant attendu :

```css
/* Styler un form qui contient un input invalide */
form:has(input:invalid) {
    border: 2px solid red;
}

/* Card qui contient une image → layout horizontal */
.card:has(img) {
    display: flex;
}

/* Section sans titre → plus de padding */
section:not(:has(h2)) {
    padding-top: 3rem;
}

/* Label dont le checkbox est coche */
label:has(input:checked) {
    background-color: #e8f5e9;
}
```

### CSS Nesting (natif)

Ecrire du CSS imbrique directement, sans preprocesseur :

```css
.card {
    background: white;
    border-radius: 8px;
    padding: 1.5rem;

    & .card__title {
        font-size: 1.25rem;
        margin-bottom: 0.5rem;
    }

    & .card__body {
        color: #666;
        line-height: 1.6;
    }

    &:hover {
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    }

    @media (min-width: 768px) {
        padding: 2rem;
    }
}
```

> [!warning] Support navigateur
> Container queries, `:has()`, et le nesting natif sont supportes par les navigateurs modernes (Chrome, Firefox, Safari recents). Verifiez toujours la compatibilite sur [caniuse.com](https://caniuse.com) avant d'utiliser ces fonctionnalites en production.

---

## Carte Mentale

```
                           CSS AVANCE
                               │
       ┌────────┬──────┬───────┼───────┬──────────┬──────────┐
       │        │      │       │       │          │          │
  Transitions Anim. Transform Methodo. Tailwind  Dark Mode  Modern
       │        │      │       │       │          │          │
   property  @keyframes │    BEM    utility  prefers-    :has()
   duration  infinite  │    B__E   -first   color-    container
   timing    alternate │    B--M   sm: md:  scheme    nesting
   delay     forwards  │   OOCSS   lg: xl:  variables
       │        │      │   SMACSS     │       toggle
    ease    fill-mode  │   ITCSS      │
    linear    play   ┌─┼─┐           │
    cubic    -state  │ │ │      ┌────┼────┐
                    translate  calc()     │
                    rotate    min()    Filtres
                    scale     max()      │
                    skew     clamp()   blur
                    origin             grayscale
                                      backdrop
                                      drop-shadow
```

---

## Exercices

### Exercice 1 : Bouton anime complet
Creez un bouton avec :
- Transition de couleur et d'ombre au hover
- Un pseudo-element `::before` qui cree un effet de "remplissage" de gauche a droite
- Un effet de scale au clic (`:active`)
- Un etat focus visible accessible

### Exercice 2 : Carte flip 3D
Creez une carte qui se retourne au survol :
- Face avant : image + titre
- Face arriere : description + lien
- Animation de rotation 3D fluide (perspective, preserve-3d)
- Version mobile : flip au clic via checkbox hack

### Exercice 3 : Dark mode complet
Implementez un dark mode pour une petite page :
- Toutes les couleurs en variables CSS
- Detection automatique `prefers-color-scheme`
- Toggle manuel avec `data-theme`
- Transitions douces lors du changement de theme
- Adaptez les images (filtres brightness)

### Exercice 4 : Page avec Tailwind
Reconstruisez une section "features" avec Tailwind CSS (via CDN play) :
- 3 cartes en grille responsive
- Hover effects
- Icones SVG
- Mobile-first avec prefixes sm:/md:/lg:

---

## Liens

- [[02 - CSS Fondamentaux]] - Revoir les bases CSS
- [[04 - Projet Web Statique]] - Mettre en pratique dans un projet complet
