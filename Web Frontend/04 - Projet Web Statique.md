# Projet Web Statique : Portfolio Personnel

Ce cours est un projet pratique de A a Z. Vous allez construire un **site portfolio complet** en utilisant uniquement HTML et CSS, en appliquant tout ce que vous avez appris dans les cours precedents : HTML semantique, Flexbox, Grid, responsive design, transitions, variables CSS et bonnes pratiques.

Le portfolio est le projet ideal pour consolider vos competences : il couvre tous les types de composants courants (navigation, hero, grilles, formulaires, footer) et vous servira concretement pour presenter vos projets futurs.

> [!tip] Analogie
> Construire un portfolio, c'est comme cuisiner un repas complet pour la premiere fois. Vous avez appris a couper les legumes (HTML), a maitriser les cuissons (CSS de base) et les techniques avancees (CSS avance). Maintenant, vous allez combiner tout cela pour preparer un repas de A a Z : entree, plat, dessert, presentation dans l'assiette, et service au convive.

---

## Structure du projet

```
portfolio/
├── index.html
├── css/
│   └── style.css
├── assets/
│   ├── images/
│   │   ├── hero-bg.jpg
│   │   ├── profile.jpg
│   │   ├── project-1.jpg
│   │   ├── project-2.jpg
│   │   └── project-3.jpg
│   └── icons/
│       └── favicon.ico
└── README.md (optionnel)
```

> [!info] Organisation
> Separez toujours votre CSS dans un dossier dedie. Les images vont dans `assets/images/`. Cette structure est standard et sera comprise par n'importe quel developpeur.

---

## Wireframe : plan du site

Avant d'ecrire une seule ligne de code, planifions la structure visuelle :

```
┌──────────────────────────────────────────────────────────┐
│  HEADER / NAVIGATION                                     │
│  [Logo]              Accueil  A propos  Projets  Contact │
├──────────────────────────────────────────────────────────┤
│                                                          │
│                    HERO SECTION                           │
│               Bonjour, je suis [Nom]                     │
│            Developpeur Web Full Stack                    │
│              [Bouton : Voir mes projets]                 │
│                                                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│                  A PROPOS DE MOI                          │
│   ┌──────────┐  Texte de presentation...                 │
│   │  Photo   │  Passions, parcours, objectifs.           │
│   │ profil   │  Technologies favorites...                │
│   └──────────┘                                           │
│                                                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│                    COMPETENCES                            │
│   ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐               │
│   │ HTML │  │ CSS  │  │  JS  │  │ React│               │
│   │ 90%  │  │ 85%  │  │ 75%  │  │ 60%  │               │
│   └──────┘  └──────┘  └──────┘  └──────┘               │
│                                                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│                      PROJETS                             │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│   │ Projet 1 │  │ Projet 2 │  │ Projet 3 │             │
│   │  image   │  │  image   │  │  image   │             │
│   │  titre   │  │  titre   │  │  titre   │             │
│   │  desc    │  │  desc    │  │  desc    │             │
│   │ [Voir]   │  │ [Voir]   │  │ [Voir]   │             │
│   └──────────┘  └──────────┘  └──────────┘             │
│                                                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│                     CONTACT                              │
│           Nom: [____________]                            │
│          Email: [____________]                            │
│        Message: [____________]                            │
│                 [Envoyer]                                │
│                                                          │
├──────────────────────────────────────────────────────────┤
│  FOOTER                                                  │
│  (c) 2024 | GitHub | LinkedIn | Twitter                  │
└──────────────────────────────────────────────────────────┘
```

---

## Etape 1 : Le HTML complet

Commencons par le squelette HTML semantique complet :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Portfolio de [Votre Nom] - Developpeur Web Full Stack">
    <title>Portfolio - [Votre Nom]</title>
    <link rel="icon" href="assets/icons/favicon.ico" type="image/x-icon">
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
          rel="stylesheet">
    <link rel="stylesheet" href="css/style.css">
