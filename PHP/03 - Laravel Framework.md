# Laravel Framework

> [!tip] Analogie
> Imagine que tu construis une maison. Tu pourrais couper chaque planche, fabriquer chaque clou et melanger toi-meme le ciment. Ou tu pourrais utiliser des materiaux prefabriques, un plan architectural clair et des outils specialises pour chaque tache.
> **Laravel, c'est la deuxieme option pour les applications web PHP.** Il te fournit la structure, les outils et les conventions pour construire rapidement sans reinventer la roue — tout en gardant un code propre et maintenable.

---

## Qu'est-ce que Laravel ?

Laravel est un **framework PHP open-source** cree par **Taylor Otwell** en 2011. Il repose sur le patron de conception **MVC (Model-View-Controller)** et est concu pour rendre le developpement web a la fois elegant et expressif.

### Historique rapide

| Version | Annee | Nouveautes majeures |
|---------|-------|---------------------|
| Laravel 1.x | 2011 | Premiere version, routing, controllers |
| Laravel 3.x | 2012 | Bundles (ancetre des packages), Artisan CLI |
| Laravel 4.x | 2013 | Rearchitecture complete sur Composer |
| Laravel 5.x | 2015 | Middleware, Elixir, Socialite |
| Laravel 6.x | 2019 | Premiere LTS moderne, Semantic Versioning |
| Laravel 8.x | 2020 | Jetstream, Models directory, Job batching |
| Laravel 10.x | 2023 | PHP 8.1 minimum, types natifs |
| Laravel 11.x | 2024 | Structure simplifiee, Reverb (WebSockets) |

### Le patron MVC applique a Laravel

```
REQUETE HTTP
     |
     v
+----------+      +------------+      +----------+
|  ROUTE   |----->| CONTROLLER |----->|  MODEL   |
| web.php  |      | UserCtrl   |      | User.php |
+----------+      +------------+      +----------+
                        |                   |
                        |              Base de donnees
                        v
                  +----------+
                  |   VIEW   |
                  | Blade    |
                  +----------+
                        |
                        v
               REPONSE HTML/JSON
```

- **Model** : represente les donnees et la logique metier (Eloquent ORM)
- **View** : les templates Blade qui generent le HTML
- **Controller** : le chef d'orchestre qui recoit la requete, interroge le Model, et renvoie la View

### L'ecosysteme Laravel

Laravel n'est pas seul — il s'accompagne d'un ecosysteme riche :

| Outil | Role |
|-------|------|
| **Composer** | Gestionnaire de dependances PHP |
| **Artisan** | CLI de generation de code et gestion |
| **Eloquent** | ORM (Object-Relational Mapper) |
| **Blade** | Moteur de templates |
| **Tinker** | REPL interactif pour tester le code |
| **Sail** | Environnement Docker local |
| **Breeze / Jetstream** | Scaffolding d'authentification |
| **Horizon** | Monitoring des queues Redis |
| **Telescope** | Debugger/observabilite locale |
| **Forge / Vapor** | Deploiement (serveurs / serverless) |

> [!info] Pourquoi Laravel plutot que du PHP pur ?
> En PHP pur, chaque projet reinvente : la gestion des routes, la protection CSRF, le hachage des mots de passe, les migrations de BDD... Laravel resout ces problemes une fois pour toutes avec des conventions eprouvees. Tu ecris moins de code repetitif et tu te concentres sur la logique metier.

---

## Installation et Structure du Projet

### Prerequis

- PHP >= 8.2
- Composer installe globalement
- Extension PHP : `pdo`, `mbstring`, `xml`, `curl`, `openssl`
- MySQL, PostgreSQL ou SQLite

### Creer un nouveau projet

```bash
# Via Composer (methode universelle)
composer create-project laravel/laravel mon-projet

# Via l'installateur Laravel (plus rapide apres installation)
composer global require laravel/installer
laravel new mon-projet

# Avec Laravel 11 : choix interactif du starter kit
laravel new mon-projet --git --pest
```

```bash
# Demarrer le serveur de developpement
cd mon-projet
php artisan serve
# Application disponible sur http://localhost:8000
```

### Structure des dossiers

```
mon-projet/
|
|-- app/                    <- Code source principal
|   |-- Console/            <- Commandes Artisan personnalisees
|   |-- Exceptions/         <- Gestionnaire d'exceptions
|   |-- Http/
|   |   |-- Controllers/    <- Tes controllers
|   |   |-- Middleware/     <- Filtres HTTP
|   |   +-- Requests/       <- Validation de formulaires
|   |-- Models/             <- Tes modeles Eloquent
|   +-- Providers/          <- Service Providers
|
|-- bootstrap/              <- Demarrage du framework
|-- config/                 <- Fichiers de configuration
|   |-- app.php
|   |-- database.php
|   +-- ...
|
|-- database/
|   |-- factories/          <- Generateurs de fausses donnees
|   |-- migrations/         <- Historique du schema BDD
|   +-- seeders/            <- Donnees initiales
|
|-- public/                 <- Seul dossier accessible depuis le web
|   +-- index.php           <- Point d'entree unique
|
|-- resources/
|   |-- css/                <- Styles (avant compilation)
|   |-- js/                 <- Scripts (avant compilation)
|   +-- views/              <- Templates Blade (.blade.php)
|
|-- routes/
|   |-- web.php             <- Routes de l'interface web
|   |-- api.php             <- Routes de l'API REST
|   +-- console.php         <- Routes des commandes Artisan
|
|-- storage/                <- Fichiers generes (logs, cache, uploads)
|-- tests/                  <- Tests PHPUnit / Pest
+-- .env                    <- Variables d'environnement (JAMAIS commite)
```

> [!warning] Le fichier .env
> Le fichier `.env` contient toutes tes donnees sensibles : cles d'API, identifiants de base de donnees, etc. Il est liste dans `.gitignore` par defaut. Ne le commite **jamais**. Utilise `.env.example` (sans valeurs) pour documenter les variables necessaires.

### Configuration de base (.env)

