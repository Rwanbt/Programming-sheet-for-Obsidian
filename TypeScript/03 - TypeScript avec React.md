# TypeScript avec React

> [!tip] Analogie
> Imagine que tu construis une maison avec des ouvriers. Sans TypeScript, chaque ouvrier peut apporter n'importe quel outil — une scie, un marteau, ou par erreur une cuillere. Avec TypeScript, tu definis des contrats : "cette piece ne peut recevoir que des ouvriers avec un marteau". Le compilateur devient le chef de chantier qui verifie les acces avant meme que la construction commence. Resultat : zero surprise en production, les bugs sont detectes dans l'editeur.

---

[[TypeScript/01 - Introduction a TypeScript]] | [[TypeScript/02 - Types Avances]] | [[React/01 - Introduction a React]] | [[React/02 - Hooks et Gestion d'Etat]]

---

## Pourquoi typer ses composants React ?

React sans TypeScript fonctionne — des millions d'apps tournent en JavaScript pur. Mais le typage apporte trois gains concrets que tu ressentiras des la premiere semaine :

**1. Autocompletion intelligente**
Quand tu passes une prop a un composant, ton editeur connait exactement quels champs existent, quels types sont attendus, quelles valeurs sont optionnelles. Plus besoin de retourner lire la definition du composant.

**2. Erreurs au compile plutot qu'au runtime**
Sans types, un `props.user.name` sur une prop `user` qui peut etre `null` crashe en production devant un utilisateur reel. Avec TypeScript, le compilateur refuse de builder ce code.

**3. Refactoring sans peur**
Renommer une prop ? TypeScript te montre instantanement tous les endroits ou cette prop est utilisee et qui sont maintenant casses. En JavaScript, tu pries pour que tes tests couvrent tout.

```
Sans TypeScript :                    Avec TypeScript :
                                     
  <Button                              <Button
    onClick={handlerClick}   →           onClick={handleClick}   ✓
    label={42}               ×           label="Valider"         ✓
    isDisable={true}         ×           disabled={true}         ✓
  />                                   />
  
  ↓ Erreur a 23h en prod               ↓ Erreur dans l'editeur a 14h
```

> [!info] React et TypeScript : une combinaison native
> Depuis 2020, React est entierement type via le package `@types/react`. Vite genere des projets TypeScript par defaut. Ce n'est plus un "bonus avance" — c'est le standard industriel.

---

## 1. Setup du projet avec Vite

### Creer un nouveau projet

La methode la plus rapide en 2024 :

```bash
npm create vite@latest mon-app -- --template react-ts
cd mon-app
npm install
npm run dev
```

La structure generee :

```
mon-app/
├── src/
│   ├── App.tsx          # Composant racine
│   ├── main.tsx         # Point d'entree
│   ├── vite-env.d.ts    # Types Vite (import.meta.env, etc.)
│   └── assets/
├── index.html
├── package.json
├── tsconfig.json        # Config TypeScript
├── tsconfig.app.json    # Config specifique a l'app
└── vite.config.ts       # Config Vite (notice : .ts, pas .js)
```

### Le tsconfig.json essentiel

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsFiles": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

> [!warning] `strict: true` est non-negociable
> Active `strictNullChecks`, `noImplicitAny`, et une dizaine d'autres verifications. Sans ca, TypeScript est beaucoup moins efficace — tu peux passer une valeur `undefined` la ou tu t'attends a un `string` sans que le compilateur proteste. Active-le toujours, meme si ca genere des erreurs au debut.

### Ajouter TypeScript a un projet existant

```bash
npm install --save-dev typescript @types/react @types/react-dom
npx tsc --init
```

Renomme ensuite tes fichiers `.jsx` en `.tsx` progressivement.

---

## 2. Typer les composants : interface vs type

### La structure d'un composant type

```tsx
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary' | 'danger';
}

function Button({ label, onClick, disabled = false, variant = 'primary' }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {label}
    </button>
  );
}

export default Button;
```

### interface vs type : lequel choisir ?

Les deux fonctionnent pour typer des props. La difference est subtile mais importante :

| Critere | `interface` | `type` |
|---|---|---|
| Extension | `extends` naturel | `&` (intersection) |
| Fusion de declaration | Oui (mergeable) | Non |
| Types primitifs | Non | Oui (`type ID = string`) |
| Unions | Non | Oui (`type Status = 'ok' \| 'err'`) |
| Lisibilite d'erreur | Meilleure | Parfois plus complexe |
| Convention React | Recommande pour les props | Pour les unions et aliases |

**Regle pratique** :
- `interface` pour les props et les objets metier
- `type` pour les unions, les aliases de types simples, et les types derives (comme `Partial<T>`)

```tsx
// Props -> interface
interface CardProps {
  title: string;
  content: string;
}

// Union -> type
type ButtonVariant = 'primary' | 'secondary' | 'ghost';

// Type derive -> type
type PartialCardProps = Partial<CardProps>;

// Equivalent fonctionnel pour les objets
type CardPropsAsType = {
  title: string;
  content: string;
};
```

> [!example] Extension d'interface
> ```tsx
> interface BaseProps {
>   className?: string;
>   style?: React.CSSProperties;
> }
> 
> interface TextInputProps extends BaseProps {
>   value: string;
>   onChange: (value: string) => void;
>   placeholder?: string;
> }
> ```
> Avec `type`, on ecrirait `TextInputProps = BaseProps & { value: string; ... }`. Les deux sont valides — choisir selon le contexte.

---

## 3. React.FC vs function declaration

### Le probleme avec `React.FC`

Tu verras souvent du code comme ca dans des tutoriels anciens :

```tsx
// PATTERN OBSOLETE - ne pas reproduire
const Button: React.FC<ButtonProps> = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};
```

Ce pattern est **deconseille depuis React 18** pour plusieurs raisons :

```
React.FC<P> ajoute implicitement :
  1. children?: ReactNode  ← force children sur TOUS les composants
  2. displayName, defaultProps, propTypes ← heritage React class
  3. Type de retour JSX.Element ← pas React.ReactNode

Problemes concrets :
  - Ton composant Button accepte children meme si tu ne veux pas
  - Les generiques sont plus difficiles a ecrire avec React.FC
  - Plus verbeux pour zero benefice
```

### La bonne pratique : function declaration explicite

```tsx
// PATTERN RECOMMANDE
function Button({ label, onClick, disabled = false }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}

// Ou arrow function sans React.FC
const Button = ({ label, onClick }: ButtonProps) => (
  <button onClick={onClick}>{label}</button>
);
```

### Typer le retour explicitement (optionnel mais propre)

```tsx
function UserAvatar({ name, avatarUrl }: UserAvatarProps): React.ReactElement {
  return (
    <div className="avatar">
      <img src={avatarUrl} alt={name} />
      <span>{name}</span>
    </div>
  );
}
```

> [!info] `JSX.Element` vs `React.ReactElement` vs `React.ReactNode`
> - `JSX.Element` : alias de `React.ReactElement<any>`, retourne toujours un element JSX concret
> - `React.ReactElement` : un element React avec un type et des props
> - `React.ReactNode` : tout ce que React peut rendre — string, number, null, boolean, fragment, tableau...
> 
> Pour le type de retour d'un composant, `React.ReactElement` ou `JSX.Element` sont equivalents. Prefere laisser TypeScript inferer le type de retour — c'est plus maintenable.

---

## 4. Props optionnelles et valeurs par defaut

### Marquer une prop comme optionnelle

```tsx
interface TooltipProps {
  content: string;           // obligatoire
  position?: 'top' | 'bottom' | 'left' | 'right';  // optionnel
  delay?: number;            // optionnel
  className?: string;        // optionnel
}
```

Le `?` signifie que la prop peut etre `undefined`. TypeScript va t'obliger a tester sa presence avant de l'utiliser.

### Valeurs par defaut : destructuring

```tsx
function Tooltip({
  content,
  position = 'top',
  delay = 300,
  className = ''
}: TooltipProps) {
  // position est garanti string ici, pas 'top' | 'bottom' | ... | undefined
  return (
    <div className={`tooltip tooltip-${position} ${className}`}>
      {content}
    </div>
  );
}
```

> [!warning] defaultProps est obsolete
> `Button.defaultProps = { variant: 'primary' }` est deprecie en React. Utilise uniquement le destructuring avec valeurs par defaut. TypeScript + destructuring est plus sur, plus lisible, et statiquement verifie.

### Props avec callbacks types

```tsx
interface SelectProps {
  options: Array<{ value: string; label: string }>;
  value: string;
  onChange: (value: string) => void;
  onFocus?: () => void;
  onBlur?: (event: React.FocusEvent<HTMLSelectElement>) => void;
}

function Select({ options, value, onChange, onFocus, onBlur }: SelectProps) {
  return (
    <select
      value={value}
      onChange={(e) => onChange(e.target.value)}
      onFocus={onFocus}
      onBlur={onBlur}
    >
      {options.map(({ value: optValue, label }) => (
        <option key={optValue} value={optValue}>
          {label}
        </option>
      ))}
    </select>
  );
}
```

---

## 5. Hooks types

### useState<T>

TypeScript infere souvent le type depuis la valeur initiale. Mais quand ca ne suffit pas, on annote explicitement.

```tsx
// Inference automatique - OK pour les types simples
const [count, setCount] = useState(0);          // number
const [name, setName] = useState('');           // string
const [isOpen, setIsOpen] = useState(false);    // boolean

// Annotation explicite - necessaire pour les types complexes
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<string[]>([]);
const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');

// Interface
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

function UserProfile() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);

  const fetchUser = async (id: number) => {
    setLoading(true);
    const response = await fetch(`/api/users/${id}`);
    const data: User = await response.json();
    setUser(data);
    setLoading(false);
  };

  if (loading) return <div>Chargement...</div>;
  if (!user) return <div>Aucun utilisateur</div>;

  // Ici TypeScript sait que user est de type User (plus null)
  return <div>{user.name} - {user.email}</div>;
}
```

### useRef<T>

`useRef` a deux usages distincts en React, chacun avec son type :

```tsx
// Usage 1 : reference vers un element DOM
// Type : RefObject<T> - la valeur peut etre null avant le montage
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const focusInput = () => {
    // inputRef.current peut etre null - on verifie
    inputRef.current?.focus();
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}

// Usage 2 : valeur mutable persistante (pas de re-render)
// Type : MutableRefObject<T> - valeur jamais null
function Timer() {
  const intervalRef = useRef<ReturnType<typeof setInterval> | null>(null);
  const [seconds, setSeconds] = useState(0);

  const start = () => {
    intervalRef.current = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
  };

  const stop = () => {
    if (intervalRef.current !== null) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  return (
    <div>
      <span>{seconds}s</span>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

> [!info] `useRef<HTMLInputElement>(null)` vs `useRef<HTMLInputElement | null>(null)`
> Pour les refs DOM, utilise `useRef<HTMLInputElement>(null)`. TypeScript donnera un `RefObject<HTMLInputElement>` (propriete `current` en lecture seule). Pour les valeurs mutables, `useRef<number>(0)` donne `MutableRefObject<number>` (propriete `current` modifiable).

### useReducer

`useReducer` brille quand l'etat est complexe avec plusieurs transitions. TypeScript nous force a etre exhaustifs sur les actions.

```tsx
// Definir l'etat
interface CounterState {
  count: number;
  step: number;
  history: number[];
}

// Union discriminee pour les actions
type CounterAction =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'RESET' }
  | { type: 'SET_STEP'; payload: number }
  | { type: 'SET_COUNT'; payload: number };

// Reducer typee
function counterReducer(state: CounterState, action: CounterAction): CounterState {
  switch (action.type) {
    case 'INCREMENT':
      return {
        ...state,
        count: state.count + state.step,
        history: [...state.history, state.count + state.step],
      };
    case 'DECREMENT':
      return {
        ...state,
        count: state.count - state.step,
        history: [...state.history, state.count - state.step],
      };
    case 'RESET':
      return { count: 0, step: 1, history: [] };
    case 'SET_STEP':
      return { ...state, step: action.payload };  // payload: number garanti
    case 'SET_COUNT':
      return { ...state, count: action.payload, history: [...state.history, action.payload] };
    // TypeScript verifie l'exhaustivite - si tu oublies un case, erreur de compile
  }
}

const initialState: CounterState = { count: 0, step: 1, history: [] };

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);

  return (
    <div>
      <p>Count: {state.count} (step: {state.step})</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
      <input
        type="number"
        value={state.step}
        onChange={(e) => dispatch({ type: 'SET_STEP', payload: Number(e.target.value) })}
      />
    </div>
  );
}
```

### useEffect et dependances

`useEffect` lui-meme n'a pas de parametre de type — ses types viennent des closures capturees.

```tsx
interface Post {
  id: number;
  title: string;
  body: string;
  userId: number;
}

