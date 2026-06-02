# Redux Avancé — Middleware et Architecture

Redux est la bibliothèque de gestion d'état la plus répandue dans l'écosystème React, mais sa puissance réelle ne se révèle qu'avec Redux Toolkit, le middleware asynchrone et une architecture rigoureuse. Ce cours couvre l'ensemble des patterns avancés utilisés en production, de RTK Query à Redux Saga, en passant par la normalisation des données et le testing.

> [!info] Prérequis
> Ce cours suppose que vous connaissez React, les hooks (`useState`, `useEffect`, `useContext`), les bases de Redux (store, actions, reducers) et TypeScript fondamental. Si ce n'est pas le cas, revenez sur le cours **React Hooks & Context** et **TypeScript Fondamentaux** avant de continuer.

---

## 1. Rappel Redux — Le socle fondamental

### 1.1 Les trois principes de Redux

Redux repose sur trois principes immuables qui guident toute son architecture.

**1. Source unique de vérité (Single Source of Truth)**
L'état global de l'application est stocké dans un unique objet JavaScript dans un seul store.

**2. L'état est en lecture seule**
La seule façon de modifier l'état est d'émettre une *action*, un objet décrivant ce qui s'est passé.

**3. Les changements se font par des fonctions pures**
Les *reducers* sont des fonctions pures qui prennent l'état précédent et une action, et retournent le nouvel état.

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLUX UNIDIRECTIONNEL REDUX                   │
│                                                                 │
│   ┌──────────┐    dispatch    ┌──────────┐    new state        │
│   │   View   │ ─────────────▶│  Action  │ ────────────────┐   │
│   │ (React)  │               │ {type,   │                 │   │
│   └──────────┘               │  payload}│                 │   │
│         ▲                    └──────────┘                 ▼   │
│         │                          │               ┌─────────┐ │
│    subscribe                       │               │ Reducer │ │
│         │                    ┌─────▼─────┐         │ (pure   │ │
│   ┌─────┴────┐               │  Store    │◀────────│  func)  │ │
│   │ getState │               │ (state)   │         └─────────┘ │
│   └──────────┘               └───────────┘                     │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Redux vanilla — La version sans toolkit

Pour comprendre ce que RTK simplifie, voici Redux dans sa forme la plus brute.

```typescript
// --- Actions types ---
const ADD_TODO = 'todos/ADD';
const TOGGLE_TODO = 'todos/TOGGLE';
const SET_LOADING = 'todos/SET_LOADING';

// --- Action creators ---
const addTodo = (text: string) => ({
  type: ADD_TODO,
  payload: { id: Date.now(), text, completed: false }
});

const toggleTodo = (id: number) => ({
  type: TOGGLE_TODO,
  payload: id
});

// --- État initial ---
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface TodoState {
  items: Todo[];
  loading: boolean;
  error: string | null;
}

const initialState: TodoState = {
  items: [],
  loading: false,
  error: null
};

// --- Reducer ---
function todosReducer(state = initialState, action: any): TodoState {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        // On DOIT créer un nouveau tableau — jamais muter l'existant
        items: [...state.items, action.payload]
      };
    case TOGGLE_TODO:
      return {
        ...state,
        items: state.items.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    case SET_LOADING:
      return { ...state, loading: action.payload };
    default:
      return state;
  }
}

// --- Store ---
import { createStore } from 'redux';
const store = createStore(todosReducer);
```

> [!warning] Le problème du vanilla Redux
> Avec des états complexes imbriqués, le spread operator devient un cauchemar :
> ```typescript
> // État profondément imbriqué — très verbeux et error-prone
> return {
>   ...state,
>   users: {
>     ...state.users,
>     byId: {
>       ...state.users.byId,
>       [action.payload.id]: {
>         ...state.users.byId[action.payload.id],
>         profile: {
>           ...state.users.byId[action.payload.id].profile,
>           name: action.payload.name
>         }
>       }
>     }
>   }
> };
> ```
> C'est pour résoudre ce problème (entre autres) que Redux Toolkit a été créé.

---

## 2. Redux Toolkit — Le standard moderne

### 2.1 Installation et configuration de base

Redux Toolkit (RTK) est la façon officielle recommandée d'écrire du Redux depuis 2020. Il embarque Immer, Reselect et plusieurs utilitaires essentiels.

```bash
# Installation avec React
npm install @reduxjs/toolkit react-redux

# Avec TypeScript (types inclus dans RTK)
# Aucun @types/* supplémentaire requis
```

### 2.2 `configureStore` — Le store amélioré

`configureStore` remplace `createStore` avec des defaults sensibles : Redux DevTools activé, middleware Thunk inclus, et détection de mutations accidentelles en développement.

```typescript
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import todosReducer from '../features/todos/todosSlice';
import authReducer from '../features/auth/authSlice';
import cartReducer from '../features/cart/cartSlice';

export const store = configureStore({
  reducer: {
    // Chaque clé devient un "slice" de l'état global
    todos: todosReducer,
    auth: authReducer,
    cart: cartReducer
  },
  // middleware est configuré automatiquement avec thunk + serializability check
  // Pour ajouter des middleware custom :
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      // Configuration Immer (détection de mutations)
      immutableStateInvariant: { enabled: true },
      // Vérification de sérialisation (utile pour DevTools)
      serializableCheck: {
        // Ignorer certains chemins si vous stockez des instances non-sérialisables
        ignoredActions: ['auth/setToken'],
        ignoredPaths: ['auth.tokenExpiry']
      }
    })
    // .concat(loggerMiddleware)  // Ajouter des middleware custom ici
    // .concat(sagaMiddleware)    // Pour Redux Saga
});

// Types TypeScript inférés automatiquement depuis le store
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// Hooks typés — à utiliser dans les composants à la place des hooks génériques
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

```typescript
// main.tsx — Point d'entrée
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import { store } from './store';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    {/* Provider injecte le store dans tout l'arbre React via Context */}
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

### 2.3 `createSlice` — La révolution RTK

`createSlice` est la pièce centrale de RTK. Il génère automatiquement les action creators et les action types à partir des reducers définis.

```typescript
// features/todos/todosSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
  priority: 'low' | 'medium' | 'high';
  createdAt: string;
}

interface TodosState {
  items: Todo[];
  filter: 'all' | 'active' | 'completed';
  loading: boolean;
  error: string | null;
}

const initialState: TodosState = {
  items: [],
  filter: 'all',
  loading: false,
  error: null
};

const todosSlice = createSlice({
  name: 'todos', // Préfixe pour les types d'actions : 'todos/addTodo'
  initialState,
  reducers: {
    // Grâce à Immer, on peut SEMBLER muter l'état directement
    addTodo: {
      // prepare() permet de préparer le payload avant le reducer
      prepare: (text: string, priority: Todo['priority'] = 'medium') => ({
        payload: {
          id: crypto.randomUUID(),
          text,
          completed: false,
          priority,
          createdAt: new Date().toISOString()
        }
      }),
      reducer: (state, action: PayloadAction<Todo>) => {
        // Immer traduit cette "mutation" en opération immuable
        state.items.push(action.payload);
      }
    },

    toggleTodo: (state, action: PayloadAction<string>) => {
      const todo = state.items.find(t => t.id === action.payload);
      if (todo) {
        // Mutation directe — Immer crée un nouvel objet en coulisse
        todo.completed = !todo.completed;
      }
    },

    deleteTodo: (state, action: PayloadAction<string>) => {
      // filter() retourne un nouveau tableau — les deux styles sont OK avec Immer
      state.items = state.items.filter(t => t.id !== action.payload);
    },

    updatePriority: (
      state,
      action: PayloadAction<{ id: string; priority: Todo['priority'] }>
    ) => {
      const todo = state.items.find(t => t.id === action.payload.id);
      if (todo) {
        todo.priority = action.payload.priority;
      }
    },

    setFilter: (state, action: PayloadAction<TodosState['filter']>) => {
      state.filter = action.payload;
    },

    clearCompleted: (state) => {
      state.items = state.items.filter(t => !t.completed);
    },

    // Reset complet — retourner un nouveau state désactive Immer pour ce reducer
    resetTodos: () => initialState
  },

  // extraReducers permet de réagir aux actions définies AILLEURS
  // (ex: thunks async, actions d'autres slices)
  extraReducers: (builder) => {
    // On verra ceci en détail dans la section Thunk
  }
});

// Les action creators sont générés automatiquement
export const {
  addTodo,
  toggleTodo,
  deleteTodo,
  updatePriority,
  setFilter,
  clearCompleted,
  resetTodos
} = todosSlice.actions;

// Le reducer est exporté par défaut pour le store
export default todosSlice.reducer;

// Sélecteurs de base (on verra les sélecteurs avancés avec Reselect)
export const selectAllTodos = (state: { todos: TodosState }) => state.todos.items;
export const selectFilter = (state: { todos: TodosState }) => state.todos.filter;
export const selectLoading = (state: { todos: TodosState }) => state.todos.loading;
```

### 2.4 `createAction` et `createReducer` — L'approche alternative

Quand vous n'avez pas besoin d'un slice complet, ces utilitaires permettent plus de flexibilité.

```typescript
import { createAction, createReducer, PayloadAction } from '@reduxjs/toolkit';

// createAction génère un action creator typé avec une méthode .toString()
const increment = createAction<number>('counter/increment');
const decrement = createAction<number>('counter/decrement');
const reset = createAction('counter/reset');

// L'action creator retourne { type: 'counter/increment', payload: 5 }
console.log(increment(5)); // { type: 'counter/increment', payload: 5 }
console.log(increment.type); // 'counter/increment'
console.log(increment.toString()); // 'counter/increment' — pratique pour les switch

interface CounterState {
  value: number;
  history: number[];
}

// createReducer utilise Immer nativement
const counterReducer = createReducer({ value: 0, history: [] } as CounterState, (builder) => {
  builder
    .addCase(increment, (state, action) => {
      state.value += action.payload;
      state.history.push(state.value);
    })
    .addCase(decrement, (state, action) => {
      state.value -= action.payload;
      state.history.push(state.value);
    })
    .addCase(reset, (state) => {
      state.value = 0;
      state.history = [];
    })
    // addMatcher — réagit à plusieurs actions avec un prédicat
    .addMatcher(
      (action) => action.type.endsWith('/rejected'),
      (state, action) => {
        console.error('Une action a échoué:', action.type);
      }
    )
    // addDefaultCase — fallback pour toutes les actions non gérées
    .addDefaultCase((state, action) => {
      // Par défaut ne rien faire — retourner state implicitement
    });
});
```

---

## 3. Immer sous le capot

### 3.1 Comment Immer fonctionne

Immer utilise les **Proxy JavaScript** (ES6) pour intercepter toutes les opérations de lecture/écriture sur l'objet state. Pendant l'exécution du reducer, vous travaillez sur un *draft* (brouillon), et Immer produit à la fin un nouvel objet immuable uniquement pour les parties modifiées.

```
┌────────────────────────────────────────────────────────────────┐
│                      IMMER : DRAFT SYSTEM                      │
│                                                                │
│  État original (frozen)      Draft (Proxy JS)                  │
│  ┌──────────────────────┐    ┌──────────────────────┐          │
│  │ state.users = [      │    │ draft.users = [      │          │
│  │   { id: 1,           │───▶│   { id: 1,           │          │
│  │     name: "Alice" }  │    │     name: "Alice" }  │          │
│  │ ]                    │    │ ]                    │          │
│  └──────────────────────┘    └──────────────────────┘          │
│                                       │                        │
│                                draft.users[0].name = "Bob"     │
│                                       │                        │
│                                       ▼                        │
│                              Immer détecte la mutation         │
│                              Crée un NOUVEAU state :           │
│                                                                │
│  Nouvel état (frozen)                                          │
│  ┌──────────────────────┐                                      │
│  │ state.users = [      │  ← Nouveau tableau                   │
│  │   { id: 1,           │  ← Nouvel objet                      │
│  │     name: "Bob" }    │                                      │
│  │ ]                    │                                      │
│  └──────────────────────┘                                      │
│                                                                │
│  Les parties NON modifiées sont PARTAGÉES (même référence)     │
└────────────────────────────────────────────────────────────────┘
```