```bash
APP_NAME=MonApplication
APP_ENV=local
APP_KEY=base64:CLES_GENEREE_PAR_ARTISAN
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=mon_projet
DB_USERNAME=root
DB_PASSWORD=secret
```

```bash
# Generer la cle d'application (obligatoire apres installation)
php artisan key:generate
```

---

## Artisan CLI

Artisan est le couteau suisse de Laravel. C'est une interface en ligne de commande qui automatise les taches repetitives : generer du code, gerer la base de donnees, lancer des tests, vider les caches...

### Commandes essentielles

```bash
# INFORMATIONS
php artisan list                    # Lister toutes les commandes
php artisan help make:controller    # Aide sur une commande specifique

# GENERATION DE CODE
php artisan make:model Article                        # Model seul
php artisan make:model Article -m                     # Model + Migration
php artisan make:model Article -mcr                   # Model + Migration + Controller Resource
php artisan make:controller ArticleController
php artisan make:controller ArticleController --resource  # CRUD complet
php artisan make:request StoreArticleRequest          # Form Request
php artisan make:middleware CheckAge                  # Middleware
php artisan make:seeder ArticleSeeder                 # Seeder
php artisan make:factory ArticleFactory               # Factory
php artisan make:job ProcessPodcast                   # Job pour les queues
php artisan make:mail WelcomeMail                     # Mailable
php artisan make:event UserRegistered                 # Evenement
php artisan make:command SendReminders                # Commande custom

# BASE DE DONNEES
php artisan migrate                  # Executer les migrations en attente
php artisan migrate:fresh            # Supprimer et recreer toutes les tables
php artisan migrate:fresh --seed     # + lancer les seeders
php artisan migrate:rollback         # Annuler la derniere migration
php artisan migrate:status           # Voir l'etat des migrations
php artisan db:seed                  # Lancer les seeders

# CACHE ET OPTIMISATION
php artisan cache:clear              # Vider le cache applicatif
php artisan config:clear             # Vider le cache de config
php artisan route:clear              # Vider le cache de routes
php artisan view:clear               # Vider le cache Blade
php artisan optimize                 # Cacher config + routes (production)

# DEVELOPPEMENT
php artisan serve                    # Serveur de dev sur :8000
php artisan tinker                   # REPL interactif
php artisan route:list               # Lister toutes les routes
```

### Tinker : le laboratoire interactif

Tinker te permet d'interagir avec ton application en direct depuis le terminal. C'est l'outil numero 1 pour tester rapidement :

```bash
php artisan tinker
```

```php
# Dans Tinker :
>>> App\Models\User::count()
=> 5

>>> $user = App\Models\User::find(1)
=> App\Models\User {#4321 id: 1, name: "Alice", ...}

>>> $user->articles()->count()
=> 12

>>> App\Models\Article::where('published', true)->latest()->first()
=> App\Models\Article {#4322 ...}
```

---

## Routing

Le routing est le systeme qui associe une URL a une action. Dans Laravel, les routes sont definies dans le dossier `routes/`.

### Routes de base

```php
// routes/web.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ArticleController;

// GET simple - closure
Route::get('/bonjour', function () {
    return 'Bonjour le monde !';
});

// GET avec une view Blade
Route::get('/', function () {
    return view('welcome');
});

// Toutes les methodes HTTP
Route::get('/articles', [ArticleController::class, 'index']);
Route::post('/articles', [ArticleController::class, 'store']);
Route::put('/articles/{id}', [ArticleController::class, 'update']);
Route::patch('/articles/{id}', [ArticleController::class, 'update']);
Route::delete('/articles/{id}', [ArticleController::class, 'destroy']);
```

### Parametres de route

```php
// Parametre obligatoire
Route::get('/articles/{id}', function (int $id) {
    return "Article numero $id";
});

// Parametre optionnel (avec valeur par defaut)
Route::get('/articles/{slug?}', function (?string $slug = null) {
    return $slug ?? 'Tous les articles';
});

// Contrainte de type (regex)
Route::get('/articles/{id}', function (int $id) {
    // ...
})->where('id', '[0-9]+');

// Contrainte pratique avec methodes helper
Route::get('/articles/{id}', ...)->whereNumber('id');
Route::get('/users/{name}', ...)->whereAlpha('name');
Route::get('/items/{uuid}', ...)->whereUuid('uuid');
```

### Routes nommees

Les routes nommees permettent de generer des URLs sans hardcoder les chemins :

```php
// Definir un nom
Route::get('/articles/{id}', [ArticleController::class, 'show'])
    ->name('articles.show');

Route::get('/tableau-de-bord', [DashboardController::class, 'index'])
    ->name('dashboard');
```

```php
// Utiliser le nom dans le code PHP
$url = route('articles.show', ['id' => 42]);
// Resultat : http://localhost/articles/42

// Redirection vers une route nommee
return redirect()->route('dashboard');
```

```blade
{{-- Dans une template Blade --}}
<a href="{{ route('articles.show', $article->id) }}">
    Lire l'article
</a>
```

### Groupes de routes

```php
// Groupe avec prefixe URL
Route::prefix('admin')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'index']);
    Route::get('/users', [AdminController::class, 'users']);
    // URLs generees : /admin/dashboard, /admin/users
});

// Groupe avec middleware
Route::middleware(['auth'])->group(function () {
    Route::get('/profil', [ProfileController::class, 'show']);
    Route::post('/profil', [ProfileController::class, 'update']);
});

// Groupe avec prefixe de nom
Route::name('admin.')->group(function () {
    Route::get('/admin/dashboard', ...)->name('dashboard');
    // Nom genere : admin.dashboard
});

// Tout combine
Route::prefix('admin')
    ->name('admin.')
    ->middleware(['auth', 'admin'])
    ->group(function () {
        Route::get('/dashboard', [AdminController::class, 'index'])->name('dashboard');
    });
```

### Middleware

Les middlewares sont des filtres qui s'executent avant (ou apres) le traitement d'une requete :