function PostDetail({ postId }: { postId: number }) {
  const [post, setPost] = useState<Post | null>(null);
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // L'effet s'execute quand postId change
    let cancelled = false; // evite les race conditions

    const fetchPost = async () => {
      try {
        setLoading(true);
        setError(null);
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${postId}`);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data: Post = await response.json();
        
        if (!cancelled) {
          setPost(data);
        }
      } catch (err) {
        if (!cancelled) {
          // err est de type 'unknown' en TypeScript strict
          setError(err instanceof Error ? err.message : 'Erreur inconnue');
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    fetchPost();

    // Cleanup : annule l'update si le composant est demonte
    return () => {
      cancelled = true;
    };
  }, [postId]); // postId est la dependance - TypeScript verifie que c'est une valeur reactive

  if (loading) return <p>Chargement...</p>;
  if (error) return <p>Erreur : {error}</p>;
  if (!post) return null;

  return (
    <article>
      <h2>{post.title}</h2>
      <p>{post.body}</p>
    </article>
  );
}
```

> [!warning] `err` est de type `unknown` avec `strict: true`
> En mode strict, les `catch (err)` donnent `err: unknown`, pas `err: any`. C'est une securite — tu dois verifier le type avant d'acceder a `err.message`. Le pattern `err instanceof Error ? err.message : String(err)` est la solution standard.

---

## 6. Context API avec TypeScript

Le Context est souvent mal type. Voici le pattern robuste.

### Pattern complet avec createContext

```tsx
// theme-context.tsx

type Theme = 'light' | 'dark' | 'system';

interface ThemeContextValue {
  theme: Theme;
  setTheme: (theme: Theme) => void;
  isDark: boolean;
}

// Ne jamais initialiser avec undefined sans le signaler
// Pattern 1 : valeur par defaut realiste (recommande si possible)
const ThemeContext = React.createContext<ThemeContextValue>({
  theme: 'system',
  setTheme: () => {}, // no-op par defaut
  isDark: false,
});

// Provider
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('system');
  
  const isDark = theme === 'dark' || 
    (theme === 'system' && window.matchMedia('(prefers-color-scheme: dark)').matches);

  return (
    <ThemeContext.Provider value={{ theme, setTheme, isDark }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook - encapsule useContext + verification
function useTheme(): ThemeContextValue {
  const context = React.useContext(ThemeContext);
  return context;
}

export { ThemeProvider, useTheme };
```

### Pattern avec valeur initiale undefined (pour forcer le Provider)

```tsx
// auth-context.tsx

interface AuthContextValue {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

// Context avec undefined - oblige a utiliser le Provider
const AuthContext = React.createContext<AuthContextValue | undefined>(undefined);

function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);

  const login = async (email: string, password: string) => {
    setIsLoading(true);
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });
      const userData: User = await response.json();
      setUser(userData);
    } finally {
      setIsLoading(false);
    }
  };

  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook avec garde - lance une erreur explicite si mal utilise
function useAuth(): AuthContextValue {
  const context = React.useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth doit etre utilise dans un AuthProvider');
  }
  return context;
}

export { AuthProvider, useAuth };
```

```
Context Flow :

  <AuthProvider>              ← Initialise le state + fournit les callbacks
    <ThemeProvider>           ← Nest autant de providers que necessaire
      <App>
        <Navbar>
          useAuth()           ← Consomme AuthContext : {user, login, logout}
          useTheme()          ← Consomme ThemeContext : {theme, setTheme}
        </Navbar>
      </App>
    </ThemeProvider>
  </AuthProvider>
```

---

## 7. Types d'evenements

React wraps les evenements natifs du DOM dans ses propres types. Voici les plus courants.

### Tableau des types d'evenements React

| Evenement | Type React | Element cible typique |
|---|---|---|
| Click | `React.MouseEvent<HTMLButtonElement>` | button, div, a |
| Change input | `React.ChangeEvent<HTMLInputElement>` | input |
| Change select | `React.ChangeEvent<HTMLSelectElement>` | select |
| Change textarea | `React.ChangeEvent<HTMLTextAreaElement>` | textarea |
| Submit form | `React.FormEvent<HTMLFormElement>` | form |
| Clavier | `React.KeyboardEvent<HTMLInputElement>` | input, div |
| Focus | `React.FocusEvent<HTMLInputElement>` | input |
| Drag | `React.DragEvent<HTMLDivElement>` | div |
| Scroll | `React.UIEvent<HTMLDivElement>` | div |

### Exemples concrets

```tsx
function EventExamples() {
  // MouseEvent : acces a clientX, clientY, button, etc.
  const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
    console.log('Position:', event.clientX, event.clientY);
    console.log('Bouton:', event.button); // 0=gauche, 1=milieu, 2=droite
    event.preventDefault();
  };

  // ChangeEvent : acces a target.value
  const handleInputChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const value: string = event.target.value;
    const checked: boolean = event.target.checked; // pour les checkboxes
    console.log(value);
  };

  // FormEvent : empêche la soumission native
  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    const form = event.currentTarget;
    const data = new FormData(form);
    console.log(Object.fromEntries(data));
  };

  // KeyboardEvent : acces a key, code, ctrlKey, etc.
  const handleKeyDown = (event: React.KeyboardEvent<HTMLInputElement>) => {
    if (event.key === 'Enter') {
      console.log('Entree pressee');
    }
    if (event.ctrlKey && event.key === 's') {
      event.preventDefault();
      console.log('Ctrl+S intercepte');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <button onClick={handleClick} type="button">Click</button>
      <input
        onChange={handleInputChange}
        onKeyDown={handleKeyDown}
      />
      <button type="submit">Envoyer</button>
    </form>
  );
}
```

### Handlers inline vs extraits

```tsx
// Inline : TypeScript infere le type depuis le contexte JSX
function InlineExample() {
  return (
    <input
      onChange={(e) => console.log(e.target.value)}
      // e est automatiquement React.ChangeEvent<HTMLInputElement>
    />
  );
}

// Extrait : on doit annoter manuellement
function ExtractedExample() {
  // Annotation obligatoire car TypeScript n'a pas le contexte JSX
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
  };

  return <input onChange={handleChange} />;
}
```

> [!tip] Type rapide pour les handlers
> `React.ChangeEventHandler<HTMLInputElement>` est un alias pour `(event: React.ChangeEvent<HTMLInputElement>) => void`. Plus court pour les props de callback :
> ```tsx
> interface InputProps {
>   onChange: React.ChangeEventHandler<HTMLInputElement>;
> }
> ```

---

## 8. Composants generiques

Les generiques permettent de creer des composants reutilisables qui fonctionnent avec n'importe quel type de donnees.

### Probleme : une liste non typee

```tsx
// Sans generique - perd l'information de type
function List({ items }: { items: unknown[] }) {
  return <ul>{items.map((item, i) => <li key={i}>{String(item)}</li>)}</ul>;
}
```

### Solution : composant generique

```tsx
// Composant generique - preserve le type
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
  emptyMessage?: string;
}

function List<T>({ items, renderItem, keyExtractor, emptyMessage = 'Aucun element' }: ListProps<T>) {
  if (items.length === 0) {
    return <p className="empty">{emptyMessage}</p>;
  }

  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// Utilisation avec users
interface User {
  id: number;
  name: string;
  email: string;
}

function UserList({ users }: { users: User[] }) {
  return (
    <List
      items={users}
      keyExtractor={(user) => user.id}   // TypeScript sait que user est User
      renderItem={(user) => (
        <div>
          <strong>{user.name}</strong>
          <span>{user.email}</span>       // .email est autocomplete et verifie
        </div>
      )}
      emptyMessage="Aucun utilisateur"
    />
  );
}

// Meme composant avec des produits
interface Product {
  sku: string;
  name: string;
  price: number;
}

function ProductList({ products }: { products: Product[] }) {
  return (
    <List
      items={products}
      keyExtractor={(product) => product.sku}
      renderItem={(product) => (
        <div>
          {product.name} — {product.price}€
        </div>
      )}
    />
  );
}
```

### Generique avec contrainte

```tsx
// T doit avoir au moins un champ 'id'
interface WithId {
  id: string | number;
}

interface SelectableListProps<T extends WithId> {
  items: T[];
  selectedId: string | number | null;
  onSelect: (item: T) => void;
  renderItem: (item: T) => React.ReactNode;
}

function SelectableList<T extends WithId>({
  items,
  selectedId,
  onSelect,
  renderItem,
}: SelectableListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li
          key={item.id}  // OK car T extends WithId - item.id garanti
          onClick={() => onSelect(item)}
          className={item.id === selectedId ? 'selected' : ''}
        >
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}
```

> [!example] Composant generique avec arrow function et `.tsx`
> En fichiers `.tsx`, la syntaxe `<T>` peut etre confondue avec une balise JSX. Deux solutions :
> ```tsx
> // Solution 1 : contrainte vide
> const Component = <T extends object>(props: Props<T>) => { ... };
> 
> // Solution 2 : trailing comma (syntaxe TSX)
> const Component = <T,>(props: Props<T>) => { ... };
> 
> // Solution 3 : function declaration (pas ce probleme)
> function Component<T>(props: Props<T>) { ... }
> ```

---

## 9. children : ReactNode, ReactElement, PropsWithChildren

Le type de `children` est une source de confusion. Voici la distinction claire.

### La hierarchie des types

```
React.ReactNode                    ← Le plus large
├── React.ReactElement             ← Un element JSX concret (<div>, <MyComp>)
│   └── JSX.Element                ← Alias pratique de ReactElement<any>
├── string
├── number
├── boolean
├── null
├── undefined
└── React.ReactNode[]              ← Tableau de tout ce qui precede
```

### Quand utiliser quoi

```tsx
// ReactNode : accepte tout ce que React peut rendre (recommande pour children)
interface ContainerProps {
  children: React.ReactNode;
}

function Container({ children }: ContainerProps) {
  return <div className="container">{children}</div>;
}

// Utilisation : tout est valide
<Container>
  <p>Du texte</p>           // ReactElement - OK
  <span>Autre</span>        // ReactElement - OK
  Texte brut                // string - OK
  {42}                      // number - OK
  {null}                    // null - OK (ne rend rien)
  {condition && <Comp />}   // boolean | ReactElement - OK
</Container>
```

```tsx
// ReactElement : un seul element JSX concret (pas de string, number, null)
interface IconButtonProps {
  icon: React.ReactElement;  // Doit etre un composant React
  label: string;
}

function IconButton({ icon, label }: IconButtonProps) {
  return (
    <button>
      {React.cloneElement(icon, { className: 'icon' })}
      <span>{label}</span>
    </button>
  );
}

// Utilisation
<IconButton icon={<SearchIcon />} label="Rechercher" />
// <IconButton icon="texte" /> ← Erreur TypeScript : string n'est pas ReactElement
```

### PropsWithChildren : raccourci propre

```tsx
// Equivaut a ajouter children?: React.ReactNode
import React, { PropsWithChildren } from 'react';

interface CardProps {
  title: string;
  className?: string;
}

// Sans PropsWithChildren
function Card({ title, children }: CardProps & { children: React.ReactNode }) {
  return <div className="card"><h3>{title}</h3>{children}</div>;
}

// Avec PropsWithChildren - plus lisible
function Card({ title, children, className }: PropsWithChildren<CardProps>) {
  return (
    <div className={`card ${className ?? ''}`}>
      <h3>{title}</h3>
      <div className="card-content">{children}</div>
    </div>
  );
}
```

> [!warning] Ne pas utiliser `React.FC` pour avoir children automatiquement
> Avant React 18, `React.FC` ajoutait `children` implicitement. Ce n'est plus le cas. La solution propre est `PropsWithChildren<YourProps>` ou `children: React.ReactNode` explicite dans ton interface.

---

## 10. Custom Hooks types

Les custom hooks sont des fonctions qui commencent par `use` et peuvent appeler d'autres hooks. TypeScript les type naturellement par inference.

### Pattern de base

```tsx
// useLocalStorage.ts
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? (JSON.parse(item) as T) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value: T) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('useLocalStorage setValue error:', error);
    }
  };

  return [storedValue, setValue];
}

// Utilisation - T infere depuis initialValue
const [theme, setTheme] = useLocalStorage('theme', 'light');
// theme : string, setTheme : (value: string) => void

const [user, setUser] = useLocalStorage<User | null>('user', null);
// user : User | null, setUser : (value: User | null) => void
```

### Hook avec type de retour complexe

```tsx
// useFetch.ts
interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

function useFetch<T>(url: string): FetchState<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [trigger, setTrigger] = useState(0);

  useEffect(() => {
    let cancelled = false;

    const fetchData = async () => {
      setLoading(true);
      setError(null);
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const json: T = await response.json();
        if (!cancelled) setData(json);
      } catch (err) {
        if (!cancelled) {
          setError(err instanceof Error ? err.message : 'Erreur');
        }
      } finally {
        if (!cancelled) setLoading(false);
      }
    };

    fetchData();
    return () => { cancelled = true; };
  }, [url, trigger]);

  const refetch = () => setTrigger(t => t + 1);

  return { data, loading, error, refetch };
}

// Utilisation
interface ApiUser {
  id: number;
  name: string;
  username: string;
  email: string;
}

function UserCard({ userId }: { userId: number }) {
  const { data: user, loading, error, refetch } = useFetch<ApiUser>(
    `https://jsonplaceholder.typicode.com/users/${userId}`
  );
  // user est de type ApiUser | null - TypeScript verifie chaque acces

  if (loading) return <div>Chargement...</div>;
  if (error) return <div>Erreur : {error} <button onClick={refetch}>Reessayer</button></div>;
  if (!user) return null;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>@{user.username}</p>
      <p>{user.email}</p>
    </div>
  );
}
```

### Hook retournant un tuple

```tsx
// useToggle.ts - retourne un tuple comme useState
function useToggle(initialValue = false): [boolean, () => void, (value: boolean) => void] {
  const [value, setValue] = useState(initialValue);
  const toggle = () => setValue(v => !v);
  return [value, toggle, setValue];
}

