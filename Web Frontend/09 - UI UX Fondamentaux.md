# UI/UX Fondamentaux pour Développeurs

Le design d'interface n'est pas une discipline réservée aux designers : tout développeur web crée de l'UI, et la qualité de cette interface détermine directement si les utilisateurs adopent ou abandonnent un produit. Ce cours couvre les fondements du design visuel (UI), de l'expérience utilisateur (UX) et de l'accessibilité, avec des exemples concrets et du code prêt à l'emploi.

> [!info] Prérequis
> Ce cours s'appuie sur les notions vues en **HTML Fondamentaux** et **CSS Fondamentaux**. Une connaissance de base de Flexbox et des sélecteurs CSS est nécessaire. Des notions de responsive design (vues en CSS Avancé) seront utiles pour la partie breakpoints.

---

## 1. Principes de design visuel : CARP

### 1.1 Qu'est-ce que CARP ?

CARP est un acronyme regroupant les quatre principes fondamentaux du design graphique, formalisés par Robin Williams dans *The Non-Designer's Design Book* :

| Lettre | Principe | Définition courte |
|--------|----------|-------------------|
| **C** | Contraste | Rendre les différences visibles et intentionnelles |
| **A** | Alignement | Aligner visuellement chaque élément avec un autre |
| **R** | Répétition | Répéter des éléments visuels pour créer une cohérence |
| **P** | Proximité | Regrouper les éléments liés, séparer les éléments distincts |

Ces principes s'appliquent à **tout** : une page web, une carte, un formulaire, un écran d'application mobile.

---

### 1.2 Contraste

Le contraste crée la hiérarchie visuelle. Il attire l'oeil vers ce qui est important. Le contraste peut être :

- **Typographique** : taille 32px vs 14px, gras vs normal
- **Chromatique** : texte sombre sur fond clair (ou inverse)
- **Spatial** : un élément isolé au milieu d'espace vide
- **De forme** : un cercle parmi des rectangles

```
MAUVAIS — pas de contraste :

  Titre de la page
  Sous-titre de section
  Corps de texte avec du contenu détaillé
  Autre sous-section
  Plus de contenu ici

BON — contraste clair :

  ██████████████████████████████
  █  TITRE PRINCIPAL (32px)    █
  ██████████████████████████████

  SOUS-TITRE (18px, gras)
  ─────────────────────
  Corps de texte normal (16px, poids 400)
  Lorem ipsum dolor sit amet...

  AUTRE SECTION (18px, gras)
  ─────────────────────
  Corps de texte...
```

> [!warning] Contraste ≠ juste les couleurs
> Le contraste de couleur est crucial pour l'accessibilité (voir section 12), mais le principe CARP de contraste est plus large : il concerne la différenciation visuelle globale. Un site entièrement en gris peut avoir du contraste grâce aux tailles et poids.

---

### 1.3 Alignement

Chaque élément doit être aligné visuellement avec un autre élément sur la page. Un alignement invisible crée un axe de lecture mental.

```
MAUVAIS — alignement incohérent :

  Logo
        Menu nav
  Titre de page
        Texte intro qui commence ici
    Bouton CTA

BON — alignement sur axe gauche :

  ┌──────────────────────────────────────────┐
  │ Logo      Menu nav                       │
  ├──────────────────────────────────────────┤
  │ Titre de page                            │
  │ Texte intro qui commence ici             │
  │ [Bouton CTA]                             │
  └──────────────────────────────────────────┘
```

En CSS, l'alignement se traduit par des systèmes de grille cohérents — on reviendra dessus en section 4.

---

### 1.4 Répétition

La répétition crée la cohérence visuelle et aide l'utilisateur à reconnaître les patterns. Si tous les boutons primaires sont bleus et arrondis, l'utilisateur comprend immédiatement qu'un nouvel élément bleu arrondi est cliquable.

Exemples de répétition :
- Même couleur pour tous les liens
- Même style de carte pour tous les articles
- Même icône de flèche pour tous les liens "voir plus"
- Même espacement entre les sections

---

### 1.5 Proximité

Des éléments proches semblent liés. Des éléments éloignés semblent distincts. La proximité organise l'information sans avoir besoin de lignes de séparation ou de bordures.

```
MAUVAIS — proximité ignorée :

  Nom : Jean
  Email : jean@exemple.fr

  Prénom : Dupont
  Téléphone : 06 12 34 56 78

BON — proximité logique :

  ┌─ Coordonnées ─────────────┐
  │  Jean Dupont               │
  │  jean@exemple.fr           │
  │  06 12 34 56 78            │
  └────────────────────────────┘

  ┌─ Adresse ─────────────────┐
  │  12 rue de la Paix         │
  │  75001 Paris               │
  └────────────────────────────┘
```

---

## 2. Typographie

### 2.1 Pourquoi la typographie est cruciale

95% du web est du texte. La typographie est donc la compétence de design la plus directement impactante pour un développeur web. Une mauvaise typographie fatigue les yeux, décourage la lecture et donne une impression d'amateurisme. Une bonne typographie est presque invisible — le lecteur n'y pense pas, il lit.

### 2.2 Les propriétés CSS typographiques essentielles

```css
/* === TAILLE === */
font-size: 16px;        /* taille de base recommandée pour le body */
font-size: 1rem;        /* préférer rem pour la scalabilité */
font-size: clamp(1rem, 2.5vw, 1.5rem); /* taille fluide responsive */

/* === POIDS === */
font-weight: 400;       /* normal / regular */
font-weight: 500;       /* medium */
font-weight: 600;       /* semi-bold */
font-weight: 700;       /* bold */
font-weight: 300;       /* light */

/* === HAUTEUR DE LIGNE (line-height) === */
line-height: 1.5;       /* bon pour le corps de texte (sans unité = ratio) */
line-height: 1.2;       /* adapté pour les titres */
line-height: 1.8;       /* texte long, grande lisibilité */

/* === ESPACEMENT ENTRE LETTRES === */
letter-spacing: 0.02em;  /* légèrement espacé, souvent utilisé pour les titres */
letter-spacing: 0.1em;   /* très espacé — souvent pour les labels ALL CAPS */
letter-spacing: -0.01em; /* légèrement serré — pour les gros titres */

/* === COULEUR ET CONTRASTE === */
color: #1a1a1a;          /* quasi-noir, plus doux que #000000 pur */
color: #6b7280;          /* gris — texte secondaire */
color: #9ca3af;          /* gris clair — texte tertiaire / placeholder */
```

### 2.3 Hiérarchie typographique

Une hiérarchie typographique claire guide le regard de l'utilisateur. On distingue généralement 4-5 niveaux :

```
Niveau 1 — H1 : 48px / font-weight 700 / line-height 1.1
════════════════════════════════════════════

Niveau 2 — H2 : 32px / font-weight 600 / line-height 1.2
─────────────────────────────────────────

  Niveau 3 — H3 : 24px / font-weight 600 / line-height 1.3
  ·····················

    Niveau 4 — H4 : 18px / font-weight 500 / line-height 1.4

    Corps — Body : 16px / font-weight 400 / line-height 1.6
    Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    Pellentesque habitant morbi tristique senectus et netus.

    Texte secondaire : 14px / font-weight 400 / color: #6b7280
    Publié le 12 mars 2024 · 5 min de lecture
```

En CSS :

```css
/* Système typographique complet */
:root {
  --font-size-xs:   0.75rem;   /* 12px */
  --font-size-sm:   0.875rem;  /* 14px */
  --font-size-base: 1rem;      /* 16px */
  --font-size-lg:   1.125rem;  /* 18px */
  --font-size-xl:   1.5rem;    /* 24px */
  --font-size-2xl:  2rem;      /* 32px */
  --font-size-3xl:  3rem;      /* 48px */
}

body {
  font-size: var(--font-size-base);
  line-height: 1.6;
  color: #1a1a1a;
}

h1 { font-size: var(--font-size-3xl); line-height: 1.1; font-weight: 700; }
h2 { font-size: var(--font-size-2xl); line-height: 1.2; font-weight: 600; }
h3 { font-size: var(--font-size-xl);  line-height: 1.3; font-weight: 600; }
h4 { font-size: var(--font-size-lg);  line-height: 1.4; font-weight: 500; }
```

### 2.4 Échelle modulaire

L'échelle modulaire est une progression mathématique de tailles de police basée sur un ratio. Les plus courants :

| Ratio | Nom | Valeurs (base 16px) |
|-------|-----|---------------------|
| 1.067 | Minor Second | 16, 17.1, 18.2, 19.4... |
| 1.125 | Major Second | 16, 18, 20.3, 22.8... |
| 1.250 | Major Third | 16, 20, 25, 31.3... |
| 1.333 | Perfect Fourth | 16, 21.3, 28.4, 37.9... |
| 1.618 | Golden Ratio | 16, 25.9, 41.9, 67.8... |

