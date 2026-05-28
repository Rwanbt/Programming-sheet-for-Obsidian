# React Avance et Tests

Maitriser React au niveau avancé, c'est comprendre quand l'application devient lente et pourquoi, connaitre les patterns architecturaux qui rendent le code maintenable a grande echelle, et tester son code avec confiance. Ce cours couvre l'optimisation des performances, les patterns avances (Error Boundaries, Portals, HOC), Zustand comme alternative legere a Redux, React Query pour le fetching, et les tests avec Vitest et React Testing Library.

> [!tip] Analogie
> Un bon architecte ne dessine pas seulement des beaux batiments : il anticipe les flux de circulation, prevoir les acces d'urgence, et s'assure que chaque element peut etre inspecte et remplace independamment. En React avance, c'est pareil : vous anticipez les goulots d'etranglement, vous concevez des composants modulaires, et vous ecrivez des tests qui servent de documentation vivante.

---

## 1. Performance React

### Identifier les problemes avec React DevTools

Avant d'optimiser, mesurer. React DevTools Profiler est l'outil indispensable.

```
Installation : Extension navigateur "React Developer Tools"

Onglet Profiler :
1. Cliquer "Start profiling"
2. Effectuer les actions lentes
3. Cliquer "Stop profiling"
4. Analyser le flamegraph :
   - Barre grise = composant non render
   - Barre coloree = composant render (plus jaune/rouge = plus lent)
   - Hover = temps de render et raison du re-render
```

### React.memo — Eviter les re-renders inutiles

```jsx
import { memo, useState } from "react";

// ─── Sans memo : re-render a chaque render du parent ───
function LigneTableau({ item, onEditer, onSupprimer }) {
    console.log(`Render LigneTableau ${item.id}`);
    return (
        <tr>
            <td>{item.nom}</td>
            <td>{item.prix} €</td>
            <td>
                <button onClick={() => onEditer(item.id)}>Editer</button>
                <button onClick={() => onSupprimer(item.id)}>Supprimer</button>
            </td>
        </tr>
    );
}

// ─── Avec memo : re-render UNIQUEMENT si les props changent ───
const LigneTableauMemo = memo(LigneTableau);

// ─── Comparaison personnalisee (si les props sont des objets complexes) ───
const LigneTableauPersonnalisee = memo(
    LigneTableau,
    (prevProps, nextProps) => {
        // Retourner true = PAS de re-render (props "egales")
        // Retourner false = re-render
        return (
            prevProps.item.id === nextProps.item.id &&
            prevProps.item.nom === nextProps.item.nom &&
            prevProps.item.prix === nextProps.item.prix
        );
    }
);
```

### Virtualisation avec react-window

Pour les listes de milliers d'elements, rendre uniquement les elements visibles.

```bash
npm install react-window
```

```jsx
import { FixedSizeList, VariableSizeList } from "react-window";

// ─── Liste a hauteur fixe ───
function GrandeListe({ articles }) {
    // Rendu = uniquement les elements DANS le viewport + quelques elements buffer
    // Meme si articles contient 100 000 elements !

    function RenduLigne({ index, style }) {
        const article = articles[index];
        return (
            // IMPORTANT : le style de positionnement doit etre applique
            <div style={style} className="ligne-article">
                <span>{article.titre}</span>
                <span>{article.prix} €</span>
            </div>
        );
    }

    return (
        <FixedSizeList
            height={600}          // Hauteur du conteneur visible (px)
            width="100%"          // Largeur
            itemCount={articles.length}   // Nombre total d'elements
            itemSize={56}         // Hauteur de chaque element (px)
            overscanCount={5}     // Elements pre-rendus hors viewport
        >
            {RenduLigne}
        </FixedSizeList>
    );
}

// ─── Liste a hauteur variable ───
function ListeVariableHauteur({ articles }) {
    const hauteurParIndex = (index) => {
        // Articles avec description longue = plus grands
        return articles[index].description.length > 100 ? 120 : 60;
    };

    return (
        <VariableSizeList
            height={600}
            width="100%"
            itemCount={articles.length}
            itemSize={hauteurParIndex}
        >
            {({ index, style }) => (
                <div style={style}>
                    <h3>{articles[index].titre}</h3>
                    <p>{articles[index].description}</p>
                </div>
            )}
        </VariableSizeList>
    );
}
```

### Code Splitting et analyse du bundle

```jsx
// ─── Code splitting manuel ───
import { lazy, Suspense } from "react";

// Charge le composant uniquement quand il est necessaire
const EditeurTexte = lazy(() => import("./components/EditeurTexte"));
const GraphiqueAvance = lazy(() => import("./components/GraphiqueAvance"));

function TableauDeBord() {
    const [afficherEditeur, setAfficherEditeur] = useState(false);

    return (
        <div>
            <button onClick={() => setAfficherEditeur(true)}>Ouvrir editeur</button>

            <Suspense fallback={<div>Chargement de l'editeur...</div>}>
                {afficherEditeur && <EditeurTexte />}
            </Suspense>
        </div>
    );
}
```

