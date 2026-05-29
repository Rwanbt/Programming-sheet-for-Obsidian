# PHP POO et Base de Donnees

> [!tip] Analogie
> Imagine que tu construis une ville. Les **classes** sont des plans d'architecte (un plan pour "Maison", un plan pour "Immeuble"). Les **objets** sont les batiments reels construits depuis ces plans. Une **interface** est un contrat : "tout batiment public DOIT avoir une sortie de secours". Un **trait** est un kit de modules reutilisables : le "kit climatisation" peut s'installer dans une maison OU un immeuble. La **base de donnees** c'est le cadastre : un registre persistant qui garde trace de tous les batiments meme apres qu'on eteint les lumieres.

Liens complementaires : [[PHP/01 - PHP et Symfony]] · [[SQL/01 - Introduction au SQL]] · [[PHP/03 - Laravel Framework]]

---

## La POO en PHP : rappel rapide

Avant d'aller dans les concepts avances, on s'assure que les bases sont solides. Si tu sais deja ce qu'est `__construct`, `extends` et `$this`, tu peux sauter au prochain `##`.

### Classes, objets, constructeur

```php
<?php
declare(strict_types=1);

class Utilisateur {
    public string $nom;
    public string $email;
    private int $age;

    public function __construct(string $nom, string $email, int $age) {
        $this->nom   = $nom;
        $this->email = $email;
        $this->age   = $age;
    }

    public function sePresenter(): string {
        return "Bonjour, je suis {$this->nom} ({$this->email})";
    }

    public function getAge(): int {
        return $this->age;
    }
}

$user = new Utilisateur('Alice', 'alice@example.com', 28);
echo $user->sePresenter(); // Bonjour, je suis Alice (alice@example.com)
```

> [!info] `declare(strict_types=1)`
> On met ca **en haut de chaque fichier PHP**. Cela force PHP a verifier les types passes aux fonctions. Sans ca, PHP convertit silencieusement `"42"` (string) en `42` (int) sans avertissement. Avec strict_types, il leve une `TypeError`. Toujours l'activer sur du code serieux.

### Visibilite : public, protected, private

| Modificateur | Accessible depuis | Accessible dans sous-classe |
|---|---|---|
| `public` | Partout | Oui |
| `protected` | Classe + sous-classes | Oui |
| `private` | Classe uniquement | Non |

### Heritage basique

```php
<?php
declare(strict_types=1);

class Animal {
    public function __construct(protected string $nom) {}

    public function parler(): string {
        return "...";
    }
}

class Chien extends Animal {
    public function parler(): string {
        return "{$this->nom} dit : Waf !";
    }
}

$rex = new Chien('Rex');
echo $rex->parler(); // Rex dit : Waf !
```

---

## POO PHP avancee

### Interfaces : le contrat

Une interface definit **ce qu'une classe doit faire**, sans dire **comment**. C'est un contrat pur. Une classe peut implementer plusieurs interfaces.

```php
<?php
declare(strict_types=1);

interface Sauvegardable {
    public function sauvegarder(): bool;
    public function charger(int $id): static;
}

interface Validable {
    public function estValide(): bool;
    public function getErreurs(): array;
}

// Une classe peut implementer plusieurs interfaces
class Produit implements Sauvegardable, Validable {
    private array $erreurs = [];

    public function __construct(
        private string $nom,
        private float  $prix
    ) {}

    public function sauvegarder(): bool {
        // logique de sauvegarde...
        return true;
    }

    public function charger(int $id): static {
        // logique de chargement...
        return $this;
    }

    public function estValide(): bool {
        $this->erreurs = [];
        if (empty($this->nom)) {
            $this->erreurs[] = "Le nom est obligatoire";
        }
        if ($this->prix < 0) {
            $this->erreurs[] = "Le prix ne peut pas etre negatif";
        }
        return empty($this->erreurs);
    }

    public function getErreurs(): array {
        return $this->erreurs;
    }
}
```

> [!warning] Interface vs Classe abstraite
> - **Interface** : que des signatures de methodes, pas d'implementation. Une classe peut en implementer plusieurs.
> - **Classe abstraite** : peut avoir des methodes concretes (avec code). Une classe ne peut heriter que d'une seule classe abstraite. Choix : interface si tu veux un contrat pur ; classe abstraite si tu veux partager du code commun.

### Traits : code reutilisable horizontal

Un trait est un mecanisme de **reutilisation de code** qui contourne la limitation de l'heritage simple. Il ressemble a une classe mais ne peut pas etre instancie seul.

```php
<?php
declare(strict_types=1);

trait Horodatable {
    private ?DateTimeImmutable $createdAt = null;
    private ?DateTimeImmutable $updatedAt = null;

    public function marquerCreation(): void {
        $this->createdAt = new DateTimeImmutable();
        $this->updatedAt = new DateTimeImmutable();
    }

    public function marquerModification(): void {
        $this->updatedAt = new DateTimeImmutable();
    }

    public function getCreatedAt(): ?DateTimeImmutable {
        return $this->createdAt;
    }

    public function getUpdatedAt(): ?DateTimeImmutable {
        return $this->updatedAt;
    }
}

trait Serialisable {
    public function toArray(): array {
        return get_object_vars($this);
    }

    public function toJson(): string {
        return json_encode($this->toArray(), JSON_THROW_ON_ERROR);
    }
}

class Article {
    use Horodatable, Serialisable; // On "colle" les deux traits

    public function __construct(
        private string $titre,
        private string $contenu
    ) {
        $this->marquerCreation();
    }
}

$article = new Article('Mon titre', 'Mon contenu...');
echo $article->getCreatedAt()->format('Y-m-d H:i:s');
echo $article->toJson();
```

> [!info] Quand utiliser un trait ?
> Un trait est ideal quand la meme logique (logging, timestamps, serialisation...) doit apparaitre dans plusieurs classes qui n'ont pas de relation d'heritage entre elles. Evite les traits qui font trop de choses differentes — garder chaque trait focalise sur une seule responsabilite.

### Classes abstraites

Une classe abstraite ne peut pas etre instanciee directement. Elle sert de **modele partiel** que les sous-classes completent.

