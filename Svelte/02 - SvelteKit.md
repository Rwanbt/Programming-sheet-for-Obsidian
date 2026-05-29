# SvelteKit

## Qu'est-ce que SvelteKit ?

> [!tip] Analogie
> Tu connais Svelte : c'est le moteur d'une voiture. SvelteKit, c'est la voiture entiere : routage, serveur, deploiement, optimisations. Svelte seul te donne des composants reactifs. SvelteKit te donne une application complete prete a etre mise en production.

SvelteKit est le **metaframework officiel** de Svelte. Il prend en charge tout ce que Svelte ne gere pas :

- **Routage base sur les fichiers** : un fichier = une route, zero configuration
- **Server-Side Rendering (SSR)** : le HTML est genere cote serveur pour le premier chargement
- **Static Site Generation (SSG)** : les pages sont pre-rendues au moment du build
- **API Routes** : creer des endpoints HTTP directement dans le projet
- **Form Actions** : soumettre des formulaires avec ou sans JavaScript
- **Adapters** : deployer partout (Vercel, Netlify, Node.js, Cloudflare, etc.)

> [!info] SvelteKit vs Next.js vs Nuxt
> SvelteKit est a Svelte ce que Next.js est a React ou Nuxt est a Vue. La philosophie est la meme : prendre un framework UI et ajouter tout ce qu'il faut pour construire une vraie application web.

### Le probleme que SvelteKit resout

Avec Svelte seul, tu peux creer des composants mais :
- Comment gerer plusieurs pages et la navigation ?
- Comment faire du rendu serveur pour le SEO ?
- Comment creer une API ?
- Comment deployer ?

SvelteKit repond a toutes ces questions avec une approche coherente et integree.

### Modes de rendu disponibles

```
┌─────────────────────────────────────────────────────────────┐
│                    MODES DE RENDU                           │
├─────────────────┬───────────────────┬───────────────────────┤
│      SSR        │       SSG         │         SPA           │
│ Server-Side     │ Static Site       │ Single Page           │
│ Rendering       │ Generation        │ Application           │
├─────────────────┼───────────────────┼───────────────────────┤
│ Rendu a chaque  │ Rendu au build,   │ Rendu uniquement      │
│ requete         │ fichiers statiques│ cote client           │
├─────────────────┼───────────────────┼───────────────────────┤
│ Donnees fresh   │ Tres rapide       │ Interactivite max     │
│ SEO parfait     │ CDN-friendly      │ Pas de serveur        │
│ Serveur requis  │ Pas de serveur    │ SEO difficile         │
└─────────────────┴───────────────────┴───────────────────────┘
```

SvelteKit peut faire les trois, parfois sur la meme application selon les pages.

---

## Installation et structure du projet

### Creer un nouveau projet

```bash
npm create svelte@latest mon-projet
cd mon-projet
npm install
npm run dev
```

L'assistant interactif te propose plusieurs options :

| Option | Description |
|--------|-------------|
| Skeleton project | Structure minimale |
| SvelteKit demo app | Avec exemples |
| Library project | Pour creer une librairie |
| TypeScript | Recommande |
| ESLint / Prettier | Qualite de code |

### Structure des fichiers generee

```
mon-projet/
├── src/
│   ├── app.html              ← Template HTML global
│   ├── app.css               ← Styles globaux (optionnel)
│   ├── hooks.server.ts       ← Hooks serveur
│   ├── lib/                  ← Code partage ($lib)
│   │   ├── components/       ← Composants reutilisables
│   │   ├── server/           ← Code serveur uniquement
│   │   └── index.ts          ← Exports de la librairie
│   └── routes/               ← Toutes les routes
│       ├── +page.svelte      ← Page d'accueil (/)
│       ├── +layout.svelte    ← Layout global
│       └── about/
│           └── +page.svelte  ← Page /about
├── static/                   ← Fichiers statiques (favicon, images)
├── svelte.config.js          ← Configuration SvelteKit
├── vite.config.ts            ← Configuration Vite
└── package.json
```

### Le fichier app.html

C'est le squelette HTML dans lequel SvelteKit injecte ton application :

```html
<!doctype html>
<html lang="fr">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%sveltekit.assets%/favicon.png" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    %sveltekit.head%
  </head>
  <body data-sveltekit-preload-data="hover">
    <div style="display: contents">%sveltekit.body%</div>
  </body>
</html>
```

Les marqueurs speciaux :
- `%sveltekit.head%` : balises `<link>`, `<meta>`, `<title>` injectees par les pages
- `%sveltekit.body%` : le contenu HTML rendu
- `%sveltekit.assets%` : chemin vers le dossier `static/`

### L'alias $lib

Le dossier `src/lib/` est accessible via l'alias `$lib` partout dans le projet :

```typescript
// Import depuis n'importe quelle route
import { formatDate } from '$lib/utils/date';
import Button from '$lib/components/Button.svelte';
import { db } from '$lib/server/database'; // serveur uniquement
```

> [!warning] $lib/server est reserve au serveur
> Tout ce qui est dans `src/lib/server/` ne peut etre importe que dans des fichiers serveur (`+page.server.ts`, `+server.ts`, `hooks.server.ts`). SvelteKit bloque automatiquement les imports depuis le client.

### svelte.config.js

```javascript
import adapter from '@sveltejs/adapter-auto';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
  preprocess: vitePreprocess(),
  kit: {
    adapter: adapter(),
    // Options supplementaires
    alias: {
      '@components': 'src/lib/components'
    }
  }
};

export default config;
```

---

## File-based Routing

### Principe fondamental

SvelteKit genere les routes a partir de la structure du dossier `src/routes/`. Chaque dossier correspond a un segment d'URL.

```
src/routes/
├── +page.svelte          → /
├── about/
│   └── +page.svelte      → /about
├── blog/
│   ├── +page.svelte      → /blog
│   └── [slug]/
│       └── +page.svelte  → /blog/mon-article
└── admin/
    ├── +layout.svelte    → Layout pour /admin/*
    ├── +page.svelte      → /admin
    └── users/
        └── +page.svelte  → /admin/users
```

### Les fichiers speciaux

