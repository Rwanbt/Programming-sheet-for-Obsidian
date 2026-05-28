# VueJS Avance — Pinia et Vue Router

Suite de [[01 - Introduction a VueJS]]. Ce cours approfondit la Composition API, les composables, Vue Router 4 pour la navigation, Pinia pour la gestion d'etat globale, et les patterns avances de Vue 3.

> [!info] Pre-requis
> Ce cours suppose que vous maitrisez les bases de Vue 3 : `ref`, `computed`, `v-model`, `v-for`, les composants et les props. Relisez [[01 - Introduction a VueJS]] si necessaire.

---

## 1. Composition API Approfondie

### 1.1 `ref` et `reactive`

```vue
<script setup>
import { ref, reactive, toRefs, isRef, isReactive } from 'vue'

// ref : pour les valeurs primitives (string, number, boolean) et les objets
// Acces via .value en JS, automatique dans le template
const count = ref(0)
const name = ref('Alice')
const user = ref({ id: 1, name: 'Alice', roles: ['admin'] })

console.log(count.value)      // 0
count.value++                 // Mise a jour reactive
console.log(user.value.name)  // 'Alice'

// reactive : pour les objets complexes
// Pas de .value — acces direct aux proprietes
const state = reactive({
  count: 0,
  users: [],
  filters: { search: '', category: 'all' }
})

state.count++                           // Reactive
state.users.push({ id: 1, name: 'Bob' }) // Reactive (tableau)
state.filters.search = 'test'           // Reactive (nested)

// DANGER : ne jamais remplacer l'objet reactive entier
// ❌ state = { count: 5 }  — perd la reactivite
// ✅ Object.assign(state, { count: 5 })

// toRefs : convertir reactive en refs (pour la destructuration)
const { count: countRef, users } = toRefs(state)
// countRef.value, users.value — reactifs !

// Verifications utiles
console.log(isRef(count))      // true
console.log(isReactive(state)) // true
</script>
```

> [!info] `ref` vs `reactive` — quand utiliser lequel ?
> - **`ref`** : valeurs primitives, ou quand vous avez besoin de remplacer toute la valeur (ex. remplacer un tableau entier apres un fetch)
> - **`reactive`** : objets complexes avec plusieurs proprietes liees
> - **Recommandation Vue team** : utiliser `ref` par defaut — plus simple, moins de pieges

### 1.2 `computed`, `watch`, `watchEffect`

```vue
<script setup>
import { ref, computed, watch, watchEffect } from 'vue'

const items = ref([])
const searchQuery = ref('')
const sortOrder = ref('asc')

// computed avec getter et setter
const filteredItems = computed(() => {
  const q = searchQuery.value.toLowerCase()
  const result = items.value.filter(item =>
    item.name.toLowerCase().includes(q)
  )
  return sortOrder.value === 'asc'
    ? result.sort((a, b) => a.name.localeCompare(b.name))
    : result.sort((a, b) => b.name.localeCompare(a.name))
})

// watch — controle explicite
const stopWatch = watch(
  // Source : ref, reactive, getter function, ou tableau
  () => [searchQuery.value, sortOrder.value],
  ([newQuery, newSort], [oldQuery, oldSort]) => {
    console.log(`Recherche: "${oldQuery}" → "${newQuery}"`)
    console.log(`Tri: ${oldSort} → ${newSort}`)
  },
  {
    immediate: false,    // Ne pas executer au montage
    deep: false,         // Observer profondeur (objets imbriques)
    flush: 'post'        // 'pre' (defaut) | 'post' (apres render) | 'sync'
  }
)

// Arreter manuellement un watcher
stopWatch()

// watchEffect — automatique, collecte ses dependances
const stopEffect = watchEffect((onCleanup) => {
  // Toutes les refs accedees ici deviennent des dependances
  console.log(`Recherche: ${searchQuery.value}`)
  
  const timer = setTimeout(() => {
    fetchItems(searchQuery.value)
  }, 300) // Debounce
  
  // Cleanup : appele avant chaque re-execution et au demontage
  onCleanup(() => clearTimeout(timer))
})

async function fetchItems(query) {
  const response = await fetch(`/api/items?q=${query}`)
  items.value = await response.json()
}
</script>
```