```php
// Middleware global dans bootstrap/app.php (Laravel 11)
// Ou dans app/Http/Kernel.php (Laravel 10 et avant)

// Middlewares fournis par Laravel
Route::middleware('auth')->group(...);           // Utilisateur authentifie
Route::middleware('guest')->group(...);          // Visiteur non connecte
Route::middleware('throttle:60,1')->group(...);  // Rate limiting (60 req/min)
```

```php
// Creer un middleware personnalise
// app/Http/Middleware/CheckAge.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckAge
{
    public function handle(Request $request, Closure $next): mixed
    {
        if ($request->age < 18) {
            return redirect('/accueil');
        }

        return $next($request); // Passer au middleware suivant / controller
    }
}
```

> [!info] Routes API vs Routes Web
> `routes/web.php` → sessions, cookies, protection CSRF — pour les interfaces HTML
> `routes/api.php` → sans session, sans CSRF, prefixe `/api/` automatique — pour les APIs REST consommees par du JavaScript ou des apps mobiles

---

## Controllers et Resources

### Controller basique

```php
// app/Http/Controllers/ArticleController.php
// php artisan make:controller ArticleController

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ArticleController extends Controller
{
    public function index()
    {
        return view('articles.index');
    }

    public function show(int $id)
    {
        return view('articles.show', ['id' => $id]);
    }
}
```

### Resource Controller (CRUD complet)

```bash
php artisan make:controller ArticleController --resource
```

Un Resource Controller implemente les 7 methodes du CRUD RESTful :

```php
namespace App\Http\Controllers;

use App\Models\Article;
use Illuminate\Http\Request;

class ArticleController extends Controller
{
    // GET /articles - lister tous les articles
    public function index()
    {
        $articles = Article::latest()->paginate(15);
        return view('articles.index', compact('articles'));
    }

    // GET /articles/create - afficher le formulaire de creation
    public function create()
    {
        return view('articles.create');
    }

    // POST /articles - enregistrer un nouvel article
    public function store(Request $request)
    {
        $article = Article::create($request->validated());
        return redirect()->route('articles.show', $article)
                         ->with('success', 'Article cree !');
    }

    // GET /articles/{article} - afficher un article
    public function show(Article $article) // Route Model Binding
    {
        return view('articles.show', compact('article'));
    }

    // GET /articles/{article}/edit - formulaire d'edition
    public function edit(Article $article)
    {
        return view('articles.edit', compact('article'));
    }

    // PUT /articles/{article} - mettre a jour
    public function update(Request $request, Article $article)
    {
        $article->update($request->validated());
        return redirect()->route('articles.show', $article)
                         ->with('success', 'Article mis a jour !');
    }

    // DELETE /articles/{article} - supprimer
    public function destroy(Article $article)
    {
        $article->delete();
        return redirect()->route('articles.index')
                         ->with('success', 'Article supprime !');
    }
}
```

### Enregistrer les routes resource

```php
// routes/web.php
Route::resource('articles', ArticleController::class);

// Ca genere automatiquement ces 7 routes :
// GET    /articles              -> index    (articles.index)
// GET    /articles/create       -> create   (articles.create)
// POST   /articles              -> store    (articles.store)
// GET    /articles/{article}    -> show     (articles.show)
// GET    /articles/{article}/edit -> edit   (articles.edit)
// PUT    /articles/{article}    -> update   (articles.update)
// DELETE /articles/{article}    -> destroy  (articles.destroy)
```

```bash
# Verifier les routes generees
php artisan route:list --name=articles
```

### Route Model Binding

Laravel peut resoudre automatiquement un modele Eloquent depuis un parametre de route :

```php
// Sans Route Model Binding (style "a l'ancienne")
public function show(int $id)
{
    $article = Article::findOrFail($id); // 404 si non trouve
    return view('articles.show', compact('article'));
}

// Avec Route Model Binding (elegant !)
public function show(Article $article) // Laravel fait le findOrFail automatiquement
{
    return view('articles.show', compact('article'));
}
```

> [!tip] Route Model Binding
> Laravel lit le parametre `{article}` dans la route, voit que le type-hint est `Article`, et execute automatiquement `Article::findOrFail($id)`. Si l'article n'existe pas, il renvoie une 404. Gratuit, zero boilerplate.

---

## Blade Templates

Blade est le moteur de templates de Laravel. Il compile les templates en PHP pur et les met en cache — zero overhead en production.

### Syntaxe de base

```blade
{{-- Ceci est un commentaire Blade (n'apparait pas dans le HTML) --}}

{{-- Afficher une variable (avec echappement XSS automatique) --}}
<h1>{{ $article->titre }}</h1>

{{-- Afficher sans echappement (attention : uniquement pour du HTML de confiance) --}}
<div>{!! $article->contenu_html !!}</div>

{{-- Valeur par defaut --}}
{{ $user->bio ?? 'Pas de biographie' }}
```

### Heritage de templates (layouts)

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>@yield('title', 'Mon Application')</title>
    <link rel="stylesheet" href="{{ asset('css/app.css') }}">
</head>
<body>
    <nav>
        <a href="{{ route('articles.index') }}">Articles</a>
        @auth
            <a href="{{ route('profile.show') }}">Mon Profil</a>
        @endauth
        @guest
            <a href="{{ route('login') }}">Connexion</a>
        @endguest
    </nav>

    <main>
        @if(session('success'))
            <div class="alert alert-success">{{ session('success') }}</div>
        @endif

        @yield('content')  {{-- Les pages injectent leur contenu ici --}}
    </main>

    @stack('scripts')  {{-- Pour les scripts JS specifiques a une page --}}
</body>
</html>
```

```blade
{{-- resources/views/articles/index.blade.php --}}
@extends('layouts.app')

@section('title', 'Liste des Articles')

@section('content')
    <h1>Tous les articles</h1>

    @forelse($articles as $article)
        <article>
            <h2>
                <a href="{{ route('articles.show', $article) }}">
                    {{ $article->titre }}
                </a>
            </h2>
            <p>Par {{ $article->user->name }} — {{ $article->created_at->diffForHumans() }}</p>
        </article>
    @empty
        <p>Aucun article pour le moment.</p>
    @endforelse

    {{-- Pagination automatique --}}
    {{ $articles->links() }}