</head>
<body>

    <!-- ==================== HEADER ==================== -->
    <header class="header" id="header">
        <nav class="nav container">
            <a href="#" class="nav__logo">Portfolio</a>

            <!-- Checkbox hack pour menu mobile -->
            <input type="checkbox" id="nav-toggle" class="nav__toggle-input">
            <label for="nav-toggle" class="nav__toggle" aria-label="Menu">
                <span class="nav__hamburger"></span>
            </label>

            <ul class="nav__list">
                <li><a href="#accueil" class="nav__link">Accueil</a></li>
                <li><a href="#a-propos" class="nav__link">A propos</a></li>
                <li><a href="#competences" class="nav__link">Competences</a></li>
                <li><a href="#projets" class="nav__link">Projets</a></li>
                <li><a href="#contact" class="nav__link nav__link--cta">Contact</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <!-- ==================== HERO ==================== -->
        <section class="hero" id="accueil">
            <div class="hero__content container">
                <p class="hero__greeting">Bonjour, je suis</p>
                <h1 class="hero__name">Votre Nom</h1>
                <p class="hero__role">Developpeur Web Full Stack</p>
                <p class="hero__description">
                    Je cree des experiences web modernes, performantes
                    et accessibles.
                </p>
                <div class="hero__actions">
                    <a href="#projets" class="btn btn--primary">Voir mes projets</a>
                    <a href="#contact" class="btn btn--outline">Me contacter</a>
                </div>
            </div>
        </section>

        <!-- ==================== A PROPOS ==================== -->
        <section class="about section" id="a-propos">
            <div class="container">
                <h2 class="section__title">A propos de moi</h2>
                <div class="about__grid">
                    <div class="about__image-wrapper">
                        <img src="assets/images/profile.jpg"
                             alt="Photo de [Votre Nom]"
                             class="about__image">
                    </div>
                    <div class="about__text">
                        <p>
                            Passionne par le developpement web depuis [X] ans,
                            je me specialise dans la creation d'interfaces
                            utilisateur intuitives et performantes.
                        </p>
                        <p>
                            Mon parcours m'a amene a travailler avec des
                            technologies variees, du frontend au backend,
                            en passant par le design UI/UX.
                        </p>
                        <p>
                            Quand je ne code pas, vous me trouverez
                            [vos hobbies].
                        </p>
                        <a href="#contact" class="btn btn--primary">
                            Travaillons ensemble
                        </a>
                    </div>
                </div>
            </div>
        </section>

        <!-- ==================== COMPETENCES ==================== -->
        <section class="skills section" id="competences">
            <div class="container">
                <h2 class="section__title">Competences</h2>
                <div class="skills__grid">

                    <div class="skill-card">
                        <div class="skill-card__icon">&#60;/&#62;</div>
                        <h3 class="skill-card__title">HTML5</h3>
                        <p class="skill-card__desc">Structure semantique et accessible</p>
                        <div class="skill-card__bar">
                            <div class="skill-card__progress" style="width: 90%"></div>
                        </div>
                        <span class="skill-card__level">90%</span>
                    </div>

                    <div class="skill-card">
                        <div class="skill-card__icon">{ }</div>
                        <h3 class="skill-card__title">CSS3</h3>
                        <p class="skill-card__desc">Flexbox, Grid, animations, responsive</p>
                        <div class="skill-card__bar">
                            <div class="skill-card__progress" style="width: 85%"></div>
                        </div>
                        <span class="skill-card__level">85%</span>
                    </div>

                    <div class="skill-card">
                        <div class="skill-card__icon">JS</div>
                        <h3 class="skill-card__title">JavaScript</h3>
                        <p class="skill-card__desc">ES6+, DOM, API, async</p>
                        <div class="skill-card__bar">
                            <div class="skill-card__progress" style="width: 75%"></div>
                        </div>
                        <span class="skill-card__level">75%</span>
                    </div>

                    <div class="skill-card">
                        <div class="skill-card__icon">R</div>
                        <h3 class="skill-card__title">React</h3>
                        <p class="skill-card__desc">Composants, hooks, state management</p>
                        <div class="skill-card__bar">
                            <div class="skill-card__progress" style="width: 65%"></div>
                        </div>
                        <span class="skill-card__level">65%</span>
                    </div>

                    <div class="skill-card">
                        <div class="skill-card__icon">Git</div>
                        <h3 class="skill-card__title">Git & GitHub</h3>
                        <p class="skill-card__desc">Versioning, branches, collaboration</p>
                        <div class="skill-card__bar">
                            <div class="skill-card__progress" style="width: 80%"></div>
                        </div>
                        <span class="skill-card__level">80%</span>
                    </div>

                    <div class="skill-card">
                        <div class="skill-card__icon">UI</div>
                        <h3 class="skill-card__title">UI/UX Design</h3>
                        <p class="skill-card__desc">Figma, prototypage, accessibilite</p>
                        <div class="skill-card__bar">
                            <div class="skill-card__progress" style="width: 60%"></div>
                        </div>
                        <span class="skill-card__level">60%</span>
                    </div>

                </div>
            </div>
        </section>

        <!-- ==================== PROJETS ==================== -->
        <section class="projects section" id="projets">
            <div class="container">
                <h2 class="section__title">Mes projets</h2>
                <div class="projects__grid">

                    <article class="project-card">
                        <div class="project-card__image-wrapper">
                            <img src="assets/images/project-1.jpg"
                                 alt="Capture d'ecran du projet E-commerce"
                                 class="project-card__image">
                            <div class="project-card__overlay">
                                <a href="#" class="btn btn--small">Voir le site</a>
                                <a href="#" class="btn btn--small btn--outline">Code source</a>
                            </div>
                        </div>
                        <div class="project-card__content">
                            <h3 class="project-card__title">Site E-commerce</h3>
                            <p class="project-card__desc">
                                Boutique en ligne responsive avec panier et paiement.
                            </p>
                            <div class="project-card__tags">
                                <span class="tag">HTML</span>
                                <span class="tag">CSS</span>
                                <span class="tag">JavaScript</span>
                            </div>
                        </div>
                    </article>

                    <article class="project-card">
                        <div class="project-card__image-wrapper">
                            <img src="assets/images/project-2.jpg"
                                 alt="Capture d'ecran de l'application meteo"
                                 class="project-card__image">
                            <div class="project-card__overlay">
                                <a href="#" class="btn btn--small">Voir le site</a>
                                <a href="#" class="btn btn--small btn--outline">Code source</a>
                            </div>
                        </div>
                        <div class="project-card__content">
                            <h3 class="project-card__title">Application Meteo</h3>
                            <p class="project-card__desc">
                                App meteo en temps reel avec API et geolocalisation.
                            </p>
                            <div class="project-card__tags">
                                <span class="tag">React</span>
                                <span class="tag">API</span>
                                <span class="tag">CSS</span>
                            </div>
                        </div>
                    </article>

                    <article class="project-card">
                        <div class="project-card__image-wrapper">
                            <img src="assets/images/project-3.jpg"
                                 alt="Capture d'ecran du tableau de bord"
                                 class="project-card__image">
                            <div class="project-card__overlay">
                                <a href="#" class="btn btn--small">Voir le site</a>
                                <a href="#" class="btn btn--small btn--outline">Code source</a>
                            </div>
                        </div>
                        <div class="project-card__content">
                            <h3 class="project-card__title">Dashboard Analytics</h3>
                            <p class="project-card__desc">
                                Tableau de bord avec graphiques et donnees en temps reel.
                            </p>
                            <div class="project-card__tags">
                                <span class="tag">JavaScript</span>
                                <span class="tag">Chart.js</span>
                                <span class="tag">Node.js</span>
                            </div>
                        </div>
                    </article>

                </div>
            </div>
        </section>

        <!-- ==================== CONTACT ==================== -->
        <section class="contact section" id="contact">
            <div class="container">
                <h2 class="section__title">Me contacter</h2>
                <p class="section__subtitle">
                    Un projet en tete ? N'hesitez pas a me contacter.
                </p>

                <form class="contact__form" action="#" method="POST">
                    <div class="form__group">
                        <label for="name" class="form__label">Nom complet</label>
                        <input type="text" id="name" name="name"
                               class="form__input" required
                               placeholder="Jean Dupont">
                    </div>

                    <div class="form__group">
                        <label for="email" class="form__label">Email</label>
                        <input type="email" id="email" name="email"
                               class="form__input" required
                               placeholder="jean@example.com">
                    </div>

                    <div class="form__group">
                        <label for="subject" class="form__label">Sujet</label>
                        <select id="subject" name="subject" class="form__input" required>
                            <option value="">-- Choisir un sujet --</option>
                            <option value="projet">Proposition de projet</option>
                            <option value="emploi">Opportunite d'emploi</option>
                            <option value="question">Question generale</option>
                            <option value="autre">Autre</option>
                        </select>
                    </div>

                    <div class="form__group form__group--full">
                        <label for="message" class="form__label">Message</label>
                        <textarea id="message" name="message"
                                  class="form__input form__textarea" required
                                  rows="6"
                                  placeholder="Decrivez votre projet ou votre question..."></textarea>
                    </div>

                    <div class="form__group form__group--full">
                        <button type="submit" class="btn btn--primary btn--full">
                            Envoyer le message
                        </button>
                    </div>
                </form>
            </div>
        </section>
    </main>

    <!-- ==================== FOOTER ==================== -->
    <footer class="footer">
        <div class="container">
            <div class="footer__grid">
                <div class="footer__brand">
                    <a href="#" class="footer__logo">Portfolio</a>
                    <p class="footer__text">
                        Developpeur web passionne par la creation
                        d'experiences numeriques de qualite.
                    </p>
                </div>
                <div class="footer__links">
                    <h4 class="footer__heading">Navigation</h4>
                    <ul>
                        <li><a href="#accueil">Accueil</a></li>
                        <li><a href="#a-propos">A propos</a></li>
                        <li><a href="#projets">Projets</a></li>
                        <li><a href="#contact">Contact</a></li>
                    </ul>
                </div>
                <div class="footer__social">
                    <h4 class="footer__heading">Reseaux</h4>
                    <ul>
                        <li><a href="https://github.com/" target="_blank"
                               rel="noopener noreferrer">GitHub</a></li>
                        <li><a href="https://linkedin.com/" target="_blank"
                               rel="noopener noreferrer">LinkedIn</a></li>
                        <li><a href="https://twitter.com/" target="_blank"
                               rel="noopener noreferrer">Twitter</a></li>
                    </ul>
                </div>
            </div>
            <div class="footer__bottom">
                <p>&copy; 2024 [Votre Nom]. Tous droits reserves.</p>
            </div>
        </div>
    </footer>