```php
<?php
declare(strict_types=1);

abstract class FormeGeometrique {
    // Methode abstraite : les sous-classes DOIVENT l'implementer
    abstract public function surface(): float;
    abstract public function perimetre(): float;

    // Methode concrete : disponible pour toutes les sous-classes
    public function decrire(): string {
        return sprintf(
            "%s — surface: %.2f, perimetre: %.2f",
            static::class,
            $this->surface(),
            $this->perimetre()
        );
    }
}

class Cercle extends FormeGeometrique {
    public function __construct(private float $rayon) {}

    public function surface(): float {
        return M_PI * $this->rayon ** 2;
    }

    public function perimetre(): float {
        return 2 * M_PI * $this->rayon;
    }
}

class Rectangle extends FormeGeometrique {
    public function __construct(
        private float $largeur,
        private float $hauteur
    ) {}

    public function surface(): float {
        return $this->largeur * $this->hauteur;
    }

    public function perimetre(): float {
        return 2 * ($this->largeur + $this->hauteur);
    }
}

$formes = [new Cercle(5.0), new Rectangle(4.0, 6.0)];
foreach ($formes as $forme) {
    echo $forme->decrire() . "\n";
}
```

### Methodes et proprietes statiques

Les membres `static` appartiennent a la **classe** et non a une instance. Ils existent sans `new`.

```php
<?php
declare(strict_types=1);

class Compteur {
    private static int $instances = 0;

    public function __construct() {
        self::$instances++;
    }

    public static function getInstances(): int {
        return self::$instances;
    }

    public static function reinitialiser(): void {
        self::$instances = 0;
    }
}

new Compteur();
new Compteur();
new Compteur();
echo Compteur::getInstances(); // 3
```

> [!warning] `self::` vs `static::`
> - `self::` fait reference a la classe ou la methode est **definie** (liaison statique precoce).
> - `static::` fait reference a la classe qui est **appelee** au runtime (liaison statique tardive, Late Static Binding).
> En cas de doute sur l'heritage, prefer `static::`.

### Le mot-cle `final`

`final` sur une **classe** interdit l'heritage. `final` sur une **methode** interdit la surcharge dans les sous-classes.

```php
<?php
declare(strict_types=1);

final class ServiceCryptographie {
    public function hasher(string $password): string {
        return password_hash($password, PASSWORD_BCRYPT);
    }
}

// Erreur fatale : Cannot extend final class ServiceCryptographie
// class MauvaisHasher extends ServiceCryptographie {}
```

> [!info] Pourquoi `final` ?
> Utilise `final` quand tu veux garantir qu'une implementation ne sera jamais modifiee par heritage. Utile pour les classes de securite, les Value Objects, et les classes utilitaires qui sont des implementations concretes sans logique d'extension prevue.

---

## Namespaces et autoloading PSR-4

### Le probleme sans namespaces

Imagine deux librairies qui definissent toutes les deux une classe `User`. Sans namespaces, collision garantie.

```
App\Models\User  →  vendor/monlib/User
```

Avec les namespaces, chaque classe a un chemin unique.

### Declarer et utiliser un namespace

```php
<?php
// Fichier : src/Models/Utilisateur.php
declare(strict_types=1);

namespace App\Models;

class Utilisateur {
    public function __construct(
        public readonly string $nom,
        public readonly string $email
    ) {}
}
```

```php
<?php
// Fichier : src/Services/UtilisateurService.php
declare(strict_types=1);

namespace App\Services;

use App\Models\Utilisateur;         // import explicite
use App\Repository\UserRepository;
use PDO;                             // classe PHP globale, besoin du backslash ou use

class UtilisateurService {
    public function __construct(
        private UserRepository $repository
    ) {}

    public function creer(string $nom, string $email): Utilisateur {
        $user = new Utilisateur($nom, $email);
        $this->repository->save($user);
        return $user;
    }
}
```

### PSR-4 : la convention de mapping namespace → dossier

PSR-4 definit une correspondance simple :

```
Namespace prefix    →  Base directory
App\                →  src/
App\Models\         →  src/Models/
App\Services\       →  src/Services/
```

Un fichier qui declare `namespace App\Models\Utilisateur` **doit** se trouver dans `src/Models/Utilisateur.php`.

### Structure typique d'un projet

```
mon-projet/
├── composer.json
├── composer.lock
├── src/
│   ├── Models/
│   │   └── Utilisateur.php          (namespace App\Models)
│   ├── Services/
│   │   └── UtilisateurService.php   (namespace App\Services)
│   └── Repository/
│       └── UserRepository.php       (namespace App\Repository)
├── tests/
│   └── UserTest.php
└── vendor/                          (genere par Composer)
    └── autoload.php
```

---

## Composer : gestionnaire de dependances

Composer est a PHP ce que `npm` est a Node.js ou `cargo` a Rust.

### Installation

```bash
# Linux/macOS
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Windows : telecharger Composer-Setup.exe depuis getcomposer.org
```

### composer.json : le fichier de configuration

```json
{
    "name": "mon-entreprise/mon-projet",
    "description": "Mon application PHP",
    "type": "project",
    "require": {
        "php": ">=8.2",
        "monolog/monolog": "^3.0",
        "vlucas/phpdotenv": "^5.5"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0",
        "phpstan/phpstan": "^1.10"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    }
}
```

### Commandes essentielles

| Commande | Effet |
|---|---|
| `composer install` | Installe les dependances depuis `composer.lock` |
| `composer update` | Met a jour les dependances selon `composer.json` |
| `composer require vendor/package` | Ajoute une dependance et met a jour `composer.json` |
| `composer require --dev vendor/package` | Dependance de dev uniquement |
| `composer remove vendor/package` | Supprime une dependance |
| `composer dump-autoload` | Regenere l'autoloader (apres ajout de classe) |
| `composer show` | Liste les paquets installes |
| `composer validate` | Verifie la syntaxe du `composer.json` |

### Utiliser l'autoloader

```php
<?php
// index.php ou bootstrap.php
require_once __DIR__ . '/vendor/autoload.php';

// Maintenant toutes les classes de src/ sont disponibles
use App\Models\Utilisateur;
use App\Services\UtilisateurService;

$user = new Utilisateur('Bob', 'bob@example.com');
```

> [!warning] Ne jamais committer le dossier `vendor/`
> Ajouter `vendor/` au `.gitignore`. Le dossier peut etre regenre a tout moment avec `composer install`. Committer `vendor/` alourdit le repo inutilement.

---

## PDO : acces a la base de donnees

PDO (PHP Data Objects) est l'abstraction standard de PHP pour parler a une base de donnees. Il supporte MySQL, PostgreSQL, SQLite, etc. avec la meme interface.

### Connexion et configuration