| Fichier | Role |
|---------|------|
| `+page.svelte` | Composant de la page |
| `+page.ts` | Load function universelle (client + serveur) |
| `+page.server.ts` | Load function serveur uniquement |
| `+layout.svelte` | Layout partagé par les pages enfants |
| `+layout.ts` | Load function du layout (universelle) |
| `+layout.server.ts` | Load function du layout (serveur) |
| `+error.svelte` | Page d'erreur personnalisee |
| `+server.ts` | API route (endpoints HTTP) |

### Routes dynamiques

Les segments entre crochets `[param]` sont dynamiques :

```
src/routes/
├── blog/
│   └── [slug]/
│       └── +page.svelte    → /blog/hello-world, /blog/autre-article
├── shop/
│   └── [category]/
│       └── [id]/
│           └── +page.svelte → /shop/velos/42
└── [...path]/
    └── +page.svelte         → Catch-all : /tout/ce/que/tu/veux
```

Acceder aux parametres dans la page :

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script>
  export let data; // recoit les donnees de la load function
</script>

<h1>{data.article.title}</h1>
```

### Groupes de routes (route groups)

Les dossiers entre parentheses `(groupe)` n'affectent pas l'URL mais permettent de partager des layouts :

```
src/routes/
├── (marketing)/
│   ├── +layout.svelte      ← Layout avec header marketing
│   ├── +page.svelte        → /
│   └── about/
│       └── +page.svelte    → /about
└── (app)/
    ├── +layout.svelte      ← Layout avec sidebar app
    ├── dashboard/
    │   └── +page.svelte    → /dashboard
    └── settings/
        └── +page.svelte    → /settings
```

> [!example] Cas d'usage
> Un site avec une partie publique (landing page, blog) et une partie application (dashboard, profil) peut avoir deux layouts completement differents sans que ca se voie dans les URLs.

### Layouts imbriques

Les layouts se combinent automatiquement. Un layout enfant s'insere dans le `<slot>` du layout parent :

```svelte
<!-- src/routes/+layout.svelte (layout racine) -->
<script>
  import Nav from '$lib/components/Nav.svelte';
</script>

<Nav />
<main>
  <slot />  <!-- Les layouts/pages enfants s'inserent ici -->
</main>
<footer>Mon footer</footer>
```

```svelte
<!-- src/routes/blog/+layout.svelte (layout blog) -->
<div class="blog-container">
  <aside>Table des matieres</aside>
  <article>
    <slot />  <!-- Les pages blog s'inserent ici -->
  </article>
</div>
```

### La page d'erreur +error.svelte

```svelte
<!-- src/routes/+error.svelte -->
<script>
  import { page } from '$app/stores';
</script>

<h1>{$page.status} - {$page.error?.message}</h1>

{#if $page.status === 404}
  <p>Cette page n'existe pas.</p>
  <a href="/">Retour a l'accueil</a>
{:else}
  <p>Une erreur s'est produite.</p>
{/if}
```

### Matcher de parametres (params)

Pour valider un parametre de route, creer un fichier dans `src/params/` :

```typescript
// src/params/integer.ts
import type { ParamMatcher } from '@sveltejs/kit';

export const match: ParamMatcher = (param) => {
  return /^\d+$/.test(param);
};
```

Utiliser dans le nom du dossier : `[id=integer]` n'accepte que des entiers.

```
src/routes/
└── users/
    └── [id=integer]/
        └── +page.svelte   → /users/42 ✓ | /users/abc ✗ (404)
```

---

## Load Functions

Les load functions fournissent des donnees aux pages et layouts avant qu'ils ne s'affichent. C'est le coeur de la gestion des donnees dans SvelteKit.

### +page.ts : load function universelle

Une load function universelle s'execute **cote serveur lors du SSR** puis **cote client lors de la navigation** :

```typescript
// src/routes/blog/[slug]/+page.ts
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ params, fetch, url }) => {
  const response = await fetch(`/api/articles/${params.slug}`);

  if (!response.ok) {
    // Lancer une erreur HTTP
    throw error(response.status, 'Article introuvable');
  }

  const article = await response.json();

  return {
    article,
    // Tout ce qui est retourne est disponible dans data
    title: article.title
  };
};
```

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script lang="ts">
  import type { PageData } from './$types';

  export let data: PageData;
  // data.article, data.title sont disponibles
</script>

<svelte:head>
  <title>{data.title}</title>
</svelte:head>

<article>
  <h1>{data.article.title}</h1>
  <p>{data.article.content}</p>
</article>
```

### +page.server.ts : load function serveur uniquement

Une load function serveur s'execute **uniquement cote serveur**, meme lors de la navigation client. Ideale pour :
- Acceder directement a la base de donnees
- Utiliser des secrets (cles API)
- Lire des cookies
- Valider des sessions

```typescript
// src/routes/dashboard/+page.server.ts
import type { PageServerLoad } from './$types';
import { redirect } from '@sveltejs/kit';
import { db } from '$lib/server/database';

export const load: PageServerLoad = async ({ locals, cookies }) => {
  // locals est peuple par les hooks (ex: session utilisateur)
  if (!locals.user) {
    throw redirect(302, '/login');
  }

  // Acces direct a la BDD - impossible dans +page.ts
  const projects = await db.project.findMany({
    where: { userId: locals.user.id }
  });

  return {
    user: locals.user,
    projects
  };
};
```

> [!warning] Ne jamais exposer des secrets dans +page.ts
> Tout ce qui est dans `+page.ts` peut etre execute dans le navigateur. Une cle API dans une load function universelle sera visible dans le bundle JavaScript. Utilise `+page.server.ts` pour tout ce qui est sensible.

### Comparaison des deux types

```
+page.ts (universelle)               +page.server.ts (serveur)
─────────────────────                ─────────────────────────
✓ Executes cote serveur (SSR)        ✓ Executes cote serveur uniquement
✓ Executes cote client (navigation)  ✓ Acces BDD directe
✓ fetch() disponible                 ✓ Secrets/cookies/sessions
✗ Pas d'acces BDD directe           ✓ locals disponibles
✗ Pas de secrets                     ~ Serialise les donnees via HTTP
```

### Parametres disponibles dans load

