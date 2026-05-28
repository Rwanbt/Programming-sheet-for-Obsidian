# React Router et Formulaires

Une application React sans routage est une page unique statique. React Router v6 apporte la navigation multi-vues dans une SPA, avec une API declarative, des routes imbriquees et des hooks puissants. Les formulaires, eux, sont le point d'entree des donnees utilisateur : bien les gerer (validation, erreurs, etats de chargement) est une competence fondamentale pour tout developpeur frontend.

Ce cours couvre React Router v6 de A a Z, puis les deux approches de formulaires (non-controles et controles), React Hook Form, la validation avec Zod, et les patterns de formulaires courants.

> [!tip] Analogie
> React Router est comme le systeme de signalisation d'un batiment : il definit quelle salle (composant) afficher selon le couloir (URL) empruntee. Le formulaire, c'est le bureau d'accueil : il collecte les informations, les verifie avant de les accepter, et informe l'utilisateur si quelque chose manque.

---

## 1. React Router v6 — Introduction

### Installation et configuration de base

```bash
npm install react-router-dom
```

```jsx
// ─── src/main.jsx : envelopper l'app dans BrowserRouter ───
import { BrowserRouter } from "react-router-dom";
import App from "./App";

createRoot(document.getElementById("root")).render(
    <BrowserRouter>
        <App />
    </BrowserRouter>
);
```

### Routes et navigation de base

```jsx
// ─── src/App.jsx ───
import { Routes, Route, Link, NavLink } from "react-router-dom";
import Accueil from "./pages/Accueil";
import Profil from "./pages/Profil";
import PasDeTrouve from "./pages/PasDeTrouve";

function App() {
    return (
        <div>
            {/* ─── Navigation ─── */}
            <nav>
                {/* Link : navigation sans rechargement de page */}
                <Link to="/">Accueil</Link>

                {/* NavLink : ajoute automatiquement la classe "active" sur la route active */}
                <NavLink
                    to="/profil"
                    style={({ isActive }) => ({
                        fontWeight: isActive ? "bold" : "normal",
                        color: isActive ? "blue" : "inherit",
                    })}
                >
                    Profil
                </NavLink>
            </nav>

            {/* ─── Definition des routes ─── */}
            <main>
                <Routes>
                    {/* Route exacte (v6 est exact par defaut) */}
                    <Route path="/" element={<Accueil />} />

                    {/* Route avec parametre dynamique */}
                    <Route path="/utilisateurs/:id" element={<DetailUtilisateur />} />

                    {/* Route avec parametre optionnel */}
                    <Route path="/articles/:categorie?" element={<ListeArticles />} />

                    {/* Route catch-all : doit etre la derniere */}
                    <Route path="*" element={<PasDeTrouve />} />
                </Routes>
            </main>
        </div>
    );
}
```

### Hooks de navigation

```jsx
import { useNavigate, useParams, useLocation, useSearchParams } from "react-router-dom";

// ─── useNavigate : navigation programmatique ───
function FormulaireConnexion() {
    const navigate = useNavigate();

    async function handleSoumettre(data) {
        await connecter(data);
        navigate("/tableau-de-bord");        // Aller a une route
        // navigate(-1);                     // Retour a la page precedente
        // navigate("/profil", { replace: true }); // Remplacer l'entree historique
    }
    // ...
}

// ─── useParams : lire les parametres de la route ───
function DetailUtilisateur() {
    const { id } = useParams(); // Recupere :id de la route /utilisateurs/:id
    const { categorie } = useParams(); // Recupere :categorie si present

    return <p>Utilisateur ID : {id}</p>;
}

// ─── useLocation : informations sur l'URL actuelle ───
function QuelquePart() {
    const location = useLocation();
    // location.pathname : "/utilisateurs/42"
    // location.search   : "?page=2&tri=nom"
    // location.hash     : "#section-2"
    // location.state    : donnees passees via navigate()

    return <p>Chemin actuel : {location.pathname}</p>;
}

// ─── useSearchParams : lire et modifier les query params ───
function ListeAvecFiltre() {
    const [searchParams, setSearchParams] = useSearchParams();
    const page = Number(searchParams.get("page") || "1");
    const tri = searchParams.get("tri") || "date";

    function changerPage(nouvellePage) {
        setSearchParams({ page: nouvellePage, tri });
    }

    return (
        <div>
            <p>Page : {page}, Tri : {tri}</p>
            <button onClick={() => changerPage(page + 1)}>Page suivante</button>
            <select
                value={tri}
                onChange={e => setSearchParams({ page, tri: e.target.value })}
            >
                <option value="date">Par date</option>
                <option value="nom">Par nom</option>
            </select>
        </div>
    );
}
```