### 1.3 Template Refs

```vue
<template>
  <input ref="inputRef" type="text">
  <canvas ref="canvasRef"></canvas>
  
  <!-- Ref sur un composant enfant -->
  <MonComposant ref="childRef"></MonComposant>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const inputRef = ref(null)  // null jusqu'au montage
const canvasRef = ref(null)
const childRef = ref(null)

onMounted(() => {
  // DOM disponible apres onMounted
  inputRef.value.focus()
  
  const ctx = canvasRef.value.getContext('2d')
  ctx.fillRect(0, 0, 100, 100)
  
  // Appeler une methode exposee par le composant enfant
  childRef.value.reset()
})
</script>
```

### 1.4 `defineExpose` — Exposer des methodes au parent

```vue
<!-- MonComposant.vue -->
<script setup>
import { ref } from 'vue'

const internalCount = ref(0)

// Par defaut, <script setup> rend tout PRIVE
// defineExpose rend des elements accessibles via template ref
defineExpose({
  reset() {
    internalCount.value = 0
  },
  increment() {
    internalCount.value++
  },
  // Exposer en lecture seule
  count: internalCount
})
</script>
```

---

## 2. Composables — L'equivalent des Custom Hooks React

Un composable est une fonction qui encapsule de la logique reactive reutilisable avec la Composition API.

### 2.1 Composable simple — `useCounter`

```javascript
// src/composables/useCounter.js
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0, options = {}) {
  const { min = -Infinity, max = Infinity, step = 1 } = options
  
  const count = ref(initialValue)
  
  const isAtMin = computed(() => count.value <= min)
  const isAtMax = computed(() => count.value >= max)
  
  function increment() {
    if (!isAtMax.value) count.value = Math.min(count.value + step, max)
  }
  
  function decrement() {
    if (!isAtMin.value) count.value = Math.max(count.value - step, min)
  }
  
  function reset() {
    count.value = initialValue
  }
  
  return { count, isAtMin, isAtMax, increment, decrement, reset }
}
```

```vue
<!-- Utilisation -->
<script setup>
import { useCounter } from '@/composables/useCounter'

const { count, increment, decrement, reset } = useCounter(0, { min: 0, max: 10 })
</script>
```

### 2.2 Composable avance — `useFetch`

```javascript
// src/composables/useFetch.js
import { ref, watch, toValue } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)
  const isLoading = ref(false)

  async function fetchData() {
    // toValue() resout ref, reactive, ou valeur brute
    const resolvedUrl = toValue(url)
    if (!resolvedUrl) return
    
    data.value = null
    error.value = null
    isLoading.value = true
    
    try {
      const response = await fetch(resolvedUrl)
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`)
      }
      data.value = await response.json()
    } catch (e) {
      error.value = e.message
    } finally {
      isLoading.value = false
    }
  }
  
  // Re-fetch si l'URL change (supporte les refs)
  watch(() => toValue(url), fetchData, { immediate: true })
  
  return { data, error, isLoading, refetch: fetchData }
}
```

```vue
<!-- Utilisation -->
<script setup>
import { ref } from 'vue'
import { useFetch } from '@/composables/useFetch'

const userId = ref(1)

// URL reactive : se re-fetche quand userId change
const { data: user, error, isLoading } = useFetch(
  () => `https://jsonplaceholder.typicode.com/users/${userId.value}`
)
</script>

<template>
  <div v-if="isLoading">Chargement...</div>
  <div v-else-if="error" class="error">{{ error }}</div>
  <div v-else-if="user">{{ user.name }}</div>
</template>
```

### 2.3 Composable — `useLocalStorage`

```javascript
// src/composables/useLocalStorage.js
import { ref, watch } from 'vue'

export function useLocalStorage(key, defaultValue) {
  function readValue() {
    try {
      const item = localStorage.getItem(key)
      return item ? JSON.parse(item) : defaultValue
    } catch {
      return defaultValue
    }
  }
  
  const storedValue = ref(readValue())
  
  watch(storedValue, (val) => {
    try {
      localStorage.setItem(key, JSON.stringify(val))
    } catch (e) {
      console.error('Erreur localStorage:', e)
    }
  }, { deep: true })
  
  return storedValue
}
```

---

## 3. Vue Router 4

Vue Router est la solution officielle de routing pour Vue.js.

### 3.1 Installation et configuration

```bash
npm install vue-router@4
```

```javascript
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'