```php
<?php
declare(strict_types=1);

namespace App\Database;

use PDO;
use PDOException;

class DatabaseConnection {
    private static ?PDO $instance = null;

    public static function getInstance(): PDO {
        if (self::$instance === null) {
            $dsn  = sprintf(
                'mysql:host=%s;port=%d;dbname=%s;charset=utf8mb4',
                $_ENV['DB_HOST'] ?? 'localhost',
                (int)($_ENV['DB_PORT'] ?? 3306),
                $_ENV['DB_NAME'] ?? 'ma_base'
            );
            $options = [
                PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                PDO::ATTR_EMULATE_PREPARES   => false,
            ];
            try {
                self::$instance = new PDO(
                    $dsn,
                    $_ENV['DB_USER'] ?? 'root',
                    $_ENV['DB_PASS'] ?? '',
                    $options
                );
            } catch (PDOException $e) {
                // En production, ne jamais afficher les details de la PDOException !
                throw new \RuntimeException("Connexion base de donnees impossible", 0, $e);
            }
        }
        return self::$instance;
    }

    // Interdit l'instanciation externe
    private function __construct() {}
    private function __clone() {}
}
```

> [!info] Options PDO importantes
> - `ERRMODE_EXCEPTION` : PDO lance des exceptions au lieu de retourner `false`. **Toujours activer.**
> - `FETCH_ASSOC` : les resultats sont des tableaux associatifs par defaut (plus lisible que les indices numeriques).
> - `EMULATE_PREPARES => false` : force le driver a utiliser de vraies requetes preparees cote serveur. Plus sur.

### Prepared Statements : la securite d'abord

Un prepared statement separe les donnees de la requete SQL. C'est la defense numero 1 contre les injections SQL.

```php
<?php
declare(strict_types=1);

use App\Database\DatabaseConnection;

$pdo = DatabaseConnection::getInstance();

// JAMAIS FAIRE CA : injection SQL possible !
// $sql = "SELECT * FROM users WHERE email = '$email'";

// TOUJOURS utiliser des placeholders
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email AND actif = :actif");
$stmt->bindValue(':email',  $email,  PDO::PARAM_STR);
$stmt->bindValue(':actif',  true,    PDO::PARAM_BOOL);
$stmt->execute();

$user = $stmt->fetch(); // tableau associatif ou false
```

Ou la version plus concise avec `execute()` :

```php
<?php
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ? AND actif = ?");
$stmt->execute([$email, 1]);
$user = $stmt->fetch();
```

### Methodes de recuperation

```php
<?php
declare(strict_types=1);

$pdo = DatabaseConnection::getInstance();

// --- Un seul enregistrement ---
$stmt = $pdo->prepare("SELECT * FROM produits WHERE id = :id");
$stmt->execute([':id' => 42]);
$produit = $stmt->fetch(PDO::FETCH_ASSOC); // tableau ou false

// --- Tous les enregistrements ---
$stmt = $pdo->query("SELECT * FROM produits WHERE actif = 1");
$produits = $stmt->fetchAll(PDO::FETCH_ASSOC); // tableau de tableaux

// --- Recuperer comme objet ---
$stmt = $pdo->prepare("SELECT * FROM produits WHERE id = :id");
$stmt->execute([':id' => 42]);
$produit = $stmt->fetchObject(Produit::class); // instancie Produit et hydrate les proprietes

// --- Tableau de tous les objets ---
$stmt = $pdo->query("SELECT * FROM produits");
$produits = $stmt->fetchAll(PDO::FETCH_CLASS, Produit::class);

// --- Une seule colonne ---
$stmt = $pdo->query("SELECT COUNT(*) FROM produits");
$count = $stmt->fetchColumn(); // retourne un scalaire
```

| Methode | Retourne |
|---|---|
| `fetch()` | Un enregistrement (tableau/objet) ou `false` |
| `fetchAll()` | Tous les enregistrements (tableau) |
| `fetchObject(Class)` | Un objet de la classe specifiee |
| `fetchColumn(n)` | La valeur de la colonne n (defaut: 0) |

### INSERT, UPDATE, DELETE

```php
<?php
declare(strict_types=1);

$pdo = DatabaseConnection::getInstance();

// INSERT
$stmt = $pdo->prepare(
    "INSERT INTO users (nom, email, created_at) VALUES (:nom, :email, NOW())"
);
$stmt->execute([
    ':nom'   => 'Claire',
    ':email' => 'claire@example.com',
]);
$newId = (int)$pdo->lastInsertId(); // ID auto-incremente

// UPDATE
$stmt = $pdo->prepare(
    "UPDATE users SET nom = :nom, updated_at = NOW() WHERE id = :id"
);
$stmt->execute([':nom' => 'Claire Martin', ':id' => $newId]);
$rowsAffected = $stmt->rowCount(); // nombre de lignes modifiees

// DELETE
$stmt = $pdo->prepare("DELETE FROM users WHERE id = :id");
$stmt->execute([':id' => $newId]);
```

### Transactions

Les transactions garantissent l'**atomicite** : soit toutes les operations reussissent, soit aucune n'est appliquee.

```php
<?php
declare(strict_types=1);

$pdo = DatabaseConnection::getInstance();

try {
    $pdo->beginTransaction();

    // Operation 1 : debit
    $stmt = $pdo->prepare("UPDATE comptes SET solde = solde - :montant WHERE id = :id");
    $stmt->execute([':montant' => 100.00, ':id' => 1]);

    // Operation 2 : credit
    $stmt = $pdo->prepare("UPDATE comptes SET solde = solde + :montant WHERE id = :id");
    $stmt->execute([':montant' => 100.00, ':id' => 2]);

    // Tout s'est passe : valider
    $pdo->commit();
    echo "Virement effectue avec succes.";

} catch (\Throwable $e) {
    // Annuler toutes les modifications si une etape echoue
    $pdo->rollBack();
    throw new \RuntimeException("Virement echoue : " . $e->getMessage(), 0, $e);
}
```

> [!warning] Toujours `rollBack()` dans le catch
> Si tu oublies `rollBack()` dans le bloc catch, la transaction reste ouverte et peut bloquer d'autres requetes. Pattern a retenir : `beginTransaction` → try → `commit` → catch → `rollBack`.

---

## Gestion des erreurs et exceptions

### Hierarchie des exceptions

```
Throwable
├── Error               (erreurs PHP internes — ne pas catch en general)
│   ├── TypeError
│   ├── ParseError
│   └── ArithmeticError
└── Exception           (exceptions applicatives — a catch)
    ├── RuntimeException
    ├── InvalidArgumentException
    ├── LogicException
    └── ... tes exceptions metier
```

### Exceptions personnalisees