---

## 2. Routes Imbriquees

Les routes imbriquees permettent de composer des layouts partagees entre plusieurs pages.

```jsx
// ─── Architecture des routes imbriquees ───
//
//  /                    → Layout + Accueil
//  /tableau-de-bord     → Layout + DashboardLayout + Vue generale
//  /tableau-de-bord/stats → Layout + DashboardLayout + Statistiques
//  /tableau-de-bord/users → Layout + DashboardLayout + Utilisateurs
//  /profil              → Layout + Profil

import { Routes, Route, Outlet } from "react-router-dom";

// ─── Layout principal : present sur toutes les pages ───
function Layout() {
    return (
        <div>
            <Header />
            <main>
                <Outlet /> {/* ICI seront rendues les routes enfants */}
            </main>
            <Footer />
        </div>
    );
}

// ─── Layout du dashboard : menu lateral specifique ───
function DashboardLayout() {
    return (
        <div className="dashboard">
            <aside>
                <NavLink to="/tableau-de-bord">Vue generale</NavLink>
                <NavLink to="/tableau-de-bord/stats">Statistiques</NavLink>
                <NavLink to="/tableau-de-bord/users">Utilisateurs</NavLink>
            </aside>
            <section>
                <Outlet /> {/* Routes enfants du dashboard */}
            </section>
        </div>
    );
}

// ─── Configuration complete ───
function App() {
    return (
        <Routes>
            {/* Layout principal engloble tout */}
            <Route path="/" element={<Layout />}>
                {/* index = route par defaut quand on est exactement sur "/" */}
                <Route index element={<Accueil />} />
                <Route path="profil" element={<Profil />} />

                {/* Routes imbriquees : dashboard avec son propre layout */}
                <Route path="tableau-de-bord" element={<DashboardLayout />}>
                    <Route index element={<VueGenerale />} />
                    <Route path="stats" element={<Statistiques />} />
                    <Route path="users" element={<Utilisateurs />} />
                    <Route path="users/:id" element={<DetailUtilisateur />} />
                </Route>

                {/* Catch-all */}
                <Route path="*" element={<PasDeTrouve />} />
            </Route>
        </Routes>
    );
}
```

---

## 3. Routes Protegees (PrivateRoute)

```jsx
// ─── Pattern PrivateRoute : rediriger si non connecte ───
import { Navigate, useLocation } from "react-router-dom";
import { useAuth } from "../contexts/AuthContext";

// Composant garde : redirige si l'utilisateur n'est pas authentifie
function RequiertAuthentification({ children }) {
    const { utilisateur, chargement } = useAuth();
    const location = useLocation();

    // Pendant la verification de session (useEffect initial)
    if (chargement) {
        return <div>Verification de la session...</div>;
    }

    if (!utilisateur) {
        // Rediriger vers /connexion en sauvegardant la destination d'origine
        // L'utilisateur sera redirige ici apres connexion
        return <Navigate to="/connexion" state={{ depuis: location }} replace />;
    }

    return children;
}

// ─── Version avec roles ───
function RequiertRole({ role, children }) {
    const { utilisateur } = useAuth();

    if (!utilisateur) {
        return <Navigate to="/connexion" replace />;
    }

    if (!utilisateur.roles.includes(role)) {
        return <Navigate to="/acces-refuse" replace />;
    }

    return children;
}

// ─── Utilisation dans les routes ───
function App() {
    return (
        <Routes>
            <Route path="/" element={<Layout />}>
                <Route index element={<Accueil />} />
                <Route path="connexion" element={<Connexion />} />

                {/* Routes protegees */}
                <Route
                    path="profil"
                    element={
                        <RequiertAuthentification>
                            <Profil />
                        </RequiertAuthentification>
                    }
                />

                {/* Route reservee aux admins */}
                <Route
                    path="admin"
                    element={
                        <RequiertRole role="admin">
                            <PanneauAdmin />
                        </RequiertRole>
                    }
                />
            </Route>
        </Routes>
    );
}
```