```bash
# ─── Analyser la taille du bundle ───
npm install --save-dev rollup-plugin-visualizer

# Dans vite.config.js
import { visualizer } from "rollup-plugin-visualizer";

export default {
    plugins: [
        react(),
        visualizer({
            open: true,           // Ouvre le rapport dans le navigateur
            gzipSize: true,       // Taille apres compression gzip
            filename: "bundle-stats.html",
        }),
    ],
};

# Apres npm run build : ouvre automatiquement bundle-stats.html
# Identifier les grosses dependances et les opportunites de code splitting
```

---

## 2. Error Boundaries

Les Error Boundaries attrapent les erreurs JavaScript dans l'arbre de composants enfants et affichent une UI de fallback au lieu de crasher toute l'application.

```jsx
import { Component } from "react";

// ─── Error Boundary : doit etre une classe (limitation React actuelle) ───
class ErrorBoundary extends Component {
    constructor(props) {
        super(props);
        this.state = {
            aUneErreur: false,
            erreur: null,
            infoErreur: null,
        };
    }

    // Declenche quand un composant enfant throw
    static getDerivedStateFromError(erreur) {
        return { aUneErreur: true, erreur };
    }

    // Informations de debug (stack trace)
    componentDidCatch(erreur, infoErreur) {
        this.setState({ infoErreur });
        // Envoyer a un service de monitoring (Sentry, Datadog...)
        console.error("ErrorBoundary a capture :", erreur, infoErreur);
        // sentryClient.captureException(erreur, { extra: infoErreur });
    }

    handleReinit() {
        this.setState({ aUneErreur: false, erreur: null, infoErreur: null });
    }

    render() {
        if (this.state.aUneErreur) {
            // Afficher le composant de fallback passe en prop
            if (this.props.fallback) {
                return this.props.fallback({
                    erreur: this.state.erreur,
                    reinitialiser: () => this.handleReinit(),
                });
            }

            return (
                <div role="alert" className="error-boundary">
                    <h2>Une erreur est survenue</h2>
                    <details>
                        <summary>Details techniques</summary>
                        <pre>{this.state.erreur?.toString()}</pre>
                        <pre>{this.state.infoErreur?.componentStack}</pre>
                    </details>
                    <button onClick={() => this.handleReinit()}>
                        Reessayer
                    </button>
                </div>
            );
        }

        return this.props.children;
    }
}

// ─── Utilisation ───
function App() {
    return (
        // ErrorBoundary global : protege toute l'app
        <ErrorBoundary
            fallback={({ erreur, reinitialiser }) => (
                <div>
                    <h1>L'application a rencontre une erreur</h1>
                    <button onClick={reinitialiser}>Recharger</button>
                </div>
            )}
        >
            <Header />

            {/* Error Boundary local : isole les widgets independants */}
            <ErrorBoundary fallback={({ reinitialiser }) => (
                <div className="widget-erreur">
                    <p>Ce graphique n'a pas pu se charger.</p>
                    <button onClick={reinitialiser}>Reessayer</button>
                </div>
            )}>
                <GraphiqueComplexe />
            </ErrorBoundary>

            <ErrorBoundary>
                <ListeUtilisateurs />
            </ErrorBoundary>
        </ErrorBoundary>
    );
}
```

> [!warning] Limitations des Error Boundaries
> Les Error Boundaries n'attrapent PAS : les erreurs dans les gestionnaires d'evenements (onClick...), les erreurs dans le code asynchrone (setTimeout, fetch), les erreurs dans les Server Components, les erreurs dans le Error Boundary lui-meme. Pour les gestionnaires d'evenements, utiliser try/catch standard.

---

## 3. Portals

Un Portal permet de rendre un composant dans un noeud DOM en dehors de la hierarchie du composant parent. Utile pour les modales, tooltips, notifications.