// useDisclosure.ts - retourne un objet (plus lisible pour plusieurs valeurs)
interface DisclosureReturn {
  isOpen: boolean;
  open: () => void;
  close: () => void;
  toggle: () => void;
}

function useDisclosure(initialValue = false): DisclosureReturn {
  const [isOpen, setIsOpen] = useState(initialValue);
  return {
    isOpen,
    open: () => setIsOpen(true),
    close: () => setIsOpen(false),
    toggle: () => setIsOpen(v => !v),
  };
}

// Utilisation
function Modal() {
  const { isOpen, open, close } = useDisclosure();

  return (
    <>
      <button onClick={open}>Ouvrir</button>
      {isOpen && (
        <div className="modal">
          <button onClick={close}>Fermer</button>
        </div>
      )}
    </>
  );
}
```

---

## 11. Exemple complet : formulaire type de A a Z

Voici un formulaire d'inscription complet qui integre tous les concepts vus.

### Types et validation

```tsx
// types/form.ts

export interface RegistrationFormData {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  confirmPassword: string;
  role: 'student' | 'teacher' | 'admin';
  agreeToTerms: boolean;
  birthYear: number;
}

export type FormErrors = Partial<Record<keyof RegistrationFormData, string>>;

export type FormField = keyof RegistrationFormData;
```

### Hook de validation

```tsx
// hooks/useFormValidation.ts
import { RegistrationFormData, FormErrors } from '../types/form';

