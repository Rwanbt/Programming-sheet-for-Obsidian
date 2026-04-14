# Projet JavaScript Interactif

Ce cours est un projet complet : construire une application TODO en JavaScript vanilla (sans framework). L'objectif est de mettre en pratique tous les concepts vus dans les cours precedents (DOM, evenements, ES6+, asynchrone) en creant une application reelle avec une architecture propre.

L'application inclut : ajout/edition/suppression de taches, marquage comme complete, filtrage, persistance avec localStorage, recherche, dark mode, animations et drag & drop.

> [!tip] Analogie
> Construire ce projet, c'est comme assembler un meuble en kit. Vous avez deja appris a utiliser chaque outil individuellement (scie = DOM, marteau = evenements, vis = ES6+). Maintenant, vous allez combiner ces outils pour construire quelque chose de concret et fonctionnel. Le plan de montage, c'est l'architecture MVC.

---

## 1. Structure HTML (Semantique + BEM)

### Nomenclature BEM (Block Element Modifier)

```
BEM = Block__Element--Modifier

Block   : composant autonome       (.todo-app, .task-list, .filter-bar)
Element : partie d'un block         (.task-list__item, .task-list__checkbox)
Modifier: variation d'un block/elem (.task-list__item--completed, .btn--primary)

Exemples :
──────────
.todo-app                    ← Block
.todo-app__header            ← Element du block todo-app
.todo-app__title             ← Element
.task-list                   ← Block
.task-list__item             ← Element
.task-list__item--completed  ← Modifier (variante "completed")
.task-list__item--editing    ← Modifier (variante "editing")
.btn                         ← Block
.btn--primary                ← Modifier
.btn--danger                 ← Modifier
```

### Structure HTML complete

```html
<!DOCTYPE html>
<html lang="fr" data-theme="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Todo App</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="todo-app">
        <!-- En-tete -->
        <header class="todo-app__header">
            <h1 class="todo-app__title">Mes Taches</h1>
            <button class="todo-app__theme-toggle" id="themeToggle"
                    aria-label="Basculer le theme">
                <span class="todo-app__theme-icon">🌙</span>
            </button>
        </header>

        <!-- Formulaire d'ajout -->
        <form class="todo-form" id="todoForm">
            <input
                type="text"
                class="todo-form__input"
                id="todoInput"
                placeholder="Ajouter une tache..."
                aria-label="Nouvelle tache"
                autocomplete="off"
                required
            />
            <button type="submit" class="todo-form__btn btn btn--primary">
                Ajouter
            </button>
        </form>

        <!-- Barre de recherche et filtres -->
        <div class="filter-bar">
            <input
                type="search"
                class="filter-bar__search"
                id="searchInput"
                placeholder="Rechercher..."
                aria-label="Rechercher une tache"
            />
            <div class="filter-bar__filters" role="radiogroup"
                 aria-label="Filtrer les taches">
                <button class="filter-bar__btn filter-bar__btn--active"
                        data-filter="all" aria-pressed="true">
                    Toutes <span class="filter-bar__count" id="countAll">0</span>
                </button>
                <button class="filter-bar__btn"
                        data-filter="active" aria-pressed="false">
                    Actives <span class="filter-bar__count" id="countActive">0</span>
                </button>
                <button class="filter-bar__btn"
                        data-filter="completed" aria-pressed="false">
                    Terminees <span class="filter-bar__count" id="countCompleted">0</span>
                </button>
            </div>
        </div>

        <!-- Liste des taches -->
        <ul class="task-list" id="taskList" aria-live="polite">
            <!-- Les taches seront injectees ici par JavaScript -->
        </ul>

        <!-- Pied de page -->
        <footer class="todo-app__footer">
            <span id="summary">0 tache(s)</span>
            <button class="btn btn--danger btn--small" id="clearCompleted">
                Supprimer terminees
            </button>
        </footer>
    </div>

    <!-- Scripts (modules ES6) -->
    <script type="module" src="js/app.js"></script>
</body>
</html>
```

> [!info] Attributs d'accessibilite
> `aria-label` fournit un libelle aux lecteurs d'ecran. `aria-live="polite"` annonce les changements dynamiques. `aria-pressed` indique l'etat d'un bouton bascule. `role="radiogroup"` signale un groupe de choix exclusifs.

---

## 2. CSS Moderne et Responsive

### Variables CSS et theme sombre

```css
/* ─── Variables (Custom Properties) ─── */
:root {
    /* Theme clair (defaut) */
    --color-bg: #f5f5f5;
    --color-surface: #ffffff;
    --color-text: #333333;
    --color-text-secondary: #666666;
    --color-primary: #4a90d9;
    --color-primary-hover: #357abd;
    --color-danger: #e74c3c;
    --color-success: #27ae60;
    --color-border: #e0e0e0;
    --color-shadow: rgba(0, 0, 0, 0.1);

    --radius: 8px;
    --transition: 0.3s ease;
    --max-width: 600px;
}

/* Theme sombre */
[data-theme="dark"] {
    --color-bg: #1a1a2e;
    --color-surface: #16213e;
    --color-text: #e0e0e0;
    --color-text-secondary: #a0a0a0;
    --color-primary: #5dade2;
    --color-primary-hover: #3498db;
    --color-danger: #e74c3c;
    --color-success: #2ecc71;
    --color-border: #2c3e6b;
    --color-shadow: rgba(0, 0, 0, 0.3);
}
```

