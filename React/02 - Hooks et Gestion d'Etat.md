# Hooks et Gestion d'Etat

Les hooks sont la fonctionnalite la plus importante de React moderne. Introduits en React 16.8 (fevrier 2019), ils permettent aux composants fonctionnels d'acceder a l'etat, aux effets de bord et a d'autres fonctionnalites de React sans ecrire une classe. Aujourd'hui, les hooks sont la norme absolue : tout nouveau code React doit les utiliser.

Ce cours couvre les hooks fondamentaux (`useState`, `useEffect`, `useRef`), les hooks d'optimisation (`useCallback`, `useMemo`), les custom hooks, la Context API et une introduction a Redux Toolkit pour la gestion d'etat globale.

> [!tip] Analogie
> Un hook, c'est comme un abonnement. `useState` vous abonne aux changements d'une variable : quand elle change, React reabonne votre composant (le re-rend). `useEffect` vous abonne a des evenements du cycle de vie du composant. Vous definissez ce qui se passe, React gere le quand.

---

## 1. Regles des Hooks

Avant tout, deux regles inviolables. Les hooks ne fonctionnent que si elles sont respectees.

### Regle 1 : Appeler les hooks uniquement au niveau superieur

```jsx
// ❌ INTERDIT : hook dans une condition
function Mauvais({ estConnecte }) {
    if (estConnecte) {
        const [nom, setNom] = useState(""); // Hook conditionnel → ERREUR
    }
    // ...
}

// ❌ INTERDIT : hook dans une boucle
function Mauvais2({ items }) {
    for (let i = 0; i < items.length; i++) {
        const [selectionne, setSelectionne] = useState(false); // ERREUR
    }
}

// ✅ CORRECT : hooks toujours en haut du composant, inconditionnellement
function Correct({ estConnecte, items }) {
    const [nom, setNom] = useState("");
    const [selections, setSelections] = useState([]);
    // La logique conditionnelle APRES les hooks
    if (!estConnecte) return null;
    // ...
}
```

### Regle 2 : Appeler les hooks uniquement dans des composants React ou des custom hooks

```jsx
// ❌ INTERDIT : hook dans une fonction utilitaire ordinaire
function calculerTotal(items) {
    const [cache, setCache] = useState(null); // ERREUR
}

// ✅ CORRECT : dans un composant React
function Panier({ items }) {
    const [cache, setCache] = useState(null);
    // ...
}

// ✅ CORRECT : dans un custom hook (commence par "use")
function useCalculTotal(items) {
    const [cache, setCache] = useState(null);
    // ...
}
```

> [!warning] Pourquoi ces regles ?
> React maintient l'ordre des hooks appelee entre les renders. Si un hook est appele conditionnellement, l'ordre peut changer d'un render a l'autre, et React perd la correspondance entre les hooks et leurs valeurs stockees. C'est un bug silencieux catastrophique.

---

## 2. useState — Gestion de l'Etat Local

### Principe fondamental

```jsx
import { useState } from "react";

function Compteur() {
    // useState retourne un tuple [valeur, setter]
    // La valeur initiale est passee en argument (0 ici)
    const [compte, setCompte] = useState(0);

    // Chaque appel a setCompte declenche un re-render du composant
    return (
        <div>
            <p>Compteur : {compte}</p>
            <button onClick={() => setCompte(compte + 1)}>+1</button>
            <button onClick={() => setCompte(compte - 1)}>-1</button>
            <button onClick={() => setCompte(0)}>Reset</button>
        </div>
    );
}
```

### Mise a jour fonctionnelle (forme callback)

```jsx
function ExempleMiseAJourFonctionnelle() {
    const [compte, setCompte] = useState(0);

    // ─── Probleme : 'compte' est capture au moment du clic ───
    function incrementerMal() {
        setCompte(compte + 1); // utilise la valeur de 'compte' au moment du clic
        setCompte(compte + 1); // meme valeur ! resultat = compte + 1, pas +2
        setCompte(compte + 1); // React batchifie ces 3 appels → un seul render
    }

    // ─── Solution : forme fonctionnelle ─── utilise la valeur LA PLUS RECENTE
    function incrementerBien() {
        setCompte(prev => prev + 1); // prev = valeur avant cet appel
        setCompte(prev => prev + 1); // prev = resultat du precedent
        setCompte(prev => prev + 1); // resultat final = compte + 3
    }

    // ─── Regle pratique ───
    // Si la nouvelle valeur depend de l'ancienne → forme fonctionnelle
    // Sinon → forme directe

    return (
        <div>
            <p>{compte}</p>
            <button onClick={incrementerMal}>+1 (incorrect x3)</button>
            <button onClick={incrementerBien}>+3 (correct)</button>
        </div>
    );
}
```

### State avec objet