### 3.2 Les règles à respecter avec Immer

```typescript
const badSlice = createSlice({
  name: 'example',
  initialState: { items: [], nested: { value: 0 } },
  reducers: {
    // ✅ CORRECT — mutation du draft
    goodMutation: (state, action) => {
      state.items.push(action.payload); // OK
      state.nested.value += 1;         // OK
    },

    // ✅ CORRECT — retourner un nouveau state (sans muter le draft)
    goodReturn: (state, action) => {
      return {
        ...state,
        items: [...state.items, action.payload]
      };
    },

    // ❌ INTERDIT — muter ET retourner
    badMutationAndReturn: (state, action) => {
      state.items.push(action.payload);
      return state; // ERREUR : ne jamais retourner le draft modifié
    },

    // ❌ INTERDIT — remplacer le draft par une primitive
    badReassignment: (state, action) => {
      state = action.payload; // ERREUR : n'a aucun effet
      // Pour remplacer tout le state, retourner la valeur :
    },

    // ✅ CORRECT — remplacer tout le state en retournant
    goodReplacement: (_state, action) => {
      return action.payload; // Underscore pour indiquer que state n'est pas utilisé
    }
  }
});
```

> [!tip] Immer et les structures non-standards
> Immer supporte les `Map`, `Set` et les tableaux imbriqués. Cependant, les classes personnalisées (avec méthodes) nécessitent soit de les marquer comme "immer-ignored" avec `setAutoFreeze(false)`, soit de les sérialiser en plain objects avant de les stocker. En règle générale : **ne stockez que des plain objects et primitives dans Redux**.

---

## 4. RTK Query — Gestion des données serveur

### 4.1 Pourquoi RTK Query ?

RTK Query est la solution de data-fetching intégrée à RTK. Elle gère automatiquement : le cache, les états de chargement, la déduplication des requêtes, l'invalidation du cache, et la synchronisation optimiste.

```
┌─────────────────────────────────────────────────────────────────────┐
│                     RTK QUERY : ARCHITECTURE                        │
│                                                                     │
│   Composant A         Composant B         Composant C               │
│   useGetUsersQuery()  useGetUsersQuery()  useGetUserQuery(id:42)     │
│         │                   │                    │                  │
│         └─────────┬─────────┘                    │                  │
│                   ▼                              ▼                  │
│         ┌─────────────────┐          ┌─────────────────┐            │
│         │   Cache Entry   │          │   Cache Entry   │            │
│         │  'getUsers'     │          │  'getUser(42)'  │            │
│         │  status: fresh  │          │  status: fresh  │            │
│         └─────────────────┘          └─────────────────┘            │
│                   │                              │                  │
│                   ▼                              ▼                  │
│         ┌──────────────────────────────────────────────┐            │
│         │              Redux Store                     │            │
│         │   api.queries / api.mutations                │            │
│         └──────────────────────────────────────────────┘            │
│                              │                                      │
│                              ▼                                      │
│                    ┌──────────────────┐                             │
│                    │   API Server     │                             │
│                    └──────────────────┘                             │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 `createApi` — Configuration complète

```typescript
// features/api/apiSlice.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import type { RootState } from '../../store';

// Types
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

interface Product {
  id: number;
  name: string;
  price: number;
  stock: number;
  categoryId: number;
}

interface Order {
  id: number;
  userId: number;
  products: Array<{ productId: number; quantity: number }>;
  total: number;
  status: 'pending' | 'confirmed' | 'shipped' | 'delivered';
}

interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  pageSize: number;
}

export const api = createApi({
  // Clé dans le store Redux où les données de cache seront stockées
  reducerPath: 'api',

  // Configuration de base de la requête HTTP
  baseQuery: fetchBaseQuery({
    baseUrl: 'https://api.myapp.com/v1',

    // Injecter le token d'authentification depuis le store
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token;
      if (token) {
        headers.set('Authorization', `Bearer ${token}`);
      }
      headers.set('Content-Type', 'application/json');
      return headers;
    }
  }),

  // Tags pour la gestion du cache — on y revient juste après
  tagTypes: ['User', 'Product', 'Order'],

  // Durée de conservation du cache après qu'un composant se désabonne
  keepUnusedDataFor: 60, // secondes (défaut : 60)

  // Endpoints — définition de toutes les opérations API
  endpoints: (builder) => ({

    // ─── QUERIES (lectures) ───────────────────────────────────────
    getUsers: builder.query<User[], void>({
      query: () => '/users',
      // Ce endpoint "fournit" des tags — si ces tags sont invalidés,
      // ce cache sera effacé et la requête relancée
      providesTags: (result) =>
        result
          ? [
              // Un tag pour chaque user individuel
              ...result.map(({ id }) => ({ type: 'User' as const, id })),
              // Un tag pour la liste complète
              { type: 'User', id: 'LIST' }
            ]
          : [{ type: 'User', id: 'LIST' }]
    }),

    getUserById: builder.query<User, number>({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }]
    }),

    getProducts: builder.query<PaginatedResponse<Product>, { page?: number; pageSize?: number; search?: string }>({
      query: ({ page = 1, pageSize = 20, search = '' }) => ({
        url: '/products',
        params: { page, pageSize, search }
      }),
      providesTags: (result) =>
        result
          ? [
              ...result.data.map(({ id }) => ({ type: 'Product' as const, id })),
              { type: 'Product', id: 'LIST' }
            ]
          : [{ type: 'Product', id: 'LIST' }]
    }),

    getOrdersByUser: builder.query<Order[], number>({
      query: (userId) => `/users/${userId}/orders`,
      providesTags: (result, error, userId) =>
        result
          ? [
              ...result.map(({ id }) => ({ type: 'Order' as const, id })),
              { type: 'Order', id: `USER_${userId}` }
            ]
          : [{ type: 'Order', id: `USER_${userId}` }]
    }),

    // ─── MUTATIONS (écritures) ────────────────────────────────────
    createUser: builder.mutation<User, Partial<User>>({
      query: (newUser) => ({
        url: '/users',
        method: 'POST',
        body: newUser
      }),
      // Invalider le cache de la liste des users après création
      invalidatesTags: [{ type: 'User', id: 'LIST' }]
    }),

    updateUser: builder.mutation<User, Pick<User, 'id'> & Partial<User>>({
      query: ({ id, ...patch }) => ({
        url: `/users/${id}`,
        method: 'PATCH',
        body: patch
      }),
      // Invalider l'entrée spécifique et la liste
      invalidatesTags: (result, error, { id }) => [
        { type: 'User', id },
        { type: 'User', id: 'LIST' }
      ]
    }),

    deleteUser: builder.mutation<void, number>({
      query: (id) => ({
        url: `/users/${id}`,
        method: 'DELETE'
      }),
      invalidatesTags: (result, error, id) => [
        { type: 'User', id },
        { type: 'User', id: 'LIST' }
      ]
    }),

    createOrder: builder.mutation<Order, Omit<Order, 'id' | 'status'>>({
      query: (newOrder) => ({
        url: '/orders',
        method: 'POST',
        body: newOrder
      }),
      invalidatesTags: (result, error, arg) => [
        { type: 'Order', id: 'LIST' },
        { type: 'Order', id: `USER_${arg.userId}` },
        // Invalider aussi le stock des produits commandés
        ...arg.products.map(({ productId }) => ({ type: 'Product' as const, id: productId }))
      ]
    }),

    // Optimistic update — mise à jour de l'UI avant confirmation serveur
    updateOrderStatus: builder.mutation<Order, { id: number; status: Order['status'] }>({
      query: ({ id, status }) => ({
        url: `/orders/${id}/status`,
        method: 'PATCH',
        body: { status }
      }),

      // onQueryStarted — pour les mises à jour optimistes
      async onQueryStarted({ id, status }, { dispatch, queryFulfilled }) {
        // Mise à jour immédiate du cache (avant la réponse serveur)
        const patchResult = dispatch(
          api.util.updateQueryData('getOrdersByUser', 1, (draft) => {
            const order = draft.find(o => o.id === id);
            if (order) {
              order.status = status; // Immer est actif ici aussi
            }
          })
        );

        try {
          await queryFulfilled; // Attendre la confirmation du serveur
        } catch {
          patchResult.undo(); // Rollback si erreur
        }
      },

      invalidatesTags: (result, error, { id }) => [{ type: 'Order', id }]
    })
  })
});

// Export des hooks générés automatiquement
export const {
  useGetUsersQuery,
  useGetUserByIdQuery,
  useGetProductsQuery,
  useGetOrdersByUserQuery,
  useCreateUserMutation,
  useUpdateUserMutation,
  useDeleteUserMutation,
  useCreateOrderMutation,
  useUpdateOrderStatusMutation,
  // Lazy queries — déclenchées manuellement
  useLazyGetUsersQuery,
  useLazyGetProductsQuery
} = api;
```

### 4.3 Intégrer RTK Query dans le store

```typescript
// store/index.ts (mise à jour)
import { configureStore } from '@reduxjs/toolkit';
import { api } from '../features/api/apiSlice';
import authReducer from '../features/auth/authSlice';