### Layout principal

```css
/* ─── Reset et base ─── */
*,
*::before,
*::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    background-color: var(--color-bg);
    color: var(--color-text);
    transition: background-color var(--transition), color var(--transition);
    min-height: 100vh;
    display: flex;
    justify-content: center;
    padding: 2rem 1rem;
}

/* ─── Application ─── */
.todo-app {
    width: 100%;
    max-width: var(--max-width);
}

.todo-app__header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1.5rem;
}

.todo-app__title {
    font-size: 2rem;
    font-weight: 700;
}

.todo-app__theme-toggle {
    background: none;
    border: 2px solid var(--color-border);
    border-radius: 50%;
    width: 40px;
    height: 40px;
    cursor: pointer;
    font-size: 1.2rem;
    transition: transform var(--transition);
}

.todo-app__theme-toggle:hover {
    transform: rotate(30deg);
}
```

### Formulaire et filtres

```css
/* ─── Formulaire ─── */
.todo-form {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
}

.todo-form__input {
    flex: 1;
    padding: 0.75rem 1rem;
    border: 2px solid var(--color-border);
    border-radius: var(--radius);
    background: var(--color-surface);
    color: var(--color-text);
    font-size: 1rem;
    transition: border-color var(--transition);
}

.todo-form__input:focus {
    outline: none;
    border-color: var(--color-primary);
}

/* ─── Boutons ─── */
.btn {
    padding: 0.75rem 1.25rem;
    border: none;
    border-radius: var(--radius);
    font-size: 0.9rem;
    font-weight: 600;
    cursor: pointer;
    transition: background-color var(--transition), transform 0.1s;
}

.btn:active {
    transform: scale(0.97);
}

.btn--primary {
    background-color: var(--color-primary);
    color: white;
}

.btn--primary:hover {
    background-color: var(--color-primary-hover);
}

.btn--danger {
    background-color: var(--color-danger);
    color: white;
}

.btn--small {
    padding: 0.4rem 0.8rem;
    font-size: 0.8rem;
}

/* ─── Filtres ─── */
.filter-bar {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    margin-bottom: 1rem;
}

.filter-bar__search {
    padding: 0.6rem 1rem;
    border: 2px solid var(--color-border);
    border-radius: var(--radius);
    background: var(--color-surface);
    color: var(--color-text);
    font-size: 0.9rem;
}

.filter-bar__filters {
    display: flex;
    gap: 0.25rem;
}

.filter-bar__btn {
    flex: 1;
    padding: 0.5rem;
    border: 2px solid var(--color-border);
    border-radius: var(--radius);
    background: var(--color-surface);
    color: var(--color-text-secondary);
    cursor: pointer;
    font-size: 0.85rem;
    transition: all var(--transition);
}

.filter-bar__btn--active {
    border-color: var(--color-primary);
    color: var(--color-primary);
    font-weight: 600;
}
```

### Style des taches et animations

```css
/* ─── Liste des taches ─── */
.task-list {
    list-style: none;
}

.task-list__item {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 0.75rem 1rem;
    margin-bottom: 0.5rem;
    background: var(--color-surface);
    border: 2px solid var(--color-border);
    border-radius: var(--radius);
    transition: all var(--transition);
    /* Animation d'entree */
    animation: slideIn 0.3s ease forwards;
}

.task-list__item:hover {
    border-color: var(--color-primary);
    box-shadow: 0 2px 8px var(--color-shadow);
}

.task-list__item--completed .task-list__text {
    text-decoration: line-through;
    color: var(--color-text-secondary);
}

.task-list__item--editing {
    border-color: var(--color-primary);
    background: var(--color-bg);
}

/* Drag & drop */
.task-list__item--dragging {
    opacity: 0.5;
    border-style: dashed;
}

.task-list__item--drag-over {
    border-color: var(--color-success);
    transform: scale(1.02);
}

.task-list__checkbox {
    width: 20px;
    height: 20px;
    accent-color: var(--color-success);
    cursor: pointer;
}

.task-list__text {
    flex: 1;
    word-break: break-word;
}

.task-list__edit-input {
    flex: 1;
    padding: 0.25rem 0.5rem;
    border: 1px solid var(--color-primary);
    border-radius: 4px;
    background: var(--color-surface);
    color: var(--color-text);
    font-size: inherit;
}

.task-list__actions {
    display: flex;
    gap: 0.25rem;
}

.task-list__action-btn {
    background: none;
    border: none;
    cursor: pointer;
    font-size: 1rem;
    padding: 0.25rem;
    border-radius: 4px;
    transition: background-color var(--transition);
}

.task-list__action-btn:hover {
    background-color: var(--color-border);
}

/* ─── Animations ─── */
@keyframes slideIn {
    from {
        opacity: 0;
        transform: translateY(-10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes slideOut {
    from {
        opacity: 1;
        transform: translateX(0);
        max-height: 100px;
    }
    to {
        opacity: 0;
        transform: translateX(100px);
        max-height: 0;
        padding: 0;
        margin: 0;
        border: 0;
    }
}

.task-list__item--removing {
    animation: slideOut 0.3s ease forwards;
}

/* ─── Footer ─── */
.todo-app__footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.75rem 0;
    color: var(--color-text-secondary);
    font-size: 0.9rem;
}

/* ─── Responsive ─── */
@media (max-width: 480px) {
    .todo-app__title {
        font-size: 1.5rem;
    }

    .todo-form {
        flex-direction: column;
    }

    .filter-bar__filters {
        flex-direction: column;
    }
}
```