@endsection

@push('scripts')
    <script src="{{ asset('js/articles.js') }}"></script>
@endpush
```

### Directives Blade essentielles

```blade
{{-- Conditions --}}
@if($user->isAdmin())
    <span class="badge">Administrateur</span>
@elseif($user->isModerator())
    <span class="badge">Moderateur</span>
@else
    <span class="badge">Membre</span>
@endif

{{-- Unless (equivalent de @if(!condition)) --}}
@unless($article->published)
    <span>Brouillon</span>
@endunless

{{-- Boucles --}}
@foreach($articles as $article)
    <li>{{ $article->titre }}</li>
@endforeach

{{-- foreach avec variable $loop --}}
@foreach($items as $item)
    @if($loop->first) <ul> @endif
    <li class="{{ $loop->even ? 'pair' : 'impair' }}">
        {{ $loop->iteration }}/{{ $loop->count }} - {{ $item->nom }}
    </li>
    @if($loop->last) </ul> @endif
@endforeach

{{-- forelse : foreach + cas vide --}}
@forelse($articles as $article)
    <li>{{ $article->titre }}</li>
@empty
    <li>Aucun article.</li>
@endforelse

{{-- for et while classiques --}}
@for($i = 0; $i < 3; $i++)
    <span>{{ $i }}</span>
@endfor

{{-- Authentification --}}
@auth
    <p>Bonjour {{ auth()->user()->name }} !</p>
@endauth

@guest
    <a href="{{ route('login') }}">Se connecter</a>
@endguest

{{-- Inclure un sous-template --}}
@include('partials.alert', ['type' => 'success', 'message' => 'OK'])

{{-- Inclure si le fichier existe --}}
@includeIf('partials.sidebar')

{{-- Components (Laravel 7+) --}}
<x-alert type="danger" :message="$erreur" />
```

### Formulaires avec protection CSRF

```blade
<form method="POST" action="{{ route('articles.store') }}">
    @csrf  {{-- Token CSRF obligatoire pour tous les POST/PUT/DELETE --}}

    <div>
        <label for="titre">Titre</label>
        <input type="text"
               id="titre"
               name="titre"
               value="{{ old('titre') }}"
               class="{{ $errors->has('titre') ? 'is-invalid' : '' }}">

        @error('titre')
            <span class="error">{{ $message }}</span>
        @enderror
    </div>

    <button type="submit">Publier</button>
</form>

{{-- Pour PUT/DELETE (HTML ne supporte que GET/POST) --}}
<form method="POST" action="{{ route('articles.update', $article) }}">
    @csrf
    @method('PUT')  {{-- Laravel intercepte et traite comme PUT --}}
    ...
</form>
```

> [!warning] @csrf obligatoire
> Toute soumission de formulaire (POST, PUT, DELETE) **doit** contenir `@csrf`. Laravel rejette les requetes sans token CSRF valide avec une erreur 419. C'est une protection contre les attaques Cross-Site Request Forgery.

---

## Eloquent ORM

Eloquent est l'ORM (Object-Relational Mapper) de Laravel. Chaque table de base de donnees correspond a un Model Eloquent — un objet PHP qui te permet d'interagir avec les donnees sans ecrire de SQL.

### Creer un Model et sa Migration

```bash
php artisan make:model Article -m
# Cree : app/Models/Article.php
#        database/migrations/2024_01_15_120000_create_articles_table.php
```

### Migrations

Les migrations sont un systeme de versioning pour ta base de donnees :

```php
// database/migrations/2024_01_15_120000_create_articles_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('articles', function (Blueprint $table) {
            $table->id();                           // BIGINT AUTO_INCREMENT PRIMARY KEY
            $table->foreignId('user_id')            // BIGINT
                  ->constrained()                   // FK -> users.id
                  ->onDelete('cascade');             // Suppression en cascade
            $table->string('titre');                // VARCHAR(255)
            $table->string('slug')->unique();       // VARCHAR(255) UNIQUE
            $table->text('contenu');                // TEXT
            $table->string('image')->nullable();    // NULL autorise
            $table->boolean('published')->default(false);
            $table->timestamp('published_at')->nullable();
            $table->timestamps();                   // created_at + updated_at
            $table->softDeletes();                  // deleted_at (soft delete)
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('articles');
    }
};
```

```bash
php artisan migrate          # Executer les nouvelles migrations
php artisan migrate:rollback # Annuler la derniere serie
php artisan migrate:fresh    # Repartir de zero (dev seulement !)
```

### Le Model Eloquent

```php
// app/Models/Article.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Article extends Model
{
    use HasFactory, SoftDeletes;

    // Colonnes modifiables en masse (securite)
    protected $fillable = ['titre', 'slug', 'contenu', 'image', 'published', 'user_id'];

    // Colonnes jamais exposees (ex: password dans User)
    // protected $guarded = ['id'];

    // Cast automatique des types
    protected $casts = [
        'published'    => 'boolean',
        'published_at' => 'datetime',
    ];
}
```

> [!warning] $fillable vs $guarded
> Sans `$fillable` ou `$guarded`, `Article::create($request->all())` pourrait laisser un utilisateur modifier n'importe quelle colonne (ex: `user_id`, `is_admin`). C'est une faille **Mass Assignment**. Toujours definir `$fillable` avec la liste des colonnes acceptees.

### Operations CRUD avec Eloquent

```php
// CREATE
$article = Article::create([
    'titre'   => 'Mon Premier Article',
    'slug'    => 'mon-premier-article',
    'contenu' => 'Contenu de l\'article...',
    'user_id' => auth()->id(),
]);

// READ
$tous = Article::all();                        // Tous les enregistrements
$article = Article::find(1);                   // Par ID (null si absent)
$article = Article::findOrFail(1);             // Par ID (404 si absent)
$premier = Article::first();                   // Le premier
$compte = Article::count();                    // Nombre total