```jsx
// ─── Rediriger vers la destination d'origine apres connexion ───
function Connexion() {
    const { connecter } = useAuth();
    const navigate = useNavigate();
    const location = useLocation();

    // Recuperer la destination sauvegardee par RequiertAuthentification
    const destination = location.state?.depuis?.pathname || "/tableau-de-bord";

    async function handleConnexion(credentials) {
        await connecter(credentials.email, credentials.motDePasse);
        navigate(destination, { replace: true }); // Remplace /connexion dans l'historique
    }
    // ...
}
```

---

## 4. Lazy Loading avec Suspense

Le code splitting charge les pages uniquement quand elles sont necessaires, reduisant le bundle initial.

```jsx
import { lazy, Suspense } from "react";
import { Routes, Route } from "react-router-dom";

// ─── Importation paresseuse (chargement a la demande) ───
// Le code de chaque page n'est telecharge que quand l'utilisateur y navigue
const Accueil = lazy(() => import("./pages/Accueil"));
const Profil = lazy(() => import("./pages/Profil"));
const Dashboard = lazy(() => import("./pages/Dashboard"));
const Admin = lazy(() => import("./pages/Admin"));

// ─── Composant de fallback pendant le chargement ───
function ChargementPage() {
    return (
        <div style={{ display: "flex", justifyContent: "center", padding: "2rem" }}>
            <div className="spinner">Chargement de la page...</div>
        </div>
    );
}

function App() {
    return (
        // Suspense : affiche le fallback pendant que le composant lazy se charge
        <Suspense fallback={<ChargementPage />}>
            <Routes>
                <Route path="/" element={<Accueil />} />
                <Route path="/profil" element={<Profil />} />
                <Route path="/dashboard/*" element={<Dashboard />} />
                <Route path="/admin/*" element={<Admin />} />
            </Routes>
        </Suspense>
    );
}
```

```
Impact sur les performances :

Sans lazy loading :
  bundle.js = 1.2 MB (toutes les pages)
  Chargement initial : ~4s sur 3G

Avec lazy loading :
  bundle.js = 200 KB (code commun + page d'accueil)
  accueil.chunk.js    = 50 KB (charge au premier acces)
  dashboard.chunk.js  = 400 KB (charge seulement si /dashboard)
  admin.chunk.js      = 200 KB (charge seulement si /admin)

  Chargement initial : ~0.7s sur 3G
```

> [!tip] Pre-chargement predictif
> On peut pre-charger un composant avant que l'utilisateur ne navigue dessus. Par exemple, quand la souris survole un lien, on peut commencer le chargement : `<Link onMouseEnter={() => import("./pages/Profil")}>`.

---

## 5. Formulaires Non-Controles vs Controles

### Formulaires non-controles

```jsx
import { useRef } from "react";

function FormulaireNonControle() {
    // ─── L'etat du formulaire vit dans le DOM, pas dans React ───
    const nomRef = useRef();
    const emailRef = useRef();
    const messageRef = useRef();

    function handleSoumettre(e) {
        e.preventDefault();
        // Lire les valeurs depuis le DOM au moment de la soumission
        const donnees = {
            nom: nomRef.current.value,
            email: emailRef.current.value,
            message: messageRef.current.value,
        };
        console.log("Donnees :", donnees);

        // Reinitialiser manuellement
        e.target.reset();
    }

    return (
        <form onSubmit={handleSoumettre}>
            <input ref={nomRef} type="text" name="nom" placeholder="Nom" />
            <input ref={emailRef} type="email" name="email" placeholder="Email" />
            <textarea ref={messageRef} name="message" />
            <button type="submit">Envoyer</button>
        </form>
    );
}
```

### Formulaires controles

```jsx
import { useState } from "react";

function FormulaireControle() {
    // ─── L'etat du formulaire vit dans React (useState) ───
    const [valeurs, setValeurs] = useState({
        nom: "",
        email: "",
        age: "",
        genre: "non-precise",
        newsletter: false,
        message: "",
    });

    // Handler generique pour tous les champs
    function handleChanger(e) {
        const { name, value, type, checked } = e.target;
        setValeurs(prev => ({
            ...prev,
            [name]: type === "checkbox" ? checked : value,
        }));
    }

    function handleSoumettre(e) {
        e.preventDefault();
        console.log("Donnees :", valeurs);
    }

    return (
        <form onSubmit={handleSoumettre}>
            {/* Text input */}
            <input name="nom" value={valeurs.nom} onChange={handleChanger} />

            {/* Select */}
            <select name="genre" value={valeurs.genre} onChange={handleChanger}>
                <option value="femme">Femme</option>
                <option value="homme">Homme</option>
                <option value="non-precise">Non precise</option>
            </select>

            {/* Checkbox */}
            <label>
                <input
                    type="checkbox"
                    name="newsletter"
                    checked={valeurs.newsletter}
                    onChange={handleChanger}
                />
                S'abonner a la newsletter
            </label>

            {/* Textarea */}
            <textarea name="message" value={valeurs.message} onChange={handleChanger} />

            <button type="submit">Envoyer</button>
        </form>
    );
}
```