export const store = configureStore({
  reducer: {
    [api.reducerPath]: api.reducer, // IMPORTANT : utiliser api.reducerPath comme clé
    auth: authReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware()
      .concat(api.middleware) // IMPORTANT : ajouter le middleware RTK Query
});
```

### 4.4 Utiliser RTK Query dans les composants

```typescript
// components/UserList.tsx
import React, { useState } from 'react';
import {
  useGetUsersQuery,
  useDeleteUserMutation,
  useCreateUserMutation
} from '../features/api/apiSlice';

const UserList: React.FC = () => {
  const [newUserName, setNewUserName] = useState('');

  // useGetUsersQuery retourne { data, isLoading, isError, error, isFetching, refetch }
  const {
    data: users,
    isLoading,
    isError,
    error,
    isFetching,  // true quand une requête de refresh est en cours
    refetch      // forcer un refresh manuel
  } = useGetUsersQuery(undefined, {
    // Options optionnelles :
    pollingInterval: 30000,     // rafraîchir toutes les 30 secondes
    refetchOnFocus: true,       // rafraîchir quand la fenêtre reprend le focus
    refetchOnReconnect: true,   // rafraîchir après reconnexion réseau
    skip: false                 // mettre à true pour désactiver la requête conditionnellement
  });

  // Les mutations retournent [triggerFn, { isLoading, isError, data }]
  const [deleteUser, { isLoading: isDeleting }] = useDeleteUserMutation();
  const [createUser, { isLoading: isCreating, error: createError }] = useCreateUserMutation();

  if (isLoading) return <div className="spinner">Chargement...</div>;
  if (isError) return <div className="error">Erreur : {JSON.stringify(error)}</div>;

  const handleDelete = async (id: number) => {
    try {
      await deleteUser(id).unwrap(); // .unwrap() lève une exception si erreur
      console.log('Utilisateur supprimé');
    } catch (err) {
      console.error('Erreur lors de la suppression:', err);
    }
  };

  const handleCreate = async () => {
    if (!newUserName.trim()) return;
    try {
      const newUser = await createUser({
        name: newUserName,
        email: `${newUserName.toLowerCase()}@example.com`,
        role: 'user'
      }).unwrap();
      console.log('Créé:', newUser);
      setNewUserName('');
    } catch (err) {
      console.error('Erreur création:', err);
    }
  };

  return (
    <div>
      {/* Indicateur de refresh en arrière-plan */}
      {isFetching && <span className="badge">Mise à jour...</span>}

      <div className="add-user">
        <input
          value={newUserName}
          onChange={e => setNewUserName(e.target.value)}
          placeholder="Nom de l'utilisateur"
        />
        <button onClick={handleCreate} disabled={isCreating}>
          {isCreating ? 'Création...' : 'Ajouter'}
        </button>
      </div>

      <ul>
        {users?.map(user => (
          <li key={user.id}>
            <span>{user.name} ({user.role})</span>
            <button
              onClick={() => handleDelete(user.id)}
              disabled={isDeleting}
            >
              Supprimer
            </button>
          </li>
        ))}
      </ul>

      <button onClick={refetch}>Rafraîchir</button>
    </div>
  );
};

export default UserList;
```

---

## 5. Entity Adapters — Normalisation des collections

### 5.1 Le problème de la normalisation

Stocker des tableaux d'objets dans Redux est inefficace pour les lookups et les mises à jour.

```typescript
// ❌ Approche naïve — tableau brut
// Rechercher un item : O(n) — scan linéaire
const user = state.users.find(u => u.id === 42);

// ✅ Approche normalisée — structure byId/allIds
// Rechercher un item : O(1) — accès direct
const user = state.users.entities[42];
```

### 5.2 `createEntityAdapter` — L'outil de normalisation

```typescript
// features/products/productsSlice.ts
import {
  createSlice,
  createEntityAdapter,
  PayloadAction,
  EntityState
} from '@reduxjs/toolkit';

interface Product {
  id: number;
  name: string;
  price: number;
  stock: number;
  categoryId: number;
  featured: boolean;
}

// createEntityAdapter génère la structure normalisée et les CRUD operations
const productsAdapter = createEntityAdapter<Product>({
  // Optionnel : spécifier la clé (défaut = 'id')
  selectId: (product) => product.id,

  // Optionnel : trier les entités dans allIds
  sortComparer: (a, b) => a.name.localeCompare(b.name)
});

// État initial normalisé : { ids: [], entities: {} }
const initialState = productsAdapter.getInitialState({
  // On peut étendre l'état normalisé avec des champs supplémentaires
  loading: false,
  error: null as string | null,
  selectedId: null as number | null,
  totalCount: 0
});

type ProductsState = typeof initialState;

const productsSlice = createSlice({
  name: 'products',
  initialState,
  reducers: {
    // L'adapter fournit des méthodes CRUD prêtes à l'emploi
    productAdded: productsAdapter.addOne,      // Ajouter une entité
    productsAdded: productsAdapter.addMany,    // Ajouter plusieurs entités
    productUpdated: productsAdapter.updateOne, // Mettre à jour partiellement
    productUpserted: productsAdapter.upsertOne, // Add ou update
    productRemoved: productsAdapter.removeOne,  // Supprimer par ID
    productsRemoved: productsAdapter.removeMany, // Supprimer plusieurs
    allProductsRemoved: productsAdapter.removeAll, // Tout vider
    productsReceived: (state, action: PayloadAction<Product[]>) => {
      // setAll remplace toutes les entités
      productsAdapter.setAll(state, action.payload);
      state.totalCount = action.payload.length;
    },

    // Reducers custom qui utilisent les capacités d'Immer
    toggleFeatured: (state, action: PayloadAction<number>) => {
      const product = state.entities[action.payload];
      if (product) {
        product.featured = !product.featured;
      }
    },

    adjustStock: (
      state,
      action: PayloadAction<{ id: number; delta: number }>
    ) => {
      const product = state.entities[action.payload.id];
      if (product) {
        product.stock = Math.max(0, product.stock + action.payload.delta);
      }
    },

    setSelectedProduct: (state, action: PayloadAction<number | null>) => {
      state.selectedId = action.payload;
    }
  },
  extraReducers: (builder) => {
    // Ces cases seront remplis avec les thunks async
  }
});

export const {
  productAdded,
  productsAdded,
  productUpdated,
  productUpserted,
  productRemoved,
  productsReceived,
  toggleFeatured,
  adjustStock,
  setSelectedProduct
} = productsSlice.actions;

export default productsSlice.reducer;

// ─── SÉLECTEURS GÉNÉRÉS PAR L'ADAPTER ───────────────────────────────────────

// L'adapter génère automatiquement des sélecteurs Reselect
export const {
  selectAll: selectAllProducts,       // Retourne un tableau trié
  selectById: selectProductById,      // Retourne un Product | undefined
  selectIds: selectProductIds,        // Retourne un tableau d'IDs
  selectEntities: selectProductEntities, // Retourne le dictionnaire { [id]: Product }
  selectTotal: selectProductsCount    // Retourne le nombre d'entités
} = productsAdapter.getSelectors(
  // Sélecteur racine pour localiser la slice dans le state global
  (state: { products: ProductsState }) => state.products
);

// Sélecteurs custom bâtis sur les sélecteurs de l'adapter
export const selectSelectedProduct = (state: { products: ProductsState }) =>
  state.products.selectedId
    ? state.products.entities[state.products.selectedId]
    : null;

export const selectFeaturedProducts = (state: { products: ProductsState }) =>
  selectAllProducts(state).filter(p => p.featured);

export const selectProductsByCategory = (categoryId: number) =>
  (state: { products: ProductsState }) =>
    selectAllProducts(state).filter(p => p.categoryId === categoryId);
```

---

## 6. Architecture Middleware Redux

### 6.1 Le pattern middleware

Le middleware Redux est une fonction de composition qui intercepte les actions avant qu'elles atteignent le reducer.

```
┌────────────────────────────────────────────────────────────────────┐
│               CHAÎNE DE MIDDLEWARE REDUX                           │
│                                                                    │
│  dispatch(action)                                                  │
│       │                                                            │
│       ▼                                                            │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐           │
│  │  Middleware 1│──▶│  Middleware 2│──▶│  Middleware 3│           │
│  │  (Logger)    │   │  (Thunk)     │   │  (Saga)      │           │
│  └──────────────┘   └──────────────┘   └──────────────┘           │
│                                               │                   │
│                                               ▼                   │
│                                        ┌─────────────┐            │
│                                        │   Reducer   │            │
│                                        │   (Store)   │            │
│                                        └─────────────┘            │
└────────────────────────────────────────────────────────────────────┘
```

### 6.2 Écrire un middleware custom

La signature d'un middleware Redux est une fonction curryfiée à trois niveaux :

```typescript
// Signature complète d'un middleware Redux
const myMiddleware = (store: MiddlewareAPI) => (next: Dispatch) => (action: Action) => {
  // Avant que l'action atteigne le reducer
  console.log('Action dispatched:', action.type);
  console.log('State before:', store.getState());

  // Passer l'action au middleware suivant (ou au reducer si dernier)
  const result = next(action);

  // Après que l'action a traversé tous les middleware et le reducer
  console.log('State after:', store.getState());

  return result; // Retourner le résultat du dispatch
};

// Middleware de logging complet pour le développement
import { Middleware, isRejectedWithValue } from '@reduxjs/toolkit';

export const loggerMiddleware: Middleware = (store) => (next) => (action) => {
  const prevState = store.getState();
  const startTime = performance.now();

  const result = next(action);

  const nextState = store.getState();
  const duration = performance.now() - startTime;

  if (process.env.NODE_ENV === 'development') {
    console.group(`%c Action: ${(action as any).type}`, 'color: #4CAF50; font-weight: bold');
    console.log('%c Prev State', 'color: #9E9E9E', prevState);
    console.log('%c Action', 'color: #03A9F4', action);
    console.log('%c Next State', 'color: #4CAF50', nextState);
    console.log(`%c Duration: ${duration.toFixed(2)}ms`, 'color: #FF9800');
    console.groupEnd();
  }

  return result;
};

// Middleware de gestion d'erreur globale pour RTK Query
export const rtkQueryErrorLogger: Middleware = (store) => (next) => (action) => {
  // isRejectedWithValue détecte les erreurs RTK Query
  if (isRejectedWithValue(action)) {
    const errorMessage = (action.payload as any)?.data?.message || 'Une erreur est survenue';
    console.error('Erreur API:', errorMessage);

    // Dispatch une notification
    store.dispatch({ type: 'notifications/addError', payload: errorMessage });
  }
  return next(action);
};

// Middleware d'analytics
export const analyticsMiddleware: Middleware = (store) => (next) => (action) => {
  const result = next(action);

  // Envoyer certaines actions à un service d'analytics
  const trackedActions = ['cart/addItem', 'order/create', 'user/login'];
  if (trackedActions.includes((action as any).type)) {
    fetch('/analytics', {
      method: 'POST',
      body: JSON.stringify({
        event: (action as any).type,
        userId: store.getState().auth?.user?.id,
        timestamp: Date.now()
      })
    }).catch(() => {}); // Fire and forget
  }

  return result;
};
```

---

## 7. Redux Thunk — Actions asynchrones

### 7.1 Pourquoi Thunk ?

Un reducer Redux est une fonction **pure** et **synchrone** — il ne peut pas faire d'appels API. Thunk résout ce problème en permettant de dispatcher des fonctions au lieu d'objets.

```typescript
// Un thunk simple — fonction qui retourne une fonction
const fetchUserThunk = (userId: number) => async (dispatch: AppDispatch, getState: () => RootState) => {
  // dispatch : permet de dispatcher d'autres actions
  // getState : accès en lecture au state actuel

  dispatch({ type: 'users/setLoading', payload: true });

  try {
    const response = await fetch(`/api/users/${userId}`);
    const user = await response.json();
    dispatch({ type: 'users/userReceived', payload: user });
  } catch (error) {
    dispatch({ type: 'users/setError', payload: error.message });
  } finally {
    dispatch({ type: 'users/setLoading', payload: false });
  }
};

// Utilisation dans un composant
dispatch(fetchUserThunk(42));
```

### 7.2 `createAsyncThunk` — La version RTK

`createAsyncThunk` automatise la gestion des états pending/fulfilled/rejected.

```typescript
// features/users/usersSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import type { RootState } from '../../store';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UsersState {
  users: User[];
  selectedUser: User | null;
  status: 'idle' | 'loading' | 'succeeded' | 'failed';
  error: string | null;
  currentRequestId: string | null; // Pour annuler les requêtes obsolètes
}

// ─── THUNKS ASYNC ────────────────────────────────────────────────────────────

// Signature : createAsyncThunk<ReturnType, ArgType, ThunkAPIConfig>
export const fetchUsers = createAsyncThunk<User[], void>(
  'users/fetchAll',  // Préfixe des actions générées
  async (_, { rejectWithValue, signal }) => {
    // signal = AbortSignal pour annuler la requête (ex: composant démonté)
    const response = await fetch('/api/users', { signal });

    if (!response.ok) {
      // rejectWithValue permet de passer des données structurées au cas 'rejected'
      return rejectWithValue({
        status: response.status,
        message: await response.text()
      });
    }

    return response.json() as Promise<User[]>;
  }
);

export const fetchUserById = createAsyncThunk<User, number>(
  'users/fetchById',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
      return response.json();
    } catch (error) {
      return rejectWithValue((error as Error).message);
    }
  }
);