// Import direct (pas de lazy loading — bundle plus grand)
import HomeView from '@/views/HomeView.vue'

// Lazy loading : le composant est charge seulement quand la route est visitee
// Cree un chunk JS separe (code splitting automatique via Vite)
const router = createRouter({
  // createWebHistory : URLs propres (/about) — necessite config serveur
  // createWebHashHistory : URLs avec hash (/#/about) — pas de config serveur
  history: createWebHistory(import.meta.env.BASE_URL),
  
  routes: [
    // Route simple
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    
    // Lazy loading
    {
      path: '/about',
      name: 'about',
      component: () => import('@/views/AboutView.vue')
    },
    
    // Route dynamique — :id est un parametre
    {
      path: '/users/:id',
      name: 'user-detail',
      component: () => import('@/views/UserDetailView.vue'),
      // Passer les params comme props au composant
      props: true
    },
    
    // Route imbriquee (nested)
    {
      path: '/dashboard',
      component: () => import('@/views/DashboardView.vue'),
      // Proteger la route avec un guard
      meta: { requiresAuth: true },
      children: [
        {
          path: '',          // /dashboard
          name: 'dashboard',
          component: () => import('@/views/dashboard/OverviewView.vue')
        },
        {
          path: 'profile',   // /dashboard/profile
          name: 'dashboard-profile',
          component: () => import('@/views/dashboard/ProfileView.vue')
        },
        {
          path: 'settings',  // /dashboard/settings
          name: 'dashboard-settings',
          component: () => import('@/views/dashboard/SettingsView.vue')
        }
      ]
    },
    
    // Redirection
    {
      path: '/home',
      redirect: '/'
    },
    
    // Route 404 (catch-all)
    {
      path: '/:pathMatch(.*)*',
      name: 'not-found',
      component: () => import('@/views/NotFoundView.vue')
    }
  ],
  
  // Comportement du scroll entre les pages
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition  // Retour en arriere : position sauvegardee
    }
    if (to.hash) {
      return { el: to.hash, behavior: 'smooth' }
    }
    return { top: 0 }       // Haut de page par defaut
  }
})

export default router
```

```javascript
// src/main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.use(createPinia())
app.use(router)
app.mount('#app')
```

### 3.2 `RouterView` et `RouterLink`

```vue
<!-- src/App.vue -->
<template>
  <nav>
    <!-- RouterLink genere un <a> sans rechargement de page -->
    <RouterLink to="/">Accueil</RouterLink>
    <RouterLink :to="{ name: 'about' }">A propos</RouterLink>
    <RouterLink :to="{ name: 'user-detail', params: { id: 42 } }">
      Mon profil
    </RouterLink>
    
    <!-- active-class : classe ajoutee quand le lien est actif -->
    <!-- exact-active-class : seulement si correspondance exacte -->
    <RouterLink
      to="/dashboard"
      active-class="menu-actif"
      exact-active-class="menu-exact"
    >
      Dashboard
    </RouterLink>
  </nav>
  
  <!-- RouterView : affiche le composant de la route actuelle -->
  <!-- La transition enveloppe les changements de route -->
  <RouterView v-slot="{ Component }">
    <Transition name="fade" mode="out-in">
      <component :is="Component" :key="$route.path" />
    </Transition>
  </RouterView>
</template>