function validateRegistration(data: RegistrationFormData): FormErrors {
  const errors: FormErrors = {};

  if (!data.firstName.trim()) {
    errors.firstName = 'Le prenom est requis';
  } else if (data.firstName.length < 2) {
    errors.firstName = 'Le prenom doit contenir au moins 2 caracteres';
  }

  if (!data.lastName.trim()) {
    errors.lastName = 'Le nom est requis';
  }

  if (!data.email.includes('@')) {
    errors.email = 'Email invalide';
  }

  if (data.password.length < 8) {
    errors.password = 'Le mot de passe doit contenir au moins 8 caracteres';
  } else if (!/[A-Z]/.test(data.password)) {
    errors.password = 'Le mot de passe doit contenir au moins une majuscule';
  } else if (!/[0-9]/.test(data.password)) {
    errors.password = 'Le mot de passe doit contenir au moins un chiffre';
  }

  if (data.password !== data.confirmPassword) {
    errors.confirmPassword = 'Les mots de passe ne correspondent pas';
  }

  if (data.birthYear < 1900 || data.birthYear > new Date().getFullYear() - 13) {
    errors.birthYear = 'Age invalide (minimum 13 ans)';
  }

  if (!data.agreeToTerms) {
    errors.agreeToTerms = 'Vous devez accepter les conditions';
  }

  return errors;
}