Le ratio **1.25 (Major Third)** est excellent pour les interfaces web : les niveaux sont distincts sans être trop écartés.

```css
/* Échelle modulaire 1.25 avec CSS custom properties */
:root {
  --scale: 1.25;
  --base: 1rem;

  --step--2: calc(var(--base) / var(--scale) / var(--scale)); /* 0.64rem */
  --step--1: calc(var(--base) / var(--scale));                /* 0.8rem  */
  --step-0:  var(--base);                                     /* 1rem    */
  --step-1:  calc(var(--base) * var(--scale));                /* 1.25rem */
  --step-2:  calc(var(--base) * var(--scale) * var(--scale)); /* 1.563rem */
  --step-3:  calc(var(--step-2) * var(--scale));              /* 1.953rem */
  --step-4:  calc(var(--step-3) * var(--scale));              /* 2.441rem */
  --step-5:  calc(var(--step-4) * var(--scale));              /* 3.052rem */
}
```

### 2.5 Choix de police

> [!tip] Règle des deux polices
> Sur un projet web, limiter à **deux polices maximum** : une pour les titres (serif ou display), une pour le corps (sans-serif). Au-delà, le résultat est visuellement chaotique.

**Combinaisons classiques et éprouvées :**

| Titre | Corps | Style |
|-------|-------|-------|
| Playfair Display | Source Sans 3 | Éditorial, magazine |
| Inter | Inter | Minimaliste, SaaS |
| Fraunces | Inter | Chaleureux, moderne |
| DM Serif Display | DM Sans | Élégant, neutre |
| Space Grotesk | Space Grotesk | Tech, developer |

```html
<!-- Import Google Fonts — placer dans <head> -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Playfair+Display:wght@700&display=swap" rel="stylesheet">
```

```css
:root {
  --font-heading: 'Playfair Display', Georgia, serif;
  --font-body:    'Inter', system-ui, -apple-system, sans-serif;
}

h1, h2, h3 { font-family: var(--font-heading); }
body        { font-family: var(--font-body); }
```

### 2.6 Longueur de ligne (measure)

La longueur de ligne optimale pour la lisibilité est **60-80 caractères** par ligne. En CSS, `max-width: 65ch` sur un conteneur de texte est une bonne règle universelle.

```css
/* Limiter la longueur de ligne pour la lisibilité */
article, .prose {
  max-width: 65ch;         /* ~65 caractères de largeur */
  margin-inline: auto;     /* centrer le conteneur */
}

/* Pour une colonne de contenu full-page plus large */
.content-wrapper {
  max-width: 720px;
  margin-inline: auto;
  padding-inline: 1.5rem;
}
```

---

## 3. Couleurs

### 3.1 La roue chromatique

La roue chromatique organise les couleurs selon leurs relations spectrales :

```
                    JAUNE
                   /     \
          VERT-JAUNE       ORANGE-JAUNE
         /                          \
      VERT                            ORANGE
         \                          /
          VERT-BLEU          ROUGE-ORANGE
                 \           /
           BLEU-VERT       ROUGE
                   \     /
                   BLEU-ROUGE (VIOLET)
                       |
                     VIOLET
```

**Relations de couleurs :**

| Relation | Définition | Exemple | Usage |
|----------|-----------|---------|-------|
| **Complémentaires** | Opposées sur la roue | Bleu + Orange | Fort contraste, accent |
| **Analogues** | Adjacentes sur la roue | Bleu, Bleu-vert, Vert | Harmonie douce |
| **Triadiques** | Équidistantes (120°) | Rouge, Jaune, Bleu | Dynamique, varié |
| **Split-comp.** | 1 couleur + 2 voisines de son complémentaire | Bleu + Jaune-orange + Rouge-orange | Équilibré |
| **Monochromatique** | Variations d'une seule teinte | Bleu 100 → 900 | Cohérent, épuré |

### 3.2 Construire une palette cohérente

Pour une interface professionnelle, on construit une palette à 3 niveaux :

```
PALETTE TYPE :

Primary (couleur de marque)
├── primary-50   : #EFF6FF  (fond très léger)
├── primary-100  : #DBEAFE  (fond léger / hover passif)
├── primary-200  : #BFDBFE
├── primary-300  : #93C5FD
├── primary-400  : #60A5FA
├── primary-500  : #3B82F6  ← valeur principale
├── primary-600  : #2563EB  (hover sur bouton)
├── primary-700  : #1D4ED8  (active / pressed)
├── primary-800  : #1E40AF
└── primary-900  : #1E3A8A  (fond sombre / texte sur clair)

Neutral (gris pour textes et fonds)
├── gray-50  : #F9FAFB  (fond de page)
├── gray-100 : #F3F4F6  (fond de carte)
├── gray-200 : #E5E7EB  (bordure légère)
├── gray-300 : #D1D5DB  (bordure)
├── gray-400 : #9CA3AF  (placeholder / icône inactive)
├── gray-500 : #6B7280  (texte secondaire)
├── gray-600 : #4B5563  (texte body alt)
├── gray-700 : #374151  (texte body)
├── gray-800 : #1F2937  (texte principal)
└── gray-900 : #111827  (titre / quasi-noir)

Semantic (couleurs fonctionnelles)
├── success : #10B981 / #D1FAE5  (vert)
├── warning : #F59E0B / #FEF3C7  (ambre)
├── error   : #EF4444 / #FEE2E2  (rouge)
└── info    : #3B82F6 / #DBEAFE  (bleu)
```

```css
/* Implémentation CSS avec custom properties */
:root {
  /* Primary */
  --color-primary-50:  #EFF6FF;
  --color-primary-500: #3B82F6;
  --color-primary-600: #2563EB;
  --color-primary-700: #1D4ED8;

  /* Neutral */
  --color-gray-50:  #F9FAFB;
  --color-gray-100: #F3F4F6;
  --color-gray-200: #E5E7EB;
  --color-gray-500: #6B7280;
  --color-gray-700: #374151;
  --color-gray-900: #111827;

  /* Semantic */
  --color-success:      #10B981;
  --color-success-bg:   #D1FAE5;
  --color-error:        #EF4444;
  --color-error-bg:     #FEE2E2;
  --color-warning:      #F59E0B;
  --color-warning-bg:   #FEF3C7;
}
```

### 3.3 Accessibilité — ratios de contraste WCAG

Le Web Content Accessibility Guidelines (WCAG) définit des ratios minimaux de contraste entre le texte et son fond.

**Calcul du ratio de contraste :**
Le ratio est calculé entre la luminance relative de deux couleurs : `(L1 + 0.05) / (L2 + 0.05)` où L1 est la luminance la plus claire.

| Niveau | Texte normal | Texte grand (≥18px ou ≥14px gras) |
|--------|-------------|----------------------------------|
| **AA** (minimum) | 4.5:1 | 3:1 |
| **AAA** (optimal) | 7:1 | 4.5:1 |

```
EXEMPLES DE RATIOS :

Texte #FFFFFF sur fond #3B82F6 (bleu 500) → ratio 3.0:1 ❌ (échoue AA texte normal)
Texte #FFFFFF sur fond #1D4ED8 (bleu 700) → ratio 5.9:1 ✅ (passe AA)
Texte #FFFFFF sur fond #1E3A8A (bleu 900) → ratio 10.7:1 ✅✅ (passe AAA)
Texte #111827 sur fond #F9FAFB             → ratio 16.4:1 ✅✅ (passe AAA)
Texte #6B7280 sur fond #FFFFFF             → ratio 4.6:1 ✅ (passe AA, juste)
```

> [!warning] Ne pas tester à l'oeil
> L'oeil humain est très mauvais juge du contraste de couleur. Toujours vérifier avec un outil :
> - **WebAIM Contrast Checker** : webaim.org/resources/contrastchecker/
> - **Colour Contrast Analyser** : application desktop
> - **Extension browser** : axe DevTools (voir section 14)

### 3.4 Utilisation des couleurs — règle 60-30-10

Une palette harmonieuse respecte approximativement la règle 60-30-10 :

```
60% — Couleur dominante (neutre : blanc, gris clair, fond)
  ┌─────────────────────────────────────────────┐
  │         FOND PRINCIPAL (gris-50)            │
  │                                             │
  │  30% — Couleur secondaire (gris moyen,     │
  │         teinte principale désaturée)        │
  │  ┌───────────────────────────────────────┐  │
  │  │      SIDEBAR / HEADER (gris-100)      │  │
  │  │                                       │  │
  │  │  10% — Couleur accent (primary-500)  │  │
  │  │  ┌─────────────────────────────────┐  │  │
  │  │  │  [BOUTON BLEU]  [LIEN BLEU]    │  │  │
  │  │  └─────────────────────────────────┘  │  │
  │  └───────────────────────────────────────┘  │
  └─────────────────────────────────────────────┘
```

