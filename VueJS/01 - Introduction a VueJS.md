# Introduction a VueJS

Vue.js est un framework [[04 - JavaScript Moderne ES6+|JavaScript]] progressif pour construire des interfaces utilisateur. Contrairement a Angular (framework complet et opiniate) ou [[01 - Introduction a React|React]] (bibliotheque centree sur le rendu), Vue se distingue par sa courbe d'apprentissage douce, sa flexibilite d'adoption incrementale, et son excellente documentation en francais.

Ce cours couvre la philosophie de Vue, l'installation, la syntaxe de template, les composants, les computed properties, les watchers, et les lifecycle hooks. A la fin, vous construirez un CRUD complet.

> [!tip] Pourquoi Vue ?
> Vue combine le meilleur de [[01 - Introduction a React|React]] (composants, Virtual DOM, JSX optionnel) et d'Angular (directives, two-way binding) dans un package accessible. Son slogan "The Progressive Framework" signifie que vous pouvez l'integrer dans une page existante avec une simple balise `<script>` ou construire une [[05 - SPA et Frameworks Introduction|SPA]] complexe avec Vite et l'ecosysteme complet.

---

## 1. Philosophie Vue — Le Framework Progressif

Vue est concu pour etre adopte incrementalement :

```
Niveau 1 — Amelioration de HTML existant
  <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
  → Ajout de reactivite a n'importe quelle page HTML

Niveau 2 — Composants Single-File (SFC)
  Avec un bundler (Vite) : fichiers .vue avec <template>, <script>, <style>
  → Application SPA complete

Niveau 3 — Ecosysteme complet
  Vue Router + Pinia + Vite + TypeScript + Vitest
  → Application enterprise-grade
```

> [!info] Single-File Components (SFC)
> Un fichier `.vue` encapsule le template HTML, le JavaScript, et le CSS d'un composant en un seul endroit. C'est le pattern recommande pour tout projet serieux avec un bundler.

---

## 2. Comparaison [[01 - Introduction a React|React]] vs Vue vs Angular

| Critere | React | Vue 3 | Angular |
|---|---|---|---|
| **Type** | Bibliotheque UI | Framework progressif | Framework complet |
| **Courbe d'apprentissage** | Moderee | Douce | Steep |
| **Langage** | JSX (JS+HTML) | Templates HTML | TypeScript obligatoire |
| **Taille bundle** | ~40 KB | ~34 KB | ~130 KB |
| **Two-way binding** | Manuel (useState) | `v-model` natif | `[(ngModel)]` |
| **Gestion d'etat** | Redux / Zustand | Pinia | NgRx / Services |
| **Routing** | React Router | Vue Router | Angular Router |
| **Rendu serveur** | Next.js | Nuxt.js | Angular Universal |
| **Entreprises** | Meta, Airbnb | Alibaba, GitLab | Google, Microsoft |
| **Performance** | Excellente | Excellente | Bonne |
| **Flexibilite** | Tres haute | Haute | Faible (opiniate) |
| **Tests** | Jest + RTL | Vitest + VTU | Karma + Jasmine |