interface UseFormValidationReturn {
  errors: FormErrors;
  validate: (data: RegistrationFormData) => boolean;
  clearError: (field: keyof RegistrationFormData) => void;
  clearAllErrors: () => void;
}

export function useFormValidation(): UseFormValidationReturn {
  const [errors, setErrors] = useState<FormErrors>({});

  const validate = (data: RegistrationFormData): boolean => {
    const newErrors = validateRegistration(data);
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const clearError = (field: keyof RegistrationFormData) => {
    setErrors(prev => {
      const next = { ...prev };
      delete next[field];
      return next;
    });
  };

  const clearAllErrors = () => setErrors({});

  return { errors, validate, clearError, clearAllErrors };
}
```

### Composants de formulaire reutilisables

```tsx
// components/form/FormField.tsx

interface FormFieldProps {
  label: string;
  error?: string;
  required?: boolean;
  children: React.ReactNode;
  htmlFor?: string;
}

function FormField({ label, error, required = false, children, htmlFor }: FormFieldProps) {
  return (
    <div className={`form-field ${error ? 'has-error' : ''}`}>
      <label htmlFor={htmlFor}>
        {label}
        {required && <span className="required" aria-label="requis"> *</span>}
      </label>
      {children}
      {error && (
        <span className="error-message" role="alert">
          {error}
        </span>
      )}
    </div>
  );
}

// components/form/TextInput.tsx

interface TextInputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  error?: string;
}

