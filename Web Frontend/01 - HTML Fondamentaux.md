# HTML Fondamentaux

Le **HTML** (HyperText Markup Language) est le langage de balisage qui structure tout le contenu du web. Chaque page que vous visitez, chaque application web que vous utilisez repose sur une fondation HTML. Ce n'est pas un langage de programmation : il ne contient ni boucles, ni conditions, ni variables. C'est un langage **descriptif** qui dit au navigateur *quoi* afficher, pas *comment* le faire (c'est le role du CSS) ni *comment* il se comporte (c'est le role du JavaScript).

Comprendre le HTML, c'est comprendre la structure meme du web. Ce cours couvre tout ce dont vous avez besoin pour ecrire du HTML solide, semantique et accessible.

> [!tip] Analogie
> Pensez au HTML comme au **squelette** d'un corps humain. Le squelette donne la structure, definit ou se trouvent la tete, le torse, les bras et les jambes. Le CSS serait la peau, les vetements et l'apparence. Le JavaScript serait les muscles et le systeme nerveux qui permettent le mouvement et l'interaction. Sans squelette, rien ne tient debout.

---

## Comment fonctionne le Web

Avant d'ecrire du HTML, il faut comprendre comment une page arrive jusqu'a votre ecran.

### Le modele Client-Serveur

```
┌─────────────┐                                    ┌─────────────┐
│             │   1. Requete HTTP (GET /index.html) │             │
│   CLIENT    │ ──────────────────────────────────► │   SERVEUR   │
│ (Navigateur)│                                     │    (Web)    │
│             │ ◄────────────────────────────────── │             │
│  Chrome,    │   2. Reponse HTTP (200 OK)          │  Apache,    │
│  Firefox,   │      + fichier HTML                 │  Nginx,     │
│  Safari...  │                                     │  Node.js... │
└─────────────┘                                    └─────────────┘
       │                                                  │
       │  3. Le navigateur parse le HTML                  │
       │  4. Il demande les ressources                    │
       │     (CSS, JS, images...)                         │
       │  5. Il construit le DOM                          │
       │  6. Il affiche la page                           │
       ▼                                                  │
┌─────────────┐                                           │
│  PAGE WEB   │                                           │
│  AFFICHEE   │                                           │
└─────────────┘                                           │
```

> [!info] Le protocole HTTP
> HTTP (HyperText Transfer Protocol) est le protocole de communication du web. Chaque echange se fait en deux temps : une **requete** (du client vers le serveur) et une **reponse** (du serveur vers le client). HTTPS est la version securisee (chiffree) de ce protocole.

### Les etapes detaillees

1. Vous tapez une URL dans la barre d'adresse (ex: `https://example.com/index.html`)
2. Le navigateur resout le nom de domaine via le **DNS** (Domain Name System) pour obtenir l'adresse IP du serveur
3. Le navigateur envoie une requete HTTP GET au serveur
4. Le serveur trouve le fichier demande et renvoie une reponse HTTP avec le contenu HTML
5. Le navigateur **parse** (analyse) le HTML ligne par ligne
6. Pour chaque ressource externe (CSS, JS, images), le navigateur envoie des requetes supplementaires
7. Le navigateur construit le **DOM** (Document Object Model), un arbre representant la structure de la page
8. La page est rendue a l'ecran

---

## Structure d'un document HTML

Tout document HTML valide suit une structure precise :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Description de la page pour les moteurs de recherche">
    <title>Titre de la page</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <!-- Le contenu visible de la page -->
    <h1>Bienvenue</h1>
    <p>Ceci est un paragraphe.</p>

    <script src="script.js"></script>