```jsx
function FormulaireUtilisateur() {
    // ─── Etat objet ───
    const [utilisateur, setUtilisateur] = useState({
        nom: "",
        email: "",
        age: 18,
        ville: "Paris",
    });

    // ─── Mise a jour partielle avec spread ───
    function handleChangerNom(e) {
        // IMPORTANT : ne pas muter l'objet directement
        // ❌ utilisateur.nom = e.target.value; → React ne detente pas le changement
        // ✅ Creer un nouvel objet
        setUtilisateur({
            ...utilisateur,           // Copie toutes les proprietes existantes
            nom: e.target.value,      // Ecrase uniquement 'nom'
        });
    }

    // ─── Version generique pour plusieurs champs ───
    function handleChamp(e) {
        const { name, value } = e.target;
        setUtilisateur(prev => ({ ...prev, [name]: value }));
    }

    return (
        <form>
            <input
                name="nom"
                value={utilisateur.nom}
                onChange={handleChamp}
                placeholder="Nom"
            />
            <input
                name="email"
                value={utilisateur.email}
                onChange={handleChamp}
                placeholder="Email"
            />
            <input
                name="age"
                type="number"
                value={utilisateur.age}
                onChange={handleChamp}
            />
            <p>Profil : {utilisateur.nom} ({utilisateur.ville})</p>
        </form>
    );
}
```

### State avec tableau

```jsx
function GestionnaireListe() {
    const [items, setItems] = useState(["React", "Vue", "Svelte"]);
    const [nouvelItem, setNouvelItem] = useState("");

    // ─── AJOUTER : ...spread a la fin ───
    function ajouter() {
        if (!nouvelItem.trim()) return;
        setItems(prev => [...prev, nouvelItem.trim()]);
        setNouvelItem("");
    }

    // ─── SUPPRIMER : filter ───
    function supprimer(index) {
        setItems(prev => prev.filter((_, i) => i !== index));
    }

    // ─── MODIFIER : map ───
    function modifier(index, nouvelleValeur) {
        setItems(prev => prev.map((item, i) =>
            i === index ? nouvelleValeur : item
        ));
    }

    // ─── DEPLACER : creer un nouveau tableau reordonne ───
    function monterElement(index) {
        if (index === 0) return;
        const nouveau = [...items];
        [nouveau[index - 1], nouveau[index]] = [nouveau[index], nouveau[index - 1]];
        setItems(nouveau);
    }

    return (
        <div>
            <div>
                <input
                    value={nouvelItem}
                    onChange={e => setNouvelItem(e.target.value)}
                    onKeyDown={e => e.key === "Enter" && ajouter()}
                    placeholder="Nouveau framework..."
                />
                <button onClick={ajouter}>Ajouter</button>
            </div>
            <ul>
                {items.map((item, index) => (
                    <li key={item}>
                        {item}
                        <button onClick={() => monterElement(index)}>↑</button>
                        <button onClick={() => supprimer(index)}>×</button>
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

### Initialisation paresseuse (lazy initialization)

```jsx
// ─── Probleme : calcul couteux execute a chaque render ───
function MauvaisInit() {
    // lireDepuisLocalStorage() est appelee a CHAQUE render, meme si la valeur
    // initiale n'est utilisee qu'une seule fois (au premier render)
    const [valeur, setValeur] = useState(lireDepuisLocalStorage("cle"));
}

// ─── Solution : passer une FONCTION (executee une seule fois) ───
function BonneInit() {
    // React appelle la fonction uniquement lors du PREMIER render
    const [valeur, setValeur] = useState(() => lireDepuisLocalStorage("cle"));
}
```

---

## 3. useEffect — Effets de Bord

### Principe

```jsx
import { useState, useEffect } from "react";