> [!tip] Analogie
> Les variables CSS (custom properties) fonctionnent comme des constantes globales en programmation. Le dark mode revient a changer les valeurs de ces constantes : tout le style s'adapte automatiquement, comme changer la palette de couleurs d'un theme dans un editeur de code.

---

## 3. Architecture MVC

### Principe MVC (Model-View-Controller)

```
┌─────────────────────────────────────────────────────┐
│                   UTILISATEUR                        │
│              (clique, tape, drag...)                 │
└──────────┬──────────────────────────────┬────────────┘
           │ Evenement                     ^ Affichage
           v                               │
┌──────────────────┐              ┌──────────────────┐
│   CONTROLLER     │              │     VIEW         │
│                  │              │                  │
│ - Ecoute events  │              │ - Render HTML    │
│ - Valide input   │              │ - MAJ DOM        │
│ - Appelle Model  │              │ - Animations     │
│ - MAJ View       │              │ - Event deleg.   │
│                  │              │                  │
└────────┬─────────┘              └──────────────────┘
         │ Lit / Modifie                   ^
         v                                 │ Donnees
┌──────────────────┐                       │
│     MODEL        │───────────────────────┘
│                  │
│ - Donnees (tasks)│
│ - CRUD           │
│ - localStorage   │
│ - Logique metier │
└──────────────────┘

Flux :
1. L'utilisateur clique "Ajouter"
2. Le CONTROLLER intercepte l'evenement
3. Le CONTROLLER demande au MODEL d'ajouter la tache
4. Le MODEL ajoute la tache et sauvegarde dans localStorage
5. Le CONTROLLER demande a la VIEW de re-afficher la liste
6. La VIEW met a jour le DOM
```

### Organisation des fichiers

```
todo-app/
├── index.html
├── styles.css
└── js/
    ├── app.js          ← Point d'entree, initialisation
    ├── model.js        ← Donnees et logique metier
    ├── view.js         ← Manipulation du DOM
    ├── controller.js   ← Gestion des evenements
    └── helpers.js      ← Fonctions utilitaires
```

> [!info] MVC en vanilla JS
> Les frameworks comme React ou Vue implementent leurs propres variantes de ce pattern (MVVM, Flux, etc.). Comprendre MVC en vanilla JS vous aidera a comprendre pourquoi ces frameworks existent et comment ils simplifient le travail.

---

## 4. Implementer le Model

Le Model gere les donnees, la logique metier et la persistance.

```javascript
// ─── js/model.js ───

/**
 * Genere un identifiant unique simple
 * (En production, utiliser crypto.randomUUID())
 */
function genererID() {
    return Date.now().toString(36) + Math.random().toString(36).slice(2, 7);
}

/**
 * Model : gere les taches et la persistance localStorage
 */
export class TodoModel {
    #tasks = [];
    #storageKey = "todo-app-tasks";
    #listeners = [];

    constructor() {
        this.#chargerDepuisStorage();
    }

    // ─── Lecture ───

    get tasks() {
        return [...this.#tasks]; // Copie pour eviter la mutation externe
    }

    getTache(id) {
        return this.#tasks.find(t => t.id === id);
    }

    filtrer(filtre, recherche = "") {
        let resultat = this.#tasks;

        // Filtre par statut
        if (filtre === "active") {
            resultat = resultat.filter(t => !t.completed);
        } else if (filtre === "completed") {
            resultat = resultat.filter(t => t.completed);
        }

        // Filtre par recherche (insensible a la casse)
        if (recherche.trim()) {
            const terme = recherche.toLowerCase().trim();
            resultat = resultat.filter(t =>
                t.text.toLowerCase().includes(terme)
            );
        }

        return resultat;
    }

    get compteurs() {
        const total = this.#tasks.length;
        const completed = this.#tasks.filter(t => t.completed).length;
        return {
            all: total,
            active: total - completed,
            completed
        };
    }

    // ─── Creation ───

    ajouter(texte) {
        const texteNettoye = texte.trim();
        if (!texteNettoye) {
            throw new Error("Le texte de la tache ne peut pas etre vide.");
        }
        if (texteNettoye.length > 200) {
            throw new Error("La tache ne peut pas depasser 200 caracteres.");
        }

        const tache = {
            id: genererID(),
            text: texteNettoye,
            completed: false,
            createdAt: new Date().toISOString(),
            order: this.#tasks.length
        };

        this.#tasks.push(tache);
        this.#sauvegarder();
        this.#notifier("add", tache);
        return tache;
    }

    // ─── Modification ───

    modifier(id, nouveauTexte) {
        const tache = this.getTache(id);
        if (!tache) throw new Error(`Tache ${id} introuvable.`);

        const texteNettoye = nouveauTexte.trim();
        if (!texteNettoye) throw new Error("Le texte ne peut pas etre vide.");

        tache.text = texteNettoye;
        tache.updatedAt = new Date().toISOString();
        this.#sauvegarder();
        this.#notifier("update", tache);
        return tache;
    }

    basculerStatut(id) {
        const tache = this.getTache(id);
        if (!tache) throw new Error(`Tache ${id} introuvable.`);

        tache.completed = !tache.completed;
        this.#sauvegarder();
        this.#notifier("toggle", tache);
        return tache;
    }

    // ─── Suppression ───

    supprimer(id) {
        const index = this.#tasks.findIndex(t => t.id === id);
        if (index === -1) throw new Error(`Tache ${id} introuvable.`);

        const [supprimee] = this.#tasks.splice(index, 1);
        this.#sauvegarder();
        this.#notifier("delete", supprimee);
        return supprimee;
    }

    supprimerTerminees() {
        const avant = this.#tasks.length;
        this.#tasks = this.#tasks.filter(t => !t.completed);
        this.#sauvegarder();
        this.#notifier("clearCompleted", { count: avant - this.#tasks.length });
    }

    // ─── Reordonner (pour le drag & drop) ───

    reordonner(idSource, idCible) {
        const indexSource = this.#tasks.findIndex(t => t.id === idSource);
        const indexCible = this.#tasks.findIndex(t => t.id === idCible);

        if (indexSource === -1 || indexCible === -1) return;

        const [tache] = this.#tasks.splice(indexSource, 1);
        this.#tasks.splice(indexCible, 0, tache);

        // Mettre a jour les ordres
        this.#tasks.forEach((t, i) => t.order = i);
        this.#sauvegarder();
        this.#notifier("reorder", null);
    }

    // ─── Observateur (pattern Observer) ───

    onChangement(callback) {
        this.#listeners.push(callback);
        // Retourner une fonction de desabonnement
        return () => {
            this.#listeners = this.#listeners.filter(l => l !== callback);
        };
    }

    #notifier(action, data) {
        this.#listeners.forEach(cb => cb(action, data));
    }

    // ─── Persistance localStorage ───

    #sauvegarder() {
        try {
            localStorage.setItem(this.#storageKey, JSON.stringify(this.#tasks));
        } catch (e) {
            console.error("Erreur de sauvegarde localStorage :", e.message);
            // localStorage peut etre plein ou desactive
        }
    }

    #chargerDepuisStorage() {
        try {
            const donnees = localStorage.getItem(this.#storageKey);
            if (donnees) {
                this.#tasks = JSON.parse(donnees);
                // Validation basique des donnees chargees
                this.#tasks = this.#tasks.filter(t =>
                    t && typeof t.id === "string" && typeof t.text === "string"
                );
            }
        } catch (e) {
            console.error("Erreur de chargement localStorage :", e.message);
            this.#tasks = [];
        }
    }
}
```