// READ avec conditions (Query Builder)
$publies = Article::where('published', true)
                  ->orderBy('created_at', 'desc')
                  ->paginate(15);

$article = Article::where('slug', 'mon-article')->firstOrFail();

// UPDATE
$article = Article::findOrFail(1);
$article->update(['titre' => 'Nouveau Titre']);

// Ou :
$article->titre = 'Nouveau Titre';
$article->save();

// DELETE
$article->delete();           // Soft delete si SoftDeletes est utilise
$article->forceDelete();      // Suppression definitive

// Restaurer un soft-deleted
Article::withTrashed()->find(1)->restore();
```

### Scopes de requete

Les scopes permettent de nommer et reutiliser des conditions de requete :

```php
// app/Models/Article.php

// Local scope : accessible via ->published()
public function scopePublished($query)
{
    return $query->where('published', true)
                 ->whereNotNull('published_at');
}

// Scope avec parametre
public function scopeOfCategory($query, string $category)
{
    return $query->where('category', $category);
}
```

```php
// Utilisation des scopes
$articles = Article::published()->latest()->paginate(10);
$tech = Article::published()->ofCategory('tech')->get();
```

### Mutateurs et Accesseurs

```php
// app/Models/Article.php
use Illuminate\Database\Eloquent\Casts\Attribute;

// Accesseur : transforme la valeur a la lecture
protected function titre(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => ucfirst($value),
    );
}

// Mutateur : transforme la valeur a l'ecriture
protected function slug(): Attribute
{
    return Attribute::make(
        set: fn (string $value) => Str::slug($value),
    );
}
```

---

## Relations Eloquent

Les relations representent les liens entre les tables. C'est l'une des fonctionnalites les plus puissantes d'Eloquent.

### Vue d'ensemble des relations

```
[users]  ----<  [articles]  ----<  [comments]
  1           N      N           N      1
  |                  |                  |
hasMany()      belongsTo()       belongsTo()

[articles]  >----<  [tags]
     N          N
     |          |
 belongsToMany() / belongsToMany()
```

### hasMany et belongsTo (Un a Plusieurs)

```php
// app/Models/User.php
class User extends Model
{
    // Un utilisateur a plusieurs articles
    public function articles(): HasMany
    {
        return $this->hasMany(Article::class);
    }
}

// app/Models/Article.php
class Article extends Model
{
    // Un article appartient a un utilisateur
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    // Un article a plusieurs commentaires
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

```php
// Utilisation
$user = User::find(1);
$articles = $user->articles;            // Collection d'Articles
$articles = $user->articles()->latest()->get(); // Avec query builder

$article = Article::find(1);
$auteur = $article->user;               // Objet User
$nom = $article->user->name;           // Acces a la propriete
```

### belongsToMany (Plusieurs a Plusieurs)

```php
// Necessaire : une table pivot 'article_tag' (order alphabetique)
// Colonnes : article_id, tag_id

// app/Models/Article.php
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class);
}

// app/Models/Tag.php
public function articles(): BelongsToMany
{
    return $this->belongsToMany(Article::class);
}
```

```php
// Utilisation
$article = Article::find(1);
$tags = $article->tags;                           // Collection de Tags
$ids = $article->tags->pluck('id');

// Attacher / detacher des tags
$article->tags()->attach([1, 2, 3]);             // Ajouter
$article->tags()->detach([1]);                   // Retirer
$article->tags()->sync([2, 3, 5]);               // Remplacer (= detach tout + attach)
$article->tags()->toggle([1, 4]);                // Inverser

// Verifier si un tag est attache
$article->tags->contains(1);
```

### Eager Loading : eviter le probleme N+1

```php
// MAUVAIS - genere N+1 requetes SQL
$articles = Article::all();
foreach ($articles as $article) {
    echo $article->user->name; // 1 requete par article !
}
// Si 100 articles : 101 requetes SQL

// BON - Eager Loading avec with()
$articles = Article::with('user')->get();
// 2 requetes SQL : une pour les articles, une pour tous les users

// Charger plusieurs relations
$articles = Article::with(['user', 'tags', 'comments'])->get();

// Charger des relations imbriquees
$articles = Article::with(['comments.user'])->get();

// Eager loading conditionnel
$articles = Article::with(['comments' => function ($query) {
    $query->where('approved', true)->latest();
}])->get();
```

> [!warning] Le probleme N+1
> Afficher 100 articles avec leurs auteurs sans eager loading = **101 requetes SQL**. Avec `with('user')` = **2 requetes**. C'est le bug de performance le plus frequent dans les apps Laravel. Utilise **Laravel Debugbar** en dev pour le detecter.

### hasOne et hasOneThrough

```php
// Un utilisateur a un profil
class User extends Model
{
    public function profile(): HasOne
    {
        return $this->hasOne(Profile::class);
    }
}

// Acces
$user->profile->avatar;
$user->profile()->updateOrCreate(['user_id' => $user->id], $data);
```

---

## Validation de Formulaires

Laravel propose un systeme de validation tres complet. La meilleure pratique est d'utiliser des **Form Request** classes.

### Validation dans le Controller (simple)

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'titre'    => 'required|string|max:255',
        'contenu'  => 'required|string|min:10',
        'image'    => 'nullable|image|max:2048', // max 2MB
        'category' => 'required|in:tech,culture,sport',
    ]);

    Article::create($validated);
    return redirect()->route('articles.index');
}
```

### Form Request (approche recommandee)

```bash
php artisan make:request StoreArticleRequest
```

```php
// app/Http/Requests/StoreArticleRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreArticleRequest extends FormRequest
{
    // Qui peut soumettre ce formulaire ?
    public function authorize(): bool
    {
        return auth()->check(); // Uniquement les utilisateurs connectes
    }

    // Les regles de validation
    public function rules(): array
    {
        return [
            'titre'        => ['required', 'string', 'max:255'],
            'slug'         => ['required', 'string', 'unique:articles,slug'],
            'contenu'      => ['required', 'string', 'min:50'],
            'image'        => ['nullable', 'image', 'mimes:jpg,png,webp', 'max:2048'],
            'published'    => ['boolean'],
            'published_at' => ['nullable', 'date', 'after:now'],
            'tags'         => ['array'],
            'tags.*'       => ['exists:tags,id'],
        ];
    }

    // Messages d'erreur personnalises
    public function messages(): array
    {
        return [
            'titre.required' => 'Le titre est obligatoire.',
            'titre.max'      => 'Le titre ne peut pas depasser :max caracteres.',
            'contenu.min'    => 'Le contenu doit contenir au moins :min caracteres.',
        ];
    }

    // Noms de champs lisibles (pour les messages auto)
    public function attributes(): array
    {
        return [
            'published_at' => 'date de publication',
        ];
    }
}
```

```php
// Dans le controller : remplacer Request par le Form Request
public function store(StoreArticleRequest $request)
{
    // Si on arrive ici, la validation a reussi
    // $request->validated() contient uniquement les champs valides
    $article = Article::create($request->validated());
    return redirect()->route('articles.show', $article);
}
```

### Regles de validation courantes

| Regle | Description |
|-------|-------------|
| `required` | Champ obligatoire |
| `nullable` | Accepte null / vide |
| `string` | Doit etre une chaine |
| `integer` | Doit etre un entier |
| `boolean` | Doit etre vrai/faux |
| `email` | Format email valide |
| `url` | Format URL valide |
| `min:3` | Min 3 caracteres (ou valeur) |
| `max:255` | Max 255 caracteres |
| `between:1,100` | Entre 1 et 100 |
| `unique:table,colonne` | Valeur unique dans la table |
| `exists:table,colonne` | Doit exister dans la table |
| `in:a,b,c` | Doit etre parmi les valeurs |
| `confirmed` | Doit etre confirme (mot de passe) |
| `image` | Fichier image (jpeg,png,gif,svg,webp) |
| `mimes:jpg,png` | Extensions autorisees |
| `max:2048` | Taille max en kilobytes (pour fichiers) |
| `date` | Date valide |
| `after:now` | Date dans le futur |
| `before:date` | Date avant une autre |
| `regex:/pattern/` | Correspond a l'expression reguliere |

---

## Authentication avec Laravel Breeze

Laravel Breeze est le starter kit d'authentification officiel — leger et simple a personnaliser.

### Installation

```bash
composer require laravel/breeze --dev
php artisan breeze:install