</body>
</html>
```

---

## Etape 2 : Le CSS complet

### Variables et reset

```css
/* ===== VARIABLES ===== */
:root {
    /* Couleurs */
    --clr-primary: #2563eb;
    --clr-primary-dark: #1d4ed8;
    --clr-primary-light: #dbeafe;
    --clr-secondary: #10b981;
    --clr-bg: #ffffff;
    --clr-bg-alt: #f8fafc;
    --clr-text: #1e293b;
    --clr-text-light: #64748b;
    --clr-border: #e2e8f0;

    /* Typographie */
    --font-main: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    --fs-sm: 0.875rem;
    --fs-base: 1rem;
    --fs-lg: 1.125rem;
    --fs-xl: 1.5rem;
    --fs-2xl: 2rem;
    --fs-3xl: clamp(2rem, 5vw, 3.5rem);

    /* Espacements */
    --sp-xs: 0.25rem;
    --sp-sm: 0.5rem;
    --sp-md: 1rem;
    --sp-lg: 2rem;
    --sp-xl: 4rem;
    --sp-section: 6rem;

    /* Effets */
    --radius: 8px;
    --radius-lg: 16px;
    --shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.1);
    --shadow-lg: 0 10px 30px rgba(0, 0, 0, 0.12);
    --transition: 0.3s ease;

    /* Layout */
    --max-width: 1200px;
    --header-height: 72px;
}

