# Svelte du Debutant a l'Expert

Svelte est un framework JavaScript radicalement different de [[01 - Introduction a React|React]] et [[01 - Introduction a VueJS|Vue]] : ce n'est pas une bibliotheque qui s'execute dans le navigateur, c'est un **compilateur** qui transforme vos composants en [[04 - JavaScript Moderne ES6+|JavaScript]] vanilla pur au moment du build. Le resultat : zero framework overhead, des bundles minuscules, et une reactivite native sans Virtual DOM.

> [!tip] L'insight cle de Svelte
> [[01 - Introduction a React|React]] et [[01 - Introduction a VueJS|Vue]] embarquent leur runtime dans le bundle final : quand votre app tourne, le framework tourne avec elle en permanence pour gerer le Virtual DOM. Svelte compile vos composants en JavaScript optimise qui manipule directement le DOM reel — le "framework" disparait apres la compilation. C'est comme la difference entre un interprete (React/Vue) et un compilateur (Svelte/C++).

---

## 1. Qu'est-ce que Svelte — Le Compilateur

```
Votre code Svelte (.svelte)
        │
        ▼
┌─────────────────────────┐
│   Compilateur Svelte    │   (pendant le build — pas au runtime)
│   (Rollup / Vite)       │
└─────────────────────────┘
        │
        ▼
JavaScript vanilla optimise
+ CSS scope automatique
+ HTML minimal
        │
        ▼
Bundle final ultra-leger   ← Pas de framework dans le bundle !
```

### Pourquoi pas de Virtual DOM ?