> [!tip] Quand choisir Vue ?
> Vue est ideal si votre equipe vient de jQuery/HTML classique, si vous voulez integrer progressivement la reactivite, ou si vous cherchez la meilleure experience developpeur pour des projets de taille moyenne. [[01 - Introduction a React|React]] domine dans les grandes entreprises tech. Angular est privilegie dans les entreprises legacy Java/.NET. Pour une alternative plus legere et compilee, voir [[01 - Svelte du Debutant a l'Expert|Svelte]].

---

## 3. Installation

### Option 1 — CDN (Prototype rapide)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Mon App Vue</title>
</head>
<body>
  <div id="app">{{ message }}</div>

  <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
  <script>
    const { createApp, ref } = Vue

    createApp({
      setup() {
        const message = ref('Bonjour Vue 3 !')
        return { message }
      }
    }).mount('#app')
  </script>
</body>
</html>
```

### Option 2 — Vite + Vue (Recommande)

```bash
# Creer un projet
npm create vue@latest mon-projet
# Reponses recommandees pour ce cours :
#   Add TypeScript? → No (pour commencer)
#   Add JSX Support? → No
#   Add Vue Router? → Yes
#   Add Pinia? → Yes
#   Add Vitest? → Yes
#   Add ESLint? → Yes

cd mon-projet
npm install
npm run dev
```

Structure generee :

```
mon-projet/
├── public/
│   └── favicon.ico
├── src/
│   ├── assets/          ← Images, fonts, CSS global
│   ├── components/      ← Composants reutilisables
│   ├── router/
│   │   └── index.js     ← Configuration Vue Router
│   ├── stores/          ← Stores Pinia
│   ├── views/           ← Pages (composants routes)
│   ├── App.vue          ← Composant racine
│   └── main.js          ← Point d'entree
├── index.html
├── package.json
└── vite.config.js
```

### Option 3 — Vue CLI (Ancienne methode, toujours supportee)

```bash
npm install -g @vue/cli
vue create mon-projet
vue ui  # Interface graphique de configuration
```

> [!warning] Vue CLI vs Vite
> Vue CLI utilise Webpack comme bundler — beaucoup plus lent que Vite. Pour tout nouveau projet en 2024+, utilisez `npm create vue@latest` qui utilise Vite. Vue CLI reste utile pour les projets legacy ou les configurations Webpack specifiques.

---

## 4. Options API vs Composition API

Vue 3 propose deux styles d'ecriture. Les deux sont valides et interoperables. La Composition API tire pleinement parti des fonctionnalites [[04 - JavaScript Moderne ES6+|ES6+]] comme les modules, les arrow functions et la destructuration.

### Options API (Vue 2 style, toujours supporte)

Organise le code par **type d'option** : data, methods, computed, watch...

```vue
<script>
export default {
  name: 'CompteurOptions',
  
  // Etat reactif
  data() {
    return {
      count: 0,
      message: 'Bonjour'
    }
  },
  
  // Proprietes calculees
  computed: {
    doubleCount() {
      return this.count * 2
    }
  },
  
  // Methodes
  methods: {
    increment() {
      this.count++
    },
    decrement() {
      if (this.count > 0) this.count--
    }
  },
  
  // Observateurs
  watch: {
    count(newVal, oldVal) {
      console.log(`Count: ${oldVal} → ${newVal}`)
    }
  },
  
  // Lifecycle hooks
  mounted() {
    console.log('Composant monte dans le DOM')
  }
}
</script>
```

### Composition API (Vue 3 style, recommande)

Organise le code par **logique metier** — tout ce qui concerne le compteur est ensemble.

```vue
<script setup>
import { ref, computed, watch, onMounted } from 'vue'

// Etat reactif
const count = ref(0)
const message = ref('Bonjour')

// Propriete calculee
const doubleCount = computed(() => count.value * 2)

// Methodes (simples fonctions)
function increment() {
  count.value++
}

function decrement() {
  if (count.value > 0) count.value--
}

// Observateur
watch(count, (newVal, oldVal) => {
  console.log(`Count: ${oldVal} → ${newVal}`)
})

// Lifecycle hook
onMounted(() => {
  console.log('Composant monte dans le DOM')
})
</script>
```

> [!info] `<script setup>` — Syntaxe sugar
> La balise `<script setup>` est du sucre syntaxique pour la Composition API. Tout ce qui est declare a l'interieur est automatiquement expose au template — pas besoin de `return { ... }`. C'est la syntaxe recommandee pour tout nouveau code Vue 3.

### Comparaison Options API vs Composition API

| Aspect | Options API | Composition API |
|---|---|---|
| **Organisation** | Par type (data/methods/computed) | Par logique metier |
| **Lisibilite** | Facile a apprendre | Meilleure sur gros composants |
| **TypeScript** | Support limite | Excellent support |
| **Reutilisation logique** | Mixins (problematiques) | Composables (propres) |
| **Vue 2** | Identique | Non disponible |
| **Recommandation** | Projets Vue 2 / debutants | Tout nouveau projet Vue 3 |

---

## 5. Template Syntax

### 5.1 Interpolation de texte

```vue
<template>
  <!-- Interpolation basique -->
  <p>{{ message }}</p>
  
  <!-- Expression JavaScript dans {{ }} -->
  <p>{{ count + 1 }}</p>
  <p>{{ message.toUpperCase() }}</p>
  <p>{{ isActive ? 'Actif' : 'Inactif' }}</p>
  
  <!-- HTML brut (ATTENTION : risque XSS) -->
  <p v-html="rawHtml"></p>
</template>
```

> [!warning] Securite avec `v-html`
> N'utilisez jamais `v-html` avec des donnees venant d'un utilisateur sans les avoir sanitisees. C'est une faille XSS (Cross-Site Scripting) classique. Utilisez une bibliotheque comme `DOMPurify` si le HTML vient d'une source externe.

### 5.2 `v-bind` — Liaison d'attributs

```vue
<template>
  <!-- Syntaxe complete -->
  <img v-bind:src="imageUrl" v-bind:alt="imageAlt">
  
  <!-- Raccourci : -->
  <img :src="imageUrl" :alt="imageAlt">
  
  <!-- Binding dynamique de classe -->
  <div :class="{ active: isActive, 'text-danger': hasError }">...</div>
  <div :class="[activeClass, errorClass]">...</div>
  
  <!-- Binding dynamique de style -->
  <div :style="{ color: textColor, fontSize: fontSize + 'px' }">...</div>
  
  <!-- Binding d'attribut dynamique -->
  <a :[attributeName]="url">Lien dynamique</a>
  
  <!-- Spread d'objet d'attributs -->
  <input v-bind="inputProps">
</template>

<script setup>
import { ref, reactive } from 'vue'

const imageUrl = ref('/img/logo.png')
const imageAlt = ref('Logo de l\'application')
const isActive = ref(true)
const hasError = ref(false)
const textColor = ref('red')
const fontSize = ref(16)
const attributeName = ref('href')
const url = ref('https://vuejs.org')
const inputProps = reactive({ type: 'text', placeholder: 'Entrez du texte' })
</script>
```

### 5.3 `v-on` — Gestion des evenements

```vue
<template>
  <!-- Syntaxe complete -->
  <button v-on:click="increment">+</button>
  
  <!-- Raccourci @ -->
  <button @click="increment">+</button>
  
  <!-- Expression inline -->
  <button @click="count++">Inline</button>
  
  <!-- Modificateurs d'evenement -->
  <form @submit.prevent="onSubmit">...</form>        <!-- preventDefault() -->
  <button @click.stop="onClick">Stop</button>        <!-- stopPropagation() -->
  <input @keyup.enter="onEnter">                     <!-- Touche Entree seulement -->
  <input @keyup.ctrl.enter="onCtrlEnter">            <!-- Ctrl + Entree -->
  <button @click.once="onClickOnce">Une fois</button><!-- Ecoute une seule fois -->
  
  <!-- Passer l'evenement natif -->
  <button @click="handleClick($event)">Avec event</button>
</template>

<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}

function onSubmit() {
  console.log('Formulaire soumis sans rechargement')
}

function handleClick(event) {
  console.log('Element clique :', event.target)
}
</script>
```

### 5.4 `v-model` — Two-Way Binding

```vue
<template>
  <!-- Input texte -->
  <input v-model="username" placeholder="Nom d'utilisateur">
  <p>Bonjour {{ username }}</p>
  
  <!-- Textarea -->
  <textarea v-model="description"></textarea>
  
  <!-- Checkbox -->
  <input type="checkbox" v-model="isAccepted">
  <label>J'accepte les CGU</label>
  
  <!-- Groupe de checkboxes → tableau -->
  <input type="checkbox" value="option1" v-model="selectedOptions">
  <input type="checkbox" value="option2" v-model="selectedOptions">
  
  <!-- Radio buttons -->
  <input type="radio" value="oui" v-model="reponse">
  <input type="radio" value="non" v-model="reponse">
  
  <!-- Select -->
  <select v-model="pays">
    <option value="">Choisir un pays</option>
    <option value="fr">France</option>
    <option value="be">Belgique</option>
  </select>
  
  <!-- Modificateurs v-model -->
  <input v-model.trim="email">          <!-- Supprime espaces debut/fin -->
  <input v-model.number="age" type="number"> <!-- Convertit en Number -->
  <input v-model.lazy="bio">           <!-- Sync sur blur, pas sur input -->
</template>

<script setup>
import { ref } from 'vue'

const username = ref('')
const description = ref('')
const isAccepted = ref(false)
const selectedOptions = ref([])
const reponse = ref('')
const pays = ref('')
const email = ref('')
const age = ref(0)
const bio = ref('')
</script>
```

### 5.5 `v-if`, `v-else-if`, `v-else` et `v-show`

```vue
<template>
  <!-- v-if : le DOM est cree/detruit -->
  <div v-if="score >= 90">Mention Tres Bien</div>
  <div v-else-if="score >= 75">Mention Bien</div>
  <div v-else-if="score >= 60">Mention Assez Bien</div>
  <div v-else>Insuffisant</div>
  
  <!-- v-show : le DOM reste, display:none est ajoute/retire -->
  <p v-show="isVisible">Visible ou cache via CSS</p>
  
  <!-- Grouper avec <template> (pas de noeud DOM supplementaire) -->
  <template v-if="isLoggedIn">
    <h1>Bonjour {{ user.name }}</h1>
    <p>Derniere connexion : {{ user.lastLogin }}</p>
  </template>
</template>
```

> [!info] `v-if` vs `v-show`
> - `v-if` : cree et detruit le composant a chaque changement. Couteux si frequent. Utiliser quand la condition change rarement.
> - `v-show` : ne fait que basculer `display: none`. Moins couteux si frequent. Utiliser pour les toggles (menus, modals).

### 5.6 `v-for` — Rendu de liste

```vue
<template>
  <!-- Tableau de valeurs primitives -->
  <ul>
    <li v-for="fruit in fruits" :key="fruit">{{ fruit }}</li>
  </ul>
  
  <!-- Tableau d'objets -->
  <div v-for="user in users" :key="user.id">
    <h3>{{ user.name }}</h3>
    <p>{{ user.email }}</p>
  </div>
  
  <!-- Avec index -->
  <li v-for="(item, index) in items" :key="item.id">
    {{ index + 1 }}. {{ item.name }}
  </li>
  
  <!-- Objet (proprietes) -->
  <div v-for="(value, key, index) in monObjet" :key="key">
    {{ index }}. {{ key }} : {{ value }}
  </div>
  
  <!-- Plage numerique -->
  <span v-for="n in 10" :key="n">{{ n }} </span>
  
  <!-- v-for + v-if : utiliser <template> pour eviter le conflit -->
  <template v-for="user in users" :key="user.id">
    <li v-if="user.isActive">{{ user.name }}</li>
  </template>
</template>
```

> [!warning] La cle `:key` est obligatoire
> La propriete `:key` permet a Vue de tracker les elements de la liste et d'optimiser le rendu. Sans elle, Vue re-rend tout le tableau a chaque changement. Utilisez toujours un identifiant unique (id de base de donnees, pas l'index du tableau si la liste est reorderable).

---

## 6. Composants

### Structure d'un composant Single-File (.vue)

```vue
<!-- src/components/UserCard.vue -->
<template>
  <div class="user-card">
    <img :src="user.avatar" :alt="user.name">
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
    <button @click="$emit('select', user)">Selectionner</button>
    <slot></slot>  <!-- Contenu passe par le parent -->
  </div>
</template>

<script setup>
// Props : donnees recues du parent
const props = defineProps({
  user: {
    type: Object,
    required: true,
    validator(value) {
      return value.name && value.email
    }
  }
})

// Emits : evenements envoyes au parent
const emit = defineEmits(['select'])
</script>

<style scoped>
/* scoped : ce CSS ne s'applique qu'a ce composant */
.user-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
}
</style>
```

### Utilisation du composant

```vue
<!-- src/views/UsersView.vue -->
<template>
  <div>
    <h1>Liste des utilisateurs</h1>
    
    <UserCard
      v-for="user in users"
      :key="user.id"
      :user="user"
      @select="onUserSelected"
    >
      <!-- Contenu du slot -->
      <badge v-if="user.isPremium">Premium</badge>
    </UserCard>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import UserCard from '@/components/UserCard.vue'