```jsx
import { createPortal } from "react-dom";
import { useEffect, useRef } from "react";

// ─── Modal avec Portal ───
function Modal({ estOuverte, onFermer, titre, children }) {
    const overlayRef = useRef(null);

    // Gestion du focus (accessibilite)
    useEffect(() => {
        if (estOuverte) {
            // Sauvegarder le focus actuel et le restaurer a la fermeture
            const elementPrecedent = document.activeElement;
            overlayRef.current?.focus();

            return () => elementPrecedent?.focus();
        }
    }, [estOuverte]);

    // Fermer avec Echap
    useEffect(() => {
        function handleKeyDown(e) {
            if (e.key === "Escape" && estOuverte) {
                onFermer();
            }
        }
        window.addEventListener("keydown", handleKeyDown);
        return () => window.removeEventListener("keydown", handleKeyDown);
    }, [estOuverte, onFermer]);

    // Ne rien rendre si la modale est fermee
    if (!estOuverte) return null;

    // ─── createPortal(contenu, cible) ───
    // Le composant est rendu dans document.body
    // meme s'il est defini profondement dans l'arbre React
    return createPortal(
        <div
            ref={overlayRef}
            role="dialog"
            aria-modal="true"
            aria-label={titre}
            tabIndex={-1}
            className="modal-overlay"
            onClick={(e) => {
                // Fermer si clic sur l'overlay (pas sur la modale elle-meme)
                if (e.target === e.currentTarget) onFermer();
            }}
        >
            <div className="modal-contenu">
                <header className="modal-header">
                    <h2>{titre}</h2>
                    <button
                        onClick={onFermer}
                        aria-label="Fermer la modale"
                        className="modal-fermer"
                    >
                        ×
                    </button>
                </header>
                <div className="modal-corps">{children}</div>
            </div>
        </div>,
        document.body // Cible : rendre dans le body, pas dans le parent
    );
}

// ─── Utilisation ───
function App() {
    const [modaleOuverte, setModaleOuverte] = useState(false);

    return (
        <div style={{ overflow: "hidden" }}> {/* Meme avec overflow:hidden, la modale est visible */}
            <button onClick={() => setModaleOuverte(true)}>Ouvrir la modale</button>
            <Modal
                estOuverte={modaleOuverte}
                onFermer={() => setModaleOuverte(false)}
                titre="Confirmation"
            >
                <p>Etes-vous sur de vouloir continuer ?</p>
                <button onClick={() => setModaleOuverte(false)}>Annuler</button>
                <button onClick={confirmer}>Confirmer</button>
            </Modal>
        </div>
    );
}
```

---

## 4. Patterns Avances

### Compound Components

Le pattern Compound Components permet de creer des composants flexibles qui partagent implicitement leur etat via le contexte.

```jsx
import { createContext, useContext, useState } from "react";

// ─── Exemple : Accordeon compound ───
const AccordeonContext = createContext(null);

function Accordeon({ children, unique = true }) {
    const [ouvert, setOuvert] = useState(null);

    function basculer(id) {
        setOuvert(prev => (unique ? (prev === id ? null : id) : null));
    }

    return (
        <AccordeonContext.Provider value={{ ouvert, basculer }}>
            <div className="accordeon">{children}</div>
        </AccordeonContext.Provider>
    );
}

Accordeon.Item = function Item({ id, children }) {
    return <div className="accordeon-item">{children}</div>;
};

Accordeon.Trigger = function Trigger({ id, children }) {
    const { ouvert, basculer } = useContext(AccordeonContext);
    const estOuvert = ouvert === id;
    return (
        <button
            aria-expanded={estOuvert}
            onClick={() => basculer(id)}
            className="accordeon-trigger"
        >
            {children}
            <span aria-hidden>{estOuvert ? "▲" : "▼"}</span>
        </button>
    );
};

Accordeon.Panel = function Panel({ id, children }) {
    const { ouvert } = useContext(AccordeonContext);
    if (ouvert !== id) return null;
    return <div className="accordeon-panel" role="region">{children}</div>;
};

// ─── Utilisation : API tres lisible ───
function FAQ() {
    return (
        <Accordeon>
            <Accordeon.Item id="q1">
                <Accordeon.Trigger id="q1">Comment s'inscrire ?</Accordeon.Trigger>
                <Accordeon.Panel id="q1">
                    Cliquez sur le bouton "S'inscrire" et remplissez le formulaire.
                </Accordeon.Panel>
            </Accordeon.Item>
            <Accordeon.Item id="q2">
                <Accordeon.Trigger id="q2">Comment payer ?</Accordeon.Trigger>
                <Accordeon.Panel id="q2">
                    Nous acceptons les cartes bancaires, PayPal et virements.
                </Accordeon.Panel>
            </Accordeon.Item>
        </Accordeon>
    );
}
```

### Render Props

Un composant transmet des donnees a ses enfants via une fonction.

```jsx
// ─── MouseTracker avec render prop ───
function SuiviSouris({ render }) {
    const [position, setPosition] = useState({ x: 0, y: 0 });

    function handleMouseMove(e) {
        setPosition({ x: e.clientX, y: e.clientY });
    }

    return (
        <div style={{ height: "100vh" }} onMouseMove={handleMouseMove}>
            {/* Passe la position via render prop */}
            {render(position)}
        </div>
    );
}

// ─── Utilisation ───
function App() {
    return (
        <SuiviSouris
            render={({ x, y }) => (
                <div className="curseur-info">
                    Position : ({x}, {y})
                </div>
            )}
        />
    );
}
```

### HOC — Higher Order Component

Un HOC est une fonction qui prend un composant et retourne un composant enrichi.