export const updateUser = createAsyncThunk<User, { id: number } & Partial<User>>(
  'users/update',
  async ({ id, ...updates }, { rejectWithValue, getState }) => {
    // getState permet d'accéder au state actuel pour des logiques conditionnelles
    const state = getState() as RootState;
    const currentUser = state.users.users.find(u => u.id === id);

    if (!currentUser) {
      return rejectWithValue('Utilisateur introuvable');
    }

    const response = await fetch(`/api/users/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updates)
    });

    if (!response.ok) {
      return rejectWithValue('Mise à jour échouée');
    }

    return response.json();
  }
);

// Thunk avec dispatch d'autres thunks (orchestration)
export const registerAndLogin = createAsyncThunk<
  { user: User; token: string },
  { name: string; email: string; password: string }
>(
  'auth/registerAndLogin',
  async (credentials, { dispatch, rejectWithValue }) => {
    try {
      // Étape 1 : Inscription
      const registerResponse = await fetch('/api/auth/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      });

      if (!registerResponse.ok) {
        const error = await registerResponse.json();
        return rejectWithValue(error.message);
      }

      // Étape 2 : Connexion automatique après inscription
      const loginResponse = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          email: credentials.email,
          password: credentials.password
        })
      });

      return loginResponse.json();
    } catch (error) {
      return rejectWithValue('Erreur réseau');
    }
  }
);

// ─── SLICE ────────────────────────────────────────────────────────────────────

const initialState: UsersState = {
  users: [],
  selectedUser: null,
  status: 'idle',
  error: null,
  currentRequestId: null
};

const usersSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    selectUser: (state, action: PayloadAction<number | null>) => {
      state.selectedUser = action.payload
        ? (state.users.find(u => u.id === action.payload) ?? null)
        : null;
    },
    clearError: (state) => {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      // ─── fetchUsers ───────────────────────────────────────────────
      .addCase(fetchUsers.pending, (state, action) => {
        state.status = 'loading';
        state.error = null;
        // Stocker requestId pour éviter les race conditions
        state.currentRequestId = action.meta.requestId;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        // Vérifier que c'est la requête la plus récente
        if (state.currentRequestId === action.meta.requestId) {
          state.status = 'succeeded';
          state.users = action.payload;
          state.currentRequestId = null;
        }
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        if (state.currentRequestId === action.meta.requestId) {
          state.status = 'failed';
          // action.payload si rejectWithValue, action.error.message sinon
          state.error = action.payload as string ?? action.error.message ?? 'Erreur inconnue';
          state.currentRequestId = null;
        }
      })

      // ─── fetchUserById ────────────────────────────────────────────
      .addCase(fetchUserById.fulfilled, (state, action) => {
        state.selectedUser = action.payload;
        // Ajouter ou mettre à jour dans la liste
        const index = state.users.findIndex(u => u.id === action.payload.id);
        if (index !== -1) {
          state.users[index] = action.payload;
        } else {
          state.users.push(action.payload);
        }
      })

      // ─── updateUser ───────────────────────────────────────────────
      .addCase(updateUser.fulfilled, (state, action) => {
        const index = state.users.findIndex(u => u.id === action.payload.id);
        if (index !== -1) {
          state.users[index] = action.payload;
        }
        if (state.selectedUser?.id === action.payload.id) {
          state.selectedUser = action.payload;
        }
      })

      // Pattern : gérer toutes les actions rejected en un seul matcher
      .addMatcher(
        (action) => action.type.endsWith('/rejected') && action.meta?.rejectedWithValue,
        (state, action: any) => {
          console.error('Action rejetée:', action.type, action.payload);
        }
      );
  }
});

export const { selectUser, clearError } = usersSlice.actions;
export default usersSlice.reducer;

// Sélecteurs
export const selectUsers = (state: RootState) => state.users.users;
export const selectUsersStatus = (state: RootState) => state.users.status;
export const selectSelectedUser = (state: RootState) => state.users.selectedUser;
export const selectUsersError = (state: RootState) => state.users.error;
```

### 7.3 Gestion de l'annulation avec `createAsyncThunk`

```typescript
// Dans un composant React
import { useEffect, useRef } from 'react';
import { useAppDispatch } from '../store';
import { fetchUsers } from '../features/users/usersSlice';

const UsersPage: React.FC = () => {
  const dispatch = useAppDispatch();
  const fetchPromiseRef = useRef<ReturnType<typeof dispatch> | null>(null);

  useEffect(() => {
    // Dispatcher le thunk retourne un objet avec une méthode abort()
    fetchPromiseRef.current = dispatch(fetchUsers());

    return () => {
      // Annuler la requête si le composant se démonte avant la fin
      fetchPromiseRef.current?.abort('Composant démonté');
    };
  }, [dispatch]);

  // ...
};
```

---

## 8. RTK Query vs createAsyncThunk

| Critère | RTK Query | createAsyncThunk |
|---|---|---|
| **Cas d'usage principal** | Données serveur (CRUD API) | Logique async custom |
| **Cache automatique** | ✅ Intégré | ❌ Manuel |
| **Déduplication des requêtes** | ✅ Automatique | ❌ À implémenter |
| **Invalidation de cache** | ✅ Tags système | ❌ Manuel |
| **Polling** | ✅ Option native | ❌ setInterval manuel |
| **Optimistic updates** | ✅ `onQueryStarted` | ⚠️ Possible mais verbeux |
| **Complexité de setup** | Moyenne (createApi) | Faible |
| **Flexibilité** | Limitée (API REST) | Totale |
| **Tests** | `msw` (mock service worker) | `redux-mock-store` |
| **Quand utiliser** | Requêtes serveur standard | Auth, side effects, orchestration |

> [!tip] Règle pratique
> - **RTK Query** pour tout ce qui touche à la synchronisation données-serveur (liste d'items, CRUD)
> - **createAsyncThunk** pour la logique métier async qui ne rentre pas dans un pattern REST : authentification, upload avec progression, orchestration multi-étapes, WebSockets

---

## 9. Redux Saga — Gestion avancée des effets de bord

### 9.1 Les générateurs JavaScript

Redux Saga exploite les générateurs ES6 pour décrire des flux asynchrones de manière synchrone et testable.

```javascript
// Rappel : les générateurs JS
function* compterJusquaTrois() {
  yield 1; // Suspendre et retourner 1
  yield 2; // Suspendre et retourner 2
  yield 3; // Suspendre et retourner 3
  return 'terminé';
}

const gen = compterJusquaTrois();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: 'terminé', done: true }

// Ce qui rend les sagas testables : yield retourne des "effets descriptifs"
// (de simples objets JS) — pas des promesses réelles
```

### 9.2 Installation et configuration

```bash
npm install redux-saga
```

```typescript
// store/index.ts avec Saga
import { configureStore } from '@reduxjs/toolkit';
import createSagaMiddleware from 'redux-saga';
import { rootSaga } from './sagas';

// Créer le middleware Saga
const sagaMiddleware = createSagaMiddleware({
  // Gestion des erreurs non capturées dans les sagas
  onError: (error, errorInfo) => {
    console.error('Saga error:', error, errorInfo);
  }
});

export const store = configureStore({
  reducer: { /* ... */ },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware()
      .concat(sagaMiddleware)
});

// Lancer la saga racine APRÈS la création du store
sagaMiddleware.run(rootSaga);
```

### 9.3 Les effets Saga — Le vocabulaire fondamental

```typescript
import {
  call,        // Appel asynchrone (API, fonction)
  put,         // Dispatcher une action Redux
  take,        // Attendre une action spécifique
  takeEvery,   // Écouter chaque occurrence d'une action
  takeLatest,  // N'exécuter que la plus récente (annule les précédentes)
  takeLeading, // Ignorer les nouvelles jusqu'à la fin de l'actuelle
  select,      // Lire le state Redux
  all,         // Exécution parallèle
  race,        // Course : le premier à finir gagne
  fork,        // Fork non-bloquant
  spawn,       // Fork détaché (ne sera pas annulé par le parent)
  cancel,      // Annuler une saga
  cancelled,   // Vérifier si la saga a été annulée
  delay,       // Pause (ms)
  throttle,    // Limiter la fréquence d'exécution
  debounce,    // Attendre silence avant d'exécuter
  retry,       // Réessayer N fois en cas d'échec
  channel      // Communication inter-sagas
} from 'redux-saga/effects';
```

### 9.4 Saga complète — Exemple e-commerce

```typescript
// sagas/productsSaga.ts
import {
  call, put, takeEvery, takeLatest, select, all, race, take, delay, retry
} from 'redux-saga/effects';
import { SagaIterator } from 'redux-saga';
import {
  fetchProductsRequest,
  fetchProductsSuccess,
  fetchProductsFailure,
  searchProductsRequest,
  addToCartRequest,
  addToCartSuccess,
  stockUpdated,
  notificationShown
} from '../actions';

// ─── API Layer ────────────────────────────────────────────────────────────────

async function fetchProductsAPI(page: number) {
  const response = await fetch(`/api/products?page=${page}`);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

async function addToCartAPI(productId: number, quantity: number) {
  const response = await fetch('/api/cart/items', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ productId, quantity })
  });
  if (!response.ok) throw new Error('Ajout au panier échoué');
  return response.json();
}

// ─── SAGA WORKERS (les fonctions qui font le travail) ──────────────────────────

// Worker saga pour fetchProducts — avec retry automatique
function* fetchProductsSaga(action: ReturnType<typeof fetchProductsRequest>): SagaIterator {
  try {
    // retry(3, 1000, fn, ...args) — réessaye 3 fois avec 1s entre chaque
    const products = yield retry(3, 1000, fetchProductsAPI, action.payload.page);
    yield put(fetchProductsSuccess(products));
  } catch (error) {
    yield put(fetchProductsFailure((error as Error).message));
  }
}

// Worker avec debounce natif pour la recherche
function* searchSaga(action: ReturnType<typeof searchProductsRequest>): SagaIterator {
  // Attendre 300ms de silence avant d'exécuter (debounce)
  yield delay(300);

  try {
    const results = yield call(
      fetch,
      `/api/products/search?q=${action.payload.query}`,
      { method: 'GET' }
    );
    const data = yield call([results, 'json']);
    yield put({ type: 'products/searchSuccess', payload: data });
  } catch (error) {
    yield put({ type: 'products/searchFailure', payload: (error as Error).message });
  }
}

// Saga complexe : ajout au panier avec vérification de stock et timeout
function* addToCartSaga(action: ReturnType<typeof addToCartRequest>): SagaIterator {
  const { productId, quantity } = action.payload;

  // Lire le state actuel
  const stock = yield select(
    (state: any) => state.products.entities[productId]?.stock ?? 0
  );

  if (stock < quantity) {
    yield put(notificationShown({
      type: 'error',
      message: `Stock insuffisant. Disponible : ${stock}`
    }));
    return;
  }

  try {
    // race : si l'API met plus de 5 secondes, timeout
    const { response, timeout } = yield race({
      response: call(addToCartAPI, productId, quantity),
      timeout: delay(5000)
    });

    if (timeout) {
      yield put(notificationShown({ type: 'error', message: 'Requête expirée' }));
      return;
    }

    yield put(addToCartSuccess(response));
    yield put(stockUpdated({ productId, delta: -quantity }));
    yield put(notificationShown({
      type: 'success',
      message: `${quantity} article(s) ajouté(s) au panier`
    }));

  } catch (error) {
    yield put(notificationShown({
      type: 'error',
      message: (error as Error).message
    }));
  }
}

// ─── SAGA D'AUTHENTIFICATION ────────────────────────────────────────────────

// Saga qui gère le cycle de vie complet login/logout
function* authSaga(): SagaIterator {
  while (true) {
    // Attendre l'action de login
    const { payload: credentials } = yield take('auth/loginRequest');

    // Lancer le processus de login dans un fork non-bloquant
    const loginTask = yield race({
      login: call(loginFlow, credentials),
      cancel: take('auth/loginCancel')
    });

    if (loginTask.login) {
      console.log('Login réussi, attendre le logout...');
      // Attendre le logout
      yield take('auth/logout');
      yield call(logoutFlow);
    }
    // Si cancel, reprendre la boucle et attendre le prochain login
  }
}

function* loginFlow(credentials: { email: string; password: string }): SagaIterator {
  try {
    const response = yield call(fetch, '/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    });
    const { token, user } = yield call([response, 'json']);

    // Stocker le token
    localStorage.setItem('token', token);

    yield put({ type: 'auth/loginSuccess', payload: { token, user } });
    return true;
  } catch (error) {
    yield put({ type: 'auth/loginFailure', payload: (error as Error).message });
    return false;
  }
}

function* logoutFlow(): SagaIterator {
  localStorage.removeItem('token');
  yield put({ type: 'auth/logoutSuccess' });
  yield put({ type: 'cart/clear' });
  yield put({ type: 'notifications/clearAll' });
}

// ─── WATCHER SAGAS (les fonctions qui écoutent les actions) ────────────────────

function* watchFetchProducts(): SagaIterator {
  // takeLatest annule la saga précédente si une nouvelle action arrive
  // Idéal pour les requêtes de pagination/filtrage
  yield takeLatest('products/fetchRequest', fetchProductsSaga);
}

function* watchSearch(): SagaIterator {
  // takeLatest + delay interne = debounce
  yield takeLatest('products/searchRequest', searchSaga);
}

function* watchAddToCart(): SagaIterator {
  // takeEvery exécute une saga pour CHAQUE action
  // (l'utilisateur peut ajouter plusieurs produits rapidement)
  yield takeEvery('cart/addRequest', addToCartSaga);
}

// ─── SAGA RACINE ──────────────────────────────────────────────────────────────

// sagas/index.ts
export function* rootSaga(): SagaIterator {
  // all() lance toutes les sagas en parallèle
  yield all([
    watchFetchProducts(),
    watchSearch(),
    watchAddToCart(),
    authSaga()
    // ... autres sagas
  ]);
}
```

---

## 10. Comparaison Thunk vs Saga

| Dimension | Redux Thunk | Redux Saga |
|---|---|---|
| **Courbe d'apprentissage** | Faible | Élevée (generators, effets) |
| **Lisibilité** | Bonne (async/await) | Excellente une fois maîtrisé |
| **Testabilité** | Moyenne (mock fetch) | Excellente (effets = objets JS) |
| **Annulation** | AbortController + manuel | `cancel()` / `race()` intégré |
| **Séquençage complexe** | Verbeux | Élégant (`take`, `race`, `fork`) |
| **Canaux (WebSocket, SSE)** | Difficile | `eventChannel` natif |
| **Débogage** | DevTools standard | Saga monitor dédié |
| **Taille du bundle** | ~800 octets | ~14 Ko |
| **Quand utiliser** | Projets petits/moyens | Projets complexes, workflows multi-étapes |

```typescript
// Test d'une saga — aucun mock API requis
import { runSaga } from 'redux-saga';
import { addToCartSaga } from './productsSaga';
import { call, put, select, race, delay } from 'redux-saga/effects';

describe('addToCartSaga', () => {
  it('devrait dispatcher addToCartSuccess quand le stock est suffisant', () => {
    const gen = addToCartSaga({ type: 'cart/addRequest', payload: { productId: 1, quantity: 2 } });

    // Tester chaque étape sans effectuer de vraie requête
    expect(gen.next().value).toEqual(
      select(expect.any(Function)) // Premier effect : select
    );

    // Simuler un stock de 10
    expect(gen.next(10).value).toEqual(
      race({
        response: call(addToCartAPI, 1, 2),
        timeout: delay(5000)
      })
    );

    // Simuler une réponse réussie
    expect(gen.next({ response: { id: 'cart-item-1' }, timeout: undefined }).value).toEqual(
      put(addToCartSuccess({ id: 'cart-item-1' }))
    );
  });
});
```

---

## 11. Architecture et structure de dossiers

### 11.1 Feature-based (Ducks Pattern) — Recommandé

```
src/
├── app/
│   ├── store.ts              # Configuration du store global
│   ├── hooks.ts              # useAppDispatch, useAppSelector
│   └── rootSaga.ts           # Racine des sagas (si Saga)
│
├── features/
│   ├── auth/
│   │   ├── authSlice.ts      # State + reducers + sélecteurs
│   │   ├── authThunks.ts     # createAsyncThunk (ou authSaga.ts)
│   │   ├── authSelectors.ts  # Sélecteurs Reselect complexes
│   │   ├── authTypes.ts      # Interfaces TypeScript
│   │   ├── AuthForm.tsx      # Composant(s) liés à la feature
│   │   ├── AuthPage.tsx
│   │   └── auth.test.ts      # Tests unitaires + intégration
│   │
│   ├── products/
│   │   ├── productsSlice.ts
│   │   ├── productsApi.ts    # RTK Query endpoints (ou dans apiSlice)
│   │   ├── productsSelectors.ts
│   │   ├── ProductList.tsx
│   │   ├── ProductCard.tsx
│   │   └── products.test.ts
│   │
│   ├── cart/
│   │   ├── cartSlice.ts
│   │   ├── cartSaga.ts       # Si Saga
│   │   ├── cartSelectors.ts
│   │   └── Cart.tsx
│   │
│   └── api/
│       └── apiSlice.ts       # RTK Query createApi centralisé
│
├── components/               # Composants partagés (non liés à une feature)
│   ├── Button/
│   ├── Modal/
│   └── Notification/
│
├── hooks/                    # Hooks React partagés
│   ├── useDebounce.ts
│   └── useLocalStorage.ts
│
└── types/                    # Types globaux partagés
    ├── api.ts
    └── common.ts
```

> [!info] Pourquoi feature-based ?
> L'organisation feature-based (aussi appelée "vertical slices") regroupe tout ce qui concerne une fonctionnalité au même endroit — reducer, composants, thunks, tests. Quand vous travaillez sur la feature "cart", vous n'avez qu'un seul dossier à ouvrir. Comparé à l'approche domain-based (`components/`, `reducers/`, `actions/` séparés), c'est bien plus maintenable à grande échelle.

---

## 12. Normalisation des données

### 12.1 Le problème de la dénormalisation

```typescript
// ❌ Données dénormalisées — problèmes de cohérence
const state = {
  orders: [
    {
      id: 1,
      user: { id: 42, name: "Alice", email: "alice@example.com" },
      products: [
        { id: 10, name: "Clavier", price: 89.99 },
        { id: 11, name: "Souris", price: 39.99 }
      ]
    },
    {
      id: 2,
      user: { id: 42, name: "Alice", email: "alice@example.com" }, // Dupliqué !
      products: [
        { id: 10, name: "Clavier", price: 89.99 } // Dupliqué encore !
      ]
    }
  ]
};

// Si Alice change d'email, il faut mettre à jour N endroits
```

### 12.2 La structure normalisée

```typescript
// ✅ Données normalisées — source unique de vérité
const normalizedState = {
  users: {
    ids: [42],
    entities: {
      42: { id: 42, name: "Alice", email: "alice@example.com" }
    }
  },
  products: {
    ids: [10, 11],
    entities: {
      10: { id: 10, name: "Clavier", price: 89.99 },
      11: { id: 11, name: "Souris", price: 39.99 }
    }
  },
  orders: {
    ids: [1, 2],
    entities: {
      1: { id: 1, userId: 42, productIds: [10, 11] },
      2: { id: 2, userId: 42, productIds: [10] }
    }
  }
};

// Si Alice change d'email — mise à jour en UN seul endroit
// state.users.entities[42].email = "newalice@example.com"
```

### 12.3 Normalisation avec `normalizr` + RTK

```typescript
// utils/normalize.ts
import { schema, normalize } from 'normalizr';

// Définir le schéma des entités
const userSchema = new schema.Entity('users');
const productSchema = new schema.Entity('products');
const orderSchema = new schema.Entity('orders', {
  user: userSchema,
  products: [productSchema]
});

// Normaliser des données dénormalisées venant de l'API
export function normalizeOrders(rawOrders: any[]) {
  const { entities, result } = normalize(rawOrders, [orderSchema]);
  return {
    users: entities.users ?? {},
    products: entities.products ?? {},
    orders: entities.orders ?? {},
    orderIds: result as number[]
  };
}

// Dans un thunk
export const fetchOrders = createAsyncThunk(
  'orders/fetchAll',
  async () => {
    const response = await fetch('/api/orders?include=user,products');
    const rawOrders = await response.json();

    // Normaliser avant de stocker
    return normalizeOrders(rawOrders);
  }
);
```

---

## 13. Sélecteurs avec Reselect

### 13.1 Le problème des sélecteurs inline

```typescript
// ❌ Sélecteur inline — recalculé à chaque render
const filteredProducts = useAppSelector(state => {
  // Cette fonction est exécutée à CHAQUE dispatch, même sans changement
  return state.products.items
    .filter(p => p.categoryId === selectedCategory)
    .sort((a, b) => a.price - b.price)
    .slice(0, 20);
});
```

### 13.2 `createSelector` — Mémoïsation des sélecteurs

```typescript
// features/products/productsSelectors.ts
import { createSelector } from '@reduxjs/toolkit'; // Reselect intégré
import type { RootState } from '../../store';
import { selectAllProducts } from './productsSlice';

// Sélecteurs primitifs (simples, toujours re-exécutés mais O(1))
const selectProductsState = (state: RootState) => state.products;
const selectSelectedCategory = (state: RootState) => state.filters.selectedCategory;
const selectPriceRange = (state: RootState) => state.filters.priceRange;
const selectSortOrder = (state: RootState) => state.filters.sortOrder;
const selectSearchQuery = (state: RootState) => state.filters.searchQuery;

// Sélecteur mémoïsé — ne recalcule QUE si ses entrées changent
export const selectFilteredProducts = createSelector(
  // Entrées (input selectors) — tableau de sélecteurs primitifs
  [
    selectAllProducts,           // Entrée 1 : tous les produits
    selectSelectedCategory,      // Entrée 2 : catégorie filtrée
    selectPriceRange,            // Entrée 3 : fourchette de prix
    selectSortOrder,             // Entrée 4 : ordre de tri
    selectSearchQuery            // Entrée 5 : recherche textuelle
  ],
  // Fonction de calcul — n'est appelée que si les entrées ont changé
  (products, category, priceRange, sortOrder, query) => {
    let filtered = products;

    if (category) {
      filtered = filtered.filter(p => p.categoryId === category);
    }

    if (priceRange) {
      filtered = filtered.filter(
        p => p.price >= priceRange.min && p.price <= priceRange.max
      );
    }

    if (query) {
      const lowerQuery = query.toLowerCase();
      filtered = filtered.filter(
        p => p.name.toLowerCase().includes(lowerQuery)
      );
    }

    return [...filtered].sort((a, b) =>
      sortOrder === 'price_asc' ? a.price - b.price :
      sortOrder === 'price_desc' ? b.price - a.price :
      a.name.localeCompare(b.name)
    );
  }
);

// Sélecteur paramétré — factory function
export const makeSelectProductsByCategory = () =>
  createSelector(
    [selectAllProducts, (_state: RootState, categoryId: number) => categoryId],
    (products, categoryId) => products.filter(p => p.categoryId === categoryId)
  );

// Utilisation dans un composant :
// const selectCategory1Products = makeSelectProductsByCategory();
// const products = useAppSelector(state => selectCategory1Products(state, 1));

// Sélecteur composite — bâti sur des sélecteurs mémoïsés
export const selectCartStats = createSelector(
  [
    (state: RootState) => state.cart.items,
    selectAllProducts
  ],
  (cartItems, products) => {
    return cartItems.reduce((stats, item) => {
      const product = products.find(p => p.id === item.productId);
      if (!product) return stats;

      return {
        totalItems: stats.totalItems + item.quantity,
        totalPrice: stats.totalPrice + (product.price * item.quantity),
        averagePrice: 0 // calculé après
      };
    }, { totalItems: 0, totalPrice: 0, averagePrice: 0 });
  }
);
```

---

## 14. Performance — Éviter les re-renders inutiles

### 14.1 `useSelector` et l'égalité de référence

```typescript
// ❌ Problème : crée un nouveau tableau à chaque appel
const productIds = useAppSelector(state => state.products.ids.map(id => Number(id)));
// → Nouveau tableau = nouvelle référence = re-render à chaque dispatch

// ✅ Solution 1 : utiliser les sélecteurs mémoïsés
const productIds = useAppSelector(selectProductIds); // Référence stable

// ✅ Solution 2 : shallowEqual pour les objets
import { shallowEqual } from 'react-redux';
const { name, email } = useAppSelector(
  state => ({ name: state.user.name, email: state.user.email }),
  shallowEqual // Compare propriété par propriété au lieu de ===
);

// ✅ Solution 3 : sélecteurs individuels
const name = useAppSelector(state => state.user.name);
const email = useAppSelector(state => state.user.email);
// React batche les re-renders dans ce cas
```

### 14.2 `React.memo` et optimisation des composants

```typescript
// components/ProductCard.tsx
import React, { memo, useCallback } from 'react';
import { useAppDispatch, useAppSelector } from '../app/hooks';
import { addToCartRequest } from '../features/cart/cartSlice';
import { selectProductById } from '../features/products/productsSlice';

interface Props {
  productId: number;
  // Ne pas passer l'objet Product complet — passer l'ID et lire depuis le store
  // Évite que ProductCard re-rende quand un autre produit change
}

// memo : re-render uniquement si les props changent
const ProductCard = memo(({ productId }: Props) => {
  const dispatch = useAppDispatch();

  // Sélecteur mémoïsé par ID — ne re-rende que si CE produit change
  const product = useAppSelector(state => selectProductById(state, productId));

  // useCallback pour éviter une nouvelle référence de fonction à chaque render
  const handleAddToCart = useCallback(() => {
    if (product) {
      dispatch(addToCartRequest({ productId: product.id, quantity: 1 }));
    }
  }, [dispatch, product?.id]);

  if (!product) return null;

  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <span>{product.price}€</span>
      <button onClick={handleAddToCart}>Ajouter au panier</button>
    </div>
  );
});

ProductCard.displayName = 'ProductCard'; // Pour le DevTools React

export default ProductCard;
```

### 14.3 Patterns anti-performance à éviter

```typescript
// ❌ NE PAS FAIRE — objet littéral dans useSelector
const data = useAppSelector(state => ({
  user: state.auth.user,
  products: state.products.items,
  cart: state.cart.items
}));
// Nouveau objet = re-render à CHAQUE dispatch, même sans changement

// ❌ NE PAS FAIRE — calcul coûteux inline
const expensiveData = useAppSelector(state =>
  state.products.items.reduce((acc, p) => {
    /* calcul O(n) */
  }, {})
);

// ❌ NE PAS FAIRE — filter sans mémoïsation
const filtered = useAppSelector(state =>
  state.products.items.filter(p => p.featured)
);

// ✅ FAIRE — sélecteurs mémoïsés avec createSelector
const { user, productsCount, cartTotal } = useAppSelector(
  createSelector(
    [
      (state: RootState) => state.auth.user,
      (state: RootState) => state.products.items.length,
      selectCartTotal
    ],
    (user, productsCount, cartTotal) => ({ user, productsCount, cartTotal })
  )
);
```

---

## 15. Redux DevTools

### 15.1 Installation et configuration

```bash
# Extension navigateur
# Chrome : Redux DevTools Extension
# Firefox : Redux DevTools

# Package optionnel pour la configuration avancée
npm install @redux-devtools/extension
```

```typescript
// Configuration avancée des DevTools
export const store = configureStore({
  reducer: { /* ... */ },
  devTools: process.env.NODE_ENV !== 'production'
    ? {
        name: 'Mon App',              // Nom affiché dans l'extension
        maxAge: 50,                   // Nombre max d'actions dans l'historique
        latency: 500,                 // Délai de report en ms
        actionsBlacklist: [          // Actions à ne pas logguer
          'auth/tokenRefreshed',
          '@@INIT'
        ],
        stateSanitizer: (state: any) => ({
          ...state,
          // Masquer les tokens dans les DevTools
          auth: { ...state.auth, token: '*** MASKED ***' }
        }),
        actionSanitizer: (action: any) => {
          if (action.type === 'auth/loginSuccess') {
            return { ...action, payload: { ...action.payload, token: '*** MASKED ***' } };
          }
          return action;
        }
      }
    : false
});
```

### 15.2 Time-travel debugging

```
┌──────────────────────────────────────────────────────────────────┐
│                    REDUX DEVTOOLS — FONCTIONS                    │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ Actions list                                               │  │
│  │ ▶ @@INIT                                         09:00:01  │  │
│  │ ▶ auth/loginSuccess                              09:00:05  │  │
│  │ ▶ products/fetchRequest                          09:00:06  │  │
│  │ ▶ api/executeMutation (createOrder)              09:00:10  │  │
│  │ ► cart/addItem                         ← Clic pour jump    │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Time Travel : Clic sur une action → état "sauté" à ce moment   │
│  Skip action : Décocher une action pour simuler sa suppression   │
│  Dispatch : Taper et dispatcher une action custom manuellement   │
│  Import/Export : Partager un état Redux en JSON avec l'équipe    │
│                                                                  │
│  Diff view : Visualiser exactement ce qui a changé dans l'état  │
│  ┌────────────────────────┐  ┌────────────────────────────────┐  │
│  │ State avant            │  │ State après                    │  │
│  │ cart.items: []         │  │ cart.items: [                  │  │
│  │                        │  │ + { id: 10, quantity: 1 }      │  │
│  │                        │  │ ]                              │  │
│  └────────────────────────┘  └────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 16. Tests — Stratégie complète

### 16.1 Tests des reducers — Fonctions pures

```typescript
// features/cart/cartSlice.test.ts
import cartReducer, {
  addItem,
  removeItem,
  updateQuantity,
  clearCart,
  applyDiscount
} from './cartSlice';
import type { CartState } from './cartSlice';

describe('Cart Reducer', () => {
  const initialState: CartState = {
    items: [],
    discount: null,
    total: 0
  };

  const mockItem = {
    productId: 1,
    name: 'Clavier Mécanique',
    price: 89.99,
    quantity: 1
  };

  // Les reducers sont des fonctions pures — tests simples et prévisibles
  describe('addItem', () => {
    it('devrait ajouter un item vide au panier', () => {
      const result = cartReducer(initialState, addItem(mockItem));
      expect(result.items).toHaveLength(1);
      expect(result.items[0]).toMatchObject(mockItem);
    });

    it('devrait incrémenter la quantité si l\'item existe déjà', () => {
      const stateWithItem = cartReducer(initialState, addItem(mockItem));
      const result = cartReducer(stateWithItem, addItem(mockItem));

      expect(result.items).toHaveLength(1);
      expect(result.items[0].quantity).toBe(2);
    });

    it('ne devrait pas muter l\'état initial (immutabilité)', () => {
      const frozenState = Object.freeze({ ...initialState, items: [] });
      expect(() => cartReducer(frozenState, addItem(mockItem))).not.toThrow();
    });
  });

  describe('removeItem', () => {
    it('devrait supprimer un item existant', () => {
      const stateWithItem = cartReducer(initialState, addItem(mockItem));
      const result = cartReducer(stateWithItem, removeItem(1));
      expect(result.items).toHaveLength(0);
    });

    it('devrait ignorer un id inexistant', () => {
      const stateWithItem = cartReducer(initialState, addItem(mockItem));
      const result = cartReducer(stateWithItem, removeItem(999));
      expect(result.items).toHaveLength(1);
    });
  });

  describe('applyDiscount', () => {
    it('devrait appliquer un pourcentage de réduction valide', () => {
      const state = { ...initialState, total: 100 };
      const result = cartReducer(state, applyDiscount({ code: 'PROMO10', percentage: 10 }));
      expect(result.discount).toEqual({ code: 'PROMO10', percentage: 10 });
    });

    it('devrait rejeter une réduction > 100%', () => {
      const state = { ...initialState, total: 100 };
      const result = cartReducer(state, applyDiscount({ code: 'BAD', percentage: 150 }));
      expect(result.discount).toBeNull();
    });
  });
});
```

### 16.2 Tests des Thunks avec `redux-mock-store`

```typescript
// features/users/usersThunks.test.ts
import { configureStore } from '@reduxjs/toolkit';
import fetchMock from 'jest-fetch-mock';
import usersReducer, { fetchUsers, fetchUserById } from './usersSlice';

// Utiliser le vrai store (pas un mock) — plus réaliste
function setupStore(preloadedState?: any) {
  return configureStore({
    reducer: { users: usersReducer },
    preloadedState
  });
}

describe('Thunks async - fetchUsers', () => {
  beforeEach(() => {
    fetchMock.enableMocks();
    fetchMock.resetMocks();
  });

  afterEach(() => {
    fetchMock.disableMocks();
  });

  it('devrait mettre users dans le state après fetchUsers.fulfilled', async () => {
    const mockUsers = [
      { id: 1, name: 'Alice', email: 'alice@test.com' },
      { id: 2, name: 'Bob', email: 'bob@test.com' }
    ];

    fetchMock.mockResponseOnce(JSON.stringify(mockUsers));

    const store = setupStore();
    await store.dispatch(fetchUsers());

    const state = store.getState();
    expect(state.users.status).toBe('succeeded');
    expect(state.users.users).toEqual(mockUsers);
    expect(state.users.error).toBeNull();
  });

  it('devrait avoir status "loading" pendant la requête', () => {
    // Ne pas résoudre la promesse immédiatement
    fetchMock.mockResponseOnce(() => new Promise(() => {}));

    const store = setupStore();
    store.dispatch(fetchUsers());

    const state = store.getState();
    expect(state.users.status).toBe('loading');
  });

  it('devrait mettre error dans le state si la requête échoue', async () => {
    fetchMock.mockRejectOnce(new Error('Network error'));

    const store = setupStore();
    await store.dispatch(fetchUsers());

    const state = store.getState();
    expect(state.users.status).toBe('failed');
    expect(state.users.error).toBeTruthy();
  });

  it('devrait permettre l\'annulation', async () => {
    fetchMock.mockResponseOnce(() => new Promise(() => {})); // Ne finit jamais

    const store = setupStore();
    const thunkPromise = store.dispatch(fetchUsers());
    thunkPromise.abort('Test d\'annulation');

    await thunkPromise;

    const state = store.getState();
    expect(state.users.status).not.toBe('succeeded');
  });
});
```

### 16.3 Tests RTK Query avec Mock Service Worker

```bash
npm install msw --save-dev
```

```typescript
// mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('https://api.myapp.com/v1/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'Alice', email: 'alice@test.com', role: 'user' },
      { id: 2, name: 'Bob', email: 'bob@test.com', role: 'admin' }
    ]);
  }),

  http.post('https://api.myapp.com/v1/users', async ({ request }) => {
    const body = await request.json() as any;
    return HttpResponse.json({ ...body, id: Math.random() }, { status: 201 });
  }),

  http.delete('https://api.myapp.com/v1/users/:id', ({ params }) => {
    return new HttpResponse(null, { status: 204 });
  })
];

// mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';
export const server = setupServer(...handlers);

// tests/setup.ts
import { server } from './mocks/server';
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// features/api/apiSlice.test.tsx
import { renderHook, waitFor } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { api, useGetUsersQuery, useDeleteUserMutation } from './apiSlice';
import { http, HttpResponse } from 'msw';
import { server } from '../../mocks/server';

function createWrapper() {
  const store = configureStore({
    reducer: { [api.reducerPath]: api.reducer },
    middleware: (getDefaultMiddleware) =>
      getDefaultMiddleware().concat(api.middleware)
  });

  const Wrapper = ({ children }: { children: React.ReactNode }) => (
    <Provider store={store}>{children}</Provider>
  );

  return { store, Wrapper };
}

describe('RTK Query - useGetUsersQuery', () => {
  it('devrait retourner les utilisateurs', async () => {
    const { Wrapper } = createWrapper();
    const { result } = renderHook(() => useGetUsersQuery(), { wrapper: Wrapper });

    expect(result.current.isLoading).toBe(true);

    await waitFor(() => expect(result.current.isLoading).toBe(false));

    expect(result.current.data).toHaveLength(2);
    expect(result.current.data?.[0].name).toBe('Alice');
  });

  it('devrait gérer les erreurs API', async () => {
    // Override du handler pour simuler une erreur
    server.use(
      http.get('https://api.myapp.com/v1/users', () => {
        return HttpResponse.json({ message: 'Server Error' }, { status: 500 });
      })
    );

    const { Wrapper } = createWrapper();
    const { result } = renderHook(() => useGetUsersQuery(), { wrapper: Wrapper });

    await waitFor(() => expect(result.current.isError).toBe(true));
    expect(result.current.error).toBeDefined();
  });

  it('devrait invalider le cache après une suppression', async () => {
    const { Wrapper, store } = createWrapper();

    // Pré-remplir le cache
    const { result } = renderHook(
      () => ({
        users: useGetUsersQuery(),
        deleteUser: useDeleteUserMutation()
      }),
      { wrapper: Wrapper }
    );

    await waitFor(() => expect(result.current.users.data).toHaveLength(2));

    // Déclencher la mutation
    result.current.deleteUser[0](1);

    // Vérifier que le cache est invalidé et la requête relancée
    await waitFor(() => {
      expect(result.current.users.isFetching).toBe(true);
    });
  });
});
```

### 16.4 Tests des Sagas

```typescript
// sagas/cartSaga.test.ts
import { testSaga, expectSaga } from 'redux-saga-test-plan';
import * as matchers from 'redux-saga-test-plan/matchers';
import { throwError } from 'redux-saga-test-plan/providers';
import { call, put, select, race, delay } from 'redux-saga/effects';
import { addToCartSaga, addToCartAPI } from './cartSaga';
import { addToCartRequest, addToCartSuccess, notificationShown } from '../actions';