# Choix du stack (interactif) :
# - blade    : HTML classique + Alpine.js
# - react    : Inertia.js + React
# - vue      : Inertia.js + Vue.js
# - api      : API uniquement (pour SPA/mobile)

php artisan migrate
npm install && npm run dev
```

Breeze genere automatiquement :
- Routes `/login`, `/register`, `/logout`, `/password/reset`
- Controllers dans `app/Http/Controllers/Auth/`
- Views dans `resources/views/auth/`
- Middleware `auth` et `guest` configures

### Utilisation de l'authentification

```php
// Verifier si un utilisateur est connecte
if (auth()->check()) { ... }

// Obtenir l'utilisateur connecte
$user = auth()->user();
$id = auth()->id();

// Proteger des routes
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
});

// Dans Blade
@auth
    <p>Connecte en tant que {{ auth()->user()->name }}</p>
@endauth
```

### Policies : Autorisation fine

```bash
php artisan make:policy ArticlePolicy --model=Article
```

```php
// app/Policies/ArticlePolicy.php

class ArticlePolicy
{
    // Un utilisateur peut-il modifier cet article ?
    public function update(User $user, Article $article): bool
    {
        return $user->id === $article->user_id;
    }

    // Un utilisateur peut-il supprimer cet article ?
    public function delete(User $user, Article $article): bool
    {
        return $user->id === $article->user_id || $user->isAdmin();
    }
}
```

```php
// Dans le Controller
public function update(StoreArticleRequest $request, Article $article)
{
    $this->authorize('update', $article); // Lance une 403 si refuse

    $article->update($request->validated());
    return redirect()->route('articles.show', $article);
}
```

```blade
{{-- Dans Blade --}}
@can('update', $article)
    <a href="{{ route('articles.edit', $article) }}">Modifier</a>
@endcan

@cannot('delete', $article)
    <span>Vous ne pouvez pas supprimer cet article</span>
@endcannot
```

---

## Queues et Jobs

Les queues permettent d'executer des taches lourdes (envoi d'email, traitement d'image, appels API) en **arriere-plan**, sans faire attendre l'utilisateur.

### Principe des Queues

```
REQUETE HTTP
     |
     v
Controller --- dispatch(Job) ---> Queue (Redis / Database)
     |                                  |
     v                            Worker process
Reponse immediate            (php artisan queue:work)
(< 200ms)                           |
                               Execute le Job
                               (envoi email, etc.)
```

### Creer et utiliser un Job

```bash
php artisan make:job SendWelcomeEmail
```

```php
// app/Jobs/SendWelcomeEmail.php

namespace App\Jobs;

use App\Models\User;
use App\Mail\WelcomeMail;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;        // Nombre de tentatives en cas d'echec
    public int $timeout = 60;     // Timeout en secondes

    public function __construct(
        private readonly User $user  // L'objet est serialise/deserialise automatiquement
    ) {}

    public function handle(): void
    {
        Mail::to($this->user->email)->send(new WelcomeMail($this->user));
    }

    // Optionnel : que faire si le job echoue definitivement ?
    public function failed(\Throwable $exception): void
    {
        // Logger, notifier, etc.
    }
}
```

```php
// Dispatcher le Job depuis un Controller
class RegisterController extends Controller
{
    public function store(Request $request)
    {
        $user = User::create($request->validated());

        // Envoi immediat (dans le meme processus)
        // SendWelcomeEmail::dispatchSync($user);

        // Envoi en arriere-plan (recommande)
        SendWelcomeEmail::dispatch($user);

        // Avec delai
        SendWelcomeEmail::dispatch($user)->delay(now()->addMinutes(5));

        // Sur une queue specifique
        SendWelcomeEmail::dispatch($user)->onQueue('emails');

        return redirect()->route('dashboard');
    }
}
```

### Configuration et lancement

```bash
# .env
QUEUE_CONNECTION=database  # ou redis, sqs, beanstalkd