```jsx
// ─── withAuth : HOC qui ajoute la verification d'authentification ───
function withAuth(ComposantProtege, roleRequis = null) {
    function ComposantAvecAuth(props) {
        const { utilisateur, chargement } = useAuth();

        if (chargement) return <div>Chargement...</div>;

        if (!utilisateur) {
            return <Navigate to="/connexion" replace />;
        }

        if (roleRequis && !utilisateur.roles.includes(roleRequis)) {
            return <div>Acces refuse.</div>;
        }

        return <ComposantProtege {...props} />;
    }

    // Conserver le nom pour React DevTools
    ComposantAvecAuth.displayName = `withAuth(${ComposantProtege.displayName || ComposantProtege.name})`;

    return ComposantAvecAuth;
}

// ─── withLogger : HOC de logging ───
function withLogger(Composant) {
    function ComposantAvecLog(props) {
        useEffect(() => {
            console.log(`${Composant.name} monte`, props);
            return () => console.log(`${Composant.name} demonte`);
        }, []);

        return <Composant {...props} />;
    }
    ComposantAvecLog.displayName = `withLogger(${Composant.name})`;
    return ComposantAvecLog;
}

// ─── Utilisation ───
const DashboardProtege = withAuth(Dashboard);
const AdminProtege = withAuth(PanneauAdmin, "admin");
const DashboardTrace = withLogger(DashboardProtege);
```

> [!info] HOC vs Hooks vs Render Props
> Les HOC et Render Props etaient les solutions pre-hooks pour partager de la logique. Aujourd'hui, les custom hooks sont la solution preferee : plus simples, plus composables, pas de probleme de "wrapper hell". Connaitre les HOC reste utile pour comprendre les bibliotheques legacy et les patterns existants.

---

## 5. Zustand — Gestion d'Etat Legere

Zustand est une bibliotheque de gestion d'etat minimaliste. Pas de boilerplate, pas de Provider, pas d'actions/reducers.

```bash
npm install zustand
```

```jsx
// ─── src/stores/panierStore.js ───
import { create } from "zustand";
import { persist, devtools } from "zustand/middleware";

// ─── create() : definit le store ───
const utiliserPanierStore = create(
    // devtools : integration avec Redux DevTools
    devtools(
        // persist : sauvegarde automatiquement dans localStorage
        persist(
            (set, get) => ({
                // ─── State ───
                items: [],
                codePromo: null,

                // ─── Actions ───
                ajouter(produit, quantite = 1) {
                    set(state => {
                        const existant = state.items.find(i => i.id === produit.id);
                        if (existant) {
                            return {
                                items: state.items.map(i =>
                                    i.id === produit.id
                                        ? { ...i, quantite: i.quantite + quantite }
                                        : i
                                ),
                            };
                        }
                        return { items: [...state.items, { ...produit, quantite }] };
                    });
                },

                retirer(id) {
                    set(state => ({
                        items: state.items.filter(i => i.id !== id),
                    }));
                },

                vider() {
                    set({ items: [], codePromo: null });
                },

                appliquerPromo(code) {
                    set({ codePromo: code });
                },

                // ─── Valeurs derivees (getters) ───
                get totalItems() {
                    return get().items.reduce((sum, i) => sum + i.quantite, 0);
                },

                get sousTotal() {
                    return get().items.reduce((sum, i) => sum + i.prix * i.quantite, 0);
                },

                get total() {
                    const sousTotal = get().sousTotal;
                    const promo = get().codePromo === "PROMO10" ? 0.1 : 0;
                    return sousTotal * (1 - promo);
                },
            }),
            { name: "panier-storage" } // Cle localStorage
        ),
        { name: "PanierStore" } // Nom dans Redux DevTools
    )
);

export default utiliserPanierStore;
```

```jsx
// ─── Utilisation dans les composants ───
import utiliserPanierStore from "../stores/panierStore";

// N'importe quel composant peut acceder au store sans Provider
function BoutonAjouterPanier({ produit }) {
    // Souscrire UNIQUEMENT a l'action (pas de re-render inutile)
    const ajouter = utiliserPanierStore(state => state.ajouter);

    return <button onClick={() => ajouter(produit)}>Ajouter</button>;
}

function IconePanier() {
    // Ne se re-rend que quand totalItems change
    const totalItems = utiliserPanierStore(state => state.totalItems);

    return (
        <div>
            🛒
            {totalItems > 0 && <span className="badge">{totalItems}</span>}
        </div>
    );
}

function RecapPanier() {
    const { items, retirer, vider, sousTotal, total, appliquerPromo } =
        utiliserPanierStore();

    return (
        <div>
            {items.map(item => (
                <div key={item.id}>
                    {item.nom} x {item.quantite} = {(item.prix * item.quantite).toFixed(2)} €
                    <button onClick={() => retirer(item.id)}>×</button>
                </div>
            ))}
            <p>Sous-total : {sousTotal.toFixed(2)} €</p>
            <p>Total : {total.toFixed(2)} €</p>
            <button onClick={vider}>Vider le panier</button>
        </div>
    );
}
```

---

## 6. TanStack Query (React Query)

TanStack Query gere le fetching, le caching, la synchronisation et la mise a jour des donnees serveur. Il remplace les patterns fetch + useEffect + useState.

```bash
npm install @tanstack/react-query
```