const users = ref([
  { id: 1, name: 'Alice', email: 'alice@example.com', avatar: '/alice.jpg', isPremium: true },
  { id: 2, name: 'Bob', email: 'bob@example.com', avatar: '/bob.jpg', isPremium: false }
])

function onUserSelected(user) {
  console.log('Utilisateur selectionne :', user.name)
}
</script>
```

### Props — Types et Validation

```vue
<script setup>
const props = defineProps({
  // Type simple
  title: String,
  
  // Requis
  id: {
    type: Number,
    required: true
  },
  
  // Valeur par defaut
  color: {
    type: String,
    default: 'blue'
  },
  
  // Plusieurs types
  value: [String, Number],
  
  // Validation personnalisee
  status: {
    type: String,
    validator(val) {
      return ['draft', 'published', 'archived'].includes(val)
    }
  },
  
  // Objet avec defaut
  config: {
    type: Object,
    default: () => ({ size: 'medium', rounded: false })
  }
})
</script>
```

### Slots

```vue
<!-- Composant Card.vue -->
<template>
  <div class="card">
    <!-- Slot nomme pour l'en-tete -->
    <header v-if="$slots.header">
      <slot name="header"></slot>
    </header>
    
    <!-- Slot par defaut -->
    <main>
      <slot>Contenu par defaut si rien n'est passe</slot>
    </main>
    
    <!-- Slot avec donnees (scoped slot) -->
    <footer>
      <slot name="footer" :closeCard="close"></slot>
    </footer>
  </div>
