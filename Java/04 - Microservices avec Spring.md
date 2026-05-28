# Microservices avec Spring

## Monolithe vs Microservices

Avant d'adopter les microservices, il faut comprendre pourquoi l'architecture monolithique montre ses limites a grande echelle.

### Architecture monolithique

Un **monolithe** est une application ou tous les modules (gestion des utilisateurs, commandes, paiements, notifications...) sont deployes ensemble dans un seul artefact (JAR, WAR) :

```
┌──────────────────────────────────────────────────────┐
│                  MONOLITHE                            │
│                                                      │
│  ┌────────────┐  ┌────────────┐  ┌────────────────┐  │
│  │ Module     │  │ Module     │  │ Module         │  │
│  │ Users      │  │ Orders     │  │ Notifications  │  │
│  └─────┬──────┘  └──────┬─────┘  └───────┬────────┘  │
│        │                │                │            │
│        └────────────────┴────────────────┘            │
│                         │                            │
│              Base de donnees unique                  │
└──────────────────────────────────────────────────────┘
```

### Architecture microservices

Les **microservices** decoupent l'application en services independants, chacun avec sa propre base de donnees :

```
                         ┌─────────────────┐
                         │   API Gateway   │
                         │  (port entree)  │
                         └────────┬────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
    ┌─────────┴──────┐  ┌────────┴───────┐  ┌────────┴────────┐
    │ Users Service  │  │ Orders Service │  │  Notif Service  │
    │ :8081          │  │ :8082          │  │  :8083          │
    └────────┬───────┘  └───────┬────────┘  └────────┬────────┘
             │                  │                     │
           [BDD]              [BDD]                [BDD]
          users_db           orders_db           notif_db
```

### Tableau comparatif