/* ===== RESET ===== */
*, *::before, *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

html {
    scroll-behavior: smooth;
    scroll-padding-top: var(--header-height);
}

body {
    font-family: var(--font-main);
    font-size: var(--fs-base);
    line-height: 1.6;
    color: var(--clr-text);
    background-color: var(--clr-bg);
}

img {
    max-width: 100%;
    height: auto;
    display: block;
}

a {
    color: inherit;
    text-decoration: none;
}

ul { list-style: none; }

/* ===== UTILITAIRES ===== */
.container {
    width: min(var(--max-width), 90%);
    margin: 0 auto;
}

.section {
    padding: var(--sp-section) 0;
}

.section__title {
    font-size: var(--fs-2xl);
    text-align: center;
    margin-bottom: var(--sp-lg);
    position: relative;
}

.section__title::after {
    content: "";
    display: block;
    width: 60px;
    height: 4px;
    background: var(--clr-primary);
    margin: var(--sp-sm) auto 0;
    border-radius: 2px;
}

.section__subtitle {
    text-align: center;
    color: var(--clr-text-light);
    margin-bottom: var(--sp-xl);
    max-width: 600px;
    margin-left: auto;
    margin-right: auto;
}
```

### Header et navigation

```css
/* ===== HEADER ===== */
.header {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: var(--header-height);
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    border-bottom: 1px solid var(--clr-border);
    z-index: 1000;
}

.nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 100%;
}

.nav__logo {
    font-size: var(--fs-xl);
    font-weight: 700;
    color: var(--clr-primary);
}

.nav__list {
    display: flex;
    gap: var(--sp-lg);
}

.nav__link {
    font-weight: 500;
    color: var(--clr-text-light);
    transition: color var(--transition);
    position: relative;
}

.nav__link::after {
    content: "";
    position: absolute;
    bottom: -4px;
    left: 0;
    width: 0;
    height: 2px;
    background: var(--clr-primary);
    transition: width var(--transition);
}

.nav__link:hover {
    color: var(--clr-primary);
}

.nav__link:hover::after {
    width: 100%;
}

.nav__link--cta {
    background: var(--clr-primary);
    color: white;
    padding: var(--sp-sm) var(--sp-md);
    border-radius: var(--radius);
}

.nav__link--cta::after { display: none; }

.nav__link--cta:hover {
    background: var(--clr-primary-dark);
    color: white;
}

/* Hamburger (cache par defaut, visible sur mobile) */
.nav__toggle-input { display: none; }
.nav__toggle { display: none; }
```

### Boutons

```css
/* ===== BOUTONS ===== */
.btn {
    display: inline-block;
    padding: 12px 28px;
    border-radius: var(--radius);
    font-weight: 600;
    font-size: var(--fs-base);
    cursor: pointer;
    transition: all var(--transition);
    border: 2px solid transparent;
    text-align: center;
}

.btn--primary {
    background: var(--clr-primary);
    color: white;
}

.btn--primary:hover {
    background: var(--clr-primary-dark);
    transform: translateY(-2px);
    box-shadow: var(--shadow-md);
}

.btn--outline {
    background: transparent;
    color: var(--clr-primary);
    border-color: var(--clr-primary);
}

.btn--outline:hover {
    background: var(--clr-primary);
    color: white;
    transform: translateY(-2px);
}

.btn--small {
    padding: 8px 16px;
    font-size: var(--fs-sm);
}

.btn--full {
    width: 100%;
}
```

### Hero section

```css
/* ===== HERO ===== */
.hero {
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding-top: var(--header-height);
    background:
        linear-gradient(135deg, var(--clr-primary-light) 0%, var(--clr-bg) 60%);
}

.hero__greeting {
    font-size: var(--fs-lg);
    color: var(--clr-primary);
    font-weight: 500;
    margin-bottom: var(--sp-sm);
}

.hero__name {
    font-size: var(--fs-3xl);
    font-weight: 700;
    line-height: 1.2;
    margin-bottom: var(--sp-sm);
}

.hero__role {
    font-size: var(--fs-xl);
    color: var(--clr-text-light);
    margin-bottom: var(--sp-md);
}

.hero__description {
    max-width: 500px;
    margin: 0 auto var(--sp-lg);
    color: var(--clr-text-light);
}

.hero__actions {
    display: flex;
    gap: var(--sp-md);
    justify-content: center;
    flex-wrap: wrap;
}
```

### Section A Propos

```css
/* ===== ABOUT ===== */
.about__grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: var(--sp-xl);
    align-items: center;
}

.about__image-wrapper {
    max-width: 350px;
    margin: 0 auto;
}

.about__image {
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow-lg);
}

.about__text p {
    margin-bottom: var(--sp-md);
    color: var(--clr-text-light);
}

.about__text .btn {
    margin-top: var(--sp-md);
}

@media (min-width: 768px) {
    .about__grid {
        grid-template-columns: 1fr 1.5fr;
    }
}
```

### Section Competences

```css
/* ===== COMPETENCES ===== */
.skills {
    background: var(--clr-bg-alt);
}

.skills__grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: var(--sp-lg);
}

.skill-card {
    background: var(--clr-bg);
    padding: var(--sp-lg);
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow);
    transition: transform var(--transition), box-shadow var(--transition);
}

.skill-card:hover {
    transform: translateY(-4px);
    box-shadow: var(--shadow-md);
}