</template>

<!-- Utilisation -->
<Card>
  <template #header>
    <h2>Titre de la carte</h2>
  </template>
  
  <p>Contenu principal ici.</p>
  
  <template #footer="{ closeCard }">
    <button @click="closeCard">Fermer</button>
  </template>
</Card>
```

---

## 7. Computed Properties, Methods, et Watchers

### Computed Properties

```vue
<script setup>
import { ref, computed } from 'vue'

const items = ref([
  { id: 1, name: 'Pomme', prix: 1.20, categorie: 'fruit' },
  { id: 2, name: 'Carotte', prix: 0.80, categorie: 'legume' },
  { id: 3, name: 'Banane', prix: 0.90, categorie: 'fruit' }
])

const filtreCategorie = ref('tous')
const searchQuery = ref('')

// Computed : mise en cache, recalcul seulement si dependances changent
const itemsFiltres = computed(() => {
  let result = items.value
  
  if (filtreCategorie.value !== 'tous') {
    result = result.filter(item => item.categorie === filtreCategorie.value)
  }
  
  if (searchQuery.value) {
    const q = searchQuery.value.toLowerCase()
    result = result.filter(item => item.name.toLowerCase().includes(q))
  }
  
  return result
})

const totalPrix = computed(() => {
  return itemsFiltres.value.reduce((sum, item) => sum + item.prix, 0).toFixed(2)
})