</body>
</html>
```

### Explication de chaque partie

#### `<!DOCTYPE html>`

Ce n'est pas une balise HTML. C'est une **declaration** qui indique au navigateur que le document utilise HTML5. Sans elle, le navigateur peut entrer en mode "quirks" et interpeter le code differemment.

#### `<html lang="fr">`

L'element racine qui contient tout le document. L'attribut `lang` indique la langue du contenu, ce qui est important pour :
- Les lecteurs d'ecran (accessibilite)
- Les moteurs de recherche (SEO)
- Les outils de traduction automatique

#### `<head>` - Les metadonnees

Le `<head>` contient des informations **sur** la page, mais rien de visible directement :

```html
<head>
    <!-- Encodage des caracteres -->
    <meta charset="UTF-8">

    <!-- Adaptation mobile (responsive) -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- Description pour les moteurs de recherche -->
    <meta name="description" content="Cours complet sur les fondamentaux du HTML">

    <!-- Mots-cles (moins utilise aujourd'hui) -->
    <meta name="keywords" content="HTML, web, cours">

    <!-- Auteur -->
    <meta name="author" content="Votre nom">

    <!-- Titre affiche dans l'onglet du navigateur -->
    <title>HTML Fondamentaux - Cours</title>

    <!-- Lien vers une feuille de style externe -->
    <link rel="stylesheet" href="styles/main.css">

    <!-- Favicon (icone de l'onglet) -->
    <link rel="icon" href="favicon.ico" type="image/x-icon">
</head>
```

> [!warning] Le charset UTF-8
> Placez **toujours** `<meta charset="UTF-8">` en premiere position dans le `<head>`. Si le navigateur commence a interpreter le texte avec un mauvais encodage, les caracteres accentues (e, a, c) seront mal affiches.

#### `<body>` - Le contenu visible

Le `<body>` contient tout ce que l'utilisateur voit et avec quoi il interagit.

---

## Elements de texte

### Titres : `<h1>` a `<h6>`

Les titres definissent la hierarchie du contenu. Il y a 6 niveaux :

```html
<h1>Titre principal (un seul par page)</h1>
<h2>Sous-titre de niveau 2</h2>
<h3>Sous-titre de niveau 3</h3>
<h4>Sous-titre de niveau 4</h4>
<h5>Sous-titre de niveau 5</h5>
<h6>Sous-titre de niveau 6</h6>
```

> [!warning] Regles importantes pour les titres
> - Un seul `<h1>` par page (le titre principal)
> - Ne sautez pas de niveaux (pas de `<h1>` suivi directement d'un `<h4>`)
> - Les titres ne sont pas faits pour mettre du texte en gros. Utilisez le CSS pour le style.
> - Les moteurs de recherche et les lecteurs d'ecran utilisent la hierarchie des titres pour comprendre la structure.

### Paragraphes et texte en ligne

```html
<!-- Paragraphe -->
<p>Ceci est un paragraphe. Les navigateurs ajoutent automatiquement
   un espacement avant et apres chaque paragraphe.</p>

<!-- Texte en gras (importance semantique) -->
<p>Ceci est <strong>tres important</strong> pour comprendre.</p>

<!-- Texte en italique (emphase semantique) -->
<p>Le mot <em>fondamental</em> est mis en emphase.</p>

<!-- Span : conteneur en ligne sans semantique -->
<p>Le prix est de <span class="prix">29,99 EUR</span></p>

<!-- Retour a la ligne -->
<p>Premiere ligne<br>Deuxieme ligne</p>

<!-- Ligne horizontale (separation thematique) -->
<hr>

<!-- Citation en bloc -->
<blockquote cite="https://example.com">
    <p>Le web est pour tout le monde.</p>
    <footer>— Tim Berners-Lee</footer>
</blockquote>

<!-- Texte preformate (conserve les espaces et sauts de ligne) -->
<pre>
    Texte     avec
    des espaces   conserves
</pre>

<!-- Code en ligne -->
<p>Utilisez la balise <code>&lt;p&gt;</code> pour un paragraphe.</p>

<!-- Bloc de code -->
<pre><code>
function saluer() {
    console.log("Bonjour !");
}
</code></pre>
```

> [!info] `<strong>` vs `<b>` et `<em>` vs `<i>`
> - `<strong>` = importance semantique forte (le contenu est **important**)
> - `<b>` = mise en gras purement visuelle, sans signification semantique
> - `<em>` = emphase semantique (le sens change si on enleve l'emphase)
> - `<i>` = italique purement visuel (termes techniques, noms propres etrangers)
>
> Preferez `<strong>` et `<em>` pour une meilleure accessibilite.

---

## Les listes

### Liste non ordonnee (`<ul>`)

Pour des elements sans ordre particulier :

```html
<ul>
    <li>HTML</li>
    <li>CSS</li>
    <li>JavaScript</li>
</ul>
```

### Liste ordonnee (`<ol>`)

Pour des elements avec un ordre logique :

```html
<ol>
    <li>Ouvrir l'editeur</li>
    <li>Creer un fichier index.html</li>
    <li>Ecrire le code HTML</li>
    <li>Ouvrir le fichier dans le navigateur</li>
</ol>

<!-- Attributs utiles -->
<ol start="5" reversed type="A">
    <li>Element E</li>
    <li>Element D</li>
</ol>
```

### Liste de definitions (`<dl>`)

Pour des paires terme/definition :

```html
<dl>
    <dt>HTML</dt>
    <dd>HyperText Markup Language - langage de balisage du web</dd>

    <dt>CSS</dt>
    <dd>Cascading Style Sheets - langage de mise en forme</dd>

    <dt>JavaScript</dt>
    <dd>Langage de programmation du web</dd>
</dl>
```

### Listes imbriquees

```html
<ul>
    <li>Frontend
        <ul>
            <li>HTML</li>
            <li>CSS</li>
            <li>JavaScript</li>
        </ul>
    </li>
    <li>Backend
        <ul>
            <li>Node.js</li>
            <li>Python</li>
        </ul>
    </li>
</ul>
```

---

## Les liens

Les liens sont l'essence meme du **hypertexte**. La balise `<a>` (anchor) cree des liens :

```html
<!-- Lien externe -->
<a href="https://developer.mozilla.org">Documentation MDN</a>

<!-- Lien interne (page du meme site) -->
<a href="contact.html">Page de contact</a>

<!-- Lien relatif vers un dossier parent -->
<a href="../index.html">Retour a l'accueil</a>

<!-- Ouvrir dans un nouvel onglet -->
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
    Lien externe (nouvel onglet)
</a>

<!-- Lien email -->
<a href="mailto:contact@example.com">Envoyer un email</a>

<!-- Lien telephone -->
<a href="tel:+33612345678">Appeler</a>

<!-- Lien ancre (vers une section de la meme page) -->
<a href="#section-contact">Aller au contact</a>

<!-- ... plus bas dans la page ... -->
<section id="section-contact">
    <h2>Contact</h2>
</section>

<!-- Lien de telechargement -->
<a href="document.pdf" download>Telecharger le PDF</a>
```

> [!warning] Securite avec `target="_blank"`
> Quand vous utilisez `target="_blank"`, ajoutez **toujours** `rel="noopener noreferrer"`. Sans cela, la page ouverte peut acceder a `window.opener` et potentiellement rediriger votre page d'origine (attaque de type "tabnapping").

---

## Les images

```html
<!-- Image simple -->
<img src="photo.jpg" alt="Description de l'image" width="800" height="600">

<!-- Image avec chemin relatif -->
<img src="images/logo.png" alt="Logo de l'entreprise">

<!-- Image avec figure et legende -->
<figure>
    <img src="graphique.png" alt="Graphique montrant l'evolution des ventes en 2024">
    <figcaption>Figure 1 : Evolution des ventes au cours de l'annee 2024</figcaption>
</figure>
```

> [!warning] L'attribut `alt` est obligatoire
> L'attribut `alt` fournit un texte alternatif quand l'image ne peut pas etre affichee et est **essentiel** pour l'accessibilite (lecteurs d'ecran). Regles :
> - Decrivez ce que montre l'image de maniere concise
> - Si l'image est purement decorative, utilisez `alt=""`
> - Ne commencez pas par "Image de..." (le lecteur d'ecran le sait deja)

> [!tip] Dimensions explicites
> Specifier `width` et `height` permet au navigateur de reserver l'espace avant le chargement de l'image, evitant ainsi les "sauts" de mise en page (layout shift).

### Formats d'image courants

| Format | Usage | Transparence | Animation |
|--------|-------|-------------|-----------|
| JPEG | Photos, images complexes | Non | Non |
| PNG | Logos, captures d'ecran | Oui | Non |
| GIF | Animations simples | Oui (1 bit) | Oui |
| SVG | Icones, graphiques vectoriels | Oui | Oui |
| WebP | Remplacement moderne JPEG/PNG | Oui | Oui |
| AVIF | Compression superieure | Oui | Oui |

---

## Les tableaux

Les tableaux servent a presenter des **donnees tabulaires** (jamais pour la mise en page !) :

```html
<table>
    <caption>Horaires des cours</caption>
    <thead>
        <tr>
            <th>Jour</th>
            <th>Matiere</th>
            <th>Horaire</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Lundi</td>
            <td>HTML</td>
            <td>09:00 - 12:00</td>
        </tr>
        <tr>
            <td>Mardi</td>
            <td>CSS</td>
            <td>14:00 - 17:00</td>
        </tr>
        <tr>
            <td>Mercredi</td>
            <td>JavaScript</td>
            <td>09:00 - 12:00</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td colspan="3">Total : 9 heures par semaine</td>
        </tr>
    </tfoot>
</table>
```

### Fusion de cellules

```html
<table>
    <tr>
        <th>Nom</th>
        <th colspan="2">Contact</th>  <!-- Fusionne 2 colonnes -->
    </tr>
    <tr>
        <td rowspan="2">Dupont</td>  <!-- Fusionne 2 lignes -->
        <td>email@example.com</td>
        <td>06 12 34 56 78</td>
    </tr>
    <tr>
        <!-- La cellule "Dupont" occupe deja cette ligne -->
        <td>autre@example.com</td>
        <td>01 23 45 67 89</td>
    </tr>
</table>
```

```
Representation visuelle du colspan/rowspan :

┌─────────┬───────────────────────┐
│   Nom   │       Contact         │  ← colspan="2"
├─────────┼───────────┬───────────┤
│         │ email@... │ 06 12...  │
│ Dupont  ├───────────┼───────────┤  ← rowspan="2"
│         │ autre@... │ 01 23...  │
└─────────┴───────────┴───────────┘
```

> [!info] Accessibilite des tableaux
> - Utilisez `<caption>` pour donner un titre au tableau
> - Utilisez `<th>` (et non `<td>`) pour les en-tetes
> - L'attribut `scope="col"` ou `scope="row"` sur les `<th>` aide les lecteurs d'ecran

---

## Les formulaires

Les formulaires permettent de collecter des donnees aupres de l'utilisateur :

```html
<form action="/api/inscription" method="POST">
    <fieldset>
        <legend>Informations personnelles</legend>

        <label for="nom">Nom complet :</label>
        <input type="text" id="nom" name="nom" required
               placeholder="Jean Dupont">

        <label for="email">Email :</label>
        <input type="email" id="email" name="email" required
               placeholder="jean@example.com">

        <label for="mdp">Mot de passe :</label>
        <input type="password" id="mdp" name="mdp" required
               minlength="8"
               pattern="(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}"
               title="Au moins 8 caracteres, une majuscule, une minuscule et un chiffre">

        <label for="age">Age :</label>
        <input type="number" id="age" name="age" min="18" max="120">

        <label for="date-naissance">Date de naissance :</label>
        <input type="date" id="date-naissance" name="date_naissance">
    </fieldset>

    <fieldset>
        <legend>Preferences</legend>

        <p>Niveau :</p>
        <label>
            <input type="radio" name="niveau" value="debutant" checked>
            Debutant
        </label>
        <label>
            <input type="radio" name="niveau" value="intermediaire">
            Intermediaire
        </label>
        <label>
            <input type="radio" name="niveau" value="avance">
            Avance
        </label>

        <p>Technologies connues :</p>
        <label>
            <input type="checkbox" name="tech" value="html"> HTML
        </label>
        <label>
            <input type="checkbox" name="tech" value="css"> CSS
        </label>
        <label>
            <input type="checkbox" name="tech" value="js"> JavaScript
        </label>

        <label for="pays">Pays :</label>
        <select id="pays" name="pays">
            <option value="">-- Choisir --</option>
            <option value="fr">France</option>
            <option value="be">Belgique</option>
            <option value="ch">Suisse</option>
            <option value="ca">Canada</option>
        </select>

        <label for="bio">Biographie :</label>
        <textarea id="bio" name="bio" rows="4" cols="50"
                  placeholder="Parlez-nous de vous..."></textarea>

        <label for="avatar">Photo de profil :</label>
        <input type="file" id="avatar" name="avatar"
               accept="image/png, image/jpeg">
    </fieldset>

    <button type="submit">S'inscrire</button>
    <button type="reset">Reinitialiser</button>
</form>
```

### Tous les types d'input courants

| Type | Affichage | Usage |
|------|-----------|-------|
| `text` | Champ texte simple | Nom, prenom, etc. |
| `email` | Champ avec validation email | Adresse email |
| `password` | Caracteres masques | Mots de passe |
| `number` | Champ numerique avec fleches | Quantites, ages |
| `date` | Selecteur de date | Dates |
| `time` | Selecteur d'heure | Heures |
| `datetime-local` | Date + heure | Rendez-vous |
| `tel` | Clavier telephone sur mobile | Numeros |
| `url` | Validation URL | Adresses web |
| `search` | Champ de recherche | Barres de recherche |
| `range` | Curseur (slider) | Valeurs dans un intervalle |
| `color` | Selecteur de couleur | Choix de couleur |
| `checkbox` | Case a cocher | Choix multiples |
| `radio` | Bouton radio | Choix unique |
| `file` | Selecteur de fichier | Upload |
| `hidden` | Invisible | Donnees techniques |
| `submit` | Bouton d'envoi | Soumettre le formulaire |

> [!tip] Attributs de validation
> HTML5 offre une validation cote client native :
> - `required` : champ obligatoire
> - `minlength` / `maxlength` : longueur min/max du texte
> - `min` / `max` : valeur min/max pour les nombres
> - `pattern` : expression reguliere a respecter
> - `placeholder` : texte d'indication (pas un remplacement du label !)

> [!warning] `<label>` est obligatoire
> Chaque champ de formulaire doit avoir un `<label>` associe via l'attribut `for` (qui correspond a l'`id` du champ). C'est essentiel pour l'accessibilite et ameliore l'ergonomie (cliquer sur le label active le champ).

---

## HTML semantique (HTML5)

Le HTML semantique utilise des balises qui **decrivent le sens** du contenu, pas juste son apparence :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Site avec HTML semantique</title>
</head>
<body>

    <header>
        <nav>
            <ul>
                <li><a href="#accueil">Accueil</a></li>
                <li><a href="#a-propos">A propos</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <section id="accueil">
            <h1>Bienvenue sur mon site</h1>
            <p>Introduction du site...</p>
        </section>

        <section id="articles">
            <h2>Derniers articles</h2>

            <article>
                <header>
                    <h3>Titre de l'article</h3>
                    <time datetime="2024-03-15">15 mars 2024</time>
                </header>
                <p>Contenu de l'article...</p>
                <footer>
                    <p>Auteur : Jean Dupont</p>
                </footer>
            </article>

            <article>
                <header>
                    <h3>Un autre article</h3>
                    <time datetime="2024-03-10">10 mars 2024</time>
                </header>
                <p>Contenu...</p>
            </article>
        </section>

        <aside>
            <h2>A propos de l'auteur</h2>
            <p>Developpeur web passionne...</p>
        </aside>
    </main>

    <footer>
        <p>&copy; 2024 Mon Site. Tous droits reserves.</p>
        <nav>
            <a href="/mentions-legales">Mentions legales</a>
            <a href="/confidentialite">Politique de confidentialite</a>
        </nav>
    </footer>

</body>
</html>
```

### Pourquoi le HTML semantique est important

```
Structure NON semantique          Structure semantique
(a eviter)                        (recommandee)

<div class="header">              <header>
  <div class="nav">                 <nav>
    ...                               ...
  </div>                            </nav>
</div>                            </header>
<div class="main">                <main>
  <div class="section">             <section>
    <div class="article">             <article>
      ...                                ...
    </div>                            </article>
  </div>                            </section>
  <div class="sidebar">             <aside>
    ...                                ...
  </div>                            </aside>
</div>                            </main>
<div class="footer">              <footer>
  ...                               ...
</div>                            </footer>
```

> [!info] Les 3 raisons d'utiliser le HTML semantique
> 1. **Accessibilite** : Les lecteurs d'ecran utilisent les balises semantiques pour naviguer. Un utilisateur aveugle peut "sauter" directement au `<main>` ou parcourir les `<nav>`.
> 2. **SEO** : Les moteurs de recherche comprennent mieux le contenu et sa hierarchie.
> 3. **Maintenabilite** : Le code est plus lisible pour les developpeurs. `<header>` est immediatement comprehensible, `<div class="hdr-wrap">` beaucoup moins.

### Resume des balises semantiques

| Balise | Role |
|--------|------|
| `<header>` | En-tete de page ou de section |
| `<nav>` | Navigation principale ou secondaire |
| `<main>` | Contenu principal (un seul par page) |
| `<section>` | Section thematique du contenu |
| `<article>` | Contenu autonome et redistribuable |
| `<aside>` | Contenu complementaire (sidebar) |
| `<footer>` | Pied de page ou de section |
| `<figure>` | Illustration avec legende |
| `<figcaption>` | Legende d'une figure |
| `<time>` | Date ou heure |
| `<mark>` | Texte surligne (pertinence) |
| `<details>` | Contenu repliable |
| `<summary>` | Titre d'un `<details>` |

---

## Multimedia

### Audio

```html
<audio controls>
    <source src="musique.mp3" type="audio/mpeg">
    <source src="musique.ogg" type="audio/ogg">
    Votre navigateur ne supporte pas l'element audio.
</audio>

<!-- Avec attributs supplementaires -->
<audio controls autoplay loop muted preload="auto">
    <source src="ambient.mp3" type="audio/mpeg">
</audio>
```

### Video

```html
<video controls width="640" height="360" poster="miniature.jpg">
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
    <track src="sous-titres-fr.vtt" kind="subtitles"
           srclang="fr" label="Francais" default>
    Votre navigateur ne supporte pas l'element video.
</video>
```

> [!tip] Formats recommandes
> - **Video** : MP4 (H.264) pour la compatibilite universelle, WebM pour de meilleures performances
> - **Audio** : MP3 pour la compatibilite, OGG comme alternative libre

---

## Iframes

Les iframes permettent d'integrer du contenu externe dans votre page :

```html
<!-- Integrer une video YouTube -->
<iframe width="560" height="315"
        src="https://www.youtube.com/embed/VIDEO_ID"
        title="Titre de la video"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope"
        allowfullscreen>
</iframe>

<!-- Integrer une carte Google Maps -->
<iframe src="https://www.google.com/maps/embed?pb=..."
        width="600" height="450"
        style="border:0;"
        allowfullscreen=""
        loading="lazy"
        title="Localisation sur Google Maps">
</iframe>
```

> [!warning] Securite des iframes
> Les iframes peuvent representer un risque de securite. Utilisez l'attribut `sandbox` pour restreindre les capacites du contenu embarque :
> ```html
> <iframe src="..." sandbox="allow-scripts allow-same-origin"></iframe>
> ```

---

## Entites HTML

Les entites permettent d'afficher des caracteres speciaux reserves par HTML :

| Entite | Caractere | Description |
|--------|-----------|-------------|
| `&lt;` | < | Inferieur a |
| `&gt;` | > | Superieur a |
| `&amp;` | & | Esperluette |
| `&quot;` | " | Guillemet double |
| `&apos;` | ' | Apostrophe |
| `&nbsp;` | (espace) | Espace insecable |
| `&copy;` | (c) | Copyright |
| `&euro;` | EUR | Euro |
| `&rarr;` | -> | Fleche droite |
| `&hearts;` | coeur | Coeur |

```html
<p>Le prix est de 29,99 &euro;</p>
<p>&copy; 2024 Mon entreprise</p>
<p>Utilisez &lt;p&gt; pour un paragraphe</p>
```

---

## Validation du code HTML

### Le validateur W3C

Le W3C (World Wide Web Consortium) fournit un validateur gratuit : [validator.w3.org](https://validator.w3.org)

> [!example] Erreurs courantes detectees par le validateur
> - Balises non fermees (`<p>` sans `</p>`)
> - Attribut `alt` manquant sur les images
> - Elements imbriques incorrectement (`<p><div>...</div></p>`)
> - `id` en doublon sur la meme page
> - Elements obsoletes (`<center>`, `<font>`)

### Bonnes pratiques generales

1. **Toujours fermer les balises** (meme si le navigateur tolere l'oubli)
2. **Indenter correctement** le code (2 ou 4 espaces par niveau)
3. **Utiliser des noms semantiques** pour les `class` et `id`
4. **Ecrire les balises en minuscules** (`<div>`, pas `<DIV>`)
5. **Toujours mettre les valeurs d'attribut entre guillemets**
6. **Valider regulierement** avec le validateur W3C

> [!example] Code HTML bien formate
> ```html
> <!-- BON -->
> <section class="hero-section">
>     <h1>Titre principal</h1>
>     <p>Un paragraphe descriptif avec du <strong>texte important</strong>.</p>
>     <a href="/contact" class="btn-primary">Contactez-nous</a>
> </section>
>
> <!-- MAUVAIS -->
> <DIV CLASS=hero>
> <H1>Titre principal
> <p>Un paragraphe descriptif avec du <b>texte important</b>.
> <a href=/contact class=btn>Contactez-nous
> </DIV>
> ```

---

## Carte Mentale

```
                            HTML FONDAMENTAUX
                                  │
            ┌─────────────┬───────┼────────┬──────────────┐
            │             │       │        │              │
        Structure      Texte   Liens &   Formulaires   Semantique
            │             │    Medias      │              │
      ┌─────┼─────┐     │       │        │         ┌────┼────┐
      │     │     │     │       │        │         │    │    │
  DOCTYPE <head> <body> │    ┌──┼──┐   <form>   <header> │ <footer>
      │     │     │     │    │  │  │     │       <nav> <main>
    html5  meta  contenu │  <a> │ <table> │      <section>
           title        │  <img>│        │      <article>
           charset      │ <video>│    ┌──┼──┐   <aside>
           viewport     │ <audio>│    │  │  │
                        │ <iframe>    │  │  │
                   ┌────┼──┐     input radio
                   │    │  │     select checkbox
                  <h1> <p> │     textarea file
                  ... <ul> │     fieldset
                  <h6> <ol>│     label
                       <dl>│     required
                      <strong>   pattern
                       <em>
                       <code>
```

---

## Exercices

### Exercice 1 : Page personnelle basique
Creez une page HTML complete avec :
- Un titre `<h1>` avec votre nom
- Un paragraphe de presentation
- Une liste de vos 5 competences
- Un lien vers votre profil LinkedIn (ou un site de votre choix)
- Une image (peut etre un placeholder comme `https://via.placeholder.com/300`)

### Exercice 2 : Formulaire de contact
Creez un formulaire de contact complet avec :
- Champs nom, email, telephone
- Un select pour le sujet du message
- Un textarea pour le message
- Des boutons radio pour la priorite (faible, moyenne, haute)
- Validation HTML5 (required, pattern pour le telephone, type email)
- Utilisez `<fieldset>` et `<legend>` pour organiser

### Exercice 3 : Page semantique d'article de blog
Creez une page avec la structure semantique complete :
- `<header>` avec navigation
- `<main>` contenant un `<article>` de blog avec titre, date (`<time>`), contenu, et auteur
- `<aside>` avec des liens vers d'autres articles
- `<footer>` avec copyright et liens legaux
- Un tableau comparatif dans l'article
- Au moins une `<figure>` avec `<figcaption>`

### Exercice 4 : Tableau de donnees complexe
Creez un tableau d'emploi du temps hebdomadaire :
- Utilisez `<thead>`, `<tbody>`, `<tfoot>`
- Fusionnez des cellules avec `colspan` et `rowspan` (ex: un cours qui dure 2 heures)
- Ajoutez un `<caption>`
- Rendez-le accessible avec les attributs `scope`

---

## Liens

- [[02 - CSS Fondamentaux]] - Apprendre a styler vos pages HTML
- [[02 - JavaScript DOM et Evenements]] - Manipuler le HTML avec JavaScript