```typescript
export const load: PageLoad = async ({
  params,    // Parametres de route { slug: 'hello-world' }
  url,       // URL courante (URLSearchParams, pathname...)
  fetch,     // fetch() patche par SvelteKit (supporte cookies, URLs relatives)
  parent,    // Donnees du layout parent (await parent())
  depends,   // Declarer des dependances pour l'invalidation
  setHeaders  // Definir des headers HTTP (cache-control, etc.)
}) => { ... };
```

```typescript
export const load: PageServerLoad = async ({
  params,
  url,
  fetch,
  parent,
  depends,
  setHeaders,
  // En plus, disponibles uniquement en server :
  locals,    // Donnees injectees par les hooks (user, session)
  cookies,   // Lire/ecrire des cookies
  request    // L'objet Request complet
}) => { ... };
```

### Gestion des erreurs dans les load functions

```typescript
import { error, redirect } from '@sveltejs/kit';

export const load: PageServerLoad = async ({ params }) => {
  const article = await db.article.findUnique({ where: { slug: params.slug } });

  if (!article) {
    // Declenche +error.svelte avec status 404
    throw error(404, {
      message: 'Article introuvable',
      code: 'ARTICLE_NOT_FOUND'
    });
  }

  if (!article.published) {
    // Redirection HTTP 302
    throw redirect(302, '/blog');
  }

  return { article };
};
```

### Donnees du layout parent

Les layouts peuvent aussi avoir des load functions. Les pages enfants heritent de ces donnees :

```typescript
// src/routes/+layout.server.ts
export const load: LayoutServerLoad = async ({ locals }) => {
  return {
    user: locals.user ?? null
  };
};
```

```typescript
// src/routes/blog/[slug]/+page.server.ts
export const load: PageServerLoad = async ({ parent, params }) => {
  const { user } = await parent(); // Donnees du layout parent
  const article = await db.article.findUnique({ where: { slug: params.slug } });
  return { article, user };
};
```

> [!info] Cascade des donnees
> Les donnees du layout sont fusionnees avec celles de la page. Une page peut acceder aux donnees de tous ses layouts parents via `data` dans le composant Svelte.

---

## Form Actions

Les form actions permettent de traiter des soumissions de formulaires **cote serveur**, avec une degradation gracieuse pour les utilisateurs sans JavaScript.

### Action simple

```typescript
// src/routes/contact/+page.server.ts
import type { Actions, PageServerLoad } from './$types';
import { fail } from '@sveltejs/kit';

export const load: PageServerLoad = async () => {
  return { sent: false };
};

export const actions: Actions = {
  // Action par defaut (appellee sans ?/action=nom)
  default: async ({ request }) => {
    const data = await request.formData();
    const email = data.get('email') as string;
    const message = data.get('message') as string;

    // Validation serveur
    if (!email || !email.includes('@')) {
      return fail(422, {
        email,
        message,
        error: 'Email invalide'
      });
    }

    if (!message || message.length < 10) {
      return fail(422, {
        email,
        message,
        error: 'Message trop court (min 10 caracteres)'
      });
    }

    // Traitement (envoyer un email, sauvegarder, etc.)
    await sendEmail({ email, message });

    return { success: true };
  }
};
```

```svelte
<!-- src/routes/contact/+page.svelte -->
<script lang="ts">
  import type { PageData, ActionData } from './$types';

  export let data: PageData;
  export let form: ActionData; // Resultat de l'action
</script>

{#if form?.success}
  <p class="success">Message envoye !</p>
{/if}

<form method="POST">
  <div>
    <label for="email">Email</label>
    <input
      id="email"
      name="email"
      type="email"
      value={form?.email ?? ''}
    />
    {#if form?.error && form.error.includes('Email')}
      <span class="error">{form.error}</span>
    {/if}
  </div>

  <div>
    <label for="message">Message</label>
    <textarea id="message" name="message">{form?.message ?? ''}</textarea>
  </div>

  <button type="submit">Envoyer</button>
</form>
```

### Actions nommees

Un meme fichier peut avoir plusieurs actions :

```typescript
// src/routes/todos/+page.server.ts
export const actions: Actions = {
  create: async ({ request }) => {
    const data = await request.formData();
    const text = data.get('text') as string;
    if (!text) return fail(400, { error: 'Texte requis' });
    await db.todo.create({ data: { text } });
  },

  delete: async ({ request }) => {
    const data = await request.formData();
    const id = data.get('id') as string;
    await db.todo.delete({ where: { id } });
  },

  toggle: async ({ request }) => {
    const data = await request.formData();
    const id = data.get('id') as string;
    const todo = await db.todo.findUnique({ where: { id } });
    await db.todo.update({
      where: { id },
      data: { done: !todo?.done }
    });
  }
};
```

```svelte
<!-- Appeler une action nommee avec ?/nom_action -->
<form method="POST" action="?/create">
  <input name="text" />
  <button type="submit">Ajouter</button>
</form>

{#each todos as todo}
  <form method="POST" action="?/toggle">
    <input type="hidden" name="id" value={todo.id} />
    <button type="submit">{todo.done ? '✓' : '○'} {todo.text}</button>
  </form>

  <form method="POST" action="?/delete">
    <input type="hidden" name="id" value={todo.id} />
    <button type="submit">Supprimer</button>
  </form>
{/each}
```

### Progressive Enhancement avec use:enhance

Sans JavaScript, les formulaires fonctionnent (soumission HTML classique). Avec `use:enhance`, on ajoute du comportement client enrichi :

```svelte
<script lang="ts">
  import { enhance } from '$app/forms';
  import type { SubmitFunction } from '@sveltejs/kit';

  let loading = false;

  const handleSubmit: SubmitFunction = ({ formElement, formData, action, cancel }) => {
    loading = true;

    return async ({ result, update }) => {
      loading = false;

      if (result.type === 'success') {
        // Ne pas reset le formulaire automatiquement
        await update({ reset: false });
      } else {
        await update();
      }
    };
  };
</script>

<form method="POST" use:enhance={handleSubmit}>
  <input name="email" type="email" />
  <button type="submit" disabled={loading}>
    {loading ? 'Envoi...' : 'Envoyer'}
  </button>
</form>
```

> [!info] Progressive Enhancement
> Sans JavaScript : le formulaire fonctionne avec un rechargement de page classique. Avec JavaScript et `use:enhance` : navigation sans rechargement, feedback immediat, gestion fine des etats. Le meme code serveur gere les deux cas.