.skill-card__icon {
    width: 48px;
    height: 48px;
    background: var(--clr-primary-light);
    color: var(--clr-primary);
    border-radius: var(--radius);
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 700;
    font-size: var(--fs-lg);
    margin-bottom: var(--sp-md);
}

.skill-card__title {
    font-size: var(--fs-lg);
    margin-bottom: var(--sp-xs);
}

.skill-card__desc {
    font-size: var(--fs-sm);
    color: var(--clr-text-light);
    margin-bottom: var(--sp-md);
}

.skill-card__bar {
    height: 8px;
    background: var(--clr-border);
    border-radius: 4px;
    overflow: hidden;
    margin-bottom: var(--sp-xs);
}

.skill-card__progress {
    height: 100%;
    background: linear-gradient(90deg, var(--clr-primary), var(--clr-secondary));
    border-radius: 4px;
    transition: width 1s ease;
}

.skill-card__level {
    font-size: var(--fs-sm);
    font-weight: 600;
    color: var(--clr-primary);
}
```

### Section Projets

```css
/* ===== PROJETS ===== */
.projects__grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: var(--sp-lg);
}

.project-card {
    border-radius: var(--radius-lg);
    overflow: hidden;
    box-shadow: var(--shadow);
    transition: transform var(--transition), box-shadow var(--transition);
    background: var(--clr-bg);
}

.project-card:hover {
    transform: translateY(-6px);
    box-shadow: var(--shadow-lg);
}

.project-card__image-wrapper {
    position: relative;
    overflow: hidden;
}

.project-card__image {
    width: 100%;
    height: 220px;
    object-fit: cover;
    transition: transform 0.4s ease;
}

.project-card:hover .project-card__image {
    transform: scale(1.05);
}

.project-card__overlay {
    position: absolute;
    inset: 0;
    background: rgba(0, 0, 0, 0.6);
    display: flex;
    align-items: center;
    justify-content: center;
    gap: var(--sp-sm);
    opacity: 0;
    transition: opacity var(--transition);
}

.project-card:hover .project-card__overlay {
    opacity: 1;
}

.project-card__overlay .btn {
    color: white;
    border-color: white;
}

.project-card__overlay .btn:hover {
    background: white;
    color: var(--clr-text);
}

.project-card__content {
    padding: var(--sp-lg);
}

.project-card__title {
    font-size: var(--fs-lg);
    margin-bottom: var(--sp-sm);
}

.project-card__desc {
    color: var(--clr-text-light);
    font-size: var(--fs-sm);
    margin-bottom: var(--sp-md);
}

.project-card__tags {
    display: flex;
    flex-wrap: wrap;
    gap: var(--sp-sm);
}

.tag {
    font-size: 0.75rem;
    padding: 4px 10px;
    background: var(--clr-primary-light);
    color: var(--clr-primary);
    border-radius: 20px;
    font-weight: 500;
}
```

### Section Contact

```css
/* ===== CONTACT ===== */
.contact {
    background: var(--clr-bg-alt);
}

.contact__form {
    max-width: 700px;
    margin: 0 auto;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: var(--sp-md);
}

.form__group--full {
    grid-column: 1 / -1;
}

.form__label {
    display: block;
    font-weight: 500;
    margin-bottom: var(--sp-xs);
    font-size: var(--fs-sm);
}

.form__input {
    width: 100%;
    padding: 12px 16px;
    border: 2px solid var(--clr-border);
    border-radius: var(--radius);
    font-family: var(--font-main);
    font-size: var(--fs-base);
    transition: border-color var(--transition), box-shadow var(--transition);
    background: var(--clr-bg);
}

.form__input:focus {
    outline: none;
    border-color: var(--clr-primary);
    box-shadow: 0 0 0 3px var(--clr-primary-light);
}

.form__input:invalid:not(:placeholder-shown) {
    border-color: #ef4444;
}

.form__textarea {
    resize: vertical;
    min-height: 120px;
}

@media (max-width: 600px) {
    .contact__form {
        grid-template-columns: 1fr;
    }
}
```

### Footer

```css
/* ===== FOOTER ===== */
.footer {
    background: var(--clr-text);
    color: #cbd5e1;
    padding: var(--sp-xl) 0 var(--sp-lg);
}

.footer__grid {
    display: grid;
    grid-template-columns: 2fr 1fr 1fr;
    gap: var(--sp-xl);
    margin-bottom: var(--sp-xl);
}

.footer__logo {
    font-size: var(--fs-xl);
    font-weight: 700;
    color: white;
}

