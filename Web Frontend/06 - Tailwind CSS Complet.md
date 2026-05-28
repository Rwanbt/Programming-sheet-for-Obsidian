# Tailwind CSS Complet

> [!info] Utility-First CSS
> Tailwind CSS est un framework CSS "utility-first" : au lieu de composants prédéfinis, vous composez les interfaces avec des classes atomiques de bas niveau directement dans votre HTML. Plus de CSS custom — l'interface est décrite là où elle est utilisée.

## Table des matières
1. [[#Utility-First vs Component-Based]]
2. [[#Installation et configuration]]
3. [[#Utilitaires essentiels]]
4. [[#Responsive Design]]
5. [[#Variants]]
6. [[#Dark Mode]]
7. [[#Configuration personnalisée]]
8. [[#Directive @apply]]
9. [[#Plugins Tailwind]]
10. [[#Tailwind v4]]
11. [[#Intégration avec les frameworks]]
12. [[#Best Practices]]
13. [[#Exercices Pratiques]]

---

## Utility-First vs Component-Based

### Philosophie comparée

| Aspect | Utility-First (Tailwind) | Component-Based (Bootstrap) |
|--------|--------------------------|------------------------------|
| **Approche** | Classes atomiques dans le HTML | Composants CSS prédéfinis |
| **Personnalisation** | Totale, via config | Limitée, via variables SASS |
| **CSS généré** | Uniquement ce qui est utilisé | Bundle complet (~200 KB min) |
| **Courbe d'apprentissage** | Plus longue (noms de classes) | Plus rapide (composants prêts) |
| **Design system** | Construit sur mesure | Identité Bootstrap reconnaissable |
| **Prototypage** | Rapide avec JIT | Très rapide |
| **Maintenance** | CSS minimal, HTML plus verbeux | CSS peut dériver |
| **Design unique** | Facile | Difficile sans overrides lourds |

### Exemple concret

```html
<!-- Bootstrap : classe composant -->
<button class="btn btn-primary btn-lg">Envoyer</button>

<!-- Tailwind : classes utilitaires composées -->
<button class="bg-blue-600 hover:bg-blue-700 text-white font-semibold 
               py-3 px-6 rounded-lg transition-colors duration-200 
               focus:outline-none focus:ring-2 focus:ring-blue-500">
    Envoyer
</button>
```

> [!tip] Le vrai avantage Tailwind
> Le CSS total d'un projet Tailwind en production fait généralement **5 à 15 KB** (après purge/JIT) contre 150-200 KB pour Bootstrap. Et chaque bouton peut être unique sans écrire une ligne de CSS.

---

## Installation et configuration

### Option 1 — CDN (prototypage seulement)

```html
<!-- Dans le <head> — Tailwind v4 CDN -->
<script src="https://cdn.tailwindcss.com"></script>
```

> [!warning] CDN uniquement pour le prototypage
> Le CDN charge 3 MB de CSS non purgé. Ne jamais utiliser en production.

### Option 2 — Vite + PostCSS (production)

```bash
# Nouveau projet Vite
npm create vite@latest mon-projet -- --template vanilla
cd mon-projet

# Installer Tailwind v3
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p  # Crée tailwind.config.js et postcss.config.js

# Ou Tailwind v4 (nouvelle syntaxe)
npm install -D tailwindcss@next @tailwindcss/vite
```

### Configuration Tailwind v3

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
    // Content paths — CRUCIAL pour le purge CSS
    content: [
        './index.html',
        './src/**/*.{js,ts,jsx,tsx,vue,svelte}',
        // Pour Laravel :
        // './resources/**/*.blade.php',
    ],
    
    // Dark mode
    darkMode: 'class', // ou 'media'
    
    theme: {
        // Remplace complètement les valeurs par défaut
        fontFamily: {
            sans: ['Inter', 'sans-serif'],
        },
        
        // Étend les valeurs par défaut
        extend: {
            colors: {
                brand: {
                    50:  '#eff6ff',
                    100: '#dbeafe',
                    500: '#3b82f6',
                    600: '#2563eb',
                    700: '#1d4ed8',
                    900: '#1e3a8a',
                },
            },
            spacing: {
                '128': '32rem',
                '144': '36rem',
            },
            borderRadius: {
                '4xl': '2rem',
            },
            animation: {
                'fade-in': 'fadeIn 0.3s ease-in-out',
            },
            keyframes: {
                fadeIn: {
                    '0%': { opacity: '0', transform: 'translateY(-10px)' },
                    '100%': { opacity: '1', transform: 'translateY(0)' },
                },
            },
        },
    },
    
    plugins: [
        require('@tailwindcss/forms'),
        require('@tailwindcss/typography'),
    ],
}
```

```css
/* src/main.css — Point d'entrée CSS */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## Utilitaires essentiels

### Spacing (espacement)

```html
<!-- Padding -->
<div class="p-4">          <!-- padding: 1rem (16px) sur tous les côtés -->
<div class="px-6 py-3">    <!-- padding horizontal 1.5rem, vertical 0.75rem -->
<div class="pt-2 pb-4">    <!-- top 0.5rem, bottom 1rem -->
<div class="pl-8 pr-4">    <!-- left 2rem, right 1rem -->

<!-- Margin -->
<div class="m-auto">       <!-- margin: auto (centrer) -->
<div class="mt-8 mb-4">    <!-- margin-top 2rem, margin-bottom 1rem -->
<div class="mx-auto">      <!-- centrer horizontalement -->
<div class="-mt-4">        <!-- margin négatif -->

<!-- Gap (flexbox/grid) -->
<div class="flex gap-4">   <!-- gap: 1rem -->
<div class="grid gap-x-6 gap-y-3"> <!-- gap horizontal 1.5rem, vertical 0.75rem -->

<!-- Space between (entre enfants) -->
<div class="flex space-x-4"> <!-- margin-left: 1rem sur chaque enfant sauf le premier -->
```

### Sizing (dimensions)

```html
<!-- Width -->
<div class="w-full">       <!-- width: 100% -->
<div class="w-1/2">        <!-- width: 50% -->
<div class="w-64">         <!-- width: 16rem (256px) -->
<div class="w-screen">     <!-- width: 100vw -->
<div class="w-fit">        <!-- width: fit-content -->
<div class="w-max">        <!-- width: max-content -->
<div class="min-w-0">      <!-- min-width: 0 (corrige flex overflow) -->
<div class="max-w-prose">  <!-- max-width: 65ch (lisibilité texte) -->
<div class="max-w-screen-lg"> <!-- max-width: 1024px -->

<!-- Height -->
<div class="h-screen">     <!-- height: 100vh -->
<div class="h-full">       <!-- height: 100% -->
<div class="h-16">         <!-- height: 4rem (64px) -->
<div class="min-h-screen"> <!-- min-height: 100vh -->

<!-- Aspect ratio -->
<div class="aspect-video"> <!-- 16/9 -->
<div class="aspect-square"> <!-- 1/1 -->
```

### Typography

```html
<!-- Taille de texte -->
<p class="text-xs">    <!-- 12px -->
<p class="text-sm">    <!-- 14px -->
<p class="text-base">  <!-- 16px (défaut) -->
<p class="text-lg">    <!-- 18px -->
<p class="text-xl">    <!-- 20px -->
<p class="text-2xl">   <!-- 24px -->
<p class="text-3xl">   <!-- 30px -->
<p class="text-4xl">   <!-- 36px -->
<p class="text-5xl">   <!-- 48px -->
<p class="text-6xl">   <!-- 60px -->

<!-- Font weight -->
<p class="font-thin">       <!-- font-weight: 100 -->
<p class="font-light">      <!-- font-weight: 300 -->
<p class="font-normal">     <!-- font-weight: 400 -->
<p class="font-medium">     <!-- font-weight: 500 -->
<p class="font-semibold">   <!-- font-weight: 600 -->
<p class="font-bold">       <!-- font-weight: 700 -->
<p class="font-extrabold">  <!-- font-weight: 800 -->

<!-- Couleur de texte -->
<p class="text-gray-900">
<p class="text-blue-600">
<p class="text-red-500">
<p class="text-white">
<p class="text-transparent bg-clip-text bg-gradient-to-r from-blue-600 to-purple-600">

<!-- Alignement et espacement -->
<p class="text-center">
<p class="text-right">
<p class="text-justify">
<p class="leading-tight">    <!-- line-height: 1.25 -->
<p class="leading-relaxed">  <!-- line-height: 1.625 -->
<p class="tracking-wide">    <!-- letter-spacing: 0.025em -->
<p class="tracking-widest">  <!-- letter-spacing: 0.1em -->

<!-- Décorations -->
<p class="underline decoration-blue-500 decoration-2">
<p class="line-through">
<p class="uppercase">
<p class="capitalize">
<p class="truncate">          <!-- text-overflow: ellipsis -->
<p class="break-words">       <!-- word-break: break-word -->
```

### Colors — Palette

Tailwind fournit une palette complète. Format : `{couleur}-{intensité}` (50 à 950).

```html
<!-- Couleurs disponibles -->
slate, gray, zinc, neutral, stone,   <!-- Neutres -->
red, orange, amber, yellow,          <!-- Chauds -->
lime, green, emerald, teal,          <!-- Verts -->
cyan, sky, blue, indigo, violet,     <!-- Bleus/Violets -->
purple, fuchsia, pink, rose          <!-- Roses -->

<!-- Exemples d'application -->
<div class="bg-slate-100 text-slate-900">
<div class="bg-emerald-500 text-white">
<div class="bg-gradient-to-br from-purple-500 to-pink-500">
<div class="border border-gray-200">
<div class="ring-2 ring-blue-500 ring-offset-2">
```

### Display et Flexbox

```html
<!-- Display -->
<div class="block">
<div class="inline-block">
<div class="inline">
<div class="hidden">          <!-- display: none -->
<div class="flex">
<div class="inline-flex">
<div class="grid">

<!-- Flexbox -->
<div class="flex flex-row">           <!-- direction: row (défaut) -->
<div class="flex flex-col">           <!-- direction: column -->
<div class="flex flex-row-reverse">
<div class="flex flex-wrap">          <!-- flex-wrap: wrap -->
<div class="flex flex-nowrap">

<!-- Justify (axe principal) -->
<div class="flex justify-start">      <!-- flex-start -->
<div class="flex justify-center">     <!-- center -->
<div class="flex justify-end">        <!-- flex-end -->
<div class="flex justify-between">    <!-- space-between -->
<div class="flex justify-around">     <!-- space-around -->
<div class="flex justify-evenly">     <!-- space-evenly -->

<!-- Align (axe secondaire) -->
<div class="flex items-start">
<div class="flex items-center">
<div class="flex items-end">
<div class="flex items-stretch">      <!-- défaut -->
<div class="flex items-baseline">

<!-- Flex children -->
<div class="flex-1">                  <!-- flex: 1 1 0% -->
<div class="flex-auto">               <!-- flex: 1 1 auto -->
<div class="flex-none">               <!-- flex: none -->
<div class="flex-grow">
<div class="flex-shrink-0">           <!-- ne pas rétrécir -->
<div class="self-center">             <!-- align-self: center -->
<div class="order-first">             <!-- order: -9999 -->
<div class="order-last">              <!-- order: 9999 -->
```

### Grid

```html
<!-- Grid container -->
<div class="grid grid-cols-3">           <!-- 3 colonnes égales -->
<div class="grid grid-cols-12">          <!-- 12 colonnes (like Bootstrap) -->
<div class="grid grid-cols-[1fr_2fr_1fr]"> <!-- custom avec valeurs arbitraires -->

<!-- Grid items -->
<div class="col-span-2">               <!-- colspan="2" -->
<div class="col-span-full">            <!-- colspan toute la largeur -->
<div class="col-start-2 col-end-4">   <!-- de colonne 2 à 4 -->
<div class="row-span-2">

<!-- Gap -->
<div class="grid grid-cols-3 gap-4">
<div class="grid grid-cols-3 gap-x-6 gap-y-4">

<!-- Auto placement -->
<div class="grid grid-cols-3 auto-rows-fr">    <!-- lignes égales -->
<div class="grid grid-flow-col">               <!-- remplissage par colonnes -->
```

### Positioning

```html
<div class="relative">               <!-- position: relative -->
<div class="absolute">               <!-- position: absolute -->
<div class="fixed">                  <!-- position: fixed -->
<div class="sticky top-0">          <!-- position: sticky, top: 0 -->

<!-- Inset (top/right/bottom/left) -->
<div class="absolute inset-0">       <!-- top:0 right:0 bottom:0 left:0 (couvre parent) -->
<div class="absolute top-4 right-4">
<div class="absolute bottom-0 left-1/2 -translate-x-1/2"> <!-- centré horizontalement -->

<!-- Z-index -->
<div class="z-10">
<div class="z-50">
```

### Borders, Shadows, Backgrounds

```html
<!-- Borders -->
<div class="border">                  <!-- border: 1px solid -->
<div class="border-2 border-gray-300">
<div class="border-b border-gray-200">  <!-- seulement en bas -->
<div class="rounded">                 <!-- border-radius: 0.25rem -->
<div class="rounded-lg">              <!-- 0.5rem -->
<div class="rounded-xl">              <!-- 0.75rem -->
<div class="rounded-2xl">             <!-- 1rem -->
<div class="rounded-full">            <!-- 9999px (cercle) -->
<div class="divide-y divide-gray-200"> <!-- borders entre enfants -->

<!-- Shadows -->
<div class="shadow-sm">
<div class="shadow">
<div class="shadow-md">
<div class="shadow-lg">
<div class="shadow-xl">
<div class="shadow-2xl">
<div class="shadow-none">

<!-- Ring (outline avec offset) -->
<input class="ring-2 ring-blue-500 ring-offset-2">

<!-- Backgrounds -->
<div class="bg-white">
<div class="bg-transparent">
<div class="bg-gradient-to-r from-cyan-500 to-blue-500">
<div class="bg-gradient-to-br from-purple-600 via-pink-500 to-orange-400">
<div class="bg-cover bg-center" style="background-image: url(...)">

<!-- Opacity -->
<div class="opacity-50">
<div class="bg-black/50">              <!-- background-color avec opacité via / -->
```

### Transitions et animations

```html
<!-- Transitions -->
<div class="transition">                          <!-- all, 150ms, ease -->
<div class="transition-colors duration-200">      <!-- seulement les couleurs -->
<div class="transition-all duration-300 ease-in-out">

<!-- Transform -->
<div class="hover:scale-105 transition-transform">
<div class="rotate-45">
<div class="translate-x-4">
<div class="-translate-y-2">
<div class="hover:-translate-y-1 transition-transform duration-200">
```

---

## Responsive Design

### Breakpoints

| Préfixe | Taille min | Équivalent |
|---------|-----------|------------|
| (aucun) | 0px | Mobile first |
| `sm:` | 640px | Grand mobile / tablette portrait |
| `md:` | 768px | Tablette |
| `lg:` | 1024px | Laptop |
| `xl:` | 1280px | Desktop |
| `2xl:` | 1536px | Grand écran |

### Mobile-first approach

```html
<!-- Mobile : 1 colonne, Desktop : 3 colonnes -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
    <div>...</div>
    <div>...</div>
    <div>...</div>
</div>

<!-- Texte : petit sur mobile, grand sur desktop -->
<h1 class="text-2xl md:text-4xl lg:text-6xl font-bold">
    Titre principal
</h1>

<!-- Masquer sur mobile, afficher sur desktop -->
<nav class="hidden lg:flex items-center gap-6">...</nav>

<!-- Afficher sur mobile, masquer sur desktop -->
<button class="lg:hidden">☰</button>

<!-- Padding adaptatif -->
<div class="px-4 sm:px-6 lg:px-8 py-8 lg:py-16">
```

---

## Variants

### Hover, Focus, Active

```html
<button class="
    bg-blue-600
    hover:bg-blue-700
    active:bg-blue-800
    focus:outline-none
    focus:ring-2
    focus:ring-blue-500
    focus:ring-offset-2
    disabled:opacity-50
    disabled:cursor-not-allowed
    transition-colors
">
    Bouton interactif
</button>

<!-- Focus-visible (meilleur que focus pour l'accessibilité) -->
<a class="focus-visible:ring-2 focus-visible:ring-blue-500">Lien</a>
```

### Group et Peer

```html
<!-- group: styler les enfants selon l'état du parent -->
<div class="group">
    <img class="group-hover:opacity-80 transition-opacity" src="...">
    <div class="opacity-0 group-hover:opacity-100 transition-opacity">
        <p>Visible au hover du parent</p>
    </div>
</div>

<!-- peer: styler un élément selon l'état d'un frère -->
<input type="checkbox" class="peer hidden" id="toggle">
<label for="toggle" class="peer-checked:bg-blue-600 ...">Toggle</label>
<div class="hidden peer-checked:block">Contenu révélé</div>
```

### State variants

```html
<!-- Premiers/derniers enfants -->
<li class="first:mt-0 mt-4">...</li>
<li class="last:border-b-0 border-b">...</li>

<!-- Paires/impaires (tables) -->
<tr class="odd:bg-white even:bg-gray-50">

<!-- Placeholder -->
<input class="placeholder:text-gray-400 placeholder:italic">

<!-- Required/Invalid -->
<input class="invalid:border-red-500 invalid:ring-red-500">
<input class="required:border-blue-300">

<!-- Selection -->
<p class="selection:bg-yellow-200 selection:text-yellow-900">
    Sélectionnez ce texte
</p>
```

---

## Dark Mode

### Configuration

```javascript
// tailwind.config.js
export default {
    darkMode: 'class', // Contrôlé par JS (ajouter classe 'dark' sur <html>)
    // ou
    darkMode: 'media', // Suit la préférence système
}
```

### Utilisation

```html
<body class="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">
    <div class="bg-white dark:bg-gray-800 shadow-md dark:shadow-gray-700">
        <h1 class="text-gray-900 dark:text-white">Titre</h1>
        <p class="text-gray-600 dark:text-gray-300">Contenu</p>
        <button class="bg-blue-600 dark:bg-blue-500 hover:bg-blue-700 dark:hover:bg-blue-400 text-white">
            Action
        </button>
    </div>
</body>
```

### Toggle Dark Mode avec JavaScript

```javascript
// Toggle
const html = document.documentElement

function toggleDark() {
    html.classList.toggle('dark')
    localStorage.setItem('theme', html.classList.contains('dark') ? 'dark' : 'light')
}

// Initialiser selon les préférences
function initTheme() {
    const saved = localStorage.getItem('theme')
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
    
    if (saved === 'dark' || (!saved && prefersDark)) {
        html.classList.add('dark')
    }
}

initTheme()
```

---

## Configuration personnalisée

### Étendre le thème

```javascript
// tailwind.config.js
export default {
    theme: {
        extend: {
            // Couleurs personnalisées
            colors: {
                primary: {
                    DEFAULT: '#2563eb',
                    hover: '#1d4ed8',
                    light: '#dbeafe',
                    dark: '#1e40af',
                },
                success: '#16a34a',
                warning: '#d97706',
                error: '#dc2626',
            },
            
            // Fonts
            fontFamily: {
                display: ['Playfair Display', 'serif'],
                body: ['Inter', 'sans-serif'],
                mono: ['JetBrains Mono', 'monospace'],
            },
            
            // Breakpoints custom
            screens: {
                'xs': '475px',
                '3xl': '1920px',
            },
            
            // Spacing custom
            spacing: {
                '18': '4.5rem',
                '88': '22rem',
                '128': '32rem',
            },
            
            // Border radius
            borderRadius: {
                '4xl': '2rem',
                '5xl': '2.5rem',
            },
            
            // Box shadow
            boxShadow: {
                'inner-lg': 'inset 0 4px 6px -1px rgb(0 0 0 / 0.1)',
                'colored': '0 4px 14px 0 rgb(37 99 235 / 0.39)',
            },
            
            // Animations
            transitionDuration: {
                '400': '400ms',
            },
        },
    },
}
```

### Valeurs arbitraires (JIT)

```html
<!-- Tailwind JIT permet des valeurs exactes entre crochets -->
<div class="w-[342px]">
<div class="top-[117px]">
<div class="text-[#1da1f2]">
<div class="bg-[url('/img/hero.jpg')]">
<div class="grid-cols-[1fr_500px_2fr]">
<div class="before:content-['→']">
```

---

## Directive @apply

> [!warning] Quand utiliser @apply
> `@apply` est utile pour extraire des patterns **répétés dans des templates** (composant bouton utilisé 50 fois). Ne pas l'utiliser pour tout — on perd l'avantage utility-first.

```css
/* src/styles/components.css */
@layer components {
    /* Bouton réutilisable */
    .btn {
        @apply inline-flex items-center justify-center 
               px-4 py-2 rounded-lg font-medium 
               transition-colors duration-200
               focus:outline-none focus:ring-2 focus:ring-offset-2;
    }
    
    .btn-primary {
        @apply btn bg-blue-600 text-white 
               hover:bg-blue-700 focus:ring-blue-500;
    }
    
    .btn-secondary {
        @apply btn bg-gray-100 text-gray-900 
               hover:bg-gray-200 focus:ring-gray-500;
    }
    
    .btn-danger {
        @apply btn bg-red-600 text-white 
               hover:bg-red-700 focus:ring-red-500;
    }
    
    /* Card */
    .card {
        @apply bg-white rounded-xl shadow-md overflow-hidden;
    }
    
    .card-body {
        @apply p-6;
    }
    
    /* Form controls */
    .form-input {
        @apply w-full px-3 py-2 border border-gray-300 rounded-lg 
               text-sm placeholder:text-gray-400
               focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent
               disabled:opacity-50 disabled:bg-gray-50;
    }
    
    .form-label {
        @apply block text-sm font-medium text-gray-700 mb-1;
    }
}

/* Utilitaires custom */
@layer utilities {
    .text-balance {
        text-wrap: balance;
    }
    
    .scrollbar-hide {
        -ms-overflow-style: none;
        scrollbar-width: none;
    }
    
    .scrollbar-hide::-webkit-scrollbar {
        display: none;
    }
}
```

---

## Plugins Tailwind

### @tailwindcss/forms

```bash
npm install -D @tailwindcss/forms
```

```javascript
// tailwind.config.js
plugins: [require('@tailwindcss/forms')]
```

```html
<!-- Reset et styles de base pour tous les inputs -->
<input type="text" class="mt-1 block w-full rounded-md border-gray-300 
                          shadow-sm focus:border-blue-500 focus:ring-blue-500">
<select class="mt-1 block w-full rounded-md border-gray-300">
<textarea class="mt-1 block w-full rounded-md border-gray-300">
<input type="checkbox" class="h-4 w-4 rounded text-blue-600">
```

### @tailwindcss/typography

Applique de beaux styles typographiques au contenu HTML généré (Markdown, CMS).

```bash
npm install -D @tailwindcss/typography
```

```html
<!-- Classe prose pour tout le contenu à l'intérieur -->
<article class="prose prose-lg prose-gray max-w-none dark:prose-invert">
    <h1>Titre généré par un CMS</h1>
    <p>Lorem ipsum...</p>
    <ul>
        <li>Item 1</li>
    </ul>
    <pre><code>du code</code></pre>
</article>
```

### Plugin custom

```javascript
// tailwind.config.js
const plugin = require('tailwindcss/plugin')

module.exports = {
    plugins: [
        plugin(function({ addUtilities, addComponents, theme }) {
            addUtilities({
                '.text-shadow': {
                    textShadow: '0 2px 4px rgba(0,0,0,0.10)',
                },
                '.text-shadow-md': {
                    textShadow: '0 4px 8px rgba(0,0,0,0.12)',
                },
            })
            
            addComponents({
                '.badge': {
                    display: 'inline-flex',
                    alignItems: 'center',
                    padding: `${theme('spacing.1')} ${theme('spacing.2')}`,
                    borderRadius: theme('borderRadius.full'),
                    fontSize: theme('fontSize.xs')[0],
                    fontWeight: theme('fontWeight.medium'),
                },
            })
        }),
    ],
}
```

---

## Tailwind v4

Tailwind CSS v4 introduit une **nouvelle architecture** basée sur CSS natif.

### Changements majeurs

```css
/* v4 — Configuration dans CSS, plus de tailwind.config.js */
@import "tailwindcss";

/* @theme remplace theme.extend */
@theme {
    --color-brand: #2563eb;
    --font-display: "Playfair Display";
    --spacing-18: 4.5rem;
    --breakpoint-xs: 475px;
}

/* Les variables CSS sont disponibles directement */
.hero {
    color: var(--color-brand);
}
```

```javascript
// vite.config.js (Tailwind v4 + Vite)
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
    plugins: [tailwindcss()],
})
```

### Migration v3 → v4

| v3 | v4 |
|----|-----|
| `tailwind.config.js` | `@theme {}` dans CSS |
| `content: [...]` | Auto-détection des fichiers |
| `darkMode: 'class'` | `@variant dark (.dark &);` |
| `require('@tailwindcss/forms')` | `@plugin "@tailwindcss/forms"` |
| `theme.extend.colors` | Variables CSS dans `@theme` |

---

## Intégration avec les frameworks

### Tailwind + React : cn() utility

```bash
npm install clsx tailwind-merge
```

```typescript
// lib/utils.ts — Pattern universel
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
    return twMerge(clsx(inputs))
}
```

```tsx
// components/Button.tsx
import { cn } from '@/lib/utils'

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
    variant?: 'primary' | 'secondary' | 'danger'
    size?: 'sm' | 'md' | 'lg'
}

export function Button({
    variant = 'primary',
    size = 'md',
    className,
    children,
    ...props
}: ButtonProps) {
    return (
        <button
            className={cn(
                // Base styles
                'inline-flex items-center justify-center rounded-lg font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2',
                // Variants
                {
                    'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500': variant === 'primary',
                    'bg-gray-100 text-gray-900 hover:bg-gray-200 focus:ring-gray-500': variant === 'secondary',
                    'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500': variant === 'danger',
                },
                // Sizes
                {
                    'px-3 py-1.5 text-sm': size === 'sm',
                    'px-4 py-2 text-sm': size === 'md',
                    'px-6 py-3 text-base': size === 'lg',
                },
                // Custom classes override
                className
            )}
            {...props}
        >
            {children}
        </button>
    )
}
```

### shadcn/ui

shadcn/ui est une collection de composants **copiables** (pas une dépendance NPM) construits sur Tailwind + Radix UI.

```bash
npx shadcn-ui@latest init
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add dialog
```

```tsx
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"

export function MonComposant() {
    return (
        <Card>
            <CardHeader>
                <CardTitle>Titre de la carte</CardTitle>
            </CardHeader>
            <CardContent>
                <p>Contenu</p>
                <Button variant="outline">Action</Button>
            </CardContent>
        </Card>
    )
}
```

---

## Best Practices

### Ordre des classes (Prettier plugin)

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

```json
// .prettierrc
{
    "plugins": ["prettier-plugin-tailwindcss"]
}
```

Prettier réordera automatiquement les classes Tailwind selon l'ordre officiel.

### Extraction de composants

```html
<!-- ❌ Répétition — extraire en composant -->
<button class="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded">
<button class="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded">

<!-- ✅ Composant React/Vue/Svelte, ou @apply CSS -->
<Button variant="primary">Envoyer</Button>
```

### Éviter le bloat

```html
<!-- ❌ Ne pas utiliser des classes Tailwind construites dynamiquement -->
<!-- Ces classes NE SERONT PAS incluses dans le build (PurgeCSS ne les voit pas) -->
<div class={`text-${color}-500`}>  <!-- ❌ -->

<!-- ✅ Utiliser des classes complètes dans le code -->
<div class={color === 'red' ? 'text-red-500' : 'text-blue-500'}>  <!-- ✅ -->
```

> [!warning] PurgeCSS et classes dynamiques
> Tailwind analyse statiquement vos fichiers pour purger le CSS inutilisé. Il ne peut pas détecter les classes construites par concaténation de chaînes. Toujours écrire les noms de classes **complets** dans le code.

---

## Exercices Pratiques

### Exercice 1 — Navbar responsive

Créez une navbar avec :
- Logo à gauche, liens au centre (cachés sur mobile), bouton CTA à droite
- Menu hamburger sur mobile avec toggle JS
- Active state sur le lien courant
- Fond blanc avec shadow-sm, sticky en haut

```html
<!-- Squelette à compléter -->
<nav class="...">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex items-center justify-between h-16">
            <!-- Logo, liens, bouton -->
        </div>
    </div>
</nav>
```

### Exercice 2 — Card de produit

Créez une card e-commerce avec :
- Image (aspect-ratio 4/3), badge "Nouveau" en overlay
- Titre, description tronquée (2 lignes max), prix, note étoiles
- Bouton "Ajouter au panier" full-width
- Hover effect : lift (translate-y + shadow)

### Exercice 3 — Formulaire de contact

Créez un formulaire complet avec :
- Champs : prénom/nom (2 colonnes), email, téléphone (optionnel), sujet (select), message (textarea)
- Validation visible (border rouge + message d'erreur) avec les variants `invalid:`
- Labels flottants ou labels au-dessus
- Bouton submit avec loading state (animate-spin)

### Exercice 4 — Dashboard Grid

Créez une page dashboard avec :
- Sidebar fixe à gauche (240px) sur desktop, cachée sur mobile
- Header sticky avec breadcrumb + avatar
- Grid de KPI cards (4 colonnes sur desktop, 2 sur tablet, 1 sur mobile)
- Table responsive avec `overflow-x-auto`
- Dark mode toggle fonctionnel

### Exercice 5 — Modale d'alerte

Créez une modale accessible avec :
- Backdrop semi-transparent (`bg-black/50`)
- Modale centrée avec animation fade-in
- En-tête, contenu, footer avec boutons Confirmer/Annuler
- Fermeture au clic backdrop et touche Escape (JS)
- Responsive (plein écran sur mobile)

---

## Liens et Références

- [[02 - CSS Fondamentaux]] — Bases CSS avant Tailwind
- [[03 - CSS Avance]] — Grid, Flexbox, animations CSS native
- [[01 - Introduction a React]] — Tailwind + React en pratique
- [[01 - Introduction a VueJS]] — Tailwind + Vue en pratique