| Dimension | Monolithe | Microservices |
|-----------|-----------|---------------|
| **Complexite initiale** | Faible | Elevee |
| **Deploiement** | Simple (un seul artefact) | Complexe (N services) |
| **Scalabilite** | Verticale (toute l'app) | Horizontale (par service) |
| **Isolation des pannes** | Faible (un bug = app entiere) | Elevee (un service = son scope) |
| **Technologie** | Unique | Heterogene (polyglot) |
| **Consistance donnees** | Forte (transactionnelle) | Eventuelle (eventual consistency) |
| **Communication** | Appels directs en memoire | HTTP/gRPC/Messages (reseau) |
| **Equipes** | Une seule equipe | Equipes independantes par service |
| **Debug** | Simple (trace locale) | Complexe (tracing distribue) |
| **Cas d'usage ideal** | Startup, MVP, equipe < 10 | Scale-up, equipes multiples |

> [!warning] Les microservices ne sont pas toujours la bonne reponse
> Martin Fowler lui-meme (qui a popularise le terme) dit : *"Don't start with microservices."* Commencez avec un monolithe bien structure, puis extrayez des services quand les frontieres metier sont claires et que la charge le justifie. Les microservices rajoutent une complexite operationnelle considerable.

---

## Domain-Driven Design : trouver les frontieres

Le **DDD (Domain-Driven Design)** aide a identifier les **Bounded Contexts** — les frontieres naturelles entre services :

```
Domaine e-commerce
├── Bounded Context : Catalogue (produits, categories, prix)
├── Bounded Context : Commandes (panier, checkout, historique)
├── Bounded Context : Paiements (transactions, remboursements)
├── Bounded Context : Livraison (expeditions, tracking)
└── Bounded Context : Utilisateurs (profils, authentification)
```

Chaque Bounded Context devient un microservice potentiel. Les **entites** d'un contexte peuvent avoir le meme nom dans un autre (`Produit` dans Catalogue vs `Produit` dans Commande) mais avec des attributs differents — c'est normal.

---

## Spring Cloud : l'ecosysteme microservices Java

**Spring Cloud** est un ensemble de bibliotheques construites sur Spring Boot pour les problematiques distribuees :

| Composant | Role | Implementation |
|-----------|------|----------------|
| **API Gateway** | Point d'entree unique | Spring Cloud Gateway |
| **Service Discovery** | Registry des services | Eureka Server |
| **Load Balancer** | Repartition de charge | Spring Cloud LoadBalancer |
| **Config Server** | Configuration centralisee | Spring Cloud Config |
| **Circuit Breaker** | Resilience aux pannes | Resilience4j |
| **Tracing** | Trace distribuee | Micrometer Tracing + Zipkin |

---

## Service Discovery avec Eureka

Eureka est un **registre de services** : chaque service s'enregistre au demarrage et annonce son adresse. Les autres services interrogent Eureka pour trouver leurs pairs — pas besoin d'adresses IP codees en dur.

### Eureka Server

```xml
<!-- pom.xml du serveur Eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer  // Active le serveur de registry
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml du serveur Eureka
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false  # Le serveur ne s'enregistre pas lui-meme
    fetch-registry: false
  server:
    wait-time-in-ms-when-sync-empty: 0
```

### Client Eureka (chaque microservice)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
# application.yml d'un microservice
spring:
  application:
    name: orders-service  # Nom avec lequel il s'enregistre dans Eureka

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

```java
@SpringBootApplication
@EnableDiscoveryClient  // Marque ce service comme client Eureka
public class OrdersServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrdersServiceApplication.class, args);
    }
}
```

---

## API Gateway avec Spring Cloud Gateway

L'**API Gateway** est le **point d'entree unique** de votre systeme : elle reçoit toutes les requetes clients, les route vers les bons services, et peut appliquer des politiques transversales (authentification, rate limiting, logging).

```xml
<!-- pom.xml de la Gateway -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
# application.yml de la Gateway
server:
  port: 8080

spring:
  application:
    name: api-gateway
  
  cloud:
    gateway:
      routes:
        # Route vers Users Service
        - id: users-route
          uri: lb://users-service   # lb:// = load balancer via Eureka
          predicates:
            - Path=/api/users/**    # Toutes les requetes /api/users/** vont vers users-service
          filters:
            - StripPrefix=1         # Retire /api du chemin avant de router
        
        # Route vers Orders Service
        - id: orders-route
          uri: lb://orders-service
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - AddRequestHeader=X-Internal-Request, true  # Ajoute un header
        
        # Route avec rate limiting
        - id: public-route
          uri: lb://catalogue-service
          predicates:
            - Path=/api/catalogue/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10   # 10 requetes/seconde
                redis-rate-limiter.burstCapacity: 20
```

---

## Communication inter-services

### RestTemplate (synchrone, classique)

```java
@Service
public class OrderService {
    
    private final RestTemplate restTemplate;
    
    public OrderService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    
    public UserDto getUserDetails(Long userId) {
        // lb://users-service = load balancing via Eureka
        String url = "lb://users-service/users/" + userId;
        return restTemplate.getForObject(url, UserDto.class);
    }
    
    public void createOrder(OrderRequest request) {
        // Verifier que l'utilisateur existe
        UserDto user = getUserDetails(request.userId());
        if (user == null) {
            throw new IllegalArgumentException("Utilisateur inexistant");
        }
        // ... creer la commande
    }
}

// Configuration du RestTemplate avec load balancing
@Configuration
public class RestConfig {
    
    @Bean
    @LoadBalanced  // Active le load balancing via Eureka
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### Feign Client (synchrone, declaratif)

**Feign** est une approche plus elegante : vous declarez l'interface HTTP comme une interface Java, et Feign genere l'implementation :

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
// Activer Feign sur l'application
@SpringBootApplication
@EnableFeignClients
public class OrdersApplication { ... }

// Declarer le client comme une interface
@FeignClient(name = "users-service", fallback = UserClientFallback.class)
public interface UserClient {
    
    @GetMapping("/users/{id}")
    UserDto getUserById(@PathVariable Long id);
    
    @PostMapping("/users/{id}/notify")
    void notifyUser(@PathVariable Long id, @RequestBody NotificationRequest request);
}

// Fallback en cas de panne du users-service
@Component
public class UserClientFallback implements UserClient {
    
    @Override
    public UserDto getUserById(Long id) {
        // Retourner un utilisateur par defaut ou null
        return new UserDto(id, "Utilisateur inconnu", "");
    }
    
    @Override
    public void notifyUser(Long id, NotificationRequest request) {
        // Logguer mais ne pas crasher
        log.warn("Impossible de notifier l'utilisateur {} : users-service indisponible", id);
    }
}

// Utilisation dans le service
@Service
public class OrderService {
    
    private final UserClient userClient;
    private final OrderRepository orderRepository;
    
    // Feign injecte automatiquement l'implementation
    public OrderService(UserClient userClient, OrderRepository orderRepository) {
        this.userClient = userClient;
        this.orderRepository = orderRepository;
    }
    
    public OrderResponse createOrder(OrderRequest request) {
        UserDto user = userClient.getUserById(request.userId());
        Order order = new Order(user.id(), request.produits(), request.total());
        Order saved = orderRepository.save(order);
        
        // Notifier l'utilisateur (async serait mieux, mais synchrone pour l'exemple)
        userClient.notifyUser(user.id(), new NotificationRequest("Commande creee : " + saved.getId()));
        
        return OrderResponse.from(saved);
    }
}
```

### Apache Kafka : communication asynchrone

Pour les communications **sans couplage temporel** (le producteur et le consommateur n'ont pas besoin d'etre disponibles en meme temps), **Kafka** est le standard :

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: orders-group
      auto-offset-reset: earliest
```

```java
// Producteur : Orders Service publie un evenement
@Service
public class OrderService {
    
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public void creerCommande(OrderRequest request) {
        // ... logique metier
        
        // Publier l'evenement sur le topic "order-created"
        OrderEvent event = new OrderEvent(order.getId(), order.getUserId(), order.getTotal());
        kafkaTemplate.send("order-created", String.valueOf(order.getId()), event);
        
        log.info("Evenement publie sur order-created : {}", event);
    }
}

// Consommateur : Notification Service ecoute les evenements
@Service
public class NotificationConsumer {
    
    @KafkaListener(topics = "order-created", groupId = "notification-group")
    public void handleOrderCreated(OrderEvent event) {
        log.info("Commande recue : {}", event.orderId());
        // Envoyer un email, une notification push, etc.
        envoyerEmail(event.userId(), "Commande #" + event.orderId() + " confirmee !");
    }
}

// Classe evenement (peut etre un record)
public record OrderEvent(Long orderId, Long userId, BigDecimal total) {}
```

---

## Configuration centralisee : Spring Cloud Config

Au lieu d'avoir un `application.yml` par service (difficile a gerer), **Spring Cloud Config Server** centralise la configuration dans un depot Git :

```
config-repo/
├── application.yml          # Config commune a tous les services
├── users-service.yml        # Config specifique au users-service
├── orders-service.yml       # Config specifique au orders-service
└── application-prod.yml     # Config commune en production
```

```java
// Config Server
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication { ... }
```

```yaml
# application.yml du Config Server
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/mon-org/config-repo
          default-label: main
```

```yaml
# bootstrap.yml de chaque microservice client
spring:
  application:
    name: orders-service
  config:
    import: configserver:http://localhost:8888
```

---

## Resilience avec Resilience4j

Quand un service appele est lent ou tombe en panne, il ne faut pas que ca propage la panne a tout le systeme. **Resilience4j** implemente plusieurs patterns de resilience :

### Circuit Breaker

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
```

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      users-service:
        sliding-window-size: 10         # Fenetre de 10 appels
        failure-rate-threshold: 50      # Ouvre si > 50% d'echecs
        wait-duration-in-open-state: 30s # Reste ouvert 30s avant de retry
        permitted-calls-in-half-open-state: 3
  
  retry:
    instances:
      users-service:
        max-attempts: 3
        wait-duration: 500ms
  
  timelimiter:
    instances:
      users-service:
        timeout-duration: 3s            # Timeout apres 3 secondes
```

```java
@Service
public class OrderService {
    
    private final UserClient userClient;
    
    // Les annotations appliquent les patterns dans cet ordre : TimeLimiter -> CircuitBreaker -> Retry
    @CircuitBreaker(name = "users-service", fallbackMethod = "getUserFallback")
    @Retry(name = "users-service")
    @TimeLimiter(name = "users-service")
    public CompletableFuture<UserDto> getUserAsync(Long userId) {
        return CompletableFuture.supplyAsync(() -> userClient.getUserById(userId));
    }
    
    // Methode de fallback appelee quand le circuit est ouvert
    public CompletableFuture<UserDto> getUserFallback(Long userId, Throwable t) {
        log.warn("Circuit ouvert pour users-service, fallback pour userId={}", userId, t);
        return CompletableFuture.completedFuture(
            new UserDto(userId, "Utilisateur temporairement indisponible", "")
        );
    }
}
```

```
Circuit Breaker — 3 etats :

  CLOSED (normal)
     │
     │ > 50% echecs sur 10 appels
     v
  OPEN (bloque, retourne le fallback immediatement)
     │
     │ apres 30s
     v
  HALF-OPEN (laisse passer 3 appels de test)
     │
     ├── succes -> CLOSED
     └── echec  -> OPEN
```

---

## Observabilite : tracing distribue

Dans un systeme distribue, une requete passe par plusieurs services. Sans **tracing distribue**, impossible de savoir ou un probleme se produit.

### Micrometer Tracing + Zipkin

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

```yaml
# application.yml
management:
  tracing:
    sampling:
      probability: 1.0  # 100% des requetes tracees (reduire en prod)
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
```

Chaque requete reçoit un **Trace ID** unique qui la suit a travers tous les services. Zipkin visualise le chemin complet :

```
Trace ID: abc123
  |
  ├── api-gateway (2ms)
  │     |
  │     ├── orders-service (45ms)
  │     │     |
  │     │     ├── users-service (12ms)   ← appel Feign
  │     │     └── orders-db (8ms)        ← requete JPA
  │     │
  │     └── Response 200 OK
```

### Prometheus + Grafana : metriques

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  endpoint:
    health:
      show-details: always
```

Spring Boot Actuator expose automatiquement `/actuator/prometheus` avec des metriques JVM, HTTP, BDD, custom. Prometheus scrape ces metriques, Grafana les visualise en dashboards.

---

## Dockerisation d'un service Spring Boot

```dockerfile
# Dockerfile multi-stage : separe la compilation de l'execution
# Stage 1 : compilation
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN --mount=type=cache,target=/root/.m2 \
    mvn clean package -DskipTests

# Stage 2 : image finale legere (sans JDK, seulement JRE)
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Utilisateur non-root pour la securite
RUN addgroup -S spring && adduser -S spring -G spring
USER spring

# Copier le JAR depuis le stage de compilation
COPY --from=builder /app/target/*.jar app.jar

# Variables d'environnement avec valeurs par defaut
ENV JAVA_OPTS="-Xms256m -Xmx512m"
ENV SPRING_PROFILES_ACTIVE=prod

EXPOSE 8080

# Lancement avec support des signaux Unix (SIGTERM pour graceful shutdown)
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

```bash
# Build et run
docker build -t taskapi:1.0 .
docker run -p 8080:8080 -e SPRING_PROFILES_ACTIVE=dev taskapi:1.0
```

---

## Docker Compose : orchestrer plusieurs services en local

```yaml
# docker-compose.yml — orchestration locale de tout le systeme
version: '3.8'

services:
  
  # Infrastructure
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: taskdb
      POSTGRES_USER: taskuser
      POSTGRES_PASSWORD: taskpass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U taskuser"]
      interval: 5s
      timeout: 5s
      retries: 5
  
  kafka:
    image: confluentinc/cp-kafka:7.6.0
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
  
  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
  
  eureka-server:
    image: eureka-server:latest
    build: ./eureka-server
    ports:
      - "8761:8761"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8761/actuator/health"]
      interval: 10s
      retries: 5
  
  api-gateway:
    image: api-gateway:latest
    build: ./api-gateway
    ports:
      - "8080:8080"
    environment:
      EUREKA_CLIENT_SERVICE-URL_DEFAULTZONE: http://eureka-server:8761/eureka/
    depends_on:
      eureka-server:
        condition: service_healthy
  
  users-service:
    image: users-service:latest
    build: ./users-service
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/taskdb
      SPRING_DATASOURCE_USERNAME: taskuser
      SPRING_DATASOURCE_PASSWORD: taskpass
      EUREKA_CLIENT_SERVICE-URL_DEFAULTZONE: http://eureka-server:8761/eureka/
    depends_on:
      postgres:
        condition: service_healthy
      eureka-server:
        condition: service_healthy
  
  orders-service:
    image: orders-service:latest
    build: ./orders-service
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/taskdb
      SPRING_KAFKA_BOOTSTRAP-SERVERS: kafka:9092
      EUREKA_CLIENT_SERVICE-URL_DEFAULTZONE: http://eureka-server:8761/eureka/
    depends_on:
      - postgres
      - kafka
      - eureka-server
  
  # Observabilite
  zipkin:
    image: openzipkin/zipkin:latest
    ports:
      - "9411:9411"
  
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  postgres_data:
  grafana_data:
```

```bash
# Lancer tout le systeme
docker-compose up -d

# Voir les logs d'un service specifique
docker-compose logs -f orders-service

# Arreter proprement
docker-compose down

# Arreter et supprimer les volumes (reset BDD)
docker-compose down -v
```

---

## Patterns avances

### Pattern Saga : gestion des transactions distribuees

Dans un monolithe, une transaction ACID garantit la coherence. Dans les microservices, chaque service a sa propre BDD — pas de transaction globale possible. Le **Saga pattern** coordonne une serie de transactions locales compensables :

```
Commande e-commerce — Saga choreographie

  Orders Service          Payments Service        Inventory Service
       │                        │                        │
  [CreateOrder]                 │                        │
       │                        │                        │
  pub: order-created ───────────►                        │
                          [ProcessPayment]               │
                          pub: payment-ok ───────────────►
                                                  [ReserveItems]
                                                  pub: items-reserved
                                                         │
                                              ◄──────────┘ (commande confirmee)
```

Si le paiement echoue :

```
  [CreateOrder] → pub: order-created
      ↓
  [ProcessPayment fails] → pub: payment-failed
      ↓
  [Orders Service] reçoit payment-failed → annule la commande (compensation)
```

```java
// Exemple simplifie de saga choreographiee avec Kafka
@Service
public class OrdersSagaHandler {
    
    @KafkaListener(topics = "payment-failed")
    public void handlePaymentFailed(PaymentFailedEvent event) {
        // Transaction de compensation : annuler la commande
        orderRepository.findById(event.orderId())
            .ifPresent(order -> {
                order.setStatus(OrderStatus.CANCELLED);
                order.setCancellationReason("Paiement refuse : " + event.reason());
                orderRepository.save(order);
                log.warn("Commande {} annulee suite a echec paiement", event.orderId());
            });
    }
    
    @KafkaListener(topics = "items-reserved")
    public void handleItemsReserved(ItemsReservedEvent event) {
        orderRepository.findById(event.orderId())
            .ifPresent(order -> {
                order.setStatus(OrderStatus.CONFIRMED);
                orderRepository.save(order);
                kafkaTemplate.send("order-confirmed", new OrderConfirmedEvent(order.getId()));
            });
    }
}
```

### CQRS : separer lecture et ecriture

**CQRS (Command Query Responsibility Segregation)** separe les operations de modification (Commands) des operations de lecture (Queries) — permettant d'optimiser chaque cote independamment :

```java
// COMMAND SIDE : modifie l'etat
public interface CommandHandler<C extends Command> {
    void handle(C command);
}

public record CreateTaskCommand(String titre, String description, Long userId) implements Command {}

@Service
public class CreateTaskCommandHandler implements CommandHandler<CreateTaskCommand> {
    
    private final TaskRepository taskRepository;
    private final KafkaTemplate<String, TaskCreatedEvent> kafka;
    
    @Override
    @Transactional
    public void handle(CreateTaskCommand command) {
        Task task = new Task(command.titre(), command.description());
        Task saved = taskRepository.save(task);
        
        // Publier un evenement pour mettre a jour le read model
        kafka.send("task-events", new TaskCreatedEvent(saved.getId(), saved.getTitre()));
    }
}

// QUERY SIDE : lecture optimisee (peut utiliser une BDD denormalisee, un cache, Elasticsearch...)
@Service
public class TaskQueryService {
    
    // Read model : version denormalisee optimisee pour la lecture
    private final TaskReadRepository readRepository;
    
    public TaskSummaryDto getTaskSummary(Long id) {
        return readRepository.findSummaryById(id);
    }
    
    public List<TaskListItemDto> searchTasks(TaskSearchQuery query) {
        return readRepository.search(query);
    }
}
```

### Event Sourcing (introduction)

Au lieu de stocker **l'etat actuel**, on stocke la **suite des evenements** qui ont amene a cet etat :

```java
// Event Store : journal append-only de tous les evenements
@Entity
public class TaskEvent {
    @Id @GeneratedValue
    private Long id;
    
    private Long taskId;
    private String eventType;         // "CREATED", "COMPLETED", "DELETED"
    private String payload;           // JSON de l'evenement
    private LocalDateTime occurredAt;
}

// Pour reconstituer l'etat courant d'une tache :
// Charger tous les evenements lies a taskId et les rejouer dans l'ordre
public Task reconstituteTask(Long taskId) {
    List<TaskEvent> events = eventStore.findByTaskId(taskId);
    Task task = new Task();
    for (TaskEvent event : events) {
        task.apply(event);  // chaque evenement mute l'etat
    }
    return task;
}
```

> [!info] CQRS et Event Sourcing
> CQRS et Event Sourcing sont souvent utilises ensemble mais sont independants. CQRS separe lecture/ecriture. Event Sourcing stocke les evenements plutot que l'etat. Ensemble, ils permettent : audit complet, replay pour corriger des bugs, projections multiples depuis les memes donnees.

---

## Liens et references

Pour approfondir les sujets d'infrastructure et d'observabilite abordes dans ce cours :

- [[04 - Kubernetes Introduction]] — orchestration de conteneurs a grande echelle
- [[03 - Tracing et Debugging Distribue]] — Zipkin, Jaeger, correlation des traces
- [[03 - Docker Compose en Pratique]] — maitriser Docker Compose pour les environnements locaux
- [[02 - Metriques et Monitoring]] — Prometheus, Grafana, alerting

---

## Exercices pratiques

> [!tip] Pour aller plus loin
> Ces exercices sont progressifs. Les exercices 1-3 sont realisables localement avec Docker Compose. L'exercice 4 necessite une connaissance de [[04 - Kubernetes Introduction]].

### Exercice 1 — Premier microservice

Creer deux microservices `users-service` et `tasks-service` :
- `users-service` (port 8081) : CRUD utilisateurs avec PostgreSQL
- `tasks-service` (port 8082) : CRUD taches liees a des utilisateurs, appel Feign vers `users-service` pour valider que l'utilisateur existe avant de creer une tache
- Eureka Server (port 8761) pour le service discovery
- Docker Compose qui lance les 3 services + PostgreSQL

### Exercice 2 — API Gateway

Ajouter une **Spring Cloud Gateway** (port 8080) qui :
- Route `/api/users/**` vers `users-service`
- Route `/api/tasks/**` vers `tasks-service`
- Ajoute un header `X-Request-ID` avec un UUID unique a chaque requete (via filtre custom)
- Retourne 503 si le service cible est indisponible (via fallback)

### Exercice 3 — Resilience et Kafka

Ajouter la resilience et la communication asynchrone :
- Circuit Breaker avec Resilience4j sur l'appel Feign `tasks → users` (fallback retourne un utilisateur "anonyme")
- Quand une tache est creee : `tasks-service` publie un evenement Kafka `task-created`
- Creer un `notifications-service` qui ecoute `task-created` et simule l'envoi d'une notification (log dans la console)
- Tester le circuit breaker en arretant `users-service` et en continuant a appeler `tasks-service`

### Exercice 4 — Observabilite complete

Configurer l'observabilite du systeme :
- Micrometer Tracing sur les 3 services avec export vers **Zipkin**
- Prometheus scraping des metriques Actuator de chaque service
- Dashboard **Grafana** minimal : nombre de requetes/seconde, taux d'erreur, latence p95
- Ajouter une metrique custom dans `tasks-service` : `tasks.created.total` (compteur incremente a chaque creation)

### Exercice 5 — Pattern Saga

Implementer un flow de commande simplifie avec Saga :
1. `orders-service` cree une commande et publie `order-placed`
2. `payments-service` ecoute `order-placed`, simule un paiement (succes 80%, echec 20%) et publie `payment-ok` ou `payment-failed`
3. `orders-service` ecoute `payment-ok` → confirme la commande ; ecoute `payment-failed` → annule la commande
4. Verifier la coherence : chaque commande doit finir soit CONFIRMED soit CANCELLED, jamais bloquee

> [!warning] Conseil de debugging microservices
> Activez **toujours** le tracing distribue avant de debugger. Sans Trace ID commun, retrouver la cause d'une erreur dans un systeme de 5 services est extremement laborieux. Consultez [[03 - Tracing et Debugging Distribue]] pour les details d'implementation.