// useEffect(callback, [dependances])
// - callback : ce qui s'execute apres le render
// - dependances : tableau des valeurs qui declenchent une re-execution
```

### Les 3 modes de useEffect

```jsx
function ExemplesUseEffect() {
    const [compte, setCompte] = useState(0);
    const [nom, setNom] = useState("Alice");

    // ─── Mode 1 : apres CHAQUE render ───
    useEffect(() => {
        console.log("Render effectue (compte ou nom a change)");
        // Equivalent de componentDidUpdate (sans condition)
    }); // Pas de tableau de dependances !

    // ─── Mode 2 : uniquement au premier render (montage) ───
    useEffect(() => {
        console.log("Composant monte !");
        // Fetch initial, abonnements, timers...
        // Equivalent de componentDidMount
        return () => {
            console.log("Composant demonte !");
            // Nettoyage : equivalent de componentWillUnmount
        };
    }, []); // Tableau vide

    // ─── Mode 3 : quand des valeurs specifiques changent ───
    useEffect(() => {
        document.title = `${nom} — compteur : ${compte}`;
        // S'execute apres le render initial ET quand compte ou nom change
    }, [compte, nom]); // Re-execute si compte OU nom change

    return (
        <div>
            <p>{nom} : {compte}</p>
            <button onClick={() => setCompte(c => c + 1)}>+1</button>
            <input value={nom} onChange={e => setNom(e.target.value)} />
        </div>
    );
}
```

### Cleanup — Nettoyage des effets

```jsx
function EffetsAvecNettoyage() {
    const [visible, setVisible] = useState(true);
    const [position, setPosition] = useState({ x: 0, y: 0 });
    const [secondes, setSecondes] = useState(0);

    // ─── Cleanup : evenement DOM ───
    useEffect(() => {
        function handleMouseMove(e) {
            setPosition({ x: e.clientX, y: e.clientY });
        }

        window.addEventListener("mousemove", handleMouseMove);

        // Cleanup : retire l'evenement quand le composant se demonte
        // ou avant la prochaine execution de l'effet
        return () => {
            window.removeEventListener("mousemove", handleMouseMove);
        };
    }, []); // Une seule fois (montage/demontage)

    // ─── Cleanup : timer ───
    useEffect(() => {
        const intervalId = setInterval(() => {
            setSecondes(s => s + 1);
        }, 1000);

        return () => clearInterval(intervalId); // Cleanup obligatoire !
    }, []);

    // ─── Cleanup : requete annulee (AbortController) ───
    useEffect(() => {
        const controller = new AbortController();

        fetch("/api/donnees", { signal: controller.signal })
            .then(r => r.json())
            .then(data => console.log(data))
            .catch(err => {
                if (err.name !== "AbortError") {
                    console.error(err);
                }
            });

        // Si le composant se demonte avant la fin de la requete → annuler
        return () => controller.abort();
    }, []);

    return (
        <div>
            <p>Souris : ({position.x}, {position.y})</p>
            <p>Temps : {secondes}s</p>
        </div>
    );
}
```

### Fetch de donnees avec useEffect

```jsx
function ListeUtilisateurs() {
    const [utilisateurs, setUtilisateurs] = useState([]);
    const [chargement, setChargement] = useState(true);
    const [erreur, setErreur] = useState(null);
    const [page, setPage] = useState(1);

    useEffect(() => {
        // Eviter les "race conditions" : ignorer les reponses des requetes anciennes
        let estMonte = true;

        async function charger() {
            try {
                setChargement(true);
                setErreur(null);

                const reponse = await fetch(
                    `https://jsonplaceholder.typicode.com/users?_page=${page}&_limit=5`
                );

                if (!reponse.ok) {
                    throw new Error(`HTTP ${reponse.status}`);
                }

                const donnees = await reponse.json();

                // Ne mettre a jour que si le composant est encore monte
                if (estMonte) {
                    setUtilisateurs(donnees);
                }
            } catch (err) {
                if (estMonte) {
                    setErreur(err.message);
                }
            } finally {
                if (estMonte) {
                    setChargement(false);
                }
            }
        }

        charger();

        return () => {
            estMonte = false; // Evite les mises a jour sur composant demonte
        };
    }, [page]); // Re-execute quand 'page' change

    if (chargement) return <p>Chargement...</p>;
    if (erreur) return <p>Erreur : {erreur}</p>;

    return (
        <div>
            <ul>
                {utilisateurs.map(u => (
                    <li key={u.id}>{u.name} — {u.email}</li>
                ))}
            </ul>
            <button onClick={() => setPage(p => p - 1)} disabled={page === 1}>
                Precedent
            </button>
            <span> Page {page} </span>
            <button onClick={() => setPage(p => p + 1)}>Suivant</button>
        </div>
    );
}
```

### Analogies avec les classes (componentDidMount, etc.)

| Classe | Equivalent Hook |
|---|---|
| `componentDidMount` | `useEffect(() => { ... }, [])` |
| `componentDidUpdate` | `useEffect(() => { ... })` sans tableau |
| `componentDidUpdate` ciblé | `useEffect(() => { ... }, [dep])` |
| `componentWillUnmount` | Fonction de retour du `useEffect` |
| `shouldComponentUpdate` | `React.memo` + `useMemo` / `useCallback` |
| `getDerivedStateFromProps` | Calcul direct dans le corps du composant |

---

## 4. useRef — References et Valeurs Persistantes

### Deux usages principaux

```jsx
import { useRef, useEffect, useState } from "react";