```php
<?php
declare(strict_types=1);

namespace App\Exceptions;

// Exception de base applicative
class AppException extends \RuntimeException {}

// Exceptions metier derivees
class UtilisateurNonTrouveException extends AppException {
    public function __construct(int $id) {
        parent::__construct("Utilisateur #$id introuvable", 404);
    }
}

class EmailDejaUtiliseException extends AppException {
    public function __construct(string $email) {
        parent::__construct("L'email '$email' est deja utilise", 409);
    }
}

class ValidationException extends AppException {
    private array $erreurs;

    public function __construct(array $erreurs) {
        parent::__construct("Erreur de validation", 422);
        $this->erreurs = $erreurs;
    }

    public function getErreurs(): array {
        return $this->erreurs;
    }
}
```

### try / catch / finally

```php
<?php
declare(strict_types=1);

use App\Exceptions\UtilisateurNonTrouveException;
use App\Exceptions\ValidationException;

function traiterDemande(int $userId, array $data): void {
    try {
        $user = $repository->findOrFail($userId);
        $user->update($data);

    } catch (UtilisateurNonTrouveException $e) {
        // Erreur specifique
        http_response_code($e->getCode());
        echo json_encode(['erreur' => $e->getMessage()]);

    } catch (ValidationException $e) {
        // Plusieurs erreurs de validation
        http_response_code(422);
        echo json_encode(['erreurs' => $e->getErreurs()]);

    } catch (\Throwable $e) {
        // Filet de securite — erreur inattendue
        error_log("Erreur non prevue : " . $e->getMessage());
        http_response_code(500);
        echo json_encode(['erreur' => 'Erreur serveur interne']);

    } finally {
        // Toujours execute, meme si une exception est levee
        // Utilise pour liberer des ressources (connexion, fichier...)
        $pdo = null; // ferme la connexion
    }
}
```

### Handler global d'exceptions

```php
<?php
declare(strict_types=1);

// bootstrap.php — charge en tout premier
set_exception_handler(function (\Throwable $e): void {
    // Log structuree
    error_log(sprintf(
        "[EXCEPTION] %s: %s in %s:%d\nTrace:\n%s",
        get_class($e),
        $e->getMessage(),
        $e->getFile(),
        $e->getLine(),
        $e->getTraceAsString()
    ));

    // Reponse HTTP propre
    if (!headers_sent()) {
        http_response_code(500);
        header('Content-Type: application/json');
    }
    echo json_encode(['erreur' => 'Une erreur inattendue s\'est produite.']);
    exit(1);
});

set_error_handler(function (int $severity, string $message, string $file, int $line): bool {
    throw new \ErrorException($message, 0, $severity, $file, $line);
});
```

---

## PHP 8.x : fonctionnalites modernes

PHP 8.0, 8.1, 8.2 et 8.3 ont apporte des ameliorations majeures. Voici les plus importantes.

### Constructor Promotion (PHP 8.0)

Avant PHP 8, declarer une propriete via le constructeur etait verbeux :

```php
// Avant PHP 8 — 4 lignes pour chaque propriete
class Point {
    private float $x;
    private float $y;

    public function __construct(float $x, float $y) {
        $this->x = $x;
        $this->y = $y;
    }
}
```

Avec constructor promotion :

```php
// PHP 8.0+ — tout en une ligne
class Point {
    public function __construct(
        private float $x,
        private float $y,
    ) {}
}
```

### Named Arguments (PHP 8.0)

Les arguments nommes permettent de passer les valeurs dans n'importe quel ordre et d'ignorer les arguments optionnels intermediaires.

```php
<?php
// Avant : ordre obligatoire, aucune clarte
array_slice($tableau, 0, 10, true);

// Apres : lisible immediatement
array_slice(array: $tableau, offset: 0, length: 10, preserve_keys: true);

// Utile avec les constructeurs a nombreux parametres
$user = new Utilisateur(
    nom: 'Alice',
    email: 'alice@example.com',
    role: 'admin',
);
```

### Match Expression (PHP 8.0)

`match` est un `switch` ameliore : pas de `break`, comparaison stricte, et c'est une expression (retourne une valeur).

```php
<?php
declare(strict_types=1);

$statut = 'actif';

// switch classique — boilerplate, comparaison lache
$libelle = match($statut) {
    'actif'    => 'Compte actif',
    'inactif'  => 'Compte desactive',
    'banni'    => 'Compte banni',
    'en_attente', 'pending' => 'En attente de validation', // plusieurs cas
    default    => 'Statut inconnu',
};

// Match peut aussi lever une exception
$priorite = match(true) {
    $score >= 90 => 'A',
    $score >= 80 => 'B',
    $score >= 70 => 'C',
    $score >= 60 => 'D',
    default      => 'F',
};
```

### Nullsafe Operator `?->` (PHP 8.0)

Evite les cascades de `if ($x !== null)` quand on chaine des methodes sur des valeurs potentiellement nulles.

```php
<?php
// Avant — code defensif verbeux
$ville = null;
if ($commande !== null) {
    $client = $commande->getClient();
    if ($client !== null) {
        $adresse = $client->getAdresse();
        if ($adresse !== null) {
            $ville = $adresse->getVille();
        }
    }
}

// Avec le nullsafe operator — propre et lisible
$ville = $commande?->getClient()?->getAdresse()?->getVille();
// Retourne null des qu'un chainon est null, sans exception
```

### Enums (PHP 8.1)

Les enumerations remplacent les constantes magiques par des types verifiables.

```php
<?php
declare(strict_types=1);

// Enum pur (pas de valeur associee)
enum Direction {
    case Nord;
    case Sud;
    case Est;
    case Ouest;
}

// Backed Enum (avec une valeur string ou int)
enum StatutCommande: string {
    case EnAttente   = 'pending';
    case Confirmee   = 'confirmed';
    case Expediee    = 'shipped';
    case Livree      = 'delivered';
    case Annulee     = 'cancelled';

    // Les enums peuvent avoir des methodes
    public function libelle(): string {
        return match($this) {
            self::EnAttente  => 'En attente',
            self::Confirmee  => 'Confirmee',
            self::Expediee   => 'En cours de livraison',
            self::Livree     => 'Livree',
            self::Annulee    => 'Annulee',
        };
    }

    public function estTerminale(): bool {
        return in_array($this, [self::Livree, self::Annulee], strict: true);
    }
}

// Utilisation
$statut = StatutCommande::Expediee;
echo $statut->value;        // "shipped"
echo $statut->libelle();    // "En cours de livraison"

// Depuis une valeur brute (ex: depuis la BDD)
$statut = StatutCommande::from('confirmed'); // leve ValueError si invalide
$statut = StatutCommande::tryFrom('xyz');    // retourne null si invalide

// Typage dans les fonctions
function traiterCommande(StatutCommande $statut): void {
    if ($statut->estTerminale()) {
        throw new \LogicException("Commande deja terminee.");
    }
}
```