# Creer la table des jobs (si QUEUE_CONNECTION=database)
php artisan queue:table
php artisan migrate

# Lancer le worker (en dev)
php artisan queue:work

# Worker avec retry et timeout (production)
php artisan queue:work --tries=3 --timeout=90 --queue=high,default,low

# En production, utiliser Supervisor pour relancer automatiquement
```

> [!info] Queues en production
> En production, utiliser **Redis** comme backend de queue (plus rapide et fiable que database). Utiliser **Supervisor** (Linux) pour maintenir les workers actifs. **Laravel Horizon** offre un dashboard de monitoring pour les queues Redis.

---

## Tests avec PHPUnit

Laravel integre PHPUnit et une couche de test tres ergonomique. Les tests sont dans le dossier `tests/`.

```
tests/
|-- Feature/     <- Tests de bout en bout (HTTP, BDD, etc.)
|-- Unit/        <- Tests unitaires purs (sans Laravel)
+-- TestCase.php <- Classe de base
```

### Tests de Feature (les plus courants)

```bash
php artisan make:test ArticleTest
php artisan make:test ArticleTest --unit  # Test unitaire
```

```php
// tests/Feature/ArticleTest.php

namespace Tests\Feature;

use App\Models\Article;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ArticleTest extends TestCase
{
    use RefreshDatabase; // Reinitialise la BDD avant chaque test

    public function test_authenticated_user_can_create_article(): void
    {
        // ARRANGE : preparer les donnees
        $user = User::factory()->create();

        // ACT : executer l'action
        $response = $this->actingAs($user)->post('/articles', [
            'titre'   => 'Mon Article de Test',
            'slug'    => 'mon-article-de-test',
            'contenu' => str_repeat('a', 50), // 50 caracteres minimum
        ]);

        // ASSERT : verifier le resultat
        $response->assertRedirect();
        $this->assertDatabaseHas('articles', [
            'titre'   => 'Mon Article de Test',
            'user_id' => $user->id,
        ]);
    }

    public function test_guest_cannot_create_article(): void
    {
        $response = $this->post('/articles', [
            'titre' => 'Article Interdit',
        ]);

        $response->assertRedirect('/login'); // Redirige vers login
        $this->assertDatabaseCount('articles', 0);
    }

    public function test_article_list_is_displayed(): void
    {
        Article::factory()->count(5)->create(['published' => true]);

        $response = $this->get('/articles');

        $response->assertStatus(200);
        $response->assertViewIs('articles.index');
        $response->assertViewHas('articles');
    }

    public function test_unpublished_article_returns_404(): void
    {
        $article = Article::factory()->create(['published' => false]);

        $response = $this->get("/articles/{$article->slug}");

        $response->assertNotFound();
    }
}
```

### Factories : generer des donnees de test

```php
// database/factories/ArticleFactory.php

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class ArticleFactory extends Factory
{
    public function definition(): array
    {
        $titre = $this->faker->sentence(6);

        return [
            'user_id'   => User::factory(), // Creer un User associe
            'titre'     => $titre,
            'slug'      => Str::slug($titre),
            'contenu'   => $this->faker->paragraphs(3, true),
            'published' => $this->faker->boolean(70), // 70% de chance d'etre publie
        ];
    }

    // Etat personnalise
    public function published(): static
    {
        return $this->state(fn () => [
            'published'    => true,
            'published_at' => now(),
        ]);
    }

    public function draft(): static
    {
        return $this->state(fn () => ['published' => false]);
    }
}
```

```php
// Utilisation des factories dans les tests
User::factory()->create();                          // 1 user
User::factory()->count(10)->create();              // 10 users
Article::factory()->published()->create();         // 1 article publie
Article::factory()->count(5)->draft()->create();   // 5 brouillons
Article::factory()->for($user)->create();          // Article lie a un user specifique
```

### Tests Unitaires

```php
// tests/Unit/ArticleTest.php

namespace Tests\Unit;

use App\Models\Article;
use PHPUnit\Framework\TestCase; // Pas Illuminate\Foundation\Testing\TestCase

class ArticleUnitTest extends TestCase
{
    public function test_slug_is_generated_from_title(): void
    {
        $article = new Article();
        $article->titre = 'Mon Super Article !';

        $this->assertEquals('mon-super-article', $article->slug);
    }

    public function test_article_is_not_published_by_default(): void
    {
        $article = new Article();
        $this->assertFalse($article->published);
    }
}
```

### Lancer les tests

```bash
php artisan test                    # Tous les tests
php artisan test --filter=ArticleTest  # Un fichier specifique
php artisan test --filter="test_guest_cannot"  # Un test specifique
php artisan test --parallel         # En parallele (plus rapide)
php artisan test --coverage         # Avec couverture de code

# Commandes PHPUnit directement
./vendor/bin/phpunit
./vendor/bin/phpunit --testdox      # Sortie lisible
```

> [!example] Exemple de sortie de tests
> ```
> PASS  Tests\Feature\ArticleTest
> ✓ authenticated user can create article (0.15s)
> ✓ guest cannot create article (0.08s)
> ✓ article list is displayed (0.12s)
> ✓ unpublished article returns 404 (0.09s)
>
> Tests: 4 passed (8 assertions)
> Duration: 0.44s
> ```

---

## Carte Mentale

```
                         LARAVEL FRAMEWORK
                               |
         +-----------+---------+----------+----------+
         |           |         |          |          |
      REQUETE     ROUTING   MIDDLEWARE  SESSION   REPONSE
         |           |
         |    web.php / api.php
         |    Route::get/post/put/delete
         |    Route::resource()
         |    Named routes
         |    Groupes + Prefixes
         v
    CONTROLLER
         |
    +----+----+
    |         |
  VIEW      MODEL