// Computed writable (get + set)
const fullName = computed({
  get() {
    return `${prenom.value} ${nom.value}`
  },
  set(value) {
    const parts = value.split(' ')
    prenom.value = parts[0]
    nom.value = parts[1] || ''
  }
})
</script>
```

> [!info] Computed vs Method
> Les **computed** sont mis en cache : `itemsFiltres` ne se recalcule que quand `items`, `filtreCategorie`, ou `searchQuery` changent. Appele 100 fois dans le template → calcule une seule fois.
> Les **methods** sont recalculees a chaque rendu, meme si les donnees n'ont pas change. Utiliser les methods pour les actions (click, submit), les computed pour les valeurs derivees.

### Watchers

Les watchers permettent de reagir aux changements d'etat, notamment pour declencher des appels [[03 - JavaScript Asynchrone|async/await]] vers une [[08 - APIs REST avec Flask|API REST]].

```vue
<script setup>
import { ref, watch, watchEffect } from 'vue'

const userId = ref(1)
const userData = ref(null)
const isLoading = ref(false)

// watch : explicit, controle precis
watch(userId, async (newId, oldId) => {
  console.log(`userId change : ${oldId} → ${newId}`)
  isLoading.value = true
  const response = await fetch(`/api/users/${newId}`)
  userData.value = await response.json()
  isLoading.value = false
}, {
  immediate: true,  // Execute aussi au montage
  deep: true        // Observer les changements profonds dans un objet
})