---

## API Routes (+server.ts)

Les fichiers `+server.ts` creent des endpoints HTTP purs, sans composant Svelte.

### Structure de base

```typescript
// src/routes/api/users/+server.ts
import type { RequestHandler } from '@sveltejs/kit';
import { json, error } from '@sveltejs/kit';

export const GET: RequestHandler = async ({ url, locals }) => {
  const page = Number(url.searchParams.get('page') ?? '1');
  const limit = Number(url.searchParams.get('limit') ?? '10');

  const users = await db.user.findMany({
    skip: (page - 1) * limit,
    take: limit
  });

  return json(users, {
    headers: {
      'Cache-Control': 'private, max-age=60'
    }
  });
};

export const POST: RequestHandler = async ({ request, locals }) => {
  if (!locals.user?.isAdmin) {
    throw error(403, 'Forbidden');
  }

  const body = await request.json();

  // Validation
  if (!body.name || !body.email) {
    throw error(400, 'name et email sont requis');
  }

  const user = await db.user.create({
    data: { name: body.name, email: body.email }
  });

  return json(user, { status: 201 });
};
```

### Tous les verbes HTTP

```typescript
// src/routes/api/articles/[id]/+server.ts
export const GET: RequestHandler = async ({ params }) => { ... };
export const PUT: RequestHandler = async ({ params, request }) => { ... };
export const PATCH: RequestHandler = async ({ params, request }) => { ... };
export const DELETE: RequestHandler = async ({ params }) => { ... };

// Fallback pour tous les verbes non geres
export const fallback: RequestHandler = async ({ request }) => {
  return new Response(`Methode ${request.method} non supportee`, { status: 405 });
};
```

### Reponses personnalisees

```typescript
export const GET: RequestHandler = async ({ params }) => {
  // json() est un helper
  return json({ status: 'ok' });

  // Equivalent a :
  return new Response(JSON.stringify({ status: 'ok' }), {
    headers: { 'Content-Type': 'application/json' }
  });
};

// Retourner du texte brut
export const GET: RequestHandler = async () => {
  return new Response('pong', {
    headers: { 'Content-Type': 'text/plain' }
  });
};

// Retourner un fichier
export const GET: RequestHandler = async ({ params }) => {
  const file = await readFile(`./uploads/${params.name}`);
  return new Response(file, {
    headers: { 'Content-Type': 'application/octet-stream' }
  });
};
```

### Streaming (Server-Sent Events)

```typescript
// src/routes/api/stream/+server.ts
export const GET: RequestHandler = async () => {
  const stream = new ReadableStream({
    async start(controller) {
      for (let i = 0; i < 10; i++) {
        controller.enqueue(`data: ${JSON.stringify({ count: i })}\n\n`);
        await new Promise(resolve => setTimeout(resolve, 1000));
      }
      controller.close();
    }
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache'
    }
  });
};
```

> [!example] Quand utiliser les API routes ?
> - Webhook receivers (Stripe, GitHub, etc.)
> - Endpoints consommes par une app mobile
> - Telechargement de fichiers generes
> - Server-Sent Events pour le temps reel
> - Authentification OAuth (callback URLs)
>
> Pour tout ce qui est rendu HTML, preferer les load functions + form actions qui sont mieux integrees.

---

## SSR vs SSG vs SPA

### Comprendre les trois modes

```
SSR (Server-Side Rendering)
┌──────────┐  requete   ┌──────────┐  HTML rendu  ┌──────────┐
│ Browser  │──────────→ │ Serveur  │─────────────→ │ Browser  │
└──────────┘            │ (SvelteKit)│              └──────────┘
                        └──────────┘
Avantages : donnees fresh, SEO parfait, pas de flash de contenu
Inconvenients : serveur requis, latence par requete


SSG (Static Site Generation)
┌──────────┐  build   ┌──────────┐  fichiers HTML  ┌──────────┐
│  Code    │────────→ │  Vite    │────────────────→ │   CDN    │
└──────────┘          └──────────┘                  └──────────┘
Avantages : ultra-rapide, CDN-friendly, zero serveur
Inconvenients : donnees figees au build


SPA (Single Page Application)
┌──────────┐           ┌──────────┐  JS Bundle  ┌──────────┐
│  Code    │──build──→ │  Vite    │────────────→ │ Browser  │
└──────────┘           └──────────┘              │ (rendu)  │
                                                 └──────────┘
Avantages : interactivite max, zero serveur
Inconvenients : SEO difficile, flash de contenu vide
```

### Configurer le mode par page

```typescript
// src/routes/blog/[slug]/+page.ts

// Activer le pre-rendu (SSG) pour cette page
export const prerender = true;

// Desactiver le SSR (mode SPA pour cette page)
export const ssr = false;

// Desactiver l'hydratation JavaScript
export const csr = false; // Pure HTML statique, zero JS
```

### Pre-rendu avec entrees dynamiques

Pour SSG avec des routes dynamiques, SvelteKit a besoin de connaitre toutes les URLs :

```typescript
// src/routes/blog/[slug]/+page.server.ts
export const prerender = true;

export const load: PageServerLoad = async ({ params }) => {
  const article = await db.article.findUnique({ where: { slug: params.slug } });
  if (!article) throw error(404);
  return { article };
};
```

```typescript
// src/routes/blog/[slug]/+page.ts
// Methode 1 : laisser SvelteKit crawler les liens
export const prerender = true;

// Methode 2 : declarer les entrees explicitement
export const entries = async () => {
  const articles = await db.article.findMany({ select: { slug: true } });
  return articles.map(a => ({ slug: a.slug }));
};
```

### Tableau de decision : quel mode choisir ?

| Critere | SSR | SSG | SPA |
|---------|-----|-----|-----|
| SEO important | ✅ | ✅ | ❌ |
| Donnees en temps reel | ✅ | ❌ | ✅ (avec fetch) |
| Serveur requis | ✅ | ❌ | ❌ |
| Vitesse de chargement | Bonne | Excellente | Moyenne |
| Blog / Marketing | Possible | Ideal | ❌ |
| Dashboard prive | Possible | ❌ | Possible |
| E-commerce | Ideal | Partiel | ❌ |
| App documentaire | Possible | Ideal | ❌ |

