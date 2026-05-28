# PHP et Symfony

## PHP en 2025 — Pourquoi Toujours Pertinent ?

PHP fait tourner **77 % du web mondial** — dont WordPress, Drupal, Magento, Shopify (partiellement). Sa réputation de "langage du passé" est erronée : PHP 8.x a introduit des fonctionnalités modernes qui en font un langage robuste, typé et performant.

> [!tip] Contexte
> PHP (PHP: Hypertext Preprocessor) a été créé en 1994 par Rasmus Lerdorf. Il est exécuté côté serveur, génère du HTML dynamique ou des réponses JSON pour les APIs. Avec Symfony et Composer, le PHP moderne est aussi structuré et maintenable que Java ou C#.

---

## PHP vs Python — Comparaison

| Critère | PHP | Python |
|---|---|---|
| **Usage principal** | Développement web (99 % web-only) | Généraliste (web, data, ML, scripts) |
| **Framework dominant** | Symfony, Laravel | Django, FastAPI, Flask |
| **Typage** | Optionnel mais disponible (declare strict_types) | Optionnel (type hints depuis Python 3.5) |
| **Gestion des dépendances** | Composer | pip / poetry |
| **Performance** | Très bonne avec PHP 8.x + OPcache + JIT | Bonne, mais GIL pour le threading |
| **Déploiement web** | php-fpm + nginx (natif) | WSGI / ASGI (gunicorn, uvicorn) |
| **Syntaxe** | Accolades C-style | Indentation significative |
| **Namespace** | `\Symfony\Component\...` | `from symfony.component import ...` |
| **Communauté** | Très large (web), vieillissante | En pleine croissance (ML driving it) |

---

## PHP 8.x — Fonctionnalités Modernes

### Types Stricts

```php
<?php
declare(strict_types=1); // TOUJOURS mettre en tête de fichier

// Types de paramètres et de retour
function diviser(float $a, float $b): float
{
    if ($b === 0.0) {
        throw new \InvalidArgumentException('Division par zéro');
    }
    return $a / $b;
}

// Union types (PHP 8.0+)
function traiter(int|string $valeur): string
{
    return is_int($valeur) ? "Entier: $valeur" : "Chaîne: $valeur";
}

// Nullable types
function trouverUtilisateur(int $id): ?Utilisateur
{
    return $this->repository->find($id); // null si non trouvé
}

// Return types avancés
function getItems(): array { ... }
function process(): void  { ... }  // ne retourne rien
function create(): never  { throw new Exception(); } // ne revient jamais
```

### Match Expression (PHP 8.0)

```php
// ❌ Switch — verbeux, fall-through risqué
switch ($statut) {
    case 'actif':
        $label = 'Actif';
        break;
    case 'inactif':
        $label = 'Inactif';
        break;
    default:
        $label = 'Inconnu';
}

// ✅ Match — concis, strict (=== comparaison), expression
$label = match($statut) {
    'actif'   => 'Actif',
    'inactif' => 'Inactif',
    default   => 'Inconnu',
};

// Match avec conditions multiples
$message = match(true) {
    $age < 18  => 'Mineur',
    $age < 65  => 'Adulte',
    default    => 'Senior',
};
```

### Enums (PHP 8.1)

```php
// Enum basique
enum Statut
{
    case Actif;
    case Inactif;
    case Suspendu;
}

// Backed enum (avec valeur scalaire)
enum StatutCommande: string
{
    case EnAttente  = 'pending';
    case Confirmee  = 'confirmed';
    case Expediee   = 'shipped';
    case Livree     = 'delivered';
    case Annulee    = 'cancelled';
    
    public function label(): string
    {
        return match($this) {
            self::EnAttente  => 'En attente',
            self::Confirmee  => 'Confirmée',
            self::Expediee   => 'Expédiée',
            self::Livree     => 'Livrée',
            self::Annulee    => 'Annulée',
        };
    }
    
    public function peutEtreAnnulee(): bool
    {
        return in_array($this, [self::EnAttente, self::Confirmee]);
    }
}

// Utilisation
$commande->statut = StatutCommande::Confirmee;
echo $commande->statut->label();       // "Confirmée"
echo $commande->statut->value;         // "confirmed"
$fromDB = StatutCommande::from('shipped'); // StatutCommande::Expediee

// Enum dans une entité Doctrine
#[Column(type: 'string', enumType: StatutCommande::class)]
private StatutCommande $statut = StatutCommande::EnAttente;
```