// Plusieurs sources
watch([userId, searchQuery], ([newId, newQuery]) => {
  console.log('userId ou searchQuery a change')
})

// watchEffect : collecte automatiquement les dependances
watchEffect(async () => {
  // userId.value est accede → vue sait qu'il faut re-executer quand userId change
  const response = await fetch(`/api/users/${userId.value}`)
  userData.value = await response.json()
})
</script>
```

---

## 8. Lifecycle Hooks

```
Creation
  ├── setup()                     ← Composition API entry point
  │
Montage dans le DOM
  ├── onBeforeMount()             ← Avant l'insertion dans le DOM
  ├── onMounted()                 ← DOM pret, refs disponibles ✓
  │
Mises a jour reactives
  ├── onBeforeUpdate()            ← Avant la mise a jour du DOM
  ├── onUpdated()                 ← Apres la mise a jour du DOM
  │
Demontage
  ├── onBeforeUnmount()           ← Avant la destruction
  └── onUnmounted()               ← Nettoyage (timers, listeners) ✓
```

```vue
<script setup>
import {
  ref,
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted
} from 'vue'

const data = ref(null)
let intervalId = null

onBeforeMount(() => {
  // Le DOM n'existe pas encore — ne pas acceder aux refs template
  console.log('Avant le montage')
})

onMounted(async () => {
  // DOM disponible — parfait pour :
  // - Requetes API initiales
  // - Initialisation de bibliotheques tierces (Chart.js, etc.)
  // - Acces aux refs de template ($refs)
  data.value = await fetchData()
  
  // Demarrer un timer
  intervalId = setInterval(() => {
    console.log('Tick...')
  }, 1000)
})

onBeforeUnmount(() => {
  // Nettoyage OBLIGATOIRE : eviter les fuites memoire
  clearInterval(intervalId)
  // Retirer les event listeners ajoutes manuellement
  window.removeEventListener('resize', onResize)
})