<style>
.fade-enter-active, .fade-leave-active { transition: opacity 0.2s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>
```

### 3.3 Navigation programmatique et `useRoute` / `useRouter`

```vue
<script setup>
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()    // Route actuelle (reactive)
const router = useRouter()  // Instance du router

// Informations de la route actuelle
console.log(route.path)              // '/users/42'
console.log(route.params.id)         // '42' (toujours une string)
console.log(route.query.page)        // '2' (depuis ?page=2)
console.log(route.name)              // 'user-detail'
console.log(route.meta.requiresAuth) // true

// Navigation programmatique
function goToHome() {
  router.push('/')
}

function goToUser(id) {
  router.push({ name: 'user-detail', params: { id } })
}

function goBack() {
  router.back()
}

function goForward() {
  router.forward()
}

// replace : comme push mais sans ajouter a l'historique
function replaceRoute() {
  router.replace({ name: 'home' })
}

// Avec query params
function search(query) {
  router.push({ path: '/search', query: { q: query, page: 1 } })
}
</script>
```

### 3.4 Navigation Guards

```javascript
// src/router/index.js

// Guard global : execute avant CHAQUE navigation
router.beforeEach(async (to, from) => {
  const authStore = useAuthStore()
  
  // Verifier l'authentification pour les routes protegees
  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    // Sauvegarder la destination pour rediriger apres login
    return {
      name: 'login',
      query: { redirect: to.fullPath }
    }
  }
  
  // Verifier les permissions de role
  if (to.meta.role && !authStore.hasRole(to.meta.role)) {
    return { name: 'forbidden' }
  }
  
  // Continuer la navigation (retourner undefined ou true)
})

// Guard apres navigation (analytics, titre de page)
router.afterEach((to) => {
  document.title = to.meta.title
    ? `${to.meta.title} — Mon App`
    : 'Mon App'
  
  // Tracking analytics
  analytics.track('page_view', { path: to.path })
})
```

```vue
<!-- Guard au niveau du composant -->
<script setup>
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

const hasUnsavedChanges = ref(false)

// Avant de quitter cette route
onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges.value) {
    const confirmed = confirm('Vous avez des modifications non sauvegardees. Quitter quand meme ?')
    if (!confirmed) return false  // Annule la navigation
  }
})

// Quand les params changent sur la meme route (/users/1 → /users/2)
onBeforeRouteUpdate(async (to, from) => {
  if (to.params.id !== from.params.id) {
    await loadUser(to.params.id)
  }
})
</script>
```

---

## 4. Pinia — Gestion d'etat Globale

Pinia est le store officiel de Vue 3, successeur de Vuex. Plus simple, plus TypeScript-friendly, et sans les mutations boilerplate de Vuex.

### 4.1 Comparaison Vuex vs Pinia

| Aspect | Vuex 4 | Pinia |
|---|---|---|
| **Mutations** | Requises (boilerplate) | Supprimees |
| **Actions** | Asynchrones seulement | Synchrones et async |
| **Modules** | Configuration complexe | Stores independants |
| **TypeScript** | Support laborieux | Excellent support natif |
| **DevTools** | Oui | Oui (meilleure integration) |
| **Taille** | ~10 KB | ~1 KB |
| **Composition API** | Non | Oui (Option + Setup) |
| **Hot reload** | Partiel | Complet |

### 4.2 Definir un Store

```javascript
// src/stores/auth.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

// Option 1 : Syntax Setup (recommande — comme la Composition API)
export const useAuthStore = defineStore('auth', () => {
  // state : refs
  const user = ref(null)
  const token = ref(localStorage.getItem('token') || null)
  const isLoading = ref(false)
  
  // getters : computed
  const isAuthenticated = computed(() => !!token.value)
  const userRole = computed(() => user.value?.role || 'guest')
  const hasRole = computed(() => (role) => {
    return user.value?.roles?.includes(role) || false
  })
  
  // actions : fonctions
  async function login(email, password) {
    isLoading.value = true
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      })
      
      if (!response.ok) throw new Error('Identifiants incorrects')
      
      const data = await response.json()
      token.value = data.token
      user.value = data.user
      localStorage.setItem('token', data.token)
    } finally {
      isLoading.value = false
    }
  }
  
  function logout() {
    user.value = null
    token.value = null
    localStorage.removeItem('token')
  }
  
  async function fetchCurrentUser() {
    if (!token.value) return
    const response = await fetch('/api/auth/me', {
      headers: { Authorization: `Bearer ${token.value}` }
    })
    if (response.ok) {
      user.value = await response.json()
    } else {
      logout()
    }
  }
  
  return { user, token, isLoading, isAuthenticated, userRole, hasRole, login, logout, fetchCurrentUser }
})
```

```javascript
// Option 2 : Syntax Options (similaire Options API)
export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0,
    history: []
  }),
  
  getters: {
    doubleCount: (state) => state.count * 2,
    // Getter qui prend un argument
    countPlusN: (state) => (n) => state.count + n
  },
  
  actions: {
    increment() {
      this.history.push(this.count)
      this.count++
    },
    async fetchCount() {
      const response = await fetch('/api/count')
      this.count = await response.json()
    },
    // Reset built-in (Options syntax uniquement)
    reset() {
      this.$reset()
    }
  }
})
```

### 4.3 Utilisation dans les composants

```vue
<script setup>
import { storeToRefs } from 'pinia'
import { useAuthStore } from '@/stores/auth'
import { useCounterStore } from '@/stores/counter'