.footer__text {
    margin-top: var(--sp-md);
    font-size: var(--fs-sm);
    line-height: 1.8;
}

.footer__heading {
    color: white;
    font-size: var(--fs-base);
    margin-bottom: var(--sp-md);
}

.footer__links ul li,
.footer__social ul li {
    margin-bottom: var(--sp-sm);
}

.footer__links a,
.footer__social a {
    font-size: var(--fs-sm);
    transition: color var(--transition);
}

.footer__links a:hover,
.footer__social a:hover {
    color: var(--clr-primary);
}

.footer__bottom {
    text-align: center;
    padding-top: var(--sp-lg);
    border-top: 1px solid #334155;
    font-size: var(--fs-sm);
}

@media (max-width: 768px) {
    .footer__grid {
        grid-template-columns: 1fr;
        text-align: center;
    }
}
```

### Responsive : menu mobile

```css
/* ===== RESPONSIVE MOBILE ===== */
@media (max-width: 768px) {
    /* Menu hamburger */
    .nav__toggle {
        display: flex;
        flex-direction: column;
        justify-content: center;
        width: 32px;
        height: 32px;
        cursor: pointer;
        z-index: 1001;
    }

    .nav__hamburger,
    .nav__hamburger::before,
    .nav__hamburger::after {
        display: block;
        width: 24px;
        height: 2px;
        background: var(--clr-text);
        transition: all var(--transition);
        position: relative;
    }

    .nav__hamburger::before,
    .nav__hamburger::after {
        content: "";
        position: absolute;
    }

    .nav__hamburger::before { top: -7px; }
    .nav__hamburger::after { top: 7px; }

    /* Animation hamburger → X */
    .nav__toggle-input:checked ~ .nav__toggle .nav__hamburger {
        background: transparent;
    }
    .nav__toggle-input:checked ~ .nav__toggle .nav__hamburger::before {
        top: 0;
        transform: rotate(45deg);
    }
    .nav__toggle-input:checked ~ .nav__toggle .nav__hamburger::after {
        top: 0;
        transform: rotate(-45deg);
    }

    /* Menu mobile (panneau lateral) */
    .nav__list {
        position: fixed;
        top: 0;
        right: -100%;
        width: 70%;
        max-width: 300px;
        height: 100vh;
        background: var(--clr-bg);
        flex-direction: column;
        padding: calc(var(--header-height) + var(--sp-lg)) var(--sp-lg) var(--sp-lg);
        box-shadow: var(--shadow-lg);
        transition: right var(--transition);
    }

    .nav__toggle-input:checked ~ .nav__list {
        right: 0;
    }
}
```

---

## Etape 3 : Deploiement sur GitHub Pages

> [!info] Prerequis
> Vous avez besoin d'un compte GitHub et de Git installe sur votre machine. Consultez [[01 - Git et GitHub]] pour les bases de Git.

### Instructions pas a pas

1. **Creer un repository sur GitHub**
   - Allez sur [github.com](https://github.com) et cliquez "New repository"
   - Nommez-le `portfolio` (ou `votre-username.github.io` pour un site principal)
   - Laissez-le **public**

2. **Initialiser et pousser votre projet**

```
cd portfolio
git init
git add .
git commit -m "Initial commit: portfolio website"
git branch -M main
git remote add origin https://github.com/VOTRE_USERNAME/portfolio.git
git push -u origin main
```

3. **Activer GitHub Pages**
   - Dans votre repository, allez dans **Settings** → **Pages**
   - Source : **Deploy from a branch**
   - Branch : **main** / **/ (root)**
   - Cliquez **Save**

4. **Votre site est en ligne**
   - Apres quelques minutes : `https://VOTRE_USERNAME.github.io/portfolio/`

> [!warning] Le fichier doit s'appeler `index.html`
> GitHub Pages cherche automatiquement un fichier `index.html` a la racine. Assurez-vous que c'est bien le cas.

---

## Performance et optimisation

### Images