onUnmounted(() => {
  console.log('Composant detruit, tout est nettoye')
})
</script>
```

> [!warning] Nettoyage dans `onBeforeUnmount`
> Tout ce que vous demarrez dans `onMounted` (setInterval, addEventListener, WebSocket, observer Intersection) doit etre arrete dans `onBeforeUnmount`. Sans cela, vous avez une fuite memoire : le callback continue de s'executer meme apres la destruction du composant.

---

## 9. Vue DevTools

Vue DevTools est une extension navigateur (Chrome, Firefox) indispensable pour le debug. Pour les tests automatises, consulter [[01 - Tests Unitaires et TDD|Tests Unitaires et TDD]] (Vitest + Vue Test Utils) et [[02 - Tests Integration et E2E|Tests Integration et E2E]] (Playwright) pour tester les composants Vue en isolation et en conditions reelles.

**Installation** : chercher "Vue.js devtools" dans le Chrome Web Store ou Firefox Add-ons.

**Fonctionnalites** :
- **Components** : arbre des composants, inspection des props/data/computed en temps reel
- **Timeline** : enregistrement des evenements, mutations, performances
- **Pinia** : inspection et mutation directe du store
- **Router** : historique de navigation, routes actives

---

## 10. Exemple Complet — CRUD Simple

Application de gestion de taches (Todo List) avec filtrage et persistence localStorage.

```vue
<!-- src/App.vue -->
<template>
  <div class="app">
    <h1>Mes Taches</h1>
    
    <!-- Formulaire d'ajout -->
    <form @submit.prevent="addTache">
      <input
        v-model.trim="nouvelleTache"
        placeholder="Nouvelle tache..."
        :disabled="isLoading"
      >
      <button type="submit" :disabled="!nouvelleTache || isLoading">
        Ajouter
      </button>
    </form>
    
    <!-- Filtres -->
    <div class="filtres">
      <button
        v-for="filtre in filtres"
        :key="filtre.value"
        :class="{ actif: filtreActif === filtre.value }"
        @click="filtreActif = filtre.value"
      >
        {{ filtre.label }}
      </button>
    </div>
    
    <!-- Statistiques (computed) -->
    <p>{{ tachesRestantes }} tache(s) restante(s) sur {{ taches.length }}</p>
    
    <!-- Liste -->
    <TransitionGroup name="liste" tag="ul">
      <li
        v-for="tache in tachesFiltrees"
        :key="tache.id"
        :class="{ completee: tache.completee }"
      >
        <input
          type="checkbox"
          :checked="tache.completee"
          @change="toggleTache(tache.id)"
        >
        
        <!-- Mode edition -->
        <template v-if="tache.id === editingId">
          <input
            v-model="editingText"
            @blur="saveEdit(tache.id)"
            @keyup.enter="saveEdit(tache.id)"
            @keyup.escape="cancelEdit"
          >
        </template>
        <span v-else @dblclick="startEdit(tache)">{{ tache.text }}</span>
        
        <button @click="deleteTache(tache.id)" class="btn-supprimer">×</button>
      </li>
    </TransitionGroup>
    
    <!-- Vide -->
    <p v-if="taches.length === 0" class="vide">
      Aucune tache. Ajoutez-en une !
    </p>
    
    <!-- Action en masse -->
    <button
      v-if="taches.some(t => t.completee)"
      @click="clearCompleted"
    >
      Supprimer les taches completees
    </button>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

// --- Etat ---
const taches = ref(loadFromStorage())
const nouvelleTache = ref('')
const filtreActif = ref('toutes')
const editingId = ref(null)
const editingText = ref('')
const isLoading = ref(false)

const filtres = [
  { value: 'toutes', label: 'Toutes' },
  { value: 'actives', label: 'Actives' },
  { value: 'completees', label: 'Completees' }
]

// --- Computed ---
const tachesFiltrees = computed(() => {
  switch (filtreActif.value) {
    case 'actives':    return taches.value.filter(t => !t.completee)
    case 'completees': return taches.value.filter(t => t.completee)
    default:           return taches.value
  }
})

const tachesRestantes = computed(
  () => taches.value.filter(t => !t.completee).length
)

// --- Actions ---
function addTache() {
  if (!nouvelleTache.value) return
  taches.value.push({
    id: Date.now(),
    text: nouvelleTache.value,
    completee: false,
    createdAt: new Date().toISOString()
  })
  nouvelleTache.value = ''
}

function toggleTache(id) {
  const tache = taches.value.find(t => t.id === id)
  if (tache) tache.completee = !tache.completee
}

function deleteTache(id) {
  taches.value = taches.value.filter(t => t.id !== id)
}

function startEdit(tache) {
  editingId.value = tache.id
  editingText.value = tache.text
}

function saveEdit(id) {
  const tache = taches.value.find(t => t.id === id)
  if (tache && editingText.value.trim()) {
    tache.text = editingText.value.trim()
  }
  cancelEdit()
}

function cancelEdit() {
  editingId.value = null
  editingText.value = ''
}

function clearCompleted() {
  taches.value = taches.value.filter(t => !t.completee)
}