const authStore = useAuthStore()
const counterStore = useCounterStore()

// storeToRefs : extraire les refs/computed sans perdre la reactivite
// Les actions sont des fonctions normales — destructurer directement
const { user, isAuthenticated, isLoading } = storeToRefs(authStore)
const { login, logout, fetchCurrentUser } = authStore

// Acces direct (sans destructuration)
// authStore.user, authStore.isAuthenticated, authStore.login()

// Modifier le state directement (simple)
// authStore.user = null  ← Possible avec Pinia !

// Pour des modifications complexes : $patch
authStore.$patch({
  user: { ...authStore.user, name: 'Nouveau nom' }
})

// $patch avec fonction (pour des modifications conditionnelles)
authStore.$patch((state) => {
  if (state.user) {
    state.user.lastSeen = new Date().toISOString()
  }
})

// S'abonner aux changements du store
const unsubscribe = authStore.$subscribe((mutation, state) => {
  console.log('Store modifie:', mutation.type, state)
})
</script>
```

### 4.4 Stores imbriques et composition

```javascript
// src/stores/cart.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { useAuthStore } from './auth'  // Utiliser un autre store

export const useCartStore = defineStore('cart', () => {
  const authStore = useAuthStore()  // Composition de stores
  
  const items = ref([])
  
  const total = computed(() =>
    items.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
  )
  
  const itemCount = computed(() =>
    items.value.reduce((sum, item) => sum + item.quantity, 0)
  )
  
  async function addItem(product) {
    if (!authStore.isAuthenticated) {
      throw new Error('Connexion requise pour ajouter au panier')
    }
    
    const existing = items.value.find(i => i.id === product.id)
    if (existing) {
      existing.quantity++
    } else {
      items.value.push({ ...product, quantity: 1 })
    }
    
    // Synchroniser avec le backend
    await syncCart()
  }
  
  function removeItem(productId) {
    items.value = items.value.filter(i => i.id !== productId)
  }
  
  async function syncCart() {
    await fetch('/api/cart', {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${authStore.token}`
      },
      body: JSON.stringify({ items: items.value })
    })
  }
  
  return { items, total, itemCount, addItem, removeItem }
})
```

---

## 5. Patterns Avances

### 5.1 Teleport

```vue
<template>
  <button @click="isOpen = true">Ouvrir la modal</button>
  
  <!-- Teleport rend le contenu dans un autre element du DOM -->
  <!-- Utile pour les modals, tooltips, drawers qui doivent etre en dehors du composant parent -->
  <Teleport to="body">
    <div v-if="isOpen" class="modal-overlay" @click.self="isOpen = false">
      <div class="modal">
        <h2>Titre de la modal</h2>
        <p>Contenu de la modal rendu directement dans &lt;body&gt;</p>
        <button @click="isOpen = false">Fermer</button>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { ref } from 'vue'
const isOpen = ref(false)
</script>
```

### 5.2 `provide` / `inject` — Injection de dependances

```vue
<!-- Composant parent (ou App.vue) -->
<script setup>
import { provide, ref } from 'vue'

const theme = ref('light')
const language = ref('fr')

// Fournir des valeurs a tous les descendants
provide('theme', theme)
provide('language', language)

// Fournir des fonctions aussi
provide('setTheme', (newTheme) => {
  theme.value = newTheme
})
</script>
```

```vue
<!-- Composant descendant (n'importe quelle profondeur) -->
<script setup>
import { inject } from 'vue'

// Injecter les valeurs (avec valeur par defaut)
const theme = inject('theme', ref('light'))
const language = inject('language', ref('fr'))
const setTheme = inject('setTheme', () => {})

function toggleTheme() {
  setTheme(theme.value === 'light' ? 'dark' : 'light')
}
</script>
```

> [!info] `provide/inject` vs Pinia
> - **`provide/inject`** : pour partager des donnees entre un composant parent et ses descendants. Ideal pour le theming, la configuration, les services qui ont une portee locale.
> - **Pinia** : pour l'etat global partagé entre plusieurs parties de l'application sans lien parent-enfant.

### 5.3 Plugins Vue

```javascript
// src/plugins/i18n.js
export const i18nPlugin = {
  install(app, options) {
    const translations = options.translations
    const currentLocale = ref(options.defaultLocale || 'fr')
    
    // Methode globale : accessible via this.$t() ou inject('t')
    app.config.globalProperties.$t = function(key) {
      return translations[currentLocale.value]?.[key] || key
    }
    
    // Directive globale
    app.directive('translate', {
      mounted(el, binding) {
        el.textContent = translations[currentLocale.value]?.[binding.value] || binding.value
      }
    })
    
    // Provide pour la Composition API
    app.provide('setLocale', (locale) => { currentLocale.value = locale })
    app.provide('currentLocale', currentLocale)
  }
}

// src/main.js
app.use(i18nPlugin, {
  defaultLocale: 'fr',
  translations: {
    fr: { greeting: 'Bonjour', farewell: 'Au revoir' },
    en: { greeting: 'Hello', farewell: 'Goodbye' }
  }
})
```

---

## 6. Tests avec Vitest et Vue Test Utils

### 6.1 Configuration

```bash
npm install -D vitest @vue/test-utils jsdom @vitest/coverage-v8
```

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'jsdom',  // Simule le DOM dans Node.js
    globals: true,          // describe, it, expect sans import
    setupFiles: ['./src/tests/setup.js']
  }
})
```

### 6.2 Tester un composant

```javascript
// src/tests/components/UserCard.test.js
import { mount, shallowMount } from '@vue/test-utils'
import { describe, it, expect, vi } from 'vitest'
import UserCard from '@/components/UserCard.vue'

describe('UserCard', () => {
  const mockUser = {
    id: 1,
    name: 'Alice Dupont',
    email: 'alice@example.com',
    avatar: '/alice.jpg'
  }
  
  it('affiche le nom et l\'email de l\'utilisateur', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser }
    })
    
    expect(wrapper.text()).toContain('Alice Dupont')
    expect(wrapper.text()).toContain('alice@example.com')
  })
  
  it('emet l\'evenement "select" au clic du bouton', async () => {
    const wrapper = mount(UserCard, { props: { user: mockUser } })
    
    await wrapper.find('button').trigger('click')
    
    expect(wrapper.emitted('select')).toBeTruthy()
    expect(wrapper.emitted('select')[0]).toEqual([mockUser])
  })
  
  it('affiche le slot par defaut', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser },
      slots: { default: '<span class="badge">Premium</span>' }
    })
    
    expect(wrapper.find('.badge').exists()).toBe(true)
  })
})
```

### 6.3 Tester un store Pinia

```javascript
// src/tests/stores/auth.test.js
import { setActivePinia, createPinia } from 'pinia'
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { useAuthStore } from '@/stores/auth'