### Readonly properties (PHP 8.1) et Readonly classes (PHP 8.2)

```php
<?php
declare(strict_types=1);

// PHP 8.1 : proprietes readonly
class Coordonnees {
    public function __construct(
        public readonly float $latitude,
        public readonly float $longitude,
    ) {}
}

$point = new Coordonnees(48.8566, 2.3522);
echo $point->latitude; // 48.8566
// $point->latitude = 0; // Error: Cannot modify readonly property

// PHP 8.2 : classe entiere readonly (toutes les proprietes le sont)
readonly class MoneyAmount {
    public function __construct(
        public int $centimes,
        public string $devise,
    ) {}
}
```

### Fibers (PHP 8.1) — apercu

Les Fibers sont des coroutines legeres, base du code asynchrone en PHP (utilisees par Amphp, ReactPHP).

```php
<?php
declare(strict_types=1);

$fiber = new Fiber(function(): void {
    $valeur = Fiber::suspend('premiere suspension');
    echo "Reprise avec : $valeur\n";
    Fiber::suspend('deuxieme suspension');
    echo "Fiber terminee\n";
});

$resultat1 = $fiber->start();
echo "Fiber suspendue, valeur : $resultat1\n";  // "premiere suspension"

$resultat2 = $fiber->resume('bonjour');
echo "Fiber suspendue, valeur : $resultat2\n";  // "deuxieme suspension"

$fiber->resume();
```

> [!info] Fibers vs Async/Await
> Les Fibers ne sont pas elles-memes asynchrones — elles permettent de **suspendre** l'execution d'une fonction sans bloquer tout le process. Les frameworks async (Amphp, REVOLT) utilisent les Fibers pour simuler de l'async sur un seul thread.

---

## Standards PSR

Les PSR (PHP Standards Recommendations) sont des conventions etablies par le PHP-FIG. Les plus importantes :

### PSR-1 : Coding Standard de base

- Balises `<?php` uniquement (jamais `<?`)
- Fichiers en UTF-8 sans BOM
- Les noms de classes en `StudlyCaps` (PascalCase)
- Les constantes de classe en `MAJUSCULE_AVEC_UNDERSCORE`
- Les methodes en `camelCase`
- Un seul namespace ou une seule classe par fichier

### PSR-4 : Autoloading

Deja couverte dans la section Composer. Le namespace correspond au chemin du fichier.

```
App\Http\Controllers\UserController  →  src/Http/Controllers/UserController.php
```

### PSR-12 : Style guide etendu

```php
<?php
declare(strict_types=1);

namespace App\Services;        // ligne vide apres la declaration de namespace

use App\Models\Utilisateur;   // tous les use en premier
use PDO;

class MonService extends BaseService implements ServiceInterface {
    // une ligne vide apres l'ouverture de classe
    private const int MAX_ESSAIS = 3; // PHP 8.3 : typed class constants

    public function __construct(
        private readonly PDO $pdo,
        private readonly int $timeout = 30,
    ) {
    }                          // fermeture sur sa propre ligne

    public function trouver(int $id): ?Utilisateur {
        // corps de methode
        return null;
    }
}
```

| PSR | Sujet | Essentiel |
|---|---|---|
| PSR-1 | Coding standard basique | Nommage, encodage, une classe par fichier |
| PSR-4 | Autoloading | Namespace = chemin de fichier |
| PSR-7 | HTTP Message Interface | Objets Request/Response immuables |
| PSR-11 | Container Interface | Injection de dependances |
| PSR-12 | Extended Coding Style | Indentation, espaces, structure |

---

## Design Patterns en PHP

### Repository Pattern

Le Repository isole la couche de persistance. Le reste du code travaille avec des objets PHP, pas avec du SQL.

```
Controller → Service → Repository → PDO → Base de donnees
```

```php
<?php
declare(strict_types=1);

namespace App\Repository;

use App\Models\Utilisateur;

// Interface : le contrat
interface UtilisateurRepositoryInterface {
    public function findById(int $id): ?Utilisateur;
    public function findByEmail(string $email): ?Utilisateur;
    /** @return Utilisateur[] */
    public function findAll(): array;
    public function save(Utilisateur $user): Utilisateur;
    public function delete(int $id): void;
}
```

```php
<?php
declare(strict_types=1);

namespace App\Repository;

use App\Exceptions\UtilisateurNonTrouveException;
use App\Models\Utilisateur;
use PDO;

class PdoUtilisateurRepository implements UtilisateurRepositoryInterface {
    public function __construct(private readonly PDO $pdo) {}

    public function findById(int $id): ?Utilisateur {
        $stmt = $this->pdo->prepare(
            "SELECT id, nom, email, created_at FROM users WHERE id = :id"
        );
        $stmt->execute([':id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        return $row ? $this->hydrater($row) : null;
    }

    public function findOrFail(int $id): Utilisateur {
        $user = $this->findById($id);
        if ($user === null) {
            throw new UtilisateurNonTrouveException($id);
        }
        return $user;
    }

    public function findByEmail(string $email): ?Utilisateur {
        $stmt = $this->pdo->prepare(
            "SELECT id, nom, email, created_at FROM users WHERE email = :email"
        );
        $stmt->execute([':email' => $email]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        return $row ? $this->hydrater($row) : null;
    }

    /** @return Utilisateur[] */
    public function findAll(): array {
        $stmt = $this->pdo->query(
            "SELECT id, nom, email, created_at FROM users ORDER BY nom"
        );
        return array_map(
            fn(array $row) => $this->hydrater($row),
            $stmt->fetchAll(PDO::FETCH_ASSOC)
        );
    }

    public function save(Utilisateur $user): Utilisateur {
        if ($user->getId() === null) {
            return $this->inserer($user);
        }
        return $this->mettreAJour($user);
    }

    private function inserer(Utilisateur $user): Utilisateur {
        $stmt = $this->pdo->prepare(
            "INSERT INTO users (nom, email, created_at) VALUES (:nom, :email, NOW())"
        );
        $stmt->execute([
            ':nom'   => $user->getNom(),
            ':email' => $user->getEmail(),
        ]);
        return $user->withId((int)$this->pdo->lastInsertId());
    }

    private function mettreAJour(Utilisateur $user): Utilisateur {
        $stmt = $this->pdo->prepare(
            "UPDATE users SET nom = :nom, email = :email WHERE id = :id"
        );
        $stmt->execute([
            ':nom'   => $user->getNom(),
            ':email' => $user->getEmail(),
            ':id'    => $user->getId(),
        ]);
        return $user;
    }

    public function delete(int $id): void {
        $stmt = $this->pdo->prepare("DELETE FROM users WHERE id = :id");
        $stmt->execute([':id' => $id]);
    }

    private function hydrater(array $row): Utilisateur {
        return new Utilisateur(
            id: (int)$row['id'],
            nom: $row['nom'],
            email: $row['email'],
        );
    }
}
```