### Comparaison : controle vs non-controle

| Critere | Controle | Non-controle |
|---|---|---|
| Etat dans | React (useState) | DOM (ref) |
| Validation temps reel | Facile | Difficile |
| Valeur accessible | A chaque render | Seulement a la soumission |
| Reinitialisation | `setState({})` | `form.reset()` |
| Cas d'usage | Formulaires complexes | Formulaires simples |
| Avec React Hook Form | Recommande | Supporte |

> [!info] Recommandation pratique
> Pour la plupart des formulaires, utiliser React Hook Form (section suivante) plutot que de reimplementer la logique manuellement. RHF gere les deux approches et optimise les performances.

---

## 6. React Hook Form

### Installation et utilisation de base

```bash
npm install react-hook-form
```

```jsx
import { useForm } from "react-hook-form";

function FormulaireInscription() {
    const {
        register,        // Enregistre un champ dans le formulaire
        handleSubmit,    // Wrapper de soumission
        formState: {     // Etat du formulaire
            errors,      // Erreurs de validation
            isSubmitting, // true pendant la soumission async
            isValid,     // false si un champ est invalide
            dirtyFields, // Champs modifies par l'utilisateur
        },
        watch,           // Observer la valeur d'un champ en temps reel
        reset,           // Reinitialiser le formulaire
        setValue,        // Modifier une valeur programmatiquement
    } = useForm({
        mode: "onBlur", // Quand valider : "onChange" | "onBlur" | "onSubmit"
        defaultValues: {
            nom: "",
            email: "",
            motDePasse: "",
            motDePasseConfirmation: "",
            accepteConditions: false,
        },
    });

    const motDePasse = watch("motDePasse"); // Observer le champ mot de passe

    async function onSoumettre(donnees) {
        // donnees = objet avec toutes les valeurs validees
        try {
            await creerCompte(donnees);
            reset(); // Reinitialiser apres succes
        } catch (err) {
            console.error(err);
        }
    }

    return (
        <form onSubmit={handleSubmit(onSoumettre)}>
            {/* ─── Champ avec validation ─── */}
            <div>
                <label htmlFor="nom">Nom *</label>
                <input
                    id="nom"
                    // register : connecte le champ a RHF + validation
                    {...register("nom", {
                        required: "Le nom est obligatoire",
                        minLength: { value: 2, message: "Minimum 2 caracteres" },
                        maxLength: { value: 50, message: "Maximum 50 caracteres" },
                        pattern: {
                            value: /^[A-Za-zÀ-ÿ\s'-]+$/,
                            message: "Caracteres invalides",
                        },
                    })}
                    aria-invalid={!!errors.nom}
                />
                {errors.nom && (
                    <span role="alert" style={{ color: "red" }}>
                        {errors.nom.message}
                    </span>
                )}
            </div>

            {/* ─── Email ─── */}
            <div>
                <label htmlFor="email">Email *</label>
                <input
                    id="email"
                    type="email"
                    {...register("email", {
                        required: "L'email est obligatoire",
                        pattern: {
                            value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
                            message: "Format d'email invalide",
                        },
                    })}
                />
                {errors.email && <span role="alert">{errors.email.message}</span>}
            </div>

            {/* ─── Mot de passe ─── */}
            <div>
                <label htmlFor="motDePasse">Mot de passe *</label>
                <input
                    id="motDePasse"
                    type="password"
                    {...register("motDePasse", {
                        required: "Le mot de passe est obligatoire",
                        minLength: { value: 8, message: "Minimum 8 caracteres" },
                        validate: {
                            majuscule: v => /[A-Z]/.test(v) || "Doit contenir une majuscule",
                            chiffre: v => /\d/.test(v) || "Doit contenir un chiffre",
                        },
                    })}
                />
                {errors.motDePasse && <span role="alert">{errors.motDePasse.message}</span>}
            </div>

            {/* ─── Confirmation mot de passe ─── */}
            <div>
                <label htmlFor="motDePasseConfirmation">Confirmer *</label>
                <input
                    id="motDePasseConfirmation"
                    type="password"
                    {...register("motDePasseConfirmation", {
                        required: "Confirmer le mot de passe",
                        validate: v =>
                            v === motDePasse || "Les mots de passe ne correspondent pas",
                    })}
                />
                {errors.motDePasseConfirmation && (
                    <span role="alert">{errors.motDePasseConfirmation.message}</span>
                )}
            </div>

            {/* ─── Checkbox ─── */}
            <div>
                <label>
                    <input
                        type="checkbox"
                        {...register("accepteConditions", {
                            required: "Vous devez accepter les conditions",
                        })}
                    />
                    J'accepte les conditions d'utilisation
                </label>
                {errors.accepteConditions && (
                    <span role="alert">{errors.accepteConditions.message}</span>
                )}
            </div>

            <button type="submit" disabled={isSubmitting || !isValid}>
                {isSubmitting ? "Creation du compte..." : "Creer mon compte"}
            </button>
        </form>
    );
}
```