### Named Arguments (PHP 8.0)

```php
// ❌ Avant — difficile de savoir quoi est quoi
$result = array_slice($array, 1, 5, true);

// ✅ Named arguments — clair et lisible
$result = array_slice(
    array: $array,
    offset: 1,
    length: 5,
    preserve_keys: true,
);

// Très utile avec des fonctions à nombreux paramètres optionnels
$date = new DateTimeImmutable(
    datetime: 'now',
    timezone: new DateTimeZone('Europe/Paris'),
);
```

### Fibers (PHP 8.1 — Coroutines)

```php
// Fiber = coroutine légère, exécution suspendue/reprise
$fiber = new Fiber(function(): void {
    $valeur = Fiber::suspend('première suspension');
    echo "Repris avec : $valeur\n";
    Fiber::suspend('deuxième suspension');
    echo "Terminé\n";
});

$valeur1 = $fiber->start();          // "première suspension"
$valeur2 = $fiber->resume('hello');  // affiche "Repris avec : hello", retourne "deuxième suspension"
$fiber->resume('monde');             // affiche "Terminé"
```

### JIT — Just In Time Compilation (PHP 8.0)

```ini
; php.ini
opcache.enable=1
opcache.jit_buffer_size=100M
opcache.jit=1255  ; Tracing JIT (le plus efficace)
```