### Mixte : SSG global + quelques pages SSR

```typescript
// src/routes/+layout.ts
// SSG par defaut pour tout le site
export const prerender = true;
```

```typescript
// src/routes/dashboard/+page.ts
// Override : cette page est SSR (donnees utilisateur)
export const prerender = false;
export const ssr = true;
```

> [!tip] Strategie recommandee
> Commence avec SSR (defaut de SvelteKit). Active SSG page par page quand le contenu est statique. Reserve SPA aux pages qui n'ont vraiment pas besoin de SEO et dont les donnees viennent uniquement du client (espace personnel, editeur, etc.).

---

## Hooks

Les hooks sont des fonctions qui s'executent pour chaque requete, avant que le routeur ne soit appele. Ils permettent d'intercepter, modifier ou enrichir les requetes et reponses.

### hooks.server.ts

```typescript
// src/hooks.server.ts
import type { Handle, HandleError, HandleFetch } from '@sveltejs/kit';
import { db } from '$lib/server/database';

// Hook principal : intercepte chaque requete
export const handle: Handle = async ({ event, resolve }) => {
  // event.locals est vide par defaut - on peut y mettre ce qu'on veut
  const sessionToken = event.cookies.get('session');

  if (sessionToken) {
    const session = await db.session.findUnique({
      where: { token: sessionToken },
      include: { user: true }
    });

    if (session && session.expiresAt > new Date()) {
      event.locals.user = session.user;
      event.locals.session = session;
    }
  }

  // Continuer le traitement de la requete
  const response = await resolve(event);

  // On peut aussi modifier la reponse
  response.headers.set('X-Custom-Header', 'SvelteKit');

  return response;
};
```

### Chainer plusieurs hooks avec sequence

```typescript
import { sequence } from '@sveltejs/kit/hooks';

const authHandler: Handle = async ({ event, resolve }) => {
  // Logique d'auth
  const token = event.cookies.get('session');
  if (token) {
    event.locals.user = await getUserFromToken(token);
  }
  return resolve(event);
};

const loggingHandler: Handle = async ({ event, resolve }) => {
  const start = Date.now();
  const response = await resolve(event);
  console.log(`${event.request.method} ${event.url.pathname} - ${Date.now() - start}ms`);
  return response;
};

const corsHandler: Handle = async ({ event, resolve }) => {
  if (event.request.method === 'OPTIONS') {
    return new Response(null, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type'
      }
    });
  }

  const response = await resolve(event);
  response.headers.set('Access-Control-Allow-Origin', '*');
  return response;
};

// Les hooks s'executent dans l'ordre
export const handle = sequence(authHandler, loggingHandler, corsHandler);
```

### handleError : capturer les erreurs non gerees

```typescript
import type { HandleError } from '@sveltejs/kit';

export const handleError: HandleError = async ({ error, event, status, message }) => {
  // Logger l'erreur (Sentry, Datadog, etc.)
  console.error(`Erreur ${status} sur ${event.url.pathname}:`, error);

  // Retourner un message safe a afficher a l'utilisateur
  // Ne jamais exposer les details internes d'une erreur
  return {
    message: status === 404 ? 'Page introuvable' : 'Une erreur est survenue',
    code: (error as any)?.code ?? 'UNKNOWN'
  };
};
```

### handleFetch : modifier les requetes fetch

```typescript
import type { HandleFetch } from '@sveltejs/kit';

export const handleFetch: HandleFetch = async ({ request, fetch }) => {
  // Exemple : ajouter un header d'auth a toutes les requetes fetch cote serveur
  if (request.url.startsWith('https://api.example.com/')) {
    const modifiedRequest = new Request(request, {
      headers: {
        ...Object.fromEntries(request.headers),
        'Authorization': `Bearer ${process.env.API_SECRET}`
      }
    });
    return fetch(modifiedRequest);
  }

  return fetch(request);
};
```

### Le type App.Locals

Pour avoir l'autocompletion sur `event.locals`, declarer les types dans `src/app.d.ts` :

```typescript
// src/app.d.ts
import type { User, Session } from '$lib/server/types';

declare global {
  namespace App {
    interface Locals {
      user: User | null;
      session: Session | null;
    }

    interface PageData {
      user?: User | null;
    }

    interface Error {
      message: string;
      code?: string;
    }
  }
}

export {};
```

---

## Variables d'Environnement

SvelteKit fournit quatre modules pour acceder aux variables d'environnement, selon qu'elles sont statiques/dynamiques et publiques/privees.

### Les quatre modules

```
┌─────────────────────────────────────────────────────────────┐
│              VARIABLES D'ENVIRONNEMENT                      │
├─────────────────────────┬───────────────────────────────────┤
│       STATIQUES         │          DYNAMIQUES               │
│ (integrees au build)    │    (lues au runtime)              │
├─────────────────────────┼───────────────────────────────────┤
│ $env/static/public      │  $env/dynamic/public              │
│ Prefixe : PUBLIC_       │  Toutes variables publiques       │
│ Client + Serveur        │  Client + Serveur                 │
├─────────────────────────┼───────────────────────────────────┤
│ $env/static/private     │  $env/dynamic/private             │
│ Pas de prefixe requis   │  Toutes variables privees         │
│ Serveur uniquement      │  Serveur uniquement               │
└─────────────────────────┴───────────────────────────────────┘
```

### $env/static/public

Variables connues au build, accessibles partout :

```
# .env
PUBLIC_API_URL=https://api.example.com
PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

```svelte
<script>
  import { PUBLIC_API_URL, PUBLIC_STRIPE_PUBLISHABLE_KEY } from '$env/static/public';

  async function fetchData() {
    const res = await fetch(`${PUBLIC_API_URL}/data`);
    return res.json();
  }
</script>
```

### $env/static/private

Variables connues au build, **serveur uniquement** :

```
# .env
DATABASE_URL=postgresql://localhost:5432/myapp
STRIPE_SECRET_KEY=sk_test_...
JWT_SECRET=super_secret_key
```

```typescript
// src/routes/+page.server.ts - OK
import { DATABASE_URL, STRIPE_SECRET_KEY } from '$env/static/private';
```

```typescript
// src/routes/+page.ts - ERREUR de compilation !
// import { DATABASE_URL } from '$env/static/private'; ← bloque
```

### $env/dynamic/private

Pour les variables qui changent selon l'environnement (Docker, K8s) :

```typescript
import { env } from '$env/dynamic/private';