### useController — pour les composants custom

```jsx
import { useController } from "react-hook-form";

// ─── Composant Select personnalise integre avec RHF ───
function SelectCustom({ name, control, options, label }) {
    const {
        field: { value, onChange, onBlur, ref },
        fieldState: { error },
    } = useController({ name, control });

    return (
        <div>
            <label>{label}</label>
            <select value={value} onChange={onChange} onBlur={onBlur} ref={ref}>
                {options.map(opt => (
                    <option key={opt.value} value={opt.value}>{opt.label}</option>
                ))}
            </select>
            {error && <span>{error.message}</span>}
        </div>
    );
}
```

---

## 7. Validation avec Zod

Zod est une bibliotheque de validation TypeScript-first qui s'integre parfaitement avec React Hook Form via `@hookform/resolvers`.

```bash
npm install zod @hookform/resolvers
```

### Schema de validation Zod

```jsx
import { z } from "zod";
import { zodResolver } from "@hookform/resolvers/zod";
import { useForm } from "react-hook-form";

// ─── Definir le schema de validation ───
const schemaInscription = z.object({
    nom: z
        .string()
        .min(2, "Minimum 2 caracteres")
        .max(50, "Maximum 50 caracteres")
        .regex(/^[A-Za-zÀ-ÿ\s'-]+$/, "Caracteres invalides"),

    email: z
        .string()
        .email("Format d'email invalide"),

    age: z
        .number({ invalid_type_error: "Doit etre un nombre" })
        .int("Doit etre un entier")
        .min(18, "Doit avoir au moins 18 ans")
        .max(120, "Age invalide"),

    role: z.enum(["utilisateur", "moderateur", "admin"], {
        errorMap: () => ({ message: "Role invalide" }),
    }),

    motDePasse: z
        .string()
        .min(8, "Minimum 8 caracteres")
        .regex(/[A-Z]/, "Doit contenir une majuscule")
        .regex(/[0-9]/, "Doit contenir un chiffre")
        .regex(/[^A-Za-z0-9]/, "Doit contenir un caractere special"),

    confirmation: z.string(),

    bio: z.string().max(500, "Maximum 500 caracteres").optional(),

    accepte: z.literal(true, {
        errorMap: () => ({ message: "Vous devez accepter les conditions" }),
    }),
}).refine(
    // Validation croisee : confirmation doit correspondre au mot de passe
    (data) => data.motDePasse === data.confirmation,
    {
        message: "Les mots de passe ne correspondent pas",
        path: ["confirmation"], // L'erreur apparait sur le champ "confirmation"
    }
);

// TypeScript : extraire le type depuis le schema
type DonneesInscription = z.infer<typeof schemaInscription>;
```