### Factory Pattern

La Factory centralise la creation d'objets complexes. Elle cache les details de construction.

```php
<?php
declare(strict_types=1);

namespace App\Factory;

use App\Models\Notification;

enum TypeNotification: string {
    case Email = 'email';
    case SMS   = 'sms';
    case Push  = 'push';
}

interface NotificationInterface {
    public function envoyer(string $destinataire, string $message): bool;
}

class EmailNotification implements NotificationInterface {
    public function envoyer(string $destinataire, string $message): bool {
        // logique d'envoi email...
        return true;
    }
}

class SmsNotification implements NotificationInterface {
    public function envoyer(string $destinataire, string $message): bool {
        // logique d'envoi SMS...
        return true;
    }
}

class NotificationFactory {
    public static function creer(TypeNotification $type): NotificationInterface {
        return match($type) {
            TypeNotification::Email => new EmailNotification(),
            TypeNotification::SMS   => new SmsNotification(),
            TypeNotification::Push  => new PushNotification(),
        };
    }
}

// Utilisation
$notif = NotificationFactory::creer(TypeNotification::Email);
$notif->envoyer('alice@example.com', 'Votre commande est confirmee.');
```

### Singleton et pourquoi l'eviter

Le Singleton garantit qu'une classe n'a qu'une seule instance. Il est souvent presente comme un "pattern" mais c'est en realite un **anti-pattern** dans la majorite des contextes modernes.

```php
<?php
// Implementation classique du Singleton
class Config {
    private static ?Config $instance = null;
    private array $data = [];

    private function __construct() {}

    public static function getInstance(): static {
        if (static::$instance === null) {
            static::$instance = new static();
        }
        return static::$instance;
    }

    public function get(string $key, mixed $default = null): mixed {
        return $this->data[$key] ?? $default;
    }
}

// Utilisation — semble pratique mais...
Config::getInstance()->get('db.host');
```

> [!warning] Pourquoi eviter le Singleton
> 1. **Etat global cache** : l'etat du Singleton est partagee entre toutes les parties de l'application. Un test modifie la config → il impacte le test suivant.
> 2. **Impossible a tester en isolation** : on ne peut pas remplacer le Singleton par un mock facilement.
> 3. **Couplage fort** : `Config::getInstance()` cree une dependance hard-coded impossible a injecter.
> 4. **Thread safety** : sur des serveurs asynchrones (Swoole, ReactPHP), le Singleton peut etre partage entre plusieurs requetes.
>
> **Alternative recommandee** : utiliser un container d'injection de dependances (DI Container) comme PHP-DI ou Symfony DI. La "singularite" d'une instance est geree par le container, pas par la classe elle-meme.

---

## Exemple complet : couche Repository avec PDO

Voici un exemple complet et realiste qui reunit tout ce qu'on a vu.

### Schema SQL

```sql
CREATE TABLE produits (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    nom        VARCHAR(255) NOT NULL,
    prix       DECIMAL(10, 2) NOT NULL,
    stock      INT NOT NULL DEFAULT 0,
    actif      TINYINT(1) NOT NULL DEFAULT 1,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_actif (actif)
);
```

### Le modele Produit

```php
<?php
declare(strict_types=1);

namespace App\Models;

use App\Exceptions\ValidationException;

final class Produit {
    private function __construct(
        private readonly ?int    $id,
        private string           $nom,
        private float            $prix,
        private int              $stock,
        private bool             $actif = true,
    ) {}

    public static function creer(string $nom, float $prix, int $stock): self {
        $erreurs = self::valider($nom, $prix, $stock);
        if (!empty($erreurs)) {
            throw new ValidationException($erreurs);
        }
        return new self(null, $nom, $prix, $stock);
    }

    public static function reconstituer(
        int $id, string $nom, float $prix, int $stock, bool $actif
    ): self {
        return new self($id, $nom, $prix, $stock, $actif);
    }

    private static function valider(string $nom, float $prix, int $stock): array {
        $erreurs = [];
        if (empty(trim($nom))) {
            $erreurs[] = "Le nom est obligatoire";
        }
        if ($prix < 0) {
            $erreurs[] = "Le prix doit etre positif";
        }
        if ($stock < 0) {
            $erreurs[] = "Le stock ne peut pas etre negatif";
        }
        return $erreurs;
    }

    // Getters
    public function getId(): ?int     { return $this->id; }
    public function getNom(): string  { return $this->nom; }
    public function getPrix(): float  { return $this->prix; }
    public function getStock(): int   { return $this->stock; }
    public function isActif(): bool   { return $this->actif; }

    // Commandes metier
    public function changerPrix(float $nouveauPrix): void {
        if ($nouveauPrix < 0) {
            throw new \InvalidArgumentException("Prix invalide : $nouveauPrix");
        }
        $this->prix = $nouveauPrix;
    }

    public function ajouterStock(int $quantite): void {
        if ($quantite <= 0) {
            throw new \InvalidArgumentException("Quantite invalide : $quantite");
        }
        $this->stock += $quantite;
    }

    public function desactiver(): void {
        $this->actif = false;
    }

    public function toArray(): array {
        return [
            'id'    => $this->id,
            'nom'   => $this->nom,
            'prix'  => $this->prix,
            'stock' => $this->stock,
            'actif' => $this->actif,
        ];
    }
}
```

### L'interface Repository

```php
<?php
declare(strict_types=1);

namespace App\Repository;

use App\Models\Produit;

interface ProduitRepositoryInterface {
    public function findById(int $id): ?Produit;
    /** @return Produit[] */
    public function findActifs(): array;
    /** @return Produit[] */
    public function findEnRuptureDeStock(): array;
    public function save(Produit $produit): Produit;
    public function delete(int $id): void;
}
```

### L'implementation PDO