- **Compresser** vos images avec [squoosh.app](https://squoosh.app) ou [tinypng.com](https://tinypng.com)
- **Format WebP** : 30% plus leger que JPEG a qualite equivalente
- **Attribut `loading="lazy"`** : charge les images hors ecran uniquement quand l'utilisateur scrolle
- **Dimensions explicites** : specifiez `width` et `height` pour eviter le layout shift

```html
<img src="photo.webp" alt="Description"
     width="600" height="400"
     loading="lazy">
```

### CSS

- **Minifier** le CSS pour la production (outils : cssnano, clean-css)
- **Une seule feuille de style** en production (evitez les imports multiples)
- **CSS critique** : les styles above-the-fold peuvent etre mis en inline dans le `<head>`

### HTML

- **Minifier** le HTML (outils : html-minifier)
- **Preconnect** aux domaines tiers (Google Fonts, CDN)
- **Charger les scripts en `defer`** pour ne pas bloquer le rendu

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<script src="script.js" defer></script>
```

---

## Checklist d'accessibilite

> [!warning] L'accessibilite n'est pas optionnelle
> Un site accessible est utilisable par tous, y compris les personnes en situation de handicap. C'est aussi une obligation legale dans de nombreux pays.

- [ ] **Chaque image** a un attribut `alt` descriptif (ou `alt=""` si decorative)
- [ ] **Contraste suffisant** : ratio minimum 4.5:1 pour le texte (verifiez avec [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/))
- [ ] **Navigation au clavier** : tous les elements interactifs sont accessibles avec Tab
- [ ] **Focus visible** : ne supprimez jamais `outline` sans le remplacer par un indicateur equivalent
- [ ] **Labels sur tous les champs** de formulaire (pas seulement des placeholders)
- [ ] **Hierarchie des titres** correcte (h1 → h2 → h3, sans sauter de niveaux)
- [ ] **Balises semantiques** utilisees (`<nav>`, `<main>`, `<section>`, `<article>`)
- [ ] **Liens descriptifs** : evitez "cliquez ici", preferez "Voir le projet E-commerce"
- [ ] **Texte redimensionnable** : le site fonctionne si l'utilisateur zoom a 200%
- [ ] **`lang="fr"`** sur la balise `<html>`
- [ ] **`aria-label`** sur les elements interactifs sans texte visible (icones, hamburger)

---

## Carte Mentale

```
                        PROJET PORTFOLIO
                              │
        ┌──────────┬──────────┼──────────┬──────────┐
        │          │          │          │          │
    Structure   Sections    Style     Deploy    Qualite
        │          │          │          │          │
    index.html  Header     Variables  GitHub    Performance
    style.css   Hero       Reset     Pages      │
    assets/     About      BEM       git push  Images
        │       Skills     Flexbox      │      lazy load
    Semantique  Projects   Grid     username   WebP
    <header>    Contact    Responsive .github.io  │
    <main>      Footer     Mobile-     │      Accessibilite
    <section>     │        first    Settings   Contraste
    <footer>   Wireframe   Media Q   Pages     Clavier
                ASCII      sticky   Branch    Semantique
                Plan       backdrop  main     Focus
                           blur              alt text
```

---

## Exercices

### Exercice 1 : Personnaliser le portfolio
Prenez le code de ce cours et personnalisez-le :
- Remplacez les textes par vos vraies informations
- Ajoutez vos propres images (compressees)
- Modifiez la palette de couleurs (changez les variables CSS)
- Ajoutez au moins une section supplementaire (temoignages, timeline parcours, blog)

### Exercice 2 : Dark mode
Ajoutez un toggle dark mode au portfolio :
- Creez un jeu de variables sombres avec `[data-theme="dark"]`
- Ajoutez un bouton toggle dans le header
- Gerez le `prefers-color-scheme` comme defaut
- (Bonus) Sauvegardez la preference dans `localStorage` avec un petit script

### Exercice 3 : Animations au scroll
Ameliorez le portfolio avec des animations d'entree :
- Les sections apparaissent en fondu depuis le bas
- Les skill cards arrivent en cascade (delai croissant)
- Les project cards ont un effet de reveal
- Utilisez des classes CSS + Intersection Observer (JavaScript)

### Exercice 4 : Deployer en ligne
Mettez votre portfolio en ligne :
- Creez un repository GitHub
- Poussez votre code
- Activez GitHub Pages
- Partagez le lien avec un ami et demandez un retour sur mobile

---

## Liens

- [[01 - HTML Fondamentaux]] - Revoir la structure HTML
- [[02 - CSS Fondamentaux]] - Revoir Flexbox, Grid, responsive
- [[01 - Git et GitHub]] - Versionner et deployer avec Git