function ExemplesUseRef() {
    // ─── Usage 1 : reference a un element DOM ───
    const inputRef = useRef(null);
    const videoRef = useRef(null);

    function focusInput() {
        inputRef.current.focus();
    }

    function lireVideo() {
        videoRef.current.play();
    }

    // ─── Usage 2 : valeur persistante SANS re-render ───
    const compteurRendersRef = useRef(0);
    const [compte, setCompte] = useState(0);

    // A chaque render, on incremente le compteur SANS provoquer un nouveau render
    compteurRendersRef.current += 1;

    // ─── Stocker une valeur de l'effet precedent ───
    const comptePrecedentRef = useRef(compte);
    useEffect(() => {
        comptePrecedentRef.current = compte;
    });

    return (
        <div>
            {/* Attacher la ref a un element DOM ─── */}
            <input ref={inputRef} placeholder="Cliquez 'Focus' pour activer" />
            <button onClick={focusInput}>Focus</button>

            <video ref={videoRef} src="video.mp4" />
            <button onClick={lireVideo}>Lire</button>

            <p>Renders effectues : {compteurRendersRef.current}</p>
            <p>Compte actuel : {compte}, precedent : {comptePrecedentRef.current}</p>
            <button onClick={() => setCompte(c => c + 1)}>+1</button>
        </div>
    );
}
```

### useRef vs useState : quand utiliser quoi ?

```
              BESOIN DE RE-RENDER           PAS BESOIN DE RE-RENDER
              quand la valeur change ?      quand la valeur change ?
                        │                              │
                        v                              v
                    useState                        useRef
                        │                              │
              Compteur visible              Timer ID (clearInterval)
              Texte affiche                Scroll position (tracking)
              Elements d'une liste         Valeur precedente
              Etat du formulaire           Flag "premier render"
              Resultat d'un calcul         Abonnements, instances
                                           References DOM
```

```jsx
// ─── Exemple : timer avec start/stop ───
function Timer() {
    const [secondes, setSecondes] = useState(0);
    const [actif, setActif] = useState(false);
    // L'ID du timer ne doit pas provoquer de re-render → useRef
    const intervalRef = useRef(null);

    function demarrer() {
        if (actif) return;
        setActif(true);
        intervalRef.current = setInterval(() => {
            setSecondes(s => s + 1);
        }, 1000);
    }

    function stopper() {
        clearInterval(intervalRef.current);
        setActif(false);
    }

    function reinitialiser() {
        stopper();
        setSecondes(0);
    }

    useEffect(() => {
        return () => clearInterval(intervalRef.current); // Cleanup au demontage
    }, []);

    return (
        <div>
            <p>{secondes}s</p>
            <button onClick={demarrer} disabled={actif}>Demarrer</button>
            <button onClick={stopper} disabled={!actif}>Stopper</button>
            <button onClick={reinitialiser}>Reinitialiser</button>
        </div>
    );
}
```

---

## 5. useCallback et useMemo — Optimisation

### Le probleme de performance

```jsx
// ─── Sans optimisation : chaque render recrée toutes les fonctions ───
function ParentSansMemo({ items }) {
    const [filtre, setFiltre] = useState("");

    // Ces fonctions sont RECREEES a chaque render du parent
    function handleSupprimer(id) {
        console.log(`Supprimer ${id}`);
    }

    const itemsFiltres = items.filter(item =>
        item.nom.includes(filtre) // Calcul refait a chaque render
    );

    return (
        <div>
            <input value={filtre} onChange={e => setFiltre(e.target.value)} />
            {/* EnfantOptimise se re-rend inutilement car handleSupprimer change */}
            <EnfantOptimise items={itemsFiltres} onSupprimer={handleSupprimer} />
        </div>
    );
}
```

### useCallback — Memoriser une fonction

```jsx
import { useCallback, useMemo, useState, memo } from "react";

// React.memo : le composant ne se re-rend que si ses props changent
const ListeEnfant = memo(function ListeEnfant({ items, onSupprimer }) {
    console.log("ListeEnfant render"); // Surveiller les re-renders
    return (
        <ul>
            {items.map(item => (
                <li key={item.id}>
                    {item.nom}
                    <button onClick={() => onSupprimer(item.id)}>×</button>
                </li>
            ))}
        </ul>
    );
});

function ParentOptimise({ items }) {
    const [filtre, setFiltre] = useState("");
    const [autreEtat, setAutreEtat] = useState(0);

    // useCallback : memorise la reference de la fonction
    // Ne recrée la fonction que si ses dependances changent
    const handleSupprimer = useCallback((id) => {
        console.log(`Supprimer ${id}`);
        // Si on avait besoin de mettre a jour items, on ferait setState ici
    }, []); // Dependances vides = meme fonction toute la vie du composant

    // useMemo : memorise le resultat d'un calcul
    // Ne recalcule que si 'items' ou 'filtre' change
    const itemsFiltres = useMemo(() => {
        console.log("Filtrage des items...");
        return items.filter(item =>
            item.nom.toLowerCase().includes(filtre.toLowerCase())
        );
    }, [items, filtre]);

    return (
        <div>
            <input value={filtre} onChange={e => setFiltre(e.target.value)} />
            <button onClick={() => setAutreEtat(n => n + 1)}>
                Autre etat ({autreEtat}) — ne re-rend pas ListeEnfant
            </button>
            <ListeEnfant items={itemsFiltres} onSupprimer={handleSupprimer} />
        </div>
    );
}
```

### Quand utiliser useCallback et useMemo ?

```
AVANT d'optimiser : MESURER avec React DevTools Profiler