```php
<?php
declare(strict_types=1);

namespace App\Repository;

use App\Models\Produit;
use PDO;

final class PdoProduitRepository implements ProduitRepositoryInterface {
    public function __construct(private readonly PDO $pdo) {}

    public function findById(int $id): ?Produit {
        $stmt = $this->pdo->prepare(
            "SELECT id, nom, prix, stock, actif FROM produits WHERE id = :id LIMIT 1"
        );
        $stmt->execute([':id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        return $row !== false ? $this->hydrater($row) : null;
    }

    /** @return Produit[] */
    public function findActifs(): array {
        $stmt = $this->pdo->query(
            "SELECT id, nom, prix, stock, actif FROM produits WHERE actif = 1 ORDER BY nom"
        );
        return array_map(
            fn(array $row) => $this->hydrater($row),
            $stmt->fetchAll(PDO::FETCH_ASSOC)
        );
    }

    /** @return Produit[] */
    public function findEnRuptureDeStock(): array {
        $stmt = $this->pdo->query(
            "SELECT id, nom, prix, stock, actif FROM produits WHERE stock = 0 AND actif = 1"
        );
        return array_map(
            fn(array $row) => $this->hydrater($row),
            $stmt->fetchAll(PDO::FETCH_ASSOC)
        );
    }

    public function save(Produit $produit): Produit {
        return $produit->getId() === null
            ? $this->inserer($produit)
            : $this->mettreAJour($produit);
    }

    private function inserer(Produit $produit): Produit {
        $stmt = $this->pdo->prepare(
            "INSERT INTO produits (nom, prix, stock, actif) VALUES (:nom, :prix, :stock, :actif)"
        );
        $stmt->execute([
            ':nom'   => $produit->getNom(),
            ':prix'  => $produit->getPrix(),
            ':stock' => $produit->getStock(),
            ':actif' => (int)$produit->isActif(),
        ]);
        $newId = (int)$this->pdo->lastInsertId();
        return Produit::reconstituer(
            $newId,
            $produit->getNom(),
            $produit->getPrix(),
            $produit->getStock(),
            $produit->isActif(),
        );
    }

    private function mettreAJour(Produit $produit): Produit {
        $stmt = $this->pdo->prepare(
            "UPDATE produits
             SET nom = :nom, prix = :prix, stock = :stock, actif = :actif
             WHERE id = :id"
        );
        $stmt->execute([
            ':nom'   => $produit->getNom(),
            ':prix'  => $produit->getPrix(),
            ':stock' => $produit->getStock(),
            ':actif' => (int)$produit->isActif(),
            ':id'    => $produit->getId(),
        ]);
        return $produit;
    }

    public function delete(int $id): void {
        $stmt = $this->pdo->prepare("DELETE FROM produits WHERE id = :id");
        $stmt->execute([':id' => $id]);
    }

    private function hydrater(array $row): Produit {
        return Produit::reconstituer(
            id:    (int)$row['id'],
            nom:   $row['nom'],
            prix:  (float)$row['prix'],
            stock: (int)$row['stock'],
            actif: (bool)$row['actif'],
        );
    }
}
```

### Le service metier

```php
<?php
declare(strict_types=1);

namespace App\Services;

use App\Models\Produit;
use App\Repository\ProduitRepositoryInterface;
use App\Exceptions\UtilisateurNonTrouveException;

final class ProduitService {
    public function __construct(
        private readonly ProduitRepositoryInterface $repository
    ) {}

    public function creerProduit(string $nom, float $prix, int $stock): Produit {
        $produit = Produit::creer($nom, $prix, $stock);
        return $this->repository->save($produit);
    }

    public function ajusterStock(int $produitId, int $quantite): Produit {
        $produit = $this->repository->findById($produitId)
            ?? throw new \RuntimeException("Produit #$produitId introuvable");

        $produit->ajouterStock($quantite);
        return $this->repository->save($produit);
    }

    /** @return Produit[] */
    public function getProduitsCatalogue(): array {
        return $this->repository->findActifs();
    }
}
```

### Exemple d'utilisation complete (index.php)

```php
<?php
declare(strict_types=1);

require_once __DIR__ . '/vendor/autoload.php';

use App\Database\DatabaseConnection;
use App\Repository\PdoProduitRepository;
use App\Services\ProduitService;

$pdo        = DatabaseConnection::getInstance();
$repository = new PdoProduitRepository($pdo);
$service    = new ProduitService($repository);

// Creer un produit
$produit = $service->creerProduit('Clavier mecanique', 89.99, 50);
echo "Cree : {$produit->getNom()} (ID: {$produit->getId()})\n";

// Ajuster le stock
$produit = $service->ajusterStock($produit->getId(), 20);
echo "Nouveau stock : {$produit->getStock()}\n";

// Lister le catalogue
$catalogue = $service->getProduitsCatalogue();
foreach ($catalogue as $p) {
    printf("- %s : %.2f EUR (%d en stock)\n", $p->getNom(), $p->getPrix(), $p->getStock());
}
```

> [!example] Ce qu'on a accompli ici
> - Le `Controller` ou `index.php` ne connait pas SQL — il parle uniquement au `Service`
> - Le `Service` ne connait pas PDO — il parle uniquement a l'interface du Repository
> - Le `Repository` est la seule couche qui connait PDO et SQL
> - Si on veut passer de MySQL a PostgreSQL : on cree `PgProduitRepository`, on change une ligne dans la composition (ou le container DI). Aucune autre modification.

---

## Carte Mentale

```
PHP POO et Base de Donnees
│
├── POO AVANCEE
│   ├── Interface ──── contrat pur, multi-implementable
│   ├── Trait ──────── code reutilisable horizontal
│   ├── Abstract ───── modele partiel, methodes abstraites
│   ├── final ──────── interdit heritage / surcharge
│   └── static ─────── appartient a la classe, pas a l'instance
│
├── NAMESPACES & PSR-4
│   ├── namespace App\Models;
│   ├── use App\Models\Produit;
│   └── App\ → src/ (dans composer.json)
│
├── COMPOSER
│   ├── composer.json ─ dependances + autoload
│   ├── composer install / update / require
│   └── vendor/autoload.php ─ point d'entree unique
│
├── PDO
│   ├── Connexion ─────── new PDO($dsn, $user, $pass, $options)
│   ├── Prepared stmt ─── prepare() → bindValue() → execute()
│   ├── Fetch ─────────── fetch() / fetchAll() / fetchObject()
│   ├── INSERT/UPDATE ─── execute() + lastInsertId() / rowCount()
│   └── Transactions ──── beginTransaction() → commit() / rollBack()
│
├── PHP 8.x FEATURES
│   ├── Constructor Promotion ── public string $x dans __construct
│   ├── Named Arguments ──────── fn(param: $val)
│   ├── Match Expression ─────── match($x) { val => result }
│   ├── Nullsafe ?-> ─────────── $a?->b()?->c()
│   ├── Enums ────────────────── enum Status: string { case A = 'a' }
│   └── Readonly ─────────────── propriete non modifiable apres init
│
├── GESTION DES ERREURS
│   ├── Exceptions perso ─── extends RuntimeException
│   ├── try/catch/finally ── capturer, traiter, liberer
│   └── set_exception_handler ─ handler global de dernier recours
│
├── DESIGN PATTERNS
│   ├── Repository ── isole PDO du domaine metier
│   ├── Factory ───── centralise la creation d'objets
│   └── Singleton ─── eviter ! preferer DI Container
│
└── STANDARDS PSR
    ├── PSR-1 ─── coding standard basique
    ├── PSR-4 ─── autoloading (namespace = chemin)
    └── PSR-12 ── style guide etendu
```