> [!warning] Validation des donnees localStorage
> Les donnees dans localStorage peuvent etre corrompues (modification manuelle, bug precedent). Il est essentiel de valider les donnees apres le `JSON.parse()` et de gerer les erreurs gracieusement.

---

## 5. Implementer la View

La View est responsable de toute la manipulation du DOM.

```javascript
// ─── js/view.js ───

/**
 * View : gere l'affichage et la manipulation du DOM
 * Ne contient AUCUNE logique metier
 */
export class TodoView {
    constructor() {
        // ─── Cache des elements DOM ───
        this.form = document.getElementById("todoForm");
        this.input = document.getElementById("todoInput");
        this.searchInput = document.getElementById("searchInput");
        this.taskList = document.getElementById("taskList");
        this.themeToggle = document.getElementById("themeToggle");
        this.clearCompletedBtn = document.getElementById("clearCompleted");
        this.summary = document.getElementById("summary");
        this.countAll = document.getElementById("countAll");
        this.countActive = document.getElementById("countActive");
        this.countCompleted = document.getElementById("countCompleted");
        this.filterBtns = document.querySelectorAll("[data-filter]");
    }

    // ─── Rendu de la liste ───

    afficherTaches(taches) {
        // Vider la liste
        this.taskList.innerHTML = "";

        if (taches.length === 0) {
            this.taskList.innerHTML = `
                <li class="task-list__empty">
                    Aucune tache a afficher.
                </li>
            `;
            return;
        }

        // Utiliser un DocumentFragment pour la performance
        const fragment = document.createDocumentFragment();

        taches.forEach(tache => {
            const li = this.#creerElementTache(tache);
            fragment.appendChild(li);
        });

        this.taskList.appendChild(fragment);
    }

    #creerElementTache(tache) {
        const li = document.createElement("li");
        li.className = `task-list__item${tache.completed ? " task-list__item--completed" : ""}`;
        li.dataset.id = tache.id;
        li.draggable = true;

        li.innerHTML = `
            <input
                type="checkbox"
                class="task-list__checkbox"
                ${tache.completed ? "checked" : ""}
                aria-label="Marquer comme ${tache.completed ? "active" : "terminee"}"
            />
            <span class="task-list__text">${this.#echapper(tache.text)}</span>
            <div class="task-list__actions">
                <button class="task-list__action-btn" data-action="edit"
                        aria-label="Modifier la tache">
                    ✏️
                </button>
                <button class="task-list__action-btn" data-action="delete"
                        aria-label="Supprimer la tache">
                    🗑️
                </button>
            </div>
        `;

        return li;
    }

    // ─── Mode edition ───

    activerEdition(id, texteActuel) {
        const li = this.taskList.querySelector(`[data-id="${id}"]`);
        if (!li) return;

        li.classList.add("task-list__item--editing");
        const textSpan = li.querySelector(".task-list__text");
        const actions = li.querySelector(".task-list__actions");

        // Remplacer le texte par un input
        const editInput = document.createElement("input");
        editInput.type = "text";
        editInput.className = "task-list__edit-input";
        editInput.value = texteActuel;

        textSpan.replaceWith(editInput);
        actions.style.display = "none";
        editInput.focus();
        editInput.select();

        return editInput;
    }

    desactiverEdition(id, nouveauTexte) {
        const li = this.taskList.querySelector(`[data-id="${id}"]`);
        if (!li) return;

        li.classList.remove("task-list__item--editing");
        const editInput = li.querySelector(".task-list__edit-input");
        const actions = li.querySelector(".task-list__actions");

        if (editInput) {
            const span = document.createElement("span");
            span.className = "task-list__text";
            span.textContent = nouveauTexte;
            editInput.replaceWith(span);
        }

        if (actions) {
            actions.style.display = "flex";
        }
    }

    // ─── Animation de suppression ───

    animerSuppression(id) {
        return new Promise(resolve => {
            const li = this.taskList.querySelector(`[data-id="${id}"]`);
            if (!li) {
                resolve();
                return;
            }

            li.classList.add("task-list__item--removing");
            li.addEventListener("animationend", () => {
                li.remove();
                resolve();
            }, { once: true });
        });
    }

    // ─── Mise a jour des compteurs ───

    mettreAJourCompteurs(compteurs) {
        this.countAll.textContent = compteurs.all;
        this.countActive.textContent = compteurs.active;
        this.countCompleted.textContent = compteurs.completed;

        const total = compteurs.all;
        const mot = total <= 1 ? "tache" : "taches";
        this.summary.textContent = `${total} ${mot}`;
    }

    // ─── Filtre actif ───

    setFiltreActif(filtre) {
        this.filterBtns.forEach(btn => {
            const estActif = btn.dataset.filter === filtre;
            btn.classList.toggle("filter-bar__btn--active", estActif);
            btn.setAttribute("aria-pressed", estActif);
        });
    }

    // ─── Theme ───

    setTheme(theme) {
        document.documentElement.setAttribute("data-theme", theme);
        const icon = this.themeToggle.querySelector(".todo-app__theme-icon");
        icon.textContent = theme === "dark" ? "☀️" : "🌙";
    }

    // ─── Utilitaires ───

    getInputValue() {
        return this.input.value;
    }

    clearInput() {
        this.input.value = "";
        this.input.focus();
    }

    getSearchValue() {
        return this.searchInput.value;
    }

    /**
     * Echappe le HTML pour eviter les injections XSS
     * ESSENTIEL : ne jamais inserer du texte utilisateur tel quel dans innerHTML
     */
    #echapper(texte) {
        const div = document.createElement("div");
        div.textContent = texte;
        return div.innerHTML;
    }
}
```

> [!warning] Protection contre les injections XSS
> La methode `#echapper()` est critique. Sans elle, un utilisateur pourrait entrer `<script>alert("hack")</script>` comme texte de tache, et le code serait execute. Toujours echapper le contenu utilisateur avant de l'inserer dans `innerHTML`. Alternative plus sure : utiliser `textContent` au lieu de `innerHTML`.

---

## 6. Implementer le Controller

Le Controller fait le lien entre le Model et la View.

```javascript
// ─── js/controller.js ───

/**
 * Controller : gere les evenements et orchestre Model <-> View
 */
export class TodoController {
    #model;
    #view;
    #filtreActuel = "all";

    constructor(model, view) {
        this.#model = model;
        this.#view = view;

        this.#initialiser();
    }

    #initialiser() {
        // Affichage initial
        this.#rafraichir();
        this.#chargerTheme();

        // ─── Evenements du formulaire ───
        this.#view.form.addEventListener("submit", (e) => {
            e.preventDefault();
            this.#ajouterTache();
        });

        // ─── Evenements sur la liste (event delegation) ───
        this.#view.taskList.addEventListener("click", (e) => {
            this.#gererClicListe(e);
        });

        this.#view.taskList.addEventListener("change", (e) => {
            if (e.target.matches(".task-list__checkbox")) {
                const id = e.target.closest("[data-id]").dataset.id;
                this.#basculerTache(id);
            }
        });

        // ─── Recherche (avec debounce) ───
        let timeoutRecherche;
        this.#view.searchInput.addEventListener("input", () => {
            clearTimeout(timeoutRecherche);
            timeoutRecherche = setTimeout(() => this.#rafraichir(), 250);
        });

        // ─── Filtres ───
        this.#view.filterBtns.forEach(btn => {
            btn.addEventListener("click", () => {
                this.#filtreActuel = btn.dataset.filter;
                this.#view.setFiltreActif(this.#filtreActuel);
                this.#rafraichir();
            });
        });

        // ─── Supprimer terminees ───
        this.#view.clearCompletedBtn.addEventListener("click", () => {
            if (this.#model.compteurs.completed === 0) return;
            this.#model.supprimerTerminees();
            this.#rafraichir();
        });

        // ─── Theme ───
        this.#view.themeToggle.addEventListener("click", () => {
            this.#basculerTheme();
        });

        // ─── Drag & Drop ───
        this.#configurerDragDrop();

        // ─── Raccourcis clavier ───
        document.addEventListener("keydown", (e) => {
            // Ctrl+Shift+A : focus sur l'input d'ajout
            if (e.ctrlKey && e.shiftKey && e.key === "A") {
                e.preventDefault();
                this.#view.input.focus();
            }
        });
    }

    // ─── Actions CRUD ───

    #ajouterTache() {
        const texte = this.#view.getInputValue();
        try {
            this.#model.ajouter(texte);
            this.#view.clearInput();
            this.#rafraichir();
        } catch (erreur) {
            alert(erreur.message); // En production : notification plus elegante
        }
    }

    #basculerTache(id) {
        try {
            this.#model.basculerStatut(id);
            this.#rafraichir();
        } catch (erreur) {
            console.error(erreur.message);
        }
    }

    async #supprimerTache(id) {
        try {
            await this.#view.animerSuppression(id);
            this.#model.supprimer(id);
            this.#rafraichir();
        } catch (erreur) {
            console.error(erreur.message);
        }
    }

    #gererClicListe(e) {
        const actionBtn = e.target.closest("[data-action]");
        if (!actionBtn) return;

        const id = actionBtn.closest("[data-id]").dataset.id;
        const action = actionBtn.dataset.action;

        switch (action) {
            case "edit":
                this.#commencerEdition(id);
                break;
            case "delete":
                this.#supprimerTache(id);
                break;
        }
    }

    // ─── Edition ───

    #commencerEdition(id) {
        const tache = this.#model.getTache(id);
        if (!tache) return;

        const editInput = this.#view.activerEdition(id, tache.text);
        if (!editInput) return;

        // Valider avec Entree ou perte de focus
        const valider = () => {
            const nouveauTexte = editInput.value.trim();
            if (nouveauTexte && nouveauTexte !== tache.text) {
                try {
                    this.#model.modifier(id, nouveauTexte);
                } catch (erreur) {
                    alert(erreur.message);
                }
            }
            this.#rafraichir();
        };

        editInput.addEventListener("keydown", (e) => {
            if (e.key === "Enter") {
                e.preventDefault();
                valider();
            } else if (e.key === "Escape") {
                this.#rafraichir(); // Annuler
            }
        });

        editInput.addEventListener("blur", valider, { once: true });
    }

    // ─── Drag & Drop ───

    #configurerDragDrop() {
        let idEnCoursDeDrag = null;

        this.#view.taskList.addEventListener("dragstart", (e) => {
            const item = e.target.closest(".task-list__item");
            if (!item) return;

            idEnCoursDeDrag = item.dataset.id;
            item.classList.add("task-list__item--dragging");
            e.dataTransfer.effectAllowed = "move";
        });

        this.#view.taskList.addEventListener("dragend", (e) => {
            const item = e.target.closest(".task-list__item");
            if (item) item.classList.remove("task-list__item--dragging");
            idEnCoursDeDrag = null;

            // Retirer tous les indicateurs visuels
            this.#view.taskList.querySelectorAll(".task-list__item--drag-over")
                .forEach(el => el.classList.remove("task-list__item--drag-over"));
        });

        this.#view.taskList.addEventListener("dragover", (e) => {
            e.preventDefault();
            e.dataTransfer.dropEffect = "move";

            const cible = e.target.closest(".task-list__item");
            if (!cible || cible.dataset.id === idEnCoursDeDrag) return;

            // Retirer les anciens indicateurs
            this.#view.taskList.querySelectorAll(".task-list__item--drag-over")
                .forEach(el => el.classList.remove("task-list__item--drag-over"));

            cible.classList.add("task-list__item--drag-over");
        });

        this.#view.taskList.addEventListener("drop", (e) => {
            e.preventDefault();
            const cible = e.target.closest(".task-list__item");
            if (!cible || !idEnCoursDeDrag) return;

            cible.classList.remove("task-list__item--drag-over");

            if (cible.dataset.id !== idEnCoursDeDrag) {
                this.#model.reordonner(idEnCoursDeDrag, cible.dataset.id);
                this.#rafraichir();
            }
        });
    }

    // ─── Theme ───

    #basculerTheme() {
        const themeActuel = document.documentElement.getAttribute("data-theme");
        const nouveauTheme = themeActuel === "dark" ? "light" : "dark";
        this.#view.setTheme(nouveauTheme);
        localStorage.setItem("todo-app-theme", nouveauTheme);
    }

    #chargerTheme() {
        const theme = localStorage.getItem("todo-app-theme") || "light";
        this.#view.setTheme(theme);
    }

    // ─── Rafraichissement ───

    #rafraichir() {
        const recherche = this.#view.getSearchValue();
        const tachesFiltrees = this.#model.filtrer(this.#filtreActuel, recherche);
        this.#view.afficherTaches(tachesFiltrees);
        this.#view.mettreAJourCompteurs(this.#model.compteurs);
    }
}
```

> [!info] Event Delegation
> Plutot que d'attacher un listener a chaque bouton de chaque tache (qui change a chaque re-render), on attache UN SEUL listener sur le parent (`taskList`). L'evenement "remonte" (bubbling) et on identifie la cible avec `e.target.closest()`. C'est plus performant et fonctionne meme pour les elements ajoutes dynamiquement.

---

## 7. Point d'Entree (app.js)

```javascript
// ─── js/app.js ───

import { TodoModel } from "./model.js";
import { TodoView } from "./view.js";
import { TodoController } from "./controller.js";

/**
 * Point d'entree de l'application
 * Initialise les 3 couches MVC
 */
function init() {
    const model = new TodoModel();
    const view = new TodoView();
    const controller = new TodoController(model, view);

    // Debug : exposer en dev pour tester dans la console
    if (location.hostname === "localhost") {
        window.__debug = { model, view, controller };
        console.log("Mode dev : window.__debug disponible");
    }
}

// Lancer quand le DOM est pret
if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", init);
} else {
    init();
}
```

---

## 8. Fonctions Utilitaires (helpers.js)

```javascript
// ─── js/helpers.js ───

/**
 * Debounce : retarde l'execution jusqu'a ce que l'utilisateur
 * arrete d'agir pendant `delai` ms
 */
export function debounce(fn, delai = 300) {
    let timer;
    return function (...args) {
        clearTimeout(timer);
        timer = setTimeout(() => fn.apply(this, args), delai);
    };
}

/**
 * Throttle : execute au maximum une fois par `delai` ms
 */
export function throttle(fn, delai = 100) {
    let dernierAppel = 0;
    return function (...args) {
        const maintenant = Date.now();
        if (maintenant - dernierAppel >= delai) {
            dernierAppel = maintenant;
            fn.apply(this, args);
        }
    };
}

/**
 * Formater une date ISO en format lisible
 */
export function formaterDate(dateISO) {
    const date = new Date(dateISO);
    return new Intl.DateTimeFormat("fr-FR", {
        day: "numeric",
        month: "short",
        year: "numeric",
        hour: "2-digit",
        minute: "2-digit"
    }).format(date);
}

/**
 * Generer un ID unique (plus robuste que Date.now)
 */
export function genererUUID() {
    if (crypto.randomUUID) {
        return crypto.randomUUID();
    }
    // Fallback pour anciens navigateurs
    return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, c => {
        const r = (Math.random() * 16) | 0;
        const v = c === "x" ? r : (r & 0x3) | 0x8;
        return v.toString(16);
    });
}
```

---

## 9. Patron de Conception : Observer

Le Model utilise le patron Observer pour notifier les changements.

```
┌─────────────────────────────────────────────────┐
│              Pattern Observer                     │
│                                                   │
│  Subject (Model)          Observers               │
│  ┌──────────────┐         ┌──────────────────┐   │
│  │ #listeners[] │         │ Controller       │   │
│  │              │────────>│   .onChangement()│   │
│  │ #notifier()  │         └──────────────────┘   │
│  │              │         ┌──────────────────┐   │
│  │ onChangement()│───────>│ Logger           │   │
│  │ (subscribe)  │         │   .onChangement()│   │
│  └──────────────┘         └──────────────────┘   │
│                                                   │
│  Usage :                                          │
│  model.onChangement((action, data) => {           │
│      console.log(`Action: ${action}`, data);      │
│  });                                              │
│                                                   │
│  // Retourne un "unsubscribe"                     │
│  const unsub = model.onChangement(fn);            │
│  unsub(); // Se desabonner                        │
└─────────────────────────────────────────────────┘
```

> [!example] Comparaison avec d'autres patterns
> ```
> Observer (ce projet) :    Pub/Sub (events) :     Redux (flux) :
> ──────────────────       ──────────────────     ──────────────
> model.onChangement(fn)   bus.on("add", fn)      store.subscribe(fn)
> model.ajouter(tache)     bus.emit("add", t)     store.dispatch({type:"ADD"})
>   -> fn(action, data)      -> fn(t)               -> reducer -> fn(state)
>
> Direct, simple.          Decouple, flexible.    Predictible, lourd.
> ```

---

## 10. Drag & Drop HTML5

### Principe du Drag & Drop

```
Evenements de Drag & Drop :

Element source :                    Element cible :
────────────────                   ────────────────
dragstart  ── debut du drag         dragenter ── entre dans la zone
drag       ── pendant le drag       dragover  ── survole la zone (*)
dragend    ── fin du drag           dragleave ── quitte la zone
                                    drop      ── lache dans la zone

(*) dragover DOIT appeler e.preventDefault() pour autoriser le drop
    Par defaut, le drop est interdit !
```

```
Flux du drag & drop dans notre app :

1. dragstart sur item A
   → Stocker l'id de A
   → Ajouter classe --dragging

2. dragover sur item B
   → e.preventDefault() (autoriser le drop)
   → Ajouter classe --drag-over sur B

3. drop sur item B
   → model.reordonner(idA, idB)
   → Rafraichir l'affichage

4. dragend
   → Retirer toutes les classes visuelles
```

---

## 11. Gestion des Erreurs

```javascript
// ─── Strategie de gestion d'erreurs ───

// 1. Erreurs de validation (previsibles)
try {
    model.ajouter("");  // Texte vide
} catch (e) {
    // Afficher un message utilisateur (pas une erreur technique)
    afficherNotification(e.message, "warning");
}

// 2. Erreurs de stockage (environnement)
try {
    localStorage.setItem(cle, donnees);
} catch (e) {
    if (e.name === "QuotaExceededError") {
        afficherNotification("Stockage plein. Supprimez des taches.", "error");
    } else {
        console.error("Erreur localStorage :", e);
    }
}

// 3. Erreurs inattendues (global handler)
window.addEventListener("error", (e) => {
    console.error("Erreur globale :", e.error);
    // En production : envoyer a un service de monitoring (Sentry, etc.)
});

window.addEventListener("unhandledrejection", (e) => {
    console.error("Promise rejetee non geree :", e.reason);
});
```

> [!warning] Ne jamais ignorer les erreurs silencieusement
> Un `catch` vide (`catch (e) {}`) est dangereux. Au minimum, loggez l'erreur. En production, envoyez-la a un service de monitoring. Un bug silencieux est bien plus difficile a diagnostiquer qu'un bug visible.

---

## 12. Bonnes Pratiques d'Organisation

### Separation des responsabilites

```
BON :                                MAUVAIS :
─────                                ─────────
model.js  → Donnees uniquement       app.js (tout en un seul fichier)
view.js   → DOM uniquement             → Donnees
controller.js → Logique uniquement      → DOM
helpers.js → Utilitaires reutilisables  → Evenements
                                        → localStorage
                                        → 500+ lignes...
```

### Conventions de nommage

```
Fichiers :     camelCase.js          (todoModel.js)
Classes :      PascalCase            (TodoModel, TodoView)
Methodes :     camelCase             (ajouterTache, filtrer)
Methodes # :   camelCase prive       (#sauvegarder, #notifier)
Constantes :   SCREAMING_SNAKE       (MAX_LENGTH, STORAGE_KEY)
Events :       camelCase             (onClick, onSubmit)
CSS BEM :      block__element--mod   (.task-list__item--completed)
Data attrs :   kebab-case            (data-filter, data-action)
```

### Checklist qualite

```
[ ] Pas de variables globales (tout est encapsule dans des classes/modules)
[ ] Pas de innerHTML avec du contenu utilisateur non echappe
[ ] Gestion des erreurs sur toutes les operations critiques
[ ] Event delegation plutot qu'un listener par element
[ ] Debounce sur la recherche
[ ] localStorage : try/catch + validation des donnees chargees
[ ] Accessibilite : aria-labels, focus management, semantique HTML
[ ] Responsive : testé sur mobile
[ ] Performance : DocumentFragment pour les rendus en lot
[ ] Code lisible : noms explicites, commentaires sur le "pourquoi"
```

---

## 13. Recapitulatif de l'Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        index.html                             │
│  <form> <input> <ul#taskList> <button>                       │
└──────────┬───────────────────────────────────────────────────┘
           │ <script type="module" src="js/app.js">
           v
┌──────────────────┐
│     app.js       │  ← Point d'entree
│  new Model()     │
│  new View()      │
│  new Controller()│
└──────────────────┘
         │
    ┌────┴────────────────────────┐
    │                             │
    v                             v
┌──────────────────┐    ┌──────────────────┐
│  controller.js   │    │    model.js       │
│                  │───>│                  │
│  #ajouterTache() │    │  #tasks = []     │
│  #supprimerTache│    │  ajouter()       │
│  #commencerEdit │    │  modifier()      │
│  #configurerDnD │    │  supprimer()     │
│  #rafraichir()  │    │  filtrer()       │
│                  │    │  #sauvegarder()  │<──> localStorage
└───────┬──────────┘    │  #charger()      │
        │               │  onChangement()  │
        v               └──────────────────┘
┌──────────────────┐
│    view.js       │
│                  │
│  afficherTaches()│
│  activerEdition()│
│  animerSuppr()   │
│  setTheme()      │
│  #echapper()     │
└──────────────────┘

Flux de donnees :
Utilisateur → Controller → Model → localStorage
                    ↓
              View (DOM)
                    ↓
              Utilisateur
```

---

## Carte Mentale ASCII

```
                        Projet Todo App
                              │
        ┌─────────┬───────────┼───────────┬─────────────┐
        │         │           │           │             │
     HTML       CSS          MVC       Features       Qualite
        │         │           │           │             │
   Semantique  Variables   Model        CRUD         Erreurs
   BEM         Dark mode   View         Filtres      XSS escape
   Accessib.   Responsive  Controller   Recherche    Validation
   data-*      Animations              localStorage  try/catch
               @keyframes              Drag & Drop
               Transitions             Theme toggle
                                       Debounce
        ┌──────────┐
        │          │
   Event Deleg.  Modules
   bubbling     import/export
   closest()    separation
   dataset      fichiers
```

---

## Exercices

### Exercice 1 : Ajouter des categories

Etendez l'application avec un systeme de categories :
- Chaque tache peut avoir une categorie (Travail, Personnel, Courses, etc.)
- Un selecteur de categorie dans le formulaire d'ajout (dropdown)
- Des filtres par categorie (en plus des filtres actif/complete)
- Une couleur differente par categorie (pastilles colorees)
- Persistance des categories personnalisees dans localStorage

### Exercice 2 : Ajouter des dates d'echeance

Implementez un systeme de dates limites :
- Input `<input type="date">` dans le formulaire
- Affichage de la date formatee a cote de chaque tache
- Tri par date d'echeance
- Indicateur visuel : vert (dans les temps), orange (demain), rouge (en retard)
- Notification quand une tache approche de sa date limite

### Exercice 3 : Undo / Redo

Implementez un systeme d'annulation :
- Pile d'historique des actions (pattern Command)
- Bouton "Annuler" (Ctrl+Z) qui restaure l'etat precedent
- Bouton "Refaire" (Ctrl+Y)
- Maximum 20 etapes d'historique
- Fonctionne pour ajout, suppression, modification et changement de statut

### Exercice 4 : Export / Import

Ajoutez des fonctionnalites d'export :
- Export en JSON (telechargement d'un fichier `.json`)
- Export en CSV
- Import depuis un fichier JSON (drag & drop d'un fichier sur l'app)
- Fusion intelligente (ne pas creer de doublons)
- Utiliser l'API `File` et `FileReader`

---

## Liens

- [[02 - JavaScript DOM et Evenements]]
- [[04 - JavaScript Moderne ES6+]]
- [[05 - SPA et Frameworks Introduction]]
- [[02 - CSS Fondamentaux]]