// env est un objet contenant toutes les variables au moment de l'execution
const dbUrl = env.DATABASE_URL;
const port = env.PORT ?? '3000';
```

### $env/dynamic/public

```typescript
import { env } from '$env/dynamic/public';

// Variables publiques disponibles au runtime
const apiUrl = env.PUBLIC_API_URL;
```

> [!warning] Choisir entre statique et dynamique
> - **Statique** : les variables sont inlinees dans le bundle au build. Plus performant. Ideal pour les variables identiques en dev/prod.
> - **Dynamique** : les variables sont lues a l'execution. Necessaire pour les deployments containerises ou la config change sans rebuild (PORT, DATABASE_URL dans Docker).

### Fichiers .env

```
.env                  ← Variables par defaut (commiter si pas de secrets)
.env.local            ← Override local (ne jamais commiter)
.env.development      ← Variables de developpement uniquement
.env.production       ← Variables de production uniquement
```

> [!warning] .gitignore
> Ne jamais commiter `.env.local` ni `.env.production` s'ils contiennent des secrets. Ajouter ces fichiers au `.gitignore`.

---

## Deploiement et Adapters

SvelteKit genere une application qui doit etre adaptee a l'environnement de deploiement via des **adapters**.

### Adapter Auto (defaut)

```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-auto';

export default {
  kit: {
    adapter: adapter()
  }
};
```

`adapter-auto` detecte automatiquement l'environnement et choisit le bon adapter :

| Environnement detecte | Adapter utilise |
|----------------------|-----------------|
| Vercel | adapter-vercel |
| Netlify | adapter-netlify |
| Cloudflare Pages | adapter-cloudflare |
| Azure Static Web Apps | adapter-azure |
| Aucun connu | adapter-node |

### Adapter Node.js

Pour un serveur Node.js classique :

```bash
npm install @sveltejs/adapter-node
```

```javascript
import adapter from '@sveltejs/adapter-node';

export default {
  kit: {
    adapter: adapter({
      out: 'build',      // Dossier de sortie
      precompress: true  // Generer des .gz et .br
    })
  }
};
```

```bash
npm run build
node build/index.js  # Lancer le serveur
```

Avec Docker :

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY build/ ./build/
COPY package.json ./
RUN npm install --production
ENV PORT=3000
EXPOSE 3000
CMD ["node", "build/index.js"]
```

### Adapter Vercel

```bash
npm install @sveltejs/adapter-vercel
```

```javascript
import adapter from '@sveltejs/adapter-vercel';

export default {
  kit: {
    adapter: adapter({
      runtime: 'nodejs20.x'
    })
  }
};
```

Deploiement : `vercel deploy` ou push sur la branche liee.

### Adapter Static (SSG pur)

Pour un site 100% statique :

```bash
npm install @sveltejs/adapter-static
```

```javascript
import adapter from '@sveltejs/adapter-static';

export default {
  kit: {
    adapter: adapter({
      pages: 'build',   // Dossier de sortie
      assets: 'build',
      fallback: null    // 'index.html' pour SPA mode
    })
  }
};
```

```typescript
// src/routes/+layout.ts
// Obligatoire avec adapter-static
export const prerender = true;
export const ssr = false; // Pour SPA avec fallback
```

### Adapter Cloudflare Pages

```bash
npm install @sveltejs/adapter-cloudflare
```

```javascript
import adapter from '@sveltejs/adapter-cloudflare';

export default {
  kit: {
    adapter: adapter({
      routes: {
        include: ['/*'],
        exclude: ['<all>']
      }
    })
  }
};
```

Acces aux bindings Cloudflare (KV, D1, R2) via `platform.env` :

```typescript
export const load: PageServerLoad = async ({ platform }) => {
  const kv = platform?.env?.MY_KV_NAMESPACE;
  const value = await kv?.get('my-key');
  return { value };
};
```

> [!info] Quel adapter choisir ?
> - **Vercel / Netlify** : le plus simple pour commencer, CI/CD integre
> - **Node.js** : VPS, Docker, Kubernetes - controle total
> - **Cloudflare** : performances edge extremes, Workers globaux
> - **Static** : blog, documentation, sites marketing sans backend

---

## Exemple Complet : Blog avec SSG

Voici un exemple concret qui combine tout ce qu'on a vu : un blog statique genere depuis des fichiers Markdown.

### Structure du projet

```
src/
├── lib/
│   ├── types.ts              ← Types partages
│   └── posts.ts              ← Chargement des articles
├── routes/
│   ├── +layout.svelte        ← Layout global
│   ├── +layout.ts            ← SSG global
│   ├── +page.svelte          ← Page d'accueil
│   ├── +page.ts              ← Load function accueil
│   └── blog/
│       ├── +page.svelte      ← Liste des articles
│       ├── +page.ts          ← Load function liste
│       └── [slug]/
│           ├── +page.svelte  ← Article individuel
│           └── +page.ts      ← Load function article
```

### Types partagés

```typescript
// src/lib/types.ts
export interface Post {
  slug: string;
  title: string;
  date: string;
  excerpt: string;
  content: string;
  tags: string[];
  published: boolean;
}
```

### Module de chargement des articles

```typescript
// src/lib/posts.ts
import type { Post } from './types';

// Vite permet d'importer des fichiers Markdown via vite-plugin-md
// ou on peut utiliser des fichiers JSON pour simplifier
const modules = import.meta.glob('/src/content/posts/*.md', {
  eager: true
}) as Record<string, { metadata: Omit<Post, 'slug' | 'content'>; default: { render: () => { html: string } } }>;

export function getAllPosts(): Post[] {
  return Object.entries(modules)
    .map(([filepath, module]) => {
      const slug = filepath.split('/').at(-1)?.replace('.md', '') ?? '';
      const { html } = module.default.render();
      return {
        slug,
        ...module.metadata,
        content: html
      };
    })
    .filter(post => post.published)
    .sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime());
}

export function getPostBySlug(slug: string): Post | undefined {
  return getAllPosts().find(post => post.slug === slug);
}
```