useCallback → utile QUAND :
  ✓ La fonction est passee en prop a un composant memo()
  ✓ La fonction est dans les dependances d'un useEffect
  ✗ INUTILE si la fonction n'est pas passee en prop
  ✗ INUTILE si l'enfant n'est pas memo()

useMemo → utile QUAND :
  ✓ Le calcul est reellement couteux (tri de 10 000 elements)
  ✓ La valeur est passee en prop a un composant memo()
  ✗ INUTILE pour des calculs simples (addition, .length)
  ✗ INUTILE si le composant est deja rapide

Regles pratiques :
  - Ne pas optimiser prematurément
  - Mesurer avant d'ajouter useCallback/useMemo
  - Si le perf profiler ne montre pas de probleme → ne pas optimiser
```

> [!warning] L'optimisation prematuree est un anti-pattern
> `useCallback` et `useMemo` ont eux-memes un cout (creation du cache, comparaison des dependances). Sur des composants simples, ils peuvent rendre l'application PLUS lente. Utiliser uniquement sur des composants identifies comme lents par le profiler.

---

## 6. Custom Hooks

### Principe — extraire et reutiliser la logique

```jsx
// ─── AVANT : logique dupliqueee dans chaque composant ───
function ComponentA() {
    const [largeur, setLargeur] = useState(window.innerWidth);
    useEffect(() => {
        function handler() { setLargeur(window.innerWidth); }
        window.addEventListener("resize", handler);
        return () => window.removeEventListener("resize", handler);
    }, []);
    // ... utilise largeur
}

function ComponentB() {
    // Code identique duplique !
    const [largeur, setLargeur] = useState(window.innerWidth);
    useEffect(() => {
        function handler() { setLargeur(window.innerWidth); }
        window.addEventListener("resize", handler);
        return () => window.removeEventListener("resize", handler);
    }, []);
}

// ─── APRES : custom hook reutilisable ───
function useWindowWidth() {
    const [largeur, setLargeur] = useState(window.innerWidth);

    useEffect(() => {
        function handler() { setLargeur(window.innerWidth); }
        window.addEventListener("resize", handler);
        return () => window.removeEventListener("resize", handler);
    }, []);

    return largeur;
}

function ComponentA() {
    const largeur = useWindowWidth(); // Une ligne !
}
function ComponentB() {
    const largeur = useWindowWidth(); // Meme logique, aucune duplication
}
```

### Custom hook : useLocalStorage

```jsx
// ─── hooks/useLocalStorage.js ───
import { useState, useEffect } from "react";

function useLocalStorage(cle, valeurInitiale) {
    // Initialisation paresseuse : lire localStorage une seule fois
    const [valeur, setValeur] = useState(() => {
        try {
            const stockee = localStorage.getItem(cle);
            return stockee !== null ? JSON.parse(stockee) : valeurInitiale;
        } catch {
            return valeurInitiale;
        }
    });

    // Synchroniser localStorage quand la valeur change
    useEffect(() => {
        try {
            localStorage.setItem(cle, JSON.stringify(valeur));
        } catch (err) {
            console.warn(`useLocalStorage: erreur pour "${cle}"`, err);
        }
    }, [cle, valeur]);

    return [valeur, setValeur];
}

// ─── Utilisation ───
function Preferences() {
    const [theme, setTheme] = useLocalStorage("theme", "clair");
    const [langue, setLangue] = useLocalStorage("langue", "fr");

    return (
        <div>
            <select value={theme} onChange={e => setTheme(e.target.value)}>
                <option value="clair">Clair</option>
                <option value="sombre">Sombre</option>
            </select>
            <select value={langue} onChange={e => setLangue(e.target.value)}>
                <option value="fr">Francais</option>
                <option value="en">English</option>
            </select>
        </div>
    );
}
```

### Custom hook : useFetch

```jsx
// ─── hooks/useFetch.js ───
import { useState, useEffect, useCallback } from "react";

function useFetch(url, options = {}) {
    const [donnees, setDonnees] = useState(null);
    const [chargement, setChargement] = useState(false);
    const [erreur, setErreur] = useState(null);

    const fetchDonnees = useCallback(async () => {
        if (!url) return;

        setChargement(true);
        setErreur(null);

        const controller = new AbortController();

        try {
            const reponse = await fetch(url, {
                ...options,
                signal: controller.signal,
            });

            if (!reponse.ok) {
                throw new Error(`Erreur HTTP : ${reponse.status}`);
            }

            const json = await reponse.json();
            setDonnees(json);
        } catch (err) {
            if (err.name !== "AbortError") {
                setErreur(err.message);
            }
        } finally {
            setChargement(false);
        }

        return () => controller.abort();
    }, [url]);

    useEffect(() => {
        const cleanup = fetchDonnees();
        return () => cleanup?.();
    }, [fetchDonnees]);

    return { donnees, chargement, erreur, recharger: fetchDonnees };
}