Le JIT améliore surtout les calculs purs (algorithmes, crypto, manipulation de données). Pour les apps web classiques (beaucoup d'I/O), le gain est plus modeste (~5-15 %).

---

## Composer — Gestionnaire de Dépendances

Composer est le `npm` de PHP.

```bash
# Installation de Composer
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer

# Créer un projet
composer create-project symfony/skeleton mon-projet
cd mon-projet

# Ajouter une dépendance
composer require symfony/orm-pack
composer require --dev symfony/maker-bundle

# Mettre à jour
composer update
composer update symfony/console  # un package spécifique

# Optimiser pour la production (génère un classmap)
composer install --no-dev --optimize-autoloader
```

### composer.json

```json
{
    "name": "monentreprise/mon-api",
    "type": "project",
    "require": {
        "php": ">=8.2",
        "symfony/framework-bundle": "7.1.*",
        "symfony/orm-pack": "^2.4",
        "lexik/jwt-authentication-bundle": "^3.1",
        "api-platform/core": "^3.3"
    },
    "require-dev": {
        "symfony/maker-bundle": "^1.59",
        "symfony/test-pack": "^1.1",
        "phpstan/phpstan": "^1.10"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "App\\Tests\\": "tests/"
        }
    }
}
```

**Autoloading PSR-4 :** quand vous créez `src/Service/EmailService.php` avec la classe `App\Service\EmailService`, Composer la trouve automatiquement — plus de `require` manuels.

---

## Symfony — Architecture MVC

Symfony suit le pattern **MVC** (Model-View-Controller) avec une organisation claire.

```
src/
├── Controller/          ← C : Controllers (gèrent les requêtes HTTP)
│   ├── ArticleController.php
│   └── SecurityController.php
├── Entity/              ← M : Entités (modèles Doctrine)
│   ├── Article.php
│   └── User.php
├── Repository/          ← M : Requêtes en base de données
│   └── ArticleRepository.php
├── Service/             ← Logique métier
│   └── ArticleService.php
├── Form/                ← Formulaires Symfony
│   └── ArticleType.php
└── Security/            ← Authentification
    └── AppAuthenticator.php

templates/               ← V : Templates Twig
├── article/
│   ├── index.html.twig
│   └── show.html.twig
└── base.html.twig
```

---

## Routing

```php
<?php
// src/Controller/ArticleController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

// Préfixe de route pour tout le controller
#[Route('/articles', name: 'article_')]
class ArticleController extends AbstractController
{
    // Route : GET /articles
    #[Route('', name: 'index', methods: ['GET'])]
    public function index(): Response
    {
        return $this->render('article/index.html.twig');
    }
    
    // Route avec paramètre : GET /articles/42
    #[Route('/{id}', name: 'show', requirements: ['id' => '\d+'], methods: ['GET'])]
    public function show(int $id): Response
    {
        // ParamConverter : Symfony trouve automatiquement l'article
        // (si on type-hinte Article $article, il fait le find automatiquement)
        return $this->render('article/show.html.twig', ['id' => $id]);
    }
    
    // Route POST : POST /articles
    #[Route('', name: 'create', methods: ['POST'])]
    public function create(Request $request): Response
    {
        $data = json_decode($request->getContent(), true);
        // ...
        return $this->json(['id' => 1], 201);
    }
}
```

```bash
# Voir toutes les routes
php bin/console debug:router

# Voir les routes qui matchent une URL
php bin/console router:match /articles/42
```

---

## Doctrine ORM

Doctrine ORM mappe les classes PHP vers des tables de base de données.

### Définir une Entité

```php
<?php
// src/Entity/Article.php
namespace App\Entity;

use App\Repository\ArticleRepository;
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ArticleRepository::class)]
#[ORM\Table(name: 'articles')]
class Article
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    private string $titre;

    #[ORM\Column(type: 'text')]
    private string $contenu;

    #[ORM\Column]
    private \DateTimeImmutable $createdAt;

    // Relation ManyToOne : un article a un auteur
    #[ORM\ManyToOne(targetEntity: User::class, inversedBy: 'articles')]
    #[ORM\JoinColumn(nullable: false)]
    private User $auteur;

    // Relation OneToMany : un article a plusieurs commentaires
    #[ORM\OneToMany(targetEntity: Commentaire::class, mappedBy: 'article', cascade: ['persist', 'remove'])]
    private Collection $commentaires;

    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
        $this->commentaires = new ArrayCollection();
    }

    // Getters et Setters...
    public function getId(): ?int { return $this->id; }
    public function getTitre(): string { return $this->titre; }
    public function setTitre(string $titre): static
    {
        $this->titre = $titre;
        return $this; // Fluent interface
    }
}
```

### Migrations

```bash
# Créer une migration depuis les changements d'entités
php bin/console make:migration

# Appliquer les migrations
php bin/console doctrine:migrations:migrate

# Voir le statut des migrations
php bin/console doctrine:migrations:status
```

### Repository — Requêtes Personnalisées

```php
<?php
// src/Repository/ArticleRepository.php
namespace App\Repository;

use App\Entity\Article;
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;

class ArticleRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Article::class);
    }

    // Méthode personnalisée avec Query Builder
    public function findPubliesRecents(int $limite = 10): array
    {
        return $this->createQueryBuilder('a')
            ->andWhere('a.publie = :publie')
            ->setParameter('publie', true)
            ->orderBy('a.createdAt', 'DESC')
            ->setMaxResults($limite)
            ->getQuery()
            ->getResult();
    }

    // Avec DQL (Doctrine Query Language)
    public function findByAuteur(string $email): array
    {
        return $this->createQueryBuilder('a')
            ->join('a.auteur', 'u')
            ->andWhere('u.email = :email')
            ->setParameter('email', $email)
            ->getQuery()
            ->getResult();
    }
    
    // Recherche full-text simple
    public function search(string $terme): array
    {
        return $this->createQueryBuilder('a')
            ->andWhere('a.titre LIKE :terme OR a.contenu LIKE :terme')
            ->setParameter('terme', "%$terme%")
            ->getQuery()
            ->getResult();
    }
}
```

---

## Templates Twig

```twig
{# templates/article/index.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}Articles{% endblock %}

{% block body %}
    <h1>Tous les articles</h1>
    
    {# Condition #}
    {% if articles is empty %}
        <p>Aucun article pour le moment.</p>
    {% else %}
        {# Boucle #}
        {% for article in articles %}
            <article class="card">
                <h2>
                    {# Génération d'URL depuis le nom de route #}
                    <a href="{{ path('article_show', {id: article.id}) }}">
                        {{ article.titre }}
                    </a>
                </h2>
                <p class="meta">
                    Par {{ article.auteur.prenom }}
                    le {{ article.createdAt|date('d/m/Y') }}
                </p>
                {# Troncature de texte #}
                <p>{{ article.contenu|slice(0, 200) }}...</p>
            </article>
        {% endfor %}
    {% endif %}
    
    {# Pagination #}
    {{ knp_pagination_render(articles) }}
{% endblock %}
```

```twig
{# templates/base.html.twig #}
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Mon Site{% endblock %}</title>
    {# Import des assets CSS compilés par Webpack Encore #}
    {{ encore_entry_link_tags('app') }}
</head>
<body>
    <nav>
        {# Vérifier si l'utilisateur est connecté #}
        {% if is_granted('ROLE_USER') %}
            <a href="{{ path('article_create') }}">Nouvel article</a>
            <a href="{{ path('app_logout') }}">Déconnexion ({{ app.user.email }})</a>
        {% else %}
            <a href="{{ path('app_login') }}">Connexion</a>
        {% endif %}
    </nav>
    
    {# Flash messages #}
    {% for type, messages in app.flashes %}
        {% for message in messages %}
            <div class="alert alert-{{ type }}">{{ message }}</div>
        {% endfor %}
    {% endfor %}
    
    <main>
        {% block body %}{% endblock %}
    </main>
    
    {{ encore_entry_script_tags('app') }}
</body>
</html>
```

---

## Formulaires Symfony

```php
<?php
// src/Form/ArticleType.php
namespace App\Form;

use App\Entity\Article;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints as Assert;

class ArticleType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('titre', TextType::class, [
                'label' => 'Titre de l\'article',
                'constraints' => [
                    new Assert\NotBlank(message: 'Le titre est obligatoire'),
                    new Assert\Length(min: 5, max: 255),
                ],
            ])
            ->add('contenu', TextareaType::class, [
                'label' => 'Contenu',
                'attr' => ['rows' => 10],
                'constraints' => [
                    new Assert\NotBlank(),
                    new Assert\Length(min: 50),
                ],
            ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults(['data_class' => Article::class]);
    }
}

// Utilisation dans le controller
#[Route('/articles/new', name: 'article_new', methods: ['GET', 'POST'])]
public function new(Request $request, EntityManagerInterface $em): Response
{
    $article = new Article();
    $form = $this->createForm(ArticleType::class, $article);
    $form->handleRequest($request);
    
    if ($form->isSubmitted() && $form->isValid()) {
        $article->setAuteur($this->getUser());
        $em->persist($article);
        $em->flush();
        
        $this->addFlash('success', 'Article créé avec succès !');
        return $this->redirectToRoute('article_show', ['id' => $article->getId()]);
    }
    
    return $this->render('article/new.html.twig', [
        'form' => $form,
    ]);
}
```

---

## Sécurité — JWT avec LexikJWT

```bash
composer require lexik/jwt-authentication-bundle
php bin/console lexik:jwt:generate-keypair
```

```yaml
# config/packages/lexik_jwt_authentication.yaml
lexik_jwt_authentication:
    secret_key: '%kernel.project_dir%/config/jwt/private.pem'
    public_key: '%kernel.project_dir%/config/jwt/public.pem'
    pass_phrase: '%env(JWT_PASSPHRASE)%'
    token_ttl: 3600  # 1 heure

# config/packages/security.yaml
security:
    firewalls:
        api:
            pattern: ^/api
            stateless: true
            jwt: ~   # Vérification automatique du token JWT
        
        login:
            pattern: ^/api/login
            stateless: true
            json_login:
                check_path: /api/login
                success_handler: lexik_jwt_authentication.handler.authentication_success
                failure_handler: lexik_jwt_authentication.handler.authentication_failure

    access_control:
        - { path: ^/api/login, roles: PUBLIC_ACCESS }
        - { path: ^/api, roles: ROLE_USER }
```

```bash
# Obtenir un token JWT
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"monpassword"}'

# Réponse : {"token":"eyJ0eXAiOiJKV1QiLCJhbGci..."}

# Utiliser le token
curl http://localhost:8000/api/articles \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGci..."
```

---

## API Platform — APIs Automatiques

API Platform génère automatiquement des APIs REST et GraphQL à partir de vos entités.

```bash
composer require api-platform/core
```

```php
<?php
// src/Entity/Article.php — transformer une entité en ressource API
namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;
use ApiPlatform\Metadata\Delete;
use ApiPlatform\Metadata\Get;
use ApiPlatform\Metadata\GetCollection;
use ApiPlatform\Metadata\Post;
use ApiPlatform\Metadata\Put;

#[ApiResource(
    operations: [
        new GetCollection(),
        new Get(),
        new Post(security: "is_granted('ROLE_USER')"),
        new Put(security: "is_granted('ROLE_USER') and object.auteur == user"),
        new Delete(security: "is_granted('ROLE_ADMIN')"),
    ],
    normalizationContext: ['groups' => ['article:read']],
    denormalizationContext: ['groups' => ['article:write']],
)]
#[ORM\Entity(repositoryClass: ArticleRepository::class)]
class Article
{
    #[ORM\Id, ORM\GeneratedValue, ORM\Column]
    #[Groups(['article:read'])]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    #[Groups(['article:read', 'article:write'])]
    private string $titre;

    // ...
}
```

API Platform génère automatiquement :
- `GET /api/articles` → liste paginée
- `POST /api/articles` → création
- `GET /api/articles/{id}` → détail
- `PUT /api/articles/{id}` → modification
- `DELETE /api/articles/{id}` → suppression
- Interface Swagger/OpenAPI sur `/api`
- Endpoint GraphQL sur `/api/graphql`

---

## Messenger — Files de Messages

```bash
composer require symfony/messenger
```

```php
<?php
// src/Message/EnvoyerEmailMessage.php — le message
namespace App\Message;

class EnvoyerEmailMessage
{
    public function __construct(
        public readonly string $destinataire,
        public readonly string $sujet,
        public readonly string $corps,
    ) {}
}

// src/MessageHandler/EnvoyerEmailMessageHandler.php — le handler
namespace App\MessageHandler;

use App\Message\EnvoyerEmailMessage;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;
use Symfony\Component\Mailer\MailerInterface;

#[AsMessageHandler]
class EnvoyerEmailMessageHandler
{
    public function __construct(private MailerInterface $mailer) {}
    
    public function __invoke(EnvoyerEmailMessage $message): void
    {
        // Ce code s'exécute en arrière-plan (async)
        $email = new Email()
            ->to($message->destinataire)
            ->subject($message->sujet)
            ->text($message->corps);
        
        $this->mailer->send($email);
    }
}

// Dans un controller : dispatcher le message
$this->bus->dispatch(new EnvoyerEmailMessage(
    'client@example.com',
    'Confirmation de commande',
    'Votre commande #123 est confirmée.',
));
// La requête HTTP répond immédiatement, l'email part en arrière-plan
```

```bash
# Consommer la file d'attente
php bin/console messenger:consume async --limit=100 -vv
```

---

## Tests avec PHPUnit et Symfony

```php
<?php
// tests/Controller/ArticleControllerTest.php
namespace App\Tests\Controller;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class ArticleControllerTest extends WebTestCase
{
    private KernelBrowser $client;
    
    protected function setUp(): void
    {
        $this->client = static::createClient();
    }
    
    public function testListeArticles(): void
    {
        $this->client->request('GET', '/api/articles');
        
        $this->assertResponseIsSuccessful();
        $this->assertResponseHeaderSame('Content-Type', 'application/ld+json; charset=utf-8');
        
        $data = json_decode($this->client->getResponse()->getContent(), true);
        $this->assertArrayHasKey('hydra:member', $data);
    }
    
    public function testCreerArticleSansAuth(): void
    {
        $this->client->request('POST', '/api/articles', [], [], [
            'CONTENT_TYPE' => 'application/json',
        ], json_encode(['titre' => 'Test', 'contenu' => 'Contenu test']));
        
        $this->assertResponseStatusCodeSame(401);
    }
    
    public function testCreerArticleAvecAuth(): void
    {
        // Simuler un utilisateur connecté
        $user = $this->getTestUser();
        $this->client->loginUser($user);
        
        $this->client->request('POST', '/api/articles', [], [], [
            'CONTENT_TYPE' => 'application/json',
        ], json_encode([
            'titre'   => 'Mon article de test',
            'contenu' => 'Contenu suffisamment long pour passer la validation min:50',
        ]));
        
        $this->assertResponseStatusCodeSame(201);
        $data = json_decode($this->client->getResponse()->getContent(), true);
        $this->assertArrayHasKey('id', $data);
    }
}
```

---

## Déploiement

### PHP-FPM + Nginx

```nginx
# /etc/nginx/sites-available/monapp
server {
    listen 80;
    server_name monapp.com;
    root /var/www/monapp/public;

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
        internal;
    }

    location ~ \.php$ {
        return 404; # Bloquer l'exécution d'autres PHP
    }
}
```

```bash
# Déploiement Symfony production
composer install --no-dev --optimize-autoloader
php bin/console cache:clear --env=prod
php bin/console doctrine:migrations:migrate --no-interaction
```

### Docker

```dockerfile
# Dockerfile
FROM php:8.3-fpm-alpine

RUN docker-php-ext-install pdo pdo_mysql opcache
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

WORKDIR /var/www

COPY . .
RUN composer install --no-dev --optimize-autoloader
RUN php bin/console cache:warmup --env=prod

EXPOSE 9000
```

---

## Projet Complet : Blog API avec Symfony + API Platform

```bash
# Créer le projet
composer create-project symfony/skeleton blog-api
cd blog-api
composer require api-platform/core symfony/orm-pack lexik/jwt-authentication-bundle symfony/security-bundle symfony/validator

# Créer les entités
php bin/console make:entity User
php bin/console make:entity Article
php bin/console make:entity Commentaire

# Générer les migrations
php bin/console make:migration
php bin/console doctrine:migrations:migrate

# Générer la paire de clés JWT
php bin/console lexik:jwt:generate-keypair

# Lancer le serveur de développement
symfony server:start
# → http://localhost:8000/api  (Swagger UI)
```

**Architecture du blog API :**
- `POST /api/login` → obtenir un JWT
- `GET /api/articles` → liste des articles (public)
- `POST /api/articles` → créer un article (ROLE_USER)
- `GET /api/articles/{id}` → détail (public)
- `PUT /api/articles/{id}` → modifier (auteur seulement)
- `DELETE /api/articles/{id}` → supprimer (admin seulement)
- `POST /api/commentaires` → commenter (ROLE_USER)

> [!info] Liens croisés
> - Pour les APIs REST en Python : [[08 - APIs REST avec Flask]] et [[09 - APIs REST avec FastAPI]]
> - Pour une API REST Java : [[03 - Spring Boot et API REST]]
> - Pour le déploiement en conteneur : [[01 - Docker]] et [[04 - CI-CD avec GitHub Actions]]

---

## Exercices Pratiques

### Exercice 1 — PHP Moderne (30 min)
Sans framework, créez un fichier PHP avec :
1. Un enum `Priorite` (Haute, Moyenne, Basse) avec une méthode `couleur(): string`
2. Une classe `Tache` avec tous les types stricts
3. Une fonction qui prend une `Tache|null` et utilise le null safe operator
4. Un exemple de `match` avec au moins 4 branches

### Exercice 2 — Entités et Relations Doctrine (1h)
Créez les entités pour un système de bibliothèque :
- `Livre` (titre, isbn, annee, disponible)
- `Auteur` (nom, prenom, nationalite) — ManyToMany avec Livre
- `Emprunt` (livre, membre, dateDebut, dateRetourPrevue, dateRetourReelle)
- `Membre` (nom, email, dateInscription)

Écrivez les migrations, créez le schéma et ajoutez 5 enregistrements de test via des fixtures Doctrine.

### Exercice 3 — API REST complète (2h)
Créez une mini-API de gestion de tâches avec Symfony + API Platform :
1. Entité `Tache` (titre, description, statut enum, priorité enum, dateEcheance)
2. Authentication JWT pour la création/modification/suppression
3. Filtre par statut et par priorité dans l'URL (`/api/taches?statut=en_cours`)
4. Validation : titre min 5 chars, dateEcheance dans le futur
5. Tests avec WebTestCase pour les endpoints principaux

### Exercice 4 — Messenger et Twig (1h)
Étendez votre API de tâches :
1. Quand une tâche est créée, envoyer un message Messenger
2. Le handler logge la création dans un fichier `var/log/tasks.log`
3. Créer un template Twig qui liste les tâches (page web non-API)
4. Le template hérite d'un `base.html.twig` avec un nav et une structure Bootstrap

> [!info] Ressources PHP/Symfony
> - **Documentation** : symfony.com/doc (excellente, avec les best practices officielles)
> - **SymfonyCasts** : tutoriels vidéo officiels Symfony (payant mais de qualité)
> - **API Platform** : api-platform.com/docs
> - **Doctrine** : doctrine-project.org/projects/doctrine-orm/en/3.2
> - **php.net** : référence officielle PHP 8.x