describe('addToCartSaga', () => {
  const action = addToCartRequest({ productId: 1, quantity: 2 });
  const mockCartItem = { id: 'cart-123', productId: 1, quantity: 2 };

  // Test de la saga entière avec état intégré (expectSaga)
  it('devrait ajouter un item avec succès quand le stock est suffisant', () => {
    return expectSaga(addToCartSaga, action)
      .provide([
        // Mocker le select
        [matchers.select.selector(expect.any(Function)), 10], // stock = 10
        // Mocker l'API
        [matchers.call.fn(addToCartAPI), mockCartItem]
      ])
      .put(addToCartSuccess(mockCartItem))
      .put(notificationShown({ type: 'success', message: expect.any(String) }))
      .run();
  });

  it('devrait notifier une erreur quand le stock est insuffisant', () => {
    return expectSaga(addToCartSaga, action)
      .provide([
        [matchers.select.selector(expect.any(Function)), 1] // stock = 1 < quantity = 2
      ])
      .put(notificationShown({ type: 'error', message: expect.stringContaining('Stock') }))
      .not.put.actionType('cart/addSuccess')
      .run();
  });

  it('devrait gérer le timeout', () => {
    return expectSaga(addToCartSaga, action)
      .provide([
        [matchers.select.selector(expect.any(Function)), 10],
        // Simuler un délai de 10 secondes (> timeout de 5s)
        [matchers.call.fn(addToCartAPI), new Promise(() => {})]
      ])
      .put(notificationShown({ type: 'error', message: expect.stringContaining('expir') }))
      .run({ timeout: 6000 });
  });

  it('devrait gérer les erreurs réseau', () => {
    return expectSaga(addToCartSaga, action)
      .provide([
        [matchers.select.selector(expect.any(Function)), 10],
        [matchers.call.fn(addToCartAPI), throwError(new Error('Network error'))]
      ])
      .put(notificationShown({ type: 'error', message: 'Network error' }))
      .run();
  });
});
```

---

## 17. Tests d'intégration — `renderWithProviders`

### 17.1 Utilitaire de test

```typescript
// tests/utils/renderWithProviders.tsx
import React, { PropsWithChildren } from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore, PreloadedState } from '@reduxjs/toolkit';
import type { AppStore, RootState } from '../../app/store';
import { api } from '../../features/api/apiSlice';
import authReducer from '../../features/auth/authSlice';
import cartReducer from '../../features/cart/cartSlice';
import productsReducer from '../../features/products/productsSlice';