// ─── Utilisation ───
function ListeArticles() {
    const { donnees, chargement, erreur, recharger } = useFetch(
        "https://jsonplaceholder.typicode.com/posts?_limit=5"
    );

    if (chargement) return <p>Chargement...</p>;
    if (erreur) return <p>Erreur : {erreur} <button onClick={recharger}>Reessayer</button></p>;

    return (
        <ul>
            {donnees?.map(article => (
                <li key={article.id}>{article.title}</li>
            ))}
        </ul>
    );
}
```

### Custom hook : useForm

```jsx
// ─── hooks/useForm.js ───
import { useState } from "react";

function useForm(champsInitiaux, onSoumettre) {
    const [valeurs, setValeurs] = useState(champsInitiaux);
    const [erreurs, setErreurs] = useState({});
    const [soumission, setSoumission] = useState(false);

    function handleChanger(e) {
        const { name, value, type, checked } = e.target;
        setValeurs(prev => ({
            ...prev,
            [name]: type === "checkbox" ? checked : value,
        }));
        // Effacer l'erreur du champ quand l'utilisateur commence a taper
        if (erreurs[name]) {
            setErreurs(prev => ({ ...prev, [name]: "" }));
        }
    }

    async function handleSoumettre(e) {
        e.preventDefault();
        setSoumission(true);
        try {
            await onSoumettre(valeurs);
        } catch (err) {
            setErreurs(err.champsErreurs || {});
        } finally {
            setSoumission(false);
        }
    }

    function reinitialiser() {
        setValeurs(champsInitiaux);
        setErreurs({});
    }

    return {
        valeurs,
        erreurs,
        soumission,
        handleChanger,
        handleSoumettre,
        reinitialiser,
    };
}

// ─── Utilisation ───
function FormulaireContact() {
    const { valeurs, erreurs, soumission, handleChanger, handleSoumettre } = useForm(
        { nom: "", email: "", message: "" },
        async (data) => {
            await new Promise(r => setTimeout(r, 1000)); // Simuler API
            console.log("Envoye :", data);
        }
    );

    return (
        <form onSubmit={handleSoumettre}>
            <input name="nom" value={valeurs.nom} onChange={handleChanger} />
            {erreurs.nom && <span>{erreurs.nom}</span>}
            <textarea name="message" value={valeurs.message} onChange={handleChanger} />
            <button type="submit" disabled={soumission}>
                {soumission ? "Envoi..." : "Envoyer"}
            </button>
        </form>
    );
}
```

---

## 7. Context API — Etat Global sans Redux

### Le probleme du prop drilling

```
Probleme : transmettre un etat profondement imbrique

App (state: utilisateur)
 └── Layout (props: utilisateur)          ← Ne l'utilise pas, le passe
      └── Sidebar (props: utilisateur)    ← Ne l'utilise pas, le passe
           └── Menu (props: utilisateur)  ← Ne l'utilise pas, le passe
                └── Avatar (props: utilisateur) ← L'utilise enfin !

Solution : Context API
App (Context.Provider: utilisateur)
 └── Layout
      └── Sidebar
           └── Menu
                └── Avatar (useContext) ← Accede directement, sans passer par les parents
```

### Creer et utiliser un contexte

```jsx
// ─── src/contexts/ThemeContext.jsx ───
import { createContext, useContext, useState } from "react";

// 1. Creer le contexte avec une valeur par defaut
const ThemeContext = createContext({
    theme: "clair",
    basculerTheme: () => {},
});

// 2. Creer le Provider (composant qui fournit la valeur)
export function ThemeProvider({ children }) {
    const [theme, setTheme] = useState("clair");

    function basculerTheme() {
        setTheme(prev => prev === "clair" ? "sombre" : "clair");
    }

    // La valeur peut contenir state ET fonctions
    const valeur = { theme, basculerTheme };

    return (
        <ThemeContext.Provider value={valeur}>
            {children}
        </ThemeContext.Provider>
    );
}

// 3. Creer un hook personnalise pour consommer le contexte (bonne pratique)
export function useTheme() {
    const contexte = useContext(ThemeContext);
    if (!contexte) {
        throw new Error("useTheme doit etre utilise dans ThemeProvider");
    }
    return contexte;
}
```

```jsx
// ─── src/main.jsx : envelopper l'app dans le Provider ───
import { ThemeProvider } from "./contexts/ThemeContext";