// Mock fetch global
global.fetch = vi.fn()

describe('useAuthStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
    vi.clearAllMocks()
    localStorage.clear()
  })
  
  it('est deconnecte par defaut', () => {
    const store = useAuthStore()
    expect(store.isAuthenticated).toBe(false)
    expect(store.user).toBeNull()
  })
  
  it('connecte un utilisateur avec succes', async () => {
    const mockResponse = {
      token: 'fake-jwt-token',
      user: { id: 1, name: 'Alice', roles: ['user'] }
    }
    
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockResponse
    })
    
    const store = useAuthStore()
    await store.login('alice@example.com', 'password123')
    
    expect(store.isAuthenticated).toBe(true)
    expect(store.user.name).toBe('Alice')
    expect(localStorage.getItem('token')).toBe('fake-jwt-token')
  })
  
  it('deconnecte et nettoie le state', async () => {
    const store = useAuthStore()
    store.user = { id: 1, name: 'Alice' }
    store.token = 'some-token'
    
    store.logout()
    
    expect(store.isAuthenticated).toBe(false)
    expect(store.user).toBeNull()
    expect(localStorage.getItem('token')).toBeNull()
  })
})
```

---

## 7. Introduction a Nuxt.js (SSR avec Vue)

Nuxt.js est le framework meta de Vue, equivalent de Next.js pour React.

### 7.1 Differences Vue (SPA) vs Nuxt (SSR/SSG)

| Aspect | Vue SPA | Nuxt SSR | Nuxt SSG |
|---|---|---|---|
| **Rendu** | Client uniquement | Serveur + Client | Pre-genere au build |
| **SEO** | Mauvais | Excellent | Excellent |
| **TTFB** | Eleve (JS charge) | Rapide | Tres rapide |
| **Serveur requis** | Non | Oui (Node.js) | Non (CDN) |
| **Use case** | App connectee, dashboard | Blog, e-commerce, marketing | Sites statiques |

### 7.2 Structure Nuxt 3

```bash
npx nuxi@latest init mon-nuxt-app
cd mon-nuxt-app && npm install && npm run dev
```

```
mon-nuxt-app/
├── pages/
│   ├── index.vue          → Route /
│   ├── about.vue          → Route /about
│   └── users/
│       ├── index.vue      → Route /users
│       └── [id].vue       → Route /users/:id (dynamique)
├── components/            → Auto-imported
├── composables/           → Auto-imported
├── server/
│   └── api/
│       └── users.get.ts   → Endpoint GET /api/users
├── layouts/
│   └── default.vue        → Layout par defaut
└── nuxt.config.ts
```

```vue
<!-- pages/users/[id].vue -->
<script setup>
// useRoute et fetch automatiques dans Nuxt
const route = useRoute()