---

## Exercices Pratiques

### Exercice 1 — Modeliser une bibliotheque

Cree une mini-application qui gere une bibliotheque.

**Modeles requis** :
- `Livre` : id, titre, auteur, isbn, annee, disponible (bool)
- `Adherent` : id, nom, email, dateInscription

**Interfaces requises** :
- `LivreRepositoryInterface` : findById, findByIsbn, findDisponibles, save, delete
- `AdherentRepositoryInterface` : findById, findByEmail, save

**Trait requis** :
- `Validable` : methode `valider(): array` (retourne tableau d'erreurs)

**Contraintes** :
1. Utiliser `declare(strict_types=1)` partout
2. Utiliser le constructor promotion de PHP 8
3. Les methodes `find*` retournent `null` si absent (pas d'exception)
4. Ecrire une methode `emprunter(Livre $livre, Adherent $adherent): void` dans un service `BibliothequeService`

**Bonus** : implementer une enum `GenreLitteraire: string` avec 5 valeurs et l'ajouter a `Livre`.

---

### Exercice 2 — Repository PDO complet

En partant de l'exercice 1, implementer `PdoLivreRepository` avec :

1. Cree la table SQL :
```sql
CREATE TABLE livres (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    titre       VARCHAR(255) NOT NULL,
    auteur      VARCHAR(255) NOT NULL,
    isbn        VARCHAR(13) UNIQUE NOT NULL,
    annee       YEAR NOT NULL,
    disponible  TINYINT(1) NOT NULL DEFAULT 1
);
```

2. Implementer toutes les methodes de `LivreRepositoryInterface`
3. Ajouter une methode `findByAuteur(string $auteur): array` avec une requete `LIKE`
4. Ecrire une methode `emprunterEnTransaction(int $livreId, int $adherentId): void` qui :
   - Verifie que le livre est disponible (SELECT + lock FOR UPDATE)
   - Marque le livre comme non disponible (UPDATE)
   - Insere un enregistrement dans la table `emprunts` (INSERT)
   - Tout ca dans une transaction — rollback si l'une echoue

---

### Exercice 3 — Exceptions metier et validation

Cree un systeme de validation robuste pour un formulaire d'inscription.

**Regles metier** :
- Email doit etre valide (`filter_var`)
- Password : 8 caracteres minimum, au moins 1 majuscule, 1 chiffre
- Age : entre 13 et 120 ans
- Pseudo : 3-30 caracteres, uniquement `[a-zA-Z0-9_-]`

**Ce que tu dois creer** :
1. Une classe `ValidationException` qui stocke `array<string, string>` (champ → message)
2. Une classe `InscriptionRequest` avec des methodes statiques de validation
3. Une methode `static fromArray(array $data): self` qui valide et construit l'objet, ou leve `ValidationException`

**Exemple d'utilisation attendue** :
```php
try {
    $request = InscriptionRequest::fromArray($_POST);
    $service->inscrire($request);
} catch (ValidationException $e) {
    // $e->getErreurs() retourne ['email' => 'Email invalide', 'password' => '...']
    foreach ($e->getErreurs() as $champ => $message) {
        echo "$champ : $message\n";
    }
}
```

---

### Exercice 4 — Architecture complete avec Composer

Cree un projet PHP complet en partant de zero :

**Structure a creer** :
```
mon-blog/
├── composer.json
├── src/
│   ├── Models/Article.php
│   ├── Repository/ArticleRepositoryInterface.php
│   ├── Repository/PdoArticleRepository.php
│   ├── Services/ArticleService.php
│   └── Exceptions/ArticleNonTrouveException.php
└── public/index.php
```

**Etapes** :
1. Initialiser Composer avec `composer init` et configurer le PSR-4 pour `App\`
2. Ajouter la dependance `vlucas/phpdotenv` et lire la config BDD depuis un fichier `.env`
3. Creer le modele `Article` avec : id, titre, contenu, statut (enum : `Brouillon | Publie | Archive`), createdAt
4. Implementer le repository avec pagination : `findPublies(int $page, int $parPage): array`
5. Dans `public/index.php` : afficher les 10 premiers articles publies en JSON

**Contraintes d'architecture** :
- `public/index.php` ne cree aucune instance de PDO directement
- Toute la composition (DI manuel) se fait dans un fichier `bootstrap.php`
- Aucun `echo` de SQL dans le code applicatif

> [!example] Correction partielle — bootstrap.php
> ```php
> <?php
> // bootstrap.php
> require_once __DIR__ . '/vendor/autoload.php';
>
> $dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
> $dotenv->load();
>
> $pdo = new PDO(
>     "mysql:host={$_ENV['DB_HOST']};dbname={$_ENV['DB_NAME']};charset=utf8mb4",
>     $_ENV['DB_USER'],
>     $_ENV['DB_PASS'],
>     [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
> );
>
> $articleRepository = new App\Repository\PdoArticleRepository($pdo);
> $articleService    = new App\Services\ArticleService($articleRepository);
>
> return $articleService; // ou utiliser un container DI
> ```

---

## Recap final : les regles d'or

| Regle | Pourquoi |
|---|---|
| `declare(strict_types=1)` partout | Evite les conversions silencieuses de types |
| Toujours des prepared statements | Previent les injections SQL |
| Repository devant PDO | Isoler la BDD du metier — testable, remplacable |
| Interface pour chaque repository | Permet les mocks dans les tests unitaires |
| `beginTransaction` → try → `commit` → catch → `rollBack` | Garantit la coherence des donnees |
| Exceptions metier derivees de `\RuntimeException` | Typage precis des erreurs, catch granulaire |
| Enums plutot que constantes magiques | Typage et autocompletion, impossible de passer une valeur invalide |
| PSR-4 + Composer = zero `require` manuel | L'autoloader charge les classes a la demande |
| Eviter Singleton, preferer injection | Testabilite, pas d'etat global cache |

Liens : [[PHP/01 - PHP et Symfony]] · [[SQL/01 - Introduction au SQL]] · [[PHP/03 - Laravel Framework]]