createRoot(document.getElementById("root")).render(
    <ThemeProvider>
        <App />
    </ThemeProvider>
);
```

```jsx
// ─── Dans n'importe quel composant enfant ───
import { useTheme } from "../contexts/ThemeContext";

function BoutonTheme() {
    const { theme, basculerTheme } = useTheme();

    return (
        <button onClick={basculerTheme}>
            Mode {theme === "clair" ? "sombre" : "clair"}
        </button>
    );
}

function Page() {
    const { theme } = useTheme();

    return (
        <div className={`page page--${theme}`}>
            <BoutonTheme />
            <p>Theme actuel : {theme}</p>
        </div>
    );
}
```

### Exemple complet : contexte d'authentification

```jsx
// ─── src/contexts/AuthContext.jsx ───
import { createContext, useContext, useState, useEffect } from "react";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
    const [utilisateur, setUtilisateur] = useState(null);
    const [chargement, setChargement] = useState(true);

    // Verifier la session au demarrage
    useEffect(() => {
        const token = localStorage.getItem("auth_token");
        if (token) {
            // Valider le token avec le serveur
            fetch("/api/me", { headers: { Authorization: `Bearer ${token}` } })
                .then(r => r.json())
                .then(user => setUtilisateur(user))
                .catch(() => localStorage.removeItem("auth_token"))
                .finally(() => setChargement(false));
        } else {
            setChargement(false);
        }
    }, []);

    async function connecter(email, motDePasse) {
        const reponse = await fetch("/api/login", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ email, motDePasse }),
        });

        if (!reponse.ok) throw new Error("Identifiants incorrects");

        const { utilisateur, token } = await reponse.json();
        localStorage.setItem("auth_token", token);
        setUtilisateur(utilisateur);
    }

    function deconnecter() {
        localStorage.removeItem("auth_token");
        setUtilisateur(null);
    }

    return (
        <AuthContext.Provider value={{ utilisateur, chargement, connecter, deconnecter }}>
            {children}
        </AuthContext.Provider>
    );
}

export const useAuth = () => {
    const ctx = useContext(AuthContext);
    if (!ctx) throw new Error("useAuth doit etre utilise dans AuthProvider");
    return ctx;
};
```

> [!info] Context ≠ Solution universelle
> Le contexte React re-rend TOUS les composants abonnes quand sa valeur change. Pour un etat qui change frequemment (ex: position de la souris), cela peut etre un probleme de performance. Pour un etat stable (theme, utilisateur connecte, langue), le contexte est parfait.

---

## 8. Introduction a Redux Toolkit

### Quand utiliser Redux ?

```
Context API suffit pour :            Redux devient utile pour :
──────────────────────               ───────────────────────
Utilisateur connecte                 Etat tres complexe (100+ actions)
Theme / langue                       Logique metier elaborate
Donnees de config                    Debug avance avec time-travel
Partage simple entre composants      Collaboration en grande equipe
                                     Tests unitaires de la logique