```jsx
// ─── src/main.jsx : configuration ───
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

const queryClient = new QueryClient({
    defaultOptions: {
        queries: {
            staleTime: 5 * 60 * 1000,  // Donnees "fraiches" pendant 5 min
            gcTime: 10 * 60 * 1000,    // Garder en cache 10 min
            retry: 2,                  // Reessayer 2 fois en cas d'echec
            refetchOnWindowFocus: true, // Recharger quand l'onglet reprend le focus
        },
    },
});

createRoot(document.getElementById("root")).render(
    <QueryClientProvider client={queryClient}>
        <App />
        {/* DevTools : onglet React Query dans les outils du navigateur */}
        <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
);
```

### useQuery — Fetching et caching

```jsx
import { useQuery, useInfiniteQuery } from "@tanstack/react-query";

// ─── Fonction de fetching (separee du composant) ───
async function recupererArticles(page = 1, filtre = "") {
    const params = new URLSearchParams({ page, limit: 10, filtre });
    const reponse = await fetch(`/api/articles?${params}`);
    if (!reponse.ok) throw new Error(`HTTP ${reponse.status}`);
    return reponse.json();
}

async function recupererArticle(id) {
    const reponse = await fetch(`/api/articles/${id}`);
    if (!reponse.ok) throw new Error(`Article non trouve`);
    return reponse.json();
}

// ─── Composant avec useQuery ───
function ListeArticles() {
    const [page, setPage] = useState(1);
    const [filtre, setFiltre] = useState("");

    const {
        data,         // Les donnees retournees par queryFn
        isLoading,    // true lors du premier chargement (pas de cache)
        isFetching,   // true pendant n'importe quel chargement (rechargement inclus)
        isError,      // true si queryFn a throw
        error,        // L'erreur lancee par queryFn
        refetch,      // Recharger manuellement
    } = useQuery({
        // queryKey : identifiant unique de la requete (et du cache)
        // Si une cle change → nouvelle requete
        queryKey: ["articles", { page, filtre }],
        queryFn: () => recupererArticles(page, filtre),
        placeholderData: previousData => previousData, // Garder les anciennes donnees pendant le rechargement
    });

    if (isLoading) return <p>Chargement initial...</p>;
    if (isError) return <p>Erreur : {error.message} <button onClick={refetch}>Reessayer</button></p>;

    return (
        <div>
            {/* isFetching = vrai meme si les donnees sont en cache */}
            {isFetching && <span>Mise a jour...</span>}

            <input value={filtre} onChange={e => setFiltre(e.target.value)} placeholder="Filtrer" />

            <ul>
                {data.articles.map(a => (
                    <li key={a.id}>{a.titre}</li>
                ))}
            </ul>

            <button onClick={() => setPage(p => p - 1)} disabled={page === 1}>Precedent</button>
            <span> Page {page} / {data.totalPages} </span>
            <button onClick={() => setPage(p => p + 1)} disabled={page >= data.totalPages}>Suivant</button>
        </div>
    );
}

// ─── Requetes dependantes ───
function PageDetailArticle({ articleId }) {
    // Charger l'article
    const { data: article, isLoading } = useQuery({
        queryKey: ["article", articleId],
        queryFn: () => recupererArticle(articleId),
    });

    // Charger les commentaires SEULEMENT si l'article est charge
    const { data: commentaires } = useQuery({
        queryKey: ["commentaires", articleId],
        queryFn: () => recupererCommentaires(articleId),
        enabled: !!article, // N'executer que si article est defini
    });

    if (isLoading) return <p>Chargement...</p>;
    return (
        <div>
            <h1>{article.titre}</h1>
            <p>{article.contenu}</p>
            {commentaires && (
                <section>
                    <h2>Commentaires</h2>
                    {commentaires.map(c => <p key={c.id}>{c.texte}</p>)}
                </section>
            )}
        </div>
    );
}
```

### useMutation — Modifications et invalidation

```jsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

async function creerCommentaire(donnees) {
    const reponse = await fetch("/api/commentaires", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(donnees),
    });
    if (!reponse.ok) throw new Error("Erreur creation commentaire");
    return reponse.json();
}

function FormulaireCommentaire({ articleId }) {
    const queryClient = useQueryClient();
    const [texte, setTexte] = useState("");

    const mutation = useMutation({
        mutationFn: creerCommentaire,

        // Mise a jour optimiste : modifier le cache AVANT la reponse serveur
        onMutate: async (nouvelleDonnee) => {
            // Annuler les rechargements en cours pour eviter les conflits
            await queryClient.cancelQueries({ queryKey: ["commentaires", articleId] });

            // Snapshot de l'ancien etat (pour rollback)
            const ancienCache = queryClient.getQueryData(["commentaires", articleId]);

            // Mettre a jour le cache immediatement (optimiste)
            queryClient.setQueryData(["commentaires", articleId], old => [
                ...old,
                { id: Date.now(), texte: nouvelleDonnee.texte, auteur: "Moi" },
            ]);

            return { ancienCache }; // Contexte pour onError
        },

        // Si la mutation echoue : restaurer l'ancien cache
        onError: (err, nouvelleDonnee, context) => {
            queryClient.setQueryData(["commentaires", articleId], context.ancienCache);
            alert(`Erreur : ${err.message}`);
        },

        // Dans tous les cas : invalider pour recharger les vraies donnees
        onSettled: () => {
            queryClient.invalidateQueries({ queryKey: ["commentaires", articleId] });
        },

        onSuccess: () => {
            setTexte(""); // Vider le champ apres succes
        },
    });

    return (
        <form onSubmit={(e) => {
            e.preventDefault();
            if (!texte.trim()) return;
            mutation.mutate({ articleId, texte });
        }}>
            <textarea
                value={texte}
                onChange={e => setTexte(e.target.value)}
                placeholder="Ecrire un commentaire..."
                disabled={mutation.isPending}
            />
            <button type="submit" disabled={mutation.isPending}>
                {mutation.isPending ? "Envoi..." : "Commenter"}
            </button>
            {mutation.isError && <p>Erreur : {mutation.error.message}</p>}
        </form>
    );
}
```