// --- Persistence localStorage ---
function loadFromStorage() {
  try {
    return JSON.parse(localStorage.getItem('taches')) || []
  } catch {
    return []
  }
}

watch(taches, (val) => {
  localStorage.setItem('taches', JSON.stringify(val))
}, { deep: true })
</script>

<style scoped>
.app { max-width: 600px; margin: 0 auto; padding: 20px; }
.completee span { text-decoration: line-through; opacity: 0.5; }
.actif { font-weight: bold; border-bottom: 2px solid currentColor; }

/* Transition pour l'ajout/suppression d'elements */
.liste-enter-active, .liste-leave-active { transition: all 0.3s ease; }
.liste-enter-from { opacity: 0; transform: translateX(-30px); }
.liste-leave-to   { opacity: 0; transform: translateX(30px); }
</style>
```

---

## 11. Exercices Pratiques

> [!tip] Exercice 1 — Calculatrice reactive
> Cree un composant `Calculatrice.vue` avec :
> - Deux inputs numeriques (`v-model.number`)
> - Un select pour l'operateur (+, -, ×, ÷)
> - Un computed `resultat` qui calcule le resultat
> - Gestion de la division par zero (afficher "Erreur" via `v-if`)
> - Style avec class binding (resultat en rouge si erreur)

> [!tip] Exercice 2 — Composant de pagination
> Cree un composant `Pagination.vue` qui recoit en props :
> - `total` : nombre total d'items
> - `perPage` : items par page (defaut : 10)
> - `currentPage` : page actuelle
> Emets un evenement `page-change` quand l'utilisateur clique sur une page.
> Affiche "Precedent", les numeros de pages, "Suivant".
> La page actuelle doit etre mise en evidence avec un class binding.

> [!tip] Exercice 3 — Fetch et affichage de donnees
> Utilise `onMounted` et `watch` pour :
> 1. Fetcher `https://jsonplaceholder.typicode.com/users` au montage
> 2. Afficher un spinner pendant le chargement (`v-if="isLoading"`)
> 3. Afficher les utilisateurs dans une liste (`v-for`)
> 4. Ajouter un input de recherche qui filtre en temps reel (computed)
> 5. Afficher un message d'erreur si le fetch echoue (`try/catch`)

> [!tip] Exercice 4 — Formulaire multi-etapes avec validation
> Cree un formulaire en 3 etapes :
> - Etape 1 : Nom, Prenom (requis)
> - Etape 2 : Email (format valide), Mot de passe (min 8 chars)
> - Etape 3 : Resume de la saisie + bouton Confirmer
> Utilise `v-model`, des computed pour la validation, et `v-show` pour switcher les etapes.
> Un indicateur de progression doit montrer l'etape actuelle.

---

## Liens Utiles

- Documentation officielle Vue 3 : https://vuejs.org/guide/
- Vue DevTools : https://devtools.vuejs.org/
- Vite : https://vitejs.dev/

---

## Notes liées

- [[02 - VueJS Avance Pinia et Vue Router]] — Pinia pour la gestion d'etat avancee, Vue Router pour le routing SPA
- [[01 - Introduction a React]] — comparatif React vs Vue, hooks vs Composition API
- [[01 - Svelte du Debutant a l'Expert]] — alternative compilee a Vue, sans Virtual DOM
- [[05 - SPA et Frameworks Introduction]] — contexte general des SPAs et comparatif des frameworks
- [[04 - JavaScript Moderne ES6+]] — modules, destructuration, arrow functions utilises dans la Composition API
- [[03 - JavaScript Asynchrone]] — async/await, Promises, fetch pour les watchers et onMounted
- [[02 - CSS Fondamentaux]] — CSS scope avec `<style scoped>`, class bindings, animations Vue
- [[01 - Tests Unitaires et TDD]] — Vitest + Vue Test Utils pour tester les composants
- [[04 - CI-CD avec GitHub Actions]] — deploiement automatique des apps Vue/SvelteKit
- [[08 - APIs REST avec Flask]] — backend REST a consommer depuis les composants Vue