---

## 4. Espacement et grilles

### 4.1 Le système de 8px (base-8)

Toutes les valeurs d'espacement doivent être des multiples de 8px (ou de 4px pour les petits ajustements). Cette règle crée une cohérence visuelle automatique et simplifie la prise de décision.

```
Échelle d'espacement base-8 :

4px  — Espacement minimum (entre icône et label)
8px  — XS (padding interne petit composant)
12px — SM (espace entre éléments proches dans un composant)
16px — MD (padding standard, espace entre composants)
24px — LG (gap dans une liste, espace entre sections proches)
32px — XL (marge entre blocs logiques)
48px — 2XL (espace entre sections majeures)
64px — 3XL (espace de respiration entre grandes sections)
96px — 4XL (section hero, espace avant/après header)
128px — 5XL (très grand espace sur desktop)
```

```css
:root {
  --space-1:  4px;
  --space-2:  8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;
  --space-16: 64px;
  --space-24: 96px;
  --space-32: 128px;
}

/* Utilisation cohérente */
.card {
  padding: var(--space-6);          /* 24px intérieur */
  border-radius: var(--space-2);    /* 8px coins */
  gap: var(--space-4);              /* 16px entre éléments internes */
  margin-bottom: var(--space-8);    /* 32px entre cartes */
}
```

### 4.2 Grilles de colonnes

Le système de grille organise les éléments horizontalement. Le standard web est la grille à **12 colonnes** (divisible par 2, 3, 4, 6).

```
Grille 12 colonnes — desktop 1200px :

│ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │10 │11 │12 │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

Layouts courants :

Plein largeur (12/12) :
[════════════════════════════════════════════════]

Sidebar + contenu (3/12 + 9/12) :
[MENU]  [CONTENU PRINCIPAL                      ]

Deux colonnes égales (6/12 + 6/12) :
[GAUCHE              ]  [DROITE                 ]

Trois colonnes (4/12 × 3) :
[CARTE 1      ]  [CARTE 2      ]  [CARTE 3      ]

Quatre colonnes (3/12 × 4) :
[ITEM 1][ITEM 2][ITEM 3][ITEM 4]
```

```css
/* Grille CSS Grid à 12 colonnes */
.grid-container {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: var(--space-6);  /* 24px entre colonnes */
  max-width: 1200px;
  margin-inline: auto;
  padding-inline: var(--space-6);
}

/* Exemples d'utilisation */
.full-width     { grid-column: span 12; }
.two-thirds     { grid-column: span 8; }
.one-third      { grid-column: span 4; }
.half           { grid-column: span 6; }
.sidebar        { grid-column: span 3; }
.main-content   { grid-column: span 9; }

/* Responsive */
@media (max-width: 768px) {
  .two-thirds,
  .one-third,
  .sidebar,
  .main-content { grid-column: span 12; }

  .half { grid-column: span 12; }
}
```

### 4.3 Padding cohérent dans les composants

```css
/* Tailles de composants cohérentes */
.btn-sm   { padding: var(--space-1) var(--space-3); font-size: 0.875rem; } /* 4px 12px */
.btn-md   { padding: var(--space-2) var(--space-4); font-size: 1rem; }     /* 8px 16px */
.btn-lg   { padding: var(--space-3) var(--space-6); font-size: 1.125rem; } /* 12px 24px */

.input-sm { padding: var(--space-1) var(--space-3); min-height: 32px; }
.input-md { padding: var(--space-2) var(--space-4); min-height: 40px; }
.input-lg { padding: var(--space-3) var(--space-4); min-height: 48px; }

/* Carte standard */
.card {
  padding: var(--space-6);       /* 24px */
  border-radius: var(--space-2); /* 8px  */
  border: 1px solid var(--color-gray-200);
  background: white;
}
```

---

## 5. Composants UI — États et interactions

### 5.1 Les états d'un composant interactif

Chaque composant interactif (bouton, champ, lien) doit avoir **tous** ses états définis. Ignorer un état crée des expériences incomplètes ou des inaccessibilités.

```
États d'un bouton :

[Default ]  → État au repos, pas d'interaction
[Hover   ]  → Souris au-dessus (cursor: pointer)
[Focus   ]  → Focus clavier (Tab) — OBLIGATOIRE pour l'accessibilité
[Active  ]  → Clic en cours (mousedown)
[Loading ]  → Traitement en cours (spinner)
[Disabled]  → Non disponible (cursor: not-allowed)
[Success ]  → Action réussie (feedback temporaire)
[Error   ]  → Action échouée
```

```css
/* Bouton complet avec tous ses états */
.btn-primary {
  /* Base */
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  padding: var(--space-2) var(--space-4);
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.15s ease;

  /* Style de base */
  background-color: var(--color-primary-500);
  color: white;

  /* IMPORTANT : outline visible pour le focus clavier */
  outline: 2px solid transparent;
  outline-offset: 2px;
}

/* Hover */
.btn-primary:hover {
  background-color: var(--color-primary-600);
  transform: translateY(-1px);
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}

/* Focus clavier — NE JAMAIS supprimer outline sans alternative */
.btn-primary:focus-visible {
  outline-color: var(--color-primary-500);
  outline-offset: 3px;
}

/* Active (clic) */
.btn-primary:active {
  background-color: var(--color-primary-700);
  transform: translateY(0);
  box-shadow: none;
}

/* Disabled */
.btn-primary:disabled,
.btn-primary[aria-disabled="true"] {
  background-color: var(--color-gray-200);
  color: var(--color-gray-400);
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}
```

> [!warning] Ne jamais supprimer :focus sans remplacement
> `* { outline: none; }` est un code qu'on trouve partout — il est catastrophique pour l'accessibilité. Les utilisateurs naviguant au clavier n'ont plus de repère visuel. Utiliser `:focus-visible` à la place, qui n'affecte pas la navigation à la souris.

### 5.2 Formulaires — conception complète

Un formulaire bien conçu réduit drastiquement les erreurs de saisie.

```html
<!-- Formulaire accessible et bien structuré -->
<form class="form" novalidate>
  <!-- Champ texte -->
  <div class="form-group">
    <label class="form-label" for="email">
      Adresse email
      <span class="required-indicator" aria-hidden="true">*</span>
    </label>
    <input
      class="form-input"
      type="email"
      id="email"
      name="email"
      placeholder="jean@exemple.fr"
      autocomplete="email"
      required
      aria-describedby="email-hint email-error"
    >
    <p class="form-hint" id="email-hint">
      Nous ne partagerons jamais votre email.
    </p>
    <p class="form-error" id="email-error" role="alert" aria-live="polite">
      <!-- Message d'erreur affiché dynamiquement -->
    </p>
  </div>

  <!-- Champ password -->
  <div class="form-group">
    <label class="form-label" for="password">Mot de passe</label>
    <div class="input-wrapper">
      <input
        class="form-input"
        type="password"
        id="password"
        name="password"
        autocomplete="new-password"
        aria-describedby="password-requirements"
      >
      <button
        type="button"
        class="input-toggle"
        aria-label="Afficher/masquer le mot de passe"
        aria-pressed="false"
      >
        <!-- Icône œil -->
      </button>
    </div>
    <ul class="form-hint" id="password-requirements">
      <li>Au moins 8 caractères</li>
      <li>Une lettre majuscule</li>
      <li>Un chiffre</li>
    </ul>
  </div>
</form>
```

```css
/* Styles du formulaire avec tous les états */
.form-group {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  margin-bottom: var(--space-6);
}

.form-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--color-gray-700);
}

.required-indicator {
  color: var(--color-error);
  margin-left: var(--space-1);
}

.form-input {
  width: 100%;
  padding: var(--space-2) var(--space-4);
  font-size: 1rem;
  border: 1.5px solid var(--color-gray-300);
  border-radius: 6px;
  background: white;
  transition: border-color 0.15s, box-shadow 0.15s;
  outline: none;
}

/* Focus */
.form-input:focus {
  border-color: var(--color-primary-500);
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.15);
}

/* Valide */
.form-input:valid:not(:placeholder-shown) {
  border-color: var(--color-success);
}

/* Invalide (après tentative de soumission) */
.form-input.is-error {
  border-color: var(--color-error);
  background-color: var(--color-error-bg);
}

/* Disabled */
.form-input:disabled {
  background-color: var(--color-gray-100);
  color: var(--color-gray-400);
  cursor: not-allowed;
}

/* Placeholder */
.form-input::placeholder {
  color: var(--color-gray-400);
}

.form-hint {
  font-size: 0.75rem;
  color: var(--color-gray-500);
}

.form-error {
  font-size: 0.75rem;
  color: var(--color-error);
  min-height: 1rem; /* éviter les sauts de layout */
}
```