---

## 7. Tests avec Vitest et React Testing Library

### Installation et configuration

```bash
npm install --save-dev vitest @testing-library/react @testing-library/user-event @testing-library/jest-dom jsdom
```

```javascript
// ─── vite.config.js ───
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
    plugins: [react()],
    test: {
        environment: "jsdom",       // Simuler un navigateur
        globals: true,              // vi, describe, it, expect disponibles globalement
        setupFiles: "./src/test-setup.js",
    },
});
```

```javascript
// ─── src/test-setup.js ───
import "@testing-library/jest-dom"; // Matchers supplementaires : toBeInTheDocument, etc.
```

### Anatomie d'un test React Testing Library

```jsx
// ─── src/components/__tests__/Bouton.test.jsx ───
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { vi } from "vitest";
import Bouton from "../Bouton";

// describe : groupe logique de tests
describe("Bouton", () => {
    it("affiche le texte passe en prop", () => {
        // 1. RENDER : monter le composant dans un DOM simule
        render(<Bouton>Cliquer</Bouton>);

        // 2. QUERY : trouver l'element dans le DOM
        // screen.getBy* : throw si non trouve (test echoue)
        // screen.queryBy* : retourne null si non trouve
        // screen.findBy* : attend que l'element apparaisse (async)
        const bouton = screen.getByRole("button", { name: "Cliquer" });

        // 3. ASSERTION : verifier l'etat
        expect(bouton).toBeInTheDocument();
    });

    it("appelle onClick quand clique", async () => {
        // userEvent simule de vraies interactions utilisateur
        const user = userEvent.setup();
        const mockOnClick = vi.fn(); // Fonction mock

        render(<Bouton onClick={mockOnClick}>Valider</Bouton>);

        const bouton = screen.getByRole("button");
        await user.click(bouton);

        expect(mockOnClick).toHaveBeenCalledTimes(1);
    });

    it("est desactive quand desactive=true", () => {
        render(<Bouton desactive>Soumettre</Bouton>);

        expect(screen.getByRole("button")).toBeDisabled();
    });

    it("n'appelle pas onClick quand desactive", async () => {
        const user = userEvent.setup();
        const mockOnClick = vi.fn();

        render(<Bouton desactive onClick={mockOnClick}>Test</Bouton>);
        await user.click(screen.getByRole("button"));

        expect(mockOnClick).not.toHaveBeenCalled();
    });
});
```

### Requetes screen — choisir la bonne

```jsx
// Hierarchie de preference des requetes (du plus accessible au moins)

// 1. getByRole : le plus accessible (cherche par role ARIA)
screen.getByRole("button", { name: "Valider" });
screen.getByRole("textbox", { name: "Email" });
screen.getByRole("heading", { level: 1 });
screen.getByRole("checkbox");
screen.getByRole("combobox"); // select

// 2. getByLabelText : pour les inputs avec label associe
screen.getByLabelText("Adresse email");

// 3. getByPlaceholderText
screen.getByPlaceholderText("Entrer votre email...");

// 4. getByText : pour les elements non-interactifs
screen.getByText("Bienvenue sur React");

// 5. getByTestId : dernier recours (ajouter data-testid sur l'element)
screen.getByTestId("liste-resultats");

// Variantes :
// getBy*   = throw si non trouve ou si plusieurs
// queryBy* = retourne null si non trouve (pas de throw)
// findBy*  = async, attend que l'element apparaisse (retourne Promise)
// getAllBy* = retourne un tableau (throw si aucun)
```

### Tester des composants avec etat et API