const TextInput = React.forwardRef<HTMLInputElement, TextInputProps>(
  ({ error, className, ...rest }, ref) => (
    <input
      ref={ref}
      className={`text-input ${error ? 'error' : ''} ${className ?? ''}`}
      aria-invalid={error ? 'true' : 'false'}
      {...rest}
    />
  )
);
TextInput.displayName = 'TextInput';
```

### Le composant formulaire principal

```tsx
// components/RegistrationForm.tsx
import React, { useState } from 'react';
import { RegistrationFormData, FormField as FormFieldKey } from '../types/form';
import { useFormValidation } from '../hooks/useFormValidation';

interface RegistrationFormProps {
  onSuccess: (data: RegistrationFormData) => void;
  onCancel?: () => void;
}

const INITIAL_FORM_DATA: RegistrationFormData = {
  firstName: '',
  lastName: '',
  email: '',
  password: '',
  confirmPassword: '',
  role: 'student',
  agreeToTerms: false,
  birthYear: new Date().getFullYear() - 18,
};

function RegistrationForm({ onSuccess, onCancel }: RegistrationFormProps) {
  const [formData, setFormData] = useState<RegistrationFormData>(INITIAL_FORM_DATA);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitError, setSubmitError] = useState<string | null>(null);
  const { errors, validate, clearError } = useFormValidation();

  // Handler generique pour les champs texte
  const handleChange = (field: FormFieldKey) => (
    event: React.ChangeEvent<HTMLInputElement | HTMLSelectElement>
  ) => {
    const value = event.target.type === 'checkbox'
      ? (event.target as HTMLInputElement).checked
      : event.target.type === 'number'
        ? Number(event.target.value)
        : event.target.value;

    setFormData(prev => ({ ...prev, [field]: value }));
    clearError(field);
  };

  const handleSubmit = async (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    
    if (!validate(formData)) return;
    
    setIsSubmitting(true);
    setSubmitError(null);

    try {
      // Simulation d'un appel API
      await new Promise<void>((resolve, reject) => {
        setTimeout(() => {
          if (Math.random() > 0.1) {
            resolve();
          } else {
            reject(new Error('Erreur serveur. Reessayez.'));
          }
        }, 1000);
      });

      onSuccess(formData);
    } catch (err) {
      setSubmitError(err instanceof Error ? err.message : 'Erreur inconnue');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      <h2>Inscription</h2>

      {submitError && (
        <div className="alert alert-error" role="alert">
          {submitError}
        </div>
      )}

      <div className="form-row">
        <FormField label="Prenom" error={errors.firstName} required htmlFor="firstName">
          <input
            id="firstName"
            type="text"
            value={formData.firstName}
            onChange={handleChange('firstName')}
            placeholder="Jean"
            autoComplete="given-name"
          />
        </FormField>

        <FormField label="Nom" error={errors.lastName} required htmlFor="lastName">
          <input
            id="lastName"
            type="text"
            value={formData.lastName}
            onChange={handleChange('lastName')}
            placeholder="Dupont"
            autoComplete="family-name"
          />
        </FormField>
      </div>

      <FormField label="Email" error={errors.email} required htmlFor="email">
        <input
          id="email"
          type="email"
          value={formData.email}
          onChange={handleChange('email')}
          placeholder="jean.dupont@exemple.fr"
          autoComplete="email"
        />
      </FormField>

      <FormField label="Mot de passe" error={errors.password} required htmlFor="password">
        <input
          id="password"
          type="password"
          value={formData.password}
          onChange={handleChange('password')}
          autoComplete="new-password"
        />
      </FormField>

      <FormField label="Confirmer le mot de passe" error={errors.confirmPassword} required htmlFor="confirmPassword">
        <input
          id="confirmPassword"
          type="password"
          value={formData.confirmPassword}
          onChange={handleChange('confirmPassword')}
          autoComplete="new-password"
        />
      </FormField>

      <FormField label="Role" error={errors.role} required htmlFor="role">
        <select
          id="role"
          value={formData.role}
          onChange={handleChange('role')}
        >
          <option value="student">Etudiant</option>
          <option value="teacher">Enseignant</option>
          <option value="admin">Administrateur</option>
        </select>
      </FormField>

      <FormField label="Annee de naissance" error={errors.birthYear} required htmlFor="birthYear">
        <input
          id="birthYear"
          type="number"
          value={formData.birthYear}
          onChange={handleChange('birthYear')}
          min={1900}
          max={new Date().getFullYear() - 13}
        />
      </FormField>

      <FormField label="" error={errors.agreeToTerms} htmlFor="agreeToTerms">
        <label className="checkbox-label">
          <input
            id="agreeToTerms"
            type="checkbox"
            checked={formData.agreeToTerms}
            onChange={handleChange('agreeToTerms')}
          />
          J'accepte les conditions generales d'utilisation
        </label>
      </FormField>

      <div className="form-actions">
        {onCancel && (
          <button type="button" onClick={onCancel} disabled={isSubmitting}>
            Annuler
          </button>
        )}
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Inscription en cours...' : 'S\'inscrire'}
        </button>
      </div>
    </form>
  );
}