interface ExtendedRenderOptions extends Omit<RenderOptions, 'queries'> {
  preloadedState?: PreloadedState<RootState>;
  store?: AppStore;
}

export function renderWithProviders(
  ui: React.ReactElement,
  {
    preloadedState = {},
    store = configureStore({
      reducer: {
        [api.reducerPath]: api.reducer,
        auth: authReducer,
        cart: cartReducer,
        products: productsReducer
      },
      middleware: (getDefaultMiddleware) =>
        getDefaultMiddleware().concat(api.middleware),
      preloadedState
    }),
    ...renderOptions
  }: ExtendedRenderOptions = {}
) {
  function Wrapper({ children }: PropsWithChildren<{}>) {
    return <Provider store={store}>{children}</Provider>;
  }

  return {
    store,
    ...render(ui, { wrapper: Wrapper, ...renderOptions })
  };
}
```

### 17.2 Test d'intégration complet

```typescript
// features/cart/Cart.integration.test.tsx
import { screen, fireEvent, waitFor } from '@testing-library/react';
import { renderWithProviders } from '../../tests/utils/renderWithProviders';
import Cart from './Cart';

describe('Cart — Tests d\'intégration', () => {
  it('devrait afficher le panier vide initialement', () => {
    renderWithProviders(<Cart />);
    expect(screen.getByText(/panier vide/i)).toBeInTheDocument();
  });

  it('devrait afficher les items du panier depuis le state initial', () => {
    renderWithProviders(<Cart />, {
      preloadedState: {
        cart: {
          items: [
            { productId: 1, name: 'Clavier', price: 89.99, quantity: 2 }
          ],
          discount: null,
          total: 179.98
        }
      }
    });

    expect(screen.getByText('Clavier')).toBeInTheDocument();
    expect(screen.getByText(/179,98/)).toBeInTheDocument();
  });

  it('devrait permettre la suppression d\'un item', async () => {
    const { store } = renderWithProviders(<Cart />, {
      preloadedState: {
        cart: {
          items: [{ productId: 1, name: 'Clavier', price: 89.99, quantity: 1 }],
          discount: null,
          total: 89.99
        }
      }
    });

    const deleteButton = screen.getByRole('button', { name: /supprimer/i });
    fireEvent.click(deleteButton);

    await waitFor(() => {
      expect(screen.queryByText('Clavier')).not.toBeInTheDocument();
    });

    expect(store.getState().cart.items).toHaveLength(0);
  });

  it('devrait mettre à jour le total après modification de quantité', async () => {
    renderWithProviders(<Cart />, {
      preloadedState: {
        cart: {
          items: [{ productId: 1, name: 'Clavier', price: 89.99, quantity: 1 }],
          discount: null,
          total: 89.99
        }
      }
    });

    const quantityInput = screen.getByRole('spinbutton');
    fireEvent.change(quantityInput, { target: { value: '3' } });

    await waitFor(() => {
      expect(screen.getByText(/269,97/)).toBeInTheDocument();
    });
  });
});
```

---

## 18. Cas pratique complet — E-commerce avec RTK + RTK Query + Auth

### 18.1 Architecture du projet

```
src/
├── app/
│   ├── store.ts
│   └── hooks.ts
├── features/
│   ├── auth/
│   │   ├── authSlice.ts        (login, logout, token refresh)
│   │   └── AuthPage.tsx
│   ├── products/
│   │   ├── productsSlice.ts    (entity adapter + filtres locaux)
│   │   └── ProductsPage.tsx
│   ├── cart/
│   │   ├── cartSlice.ts        (items, discount, total calculé)
│   │   └── CartDrawer.tsx
│   └── api/
│       └── ecommerceApi.ts     (RTK Query : products, orders, users)
└── components/
    └── Notification.tsx
```

### 18.2 Auth Slice — Gestion complète de l'authentification

```typescript
// features/auth/authSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'customer';
}

interface AuthState {
  user: User | null;
  token: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  status: 'idle' | 'loading' | 'failed';
  error: string | null;
}

interface LoginCredentials {
  email: string;
  password: string;
}

interface AuthResponse {
  user: User;
  token: string;
  refreshToken: string;
}

// Helpers localStorage
const TOKEN_KEY = 'ecommerce_token';
const REFRESH_KEY = 'ecommerce_refresh';

// Thunk de login
export const login = createAsyncThunk<AuthResponse, LoginCredentials>(
  'auth/login',
  async (credentials, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      });

      if (!response.ok) {
        const error = await response.json();
        return rejectWithValue(error.message || 'Identifiants invalides');
      }

      const data: AuthResponse = await response.json();
      // Persister le token
      localStorage.setItem(TOKEN_KEY, data.token);
      localStorage.setItem(REFRESH_KEY, data.refreshToken);

      return data;
    } catch {
      return rejectWithValue('Erreur réseau — vérifiez votre connexion');
    }
  }
);

// Thunk de rafraîchissement de token
export const refreshAuthToken = createAsyncThunk<
  Pick<AuthResponse, 'token'>,
  void,
  { state: { auth: AuthState } }
>(
  'auth/refreshToken',
  async (_, { getState, rejectWithValue }) => {
    const { refreshToken } = getState().auth;
    if (!refreshToken) return rejectWithValue('Pas de refresh token');

    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ refreshToken })
    });

    if (!response.ok) return rejectWithValue('Session expirée');
    const data = await response.json();
    localStorage.setItem(TOKEN_KEY, data.token);
    return data;
  }
);

// Rehydrater depuis localStorage au démarrage
const storedToken = localStorage.getItem(TOKEN_KEY);
const storedRefresh = localStorage.getItem(REFRESH_KEY);

const initialState: AuthState = {
  user: null,
  token: storedToken,
  refreshToken: storedRefresh,
  isAuthenticated: !!storedToken,
  status: 'idle',
  error: null
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    logout: (state) => {
      state.user = null;
      state.token = null;
      state.refreshToken = null;
      state.isAuthenticated = false;
      localStorage.removeItem(TOKEN_KEY);
      localStorage.removeItem(REFRESH_KEY);
    },
    clearError: (state) => { state.error = null; }
  },
  extraReducers: (builder) => {
    builder
      .addCase(login.pending, (state) => {
        state.status = 'loading';
        state.error = null;
      })
      .addCase(login.fulfilled, (state, action) => {
        state.status = 'idle';
        state.user = action.payload.user;
        state.token = action.payload.token;
        state.refreshToken = action.payload.refreshToken;
        state.isAuthenticated = true;
      })
      .addCase(login.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload as string;
        state.isAuthenticated = false;
      })
      .addCase(refreshAuthToken.fulfilled, (state, action) => {
        state.token = action.payload.token;
      })
      .addCase(refreshAuthToken.rejected, (state) => {
        // Le refresh a échoué — déconnecter l'utilisateur
        state.user = null;
        state.token = null;
        state.refreshToken = null;
        state.isAuthenticated = false;
        localStorage.removeItem(TOKEN_KEY);
        localStorage.removeItem(REFRESH_KEY);
      });
  }
});