```jsx
// ─── Integration avec React Hook Form ───
function FormulaireInscriptionAvecZod() {
    const {
        register,
        handleSubmit,
        formState: { errors, isSubmitting },
    } = useForm({
        // zodResolver valide automatiquement avec le schema
        resolver: zodResolver(schemaInscription),
        defaultValues: {
            nom: "",
            email: "",
            age: undefined,
            role: "utilisateur",
            motDePasse: "",
            confirmation: "",
            bio: "",
            accepte: false,
        },
    });

    async function onSoumettre(donnees) {
        // donnees est deja valide et type-safe
        console.log(donnees);
        await creerCompte(donnees);
    }

    return (
        <form onSubmit={handleSubmit(onSoumettre)}>
            <div>
                <input {...register("nom")} placeholder="Nom" />
                {errors.nom && <p>{errors.nom.message}</p>}
            </div>

            <div>
                <input {...register("email")} type="email" placeholder="Email" />
                {errors.email && <p>{errors.email.message}</p>}
            </div>

            <div>
                <input
                    {...register("age", { valueAsNumber: true })} // Convertir en nombre
                    type="number"
                    placeholder="Age"
                />
                {errors.age && <p>{errors.age.message}</p>}
            </div>

            <div>
                <select {...register("role")}>
                    <option value="utilisateur">Utilisateur</option>
                    <option value="moderateur">Moderateur</option>
                    <option value="admin">Administrateur</option>
                </select>
                {errors.role && <p>{errors.role.message}</p>}
            </div>

            <div>
                <input {...register("motDePasse")} type="password" />
                {errors.motDePasse && <p>{errors.motDePasse.message}</p>}
            </div>

            <div>
                <input {...register("confirmation")} type="password" />
                {errors.confirmation && <p>{errors.confirmation.message}</p>}
            </div>

            <button type="submit" disabled={isSubmitting}>
                {isSubmitting ? "Envoi..." : "S'inscrire"}
            </button>
        </form>
    );
}
```

---

## 8. Soumission a une API REST

### Gestion complete des etats de soumission

```jsx
import { useState } from "react";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

// ─── Etats possibles d'un formulaire ───
// idle → loading → success (→ idle apres redirect/reset)
//                → error (→ idle apres modification)

function FormulaireCreationArticle() {
    const [etat, setEtat] = useState("idle"); // "idle" | "loading" | "success" | "error"
    const [erreurAPI, setErreurAPI] = useState(null);

    const {
        register,
        handleSubmit,
        reset,
        setError,
        formState: { errors },
    } = useForm({ resolver: zodResolver(schemaArticle) });

    async function onSoumettre(donnees) {
        setEtat("loading");
        setErreurAPI(null);

        try {
            const reponse = await fetch("/api/articles", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Bearer ${localStorage.getItem("token")}`,
                },
                body: JSON.stringify(donnees),
            });

            const resultat = await reponse.json();

            if (!reponse.ok) {
                // Erreurs de validation retournees par le serveur
                if (reponse.status === 422 && resultat.errors) {
                    // Propager les erreurs serveur vers les champs RHF
                    Object.entries(resultat.errors).forEach(([champ, message]) => {
                        setError(champ, { type: "server", message });
                    });
                    setEtat("idle");
                    return;
                }

                throw new Error(resultat.message || "Erreur lors de la creation");
            }

            setEtat("success");
            reset();
            // Optionnel : rediriger vers l'article cree
            // navigate(`/articles/${resultat.id}`);
        } catch (err) {
            setErreurAPI(err.message);
            setEtat("error");
        }
    }

    // ─── Rendu selon l'etat ───
    if (etat === "success") {
        return (
            <div className="succes">
                <h2>Article cree avec succes !</h2>
                <button onClick={() => setEtat("idle")}>Creer un autre article</button>
            </div>
        );
    }

    return (
        <form onSubmit={handleSubmit(onSoumettre)}>
            {/* Erreur globale de l'API */}
            {etat === "error" && erreurAPI && (
                <div role="alert" className="alerte-erreur">
                    <strong>Erreur :</strong> {erreurAPI}
                    <button onClick={() => setEtat("idle")}>Reessayer</button>
                </div>
            )}

            <div>
                <label htmlFor="titre">Titre *</label>
                <input id="titre" {...register("titre")} disabled={etat === "loading"} />
                {errors.titre && <span role="alert">{errors.titre.message}</span>}
            </div>

            <div>
                <label htmlFor="contenu">Contenu *</label>
                <textarea id="contenu" {...register("contenu")} rows={10} disabled={etat === "loading"} />
                {errors.contenu && <span role="alert">{errors.contenu.message}</span>}
            </div>

            <button type="submit" disabled={etat === "loading"}>
                {etat === "loading" ? (
                    <>
                        <span className="spinner" aria-hidden="true" />
                        Creation en cours...
                    </>
                ) : (
                    "Creer l'article"
                )}
            </button>
        </form>
    );
}
```

---

## 9. Patterns de Formulaires Courants

### Formulaire de connexion

```jsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { useNavigate, useLocation } from "react-router-dom";
import { useAuth } from "../contexts/AuthContext";

const schemaConnexion = z.object({
    email: z.string().email("Email invalide"),
    motDePasse: z.string().min(1, "Le mot de passe est requis"),
    seRappelerDeMoi: z.boolean().default(false),
});