### 5.3 Cartes (Cards)

```html
<!-- Carte d'article -->
<article class="card">
  <img class="card-image" src="article.jpg" alt="Description de l'image">
  <div class="card-body">
    <div class="card-meta">
      <span class="tag">Design</span>
      <time datetime="2024-03-12">12 mars 2024</time>
    </div>
    <h3 class="card-title">
      <a href="/article" class="card-link">Titre de l'article</a>
    </h3>
    <p class="card-description">
      Description courte de l'article en deux ou trois lignes maximum pour ne pas
      surcharger la carte...
    </p>
    <div class="card-footer">
      <div class="author">
        <img class="author-avatar" src="avatar.jpg" alt="">
        <span>Jean Dupont</span>
      </div>
      <span class="reading-time">5 min</span>
    </div>
  </div>
</article>
```

```css
.card {
  background: white;
  border-radius: 12px;
  border: 1px solid var(--color-gray-200);
  overflow: hidden;
  transition: transform 0.2s, box-shadow 0.2s;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px -4px rgba(0, 0, 0, 0.1);
}

.card-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.card-body {
  padding: var(--space-6);
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
}

.card-meta {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  font-size: 0.75rem;
  color: var(--color-gray-500);
}

.card-title {
  font-size: 1.125rem;
  font-weight: 600;
  line-height: 1.4;
  color: var(--color-gray-900);
  margin: 0;
}

.card-link {
  color: inherit;
  text-decoration: none;
}

/* La zone cliquable couvre toute la carte */
.card-link::after {
  content: '';
  position: absolute;
  inset: 0;
}

.card { position: relative; } /* nécessaire pour card-link::after */
```

### 5.4 Navigation

```html
<!-- Navigation principale accessible -->
<header class="site-header">
  <nav class="nav" aria-label="Navigation principale">
    <a class="nav-logo" href="/" aria-label="Accueil — NomDuSite">
      <img src="logo.svg" alt="" width="120" height="40">
    </a>

    <!-- Bouton menu mobile -->
    <button
      class="nav-toggle"
      aria-expanded="false"
      aria-controls="nav-menu"
      aria-label="Ouvrir le menu"
    >
      <span class="hamburger-line"></span>
      <span class="hamburger-line"></span>
      <span class="hamburger-line"></span>
    </button>

    <ul class="nav-menu" id="nav-menu" role="list">
      <li><a class="nav-link" href="/produit">Produit</a></li>
      <li><a class="nav-link" href="/tarifs">Tarifs</a></li>
      <li>
        <!-- Sous-menu -->
        <button class="nav-link nav-dropdown-trigger" aria-expanded="false" aria-haspopup="true">
          Ressources
          <svg aria-hidden="true"><!-- chevron --></svg>
        </button>
        <ul class="nav-dropdown" role="list">
          <li><a href="/blog">Blog</a></li>
          <li><a href="/docs">Documentation</a></li>
        </ul>
      </li>
    </ul>

    <div class="nav-actions">
      <a class="btn-ghost" href="/connexion">Connexion</a>
      <a class="btn-primary" href="/inscription">Commencer</a>
    </div>
  </nav>
</header>
```

---

## 6. Icônes

### 6.1 SVG vs Icon Font

| Critère | SVG inline | SVG sprite | Icon font (Font Awesome) |
|---------|-----------|-----------|--------------------------|
| Performance | Bon (pas de requête) | Excellent (1 requête) | Mauvais (charge tout) |
| Accessibilité | Excellent (aria-*) | Excellent | Moyen (pseudo-éléments) |
| Personnalisation | Excellent (CSS complet) | Très bon | Limité (color seulement) |
| Multi-couleurs | Oui | Oui | Non |
| Mise en cache | Non | Oui | Oui |
| Usage recommandé | Icônes spécifiques | Applications | Legacy seulement |

### 6.2 Icône SVG accessible

```html
<!-- Icône décorative (pas d'info = hidden) -->
<svg aria-hidden="true" focusable="false" width="20" height="20">
  <use href="/icons.svg#icon-star"></use>
</svg>

<!-- Icône avec sens (bouton icon-only) -->
<button type="button" aria-label="Supprimer l'article">
  <svg aria-hidden="true" focusable="false" width="20" height="20">
    <use href="/icons.svg#icon-trash"></use>
  </svg>
</button>

<!-- Icône avec texte (icône = décorative) -->
<button type="button">
  <svg aria-hidden="true" focusable="false" width="16" height="16">
    <use href="/icons.svg#icon-download"></use>
  </svg>
  Télécharger
</button>
```

```css
/* Tailles cohérentes pour les icônes */
:root {
  --icon-xs: 12px;
  --icon-sm: 16px;
  --icon-md: 20px;
  --icon-lg: 24px;
  --icon-xl: 32px;
}

/* Icône alignée avec le texte */
.icon {
  display: inline-block;
  vertical-align: middle;
  fill: currentColor; /* hérite la couleur du texte parent */
  flex-shrink: 0;     /* ne se compresse pas dans un flex container */
}
```

> [!tip] Librairies SVG recommandées
> - **Heroicons** (Tailwind Labs) — propre, MIT
> - **Lucide** — fork amélioré de Feather Icons
> - **Phosphor Icons** — très complet, 6 styles
> - **Radix Icons** — minimaliste, parfait pour interfaces SaaS

---

## 7. Principes fondamentaux UX — Don Norman

### 7.1 Les 6 principes de Norman

Don Norman a formalisé dans *The Design of Everyday Things* les principes qui rendent un objet (ou une interface) intuitif :

**Affordances** — Un objet communique comment on peut l'utiliser.
```
Un bouton 3D donne envie d'être cliqué (affordance perçue de clic)
Un champ de formulaire avec une ombre intérieure suggère qu'on peut écrire dedans
Un lien souligné suggère qu'on peut naviguer vers un autre endroit
```

**Feedback** — L'interface répond à chaque action de l'utilisateur.
```
Clic bouton → changement visuel immédiat (< 100ms)
Soumission formulaire → loader visible (< 1s) puis confirmation
Upload fichier → barre de progression avec pourcentage
```

**Mapping** — La relation entre contrôle et résultat est naturelle.
```
MAUVAIS : Interrupteur gauche = lumière droite (mapping non intuitif)
BON     : Interrupteur gauche = lumière gauche (mapping spatial)

MAUVAIS : Bouton "Volume" qui agit sur la luminosité
BON     : Icône de son identifiable sur le curseur volume
```

**Visibilité** — Toutes les fonctions pertinentes sont visibles.
```
Ne pas cacher des actions importantes dans des menus profonds
Montrer les options disponibles (pas juste les actives)
Les affordances doivent être perceptibles sans hover
```

**Contraintes** — Empêcher les erreurs plutôt que les corriger.
```
Désactiver le bouton "Envoyer" tant que le formulaire est invalide
Ne pas proposer de dates passées dans un datepicker de réservation futur
Demander confirmation avant de supprimer des données importantes
```

**Cohérence** — Les patterns sont prévisibles à travers l'interface.
```
Même position pour la navigation sur toutes les pages
Même comportement pour tous les modales
Même icône pour la même action
```

### 7.2 Les lois UX essentielles

| Loi | Définition | Application pratique |
|-----|-----------|---------------------|
| **Loi de Fitts** | Le temps d'atteinte d'une cible dépend de sa taille et sa distance | Boutons importants = grands et proches (FAB mobile) |
| **Loi de Hick** | Plus il y a d'options, plus le temps de décision augmente | Limiter les items de menu à 5-7 max |
| **Loi de Miller** | La mémoire de travail traite 7±2 éléments | Grouper l'info, chunking, pagination |
| **Effet de position sérielle** | On retient mieux le début et la fin d'une liste | CTA en haut ET en bas des pages longues |
| **Loi de Jakob** | Les utilisateurs passent la plupart du temps sur d'autres sites | Respecter les conventions (ex: logo = lien accueil) |
| **Loi de Parkinson** | Une tâche occupe tout le temps disponible | Limiter les formulaires à l'essentiel |

---

## 8. Architecture d'information

### 8.1 Hiérarchie de l'information

L'architecture d'information (IA) organise le contenu pour qu'il soit trouvable. Elle se conçoit avant le design visuel.