```jsx
// ─── src/components/__tests__/ListeArticles.test.jsx ───
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { vi } from "vitest";
import ListeArticles from "../ListeArticles";

// ─── Mocker fetch ───
const articlesMock = [
    { id: 1, titre: "Introduction a React", auteur: "Alice" },
    { id: 2, titre: "Les hooks en detail", auteur: "Bob" },
];

beforeEach(() => {
    // Remplacer fetch par un mock pour les tests
    global.fetch = vi.fn().mockResolvedValue({
        ok: true,
        json: () => Promise.resolve(articlesMock),
    });
});

afterEach(() => {
    vi.restoreAllMocks();
});

describe("ListeArticles", () => {
    it("affiche un spinner pendant le chargement", () => {
        render(<ListeArticles />);
        expect(screen.getByText(/chargement/i)).toBeInTheDocument();
    });

    it("affiche les articles apres chargement", async () => {
        render(<ListeArticles />);

        // Attendre que les articles apparaissent
        await waitFor(() => {
            expect(screen.getByText("Introduction a React")).toBeInTheDocument();
            expect(screen.getByText("Les hooks en detail")).toBeInTheDocument();
        });
    });

    it("affiche un message d'erreur si le fetch echoue", async () => {
        global.fetch = vi.fn().mockRejectedValue(new Error("Reseau indisponible"));

        render(<ListeArticles />);

        await waitFor(() => {
            expect(screen.getByText(/erreur/i)).toBeInTheDocument();
        });
    });

    it("filtre les articles en temps reel", async () => {
        const user = userEvent.setup();
        render(<ListeArticles />);

        // Attendre le chargement
        await waitFor(() => screen.getByText("Introduction a React"));

        // Taper dans le champ de recherche
        const recherche = screen.getByRole("textbox", { name: /rechercher/i });
        await user.type(recherche, "hooks");

        // "hooks" doit rester visible, "Introduction" doit disparaitre
        expect(screen.getByText("Les hooks en detail")).toBeInTheDocument();
        expect(screen.queryByText("Introduction a React")).not.toBeInTheDocument();
    });
});
```

### Tests avec React Router et Context

```jsx
// ─── Utilitaire de test : renderAvecContexte ───
// src/test-utils.jsx
import { render } from "@testing-library/react";
import { MemoryRouter } from "react-router-dom";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ThemeProvider } from "./contexts/ThemeContext";
import { AuthProvider } from "./contexts/AuthContext";

// Creer un QueryClient neuf pour chaque test (evite le partage de cache)
function creerTestQueryClient() {
    return new QueryClient({
        defaultOptions: {
            queries: { retry: false }, // Ne pas reessayer les requetes en echec dans les tests
        },
    });
}

export function renderAvecContexte(ui, {
    route = "/",
    utilisateurMock = null,
} = {}) {
    const queryClient = creerTestQueryClient();

    return render(
        <QueryClientProvider client={queryClient}>
            <MemoryRouter initialEntries={[route]}>
                <ThemeProvider>
                    {/* Si utilisateurMock est fourni, simuler un utilisateur connecte */}
                    {ui}
                </ThemeProvider>
            </MemoryRouter>
        </QueryClientProvider>
    );
}
```

```jsx
// ─── Test d'un composant avec Router ───
import { renderAvecContexte } from "../test-utils";

it("redirige vers /connexion si non connecte", async () => {
    renderAvecContexte(<PageProtegee />, { route: "/dashboard" });

    await waitFor(() => {
        // Verifier que l'utilisateur a ete redirige
        expect(screen.getByRole("heading", { name: /connexion/i })).toBeInTheDocument();
    });
});
```

---

## 8. Accessibilite (a11y)

### Principes de base

```jsx
// ─── Bonnes pratiques d'accessibilite ───
function FormulaireBienAccessible() {
    return (
        <form aria-label="Formulaire de contact" noValidate>
            {/* Labels explicitement associes aux inputs */}
            <div>
                <label htmlFor="nom">Nom *</label>
                <input
                    id="nom"
                    name="nom"
                    type="text"
                    aria-required="true"
                    aria-describedby="nom-aide nom-erreur"
                    autoComplete="name"
                />
                <p id="nom-aide">Entrer votre nom complet.</p>
                {/* Les erreurs sont annoncees aux lecteurs d'ecran */}
                <p id="nom-erreur" role="alert" aria-live="polite">
                    {/* Erreur ici si invalide */}
                </p>
            </div>

            {/* Bouton semantique avec label descriptif */}
            <button type="submit" aria-busy={false}>
                Envoyer le formulaire
            </button>
        </form>
    );
}

// ─── Navigation au clavier ───
function MenuNavigation() {
    return (
        <nav aria-label="Navigation principale">
            <ul role="menubar">
                <li role="none">
                    <a role="menuitem" href="/">Accueil</a>
                </li>
                <li role="none">
                    <a role="menuitem" href="/profil">Profil</a>
                </li>
            </ul>
        </nav>
    );
}
```

### Tests d'accessibilite avec jest-axe

```bash
npm install --save-dev jest-axe
```

```jsx
import { axe, toHaveNoViolations } from "jest-axe";
expect.extend(toHaveNoViolations);

it("est accessible", async () => {
    const { container } = render(<MonFormulaire />);
    const resultats = await axe(container);
    expect(resultats).toHaveNoViolations();
});
```

---

## 9. Deploiement

### Build de production

```bash
# ─── Build Vite ───
npm run build
# Genere dist/
# ├── index.html
# ├── assets/
# │   ├── index-abc123.js    (bundle JS minifie)
# │   └── index-def456.css   (CSS minifie)

# ─── Analyser le bundle ───
npm run build -- --report

# ─── Previsualiser localement ───
npm run preview
```