// useFetch : fetch hybride serveur + client avec cache
const { data: user, pending, error } = await useFetch(
  `/api/users/${route.params.id}`
)

// definePageMeta : metadonnees de la page
definePageMeta({
  layout: 'dashboard',
  middleware: 'auth'
})

// useSeoMeta : SEO automatique
useSeoMeta({
  title: () => `Profil de ${user.value?.name}`,
  description: () => `Page de profil de ${user.value?.name}`
})
</script>
```

---

## 8. Exercices Pratiques

> [!tip] Exercice 1 — Composable `useForm`
> Cree un composable `useForm(initialValues, validationRules)` qui :
> - Gere les valeurs du formulaire avec `reactive`
> - Valide chaque champ avec les regles fournies
> - Expose `errors`, `isValid`, `values`, `handleChange`, `handleSubmit`
> - Remet le formulaire a zero avec `reset()`
> Tester avec un formulaire d'inscription (nom, email, mot de passe).

> [!tip] Exercice 2 — Application CRUD avec Pinia + Vue Router
> Construis une app de gestion de contacts :
> - **Store Pinia** : `useContactsStore` avec state, getters, et actions CRUD
> - **Routes** : `/contacts` (liste), `/contacts/new` (creation), `/contacts/:id` (detail), `/contacts/:id/edit` (edition)
> - **Navigation guard** : rediriger vers `/` si un contact inexistant est demande
> - **RouterLink** avec classes actives dans un menu lateral
> - Persistence localStorage via `$subscribe`

> [!tip] Exercice 3 — Tests complets
> Ajoute des tests pour l'application de l'exercice 2 :
> - Test du store : CRUD, cas d'erreur, persistence
> - Test du composant liste : rendu, filtre de recherche, click
> - Test du composant detail : affichage, navigation au retour
> - Objectif : couverture > 80% sur les stores et composants metier

> [!tip] Exercice 4 — Mini clone de Notion avec Nuxt
> Avec Nuxt 3 :
> - Page d'accueil SSR avec liste de notes
> - Route dynamique `/notes/[id]` avec chargement SSR
> - API route `server/api/notes.get.ts` et `server/api/notes/[id].put.ts`
> - Auth middleware qui verifie un token dans les cookies
> - SEO avec `useSeoMeta` pour chaque note

---

## Liens Utiles

- Pinia : https://pinia.vuejs.org/
- Vue Router : https://router.vuejs.org/
- Vue Test Utils : https://test-utils.vuejs.org/
- Nuxt 3 : https://nuxt.com/
- Voir aussi : [[01 - Introduction a VueJS]]