export default RegistrationForm;
```

### Utilisation du formulaire

```tsx
// App.tsx
function App() {
  const [registeredUser, setRegisteredUser] = useState<RegistrationFormData | null>(null);

  const handleSuccess = (data: RegistrationFormData) => {
    setRegisteredUser(data);
    console.log('Inscription reussie:', data);
  };

  if (registeredUser) {
    return (
      <div>
        <h1>Bienvenue, {registeredUser.firstName} !</h1>
        <p>Votre compte a ete cree avec succes.</p>
      </div>
    );
  }

  return (
    <div className="app">
      <RegistrationForm
        onSuccess={handleSuccess}
        onCancel={() => console.log('Annule')}
      />
    </div>
  );
}
```

---

## 12. Patterns avances

### Discriminated Union pour les etats de chargement

```tsx
// Pattern "state machine" tres propre avec TypeScript
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function useAsyncData<T>(url: string): AsyncState<T> {
  const [state, setState] = useState<AsyncState<T>>({ status: 'idle' });

  useEffect(() => {
    setState({ status: 'loading' });
    fetch(url)
      .then(r => r.json())
      .then((data: T) => setState({ status: 'success', data }))
      .catch((err: Error) => setState({ status: 'error', error: err.message }));
  }, [url]);

  return state;
}

// Utilisation avec narrowing exhaustif
function DataDisplay({ url }: { url: string }) {
  const state = useAsyncData<User[]>(url);

  switch (state.status) {
    case 'idle':    return null;
    case 'loading': return <Spinner />;
    case 'error':   return <ErrorMessage message={state.error} />;
    case 'success': return <UserList users={state.data} />;
    // TypeScript verifie l'exhaustivite - si on ajoute un status, erreur ici
  }
}
```

### Composant polymorphe (as prop)

```tsx
// Composant qui peut rendre differentes balises HTML
type PolymorphicProps<C extends React.ElementType> = {
  as?: C;
  children: React.ReactNode;
  className?: string;
} & Omit<React.ComponentPropsWithoutRef<C>, 'as' | 'children' | 'className'>;

function Text<C extends React.ElementType = 'p'>({
  as,
  children,
  className,
  ...rest
}: PolymorphicProps<C>) {
  const Component = as ?? 'p';
  return (
    <Component className={className} {...rest}>
      {children}
    </Component>
  );
}

// Utilisation
<Text>Paragraphe par defaut</Text>
<Text as="h1">Titre H1</Text>
<Text as="span" onClick={() => {}}>Span cliquable</Text>
<Text as="a" href="/page">Lien typique</Text>
// href est propose en autocompletion uniquement quand as="a"
```

### forwardRef avec TypeScript

```tsx
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

// forwardRef necessite deux parametres de type : element et props
const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, id, ...rest }, ref) => {
    const inputId = id ?? `input-${label.toLowerCase().replace(/\s/g, '-')}`;
    
    return (
      <div className="input-wrapper">
        <label htmlFor={inputId}>{label}</label>
        <input
          id={inputId}
          ref={ref}
          aria-invalid={error ? 'true' : 'false'}
          aria-describedby={error ? `${inputId}-error` : undefined}
          {...rest}
        />
        {error && (
          <span id={`${inputId}-error`} className="error">
            {error}
          </span>
        )}
      </div>
    );
  }
);

Input.displayName = 'Input'; // Obligatoire avec forwardRef pour les DevTools

// Utilisation avec ref
function Form() {
  const emailRef = useRef<HTMLInputElement>(null);

  const focusEmail = () => emailRef.current?.focus();

  return (
    <form>
      <Input ref={emailRef} label="Email" type="email" />
      <button type="button" onClick={focusEmail}>Focus email</button>
    </form>
  );
}
```

---

## 13. Erreurs courantes et solutions

### Tableau des pieges TypeScript + React

| Erreur | Cause | Solution |
|---|---|---|
| `Object is possibly null` | Ref DOM avant montage | `ref.current?.method()` ou guard |
| `Type 'string' is not assignable to 'ButtonVariant'` | Valeur dynamique non narrowee | `as ButtonVariant` ou validation runtime |
| `Property 'X' does not exist on type 'EventTarget'` | target non type | Caster : `(e.target as HTMLInputElement).value` |
| `Cannot read properties of undefined (reading 'X')` | Context hors Provider | Custom hook avec guard `if (!context) throw` |
| `JSX element type does not have 'children' prop` | FC obsolete ou children manquant | Ajouter `children: React.ReactNode` explicitement |
| `useEffect ran twice in development` | React.StrictMode intentionnel | Normal — c'est une detection d'effets non idempotents |
| `Type 'void' is not assignable to type 'ReactNode'` | Fonction sans return dans JSX | Ajouter `return null` ou corriger la fonction |

### Le piege `as` : ne pas en abuser

```tsx
// MAUVAIS : forcer le type sans verification
const user = data as User; // Peut cacher un bug si data ne correspond pas

// MIEUX : type guard
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    'email' in value
  );
}