**Exercice de tri par cartes (card sorting) :**
```
Méthode : écrire chaque page/section sur une carte,
faire trier par des utilisateurs réels en groupes logiques.

Résultat typique d'un site e-commerce :

Racine
├── Accueil
├── Catalogue
│   ├── Vêtements
│   │   ├── Hommes
│   │   ├── Femmes
│   │   └── Enfants
│   └── Accessoires
├── Mon compte
│   ├── Commandes
│   ├── Adresses
│   └── Paramètres
└── Service client
    ├── FAQ
    ├── Contact
    └── Retours
```

### 8.2 Flux utilisateur (User Flow)

Un user flow décrit le chemin d'un utilisateur pour accomplir une tâche.

```
Flux d'inscription :

[Arrivée page accueil]
         │
         ▼
  [Clic "S'inscrire"]
         │
         ▼
  ┌─────────────────────────────────┐
  │  Formulaire d'inscription       │
  │  • Email                        │
  │  • Mot de passe                 │
  │  • Confirmation MDP             │
  └─────────────────────────────────┘
         │
   ┌─────┴──────┐
   ▼            ▼
[Valide]    [Invalide]
   │             │
   │         [Afficher erreurs]
   │             │
   │         [Corriger] ────────┐
   │                            │
   │◄───────────────────────────┘
   ▼
[Email de confirmation envoyé]
         │
         ▼
[Clic lien email]
         │
         ▼
[Compte activé → Dashboard]
```

### 8.3 Patterns de navigation

| Pattern | Usage | Avantages | Inconvénients |
|---------|-------|-----------|---------------|
| **Top nav bar** | Sites contenu, SaaS | Familier, visible | Limité en espace mobile |
| **Sidebar** | Applications, dashboards | Hiérarchie visible, extensible | Prend de l'espace |
| **Bottom nav** | Applications mobiles | Accessibilité pouce | Limité à 5 items |
| **Hamburger menu** | Mobile uniquement | Économise l'espace | Navigation cachée |
| **Mega menu** | Grands e-commerce | Profondeur visible | Complexe, lourd |
| **Breadcrumb** | Complémentaire | Localisation utilisateur | Redondant si nav claire |

---

## 9. Wireframing

### 9.1 Lo-fi vs Hi-fi

| | Lo-fi | Hi-fi |
|-|-------|-------|
| **Fidélité** | Très basse — boîtes, textes placeholder | Haute — couleurs, polices, images réelles |
| **Outil** | Papier, Balsamiq, Whimsical | Figma, Sketch, Adobe XD |
| **Durée** | Minutes à heures | Heures à jours |
| **Usage** | Exploration rapide, validation de structure | Validation visuelle, handoff développeurs |
| **Quand** | Début du projet, sprint de design | Après validation lo-fi, avant développement |

### 9.2 Wireframe lo-fi en ASCII

```
Page d'accueil — Wireframe Lo-fi :

┌─────────────────────────────────────────────────┐
│ [LOGO]        [Nav Item] [Nav Item] [Nav Item]  │
│                                    [CTA]        │
├─────────────────────────────────────────────────┤
│                                                  │
│         ███████ HERO HEADLINE ████████           │
│         Sous-titre explicatif en 1-2 lignes      │
│                                                  │
│              [CTA Principal]  [CTA Sec.]         │
│                                                  │
│    ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│    │ Screenshot│  │Screenshot│  │Screenshot│     │
│    └──────────┘  └──────────┘  └──────────┘     │
├─────────────────────────────────────────────────┤
│                                                  │
│  SECTION FEATURES                               │
│                                                  │
│  ╔══════╗  ╔══════╗  ╔══════╗  ╔══════╗        │
│  ║ ICON ║  ║ ICON ║  ║ ICON ║  ║ ICON ║        │
│  ║      ║  ║      ║  ║      ║  ║      ║        │
│  ║Titre ║  ║Titre ║  ║Titre ║  ║Titre ║        │
│  ║Texte ║  ║Texte ║  ║Texte ║  ║Texte ║        │
│  ╚══════╝  ╚══════╝  ╚══════╝  ╚══════╝        │
│                                                  │
├─────────────────────────────────────────────────┤
│ Footer : [Logo] [Liens] [Liens] [Réseaux]       │
└─────────────────────────────────────────────────┘
```

### 9.3 Introduction à Figma

Figma est l'outil de design collaboratif standard de l'industrie (acquis par Adobe en 2022 mais resté indépendant). Il fonctionne dans le navigateur.

**Concepts fondamentaux :**

```
Workspace Figma :

  ┌─────────────────────────────────────────────────┐
  │ FILE BROWSER — projets et fichiers              │
  ├─────────────────────────────────────────────────┤
  │ TOOLBAR  [Move][Frame][Shape][Text][Pen][Hand]  │
  ├──────────┬──────────────────────────┬───────────┤
  │          │                          │           │
  │  LAYERS  │         CANVAS           │ INSPECT   │
  │  PANEL   │                          │ PANEL     │
  │          │   ┌──────────────────┐   │           │
  │ Page 1   │   │     FRAME        │   │ W: 375    │
  │  └ Frame │   │  ┌────────────┐  │   │ H: 812    │
  │    └ Box │   │  │  Component │  │   │ Fill:     │
  │      └Txt│   │  └────────────┘  │   │ #3B82F6   │
  │          │   └──────────────────┘   │           │
  └──────────┴──────────────────────────┴───────────┘
```

**Raccourcis essentiels :**

| Action | Raccourci |
|--------|-----------|
| Créer un frame | `F` |
| Créer un rectangle | `R` |
| Créer un texte | `T` |
| Copier/Coller | `Ctrl+C / Ctrl+V` |
| Dupliquer | `Ctrl+D` |
| Grouper | `Ctrl+G` |
| Créer un composant | `Ctrl+Alt+K` |
| Zoom ajusté | `Shift+1` |
| Zoom 100% | `Ctrl+0` |
| Rechercher | `Ctrl+P` |
| Auto layout | `Shift+A` |

---

## 10. Prototypage dans Figma

### 10.1 Frames et organisation

Les **Frames** sont les conteneurs de base en Figma (équivalents à des écrans ou des composants). On commence toujours par créer un Frame avec les bonnes dimensions.

**Dimensions standard :**

| Device | Dimensions |
|--------|-----------|
| iPhone 14 Pro | 393 × 852 px |
| iPhone SE | 375 × 667 px |
| Android standard | 360 × 800 px |
| iPad | 768 × 1024 px |
| Desktop | 1440 × 900 px |
| Desktop wide | 1920 × 1080 px |

### 10.2 Composants et variants

Un **composant** Figma est un élément réutilisable (l'équivalent d'un composant React). Les **variants** regroupent les différents états d'un composant.

```
Composant Bouton avec variants :

  Composant Principal (Main Component) :
  ┌─────────────────────────────────────────┐
  │  Variants du bouton :                   │
  │                                         │
  │  Type=Primary, Size=MD, State=Default   │
  │  [  Bouton Principal  ]                 │
  │                                         │
  │  Type=Primary, Size=MD, State=Hover     │
  │  [  Bouton Principal  ]  ← plus sombre  │
  │                                         │
  │  Type=Primary, Size=MD, State=Disabled  │
  │  [  Bouton Principal  ]  ← grisé        │
  │                                         │
  │  Type=Secondary, Size=MD, State=Default │
  │  [  Bouton Secondaire  ]  ← outline     │
  └─────────────────────────────────────────┘
```

### 10.3 Auto Layout

Auto Layout est l'équivalent de Flexbox dans Figma. Il rend les composants redimensionnables et adaptatifs.

```
Auto Layout — Propriétés :

┌─────────────────────────────────────────┐
│ Auto Layout                              │
│                                         │
│ Direction : [→ Horizontal] [↓ Vertical] │
│ Gap : [ 16px ]                          │
│ Padding : [ 12px ] [ 24px ]             │
│ Alignment : [◀] [◆] [▶]                │
│             [▲] [◆] [▼]                │
│ Sizing : [Hug contents] [Fill container]│
└─────────────────────────────────────────┘
```

### 10.4 Prototype interactif

L'onglet Prototype de Figma permet de créer des interactions entre frames.

```
Flux de prototype :

[Frame : Accueil]  →  Clic bouton  →  [Frame : Page produit]
[Frame : Accueil]  →  Hover card   →  [Frame : Card hover state]
[Frame : Modal]    →  Clic overlay →  [Frame : Modal fermée]
```

**Types d'interactions disponibles :**
- On Click / On Tap
- While Hovering
- On Drag
- Key/Gamepad