### Deploiement sur Vercel

```bash
# ─── Via CLI ───
npm install -g vercel
vercel         # Premier deploiement (configuration interactive)
vercel --prod  # Deploiement de production

# ─── Configuration vercel.json (pour React Router avec History API) ───
# Sans cette config, les rechargements sur /about retournent 404
```

```json
{
    "rewrites": [
        { "source": "/(.*)", "destination": "/index.html" }
    ]
}
```

### Deploiement sur Netlify

```bash
# ─── Via CLI ───
npm install -g netlify-cli
netlify deploy --dir=dist        # Apercu
netlify deploy --dir=dist --prod # Production

# ─── Fichier public/_redirects (pour React Router) ───
# /*   /index.html   200
```

### Variables d'environnement avec Vite

```bash
# ─── .env (commite) ───
VITE_NOM_APP="Mon Application"

# ─── .env.local (ne pas commiter) ───
VITE_API_URL="http://localhost:3000"
VITE_STRIPE_PUBLIC_KEY="pk_test_xxx"

# ─── .env.production (commite, valeurs de prod) ───
VITE_API_URL="https://api.monapp.com"
```

```jsx
// Acces dans le code (prefixe VITE_ obligatoire pour exposer)
const apiUrl = import.meta.env.VITE_API_URL;
const nomApp = import.meta.env.VITE_NOM_APP;
const estDev = import.meta.env.DEV;      // true en developpement
const estProd = import.meta.env.PROD;    // true en production
```

---

## Carte Mentale ASCII

```
              REACT AVANCE + TESTS + DEPLOIEMENT
                              │
        ┌─────────────────────┼───────────────────────┐
        │                     │                       │
   PERFORMANCE            PATTERNS             TESTS
        │                 AVANCES                    │
   React.memo         Compound Comp.        Vitest + RTL
   useCallback        Render Props          screen queries
   useMemo            HOC                   userEvent
   Virtualisation     Error Boundary        waitFor
   Code Splitting     Portals               vi.fn() mocks
   Bundle analysis        │                axe (a11y)
        │                 │                    │
   ZUSTAND          REACT QUERY          DEPLOIEMENT
        │                 │                    │
   create()          useQuery              Vercel
   get/set           useMutation           Netlify
   persist           QueryClient           Build Vite
   devtools          invalidateQueries     _redirects
   selectors         optimistic updates    env vars
   middleware         staleTime/gcTime     VITE_ prefix
```

---

## Exercices

### Exercice 1 : Optimisation d'une liste de 10 000 elements

Creez une application avec :
- Une liste de 10 000 contacts generee aleatoirement (nom, email, departement, salaire)
- Sans optimisation : mesurer le temps de rendu avec le Profiler
- Implementer la virtualisation avec react-window
- Ajouter un champ de recherche (debounce 300ms avec useMemo)
- Tri par colonne (useMemo sur la liste triee)
- Comparer les performances avant/apres et documenter les gains

### Exercice 2 : Error Boundary avec monitoring

Implementez un systeme d'Error Boundary complet :
- Error Boundary global qui capture toutes les erreurs non gerees
- Error Boundary local autour de chaque "widget" independant
- Page de fallback avec : message utilisateur, bouton "Reessayer", bouton "Recharger la page"
- Logging console avec la stack trace complete
- Composant de test qui throw intentionnellement (bouton "Provoquer une erreur")
- Verification que les autres widgets continuent de fonctionner quand l'un crashe

### Exercice 3 : Suite de tests complete

Ecrivez des tests pour un composant `ShoppingCart` :
- Teste que le panier vide affiche "Votre panier est vide"
- Teste qu'ajouter un produit l'affiche dans la liste
- Teste que cliquer `×` retire le produit
- Teste que le total se calcule correctement (incluant les quantites)
- Teste que le bouton "Commander" declenche une fonction callback
- Teste que la validation empeche de commander si le panier est vide
- Atteindre 100% de couverture des branches du composant
- Les tests ne doivent pas tester l'implementation, seulement le comportement observable

### Exercice 4 : Application complete avec React Query + Zustand

Creez une application de gestion de taches collaborative :
- Liste de taches depuis une API (jsonplaceholder) avec React Query (pagination infinie)
- Filtre par statut et recherche (params URL persistants avec useSearchParams)
- Ajouter/modifier/supprimer avec mutations optimistes
- Theme (clair/sombre) et preferences utilisateur avec Zustand + persist
- Error Boundary autour de la liste
- Au moins 10 tests couvrant les cas d'usage principaux
- Deployer sur Vercel ou Netlify avec configuration des redirections

---

## Liens

- [[01 - Introduction a React]]
- [[02 - Hooks et Gestion d'Etat]]
- [[03 - React Router et Formulaires]]
- [[01 - Tests Unitaires et TDD]]
- [[02 - Tests Integration et E2E]]
- [[05 - SPA et Frameworks Introduction]]