```

### Concepts fondamentaux Redux Toolkit

```
┌─────────────────────────────────────────────────────────┐
│                     STORE                                │
│  (source de verite unique pour tout l'etat global)      │
│                                                          │
│  state = {                                               │
│    panier: { items: [], total: 0 },                      │
│    auth: { utilisateur: null, token: null },             │
│    ui: { theme: "clair", sidebarOuverte: false }         │
│  }                                                       │
│                                                          │
│   Dispatcher une ACTION                                  │
│   { type: "panier/ajouter", payload: { produit, qte } } │
│              │                                           │
│              v                                           │
│   REDUCER traite l'action → nouveau state               │
│   (fonction pure : (state, action) => newState)         │
└─────────────────────────────────────────────────────────┘
```

### Mise en place avec Redux Toolkit

```bash
npm install @reduxjs/toolkit react-redux
```

```jsx
// ─── src/store/panierSlice.js ───
import { createSlice } from "@reduxjs/toolkit";

const panierSlice = createSlice({
    name: "panier",
    initialState: {
        items: [],
        total: 0,
    },
    // reducers : definit les actions ET leur logique
    // Redux Toolkit utilise Immer → on peut "muter" directement
    reducers: {
        ajouterAuPanier(state, action) {
            const { produit, quantite } = action.payload;
            const existant = state.items.find(i => i.id === produit.id);

            if (existant) {
                existant.quantite += quantite;
            } else {
                state.items.push({ ...produit, quantite });
            }

            state.total = state.items.reduce(
                (sum, item) => sum + item.prix * item.quantite,
                0
            );
        },

        retirerDuPanier(state, action) {
            state.items = state.items.filter(i => i.id !== action.payload.id);
            state.total = state.items.reduce(
                (sum, item) => sum + item.prix * item.quantite,
                0
            );
        },

        viderPanier(state) {
            state.items = [];
            state.total = 0;
        },
    },
});

// Les action creators sont generes automatiquement
export const { ajouterAuPanier, retirerDuPanier, viderPanier } = panierSlice.actions;
export default panierSlice.reducer;
```

```jsx
// ─── src/store/index.js ───
import { configureStore } from "@reduxjs/toolkit";
import panierReducer from "./panierSlice";
import authReducer from "./authSlice";

export const store = configureStore({
    reducer: {
        panier: panierReducer,
        auth: authReducer,
    },
    // Redux DevTools est active automatiquement en developpement
});
```

```jsx
// ─── src/main.jsx : Provider Redux ───
import { Provider } from "react-redux";
import { store } from "./store";

createRoot(document.getElementById("root")).render(
    <Provider store={store}>
        <App />
    </Provider>
);
```

```jsx
// ─── Utilisation dans un composant ───
import { useSelector, useDispatch } from "react-redux";
import { ajouterAuPanier, viderPanier } from "../store/panierSlice";

function BoutonAjouterPanier({ produit }) {
    const dispatch = useDispatch();

    function handleAjouter() {
        dispatch(ajouterAuPanier({ produit, quantite: 1 }));
    }

    return <button onClick={handleAjouter}>Ajouter au panier</button>;
}

function RecapPanier() {
    // useSelector : lire une partie du store
    const { items, total } = useSelector(state => state.panier);
    const dispatch = useDispatch();

    return (
        <div>
            <h3>Panier ({items.length} articles)</h3>
            <p>Total : {total.toFixed(2)} €</p>
            {items.map(item => (
                <div key={item.id}>
                    {item.nom} x {item.quantite} = {(item.prix * item.quantite).toFixed(2)} €
                </div>
            ))}
            <button onClick={() => dispatch(viderPanier())}>Vider</button>
        </div>
    );
}
```

---

## Carte Mentale ASCII

```
                    HOOKS ET GESTION D'ETAT
                              │
      ┌───────────┬───────────┼──────────────┬────────────┐
      │           │           │              │            │
  useState    useEffect    useRef       useCallback    useMemo
      │           │           │              │            │
  Scalaire    3 modes     DOM ref       Memoriser     Memoriser
  Objet       []          Valeur         fonction      valeur
  Tableau     [deps]      persistante    (pour memo)   (calcul couteux)
  Lazy init   cleanup     Sans render
      │
  Forme fonct.
  prev => prev + 1
      │
      ├─── CUSTOM HOOKS ───
      │    useLocalStorage
      │    useFetch
      │    useForm
      │    useWindowWidth
      │
      ├─── CONTEXT API ───
      │    createContext
      │    Provider
      │    useContext
      │    Custom hook (useXxx)
      │
      └─── REDUX TOOLKIT ───
           Store unique
           Slice (state + reducers)
           Action creators
           useSelector
           useDispatch
```

---

## Exercices

### Exercice 1 : Panier d'achat avec useState

Creez une application de panier avec :
- Une liste de produits (au moins 6, avec id, nom, prix, image)
- Bouton "Ajouter au panier" sur chaque produit
- Le panier affiche les articles, leurs quantites et le total
- Possibilite d'incrementer/decrementer la quantite dans le panier
- Suppression d'un article du panier
- Badge avec le nombre d'articles sur le bouton "Panier"

### Exercice 2 : Fetch avec useEffect et gestion d'erreur

Creez un composant `ExplorateurAPI` qui :
- Charge des donnees depuis `https://jsonplaceholder.typicode.com/posts`
- Affiche un spinner pendant le chargement
- Gere les erreurs reseau avec un message et un bouton "Reessayer"
- Permet de naviguer entre les pages (pagination)
- Au clic sur un article, charge ses commentaires (`/posts/{id}/comments`) et les affiche
- Gere correctement les race conditions et le cleanup

### Exercice 3 : Custom hook useDebounce + recherche

Implementez un custom hook `useDebounce(valeur, delai)` :
- Retourne la valeur debounced (mise a jour uniquement apres `delai` ms d'inactivite)
- Creez un composant de recherche qui utilise ce hook
- La recherche se fait sur une liste locale de 50+ elements
- Afficher combien de renders sont evites grace au debounce

### Exercice 4 : Context multi-theme avec persistance

Creez un systeme de theme complet avec Context :
- 3 themes : "clair", "sombre", "bleu"
- Persistance dans localStorage (meme theme apres reload)
- Le theme s'applique via des classes CSS sur le `<body>`
- Un composant `SelecteurTheme` accessible depuis n'importe ou dans l'arbre
- Les preferences incluent aussi une taille de police (petite, normale, grande)
- Context separe pour le theme et pour les preferences utilisateur

---

## Liens

- [[01 - Introduction a React]]
- [[03 - React Router et Formulaires]]
- [[04 - React Avance et Tests]]
- [[05 - SPA et Frameworks Introduction]]
