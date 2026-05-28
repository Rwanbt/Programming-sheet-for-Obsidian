# UML Fondamentaux et Diagrammes

> [!info] Pourquoi modéliser avant de coder ?
> Un plan d'architecte avant de construire un bâtiment. UML est le langage universel pour décrire la structure et le comportement d'un système logiciel — avant que le code n'existe, pendant sa conception, et comme documentation vivante.

## Table des matières
1. [[#Introduction à UML]]
2. [[#Diagramme de Classes]]
3. [[#Diagramme de Séquence]]
4. [[#Diagramme de Cas d'Utilisation]]
5. [[#Diagramme d'Activité]]
6. [[#Diagramme d'État]]
7. [[#Diagramme de Composants]]
8. [[#Diagramme de Déploiement]]
9. [[#PlantUML — Syntaxe Complète]]
10. [[#Mermaid dans Obsidian]]
11. [[#Quand utiliser quel diagramme]]
12. [[#Exercices Pratiques]]

---

## Introduction à UML

### Qu'est-ce qu'UML ?

**UML** (Unified Modeling Language) est un langage de modélisation standardisé par l'OMG (Object Management Group) en 1997. Il fournit une notation graphique commune pour décrire des systèmes logiciels orientés objet.

> [!tip] L'analogie architecturale
> Un architecte produit des plans (façade, coupe, électricité, plomberie) avant de construire. Chaque plan cible un public différent (client, maçon, électricien). UML fonctionne de la même manière : différents diagrammes pour différentes perspectives du même système.

### UML 1.x vs UML 2.x

| Aspect | UML 1.x (1997–2001) | UML 2.x (2005–aujourd'hui) |
|--------|---------------------|---------------------------|
| Diagrammes | 9 types | 14 types |
| Séquence | Limité | Fragments (alt/loop/par) |
| Composants | Basique | Interfaces fournies/requises |
| Activité | Simple flowchart | BPMN-like avec swim lanes |
| État | Basique | États composites, historique |
| Adoption | Largement remplacé | Standard actuel |

### Les 14 types de diagrammes UML 2.x

```
Diagrammes UML 2.x
├── Diagrammes Structurels (7)
│   ├── Classes ← le plus important
│   ├── Objets
│   ├── Composants
│   ├── Déploiement
│   ├── Packages
│   ├── Structure Composite
│   └── Profils
└── Diagrammes Comportementaux (7)
    ├── Cas d'Utilisation
    ├── Activité
    ├── État (Machine à états)
    ├── Séquence
    ├── Communication
    ├── Temporisation
    └── Vue d'Ensemble des Interactions
```

### Outils

| Outil | Type | Gratuit | PlantUML | Mermaid | Collaboratif |
|-------|------|---------|----------|---------|--------------|
| **PlantUML** | Code → Diagramme | Oui | Natif | Non | Via export |
| **Mermaid** | Code → Diagramme | Oui | Non | Natif | Via intégrations |
| **draw.io / diagrams.net** | GUI | Oui | Import | Import | Oui |
| **Lucidchart** | GUI | Freemium | Import | Non | Oui |
| **StarUML** | GUI | Freemium | Non | Non | Non |
| **Visual Paradigm** | GUI | Freemium | Non | Non | Oui |
| **Obsidian** | Notes | Oui | Plugin | Natif | Non |

> [!tip] Recommandation
> Pour des cours et documentation : **Mermaid** (natif Obsidian) + **PlantUML** (plugin Obsidian). Pour du travail en équipe : **draw.io** (gratuit, exportable, intégré à Confluence/GitHub).

---

## Diagramme de Classes

Le diagramme de classes est le **cœur d'UML** — il modélise la structure statique du système : les entités, leurs attributs, leurs méthodes et leurs relations.

### Anatomie d'une classe

```
┌─────────────────────────┐
│        NomClasse        │  ← Compartiment Nom (gras, centré)
├─────────────────────────┤
│  - attributPrivé: Type  │  ← Compartiment Attributs
│  # attributProtégé: int │
│  + attributPublic: str  │
│  ~ attributPackage: bool│
├─────────────────────────┤
│  + méthodePublique(): void │  ← Compartiment Méthodes
│  - méthodePrivée(): int    │
│  # méthodeProtégée(): str  │
└─────────────────────────┘
```

### Visibilité des membres

| Symbole | Visibilité | Équivalent Python | Équivalent Java |
|---------|-----------|-------------------|-----------------|
| `+` | Public | (rien) | `public` |
| `-` | Private | `__` (name mangling) | `private` |
| `#` | Protected | `_` (convention) | `protected` |
| `~` | Package/Internal | N/A | (défaut, package-private) |

### Exemple complet : Système de bibliothèque

#### Code Python correspondant

```python
from abc import ABC, abstractmethod
from datetime import datetime
from typing import Optional

class Personne(ABC):
    """Classe abstraite — ne peut pas être instanciée directement."""
    
    def __init__(self, nom: str, email: str):
        self._nom = nom          # protected
        self.__email = email     # private
    
    @property
    def nom(self) -> str:
        return self._nom
    
    @abstractmethod
    def identifier(self) -> str:
        pass

class Membre(Personne):
    """Héritage de Personne."""
    
    def __init__(self, nom: str, email: str, numero_carte: str):
        super().__init__(nom, email)
        self.__numero_carte = numero_carte
        self.__emprunts: list['Emprunt'] = []  # composition
    
    def identifier(self) -> str:
        return f"Membre #{self.__numero_carte}"
    
    def emprunter(self, livre: 'Livre') -> 'Emprunt':
        emprunt = Emprunt(self, livre)
        self.__emprunts.append(emprunt)
        return emprunt

class Livre:
    """Agrégation avec Auteur."""
    
    def __init__(self, titre: str, isbn: str, auteur: 'Auteur'):
        self.titre = titre
        self.__isbn = isbn
        self.auteur = auteur      # association (agrégation)
        self.__disponible = True
    
    @property
    def disponible(self) -> bool:
        return self.__disponible

class Auteur(Personne):
    def __init__(self, nom: str, email: str):
        super().__init__(nom, email)
        self.__livres: list[Livre] = []
    
    def identifier(self) -> str:
        return f"Auteur: {self._nom}"

class Emprunt:
    """Composition — l'emprunt appartient entièrement au membre."""
    
    def __init__(self, membre: Membre, livre: Livre):
        self.__membre = membre
        self.__livre = livre
        self.__date_debut = datetime.now()
        self.__date_retour: Optional[datetime] = None
    
    def retourner(self):
        self.__date_retour = datetime.now()
```

#### Code Java correspondant

```java
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

// Classe abstraite
public abstract class Personne {
    protected String nom;      // protected
    private String email;      // private
    
    public Personne(String nom, String email) {
        this.nom = nom;
        this.email = email;
    }
    
    public String getNom() { return nom; }
    public abstract String identifier();
}

// Héritage (généralisation)
public class Membre extends Personne {
    private String numeroCarte;
    private List<Emprunt> emprunts = new ArrayList<>();  // composition
    
    public Membre(String nom, String email, String numeroCarte) {
        super(nom, email);
        this.numeroCarte = numeroCarte;
    }
    
    @Override
    public String identifier() {
        return "Membre #" + numeroCarte;
    }
    
    public Emprunt emprunter(Livre livre) {
        Emprunt emprunt = new Emprunt(this, livre);
        emprunts.add(emprunt);
        return emprunt;
    }
}

// Interface (réalisation)
public interface Cataloguable {
    String getReference();
    boolean estDisponible();
}

public class Livre implements Cataloguable {
    private String titre;
    private String isbn;
    private Auteur auteur;    // association (agrégation)
    private boolean disponible = true;
    
    @Override
    public String getReference() { return isbn; }
    
    @Override
    public boolean estDisponible() { return disponible; }
}
```

### Relations entre classes

#### 1. Association (lien simple)
Une classe **connaît** une autre. Relation faible, sans ownership.

```
Etudiant ────────── Cours
          0..* inscrits à 0..*
```

#### 2. Agrégation (« a un », tout-partie faible)
La partie **peut exister** sans le tout. Représentée par un losange vide.

```
Departement ◇────── Employe
             1    0..*
```
Si le département est dissous, les employés continuent d'exister.

#### 3. Composition (« contient », tout-partie forte)
La partie **ne peut pas exister** sans le tout. Représentée par un losange plein.

```
Commande ◆────── LigneCommande
          1    1..*
```
Si la commande est supprimée, les lignes sont supprimées.

#### 4. Héritage / Généralisation
Relation **est-un**. Flèche à tête fermée vide vers la superclasse.

```
Animal ◁────── Chien
       ◁────── Chat
```

#### 5. Réalisation / Interface
Une classe **implémente** une interface. Flèche en pointillés à tête fermée vide.

```
<<interface>>
Payable ◁- - - CarteBancaire
        ◁- - - Virement
```

#### 6. Dépendance
Une classe **utilise** une autre temporairement. Flèche en pointillés simple.

```
CommandeService - - - > EmailService
```

### Tableau des multiplicités

| Notation | Signification |
|----------|---------------|
| `1` | Exactement un |
| `0..1` | Zéro ou un (optionnel) |
| `*` ou `0..*` | Zéro ou plusieurs |
| `1..*` | Un ou plusieurs |
| `3` | Exactement trois |
| `2..5` | Entre deux et cinq |

---

## Diagramme de Séquence

Modélise les **interactions dans le temps** entre objets ou composants pour un scénario donné.

### Éléments clés

| Élément | Représentation | Description |
|---------|----------------|-------------|
| Acteur | Bonhomme | Utilisateur externe |
| Objet | Rectangle + ligne de vie | Participant |
| Ligne de vie | Ligne verticale pointillée | Existence dans le temps |
| Boîte d'activation | Rectangle sur ligne de vie | Période d'activité |
| Message synchrone | Flèche pleine → | Attend la réponse |
| Message asynchrone | Flèche ouverte → | N'attend pas |
| Retour | Flèche pointillée ← | Valeur de retour |
| Création | → «create» | Instanciation |
| Destruction | → «destroy» + X | Fin de vie |

### Fragments combinés

| Fragment | Signification |
|----------|---------------|
| `alt` | Alternative (if/else) |
| `opt` | Optionnel (if sans else) |
| `loop` | Boucle |
| `par` | Parallèle |
| `ref` | Référence vers un autre diagramme |
| `break` | Interruption |
| `critical` | Section critique |

### Exemple : Flux de login

```plantuml
@startuml Login Flow
actor Utilisateur
participant "Page Login" as UI
participant "AuthService" as Auth
participant "UserRepository" as DB
participant "SessionManager" as Session

Utilisateur -> UI : soumettre formulaire(email, password)
activate UI

UI -> Auth : authenticate(email, password)
activate Auth

Auth -> DB : findByEmail(email)
activate DB
DB --> Auth : User | null
deactivate DB

alt utilisateur trouvé
    Auth -> Auth : verifyPassword(hash, password)
    
    alt mot de passe valide
        Auth -> Session : createSession(userId)
        activate Session
        Session --> Auth : sessionToken
        deactivate Session
        Auth --> UI : AuthResult(success, token)
    else mot de passe invalide
        Auth --> UI : AuthResult(failure, "Invalid credentials")
    end
else utilisateur non trouvé
    Auth --> UI : AuthResult(failure, "User not found")
end

deactivate Auth

alt authentification réussie
    UI --> Utilisateur : redirect("/dashboard")
else échec
    UI --> Utilisateur : afficher erreur
end

deactivate UI
@enduml
```

### Exemple : Appel API REST

```plantuml
@startuml REST API Call
participant "Client JS" as Client
participant "API Gateway" as Gateway
participant "AuthMiddleware" as Auth
participant "ProductController" as Controller
participant "ProductService" as Service
participant "Database" as DB

Client -> Gateway : GET /api/products\nAuthorization: Bearer <token>
activate Gateway

Gateway -> Auth : validateToken(token)
activate Auth
Auth --> Gateway : { userId: 42, role: "user" }
deactivate Auth

Gateway -> Controller : getProducts(userId, filters)
activate Controller

Controller -> Service : findProducts(filters)
activate Service

Service -> DB : SELECT * FROM products WHERE ...
activate DB
DB --> Service : [Product]
deactivate DB

Service --> Controller : ProductList
deactivate Service

Controller --> Gateway : 200 OK { data: [...], total: 50 }
deactivate Controller

Gateway --> Client : HTTP 200\n{ data: [...], total: 50 }
deactivate Gateway
@enduml
```

---

## Diagramme de Cas d'Utilisation

Modélise les **fonctionnalités du système** du point de vue des utilisateurs. Répond à la question : **"Que fait le système ?"**

### Éléments

| Élément | Représentation | Description |
|---------|----------------|-------------|
| Acteur | Bonhomme | Entité externe (humain, système) |
| Cas d'utilisation | Ellipse | Fonctionnalité du système |
| Frontière système | Rectangle | Limite du système modélisé |
| Association | Ligne | Acteur participe au cas |
| `<<include>>` | Flèche pointillée | Toujours inclus |
| `<<extend>>` | Flèche pointillée | Conditionnellement ajouté |
| Généralisation | Flèche | Spécialisation d'acteur ou cas |

### Différence include vs extend

> [!warning] Include vs Extend — source de confusion fréquente
> - **`<<include>>`** : le cas inclus est **toujours** exécuté. C'est une factorisation (DRY). Ex : "S'authentifier" est inclus dans tout cas nécessitant un login.
> - **`<<extend>>`** : le cas étendant est **optionnel et conditionnel**. Ex : "Ajouter une note" peut étendre "Passer une commande" seulement si le client le souhaite.

### Exemple : Système de bibliothèque

```plantuml
@startuml Bibliothèque Use Cases
left to right direction

actor "Membre" as Membre
actor "Bibliothécaire" as Biblio
actor "Administrateur" as Admin
actor "Système Email" as Email <<system>>

Biblio --|> Membre

rectangle "Système de Bibliothèque" {
    usecase "Rechercher un livre" as UC1
    usecase "Emprunter un livre" as UC2
    usecase "Retourner un livre" as UC3
    usecase "Renouveler un emprunt" as UC4
    usecase "S'authentifier" as UC_Auth
    usecase "Vérifier disponibilité" as UC5
    usecase "Envoyer notification" as UC6
    usecase "Gérer le catalogue" as UC7
    usecase "Gérer les membres" as UC8
    usecase "Générer des rapports" as UC9
}

Membre --> UC1
Membre --> UC2
Membre --> UC3
Membre --> UC4

Biblio --> UC7
Biblio --> UC8

Admin --> UC9
Admin --> UC7
Admin --> UC8

UC2 ..> UC_Auth : <<include>>
UC3 ..> UC_Auth : <<include>>
UC4 ..> UC_Auth : <<include>>

UC2 ..> UC5 : <<include>>
UC4 ..> UC6 : <<extend>>
UC3 ..> UC6 : <<extend>>

Email <-- UC6
@enduml
```

---

## Diagramme d'Activité

Modélise les **flux de travail et algorithmes**. Proche des flowcharts mais avec des concepts OO (swim lanes, fork/join pour la concurrence).

### Éléments clés

| Élément | Représentation | Description |
|---------|----------------|-------------|
| Nœud initial | Cercle noir plein | Point de départ |
| Nœud final | Cercle noir dans cercle | Point d'arrivée |
| Action | Rectangle arrondi | Tâche élémentaire |
| Décision | Losange | Branchement conditionnel |
| Fork | Barre horizontale → multiple | Parallélisation |
| Join | Multiple → barre horizontale | Synchronisation |
| Swim Lane | Colonne/rangée | Responsabilité d'un acteur |

### Exemple : Processus commande e-commerce

```plantuml
@startuml Commande E-Commerce
|Client|
start
:Sélectionner produits;
:Valider panier;
:Saisir adresse livraison;

|Système Paiement|
:Choisir moyen de paiement;

if (Paiement valide ?) then (oui)
    :Confirmer paiement;
    |Système Commande|
    :Créer commande;
    :Réserver stock;
    
    fork
        |Système Email|
        :Envoyer confirmation email;
    fork again
        |Entrepôt|
        :Préparer colis;
        :Expédier commande;
        :Générer numéro suivi;
    end fork
    
    |Client|
    :Recevoir notification expédition;
    :Réceptionner colis;
    stop
    
else (non)
    |Client|
    :Afficher erreur paiement;
    :Proposer autre moyen;
    stop
end if
@enduml
```

---

## Diagramme d'État

Modélise le **cycle de vie d'un objet** — comment il change d'état en réponse à des événements.

### Éléments

| Élément | Description |
|---------|-------------|
| État initial | Cercle plein |
| État | Rectangle arrondi avec nom |
| Transition | Flèche avec `événement [garde] / action` |
| État final | Cercle plein dans cercle |
| État composite | État contenant d'autres états |
| Historique | `H` dans cercle (mémorise le dernier état) |

### Syntaxe des transitions

```
événement [condition_garde] / action_exécutée
```

### Exemple : Cycle de vie d'une commande

```plantuml
@startuml Commande State Machine
[*] --> EnAttente : créer()

EnAttente --> EnCours : payer() [paiement valide]
EnAttente --> Annulée : annuler()
EnAttente --> Expirée : timeout [> 30 min sans paiement]

EnCours --> Préparée : préparer()
EnCours --> Annulée : annuler() [stock insuffisant]

Préparée --> Expédiée : expédier() / envoyer_notification()

Expédiée --> Livrée : confirmer_livraison()
Expédiée --> EnLitige : signaler_problème()

EnLitige --> Remboursée : valider_litige()
EnLitige --> Livrée : résoudre_litige()

Livrée --> [*]
Annulée --> [*]
Expirée --> [*]
Remboursée --> [*]

state EnCours {
    [*] --> PaiementReçu
    PaiementReçu --> StockRéservé : réserver_stock()
    StockRéservé --> [*]
}
@enduml
```

---

## Diagramme de Composants

Modélise la **structure physique** du système — les modules, bibliothèques, services et leurs interfaces.

### Éléments

| Élément | Description |
|---------|-------------|
| Composant | Rectangle avec icône composant ou `<<component>>` |
| Interface fournie | Cercle (lollipop) — ce que le composant offre |
| Interface requise | Demi-cercle (socket) — ce que le composant consomme |
| Port | Carré sur le bord d'un composant |
| Connecteur | Ligne reliant lollipop et socket |
| Artefact | Fichier physique (`.jar`, `.exe`, `.dll`) |

### Exemple : Architecture Microservices

```plantuml
@startuml Microservices Architecture
package "Frontend" {
    [React App] <<component>>
    [Mobile App] <<component>>
}

package "API Gateway" {
    [Kong Gateway] <<component>>
    [Auth Middleware] <<component>>
}

package "Services" {
    [User Service] <<component>>
    [Product Service] <<component>>
    [Order Service] <<component>>
    [Notification Service] <<component>>
}

package "Infrastructure" {
    database "PostgreSQL\n(Users)" as UserDB
    database "MongoDB\n(Products)" as ProductDB
    database "PostgreSQL\n(Orders)" as OrderDB
    queue "RabbitMQ" as MQ
}

[React App] --> [Kong Gateway] : HTTPS
[Mobile App] --> [Kong Gateway] : HTTPS

[Kong Gateway] --> [Auth Middleware]
[Kong Gateway] --> [User Service]
[Kong Gateway] --> [Product Service]
[Kong Gateway] --> [Order Service]

[User Service] --> UserDB
[Product Service] --> ProductDB
[Order Service] --> OrderDB

[Order Service] --> MQ : publish
[Notification Service] <-- MQ : subscribe
@enduml
```

---

## Diagramme de Déploiement

Modélise l'**infrastructure physique** — sur quelles machines tournent quels logiciels.

### Éléments

| Élément | Description |
|---------|-------------|
| Nœud | Ressource physique ou virtuelle (serveur, VM, conteneur) |
| Artefact | Fichier déployable (`.war`, image Docker, `.exe`) |
| Communication | Lien entre nœuds avec protocole |

### Exemple : Cloud Deployment

```plantuml
@startuml Cloud Deployment
node "Internet" as Internet

node "CDN\n(CloudFront)" as CDN {
    artifact "Static Assets\n(React Build)"
}

node "Load Balancer\n(AWS ALB)" as LB

node "App Cluster\n(EKS)" as Cluster {
    node "Pod 1" {
        artifact "API Server\n(Django)"
    }
    node "Pod 2" {
        artifact "API Server\n(Django)"
    }
    node "Pod 3" {
        artifact "Worker\n(Celery)"
    }
}

node "Database Cluster\n(RDS Multi-AZ)" as DB {
    database "PostgreSQL\nPrimary"
    database "PostgreSQL\nReplica"
}

node "Cache\n(ElastiCache)" as Cache {
    database "Redis"
}

node "Message Queue\n(SQS)" as Queue

Internet --> CDN : HTTPS
Internet --> LB : HTTPS
LB --> Cluster : HTTP
Cluster --> DB : TCP 5432
Cluster --> Cache : TCP 6379
Cluster --> Queue : HTTPS (AWS SDK)
@enduml
```

---

## PlantUML — Syntaxe Complète

> [!info] PlantUML dans Obsidian
> Installer le plugin communautaire **PlantUML** dans Obsidian. Utiliser le bloc ` ```plantuml ` pour les diagrammes.

### Diagramme de classes PlantUML

```plantuml
@startuml Système Blog
skinparam classAttributeIconSize 0

abstract class Utilisateur {
    # id: Long
    # email: String
    # motDePasse: String
    + seConnecter(): boolean
    + {abstract} getRole(): String
}

class Admin {
    + supprimerArticle(id: Long): void
    + getRole(): String
}

class Auteur {
    - pseudonyme: String
    + creerArticle(titre: String): Article
    + getRole(): String
}

interface Commentable {
    + ajouterCommentaire(texte: String): Commentaire
    + supprimerCommentaire(id: Long): void
}

class Article {
    - id: Long
    - titre: String
    - contenu: String
    - datePublication: Date
    - publie: boolean
    + publier(): void
    + addTag(tag: Tag): void
}

class Commentaire {
    - id: Long
    - texte: String
    - dateCreation: Date
}

class Tag {
    - nom: String
}

' Héritage
Utilisateur <|-- Admin
Utilisateur <|-- Auteur

' Réalisation interface
Article ..|> Commentable

' Composition (le commentaire appartient à l'article)
Article "1" *-- "0..*" Commentaire

' Association
Auteur "1" -- "0..*" Article : rédige >

' Agrégation (le tag peut exister sans article)
Article "0..*" o-- "0..*" Tag

@enduml
```

### Diagramme de séquence PlantUML

```plantuml
@startuml Checkout Process
autonumber

actor Client
participant "CartService" as Cart
participant "OrderService" as Order
participant "PaymentGateway" as Payment
participant "InventoryService" as Inventory
database "Database" as DB

Client -> Cart : checkout()
Cart -> Order : createOrder(cartItems)
activate Order

Order -> DB : beginTransaction()

loop pour chaque article
    Order -> Inventory : reserveStock(productId, qty)
    activate Inventory
    Inventory --> Order : StockResult
    deactivate Inventory
end

alt tous les stocks disponibles
    Order -> DB : saveOrder(order)
    Order -> Payment : processPayment(amount, cardToken)
    activate Payment
    
    alt paiement accepté
        Payment --> Order : PaymentConfirmation
        Order -> DB : updateOrderStatus("PAID")
        Order -> DB : commit()
        Order --> Cart : OrderConfirmation
        Cart --> Client : 200 OK { orderId, total }
    else paiement refusé
        Payment --> Order : PaymentError
        Order -> DB : rollback()
        Order --> Cart : PaymentFailure
        Cart --> Client : 402 Payment Required
    end
    
    deactivate Payment
else stock insuffisant
    Order -> DB : rollback()
    Order --> Cart : StockError
    Cart --> Client : 409 Conflict { unavailableItems }
end

deactivate Order
@enduml
```

---

## Mermaid dans Obsidian

Obsidian supporte **nativement** Mermaid sans plugin supplémentaire. Utiliser le bloc ` ```mermaid `.

### Diagramme de classes Mermaid

```mermaid
classDiagram
    class Animal {
        +String nom
        +int age
        +faire_son() String
        +se_deplacer() void
    }
    
    class Chien {
        +String race
        +aboyer() void
        +faire_son() String
    }
    
    class Chat {
        +bool estDomestique
        +ronronner() void
        +faire_son() String
    }
    
    class Veterinaire {
        +String prenom
        +String specialite
        +examiner(animal: Animal) Bilan
    }
    
    Animal <|-- Chien
    Animal <|-- Chat
    Veterinaire "1" --> "0..*" Animal : soigne
```

### Diagramme de séquence Mermaid

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant A as AuthAPI
    participant D as Database
    participant R as Redis

    U->>A: POST /login {email, password}
    A->>D: SELECT user WHERE email=?
    D-->>A: User record
    
    alt Mot de passe valide
        A->>R: SET session:token userId TTL 3600
        R-->>A: OK
        A-->>U: 200 {token, user}
    else Mot de passe invalide
        A-->>U: 401 Unauthorized
    end
```

### Diagramme d'état Mermaid

```mermaid
stateDiagram-v2
    [*] --> Brouillon
    Brouillon --> EnRevue : soumettre()
    EnRevue --> Approuvé : approuver()
    EnRevue --> Brouillon : rejeter()
    Approuvé --> Publié : publier()
    Publié --> Archivé : archiver()
    Archivé --> [*]
```

### Diagramme d'activité (flowchart) Mermaid

```mermaid
flowchart TD
    A([Début]) --> B[Réceptionner commande]
    B --> C{Stock disponible ?}
    C -->|Oui| D[Préparer colis]
    C -->|Non| E[Notifier rupture]
    E --> F[Commander fournisseur]
    F --> D
    D --> G[Expédier]
    G --> H{Livraison OK ?}
    H -->|Oui| I([Fin — Succès])
    H -->|Non| J[Ouvrir litige]
    J --> K[Rembourser client]
    K --> I
```

---

## Quand utiliser quel diagramme

| Question | Diagramme |
|----------|-----------|
| Quelles sont les entités et leurs relations ? | Classes |
| Comment les objets interagissent-ils dans un scénario ? | Séquence |
| Que fait le système pour ses utilisateurs ? | Cas d'utilisation |
| Comment se déroule un processus métier ? | Activité |
| Comment évolue l'état d'un objet ? | État |
| Comment les modules sont-ils assemblés ? | Composants |
| Sur quelle infrastructure tourne le système ? | Déploiement |

### Workflow de modélisation recommandé

```
1. Cas d'Utilisation     → "Que doit faire le système ?"
        ↓
2. Classes               → "Avec quelles entités ?"
        ↓
3. Séquence              → "Comment pour chaque scénario ?"
        ↓
4. État                  → "Cycle de vie des entités complexes ?"
        ↓
5. Activité              → "Détail des processus métier ?"
        ↓
6. Composants            → "Comment organiser le code ?"
        ↓
7. Déploiement           → "Sur quelle infrastructure ?"
```

> [!tip] Règle pratique
> Ne modéliser que ce qui apporte de la valeur. Un diagramme de classes à 50 entités est moins utile qu'un diagramme ciblé sur 5-10 entités clés. UML est un outil de communication — s'il ne facilite pas la conversation, il est trop détaillé.

---

## Exercices Pratiques

### Exercice 1 — Diagramme de classes : Application bancaire

Créez un diagramme de classes pour un système bancaire simple :
- Un `Client` peut avoir plusieurs `Comptes` (Courant, Épargne)
- Un `Compte` a un `solde`, une `dateOuverture`, et un `numero`
- `CompteEpargne` hérite de `Compte` et ajoute un `tauxInteret`
- Un `Compte` peut effectuer des `Virements` vers un autre `Compte`
- Un `Virement` a un `montant`, une `date`, une `description`
- `Banque` gère l'ensemble des comptes et clients

**À faire :** Dessiner le diagramme avec les bonnes multiplicités et types de relation (composition pour virement, agrégation pour compte dans banque, héritage pour épargne).

### Exercice 2 — Diagramme de séquence : Reset de mot de passe

Modélisez le flux de réinitialisation de mot de passe :
1. Utilisateur demande un reset avec son email
2. Système vérifie si l'email existe
3. Si oui : génère un token unique, l'enregistre en base avec expiration, envoie un email
4. Utilisateur clique le lien → système vérifie le token (valide ? expiré ?)
5. Si valide : affiche formulaire nouveau mot de passe
6. Utilisateur soumet → hachage + sauvegarde + invalidation du token

**Utiliser les fragments alt/opt.**

### Exercice 3 — Diagramme d'état : Ticket de support

Modélisez le cycle de vie d'un ticket de support :
- États : Ouvert, En cours, En attente client, Résolu, Fermé, Rouvert
- Transitions avec événements et gardes (ex : "Fermer [résolu depuis > 7 jours]")
- Ajouter un état composite pour "En cours" avec sous-états : "Assigné" et "EnAnalyse"

### Exercice 4 — Mermaid dans Obsidian

Créez ces diagrammes en Mermaid directement dans Obsidian :
1. Diagramme de classes : système de réservation d'hôtel (Hotel, Chambre, Reservation, Client)
2. Flowchart : algorithme de tri par sélection
3. Diagramme de séquence : ajout d'un article au panier e-commerce

### Exercice 5 — Architecture complète

Pour une application de gestion de tâches (type Trello) :
1. Cas d'utilisation : identifier les acteurs (Admin, User, Viewer) et les cas (créer carte, déplacer carte, inviter membre, etc.)
2. Classes : Board, Column, Card, User, Label, Attachment
3. Séquence : déplacer une carte entre colonnes (optimistic update + sync serveur)
4. Déploiement : déploiement cloud (CDN + Load Balancer + Backend + DB + Cache)

> [!tip] Ressources
> - PlantUML online : https://www.plantuml.com/plantuml
> - Mermaid Live Editor : https://mermaid.live
> - draw.io : https://app.diagrams.net

---

## Liens et Références

- [[03 - POO en Python]] — Classes et héritage Python en pratique
- [[01 - Introduction a Java]] — Classes et interfaces Java
- [[01 - Methodes Agiles Scrum et Kanban]] — Contexte d'utilisation d'UML en équipe Agile
- [[Architecture Logicielle/01 - Principes SOLID]] — Design patterns liés aux diagrammes de classes