React et Vue utilisent un Virtual DOM (arbre d'objets JS) pour calculer les differences avant de mettre a jour le DOM reel. Cette approche a un cout : creer le VDOM, le differ, puis patcher le DOM reel — trois etapes au lieu d'une.

Svelte sait exactement quelles parties du DOM peuvent changer (il les a analysees a la compilation) et genere du code qui les met a jour directement. Resultat : moins de travail, moins de memoire, plus rapide.

---

## 2. Comparaison [[01 - Introduction a React|React]] vs [[01 - Introduction a VueJS|Vue]] vs Svelte

| Critere | React 18 | Vue 3 | Svelte 5 |
|---|---|---|---|
| **Type** | Bibliotheque | Framework | Compilateur |
| **Bundle runtime** | ~40 KB | ~34 KB | ~1-5 KB |
| **Virtual DOM** | Oui | Oui | Non |
| **Reactivite** | useState/useEffect | ref/reactive | Native ($:) |
| **Boilerplate** | Moyen | Faible | Tres faible |
| **TypeScript** | Excellent | Excellent | Excellent |
| **Courbe d'app.** | Moderee | Douce | Tres douce |
| **Ecosysteme** | Enorme | Grand | Croissant |
| **SSR** | Next.js | Nuxt.js | SvelteKit |
| **Performance** | Tres bonne | Tres bonne | Excellente |
| **Animations** | Framer Motion | Transition | Natif |
| **Gestion d'etat** | Redux/Zustand | Pinia | Stores natifs |

### Benchmark de taille de bundle (Hello World + TodoMVC)

```
Hello World :
  React    : 42 KB (gzip: 14 KB)
  Vue      : 34 KB (gzip: 12 KB)
  Svelte   :  1.6 KB (gzip: 0.7 KB)

TodoMVC complet :
  React    : 48 KB
  Vue      : 39 KB
  Svelte   :  9 KB
```

> [!info] La taille compte-t-elle encore ?
> Sur desktop en fibre, non. Sur mobile 3G en Afrique/Asie du Sud-Est, oui enormement. Svelte est ideallement positionne pour les apps mobiles, les apps embarquees (IoT), et les applications ou chaque KB compte pour le Core Web Vitals.

---

## 3. Installation avec SvelteKit

SvelteKit est le framework meta officiel de Svelte (l'equivalent de Next.js pour [[01 - Introduction a React|React]] ou Nuxt pour [[01 - Introduction a VueJS|Vue]]). C'est la solution privilegiee pour construire des [[05 - SPA et Frameworks Introduction|SPAs]] et des apps SSR avec Svelte.

```bash
# Creer un projet SvelteKit
npm create svelte@latest mon-projet

# Reponses recommandees :
#   Template : SvelteKit demo app (pour voir les exemples) ou Skeleton project
#   TypeScript : Yes (recommande)
#   ESLint : Yes
#   Prettier : Yes
#   Playwright : Yes (tests e2e)
#   Vitest : Yes

cd mon-projet
npm install
npm run dev
```

### Structure d'un projet SvelteKit

```
mon-projet/
├── src/
│   ├── lib/
│   │   ├── components/     ← Composants reutilisables
│   │   ├── stores/         ← Stores Svelte
│   │   └── utils/          ← Utilitaires
│   ├── routes/
│   │   ├── +layout.svelte  ← Layout global
│   │   ├── +layout.server.js ← Load function serveur du layout
│   │   ├── +page.svelte    ← Page principale (/)
│   │   ├── +page.server.js ← Load function serveur de la page
│   │   ├── about/
│   │   │   └── +page.svelte   ← Route /about
│   │   └── users/
│   │       ├── +page.svelte   ← Route /users
│   │       └── [id]/
│   │           └── +page.svelte ← Route /users/:id
│   └── app.html            ← Template HTML racine
├── static/                 ← Fichiers statiques servis tels quels
├── svelte.config.js
├── vite.config.js
└── package.json
```

---

## 4. Syntaxe Svelte — Les Fondamentaux

### 4.1 Structure d'un composant

```svelte
<!-- src/lib/components/Hello.svelte -->

<!-- SCRIPT : logique JavaScript -->
<script>
  // Props (parametres du composant)
  export let name = 'Monde'
  export let count = 0
  
  // Variables locales (etat reactif automatique !)
  let message = 'Bonjour'
  
  // Fonction
  function greet() {
    message = `Bonjour ${name} !`  // Assigner = reactive !
  }
</script>

<!-- TEMPLATE : HTML avec syntaxe Svelte -->
<h1>{message}, {name} !</h1>
<p>Compteur : {count}</p>
<button on:click={greet}>Saluer</button>

<!-- STYLE : CSS scope automatique (pas de class conflicts) -->
<style>
  h1 {
    color: purple;
    /* Ce style ne s'applique QU'a ce composant */
  }
</style>
```

### 4.2 Reactivite native avec `$:`

La reactivite de Svelte est syntaxiquement la plus simple des trois frameworks. Elle repose sur des declarations reactives, sans les `useState`/`useEffect` de [[01 - Introduction a React|React]] ni les `ref`/`reactive` de [[01 - Introduction a VueJS|Vue]].

```svelte
<script>
  let count = 0
  let name = 'Alice'
  
  // $: est une "reactive declaration" — recalcule quand les dependances changent
  // Equivalent de computed() dans Vue ou useMemo() dans React
  $: doubled = count * 2
  $: greeting = `Bonjour ${name}, tu as ${count} points`
  
  // $: peut aussi executer des effets (side effects)
  // Equivalent de useEffect() dans React ou watchEffect() dans Vue
  $: {
    console.log('count a change :', count)
    if (count > 10) {
      alert('Plus de 10 !')
    }
  }
  
  // $: avec condition
  $: isEven = count % 2 === 0
  
  // Reactif sur des tableaux/objets
  let todos = []
  $: remaining = todos.filter(t => !t.done).length
  
  function addTodo(text) {
    // IMPORTANT : l'assignation directe declenche la reactivite
    // Pour les tableaux, utiliser une nouvelle reference
    todos = [...todos, { id: Date.now(), text, done: false }]
    // todos.push() NE declenche PAS la reactivite en Svelte 4 !
    // (Svelte 5 avec runes corrige ce comportement)
  }
  
  function toggleTodo(id) {
    // Mutation puis reassignation pour declencher la reactivite
    todos = todos.map(t =>
      t.id === id ? { ...t, done: !t.done } : t
    )
  }
</script>

<p>{count} × 2 = {doubled}</p>
<p>{greeting}</p>
<p>{count} est {isEven ? 'pair' : 'impair'}</p>
<p>{remaining} tache(s) restante(s)</p>
<button on:click={() => count++}>Incrementer</button>
```

> [!warning] Reactivite et mutations de tableau en Svelte 4
> En Svelte 4, `array.push()`, `array.splice()` etc. ne declenchent PAS la reactivite. Vous devez reassigner : `todos = [...todos, newItem]` ou `todos = todos; // force update` apres mutation. Svelte 5 (runes) resout ce probleme.

### 4.3 Blocs de template — `{#if}`, `{#each}`, `{#await}`

Le bloc `{#await}` est particulierement puissant pour gerer les [[03 - JavaScript Asynchrone|Promises]] directement dans le template, sans passer par un store intermediaire.

```svelte
<script>
  let user = null
  let score = 75
  let fruits = ['Pomme', 'Banane', 'Cerise']
  let products = [
    { id: 1, name: 'Clavier', price: 79 },
    { id: 2, name: 'Souris', price: 29 }
  ]
  
  async function fetchUser() {
    const res = await fetch('/api/user/1')
    return res.json()
  }
  
  const userPromise = fetchUser()
</script>

<!-- {#if} / {:else if} / {:else} -->
{#if !user}
  <p>Veuillez vous connecter</p>
{:else if user.role === 'admin'}
  <p>Panneau administrateur</p>
{:else}
  <p>Bonjour {user.name}</p>
{/if}

{#if score >= 90}
  <span class="mention">Tres Bien</span>
{:else if score >= 75}
  <span class="mention">Bien</span>
{:else}
  <span>Insuffisant</span>
{/if}

<!-- {#each} -->
<ul>
  {#each fruits as fruit}
    <li>{fruit}</li>
  {/each}
</ul>

<!-- Avec index et cle (important pour les listes dynamiques) -->
<ul>
  {#each products as product, index (product.id)}
    <li>{index + 1}. {product.name} — {product.price} €</li>
  {:else}
    <li>Aucun produit disponible</li>
  {/each}
</ul>

<!-- Destructuration dans each -->
{#each products as { id, name, price } (id)}
  <div data-id={id}>
    <strong>{name}</strong>: {price} €
  </div>
{/each}

<!-- {#await} : gestion des Promises dans le template -->
{#await userPromise}
  <p>Chargement...</p>
{:then user}
  <h2>Bonjour {user.name}</h2>
{:catch error}
  <p class="error">Erreur : {error.message}</p>
{/await}

<!-- Version courte (pas d'etat de chargement) -->
{#await userPromise then user}
  <p>{user.name}</p>
{/await}
```

### 4.4 Binding — `bind:`

```svelte
<script>
  let name = ''
  let age = 25
  let isAccepted = false
  let selectedFruits = []
  let selectedCountry = ''
  let sliderValue = 50
  let inputRef
  
  function focusInput() {
    inputRef.focus()
  }
</script>

<!-- Two-way binding (equivalent de v-model en Vue) -->
<input bind:value={name} placeholder="Nom">
<p>Bonjour {name}</p>

<!-- Binding numerique -->
<input type="number" bind:value={age}>

<!-- Binding checkbox -->
<input type="checkbox" bind:checked={isAccepted}>
<label>Accepter les CGU</label>

<!-- Binding groupe de checkboxes -->
<input type="checkbox" bind:group={selectedFruits} value="pomme"> Pomme
<input type="checkbox" bind:group={selectedFruits} value="banane"> Banane
<p>Selectionne : {selectedFruits.join(', ')}</p>

<!-- Binding select -->
<select bind:value={selectedCountry}>
  <option value="">Choisir</option>
  <option value="fr">France</option>
  <option value="be">Belgique</option>
</select>

<!-- Binding range -->
<input type="range" min="0" max="100" bind:value={sliderValue}>
<p>Valeur : {sliderValue}</p>

<!-- Binding DOM ref (this) -->
<input bind:this={inputRef} type="text">
<button on:click={focusInput}>Focus</button>

<!-- Binding de proprietes de dimension (readonly) -->
<div bind:clientWidth={width} bind:clientHeight={height}>
  Dimensions: {width}×{height}
</div>
```

### 4.5 Evenements — `on:` et `createEventDispatcher`

```svelte
<!-- Composant enfant : CustomButton.svelte -->
<script>
  import { createEventDispatcher } from 'svelte'
  
  export let label = 'Cliquer'
  export let disabled = false
  
  const dispatch = createEventDispatcher()
  
  function handleClick(event) {
    if (!disabled) {
      // Emettre un evenement personnalise
      dispatch('click', {
        label,
        timestamp: new Date().toISOString()
      })
    }
  }
</script>

<button
  class:disabled
  on:click={handleClick}
  {disabled}
>
  {label}
</button>
```

```svelte
<!-- Composant parent -->
<script>
  import CustomButton from './CustomButton.svelte'
  
  function handleCustomClick(event) {
    // event.detail contient les donnees de dispatch()
    console.log('Clique :', event.detail.label, event.detail.timestamp)
  }
</script>

<!-- Ecouter l'evenement personnalise -->
<CustomButton
  label="Valider"
  on:click={handleCustomClick}
/>

<!-- Modificateurs d'evenement -->
<form on:submit|preventDefault={handleSubmit}>
  <button type="submit" on:click|stopPropagation={onClick}>
    Soumettre
  </button>
</form>

<!-- event forwarding : transmettre tous les evenements DOM -->
<input on:input on:blur on:focus>
```

---

## 5. Stores Svelte

Les stores Svelte permettent de partager de l'etat entre composants sans relation parent-enfant.

### 5.1 `writable` — Store modifiable

```javascript
// src/lib/stores/counter.js
import { writable } from 'svelte/store'

// Creer un store avec valeur initiale
export const count = writable(0)
export const user = writable(null)

// Store avec methodes personnalisees
function createCartStore() {
  const { subscribe, set, update } = writable([])
  
  return {
    subscribe,  // Obligatoire pour l'abonnement
    
    addItem(item) {
      update(items => {
        const existing = items.find(i => i.id === item.id)
        if (existing) {
          return items.map(i =>
            i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
          )
        }
        return [...items, { ...item, quantity: 1 }]
      })
    },
    
    removeItem(id) {
      update(items => items.filter(i => i.id !== id))
    },
    
    clear() {
      set([])
    }
  }
}

export const cart = createCartStore()
```

```svelte
<!-- Utilisation du store -->
<script>
  import { count, cart } from '$lib/stores/counter'
  
  // Syntaxe $store : abonnement automatique + nettoyage
  // Le $ devant le nom du store est une syntaxe Svelte speciale
</script>

<!-- $count = valeur actuelle du store (mise a jour automatique) -->
<p>Compteur : {$count}</p>
<p>Articles dans le panier : {$cart.length}</p>

<button on:click={() => count.update(n => n + 1)}>+</button>
<button on:click={() => count.set(0)}>Reset</button>
<button on:click={() => cart.clear()}>Vider le panier</button>
```

```javascript
// Hors d'un composant Svelte (fichier .js) :
import { count } from '$lib/stores/counter'
import { get } from 'svelte/store'

// Lire la valeur actuelle
const currentCount = get(count)

// S'abonner manuellement (retourne une fonction de desabonnement)
const unsubscribe = count.subscribe(value => {
  console.log('Nouvelle valeur :', value)
})

// Modifier
count.update(n => n + 1)
count.set(42)

// Ne pas oublier de se desabonner !
unsubscribe()
```

### 5.2 `readable` — Store en lecture seule

```javascript
// src/lib/stores/clock.js
import { readable } from 'svelte/store'

// Le store gere lui-meme ses abonnements
export const time = readable(new Date(), function start(set) {
  // Appele quand le premier abonne rejoint
  const interval = setInterval(() => {
    set(new Date())
  }, 1000)
  
  // Retourne la fonction de nettoyage
  return function stop() {
    // Appele quand le dernier abonne part
    clearInterval(interval)
  }
})

// Store de geolocalisation
export const position = readable(null, (set) => {
  const watchId = navigator.geolocation.watchPosition(
    pos => set({ lat: pos.coords.latitude, lng: pos.coords.longitude }),
    err => set({ error: err.message })
  )
  return () => navigator.geolocation.clearWatch(watchId)
})
```

### 5.3 `derived` — Store derive

Les stores `derived` permettent de creer des valeurs calculees a partir d'autres stores, avec support des [[03 - JavaScript Asynchrone|appels asynchrones]] via `set`. Ils peuvent consommer des [[08 - APIs REST avec Flask|APIs REST]] pour charger des donnees reactives.

```javascript
// src/lib/stores/derived.js
import { derived } from 'svelte/store'
import { time } from './clock'
import { cart } from './counter'

// Derive d'un seul store
export const formattedTime = derived(time, $time =>
  $time.toLocaleTimeString('fr-FR')
)

// Derive de plusieurs stores
export const cartTotal = derived(cart, $cart =>
  $cart.reduce((sum, item) => sum + item.price * item.quantity, 0)
)

// Derive asynchrone
export const userPosts = derived(
  currentUser,
  ($user, set) => {
    if (!$user) {
      set([])
      return
    }
    
    fetch(`/api/users/${$user.id}/posts`)
      .then(r => r.json())
      .then(set)
    
    return () => { /* cleanup */ }
  },
  []  // Valeur initiale
)
```

---

## 6. Animations Integrees

L'un des avantages de Svelte : les transitions et animations sont natives, sans bibliotheque tierce. Contrairement a [[01 - Introduction a React|React]] (Framer Motion) ou [[01 - Introduction a VueJS|Vue]] (Transition component), Svelte compile les animations directement dans le CSS genere. Les styles scopés rappellent les principes de [[02 - CSS Fondamentaux|CSS Fondamentaux]] appliques par composant.

### 6.1 `transition:` — Entree et sortie

```svelte
<script>
  import { fade, fly, slide, scale, blur, draw } from 'svelte/transition'
  import { quintOut, elasticOut } from 'svelte/easing'
  
  let visible = true
  let items = ['Pomme', 'Banane', 'Cerise']
</script>

<button on:click={() => visible = !visible}>Basculer</button>

<!-- fade : opacite -->
{#if visible}
  <div transition:fade={{ duration: 300 }}>
    Contenu qui apparait/disparait
  </div>
{/if}

<!-- fly : translate + opacite -->
{#if visible}
  <p transition:fly={{ y: -20, duration: 400, easing: quintOut }}>
    Contenu qui vole
  </p>
{/if}

<!-- slide : hauteur -->
{#if visible}
  <div transition:slide={{ duration: 300 }}>Menu depliable</div>
{/if}

<!-- Transitions differentes a l'entree et a la sortie -->
{#if visible}
  <div in:fly={{ y: -30 }} out:fade>
    Entree par le haut, sortie en fondu
  </div>
{/if}

<!-- Transition sur une liste -->
{#each items as item (item)}
  <div animate:flip transition:slide>
    {item}
  </div>
{/each}
```

### 6.2 `animate:` — Animations de reordonnancement (FLIP)

```svelte
<script>
  import { flip } from 'svelte/animate'
  import { quintOut } from 'svelte/easing'
  
  let todos = [
    { id: 1, text: 'Apprendre Svelte', done: false },
    { id: 2, text: 'Builder une app', done: false },
    { id: 3, text: 'Deployer', done: false }
  ]
  
  function move(id) {
    const todo = todos.find(t => t.id === id)
    todos = [todo, ...todos.filter(t => t.id !== id)]
  }
</script>

<ul>
  {#each todos as todo (todo.id)}
    <!-- animate:flip anime le deplacement quand l'ordre change -->
    <li animate:flip={{ duration: 300, easing: quintOut }}>
      {todo.text}
      <button on:click={() => move(todo.id)}>Monter</button>
    </li>
  {/each}
</ul>
```

### 6.3 `tweened` et `spring` — Valeurs animees

```svelte
<script>
  import { tweened, spring } from 'svelte/motion'
  import { cubicOut } from 'svelte/easing'
  
  // tweened : animation sur une duree fixe avec easing
  const progress = tweened(0, {
    duration: 400,
    easing: cubicOut
  })
  
  // spring : animation physique (rebond naturel)
  const coords = spring({ x: 0, y: 0 }, {
    stiffness: 0.1,
    damping: 0.25
  })
  
  function moveTo(x, y) {
    coords.set({ x, y })
  }
</script>

<!-- progress.set(1) → barre se remplit progressivement -->
<button on:click={() => progress.set(1)}>Charger</button>
<progress value={$progress}></progress>

<!-- La balle suit la souris avec une animation spring -->
<div
  on:mousemove={e => moveTo(e.clientX, e.clientY)}
  style="position: relative; height: 400px;"
>
  <div style="
    position: absolute;
    left: {$coords.x}px;
    top: {$coords.y}px;
    width: 20px; height: 20px;
    border-radius: 50%;
    background: purple;
    transform: translate(-50%, -50%);
  "></div>
</div>
```

---

## 7. SvelteKit en Profondeur

### 7.1 Routing base sur les fichiers

```
src/routes/
├── +page.svelte         → GET /
├── +page.server.js      → Load function (serveur uniquement)
├── +layout.svelte       → Layout partagé par toutes les pages
├── +layout.server.js    → Load function du layout
├── +error.svelte        → Page d'erreur personnalisee
├── about/
│   └── +page.svelte     → GET /about
├── blog/
│   ├── +page.svelte     → GET /blog
│   └── [slug]/
│       └── +page.svelte → GET /blog/:slug
└── api/
    └── users/
        ├── +server.js   → GET /api/users (API route)
        └── [id]/
            └── +server.js → GET/PUT/DELETE /api/users/:id
```

### 7.2 Load Functions

```javascript
// src/routes/blog/[slug]/+page.server.js
// S'execute TOUJOURS cote serveur (Node.js)
// Peut acceder a la base de donnees directement

import { error, redirect } from '@sveltejs/kit'
import { db } from '$lib/server/database'

export async function load({ params, locals, cookies, fetch }) {
  const { slug } = params
  
  // Acces direct a la BDD (pas d'API needed)
  const post = await db.posts.findOne({ slug })
  
  if (!post) {
    throw error(404, { message: `Article "${slug}" introuvable` })
  }
  
  if (post.isDraft && !locals.user?.isAdmin) {
    throw redirect(302, '/blog')
  }
  
  return {
    post,
    // Donnees serializables uniquement (pas de classes, fonctions)
    author: post.author
  }
}
```

```javascript
// src/routes/blog/[slug]/+page.js
// S'execute cote serveur ET cote client
// Utiliser pour les donnees qui peuvent etre fetchees des deux cotes

export async function load({ params, fetch }) {
  // Ce fetch est isomorphe (fonctionne cote serveur et client)
  const response = await fetch(`/api/posts/${params.slug}`)
  
  if (!response.ok) {
    throw error(response.status)
  }
  
  return { post: await response.json() }
}
```

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script>
  // data contient ce que load() retourne
  export let data
  
  const { post, author } = data
</script>

<svelte:head>
  <title>{post.title}</title>
  <meta name="description" content={post.excerpt}>
</svelte:head>

<article>
  <h1>{post.title}</h1>
  <p>Par {author.name} — {post.publishedAt}</p>
  {@html post.content}
</article>
```

### 7.3 Form Actions — Traitement de formulaires

Les form actions de SvelteKit remplacent les appels `fetch` manuels vers une [[08 - APIs REST avec Flask|API REST]] pour les operations CRUD classiques. La directive `use:enhance` cote client reproduit le comportement des SPAs decrit dans [[05 - SPA et Frameworks Introduction|SPA et Frameworks Introduction]].

```javascript
// src/routes/login/+page.server.js
import { fail, redirect } from '@sveltejs/kit'
import { validateCredentials } from '$lib/server/auth'

export const actions = {
  // Action par defaut : appelee si <form> sans action=""
  default: async ({ request, cookies }) => {
    const data = await request.formData()
    const email = data.get('email')
    const password = data.get('password')
    
    // Validation
    if (!email || !password) {
      return fail(400, {
        email,
        error: 'Email et mot de passe requis'
      })
    }
    
    const user = await validateCredentials(email, password)
    
    if (!user) {
      return fail(401, {
        email,
        error: 'Identifiants incorrects'
      })
    }
    
    // Creer une session
    cookies.set('session', user.sessionToken, {
      path: '/',
      httpOnly: true,
      secure: true,
      maxAge: 60 * 60 * 24 * 30  // 30 jours
    })
    
    throw redirect(302, '/dashboard')
  },
  
  // Action nommee : appelee si <form action="?/logout">
  logout: async ({ cookies }) => {
    cookies.delete('session', { path: '/' })
    throw redirect(302, '/login')
  }
}
```

```svelte
<!-- src/routes/login/+page.svelte -->
<script>
  import { enhance } from '$app/forms'
  
  export let form  // Retour de fail() ou de l'action
</script>

<!-- enhance : soumet le formulaire via fetch (pas de rechargement) -->
<!-- et met a jour les donnees automatiquement -->
<form method="POST" use:enhance>
  {#if form?.error}
    <div class="error">{form.error}</div>
  {/if}
  
  <input
    name="email"
    type="email"
    value={form?.email ?? ''}
    required
  >
  <input name="password" type="password" required>
  <button type="submit">Se connecter</button>
</form>

<form method="POST" action="?/logout" use:enhance>
  <button type="submit">Se deconnecter</button>
</form>
```

### 7.4 API Routes — Endpoints REST

```javascript
// src/routes/api/users/+server.js
import { json, error } from '@sveltejs/kit'
import { db } from '$lib/server/database'

// GET /api/users
export async function GET({ url, locals }) {
  const page = Number(url.searchParams.get('page') || '1')
  const limit = Number(url.searchParams.get('limit') || '20')
  
  const users = await db.users.findAll({
    skip: (page - 1) * limit,
    take: limit
  })
  
  return json(users)
}

// POST /api/users
export async function POST({ request, locals }) {
  if (!locals.user?.isAdmin) {
    throw error(403, 'Acces refuse')
  }
  
  const body = await request.json()
  
  if (!body.email || !body.name) {
    throw error(400, 'email et name requis')
  }
  
  const newUser = await db.users.create(body)
  return json(newUser, { status: 201 })
}
```

### 7.5 SSR vs CSR — Quand utiliser quoi

```javascript
// src/routes/dashboard/+page.js
// Desactiver le SSR pour une page specifique
export const ssr = false

// Prerendering a la compilation
export const prerender = true  // Route statique

// src/svelte.config.js
// Ou globalement
const config = {
  kit: {
    // Changer l'adaptateur selon le deploiement cible
    adapter: adapter({
      // 'node' : serveur Node.js (SSR dynamique)
      // 'static' : site statique (SSG)
      // 'vercel' : Vercel (Edge Functions)
      // 'cloudflare' : Cloudflare Pages
    })
  }
}
```

| Mode | Quand utiliser |
|---|---|
| **SSR** (defaut) | Pages publiques avec contenu dynamique, SEO important |
| **SSG** (prerender) | Blog, documentation, pages marketing statiques |
| **CSR** (ssr=false) | Dashboards prives, apps connectees, contenus sans SEO |

---

## 8. Svelte 5 — Les Runes (Nouvelle Reactivite)

Svelte 5 introduit les "runes" : une API explicite qui remplace la magie implicite de `$:`.

```svelte
<script>
  // Svelte 5 : runes
  
  // $state remplace les variables locales reactives
  let count = $state(0)
  let name = $state('Alice')
  
  // $derived remplace $:
  let doubled = $derived(count * 2)
  let greeting = $derived(`Bonjour ${name}, tu as ${count} points`)
  
  // $effect remplace les blocs $: avec side effects
  $effect(() => {
    console.log(`count a change : ${count}`)
    return () => {
      // Cleanup (comme onBeforeUnmount)
      console.log('Cleanup')
    }
  })
  
  // Props avec $props (remplace export let)
  let { title, count: initialCount = 0 } = $props()
</script>
```

> [!info] Svelte 4 vs Svelte 5
> Svelte 5 est sorti fin 2024. Les runes corrigent des cas limites de la magie implicite (mutations de tableaux, etc.) et rendent la reactivite plus previsible. Svelte 4 reste pleinement supporte. Ce cours couvre principalement Svelte 4 car c'est ce que vous rencontrerez dans la plupart des projets en production actuellement.

---

## 9. Deploiement

Pour automatiser le deploiement d'une app SvelteKit, les pipelines [[04 - CI-CD avec GitHub Actions|CI/CD avec GitHub Actions]] permettent de builder et deployer sur Vercel ou Netlify a chaque push sur la branche principale. Voir aussi [[01 - Tests Unitaires et TDD|Tests Unitaires]] et [[02 - Tests Integration et E2E|Tests E2E]] pour integrer Vitest et Playwright dans la CI.

### 9.1 Adaptateur SvelteKit

```bash
# Site statique (SSG) — deploiement sur tout CDN
npm install -D @sveltejs/adapter-static

# Vercel
npm install -D @sveltejs/adapter-vercel

# Netlify
npm install -D @sveltejs/adapter-netlify

# Node.js (serveur traditionnel)
npm install -D @sveltejs/adapter-node
```

```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-node'

export default {
  kit: {
    adapter: adapter({
      out: 'build'
    })
  }
}
```

```bash
# Build
npm run build

# Lancer en production
node build/index.js
```

### 9.2 Variables d'environnement

```bash
# .env
PUBLIC_API_URL=https://api.example.com   # Accessible cote client
PRIVATE_DB_URL=postgres://...             # Accessible cote serveur uniquement
```

```javascript
// Cote serveur uniquement
import { PRIVATE_DB_URL } from '$env/static/private'

// Cote client et serveur
import { PUBLIC_API_URL } from '$env/static/public'
```

---

## 10. Exemple Complet — App de Notes avec SvelteKit

Application complete avec CRUD, stores, et persistance.

```javascript
// src/lib/stores/notes.js
import { writable, derived } from 'svelte/store'

function createNotesStore() {
  const notes = writable(loadNotes())
  
  function loadNotes() {
    try {
      return JSON.parse(localStorage.getItem('notes') || '[]')
    } catch { return [] }
  }
  
  notes.subscribe(val => {
    try { localStorage.setItem('notes', JSON.stringify(val)) } catch {}
  })
  
  const { subscribe, update, set } = notes
  
  return {
    subscribe,
    add(title, content) {
      update(n => [{
        id: crypto.randomUUID(),
        title: title.trim(),
        content: content.trim(),
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString()
      }, ...n])
    },
    update(id, changes) {
      update(n => n.map(note =>
        note.id === id
          ? { ...note, ...changes, updatedAt: new Date().toISOString() }
          : note
      ))
    },
    delete(id) {
      update(n => n.filter(note => note.id !== id))
    }
  }
}

export const notesStore = createNotesStore()
export const searchQuery = writable('')

export const filteredNotes = derived(
  [notesStore, searchQuery],
  ([$notes, $query]) => {
    if (!$query.trim()) return $notes
    const q = $query.toLowerCase()
    return $notes.filter(note =>
      note.title.toLowerCase().includes(q) ||
      note.content.toLowerCase().includes(q)
    )
  }
)
```

```svelte
<!-- src/routes/+page.svelte -->
<script>
  import { fly, fade } from 'svelte/transition'
  import { notesStore, filteredNotes, searchQuery } from '$lib/stores/notes'
  
  let title = ''
  let content = ''
  let editingId = null
  let editTitle = ''
  let editContent = ''
  
  function addNote() {
    if (!title.trim()) return
    notesStore.add(title, content)
    title = ''
    content = ''
  }
  
  function startEdit(note) {
    editingId = note.id
    editTitle = note.title
    editContent = note.content
  }
  
  function saveEdit() {
    notesStore.update(editingId, { title: editTitle, content: editContent })
    editingId = null
  }
  
  function cancelEdit() {
    editingId = null
  }
</script>

<svelte:head>
  <title>Mes Notes</title>
</svelte:head>

<main>
  <h1>Mes Notes</h1>
  
  <!-- Formulaire d'ajout -->
  <form on:submit|preventDefault={addNote} class="add-form">
    <input
      bind:value={title}
      placeholder="Titre de la note"
      required
    >
    <textarea
      bind:value={content}
      placeholder="Contenu..."
      rows="4"
    ></textarea>
    <button type="submit" disabled={!title.trim()}>
      Ajouter la note
    </button>
  </form>
  
  <!-- Recherche -->
  <input
    bind:value={$searchQuery}
    placeholder="Rechercher..."
    class="search"
  >
  
  <!-- Compteur -->
  <p class="count">
    {$filteredNotes.length} note{$filteredNotes.length !== 1 ? 's' : ''}
    {$searchQuery ? `pour "${$searchQuery}"` : ''}
  </p>
  
  <!-- Liste des notes -->
  <div class="notes-grid">
    {#each $filteredNotes as note (note.id)}
      <div
        class="note-card"
        in:fly={{ y: 20, duration: 300 }}
        out:fade={{ duration: 200 }}
      >
        {#if editingId === note.id}
          <!-- Mode edition -->
          <input bind:value={editTitle}>
          <textarea bind:value={editContent} rows="4"></textarea>
          <div class="actions">
            <button on:click={saveEdit} class="btn-primary">Sauvegarder</button>
            <button on:click={cancelEdit}>Annuler</button>
          </div>
        {:else}
          <!-- Mode affichage -->
          <h3>{note.title}</h3>
          <p>{note.content}</p>
          <small>{new Date(note.updatedAt).toLocaleDateString('fr-FR')}</small>
          <div class="actions">
            <button on:click={() => startEdit(note)}>Modifier</button>
            <button
              on:click={() => notesStore.delete(note.id)}
              class="btn-danger"
            >
              Supprimer
            </button>
          </div>
        {/if}
      </div>
    {:else}
      <p class="empty">
        {$searchQuery ? 'Aucun resultat' : 'Aucune note. Creez-en une !'}
      </p>
    {/each}
  </div>
</main>

<style>
  main { max-width: 900px; margin: 0 auto; padding: 20px; }
  .notes-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 16px; }
  .note-card { border: 1px solid #ddd; border-radius: 8px; padding: 16px; background: #fff; }
  .add-form { display: flex; flex-direction: column; gap: 8px; margin-bottom: 24px; }
  .search { width: 100%; padding: 8px; margin-bottom: 16px; border: 1px solid #ddd; border-radius: 4px; }
  .btn-danger { color: red; border-color: red; }
</style>
```

---

## 11. Exercices Pratiques

> [!tip] Exercice 1 — Store de panier d'achat
> Cree un store `cartStore` avec :
> - `items` : liste des produits dans le panier
> - `total` derive : prix total
> - `itemCount` derive : nombre total d'articles
> - Methodes : `addItem`, `removeItem`, `updateQuantity`, `clear`
> - Persistence localStorage
> Affiche le panier dans un composant `Cart.svelte` avec animations `flip` pour les changements d'ordre.

> [!tip] Exercice 2 — Application meteo avec {#await}
> Avec l'API OpenWeatherMap (gratuite) :
> - Input de recherche de ville avec debounce (store + $: + setTimeout)
> - Affichage de la meteo actuelle via `{#await}` avec etats loading/success/error
> - Store `recentCities` (writable) pour sauvegarder les recherches recentes
> - Transitions sur l'affichage des resultats

> [!tip] Exercice 3 — Blog SSR avec SvelteKit
> Construis un blog avec SvelteKit :
> - Route `/blog` : liste des articles chargee via load function
> - Route `/blog/[slug]` : article complet avec load function serveur
> - `+layout.svelte` avec navigation commune
> - API route `GET /api/posts` qui retourne du JSON
> - Page d'erreur 404 personnalisee (`+error.svelte`)
> - Prerendering active pour les articles statiques

> [!tip] Exercice 4 — Formulaire d'inscription multi-etapes
> Avec les form actions SvelteKit :
> - Etape 1 : Informations personnelles (nom, email)
> - Etape 2 : Mot de passe + confirmation (validation serveur)
> - Etape 3 : Preferences (newsletter, notifications)
> - Action `default` pour soumettre, retourner `fail()` si erreur
> - `use:enhance` pour une soumission sans rechargement
> - Indicateur de progression entre les etapes avec transition `slide`

---

## Liens Utiles

- Documentation Svelte : https://svelte.dev/docs
- SvelteKit : https://kit.svelte.dev/
- REPL interactif : https://svelte.dev/repl

---

## Notes liées

- [[01 - Introduction a VueJS]] — comparatif Vue vs Svelte, reactivite, composants
- [[01 - Introduction a React]] — comparatif React vs Svelte, JSX, hooks
- [[05 - SPA et Frameworks Introduction]] — contexte SPA, routing, rendu SSR/CSR
- [[04 - JavaScript Moderne ES6+]] — modules ES6, classes, arrow functions utilises dans Svelte
- [[03 - JavaScript Asynchrone]] — Promises, async/await, fetch dans les stores et le bloc {#await}
- [[02 - CSS Fondamentaux]] — CSS scope, variables CSS, animations
- [[01 - Tests Unitaires et TDD]] — Vitest pour les tests unitaires de composants Svelte
- [[02 - Tests Integration et E2E]] — Playwright pour les tests E2E SvelteKit
- [[04 - CI-CD avec GitHub Actions]] — pipelines de deploiement pour apps SvelteKit
- [[08 - APIs REST avec Flask]] — backend a consommer depuis les load functions et les stores