export const { logout, clearError } = authSlice.actions;
export default authSlice.reducer;

export const selectIsAuthenticated = (state: { auth: AuthState }) => state.auth.isAuthenticated;
export const selectCurrentUser = (state: { auth: AuthState }) => state.auth.user;
export const selectAuthStatus = (state: { auth: AuthState }) => state.auth.status;
export const selectAuthError = (state: { auth: AuthState }) => state.auth.error;
```

### 18.3 Cart Slice avec calculs dérivés

```typescript
// features/cart/cartSlice.ts
import { createSlice, createSelector, PayloadAction } from '@reduxjs/toolkit';
import type { RootState } from '../../app/store';

interface CartItem {
  productId: number;
  name: string;
  price: number;
  quantity: number;
  imageUrl: string;
}

interface CartState {
  items: CartItem[];
  promoCode: string | null;
  discountPercentage: number;
  isOpen: boolean;
}

const initialState: CartState = {
  items: [],
  promoCode: null,
  discountPercentage: 0,
  isOpen: false
};

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addItem: (state, action: PayloadAction<Omit<CartItem, 'quantity'> & { quantity?: number }>) => {
      const existing = state.items.find(i => i.productId === action.payload.productId);
      if (existing) {
        existing.quantity += action.payload.quantity ?? 1;
      } else {
        state.items.push({ ...action.payload, quantity: action.payload.quantity ?? 1 });
      }
    },

    removeItem: (state, action: PayloadAction<number>) => {
      state.items = state.items.filter(i => i.productId !== action.payload);
    },

    updateQuantity: (state, action: PayloadAction<{ productId: number; quantity: number }>) => {
      const item = state.items.find(i => i.productId === action.payload.productId);
      if (item) {
        if (action.payload.quantity <= 0) {
          state.items = state.items.filter(i => i.productId !== action.payload.productId);
        } else {
          item.quantity = action.payload.quantity;
        }
      }
    },

    applyPromoCode: (state, action: PayloadAction<{ code: string; discount: number }>) => {
      state.promoCode = action.payload.code;
      state.discountPercentage = action.payload.discount;
    },

    removePromoCode: (state) => {
      state.promoCode = null;
      state.discountPercentage = 0;
    },

    clearCart: (state) => {
      state.items = [];
      state.promoCode = null;
      state.discountPercentage = 0;
    },

    toggleCart: (state) => { state.isOpen = !state.isOpen; },
    openCart: (state) => { state.isOpen = true; },
    closeCart: (state) => { state.isOpen = false; }
  }
});

export const {
  addItem, removeItem, updateQuantity,
  applyPromoCode, removePromoCode, clearCart,
  toggleCart, openCart, closeCart
} = cartSlice.actions;

export default cartSlice.reducer;

// ─── SÉLECTEURS MÉMOÏSÉS ─────────────────────────────────────────────────────

const selectCartState = (state: RootState) => state.cart;

export const selectCartItems = createSelector(
  [selectCartState],
  cart => cart.items
);

export const selectCartIsOpen = createSelector(
  [selectCartState],
  cart => cart.isOpen
);

export const selectCartSubtotal = createSelector(
  [selectCartItems],
  items => items.reduce((sum, item) => sum + item.price * item.quantity, 0)
);

export const selectCartDiscount = createSelector(
  [selectCartState, selectCartSubtotal],
  (cart, subtotal) => ({
    code: cart.promoCode,
    percentage: cart.discountPercentage,
    amount: subtotal * (cart.discountPercentage / 100)
  })
);

export const selectCartTotal = createSelector(
  [selectCartSubtotal, selectCartDiscount],
  (subtotal, discount) => subtotal - discount.amount
);

export const selectCartItemCount = createSelector(
  [selectCartItems],
  items => items.reduce((count, item) => count + item.quantity, 0)
);

export const selectIsInCart = (productId: number) =>
  createSelector(
    [selectCartItems],
    items => items.some(i => i.productId === productId)
  );
```

### 18.4 Composant ProductsPage connecté

```typescript
// features/products/ProductsPage.tsx
import React, { useState, useMemo } from 'react';
import { useAppDispatch, useAppSelector } from '../../app/hooks';
import { useGetProductsQuery } from '../api/ecommerceApi';
import { addItem, toggleCart, selectIsInCart } from '../cart/cartSlice';
import { selectCurrentUser } from '../auth/authSlice';

// Composant individuel mémoïsé
const ProductCard = React.memo(({ product, onAddToCart, isInCart }: {
  product: any;
  onAddToCart: (product: any) => void;
  isInCart: boolean;
}) => (
  <div className={`product-card ${isInCart ? 'in-cart' : ''}`}>
    <img src={product.imageUrl} alt={product.name} />
    <div className="product-info">
      <h3>{product.name}</h3>
      <p className="price">{product.price.toFixed(2)} €</p>
      <span className={`stock ${product.stock < 5 ? 'low' : ''}`}>
        {product.stock < 5 ? `Plus que ${product.stock} !` : 'En stock'}
      </span>
    </div>
    <button
      className={isInCart ? 'btn-secondary' : 'btn-primary'}
      onClick={() => onAddToCart(product)}
      disabled={product.stock === 0}
    >
      {isInCart ? 'Dans le panier ✓' : 'Ajouter au panier'}
    </button>
  </div>
));

const ProductsPage: React.FC = () => {
  const dispatch = useAppDispatch();
  const currentUser = useAppSelector(selectCurrentUser);

  const [page, setPage] = useState(1);
  const [search, setSearch] = useState('');
  const [debouncedSearch, setDebouncedSearch] = useState('');

  // RTK Query avec paramètres
  const { data, isLoading, isFetching, isError } = useGetProductsQuery({
    page,
    pageSize: 12,
    search: debouncedSearch
  });

  // Débounce de la recherche
  React.useEffect(() => {
    const timer = setTimeout(() => setDebouncedSearch(search), 300);
    return () => clearTimeout(timer);
  }, [search]);

  const handleAddToCart = React.useCallback((product: any) => {
    dispatch(addItem({
      productId: product.id,
      name: product.name,
      price: product.price,
      imageUrl: product.imageUrl
    }));
    dispatch(openCart());
  }, [dispatch]);

  const cartSelector = useMemo(
    () => selectIsInCart(0), // Sera recréé par chaque ProductCard avec son propre ID
    []
  );

  if (isError) {
    return (
      <div className="error-page">
        <h2>Impossible de charger les produits</h2>
        <p>Vérifiez votre connexion et réessayez.</p>
      </div>
    );
  }

  return (
    <div className="products-page">
      <header className="page-header">
        <h1>Notre catalogue</h1>
        {currentUser && <span>Bonjour, {currentUser.name} !</span>}
        <input
          type="search"
          value={search}
          onChange={e => setSearch(e.target.value)}
          placeholder="Rechercher un produit..."
        />
      </header>

      {isLoading ? (
        <div className="products-grid skeleton">
          {Array.from({ length: 12 }).map((_, i) => (
            <div key={i} className="product-card-skeleton" />
          ))}
        </div>
      ) : (
        <>
          {/* Indicateur de refresh en arrière-plan */}
          {isFetching && <div className="fetching-banner">Mise à jour...</div>}

          <div className="products-grid">
            {data?.data.map(product => (
              <ProductCardWithCartStatus
                key={product.id}
                product={product}
                onAddToCart={handleAddToCart}
              />
            ))}
          </div>

          <div className="pagination">
            <button
              onClick={() => setPage(p => Math.max(1, p - 1))}
              disabled={page === 1 || isFetching}
            >
              Precedent
            </button>
            <span>Page {page} sur {Math.ceil((data?.total ?? 0) / 12)}</span>
            <button
              onClick={() => setPage(p => p + 1)}
              disabled={!data || page >= Math.ceil(data.total / 12) || isFetching}
            >
              Suivant
            </button>
          </div>
        </>
      )}
    </div>
  );
};

// Composant wrapper pour injecter l'état "in cart" individuellement
const ProductCardWithCartStatus = React.memo(({ product, onAddToCart }: {
  product: any;
  onAddToCart: (product: any) => void;
}) => {
  // Chaque carte lit SON état dans le cart — pas de re-render global
  const isInCart = useAppSelector(
    useMemo(() => selectIsInCart(product.id), [product.id])
  );

  return (
    <ProductCard
      product={product}
      onAddToCart={onAddToCart}
      isInCart={isInCart}
    />
  );
});

export default ProductsPage;
```

### 18.5 Store final assemblé

```typescript
// app/store.ts
import { configureStore } from '@reduxjs/toolkit';
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import { ecommerceApi } from '../features/api/ecommerceApi';
import authReducer from '../features/auth/authSlice';
import cartReducer from '../features/cart/cartSlice';
import productsReducer from '../features/products/productsSlice';
import { rtkQueryErrorLogger } from './middleware/errorLogger';

export const store = configureStore({
  reducer: {
    [ecommerceApi.reducerPath]: ecommerceApi.reducer,
    auth: authReducer,
    cart: cartReducer,
    products: productsReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['auth/loginSuccess'],
        ignoredPaths: ['auth.tokenExpiry']
      }
    })
      .concat(ecommerceApi.middleware)
      .concat(rtkQueryErrorLogger),
  devTools: {
    name: 'E-Commerce Store',
    maxAge: 100,
    stateSanitizer: (state: any) => ({
      ...state,
      auth: { ...state.auth, token: state.auth.token ? '***' : null }
    })
  }
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

---

## Exercices pratiques

> [!tip] Challenge 1 — Niveau intermédiaire
> **Système de notifications avec auto-dismiss**
> Créer un `notificationsSlice` avec :
> - Actions `addNotification(message, type, duration?)` et `removeNotification(id)`
> - Un thunk `showNotification` qui dispatch `addNotification` puis `removeNotification` après le délai
> - Un sélecteur `selectVisibleNotifications` avec Reselect
> - Un composant `NotificationStack` qui affiche les notifications avec animation

> [!tip] Challenge 2 — Niveau avancé
> **Panier persistant avec synchronisation cloud**
> Étendre le `cartSlice` pour :
> - Persister le panier dans `localStorage` via un middleware custom
> - Synchroniser avec l'API au login (merger le panier local avec le panier serveur)
> - Gérer les conflits (même produit en local ET serveur, quantités différentes)
> - Utiliser RTK Query pour les endpoints `/cart/sync` et `/cart/merge`

> [!tip] Challenge 3 — Niveau expert
> **Saga de paiement multi-étapes**
> Implémenter avec Redux Saga :
> - Saga `checkoutSaga` qui orchestre : validation panier -> vérification stock -> création commande -> paiement -> confirmation
> - Si une étape échoue, rollback des étapes précédentes
> - Annulation possible à tout moment avec `take('checkout/cancel')`
> - Retry automatique sur les erreurs réseau (max 3 tentatives)
> - Tests complets avec `expectSaga`

---

> [!warning] Erreurs courantes à éviter
> 1. **Stocker des données non-sérialisables** (dates JS, instances de classe, fonctions) dans Redux — utilisez des timestamps ISO 8601 strings
> 2. **Oublier `.concat(api.middleware)`** dans la config du store RTK Query — le cache ne fonctionnera pas
> 3. **Créer des sélecteurs inline** dans `useSelector` sans Reselect — cause des re-renders en cascade
> 4. **Muter l'état hors d'Immer** (dans `extraReducers` en retournant le state modifié directement)
> 5. **Passer des objets entiers en props** quand seul l'ID est nécessaire — empêche React.memo de fonctionner
> 6. **Oublier le `reducerPath`** lors de l'ajout de RTK Query au store — conflits de clés