function FormulaireConnexion() {
    const { connecter } = useAuth();
    const navigate = useNavigate();
    const location = useLocation();
    const destination = location.state?.depuis?.pathname || "/tableau-de-bord";

    const {
        register,
        handleSubmit,
        setError,
        formState: { errors, isSubmitting },
    } = useForm({
        resolver: zodResolver(schemaConnexion),
        defaultValues: { email: "", motDePasse: "", seRappelerDeMoi: false },
    });

    async function onSoumettre({ email, motDePasse, seRappelerDeMoi }) {
        try {
            await connecter(email, motDePasse, seRappelerDeMoi);
            navigate(destination, { replace: true });
        } catch (err) {
            // Erreur d'authentification : afficher sur le champ mot de passe
            setError("motDePasse", {
                type: "server",
                message: "Email ou mot de passe incorrect",
            });
        }
    }

    return (
        <div className="formulaire-connexion">
            <h1>Connexion</h1>
            <form onSubmit={handleSubmit(onSoumettre)} noValidate>
                <div className="champ">
                    <label htmlFor="email">Email</label>
                    <input
                        id="email"
                        type="email"
                        autoComplete="email"
                        {...register("email")}
                    />
                    {errors.email && <p className="erreur">{errors.email.message}</p>}
                </div>

                <div className="champ">
                    <label htmlFor="motDePasse">Mot de passe</label>
                    <input
                        id="motDePasse"
                        type="password"
                        autoComplete="current-password"
                        {...register("motDePasse")}
                    />
                    {errors.motDePasse && (
                        <p className="erreur" role="alert">{errors.motDePasse.message}</p>
                    )}
                </div>

                <label>
                    <input type="checkbox" {...register("seRappelerDeMoi")} />
                    Se souvenir de moi
                </label>

                <button type="submit" disabled={isSubmitting}>
                    {isSubmitting ? "Connexion..." : "Se connecter"}
                </button>
            </form>

            <p>
                Pas encore de compte ? <Link to="/inscription">S'inscrire</Link>
            </p>
            <p>
                <Link to="/mot-de-passe-oublie">Mot de passe oublie ?</Link>
            </p>
        </div>
    );
}
```

### Formulaire multi-etapes (wizard)

```jsx
import { useState } from "react";
import { useForm } from "react-hook-form";

const ETAPES = ["Informations", "Adresse", "Confirmation"];

function FormulaireMultiEtapes() {
    const [etapeActuelle, setEtapeActuelle] = useState(0);
    const { register, handleSubmit, trigger, getValues, formState: { errors } } = useForm();

    // Validation de l'etape actuelle avant de passer a la suivante
    async function etapeSuivante() {
        // Definir quels champs valider selon l'etape
        const champsParEtape = [
            ["nom", "email", "telephone"],
            ["rue", "ville", "codePostal", "pays"],
        ];

        const champsAValider = champsParEtape[etapeActuelle];
        const valide = await trigger(champsAValider);

        if (valide) {
            setEtapeActuelle(e => e + 1);
        }
    }

    async function onSoumettre(donnees) {
        console.log("Toutes les donnees :", donnees);
        await creerCompte(donnees);
    }

    return (
        <div>
            {/* Indicateur de progression */}
            <div className="progression">
                {ETAPES.map((etape, i) => (
                    <div
                        key={etape}
                        className={`etape ${i <= etapeActuelle ? "active" : ""}`}
                    >
                        <span>{i + 1}</span>
                        <p>{etape}</p>
                    </div>
                ))}
            </div>

            <form onSubmit={handleSubmit(onSoumettre)}>
                {/* Etape 1 : Informations personnelles */}
                {etapeActuelle === 0 && (
                    <div>
                        <h2>Informations personnelles</h2>
                        <input {...register("nom", { required: "Requis" })} placeholder="Nom" />
                        {errors.nom && <p>{errors.nom.message}</p>}
                        <input {...register("email", { required: "Requis" })} type="email" placeholder="Email" />
                        {errors.email && <p>{errors.email.message}</p>}
                    </div>
                )}

                {/* Etape 2 : Adresse */}
                {etapeActuelle === 1 && (
                    <div>
                        <h2>Adresse</h2>
                        <input {...register("rue", { required: "Requis" })} placeholder="Rue" />
                        {errors.rue && <p>{errors.rue.message}</p>}
                        <input {...register("ville", { required: "Requis" })} placeholder="Ville" />
                    </div>
                )}

                {/* Etape 3 : Recapitulatif */}
                {etapeActuelle === 2 && (
                    <div>
                        <h2>Confirmation</h2>
                        <pre>{JSON.stringify(getValues(), null, 2)}</pre>
                    </div>
                )}

                {/* Boutons de navigation */}
                <div className="navigation">
                    {etapeActuelle > 0 && (
                        <button type="button" onClick={() => setEtapeActuelle(e => e - 1)}>
                            Precedent
                        </button>
                    )}
                    {etapeActuelle < ETAPES.length - 1 ? (
                        <button type="button" onClick={etapeSuivante}>
                            Suivant
                        </button>
                    ) : (
                        <button type="submit">Valider</button>
                    )}
                </div>
            </form>
        </div>
    );
}
```

---

## Carte Mentale ASCII

```
              REACT ROUTER v6 + FORMULAIRES
                          │
        ┌─────────────────┼──────────────────┐
        │                 │                  │
   REACT ROUTER      FORMULAIRES         PATTERNS
        │                 │                  │
   BrowserRouter     Controle           Connexion
   Routes/Route      Non-controle       Inscription
   Link/NavLink      React Hook Form    Multi-etapes
   Outlet            register           Profil edit
        │            handleSubmit       Recherche
   useNavigate       errors             Upload fichier
   useParams         isSubmitting           │
   useLocation            │           FEEDBACK
   useSearchParams   VALIDATION        loading state
        │            Zod schema        success message
   Routes imbriq.    zodResolver       erreur API
   Layout pattern    refine()          setError()
   Outlet            validate custom       │
        │                 │           ACCESSIBILITE
   Route protegee     API Submit       aria-invalid
   RequiertAuth       fetch POST       aria-label
   Navigate           setError()       role="alert"
   replace            states           htmlFor/id
        │             idle/loading/    noValidate
   Lazy Loading       success/error
   lazy()
   Suspense
   Code splitting