### Layout global avec SSG

```typescript
// src/routes/+layout.ts
// Active le SSG pour tout le site
export const prerender = true;
```

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
  import { page } from '$app/stores';
</script>

<svelte:head>
  <meta name="description" content="Mon blog SvelteKit" />
</svelte:head>

<header>
  <nav>
    <a href="/" class:active={$page.url.pathname === '/'}>Accueil</a>
    <a href="/blog" class:active={$page.url.pathname.startsWith('/blog')}>Blog</a>
  </nav>
</header>

<main>
  <slot />
</main>

<footer>
  <p>© {new Date().getFullYear()} Mon Blog</p>
</footer>

<style>
  header {
    padding: 1rem 2rem;
    background: #1a1a2e;
    color: white;
  }
  nav { display: flex; gap: 1.5rem; }
  a { color: #e0e0e0; text-decoration: none; }
  a.active { color: white; font-weight: bold; }
  main { max-width: 900px; margin: 0 auto; padding: 2rem; }
</style>
```

### Page d'accueil

```typescript
// src/routes/+page.ts
import type { PageLoad } from './$types';
import { getAllPosts } from '$lib/posts';

export const load: PageLoad = () => {
  const recentPosts = getAllPosts().slice(0, 3);
  return { recentPosts };
};
```

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
  import type { PageData } from './$types';
  export let data: PageData;
</script>

<svelte:head>
  <title>Mon Blog SvelteKit</title>
</svelte:head>

<section class="hero">
  <h1>Bienvenue sur mon blog</h1>
  <p>Exploration de SvelteKit et du web moderne.</p>
  <a href="/blog" class="cta">Voir tous les articles →</a>
</section>

<section class="recent">
  <h2>Articles recents</h2>
  <div class="grid">
    {#each data.recentPosts as post}
      <article>
        <time>{post.date}</time>
        <h3><a href="/blog/{post.slug}">{post.title}</a></h3>
        <p>{post.excerpt}</p>
        <div class="tags">
          {#each post.tags as tag}
            <span class="tag">{tag}</span>
          {/each}
        </div>
      </article>
    {/each}
  </div>
</section>
```

### Liste des articles

```typescript
// src/routes/blog/+page.ts
import type { PageLoad } from './$types';
import { getAllPosts } from '$lib/posts';

export const load: PageLoad = ({ url }) => {
  const tag = url.searchParams.get('tag');
  const allPosts = getAllPosts();

  const posts = tag
    ? allPosts.filter(post => post.tags.includes(tag))
    : allPosts;

  const allTags = [...new Set(allPosts.flatMap(p => p.tags))];

  return { posts, allTags, activeTag: tag };
};
```

### Page d'article avec entrees statiques

```typescript
// src/routes/blog/[slug]/+page.ts
import type { PageLoad } from './$types';
import { error } from '@sveltejs/kit';
import { getAllPosts, getPostBySlug } from '$lib/posts';

// Declarer toutes les entrees pour SSG
export const entries = () => {
  return getAllPosts().map(post => ({ slug: post.slug }));
};

export const load: PageLoad = ({ params }) => {
  const post = getPostBySlug(params.slug);

  if (!post) {
    throw error(404, `Article "${params.slug}" introuvable`);
  }

  return { post };
};
```

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script lang="ts">
  import type { PageData } from './$types';
  export let data: PageData;
</script>

<svelte:head>
  <title>{data.post.title}</title>
  <meta name="description" content={data.post.excerpt} />
  <!-- Open Graph -->
  <meta property="og:title" content={data.post.title} />
  <meta property="og:description" content={data.post.excerpt} />
  <meta property="og:type" content="article" />
</svelte:head>

<article class="post">
  <header>
    <div class="tags">
      {#each data.post.tags as tag}
        <a href="/blog?tag={tag}" class="tag">{tag}</a>
      {/each}
    </div>
    <h1>{data.post.title}</h1>
    <time datetime={data.post.date}>
      {new Date(data.post.date).toLocaleDateString('fr-FR', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })}
    </time>
  </header>

  <div class="content">
    {@html data.post.content}
  </div>

  <footer>
    <a href="/blog">← Retour au blog</a>
  </footer>
</article>

<style>
  .post { max-width: 720px; margin: 0 auto; }
  h1 { font-size: 2.5rem; line-height: 1.2; margin-top: 0.5rem; }
  time { color: #666; font-size: 0.9rem; }
  .tag {
    display: inline-block;
    padding: 0.2rem 0.6rem;
    background: #e8f4fd;
    border-radius: 999px;
    font-size: 0.8rem;
    color: #0077cc;
    text-decoration: none;
    margin-right: 0.3rem;
  }
  .content { line-height: 1.8; margin-top: 2rem; }
  .content :global(code) { background: #f4f4f4; padding: 0.1em 0.3em; border-radius: 3px; }
  .content :global(pre) { background: #1a1a2e; color: #e0e0e0; padding: 1rem; border-radius: 6px; overflow-x: auto; }
</style>
```

---

## Stores et Navigation

### Les stores $app

SvelteKit fournit des stores reactifs pour acceder aux informations de navigation :

```svelte
<script>
  import { page, navigating, updated } from '$app/stores';
  import { goto, invalidate, invalidateAll, preloadData } from '$app/navigation';

  // $page : informations sur la page courante
  // $page.url - URLObject
  // $page.params - parametres de route
  // $page.data - donnees de toutes les load functions
  // $page.status - status HTTP
  // $page.error - erreur courante
  // $page.route.id - identifiant de la route

  // $navigating : null si pas de navigation en cours
  // $navigating.from - route de depart
  // $navigating.to - route de destination
</script>

<!-- Indicateur de chargement -->
{#if $navigating}
  <div class="loading-bar" />
{/if}

<!-- Breadcrumb dynamique -->
<nav>
  <a href="/">Accueil</a>
  {#if $page.params.slug}
    <span>/</span>
    <span>{$page.params.slug}</span>
  {/if}
</nav>
```

### Navigation programmatique

```typescript
import { goto, back, forward } from '$app/navigation';

// Naviguer vers une URL
await goto('/dashboard');
await goto('/profil', { replaceState: true }); // Sans entree dans l'historique

// Invalider les donnees d'une load function
await invalidate('app:user'); // Recharge les load functions qui dependent de 'app:user'
await invalidateAll();        // Recharge toutes les load functions

// Precharger des donnees avant navigation
await preloadData('/dashboard'); // Prepare les donnees a l'avance
```

---

## Carte Mentale

```
                        ┌─────────────────────────────┐
                        │          SVELTEKIT          │
                        │    (Metaframework Svelte)   │
                        └──────────────┬──────────────┘
                                       │
          ┌────────────────────────────┼────────────────────────────┐
          │                            │                            │
   ┌──────┴──────┐              ┌──────┴──────┐              ┌──────┴──────┐
   │   ROUTING   │              │    DATA     │              │  RENDERING  │
   │  (fichiers) │              │  (loading)  │              │   (modes)   │
   └──────┬──────┘              └──────┬──────┘              └──────┬──────┘
          │                            │                            │
   ┌──────┴──────┐              ┌──────┴──────┐              ┌──────┴──────┐
   │+page.svelte │        ┌─────┤  +page.ts   │              │    SSR      │
   │+layout.svlt │        │     │ (universel) │              │ (par defaut)│
   │+error.svlt  │        │     └─────────────┘              └─────────────┘
   │+server.ts   │        │     ┌─────────────┐              ┌─────────────┐
   │[param]/     │        └─────┤page.server  │              │    SSG      │
   │(groupes)/   │              │  (server)   │              │ (prerender) │
   │[...catch]   │              └─────────────┘              └─────────────┘
   └─────────────┘              ┌─────────────┐              ┌─────────────┐
                                │   Actions   │              │    SPA      │
   ┌─────────────┐              │  (forms)    │              │ (ssr=false) │
   │   HOOKS     │              └─────────────┘              └─────────────┘
   │             │
   │ handle()    │        ┌─────────────────────────────────────────────┐
   │ handleError │        │              ENVIRONNEMENT                  │
   │ handleFetch │        ├──────────────┬──────────────┬───────────────┤
   │ sequence()  │        │$env/static/  │$env/static/  │$env/dynamic/  │
   └──────┬──────┘        │  public      │  private     │  private      │
          │               │ (build+CDN)  │ (secrets)    │ (runtime)     │
   ┌──────┴──────┐        └──────────────┴──────────────┴───────────────┘
   │   DEPLOY    │
   │             │        ┌─────────────────────────────────────────────┐
   │adapter-auto │        │                STRUCTURE                    │
   │adapter-node │        ├────────────┬──────────────┬─────────────────┤
   │adapter-vercel│       │ src/routes │   src/lib    │    static/      │
   │adapter-static│       │ (routing)  │  ($lib alias)│   (assets)      │
   └─────────────┘        └────────────┴──────────────┴─────────────────┘
```

---

## Exercices Pratiques

### Exercice 1 : Portfolio avec SSG

Cree un portfolio personnel en SSG avec SvelteKit.

**Objectif** : comprendre le routage et le SSG.

**Etapes** :
1. Creer les routes `/`, `/projets`, `/projets/[id]`, `/contact`
2. Ajouter `export const prerender = true` dans `+layout.ts`
3. La page `/projets/[id]` doit avoir une fonction `entries()` qui liste les IDs
4. Le layout global doit avoir une navigation avec indication de la page active via `$page`

**Bonus** : ajouter une page `/contact` avec une form action qui affiche un message de confirmation sans rechargement de page (via `use:enhance`).

---

### Exercice 2 : API REST + SSR

Cree une mini-application de gestion de taches avec :

**API (`src/routes/api/todos/+server.ts`)** :
- `GET /api/todos` : retourne toutes les taches (JSON)
- `POST /api/todos` : cree une tache (body JSON)
- `DELETE /api/todos/[id]` : supprime une tache

**Page (`src/routes/todos/`)** :
- `+page.server.ts` : charge les taches depuis l'API ou la BDD directement
- `+page.svelte` : affiche la liste + formulaire d'ajout
- Form actions : `create`, `delete`, `toggle`

**Contrainte** : la page doit fonctionner sans JavaScript (degradation gracieuse).

---

### Exercice 3 : Authentification avec Hooks

Implemente un systeme d'authentification simple.

**Structure** :
```
src/
├── hooks.server.ts          ← Intercepter les requetes, charger l'utilisateur
├── routes/
│   ├── login/
│   │   ├── +page.svelte     ← Formulaire de connexion
│   │   └── +page.server.ts  ← Action login (verifier mot de passe, creer session)
│   ├── logout/
│   │   └── +server.ts       ← DELETE handler, supprimer le cookie
│   └── dashboard/
│       ├── +page.svelte     ← Page protegee
│       └── +page.server.ts  ← Redirection si non connecte
```

**Points cles** :
- Dans `hooks.server.ts` : lire le cookie de session, populer `locals.user`
- Dans `dashboard/+page.server.ts` : verifier `locals.user`, `redirect(302, '/login')` si absent
- Utiliser `$env/static/private` pour le secret JWT

---

### Exercice 4 : Blog avec filtrage et pagination

Etends l'exemple du blog avec :

**Filtrage par tag** :
- `src/routes/blog/+page.ts` : lire `url.searchParams.get('tag')`
- Filtrer les articles cote serveur dans la load function
- Afficher les tags comme liens dans la page

**Pagination** :
- Parametres `?page=1&limit=5` dans l'URL
- Retourner `{ posts, total, page, totalPages }` depuis la load function
- Composant `Pagination.svelte` dans `$lib/components/`

**Bonus** : ajouter une barre de recherche qui filtre les articles en temps reel via `goto()` et `invalidate()`, sans rechargement de page.

---

## References et Liens

- [[Svelte/01 - Svelte du Debutant a l'Expert]] - Prerequis : composants, stores, reactivity
- [[Node Express/01 - Express.js et NestJS]] - Comparaison avec un framework serveur classique
- [[JavaScript/03 - JavaScript Asynchrone]] - Promises, async/await indispensables pour les load functions

> [!info] Ressources officielles
> - **Documentation** : kit.svelte.dev
> - **Examples** : learn.svelte.dev (tutoriel interactif officiel)
> - **Playground** : svelte.dev/repl
> - **Discord** : discord.gg/svelte