**Types de transitions :**
- Instant (pas d'animation)
- Dissolve (fondu)
- Smart Animate (anime les éléments communs)
- Move In / Move Out (slide)
- Push (pousse le contenu actuel)

---

## 11. User Testing

### 11.1 Pourquoi 5 utilisateurs suffisent

Jakob Nielsen (Nielsen Norman Group) a démontré statistiquement que **5 utilisateurs** permettent de détecter 85% des problèmes d'utilisabilité. Passer à 15 utilisateurs ne découvre que 5% de problèmes supplémentaires, pour un coût 3x plus élevé.

```
Problèmes détectés selon le nombre d'utilisateurs :

Utilisateurs  1    2    3    4    5    6   10   15
Problèmes %   31%  52%  67%  75%  85%  89%  95%  97%

          100% ┤                                  ╭───
               │                            ╭────╯
           85% ┤                       ╭────╯
               │                  ╭────╯
           67% ┤             ╭────╯
               │         ╭───╯
           31% ┤    ╭─────╯
               └────┴────┴────┴────┴────┴───────────
                    1    2    3    4    5   10    15
```

### 11.2 Think-Aloud Protocol

**Méthode :** demander à l'utilisateur de verbaliser ses pensées pendant qu'il utilise l'interface.

**Script d'introduction :**
```
"Bonjour, je vais vous demander d'utiliser [l'application].
Pendant que vous l'utilisez, verbalisez tout ce que vous pensez :
ce que vous voyez, ce que vous essayez de faire, ce qui vous surprend.
Il n'y a pas de bonne ou mauvaise réponse — c'est l'interface
que nous testons, pas vous. Soyez franc, ça nous aide vraiment.
Avez-vous des questions ?"
```

**Ce qu'on observe :**
- Hésitations et silences
- Expressions de confusion ou de surprise
- Clics inattendus
- Ce que l'utilisateur cherche visuellement
- Les termes qu'il utilise (vs ceux qu'on a mis dans l'interface)

### 11.3 Analyse des résultats

**Grille d'observation :**

```
| Tâche | Utilisateur | Réussi ? | Temps | Problème observé |
|-------|-------------|---------- |-------|------------------|
| S'inscrire | U1 | Oui | 2m30 | Confus par le champ "pseudonyme" |
| S'inscrire | U2 | Non | 5m00 | N'a pas vu le bouton sous la fold |
| S'inscrire | U3 | Oui | 1m45 | Aucun problème |
| Ajouter au panier | U1 | Oui | 0m45 | — |
| Ajouter au panier | U2 | Non | 3m00 | Cherchait dans le menu |
```

**Critères de sévérité des problèmes (Nielsen) :**

| Niveau | Description | Action |
|--------|-------------|--------|
| 0 | Pas un problème | Ignorer |
| 1 | Cosmétique | Corriger si temps libre |
| 2 | Problème mineur | Faible priorité |
| 3 | Problème majeur | Priorité importante |
| 4 | Catastrophique | À corriger avant lancement |

---

## 12. Accessibilité — WCAG 2.1

### 12.1 Les 4 principes POUR

WCAG 2.1 est organisé autour de 4 principes mnémotechniques : **POUR**

**Perceptible** — L'information doit être présentable sous des formes que les utilisateurs peuvent percevoir.
- Textes alternatifs pour les images
- Sous-titres pour les vidéos
- Contraste suffisant
- Contenu adaptable (pas de mise en forme seule pour transmettre l'info)

**Opérable** — Les composants d'interface et la navigation doivent être utilisables.
- Navigation au clavier complète
- Temps suffisant pour les contenus à durée limitée
- Pas de contenu qui clignote plus de 3 fois/seconde (risque épilepsie)
- Titres et labels descriptifs

**Compréhensible** — L'information et les opérations doivent être compréhensibles.
- Langue de la page déclarée (`lang="fr"`)
- Comportement prévisible des pages
- Aide pour corriger les erreurs

**Robuste** — Le contenu doit être interprétable par une variété de technologies.
- HTML valide
- Attributs ARIA corrects
- Compatibilité avec les technologies d'assistance

### 12.2 Les niveaux A, AA, AAA

| Niveau | Description | Obligation légale France |
|--------|-------------|--------------------------|
| **A** | Exigences de base | Oui (secteur public) |
| **AA** | Exigences standard | Oui (secteur public, recommandé privé) |
| **AAA** | Exigences avancées | Non (trop contraignant comme obligation universelle) |

> [!info] En France
> Le RGAA (Référentiel Général d'Accessibilité pour les Administrations) s'impose aux organismes publics et grandes entreprises. Il est basé sur WCAG 2.1 niveau AA.

### 12.3 Critères AA les plus importants

**Images et médias :**
```html
<!-- Image informative — alt obligatoire -->
<img src="graphique.png" alt="Graphique montrant une hausse de 23% des ventes en Q3 2024">

<!-- Image décorative — alt vide (pas omis !) -->
<img src="decoration.svg" alt="">

<!-- Image complexe (graphique, infographie) — description longue -->
<figure>
  <img src="schema.png" alt="Schéma de l'architecture" aria-describedby="schema-desc">
  <figcaption id="schema-desc">
    Le schéma montre 3 couches : présentation (React), logique (Node.js) et données (PostgreSQL).
    Les flèches indiquent les flux de données bidirectionnels entre chaque couche.
  </figcaption>
</figure>
```

**Formulaires :**
```html
<!-- Label explicitement associé — PAS seulement placeholder -->
<label for="search">Rechercher</label>
<input type="search" id="search" name="search" placeholder="Ex: chaussures rouges">

<!-- Groupe de champs radio/checkbox -->
<fieldset>
  <legend>Préférence de contact</legend>
  <label><input type="radio" name="contact" value="email"> Email</label>
  <label><input type="radio" name="contact" value="tel"> Téléphone</label>
</fieldset>

<!-- Erreur associée au champ -->
<label for="email">Email</label>
<input
  type="email"
  id="email"
  aria-invalid="true"
  aria-describedby="email-error"
>
<p id="email-error" role="alert">
  Veuillez entrer une adresse email valide (ex: jean@exemple.fr)
</p>
```

---

## 13. HTML sémantique et ARIA

### 13.1 HTML sémantique en premier

La règle d'or : **utiliser d'abord les éléments HTML natifs avant ARIA**. Un `<button>` vaut mieux qu'un `<div role="button">`.

```html
<!-- ❌ Mauvais — div non sémantique -->
<div class="btn" onclick="submit()">Envoyer</div>

<!-- ✅ Bon — élément natif avec comportement built-in -->
<button type="submit">Envoyer</button>
<!-- → focusable au clavier automatiquement
     → activable par Entrée/Espace automatiquement
     → annoncé "bouton" par le lecteur d'écran
     → attribut disabled géré nativement -->
```

**Éléments sémantiques fondamentaux :**

```html
<header>    <!-- En-tête de page ou de section -->
<nav>       <!-- Navigation principale -->
<main>      <!-- Contenu principal (unique par page) -->
<article>   <!-- Contenu autonome (article, post) -->
<section>   <!-- Section thématique avec titre -->
<aside>     <!-- Contenu tangentiel (sidebar, publicité) -->
<footer>    <!-- Pied de page ou de section -->
<h1>...<h6> <!-- Hiérarchie de titres (pas de saut) -->
<figure>    <!-- Média avec légende -->
<figcaption><!-- Légende du figure -->
<time>      <!-- Dates et heures -->
<address>   <!-- Coordonnées de contact -->
<mark>      <!-- Texte mis en surbrillance -->
<abbr>      <!-- Abréviation avec expansion -->
```

### 13.2 Attributs ARIA essentiels

ARIA (Accessible Rich Internet Applications) complète HTML pour les widgets complexes.

```html
<!-- aria-label : nom accessible quand le texte visible est insuffisant -->
<button aria-label="Fermer la boîte de dialogue">×</button>

<!-- aria-labelledby : référence un autre élément comme label -->
<h2 id="modal-title">Confirmer la suppression</h2>
<dialog aria-labelledby="modal-title">...</dialog>

<!-- aria-describedby : description supplémentaire -->
<input aria-describedby="password-hint">
<p id="password-hint">Le mot de passe doit contenir au moins 8 caractères.</p>

<!-- aria-expanded : état d'un élément accordéon/dropdown -->
<button aria-expanded="false" aria-controls="menu">Menu</button>
<ul id="menu" hidden>...</ul>

<!-- aria-current : élément actif dans une navigation -->
<nav>
  <a href="/" aria-current="page">Accueil</a>
  <a href="/blog">Blog</a>
</nav>

<!-- aria-live : zones mises à jour dynamiquement -->
<div aria-live="polite" aria-atomic="true">
  <!-- Le lecteur annonce les changements ici après l'action courante -->
  Formulaire soumis avec succès !
</div>

<!-- aria-busy : chargement en cours -->
<div aria-busy="true" aria-label="Chargement des résultats...">
  <span class="spinner"></span>
</div>

<!-- role : pour les composants complexes sans équivalent HTML natif -->
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="panel-1">Onglet 1</button>
  <button role="tab" aria-selected="false" aria-controls="panel-2">Onglet 2</button>
</div>
<div role="tabpanel" id="panel-1">Contenu onglet 1</div>
<div role="tabpanel" id="panel-2" hidden>Contenu onglet 2</div>
```

### 13.3 Navigation au clavier et gestion du focus

```javascript
// Trap focus dans une modale (accessibilité obligatoire)
function trapFocus(modalElement) {
  // Sélectionner tous les éléments focusables dans la modale
  const focusableSelectors = [
    'a[href]',
    'button:not([disabled])',
    'input:not([disabled])',
    'select:not([disabled])',
    'textarea:not([disabled])',
    '[tabindex]:not([tabindex="-1"])'
  ].join(', ');

  const focusableElements = modalElement.querySelectorAll(focusableSelectors);
  const firstElement = focusableElements[0];
  const lastElement = focusableElements[focusableElements.length - 1];

  // Mettre le focus sur le premier élément à l'ouverture
  firstElement.focus();

  modalElement.addEventListener('keydown', (event) => {
    if (event.key !== 'Tab') return;

    if (event.shiftKey) {
      // Shift+Tab : aller en arrière
      if (document.activeElement === firstElement) {
        event.preventDefault();
        lastElement.focus();
      }
    } else {
      // Tab : aller en avant
      if (document.activeElement === lastElement) {
        event.preventDefault();
        firstElement.focus();
      }
    }
  });
}

// Restaurer le focus à la fermeture de la modale
function openModal(modal, triggerButton) {
  modal.removeAttribute('hidden');
  trapFocus(modal);

  // Gérer Echap pour fermer
  const handleEscape = (event) => {
    if (event.key === 'Escape') {
      closeModal(modal, triggerButton);
      document.removeEventListener('keydown', handleEscape);
    }
  };
  document.addEventListener('keydown', handleEscape);
}

function closeModal(modal, triggerButton) {
  modal.setAttribute('hidden', '');
  triggerButton.focus(); // Retour au déclencheur
}
```

### 13.4 Ordre de lecture et tabindex

```html
<!-- L'ordre de tabulation doit suivre l'ordre visuel logique -->

<!-- ✅ tabindex="0" : ajoute à l'ordre de tab naturel -->
<div class="custom-widget" tabindex="0" role="button">Widget interactif</div>

<!-- ✅ tabindex="-1" : focusable via JavaScript, pas via Tab -->
<div id="modal-content" tabindex="-1">
  <!-- Utile pour le focus management programmatique -->
</div>

<!-- ❌ tabindex positif (>0) : éviter absolument -->
<!-- tabindex="5" brise l'ordre naturel et crée de la confusion -->
```

---

## 14. Tests d'accessibilité

### 14.1 axe DevTools

axe est une extension browser (Chrome/Firefox) qui scanne automatiquement une page pour les violations WCAG.

**Comment utiliser :**
1. Installer l'extension "axe DevTools" depuis le store
2. Ouvrir les DevTools (F12)
3. Aller dans l'onglet "axe DevTools"
4. Cliquer "Scan ALL of my page"
5. Examiner les violations par sévérité (Critical → Serious → Moderate → Minor)

```
Résultat axe — violations types :

Critical:
  ✗ Images must have alternate text
    #hero-image (img sans alt)
  ✗ Form elements must have labels
    #email-input (input sans label associé)

Serious:
  ✗ Color contrast must meet WCAG AA
    .nav-link (ratio 2.8:1, requis 4.5:1)

Moderate:
  ✗ Heading levels should only increase by one
    h4 après h2 (h3 manquant)
```

> [!tip] Automatiser axe en CI
> ```bash
> npm install --save-dev @axe-core/playwright
> ```
> ```javascript
> // tests/accessibility.test.js
> const { checkA11y } = require('axe-playwright');
> test('page accueil est accessible', async ({ page }) => {
>   await page.goto('http://localhost:3000');
>   await checkA11y(page, null, {
>     detailedReport: true,
>     detailedReportOptions: { html: true }
>   });
> });
> ```

### 14.2 Lighthouse

Lighthouse est intégré dans Chrome DevTools. Il mesure l'accessibilité (et aussi performance, SEO, PWA).

```
Score Lighthouse Accessibility :
Score sur 100, basé sur les critères axe les plus impactants

100 — Parfait (rare, ne vérifie pas tout)
 90 — Bon (peut être présenté comme "accessible")
 70 — Améliorations nécessaires
< 50 — Problèmes majeurs à corriger d'urgence
```

> [!warning] Limites des outils automatiques
> axe et Lighthouse détectent ~30-40% des problèmes d'accessibilité. Les 60% restants nécessitent des tests manuels : navigation au clavier, test avec lecteur d'écran, tests avec de vrais utilisateurs en situation de handicap.

### 14.3 Test avec lecteur d'écran

**NVDA (Windows, gratuit) :**
```
Installation : nvaccess.org/download
Mode navigation : Ins+F7 (liste des titres, liens, landmarks)
Lire la page : Ins+F6 ou simplement flèches
Arrêter : Ctrl
Saut de titre : H (titre suivant), 1-6 (niveau spécifique)
Saut de landmark : D
```

**VoiceOver (macOS/iOS, intégré) :**
```
Activer : Cmd+F5 (mac) ou Triple-clic bouton home (iOS)
Naviguer : VO+flèches (VO = Ctrl+Alt)
Rotor : VO+U (liste titres, liens, formulaires...)
```

**Ce qu'on vérifie :**
- Les titres forment une hiérarchie logique
- Tous les boutons ont un nom compréhensible
- Les images ont des descriptions pertinentes
- Les formulaires annoncent leurs erreurs
- La navigation au clavier est complète et logique

---

## 15. Responsive Design

### 15.1 Mobile-First

L'approche **mobile-first** consiste à écrire d'abord les styles pour mobile, puis à surcharger pour les écrans plus larges avec des media queries `min-width`. C'est l'opposé du responsive "desktop-first".

```css
/* === MOBILE FIRST === */

/* Base : mobile (0px → 639px) */
.hero {
  padding: var(--space-12) var(--space-6);  /* 48px 24px */
  text-align: center;
}

.hero-title {
  font-size: var(--font-size-2xl);  /* 32px sur mobile */
  line-height: 1.2;
}

.hero-actions {
  display: flex;
  flex-direction: column;           /* empilés sur mobile */
  gap: var(--space-4);
}

/* Tablette : ≥ 640px */
@media (min-width: 640px) {
  .hero-actions {
    flex-direction: row;            /* côte à côte sur tablette */
    justify-content: center;
  }
}

/* Desktop : ≥ 1024px */
@media (min-width: 1024px) {
  .hero {
    padding: var(--space-24) var(--space-16); /* plus grand sur desktop */
    text-align: left;
  }

  .hero-title {
    font-size: var(--font-size-3xl);  /* 48px sur desktop */
  }
}
```

### 15.2 Breakpoints standards

```css
/* Système de breakpoints — compatible Tailwind CSS */
:root {
  --breakpoint-sm:  640px;   /* Petits mobiles paysage, tablettes portrait */
  --breakpoint-md:  768px;   /* Tablettes */
  --breakpoint-lg:  1024px;  /* Desktop petit, tablettes paysage */
  --breakpoint-xl:  1280px;  /* Desktop standard */
  --breakpoint-2xl: 1536px;  /* Desktop large */
}

/* Media queries correspondantes */
@media (min-width: 640px)  { /* sm  */ }
@media (min-width: 768px)  { /* md  */ }
@media (min-width: 1024px) { /* lg  */ }
@media (min-width: 1280px) { /* xl  */ }
@media (min-width: 1536px) { /* 2xl */ }
```

### 15.3 Images adaptatives

```html
<!-- Image responsive avec srcset et sizes -->
<img
  src="photo-800.jpg"
  srcset="
    photo-400.jpg  400w,
    photo-800.jpg  800w,
    photo-1200.jpg 1200w,
    photo-1600.jpg 1600w
  "
  sizes="
    (max-width: 640px)  100vw,
    (max-width: 1024px) 50vw,
    33vw
  "
  alt="Description de la photo"
  loading="lazy"
  decoding="async"
  width="800"
  height="600"
>

<!-- Image orientée selon le contexte (art direction) -->
<picture>
  <!-- Version mobile : image carrée, centrée sur le sujet -->
  <source
    media="(max-width: 639px)"
    srcset="photo-portrait-400.jpg"
  >
  <!-- Version desktop : image panoramique -->
  <source
    media="(min-width: 640px)"
    srcset="photo-paysage-1200.jpg"
  >
  <img src="photo-paysage-1200.jpg" alt="Description">
</picture>

<!-- SVG : nativement scalable, idéal pour logos et icônes -->
<img src="logo.svg" alt="NomDuSite" width="120" height="40">
```

```css
/* CSS pour les images responsives */
img {
  max-width: 100%;      /* ne jamais dépasser le conteneur */
  height: auto;         /* conserver le ratio */
  display: block;       /* éviter l'espace sous l'image (inline baseline) */
}

/* Image de fond responsive */
.hero-bg {
  background-image: url('hero-mobile.jpg');
  background-size: cover;
  background-position: center;
}

@media (min-width: 768px) {
  .hero-bg {
    background-image: url('hero-desktop.jpg');
  }
}
```

### 15.4 Responsive typography (Fluid Type)

```css
/* Typographie fluide avec clamp() */
/* clamp(minimum, préféré, maximum) */

h1 {
  /* 32px sur mobile → 64px sur desktop, proportionnel entre */
  font-size: clamp(2rem, 5vw + 1rem, 4rem);
}

h2 {
  font-size: clamp(1.5rem, 3vw + 0.75rem, 2.5rem);
}

body {
  /* 16px sur mobile, 18px sur desktop */
  font-size: clamp(1rem, 0.5vw + 0.875rem, 1.125rem);
  /* La lecture reste confortable à toutes les tailles */
}
```

### 15.5 Layout responsive — patterns communs

```css
/* Pattern 1 : Holy Grail Layout (header, sidebar, main, footer) */
.page-layout {
  display: grid;
  grid-template-areas:
    "header"
    "main"
    "sidebar"
    "footer";
  grid-template-columns: 1fr;
  min-height: 100vh;
}

@media (min-width: 768px) {
  .page-layout {
    grid-template-areas:
      "header  header"
      "sidebar main"
      "footer  footer";
    grid-template-columns: 250px 1fr;
  }
}

/* Pattern 2 : Auto-fit grid (responsive sans media queries) */
.auto-grid {
  display: grid;
  grid-template-columns: repeat(
    auto-fit,
    minmax(min(280px, 100%), 1fr)
  );
  gap: var(--space-6);
}
/* → 1 colonne sur mobile, 2-3-4 automatiquement selon l'espace */

/* Pattern 3 : Stack / Inline switch */
.button-group {
  display: flex;
  flex-wrap: wrap;   /* passe à la ligne si pas assez de place */
  gap: var(--space-4);
}
```

---

## 16. Exercices pratiques

### Exercice 1 — Audit CARP

> [!tip] Challenge
> Prendre un site de votre choix (ou le site d'une école, d'une mairie) et identifier :
> 1. Un manquement au principe de Contraste
> 2. Un manquement au principe d'Alignement
> 3. Une application réussie de la Répétition
> 4. Un problème de Proximité
>
> Prendre des captures d'écran et annoter les problèmes trouvés.

### Exercice 2 — Système typographique

> [!tip] Challenge
> Créer un fichier `typography.css` contenant :
> 1. Un système de variables CSS pour les tailles (échelle 1.25)
> 2. Les styles pour h1-h4 et body
> 3. Un composant `.prose` avec `max-width: 65ch` et `line-height` optimal
> 4. Une démo HTML qui montre tous les niveaux
>
> Vérifier le rendu à la fois sur mobile (375px) et desktop (1280px).

### Exercice 3 — Palette accessible

> [!tip] Challenge
> Construire une palette de 5 couleurs pour un site fictif de votre choix :
> - 1 couleur primaire (avec 5 teintes)
> - 1 couleur secondaire
> - Les couleurs sémantiques (success, error, warning)
> - Vérifier que tous les couples texte/fond passent WCAG AA
>
> Utiliser [coolors.co](https://coolors.co) pour générer, [webaim.org/resources/contrastchecker](https://webaim.org/resources/contrastchecker/) pour vérifier.

### Exercice 4 — Composant bouton complet

> [!tip] Challenge
> Créer un composant bouton en HTML/CSS avec :
> - 3 variants : primary, secondary, ghost
> - 3 tailles : sm, md, lg
> - Tous les états : default, hover, focus-visible, active, disabled, loading
> - Accessible au clavier et annoncé correctement par un lecteur d'écran
>
> Bonus : ajouter un variant avec icône à gauche du texte.

### Exercice 5 — Wireframe et audit UX

> [!tip] Challenge
> 1. Choisir un formulaire d'inscription réel (Airbnb, GitHub, votre école)
> 2. Dessiner un wireframe lo-fi du flux actuel (papier ou Figma)
> 3. Identifier 3 violations des principes de Norman
> 4. Proposer une version améliorée du wireframe
> 5. Expliquer comment tester vos améliorations avec 5 utilisateurs

### Exercice 6 — Audit d'accessibilité complet

> [!tip] Challenge — Niveau avancé
> Sur un projet web que vous avez réalisé en cours :
> 1. Lancer Lighthouse Accessibility → noter le score de départ
> 2. Scanner avec axe DevTools → lister toutes les violations
> 3. Corriger les violations Critical et Serious
> 4. Tester la navigation complète au clavier (Tab, Shift+Tab, Enter, Espace, Echap)
> 5. Activer NVDA ou VoiceOver et naviguer dans le formulaire principal
> 6. Relancer Lighthouse → comparer les scores avant/après

---

## 17. Récapitulatif et bonnes pratiques

### 17.1 Checklist UI avant livraison

```
□ Hiérarchie typographique claire (au moins 3 niveaux distincts)
□ Palette de couleurs avec contraste WCAG AA vérifié
□ Espacement cohérent (multiples de 8px)
□ Tous les composants interactifs ont tous leurs états définis
□ Les formulaires ont des labels visibles (pas seulement placeholder)
□ Les erreurs de formulaire sont claires et associées aux champs
□ Les images ont des textes alternatifs appropriés
□ La typographie est lisible (font-size ≥ 16px body, line-height ≥ 1.5)
□ Les boutons sont suffisamment grands (min 44×44px pour mobile)
```

### 17.2 Checklist UX avant livraison

```
□ Chaque action importante donne un feedback visible (< 1 seconde)
□ Les erreurs expliquent comment les corriger
□ La navigation principale a ≤ 7 items
□ Le chemin vers l'action principale est ≤ 3 clics depuis l'accueil
□ Les actions destructives (supprimer) demandent confirmation
□ Le contenu le plus important est visible sans scroll (above the fold)
□ Le site a une page 404 utile et un plan du site
□ Les messages de chargement sont présents pour les opérations > 1 sec
```

### 17.3 Checklist accessibilité avant livraison

```
□ Score Lighthouse Accessibility ≥ 90
□ Zéro violation axe Critical ou Serious
□ Navigation complète au clavier fonctionne (Tab, Shift+Tab, Enter, Esc)
□ Focus visible sur tous les éléments interactifs
□ Pas de `outline: none` sans alternative visible
□ Contraste 4.5:1 minimum pour tout texte normal
□ Contraste 3:1 minimum pour grand texte et composants UI
□ Toutes les images ont des alt appropriés
□ Les formulaires ont des labels explicites (fieldset+legend pour radio/checkbox)
□ Les erreurs sont annoncées via aria-live ou role="alert"
□ La langue de la page est déclarée : <html lang="fr">
□ La hiérarchie des titres est logique (h1 → h2 → h3, pas de sauts)
□ Responsive fonctionne de 320px à 1920px
```

### 17.4 Ressources de référence

| Ressource | URL | Usage |
|-----------|-----|-------|
| WCAG 2.1 | w3.org/TR/WCAG21/ | Standard complet |
| RGAA | accessibilite.numerique.gouv.fr | Version française |
| WebAIM | webaim.org | Guides pratiques |
| Contrast Checker | webaim.org/resources/contrastchecker/ | Vérification contaste |
| axe DevTools | deque.com/axe/devtools/ | Extension browser |
| Google Fonts | fonts.google.com | Polices web |
| Heroicons | heroicons.com | Icônes SVG |
| Coolors | coolors.co | Générateur palettes |
| Figma | figma.com | Design/prototypage |
| Nielsen Norman | nngroup.com | Articles UX de référence |

> [!info] Lien avec les autres cours Holberton
> - **HTML Fondamentaux** : les éléments sémantiques couverts ici sont l'extension naturelle de ce cours
> - **CSS Fondamentaux** : le système de variables CSS et les media queries s'appuient directement sur ce cours
> - **CSS Avancé** : Flexbox et Grid approfondis pour les layouts responsives
> - **React / Vue / Svelte** : les composants UI avec états se translatent directement en composants framework
> - **Tests et Qualité** : les tests axe s'intègrent dans une pipeline de tests automatisés