```

---

## Exercices

### Exercice 1 : Application multi-pages avec React Router

Creez une application de blog avec :
- Route `/` : liste des articles (avec pagination)
- Route `/articles/:id` : detail d'un article
- Route `/articles/:id/commentaires` : commentaires (route enfant)
- Route `/ecrire` : formulaire de creation (protegee)
- Route `/connexion` : formulaire de connexion
- Layout partagee avec Header et Footer
- NavLink avec style actif sur la navigation
- 404 page stylisee
- Lazy loading pour la page d'ecriture

### Exercice 2 : Formulaire d'inscription complet avec validation Zod

Creez un formulaire d'inscription avec au moins 8 champs :
- Nom, prenom, email, mot de passe, confirmation
- Date de naissance (avec validation age minimum)
- Pays (select avec liste de pays)
- Bio optionnelle (textarea, max 500 caracteres)
- Avatar URL optionnel (validate que c'est une URL valide si rempli)
- Case "J'accepte les CGU" obligatoire
- Tous les messages d'erreur en francais
- Les champs invalides ont une bordure rouge, les valides une bordure verte
- Le bouton est desactive si le formulaire est invalide

### Exercice 3 : Formulaire d'edition de profil avec pre-remplissage

Creez un composant `EditerProfil` qui :
- Charge les donnees du profil depuis une API au montage (useEffect ou React Query)
- Pre-remplit le formulaire avec les donnees existantes (`defaultValues` dynamiques)
- Detecte les modifications (champs "dirty")
- Affiche un message "Modifications non sauvegardees" si on quitte avec des changements (useBeforeUnload ou Prompt React Router)
- Valide et soumet les changements a l'API (PATCH)
- Affiche un toast de confirmation ou d'erreur

### Exercice 4 : Wizard de creation de produit

Creez un formulaire en 4 etapes pour creer un produit e-commerce :
- Etape 1 : Informations de base (nom, description, categorie)
- Etape 2 : Prix et stock (prix HT, TVA auto-calculee, quantite)
- Etape 3 : Images (URL multiples, preview)
- Etape 4 : Confirmation avec recapitulatif
- Barre de progression visuelle
- Validation specifique a chaque etape avant d'avancer
- Possibilite de revenir aux etapes precedentes sans perdre les donnees
- Sauvegarder le brouillon en localStorage a chaque etape

---

## Liens

- [[01 - Introduction a React]]
- [[02 - Hooks et Gestion d'Etat]]
- [[04 - React Avance et Tests]]
- [[05 - SPA et Frameworks Introduction]]