if (isUser(data)) {
  // TypeScript sait que data est User ici
  console.log(data.name);
}
```

---

## Carte Mentale

```
TypeScript + React
│
├── SETUP
│   ├── Vite : npm create vite@latest -- --template react-ts
│   ├── tsconfig : strict: true obligatoire
│   └── Extensions : .tsx (JSX) / .ts (logic pure)
│
├── COMPOSANTS
│   ├── Props → interface (objets) ou type (unions/aliases)
│   ├── Declaration → function Comp(props: P) { } ← RECOMMANDE
│   ├── React.FC → EVITER (children implicite, generiques difficiles)
│   └── forwardRef → React.forwardRef<HTMLElement, Props>(fn)
│
├── HOOKS
│   ├── useState<T>(initial)
│   │   ├── Infere pour string/number/boolean
│   │   └── Annoter pour T | null, unions, objets
│   ├── useRef<HTMLElement>(null) → DOM refs
│   ├── useRef<T>(value) → valeurs mutables
│   ├── useReducer(reducer, initial) → Action en union discriminee
│   └── useEffect → err: unknown dans catch
│
├── CONTEXT
│   ├── createContext<T>(defaultValue)
│   ├── Provider avec value typee
│   └── Custom hook useXxx() avec guard undefined
│
├── EVENEMENTS
│   ├── MouseEvent<HTMLButtonElement>
│   ├── ChangeEvent<HTMLInputElement | HTMLSelectElement>
│   ├── FormEvent<HTMLFormElement>
│   └── KeyboardEvent<HTMLElement>
│
├── PATTERNS AVANCES
│   ├── Composants generiques : <T>(props: Props<T>) => JSX
│   ├── children : ReactNode (tout) > ReactElement (JSX seul)
│   ├── PropsWithChildren<P> = P & { children?: ReactNode }
│   ├── Discriminated Union pour AsyncState<T>
│   └── Composant polymorphe avec "as" prop
│
└── PIEGES
    ├── err: unknown → instanceof Error check obligatoire
    ├── as cast → preferer les type guards
    └── React.FC → ne plus utiliser
```

---

## Exercices Pratiques

### Exercice 1 — Composant Card type

Cree un composant `ProductCard` avec les specs suivantes :

**Specs :**
- Props : `name` (string, obligatoire), `price` (number, obligatoire), `description` (string, optionnel), `inStock` (boolean, defaut `true`), `category` (`'electronics' | 'clothing' | 'food'`)
- Affiche le prix formate en euros avec `toLocaleString('fr-FR', { style: 'currency', currency: 'EUR' })`
- Badge "Rupture de stock" si `inStock` est false
- La category doit etre une union discriminee, pas un `string` libre
- Ajoute un bouton "Ajouter au panier" desactive si pas en stock, avec un callback `onAddToCart: (productName: string) => void`

**Ce que tu pratiques :** interfaces, props optionnelles, union types, callbacks types.

---

### Exercice 2 — Custom hook `usePagination`

Cree un hook `usePagination<T>` qui prend :
- `items: T[]` — la liste complete
- `itemsPerPage: number` — elements par page (defaut 10)

Et retourne :
- `currentPage: number`
- `totalPages: number`
- `currentItems: T[]` — les items de la page courante
- `goToPage: (page: number) => void`
- `nextPage: () => void`
- `prevPage: () => void`
- `hasNextPage: boolean`
- `hasPrevPage: boolean`

Contraintes : `goToPage` doit etre sans effet si la page est hors limites. Teste le hook avec un tableau de nombres et un tableau d'objets `User`.

**Ce que tu pratiques :** custom hooks generiques, type de retour explicite, bornes.

---

### Exercice 3 — Context de panier d'achat

Cree un contexte `CartContext` complet :

**Types necessaires :**
```tsx
interface CartItem {
  productId: string;
  name: string;
  price: number;
  quantity: number;
}
```

**Actions disponibles :**
- `addItem(item: Omit<CartItem, 'quantity'>): void`
- `removeItem(productId: string): void`
- `updateQuantity(productId: string, quantity: number): void`
- `clearCart(): void`
- Computed : `totalItems: number`, `totalPrice: number`

**Contraintes :**
- Utilise `useReducer` avec une union discriminee pour les actions
- Le `CartProvider` doit persister dans `localStorage` via `useEffect`
- Le custom hook `useCart()` doit lancer une erreur si utilise hors Provider
- `addItem` sur un produit deja present incremente la quantite au lieu de dupliquer

**Ce que tu pratiques :** useReducer type, Context API, union discriminee, Omit.

---

### Exercice 4 — Formulaire de recherche avec debounce

Cree un composant `SearchBar` avec :
- Un champ texte controle
- Un hook `useDebounce<T>(value: T, delay: number): T` qui retarde l'emission de la valeur
- Un hook `useSearch(query: string)` qui appelle `https://jsonplaceholder.typicode.com/users?q=${query}` (si query >= 2 chars) et retourne `AsyncState<User[]>`
- Affiche les resultats dans une liste ou les messages idle/loading/error selon l'etat
- Gere la navigation clavier (fleches haut/bas pour selectionner, Entree pour valider) avec les bons types `KeyboardEvent`

**Ce que tu pratiques :** Discriminated Union `AsyncState<T>`, generiques, evenements clavier, composition de hooks.

---

> [!info] Ressources complementaires
> - Documentation officielle TypeScript + React : https://react-typescript-cheatsheet.netlify.app/
> - Documentation TypeScript : https://www.typescriptlang.org/docs/
> - `@types/react` source : https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react
> - Pour aller plus loin sur les generiques : [[TypeScript/02 - Types Avances]]
> - Pour approfondir les hooks : [[React/02 - Hooks et Gestion d'Etat]]
> - Architecture de composants plus complexes : [[React/04 - React Avance et Tests]]

---

*Note creee le 2026-05-29 — [[TypeScript/01 - Introduction a TypeScript]] | [[React/01 - Introduction a React]]*