(Blade)  (Eloquent)
    |         |
    |    +----+------+------+
    |    |    |      |      |
    |  CRUD  REL.  SCOPE  MUTATOR
    |  find  hasMany  published()
    |  create belongsTo
    |  update manyToMany
    |  delete
    |
 BLADE TEMPLATES
    |
 @extends / @section / @yield
 @if @foreach @auth @can
 {{ $var }} {!! $html !!}
 @csrf @method
    |
    +-- VALIDATION (Form Request)
    |   rules() / authorize()
    |   $request->validated()
    |
    +-- AUTHENTICATION (Breeze)
    |   auth()->user()
    |   @auth / @guest
    |   Policies + @can
    |
    +-- QUEUES (Jobs)
    |   ShouldQueue
    |   dispatch() / delay()
    |   queue:work
    |
    +-- TESTS (PHPUnit)
        Feature (HTTP)
        Unit (logique)
        Factories
        RefreshDatabase
```

---

## Exercices Pratiques

### Exercice 1 — Blog Minimal (debutant)

**Objectif** : Creer un blog avec CRUD d'articles.

1. Cree un projet Laravel : `laravel new blog`
2. Configure `.env` avec tes identifiants MySQL
3. Cree le Model `Article` avec migration (`-m`) incluant : `titre`, `contenu`, `slug`, `published` (boolean, default false)
4. Cree un `ArticleController --resource`
5. Ajoute `Route::resource('articles', ArticleController::class)` dans `web.php`
6. Cree les vues Blade : `articles/index.blade.php`, `articles/show.blade.php`, `articles/create.blade.php`
7. Implemente `index`, `create`, `store`, `show`
8. Valide que `titre` est required et `slug` est unique

**Criteres de reussite** :
- `php artisan route:list` montre 7 routes pour articles
- Tu peux creer un article via le formulaire
- La liste affiche les articles existants

---

### Exercice 2 — Relations et Eager Loading (intermediaire)

**Objectif** : Ajouter des categories aux articles.

1. Cree un Model `Category` avec migration : `nom`, `slug`
2. Cree un Model `Tag` avec migration : `nom`, `slug`
3. Ajoute une colonne `category_id` a la table articles (nouvelle migration !)
4. Cree une table pivot `article_tag` (migration manuelle)
5. Configure les relations :
   - `Article` → `belongsTo(Category::class)`
   - `Category` → `hasMany(Article::class)`
   - `Article` → `belongsToMany(Tag::class)`
   - `Tag` → `belongsToMany(Article::class)`
6. Dans `ArticleController::index()`, charge les relations avec `with()`
7. Affiche la categorie et les tags de chaque article dans la vue

**Criteres de reussite** :
- `php artisan tinker` : `Article::with('category', 'tags')->first()->tags` retourne une Collection
- La page index ne genere pas plus de 3 requetes SQL (verifier avec Debugbar)

---

### Exercice 3 — Validation et Authentication (intermediaire)

**Objectif** : Securiser le blog.

1. Installe Laravel Breeze : `composer require laravel/breeze --dev && php artisan breeze:install blade`
2. Protege les routes de creation/edition/suppression avec le middleware `auth`
3. Cree un `StoreArticleRequest` avec regles :
   - `titre` : required, string, max 255
   - `contenu` : required, min 100
   - `slug` : required, unique dans la table articles
   - `category_id` : required, exists dans categories
4. Ajoute une `ArticlePolicy` : seul l'auteur peut modifier/supprimer son article
5. Dans la vue, affiche le bouton "Modifier" uniquement avec `@can('update', $article)`

**Criteres de reussite** :
- Un visiteur non connecte est redirige vers `/login` s'il essaie de creer un article
- Un utilisateur ne peut pas modifier l'article d'un autre (reponse 403)
- Les erreurs de validation s'affichent sous les champs du formulaire

---

### Exercice 4 — Tests Automatises (avance)

**Objectif** : Ecrire une suite de tests pour le blog.

Ecris des Feature Tests qui couvrent :

1. **Test de creation** : un utilisateur authentifie peut creer un article → verifie `assertDatabaseHas`
2. **Test d'acces** : un visiteur ne peut pas acceder a la creation → verifie `assertRedirect('/login')`
3. **Test de validation** : soumettre un article sans titre → verifie `assertSessionHasErrors('titre')`
4. **Test d'autorisation** : un utilisateur ne peut pas modifier l'article d'un autre → `assertForbidden()`
5. **Test d'affichage** : la page index affiche bien les articles publies → `assertSee($article->titre)`

```bash
php artisan make:test ArticleFeatureTest
```

**Criteres de reussite** :
- `php artisan test` : tous les tests passent en vert
- Utilise `RefreshDatabase` et des `Factory` pour les donnees
- Aucun `assertDatabaseHas` sans avoir cree l'entite via Factory d'abord

---

## Pour aller plus loin

| Sujet | Ressource |
|-------|-----------|
| API REST avec Laravel Sanctum | Authentification par token pour les SPAs |
| Laravel Livewire | Composants reactifs sans JavaScript |
| Inertia.js | Bridge Laravel + React/Vue sans API separee |
| Laravel Horizon | Monitoring des queues Redis |
| Laravel Telescope | Debugger et observabilite |
| Laravel Octane | Boost de performance avec Swoole/RoadRunner |
| Task Scheduling | `php artisan schedule:run` — cron jobs |
| Events et Listeners | Systeme evenementiel decouple |
| Notifications | Email, SMS, Slack, Push depuis un objet Notification |
| Broadcasting | WebSockets temps reel avec Pusher ou Reverb |

> [!info] Documentation officielle
> La documentation Laravel (`laravel.com/docs`) est une des meilleures de tout l'ecosysteme PHP. Elle est mise a jour a chaque version et contient des exemples concrets. Commence par la section "Getting Started" puis suis les sections dans l'ordre de ce cours.

---

Voir aussi : [[PHP/01 - PHP et Symfony]], [[PHP/02 - PHP POO et Base de Donnees]], [[SQL/03 - Conception de Bases de Donnees]]
